# scouter(sonnet) 조사 보고서 — Rust 동영상 플레이어 원점 설계 (Windows 11)

조사일: 2026-11-01. 범위: 웹검색 + crates.io API + GitHub PR 1건. 시간 제약으로 B2(UI 후보)의 일부 항목은 공식 문서를 직접 읽지 못했다. 그런 곳은 "미확인" 또는 "추정"으로 표시했다.

## 요약 (10줄)
1. 1순위 스택(추정): **winit + wgpu(DX12) + egui 위젯 + libmpv**. 또는 eframe 위에 libmpv를 얹는 구성.
2. 2순위 스택(추정): **Win32 + D3D11/DirectComposition 직접 + libmpv(D3D11 render API 또는 자식 창 `wid`)**. 성능과 제어가 가장 좋지만 UI를 직접 만들어야 한다.
3. 확인: libmpv에 **D3D11 render API**가 mpv master에 2026-10-15 병합됐다. 클라이언트 API 2.6이고, 마일스톤은 v0.42.0이다. 정식 릴리스 포함 여부는 미확인. https://github.com/mpv-player/mpv/pull/17764
4. 확인: Rust libmpv 바인딩 `libmpv2` 7.0.0 (2026-10-30). 저장소는 kohsine/libmpv-rs. https://crates.io/api/v1/crates/libmpv2
5. 확인: wgpu 30.0.1 (2026-08-22), egui/eframe 0.37.0 (2026-10-15), winit는 정식 0.30.13, 0.31.0-beta.4 (2026-10-24), slint 1.18.1, iced 0.14.0 (2025-12-07).
6. 확인: floem 0.2.0 (2024-11 이후 갱신 없음), makepad-widgets 1.0.0 (2025-05), xilem 0.4.0 (2025-10)은 **갱신이 뜸하거나 pre-1.0**이라 동영상 앱의 기반으로는 위험하다.
7. D3D11 디코더 텍스처를 wgpu로 복사 없이 표시하는 방법은 wgpu-hal DX12의 `texture_from_raw` + `OpenSharedHandle` 패턴이다. 서드파티 크레이트(mpv-engine, grafting 등)가 이 패턴을 쓴다. **wgpu 공식 API는 아니다.**
8. egui 생태계에는 egui_dock 0.21.1, egui_tiles 0.18.0이 있어 탭/분할이 가능하다. 트리 위젯은 별도 크레이트나 자체 구현이 필요하다 (미확인).
9. R2(선명/블러/노이즈/디인터레이스/밝기·대비·채도)는 libmpv 속성·`vf`·셰이더(`glsl-shaders`)로 가능하다. 자체 렌더러면 wgpu WGSL 패스로 구현한다. libmpv D3D11 경로에서 셰이더 지원 범위는 미확인.
10. 라이선스: libmpv는 기본 빌드가 GPLv2+ (mpv-engine README 인용)이다. 바이너리를 배포한다면 GPL/LGPL 의무를 확인해야 한다. gstreamer-rs 0.25.4는 MIT/Apache-2.0이다. 단, GStreamer 본체는 LGPL(추정)이다.

---

## B1 원점 권장 스택
**권장 A (추정): winit + wgpu(DX12 백엔드) + egui + libmpv**
- 이유:
  - egui는 0.37.0으로 활발히 갱신된다. https://crates.io/api/v1/crates/egui 에서 2026-10-15 확인.
  - 탭/분할 크레이트가 있다: egui_dock 0.21.1 (2026-08-06), egui_tiles 0.18.0 (2026-10-19). 각각 https://crates.io/api/v1/crates/egui_dock, https://crates.io/api/v1/crates/egui_tiles
  - 영상은 egui의 wgpu 콜백으로 직접 그린다. 영상 위 오버레이 UI는 egui의 즉시 모드라 쉽다 (추정).
- 약점: 즉시 모드 UI는 유휴 시 전력/재그리기 관리가 필요하다. 네이티브 느낌의 트리/탭은 직접 다듬어야 한다 (추정).
- 사례: MTT File Manager는 egui 기반이고 탭과 mpv 플레이어를 가졌으며 Windows 전용이고 libmpv-2.dll에 의존한다. https://awesome.ecosyste.ms/projects/github.com%2Fmttamurex%2Fmtt-file-manager-rust (검색 요약만 읽음)
- 디코딩은 libmpv (B4 참조).

**권장 B (추정): Win32(windows 크레이트) + D3D11 + DirectComposition + libmpv**
- 이유: 스왑체인, 프레임 타이밍, HDR, 전체화면을 완전히 제어할 수 있다. mpv D3D11 render API는 앱이 만든 `ID3D11Device`와 `ID3D11Texture2D`를 쓰므로 맞물린다.
- 약점: 트리/탭/오버레이 UI를 전부 직접 만들거나 별도 UI 계층이 필요하다. 개발 기간이 가장 길다 (추정).

## B2 창·UI 후보 비교
버전은 모두 crates.io API로 2026-11-01에 확인했다. URL은 `https://crates.io/api/v1/crates/<이름>`.

| 후보 | 확인 버전 / 갱신 | 동영상 적합성 | 트리·탭 위젯 | 유지 상태 |
|---|---|---|---|---|
| winit + wgpu 직접 | winit 0.30.13 정식 / 0.31.0-beta.4 (2026-10-24), wgpu 30.0.1 (2026-08-22) | swapchain·vsync·전체화면·HDR을 직접 제어. 위젯이 없다 | 없음 (자체 구현) | 활발. winit 0.31은 아직 베타 |
| egui(eframe) | 0.37.0 (2026-10-15) | wgpu/glow 백엔드. 영상 텍스처는 커스텀 렌더 콜백으로. 오버레이는 쉬움 (추정) | egui_dock, egui_tiles 있음. 트리는 `CollapsingHeader` 조합 (추정) | 매우 활발. 다운로드 약 2,930만 |
| iced | 0.14.0 (2025-12-07) | wgpu 기반. 커스텀 셰이더 위젯 가능 (추정). 영상 사례는 미확인 | 내장 pane_grid. 트리는 미확인 | 갱신 약 11개월 전. 2026-11 기준 정체 가능성 (추정) |
| slint | 1.18.1 (2026-09-21) | wgpu 연동 사례는 미확인. 렌더러는 skia/femtovg/software 등 (추정) | 선언형 UI, 탭 위젯 있음 (추정). 라이선스는 GPL/로열티프리/상용 3중 (추정, 미확인) | 활발 |
| floem | 0.2.0 (2024-11-14) | 미확인 | 미확인 | 크레이트 게시 기준 약 2년 정체 |
| makepad | makepad-widgets 1.0.0 (2025-05-13) | 자체 GPU 렌더러. 외부 텍스처 통합은 미확인 | 미확인 | 1.0 이후 갱신 미확인 |
| xilem | 0.4.0 (2025-10-29) | vello 기반. 영상 표면 통합은 미확인 | 미확인 | pre-1.0 실험적 |
| windows 크레이트 + Win32 + D3D11/DComp | windows 0.62.2 (게시일 2025-10-06) | 최고 수준의 제어 (추정) | 없음. 직접 구현 | 크레이트는 Microsoft 관리. 위 날짜가 최신인지는 미확인 |

- 그 밖의 후보: **egui-sharkplayer** 0.5.1 (2026-07-05)은 libmpv2 ^6.0과 eframe ^0.35에 의존하는 egui용 libmpv 위젯이다. 표시된 플랫폼은 Linux뿐이라 Windows 지원은 미확인이다. https://docs.rs/crate/egui-sharkplayer/latest
- 주의: 우리가 읽은 건 크레이트 메타데이터뿐이고, 각 UI 프레임워크의 공식 문서·이슈는 읽지 못했다. 표의 "동영상 적합성" 칸은 대부분 추정이다.

## B3 렌더링
**wgpu vs D3D11 직접**
- wgpu 장점: egui와 같은 디바이스를 공유하고, WGSL 셰이더로 R2를 한 번 구현하면 된다.
- wgpu 약점: 디코더(D3D11VA)의 D3D11 텍스처를 쓰려면 DX12 백엔드로 공유 핸들을 가져와야 한다.
- D3D11 직접 장점: D3D11VA 디코더와 같은 디바이스라 가장 단순하다.

**복사 없는 표시 경로 (확인된 패턴)**
- D3D11 쪽에서 `D3D11_RESOURCE_MISC_SHARED` 계열 플래그로 텍스처를 만들고 NT 핸들을 내보낸다.
- wgpu의 DX12 디바이스에서 `ID3D12Device::OpenSharedHandle`로 열고, `wgpu_hal::dx12::Device::texture_from_raw`로 감싼 뒤 `create_texture_from_hal`로 `wgpu::Texture`를 만든다.
- 출처: grafting https://docs.rs/crate/grafting/latest/source/src/dx12_shared_texture.rs , mpv-engine https://docs.rs/crate/mpv-engine/0.2.0/source/src/export/wgpu_windows.rs
- 조건:
  - 같은 어댑터(LUID)여야 한다.
  - 초기 리소스 상태가 실제와 일치해야 한다.
  - NV12를 직접 쓰려면 `Features::TEXTURE_FORMAT_NV12`(native 전용)가 필요하다.
  - NV12 공유는 `D3D11_SHARED_RESOURCE_TIER_2` 이상이 필요하다 (Dawn 테스트에서 언급, 하드웨어 의존).
  - 출처: mediaway https://docs.rs/crate/mediaway/latest/source/src/wgpu/dx12_decode.rs
- 한계: 한 크레이트(mediaway-decoder)는 자기 브리지를 **"Not Zero-Copy"**(D3D11→D3D11 `CopySubresourceRegion`)로 명시했다. 동기화(펜스·키드뮤텍스) 방법은 각 크레이트 코드를 직접 읽지 못했다 (미확인).
- wgpu 30에 외부 텍스처(YUV 샘플링) 공식 지원이 들어갔는지는 미확인이다.

**YUV→RGB 위치**
- 추정: 렌더 패스의 프래그먼트 셰이더에서 NV12의 Y/UV 두 평면을 샘플링해 RGB로 변환한다. BT.601/709/2020 행렬과 범위(limited/full)를 uniform으로 전달한다.
- **R2 셰이더를 둘 위치(추정)**: YUV→RGB 직후, 화면 출력 전의 별도 패스에 둔다. 밝기·대비·채도는 같은 셰이더에 uniform으로 넣고, 선명/블러/노이즈 제거는 이후 패스로 둔다. 디인터레이스는 디코더 직후 프레임 단위 패스로 둔다.

## B4 디코딩 바인딩
| 항목 | libmpv (libmpv2 7.0.0) | FFmpeg (ffmpeg-next 9.0.0) | GStreamer (gstreamer 0.25.4) |
|---|---|---|---|
| 확인 날짜 | 2026-10-30 | 2026-08-05 | 2026-09-21 |
| 역할 | 완성형 플레이어 코어 (자막, 트랙, 속도, hwdec 포함) | 저수준 디코더, 나머지는 직접 구현 | 파이프라인 프레임워크 |
| 렌더 통합 | OpenGL/SW render API. **D3D11 render API는 mpv master에 병합** (아래 참조) | 프레임을 받아 직접 렌더링 | appsink/GL/D3D11 메모리 (추정) |
| 크레이트 라이선스 | 크레이트 자체는 미확인. 바이너리 libmpv는 기본 GPLv2+ | 미확인 (FFmpeg 본체는 LGPL/GPL 빌드 옵션에 따름, 추정) | MIT OR Apache-2.0 (크레이트) |

출처: https://crates.io/api/v1/crates/libmpv2 , https://crates.io/api/v1/crates/ffmpeg-next , https://crates.io/api/v1/crates/gstreamer

**mpv D3D11 render API (핵심 발견)**
- 앱이 `ID3D11Device`를 `MPV_RENDER_PARAM_D3D11_INIT_PARAMS`로 넘기고, 매 프레임 `MPV_RENDER_PARAM_D3D11_FBO`로 대상 `ID3D11Texture2D`를 넘긴다. 스왑체인 생성과 Present는 앱이 한다.
- 2026-10-15 master에 병합됐다 (commit 4255130). 클라이언트 API 2.6이고 마일스톤은 v0.42.0이다. 정식 릴리스 포함 여부는 미확인이다.
- 병합 시 리뷰어(kasper93)는 Windows D3D11에서는 창 임베딩이 더 나을 수 있고 render API는 제한적이라고 했으나, 제한 내용은 적지 않았다.
- 출처: https://github.com/mpv-player/mpv/pull/17764
- `libmpv2` 7.0.0이 이 D3D11 파라미터를 래핑하는지는 **미확인**이다. 없으면 `libmpv-sys` 수준의 FFI로 직접 호출해야 한다.
- 현재 확인된 Rust 쪽 경로: libmpv2는 OpenGL render API 예제(SDL2)를 제공한다. https://docs.rs/crate/libmpv2/5.0.3/source/README.md

**통합 방법 후보**
1. libmpv render API(D3D11): 앱이 텍스처를 소유하고, mpv가 그 텍스처에 그리고, 앱이 egui/wgpu 또는 D3D11 쪽에서 합성한다. 위 B3 공유 텍스처 경로가 필요하다.
2. libmpv `wid`(자식 창): mpv가 자식 HWND에 직접 그린다. 단순하지만 egui 위에 오버레이를 그릴 수 없다 (HWND 에어스페이스 문제, 추정). 탭 전환 시 HWND 표시/숨김으로 처리한다.
3. mpv-engine 0.2.1 (2026-09-07)은 wgpu로 내보내는 옵션 기능을 제공한다. 다운로드 444건으로 채택이 거의 없다. https://crates.io/api/v1/crates/mpv-engine
4. FFmpeg: 디코딩과 프레임 전달은 앱이 담당하며 자막·트랙·필터를 직접 구현해야 한다.

## B5 파일 탐색기 + 탭 UI (egui 기준 추정)
- **트리**: 직접 구현이 현실적이다 (`CollapsingHeader` 재귀 + 가상화). 검색에서 전용 트리 크레이트는 확인하지 못했다 (미확인).
- **탭/분할**: egui_dock 0.21.1, egui_tiles 0.18.0. 참고 구현으로 Playa의 `DockTabs`(egui_dock `TabViewer` 래퍼)가 있다. https://docs.rs/crate/playa/latest/source/src/app/tabs.rs
- **탭 안 동영상 표면**:
  - 단일 wgpu 텍스처를 탭 영역에 그리면 되므로 자식 창이 필요 없다.
  - 숨김 탭의 재생은 정책이 필요하다. 일시정지하거나 디코딩만 유지(오디오 뮤트)하거나 렌더링만 건너뛴다. libmpv 쪽에는 `MPV_RENDER_PARAM_SKIP_RENDERING`이 있다 (docs.rs libmpv-sys에서 상수 확인, 의미는 추정).
  - 여러 탭 동시 재생은 mpv 인스턴스를 탭마다 따로 만든다. 활성 탭에만 오디오 출력을 허용한다 (추정).
- 자식 창 방식은 탭 전환 때마다 HWND 관리가 필요해 복잡하다 (추정).

## B6 코어·UI 분리 (추정)
```
player-core   (UI 무관: 세션, 명령 enum, 이벤트 enum)
player-mpv    (libmpv 래퍼, 렌더 컨텍스트)
render        (wgpu / D3D11 공유 텍스처, 필터 패스)
app-ui        (egui, 트리, 탭)
app           (main, 설정)
```
- 스레드: UI 스레드 / mpv 이벤트 스레드 / 렌더 스레드(필요 시).
- 채널: `Command`(UI→코어, mpsc), `Event`(코어→UI, mpsc + 필요 시 `request_repaint`). mpv-engine이 "typed events"와 "render seams" 구조를 이미 제공한다. https://crates.io/api/v1/crates/mpv-engine

## B7 기능별 구현 방법·난이도 (추정 포함)
| 기능 | 방법 | 난이도 |
|---|---|---|
| R1 GPU 가속 ON/OFF | libmpv `hwdec` 속성(`no` / `auto-safe` 등)을 재생 중 변경. 실패 감지는 이벤트·로그로 하고 `no`로 전환 | 중. 속성 변경 시 재초기화 동작은 미확인 |
| R1 실패 시 자동 전환 | mpv는 `auto-safe`/`auto-copy` 폴백 경로가 있음 (추정). 앱은 로그/속성으로 현재 hwdec 상태를 읽어 UI에 표시 | 중 |
| R2 밝기·대비·채도 | mpv `brightness/contrast/saturation` 속성 (추정) 또는 WGSL uniform | 낮음 |
| R2 선명/블러/노이즈 | mpv `vf`(예: unsharp, hqdn3d)나 user shader, 또는 WGSL 패스 | 중 |
| R2 디인터레이스 | mpv `deinterlace` 속성·`vf=yadif`/`bwdif` (추정) | 낮음~중 |
| 탐색·속도·음량·트랙·자막(SRT/ASS) | libmpv 속성·명령으로 거의 그대로 제공됨 (libmpv 일반 기능) | 낮음 |
| 전체 화면 | winit 전체화면 + 오버레이 UI 숨김 | 낮음~중 |
| HDR | wgpu 스왑체인 포맷/색공간 지원 여부는 미확인 | 높음 |

R2 구현 방식은 libmpv 필터에 맡기면 가장 빠르고, 자체 WGSL 패스는 libmpv 렌더 경로 밖에서 후처리할 때만 의미가 있다 (추정).

## B8 후보 스택 비교표 (추정 중심)
| 스택 | 개발 기간 | 성능 | R1 | R2 | 배포 크기 | 라이선스·유지 위험 |
|---|---|---|---|---|---|---|
| A. egui + wgpu + libmpv | 짧음 | 좋음 (공유 텍스처 성공 시) | libmpv hwdec | mpv 필터 + WGSL | 중 (libmpv DLL 포함) | libmpv GPL, egui 활발 |
| B. Win32 + D3D11 + libmpv | 김 | 최고 | libmpv hwdec | mpv 필터 | 작음 | libmpv GPL, UI 직접 구현 |
| C. egui + wgpu + FFmpeg 직접 | 매우 김 | 좋음 | D3D11VA 직접 연동 필요 | WGSL 전부 직접 | 중 | FFmpeg LGPL 동적 링크 조건 (추정) |
| D. slint + libmpv | 중 | 미확인 | libmpv | mpv 필터 | 중 | slint 라이선스 확인 필요 (미확인) |

## B9 권장 목차
1. 목표·범위·PotPlayer 기능 기준
2. 후보 스택 비교와 결정 (B1, B2, B8)
3. 디코딩 엔진 선택 (B4)과 라이선스
4. 렌더링 파이프라인 (B3)
5. 플레이어 코어 구조 (B6)
6. 파일 탐색기·탭 UI (B5)
7. R1 하드웨어 디코딩 전환
8. R2 영상 옵션
9. 기능별 구현표 (B7)
10. 검증 계획과 위험

## B10 확신 낮은 부분
- **libmpv D3D11 render API가 정식 릴리스에 들어갔는지**: PR은 master에 병합됐고 마일스톤은 v0.42.0이다. 릴리스 여부와 `libmpv2` 7.0.0의 래핑 여부는 미확인.
- **wgpu 30에서 외부 텍스처·NV12 공식 지원 범위**: 서드파티 코드에서만 확인했다. 동기화 방식은 미확인.
- **slint, iced, floem, makepad, xilem의 영상 통합 사례**: 미확인. 버전·날짜만 확인했다.
- **windows 크레이트 0.62.2가 2026-11 기준 최신인지**: crates.io에서는 이 값이 반환됐으나 게시일이 2025-10이라 이상하다. 미확인.
- **egui-sharkplayer의 Windows 지원 여부**: docs.rs에는 Linux만 표시됐다.
- **HDR, 숨김 탭 동시 재생 성능, 전력 소비**: 추정이며 검증이 필요하다.
- **라이선스**: libmpv 기본 빌드가 GPLv2+라는 점은 mpv-engine README를 통한 간접 확인이다. LGPL 빌드 가능 여부와 FFmpeg·GStreamer 라이선스는 미확인.
- 이 보고서는 공식 문서 대신 검색 요약과 크레이트 메타데이터에 크게 의존했다. leader가 다른 scouter 결과와 교차 확인하길 권한다.
