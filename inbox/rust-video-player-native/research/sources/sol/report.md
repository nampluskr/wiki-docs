1. 원점 권장 1순위는 Rust UI인 egui/eframe의 glow 경로 + libmpv 안정판이다. 트리·탭 재사용과 영상 위 UI 합성이 쉽다는 설계 판단이다. [eframe](https://docs.rs/eframe/latest/eframe/), [render API](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)
2. Windows 영상 출력·HDR 우선의 대안은 windows 크레이트 + Win32 트리·탭 + libmpv의 D3D11 자식 창이다. 오버레이 합성 비용을 별도로 감수한다. [Win32](https://learn.microsoft.com/en-us/windows/win32/Controls/tab-controls), [mpv](https://mpv.io/manual/stable/)
3. wgpu는 UI·셰이더에 좋지만 D3D11 디코더 텍스처를 바로 받는 범용 고수준 API로 보아서는 안 된다. [wgpu Device](https://docs.rs/wgpu/latest/wgpu/struct.Device.html)
4. libmpv 0.41.0 안정판 render API는 OpenGL/SW이며, D3D11 render API는 개발 계열의 0.42.0/API 2.6 항목이다. [안정판 헤더](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h), [변경 기록](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst)
5. R1은 mpv의 재생 중 hwdec 변경과 SW fallback을 활용하되, 요청 상태와 실제 hwdec-current를 따로 표시한다. [옵션](https://mpv.io/manual/stable/), [상태](https://github.com/mpv-player/mpv/blob/master/DOCS/man/input.rst)
6. R2는 색 조정·공간 필터·시간 필터를 나눠 구현한다. GPU 셰이더와 CPU lavfi 필터의 복사·성능 차이를 명시해야 한다. [셰이더](https://mpv.io/manual/stable/), [필터](https://www.ffmpeg.org/ffmpeg-filters.html)
7. eframe 0.37.0, egui_dock 0.21.1, egui_ltreeview 0.10.0을 무조건 함께 쓰면 안 된다. dock은 egui 0.36, 최신 tree는 0.37 의존이다. [dock](https://docs.rs/crate/egui_dock/latest), [tree](https://docs.rs/crate/egui_ltreeview/latest)
8. 숨김 탭은 기본 일시 정지·디코더 예산 제한을 권한다. 다중 재생은 별도 mpv core와 오디오 정책이 필요하다는 설계 제안이다. [core별 render context](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)
9. FFmpeg 직접 구현은 가장 큰 통제권을 주지만 A/V 동기·ASS·오디오·탐색까지 플레이어를 직접 만드는 비용이 든다는 추정이다. [FFmpeg API](https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h_source.html)
10. 확인 기준일은 2026-11-01이다. 웹 문서 조사만 수행했으며 Windows 11/MSVC 실행·GPU 복사 계측·배포 DLL 호환성은 미확인이다.

# Rust 동영상 플레이어 원점 설계 조사 — rvpn-scouter-sol

기준: Windows 11, Rust stable-msvc, 2026-11-01 확인. 기존 앱·다른 문서 묶음·다른 scouter 결과는 참고하지 않았다. 본문의 **제안/판단/추정**은 조사자의 설계 의견이며 검증된 구현 결과가 아니다. 버전은 웹에서 관측한 값이고, `latest`·`master`는 이후 변할 수 있다. 설치·빌드·설정 변경은 하지 않았다.

## B1 원점 권장 스택 1~2개와 이유

**A — 기본 권장: eframe/egui + glow(OpenGL) + libmpv 0.41.0 + libmpv2 바인딩.** eframe은 glow와 wgpu를 선택할 수 있고, 안정판 libmpv의 GPU render API는 OpenGL이다. 이 공통 API를 고르면 D3D11→D3D12→wgpu 브리지를 첫 버전에 만들 필요가 없다는 판단이다. 앱 로직·UI 코드는 Rust로 작성한다. [eframe](https://docs.rs/eframe/latest/eframe/), [mpv 안정판 헤더](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h), [바인딩](https://docs.rs/libmpv2/latest/libmpv2/)

제안 조합은 **eframe/egui 0.36.2 + egui_dock 0.21.1 + egui_ltreeview 0.9.0**이다. 최신 eframe은 0.37.0이지만 dock의 egui 의존은 ^0.36이며, tree 0.9 README는 egui 0.36을 명시한다. 0.37을 원하면 탭을 직접 작성하거나 dock 포팅을 별도 결정한다. Cargo 해석·컴파일 검증은 미확인이다. [eframe 버전](https://docs.rs/crate/eframe/latest), [dock 의존성](https://docs.rs/crate/egui_dock/latest), [tree 0.9 호환 기록](https://docs.rs/crate/egui_ltreeview/latest/source/README.md)

선택 이유는 전체 화면보다도 **탭 안 영상과 트리·탭·슬라이더·오버레이를 한 UI 장면으로 합성하는 비용**을 낮출 수 있기 때문이다(판단). glow paint callback에서 libmpv를 같은 GL context의 FBO에 그리게 하고 이후 UI를 합성하는 설계다. GL context 소유권·렌더 상태·콜백 재진입은 직접 관리해야 하며 eframe이 완성된 영상 위젯을 주는 것은 아니다. [egui_glow callback](https://docs.rs/egui_glow/latest/egui_glow/painter/index.html), [mpv render threading](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)

**B — Windows 출력 우선 대안: windows 0.62.2 + Win32 Common Controls + libmpv 0.41.0의 wid 자식 HWND + vo=gpu/gpu-api=d3d11.** 네이티브 트리·탭과 D3D11 영상 출력을 분리한다. Rust 측 자체 D3D11/DirectComposition 합성기는 필요가 생긴 단계에 도입하는 제안이다. [windows](https://docs.rs/windows/latest/windows/), [TreeView](https://learn.microsoft.com/en-us/windows/win32/Controls/tree-view-controls), [탭](https://learn.microsoft.com/en-us/windows/win32/Controls/tab-controls), [mpv 출력 옵션](https://mpv.io/manual/stable/)

B는 D3D11 출력·HDR 설정에 접근하기 쉽지만(판단), 자식 HWND는 부모 UI 텍스처 안에 자동으로 들어가지 않는다. 영상 위 반투명 컨트롤·도킹 드래그·클리핑·DPI·포커스 처리를 합성/별도 창으로 설계해야 한다. render API 문서 자체도 일반적인 window embedding의 문제를 경고하며 render API를 권한다. 따라서 HDR이 필수가 아닌 현재 범위에는 A를 앞세운다. [mpv embedding 경고](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h), [DirectComposition swapchain](https://learn.microsoft.com/en-us/windows/win32/direct3ddxgi/dxgi-1-2-presentation-improvements)

핵심 분기: mpv 개발판에는 **앱 소유 D3D11Device/Texture2D로 그리는 render API**가 추가되어 있다. 이는 B의 오버레이 문제를 줄일 유력 경로지만, 확인일 공식 release 목록의 안정판은 0.41.0이고 D3D11 API는 0.42.0 항목이다. 안정판에 이미 들어갔다고 쓰면 안 된다. [개발 헤더](https://github.com/mpv-player/mpv/blob/master/include/mpv/render_d3d11.h), [API 이력](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst), [release 목록](https://github.com/mpv-player/mpv/releases)

## B2 창·UI 후보 비교

다음 표의 적합성·성숙도는 공식 API와 문서로 판단한 상대 평가다. 모든 버전 관측일은 **2026-11-01**이며, 최신 출시 여부와 실제 해당 기능의 동작 검증을 구분한다.

| 후보·관측 버전 | 영상·swapchain·타이밍·전체 화면 | 트리·탭·분할 | HDR·영상 위 UI·유지 판단 및 근거 |
|---|---|---|---|
| winit 0.30.13 + wgpu 30.0.1 직접 | 창/이벤트와 렌더 루프를 직접 소유. fullscreen API 사용 가능, vsync·PTS 스케줄은 직접 설계 | 위젯 없음. Rust UI 라이브러리를 추가하거나 직접 구현 | 동일 GPU 장면의 오버레이는 용이(판단), HDR는 별도 출력 색공간 통합 필요. 양쪽 버전 업데이트 추적 필요. [winit](https://docs.rs/winit/latest/winit/), [wgpu](https://docs.rs/crate/wgpu/latest), [surface](https://docs.rs/wgpu/latest/wgpu/type.SurfaceConfiguration.html) |
| egui/eframe 0.37.0; 제안 pin 0.36.2 | winit 기반, 기본 wgpu/선택 glow. callback으로 영상 합성, 재생 콜백에서 repaint 요청하는 설계 | dock 0.21.1로 탭·분할, tree 0.9/0.10로 키보드·다중 선택 | 오버레이 우수(판단); HDR end-to-end 보장은 미확인. 최신 UI와 위젯 semver 불일치 주의. [eframe](https://docs.rs/crate/eframe/latest), [dock](https://docs.rs/crate/egui_dock/latest), [tree](https://docs.rs/crate/egui_ltreeview/latest) |
| iced 0.14.0 | wgpu Shader 위젯과 redraw 스케줄 API가 있어 사용자 렌더링 후보. present mode 설정 경로 존재 | PaneGrid 제공. 완성된 파일 탐색기 트리·도킹 탭 세트는 미확인, 모델/위젯 직접 구성 | GPU 장면 내 오버레이 가능(판단), HDR 미확인. 공식 문서가 experimental 명시. [iced](https://docs.rs/crate/iced/latest), [widgets](https://docs.rs/iced/latest/iced/widget/), [present mode](https://docs.rs/iced_wgpu/latest/iced_wgpu/settings/fn.present_mode_from_env.html) |
| slint 1.18.1 | rendering notifier·borrowed OpenGL texture·fullscreen API 존재 | 기본 컨트롤/모델 활용 가능. 대규모 트리·IDE 도킹 패키지 성숙도 미확인 | GL 영상과 합성 가능성은 있으나 PoC 필요. wgpu 확장은 unstable 버전별 API. .slint markup을 Rust macro 안에 넣어도 별도 DSL이므로 ‘Rust 문법만’의 엄격 해석에는 부적합. [API](https://docs.rs/slint/latest/slint/), [Window](https://docs.rs/slint/latest/slint/struct.Window.html), [버전](https://docs.rs/crate/slint/latest) |
| floem 0.2.0 | 공식 저장소는 wgpu/vger/vello 또는 AnyRender Skia 렌더링 설명. 영상 외부 텍스처 합성·프레임 제어의 안정 API는 미확인 | virtual list/stack으로 트리 행 구성 가능. Lapce가 사용하지만 Lapce의 탐색기를 범용 위젯으로 바로 재사용 가능하다는 근거는 없음 | HDR/자식 HWND/영상 callback 미확인. 앱과 공개 crate·개발 브랜치의 차이를 확인할 필요(판단). [floem](https://docs.rs/floem/latest/floem/), [views](https://docs.rs/floem/latest/floem/views/index.html), [저장소](https://github.com/lapce/floem), [Lapce](https://github.com/lapce/lapce) |
| makepad-widgets 1.0.0 | 저장소는 Windows DX11 렌더링 명시. video 모듈 존재만으로 FFmpeg/mpv 범용 재생 보장 불가 | FileTree·Dock·splitter·tab 관련 모듈 존재. 후보 중 구조적 적합성 높음(판단) | HDR·외부 D3D11 텍스처 공개 API·R1/R2 연결은 미확인. 공개 crate는 2025-05-13, git 최신과 차이 주의. live_design DSL 사용 범위도 Rust-only 해석 확인 필요. [저장소](https://github.com/makepad/makepad), [버전](https://docs.rs/crate/makepad-widgets/latest), [모듈](https://docs.rs/makepad-widgets/latest/makepad_widgets/index.html) |
| xilem 0.4.0 | GPU UI 개발 후보이나 영상 전용 표면·타이밍·HDR 완성 경로 미확인 | 트리·도킹·탭의 완성 묶음 미확인; 직접 구성 비용 예상 | 저장소가 experimental 명시, Apache-2.0, 문서 MSRV 1.88. 이번 일정에는 후순위(판단). [crate](https://docs.rs/crate/xilem/latest), [저장소](https://github.com/linebender/xilem) |
| windows 0.62.2 + Win32+D3D11/DirectComposition 직접 | flip swapchain·waitable frame latency·출력 색공간을 직접 관리. 전체 화면 정책·장치 손실도 앱 책임 | Common Controls의 TreeView/TabControl은 오래된 Windows API. 분할/닫기/드래그 도킹은 추가 구현 | HDR 제어 범위 넓음(판단). 같은 D3D11 렌더 타깃 또는 DComp visual 합성으로 오버레이. 낮은 추상화·unsafe/COM 수명 관리 비용. [windows](https://docs.rs/windows/latest/windows/), [flip](https://learn.microsoft.com/en-us/windows/win32/direct3ddxgi/for-best-performance--use-dxgi-flip-model), [HDR](https://learn.microsoft.com/en-us/windows/win32/direct3darticles/high-dynamic-range), [트리](https://learn.microsoft.com/en-us/windows/win32/Controls/tree-view-controls), [탭](https://learn.microsoft.com/en-us/windows/win32/Controls/tab-controls) |

추가 후보는 **eframe 대신 winit+glutin+egui_glow를 직접 조립**하는 방식이다. eframe은 앱 셸 작업을 줄이고, 직접 조립은 render thread/context 소유권을 세밀하게 정할 수 있다는 판단이다. 공통 GL 기반을 유지하므로 libmpv OpenGL API와의 연결은 같은 원리다. 별도 패키지 버전 묶음·MSVC 빌드 검증은 미확인이다. [eframe이 안내하는 대안](https://docs.rs/crate/eframe/latest), [mpv render](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)

공통 주의: **vsync 지원 ≠ 영상 PTS·오디오 동기 보장**이다(설계 판단). wgpu의 Fifo는 보장되지만 나머지 present mode는 지원 조회가 필요하며, desired_maximum_frame_latency는 힌트다. 제안은 UI 입력 시와 새 영상 프레임 시만 redraw하고, 정지 화면에서 무제한 GPU 루프를 피하는 것이다. [wgpu surface 설정](https://docs.rs/wgpu/latest/wgpu/type.SurfaceConfiguration.html)

## B3 렌더링

### wgpu와 D3D11 직접의 선택

wgpu는 휴대 가능한 render/compute pass·텍스처·WGSL 파이프라인을 제공하며 backend별 unsafe HAL 가져오기 API가 있다. 확인한 `create_texture_from_hal` backend 목록은 Vulkan/Metal/Dx12/Gles이며 **D3D11 HAL은 없다**. 따라서 `ID3D11Texture2D*`를 그대로 `wgpu::Texture`로 바꾸는 것으로 설명하면 틀린다. [wgpu Device](https://docs.rs/wgpu/latest/wgpu/struct.Device.html)

D3D11 직접은 디코더와 렌더러의 device를 맞추기 쉬운 Windows 설계다(판단). FFmpeg D3D11 HW context는 사용자 device와 frame BindFlags/MiscFlags를 다루며, 포맷에 따라 shader-resource 바인딩 제한이 있다. **디코드 표면 직접 샘플링·GPU 복사 후 샘플링·CPU 왕복**을 각각 구분해야 한다. [FFmpeg D3D11 context](https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h_source.html)

### ‘공유 텍스처 → wgpu’에서 실제 필요한 것

다음은 구현 제안이지 즉시 사용 가능한 안정 Rust 통합 레시피가 아니다. D3D11 측 공유 가능한 출력 텍스처를 만들고, NT shared handle을 D3D12에서 열어, 호환되는 wgpu-hal Dx12 텍스처로 감싼 뒤 `create_texture_from_hal`로 가져온다. 동일 어댑터·포맷·usage·수명·소유권·D3D12 상태·fence 동기화를 모두 맞춰야 한다. wgpu가 내부 제출을 소유하므로 외부 큐 동기화 설계도 필요하다. [Microsoft D3D12 interop](https://learn.microsoft.com/en-us/windows/win32/direct3d12/direct3d-12-with-direct3d-11--direct-2d-and-gdi), [shared heaps](https://learn.microsoft.com/en-us/windows/win32/direct3d12/shared-heaps), [wgpu HAL 경계](https://docs.rs/wgpu/latest/wgpu/struct.Device.html)

디코더가 준 NV12/P010 texture array가 그대로 공유·샘플링 가능한지는 별개다. 공유 RGBA 중간 타깃으로 GPU 변환/복사가 필요할 수 있으므로, 이 경로의 **CPU 왕복 없음**과 **GPU 복사 없음**은 동의어가 아니다. mpv D3D11 출력도 기본적으로 decoder surface→shader resource GPU 복사를 하며, `d3d11va-zero-copy`는 드라이버/패딩 문제 가능성을 문서화한다. 무복사를 제품 필수 조건으로 삼지 않는 편을 권한다. [mpv zero-copy 설명](https://mpv.io/manual/stable/), [FFmpeg texture pool 제약](https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h_source.html)

### YUV→RGB와 R2 위치

직접 렌더러를 택할 경우의 제안 순서는 `decode → deinterlace/temporal denoise → spatial denoise → YUV range/matrix 및 transfer 처리 → RGB 효과/scale → tone map/output transform → subtitle → UI`다. 실제 필터에 따라 YUV/RGB 단계는 달라진다. 범위(full/limited), BT.601/709/2020, chroma 위치, P010 유효 비트, PQ/HLG를 명시적 FrameMetadata로 운반해야 한다는 설계다. Advanced Color는 swapchain 포맷뿐 아니라 색공간·desktop HDR 상태까지 다룬다. [Microsoft HDR 지침](https://learn.microsoft.com/en-us/windows/win32/direct3darticles/high-dynamic-range)

A에서는 색변환·자막·스케일링을 mpv에 맡기고 GLSL hook/영상 필터를 사용한다. B의 안정판 자식 창도 R2는 mpv 내부에 둔다. wgpu 또는 D3D11 직접 렌더러라면 WGSL/HLSL pass에 두되 UI 이후에 효과를 적용해 글자까지 흐리게 만들지 않는 설계다. shader 언어는 GPU 자산이고 Rust 앱 로직과 구분한다. 이를 포함해 모든 소스 문법까지 Rust만 요구한다면 사용자 정의 shader 작성 자체가 제약이 되므로 내장 엔진 필터를 우선한다. [mpv shader hook](https://mpv.io/manual/stable/), [영상 필터](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst)

HDR은 현재 필수 기능 목록에는 없지만 후보 평가에는 포함했다. A의 GL+eframe 합성에서 10bit/HDR 출력·SDR UI 밝기·다중 모니터 전환까지 검증된 경로는 **미확인**이다. B의 mpv D3D11은 output format/csp 옵션이 있으나 옵션 존재만으로 목표 PC HDR 품질이 검증된 것은 아니다. [mpv D3D11 출력 옵션](https://github.com/mpv-player/mpv/blob/master/DOCS/man/options.rst), [HDR 원칙](https://learn.microsoft.com/en-us/windows/win32/direct3darticles/high-dynamic-range)

## B4 디코딩 바인딩 선택과 통합 방법

| 엔진 | 연결 방식 | 선택 판단·남는 일 | 출처 |
|---|---|---|---|
| libmpv; Rust libmpv2 | A는 OpenGL render API→FBO/texture→egui 장면. B는 `wid`에 자식 HWND 전달 후 엔진이 D3D11 출력. 일반 API의 명령·property·event로 제어 | 범용 플레이어 기능을 가장 많이 재사용하므로 1순위(판단). 최신 D3D11 render API의 Rust wrapper 커버리지는 미확인 | [libmpv2](https://docs.rs/libmpv2/latest/libmpv2/), [wrapper render 목록](https://docs.rs/libmpv2/latest/libmpv2/render/index.html), [안정판 render](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h) |
| FFmpeg; ffmpeg-next 9.0.0 + 필요 시 ffmpeg-sys 계열 | demux/send packet/receive frame, D3D11 AVHWDevice/AVHWFrames, CPU면 plane upload, GPU면 texture 전달. 렌더러와 오디오는 앱이 작성 | 복사 경로·필터 완전 통제에 적합. 완성 플레이어 아님. 안전 wrapper에 없는 HW API는 좁은 unsafe FFI 모듈 필요(설계). 유지자가 새 API 포팅은 보장하지 않는다고 명시 | [ffmpeg-next](https://docs.rs/crate/ffmpeg-next/latest), [D3D11 context](https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h_source.html) |
| GStreamer; gstreamer-rs 0.25.4 | playbin3로 재생/트랙/자막·bus 관리. d3d11videosink+GstVideoOverlay에 HWND 전달, 또는 appsink로 D3D11Memory/CPU sample을 받아 직접 합성 | Rust API와 Windows MSVC 배포 경로가 잘 문서화된 대안. 동적 decoder/filter 교체·caps/device negotiation·플러그인 배포가 추가 비용(판단) | [Rust](https://docs.rs/gstreamer/latest/gstreamer/), [playbin3](https://gstreamer.freedesktop.org/documentation/playback/playbin3.html), [D3D11 sink](https://gstreamer.freedesktop.org/documentation/d3d11/d3d11videosink.html), [Windows](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html) |

확인일 libmpv2 root 문서는 **6.0.1**, 검색·render 하위 문서는 **6.0.0**으로 서로 다르게 관측됐다. 버전 확정은 미확인으로 두고 의존성 lock 전에 재확인해야 한다. wrapper에는 OpenGLInitParams/FBO가 보이며 이것만으로 D3D11 enum 지원까지 추론하지 않는다. [root](https://docs.rs/libmpv2/latest/libmpv2/), [render](https://docs.rs/libmpv2/latest/libmpv2/render/index.html)

libmpv의 render API는 **디코드 원본 프레임을 앱에 주는 API가 아니라 엔진이 앱의 타깃에 그리는 API**다. decode-only AVFrame 파이프라인을 원하면 FFmpeg/GStreamer로 설계해야 한다는 판단이다. mpv는 한 core당 render context 하나이며 여러 client handle 생성과 여러 독립 플레이어 생성은 구분한다. [render 헤더](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)

GStreamer의 d3d11videosink 문서에는 공유 텍스처 접근용 과거 경로 대신 appsink 권장이 있다. `appsink`에 도달했다고 CPU 프레임이라고 가정하지 말고 negotiated memory caps를 확인한다. 렌더할 sample 수명을 유지하고 streaming thread에서 UI 작업을 하지 않는 설계가 필요하다. [sink](https://gstreamer.freedesktop.org/documentation/d3d11/d3d11videosink.html), [appsink](https://gstreamer.freedesktop.org/documentation/app/appsink.html)

MSVC 주의: Rust stable-msvc와 DLL의 C ABI 연결을 분리해 검증한다. 공식 mpv 개발 배포에는 Windows libmpv DLL/import library/headers와 GPL/LGPL 변형이 있으나 선택한 안정 DLL·import library가 이 Rust target에서 링크되는지는 미확인이다. GStreamer는 Rust toolchain과 MinGW/MSVC 패키지를 맞추도록 문서화한다. [mpv 배포](https://github.com/mpv-player/mpv/releases), [GStreamer Rust Windows](https://docs.rs/gstreamer/latest/gstreamer/)

## B5 파일 탐색기 + 탭 UI 구현

다음은 A를 기준으로 한 설계 제안이다. `ExplorerModel`과 `TabModel`을 위젯 상태와 분리하고, 트리 노드 활성화 시 `OpenMedia(PathBuf)`를 보내 새 `SessionId`를 가진 탭을 만든다. UI에는 경로 문자열 대신 ID를 붙여 rename·정렬·비동기 열거 시 선택이 엉키지 않게 한다.

| 영역 | 재사용·설계 | 근거 |
|---|---|---|
| 탐색기 | egui_ltreeview의 행·선택·키보드·D&D 사용. 디렉터리 열거/권한 오류/정렬/순환 링크/확장자는 앱 책임. 열린 노드만 비동기 열거하는 제안 | [tree 기능](https://docs.rs/egui_ltreeview/latest/egui_ltreeview/) |
| 탭·분할 | DockState<SessionId>, TabViewer, DockArea 사용. 첫 버전은 외부 native window로 탭 떼어내기 제한 권장: GL context·mpv render context 이동 비용 때문(판단) | [dock](https://docs.rs/egui_dock/latest/egui_dock/), [GL context 조건](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h) |
| 영상 표면 | 탭 content rect를 물리 픽셀로 변환, 종횡비 보존 viewport/FBO, 그 위에 타임라인·컨트롤. 마지막 프레임과 로딩/오류 상태를 분리하는 제안 | [egui callback](https://docs.rs/egui_glow/latest/egui_glow/painter/index.html) |
| 파일 열기·갱신 | rfd를 선택 파일 대화상자에, notify를 폴더 변경 관측에 검토. 내장 탐색기 자체를 rfd로 대체하는 것은 아님. 재사용 버전 조합은 미확인 | [rfd](https://docs.rs/rfd/latest/rfd/), [notify](https://docs.rs/notify/latest/notify/) |
| Windows 네이티브 B | WC_TREEVIEW와 WC_TABCONTROL, 탭 client rect 안 자식 영상 HWND. 숨김·크기·focus를 탭 선택과 동기화. splitter/닫기/재정렬 직접 구현 | [트리](https://learn.microsoft.com/en-us/windows/win32/Controls/tree-view-controls), [탭](https://learn.microsoft.com/en-us/windows/win32/Controls/tab-controls) |

탭 정책 제안: 기본은 **보이는 주 탭 1개만 재생, 숨김은 pause**다. 최근 위치·트랙·필터는 보존하고, 메모리 예산을 넘는 숨김 세션은 닫은 뒤 복원한다. ‘음악처럼 백그라운드 재생’과 ‘분할 화면 여러 영상 동시 재생’은 명시적 옵션으로 구분한다. 이는 사용자 경험·GPU 세션 예산을 위한 제안이며 엔진의 필수 제한이 아니다.

동시 재생을 허용하면 독립 파일별 mpv core, decode queue, 볼륨/mute, renderer 상태를 둔다. 기본 audible session 하나를 선택하고, 다중 오디오 믹싱은 별도 기능으로 둔다. 숨김 render API context를 방치하지 말고 업데이트/skip rendering을 처리하거나 pause/해제한다. pause 자체가 할당된 HW decoder texture pool을 모두 반환한다는 가정은 금지한다. [core/context와 skip rendering](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h), [명령·property](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst)

4K 8bit RGBA 한 장은 산술상 약 31.6 MiB이며(3840×2160×4/2^20), 여러 프레임·다중 탭·필터 이력·디코더 pool은 별도다. 따라서 탭 개수만으로 메모리를 추정하지 말고 활성 surface 수와 필터 이력을 예산화한다(설계 제안).

## B6 플레이어 코어와 UI 분리 구조

아래는 제안 crate 구성이다. UI 프레임워크 타입과 엔진 포인터를 공용 프로토콜로 노출하지 않는다.

```text
app-egui / app-win32   창·입력·트리·탭·레이아웃·전체 화면 정책
player-api            SessionId, Command, Event, State, Capabilities
player-mpv            바인딩·명령·property 관찰·엔진 수명
render-gl             GL context/FBO·mpv render API·paint callback
render-win            HWND·D3D11/DComp 경계(대안 구현)
library-fs            비동기 디렉터리 열거·파일 감시·경로 처리
settings              키맵·탭 복원·필터 preset·버전 migration
```

스레드 제안: UI thread는 창·위젯만 처리, player control worker는 명령/이벤트, mpv 내부 thread는 디코드·오디오, render owner는 GL context를 소유한다. 파일 열거 worker는 별도다. eframe callback 통합은 GL가 current인 UI/render thread에서 처리할 수 있으나 **그 안에서 일반 mpv 동기 명령을 호출하거나 control thread의 완료를 기다리지 않는다**. 별도 render thread로 바꿀 경우 공유 texture와 context 전달을 명시적으로 설계한다. [mpv thread 규칙](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h), [egui_glow callback](https://docs.rs/egui_glow/latest/egui_glow/painter/index.html)

명령 채널 제안: bounded channel에 `Open/Play/Pause/Seek/SetRate/SetVolume/SelectTrack/SetHwdec/SetVideoOptions/Close`를 넣는다. `Seek`·슬라이더는 최종 값으로 병합하고, `Open/Close/Error`는 유실하지 않는다. 이벤트는 `Loaded/Tracks/Position/RequestedHwdec/ActualHwdec/FilterApplied/Error/Ended`로 나눈다. SessionId+generation으로 닫힌 탭의 늦은 응답을 무시한다.

프레임 채널 제안: control channel에 픽셀 대용량을 싣지 않고 최신 프레임 알림과 소유권 있는 frame/texture handle을 분리한다. mpv update callback은 원자 플래그 설정과 UI wake만 한다. 렌더 락과 명령 락을 서로 기다리지 않는다. 종료는 렌더 중단→render context 해제→mpv core 종료→UI surface 해제 순서로 설계한다. [mpv callbacks/lifecycle](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)

## B7 R1·R2·PotPlayer 주요 기능별 구현 방법과 난이도

난이도는 A에서의 상대 추정(하/중/상)이다. ‘재생 중 변경’은 무중단·프레임 유실 0 보장을 뜻하지 않는다. 비교 기준은 명령 적용 후 복구 가능하고 UI가 멎지 않는 것이다(제안).

| 기능 | 구현 방법·실제 동작 확인 기준 | 난이도·출처 |
|---|---|---|
| MP4/MKV/WebM/TS, H.264/HEVC/VP9/AV1, AAC/AC3/Opus | libmpv/FFmpeg 기반 배포의 demux/decoder 구성을 확인하고 실제 파일 corpus로 조합 검증. 확장자 지원과 모든 profile/bit depth HW 지원은 별개 | 통합 하, 호환 QA 중. [mpv 저장소](https://github.com/mpv-player/mpv), [hwdec 제약](https://mpv.io/manual/stable/) |
| R1 ON/OFF | 요청 ON=`hwdec=auto`(또는 검증한 d3d11va 우선 정책), OFF=`no`를 재생 중 property로 설정. UI에는 실제 `hwdec-current`와 fallback 이유 표시 | 중. [runtime toggle/fallback](https://mpv.io/manual/stable/), [실제 상태](https://github.com/mpv-player/mpv/blob/master/DOCS/man/input.rst) |
| R1 자동 fallback | mpv의 HW 불가 시 SW fallback 활용. 반복 오류·프레임 진행 정지는 앱 watchdog으로 감지하여 SW 재시도/마지막 위치 복원하는 제안. GPU 전체 device loss가 SW decoder만으로 해결된다고 보장하지 않음 | 중~상. [fallback](https://github.com/mpv-player/mpv/blob/master/DOCS/man/options.rst) |
| 선명하게 | GPU unsharp 계열 custom GLSL hook 또는 lavfi unsharp. 강도 0=bypass, 음수/과도한 값 제한·halo 검사 제안 | 중. [mpv shader](https://mpv.io/manual/stable/), [FFmpeg 필터](https://www.ffmpeg.org/ffmpeg-filters.html) |
| 흐리게 | Gaussian/separable shader 또는 lavfi gblur. sigma·반경 제어, 해상도에 따른 비용 상한 제안 | 중. [FFmpeg gblur](https://www.ffmpeg.org/ffmpeg-filters.html), [mpv filter 연결](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst) |
| 노이즈 제거 | 초기에는 lavfi hqdn3d로 기능 충족 경로 검토. 시간축 denoise는 frame history 필요. shader의 공간 blur를 완성된 temporal denoise로 부르지 않음 | 상(화질·성능). [hqdn3d](https://ffmpeg.org/ffmpeg-filters.html?from=20421&from_column=20421), [lavfi bridge](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst) |
| 디인터레이스 | `deinterlace` property, d3d11vpp 또는 lavfi bwdif/yadif 경로 비교. 필드 parity/두 배 출력률·A/V sync 확인 | 중~상. [mpv 필터](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst), [FFmpeg](https://www.ffmpeg.org/ffmpeg-filters.html) |
| 밝기·대비·채도 | mpv brightness/contrast/saturation property로 설정. UI 값과 엔진 범위를 매핑, reset/preset 제공 제안 | 하. [property 문서](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst) |
| R2 실시간 ON/OFF·강도 | 태그 있는 vf graph와 shader 목록/parameter 상태를 앱 모델에 보관. 필터 교체와 parameter 변경은 별개이며 모든 lavfi 옵션이 runtime command 가능한 것은 아님. debounce·적용 결과 표시 제안 | 중~상. [mpv vf 명령](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst), [FFmpeg runtime T 옵션](https://ffmpeg.org/ffmpeg-filters.html) |
| 탐색 | seek 명령의 keyframe/정밀 모드 선택, 드래그 중 preview와 최종 seek 분리 제안. 긴 GOP/TS timestamp discontinuity 파일 검사 | 중. [seek](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst) |
| 속도·음량 | speed/volume/mute property. 피치 유지 정책·최대 증폭·오디오 장치 교체 확인 | 하~중. [mpv 옵션](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| SRT/ASS 자막 | mpv 자막 처리 재사용, 외부 파일 추가·자막 지연·ASS 폰트/첨부폰트·한글 확인. 별도 egui label로 ASS를 재구현하지 않는 제안 | 중. [mpv 자막 옵션](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| 트랙 선택 | track-list로 목록, aid/vid/sid 변경. 세션 재로드 시 ID 재사용 여부를 가정하지 않는 제안 | 하~중. [track properties](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst) |
| 전체 화면 | A는 앱 전체를 borderless fullscreen으로 바꾸고 트리·탭 숨김. B도 부모 창 기준 처리, 자식 HWND rect 갱신. 엔진 자체 창 fullscreen과 혼용하지 않는 제안 | 중(DPI/모니터/복원). [winit](https://docs.rs/winit/latest/winit/), [mpv embedding](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h) |

R1/R2 조합에서 가장 큰 함정은 **CPU 필터가 HW 표면을 요구 형식으로 내려받게 만드는 경우**다. `auto-copy`/`d3d11va-copy`는 HW decode ON이면서도 CPU processing이 가능한 경로이므로, HW ON과 zero-copy를 분리해 UI/문서에 적는다. 품질 경로를 위해 SW fallback이 필요할 수도 있다. [mpv hwdec copy 설명](https://github.com/mpv-player/mpv/blob/master/DOCS/man/options.rst)

R1 전환 제안 상태 기계: `PlayingHardware → Reconfiguring → PlayingSoftware` 또는 반대. 현재 위치·pause·speed·track/filter를 snapshot하고 새 decoder 확인 후 성공 이벤트를 낸다. 자동 fallback은 동일 파일에서 무한 HW 재시도하지 않는다. FFmpeg 직접이면 decoder 재개방·packet flush·keyframe seek·PTS 재기준·audio clock 조정을 앱이 구현해야 하므로 난이도 상(추정)이다. [FFmpeg HW API 경계](https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h_source.html)

향후 검증 제안: 각 컨테이너/코덱의 8/10bit·1080p/4K·CFR/VFR·손상 파일·한글 경로·외부 ASS를 준비하고, R1 전환 중 seek/속도/탭 닫기와 R2 모든 조합을 교차한다. 23.976/24/25/29.97/50/59.94/60fps를 60/120/144Hz 출력에서 비교하고 present 지연·drop·A/V drift·메모리·전력을 기록한다. 이는 수행한 테스트가 아니다.

## B8 후보 스택 비교표

기간·배포 크기는 **숙련 Rust/Windows 개발자 1인, 풀타임, 기능 구현과 제한된 QA를 포함한 추정**이다. PotPlayer 전체 기능 동등성·드라이버 전수 QA는 제외한다. 크기는 압축 전 runtime/DLL 포함 가설이며 실제 빌드 측정은 미확인이다. UI별 수치 차이보다 엔진 빌드 옵션 차이가 클 수 있다.

| 스택 | 개발 기간 추정 | 성능 예상 | R1/R2 | 배포 크기 추정 | 라이선스·유지 위험 |
|---|---|---|---|---|---|
| A: egui/glow+libmpv | 기능 시제품 3~6주, 사용 가능판 3~5개월 | HW 사용 가능. GL interop/copy 경로에 따라 전력 차이 | R1 엔진 활용, R2 shader+lavfi; CPU 필터 비용 | 60~180MB, 미확인 | egui MIT/Apache-2.0, mpv DLL의 GPL/LGPL 구분. 위젯 버전 추적. [eframe](https://docs.rs/crate/eframe/latest), [mpv 배포](https://github.com/mpv-player/mpv/releases) |
| B: Win32+libmpv D3D11 자식 창 | 시제품 3~6주, 사용 가능판 3~6개월 | mpv가 영상 swapchain 관리. UI 영상 합성 제약 | R1/R2 엔진 활용, overlay 별도 | 50~170MB, 미확인 | Windows UI 유지·DPI/포커스·배포 엔진 라이선스. [Win32](https://learn.microsoft.com/en-us/windows/win32/Controls/tab-controls), [mpv](https://github.com/mpv-player/mpv/releases) |
| Win32/D3D11+libmpv 개발 render API | 시제품 4~8주, 사용 가능판 4~7개월 | 앱 타깃에 직접 렌더·같은 API 합성 기대 | R1/R2 엔진 활용 + D3D11 효과 여지 | 50~180MB, 미확인 | 미출시 API/바인딩/회귀 검증 부담. 출시판 의존으로 바뀌면 재평가. [D3D11 API](https://github.com/mpv-player/mpv/blob/master/include/mpv/render_d3d11.h), [이력](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst) |
| winit+wgpu+FFmpeg 직접 | 시제품 2~3개월, 사용 가능판 9~18개월+ | zero-copy 실현 시 유리할 수 있으나 interop 실패 시 복사 비용 | R1 decoder 재생성 직접, R2·자막·오디오·sync 직접 | 40~160MB, 미확인 | FFmpeg 빌드별 LGPL/GPL, unsafe bridge와 wgpu major 변경. [FFmpeg](https://ffmpeg.org/legal.html), [wgpu](https://docs.rs/wgpu/latest/wgpu/struct.Device.html) |
| iced+GStreamer | 시제품 1~2개월, 사용 가능판 5~9개월 | D3D11 appsink/child sink 경로별 차이 | R1 pipeline 재구성, R2 필터 negotiation 필요 | 100~300MB+, 미확인 | LGPL 본체+선택 plugin별 확인, runtime 배포·iced custom 위젯 비용. [Rust/GStreamer](https://docs.rs/gstreamer/latest/gstreamer/), [Windows 배포](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html), [iced](https://docs.rs/iced/latest/iced/) |

Slint/Floem/Makepad/Xilem은 B2에서 기능·API 미확인 항목이 많아, 동등 기능 제품 기간 숫자를 제시할 근거가 부족하다. **기간·크기 미확인**, 영상 통합 PoC 후 재산정이 적절하다는 판단이다. UI toolkit을 바꿔도 R1/R2 엔진 책임이 사라지지 않으며, 네이티브 TreeView를 쓴다고 동영상 core가 단순해지지는 않는다.

라이선스 판단: FFmpeg는 기본 LGPL-2.1-or-later이며 선택 GPL 부분을 넣으면 FFmpeg 전체에 GPL이 적용된다. DLL 동적 링크만으로 의무가 모두 끝나는 것은 아니며 정확한 대응 소스·구성·고지가 필요하다. H.264/HEVC 등을 **디코드**하려고 x264/x265 인코더를 자동 포함할 이유는 없다는 설계 판단이다. [FFmpeg 공식 법적 안내](https://ffmpeg.org/legal.html)

mpv 공식 개발 배포는 GPLv3와 `-Dgpl=false`/LGPL FFmpeg 기반 LGPLv3 변형을 구분한다. 앱 공개 범위에 맞는 실제 바이너리·의존 라이브러리 조합을 확정해야 한다. Slint는 현행 Royalty-free/GPLv3/Commercial 선택지가 있으며 무료 독점 데스크톱도 조건 확인이 필요하다. ‘Rust crate가 permissive이므로 배포 전체도 permissive’라고 판단하면 안 된다. [mpv 배포 설명](https://github.com/mpv-player/mpv/releases), [Slint 라이선스](https://github.com/slint-ui/slint/blob/master/LICENSE.md?plain=1), [GStreamer 본체/바인딩](https://docs.rs/gstreamer/latest/gstreamer/)

## B9 권장 문서 목차

아래는 독립 문서 묶음에 대한 제안이며 실제 파일은 이 조사 report 외에는 만들지 않았다.

1. `00-requirements.md` — Rust-only 해석, 지원 파일 corpus, R1/R2 인수 기준, HDR의 필수 여부.
2. `01-stack-decision.md` — B2 후보표, A/B 선택 이유, 안정판/개발판 경계, 버전 pin.
3. `02-player-core.md` — 명령·이벤트·SessionId·탭 수명·thread/lock 규칙.
4. `03-explorer-tabs.md` — 비동기 트리, 경로/권한, 도킹·분할, 숨김 탭·오디오 정책.
5. `04-video-surface.md` — GL render API/FBO 및 Win32 자식 창 대안, DPI·전체 화면·overlay.
6. `05-hwdec-r1.md` — ON/OFF, 실제 상태, fallback, 전환 state machine, device loss.
7. `06-video-options-r2.md` — 색·공간·시간 필터, pass 순서, runtime parameter, CPU/GPU 비용.
8. `07-playback-features.md` — seek·rate·volume·SRT/ASS·track·A/V sync.
9. `08-color-hdr-interop.md` — SDR 기준색, NV12/P010, D3D11/D3D12/wgpu 공유의 한계와 실험.
10. `09-distribution.md` — stable-msvc ABI, DLL 탐색·버전·라이선스·소스 대응·업데이트.
11. `10-validation.md` — 파일/GPU/모니터 corpus, 자동·수동 검증, 성능/전력/drop/복구 지표.
12. `11-open-questions.md` — 미확인 목록, 버전 재확인 날짜, 채택/기각 근거 기록.

## B10 확신 낮은 부분

| 미확인·불확실성 | 영향 | 확인할 다음 단계(실행하지 않음) |
|---|---|---|
| libmpv 0.42.0 D3D11 render API의 안정판 배포 시기·Rust wrapper 지원 | B를 한 surface 합성으로 바꿀 결정적 요소 | release tag/header/API 버전과 wrapper enum 검토. 현재는 개발 API로 분류. [이력](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst) |
| libmpv2 최신 6.0.0/6.0.1 문서 혼재 | 잘못 pin하거나 문서와 ABI가 어긋날 수 있음 | crates.io index/실제 tarball metadata 재확인. [root](https://docs.rs/libmpv2/latest/libmpv2/), [render](https://docs.rs/libmpv2/latest/libmpv2/render/index.html) |
| GL render API에서 목표 GPU별 D3D11VA interop·zero-copy·auto-copy fallback | A 전력·4K 성능 차이 | Intel/AMD/NVIDIA·hybrid GPU별 실제 hwdec/interop/log/copy 측정. [옵션](https://github.com/mpv-player/mpv/blob/master/DOCS/man/options.rst) |
| eframe glow의 HDR end-to-end·모니터 전환·SDR overlay 밝기 | HDR 필수가 되면 A 재평가 필요 | 10bit/PQ/scRGB 출력과 UI 색 관리 PoC. 현재 HDR 미검증. [HDR 기준](https://learn.microsoft.com/en-us/windows/win32/direct3darticles/high-dynamic-range) |
| D3D11 decoder array texture→D3D12→wgpu 동기화·포맷 허용 | 무복사 약속 불가 | 동일 adapter/resource flags/fence/HAL ownership prototype와 GPU capture 필요. [interop](https://learn.microsoft.com/en-us/windows/win32/direct3d12/direct3d-12-with-direct3d-11--direct-2d-and-gdi), [HAL](https://docs.rs/wgpu/latest/wgpu/struct.Device.html) |
| R2의 모든 강도 값이 runtime 변경 가능한지·필터 재구성 시 stall | ‘실시간’ UX 품질 | 필터별 T 옵션/engine graph 교체와 프레임 정체 시간 확인. [FFmpeg runtime](https://ffmpeg.org/ffmpeg-filters.html) |
| egui pin 조합 MSVC 빌드·eFrame callback과 mpv lock 관계 | A 실제 통합 착수 위험 | 0.36.2/dock0.21.1/tree0.9의 최소 surface PoC. 문서 의존성만 확인, cargo 빌드 안 함. [dock](https://docs.rs/crate/egui_dock/latest), [tree 호환](https://docs.rs/crate/egui_ltreeview/latest/source/README.md) |
| Makepad 공개 video 위젯의 목표 Windows 코덱/R1/R2·외부 texture 통합 | 잠재적으로 좋은 대안을 과소평가할 수 있음 | video/backend 소스·공개 API 검토 후 범용 libmpv 연결 PoC. 모듈 존재만 확인. [모듈](https://docs.rs/makepad-widgets/latest/makepad_widgets/index.html) |
| Floem/Xilem/Slint에서 HDR·native child·영상 전용 timing의 안정 통합 | B2의 낮은 평가가 구현 불가를 뜻하지 않음 | toolkit별 공식 예제·renderer extension·실제 demo 검증. [Floem](https://github.com/lapce/floem), [Xilem](https://github.com/linebender/xilem), [Slint](https://docs.rs/slint/latest/slint/) |
| 배포 크기·기간·전력·세션 한도·PotPlayer 대비 화질 | B8은 계획용 가설뿐 | 동일 corpus와 목표 PC에서 빌드/설치/계측. 이번 조사에서는 모두 추정으로 유지 |

종합 설계 판단: **필수 R1/R2와 파일 탐색기·탭의 완성도를 먼저 얻는 A**, **Windows 출력/HDR이 우선으로 바뀌면 B**, **decoder texture 직접 통제가 제품 핵심이 될 때 FFmpeg+D3D11 자체 core** 순으로 위험을 늘리는 접근을 권한다. 이는 성능 측정 결론이 아니라 위 공식 API 경계에 근거한 조사자의 우선순위다.
