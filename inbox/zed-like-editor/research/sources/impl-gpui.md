# gpui + gpui-kit 구현 출발 문서 재료 (ID 접두사 G-)

- 담당: scouter-opus · 2026-10-31 · 대상: 별도 구현 프로젝트의 `02-gpui/README.md` 재료 (상세 구현·코드 아님)
- 기준 명세: `01-zed/01-LAYOUT.md`(L-), `02-INTERACTION.md`(I-, E-, P-, K-), `03-THEME-ICONS.md`(T-), `04-APP-SPEC.md`(A-), 확장 조사 `zed-extensions.md`(X-), 결정 `_decisions.md`
- 버전 기준: crates.io 2026-10-31 조회값. **판단**은 작성자 판단이고, **미확인**은 확인하지 못한 것이다.
- Zed 소스 참조 커밋: `14b7b0ef3764d62a85070c8b5ccea105a2ad3235` (아래 `Z/` = `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/`)

---

## 1. 전체 구조

```
zedlike.exe (단일 프로세스, Rust)
├─ gpui 런타임 (gpui-pre =0.3.10, gpui-kit 0.8.1이 고정)        ← 창·입력·렌더(DirectX 11 / DirectWrite)
│   └─ Application → Window(커스텀 타이틀 바) → Root(gpui-kit)
├─ app (우리 코드)
│   ├─ state     : App / Folder / Workspace / PaneNode / Tab  (A-23, Entity<T>로 보관)
│   ├─ actions   : actions! 정의 + 키맵 로더(기본 + VSCode 덧씌움 + 사용자 keymap.json)  (A-27, K-)
│   ├─ ui        : TitleBar · Sidebar · Dock(ProjectPanel) · PaneGroup/Pane/TabBar · StatusBar
│   │              · 메뉴(☰·∨·우클릭) · Picker(명령 팔레트·파일 찾기·Open Recent·테마·Go to Line)
│   │              · 대화상자 · KeymapEditor · 내장 뷰어(Editor, MarkdownPreview, 이미지)  (A-47)
│   ├─ editor    : gpui-kit Editor(EditorState) + tree-sitter 강조 + 찾기 창  (E-)
│   ├─ theme     : Zed 테마 JSON → 토큰 → gpui-kit ThemeColor 매핑 (T-)
│   ├─ icons     : 아이콘 테마 JSON(file_icons·file_stems·file_suffixes) → SVG 경로 (T-)
│   ├─ ext       : 확장 설치·로드(extension.toml, 테마·아이콘·언어), 문법 .wasm 로드 (A-46, X-)
│   └─ io        : 파일 읽기·쓰기·휴지통, notify 감시, settings/keymap/state.json (A-20~A-25)
└─ 백그라운드 실행기 (gpui background executor: 파일 I/O, 퍼지 매칭, 파싱)
```

| ID | 항목 | 접근 | 근거 |
|---|---|---|---|
| G-001 | 프로세스 | 단일 Rust 프로세스, 창마다 `Window` 하나. New Window는 같은 프로세스에 창을 하나 더 연다 (판단) | gpui 구조 [Z/crates/gpui/README.md](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/README.md) |
| G-002 | 상태 관리 | 모든 상태를 gpui `Entity<T>`로 둔다. 변경 후 `cx.notify()`, 이벤트는 `EventEmitter`/`subscribe`. 폴더별 Workspace 엔티티를 `HashMap<FolderId, Entity<Workspace>>`로 보관 (A-15, A-23) | 위 README |
| G-003 | 렌더링 | gpui 요소 트리(tailwind식 스타일) + gpui-kit 컴포넌트. 치수는 rem·DynamicSpacing 규칙을 우리 쪽 헬퍼로 재현 (L-01~L-10) | [docs.rs gpui-component 0.8.1](https://docs.rs/gpui-component/0.8.1/gpui_component/) |
| G-004 | 파일 I/O | `smol`/gpui 백그라운드 실행기에서 읽기·쓰기. 원자적 쓰기(A-24.1), 휴지통은 `trash`, 감시는 `notify` | 2절 |
| G-005 | 액션·키 | gpui `actions!` + `KeyBinding` + `key_context` 체계를 그대로 쓴다. Zed와 같은 엔진이라 컨텍스트 우선순위 재현이 가장 쉽다 (판단) | [Z/crates/gpui/src/keymap.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/keymap.rs) |

## 2. 기술 스택

| ID | 크레이트 | 버전 | 라이선스 | 역할 | 출처 |
|---|---|---|---|---|---|
| G-010 | `gpui-kit` | 0.8.1 | Apache-2.0 | 진입점 (gpui + gpui-base + gpui-component + 자산) | [crates.io](https://crates.io/crates/gpui-kit) |
| G-011 | `gpui-component` | 0.8.1 | Apache-2.0 | dock·tab·tree·sidebar·input(Editor)·text(markdown)·menu·dialog·notification·table·theme·TitleBar·resizable·scroll·kbd | [docs.rs](https://docs.rs/gpui-component/0.8.1/gpui_component/) |
| G-012 | `gpui-base` | 0.8.1 | Apache-2.0 | 상태·동작 계층 (rich text: `markdown` 1.0, `html5ever` 0.27, `ropey` =2.0.0-beta.1) | [crates.io 의존성](https://crates.io/crates/gpui-base/0.8.1/dependencies) |
| G-013 | `gpui-pre` | =0.3.10 | Apache-2.0 | Zed `gpui` 스냅샷(zed@08cbb88). **직접 의존하지 않고 gpui-kit 재수출을 쓴다** | [crates.io](https://crates.io/crates/gpui-pre) |
| G-014 | `tree-sitter` | 0.26.x (gpui-component가 `^0.26.13`, optional), 최신 0.27.1 | MIT | 구문 강조·접기·구문 단위 선택. gpui-component는 `wasm` 기능을 켜지 않는다 → 확장 문법(.wasm)을 쓰려면 직접 켜야 한다 | [crates.io](https://crates.io/crates/tree-sitter), [gpui-component 의존성](https://crates.io/crates/gpui-component/0.8.1/dependencies) |
| G-015 | `tree-sitter-*` (rust·md·json·toml·python 등 30여 종, optional 기능) | 각 0.2x | 대부분 MIT | 내장 언어. gpui-component 기능 `tree-sitter-languages` 또는 개별 기능 | 위 의존성 목록 |
| G-016 | `notify` | 8.2.x (gpui-component가 이미 의존) | CC0-1.0 | 폴더 트리·열린 파일·settings/keymap.json 감시 | [crates.io](https://crates.io/crates/notify) |
| G-017 | `trash` | 5.2.9 | MIT | Windows 휴지통 (I-552) | [crates.io](https://crates.io/crates/trash) |
| G-018 | `rfd` | 0.17.2 | MIT | 폴더·파일 열기, 다른 이름 저장 (OS 대화상자) | [crates.io](https://crates.io/crates/rfd) |
| G-019 | `windows` | 0.62.2 | MIT/Apache-2.0 | TaskDialog 확인창(E-080), `ReplaceFileW`(A-24.1), 시스템 설정 조회 | [crates.io](https://crates.io/crates/windows) |
| G-020 | `serde_json` + `jsonc-parser` | 1.0.154 / 0.34.0 | MIT(/Apache) | 설정·키맵·테마 JSON. Zed 설정 파일은 주석·끝 쉼표를 허용 → JSONC 파서 필요 (판단) | [serde_json](https://crates.io/crates/serde_json), [jsonc-parser](https://crates.io/crates/jsonc-parser) |
| G-021 | `nucleo` | 0.5.0 | MPL-2.0 | Picker 퍼지 매칭 후보 (Zed는 자체 `fuzzy` 크레이트, GPL이라 쓰지 않는다) | [crates.io](https://crates.io/crates/nucleo) |
| G-022 | `wasmtime` | 50.0.1 | Apache-2.0 WITH LLVM-exception | 확장 문법 .wasm 로드(tree-sitter `wasm` 기능이 사용). 우리 범위(테마·아이콘·언어)에서는 `extension.wasm` 실행기가 필요 없다 (X-29) | [crates.io](https://crates.io/crates/wasmtime) |
| G-023 | `dirs` | 7.0.0 | MIT/Apache-2.0 | `%APPDATA%` 경로 (A-20, K-090) | [crates.io](https://crates.io/crates/dirs) |
| G-024 | 글꼴·아이콘 자산 | IBM Plex Sans, Lilex(OFL), Lucide(ISC), Zed 테마(MIT) | 각각 | `.ZedSans`·`.ZedMono` 별칭 재현, 아이콘 SVG | 03-THEME-ICONS T-14~T-17 |

## 3. 01-zed 항목 → 구성 요소 연결

기본 제공: **있음** = gpui-kit 컴포넌트로 거의 그대로 / **일부** = 컴포넌트는 있으나 모양·동작을 Zed에 맞춰 손봐야 함 / **직접** = 새로 만든다.

| ID | 영역 | 01-zed ID | 쓸 구성 요소 | 기본 제공 |
|---|---|---|---|---|
| G-030 | 창·타이틀 바 | L-21, L-39, L-81~L-89, I-645 | `TitleBar` + gpui `WindowControlArea`(Drag/Min/Max/Close). 창 버튼 글리프(Segoe Fluent Icons)·닫기 #E81120은 Zed 방식 재현 | 일부 |
| G-031 | 사이드바(등록 폴더) | L-11, L-12, L-33~L-35, A-10~A-17 | `sidebar` 또는 `list`/`VirtualList` + `resizable` | 일부 |
| G-032 | 왼쪽 Dock | L-14, L-16, L-30~L-32 | `dock`(DockArea) 또는 `resizable` 패널. 6px 핸들·더블클릭 초기화 | 일부 |
| G-033 | 프로젝트 트리 | L-36~L-52, I-087~I-097 | `tree`(Tree, TreeState 가상화). 접힌 경로(`docs \ guides`)·들여쓰기 안내선·인라인 이름 입력·드래그 | 일부 |
| G-034 | 탭 바·Pane 분할 | L-22~L-27, L-61~L-74, I-060~I-076, A-26 | `tab`(TabBar) + `dock`/`resizable`. Zed 탭 테두리 규칙·미리보기 탭(기울임)·핀 영역·드래그 분할 영역은 직접 | 직접 (뼈대만 있음) |
| G-035 | 편집기 | E-001~E-059, A-04 | `input::Editor`/`EditorState`(Rope, tree-sitter `SyntaxHighlighter`). 거터 치수·현재 줄·커서 2px·선택 모서리·스크롤바 15px·클릭 횟수 선택·Alt 다중 커서·열 선택은 지원 범위 미확인 | 일부 (G-101) |
| G-036 | 찾기·바꾸기 창 | E-060~E-079 | 입력·IconButton·toolbar 직접 조립. gpui-kit `input`에 검색 기능(Search 액션)이 있으나 Zed 모양과 다름 | 직접 |
| G-037 | 마크다운 미리보기 | L-101~L-111, I-077~I-086 | `text::markdown()`/`TextView` + `MarkdownExtensions`. frontmatter 표·제목 밑줄·표 줄무늬·체크박스 토글·원문 동기화는 직접 | 일부 |
| G-038 | 상태 표시줄 | L-28, L-91~L-98, E-100~E-106 | `status_bar` 모듈 + IconButton | 일부 |
| G-039 | 메뉴 (☰·∨·우클릭) | I-300~I-488, I-600~I-655, I-500~I-668(재번호 I-9xx) | `menu`(PopupMenu/ContextMenu), `native_menu`. 최소 폭 200·75% 높이·단축키 표시·50ms 깜빡임은 맞춤 | 일부 |
| G-040 | 툴팁 | L-200~L-272 | gpui 툴팁 + gpui-kit `Tooltip`. 500ms·마우스 기준 (+1,+1)·뒤집기·elevation_2는 gpui 동작과 같을 가능성이 높다 (G-111) | 있음/일부 |
| G-041 | Picker | P-001~ | gpui-kit `command`(명령 팔레트) + `list`/`searchable_list` | 일부 |
| G-042 | 대화상자 | E-080~E-089, P- | Windows TaskDialog(`windows` 크레이트) — Zed와 같음. 모달은 `dialog` | 직접 (OS) |
| G-043 | 키맵 편집 화면 | K-001~K-097 | `table`(DataTable: 열 폭 조절·정렬·가상 스크롤) + 키 입력 녹화 컴포넌트 | 일부 (녹화 직접) |
| G-044 | 테마 | T-01~T-31, A-21 | `theme`(Theme, ThemeColor, ThemeRegistry). Zed 테마 JSON 키 → gpui-kit 토큰 매핑 표 필요 | 일부 (매핑 직접) |
| G-045 | 아이콘 테마 | T-20~T-29 | `Icon::path` + 아이콘 테마 JSON 해석기 | 직접 |
| G-046 | 확장 | A-46, X-01~X-64 | 직접: extension.toml 파서, 설치 폴더, 테마·아이콘·언어 등록, 문법 .wasm 로드 | 직접 |
| G-047 | 내장 뷰어 구조 | A-47 | `Item` 같은 트레이트(탭 제목·아이콘·툴팁·우클릭 추가 항목) + 파일 종류 → 뷰어 등록표. 첫 사례 마크다운 미리보기, 다음 이미지 | 직접 |
| G-048 | 설정·상태 파일 | A-20~A-25 | serde + 원자적 쓰기 + notify 재로드 | 직접 |

## 4. 상세 조사·검토 항목 (구현 전에 확인)

| ID | 질문 | 왜 중요한가 | 관련 01-zed ID | 확인 방법 | 우선순위 |
|---|---|---|---|---|---|
| G-100 | gpui-pre 0.3.10에서 **한글 IME**(조합 중 표시·후보창 위치·확정)가 gpui-kit Editor·Input·인라인 이름 입력에서 제대로 되는가 | 한국어 문서 편집이 주 용도 | E-, I-091, K-055 | PoC: Editor에 한글 입력, 조합 중 커서·후보창 위치 캡처. Zed `gpui_windows` IME 처리와 비교 | 높음 |
| G-101 | gpui-kit `Editor`가 Zed 편집기 동작을 어디까지 지원하나: 거터(줄 번호 최소 4자리·접기 화살표), 현재 줄 강조 범위, 커서 2px·500ms 깜빡임, 선택 모서리 0.15×줄 높이, 더블/트리플 클릭, Alt+클릭 다중 커서, Ctrl+Shift 드래그 열 선택, 선택 끌어 옮기기, soft wrap, 들여쓰기 안내선, 공백 표시, 스크롤바 15px·마커 | 요건 4의 핵심. 부족하면 편집기 요소를 직접 써야 한다 | E-001~E-051 | docs.rs `input` 모듈 + 소스 읽기, PoC로 항목별 체크 | 높음 |
| G-102 | 큰 파일 성능 (수 MB·10만 줄): Rope(ropey 2.0 beta) + tree-sitter 증분 파싱 + 가상 렌더링 | 편집기 반응성 | E-010, E-035 | PoC: 10만 줄 파일 열기·스크롤·입력 지연 측정 | 높음 |
| G-103 | 커스텀 타이틀 바에서 **Windows 스냅 레이아웃**(최대화 버튼 hover), 더블클릭 최대화, 끌기 이동, 창 테두리 리사이즈, 최소 크기 360×240이 되는가 | 타이틀 바 재현 | L-21, L-39, L-81~L-84 | PoC: `WindowControlArea` Max에 hover → 스냅 플라이아웃 확인. Zed `Z/crates/gpui_windows/src/window.rs` 대조 | 높음 |
| G-104 | **탭 드래그·Pane 분할**: 탭을 끌어 순서 변경, 다른 Pane 이동, 가장자리 드롭으로 새 분할, 드롭 오버레이 색, 분할선 끌기 | 사용자 필수 요건 | I-074, A-26, I-7xx | gpui-kit `dock`(DockArea·TabGroup 드래그) 지원 범위 확인 → PoC | 높음 |
| G-105 | **키맵 컨텍스트 우선순위** 재현: 기본 + VSCode 덧씌움 + 사용자 keymap.json, `unbind`, 컨텍스트 술어(`Editor && mode == full`, `>` 자손) | 키·메뉴·툴팁 표시가 모두 여기에 기댄다 | A-27, L-217, K-045, K-093 | gpui `KeyBindingContextPredicate` 공개 여부 확인(gpui-pre 재수출), JSON 로더 PoC | 높음 |
| G-106 | **테마 토큰 매핑**: Zed 테마 JSON(`style.*` 수백 키)을 gpui-kit `ThemeColor`/`ThemeConfig`로 어떻게 옮기나. 우리 화면에만 쓰는 토큰(`editor.active_line.background` 등)은 별도 구조체가 필요한가 | 6개 테마(One·Catppuccin) 색 일치 판정 | T-01~T-26, A-31.8 | gpui-kit theme 소스·JSON 스키마 대조, 토큰 매핑 표 작성 | 높음 |
| G-107 | **아이콘 테마** 적용: 아이콘 테마 JSON의 `file_stems`·`file_suffixes`·`named_directory_icons` 규칙과 SVG 렌더(색 덮어쓰기 포함, resvg) | 트리·탭 아이콘 일치 | T-20~T-29 | `Icon::path` SVG 렌더 PoC, Catppuccin Icons 패키지 하나로 시험 | 중간 |
| G-108 | **확장 문법 로드**: tree-sitter `wasm` 기능 + wasmtime으로 `grammars/*.wasm`을 런타임에 불러와 gpui-kit `LanguageRegistry`에 등록할 수 있나. 강조 쿼리(`highlights.scm`) 캡처 이름 → 테마 syntax 토큰 매핑 | 확장(언어) 지원 | A-46, X-29, X-36~X-43 | tree-sitter `WasmStore` 문서·Zed `language` 크레이트 대조, PoC로 Zed 확장 하나 로드 | 중간 |
| G-109 | 확장 패키지 처리: extension.toml 파싱, 설치 폴더(`%LOCALAPPDATA%`?), index, 테마·아이콘 테마 목록 갱신, 제거·재로드 | 앱 재빌드 없는 설치 | X-01~X-43, A-46 | zed-extensions.md의 경로·순서를 그대로 따를지 결정 | 중간 |
| G-110 | **설정·키맵 JSON 감시**: settings.json·keymap.json을 탭에서 고쳐 저장하면 즉시 반영(테마·글꼴·키). JSONC(주석·끝 쉼표) 허용 | 설정 파일 열기 결정 | A-21, K-096 | notify + jsonc 파서 PoC | 중간 |
| G-111 | gpui 툴팁·메뉴 기본 동작이 Zed와 같은지 (같은 gpui 계열이므로 대부분 같다고 예상): 500ms·(+1,+1)·뒤집기, `ContextMenu` 최소 폭 200·75% 높이 | 툴팁·메뉴 일치 | L-206~L-210, I-504~I-521 | gpui-pre 소스 대조 (Zed 소스와 차이 확인) | 중간 |
| G-112 | 고DPI·배율 변경: 100/150/200% 및 창을 다른 배율 모니터로 옮길 때 rem·px·글꼴 선명도 | 기준 화면 200% 가정 | A-30.1, L-01 | PoC: 배율별 캡처, 타이틀 바 32px 확인 | 중간 |
| G-113 | **상태 복원**: Pane 분할 트리·탭·임시·핀·Dock 폭·창 위치 저장·복원, 원자적 쓰기(`ReplaceFileW`) | 재시작 사용감 | A-22~A-25 | 직렬화 PoC, 비정상 종료 시험 | 중간 |
| G-114 | 마크다운 미리보기: gpui-kit `markdown()`으로 frontmatter 2열 표, h1/h2 밑줄, 표 줄무늬·머리행 색, 인용 4px 선, 링크 밑줄, 체크박스 클릭 → 원문 수정, 원문 커서 → 블록 강조·스크롤이 가능한가 | 기준 화면의 주 내용 | L-101~L-111, I-077~I-086 | `MarkdownExtensions`/`MarkdownPlugin` API 확인, 기준 문서로 PoC 캡처 비교 | 높음 |
| G-115 | 디스크 변경 감지·충돌: 깨끗한 버퍼 자동 다시 읽기, 수정 중이면 충돌 표시, 지워진 파일 취소선 | 저장 사용감 | E-084~E-087, A-44 | notify 이벤트 + mtime 비교 PoC | 중간 |
| G-116 | Picker 공통(모달 위치·폭·퍼지 매칭 성능): 수만 개 파일 Go to File | 파일 찾기 결정 포함 | P-001~ | nucleo vs gpui-kit `searchable_list` 비교 PoC | 중간 |
| G-117 | 키맵 편집 화면: gpui-kit `table`이 열 폭 재분배·줄무늬·가상 스크롤·행 선택 테두리를 지원하나, 키 녹화(Esc×3로 끝내기, 최대 3타) 입력 가로채기 | 키맵 편집 결정 포함 | K-010~K-097 | `table` 문서·소스, 키 입력 가로채기 PoC | 중간 |
| G-118 | gpui-kit·gpui-pre 버전 고정과 업그레이드 경로: pre-1.0이라 API가 바뀐다. `=0.8.1` 고정 + Cargo.lock 커밋 | 장기 유지 | — | gpui-kit 릴리스 노트 추적 | 중간 |
| G-119 | 접근성(AccessKit, Narrator)과 키보드만으로 모든 조작 | 품질 | L-05, I-517 | gpui-kit AccessKit 지원 범위 확인 | 낮음 |
| G-120 | 패키징·서명: MSI/MSIX 또는 설치 프로그램, 코드 서명(스마트 앱 컨트롤 환경), 글꼴·아이콘 자산 포함, 라이선스 고지 | 배포 | T-17, A-40.5 | cargo-packager 등 후보 비교, 서명 인증서 결정 | 낮음 |
| G-121 | 빌드 준비물: MSVC, Windows SDK, CMake(gpui-kit 설치 문서), Rust 1.92 이상 | 출발 조건 | — | gpui-kit 설치 문서 대조 ([gpui-kit.com](https://gpui-kit.com/docs/installation/)) | 낮음 |
| G-122 | 내장 뷰어 등록 구조: 파일 종류 → 뷰어 탭, 탭 아이콘·제목·툴팁·우클릭 추가 항목을 한 트레이트로 표현. 이미지 뷰어를 두 번째 사례로 | 확장 없이 뷰어를 늘리는 결정 | A-47 | 마크다운 미리보기·이미지 뷰어로 설계 PoC | 낮음 |

## 5. 위험 요소와 대응

| ID | 위험 | 영향 | 대응 (판단) |
|---|---|---|---|
| G-130 | gpui·gpui-kit pre-1.0, 잦은 API 변경 | 업그레이드마다 수정 | 버전 고정, 우리 쪽 얇은 래퍼 계층(ui 헬퍼)으로 gpui-kit 의존을 가둔다 |
| G-131 | gpui-kit Editor가 Zed 편집기 수준에 못 미침 | 요건 4 미달 | G-101 PoC를 맨 먼저. 모자라면 gpui `Element`로 편집기 요소를 직접 작성(가장 큰 작업) |
| G-132 | 한글 IME 문제 | 주 사용 불가 | G-100을 첫 주에 확인. 문제가 있으면 gpui-kit·gpui 저장소에 이슈, 우회 경로 조사 |
| G-133 | Zed GPL 코드 오염 | 라이선스 위반 | Zed 소스는 사실만 참고, 코드 복사 금지. gpui(Apache-2.0)·gpui-kit(Apache-2.0)만 의존. `fuzzy`·`editor` 등 Zed GPL 크레이트를 의존하지 않는다 |
| G-134 | 테마 토큰 불일치 | 색 일치 판정 실패 | 매핑 표(G-106)를 단일 원본으로 두고 자동 비교 스크립트로 검사 |
| G-135 | 탭 드래그·분할 구현량 | 일정 | gpui-kit `dock` 재사용 가능 범위를 먼저 확정. 부족하면 Zed 동작 명세(I-7xx)대로 직접 구현 |
| G-136 | wasmtime 크기·빌드 시간 | 실행 파일 비대 | 확장 문법 지원을 기능 플래그로 분리. 내장 문법은 정적 링크 |

## 6. 시작 순서 제안 (PoC 우선)

| 단계 | 내용 | 끝났다고 보는 기준 |
|---|---|---|
| 1 | 빈 창 + 커스텀 타이틀 바 + 창 버튼 (G-103) | 스냅·더블클릭 최대화·32px 확인 |
| 2 | gpui-kit Editor PoC: 한글 IME, 큰 파일, 선택·커서 동작 (G-100, G-101, G-102) | 판정표에서 직접 구현 범위 확정 |
| 3 | 레이아웃 뼈대: 사이드바·Dock·Pane·탭 바·상태 표시줄, rem 헬퍼 (L-11~L-40) | 치수 점검표 A-32 ±1px |
| 4 | 키맵 로더 + 액션·컨텍스트 + 툴팁·메뉴 단축키 표시 (G-105, G-111) | VSCode 덧씌움 결과로 메뉴·툴팁 키 일치 |
| 5 | 폴더 등록·트리·파일 작업·감시 (A-10~A-17, I-087~I-097, G-115) | 시나리오 A-33.2, A-33.4 |
| 6 | 탭·미리보기 탭·핀·분할·드래그 (G-104), 마크다운 미리보기 (G-114) | 시나리오 A-33.3, A-33.7 |
| 7 | 테마·아이콘 테마 매핑과 선택기 (G-106, G-107) | 6개 테마 색 비교 통과 |
| 8 | 찾기 창·Picker·키맵 편집·확장·상태 복원 (G-108~G-117) | 일치도 판정 A-35 |

## 7. 확신 낮은 부분

- **gpui-kit Editor 지원 범위 (G-035, G-101):** docs.rs 모듈 목록과 의존성만 보고 판단했다. 거터·다중 커서·열 선택 등 세부 지원은 미확인이다.
- **dock 드래그 분할 (G-034, G-104):** gpui-kit `dock`이 Zed식 Pane 분할(가장자리 드롭으로 새 분할)을 지원하는지 미확인이다.
- **tree-sitter wasm 기능 (G-014, G-108):** gpui-component가 `wasm` 기능 없이 tree-sitter를 쓴다는 것은 crates.io 의존성 정보에서 확인했다. 우리가 기능을 켜도 gpui-component 강조기와 함께 쓸 수 있는지(같은 tree-sitter 버전 공유)는 미확인이다.
- **TaskDialog (G-042):** gpui-pre가 Zed처럼 Windows 프롬프트를 TaskDialog로 구현했는지 미확인이다 (Zed 소스 `gpui_windows`에서는 확인).
- **nucleo 라이선스:** MPL-2.0이다. 파일 단위 카피레프트라 정적 링크 배포에는 문제없다고 보지만 판단이다.
- **확장 설치 경로 (G-109):** zed-extensions.md X-30~X-35를 따른다는 전제이고, 우리 앱 이름으로 바꾼 경로는 정하지 않았다.
- **버전:** crates.io 최신값은 조회 시점(2026-10-31) 기준이다. gpui-kit가 고정한 하위 의존성(tree-sitter 0.26 등)과 최신값이 다를 수 있다.
