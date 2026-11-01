---
title: Rust GUI 프레임워크 — 설치와 선택 (Windows 11)
sources:
  - raw/web/261101-rust-gui/README.md
  - raw/web/261101-rust-gui/01-INSTALL.md
  - raw/web/261101-rust-gui/02-GPUI.md
  - raw/web/261101-rust-gui/03-EGUI.md
  - raw/web/261101-rust-gui/04-ICED.md
  - raw/web/261101-rust-gui/05-SLINT.md
  - raw/web/261101-rust-gui/06-TAURI.md
  - raw/web/261101-rust-gui/07-DIOXUS.md
  - raw/web/261101-rust-gui/08-COMPARISON.md
updated: 2026-11-01
tags: [rust, gui, windows, gpui, egui, iced, slint, tauri, dioxus]
status: draft
---

# Rust GUI 프레임워크 — 설치와 선택 (Windows 11)

Windows 11에서 Rust로 데스크톱 GUI 앱을 만들 때의 설치 절차와 프레임워크 6개(gpui·egui·iced·Slint·Tauri·Dioxus)의 비교다. 원본은 **Zed 편집기 같은 구조의 앱**(패널·탭·명령 팔레트·키맵)을 목표로 조사했다.

원본 기준: Rust 1.99.1(stable-x86_64-pc-windows-msvc), 2026-10-31의 공식 문서·crates.io. 설치 절차(1절)만 실제로 해 봤고, 프레임워크 예제는 실행 검증하지 않았다. 버전은 시작 전에 crates.io에서 다시 확인한다.

## 1. Rust 설치 (Windows 11)

| 단계 | 명령·할 일 | 이유 |
| --- | --- | --- |
| 1 | `winget install --id Microsoft.VisualStudio.2022.BuildTools -e --override "--quiet --wait --norestart --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"` | MSVC 툴체인은 MSVC 링커와 Windows SDK를 쓴다. 이 옵션은 CMake도 함께 넣는다(gpui-kit에 필요, PATH에는 안 들어감) |
| 2 | `winget install --id Rustlang.Rustup -e` | 기본 툴체인 `stable-x86_64-pc-windows-msvc` |
| 3 | `rustup component add rustfmt clippy rust-analyzer rust-src` | 포매터·린터·언어 서버·표준 라이브러리 소스 |
| 4 | 스마트 앱 컨트롤 확인 | 켜져 있으면 직접 빌드한 `.exe`(와 `build.rs`)가 실행되지 않는다 |
| 5 | 터미널·편집기 재시작 후 `cargo new hello_rust` → `cargo run` | 새 PATH를 읽게 한다 |

- **스마트 앱 컨트롤**: 빌드는 되는데 실행에서 `애플리케이션 제어 정책에서 이 파일을 차단했습니다. (os error 4551)`이 나면 이것 때문이다. 서명 없는 프로그램을 막는다. `(Get-MpComputerStatus).SmartAppControlState`가 `On`이면 막힌다. Windows 보안 → 앱 및 브라우저 컨트롤에서 끌 수 있지만, 빌드에 따라 한 번 끄면 다시 켤 수 없을 수 있으니 본인이 판단한다.
- **PATH**: rustup이 넣은 `%USERPROFILE%\.cargo\bin`은 이미 열린 터미널에 반영되지 않는다. Herdr 같은 멀티플렉서 서버가 살아 있으면 새 패널도 옛 PATH를 받을 수 있다. 그때는 그 셸에서 다시 읽는다:

  ```powershell
  $env:Path = [Environment]::GetEnvironmentVariable('Path','Machine') + ';' + [Environment]::GetEnvironmentVariable('Path','User')
  ```

## 2. UI 방식으로 나눈 6개 프레임워크

| 방식 | 뜻 | 프레임워크 |
| --- | --- | --- |
| GPU 네이티브 혼합형 | 상태는 보존하는 뷰에 두고, 매 프레임 요소 트리를 새로 만들어 GPU로 그린다 | gpui (+ gpui-kit) |
| 즉시 모드 | 매 프레임 UI 코드를 다시 실행한다. 위젯 상태를 따로 두지 않는다 | egui / eframe |
| Elm식 보존 모드 | State → view, Message → update의 한 방향 흐름 | iced |
| DSL | `.slint` 언어로 UI를 쓰고 컴파일해 Rust와 묶는다 | Slint |
| 웹뷰 | 시스템 웹뷰(WebView2)에 HTML/CSS로 그리고 로직은 Rust | Tauri 2, Dioxus |

## 3. 비교표

| | gpui + gpui-kit | egui / eframe | iced | Slint | Tauri 2 | Dioxus |
| --- | --- | --- | --- | --- | --- | --- |
| 최신 버전 (2026-10) | kit 0.8.1 (gpui-pre 0.3.10) | 0.37.0 | 0.14.0 | 1.18.1 | 2.12.5 | 0.7.10 |
| 성숙도 | pre-1.0, 잦은 breaking change | 0.x, 실사용 많음 | 스스로 experimental | 1.x 안정 | 2.x 안정 | 0.x, 0.8 알파 |
| 라이선스 | Apache-2.0 | MIT/Apache | MIT | GPLv3 / Royalty-Free / 상용 | MIT/Apache | MIT/Apache |
| Windows 렌더링 | DX11 + DirectWrite | glow / wgpu | wgpu / tiny-skia | femtovg / skia / sw | WebView2 | WebView2 |
| 추가 준비물 | CMake | 없음 | 없음 | 없음 (skia는 clang) | WebView2, (Node) | WebView2, (dx) |
| Dock·분할·탭 | 기본 (`dock`) | 외부 (`egui_dock`, `egui_tiles`) | `pane_grid` | 직접 | 웹 라이브러리 | 직접 / CSS |
| 명령 팔레트 | 기본 (`command`) | 직접 | 직접 | 직접 | 웹 라이브러리 | 직접 |
| 컨텍스트별 키맵 | 기본 (key_context) | 직접 | 직접 | 제한적 | 웹 쪽 | 직접 |
| Zed 같은 앱 적합도 | **최상** | 중 | 중 | 중하 | 중 (외형 높음) | 중 |
| 학습 난이도 | 높음 | 낮음 | 중간 | 중간 | 웹 경험에 따라 | 중간 |

적합도·난이도는 원본이 기능 설명을 근거로 한 판단이고 공식 수치가 아니다. 실행 파일 크기·메모리·FPS는 출처 있는 공통 측정이 없어 원본에도 없다.

## 4. 무엇을 고를까

```text
Zed 같은 작업 공간(Dock·탭·명령 팔레트·키맵)이 1순위인가?
├─ 예 ──────────────▶ gpui + gpui-kit  (pre-1.0이라 버전 고정 필수)
└─ 아니오
   ├─ 웹 기술로 UI    ├─ JS 프런트엔드 ──▶ Tauri 2
   │                 └─ Rust로만 ──────▶ Dioxus
   ├─ 도구·뷰어·디버그 UI를 빨리 ──────▶ egui
   ├─ 상태 흐름을 엄격하게 ───────────▶ iced
   └─ UI와 로직을 파일로 분리(디자이너·임베디드) ─▶ Slint (라이선스 확인)
```

Zed 같은 앱이 목표일 때 권장 순서: gpui-kit Hello World → gpui-kit 저장소 `examples/dock` 실행·읽기 → 액션·키맵 → 명령 팔레트 → 테마. gpui 변화가 부담스러우면 egui(+`egui_dock`)로 원형을 먼저 만든다.

## 5. 프레임워크별 요점

### gpui + gpui-kit

- gpui는 Zed 팀의 GPU 가속 UI 프레임워크로 Zed 저장소의 `crates/gpui`에 있다. gpui-kit(구 gpui-component)은 Longbridge가 만든 위젯 킷으로 60개가 넘는 컴포넌트(Dock, Tab, Table, Command, 편집기 등)를 준다.
- **버전 고정**: `cargo add gpui-kit@=0.8.1`로 정확히 고정하고 `Cargo.lock`을 커밋한다. gpui-kit 0.8.1은 `gpui-pre =0.3.10`을 고정하므로 `gpui`(0.2.2)나 git의 gpui를 **따로 넣지 않는다**. 섞으면 서로 다른 gpui 타입이 충돌한다.
- 핵심 개념: `App`(전역), `Entity<T>`(gpui가 소유하는 공유 상태), View(`Render`를 구현한 Entity), `Context<T>`(상태를 바꾼 뒤 `cx.notify()`), `Window`, Element(`div()` 등, 매 프레임 새로 만듦). 레이아웃은 Flexbox(Taffy), 스타일은 tailwind식 메서드 체인.
- 키보드 중심 구조는 `actions!`(명령 타입) + `.key_context("Editor")` + `FocusHandle`/`.track_focus()` + `.on_action()` + `cx.bind_keys([...])`로 만든다. 키 입력은 포커스된 요소에서 부모로 올라가며 컨텍스트가 맞는 바인딩을 찾는다.
- 주의: 공식 문서가 적어 예제와 소스를 읽어야 한다. Zed 저장소에는 GPL/AGPL 코드도 있으니 gpui(Apache-2.0)만 의존하고 Zed 앱 코드는 가져오지 않는다. 한글 IME·고DPI는 직접 확인한다.

### egui / eframe

- 즉시 모드라 `if ui.button("+1").clicked() { … }`처럼 그리기와 처리가 한곳에 있다. 데스크톱 앱은 eframe으로 띄운다. 추가 준비물 없이 빌드된다.
- 단점: 큰 UI는 매 프레임 레이아웃 비용이 크고, 네이티브 외형이 아니며, 전역 키맵·포커스 라우팅은 직접 만든다.

### iced

- Elm 아키텍처(State → view, Message → update). 비동기는 `Task`, 외부 이벤트는 `Subscription`. `pane_grid`로 타일링 분할.
- 단점: 스스로 experimental이고 0.14.0(2025-12) 이후 새 릴리스가 없다(2026-10 기준). 메시지 enum이 커지면 장황해진다.

### Slint

- `.slint` DSL로 UI를 쓰고(`slint!` 매크로 또는 `slint-build`), 로직은 Rust. 실시간 미리보기, 1.x 안정판, 임베디드까지.
- 단점: **MIT가 아니다**(GPLv3 / Royalty-Free / 상용). 런타임에 구조가 바뀌는 Dock·명령 팔레트에는 DSL 제약이 있다.

### Tauri 2

- 웹 프런트엔드(HTML/CSS/JS, React 등)를 시스템 웹뷰가 그리고, 로직은 Rust 백엔드를 IPC(`invoke`)로 부른다. 창·메뉴·트레이·업데이트·설치 패키지를 주는 **앱 셸**이다.
- 단점: IPC 경계에서 데이터가 직렬화되고, 웹뷰가 메모리를 쓴다. 웹 경험이 없으면 배울 것이 두 배다.

### Dioxus

- React식 컴포넌트를 Rust 함수와 `rsx!`로 쓰고 상태는 signals(`use_signal`). 같은 코드로 웹·데스크톱·모바일.
- 데스크톱 안정 경로는 웹뷰다. 네이티브 렌더러 Blitz는 실험 단계다. 0.8이 알파라 API가 곧 바뀔 수 있다.

## 6. 공통 주의점

- **버전**: gpui-kit은 `=0.8.1`과 `Cargo.lock`. egui·iced·Dioxus도 0.x라 마이너 버전에 API가 바뀐다. 인터넷 예제는 버전부터 확인한다.
- **한글 IME**: 웹뷰 기반(Tauri, Dioxus)은 브라우저 엔진 IME를 쓴다. 자체 렌더링(gpui, egui, iced, Slint)은 조합 중 커서·후보창 위치가 버전마다 다를 수 있으니 일찍 확인한다.
- **배포**: 웹뷰 앱은 대상 PC에 WebView2가 있어야 한다(Windows 11 기본). 콘솔 창은 `#![windows_subsystem = "windows"]`로 숨긴다. 서명 없는 `.exe`는 스마트 앱 컨트롤이 켜진 PC에서 실행되지 않는다.
- **기타 후보**: Floem(Lapce 팀, crates.io 0.2.0에서 멈춤), Xilem(Linebender, 실험 단계)은 원본에서 뺐다.

## 원본

원본 묶음의 입구는 [README](../../../raw/web/261101-rust-gui/README.md)이고, 프레임워크별 설치·예제 코드는 각 파일(예: [02-GPUI](../../../raw/web/261101-rust-gui/02-GPUI.md))에 있다. 공식 문서 링크는 [08-COMPARISON](../../../raw/web/261101-rust-gui/08-COMPARISON.md) 4.2절에 모여 있다.
