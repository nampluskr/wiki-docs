---
tags: [rust, gui, slint, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 05 · Slint

> 이 파일을 읽는 때: UI를 전용 DSL 파일로 나눠 디자인과 로직을 분리하고 싶을 때.

입구는 [README.md](README.md)다.

| 항목 | 내용 |
| --- | --- |
| 방식 | 선언형 DSL(`.slint`)을 컴파일하는 보존 모드 |
| 버전 | 1.18.1 (2026-09-21) |
| 라이선스 | GPLv3 / Royalty-Free / 상용 중 선택 |
| Windows | 지원. 렌더러 femtovg(OpenGL), skia, software |
| 검증 | 실행 검증 안 함. 예제는 docs.rs 1.18 기준 |

## 1. 특징

- UI는 `.slint` 언어로 쓴다. 속성(property), 콜백, 바인딩을 선언하면 Rust 코드가 생성된다. 로직은 Rust에 둔다. https://docs.rs/slint/1.18.1/slint/
- 두 가지로 쓴다.
  - `slint::slint! { ... }` 매크로: Rust 파일 안에 UI를 쓴다. 시작용.
  - 별도 `.slint` 파일 + `slint-build`: `build.rs`에서 컴파일한다. 큰 앱용.
- VS Code·Zed 등에서 쓸 수 있는 언어 서버와 실시간 미리보기가 있다.
- 1.x 안정판이고, 임베디드(마이크로컨트롤러)까지 지원한다.

## 2. 설치

추가 준비물은 없다. skia 렌더러를 고르면 clang/LLVM이 더 필요할 수 있다. https://docs.rs/slint-build

```powershell
cargo new slint_app
cd slint_app
cargo add slint@1.18
```

별도 `.slint` 파일을 쓸 때

```powershell
cargo add --build slint-build@1.18
```

## 3. 사용법

`src/main.rs` (매크로 방식)

```rust
slint::slint! {
    import { Button, VerticalBox } from "std-widgets.slint";

    export component AppWindow inherits Window {
        in-out property <int> counter: 0;
        title: "Hello Slint";
        VerticalBox {
            Text { text: "Hello Slint"; }
            Text { text: "value = \{root.counter}"; }
            Button {
                text: "+1";
                clicked => { root.counter += 1; }
            }
        }
    }
}

fn main() -> Result<(), slint::PlatformError> {
    AppWindow::new()?.run()
}
```

```powershell
cargo run
```

- Rust 쪽에서 속성은 `app.get_counter()`/`set_counter()`, 콜백은 `app.on_<이름>(...)`으로 다룬다.

## 4. 응용 사례

| 앱 | 내용 |
| --- | --- |
| LibrePCB | 오픈소스 PCB 설계 도구 |
| WesAudio | 오디오 플러그인 UI |
| SK Signet | 전기차 충전기 HMI (임베디드) |

출처: https://slint.dev/showcase , https://madewithslint.com

## 5. 장단점

| 장점 | 단점 |
| --- | --- |
| UI와 로직이 파일로 나뉜다. 디자이너와 나눠 일하기 쉽다 | DSL을 따로 배워야 한다 |
| 1.x 안정판, 임베디드까지 | 라이선스가 GPLv3 / Royalty-Free / 상용으로 나뉜다. MIT가 아니다 |
| 실시간 미리보기 도구 | 동적 Dock·명령 팔레트처럼 런타임에 구조가 바뀌는 UI에는 DSL 제약이 있다 |
| 렌더러 선택 폭 | 응용 사례가 임베디드·산업용 위주 |

라이선스: 독점 데스크톱 앱은 Royalty-Free 조건(저작자 표시 등)이나 상용 라이선스를 확인한다. https://slint.dev/pricing

Zed 같은 앱에는 **중하** 정도로 맞는다.

---

이전: [04-ICED.md](04-ICED.md) · 다음: [06-TAURI.md](06-TAURI.md) · 입구: [README.md](README.md)
