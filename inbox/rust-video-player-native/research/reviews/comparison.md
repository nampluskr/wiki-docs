---
tags: [rust, video-player, research, comparison]
created: 2026-11-01
updated: 2026-11-01
---

# scouter 3인 보고서 비교 (B1~B10)

- 대상: `research/sources/sonnet/report.md`(S), `research/sources/sol/report.md`(L), `research/sources/argon/report.md`(A)
- 분류: **셋 다 일치** / **둘만 일치** / **상충** / **한쪽만**
- 상충은 rvpn-lead 가 출처를 직접 열어 판정했다(2026-11-01). 판정 근거는 각 행의 "판정" 칸과 맨 아래 "직접 확인한 출처" 에 있다.
- 항목 ID(`B1-1` 형식)는 `outline.md` 에서 근거로 쓴다.

## 판정 요약

| ID | 쟁점 | 분류 | 판정 |
|---|---|---|---|
| B1-2 | 1순위 렌더 경로: wgpu(S·A) vs glow/OpenGL(L) | 상충 | 안정판 libmpv 기준이면 L 이 맞다. 최종 선택은 **사용자 판단 필요** |
| B4-1 | libmpv2 최신 버전: 7.0.0(S) vs 6.0.x(L·A) | 상충 | **7.0.0 (2026-10-30)**. S 가 맞다 |
| B4-2 | libmpv D3D11 render API 상태 | 상충 (S·L vs A) | 0.42.0 개발판에만 있다. 안정판은 v0.41.0. libmpv2 7.0.0 에는 래퍼가 없다. A 의 서술은 틀렸다 |
| B3-2 | "Zero-Copy" 표현 | 상충 (L vs A) | GPU→GPU 복사가 기본이다. L 의 구분(CPU 왕복 없음 ≠ GPU 복사 없음)이 맞다 |
| B5-2 | egui 0.37 과 탭/트리 크레이트 호환 | 상충 (L vs S·A) | egui_dock 0.21.1 은 egui ^0.36 이다. egui_tiles 0.18.0 과 egui_ltreeview 0.10.0 은 ^0.37 이다 |
| B2-3 | iced 버전 0.14.0(S·L) vs 0.14.0/0.14.1(A) | 상충 | 0.14.0 이 최신이다(2025-12-07). 0.14.1 은 없다 |
| B7-1 | R1 재생 중 전환이 "무중단"(A)인가 | 상충 (A vs L) | 런타임 전환은 공식 문서에 있다. "무중단·프레임 유실 0" 은 근거가 없다. L 의 표현을 쓴다 |
| B8-1 | 개발 기간·배포 크기 추정 | 상충 | 모두 근거 없는 추정이다. **사용자 판단 필요**(실측 전 범위로만 표기) |
| B4-4 | libmpv 라이선스 | 상충 (세 표현이 다름) | 원문은 기본 GPLv2+, `-Dgpl=false` 이면 LGPLv2.1+ 이다. 배포 바이너리의 GPLv3/LGPLv3 표기(L)는 미확인 |

## B1 원점 권장 스택

| ID | 내용 | 분류 | 근거·판정 |
|---|---|---|---|
| B1-1 | 1순위 디코딩 엔진은 libmpv(libmpv2 바인딩)다. 이유는 코덱·자막·트랙·hwdec 를 재사용할 수 있어서다 | 셋 다 일치 | S·L·A |
| B1-2 | 1순위 UI·렌더: S·A 는 winit+wgpu(DX12)+egui+libmpv. L 은 eframe/egui + **glow(OpenGL)** + libmpv 0.41.0 | 상충 | 안정판 v0.41.0 의 `render.h` 는 OpenGL 과 SW API 만 제공한다. 그래서 wgpu(DX12) 위에 libmpv 를 얹으려면 둘 중 하나가 필요하다. (a) 아직 출시되지 않은 0.42.0 D3D11 render API 와 D3D11→D3D12→wgpu 브리지, (b) 별도의 GL↔DX interop. **안정판만 쓴다는 조건이면 L 의 glow 경로만 확인된 경로다.** 둘 중 무엇을 고를지(안정판 glow 로 지금 시작할지, 0.42 출시 후 wgpu 로 갈지)는 **사용자 판단 필요** |
| B1-3 | 2순위 대안: S·L 은 Win32(windows 크레이트)+D3D11+libmpv. A 는 winit+wgpu+egui+**ffmpeg-next** 직접 구현 | 둘만 일치 (S·L) | A 의 2순위는 L 의 B8 "winit+wgpu+FFmpeg 직접"과 같고, L 은 이를 가장 위험한 단계로 둔다. 문서에는 "Win32 대안"과 "FFmpeg 직접 대안"을 둘 다 적는다 |
| B1-4 | 단계적 접근: A(egui+libmpv) → 출력·HDR 이 우선이면 Win32 → decoder texture 통제가 핵심이면 FFmpeg 직접 | 한쪽만 (L) | 결정 순서의 틀로 채택할 만하다 |

## B2 창·UI 후보 비교

| ID | 내용 | 분류 | 근거·판정 |
|---|---|---|---|
| B2-1 | 동영상 + 트리·탭 용도로 egui 가 가장 성숙하다 | 셋 다 일치 | eframe 0.37.0 (2026-10-15) 확인 |
| B2-2 | 버전: winit 0.30.13, wgpu 30.0.1, slint 1.18.1, floem 0.2.0, makepad-widgets 1.0.0, xilem 0.4.0, windows 0.62.2 | 셋 다 일치 | windows 0.62.2 는 2025-10-06 게시인데, crates.io 에서 이것이 실제 최신이다(S 의 의문은 해소됨) |
| B2-3 | iced 0.14.0 (S·L) vs "0.14.0/0.14.1" (A) | 상충 | crates.io 최신은 0.14.0 (2025-12-07). A 의 0.14.1 은 틀렸다 |
| B2-4 | floem·xilem 은 정체되었거나 실험 단계라 후순위다 | 셋 다 일치 | |
| B2-5 | makepad 에 FileTree/Dock 모듈이 있어 구조상 적합성이 높다(영상 통합은 미확인) | 둘만 일치 (L·A) | S 는 미확인 |
| B2-6 | slint: 라이선스 3중(GPLv3/Royalty-free/상용). `.slint` DSL 은 "Rust 문법만" 조건에 맞지 않을 수 있다 | 라이선스 셋 다 일치, DSL 지적은 한쪽만 (L) | |
| B2-7 | Win32 Common Controls 로 트리·탭을 만들 수 있지만, 현대적 분할·도킹·오버레이는 직접 구현해야 한다 | 셋 다 일치 | |
| B2-8 | 추가 후보 dioxus 0.7.9, freya 0.4.3, vizia 0.4.0 은 부적합하다 | 한쪽만 (A) | 버전 미확인 |
| B2-9 | 추가 후보 egui-sharkplayer 0.5.1 (Linux만 표시) | 한쪽만 (S) | |
| B2-10 | 추가 후보 winit+glutin+egui_glow 를 eframe 없이 직접 조립 | 한쪽만 (L) | |
| B2-11 | vsync 지원이 영상 PTS·A/V 동기를 보장하지는 않는다. 정지 화면에서 무한 GPU 루프를 피한다 | 한쪽만 (L) | |

## B3 렌더링

| ID | 내용 | 분류 | 근거·판정 |
|---|---|---|---|
| B3-1 | D3D11 텍스처 → wgpu 경로: NT 공유 핸들 → `ID3D12Device::OpenSharedHandle` → `wgpu_hal::dx12` `texture_from_raw` → `create_texture_from_hal`. wgpu 공식 고수준 API 는 아니다 | 셋 다 일치 | L 은 덧붙여 `create_texture_from_hal` 의 백엔드에 **D3D11 HAL 이 없다**고 적었다 |
| B3-2 | "Zero-Copy" 가능 여부 | 상충 (A vs L, S 는 L 쪽) | A 는 "Zero-Copy 가능"이라 쓰면서 1단계에서 `CopySubresourceRegion`(GPU 복사)을 요구한다. mpv 문서도 d3d11 에서 기본은 decoder surface→shader resource GPU-to-GPU 복사라고 하고, `--d3d11va-zero-copy` 에는 가장자리 오샘플링과 드라이버 버그 가능성이 있다고 적는다. **판정: "CPU 왕복 없음"과 "GPU 복사 없음"을 구분하는 L 이 맞다** |
| B3-3 | 동기화(fence·keyed mutex)와 같은 어댑터(LUID)가 필요하다 | 셋 다 일치 | S 는 동기화 방법이 미확인이라고 적었다 |
| B3-4 | wgpu `TEXTURE_FORMAT_NV12`·`P010` 피처 | 둘만 일치 (S·A) | docs.rs wgpu 30.0.1: DX12·Vulkan, native 전용. 확인됨 |
| B3-5 | `Rgba16Float` scRGB 스왑체인으로 HDR10 을 "완벽 지원" | 한쪽만 (A) | 근거 없음. S·L 은 HDR 을 미확인으로 둔다. 문서에서는 미확인으로 쓴다 |
| B3-6 | YUV→RGB 는 프래그먼트 셰이더에서 한다. 행렬(601/709/2020)과 범위(limited/full)는 uniform 으로 넘긴다 | 셋 다 일치 | |
| B3-7 | R2 위치: 디인터레이스를 먼저 하고, 공간 필터, 색 조정은 마지막 | 셋 다 일치 (세부 순서는 다름) | L 은 "UI 이후에 효과를 적용하지 않는다(글자 흐림 방지)"를 추가했다 |
| B3-8 | libmpv 경로에서는 R2 를 mpv 내부(GLSL hook·vf)에 둔다 | 둘만 일치 (L·A) | |

## B4 디코딩 바인딩

| ID | 내용 | 분류 | 근거·판정 |
|---|---|---|---|
| B4-1 | libmpv2 최신 버전 | 상충 | crates.io API: **7.0.0 (2026-10-30)**, 6.0.1 (2026-10-29), 6.0.0 (2026-05-12). S 가 맞고 L·A 는 옛 버전이다. 7.0.0 은 libmpv2-sys ^4.0.1 에 의존한다 |
| B4-2 | libmpv D3D11 render API | 상충 (S·L vs A) | mpv `client-api-changes.rst`(master): "--- mpv 0.42.0 --- 2.6 add MPV_RENDER_API_TYPE_D3D11". GitHub 릴리스의 최신 안정판은 v0.41.0 (2025-12-21) 이다. docs.rs libmpv2 7.0.0 의 render 모듈에는 D3D11 타입이 없다(OpenGLInitParams 만 있음). **판정: 안정판에는 없고, Rust 래퍼도 없다.** A 의 "mpv_render_context gpu-next D3D11" 서술은 안정판 기준으로 틀렸다 |
| B4-3 | ffmpeg-next 9.0.0, gstreamer 0.25.4 | 셋 다 일치 | |
| B4-4 | libmpv 라이선스 | 상충 | mpv `Copyright` 원문: 기본은 GPLv2+, `-Dgpl=false` 로 GPL 전용 파일 없이 빌드하면 LGPLv2.1+. S(GPLv2+)는 원문과 맞는다. A 의 "LGPLv2.1+/GPLv2+"도 범위는 맞다. L 의 "공식 배포 GPLv3 / LGPLv3 변형"은 배포 바이너리 표기라 **미확인**이다(본문 작성 때 L 에게 재질의) |
| B4-5 | FFmpeg 는 기본 LGPL-2.1+ 이고, GPL 부분을 넣으면 전체가 GPL 이 된다. 동적 링크만으로 의무가 끝나지 않는다 | 둘만 일치 (L·A) | L 이 ffmpeg.org/legal.html 을 인용했다 |
| B4-6 | 통합 방식 3가지: render API(텍스처) / `wid` 자식 HWND / CPU 업로드 | 셋 다 일치 | 자식 HWND 의 airspace(오버레이 불가) 문제도 셋 다 일치. mpv `render.h` 도 window embedding 의 문제를 경고한다(확인) |
| B4-7 | libmpv render API 는 디코드된 프레임을 주지 않고, 앱의 타깃에 그려 준다. core 하나에 render context 하나 | 한쪽만 (L) | render.h 에 근거 |
| B4-8 | 배포 난이도: libmpv DLL 1개 < FFmpeg DLL 5~7개 < GStreamer 수십 개 | 한쪽만 (A) | 경험칙, 미확인 |
| B4-9 | mpv-engine 0.2.1 (wgpu 내보내기, 다운로드 444건) | 한쪽만 (S) | |

## B5 파일 탐색기 + 탭

| ID | 내용 | 분류 | 근거·판정 |
|---|---|---|---|
| B5-1 | 탭·분할은 egui_dock 또는 egui_tiles. 트리는 egui_ltreeview 또는 CollapsingHeader 직접 구현 | 둘만 일치 (L·A). S 는 트리 크레이트를 찾지 못했다 | egui_ltreeview 는 crates.io 에 있다: 0.10.0 (2026-10-25), 0.9.0 (2026-08-23) |
| B5-2 | egui 0.37 과 위젯 크레이트 호환 | 상충 (L 만 지적) | crates.io 의존성: egui_dock 0.21.1 → egui **^0.36**. egui_tiles 0.18.0 → egui **^0.37.0**. egui_ltreeview 0.10.0 → **^0.37**, 0.9.0 → ^0.36. **판정: egui 0.37 + egui_tiles 0.18 + egui_ltreeview 0.10 조합이 의존성상 일치한다**(cargo 빌드는 하지 않음). egui_dock 을 쓰려면 0.36.2 로 고정해야 한다(L) |
| B5-3 | 숨김 탭은 기본으로 일시정지한다. 동시 재생은 탭마다 mpv core 를 따로 두고, 오디오는 포커스된 탭 하나에서만 낸다 | 셋 다 일치 | |
| B5-4 | 렌더 skip: `MPV_RENDER_PARAM_SKIP_RENDERING` 를 쓰거나 업데이트를 처리한다 | 둘만 일치 (S·L) | |
| B5-5 | rfd·notify 재사용 (A 는 rfd 0.17.2, notify 8.2.1, walkdir 2.5.0) | 둘만 일치 (L·A) | 버전은 A 만 적음, 미확인 |
| B5-6 | 첫 버전은 탭을 외부 창으로 떼어내지 않는다(GL context 이동 비용) | 한쪽만 (L) | |
| B5-7 | HW 디코더 세션 수 한계에 걸리면 추가 탭은 SW 로 내린다 | 한쪽만 (A) | 근거 URL 없음 |
| B5-8 | 트리 ID 를 경로 문자열과 분리한다(SessionId) | 한쪽만 (L) | |

## B6 코어·UI 분리

| ID | 내용 | 분류 | 근거·판정 |
|---|---|---|---|
| B6-1 | UI 와 무관한 코어 + 엔진 래퍼 + 렌더 + UI 크레이트로 나누고, Command/Event 채널을 둔다 | 셋 다 일치 | 크레이트 수는 S 5개, L 7개, A 4개로 다르다. 설계 선택이라 상충으로 보지 않는다 |
| B6-2 | mpv update 콜백은 플래그를 세우고 UI 를 깨우기만 한다. 렌더 콜백 안에서 mpv 동기 명령을 부르지 않는다 | 한쪽만 (L) | mpv render.h 스레드 규칙에 근거 |
| B6-3 | Seek 명령은 마지막 값으로 합친다. SessionId+generation 으로 늦은 응답을 버린다 | 한쪽만 (L) | |
| B6-4 | 고빈도 상태는 `Arc<Atomic*>` 로 공유하고 채널은 crossbeam-channel 을 쓴다 | 한쪽만 (A) | |
| B6-5 | 종료 순서: 렌더 중단 → render context 해제 → mpv core 종료 | 한쪽만 (L) | |

## B7 R1·R2·PotPlayer 기능

| ID | 내용 | 분류 | 근거·판정 |
|---|---|---|---|
| B7-1 | R1: 재생 중 `hwdec` 속성을 auto ↔ no 로 바꾼다 | 셋 다 일치 (방법) / 상충 (A 의 "무중단") | mpv v0.41.0 options.rst: "Use the Ctrl+h shortcut to toggle hardware decoding at runtime. It toggles this option between auto and no." 런타임 전환은 확인됐다. "무중단"은 근거가 없다 |
| B7-2 | R1 자동 폴백 | 셋 다 일치 | options.rst: "If hardware decoding is not possible, mpv will fall back on software decoding." `--hwdec-software-fallback`(기본 3 = 연속 3프레임 실패 시). 폴백이 일어나면 파일 일부를 건너뛸 수 있다는 단서도 있다 |
| B7-3 | v0.41.0 에서 `auto-safe` 는 `auto` 와 같다 | 새 확인 | options.rst 확인. S·A 의 `auto-safe` 표기는 유효하지만 의미는 auto 와 같다 |
| B7-4 | UI 에는 요청 상태와 실제 `hwdec-current` 를 따로 표시한다 | 셋 다 일치 | |
| B7-5 | CPU 필터(vf/lavfi)는 HW 표면의 copy-back 이 필요하다(`auto-copy`) | 셋 다 일치 | options.rst: auto-copy 는 "will allow CPU processing with video filters" |
| B7-6 | 밝기·대비·채도는 mpv 속성으로 조절하고 난이도는 하 | 셋 다 일치 | `--brightness/--contrast/--saturation=<-100-100>` 확인. "VO 가 지원할 때만" 단서가 있다 |
| B7-7 | 디인터레이스는 `deinterlace` 속성 | 셋 다 일치 | options.rst: 기본 no. 켜면 보통 bwdif 를 넣거나 VO 디인터레이스를 쓴다 |
| B7-8 | 선명·흐림·노이즈 제거는 GLSL 셰이더 또는 lavfi(unsharp, gblur, hqdn3d) | 셋 다 일치 | 노이즈 제거 난이도는 S 중, L 상, A 중상 → "중~상"으로 쓴다 |
| B7-9 | 노이즈 제거를 D3D11 VideoProcessor 하드웨어 필터로 | 한쪽만 (A) | Microsoft 문서 URL 1개 |
| B7-10 | 모든 lavfi 옵션을 실행 중 바꿀 수 있지는 않다(필터 교체와 파라미터 변경은 다르다) | 한쪽만 (L) | |
| B7-11 | 탐색·속도·음량·트랙·SRT/ASS 는 libmpv 로 난이도 하~중 | 셋 다 일치 | |
| B7-12 | 전체 화면: 트리·탭을 숨기고 winit borderless | 셋 다 일치 | |

## B8 스택 비교표

| ID | 내용 | 분류 | 근거·판정 |
|---|---|---|---|
| B8-1 | 개발 기간: egui+libmpv 를 A 는 2~3개월, L 은 시제품 3~6주 + 사용 가능판 3~5개월로 본다. S 는 "짧음" | 상충 | 모두 추정이다. **사용자 판단 필요** — 문서에는 범위와 "추정"만 적는다 |
| B8-2 | 배포 크기: A 45~65MB, L 60~180MB | 상충 | 실측 없음. **사용자 판단 필요** (빌드 측정 전에는 미확인으로 표기) |
| B8-3 | FFmpeg 직접 구현이 가장 길다 | 셋 다 일치 | |
| B8-4 | Win32 직접은 UI 구현 비용이 가장 크다 | 셋 다 일치 | |
| B8-5 | iced+GStreamer 대안 | 둘만 일치 (L·A) | |

## B9 권장 목차

| ID | 내용 | 분류 |
|---|---|---|
| B9-1 | 요구사항 → 스택 결정 → 디코딩 → 렌더 → 코어 → 탐색기·탭 → R1 → R2 → 재생 기능 → 배포 → 검증 | 셋 다 일치 (순서 대체로 같음) |
| B9-2 | 배포·라이선스를 별도 파일로 | 둘만 일치 (L·A) |
| B9-3 | 미확인 목록 파일(open questions)을 따로 둔다 | 한쪽만 (L) |
| B9-4 | 색·HDR·interop 을 별도 파일로 | 한쪽만 (L) |

## B10 확신 낮은 부분 (통합)

| ID | 내용 | 제기 | 현재 상태 |
|---|---|---|---|
| B10-1 | libmpv D3D11 render API 의 출시 시기와 Rust 래퍼 | S·L·A | 확인: 0.42.0 미출시, libmpv2 7.0.0 미지원. 출시 시기는 미확인 |
| B10-2 | GL render API 에서 D3D11VA interop·copy 경로의 GPU별 동작 | L·A | 미확인 (실측 필요) |
| B10-3 | HDR end-to-end (egui 경로) | S·L (A 는 "완벽"이라 함) | 미확인 |
| B10-4 | D3D11→D3D12→wgpu 동기화와 포맷 허용 | S·L·A | 미확인 (PoC 필요) |
| B10-5 | R2 강도를 실시간으로 바꿀 때 stall | L | 미확인 |
| B10-6 | egui 위젯 조합의 MSVC 빌드 | L | 의존성은 B5-2 에서 확인. 빌드는 미확인 |
| B10-7 | 다중 탭 4K 에서 R2 의 GPU 부하 | A | 추정 |
| B10-8 | wgpu-hal 메이저 버전마다 API 가 바뀐다 → 고정·격리 | A | 주의사항으로 채택 |
| B10-9 | libmpv 배포 바이너리의 라이선스 변형 | S·L | B4-4 참조, L 에게 재질의 |
| B10-10 | 기간·크기·전력 | S·L·A | 추정 (B8) |

## 사용자 판단 필요 (모음)

1. **B1-2** 1순위 렌더 경로: (가) eframe+glow+libmpv 0.41 안정판(지금 확인된 경로) / (나) wgpu+libmpv 0.42 D3D11 render API(미출시, Rust 래퍼 없음, 직접 FFI 와 D3D12 브리지 필요) / (다) (가)로 시작하고 0.42 출시 후 (나)로 옮기는 단계안.
2. **B8-1·B8-2** 기간·배포 크기 수치: 범위로 적을지, 빼고 "실측 필요"로만 적을지.

### 사용자 결정 (2026-11-01, rvp-leader 경유)

1. **B1-2** → (다) 단계안. eframe+glow+libmpv 0.41 안정판으로 시작한다. 0.42 D3D11 render API 가 출시되면 wgpu 로 옮기는 경로를 함께 적는다.
2. **B8-1·B8-2** → 수치는 빼고 "실측 필요"로만 적는다. 출처가 있는 사례 수치(특정 DLL 크기 등)만 예시로 남긴다.
3. **라이선스** → GPL 을 허용한다. 일반 libmpv 빌드와 FFmpeg eq·hqdn3d 같은 GPL 필터를 그대로 쓸 수 있다. LGPL 로 바꿀 때의 대체 수단은 `09-distribution-licensing.md` 에 짧게 적는다.
4. **PotPlayer 대응표** → 기능 이름을 기준으로 한다. 메뉴 경로·단축키는 넣지 않는다.

## 직접 확인한 출처 (2026-11-01)

- crates.io API: https://crates.io/api/v1/crates/libmpv2 , /egui_dock , /egui_ltreeview , /egui_tiles , /windows , /iced , /eframe 그리고 `/{crate}/{ver}/dependencies` (egui_dock 0.21.1, egui_tiles 0.18.0, egui_ltreeview 0.10.0·0.9.0, libmpv2 7.0.0)
- mpv 릴리스 목록: https://github.com/mpv-player/mpv/releases (최신 안정판 v0.41.0, 2025-12-21)
- mpv API 변경 이력: https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst
- mpv 안정판 render 헤더: https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h
- mpv 안정판 옵션 문서: https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst (`--hwdec`, `--hwdec-software-fallback`, `--d3d11va-zero-copy`, `--deinterlace`, `--brightness` 등)
- mpv 라이선스: https://github.com/mpv-player/mpv/blob/master/Copyright
- libmpv2 7.0.0 render 모듈: https://docs.rs/libmpv2/7.0.0/libmpv2/render/index.html
- wgpu 30.0.1 Features: https://docs.rs/wgpu/latest/wgpu/struct.Features.html
