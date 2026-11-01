# 03-tauri 출발 문서 재료: Tauri 2.12.5 + 웹 프런트엔드 (`impl-tauri.md`)

- **담당**: `scouter-agy2` · 조사일 `2026-10-31`
- **기준 환경**: Windows 11, Rust `1.99.1`, `tauri` `2.12.5` ([docs.rs/tauri/2.12.5](https://docs.rs/tauri/2.12.5/tauri/)), WebView2 런타임
- **기준 명세**: `01-zed/01-LAYOUT.md`(`L-`), `02-INTERACTION.md`(`I-`), `03-THEME-ICONS.md`(`T-`), `04-APP-SPEC.md`(`A-`) 및 보충 조사 `zed-editor.md`(`E-`), `zed-pickers-dialogs.md`(`P-`), `zed-keymap-editor.md`(`K-`), `zed-extensions.md`(`X-`), `_decisions.md`
- **문서 성격**: 별도 프로젝트에서 구현할 `03-tauri/README.md`의 **출발 문서**다. 코드는 싣지 않고 아키텍처·기술 후보·01-zed 매핑·구현 전 상세 조사 체크리스트(`R-01`~`R-18`)만 정리한다.

---

## 1. 전체 구조

### 1.1 프로세스·모듈 구성도 (텍스트)

```
┌───────────────────────────────────────────────────────────────────────────────┐
│ Tauri Core 프로세스 (Rust 백엔드, src-tauri)                                   │
│ ├─ AppState / WindowManager : 다중 창(A-43.1, windows[]), 창 위치·최대화 복원   │
│ ├─ ConfigStore / StateStore : settings.json · state.json · keymap.json ·      │
│ │                             extensions.json 원자적 저장(ReplaceFileW, A-24) │
│ ├─ FsService / Watcher      : 폴더 트리 스캔, 파일 읽기/쓰기(unsaved/ 포함),    │
│ │                             notify 디스크 변경 감시(E-086), trash 휴지통     │
│ ├─ ExtensionHost            : installed/<id>/extension.toml 스캔, index.json, │
│ │                             테마·아이콘·언어(.scm, grammars/*.wasm) 자산 제공│
│ └─ Dialog / System Bridge   : 네이티브 파일·폴더 선택기, OS TaskDialog(E-080)  │
└───────────────────────────────┬───────────────────────────────▲───────────────┘
        Tauri Event (push)      │ IPC (invoke / Channel /       │ invoke 명령
        fs-changed, ext-updated ▼ convertFileSrc 커스텀 프로토콜)│
┌───────────────────────────────────────────────────────────────────────────────┐
│ WebView2 프로세스 (창마다 1개, HTML/CSS/TS 프런트엔드)                         │
│ ├─ ThemeEngine     : Zed 테마 JSON → CSS 변수(--zed-*) 실시간 주입 + 미리보기 │
│ ├─ KeymapEngine    : 포커스 컨텍스트 스택(Workspace > Pane > Editor) +        │
│ │                    VSCode 덧씌움(I-700) + Chord 상태기 + 단축키 포맷(L-219) │
│ ├─ LayoutShell     : TitleBar(32px) · Sidebar(전체 높이) · LeftDock(240px) ·  │
│ │                    PaneGroup(재귀 Split/TabBar 32px) · StatusBar(30px)      │
│ ├─ ViewerRegistry  : ViewerSpec(A-47) → EditorView(CM6 + web-tree-sitter) /   │
│ │                    MarkdownPreview(실시간 버퍼 연동) / ImageViewer          │
│ └─ OverlayLayer    : ModalLayer(top 80px, Picker P-001) · Menu · Tooltip(500ms)│
└───────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 책임 배치 (상태 관리 · 렌더링 · 파일 I/O)

| 영역 | 위치 (판단) | 동작 방식 및 01-zed 연결 | 근거 · 출처 |
|---|---|---|---|
| **영속 상태·파일 I/O** | **Rust 코어 (`src-tauri`)** | `%APPDATA%\ZedLike\` 하위 `settings.json`, `state.json`, `keymap.json`, `extensions.json`, `unsaved/{id}.txt` 읽기·쓰기(`A-20`). 저장 시 `.tmp` 작성 → `sync_all` → `ReplaceFileW`/`MoveFileExW` 원자적 교체(`A-24.1`). | [Tauri Calling Rust](https://v2.tauri.app/develop/calling-rust/), `04-APP-SPEC` `A-20`~`A-25` |
| **UI·편집·레이아웃 상태** | **WebView 프런트엔드** (+ Rust로 500ms 디바운스 스냅샷 전송) | 활성 폴더, `PaneNode` 분할 트리(`A-23.4`), 탭 배열(핀·임시 탭 `I-060`), 열린 텍스트 버퍼·Undo 이력(`300ms` 그룹 `E-059c`), 마크다운 실시간 미리보기(`I-082`)는 IPC 직렬화 지연을 피하기 위해 프런트엔드 메모리가 주도하고 변경 후 `500ms` 디바운스로 Rust에 `state.json` 저장을 요청(`A-24.2`). | [Tauri State Management](https://v2.tauri.app/develop/state-management/), `A-23`~`A-24` |
| **렌더링·스타일링** | **WebView2 (Blink + DirectComposition)** | 루트 `html { font-size: var(--ui-font-size, 16px) }`로 두어 Zed의 `rems_from_px(px) = px / 16` 규칙(`L-01`, `L-02`)을 CSS `rem`/`px`로 1:1 재현. 테마 토큰(`T-01`~`T-06`)과 누락 토큰 폴백(`T-26`)은 CSS Custom Properties로 루트에 주입. | `01-LAYOUT` `L-01`~`L-10`, `03-THEME-ICONS` `T-01`~`T-26` |
| **구문 강조·구문 선택** | **WebView (`web-tree-sitter`)** 또는 **Rust (`tree-sitter`)** | 확장 패키지의 `grammars/<lang>.wasm`이 이미 WASI/WASM으로 컴파일되어 배포되므로(`X-21`, `X-40`), 프런트엔드에서 `web-tree-sitter`로 직접 로드해 `highlights.scm` 및 구문 단위 선택(`Expand/Shrink Selection`, `E-090`~`E-092`)을 지연 없이 수행하는 구성을 기본 후보로 둔다(판단). | `zed-extensions.md` `X-29`·`X-40`, `zed-editor.md` `E-090`~`E-093` |

---

## 2. 기술 스택 (크레이트·라이브러리 후보, 버전, 라이선스)

### 2.1 Rust 백엔드 크레이트 (`src-tauri/Cargo.toml`)

| 구분 | 크레이트 후보 | 최신 안정 버전 | 라이선스 | 역할 및 채택 여부 (판단) | 출처 URL |
|---|---|---|---|---|---|
| 앱 프레임워크 | `tauri` / `tauri-build` | `2.12.5` | `Apache-2.0 OR MIT` | 다중 창(`WebviewWindowBuilder`), 커스텀 타이틀 바(`decorations(false)`), IPC(`invoke`, `Channel`), 에셋 프로토콜 | [crates.io/crates/tauri](https://crates.io/crates/tauri) · [docs.rs/tauri/2.12.5](https://docs.rs/tauri/2.12.5/tauri/) |
| 대화상자·파일·열기 | `tauri-plugin-dialog` / `tauri-plugin-fs` / `tauri-plugin-opener` | `2.8.4` / `2.7.1` / `2.7.2` | `Apache-2.0 OR MIT` | 폴더·파일 열기/저장 네이티브 창(`I-003`, `I-110`), 기본 브라우저/탐색기 열기(`Reveal in File Explorer` `I-512`) | [crates.io/crates/tauri-plugin-dialog](https://crates.io/crates/tauri-plugin-dialog) · [tauri-plugin-fs](https://crates.io/crates/tauri-plugin-fs) |
| 파일 감시 | `notify` | `8.2.1` | `CC0-1.0` | 등록 폴더 트리 감시, 열린 버퍼 디스크 변경/충돌 감지(`E-086`), `settings.json`·`keymap.json`·`extensions/installed/` 실시간 감시(`100ms`/`200ms` 디바운스 `X-39`) | [crates.io/crates/notify](https://crates.io/crates/notify) |
| 휴지통 삭제 | `trash` | `5.2.9` | `MIT` | 프로젝트 트리 `Delete` 시 OS 휴지통 이동(`I-039`, 영구 삭제 `Shift+Delete`는 `std::fs` 사용 `I-040`) | [crates.io/crates/trash](https://crates.io/crates/trash) |
| 설정·매니페스트 파서 | `serde_json` / `toml` | `1.0.154` / `1.1.8` | `MIT OR Apache-2.0` | `settings.json`·`state.json`·`keymap.json`·테마 JSON(`T-01`) 및 확장 `extension.toml`·`languages/*/config.toml`(`X-01`, `X-22`) 파싱 | [crates.io/crates/serde_json](https://crates.io/crates/serde_json) · [crates.io/crates/toml](https://crates.io/crates/toml) |
| 퍼지 검색 (선택) | `nucleo-matcher` | `0.3.1` | `MPL-2.0` | Go to File(`Ctrl+P`, `P-007`) 대규모 워크트리 파일 경로 퍼지 매칭을 Rust 백엔드에서 수행할 때 사용 | [crates.io/crates/nucleo-matcher](https://crates.io/crates/nucleo-matcher) |
| 확장 호스트 / 문법 (선택) | `tree-sitter` / `wasmtime` | `0.27.1` / `50.0.1` | `MIT` / `Apache-2.0 WITH LLVM-exception` | 우리 앱의 확장 3종(테마·아이콘·언어)은 프로시저럴 `extension.wasm` 실행이 불필요하나(`X-28`~`X-29`), Rust 측 문법 검증 또는 향후 `wasm32-wasip2` 호스트 탑재 시 사용 | [crates.io/crates/tree-sitter](https://crates.io/crates/tree-sitter) · [crates.io/crates/wasmtime](https://crates.io/crates/wasmtime) |

### 2.2 웹 프런트엔드 라이브러리 (`package.json`)

| 구분 | 패키지 후보 | 최신 버전 | 라이선스 | 비교 및 선택 판단 | 출처 URL |
|---|---|---|---|---|---|
| Tauri JS 브리지 | `@tauri-apps/api` | `2.12.4` | `Apache-2.0 OR MIT` | **필수**: `core.invoke`, `event.listen`, `window.getCurrentWindow`(창 드래그·최소화·최대화·닫기 `L-39`, `L-312`) | [npmjs.com/package/@tauri-apps/api](https://www.npmjs.com/package/@tauri-apps/api) |
| UI 프레임워크 후보 | **후보 A: `react`** (`19.3.1`)<br>**후보 B: `solid-js`** (`1.9.18`)<br>**후보 C: `svelte`** (`5.57.3`) | `19.3.1`<br>`1.9.18`<br>`5.57.3` | `MIT`<br>`MIT`<br>`MIT` | **판단 필요 (`R-01`)**: `solid-js`·`svelte`는 세밀한 반응성으로 탭·트리·커서 위치 갱신 오버헤드가 낮고, `react`는 생태계가 넓다. 어느 쪽이든 상태 스토어와 CSS 변수 구조는 동일하다. | [npmjs.com/package/react](https://www.npmjs.com/package/react) · [solid-js](https://www.npmjs.com/package/solid-js) · [svelte](https://www.npmjs.com/package/svelte) |
| 코드 편집기 코어 | **후보 A (권장): CodeMirror 6** (`@codemirror/view` `6.43.15`, `@codemirror/state` `6.7.6`, `@codemirror/search` `6.7.2`)<br>**후보 B: `monaco-editor`** (`0.57.0`) | `6.43.15`<br>`0.57.0` | `MIT`<br>`MIT` | **CodeMirror 6 권장 (판단, `R-02`)**:<br>- **Monaco**는 VSCode 자체 키맵·명령 팔레트·컨텍스트 메뉴·Find 위젯이 강하게 결합되어 있어 Zed 고유 치수(거터 폭 `E-002`~`E-005`, 툴바 내 찾기 바 `E-060`, 둥근 선택 영역 `E-015`, Tree-sitter `.scm` 쿼리 `X-23`)로 개조하기 어렵고 번들이 무겁다.<br>- **CM6**는 뷰포트 가상화·다중 커서(`E-046`)·열 선택(`E-047`)·300ms Undo 그룹(`E-059c`)을 모듈형으로 제공하며, 커스텀 Tree-sitter 하이라이터와 Zed 전용 찾기 바(`E-060`~`E-079`)·키맵 엔진을 충돌 없이 얹을 수 있다. | [npmjs.com/package/@codemirror/view](https://www.npmjs.com/package/@codemirror/view) · [monaco-editor](https://www.npmjs.com/package/monaco-editor) |
| Tree-sitter 런타임 | `web-tree-sitter` | `0.27.1` | `MIT` | 브라우저/WebView2 WebAssembly로 `grammars/<lang>.wasm`(`X-21`, `X-40`)을 로드하고 `highlights.scm`·`brackets.scm`·`indents.scm`(`X-23`) 및 구문 단위 선택(`E-090`~`E-092`) 실행 | [npmjs.com/package/web-tree-sitter](https://www.npmjs.com/package/web-tree-sitter) |
| 마크다운 렌더러 | **후보 A: `unified` + `remark-gfm` (`4.0.1`) + `remark-frontmatter`**<br>**후보 B: `markdown-it` (`15.0.3`)** + `dompurify` (`3.4.17`) | `4.0.1`<br>`15.0.3` / `3.4.17` | `MIT`<br>`MIT` / `MPL-2.0 OR Apache-2.0` | **판단 (`R-03`)**: 마크다운 미리보기는 YAML frontmatter 표 렌더링(`L-110`), 소스 줄↔프리뷰 블록 클릭 매핑(`I-084`, `A-47.5e`), 체크박스·GFM 표(`L-107`)가 필수이므로 AST 위치(source offset/line)를 보존하는 파서를 쓴다. | [npmjs.com/package/remark-gfm](https://www.npmjs.com/package/remark-gfm) · [markdown-it](https://www.npmjs.com/package/markdown-it) · [dompurify](https://www.npmjs.com/package/dompurify) |
| 가상 스크롤 | `@tanstack/virtual-core` | `3.19.9` | `MIT` | 프로젝트 트리(`L-36` 26px 행), Picker 결과(`P-004`), Keymap Editor 표(`K-020`), Extensions 카드 목록(`X-49` 110px 고정 높이)의 `uniform_list` 가상 렌더링 | [npmjs.com/package/@tanstack/virtual-core](https://www.npmjs.com/package/@tanstack/virtual-core) |

---

## 3. 01-zed 항목 → 구성 요소 연결표

| 영역 | 관련 01-zed ID | Tauri + 웹 프런트엔드 구현 방식 (컴포넌트 / API) | 기본 제공 여부 |
|---|---|---|---|
| **타이틀 바 · 창 장식** | `L-21`, `L-39`, `L-81`~`L-89`, `L-311`, `L-312`, `A-19` | `tauri.conf.json`에서 `decorations: false` 설정 후 32px HTML 헤더 구현. 드래그 영역은 `data-tauri-drag-region`, 우측 창 버튼(36×32px, 글리프 10px)은 `getCurrentWindow().minimize() / toggleMaximize() / close()` 연결. 왼쪽 ☰ 버튼과 오른쪽 `∨` 버튼(22×22px) 배치. | **일부** (창 제어 API는 Tauri 기본 제공, 32px UI·창 스냅 보정은 직접 구현) |
| **멀티 폴더 사이드바** | `L-11`, `L-12`, `L-33`~`L-35`, `I-105`~`I-111`, `I-934`~`I-940`, `A-10`~`A-15` | 창 전체 높이 왼쪽 Flex 열(기본 300px, 범위 200~800px 리사이즈). 상단 32px 빈 드래그 헤더(`border_b_1`), 중단 폴더 행(31px, 클릭 시 활성 폴더 전환, 우클릭 `Move Up`/`Move Down`/`Remove`), 하단 30px 푸터(사이드바 토글·폴더 추가). | **직접 구현** |
| **왼쪽 Dock (프로젝트 패널)** | `L-14`, `L-16`, `L-30`, `L-32`, `A-21.8`, `A-43.11` | 폭 240px 기본, 우측 1px `border` + 6px 투명 리사이즈 핸들(`cursor: col-resize`). `Ctrl+B` / `Ctrl+Shift+Y`로 열림 토글. | **직접 구현** |
| **프로젝트 트리 · 인라인 편집** | `L-36`~`L-51`, `I-030`~`I-043`, `I-087`~`I-097`, `P-021`~`P-028` | `@tanstack/virtual-core`(행 높이 26px) + 단계당 20px 들여쓰기 및 SVG/CSS 안내선(`x = depth*20 + 15`). `auto_fold_dirs`(`L-42`) 단일 자식 폴더 합치기. `Enter`/`F2` 이름 변경 및 `Ctrl+N`/`Alt+N` 생성 시 인라인 `<input>`(`border.focused`), `Delete`는 Rust `trash` 호출. | **직접 구현** (가상 스크롤 라이브러리 활용) |
| **탭 바 · 재귀 PaneGroup 분할** | `L-15`, `L-17`, `L-22`~`L-27`, `L-61`~`L-74`, `I-060`~`I-076`, `I-706`~`I-739`, `A-26` | `state.json`의 `PaneNode`(`split` \| `pane`) 트리를 재귀 렌더링(외부 레이아웃 라이브러리 대신 직접 구현해야 `min(w,h)*0.2` 가장자리 드롭 분할, 2px 삽입선 `T-26.13`, 핀 탭 경계 `I-719`, 1px 분할선 + 4px 히트영역 `I-725`~`I-727`, 더블클릭 균등화를 정확히 재현 가능). | **직접 구현** |
| **원문 코드 편집기 + 찾기 바** | `E-001`~`E-106`, `I-048`~`I-059`, `I-742`~`I-752`, `A-21.15`~`A-21.20` | **CodeMirror 6**(`@codemirror/view`, `@codemirror/state`, `@codemirror/search` 헤드리스 코어) + `web-tree-sitter` 구문 강조·구문 선택(`Shift+Alt+Right/Left`). 상단 32px 편집기 툴바(브레드크럼 + 빠른 동작 버튼 `E-038`~`E-039`)와 그 아래 열리는 Zed 규격 찾기·바꾸기 바(`E-060`~`E-079`)는 커스텀 UI로 결합. 편집기 스크롤바는 15px 각진 커스텀 오버레이(`E-030`~`E-034`). | **일부** (편집·다중커서·검색 엔진은 CM6 제공, 툴바·찾기 바·15px 스크롤바·Tree-sitter 연동은 직접 구현) |
| **마크다운 미리보기** | `L-101`~`L-111`, `I-044`~`I-047`, `I-077`~`I-086`, `A-47.5` | `unified`/`remark-gfm`/`remark-frontmatter`(또는 `markdown-it`)로 HTML 변환 후 `max-width: 800px` 컨테이너에 렌더링. 원본 CM6 버퍼 변경 시 실시간 재렌더링(`I-082`), AST 소스 오프셋 데이터 속성(`data-source-line`)으로 본문 클릭 시 원문 커서 이동(`I-084`), 로컬 이미지 경로는 `convertFileSrc`로 변환. | **일부** (파서는 라이브러리, frontmatter 표·원문 위치 연동은 직접 구현) |
| **내장 뷰어 등록 구조** | `A-47.1`~`A-47.9` | 프런트엔드에 `ViewerRegistry` 맵(`Map<string, ViewerSpec>`)을 두어 `markdown_preview`(`companion` 모드)와 `image_viewer`(`replace` 모드, `<img>` + 확대/축소)를 등록. 탭 바·툴팁·우클릭 메뉴·`state.json` 복원(`pane.tabs[].kind`)이 `ViewerSpec` 인터페이스를 공통 참조. | **직접 구현** |
| **상태 표시줄 · Go to Line** | `L-28`, `L-29`, `L-91`~`L-98`, `E-100`~`E-106` | 하단 30px Flex 바(`status_bar.background`). 좌측 사이드바/프로젝트 패널 토글 버튼, 우측 `{줄}:{열}` 버튼(클릭 또는 `Ctrl+G` 시 384px 폭 `Go to Line` 모달 `E-103`~`E-105` 오픈 및 실시간 줄 하이라이트 프리뷰). | **직접 구현** |
| **메뉴 (☰ · ∨ · 우클릭)** | `I-300`~`I-488`, `I-500`~`I-668`, `I-900`~`I-952`, `A-18`, `A-19` | 네이티브 OS 메뉴가 아닌 **DOM/CSS 팝오버 메뉴**(`elevated_surface.background`, 최소 폭 200px, 최대 높이 창의 75%, 서브메뉴 hover 지연, 우측 단축키 `Ctrl-Shift-E` 표기 `I-704`). 브라우저 기본 우클릭 메뉴는 전역 `contextmenu` 이벤트에서 `preventDefault()`로 차단. | **직접 구현** |
| **툴팁 (Hover 메시지)** | `L-200`~`L-272`, `T-30`, `A-34` | 전역 싱글턴 툴팁 포털. 일반 요소 `500ms`·트리 항목 `1500ms` 타이머(`L-206`, `L-207`), 마우스 좌표 `(+1, +1)` 배치 및 창 가장자리 반전(`L-208`~`L-210`), `elevated_surface.background` + 2중 그림자 + 우측 액션 단축키 자동 조회(`I-704`). | **직접 구현** |
| **Picker 모달 5종** | `P-001`~`P-016`, `T-21`, `A-42.2`~`A-42.4`, `A-07` | 공통 `PickerModal` 컴포넌트(상단 `80px`, 폭 `544px`, 최대 높이 `384px`, `elevation_3` 4중 그림자 `P-002`, 상단 `36px` 입력칸 `P-003`) 위에 Delegate만 교체: ① 명령 팔레트(`Ctrl+Shift+P`), ② Go to File(`Ctrl+P`, 분할 열기 `Ctrl+J/K/H/L` `I-705`), ③ Open Recent(`Ctrl+R`), ④ 테마 선택기(`Ctrl+K Ctrl+T`, 이동 시 즉시 CSS 변수 프리뷰·`Esc` 롤백), ⑤ 아이콘 테마 선택기. | **직접 구현** |
| **대화상자 · 알림(토스트)** | `E-080`~`E-085`, `P-017`~`P-020`, `A-44`, `A-45` | 저장 확인·삭제 확인·충돌 경고는 Rust 백엔드에서 Windows 네이티브 **TaskDialog**(`windows` 크레이트 `TaskDialogIndirect` 또는 `tauri-plugin-dialog`) 호출(`E-080`, `P-017`). About 창(`P-019`)과 우하단 토스트 알림(`P-020`, `A-45`)은 인앱 오버레이로 렌더링. | **일부** (OS 대화상자는 Win32/플러그인, 토스트·About은 직접 구현) |
| **키맵 엔진 · 키맵 편집 화면** | `I-001`~`I-002`, `I-700`~`I-705`, `K-001`~`K-085`, `A-27`, `A-43.9` | DOM 요소의 `data-key-context` 속성을 따라 루트→포커스 요소로 컨텍스트 경로를 수집하고, 깊이 우선·등록 순서·출처 등급(`User > Base(VSCode) > Default`) 및 2단계 Chord(`Ctrl+K ...`)를 평가하는 자체 `KeymapEngine`. `Ctrl+K Ctrl+S` 키맵 편집 화면은 6열 줄무늬 가상 테이블(`K-020`~`K-029`) + 키 녹화 모달(`KeystrokeInput` `K-050`)로 구현. | **직접 구현** |
| **테마 · 아이콘 테마** | `T-01`~`T-31`, `A-07`, `A-21.1`~`A-21.2` | One Dark/Light, Catppuccin 4종(`T-01.8`~`T-01.9`) JSON을 로드해 누락 토큰 폴백(`T-26`) 계산 후 CSS 변수로 주입. 아이콘 테마(`Zed (Default)` 및 `Catppuccin Icons` `T-20`~`T-22`)는 `file_stems` → `file_suffixes` → 기본 폴백(`X-58`) 순으로 SVG 경로를 해석해 렌더링. | **직접 구현** |
| **확장 시스템 (테마·아이콘·언어)** | `X-01`~`X-66`, `A-46` | Rust가 `%APPDATA%\ZedLike\extensions\installed\<id>\extension.toml`을 스캔해 `index.json`(`X-35`)을 구축하고, 테마 JSON·아이콘 SVG·언어 `config.toml`/`.scm`/`grammars/*.wasm`을 Tauri 커스텀 프로토콜(`asset://`)로 WebView에 제공. 확장 탭(`ctrl-shift-x`, 카드 높이 `110px` `X-49`~`X-50`) 및 `Install Dev Extension`(폴더 심볼릭 링크 `X-56`) 구현. | **직접 구현** |

---

## 4. 상세 조사·검토 항목 목록 (`R-01` ~ `R-18`)

별도 프로젝트에서 본격 구현에 들어가기 전에 PoC·문서·소스로 검증해야 할 핵심 질문 목록이다.

| ID | 질문 (검토 항목) | 왜 중요한가 | 관련 01-zed ID | 확인 방법 | 우선순위 |
|---|---|---|---|---|---|
| `R-01` | **프런트엔드 프레임워크 선택 (`Solid` vs `Svelte 5` vs `React 19`)** | 다중 Pane 분할 트리, 탭 드래그, 실시간 마크다운 미리보기, 커서 위치 표시에서 불필요한 리렌더링 없이 CodeMirror 6 인스턴스 수명주기를 안정적으로 유지해야 한다. | `L-15`, `I-706`~`I-730`, `E-001` | 2개 Pane 분할 상태에서 탭을 다른 Pane으로 옮길 때 CM6 `EditorView` DOM 노드가 파괴되지 않고 `appendChild`로 이동되는지 소형 PoC로 검증 | **P0 (최우선)** |
| `R-02` | **CodeMirror 6에서 Zed 편집기 치수·시각 규격 100% 재현 가능성** | 거터 4자리 최소 폭(`E-002`), 여백(`E-003`~`E-005`), 줄 높이 `1.618`(`E-010`), 선택 없는 커서 줄만 전체 폭 강조(`E-011`), 모서리 반경 `0.15 × 줄높이` 둥근 선택 영역(`E-015`), 15px 각진 스크롤바(`E-030`)를 맞춰야 일치도 판정(`A-31.1`)을 통과한다. | `E-001`~`E-037`, `A-31.1` | CM6 커스텀 Extension(`lineNumbers`, `drawSelection` 오버라이드 또는 Canvas/SVG 선택 레이어, 커스텀 15px 스크롤바) PoC 작성 후 픽셀 측정 | **P0 (최우선)** |
| `R-03` | **`web-tree-sitter`와 Zed `.scm` 쿼리 호환성** | Zed 언어 확장의 `highlights.scm`은 `#set!`, `#match?`, `#any-of?` 등 술어와 우→좌 점 계층 폴백(`constant.builtin.boolean` → `constant.builtin` → `constant`, `T-27.1`, `X-23`)을 사용한다. 또한 `Shift+Alt+Right/Left` 구문 선택(`E-090`~`E-092`)에 AST 순회가 필요하다. | `T-27.1`, `E-090`~`E-093`, `X-21`~`X-23`, `X-40` | 로컬 Zed의 `installed/html/grammars/html.wasm`과 `highlights.scm`을 `web-tree-sitter` `0.27.1`에서 직접 로드해 캡처·술어 동작 여부 PoC 확인 | **P0 (최우선)** |
| `R-04` | **Zed 키맵 컨텍스트 우선순위·Chord·단축키 가로채기 차단** | WebView2(Chromium)는 `Ctrl+P`(인쇄), `Ctrl+F`(브라우저 검색), `Ctrl+R`(새로고침), `Ctrl+W`(창 닫기), `Ctrl+Tab`, `F1`, `F3`, `F6`, `Alt+Left/Right`를 기본 가로챈다. 또한 Zed는 포커스 깊이 우선 규칙(`I-702`), `null` 해제 등급(`I-703`), `Ctrl+K` 연속 키(`I-001`)를 쓴다. | `I-001`~`I-002`, `I-700`~`I-705`, `A-27` | WebView2 `ICoreWebView2Settings::AreBrowserAcceleratorKeysEnabled(false)` 설정 또는 캡처 단계 `keydown` `preventDefault()`로 위 단축키 전부가 웹 앱의 `KeymapEngine`에 도달하는지 Windows 11에서 PoC 검증 | **P0 (최우선)** |
| `R-05` | **키 입력 녹화(`KeystrokeInput`)에서 OS/브라우저 예약 키 캡처** | 키맵 편집 화면(`Ctrl+K Ctrl+S`, `K-041`, `K-050`)에서 사용자가 `Ctrl+W`, `Ctrl+Shift+P`, `Alt+Space`, `Tab`, `Escape` 등을 누를 때 액션이 실행되지 않고 키 조합으로 정확히 녹화되어야 한다. | `K-003`, `K-041`, `K-050`~`K-058` | `KeystrokeInput` 포커스 상태에서 모든 조합키 입력 시 `KeyboardEvent.key`/`code`를 Zed 키 표기(`Ctrl-Shift-P`, `L-219`)로 정규화하는 매핑 표 작성 및 PoC 테스트 | **P1 (높음)** |
| `R-06` | **커스텀 타이틀 바(`decorations: false`)에서 Windows 11 창 스냅·둥근 모서리·테두리** | 기준 화면 2(`L-311`, `L-312`)처럼 비최대화 창은 10px 둥근 모서리와 1px 테두리·그림자를 갖고, 최대화 시 직각으로 꽉 차며, Windows 11 Snap Layout(최대화 버튼 hover 시 스냅 레이아웃 팝업, `Win+방향키`)과 창 가장자리 리사이즈가 작동해야 한다. | `L-21`, `L-39`, `L-311`, `L-312`, `A-30.6` | Tauri 2 Windows에서 `decorations: false` + `shadow: true`(또는 DWM `DWMWA_WINDOW_CORNER_PREFERENCE`) 조합 시 둥근 모서리·리사이즈 히트테스트·최대화 여백 유무를 PoC로 실측 | **P0 (최우선)** |
| `R-07` | **배율 200%(고DPI)에서 1px 경계선 및 글꼴 서브픽셀 렌더링 일치도** | 기준 화면은 배율 200%(`devicePixelRatio = 2`)다(`L-310`, `A-30.1`). CSS `1px solid` 테두리(`L-301`~`L-303`)와 트리 안내선(`x = depth*20 + 15`, `L-38`)이 물리 픽셀에 반올림 오차 없이 선명하게 그려지는지 확인해야 한다. | `L-01`~`L-10`, `L-38`, `L-301`~`L-304`, `A-31.1` | Windows 디스플레이 배율 100%·150%·200%에서 WebView2 캡처 후 경계선 두께와 `.ZedSans`(`IBM Plex Sans`) / `.ZedMono`(`Lilex`) 글꼴 렌더링 비교 | **P1 (높음)** |
| `R-08` | **한글 IME 조합 중 편집기·Picker·인라인 입력 동작** | 한글 조합 중(`compositionstart`~`compositionend`)에 ① CM6 다중 커서·자동 괄호 닫기(`E-057`), ② Picker 퍼지 검색(`P-003`) 실시간 결과 갱신, ③ 트리 이름 변경(`I-035`)에서 `Enter`/`Escape` 확정·취소가 마지막 글자 잘림이나 중복 입력 없이 동작해야 한다. | `I-035`, `E-057`, `P-003`, `P-021` | Windows 기본 Microsoft 입력기에서 한글 입력 중 `Enter`, `Escape`, 방향키, `Backspace` 입력 시 `KeyboardEvent.isComposing` 처리 규칙 PoC 확인 | **P1 (높음)** |
| `R-09` | **탭·트리 드래그 앤 드롭 구현 방식 (HTML5 DnD vs Pointer Events)** | Tauri WebView2는 기본적으로 OS 파일 드롭(`dragDropEnabled: true`)을 가로채므로 HTML5 `draggable="true"`가 막히거나 커스텀 드래그 고스트·`Ctrl` 키 누름에 따른 복제 전환(`I-718`)·2px 삽입선(`I-715`) 제어가 까다롭다. | `I-074`, `I-094`, `I-712`~`I-722`, `A-26.4`~`A-26.8` | HTML5 Drag API 대신 `pointerdown` / `pointermove` / `pointerup` (`setPointerCapture`) 기반 자체 드래그 엔진으로 구현하여 `Ctrl` 실시간 토글과 Pane 20% 가장자리 분할 오버레이를 검증 | **P1 (높음)** |
| `R-10` | **대용량 파일(수만 줄) 열기·편집·검색 시 IPC 및 CM6 성능** | 큰 파일을 `invoke` JSON 문자열로 넘기면 직렬화 지연이 생길 수 있다. Tauri 2의 Raw `ipc::Response`(바이트 버퍼) 또는 커스텀 프로토콜 스트리밍과 CM6 대용량 문서 성능을 확인해야 한다. | `E-001`, `E-060`~`E-074` | 10MB 텍스트 파일 로드 시 `tauri::ipc::Response` 바이트 전송 시간과 CM6 초기 렌더링·정규식 전체 검색(`E-066`) 응답 시간 측정 | **P1 (높음)** |
| `R-11` | **테마 JSON 로드·누락 토큰 폴백(`T-26`)·실시간 미리보기 구조** | 6개 기본 테마 + 확장 테마 JSON은 `#RRGGBB`/`#RRGGBBAA`가 혼재되어 있고(`T-01.7`), 누락·`null` 키는 `T-26` 규칙으로 대체해야 하며, 테마 선택기에서 위/아래 이동 시 디스크 쓰기 없이 즉시 전체 화면(CM6 토큰 포함)이 바뀌어야 한다(`T-21`, `X-60`). | `T-01`~`T-06`, `T-21`, `T-26`, `X-57`~`X-60` | 테마 JSON → 폴백 병합기 → `:root` CSS 변수 & CM6 `HighlightStyle` 동적 교체기의 전환 지연(`16ms` 이내 여부) 확인 | **P1 (높음)** |
| `R-12` | **아이콘 테마 SVG 로드 및 색조(CurrentColor vs 다색 SVG) 처리** | `Zed (Default)` UI/파일 아이콘은 단색(`icon.muted` 등 색 적용)인 반면, `Catppuccin Icons`(`T-20`, `X-20`)는 자체 색상이 들어간 SVG다. 또한 확장 설치 폴더(`%APPDATA%\ZedLike\extensions\installed\<id>\icons\...`)의 SVG를 WebView에서 빠르게 로드해야 한다. | `T-10`, `T-20`~`T-22`, `T-31`, `X-19`~`X-20`, `X-58` | Tauri `assetProtocol` (`convertFileSrc`) 스코프에 확장 폴더를 허용하고, UI 아이콘(인라인 SVG `currentColor`)과 파일 아이콘(`<img>` 또는 SVG 스프라이트 캐시) 구분 렌더링 확인 | **P1 (높음)** |
| `R-13` | **확장 시스템에서 WASM 호스트(`wasmtime`) 포함 여부 결정** | `zed-extensions.md`(`X-28`~`X-29`) 조사 결과, 우리 앱이 지원하는 확장 3종(테마·아이콘 테마·언어)은 `extension.wasm`(`wasm32-wasip2`)을 전혀 호출하지 않고 정적 파일(`themes/*.json`, `icon_themes/*.json`, `languages/*`, `grammars/*.wasm`)만으로 100% 동작한다. | `X-08`, `X-25`~`X-29`, `A-46.1`, `A-46.11` | `extension.toml`에 `[lib]`나 `[language_servers]`가 있어도 무시하고 정적 에셋과 `grammars/*.wasm`만 로드하는 정책을 확정할지, `Install Dev Extension`(`X-55`)용 `wasi-sdk` 문법 빌더까지 넣을지 결정 | **P1 (높음)** |
| `R-14` | **확장 레지스트리 데이터 소스 결정 (`api.zed.dev` vs 자체 인덱스/로컬 설치)** | Extensions 화면(`ctrl-shift-x`, `X-44`~`X-52`)에서 원격 확장 목록 검색·다운로드를 제공할 때 `api.zed.dev`를 직접 호출해도 되는지 정책 확인이 필요하다(`X-52`). | `X-43`, `X-44`~`X-52`, `A-46.9` | 1단계는 로컬 폴더/아카이브 설치 + 사전 번들 확장으로 구현하고, 원격 목록은 `zed-industries/extensions` 기반 정적 미러 또는 설정 가능한 레지스트리 URL로 둘지 결정 | **P2 (보통)** |
| `R-15` | **Windows 네이티브 확인창(`TaskDialog`) 호출 방식** | Zed는 Windows에서 `use_system_prompts: true`가 기본이라 수정된 탭 닫기(`Save` / `Don't Save` / `Cancel`, `E-081`)나 복수 탭 닫기(`Save all` / `Discard all` / `Cancel`, `E-082`)에 커스텀 버튼 라벨을 가진 OS `TaskDialog`를 띄운다. | `E-080`~`E-085`, `P-017`~`P-018`, `A-44` | `tauri-plugin-dialog`의 기본 `message`/`ask`가 3버튼 커스텀 라벨(`Save` / `Don't Save` / `Cancel`)을 지원하는지 확인하고, 미지원 시 Rust에서 `windows::Win32::UI::Controls::TaskDialogIndirect`를 직접 호출하는 커맨드 구현 | **P1 (높음)** |
| `R-16` | **설정·키맵 파일 외부 편집 감시 및 자기 쓰기(Self-write) 루프 방지** | 사용자가 편집기 탭에서 `settings.json`·`keymap.json`을 저장하거나 키맵 편집 화면에서 바인딩을 고칠 때, `notify` 파일 감시기가 중복 이벤트를 일으켜 무한 리로드나 알림 깜빡임이 발생하지 않아야 한다. | `I-704`, `K-070`~`K-075`, `A-43.7`~`A-43.8` | 파일 쓰기 시 마지막 저장 해시/타임스탬프를 기록해 동일 내용 감시 이벤트는 무시하고, 문법 오류 시에는 이전 설정을 유지하며 토스트 알림(`A-45.1`)만 띄우는지 검증 | **P2 (보통)** |
| `R-17` | **다중 창(`Ctrl+Shift+N`) 상태 격리와 종료 시 전체 창 복원(`A-25.10`)** | 창 하나를 닫을 때는 그 창만 `state.json`에서 제거하고, 마지막 창을 닫거나 앱을 종료할 때는 열려 있던 모든 창과 미저장 버퍼(`unsaved/`)를 보존해 다음 실행 시 복원해야 한다(`A-22.24`, `A-25.8`, `A-25.10`). | `A-22`, `A-24`, `A-25`, `A-43.1` | Tauri `RunEvent::WindowEvent(CloseRequested)` 및 `ExitRequested` 훅에서 개별 창 닫기 vs 앱 종료를 구분하고, 미저장 버퍼가 있을 때도 종료 확인창 없이 `unsaved/`에 즉시 기록 후 종료되는지 확인 | **P1 (높음)** |
| `R-18` | **이미지 뷰어(`A-47.6`) 및 1차 미확정 세부 명세 보충** | `04-APP-SPEC`에서 이미지 뷰어(`A-47.6a`~`f`: 지원 확장자, `replace` 여부, 임시 탭 여부, 확대/배경)와 브레드크럼 세부(`E-038`)가 조사 대기로 남아 있다. | `A-47.6`, `A-40.20`, `E-038` | 구현 착수 전 Zed `crates/image_viewer` 소스를 추가 확인하여 `ViewerSpec` 필드를 확정한 뒤 뷰어 컴포넌트 작성 | **P2 (보통)** |

---

## 5. 위험 요소와 대응

| 위험 요소 | 원인 (Tauri / WebView2 특성) | 영향 받는 명세 | 대응 방안 (판단) |
|---|---|---|---|
| **브라우저 기본 단축키·우클릭 메뉴 충돌** | Chromium WebView2가 `Ctrl+P`, `Ctrl+F`, `Ctrl+R`, `Ctrl+W`, `F1`, `F3`, `F6`, 기본 컨텍스트 메뉴를 가로챔 | `I-003`~`I-059`, `I-500`~, `K-050`, `A-31.3` | WebView2 설정에서 브라우저 가속키를 제어하거나 최상위 `window` 캡처 리스너에서 `e.preventDefault()` 처리 후 자체 `KeymapEngine`으로 라우팅(`R-04`). |
| **Pane 분할·탭 이동 시 편집기 상태 유실** | 선언형 UI 프레임워크에서 트리 구조가 바뀌면 하위 DOM/컴포넌트가 언마운트·재마운트되어 CM6 스크롤·선택·커서가 초기화될 위험 | `I-706`~`I-730`, `A-26`, `A-31.4a` | 각 탭의 CM6 `EditorView`(및 `EditorState`)를 컴포넌트 트리 바깥의 `BufferViewPool` 맵에 보관하고, Pane 슬롯에는 컨테이너 DOM 노드로 `appendChild` 마운트만 수행(`R-01`). |
| **첫 프레임 테마 깜빡임 (FOUC) 및 창 복원 점프** | WebView가 HTML을 로드한 뒤 비동기 IPC로 `settings.json`·`state.json`을 읽으면 초기 흰 화면이나 창 위치 점프가 보일 수 있음 | `T-07`, `X-59`, `A-25.1`~`A-25.10` | Rust에서 창 생성(`WebviewWindowBuilder`) 시점에 이미 읽은 `state.json` 좌표·크기와 테마 배경색(`background_color`)을 지정하고, `initialization_script`로 초기 테마·상태 JSON을 `window.__INITIAL_STATE__`에 동기 주입. |
| **네이티브 창 프레임 제거(`decorations: false`) 부작용** | OS 기본 타이틀 바를 끄면 Windows 스냅 레이아웃이나 비최대화 10px 둥근 모서리가 깨질 수 있음 | `L-21`, `L-39`, `L-311`, `L-312` | Windows 11 DWM API(`DWMWA_WINDOW_CORNER_PREFERENCE`) 및 `WM_NCHITTEST` 처리 여부를 `R-06` PoC에서 가장 먼저 검증. |
| **다중 창 메모리 사용량** | WebView2는 창마다 렌더러 프로세스를 생성하므로 GPUI보다 기본 메모리가 높음 | `A-43.1`, `A-25.10` | 백엔드 상태·파일 감시·확장 인덱스는 단일 Rust 코어 프로세스에서 공유하고, 비활성 폴더의 워크스페이스 탭은 폴더 활성화 시 지연 생성(`A-25.2`). |

---

## 6. 시작 순서 제안 (PoC 우선 7단계)

`04-APP-SPEC` 6절(`A-36.1`~`A-36.6`)의 단계와 맞추되, Tauri/WebView2의 기술 위험(`R-01`~`R-06`)을 가장 먼저 해소하는 순서다.

| 단계 | 목표 (마일스톤) | 핵심 구현·검증 내용 | 연결된 01-zed / R- ID |
|---|---|---|---|
| **1단계** | **창·단위·테마·키맵 기반 PoC** | Tauri 2.12.5 프로젝트 생성, `decorations: false` 32px 타이틀 바 + 창 스냅(`R-06`), WebView 브라우저 단축키 차단 + 포커스 컨텍스트/Chord `KeymapEngine`(`R-04`), 6개 테마 JSON → CSS 변수 주입(`R-11`), 5종 영역 골격(`L-11`~`L-17`). | `A-36.1`, `L-01`~`L-21`, `T-01`~`T-26`, `R-04`, `R-06`, `R-11` |
| **2단계** | **멀티 폴더 사이드바 · 프로젝트 트리 · 파일 I/O** | `state.json`·`settings.json` 원자적 읽기/쓰기(`A-24`), 사이드바 폴더 등록·순서 변경·제거(`A-10`~`A-15`), 왼쪽 Dock 가상 스크롤 트리(`auto_fold_dirs`, 안내선 `L-38`, 인라인 생성·이름 변경 `P-021`, `trash` 휴지통 삭제, `notify` 감시), Catppuccin/기본 아이콘 매핑(`R-12`). | `A-36.2`, `A-01`~`A-03`, `L-30`~`L-51`, `I-030`~`I-043`, `R-12` |
| **3단계** | **재귀 PaneGroup 분할 · 탭 바 · 드래그** | `PaneNode` 재귀 분할·크기 조절(1px 선 + 4px 반응, 더블클릭 균등화), 탭(임시 탭 기울임꼴·승격, 핀 탭 앞쪽 고정·닫기 보호 `I-014`), Pointer Events 기반 탭/트리 드래그 앤 드롭(20% 가장자리 분할, 2px 삽입선, `Ctrl` 복제 `R-09`). | `A-36.3`, `A-08`, `A-26`, `L-22`~`L-27`, `I-060`~`I-076`, `I-706`~`I-739`, `R-01`, `R-09` |
| **4단계** | **원문 편집기(CM6 + Tree-sitter) · 찾기 바 · 저장 확인** | CM6 편집기(`E-001`~`E-059`: 거터, 현재 줄 강조, 15px 스크롤바, 다중 커서, 한글 IME `R-08`) + `web-tree-sitter` 구문 강조 및 구문 단위 선택(`E-090`~`E-093`, `R-03`) + 툴바 및 파일 안 찾기·바꾸기 바(`E-060`~`E-079`) + `autosave: "off"` 수정 점(●)·OS `TaskDialog` 닫기 확인(`E-080`~`E-088`, `R-15`). | `A-36.3`, `A-04`, `E-001`~`E-106`, `R-02`, `R-03`, `R-08`, `R-10`, `R-15` |
| **5단계** | **내장 뷰어 등록 구조 · 마크다운 미리보기 · 이미지 뷰어** | `ViewerRegistry`(`A-47`) 구축, 첫 사례로 마크다운 미리보기(`Ctrl+Shift+V` / `Ctrl+K V`, YAML frontmatter 표, GFM 표·인용, 원본 버퍼 실시간 반영, 클릭 시 원문 커서 이동 `A-47.5`), 둘째 사례로 이미지 뷰어(`A-47.6`, `R-18`) 연결. | `A-36.3`, `A-47`, `L-101`~`L-111`, `I-044`~`I-047`, `I-077`~`I-086`, `R-03`, `R-18` |
| **6단계** | **메뉴(☰·∨·우클릭) · 툴팁 · Picker 5종 · 키맵 편집 화면** | 500ms/1500ms 툴팁(`A-34`), ☰ 앱 메뉴·`∨` 사용자 메뉴·영역별 우클릭 메뉴(`I-300`~`I-668`, `I-900`~`I-952`), 공통 `PickerModal`(`P-001`~`P-016`: 명령 팔레트, Go to File, Open Recent, 테마·아이콘 테마 실시간 프리뷰 선택기), `Ctrl+K Ctrl+S` 키맵 편집 화면(`K-001`~`K-085`, `R-05`), About·토스트 알림(`P-019`~`P-020`). | `A-36.4`, `A-36.6`, `A-42`, `A-43`, `P-001`~`P-020`, `K-001`~`K-085`, `R-05` |
| **7단계** | **확장 시스템(`X-`) · 다중 창 상태 복원 · 일치도 판정** | `extensions/installed/` 로더 및 `index.json` 캐시, Extensions 탭(`ctrl-shift-x`, 110px 카드, `Install Dev Extension` `X-44`~`X-56`), 미저장 버퍼(`unsaved/`) 및 다중 창 세션 복원(`A-25`, `R-17`), 기준 화면 2장 대비 픽셀·키·색상 일치도 점검(`A-30`~`A-35`). | `A-36.5`, `A-46`, `X-01`~`X-66`, `A-25`, `A-30`~`A-35`, `R-13`, `R-14`, `R-17` |

---

## 7. 확신 낮은 부분

1. **Windows 11에서 `decorations: false`와 스냅 레이아웃(Snap Flyout) 양립 여부 (`R-06`, 미확인)**:
   - Tauri 2에서 기본 타이틀 바를 끄고 HTML로 그린 최대화 버튼(`□`/`⧉`, `L-312`) 위에 마우스를 올렸을 때 Windows 11 고유의 스냅 레이아웃 팝업이 뜨게 하려면 Win32 서브클래싱(`WM_NCHITTEST`에서 `HTMAXBUTTON` 반환)이 추가로 필요할 수 있다. 구현 전 PoC로 확인해야 한다.
2. **Zed 모든 언어 확장의 `.scm` 쿼리와 `web-tree-sitter` 술어 호환성 (`R-03`, 미확인)**:
   - `web-tree-sitter` `0.27.1`이 기본 캡처와 표준 술어는 지원하지만, Zed의 네이티브 `crates/language`가 자체 해석하는 커스텀 디렉티브(예: `(#set! rainbow.exclude)`, `overrides.scm`의 `.inclusive` 범위 등 `X-23`)는 프런트엔드 하이라이터/에디터 확장에서 별도 해석 로직을 작성해야 한다.
3. **이미지 뷰어 상세 규격 (`A-47.6`, `R-18`, 미확인)**:
   - `04-APP-SPEC`에서 이미지 뷰어의 `ViewerSpec` 슬롯(`A-47.6a`~`f`)은 정의되었으나 Zed 원본의 정확한 확대/축소 UI·체커보드 배경색·우클릭 메뉴는 아직 조사 대기 상태다.
