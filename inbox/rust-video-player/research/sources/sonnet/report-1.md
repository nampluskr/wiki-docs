# 조사 보고서 (sonnet) — Rust 동영상 플레이어 (gpui·Tauri, Windows 11) — 1/2

- 작성: rvp-scouter-sonnet / 확인 기준일 2026-11-01. 저장 폴더는 지시문의 `sources$n`을 `sources\sonnet`으로 해석했다(미치환 변수로 판단, 추정).
- 2/2 는 `report-2.md` (Q9~Q17). 표기: **[확인]** = 이번에 직접 열어 봄, **[일반지식]** = 기억에 의존, 검증 못 함, **추정**, **미확인**.
- 출처 약칭: CIO=`https://crates.io/api/v1/crates/<이름>` (2026-11-01 조회), GH=`https://api.github.com/repos/<소유자/저장소>` (gh api, 2026-11-01), GPUI-SRC=로컬 `~/.cargo/registry/src/*/gpui-pre-0.3.10` 및 `gpui-pre-windows-0.3.10` (workbench-gpui 가 쓰는 버전의 실제 소스).

## 요약 (10줄)

1. **gpui 1순위: libmpv(render API 또는 --wid) 또는 FFmpeg 직접 + 소프트 업로드.** gpui(`gpui-pre 0.3.10`)의 `paint_surface`는 macOS/iOS 전용이고, Windows `draw_surfaces`는 빈 함수다 [확인 GPUI-SRC window.rs:5173, directx_renderer.rs:816]. Windows 에서 영상 프레임을 그릴 방법은 `RenderImage`(BGRA CPU 버퍼 → 스프라이트 아틀라스 `UpdateSubresource` 업로드)뿐이다.
2. 그래서 gpui 는 **(A) libmpv 를 자식 HWND 로 붙이는 방식(R1·R2 를 mpv 에 위임, 가장 쉬움, 단 Z-순서/오버레이 제약)** 과 **(B) FFmpeg 디코딩 + CPU BGRA → `RenderImage`(1080p 까지 현실적, 4K60 은 한계)** 의 두 길이 현실적이다. 기존 gpui 사례 `gpui-video-player 0.1.3`(GStreamer)도 Windows 는 소프트 렌더링이다 [확인 lib.rs].
3. **Tauri 2 는 두 길**: WebView2 `<video>`(구현 최소, 그러나 MKV/AC3/ASS/트랙 선택/GPU 끄기 한계) vs 네이티브(mpv `--wid` 자식 창 또는 별도 디코딩). 범용 재생 요구(MKV·AC3·ASS·트랙)는 `<video>` 만으로는 안 된다(추정, 아래 Q5).
4. **R1·R2 를 가장 쉽게 만족: libmpv.** `hwdec` 속성(`no`/`auto`/`d3d11va`/`d3d11va-copy`)을 재생 중 바꿀 수 있고(Ctrl+h 가 auto↔no 토글 [확인 mpv options.rst:1340]), `vf` 추가/제거/토글, `vf-command`(lavfi 필터만), `brightness/contrast/saturation` 속성, `--deinterlace` 가 모두 문서화돼 있다 [확인].
5. **FFmpeg 필터 라이선스 함정 [확인 FFmpeg configure @master]**: `boxblur`, `eq`, `hqdn3d` 는 `--enable-gpl` 필요(GPL 전용). `unsharp`, `gblur`, `nlmeans`, `yadif`, `bwdif` 는 GPL 의존 표시 없음. 밝기·대비·채도(`eq`)와 노이즈 제거(`hqdn3d`)가 GPL 이라 LGPL 배포와 충돌 → 대체: `colorlevels`/`colorbalance`/`lut`/`nlmeans`, 또는 셰이더/D3D11.
6. **libmpv 바인딩**: `libmpv2 7.0.0`(2026-10-30, LGPL-2.1, kohsine, 별 66) 이 유일하게 활발. 옛 `libmpv`/`libmpv-sys` 는 2020 이후 정지 [확인 CIO]. 버전이 1주일에 6→7 로 올라 API 불안정(추정).
7. **FFmpeg 바인딩**: `ffmpeg-next 9.0.0`(2026-08-05, WTFPL), `ffmpeg-the-third 6.0.0+ffmpeg-9.0`(2026-08-09), `rsmpeg 0.18.0+ffmpeg.8.0`(2025-08-24 이후 릴리스 없음, 단 저장소 push 동일). Windows 빌드는 vcpkg/prebuilt 필요(Q12).
8. **순수 Rust 로는 범용 재생 불가**: `symphonia 0.6.1`(오디오 전용, MPL-2.0), `rav1d 1.1.0`(AV1 만, 2025-05 이후 릴리스 없음), `openh264 0.9.9`(H.264 만, BSD-2, 시스템 H.264 특허료는 Cisco 바이너리 쓸 때 면제). HEVC/VP9/AC3·MKV·자막이 빠진다(추정).
9. **Windows 최대 위험**: ① gpui 에 네이티브 비디오 표면이 없어 4K/HDR 에서 CPU 복사 병목, 자식 HWND 는 gpui 요소 위에 겹쳐 그리기 어려움 ② 코덱·DLL 라이선스(GPL 필터, HEVC 특허, mpv 빌드가 GPL) ③ 하드웨어 디코딩이 환경마다 다르게 실패(D3D11 vs DXVA2, 드라이버)하며 재생 중 전환 시 프레임/스레드 정리 필요.
10. **베이스 앱 현황** [확인 파일 읽음]: workbench-gpui 는 `gpui-kit =0.8.1` 만 의존하고 `Workbench` 구조체가 `editor_poc` 를 "가운데 Pane 의 활성 아이템 자리"에 둔다(`src/ui/mod.rs`). 탭 바는 정적 그리기 PoC(`layout.rs tab_bar`)이고 Pane/Item 추상은 아직 없다. workbench-tauri 는 `App.tsx` 4줄, `lib.rs` 6줄의 빈 껍데기 — 뷰어 구조는 새로 설계해야 한다.

---

## Q1 권장안

### gpui 탭 (1순위)
| 순위 | 조합 | 이유 |
|---|---|---|
| 1 | **libmpv 렌더 컨텍스트가 아니라 `--wid` 자식 HWND** (디코딩·필터·자막·오디오·동기화 전부 mpv) | R1·R2·트랙·자막·속도·탐색이 mpv 속성/명령으로 즉시 충족 [확인 mpv 문서]. gpui 에는 D3D11 텍스처를 직접 받는 경로가 없다. 단점은 Q4 참조(겹침·포커스·분할). |
| 2 | **FFmpeg(`ffmpeg-next`/`-the-third`) 디코딩 + swscale/libswscale 으로 BGRA 변환 + gpui `RenderImage` + cpal 오디오** | gpui 안에서 완전 제어(오버레이 컨트롤, 탭 분할 정상). 1080p60 정도까지, 4K 는 CPU 복사 한계(추정, Q4). 필터는 libavfilter. |
| 3 | GStreamer `playbin3`+`appsink`(BGRA/RGBA) → `RenderImage` | `gpui-video-player 0.1.3`(MIT, GStreamer, Windows 는 아틀라스 소프트 경로)가 선례 [확인 `https://lib.rs/crates/gpui-video-player`]. 단 GStreamer 런타임 설치 부담(Q12). |

핵심 근거: gpui Windows 렌더러는 D3D11 이고(`directx_renderer.rs`, `ID3D11Texture2D` 아틀라스) `paint_surface` 는 `#[cfg(any(target_os="macos",target_os="ios"))]` [확인 GPUI-SRC window.rs:5170-5176]. 스프라이트 아틀라스 텍스처는 `DXGI_FORMAT_B8G8R8A8_UNORM` [확인 directx_atlas.rs:185]. 그래서 YUV 텍스처를 그대로 넘기는 길이 없다.

### Tauri 탭 (2순위)
| 순위 | 조합 | 이유 |
|---|---|---|
| 1 | **mpv `--wid` 를 쓰는 `tauri-plugin-libmpv 0.3.2` 류 또는 직접 자식 창** | R1·R2·자막·트랙을 mpv 에 위임. 단 웹뷰와 네이티브 자식 창의 겹침 문제(Q5). |
| 2 | WebView2 `<video>` + 사용자 정의 프로토콜 Range 서빙 + (필요시) 서버측 변환 | 구현 최소. 아래 한계. |
| 3 | 프레임 IPC(Rust 디코딩 → 캔버스/WebGL) | 4K 에서 비쌈(Q5). |

**`<video>` 만으로 되는가 — 아니오(추정)**: WebView2 는 Edge(Chromium) 런타임이라 MKV/WebM/MP4, H.264/VP9/AV1, AAC/Opus 는 대체로 되나, HEVC 는 Windows 의 HEVC 확장(하드웨어 디코딩 한정)에 의존하고 AC3 는 불확실하며, ASS 자막·오디오 트랙 선택 UI·GPU 가속 끄기(R1)·sharpen/denoise/deinterlace(R2)가 표준 `<video>` 에 없다. CSS `filter`(brightness/contrast/saturate/blur)는 가능하나 sharpen/denoise/deinterlace 는 WebGL 셰이더 후처리가 필요(추정). 출처: 포럼 수준의 비공식 자료만 확인 `https://emby.media/community/topic/102877-question-about-browser-support-for-hevc`, `https://techcommunity.microsoft.com/tag/hevc` — **WebView2 공식 코덱 표는 미확인**.

---

## Q2 파이프라인 구조 (FFmpeg/mpv 방식 요약 — 일반지식 중심, 구조는 ffplay/mpv 공통)

```
file ─ demux(avformat) ─┬─ video packet queue ─ decode(HW/SW) ─ filter(libavfilter) ─ video frame queue ─ present
                        ├─ audio packet queue ─ decode ─ resample/atempo ─ audio ring buffer ─ cpal callback
                        └─ subtitle packet queue ─ decode ─ libass render (video frame 위에 합성)
```
- **스레드**: demux 1, video decode 1, audio decode 1(또는 합침), 표시 타이머 1(또는 UI 스레드), 오디오는 cpal 이 호출하는 실시간 콜백 스레드. 콜백은 잠금·할당·디스크 I/O 금지(링버퍼만).
- **버퍼링**: 패킷 큐는 바이트/시간 상한(예: 몇 MB 또는 수 초), 프레임 큐는 소수(비디오 3~6장) — 큐가 가득 차면 상류가 블록(역압).
- **상태 기계**: `Idle → Opening → Playing ⇄ Paused → Seeking → Playing/Paused → Ended → (Closed)`. 에러는 `Error(kind)` 로 분리, `Seeking` 은 중첩 요청을 마지막 값으로 합치기.
- **A/V 동기화**: 오디오 시계를 마스터로 삼고(보통 정답), 비디오 프레임 PTS 와 오디오 시계 차이로 대기(early)·반복(late 소폭)·드롭(late 대폭)을 결정. 속도 변경은 오디오는 `atempo`/scaletempo2 로 피치 보정, 비디오는 시계 배율을 곱한다. mpv 는 `audio-pitch-correction` 이 기본 켜짐이며 속도가 정상이 아니면 `scaletempo2` 를 자동 삽입한다고 문서에 있다 [확인 `https://mpv.io/manual/stable/`]. ffplay 의 동기 기준 기본이 오디오 마스터라는 점은 [일반지식].
- **탐색**: 키프레임 탐색은 `av_seek_frame(...BACKWARD)` 로 빠르나 위치가 어긋남. 정확 탐색은 키프레임으로 가서 목표 PTS 까지 디코딩만 하고 버리기(`avcodec_flush_buffers` 후). **순서**: ① 탐색 세대 번호 증가 → ② 모든 큐 비우기 + 디코더 flush → ③ demux seek → ④ 새 세대 패킷만 수용 → ⑤ 오디오 링버퍼 비우고 시계 재설정. mpv 는 `seek` 명령에 `absolute/relative/exact/keyframes` 플래그 [일반지식, input.rst 의 seek 항목은 이번에 열지 않음].
- 하드웨어 디코딩 프레임은 `hw_frames_ctx` 에 묶여 있어 필터/flush/재생 중 디코더 교체 시 수명 관리가 필요(Q6).

---

## Q3 디코딩 백엔드 비교 (CIO·GH 2026-11-01)

| 백엔드 | 크레이트/버전 | 최근 릴리스 | 라이선스 | 유지 | 장점 | 단점 |
|---|---|---|---|---|---|---|
| FFmpeg | `ffmpeg-next 9.0.0` (zmwangx/rust-ffmpeg, 별 2021) | 2026-08-05 | WTFPL | 활발(push 2026-10-09) | 범용 최강, hwaccel·libavfilter·자막 | 안전 래퍼가 불완전(raw FFI 필요), 빌드/DLL, GPL/LGPL 선택 |
| FFmpeg(포크) | `ffmpeg-the-third 6.0.0+ffmpeg-9.0` (별 162) | 2026-08-09 | WTFPL | 활발 | ffmpeg-next 포크, 새 FFmpeg 추종 | 별 적음 |
| FFmpeg(저수준) | `rsmpeg 0.18.0+ffmpeg.8.0` (larksuite, 별 880) | 2025-08-24 | MIT | 1년+ 릴리스 없음(저장소 push 2025-08-24) | 바인딩 두껍지 않고 rusty_ffmpeg 기반 | FFmpeg 9 추종 안 됨(추정) |
| FFmpeg(sys) | `ffmpeg-sys-next 9.0.1`, `rusty_ffmpeg 0.17.0+ffmpeg.8.1` | 2026-10-09 / 2026-04-10 | WTFPL / MIT | 활발 | 직접 FFI | unsafe |
| GStreamer | `gstreamer 0.25.4`, `gstreamer-app 0.25.2`, `gstreamer-video 0.25.4` | 2026-09-21 / 2026-05-11 / 2026-09-21 | MIT OR Apache-2.0 | 매우 활발(gitlab) | playbin3 로 포맷·트랙·자막·HW(d3d11 요소) 자동, Rust 친화 | Windows 런타임 대형 설치, 프로파일/플러그인 조합 복잡 |
| libmpv | `libmpv2 7.0.0` (kohsine, 별 66), `libmpv2-sys 4.0.1` | 2026-10-30 / 2025-12-29 | LGPL-2.1(바인딩) | 활발하나 API 불안정 | R1·R2·자막·트랙 전부 내장, render API/`--wid` | libmpv DLL 은 빌드에 따라 GPL, DLL 구하기, 바인딩 소규모 |
| Media Foundation | `windows 0.62.2` 크레이트 | 2025-10-06 | MIT OR Apache-2.0 | 활발 | Windows 내장 코덱·HW 가속, 라이선스 걱정 적음 | MKV/ASS/AC3 등 컨테이너·코덱 한계(일반지식), 직접 구현량 많음 |
| 순수 Rust | `symphonia 0.6.1`(오디오), `rav1d 1.1.0`(AV1), `openh264 0.9.9`(H.264) | 2026-08-13 / 2025-05-07 / 2026-10-15 | MPL-2.0 / BSD-2 / BSD-2 | symphonia·openh264 활발, rav1d 느림 | 의존성 단순 | **범용 불가**(HEVC/VP9/MKV 영상 디코더 없음) |

- libmpv `render API`: `MPV_RENDER_API_TYPE_D3D11`(render_d3d11.h), `OPENGL`, `SW` 세 종류 [확인 `https://raw.githubusercontent.com/mpv-player/mpv/master/include/mpv/render.h:52-54`]. `SW` 는 "극단적으로 단순하지만 느리다"고 문서화 [확인 render.h:129]. → gpui 에서 render API 를 쓰려면 SW(느림) 또는 D3D11 을 gpui 가 못 받는 문제로 막힘(추정). 그래서 `--wid` 가 현실적.
- 최소 흐름(FFmpeg): `avformat open → find_stream_info → find decoder (+hw_device_ctx) → loop {read packet → send_packet → receive_frame → (hw→sw transfer) → filter → sws → queue}`.
- 최소 흐름(libmpv): `Mpv::new → set_property("wid", hwnd) → command("loadfile", path) → set_property("hwdec","auto-safe")`.

---

## Q4 gpui 통합

**탭(pane item) 구조 근거**: `workbench-gpui/src/ui/mod.rs` — `struct Workbench { editor_poc: Option<Entity<EditorPoc>>, ... }`, 주석 "있으면 가운데 Pane 의 활성 아이템 자리에 둔다". `layout.rs` 의 `tab_bar(cx)` 는 정적 요소(탭 하나 "tab", 버튼 back/forward/new/split/zoom). 즉 **Item 추상(trait)이 아직 없다**. 뷰어를 넣으려면 `trait PaneItem { fn title; fn render(&mut self, window, cx) -> AnyElement; fn on_activate/deactivate; fn close }` 같은 것을 먼저 만들어 `EditorItem` 과 `VideoItem` 이 구현(추정, 설계안). 의존은 D-3/D-5(gpui-kit 래퍼 안에 가둠) [확인 `specs/v0.1/DECISIONS.md`].

**프레임 그리기 [확인 GPUI-SRC]**
- `RenderImage::new(SmallVec<[Frame;1]>)` — "BGRA format" CPU 이미지. 새 `RenderImage` 마다 새 `ImageId`(전역 카운터) → 아틀라스에 새 항목 업로드. 오래된 것은 `Window::drop_image(Arc<RenderImage>)` 로 해제해야 한다(window.rs:5193).
- `paint_image(bounds, image_bounds, corner_radii, Arc<RenderImage>, frame_index, grayscale)` 는 아틀라스를 `get_or_insert_with` 로 채움 — 내부에서 `UpdateSubresource` 로 업로드(directx_atlas.rs:279-304).
- `paint_surface` / `Surface` 요소는 macOS 전용(`objc2_core_video::CVPixelBuffer`). Windows `draw_surfaces` 는 항목이 있어도 `Ok(())` 만 반환 [확인 directx_renderer.rs:816-821].
- 매 프레임 갱신: `Window::request_animation_frame()`(window.rs:2742) 또는 `cx.notify()` + 타이머. 영상 프레임 도착마다 `cx.notify()`.
- **무복사 불가, YUV→RGB 위치**: 디코더(HW→`av_hwframe_transfer_data`)→CPU NV12→swscale 로 BGRA→아틀라스 업로드. 변환은 CPU(swscale SIMD)에서 일어남. 4K60 BGRA = 3840×2160×4×60 ≈ 1.99 GB/s 메모리 복사 + 변환 → 비현실적일 수 있음(**추정, 계산만**). 1080p60 ≈ 0.5 GB/s 로 가능(추정).
- 대안 개선: 아틀라스 갱신을 `Img` 요소 대신 같은 `RenderImage` 를 재사용하는 API 는 없음(불변 `Arc`). 프레임마다 객체 생성 + `drop_image` 반복이 안전한 방식인지는 **미확인**.
- **libmpv 자식 HWND**: gpui 창 HWND 를 얻어 `CreateWindowEx(WS_CHILD)` 후 `wid` 로 mpv 에 전달. 자식 HWND 는 gpui(D3D11 스왑체인) 위에 항상 덮여 그려져 컨트롤 오버레이·팝업이 가려진다(일반지식, Win32 자식 창 합성 특성). 탭 전환/분할 시 `ShowWindow`/`MoveWindow` 를 레이아웃 변경 이벤트에 동기화해야 한다. gpui 가 HWND 를 노출하는지는 `raw-window-handle` 지원 여부로 확인 필요 — **미확인**.
- **DirectX 11 렌더러**: gpui Windows 는 D3D11 (`D3D_FEATURE_LEVEL_11_1/11_0/10_1`) [확인 directx_devices.rs]. 외부 D3D11 텍스처를 공유하는 공개 API는 찾지 못함(미확인).
- **실제 사례**: `gpui-video-player 0.1.3`(MIT, GStreamer, 2026-10-23) — macOS 는 CVPixelBuffer 하드웨어 경로, Linux/Windows 는 스프라이트 아틀라스 소프트 경로 [확인 `https://lib.rs/crates/gpui-video-player`, 저장소 `https://github.com/cijiugechu/gpui-video-player`]. `gpui-pre =0.3.9` 고정 필요 → workbench 의 `gpui-pre 0.3.10` 과 버전 차이로 호환 확인 필요(추정).

---

## Q5 Tauri 통합

- **탭 구조 근거**: `workbench-tauri/src/App.tsx` 는 `<main>ZedLike</main>` 한 줄, `src-tauri/src/lib.rs` 는 `tauri::Builder::default().run(...)` 만 있다. 프런트는 Solid(`solid-js 1.9.18`), Tauri `2.12.5`, rust-version 1.90 [확인 파일]. 뷰어 구조는 새로 세워야 함.
- **로컬 파일 제공**: `app.security.assetProtocol.enable`(기본 false) + `scope` 글롭 [확인 `https://v2.tauri.app/reference/config/`]. 현재 CSP 가 `default-src 'self' ipc: http://ipc.localhost` 라서 `<video src>` 를 쓰려면 `media-src` 추가가 필요(추정). **asset 프로토콜의 HTTP Range 지원 여부는 미확인** → 사용자 정의 URI 스킴(`register_uri_scheme_protocol`)에서 Range 를 직접 처리하는 쪽이 안전(추정).
- **mpv `--wid`**: `tauri-plugin-libmpv 0.3.2` 는 Tauri 창 핸들을 `wid` 로 넘겨 libmpv 를 로드하고 `libmpv-wrapper` 동적 라이브러리를 사용; 임베딩에는 투명 창 설정이 필요하다고 README 에 있음; 리사이즈/위치는 별도로 맞춰야 하며 `video-margin-ratio-*` 속성 제공 [확인 `https://docs.rs/crate/tauri-plugin-libmpv/latest/source/src/desktop.rs`, README 0.3.1]. 커뮤니티 우회: 자식 WebviewWindow 에 mpv 를 띄우고 부모 이동/리사이즈마다 HTML div 위치에 맞춤 [확인 검색 요약 `https://github.com/orgs/tauri-apps/discussions/6343`].
- **탭 영역에 맞추기**: 웹뷰 위에 네이티브 자식 창은 위에 얹힘 → 탭 전환/모달/드롭다운이 영상에 가려짐. 해법 후보: 영상 영역 밖에서만 UI 배치, 또는 영상 창을 투명 웹뷰 뒤에 두고 웹뷰 배경 투명(`transparent`) 처리 — 플러그인 README 의 "투명 창" 요구와 일치(추정).
- **프레임 IPC**: 1080p RGBA 1프레임 ≈ 8.3 MB, 60fps ≈ 0.5 GB/s 를 IPC/JS 로 보내는 것은 비현실적(계산, 추정). 대안 로컬 WebSocket/공유메모리도 4K 는 어렵다(추정).
- **사례**: `Kenichi Preview`(Tauri v2 + wgpu + `ffmpeg-next`, FFmpeg 6.x 공유 라이브러리 필요) `https://github.com/simplearyan/kenichi-preview` [확인 검색 요약, 저장소 미확인]; `tauri-plugin-mpv 0.5.2`(mpv 프로세스 JSON IPC 제어, `https://docs.rs/crate/tauri-plugin-mpv/latest`) [확인 검색 결과 요약].

---

## Q6 R1 GPU 가속

| 경로 | 켜기/끄기 | 재생 중 전환 | 실패 시 소프트 전환 | 무복사 |
|---|---|---|---|---|
| mpv | `--hwdec=<api,...|no|auto|auto-copy>`; `d3d11va`(vo=gpu/gpu-next + d3d11 컨텍스트 필요), `d3d11va-copy`(RAM 복사) [확인 options.rst:1340-1420] | **가능**: `hwdec` 는 런타임 속성; Ctrl+h 가 `auto`↔`no` 토글 [확인 options.rst:1359-1363 주석] | 문서: "하드웨어 디코딩이 불가능하면 소프트로 대체" [확인 options.rst:1343] ; `hwdec-current` 속성으로 실제 사용 중 API 확인 [확인 input.rst:2853] | `d3d11va` 는 vo 와 같은 D3D11 에서 무복사; `-copy` 는 복사 |
| FFmpeg 직접 | `hw_device_ctx`(AV_HWDEVICE_TYPE_D3D11VA/DXVA2/CUDA/QSV) 만들어 `AVCodecContext` 에 연결, `get_format` 콜백으로 `AV_PIX_FMT_D3D11` 선택 [일반지식] | 디코더 재오픈 필요(해당 시점 키프레임부터); 현재 위치 저장 → flush → 재오픈 | `get_format` 에서 SW 포맷 반환 또는 hw 초기화 실패 시 `hw_device_ctx` 없이 재오픈 | gpui 경로에서는 CPU 전송 불가피 |
| GStreamer | `d3d11h264dec` 등 d3d11 디코더 요소, `d3d11videosink`(GstVideoOverlay 구현, 어댑터 선택) [확인 `https://gstreamer.freedesktop.org/documentation/d3d11/d3d11videosink.html`] | 파이프라인 재구성(디코더 랭크 조정 후 재시작) 필요(추정) | `decodebin` 이 랭크로 후보 시도 | d3d11 요소끼리 무복사, `appsink` 에서는 복사 |

- 하드웨어 종류(일반지식): D3D11VA/DXVA2 는 벤더 중립(Windows 내장), NVDEC·QSV 는 벤더 전용 — 범용 앱은 D3D11VA 를 기본으로 하고 필요 시 선택지로 노출.
- "자동 전환" 설계: 오류 콜백(디코더 오류 N회/프레임 연속 실패/시간 초과) → 위치 기억 → HW 끄고 재오픈 → UI 에 "소프트웨어로 전환됨" 알림.

---

## Q7 R2 영상 옵션

| 기능 | libavfilter | mpv | GStreamer | 비고 |
|---|---|---|---|---|
| 선명 | `unsharp`(`luma_amount` -1.5~1.5, 양수=선명, 음수=흐림, 0=끔) [확인 ffmpeg-filters.html 11.269] — **런타임 명령 섹션 없음**(재구성 필요) | `vf add lavfi=[unsharp=...]` 또는 셰이더 | `gdkpixbuf`/`videofilter` 계열, `gleffects`, 별도 플러그인(미확인) | GPL 아님 |
| 흐림 | `gblur`(명령 지원 [확인 11.111.1]) , `boxblur`(**GPL 전용**, 명령 섹션 확인 안 함) | `vf-command` 로 `gblur` 의 sigma 변경 가능(lavfi 한정 [확인 input.rst:1660]) | `gaussianblur`(gst-plugins-bad, 일반지식) | gblur 권장 |
| 노이즈 제거 | `hqdn3d`(명령 지원 [확인 11.122.1], **GPL 전용** [확인 configure:4185]), `nlmeans`(LGPL, 매우 느림, 명령 미확인) | 위와 동일 | `videobalance` 에는 없음, 별도(미확인) | 라이선스 때문에 `nlmeans` 또는 셰이더 |
| 디인터레이스 | `yadif`(명령 = 옵션과 동일 [확인 11.295.1 근처]), `bwdif`(명령 미확인) | `--deinterlace=<yes|no|auto>` [확인 options.rst:1832] , 속성 `deinterlace-active` | `deinterlace` 요소, `d3d11deinterlace`(일반지식) | yadif/bwdif 는 `*_filter_deps` 에 gpl 없음 [확인 configure] |
| 밝기·대비·채도 | `eq`(명령 지원 [확인 ffmpeg-filters 목록], **GPL 전용** [확인 configure:4169]) , 대체 `lut`/`colorbalance`/`colorlevels`(미확인) | 속성 `brightness`/`contrast`/`saturation` `-100..100` (예: `--brightness=<-100-100>` [확인 options.rst:4088]) | `videobalance`(brightness/contrast/saturation/hue, 일반지식) | mpv 가 가장 쉬움 |

- **재생 중 변경**: mpv — `vf toggle @라벨`/`vf-command`/속성 설정 [확인 input.rst:1650-1670]. FFmpeg — `avfilter_graph_send_command`(옵션 "T" 표시된 항목만 [확인 ffmpeg-filters 5절]), 또는 그래프 재구성(프레임 단절 없이 하려면 현재 PTS 에서 교체).
- **출력 셰이더 방식**: gpui 에서는 불가(스프라이트 파이프라인 고정, 사용자 셰이더 주입 API 미확인). mpv `glsl-shaders`/libplacebo 는 mpv 내부에서 처리(일반지식, 문서 미열람). D3D11 비디오 프로세서(`ID3D11VideoProcessor`)는 밝기·대비·채도·노이즈·선명 필터를 하드웨어로 제공(일반지식 — 세부 지원은 GPU 마다 다름): gpui 와 결합 시 결과를 CPU 로 읽어야 해서 이득이 줄어듦(추정).
- **성능·품질**: `unsharp`/`gblur`/`yadif` 는 1080p 에서 CPU 부담이 작고, `nlmeans` 는 실시간 부적합(일반지식). 필터 전에 HW 프레임을 CPU 로 내려야 하므로 R1+R2 동시 사용 시 복사 비용 증가.

---

## Q8 PotPlayer 대응표 (PotPlayer 메뉴 이름/위치는 **미확인** — 공식 문서를 못 찾음. 검색에서 확인된 것: D3D11 내장 렌더러에 "GPU Super Resolution" 추가(230405), 디코더 설정에서 DXVA/D3D11 선택, F5 환경설정 `https://www.neowin.net/software/potplayer-230405/`, `https://forum.doom9.org/archive/index.php/t-162442-p-12.html`)

| PotPlayer 기능(추정 명칭) | 구현 |
|---|---|
| 하드웨어 디코더(DXVA/D3D11/NVDEC/QSV) 선택 | FFmpeg `hw_device_ctx` 종류 선택 / mpv `hwdec=d3d11va|d3d11va-copy|auto|no` |
| 하드웨어 사용 안 함 | `hwdec=no` / `hw_device_ctx` 제거 |
| 선명/흐림 | `unsharp` / `gblur` 또는 mpv 셰이더 |
| 노이즈 제거 | `nlmeans`(LGPL) / `hqdn3d`(GPL) |
| 디인터레이스 | `yadif`/`bwdif` / mpv `--deinterlace` |
| 밝기·대비·채도·색조 | mpv 속성 / `eq`(GPL) |
| 스케일러/Super Resolution | mpv `scale` 옵션, libplacebo(미확인) |
| 속도 | `atempo`·scaletempo2 / mpv `speed` |
