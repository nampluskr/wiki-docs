---
tags: [rust, video-player, stack, egui, libmpv, windows]
created: 2026-11-01
updated: 2026-11-01
---

# 01. 스택 결정

> 이 파일을 읽는 때: 어떤 창·UI·렌더·디코딩 조합으로 갈지 정하거나, 그 결정을 다시 볼 때.

## 1. 결론

| 단계 | 스택 | 상태 |
| --- | --- | --- |
| **1단계 (지금)** | eframe 0.37 (glow, OpenGL) + egui_tiles 0.18 + egui_ltreeview 0.10 + **libmpv 0.41.0 안정판** (OpenGL render API) + libmpv2 7.0.0 | 출시된 API 만으로 구성할 수 있다 |
| **2단계 (0.42 출시 후)** | eframe/egui (wgpu, DX12) + libmpv 0.42 **D3D11 render API** + D3D11→D3D12 공유 텍스처 | mpv 0.42.0 은 미출시. Rust 래퍼도 없다 |
| 대안 1 | windows 크레이트 Win32 UI + libmpv `wid` 자식 창 (D3D11 출력) | 영상 위 UI 합성에 제약이 있다 |
| 대안 2 | winit + wgpu + egui + ffmpeg-next 직접 구현 | 플레이어 기능을 직접 만들어야 한다 |

이 단계안은 사용자가 결정했다(2026-11-01). 근거는 아래 3절에 있다.

## 2. UI 후보 비교 (2026-11-01 crates.io)

| 후보 | 버전 | 영상 + UI 합성 | 트리·탭 | 판단 |
| --- | --- | --- | --- | --- |
| **egui / eframe** | 0.37.0 | glow·wgpu paint callback 으로 탭 안에 영상을 그리고, 그 위에 UI 를 그린다 | egui_tiles 0.18, egui_ltreeview 0.10 (둘 다 egui 0.37) | **채택** |
| iced | 0.14.0 (2025-12-07) | wgpu Shader 위젯. 외부 하드웨어 텍스처를 넣는 사례는 미확인. 공식 문서에 experimental 표시 | PaneGrid 는 있다. 트리는 직접 만든다 | 후순위 |
| slint | 1.18.1 | rendering notifier·borrowed OpenGL texture API 가 있다 | 기본 위젯. 대형 트리·도킹은 미확인 | `.slint` DSL 이 "Rust 만" 조건에 맞지 않을 수 있다. 라이선스는 Royalty-free / GPLv3 / 상용 중에서 고른다 |
| makepad-widgets | 1.0.0 (2025-05-13) | Windows DX11 렌더러. 외부 텍스처 공개 API 는 미확인 | FileTree·Dock 모듈이 있다 | DSL 사용, 공개 crate 갱신 정체 |
| floem | 0.2.0 (2024-11) | 영상 표면 API 미확인 | 직접 구성 | 정체 |
| xilem | 0.4.0 | 영상 경로 미확인 | 미성숙 | 실험적 |
| winit + wgpu 직접 | 0.30.13 / 30.0.1 | 완전한 제어 | 위젯 없음 | UI 레이어가 따로 필요하다 |
| windows 크레이트 Win32 | 0.62.2 (2025-10-06) | D3D11/DComp 를 완전히 제어 | Common Controls TreeView/TabControl. 분할·도킹은 직접 만든다 | 대안 1 |

출처: crates.io 의 각 크레이트 페이지([eframe](https://crates.io/crates/eframe), [iced](https://crates.io/crates/iced), [slint](https://crates.io/crates/slint), [makepad-widgets](https://crates.io/crates/makepad-widgets), [floem](https://crates.io/crates/floem), [xilem](https://crates.io/crates/xilem), [winit](https://crates.io/crates/winit), [wgpu](https://crates.io/crates/wgpu), [windows](https://crates.io/crates/windows)). iced·slint·makepad 의 기능 서술은 sol 보고서 B2 를 따른다([iced widgets](https://docs.rs/iced/latest/iced/widget/), [slint Window](https://docs.rs/slint/latest/slint/struct.Window.html), [makepad 저장소](https://github.com/makepad/makepad), [Slint 라이선스](https://github.com/slint-ui/slint/blob/master/LICENSE.md?plain=1)).

## 3. 왜 1단계는 glow 인가

| 사실 | 출처 |
| --- | --- |
| mpv 최신 안정판은 v0.41.0 (2025-12-21) 이다 | [GitHub Releases](https://github.com/mpv-player/mpv/releases) |
| v0.41.0 render API 의 백엔드는 OpenGL 과 Software 둘뿐이다 | [render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h) |
| D3D11 render API(`MPV_RENDER_API_TYPE_D3D11`, API 2.6)는 "mpv 0.42.0" 항목에 있다 | [client-api-changes.rst](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst), [PR #17764](https://github.com/mpv-player/mpv/pull/17764) |
| libmpv2 7.0.0 render 모듈에는 OpenGL 타입(`OpenGLInitParams`, `FBO`)만 있다 | [docs.rs libmpv2 7.0.0](https://docs.rs/libmpv2/7.0.0/libmpv2/render/index.html) |
| egui_glow 0.37 의 콜백 타입은 `egui_glow::CallbackFn::new` 이고, eframe `CreationContext::get_proc_address` 로 GL 함수 resolver 를 얻을 수 있다 | [painter.rs 0.37.0](https://github.com/emilk/egui/blob/0.37.0/crates/egui_glow/src/painter.rs), [epi.rs 0.37.0](https://github.com/emilk/egui/blob/0.37.0/crates/eframe/src/epi.rs) (sol followup-1) |

정리하면, 출시된 mpv 로 앱이 소유한 표면에 그리는 방법은 OpenGL 뿐이다. 그리고 egui 가 OpenGL 로 그리는 길은 eframe 의 glow 백엔드다. 그래서 1단계는 glow 다.

### 3.1 1단계의 대가: 하드웨어 디코딩은 copy-back 이 된다

mpv `render_gl.h` 는 OpenGL render API 에서 하드웨어 디코딩을 직접 연결하려면 "Windows: ANGLE is required" 라고 적는다 ([render_gl.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h)). eframe 0.37 의 Windows 기본 경로는 WGL 을 먼저 쓰고 EGL 은 대체로 쓴다. ANGLE 을 고르는 공개 옵션은 없다 ([glow_integration.rs 0.37.0](https://github.com/emilk/egui/blob/0.37.0/crates/eframe/src/native/glow_integration.rs), sol followup-1).

그 결과(소스 판독, 실행 미확인):

| hwdec | eframe WGL 에서 |
| --- | --- |
| `d3d11va` (직접) | 쓸 수 없다. ANGLE/EGL 컨텍스트와 D3D11 장치가 필요하다 ([hwdec_d3d11egl.c](https://github.com/mpv-player/mpv/blob/v0.41.0/video/out/opengl/hwdec_d3d11egl.c)) |
| `d3d11va-copy` | 쓸 수 있다. GPU 로 디코딩한 뒤 시스템 RAM 으로 복사하고 GL 로 올린다 ([options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)) |
| `nvdec` (NVIDIA) | CUDA/GL interop 후보. NVIDIA 에서만 해당한다 ([hwdec_cuda_gl.c](https://github.com/mpv-player/mpv/blob/v0.41.0/video/out/hwdec/hwdec_cuda_gl.c)) |
| `auto` | 직접 모드를 먼저 시도하고, 그다음 `d3d11va-copy` 등 copy 모드를 시도한다(autoprobe 표에 copy 모드도 있다) ([vd_lavc.c v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/video/decode/vd_lavc.c)) |

**R1(ON/OFF·폴백)은 이 경로에서도 충족된다.** copy-back 도 하드웨어 디코딩이다. 다만 4K 고프레임에서는 복사 대역폭 때문에 CPU·전력 이점이 줄어들 수 있다. 얼마나 줄어드는지는 실측해야 한다. 반대로 copy-back 이면 프레임이 시스템 RAM 에 있으므로 CPU 필터(R2 의 lavfi)와 바로 함께 쓸 수 있다 ([options.rst `auto-copy`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)).

직접 interop 이 꼭 필요해지면 두 길이 있다. (a) eframe 대신 winit+glutin 으로 셸을 직접 만들어 EGL(ANGLE)을 강제한다. (b) 2단계(D3D11 render API)로 간다. (a)의 동작은 미확인이다 ([glutin DisplayApiPreference](https://docs.rs/glutin/0.32.3/glutin/display/enum.DisplayApiPreference.html)).

## 4. 2단계로 옮기는 조건과 방법

| 조건 | 확인할 곳 |
| --- | --- |
| mpv 0.42.0 안정판 출시 | [Releases](https://github.com/mpv-player/mpv/releases) |
| `render_d3d11.h` 가 안정판에 들어 있음 | [render_d3d11.h (master)](https://github.com/mpv-player/mpv/blob/master/include/mpv/render_d3d11.h) |
| libmpv2 가 D3D11 파라미터를 래핑함. 아니면 `libmpv2-sys` 수준에서 직접 FFI | [libmpv2](https://crates.io/crates/libmpv2) |
| D3D11 텍스처를 wgpu(DX12)로 가져오는 브리지의 PoC 성공 | [03-rendering-pipeline.md](03-rendering-pipeline.md) |

PR #17764 의 리뷰어는 Windows D3D11 에서는 창 임베딩이 더 나을 수 있고 render API 에는 제한이 있다고 했다. 다만 그 제한이 무엇인지는 적지 않았다 ([PR #17764](https://github.com/mpv-player/mpv/pull/17764), sonnet 보고서). 그래서 2단계로 넘어가기 전에 다시 조사한다.

옮길 때 바뀌는 것은 `render-gl` 크레이트를 `render-wgpu` 로 바꾸는 일과 eframe 렌더러 설정뿐이다. `player-api` 와 UI 코드는 그대로 둔다([04-player-core.md](04-player-core.md)).

## 5. 대안 스택

| 대안 | 좋은 점 | 대가 | 언제 고르나 |
| --- | --- | --- | --- |
| Win32 + libmpv `wid` | mpv 가 D3D11 스왑체인을 직접 관리한다. `d3d11va` 직접 디코딩과 `d3d11vpp` 를 쓸 수 있다 ([options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst), [vf.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst)) | 자식 HWND 위에 부모 UI 를 그릴 수 없다(airspace). 분할·도킹·DPI·포커스를 직접 만든다. mpv 도 window embedding 의 문제를 경고한다 ([render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)) | HDR·직접 디코딩이 필수가 될 때 |
| wgpu + ffmpeg-next | 디코더 텍스처와 셰이더를 완전히 통제한다 | A/V 동기·탐색·ASS 자막·오디오·R1 재초기화를 모두 직접 만든다. ffmpeg-next 유지자는 새 API 포팅을 보장하지 않는다 ([ffmpeg-next](https://docs.rs/crate/ffmpeg-next/latest)) | 디코더 제어 자체가 제품의 핵심일 때 |
| iced + GStreamer | GStreamer 의 Rust API·Windows 배포 문서가 잘 되어 있다 ([gstreamer-rs](https://docs.rs/gstreamer/latest/gstreamer/)) | 플러그인 배포, caps 협상, iced 커스텀 위젯 비용 | 파이프라인 구성이 필요할 때 |

## 6. 스택 비교표

| 스택 | 개발 기간 | 성능 | R1 | R2 | 배포 크기 | 위험 |
| --- | --- | --- | --- | --- | --- | --- |
| 1단계 glow + libmpv 0.41 | 실측 필요 | HW 는 copy-back(일부 NVIDIA 제외). 실측 필요 | mpv `hwdec` | mpv 속성 + lavfi + GLSL | 실측 필요 | copy-back 비용, eframe GL 콜백과 mpv 스레드 규칙 |
| 2단계 wgpu + libmpv 0.42 | 실측 필요 | 직접 디코딩 기대(검증 전) | mpv `hwdec` | 같음 | 실측 필요 | 미출시 API, D3D11↔D3D12 브리지 |
| Win32 + `wid` | 실측 필요 | mpv D3D11 출력 | mpv `hwdec` | mpv 내부 + `d3d11vpp` | 실측 필요 | UI 합성 제약, unsafe COM 코드 |
| wgpu + FFmpeg 직접 | 가장 김(세 보고서 공통) | 브리지 성공 여부에 달림 | 직접 구현 | 직접 구현(WGSL) | 실측 필요 | 구현량, wgpu-hal 메이저 변경 |

기간·크기 수치는 넣지 않는다(사용자 결정). 출처가 있는 사례 수치는 [09-distribution-licensing.md](09-distribution-licensing.md) 에 있다. UI 툴킷을 바꾼다고 R1/R2 를 책임지는 쪽이 바뀌지는 않는다. R1/R2 는 언제나 엔진이 맡는다.

## 7. 장단점과 최종 선택

### 7.1 후보별 장단점

| 후보 | 장점 | 단점 |
| --- | --- | --- |
| **1단계: eframe+glow + libmpv 0.41 (OpenGL render API)** | 출시된 API 만 쓴다([render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)). 영상과 UI 가 한 GL 장면이라 탭 안 영상 위에 OSD 를 그릴 수 있다. R1·R2·자막·트랙을 mpv 가 준다. Rust 한 언어, 한 프로세스. egui_tiles·egui_ltreeview 가 egui 0.37 과 맞는다 | WGL 에서는 직접 HW interop 이 안 되어 HW 디코딩이 대부분 copy-back 이 된다([render_gl.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h)). `d3d11vpp` 를 못 쓴다. UI 스레드가 렌더 스레드를 겸하므로 mpv 스레드 규칙을 엄격히 지켜야 한다. HDR 은 미확인 |
| **2단계: eframe/egui wgpu(DX12) + libmpv 0.42 D3D11 render API** | 앱이 만든 D3D11 장치를 mpv 가 쓰므로 `d3d11va` 직접 디코딩을 기대할 수 있다. 영상 위 UI 합성은 1단계와 같다. UI·코어 코드를 그대로 두고 렌더 크레이트만 바꾼다 | 0.42.0 은 미출시이고 Rust 래퍼도 없다([client-api-changes.rst](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst), [libmpv2 7.0.0](https://docs.rs/libmpv2/7.0.0/libmpv2/render/index.html)). D3D11→D3D12→wgpu-hal 브리지(unsafe, 동기화, wgpu 버전 고정)가 필요하다. 리뷰어가 말한 "제한"의 내용은 미확인이다([PR #17764](https://github.com/mpv-player/mpv/pull/17764)) |
| **대안: Win32 + libmpv `wid`** | mpv 가 D3D11 스왑체인을 관리해서 직접 디코딩·`d3d11vpp`·HDR 옵션에 접근하기 쉽다([options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)). 지금 출시된 mpv 로 된다 | 자식 HWND 위에 UI 를 그릴 수 없다(airspace). 트리·분할·도킹·다크 모드·DPI·포커스를 Win32 로 직접 만든다. unsafe COM 코드가 많다. mpv 도 임베딩의 문제를 경고한다([render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)) |
| **대안: wgpu + egui + ffmpeg-next 직접** | 디코더 텍스처·색 변환·셰이더를 완전히 통제한다. wgpu NV12/P010 피처가 있다([wgpu Features](https://docs.rs/wgpu/latest/wgpu/struct.Features.html)) | A/V 동기·탐색·오디오·ASS 자막·R1 재초기화를 모두 직접 만든다. 세 보고서 모두 구현량이 가장 크다고 봤다. ffmpeg-next 는 새 API 포팅을 보장하지 않는다([ffmpeg-next](https://docs.rs/crate/ffmpeg-next/latest)) |

공통 위험: GPU·드라이버마다 하드웨어 디코딩 결과가 다르다. libmpv DLL 의 배포·ABI 문제도 있다([09-distribution-licensing.md](09-distribution-licensing.md)). UI 를 바꿔도 R1/R2 는 엔진의 일이다.

### 7.2 최종 선택 — 1단계로 시작, 0.42 출시 후 2단계

사용자 확정(2026-11-01). W1 확인: 1단계 copy-back 비용은 실측한 뒤 다시 검토한다.

| 단계 | 할 일 | 끝내는 조건 |
| --- | --- | --- |
| 0. PoC | eframe glow 에서 탭 하나에 libmpv 영상을 그리고 OSD 를 얹는다. `hwdec` 를 토글하고 `hwdec-current` 를 기록한다 ([10](10-validation-and-open-questions.md) P1·P2) | 영상·OSD 가 나오고, stuck 로그 없이 R1 토글이 된다. 막히면 대안(Win32 + `wid`)을 검토한다 |
| 1. 1단계 구현 | 코어·탭·트리·R1·R2·재생 기능 ([04](04-player-core.md)~[08](08-playback-features.md)) | [00-requirements.md](00-requirements.md) 의 R1·R2 인수 기준과 P3·P4 를 통과한다 |
| 1.5 copy-back 재검토 | 4K·고프레임 corpus 로 CPU·전력·drop 을 잰다 (Q4) | 허용 범위면 1단계를 유지한다. 아니면 ANGLE 강제 셸(Q5) 또는 2단계를 앞당긴다 |
| 2. 2단계 전환 | mpv 0.42.0 안정판이 나오면 D3D11 render API + wgpu 브리지 PoC 를 만들고 `render-wgpu` 로 교체한다 (Q1·Q2·Q3·Q15) | 0.42 출시, 래퍼 또는 FFI 준비, 브리지 PoC 성공, 1단계와 같은 검증 통과 |
| 예비 | 디코더 통제가 제품의 핵심이 되면 FFmpeg 직접 경로를 다시 본다 | 요구 변경 시 |

이유:
- 출시된 mpv 로 앱 소유 표면에 그리는 방법은 OpenGL 뿐이다. 그래서 지금 바로 시작할 수 있는 길은 glow 다.
- 영상 위 UI(탭 안 OSD)는 1·2단계 모두 된다. Win32 `wid` 로는 안 된다.
- R1 은 copy-back 이어도 충족된다(ON/OFF·폴백·상태 표시). 비용만 실측하면 된다.
- 렌더만 갈아 끼우도록 크레이트를 나누었으므로([04-player-core.md](04-player-core.md)), 2단계로 옮기는 비용은 렌더 크레이트에 한정된다.
- FFmpeg 직접 구현은 같은 기능을 얻는 데 드는 구현량이 가장 크다.

## 8. 버전 고정 표 (2026-11-01)

| 크레이트 | 버전 | 비고 |
| --- | --- | --- |
| eframe / egui / egui_glow | 0.37.0 | glow feature |
| egui_tiles | 0.18.0 | egui ^0.37.0 |
| egui_ltreeview | 0.10.0 | egui ^0.37 |
| libmpv2 | 7.0.0 (2026-10-30) | libmpv2-sys ^4.0.1 |
| libmpv (DLL) | 0.41.0 안정판 API 기준 | 0.41 API 에 맞춘다 |
| rfd / notify | 0.17.2 / 8.2.1 | argon followup-1 재확인 |

출처: crates.io API (`/api/v1/crates/<이름>` 와 `/dependencies`). cargo 빌드는 하지 않았다.

---

이전: [00-requirements.md](00-requirements.md) · 다음: [02-decoding-engine.md](02-decoding-engine.md)
