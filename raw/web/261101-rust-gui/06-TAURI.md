---
tags: [rust, gui, tauri, webview, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 06 · Tauri 2

> 이 파일을 읽는 때: HTML/CSS/JS로 UI를 만들고 로직은 Rust로 쓰고 싶을 때.

입구는 [README.md](README.md)다.

| 항목 | 내용 |
| --- | --- |
| 방식 | 웹뷰 (Windows는 WebView2) + Rust 백엔드 |
| 버전 | 2.12.5 (2026-10-21) |
| 라이선스 | Apache-2.0 OR MIT |
| Windows | 지원. WebView2 필요 (Windows 11 기본 포함) |
| 검증 | 실행 검증 안 함. 절차는 Tauri 공식 문서 기준 |

## 1. 특징

- UI는 웹 프런트엔드(HTML/CSS/JS, 또는 React·Vue·Svelte 등)로 만들고, 시스템 웹뷰가 그린다. 로직은 Rust 백엔드에 두고 IPC(`invoke`)로 부른다. https://docs.rs/tauri/2.12.5/tauri/
- 위젯 렌더러라기보다 **앱 셸**이다. 창, 메뉴, 트레이, 파일 접근, 업데이트, 설치 패키지를 준다.
- 브라우저 엔진을 내장하지 않고 시스템 웹뷰를 쓰므로, Electron보다 설치 파일이 작다.
- 플러그인으로 기능을 보탠다 (파일 시스템, 대화 상자, 셸 등).

## 2. 설치

| 준비물 | 내용 |
| --- | --- |
| WebView2 | Windows 11에는 기본으로 있다 |
| Node.js LTS | JS 프런트엔드 프레임워크를 쓸 때만 필요 |
| VBSCRIPT 기능 | MSI 설치 패키지를 만들 때 Windows 선택적 기능에서 켠다 |

출처: https://v2.tauri.app/start/prerequisites/

```powershell
cargo install create-tauri-app --locked
cargo create-tauri-app
```

- 질문에 답하면 프로젝트가 만들어진다. 프런트엔드로 "Vanilla"(HTML/JS만)를 고르면 Node 없이 시작할 수 있다.
- `cargo add tauri`만으로는 앱이 완성되지 않는다. `tauri.conf.json`, `build.rs`, 아이콘이 함께 필요하므로 템플릿으로 시작한다. https://v2.tauri.app/start/create-project/

## 3. 사용법

생성된 `src-tauri/src/main.rs`(또는 `lib.rs`)의 핵심

```rust
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {name}!")
}

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![greet])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

프런트엔드에서 Rust 함수 부르기

```js
// npm 템플릿 (React·Vue 등)
import { invoke } from "@tauri-apps/api/core";
const message = await invoke("greet", { name: "Windows" });

// Vanilla 템플릿: tauri.conf.json 의 "withGlobalTauri": true 일 때
// const { invoke } = window.__TAURI__.core;
```

실행

```powershell
cargo tauri dev     # tauri-cli 를 설치한 경우 (cargo install tauri-cli --locked)
# 또는 템플릿이 npm 을 쓰면: npm run tauri dev
```

## 4. 응용 사례

| 앱 | 내용 |
| --- | --- |
| Clash Verge Rev | 프록시 클라이언트 |
| Yaak | API 클라이언트 |
| Hoppscotch Desktop | API 개발 도구 |

출처: https://github.com/clash-verge-rev/clash-verge-rev , https://v2.tauri.app

## 5. 장단점

| 장점 | 단점 |
| --- | --- |
| 웹 UI 생태계를 그대로 쓴다 (Monaco 편집기, 명령 팔레트 라이브러리 등) | 프런트엔드와 Rust 사이에 IPC 경계가 있다. 데이터가 직렬화된다 |
| 2.x 안정판, 패키징·업데이트 도구가 갖춰져 있다 | 웹뷰가 메모리를 쓴다. 실행 파일이 작아도 전체 메모리는 별개다 |
| 설치 파일이 작다 | 순수 GPU 네이티브 렌더링 구조가 아니다 |
| 웹 개발 경험이 있으면 빠르다 | 웹 경험이 없으면 배울 것이 두 배다 (Rust + 프런트엔드) |

Zed 같은 앱에는 **외형은 높게, 구조는 다르게** 맞는다. 웹 라이브러리로 Dock·명령 팔레트를 빨리 만들 수 있지만, 렌더링은 웹뷰가 한다.

---

이전: [05-SLINT.md](05-SLINT.md) · 다음: [07-DIOXUS.md](07-DIOXUS.md) · 입구: [README.md](README.md)
