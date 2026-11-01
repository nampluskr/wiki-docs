# Zed 툴팁(hover 메시지) 조사 (ID 접두사 L-, L-200부터)

- 담당: scouter-opus · 조사일 2026-10-31
- 소스 기준: zed-industries/zed 커밋 `14b7b0ef3764d62a85070c8b5ccea105a2ad3235` (링크는 모두 이 커밋에 고정). GPL 코드 원문은 옮기지 않고 문구·값·이름만 적었다. 툴팁 문구는 화면에 보이는 짧은 UI 문자열이라 그대로 적는다.
- 단축키는 **Windows 기본 키맵** `assets/keymaps/default-windows.json` 기준이다. 사용자 설정은 `base_keymap: "VSCode"`이지만, Windows 기본 키맵이 이미 VSCode 계열이라 결과가 같다고 본다 (추정, ⑥ 참고).
- 링크 접두어 생략 표기: `Z/` = `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/`

---

## 1. 툴팁 공통 명세

### 1.1 컴포넌트 종류 (`ui::Tooltip`)

| ID | 생성 방법 | 내용 | 출처 |
|---|---|---|---|
| L-200 | `Tooltip::text(title)` / `Tooltip::simple` | 제목 한 줄만 | [Z/crates/ui/src/components/tooltip.rs#L37-L56](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L37-L56) |
| L-201 | `Tooltip::for_action(title, action)` | 제목 + 그 액션의 **단축키**(오른쪽). 단축키는 전역 키맵에서 찾는다 | [tooltip.rs#L96-L107](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L96-L107) |
| L-202 | `Tooltip::for_action_in(title, action, focus)` | 위와 같지만, 주어진 포커스 위치의 키 컨텍스트에서 단축키를 찾는다 (예: Pane 안에서만 유효한 키) | [tooltip.rs#L109-L121](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L109-L121) |
| L-203 | `Tooltip::with_meta(title, action?, meta)` / `with_meta_in` | 제목(+단축키) 아래에 **메타 줄**(두 번째 줄) | [tooltip.rs#L123-L150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L123-L150) |
| L-204 | `Tooltip::element(...)` | 임의 요소. 예: 사이드바 토글의 두 줄 툴팁(L-232) | [tooltip.rs#L160-L181](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L160-L181) |
| L-205 | `PopoverMenu::trigger_with_tooltip` | 메뉴를 여는 버튼의 툴팁. **메뉴가 열려 있는 동안에는 툴팁을 달지 않는다**. 열려 있으면 버튼이 선택 상태(toggle_state) | [Z/crates/ui/src/components/popover_menu.rs#L198-L212](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L198-L212) |

### 1.2 표시·사라짐·위치

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| L-206 | 표시 지연 | 기본 **500ms** (`DEFAULT_TOOLTIP_SHOW_DELAY`). 요소별로 `tooltip_show_delay`로 바꿀 수 있다 (예: 프로젝트 패널 항목 1500ms, L-240) | [Z/crates/gpui/src/elements/div.rs#L53](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L53), [div.rs#L758-L761](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L758-L761) |
| L-207 | 사라지는 조건 | (1) 마우스가 요소 hitbox를 벗어나면, (2) **마우스 버튼을 누르면**, (3) **스크롤하면**. hoverable 툴팁(우리 대상 아님)은 툴팁 위로 옮겨도 유지되고, 벗어난 뒤 500ms(`HOVERABLE_TOOLTIP_HIDE_DELAY`) 뒤에 사라진다 | [div.rs#L54](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L54), [div.rs#L3744-L3860](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L3744-L3860), [div.rs#L3875-L3935](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L3875-L3935) |
| L-208 | 위치 기준 | **마우스 기준**(요소 기준 아님). 툴팁을 띄우는 순간의 마우스 위치를 기록해 그 점에서 (+1, +1)px에 둔다 | [div.rs#L3987-L3989](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L3987-L3989), [Z/crates/gpui/src/window.rs#L3884-L3887](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/window.rs#L3884-L3887) |
| L-209 | 커서와의 간격 | 툴팁 바깥 상자에 왼쪽 `pl_2`(8px), 위 `pt_2p5`(10px)의 투명 여백이 있다. "커서 바로 아래에 겹치지 않게" 하려는 것. 그래서 보이는 상자는 커서에서 약 **(9, 11)px** 떨어진다 (1px + 여백) | [tooltip.rs#L223-L224](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L223-L224) |
| L-210 | 창 가장자리 처리 | 오른쪽이 창을 넘으면 커서 **왼쪽**(x − 폭 − 1)으로 뒤집는다. 그래도 0보다 작으면 0에 붙인다. 아래쪽이 넘으면 커서 **위쪽**(y − 높이 − 1)으로 뒤집는다 | [window.rs#L3893-L3913](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/window.rs#L3893-L3913) |

### 1.3 모양

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| L-211 | 바탕 | `elevation_2` = 배경 `elevated_surface_background`, 모서리 `rounded_lg`, 테두리 `border_1`(`border_variant`), 그림자 ElevatedSurface | [Z/crates/ui/src/traits/styled_ext.rs#L6-L11](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/traits/styled_ext.rs#L6-L11), [styled_ext.rs#L62-L64](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/traits/styled_ext.rs#L62-L64) |
| L-212 | 그림자 | 두 겹: ① y +2px, blur 3px, 검정 12% ② y +1px, blur 0, 검정 **라이트 3% / 다크 6%** | [Z/crates/ui/src/styles/elevation.rs#L49-L56](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/elevation.rs#L49-L56) |
| L-213 | 안쪽 여백 | 위아래 `py_1` = 4px, 좌우 `px_2` = 8px | [tooltip.rs#L230-L231](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L230-L231) |
| L-214 | 글꼴·크기·색 | UI 글꼴, `text_ui` = 14px(rem 16 기준, `TextSize::Default`), 색 `text` | [tooltip.rs#L221-L229](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L221-L229), [typography.rs#L139](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/typography.rs#L139) |
| L-215 | 첫 줄 배치 | 가로: [제목 (최대 폭 `max_w_72` = 18rem = 288px)] … [단축키]. 단축키가 있으면 양끝 정렬(`justify_between`), 둘 사이 gap `_4` = 16px | [tooltip.rs#L196-L203](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L196-L203) |
| L-216 | 메타 줄 | 제목 줄 아래에 별도 줄. `LabelSize::Small`(12px), 색 `Muted`, 최대 폭 288px | [tooltip.rs#L205-L211](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L205-L211) |

### 1.4 단축키 표시 형식 (Windows)

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| L-217 | 어떤 키를 보여주나 | 액션에 묶인 키 중 **우선순위가 가장 높은 하나**. 키맵에 나중에 추가된 바인딩이 우선이고(사용자 키맵이 기본 키맵을 이긴다), `_in` 계열은 포커스 위치의 컨텍스트를 먼저 본다 | [Z/crates/ui/src/components/keybinding.rs#L82-L99](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L82-L99), [Z/crates/gpui/src/keymap.rs#L137-L159](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/keymap.rs#L137-L159) |
| L-218 | 글자 방식 | Windows·Linux(그리고 Vim 모드)는 아이콘 없이 **문자열 하나**로 그린다 (macOS만 ⌘ 같은 기호) | [keybinding.rs#L342-L359](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L342-L359) |
| L-219 | 구분자·순서 | 수식키 사이 구분자는 **`-`(하이픈)**이고 `+`가 아니다. 순서는 **Fn → Ctrl → Win → Alt → Shift → 키**. 예: `Ctrl-B`, `Ctrl-Shift-E`, `Ctrl-Alt-J` | [keybinding.rs#L691-L760](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L691-L760) |
| L-220 | 키 이름 | 첫 글자를 대문자로 (`capitalize`): `left` → `Left`, `escape` → `Escape`, `g` → `G`. 예외 `pageup` → `PageUp`, `pagedown` → `PageDown` | [keybinding.rs#L752-L760](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L752-L760) |
| L-221 | 연속 키(chord) | 키 입력마다 덩어리 하나, 사이 gap Base04 = 4px. 예: `Ctrl-K Ctrl-T` | [keybinding.rs#L280-L288](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L280-L288) |
| L-222 | 키 글자 모양 | 크기 기본 `TextSize::Default`(14px), 줄 높이 1.0, 높이 = 글자 크기, 좌우 `px_0p5`(2px), 위아래 `py_0p5`, 색 **Muted**(비활성이면 Disabled) | [keybinding.rs#L535-L560](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L535-L560), [keybinding.rs#L259-L287](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L259-L287) |
| L-223 | 바인딩이 없을 때 | 단축키 영역 없이 제목만 (추정: `for_action`이 바인딩을 못 찾으면 빈 요소) | [keybinding.rs#L196-L205](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L196-L205) |

---

## 2. 요소별 툴팁 문구

단축키 열은 L-217~L-221 규칙을 Windows 기본 키맵에 적용한 결과다.

### 2.1 타이틀 바

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
|---|---|---|---|---|---|
| L-225 | ☰ 메뉴 버튼 | **Open Application Menu** | `text` (단축키 없음). 메뉴가 열려 있으면 툴팁 없음 | 참고: `app_menu::OpenApplicationMenu`는 F10에 묶여 있지만 툴팁에는 표시하지 않는다 | [Z/crates/title_bar/src/application_menu.rs#L192-L200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L192-L200), [default-windows.json#L662](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L662) |
| L-226 | 프로젝트(폴더) 이름 버튼 | **Recent Projects** | `for_action`, 메뉴가 열려 있으면 툴팁 없음 | `projects::OpenRecent` → **Ctrl-R** | [Z/crates/title_bar/src/title_bar.rs#L831-L858](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L831-L858), [default-windows.json#L648](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L648) |
| L-227 | 프로젝트 이름 버튼 (상태별 표시) | 폴더가 없으면 버튼 글자가 "Open Recent Project"(Muted), 툴팁은 같다. 사이드바가 열려 있고 스레드 목록 화면이면 다른 렌더 경로(`render_recent_projects_popover`)를 타지만 툴팁 문구와 액션은 같다 | — | 위와 같음 | [title_bar.rs#L792-L816](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L792-L816), [title_bar.rs#L864-L912](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L864-L912) |
| L-228 | 최소화·최대화·복원·닫기 | **Zed는 툴팁을 달지 않는다** (Windows 창 버튼 코드에 tooltip 호출 없음) | — | — | [Z/crates/platform_title_bar/src/platforms/platform_windows.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs) (파일 전체에 tooltip 없음) |

### 2.2 사이드바

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
|---|---|---|---|---|---|
| L-230 | 폴더 추가 (`FolderAdd` 아이콘) | **Add Project** | `for_action`, 메뉴가 열려 있으면 툴팁 없음 | `projects::OpenRecent` → **Ctrl-R** | [Z/crates/sidebar/src/sidebar.rs#L6770-L6776](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L6770-L6776) |
| L-231 | 프로젝트(폴더) 헤더 행 hover | **툴팁 없음** (헤더 렌더 코드에 tooltip 없음, 원격 프로젝트일 때만 아이콘에 "Remote Project") | — | — | [sidebar.rs#L2326](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2326), [sidebar.rs#L2331-L2440](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2331-L2440) |
| L-232 | 사이드바 닫기 토글 (`ThreadsSidebarLeftOpen`/`RightOpen`) | **두 줄 툴팁**: 1줄 "**Toggle Sidebar**" + 단축키 / 구분선(`border_t_1`, `border_variant`, 위 padding 4) / 2줄 "**Focus Sidebar**" + 단축키. 각 줄은 gap 8, 양끝 정렬, 두 줄 사이 gap 4 | `element` | `multi_workspace::ToggleWorkspaceSidebar` → **Ctrl-Alt-J** / `multi_workspace::FocusWorkspaceSidebar` → **Ctrl-Alt-;** | [sidebar.rs#L7495-L7518](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7495-L7518), [default-windows.json#L675-L676](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L675-L676) |
| L-233 | (참고, 제외) 기록 시계 버튼 | "Show Thread History" / 기록 화면에서는 "Hide Thread History" | `for_action` | `agents_sidebar::ToggleThreadHistory` → Ctrl-G (ThreadsSidebar 컨텍스트) | [sidebar.rs#L7542-L7551](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7542-L7551), [default-windows.json#L763](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L763) |
| L-234 | (참고, 제외) 헤더의 ⋯ 메뉴, 새 스레드 + | ⋯은 툴팁 없음, + 는 "Start New Agent Thread" | — | — | [sidebar.rs#L2868-L2915](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2868-L2915), [sidebar.rs#L2610-L2638](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2610-L2638) |

### 2.3 탭 바

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
|---|---|---|---|---|---|
| L-236 | ← 뒤로 | **Go Back** | `for_action_in`(현재 포커스, 없으면 Pane) | `pane::GoBack` → **Alt-Left** (Pane 컨텍스트 바인딩 `back`, `alt--`, `alt-left` 중 마지막) | [Z/crates/workspace/src/pane.rs#L3508-L3530](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3508-L3530), [default-windows.json#L516-L518](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L516-L518) |
| L-237 | → 앞으로 | **Go Forward** | `for_action_in` | `pane::GoForward` → **Alt-Right** (`forward`, `alt-=`, `alt-right` 중 마지막) | [pane.rs#L3531-L3553](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3531-L3553), [default-windows.json#L519-L521](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L519-L521) |
| L-238 | 탭 본체 hover (편집기 탭) | **파일 전체 경로**: 워크트리 기준 절대 경로, 홈 디렉터리는 `~`로 줄임(`compact`). 예: `D:\projects\docs\guides\rust-gui\README.md`. 경로가 없는 버퍼는 제목. 읽기 전용 탭이면 메타 줄 "**Read-Only Tab**" 추가 | `text`, 읽기 전용이면 `with_meta` | 단축키 없음 | [pane.rs#L3099-L3125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3099-L3125), [Z/crates/editor/src/items.rs#L706-L726](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L706-L726) |
| L-239 | 미리보기 탭 hover ("Preview X.md") | **원본 편집기의 툴팁 그대로**(원본 파일 전체 경로, L-238과 같음) | `text` | 없음 | [Z/crates/markdown_preview/src/markdown_preview_view.rs#L1745-L1751](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1745-L1751) |
| L-240 | 탭 닫기 × | **Close Tab**. **활성 탭에서만 단축키**가 붙고, 비활성 탭은 문구만 | 활성: `for_action_in` / 비활성: `text` | `pane::CloseActiveItem {close_pinned: false}` → **Ctrl-W** (Pane 컨텍스트 `ctrl-f4`, `ctrl-w` 중 마지막) | [pane.rs#L3053-L3085](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3053-L3085), [default-windows.json#L507-L508](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L507-L508) |
| L-241 | 고정(pinned) 탭의 핀 | **Unpin Tab** (활성 탭이면 단축키) | 위와 같음 | `pane::TogglePinTab` → Ctrl-K Shift-Enter | [pane.rs#L3036-L3053](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3036-L3053), [default-windows.json#L534](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L534) |
| L-242 | 읽기 전용 자물쇠 (아이콘 자리) | 잠금 해제 가능: "**Unlock Tab**" + 메타 "This will make this tab editable" / 해제 불가: "**Locked Tab**" + 메타 "This tab is read-only" | `with_meta` | 없음 | [pane.rs#L2913-L2928](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2913-L2928) |
| L-243 | + (새 항목) | **New…** (말줄임표 문자 `…`) | `text`, 메뉴가 열려 있으면 없음 | 없음 | [pane.rs#L4364-L4370](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4364-L4370) |
| L-244 | 분할 | **Split Pane** | `text`, 메뉴가 열려 있으면 없음 | 없음 | [pane.rs#L4388-L4394](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4388-L4394) |
| L-245 | 최대화(줌) | 평소 **Zoom In**, 확대 상태 **Zoom Out** | `for_action` | `workspace::ToggleZoom` → **Shift-Escape** | [pane.rs#L4417-L4430](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4417-L4430), [default-windows.json#L26](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L26) |

### 2.4 프로젝트 패널

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
|---|---|---|---|---|---|
| L-247 | 항목 hover | **항목의 절대 경로**(`compact`: 홈은 `~`). 지연 **1500ms**(`title_tooltip_delay: "default"`), 설정으로 사용자 값 또는 `disabled` | `text` | 없음 | [Z/crates/project_panel/src/project_panel.rs#L6234-L6268](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6234-L6268), [default.json#L886](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L886) |
| L-248 | 심볼릭 링크 표시(↗) | 대상 경로 + 메타 "**Symbolic Link**" | `with_meta` | 없음 | [project_panel.rs#L6640-L6651](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6640-L6651) |
| L-249 | 패널 상단·하단 아이콘 | **없음**. 프로젝트 패널 render에 IconButton이 없고, 폴더가 없을 때만 빈 상태(`ProjectEmptyState`) 화면 | — | — | [project_panel.rs#L8157](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8157) |

### 2.5 상태 표시줄

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
|---|---|---|---|---|---|
| L-251 | 사이드바 토글 (사이드바가 닫혀 있을 때만 보임) | **Open Threads Sidebar** | `for_action` | `multi_workspace::ToggleWorkspaceSidebar` → **Ctrl-Alt-J** | [Z/crates/workspace/src/status_bar.rs#L241-L270](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L241-L270) |
| L-252 | 폴더 추가 | Zed 상태 표시줄에는 없다. 사이드바의 "Add Project"(L-230)가 해당 | — | Ctrl-R | — |
| L-253 | 프로젝트 패널 토글 (패널이 **닫혀 있거나 다른 패널이 활성**일 때) | **Project Panel** (`icon_tooltip`) | `for_action` | 패널의 `toggle_action` = `project_panel::ToggleFocus` → **Ctrl-Shift-E** | [Z/crates/workspace/src/dock.rs#L1416-L1441](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1416-L1441), [project_panel.rs#L8284-L8290](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8284-L8290), [default-windows.json#L699](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L699) |
| L-254 | 프로젝트 패널 토글 (패널이 **열려 있고 활성**) | **Close Left Dock** (패널이 오른쪽 Dock이면 "Close Right Dock", 아래면 "Close Bottom Dock") | `for_action` | Dock의 `toggle_action` = `workspace::ToggleLeftDock` → **Ctrl-B** (오른쪽: `ToggleRightDock` → Ctrl-Alt-B) | [dock.rs#L1431-L1441](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1431-L1441), [dock.rs#L360-L366](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L360-L366), [default-windows.json#L672-L673](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L672-L673) |
| L-255 | 패널 버튼 공통 | 오른쪽 클릭 메뉴(Dock 위치 변경)가 **열려 있는 동안 툴팁 없음** | — | — | [dock.rs#L1539-L1558](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1539-L1558) |
| L-256 | 커서 위치 ("9:11") | **Go to Line/Column** (누르면 Go to Line 대화상자) | 편집기 포커스 컨텍스트가 있으면 `for_action_in`, 없으면 `for_action` | `go_to_line::Toggle`(`editor::actions::ToggleGoToLine`) → **Ctrl-G** (`Editor && mode == full` 컨텍스트) | [Z/crates/go_to_line/src/cursor_position.rs#L248-L260](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L248-L260), [Z/crates/editor/src/actions.rs#L407-L412](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/actions.rs#L407-L412), [default-windows.json#L626-L630](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L626-L630) |

### 2.6 테마·아이콘 테마 선택

| ID | 요소 | 내용 | 출처 |
|---|---|---|---|
| L-258 | 전용 버튼 | Zed 화면에는 테마 선택 **아이콘 버튼이 없다** (따라서 툴팁도 없음). 메뉴 항목으로만 노출 | 아래 두 출처 |
| L-259 | ☰ 메뉴 항목 | Zed 메뉴 > Settings 하위: "**Select Theme...**"(`theme_selector::Toggle`), "**Select Icon Theme...**"(`icon_theme_selector::Toggle`). 메뉴 항목 오른쪽에는 단축키가 표시된다 (컨텍스트 메뉴는 항목마다 KeyBinding을 그림) | [Z/crates/zed/src/zed/app_menus.rs#L87-L95](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L87-L95), [Z/crates/ui/src/components/context_menu.rs#L1805-L1806](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1805-L1806) |
| L-260 | (참고, 제외) 사용자 메뉴 | "Themes…", "Icon Themes…" 항목 (Sign In 쪽 메뉴라 우리 앱에서는 제외) | [title_bar.rs#L1374-L1383](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1374-L1383) |
| L-261 | 단축키 | `theme_selector::Toggle` → **Ctrl-K Ctrl-T**. `icon_theme_selector::Toggle`은 Windows 기본 키맵에 바인딩 없음 | [default-windows.json#L688](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688) |

---

## 3. 상태에 따라 바뀌는 문구 모음

| ID | 요소 | 상태 → 문구 |
|---|---|---|
| L-265 | 최대화(줌) 버튼 | 확대 아님 → "Zoom In" / 확대됨 → "Zoom Out" (아이콘도 Maximize → Minimize) |
| L-266 | Dock 패널 버튼 | 닫힘·비활성 → "{패널 이름}" (예: "Project Panel") + 패널 토글 키 / 열림·활성 → "Close {Left·Right·Bottom} Dock" + Dock 토글 키 |
| L-267 | 탭 끝 슬롯 | 일반 탭 → "Close Tab" / 고정 탭 → "Unpin Tab". 활성 탭만 단축키 표시 |
| L-268 | 탭 본체 | 일반 → 경로 / 읽기 전용 → 경로 + "Read-Only Tab" |
| L-269 | 자물쇠 | 해제 가능 → "Unlock Tab"(+"This will make this tab editable") / 해제 불가 → "Locked Tab"(+"This tab is read-only") |
| L-270 | 사이드바 토글 | 사이드바 닫힘 → 상태 표시줄 "Open Threads Sidebar" / 열림 → 사이드바 안 버튼의 두 줄 "Toggle Sidebar" + "Focus Sidebar" |
| L-271 | (제외) 기록 버튼 | "Show Thread History" ↔ "Hide Thread History" |
| L-272 | 메뉴를 여는 버튼 공통 (☰, 프로젝트 이름, +, 분할, 폴더 추가, 패널 버튼 우클릭 메뉴) | 메뉴가 열려 있으면 툴팁 없음 |

---

## 4. 확신 낮은 부분

- **줄 번호:** 일부 링크의 줄 범위는 grep으로 잡은 근사치다. L-225, L-236~L-245, L-247은 ±수 줄 차이가 날 수 있다. 파일과 커밋은 정확하다.
- **표시되는 단축키 선택(L-236, L-237, L-240):** "Alt-Left", "Alt-Right", "Ctrl-W"는 "같은 컨텍스트에서 마지막에 정의된 바인딩이 우선"이라는 규칙(L-217)을 키맵 순서에 적용해 고른 **추정**이다. 실제 Zed 화면으로 확인하지 않았다. 포커스가 편집기 안에 있으면 더 깊은 컨텍스트의 다른 바인딩이 이길 가능성도 있다.
- **base_keymap:** 사용자의 `base_keymap: "VSCode"`가 Windows 기본 키맵 위에 다른 바인딩을 더하는지는 확인하지 않았다(추정: 같음).
- **Windows 창 버튼:** Zed 코드에는 툴팁이 없다. 다만 각 버튼이 `WindowControlArea`(Min/Max/Close) 히트테스트에 연결되어 있어, **Windows가 시스템 툴팁("최소화" 등)이나 스냅 레이아웃 플라이아웃을 보여 주는지**는 미확인이다.
- **위치 고정(L-208):** 툴팁이 처음 뜬 마우스 위치에 고정되고 마우스를 따라가지 않는다고 봤다. `mouse_position`을 툴팁 생성 시 기록하는 코드에 근거한 추정이다.
- **바인딩 없는 액션(L-223):** 바인딩이 없을 때 단축키 영역이 완전히 빠지는지(빈 공간 없이)는 코드 일부만 보고 판단했다.
- **`rounded_lg`, `max_w_72`의 px 값:** 8px, 288px은 gpui tailwind 단위 관례(rem 16 기준)에 따른 추정이다.
- **사이드바 툴팁 단축키(L-232):** "Ctrl-Alt-;"는 키 이름 `;`를 대문자화해도 그대로라는 규칙에서 나온 추정 표기다.
