---
tags: [rust, video-player, comparison, risk, checklist]
created: 2026-11-01
updated: 2026-11-01
---

# 07 · 방식 비교와 최종 선택

> 이 파일을 읽는 때: 어떤 조합으로 갈지 정할 때, 위험과 흔한 오해를 확인할 때, 구현 후 검증 계획을 세울 때.

입구는 [README.md](README.md)다. 각 방식의 상세는 해당 파일에 있고 여기서는 비교와 결론만 쓴다.

## 1. 방식별 비교

R1·R2 칸의 "위임"은 mpv 속성·필터로 해결된다는 뜻이다.

### 1.1 gpui

| 방식 | 4K60 | R1·R2 | 범용 재생 | 구현량 | 오버레이 | 상세 |
| --- | --- | --- | --- | --- | --- | --- |
| A. libmpv + 전용 자식 HWND | 가능 (mpv 무복사) | 위임 | mpv 빌드에 달림 | 작음 | 영상 위 겹침 제약 → 컨트롤을 밖에 | [03-GPUI.md](03-GPUI.md) 3.2 |
| B. libmpv D3D11 render API + gpui 합성 | 가능 (이론) | 위임 | 같음 | 큼. mpv master·FFI·gpui renderer 변경 | 없음 | [03-GPUI.md](03-GPUI.md) 3.3 |
| C. FFmpeg 디코딩 + `RenderImage` + cpal | 병목 (추정) | 직접 구현 | 직접 구현 | 큼 | 없음 | [03-GPUI.md](03-GPUI.md) 3.1 |
| D. GStreamer appsink + `RenderImage` | 병목 (추정) | 부분 | playbin3 | 중 | 없음 | [02-DECODING.md](02-DECODING.md) 3절 |

### 1.2 Tauri

| 방식 | 4K60 | R1·R2 | 범용 재생 | 구현량 | 오버레이 | 상세 |
| --- | --- | --- | --- | --- | --- | --- |
| E. libmpv + 자식 HWND | 가능 | 위임 | mpv 빌드에 달림 | 작음~중 | 겹침 제약 | [04-TAURI.md](04-TAURI.md) 4절 |
| F. libmpv render API(ANGLE) + 투명 WebView | 가능 | 위임 | 같음 | 큼 (hit-testing·z-order) | DOM 컨트롤이 영상 위에 보임 | [04-TAURI.md](04-TAURI.md) 5.1 |
| G. WebView2 `<video>` | 하드웨어 디코딩 여부 제어 불가 | R1 불가, R2 부분 | 제한적 | 최소 | DOM | [04-TAURI.md](04-TAURI.md) 3절 |
| H. TextureStream (실험) | GPU 경로 | 불확실 | GStreamer | 중 | DOM | [04-TAURI.md](04-TAURI.md) 5.2 |
| I. Rust 디코딩 프레임 IPC | 불가 (약 1.99 GB/s) | 직접 | 직접 | 큼 | DOM | [04-TAURI.md](04-TAURI.md) 5.3 |

## 2. 최종 선택

| 대상 | 선택 | 순서 |
| --- | --- | --- |
| gpui | **A. libmpv + 전용 자식 HWND**, 컨트롤은 영상 밖 | ① 세션·`PaneItem` 구조 → ② `RenderImage` PoC 로 탭·세션 흐름 검증 → ③ 자식 HWND 로 교체 → ④ (장래) B 검토 |
| Tauri | **E. libmpv + 자식 HWND**, 컨트롤은 영상 밖 | ① 탭 모델·세션 레지스트리·`set_bounds` → ② 자식 HWND → ③ DOM 오버레이가 꼭 필요하면 F 검토 |
| 공통 | 같은 `PlayerSession`, 같은 libmpv 설정, R1·R2 는 mpv 속성·필터 | [01-ARCHITECTURE.md](01-ARCHITECTURE.md) |
| 라이선스 | GPL 허용 | [05-WINDOWS.md](05-WINDOWS.md) 5절 |

선택 이유 한 줄씩:
- 요구 기능(R1·R2·자막·트랙·탐색)을 가장 적은 구현량으로 충족한다. https://mpv.io/manual/stable/
- gpui 는 Windows 에 외부 영상 표면 경로가 없어 `RenderImage`(CPU)뿐이다. https://docs.rs/gpui-pre/0.3.10/gpui/struct.Window.html
- `<video>` 는 범용·R1·R2 를 보장하지 못한다. https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/develop/media-authoring-processing/supported-codecs.md
- D3D11 render API 는 master 전용이라 지금은 쓰지 않는다. https://github.com/mpv-player/mpv/releases/tag/v0.41.0

### 2.1 gpui 와 Tauri 의 장단점

두 방식 모두 **엔진은 libmpv, 출력은 탭 전용 자식 HWND** 로 같다. 그래서 재생 품질·기능(R1·R2·자막·트랙·4K60)은 같고, 차이는 UI 를 무엇으로 만드는가다.

| | gpui + libmpv 자식 HWND (A) | Tauri + libmpv 자식 HWND (E) |
| --- | --- | --- |
| 장점 | Rust 한 언어, 한 프로세스 | 컨트롤·메뉴·옵션 패널을 HTML/CSS(Solid)로 빨리 만든다 |
| | 장래 경로가 있다: mpv D3D11 render API 가 정식 출시되면 gpui D3D11 렌더러 안에 합성해 airspace 문제를 없앨 수 있다 ([03-GPUI.md](03-GPUI.md) 3.3) | 영상 위 DOM 오버레이(ANGLE + 투명 WebView)를 쓴 실제 사례가 있다. https://github.com/kaiserbh/cathode |
| | workbench-gpui 가 1순위라 기존 계획과 맞는다 | mpv 플러그인·사례가 있다. https://github.com/nini22P/tauri-plugin-libmpv |
| 단점 | **gpui 창 HWND 를 얻는 방법이 미확인** (가장 큰 불확실성, 6절) | 좌표 단계가 많다(CSS 픽셀 → devicePixelRatio → `SetWindowPos`). 리사이즈·분할·스크롤마다 맞춘다 ([04-TAURI.md](04-TAURI.md) 4절) |
| | gpui-kit 컴포넌트가 적어 탐색 바·음량·옵션 패널을 직접 만드는 양이 많다 | Rust + JS 두 언어, IPC 명령·이벤트 설계가 필요하다 |
| | `gpui-pre` 를 버전 고정해야 하고 업스트림 변화를 따라가는 부담이 있다 | WebView2 런타임 의존. 자식 창 airspace 문제는 gpui 와 같다 |
| 공통 위험 | 자식 창이 메뉴·툴팁을 가린다(컨트롤을 영상 밖에), DLL·ABI 배포, GPU·드라이버별 하드웨어 디코딩 편차 (3절) | 같음 |

### 2.2 최종 제안 — gpui 주 경로, Tauri 예비 경로

leader 제안 (2026-11-01). 구현 전에 사람이 확정한다.

| 단계 | 할 일 | 끝내는 조건 |
| --- | --- | --- |
| 0. spike (약 1일, 추정) | gpui 창에서 HWND 를 얻어 탭 영역에 자식 창을 붙이고 `libmpv2` 로 영상 하나를 재생한다. DPI 변경·리사이즈 때 위치를 맞춘다 | 재생되고 위치가 맞으면 1 로. 막히면 3 으로 |
| 1. gpui 구조 | `PlayerSession`·`PaneItem` 을 만든다 ([01-ARCHITECTURE.md](01-ARCHITECTURE.md), [03-GPUI.md](03-GPUI.md) 2절) | 텍스트 탭과 영상 탭이 같은 탭 바에서 열린다 |
| 2. gpui 출력·컨트롤 | 자식 HWND 출력, 컨트롤은 영상 아래. R1·R2 는 mpv 속성·필터 | 5절 검증 체크리스트 통과 |
| 장래 | mpv D3D11 render API 가 정식 릴리스되고 바인딩이 지원하면 gpui 합성으로 옮긴다 | airspace 제약 제거 |
| 3. 예비: Tauri | spike 에서 HWND 를 얻을 수 없거나 gpui 렌더러와 충돌하면 Tauri(E)로 간다. 엔진·세션 코드는 그대로 쓰고 UI 만 바꾼다 | 같은 체크리스트 |

이유:
- 엔진이 같아 재생 품질·기능은 두 방식이 같다. 차이는 UI 와 장래성이다.
- airspace 문제를 근본적으로 없앨 길(D3D11 합성)은 gpui 에만 있다.
- 사용자 계획(gpui 1순위, Rust 중심)과 맞는다.
- Tauri 의 장점(HTML 로 빠른 UI)은 예비 경로로 살릴 수 있고, 엔진·세션을 공유하므로 전환 비용이 UI 에 한정된다.

## 3. 위험 3가지

| # | 위험 | 내용 | 대응 |
| --- | --- | --- | --- |
| 1 | **합성과 GPU interop** | 자식 HWND 는 gpui 요소·DOM 위를 덮는다(airspace). DPI·분할·숨김 때 좌표가 어긋난다. D3D11VA 가 켜져도 CPU copy-back·필터·atlas 업로드가 있으면 4K60 이 무너진다. 서로 다른 device/adapter·HDR·device loss, render/core 데드록이 겹친다 | 컨트롤을 영상 밖에. `set_bounds` 를 모든 layout 변경에 연결. 자식 창과 core 를 정해진 순서로 해제 |
| 2 | **배포 ABI·구성·권리** | MSVC 와 MinGW 산출물 혼용, DLL·import lib·header 버전 차이, plugin 누락, GPL 필터·코덱 특허를 빼먹기 쉽다 | 버전 한 세트 고정. 깨끗한 Windows 에서 DLL 검색 확인. 라이선스 manifest 작성 |
| 3 | **환경별 지원 편차** | WebView2/OS 코덱, Windows N 에디션, 24H2 의 AC3, GPU 별 codec profile 이 달라 개발 PC 에서만 재생될 수 있다. 하드웨어 디코딩 실패·재생 중 전환 실패가 흔하다(추정) | 자동 소프트웨어 전환을 반드시 구현. 지원 환경 표로 검증 |

출처: sol Q15 https://github.com/mpv-player/mpv/blob/master/include/mpv/render_d3d11.h , https://ffmpeg.org/legal.html , https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/develop/media-authoring-processing/supported-codecs.md ; sonnet Q15; argon Q15. 세 보고서의 위험 3가지는 표현이 달라도 같은 영역에 모인다.

## 4. 자주 틀리는 점

| 틀린 전제 | 사실 | 출처 |
| --- | --- | --- |
| gpui 가 DX11 이니 mpv render API 를 바로 연결 | D3D11 render API 는 master 전용, `libmpv2` 미지원, gpui 에 외부 texture 경로 없음 | https://github.com/mpv-player/mpv/commit/4255130de410e0cbdab694b951085409d466b704 |
| `paint_surface` 가 Windows 무복사 API | macOS/iOS 전용. Windows 는 빈 함수 | https://docs.rs/gpui-pre/0.3.10/gpui/struct.Window.html |
| GPU ON 이면 copy 0회 | CPU 필터·copy 방식·자막 합성이 있으면 복사가 생긴다 | [05-WINDOWS.md](05-WINDOWS.md) 1.4 |
| `<video>` 로 MKV/AC3/HEVC 가 다 된다 | 컨테이너·코덱 조합과 OS·확장에 달렸다. 24H2 에서 AC3 기본 제외 | [04-TAURI.md](04-TAURI.md) 3절 |
| Tauri asset 프로토콜은 Range 가 안 된다 | 내장 핸들러가 206·416 을 처리한다 (실기 검증 필요) | https://github.com/tauri-apps/tauri/blob/dev/crates/tauri/src/protocol/asset.rs |
| mpv `sharpen` 은 어디서나 된다 | `vo=gpu` 전용 | [06-FILTERS.md](06-FILTERS.md) 1절 |
| `d3d11vpp` 가 노이즈 제거도 한다 | 디인터레이스·스케일·Super Resolution 만 | [06-FILTERS.md](06-FILTERS.md) 1절 |
| 바인딩이 MIT/LGPL 이면 전체가 그렇다 | 네이티브 DLL·FFmpeg 구성·코덱 특허가 따로 있다 | [05-WINDOWS.md](05-WINDOWS.md) 5.3 |
| decode 만 하니 LGPL / mpv 는 항상 LGPL | 둘 다 아니다 | https://ffmpeg.org/legal.html , https://github.com/mpv-player/mpv/blob/master/Copyright |
| 탐색 뒤 큐·오디오 버퍼를 안 비워도 된다 | 소리가 어긋난다 (직접 구현 시) | [01-ARCHITECTURE.md](01-ARCHITECTURE.md) 3.4 |
| 오디오 콜백 안에서 잠금·할당 | 금지 | [05-WINDOWS.md](05-WINDOWS.md) 2.2 |
| `RenderImage` 를 만들고 `drop_image` 를 안 부른다 | GPU 아틀라스 메모리가 늘어난다 (추정) | [03-GPUI.md](03-GPUI.md) 3.1 |
| 숨긴 탭도 디코딩을 계속해도 된다 | 창만 숨기면 디코더 사용량이 안 준다 | [01-ARCHITECTURE.md](01-ARCHITECTURE.md) 4절 |
| `gpui-pre`·`libmpv2` 를 느슨하게 둔다 | 업그레이드로 깨진다. `libmpv2` 는 6.0.1 → 7.0.0 이 1일 간격 | [02-DECODING.md](02-DECODING.md) 4.1 |

## 5. 검증 체크리스트

**미실행 설계 제안이다.** 이 가이드는 조사만 했다. 구현 후 아래를 확인한다 (sol Q15, sonnet 보고서).

### 5.1 환경 매트릭스

| 축 | 값 |
| --- | --- |
| GPU | Intel / AMD / NVIDIA, 내장·외장 혼합 |
| 모니터 | DPI 변경 (100% ↔ 150%), 다중 모니터 |
| OS | Windows 11 24H2 클린 설치 |

### 5.2 시나리오

| 영역 | 확인할 것 |
| --- | --- |
| 코덱 | H.264 8-bit, HEVC Main10, VP9, AV1, AC3, Opus |
| 컨테이너 | MKV 다중 트랙 + ASS 폰트, TS 인터레이스, VFR, 손상 파일 |
| R1 | 반복 hwdec 토글, 실패 시 소프트웨어 전환, `hwdec-current` 표시, 전환 중 소리 끊김 |
| R2 | 필터 슬라이더 드래그, 프레임 끊김 여부, CPU 필터 시 CPU/GPU 사용률, `vf` 변경 시 끊김 |
| 탭 | 분할, 숨김, 동시 재생, 전체 화면 복원, 닫기 시 해제 순서 |
| 합성 | 자식 HWND 좌표 (DPI·사이드바 드래그·최대화), 컨트롤 가림 |
| 오디오 | 장치 분리·변경 |
| 파일 제공 | Tauri `<video>` 경로의 Range 탐색 (사용하는 경우) |

### 5.3 측정 항목

실제 hwdec, CPU/GPU 사용률, dropped frames, A/V drift, 메모리(VRAM 포함), device loss 복구. 통계는 mpv 속성으로 얻는다. https://mpv.io/manual/stable/#property-list

## 6. 확신이 낮은 부분

| 항목 | 상태 |
| --- | --- |
| WebView2 의 정확한 코덱 지원표 | 공식 자료 **미확인**. 포럼 글만 |
| gpui 의 창 HWND 노출(`raw-window-handle`)과 자식 창 생성 | **미확인** |
| gpui 에서 프레임마다 `RenderImage` 교체 시 아틀라스 해제·성능 | `drop_image` 존재만 확인, 실측 없음. 1080p60·4K60 한계는 계산 추정 |
| upstream main 과 `gpui-pre 0.3.10` 의 차이 (Windows `paint_surface` 등) | **미확인** |
| 배포 mpv DLL 의 실제 client API·기능, D3D11 render API 의 정식 릴리스 날짜 | **미확인** |
| lavfi 필터를 걸 때 mpv 의 하드웨어 프레임 처리 | **미확인** |
| `hwdec` 전환의 끊김, `vf` 변경의 프레임 끊김 | **미측정** |
| 하드웨어 디코더 동시 세션 수, 4K HEVC 탭당 VRAM(200~400 MB) | 수는 **미확인**, VRAM 은 **추정** |
| `ffmpeg-next` 9.0.0 과 FFmpeg 9 DLL 의 MSVC 빌드·링크 | **미검증** |
| `tauri-plugin-video` 의 안정 Runtime 지원, 속도·정확 탐색 | **미확인** |
| `alhazen-core` 의 AC3 주장, R1 OFF, ASS, R2 | **미검증** |
| `rubato`, `ass-renderer`, `atadenoise`, `cas` 필터의 라이선스 등 argon 단독 값 | **미확인** |
| PotPlayer 의 메뉴·알고리즘·기본값 | **미확인**. 기능 이름 기준으로만 대응 |
| 설치본 크기, 노트북 전력, GPU별 denoise 품질 | **미측정** |
| mpv LGPL 빌드 옵션 이름 | 확인됨: meson `-Dgpl=false` ([05-WINDOWS.md](05-WINDOWS.md) 5절). 이 옵션만으로 라이선스 조건이 모두 정리되지는 않는다 |
| 코덱 특허·법률 | 이 조사 범위 밖 |

## 7. 보고서와 다르게 쓴 값 (판정 요약)

세 scouter 보고서가 어긋난 값은 leader 가 출처를 직접 열어 판정했다. 전체는 [research/reviews/comparison.md](research/reviews/comparison.md) 2절.

| 항목 | 채택 | 본문 위치 |
| --- | --- | --- |
| `libmpv2` 최신 | 7.0.0 (2026-10-30) | [02-DECODING.md](02-DECODING.md) 4.1 |
| mpv D3D11 render API | master 전용 | [03-GPUI.md](03-GPUI.md) 3.3 |
| Tauri asset Range | 내장 핸들러가 처리 | [04-TAURI.md](04-TAURI.md) 3.1 |
| mpv `sharpen` | `vo=gpu` 전용 | [06-FILTERS.md](06-FILTERS.md) 1절 |
| `d3d11vpp` | 노이즈 제거 없음 | [06-FILTERS.md](06-FILTERS.md) 1절 |
| `egui-video` | 정체 | [02-DECODING.md](02-DECODING.md) 6절 |
| `gpui-video-player` 별 | 97 | [03-GPUI.md](03-GPUI.md) 5절 |
| `rsmpeg` | 최근 릴리스 2025-08-24 | [02-DECODING.md](02-DECODING.md) 5.1 |
| Tauri mpv 플러그인 | 둘 다 nini22P, 방식이 다름 | [04-TAURI.md](04-TAURI.md) 4.3 |
| libmpv DLL 크기 | 빌드마다 다름 | [05-WINDOWS.md](05-WINDOWS.md) 4.6 |
| `ffmpeg-next` | 유지보수 모드 | [02-DECODING.md](02-DECODING.md) 5.1 |

---

이전: [06-FILTERS.md](06-FILTERS.md) · 입구: [README.md](README.md)
