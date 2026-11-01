# 01-zed 누락·구현 가능성 검토 — scouter-opus 담당 (02-INTERACTION.md, 04-APP-SPEC.md)

- 검토일 2026-10-31. 문서는 읽기만 했다. 02 → 04 순서로 봤고, 04는 writer-tauri가 고치는 중인 판(11:41 저장본)이다.
- 기준: README 요건 1~7, `_common.md` 끝의 추가 요건(툴팁, ☰·∨·우클릭 메뉴, 탭 분할·이동, base_keymap=VSCode), 기준 화면 `zed-reference.png`.
- **조사 중**: keymap(VSCode 덧씌움)과 탭 분할·이동은 scouter-sol이 `zed-panes-keymap.md`(I-700~)로 조사하고 있다. 이 두 주제는 표에 "조사 중"으로만 표시했고, 우선순위 개수에 넣지 않았다.
- Zed 소스는 커밋 `14b7b0ef3764d62a85070c8b5ccea105a2ad3235`에서 직접 확인했다. 아래 `Z/` = `https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/`. 인용 문구는 화면에 보이는 짧은 UI 문자열만 적었다.

---

## A. 누락 항목

| 번호 | 영역 | 빠진 것 | 왜 필요한가 (요건·기준 화면 근거) | 확인할 곳 (Zed 소스·설정) | 우선순위 |
|---|---|---|---|---|---|
| A-01 | 편집기 기본 동작 설정 | 편집기의 기본값과 동작이 01~04 어디에도 없다. 저장할 때 끝 공백 제거(`remove_trailing_whitespace_on_save: true`), 마지막 줄바꿈 보장(`ensure_final_newline_on_save: true`), `format_on_save: "off"`, `tab_size: 4`, `hard_tabs: false`, 괄호·따옴표 자동 닫기(`use_autoclose: true`), `auto_indent: "syntax_aware"`, `auto_indent_on_paste: true`, 커서 `cursor_blink: true`·`cursor_shape: "bar"`, `current_line_highlight: "all"`, `selection_highlight: true`, `rounded_selection: true`, `show_whitespaces: "selection"`, `soft_wrap: "none"`, `scroll_beyond_last_line: "one_page"`, `vertical_scroll_margin: 3`, `middle_click_paste: true`, `drag_and_drop_selection`, `show_wrap_guides`. 키 이름은 grep으로 모두 0건 | 요건 4(편집·코드 강조 편집기)와 "Zed 기본 편집기 기능". 저장 시 파일 내용이 바뀌는 규칙(공백 제거·끝 줄바꿈)이 다르면 같은 파일을 저장한 결과가 Zed와 달라진다 | [Z/assets/settings/default.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json) L295, L307, L359-363, L438, L498-500, L515-526, L583, L743, L792-813, L1617, L1626, L1653, L1687-1701 | 높음 |
| A-02 | 편집기 마우스 조작 | 클릭 횟수별 선택(더블클릭 = 단어, 세 번 = 줄), 끌어서 선택, Alt+드래그 세로(열) 선택, Ctrl+클릭 다중 커서 추가(VSCode keymap에서 바뀔 수 있음), 선택 영역 끌어 옮기기, 가운데 클릭 붙여넣기, 거터(줄 번호) 클릭으로 줄 선택. 02의 편집기 절(I-048~I-059)은 키만 다룬다 | 요건 4. 마우스 선택 규칙이 다르면 사용감이 바로 달라진다 | `Z/crates/editor/src/element.rs`(mouse_down·drag 처리), `Z/crates/editor/src/editor.rs`(begin_selection, SelectMode), default.json L438(`drag_and_drop_selection`), L743 | 높음 |
| A-03 | 찾기·바꾸기 창 | ☰ Edit에 Find(Ctrl+F)와 Replace†(Ctrl+H)를 남겼는데(02 I-367, I-479), 찾기 창의 모양·동작 명세가 없다. 위치(편집기 위 툴바), 입력칸, 대소문자·단어·정규식 토글, 다음/이전(Enter/Shift+Enter), 일치 개수 표시, 강조 색, Esc 닫기, 커서 위치 단어로 미리 채우기(`seed_search_query_from_cursor: "always"`), `search` 기본값 | 02 9.2에서 채용했으므로 구현 대상이다. 명세 없이는 "Zed와 같은" 판정이 불가능하다 | `Z/crates/search/src/buffer_search.rs`, default.json L817-842 | 높음 |
| A-04 | 저장·닫기 확인창 문구와 경우 | 02 I-069·I-070은 버튼 이름만 있다. 확인창 **본문 문구**와 **다른 경우**가 빠졌다. ① 한 개: 본문 "{name} contains unsaved edits. Do you want to save it?" + [Save, Don't Save, Cancel] ② 여러 개: "Do you want to save changes to the following files?" + [Save all, Discard all, Cancel] ③ 디스크에서 지워진 파일: `DELETED_MESSAGE` + [Save, Close, Cancel] ④ 디스크와 충돌: `CONFLICT_MESSAGE` + [Overwrite, Discard Edits, Cancel] ⑤ 저장 실패: "Unable to save file: {err}" + [Close Without Saving, Cancel]. 모두 `PromptLevel::Warning` | 요건 4(편집·저장). 문구·버튼 순서가 사용감의 일부다 | `Z/crates/workspace/src/pane.rs` L2011-2013, L2055-2066, L2334-2375, L5075-5076 (`DELETED_MESSAGE`·`CONFLICT_MESSAGE` 상수 본문은 미확인) | 높음 |
| A-05 | 디스크 변경 감지 | 다른 프로그램이 파일을 바꾸거나 지웠을 때의 동작이 없다. 깨끗한 버퍼는 자동으로 다시 읽기, 수정된 버퍼는 충돌 표시(Warning 점, 02 I-071), 지워진 파일 탭 표시, 저장할 때의 A-04 ③④ 확인. 02 I-071은 점 색만 적었다 | 요건 3·4. 트리에서 이름 변경·삭제를 하므로 열린 탭과 디스크가 어긋나는 경우가 반드시 생긴다 | `Z/crates/language/src/buffer.rs`(file_updated·reload), pane.rs L2292-2375, `Z/crates/editor/src/items.rs`(탭 제목 색 Conflict·Deleted, L2287-2295) | 높음 |
| A-06 | 프로젝트 패널 확인창 문구 | 휴지통·영구 삭제 확인창의 Zed 문구가 있는데 문서에 없다. 휴지통: "Do you want to trash `name`?" (버튼 Trash). 영구 삭제: "Are you sure you want to permanently delete `name`?" + 보조 문구 "This cannot be undone." (버튼 Delete). 여러 개면 "… the following N files?"와 목록(넘치면 ".. N files not shown"). 수정된 버퍼가 있으면 "It has unsaved changes, which will be lost."를 덧붙인다. 실패 시 "Failed to trash/delete X of Y file(s)." | 요건 3(삭제). 04 3.5는 이 문구를 확인하지 못했다며 임의 문구를 쓴다 (B-06) | `Z/crates/project_panel/src/project_panel.rs` L2838-2880, L3039-3042 | 높음 |
| A-07 | 이름 입력 검증 | 인라인 생성·이름 변경 입력칸의 오류·경고 표시가 없다. "File or directory name cannot be empty.", "File or directory '{name}' already exists at location. Please choose a different name." 등 `ValidationState::Error/Warning`의 문구, 표시 위치·색(오류 색 테두리와 메시지 줄, 01-LAYOUT과 함께), 오류일 때 Enter의 동작. 이름에 `/`·`\`를 넣으면 중간 폴더까지 만드는지도 없다 | 요건 3(생성·이름 변경) | project_panel.rs L2078, L2131, L6180-6200(validation 색), `ValidationState` 정의 | 높음 |
| A-08 | 프로젝트 패널 드래그 세부 | 02 I-094는 이동·복사만 적었다. 빠진 것: 폴더 위에 머무르면 자동 펼침(지연 타이머), 목록 위·아래 끝에서 자동 스크롤, 드롭 대상 강조(`drop_target_background`, 01 L-47), 드래그 중 미리보기 모양, 자기 자신·하위 폴더로 놓기 금지, 같은 이름이 있을 때 "Failed to move `{from}` to `{to}`. A file or folder already exists there." 등의 처리, 외부(탐색기) 파일 드롭 | 요건 3. 트리 조작의 핵심 경로 | project_panel.rs L781(30ms), L6428(500ms 타이머), L7631(16ms), L5082, L8432, `hover_expand_task`·`hover_scroll_task` | 중간 |
| A-09 | 마크다운 미리보기 상호작용 | ① 작업 목록 체크박스(`- [ ]`)를 미리보기에서 누르면 원문을 고친다(`on_checkbox_toggle`) ② 이미지 표시(상대 경로 해석, `image_resolver`) ③ 코드 블록 복사 버튼(hover 시) ④ mermaid 다이어그램. 02 4절에 없다. ①은 사용자가 누를 수 있는 동작이다 | 요건 4(마크다운 미리보기). 기준 화면은 문서 미리보기 위주 | `Z/crates/markdown_preview/src/markdown_preview_view.rs` L1128-1135, L1168-1195, `Z/crates/markdown/src/markdown.rs`(Checkbox, mermaid) | 중간 |
| A-10 | 줄 이동 대화상자 | 상태 표시줄 커서 위치를 누르거나 Ctrl+G를 누르면 Go to Line 대화상자가 뜬다(L-256, ☰ Go 메뉴에도 유지). 모양(위쪽 가운데 모달), 입력 형식(`줄:열`), 미리 이동, Enter·Esc 동작이 없다 | L-256과 ☰ 메뉴에서 채용했으므로 구현 대상 | `Z/crates/go_to_line/src/go_to_line.rs` | 중간 |
| A-11 | 알림(토스트) | 04가 여러 곳에서 "알림 한 줄"을 쓴다(A-16.4, A-24.4, A-24.6). 그러나 Zed 알림의 위치·모양·닫기·자동 사라짐 명세가 없다 | 오류를 보이는 모든 경로(A-06 실패, A-07 등)가 같은 알림을 쓴다 | `Z/crates/workspace/src/notifications.rs`, `toast_layer.rs` | 중간 |
| A-12 | 빈 상태 화면 | 폴더가 없을 때 Zed 프로젝트 패널은 "Choose one of the options below to use the Project Panel" + [Open Project] "or" [Clone Repository]를 보인다. 탭이 하나도 없는 Pane의 화면도 명세가 없다. 04 A-16.1은 임의 문구를 쓴다 (B-07) | 요건 1·2(폴더가 없는 시작 상태) | `Z/crates/ui/src/components/project_empty_state.rs`, project_panel.rs L8150-8175, pane.rs(빈 Pane 렌더) | 중간 |
| A-13 | 새 버퍼(제목 없음) | I-005가 Ctrl+N(Workspace = 새 버퍼)을 채용했지만 제목 없는 버퍼의 탭 제목, 첫 저장 때 다른 이름으로 저장 창, 닫을 때 확인이 없다 | I-005 채용, ☰ File > New File | `Z/crates/workspace/src/workspace.rs`(NewFile), `Z/crates/editor/src/items.rs`(untitled 제목) | 중간 |
| A-14 | 사이드바 키보드 | 04 A-17.3~A-17.5는 모두 "판단"이다. Zed `ThreadsSidebar` 컨텍스트의 실제 키(Left/Right = 부모·자식 선택, Enter·Space = Confirm, Ctrl+F = 필터 포커스 등)와 키보드 선택 표시 규칙을 먼저 적고, 우리 앱에서 무엇을 바꿨는지 구분해야 한다 | 요건 2. "Zed와 같은 사용감" | [Z/assets/keymaps/default-windows.json L755-777](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L755-L777), `Z/crates/sidebar/src/sidebar.rs`(selection, expand/collapse_selected_entry) | 중간 |
| A-15 | 사이드바·Dock 크기 조절 조작 | 사이드바 폭을 끄는 핸들(위치·폭·커서), 범위 200~800 고정, 더블클릭 초기화 여부. Dock 쪽은 L-30에 있으나 02에는 조작 규칙(끄는 중 최소·최대, `resize_all_panels_in_dock`)이 없다 | 04 A-11.1 "끌어서 조절"의 근거가 필요 | `Z/crates/workspace/src/multi_workspace.rs` L2290-2310, sidebar.rs L7873, L7935, dock.rs L1281-1336 | 중간 |
| A-16 | 스크롤바 동작 | 편집기·트리·미리보기 스크롤바가 언제 보이고 사라지는지(`scrollbar.show: "auto"`, 스크롤 시 표시·지연 후 숨김), 썸 끌기, 트랙 클릭이 없다. 기준 화면 트리 아래에 가로 스크롤바가 보인다 | 기준 화면, 요건 3(가로 스크롤) | `Z/crates/ui/src/components/scrollbar.rs`, default.json L655-668 | 중간 |
| A-17 | 창 최소 크기·창 조작 | 창 최소 크기 **360×240**(`window_min_size`)이 없다. 타이틀 바 더블클릭 최대화, 끌어서 이동, Windows 스냅은 OS 동작(`WindowControlArea::Drag`)이라고 한 줄이라도 있어야 한다 | 04 A-22.8(창 위치 복원)과 짝 | [Z/crates/zed/src/zed.rs L416-419](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L416-L419), `Z/crates/platform_title_bar/src/platform_title_bar.rs` L219-221 | 낮음 |
| A-18 | 탭 바 조작 세부 | 탭 줄 위 마우스 휠의 가로 스크롤, 탭 hover·활성·비활성 배경 상태, 탭 바 빈 곳 더블클릭(새 파일 여부) | 기준 화면 탭이 넘친다("Preview (" 잘림) | pane.rs L3690-3740(탭 목록 스크롤 영역), `Z/crates/ui/src/components/tab.rs` | 낮음 (일부 **조사 중**: 탭 드래그·분할) |
| A-19 | 한글 IME | 편집기·인라인 이름 입력·찾기 입력칸의 한글 조합 표시(조합 중 밑줄·후보창 위치). 03·04에 일부 언급만 있다 | 사용자 환경(한국어 문서 편집, 기준 화면 본문이 한글) | `Z/crates/editor/src/element.rs`(InputHandler, marked text) | 낮음 |
| A-20 | keymap (VSCode 덧씌움) | 02 머리표 "근거"가 "사용자 재정의·Vim·**다른 base keymap 제외**"라고 적고 있어 추가 요건(base_keymap=VSCode)과 어긋난다. 1절 키 표·툴팁·메뉴의 키 전체가 바뀔 수 있다 | 추가 요건 | `zed-panes-keymap.md` (I-700~) | **조사 중** |
| A-21 | 탭 분할·이동 | 드롭 영역 판정, 분할선, 최소 크기, 빈 Pane 처리, 키로 탭 옮기기 (02 I-074, 04 A-26.11) | 추가 요건 | `zed-panes-keymap.md` (I-700~) | **조사 중** |

---

## B. 구현 가능성 부족 항목

| 번호 | 파일·ID | 문제 | 구현에 필요한 정보 | 확인할 곳 | 우선순위 |
|---|---|---|---|---|---|
| B-01 | 02 I-007, I-028 ↔ 04 A-05.6, A-18 | **문서끼리 충돌.** 02는 파일 선택기(Ctrl+P)·명령 팔레트를 "선택"으로 두고 ☰ Go 메뉴(I-416, I-418)에 유지한다. 04는 둘 다 "넣지 않는다"(A-05.6)고 하고 ☰ 목록에서도 뺀다 | 하나로 결정한다. 유지하면 피커 UI 명세(A-03·A-10과 같은 모달 계열)가 필요하다 | 02 9.2 Go 메뉴, 04 2절 | 높음 |
| B-02 | 04 A-18 (3.9절) | ☰ 메뉴를 "메뉴 막대 대신 하위 메뉴가 있는 팝업"(L-85)이라고 쓴다. 02 I-302는 이를 바로잡아 "☰를 누르면 첫 메뉴가 열리고 그동안 **가로 메뉴 바**가 나타난다"고 한다. 또 04는 "잠정 목록, 작성 중"인데 02 9.2에는 최종 구조가 있다 | 04 3.9를 02 9.1~9.2(I-300~I-488)의 최종 구조와 표시 모델로 바꾼다. 01-LAYOUT L-85도 같이 고친다 | 02 I-302, [Z/crates/title_bar/src/application_menu.rs L176-204, L307-367](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L176-L204) | 높음 |
| B-03 | 04 A-13.4, A-14.2, A-14 표 | 메뉴 문구를 Zed와 다르게 바꿨다: "Remove from Sidebar"(Zed 사이드바 = **Remove**, I-635 / 트리 루트 = **Remove from Project**, I-555), "Delete (휴지통으로)"·"Delete Permanently"(Zed = **Trash**·**Delete**, I-552·I-553). 02 11절은 Zed 문구를 쓴다. "메뉴도 Zed와 동일" 요건과 어긋난다 | 문구를 Zed 그대로 쓰거나, 바꾸면 "우리 앱 문구(판단)"로 표시하고 02 11절·툴팁과 맞춘다. 사이드바 Delete 키(A-17.5)도 Zed에는 없는 판단임을 표시 | 02 11.2·11.6, [Z/crates/sidebar/src/sidebar.rs L3005-3010](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L3005-L3010), project_panel.rs L1283-1299 | 높음 |
| B-04 | 04 A-13.6 ↔ 02 I-667 | 04는 "순서는 등록 순서, 바꾸기 없음"인데 02 11.6은 우클릭 **Move Up / Move Down**(폴더 2개 이상, 끝이면 비활성)을 유지한다 | 04에 Move Up/Down 동작과 state.json `folders` 순서 갱신 규칙을 넣는다 | 02 I-634, sidebar.rs L2963-2995 | 높음 |
| B-05 | 04 3.5 확인창 문구, A-40.2 | "Zed 문구는 확인하지 못했다"며 임의 문구(`Move "{name}" to the Recycle Bin?` 등)를 쓴다. Zed 문구는 확인 가능하다 (A-06) | A-06의 문구·버튼·여러 개 목록 규칙·수정 버퍼 경고를 그대로 쓴다 | project_panel.rs L2838-2880 | 높음 |
| B-06 | 02 I-069, I-070 | 버튼 이름만 있고 본문 문구, "같은 버퍼가 다른 곳에 남아 있으면 건너뛴다"는 예외의 정확한 조건, 지워진 파일·충돌·저장 실패의 경우가 없다 (A-04) | 다섯 경우의 본문·버튼·순서·기본 버튼(Enter)·Esc 결과 | pane.rs L1910-1935, L2011-2066, L2292-2420, L5075 | 높음 |
| B-07 | 04 A-16.1 | 빈 상태 문구 `No folder is registered.`가 임의다. Zed는 ProjectEmptyState 문구가 있다 (A-12) | Zed 문구를 쓰고, 우리 앱에서 없는 [Clone Repository]를 뺀 판단을 표시한다 | `Z/crates/ui/src/components/project_empty_state.rs` | 중간 |
| B-08 | 04 A-12.8 | 사이드바 폴더 항목에 "전체 경로" 툴팁(500ms)을 단다. Zed 헤더 행에는 툴팁이 없다 (02 L-231). "툴팁도 Zed와 같게" 요건과 어긋난다 | 빼거나 "Zed와 다름(판단)"으로 표시하고 A-34 점검표에 예외로 적는다 | 02 L-231, sidebar.rs L2331-2440 | 중간 |
| B-09 | 04 A-34.6 | 비활성 버튼(← → 등)에도 툴팁이 뜨는지 "미확인". `ButtonLike`는 disabled와 상관없이 툴팁을 붙인다(클릭 처리만 disabled로 막음). 그래서 뜬다고 확인할 수 있다 (hover 이벤트가 disabled 요소에서도 오는지는 gpui 쪽 추가 확인) | "비활성이어도 툴팁 표시"로 확정 | [Z/crates/ui/src/components/button/button_like.rs L883-895](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L883-L895) | 중간 |
| B-10 | 02 I-094 | 드래그 앤 드롭의 상태(드래그 중 강조, 금지 대상, 자동 펼침·스크롤 시간)가 빠져 구현 값을 정할 수 없다 (A-08) | 타이머 값(500ms·16ms·30ms의 용도 확인), 강조 토큰, 금지 규칙, 오류 문구 | project_panel.rs L5203-5338, L6428, L7631 | 중간 |
| B-11 | 02 I-091, I-092 | 인라인 입력의 모양(입력칸 테두리·배경, 검증 오류 색과 메시지 위치)과 검증 문구가 없다 (A-07). 경로 구분자가 들어간 이름의 처리도 없다 | 입력칸 스타일 토큰, 메시지 문구 2종 이상, 오류일 때 확정 차단 여부 | project_panel.rs L2060-2140, L6170-6200 | 중간 |
| B-12 | 02 L-226, L-251 (8.2) | 프로젝트 이름 버튼 "Recent Projects"(Ctrl-R), 상태 표시줄 "Open Threads Sidebar" 문구가 우리 앱 동작과 맞는지 "미결"이다. 04도 이 버튼이 무엇을 여는지(최근 목록 없음, A-10.5) 정하지 않아 툴팁이 정해지지 않는다 | 04에서 버튼 동작(등록 폴더 전환 팝업 등)과 문구를 결정 | 02 8.2, 04 A-10.5, A-15.5 | 중간 |
| B-13 | 02 I-570 (11.10) | 우클릭 "Close"의 단축키가 보이는지 미확인. gpui는 바인딩을 찾을 때 액션을 **인자까지** 비교한다(`partial_eq`)고 보이므로, 인자가 다른(`close_pinned`) Ctrl+W는 표시되지 않을 가능성이 높다 | gpui keymap의 액션 비교 방식을 확인해 "키 없음"으로 확정 | `Z/crates/gpui/src/keymap.rs` L137-159(bindings_for_action), `Z/crates/gpui/src/action.rs`(partial_eq) | 낮음 |
| B-14 | 02 8.2·9·10·11의 모든 단축키 | 표시 키를 "마지막 정의 우선"으로 고른 추정값이고 Windows 기본 키맵 기준이다 | VSCode 덧씌움 결과로 다시 계산 | `zed-panes-keymap.md` | **조사 중** |
| B-15 | 02 I-074, 04 A-26.9~A-26.12 | 탭 드래그·분할의 판정 값이 없다 | 드롭 영역 비율, 오버레이, 분할선 | `zed-panes-keymap.md` | **조사 중** |
| B-16 | 02 10·11절 머리말 | I-600~I-655가 ∨ 메뉴와 우클릭 메뉴 두 조사에서 겹쳐 "절 번호를 함께 적는다"는 규칙으로 버틴다. 04에서 I-6xx를 참조할 때(A-19 등) 어느 쪽인지 헷갈린다 | 한쪽 대역을 옮긴다(예: 우클릭 편집기·미리보기·사이드바를 I-5xx 안쪽 번호로) | 02 10·11절 | 중간 |
| B-17 | 04 A-24.4, A-16.4, A-24.6 | "알림 한 줄"의 모양·위치·사라짐이 없다 (A-11) | Zed 토스트·알림 명세 | workspace/src/notifications.rs, toast_layer.rs | 중간 |
| B-18 | 04 A-25.8 ↔ 02 I-118 | Zed 기본은 미저장 버퍼를 복원하고 종료 때 묻지 않는다(`session.restore_unsaved_buffers: true`). 04는 반대로 정했다(판단 표시 있음). 일치도 판정 A-31.12에서 Zed와 다른 점을 예외로 적지 않았다 | A-31·04-COMPARISON에 "의도한 차이"로 명시 | default.json L2727-2734 | 낮음 |
| B-19 | 04 A-29.5 ↔ 02 I-647/I-668·11.9 | 04는 프로젝트 패널 위치를 왼쪽 고정으로 정했다. 02는 버튼 우클릭 "Dock Left / Dock Right"를 결정 대기로 둔다 | 04 결정에 맞춰 02 11.7·11.9 정리 (메뉴에 Dock Left만 남길지, 메뉴 자체를 뺄지) | 02 11.7 | 낮음 |
| B-20 | 04 A-06, A-31.13, A-31.15, A-40.4·A-40.9 | "작성 중" 표시가 남아 있다(I-300~I-349, I-500~, L-200~L-272). 02에는 이미 최종 구조가 들어왔다 | 참조 범위를 실제 ID(I-300~I-488, I-500~I-668, L-200~L-272)로 고치고 "작성 중"을 지운다 | 02 8~11절 | 낮음 |
| B-21 | 04 A-17.6, A-13.1 ↔ 02 I-004, I-122 | Ctrl+K Ctrl+O를 "폴더 추가"로 쓴다. Zed에서는 `workspace::Open`(파일·폴더 열기 진입)이고, 실제 대화상자가 무엇을 허용하는지 미확인(I-122)이다. VSCode 덧씌움에서 바뀔 수도 있다 | 덧씌움 결과 확인 후 "Zed 키를 다른 동작에 재사용(판단)"으로 표시 | `zed-panes-keymap.md`, keymap L26-27 | 낮음 (**조사 중** 연동) |
| B-22 | 02 I-083, I-123 | 미리보기 스크롤 동기화가 미확인이다. 02는 "약속하지 않는다"로 정리했다 | 현재 결정으로 구현 가능. 04-COMPARISON에서 판정 제외로 둔다 | markdown_preview_view.rs L561-589 | 낮음 |
| B-23 | 02 I-120, I-124 | 트리 펼침 영속화가 미확인이다. 04 A-25.7이 "복원 안 함"으로 정해 구현에는 지장이 없다 | — | — | 낮음 |

---

## C. 종합 판정

02-INTERACTION.md는 탭·임시 탭·핀·트리 키·포커스·상태 복원·툴팁·세 메뉴(☰·∨·우클릭)를 소스 줄 단위로 정리해, 그 범위는 대부분 **새 코드로 다시 만들 수 있는 수준**이다. 다만 아래 두 가지 때문에 담당 범위 전체를 "구현 가능 수준"으로 판정하지는 않는다.

1. **편집기 자체의 동작이 거의 비어 있다.** 기본 설정값(A-01), 마우스 선택(A-02), 찾기 창(A-03), 저장·충돌 확인창(A-04), 디스크 변경 감지(A-05)가 없다. 요건 4 "편집"의 핵심이라 이것 없이는 "Zed와 같은 편집기"를 판정할 수 없다.
2. **04-APP-SPEC이 02와 충돌하는 곳이 남아 있다.** ☰ 메뉴 표시 모델(B-02), 명령 팔레트·파일 선택기 채용 여부(B-01), 메뉴 문구(B-03), 순서 바꾸기(B-04), 임의 확인창 문구(B-05)가 그렇다. 04는 writer-tauri가 고치는 중이라 일부는 이미 반영됐을 수 있다.

**높음 우선순위:** A 7개(A-01~A-07) + B 6개(B-01~B-06) = **13개**. "조사 중"(A-20, A-21, B-14, B-15)은 세지 않았다. keymap(VSCode)과 탭 분할·이동이 끝나면 02의 키 표·툴팁·메뉴 단축키 열과 04 A-17·A-18·A-26·A-33을 다시 대조해야 한다.

---

## 확신 낮은 부분

- **A-04 확인창 본문:** `DELETED_MESSAGE`·`CONFLICT_MESSAGE` 상수의 본문은 열어 보지 않았다. 버튼 배열과 `PromptLevel`만 확인했다.
- **A-08 타이머 용도:** project_panel.rs의 30ms·500ms·16ms 타이머가 각각 무엇(자동 펼침·자동 스크롤·기타)인지 정확히 대응시키지 않았다. 줄 번호만 근거로 적었다.
- **B-09 hover 이벤트:** disabled 버튼에서 hover 이벤트가 오는지는 gpui 쪽을 확인하지 않았다. ButtonLike가 툴팁을 disabled와 무관하게 붙인다는 것만 확인했다.
- **B-13 액션 비교:** "인자까지 비교한다"는 gpui `partial_eq`에 근거한 추정이다.
- **A-02 Ctrl+클릭 키:** 다중 커서 수식키가 VSCode 덧씌움에서 바뀌는지는 **조사 중**인 keymap 결과에 달려 있다.
- **04 시점:** 04-APP-SPEC은 수정 중인 11:41 저장본을 읽었다. 그 뒤 반영된 수정은 보지 못했다.
- **줄 번호:** 일부 링크 줄 번호는 grep 결과로 잡은 근사치(±수 줄)다.
