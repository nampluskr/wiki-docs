# 재질의 1 — WGL 하드웨어 디코딩·ANGLE·Windows 배포 표기

작성자: rvpn-scouter-sol. 확인일: **2026-11-01**. mpv는 **v0.41.0 태그**, egui/eframe/egui_glow는 **0.37.0 태그**를 직접 열어 조사했다. 소스에서 도출한 결론은 ‘소스 판독’, 실제 Windows 실행 여부는 별도로 표시한다. 패키지 설치·빌드·시스템 설정 변경은 하지 않았다.

## 1. mpv v0.41.0 OpenGL render API + Windows 네이티브 WGL

### 결론과 앞선 보고서 보정

**WGL의 libmpv OpenGL render API에서 `d3d11va` 직접 interop은 지원 경로가 아니다. `d3d11va-copy`는 사용할 수 있는 HW decode 후보이며, `hwdec=auto`도 copy 후보를 시도한다. 그러나 “WGL이면 dxva2+dxinterop이 직접 된다”는 설명도 외부 컨텍스트의 libmpv에 그대로 적용하면 안 된다.** mpv 소유 `gpu-context=dxinterop`과 eframe이 만든 외부 WGL context를 구분해야 한다. 또한 NVIDIA CUDA/OpenGL interop이라는 별도 후보가 있으므로 WGL의 HW 경로를 DXVA2만으로 한정할 수 없다. [v0.41.0 GL API의 HW 조건](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/include/mpv/render_gl.h), [DXVA2 interop 초기화](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/hwdec_dxva2gldx.c), [CUDA/GL](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/hwdec/hwdec_cuda_gl.c)

앞선 `report.md`의 A 추천은 **기능 통합 후보**로는 유지하되, Windows 전반에서 D3D11VA→GL의 CPU 왕복 없는 경로가 쉽게 제공된다는 의미로 읽어서는 안 된다. 특히 Intel/AMD의 기본 WGL에서는 copy-back 성능을 전제로 검증해야 한다는 설계 보정이다. 모든 GPU에서의 실제 성능·자동 선택 결과는 **미확인**이다. [GL API 문서](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/include/mpv/render_gl.h), [외부 GL backend](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/libmpv_gl.c)

### 1.1 `d3d11va`가 WGL에서 직접 연결되지 않는 이유

v0.41.0 `options.rst`는 `d3d11va`에 D3D11 또는 ANGLE context를 요구하며, `d3d11va-copy`는 시스템 RAM으로 복사하는 모드로 설명한다. 외부 render API의 정확한 조건은 `render_gl.h`에도 있으며, Windows 직접 HW interop에는 ANGLE이 필요하고 GL/DX는 이론적 가능성으로만 언급한다. [options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst), [render_gl.h:77–96](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/include/mpv/render_gl.h)

구현도 이를 뒷받침한다. 네이티브 D3D11 hwdec의 `init`은 `ra_is_d3d11`이 아니면 실패한다. ANGLE 경로인 **`d3d11-egl`**의 `init`은 current EGL display/context, ANGLE의 D3D texture/EGLStream 관련 확장, 그리고 `EGL_D3D11_DEVICE_ANGLE`을 요구한다. 순수 WGL context는 이 조건을 제공하지 못한다(소스 판독). [D3D11 init:67–75](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/d3d11/hwdec_d3d11va.c), [ANGLE init:77–179 및 driver name:319–324](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/hwdec_d3d11egl.c)

외부 libmpv GL backend는 호출자가 제공한 GL 함수로 빈 `ra_ctx`를 초기화하며 mpv의 창 context를 만들지 않는다. 따라서 **`gpu-context=angle` 옵션만 설정해 이미 생성된 eframe WGL context를 ANGLE로 바꿀 수 없다**(소스 판독). 실제 EGL/GLES context를 앱에서 준비해야 한다. [libmpv_gl.c:13–55](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/libmpv_gl.c)

### 1.2 DXVA2 + dxinterop은 외부 WGL에도 그대로 가능한가?

**아니다. v0.41.0의 stock libmpv render API에 대해서는 직접 사용 가능하다고 말할 수 없다.** `hwdec_dxva2gldx.c`의 driver 이름은 **`dxva2-dxinterop`**이다. 초기화에는 GL의 `MPGL_CAP_DXINTEROP`뿐 아니라 renderer native resource의 **`dxinterop_device_HANDLE`과 `IDirect3DDevice9Ex`**가 필수이며, 없으면 즉시 실패한다. 해당 소스는 이 자원을 `context_dxinterop`에서 재사용한다고 설명한다. GPU 내 변환에는 `StretchRect`도 있으므로 이 경로 자체도 ‘GPU 작업/복사 0’은 아니다. [init:54–87, map:187–214, driver:216–229](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/hwdec_dxva2gldx.c)

외부 libmpv의 native-resource mapping은 X11/Wayland/DRM 항목만 제공하고 위 D3D9 자원의 전달 항목은 없다. 외부 GL 초기화에도 그 자원을 등록하는 코드가 없다. 따라서 **드라이버가 WGL_NV_DX_interop을 지원하는 것만으로는 부족**하며, stock render API + eframe WGL에서 `dxva2-dxinterop`을 켤 수 있다는 주장은 성립하지 않는다(두 소스를 결합한 판독). mpv 수정이나 별도 interop 구현의 가능성·구현 비용은 **미확인**이다. [native_resource_map:19–40 및 등록:67–78](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/gpu/libmpv_gpu.c), [외부 GL 초기화](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/libmpv_gl.c)

`WGL_NV_DX_interop` 함수 로딩과 capability의 연결 자체는 `common.c`에 있다. 하지만 이것은 필요한 자원 등록·context 생성까지 보장하는 API가 아니다. `context_dxinterop.c` URL은 이번 웹 도구에서 fetch에 실패했으므로 **그 파일 본문은 직접 확인하지 못했다**. 위 결론은 성공적으로 연 `hwdec_dxva2gldx.c`, `libmpv_gl.c`, `libmpv_gpu.c`를 근거로 한다. [common.c:440–455](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/common.c), [열람 실패한 context 파일](https://github.com/mpv-player/mpv/blob/v0.41.0/video/out/opengl/context_dxinterop.c)

### 1.3 WGL 외부 render API의 후보 정리

표는 **stock v0.41.0 소스 기준**이며, 엔진 빌드 옵션·GPU·드라이버·코덱/profile 지원이 충족되는 경우의 후보이다. ‘가능’은 실제 목표 PC에서의 성공 확인이 아니다.

| hwdec / interop | eframe 외부 WGL에서의 판정 | 근거 |
|---|---|---|
| `no` | SW decode→GL 표시. GPU로 UI/영상 표시하는 것과 HW decode는 별개 | [render GL API](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/include/mpv/render_gl.h) |
| `d3d11va` / `d3d11-egl` | 순수 WGL에서 직접 불가. ANGLE/EGL의 current context·확장·D3D11 device 필요 | [hwdec_d3d11egl.c](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/hwdec_d3d11egl.c) |
| `d3d11va-copy` | HW decode→시스템 RAM→GL upload 후보. Windows 공통 baseline으로 검증할 경로 | [options](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| `dxva2` / `dxva2-dxinterop` | mpv 소유 dxinterop context와 구분. stock 외부 WGL에는 필수 native D3D9 자원 전달이 없어 직접 사용 경로 아님 | [driver](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/hwdec_dxva2gldx.c), [외부 resource map](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/gpu/libmpv_gpu.c) |
| `dxva2-copy` | HW decode→시스템 RAM의 후보. D3D11VA-copy 실패 시 등의 대안 | [options](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| `nvdec` / `cuda-nvdec` interop driver | NVIDIA에서 CUDA/OpenGL 경로 후보. GL에서 CUDA device를 찾고 GL texture를 CUDA resource로 등록한다. Intel/AMD 공통 경로 아님 | [cuda GL device/registration](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/hwdec/hwdec_cuda_gl.c), [CUDA interop driver](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/hwdec/hwdec_cuda.c) |
| `nvdec-copy` | NVIDIA의 시스템 RAM copy-back 후보 | [options](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |

`cuda`/`cuda-copy`라는 구형 decoder도 옵션 문서에 있지만 NVDEC와 같은 것으로 부르지 않는다. 문서는 NVDEC를 우선하도록 설명한다. Windows의 기타 특수 `vaapi-copy` 등까지 이 앱의 검증된 호환 목록으로 포함시키지는 않는다. [v0.41.0 options.rst의 nvdec/cuda 설명](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)

### 1.4 `hwdec=auto`가 copy를 고르는가?

**예. v0.41.0 소스에서 `auto`는 non-copy만 선택하는 모드가 아니다.** `hwdec_autoprobe_info`에 `d3d11va-copy`, `dxva2-copy`, `nvdec-copy` 등이 whitelist로 있고, `hwdec_compare`는 non-copy를 먼저 정렬한다. `auto-copy`는 반대로 non-copy를 제외한다. `dxva2` non-copy에는 whitelist flag가 없어 기본 safe `auto` 대상과도 구분된다. [vd_lavc.c:239–290, 458–537](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/decode/vd_lavc.c)

WGL에서 D3D11 direct device를 얻지 못하면 다음 후보로 진행하고, NVIDIA NVDEC direct도 불가하면 copy 후보를 시도할 수 있다. copy 모드는 별도 device를 만들며, 프레임은 `mp_image_hw_download`로 내려받는다. 모두 실패하면 SW 경로다. **특정 PC에서 반드시 `d3d11va-copy`가 선택된다고 보장하는 것은 아니며**, 실제 선택은 미확인이다. [vd_lavc.c:422–456, 572–585, 1226–1235](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/decode/vd_lavc.c)

소스 판독에 따른 제안: 기본 정책은 `auto`, 재현 가능한 진단/기준 경로는 `d3d11va-copy`, OFF는 `no`로 구분한다. 실제 decoder/interop 상태와 로그를 기록하고 copy-back bandwidth를 측정한다. 이는 제안이며 실행 검증한 설정 조합은 아니다.

## 2. eframe / egui_glow 0.37의 callback API와 ANGLE

### 2.1 확인한 callback API 이름

**정확한 타입 이름은 `egui_glow::CallbackFn`**이며 root에서 `painter::CallbackFn`을 re-export한다. 생성 함수는 **`egui_glow::CallbackFn::new`**, closure 인자는 **`egui::PaintCallbackInfo`, `&egui_glow::Painter`**이며 `Send + Sync + 'static` 조건이 있다. 이를 `egui::PaintCallback`의 callback에 넣는 방식이다. `Painter::gl()`로 context에 접근하고 offscreen 작업 후에는 `Painter::intermediate_fbo()`의 타깃으로 복원한다. [0.37 lib.rs re-export](https://github.com/emilk/egui/blob/0.37.0/crates/egui_glow/src/lib.rs), [0.37 painter.rs:99–115, 253–274](https://raw.githubusercontent.com/emilk/egui/0.37.0/crates/egui_glow/src/painter.rs)

eframe의 `CreationContext::gl`과 **`CreationContext::get_proc_address`**도 확인했다. 둘 다 glow feature 관련 항목이며 후자는 libmpv에 GL 함수 resolver를 연결할 때 사용할 수 있는 공개 경로라는 설계 판단이다. GL callback closure 안에서 일반 mpv 동기 명령과 renderer 동작을 섞지 않는 thread/lock 규칙은 별도 준수해야 한다. [0.37 epi.rs:45–67](https://raw.githubusercontent.com/emilk/egui/0.37.0/crates/eframe/src/epi.rs), [mpv GL context 규칙](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/include/mpv/render_gl.h)

### 2.2 Windows ANGLE: 가능성과 기본 선택을 구분

**glutin 0.32.3의 Windows EGL backend는 ANGLE을 사용할 수 있다.** 공식 문서는 DLL 검색 경로에 `libEGL.dll`과 `libGLESv2.dll`이 있으면 ANGLE 사용이 가능하다고 설명한다. eframe 0.37의 glow 의존성은 glutin/glutin-winit의 `egl`, `wgl` feature를 모두 활성화한다. [glutin DisplayApiPreference::Egl](https://docs.rs/glutin/0.32.3/glutin/display/enum.DisplayApiPreference.html), [eframe 0.37 Cargo.toml:163–167](https://raw.githubusercontent.com/emilk/egui/0.37.0/crates/eframe/Cargo.toml)

그러나 eframe 0.37은 DisplayBuilder에 **`ApiPreference::FallbackEgl`을 하드코딩**한다. glutin-winit 0.5 소스에서 Windows의 이 값은 **`WglThenEgl`**로 변환된다. 즉 WGL display 생성이 성공하면 단지 ANGLE DLL이 있다고 EGL이 우선 선택되지는 않는다. GLES context 재시도도 존재하지만 display API를 EGL로 바꾸는 옵션과 동일하지 않다. [eframe glow_integration.rs:1129–1135, 1721–1733](https://raw.githubusercontent.com/emilk/egui/0.37.0/crates/eframe/src/native/glow_integration.rs), [glutin-winit 소스:178–181](https://docs.rs/glutin-winit/0.5.0/src/glutin_winit/lib.rs.html)

따라서 판정은 다음과 같다. **egui_glow의 Windows ANGLE 사용은 기술적으로 가능한 계열이며, stock eframe 0.37의 기본 Windows 경로를 ANGLE이라고 부르면 안 된다.** 확인한 `NativeOptions`에는 EGL/ANGLE 우선 선택 필드가 없고, 구현에도 향후 공개 여부를 고려하는 주석이 있다. ANGLE을 확실히 우선하려면 자체 winit/glutin 셸에서 `PreferEgl`/EGL 전용 선택을 하거나 eframe 통합을 수정하는 설계가 필요하다. 해당 수정의 빌드·실행 성공은 **미확인**이다. [NativeOptions 정의](https://raw.githubusercontent.com/emilk/egui/0.37.0/crates/eframe/src/epi.rs), [하드코딩 위치](https://raw.githubusercontent.com/emilk/egui/0.37.0/crates/eframe/src/native/glow_integration.rs), [API preference](https://docs.rs/glutin-winit/0.5.0/src/glutin_winit/lib.rs.html)

ANGLE을 사용하더라도 libmpv가 요구하는 EGLStream/ANGLE 확장과 D3D11 device가 그 DLL 빌드에 있어야 한다. 따라서 **“eframe ANGLE 가능” ≠ “어떤 ANGLE DLL에서도 libmpv d3d11-egl 성공”**이다. DLL을 앱 로컬에 둔 경우의 glutin/mpv 양쪽 로딩 일치, GPU 선택, 코덱별 실제 HW decode, R1 전환은 모두 **미확인**이다. [mpv ANGLE init 조건](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/hwdec_d3d11egl.c)

## 3. mpv Windows GPL/LGPL 바이너리 표기 출처 — B4-4

**직접 확인한 가장 구체적인 출처는 mpv 저장소의 `git-release` 페이지의 `libmpv (Windows)` 절**이다. 이 절은 `libmpv-2.dll`, import library, headers를 설명하고, GPLv3 빌드와 LGPLv3 빌드를 구분한다. GPL 변형은 mpv GPL 부분과 FFmpeg의 GPL/version3 옵션, LGPL 변형은 `-Dgpl=false`와 LGPL FFmpeg 조합이라고 명시한다. x86_64와 arm64 각각 GPL/LGPL 링크가 있다. [직접 URL](https://github.com/mpv-player/mpv/releases/tag/git-release)

확인 시 페이지는 `v0.41.0-1373-g9e2ed65eb`, `Built: 2026-10-30 18:27:33 UTC`를 표시했다. 이 값은 변경 가능한 개발 빌드 식별 정보다. 동일 페이지는 미검증 master 개발 빌드이며 공식 안정 release가 아니라고 경고한다. ZIP 내용·라이선스 파일·import library의 MSVC 링크 성공은 **미확인**이다. [개발 배포 페이지](https://github.com/mpv-player/mpv/releases/tag/git-release)

표현을 **“공식 프로젝트가 제공하는 first-party CI 개발 바이너리의 GPL/LGPL 변형”**으로 좁히는 것이 정확하다. mpv 공식 installation 페이지도 GitHub CI를 first-party/latest commit 경로로 구분하고, 테스트용이며 기능이 빠질 수 있다고 설명한다. shinchiro·zhongfly 등 다른 Windows 배포는 별도 third-party 경로다. [공식 installation](https://mpv.io/installation/)

**v0.41.0 안정판 DLL의 GPL/LGPL 두 변형이 공식 안정 바이너리로 배포된다는 근거는 이번 확인에서 찾지 못했다.** 안정 release 페이지에서는 LGPL 표기를 확인하지 못했으므로 B4-4를 안정판 바이너리 양종 제공의 근거로 인용하면 안 된다. 앞선 report의 바이너리 설명은 위 CI 개발 배포에 한정한다. [v0.41.0 안정 release](https://github.com/mpv-player/mpv/releases/tag/v0.41.0), [GPL/LGPL 표기가 실제 있는 CI 개발 release](https://github.com/mpv-player/mpv/releases/tag/git-release)

## 판정 요약

1. stock libmpv v0.41.0 + 외부 WGL: **D3D11VA 직접 불가, D3D11VA-copy 후보 가능**. DXVA2의 mpv dxinterop context도 외부 WGL에 자동 이식되지 않는다. NVIDIA에는 CUDA/GL 후보가 별도로 있다. [GL API](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/include/mpv/render_gl.h), [DXVA2 driver](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/opengl/hwdec_dxva2gldx.c), [CUDA GL](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/out/hwdec/hwdec_cuda_gl.c)
2. `auto`: **copy 후보까지 포함**한다. 특정 시스템의 최종 선택은 미확인. [선택 구현](https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/video/decode/vd_lavc.c)
3. 0.37 callback: **`egui_glow::CallbackFn::new`**. [정의](https://raw.githubusercontent.com/emilk/egui/0.37.0/crates/egui_glow/src/painter.rs)
4. ANGLE: glutin은 지원하지만 eframe 0.37 기본은 **WGL 우선/EGL fallback**. ANGLE 강제 선택 및 mpv interop의 실동작은 미확인. [eframe 소스](https://raw.githubusercontent.com/emilk/egui/0.37.0/crates/eframe/src/native/glow_integration.rs), [glutin](https://docs.rs/glutin/0.32.3/glutin/display/enum.DisplayApiPreference.html)
5. GPL/LGPL 표기: **공식 저장소의 first-party CI 개발 libmpv 배포에 명시**됨. v0.41.0 공식 안정 DLL 양종 제공은 미확인. [git-release](https://github.com/mpv-player/mpv/releases/tag/git-release)
