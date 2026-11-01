---
tags: [rust, video-player, mpv, gpui, tauri, windows, guide]
created: 2026-11-01
updated: 2026-11-01
---

# Rust 동영상 플레이어 구현 가이드 — gpui·Tauri 기반 (Windows 11)

> 이 파일을 읽는 때: 처음, 그리고 무엇을 읽을지 모를 때. 이 문서 묶음의 입구다.

workbench-gpui(gpui-kit) 또는 workbench-tauri(Tauri 2) 위에 올리는 **별도 앱**의 동영상 재생 설계를 정리한다. 파일 트리에서 동영상을 열면 **탭 안에서 재생**된다. 재생 범위는 PotPlayer 급 범용 재생이고, 세 가지 요구가 붙는다.

| 요구 | 뜻 |
| --- | --- |
| R1 | GPU 가속(하드웨어 디코딩)을 켜고 끈다. 재생 중에도 바꾼다. 실패하면 소프트웨어로 자동 전환한다 |
| R2 | 영상 옵션을 실시간으로 조절한다. 선명·흐림·노이즈 제거·디인터레이스·밝기/대비/채도 |
| R3 | 유튜브 링크를 클릭하면 새 탭에서 같은 플레이어 화면으로 재생한다. R1·R2 가 그대로 적용된다. 상세는 공통 문서 [../rust-video-player-youtube.md](../rust-video-player-youtube.md) (2026-11-01 추가, 작성 중) |

| 항목 | 내용 |
| --- | --- |
| 대상 | workbench-gpui / workbench-tauri 위에 동영상 뷰어 탭을 붙이려는 사람 |
| 결과 | 엔진 선택, 탭 통합 방식, R1·R2 구현 수단, 빌드·배포 주의, 검증 체크리스트 |
| 기준 | Windows 11, Rust stable-x86_64-pc-windows-msvc, 버전·별·커밋 확인일 2026-11-01 |
| 검증 범위 | **조사만 했다.** 패키지 설치, cargo 빌드, 앱 실행, 실제 영상 재생 검증은 하지 않았다. 버전·날짜·라이선스는 crates.io·GitHub·공식 문서를 열어 확인한 값이고, 성능 수치는 계산값이거나 추정이다. 코드 조각은 모두 설계 예시다 |

## 1. 결론 요약 (권장 조합 한 장)

| 항목 | gpui 앱 | Tauri 앱 |
| --- | --- | --- |
| 엔진 | libmpv + 바인딩 `libmpv2` 7.0.0 (2026-10-30) https://crates.io/api/v1/crates/libmpv2 | 같은 libmpv 를 Rust 쪽에 둔다 |
| 이유 | 디코딩·탐색·트랙·자막·A/V 동기·R1·R2 를 mpv 가 이미 갖췄다 https://mpv.io/manual/stable/ | 같음. `<video>` 단독은 범용·R1·R2 를 보장하지 못한다 |
| 화면 출력 | ① `RenderImage` PoC → ② **탭 전용 자식 HWND(실용안)** → ③ D3D11 render API(장래, mpv master 전용) | **자식 HWND**. 대안은 투명 WebView·TextureStream(실험) |
| 컨트롤 UI | gpui 요소. 영상 **밖**에 둔다 | Solid DOM. 영상 **밖**에 둔다 |
| R1 | `hwdec` 속성 `auto-safe` ↔ `no`. 실제 상태는 `hwdec-current` 로 표시 https://mpv.io/manual/stable/#video | 같음 |
| R2 | mpv 속성(밝기·대비·채도) + lavfi 필터·셰이더(나머지) | 같음 |
| 라이선스 | **GPL 허용**(사용자 결정). 일반 libmpv 빌드와 `eq`·`hqdn3d` 를 쓸 수 있다 | 같음 |
| 가장 큰 위험 | 합성(자식 HWND 겹침)·GPU interop, 배포 ABI·라이선스, 환경별 하드웨어 디코딩 편차 | 같음 + WebView 위 네이티브 창 겹침 |

판단 근거는 [07-COMPARISON.md](07-COMPARISON.md)에 있다. gpui 와 Tauri 중 무엇으로 시작할지는 같은 파일 2.2 절의 제안(**gpui 주 경로, Tauri 예비 경로**, 먼저 HWND spike)을 본다.

## 2. 읽기 지도

| 지금 할 일 | 읽을 파일 |
| --- | --- |
| 요구와 공통 구조(세션·상태 기계·탭 정책)를 잡는다 | [01-ARCHITECTURE.md](01-ARCHITECTURE.md) |
| 엔진(libmpv·FFmpeg·GStreamer…)을 고른다 | [02-DECODING.md](02-DECODING.md) |
| gpui 앱에 탭으로 붙인다 | [03-GPUI.md](03-GPUI.md) |
| Tauri 앱에 탭으로 붙인다 | [04-TAURI.md](04-TAURI.md) |
| GPU 가속·소리·자막·빌드·배포를 정한다 | [05-WINDOWS.md](05-WINDOWS.md) |
| 영상 옵션(R2)과 PotPlayer 대응을 정한다 | [06-FILTERS.md](06-FILTERS.md) |
| 방식을 비교하고 검증 계획을 세운다 | [07-COMPARISON.md](07-COMPARISON.md) |
| 유튜브 링크 재생(R3)을 붙인다 | [../rust-video-player-youtube.md](../rust-video-player-youtube.md) (묶음 A·B 공통) |
| 조사 원자료를 확인한다 | [research/README.md](research/README.md) |

| 파일 | 담은 것 |
| --- | --- |
| README.md | 입구, 결론 요약, 검증 범위, 읽기 지도 |
| 01-ARCHITECTURE.md | 요구 정의, 공통 `PlayerSession`, 파이프라인·A/V 동기·탐색, 탭 동작 정책 |
| 02-DECODING.md | 백엔드 비교, libmpv 를 권하는 이유와 대안, 크레이트 표 |
| 03-GPUI.md | workbench-gpui 현황, `PaneItem` 설계, 출력 3단계, 컨트롤 UI, 사례 |
| 04-TAURI.md | workbench-tauri 현황, 탭 모델·세션 레지스트리·IPC, `<video>` 한계, 자식 HWND, 대안, 사례 |
| 05-WINDOWS.md | R1, 소리, 자막, 빌드·DLL·번들·크기, 라이선스 |
| 06-FILTERS.md | R2 기능별 구현표, 실시간 조절, CPU vs GPU, GPL 필터 표, PotPlayer 대응표 |
| 07-COMPARISON.md | 방식별 비교, 최종 선택, 위험, 검증 체크리스트, 확신 낮은 부분 |
| research/ | 조사 원자료(sources)와 검토 기록(reviews) |

## 3. 읽는 법

### 3.1 표기

| 표기 | 뜻 |
| --- | --- |
| 확인 | 조사자가 출처를 직접 열어 본 값. 출처 URL 을 옆에 적었다 |
| 추정 | 계산이나 일반 지식에서 끌어낸 값. 실측하지 않았다 |
| 미확인 | 조사에서 확인하지 못했다. 구현 전에 확인해야 한다 |
| 설계 예시 | 이 문서가 제안하는 구조. 기존 API 인용이 아니다 |

### 3.2 규칙

- 같은 사실은 한 파일에만 자세히 쓰고 나머지는 링크한다.
- 조사 보고서끼리 어긋난 값은 `research/reviews/comparison.md` 의 판정을 따랐다. 그 판정을 적용한 곳은 본문에 근거 URL 을 달았다.
- GPL·LGPL 코드 원문과 Zed 앱 코드는 싣지 않았다. API 이름과 동작만 적었다.
- PotPlayer 의 메뉴 경로·단축키는 확인하지 못해 넣지 않았다. 기능 이름 기준으로만 대응시켰다.

### 3.3 사용자 결정 (2026-11-01)

| 결정 | 내용 |
| --- | --- |
| GPL | **허용**. 일반 libmpv 빌드와 GPL 필터를 그대로 쓴다. workbench-gpui 의 D-2 와 다르다. 자세한 내용은 [05-WINDOWS.md](05-WINDOWS.md) 5절 |
| PotPlayer 대응표 | 기능 이름 기준. 메뉴 경로·단축키 없음 |
| 목차 | `research/reviews/outline.md` 그대로 |

---

다음: [01-ARCHITECTURE.md](01-ARCHITECTURE.md)
