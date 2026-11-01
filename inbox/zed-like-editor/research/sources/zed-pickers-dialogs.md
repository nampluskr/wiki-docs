# Zed Picker · 대화상자 · 인라인 입력 · 텍스트 상태색 상세 명세 (`P-001` ~ `P-028`)

- **기준 커밋**: `14b7b0ef3764d62a85070c8b5ccea105a2ad3235` (`https://github.com/zed-industries/zed/tree/14b7b0ef3764d62a85070c8b5ccea105a2ad3235`)
- **기준 해상도·배율**: Windows 11 논리 픽셀(`100%` 배율), `1rem = 16px`, `ui_font_size = 16px`, `buffer_font_size = 15px` (`_decisions.md` 기준)
- **기준 화면(보조 실측)**:
  1. `D:\projects\docs\guides\zed-like-editor\01-zed\assets\zed-reference.png` (다크 테마 `Ayu Dark`, 창 최대화 상태, `2880×1728` @ 150%)
  2. `D:\projects\docs\guides\zed-like-editor\01-zed\assets\zed-reference-latte.png` (라이트 테마 `Catppuccin Latte`, 창 비최대화/복원 상태, `2448×1480` @ 150%)
- **기준 키맵**: Windows 기본값 (`base_keymap = "VSCode"`, `assets/keymaps/default-windows.json` + `assets/keymaps/linux/vscode.json` + `assets/keymaps/vscode/specific-overrides.json`)

---

## 1. Picker 공통 레이아웃·동작·색상 명세 (`P-001` ~ `P-006`)

### `P-001` 모달 레이어(`ModalLayer`) 배치 및 외부 배경 처리
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/modal_layer.rs#L50-L71`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/modal_layer.rs#L298-L331`
- **배치 구조**:
  - 창 전체를 덮는 절대 위치 오버레이 컨테이너(`div().absolute().size_full().inset_0().occlude()`) 위에 렌더링한다 (`modal_layer.rs:298-302`).
  - 내부 정렬 래퍼는 `v_flex().h(px(0.0)).top_20().items_center()`로 구성되어, **창 상단에서 `top_20()` = `5rem = 80px` 내려온 위치**에 상단 가장자리를 맞추고 **수평 중앙 정렬(`items_center()`)** 한다 (`modal_layer.rs:314-318`).
- **배경 페이드(Backdrop Wash)**:
  - `ModalView::fade_out_background(&self)` 트레이트 메서드의 기본값은 `false`이다 (`modal_layer.rs:58-60`). 따라서 일반 Picker(명령 팔레트, 파일 찾기, 최근 폴더, 테마 선택기, 줄 이동)는 창 전체에 어두운 딤(dim) 배경을 깔지 않는다.
  - `fade_out_background()`가 `true`인 모달만 `elevated_surface_background` 색상에 `fade_out(0.2)`(불투명도 `80%`)를 적용한 배경 워시를 깐다 (`modal_layer.rs:303-307`).
- **외부 클릭 닫기**:
  - 바깥 오버레이 영역에서 마우스 왼쪽 버튼(`MouseButton::Left`) 다운 이벤트가 발생하면 `hide_modal(window, cx)`를 호출해 모달을 닫는다 (`modal_layer.rs:308-313`).
  - 모달 본체를 감싸는 내부 `h_flex().occlude()`가 `MouseButton::Left` 다운 이벤트의 전파를 차단(`cx.stop_propagation()`)하므로 모달 내부 클릭 시에는 닫히지 않는다 (`modal_layer.rs:319-327`).

### `P-002` Picker 컨테이너 치수 (너비 · 최대 높이 · 크기 경계 · 외곽 스타일)
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L45-L46`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L610-L706`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/shape.rs#L198-L253`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/shape.rs#L664-L671`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L72-L91`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/traits/styled_ext.rs#L6-L12`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/elevation.rs#L58-L90`
- **기본 치수 상수**:
  - `DEFAULT_MODAL_WIDTH`: `Rems(34.0)` = **`544px`** (`picker.rs:45`).
  - `DEFAULT_MODAL_MAX_HEIGHT`: `Rems(24.0)` = **`384px`** (`picker.rs:46`).
  - 개별 Picker가 `.initial_width(...)` 또는 `.max_height(...)`를 지정하지 않으면 위 기본값(`544px × 최대 384px`)이 적용된다 (`picker.rs:601-634`).
  - 미리보기(Telescope) 지원 Picker(`Picker::uniform_list_with_preview`)의 기본 비율은 뷰포트 대비 너비 `60%`(`0.6`), 높이 `60%`(`0.6`), 미리보기 영역 `30%`(`0.3`)이며, 미리보기가 꺼져 있을 때는 일반 모달 크기(`initial_width`, `max_height`)로 동작한다 (`shape.rs:664-671`, `picker.rs:658-680`).
- **뷰포트 경계 제약 (`SizeBounds::new_for_window`, `shape.rs:237-253`)**:
  - `max_width`: 뷰포트 너비의 `95%` (`viewport_size.width * 0.95`).
  - `max_height`: `(뷰포트 높이 - 10rem(160px)) * 0.95` (상하 여백 `5rem = 80px`씩을 제외한 영역의 `95%`).
  - `min_results`: 기본 `280px × 320px` (`px(280.)`, `px(320.)`), 단 미리보기가 없는 일반 Picker는 초기 너비(`initial_width`)를 최소 너비로 고정한다 (`picker.rs:623-625`).
  - `min_preview`: `128px × 96px` (`px(128.)`, `px(96.)`).
- **높이 수축 동작 (`ListSizingBehavior`)**:
  - 미리보기가 활성화되지 않았고 `widest_item`이 지정되지 않은 기본 상태에서 `fill_height()`는 `false`이며, 리스트는 `ListSizingBehavior::Infer`로 동작해 항목 수가 적을 때 `max_height`(`384px`)까지 늘어나지 않고 항목 높이에 맞춰 자동 수축한다 (`picker.rs:146-148`, `1519-1527`, `render.rs:220-239`).
- **외곽 컨테이너 스타일 (`elevation_3`, `render.rs:72-75`, `styled_ext.rs:6-12, 82-84`)**:
  - `draws_own_container`가 `true`(모달 기본값)일 때 `.elevation_3(cx)`(`ElevationIndex::ModalSurface`)와 `.overflow_hidden()`을 적용한다:
    - 배경색: `elevated_surface_background`
    - 모서리 반경: `rounded_lg()` = `0.5rem` = **`8px`**
    - 외곽 테두리: `border_1()` = **`1px` 실선**, 색상 `border_variant`
    - 그림자(`MODAL_SURFACE_SHADOW`, `elevation.rs:58-77`):
      1. `offset: (0, 2px)`, `blur: 3px`, `spread: 0px`, `hsla(0, 0, 0, 0.12)`
      2. `offset: (0, 4px)`, `blur: 8px`, `spread: 0px`, `hsla(0, 0, 0, 0.08)`
      3. `offset: (0, 8px)`, `blur: 16px`, `spread: 0px`, `hsla(0, 0, 0, 0.04)`
      4. `offset: (0, 12px)`, `blur: 24px`, `spread: 0px`, `hsla(0, 0, 0, 0.12)`

### `P-003` 검색 입력칸(`Head::Editor`) 레이아웃 및 구성 요소
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/head.rs#L17-L56`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L96-L146`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L177-L185`
- **위치**:
  - `PickerEditorPosition::Start`(기본값)일 때 모달 최상단에 배치되고, 바로 아래에 `Divider::horizontal()`(`1px` 높이, 색상 `border_variant`)로 결과 목록과 구분된다 (`render.rs:148-159`).
- **입력칸 행 치수**:
  - 컨테이너: `h_flex().w_full().h_9().px_2p5().justify_between().flex_none().overflow_hidden()` (`head.rs:28`, `render.rs:97-101`).
  - 높이: `h_9()` = `2.25rem` = **`36px`**.
  - 좌우 패딩: `px_2p5()` = `0.625rem` = **`10px`**.
- **내부 구성**:
  - 왼쪽(`div().flex_1().overflow_x_hidden()`): 단일 행 `Editor`(`Editor::single_line`)를 렌더링하며, 쿼리가 비어 있을 때 `delegate.placeholder_text(window, cx)`를 `text_placeholder` 색상으로 표시한다 (`head.rs:22-27`, `render.rs:102-104`).
  - 오른쪽(`h_flex().gap_1().flex_none()`):
    - `delegate.render_searchbar_trailer(...)`: Picker별 추가 아이콘 버튼이나 로딩 스피너 (`render.rs:105-107`).
    - 다중 선택 지원 Picker(`supports_multi_select == true`, 예: `file_finder`)일 때: `IconButton::new("toggle_multi_select_button", IconName::FileMultiple)` (`IconSize::Small` = `14px`, 선택 시 `ButtonStyle::Tinted(TintColor::Accent)`, 툴팁 `"Toggle Multi Select"`, 액션 `picker::ToggleMultiSelect` = `Ctrl+Shift+S`) (`render.rs:108-144`, `specific-overrides.json:10`).

### `P-004` 결과 목록 · 항목 높이 · 선택 강조 · 빈 결과(`No matches`)
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L227-L229`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L1452-L1549`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L220-L272`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L298-L474`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/label/highlighted_label.rs#L220-L237`
- **목록 컨테이너 패딩 및 스크롤바**:
  - `uniform_list`(또는 가변 높이 `list`)에 상하 패딩 `.py_1()`(`0.25rem = 4px`)을 부여한다 (`picker.rs:1541`).
  - 스크롤바는 `show_scrollbar`가 `true`일 때 표시되며, 기본값은 `false`(스크롤 시에도 숨김, 단 `CommandPalette`는 `.show_scrollbar(true)`로 켬)이다 (`picker.rs:582`, `render.rs:251-257`).
- **목록 항목(`ListItem`) 치수 및 여백**:
  - 모든 기본 Picker 항목은 `ListItem::new(ix).inset(true).spacing(ListItemSpacing::Sparse).toggle_state(selected)`를 사용한다.
  - `.inset(true)` 적용 시 바깥 `h_flex`에 좌우 패딩 `DynamicSpacing::Base04.rems(cx)` = **`4px`**가 들어가 모달 테두리에서 `4px` 떨어진다 (`list_item.rs:365-367`).
  - 내부 `inner_list_item`은 좌우 패딩 `DynamicSpacing::Base06.rems(cx)` = **`6px`**(따라서 모달 외곽 기준 좌우 총 `10px`), 모서리 반경 `rounded_sm()` = **`4px`**, 내부 요소 간격 `gap_1()` = **`4px`**를 갖는다 (`list_item.rs:378, 404-406`).
  - `ListItemSpacing::Sparse`는 상하 패딩 `.py_1()`(`4px`씩 총 `8px`)을 추가한다 (`list_item.rs:307-310`).
  - **행 높이**:
    - 1줄 항목(명령 팔레트, 파일 찾기, 테마 선택기, 아이콘 테마 선택기): 기본 줄 높이(`~20px`~`22px`) + 상하 패딩 `8px` = **`28px` ~ `30px`** (`CommandPalette`는 내부에 `.py_px()` `1px`가 추가되어 약 `30px`).
    - 2줄 항목(`RecentProjects` 모달에서 경로 표시 시): 1행 이름(`~20px`) + 2행 `LabelSize::Small` 경로(`~16px`) + 상하 패딩 `8px` = **`44px`**.
- **항목 구분선(`separators_after_indices`)**:
  - `delegate.separators_after_indices()`가 반환한 인덱스 다음에는 `ListItem::new(ix).disabled(true).child(div().w_full().border_color(border_variant).border_b_1().py(px(-1.0)))`로 `1px` `border_variant` 가로선을 그린다 (`picker.rs:1491-1501`).
- **선택·호버 강조 및 퍼지 매칭 강조**:
  - `select_on_hover()` 트레이트 메서드의 기본값은 `true`이다 (`picker.rs:213-215`). 마우스 커서가 항목 위에서 움직이면(`on_mouse_move`) 즉시 해당 항목으로 `selected_index`가 이동한다 (`picker.rs:1452-1459`).
  - 선택된 항목(`toggle_state(true)`): 배경색 `ghost_element_selected` (`list_item.rs:384-385`).
  - 호버 항목(`select_on_hover == false`일 때): 배경색 `ghost_element_hover`, 클릭 누름 시 `ghost_element_active` (`list_item.rs:380-383`).
  - 다중 선택(`marked_indices`) 항목: 왼쪽에 `2px` 너비(`w_0p5()`) 세로 막대를 `status().info`(`Color::Info`) 색상으로 표시한다 (`picker.rs:1468-1482`).
  - 퍼지 매칭 글자 강조(`HighlightedLabel`): 쿼리와 일치하는 문자 인덱스에 `cx.theme().colors().text_accent`(`Color::Accent`) 색상을 입힌다 (`highlighted_label.rs:220-237`).
- **빈 결과(`No matches`) 표시**:
  - `delegate.match_count() == 0`일 때 `delegate.no_matches_text(window, cx)`(기본값 `"No matches"`)를 표시한다 (`picker.rs:227-229`, `render.rs:260-272`).
  - 레이아웃: `v_flex().flex_grow_1().py_2()`(`상하 8px`) 안에 `ListItem::new("empty_state").inset(true).spacing(ListItemSpacing::Sparse).disabled(true).child(Label::new("No matches").color(Color::Muted))`를 렌더링한다 (`render.rs:261-271`).

### `P-005` Picker 키보드 이동 및 닫기(Dismiss) 조건
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L910-L1204`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L17-L70`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L28-L55`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/vscode/specific-overrides.json#L1-L25`
- **키 컨텍스트**: `key_context`에 `"Picker"`와 `"menu"`가 함께 설정되며, 검색 입력칸(`Editor`)에 포커스가 있다 (`render.rs:20-24`).
- **키보드 바인딩 표 (Windows / VSCode 기본 키맵 기준)**:

| 동작 | 액션 이름 | 기본 키 바인딩 (Windows) | 세부 동작 (`picker.rs`) |
|---|---|---|---|
| 이전 항목 선택 | `menu::SelectPrevious` / `editor::MoveUp` | `Up`, `Ctrl+P`, `Shift+Tab`, `PageUp` | `selected_index`를 1 감소 (`0`에서 누르면 마지막 항목 `count - 1`로 순환, `can_select == false`인 헤더는 건너뜀) (`L910-965`) |
| 다음 항목 선택 | `menu::SelectNext` / `editor::MoveDown` | `Down`, `Ctrl+N`, `Tab`(단일 선택 시), `PageDown` | `selected_index`를 1 증가 (`count - 1`에서 누르면 첫 항목 `0`으로 순환, 헤더는 건너뜀) (`L967-1020`) |
| 첫 항목 선택 | `menu::SelectFirst` | `Home` (`menu` 컨텍스트) | 선택 가능한 첫 번째 항목으로 이동 (`L1052-1069`) |
| 마지막 항목 선택 | `menu::SelectLast` | `End` (`menu` 컨텍스트) | 선택 가능한 마지막 항목으로 이동 (`L1071-1089`) |
| 기본 확정 | `menu::Confirm` / `picker::ConfirmCompletion` | `Enter` | `delegate.confirm(false, window, cx)` 호출 (`L1022-1035`) |
| 보조 확정 | `menu::SecondaryConfirm` | `Alt+Enter` (`default-windows.json:33`), `Ctrl+Enter` (`Picker > Editor`) | `delegate.confirm(true, window, cx)` 호출 (새 창 열기, 키 바인딩 변경 등) (`L1037-1045`) |
| 자동완성 텍스트 적용 | `picker::ConfirmInput` | `Ctrl+Alt+Enter` (`default-windows.json:835`) | `delegate.confirm_input(false, window, cx)` 호출 (`L1150-1161`) |
| 취소 / 닫기 | `menu::Cancel` | `Escape` | `delegate.dismissed(window, cx)` 호출 후 `cx.emit(DismissEvent)` 발생 (`L1047-1050`) |

- **닫기(Dismiss) 발동 조건 4가지**:
  1. `Escape` 키 (`menu::Cancel` 액션, `picker.rs:1047-1050`).
  2. 모달 바깥 영역 마우스 왼쪽 클릭 (`modal_layer.rs:308-313`).
  3. 검색 입력칸 포커스 상실(`ErasedEditorEvent::Blurred`): `draws_own_container()`가 `true`이고 창이 활성 상태(`window.is_window_active()`)이며 하위 메뉴(`menu_focused`)로 포커스가 옮겨간 것이 아닐 때 자동으로 `cancel(&menu::Cancel, window, cx)`를 호출해 닫는다 (`picker.rs:1195-1203`).
  4. 같은 토글 단축키 재입력(예: `Ctrl+Shift+P`로 명령 팔레트가 열린 상태에서 다시 `Ctrl+Shift+P` 입력 시 `workspace.toggle_modal`이 기존 모달을 닫음, `modal_layer.rs:108-122`).

### `P-006` Picker 공통 색상 토큰 매핑표
- **출처**: `crates/picker/src/render.rs`, `crates/ui/src/styles/elevation.rs`, `crates/ui/src/components/list/list_item.rs`, `crates/ui/src/components/label/highlighted_label.rs`

| UI 요소 | 테마 색상 토큰 (`ThemeColors`) | 비고 |
|---|---|---|
| 모달 표면 배경 | `elevated_surface_background` | `elevation_3` (`ElevationIndex::ModalSurface`) |
| 모달 외곽 테두리 (`1px`) | `border_variant` | `rounded_lg` (`8px`) |
| 검색창 하단 / 푸터 상단 / 그룹 구분선 (`1px`) | `border_variant` | `Divider::horizontal()` |
| 검색 입력 텍스트 / 커서 | `text` / `players[0].cursor` | `Editor::single_line` |
| 검색 입력 placeholder | `text_placeholder` | 쿼리가 비어 있을 때 표시 |
| 항목 기본 배경 | 투명 (`transparent_black`) | 모달의 `elevated_surface_background` 투과 |
| 선택된 항목 배경 (`selected == true`) | `ghost_element_selected` | `select_on_hover == true`이므로 마우스 이동 시 즉시 따라옴 |
| 항목 호버 배경 (`select_on_hover == false` 시) | `ghost_element_hover` | 마우스 누름(`active`) 시 `ghost_element_active` |
| 항목 기본 텍스트 | `text` (`Color::Default`) | 주 레이블(명령어 이름, 파일명, 폴더명, 테마명) |
| 항목 보조 텍스트 (경로, 브랜치, 빈 결과) | `text_muted` (`Color::Muted`) | `No matches` 포함 |
| 퍼지 매칭 강조 글자 | `text_accent` (`Color::Accent`) | `HighlightedLabel` 일치 문자 인덱스 |
| 항목 오른쪽 보조 아이콘 | `icon_muted` (`Color::Muted`) | `HistoryRerun`, `Check`, `Close` 등 |

---

## 2. 개별 Picker 상세 비교 및 명세 (`P-007` ~ `P-012`)

| 구분 | 명령 팔레트 (`P-007`) | 파일 찾기 (`P-008`) | 최근 폴더 (`P-009`) | 테마 선택기 (`P-010`) | 아이콘 테마 선택기 (`P-011`) | 줄 이동 (`P-012`) |
|---|---|---|---|---|---|---|
| 크레이트 / 구조체 | `command_palette::CommandPalette` | `file_finder::FileFinder` | `recent_projects::RecentProjects` | `theme_selector::ThemeSelector` | `theme_selector::IconThemeSelector` | `go_to_line::GoToLine` |
| 기반 위젯 | `Picker<UniformList>` | `Picker<UniformListWithPreview>` | `Picker<List>` (가변 높이) | `Picker<UniformList>` | `Picker<UniformList>` | 자체 `ModalView` (`Picker` 미사용) |
| 모달 너비 | `38rem` = **`608px`** | 기본 `34rem` = **`544px`** (`modal_max_width` 설정 가능) | 모달 `42rem` = **`672px`** / 팝오버 `20rem` = **`320px`** | `34rem` = **`544px`** | `34rem` = **`544px`** | `24rem` = **`384px`** |
| 최대 높이 | `24rem` = **`384px`** | `24rem` = **`384px`** (미리보기 꺼짐 시) | `24rem` = **`384px`** | `24rem` = **`384px`** | `24rem` = **`384px`** | 입력칸+안내줄 2단 고정 (약 **`62px`**) |
| 상단 별도 제목 헤더 | 없음 (`head`가 검색 입력칸) | 없음 | 없음 (목록 내 섹션 서브헤더 존재) | 없음 | 없음 | 없음 |
| Placeholder 문구 | `"Execute a command..."` | `"Search project files..."` | `"Search projects…"` (U+2026) | `"Select Theme..."` | `"Select Icon Theme..."` | `"{line}:{column}"` (현재 커서 1-based 좌표) |
| 오른쪽 표시 요소 | 해당 액션의 `KeyBinding` + 히스토리 항목 시 `HistoryRerun`(호버 시 `Close` 버튼) | 히스토리 항목 시 `HistoryRerun` 아이콘(`14px`), 일반 검색 시 `14px` 빈 여백 | 호버 시 우측 액션 아이콘 버튼들(`FolderInclude`, `ArrowUpRight`, `Close`) | 원래 활성 테마 항목에만 `Check` 아이콘(`Color::Muted`) | 원래 활성 아이콘 테마 항목에만 `Check` 아이콘(`Color::Muted`) | 없음 |
| 실시간 미리 적용 여부 | 없음 | 푸터에서 `Preview` 토글 시 파일 미리보기 창 표시 | 없음 | **있음** (상하 이동·마우스 호버 즉시 인메모리 테마 교체, `Esc` 시 원복) | **있음** (상하 이동·마우스 호버 즉시 인메모리 아이콘 테마 교체, `Esc` 시 원복) | **있음** (입력 즉시 대상 줄로 스크롤 및 줄 배경 강조, `Esc` 시 스크롤·강조 원복) |

### `P-007` 명령 팔레트 (`CommandPalette`)
- **출처**: `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/command_palette/src/command_palette.rs#L147-L154`, `#L520-L868`
- **호출 액션 및 기본 키**: `command_palette::Toggle` — `Ctrl+Shift+P` 또는 `F1` (`default-windows.json:605`, `linux/vscode.json:103`).
- **치수 및 설정**:
  - `.initial_width(rems(38.0))` = **`608px`**, 최대 높이 기본값 `24rem = 384px`, `.show_scrollbar(true)`, `.reopenable(false, cx)` (`command_palette.rs:147-154`).
- **Placeholder**: `"Execute a command..."` (`command_palette.rs:528`).
- **항목 구조 (`render_match`, `command_palette.rs:768-815`)**:
  - `ListItem::new(ix).inset(true).spacing(ListItemSpacing::Sparse).toggle_state(selected)` 내부에 `h_flex().w_full().py_px().justify_between().gap_2()` 배치.
  - **왼쪽**: 사람이 읽기 좋게 변환된 명령어 이름(예: `workspace: save`, `theme selector: toggle`)을 `HighlightedLabel::new(command.name, positions). truncate()`로 표시 (`command_palette.rs:791-795`).
  - **오른쪽**:
    - 팔레트를 열기 직전 포커스 핸들(`previous_focus_handle`) 기준 단축키 배지 `KeyBinding::for_action_in(&*command.action, &self.previous_focus_handle, cx)` (`command_palette.rs:796-800`).
    - 최근 사용한 명령어(`command.usage.is_some()`)인 경우 오른쪽 끝에 `IconName::HistoryRerun`(`Color::Muted`, `IconSize::Small` = `14px`)을 표시하고, 해당 행에 마우스를 올리면(`visible_on_hover`) `IconButton::new("remove-from-history", IconName::Close)`(툴팁 `"Remove from Command History"`, 액션 `RemoveSelected` = `Delete`)로 교체된다 (`command_palette.rs:774-812`).
- **하단 푸터 (`render_footer`, `command_palette.rs:817-868`)**:
  - `h_flex().w_full().p_1p5().gap_1().justify_end().border_t_1().border_color(border_variant)` (`p_1p5()` = `6px`, `gap_1()` = `4px`).
  - 오른쪽 정렬 버튼 2개:
    1. 기존 키 바인딩 유무에 따라 `"Change Keybinding…"` 또는 `"Add Keybinding…"` 버튼 (`menu::SecondaryConfirm` = `Alt+Enter`, 클릭 시 `zed_actions::ChangeKeybinding { action }` 디스패치).
    2. `"Run"` 버튼 (`menu::Confirm` = `Enter`).
  - **우리 앱 적용 기준**: 키맵 편집기 화면을 두지 않는 초기 버전에서는 `"Run"` 버튼만 표시하거나 푸터를 생략할 수 있다(판단).

### `P-008` 파일 찾기 (`FileFinder` / Go to File)
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L100-L105`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L205-L249`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L358-L365`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L1945-L2360`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/footer.rs#L95-L226`
- **호출 액션 및 기본 키**: `file_finder::Toggle` — `Ctrl+P` (`default-windows.json:608`, `linux/vscode.json:101`).
- **치수**:
  - `file_finder.modal_max_width` 설정이 기본값(`small`)일 때 `DEFAULT_MODAL_WIDTH` = `34rem` = **`544px`** (`file_finder.rs:358-365`).
  - `reopenable: true`, `supports_multi_select: true` (`file_finder.rs:212-215`).
- **Placeholder 및 쿼리 문법**:
  - `"Search project files..."` (`file_finder.rs:1953`).
  - 파일명 뒤에 `:줄번호` 또는 `:줄번호:열번호`를 붙여 검색하면 해당 파일을 열면서 즉시 지정한 줄·열로 이동한다 (`file_finder.rs:480-520`).
- **검색창 우측 트레일러 (`render_searchbar_trailer`, `file_finder.rs:1956-1991`)**:
  - `IconButton::new("toggle_ignored_files", IconName::FileIgnored)` (`IconSize::Small` = `14px`, 선택 시 `ButtonStyle::Tinted(TintColor::Accent)`, 툴팁 `"Include Ignored Files"` / `"Exclude Ignored Files"`, 액션 `file_finder::ToggleIncludeIgnored`) + 프로젝트 인덱싱 중일 때 회전하는 `IconName::LoadCircle`(`Color::Accent`, `IconSize::Small`) + 다중 선택 토글 버튼(`IconName::FileMultiple`).
- **항목 구조 (`render_match_impl`, `file_finder.rs:2281-2360`)**:
  - `ListItem::new(ix).spacing(ListItemSpacing::Sparse).inset(true).toggle_state(selected)`:
    - `start_slot`: `file_finder.file_icons` 설정(기본 `true`)에 따라 파일 확장자 아이콘 `FileIcons::get_icon`을 `Icon::from_path(icon).color(Color::Muted)`(`16px`)로 렌더링 (절대 경로 외부 파일은 `IconName::FileLock` 또는 `IconName::File`) (`file_finder.rs:2262-2279`, `2343`).
    - **본문(`h_flex().gap_1p5()`)**:
      1. **파일명**: `HighlightedLabel` (`Color::Default`, `.truncate_middle()`, 퍼지 매칭 위치는 `Color::Accent`).
      2. **상대 경로**: `HighlightedLabel` (`LabelSize::Small` = `14px`, `Color::Muted`, `.truncate_start()`, 다중 루트 폴더일 때는 루트 폴더명 접두사 포함) (`file_finder.rs:2299-2339`).
    - `end_slot`: 최근 열었던 히스토리 항목(`Match::History`)이면 `IconName::HistoryRerun`(`Color::Muted`, `IconSize::Small` = `14px`), 일반 검색 결과(`Match::Search`)이면 정렬 유지를 위해 `div().w(IconSize::Small.rems())`(`14px` 빈 공간)를 둔다 (`file_finder.rs:2344-2356`).
  - **히스토리 구분선**: 히스토리 항목과 일반 검색 항목 사이에 `1px` `border_variant` 구분선을 표시한다 (`separators_after_indices`, `file_finder.rs:2022-2034`).
- **하단 푸터 (`footer.rs:95-226`)**:
  - 왼쪽: `"Preview"` 토글 버튼(`picker::TogglePreview`) + 미리보기 켜짐 시 레이아웃 전환 아이콘 버튼(`DiffUnified` = `"Preview Below"`, `DiffSplit` = `"Preview to the Right"`).
  - 오른쪽: `"Actions…"` 팝오버 메뉴 버튼(`Split…` → `Left`, `Right`, `Up`, `Down`, 구분선, `Open File`).
- **`Ctrl+P` 홀드 순환 동작**: `Ctrl` 키를 누른 채 `P`를 연속 입력하면 다음 항목으로 이동하고, `Ctrl` 키를 떼는 순간(`ModifiersChangedEvent`) 선택된 파일을 확정해 연다 (`file_finder.rs:100-105`, `234-249`).

### `P-009` 최근 폴더 / 프로젝트 선택기 (`RecentProjects`)
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L653-L851`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L925-L1990`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/highlighted_match_with_paths.rs#L66-L134`
- **호출 액션 및 기본 키**: `projects::OpenRecent` — `Ctrl+Alt+O` (`default-windows.json:634`), 또는 사이드바 상단 `+` (`IconName::FolderAdd`, `"Add Project"`) 버튼 클릭 시 팝오버 형태로 호출 (`sidebar.rs:6758-6782`).
- **두 가지 스타일(`ProjectPickerStyle`) 및 너비**:
  - **모달(`ProjectPickerStyle::Modal`)**: `.initial_width(rems(42.0))` = **`672px`**, `render_paths = true` (폴더명 아래에 `LabelSize::Small` `Color::Muted` 경로 줄을 함께 표시) (`recent_projects.rs:761-784`, `877`).
  - **팝오버(`ProjectPickerStyle::Popover`, 사이드바 `+` 버튼)**: `.initial_width(rems(20.0))` = **`320px`**, `.max_height(Some(rems(20.0).into()))` = **`320px`**, `render_paths = false` (폴더명만 1줄로 표시하고 전체 경로는 호버 툴팁으로 표시) (`recent_projects.rs:668-693`, `877`).
- **Placeholder 및 빈 결과 문구**:
  - Placeholder: `"Search projects…"` (`recent_projects.rs:932-934`).
  - 빈 결과(`no_matches_text`, `recent_projects.rs:1218-1225`): 등록된 프로젝트와 열린 폴더가 하나도 없을 때는 `"Recently opened projects will show up here"`, 검색어와 일치하는 항목이 없을 때는 `"No matches"`.
- **목록 3단 섹션 구조 (`update_matches`, `recent_projects.rs:1043-1125`, `1235-1242`)**:
  - 각 섹션 헤더는 `ListSubHeader::new(title).inset(true)`로 렌더링되며(`can_select == false`), 두 번째 섹션부터는 위에 `mt_1()`(`4px`) 여백과 `Divider::horizontal()`(`1px` `border_variant`)이 들어간다:
    1. **`"Current Folders"` (`ProjectPickerEntry::OpenFolder`)**: 현재 워크스페이스에 열려 있는 작업 폴더들.
       - 항목 좌측: 폴더명(`HighlightedLabel`) + Git 브랜치명(`Label`, `Color::Muted`) + 활성 폴더일 때 `IconName::Check`(`IconSize::Small` = `14px`, `Color::Accent`) + (모달일 때 2행에 축약 경로 `LabelSize::Small`, `Color::Muted`) (`recent_projects.rs:1304-1331`).
       - 항목 우측(호버 시 표시 `show_end_slot_on_hover`): `IconName::Close` 버튼(툴팁 `"Remove Folder from Project"`, 액션 `RemoveSelected` = `Delete`) (`recent_projects.rs:1253-1278`).
    2. **`"This Window"` (`ProjectPickerEntry::ProjectGroup`)**: 현재 창(멀티 워크스페이스)에 열려 있는 프로젝트 그룹들.
       - 항목 좌측: `HighlightedMatchWithPaths` (활성 그룹이면 우측에 `IconName::Check` `Color::Accent` `14px` 표시, `highlighted_match_with_paths.rs:104-110`).
       - 항목 우측(호버 시 표시): `"Open in New Window"`(`IconName::ArrowUpRight`, `SecondaryConfirm`) + `"Remove Project from Window"`(`IconName::Close`, `RemoveSelected`) (`recent_projects.rs:1387-1447`).
    3. **`"Recent Projects"` (`ProjectPickerEntry::RecentProject`)**: 최근에 열었던 프로젝트 기록.
       - 항목 좌측: `HighlightedMatchWithPaths` (폴더명 + 모달일 때 2행에 경로).
       - 항목 우측(호버 시 표시, `recent_projects.rs:1549-1621`):
         - 로컬 폴더를 현재 프로젝트에 멀티루트로 추가하는 `IconName::FolderInclude` 버튼(툴팁 `"Add Folder to this Project"`, 부제 `"As a multi-root folder"`).
         - 새 창/현재 창 전환 열기 버튼(`IconName::ArrowUpRight` 또는 `IconName::ThisWindow`, `menu::SecondaryConfirm`).
         - 최근 목록에서 삭제하는 `IconName::Close` 버튼(툴팁 `"Remove from Recent Projects"`, `RemoveSelected` = `Delete`).
- **하단 푸터 (`render_footer`, `recent_projects.rs:1668-1990`)**:
  - **팝오버 스타일일 때**: `v_flex().p_1p5().gap_1().border_t_1().border_color(border_variant)` 안에 전체 너비 `ButtonLike` 항목 `"Open Local Folders"`(`workspace::Open`, `Ctrl+O` / `Ctrl+K Ctrl+O`)와(원본 Zed는 `"Open Remote Folder"` 포함, 우리 앱은 로컬 전용이므로 `"Open Local Folders"`만 유지) 표시 (`recent_projects.rs:1688-1757`).
  - **모달 스타일일 때**: `h_flex().p_1p5().gap_1().justify_end().border_t_1().border_color(border_variant)` 안에 선택된 항목 종류에 따라 `"Remove Folder"` / `"Remove"`(`Delete`) 버튼 + `"New Window"`(`Alt+Enter`) + `"Open"` 또는 `"Activate"`(`Enter`) + 수직 구분선(`Divider::vertical()`) + `"Actions"` 팝오버 메뉴(`"Add Folder to this Project"`, `"Open Local Folders"`) 표시 (`recent_projects.rs:1770-1988`).

### `P-010` 테마 선택기 (`ThemeSelector`)
- **출처**: `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L107-L125`, `#L175-L185`, `#L359-L570`
- **호출 액션 및 기본 키**: `theme_selector::Toggle` — `Ctrl+K Ctrl+T` (`default-windows.json:625`, `linux/vscode.json:112`).
- **치수**: 바깥 래퍼 `v_flex().key_context("ThemeSelector").w(rems(34.))` = **`544px`**, 최대 높이 기본값 `24rem = 384px` (`theme_selector.rs:116-125`).
- **Placeholder**: `"Select Theme..."` (`theme_selector.rs:386`).
- **정렬 및 초기 선택**:
  - 다크(`Appearance::Dark`) 테마를 먼저 오름차순 정렬하고, 이어서 라이트(`Appearance::Light`) 테마를 오름차순 정렬한다 (`a.appearance.is_light().cmp(&b.appearance.is_light()).then(a.name.cmp(&b.name))`, `theme_selector.rs:179-183`).
  - 모달을 열 때 현재 적용 중인 테마(`original_theme`)의 인덱스가 초기 선택된다 (`theme_selector.rs:372-377`).
- **항목 구조 (`render_match`, `theme_selector.rs:505-528`)**:
  - `ListItem::new(ix).inset(true).spacing(ListItemSpacing::Sparse).toggle_state(selected)`:
    - 좌측: 테마 이름 `HighlightedLabel::new(theme_match.string, theme_match.positions)`.
    - 우측(`end_slot`): 모달을 열 당시 활성 상태였던 원래 테마(`self.is_original_theme(ix)`)에만 `Icon::new(IconName::Check).color(Color::Muted)`(`16px`)를 표시한다 (`theme_selector.rs:519-524`).
- **실시간 미리 적용(Live Preview) 및 복원 동작**:
  - 키보드 상하 이동 또는 마우스 호버(`select_on_hover == true`)로 `set_selected_index`가 호출될 때마다 `show_selected_theme(cx)`를 실행해 `ThemeSettings::override_global_theme`으로 앱 전체 테마를 메모리에서 즉시 교체한다 (`theme_selector.rs:314-357`, `403-413`).
  - `Enter` 또는 클릭(`confirm`) 시에만 `theme_settings::set_theme`을 호출하여 `settings.json`에 영구 저장하고 모달을 닫는다 (`theme_selector.rs:463-489`).
  - `Escape` 또는 외부 클릭으로 닫힐 때(`on_before_dismiss` 및 `dismissed`)는 `revert_theme(cx)`를 호출해 `original_theme`으로 즉시 되돌린다 (`theme_selector.rs:107-112`, `491-503`).
- **하단 푸터 (`render_footer`, `theme_selector.rs:530-569`)**:
  - 원본 Zed는 `h_flex().p_2().w_full().justify_between().gap_2().border_t_1().border_color(border_variant)` 안에 좌측 `"View Theme Docs"`(`IconName::ArrowUpRight`)와 우측 `"Install Themes"` 버튼을 둔다.
  - **우리 앱 적용 기준**: 확장 스토어를 포함하지 않으므로 푸터를 생략하거나 내장 32개 테마 안내 레이블만 둘 수 있다 (`zed-theme-icons.md` `T-21`과 동일).

### `P-011` 아이콘 테마 선택기 (`IconThemeSelector`)
- **출처**: `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L42-L60`, `#L162-L361`
- **호출 액션**: `icon_theme_selector::Toggle` (명령 팔레트에서 `icon theme selector: toggle`로 실행, 기본 키 바인딩 없음).
- **치수 및 레이아웃**: `v_flex().key_context("IconThemeSelector").w(rems(34.))` = **`544px`**, 최대 높이 `24rem = 384px` (`icon_theme_selector.rs:51-60`).
- **Placeholder**: `"Select Icon Theme..."` (`icon_theme_selector.rs:170`).
- **항목 구조 및 실시간 미리 적용 (`icon_theme_selector.rs:187-320`)**:
  - `ThemeSelector`와 완전히 동일한 구조(`ListItem`, `inset(true)`, `ListItemSpacing::Sparse`, 원래 활성 아이콘 테마 우측에 `IconName::Check` `Color::Muted`).
  - 항목 이동·호버 시 `show_selected_icon_theme(cx)`가 인메모리 아이콘 테마를 즉시 교체하여 프로젝트 패널·탭의 아이콘이 실시간으로 바뀌며, `Esc`/외부 클릭 시 `original_icon_theme`으로 복원하고 `Enter`/클릭 시 `theme_settings::set_icon_theme`으로 저장한다.

### `P-012` 줄 이동 모달 (`GoToLine`)
- **출처**: `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L78-L349`
- **호출 액션 및 기본 키**: `editor::ToggleGoToLine` — `Ctrl+G` (`default-windows.json:210`, `linux/vscode.json:100`) 또는 상태바 우측 `줄:열` 버튼 클릭 (`cursor_position.rs:233-246`).
- **구조적 특징 (`Picker` 미사용)**:
  - 결과 목록이 필요 없으므로 `Picker<D>`를 쓰지 않고 직접 `ModalView`를 구현한다 (`go_to_line.rs:78`).
  - 컨테이너: `v_flex().key_context("GoToLine").w(rems(24.)).elevation_2(cx)` → 너비 `24rem` = **`384px`**, `elevation_2`(`ElevatedSurface` 그림자, `elevated_surface_background` 배경, `rounded_lg` = `8px`, `1px` `border_variant` 테두리) (`go_to_line.rs:329-333`).
- **상단 입력칸 + 하단 상태 문구 2단 구조 (`go_to_line.rs:334-348`)**:
  1. **상단 입력 영역**: `div().border_b_1().border_color(border_variant).px_2().py_1().child(self.line_editor)` (`px_2()` = `8px`, `py_1()` = `4px`, 하단 `1px` `border_variant` 구분선).
     - Placeholder는 현재 커서의 1-based 좌표 문자열 `"{line}:{column}"`(예: `"42:10"`)이다 (`go_to_line.rs:94-103`).
     - 입력칸이 비어 있을 때 `Tab`(`editor::AcceptEditPrediction`)을 누르면 placeholder 문자열이 그대로 입력칸 텍스트로 채워진다 (`go_to_line.rs:112-125`).
  2. **하단 안내 레이블 영역**: `h_flex().px_2().py_1().gap_1().child(Label::new(help_text).color(Color::Muted))` (`px_2()` = `8px`, `py_1()` = `4px`).
     - 입력값(`TargetLocation`)에 따른 정확한 안내 문구 (`go_to_line.rs:312-327`):
       - 입력이 비어 있거나 파싱 불가할 때: `"Current Line: {line} of {total_lines} (column {column})"`
       - 줄 번호만 입력(`42` 또는 `:42`): `"Go to line {line}"`
       - 줄과 열을 입력(`42:10` 또는 `42:10:`): `"Go to line {line}, character {character}"`
       - 상대 줄 오프셋 입력(`+5`, `-3`, `f5`, `b3`): `"Go to line {target_line} ({offset:+} from current)"`
- **실시간 줄 강조·스크롤 및 취소 복원 (`go_to_line.rs:154-210`, `288-305`)**:
  - 입력 내용이 바뀔 때마다(`EditorEvent::BufferEdited`) 대상 줄을 계산해 편집기에서 `editor.highlight_rows(...)`(`editor_highlighted_line_background` 색상)로 강조하고 `Autoscroll::center()`로 화면 중앙에 스크롤한다 (`go_to_line.rs:182-210`).
  - `Enter`(`menu::Confirm`) 입력 시: `confirmed = true`로 표시하고 커서를 대상 위치로 이동시킨 뒤 접힌 코드 블록(fold)을 펼치고 편집기에 포커스를 돌려준다 (`go_to_line.rs:288-305`).
  - `Escape`(`menu::Cancel`) 또는 외부 클릭으로 닫힐 때: `Drop` 구현(`release`)에서 `confirmed == false`이면 줄 강조(`clear_row_highlights`)를 지우고 모달을 열기 전의 스크롤 위치(`prev_scroll_position`)로 정확히 되돌린다 (`go_to_line.rs:154-163`).

---

## 3. 대화상자 (Prompt / Modal / About 창) 명세 (`P-013` ~ `P-018`)

### `P-013` 프롬프트 렌더링 아키텍처 (Windows 네이티브 `TaskDialogIndirect` vs Zed 내장 `ZedPromptRenderer`)
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui_prompt/src/ui_prompt.rs#L20-L181`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L293`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui_windows/src/window.rs#L723-L815`
- **동작 분기 (`ui_prompt.rs:20-27`)**:
  - Zed는 `window.prompt(level, message, detail, answers, cx)` API로 모든 확인 대화상자를 호출한다.
  - `settings/default.json:293`의 기본값은 `"use_system_prompts": true`이다.
  - **모드 A — Windows 시스템 기본 프롬프트 (`use_system_prompts: true`, 기본값)**:
    - `gpui_windows/src/window.rs:738-796`에서 Win32 네이티브 `TaskDialogIndirect`(`TASKDIALOGCONFIG`)를 호출한다.
    - 창 제목(`pszWindowTitle`)과 아이콘(`pszMainIcon`):
      - `PromptLevel::Info` → 제목 `"Info"`, 아이콘 `TD_INFORMATION_ICON`
      - `PromptLevel::Warning` → 제목 `"Warning"`, 아이콘 `TD_WARNING_ICON`
      - `PromptLevel::Critical` → 제목 `"Critical"`, 아이콘 `TD_ERROR_ICON`
    - 주 문구(`msg`)는 `pszMainInstruction`에, 상세 문구(`detail`)는 `pszContent`에 들어가며, 버튼 배열(`answers`)은 순서대로 `TASKDIALOG_BUTTON`으로 배치된다(첫 번째 버튼이 기본 포커스, `"Cancel"` 레이블 버튼은 `IDCANCEL`에 매핑되어 `Esc` 키로 선택됨, `TDF_ALLOW_DIALOG_CANCELLATION`).
  - **모드 B — Zed 내장 GPUI 프롬프트 모달 (`use_system_prompts: false` 설정 시 `ZedPromptRenderer`, `ui_prompt.rs:111-181`)**:
    - 전체 창 오버레이: `v_flex().occlude().absolute().inset_0().bg(gpui::black().opacity(0.2)).items_center().justify_center()` (화면 정중앙 배치, 배경은 검은색 `20%` 불투명도 딤 처리) (`ui_prompt.rs:162-179`).
    - 대화상자 본체: `.w_80()` = `20rem` = **`320px` 고정 너비**, 패딩 `.p_4()` = **`16px`**, 요소 간격 `.gap_4()` = **`16px`**, `.elevation_3(cx)`(`elevated_surface_background`, `rounded_lg` = `8px`, `1px` `border_variant` 테두리, 4중 그림자) (`ui_prompt.rs:125-130`).
    - 본문 텍스트: 주 문구는 `ui_font_size`(`16px`) `Color::Default`(`text`)의 `MarkdownElement`(따라서 백틱 `` `파일명` ``이 인라인 코드로 렌더링됨), 상세(`detail`)는 `text_xs()`(`12px`) `Color::Muted`(`text_muted`)의 `MarkdownElement` (`ui_prompt.rs:131-140`, `183-206`).
    - 버튼 배치: 하단에 **수직 스택(`v_flex().gap_1()`, 간격 `4px`)**으로 각 버튼을 전체 너비(`Button::new(ix, label).full_width()`)로 쌓는다 (`ui_prompt.rs:142-156`).
    - 기본 버튼 및 스타일: 인덱스 `0`번 버튼(`active_action_id = 0`)이 기본 선택되어 `ButtonStyle::Tinted(TintColor::Accent)`(`info_background` + `info_border`)로 강조되고, 나머지 버튼은 `ButtonStyle::Outlined`(`element_background` + `border_variant`)로 렌더링된다 (`ui_prompt.rs:46`, `147-150`).
    - 키보드 단축키 (`ui_prompt.rs:65-108`, `default-windows.json`):
      - `Enter` (`menu::Confirm`): 현재 선택된 버튼(`active_action_id`) 실행.
      - `Escape` (`menu::Cancel`): 버튼 목록 중 `"Cancel"`과 일치하는 버튼 인덱스를 찾아 실행.
      - `Down` / `Up` / `Tab` / `Shift+Tab` (`menu::SelectNext` / `menu::SelectPrevious`): 활성 버튼 인덱스 순환.

### `P-014` 저장 확인 대화상자 (탭 닫기 / 다중 탭 닫기 / 창 닫기 / 저장 실패)
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2006-L2073`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2401-L2435`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L5067-L5082`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L4028-L4045`
- **단일 파일 미저장 탭 닫기 (`pane.rs:2411-2418`, `5067-5082`)**:
  - 레벨: `PromptLevel::Warning`.
  - 주 문구 (`dirty_message_for`):
    - 경로가 있는 파일(앞부분을 잘라 최대 80자): ``"`{path}` contains unsaved edits. Do you want to save it?"``
    - 경로가 없는 새 버퍼(Untitled): `"This buffer contains unsaved edits. Do you want to save it?"`
  - 상세 문구(`detail`): `None`.
  - 버튼 배열 (순서대로, `0`번이 기본 버튼): **`["Save", "Don't Save", "Cancel"]`**.
- **다중 미저장 파일 한꺼번에 닫기 (`pane.rs:2006-2023`)**:
  - 조건: `dirty_items.len() > 1`일 때 개별 프롬프트 전에 먼저 표시.
  - 레벨: `PromptLevel::Warning`.
  - 주 문구: `"Do you want to save changes to the following files?"`
  - 상세 문구(`detail`): 변경된 파일 이름 목록(줄바꿈 구분).
  - 버튼 배열: **`["Save all", "Discard all", "Cancel"]`** (`0`번 `"Save all"`이 기본 버튼).
- **창 닫기 / 앱 종료 시 미저장 확인 (`workspace.rs:4028-4045`)**:
  - 레벨: `PromptLevel::Warning`.
  - 주 문구: `"Do you want to save changes to the following files?"`
  - 상세 문구(`detail`): 미저장 파일 목록 (`Pane::file_names_for_prompt`).
  - 버튼 배열: **`["Save", "Don't Save", "Cancel"]`** (`0`번 `"Save"`가 기본 버튼).
- **파일 저장 실패 시 경고 (`pane.rs:2054-2072`)**:
  - 레벨: `PromptLevel::Warning`.
  - 주 문구: `"Unable to save file: {err}"`
  - 상세 문구(`detail`): 해당 파일명.
  - 버튼 배열: **`["Close Without Saving", "Cancel"]`**.

### `P-015` 삭제 · 휴지통 이동 확인 대화상자 (`ProjectPanel::build_removal_prompt`)
- **출처**: `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2836-L2956`
- **호출 액션 및 기본 키 (Windows VSCode 키맵 기준)**:
  - `project_panel::Trash` (휴지통으로 이동): `Delete` 키 (`default-windows.json:898`: `"delete": ["project_panel::Trash", { "skip_prompt": false }]`).
  - `project_panel::Delete` (영구 삭제): `Shift+Delete` 키 (`default-windows.json:899`: `"shift-delete": ["project_panel::Delete", { "skip_prompt": false }]`).
- **레벨**: `PromptLevel::Info` (`project_panel.rs:2950`).
- **휴지통 이동 (`RemovalKind::Trash`)**:
  - 주 문구:
    - 단일 항목: ``"Do you want to trash `{name}`?"``
    - 다중 항목(`N`개, 최대 10개까지 줄바꿈 나열 후 초과분은 `".. 1 file not shown"` 또는 `".. {omitted_count} files not shown"`):
      `"Do you want to trash the following {N} files?\n`{name1}`\n`{name2}`..."`
    - 삭제 대상에 미저장 버퍼(`dirty_buffers`)가 포함된 경우 주 문구 끝에 두 줄 개행(`\n\n`) 후 덧붙임 (`project_panel.rs:2876-2889`):
      - 단일 항목이면서 미저장: `"\n\nIt has unsaved changes, which will be lost."`
      - 다중 항목 중 1개 미저장: `"\n\n1 of these has unsaved changes, which will be lost."`
      - 다중 항목 중 `K`개 미저장: `"\n\n{K} of these have unsaved changes, which will be lost."`
  - 상세 문구(`detail`): `None`.
  - 버튼 배열: **`["Trash", "Cancel"]`** (`0`번 `"Trash"`가 기본 버튼).
- **영구 삭제 (`RemovalKind::Delete`)**:
  - 주 문구:
    - 단일 항목: ``"Are you sure you want to permanently delete `{name}`?"`` (+ 미저장 시 동일한 경고 문구 덧붙임)
    - 다중 항목: `"Are you sure you want to permanently delete the following {N} files?\n..."`
  - 상세 문구(`detail`): **`Some("This cannot be undone.")`** (`project_panel.rs:2849`).
  - 버튼 배열: **`["Delete", "Cancel"]`** (`0`번 `"Delete"`가 기본 버튼).

### `P-016` 덮어쓰기 · 디스크 충돌 · 외부 삭제 확인 대화상자
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2263-L2265`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2330-L2399`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L5078-L5104`
- **편집 중 디스크의 파일이 외부에서 변경된 상태에서 저장할 때 (`CONFLICT_MESSAGE`, `pane.rs:2263, 2369-2375`)**:
  - 레벨: `PromptLevel::Warning`.
  - 주 문구: `"This file has changed on disk since you started editing it. Do you want to overwrite it?"`
  - 상세 문구(`detail`): `None`.
  - 버튼 배열: **`["Overwrite", "Discard Edits", "Cancel"]`** (`0`번 `"Overwrite"`가 기본 버튼).
- **편집 중 디스크의 파일이 외부에서 삭제된 상태에서 저장할 때 (`DELETED_MESSAGE`, `pane.rs:2265, 2334-2340`)**:
  - 레벨: `PromptLevel::Warning`.
  - 주 문구: `"This file has been deleted on disk since you started editing it. Do you want to recreate it?"`
  - 상세 문구(`detail`): `None`.
  - 버튼 배열: **`["Save", "Close", "Cancel"]`** (`0`번 `"Save"`가 기본 버튼).
- **프로젝트 패널에 외부 파일 드롭 시 동명 파일 덮어쓰기 확인 (`project_panel.rs:5078-5096`)**:
  - 레벨: `PromptLevel::Info`.
  - 주 문구: ``"A file or folder with name `{filename}` already exists in the destination folder. Do you want to replace it?"``
  - 상세 문구(`detail`): `None`.
  - 버튼 배열: **`["Replace", "Cancel"]`** (`0`번 `"Replace"`가 기본 버튼).

### `P-017` Git 변경사항 되돌리기(`git::RestoreFile`) 확인 대화상자
- **출처**: `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2669-L2674`
- **조건**: 프로젝트 패널에서 수정된 파일에 대해 `git::RestoreFile` 실행 시(`skip_prompt == false`).
- **레벨**: `PromptLevel::Info`.
- **주 문구**: ``"Discard changes to `{file_name}`?"``
- **상세 문구**: `None`.
- **버튼 배열**: **`["Restore", "Cancel"]`** (`0`번 `"Restore"`가 기본 버튼).

### `P-018` About 창 (`open_about_window`)
- **출처**: `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1529-L1741`
- **호출 액션**: `zed::About` (애플리케이션 메뉴 `Help > About Zed` 또는 명령 팔레트 `zed: about`). 이미 열려 있으면 새 창을 띄우지 않고 기존 창을 활성화한다 (`zed.rs:1698-1711`).
- **창 종류 및 치수 (`zed.rs:1713-1731`)**:
  - 모달 오버레이가 아니라 **별도의 독립 부동 창(`WindowKind::Floating`)**으로 열린다.
  - 창 크기: 너비 `px(440.)` = **`440px`**, 높이 `px(300.)` = **`300px`**, 화면 중앙 배치(`WindowBounds::centered`), 크기 조절 불가(`is_resizable: false`), 최소화 불가(`is_minimizable: false`).
  - 타이틀바 제목: `"About Zed"`.
- **레이아웃 및 색상 (`zed.rs:1604-1689`)**:
  - 루트 컨테이너: `v_flex().id("about-window").size_full().bg(editor_background).text_color(text).p_4().gap_4().text_center().justify_between()` (`p_4()` = `16px`, `gap_4()` = `16px`).
  - **상단 정보 영역** (`v_flex().w_full().gap_2().items_center()`, 간격 `8px`):
    1. 앱 아이콘 이미지: `img(app_icon).size_16().flex_none()` (`size_16()` = `4rem` = **`64px × 64px`**).
    2. 헤드라인 문구: `Headline::new("{release_channel_name} {CARGO_PKG_VERSION} {debug}")` (예: `"Zed 0.180.0 "`, 디버그 빌드면 끝에 `"(debug)"`).
    3. 커밋 SHA가 있을 때: `Label::new("Commit").color(Color::Muted).size(LabelSize::XSmall)`(`12px`) 아래에 `Label::new(commit).size(LabelSize::Small)`(`14px`, `Color::Default`).
    4. 버전: `Label::new("Version").color(Color::Muted).size(LabelSize::XSmall)`(`12px`) 아래에 `Label::new(full_version).size(LabelSize::Small)`(`14px`, `Color::Default`).
  - **하단 버튼 영역** (`h_flex().w_full().gap_1()`, 간격 `4px`, 좌우 `flex_1()` 균등 분할):
    1. **좌측 `"OK"` 버튼** (기본 포커스 `ok_entry.focus_handle`): `Button::new("ok", "OK").full_width().style(ButtonStyle::OutlinedGhost).toggle_state(ok_is_focused).selected_style(ButtonStyle::Tinted(TintColor::Accent))` — 클릭 또는 포커스 상태에서 `Enter`(`menu::Confirm`) 시 창 닫기(`window.remove_window()`).
    2. **우측 `"Copy"` 버튼** (`copy_entry.focus_handle`): `Button::new("copy", "Copy").full_width().style(ButtonStyle::Tinted(TintColor::Accent)).toggle_state(copy_is_focused).selected_style(ButtonStyle::Tinted(TintColor::Accent))` — 클릭 또는 포커스 상태에서 `Enter` 시 클립보드에 `"{message}\nCommit: {commit}\nVersion: {full_version}"`을 복사하고 즉시 창 닫기 (`zed.rs:1584-1596`).
  - **키보드 단축키**: `Navigable` 래퍼를 통해 `Tab`/`Shift+Tab`/화살표 키로 `"OK"`와 `"Copy"` 버튼 간 포커스 이동, `Enter`(`menu::Confirm`)로 포커스된 버튼 실행, `Escape`(`menu::Cancel`)로 즉시 창 닫기 (`zed.rs:1604-1610, 1687-1688`).

---

## 4. 프로젝트 패널 인라인 이름 입력칸 명세 (`P-019` ~ `P-022`)

### `P-019` 인라인 입력칸(`filename_editor`) 생성 위치 · 치수 · 초기 선택 범위
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L815-L852`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2523-L2625`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6131-L6137`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6784-L6786`
- **위젯 정체 및 치수**:
  - `ProjectPanel` 구조체가 단일 행 편집기 `filename_editor: Entity<Editor>`(`Editor::single_line(window, cx)`) 인스턴스 하나를 보유하고, 편집 중인 트리 행(`details.is_editing && !details.is_processing`)의 파일명 레이블 자리에 인라인으로 끼워 넣어 렌더링한다 (`project_panel.rs:815`, `6116`, `6784-6786`).
  - 입력칸 래퍼: `h_flex().h_6().w_full().child(self.filename_editor.clone())` — 높이 `h_6()` = `1.5rem` = **`24px`**, 너비 `w_full()`(아이콘/셰브론 오른쪽부터 행 끝까지 전체 너비) (`project_panel.rs:6785`).
  - 행 자체의 높이는 일반 파일/폴더 항목과 동일하며(`entry_spacing: Comfortable` 기본값 기준 내부 `24px` + 상하 `1px` 테두리 = **`26px`**), 들여쓰기(`depth * indent_size`)와 좌측 폴더/파일 아이콘 슬롯도 그대로 유지된다 (`project_panel.rs:6628-6635`).
- **실시간 파일 아이콘 갱신 (`project_panel.rs:6132-6137`)**:
  - 새 파일 생성 또는 파일 이름 변경 중 `settings.file_icons`가 `true`이고 입력한 텍스트 길이가 `3글자`(`filename.len() > 2`) 이상이면, 입력 중인 파일명(`Path::new(&filename)`)으로 `FileIcons::get_icon`을 실시간 조회해 좌측 파일 아이콘을 즉시 바꾼다.
- **초기 텍스트 및 선택 영역 (`project_panel.rs:2535-2537`, `2601-2620`)**:
  - **새 파일(`project_panel::NewFile`) / 새 폴더(`project_panel::NewDirectory`)**: 대상 디렉터리를 펼친 뒤 가상 엔트리(`NEW_ENTRY_ID`)를 삽입하고 `filename_editor.clear(window, cx)`로 빈 문자열 상태에서 포커스한다.
  - **이름 변경(`project_panel::Rename`, `F2`)**: 기존 파일/폴더 이름을 입력칸에 채우고,
    - **폴더(`entry.is_dir()`)**: 이름 전체(`0..file_name.len()`)를 선택(블록 지정)한다.
    - **파일**: 확장자를 제외한 줄기(`file_stem`, `0..file_stem.len()`)만 선택하여 확장자를 유지한 채 바로 새 이름을 타이핑할 수 있게 한다 (`project_panel.rs:2602-2612`). (단, Windows에서는 워크트리 루트 폴더 자체의 이름 변경은 차단되어 있음, `project_panel.rs:2576-2579`).

### `P-020` 인라인 입력칸 테두리 두께 및 상태별 색상 토큰
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L659-L677`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6165-L6199`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6273-L6278`
- **테두리 두께**:
  - 프로젝트 패널의 각 행 `div`는 `.rounded_none().bg(bg_color).border_1().border_r_2().border_color(border_color)`를 가진다 (`project_panel.rs:6272-6276`).
  - 즉, **상·하·좌 테두리는 `1px`, 우측 테두리는 `2px`**(오른쪽 스크롤바/경계에서 잘리지 않도록 `2px` 적용), 모서리 반경은 **`0px`**(`rounded_none`)이다.
- **상태별 행 테두리 색상 (`border_color`, `project_panel.rs:6171-6198`)**:
  - 인라인 입력칸이 포커스를 갖고 있을 때(`show_focused_border == true`):
    1. **`ValidationState::None` (정상)**: `item_colors.focused` = **`panel_focused_border`** (미지정 테마에서는 `border_focused`로 폴백).
    2. **`ValidationState::Warning(msg)` (경고)**: `Color::Warning.color(cx)` = **`status().warning`** (`text.warning`).
    3. **`ValidationState::Error(msg)` (오류)**: `Color::Error.color(cx)` = **`status().error`** (`text.error`).

### `P-021` 유효성 검사(`populate_validation_error`) 조건 · 문구 · 플로팅 오류 배너 레이아웃
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2069-L2141`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6842-L6861`
- **검사 시점**: `filename_editor`에서 `EditorEvent::BufferEdited`가 발생할 때마다 `populate_validation_error(cx)`를 호출한다 (`project_panel.rs:821-823`).
- **유효성 검사 조건 및 정확한 영문 문구 (`project_panel.rs:2074-2140`)**:

| 상태 | 발동 조건 | 정확한 표시 문구 | 색상 토큰 (테두리 및 글자) |
|---|---|---|---|
| `ValidationState::None` | 초기 빈 상태이거나 유효한 새 이름일 때 | 배너 미표시 | 행 테두리 `panel_focused_border` |
| `ValidationState::Warning` | 앞뒤에 공백이 있을 때(`filename.trim() != filename`) 또는 `RelPath::from_unix_str` 변환 실패 시 | `"File or directory name contains leading or trailing whitespace."` | `Color::Warning` (`status().warning`) |
| `ValidationState::Error` | 같은 위치에 동일한 파일/폴더명이 이미 존재할 때(`already_exists == true`, 단 이름 변경 시 자기 자신과 같은 이름은 제외) | `"File or directory '{filename}' already exists at location. Please choose a different name."` | `Color::Error` (`status().error`) |
| (참고: 데드코드 분기) | `if !filename.is_empty() { if filename.is_empty() { ... } }` (`L2075-2078`) | `"File or directory name cannot be empty."` (외부 `if !filename.is_empty()` 조건 때문에 실제로는 도달하지 않으며, 빈 이름은 배너 없이 `confirm_edit`에서 무시·취소됨) | `Color::Error` |

- **플로팅 오류/경고 배너 레이아웃 (`project_panel.rs:6842-6861`)**:
  - `validation_color_and_message`가 `Some((color, message))`일 때 편집 중인 행 바로 아래에 `deferred(...)`로 오버레이 배너를 띄운다:
    - 배치: `div().occlude().absolute().top_full().left(px(-1.)).right(px(-0.5))` — 편집 중인 행의 **하단(`top_full()`)에 딱 붙여서** 좌측 `-1px`, 우측 `-0.5px`로 행 테두리와 외곽선을 정렬한다.
    - 패딩 및 테두리: 상하 패딩 `.py_1()` = **`4px`**, 좌우 패딩 `.px_2()` = **`8px`**, 테두리 `.border_1()` = **`1px` 실선**, 테두리 색상 `color` (`Color::Error` 또는 `Color::Warning`).
    - 배경 및 텍스트: 배경색 `cx.theme().colors().background`, 내부 텍스트 `Label::new(message).color(Color::from(color)).size(LabelSize::Small)` (`14px`).

### `P-022` 인라인 입력칸 `Enter` · `Escape` · 포커스 해제(`Blurred`) 동작
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L828-L848`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1960-L1964`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2143-L2255`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2383-L2420`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7237-L7250`
- **키 컨텍스트 (`dispatch_context`, `project_panel.rs:7237-7250`)**:
  - `filename_editor`에 포커스가 있을 때는 `ProjectPanel > menu > editing`, 없을 때는 `ProjectPanel > menu > not_editing` 컨텍스트가 부여된다.
- **`Enter` (`menu::Confirm` → `ProjectPanel::confirm`, `project_panel.rs:1960-1964`, `2143-2255`)**:
  - `confirm_edit(true, window, cx)`를 호출한다 (`refocus = true`).
  - Windows 경로 스타일(`path_style.is_windows()`)에서는 파일명 끝의 마침표(`.`)들을 자동으로 제거한다(`while let Some(trimmed) = filename.strip_suffix('.')`, `L2154-2161`).
  - 새 항목 생성 중 입력값이 `/` 또는 `\`로 끝나면 새 파일 모드에서 시작했더라도 자동으로 폴더 생성(`edit_state.is_dir = true`)으로 전환한다 (`L2166-2180`).
  - 하위 경로(`foo/bar/baz.rs`)를 입력하면 중간 부모 디렉터리들을 자동으로 함께 생성한다.
  - **거부 조건 (`confirm_edit`가 `None` 반환)**:
    1. 공백 제거 후 빈 문자열(`filename.trim().is_empty()`, `L2162-2164`) → 아무 작업도 하지 않고 입력칸을 열린 채로 유지한다(`Enter`를 눌러도 닫히지 않음).
    2. 이미 존재하는 파일/폴더명과 충돌할 때(`L2195-2197`, `L2216-2223`) →- 단, 이름 변경 시 기존 자기 자신의 이름과 똑같으면(`existing.id == entry.id`) `project_panel`로 포커스를 돌려주며 편집을 종료한다 (`L2219-2221`). 다른 항목과 충돌하면 `None`을 반환해 입력칸을 열린 채로 유지한다.
- **`Escape` (`menu::Cancel` → `ProjectPanel::cancel`, `project_panel.rs:2411-2420`)**:
  - `discard_edit_state(window, cx)`를 호출해 `state.edit_state`를 제거하고(새 항목 생성 시 임시로 펼쳤던 폴더와 가상 행 `NEW_ENTRY_ID`도 제거), 이전 선택 항목으로 복원한 뒤 `window.focus(&self.focus_handle, cx)`로 프로젝트 패널에 포커스를 돌려준다.
- **포커스 해제 (`EditorEvent::Blurred`, `project_panel.rs:828-848`)**:
  - 창이 활성 상태일 때 다른 곳을 클릭해 입력칸이 포커스를 잃으면 `confirm_edit(false, window, cx)`를 호출한다:
    - 입력한 이름이 유효하면(`Some(task)`) **자동으로 확정(생성/이름 변경 실행)** 한다!
    - 입력한 이름이 비어 있거나 충돌 오류가 있으면(`None`) `discard_edit_state(window, cx)`를 호출해 **자동으로 편집을 취소**한다.

---

## 5. 영역별 텍스트 상태색 및 제2 기준 화면(`zed-reference-latte.png`) 대조 확정 명세 (`P-023` ~ `P-028`)

> `review-gaps-agy.md`의 `A-3`에서 제기된 `zed-reference.png`(다크, 최대화)와 추가 기준 화면 `zed-reference-latte.png`(Catppuccin Latte 라이트, 비최대화)의 픽셀 실측값을 Zed 커밋 `14b7b0ef3764d62a85070c8b5ccea105a2ad3235`의 실제 렌더링 소스로 확정한다.

### `P-023` 영역별 텍스트·아이콘·배경 상태색 종합 확정표 (`One Dark` 및 `zed-reference-latte.png` 실측값 병기)

| 영역 | 상태 | 텍스트 색상 토큰 (`Color` / `ThemeColors`) | `One Dark` Hex | `Catppuccin Latte` (`zed-reference-latte.png` 실측) | 아이콘 색상 토큰 | 행/요소 배경 토큰 (`Latte` 실측) | 출처 (커밋 `14b7b0ef...`) |
|---|---|---|---|---|---|---|---|
| **프로젝트 패널** (`ProjectPanel`) | 비선택 항목 (`is_marked == false`, Git 무변화) | `Color::Muted` (`text_muted`) | `#878a98` | **`#5c5f77`** (`projects`, `herdr`, `01-LAYOUT.md`) | `Color::Muted` (`icon_muted`) | `panel_background` (`#ccd0da`) | `editor/src/items.rs:2256-2299`, `project_panel.rs:7202-7203` |
| **프로젝트 패널** (`ProjectPanel`) | 선택/활성 항목 (`is_marked == true`, Git 무변화) | `Color::Default` (`text`) | `#dce0e5` | **`#4c4f69`** | `Color::Muted` (`icon_muted`) | `element_selected` (포커스 시 `panel_focused_border` 테두리) | `editor/src/items.rs:2256-2262`, `project_panel.rs:659-677, 985-989` |
| **프로젝트 패널** (`ProjectPanel`) | 마우스 호버 (비선택 항목 위) | `Color::Muted` (`text_muted`, 텍스트 색 불변) | `#878a98` | **`#5c5f77`** | `Color::Muted` (`icon_muted`) | `element_hover` | `project_panel.rs:671, 6160-6163, 6278` |
| **프로젝트 패널** (`ProjectPanel`) | Git 무시됨 (`entry.is_ignored == true`) | `Color::Ignored` (`text_ignored`) | `#636772` | **`#9ca0b0`** | `Color::Muted` (`icon_muted`) | 상태별 동일 | `editor/src/items.rs:2294-2295` |
| **프로젝트 패널** (`ProjectPanel`) | Git 수정됨 (`git_status.modified > 0`) | `Color::Modified` (`modified`) | `#e5c07b` | **`#df8e1e`** (`workbench-kit` 폴더명) | `Color::Muted` (단, 파일 테마 아이콘은 고유색 `#fe640b` 등 유지) | 상태별 동일 | `editor/src/items.rs:2290-2291` |
| **사이드바** (`Sidebar` 프로젝트 헤더) | 활성 프로젝트 헤더 (`is_active == true`) | `Color::Default` (`text`) | `#dce0e5` | **`#4c4f69`** (상단 `projects` 헤더) | `Color::Muted` (`icon_muted`) | `panel_background` (`#c3c7d2`, 오버레이 블렌딩) | `sidebar/src/sidebar.rs:2377-2387` |
| **사이드바** (`Sidebar` 프로젝트 헤더) | 비활성 프로젝트 헤더 (`is_active == false`) | `Color::Muted` (`text_muted`) | `#878a98` | **`#5c5f77`** | `Color::Muted` (`icon_muted`) | `panel_background` (`#c3c7d2`) | `sidebar/src/sidebar.rs:2379, 2384` |
| **사이드바** (`Sidebar` 스레드 항목) | 스레드 제목 (`ThreadItem`, 선택·비선택·호버 공통) | `Color::Default` (`text`) | `#dce0e5` | **`#4c4f69`** (`BOOK-RH00M2K19S: ...`) | `Color::Muted` (`icon_muted`) | 기본 투명, 선택 `ghost_element_selected`, 호버 `ghost_element_hover` | `ui/src/components/ai/thread_item.rs:264-271, 375-384, 440-447` |
| **사이드바** (`Sidebar` 스레드 항목) | 스레드 보조 정보 (프로젝트명·경로·브랜치·시각) | `Color::Muted` (`text_muted`, `LabelSize::Small`) | `#878a98` | **`#5c5f77`** (`13h`) | `Color::Muted` | 위와 동일 | `ui/src/components/ai/thread_item.rs:510-585` |
| **사이드바** (`Sidebar` 검색창/빈 목록) | `"Search threads..."` placeholder / `"No threads yet"` | `Color::Placeholder` (`text_placeholder`) | `#878a98` | **`#acb0be`** (`Search threads...`) | `icon_placeholder` | `#dce0e8` (헤더 영역) | `sidebar/src/sidebar.rs:2550-2557` |
| **탭** (`Editor` 탭) | 활성 편집기 탭 (`selected == true`, 포커스 무관) | `Color::Default` (`text`) | `#dce0e5` | **`#4c4f69`** | `Color::Muted` (`icon_muted`) | `tab_active_background` (`#eff1f5`) | `ui/src/components/tab.rs:112-125`, `editor/src/items.rs:771-796` |
| **탭** (`Editor` 탭) | 비활성 편집기 탭 (`selected == false`, 포커스 무관) | `Color::Muted` (`text_muted`) | `#878a98` | **`#5c5f77`** (`README.md`, `01-LAYOUT.md`) | `Color::Muted` (`icon_muted`) | `tab_inactive_background` (`#dce0e8`) | `ui/src/components/tab.rs:112-118`, `editor/src/items.rs:793-796` |
| **탭** (`MarkdownPreviewView` 등 일반 `Item` 탭) | 포커스된 패인의 활성/비활성 탭 (`deemphasized == false`) | 활성 `Color::Default` / 비활성 `Color::Muted` | `#dce0e5` / `#878a98` | **`#4c4f69`** / **`#5c5f77`** | `Color::Muted` | 활성 `#eff1f5` / 비활성 `#dce0e8` | `workspace/src/item.rs:143-155`, `markdown_preview_view.rs:1707` |
| **탭** (`MarkdownPreviewView` 등 일반 `Item` 탭) | **비포커스 패인(`deemphasized == true`)의 활성/비활성 탭** | **활성 `Color::Muted` / 비활성 `Color::Hidden`** (`text_hidden`) | `#878a98` / `#636772` | **활성 `#5c5f77` (`Preview 01-LAYOUT.md`) / 비활성 `#9ca0b0` (`Preview README.md`)** | `Color::Muted` | 활성 `#eff1f5` / 비활성 `#dce0e8` | `workspace/src/item.rs:143-155`, `markdown_preview_view.rs:1707` |
| **탭** (`Editor` / `MarkdownPreviewView`) | 탭 내 중복 파일 구분용 보조 경로 (`description`) | `Color::Muted` (`text_muted`, `LabelSize::XSmall`) | `#878a98` | **`#5c5f77`** | - | 상태별 동일 | `editor/src/items.rs:837-846`, `markdown_preview_view.rs:1712-1721` |
| **상태바** (`StatusBar`) | 모든 상태바 텍스트 버튼 (`ButtonStyle::Subtle`, 예: 커서 위치 `'1:1'`) | `Color::Default` (`text`, `LabelSize::Small`) | `#dce0e5` | **`#4c4f69`** (`1:1`) | 기본 `Color::Default`, 토글 활성 패널 아이콘은 `Color::Accent` (`#8839ef`) | `status_bar_background` (`#dce0e8`), 호버 시 `ghost_element_hover` | `ui/src/components/button/button_like.rs:242-247`, `cursor_position.rs:226-227` |
| **상태바 / 공통** | 비활성화(`disabled == true`) 상태 버튼·레이블 | `Color::Disabled` (`text_disabled`) | `#636772` | **`#9ca0b0`** | `Color::Disabled` (`icon_disabled`) | `ghost_element_disabled` | `ui/src/components/button/button_like.rs:417-422` |

### `P-024` 프로젝트 패널(`ProjectPanel`) 텍스트 상태색 상세 해설 (`zed-reference.png` + `zed-reference-latte.png` 교차 검증)
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L982-L990`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6808-L6817`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7164-L7204`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L2256-L2299`
- **소스 코드 동작 원리**:
  1. 프로젝트 패널에서 항목의 레이블 색상은 `entry_git_aware_label_color(git_status, entry.is_ignored, is_marked)`로 결정된다 (`project_panel.rs:7202-7203`).
  2. Git 상태(`Conflict`, `Deleted`, `Modified`, `Created`)나 Git 무시(`Ignored`)가 없는 일반 파일·폴더는 `entry_label_color(is_marked)`를 호출하며, **`is_marked == true`(현재 선택/열린 항목 또는 다중 선택된 항목)이면 `Color::Default`(`text`), `is_marked == false`(비선택 항목)이면 `Color::Muted`(`text_muted`)**를 반환한다 (`editor/src/items.rs:2256-2262`).
  3. 폴더와 파일 모두 동일한 `filename_text_color` 함수를 거치며, 폴더는 `settings.bold_folder_labels`가 `true`일 때만 `FontWeight::SEMIBOLD`가 추가된다 (`project_panel.rs:6809-6816`).
  4. 마우스 호버 시에는 행 배경만 `element_hover`(스티키 헤더는 `panel_overlay_hover`)로 바뀌고 **텍스트 색상과 아이콘 색상은 바뀌지 않는다** (`project_panel.rs:6160-6163`, `6278`).
- **`zed-reference-latte.png` 실측 교차 검증**:
  - `zed-reference-latte.png`에서 사이드바 헤더 `"projects"`와 상태바 `"1:1"`은 `#4c4f69`(`Catppuccin Latte`의 `text` = `Color::Default`)로 찍히는 반면, 프로젝트 패널의 비선택 폴더·파일(`projects`, `herdr`, `01-LAYOUT.md`)은 모두 **`#5c5f77`(`Catppuccin Latte`의 `text_muted` = `Color::Muted`)**로 정확히 구분되어 찍힌다.
  - 또한 Git 변경사항이 있는 `workbench-kit` 폴더명은 **`#df8e1e`(`Catppuccin Latte`의 `modified` = `Color::Modified`)**로 렌더링되어 `entry_git_aware_label_color`의 우선순위(`Modified` > `Default`/`Muted`)가 그대로 증명된다.

### `P-025` 사이드바(`Sidebar`) 텍스트 상태색 상세 해설
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2377-L2387`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2550-L2557`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/ai/thread_item.rs#L360-L385`
- **프로젝트 그룹 헤더 (`render_project_header`, `sidebar.rs:2377-2387`)**:
  - 현재 창에서 활성화된 프로젝트 그룹(`is_active == true`)의 헤더 이름은 `Color::Default`(`text`, `zed-reference-latte.png`에서 `#4c4f69`), 비활성 프로젝트 그룹(`is_active == false`)의 헤더 이름은 `.when(!is_active, |this| this.color(Color::Muted))`로 `Color::Muted`(`text_muted`, `#5c5f77`)를 사용한다.
- **하위 항목 (`ThreadItem`, `thread_item.rs:360-385`, `440-447`)**:
  - `ThreadItem`의 주 제목(`title_label`)은 선택(`selected`) 여부와 관계없이 `Color::Default`(`text`, `#4c4f69`)로 렌더링되며, 보조 시각 표시(`13h`)는 `Color::Muted`(`text_muted`, `#5c5f77`), 상단 검색 입력칸 `"Search threads..."` placeholder는 `Color::Placeholder`(`text_placeholder`, `#acb0be`)로 렌더링된다.
  - 우리 앱에서 사이드바에 작업 폴더 목록을 표시할 때도 **활성 작업 폴더 = `Color::Default`(`text`) + `ghost_element_selected` 배경, 비활성 작업 폴더 헤더 = `Color::Muted`(`text_muted`), 호버 시 = `ghost_element_hover` 배경** 규칙을 그대로 따른다.

### `P-026` 편집기 탭(`Editor`) vs 마크다운 미리보기 탭(`MarkdownPreviewView`)의 비활성·비포커스 흐림 차이 (`zed-reference-latte.png` 완벽 규명)
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L112-L125`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2887-L2898`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/item.rs#L130-L155`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L771-L849`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1667-L1711`
- **`zed-reference-latte.png`에서 관찰되는 탭 흐림 현상**:
  - `zed-reference-latte.png`의 탭 바에는 5개의 탭이 열려 있다:
    1. `README.md` (비활성 `Editor` 탭) → 실측 글자색 **`#5c5f77`** (`Color::Muted`)
    2. `01-LAYOUT.md` (비활성 `Editor` 탭) → 실측 글자색 **`#5c5f77`** (`Color::Muted`)
    3. `Preview 01-LAYOUT.md` (활성 `MarkdownPreviewView` 탭, 배경 `#eff1f5`) → 실측 글자색 **`#5c5f77`** (`Color::Muted`)
    4. `Preview README.md` (비활성 `MarkdownPreviewView` 탭, 배경 `#dce0e8`) → 실측 글자색 **`#9ca0b0`** (`Color::Hidden`)
    5. `Preview ...` (비활성 `MarkdownPreviewView` 탭) → 실측 글자색 **`#9ca0b0`** (`Color::Hidden`)
- **원인 (Zed 소스 코드의 `Editor` vs `MarkdownPreviewView` 구현 차이)**:
  1. `Pane::render_tab`(`pane.rs:2887-2895`)은 패인이 포커스를 갖고 있지 않을 때(`!self.has_focus(window, cx)`) `TabContentParams { selected: is_active, deemphasized: true, .. }`를 전달한다.
  2. **`MarkdownPreviewView::tab_content`**(`markdown_preview_view.rs:1705-1708`)는 `Label::new(title).single_line().color(params.text_color())`를 호출한다. `TabContentParams::text_color(&self)`(`workspace/src/item.rs:143-155`)는 다음과 같이 정의되어 있다:
     - `deemphasized == false` (패인이 포커스를 가질 때): `selected == true` → **`Color::Default`(`text`, `#4c4f69`)**, `selected == false` → **`Color::Muted`(`text_muted`, `#5c5f77`)**.
     - `deemphasized == true` (패인이 포커스를 잃었을 때, 예: 사이드바·프로젝트 패널·외부 창 포커스 시): `selected == true` → **`Color::Muted`(`text_muted`, `#5c5f77`)**, `selected == false` → **`Color::Hidden`(`text_hidden`, `#9ca0b0`)**!
  3. 반면 **`Editor::tab_content`**(`editor/src/items.rs:771-796`)는 `params.text_color()`를 쓰지 않고 `entry_git_aware_label_color(..., params.selected)` → `entry_label_color(params.selected)`를 호출하여 `params.deemphasized`를 무시한다. 따라서 패인이 포커스를 잃어도 비활성 `Editor` 탭(`README.md`, `01-LAYOUT.md`)은 `Color::Hidden`으로 내려가지 않고 **`Color::Muted`(`#5c5f77`)**를 유지한다!
  4. 또한 `MarkdownPreviewView` 탭의 좌측 아이콘은 파일 확장자 아이콘이 아니라 `IconName::FileDoc`(펼쳐진 책 모양 문서 아이콘, `markdown_preview_view.rs:1663-1665`)을 고정 사용한다.

### `P-027` 상태바(`StatusBar`) 텍스트 상태색 및 커서 위치(`'1:1'`) 표시 조건
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L115-L290`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L105-L260`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L242-L247`
- **상태바 커서 위치(`'1:1'`) 표시 및 텍스트 색상**:
  - `CursorPosition`은 `Button::new("go-to-line-column", text).label_size(LabelSize::Small)`로 렌더링되며(`cursor_position.rs:226-227`), 기본 `ButtonStyle::Subtle`에 따라 기본·호버·누름 상태 모두 **`Color::Default`(`text`)** 색상을 사용한다 (`zed-reference.png`에서 `#adc0d0`, `zed-reference-latte.png`에서 **`#4c4f69`**로 두 기준 화면 모두 `text` 토큰과 100% 일치).
  - 참고로 `zed-reference-latte.png`에서 활성 탭이 마크다운 미리보기(`Preview 01-LAYOUT.md`)인데도 상태바 우측에 `"1:1"`이 표시되는 이유: `CursorPosition::update_position`(`cursor_position.rs:105-145`)은 활성 패인 아이템이 `Editor`가 아닐 때(`item.act_as::<Editor>(cx)`가 `None`일 때) `self.position`을 지우지 않고 **직전에 활성화되었던 `Editor`의 마지막 커서 좌표(`1:1`)를 그대로 유지**하기 때문이다(단, 클릭 시에는 현재 활성 아이템이 `Editor`일 때만 `GoToLine` 모달을 띄움, `cursor_position.rs:236-244`).
- **상태바 좌측 패널 토글 아이콘 강조색**:
  - `zed-reference-latte.png` 좌측 하단에서 프로젝트 패널 아이콘이 보라색(**`#8839ef`**, `Catppuccin Latte`의 `accent` = `Color::Accent`)으로 표시되는 것은 해당 Dock 패널이 현재 열려(selected) 있기 때문이다.

### `P-028` 창 최대화 vs 비최대화(복원) 상태의 외곽선·모서리·창 조절 버튼 차이 (`zed-reference.png` vs `zed-reference-latte.png`)
- **출처**:
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/platforms/platform_windows.rs#L1-L165`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L155-L183`
  - `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/system.rs#L1-L55`
- **우측 상단 창 제어 버튼(최대화 vs 복원 아이콘)**:
  - `platform_windows.rs`에서 최소화(`-`)와 닫기(`×`) 사이의 가운데 버튼은 `window.is_maximized()` 상태에 따라 분기한다:
    - **비최대화(복원) 상태 (`zed-reference-latte.png`)**: 단일 사각형 모양의 최대화 아이콘(`WindowsCaptionButtonIcon::Maximize` / Segoe Fluent Icons `\u{e922}`)을 표시한다.
    - **최대화 상태 (`zed-reference.png`)**: 두 사각형이 겹친 복원 아이콘(`WindowsCaptionButtonIcon::Restore` / Segoe Fluent Icons `\u{e923}`)을 표시한다.
- **창 외곽 테두리 및 모서리 라운딩**:
  - **최대화 상태 (`zed-reference.png`)**: 화면 네 변에 꽉 차므로 외곽 테두리와 둥근 모서리가 없다 (`0px`).
  - **비최대화 상태 (`zed-reference-latte.png`)**:
    - Windows 11 DWM(`DWMWCP_ROUND`) 및 클라이언트 데코레이션에 의해 창 사방에 물리 `2px`(150% 배율 기준 논리 약 `1px`) 외곽 프레임 테두리(`#303134`)와 **논리 `8px` 반경(`CLIENT_SIDE_DECORATION_ROUNDING = px(8.0)`, 물리 `12px`)의 둥근 모서리**가 적용된다.
    - 상태바(`status_bar.rs:155-183`) 역시 비최대화·비타일링 상태(`!(tiling.bottom || tiling.left/right)`)에서 좌하단·우하단 모서리(사이드바가 닫혀 있거나 반대편에 있을 때)에 `rounded_bl(CLIENT_SIDE_DECORATION_ROUNDING)` / `rounded_br(CLIENT_SIDE_DECORATION_ROUNDING)`(`8px`)을 적용해 창 모서리 밖으로 배경이 삐져나오지 않게 마감한다.
- **타이틀바·패널·편집기·상태바 라이트 테마(`Catppuccin Latte`) 배경 계층 실측**:
  - 타이틀바 배경 (`title_bar_background`): **`#dce0e8`** (Catppuccin Latte Crust)
  - 비활성 탭 배경 (`tab_inactive_background` / `tab_bar_background`): **`#dce0e8`**
  - 상태바 배경 (`status_bar_background`): **`#dce0e8`**
  - 프로젝트 패널 배경 (`panel_background`): **`#ccd0da`** (Catppuccin Latte Surface0)
  - 사이드바 배경 (`panel_background` + 윈도우 틴트 블렌딩): **`#c3c7d2`** (상단 검색 헤더는 `#dce0e8`)
  - 편집기 및 마크다운 미리보기 본문 / 활성 탭 배경 (`editor_background` / `tab_active_background`): **`#eff1f5`** (Catppuccin Latte Base)

---

## 6. 확신 낮은 부분

1. **대화상자 렌더링 방식 선택 (`P-013`)**:
   - Zed 원본은 Windows에서 `use_system_prompts` 기본값이 `true`여서 Win32 네이티브 `TaskDialogIndirect`를 띄우지만, `use_system_prompts: false`로 바꾸면 GPUI 기반의 `ZedPromptRenderer`(`320px` 너비, 수직 스택 버튼, 마크다운 인라인 코드 렌더링)를 사용한다 (`ui_prompt.rs:20-27`).
   - 특히 Zed의 확인 문구들은 파일명을 마크다운 백틱(`` `file.rs` ``)으로 감싸서 생성하는데(`MarkdownInlineCode`), 네이티브 `TaskDialogIndirect`에서는 백틱 기호가 문자 그대로 노출되는 반면 `ZedPromptRenderer`에서는 인라인 코드 스타일로 깔끔하게 렌더링된다. 우리 앱에서는 테마 일관성과 마크다운 문구 렌더링을 위해 `ZedPromptRenderer`(내장 모달) 방식을 기본으로 채택할지 여부를 최종 결정해야 한다.
2. **줄바꿈 높이(`line_height`)에 따른 Picker 1줄 항목의 `1~2px` 편차 (`P-004`)**:
   - `ListItem`(`ListItemSpacing::Sparse`)은 상하 패딩 `4px + 4px = 8px`이 고정이고 텍스트 높이는 폰트 메트릭(`16px` 폰트의 기본 줄 높이 약 `20px~22px`)에서 결정되므로 Windows DirectWrite의 `.ZedMono` / `Segoe UI` 메트릭에 따라 1줄 항목 높이가 `28px` 또는 `30px`로 계산될 수 있다.
