---
tags: [rust, gui, iced, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 04 · iced

> 이 파일을 읽는 때: 상태 흐름이 분명한(Elm식) 구조로 앱을 만들고 싶을 때.

입구는 [README.md](README.md)다.

| 항목 | 내용 |
| --- | --- |
| 방식 | Elm식 보존 모드 (State → view, Message → update) |
| 버전 | 0.14.0 (2025-12-07) |
| 라이선스 | MIT |
| Windows | 지원. 렌더러 wgpu(DX12/Vulkan) 또는 tiny-skia(소프트웨어) |
| 검증 | 실행 검증 안 함. 예제는 iced README 기준 |

## 1. 특징

- **Elm 아키텍처**: 상태(State)가 화면(`view`)을 만들고, 사용자 입력은 메시지(`Message`)가 되어 `update`에서 상태를 바꾼다. 흐름이 한 방향이라 상태 변화를 따라가기 쉽다. https://docs.rs/iced/0.14.0/iced/
- 비동기 작업은 `Task`, 외부 이벤트(타이머, 키 입력)는 `Subscription`으로 다룬다.
- `pane_grid` 위젯으로 타일링 분할 화면을 만든다. https://docs.rs/iced/0.14.0/iced/widget/pane_grid/
- 스스로 "experimental"이라고 밝힌다. https://github.com/iced-rs/iced

## 2. 설치

추가 준비물은 없다.

```powershell
cargo new iced_app
cd iced_app
cargo add iced@0.14
```

## 3. 사용법

`src/main.rs`

```rust
use iced::widget::{button, column, text};
use iced::Element;

#[derive(Default)]
struct Counter {
    value: i64,
}

#[derive(Debug, Clone)]
enum Message {
    Increment,
}

impl Counter {
    fn update(&mut self, message: Message) {
        match message {
            Message::Increment => self.value += 1,
        }
    }

    fn view(&self) -> Element<'_, Message> {
        column![
            text("Hello iced"),
            button("+1").on_press(Message::Increment),
            text(self.value).size(40),
        ]
        .spacing(10)
        .into()
    }
}

fn main() -> iced::Result {
    iced::run(Counter::update, Counter::view)
}
```

```powershell
cargo run
```

- `iced::run(update, view)`는 상태 타입의 `Default`로 시작한다. 창 제목·크기를 정하려면 `iced::application(...)` 빌더를 쓴다.
- 0.13 이전 예제는 `Sandbox`/`Application` 트레이트를 쓴다. 0.14와 다르다.

## 4. 응용 사례

| 앱 | 내용 |
| --- | --- |
| COSMIC 데스크톱 | System76의 Linux 데스크톱 환경. 앱 툴킷 libcosmic이 iced 기반 |
| Halloy | IRC 클라이언트 |
| Sniffnet | 네트워크 트래픽 모니터 |

출처: https://github.com/iced-rs/iced , https://github.com/iced-rs/awesome-iced

## 5. 장단점

| 장점 | 단점 |
| --- | --- |
| 상태 흐름이 명확하고 테스트하기 쉽다 | 스스로 experimental. 버전마다 API가 크게 바뀐다 |
| 순수 Rust, 타입 안전 | 메시지 enum이 커지면 코드가 장황해진다 |
| `pane_grid`로 분할 화면 기본 제공 | 다중 포커스, 복합 Dock·탭 바, 명령 팔레트는 직접 만든다 |
| 소프트웨어 렌더러 대안 | 0.14.0(2025-12) 이후 새 릴리스가 없다 (2026-10 기준) |

Zed 같은 앱에는 **중간** 정도로 맞는다. 분할은 `pane_grid`로 되지만 작업 공간 전체 설계는 직접 해야 한다.

---

이전: [03-EGUI.md](03-EGUI.md) · 다음: [05-SLINT.md](05-SLINT.md) · 입구: [README.md](README.md)
