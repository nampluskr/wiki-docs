---
tags: [rust, video-player, libmpv, ffmpeg, gstreamer, crates]
created: 2026-11-01
updated: 2026-11-01
---

# 02 · 디코딩 엔진 선택

> 이 파일을 읽는 때: libmpv·FFmpeg·GStreamer 중 무엇으로 재생할지 정하거나, 쓸 크레이트의 버전·유지 상태를 확인할 때.

입구는 [README.md](README.md)다. 공통 구조는 [01-ARCHITECTURE.md](01-ARCHITECTURE.md).

## 1. 백엔드 비교

| 백엔드 | 대표 크레이트 | Windows(MSVC) 연결 | 장점 | 비용·한계 |
| --- | --- | --- | --- | --- |
| **libmpv** | `libmpv2` 7.0.0 | libmpv DLL + MSVC용 import library(`.lib`) 필요. MinGW `.dll.a` 와 같은 것으로 취급하지 않는다 | 요구 기능(R1·R2·자막·트랙·탐색)을 가장 적은 구현량으로 충족 | 합성(HWND 또는 render API)과 라이선스가 핵심 비용. 바인딩이 소규모이고 API 변동이 크다 |
| FFmpeg 직접 | `ffmpeg-next` 9.0.0, `ffmpeg-the-third`, `rsmpeg` | FFmpeg DLL·헤더·import lib 와 바인딩 버전을 한 세트로 고정 | 최대 제어. gpui 안에서 합성 가능 | A/V 동기·탐색·자막·오디오 출력·HW 프레임 interop 를 모두 직접 만든다 |
| GStreamer | `gstreamer` 0.25.4 | 공식 MSVC runtime/devel 설치, plugin DLL 번들 | `playbin3` 가 포맷·트랙·자막·D3D11 요소를 자동 처리 | 런타임이 크고 plugin 조합·배포가 복잡하다 |
| Media Foundation | `windows` 0.62.2 | OS 내장 API | 추가 DLL 없음, HW 가속 | MKV·ASS·AC3 등 범용 한계. Windows 11 24H2 부터 AC3 가 OS 기본에서 빠졌다 |
| 순수 Rust | `symphonia`(오디오), `rav1d`(AV1), `openh264`(H.264) | `cargo build` 만 | 의존성이 단순하다 | HEVC·VP9·MKV 영상·ASS·필터가 없어 **범용 재생 불가** |

출처: 크레이트 표는 4절. 연결 방식 https://github.com/zmwangx/rust-ffmpeg/wiki/Notes-on-building , https://github.com/larksuite/rsmpeg/blob/master/doc/windows.md , https://gstreamer.freedesktop.org/download/ , https://github.com/kohsine/libmpv-rs . Media Foundation 한계와 AC3: https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/develop/media-authoring-processing/supported-codecs.md , https://support.microsoft.com/en-us/windows/apps/windowsmediaplayer/codecs-faq . 순수 Rust: https://github.com/pdeljanov/Symphonia , https://github.com/memorysafety/rav1d .

## 2. libmpv 를 권하는 이유

| # | 이유 | 출처 |
| --- | --- | --- |
| 1 | 디코딩·탐색·속도·다중 트랙·자막·A/V 동기를 이미 갖췄다. 직접 FFmpeg 플레이어를 만드는 것보다 구현량이 적다(평가) | https://mpv.io/manual/stable/ |
| 2 | R1: `hwdec` 가 런타임 속성이다. 재생 중 `auto-safe` ↔ `no` 로 바꾼다. 불가능하면 소프트웨어로 대체한다고 문서에 있다. 실제 상태는 `hwdec-current` | https://mpv.io/manual/stable/#video , https://mpv.io/manual/stable/#property-list |
| 3 | R2: 밝기·대비·채도 속성, `vf`·`vf-command`, 셰이더, `deinterlace` | https://mpv.io/manual/stable/#video-filters |
| 4 | 자막: libass 경로로 SRT·ASS·내장 트랙 처리 | https://mpv.io/manual/stable/#subtitles |
| 5 | 오디오: 자체 WASAPI 출력. 장치 목록 `audio-device-list` | https://mpv.io/manual/stable/#audio-output-drivers |
| 6 | gpui·Tauri 두 UI 에서 같은 엔진·같은 세션 코드를 쓴다 | [01-ARCHITECTURE.md](01-ARCHITECTURE.md) 2절 |

한계도 있다.
- gpui 에는 외부 D3D11 텍스처를 받는 공개 경로가 없어 처음에는 **자식 HWND(`wid`)** 로 붙인다. ([03-GPUI.md](03-GPUI.md) 3절)
- mpv 의 D3D11 render API 는 **master 에만 있다.** 커밋 2026-10-15, 최신 정식 v0.41.0(2025-12-21)에는 없다. `libmpv2` 도 지원하지 않는다. https://github.com/mpv-player/mpv/commit/4255130de410e0cbdab694b951085409d466b704 , https://github.com/mpv-player/mpv/releases/tag/v0.41.0 (comparison.md 판정 2)
- 배포 DLL 의 코덱 범위는 포함된 FFmpeg 빌드에 달렸다. 라이선스는 [05-WINDOWS.md](05-WINDOWS.md) 5절.

## 3. 대안은 언제

| 대안 | 고르는 때 | 주의 |
| --- | --- | --- |
| FFmpeg 직접 + `RenderImage` | gpui 안에서 합성해야 하고(분할·오버레이) 영상이 1080p 급일 때 | 4K60 은 CPU 복사 한계 ([03-GPUI.md](03-GPUI.md) 3.1). 오디오는 `cpal` + 링버퍼 ([05-WINDOWS.md](05-WINDOWS.md) 2절) |
| GStreamer `playbin3` + `appsink` | 선례(`gpui-video-player`)를 따라 빨리 PoC 를 만들 때 | Windows 런타임 설치 부담. GPU 디코딩이어도 `appsink` 에서 CPU 로 내려오면 무복사가 아니다. https://github.com/cijiugechu/gpui-video-player#platform-notes |
| Media Foundation | 추가 DLL 없이 H.264 중심의 제한된 입력만 다룰 때 | 범용 요구에는 다른 엔진이 더 필요하다 |
| 별 프로세스 mpv | 라이선스 분리나 크래시 격리가 필요할 때 | `tauri-plugin-mpv` 0.5.2 가 프로세스 JSON IPC 방식이다. https://docs.rs/crate/tauri-plugin-mpv/latest |

## 4. libmpv 연결 요약

### 4.1 `libmpv2` 현황

| 항목 | 값 | 출처 |
| --- | --- | --- |
| 최신 | **7.0.0** (2026-10-30), LGPL-2.1, 저장소 `kohsine/libmpv-rs` 별 66 | https://crates.io/api/v1/crates/libmpv2 (comparison.md 판정 1. argon 보고서의 6.0.0 은 틀렸다) |
| sys | `libmpv2-sys` 4.0.1 (2025-12-29) | https://crates.io/api/v1/crates/libmpv2-sys |
| 구 바인딩 | `libmpv` 2.0.1 (2020-09-29). 새 프로젝트에서 쓰지 않는다 | https://crates.io/api/v1/crates/libmpv |
| 주의 | 6.0.1 → 7.0.0 이 1일 간격이었다. API 가 불안정하다(추정). **버전을 `=` 로 고정**한다 | sonnet 보고서. https://crates.io/api/v1/crates/libmpv2 |
| render | 저장소의 render enum 은 OpenGL 만 노출한다. D3D11 은 미지원 | https://github.com/kohsine/libmpv-rs/blob/master/src/mpv/render.rs |

### 4.2 최소 흐름

설계 예시 (의사코드, 실제 API 시그니처는 `libmpv2` 문서로 확인):

```rust
let mpv = Mpv::new()?;                         // 코어 생성
mpv.set_property("wid", child_hwnd)?;          // 전용 자식 HWND (03, 04 참조)
mpv.set_property("hwdec", "auto-safe")?;       // R1 ON
mpv.command("loadfile", &[path])?;             // 재생 시작
// 이벤트 펌프 스레드에서 pause/duration/hwdec-current 를 관찰
```

흐름 출처: sonnet 보고서 Q3 (`Mpv::new → set_property("wid", hwnd) → command("loadfile", path) → set_property("hwdec","auto-safe")`). `wid` 계약 https://mpv.io/manual/stable/#window .

## 5. 크레이트 표

조회일 2026-11-01. 버전·날짜는 crates.io API(`https://crates.io/api/v1/crates/<이름>`), 별·push 는 GitHub API 기준이다. "유지" 칸은 comparison.md 판정을 반영한 평가다.

### 5.1 재생 엔진·바인딩

| 이름 | 최신 | 릴리스 | 라이선스 | 유지 |
| --- | --- | --- | --- | --- |
| libmpv2 | 7.0.0 | 2026-10-30 | LGPL-2.1 | 활발, API 변동 큼 |
| libmpv2-sys | 4.0.1 | 2025-12-29 | LGPL-2.1 | master 새 API 와 헤더 일치는 따로 검증 |
| ffmpeg-next | 9.0.0 | 2026-08-05 | WTFPL | **유지보수 모드**(README 명시). README 지원 범위(3.4~8.0)와 9.0.0 이 어긋나 FFmpeg 9 조합은 **미검증** https://github.com/zmwangx/rust-ffmpeg |
| ffmpeg-the-third | 6.0.0+ffmpeg-9.0 | 2026-08-09 | WTFPL | 활발, 별 162 |
| ffmpeg-sys-next | 9.0.1 | 2026-10-09 | WTFPL | 활발 |
| rsmpeg | 0.18.0+ffmpeg.8.0 | 2025-08-24 | MIT | 최근 릴리스 1년 이상 전 (comparison.md 판정 8) |
| rusty_ffmpeg | 0.17.0+ffmpeg.8.1 | 2026-04-10 | MIT | 보통 |
| gstreamer / -app / -video | 0.25.4 / 0.25.2 / 0.25.4 | 2026-09-21 / 05-11 / 09-21 | MIT OR Apache-2.0 | 매우 활발 |
| windows | 0.62.2 | 2025-10-06 | MIT OR Apache-2.0 | Microsoft 공식 |
| wgpu | 30.0.1 | 2026-08-22 | MIT OR Apache-2.0 | 활발 |

### 5.2 오디오·자막·순수 Rust

| 이름 | 최신 | 릴리스 | 라이선스 | 유지·비고 |
| --- | --- | --- | --- | --- |
| cpal | 0.18.2 | 2026-08-16 | Apache-2.0 | 활발. 직접 PCM 엔진용 |
| rodio | 0.23.0 | 2026-10-11 | MIT OR Apache-2.0 | 활발. PoC 용 |
| rubato | 5.0.1 | (argon 보고) 2026-10 | MIT OR Apache-2.0 | **argon 보고서 단독 값**. sonnet 은 미조회. 사용 전 확인 |
| symphonia | 0.6.1 | 2026-08-13 | MPL-2.0 | 활발. 오디오 전용 |
| rav1d | 1.1.0 | 2025-05-07 | BSD-2-Clause | AV1 만. 릴리스 저빈도 |
| dav1d | 0.11.1 | 2025-11-25 | MIT | 보통 |
| openh264 | 0.9.9 | 2026-10-15 | BSD-2-Clause | H.264 만 |
| libass / libass-sys | 0.2.0 / 0.1.2 | 2020-11-21 | ISC | **정지**. mpv 내장 libass 를 쓴다 |
| ass-renderer | 0.1.2 | (argon 보고) 2026 | MIT OR Apache-2.0 | argon 단독, 순수 Rust ASS. 대안으로만. 품질 **미검증** |
| srtlib | 0.2.0 | (argon 보고) | MIT OR Apache-2.0 | argon 단독. SRT 파서 |

### 5.3 UI·사례 쪽 크레이트

| 이름 | 최신 | 릴리스 | 라이선스 | 비고 |
| --- | --- | --- | --- | --- |
| gpui-kit | 0.8.1 | 2026-10-26 | Apache-2.0 | workbench-gpui 가 `=0.8.1` 고정 |
| gpui-pre | 0.3.10 | 2026-10-26 | Apache-2.0 | 로컬 lock 일치. 공식 `gpui` 0.2.2(2025-10-22)와 타입을 섞지 않는다 |
| tauri | 2.12.5 | 2026-10-21 | Apache-2.0 OR MIT | 로컬 manifest 일치. crates.io 최신은 3.0.0-alpha.5(2026-10-14, 프리릴리스) |
| gpui-video-player | 0.1.3 | 2026-10-23 | MIT | [03-GPUI.md](03-GPUI.md) 5절 |
| alhazen-core / alhazen-gpui | 0.6.0 | 2026-10-10 | Apache-2.0 | 신생. **검증 필요** ([03-GPUI.md](03-GPUI.md) 5절) |
| tauri-plugin-libmpv | 0.3.2 | 2025-11-24 | MPL-2.0 | [04-TAURI.md](04-TAURI.md) 4절 |
| tauri-plugin-mpv | 0.5.2 | 2025-11-18 | 라이선스 **미확인** | 같음 |
| tauri-plugin-video | 0.3.0 | 2026-08-20 | MIT OR Apache-2.0 | 실험 API. [04-TAURI.md](04-TAURI.md) 5절 |
| iced_video_player | 0.6.0 | 2025-12-14 | MIT OR Apache-2.0 | 비교용 |

출처: sonnet report-2 Q13, sol Q13 (모두 crates.io API). 유지 판정 https://github.com/kohsine/libmpv-rs , comparison.md 2절.

## 6. UI 중립 참고 사례

별은 2026-11-01 GitHub API 값이다. 코드 복사는 하지 않고 구조만 참고한다.

| 사례 | 백엔드·GUI | 별 | 참고점 |
| --- | --- | --- | --- |
| mpv 본체 https://github.com/mpv-player/mpv | C | 37,648 | 동기·필터·옵션의 기준 동작. **GPL/LGPL 이므로 코드 복사 금지** |
| libmpv2-rs https://github.com/kohsine/libmpv-rs | libmpv, SDL OpenGL 예제 | 66 | 이벤트 펌프, render callback, OpenGL FBO |
| iced_video_player https://github.com/jazzfool/iced_video_player | GStreamer appsink, Iced·wgpu | 207 | 큐·video widget 구조. gpui-video-player 가 참고했다고 명시 |
| egui-video https://github.com/n00kii/egui-video | FFmpeg, egui | 150 | **정체**(crate·push 모두 2024-10-28, comparison.md 판정 6) |
| video-rs https://github.com/oddity-ai/video-rs | `ffmpeg-next` 고수준 | 428 | 디코딩·인코딩 래퍼 API 참고 |
| gyroflow https://github.com/gyroflow/gyroflow | FFmpeg D3D11 → wgpu | 9,660 | 무복사 interop 사례. 이 가이드(재생)보다 후속 영상 처리 문서에 더 맞는다 |
| egui-sharkplayer (crates.io) | libmpv, egui | 미조회 | 0.5.1 (2026-07-05). 검색 결과 요약만 확인 |
| mpv-engine https://docs.rs/crate/mpv-engine/0.1.0 | libmpv, iced/GTK 어댑터 | 미조회 | 0.2.1 (2026-09-07). 기본 빌드 GPLv2+ |

출처: sonnet report-2 Q14, sol Q14, comparison.md 3절.

---

이전: [01-ARCHITECTURE.md](01-ARCHITECTURE.md) · 다음: [03-GPUI.md](03-GPUI.md)
