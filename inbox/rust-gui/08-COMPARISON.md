---
tags: [rust, gui, comparison, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 08 · 종합 비교와 선택

> 이 파일을 읽는 때: 프레임워크를 고를 때, 시작하기 전에 주의점을 확인할 때.

입구는 [README.md](README.md)다.

## 1. 종합 비교표

| | gpui + gpui-kit | egui / eframe | iced | Slint | Tauri 2 | Dioxus |
| --- | --- | --- | --- | --- | --- | --- |
| 문서 | [02](02-GPUI.md) | [03](03-EGUI.md) | [04](04-ICED.md) | [05](05-SLINT.md) | [06](06-TAURI.md) | [07](07-DIOXUS.md) |
| UI 방식 | GPU 네이티브 혼합형 | 즉시 모드 | Elm식 보존 | DSL 보존 | 웹뷰 + 웹 프런트 | React식 + 웹뷰 |
| 최신 버전 | kit 0.8.1 (gpui-pre 0.3.10) | 0.37.0 | 0.14.0 | 1.18.1 | 2.12.5 | 0.7.10 |
| 성숙도 | pre-1.0, 잦은 breaking change | 0.x, 실사용 많음 | 스스로 experimental | 1.x 안정 | 2.x 안정 | 0.x, 0.8 알파 |
| 라이선스 | Apache-2.0 | MIT/Apache | MIT | GPLv3/Royalty-Free/상용 | MIT/Apache | MIT/Apache |
| Windows 렌더링 | DX11 + DirectWrite | glow/wgpu | wgpu/tiny-skia | femtovg/skia/sw | WebView2 | WebView2 |
| 추가 준비물 | CMake | 없음 | 없음 | 없음 (skia는 clang) | WebView2, (Node) | WebView2, (dx) |
| ★ Zed 같은 앱 적합도 | **최상** | 중 | 중 | 중하 | 중 (외형 높음) | 중 |
| ★ 학습 난이도 | 높음 | 낮음 | 중간 | 중간 | 웹 경험에 따라 | 중간 |

- ★ 열은 각 문서의 기능 설명을 근거로 한 판단이다. 공식 수치가 아니다.
- 실행 파일 크기·메모리·FPS는 출처가 있는 공통 측정이 없어 싣지 않았다. 필요하면 같은 화면을 만들어 직접 잰다.

### Zed 같은 앱의 요건별

| 요건 | gpui-kit | egui | iced | Slint | Tauri | Dioxus |
| --- | --- | --- | --- | --- | --- | --- |
| Dock·분할·탭 | 기본 (`dock`) | 외부 크레이트 | `pane_grid` | 직접 | 웹 라이브러리 | 직접/웹 CSS |
| 명령 팔레트 | 기본 (`command`) | 직접 | 직접 | 직접 | 웹 라이브러리 | 직접 |
| 컨텍스트별 키맵 | 기본 (key_context) | 직접 | 직접 | 제한적 | 웹 쪽에서 | 직접 |
| 코드 편집기 | 기본 (Tree-sitter, LSP) | 외부 | 외부 | 직접 | Monaco 등 | 직접 |
| GPU 직접 렌더링 | 예 | 예 | 예 | 예 | 아니오 | 실험 (Blitz) |

## 2. 무엇을 고를까

| 상황 | 추천 |
| --- | --- |
| Zed 같은 작업 공간이 1순위이고, API 변경을 감수한다 | **gpui + gpui-kit** |
| 도구·뷰어·디버그 UI를 짧은 코드로 빨리 | egui |
| 상태 흐름을 엄격히 관리하고 싶다 | iced |
| 디자이너와 UI를 나눠 작업, 또는 임베디드까지 | Slint (라이선스 확인) |
| 웹 프런트엔드 경험이 있고 웹 라이브러리를 쓰고 싶다 | Tauri 2 |
| React식으로, Rust만으로 웹과 데스크톱을 함께 | Dioxus |

권장 순서 (Zed 같은 앱이 목표일 때)
1. [02-GPUI.md](02-GPUI.md)의 Hello World로 시작한다
2. gpui-kit 저장소의 `examples/dock`을 실행해 보고 구조를 읽는다
3. 액션·키맵 → 명령 팔레트 → 테마 순으로 기능을 붙인다
4. gpui가 너무 빨리 바뀌어 부담스러우면 egui(+`egui_dock`)로 원형을 먼저 만든다

## 3. 주의점

### 3.1 버전 고정
- gpui-kit은 `=0.8.1`로, `Cargo.lock`까지 저장소에 올린다. 다른 gpui 크레이트(`gpui` 0.2.2, git gpui)를 섞지 않는다.
- egui·iced·Dioxus도 0.x다. 마이너 버전 업에 API가 바뀐다. 인터넷 예제는 버전부터 확인한다.
- 이 문서는 안정판 기준이다. Dioxus 0.8 알파, Tauri 3 알파는 다루지 않았다.

### 3.2 한글 IME
- 웹뷰 기반(Tauri, Dioxus)은 브라우저 엔진의 IME 처리를 그대로 쓴다.
- 자체 텍스트 렌더링(gpui, egui, iced, Slint)은 한글 조합 중 커서 위치, 후보창 위치가 버전에 따라 다를 수 있다. 입력이 중요한 앱이면 일찍 확인한다.

### 3.3 라이선스
- Slint는 MIT가 아니다. GPLv3 / Royalty-Free / 상용 중 고른다. https://slint.dev/pricing
- Zed 저장소에는 GPL/AGPL 코드가 섞여 있다. gpui(Apache-2.0)만 의존하고, Zed 앱 코드를 복사하지 않는다.

### 3.4 배포
- 웹뷰 기반 앱은 배포 대상 PC에 WebView2 런타임이 있어야 한다. Windows 11에는 기본으로 있지만 Windows 10 일부에는 없을 수 있다.
- 콘솔 창을 숨기려면 `main.rs` 맨 위에 `#![windows_subsystem = "windows"]`를 넣는다.
- 직접 빌드한 `.exe`는 서명이 없다. 스마트 앱 컨트롤이 켜진 PC에서는 실행되지 않는다 ([01-INSTALL.md](01-INSTALL.md) 5절). 배포하려면 코드 서명을 검토한다.

## 4. 부록

### 4.1 기타 후보

| 후보 | 내용 | 이 문서에서 뺀 이유 |
| --- | --- | --- |
| Floem | Lapce 편집기 팀의 반응형 GPU UI. 편집기에서 나와 구조가 Zed와 비슷하다 | crates.io 최신이 0.2.0(2024-11)에서 멈춰 있다 https://crates.io/crates/floem |
| Xilem | Linebender(Vello 렌더러) 프로젝트의 반응형 UI | 아직 실험 단계 https://github.com/linebender/xilem |

### 4.2 출처

| 대상 | 공식 문서 · 저장소 |
| --- | --- |
| gpui | https://www.gpui.rs/ · https://github.com/zed-industries/zed/tree/main/crates/gpui |
| gpui-kit | https://gpui-kit.com/ · https://github.com/longbridge/gpui-kit · https://docs.rs/gpui-component/0.8.1/gpui_component/ |
| egui | https://github.com/emilk/egui · https://docs.rs/eframe/0.37.0/eframe/ |
| iced | https://iced.rs · https://docs.rs/iced/0.14.0/iced/ |
| Slint | https://slint.dev · https://docs.rs/slint/1.18.1/slint/ |
| Tauri 2 | https://v2.tauri.app · https://docs.rs/tauri/2.12.5/tauri/ |
| Dioxus | https://dioxuslabs.com/learn/0.7/ · https://docs.rs/dioxus/0.7.10/dioxus/ |
| Rust 설치 | https://rust-lang.org/tools/install/ · https://v2.tauri.app/start/prerequisites/ |

버전·날짜는 조사 시점(2026-10-31)의 crates.io 기준이다. 시작하기 전에 각 crates.io 페이지에서 최신 버전을 다시 확인한다.

---

이전: [07-DIOXUS.md](07-DIOXUS.md) · 입구: [README.md](README.md)
