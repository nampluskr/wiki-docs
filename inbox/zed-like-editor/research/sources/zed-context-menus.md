# Zed 우클릭 컨텍스트 메뉴 조사 (ID 접두사 I-, I-500부터)

- 담당: scouter-opus · 조사일 2026-10-31
- 소스 기준: zed-industries/zed 커밋 `14b7b0ef3764d62a85070c8b5ccea105a2ad3235` (링크는 모두 이 커밋에 고정). GPL 코드 원문은 옮기지 않고 **항목 문구·액션 이름·단축키·구분선 위치·조건**만 적었다.
- 단축키는 Windows 기본 키맵 `assets/keymaps/default-windows.json` 기준이고, 표기는 Zed 툴팁·메뉴 표기 규칙(수식키 순서 Ctrl → Win → Alt → Shift, 구분자 `-`)을 따른다 (zed-tooltips.md L-219). 한 액션에 키가 여러 개면 **메뉴가 속한 포커스 컨텍스트에서 마지막에 정의된 키**를 적었다 (L-217 규칙 적용, 추정).
- `─` 표시는 구분선(separator)이다.
- 판정: **유지** = 우리 앱에 넣는다 / **제외** = 범위 밖 (Git, 터미널, AI, 협업, 디버그, LSP, 원격, 확장, 검색, diff) / **판단 필요** = 우리 앱 모델에 맞춰 바꾸거나 결정해야 한다.

---

## 1. 컨텍스트 메뉴 공통 명세

### 1.1 컴포넌트와 열리는 위치

| ID | 항목 | 내용 | 출처 |
|---|---|---|---|
| I-500 | 메뉴 본체 `ContextMenu` | 항목 종류: action 항목, entry(직접 핸들러), 비활성 항목(`action_disabled_when`, `ContextMenuEntry::disabled`), 토글 항목(`toggleable_entry`: 체크 아이콘), 구분선, 헤더(`header`), 하위 메뉴(`submenu`), custom entry | [crates/ui/src/components/context_menu.rs#L497-L860](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L497-L860) |
| I-501 | 우클릭 래퍼 `right_click_menu(id)` | 요소를 우클릭하면 메뉴를 띄운다. 기본 위치는 **마우스 커서 위치**에서 메뉴의 왼쪽 위 모서리. `anchor`(메뉴의 어느 모서리를 맞출지), `attach`(요소의 어느 모서리에 붙일지)로 바꿀 수 있다. 창 밖으로 나가지 않게 **창 가장자리에서 8px 여백**을 두고 밀어 넣는다(`snap_to_window_with_margin`) | [crates/ui/src/components/right_click_menu.rs#L43-L52](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/right_click_menu.rs#L43-L52), [right_click_menu.rs#L146-L152](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/right_click_menu.rs#L146-L152) |
| I-502 | 프로젝트 패널 메뉴 위치 | 마우스로 열면 **마우스 위치**. 키보드(`project_panel::OpenContextMenu`: Menu 키, Shift-F10)로 열면 **선택 항목 옆**에 (2, 2)px 띄워서 연다 (아래쪽에 붙을 때는 −2) | [crates/project_panel/src/project_panel.rs#L1317-L1336](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1317-L1336), [project_panel.rs#L203](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L203), [default-windows.json#L1043-L1044](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1043-L1044) |
| I-503 | 편집기 메뉴 위치 | 클릭한 위치에 고정(`pinned_to_editor`). 클릭 지점이 기존 선택 영역 밖이면 **먼저 커서를 그 위치로 옮긴다** | [crates/editor/src/mouse_context_menu.rs#L160-L200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/mouse_context_menu.rs#L160-L200), [mouse_context_menu.rs#L338-L346](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/mouse_context_menu.rs#L338-L346) |

### 1.2 모양

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| I-504 | 바탕 | `elevation_2`: 배경 `elevated_surface_background`, 모서리 `rounded_lg`, 테두리 1px `border_variant`, 그림자 2겹 (툴팁과 같음, zed-tooltips.md L-211, L-212) | [context_menu.rs#L2270-L2276](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2270-L2276) |
| I-505 | 글꼴 | UI 글꼴, 메뉴 안 rem = `ui_font_size`(`WithRemSize`), 줄 높이 **1.618**(`BufferLineHeight::Comfortable`, 편집기의 줄 높이 설정과 무관하게 고정) | [context_menu.rs#L2219-L2226](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2219-L2226) |
| I-506 | 크기 | **최소 폭 200px**(`CONTEXT_MENU_MIN_WIDTH`, 고정 폭 지정이 없을 때), **최대 높이 = 창 높이의 75%**, 넘치면 세로 스크롤 | [context_menu.rs#L22](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L22), [context_menu.rs#L2284-L2293](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2284-L2293) |
| I-507 | 안쪽 여백 | 목록(`List`) 위아래 Base04 = **4px**. 항목은 `inset` 모드: 항목 바깥 좌우 Base04 = 4px, 항목 안 좌우 Base06 = 6px | [crates/ui/src/components/list/list.rs#L80-L82](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list.rs#L80-L82), [list_item.rs#L306-L310, L363-L364](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L306-L310) |
| I-508 | 항목 높이 | 명시값 없음. Dense 간격(추가 padding 0) + 라벨 14px × 1.618 ≈ 22.6 + inset 테두리 1px×2 ≈ **약 25px** (추정) | [context_menu.rs#L1923-L1930](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1923-L1930), [list_item.rs#L365-L368](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L365-L368) |
| I-509 | 항목 배치 | [토글 체크 아이콘(시작 위치, 꺼져 있으면 자리만 차지)] [라벨] … [단축키]. 라벨과 단축키는 양끝 정렬, 단축키 앞 여백 `ml_4` = 16px. 하위 메뉴 항목은 오른쪽 끝에 `ChevronRight`(Muted) | [context_menu.rs#L2008-L2050](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2008-L2050), [context_menu.rs#L1682-L1684](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1682-L1684) |
| I-510 | 단축키 표시 | 메뉴에 컨텍스트(포커스)가 지정돼 있으면 그 컨텍스트 기준(`for_action_in`)으로, 아니면 전역으로 찾는다. 비활성 항목은 단축키도 Disabled 색. 표시 형식은 툴팁과 같음 (Muted, `Ctrl-X`) | [context_menu.rs#L2034-L2040](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2034-L2040) |
| I-511 | 아이콘 크기 | 토글 체크 등 항목 아이콘 기본 `IconSize::Small`(14px) | [context_menu.rs#L114](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L114) |
| I-512 | 구분선 | 높이 1px, 위아래 margin Base06 = **6px**, 색 `border_variant` | [crates/ui/src/components/list/list_separator.rs#L9-L12](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_separator.rs#L9-L12) |
| I-513 | 헤더 | `LabelSize::Small`, 색 Muted (예: 사이드바 메뉴의 "Open Worktrees") | [context_menu.rs#L1418-L1425](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1418-L1425) |
| I-514 | 색 | 라벨: 일반 `Default`, 비활성 **`Disabled`** / 아이콘: 비활성 Muted, 토글 Accent / hover 배경 `ghost_element_hover`, 누름 `ghost_element_active`, 키보드 선택 `ghost_element_selected`, 키보드 포커스 테두리 `border_focused` | [context_menu.rs#L1812-L1824](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1812-L1824), [list_item.rs#L370-L385](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L370-L385) |
| I-515 | 비활성 항목 | 회색(Disabled) 라벨, hover·클릭 반응 없음, 키보드 선택에서도 건너뛴다 | [list_item.rs#L379](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L379), [context_menu.rs#L1098-L1135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1098-L1135) |

### 1.3 키보드·닫힘

| ID | 항목 | 내용 | 출처 |
|---|---|---|---|
| I-516 | 키 컨텍스트 | 메뉴 키 컨텍스트 이름은 `menu` | [context_menu.rs#L357](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L357) |
| I-517 | 키 바인딩 (Windows) | 다음: Down, Tab, Ctrl-N / 이전: Up, Shift-Tab, Ctrl-P / 처음: Home, PageUp, Shift-PageUp / 끝: End, PageDown, Shift-PageDown / 실행: **Enter** / 닫기: **Escape**, Ctrl-C / 하위 메뉴 열기: Right / 상위로: Left | [default-windows.json#L6-L21](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L6-L21), [default-windows.json#L43-L55](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L43-L55) |
| I-518 | 처음 선택 | 키보드로 처음 아래 키를 누르면 첫 번째 선택 가능한 항목을 고른다. 끝에서 다음으로 가면 처음으로 순환 | [context_menu.rs#L1098-L1135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1098-L1135) |
| I-519 | 닫히는 조건 | 메뉴 밖을 마우스로 누를 때(`on_mouse_down_out`), Esc(`menu::Cancel`), 항목 실행 후(`DismissEvent`), 메뉴가 포커스를 잃을 때(하위 메뉴로 포커스가 간 경우는 제외) | [context_menu.rs#L2315-L2345](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2315-L2345), [context_menu.rs#L899-L947](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L899-L947) |
| I-520 | 단축키로 실행 시 깜빡임 | 메뉴가 열린 채로 메뉴 항목의 단축키를 누르면, 해당 항목을 **선택 표시한 뒤 50ms 후** 메뉴를 닫고 액션을 실행한다 | [context_menu.rs#L1350-L1385](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1350-L1385) |
| I-521 | 하위 메뉴 | 마우스 hover 또는 Right/Enter로 열린다. 하위 메뉴가 열릴 때 부모 메뉴가 150~200ms 동안 blur를 무시해 닫히지 않게 한다 | [context_menu.rs#L1015-L1020](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1015-L1020), [context_menu.rs#L1338](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1338) |

---

## 2. 영역별 우클릭 메뉴 (Zed 원본 전체)

### 2.1 프로젝트 패널 — 파일·폴더·루트 (읽기·쓰기 가능한 로컬 프로젝트)

출처: [project_panel.rs#L1128-L1312](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1128-L1312). 단축키는 `ProjectPanel` 컨텍스트([default-windows.json#L1009-L1047](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1009-L1047)) 기준이다. 메뉴의 컨텍스트가 프로젝트 패널 포커스로 지정된다.

| ID | 순서 | 항목 문구 | 액션 | 단축키 | 조건 | 판정 |
|---|---|---|---|---|---|---|
| I-530 | 1 | New File | `project_panel::NewFile` | Ctrl-N | 항상 | 유지 |
| I-531 | 2 | New Folder | `project_panel::NewDirectory` | Alt-N | 항상 | 유지 |
| | ─ | | | | | |
| I-532 | 3 | Reveal in File Explorer (macOS "Reveal in Finder", 그 외 "Reveal in File Manager") | `project_panel::RevealInFileManager` | Ctrl-Alt-R | 로컬 프로젝트 | 유지 |
| I-533 | 4 | Open in Default App | `project_panel::OpenWithSystem` | Ctrl-Shift-Enter | 로컬 프로젝트 | 유지 |
| I-534 | 5 | Open in Terminal | `project_panel::OpenInTerminal` | — | 항상 | **제외** (터미널) |
| I-535 | 6 | Open Markdown Preview | `project_panel::OpenMarkdownPreview` | — | 마크다운 파일 | 유지 |
| I-536 | 7 | Open SVG Preview | `project_panel::OpenSvgPreview` | — | SVG 파일 | 제외 (SVG 미리보기는 범위 밖, 판단) |
| | ─ | | | | 폴더일 때만 이 구분선 | |
| I-537 | 8 | Find in Folder… | `project_panel::NewSearchInDirectory` | Ctrl-K Ctrl-Shift-F | 폴더 | **제외** (프로젝트 검색 없음) |
| I-538 | 9 | Unfold Directory | `project_panel::UnfoldDirectory` | — | `auto_fold_dirs` 켜짐 + 접혀 있는 경로 | 유지 |
| I-539 | 10 | Fold Directory | `project_panel::FoldDirectory` | — | `auto_fold_dirs` 켜짐 + 접을 수 있는 폴더 | 유지 |
| | ─ | | | | 비교 가능할 때만 | |
| I-540 | 11 | Compare Marked Files | `project_panel::CompareMarkedFiles` | Alt-D | 파일 + 비교 대상 표시됨 | **제외** (diff) |
| | ─ | | | | | |
| I-541 | 12 | Cut | `project_panel::Cut` | Ctrl-X | 항상 | 유지 |
| I-542 | 13 | Copy | `project_panel::Copy` | Ctrl-C | 항상 | 유지 |
| I-543 | 14 | Duplicate | `project_panel::Duplicate` | — | 항상 | 유지 |
| I-544 | 15 | Paste | `project_panel::Paste` | Ctrl-V | 붙여 넣을 것이 없으면 **비활성** | 유지 |
| I-545 | 16 | Undo | `project_panel::Undo` | Ctrl-Z | 협업 아님. 되돌릴 것이 없으면 비활성 | 판단 필요 (파일 작업 실행 취소 스택을 만들지) |
| I-546 | 17 | Redo | `project_panel::Redo` | Ctrl-Shift-Z | 협업 아님. 다시 할 것이 없으면 비활성 | 판단 필요 (위와 같음) |
| | ─ | | | | 원격일 때만 | |
| I-547 | 18 | Download... | `project_panel::DownloadFromRemote` | — | 원격 프로젝트 | **제외** (원격) |
| | ─ | | | | | |
| I-548 | 19 | Copy Path | `workspace::CopyPath` | Alt-Shift-C | 항상 | 유지 |
| I-549 | 20 | Copy Relative Path | `workspace::CopyRelativePath` | Ctrl-K Ctrl-Shift-C | 항상 | 유지 |
| | ─ | | | | Git 저장소 안일 때만 | |
| I-550 | 21~26 | Restore File / Add to .gitignore / Add to .git/info/exclude / View History / Open File Permalink / Copy File Permalink | `git::…` | — | Git 저장소 (일부는 파일만, 변경 있음, 기록 있음) | **제외** (Git) |
| | ─ | | | | 이름 변경을 보일 때 | |
| I-551 | 27 | Rename | `project_panel::Rename` | F2 | **Windows에서는 루트 항목이면 숨김**. 다른 OS는 `hide_root`이고 폴더가 하나일 때 숨김 | 유지 |
| I-552 | 28 | Trash | `project_panel::Trash {skip_prompt:false}` | Delete | 루트 아님 + 협업 아님 | 유지 (휴지통으로) |
| I-553 | 29 | Delete | `project_panel::Delete {skip_prompt:false}` | Ctrl-Delete | 루트 아님 | 유지 (영구 삭제, 확인 대화상자) |
| | ─ | | | | 루트일 때만 | |
| I-554 | 30 | Add Folders to Project… | `workspace::AddFolderToProject` | — (Workspace 컨텍스트에 바인딩 없음) | 루트 + 협업 아님 | 판단 필요 (우리 앱은 폴더 등록을 사이드바에서 함) |
| I-555 | 31 | Remove from Project | `project_panel::RemoveFromProject` | — | 루트 + 협업 아님 | 판단 필요 (사이드바의 "폴더 등록 해제"로 연결할지) |
| | ─ | | | | 폴더일 때 | |
| I-556 | 32 | Expand All | 루트: `project_panel::ExpandAllEntries` / 하위 폴더: `ExpandSelectedEntryAndChildren` | 루트: Ctrl-Right / 하위: — | 폴더 | 유지 |
| I-557 | 33 | Collapse All | 루트: `project_panel::CollapseAllEntries` / 하위 폴더: `CollapseSelectedEntryAndChildren` | 루트: Ctrl-Left / 하위: — | 폴더 | 유지 |

추가 규칙

| ID | 내용 | 출처 |
|---|---|---|
| I-558 | **읽기 전용 프로젝트**에서는 메뉴가 줄어든다: [Open Markdown Preview(md)] [Open SVG Preview(svg)] [Search Inside(폴더)]만 | [project_panel.rs#L1189-L1200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1189-L1200) |
| I-559 | **빈 공간**(마지막 항목 아래) 우클릭 = **마지막 워크트리의 루트 항목**을 우클릭한 것과 같은 메뉴 | [project_panel.rs#L8076-L8090](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8076-L8090) |
| I-560 | 우클릭한 항목이 선택(selection)이 된다. 메뉴는 선택 항목에 대해 동작한다 | [project_panel.rs#L1143-L1146](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1143-L1146) |
| I-561 | 접힌 경로(`docs \ guides`)의 조각을 우클릭하면, 그 조각을 활성 단계로 바꾼 뒤 메뉴를 연다 (어느 폴더에 대한 메뉴인지가 조각에 따라 달라짐) | [project_panel.rs#L6979-L6990](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6979-L6990) |
| I-562 | 여러 항목을 표시(marked)한 상태에서도 메뉴 구성은 같다. 다만 Cut, Copy, Trash, Delete 등은 표시된 모든 항목에 적용된다 (추정) | [project_panel.rs#L150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L150) |

### 2.2 탭 바 — 탭

출처: [crates/workspace/src/pane.rs#L3136-L3500](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3136-L3500). 메뉴 컨텍스트는 **그 탭 아이템의 포커스**라서, 편집기 탭이면 Editor·Pane·Workspace 컨텍스트 키가 보인다.

| ID | 순서 | 항목 문구 | 액션 | 단축키 | 조건 | 판정 |
|---|---|---|---|---|---|---|
| I-570 | 1 | Close | `pane::CloseActiveItem {close_pinned: true}` | (키맵의 Ctrl-W는 `close_pinned:false`라 인자가 달라 표시되지 않을 수 있음, ⑥) | 항상 | 유지 |
| I-571 | 2 | Close Others | `pane::CloseOtherItems {close_pinned:false}` | Ctrl-Shift-Alt-T (표기: Ctrl-Alt-Shift-T) | 탭이 하나뿐이면 비활성 | 유지 |
| I-572 | 3 | Close Multibuffers | `pane::CloseMultibufferItems` | — | 멀티버퍼가 있을 때만 (없으면 아예 안 보임) | 제외 (멀티버퍼 없음) |
| | ─ | | | | | |
| I-573 | 4 | Close Left | `pane::CloseItemsToTheLeft {close_pinned:false}` | Ctrl-K E | 왼쪽에 탭이 없으면 비활성 | 유지 |
| I-574 | 5 | Close Right | `pane::CloseItemsToTheRight {close_pinned:false}` | Ctrl-K T | 오른쪽에 탭이 없으면 비활성 | 유지 |
| | ─ | | | | | |
| I-575 | 6 | Close Clean | `pane::CloseCleanItems {close_pinned:false}` | Ctrl-K U | 저장된(깨끗한) 탭이 없으면 비활성 | 유지 |
| I-576 | 7 | Close All | `pane::CloseAllItems {close_pinned:false}` | Ctrl-K W | 항상 | 유지 |
| | ─ | | | | | |
| I-577 | 8 | Pin Tab / Unpin Tab | `pane::TogglePinTab` | Ctrl-K Shift-Enter | 고정 안 됨 → "Pin Tab", 고정됨 → "Unpin Tab" | 판단 필요 (고정 탭 기능을 넣을지) |
| | ─ | | | | 아래 항목이 있을 때 | |
| I-578 | 9 | Make Tab Read-Only / Make Tab Editable | (핸들러, 액션 없음) | — | 아이템이 읽기 전용 전환을 지원할 때. 편집 가능 → "Make Tab Read-Only", 읽기 전용 → "Make Tab Editable" | 판단 필요 |
| | ─ | | | | 파일 탭일 때 | |
| I-579 | 10 | Copy Path | `workspace::CopyPath` | Ctrl-K P (Editor 컨텍스트의 `editor::CopyPath` 별칭) | 절대 경로가 있을 때 | 유지 |
| I-580 | 11 | Copy Relative Path | `workspace::CopyRelativePath` | — (Editor 컨텍스트에 바인딩 없음, 추정) | 워크트리 루트가 폴더일 때 | 유지 |
| | ─ | | | | Git일 때 | |
| I-581 | 12~13 | Open File Permalink / Copy File Permalink | `git::…` | — | Git 저장소 | **제외** (Git) |
| | ─ | | | | 로컬일 때 | |
| I-582 | 14 | Reveal in File Explorer | `editor::RevealInFileManager` | Ctrl-K R | 로컬 + 경로 있음 | 유지 |
| I-583 | 15 | Reveal In Project Panel | `pane::RevealInProjectPanel` | Ctrl-Shift-E (편집기 포커스 시, `!AcpThread > Editor && mode == full`) | 프로젝트 패널에 보이는 파일 | 유지 |
| I-584 | 16 | Open in Terminal | `workspace::OpenInTerminal` | — | 상위 폴더가 있을 때 | **제외** (터미널) |
| | ─ | | | | 추가 항목이 있을 때 | |
| I-585 | 17 | (편집기, 마크다운 파일) Open Markdown Preview | `markdown::OpenPreview` | Ctrl-Shift-V (`Editor && extension == md`) | 언어가 Markdown | 유지 |
| I-586 | 17 | (편집기, SVG 파일) Open SVG Preview | `svg::OpenPreview` | — | SVG | 제외 (판단) |
| I-587 | 17 | (**마크다운 미리보기 탭**) Show Source | `markdown::CloseAndReturnToEditor` | Ctrl-Shift-V (`MarkdownPreview` 컨텍스트) | 미리보기 탭 | 유지 |

탭 메뉴 추가 규칙

| ID | 내용 | 출처 |
|---|---|---|
| I-588 | **미리보기(italic) 탭**도 메뉴 구성은 같다. 미리보기 전용 항목은 없다 | [pane.rs#L3136-L3150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3136-L3150) |
| I-589 | **마크다운 미리보기 탭**은 Copy Path 이하 파일 항목이 나오는지 미확인 (미리보기 아이템이 프로젝트 항목을 갖지 않으면 빠짐, 추정). 끝에 "Show Source"가 붙는 것은 확인 | [markdown_preview_view.rs#L1757-L1763](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1757-L1763) |
| I-590 | **탭 바 빈 공간**: 우클릭 메뉴 **없음** (pane.rs의 `right_click_menu`는 탭에만 있음) | [pane.rs#L3138](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3138) |
| I-591 | 탭 바 오른쪽 + · 분할 버튼은 **왼쪽 클릭 메뉴**(우클릭 아님): + = New File, Open File, ─, Search Project, Search Symbols, ─, New Terminal, New Center Terminal / 분할 = Split Right, Split Left, Split Up, Split Down. 우리 앱의 +는 New File만 남긴다 (판단) | [pane.rs#L4364-L4412](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4364-L4412) |

### 2.3 편집기 본문 (텍스트 편집기)

출처: [mouse_context_menu.rs#L202-L337](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/mouse_context_menu.rs#L202-L337). 전체 모드 편집기에 프로젝트가 있을 때만 나온다.

| ID | 순서 | 항목 문구 | 액션 | 단축키 | 조건 | 판정 |
|---|---|---|---|---|---|---|
| I-600 | 1 | Run to Cursor | `debugger::RunToCursor` | — | 디버거 실행 중 | **제외** (디버그) |
| I-601 | 2 | Evaluate Selection | `debugger::EvaluateSelectedText` | — | 디버거 + 선택 있음 | **제외** (디버그) |
| | ─ | | | | 위 둘 중 하나라도 있을 때 | |
| I-602 | 3~9 | Go to Definition / Go to Declaration / Go to Type Definition / Go to Implementation / Find All References / Show Incoming Calls / Show Outgoing Calls | `editor::GoTo…`, `editor::FindAllReferences` 등 | F12, …, Shift-Alt-F12 등 | 항상 | **제외** (LSP) |
| | ─ | | | | | |
| I-603 | 10 | Rename Symbol | `editor::Rename` | F2 | 항상 | **제외** (LSP) |
| I-604 | 11 | Format Buffer | `editor::Format` | Alt-Shift-F | 항상 | **제외** (LSP·포매터 의존, 판단) |
| I-605 | 12 | Format Selections | `editor::FormatSelections` | — | 사용 가능할 때 | **제외** |
| I-606 | 13 | Show Code Actions | `editor::ToggleCodeActions` | Ctrl-. | 항상 | **제외** (LSP) |
| I-607 | 14 | Add to Agent Thread | `agent::AddSelectionToThread` | — | AI 켜짐 + 선택 있음 | **제외** (AI) |
| | ─ | | | | | |
| I-608 | 15 | Cut | `editor::Cut` | Ctrl-X | 항상 | 유지 |
| I-609 | 16 | Copy | `editor::Copy` | Ctrl-C | 항상 | 유지 |
| I-610 | 17 | Copy and Trim | `editor::CopyAndTrim` | — | 항상 | 유지 (판단: 줄 앞 공통 들여쓰기를 빼고 복사) |
| I-611 | 18 | Paste | `editor::Paste` | Ctrl-V | 항상 | 유지 |
| | ─ | | | | | |
| I-612 | 19 | Reveal in File Explorer | `editor::RevealInFileManager` | Ctrl-K R | 파일이 없으면 **비활성** | 유지 |
| I-613 | 20 | Open Markdown Preview | `markdown::OpenPreview` | Ctrl-Shift-V | 마크다운 버퍼 | 유지 |
| I-614 | 21 | Open SVG Preview | `svg::OpenPreview` | — | SVG | 제외 (판단) |
| I-615 | 22 | Open in Terminal | `workspace::OpenInTerminal` | — | 파일이 없으면 비활성 | **제외** (터미널) |
| I-616 | 23 | Copy Permalink to Line | `editor::CopyPermalinkToLine` | — | Git이 없으면 비활성 | **제외** (Git) |
| I-617 | 24 | View File History | `git::FileHistory` | — | Git이 없으면 비활성 | **제외** (Git) |

### 2.4 마크다운 미리보기 본문

출처: [markdown_preview_view.rs#L1960-L2012](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1960-L2012)

| ID | 순서 | 항목 문구 | 액션 | 단축키 | 조건 | 판정 |
|---|---|---|---|---|---|---|
| I-620 | 1 | Copy | `markdown::Copy` | Ctrl-C (`Markdown` 컨텍스트) | 선택한 텍스트가 있을 때 | 유지 |
| I-621 | 2 | Copy as Markdown | `markdown::CopyAsMarkdown` | — | 선택한 마크다운 원문이 있을 때 | 유지 |
| I-622 | 3 | Copy Link | (핸들러, 링크 URL을 클립보드로) | — | 링크 위에서 우클릭했을 때 | 유지 |
| I-623 | — | **세 조건이 모두 없으면 메뉴를 열지 않는다** (선택 없음 + 링크 아님 → 우클릭해도 아무것도 안 나옴). 구분선 없음 | — | — | — | 유지 (같게) |

### 2.5 사이드바 (맨 왼쪽)

| ID | 대상 | 메뉴 | 출처 | 판정 |
|---|---|---|---|---|
| I-630 | 프로젝트(폴더) 헤더 행 우클릭 | 헤더의 ⋯ 버튼과 **같은 메뉴**를 토글한다 (persistent 메뉴) | [sidebar.rs#L2518-L2528](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2518-L2528) | — |
| I-631 | └ 1 | **Open Project in New Window** | [sidebar.rs#L2970-L2985](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2970-L2985) | 조건: 로컬 + 프로젝트 2개 이상 ([L2881-L2884](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2881-L2884)). **판단 필요** (우리 앱은 단일 창 멀티 폴더) |
| I-632 | └ 2 | **Focus Project** / 스레드가 있으면 **Focus Last Project**. 오른쪽에 보조 수식키(Windows: Ctrl) + 클릭 힌트. 이미 활성이면 Disabled 색 | [sidebar.rs#L2993-L3030](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2993-L3030) | 판단 필요 ("이 폴더 열기"로 바꿀지) |
| I-633 | └ ─ + 헤더 "Open Worktrees" + 워크트리 목록(활성은 체크, 나머지는 hover 시 × "Close Worktree") | 열린 워크스페이스가 있을 때 | [sidebar.rs#L3045-L3120](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L3045-L3120) | **제외** (Git worktree) |
| I-634 | └ ─ + **Move Up** / **Move Down** (`MoveProjectUp`/`MoveProjectDown`) | 프로젝트 그룹이 2개 이상. 맨 위·맨 아래면 해당 항목 비활성. Windows 기본 키 없음 | [sidebar.rs#L2963-L2995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2963-L2995) | 유지 (등록 폴더 순서 바꾸기) |
| I-635 | └ ─ + **Remove** | 항상 | [sidebar.rs#L3005-L3010](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L3005-L3010) | 유지 (폴더 등록 해제) |
| I-636 | 스레드 항목 우클릭 | 스레드 메뉴 (Rename Title 등) | [sidebar.rs#L6493](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L6493), [sidebar.rs#L6702-L6712](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L6702-L6712) | **제외** (AI 스레드) |
| I-637 | 사이드바 빈 공간 | **없음** (헤더·스레드 항목 외에 우클릭 처리 없음) | sidebar.rs 전체에서 `MouseButton::Right`는 L2518 하나, `right_click_menu`는 L6493·L6702 둘뿐 | — |
| I-638 | 사이드바 푸터의 사이드바 닫기 토글 | "Left" / "Right" 토글 항목(현재 위치에 체크) → 사이드바 위치 변경(`sidebar_side_context_menu`) | [sidebar.rs#L7478](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7478), [multi_workspace.rs#L69-L100](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/multi_workspace.rs#L69-L100) | 판단 필요 (사이드바 오른쪽 배치를 지원할지) |
| I-639 | 폴더 추가 (`FolderAdd`) 버튼 | 우클릭 메뉴 없음 (왼쪽 클릭 = 최근 프로젝트 팝오버) | [sidebar.rs#L6770-L6776](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L6770-L6776) | — |

### 2.6 타이틀 바·상태 표시줄·Dock 경계·아이콘 버튼

| ID | 대상 | 우클릭 메뉴 | 출처 | 판정 |
|---|---|---|---|---|
| I-645 | 타이틀 바 빈 곳 | **Zed 메뉴 없음**. Linux + 클라이언트 장식일 때만 OS 창 메뉴(`show_window_menu`)를 부른다. Windows는 처리 코드 없음 | [platform_title_bar.rs#L339-L347](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platform_title_bar.rs#L339-L347) | 없음 (⑥: Windows 시스템 메뉴 여부 미확인) |
| I-646 | 타이틀 바 ☰·프로젝트 이름 | 우클릭 메뉴 없음 (왼쪽 클릭 팝오버만) | [title_bar.rs#L831-L858](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L831-L858) | — |
| I-647 | 상태 표시줄 Dock 패널 버튼 (예: 프로젝트 패널 `FileTree`) | 메뉴: 패널이 둘 수 있는 위치마다 **"Dock Left" / "Dock Right"**(/ "Dock Bottom") 토글 항목(현재 위치에 체크) → ─ → (유연 폭을 지원하는 패널만) "Flex Width" / "Fixed Width" → ─ → **"Hide Button"**(버튼 숨기기 설정이 있을 때). 프로젝트 패널은 Left/Right만 가능하고(`position_is_valid`) 유연 폭을 지원하지 않으므로: **Dock Left ✓, Dock Right, ─, Hide Button**. 메뉴는 버튼 위쪽에 열린다 (왼쪽 Dock 버튼: 메뉴 BottomLeft를 버튼 TopLeft에) | [dock.rs#L1450-L1535](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1450-L1535), [dock.rs#L1403-L1406](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1403-L1406), [project_panel.rs#L8260-L8262](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8260-L8262), [project_panel.rs#L8325-L8330](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8325-L8330) | 유지 (Hide Button 포함은 판단) |
| I-648 | 상태 표시줄 일반 아이템 (커서 위치 등, `hide_setting`이 있는 것) | **"Hide Button"** 하나 (체크 자리 맞춤용 토글 항목, 꺼진 상태). 누르면 해당 설정(예: `status_bar.cursor_position_button`)을 false로 저장 | [status_bar.rs#L204-L225](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L204-L225), [status_bar.rs#L292-L330](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L292-L330), [cursor_position.rs#L299-L305](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L299-L305) | 판단 필요 (설정 저장을 지원할지) |
| I-649 | 상태 표시줄 사이드바 토글 (사이드바가 닫혀 있을 때) | "Left" / "Right" (I-638과 같은 메뉴) | [status_bar.rs#L241](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L241) | I-638과 같음 |
| I-650 | 상태 표시줄 빈 곳 | 없음 | status_bar.rs의 `right_click_menu`는 아이템·토글에만 있음 | — |
| I-651 | Dock 경계 (리사이즈 핸들) | 없음 (더블클릭 = 크기 초기화만) | [dock.rs#L1281-L1336](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1281-L1336) | — |
| I-652 | 탭 바 ← → · 최대화 버튼 | 없음 | [pane.rs#L3508-L3553](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3508-L3553), [pane.rs#L4417-L4430](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4417-L4430) | — |

---

## 3. 우리 앱 최종 메뉴 구조 (제안, 유지 항목만)

구분선 위치는 Zed와 같게 했다. 가운데 그룹이 모두 빠져 구분선이 연속되면 하나로 합친다 (Zed도 조건부 구분선을 해당 그룹이 있을 때만 넣는다).

### I-660 프로젝트 패널 — 파일

```
New File                     Ctrl-N
New Folder                   Alt-N
──────────────
Reveal in File Explorer      Ctrl-Alt-R
Open in Default App          Ctrl-Shift-Enter
Open Markdown Preview                    (마크다운 파일만)
──────────────
Cut                          Ctrl-X
Copy                         Ctrl-C
Duplicate
Paste                        Ctrl-V      (붙여 넣을 것 없으면 비활성)
[Undo                        Ctrl-Z]     (판단 필요)
[Redo                        Ctrl-Shift-Z] (판단 필요)
──────────────
Copy Path                    Alt-Shift-C
Copy Relative Path           Ctrl-K Ctrl-Shift-C
──────────────
Rename                       F2
Trash                        Delete
Delete                       Ctrl-Delete
```

### I-661 프로젝트 패널 — 폴더 (루트 아님)

```
New File                     Ctrl-N
New Folder                   Alt-N
──────────────
Reveal in File Explorer      Ctrl-Alt-R
Open in Default App          Ctrl-Shift-Enter
──────────────
Unfold Directory                         (접힌 경로일 때)
Fold Directory                           (접을 수 있을 때)
──────────────
Cut / Copy / Duplicate / Paste / [Undo / Redo]   (파일과 같음)
──────────────
Copy Path                    Alt-Shift-C
Copy Relative Path           Ctrl-K Ctrl-Shift-C
──────────────
Rename                       F2
Trash                        Delete
Delete                       Ctrl-Delete
──────────────
Expand All
Collapse All
```
- Zed에서는 이 자리(Unfold·Fold 앞)의 구분선 뒤에 "Find in Folder…"가 먼저 온다. 우리 앱은 검색을 빼므로 구분선 뒤에 Unfold·Fold만 남는다.

### I-662 프로젝트 패널 — 루트 (= 사이드바에서 선택한 등록 폴더), 빈 공간 우클릭도 같음

```
New File                     Ctrl-N
New Folder                   Alt-N
──────────────
Reveal in File Explorer      Ctrl-Alt-R
Open in Default App          Ctrl-Shift-Enter
──────────────
Unfold Directory / Fold Directory        (해당할 때)
──────────────
Cut / Copy / Duplicate / Paste / [Undo / Redo]
──────────────
Copy Path                    Alt-Shift-C
Copy Relative Path           Ctrl-K Ctrl-Shift-C
──────────────
[Remove from Project]                    (판단 필요: 사이드바 "Remove"와 같은 동작으로 연결)
──────────────
Expand All                   Ctrl-Right
Collapse All                 Ctrl-Left
```
- Windows에서 루트는 Rename·Trash·Delete가 없다(Zed 동일).
- "Add Folders to Project…"는 사이드바 폴더 추가로 대체하므로 뺀다 (판단).

### I-663 탭 (편집기 탭·미리보기 탭)

```
Close
Close Others                 Ctrl-Alt-Shift-T   (탭 하나뿐이면 비활성)
──────────────
Close Left                   Ctrl-K E           (왼쪽 탭 없으면 비활성)
Close Right                  Ctrl-K T           (오른쪽 탭 없으면 비활성)
──────────────
Close Clean                  Ctrl-K U           (깨끗한 탭 없으면 비활성)
Close All                    Ctrl-K W
──────────────
[Pin Tab / Unpin Tab         Ctrl-K Shift-Enter] (판단 필요)
──────────────
Copy Path                    Ctrl-K P
Copy Relative Path
──────────────
Reveal in File Explorer      Ctrl-K R
Reveal In Project Panel      Ctrl-Shift-E
──────────────
Open Markdown Preview        Ctrl-Shift-V       (마크다운 편집기 탭)
```

### I-664 탭 (마크다운 미리보기 탭)

```
Close / Close Others / ─ / Close Left / Close Right / ─ / Close Clean / Close All / ─ / [Pin·Unpin]
(파일 항목 표시 여부는 확인 필요 — I-589)
──────────────
Show Source                  Ctrl-Shift-V
```

### I-665 편집기 본문

```
Cut                          Ctrl-X
Copy                         Ctrl-C
Copy and Trim
Paste                        Ctrl-V
──────────────
Reveal in File Explorer      Ctrl-K R     (파일 없으면 비활성)
Open Markdown Preview        Ctrl-Shift-V (마크다운 버퍼)
```
- Zed는 맨 위에 디버그 항목, LSP 7개, Rename Symbol·Format·Code Actions·Agent를 둔다. 모두 제외하므로 메뉴는 Cut부터 시작한다 (앞쪽 구분선은 생략).

### I-666 마크다운 미리보기 본문

```
Copy                         Ctrl-C   (선택 있을 때)
Copy as Markdown                      (선택 있을 때)
Copy Link                             (링크 위일 때)
```
- 세 조건이 모두 없으면 메뉴를 열지 않는다.

### I-667 사이드바 폴더 항목 (Zed 프로젝트 헤더 메뉴 대응)

```
[Focus Project]                          (판단 필요: 이 폴더로 전환)
──────────────
Move Up                                  (폴더 2개 이상, 맨 위면 비활성)
Move Down                                (폴더 2개 이상, 맨 아래면 비활성)
──────────────
Remove
```
- 빈 공간 우클릭은 없음.

### I-668 상태 표시줄

```
프로젝트 패널 버튼:  ✓ Dock Left / Dock Right / ─ / [Hide Button]
커서 위치:           [Hide Button]
사이드바 토글:       [✓ Left / Right]   (사이드바 위치 변경을 지원할 때)
```
- 타이틀 바, 탭 바 빈 공간, Dock 경계, 탭 바 버튼에는 우클릭 메뉴가 없다 (Zed와 같음).

---

## 4. 확신 낮은 부분

- **"Close" 항목의 단축키 (I-570):** 메뉴 액션은 `close_pinned: true`인데 키맵의 Ctrl-W는 `close_pinned: false`다. 액션 비교가 인자까지 보면 단축키가 표시되지 않는다. 실제 표시는 미확인이다.
- **표시 단축키 선택:** 같은 액션에 키가 여럿일 때 "마지막 정의 우선"으로 고른 값은 추정이다. 해당 항목은 Copy = Ctrl-C, Paste = Ctrl-V, Trash = Delete, Delete = Ctrl-Delete, Cut = Ctrl-X다. 탭 메뉴의 Copy Relative Path에 키가 없다는 것도 Editor 컨텍스트에서 바인딩을 찾지 못한 데 근거한 추정이다.
- **수식키 표기 순서:** Close Others 키는 키맵에 `ctrl-shift-alt-t`로 적혀 있다. 표기 순서 규칙(Ctrl → Alt → Shift)을 적용해 "Ctrl-Alt-Shift-T"로 보았다.
- **미리보기 탭의 파일 항목 (I-589):** 마크다운 미리보기 탭 메뉴에 Copy Path·Reveal 항목이 나오는지 확인하지 못했다.
- **여러 항목 표시 상태 (I-562):** 여러 항목을 marked 했을 때 각 액션이 표시된 모든 항목에 적용되는지는 액션 구현까지 보지 않았다.
- **항목 높이 (I-508):** 약 25px은 줄 높이와 테두리로 계산한 추정이다.
- **Windows 타이틀 바 우클릭 (I-645):** Zed 코드에는 처리가 없다. 타이틀 바가 OS에 캡션(드래그) 영역으로 등록되어 있어 **Windows 시스템 메뉴(이동·크기·최소화·닫기)가 OS 차원에서 뜨는지**는 미확인이다.
- **사이드바 메뉴 (I-631, I-632):** "Focus Project" 항목도 "프로젝트 2개 이상" 조건 블록 안에 있다고 보았다 (코드 구조상 추정). 오른쪽 수식키 힌트의 정확한 모양은 미확인이다.
- **줄 번호:** 일부 링크의 줄 범위는 grep 결과로 잡은 근사치(±수 줄)다. 특히 I-502, I-519, I-631~I-635, I-647, I-648이 그렇다.
