---
tags: [rust, video-player, tauri, webview2, mpv, hwnd]
created: 2026-11-01
updated: 2026-11-01
---

# 04 · Tauri 앱에 붙이기

> 이 파일을 읽는 때: workbench-tauri 위에 동영상 탭을 만들 때. `<video>` 로 충분한지, 네이티브 표면을 어떻게 맞출지 정할 때.

입구는 [README.md](README.md)다. 세션·탭 정책은 [01-ARCHITECTURE.md](01-ARCHITECTURE.md), 엔진은 [02-DECODING.md](02-DECODING.md).

## 1. workbench-tauri 현황

조사자가 로컬 소스를 읽은 결과다 (읽기만 함).

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 프런트 | `src/App.tsx` 는 `<main>ZedLike</main>` 한 줄짜리 빈 껍데기. Solid(`solid-js` 1.9.18) | `D:\projects\workbench-tauri\src\App.tsx` |
| Rust | `src-tauri/src/lib.rs` 는 `tauri::Builder::default().run(...)` 만 있다 | `src-tauri\src\lib.rs` |
| 버전 | `tauri = "2.12.5"`, rust-version 1.90 | `src-tauri\Cargo.toml` |
| CSP | `default-src 'self' ipc: http://ipc.localhost`. `<video src>` 를 쓰려면 `media-src` 추가가 필요하다(추정) | sonnet 보고서 Q5. https://v2.tauri.app/reference/config/ |
| 결정 | D-3 프런트는 Solid. D-10 CSP 를 처음부터 켠다 | `D:\projects\workbench-tauri\specs\v0.1\DECISIONS.md` |

탭 store·컴포넌트는 아직 없다. 이미 있는 것처럼 인용하지 않는다 (sol). crates.io 의 tauri 최신 3.0.0-alpha.5 는 프리릴리스라 쓰지 않는다. https://crates.io/api/v1/crates/tauri

## 2. 탭 모델·세션 레지스트리·IPC

설계다. 구체적인 탭/트리 구현은 별도 작업이다.

| 위치 | 내용 |
| --- | --- |
| 프런트 | `Pane`/`Tab { id, path, kind }` 와 `VideoTab` 컴포넌트를 추가한다. D-3 의 이유(Solid 컴포넌트가 실제 DOM 노드를 반환해 pane 이동 시 뷰를 언마운트하지 않아도 된다)가 영상 컨테이너 보존에 도움이 된다 (argon 보고서) |
| Rust | `tauri::State` 에 **세션 레지스트리**(`SessionId → PlayerSession`). https://v2.tauri.app/develop/state-management/ |
| 명령 | `open / pause / seek / set_hwdec / set_filter / select_track / close / set_bounds` https://v2.tauri.app/develop/calling-rust/ |
| 이벤트 | 세션 ID·generation 이 붙은 상태·위치·트랙 이벤트 ([01-ARCHITECTURE.md](01-ARCHITECTURE.md) 2.2) |
| open | 확장자만 믿지 않고 probe 한다 |

설계 예시:

```rust
struct Sessions(Mutex<HashMap<SessionId, PlayerSession>>);
#[tauri::command]
fn set_bounds(id: SessionId, rect: PhysRect, visible: bool, s: State<Sessions>) { /* SetWindowPos */ }
// open/pause/seek/set_hwdec/set_filter/close 도 같은 모양
```

PCM·영상 프레임을 이벤트 JSON 으로 지속 전송하는 구조는 권하지 않는다 (5.3).

**유튜브 링크 (R3, 2026-11-01 추가)**: 프런트의 미리보기에서 링크 클릭을 가로채 `open` 명령에 URL 소스로 넘기고, 같은 `VideoTab` 을 연다. URL 검증은 Rust 쪽에서 `url` 크레이트로 한다. 웹뷰가 링크를 직접 열지 않게 막는다. 공식 YouTube IFrame Player API 를 웹뷰에 넣는 방법도 있지만 R1·R2 가 안 되므로 R3 대체안이 아니다(보조 "공식 플레이어로 열기" 정도). 상세는 [../rust-video-player-youtube.md](../rust-video-player-youtube.md).

## 3. `<video>` 의 한계

결론: `<video>` 단독으로는 범용 재생·R1·R2 를 보장하지 못한다. MP4/H.264/AAC 같은 제한된 입력의 **보조 경로**로만 쓸 수 있다. 근거: https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/develop/media-authoring-processing/supported-codecs.md , https://html.spec.whatwg.org/multipage/media.html

| 요구 | `<video>`(WebView2) | 출처 |
| --- | --- | --- |
| MKV/TS | 컨테이너·내부 코덱 조합에 따라 다르다. TS 는 별도 streaming/remux 경로가 필요하다 | sol. WebView2 버전별 전 조합은 **미확인** |
| HEVC | Windows 의 HEVC 확장·GPU·런타임에 조건부 | https://emby.media/community/topic/102877-question-about-browser-support-for-hevc 등 포럼 수준 자료만 확인. **WebView2 공식 코덱 표는 미확인** (sonnet) |
| AC3 | Windows 11 24H2 부터 OS 기본에서 빠졌다. MF 의 AC3 를 기본 보장하면 틀리다 | https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/develop/media-authoring-processing/supported-codecs.md , https://support.microsoft.com/en-us/windows/apps/windowsmediaplayer/codecs-faq |
| ASS 자막 | 표준 `<video>` 에 없다. WebVTT 모델로 ASS 를 재현하지 못한다 | https://html.spec.whatwg.org/multipage/media.html#text-tracks |
| R1 | 디코더를 지정해 재생 중 하드웨어 디코딩을 강제로 ON/OFF 하는 표준 속성이 없다. WebView2 시작 인자는 WebView 전체 범위다. WebCodecs `hardwareAcceleration` 은 힌트일 뿐이다 | https://learn.microsoft.com/en-us/microsoft-edge/webview2/reference/win32/icorewebview2environmentoptions , https://www.w3.org/TR/webcodecs/#hardware-acceleration |
| R2 | CSS filter 는 brightness/contrast/saturate/blur 만. 노이즈 제거·디인터레이스·일반 sharpen 이 없다. WebGL/WebGPU 셰이더는 가능하나 프레임 추출·업로드·색관리를 직접 관리 | https://www.w3.org/TR/filter-effects-1/#FilterProperty |

FFmpeg 를 설치한다고 WebView `<video>` 의 디코더가 그것으로 바뀌지는 않는다. 확장하려면 별도 네이티브 파이프라인이 필요하다 (sol 구조적 판단).

### 3.1 로컬 파일 제공 (`<video>` 를 쓸 때)

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 설정 | `app.security.assetProtocol.enable`(기본 false) + `scope` 글롭. `convertFileSrc` 로 URL 변환 | https://v2.tauri.app/reference/config/#assetprotocolconfig , https://v2.tauri.app/reference/javascript/api/namespacecore/#convertfilesrc |
| Range | **내장 asset 핸들러가 Range 를 처리한다**(206, `Accept-Ranges`, 416). 별도 루프백 HTTP 서버가 반드시 필요한 것은 아니다 | https://github.com/tauri-apps/tauri/blob/dev/crates/tauri/src/protocol/asset.rs (comparison.md 판정 3) |
| 근거 차이 | argon 은 Range 버그로 루프백 서버가 필요하다고 했으나 그 근거는 옛 이슈(https://github.com/tauri-apps/tauri/issues/4775)다. sonnet 은 Range 지원 **미확인**. 판정은 "처리함"이고, **실기 검증 항목**으로 남긴다 | comparison.md |
| 사용자 정의 프로토콜 | 직접 만들면 `Range` → 206/`Content-Range`/`Accept-Ranges`, 416, MIME, 길이를 구현하고 파일 전체를 메모리에 올리지 않는다 | https://v2.tauri.app/develop/calling-rust/ (sol 보고서) |
| CSP | `media-src` 허용 추가 (D-10 과 같이) | 위 1절 |

## 4. 자식 HWND 방식 (권장)

mpv 를 Rust 쪽에 두고, 탭 영상 영역에 맞춘 **전용 자식 HWND** 를 `wid` 로 준다. 최상위 창 HWND 전체를 주지 않는다. https://mpv.io/manual/stable/#window

### 4.1 좌표 동기화

| 단계 | 내용 |
| --- | --- |
| ① | 프런트에서 영상 컨테이너 `div` 에 `ResizeObserver` 를 단다 |
| ② | pane layout 이 바뀔 때 CSS rect × `devicePixelRatio` 로 물리 픽셀을 구한다 |
| ③ | `invoke("set_bounds", { id, x, y, w, h, visible })` |
| ④ | Rust 가 `SetWindowPos` 로 자식 HWND 를 옮기고, 숨김 탭은 `ShowWindow(SW_HIDE)` |

갱신 시점: 창 이동, DPI 변경, 분할, 사이드바 드래그, 스크롤, 최대화. 프런트 좌표를 OS 화면 좌표로 오해하지 않는다 (client 좌표). 출처: https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowpos (sol, argon 설계 제안).

### 4.2 겹침(airspace)

WebView 위에 네이티브 자식 창을 올리면 DOM 의 툴팁·메뉴·드롭다운·컨트롤이 영상에 가려진다. 1차 대응은 **컨트롤을 영상 밖에 두는 것**이다. https://github.com/mpv-player/mpv/blob/master/include/mpv/render.h (mpv 임베딩 경고, sol). 영상 위 DOM 오버레이가 꼭 필요하면 5.1 을 본다.

### 4.3 플러그인

둘 다 nini22P 가 만들었다 (comparison.md 판정 9).

| 플러그인 | 방식 | 버전 | 비고 |
| --- | --- | --- | --- |
| `tauri-plugin-libmpv` | libmpv 임베드. Tauri 창 핸들을 `wid` 로 넘기고 `libmpv-wrapper` 동적 라이브러리 사용 | 0.3.2 (2025-11-24), MPL-2.0, 별 23 | 임베딩에 **투명 창 설정이 필요**하다고 README 에 있다. 리사이즈·위치는 따로 맞춘다. `video-margin-ratio-*` 속성 제공 https://docs.rs/crate/tauri-plugin-libmpv/latest/source/src/desktop.rs |
| `tauri-plugin-mpv` | mpv **프로세스**를 띄워 JSON IPC 로 제어 | 0.5.2 (2025-11-18), 라이선스 **미확인** | https://docs.rs/crate/tauri-plugin-mpv/latest |

커뮤니티 우회: 자식 WebviewWindow 에 mpv 를 띄우고 부모 이동·리사이즈마다 HTML div 위치에 맞춘다. https://github.com/orgs/tauri-apps/discussions/6343 (sonnet, 검색 요약). 플러그인을 그대로 쓰지 말고 구조 참고용으로 보는 쪽이 안전하다. 우리 세션 모델([01-ARCHITECTURE.md](01-ARCHITECTURE.md))과 맞지 않을 수 있다 (추정).

## 5. 대안

### 5.1 투명 WebView + 뒤쪽 네이티브 표면

네이티브 표면을 WebView **뒤**에 두고 영상 영역만 WebView 를 투명하게 뚫는다. DOM 컨트롤이 영상 위에 보인다.
- 사례 Cathode: Windows 에서 libmpv render API + ANGLE EGL/D3D11 + 투명 WebView 구성. https://github.com/kaiserbh/cathode
- 비용: hit-testing, 배경, z-order, 입력 동기화가 추가된다. 자동 해결책이 아니다. https://github.com/mpv-player/mpv/blob/master/include/mpv/render.h
- Cathode 앱은 GPLv3 라서 코드를 복사하지 않는다 (구조만 참고).

### 5.2 WebView2 TextureStream (실험)

get-air `tauri-video-plugin` main: Windows 에서 GStreamer D3D11Memory → WebView2 TextureStream 으로 HTML video 에 GPU 텍스처 pool 을 표시한다. CPU readback 은 없고 GPU NV12 copy 가 1회 있다. 이 API 는 **experimental** 이다. https://github.com/get-air/tauri-video-plugin/blob/main/docs/windows.md , https://learn.microsoft.com/en-us/microsoft-edge/webview2/reference/win32/icorewebview2experimentaltexturestream?view=webview2-1.0.4071-prerelease
- 그대로 쓰지 않는다. README 는 0.4, crates.io 최신 `tauri-plugin-video` 는 0.3.0 으로 어긋난다. 재생 속도 변경·정확 탐색이 미지원이다. https://crates.io/crates/tauri-plugin-video
- 쓰려면 WebView2 Fixed Runtime 로 검증해야 한다.

### 5.3 프레임 IPC·WebCodecs

| 방식 | 판단 | 출처 |
| --- | --- | --- |
| Rust 디코딩 프레임을 IPC 로 캔버스에 | 4K60 BGRA 약 1.99 GB/s(계산). JSON/base64 는 더 크다. Tauri 는 raw binary Channel 을 지원하므로 "항상 JSON" 은 틀리지만 프로세스 복사·JS lifetime·canvas upload 가 남는다 | https://v2.tauri.app/reference/javascript/api/namespacecore/ , https://v2.tauri.app/concept/inter-process-communication/ |
| 1080p RGBA | 1프레임 약 8.3 MB, 60fps 약 0.5 GB/s | 산술값 (sonnet) |
| 인코딩 패킷을 보내 WebCodecs 로 | CPU RGBA IPC 는 줄지만 코덱 지원·demux·오디오 시계·탐색·ASS·R1 을 다시 풀어야 한다 | https://www.w3.org/TR/webcodecs/ |
| wgpu | 별도 네이티브 surface 출력에는 쓸 수 있다. WebView2 DOM 과 같은 합성기가 되지는 않는다 | https://docs.rs/wgpu/latest/wgpu/ (sol 평가) |

## 6. 사례

별·최근 커밋은 2026-11-01 GitHub API 값 (sol, sonnet).

| 사례 | 구성 | 별 | 최근 | 참고점·주의 |
| --- | --- | --- | --- | --- |
| Cathode https://github.com/kaiserbh/cathode | libmpv render API·ANGLE + Tauri 2·Dioxus | 4 | 2026-10-13 | 투명 WebView, DLL/ANGLE 번들, typed command. 앱 GPLv3 |
| frame-player https://github.com/risenxxx/frame-player | libmpv + Tauri 2·Svelte | 46 | 2026-10-29 | 배포·mpv state 모델·네이티브 컨트롤 경계. R1/R2 실측 **미확인** |
| Newish0/tauri-media-player https://github.com/Newish0/tauri-media-player | mpv FFI + Tauri·React | 2 | 2025-01-01 | Windows 전용 실험. DLL/FFI 구조 참고. 장기 무커밋 |
| tauri-video-plugin https://github.com/get-air/tauri-video-plugin | GStreamer + Tauri 2 | 2 | 2026-08-27 | 5.2. 실험 API |
| tauri-plugin-libmpv https://github.com/nini22P/tauri-plugin-libmpv | libmpv `wid` | 23 | 2025-11-24 | 4.3 |
| Kenichi Preview https://github.com/simplearyan/kenichi-preview | FFmpeg + wgpu + Tauri 2 | 2 | 2026-02-02 | FFmpeg 6.x 공유 라이브러리 필요. sonnet 이 검색 요약으로만 확인 |

## 7. 주의점

| 주의 | 내용 |
| --- | --- |
| 번들 | libmpv DLL 은 `bundle.resources`, 실행형은 `externalBin`. 빌드·크기는 [05-WINDOWS.md](05-WINDOWS.md) 4절 |
| 분할 이동 | Solid 가 뷰를 보존해도 자식 HWND 위치는 `set_bounds` 로 다시 맞춘다 |
| 상태 | 재생 상태는 Rust 세션에. 프런트는 이벤트로 표시만 |

---

이전: [03-GPUI.md](03-GPUI.md) · 다음: [05-WINDOWS.md](05-WINDOWS.md)
