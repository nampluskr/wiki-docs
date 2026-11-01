# Rust 동영상 플레이어 구현 가이드 조사 보고서 (scouter: argon)

## 요약 (10줄)
1. **gpui 1순위 권장안**: 범용 코덱·자막·탐색·필터를 즉시 확보하려면 **libmpv (`libmpv2` 6.0.0)** 백엔드에 오프스크린 렌더 또는 D3D11 텍스처 공유를 결합하고, 완전 자체 제어가 목적이면 **FFmpeg (`ffmpeg-the-third` 6.0.0+ffmpeg-9.0 / `rsmpeg` 0.18.0)** + D3D11VA + HLSL/D3D11 비디오 프로세서를 권장한다.
2. **Tauri 2 2순위 권장안**: WebView2 `<video>` 단독으로는 MKV/TS, AC3, ASS 자막, R1(런타임 HW/SW 디코더 강제 전환)을 만족하지 못하므로 **libmpv (`--wid` 자식 HWND 또는 로컬 IPC 플러그인)** 조합을 권장한다.
3. **R1(GPU 가속 ON/OFF) 핵심**: Windows 11에서는 **D3D11VA**가 표준이며, `libmpv`는 `set_property("hwdec", "auto-safe" | "no")` 한 줄로 재생 중 즉시 전환 및 실패 시 SW 자동 폴백을 지원한다.
4. **R2(실시간 영상 옵션) 핵심**: 밝기·대비·채도·선명·블러·노이즈 제거는 **출력단 GPU 셰이더(mpv `glsl-shaders`/복합 속성, 또는 D3D11/wgpu 픽셀 셰이더)**로 처리해야 재생 끊김 없이 4K60 실시간 슬라이더 조절이 가능하며, 디인터레이스는 **`d3d11vpp` (`ID3D11VideoProcessor`)** 또는 `bwdif`를 쓴다.
5. **gpui 렌더링 병목**: `workbench-gpui`(`gpui-kit =0.8.1`, 내부 `gpui-pre 0.3.10`)의 Windows 백엔드(`DirectXRenderer`, DX11)에서 `RenderImage` CPU 경로로 4K60 RGBA(약 1.99 GB/s)를 매 프레임 업로드하면 병목이 발생하므로 GPU 내 YUV→RGB 변환 또는 D3D11 텍스처 직접 바인딩이 필수다.
6. **Tauri WebView2 병목**: `convertFileSrc`(`asset://`)는 대용량 동영상의 HTTP `206 Partial Content` Range 탐색이 불안정하고, Rust→JS IPC로 4K 원시 프레임을 보내는 것은 대역폭 한계로 불가능하다.
7. **오디오/자막**: 자체 파이프라인 구성 시 오디오는 **`cpal` 0.18.2**(WASAPI 공유 모드 + `rubato` 5.0.1 리샘플링)를 오디오 마스터 시계로 삼고, ASS 자막은 `libass` 또는 순수 Rust `ass-renderer`(0.1.2)로 처리한다.
8. **라이선스 주의**: FFmpeg에서 `hqdn3d`, `eq`, `pp` 필터를 켜면 `--enable-gpl`이 강제되어 앱 전체에 GPL 의무가 발생하므로 상용/독점 배포 시 LGPL 빌드(`unsharp`, `gblur`, `yadif`, `bwdif`, `d3d11va`)와 자체 GPU 셰이더를 조합해야 한다.
9. **배포 및 크기**: Windows에서는 MSVC 툴체인(`stable-msvc`)에 맞춰 BtbN(FFmpeg shared), shinchiro(libmpv), GStreamer 공식 MSVC 런타임 DLL을 실행 파일 옆에 번들하며, 최소 구성 시 약 35~80 MB가 추가된다.
10. **다중 탭·분할 뷰 대응**: 비활성(숨김) 탭은 디코더의 비디오 트랙 디코딩이나 렌더 업로드를 중단해 VRAM(D3D11 디코더 출력 텍스처 배열) 고갈을 막고, 동시 재생은 분할 Pane에 보이는 탭(최대 2~4개)으로 제한해야 한다.

---

## Q1 권장안

### 1. gpui 탭 재생 권장 조합과 이유
- **권장 1안 (실용·완성도 최우선 — PotPlayer급 범용성)**: **libmpv (`libmpv2` 6.0.0) + D3D11 렌더/자식 뷰 결합**
  - **이유**: 요구된 컨테이너(MP4/MKV/WebM/TS), 코덱(H.264/HEVC/VP9/AV1, AAC/AC3/Opus), 자막(SRT/ASS 내장 `libass`), 트랙 선택, 정밀 탐색이 단일 엔진에 완비되어 있다 (출처: https://mpv.io/manual/stable/). R1은 `hwdec` 속성(`auto-safe` ↔ `no`)으로 재생 중 즉시 전환되고, R2는 내장 속성(`brightness`, `contrast`, `saturation`)과 `vf=d3d11vpp` 및 `glsl-shaders`로 지연 없이 동작한다 (출처: https://mpv.io/manual/stable/#options-hwdec).
- **권장 2안 (네이티브 UI 합성·LGPL 통제 우선)**: **FFmpeg (`ffmpeg-the-third` 6.0.0+ffmpeg-9.0 또는 `rsmpeg` 0.18.0) + D3D11VA + `ID3D11VideoProcessor`/HLSL 셰이더 + `cpal` 0.18.2**
  - **이유**: gpui의 `DirectXRenderer`(D3D11 기반)와 같은 그래픽 API(Direct3D 11)를 공유하므로 `AV_PIX_FMT_D3D11`(`ID3D11Texture2D` 배열 슬라이스)을 CPU 복사 없이 비디오 프로세서나 픽셀 셰이더로 넘겨 gpui 탭 요소 안에서 네이티브 오버레이 UI와 자연스럽게 합성할 수 있다 (출처: https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h.html). 단, A/V 동기화·탐색·ASS 자막 렌더러를 직접 구현해야 한다.

### 2. Tauri 2 탭 재생 권장 조합과 이유 (웹뷰 `<video>` 한계 포함)
- **권장 조합**: **libmpv (`libmpv2` 또는 `tauri-plugin-mpv`) + 자식 HWND(`--wid`) 탭 영역 동기화 + WebView2 투명 컨트롤/별도 제어바** (또는 로컬 루프백 HTTP 서버 + WebCodecs/WASM 보조)
  - **이유**: Tauri 2 백엔드(Rust)에서 `libmpv` 인스턴스를 띄우고 탭 컨테이너 DOM의 `getBoundingClientRect()` 좌표에 맞춰 자식 창(HWND) 위치를 동기화하면, IPC 프레임 복사 비용(0 바이트)으로 4K60 재생과 R1·R2·ASS·AC3를 모두 달성한다 (출처: https://github.com/nini22P/tauri-plugin-mpv).
- **웹뷰 `<video>` 단독으로 범용·R1·R2가 가능한가? → 불가능 (부분 충족에 그침)**
  - **범용 코덱/컨테이너 불가**: WebView2(Chromium Edge 기반)는 특허·라이선스 문제로 **AC3(Dolby Digital) 오디오를 기본 디코딩하지 못하며**, **HEVC**는 사용자 PC에 Microsoft Store의 'HEVC Video Extensions'가 설치되어 있어야만 동작한다. 또한 MPEG-TS(`.ts`) 컨테이너와 내장 **ASS 자막(고급 스타일·애니메이션)**을 `<video>` 기본 기능으로 재생할 수 없다 (출처: https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/overview-features-apis).
  - **R1(GPU 가속 런타임 ON/OFF) 불가**: Chromium의 하드웨어 디코딩은 브라우저 프로세스 기동 플래그(`--disable-accelerated-video-decode`)로만 제어되며, JS API로 특정 `<video>` 탭에서 재생 중 HW/SW 디코딩을 동적으로 전환할 수 없다 (출처: https://www.chromium.org/developers/how-tos/run-chromium-with-flags/).
  - **R2(영상 옵션) 반쪽 지원**: CSS `filter: brightness() contrast() saturate() blur()`와 SVG `feConvolveMatrix`(선명하게)로 일부 흉내 낼 수 있으나, **디인터레이스(`yadif`/`d3d11vpp`)와 시공간 노이즈 제거(`hqdn3d`)는 `<video>`에서 지원하지 않는다** (출처: https://developer.mozilla.org/en-US/docs/Web/CSS/filter).

---

## Q2 파이프라인 구조

### 1. 전체 데이터 흐름 및 스레드 모델 (`demux → decode → filter → 큐 → 출력`)
FFmpeg/자체 구현 기준 표준 멀티스레드 아키텍처는 최소 4개 스레드로 분리해야 UI 스레드 블로킹을 막을 수 있다 (출처: https://github.com/FFmpeg/FFmpeg/blob/master/fftools/ffplay.c).
1. **Demux 스레드**: `av_read_frame()`으로 컨테이너에서 `AVPacket`을 읽어 오디오/비디오/자막 패킷 큐(`PacketQueue`)에 분배한다. 큐가 상한(예: 총 15~25 MB 또는 256 패킷)에 도달하면 조건 변수/채널에서 대기(Backpressure)한다.
2. **Video Decode & Filter 스레드**: 비디오 `PacketQueue`에서 패킷을 꺼내 `avcodec_send_packet()` / `avcodec_receive_frame()`을 수행하고, 필요 시 `AVFilterGraph`나 GPU 프로세서를 거쳐 `FrameQueue`(크기 3~8 프레임, 링버퍼)에 넣는다.
3. **Audio Decode & Callback 스레드**: 오디오 패킷을 디코딩해 `swr_convert()` 또는 `rubato`로 출력 장치 포맷(예: 48kHz f32 interleaved)에 맞춘 뒤 링버퍼(`rtrb` 등 Lock-free SPSC)에 적재하고, WASAPI(`cpal`) 오디오 콜백이 이를 소비한다.
4. **UI / Render 스레드(gpui/Tauri)**: VSync 또는 프레임 타이머에 맞춰 `FrameQueue`의 head 프레임 PTS를 오디오 시계와 비교해 화면에 그린다.

### 2. 상태 기계 (State Machine)
- 상태: `Idle` → `Opening` → `Playing` ⇄ `Paused` / `Seeking` / `Buffering` → `Ended` / `Error(FallbackToSw)`.
- R1(HW→SW 전환) 또는 R2(필터 그래프 재구성) 발생 시: 현재 `pts`와 `Paused/Playing` 상태를 보존한 채 디코더 컨텍스트만 재초기화하거나(`Reconfiguring`), 출력 셰이더 유니폼만 원자적으로 교체한다.

### 3. A/V 동기화 (오디오 시계, 프레임 드롭, PTS, 속도 변경)
- **오디오 마스터 시계 (Audio Master Clock)**: 사람은 영상 지터보다 오디오 끊김(클리핑)에 훨씬 민감하므로 오디오 출력 샘플 위치를 기준 시계 $C_{\text{audio}}$로 삼는다 (출처: https://github.com/FFmpeg/FFmpeg/blob/master/fftools/ffplay.c).
  $$C_{\text{audio}} = \text{PTS}_{\text{last\_written}} - \frac{\text{BytesInRingBuffer} + \text{WASAPIDeviceLatencyBytes}}{\text{BytesPerSec}}$$
- **PTS 비교 및 프레임 드롭**: 비디오 프레임의 표시 시각 $\text{PTS}_v = \text{frame.best\_effort\_timestamp} \times \text{time\_base}$와 $C_{\text{audio}}$의 차이 $\Delta = \text{PTS}_v - C_{\text{audio}}$를 계산한다.
  - $\Delta > \text{sync\_threshold}$ (예: +10ms 이상 빠름): 다음 VSync 또는 $\Delta$만큼 대기 후 출력.
  - $-\text{sync\_threshold} \le \Delta \le \text{sync\_threshold}$: 즉시 출력.
  - $\Delta < -0.04\text{s}$ (40ms 이상 뒤처짐): **Late Frame Drop** — 현재 프레임을 렌더링/텍스처 업로드하지 않고 폐기한 뒤 큐의 다음 프레임으로 넘어간다. 심하게 밀리면(예: -0.5s 이하) 디코더에서 비참조 프레임(`AVDISCARD_NONREF`) 스킵이나 다음 키프레임까지 건너뛴다.
- **재생 속도 변경 (0.25x ~ 4.0x)**:
  - 비디오는 목표 프레임 간격을 $1 / \text{speed}$로 스케일링(또는 오디오 시계 증가율을 `speed`배로 반영)한다.
  - 오디오는 피치(음정) 보존을 위해 FFmpeg `atempo` 필터(0.5~100.0 범위, LGPL) 또는 시간 영역 WSOLA/SONIC 알고리즘을 거치게 해야 목소리가 변조되지 않는다 (출처: https://ffmpeg.org/ffmpeg-filters.html#atempo).

### 4. 탐색 (Keyframe 탐색 vs 정확 탐색)
- **키프레임(고속) 탐색**: `av_seek_frame(fmt_ctx, stream_idx, target_ts, AVSEEK_FLAG_BACKWARD)` 후 `avcodec_flush_buffers()`를 호출해 목표 시각 직전의 가장 가까운 IDR(키프레임)부터 즉시 재생한다. 드래그 스크러빙 중에 유리하다 (출처: https://ffmpeg.org/doxygen/trunk/group__lavf__decoding.html).
- **정확(Exact) 탐색**: `AVSEEK_FLAG_BACKWARD`로 이전 키프레임으로 이동한 뒤, 디코더 루프에서 `frame.pts < target_ts`인 프레임들은 **화면에 그리지 않고(Render Skip)** 고속 디코딩만 수행하다가 `frame.pts >= target_ts`에 도달한 첫 프레임을 출력하고 오디오 버퍼도 해당 타임스탬프부터 잘라낸다.

---

## Q3 디코딩 백엔드 비교

(확인 기준일: 2026-11-01)

| 백엔드 분류 | 대표 크레이트 (최신 버전) | 유지 상태 | Windows (stable-msvc) 빌드 난이도 | 장점 | 단점 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FFmpeg 바인딩** | `ffmpeg-the-third` (`6.0.0+ffmpeg-9.0`)<br>`ffmpeg-next` (`9.0.0`)<br>`rsmpeg` (`0.18.0+ffmpeg.8.0`) | 활발 (`ffmpeg-the-third`, `rsmpeg`) / 유지보수 (`ffmpeg-next`) | **중**: BtbN shared 빌드 또는 `vcpkg` + `LIBCLANG_PATH`(bindgen) 필요 | 모든 코덱/컨테이너 지원, 프레임·패킷·D3D11VA 텍스처에 대한 완전한 저수준 제어 | A/V 싱크, 탐색, 자막(libass), 오디오 출력을 직접 조립해야 함 |
| **GStreamer 바인딩** | `gstreamer` (`0.25.4`, GStreamer 1.28 대응) | 매우 활발 (공식 freedesktop/Centricular) | **중~상**: 공식 MSVC `.msi` Runtime + Devel 설치 및 `PKG_CONFIG_PATH` 설정 필수 | `playbin3`, `d3d11*`/`d3d12*` 플러그인으로 무복사 파이프라인·싱크·탐색 자동화 | Windows 배포 시 수많은 플러그인 DLL 번들 관리가 번거롭고 크기가 큼 (80~150MB+) |
| **libmpv 바인딩** | `libmpv2` (`6.0.0`)<br>`mpv-client` (`1.1.0`) | 활발 (Stremio 및 커뮤니티 유지) | **하**: `libmpv-2.dll` (shinchiro 빌드) 및 `.lib`만 링크하면 즉시 빌드 | **PotPlayer급 재생기 완성도**, `hwdec` 런타임 전환, `libass` 자막, GLSL/d3d11vpp 내장 | gpui 내부 요소로 무복사 합성하려면 Render API/D3D11 인터롭 작업 필요 (`--wid`는 자식 창 한계) |
| **Media Foundation 직접** | `windows` (`0.62.2`) | 매우 활발 (Microsoft 공식) | **최하**: OS 내장 DLL(`mfplat.dll`, `mfreadwrite.dll`)만 사용, 외부 의존 0 | 배포 용량 0MB 추가, D3D11(`IMFDXGIDeviceManager`) 네이티브 통합, 특허/GPL 걱정 없음 | **범용 재생 불가**: HEVC/AV1 MS 스토어 확장 의존, ASS 자막 미지원, 구형 TS/복잡한 MKV 호환성 낮음 |
| **순수 Rust** | `symphonia` (`0.6.1`, 오디오/컨테이너)<br>`rav1d` / `rav1d-safe` (`0.6.0`, AV1) | 활발 | **하**: `cargo build`만으로 동작 (`rav1d`는 `nasm` 선택) | 메모리 안전성, C 툴체인/DLL 지옥 없음, MIT/Apache-2.0 라이선스 | **범용 비디오 코덱 부재**(순수 Rust H.264/HEVC/VP9 고속 디코더 및 HW 가속 부재로 단독 사용 불가) |

- 출처: `ffmpeg-the-third` (https://crates.io/crates/ffmpeg-the-third), `rsmpeg` (https://crates.io/crates/rsmpeg), `gstreamer` (https://crates.io/crates/gstreamer), `libmpv2` (https://crates.io/crates/libmpv2), `windows` (https://crates.io/crates/windows), `symphonia` (https://crates.io/crates/symphonia), `rav1d-safe` (https://crates.io/crates/rav1d-safe).

---

## Q4 gpui 통합

### 1. `workbench-gpui`의 탭(Pane Item) 구조에 뷰어 추가하는 법 (src 근거)
- **현재 코드 구조**:
  - [`Cargo.toml`](file:///D:/projects/workbench-gpui/Cargo.toml#L1-L7)에서 `gpui-kit = "=0.8.1"`(내부적으로 `gpui-pre 0.3.10` 재노출)을 사용한다.
  - [`src/ui/mod.rs`](file:///D:/projects/workbench-gpui/src/ui/mod.rs#L30-L39)의 `Workbench` 구조체는 현재 단일 아이템 슬롯인 `editor_poc: Option<Entity<editor_poc::EditorPoc>>`를 보관한다.
  - [`src/ui/mod.rs`](file:///D:/projects/workbench-gpui/src/ui/mod.rs#L80-L83) 및 [`L109-L116`](file:///D:/projects/workbench-gpui/src/ui/mod.rs#L109-L116)에서 `tab_bar(cx)` 아래의 `active-item` 영역에 `let item: AnyElement = ...`를 자식(`div().flex_1().min_h_0().child(item)`)으로 넣는다.
- **동영상 뷰어 추가 방법**:
  1. 탭 아이템을 열거형 또는 트레이트 객체로 일반화한다:
     ```rust
     enum PaneItem {
         Editor(Entity<EditorPoc>),
         VideoPlayer(Entity<VideoPlayerView>),
     }
     ```
  2. 프로젝트 트리([`src/ui/layout.rs`](file:///D:/projects/workbench-gpui/src/ui/layout.rs#L290-L299))에서 파일 확장자(`.mp4`, `.mkv`, `.webm`, `.ts`) 클릭 시 `cx.new(|cx| VideoPlayerView::new(path, window, cx))`를 생성해 활성 탭 `PaneItem::VideoPlayer`로 등록한다.
  3. [`src/ui/mod.rs`](file:///D:/projects/workbench-gpui/src/ui/mod.rs#L80-L83)에서 `PaneItem::VideoPlayer(view) => div().size_full().child(view.clone()).into_any_element()`로 분기한다.

### 2. 프레임 그리기 (`RenderImage`, 사용자 정의 `Element`, `paint_surface`, DX11 렌더러)
- **gpui Windows 렌더러 구조**: Windows에서 gpui는 `DirectXRenderer`(**Direct3D 11** 기반, `gpui_windows`)를 사용한다 (출처: https://zed.dev/blog/zed-on-windows).
- **`RenderImage` 경로**: `gpui::RenderImage`는 CPU 메모리의 BGRA 픽셀 버퍼(`smallvec::SmallVec<[image::Frame; 1]>`)를 받아 아틀라스/텍스처에 업로드한다 (출처: https://docs.rs/gpui/latest/gpui/struct.RenderImage.html). 매 프레임 새 `Arc<RenderImage>`를 만들면 GPU 아틀라스 메모리가 누수되므로 이전 프레임 이미지를 `window.drop_image()` 등으로 해제해야 한다.
- **`paint_surface` 지원 여부 (중요 함정)**: gpui의 `window.paint_surface(bounds, paint_surface)`는 macOS(`CVPixelBuffer`)용으로 설계되었으며, Windows의 `DirectXRenderer`에서는 오랫동안 미구현(stub)이거나 외부 D3D11 텍스처 핸들을 직접 받지 못하는 한계가 보고되었다 (출처: https://github.com/zed-industries/zed/issues).
- **매 프레임 갱신 루프**: [`src/ui/editor_poc.rs`](file:///D:/projects/workbench-gpui/src/ui/editor_poc.rs#L231-L238)에 확인된 것처럼 `window.on_next_frame(...)` 또는 디코더 알림 시 `cx.notify()`를 호출해 비디오 뷰 엔티티만 다시 그리도록 유도한다.

### 3. 텍스처 업로드 비용 · YUV→RGB 위치 · 4K60 한계
- **대역폭 계산**:
  - 1080p60 RGBA: $1920 \times 1080 \times 4 \times 60 \approx 497.6\text{ MB/s}$ (CPU 업로드 가능하나 점유율 상승).
  - **4K60 (3840×2160) RGBA**: $3840 \times 2160 \times 4 \times 60 \approx \mathbf{1,990.6\text{ MB/s (약 2 GB/s)}}$ — CPU에서 `sws_scale`(YUV→RGBA) 후 `RenderImage`로 매 프레임 업로드하면 프레임 드롭이 발생한다.
  - **4K60 NV12 (YUV 4:2:0 8-bit)**: $3840 \times 2160 \times 1.5 \times 60 \approx 746.5\text{ MB/s}$ (무복사 D3D11VA 사용 시 PCIe 전송 0 바이트).
- **결론**: YUV→RGB 색공간 변환(BT.709/BT.2020)은 절대 CPU(`libswscale`)에서 하지 말고, **GPU(D3D11VideoProcessor 또는 픽셀 셰이더)**에서 수행해야 4K60이 유지된다.

### 4. libmpv 자식 창(HWND) 방식과 실제 사례
- **자식 창(HWND) 방식**: gpui 요소의 `on_children_prepainted`([`src/ui/mod.rs`](file:///D:/projects/workbench-gpui/src/ui/mod.rs#L86)의 `probe`와 동일한 원리)에서 비디오 영역의 `Bounds<Pixels>`와 `window.scale_factor()`를 읽어 물리 픽셀 사각형을 구한 뒤, `CreateWindowExW(WS_CHILD | WS_VISIBLE, ...)`로 만든 자식 HWND에 `SetWindowPos`를 호출하고 `mpv_set_option_string(mpv, "wid", hwnd_str)`을 넘긴다.
  - 장점: 무복사 D3D11/Vulkan 렌더러(`vo=gpu-next`)가 그대로 동작해 4K60·HDR·R1·R2가 완벽하다.
  - 단점: 자식 HWND가 gpui 다이얼로그나 드롭다운 메뉴를 가리는 **Airspace(Z-order) 문제**가 생기므로 컨트롤 바를 비디오 영역 하단 바깥에 배치하거나 별도 `WS_POPUP` 레이어로 띄워야 한다.
- **실제 사례**:
  - `cijiugechu/gpui-video-player`: GStreamer `AppSink`로 디코딩한 프레임을 gpui 요소로 렌더링하는 라이브러리 (출처: https://github.com/cijiugechu/gpui-video-player).
  - `everpcpc/syncplay-gpui`: gpui 기반 다중 사용자 동기화 미디어 플레이어 구현체 (출처: https://github.com/everpcpc/syncplay-gpui).

---

## Q5 Tauri 통합

### 1. `workbench-tauri` 탭 구조에 뷰어 추가 (src 근거)
- **현재 코드 구조**:
  - [`src-tauri/Cargo.toml`](file:///D:/projects/workbench-tauri/src-tauri/Cargo.toml#L13-L16)은 `tauri = "2.12.5"`를 사용하며, [`specs/v0.1/DECISIONS.md`](file:///D:/projects/workbench-tauri/specs/v0.1/DECISIONS.md#L51-L65)의 `D-3`에 따라 프런트엔드는 **Solid(`solid-js`) + Vite**로 확정되어 있다.
  - `D-3`의 핵심 근거는 Solid 컴포넌트가 실제 DOM 노드를 반환하므로 Pane 분할·이동 시 기존 뷰 인스턴스를 언마운트하지 않고 그대로 옮길 수 있다는 점이다.
- **뷰어 추가 방법**:
  - Solid의 탭 아이템 라우터에서 미디어 파일 탭일 때 `<VideoPlayerPane path={item.path} />` 컴포넌트를 렌더링하고, Pane 이동 시에도 비디오 컨테이너 DOM(또는 연결된 Rust 플레이어 세션 ID)을 파괴하지 않고 유지한다.

### 2. WebView2 코덱 한계 및 로컬 파일 제공 (`asset://` vs Range 요청)
- **WebView2 코덱 한계**: Q1에서 서술한 대로 AC3/E-AC3 오디오 미지원, HEVC 확장 의존, `.ts` 컨테이너 및 `.ass` 자막 미지원으로 PotPlayer 대체 불가 (출처: https://learn.microsoft.com/en-us/microsoft-edge/webview2/).
- **로컬 파일 제공 (`convertFileSrc` / `asset://`의 Range 한계)**:
  - Tauri의 `convertFileSrc`(`http://asset.localhost/...`)나 커스텀 프로토콜(`register_uri_scheme_protocol`)을 통해 수 GB 크기의 MKV/MP4를 `<video>`에 연결하면, WebView2의 `Range: bytes=start-end` (`206 Partial Content`) 요청 처리 버그로 인해 시크(탐색)가 먹통이 되거나 무한 재요청 루프에 빠지는 이슈가 반복 보고되었다 (출처: https://github.com/tauri-apps/tauri/issues/4775).
  - **해결책**: `<video>`를 쓸 때는 Rust 백엔드에서 `axum` + `tower_http::services::ServeFile`로 `127.0.0.1:0` 로컬 루프백 HTTP 서버를 띄워 표준 HTTP `206 Partial Content`로 스트리밍해야 한다. 단, [`specs/v0.1/DECISIONS.md`](file:///D:/projects/workbench-tauri/specs/v0.1/DECISIONS.md#L197-L203)의 `D-10`(CSP 처음부터 활성화)에 따라 `tauri.conf.json`의 CSP `media-src`와 `connect-src`에 `http://127.0.0.1:*`를 반드시 허용해야 한다.

### 3. `mpv --wid` 자식 창을 탭 영역에 맞추기 & 프레임 IPC 전송 비용 & 플러그인
- **프레임 IPC 전송 비용**: Tauri `ipc::Channel`이나 WebSocket으로 디코딩된 1080p60(500 MB/s) / 4K60(2 GB/s) 원시 RGBA 프레임을 WebView `<canvas>`로 보내면 직렬화·복사 오버헤드로 인해 실사용이 불가능하다.
- **`mpv --wid` 자식 창(HWND) 또는 투명 WebView 합성 방식**:
  1. **자식 HWND 동기화**: Solid의 `<div ref={containerRef}>`에 `ResizeObserver`를 달아 탭 크기 변화·사이드바 리사이즈·탭 전환 시 `invoke("update_video_rect", { x, y, width, height, visible })`를 호출하고, Rust에서 `SetWindowPos` 및 `ShowWindow(SW_HIDE/SW_SHOW)`로 자식 HWND를 맞춘다.
  2. **배경 렌더 + 투명 WebView 방식 (`tauri-plugin-libmpv`)**: 메인 창 배경에 `libmpv`를 렌더링하고 WebView 배경을 투명하게 뚫어 HTML/CSS 컨트롤 오버레이를 자연스럽게 올리는 방식이다 (출처: https://github.com/nini22P/tauri-plugin-mpv).

---

## Q6 R1 GPU 가속

### 1. Windows 하드웨어 디코딩 API 비교
- **D3D11VA (권장 표준)**: Windows 8~11의 표준 비디오 가속 API. NVIDIA/AMD/Intel/Qualcomm 모든 GPU에서 H.264, HEVC, VP9, AV1 하드웨어 디코딩과 D3D11 렌더러 무복사 공유를 지원한다 (출처: https://learn.microsoft.com/en-us/windows/win32/medfound/direct3d-11-video-apis).
- **DXVA2**: D3D9 기반 레거시 API. 'DXVA2 Native'는 필터/셰이더 개입이 어렵고 D3D11과 직접 공유할 수 없으며, 'DXVA2 Copy-Back'은 VRAM→RAM 복사가 발생하므로 D3D11VA를 쓸 수 없는 구형 드라이버 폴백용으로만 둔다.
- **NVDEC (CUDA) / QSV (Intel) / AMF (AMD)**: 특정 제조사 GPU 전용. 범용 플레이어에서는 D3D11VA가 동일한 하드웨어 디코더 블록(NVDEC/QSV/VCN)을 추상화하여 구동하므로 굳이 벤더 종속 API를 기본으로 쓸 필요가 없다.

### 2. 백엔드별 R1 켜고 끄기 · 재생 중 전환 · 자동 폴백 · 무복사

| 구분 | FFmpeg (`ffmpeg-the-third` / `rsmpeg`) | libmpv (`libmpv2`) | GStreamer (`gstreamer-rs`) |
| :--- | :--- | :--- | :--- |
| **켜기 (ON)** | `av_hwdevice_ctx_create(&ctx, AV_HWDEVICE_TYPE_D3D11VA, ...)`를 `codec_ctx->hw_device_ctx`에 설정 | `mpv.set_property("hwdec", "auto-safe")` 또는 `"d3d11va"` | `d3d11h264dec`, `d3d11hevcdec`, `d3d11vp9dec`, `d3d11av1dec` 랭크를 `PRIMARY + 1`로 상향 |
| **끄기 (OFF)** | `hw_device_ctx`를 `NULL`로 두고 SW `AVCodec`으로 디코더 오픈 | `mpv.set_property("hwdec", "no")` | `d3d11*dec` 플러그인 랭크를 `NONE`(0)으로 설정하거나 `avdec_*` 강제 연결 |
| **재생 중 전환** | 현재 PTS 저장 → 패킷 큐 일시 정지 → `avcodec_free_context` 후 새 모드로 재생성 → 이전 키프레임부터 목표 PTS까지 고속 디코딩(정확 탐색) 후 재개 | **즉시 지원**: 재생 중 `set_property("hwdec", "no")` 또는 `"auto-safe"` 호출 시 mpv 내부에서 끊김 없이 디코더를 재초기화함 | 파이프라인 `PAUSED` 또는 디코드 빈 재협상(`SEEK_FLAG_FLUSH` 동반)으로 디코더 엘리먼트 교체 |
| **실패 시 SW 자동 전환** | `get_format` 콜백에서 `AV_PIX_FMT_D3D11` 초기화 실패 시 `AV_PIX_FMT_YUV420P`(SW)를 반환하면 FFmpeg이 자동으로 SW 디코딩 수행 | **`auto-safe`가 자동 처리**: 코덱/프로파일(예: Hi10P) 미지원 시 자동으로 SW 디코딩으로 폴백 | `decodebin3` / `playbin3`가 `d3d11*dec` 협상 실패 시 자동으로 다음 우선순위인 `avdec_*`(SW) 선택 |
| **무복사 (Zero-copy)** | `AVFrame->data[0]`(`ID3D11Texture2D*`)과 `data[1]`(`ArraySlice` 인덱스)을 D3D11 렌더러/셰이더에서 직접 바인딩 | `vo=gpu-next` + `gpu-api=d3d11` + `hwdec=d3d11va` 사용 시 디코딩부터 출력까지 완전 무복사 | `video/x-raw(memory:D3D11Memory)` 캡스로 `d3d11convert` → `d3d11videosink` 연결 시 완전 무복사 |

- 출처: FFmpeg hwcontext_d3d11va (https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h.html), mpv hwdec (https://mpv.io/manual/stable/#options-hwdec), GStreamer d3d11 plugin (https://gstreamer.freedesktop.org/documentation/d3d11/).

---

## Q7 R2 영상 옵션

### 1. 세 가지 구현 위치 비교 (디코더 필터 vs 출력 셰이더 vs D3D11 비디오 프로세서)

| R2 요구 항목 | ① 디코더 쪽 필터 (`libavfilter` / GStreamer) | ② 출력 셰이더 (mpv `glsl-shaders`·속성 / 자체 WGSL·HLSL) | ③ D3D11 비디오 프로세서 (`ID3D11VideoProcessor`) | 권장 방식 |
| :--- | :--- | :--- | :--- | :--- |
| **밝기·대비·채도** | FFmpeg `eq` (**주의: GPL 전용**) 또는 `colorbalance`(LGPL) / GStreamer `videobalance` | mpv 내장 속성(`brightness`, `contrast`, `saturation`: -100~100) / HLSL 색상 행렬 곱 | `D3D11_VIDEO_PROCESSOR_FILTER_BRIGHTNESS` / `CONTRAST` / `SATURATION` | **② 출력 셰이더** (단일 행렬 연산, 지연 0, HW/SW 무관) |
| **선명하게 (Sharpen)** | FFmpeg `unsharp=5:5:1.0` (LGPL) 또는 `cas` (Contrast Adaptive Sharpening, LGPL) | mpv 내장 `sharpen` 속성 또는 `CAS.glsl` / 자체 픽셀 셰이더(3x3 커널/CAS) | `D3D11_VIDEO_PROCESSOR_FILTER_EDGE_ENHANCEMENT` (드라이버 의존) | **② 출력 셰이더 (CAS)** (무복사 유지, 실시간 강도 슬라이더 즉시 반영) |
| **흐리게 (Blur)** | FFmpeg `gblur=sigma=2.0` (LGPL) 또는 `boxblur`(GPL) | mpv 내장 `sharpen` 음수값 또는 2-pass Gaussian GLSL/HLSL 셰이더 | 미지원 | **② 출력 셰이더** |
| **노이즈 제거 (Denoise)** | FFmpeg `hqdn3d` (**GPL 전용**), `nlmeans` (LGPL이나 **CPU에서 매우 느림**), `atadenoise` (LGPL) | 공간 양방향 필터(Bilateral/NLM) GLSL/HLSL 셰이더 | `D3D11_VIDEO_PROCESSOR_FILTER_NOISE_REDUCTION` (GPU 하드웨어 가속) | **③ D3D11 VPP** 또는 **② 경량 Bilateral 셰이더** |
| **디인터레이스** | FFmpeg `yadif` (LGPL), `bwdif` (LGPL, 고품질) / GStreamer `d3d11deinterlace` | `bwdif` 컴퓨트/픽셀 셰이더 | `VideoProcessorSetStreamFrameFormat` + `D3D11_VIDEO_PROCESSOR_PROCESSOR_CAPS_DEINTERLACE_ADAPTIVE` (`d3d11vpp`) | **HW 디코딩 시 ③ D3D11 VPP (`d3d11vpp`)**, SW 디코딩 시 **① `bwdif`** |

- 출처: FFmpeg Filters (https://ffmpeg.org/ffmpeg-filters.html), D3D11_VIDEO_PROCESSOR_FILTER (https://learn.microsoft.com/en-us/windows/win32/api/d3d11/ne-d3d11-d3d11_video_processor_filter).

### 2. 실시간 켜고 끄기 및 강도 조절 방법과 성능·품질 주의점
- **왜 CPU `libavfilter` 대신 출력 GPU 셰이더/D3D11 VPP를 써야 하는가?**:
  - `AV_PIX_FMT_D3D11` 하드웨어 디코딩 상태에서 `unsharp`, `gblur`, `bwdif` 같은 CPU 필터를 끼우면 **`hwdownload` (VRAM→RAM) → CPU 필터 → `hwupload` (RAM→VRAM)** 왕복이 발생해 R1(무복사 GPU 가속)의 이점이 사라지고 4K60에서 프레임이 드롭된다.
  - 또한 `AVFilterGraph`를 재생성하면 파이프라인에 들어있던 디코딩 프레임들이 플러시되면서 순간적인 깜빡임이 생길 수 있다 (`avfilter_graph_send_command`로 런타임 파라미터 변경이 가능한 필터도 일부에 한정됨).
- **권장 실시간 제어법**:
  - **libmpv**: `mpv.set_property("brightness", val)`, `mpv.set_property("sharpen", val)`, `mpv.command("change-list", &["glsl-shaders", "toggle", "..."])`, `mpv.set_property("deinterlace", "yes"|"no")`로 재생 중 프레임 드롭 없이 즉시 반영된다 (출처: https://mpv.io/manual/stable/#properties).
  - **자체 D3D11/wgpu 파이프라인**: 디인터레이스와 노이즈 제거는 `ID3D11VideoContext::VideoProcessorSetStreamFilter()`로 제어하고, 밝기·대비·채도·선명·블러는 픽셀 셰이더의 `cbuffer FilterParams` 상수 버퍼만 매 프레임 업데이트한다.

---

## Q8 PotPlayer 대응표

PotPlayer 공식 환경설정(F5) 및 우클릭 영상 메뉴 항목을 본 프로젝트(FFmpeg / libmpv / D3D11)로 대응시킨 표다.

| PotPlayer 메뉴 경로 및 기능 | 세부 동작 / 옵션 | libmpv 대응 구현 | FFmpeg + D3D11 자체 파이프라인 대응 구현 |
| :--- | :--- | :--- | :--- |
| **환경설정 → 필터 → 비디오 코덱 → 내장 코덱/DXVA 설정** | 사용 안 함 (SW) / DXVA2 / **D3D11 (권장)** | `hwdec=no` (OFF) ⇄ `hwdec=auto-safe` 또는 `d3d11va` (ON) | `AV_HWDEVICE_TYPE_D3D11VA` 활성/비활성, 실패 시 `get_format`에서 `AV_PIX_FMT_YUV420P` 폴백 |
| **영상 → 디인터레이스 (Ctrl+I)** | 자동 / 항상 사용 / 끄기 (모션 적응형·Bob·Weave) | `deinterlace=auto` / `yes` / `no` 및 `vf=d3d11vpp=deint=yes` | HW: `ID3D11VideoContext::VideoProcessorSetStreamFrameFormat(INT_TOP_FIELD_FIRST)`<br>SW: `libavfilter` `bwdif=mode=send_field:parity=auto:deint=all` |
| **영상 → 영상 처리 → 선명하게 (Sharpen, Ctrl+R)** | 켜기/끄기 및 강도 조절 | `sharpen` 속성 (`0.0` ~ `2.0`) 또는 `adaptive-sharpen.glsl` | HLSL/WGSL 픽셀 셰이더 (Contrast Adaptive Sharpening 커널, `sharpen_strength` 유니폼) 또는 `unsharp` |
| **영상 → 영상 처리 → 부드럽게 (Blur, Ctrl+N)** | 켜기/끄기 및 강도 조절 | `sharpen` 음수값 (`-1.0` ~ `0.0`) 또는 Gaussian Blur GLSL | HLSL/WGSL 5-tap/9-tap 분리형 가우시안 블러 픽셀 셰이더 (`blur_sigma` 유니폼) 또는 `gblur` |
| **영상 → 영상 처리 → 노이즈 제거 (3D/시간축·공간축)** | 약하게 / 중간 / 강하게 | `vf=d3d11vpp` 또는 Denoise GLSL 셰이더 (SW 시 `vf=hqdn3d`) | HW: `VideoProcessorSetStreamFilter(..., D3D11_VIDEO_PROCESSOR_FILTER_NOISE_REDUCTION, TRUE, level)`<br>셰이더: Bilateral Denoise |
| **영상 → 색상 조절 (명도 Z/A, 대비 C/X, 채도 V/S, 초기화 BackSpace)** | 밝기·대비·채도·색상 슬라이더 (-100 ~ +100) | `brightness`, `contrast`, `saturation`, `hue` 속성 실시간 변경 | 픽셀 셰이더 내 YUV→RGB 변환 행렬에 밝기 오프셋·대비 기울기·채도 행렬 결합 (또는 `D3D11_VIDEO_PROCESSOR_FILTER_*`) |

- 출처: PotPlayer 사용자 가이드 및 mpv 매뉴얼 (https://mpv.io/manual/stable/).

---

## Q9 소리 출력

(libmpv/GStreamer를 쓰면 내장 WASAPI 오디오 렌더러가 자동 처리하지만, FFmpeg 직접 구현 시 아래 구성을 쓴다.)
1. **`cpal` (`0.18.2`) vs `rodio` (`0.23.0`)**:
   - 동영상 플레이어에서는 A/V 동기화를 위해 오디오 버퍼 잔량과 콜백 시점을 정밀하게 추적해야 하므로 고수준 `rodio`보다 저수준 **`cpal` 직접 사용**이 적합하다 (출처: https://docs.rs/cpal/latest/cpal/).
   - Windows에서 `cpal`은 기본적으로 **WASAPI 공유 모드(Shared Mode)**로 동작한다.
2. **리샘플링 (`rubato` `5.0.1` 또는 FFmpeg `libswresample`)**:
   - WASAPI 공유 모드는 OS 믹서 샘플레이트(보통 48,000 Hz)만 받으므로, 동영상 오디오(44.1 kHz 등)를 장치 레이트로 변환해야 한다.
   - FFmpeg을 쓴다면 이미 링크된 **`libswresample` (`swr_convert`)**로 채널 레이아웃(5.1채널 AC3 → 2채널 스테레오 다운믹스 포함)과 샘플레이트를 한 번에 변환하는 것이 가장 간단하며, 재생 속도 미세 조절을 통한 드리프트 보정에는 순수 Rust 비동기 리샘플러인 **`rubato` (`5.0.1`)**를 함께 쓸 수 있다 (출처: https://docs.rs/rubato/latest/rubato/).
3. **오디오 출력 장치 변경 (이어폰 탈착 등) 대응**:
   - 사용자가 재생 중 기본 오디오 장치를 바꾸면 `cpal`의 `StreamError::DeviceNotAvailable` (`AUDCLNT_E_DEVICE_INVALIDATED`) 에러가 콜백에서 발생한다.
   - 이때 오디오 스레드가 패닉하지 않도록 에러 콜백에서 재연결 플래그를 세우고, `cpal::default_host().default_output_device()`를 다시 조회해 새 샘플레이트로 리샘플러와 스트림을 재생성한다.

---

## Q10 자막

1. **SRT (SubRip) 자막**:
   - 시간 구간과 평문(간단한 `<b>`, `<i>`, `<font color>` 태그 포함)만 있으므로 `srtparse` / `srtlib` (`0.2.0`) 또는 직접 작성한 파서로 파싱해 **gpui 텍스트 요소(`div().child(...)`)나 Tauri HTML 오버레이**로 비디오 위에 그리면 가장 깨끗하고 빠르다 (출처: https://crates.io/crates/srtlib).
2. **ASS / SSA (Advanced SubStation Alpha) 자막**:
   - 애니메이션(`\move`, `\fad`), 벡터 드로잉(`\p1`), 복잡한 폰트·테두리·그림자 스타일이 포함되어 있어 단순 UI 텍스트로 그릴 수 없다.
   - **방식 A (libmpv / GStreamer)**: 내장된 `libass`가 DirectWrite 폰트 탐색부터 자막 비트맵 합성까지 완벽히 처리한다 (`sub-ass-override`, `sub-scale` 등으로 제어).
   - **방식 B (FFmpeg 자체 파이프라인)**:
     - **내장 `subtitles` 필터 (`vf_subtitles.c`, LGPL)**: `libass`가 포함된 FFmpeg 빌드(BtbN 등)에서 사용 가능하지만, CPU 프레임 위에 굽는 방식이라 무복사 HW 디코딩을 깬다.
     - **`ass-renderer` (`0.1.2`, 순수 Rust, MIT/Apache-2.0) 또는 `libass` 직접 호출**: 현재 PTS에 해당하는 ASS 이벤트를 RGBA 비트맵 리스트(위치 + 텍스처 조각)로 렌더링한 뒤, 비디오 셰이더 위(또는 gpui 오버레이 이미지 레이어)에 알파 블렌딩으로 합성하면 무복사 비디오 파이프라인을 깨지 않는다 (출처: https://crates.io/crates/ass-renderer).

---

## Q11 탭 동작

Zed 레이아웃 기반인 `workbench-gpui` / `workbench-tauri`에서 동영상 탭을 다룰 때의 필수 정책이다.
1. **비활성(숨김) 탭 전환**:
   - 다른 탭으로 전환해 동영상 탭이 화면에서 가려졌을 때의 정책을 옵션화한다:
     - **기본값 (권장)**: 탭이 숨겨지면 자동으로 `pause()`하여 CPU/GPU/전력을 아낀다.
     - **백그라운드 오디오 계속 재생 모드**: 오디오 디코딩과 시계는 유지하되, **비디오 패킷/프레임 디코딩(`AVDiscard::AVDISCARD_ALL` 또는 mpv `vid=no` / 렌더 콜백 중단)과 GPU 텍스처 업로드를 중단**한다. 다시 탭이 활성화되면 즉시 현재 오디오 PTS 위치의 키프레임부터 비디오 디코딩을 재개한다.
2. **화면 분할 (Split Pane) 및 여러 탭 동시 재생**:
   - D3D11VA 하드웨어 디코더는 인스턴스마다 GPU 내에 디코더 참조 프레임용 **`ID3D11Texture2D` 배열(보통 16~24장 슬라이스)**을 할당한다.
   - 4K HEVC/AV1 탭 하나당 약 200~400 MB의 VRAM이 소모되므로, 탭을 10개 열어 모두 HW 디코딩 상태로 두면 `E_OUTOFMEMORY` 또는 드라이버 리셋(TDR)이 발생한다.
   - **해결책 (LRU 디코더 풀)**: **현재 화면(활성 Pane들)에 실제로 보이는 탭(최대 2~4개)만 활성 디코더/VRAM 세션을 유지**하고, 가려진 비활성 탭은 마지막 타임스탬프와 썸네일 1장만 남긴 채 하드웨어 디코더 컨텍스트를 해제(Suspend)한다.

---

## Q12 빌드·배포·라이선스

### 1. Windows DLL 구하는 곳 및 빌드 연동 (`stable-msvc`)
- **FFmpeg**:
  - **BtbN / FFmpeg-Builds** (https://github.com/BtbN/FFmpeg-Builds): `ffmpeg-n8.0-latest-win64-lgpl-shared-8.0.zip` (또는 `gpl-shared`). `.dll`과 `.lib`(MSVC 호환 임포트 라이브러리), 헤더가 모두 들어 있다.
  - **vcpkg**: `vcpkg install ffmpeg[core,avcodec,avformat,avfilter,swresample,swscale,d3d11va]:x64-windows`로 순수 LGPL MSVC 빌드를 얻을 수 있다 (출처: https://github.com/microsoft/vcpkg).
  - 빌드 환경 변수: `FFMPEG_DIR`(설치 경로), `LIBCLANG_PATH`(bindgen 실행용 LLVM 경로).
- **libmpv**:
  - **shinchiro / mpv-winbuild-cmake** (https://github.com/shinchiro/mpv-winbuild-cmake/releases): `mpv-dev-x86_64-*.7z`에서 `libmpv-2.dll`과 `mpv.lib`(또는 `dlltool`/`lib.exe`로 `.def`에서 생성)를 사용한다.
- **GStreamer**:
  - **공식 MSVC 패키지** (https://gstreamer.freedesktop.org/download/#windows): 반드시 MinGW가 아닌 **MSVC 64-bit (VS 2019/2022) runtime installer**와 **development installer** 두 `.msi`를 설치하고 `GSTREAMER_1_0_ROOT_MSVC_X86_64` 환경 변수를 설정한다.

### 2. 앱과 함께 묶기 및 설치본 크기
- **묶는 법**:
  - `workbench-gpui`: 실행 파일(`workbench-gpui.exe`)과 같은 폴더에 DLL들을 배치한다(Windows DLL 검색 순서 1순위).
  - `workbench-tauri`: `src-tauri/tauri.conf.json`의 `bundle.resources` 배열에 `["resources/dll/*.dll"]`을 지정해 MSI/NSIS 설치본에 포함시킨다 (출처: https://v2.tauri.app/develop/resources/).
- **배포 추가 크기**:
  - FFmpeg shared (LGPL 기본 코덱): 약 **35 ~ 55 MB** (`avcodec`, `avformat`, `avutil`, `avfilter`, `swresample`, `swscale`).
  - `libmpv-2.dll` 단일 파일 (FFmpeg + libass + libplacebo 정적 내장): 약 **65 ~ 85 MB** (비압축 기준, 설치본 압축 시 약 25~30 MB).
  - GStreamer MSVC 런타임 + 필수 플러그인 묶음: 약 **80 ~ 150 MB**.

### 3. 라이선스 및 특허 코덱 위험 (핵심)
- **FFmpeg LGPL v2.1+ vs GPL v2+ (필터 함정)**:
  - `--enable-gpl` 없이 빌드한 FFmpeg을 **동적 링크(DLL)**로 쓰면 앱 코드는 독점/MIT/Apache 라이선스를 유지할 수 있다 (출처: https://www.ffmpeg.org/legal.html).
  - **주의할 GPL 전용 필터**: R2 요구사항을 FFmpeg 필터로 구현할 때 **`hqdn3d`(노이즈 제거), `eq`(밝기/대비/채도), `pp`(포스트프로세싱), `boxblur`는 GPL 라이선스**다 (`libavfilter/Makefile`의 `CONFIG_GPL` 항목). 반면 **`unsharp`, `gblur`, `cas`, `yadif`, `bwdif`, `nlmeans`, `atadenoise`, `colorbalance`, `d3d11va`는 LGPL**이다. 따라서 LGPL을 유지하려면 `hqdn3d`와 `eq` 대신 D3D11 비디오 프로세서 또는 자체 셰이더를 써야 한다.
- **libmpv**: 공식/shinchiro 배포 빌드는 GPL 컴포넌트가 포함되어 **GPLv2+**인 경우가 많으며, 상용/폐쇄형 배포를 원하면 `--enable-lgpl` 옵션으로 빌드된 `libmpv-2.dll`(LGPLv2.1+)을 사용하고 동적 링크 의무를 준수해야 한다 (출처: https://github.com/mpv-player/mpv/blob/master/Copyright).
- **GStreamer**: 코어는 LGPLv2+이나 플러그인이 `base`/`good`(LGPL), `bad`(LGPL 및 일부 특허/실험), `ugly`(GPL 또는 특허 코덱 포함)로 나뉘므로 `gst-plugins-ugly`를 제외해야 LGPL이 유지된다 (출처: https://gstreamer.freedesktop.org/documentation/frequently-asked-questions/licensing.html).
- **특허 코덱 (H.264, HEVC, AAC, AC3)**: 오픈소스 라이선스(LGPL/GPL)와 별개로 MPEG LA / Via Licensing / Velos Media의 특허 풀 대상이므로, 상용 대량 배포 시 코덱 특허 라이선스 고지 또는 OS 하드웨어 디코더 활용 여부를 법무 검토해야 한다.

---

## Q13 크레이트 목록

(확인 날짜: 2026-11-01 기준, 출처: https://crates.io)

| 크레이트 이름 | 최신 버전 (확인일) | 최근 릴리스 시기 | 라이선스 | 유지 상태 | 용도 및 비고 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`ffmpeg-the-third`** | `6.0.0+ffmpeg-9.0` (2026-11-01) | 2026-08 | WTFPL | **활발** | `ffmpeg-next`의 현대적 포크, FFmpeg 5.1~9.0 지원 (https://crates.io/crates/ffmpeg-the-third) |
| **`ffmpeg-next`** | `9.0.0` (2026-11-01) | 2026-03 | WTFPL | 유지보수 모드 | 고수준 FFmpeg 래퍼 (https://crates.io/crates/ffmpeg-next) |
| **`rsmpeg`** | `0.18.0+ffmpeg.8.0` (2026-11-01) | 2026 | MIT | **활발** | 얇고 안전한 FFmpeg FFI 래퍼, D3D11VA 저수준 제어에 유리 (https://crates.io/crates/rsmpeg) |
| **`libmpv2`** | `6.0.0` (2026-11-01) | 2026-07 | LGPL-2.1 | **활발** | Stremio 팀 등이 관리하는 `libmpv` v2.0+ 안전 바인딩 (https://crates.io/crates/libmpv2) |
| **`gstreamer`** | `0.25.4` (2026-11-01) | 2026-09 | MIT OR Apache-2.0 | **매우 활발** | 공식 GStreamer 1.28 바인딩, `gstreamer-video`, `gstreamer-app` 동반 사용 (https://crates.io/crates/gstreamer) |
| **`windows`** | `0.62.2` (2026-11-01) | 2025-10 (관련 메타데이터 2026-09) | MIT OR Apache-2.0 | **매우 활발** | MS 공식 Win32/D3D11/DXGI/Media Foundation 바인딩 (https://crates.io/crates/windows) |
| **`cpal`** | `0.18.2` (2026-11-01) | 2026-08 | Apache-2.0 | **활발** | 저수준 크로스플랫폼 오디오 입출력 (Windows WASAPI) (https://crates.io/crates/cpal) |
| **`rodio`** | `0.23.0` (2026-11-01) | 2026-10 | MIT OR Apache-2.0 | **활발** | `cpal` 기반 고수준 오디오 재생기 (https://crates.io/crates/rodio) |
| **`rubato`** | `5.0.1` (2026-11-01) | 2026-10 | MIT OR Apache-2.0 | **활발** | 순수 Rust 비동기 오디오 리샘플러 (https://crates.io/crates/rubato) |
| **`symphonia`** | `0.6.1` (2026-11-01) | 2026-08 | MPL-2.0 | **활발** | 순수 Rust 컨테이너 디먹서 및 오디오 디코더 (https://crates.io/crates/symphonia) |
| **`rav1d-safe`** | `0.6.0` (2026-11-01) | 2026-09 | BSD-2-Clause | **활발** | `dav1d`의 Rust 포팅(`rav1d`) 안전 래퍼 (https://crates.io/crates/rav1d-safe) |
| **`ass-renderer`** | `0.1.2` (2026-11-01) | 2026 | MIT OR Apache-2.0 | **활발** | 순수 Rust ASS 자막 파싱·레이아웃·렌더링 엔진 (https://crates.io/crates/ass-renderer) |
| **`srtlib`** | `0.2.0` (2026-11-01) | 유지보수 | MIT OR Apache-2.0 | 안정 | SRT 자막 파서 (https://crates.io/crates/srtlib) |
| **`wgpu`** | `30.0.1` (2026-11-01) | 2026-08 | MIT OR Apache-2.0 | **매우 활발** | 크로스플랫폼 GPU API, 커스텀 YUV/필터 셰이더 파이프라인용 (https://crates.io/crates/wgpu) |
| **`tauri-plugin-mpv`** | `0.5.2` (2026-11-01) | 2025-12 | MIT / Apache-2.0 | 유지보수 | Tauri 2용 mpv 프로세스/IPC 제어 플러그인 (https://crates.io/crates/tauri-plugin-mpv) |

---

## Q14 기존 Rust 동영상 플레이어·라이브러리 사례 최소 8개

(확인 날짜: 2026-11-01 기준)

| # | 이름 (저장소 URL) | 백엔드 · GUI | 별 수 (Stars) | 최근 커밋 / 상태 | 본 프로젝트에서 참고할 점 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | **`cijiugechu/gpui-video-player`**<br>(https://github.com/cijiugechu/gpui-video-player) | GStreamer · **gpui** | ~45 | 2025~2026 (활발) | **gpui 직접 통합 1순위 참고 사례**. GStreamer 프레임을 gpui 커스텀 Element로 렌더링하고 재생 컨트롤을 구성하는 패턴 확인 가능 |
| 2 | **`everpcpc/syncplay-gpui`**<br>(https://github.com/everpcpc/syncplay-gpui) | mpv/외부연동 · **gpui** | 미확인 (~162 커밋) | 2025~2026 (활발) | gpui 앱 상태 모델(`Entity`, 비동기 태스크)과 외부 미디어 플레이어 상태를 동기화하는 구조 참고 |
| 3 | **`jazzfool/iced_video_player`**<br>(https://github.com/jazzfool/iced_video_player) | GStreamer · Iced (`wgpu`) | ~207 | 2025~2026 (활발) | GStreamer `AppSink`에서 YUV 프레임을 받아 **WGSL 셰이더로 GPU에서 YUV→RGB 변환** 및 A/V 싱크를 맞추는 깔끔한 표준 구현 |
| 4 | **`n00kii/egui-video`**<br>(https://github.com/n00kii/egui-video) | FFmpeg + SDL2 · egui | ~150 | 2025~2026 (활발) | **FFmpeg 디먹스·디코드·오디오 시계 동기화·탐색 상태 기계**를 순수 Rust 코드로 어떻게 짜는지 가장 직관적으로 보여주는 레퍼런스 |
| 5 | **`gyroflow/gyroflow`**<br>(https://github.com/gyroflow/gyroflow) | FFmpeg (D3D11VA 등) + `wgpu`/OpenCL · Qt/QML | ~9,700+ | 2026 (매우 활발) | **Windows D3D11 하드웨어 디코딩 텍스처(`AV_PIX_FMT_D3D11`)를 `wgpu`/셰이더와 무복사(Zero-copy) 인터롭**하는 프로덕션급 Rust 코드 보유 |
| 6 | **`nini22P/tauri-plugin-mpv`**<br>(https://github.com/nini22P/tauri-plugin-mpv) | mpv · **Tauri v2** | ~30 | 2025~2026 (활발) | Tauri 2 앱에서 `mpv`를 내장하고 WebView 프런트엔드와 이벤트/명령을 주고받는 구조 참고 |
| 7 | **`vgarleanu/lumiere`**<br>(https://github.com/vgarleanu/lumiere) | `libmpv` · Slint | 소규모 (예제) | 유지보수 | `libmpv`의 **Render API(오프스크린 프레임버퍼)**를 사용해 Rust 네이티브 GUI 텍스처 안에 비디오와 커스텀 OSD를 합성하는 방법 제시 |
| 8 | **`tsukinaha/tsukimi`**<br>(https://github.com/tsukinaha/tsukimi) | `libmpv` / GStreamer · GTK4 | ~1,000+ (추정) | 2025~2026 (활발) | 다중 트랙 선택(오디오/자막), 재생 속도, 하드웨어 디코딩 토글 등 본격적인 데스크톱 미디어 클라이언트 UI/UX 설계 참고 |
| 9 | **`oddity-ai/video-rs`**<br>(https://github.com/oddity-ai/video-rs) | `ffmpeg-next` · 라이브러리 | ~500+ (추정) | 2025~2026 (활발) | 복잡한 FFmpeg C API 호출을 Rust다운 빌더/이터레이터 패턴으로 감싸는 모듈 설계 참고 |
| 10 | **`philn/glide`**<br>(https://github.com/philn/glide) | GStreamer · GTK | ~600+ (추정) | 유지보수 | GStreamer `Player` API를 이용한 자막 트랙 전환, 오디오 트랙 전환, 탐색 바 처리 참고 |

---

## Q15 Windows 에서 가장 큰 위험 3가지와 사람들이 자주 틀리는 점

### 가장 큰 위험 3가지
1. **위험 1: 4K60 CPU 텍스처 업로드 및 CPU 색공간 변환 병목 (gpui/Tauri 공통)**
   - FFmpeg/GStreamer에서 디코딩한 프레임을 CPU 메모리(`Vec<u8>` RGBA)로 내려받아 `sws_scale`을 돌린 뒤 gpui `RenderImage`나 Tauri IPC로 넘기면 4K60에서 초당 ~2GB 메모리 복사가 일어나 재생이 끊긴다. 반드시 GPU 내 YUV→RGB 변환 또는 D3D11 무복사/자식 창 경로를 타야 한다.
2. **위험 2: 자식 창(HWND, `--wid`) 사용 시의 Airspace(Z-order)·DPI 스케일링·클리핑 충돌**
   - `libmpv --wid`로 자식 HWND를 탭 영역에 붙이면, 부모 창 안에 그려지는 gpui 요소나 WebView2 DOM(우클릭 컨텍스트 메뉴, 모달, 툴팁)이 자식 HWND 뒤로 가려진다. 또한 Windows 11 다중 모니터 DPI 변경(100% ↔ 150%) 시 논리 픽셀(`Pixels`)과 물리 픽셀(`scale_factor` 곱) 계산이 어긋나면 비디오 영역이 어긋난다.
3. **위험 3: D3D11 멀티스레드 컨텍스트 경합 및 FFmpeg D3D11VA `Texture2DArray` 수명 관리 실패**
   - FFmpeg D3D11VA 디코더 스레드와 렌더 스레드가 동일한 `ID3D11DeviceContext`를 락 없이 동시에 호출하면 드라이버 크래시가 난다(`ID3D11Multithread::SetMultithreadProtected(TRUE)` 필수). 또한 FFmpeg이 반환한 `ID3D11Texture2D`는 단일 텍스처가 아니라 **고정 크기 텍스처 배열의 슬라이스(`data[1]`)**이므로, `AVFrame` 참조를 큐에 너무 많이 쌓아두면 디코더 프레임 풀이 고갈되어 디코딩이 교착(Deadlock) 상태에 빠진다.

### 사람들이 자주 틀리는 점 (흔한 실수)
- **MinGW vs MSVC 혼합 실수**: Rust는 `x86_64-pc-windows-msvc`로 빌드하면서 GStreamer MinGW MSI를 설치하거나 `libclang` 없이 `ffmpeg-sys-next`를 빌드하려다 링커 에러에 막힌다.
- **GPL 필터 무의식적 사용**: FFmpeg을 LGPL로 쓴다고 생각하면서 R2 노이즈 제거에 `hqdn3d`, 색 보정에 `eq` 필터를 넣어버려 GPL 빌드에 강제 종속된다.
- **Tauri `convertFileSrc` 맹신**: 대용량 로컬 MKV 파일을 `convertFileSrc`로 `<video>`에 물렸다가 탐색(Seek)이 안 되고 AC3 소리가 안 나오는 문제를 겪는다.

---

## Q16 권장 문서 목차

`D:\projects\docs\guides\rust-video-player\` 문서 묶음의 권장 구성(총 7편)이다.

1. **`00-overview.md` — 아키텍처 개요 및 기술 선택 가이드**
   - 요구사항(범용 코덱, R1 GPU 가속, R2 실시간 필터, 다중 탭) 요약
   - 백엔드 3대장(libmpv vs FFmpeg D3D11VA vs GStreamer d3d11) 최종 결정 매트릭스
   - PotPlayer 기능과 자체 구현 컴포넌트 1:1 매핑표
2. **`01-pipeline-and-sync.md` — 미디어 파이프라인 · A/V 동기화 · 오디오/자막 설계**
   - 4-스레드 모델(Demux, Video, Audio, UI) 및 Lock-free/Bounded 큐 설계
   - WASAPI(`cpal` + `rubato`/`swresample`) 오디오 마스터 시계 수식과 프레임 드롭·속도 변경(`atempo`)
   - 키프레임 탐색 vs 정확 탐색 상태 기계, SRT 및 ASS(`libass`/`ass-renderer`) 오버레이
3. **`02-r1-r2-gpu-and-filters.md` — R1 하드웨어 가속(D3D11VA) 및 R2 실시간 영상 필터 구현**
   - D3D11VA 무복사(`ID3D11Texture2D` ArraySlice) 추출 및 `ID3D11Multithread` 안전 규칙
   - R1 런타임 HW⇄SW 전환 및 실패 시 자동 폴백 시퀀스
   - R2 구현: `ID3D11VideoProcessor`(디인터레이스·노이즈 제거) + HLSL/GLSL 픽셀 셰이더(CAS 선명·가우시안 블러·밝기/대비/채도)
4. **`03-gpui-integration.md` — `workbench-gpui` 탭 뷰어 통합 가이드 (1순위)**
   - `Workbench`의 `PaneItem` 확장 및 파일 트리 연동(`src/ui/mod.rs`, `layout.rs` 기준)
   - `DirectXRenderer` 위에서의 비디오 출력 전략(D3D11 텍스처 공유 vs `RenderImage` 최적화 vs 자식 HWND)
   - 탭 전환·분할(Split Pane)·숨김 탭 디코더 일시정지 및 VRAM 한계 관리
5. **`04-tauri-integration.md` — `workbench-tauri` 탭 뷰어 통합 가이드 (2순위)**
   - Solid(`solid-js`) 탭 레이아웃에서 비디오 뷰어 인스턴스 보존(`D-3`) 및 CSP 설정(`D-10`)
   - `libmpv` 자식 HWND(`--wid`) ResizeObserver 좌표 동기화 및 투명 오버레이 제어
   - 로컬 루프백 HTTP Range 서버(`206 Partial Content`) 보조 구성
6. **`05-build-bundle-license.md` — Windows 11 빌드 · DLL 번들링 · 라이선스 체크리스트**
   - `stable-msvc` 환경에서 FFmpeg(BtbN/vcpkg), libmpv(shinchiro), GStreamer(MSVC MSI) 설정
   - 설치본 패키징, 용량 최적화, LGPL vs GPL 필터(`hqdn3d`/`eq` 회피) 및 코덱 특허 점검표

---

## Q17 확신 낮은 부분

1. **`gpui-kit =0.8.1` (`gpui-pre 0.3.10`)의 Windows `DirectXRenderer` 외부 `ID3D11Texture2D` 직접 주입 API 노출 여부 (미확인)**
   - Upstream Zed(`gpui`)의 Windows DX11 렌더러는 내부적으로 `ID3D11Device`를 보유하고 있으나, 본 저장소가 의존하는 `gpui-kit =0.8.1`(`gpui-pre 0.3.10`)의 공개 API에서 `ID3D11ShaderResourceView`나 `ID3D11Texture2D`를 CPU 복사 없이 바로 끼워 넣는 훅이 열려 있는지는 `cargo` 빌드나 크레이트 소스 언패킹 없이 확인하지 못했다(미확인). 만약 닫혀 있다면 자식 HWND 방식이나 `gpui-kit` 래퍼 소폭 확장이 필요하다.
2. **GPU 드라이버별 `ID3D11VideoProcessor` 필터 작동 편차 (추정)**
   - `D3D11_VIDEO_PROCESSOR_FILTER_NOISE_REDUCTION` 및 `EDGE_ENHANCEMENT`는 표준 D3D11 API지만, 실제 동작 여부는 NVIDIA/AMD/Intel 드라이버의 `GetVideoProcessorCaps` 반환값에 따라 일부 최신 GPU에서 무시되거나 VSR(Video Super Resolution) 설정에 종속될 가능성이 있다(추정). 따라서 선명/블러/색 보정은 반드시 자체 HLSL/GLSL 픽셀 셰이더를 기본으로 두는 것이 안전하다.
3. **일부 GitHub 소규모 저장소의 정확한 최신 별 수 (추정)**
   - Q14의 `syncplay-gpui`, `tsukimi`, `video-rs`, `glide`의 일부 스타 수는 웹 검색 스니펫 시점 기준 근사치(추정)이며, 아키텍처 및 백엔드 구성 사실은 각 저장소 문서로 확인하였다.
