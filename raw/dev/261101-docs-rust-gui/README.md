---
tags: [rust, gui, windows, gpui, guide]
created: 2026-10-31
updated: 2026-10-31
---

# Rust GUI 앱 개발 가이드 (Windows 11)

> 이 파일을 읽는 때: 처음, 그리고 무엇을 읽을지 모를 때. 이 문서 묶음의 입구다.

Windows 11에서 Rust로 데스크톱 GUI 앱을 만드는 방법을 정리한다. 목표는 **Zed 편집기와 비슷한 구조의 앱**이다. 그래서 Zed의 UI 프레임워크 gpui와, 그 위의 위젯 킷 gpui-kit을 가장 깊게 다룬다. 나머지 다섯 프레임워크는 같은 구조로 고르게 소개하고 마지막에 비교한다.

| 항목 | 내용 |
| --- | --- |
| 대상 | Rust로 데스크톱 GUI를 처음 만들어 보는 사람 |
| 결과 | Rust 설치, 프레임워크별 최소 앱 실행, Zed 같은 앱에 맞는 프레임워크 선택 |
| 기준 | Windows 11, Rust 1.99.1 (stable-x86_64-pc-windows-msvc), 편집기 Zed, 2026-10 기준 문서와 crates.io |
| 검증 | 01-INSTALL.md의 설치 절차만 이 노트북에서 실제로 해 봤다. 02~07의 예제는 공식 문서·저장소·crates.io 기준이고, 실행 검증은 하지 않았다 |

## 1. 읽기 지도

| 지금 할 일 | 읽을 파일 |
| --- | --- |
| Rust가 아직 없다 | [01-INSTALL.md](01-INSTALL.md) |
| Zed 같은 앱을 만들고 싶다 (1순위) | [02-GPUI.md](02-GPUI.md) |
| 도구·뷰어를 빨리 만들고 싶다 | [03-EGUI.md](03-EGUI.md) |
| 상태 흐름이 분명한 구조를 원한다 | [04-ICED.md](04-ICED.md) |
| UI를 디자이너용 DSL로 나누고 싶다 | [05-SLINT.md](05-SLINT.md) |
| 웹 기술(HTML/CSS/JS)로 UI를 만들고 싶다 | [06-TAURI.md](06-TAURI.md), [07-DIOXUS.md](07-DIOXUS.md) |
| 무엇을 고를지 정한다 | [08-COMPARISON.md](08-COMPARISON.md) |

| 파일 | 담은 것 |
| --- | --- |
| README.md | 읽기 지도, Zed 같은 앱의 요건, UI 방식 분류, 고르는 방법 |
| 01-INSTALL.md | VS Build Tools, rustup, 스마트 앱 컨트롤, Zed 설정 |
| 02-GPUI.md | gpui + gpui-kit 심층: 크레이트 구조, 설치, 개념, Dock·키맵·명령 팔레트·테마 |
| 03-EGUI.md ~ 07-DIOXUS.md | 프레임워크별: 특징 → 설치 → 사용법 → 응용 사례 → 장단점 |
| 08-COMPARISON.md | 종합 비교표, 무엇을 고를까, 주의점, 기타 후보, 출처 |

## 2. 개요

### 2.1 목표: Zed 같은 앱

Zed 같은 앱이란 아래 요건을 갖춘 데스크톱 앱을 말한다.

| 요건 | 내용 |
| --- | --- |
| 작업 공간 | 여러 패널, 탭, 사이드바. 패널을 나누고 끌어서 옮기고 배치를 저장한다 |
| 명령 | 명령 팔레트로 모든 기능을 검색해 실행한다 |
| 키보드 | 키 바인딩 중심. 포커스가 있는 패널에 따라 같은 키가 다른 명령이 된다 |
| 렌더링 | GPU로 직접 그려서 큰 목록·텍스트도 부드럽게 움직인다 |
| 테마 | 라이트·다크와 사용자 테마 |

프레임워크마다 이 요건을 **얼마나 기본으로 주는지**가 다르다. 어떤 것은 위젯으로 바로 주고, 어떤 것은 직접 만들어야 한다.

### 2.2 UI 방식 분류

| 방식 | 뜻 | 프레임워크 |
| --- | --- | --- |
| GPU 네이티브 (혼합형) | 상태는 보존하는 뷰에 두고, 매 프레임 요소 트리를 새로 만들어 GPU로 그린다 | gpui (+ gpui-kit) |
| 즉시 모드 | 매 프레임 UI 코드를 다시 실행해 화면을 그린다. 위젯 상태를 따로 두지 않는다 | egui / eframe |
| Elm식 보존 모드 | 상태(State) → 화면(view), 메시지(Message) → 상태 갱신(update)의 한 방향 흐름 | iced |
| DSL | `.slint` 전용 언어로 UI를 쓰고, 컴파일해서 Rust 코드와 묶는다 | Slint |
| 웹뷰 | 시스템 웹뷰(Windows는 WebView2)에 HTML/CSS로 그리고, 로직은 Rust | Tauri 2, Dioxus |

### 2.3 고르는 방법

```
Zed 같은 작업 공간(Dock·탭·명령 팔레트·키맵)이 1순위인가?
├─ 예 ──────────────▶ gpui + gpui-kit (02-GPUI.md)
│                     pre-1.0이라 버전 고정이 필수
└─ 아니오
   ├─ 웹 기술로 UI를 짜고 싶다
   │  ├─ JS 프런트엔드를 쓴다 ──▶ Tauri 2 (06-TAURI.md)
   │  └─ Rust로만 쓴다 ─────────▶ Dioxus (07-DIOXUS.md)
   ├─ 도구·뷰어·디버그 UI를 빨리 ──▶ egui (03-EGUI.md)
   ├─ 상태 흐름을 엄격하게 ────────▶ iced (04-ICED.md)
   └─ UI와 로직을 파일로 분리 ─────▶ Slint (05-SLINT.md)
```

자세한 비교는 [08-COMPARISON.md](08-COMPARISON.md)에 있다.

---

다음: [01-INSTALL.md](01-INSTALL.md)
