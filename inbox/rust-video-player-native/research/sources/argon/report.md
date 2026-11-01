# Rust 동영상 플레이어 원점 설계 조사 보고서 (Windows 11, 2026-11 기준)

## 요약 (10줄)
1. **1순위 권장 스택(실용·완성도형)**: `winit` + `wgpu` + `egui`(`egui_tiles`) + `libmpv2`(`mpv_render_context` / `gpu-next`) 조합으로, PotPlayer 급 코덱·자막(ASS)·트랙·탐색·R1·R2를 최소 기간(추정 2~3개월)에 달성한다.
2. **2순위 권장 스택(순수 제어·Zero-Copy형)**: `winit` + `wgpu`(DX12 백엔드 + `wgpu-hal` D3D11/D3D12 공유 텍스처) + `egui`(`egui_tiles`) + `ffmpeg-next`(D3D11VA) 조합으로, 디코더부터 WGSL 후처리 셰이더까지 완전한 통제권을 확보한다.
3. **창·UI 선택**: 11종 후보 비교 결과, 동영상 뷰포트 위에 반투명 오버레이 OSD를 얹고 IDE급 트리·탭·분할(`egui_tiles` v0.18.0)을 즉시 구현할 수 있는 `egui` v0.37.0이 가장 성숙하다 (https://crates.io/crates/egui, 2026-11-01 확인).
4. **렌더링 경로(B3)**: D3D11VA 하드웨어 디코더가 출력한 `ID3D11Texture2D`(NV12/P010)를 `IDXGIResource1::CreateSharedHandle`(NT 핸들) 및 `ID3D12Device::OpenSharedHandle`, `wgpu_hal::api::Dx12`로 가져오면 CPU 복사 없는 Zero-Copy 표시가 가능하다.
5. **색공간·HDR**: `wgpu` v30.0.1의 `TEXTURE_FORMAT_NV12`·`TEXTURE_FORMAT_P010` 피처와 `Rgba16Float`(scRGB) 스왑체인을 결합하면 10-bit HDR10 영상을 색 손실 없이 출력할 수 있다 (https://docs.rs/wgpu/latest/wgpu/, 2026-11-01 확인).
6. **R1(HW/SW 디코딩 전환)**: `libmpv`는 런타임 `hwdec` 속성(`d3d11va` ↔ `no`) 변경으로 즉시 전환·폴백되며, `ffmpeg-next`는 현재 PTS를 기록한 뒤 디코더 컨텍스트를 재생성하고 키프레임 탐색 후 디코딩을 이어간다.
7. **R2(실시간 영상 필터)**: CPU 필터(`vf`의 `hqdn3d`, `unsharp` 등)는 `d3d11va` Zero-Copy와 양립하지 않고 `d3d11va-copy`를 강제하므로, `wgpu` WGSL 프래그먼트/컴퓨트 셰이더 또는 `libplacebo` 커스텀 GLSL 훅(`glsl-shaders`)에서 처리해야 한다.
8. **다중 탭·숨김 탭 자원 관리**: 활성 탭만 GPU 텍스처 렌더링·오디오 출력을 유지하고, 비활성(숨김) 탭은 일시정지 또는 디코더 참조 프레임만 갱신하며 스왑체인/렌더 패스 제출을 생략해 VRAM·GPU 점유를 억제한다.
9. **코어·UI 분리(B6)**: 워크스페이스를 `player-core`, `video-render`, `ui-explorer-tabs`, `app-shell` 4개 크레이트로 분리하고, UI와 코어 사이는 `crossbeam-channel` 명령/이벤트 큐와 `Arc<Atomic*>` 상태 공유로 비동기 격리한다.
10. **라이선스·배포 주의**: FFmpeg와 libmpv는 기본적으로 LGPLv2.1+/GPLv2+ 조건이 따르므로 상용/독점 배포 시 동적 링크(`libmpv-2.dll`, `avcodec-*.dll`) 및 LGPL 빌드 구성을 엄수해야 한다.

---

## B1 원점 권장 스택 1~2개와 이유

### 권장 스택 1 (실용·고완성도 최우선): `winit` + `wgpu` + `egui` (`egui_tiles`) + `libmpv2`
- **구성 버전 (2026-11-01 확인)**:
  - 창/이벤트: `winit` v0.30.13 (https://crates.io/crates/winit)
  - 그래픽/합성: `wgpu` v30.0.1 (https://crates.io/crates/wgpu) + `windows` v0.62.2 (https://crates.io/crates/windows)
  - UI/탭/트리: `egui` v0.37.0 (https://crates.io/crates/egui), `egui_tiles` v0.18.0 (https://crates.io/crates/egui_tiles)
  - 미디어 코어: `libmpv2` v6.0.0 (https://crates.io/crates/libmpv2) — 내부 `libmpv-2.dll` (`vo=gpu-next` / `libplacebo` 기반)
- **선정 이유**:
  1. **PotPlayer급 재생 기능 즉시 확보**: 컨테이너(MP4/MKV/WebM/TS), 코덱(H.264/HEVC/VP9/AV1), 오디오(AAC/AC3/Opus), 고급 자막 렌더러(`libass` 기반 SRT/ASS 스타일·애니메이션), 트랙 전환, 정밀 탐색(Keyframe/Exact), 배속 재생(`scaletempo2`)이 검증된 C 코어에 내장되어 있다 (https://mpv.io/manual/master/, 2026-11-01 확인).
  2. **R1·R2의 무결점 충족**: `hwdec=d3d11va`(또는 `auto-safe`) 속성을 재생 중 런타임에 `no`로 바꿔 즉시 HW↔SW 전환이 가능하며, 하드웨어 디코딩 실패 시 자동으로 소프트웨어 디코딩으로 폴백한다. R2 필터는 `gpu-next`(`libplacebo`)의 내장 색상 속성(`brightness`, `contrast`, `saturation`), 하드웨어/GPU 디인터레이스(`d3d11vpp`/`bwdif`), 그리고 GPU `glsl-shaders` 체인으로 Zero-Copy 상태에서 실시간 강도 조절이 가능하다 (https://github.com/mpv-player/mpv/blob/master/DOCS/man/options.rst, 2026-11-01 확인).
  3. **파일 탐색기·탭 UI 생산성**: `egui`의 `CollapsingHeader`(또는 가상 스크롤 트리)와 Rerun 팀이 관리하는 `egui_tiles`를 결합하면 좌측 탐색기 패널 + 우측 다중 동영상 탭·분할 뷰와 탭 내부 커스텀 텍스처 및 플로팅 OSD 오버레이를 단일 `wgpu` 렌더 패스 위에서 바로 구현할 수 있다 (https://github.com/rerun-io/egui_tiles, 2026-11-01 확인).

### 권장 스택 2 (순수 Rust 렌더 파이프라인·Zero-Copy 제어형): `winit` + `wgpu` (`wgpu-hal` DX12/D3D11 Interop) + `egui` (`egui_tiles`) + `ffmpeg-next`
- **구성 버전 (2026-11-01 확인)**:
  - 창/렌더링/UI: `winit` v0.30.13 + `wgpu` v30.0.1 (`wgpu-hal` 직접 사용) + `egui` v0.37.0 + `egui_tiles` v0.18.0 + `windows` v0.62.2
  - 디코딩/오디오/자막: `ffmpeg-next` v9.0.0 (https://crates.io/crates/ffmpeg-next), 오디오 출력 `cpal` v0.16.0 (https://crates.io/crates/cpal)
- **선정 이유**:
  1. **렌더링·셰이더 완전 통제**: `ffmpeg-next`로 `AV_HWDEVICE_TYPE_D3D11VA` 디코더를 열어 NV12/P010 `ID3D11Texture2D`를 얻고, DXGI 공유 핸들(`CreateSharedHandle`)을 통해 `wgpu`(DX12 백엔드)로 Zero-Copy 임포트한 뒤, YUV→RGB 변환과 R2 후처리(Unsharp, Gaussian Blur, Bilateral/Temporal Denoise, Yadif/Bwdif 컴퓨트 셰이더, EQ)를 모두 순수 WGSL 셰이더 파이프라인으로 직접 작성·제어할 수 있다.
  2. **단일 그래픽 컨텍스트 통합**: `libmpv`의 외부 렌더 컨텍스트 브리지 비용 없이 모든 탭의 비디오 텍스처와 UI 메쉬가 하나의 `wgpu::Device` 안에서 합성되므로 다중 탭 동시 재생과 커스텀 비주얼 효과가 가장 깔끔하다. 단, A/V 동기화·탐색 상태 머신·ASS 자막 렌더링을 직접 조립해야 하므로 개발 기간이 스택 1보다 약 2배 소요된다(추정).

---

## B2 창·UI 후보 비교

Windows 11 기준 주요 Rust 창·UI 후보 11종의 동영상 적합성, 위젯 성숙도, 버전·유지 상태 비교다 (모두 2026-11-01 확인).

| 후보 스택 | 최신 버전 (확인일) | 동영상 적합성 (Swapchain·VSync·타이밍·전체화면·HDR·오버레이) | 트리·탭 위젯 성숙도 | 유지 상태·비고 |
| :--- | :--- | :--- | :--- | :--- |
| **`winit` + `wgpu` 직접** | `winit` 0.30.13<br>`wgpu` 30.0.1<br>(2026-11-01) | **최상**: DX12/Vulkan 스왑체인 직접 제어, `PresentMode::AutoVsync`/`Mailbox`, Borderless/Exclusive 전체화면, `Rgba16Float` HDR 완벽 지원 (https://docs.rs/wgpu/latest/wgpu/) | **없음**: 자체 UI 위젯 전무. 탐색기·탭·텍스트 입력·스크롤을 바닥부터 구현해야 함 | 매우 활발. 단독 UI 구현은 비현실적이며 UI 레이어와 결합 필수 |
| **`egui` (`eframe` / `egui-wgpu`)** | `egui` 0.37.0<br>`eframe` 0.37.0<br>(2026-11-01) | **상**: `egui::TextureId` 또는 `CallbackTrait`로 탭 영역에 `wgpu` 커스텀 렌더 패스 삽입 가능. 비디오 렉트 위에 반투명 `Area`/`Window` 오버레이 즉시 렌더링 (https://docs.rs/egui/latest/egui/) | **최상**: `egui_tiles` 0.18.0 / `egui_dock` 0.21.1로 드래그 가능한 탭·분할 완벽 지원, `egui_ltreeview` 0.9.0 및 내장 트리로 탐색기 구현 용이 | 매우 활발 (Rerun 후원). 즉시 모드 특성상 재생 중 60fps 루프와 자연스럽게 결합됨 |
| **`iced`** | `iced` 0.14.0/0.14.1<br>(2026-11-01) | **중상**: `iced_wgpu`의 `shader::Pipeline`(`Primitive`)으로 커스텀 WGSL/텍스처 렌더링 가능하나, 외부 하드웨어 핸들 직접 주입 시 내부 `wgpu::Device` 접근 제약이 있음 (https://crates.io/crates/iced) | **중**: 공식 코어에 복합 도킹 탭·가상 트리 뷰가 부족하여 서드파티(`iced_aw`)나 직접 구현 필요 | 활발 (System76 COSMIC 채택). Elm 아키텍처는 고빈도 프레임 동기화 시 메시지 보일러플레이트 발생 |
| **`slint`** | `slint` 1.18.1<br>(2026-11-01) | **중**: 1.x 후반에 WGPU 통합(`RenderingNotifier`/외부 텍스처)이 개선되었으나 DSL 중심 구조로 로우레벨 DXGI 공유 핸들·HDR 스왑체인 세밀 제어는 까다로움 (https://crates.io/crates/slint) | **중**: 기본 `TabWidget`과 리스트는 제공하나 VSCode/IDE 스타일 동적 분할 탭·대규모 파일 트리는 직접 작성 필요 | 매우 활발. GPLv3 / 상용 유료 / Royalty-free Desktop 라이선스 정책 확인 필수 (https://slint.dev/) |
| **`floem`** | `floem` 0.2.0<br>(2026-11-01) | **중하**: `vello`/`wgpu` 기반이나 커스텀 비디오 서피스 제로카피 주입 API가 성숙하지 않음 (https://crates.io/crates/floem) | **중**: Lapce 에디터 유래로 트리·탭 개념은 있으나 범용 위젯 크레이트 생태계가 작음 | 보통. 릴리스 주기가 느리고 비디오 앱 사례 부족 |
| **`makepad`** | `makepad-widgets` 1.0.0<br>(2026-11-01) | **중상**: 자체 GPU 셰이더 DSL과 D3D11 백엔드를 내장해 영상/셰이더 합성 잠재력이 높으나 문서화가 부족함 (https://crates.io/crates/makepad-widgets) | **상**: 자체 IDE(Makepad Studio)용 도킹 탭과 파일 트리를 기본 내장 | 활발 (Robrix 등). 독자 매크로·DSL 학습 곡선이 높음 |
| **`xilem`** | `xilem` 0.4.0<br>(2026-11-01) | **하**: Linebender의 `Vello`(컴퓨트 셰이더 2D 렌더러) 기반으로 일반 비디오 파이프라인 연동이 아직 실험 단계 (https://crates.io/crates/xilem) | **하**: 알파 단계(0.4.0)로 고급 도킹 탭·파일 탐색기 위젯 미성숙 | 활발하나 실험적(Alpha). 상용 수준 플레이어에 부적합 |
| **`windows` 크레이트 (Win32 + D3D11 / DirectComposition 직접)** | `windows` 0.62.2<br>(2026-11-01) | **최상**: `IDXGISwapChain1`, `IDCompositionDevice`, `ID3D11VideoProcessor`, 하드웨어 오버레이(MPO), 프레임 레이턴시 웨이터블 오브젝트 완벽 제어 (https://crates.io/crates/windows) | **최하**: Win32 `SysTreeView32`·`SysTabControl32`는 현대적 다크모드·탭 분할·커스텀 오버레이 구현이 극도로 고통스러움 | Microsoft 공식 유지. 렌더링 계층에만 부분 채택하고 UI 전체 구축용으로는 비권장 |
| **그 밖의 후보 1: `dioxus` (Desktop / Native)** | `dioxus` 0.7.9<br>(2026-11-01) | **하(WebView) / 중하(Blitz)**: 기본 데스크톱 모드는 WebView2 위에서 동작하므로 Rust D3D11 텍스처를 WebView DOM 탭 안에 Zero-Copy로 넣기 어려움 (https://github.com/DioxusLabs/dioxus) | **상(HTML/CSS)**: 웹 생태계로 트리·탭 구현은 쉬우나 네이티브 GPU 비디오 표면 결합이 병목 | 매우 활발. 단, 순수 네이티브 비디오 플레이어 목적에는 부적합 |
| **그 밖의 후보 2: `freya` / `vizia`** | `freya` 0.4.3<br>`vizia` 0.4.0<br>(2026-11-01) | **중하**: `freya`는 Skia 기반, `vizia`는 FemtoVG/Skia 기반으로 D3D11/DX12 비디오 텍스처 Zero-Copy 브리지 구현 부담이 큼 (https://crates.io/crates/freya, https://crates.io/crates/vizia) | **중**: 기본 레이아웃은 갖추었으나 `egui_tiles` 수준의 성숙한 도킹 탭 생태계 부재 | 보통. 범용 오디오 플러그인(`vizia`) 등에는 쓰이나 비디오 플레이어로는 비주류 |

---

## B3 렌더링: wgpu vs D3D11 직접, Zero-Copy 공유 텍스처, YUV→RGB, R2 셰이더 위치

### 1. `wgpu` vs `D3D11` 직접 렌더링 비교
- **`D3D11` (`windows` 크레이트) 직접**: FFmpeg `D3D11VA`나 Media Foundation과 동일한 API 공간에 있으므로 `ID3D11VideoDevice`·`ID3D11VideoContext`를 즉시 호출할 수 있고, Windows MPO(Multiplane Overlay) 및 DirectComposition 스왑체인과의 결합이 가장 직관적이다 (https://learn.microsoft.com/en-us/windows/win32/medfound/direct3d-11-video-apis, 2026-11-01 확인). 그러나 `egui` 같은 모던 Rust UI 렌더러(`wgpu` 기반)와 한 창에서 합성하려면 결국 D3D11↔wgpu 브리지가 필요하다.
- **`wgpu` (Windows 기본 DX12 백엔드)**: UI(`egui-wgpu`)와 동영상 프레임을 단일 렌더 패스(또는 연속 패스)에서 그릴 수 있어 **탭 내부 클리핑, 분할 뷰, 영상 위 반투명 컨트롤 바·자막 오버레이** 처리가 매우 일관적이다. `wgpu` v30.0.1은 `Features::TEXTURE_FORMAT_NV12` 및 `Features::TEXTURE_FORMAT_P010`을 네이티브로 지원한다 (https://docs.rs/wgpu/latest/wgpu/struct.Features.html, 2026-11-01 확인).

### 2. 디코더 D3D11 텍스처를 복사 없이(Zero-Copy) `wgpu`에 표시하는 방법
Windows 하드웨어 디코더(`D3D11VA`)는 디코딩된 프레임을 GPU VRAM 상의 `ID3D11Texture2D`(주로 `DXGI_FORMAT_NV12` 또는 10-bit `DXGI_FORMAT_P010` 텍스처 배열 슬라이스)로 출력한다. 이를 CPU RAM(`av_hwframe_transfer_data`)으로 내리지 않고 `wgpu`(DX12)로 가져오는 표준 경로는 다음과 같다:
1. **공유 텍스처(Staging Shared Texture) 생성**: 디코더 출력 텍스처 배열은 드라이버 내부 제약으로 직접 공유 핸들 생성이 거부될 수 있으므로, 동일 `ID3D11Device`에서 `D3D11_RESOURCE_MISC_SHARED_NTHANDLE | D3D11_RESOURCE_MISC_SHARED_KEYEDMUTEX`(또는 Fence 동기화용 `D3D11_RESOURCE_MISC_SHARED_NTHANDLE`) 플래그를 가진 단일 NV12/P010 2D 텍스처를 생성하고 `CopySubresourceRegion`으로 GPU 내부 복사(VRAM→VRAM, 마이크로초 단위)를 수행한다 (https://learn.microsoft.com/en-us/windows/win32/api/d3d11/ne-d3d11-d3d11_resource_misc_flag, 2026-11-01 확인).
2. **DXGI NT 핸들 내보내기**: `IDXGIResource1::CreateSharedHandle`로 NT `HANDLE`을 생성한다.
3. **D3D12 리소스로 열기 및 `wgpu-hal` 래핑**: `wgpu`의 내부 `ID3D12Device`에서 `OpenSharedHandle`로 `ID3D12Resource`를 얻은 뒤, `wgpu_hal::api::Dx12`를 통해 `wgpu::Texture`로 감싼다 (https://docs.rs/wgpu-hal/latest/wgpu_hal/, 2026-11-01 확인):

```rust
// 10줄 이하 API 핵심 흐름 예시 (windows + wgpu-hal)
let dxgi_res: IDXGIResource1 = shared_d3d11_tex.cast()?;
let nt_handle = unsafe { dxgi_res.CreateSharedHandle(None, DXGI_SHARED_RESOURCE_READ.0, None)? };
let mut d3d12_res: Option<ID3D12Resource> = None;
unsafe { d3d12_device.OpenSharedHandle(nt_handle, &mut d3d12_res)? };
let hal_tex = unsafe { wgpu_hal::dx12::Device::texture_from_raw(d3d12_res.unwrap(), format, dim, 1, 1) };
let wgpu_tex = unsafe { device.create_texture_from_hal::<wgpu_hal::api::Dx12>(hal_tex, &desc) };
```

4. **동기화(Fence / KeyedMutex)**: D3D11 컨텍스트에서 복사 완료 후 `ID3D11DeviceContext4::Signal`로 공유 `ID3D11Fence` 값을 올리고, D3D12 큐에서 해당 Fence를 대기(`Wait`)한 뒤 `wgpu` 렌더 패스에서 샘플링해야 프레임 찢김·이전 프레임 참조를 방지한다.

### 3. YUV→RGB 변환과 R2 셰이더 배치 위치
- **YUV→RGB 변환**: NV12(`Plane 0: R8Unorm`, `Plane 1: Rg8Unorm`) 또는 P010(`Plane 0: R16Unorm`, `Plane 1: Rg16Unorm`) 텍스처에서 각 평면의 `TextureView`(`TextureAspect::Plane0`, `TextureAspect::Plane1`)를 생성하여 WGSL 프래그먼트 셰이더에 바인딩한다. BT.601(SD), BT.709(HD), BT.2020(UHD/HDR) 색 행렬과 Limited(16~235) / Full(0~255) 범위 유니폼을 전달해 선형 RGB로 변환한다.
- **R2 영상 옵션 셰이더를 둘 위치**:
  - **1단계 (디인터레이스 - YUV 또는 초기 선형 RGB 공간)**: 인터레이스 원본(`Top/Bottom Field`)은 크로마 업샘플링 직후 또는 YUV 평면 단계에서 컴퓨트/프래그먼트 셰이더(Bob / Yadif 계열 공간·시간 보간)로 먼저 프로그레시브 프레임으로 복원해야 공간 필터 왜곡이 없다.
  - **2단계 (노이즈 제거·흐리게·선명하게 - 선형 RGB Ping-Pong FBO)**: `Rgba16Float` 중간 렌더 타깃(Ping-Pong 텍스처 2장)을 두고 ① 노이즈 제거(Bilateral / 3D Spatial-Temporal Denoise) → ② 흐리게(Separable 2-Pass Gaussian Blur) → ③ 선명하게(Unsharp Mask / CAS - Contrast Adaptive Sharpening) 순서로 실행한다. 비활성화된 필터 패스는 건너뛰어(Bypass) 오버헤드를 0으로 만든다.
  - **3단계 (밝기·대비·채도 + 톤매핑 - 최종 출력 프래그먼트 패스)**: 별도 패스 없이 마지막 화면 출력(또는 최종 탭 텍스처 기록) 패스 안에서 단일 행렬/수식으로 처리한다.

---

## B4 디코딩 바인딩 선택(libmpv / FFmpeg / GStreamer)과 통합 방법

### 1. 3대 디코딩 바인딩 비교 (2026-11-01 확인)
| 항목 | `libmpv2` v6.0.0 (`libmpv`) | `ffmpeg-next` v9.0.0 (`FFmpeg`) | `gstreamer` v0.25.4 (`GStreamer`) |
| :--- | :--- | :--- | :--- |
| **출처 URL** | https://crates.io/crates/libmpv2 | https://crates.io/crates/ffmpeg-next | https://crates.io/crates/gstreamer |
| **추상화 수준** | 완제품 플레이어 엔진 (Demux+Decode+Sync+Sub+Render) | 저수준 코덱/포맷 라이브러리 (Demux+Decode+Filter) | 모듈형 파이프라인 그래프 프레임워크 (`playbin3`) |
| **Windows D3D11 통합** | `mpv_render_context` (`gpu-next` D3D11/OpenGL/Vulkan) 또는 `wid` | `AV_HWDEVICE_TYPE_D3D11VA`로 `ID3D11Texture2D` 직접 추출 | `gst-d3d11` (`d3d11h264dec`, `GstD3D11Memory`, `appsink`) |
| **자막(SRT/ASS)** | `libass` 완벽 내장 (폰트·애니메이션·위치 자동) | `libavcodec` 자막 디코딩만 제공, ASS 렌더러 별도 연동 필요 | `assrender` 플러그인 제공 (Windows 배포 시 플러그인 번들링 무거움) |
| **Windows 배포 난이도** | **낮음**: `libmpv-2.dll` 단일(또는 소수) DLL 배포 | **중간**: `avcodec`, `avformat`, `avutil`, `swresample` 등 DLL 5~7개 | **높음**: 수십 개 플러그인 DLL 및 GLib/GObject 런타임 동반 배포 |

### 2. 선택한 스택에서의 통합 방법 3가지 비교
1. **Render API / 텍스처 렌더 타깃 방식 (권장 - `libmpv` `mpv_render_context` 또는 `ffmpeg-next` 공유 텍스처)**:
   - 각 동영상 탭마다 오프스크린 공유 텍스처(`ID3D11Texture2D` ↔ `wgpu::Texture`)를 할당하고, 해당 텍스처를 `egui::TextureId`로 등록해 `ui.image()`로 탭 영역 안에 그린다.
   - **장점**: 탭 분할, 탭 전환, 반응형 리사이즈, 영상 위 `egui` 반투명 컨트롤 오버레이, 커스텀 WGSL 셰이더 적용이 완벽히 동작한다.
2. **자식 창(`HWND` / `wid`) 임베딩 방식 (비권장)**:
   - Win32 자식 창(`WS_CHILD`)을 만들어 `libmpv`의 `wid` 옵션에 넘기는 방식이다.
   - **단점**: Windows의 악명 높은 **Airspace 문제**(자식 `HWND` 위에 부모 창의 `wgpu`/`egui` 반투명 오버레이 UI를 그릴 수 없음)가 발생하며, 탭 드래그·분할·애니메이션 시 창 위치 동기화 지연(flickering)이 생긴다.
3. **CPU 프레임 전달(`readback` / `write_texture`) 방식 (폴백 전용)**:
   - 디코딩된 RGBA/YUV 바이트 배열을 `queue.write_texture()`로 매 프레임 업로드하는 방식이다.
   - 4K 60fps 기준 프레임당 약 33MB(RGBA) 대역폭(초당 ~2GB/s)을 소모하므로 하드웨어 가속 실패 시(R1 폴백)에만 제한적으로 사용한다.

---

## B5 파일 탐색기 + 탭 UI 구현 설계

### 1. 트리·탭·분할 레이아웃 구조
- **좌측 파일 탐색기 (`egui::SidePanel::left`)**:
  - 드라이브/폴더 계층은 `egui_ltreeview` v0.9.0 (https://crates.io/crates/egui_ltreeview, 2026-11-01 확인) 또는 `egui::CollapsingHeader` + `ScrollArea::vertical().show_rows()` 가상 스크롤로 구현한다.
  - 수만 개 파일이 있는 폴더도 지연 로딩(Lazy expansion)과 확장자 필터(`.mp4`, `.mkv`, `.webm`, `.ts`, `.avi`)를 적용해 UI 프레임 드랍을 막는다.
  - 파일 더블클릭 시 활성 탭에서 열기, 휠클릭 또는 `Ctrl+클릭` 시 새 탭에서 열기, 탭 영역으로 드래그 앤 드롭 시 분할 뷰 생성을 지원한다.
- **우측 다중 탭·분할 영역 (`egui::CentralPanel` + `egui_tiles` v0.18.0)**:
  - `egui_tiles::Tree<TabState>`를 사용해 탭 그룹(Tabs), 수평 분할(Horizontal Linear), 수직 분할(Vertical Linear), 그리드(Grid) 레이아웃을 모두 기본 지원한다 (https://docs.rs/egui_tiles/latest/egui_tiles/, 2026-11-01 확인).

### 2. 탭 안 동영상 표면 및 오버레이 UI
- `egui_tiles::Behavior::pane_ui(ui, tile_id, tab_state)` 내부에서:
  1. `let rect = ui.available_rect_before_wrap();`로 탭의 비디오 영역을 계산하고 원본 종횡비(Aspect Ratio, Letterbox/Pillarbox)에 맞춘 `video_rect`를 산출한다.
  2. `ui.painter().image(tab_state.wgpu_texture_id, video_rect, uv_rect, Color32::WHITE)`로 비디오 텍스처를 그린다.
  3. 마우스가 `rect` 내부에서 움직이거나 최근 2.5초 이내 입력이 있으면 `rect` 하단에 반투명 OSD 바(탐색 슬라이더, 재생/일시정지, 음량, 트랙, R1/R2 팝업 버튼)를 `ui.allocate_ui_at_rect()`로 겹쳐 그린다.

### 3. 숨김 탭(비활성 탭)·여러 탭 동시 재생 처리 정책
- **숨김 탭(가려진 탭, `Behavior::pane_ui`가 해당 프레임에 호출되지 않는 탭)**:
  - 기본 정책: 다른 탭으로 전환되어 화면에서 가려진 탭은 **자동 일시정지(Auto-Pause)**하거나, 사용자 설정("백그라운드 오디오 계속 재생")이 켜진 경우에만 **오디오 디코딩만 유지하고 비디오 디코딩/셰이더 패스(`Video Render Bypass`)를 중단**한다(다음 키프레임까지 비디오 패킷 스킵 또는 렌더 텍스처 갱신 생략).
  - 메모리 관리: 비활성 탭이 5분 이상 유지되면 공유 스테이징/Ping-Pong FBO 텍스처를 해제하고 마지막 프레임 썸네일(저해상도)만 남겨 VRAM 고갈을 방지한다.
- **여러 탭 동시 재생(분할 화면 - Split View)**:
  - 화면에 동시에 보이는 활성 탭(Pane)이 2~4개일 때는 각 탭이 독립된 `PlayerInstance` 워커 스레드를 가지며 동시에 디코딩·렌더링한다.
  - **오디오 포커스 정책**: 여러 탭이 동시 재생 중일 때 오디오 혼선을 막기 위해 **"현재 포커스된(마지막 클릭한) 탭만 오디오 출력(Active Audio Focus)"** 모드를 기본값으로 두고, 탭 헤더의 스피커 아이콘을 눌러 개별 탭 음소거 해제(Multi-Audio)를 허용한다.
  - **하드웨어 디코더 세션 제한 대응**: GPU의 D3D11VA 동시 디코딩 세션 한계(예: 4K 다중 스트림 8개 초과 시)로 초기화 실패 시, R1 폴백 로직에 의해 추가 탭은 자동으로 소프트웨어 디코딩으로 전환한다.

### 4. 재사용할 파일 시스템·UI 보조 크레이트 (2026-11-01 확인)
- `notify` v8.2.1 (https://crates.io/crates/notify): 탐색기에서 열린 디렉터리의 파일 생성·삭제·이름 변경 실시간 감지 (`ReadDirectoryChangesW` 백엔드).
- `rfd` v0.17.2 (https://crates.io/crates/rfd): Windows 네이티브 폴더 열기/자막 파일 추가 대화상자 (`IFileDialog`).
- `walkdir` v2.5.0 / `ignore` v0.4.23 (https://crates.io/crates/walkdir): 백그라운드 스레드 디렉터리 빠른 스캔.

---

## B6 플레이어 코어와 UI 분리 구조 (크레이트·스레드·채널)

### 1. Cargo 워크스페이스 크레이트 구성 (4-Crate 분리)
```text
rust-video-player/
├── crates/
│   ├── player-core/       # UI 무관 미디어 엔진: Demux, HW/SW Decode(R1), A/V Sync, Subtitle, Audio(cpal)
│   ├── video-render/      # D3D11↔DX12(wgpu-hal) Zero-Copy 브리지, YUV→RGB 및 R2 WGSL 후처리 셰이더 파이프라인
│   ├── ui-explorer-tabs/  # egui + egui_tiles 탭/분할 매니저, 파일 탐색기 트리(notify 연동), OSD 오버레이 위젯
│   └── app-shell/         # winit 이벤트 루프, 창/전체화면 전환, 단축키 라우팅, 설정 저장
```

### 2. 스레드 모델 (탭당 격리 + 중앙 렌더러)
- **Main(UI/Render) 스레드 (`winit` EventLoop)**:
  - `egui` 레이아웃 계산, 활성 탭들의 공유 비디오 텍스처에 대한 R2 WGSL 패스 및 스왑체인 `present()` 실행. 블로킹 I/O나 디코딩은 절대 수행하지 않는다.
- **Directory Watcher 스레드 (1개)**:
  - `notify` 이벤트를 받아 디바운싱 후 UI 스레드로 파일 트리 변경 스냅샷을 전달한다.
- **탭별 `PlayerEngine` 워커 스레드 세트 (활성 탭당 생성)**:
  - `libmpv` 사용 시: mpv 내부 디코딩/오디오 스레드 풀 + 이벤트 수신 루프 1개.
  - `ffmpeg-next` 사용 시: ① Demux 스레드(패킷 큐 공급) ② Video Decode 스레드(D3D11VA/SW 디코딩 및 PTS 스케줄링) ③ Audio Decode & `cpal` 콜백 스레드(마스터 클럭 제공).

### 3. 명령·이벤트 채널 및 상태 공유 설계
- **UI → Core 명령 채널**: `crossbeam-channel` v0.5.17 (https://crates.io/crates/crossbeam-channel, 2026-11-01 확인)의 `Sender<PlayerCommand>` (bounded 64).
- **Core → UI 이벤트 채널**: `crossbeam_channel::Receiver<PlayerEvent>` + `egui::Context::request_repaint()` 호출로 새 프레임 준비 시 즉시 UI 스레드를 깨운다.
- **고빈도 상태(재생 위치·음량·R2 파라미터)**: 채널 오버헤드 없이 `Arc<SharedPlayerState>` 내부의 `AtomicU64`(PTS 마이크로초, `f64::to_bits`로 R2 슬라이더 강도 저장) 및 `AtomicBool`로 Lock-Free 읽기/쓰기를 수행한다.

```rust
pub enum PlayerCommand {
    Open { path: std::path::PathBuf, start_pts_ms: u64 },
    SetPaused(bool),
    Seek { target_ms: u64, exact: bool },
    SetSpeed(f64),                       // 0.25 ~ 4.0
    SetHwAcceleration(bool),             // R1: 재생 중 HW/SW 전환
    UpdateVideoFilter(VideoFilterConfig),// R2: 실시간 필터 ON/OFF 및 강도
    SelectTrack { kind: TrackKind, id: i32 },
}
```

---

## B7 R1·R2·PotPlayer 주요 기능별 구현 방법과 난이도

| 분류 | 세부 기능 | 구현 방법 (`스택 1: libmpv2` / `스택 2: ffmpeg-next + wgpu`) | 난이도 |
| :--- | :--- | :--- | :---: |
| **R1** | **GPU 가속(HW 디코딩) 재생 중 ON/OFF** | • **스택 1**: `mpv.set_property("hwdec", "d3d11va")` ↔ `"no"` 런타임 호출. mpv가 현재 PTS에서 내부 디코더를 즉시 재초기화함 (https://mpv.io/manual/master/#options-hwdec)<br>• **스택 2**: 현재 재생 `pts` 저장 → 패킷 큐에서 해당 `pts` 이전 최근 키프레임 탐색 → 새 `AVCodecContext`(D3D11VA 또는 CPU 멀티스레드) 생성 → `pts`까지 고속 디코딩(드랍) 후 렌더 파이프라인 재연결 | 스택 1: **하**<br>스택 2: **상** |
| **R1** | **HW 디코딩 실패 시 SW 자동 전환(Fallback)** | • **스택 1**: `hwdec=auto-safe` 설정 시 미지원 프로파일(예: 구형 GPU의 AV1 10-bit)이나 드라이버 오류 시 자동 SW 전환, `hwdec-current` 속성 관찰로 UI에 `"HW"` / `"SW"` 배지 표시<br>• **스택 2**: `avcodec_open2` 실패, `AV_PIX_FMT_D3D11` 협상 실패, 또는 `avcodec_receive_frame` 에러 시 즉시 SW 디코더로 재생성 후 토스트 알림 발생 | 스택 1: **하**<br>스택 2: **중** |
| **R2** | **밝기·대비·채도 (실시간 조절)** | • 공통: 마지막 WGSL 프래그먼트 셰이더에서 유니폼 버퍼(`brightness: -1.0..1.0`, `contrast: 0.0..2.0`, `saturation: 0.0..2.0`)를 매 프레임 갱신하여 픽셀당 5줄 수식으로 즉시 반영 (또는 `libmpv` `brightness`/`contrast`/`saturation` 프로퍼티 연동) | **하** |
| **R2** | **선명하게 (Sharpen) / 흐리게 (Blur)** | • **WGSL 셰이더 (`video-render`)**: `Rgba16Float` 중간 텍스처에서 흐리게는 2-Pass Separable Gaussian Blur(반경 `sigma` 유니폼), 선명하게는 Contrast Adaptive Sharpening(CAS) 또는 Unsharp Mask(`amount` 유니폼) 적용. `libmpv`에서는 `glsl-shaders` 동적 교체 또는 `wgpu` 후처리 패스에서 처리 | **중** |
| **R2** | **노이즈 제거 (Denoise)** | • **WGSL / D3D11**: 공간 노이즈 제거(Edge-preserving Bilateral Filter 셰이더) 또는 `ID3D11VideoContext::VideoProcessorSetStreamFilter(..., D3D11_VIDEO_PROCESSOR_FILTER_NOISE_REDUCTION, TRUE, level)` 하드웨어 필터 결합 (https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videocontext-videoprocessorsetstreamfilter) | **중상** |
| **R2** | **디인터레이스 (Deinterlace)** | • **스택 1**: `deinterlace=yes` 프로퍼티 토글 시 `d3d11vpp`(GPU 하드웨어 모션 적응형 디인터레이스) 또는 `bwdif` GPU 필터 즉시 동작<br>• **스택 2**: `ID3D11VideoProcessor`의 `VideoProcessorSetStreamFrameFormat`을 통해 GPU 하드웨어 디인터레이스를 수행하거나 WGSL 컴퓨트 셰이더(Yadif/Bob) 실행 | 스택 1: **하**<br>스택 2: **상** |
| **재생** | **컨테이너·코덱 (MP4/MKV/WebM/TS, H.264/HEVC/VP9/AV1, AAC/AC3/Opus)** | • 두 스택 모두 FFmpeg(`libavformat`/`libavcodec`) 기반이므로 전 포맷 기본 지원. 배포 시 `dav1d`(AV1 고속 디코더)가 포함된 FFmpeg/libmpv 빌드 사용 | **하** |
| **재생** | **탐색(키프레임/정밀), 배속(0.25~4x), 음량·피치 보정** | • **스택 1**: `seek <sec> exact`/`keyframes`, `speed` 프로퍼티(내장 `scaletempo2`로 피치 유지), `volume`/`ao-volume` 즉시 사용<br>• **스택 2**: `av_seek_frame` 후 목표 PTS까지 디코딩 스킵, 배속 시 오디오 리샘플러(`rubato` 또는 `libavfilter`의 `atempo`) 체인 구성 필요 | 스택 1: **하**<br>스택 2: **중상** |
| **자막** | **SRT / ASS 자막 및 다중 트랙 선택** | • **스택 1**: `sid`/`aid`/`vid` 프로퍼티로 오디오·자막·비디오 트랙 즉시 전환, `libass`가 렌더 컨텍스트 내부에서 ASS特效 자막까지 완벽 합성<br>• **스택 2**: SRT는 `subparse` 후 `egui` 텍스트로 렌더링 가능하나, 복잡한 ASS 조판은 `libass` C 바인딩을 별도로 붙여 알파 텍스처로 블렌딩해야 함 | 스택 1: **하**<br>스택 2: **상** |
| **창** | **전체 화면 (단일 탭 전체 화면)** | • 더블클릭 또는 `Enter`/`F` 입력 시 `egui` 좌측 탐색기 패널과 탭 바를 숨기고(`show_chrome = false`), 현재 활성 탭 비디오 뷰포트를 창 전체에 채우며 `winit::window::Window::set_fullscreen(Some(Fullscreen::Borderless(None)))` 호출 | **하** |

---

## B8 후보 스택 비교표

| 평가 항목 | **권장 1: `winit` + `wgpu` + `egui`(`egui_tiles`) + `libmpv2`** | **권장 2: `winit` + `wgpu`(`wgpu-hal`) + `egui`(`egui_tiles`) + `ffmpeg-next`** | **대안 A: `iced` + `ffmpeg-next` / `gstreamer`** | **대안 B: `windows` (Win32 + D3D11/DirectComposition 직접)** |
| :--- | :--- | :--- | :--- | :--- |
| **예상 개발 기간 (추정)** | **약 2~3개월** (1인 기준, 코어 플레이어 완성도 즉시 확보) | **약 4~6개월** (A/V 동기화, 정밀 탐색, ASS 자막, HW 폴백 직접 구현) | **약 5~7개월** (도킹 탭·트리 위젯 직접 구축 및 렌더 브리지 비용) | **약 6~9개월+** (모던 다크모드 트리·분할 탭·OSD UI 직접 구축 비용 극대) |
| **렌더링·재생 성능** | **매우 높음** (`gpu-next` + `wgpu` 후처리, 4K60~8K 재생 원활) | **최상** (D3D11VA → DX12 NT 핸들 Zero-Copy + 단일 WGSL 파이프라인) | **높음** (`iced_wgpu` 경유 시 추가 텍스처 복사 가능성 있음) | **최상** (OS 컴포지터 직결, 최소 메모리·전력 소모) |
| **R1 (HW/SW 실시간 전환·폴백)** | **최상** (`hwdec` 프로퍼티 변경 1줄로 무중단 전환 및 자동 폴백) | **상** (키프레임 재탐색 + 디코더 컨텍스트 교체 상태 머신 직접 구현) | **중상** (GStreamer 파이프라인 재협상 또는 FFmpeg 재초기화 필요) | **상** (FFmpeg D3D11VA 직접 제어, 폴백 로직 수동 구현) |
| **R2 (6종 영상 필터 실시간 제어)** | **최상** (`libplacebo` 내장 필터 + `wgpu` WGSL 후처리 패스 병행) | **최상** (모든 필터를 WGSL 셰이더 및 `ID3D11VideoProcessor`로 세밀 제어) | **중** (커스텀 `shader::Primitive` 내 파이프라인 수동 관리) | **최상** (`ID3D11VideoProcessor` + HLSL 픽셀/컴퓨트 셰이더 직접 제어) |
| **배포 크기 (Windows x64, 추정)** | **약 45~65 MB** (`exe` ~10MB + `libmpv-2.dll` ~35~55MB) | **약 35~55 MB** (`exe` ~12MB + FFmpeg 필수 DLL 세트 ~25~40MB) | **약 40~120 MB** (GStreamer 선택 시 플러그인 포함 100MB 초과) | **약 25~40 MB** (`exe` ~5MB + FFmpeg DLL) |
| **라이선스 위험** | UI 크레이트 MIT/Apache-2.0. **`libmpv`는 LGPLv2.1+ 빌드 동적 링크 필수** (`--enable-lgpl` 확인 필요, 기본 빌드는 GPLv2+인 경우 많음) | UI 크레이트 MIT/Apache-2.0. **FFmpeg는 LGPLv2.1+ 동적 링크 필수** (`--enable-gpl` 및 `x264`/`x265` 제외 빌드 확인) | `iced` MIT, `gstreamer` LGPLv2+. 플러그인 중 `gst-plugins-ugly` 제외 필수 | `windows` MIT/Apache-2.0. 디코더(FFmpeg) LGPL 동적 링크 조건만 관리 |
| **유지보수 위험** | **낮음**: 코덱·자막 버그를 업스트림 mpv/FFmpeg가 해결함 | **중간**: FFmpeg 메이저 업데이트 시 `AVHWFramesContext` FFI 유지보수 필요 | **중상**: `iced` 버전업 시 위젯/렌더러 API 변경 폭이 큼 | **높음**: 대량의 `unsafe` Win32/COM 코드 직접 유지보수 부담 |

---

## B9 권장 문서 목차

독립 문서 묶음(`D:\projects\docs\guides\rust-video-player-native\`) 작성을 위한 권장 목차는 다음과 같다:

1. **`00-overview-and-architecture.md` — 아키텍처 개요 및 기술 스택 선정**
   - 1.1 설계 목표 (PotPlayer급 재생 경험 + 탐색기·다중 탭 워크벤치 + R1/R2 필수 요건)
   - 1.2 권장 스택 아키텍처 다이어그램 (`winit` + `wgpu` + `egui_tiles` + `libmpv2` / `ffmpeg-next`)
   - 1.3 4-Crate 워크스페이스 구조 및 스레드/채널(`crossbeam-channel` + `Atomic`) 통신 규약
2. **`01-windowing-and-explorer-tabs-ui.md` — 창 관리, 파일 탐색기 트리, 다중 탭 UI 구현**
   - 2.1 `winit` v0.30 + `wgpu` v30 스왑체인 초기화 (HDR `Rgba16Float`, VSync, 전체 화면 전환)
   - 2.2 좌측 파일 탐색기 구현 (가상 스크롤 트리, `notify` v8 실시간 감시, 드래그 앤 드롭)
   - 2.3 `egui_tiles` v0.18 기반 다중 탭·화면 분할(Split View) 및 탭 내 플로팅 OSD 오버레이
   - 2.4 다중 탭 동시 재생 시 오디오 포커스 정책과 숨김 탭(Background Tab) VRAM 절약 전략
3. **`02-zero-copy-decoding-and-r1-hwdec.md` — 디코딩 엔진 통합 및 R1(하드웨어 가속 전환·폴백)**
   - 3.1 트랙 A (`libmpv2` `mpv_render_context` + `gpu-next`) 통합 상세
   - 3.2 트랙 B (`ffmpeg-next` `D3D11VA` → DXGI NT Shared Handle → `wgpu-hal` DX12) Zero-Copy 구현
   - 3.3 R1 구현: 재생 중 무중단 HW↔SW 디코딩 전환 상태 머신과 드라이버 실패 시 자동 폴백
4. **`03-video-rendering-and-r2-shaders.md` — YUV→RGB 색공간 파이프라인 및 R2 실시간 영상 필터**
   - 4.1 NV12 / P010 다중 평면 `TextureView` 바인딩 및 BT.601/709/2020·HDR10 색 변환
   - 4.2 R2 WGSL 셰이더 파이프라인 설계 (디인터레이스 → 노이즈 제거 → 블러 → 샤프닝 → 밝기/대비/채도)
   - 4.3 `ID3D11VideoProcessor` 하드웨어 스트림 필터(`NoiseReduction`, `EdgeEnhancement`, `Deinterlace`) 하이브리드 활용
5. **`04-playback-features-and-subtitles.md` — PotPlayer 핵심 기능(탐색·배속·트랙·자막·오디오) 구현**
   - 5.1 키프레임/정밀 탐색(Seek Scrubbing), 프레임 단위 이동, 피치 유지 배속 재생
   - 5.2 다중 비디오/오디오/자막 트랙 선택 및 외부 SRT/ASS 자막 렌더링(`libass` 연동)
6. **`05-packaging-and-licensing-windows.md` — Windows 11 빌드, DLL 번들링 및 LGPL 규정 준수**
   - 6.1 MSVC 정적/동적 링킹 구성, `libmpv-2.dll` / FFmpeg LGPL 커스텀 빌드 체크리스트, 설치 파일 패키징

---

## B10 확신 낮은 부분

1. **`libmpv` `mpv_render_context`의 Windows D3D11 네이티브 백엔드 파라미터 안정성 (미확인/추정)**:
   - `libmpv`의 `vo=gpu-next`(`libplacebo`) 자체는 D3D11과 Vulkan을 완벽히 지원하지만, C API인 `mpv_render_context_create`를 통해 호스트 앱(`wgpu` DX12)과 텍스처를 공유할 때 `MPV_RENDER_API_TYPE_OPENGL`(WGL_NV_DX_interop2 경유) 대비 D3D11/Vulkan 직접 공유 파라미터가 모든 Windows GPU 드라이버(특히 구형 Intel 내장 GPU)에서 결함 없이 동작하는지 실제 프로토타입 검증이 필요하다 (https://github.com/mpv-player/mpv/blob/master/libmpv/render.h, 2026-11-01 확인). 반면 `ffmpeg-next` + `D3D11VA` 공유 텍스처 경로는 Win32/DXGI 표준 API만으로 구성되어 동작이 확실하다.
2. **4K 60fps 다중 탭(4분할 이상) 동시 재생 시 R2 고비용 필터(3D Denoise)의 GPU 부하 한계 (추정)**:
   - 밝기·대비·채도나 3x3 Unsharp Mask는 4K 60fps 4개 탭 동시 실행 시에도 GPU 점유율이 미미하나, 반경이 큰 Bilateral/Temporal Denoise와 Gaussian Blur를 4개 탭에서 동시에 켜면 중급 내장 그래픽(Intel Iris Xe 등)에서 16.6ms 프레임 예산을 초과할 가능성이 있다(추정). 분할 재생 시 비포커스 탭의 R2 필터 패스 해상도를 1/2로 낮추는 적응형 품질 제어(Adaptive Quality) 도입 여부는 실측 후 결정해야 한다.
3. **`wgpu` 메이저 버전 업그레이드 시 `wgpu-hal` 내부 API 변동성 (확인됨, 주의 필요)**:
   - `wgpu`는 안전한 상위 API보다 `wgpu_hal::api::Dx12`의 `texture_from_raw` 및 `create_texture_from_hal` 시그니처가 메이저 버전마다 자주 변경된다 (https://docs.rs/wgpu-hal/latest/wgpu_hal/, 2026-11-01 확인). 따라서 `video-render` 크레이트 내부의 Zero-Copy 브리지 모듈(약 150줄 내외)에 `wgpu` 버전을 고정(Pinning)하고 격리해야 한다.
