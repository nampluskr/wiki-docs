---
tags: [rust, gui, gpui, gpui-kit, zed, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 02 · gpui + gpui-kit (1순위)

> 이 파일을 읽는 때: Zed 같은 구조(Dock·탭·명령 팔레트·키맵)의 앱을 만들려 할 때.

입구는 [README.md](README.md)다. Rust 설치는 [01-INSTALL.md](01-INSTALL.md)에 있다.

| 항목 | 내용 |
| --- | --- |
| 무엇 | gpui = Zed 팀이 만든 GPU 가속 UI 프레임워크. gpui-kit = Longbridge가 만든 gpui 기반 위젯 킷 |
| 버전 | gpui-kit 0.8.1 (2026-10-26), 내부 gpui는 `gpui-pre =0.3.10` 스냅샷. 공식 `gpui` 크레이트는 0.2.2 (2025-10) |
| 라이선스 | Apache-2.0 (gpui, gpui-kit 모두) |
| Windows | 지원. 창은 Win32, 렌더링은 DirectX 11, 텍스트는 DirectWrite |
| 검증 | 실행 검증 안 함. 예제는 gpui-kit README·문서 발췌 기준 |

## 1. 정체와 크레이트 구조

### 1.1 gpui

- Zed 편집기의 UI 프레임워크다. 상태는 보존하는 뷰(Entity)에 두고, 매 프레임 tailwind 스타일 API로 요소 트리를 새로 만들어 GPU로 그린다. 즉시 모드와 보존 모드를 섞은 구조다. https://www.gpui.rs/
- Zed 저장소(`zed-industries/zed`)의 `crates/gpui`에 있다. 따로 떨어진 저장소가 없고, 기여도 Zed 저장소로 받는다.
- **1.0 이전이다.** README에 "버전 사이에 breaking change가 자주 있다"고 적혀 있다. https://github.com/zed-industries/zed/blob/main/crates/gpui/README.md

### 1.2 gpui-kit (구 gpui-component)

- Longbridge가 만든 gpui 위젯 킷이다. "GPUI Component is now GPUI Kit" — 저장소가 `longbridge/gpui-component`에서 `longbridge/gpui-kit`으로 바뀌었고, 문서 사이트는 https://gpui-kit.com/ 이다.
- 60개가 넘는 데스크톱 컴포넌트를 준다. 상용 앱 Longbridge Pro가 처음부터 이 킷으로 만들어졌다. https://github.com/longbridge/gpui-component

### 1.3 크레이트 구조

| 크레이트 | 버전 | 역할 |
| --- | --- | --- |
| `gpui-kit` | 0.8.1 | 앱용 통합 진입점. 아래 크레이트를 한 의존성으로 묶는다 |
| `gpui-component` | 0.8.1 | 스타일이 입혀진 위젯 (Button, Dock, Tab, Table, Command 등) |
| `gpui-base` | 0.8.1 | 위젯의 동작·상태 계층 (예: 도킹 상태) |
| `gpui-kit-assets` | 0.8.1 | 기본 아이콘·자산 |
| `gpui-pre` (+ `-platform`, `-macros`, `-web`) | =0.3.10 | Zed 소스의 특정 커밋을 crates.io로 배포한 gpui 스냅샷 |
| `gpui-fast` | 0.1.12 | Longbridge의 렌더링·레이아웃 최적화판 (선택 기능) |
| `gpui` | 0.2.2 | 공식 crates.io 배포판. 1년째 새 버전이 없다 |

출처: https://crates.io/crates/gpui-kit/0.8.1/dependencies , https://gpui-kit.com/docs/installation/

**핵심:** gpui-kit 0.8.1은 `gpui-pre =0.3.10`을 정확히 고정한다. 그래서 **`gpui`를 따로 추가하지 않는다.** `gpui = "0.2.2"`나 git의 gpui를 같이 넣으면 서로 다른 gpui 타입이 섞여 컴파일 오류가 난다.

## 2. 설치

### 2.1 준비물

[01-INSTALL.md](01-INSTALL.md)의 Rust 설치에 더해 아래가 필요하다. https://gpui-kit.com/docs/installation/

| 준비물 | 내용 |
| --- | --- |
| Rust | 1.92 이상 (현재 의존성 그래프 기준) |
| VS Build Tools | "Desktop development with C++" 워크로드, MSVC, Windows SDK |
| CMake | PATH에 있어야 한다. VS Build Tools를 `--includeRecommended`로 설치했다면 이미 들어 있다 (01-INSTALL.md 2절) |

VS Build Tools에 들어 있는 CMake를 현재 셸의 PATH에 추가하는 방법

```powershell
$env:Path += ";${env:ProgramFiles(x86)}\Microsoft Visual Studio\2022\BuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin"
cmake --version
```

- Zed 문서의 Windows 빌드 안내에는 Spectre 완화 라이브러리, Windows SDK 10.0.20348.0 이상도 있다. 이것은 **Zed 전체를 소스로 빌드할 때의 조건**이다. 작은 gpui-kit 앱의 필수 조건으로 옮기지 않는다. https://zed.dev/docs/development/windows

### 2.2 프로젝트 만들기

```powershell
cargo new my_app
cd my_app
cargo add gpui-kit@=0.8.1
```

- 버전은 `=0.8.1`처럼 **정확히 고정**한다. pre-1.0이라 0.8.x 안에서도 gpui 스냅샷이 바뀔 수 있다.
- `Cargo.lock`을 저장소에 함께 올린다.
- 첫 빌드는 의존성이 많아 오래 걸린다 (수백 개 크레이트). 두 번째부터는 빠르다.

## 3. 사용법

### 3.1 Hello World

`src/main.rs` (gpui-kit README 발췌, https://github.com/longbridge/gpui-component)

```rust
use gpui_kit::component::button::*;
use gpui_kit::component::*;
use gpui_kit::*;

pub struct HelloWorld;

impl Render for HelloWorld {
    fn render(&mut self, _: &mut Window, _: &mut Context<Self>) -> impl IntoElement {
        div()
            .v_flex()
            .gap_2()
            .size_full()
            .items_center()
            .justify_center()
            .child("Hello, World!")
            .child(
                Button::new("ok")
                    .primary()
                    .label("Let's Go!")
                    .on_click(|_, _, _| println!("Clicked!")),
            )
    }
}

fn main() {
    gpui_kit::application().run(move |cx| {
        gpui_kit::init(cx); // 컴포넌트를 쓰기 전에 반드시 호출
        gpui_kit::open_window(WindowOptions::default(), cx, |_, cx| cx.new(|_| HelloWorld))
            .expect("window");
    });
}
```

```powershell
cargo run
```

- `gpui_kit::application()`과 `gpui_kit::open_window()`는 gpui의 `Application::new()`·`cx.open_window()`를 감싼 것이다. 오버레이(팝업·알림)를 띄울 루트 구성까지 해 준다. https://docs.rs/gpui-kit/0.8.1/gpui_kit/
- gpui-kit README에는 의존성이 `"0.7"`로 적혀 있지만, crates.io 최신은 0.8.1이다. 다른 예제를 가져올 때는 버전을 확인한다.

### 3.2 핵심 개념

| 개념 | 뜻 |
| --- | --- |
| `App` | 앱 전체의 상태와 전역 서비스 (키 바인딩, 창 목록). `run(|cx| ...)`의 `cx` |
| `Entity<T>` | gpui가 소유하는 공유 상태. `Rc`와 비슷하다. `cx.new(|cx| T { ... })`로 만든다 |
| View | `Render`를 구현한 Entity. 화면 한 덩어리 (패널, 편집기, 상태 표시줄) |
| `Context<T>` | 그 Entity 안에서 쓰는 `cx`. 상태를 바꾼 뒤 `cx.notify()`로 다시 그리게 한다 |
| `Window` | 창. 포커스, 크기, 입력을 다룬다 |
| Element | `div()` 같은 저수준 요소. 매 프레임 `render()`가 새로 만든다. 큰 목록·편집기는 Element를 직접 구현한다 |

출처: https://github.com/zed-industries/zed/blob/main/crates/gpui/README.md

레이아웃은 Flexbox(Taffy)이고, 스타일은 tailwind처럼 메서드를 잇는다: `div().flex().gap_2().p_4().bg(...)`.

## 4. Zed 같은 구조 만들기

### 4.1 Dock·탭·사이드바

gpui-kit의 `dock` 모듈이 Zed의 작업 공간과 비슷한 구조를 준다. https://gpui-kit.com/ , https://github.com/longbridge/gpui-kit/blob/main/docs/ARCHITECTURE.md

| 구성 요소 | 하는 일 |
| --- | --- |
| `DockArea` | 작업 공간 전체. 가운데 영역과 왼쪽·오른쪽·아래 Dock |
| `DockItem` / `DockLayout` | 분할(split)과 탭 그룹을 나무 구조로 표현 |
| `TabGroup` / `Panel` | 탭 하나하나가 Panel. 드래그로 옮기고, 줌하고, 닫는다 |
| 레이아웃 직렬화 | 배치를 저장했다가 다음 실행에 복원 |
| `resizable` | `h_resizable`/`v_resizable`로 크기를 바꿀 수 있는 분할 |

- 도킹 **상태**는 `gpui-base`가, **외형**은 `gpui-component`가 맡는다.
- 실제 코드는 저장소의 Dock 예제가 가장 정확하다: https://github.com/longbridge/gpui-kit/blob/main/examples/dock/src/main.rs
- 사이드바는 `sidebar`, 파일 트리는 `tree` 컴포넌트를 쓴다. https://docs.rs/gpui-component/0.8.1/gpui_component/

### 4.2 액션·키맵·포커스

Zed의 키보드 중심 구조는 gpui의 **액션 + 키 컨텍스트 + 포커스**로 만든다. https://github.com/zed-industries/zed/blob/main/crates/gpui/docs/key_dispatch.md , https://gpui-kit.com/docs/keybinding/ , https://gpui-kit.com/docs/focus/

| 요소 | 하는 일 |
| --- | --- |
| `actions!(ns, [A, B])` | 명령을 타입으로 정의한다 |
| `.key_context("Editor")` | 이 요소 아래를 "Editor" 컨텍스트로 표시한다 |
| `FocusHandle` + `.track_focus()` | 이 요소가 포커스를 받을 수 있게 한다 |
| `.on_action(...)` | 포커스가 이 요소(또는 자식)에 있을 때 액션을 처리한다 |
| `cx.bind_keys([KeyBinding::new("ctrl-k", A, Some("Editor"))])` | 키 → 액션 매핑. 컨텍스트가 맞을 때만 작동한다 |

키 입력은 포커스된 요소에서 시작해 부모 쪽으로 올라가며, 컨텍스트가 맞는 바인딩을 찾는다. 그래서 같은 `ctrl-k`도 편집기와 파일 트리에서 다른 명령이 될 수 있다. 연속 키 조합(`ctrl-k ctrl-s` 같은)도 지원한다. Zed처럼 키맵을 JSON 파일(컨텍스트 → 키 → 액션)로 두고 읽어 들일 수도 있다.

뼈대 예시 (위 출처의 API 이름으로 구성한 스케치. 실행 검증 안 함, 버전에 따라 시그니처가 다를 수 있다)

```rust
use gpui_kit::*;

actions!(demo, [Increment]);

struct CounterView {
    count: usize,
    focus_handle: FocusHandle,
}

impl Render for CounterView {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .key_context("Counter")
            .track_focus(&self.focus_handle)
            .on_action(cx.listener(|this, _: &Increment, _, cx| {
                this.count += 1;
                cx.notify();
            }))
            .size_full()
            .child(format!("count = {}  (ctrl-k)", self.count))
    }
}

fn main() {
    gpui_kit::application().run(|cx| {
        gpui_kit::init(cx);
        cx.bind_keys([KeyBinding::new("ctrl-k", Increment, Some("Counter"))]);
        gpui_kit::open_window(WindowOptions::default(), cx, |window, cx| {
            cx.new(|cx| {
                let focus_handle = cx.focus_handle();
                window.focus(&focus_handle);
                CounterView { count: 0, focus_handle }
            })
        })
        .expect("window");
    });
}
```

### 4.3 명령 팔레트

- `gpui_component::command` 모듈이 명령 팔레트를 준다: 검색, 그룹, 키 힌트 표시, 키보드 탐색. https://docs.rs/gpui-component/0.8.1/gpui_component/command/index.html
- 팔레트의 항목은 4.2의 액션과 연결한다. 액션 목록과 키 바인딩을 같이 보여 주면 Zed의 `ctrl-shift-p`와 같은 구조가 된다.

### 4.4 테마와 편집기 컴포넌트

| 기능 | 내용 |
| --- | --- |
| 테마 | `Theme`/`ThemeColor` 시맨틱 토큰. 라이트·다크·사용자 테마, 실행 중 전환 |
| 편집기 | Rope 기반 Input/Editor. Tree-sitter 구문 강조, LSP 진단·자동 완성·hover |
| 목록·표 | `VirtualList`(보이는 줄만 그림), `DataTable` |
| 테스트 | `gpui_kit::test`로 창 없이(headless) UI 테스트 |

출처: https://gpui-kit.com/ , https://docs.rs/gpui-component/0.8.1/gpui_component/

## 5. 응용 사례

| 앱 | 내용 |
| --- | --- |
| Zed | gpui를 만든 바로 그 편집기. Windows 정식 지원 |
| Longbridge Pro | 증권 거래 데스크톱 앱. gpui-component(현 gpui-kit)를 처음부터 사용 |

## 6. 장단점과 주의점

| 장점 | 단점 |
| --- | --- |
| Zed 같은 앱에 필요한 Dock·탭·명령 팔레트·키맵·편집기가 위젯으로 준비돼 있다 | pre-1.0. 버전이 바뀔 때마다 API가 깨질 수 있다 |
| GPU 직접 렌더링, 가상 목록 | 공식 문서가 적다. 예제 코드와 소스를 읽어야 한다 |
| Zed라는 큰 실사용 사례 | 공급 경로가 나뉘어 있다 (`gpui` 0.2.2 / `gpui-pre` / `gpui-fast`) |
| Apache-2.0 | 학습 난이도가 높다 (Entity·Context·수명) |

주의점
- **버전 고정**: `gpui-kit@=0.8.1`과 `Cargo.lock`. 다른 gpui 크레이트를 섞지 않는다.
- **gpui-fast**는 Longbridge의 최적화 포크다. 업스트림과 갈라질 수 있으니 필요할 때만 켠다.
- **라이선스**: Zed 저장소에는 GPL/AGPL 크레이트도 있다. gpui(Apache-2.0)만 쓰고, Zed 앱 코드를 그대로 가져오지 않는다. https://github.com/zed-industries/zed
- **Windows 품질**: 한글 IME 조합·후보창 위치, 고DPI, 스크린 리더는 버전에 따라 다를 수 있다. 직접 확인한다. https://gpui-kit.com/docs/accessibility/
- 비동기 작업(`cx.spawn`), 큰 목록, 패키징(`#![windows_subsystem = "windows"]`로 콘솔 창 숨기기, 아이콘)은 gpui-kit 문서와 예제 폴더에서 이어서 본다.

---

이전: [01-INSTALL.md](01-INSTALL.md) · 다음: [03-EGUI.md](03-EGUI.md) · 입구: [README.md](README.md)
