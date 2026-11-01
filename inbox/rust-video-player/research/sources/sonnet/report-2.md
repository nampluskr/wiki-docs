# 조사 보고서 (sonnet) — Rust 동영상 플레이어 (gpui·Tauri, Windows 11) — 2/2

- 1/2(요약, Q1~Q8)는 `report-1.md`. 표기·출처 약칭은 report-1 과 같다(CIO=crates.io API, GH=GitHub API, 조회일 2026-11-01).
- 마크: **[확인]** 이번에 직접 확인, **[일반지식]** 기억 기반 미검증, **추정**, **미확인**.

---

## Q9 소리 출력

| 크레이트 | 버전 | 릴리스 | 라이선스 | 유지 |
|---|---|---|---|---|
| `cpal` | 0.18.2 | 2026-08-16 | Apache-2.0 | 활발(별 4002, push 2026-10-28) [확인 CIO, GH `RustAudio/cpal`] |
| `rodio` | 0.23.0 | 2026-10-11 | MIT OR Apache-2.0 | 활발(별 2502) [확인 CIO, GH `RustAudio/rodio`] |
| `symphonia` | 0.6.1 | 2026-08-13 | MPL-2.0 | 활발(별 3462) [확인] |

- **권장**: 디코딩이 FFmpeg/mpv/GStreamer 쪽이므로 오디오는 직접 제어가 필요 → **`cpal`(Windows 는 WASAPI 호스트, 일반지식)** 에 PCM 링버퍼를 물리는 방식. `rodio` 는 `Source` 추상화가 편하나 재생 위치(오디오 시계) 정밀 조회·속도 변경 시 버퍼 제어가 약해 A/V 동기 마스터 시계로는 `cpal` 이 낫다(추정).
- **오디오 시계**: 콜백이 소비한 샘플 수 ÷ 샘플레이트 + 시작 PTS − (출력 지연 추정). 콜백 지연(`OutputCallbackInfo`)은 Windows WASAPI 공유 모드에서 수 ms~수십 ms(추정).
- **장치 변경**: 장치 핫플러그(USB 헤드셋 등) 시 스트림 오류 콜백 → 스트림 재생성, 위치 유지(일반지식).
- **리샘플링**: 디코더 출력 포맷(예: 48 kHz float planar) → 장치 포맷으로 변환. FFmpeg `swresample` 이 가장 안전(속도 변경 시 `atempo` 포함). 순수 Rust 는 `rubato` (이번 조사에서 크레이트 정보 미조회 — **미확인**).
- mpv/GStreamer 를 쓰면 오디오 출력까지 그 안에서 처리되므로 이 절은 FFmpeg 직접 경로에서만 필요하다.

## Q10 자막

- mpv: `sub-add <url> [flags title lang]` 로 외부 자막 추가, `sid`/`aid` 로 트랙 선택 [확인 input.rst:687, options.rst `--aid/--sid=<ID|auto|no>`]. ASS 는 libass 내장 렌더(일반지식 — 이번 조사에서 mpv 문서의 libass 문장 미열람).
- FFmpeg: 필터 `subtitles`(libass+avformat 필요, [확인 configure:4262 `subtitles_filter_deps="avformat avcodec libass"`]) 와 `ass` 필터(`ass_filter_deps="libass"` [확인 configure:4129]). `ass` 필터는 명령 지원 섹션이 있음 [확인 ffmpeg-filters 목차]. 즉 **FFmpeg 빌드에 `--enable-libass` 필요**. 번인(burn-in)이라 자막 켜기/끄기는 그래프 교체 또는 `ass` 의 명령/`enable` 로 처리(추정).
- gpui 직접: libass 를 쓰지 않고 SRT 만 자체 파서(타임코드 파싱, 시계 비교 후 텍스트를 gpui 요소로 오버레이)하면 gpui 가 폰트/한글을 이미 처리 → 구현 쉬움(추정). ASS 스타일은 libass 필요.
- Rust libass 바인딩: `libass` 크레이트 마지막 업데이트 2020-11-21 [확인 CIO] → 사실상 정지. 대안은 FFmpeg `subtitles` 필터 또는 mpv.
- GStreamer: `playbin3` 의 `text-sink`/`suburi`, `subtitleoverlay`(일반지식, 미열람).
- 인코딩: 한글 SRT 는 CP949/UTF-8-BOM 이 흔함 → `chardetng`/`encoding_rs` 로 감지(일반지식, 미조회).

## Q11 탭 동작

원칙: **재생 상태는 뷰(위젯)가 아니라 `VideoSession`(Entity/상태 객체)에 둔다.** 뷰는 세션을 보여 줄 뿐. (workbench-gpui D-7 "상태는 gpui Entity, `cx.notify()`" 와 일치 [확인 DECISIONS.md D-7].)

| 상황 | 권장 동작 |
|---|---|
| 탭 전환(활성→비활성) | 기본: 일시정지(PotPlayer/브라우저 관례, 추정) + 마지막 프레임 유지. 옵션으로 "백그라운드 재생". 오디오 스트림은 일시정지 때 닫지 않고 ring buffer 만 멈춤. |
| 숨김 탭(렌더되지 않음) | 비디오 렌더 큐/디코딩 스레드 정지. mpv `--wid` 자식 창은 `ShowWindow(SW_HIDE)`. 디코더 컨텍스트는 유지하고 N분 후 해제(메모리 정책). 재활성 시 현재 PTS 에서 다시 탐색. |
| 분할(split) | 같은 파일의 두 번째 뷰는 **같은 세션 공유** 가능 → 프레임 큐 팬아웃. 다른 파일이면 독립 세션. 자식 HWND 방식은 분할마다 창 하나를 만들어 위치 동기화. |
| 여러 탭 동시 재생 | 기본값: **한 번에 하나만 소리 출력**(활성 탭). 나머지는 일시정지 또는 음소거 재생. 동시 하드웨어 디코더 세션 수는 GPU 마다 제한이 있음(일반지식, 정확한 수 **미확인**) → 초과 시 소프트로 자동 전환. |
| 닫기 | 세션 `Closed` 상태로 큐 비우고 디코더·오디오 스트림·GPU 자원(`drop_image`, [확인 GPUI-SRC window.rs:5193])·자식 창 해제. |
| 전체 화면 | 별도 창(자식 HWND) 또는 현재 창 전체화면 — mpv `--wid` 에서는 부모 창 전환 필요(추정). |
| 창 최소화 | 일시정지 또는 렌더만 중단(옵션). |

## Q12 빌드·배포·라이선스

- **FFmpeg DLL**: 공식은 소스만 제공(`https://ffmpeg.org/download.html`, 일반지식 — 이번엔 페이지 미열람); Windows 빌드는 BtbN, gyan.dev 등 제3자(미열람, 일반지식). Rust 빌드: `ffmpeg-sys-next`/`rusty_ffmpeg` 가 `FFMPEG_DIR`/`vcpkg`/`pkg-config` 로 헤더·lib 를 찾음(일반지식, 정확한 환경변수는 각 README 확인 필요 **미확인**). 라이선스 선택 시 `--enable-gpl` 여부가 핵심.
- **FFmpeg LGPL vs GPL [확인 FFmpeg configure @master 2026-11-01 `https://raw.githubusercontent.com/FFmpeg/FFmpeg/master/configure`]**: 다음 필터는 `--enable-gpl` 필요: `boxblur`, `eq`, `hqdn3d`, `histeq`, `nnedi`, `owdenoise`, `vaguedenoiser`, `kerndeint`, `pullup`, `colormatrix` 등. 다음은 gpl 표시 없음: `yadif`, `bwdif`, `unsharp`, `gblur`, `nlmeans`(이번 grep 에서 `*_filter_deps` 목록에 나타나지 않음 = gpl 의존 없음). 주의: configure 에 의존 표기가 없다고 해서 소스 라이선스가 LGPL 이라는 보장은 아님(소스 헤더 확인은 이번에 실패, **미확인**).
- **libmpv**: 소스 코드는 GPLv2+ 기본이며 `-Dgpl=false` 로 LGPL 빌드 가능(일반지식, 이번 조사 미열람 **미확인**). 배포 DLL(shinchiro/zhongfly 등 서드파티 빌드)의 라이선스는 빌드마다 다름(일반지식). 바인딩 `libmpv2` 자체는 LGPL-2.1 [확인 CIO]. 같은 앱에 GPL libmpv 를 동적 링크로 묶으면 앱 전체 배포 조건 검토 필요(추정, 법률 검토 필요).
- **GStreamer**: 크레이트는 MIT/Apache, 런타임은 LGPL 코어 + 플러그인별 (gst-plugins-bad/ugly 포함 GPL 혼재, 일반지식). Windows MSVC 런타임/개발 설치 프로그램 사용, 설치 용량 큼(수백 MB, **추정**).
- **특허 코덱**: H.264/HEVC 등은 사용 허가 문제(MPEG-LA/Access Advance 등). OS 내장(Media Foundation)이나 사용자 시스템 설치 코덱을 쓰면 앱이 코덱을 재배포하지 않음. 오픈소스 openh264 는 Cisco 바이너리 사용 시 특허료 면제(일반지식, 이번 조사 미열람). 법률 자문은 이 문서 범위 밖.
- **gpui 쪽**: workbench-gpui 는 `gpui-pre`(Apache-2.0), `gpui-kit`(Apache-2.0)만 의존해 D-2 규칙(GPL 금지)을 지킨다 [확인 DECISIONS.md D-2, CIO]. 영상 의존이 GPL 이면 이 규칙과 충돌하므로 **LGPL FFmpeg(필터 제한) 또는 mpv 별 프로세스**가 규칙에 맞음(추정, 설계 판단은 leader).
- **묶는 법**: `exe` 옆에 DLL 폴더(`lib/`), Tauri 는 `bundle.resources` 에 DLL 포함(tauri-plugin-libmpv README: "번들 설정 필요" [확인 검색 요약]). 크기 예상: FFmpeg 공유 빌드 수십~백 MB, 슬림 빌드 가능(**추정**).

## Q13 크레이트 목록 (CIO·GH 조회 2026-11-01)

| 이름 | 최신 | 릴리스 | 라이선스 | 유지 |
|---|---|---|---|---|
| ffmpeg-next | 9.0.0 | 2026-08-05 | WTFPL | 활발 (별 2021) |
| ffmpeg-the-third | 6.0.0+ffmpeg-9.0 | 2026-08-09 | WTFPL | 활발 (별 162) |
| ffmpeg-sys-next | 9.0.1 | 2026-10-09 | WTFPL | 활발 |
| rsmpeg | 0.18.0+ffmpeg.8.0 | 2025-08-24 | MIT | 정체(1년+) |
| rusty_ffmpeg | 0.17.0+ffmpeg.8.1 | 2026-04-10 | MIT | 보통 |
| gstreamer / -app / -video | 0.25.4 / 0.25.2 / 0.25.4 | 2026-09-21 / 05-11 / 09-21 | MIT OR Apache-2.0 | 매우 활발 |
| libmpv2 / libmpv2-sys | 7.0.0 / 4.0.1 | 2026-10-30 / 2025-12-29 | LGPL-2.1 | 활발, API 변동 큼 |
| libmpv, libmpv-sys (구) | — | 2020 | — | 정지 |
| symphonia | 0.6.1 | 2026-08-13 | MPL-2.0 | 활발 |
| rav1d | 1.1.0 | 2025-05-07 | BSD-2-Clause | 저빈도(저장소 push 2026-10-27) |
| dav1d | 0.11.1 | 2025-11-25 | MIT | 보통 |
| openh264 / -sys2 | 0.9.9 / 0.9.9 | 2026-10-15 | BSD-2-Clause | 활발(별 130) |
| cpal | 0.18.2 | 2026-08-16 | Apache-2.0 | 활발 |
| rodio | 0.23.0 | 2026-10-11 | MIT OR Apache-2.0 | 활발 |
| windows | 0.62.2 | 2025-10-06 | MIT OR Apache-2.0 | 보통 |
| wgpu | 30.0.1 | 2026-08-22 | MIT OR Apache-2.0 | 활발 |
| gpui-kit / gpui-component | 0.8.1 | 2026-10-26 | Apache-2.0 | 활발 (GH 별 약 1.9만) |
| tauri (2.x 안정) | 2.12.5 사용 중 [확인 workbench-tauri/Cargo.toml], crates.io 최신은 3.0.0-alpha.5(2026-10-14) | — | Apache-2.0 OR MIT | 매우 활발 |
| tauri-plugin-libmpv | 0.3.2 | 2025-11-24 | MPL-2.0 | 느림(별 23) |
| tauri-plugin-mpv | 0.5.2 | 2025-11-18 | 미확인 | 느림 |
| libass (Rust) | — | 2020-11-21 | — | 정지 |

## Q14 기존 사례 (8개 이상)

| # | 이름 | 백엔드·GUI | 별 | 최근 커밋/릴리스 | 참고점 |
|---|---|---|---|---|---|
| 1 | gpui-video-player `https://github.com/cijiugechu/gpui-video-player` | GStreamer + gpui | 97 | push 2026-10-30, 0.1.3 (2026-10-23) | **gpui 선례**. Windows 는 아틀라스 소프트 경로 [확인] |
| 2 | iced_video_player `https://github.com/jazzfool/iced_video_player` | GStreamer + iced | 207 | push 2026-04-05, 0.6.0 (2025-12-14) | gpui-video-player 가 영감 받은 원형(검색 요약) |
| 3 | egui-video `https://github.com/n00kii/egui-video` | FFmpeg + egui | 150 | push 2024-10-28 (정체) | 프레임 per-`ui` 호출 방식 |
| 4 | egui-sharkplayer (crates.io) | libmpv + egui | 미조회 | 0.5.1 (2026-07-05) | 하드웨어 가속 libmpv 임베드, 0.3.0 yanked [확인 검색 결과] |
| 5 | mpv-engine `https://docs.rs/crate/mpv-engine/0.1.0` | libmpv + iced/GTK 어댑터 | 미조회 | 0.2.1 (2026-09-07) | 기본 빌드 GPLv2+ 주의 [확인 검색 결과] |
| 6 | video-rs `https://github.com/oddity-ai/video-rs` | FFmpeg(`ffmpeg-next`) 고수준 | 428 | push 2026-09-09, 0.12.0 | 디코딩/인코딩 래퍼 API 참고 |
| 7 | tauri-plugin-libmpv `https://github.com/nini22P/tauri-plugin-libmpv` | libmpv(`--wid`) + Tauri | 23 | push 2025-11-24 | Tauri `wid` 임베드 [확인] |
| 8 | tauri-plugin-mpv | mpv 프로세스 JSON IPC + Tauri | 미조회 | 0.5.2 (2025-11-18) | 프로세스 분리 방식 |
| 9 | Kenichi Preview `https://github.com/simplearyan/kenichi-preview` | FFmpeg + wgpu + Tauri v2 | 2 | push 2026-02-02 | 네이티브 레이어 재생 시도 |
| 10 | mpv 본체 `https://github.com/mpv-player/mpv` | C | 37648 | push 2026-10-31 | 동기·필터·옵션 참고(코드 복사 금지: GPL/LGPL) |

## Q15 Windows 최대 위험 3가지 + 자주 하는 실수

1. **gpui 에 네이티브 비디오 표면이 없음**: `paint_surface` 는 macOS 전용, Windows 는 `RenderImage` 업로드뿐 [확인]. 4K·HDR·고프레임에서 CPU 복사 병목 → 이 한계가 설계 전체를 정함. 자식 HWND 로 우회하면 오버레이/Z-순서 문제.
2. **라이선스·특허 혼선**: GPL 전용 FFmpeg 필터(`eq`, `hqdn3d`, `boxblur` 등 [확인]) 를 쓰면 GPL 빌드가 필요해지고, workbench-gpui 의 "GPL 비의존" 원칙(D-2)과 충돌할 수 있음. libmpv 배포 빌드의 GPL 여부도 동일. HEVC 특허.
3. **하드웨어 디코딩 환경 편차**: D3D11VA/DXVA2 선택에 따라 일부 파일/드라이버에서 실패·깨짐. 재생 중 전환 시 디코더 재생성·큐 정리·시간 위치 복원 실패가 흔하다(추정). 자동 소프트 전환을 반드시 구현.
- **자주 틀리는 점**: (a) 탐색 후 큐/오디오 버퍼를 안 비워 소리가 어긋남 (b) 오디오 콜백 안에서 잠금/할당 (c) `RenderImage` 를 프레임마다 만들고 `drop_image` 를 빼먹어 GPU 아틀라스 메모리 증가(추정) (d) 탭 숨김에도 디코딩 지속 (e) 웹뷰 `<video>` 로 MKV/AC3/HEVC 가 다 된다고 가정 (f) 버전 미고정으로 `gpui-pre`/`libmpv2` 업그레이드 파손(`libmpv2` 가 6.0.1→7.0.0 이 1일 간격 [확인 CIO]).

## Q16 권장 문서 목차

1. 목표·범위·PotPlayer 기능 기준 / R1·R2 정의
2. 결정표: gpui·Tauri 별 권장 조합(Q1)과 선택 이유
3. 재생 파이프라인 일반(Q2): 상태 기계, A/V 동기, 탐색
4. 백엔드 비교(Q3) — 선택지와 제외 이유
5. gpui 통합: 탭(Item) 추상, 프레임 경로, 한계(Q4)
6. Tauri 통합: 파일 제공, 코덱 한계, 네이티브 창(Q5)
7. R1 하드웨어 가속 ON/OFF·자동 전환(Q6)
8. R2 영상 옵션 + 라이선스 필터 표(Q7)
9. 소리(Q9)·자막(Q10)
10. 탭 동작 규칙(Q11)
11. 빌드·배포·라이선스(Q12)
12. 크레이트 표(Q13)·사례(Q14)
13. 위험·실수(Q15), 부록: PotPlayer 대응표(Q8)

## Q17 확신 낮은 부분

- **WebView2 의 정확한 코덱 지원표**: 공식 자료 미확인, 포럼 글만 확인.
- **gpui 가 창 HWND 를 어떻게 노출하는지**(`raw-window-handle`) 및 자식 창 생성 가능성 — 미확인. 소스에 `gpui_windows.rs`/`window.rs` 가 있으나 이번엔 읽지 않음.
- **gpui 에서 프레임마다 `RenderImage` 교체 시 아틀라스 해제·성능**: `drop_image` 존재만 확인, 실측 없음. 4K60/1080p60 한계 수치는 계산 추정.
- **Tauri asset 프로토콜 Range 지원**, `media-src` CSP 필요 여부: 미확인.
- **PotPlayer 메뉴 정확한 항목명·위치**: 미확인(공식 문서 못 찾음).
- **mpv 문서 중 `hwdec` 런타임 변경 세부(`hwdec` 값 바꿀 때 디코더 재초기화 동작), libass 문장, `seek` 플래그**: 직접 열지 못함. 일반지식 표시.
- **FFmpeg 필터의 소스 라이선스 헤더 확인 실패**: `configure` 의 `gpl` 의존 표기만 근거. `nlmeans`/`bwdif` 의 명령 지원은 미확인.
- **GStreamer 필터 요소(`videobalance`, `gaussianblur`, `d3d11deinterlace`) 및 재생 중 속성 변경**: 일반지식, 이번 조사에서 문서 미열람.
- **rodio/cpal 와 `rubato` 세부, ffmpeg 빌드 환경변수, libmpv GPL/LGPL 빌드 옵션**: 미확인.
- 다른 scouter 와 달리 이 보고서는 공식 문서 대신 로컬에 이미 있는 gpui-pre 0.3.10 소스를 1차 근거로 썼다 — **원 저장소 최신(master)에서는 `paint_surface` 가 Windows 에 추가됐을 수 있음(미확인)**.
