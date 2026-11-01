---
tags: [rust, video-player, windows, egui, libmpv, guide]
created: 2026-11-01
updated: 2026-11-01
---

# Rust 동영상 플레이어 원점 설계 가이드 — 파일 탐색기·탭 포함 (Windows 11)

> 이 파일을 읽는 때: 처음, 그리고 무엇을 읽을지 모를 때. 이 문서 묶음의 입구다.

Windows 11 에서 Rust 로 동영상 플레이어를 만드는 설계를 처음부터 다시 정리한다. 기존 앱 구조는 전제하지 않는다. 화면은 **왼쪽 파일 탐색기 트리 + 오른쪽 탭**이고, 동영상은 탭 안에서 재생한다. 기능 기준은 PotPlayer 이고, 필수 요구는 R1(GPU 가속 ON/OFF·자동 전환)과 R2(영상 옵션 실시간 조절)다.

| 항목 | 내용 |
| --- | --- |
| 대상 | Rust 로 Windows 동영상 플레이어를 설계·구현할 사람 |
| 요구 | R1 GPU 가속 ON/OFF·자동 전환, R2 영상 옵션 실시간 조절 ([00-requirements.md](00-requirements.md)). R3 유튜브 링크를 클릭하면 새 탭의 같은 플레이어 화면에서 재생하고, R1·R2 가 그대로 적용된다 — 2026-11-01 추가, 공통 문서 작성 중: [../rust-video-player-youtube.md](../rust-video-player-youtube.md) |
| 결과 | 스택 결정, 코어·UI 구조, R1/R2 구현 방법, PotPlayer 기능 대응, 배포·라이선스, PoC 계획 |
| 기준 | Windows 11, Rust stable-msvc, 2026-11-01 에 확인한 공식 문서·소스·crates.io |
| 검증 | 빌드·실행은 하지 않았다. 사실마다 출처 URL 을 달았고, 추정은 "추정"·"설계 제안"으로, 확인 못 한 것은 "미확인"으로 표시했다 |
| 근거 | scouter 3명의 조사 보고서와 그 비교 ([research/reviews/comparison.md](research/reviews/comparison.md)) |

## 1. 읽기 지도

| 지금 할 일 | 읽을 파일 |
| --- | --- |
| 무엇을 만들지, "다 됐다"의 기준 | [00-requirements.md](00-requirements.md) |
| 어떤 스택으로 갈지 | [01-stack-decision.md](01-stack-decision.md) |
| libmpv·FFmpeg·GStreamer 중 무엇을 쓸지 | [02-decoding-engine.md](02-decoding-engine.md) |
| 탭 안에 영상을 그리는 코드를 짠다 | [03-rendering-pipeline.md](03-rendering-pipeline.md) |
| 크레이트·스레드·채널을 나눈다 | [04-player-core.md](04-player-core.md) |
| 트리·탭·숨김 탭 정책을 만든다 | [05-explorer-tabs.md](05-explorer-tabs.md) |
| R1 (하드웨어 디코딩 전환) | [06-r1-hwdec.md](06-r1-hwdec.md) |
| R2 (선명·흐림·노이즈·디인터레이스·색) | [07-r2-video-options.md](07-r2-video-options.md) |
| PotPlayer 기능을 mpv 로 옮긴다 | [08-playback-features.md](08-playback-features.md) |
| DLL·라이선스 | [09-distribution-licensing.md](09-distribution-licensing.md) |
| PoC 계획, 아직 모르는 것 | [10-validation-and-open-questions.md](10-validation-and-open-questions.md) |
| 방식 비교, 최종 제안, 위험, 흔한 오해 | [11-comparison.md](11-comparison.md) |
| 유튜브 링크 재생(R3) | [../rust-video-player-youtube.md](../rust-video-player-youtube.md) (묶음 A·B 공통) |

| 파일 | 담은 것 |
| --- | --- |
| README.md | 읽기 지도, 결론, 용어 |
| 00-requirements.md | 범위, "Rust 만"의 해석, R1·R2 인수 기준, PotPlayer 범위 |
| 01-stack-decision.md | UI 후보 비교, 단계안(1단계 glow → 2단계 wgpu), 대안, 버전 고정 |
| 02-decoding-engine.md | 엔진 비교, render API 의 성격, 통합 방식, libmpv2 범위 |
| 03-rendering-pipeline.md | 복사의 종류, glow 콜백 흐름, 2단계 D3D11→wgpu 브리지 |
| 04-player-core.md | 크레이트 구성, mpv 스레드 규칙, Command/Event, 종료 순서 |
| 05-explorer-tabs.md | egui_ltreeview·egui_tiles, 탭 표면, 숨김 탭·동시 재생 |
| 06-r1-hwdec.md | hwdec 값, copy-back, 상태 표시, 전환 상태 기계 |
| 07-r2-video-options.md | 속성·vf-command·필터 교체, 기능별 필터, 체인 설계 |
| 08-playback-features.md | PotPlayer 기능 이름 → mpv 구현 대응표 |
| 09-distribution-licensing.md | GPL 결정, DLL 구하기, MSVC 링크, LGPL 대체 수단 |
| 10-validation-and-open-questions.md | PoC 순서, 측정 corpus, 미확인 목록, 결정 기록 |
| 11-comparison.md | 방식별 비교, 최종 선택·단계, 위험 3가지, 자주 틀리는 점, 검증 체크리스트, 판정 요약 |

## 2. 한눈에 보는 결론

1. **1단계 스택**: eframe 0.37(glow) + egui_tiles 0.18 + egui_ltreeview 0.10 + libmpv2 7.0.0 + libmpv 0.41 OpenGL render API. 출시된 mpv 가 앱 소유 표면에 그리는 방법은 OpenGL 과 SW 뿐이다 ([render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)).
2. **2단계**: mpv 0.42 의 D3D11 render API 가 출시되면 eframe 을 wgpu(DX12)로 바꾸고 D3D11→D3D12 공유 텍스처로 옮긴다. 0.42.0 은 아직 출시되지 않았다 ([client-api-changes.rst](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst), [Releases](https://github.com/mpv-player/mpv/releases)).
3. **R1**: mpv `hwdec` 속성을 재생 중에 `auto` ↔ `no` 로 바꾼다. 실패하면 mpv 가 SW 로 전환한다. 1단계(WGL)에서는 직접 interop 에 ANGLE 이 필요하므로, 하드웨어 디코딩이 대부분 copy-back(`d3d11va-copy`)이 된다 ([render_gl.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h)).
4. **R2**: 밝기·대비·채도·디인터레이스는 mpv 속성, 선명·흐림·노이즈 제거는 lavfi `cas`·`gblur`·`hqdn3d` 를 레이블 필터로 두고 `vf-command`·`vf toggle` 로 실시간 조절한다 ([input.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst), [FFmpeg Filters](https://ffmpeg.org/ffmpeg-filters.html)).
5. **주요 위험**: copy-back 비용(실측 필요), eframe GL 콜백과 mpv 스레드 규칙, Windows 용 안정판 libmpv 바이너리의 출처, mingw DLL 의 MSVC 링크.

판단 근거와 최종 제안은 [11-comparison.md](11-comparison.md) 2.2 절에 있다.

## 3. 용어

| 용어 | 뜻 |
| --- | --- |
| hwdec | mpv 의 하드웨어 디코딩 설정. `no` = 소프트웨어 |
| render API | mpv 가 앱이 준 표면(GL FBO 등)에 영상을 그리는 libmpv API |
| copy-back | GPU 로 디코딩한 프레임을 시스템 RAM 으로 내렸다가 다시 올리는 방식 (`-copy` 모드) |
| zero-copy | 디코더 표면을 복사 없이 바로 샘플링하는 방식. "CPU 왕복 없음"과는 다르다 |
| airspace | 자식 HWND 위에 부모 창의 UI 를 그릴 수 없는 Windows 의 제약 |
| lavfi | mpv 에서 FFmpeg libavfilter 필터를 쓰는 브리지 (`--vf=lavfi=[…]`) |
| 1단계 / 2단계 | 01-stack-decision.md 의 단계안. 1단계 glow + 0.41, 2단계 wgpu + 0.42 |

---

다음: [00-requirements.md](00-requirements.md)
