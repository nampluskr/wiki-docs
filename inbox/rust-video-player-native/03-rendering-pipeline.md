---
tags: [rust, video-player, rendering, opengl, wgpu, d3d11]
created: 2026-11-01
updated: 2026-11-01
---

# 03. 렌더링 파이프라인

> 이 파일을 읽는 때: 탭 안에 영상을 그리는 코드를 짤 때(1단계 glow), 그리고 2단계 wgpu 로 옮기는 브리지를 설계할 때.

## 1. 용어: 복사의 세 종류

이 세 가지는 서로 다르다. 문서와 UI 에서 섞어 쓰지 않는다.

| 용어 | 뜻 | mpv 에서의 예 |
| --- | --- | --- |
| CPU 왕복 (copy-back) | GPU 에서 디코딩한 프레임을 시스템 RAM 으로 내렸다가 다시 GPU 로 올린다 | `d3d11va-copy`, `auto-copy` ([options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)) |
| GPU 복사 | VRAM 안에서 다른 텍스처로 복사한다. CPU 는 거치지 않는다 | D3D11 출력에서는 기본값이 decoder surface → shader resource GPU-to-GPU 복사다 (`--d3d11va-zero-copy` 설명) |
| zero-copy | 디코더 표면을 그대로 샘플링한다 | `--d3d11va-zero-copy=yes`. 가장자리 오샘플링과 드라이버 버그 가능성이 있다고 문서에 적혀 있다 |

"CPU 왕복 없음"이 "복사 없음"과 같은 말은 아니다. zero-copy 는 제품 요구사항으로 두지 않는다(sol 보고서 B3, [options.rst `--d3d11va-zero-copy`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)).

## 2. 1단계: eframe glow + libmpv OpenGL render API

### 2.1 사용하는 API (확인된 것)

| 쪽 | API | 출처 |
| --- | --- | --- |
| eframe | `CreationContext::gl`, `CreationContext::get_proc_address` (glow feature) | [epi.rs 0.37.0](https://github.com/emilk/egui/blob/0.37.0/crates/eframe/src/epi.rs) |
| eframe | `Frame::register_native_glow_texture(glow::Texture) -> TextureId` | 같은 파일 |
| egui_glow | `CallbackFn::new(|info: PaintCallbackInfo, painter: &Painter| …)`, `Painter::gl()`, `Painter::intermediate_fbo()` | [painter.rs 0.37.0](https://github.com/emilk/egui/blob/0.37.0/crates/egui_glow/src/painter.rs) |
| mpv | `mpv_render_context_create` (`API_TYPE_OPENGL`, `OPENGL_INIT_PARAMS` 에 `get_proc_address`) | [render_gl.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h) |
| mpv | `mpv_render_context_render` + `MPV_RENDER_PARAM_OPENGL_FBO` (`mpv_opengl_fbo { fbo, w, h, internal_format }`) | 같은 파일 |

`mpv_opengl_fbo` 의 `w`·`h` 는 "must refer to the size of the framebuffer" 이다. 즉 mpv 는 FBO **전체**에 그린다. 그래서 화면 FBO 의 일부(탭 영역)에 바로 그리지 않고, **탭마다 별도 FBO** 를 둔다.

### 2.2 프레임 흐름 (설계)

```text
mpv update 콜백 ──▶ 원자 플래그 set + request_repaint()
                         │
UI 스레드 (eframe update)  ▼
  1. 탭 rect → 물리 픽셀 크기 계산, 크기가 바뀌면 탭 FBO/텍스처 다시 만들기
  2. paint callback (CallbackFn):
       - mpv_render_context_update() 로 새 프레임 여부 확인
       - 탭 FBO 바인드 → mpv_render_context_render(FBO, w, h)
       - painter.intermediate_fbo() 로 되돌리기
  3. 탭 텍스처(register_native_glow_texture 로 얻은 TextureId)를 video_rect 에 그린다
  4. 그 위에 OSD·패널을 그린다
```

- 2번 콜백과 3번 이미지가 같은 프레임 안에서 순서대로 실행되는지(콜백이 먼저 실행되는지)는 egui 의 도형 순서에 달려 있다. 실제 동작은 **미확인**이다. 안전하게 하려면 3번은 직전 프레임 텍스처를 그려도 되게 설계한다(한 프레임 지연 허용).
- 콜백 안에서는 mpv_render_* 와 GL 호출만 한다. 일반 mpv API 를 호출하거나 제어 스레드를 기다리지 않는다 ([04-player-core.md](04-player-core.md), [render.h 스레드 규칙](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)).
- GL 상태: mpv 가 GL 상태를 바꾸므로, 콜백이 끝나면 egui 가 쓰는 FBO·viewport 를 되돌린다. egui_glow 문서도 오프스크린 작업 뒤에 `intermediate_fbo()` 로 되돌리라고 한다 ([painter.rs](https://github.com/emilk/egui/blob/0.37.0/crates/egui_glow/src/painter.rs)).

### 2.3 이 경로의 하드웨어 디코딩

eframe 0.37 의 Windows GL 은 기본이 WGL 이다. mpv OpenGL API 에서 직접 하드웨어 interop 을 쓰려면 "Windows: ANGLE is required" 이다. 그래서 이 경로에서는 대부분 `d3d11va-copy`(copy-back)가 된다. NVIDIA 는 CUDA/GL interop 이 따로 있다. 자세한 내용은 [01-stack-decision.md](01-stack-decision.md) 3.1 에 있다 ([render_gl.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h), sol followup-1).

`d3d11vpp`(D3D11 비디오 후처리 필터)는 "Requires a D3D11 context" 이므로 이 경로에서는 쓸 수 없다 ([vf.rst v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst)).

### 2.4 색 변환과 R2 의 위치

1단계에서는 YUV→RGB, 스케일링, 자막 합성, 색 조정을 **mpv 가 한다**. 앱은 mpv 가 그린 RGB 텍스처만 받는다. R2 는 mpv 안에 둔다.

| R2 수단 | 위치 | 출처 |
| --- | --- | --- |
| `brightness`·`contrast`·`saturation` 등 | mpv GPU 렌더러의 색 처리 | [options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| `--vf=lavfi=[…]` (hqdn3d·gblur·cas·unsharp·bwdif) | 디코딩 직후 CPU 필터 체인 | [vf.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst) |
| `--glsl-shaders` | mpv GPU 렌더러의 셰이더 훅. `vo=libmpv` 에도 적용된다("specific to --vo=gpu, --vo=libmpv and --vo=gpu-next") | [options.rst GPU renderer options](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst), [매뉴얼](https://mpv.io/manual/stable/#gpu-renderer-options) |

앱이 탭 텍스처에 자체 셰이더를 거는 방식은 쓰지 않는다. 그렇게 하면 mpv 가 이미 합성한 자막까지 흐려지거나 선명해진다. 효과는 자막·UI 를 합성하기 전에 적용한다 (sol 보고서 B3).

## 3. 2단계: wgpu (DX12) + libmpv 0.42 D3D11 render API

> 이 절은 **설계 초안**이다. mpv 0.42.0 은 아직 출시되지 않았고(2026-11-01), 아래 브리지는 PoC 로 검증해야 한다.

### 3.1 mpv 쪽 (master 기준)

앱이 `ID3D11Device` 를 `MPV_RENDER_PARAM_D3D11_INIT_PARAMS` 로 넘긴다. 프레임마다 `MPV_RENDER_PARAM_D3D11_FBO` 로 대상 `ID3D11Texture2D` 를 넘긴다. 스왑체인과 Present 는 앱이 맡는다 ([PR #17764](https://github.com/mpv-player/mpv/pull/17764), [render_d3d11.h (master)](https://github.com/mpv-player/mpv/blob/master/include/mpv/render_d3d11.h), [client-api-changes.rst](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst)).

### 3.2 D3D11 텍스처 → wgpu

wgpu 에는 D3D11 HAL 이 없다. `create_texture_from_hal` 이 받는 백엔드는 Vulkan·Metal·Dx12·Gles 이다 ([wgpu Device](https://docs.rs/wgpu/latest/wgpu/struct.Device.html)). 그래서 D3D12 를 거친다.

1. D3D11 쪽: 공유 가능한 RGBA 텍스처를 만든다(`D3D11_RESOURCE_MISC_SHARED_NTHANDLE`, 필요하면 keyed mutex). mpv 는 여기에 그린다 ([D3D11_RESOURCE_MISC_FLAG](https://learn.microsoft.com/en-us/windows/win32/api/d3d11/ne-d3d11-d3d11_resource_misc_flag)).
2. `IDXGIResource1::CreateSharedHandle` 로 NT 핸들을 만든다.
3. wgpu 의 `ID3D12Device` 에서 `OpenSharedHandle` 로 `ID3D12Resource` 를 연다 ([D3D12 와 D3D11 상호운용](https://learn.microsoft.com/en-us/windows/win32/direct3d12/direct3d-12-with-direct3d-11--direct-2d-and-gdi), [shared heaps](https://learn.microsoft.com/en-us/windows/win32/direct3d12/shared-heaps)).
4. `wgpu_hal::dx12::Device::texture_from_raw` 로 감싸고 `Device::create_texture_from_hal` 로 `wgpu::Texture` 를 만든다 ([wgpu-hal](https://docs.rs/wgpu-hal/latest/wgpu_hal/)).
5. 동기화: D3D11 쪽 쓰기가 끝난 것을 공유 fence(또는 keyed mutex)로 알리고, D3D12 쪽에서 기다린 뒤 샘플링한다.

조건: 같은 어댑터(LUID), 포맷·usage·초기 리소스 상태 일치, 수명·소유권 관리. 이 패턴은 서드파티 크레이트(grafting, mpv-engine)에서 확인했을 뿐이다. wgpu 공식 API 가 아니며, wgpu-hal 의 시그니처는 메이저 버전마다 바뀔 수 있다. 그래서 브리지 모듈은 wgpu 버전을 고정하고 따로 격리한다 ([grafting](https://docs.rs/crate/grafting/latest/source/src/dx12_shared_texture.rs), [mpv-engine](https://docs.rs/crate/mpv-engine/0.2.0/source/src/export/wgpu_windows.rs), argon 보고서 B10).

이 경로는 mpv 가 **RGB 로 그린 결과**를 공유한다. 디코더의 NV12 표면을 직접 공유하는 것이 아니다. 그래서 NV12/P010 피처(wgpu 30.0.1 `TEXTURE_FORMAT_NV12`/`P010`, DX12·Vulkan, native 전용)는 FFmpeg 직접 경로에서만 필요하다 ([wgpu Features](https://docs.rs/wgpu/latest/wgpu/struct.Features.html)).

## 4. 대안: FFmpeg 직접 경로의 YUV→RGB (참고)

FFmpeg 직접 경로를 고르면 앱이 색 변환을 해야 한다.

- NV12/P010 의 Y·UV 평면을 각각 뷰로 만들어 셰이더에 바인딩한다.
- 행렬(BT.601/709/2020), 범위(limited/full), 크로마 위치, 전달 함수(SDR/PQ/HLG)를 프레임 메타데이터로 넘긴다.
- 패스 순서 제안: 디인터레이스 → 노이즈 제거 → 흐림·선명 → 색 조정 → 톤 매핑·출력 → 자막 → UI.

(세 보고서 B3 의 공통 제안. HDR 출력은 [Microsoft HDR 지침](https://learn.microsoft.com/en-us/windows/win32/direct3darticles/high-dynamic-range) 에 따라 별도로 설계해야 하며 미확인이다.)

## 5. 프레임 타이밍

- vsync 를 지원한다고 영상 PTS·오디오 동기가 보장되지는 않는다. 1단계에서는 mpv 가 타이밍을 관리하고, 앱은 update 콜백이 오면 다시 그린다.
- 정지 화면에서 매 프레임 다시 그리는 루프를 돌리지 않는다. 입력이 있을 때와 새 영상 프레임이 있을 때만 다시 그린다 (sol 보고서 B2).
- wgpu(2단계)에서는 `PresentMode::Fifo` 만 보장되고 나머지는 지원 여부를 조회해야 한다. `desired_maximum_frame_latency` 는 힌트일 뿐이다 ([SurfaceConfiguration](https://docs.rs/wgpu/latest/wgpu/type.SurfaceConfiguration.html)).

---

이전: [02-decoding-engine.md](02-decoding-engine.md) · 다음: [04-player-core.md](04-player-core.md)
