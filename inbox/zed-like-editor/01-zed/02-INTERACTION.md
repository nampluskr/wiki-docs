---
tags: [zed, interaction, keymap, gpui, tauri, spec, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 01-zed · 02 · 사용감과 동작

> 이 파일을 읽는 때: 키·탭·트리·미리보기·포커스·상태 복원의 동작을 Zed와 같게 만들기 전에.

| 항목 | 내용 |
| --- | --- |
| 근거 | Zed 커밋 [14b7b0e](https://github.com/zed-industries/zed/commit/14b7b0ef3764d62a85070c8b5ccea105a2ad3235) (2026-10-30). 키는 **적용 keymap** 기준이다: Windows 기본 키맵 → VSCode 덧씌움(`base_keymap = VSCode`) → specific-overrides 순서로 불러온 결과 (1.1절 I-700~I-705). 사용자 재정의·Vim은 제외 |
| 검증 | 소스 대조 / 실행 검증 안 함 |
| 라이선스 | 키맵 JSON 원문은 옮기지 않았다. 어떤 키가 무엇을 하는지(사실)만 우리 말로 적었다 |
| ID | `I-NNN`은 조사 자료의 항목 ID를 그대로 쓴다. 치수는 [01-LAYOUT.md](01-LAYOUT.md)의 `L-NN` |
| 채용 표기 | **채용**: 구현한다 / **제외**: 우리 앱 범위 밖. 사용자 결정(2026-10-31, `_decisions.md`)을 기준으로 확정했다. 결정 목록에 없는 기능은 제외로 둔다 |

이전 문서는 [01-LAYOUT.md](01-LAYOUT.md), 다음 문서는 [03-THEME-ICONS.md](03-THEME-ICONS.md), 입구는 [../README.md](../README.md)다.

## 1. 키맵 표 (범위 안)

### 1.1 읽는 법

| ID | 규칙 | 출처 |
| --- | --- | --- |
| I-001 | 키 표의 원본은 Windows 기본 키맵에 VSCode 덧씌움을 얹은 **적용 keymap**이다 (I-700). 공백으로 나뉜 조합은 **순서대로 누르는 chord**다. `Ctrl+K V`는 Ctrl+K를 누른 뒤 V를 따로 누른다 (두 번째 V에 Ctrl을 유지하지 않는다). 기본 키맵에서 바뀐 행에는 **기본→VSCode**를 적었다 | [키 바인딩 문서](https://zed.dev/docs/key-bindings) · [keymap:600-607](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L600-L607) · [vscode.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json) |
| I-002 | 컨텍스트가 다르면 같은 키의 의미가 다르다. 트리는 `ProjectPanel`과 `menu`, `editing`/`not_editing`을 함께 보고, 인라인 이름 입력에는 `Editor` 컨텍스트도 관여한다. `Ctrl+N`은 Workspace에서 새 버퍼, 트리에서 새 파일 생성이다 | [project_panel.rs:7237-7249](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7237-L7249) · [keymap:1008-1049](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1008-L1049) · [keymap:654-660](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L654-L660) |
| I-700 | 불러오는 순서 | Windows 기본 키맵 → VSCode 덧씌움 → (Vim·Helix를 쓰면 그것) → specific-overrides → 사용자 키맵. 두 파일을 합치기만 하면 실행 키맵이 아니다. 이 문서는 specific-overrides까지 반영했다 | [zed.rs:2329-2380](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L2329-L2380) · [settings.rs:161-188](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/settings.rs#L161-L188) |
| I-701 | VSCode 파일 | VSCode 덧씌움은 Windows에서도 `linux/vscode.json`을 쓴다. 기본 설정의 `base_keymap`은 Zed이고, 우리 앱이 따르는 것은 사용자 기준 설정 VSCode다 (둘을 구별한다) | [base_keymap_setting.rs:98-123](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/base_keymap_setting.rs#L98-L123) · [default.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json) |
| I-702 | 키 충돌 | 같은 키가 여러 곳에 있으면 **일치한 포커스 컨텍스트가 더 깊은 쪽**이 이긴다 (편집기 > Pane > 작업 영역). 깊이가 같으면 **나중에 등록된 바인딩**이 이긴다. 컨텍스트가 없는 바인딩은 가장 깊은 것으로 본다. 조건식이 길다고 우선하지 않는다 | [keymap.rs:150-218](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/keymap.rs#L150-L218) |
| I-703 | 해제 (null) | `null`은 그 컨텍스트에서 낮은 우선순위 후보를 해제한다. 출처 등급은 사용자 > Vim > 기본 덧씌움(VSCode) > Zed 기본이다. 덧씌움의 null은 사용자 설정을 지우지 않는다. 이 등급을 일반 충돌의 깊이·순서 규칙과 섞지 않는다 | [keymap.rs:220-250](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/keymap.rs#L220-L250) · [keymap.rs:631-691](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/keymap.rs#L631-L691) · [keymap_file.rs:1414-1445](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L1414-L1445) |
| I-704 | 메뉴·툴팁 표시 키 | 메뉴·툴팁에 보이는 키는 **액션 인수까지 같은 바인딩** 중에서 고르고, 현재 컨텍스트에서 다른 액션에 가려진 키는 뺀다. 남은 후보 중 마지막 바인딩을 쓴다. 사용자 키맵을 다시 불러오면 메뉴도 다시 만든다 (8.4절 L-217 보충) | [keymap.rs:91-114](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/keymap.rs#L91-L114) · [key_dispatch.rs:396-445](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/key_dispatch.rs#L396-L445) · [zed.rs:2329-2337](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L2329-L2337) |
| I-705 | specific-overrides | 선택기 입력칸(`Picker > Editor`)에서 Ctrl+Shift+A = 동작 메뉴 열기(`picker::ToggleActionsMenu`), Ctrl+Shift+S = 다중 선택 토글. 파일 선택기에서 Ctrl+J/K/H/L = 아래·위·왼쪽·오른쪽 분할로 열기, Tab = 다음 다중 선택, Ctrl+Shift+P = 이전 선택. VSCode의 일반 편집기 바인딩보다 **뒤에** 불러오므로 유지해야 한다 | [specific-overrides.json:8-34](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/specific-overrides.json#L8-L34) |

### 1.2 파일·탭·분할·패널

| ID | 키 | 컨텍스트 | 동작 | 채용 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-003 | Ctrl+O | 전역 | 파일 열기 (`workspace::OpenFiles`) | 채용 | [keymap:26-27](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L26-L27) |
| I-004 | Ctrl+K Ctrl+O | 전역 | 파일/폴더 열기 진입 (`workspace::Open`) | 채용 | [keymap:26-27](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L26-L27) |
| I-005 | Ctrl+N / Ctrl+S / Ctrl+Shift+S | Workspace | 새 버퍼 / 저장 / 다른 이름으로 저장 | 채용 | [keymap:654-660](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L654-L660) |
| I-006 | Ctrl+K S | Workspace | 전체 저장 | 채용 | [keymap:699-705](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L699-L705) |
| I-007 | Ctrl+P 또는 Ctrl+E | Workspace | 프로젝트 파일 선택기 (`file_finder::Toggle`). 앱 메뉴 Go > Go to File...(I-418)에서 유지한다. 선택기 안에서는 Ctrl+J/K/H/L이 분할로 열기다 (I-705) | 채용 | [keymap:688-697](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688-L697) |
| I-008 | Ctrl+W 또는 Ctrl+F4 | Pane, Workspace | 활성 탭 닫기. `close_pinned=false`라 핀 탭은 보호 | 채용 | [keymap:503-514](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L503-L514) |
| I-009 | Ctrl+Shift+T | Workspace | 닫은 탭 다시 열기 | 채용 | [keymap:681-686](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L681-L686) |
| I-010 | Ctrl+PageUp / Ctrl+PageDown | Pane | 탭 순서로 이전·다음 이동 | 채용 | [keymap:490-506](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L490-L506) |
| I-011 | Ctrl+Tab / Ctrl+Shift+Tab | Workspace | 탭 전환기 (역방향은 마지막 항목부터) | 채용 | [keymap:688-697](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688-L697) |
| I-012 | Ctrl+Shift+PageUp / Ctrl+Shift+PageDown | Pane | 탭 순서 바꾸기 (왼쪽·오른쪽) | 채용 | [keymap:503-506](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L503-L506) |
| I-013 | Alt+1…9 / Alt+0 | Pane | n번째 탭 / 마지막 탭. Workspace 컨텍스트의 같은 Alt+숫자는 Pane 이동이므로 구별한다 | 채용 | [keymap:490-502](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L490-L502) |
| I-014 | Ctrl+K Shift+Enter | Pane | 핀 고정/해제. 임시 탭 유지와 다른 기능이다 | 채용 | [keymap:532-535](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L532-L535) |
| I-015 | Ctrl+K W | Pane | 모든 탭 닫기 (`close_pinned=false`) | 채용 | [keymap:509-515](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L509-L515) |
| I-016 | Ctrl+K E / T / U | Pane | 왼쪽 탭 닫기 / 오른쪽 탭 닫기 / 깨끗한 탭 닫기. 핀 탭 제외 | 채용 | [keymap:509-515](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L509-L515) |
| I-017 | Alt+Left / Alt+Right | Pane | 탐색 이력 뒤로·앞으로 (탭 번호가 아니라 이력). 탭 바 ← → 버튼이 같은 동작 | 채용 | [keymap:516-521](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L516-L521) |
| I-018 | Ctrl+\ | Editor | 오른쪽으로 분할 | 채용 | [keymap:538-599](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L538-L599) |
| I-019 | Ctrl+K 방향키 | Pane | 위·아래·왼쪽·오른쪽 분할 | 채용 | [keymap:812-821](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L812-L821) |
| I-020 | Ctrl+K Ctrl+방향키 | Workspace | 그 방향 Pane으로 포커스 이동 | 채용 | [keymap:709-712](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L709-L712) |
| I-021 | Ctrl+K Shift+방향키 | Workspace | Pane 배치 교환 | 제외 | [keymap:713-716](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L713-L716) |
| I-022 | Ctrl+B | Workspace | 왼쪽 Dock 열기/닫기 | 채용 | [keymap:672-678](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L672-L678) |
| I-023 | Ctrl+Shift+E | Workspace | 프로젝트 패널 포커스 토글 (4절 포커스 참조) | 채용 | [keymap:697-700](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L697-L700) |
| I-024 | Ctrl+Alt+J / Ctrl+Alt+; | Workspace | 사이드바 표시 토글 / 사이드바 포커스 | 채용 | [keymap:672-678](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L672-L678) |
| I-025 | Shift+Escape | 전역 | 현재 영역 확대/복구 (탭 바 최대화 버튼과 같음, L-73) | 채용 | [keymap:24-27](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L24-L27) |
| I-026 | F6 / Shift+F6 / Ctrl+F6 | Workspace | 주요 영역 순환 (다음·이전·다음) | 채용 | [keymap:634-644](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L634-L644) |
| I-027 | Ctrl+Alt+W | Workspace | 활성 Dock 닫기 | 제외 | [keymap:728-732](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L728-L732) |
| I-028 | Ctrl+Shift+P 또는 F1 | Workspace | 명령 팔레트. 앱 메뉴 Go > Command Palette...(I-416)에서 유지하며 채택한 기능의 명령만 노출 | 채용 | [keymap:692-697](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L692-L697) |
| I-029 | Ctrl+Shift+A | RecentProjects(및 그 안 Picker·Editor) | 프로젝트에 폴더 추가. **전역 키가 아니다.** **기본→VSCode 보정:** 최근 프로젝트 선택기의 입력칸에서는 나중에 불러오는 `picker::ToggleActionsMenu`가 이긴다 (I-789, I-705). 일반 편집기에서는 같은 키가 블록 주석 토글이다 (I-745). 그래서 우리 앱의 폴더 추가 키는 Zed에 대응이 없고 [04-APP-SPEC.md](04-APP-SPEC.md)에서 정한다 | 변경 | [keymap:1303-1311](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1303-L1311) · [specific-overrides.json:8-14](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/specific-overrides.json#L8-L14) |
| — | (참조 I-367) Ctrl+F | Editor, 전체 모드 | 파일 안 찾기 `buffer_search::Deploy` (사용자 결정으로 포함, 찾기 창은 13절 자리) | 채용 | [keymap:137](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L137) |
| — | (참조 I-479) Ctrl+H | Editor, 전체 모드 | 바꾸기 `buffer_search::DeployReplace` | 채용 | [keymap:138](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L138) |
| — | (참조 I-343) Ctrl+Shift+N | Workspace | 새 창 `workspace::NewWindow` | 채용 | [keymap:660](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L660) |
| — | (참조 I-347) Ctrl+R | Workspace | 최근 폴더 `projects::OpenRecent` | 채용 | [keymap:648](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L648) |
| — | (참조 I-394) Ctrl+Shift+Y | Workspace | 모든 Dock 토글 `workspace::ToggleAllDocks` | 채용 | [keymap:677](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L677) |
| — | (참조 I-327) Ctrl+Alt+, (Ctrl+,도 같은 동작에 연결) | 전역 | settings.json 탭으로 열기 `zed::OpenSettingsFile`. Zed의 Ctrl+,는 설정 화면 키이며 우리는 화면이 없어 이 동작에 연결한다 (판단) | 채용 | [keymap:33](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L33) · [keymap:1536](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1536) |
| — | (참조 I-332) Ctrl+K Ctrl+S | Workspace | 키맵 편집 화면 `zed::OpenKeymap` (15절) | 채용 | [keymap:687](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L687) |
| — | (참조 I-372) Shift+Alt+Right / Shift+Alt+Left | Editor | 구문 단위 선택 넓히기 / 좁히기 (Tree-sitter, 사용자 결정으로 포함) | 채용 | [keymap:551-552](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L551-L552) |
| — | (참조 X-44) Ctrl+Shift+X | Workspace | 확장 관리 화면 `zed::Extensions` (16절). VSCode 덧씌움이 바꾸지 않는다 | 채용 | [keymap:717](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L717) |

### 1.3 프로젝트 트리

| ID | 키 | 컨텍스트 | 동작 | 채용 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-030 | Up / Down, Home / End | ProjectPanel + menu | 이전·다음 / 처음·끝 항목 선택. Tab·Shift+Tab도 전역 menu의 다음·이전 선택에 연결 | 채용 | [keymap:5-20](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L5-L20) |
| I-031 | Left / Right | ProjectPanel | 선택 항목 접기 / 펼치기 | 채용 | [keymap:1008-1016](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1008-L1016) |
| I-032 | Ctrl+Left / Ctrl+Right | ProjectPanel | 전체 접기 / 전체 펼치기 | 채용 | [keymap:1011-1014](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1011-L1014) |
| I-033 | Space | ProjectPanel, 이름 편집 중 아님 | 파일 열기: 임시 탭으로 열고 트리에 포커스를 남긴다 | 채용 | [keymap:1044-1049](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1044-L1049) |
| I-034 | Enter 또는 F2 | ProjectPanel | **이름 변경** (Windows 기본). Enter를 "파일 열기"로 구현하면 Zed와 다르다 | 채용 | [keymap:1024-1029](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1024-L1029) |
| I-035 | Enter / Escape | 이름 입력, ProjectPanel + menu | 확정 / 취소. 확정은 `menu::Confirm`이 처리하며 단순 Rename 경로와 분리된다 | 채용 | [keymap:17-20](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L17-L20) |
| I-036 | Ctrl+N / Alt+N | ProjectPanel | 새 파일 / 새 폴더 | 채용 | [keymap:1013-1017](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1013-L1017) |
| I-037 | Ctrl+C 또는 Ctrl+Insert / Ctrl+X / Ctrl+V 또는 Shift+Insert | ProjectPanel | 복사 / 잘라내기 / 붙여넣기. 텍스트가 아니라 **파일·폴더** 작업 | 채용 | [keymap:1017-1021](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1017-L1021) |
| I-038 | Ctrl+Z / Ctrl+Shift+Z | ProjectPanel | 패널 파일 작업 되돌리기 / 다시 실행 (포함 목록에 없음) | 제외 | [keymap:1022-1025](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1022-L1025) |
| I-039 | Delete 또는 Backspace | ProjectPanel | **휴지통**으로 보냄 (`skip_prompt=false`: 확인 있음) | 채용 | [keymap:1026-1032](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1026-L1032) |
| I-040 | Shift+Delete / Ctrl+Delete / Ctrl+Backspace | ProjectPanel | **영구 삭제** (확인 있음) | 채용 | [keymap:1028-1032](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1028-L1032) |
| I-041 | Shift+Alt+C / Ctrl+K Ctrl+Shift+C | ProjectPanel | 경로 복사 / 상대 경로 복사 | 채용 | [keymap:1022-1023](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1022-L1023) |
| I-042 | Menu 키 또는 Shift+F10 | ProjectPanel | 우클릭 메뉴 열기 | 채용 | [keymap:1033-1041](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1033-L1041) |
| I-043 | Shift+Up / Shift+Down | ProjectPanel | 선택 범위 확장 | 채용 | [keymap:1037-1041](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1037-L1041) |

### 1.4 마크다운 미리보기와 편집기 기본

| ID | 키 | 컨텍스트 | 동작 | 채용 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-044 | Ctrl+Shift+V / Ctrl+K V | Editor, 확장자 md | 미리보기 열기(같은 Pane) / 옆 Pane에 열기 | 채용 | [keymap:600-607](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L600-L607) |
| I-045 | Ctrl+Shift+V | MarkdownPreview | 미리보기를 닫고 원문으로 돌아가기 | 채용 | [keymap:1443-1458](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1443-L1458) |
| I-046 | Up/Down, PageUp/PageDown, Ctrl+Home/End | MarkdownPreview | 한 줄·한 쪽·맨 위/아래 스크롤 | 채용 | [keymap:1443-1455](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1443-L1455) |
| I-047 | Alt+Up / Alt+Down | MarkdownPreview | 블록 하나씩 스크롤 | 채용 | [keymap:1449-1454](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1449-L1454) |
| I-048 | 방향키 | Editor | 커서 이동 | 채용 | [keymap:79-105](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L79-L105) |
| I-049 | Ctrl+Left / Ctrl+Right | Editor | 단어 단위 이동 (이전 단어 시작 / 다음 단어 끝) | 채용 | [keymap:91-105](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L91-L105) |
| I-050 | Home / End | Editor | 줄 처음·끝. 줄바꿈 경계에서 멈추고 Home은 들여쓰기도 고려 | 채용 | [keymap:79-95](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L79-L95) |
| I-051 | Ctrl+Home / Ctrl+End | Editor | 문서 처음·끝 | 채용 | [keymap:91-105](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L91-L105) |
| I-052 | Shift+방향키 / Ctrl+Shift+Left·Right | Editor | 선택 확장 (글자·단어) | 채용 | [keymap:97-105](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L97-L105) |
| I-053 | Shift+Home·End / Ctrl+Shift+Home·End | Editor | 줄 처음·끝까지 / 문서 처음·끝까지 선택 | 채용 | [keymap:97-110](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L97-L110) |
| I-054 | PageUp·PageDown / Shift+PageUp·PageDown | Editor | 쪽 이동 / 쪽 선택 | 채용 | [keymap:79-90](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L79-L90) |
| I-055 | Ctrl+A / Ctrl+L | Editor | 전체 선택 / 줄 선택 | 채용 | [keymap:103-106](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L103-L106) |
| I-056 | Ctrl+C / Ctrl+X / Ctrl+V, Ctrl+Z, Ctrl+Y 또는 Ctrl+Shift+Z | Editor | 복사·잘라내기·붙여넣기, 실행 취소, 다시 실행 | 채용 | [keymap:68-78](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L68-L78) |
| I-057 | Backspace / Delete, Ctrl+Backspace / Ctrl+Delete | Editor | 글자 삭제, 단어 삭제 | 채용 | [keymap:60-78](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L60-L78) |
| I-058 | Enter / Ctrl+Enter / Ctrl+Shift+Enter | Editor, 전체 모드 | 줄바꿈 / 아래에 새 줄 / 위에 새 줄. VSCode 덧씌움이 Ctrl+Enter를 `!AcpThread > Editor && mode == full` 조건으로 다시 등록하지만 결과는 같다 (조건 보강, I-818, I-752) | 채용 | [keymap:126-135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L126-L135) · [vscode.json:35](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L35) |
| I-059 | Tab / Shift+Tab | Editor | 들여쓰기 / 내어쓰기. 편집기에서 Tab을 포커스 이동으로 바꾸지 않는다 | 채용 | [keymap:60-65](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L60-L65) |
| I-743 | Alt+Z | Editor | 소프트 랩(줄 바꿈 표시) 토글 `editor::ToggleSoftWrap`. **기본 없음→VSCode 추가** | 채용 | [vscode.json:13](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L13) |
| I-820 | Ctrl+K Z | Editor, 전체 모드 | **기본: 소프트 랩 토글→VSCode: 가운데 정렬 레이아웃 토글 `workspace::ToggleCenteredLayout`.** 가운데 정렬 레이아웃은 포함 목록에 없어 이 키는 아무 동작도 하지 않는다. 소프트 랩은 Ctrl+K Ctrl+Z와 Alt+Z(I-743)로 남는다 | 제외 | [vscode.json:13-26](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L13-L26) · [keymap:126-136](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L126-L136) |
| I-742 | Shift+Alt+I | Editor | 선택을 줄마다 커서로 나누기 `editor::SplitSelectionIntoLines`. **기본 없음→VSCode 추가** | 채용 | [vscode.json:12](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L12) |
| I-745 | Ctrl+Shift+A | Editor | 블록 주석 토글 `editor::ToggleBlockComments`. **기본 없음→VSCode 추가.** 기존 Shift+Alt+A, Ctrl+K Ctrl+/도 남는다 (I-821). 최근 프로젝트 선택기 안에서는 다른 동작 (I-029) | 채용 | [vscode.json:15](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L15) · [keymap:562-565](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L562-L565) |
| I-746 | Ctrl+K Ctrl+C | Editor | 줄 주석 토글 `editor::ToggleComments` (`advance_downwards = false`). **기본→VSCode: `comment_empty_lines`가 true→false.** 소스 주석의 Add Line Comment라는 이름만 보고 추가 전용 명령으로 구현하지 않는다 (I-822) | 채용 | [vscode.json:16](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L16) · [keymap:562-563](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L562-L563) |
| I-747 | Ctrl+/ | Editor | 위와 같음 (줄 주석 토글, **기본→VSCode: `comment_empty_lines` true→false**) | 채용 | [vscode.json:17](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L17) · [keymap:562-563](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L562-L563) |
| I-752 | Ctrl+Enter | Editor, 전체 모드 (`!AcpThread >` 조건) | 아래에 새 줄. 결과는 기본과 같다 (I-058, I-818) | 채용 | [vscode.json:35](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L35) |

VSCode 덧씌움 바인딩 21개(I-740~I-760) 중 우리 범위는 위 7개다. 포맷(Shift+Alt+F, Ctrl+K Ctrl+F; I-740, I-741, I-823)은 기본 키맵에도 있지만 언어 서버·외부 포맷터가 필요해 범위 밖이다. **제외로 확정**했고 표에 넣지 않았다 (사용자 결정, `_decisions.md`). 나머지(코드 이동·시그니처, 인라인 AI, 호출 계층, 디버그, 에이전트, 터미널)는 범위 밖이라 쓰지 않았다.

## 2. 미리보기 탭과 핀 탭

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| I-060 | 서로 다른 세 개념 | **임시(preview) 탭**: 다음 탐색 때 교체되는 탭. **일반 탭**: 계속 남는 탭. **핀 탭**: 탭 바 앞쪽에 고정하고 일반 닫기에서 보호하는 탭. 마크다운의 "Preview X.md" 제목은 렌더링 종류를 뜻하며 이 세 상태와 별개다 (L-69) | [pane.rs:1018-1049](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1018-L1049) · [pane.rs:2736-2768](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2736-L2768) · [markdown_preview_view.rs:1664-1685](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1664-L1685) |
| I-061 | 기본값 | `preview_tabs.enabled` = true, `enable_preview_from_project_panel` = true. 트리에서 한 번 눌러 열면 임시 탭이다. `enable_preview_from_file_finder`는 false이므로 "파일 선택기 열기도 모두 임시"라고 쓰면 틀린다 (L-132) | [default.json:1533-1545](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1533-L1545) |
| I-062 | 개수·교체 위치 | Pane마다 임시 탭 id를 **하나** 가진다. 다른 임시 파일을 열면 기존 임시 탭을 닫고 **그 자리(인덱스)를 재사용**한다. 모든 Pane을 통틀어 하나는 아니다 | [pane.rs:1014-1049](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1014-L1049) · [pane.rs:1147-1157](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1147-L1157) |
| I-063 | 일반 탭으로 승격 | 편집기의 **편집 이벤트**가 임시 상태를 푼다 (항목이 보존을 요청하는 예외는 있다). 탭 더블클릭, 트리의 비임시 열기(더블클릭·가운데 클릭)도 일반 탭으로 바꾼다 | [pane.rs:1062-1068](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1062-L1068) · [pane.rs:1106-1115](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1106-L1115) · [pane.rs:2955-2974](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2955-L2974) · [project_panel.rs:6607-6624](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6607-L6624) |
| I-064 | 시각 구분 | 원문 편집기 탭의 임시 제목은 **기울임꼴**이다. 마크다운 미리보기 탭은 별도 렌더링이라 같은 기울임이라고 일반화하지 않는다 (L-68) | [items.rs:820-835](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L820-L835) · [markdown_preview_view.rs:1667-1723](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1667-L1723) |
| I-065 | 핀 동작 | 핀은 임시 상태를 풀고 `pinned_tab_count` 경계 안으로 탭을 옮긴다. Ctrl+K Shift+Enter로 토글. 더블클릭(임시→일반)과 핀을 섞지 않는다 | [pane.rs:2710-2719](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2710-L2719) · [pane.rs:2744-2768](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2744-L2768) · [keymap:532-535](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L532-L535) |
| I-066 | 핀 보호·표시 | Ctrl+W·Ctrl+F4는 핀 탭을 닫지 않는다. 가운데 클릭도 핀 탭은 닫지 않는다. `show_pinned_tabs_in_separate_row` 기본 false라 같은 줄에 둔다 | [keymap:503-508](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L503-L508) · [pane.rs:2976-2984](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2976-L2984) · [default.json:1475-1485](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1475-L1485) |

## 3. 탭 동작

| ID | 상황 | 동작 | 출처 |
| --- | --- | --- | --- |
| I-067 | 새 일반 탭 위치 | 위치를 지정하지 않으면 **활성 탭 바로 다음**에 넣되 핀 영역 뒤쪽 경계를 지킨다. 임시 탭 교체는 이전 임시 위치를 재사용하는 별도 경로다 | [pane.rs:1265-1275](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1265-L1275) · [pane.rs:1151-1157](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1151-L1157) |
| I-068 | 같은 파일 다시 열기 | 같은 Pane에서 이미 열린 탭을 찾아 활성화한다. 비임시 열기이면 기존 임시 상태도 푼다. 다른 Pane까지 중복을 막지는 않는다 (`reveal_if_open` 기본 false) | [pane.rs:1083-1115](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1083-L1115) · [default.json:198-199](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L198-L199) |
| I-069 | 닫기·저장 확인 | `autosave`는 **off가 기본**이다 (`default.json` 1471행에 `"autosave": "off"`). `on_focus_change`는 고를 수 있는 값일 뿐 기본값이 아니고, 기준 화면의 사용자 설정에도 없다. 그래서 **수정 표시(점)와 닫기 저장 확인이 기본 동작**이다. 확인창의 본문과 버튼, 디스크 충돌·삭제·저장 실패의 경우는 13.4절. 수정된 탭을 닫으면 **Save / Don't Save / Cancel** 확인. 같은 버퍼가 다른 곳에 남아 있으면 확인을 건너뛰는 예외가 있다 | [default.json:1464-1473](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1464-L1473) · [pane.rs:1910-1935](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1910-L1935) · [pane.rs:2400-2448](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2400-L2448) |
| I-070 | 수정된 탭 여러 개 닫기 | 둘 이상이면 **Save all / Discard all / Cancel** 일괄 선택 | [pane.rs:1996-2027](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1996-L2027) |
| I-071 | 수정 표시 | 수정됨은 Accent 색 점, 충돌은 Warning 색 점, 깨끗하면 없음. 제목 뒤 별표가 아니다 (L-65) | [pane.rs:5090-5099](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L5090-L5099) |
| I-072 | 닫은 뒤 활성 탭 | `activate_on_close` 기본 `history`: 최근 활성 이력의 탭을 고르고, 이력이 없으면 왼쪽 이웃. `neighbour`는 오른쪽 우선 후 왼쪽, `left_neighbour`는 왼쪽 우선 | [default.json:1512-1520](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1512-L1520) · [pane.rs:2154-2186](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2154-L2186) |
| I-073 | 닫기 버튼 | `close_position` = right, `show_close_button` = hover (always·hidden도 가능). 버튼으로 닫아도 저장 확인을 거친다 (L-66) | [default.json:1495-1511](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1495-L1511) · [pane.rs:3055-3070](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3055-L3070) |
| I-074 | 탭 드래그 | 같은 Pane 안 재정렬, 다른 Pane으로 이동, 분할 드롭 영역으로 새 Pane 만들기. Windows에서 Ctrl+드래그는 복제(분할 가능한 항목). 드롭 위치 강조를 그린다 | [pane.rs:2987-3021](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2987-L3021) · [pane.rs:3895-3945](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3895-L3945) |
| I-075 | 가운데 클릭 | 일반 탭은 닫기 의도(저장 확인 포함)로 닫는다. 핀 탭은 무시. 수정된 탭도 강제 폐기가 아니다 | [pane.rs:2976-2984](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2976-L2984) |
| I-076 | 탭 전환기 | Ctrl+Tab 계열은 **최근 사용 순(MRU)**이고 Ctrl+PageUp/PageDown은 **위치 순서**다. 둘을 구별한다. 전환기가 열린 동안 미리보기를 보일지는 `tab_switcher.show_file_preview` (기본 true) | [default.json:1487-1494](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1487-L1494) · [keymap:503-506](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L503-L506) · [keymap:688-694](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688-L694) |

### 3.1 탭 분할·이동

탭 분할·이동 조사(`zed-panes-keymap.md`)의 ID(I-706~I-739)를 그대로 쓴다. 분할선·드롭 오버레이의 치수와 색은 [01-LAYOUT.md](01-LAYOUT.md) 4.5절에 있다. 위 3절의 I-074(탭 드래그), I-017~I-021(분할·포커스 키)와 같은 기능의 상세판이다. 기본 분할·이동 키는 3.1.4의 최종 적용표가 우선한다.

**3.1.1 분할 버튼과 분할 방식**

| ID | 규칙 | 출처 |
| --- | --- | --- |
| I-706 | 탭 바 오른쪽 버튼은 Pane에 포커스가 있거나 그 Pane의 컨텍스트 메뉴가 포커스를 가진 때 그려진다. 순서는 `+`(New…), 분할(Split 아이콘), 확대(Zoom In/Out). 버튼 사이 gap은 Base04 = 4px. 분할 버튼은 클릭하면 바로 나누지 않고 **오른쪽 위 기준 팝업**을 연다 | [pane.rs:4345-4434](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4345-L4434) |
| I-707 | 분할 팝업의 순서는 **Split Right → Split Left → Split Up → Split Down**. 분할할 수 있는 활성 항목이면 기본 방식은 `ClonePane`. 분할할 수 없는 항목이지만 탭이 2개 이상이면 `MovePane`. 빈 Pane이거나 분할 불가 단일 항목이면 분할 버튼이 **비활성** | [pane.rs:4350-4413](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4350-L4413) |
| I-708 | `ClonePane`은 Pane의 모든 탭을 복사하지 않고 **활성 항목 하나**의 복제본만 새 Pane에 연다. 원래 Pane의 탭은 그대로 두고 새 Pane에는 탐색 이력을 갈라서 넘긴다. 일반 편집기는 분할할 수 있다. 빈 Pane이거나 분할 불가이면 이 명령은 새 Pane을 만들지 않는다 | [workspace.rs:6447-6478](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L6447-L6478) · [items.rs:876-889](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L876-L889) |
| I-709 | 분할 방식은 셋이다: `ClonePane`(기본), `EmptyPane`(빈 Pane), `MovePane`(활성 항목 이동). `MovePane`에서 원래 Pane의 탭이 1개 이하이면 반대편에 빈 Pane을 만들고 원래 항목에 포커스를 남겨, 항목이 요청한 방향으로 옮겨 간 것과 같은 배치가 된다. 열거형 주석의 설명보다 실제 분할 함수의 동작을 따른다 | [pane.rs:214-224](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L214-L224) · [pane.rs:2582-2606](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2582-L2606) · [workspace.rs:6429-6444](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L6429-L6444) |
| I-710 | 명시적 Split Right/Left/Up/Down은 설정의 영향을 받지 않는다. **SplitHorizontal만** `pane_split_direction_horizontal`(기본 down)을, **SplitVertical만** `pane_split_direction_vertical`(기본 right)을 따른다. 이름에 주의: horizontal은 위·아래, vertical은 좌·우 배치다 | [default.json:98-101](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L98-L101) · [pane_group.rs:1071-1082](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1071-L1082) · [pane.rs:4472-4500](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4472-L4500) |
| I-711 | ☰ View의 Split Up/Down/Left/Right(9.2절)도 기본 `ClonePane`이다. 탭 바 분할 팝업의 `MovePane` 분기(분할 불가 항목)와 구별한다 | [app_menus.rs:29-43](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L29-L43) · [pane.rs:214-246](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L214-L246) |

**3.1.2 드래그 앤 드롭 판정**

| ID | 규칙 | 출처 |
| --- | --- | --- |
| I-712 | 분할 드롭 판정은 **탭 바를 뺀 Pane 본문 영역**의 크기를 기준으로 한다. 임계값 d = min(폭, 높이) × `drop_target_size` (기본 **0.2**). 예: 800×600이면 d = 120px. x < d, x > 폭 − d, y < d, y > 높이 − d 중 하나이면 가장자리 후보이고, 등호 경계는 중앙이다 | [pane.rs:3839-3871](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3839-L3871) · [pane.rs:4631-4643](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4631-L4643) · [default.json:206-210](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L206-L210) |
| I-713 | 가장자리 후보에서는 위·오른쪽·아래·왼쪽 변까지의 거리 중 **가장 작은 쪽**을 방향으로 고른다. 모서리를 네 구역으로 나누는 우선순위는 없다. 거리가 같을 때의 선택은 실행 검증을 못 해서 확정하지 않는다. 분할이 허용되지 않는 Pane이면 이 갱신을 건너뛴다 | [pane.rs:3845-3892](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3845-L3892) |
| I-714 | 본문 오버레이는 `drop_target_background`로 칠한 절대 배치 직사각형이다. 중앙이면 본문 전체, 방향 분할이면 해당 쪽 **절반(50%)**을 덮는다. **판정 폭 20%와 보이는 폭 50%는 서로 다르다.** 본문 오버레이에는 테두리·모서리 지정이 없다 | [pane.rs:4684-4732](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4684-L4732) |
| I-715 | 탭 위에 놓으면 분할 방향을 지우고 **그 탭의 위치**에 넣거나 옮긴다. 같은 Pane이면 재정렬, 다른 Pane이면 이동. 표시는 배경 `drop_target_background`, 경계 `drop_target_border`, 그리고 드래그 시작 위치보다 앞이면 탭 **왼쪽 2px**, 뒤이면 **오른쪽 2px** 선. 포인터가 탭 절반을 넘었는지로 선 위치를 정하지 않는다 | [pane.rs:2987-3026](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2987-L3026) · [pane.rs:3895-3987](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3895-L3987) |
| I-716 | 탭 바의 남는 영역에 놓으면 **맨 끝에 넣고 분할하지 않는다.** 영역 최소 폭 1.5rem(rem 16이면 24px), 높이는 탭 높이(32px). rem 계산은 root rem 16 가정이고 실측은 아니다 | [pane.rs:3731-3769](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3731-L3769) |
| I-717 | 본문 **중앙**에 놓으면 기존 Pane의 활성 위치로 이동. **가장자리**에 놓으면 그 방향에 새 빈 Pane을 만들고 항목을 옮긴다. 활성 항목이 자기 드롭 처리(`handle_drop`)를 가지고 있으면 그것이 먼저다. 드래그한 임시 탭은 **일반 탭으로 바뀐다** | [pane.rs:3895-3925](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3895-L3925) · [pane.rs:4698-4709](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4698-L4709) |
| I-718 | Windows에서 놓을 때 **Ctrl을 누르고 있으면 복제** 경로를 쓴다. 복제할 수 있는 항목에만 적용하고 불가하면 이동으로 되돌린다. 복제 항목은 위치 지정 없이 추가되므로 일반 이동의 위치 삽입과 똑같다고 가정하지 않는다 | [pane.rs:3916-3960](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3916-L3960) |
| I-719 | 같은 Pane 안에서 일반 탭을 핀 구역으로 옮기면 핀 개수가 늘고, 핀을 일반 구역으로 옮기면 줄어든다. 핀 여부는 **새 위치**로 판단한다. 핀은 파일의 속성이 아니라 Pane 앞쪽 연속 구간의 경계다 | [pane.rs:3961-3975](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3961-L3975) |
| I-720 | 다른 Pane으로 옮긴 탭은 대상의 핀 구역에 놓였을 때 핀으로 처리한다. 출발지에서 핀이던 탭을 새 Pane의 0번 위치에 놓을 때도 핀이 유지된다. 일반 구간으로 옮기면 무조건 핀을 유지하지는 않는다. 핀 바 전용 드롭 영역은 핀 경계로 옮긴 뒤 명시적으로 핀 처리한다 | [pane.rs:3976-4040](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3976-L4040) |
| I-721 | 프로젝트 패널에서 드래그하면 **선택 항목 하나**(`active_selection`)만 쓴다. 본문 가장자리에 놓으면 새 Pane을 나눠 열고, 중앙이면 기존 Pane에 연다. 탭 위는 그 위치, 빈 탭 바는 끝. 다중 선택 항목을 모두 연다고 쓰면 틀린다 | [pane.rs:4048-4121](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4048-L4121) · [pane.rs:3756-3764](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3756-L3764) |
| I-722 | 트리 드롭으로 열기: 포커스를 주고, **임시 탭을 허용하지 않고**(일반 탭), 활성화한다. 대상 위치가 핀 구역이면 열린 탭을 핀으로 만든다. 이미 열린 같은 파일이 있으면 그 탭을 찾아 쓴다 | [pane.rs:1071-1098](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1071-L1098) · [pane.rs:4109-4134](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4109-L4134) |

**3.1.3 PaneGroup 구조·크기·활성 표시**

| ID | 규칙 | 출처 |
| --- | --- | --- |
| I-723 | 루트는 Pane 하나이거나 재귀적인 Axis다. Left/Right는 가로(Horizontal) 나열, Up/Down은 세로(Vertical) 나열. 같은 축이면 형제 위치에 끼우고, 다른 축이면 그 잎을 새 Axis로 감싼다. 왼쪽·위는 기존 항목 앞, 오른쪽·아래는 뒤에 둔다 | [pane_group.rs:500-515](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L500-L515) · [pane_group.rs:685-719](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L685-L719) |
| I-724 | 새 형제를 끼우면 **그 Axis의 flex를 모두 1로 초기화**한다. 기존 비율을 유지한 채 대상 하나만 반으로 가르는 방식이 아니다. 자식 길이 ≈ 그룹 축 길이 × flex[i] ÷ 자식 수 (px로 반올림). 저장된 flex의 길이나 합이 올바르지 않으면 모두 1로 복구한다 | [pane_group.rs:657-719](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L657-L719) · [pane_group.rs:1403-1426](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1403-L1426) |
| I-725 | 분할선 **1px**, 색 `pane_group_border`. 마우스 반응 영역은 **4px**(경계 앞뒤 2px). 가로 나열은 `ResizeColumn` 커서, 세로 나열은 `ResizeRow` 커서 | [pane_group.rs:22-24](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L22-L24) · [pane_group.rs:1161](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1161) · [pane_group.rs:1322-1340](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1322-L1340) · [pane_group.rs:1530-1549](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1530-L1549) |
| I-726 | 드래그 최소 **폭 80px**(가로 나열), **높이 100px**(세로 나열). 인접 Pane이 최소치에 닿으면 더 먼 형제까지 변화량을 넘긴다. 창 자체가 이미 최소보다 작으면 이 최소가 전체 레이아웃 보장은 아니며 크기 조절이 일찍 끝날 수 있다 | [pane_group.rs:22-24](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L22-L24) · [pane_group.rs:1224-1319](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1224-L1319) |
| I-727 | 분할선 **더블클릭**(클릭 횟수 2 이상)은 그 Axis의 형제 flex를 모두 1로 만들어 **균등화**하고 저장한다. 중첩된 모든 그룹을 재귀적으로 초기화하는 명령과 다르다. 마우스를 놓으면 드래그 상태가 풀린다 | [pane_group.rs:1551-1604](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1551-L1604) · [pane_group.rs:779-784](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L779-L784) |
| I-728 | `close_pane_if_empty` 기본 true: 마지막 항목이 사라지면 그 Pane이 그룹에서 빠지고, 형제가 하나만 남으면 부모를 접으며 그 그룹의 비율을 초기화한다. **유일한 루트 Pane은 남아** 빈 편집 영역이 된다. 마지막 탭을 닫는 것을 창 닫기로 구현하지 않는다 | [pane.rs:618](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L618) · [pane.rs:2203-2213](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2203-L2213) · [pane_group.rs:170-187](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L170-L187) · [pane_group.rs:737-770](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L737-L770) · [workspace.rs:6510-6538](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L6510-L6538) |
| I-729 | 활성 표시 기본값 `border_size` = 0.0, `inactive_opacity` = 1.0: **기본으로 포커스 테두리나 비활성 흐림을 더하지 않는다.** 사용자가 켜면 활성 잎에는 `border_selected`, 비활성 잎에는 `editor_background` 기반 투명 덮개를 쓴다. 기본 활성·비활성 탭 색과는 별개다 | [default.json:87-94](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L87-L94) · [pane_group.rs:1477-1525](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1477-L1525) |
| I-730 | 확대(Zoom) 버튼: 기본 `Maximize` 아이콘, 확대 상태에서는 선택 표시 + `Minimize` 아이콘. 툴팁 Zoom In/Zoom Out, 키 Shift+Escape. 확대는 창 최대화·글꼴 확대와 별개이며, 확대한 대상은 원래 그룹에서 숨기고 확대 화면으로 다룬다 | [pane.rs:4415-4430](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4415-L4430) · [pane_group.rs:542-549](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L542-L549) · [workspace.rs:6376-6394](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L6376-L6394) · [keymap:24-32](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L24-L32) |

**3.1.4 Pane 키보드 최종 적용표**

적용 keymap(Windows 기본 → VSCode 덧씌움 → specific-overrides) 결과다. 분할·이동에 관한 VSCode 덧씌움은 **없다** (I-731). 채용 열은 이 문서의 판단이다.

| ID | 컨텍스트·키 | 액션·결과 | 채용 | 출처 |
| --- | --- | --- | --- | --- |
| I-731 | 편집기 Ctrl+\ / Pane Ctrl+K Up·Down·Left·Right | Split Right / Split Up·Down·Left·Right. 기본 `ClonePane`. VSCode 덧씌움이 바꾸지 않는다 (I-018, I-019) | 채용 | [keymap:596](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L596) · [keymap:814-820](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L814-L820) · [vscode.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json) |
| I-732 | 작업 영역 Ctrl+K Ctrl+방향키 | 방향으로 Pane 포커스 이동. 가운데의 인접 Pane을 먼저 찾고 Dock·사이드바 경계까지 간다 (I-020, I-101) | 채용 | [keymap:709-712](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L709-L712) · [workspace.rs:5873-6044](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L5873-L6044) |
| I-733 | 작업 영역 Ctrl+K Shift+방향키 | Pane 배치 교환. 항목 하나를 옮기는 기능과 다르다 (I-021) | 제외 | [keymap:713-716](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L713-L716) |
| I-734 | Pane Ctrl+Shift+PageUp / PageDown | 같은 Pane 안에서 탭을 왼쪽·오른쪽으로 옮기기 (I-012) | 채용 | [keymap:503-506](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L503-L506) |
| I-735 | Pane Ctrl+W / Ctrl+F4, Ctrl+K W | 활성 탭 닫기 / 모든 탭 닫기. `close_pinned = false`. 마지막 항목이 사라지면 Pane 제거 규칙(I-728) 적용 | 채용 | [keymap:507-515](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L507-L515) · [pane.rs:2203-2213](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2203-L2213) |
| I-736 | Pane Ctrl+K Ctrl+W / Ctrl+Shift+Alt+W | 모든 탭과 Pane 닫기 / 비활성 탭과 Pane 닫기. 개별 Pane 닫기 전용 키가 아니다 | 제외 | [keymap:509-515](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L509-L515) · [workspace.rs:4487-4525](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L4487-L4525) |
| I-737 | 전역 Shift+Escape | 확대(`ToggleZoom`). 편집기 Ctrl+K Z의 가운데 정렬 레이아웃과 별개 | 채용 | [keymap:26](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L26) · [vscode.json:25-26](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L25-L26) |
| I-738 | `MoveItemToPane` / `MoveItemToPaneInDirection`: **기본 키 없음** | 대상 위치·방향으로 항목 이동. 포커스 기본 true, 복제 기본 false. 방향 대상이 없고 원래 Pane의 항목이 2개 미만이면 아무 일도 하지 않고, 나머지는 새 분할을 만들 수 있다. 키를 정하면 앱 고유 정책 (판단) | 키 없음 | [workspace.rs:508-535](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L508-L535) · [workspace.rs:6075-6119](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L6075-L6119) |
| I-739 | `SplitAndMove`방향 / `JoinIntoNext` / `JoinAll` / 다음·이전 Pane 활성화: **기본 키 없음** | 명령은 있지만 두 키맵 어디에도 연결된 키가 없다. `SplitAndMove`는 `MovePane`. VSCode의 Ctrl+1/2 같은 키를 임의로 더하면 Zed의 VSCode 덧씌움 재현과 달라진다 (판단) | 키 없음 | [pane.rs:4490-4506](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4490-L4506) · [workspace.rs:389-391](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L389-L391) · [workspace.rs:6481-6508](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L6481-L6508) |

**3.1.5 분할·이동에서 확신 낮은 부분**

- **실행 대조 (I-839):** 소스 정적 조사다. 메뉴 단축키 표시, Ctrl 드래그의 실제 커서 모양, 여러 DPI에서의 경계 픽셀은 재지 않았다.
- **크기 조절 수식 (I-840):** 80·100px는 드래그 제한이지 창이 줄어들 때의 절대 최소가 아니다. flex 보정을 단순 비례로 바꾸지 말고, 형제가 3개 이상이거나 중첩 분할일 때 포인터를 따라가는지 따로 검증해야 한다.
- **색·rem (I-841):** 토큰 이름만 확정했고 One Light·Catppuccin Mocha의 최종 RGBA는 다시 대조하지 않았다. 24px은 root rem 16 가정이다.
- **거리 같을 때, 항목 고유 드롭 (I-842):** 모서리에서 변까지 거리가 같을 때의 순서, 편집기·마크다운 미리보기가 자체 드롭 처리를 먼저 하는 세부 경우는 실행 검증 전이다. 공통 Pane 규칙보다 항목 고유 처리가 먼저라는 것만 확인했다.
- **확대와 최대화 (I-843):** 이 문서의 UI는 탭 바의 `ToggleZoom`뿐이다. 작업 영역 내부의 별도 최대화 Pane 경로는 조사하지 않았다. 둘을 섞어 기본 활성 Pane에 그림자·테두리를 더하지 않는다.

## 4. 마크다운 미리보기

| ID | 항목 | 동작 | 출처 |
| --- | --- | --- | --- |
| I-077 | 같은 Pane에 열기 | `OpenPreview`는 호출한 편집기의 Pane에 미리보기 탭을 활성화·추가하고 포커스를 준다. **원문 탭을 없애지 않는다** (기준 화면: README.md 탭과 Preview README.md 탭이 함께 있다) | [markdown_preview_view.rs:200-207](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L200-L207) · [markdown_preview_view.rs:243-251](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L243-L251) · [markdown_preview_view.rs:273-295](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L273-L295) |
| I-078 | 옆 Pane에 열기 | `OpenPreviewToTheSide`는 원문 Pane의 오른쪽 이웃을 쓰고 없으면 오른쪽 분할을 만든다. 만든 뒤 **포커스는 원문 편집기에 남긴다** | [markdown_preview_view.rs:253-270](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L253-L270) · [workspace.rs:6571-6580](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L6571-L6580) |
| I-079 | 중복 미리보기 | 대상 Pane에 같은 원문에 연결된 기본 모드 미리보기가 있으면 그것을 활성화한다. 누를 때마다 새 탭을 만들지 않는다 | [markdown_preview_view.rs:273-316](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L273-L316) |
| I-080 | 제목·아이콘 | 제목 "Preview " + 원문 파일명, 원문이 없으면 "Markdown Preview". 아이콘 `FileDoc` (책 모양, L-69) | [markdown_preview_view.rs:1664-1685](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1664-L1685) |
| I-081 | 자동 미리보기 열기 | `open_markdown_files_in_preview` 기본 **false**. true이면 깨끗한 새 원문을 미리보기로 대체할 수 있고, 수정된 원문과 복원 중에는 대체하지 않는다. 수동 `OpenPreview`와 분리해 구현한다 (우리 앱 기본 false) | [markdown_preview_settings.rs:14-25](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_settings.rs#L14-L25) · [markdown_preview_view.rs:143-195](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L143-L195) |
| I-082 | 실시간 반영 | 원문 버퍼의 편집·수정 상태 변화를 관찰해 **저장하지 않아도** 내용을 갱신한다. 다시 파싱까지 200ms 지연(debounce). 이미 대기 중이면 중복 예약하지 않는다 | [markdown_preview_view.rs:63](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L63) · [markdown_preview_view.rs:561-575](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L561-L575) · [markdown_preview_view.rs:648-719](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L648-L719) |
| I-083 | 스크롤 동기화 범위 | 확인된 것은 **원문 커서·선택 → 해당 블록 강조와 필요 시 자동 스크롤**이다. 양쪽 휠 스크롤을 같은 비율로 맞춘다는 근거는 없다 (구현에서 약속하지 않는다) | [markdown_preview_view.rs:578-589](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L578-L589) · [markdown_preview_view.rs:723-760](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L723-L760) |
| I-084 | 본문 클릭 | 원문 위치가 연결된 본문을 한 번 누르면 원문의 선택 위치만 옮긴다. 원문에 포커스를 빼앗지 않는다. 링크는 별도 경로다 | [markdown_preview_view.rs:805-824](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L805-L824) · [markdown_preview_view.rs:1171-1185](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1171-L1185) |
| I-085 | 링크 | 로컬 마크다운 링크는 미리보기로 열고 제목·줄/열 대상을 반영한다. Alt를 누르면 이 경로를 건너뛴다. 상대 링크는 원문 폴더 기준. `://` 포함 URL 등은 외부 열기 경로 | [markdown_preview_view.rs:1299-1382](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1299-L1382) |
| I-086 | 원문으로 돌아가기 | 미리보기에서 Ctrl+Shift+V는 미리보기를 닫고 원문으로 간다. 원문 탭이 이미 닫혔으면 다시 연다. 읽기만 한 미리보기를 닫아도 원문의 미저장 변경은 사라지지 않는다 | [keymap:1443-1455](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1443-L1455) · [markdown_preview_view.rs:1018-1083](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1018-L1083) |

frontmatter를 표로 그리는 조건은 [01-LAYOUT.md](01-LAYOUT.md)의 L-110·L-111에 있다.

## 5. 프로젝트 패널 조작

### 5.1 누르기·키보드

| ID | 항목 | 동작 | 출처 |
| --- | --- | --- | --- |
| I-087 | 파일 단일·더블·가운데 클릭 | **한 번 클릭**: 임시 탭으로 열고 편집기에 포커스를 주지 않는다. **더블클릭**: 일반 탭으로 열고 편집기에 포커스. **가운데 클릭**도 일반 탭 + 편집기 포커스. 우클릭·이름 입력 중에는 일반 열기를 하지 않는다 | [project_panel.rs:6504-6512](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6504-L6512) · [project_panel.rs:6607-6624](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6607-L6624) |
| I-088 | 키보드 선택 | Up/Down은 선택·스크롤만 바꾸고 파일을 열지 않는다. 열기는 Space. Windows의 Enter·F2는 이름 변경 (I-034) | [project_panel.rs:1887-1933](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1887-L1933) · [project_panel.rs:3250-3304](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L3250-L3304) · [keymap:1024-1049](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1024-L1049) |
| I-089 | 폴더 클릭·방향키 | 폴더를 누르면 펼침 토글, Alt+클릭은 하위 전체 토글. Right는 닫힌 폴더를 펼치고, 이미 펼쳤으면 다음 항목을 선택. Left는 펼친 폴더를 접고, 아니면 부모로 올라간다. 접힌 경로(`docs \ guides`)는 조각 선택도 고려한다 | [project_panel.rs:1482-1588](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1482-L1588) · [project_panel.rs:6576-6605](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6576-L6605) |
| I-090 | 복수 선택 | Shift+클릭은 보이는 항목 범위 선택. Windows Ctrl+클릭은 개별 선택 토글. Ctrl+더블클릭은 분할 열기 경로라 일반 Ctrl+선택과 구별한다 | [project_panel.rs:6514-6574](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6514-L6574) |

### 5.2 생성·이름 변경·삭제

| ID | 항목 | 동작 | 출처 |
| --- | --- | --- | --- |
| I-091 | 생성·이름 변경 | 새 파일·새 폴더·이름 변경은 트리 안의 **인라인 입력칸**으로 한다. 이름 변경은 파일이면 **확장자를 뺀 이름만**, 폴더면 이름 전체를 선택한다. Windows에서 루트 폴더의 이름 변경은 허용하지 않는다 | [project_panel.rs:2452-2541](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2452-L2541) · [project_panel.rs:2562-2623](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2562-L2623) |
| I-092 | 인라인 입력 확정·취소 | Enter = 확정. Escape = 입력을 버리고 패널로 포커스 복귀. 입력칸이 포커스를 잃으면 확정을 시도하고, 안 되면 입력을 버린다 | [project_panel.rs:1960-1963](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1960-L1963) · [project_panel.rs:2411-2419](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2411-L2419) · [project_panel.rs:828-847](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L828-L847) |
| I-093 | 휴지통과 영구 삭제 | **Trash**(휴지통)와 **Delete**(영구 삭제)는 다른 동작. 기본은 확인 창을 띄우고, 수정된 버퍼가 있으면 손실 경고를 포함한다. **루트 우클릭 메뉴에는 Trash·Delete가 없고 "Remove from Project"가 있다. 이것은 등록 해제이며 디스크 삭제가 아니다** | [project_panel.rs:2830-2888](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2830-L2888) · [project_panel.rs:2903-3006](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2903-L3006) · [project_panel.rs:1283-1299](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1283-L1299) · [project_panel.rs:4142-4152](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L4142-L4152) |
| I-094 | 드래그 앤 드롭 | `drag_and_drop` 기본 true. 선택 항목 드래그는 이동, Windows Ctrl을 누르면 복사. 루트 드래그는 루트 순서 이동이라 디스크 폴더 이동과 같지 않다 | [default.json:991-996](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L991-L996) · [project_panel.rs:5203-5206](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L5203-L5206) · [project_panel.rs:5245-5338](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L5245-L5338) |
| I-095 | 우클릭 메뉴 (채택 범위) | 파일·폴더: New File, New Folder / 마크다운 파일: Open Markdown Preview / Cut, Copy, Duplicate, Paste / Undo, Redo / Copy Path, Copy Relative Path / Rename / 루트가 아니면 Trash, Delete / 루트: Add Folders to Project…, Remove from Project / 폴더: Fold·Unfold, Expand·Collapse All. 검색·Git·터미널 항목은 제외 | [project_panel.rs:1187-1312](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1187-L1312) |
| I-096 | 자동 표시 | `auto_reveal_entries` 기본 true: 활성 파일이 바뀌면 트리에서 그 항목을 펼쳐 보인다. "활성 파일을 찾았다"와 "트리에 포커스를 가져왔다"는 별개다 (포커스는 옮기지 않는다) | [default.json:908-916](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L908-L916) · [project_panel.rs:740-748](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L740-L748) · [project_panel.rs:7252-7335](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7252-L7335) |
| I-097 | 새 파일 자동 열기 | `auto_open.on_create`·`on_paste`·`on_drop` 모두 true: 생성, 붙여넣기·복제, 외부 드롭 뒤에 파일을 연다 | [default.json:1003-1013](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1003-L1013) |

### 5.3 여러 폴더 (04-APP-SPEC에서 참조)

| ID | 구분 | 사실 / 우리 앱으로 옮길 때의 판단 | 출처 |
| --- | --- | --- | --- |
| I-105 | 세 층을 구별 | Zed 한 창의 **여러 프로젝트(workspace)**, 한 프로젝트의 **여러 루트 폴더(worktree)**, 프로젝트 아래 thread 목록은 서로 다른 층이다. 우리의 등록 폴더 목록을 thread 목록과 같은 데이터 모델이라고 설명하지 않는다 | [Windows & Projects](https://zed.dev/docs/windows-and-projects) · [Multi-root](https://zed.dev/docs/ai/parallel-agents#multi-root-folder-projects) · [workspace.rs:4334-4393](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L4334-L4393) |
| I-106 | 한 프로젝트에 폴더 추가 | `AddFolderToProject`는 폴더를 여러 개 고르게 하고 현재 프로젝트에 루트로 추가한다. 트리 루트·빈 공간 메뉴의 "Add Folders to Project…"에서 쓴다 | [workspace.rs:4334-4378](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L4334-L4378) · [project_panel.rs:1292-1299](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1292-L1299) |
| I-107 | 등록 제거 | 루트의 "Remove from Project"는 프로젝트 목록에서만 뺀다. **폴더 내용은 지우지 않는다.** 우리 사이드바의 "등록 해제"에 대응시키기 알맞다 (대응은 판단) | [project_panel.rs:4142-4152](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L4142-L4152) |
| I-108 | 여러 프로젝트 사이드바 | 기본 `default_open_behavior`·`cli_default_open_behavior`는 `existing_window`. 프로젝트마다 파일 트리와 열린 탭·분할 문맥을 유지한 채 전환한다 | [프로젝트 동작](https://zed.dev/docs/windows-and-projects) · [default.json:175-188](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L175-L188) |
| I-109 | 프로젝트 헤더 클릭 | Zed: 보통 클릭은 아래 thread 묶음을 접고 펼친다 (Ctrl+클릭이 프로젝트 활성화). **우리 앱은 자식 thread가 없으므로 일반 클릭 = 폴더 활성화**로 바꾼다 (판단, Zed 사실과 구분) | [sidebar.rs:2529-2535](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2529-L2535) · [공식 설명](https://zed.dev/docs/windows-and-projects#working-with-multiple-projects) |
| I-110 | 추가 버튼 | Zed 하단의 Add Project는 최근 프로젝트와 로컬 폴더 추가 진입을 준다. 우리는 폴더 등록 버튼만 남긴다. 폴더를 여러 개 고르면 폴더마다 독립 항목으로 등록할지 묶음으로 둘지는 04에서 정한다 (판단) | [multi-root 설명](https://zed.dev/docs/ai/parallel-agents#multiple-projects) · [sidebar.rs:6738-6776](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L6738-L6776) |
| I-111 | 권장 모델 (판단) | 요건 "선택한 폴더만 트리에 표시"에 맞추면, 등록 폴더 id별로 열린 문서·활성 탭·트리 상태를 보관하고 사이드바 클릭으로 활성 폴더 id를 바꾸는 단순 모델이 맞다. Zed 프로젝트의 여러 루트를 한 트리에 그대로 보이는 방식은 요구와 달라 선택 옵션으로 남긴다 | README 요건 1~3 |

## 6. 포커스

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| I-098 | 표시 토글 ≠ 포커스 토글 | **Ctrl+B**는 왼쪽 Dock을 열고 닫는다. **Ctrl+Shift+E**는 프로젝트 패널 포커스 토글: 바깥에서 누르면 Dock을 열고 패널에 포커스, 패널에서 다시 누르면 **가운데 Pane으로 돌아간다**. `close_panel_on_toggle` 기본 false라 두 번째 누름이 패널을 닫지 않는다 | [keymap:672-699](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L672-L699) · [workspace.rs:4805-4818](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L4805-L4818) · [workspace.rs:4863-4905](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L4863-L4905) · [default.json:202-206](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L202-L206) |
| I-099 | 사이드바 포커스 | Ctrl+Alt+;: 닫혀 있으면 열고 포커스. 열려 있고 사이드바에 포커스가 있으면 저장해 둔 이전 포커스로 돌아가고 사이드바는 남긴다. 바깥에서 들어갈 때 이전 포커스를 기록한다 | [keymap:675-676](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L675-L676) · [multi_workspace.rs:457-484](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/multi_workspace.rs#L457-L484) |
| I-100 | 사이드바 표시 토글 | Ctrl+Alt+J로 열면 포커스를 옮긴다. 닫을 때 사이드바에 포커스가 있었으면 이전 위치로, 없으면 가운데 Pane으로 복구. 토글과 포커스는 다른 동작이다 | [multi_workspace.rs:430-444](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/multi_workspace.rs#L430-L444) · [multi_workspace.rs:515-555](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/multi_workspace.rs#L515-L555) |
| I-101 | 방향 이동 | Ctrl+K Ctrl+방향키: 가운데에서는 그 방향의 다른 Pane을 먼저, 없으면 열려 있는 Dock·사이드바를 찾는다. 왼쪽 Dock에서 Right는 최근 가운데 Pane, Left는 왼쪽 사이드바. 사이드바에서 Right는 왼쪽 Dock 우선, 없으면 최근 가운데 Pane. **닫힌 Dock은 이 동작으로 열지 않는다** | [workspace.rs:5873-6044](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L5873-L6044) |
| I-102 | F6 순환 | 타이틀 바 → 열린 왼쪽 Dock → 최근 가운데 Pane → (오른쪽·아래 Dock은 우리 앱에 없음) → 상태 표시줄. Shift+F6은 역방향. 닫힌 Dock은 건너뛴다. 바깥 사이드바는 이 순환에 없다 | [keymap:638-644](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L638-L644) · [workspace.rs:8944-9049](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L8944-L9049) |
| I-103 | 마우스 따라가기 | `focus_follows_mouse.enabled` 기본 false (debounce 250ms). 마우스를 올리기만 해서 포커스가 바뀌지 않는다 | [default.json:333-336](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L333-L336) |
| I-104 | 포커스 표시 | 탭은 활성 여부와 Pane 포커스를 따로 받아 강조를 정한다. 트리는 활성 행에 `panel_focused_border`를 쓴다 (L-48). 사이드바 헤더도 포커스 시 `panel_focused_border`. 활성 Pane 전체 테두리는 기본 0이고 비활성 Pane을 흐리게 하지 않는다 (L-137) | [pane.rs:2881-2894](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2881-L2894) · [project_panel.rs:663-675](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L663-L675) · [project_panel.rs:6170-6183](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6170-L6183) · [sidebar.rs:2405-2422](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2405-L2422) · [활성 Pane 설정](https://zed.dev/docs/reference/all-settings#active-pane-modifiers) |
| L-306 | 탭 글자색과 Pane 포커스 | Pane에 포커스가 없으면(deemphasized) 마크다운 미리보기 탭의 활성 글자는 `text_muted`, 비활성 글자는 `status.hidden`으로 훨씬 옅어진다. 편집기 탭은 Pane 포커스와 무관하게 활성 `text`, 비활성 `text_muted`다. 규칙과 근거는 [01-LAYOUT.md](01-LAYOUT.md) 4.11절 | [item.rs:141-156](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/item.rs#L141-L156) · [pane.rs:2872-2895](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2872-L2895) |

## 7. 상태 복원

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| I-112 | 시작 시 복원 | `restore_on_startup` 기본 `last_session` (`last_workspace`·`none`도 선택 가능). 우리 앱 기본은 마지막 상태 복원 (판단) | [default.json:163-171](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L163-L171) |
| I-113 | 탭·분할·활성·핀·임시 | Pane은 탭 순서, 활성 탭, 핀 개수를 저장한다. 탭은 활성 여부와 **임시 여부**를 저장한다. 복원할 때 탭을 되살린 뒤 활성·임시·핀을 다시 적용한다. 분할 나무는 방향·구성원·비율을 복원한다 | [model.rs:230-299](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/persistence/model.rs#L230-L299) · [model.rs:340-425](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/persistence/model.rs#L340-L425) · [model.rs:434-449](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/persistence/model.rs#L434-L449) |
| I-114 | 미리보기 탭 복원 | 마크다운 미리보기 탭도 저장·복원 대상이다. 복원 중에는 "자동 미리보기 열기"(I-081)가 동작하지 않는다. 복원한 미리보기를 새 파일 열기와 같은 이벤트로 다시 만들지 않는다 (구현 주의, 판단) | [markdown_preview_view.rs:2214-2299](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L2214-L2299) · [markdown_preview_view.rs:154-157](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L154-L157) |
| I-115 | Dock 표시·활성·확대 | Dock의 열림, 활성 패널, 확대 상태를 복원한다. 패널이 아직 등록되지 않았으면 나중에 복원한다 | [dock.rs:826-886](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L826-L886) |
| I-116 | Dock 크기 | 패널 크기(폭)는 창 id와 패널 키를 조합한 값으로 저장·로드한다. **설정의 `default_width`와 사용자가 끌어서 저장한 크기를 구별**한다 (L-30 더블클릭은 기본값으로 초기화) | [dock.rs:376-394](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L376-L394) · [dock.rs:1257-1272](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1257-L1272) · [workspace.rs:2652-2685](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L2652-L2685) · [project_panel.rs:8274-8297](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8274-L8297) |
| I-117 | 파일 다시 열기 | `restore_on_file_reopen` 기본 true: 닫았다 다시 연 파일의 선택·접힘·스크롤을 복원한다. 트리 펼침 복원과는 별개 설정이다 | [default.json:189-197](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L189-L197) |
| I-118 | 미저장 버퍼·종료 | `session.restore_unsaved_buffers` 기본 true: 재시작 시 미저장 버퍼를 복원하고, 이 설정에서는 앱을 끌 때 저장/폐기를 묻지 않는다. 탭 하나를 닫는 확인(I-069)과 앱 종료를 혼동하지 않는다 | [default.json:2727-2734](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2727-L2734) · [pane.rs:2400-2417](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2400-L2417) |
| I-119 | 바깥 사이드바 | 활성 workspace, 사이드바 열림 여부, 사이드바 고유 상태를 저장한다. 프로젝트 전환 때 메모리 상태를 유지하는 것과 재시작 때 저장소에서 복원하는 것을 구별한다 | [multi_workspace.rs:1330-1385](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/multi_workspace.rs#L1330-L1385) · [multi_workspace.rs:1464-1492](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/multi_workspace.rs#L1464-L1492) |
| I-120 | 트리 펼침 상태 | **미확인.** 조사한 코드는 펼침 상태를 빈 값으로 시작한다. 재시작 뒤 펼침을 디스크에 저장·복원한다는 근거는 찾지 못했다. 루트 자동 펼침과 활성 파일 자동 표시(I-096)로 일부가 복원된 듯 보일 수 있다 (추정). 이 문서는 "모든 펼침이 보존된다"고 쓰지 않는다. 우리 앱의 저장 여부는 04에서 정한다 | [project_panel.rs:914-932](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L914-L932) · [project_panel.rs:1026-1032](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1026-L1032) · [project_panel.rs:740-748](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L740-L748) |

## 8. 툴팁

사용자 추가 요건: 아이콘 hover 툴팁도 Zed와 같게 한다. ID는 Zed 툴팁 조사의 `L-2xx`를 그대로 쓴다. 크기·여백·색·표시 지연·위치는 [01-LAYOUT.md](01-LAYOUT.md) 4.7절(L-206~L-216, L-222)에 있다. 이 절은 **우리 앱에 남기는 요소**의 문구와 동작만 적는다. 기록·Git·AI 같은 제외 요소와 우리에게 없는 읽기 전용 탭·심볼릭 링크 표시는 뺐다.

### 8.1 종류와 규칙

| ID | 생성 방법 | 내용 | 출처 |
| --- | --- | --- | --- |
| L-200 | 문구만 (`Tooltip::text`) | 제목 한 줄 | [tooltip.rs:37-56](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L37-L56) |
| L-201 | 문구 + 액션 (`for_action`) | 제목 + 그 액션의 **단축키**(오른쪽). 단축키는 전역 키맵에서 찾는다 | [tooltip.rs:96-107](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L96-L107) |
| L-202 | 문구 + 액션 + 포커스 (`for_action_in`) | 위와 같지만 지정한 포커스 위치의 키 컨텍스트에서 단축키를 찾는다 (예: Pane 안에서만 유효한 키) | [tooltip.rs:109-121](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L109-L121) |
| L-203 | 메타 있음 (`with_meta`) | 제목(+단축키) 아래에 **메타 줄** | [tooltip.rs:123-150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L123-L150) |
| L-204 | 임의 요소 (`element`) | 예: 사이드바 토글의 두 줄 툴팁 (L-232) | [tooltip.rs:160-181](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L160-L181) |
| L-205 | 메뉴 여는 버튼 | **메뉴가 열려 있는 동안에는 툴팁을 달지 않는다.** 열려 있으면 버튼이 선택 상태 | [popover_menu.rs:198-212](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L198-L212) |
| L-206 | 지연 | 500ms. 항목 이름만 1500ms (L-247) | 01-LAYOUT 4.7 |
| L-207 | 사라짐 | 마우스가 벗어남 · 버튼을 누름 · 스크롤 | 01-LAYOUT 4.7 |
| — | 메뉴 버튼 공통 (8.3절 L-272와 같음) | ☰, ∨, 프로젝트 이름, `+`, 분할, 폴더 추가의 메뉴·팝업이 열려 있으면 툴팁 없음 (L-205) | — |
| — | 비활성 버튼 | ← → 같은 비활성 버튼에도 툴팁은 **그대로 뜬다.** 버튼은 비활성이어도 툴팁 설정을 건너뛰지 않고 클릭 처리만 막는다 (검토 지적 B-09, 소스 확인. 비활성 요소에서 마우스 올림 이벤트가 오는지는 gpui 쪽 미확인) | [button_like.rs:883-895](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L883-L895) |

### 8.2 요소별 문구

단축키 열은 8.4절 규칙을 **적용 keymap(VSCode 덧씌움 후)**에 적용한 결과다. 채택 범위의 키는 덧씌움 전후가 같다 (I-832): Ctrl-R, Ctrl-Alt-J, Ctrl-Alt-;, Alt-Left/Right, Ctrl-W, Ctrl-K Shift-Enter, Shift-Escape, Ctrl-Shift-E, Ctrl-B, Ctrl-G, Ctrl-K Ctrl-T. Split Pane은 문구만 있는 툴팁이라 키가 없다. 우리 앱의 키맵이 Zed와 다르면 **우리 키맵의 키를 표시**한다 (판단). 문구는 화면에 보이는 짧은 UI 문자열이라 그대로 적는다.

**타이틀 바**

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
| --- | --- | --- | --- | --- | --- |
| L-225 | ☰ 메뉴 버튼 | **Open Application Menu** | 문구만. 메뉴가 열려 있으면 없음 | 없음 (F10에 묶여 있지만 툴팁에는 표시하지 않는다) | [application_menu.rs:192-200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L192-L200) · [default-windows.json:662](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L662) |
| L-226 | 프로젝트(폴더) 이름 버튼 | **Recent Projects** | 문구 + 액션. 메뉴가 열려 있으면 없음 | `projects::OpenRecent` → **Ctrl-R**. 사용자 결정으로 Open Recent를 포함하므로 우리 앱의 이 버튼도 Recent Projects 팝업(최근 폴더 + 등록 폴더 목록)을 연다. 문구와 키는 Zed 그대로 | [title_bar.rs:831-858](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L831-L858) · [default-windows.json:648](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L648) |
| L-227 | 프로젝트 이름 버튼, 폴더가 없을 때 | 버튼 글자가 "Open Recent Project" (Muted). 툴팁은 L-226과 같다 | — | 위와 같음 | [title_bar.rs:792-816](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L792-L816) · [title_bar.rs:864-912](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L864-L912) |
| L-262 | ∨ 메뉴 버튼 | **툴팁 없음.** 접근성 이름(aria-label) "User menu"만 있고 마우스 hover 툴팁은 달지 않는다 (10.4절 I-637). 메뉴가 열려 있어도 툴팁 없음 | — | — | [title_bar.rs:1258-1267](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1258-L1267) |
| L-228 | 최소화·최대화·복원·닫기 | **툴팁 없음.** Zed는 창 버튼에 툴팁을 달지 않는다. 우리도 달지 않는다 | — | — | [platform_windows.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs) |

**사이드바**

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
| --- | --- | --- | --- | --- | --- |
| L-230 | 폴더 추가 (`FolderAdd` 아이콘) | **Add Project** | 문구 + 액션. 메뉴가 열려 있으면 없음 | `projects::OpenRecent` → **Ctrl-R**. 우리 앱의 폴더 추가 키는 Zed에 대응이 없으므로 04에서 정한 키를 표시한다 (I-029) | [sidebar.rs:6770-6776](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L6770-L6776) |
| L-231 | 프로젝트(폴더) 헤더 행 | **툴팁 없음** | — | — | [sidebar.rs:2326](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2326) · [sidebar.rs:2331-2440](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2331-L2440) |
| L-232 | 사이드바 닫기 토글 (사이드바가 열려 있을 때) | **두 줄 툴팁.** 1줄 "**Toggle Sidebar**" + 단축키 / 구분선(위 padding 4, 1px `border_variant`) / 2줄 "**Focus Sidebar**" + 단축키. 각 줄은 gap 8, 양끝 정렬, 두 줄 사이 gap 4 | 임의 요소 | `multi_workspace::ToggleWorkspaceSidebar` → **Ctrl-Alt-J** / `FocusWorkspaceSidebar` → **Ctrl-Alt-;** | [sidebar.rs:7495-7518](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7495-L7518) · [default-windows.json:675-676](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L675-L676) |

**탭 바**

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
| --- | --- | --- | --- | --- | --- |
| L-236 | ← 뒤로 | **Go Back** | 문구 + 액션 + 포커스 (현재 포커스, 없으면 Pane) | `pane::GoBack` → **Alt-Left** | [pane.rs:3508-3530](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3508-L3530) · [default-windows.json:516-518](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L516-L518) |
| L-237 | → 앞으로 | **Go Forward** | 문구 + 액션 + 포커스 | `pane::GoForward` → **Alt-Right** | [pane.rs:3531-3553](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3531-L3553) · [default-windows.json:519-521](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L519-L521) |
| L-238 | 탭 본체 (편집기 탭) | **파일 전체 경로**: 절대 경로, 홈 디렉터리는 `~`로 줄인다. 예: `D:\projects\docs\guides\rust-gui\README.md`. 경로가 없는 버퍼는 제목 | 문구만 | 없음 | [pane.rs:3099-3125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3099-L3125) · [items.rs:706-726](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L706-L726) |
| L-239 | 미리보기 탭 ("Preview X.md") | **원본 편집기 탭과 같은 툴팁** (원본 파일 전체 경로) | 문구만 | 없음 | [markdown_preview_view.rs:1745-1751](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1745-L1751) |
| L-240 | 탭 닫기 × | **Close Tab.** **활성 탭에서만 단축키**가 붙고, 비활성 탭은 문구만 | 활성: 문구 + 액션 + 포커스 / 비활성: 문구만 | `pane::CloseActiveItem` (핀 닫기 없음) → **Ctrl-W** | [pane.rs:3053-3085](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3053-L3085) · [default-windows.json:507-508](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L507-L508) |
| L-241 | 고정(pinned) 탭의 핀 | **Unpin Tab** (활성 탭이면 단축키) | 위와 같음 | `pane::TogglePinTab` → Ctrl-K Shift-Enter | [pane.rs:3036-3053](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3036-L3053) · [default-windows.json:534](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L534) |
| L-243 | + (새 항목) | **New…** (말줄임표 문자 `…`) | 문구만. 메뉴가 열려 있으면 없음 | 없음 | [pane.rs:4364-4370](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4364-L4370) |
| L-244 | 분할 | **Split Pane** | 문구만. 메뉴가 열려 있으면 없음 | 없음 | [pane.rs:4388-4394](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4388-L4394) |
| L-245 | 최대화(줌) | 평소 **Zoom In**, 확대 상태 **Zoom Out** | 문구 + 액션 | `workspace::ToggleZoom` → **Shift-Escape** | [pane.rs:4417-4430](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4417-L4430) · [default-windows.json:26](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L26) |
| L-242 | 읽기 전용 자물쇠 (아이콘 자리) | **없음.** Zed의 "Unlock Tab"(메타 "This will make this tab editable")·"Locked Tab"(메타 "This tab is read-only") 툴팁은 만들지 않는다 | — | — | 탭 읽기 전용(Make Tab Read-Only / Unlock Tab)은 사용자 결정으로 제외 |

**프로젝트 패널**

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
| --- | --- | --- | --- | --- | --- |
| L-247 | 항목 hover | **항목의 절대 경로** (홈은 `~`). 지연 **1500ms** (`title_tooltip_delay` 기본). 설정으로 다른 값 또는 끔 | 문구만 | 없음 | [project_panel.rs:6234-6268](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6234-L6268) · [default.json:886](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L886) |
| L-249 | 패널 상단·하단 아이콘 | **없음.** 프로젝트 패널 안에는 아이콘 버튼이 없다 | — | — | [project_panel.rs:8157](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8157) |

**상태 표시줄**

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
| --- | --- | --- | --- | --- | --- |
| L-251 | 사이드바 토글 (사이드바가 닫혀 있을 때) | **Open Folder Sidebar.** Zed 문구는 "Open Threads Sidebar"인데 우리 앱에는 Threads가 없어 Threads를 Folder로 바꿨다 (판단) | 문구 + 액션 | `ToggleWorkspaceSidebar` → **Ctrl-Alt-J** | [status_bar.rs:241-270](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L241-L270) |
| L-252 | 폴더 추가 | Zed 상태 표시줄에는 없다. 사이드바의 "Add Project"(L-230)가 해당 | — | (Ctrl-R) | — |
| L-253 | 프로젝트 패널 토글 (패널이 **닫혀 있거나 다른 패널이 활성**일 때) | **Project Panel** | 문구 + 액션 | `project_panel::ToggleFocus` → **Ctrl-Shift-E** | [dock.rs:1416-1441](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1416-L1441) · [project_panel.rs:8284-8290](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8284-L8290) · [default-windows.json:699](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L699) |
| L-254 | 프로젝트 패널 토글 (패널이 **열려 있고 활성**) | **Close Left Dock** (패널이 오른쪽 Dock이면 "Close Right Dock") | 문구 + 액션 | `workspace::ToggleLeftDock` → **Ctrl-B** | [dock.rs:1431-1441](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1431-L1441) · [dock.rs:360-366](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L360-L366) · [default-windows.json:672-673](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L672-L673) |
| L-256 | 커서 위치 ("9:11") | **Go to Line/Column** (누르면 줄 이동 대화상자) | 편집기 포커스가 있으면 문구 + 액션 + 포커스, 없으면 문구 + 액션 | `go_to_line::Toggle` → **Ctrl-G** (편집기 컨텍스트) | [cursor_position.rs:248-260](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L248-L260) · [actions.rs:407-412](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/actions.rs#L407-L412) · [default-windows.json:626-630](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L626-L630) |

**테마·아이콘 테마 선택 (요건 7)**

| ID | 항목 | 내용 | 출처 |
| --- | --- | --- | --- |
| L-258 | 전용 버튼 | Zed 화면에는 테마 선택 **아이콘 버튼이 없다**. 그래서 툴팁도 없다. 메뉴 항목으로만 나온다 | [app_menus.rs:87-95](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L87-L95) |
| L-259 | ☰ 메뉴 항목 | Settings 하위에 "**Select Theme...**" (`theme_selector::Toggle`), "**Select Icon Theme...**" (`icon_theme_selector::Toggle`). 메뉴 항목 오른쪽에 단축키를 그린다 | [app_menus.rs:87-95](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L87-L95) · [context_menu.rs:1805-1806](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1805-L1806) |
| L-260 | ∨ 사용자 메뉴 항목 | ∨ 메뉴에 "Themes…"(Ctrl-K Ctrl-T), "Icon Themes…"(키 없음) 항목이 있다 (10.2절 I-622, I-623). 항목에는 hover 툴팁이 없다 (10.4절) | [title_bar.rs:1374-1383](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1374-L1383) |
| L-261 | 단축키 | `theme_selector::Toggle` → **Ctrl-K Ctrl-T**. `icon_theme_selector::Toggle`은 Windows 기본 키맵에 바인딩이 없다 | [default-windows.json:688](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688) |

**편집기 툴바와 찾기 창** (편집기 조사 E-ID. 동작은 13.3절)

| ID | 요소 | 툴팁 문구 | 종류 | 액션 → 표시 단축키 | 출처 |
| --- | --- | --- | --- | --- | --- |
| — | (참조 E-039) 툴바 마크다운 미리보기 버튼 (`Eye`, 마크다운 파일만) | **Preview Markdown**, 메타 줄 "**Alt-Click to open in a split**" | 문구 + 액션 + 메타 | `markdown::OpenPreview` → **Ctrl-Shift-V** | [preview.rs:38-80](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/quick_action_bar/preview.rs#L38-L80) |
| — | (참조 E-039) 툴바 찾기 버튼 | **Buffer Search** | 문구 + 액션 | `buffer_search::Deploy` → Ctrl-F (표시 여부는 미확인) | [quick_action_bar.rs:255-320](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/quick_action_bar.rs#L255-L320) |
| — | (참조 E-068) 찾기 창 닫기 | **Close Search Bar** | 문구 + 액션 + 포커스 | Escape | [buffer_search.rs:443-470](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L443-L470) |
| — | (참조 E-076) 다음 / 이전 일치 | **Select Next Match** / **Select Previous Match** | 문구 + 액션 + 포커스 | Enter / Shift-Enter (F3 / Shift-F3도 있다) | [keymap:409-423](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L409-L423) · [keymap:522-523](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L522-L523) |
| — | (참조 E-077) 모든 일치 선택 | **Select All Matches** | 문구 + 액션 + 포커스 | Alt-Enter | 위와 같음 |
| — | (참조 E-078) 옵션 토글 | **Match Case Sensitivity** / **Match Whole Words** / **Use Regular Expressions** | 문구 + 액션 + 포커스 | Alt-C / Alt-W / Alt-R | [keymap:524-532](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L524-L532) |
| — | (참조 E-079) 바꾸기 토글 / 선택 영역 찾기 | **Toggle Replace** / **Toggle Search Selection** | 문구 + 액션 + 포커스 | Ctrl-H, Shift-Alt-H / Ctrl-L, Alt-L | [keymap:418-420](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L418-L420) · [keymap:525-526](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L525-L526) |
| — | (참조 E-079a) 바꾸기 다음 / 모두 | **Replace Next Match** / **Replace All Matches** | 문구 + 액션 + 포커스 | Enter / Ctrl-Enter (바꾸기 입력칸) | [keymap:425-431](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L425-L431) |

찾기 창 버튼은 모두 찾기 입력칸의 포커스 기준으로 키를 찾으므로 단축키가 함께 보인다 (E-079d).

### 8.3 상태별 문구

| ID | 요소 | 상태 → 문구 |
| --- | --- | --- |
| L-265 | 최대화(줌) 버튼 | 확대 아님 → "Zoom In" / 확대됨 → "Zoom Out" (아이콘도 Maximize → Minimize) |
| L-266 | 프로젝트 패널 토글 버튼 | 닫힘·비활성 → "Project Panel" + 패널 토글 키 / 열림·활성 → "Close Left Dock" + Dock 토글 키 |
| L-267 | 탭 끝 슬롯 | 일반 탭 → "Close Tab" / 고정 탭 → "Unpin Tab". 활성 탭만 단축키 표시 |
| L-268 | 탭 본체 | 항상 경로 (읽기 전용 탭이 없으므로 "Read-Only Tab" 줄은 쓰지 않는다) |
| L-269 | 자물쇠 | **제외** (읽기 전용 탭이 없다). "Unlock Tab" · "Locked Tab" 문구와 메타 줄은 쓰지 않는다 |
| L-270 | 사이드바 토글 | 사이드바 닫힘 → 상태 표시줄 "Open Folder Sidebar"(Zed: "Open Threads Sidebar") / 열림 → 사이드바 안 버튼의 두 줄 "Toggle Sidebar" + "Focus Sidebar" |
| L-272 | 메뉴를 여는 버튼 | 메뉴·팝업이 열려 있으면 툴팁 없음 (L-205) |

### 8.4 단축키 표시 형식 (Windows)

툴팁의 단축키는 1절 키 표의 `Ctrl+B`와 **표기가 다르다.** 툴팁은 아래 규칙으로 그린다.

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-217 | 어떤 키를 보여주나 | 액션에 묶인 키 중 **우선순위가 가장 높은 하나**. 키맵에 나중에 추가된 바인딩이 우선이고 (사용자 키맵이 기본 키맵을 이긴다), 포커스를 받는 종류는 포커스 위치의 컨텍스트를 먼저 본다 | [keybinding.rs:82-99](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L82-L99) · [keymap.rs:137-159](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/keymap.rs#L137-L159) |
| L-218 | 글자 방식 | Windows·Linux는 아이콘 없이 **문자열**로 그린다 (macOS만 ⌘ 같은 기호) | [keybinding.rs:342-359](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L342-L359) |
| L-219 | 구분자·순서 | 수식키 사이 구분자는 **`-`(하이픈)**이고 `+`가 아니다. 순서는 **Fn → Ctrl → Win → Alt → Shift → 키**. 예: `Ctrl-B`, `Ctrl-Shift-E`, `Ctrl-Alt-J` | [keybinding.rs:691-760](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L691-L760) |
| L-220 | 키 이름 | 첫 글자를 대문자로: `left` → `Left`, `escape` → `Escape`, `g` → `G`. 예외 `pageup` → `PageUp`, `pagedown` → `PageDown` | [keybinding.rs:752-760](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L752-L760) |
| L-221 | 연속 키 (chord) | 키 입력마다 덩어리 하나, 사이 gap 4px. 예: `Ctrl-K Ctrl-T` | [keybinding.rs:280-288](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L280-L288) |
| L-222 | 키 글자 모양 | 14px, Muted 색 (01-LAYOUT 4.7) | [keybinding.rs:535-560](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L535-L560) |
| L-223 | 바인딩이 없을 때 | 단축키 영역 없이 제목만 (추정) | [keybinding.rs:196-205](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L196-L205) |
| — | 키 고르기 보충 (I-704, 1.1절) | 인수까지 같은 바인딩만 후보로 삼고, 현재 컨텍스트에서 다른 액션에 가려진 키는 뺀다. 그래서 **같은 액션이라도 인수가 다른 키는 표시되지 않는다** (예: 탭 바 분할 팝업의 `MovePane` 항목에 `ClonePane` 키를 붙이지 않는다, I-833) | [key_dispatch.rs:396-445](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/key_dispatch.rs#L396-L445) · [keymap.rs:91-114](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/keymap.rs#L91-L114) |

### 8.5 툴팁에서 확신 낮은 부분

- **표시 단축키 선택 (L-236, L-237, L-240):** VSCode 덧씌움은 이 키들을 바꾸지 않는다 (I-832). "Alt-Left", "Alt-Right", "Ctrl-W"는 "같은 컨텍스트에서 마지막에 정의된 바인딩이 우선"(L-217)을 키맵 순서에 적용한 **추정**이다. 실제 화면으로 확인하지 않았다. 포커스가 편집기 안이면 더 깊은 컨텍스트의 바인딩이 이길 수 있다.
- **줄 번호:** L-225, L-236~L-245, L-247은 ±수 줄 차이가 날 수 있다.
- **창 버튼 (L-228):** Zed 코드에는 툴팁이 없다. Windows가 시스템 툴팁이나 스냅 레이아웃을 보여 주는지는 미확인이다.
- **위치 고정 (L-208):** 처음 뜬 마우스 위치에 고정되고 따라가지 않는다는 것은 코드에 근거한 추정이다.
- **px 값 (L-211, L-215):** `rounded_lg` 8px, 최대 폭 288px은 tailwind 단위 관례에 따른 추정이다.
- **사이드바 툴팁 키 (L-232):** "Ctrl-Alt-;"는 `;`가 대문자화되지 않는다는 규칙에서 나온 추정 표기다.
- **우리 앱 문구 (L-226, L-251):** L-226은 Zed 문구 "Recent Projects"를 그대로 쓰고(Open Recent 포함), L-251은 Threads를 Folder로 바꾼 판단이다.

## 9. ☰ 앱 메뉴

사용자 요건: 타이틀 바 ☰ 앱 메뉴는 Zed와 같은 구조로 하되 우리 앱에서 쓸 수 있는 항목만 남긴다. 값은 앱 메뉴 조사(`zed-app-menu.md`)의 ID(I-300~I-488)를 그대로 쓴다. 크기·여백·색은 [01-LAYOUT.md](01-LAYOUT.md) 4.8절에 있다. 이 절은 표시 방식(9.1)과 유지 항목만 남긴 최종 구조(9.2)를 적는다. 포함·제외는 사용자 결정(2026-10-31, `_decisions.md`)에 따라 확정했고, 뺀 항목은 9.4에 이유와 함께 적었다.

### 9.1 표시 방식

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| I-300 | 메뉴 원본 | 메뉴 정의 한 곳이 전체 항목·순서·구분선을 가진다. 최상위 순서: Zed, File, Edit, Selection, View, Go, Run, Window, Help. 모든 항목의 비활성 기본값은 false | [app_menus.rs:69-174](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L69-L174) · [app_menus.rs:221-340](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L221-L340) |
| I-301 | Windows 표시 | Windows 네이티브 메뉴를 띄우지 않는다. 앱이 **직접 그린 메뉴**(팝업 메뉴 + 목록 구성요소)다. 우리도 직접 그린다 | [title_bar.rs:452-466](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L452-L466) · [application_menu.rs:126-174](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L126-L174) |
| I-302 | ☰ 클릭 모델 | `title_bar.show_menus` = false, `open_menus_on_hover` = false가 기본. 닫힌 상태에는 첫 메뉴(앱 이름)의 `Menu` 아이콘 **하나만** 보인다. 누르면 첫 메뉴가 열리고 **열려 있는 동안 가로 메뉴 바**(Zed, File, Edit, …)가 나타난다. ☰ 아래에 File·Edit를 세로로 나열하는 계층형 메뉴가 **아니다** | [default.json:636-640](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L636-L640) · [application_menu.rs:176-204](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L176-L204) · [application_menu.rs:307-367](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L307-L367) |
| I-303 | 가로 메뉴 전환 | 메뉴 이름은 Small 라벨의 Subtle 버튼. 어느 메뉴가 열려 있으면 **다른 이름에 마우스를 올리는 것만으로** 이전 메뉴를 닫고 새 메뉴를 연다. `open_menus_on_hover`가 true이면 먼저 누르지 않아도 hover로 열린다. Left/Right 키는 끝에서 반대편으로 순환한다 | [application_menu.rs:206-248](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L206-L248) · [application_menu.rs:263-304](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L263-L304) |
| I-304 | 하위 메뉴 없음 | 원본 데이터에는 Zed > Settings, View > Editor Layout 묶음이 있지만 Windows 변환은 그 안의 항목과 구분선을 **부모 메뉴에 평탄하게 넣는다.** 별도 하위 메뉴·오른쪽 화살표가 그려지지 않는다. 더 깊은 하위 메뉴·시스템 메뉴는 건너뛴다. 우리도 중첩 메뉴를 만들지 않는다 | [application_menu.rs:135-173](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L135-L173) |
| I-305 | 일반 하위 메뉴 (참고) | 일반 컨텍스트 메뉴의 하위 메뉴는 hover·클릭·키보드로 옆에 열고, 오른쪽 공간이 200px보다 작고 왼쪽이 더 넓으면 왼쪽으로 뒤집는다. 트리거와 2px 겹치고 창 가장자리 여유 8px. **앱 메뉴 경로에서는 쓰지 않는다** (우클릭 메뉴 11절에서 다시 본다) | [context_menu.rs:1539-1689](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1539-L1689) · [context_menu.rs:1707-1767](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1707-L1767) |
| I-306 | 구분선 정리 | 연속된 구분선은 하나로 줄이고, 마지막 구분선은 지우고, 비어 있는 그룹은 건너뛴다. 평탄화된 Settings 안의 구분선은 그 자리에 남는다. 우리 최종안은 빈 그룹을 아예 만들지 않아 선두·연속·끝 구분선이 생기지 않는다 (판단) | [application_menu.rs:92-124](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L92-L124) · [application_menu.rs:135-173](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L135-L173) |
| I-307 | 단축키 표시 | 메뉴를 열기 직전의 포커스를 저장하고, 그 컨텍스트에서 같은 액션의 **우선순위가 가장 높은 키 하나**를 오른쪽에 보인다. 없으면 창 전체에서 찾는다. 여러 키를 모두 적지 않는다. 사용자 키맵이 반영된다. 라벨과 단축키 사이 16px | [application_menu.rs:132-137](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L132-L137) · [context_menu.rs:2026-2044](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2026-L2044) · [keybinding.rs:188-204](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L188-L204) |
| I-308 | 비활성 표시 | 비활성 항목은 라벨 `text_disabled`, 아이콘 Muted, 단축키 Disabled. 클릭과 키보드 선택 대상에서 빠진다. 원본 메뉴 정의는 항목을 모두 활성으로 만들므로 "저장할 파일이 없으면 Save가 회색"이 Zed의 자동 동작이라는 근거는 없다 (우리 앱의 비활성 계산은 우리 설계, I-480) | [context_menu.rs:1813-1825](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1813-L1825) · [context_menu.rs:1923-1940](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1923-L1940) · [context_menu.rs:2203-2212](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2203-L2212) · [app_menu.rs:138-175](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/platform/app_menu.rs#L138-L175) |
| I-309 | 실행 시 포커스 | 항목을 실행하기 전에 **원래 포커스를 복원**한 뒤 명령을 보낸다. 메뉴에서 Copy·Find를 실행해도 메뉴 자신이 대상이 되지 않게 하는 규칙이다 | [context_menu.rs:706-725](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L706-L725) |
| I-310 | 키보드 | 작업 영역에서 **F10** = 앱 메뉴 열기. 메뉴가 열려 있을 때 F10 = 닫기, Left/Right = 이전·다음 메뉴. 메뉴 목록 안에서 Up/Down·Home/End = 선택 이동, Enter = 실행, Escape = 닫기. 구분선과 비활성 항목은 건너뛴다. VSCode 덧씌움은 디버그 세션 컨텍스트에서만 F10(Step Over)·F11을 가로채므로 **디버그가 없는 우리 앱에서는 F10 = 앱 메뉴, F11 = 전체 화면 그대로**다 (I-831) | [keymap:5-20](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L5-L20) · [keymap:662](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L662) · [keymap:786-792](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L786-L792) · [context_menu.rs:1098-1132](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1098-L1132) · [context_menu.rs:2203-2212](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2203-L2212) · [vscode.json:67-73](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L67-L73) |
| I-311 | 조건부·플랫폼 항목 | Windows에서 만들어지지 않는 항목(Services, Hide Zed, Hide Others, Show All, Install CLI)과 AI·개발 채널 전용 항목은 우리 앱에 없다 | [app_menus.rs:50-67](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L50-L67) · [app_menus.rs:97-112](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L97-L112) |

### 9.2 최종 메뉴 구조 (유지 항목만)

기준: 조사 자료의 "유지" 판정 항목에, 우리 기능 진입점으로 새로 배치한 항목(†)을 더했다. 기존 항목의 **상대 순서와, 살아남은 그룹 사이의 구분선은 Zed 그대로**다. 전부 사라진 그룹의 구분선은 없앴고 연속·끝 구분선도 없다. 추가(†) 항목은 가장 가까운 기존 그룹 안에 넣어 구분선의 의미를 지켰다. Run 메뉴는 유지 항목이 없어 최상위 자체를 뺐다. Settings·Editor Layout 묶음은 Windows 원본처럼 부모 메뉴에 평탄하게 둔다 (I-304).

- `†` = 원본 메뉴 정의에는 없지만 요구 기능(마크다운 미리보기·사이드바·탭 이동·바꾸기·최대화)의 진입점으로 **우리가 추가한 항목**이다. Zed에 있던 항목이라고 쓰지 않는다 (I-481, 판단).
- 이름의 `앱 이름`은 Zed 메뉴·"Quit Zed"의 Zed를 우리 앱 이름으로 바꾼 것이다 (I-341). 라이선스 항목은 우리 배포물의 고지로 새로 만든다 (I-450).
- 키는 **적용 keymap(VSCode 덧씌움 후)**이다. 같은 액션에 키가 여러 개면 화면에는 하나만 보이므로(I-307) 후보를 모두 적었다. 덧씌움으로 표시가 바뀐 행은 **기본→VSCode**로 적었다 (Toggle Line Comment와 Add Folder to Project 두 곳, I-826, I-827).

```text
☰ → 펼쳐진 가로 메뉴: [앱 이름] [File] [Edit] [Selection] [View] [Go] [Window] [Help]

앱 이름
├─ About <앱 이름>
├─ ────────────────
├─ Open Settings File
├─ ────────────────
├─ Open Keymap
├─ Open Keymap File
├─ ────────────────
├─ Select Theme...
├─ Select Icon Theme...
├─ ────────────────
├─ Extensions
├─ ────────────────
└─ Quit <앱 이름>

File
├─ New
├─ New Window
├─ ────────────────
├─ Open File...
├─ Open Folder...
├─ Open Recent…
├─ ────────────────
├─ Add Folder to Project…
├─ ────────────────
├─ Save
├─ Save As…
├─ Save All
├─ ────────────────
├─ Close Editor
└─ Close Window

Edit
├─ Undo
├─ Redo
├─ ────────────────
├─ Cut
├─ Copy
├─ Copy and Trim
├─ Paste
├─ ────────────────
├─ Find
├─ Replace †
├─ ────────────────
└─ Toggle Line Comment

Selection
├─ Select All
├─ Expand Selection
├─ Shrink Selection
├─ Select Next Sibling
├─ Select Previous Sibling
├─ ────────────────
├─ Add Cursor Above
├─ Add Cursor Below
├─ Select Next Occurrence
├─ Select Previous Occurrence
├─ Select All Occurrences
├─ ────────────────
├─ Move Line Up
├─ Move Line Down
└─ Duplicate Selection

View
├─ Zoom In
├─ Zoom Out
├─ Reset Zoom
├─ Reset All Zoom
├─ ────────────────
├─ Toggle Left Dock
├─ Toggle All Docks
├─ Toggle Folder Sidebar †
├─ Focus Folder Sidebar †
├─ Split Up
├─ Split Down
├─ Split Left
├─ Split Right
├─ Maximize Pane †
├─ ────────────────
├─ Project Panel
├─ Open Markdown Preview †
└─ Open Markdown Preview to the Side †

Go
├─ Back
├─ Forward
├─ Previous Tab †
├─ Next Tab †
├─ ────────────────
├─ Command Palette...
├─ ────────────────
├─ Go to File...
└─ Go to Line/Column...

Window
├─ Minimize
└─ Zoom

Help
└─ View Dependency Licenses
```

문구의 `...`(점 세 개)와 `…`(말줄임표 한 글자)는 원본대로 구별했다. 표의 "구분선" 행은 위 그림의 `────` 위치다.

**앱 이름**

| ID | 문구 | 액션 | 키 | 비고 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-322 | About <앱 이름> | `zed::About` | 없음 | 우리 앱 정보 창 (사용자 결정으로 포함). 문구의 Zed를 앱 이름으로 치환 | [app_menus.rs:74](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L74) |
| — | 구분선 | | | | I-324 |
| I-327 | Open Settings File | `zed::OpenSettingsFile` | Ctrl+Alt+, / Ctrl+, | **settings.json을 탭으로 열어 편집**한다 (사용자 결정). 설정 화면은 만들지 않는다 (원본의 Open Settings는 제외, 9.4). Ctrl+,는 Zed에서 설정 화면 키이며 우리는 이 동작에 연결한다 (판단) | [app_menus.rs:79](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L79) · [keymap:33](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L33) · [keymap:1536](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1536) |
| — | 구분선 | | | | I-331 |
| I-332 | Open Keymap | `zed::OpenKeymap` | Ctrl+K Ctrl+S | **키맵 편집 화면**(Zed Keymap Editor와 같게, 사용자 결정). 화면 명세는 15절 | [app_menus.rs:84](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L84) · [keymap:687](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L687) |
| I-333 | Open Keymap File | `zed::OpenKeymapFile` | 일반 편집 컨텍스트에는 없음 (키맵 편집 화면 안에서 Ctrl+E) | keymap.json을 탭으로 연다 (사용자 결정) | [app_menus.rs:85](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L85) · [keymap:1473](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1473) |
| — | 구분선 | | | | I-335 |
| I-336 | Select Theme... | `theme_selector::Toggle` | Ctrl+K Ctrl+T | 요건 7의 테마 목록(One Dark·One Light·Catppuccin 4종)으로 제한 | [app_menus.rs:88-91](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L88-L91) · [keymap:688](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688) |
| I-337 | Select Icon Theme... | `icon_theme_selector::Toggle` | 없음 | 아이콘 테마 목록(Zed 기본·Catppuccin Icons)으로 제한 | [app_menus.rs:92-95](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L92-L95) |
| — | 구분선 | | | | I-338 |
| I-339 | Extensions | `zed::Extensions` | Ctrl+Shift+X | **확장 관리 화면**을 연다 (16절, 사용자 지시로 포함. 테마·아이콘 테마·언어 확장만) | [app_menus.rs:101](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L101) · [keymap:717](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L717) |
| — | 구분선 | | | | I-340 |
| I-341 | Quit <앱 이름> | `zed::Quit` | Ctrl+Q | 문구의 Zed를 앱 이름으로 치환 | [app_menus.rs:112](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L112) · [keymap:34](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L34) |

**File**

| ID | 문구 | 액션 | 키 | 비고 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-342 | New | `workspace::NewFile` | Ctrl+N | | [app_menus.rs:119](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L119) · [keymap:659](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L659) |
| I-343 | New Window | `workspace::NewWindow` | Ctrl+Shift+N | | [app_menus.rs:120](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L120) · [keymap:660](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L660) |
| — | 구분선 | | | | I-344 |
| I-345 | Open File... | `workspace::OpenFiles` | Ctrl+O | | [app_menus.rs:123](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L123) · [keymap:27](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L27) |
| I-346 | Open Folder... | `workspace::Open` | Ctrl+K Ctrl+O | 우리 앱에서는 등록 폴더 선택 흐름에 연결 | [app_menus.rs:124-131](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L124-L131) · [keymap:28](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L28) |
| I-347 | Open Recent… | `projects::OpenRecent` | Ctrl+R | 최근 폴더와 등록 폴더 목록 팝업 (사용자 결정으로 포함, 14.4절) | [app_menus.rs:132](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L132) · [keymap:648](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L648) |
| — | 구분선 | | | | I-349 |
| I-350 | Add Folder to Project… | `workspace::AddFolderToProject` | **기본→VSCode 보정:** 표시하는 키 없음. Ctrl+Shift+A를 전역 키로 표시하지 않는다 | 일반 편집기에서 Ctrl+Shift+A는 블록 주석(I-745), 최근 프로젝트 선택기 입력칸에서는 `picker::ToggleActionsMenu`가 이긴다. 원래 최근 프로젝트 바인딩은 남지만 그 깊은 입력칸에서는 가려진다. 포커스마다 달라져 상수로 고정하지 않는다 (I-827). 우리 앱에서는 **폴더 등록**에 대응. 전역 키를 정하면 우리 앱의 변경 사항으로 적는다 (I-468, I-029) | [app_menus.rs:135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L135) · [keymap:1303-1311](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1303-L1311) · [specific-overrides.json:8-14](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/specific-overrides.json#L8-L14) |
| — | 구분선 | | | | I-351 |
| I-352 | Save | `workspace::Save` | Ctrl+S | | [app_menus.rs:137](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L137) · [keymap:656](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L656) |
| I-353 | Save As… | `workspace::SaveAs` | Ctrl+Shift+S | | [app_menus.rs:138](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L138) · [keymap:658](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L658) |
| I-354 | Save All | `workspace::SaveAll` | Ctrl+K S | | [app_menus.rs:139](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L139) · [keymap:704](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L704) |
| — | 구분선 | | | | I-355 |
| I-356 | Close Editor | `pane::CloseActiveItem` (`close_pinned = true`) | 같은 인자의 키 없음 (Ctrl+W·Ctrl+F4는 `close_pinned = false`) | **메뉴로 닫으면 핀 탭도 닫힌다.** 단축키 칸이 비어 있을 수 있다 (I-467) | [app_menus.rs:141-147](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L141-L147) · [keymap:507-508](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L507-L508) |
| I-358 | Close Window | `workspace::CloseWindow` | Ctrl+Shift+W | Alt+F4는 OS 키 | [app_menus.rs:149](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L149) · [keymap:25](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L25) |

**Edit**

| ID | 문구 | 액션 | 키 | 비고 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-359 | Undo | `editor::Undo` | Ctrl+Z | | [app_menus.rs:156](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L156) · [keymap:77](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L77) |
| I-360 | Redo | `editor::Redo` | Ctrl+Y / Ctrl+Shift+Z | | [app_menus.rs:157](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L157) · [keymap:78-79](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L78-L79) |
| — | 구분선 | | | | I-361 |
| I-362 | Cut | `editor::Cut` | Ctrl+X / Shift+Delete | | [app_menus.rs:159](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L159) · [keymap:71-72](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L71-L72) |
| I-363 | Copy | `editor::Copy` | Ctrl+C / Ctrl+Insert | | [app_menus.rs:160](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L160) · [keymap:73-74](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L73-L74) |
| I-364 | Copy and Trim | `editor::CopyAndTrim` | 없음 | | [app_menus.rs:161](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L161) |
| I-365 | Paste | `editor::Paste` | Ctrl+V / Shift+Insert | | [app_menus.rs:162](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L162) · [keymap:75-76](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L75-L76) |
| — | 구분선 | | | | I-366 |
| I-367 | Find | `buffer_search::Deploy` | Ctrl+F | 바꾸기도 같은 검색 창의 Replace로 들어간다 | [app_menus.rs:164](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L164) · [keymap:137](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L137) |
| I-479 † | Replace † | `buffer_search::DeployReplace` | Ctrl+H | 원본 메뉴에는 Find만 있다 | [keymap:138](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L138) |
| — | 구분선 | | | | I-369 |
| I-370 | Toggle Line Comment | `editor::ToggleComments` (메뉴 인수: `comment_empty_lines = true`) | **기본→VSCode:** Ctrl+/ · Ctrl+K Ctrl+C **표시가 사라진다** (추정) | VSCode가 두 키를 `comment_empty_lines = false`로 덮어써 메뉴의 인수와 달라졌고, 인수가 같은 유효 바인딩이 없다 (I-826). 키 입력의 주석 토글은 그대로 동작한다. 메뉴 인수도 false로 바꾸면 키를 표시할 수 있으나 원본 메뉴 동작과 다른 앱 정책이다 (판단). 기본값: 아래 줄로 이동 안 함, 들여쓰기 무시 안 함 (I-469) | [app_menus.rs:167-170](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L167-L170) · [actions.rs:144-166](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/actions.rs#L144-L166) · [vscode.json:16-17](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L16-L17) · [keymap:562-563](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L562-L563) |

**Selection**

| ID | 문구 | 액션 | 키 | 비고 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-371 | Select All | `editor::SelectAll` | Ctrl+A | | [app_menus.rs:177-181](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L177-L181) · [keymap:106](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L106) |
| I-372 | Expand Selection | `editor::SelectLargerSyntaxNode` | Shift+Alt+Right | 구문 트리(Tree-sitter) 기반 (사용자 결정으로 포함) | [app_menus.rs:182](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L182) · [keymap:551](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L551) |
| I-373 | Shrink Selection | `editor::SelectSmallerSyntaxNode` | Shift+Alt+Left | 위와 같음 | [app_menus.rs:183](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L183) · [keymap:552](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L552) |
| I-374 | Select Next Sibling | `editor::SelectNextSyntaxNode` | 없음 | 위와 같음 | [app_menus.rs:184](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L184) |
| I-375 | Select Previous Sibling | `editor::SelectPreviousSyntaxNode` | 없음 | 위와 같음 | [app_menus.rs:185-188](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L185-L188) |
| — | 구분선 | | | | I-376 |
| I-377 | Add Cursor Above | `editor::AddSelectionAbove` | Ctrl+Alt+Up | `skip_soft_wrap = true` | [app_menus.rs:190-195](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L190-L195) · [keymap:544](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L544) |
| I-378 | Add Cursor Below | `editor::AddSelectionBelow` | Ctrl+Alt+Down | `skip_soft_wrap = true` | [app_menus.rs:196-201](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L196-L201) · [keymap:545](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L545) |
| I-379 | Select Next Occurrence | `editor::SelectNext` | Ctrl+D / Ctrl+F3 | `replace_newest = false` | [app_menus.rs:202-207](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L202-L207) · [keymap:555-557](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L555-L557) |
| I-380 | Select Previous Occurrence | `editor::SelectPrevious` | Ctrl+Shift+F3 | `replace_newest = false` | [app_menus.rs:208-213](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L208-L213) · [keymap:558](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L558) |
| I-381 | Select All Occurrences | `editor::SelectAllMatches` | Ctrl+Shift+L / Ctrl+F2 | | [app_menus.rs:214](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L214) · [keymap:553-554](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L553-L554) |
| — | 구분선 | | | | I-382 |
| I-383 | Move Line Up | `editor::MoveLineUp` | Alt+Up | | [app_menus.rs:216](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L216) · [keymap:547](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L547) |
| I-384 | Move Line Down | `editor::MoveLineDown` | Alt+Down | | [app_menus.rs:217](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L217) · [keymap:548](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L548) |
| I-385 | Duplicate Selection | `editor::DuplicateLineDown` | Shift+Alt+Down | | [app_menus.rs:218](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L218) · [keymap:550](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L550) |

**View**

| ID | 문구 | 액션 | 키 | 비고 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-386 | Zoom In | `zed::IncreaseBufferFontSize` (`persist = false`) | Ctrl+= / Ctrl+Shift+= | 글꼴 크기를 일시적으로 키운다. 창 Zoom(Window 메뉴)과 다르다 (I-471) | [app_menus.rs:12-13](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L12-L13) · [keymap:29-30](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L29-L30) |
| I-387 | Zoom Out | `zed::DecreaseBufferFontSize` (`persist = false`) | Ctrl+- | 위와 같음 | [app_menus.rs:16-17](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L16-L17) · [keymap:31](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L31) |
| I-388 | Reset Zoom | `zed::ResetBufferFontSize` (`persist = false`) | Ctrl+0 | 위와 같음 | [app_menus.rs:20-21](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L20-L21) · [keymap:32](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L32) |
| I-389 | Reset All Zoom | `zed::ResetAllZoom` (`persist = false`) | 없음 | 위와 같음 | [app_menus.rs:24-25](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L24-L25) |
| — | 구분선 | | | | I-390 |
| I-391 | Toggle Left Dock | `workspace::ToggleLeftDock` | Ctrl+B | | [app_menus.rs:28](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L28) · [keymap:673](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L673) |
| I-394 | Toggle All Docks | `workspace::ToggleAllDocks` | Ctrl+Shift+Y | Dock들을 한꺼번에 토글한다. 바깥 사이드바는 포함하지 않는다 (사용자 결정으로 포함) | [app_menus.rs:31](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L31) · [keymap:677](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L677) |
| I-472 † | Toggle Folder Sidebar † | `multi_workspace::ToggleWorkspaceSidebar` (우리 폴더 사이드바 토글에 대응) | Ctrl+Alt+J | 원본 메뉴에 없다 | [keymap:675](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L675) |
| I-473 † | Focus Folder Sidebar † | `multi_workspace::FocusWorkspaceSidebar` 대응 | Ctrl+Alt+; | 원본 메뉴에 없다 | [keymap:676](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L676) |
| I-396 | Split Up | `pane::SplitUp` | Ctrl+K Up | Zed의 Editor Layout 묶음을 평탄화. 분할 네 방향 키는 덧씌움 전후가 같다. 탭 바 분할 팝업의 `MovePane` 분기(분할 불가 항목)는 인수가 달라 이 키를 붙이지 않는다 (I-833) | [app_menus.rs:36](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L36) · [keymap:817](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L817) |
| I-397 | Split Down | `pane::SplitDown` | Ctrl+K Down | 위와 같음 | [app_menus.rs:37](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L37) · [keymap:818](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L818) |
| I-398 | Split Left | `pane::SplitLeft` | Ctrl+K Left | 위와 같음 | [app_menus.rs:38](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L38) · [keymap:819](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L819) |
| I-399 | Split Right | `pane::SplitRight` | Ctrl+K Right / Ctrl+\ | 위와 같음 | [app_menus.rs:39](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L39) · [keymap:596](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L596) · [keymap:820](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L820) |
| I-476 † | Maximize Pane † | `workspace::ToggleZoom` | Shift+Escape | 원본 메뉴에 없다. 창 Zoom·글꼴 Zoom과 구별 | [keymap:26](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L26) |
| — | 구분선 | | | | I-400 |
| I-401 | Project Panel | `project_panel::ToggleFocus` | Ctrl+Shift+E | | [app_menus.rs:43](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L43) · [keymap:699](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L699) |
| I-474 † | Open Markdown Preview † | `markdown::OpenPreview` | Ctrl+Shift+V (확장자 md인 편집기) | 원본 메뉴에 없다 | [keymap:606](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L606) |
| I-475 † | Open Markdown Preview to the Side † | `markdown::OpenPreviewToTheSide` | Ctrl+K V (확장자 md인 편집기) | 원본 메뉴에 없다 | [keymap:605](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L605) |

**Go**

| ID | 문구 | 액션 | 키 | 비고 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-413 | Back | `pane::GoBack` | Alt+Left / Alt+- | 탐색 이력 | [app_menus.rs:230](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L230) · [keymap:516-518](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L516-L518) |
| I-414 | Forward | `pane::GoForward` | Alt+Right / Alt+= | 탐색 이력 | [app_menus.rs:231](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L231) · [keymap:519-521](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L519-L521) |
| I-477 † | Previous Tab † | `pane::ActivatePreviousItem` | Ctrl+PageUp | 원본 메뉴에 없다 | [keymap:503](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L503) |
| I-478 † | Next Tab † | `pane::ActivateNextItem` | Ctrl+PageDown | 원본 메뉴에 없다 | [keymap:504](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L504) |
| — | 구분선 | | | | I-415 |
| I-416 | Command Palette... | `command_palette::Toggle` | Ctrl+Shift+P / F1 | 우리가 채택한 명령만 등록 | [app_menus.rs:233](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L233) · [keymap:696-697](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L696-L697) |
| — | 구분선 | | | | I-417 |
| I-418 | Go to File... | `file_finder::Toggle` | Ctrl+P / Ctrl+E | | [app_menus.rs:235](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L235) · [keymap:692](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L692) · [keymap:695](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L695) |
| I-420 | Go to Line/Column... | `go_to_line::Toggle` | Ctrl+G (편집기) | | [app_menus.rs:241](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L241) · [keymap:630](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L630) |

**Window**

| ID | 문구 | 액션 | 키 | 비고 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-445 | Minimize | `zed::Minimize` | 없음 | 창 최소화 | [app_menus.rs:298](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L298) |
| I-446 | Zoom | `zed::Zoom` | 없음 (F11은 전체 화면으로 다른 액션) | 창 확대(최대화). View의 글꼴 Zoom과 별개 | [app_menus.rs:299](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L299) · [zed.rs:1025-1032](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1025-L1032) |

**Help**

| ID | 문구 | 액션 | 키 | 비고 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-450 | View Dependency Licenses | `zed::OpenLicenses` | 없음 | 우리 앱 의존성 고지(03-THEME-ICONS의 라이선스 고지 목록)로 구현을 대체 | [app_menus.rs:312](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L312) |

### 9.3 구현 합의점

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| I-480 | 초기 포커스·실행 | 메뉴를 열 때 활성 편집기·트리의 포커스를 저장하고, 항목을 실행할 때 복원한 뒤 명령을 처리한다. 항목의 비활성 계산은 우리 앱 기능 가용성에 맞춘 **우리 설계**다 (Zed의 자동 동작이라고 쓰지 않는다) | [application_menu.rs:132-137](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L132-L137) · [context_menu.rs:706-725](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L706-L725) |
| I-481 | † 항목이 필요한 이유 | 최소안에서 마크다운·바깥 사이드바·탭 이동·바꾸기·최대화를 빼면 요구 기능의 진입점이 없어진다. 그러나 Zed의 메뉴에 있던 항목이라고 문서화하면 사실 오류다 (판단) | [app_menus.rs:69-340](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L69-L340) · [keymap:600-607](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L600-L607) · [keymap:672-676](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L672-L676) · [keymap:503-504](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L503-L504) · [keymap:130-137](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L130-L137) |
| I-483 | 영역 토글 관계 | **Toggle Left Dock**과 **Project Panel**은 표시 토글·포커스 토글로 따로 둔다 (I-098). **Toggle All Docks**(I-394)는 Dock만 토글하고 바깥 사이드바는 포함하지 않는다 (사이드바는 별개 바깥 영역) | [workspace.rs:4805-4818](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L4805-L4818) · [keymap:672-699](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L672-L699) |
| I-465 | 액션 이름 | 키맵의 `pane::…` 이름과 메뉴 코드의 `workspace::…` 이름(GoBack, SplitRight, CloseActiveItem 등)은 같은 액션 계열이다 | [workspace.rs:83-86](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L83-L86) · [pane.rs:110-119](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L110-L119) |

### 9.4 제외한 항목 (사용자 결정 반영)

사용자 결정(2026-10-31, `_decisions.md`)에 따라 포함·제외를 확정했다. **포함**으로 정해진 항목(About, 설정 파일 열기, 키맵 편집 화면·키맵 파일 열기, Open Recent, New Window, Open File, Toggle All Docks, 구문 단위 선택, 파일 안 찾기·바꾸기, 명령 팔레트, Go to File)은 9.2의 최종 구조에 넣었다. 아래는 최종 구조에서 **뺀** ☰ 항목과 이유다. 결정 목록에 없는 항목은 "포함 목록에 없음"으로 제외했다.

| ID | 원본 문구 | 원래 위치 | 이유 |
| --- | --- | --- | --- |
| I-323 | Check for Updates | 앱 이름 | 제외 목록 (Check for Updates) |
| I-326 | Open Settings | 앱 이름 (Settings 묶음) | 제외 목록 (Settings UI 화면). 같은 자리에 "설정 파일 열기"(I-327)를 둔다 |
| I-328 | Open Project Settings | 앱 이름 (Settings 묶음) | 포함 목록에 없음 (폴더별 설정 없음) |
| I-329 | Open Project Settings File | 앱 이름 (Settings 묶음) | 포함 목록에 없음 |
| I-330 | Open Default Settings | 앱 이름 (Settings 묶음) | 포함 목록에 없음 |
| I-334 | Open Default Key Bindings | 앱 이름 (Settings 묶음) | 포함 목록에 없음 |
| I-357 | Close Project | File | 포함 목록에 없음. 폴더 등록 해제는 사이드바·루트 우클릭 메뉴의 Remove(11절)가 맡는다 |
| I-368 | Find in Project | Edit | 제외 목록 (프로젝트 전체 검색) |
| I-451 | Show Welcome | Help | 제외 목록 (Welcome/Onboarding) |
| I-457 | Documentation | Help | 포함 목록에 없음 (Zed 문서 링크라서 우리 앱에 대응이 없다) |
| I-392, I-393 | Toggle Right Dock, Toggle Bottom Dock | View | 제외 목록 (오른쪽·하단 Dock) |
| I-348 | Open Remote… | File | 기존 제외 (원격) |

### 9.5 앱 메뉴에서 확신 낮은 부분

- **실행 화면:** 앱 메뉴를 실제 Windows 창에서 열어 보지 않았다. 고정 커밋의 코드와 기본값만 근거다. 설치본 버전·`title_bar.show_menus`·접근성 설정에 따라 가로 메뉴가 항상 펼쳐져 보일 수 있다 (I-484).
- **오른쪽 단축키:** 표의 키는 후보 목록이고 화면에는 컨텍스트에 맞는 하나만 보인다. Close Editor, 진단, 최근 프로젝트의 인자 일치는 특히 확인이 필요하다 (I-485).
- **메뉴 전체 크기:** 200px은 최소 폭이고 22.652px은 라벨 계산값이다. 글꼴 대체, 단축키, 스크롤바, DPI 반올림을 포함한 메뉴별 전체 크기는 재지 않았다 (I-486).
- **비활성 자동 갱신:** 비활성 표시 경로는 확인했지만 Undo·Save 등의 상태별 자동 비활성은 확인하지 못했다 (I-487).

## 10. ∨ 메뉴

사용자 요건: 타이틀 바 오른쪽의 Sign In 버튼은 빼고, 옆의 **∨ 드롭다운 메뉴는 채용**한다. 항목은 우리 앱에서 쓸 수 있는 것만 남긴다. 값은 ∨ 메뉴 조사(`zed-user-menu.md`)의 ID(I-600~I-655)를 그대로 쓴다. 버튼 크기·위치·팝업 모양은 [01-LAYOUT.md](01-LAYOUT.md) 4.1절(I-600~I-612)에 있다. 포함·제외는 사용자 결정(2026-10-31, `_decisions.md`)에 따라 확정했다 (10.5).

**ID 번호대:** 이 절의 I-600~I-655는 ∨ 메뉴 조사의 ID 그대로다. 우클릭 메뉴 조사와 겹치던 번호는 11절에서 I-900번대로 바꿨다.

### 10.1 버튼 동작

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| I-610 | 열기·닫기·포커스 | ∨를 누르면 메뉴가 열리고 버튼이 선택 상태가 된다. 열린 상태에서 ∨를 다시 누르면 닫히기만 한다. 닫을 때 포커스가 메뉴 안에 있었으면 열기 전 포커스로 복원한다 (01-LAYOUT I-610) | [popover_menu.rs:182-194](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L182-L194) · [popover_menu.rs:274-312](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L274-L312) · [popover_menu.rs:480-493](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L480-L493) |
| I-650 | 버튼 보존 | Sign In 자리만 제거한다. 일반 버튼 + `ChevronDown` 아이콘, 탭 순서, 접근성 이름 "User menu", hover·누름 색, 오른쪽 위 기준 팝업 정렬은 그대로 만든다. 사용자 메뉴라는 이름을 쓰더라도 실제 계정 기능은 만들지 않는다 (판단) | [title_bar.rs:1258-1267](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1258-L1267) · [title_bar.rs:1424-1425](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1424-L1425) · [button_like.rs:802-849](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L802-L849) |
| I-649 | ☰ 메뉴와의 중복 | ☰와 ∨에 설정·테마 진입점이 함께 있어도 **같은 명령과 같은 상태**를 쓴다. 테마 저장소나 키맵을 따로 만들지 않는다 (판단). 원본 ∨ 메뉴도 전역 액션을 쓴다 | [title_bar.rs:1374-1386](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1374-L1386) |
| I-636 | 단축키 표시 | 항목 오른쪽의 단축키는 메뉴의 라벨 오른쪽에 그리는 키 표시다. 메뉴에 별도 포커스 컨텍스트를 지정하지 않으므로 창 전체에서 그 액션의 우선순위 가장 높은 키 하나를 찾는다. 항목 설명 툴팁과는 별개다 | [context_menu.rs:2026-2044](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2026-L2044) · [context_menu.rs:400-401](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L400-L401) · [title_bar.rs:1280-1422](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1280-L1422) |

### 10.2 유지 항목

원본 ∨ 메뉴 전체에서 우리 앱에 남기는 것은 아래 다섯 항목이다. 다섯 항목은 원본에서도 연속이며 **서로 사이에 구분선이 없다.**

| ID | 문구 | 원본 조건 | 액션 | Windows 기본 키 | 구분선 | 비고 | 출처 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| I-620 | Settings | 항상 | `zed::OpenSettingsFile` (**settings.json을 탭으로 열기**) | Ctrl+Alt+, / Ctrl+, | 앞 구분선은 조직 그룹이 있을 때만(우리는 없음). 다음 Keymap과 선 없음 | Zed 원본은 `zed::OpenSettings`(설정 화면, Ctrl+,)다. 우리는 설정 화면을 만들지 않으므로(사용자 결정) 같은 자리에서 settings.json을 연다 (I-647). Ctrl+,도 같은 동작에 연결한다 (판단) | [title_bar.rs:1374](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1374) · [keymap:33](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L33) · [keymap:1536](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1536) |
| I-621 | Keymap | 항상 | `zed::OpenKeymap` (**키맵 편집 화면**) | Ctrl+K Ctrl+S | 선 없음 | 사용자 결정으로 키맵 편집 화면(Zed Keymap Editor와 같게)을 만든다. 화면 명세는 15절 | [title_bar.rs:1375](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1375) · [keymap:683-689](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L683-L689) |
| I-622 | Themes… | 항상 | `theme_selector::Toggle` | Ctrl+K Ctrl+T | 선 없음 | 문구의 `…`는 **한 글자 말줄임표**다 (점 세 개가 아니다). 목록은 One Dark·One Light·Catppuccin Latte/Frappé/Macchiato/Mocha로 제한 (I-648) | [title_bar.rs:1376-1379](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1376-L1379) · [keymap:688](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688) |
| I-623 | Icon Themes… | 항상 | `icon_theme_selector::Toggle` | 없음 | 선 없음 | 목록은 Zed 기본·Catppuccin Icons로 제한 (I-648). 임의 단축키를 덧붙이지 않는다 (I-633) | [title_bar.rs:1380-1383](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1380-L1383) |
| I-624 | Extensions | 항상 | `zed::Extensions` | Ctrl+Shift+X | 선 없음 (원본은 뒤에 AI 조건부 구분선 I-625가 있으나 AI 그룹이 없어 만들지 않는다) | **확장 관리 화면** (16절, 사용자 지시로 포함) | [title_bar.rs:1384-1387](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1384-L1387) · [keymap:717](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L717) |

보충 사실:

| ID | 항목 | 내용 | 출처 |
| --- | --- | --- | --- |
| I-632 | 키 근거 | Settings의 Ctrl+,는 "설정 창이 아닐 때" 컨텍스트 키이고, Keymap의 Ctrl+K Ctrl+S와 Themes…의 Ctrl+K Ctrl+T는 작업 영역 컨텍스트 키다. 사용자 키맵과 현재 컨텍스트에 따라 실제 표시 키가 달라질 수 있다 | [keymap:1530-1537](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1530-L1537) · [keymap:683-689](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L683-L689) · [keybinding.rs:188-204](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L188-L204) |
| I-633 | 키 없는 항목 | `icon_theme_selector::Toggle`은 Windows 기본 키맵에 바인딩이 없다. 버튼 이름 "User menu"에서 키를 추론하지 않는다 | [keymap:717](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L717) |
| I-635 | 이 메뉴에 없는 것 | 글꼴 크기·글꼴 선택·원격·채널·피드백·협업 초대는 원본 ∨ 메뉴 항목이 아니다. Sign In도 이 메뉴 안의 항목이 아니라 별도 버튼이다 | [title_bar.rs:1184-1201](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1184-L1201) · [title_bar.rs:1280-1422](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1280-L1422) |
| I-648 | 선택기 내용 제한 | Themes… 선택기는 One Dark·One Light·Catppuccin Latte/Frappé/Macchiato/Mocha 6종, Icon Themes… 선택기는 Zed 기본·Catppuccin Icons로 제한한다. 시스템 라이트·다크 설정은 기준 화면 값을 따른다. 항목 문구는 원본, 선택기 내용 제한은 사용자 요건이다 | [title_bar.rs:1376-1383](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1376-L1383) |
| I-647 | Settings·Keymap 범위 | Settings = settings.json 열기, Keymap = 키맵 편집 화면 (사용자 결정). 별도 설정 화면은 없다. 설정 파일에 AI·Git·LSP·터미널 항목을 두지 않는다 (설계 판단) | [title_bar.rs:1374-1375](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1374-L1375) |

### 10.3 최종 메뉴 구조

계정·업데이트·패널 배치·로그아웃 그룹이 모두 없어졌으므로 그 앞뒤 구분선도 함께 없앴다. 빈 구분선, 맨 앞 구분선, 끝 구분선은 남기지 않는다. 구분선은 하나도 없다.

```text
타이틀 바 오른쪽: [∨] [최소화] [최대화/복원] [닫기]
                  │
                  ├─ Settings       Ctrl+,
                  ├─ Keymap         Ctrl+K Ctrl+S
                  ├─ Themes…        Ctrl+K Ctrl+T
                  ├─ Icon Themes…
                  └─ Extensions     Ctrl+Shift+X

구분선: 없음
Sign In / Signing in… / 아바타 / 조직 이름 / 업데이트 점: 없음
∨ hover 툴팁: 없음 (접근성 이름 "User menu"는 유지)
```

∨와 창 버튼 사이의 정확한 가로 간격은 기준 화면 실측이 필요하다. 확인된 것은 ∨가 오른쪽 묶음에서 창 버튼 바로 앞에 오고 묶음 gap이 4px이라는 것이다 (01-LAYOUT I-603). 키 표시는 8.4절의 형식을 따른다 (위 그림은 읽기 쉬운 조합 표기).

### 10.4 툴팁

툴팁 일반 규칙은 8절에 있다. ∨ 메뉴 쪽은 **툴팁이 거의 없다.**

| ID | 대상 | 문구 / 유무 | 구현 지침 | 출처 |
| --- | --- | --- | --- | --- |
| I-637 | ∨ 버튼 | **hover 툴팁 없음.** 접근성 이름(aria-label)만 "User menu" | 접근성 이름을 마우스 hover 툴팁으로 복제하면 Zed와 달라진다. 이 버튼은 툴팁을 다는 경로가 아니라 단순 버튼 경로를 쓴다 | [title_bar.rs:1258-1267](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1258-L1267) · [button_like.rs:532-540](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L532-L540) · [popover_menu.rs:182-214](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L182-L214) |
| I-639 | Settings | 항목 라벨 "Settings". hover 툴팁 없음 | Ctrl+,는 오른쪽 단축키 표시이지 툴팁이 아니다 | [title_bar.rs:1374](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1374) · [keymap:1530-1537](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1530-L1537) |
| I-640 | Keymap | 라벨 "Keymap". hover 툴팁 없음 | Ctrl+K Ctrl+S 표시와 툴팁 문구를 분리한다 | [title_bar.rs:1375](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1375) · [keymap:687](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L687) |
| I-641 | Themes… | 라벨 "Themes…". hover 툴팁 없음 | 한 글자 말줄임표. 키 Ctrl+K Ctrl+T | [title_bar.rs:1376-1379](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1376-L1379) · [keymap:688](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688) |
| I-642 | Icon Themes… | 라벨 "Icon Themes…". hover 툴팁 없음 | 기본 키 없음. 툴팁에 임의 단축키를 덧붙이지 않는다 | [title_bar.rs:1380-1383](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1380-L1383) |
| I-643 | Extensions | 라벨 "Extensions". hover 툴팁 없음 | 오른쪽에 Ctrl+Shift+X 표시. 툴팁에 임의 문구를 붙이지 않는다 | [title_bar.rs:1384-1387](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1384-L1387) |
| I-646 | 공통 위젯의 예외 | 메뉴 위젯에는 항목 오른쪽 아이콘에 단축키 툴팁을 다는 경로가 있다. 그러나 이 메뉴는 그 아이콘·제목·핸들러를 넣지 않으므로 해당 없음 | 위젯 어딘가에 툴팁 구현이 있다고 해서 모든 메뉴 행에 툴팁이 붙는 것이 아니다 | [context_menu.rs:2054-2083](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2054-L2083) · [title_bar.rs:1280-1422](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1280-L1422) |

### 10.5 결정 반영

| ID | 원본 문구 | 결정 | 출처 |
| --- | --- | --- | --- |
| — | (참조 I-620) Settings | **settings.json을 탭으로 열어 편집한다.** 설정 화면(Settings UI)은 만들지 않는다. ☰ 앱 이름 메뉴의 "Open Settings File"(I-327)과 같은 명령이다 | `_decisions.md` |
| — | (참조 I-621) Keymap | **키맵 편집 화면**(Zed Keymap Editor와 같게)을 연다. 화면 명세는 15절. ☰의 "Open Keymap"(I-332)과 같은 명령이다 | `_decisions.md` |
| — | (참조 I-627) Classic (Panel Layout 하위) | **제외** (Panel Layout 전체 제외). I-626, I-628, I-629도 제외 | `_decisions.md` |

### 10.6 제외한 항목 (참고)

원본 ∨ 메뉴에서 우리가 넣지 않은 항목이다. 이유는 계정·업데이트·에이전트 때문이다. 우리 앱에서는 만들지 않는다.

| ID | 원본 항목 | 제외 이유 |
| --- | --- | --- |
| I-613, I-614 | 사용자 이름, 그 뒤 구분선 | 계정 |
| I-615, I-616 | Restart to update Zed, 그 뒤 구분선 | 업데이트 |
| I-617, I-618, I-619 | Organization 머리글, 조직 목록, 그 뒤 구분선 | 계정·조직 |
| I-625 | Extensions 뒤 구분선 | AI 그룹이 없어 만들지 않는다 |
| I-626, I-628, I-629 | Panel Layout, Agentic, Custom | 에이전트 레이아웃·레이아웃 상태 행 |
| I-630, I-631 | 구분선, Sign Out | 계정 |

### 10.7 ∨ 메뉴에서 확신 낮은 부분

- **실행 화면:** ∨를 실제 Windows 창에서 열어 보지 않았다. 기준 화면은 메뉴가 닫힌 장면이다. 로그인·업데이트 조건부 메뉴를 시각 대조하지 않았다 (I-651).
- **버튼 크기:** 22×22px는 UI 16px·밀도 Default·`ChevronDown` 단독 구성의 소스 계산값이다. 부모 rem, DPI 반올림, 창 버튼과의 간격, 히트테스트는 실측하지 않았다 (I-652).
- **툴팁 없음의 범위:** ∨ 버튼과 일반 항목에 툴팁 지정이 없음을 확인했다. 스크린 리더의 접근성 이름 읽기와 제외된 조직 항목 안쪽 툴팁은 hover 툴팁과 별개이며 후자는 미확인이다 (I-653).
- **표시 단축키:** 사용자 재정의와 현재 포커스에 따라 달라질 수 있다. 표는 Windows 기본 키맵의 유효 후보다 (I-654).

## 11. 우클릭 메뉴

> 표기는 메뉴·툴팁 표시 형식(8.4절, `Ctrl-N`)을 따른다. 키는 적용 keymap(VSCode 덧씌움 후) 기준이며, 이 절의 항목 중 덧씌움이 바꾸는 키는 없다 (1.4절 변경 목록과 대조, I-832).

사용자 요건: 아이콘과 각 영역의 우클릭 메뉴도 Zed와 같게 한다. 쓸 수 없는 항목은 뺀다. 값은 우클릭 메뉴 조사(`zed-context-menus.md`)의 ID를 쓰되, ∨ 메뉴 조사의 I-600~I-655와 겹치던 ID는 **I-900번대로 다시 매겼다** (옛 ID + 300, 대응표 `id-renumber.md`). 크기·여백·색·열리는 위치는 [01-LAYOUT.md](01-LAYOUT.md) 4.9절(I-501~I-515)에 있다. 이 절은 영역별 **최종 구조**(구분선 위치는 Zed 그대로)와 열기·닫기·키보드 조작을 적는다. 포함·제외는 사용자 결정(2026-10-31, `_decisions.md`)에 따라 확정했고, 뺀 항목은 11.9에 이유와 함께 적었다.

**ID 번호대:** 이 절의 I-9xx(편집기 I-900~I-917, 미리보기 I-920~I-923, 사이드바 I-930~I-940, 기타 I-945~I-952)는 우클릭 메뉴 조사의 옛 I-6xx를 +300 한 것이다. Move Up·Move Down은 I-934·I-940으로 나눴다. 10절의 I-600~I-655는 ∨ 메뉴 조사의 ID 그대로다.

규칙: 구분선은 그 위아래에 항목이 남을 때만 둔다. 가운데 그룹이 모두 빠져 구분선이 연속되면 하나로 합친다. 앞쪽·끝 구분선은 만들지 않는다 (Zed도 조건부 구분선은 해당 그룹이 있을 때만 넣는다).

### 11.1 열기·닫기·키보드 (공통)

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| I-500 | 항목 종류 | 일반 액션 항목, 직접 핸들러 항목, 비활성 항목, 토글 항목(체크 아이콘), 구분선, 헤더, 하위 메뉴, 직접 그린 항목 | [context_menu.rs:497-860](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L497-L860) |
| I-516 | 키 컨텍스트 | 메뉴의 키 컨텍스트 이름은 `menu` | [context_menu.rs:357](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L357) |
| I-517 | 키 (Windows) | 다음: Down, Tab, Ctrl-N / 이전: Up, Shift-Tab, Ctrl-P / 처음: Home, PageUp, Shift-PageUp / 끝: End, PageDown, Shift-PageDown / 실행: **Enter** / 닫기: **Escape**, Ctrl-C / 하위 메뉴 열기: Right / 상위로: Left | [keymap:6-21](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L6-L21) · [keymap:43-55](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L43-L55) |
| I-518 | 처음 선택·순환 | 키보드로 처음 아래 키를 누르면 선택 가능한 첫 항목을 고른다. 끝에서 다음으로 가면 처음으로 순환한다. 구분선·비활성 항목은 건너뛴다 | [context_menu.rs:1098-1135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1098-L1135) |
| I-519 | 닫히는 조건 | 메뉴 밖을 마우스로 누를 때, Escape, 항목 실행 후, 메뉴가 포커스를 잃을 때 (하위 메뉴로 포커스가 간 경우는 제외) | [context_menu.rs:2315-2345](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2315-L2345) · [context_menu.rs:899-947](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L899-L947) |
| I-520 | 단축키로 실행 | 메뉴가 열린 채로 항목의 단축키를 누르면 그 항목을 **선택 표시한 뒤 50ms 후** 메뉴를 닫고 액션을 실행한다 | [context_menu.rs:1350-1385](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1350-L1385) |
| I-521 | 하위 메뉴 | hover 또는 Right/Enter로 열린다. 하위 메뉴가 열리는 동안 부모 메뉴가 150~200ms 동안 포커스 잃음을 무시해 닫히지 않는다. (유지하는 우클릭 메뉴에는 하위 메뉴가 없다) | [context_menu.rs:1015-1020](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1015-L1020) · [context_menu.rs:1338](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1338) |
| — | 열려 있는 동안 툴팁 | 메뉴를 여는 버튼에는 툴팁이 없다 (L-205, L-272) | 8.1절 |
| — | 실행 | 항목을 실행하기 전에 원래 포커스를 복원한다 (9.1절 I-309와 같은 규칙) | 9.1절 |

### 11.2 프로젝트 패널

단축키는 프로젝트 패널 컨텍스트 기준이다 ([keymap:1009-1047](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1009-L1047)). 메뉴를 열면 메뉴의 포커스 컨텍스트가 프로젝트 패널로 지정된다. 출처: [project_panel.rs:1128-1312](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1128-L1312).

**공통 규칙**

| ID | 내용 | 출처 |
| --- | --- | --- |
| I-560 | 우클릭한 항목이 선택이 되고, 메뉴는 선택 항목에 대해 동작한다 | [project_panel.rs:1143-1146](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1143-L1146) |
| I-559 | **빈 공간**(마지막 항목 아래)을 우클릭하면 **마지막 루트 항목**을 우클릭한 것과 같은 메뉴가 나온다 | [project_panel.rs:8076-8090](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8076-L8090) |
| I-561 | 접힌 경로(`docs \ guides`)의 조각을 우클릭하면, 그 조각을 활성 단계로 바꾼 뒤 메뉴를 연다 (어느 폴더에 대한 메뉴인지가 조각에 따라 달라진다) | [project_panel.rs:6979-6990](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6979-L6990) |
| I-562 | 여러 항목을 표시한 상태에서도 메뉴 구성은 같고, Cut·Copy·Trash·Delete는 표시된 모든 항목에 적용된다 (추정) | [project_panel.rs:150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L150) |
| I-558 | 읽기 전용 프로젝트에서는 메뉴가 Open Markdown Preview(md)뿐으로 줄어든다. 우리 앱에는 읽기 전용 폴더가 없다 (판단) | [project_panel.rs:1189-1200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1189-L1200) |
| I-502 | 키보드(Menu 키, Shift+F10)로 열면 선택 항목 옆 (2, 2)px (01-LAYOUT 4.9절) | [project_panel.rs:1317-1336](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1317-L1336) |

**I-660 파일**

```text
New File                     Ctrl-N
New Folder                   Alt-N
──────────────
Reveal in File Explorer      Ctrl-Alt-R
Open in Default App          Ctrl-Shift-Enter
Open Markdown Preview                       (마크다운 파일만)
──────────────
Cut                          Ctrl-X
Copy                         Ctrl-C
Duplicate
Paste                        Ctrl-V         (붙여 넣을 것이 없으면 비활성)
──────────────
Copy Path                    Alt-Shift-C
Copy Relative Path           Ctrl-K Ctrl-Shift-C
──────────────
Rename                       F2
Trash                        Delete
Delete                       Ctrl-Delete
```

**I-661 폴더 (루트 아님)**

```text
New File                     Ctrl-N
New Folder                   Alt-N
──────────────
Reveal in File Explorer      Ctrl-Alt-R
Open in Default App          Ctrl-Shift-Enter
──────────────
Unfold Directory                            (접힌 경로일 때)
Fold Directory                              (접을 수 있을 때)
──────────────
Cut / Copy / Duplicate / Paste              (파일과 같음)
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

Zed에서는 Unfold·Fold 앞 구분선 바로 뒤에 "Find in Folder…"(프로젝트 검색)가 먼저 온다. 우리 앱은 검색을 빼므로 구분선 뒤에 Unfold·Fold만 남는다.

**I-662 루트 폴더 (사이드바에서 선택한 등록 폴더). 빈 공간 우클릭도 같다**

```text
New File                     Ctrl-N
New Folder                   Alt-N
──────────────
Reveal in File Explorer      Ctrl-Alt-R
Open in Default App          Ctrl-Shift-Enter
──────────────
Unfold Directory / Fold Directory           (해당할 때)
──────────────
Cut / Copy / Duplicate / Paste
──────────────
Copy Path                    Alt-Shift-C
Copy Relative Path           Ctrl-K Ctrl-Shift-C
──────────────
Add Folders to Project…
Remove from Project
──────────────
Expand All                   Ctrl-Right
Collapse All                 Ctrl-Left
```

- 루트에는 Rename·Trash·Delete가 **없다** (Windows에서 루트 이름 변경은 막혀 있고 Trash·Delete는 루트가 아닐 때만 나온다, I-551~I-553).
- **Add Folders to Project…**는 우리 앱에서 **폴더 등록**(사이드바 폴더 추가 버튼과 같은 동작)이다. 단축키 없음.
- **Remove from Project**는 우리 앱에서 **등록 해제**(사이드바 Remove와 같은 동작)다. **디스크의 폴더·파일은 지우지 않는다** (I-107, I-093). 단축키 없음.
- 두 항목은 루트 + 협업 아님 조건이다. 우리 앱에는 협업이 없으므로 항상 나온다.

**항목표 (파일·폴더·루트)**

| ID | 문구 | 액션 | 단축키 | 표시 조건 | 구분선 관계 |
| --- | --- | --- | --- | --- | --- |
| I-530 | New File | `project_panel::NewFile` | Ctrl-N | 항상 | 다음 줄 New Folder와 연속, 그 뒤 구분선 |
| I-531 | New Folder | `project_panel::NewDirectory` | Alt-N | 항상 | |
| I-532 | Reveal in File Explorer | `project_panel::RevealInFileManager` | Ctrl-Alt-R | 로컬 폴더 | 앞 구분선 |
| I-533 | Open in Default App | `project_panel::OpenWithSystem` | Ctrl-Shift-Enter | 로컬 폴더 | |
| I-535 | Open Markdown Preview | `project_panel::OpenMarkdownPreview` | 없음 | 마크다운 파일 | 파일 메뉴에서만 |
| I-538 | Unfold Directory | `project_panel::UnfoldDirectory` | 없음 | 경로 접기가 켜져 있고 접힌 경로일 때 | 폴더 그룹 앞 구분선 (폴더일 때만) |
| I-539 | Fold Directory | `project_panel::FoldDirectory` | 없음 | 경로 접기가 켜져 있고 접을 수 있을 때 | |
| I-541 | Cut | `project_panel::Cut` | Ctrl-X | 항상 | 앞 구분선 |
| I-542 | Copy | `project_panel::Copy` | Ctrl-C | 항상 | |
| I-543 | Duplicate | `project_panel::Duplicate` | 없음 | 항상 | |
| I-544 | Paste | `project_panel::Paste` | Ctrl-V | 붙여 넣을 것이 없으면 **비활성** | |
| I-548 | Copy Path | `workspace::CopyPath` | Alt-Shift-C | 항상 | 앞 구분선 |
| I-549 | Copy Relative Path | `workspace::CopyRelativePath` | Ctrl-K Ctrl-Shift-C | 항상 | |
| I-551 | Rename | `project_panel::Rename` | F2 | **Windows에서는 루트면 숨김** | 앞 구분선 |
| I-552 | Trash | `project_panel::Trash` (확인 있음) | Delete | 루트가 아닐 때 | 휴지통으로 보냄 |
| I-553 | Delete | `project_panel::Delete` (확인 있음) | Ctrl-Delete | 루트가 아닐 때 | 영구 삭제 |
| I-554 | Add Folders to Project… | `workspace::AddFolderToProject` | 없음 | 루트 + 협업 아님 | 앞 구분선 (루트일 때만). 우리 앱: 폴더 등록 |
| I-555 | Remove from Project | `project_panel::RemoveFromProject` | 없음 | 루트 + 협업 아님 | 위와 같음. 우리 앱: 등록 해제 (디스크는 그대로) |
| I-556 | Expand All | 루트: `ExpandAllEntries` / 하위 폴더: `ExpandSelectedEntryAndChildren` | 루트: Ctrl-Right / 하위 폴더: 없음 | 폴더 | 앞 구분선 |
| I-557 | Collapse All | 루트: `CollapseAllEntries` / 하위 폴더: `CollapseSelectedEntryAndChildren` | 루트: Ctrl-Left / 하위 폴더: 없음 | 폴더 | |

### 11.3 탭

출처: [pane.rs:3136-3500](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3136-L3500). 메뉴의 컨텍스트는 그 탭 아이템의 포커스라서 편집기 탭이면 편집기·Pane·작업 영역 컨텍스트 키가 보인다. 미리보기(기울임) 탭도 메뉴 구성은 같고 전용 항목이 없다 (I-588, [pane.rs:3136-3150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3136-L3150)).

**I-663 편집기 탭**

```text
Close
Close Others                 Ctrl-Alt-Shift-T   (탭이 하나뿐이면 비활성)
──────────────
Close Left                   Ctrl-K E           (왼쪽 탭이 없으면 비활성)
Close Right                  Ctrl-K T           (오른쪽 탭이 없으면 비활성)
──────────────
Close Clean                  Ctrl-K U           (깨끗한 탭이 없으면 비활성)
Close All                    Ctrl-K W
──────────────
Pin Tab / Unpin Tab          Ctrl-K Shift-Enter (고정 안 됨 → Pin Tab, 고정됨 → Unpin Tab)
──────────────
Copy Path                    Ctrl-K P
Copy Relative Path
──────────────
Reveal in File Explorer      Ctrl-K R
Reveal In Project Panel      Ctrl-Shift-E
──────────────
Open Markdown Preview        Ctrl-Shift-V       (마크다운 편집기 탭만)
```

Zed 원본에는 Pin Tab 아래에 "Make Tab Read-Only / Make Tab Editable"(I-578)이 있으나 포함 목록에 없어 뺐다 (11.9). 그래서 Pin Tab과 Copy Path 사이 구분선은 하나다.

**I-664 마크다운 미리보기 탭**

```text
Close
Close Others
──────────────
Close Left
Close Right
──────────────
Close Clean
Close All
──────────────
Pin Tab / Unpin Tab
──────────────
Show Source                  Ctrl-Shift-V
```

마크다운 미리보기 탭에 Copy Path 이하 파일 항목이 나오는지는 **미확인**이라서 넣지 않았다 (I-589).

| ID | 문구 | 액션 | 단축키 | 표시 조건 |
| --- | --- | --- | --- | --- |
| I-570 | Close | `pane::CloseActiveItem` (`close_pinned = true`) | 표시 안 될 수 있음 (키맵의 Ctrl-W는 `close_pinned = false`라 인자가 다르다) | 항상 |
| I-571 | Close Others | `pane::CloseOtherItems` | Ctrl-Alt-Shift-T | 탭이 하나뿐이면 비활성 |
| I-573 | Close Left | `pane::CloseItemsToTheLeft` | Ctrl-K E | 왼쪽 탭이 없으면 비활성 |
| I-574 | Close Right | `pane::CloseItemsToTheRight` | Ctrl-K T | 오른쪽 탭이 없으면 비활성 |
| I-575 | Close Clean | `pane::CloseCleanItems` | Ctrl-K U | 저장된(깨끗한) 탭이 없으면 비활성 |
| I-576 | Close All | `pane::CloseAllItems` | Ctrl-K W | 항상 |
| I-577 | Pin Tab / Unpin Tab | `pane::TogglePinTab` | Ctrl-K Shift-Enter | 고정 안 됨 → Pin Tab, 고정됨 → Unpin Tab (핀 탭 I-014, I-065와 같은 동작) |
| I-579 | Copy Path | `workspace::CopyPath` | Ctrl-K P | 절대 경로가 있을 때 |
| I-580 | Copy Relative Path | `workspace::CopyRelativePath` | 없음 | 루트 폴더가 있을 때 |
| I-582 | Reveal in File Explorer | `editor::RevealInFileManager` | Ctrl-K R | 로컬 + 경로 있음 |
| I-583 | Reveal In Project Panel | `pane::RevealInProjectPanel` | Ctrl-Shift-E | 프로젝트 패널에 보이는 파일 |
| I-585 | Open Markdown Preview | `markdown::OpenPreview` | Ctrl-Shift-V | 마크다운 편집기 |
| I-587 | Show Source | `markdown::CloseAndReturnToEditor` | Ctrl-Shift-V | 마크다운 미리보기 탭 |

`Close`의 `close_pinned = true`는 메뉴로 닫으면 핀 탭도 닫힌다는 뜻이다. 나머지 닫기 항목은 `close_pinned = false`로 핀 탭을 남긴다.

### 11.4 편집기

출처: [mouse_context_menu.rs:202-337](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/mouse_context_menu.rs#L202-L337). 전체 모드 편집기에 프로젝트가 있을 때만 나온다. 우클릭한 지점이 선택 영역 밖이면 먼저 커서를 그 위치로 옮긴다 (I-503).

**I-665 편집기 본문**

```text
Cut                          Ctrl-X
Copy                         Ctrl-C
Copy and Trim
Paste                        Ctrl-V
──────────────
Reveal in File Explorer      Ctrl-K R           (파일이 없으면 비활성)
Open Markdown Preview        Ctrl-Shift-V       (마크다운 버퍼만)
```

Zed는 맨 위에 디버그 항목, 코드 이동 7개, Rename Symbol·Format·Code Actions·AI 항목을 둔다. 모두 제외했으므로 메뉴는 Cut부터 시작하고 앞쪽 구분선은 없다.

| ID | 문구 | 액션 | 단축키 | 표시 조건 |
| --- | --- | --- | --- | --- |
| I-908 | Cut | `editor::Cut` | Ctrl-X | 항상 |
| I-909 | Copy | `editor::Copy` | Ctrl-C | 항상 |
| I-910 | Copy and Trim | `editor::CopyAndTrim` | 없음 | 항상 (줄 앞 공통 들여쓰기를 빼고 복사, 판단) |
| I-911 | Paste | `editor::Paste` | Ctrl-V | 항상 |
| I-912 | Reveal in File Explorer | `editor::RevealInFileManager` | Ctrl-K R | 파일이 없으면 **비활성** |
| I-913 | Open Markdown Preview | `markdown::OpenPreview` | Ctrl-Shift-V | 마크다운 버퍼 |

거터(줄 번호 영역)와 스크롤바의 우클릭 메뉴는 조사하지 않았다 (확신 낮은 부분).

### 11.5 마크다운 미리보기 본문

출처: [markdown_preview_view.rs:1960-2012](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1960-L2012).

**I-666**

```text
Copy                         Ctrl-C             (선택한 텍스트가 있을 때)
Copy as Markdown                                (선택한 마크다운 원문이 있을 때)
Copy Link                                       (링크 위에서 우클릭했을 때)
```

구분선은 없다. **세 조건이 모두 없으면 메뉴를 열지 않는다** (선택 없음 + 링크 아님이면 우클릭해도 아무것도 나오지 않는다, I-923).

| ID | 문구 | 액션 | 단축키 | 표시 조건 |
| --- | --- | --- | --- | --- |
| I-920 | Copy | `markdown::Copy` | Ctrl-C (미리보기 텍스트 컨텍스트) | 선택한 텍스트가 있을 때 |
| I-921 | Copy as Markdown | `markdown::CopyAsMarkdown` | 없음 | 선택한 마크다운 원문이 있을 때 |
| I-922 | Copy Link | 링크 URL을 클립보드로 복사하는 핸들러 | 없음 | 링크 위일 때 |

### 11.6 사이드바

사이드바에서 우클릭 메뉴가 있는 곳은 **프로젝트(폴더) 헤더 행** 하나다. 헤더 오른쪽의 ⋯ 버튼과 같은 메뉴를 연다 (I-930, [sidebar.rs:2518-2528](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2518-L2528)). 우리 앱에서 이 행은 **등록 폴더 항목**이다.

**I-667 등록 폴더 항목**

```text
Move Up                                         (폴더 2개 이상, 맨 위면 비활성)
Move Down                                       (폴더 2개 이상, 맨 아래면 비활성)
──────────────
Remove
```

| ID | 문구 | 동작 | 단축키 | 표시 조건 | 출처 |
| --- | --- | --- | --- | --- | --- |
| I-934 | Move Up | 등록 폴더 순서를 한 칸 위로 | 없음 | 폴더가 2개 이상. 맨 위면 비활성 | [sidebar.rs:2963-2995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2963-L2995) |
| I-940 | Move Down | 등록 폴더 순서를 한 칸 아래로 | 없음 | 폴더가 2개 이상. 맨 아래면 비활성 | [sidebar.rs:2963-2995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2963-L2995) |
| I-935 | Remove | 폴더 **등록 해제**. 디스크의 폴더·파일은 지우지 않는다 (I-107, I-093) | 없음 | 항상 | [sidebar.rs:3005-3010](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L3005-L3010) |

Zed 원본은 맨 위에 "Open Project in New Window"와 "Focus Project"가 있다. 둘 다 포함 목록에 없어 뺐고(11.9) 그 앞 구분선도 만들지 않았다. 등록 폴더 순서 바꾸기(Move Up·Move Down)는 사용자 결정으로 포함이다. 끌어서 순서를 바꾸는 동작은 Zed 기준을 따르되 소스에서 확인하지 못했다 (확신 낮은 부분). 나머지 사이드바 우클릭:

- **사이드바 빈 공간:** 우클릭 메뉴 없음 (I-937, [sidebar.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs)).
- **폴더 추가 버튼:** 우클릭 메뉴 없음. 왼쪽 클릭이 최근 프로젝트 팝업을 연다 (I-939, [sidebar.rs:6770-6776](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L6770-L6776)).

### 11.7 상태 표시줄 (아이콘 버튼)

**우리 앱의 상태 표시줄 아이콘에는 우클릭 메뉴가 없다.** Zed의 메뉴는 모두 제외 결정에 걸린다.

| ID | 대상 | Zed의 우클릭 메뉴 | 우리 앱 | 출처 |
| --- | --- | --- | --- | --- |
| I-947 | 프로젝트 패널 버튼 | "Dock Left" / "Dock Right"(현재 위치에 체크) + 구분선 + "Hide Button" | **메뉴 없음.** 오른쪽·하단 Dock 관련 메뉴는 제외 (Dock Right 포함), Hide Button은 포함 목록에 없다 | [dock.rs:1450-1535](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1450-L1535) · [dock.rs:1403-1406](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1403-L1406) · [project_panel.rs:8260-8262](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8260-L8262) · [project_panel.rs:8325-8330](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8325-L8330) |
| I-948 | 커서 위치 등 일반 아이템 | "Hide Button" 하나 (해당 설정을 false로 저장) | **메뉴 없음** (포함 목록에 없음) | [status_bar.rs:204-225](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L204-L225) · [status_bar.rs:292-330](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L292-L330) · [cursor_position.rs:299-305](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L299-L305) |
| I-949 | 사이드바 토글 (상태 표시줄) | "Left" / "Right"(사이드바 위치 변경) | **메뉴 없음** (사이드바는 왼쪽 고정) | [status_bar.rs:241](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L241) |
| I-950 | 상태 표시줄 빈 곳 | 없음 | 없음 | [status_bar.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs) |

### 11.8 우클릭 메뉴가 없는 곳

Zed와 같게 아래에는 우클릭 메뉴를 만들지 않는다.

| ID | 대상 | 사실 | 출처 |
| --- | --- | --- | --- |
| I-590 | 탭 바 빈 공간 | 우클릭 메뉴 **없음** (우클릭 메뉴는 탭에만 있다) | [pane.rs:3138](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3138) |
| I-591 | 탭 바 오른쪽 `+`·분할 | 우클릭이 아니라 **왼쪽 클릭 메뉴**다. `+` = New File (Zed 원본의 Open File·Search·Terminal 항목은 뺀다, 01-LAYOUT L-71) / 분할 = Split Right, Split Left, Split Up, Split Down | [pane.rs:4364-4412](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4364-L4412) |
| I-945 | 타이틀 바 빈 곳 | Zed 메뉴 없음 (Linux 클라이언트 장식일 때만 OS 창 메뉴를 부른다). Windows는 처리 코드가 없다 | [platform_title_bar.rs:339-347](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platform_title_bar.rs#L339-L347) |
| I-946 | ☰·프로젝트 이름 버튼 | 우클릭 메뉴 없음 (왼쪽 클릭 메뉴·팝업만) | [title_bar.rs:831-858](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L831-L858) |
| I-951 | Dock 경계 (리사이즈 핸들) | 없음 (더블클릭 = 크기 초기화만) | [dock.rs:1281-1336](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1281-L1336) |
| I-952 | 탭 바 ← → · 최대화 버튼 | 없음 | [pane.rs:3508-3553](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3508-L3553) · [pane.rs:4417-4430](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4417-L4430) |

### 11.9 제외한 항목 (사용자 결정 반영)

최종 구조(11.2~11.7)에서 뺀 항목이다. 결정 목록(`_decisions.md`)에 없는 항목은 "포함 목록에 없음"으로 제외했다. 사이드바 등록 폴더의 Move Up·Move Down, 탭의 Pin Tab, 루트의 폴더 등록·해제는 포함이라 최종 구조에 있다.

| ID | 영역 | 원본 문구 | 이유 |
| --- | --- | --- | --- |
| I-545, I-546 | 프로젝트 패널 (파일·폴더) | Undo, Redo | 포함 목록에 없음 (파일 작업 실행 취소 없음). 키 표의 I-038도 제외 |
| I-578 | 탭 | Make Tab Read-Only / Make Tab Editable (Unlock Tab) | **사용자 결정으로 제외** (탭 읽기 전용은 결정 목록에 없다) |
| I-931 | 사이드바 | Open Project in New Window | 포함 목록에 없음. 우리 앱은 새 창이 아니라 창 하나에서 폴더를 바꾼다 |
| I-932 | 사이드바 | Focus Project / Focus Last Project | 포함 목록에 없음. 폴더 항목을 누르면 활성화된다 |
| I-933 | 사이드바 | Open Worktrees 목록, Close Worktree | 기존 제외 (Git worktree) |
| I-938, I-949 | 사이드바 토글 | Left / Right (사이드바 위치) | 포함 목록에 없음 (사이드바는 왼쪽 고정) |
| I-947, I-948 | 상태 표시줄 | Dock Left / Dock Right, Hide Button | 제외 목록 (오른쪽·하단 Dock 메뉴), Hide Button은 포함 목록에 없음 |
| I-537 | 프로젝트 패널 (폴더) | Find in Folder… | 제외 목록 (프로젝트 검색) |
| I-902~I-906 | 편집기 | 코드 이동, Rename Symbol, Format, Code Actions, AI | 기존 제외 (LSP·AI) |

### 11.10 우클릭 메뉴에서 확신 낮은 부분

- **Close의 단축키 (I-570):** 메뉴 액션은 `close_pinned = true`인데 Ctrl-W 바인딩은 `false`다. 액션 비교가 인자까지 보면 키가 표시되지 않는다. 실제 표시는 미확인이다.
- **표시 단축키 선택:** 같은 액션에 키가 여럿이면 "마지막 정의 우선"으로 골랐다 (추정). Copy = Ctrl-C, Paste = Ctrl-V, Trash = Delete, Delete = Ctrl-Delete, Cut = Ctrl-X. 탭 메뉴의 Copy Relative Path에 키가 없다는 것도 편집기 컨텍스트에서 못 찾은 데 근거한 추정이다. VSCode 덧씌움 후에도 같은 키다 (I-832).
- **수식키 표기 순서:** Close Others는 키맵에 `ctrl-shift-alt-t`로 적혀 있다. 표기 순서 규칙(Ctrl → Alt → Shift)에 따라 "Ctrl-Alt-Shift-T"로 적었다.
- **미리보기 탭의 파일 항목 (I-589):** Copy Path·Reveal 항목이 나오는지 확인하지 못했다.
- **여러 항목 표시 상태 (I-562):** 액션 구현까지 보지 않았다.
- **항목 높이 (I-508):** 약 25px은 계산한 추정이다.
- **Windows 타이틀 바 우클릭 (I-945):** Zed 코드에는 처리가 없다. 타이틀 바가 OS의 이동 영역으로 등록돼 있어 Windows 시스템 메뉴가 OS 차원에서 뜨는지는 미확인이다.
- **편집기 거터·스크롤바 우클릭:** 조사하지 않았다.
- **사이드바 순서 바꾸기 (I-934, I-940):** Move Up·Move Down 메뉴는 확인했다. 끌어서 순서를 바꾸는 동작이 Zed 사이드바에 있는지는 확인하지 못했다.
- **줄 번호:** 일부 링크의 줄 범위는 ±수 줄 차이가 날 수 있다 (I-502, I-519, I-931~I-935, I-947, I-948).

## 12. 확신 낮은 부분

| ID | 항목 | 한계 | 출처 |
| --- | --- | --- | --- |
| I-121 | 실행 검증 | Zed 앱을 조작하지 않고 이미지와 소스만 봤다. 설치본 버전, 사용자 설정, Vim·base keymap에 따라 화면·동작이 다를 수 있다. 모든 키는 위 커밋의 Windows 기본 키맵 기준이다 | [keymap:1-4](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1-L4) · [키맵 재정의](https://zed.dev/docs/key-bindings) |
| I-122 | 문서와 키맵 충돌 | 공식 시작 문서는 Ctrl+O를 폴더 열기로 넓게 설명하지만, 이 커밋의 Windows 키맵은 Ctrl+O = 파일 열기(OpenFiles), Ctrl+K Ctrl+O = Open이다. **키 표는 키맵 소스를 따랐다.** 실제 대화 상자의 허용 항목은 미확인 | [keymap:24-27](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L24-L27) · [Getting Started](https://zed.dev/docs/) |
| I-123 | 스크롤 동기화 | 커서·선택 기반 블록 강조와 본문 클릭 → 원문 위치 이동만 확인했다. 휠·스크롤바의 양방향 비율 동기화는 미확인이다 (I-083) | [markdown_preview_view.rs:561-589](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L561-L589) · [markdown_preview_view.rs:743-760](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L743-L760) · [markdown_preview_view.rs:1171-1185](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1171-L1185) |
| I-124 | 트리 상태 영속화 | 펼침, 접힌 경로 조각, 가로·세로 스크롤의 재시작 복원은 미확인이다. 프로젝트를 바꿨다 돌아오는 메모리 유지와 앱 완전 종료 후 복원을 따로 검증해야 한다 (I-120) | [project_panel.rs:888-932](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L888-L932) · [multi_workspace.rs:1330-1385](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/multi_workspace.rs#L1330-L1385) |
| I-125 | 휴지통·링크 예외 | Windows 휴지통 권한, 볼륨·네트워크 경로, 실패 때 동작, OS URL 처리기별 결과는 실행으로 확인하지 않았다. 휴지통·영구 삭제의 분리와 확인 창, 로컬 마크다운 링크의 전용 처리는 소스에서 확인한 범위다 | [project_panel.rs:2355-2399](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2355-L2399) · [project_panel.rs:2935-3006](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2935-L3006) · [markdown_preview_view.rs:1299-1382](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1299-L1382) |
| I-126 | 채용 표기 | 1절 표의 "채용·제외"는 사용자 결정(`_decisions.md`)과 요건 1~7, 기준 화면 규칙에 맞춘 것이고 Zed 사실이 아니다 | 판단 |
| I-127 | 줄 번호 | 일부 링크의 줄 범위는 grep으로 잡은 근사치다. 파일과 커밋은 정확하다 | — |


## 13. 편집기 화면과 조작

요건 4(원문 편집)의 동작이다. 편집기 조사(`zed-editor.md`)의 ID(E-040~E-106)를 그대로 쓴다. 화면 치수와 색은 [01-LAYOUT.md](01-LAYOUT.md) 4.12절이다. 키는 **적용 keymap(VSCode 덧씌움 후)** 기준이고 표기는 메뉴·툴팁 형식(`Ctrl-F`, 8.4절)이다. 사용자 결정(`_decisions.md`)의 제외 기능(LSP·AI·Git·디버그·원격·프로젝트 검색)은 쓰지 않았다. Picker·대화상자·인라인 입력은 14절이다.

### 13.1 마우스 선택

| ID | 조작 | 결과 | 출처 |
| --- | --- | --- | --- |
| E-040 | 클릭 1번 | 커서 이동 (문자 단위 선택 모드) | [selection.rs:1224-1290](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/selection.rs#L1224-L1290) |
| E-041 | 더블클릭 | 클릭 위치의 **단어** 선택. 이어서 끌면 단어 단위로 늘어난다 | 위와 같음 |
| E-042 | 세 번 클릭 | **줄 전체**(다음 줄 시작까지, 줄바꿈 포함) 선택. 끌면 줄 단위 | 위와 같음 |
| E-043 | 네 번 이상 | **전체** 선택 | 위와 같음 |
| E-044 | 끌기 | 누른 모드(문자·단어·줄) 단위로 선택을 늘린다 | [mouse.rs:460-480](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element/mouse.rs#L460-L480) |
| E-045 | Shift+클릭 | 기존 선택을 클릭 위치까지 늘린다 (Shift만 누르고 Alt·Ctrl은 아닐 때). 이전 클릭 횟수 모드를 유지한다 | [mouse.rs:778-790](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element/mouse.rs#L778-L790) · [selection.rs:1157-1172](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/selection.rs#L1157-L1172) |
| E-046 | **Alt+클릭** | **커서 추가**(다중 커서). `multi_cursor_modifier` 기본 `"alt"` (`"cmd_or_ctrl"`로 바꾸면 Ctrl) | [mouse.rs:793-800](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element/mouse.rs#L793-L800) · [editor.rs:4212-4217](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/editor.rs#L4212-L4217) · [default.json:142](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L142) |
| E-047 | 열(세로) 선택 | **Ctrl+Shift+끌기** = 마우스 위치부터 열 선택. **Alt+Shift+끌기** = 기존 선택에서부터 열 선택. **가운데 버튼 끌기**(Windows) = 열 선택 | [editor.rs:4219-4235](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/editor.rs#L4219-L4235) · [mouse.rs:847-870](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element/mouse.rs#L847-L870) |
| E-048 | 가운데 클릭 붙여넣기 | 설정은 true지만 **Linux·FreeBSD 전용**(primary selection)이다. **Windows에서는 하지 않는다** | [mouse.rs:940-965](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element/mouse.rs#L940-L965) · [default.json:743](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L743) |
| E-049 | 선택 끌어 옮기기 | `drag_and_drop_selection.enabled` true, `delay` **300ms**: 선택 영역 안을 눌러 300ms가 지난 뒤 끌면 텍스트를 옮긴다. 그 전에 끌면 새 선택을 시작한다. 조건: 클릭 1번, Shift 없음 | [default.json:438-444](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L438-L444) · [mouse.rs:665-685](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element/mouse.rs#L665-L685) |
| E-050 | Ctrl+클릭 | 정의로 이동(LSP). **우리 앱에서 제외** | [hover_links.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/hover_links.rs) |
| E-051 | 우클릭 | 선택 밖이면 커서를 그 위치로 옮긴 뒤 우클릭 메뉴 (11.4절 I-503, I-665) | [mouse_context_menu.rs:160-200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/mouse_context_menu.rs#L160-L200) |
| E-009 | 거터 클릭 | 줄 전체 선택 (세 번 클릭과 같음). 끌면 여러 줄 선택 | [mouse.rs:595-617](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element/mouse.rs#L595-L617) |

### 13.2 기본 편집 설정값

`assets/settings/default.json`의 값이다.

| ID | 키 | 기본값 | 동작 | 줄 |
| --- | --- | --- | --- | --- |
| E-055 | `tab_size` / `hard_tabs` | 4 / false | Tab = 공백 4칸 | [default.json:1699-1701](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1699-L1701) |
| E-056 | `auto_indent` / `auto_indent_on_paste` | `"syntax_aware"` / true | 새 줄 들여쓰기를 구문에 맞춘다. 붙여넣기도 맞춘다 | [default.json:519-526](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L519-L526) |
| E-057 | `use_autoclose` / `use_auto_surround` | true / true | 여는 괄호·따옴표를 치면 닫는 문자를 넣는다. 선택한 채 치면 감싼다. 자동으로 넣은 닫는 문자만 건너뛰기·자동 삭제 대상이다 | [default.json:512-530](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L512-L530) |
| E-058 | `remove_trailing_whitespace_on_save` | **true** | 저장할 때 줄 끝 공백을 지운다 | [default.json:1617](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1617) |
| E-059 | `ensure_final_newline_on_save` | **true** | 저장할 때 파일 끝 줄바꿈을 하나 보장한다 | [default.json:1626](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1626) |
| E-059a | `format_on_save` | `"off"` | 저장 시 포맷 없음 (포매터를 만들지 않는 것과 일치) | [default.json:1653](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1653) |
| E-059b | `autosave` | `"off"` | 자동 저장 없음. 수정 표시 점(L-65)과 닫기 확인창(13.4절)이 기본 동작 | [default.json:1471](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1471) |
| E-059c | 실행 취소 묶음 | 편집 간격이 **300ms** 안이면 같은 실행 취소 단위로 묶는다 (`group_interval`). 키: Ctrl-Z 실행 취소, Ctrl-Y 또는 Ctrl-Shift-Z 다시 실행 | [text.rs:226-229](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/text/src/text.rs#L226-L229) · [text.rs:290-296](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/text/src/text.rs#L290-L296) · [keymap:77-79](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L77-L79) |
| E-059d | 자주 쓰는 편집 키 (편집기) | Ctrl-D 다음 일치 항목 추가(`replace_newest:false`, Ctrl-F3도 같음) · Ctrl-Shift-L / Ctrl-F2 모든 일치 선택 · Ctrl-Alt-Up/Down 커서 위·아래 추가 · Alt-Up/Down 줄 옮기기 · Shift-Alt-Up/Down 줄 복제 · Ctrl-L 줄 선택 · Ctrl-/ 줄 주석 토글 (VSCode 덧씌움: Ctrl-K Ctrl-C도) · Ctrl-Shift-A 블록 주석 (VSCode) · Shift-Alt-I 선택을 줄마다 커서로 (VSCode). 덧씌움으로 바뀐 것은 1.4절에 **기본→VSCode**로 표시했다 | [keymap:544-556](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L544-L556) · [vscode.json:12-17](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json#L12-L17) |
| E-022 | `soft_wrap` | `"none"` | 줄을 접지 않고 가로 스크롤. 토글 Alt-Z (VSCode) | [default.json:1687-1696](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1687-L1696) |

### 13.3 찾기·바꾸기 창 동작·키·툴팁

치수와 색은 01-LAYOUT 4.12.6절(E-060~E-072)이다. 이 절은 동작과 키다. 사용자 결정으로 포함한 기능이다 (프로젝트 전체 검색은 제외).

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| E-073 | 끝까지 갔을 때 | `search_wrap` 기본 true: 끝에서 처음으로 돈다. false이면 "No more matches" 토스트(자동 사라짐) | [default.json:815-818](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L815-L818) · [search.rs:257](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/search.rs#L257) |
| E-074 | 열 때 미리 채우기 | `seed_search_query_from_cursor` 기본 **"always"**: 선택 텍스트(없으면 커서의 단어)로 입력칸을 채운다 | [default.json:842](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L842) |

| ID | 동작 | 키 (컨텍스트) | 툴팁 문구 | 출처 |
| --- | --- | --- | --- | --- |
| E-075 | 찾기 열기 / 바꾸기로 열기 | Ctrl-F / Ctrl-H (편집기, 전체 모드) | — (툴바 "Buffer Search" 버튼) | [keymap:137-138](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L137-L138) |
| E-076 | 다음 / 이전 일치 | Enter / Shift-Enter (찾기 창), F3 / Shift-F3 (Pane) | "Select Next Match" / "Select Previous Match" | [keymap:409-423](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L409-L423) · [keymap:522-523](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L522-L523) |
| E-077 | 모든 일치 선택 | Alt-Enter | "Select All Matches" | 위와 같음 |
| E-078 | 옵션 토글 | Alt-C 대소문자 · Alt-W 단어 · Alt-R 정규식 (Pane) | "Match Case Sensitivity" · "Match Whole Words" · "Use Regular Expressions" | [keymap:524-532](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L524-L532) · [search.rs:114-118](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/search.rs#L114-L118) |
| E-079 | 바꾸기 토글 / 선택 영역 찾기 | Ctrl-H / Ctrl-L (찾기 창), Shift-Alt-H / Alt-L (Pane) | "Toggle Replace" / "Toggle Search Selection" | [keymap:418-420](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L418-L420) · [keymap:525-526](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L525-L526) |
| E-079a | 바꾸기 다음 / 모두 | Enter / Ctrl-Enter (바꾸기 입력칸) | "Replace Next Match" / "Replace All Matches" | [keymap:425-431](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L425-L431) |
| E-079b | 닫기 / 편집기로 포커스 | Escape / Tab | "Close Search Bar" | [keymap:412-413](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L412-L413) |
| E-079c | 이전 검색어 기록 | Up / Down (찾기 입력칸) | — | [keymap:439-445](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L439-L445) |
| E-079d | 툴팁 형식 | 모든 버튼은 포커스를 받는 방식(`for_action_in`, 찾기 입력 포커스 기준)이라 단축키가 함께 보인다 | — | [search_bar.rs:42-70](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/search_bar.rs#L42-L70) |

### 13.4 저장·닫기·디스크 변경·새 파일

| ID | 항목 | 내용 | 출처 |
| --- | --- | --- | --- |
| E-080 | 확인창 방식 (Windows) | `use_system_prompts` 기본 **true** → Windows에서는 Zed가 그린 창이 아니라 **OS TaskDialog**를 띄운다. 창 제목 = 수준 이름(**"Warning"**·"Info"·"Critical"), 아이콘 = 경고·정보·오류, 큰 글 = 메시지, 작은 글 = 상세. Esc·창 닫기로 취소할 수 있다. 외형은 14.7절(P-013)에서 정했다 (우리 앱도 OS TaskDialog를 쓴다) | [default.json:290-293](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L290-L293) · [window.rs:723-805](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui_windows/src/window.rs#L723-L805) · [ui_prompt.rs:13-25](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui_prompt/src/ui_prompt.rs#L13-L25) |
| E-081 | 수정된 탭 닫기 (1개) | Warning. 메시지 "**`{경로}` contains unsaved edits. Do you want to save it?**" (경로는 워크트리 기준 상대 경로, 앞쪽을 잘라 80자 이내, 인라인 코드 표기). 경로가 없으면 "**This buffer contains unsaved edits. Do you want to save it?**". 버튼 **[Save] [Don't Save] [Cancel]** | [pane.rs:5067-5081](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L5067-L5081) · [pane.rs:2410-2420](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2410-L2420) |
| E-082 | 수정된 탭 여러 개 | Warning. "**Do you want to save changes to the following files?**" + 파일 목록(상세). 버튼 **[Save all] [Discard all] [Cancel]** | [pane.rs:2005-2015](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2005-L2015) |
| E-083 | 저장 실패 | "**Unable to save file: {오류}**" + **[Close Without Saving] [Cancel]** | [pane.rs:2055-2066](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2055-L2066) |
| E-084 | 디스크와 충돌한 상태에서 저장 | Warning. "**This file has changed on disk since you started editing it. Do you want to overwrite it?**" + **[Overwrite] [Discard Edits] [Cancel]** | [pane.rs:2263](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2263) · [pane.rs:2369-2375](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2369-L2375) |
| E-085 | 디스크에서 지워진 파일 저장 | Warning. "**This file has been deleted on disk since you started editing it. Do you want to recreate it?**" + **[Save] [Close] [Cancel]** | [pane.rs:2265](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2265) · [pane.rs:2334-2340](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2334-L2340) |
| E-086 | 디스크 변경 감지 | 파일이 바깥에서 바뀌면 **수정하지 않은 버퍼는 자동으로 다시 읽는다.** 수정한 버퍼는 그대로 두고 **충돌**로 본다 (디스크 시각이 마지막 저장 시각보다 늦고 저장 안 한 편집이 있음). 충돌이면 dirty로 취급하고 탭 점은 **Warning 색** (L-65, I-071) | [buffer.rs:1795-1825](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/language/src/buffer.rs#L1795-L1825) · [buffer.rs:2534-2574](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/language/src/buffer.rs#L2534-L2574) · [pane.rs:5090-5099](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L5090-L5099) |
| E-087 | 지워진 파일 탭 | 탭 제목에 **취소선**. 내용이 비어 있지 않으면 수정됨(dirty)으로 본다 | [items.rs:810-836](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L810-L836) · [buffer.rs:2541-2546](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/language/src/buffer.rs#L2541-L2546) |
| E-088 | 저장 키 | Ctrl-S 저장, Ctrl-Shift-S 다른 이름으로 저장(OS 저장 창), Ctrl-K S 모두 저장 (1.2절 I-005, I-006) | [keymap:654-660](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L654-L660) |
| E-089 | 새 파일 제목 | Ctrl-N(작업 영역)은 파일 없는 버퍼를 연다. 탭 제목 = **버퍼 첫 줄 내용**: 앞쪽 공백을 건너뛰고, 연속 공백은 하나로 줄이고, 줄바꿈 또는 **40자**에서 끊는다. 내용이 없으면 **"untitled"**. 저장 창의 제안 파일 이름은 제목 + 언어 확장자(예 `untitled.rs`) | [multi_buffer.rs:2196-2245](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/multi_buffer/src/multi_buffer.rs#L2196-L2245) · [items.rs:740-755](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L740-L755) |
| E-089a | 새 파일 저장 | 경로가 없는 버퍼에서 Ctrl-S → 다른 이름으로 저장 창. 비어 있는 새 버퍼는 수정됨이 아니라서 닫을 때 묻지 않는다 | [buffer.rs:2541-2546](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/language/src/buffer.rs#L2541-L2546) |

### 13.5 구문 단위 선택 (Tree-sitter)

사용자 결정으로 포함한 기능이다 (☰ Selection 메뉴 I-372~I-375).

| ID | 동작 | 키 | 규칙 | 출처 |
| --- | --- | --- | --- | --- |
| E-090 | Expand Selection | **Shift-Alt-Right** | 선택을 감싸는 다음 큰 구문 노드로 넓힌다. 문자열 안·인라인 노드에서는 먼저 **단어**를 고른다. 이전 선택을 기록 스택에 쌓는다 | [selection.rs:634-756](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/selection.rs#L634-L756) · [keymap:551](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L551) |
| E-091 | Shrink Selection | **Shift-Alt-Left** | 기록 스택에서 꺼내 바로 전 선택으로 되돌린다. 스택이 비면 아무것도 하지 않는다 | [selection.rs:758-795](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/selection.rs#L758-L795) · [keymap:552](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L552) |
| E-092 | Select Next / Previous Sibling | 기본 키 없음 (☰ Selection 메뉴, 명령 팔레트) | 같은 부모의 다음·이전 형제 노드를 고른다. 형제가 없으면 그 선택은 그대로 | [selection.rs:797-900](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/selection.rs#L797-L900) · [actions.rs:809-820](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/actions.rs#L809-L820) |
| E-093 | 스크롤 | — | 마지막으로 바뀐 선택이 보이게 자동 스크롤한다. 구문 선택 중에는 기록이 지워지지 않게 한다 | [selection.rs:700-720](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/selection.rs#L700-L720) |

### 13.6 커서 위치 표시와 Go to Line

| ID | 항목 | 내용 | 출처 |
| --- | --- | --- | --- |
| E-100 | 표시 형식 | "**{줄}:{열}**", 1부터 센다 (기준 화면 "9:11", Latte 화면 "1:1"). 열은 줄 시작부터 센 **문자 수** + 1. 선택이 있으면 뒤에 " (…)": `line_indicator_format` 기본 `"long"` → "**2 selections, 15 lines, 32 characters**"(개수 1이면 단수형), `"short"` → "2 s, 15 l, 32 c". 선택 줄 수는 끝 열이 0이 아니면 +1 | [cursor_position.rs:31-160](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L31-L160) · [cursor_position.rs:175-225](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L175-L225) · [default.json:2940-2945](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2940-L2945) |
| E-101 | 위치 기준 | 여러 선택이면 **가장 나중에 만든 선택의 끝**(head) 위치를 보인다 | [cursor_position.rs:112-145](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L112-L145) |
| E-102 | 버튼 | 일반 버튼, 글자 Small(12px). 툴팁 "**Go to Line/Column**" + **Ctrl-G**. 누르면 Go to Line 창 | [cursor_position.rs:230-262](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L230-L262) |
| E-103 | Go to Line 창 | 모달, 폭 **24rem(384px)**, `elevation_2`(툴팁·메뉴와 같은 떠 있는 표면, 01-LAYOUT 4.7절 L-211). 위: 한 줄 입력칸(아래 1px `border_variant`, padding 8×4). 아래: 도움말 줄(padding 8×4, Muted). placeholder = 현재 위치 "**{줄}:{열}**" | [go_to_line.rs:108-147](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L108-L147) · [go_to_line.rs:307-350](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L307-L350) |
| E-104 | 도움말 문구 | 입력 없음: "**Current Line: {줄} of {전체} (column {열})**" / `줄` 입력: "**Go to line {줄}**" / `줄:열`: "**Go to line {줄}, character {열}**" / `+n`·`-n`: "**Go to line {대상} (+n from current)**" | [go_to_line.rs:136-140](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L136-L140) · [go_to_line.rs:308-323](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L308-L323) |
| E-105 | 미리보기·확정·취소 | 입력하는 동안 대상 줄을 `editor_highlighted_line_background`로 강조하고 그 줄로 스크롤한다. Enter = 커서를 옮기고 닫는다. Esc·바깥 클릭 = 닫고 **원래 스크롤 위치로 되돌린다** | [go_to_line.rs:150-240](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L150-L240) |
| E-106 | 편집기가 아닐 때 | 활성 탭이 편집기가 아니어도 커서 위치를 **지우지 않고, 직전에 활성이던 편집기의 마지막 좌표를 그대로 보인다.** 그래서 Latte 화면은 미리보기 탭이 활성인데 "1:1"이 보인다. 이 버튼을 눌러 Go to Line을 여는 것은 활성 항목이 편집기일 때만이다 (P-027) | [cursor_position.rs:105-145](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L105-L145) · [cursor_position.rs:236-244](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L236-L244) |

### 13.7 스크롤 동작

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| E-033 | 스크롤바 표시 정책 | `scrollbar.show` 기본 `"auto"`: OS가 스크롤바 자동 숨김이면 스크롤할 때 보이고 **1초** 뒤 숨긴다. 자동 숨김이 아니면 항상 보인다. 가로·세로 모두 켜짐. Windows의 실제 판정은 미확인 | [default.json:655-692](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L655-L692) · [scroll.rs:28](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/scroll.rs#L28) · [scroll.rs:440-460](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/scroll.rs#L440-L460) |
| E-035 | 마지막 줄 너머 | `scroll_beyond_last_line` 기본 **`"one_page"`**: 마지막 줄을 화면 맨 위까지 올릴 수 있다 (한 화면만큼 더 스크롤) | [default.json:792](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L792) |
| E-036 | 스크롤 여백 | 키보드로 움직일 때 커서 위아래로 **3줄**(`vertical_scroll_margin`), 가로 **5글자**(`horizontal_scroll_margin`)를 남긴다. `autoscroll_on_clicks` false | [default.json:794-800](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L794-L800) |
| E-037 | 휠 | `scroll_sensitivity` 1.0, **Alt + 휠 = 빠른 스크롤 ×4**. Ctrl + 휠 글꼴 확대(`mouse_wheel_zoom`)는 기본 false라 쓰지 않는다 | [default.json:801-808](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L801-L808) · [mouse.rs:505-560](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element/mouse.rs#L505-L560) |

### 13.8 아직 조사하지 않은 것 (자리)

아래는 모두 **미조사**다. 값을 추측해서 채우지 않는다. 열 "확인할 곳"은 검토에서 지적된 시작점이다.

| 항목 | 규칙 | 확인할 곳 |
| --- | --- | --- |
| 한글 입력(IME) 조합 표시 (편집기·인라인 입력·찾기 입력칸) | 미조사 | `editor/src/element.rs` |
| 마크다운 미리보기의 추가 동작 (작업 목록 체크박스 클릭, 이미지, 코드 블록 복사 버튼) | 미조사 | `markdown_preview_view.rs`, `markdown/src/markdown.rs` |
| 탭 바 조작 세부 (탭 줄 위 바퀴 가로 스크롤, 탭 hover·활성 배경, 빈 곳 더블클릭) | 미조사 | `pane.rs`, `ui/src/components/tab.rs` |
| 사이드바 키보드 (Left/Right 선택 이동, Enter·Space, 선택 표시), 폭 조절 핸들(위치·폭·커서·더블클릭) | 미조사 | `default-windows.json`의 사이드바 컨텍스트, `sidebar.rs`, `multi_workspace.rs` |
| 알림(토스트) (위치, 모양, 닫기, 자동 사라짐; 13.3절 E-073 "No more matches"도 쓴다) | 미조사 | `workspace/src/notifications.rs`, `toast_layer.rs` |
| 빈 상태 화면 (폴더가 없을 때 프로젝트 패널, 탭이 없는 Pane) | 미조사 | `ui/src/components/project_empty_state.rs`, `project_panel.rs`, `pane.rs` |
| 창 최소 크기와 창 조작 (최소 360×240 확인 필요, 타이틀 바 더블클릭 최대화는 OS 동작) | 미조사 | `crates/zed/src/zed.rs`, `platform_title_bar.rs` |

### 13.9 편집기 동작에서 확신 낮은 부분

- **확인창 버튼 (E-080~E-085):** 버튼 순서가 Zed 배열 순서 그대로인지, 기본 버튼(Enter)이 첫 버튼인지, 인라인 코드 표기(백틱)가 OS 창에서 그대로 보이는지는 확인하지 못했다.
- **"1:1" 표시 (E-106):** 소스(P-027)로 확인했다. 편집기가 아닌 탭이 활성이어도 직전 편집기의 마지막 위치를 계속 보인다. 한 번도 편집기가 활성이 된 적이 없을 때의 표시는 미확인이다.
- **Latte 화면의 흐린 탭:** 편집기 조사는 판단하지 않았다. 규칙은 01-LAYOUT 4.11절 L-306이다.
- **스크롤바 자동 숨김 (E-033):** Windows에서 어떤 OS 설정을 읽는지 확인하지 않았다.
- **줄 번호:** 일부 링크의 줄 범위는 ±수 줄 차이가 날 수 있다.

## 14. Picker와 대화상자

Picker·대화상자 조사(`zed-pickers-dialogs.md`)의 ID(P-001~P-027)를 그대로 쓴다. 모양과 치수는 [01-LAYOUT.md](01-LAYOUT.md) 4.13절이다. 포함·제외는 사용자 결정(`_decisions.md`)에 따랐다. 키는 **적용 keymap(VSCode 덧씌움 후)** 기준이고 소스에서 다시 확인했다. 조사 문서와 소스가 다른 곳은 소스를 따랐고 14.10에 적었다.

### 14.1 Picker 공통 동작

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| P-001 | 열기·닫기 | 모달로 열린다 (위치·크기는 01-LAYOUT 4.13.1). 같은 토글 키를 다시 누르면 닫힌다 | [modal_layer.rs:108-122](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/modal_layer.rs#L108-L122) |
| P-004 | 선택·순환 | 위·아래로 움직이며 **끝에서 반대편으로 순환**한다. 머리글처럼 선택할 수 없는 줄은 건너뛴다. **마우스를 올리면 그 항목이 바로 선택된다.** 빈 결과는 "No matches" | [picker.rs:910-1089](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L910-L1089) · [picker.rs:1452-1459](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L1452-L1459) |
| P-005 | 키 컨텍스트 | `Picker`와 `menu`가 함께 켜지고 포커스는 검색 입력칸에 있다 | [render.rs:17-70](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L17-L70) |
| 〃 | 키 | 이전: **Up, Ctrl-P, Shift-Tab, PageUp** / 다음: **Down, Ctrl-N, Tab, PageDown** / 처음·끝: **Home, End** / 확정: **Enter** / 보조 확정: **Ctrl-Enter** / 입력 그대로 확정: **Alt-Enter** (Ctrl-Alt-Enter는 보조) / 닫기: **Escape, Ctrl-C** | [keymap:5-26](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L5-L26) |
| 〃 | 닫히는 조건 4가지 | ① Escape ② 모달 바깥 왼쪽 클릭 ③ 입력칸이 포커스를 잃을 때 (창이 활성이고 하위 메뉴로 간 것이 아닐 때) ④ 같은 토글 키 다시 입력 | [picker.rs:1047-1050](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L1047-L1050) · [picker.rs:1195-1203](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L1195-L1203) · [modal_layer.rs:308-313](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/modal_layer.rs#L308-L313) |
| 〃 | 우리 앱에서 쓰지 않는 키 | 선택기 입력칸의 Ctrl-Shift-A(동작 메뉴 `picker::ToggleActionsMenu`), Ctrl-Shift-S(다중 선택), Ctrl-Alt-P(미리보기)는 동작 메뉴·다중 선택·미리보기를 만들지 않으므로 **쓰지 않는다** (판단, 1.1절 I-705) | [specific-overrides.json:8-25](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/specific-overrides.json#L8-L25) |

### 14.2 명령 팔레트 (P-007)

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| P-007 | 열기 | **Ctrl-Shift-P** 또는 **F1** (`command_palette::Toggle`). 사용자 결정으로 포함. 우리가 채택한 명령만 등록한다 (☰ 메뉴·키 표에 있는 명령) | [keymap:696-697](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L696-L697) |
| 〃 | 항목 | 왼쪽: 사람이 읽기 좋게 바꾼 명령 이름 (예 `workspace: save`), 퍼지 일치 글자 강조. 오른쪽: 팔레트를 열기 직전 포커스 기준의 **키 배지**. 최근 쓴 명령에는 `HistoryRerun`(14, Muted)이 붙고, 그 행에 마우스를 올리면 `Close` 버튼(툴팁 "**Remove from Command History**")으로 바뀐다 | [command_palette.rs:768-815](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/command_palette/src/command_palette.rs#L768-L815) |
| 〃 | 기록 삭제 키 | **Shift-Backspace** (`command_palette::RemoveSelected`). 조사 문서는 Delete라고 썼으나 키맵 소스는 Shift-Backspace다 | [keymap:1297-1302](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1297-L1302) |
| 〃 | 푸터 | 오른쪽 정렬 버튼 둘: 기존 바인딩이 있으면 "**Change Keybinding…**", 없으면 "**Add Keybinding…**" (보조 확정 키) 그리고 "**Run**" (Enter). 앞 버튼은 15절의 키맵 편집 화면에서 그 액션을 연다 | [command_palette.rs:817-868](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/command_palette/src/command_palette.rs#L817-L868) |

### 14.3 파일 찾기 Go to File (P-008)

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| P-008 | 열기 | **Ctrl-P** 또는 **Ctrl-E** (`file_finder::Toggle`). 사용자 결정으로 포함 | [keymap:692](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L692) · [keymap:695](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L695) |
| 〃 | 검색어 | 파일 이름을 퍼지 검색한다. 이름 뒤에 `:줄` 또는 `:줄:열`을 붙이면 그 파일을 열고 바로 그 위치로 간다 | [file_finder.rs:480-520](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L480-L520) |
| 〃 | 항목 | 파일 아이콘, 파일 이름(가운데 말줄임), 상대 경로(작은 글자 Muted, 앞쪽 말줄임). 최근 연 파일(history)은 위쪽에 두고 오른쪽 끝에 `HistoryRerun`. 최근 항목과 검색 결과 사이에 구분선 | [file_finder.rs:2022-2034](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L2022-L2034) · [file_finder.rs:2281-2360](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L2281-L2360) |
| 〃 | Ctrl 누른 채 순환 | Ctrl을 누른 채 P를 연속으로 누르면 다음 항목으로 가고, **Ctrl을 떼는 순간** 선택한 파일을 연다 | [file_finder.rs:100-105](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L100-L105) · [file_finder.rs:234-249](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L234-L249) |
| 〃 | 파일 찾기 안의 키 | **Ctrl-Shift-P** 이전 항목, **Ctrl-J / K / H / L** = 아래·위·왼쪽·오른쪽 분할로 열기 (포함: 분할은 포함 기능). Tab은 다중 선택이 아니라 일반 다음 항목 이동으로 쓴다 (다중 선택 제외) | [specific-overrides.json:27-40](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/specific-overrides.json#L27-L40) |
| 〃 | 만들지 않는 것 | 파일 미리보기와 레이아웃 전환, 다중 선택, "Include Ignored Files"(Ctrl-Shift-I, Git 무시 파일), 푸터의 "Actions…" | [footer.rs:95-226](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/footer.rs#L95-L226) |

### 14.4 최근 폴더 Open Recent (P-009)

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| P-009 | 열기 | **Ctrl-R** (`projects::OpenRecent`). 타이틀 바 프로젝트 이름 버튼과 사이드바 폴더 추가 버튼도 같은 Picker를 연다. 조사 문서의 Ctrl+Alt+O는 키맵에서 주석 처리된 줄이라 **Ctrl-R이 맞다** | [keymap:646-648](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L646-L648) |
| 〃 | 두 가지 모양 | **모달**(Ctrl-R, 타이틀 바 버튼): 폭 672, 항목에 폴더 이름 + 아래 줄 경로. **팝오버**(사이드바 폴더 추가 버튼, 툴팁 "Add Project"): 폭 320, 폴더 이름만 한 줄(전체 경로는 hover 툴팁) | [recent_projects.rs:668-693](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L668-L693) · [recent_projects.rs:761-784](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L761-L784) |
| 〃 | 구역 (판단) | Zed는 "Current Folders", "This Window", "Recent Projects" 세 구역이다. 우리 앱은 폴더가 하나만 활성이므로 **"This Window"**(= 사이드바에 등록한 폴더 목록, 활성 폴더에 `Check`(14, Accent)) 와 **"Recent Projects"**(최근 연 폴더 기록) 두 구역으로 한다. 구역 머리글은 선택할 수 없는 줄이고, 둘째 구역 위에 위 여백 4px와 1px 구분선이 들어간다 | [recent_projects.rs:1043-1125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L1043-L1125) · [recent_projects.rs:1235-1242](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L1235-L1242) |
| 〃 | 항목 오른쪽 버튼 (hover) | "This Window" 행: `Close`, 툴팁 "**Remove Project from Window**" (= 우리 앱의 **등록 해제**, 디스크는 그대로). "Recent Projects" 행: `FolderInclude` 툴팁 "**Add Folder to this Project**" (= 폴더 **등록**, 원본의 "As a multi-root folder" 부제는 쓰지 않는다), `Close` 툴팁 "**Remove from Recent Projects**". 새 창으로 열기(`ArrowUpRight`)는 뺀다 (제외 목록에 없는 "새 창에서 프로젝트 열기"는 포함 목록에 없음) | [recent_projects.rs:1387-1447](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L1387-L1447) · [recent_projects.rs:1549-1621](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L1549-L1621) |
| 〃 | placeholder·빈 결과 | placeholder "**Search projects…**" (한 글자 `…`). 등록·최근 항목이 하나도 없으면 "**Recently opened projects will show up here**", 검색어와 일치하는 것이 없으면 "**No matches**" | [recent_projects.rs:932-934](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L932-L934) · [recent_projects.rs:1218-1225](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L1218-L1225) |
| 〃 | 푸터 | 팝오버: 전체 폭 버튼 "**Open Local Folders**" (`workspace::Open`, Ctrl-O / Ctrl-K Ctrl-O). 모달: 오른쪽 정렬, 선택한 항목 종류에 따라 "**Remove**" (Shift-Backspace) + "**Open**" 또는 "**Activate**" (Enter). 원격 폴더 항목과 "Actions" 팝오버·새 창 버튼은 만들지 않는다 | [recent_projects.rs:1688-1757](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L1688-L1757) · [recent_projects.rs:1770-1988](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L1770-L1988) |
| 〃 | 이 Picker의 키 | **Shift-Backspace** 선택 항목 제거(조사 문서의 Delete는 소스와 다르다), **Ctrl-K Ctrl-O** 로컬 폴더 열기, **Ctrl-Shift-Enter** 선택한 폴더를 등록(`AddToWorkspace`), **Ctrl-Shift-A** 폴더 추가(`AddFolderToProject`). 원본에서는 입력칸 안의 Ctrl-Shift-A가 동작 메뉴에 가려지지만(I-705) 우리는 동작 메뉴가 없어 **폴더 추가 키로 쓴다** (판단, 1.2절 I-029 보정) | [keymap:1303-1311](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1303-L1311) |

### 14.5 테마·아이콘 테마 선택기 (P-010, P-011)

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| P-010 | 열기 | 테마: **Ctrl-K Ctrl-T** (`theme_selector::Toggle`), ☰ Select Theme..., ∨ Themes…. 아이콘 테마: 기본 키 없음, ☰ Select Icon Theme..., ∨ Icon Themes…, 명령 팔레트 | [keymap:688](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688) |
| 〃 | 목록 | 테마 6종(One Dark, One Light, Catppuccin Latte·Frappé·Macchiato·Mocha), 아이콘 테마 2종(Zed 기본, Catppuccin Icons)만. 정렬: **다크를 먼저 이름 오름차순, 이어서 라이트를 이름 오름차순** → Catppuccin Frappé, Catppuccin Macchiato, Catppuccin Mocha, One Dark, Catppuccin Latte, One Light. 열 때 **현재 적용 중인 항목**이 선택돼 있다 | [theme_selector.rs:175-185](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L175-L185) · [theme_selector.rs:372-377](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L372-L377) |
| 〃 | 항목 | 이름(퍼지 강조) + 오른쪽에 **열 때 활성이던 항목**에만 `Check`(16, Muted) | [theme_selector.rs:505-528](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L505-L528) |
| P-010, P-011 | 푸터 | 테마 선택기 오른쪽 "**Install Themes**", 아이콘 테마 선택기 오른쪽 "**Install Icon Themes**" 버튼이 **확장 관리 화면을 해당 카테고리 필터로 연다** (16.1절). 왼쪽 "View Theme Docs" / "View Icon Theme Docs"는 Zed 문서 링크라 뺀다 | [theme_selector.rs:530-569](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L530-L569) · [icon_theme_selector.rs:303-360](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L303-L360) |
| P-010, P-011 | 미리 적용 | 위·아래 이동이나 마우스 올림마다 **앱 전체 테마(아이콘 테마)를 메모리에서 바로 바꾼다.** **Enter·클릭**일 때만 settings.json에 저장하고 닫는다. **Escape·바깥 클릭**으로 닫으면 **처음 테마로 되돌린다** | [theme_selector.rs:314-357](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L314-L357) · [theme_selector.rs:403-413](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L403-L413) · [theme_selector.rs:463-503](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L463-L503) |
| P-011 | 아이콘 테마 | 위와 같은 구조. 이동하는 동안 프로젝트 패널·탭의 아이콘이 실시간으로 바뀐다 | [icon_theme_selector.rs:187-320](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L187-L320) |

### 14.6 줄 이동 Go to Line (P-012)

13.6절(E-100~E-106)에 모양과 문구를 채웠다. 여기서는 Picker와 다른 점만 적는다.

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| P-012 | 구조 | **Picker를 쓰지 않는 자체 모달**이다. 열기 **Ctrl-G** (`go_to_line::Toggle`, 편집기 컨텍스트) 또는 상태 표시줄 줄:열 버튼 | [keymap:630](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L630) · [go_to_line.rs:78](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L78) |
| 〃 | 입력 형식 | `줄`, `:줄`, `줄:열`, 상대 `+5`·`-3`·`f5`·`b3`. 입력이 비어 있을 때 **Tab**을 누르면 placeholder(현재 위치 "줄:열")가 입력칸에 채워진다 | [go_to_line.rs:112-125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L112-L125) · [go_to_line.rs:312-327](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L312-L327) |
| 〃 | 확정·취소 | Enter = 커서를 옮기고 접힌 곳을 펼치고 편집기에 포커스를 돌려준다. Esc·바깥 클릭 = 줄 강조를 지우고 **열기 전 스크롤 위치로 되돌린다** | [go_to_line.rs:154-163](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L154-L163) · [go_to_line.rs:288-305](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L288-L305) |

### 14.7 대화상자

문구는 화면에 보이는 짧은 UI 문자열이라 그대로 적는다. 작은따옴표·백틱은 원본 그대로다. **메시지 속 백틱은 OS 대화상자에서 글자 그대로 보인다** (01-LAYOUT P-013). **기본 버튼은 배열의 첫 번째, Esc는 "Cancel" 버튼**이다.

| ID | 상황 | 수준 | 메시지 / 상세 | 버튼 (순서대로) | 출처 |
| --- | --- | --- | --- | --- | --- |
| P-014 | 수정된 탭 하나 닫기 | Warning | "`{경로}` contains unsaved edits. Do you want to save it?" (경로가 없으면 "This buffer contains unsaved edits. Do you want to save it?"). 경로는 워크트리 기준, 앞쪽을 잘라 80자 이내. 상세 없음 | **Save**, **Don't Save**, **Cancel** | [pane.rs:2411-2418](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2411-L2418) · [pane.rs:5067-5082](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L5067-L5082) |
| 〃 | 수정된 탭 여러 개 닫기 | Warning | "Do you want to save changes to the following files?" / 상세 = 파일 이름 목록(줄바꿈) | **Save all**, **Discard all**, **Cancel** | [pane.rs:2006-2023](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2006-L2023) |
| 〃 | 창 닫기 | Warning | 위와 같은 메시지 / 상세 = 저장 안 한 파일 목록 | **Save**, **Don't Save**, **Cancel** | [workspace.rs:4028-4045](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L4028-L4045) |
| 〃 | 저장 실패 | Warning | "Unable to save file: {오류}" / 상세 = 그 파일 이름 | **Close Without Saving**, **Cancel** | [pane.rs:2054-2072](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2054-L2072) |
| P-016 | 디스크와 충돌한 상태에서 저장 | Warning | "This file has changed on disk since you started editing it. Do you want to overwrite it?" | **Overwrite**, **Discard Edits**, **Cancel** | [pane.rs:2263](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2263) · [pane.rs:2369-2375](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2369-L2375) |
| 〃 | 디스크에서 지워진 파일 저장 | Warning | "This file has been deleted on disk since you started editing it. Do you want to recreate it?" | **Save**, **Close**, **Cancel** | [pane.rs:2265](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2265) · [pane.rs:2334-2340](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2334-L2340) |
| 〃 | 외부 파일을 놓았는데 같은 이름이 있음 | Info | "A file or folder with name `{이름}` already exists in the destination folder. Do you want to replace it?" | **Replace**, **Cancel** | [project_panel.rs:5078-5096](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L5078-L5096) |
| P-015 | 휴지통 (Delete 키) | Info | 한 개 "Do you want to trash `{이름}`?" / 여러 개 "Do you want to trash the following {N} files?" + 이름 줄 목록(최대 10개, 넘으면 ".. {N} files not shown"). 저장 안 한 버퍼가 있으면 끝에 빈 줄 + "It has unsaved changes, which will be lost." (여러 개면 "1 of these has unsaved changes, which will be lost." / "{K} of these have unsaved changes, which will be lost."). 상세 없음 | **Trash**, **Cancel** | [project_panel.rs:2836-2956](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2836-L2956) |
| 〃 | 영구 삭제 (Shift-Delete) | Info | 한 개 "Are you sure you want to permanently delete `{이름}`?" / 여러 개 "Are you sure you want to permanently delete the following {N} files?" + 목록·경고는 위와 같음. **상세 "This cannot be undone."** | **Delete**, **Cancel** | [project_panel.rs:2849](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2849) |

- 수정된 탭의 확인창 문구는 13.4절 E-081~E-085와 같다. 이 표가 상세 문구·창 닫기·삭제·덮어쓰기까지 더 자세하다.
- 창을 닫을 때 저장 확인을 할지는 04-APP-SPEC의 정책이다. Zed 기본(`restore_unsaved_buffers` = true)은 종료 때 묻지 않는다 (I-118).
- Git 변경 되돌리기 확인(P-017)은 Git 기능이라 뺀다.

**About 창 (P-018)** — 사용자 결정으로 포함

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| P-018 | 열기 | ☰ 앱 이름 메뉴의 "About <앱 이름>"(I-322, 키 없음) 또는 명령 팔레트. **이미 열려 있으면 새 창을 만들지 않고 기존 창을 활성화한다** | [zed.rs:1698-1711](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1698-L1711) |
| 〃 | 키 | Tab·Shift-Tab·화살표로 OK와 Copy 사이 포커스 이동, Enter로 포커스된 버튼 실행, **Escape로 창 닫기**. 처음 포커스는 OK | [zed.rs:1604-1610](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1604-L1610) · [zed.rs:1687-1688](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1687-L1688) |
| 〃 | 내용 | 01-LAYOUT 4.13.6. Copy를 누르면 "{메시지}\nCommit: {커밋}\nVersion: {버전}"이 클립보드에 들어가고 창이 닫힌다 | [zed.rs:1584-1596](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1584-L1596) |

### 14.8 인라인 이름 입력칸 (프로젝트 패널)

모양은 01-LAYOUT 4.13.7이다. 5.2절 I-091, I-092와 같은 동작을 더 자세히 적는다.

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| P-019 | 시작 | **새 파일·새 폴더:** 대상 폴더를 펼치고 가상 행을 넣은 뒤 **빈 입력칸**에 포커스. **이름 변경(F2, Enter):** 기존 이름을 채운다. 폴더는 이름 **전체**, 파일은 **확장자를 뺀 이름만** 선택한다. Windows에서 루트 폴더 이름 변경은 막혀 있다 | [project_panel.rs:2535-2537](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2535-L2537) · [project_panel.rs:2576-2579](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2576-L2579) · [project_panel.rs:2601-2620](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2601-L2620) |
| P-022 | 키 컨텍스트 | 입력칸에 포커스가 있으면 `ProjectPanel > menu > editing`, 없으면 `not_editing` | [project_panel.rs:7237-7250](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7237-L7250) |
| 〃 | Enter | 확정한다. **Windows에서는 이름 끝의 마침표를 모두 지운다.** 새 항목에서 입력이 `/` 또는 `\`로 끝나면 **폴더 생성으로 바뀐다.** `foo/bar/baz.rs`처럼 쓰면 **중간 폴더도 함께 만든다** | [project_panel.rs:1960-1964](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1960-L1964) · [project_panel.rs:2143-2255](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2143-L2255) |
| 〃 | 확정이 거부되는 경우 | ① 공백을 뺀 이름이 비어 있다 → 아무 일 없이 입력칸이 열려 있다 ② 같은 위치에 같은 이름이 이미 있다 → 입력칸이 열려 있다. 단 **이름 변경에서 자기 자신과 같은 이름**이면 편집만 끝내고 패널로 포커스를 돌려준다 | [project_panel.rs:2162-2164](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2162-L2164) · [project_panel.rs:2195-2223](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2195-L2223) |
| 〃 | Escape | 편집 상태를 버린다 (새 항목의 가상 행과 임시로 펼친 폴더도 없앤다). 이전 선택으로 되돌리고 **프로젝트 패널에 포커스** | [project_panel.rs:2411-2420](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2411-L2420) |
| 〃 | 포커스 잃음 | 창이 활성일 때 다른 곳을 누르면 확정을 시도한다. 이름이 유효하면 **자동으로 만들거나 이름을 바꾸고**, 비어 있거나 충돌이면 **편집을 취소**한다 | [project_panel.rs:828-848](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L828-L848) |
| P-021 | 검증 (입력할 때마다) | 앞뒤에 공백이 있으면 **경고** "**File or directory name contains leading or trailing whitespace.**". 같은 위치에 같은 이름이 이미 있으면(이름 변경에서 자기 자신은 제외) **오류** "**File or directory '{이름}' already exists at location. Please choose a different name.**". 빈 이름 문구("File or directory name cannot be empty.")는 코드에 있으나 조건 때문에 **나오지 않는다** (빈 이름은 배너 없이 무시·취소). 경고·오류는 행 아래 배너와 행 테두리 색으로 보인다 (01-LAYOUT P-020, P-021) | [project_panel.rs:2069-2141](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L2069-L2141) |

### 14.9 아직 조사하지 않은 것 (자리)

| 항목 | 규칙 | 확인할 곳 |
| --- | --- | --- |
| 프로젝트 패널 드래그 세부 (폴더 위에 머물면 자동 펼침, 가장자리 자동 스크롤, 금지 대상, 실패 메시지, 탐색기에서 끌어 놓기) | 미조사. 이동·복사는 5.2절 I-094 | `project_panel.rs` |

### 14.10 Picker·대화상자에서 확신 낮은 부분

- **조사 문서와 소스가 다른 곳 (소스를 따랐다):** 최근 폴더 열기 키 Ctrl+Alt+O는 키맵에서 주석 처리된 줄이고 실제 키는 **Ctrl-R**이다. 명령 팔레트·최근 폴더의 항목 제거 키는 Delete가 아니라 **Shift-Backspace**다. 상태 표시줄 커서 위치 글자색은 `text`다 (조사는 그렇게 적었고 이전 문서의 `text_muted`가 틀렸다).
- **테마 가정:** 조사의 글자색 해설은 기준 화면을 "Ayu Dark"라고 가정했다. 기준 화면은 Catppuccin Mocha·Latte라서 그 가정과 거기서 나온 색 값은 쓰지 않았고 토큰 이름만 썼다.
- **대화상자 방식:** OS TaskDialog 방식은 우리의 판단이다 (Zed 기본과 같다). 버튼 순서가 배열 순서 그대로인지, 인라인 코드 표기가 OS 창에서 백틱으로 보이는지는 확인하지 못했다.
- **행 높이:** 1줄 항목 약 30.65px은 줄 높이 22.652(14 × 1.618) + 8의 소스 계산이다. 실제 렌더 반올림과 2줄 항목 44px은 실행 대조 전이다.
- **명령 팔레트 푸터의 보조 확정 키:** 조사는 Alt-Enter라고 적었으나 해당 컨텍스트의 키 줄을 확인하지 못했다.
- **"This Window" 구역 (14.4):** 우리 앱의 두 구역 구성은 판단이다. Zed의 세 구역 중 "Current Folders"는 폴더가 하나뿐이라 뺐다.
- **창 모서리 (P-028):** 조사는 둥근 모서리를 8px로 적었으나 고정 커밋 상수는 10px이다 (01-LAYOUT L-311). 소스를 따랐다.
- **Latte 화면 측정색 (P-023):** 조사가 Latte 화면에서 잰 패널·비활성 탭 배경색은 설치된 테마 파일 값과 달라 쓰지 않았다 (01-LAYOUT L-313). 글자색 측정(`text` `#4c4f69`, `text_muted` `#5c5f77`, `hidden` `#9ca0b0`)은 테마 파일과 일치한다.
- **줄 번호:** 일부 링크의 줄 범위는 ±수 줄 차이가 날 수 있다.

## 15. 키맵 편집 화면

사용자 결정으로 포함한 기능이다 (Zed Keymap Editor와 같게). 키맵 편집 화면 조사(`zed-keymap-editor.md`)의 ID(K-001~K-097)를 그대로 쓴다. 화면 구성·치수·색은 [01-LAYOUT.md](01-LAYOUT.md) 4.14절이다. 키는 **적용 keymap(VSCode 덧씌움 후)** 기준이다. 이 화면의 키(`KeymapEditor`·`KeystrokeInput` 컨텍스트)는 VSCode 덧씌움이 바꾸지 않는다 (덧씌움 파일에 해당 컨텍스트가 없다). 표기는 메뉴·툴팁 형식(`Ctrl-K`, 8.4절)이다.

### 15.1 열기와 탭

| ID | 항목 | 내용 | 출처 |
| --- | --- | --- | --- |
| K-001 | 열기 | `zed::OpenKeymap` = **Ctrl-K Ctrl-S** (작업 영역). ☰ 앱 이름 메뉴의 Open Keymap(I-332)과 ∨ 메뉴의 Keymap(I-621)이 같은 동작이다 | [keymap:687](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L687) · [app_menus.rs:84](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L84) |
| K-002 | 탭 | 가운데 Pane의 **일반 탭**으로 연다. 탭 제목 "**Keymap Editor**". 세션 복원 대상이다 | [keymap_editor.rs:1975-1979](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1975-L1979) · [keymap_editor.rs:3955-4020](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3955-L4020) |
| K-003 | 키 컨텍스트 | 화면 전체 `KeymapEditor`, 키 입력칸 `KeystrokeInput`, 편집 모달 `KeybindEditorModal` | [keymap_editor.rs:1025](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1025) · [keystroke_input.rs:22](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L22) · [keymap_editor.rs:2994](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2994) |

### 15.2 표·정렬·행 상태

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| K-023 | Action 칸 | 사람이 읽는 액션 이름. hover 툴팁 = 액션 이름 + 메타 줄에 액션 설명 | [keymap_editor.rs:2180-2210](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2180-L2210) |
| K-024 | Arguments 칸 | JSON 인자를 구문 강조해 보인다. 인자가 없으면 흐린 글자 "**<no arguments>**", 값이 null이면 "**<null>**" | [keymap_editor.rs:57](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L57) · [keymap_editor.rs:2185-2230](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2185-L2230) |
| K-025 | Keystrokes 칸 | 툴팁과 같은 키 표시 형식(예 `Ctrl-K Ctrl-S`). 바인딩이 없는 액션은 빈 칸 | [keymap_editor.rs:2215-2225](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2215-L2225) |
| K-026 | Context 칸 | 컨텍스트 술어를 키 컨텍스트 문법으로 구문 강조. 컨텍스트가 없으면 "**<global>**". hover 툴팁에 전체 술어 | [keymap_editor.rs:1931-1940](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1931-L1940) · [keymap_editor.rs:2232-2260](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2232-L2260) |
| K-027 | Source 칸 | 바인딩 출처 이름: "**User**"(사용자 keymap.json) · "**Base**"(base_keymap 덧씌움 = 우리 앱의 VSCode) · "**Default**"(기본 키맵) · "Unknown". "Vim"은 우리 앱에 없다 (K-097) | [keymap_file.rs:1414-1436](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L1414-L1436) |
| K-028 | 행 목록 | **키가 묶인 모든 바인딩** + **키가 없는 모든 등록 액션**(빈 Keystrokes 행). 같은 액션의 바인딩이 여러 개면 행도 여러 개 | [keymap_editor.rs:835-930](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L835-L930) |
| K-029 | 정렬 (검색어가 비었을 때) | ① 바인딩 있는 행이 먼저, 없는 행이 뒤 ② 바인딩 행은 출처 순서 **User → Base → Default → Unknown**(원본은 User → Vim → Base → Default → Unknown) ③ 같은 출처 안에서는 액션 이름 순. 검색어가 있으면 퍼지 점수 순 | [keymap_editor.rs:1912-1928](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1912-L1928) · [keymap_editor.rs:812-820](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L812-L820) |
| K-034 | 해제된 바인딩 (unbind) | 클릭·우클릭 반응 없음. 툴팁 "**This action is unbound**" | [keymap_editor.rs:2280-2362](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2280-L2362) |
| K-033 | 가려진 바인딩 (사용자 충돌이 아님) | 행 불투명도 0.5. hover 툴팁 "**This keybinding is overridden by the '{액션}' binding from {출처}.**" (출처 = "your keymap" / "your base keymap"). 출처를 모르면 "This binding is overridden." | [keymap_editor.rs:2330-2358](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2330-L2358) |
| K-035 | 클릭 | 한 번 = 행 선택. **더블클릭 = 편집**. 우클릭 = 행 선택 + 컨텍스트 메뉴 (15.5) | [keymap_editor.rs:2290-2325](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2290-L2325) |
| K-036 | 아이콘 열 버튼 | 우선순위: ① unbind로 해제 → `Warning` 아이콘(Warning 색), 툴팁 "This action is unbound" ② 사용자 충돌 → `Warning` 아이콘, 툴팁 "**View conflicts**" + 메타 "**Use Alt+click to show all conflicts**". 클릭 = 충돌 필터, Alt+클릭 = 같은 키 바인딩 보기 ③ 가려짐 → `Info` 아이콘, 툴팁 "**Edit this binding**" 또는 "**Show matching keybinds**" + 메타 "**This binding is overridden by other bindings.**" ④ 그 밖 → `Pencil` 아이콘, 툴팁 "**Edit Keybinding**" + 키(Enter). 선택한 행은 항상, 다른 행은 hover 때만 보인다 | [keymap_editor.rs:1160-1255](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1160-L1255) · [keymap_editor.rs:2403-2407](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2403-L2407) |
| K-037 | hover 툴팁 억제 | 컨텍스트 메뉴가 열려 있으면 행·칸의 hover 툴팁과 아이콘 표시를 멈춘다 | [keymap_editor.rs:2195-2200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2195-L2200) |

### 15.3 검색·필터·정렬

| ID | 항목 | 내용 | 출처 |
| --- | --- | --- | --- |
| K-040 | 액션 이름 검색 | 입력칸 글자로 **사람이 읽는 액션 이름**을 퍼지 검색한다. Context·Arguments는 검색 대상이 아니다 | [keymap_editor.rs:733-760](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L733-L760) |
| K-041 | 키로 찾기 | ⌨ 버튼 또는 **Alt-F**: 키 검색 줄을 열고 키 입력칸에 녹화를 시작한다. 정확 일치 기본 **켜짐**. 정확 일치면 키 순서 전체가 같은 바인딩만, 끄면 입력한 키들이 바인딩 키 순서 안에 차례로 들어 있으면 일치 | [keymap_editor.rs:196-212](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L196-L212) · [keymap_editor.rs:769-805](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L769-L805) · [keymap_editor.rs:496](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L496) |
| K-042 | 필터 메뉴 (`Filter` 아이콘, 툴팁 "**Filters**", 버튼 오른쪽 위 기준) | 헤더 "**Filters**": 토글 "**Conflicts**"(Alt-C) · "**No Action**" / 구분선 / 헤더 "**Categories**": 토글 "**User**" · "**Default**"(출처별 표시). Zed의 "Vim" 토글은 뺀다 (K-097). 우리 앱은 Base(VSCode 덧씌움)를 **Default 토글에 묶는다** (판단, 원본 미확인 K-043) | [keymap_editor.rs:1635-1715](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1635-L1715) |
| K-043 | 기본 필터 | 충돌 필터 꺼짐(All), NoAction 바인딩 숨김. 출처 토글은 모두 켜짐 (판단) | [keymap_editor.rs:216-222](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L216-L222) · [keymap_editor.rs:806-809](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L806-L809) |
| K-044 | 결과 없음 문구 | 충돌 필터 + 검색어: "**No conflicting keybinds found that match the provided query**" / 충돌 필터만: "**No conflicting keybinds found**" / 키 검색: "**No keybinds found matching the entered keystrokes**" / 일반: "**No matches found for the provided query**" | [keymap_editor.rs:1258-1272](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1258-L1272) |
| K-045 | 충돌 정의 | **같은 키 순서 + 같은 컨텍스트**의 바인딩이 둘 이상이면 충돌 후보. 출처 우선순위(User > Base > Default)가 높은 쪽이 낮은 쪽을 가린다 (K-033). 둘 다 User면 서로 충돌 (K-032, 행 배경 오류색) | [keymap_editor.rs:264-375](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L264-L375) |

### 15.4 키 입력 기록 (KeystrokeInput)

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| K-054 | 키 | **Enter** = 녹화 시작, **Escape Escape Escape**(Esc 세 번) = 녹화 끝, **Delete** = 입력한 키 지우기 (`KeystrokeInput` 컨텍스트) | [keymap:1476-1483](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1476-L1483) |
| K-055 | 녹화 규칙 | 키 입력을 그대로 받아 적는다(다른 단축키를 실행하지 않는다). 최대 **3타**, 넘으면 자동으로 녹화를 끝낸다. 수식키만 누른 상태도 표시한다. 마지막 Esc들이 끝내기 키(Esc×3)의 일부일 수 있어 끝낼 때 **300ms** 안의 그 Esc들은 기록에서 뺀다. 포커스를 잃으면 녹화를 멈춘다 (추정) | [keystroke_input.rs:24-71](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L24-L71) · [keystroke_input.rs:147-345](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L147-L345) · [keystroke_input.rs:436-450](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L436-L450) |
| K-056 | placeholder | 편집할 때 키를 아직 누르지 않았으면 기존 바인딩 키를 자리 표시로 보인다. 녹화를 시작하면 사라진다 | [keystroke_input.rs:108-118](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L108-L118) · [keystroke_input.rs:362-372](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L362-L372) |

### 15.5 행 우클릭 메뉴

우클릭 메뉴 공통 모양은 01-LAYOUT 4.9절(I-501~I-515)이다. 유지 항목만 남기는 규칙에 해당하는 제외 항목은 없다 (7개 모두 우리 앱에 있다).

```text
Create                       Alt-Enter     (키가 없는 액션 행일 때만)
Edit                         Enter         (키 없는 행이면 비활성)
Delete                                      (키 없는 행이면 비활성)
──────────────
Copy Action                  Ctrl-C
Copy Context                 Ctrl-Shift-C  (컨텍스트가 없으면 비활성)
──────────────
Show Matching Keybindings    Ctrl-T        (키 없는 행이면 비활성)
```

| ID | 항목 | 액션 | 키 | 조건 | 출처 |
| --- | --- | --- | --- | --- | --- |
| K-060 | Create | `CreateBinding` | Alt-Enter | 키가 없는 액션 행일 때만 | [keymap_editor.rs:1108-1112](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1108-L1112) |
| K-061 | Edit | `EditBinding` | Enter | 키 없는 행이면 비활성 | [keymap_editor.rs:1112-1118](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1112-L1118) |
| K-062 | Delete | `DeleteBinding` | 없음 | 키 없는 행이면 비활성 | [keymap_editor.rs:1118-1122](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1118-L1122) |
| K-063 | Copy Action | `CopyAction` | Ctrl-C | 항상 | [keymap_editor.rs:1124](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1124) |
| K-064 | Copy Context | `CopyContext` | Ctrl-Shift-C | 컨텍스트가 없으면 비활성 | [keymap_editor.rs:1125-1129](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1125-L1129) |
| K-065 | Show Matching Keybindings | `ShowMatchingKeybinds` | Ctrl-T | 키 없는 행이면 비활성. 같은 키 순서로 키 검색을 연다 | [keymap_editor.rs:1130-1136](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1130-L1136) |

### 15.6 키 (KeymapEditor 컨텍스트)

| ID | 키 | 동작 | 출처 |
| --- | --- | --- | --- |
| K-070 | Ctrl-F | 검색 입력칸에 포커스 | [keymap:1460-1474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1460-L1474) |
| K-071 | Alt-F | 키로 찾기 토글 (툴팁 "**Search by Keystrokes**") | [keymap:1460-1474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1460-L1474) · [keymap_editor.rs:2071-2090](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2071-L2090) |
| K-072 | Alt-C | 충돌 필터 토글 | [keymap:1460-1474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1460-L1474) |
| K-073 | Enter / Alt-Enter | 선택 행 편집 / 선택한 액션에 새 바인딩 추가 | [keymap:1460-1474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1460-L1474) |
| K-074 | Ctrl-K | 새 키바인딩 만들기 모달 ("Create Keybinding" 버튼과 같음) | [keymap:1460-1474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1460-L1474) |
| K-075 | Ctrl-C / Ctrl-Shift-C | 액션 이름 복사 / 컨텍스트 복사 | [keymap:1460-1474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1460-L1474) |
| K-076 | Ctrl-T | 같은 키 바인딩 보기 | [keymap:1460-1474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1460-L1474) |
| K-077 | Ctrl-E | keymap.json 열기 (`zed::OpenKeymapFile`, "Edit in JSON" 버튼과 같음. ☰ Open Keymap File I-333) | [keymap:1460-1474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1460-L1474) |
| K-078 | 정확 일치 토글 | `ToggleExactKeystrokeMatching` (기본 키 없음), 툴팁 "**Toggle Exact Match Mode**" | [keymap_editor.rs:1982-1995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1982-L1995) |
| K-079 | 위·아래 등 | 표 선택 이동은 목록 선택 키(Up/Down/Home/End/PageUp/PageDown) (추정: 표가 목록 선택 액션을 받음) | [keymap:5-21](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L5-L21) |
| K-084 | 모달 키 | **Ctrl-Enter = 저장**, **Escape = 취소** (`KeybindEditorModal`) | [keymap:1485-1491](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1485-L1491) |

### 15.7 바인딩 편집·추가 모달

화면 치수는 01-LAYOUT 4.14.4절(K-080~K-083)이다.

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| K-081 | 새로 만들기 | Ctrl-K 또는 "Create Keybinding" 버튼: 라벨 "**Create Keybinding**" + 액션 입력칸(라벨 "**Action**", placeholder "**Type an action name**", 액션 이름 자동완성) | [keymap_editor.rs:3090-3125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3090-L3125) · [action_completion_provider.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/action_completion_provider.rs) |
| K-082 | 본문 문구 | "**Edit Keystroke**" + 키 입력칸. 같은 키의 바인딩이 있으면 "**There is 1 binding with the same keystrokes.**" / "There are N bindings …" + "**View**" 버튼. 인자가 있는 액션이면 "**Edit Arguments**"(placeholder "**Action Arguments**"). 컨텍스트 칸 "**Edit Context**"(placeholder "**Keybinding Context**") | [keymap_editor.rs:3124-3200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3124-L3200) · [keymap_editor.rs:3415-3495](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3415-L3495) |
| K-085 | 검증 오류 문구 | "**Action name is required**" · "**Action '{이름}' not found**" · "**Failed to parse action arguments as JSON**" · "**Failed to validate action arguments**" · "**Keystrokes cannot be empty**" · "**Failed to parse key context**" | [keymap_editor.rs:2740-2795](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2740-L2795) |
| K-086 | 충돌 경고 | 저장 전 같은 키·컨텍스트를 확인한다: "**Your keybind would conflict with the "{액션}" action**", 여러 개면 "… action and {N} other bindings", 이름을 모르면 "**Your keybind would conflict with other actions**". 경고를 보인 뒤 다시 Save를 누르면 저장한다 (추정) | [keymap_editor.rs:2835-2860](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2835-L2860) |
| K-087 | 저장 뒤 | 성공하면 모달을 닫고 토스트 "**Saved edits to the {액션} action.**"(체크 아이콘)를 보인다. 수정·추가한 행을 다시 찾아 선택한다 | [keymap_editor.rs:2910-2930](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2910-L2930) · [keymap_editor.rs:469](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L469) |

### 15.8 keymap.json 저장 규칙

| ID | 항목 | 내용 | 출처 |
| --- | --- | --- | --- |
| K-090 | 저장 파일 | Zed는 `%APPDATA%\Zed\keymap.json` (백업 이름 `keymap_backup.json`). **우리 앱은 `%APPDATA%\{앱 이름}\keymap.json`** (판단, 설정 파일과 같은 폴더) | [paths.rs:122-135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/paths/src/paths.rs#L122-L135) · [paths.rs:296-305](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/paths/src/paths.rs#L296-L305) |
| K-091 | 쓰기 방식 | 파일을 읽어 해당 바인딩만 고친 새 내용을 만들고 파일 전체를 다시 쓴다. 읽을 수 없거나 형식이 틀린 파일은 고치지 않는다. 오류 문구: "**Failed to load keymap file**", "**Could not save updated keybinding**", "**Failed to write keymap file**" | [keymap_editor.rs:3640-3705](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3640-L3705) · [keymap_file.rs:929-940](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L929-L940) |
| K-092 | User 바인딩 수정·삭제 | keymap.json 안의 그 항목을 바꾸거나 지운다 | [keymap_file.rs:895-940](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L895-L940) |
| K-093 | Default·Base 바인딩 수정 | 기본 파일은 고치지 않는다. **새 사용자 바인딩을 추가**하고, 원래 바인딩을 끄는 **unbind 항목**을 같은 섹션의 `unbind` 객체(키 → 액션)에 쓴다 | [keymap_file.rs:895-920](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L895-L920) · [keymap_file.rs:77-80](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L77-L80) |
| K-094 | Default·Base 바인딩 삭제 | unbind 항목만 추가한다 (그 행은 K-034의 "unbound" 상태가 된다) | [keymap_file.rs:918-928](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L918-L928) |
| K-095 | 없는 바인딩 삭제 | 오류 "**Cannot remove a keybinding that does not exist**" | [keymap_editor.rs:3720-3723](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3720-L3723) |
| K-096 | 반영 | keymap.json이 바뀌면 키맵을 다시 읽어 표가 갱신된다. 메뉴·툴팁 단축키도 새 바인딩을 따른다 (사용자 바인딩이 우선, 8.4절 L-217, 1.1절 I-704) | [keymap_editor.rs:835-930](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L835-L930) |
| K-097 | 우리 앱 범위 (판단) | 표의 행은 **우리 앱이 등록한 액션과 그 바인딩만** 보인다 (제외 기능의 액션은 등록하지 않으므로 자연히 빠진다). 출처 "Vim"은 없다. "Base"는 VSCode 덧씌움이다 | `_decisions.md` |

### 15.9 툴팁

문구는 화면에 보이는 짧은 UI 문자열이다. 키는 8.4절 형식이다.

| ID | 요소 | 툴팁 문구 | 출처 |
| --- | --- | --- | --- |
| — | (참조 K-071) 키로 찾기 버튼 (⌨) | **Search by Keystrokes** + Alt-F | [keymap_editor.rs:2071-2090](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2071-L2090) |
| — | (참조 K-042) 필터 버튼 | **Filters** | [keymap_editor.rs:1635-1715](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1635-L1715) |
| — | (참조 K-078) 정확 일치 토글 | **Toggle Exact Match Mode** | [keymap_editor.rs:1982-1995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1982-L1995) |
| — | (참조 K-053) 키 입력칸 오른쪽 버튼 | 녹화 중 **Stop Recording** (키 검색이면 **Stop Searching**). 평소 **Start Recording** / **Start Searching** | [keystroke_input.rs:585-620](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L585-L620) |
| — | (참조 K-036) 아이콘 열 버튼 | 15.2절 (View conflicts, Edit this binding, Edit Keybinding + Enter, This action is unbound 등) | 15.2절 |
| K-033, K-034 | 행·칸 hover | 가려진 바인딩, 해제된 바인딩 문구는 15.2절. 컨텍스트 메뉴가 열려 있으면 모두 숨긴다 (K-037) | 15.2절 |

### 15.10 키맵 편집 화면에서 확신 낮은 부분

- **출처 토글 (K-042, K-043):** 원본에서 "Base" 출처가 어느 토글에 묶이는지 확인하지 못했다. 우리 앱은 Default 토글에 묶는 판단을 했다.
- **충돌 경고 뒤 저장 (K-086):** 두 번째 Save로 저장되는지는 코드 흐름에서 추정했다.
- **녹화 중 포커스 잃음 (K-055):** 녹화를 멈추는지는 추정이다. 입력 중 IME 처리도 확인하지 않았다.
- **표 키 탐색 (K-079):** 표가 목록 선택 액션을 받는다는 것은 추정이다.
- **액션 이름 사람 읽기 규칙 (K-023):** 명령 팔레트(14절)와 같은 정규화인지는 Picker 조사와 함께 확인해야 한다.
- **줄 번호:** K-023~K-037, K-080~K-087의 링크는 ±수십 줄 차이가 날 수 있다.

## 16. 확장 관리 화면 (Extensions)

사용자 지시로 포함한 기능이고, 사용자 결정(`_decisions.md` "확장 세부")에 따라 **로컬 설치만** 한다. 지원 범위는 **테마·아이콘 테마·언어(구문 강조) 확장 3종**이다. LSP·디버거·AI 슬래시 명령·MCP·스니펫·언어 모델·에이전트 서버·문서 인덱서 확장은 만들지 않는다 (X-13~X-15). 온라인 목록·검색·내려받기·자동 업데이트(레지스트리 서버, `api.zed.dev`, 미러)는 **없다.** 확장을 끄는 기능도 **없다** (설치돼 있으면 켜져 있다, Zed와 같다). 확장 조사(`zed-extensions.md`)의 ID(X-36~X-56)를 그대로 쓴다. 화면 치수는 [01-LAYOUT.md](01-LAYOUT.md) 4.15절이다. 키는 적용 keymap(VSCode 덧씌움 후) 기준이다. 확장 패키지 형식(`extension.toml`, 폴더 구조, 스키마, X-01~X-24)은 [04-APP-SPEC.md](04-APP-SPEC.md)가 다룬다. 내장 테마·아이콘 테마(One Dark, One Light, Catppuccin 4종, Zed (Default) 아이콘, Catppuccin Icons)는 본체에 있어 이 화면에 나오지 않는다.

### 16.1 진입 경로

| ID | 진입점 | 규칙 | 출처 |
| --- | --- | --- | --- |
| X-44 | 키 | **Ctrl-Shift-X** (`zed::Extensions`, 작업 영역). VSCode 덧씌움이 바꾸지 않는다 | [keymap:717](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L717) · [vscode.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/linux/vscode.json) |
| — | ☰ 메뉴 | 앱 이름 메뉴의 **Extensions** (9.2절 I-339) | 9.2절 |
| — | ∨ 메뉴 | **Extensions** (10.2절 I-624) | 10.2절 |
| — | 명령 팔레트 | "zed: extensions"에 해당하는 명령. 개발 확장 명령은 16.5절 | [extensions_ui.rs:119-169](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L119-L169) |
| X-60 | 테마 선택기 푸터 | 테마 선택기의 "**Install Themes**", 아이콘 테마 선택기의 "**Install Icon Themes**" 버튼은 이 화면을 **카테고리 필터(Themes / Icon Themes)가 선택된 채로** 연다 (14.5절). 온라인 설치가 없으므로 이 화면에서 "Install Extension…"으로 로컬 설치를 한다 | [theme_selector.rs:530-569](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L530-L569) · [icon_theme_selector.rs:303-360](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L303-L360) |
| — | 탭 | 한 Pane에 이 탭이 하나만 있다. 이미 있으면 활성화한다. 탭 제목 "Extensions" (01-LAYOUT X-44) | [extensions_ui.rs:1640-1666](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L1640-L1666) |

### 16.2 목록·검색·필터·빈 상태

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| X-46 | 검색 | 설치된 확장을 **입력하는 즉시** 퍼지 검색으로 거른다. `id:<확장 ID>` 형식이면 그 ID만 일치시킨다. 원격 조회와 250ms 디바운스는 없다 | [extensions_ui.rs:535-592](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L535-L592) · [extensions_ui.rs:862-897](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L862-L897) |
| X-48 | 카테고리 필터 | **All**, **Themes**, **Icon Themes**, **Languages**. 확장이 제공하는 기능으로 거르고 검색어와 함께 적용한다 | [extensions_ui.rs:1550-1599](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L1550-L1599) |
| X-47 | 설치 상태 필터 | **만들지 않는다.** 목록이 설치된 확장뿐이다 | [extensions_ui.rs:1502-1547](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L1502-L1547) |
| X-49 | 빈 상태 문구 | 결과가 없을 때 "**No installed extensions**". 검색어가 있으면 끝에 " that match \"{검색어}\""를 붙인다. 이 화면은 Zed의 "Installed" 필터가 선택된 상태와 같아서 그 문구를 쓴다 (판단). "Loading extensions..."은 없다 | [extensions_ui.rs:899-934](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L899-L934) |
| 〃 | 정렬 | 이름 오름차순 (판단: Zed는 원격 목록을 다운로드 수로 정렬하고 설치 목록의 정렬은 확인하지 못했다) | — |

### 16.3 카드의 버튼과 툴팁

| ID | 상태 | 버튼 | 툴팁·설명 | 출처 |
| --- | --- | --- | --- | --- |
| X-50 | 설치됨 (일반) | "**Uninstall**" (`OutlinedGhost`) | — | [extensions_ui.rs:688-859](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L688-L859) · [extension_card.rs:626-775](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/components/extension_card.rs#L626-L775) |
| — | 설치됨 (개발 확장) | "**Rebuild**" (Accent 색, 다시 읽는 중 "**Rebuilding...**") + "**Uninstall**" (Accent 색) | — | 위와 같음 |
| — | 저장소 버튼 | 저장소 주소가 있으면 아이콘 버튼(Git 공급자 아이콘, 없으면 `Link`) | 툴팁 "**Visit Extension Repository**" + 주소. 누르면 브라우저로 연다 | 위와 같음 |
| — | 기능 배지 | 카드 이름 옆에 "Themes", "Icon Themes", "Languages" 중 확장이 제공하는 것 | — | 위와 같음 |
| — | 만들지 않는 것 | Install·Installing...·Upgrade 버튼과 "not compatible" 툴팁, "Downloads: n", ⋯ 메뉴("Install Another Version...", "Copy Extension ID", "Copy Author Info"), "Overridden by dev extension." 덮개 | 온라인 카드 전용 | 위와 같음 |

### 16.4 로컬 설치·제거 동작

| ID | 항목 | 규칙 | 출처 |
| --- | --- | --- | --- |
| X-43 | 로컬 설치 ("Install Extension…") | 버튼을 누르면 **확장 폴더**(안에 `extension.toml`) 또는 **보관 파일**(`.tar.gz`, `.zip`; 판단: Zed 패키지가 `.tar.gz`이고 Windows에서 흔한 `.zip`도 받는다)을 고르는 대화상자를 연다. ① 매니페스트를 읽고 `schema_version`이 우리 앱이 아는 값(1) 이하인지 확인한다 ② 임시 폴더(`staging`)에 풀거나 복사한다 ③ 같은 `installed/<ID>`가 일반 확장으로 이미 있으면 지우고(**교체**), 개발 확장이면 중복 오류 ④ `installed/<ID>`로 **한 번에 옮긴다** ⑤ 확장 목록을 다시 읽는다. 내려받기와 업데이트 확인은 없다 | [extension_host.rs:851-973](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L851-L973) · [extension_host.rs:1119-1149](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L1119-L1149) |
| X-41 | 제거 (비활성화 없음) | 확장별 켜기·끄기는 **없다.** 끄려면 **제거**한다. 제거: ① `installed/<ID>` 폴더를 지운다 ② 목록을 다시 읽어 테마·아이콘 테마·언어·문법을 **즉시 내리고** 색인을 갱신한다 ③ `work/<ID>`를 지운다 (Windows에서 프로세스 종료가 늦을 수 있어 0·100·200ms 간격으로 최대 3번 다시 시도) ④ "확장 제거됨" 이벤트 | [extension_host.rs:1040-1117](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L1040-L1117) |
| X-36, X-37 | 시작·색인 | 앱을 시작할 때 `index.json`을 빠르게 읽어 바로 등록한다. 색인이 없거나 깨졌거나 `installed/` 폴더가 색인보다 **더 최근에 바뀌었으면** 폴더를 훑어 색인을 다시 만든다 | [extension_host.rs:416-461](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L416-L461) · [extension_host.rs:1754-1952](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L1754-L1952) |
| X-39 | 폴더 감시 | `installed/`를 100ms 지연으로 감시하고, 바뀌면 **200ms 디바운스** 뒤에 목록을 다시 읽는다. 그래서 확장 폴더를 직접 넣거나 지워도 반영된다 | [extension_host.rs:78-79](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L78-L79) · [extension_host.rs:480-535](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L480-L535) |
| X-51 | 설치 직후 선택기 | 이 화면에서 확장을 설치하면, 그 확장이 **테마**를 제공하면 곧바로 **테마 선택기**를 그 확장의 테마만 보이게 열고, 테마는 없고 **아이콘 테마**가 있으면 **아이콘 테마 선택기**를 그 확장의 항목만 보이게 연다 (14.5절의 미리 적용·저장 규칙을 따른다) | [extensions_ui.rs:487-531](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L487-L531) |
| X-57, X-58 | 내장과 확장의 합류 | 내장 테마·내장 아이콘 테마와 확장이 제공하는 것은 **같은 목록**에 이름을 키로 들어간다. 선택기에서 구별하지 않는다. 확장 아이콘 테마가 정의하지 않은 파일·폴더는 **내장 기본 아이콘 매핑으로 채운다** | [registry.rs:61-82](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/registry.rs#L61-L82) · [registry.rs:296-318](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/registry.rs#L296-L318) |
| X-59 | 시작 때 폴백 | 설정에 확장 테마가 지정돼 있어도 확장 로드가 끝나기 전에는 내장 기본 테마로 임시 표시했다가, 로드가 끝나면 설정한 테마로 **즉시 바꾸고 전체를 다시 그린다** | [theme_settings.rs:171-249](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/theme_settings.rs#L171-L249) · [extension_host.rs:1739-1741](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L1739-L1741) |
| X-30, X-31 | 설치 위치 (판단) | Zed는 `%LOCALAPPDATA%\Zed\extensions\installed\<ID>`. 우리 앱은 `%LOCALAPPDATA%\{앱 이름}\extensions\installed\<ID>` (그 안의 `staging`, `work`, `index.json` 이름도 같게). 정확한 경로는 04-APP-SPEC | [paths.rs:58-65](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/paths/src/paths.rs#L58-L65) · [paths.rs:122-166](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/paths/src/paths.rs#L122-L166) |
| X-42 | 자동 설치·업데이트 | **없다.** 온라인 출처가 없으므로 `auto_install_extensions`, `auto_update_extensions`는 만들지 않는다 | [extension_settings.rs:9-49](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_settings.rs#L9-L49) |

### 16.5 개발용 확장 설치 (Install Dev Extension)

| ID | 단계 | 규칙 | 출처 |
| --- | --- | --- | --- |
| X-53 | 진입 | 화면 오른쪽 위 **[Install Dev Extension]** 버튼 또는 명령 팔레트("zed: install dev extension"). **폴더 선택 대화상자**(폴더 하나)를 연다. "Rebuild"는 카드 버튼 또는 명령 팔레트("zed: rebuild dev extension")이고, **설치된 개발 확장이 하나도 없으면 명령 팔레트에서 숨긴다** | [extensions_ui.rs:48-115](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L48-L115) · [extensions_ui.rs:254-320](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L254-L320) |
| X-54 | 1. 매니페스트 | 고른 폴더에서 `extension.toml`을 읽는다. 같은 `id`의 **일반 확장이 이미 설치돼 있으면 먼저 제거**한다. 이미 개발 확장으로 설치돼 있으면 중복 오류 | [extension_host.rs:1119-1149](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L1119-L1149) |
| X-55 | 2. 빌드 | **테마·아이콘 테마 확장은 빌드가 없다** (외부 도구 없이 바로 끝난다). 언어 확장은 `[grammars]`가 있으면 Tree-sitter 문법을 `grammars/<이름>.wasm`으로 컴파일해야 한다 (`wasi-sdk` 자동 내려받기). **우리 앱은 미리 컴파일된 `grammars/*.wasm`이 이미 있는 확장만 개발 확장으로 설치한다. 문법 소스 빌드와 Rust `extension.wasm` 빌드는 지원하지 않는다** (판단: LSP 제외, 도구·네트워크 부담) | [extension_builder.rs:53-268](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension/src/extension_builder.rs#L53-L268) · [extension_host.rs:1178-1197](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L1178-L1197) |
| X-56 | 3. 연결·리로드 | 성공하면 `installed/<ID>`를 **개발자의 원본 폴더를 가리키는 심볼릭 링크**로 만들고 목록을 다시 읽는다. 링크이면 색인에 `"dev": true`로 적는다. 이후 **테마·아이콘 테마 JSON이나 `.scm` 쿼리를 고치면 다시 만들지 않아도 감시(X-39)가 반영한다.** 문법(`.wasm`)을 바꿨을 때는 카드의 "Rebuild"를 눌러 다시 읽는다 (우리 앱의 Rebuild = 매니페스트와 색인을 다시 읽고 등록을 갱신) | [extension_host.rs:1199-1254](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L1199-L1254) · [extension_host.rs:1817-1821](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L1817-L1821) |

### 16.6 확장 관리 화면에서 확신 낮은 부분

- **조사하지 않은 것:** 카드·화면의 키보드 조작(Tab 순서, 카드 포커스), 카테고리 바 가로 스크롤 중 선택 이동, 설치 실패 때의 오류 문구와 토스트, 설치 목록의 정렬 순서는 조사에 없다. 로컬 설치의 보관 파일 형식(`.tar.gz`·`.zip`)과 교체 때 확인 창이 없는 것은 우리 판단이다.
- **개발 확장의 심볼릭 링크 (X-56):** Windows에서 심볼릭 링크를 만들려면 개발자 모드나 관리자 권한이 필요할 수 있다. Zed가 어떻게 처리하는지는 확인하지 못했다. 만들지 못하면 폴더를 복사하되 "dev"로 표시하는 대체안을 04-APP-SPEC에서 정해야 한다.
- **언어 확장 설치:** 언어 확장 패키지가 `[language_servers]`와 `extension.wasm`을 함께 가지고 있어도 우리 앱은 그 필드를 **오류 없이 무시하고** `languages/*`와 `grammars/*.wasm`만 쓴다 (X-29, 04-APP-SPEC). 내장 언어와 이름이 겹치는 확장 언어는 등록이 거부된다 (X-38).
- **줄 번호:** 일부 링크의 줄 범위는 ±수 줄 차이가 날 수 있다.


---

이전: [01-LAYOUT.md](01-LAYOUT.md) · 다음: [03-THEME-ICONS.md](03-THEME-ICONS.md) · 입구: [../README.md](../README.md)
