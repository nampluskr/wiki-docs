---
tags: [rust, gui, egui, eframe, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 03 · egui / eframe

> 이 파일을 읽는 때: 도구·뷰어·디버그 UI를 빨리 만들고 싶을 때.

입구는 [README.md](README.md)다.

| 항목 | 내용 |
| --- | --- |
| 방식 | 즉시 모드 |
| 버전 | egui·eframe 0.37.0 (2026-10-15) |
| 라이선스 | MIT OR Apache-2.0 |
| Windows | 지원. 렌더러 glow(OpenGL) 또는 wgpu, 창은 winit |
| 검증 | 실행 검증 안 함. 예제는 docs.rs 0.37 API 기준 |

## 1. 특징

- **즉시 모드**: 매 프레임 UI 코드를 다시 실행한다. `if ui.button("+1").clicked() { ... }`처럼 그리는 코드와 처리 코드가 한곳에 있다. 위젯 객체나 콜백을 따로 관리하지 않는다. https://github.com/emilk/egui
- `egui`는 UI 라이브러리, `eframe`은 egui를 창으로 띄우는 공식 앱 프레임워크다. 데스크톱 앱은 eframe을 쓴다.
- 게임 엔진(bevy_egui)이나 다른 렌더러 안에 쉽게 끼워 넣을 수 있다.
- Dock·탭은 외부 크레이트로 보탠다: `egui_dock`, `egui_tiles`. https://crates.io/crates/egui_tiles

## 2. 설치

추가 준비물은 없다. [01-INSTALL.md](01-INSTALL.md)의 Rust 설치만으로 된다.

```powershell
cargo new egui_app
cd egui_app
cargo add eframe@0.37
```

시작 템플릿: https://github.com/emilk/eframe_template

## 3. 사용법

`src/main.rs`

```rust
use eframe::egui;

#[derive(Default)]
struct Counter {
    value: i32,
}

impl eframe::App for Counter {
    fn ui(&mut self, ui: &mut egui::Ui, _frame: &mut eframe::Frame) {
        ui.heading("Hello egui");
        if ui.button("+1").clicked() {
            self.value += 1;
        }
        ui.label(format!("value = {}", self.value));
    }
}

fn main() -> eframe::Result {
    eframe::run_native(
        "Hello egui",
        eframe::NativeOptions::default(),
        Box::new(|_cc| Ok(Box::new(Counter::default()))),
    )
}
```

```powershell
cargo run
```

- 0.37의 `eframe::App`은 `ui(&mut self, ui, frame)`를 구현한다. 인터넷의 오래된 예제는 `update(&mut self, ctx, frame)` 안에서 `egui::CentralPanel::default().show(ctx, ...)`를 쓴다. 복사할 때 버전을 확인한다. https://docs.rs/eframe/0.37.0/eframe/
- 패널 배치는 `egui::SidePanel`, `TopBottomPanel`, `CentralPanel`로 한다.

## 4. 응용 사례

| 앱 | 내용 |
| --- | --- |
| Rerun Viewer | 로보틱스·컴퓨터 비전 데이터 시각화 도구. egui 개발자가 만든 회사의 제품 |
| bevy_egui | Bevy 게임 엔진의 에디터·디버그 UI |

출처: https://github.com/emilk/egui , https://rerun.io

## 5. 장단점

| 장점 | 단점 |
| --- | --- |
| 코드가 짧고 배우기 쉽다 | 매 프레임 레이아웃을 다시 해서 큰 UI는 CPU 비용이 크다 |
| 추가 준비물 없이 빌드된다 | 네이티브 외형이 아니다 |
| 다른 렌더러·엔진에 끼워 넣기 쉽다 | 위젯 ID를 직접 관리해야 할 때가 있다. 첫 프레임에 깜빡일 수 있다 |
| 위젯이 많다 | 0.x라 API가 바뀐다. 전역 키맵·포커스 라우팅은 직접 만든다 |

출처(단점): egui README "Why immediate mode" 절, https://github.com/emilk/egui

Zed 같은 앱에는 **중간** 정도로 맞는다. Dock은 `egui_dock`/`egui_tiles`로 되지만, 컨텍스트별 키맵과 명령 팔레트는 직접 짜야 한다.

---

이전: [02-GPUI.md](02-GPUI.md) · 다음: [04-ICED.md](04-ICED.md) · 입구: [README.md](README.md)
