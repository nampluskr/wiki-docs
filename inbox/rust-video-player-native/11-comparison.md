---
tags: [rust, video-player, comparison, risk, checklist]
created: 2026-11-01
updated: 2026-11-01
---

# 11. 방식 비교와 최종 선택

> 이 파일을 읽는 때: 어떤 조합으로 갈지 정하거나 다시 볼 때, 위험과 흔한 오해를 확인할 때, 구현 후 검증 계획을 세울 때.

입구는 [README.md](README.md) 다. 각 방식의 상세는 해당 파일에 있고, 여기서는 비교와 결론만 쓴다. 수치(기간·크기·성능)는 사용자 결정에 따라 넣지 않고 "실측 필요"로 적는다.

## 1. 방식별 비교

R1·R2 칸의 "위임"은 mpv 속성·필터로 해결된다는 뜻이다 ([06-r1-hwdec.md](06-r1-hwdec.md), [07-r2-video-options.md](07-r2-video-options.md)).

| 방식 | 4K60 | R1·R2 | 범용 재생 | 구현량 | 영상 위 OSD | 상세 |
| --- | --- | --- | --- | --- | --- | --- |
| **1단계.** eframe+glow + libmpv 0.41 OpenGL render API | 실측 필요. HW 디코딩이 대부분 copy-back 이다 | 위임 | mpv 빌드에 달림 | 작음~중 | 가능 (같은 GL 장면) | [01](01-stack-decision.md) 3절, [03](03-rendering-pipeline.md) 2절 |
| **2단계.** eframe/egui wgpu(DX12) + libmpv 0.42 D3D11 render API | 실측 필요. `d3d11va` 직접 디코딩을 기대한다 | 위임 | 같음 | 중~큼. 미출시 API, FFI, D3D11→D3D12 브리지 | 가능 | [01](01-stack-decision.md) 4절, [03](03-rendering-pipeline.md) 3절 |
| 대안. Win32 + libmpv `wid` 자식 창 | 실측 필요. mpv 가 D3D11 출력 | 위임 (+`d3d11vpp`) | 같음 | 큼 (Win32 UI 직접) | 불가 (airspace) | [01](01-stack-decision.md) 5절 |
| 대안. wgpu + egui + ffmpeg-next 직접 | 실측 필요. 브리지 성공 여부에 달림 | 직접 구현 | 직접 구현 | 가장 큼 | 가능 | [01](01-stack-decision.md) 5절, [03](03-rendering-pipeline.md) 4절 |
| (참고) iced + GStreamer | 실측 필요 | 부분 (파이프라인 재구성) | playbin3 | 중~큼 | 가능 (추정) | [01](01-stack-decision.md) 5절 |

근거: 안정판 render API 는 OpenGL/SW 뿐이다([render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)). Windows 에서 직접 HW interop 을 쓰려면 ANGLE 이 필요하다([render_gl.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h)). D3D11 render API 는 0.42.0 항목이다([client-api-changes.rst](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst)). 구현량은 세 scouter 보고서(B8)의 상대 평가다.

## 2. 최종 선택

| 대상 | 선택 | 순서 |
| --- | --- | --- |
| 지금 | **1단계: eframe+glow + libmpv 0.41** | PoC → 1단계 구현 → copy-back 실측 후 재검토 |
| 0.42 출시 후 | **2단계: wgpu + D3D11 render API** | 브리지 PoC → `render-gl` 을 `render-wgpu` 로 교체 |
| 공통 | 같은 `player-api`(Command/Event), 탭마다 mpv core, R1·R2 는 mpv 속성·필터 | [04-player-core.md](04-player-core.md) |
| UI 크레이트 | egui 0.37 + egui_tiles 0.18 + egui_ltreeview 0.10 | [05-explorer-tabs.md](05-explorer-tabs.md) 1절 |
| 라이선스 | GPL 허용 | [09-distribution-licensing.md](09-distribution-licensing.md) |

선택 이유 한 줄씩:
- 출시된 mpv 로 앱 소유 표면에 그리는 방법은 OpenGL 뿐이고, egui 로 OpenGL 을 쓰는 길은 glow 다. https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h
- 영상 위 UI(탭 안 OSD)는 render API 경로에서만 된다. 자식 창 임베딩은 mpv 도 문제를 경고한다. 같은 출처
- R1 은 copy-back 이어도 충족된다: `hwdec` 는 render API 에서도 "enabled/disabled at any time" 이다. https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h
- D3D11 render API 는 0.42.0 항목이고 아직 출시되지 않았다(최신 안정판 v0.41.0). https://github.com/mpv-player/mpv/releases

### 2.1 후보별 장단점

상세 표는 [01-stack-decision.md](01-stack-decision.md) 7.1 에 있다. 여기서는 요약한다.

| | 1단계 glow + 0.41 | 2단계 wgpu + 0.42 D3D11 | 대안 Win32 + `wid` | 대안 FFmpeg 직접 |
| --- | --- | --- | --- | --- |
| 장점 | 출시된 API 만 쓴다. OSD 가능. Rust 한 언어. 위젯 버전이 맞는다 | `d3d11va` 직접 디코딩 기대. 렌더 크레이트만 교체 | 직접 디코딩·`d3d11vpp`·HDR 옵션. 지금 된다 | 디코더·셰이더 완전 통제. wgpu NV12/P010 피처 ([Features](https://docs.rs/wgpu/latest/wgpu/struct.Features.html)) |
| 단점 | WGL 이라 HW 가 copy-back. `d3d11vpp` 불가 ([vf.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst)). UI 스레드 = 렌더 스레드 | 미출시, Rust 래퍼 없음 ([libmpv2 7.0.0](https://docs.rs/libmpv2/7.0.0/libmpv2/render/index.html)), unsafe 브리지, 리뷰어가 말한 "제한" 미확인 ([PR #17764](https://github.com/mpv-player/mpv/pull/17764)) | airspace. 트리·분할·도킹·DPI 를 Win32 로 직접 | A/V 동기·탐색·ASS·오디오·R1 을 전부 직접. 구현량이 가장 크다 |
| 공통 위험 | GPU·드라이버별 HW 디코딩 편차, libmpv DLL 배포·ABI (3절) | 같음 | 같음 | 같음 + FFmpeg 빌드 관리 |

### 2.2 최종 제안 — 1단계로 시작, 0.42 출시 후 2단계

사용자 확정(2026-11-01). W1: 단계안 유지, 1단계 copy-back 비용은 실측한 뒤 다시 검토한다 ([research/reviews/writer-notes.md](research/reviews/writer-notes.md)).

| 단계 | 할 일 | 끝내는 조건 |
| --- | --- | --- |
| 0. PoC | eframe glow 탭 하나에 libmpv 영상 + OSD. `hwdec` 토글, `hwdec-current` 기록 ([10](10-validation-and-open-questions.md) P1·P2) | 영상·OSD 가 나오고, stuck 로그 없이 R1 토글이 된다. 막히면 Win32 대안을 검토한다 |
| 1. 1단계 구현 | 코어·탭·트리·R1·R2·재생 기능 ([04](04-player-core.md)~[08](08-playback-features.md)) | [00-requirements.md](00-requirements.md) 의 R1·R2 인수 기준 + P3·P4 통과 |
| 1.5 copy-back 재검토 | 4K·고프레임 corpus 로 CPU·전력·drop 측정 (Q4) | 허용 범위면 유지. 아니면 ANGLE 강제 셸(Q5) 또는 2단계를 앞당긴다 |
| 2. 2단계 전환 | 0.42.0 안정판 출시 후 D3D11 render API + wgpu 브리지 PoC, `render-wgpu` 교체 (Q1·Q2·Q3·Q15) | 0.42 출시, 래퍼 또는 FFI, 브리지 PoC 성공, 1단계와 같은 검증 통과 |
| 예비 | 디코더 통제가 제품의 핵심이 되면 FFmpeg 직접 경로 | 요구 변경 시 |

이유:
- 엔진(libmpv)이 같아 R1·R2·자막·트랙은 1·2단계가 같다. 차이는 출력 경로와 복사 비용이다.
- 렌더만 갈아 끼우도록 크레이트를 나누었으므로 2단계 전환 비용은 렌더 크레이트에 한정된다 ([04-player-core.md](04-player-core.md) 1절).
- Win32 `wid` 는 직접 디코딩이 되지만 영상 위 UI 를 포기해야 하고 UI 를 전부 직접 만들어야 한다.
- FFmpeg 직접 구현은 같은 기능을 얻는 데 드는 구현량이 가장 크다.

## 3. 위험 3가지

| # | 위험 | 내용 | 대응 |
| --- | --- | --- | --- |
| 1 | **copy-back 비용** | eframe 0.37 의 Windows GL 은 WGL 이 기본이고, mpv OpenGL API 의 직접 HW interop 에는 ANGLE 이 필요하다. 그래서 대부분 `d3d11va-copy` 가 된다. 4K 고프레임에서 CPU·전력 이점이 줄 수 있다 | PoC P2 와 1.5 단계에서 실측. 나쁘면 ANGLE 강제 셸(Q5)이나 2단계를 앞당긴다 |
| 2 | **스레드 규칙과 교착** | glow 콜백이 UI 스레드에서 돈다. render 스레드에서 일반 mpv API 를 부르거나 기다리면 "not being called or stuck" 이 생기고, `ADVANCED_CONTROL` 을 켜면 진짜 교착이 된다 | UI 는 Command 채널만 쓴다. update 콜백은 플래그와 repaint 만. render context 를 core 보다 먼저 해제 ([04](04-player-core.md)) |
| 3 | **libmpv 바이너리·링크** | Windows 용으로 바로 받을 수 있는 libmpv 는 CI·서드파티 **개발 빌드**(mingw)다. 안정판 근거가 없고, MSVC import library 도 미확인이다 | 빌드를 고정하고 `mpv_client_api_version()` 을 기록한다. client API 2.5 만 쓴다. 필요하면 v0.41.0 태그를 직접 빌드한다 ([09](09-distribution-licensing.md) 3절) |

출처: [render_gl.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h), [eframe glow_integration.rs 0.37.0](https://github.com/emilk/egui/blob/0.37.0/crates/eframe/src/native/glow_integration.rs), [render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h), [git-release](https://github.com/mpv-player/mpv/releases/tag/git-release), sol followup-1.

## 4. 자주 틀리는 점

| 틀린 전제 | 사실 | 출처 |
| --- | --- | --- |
| libmpv 로 wgpu(DX12)에 바로 그릴 수 있다 | 안정판 render API 는 OpenGL/SW 뿐. D3D11 은 0.42.0, libmpv2 7.0.0 에는 래퍼가 없다 | [render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h), [libmpv2 render](https://docs.rs/libmpv2/7.0.0/libmpv2/render/index.html) |
| glow 로 가면 `d3d11va` 직접 디코딩이 된다 | WGL 에서는 안 된다. "Windows: ANGLE is required" | [render_gl.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h) |
| `hwdec=auto` 는 직접 모드만 고른다 | autoprobe 표에 `d3d11va-copy`·`dxva2-copy`·`nvdec-copy` 도 있다 | [vd_lavc.c v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/video/decode/vd_lavc.c) |
| "CPU 왕복 없음" = zero-copy | D3D11 출력의 기본은 GPU-to-GPU 복사다. zero-copy 는 별도 옵션이고 위험이 있다 | [options.rst `--d3d11va-zero-copy`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| `hwdec` 전환은 무중단이다 | 실행 중 전환은 되지만 무중단 보장은 없다. 폴백 때 일부를 건너뛸 수 있다 | options.rst `--hwdec-software-fallback` |
| egui_dock 을 최신 egui 와 같이 쓴다 | egui_dock 0.21.1 은 egui ^0.36. egui_tiles 0.18 은 ^0.37 | [crates.io egui_dock](https://crates.io/crates/egui_dock), [egui_tiles](https://crates.io/crates/egui_tiles) |
| 모든 lavfi 필터는 실행 중 값을 바꿀 수 있다 | `hqdn3d`·`gblur`·`cas`·`eq` 는 명령을 받지만 `unsharp`·`bwdif`·`nlmeans` 는 아니다 → 교체한다 | [FFmpeg Filters](https://ffmpeg.org/ffmpeg-filters.html) |
| `deinterlace` 를 켜고 디인터레이스 필터도 넣는다 | 수동 디인터레이스 필터와 충돌한다 | [options.rst `--deinterlace`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| `d3d11vpp` 는 어디서나 쓴다 | "Requires a D3D11 context". 1단계(GL)에서는 불가 | [vf.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst) |
| HW 디코더는 세션 수 상한(예: 8개)이 있다 | 고정 상한은 근거가 없다. 원인은 VRAM 부족·미지원 프로파일 | [CreateVideoDecoder](https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videodevice-createvideodecoder), argon followup-1 |
| mpv 는 항상 LGPL / 크레이트가 MIT 면 전체가 MIT | mpv 기본은 GPLv2+, `-Dgpl=false` 일 때 LGPLv2.1+. FFmpeg 에 GPL 필터가 있다 | [mpv Copyright](https://github.com/mpv-player/mpv/blob/master/Copyright), [FFmpeg LICENSE.md](https://github.com/FFmpeg/FFmpeg/blob/master/LICENSE.md) |
| `libmpv2` 최신은 6.0.x | 7.0.0 (2026-10-30) | [crates.io libmpv2](https://crates.io/crates/libmpv2) |

## 5. 검증 체크리스트

**미실행 설계 제안이다.** 이 묶음은 조사만 했다. PoC 순서와 미확인 목록 전체는 [10-validation-and-open-questions.md](10-validation-and-open-questions.md) 에 있다.

### 5.1 환경 매트릭스

| 축 | 값 |
| --- | --- |
| GPU | Intel 내장 / AMD / NVIDIA, 하이브리드 노트북 |
| 모니터 | 60/120/144Hz, DPI 100% ↔ 150%, 다중 모니터 |
| OS·빌드 | Windows 11, Rust stable-msvc, 고정한 libmpv 빌드(해시·client API 버전 기록) |

### 5.2 시나리오

| 영역 | 확인할 것 |
| --- | --- |
| 코덱·컨테이너 | MP4·MKV·WebM·TS, H.264·HEVC 8/10bit·VP9·AV1, AAC·AC3·Opus, VFR, 인터레이스 원본, 손상 파일 |
| R1 | `auto` ↔ `no` 반복 토글, `hwdec-current` 값, 실패 시 SW 전환, 전환 중 위치·트랙·필터 유지 |
| R2 | 슬라이더 드래그 중 정지 시간, toggle·교체·`vf-command`, CPU 필터의 CPU 사용률 |
| 탭 | 분할, 숨김 탭 일시정지, 동시 재생·오디오 포커스, 닫기 시 해제 순서 |
| 렌더 | stuck 로그 유무, DPI·리사이즈 때 탭 FBO 재생성, 한 프레임 지연(Q6) |
| 자막 | 외부 SRT·ASS·SMI, 한글 경로 |
| 배포 | 깨끗한 PC 에서 DLL 로드, MSVC 링크 |

### 5.3 측정 항목

실제 hwdec, CPU/GPU 사용률, dropped frames, A/V drift, 메모리(VRAM 포함), 전력, R1 전환 시간, R2 적용 지연. 통계는 mpv 속성으로 얻는다 ([매뉴얼 Property List](https://mpv.io/manual/stable/#property-list)).

## 6. 확신이 낮은 부분

| 항목 | 상태 |
| --- | --- |
| WGL 에서 `auto` 가 실제로 고르는 값, copy-back 비용 | **미측정** (Q4) |
| ANGLE 을 강제한 셸에서 `d3d11va` 직접 interop | **미확인** (Q5) |
| egui paint callback 과 이미지 도형의 실행 순서 | **미확인** (Q6) |
| `hwdec` 전환 때 위치·필터 유지, 정지 시간 | **미측정** (Q7) |
| disabled 필터 비용, 필터 교체 정지, `vf-command` 의 label 표기 | **미확인** (Q8·Q9) |
| mingw libmpv DLL 의 MSVC 링크, 공식 안정판 Windows 바이너리 | **미확인** (Q11·Q12) |
| libmpv2 6.x → 7.0 API 변경 | **미확인** (Q13) |
| mpv 0.42.0 출시 시기, D3D11 render API 의 "제한" | **미확인** (Q1·Q3) |
| D3D11→D3D12→wgpu 브리지의 동기화·포맷 | **미확인** (Q15) |
| HDR end-to-end | **미확인** (Q14). 필수 아님 |
| `.smi` 자동 로드 | **미확인** (Q16) |
| PotPlayer 의 R2 기능 | 공식 출처 없음. 이 프로젝트의 요구사항으로 본다 (W3) |
| 개발 기간·배포 크기·전력 | **실측 필요** (사용자 결정) |

## 7. 보고서와 다르게 쓴 값 (판정 요약)

세 scouter 보고서가 어긋난 값은 rvpn-lead 가 출처를 직접 열어 판정했다. 전체는 [research/reviews/comparison.md](research/reviews/comparison.md) 의 "판정 요약".

| 항목 | 채택 | 본문 위치 |
| --- | --- | --- |
| 1순위 렌더 경로 (wgpu vs glow) | glow 로 시작하는 단계안 (사용자 결정) | [01](01-stack-decision.md) 7절 |
| `libmpv2` 최신 | 7.0.0 (2026-10-30) | [02](02-decoding-engine.md) 4절 |
| mpv D3D11 render API | 0.42.0 개발판에만 있다. libmpv2 래퍼 없음 | [01](01-stack-decision.md) 3·4절 |
| "Zero-Copy" | CPU 왕복 / GPU 복사 / zero-copy 를 구분한다 | [03](03-rendering-pipeline.md) 1절 |
| egui 위젯 호환 | egui_tiles 0.18 + egui_ltreeview 0.10 (egui 0.37). egui_dock 0.21.1 은 0.36 | [05](05-explorer-tabs.md) 1절 |
| iced 최신 | 0.14.0 (0.14.1 없음) | [01](01-stack-decision.md) 2절 |
| R1 "무중단" | 근거 없음. 실행 중 전환·폴백만 | [06](06-r1-hwdec.md) 1절 |
| 기간·배포 크기 | 수치 없음, 실측 필요 (사용자 결정) | [09](09-distribution-licensing.md) 4절 |
| libmpv 라이선스 | 기본 GPLv2+, `-Dgpl=false` LGPLv2.1+. GPL/LGPL 변형 바이너리는 CI 개발 빌드에만 | [09](09-distribution-licensing.md) 2·3절 |
| HW 세션 수 상한 | 근거 없음 (argon 정정) | [05](05-explorer-tabs.md) 5.1 |
| libmpv-2.dll 크기 | 출처 없는 값은 빼고 압축 파일 크기만 | [09](09-distribution-licensing.md) 4절 |

## 8. 묶음 A 와의 관계

엔진은 두 묶음 모두 같은 libmpv 이고, R1·R2 도 mpv 속성·필터로 같다. 차이는 출력 방식이다. 묶음 A 는 탭 전용 자식 HWND(mpv 가 D3D11 로 직접 출력, 무복사 경로, 영상 위 OSD 불가)를 쓴다. 묶음 B 는 glow render API(같은 GL 장면이라 OSD 가능, 대신 HW 디코딩이 copy-back)를 쓴다. 묶음 A 의 비교는 [../rust-video-player/07-COMPARISON.md](../rust-video-player/07-COMPARISON.md) 에 있다.

---

이전: [10-validation-and-open-questions.md](10-validation-and-open-questions.md) · 입구: [README.md](README.md)
