---
tags: [rust, gui, dioxus, webview, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 07 · Dioxus

> 이 파일을 읽는 때: React와 비슷한 방식으로, Rust만으로 웹·데스크톱 UI를 함께 만들고 싶을 때.

입구는 [README.md](README.md)다.

| 항목 | 내용 |
| --- | --- |
| 방식 | React식 컴포넌트 + `rsx!` + signals. 데스크톱은 웹뷰 |
| 버전 | 안정판 0.7.10, 프리릴리스 0.8.0-alpha.3 (2026-10-16) |
| 라이선스 | MIT OR Apache-2.0 |
| Windows | 지원. WebView2 필요 (Windows 11 기본 포함) |
| 검증 | 실행 검증 안 함. 예제는 docs.rs 0.7.10 기준 |

## 1. 특징

- 컴포넌트를 Rust 함수로 쓰고, `rsx!` 매크로로 HTML 같은 구조를 적는다. 상태는 `use_signal` 같은 signals로 둔다. https://docs.rs/dioxus/0.7.10/dioxus/
- 같은 코드로 웹, 데스크톱, 모바일을 겨냥한다.
- 데스크톱의 안정 경로는 **WebView**다. 웹뷰 없이 GPU로 그리는 네이티브 렌더러(Blitz)는 실험 단계이고, CSS를 다 지원하지 않는다. https://github.com/DioxusLabs/dioxus
- Tauri와 달리 프런트엔드도 Rust로 쓴다. JS 빌드 도구가 필요 없다.

## 2. 설치

| 준비물 | 내용 |
| --- | --- |
| WebView2 | Windows 11에는 기본으로 있다 |
| dioxus-cli (`dx`) | 개발 서버·핫 리로드·번들. `cargo run`만으로도 데스크톱 앱은 실행된다 |

```powershell
cargo new dioxus_app
cd dioxus_app
cargo add dioxus@0.7 --features desktop

# 선택: CLI
cargo install dioxus-cli --version 0.7.10 --locked
dx doctor        # 환경 진단
```

출처: https://dioxuslabs.com/learn/0.7/getting_started/

## 3. 사용법

`src/main.rs`

```rust
use dioxus::prelude::*;

fn main() {
    dioxus::launch(app);
}

fn app() -> Element {
    let mut count = use_signal(|| 0);
    rsx! {
        h1 { "Hello Dioxus" }
        button { onclick: move |_| count += 1, "+1" }
        p { "value = {count}" }
    }
}
```

```powershell
cargo run
# 또는 CLI: dx serve --desktop
```

- 스타일은 CSS로 준다. `rsx!` 안에 `style: "..."`를 쓰거나 CSS 파일을 자산으로 넣는다.

## 4. 응용 사례

공식 큐레이션 목록(awesome-dioxus)에 Ebou(Mastodon 클라이언트), Uplink(메신저), Kopuz 등이 올라 있다. 유지보수 상태는 각 저장소에서 확인한다. https://github.com/DioxusLabs/awesome-dioxus

## 5. 장단점

| 장점 | 단점 |
| --- | --- |
| React 경험이 있으면 익숙하다 | 0.x. 0.8이 알파 단계라 곧 API가 바뀔 수 있다 |
| 프런트엔드까지 Rust로 쓴다 | 데스크톱 기본 렌더러가 웹뷰다 |
| 웹·데스크톱·모바일을 한 코드로 | 네이티브 렌더러(Blitz)는 실험 단계 |
| CSS로 스타일을 자유롭게 | 웹뷰 렌더러와 네이티브 렌더러의 동작이 다를 수 있다 |

Zed 같은 앱에는 **중간** 정도로 맞는다.

---

이전: [06-TAURI.md](06-TAURI.md) · 다음: [08-COMPARISON.md](08-COMPARISON.md) · 입구: [README.md](README.md)
