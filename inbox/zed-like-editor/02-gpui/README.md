---
tags: [rust, gui, gpui, gpui-kit, zed, editor, starter]
created: 2026-10-31
updated: 2026-10-31
---

# 02-gpui · gpui-kit 프로젝트 출발 문서

> 이 파일을 읽는 때: gpui-kit으로 Zed 같은 편집기를 **별도 프로젝트에서 만들기 시작할 때**. 전체 구조와 접근, 구현 전에 확인할 항목을 찾을 때.

| 항목 | 내용 |
| --- | --- |
| 상태 | **출발 문서.** 구현 코드와 상세 절차는 없다. 전체 내용과 "구현 전에 확인할 목록"만 담는다 |
| 기준 버전 | Windows 11, Rust 1.99.1, gpui-kit 0.8.1 (내부 gpui는 `gpui-pre =0.3.10`), Zed 커밋 [14b7b0e](https://github.com/zed-industries/zed/commit/14b7b0ef3764d62a85070c8b5ccea105a2ad3235). 버전은 crates.io 2026-10-31 조회값 |
| 명세 기준 | [../01-zed/](../01-zed/)의 01~04. 항목 ID로 참조한다 |
| 짝 문서 | [../03-tauri/README.md](../03-tauri/README.md) (같은 절 번호·제목) |
| 표기 | **판단** = 작성자 판단, **미확인** = 확인하지 못함. 원본 재료의 표기를 그대로 두었다 |
| 검증 | 실행 검증 안 함. 크레이트 정보는 crates.io·docs.rs 기준 |

## 1. 이 문서의 역할

| 항목 | 내용 |
| --- | --- |
| 무엇 | 앞으로 만들 **gpui-kit 앱 프로젝트의 출발점**이다. 이 묶음(`zed-like-editor/`)은 명세와 출발 문서만 둔다 |
| 명세 기준 | 화면·동작·색·키는 **01-zed가 기준**이다. 이 문서는 그것을 gpui-kit의 어느 부분으로 만들지를 정한다 |
| 쓰지 않는 것 | 코드, 모듈 내부 설계, 단계별 구현 절차. 구현 프로젝트에서 새로 정한다 |
| 쓰는 것 | 전체 구조, 기술 스택, 01-zed 항목 ↔ 구성 요소, **구현 전 확인 목록(5절)**, 위험, PoC 순서 |
| 사용자 결정 | 방식은 포크가 아니라 **reverse engineering 재구현**이다. Zed 앱 코드(GPL-3.0)는 복사하지 않고 gpui·gpui-kit(둘 다 Apache-2.0)만 의존한다. 포함·제외는 `_decisions.md`가 최우선이다 |

01-zed의 ID 접두사:

| 접두사 | 문서 | 내용 |
| --- | --- | --- |
| `L-` | [01-LAYOUT.md](../01-zed/01-LAYOUT.md) | 화면 구조·치수·색 상태 |
| `I-`, `E-`, `P-`, `K-`, `X-` | [02-INTERACTION.md](../01-zed/02-INTERACTION.md) | 키·탭·메뉴(I), 편집기(E), Picker·대화상자(P), 키맵 편집(K), 확장(X) |
| `T-` | [03-THEME-ICONS.md](../01-zed/03-THEME-ICONS.md) | 테마 토큰, 아이콘, 글꼴, 라이선스 고지 |
| `A-` | [04-APP-SPEC.md](../01-zed/04-APP-SPEC.md) | 우리 앱 명세, 데이터 모델, 일치도 판정 |
| `G-` | 이 문서 | gpui 쪽 항목 (구조 G-001~, 스택 G-010~, 영역 G-030~, 확인 G-100~, 위험 G-130~) |

읽는 순서: 1~2절(전체) → 4절(영역별 연결) → **5절(먼저 PoC로 확인할 것)** → 7절.

## 2. 전체 구조

```
zedlike.exe (단일 프로세스, Rust)
├─ gpui 런타임 (gpui-pre =0.3.10, gpui-kit 0.8.1이 고정)      ← 창·입력·렌더 (DirectX 11 / DirectWrite)
│   └─ Application → Window(커스텀 타이틀 바) → Root(gpui-kit)
├─ app (우리 코드)
│   ├─ state   : App / Folder / Workspace / PaneNode / Tab            (A-23, Entity<T>로 보관)
│   ├─ actions : actions! 정의 + 키맵 로더(기본 + VSCode 덧씌움 + 사용자 keymap.json)   (A-27, K-)
│   ├─ ui      : TitleBar · Sidebar · Dock(ProjectPanel) · PaneGroup/Pane/TabBar · StatusBar
│   │            · 메뉴(☰·∨·우클릭) · Picker(명령 팔레트·파일 찾기·Open Recent·테마·Go to Line)
│   │            · 대화상자·About 창 · KeymapEditor · ExtensionsPage
│   │            · 내장 뷰어(Editor, MarkdownPreview, 이미지)                             (A-47)
│   ├─ editor  : gpui-kit Editor(EditorState) + tree-sitter 강조 + 찾기 창                (E-)
│   ├─ theme   : Zed 테마 JSON → 토큰 → gpui-kit ThemeColor 매핑                          (T-)
│   ├─ icons   : 아이콘 테마 JSON(file_stems·file_suffixes·named_directory_icons) → SVG   (T-)
│   ├─ ext     : 로컬 설치·제거·개발용 설치, extension.toml, 테마·아이콘·언어 등록, 문법 .wasm 로드 (A-46~A-57, X-)
│   └─ io      : 파일 읽기·쓰기·휴지통, notify 감시, settings/keymap/state.json              (A-20~A-25)
└─ 백그라운드 실행기 (gpui background executor: 파일 I/O, 퍼지 매칭, 파싱)
```

| ID | 항목 | 접근 | 근거 |
| --- | --- | --- | --- |
| G-001 | 프로세스 | 단일 Rust 프로세스, 창마다 `Window` 하나. New Window는 같은 프로세스에 창을 하나 더 연다 (판단) | [gpui README](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/README.md) |
| G-002 | 상태 관리 | 모든 상태를 gpui `Entity<T>`로 둔다. 변경 후 `cx.notify()`, 이벤트는 `EventEmitter`·`subscribe`. 폴더별 Workspace 엔티티를 `HashMap<FolderId, Entity<Workspace>>`로 보관 (A-15, A-23) | 위 README |
| G-003 | 렌더링 | gpui 요소 트리(tailwind식 스타일) + gpui-kit 컴포넌트. 치수는 rem·간격 규칙을 우리 쪽 헬퍼로 재현 (L-01~L-10) | [docs.rs gpui-component 0.8.1](https://docs.rs/gpui-component/0.8.1/gpui_component/) |
| G-004 | 파일 I/O | `smol`·gpui 백그라운드 실행기에서 읽기·쓰기. 원자적 쓰기(A-24.1), 휴지통은 `trash`, 감시는 `notify` | 3절 |
| G-005 | 액션·키 | gpui `actions!` + `KeyBinding` + `key_context`를 그대로 쓴다. Zed와 같은 엔진이라 컨텍스트 우선순위 재현이 가장 쉽다 (판단) | [keymap.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/keymap.rs) |
| G-006 | 설정·키맵 | settings.json·keymap.json은 JSONC(주석·끝 쉼표 허용)로 읽는다. 사용자가 탭에서 고쳐 저장하면 감시로 즉시 반영 (A-21, K-096) | G-110 |
| G-007 | 확장 호스트 | 앱 안의 `ext` 모듈이 `installed/<ID>/`를 읽어 테마·아이콘 테마·언어 레지스트리에 합류시킨다. **`extension.wasm` 실행기(wasmtime 컴포넌트 호스트)는 두지 않는다.** 문법 `.wasm`만 tree-sitter가 읽는다 (X-29) | 3절 G-014, G-022 |

## 3. 기술 스택

| ID | 크레이트 | 버전 | 라이선스 | 역할 | 출처 |
| --- | --- | --- | --- | --- | --- |
| G-010 | `gpui-kit` | 0.8.1 | Apache-2.0 | 진입점 (gpui + gpui-base + gpui-component + 자산). **`gpui`를 따로 추가하지 않는다** | [crates.io](https://crates.io/crates/gpui-kit) |
| G-011 | `gpui-component` | 0.8.1 | Apache-2.0 | dock·tab·tree·sidebar·input(Editor)·text(markdown)·menu·dialog·notification·table·theme·TitleBar·resizable·scroll·kbd | [docs.rs](https://docs.rs/gpui-component/0.8.1/gpui_component/) |
| G-012 | `gpui-base` | 0.8.1 | Apache-2.0 | 상태·동작 계층 (rich text: `markdown` 1.0, `html5ever` 0.27, `ropey` =2.0.0-beta.1) | [의존성](https://crates.io/crates/gpui-base/0.8.1/dependencies) |
| G-013 | `gpui-pre` | =0.3.10 | Apache-2.0 | Zed `gpui` 스냅샷(zed@08cbb88). **직접 의존하지 않고 gpui-kit 재수출을 쓴다** | [crates.io](https://crates.io/crates/gpui-pre) |
| G-014 | `tree-sitter` | 0.26.x (gpui-component가 `^0.26.13`, optional), 최신 0.27.1 | MIT | 구문 강조·구문 단위 선택. gpui-component는 `wasm` 기능을 켜지 않는다 → 확장 문법(.wasm)을 쓰려면 직접 켜야 하고, 켜면 `wasmtime-c-api-impl`이 따라온다 (G-022) | [crates.io](https://crates.io/crates/tree-sitter), [gpui-component 의존성](https://crates.io/crates/gpui-component/0.8.1/dependencies) |
| G-015 | `tree-sitter-*` (rust·md·json·toml·python 등 30여 종, optional) | 각 0.2x | 대부분 MIT | 내장 언어. gpui-component 기능 `tree-sitter-languages` 또는 개별 기능 | 위 의존성 목록 |
| G-016 | `notify` | 8.2.x (gpui-component가 이미 의존) | CC0-1.0 | 폴더 트리·열린 파일·settings/keymap.json·`installed/` 감시 | [crates.io](https://crates.io/crates/notify) |
| G-017 | `trash` | 5.2.9 | MIT | Windows 휴지통 (I-552) | [crates.io](https://crates.io/crates/trash) |
| G-018 | `rfd` | 0.17.2 | MIT | 폴더·파일 열기, 다른 이름 저장, 확장 폴더·보관 파일 선택 (OS 대화상자) | [crates.io](https://crates.io/crates/rfd) |
| G-019 | `windows` | 0.62.2 | MIT/Apache-2.0 | TaskDialog 확인창(E-080, P-013), `ReplaceFileW`(A-24.1), 시스템 설정 조회 | [crates.io](https://crates.io/crates/windows) |
| G-020 | `serde_json` + `jsonc-parser` | 1.0.154 / 0.34.0 | MIT(/Apache) | 설정·키맵·테마 JSON. Zed 설정 파일은 주석·끝 쉼표를 허용 → JSONC 파서 필요 (판단) | [serde_json](https://crates.io/crates/serde_json), [jsonc-parser](https://crates.io/crates/jsonc-parser) |
| G-021 | `nucleo` | 0.5.0 | MPL-2.0 | Picker 퍼지 매칭 후보 (Zed는 자체 `fuzzy` 크레이트, GPL이라 쓰지 않는다) | [crates.io](https://crates.io/crates/nucleo) |
| G-022 | `wasmtime-c-api-impl` (tree-sitter `wasm` 기능이 간접으로 가져옴. **직접 의존하지 않는다**) | `^0.72.0` (tree-sitter 0.26.13 의존성 기준. wasmtime 본체 번호와의 대응은 미확인) | 미확인 (구현 때 확인) | 확장 문법 .wasm 로드. 이 크레이트의 버전은 **tree-sitter가 자체로 고정**한다. 앱이 `wasmtime`을 따로 넣으면 버전이 둘이 될 수 있어 주의한다 (G-108). 우리 범위(테마·아이콘·언어)에서는 `extension.wasm` 실행기가 필요 없다 (X-29) | [tree-sitter 0.26.13 의존성](https://crates.io/crates/tree-sitter/0.26.13/dependencies), [wasmtime-c-api-impl](https://crates.io/crates/wasmtime-c-api-impl) |
| G-023 | `dirs` | 7.0.0 | MIT/Apache-2.0 | `%APPDATA%`·`%LOCALAPPDATA%` 경로 (A-20, K-090, X-30) | [crates.io](https://crates.io/crates/dirs) |
| G-024 | 글꼴·아이콘 자산 | IBM Plex Sans, Lilex(OFL), Lucide(ISC), Zed 테마(MIT), Catppuccin 테마·아이콘(MIT) | 각각 | `.ZedSans`·`.ZedMono` 별칭 재현, 아이콘 SVG. Catppuccin·One Dark·One Light·Zed (Default) 아이콘은 **본체에 내장** (A-57.4) | 03-THEME-ICONS T-14~T-17 |
| G-025 | 보관 파일 해제 (후보, 미정) | `flate2`+`tar`, `zip` | MIT/Apache-2.0 | 확장 로컬 설치의 `.tar.gz`·`.zip` 해제 (X-43, 02 16.4절). 버전은 구현 때 확인 | G-124 |

## 4. 01-zed 항목 ↔ 구성 요소

기본 제공: **있음** = gpui-kit 컴포넌트로 거의 그대로 / **일부** = 컴포넌트는 있으나 모양·동작을 Zed에 맞춰 손봐야 함 / **직접** = 새로 만든다.

| ID | 영역 | 01-zed ID | 쓸 구성 요소 | 기본 제공 |
| --- | --- | --- | --- | --- |
| G-030 | 창·타이틀 바 | L-21, L-39, L-81~L-89, L-311, L-312, I-945 | `TitleBar` + gpui `WindowControlArea`(Drag/Min/Max/Close). 창 버튼 글리프(Segoe Fluent Icons)·닫기 `#E81120`·최대화/복원 아이콘·창 모서리 10px·테두리 1px는 Zed 방식 재현 | 일부 |
| G-031 | 사이드바(등록 폴더) | L-11, L-12, L-33~L-35, A-10~A-17 | `sidebar` 또는 `list`·`VirtualList` + `resizable` | 일부 |
| G-032 | 왼쪽 Dock | L-14, L-16, L-30~L-32, L-301~L-304 | `dock`(DockArea) 또는 `resizable` 패널. 6px 핸들·더블클릭 초기화·영역 경계선 | 일부 |
| G-033 | 프로젝트 트리 | L-36~L-52, L-305, I-087~I-097, P-019~P-022 | `tree`(Tree, TreeState 가상화). 접힌 경로(`docs \ guides`)·들여쓰기 안내선·인라인 이름 입력·검증 배너·드래그 | 일부 |
| G-034 | 탭 바·Pane 분할 | L-22~L-27, L-61~L-74, L-63, L-306, I-060~I-076, I-706~I-739, A-26 | `tab`(TabBar) + `dock`·`resizable`. 탭 테두리 규칙·미리보기 탭(기울임)·핀 영역·드래그 분할 영역·분할선 1px/4px·최소 80/100px는 직접 | 직접 (뼈대만 있음) |
| G-035 | 편집기 | E-001~E-059, E-090~E-093, A-04 | `input::Editor`·`EditorState`(Rope, tree-sitter `SyntaxHighlighter`). 거터 치수·현재 줄·커서 2px·선택 모서리·스크롤바 15px·클릭 횟수 선택·Alt 다중 커서·열 선택은 지원 범위 미확인 | 일부 (G-101) |
| G-036 | 찾기·바꾸기 창 | E-060~E-079 | 입력·IconButton·툴바 직접 조립. gpui-kit `input`의 검색 기능은 Zed 모양과 다름 | 직접 |
| G-037 | 마크다운 미리보기 | L-101~L-111, I-077~I-086 | `text::markdown()`·`TextView` + `MarkdownExtensions`. frontmatter 표·제목 밑줄·표 줄무늬·체크박스 토글·원문 동기화는 직접 | 일부 |
| G-038 | 상태 표시줄 | L-28, L-91~L-98, L-307, E-100~E-106 | `status_bar` 모듈 + IconButton | 일부 |
| G-039 | 메뉴 (☰·∨·우클릭) | I-300~I-488, I-600~I-655, I-500~I-562, I-570~I-591, I-660~I-668, I-900~I-952 | `menu`(PopupMenu·ContextMenu), `native_menu`. 최소 폭 200·75% 높이·단축키 표시·50ms 깜빡임·가로 메뉴 바 전환은 맞춤 | 일부 |
| G-040 | 툴팁 | L-200~L-272 | gpui 툴팁 + gpui-kit `Tooltip`. 500ms·마우스 기준 (+1,+1)·뒤집기·elevation_2는 gpui 동작과 같을 가능성이 높다 (G-111) | 있음·일부 |
| G-041 | Picker | P-001~P-012, E-103~E-105 | gpui-kit `command`(명령 팔레트) + `list`·`searchable_list`. Go to Line은 자체 모달 | 일부 |
| G-042 | 대화상자·About 창 | E-080~E-089, P-013~P-018 | Windows TaskDialog(`windows` 크레이트, Zed와 같음). About은 별도 부동 창 440×300. 모달은 `dialog` | 직접 (OS) |
| G-043 | 키맵 편집 화면 | K-001~K-097 | `table`(DataTable: 열 폭 조절·정렬·가상 스크롤) + 키 입력 녹화 컴포넌트 | 일부 (녹화 직접) |
| G-044 | 테마 | T-01~T-31, A-21 | `theme`(Theme, ThemeColor, ThemeRegistry). Zed 테마 JSON 키 → gpui-kit 토큰 매핑 표 필요 | 일부 (매핑 직접) |
| G-045 | 아이콘 테마 | T-20~T-29 | `Icon::path` + 아이콘 테마 JSON 해석기 | 직접 |
| G-046 | 확장 호스트 | A-46~A-57, X-01~X-43 | 직접: `extension.toml` 파서, 설치 폴더, 테마·아이콘·언어 등록, 문법 .wasm 로드. **로컬 설치만**, 끄기 없음 | 직접 |
| G-047 | 내장 뷰어 구조 | A-47 | `Item` 같은 트레이트(탭 제목·아이콘·툴팁·우클릭 추가 항목) + 파일 종류 → 뷰어 등록표. 첫 사례 마크다운 미리보기, 다음 이미지 | 직접 |
| G-048 | 설정·상태 파일 | A-20~A-25 | serde + 원자적 쓰기 + notify 재로드 | 직접 |
| G-049 | 확장 관리 화면 | X-44~X-56, L-308(스크롤바), 02 16절, A-58 | 일반 탭(`Item`) 안에 헤더·검색 상자·카테고리 바·카드 목록을 직접 조립(가상 목록). 폴더·파일 선택은 `rfd`. 카드 높이 110·padding 12 등 치수는 01-LAYOUT 4.15 | 직접 |
| G-050 | 다중 창 | A-43 (New Window) | 같은 프로세스에 `Window`를 더 연다. 창 상태 복원은 A-22 (G-113) | 일부 |

## 5. 상세 조사·검토 항목

구현 전에 PoC·문서·소스로 풀어야 할 질문이다. 우선순위: **P0** = 구현 전 필수, **P1** = 초기 단계, **P2** = 후순위. 이 절의 ID는 G-100 ~ G-129, G-139이고 G-130 ~ G-138은 6절의 위험 ID다.

| ID | 질문 | 왜 | 관련 01-zed ID | 확인 방법 | 우선순위 |
| --- | --- | --- | --- | --- | --- |
| G-100 | gpui-pre 0.3.10에서 **한글 IME**(조합 중 표시·후보창 위치·확정)가 gpui-kit Editor·Input·인라인 이름 입력·찾기 입력칸에서 제대로 되는가 | 한국어 문서 편집이 주 용도 | E-, I-091, K-055 | PoC: Editor에 한글 입력, 조합 중 커서·후보창 위치 캡처. Zed `gpui_windows` IME 처리와 비교 | P0 |
| G-101 | gpui-kit `Editor`가 Zed 편집기 동작을 어디까지 지원하나: 거터(줄 번호 최소 4자리), 현재 줄 강조 범위, 커서 2px·500ms 깜빡임, 선택 모서리 0.15×줄 높이, 더블·트리플 클릭, Alt+클릭 다중 커서, Ctrl+Shift 드래그 열 선택, 선택 끌어 옮기기, soft wrap, 들여쓰기 안내선, 공백 표시, 스크롤바 15px·마커 | 요건 4의 핵심. 부족하면 편집기 요소를 직접 써야 한다 | E-001~E-051 | docs.rs `input` 모듈 + 소스 읽기, PoC로 항목별 체크 | P0 |
| G-102 | 큰 파일 성능 (수 MB·10만 줄): Rope(ropey 2.0 beta) + tree-sitter 증분 파싱 + 가상 렌더링 | 편집기 반응성 | E-010, E-035 | PoC: 10만 줄 파일 열기·스크롤·입력 지연 측정 | P0 |
| G-103 | 커스텀 타이틀 바에서 **Windows 스냅 레이아웃**(최대화 버튼 hover), 더블클릭 최대화, 끌기 이동, 창 테두리 리사이즈, 최소 크기 360×240이 되는가. 최대화하지 않은 창의 **둥근 모서리 10px·테두리 1px**와 최대화 때 없어지는 처리 | 타이틀 바 재현 | L-21, L-39, L-81~L-84, L-311, L-312 | PoC: `WindowControlArea` Max에 hover → 스냅 플라이아웃 확인. Zed `crates/gpui_windows/src/window.rs` 대조 | P0 |
| G-104 | **탭 드래그·Pane 분할**: 탭을 끌어 순서 변경, 다른 Pane 이동, 가장자리 드롭으로 새 분할(판정 폭 20%·오버레이 50%), 분할선 끌기·더블클릭 균등화, Ctrl 드래그 복제 | 사용자 필수 요건 | I-074, I-712~I-722, A-26 | gpui-kit `dock`(DockArea·TabGroup 드래그) 지원 범위 확인 → PoC | P0 |
| G-105 | **키맵 컨텍스트 우선순위** 재현: 기본 + VSCode 덧씌움 + specific-overrides + 사용자 keymap.json, `unbind`, 컨텍스트 술어(`Editor && mode == full`, `>` 자손) | 키·메뉴·툴팁 표시가 모두 여기에 기댄다 | A-27, I-700~I-705, L-217, K-045, K-093 | gpui `KeyBindingContextPredicate` 공개 여부 확인(gpui-pre 재수출), JSON 로더 PoC | P0 |
| G-106 | **테마 토큰 매핑**: Zed 테마 JSON(`style.*` 수백 키)을 gpui-kit `ThemeColor`·`ThemeConfig`로 어떻게 옮기나. 우리 화면에만 쓰는 토큰(`editor.active_line.background` 등)은 별도 구조체가 필요한가 | 6개 테마(One·Catppuccin) 색 일치 판정 | T-01~T-26, A-31.8 | gpui-kit theme 소스·JSON 스키마 대조, 토큰 매핑 표 작성 | P0 |
| G-107 | **아이콘 테마** 적용: `file_stems`·`file_suffixes`·`named_directory_icons` 규칙과 SVG 렌더(색 덮어쓰기 포함, resvg) | 트리·탭 아이콘 일치 | T-20~T-29 | `Icon::path` SVG 렌더 PoC, Catppuccin Icons 패키지 하나로 시험 | P1 |
| G-108 | **Tree-sitter WASM 로더 (A-57.7)**: tree-sitter `wasm` 기능(`wasmtime-c-api-impl`을 자체 버전으로 고정해 가져옴, G-022)으로 `grammars/*.wasm`을 런타임에 불러와 gpui-kit `LanguageRegistry`에 등록할 수 있나. gpui-component가 고정한 tree-sitter 0.26 계열에서 `wasm` 기능을 켜도 강조기와 같은 버전을 공유하나. 별도 `wasmtime` 버전이 하나 더 생기지 않게 할 수 있나. 강조 쿼리(`highlights.scm`) 캡처 이름 → 테마 syntax 토큰 매핑 | 확장(언어) 지원 | A-46.7, A-57.7, X-29, X-40 | tree-sitter `WasmStore` 문서·Zed `language` 크레이트 대조, `cargo tree`로 wasmtime 계열 버전 중복 확인, `wasm` 기능을 켠 빌드의 크기·시간 측정, PoC로 Zed 확장 하나 로드 | P1 |
| G-109 | 확장 패키지 처리: `extension.toml` 파싱, 설치 폴더(`%LOCALAPPDATA%\{앱}\extensions\installed`), `index.json`, `installed/` 감시(100ms·200ms 디바운스), 테마·아이콘 테마 목록 갱신, 제거·재로드, 설치 직후 선택기 열기 | 앱 재빌드 없는 설치 | X-01~X-43, A-46 | zed-extensions.md의 경로·순서를 그대로 따를지 결정 | P1 |
| G-110 | **설정·키맵 JSON 감시**: settings.json·keymap.json을 탭에서 고쳐 저장하면 즉시 반영(테마·글꼴·키). JSONC 허용 | 설정 파일 열기 결정 | A-21, K-096 | notify + jsonc 파서 PoC | P1 |
| G-111 | gpui 툴팁·메뉴 기본 동작이 Zed와 같은지 (같은 gpui 계열이므로 대부분 같다고 예상): 500ms·(+1,+1)·뒤집기, `ContextMenu` 최소 폭 200·75% 높이 | 툴팁·메뉴 일치 | L-206~L-210, I-504~I-521 | gpui-pre 소스 대조 (Zed 소스와 차이 확인) | P1 |
| G-112 | 고DPI·배율 변경: 100·150·200% 및 창을 다른 배율 모니터로 옮길 때 rem·px·글꼴 선명도 | 기준 화면 200% 가정 | A-30.1, L-01 | PoC: 배율별 캡처, 타이틀 바 32px 확인 | P1 |
| G-113 | **상태 복원**: Pane 분할 트리·탭·임시·핀·Dock 폭·창 위치 저장·복원, 원자적 쓰기(`ReplaceFileW`) | 재시작 사용감 | A-22~A-25 | 직렬화 PoC, 비정상 종료 시험 | P1 |
| G-114 | 마크다운 미리보기: gpui-kit `markdown()`으로 frontmatter 2열 표, h1·h2 밑줄, 표 줄무늬·머리행 색, 인용 4px 선, 링크 밑줄, 체크박스 클릭 → 원문 수정, 원문 커서 → 블록 강조·스크롤이 가능한가 | 기준 화면의 주 내용 | L-101~L-111, I-077~I-086 | `MarkdownExtensions`·`MarkdownPlugin` API 확인, 기준 문서로 PoC 캡처 비교 | P0 |
| G-115 | 디스크 변경 감지·충돌: 깨끗한 버퍼 자동 다시 읽기, 수정 중이면 충돌 표시, 지워진 파일 취소선 | 저장 사용감 | E-084~E-087, A-44 | notify 이벤트 + mtime 비교 PoC | P1 |
| G-116 | Picker 공통(모달 위치·폭·퍼지 매칭 성능): 수만 개 파일 Go to File, Ctrl 누른 채 순환 | 파일 찾기 결정 포함 | P-001~P-009 | nucleo vs gpui-kit `searchable_list` 비교 PoC | P1 |
| G-117 | 키맵 편집 화면: gpui-kit `table`이 열 폭 재분배·줄무늬·가상 스크롤·행 선택 테두리를 지원하나, 키 녹화(Esc×3로 끝내기, 최대 3타) 입력 가로채기 | 키맵 편집 결정 포함 | K-010~K-097 | `table` 문서·소스, 키 입력 가로채기 PoC | P1 |
| G-118 | gpui-kit·gpui-pre 버전 고정과 업그레이드 경로: pre-1.0이라 API가 바뀐다. `=0.8.1` 고정 + Cargo.lock 커밋 | 장기 유지 | — | gpui-kit 릴리스 노트 추적 | P1 |
| G-119 | 접근성(AccessKit, Narrator)과 키보드만으로 모든 조작 | 품질 | L-05, I-517 | gpui-kit AccessKit 지원 범위 확인 | P2 |
| G-120 | 패키징·서명: MSI·MSIX 또는 설치 프로그램, 코드 서명(스마트 앱 컨트롤 환경), 글꼴·아이콘·내장 테마 자산 포함, 라이선스 고지 | 배포 | T-17, A-40.5 | cargo-packager 등 후보 비교, 서명 인증서 결정 | P2 |
| G-121 | 빌드 준비물: MSVC, Windows SDK, CMake(gpui-kit 설치 문서). Rust 최소 버전은 **두 값**이다: crates.io `rust_version` 1.90, 설치 문서 1.92(의존성 기준) | 출발 조건 | — | gpui-kit 설치 문서·crates.io 대조 ([gpui-kit.com](https://gpui-kit.com/docs/installation/), [crates.io](https://crates.io/crates/gpui-kit)). 기준 버전의 Rust 1.99.1은 둘 다 만족 | P2 |
| G-122 | 내장 뷰어 등록 구조: 파일 종류 → 뷰어 탭, 탭 아이콘·제목·툴팁·우클릭 추가 항목을 한 트레이트로 표현. 이미지 뷰어를 두 번째 사례로 | 확장 없이 뷰어를 늘리는 결정 | A-47 | 마크다운 미리보기·이미지 뷰어로 설계 PoC | P2 |
| G-123 | **개발용 문법 빌드와 링크 방식 (A-57.3)**: (a) 우리 앱이 문법 소스 빌드(`wasi-sdk` 내려받기·clang)를 지원할지, 아니면 02 16.5절의 판단대로 **미리 컴파일된 `grammars/*.wasm`만** 받을지 (b) `installed/<ID>`를 **심볼릭 링크**로 만들지 **디렉터리 정션**을 쓸지 — Windows에서 개발자 모드가 꺼져 있어도 되는지, 권한 오류 때 대체안 | 개발용 설치 화면의 동작과 도구 의존 | X-53~X-56, A-53.3, A-53.5, A-57.3 | PoC: 일반 사용자 권한에서 `CreateSymbolicLinkW`(비특권 플래그)와 정션(`mklink /J` 동등) 시험. 개발용 확장 하나(테마, 언어)로 감시 반영(200ms) 확인 | P1 |
| G-124 | 확장 **로컬 설치**(폴더·`.tar.gz`·`.zip`): 해제 크레이트 선택, 임시 폴더(`staging`) → `installed/<ID>` 원자적 이동, 같은 ID 교체, 제거 중 `work/<ID>`·열린 파일 잠금 재시도(0·100·200ms), 경로 탈출(`..`) 검증 | 설치·제거 안정성, 보안 | X-41, X-43, X-66, A-52, A-54 | 보관 파일 몇 개로 PoC(정상, `..` 포함, 매니페스트 없음), 잠금 시험 | P1 |
| G-125 | **TaskDialog 호출** (`windows` 크레이트): gpui 창을 부모로 두는 모달, Esc↔Cancel 연결, 기본 버튼, 포커스 복귀, 백틱이 글자 그대로 보이는 것. gpui-pre 자체에 Windows 프롬프트가 있는지. `TaskDialogIndirect`는 Common Controls v6 매니페스트가 있어야 쓸 수 있다 (G-128) | 저장·삭제·덮어쓰기 확인창 | E-080~E-085, P-013~P-016 | PoC: `TaskDialogIndirect` 3버튼 호출, gpui 이벤트 루프와 충돌 확인. `Window::prompt` 유무를 gpui-pre에서 확인. 근거 [MS Learn](https://learn.microsoft.com/windows/win32/api/commctrl/nf-commctrl-taskdialogindirect) | P1 |
| G-126 | **다중 창**(New Window)과 About 창: 같은 프로세스 상태 공유, 창별 Workspace, 마지막 창을 닫을 때 앱 종료·저장 확인, About 이중 열기 방지 | 새 창·About 포함 결정 | A-43, P-018, E-081 | PoC: 창 둘, 설정 변경(테마) 전파, 한 창 닫기 | P2 |
| G-128 | **앱 매니페스트**: Common Controls v6(TaskDialog 필수), per-monitor DPI v2, `longPathAware`를 누가 넣나. gpui-component의 `gpui-fast` 기능 `windows-manifest-provided-by-host`가 있어 매니페스트를 크레이트가 넣는지 앱이 넣는지가 설정에 걸려 있다. 앱 쪽 방법은 `embed-resource` 류 빌드 스크립트 후보 (미확인) | 매니페스트가 없으면 TaskDialog 호출이 실패하거나 배율·긴 경로 동작이 달라진다 | E-080~E-085, L-01, A-30.1, A-28 | gpui-component 0.8.1 기능 목록([의존성](https://crates.io/crates/gpui-component/0.8.1/dependencies))과 빌드 결과 exe의 매니페스트를 확인. 빈 창에서 TaskDialog 호출·150% 배율·260자 넘는 경로 열기 시험 | P0 |
| G-129 | **줄 끝(CRLF/LF) 보존과 인코딩**: CRLF 파일을 열고 저장해도 줄 끝이 그대로인가. UTF-8이 아닌 파일(UTF-16, CP949)을 어떻게 다룰지. Zed는 버퍼마다 줄 끝을 기억하고, 상태 표시줄에 인코딩 버튼(`active_encoding_button: "non_utf8"`)이 있다 | Windows 파일 보존. 저장 때 공백 제거·끝 줄바꿈 옵션(E-058, E-059)과도 얽힌다 | E-058, E-059, E-100~E-106, A-24 | gpui-kit `Editor`(Rope)가 줄 끝을 어떻게 저장하는지 확인. CRLF·UTF-16 LE·CP949 파일로 열기→저장→바이트 비교 PoC. 근거 [default.json status_bar](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2031-L2045) | P1 |
| G-139 | **한글 대체 글꼴**: `.ZedSans`(IBM Plex Sans)·`.ZedMono`(Lilex)에는 한글이 없어 DirectWrite 글꼴 대체가 줄 높이·기준선·폭을 바꿀 수 있다 | 한글 섞인 줄에서 줄 높이·거터·커서 위치가 Zed 기준 화면과 달라질 수 있다 (IME와 별개) | T-15.5, T-18.5, E-002, E-010, L-101 | DirectWrite 대체 글꼴(`Malgun Gothic` 등)을 적용한 한글 혼합 줄의 높이·거터·커서·마크다운 본문 측정 | P1 |
| G-127 | 내장 테마·아이콘 번들: Catppuccin 4종·One 2종·Zed (Default) 아이콘·Catppuccin Icons를 같은 JSON 파서로 읽고(내장 = 확장과 같은 레지스트리, X-57), MIT 고지 목록을 앱 안에서 보일 수 있는가 | 요건 7, 라이선스 | T-17~T-29, A-57.4, X-61 | 번들 에셋을 `include_dir` 류로 포함하고 설치된 확장과 이름 충돌 시 동작 확인 | P1 |

## 6. 위험 요소와 대응

| ID | 위험 | 영향 | 대응 (판단) |
| --- | --- | --- | --- |
| G-130 | gpui·gpui-kit pre-1.0, 잦은 API 변경 | 업그레이드마다 수정 | 버전 고정, 우리 쪽 얇은 래퍼 계층(ui 헬퍼)으로 gpui-kit 의존을 가둔다 |
| G-131 | gpui-kit Editor가 Zed 편집기 수준에 못 미침 | 요건 4 미달 | G-101 PoC를 맨 먼저. 모자라면 gpui `Element`로 편집기 요소를 직접 작성(가장 큰 작업) |
| G-132 | 한글 IME 문제 | 주 사용 불가 | G-100을 첫 주에 확인. 문제가 있으면 gpui-kit·gpui 저장소에 이슈, 우회 경로 조사 |
| G-133 | Zed GPL 코드 오염 | 라이선스 위반 | Zed 소스는 사실만 참고, 코드 복사 금지. gpui(Apache-2.0)·gpui-kit(Apache-2.0)만 의존. `fuzzy`·`editor` 등 Zed GPL 크레이트를 의존하지 않는다 |
| G-134 | 테마 토큰 불일치 | 색 일치 판정 실패 | 매핑 표(G-106)를 단일 원본으로 두고 자동 비교 스크립트로 검사 |
| G-135 | 탭 드래그·분할 구현량 | 일정 | gpui-kit `dock` 재사용 가능 범위를 먼저 확정. 부족하면 Zed 동작 명세(I-706~I-739)대로 직접 구현 |
| G-136 | tree-sitter `wasm` 기능이 가져오는 wasmtime 계열의 크기·빌드 시간, 그리고 앱이 `wasmtime`을 따로 넣을 때 생기는 버전 중복 (G-022) | 실행 파일 비대, 빌드 시간 | 확장 문법 지원을 기능 플래그로 분리. 내장 문법은 정적 링크. 별도 `wasmtime`은 넣지 않는다 |
| G-137 | 확장 폴더 링크·잠금 문제 (G-123, G-124) | 개발용 설치 실패, 제거 실패 | 링크 대체안(정션·복사)과 제거 재시도를 정하고, 오류 문구는 02 16.6절의 "조사하지 않은 것"으로 둔다 |
| G-138 | 확장 패키지가 `[language_servers]`·`extension.wasm`을 가진 경우 | 설치 실패로 오해 | 해당 필드를 **오류 없이 무시**하고 `languages/*`·`grammars/*.wasm`만 쓴다 (X-29) |

**확신 낮은 부분**

- **gpui-kit Editor 지원 범위 (G-035, G-101):** docs.rs 모듈 목록과 의존성만 보고 판단했다. 거터·다중 커서·열 선택 등 세부 지원은 미확인이다.
- **dock 드래그 분할 (G-034, G-104):** gpui-kit `dock`이 Zed식 Pane 분할(가장자리 드롭으로 새 분할)을 지원하는지 미확인이다.
- **tree-sitter wasm 기능 (G-014, G-108):** gpui-component가 `wasm` 기능 없이 tree-sitter를 쓴다는 것은 crates.io 의존성 정보로 확인했다. 우리가 기능을 켜도 gpui-component 강조기와 함께 쓸 수 있는지는 미확인이다.
- **wasmtime-c-api-impl (G-022):** `^0.72.0`은 crates.io 의존성 응답을 그대로 읽은 값이다. 이 번호가 wasmtime 본체 번호와 어떻게 대응하는지는 확인하지 않았다. "tree-sitter가 자체로 고정한다"는 결론은 바뀌지 않는다.
- **TaskDialog (G-042, G-125):** gpui-pre가 Zed처럼 Windows 프롬프트를 TaskDialog로 구현했는지 미확인이다 (Zed 소스 `gpui_windows`에서는 확인).
- **앱 매니페스트 (G-128):** gpui-kit 앱의 기본 매니페스트에 Common Controls v6가 들어 있는지는 확인하지 않았다. 확인 항목으로 둔 것이다.
- **nucleo 라이선스:** MPL-2.0이다. 파일 단위 카피레프트라 정적 링크 배포에는 문제없다고 보지만 판단이다.
- **확장 설치 경로 (G-109):** zed-extensions.md X-30~X-35를 따른다는 전제이고, 앱 이름으로 바꾼 경로는 04-APP-SPEC가 정한다.
- **버전:** crates.io 최신값은 조회 시점(2026-10-31) 기준이다. gpui-kit가 고정한 하위 의존성(tree-sitter 0.26 등)과 최신값이 다를 수 있다.
- **보관 파일 크레이트 (G-025):** 후보일 뿐 버전·기능을 확인하지 않았다.

## 7. 시작 순서 (PoC 단계)

각 단계는 "판정표에서 직접 구현 범위가 정해졌다"를 끝으로 본다. 코드는 구현 프로젝트에서 쓴다.

| 단계 | 내용 | 확인할 항목 | 끝났다고 보는 기준 |
| --- | --- | --- | --- |
| 1 | 빈 창 + 커스텀 타이틀 바 + 창 버튼 (최대화/복원 아이콘, 둥근 모서리·테두리), 앱 매니페스트 | G-103, G-112, G-128 | 스냅·더블클릭 최대화·32px 확인 |
| 2 | gpui-kit Editor PoC: 한글 IME, 한글 대체 글꼴, 큰 파일, 선택·커서 동작, 줄 끝·인코딩 | G-100, G-101, G-102, G-129, G-139 | 직접 구현 범위 확정 |
| 3 | 레이아웃 뼈대: 사이드바·Dock·Pane·탭 바·상태 표시줄, rem 헬퍼 | L-11~L-40, A-36.1 | 치수 점검표 A-32 ±1px |
| 4 | 키맵 로더 + 액션·컨텍스트 + 툴팁·메뉴 단축키 표시 | G-105, G-111 | VSCode 덧씌움 결과로 메뉴·툴팁 키 일치 |
| 5 | 폴더 등록·트리·파일 작업·감시 | A-10~A-17, I-087~I-097, G-115 | 시나리오 A-33.2, A-33.4 |
| 6 | 탭·미리보기 탭·핀·분할·드래그, 마크다운 미리보기 | G-104, G-114 | 시나리오 A-33.3, A-33.7 |
| 7 | 테마·아이콘 테마 매핑과 선택기, 내장 테마 번들 | G-106, G-107, G-127 | 6개 테마 색 비교 통과 |
| 8 | 찾기 창·Picker·키맵 편집·확장(로컬 설치·개발용 설치·문법 로드)·확인창·상태 복원 | G-108~G-117, G-123~G-127 | 일치도 판정 A-35 |

## 8. 참고 링크

| 구분 | 링크 |
| --- | --- |
| gpui-kit | [gpui-kit.com](https://gpui-kit.com/), [설치](https://gpui-kit.com/docs/installation/), [crates.io](https://crates.io/crates/gpui-kit), [GitHub](https://github.com/longbridge/gpui-kit) |
| 컴포넌트 문서 | [docs.rs gpui-component 0.8.1](https://docs.rs/gpui-component/0.8.1/gpui_component/), [gpui-base 의존성](https://crates.io/crates/gpui-base/0.8.1/dependencies) |
| gpui | [gpui.rs](https://www.gpui.rs/), [gpui README (Zed 저장소)](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/README.md), [gpui-pre](https://crates.io/crates/gpui-pre) |
| Zed 소스 (읽기만, GPL) | [커밋 14b7b0e](https://github.com/zed-industries/zed/tree/14b7b0ef3764d62a85070c8b5ccea105a2ad3235). 사실 참고용이고 코드는 복사하지 않는다 |
| 크레이트 | [tree-sitter](https://crates.io/crates/tree-sitter), [notify](https://crates.io/crates/notify), [trash](https://crates.io/crates/trash), [rfd](https://crates.io/crates/rfd), [windows](https://crates.io/crates/windows), [jsonc-parser](https://crates.io/crates/jsonc-parser), [nucleo](https://crates.io/crates/nucleo), [wasmtime-c-api-impl](https://crates.io/crates/wasmtime-c-api-impl), [dirs](https://crates.io/crates/dirs) |
| 이 묶음의 명세 | [01-LAYOUT](../01-zed/01-LAYOUT.md), [02-INTERACTION](../01-zed/02-INTERACTION.md), [03-THEME-ICONS](../01-zed/03-THEME-ICONS.md), [04-APP-SPEC](../01-zed/04-APP-SPEC.md) |
| 선행 문서 | [gpui + gpui-kit 입문](../../rust-gui/02-GPUI.md) |

---

이전: [../01-zed/04-APP-SPEC.md](../01-zed/04-APP-SPEC.md) · 다음: [../03-tauri/README.md](../03-tauri/README.md) · 입구: [../README.md](../README.md)
