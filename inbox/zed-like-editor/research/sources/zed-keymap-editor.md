# Zed 키맵 편집 화면 (Keymap Editor) (ID 접두사 K-)

- 담당: scouter-opus · 조사일 2026-10-31 · 2차 보충 조사 (사용자 결정: 키맵 편집 화면을 Zed와 같게, Ctrl-K Ctrl-S)
- 소스 기준: zed-industries/zed 커밋 `14b7b0ef3764d62a85070c8b5ccea105a2ad3235`. 표의 `KE` = [crates/keymap_editor/src/keymap_editor.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs), `KI` = [crates/keymap_editor/src/ui_components/keystroke_input.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs), `KF` = [crates/settings/src/keymap_file.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs), `KM` = [assets/keymaps/default-windows.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json)이다. 줄 번호는 `#L줄`로 붙였다. keymap_editor 크레이트는 GPL이므로 원문은 옮기지 않았고, 값·이름·동작과 화면 문구만 적었다.
- 키 표기는 Zed 메뉴·툴팁 형식(`Ctrl-K`, L-219)이다. base_keymap=VSCode 덧씌움은 이 화면의 키를 바꾸지 않는다 (`linux/vscode.json`에 KeymapEditor 컨텍스트가 없음, zed-panes-keymap.md 6절).

---

## 1. 열기와 탭

| ID | 항목 | 내용 | 출처 |
|---|---|---|---|
| K-001 | 열기 | `zed::OpenKeymap` = **Ctrl-K Ctrl-S** (Workspace). ☰ 메뉴 Zed > Settings > Open Keymap, ∨ 메뉴 Keymap 항목도 같은 액션 | [KM#L687](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L687), [app_menus.rs#L84](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L84) |
| K-002 | 탭 | 가운데 Pane의 **일반 탭**으로 연다. 탭 제목 "**Keymap Editor**". 세션 복원 대상이다(`KeybindingEditorDb`) | [KE#L1975-L1979](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1975-L1979), [KE#L3955-L4020](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3955-L4020) |
| K-003 | 키 컨텍스트 | 화면 전체 = `KeymapEditor`, 키 입력칸 = `KeystrokeInput`, 편집 모달 = `KeybindEditorModal` | [KE#L1025](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1025), [KI#L22](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L22), [KE#L2994](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2994) |

## 2. 화면 구성

```
┌ Keymap Editor (p 8, gap 4, 배경 editor_background) ─────────────────────────────┐
│ [검색 입력 (h 32, px 8, 테두리 1 border, rounded_md) ] [⌨][⚲ 필터] [Edit in JSON][Create Keybinding] │
│ (키 검색 모드일 때) [키 입력칸 KeystrokeInput (최소 높이 40)] [Aa 정확 일치] [빈 칸 min_w 320]     │
├──────┬──────────────┬──────────────┬────────────┬──────────────────────┬────────┤
│      │ Action       │ Arguments    │ Keystrokes │ Context              │ Source │
├──────┼──────────────┼──────────────┼────────────┼──────────────────────┼────────┤
│ ✎/⚠/ⓘ│ Go Back      │ <no arguments>│ Alt-Left  │ Pane                 │ Default│  ← 줄무늬
└──────┴──────────────┴──────────────┴────────────┴──────────────────────┴────────┘
```

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| K-010 | 바깥 상자 | 안쪽 여백 `p_2` **8px**, 세로 gap `_1` **4px**, 배경 `editor_background` | [KE#L2018-L2035](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2018-L2035) |
| K-011 | 위쪽 줄 | 가로, gap `_2` 8px: [검색 입력 상자] [버튼 묶음] | [KE#L2035-L2130](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2035-L2130) |
| K-012 | 검색 입력 상자 | 높이 `h_8` **32px**, 좌우 padding 8, 테두리 1px `border`, 모서리 `rounded_md`(6px), 남는 폭을 모두 차지. 키 컨텍스트에 `BufferSearchBar`를 더해 찾기 창과 같은 키(Esc 등)를 쓴다. placeholder "**Filter action names…**" | [KE#L2040-L2058](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2040-L2058), [KE#L585](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L585) |
| K-013 | 버튼 묶음 | gap 4, **최소 폭 `min_w_80`(20rem = 320px)**. 순서: ① **키로 찾기**(`IconName::Keyboard`, Small, 키 검색 모드면 선택 상태) ② **필터**(`IconName::Filter`, 메뉴) ③ **Edit in JSON** 버튼 ④ **Create Keybinding** 버튼. ③④는 오른쪽에 단축키를 작게(10px) 붙인다 | [KE#L2059-L2125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2059-L2125), [KE#L1700-L1715](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1700-L1715) |
| K-014 | 키 검색 줄 | 키 검색 모드에서만 위쪽 줄 아래에 나타난다: [키 입력칸 KeystrokeInput (K-040)] [정확 일치 토글 `IconName::CaseSensitive`, 정사각] [빈 칸 min_w 320, 위 버튼 묶음과 오른쪽 끝 정렬] | [KE#L1980-L1995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1980-L1995), [KE#L2125-L2135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2125-L2135) |

### 2.1 표

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| K-020 | 표 컴포넌트 | 6열, **줄무늬**(`striped`), 가상 목록(행 수만큼). 열 폭은 **끌어서 재분배**(redistributable) | [KE#L2136-L2155](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2136-L2155), [KE#L58](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L58) |
| K-021 | 머리글 | "" (아이콘 열) · "**Action**" · "**Arguments**" · "**Keystrokes**" · "**Context**" · "**Source**" | [KE#L2152](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2152) |
| K-022 | 처음 열 폭 | 아이콘 열 **36px 고정**, 나머지는 비율 Action **0.25** · Arguments **0.20** · Keystrokes **0.14** · Context **0.45** · Source **0.08** | [KE#L653-L664](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L653-L664) |
| K-023 | Action 칸 | 사람이 읽는 액션 이름(`humanized_name`, 예: `go back`을 보기 좋게 바꾼 이름). hover 툴팁 = 액션 이름 + 메타 줄에 액션 설명(문서 주석) | [KE#L2180-L2210](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2180-L2210) |
| K-024 | Arguments 칸 | JSON 인자를 구문 강조해 보인다. 인자가 없으면 흐린 글자 "**<no arguments>**", 값이 null이면 "**<null>**" | [KE#L57](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L57), [KE#L2185-L2230](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2185-L2230) |
| K-025 | Keystrokes 칸 | `ui::KeyBinding` 표시(툴팁과 같은 형식, 예: `Ctrl-K Ctrl-S`). 바인딩이 없는 액션은 빈 칸 | [KE#L2215-L2225](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2215-L2225) |
| K-026 | Context 칸 | 컨텍스트 술어를 "Zed Keybind Context" 문법으로 구문 강조. 컨텍스트가 없으면 "**<global>**". hover 툴팁에 전체 술어 | [KE#L1931-L1940](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1931-L1940), [KE#L2232-L2260](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2232-L2260) |
| K-027 | Source 칸 | 바인딩 출처 이름: "**User**"(사용자 keymap.json) · "**Vim**" · "**Base**"(base_keymap 덧씌움 = 우리 앱의 VSCode) · "**Default**"(기본 키맵) · "Unknown" | [KF#L1414-L1436](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L1414-L1436) |
| K-028 | 행 목록 | **키가 묶인 모든 바인딩** + **키가 없는 모든 등록 액션**(빈 Keystrokes 행). 같은 액션의 바인딩이 여러 개면 행도 여러 개 | [KE#L835-L930](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L835-L930) |
| K-029 | 정렬 (검색어가 비었을 때) | ① 바인딩 있는 행이 먼저, 없는 행이 뒤 ② 바인딩 행은 출처 순서 **User → Vim → Base → Default → Unknown** ③ 같은 출처 안에서는 액션 이름 순. 검색어가 있으면 퍼지 점수 순 | [KE#L1912-L1928](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1912-L1928), [KE#L812-L820](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L812-L820) |

### 2.2 행 상태

| ID | 상태 | 모양·동작 | 출처 |
|---|---|---|---|
| K-030 | 기본 | 행 테두리 `border_2`(2px, 평소 색은 보이지 않음, 추정), 줄무늬 배경 | [KE#L2364-L2372](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2364-L2372) |
| K-031 | 선택 | 테두리 색 **`panel_focused_border`** | [KE#L2369-L2372](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2369-L2372) |
| K-032 | 사용자 바인딩끼리 충돌 | 행 배경 **`status().error_background`** | [KE#L2365-L2368](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2365-L2368) |
| K-033 | 다른 바인딩에 가려짐 (사용자 충돌이 아님) | 행 **불투명도 0.5**. hover 툴팁 "**This keybinding is overridden by the '{액션}' binding from {출처}.**" (출처 = "your keymap" / "the vim keymap" / "your base keymap"). 출처를 모르면 "This binding is overridden." | [KE#L2330-L2358](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2330-L2358) |
| K-034 | unbind로 해제된 바인딩 | 클릭·우클릭 반응 없음. 툴팁 "**This action is unbound**" | [KE#L2280-L2362](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2280-L2362) |
| K-035 | 클릭 | 한 번 = 행 선택. **더블클릭 = 편집**(EditBinding). 우클릭 = 행 선택 + 컨텍스트 메뉴(K-060) | [KE#L2290-L2325](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2290-L2325) |
| K-036 | 아이콘 열 버튼 | `ButtonSize::Compact`(18px), 정사각. 우선순위: ① unbind로 해제 → `Warning` 아이콘(Warning 색), 툴팁 "This action is unbound" ② 사용자 충돌 → `Warning` 아이콘(Warning 색), 툴팁 "**View conflicts**" + 메타 "**Use Alt+click to show all conflicts**". 클릭 = 충돌 필터, Alt+클릭 = 같은 키 바인딩 보기 ③ 가려짐 → `Info` 아이콘, 툴팁 "**Edit this binding**" 또는 "**Show matching keybinds**" + 메타 "**This binding is overridden by other bindings.**"(Alt+클릭 안내 포함) ④ 그 밖 → `Pencil` 아이콘, 툴팁 "**Edit Keybinding**" + 키(Enter). **선택한 행은 항상, 다른 행은 hover 때만 보인다** | [KE#L1160-L1255](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1160-L1255), [KE#L2403-L2407](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2403-L2407) |
| K-037 | hover 툴팁 억제 | 컨텍스트 메뉴가 열려 있으면 행·칸의 hover 툴팁과 아이콘 표시를 멈춘다(`show_hover_menus`) | [KE#L2195-L2200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2195-L2200) |

---

## 3. 검색·필터

| ID | 항목 | 내용 | 출처 |
|---|---|---|---|
| K-040 | 액션 이름 검색 | 입력칸 글자로 **사람이 읽는 액션 이름**을 퍼지 검색한다(명령 팔레트와 같은 정규화). Context·Arguments는 검색 대상이 아니다 | [KE#L733-L760](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L733-L760) |
| K-041 | 키로 찾기 | ⌨ 버튼 또는 **Alt-F**(`ToggleKeystrokeSearch`): 키 검색 줄을 열고 키 입력칸에 녹화를 시작한다. 정확 일치 기본 **켜짐**. 정확 일치면 키 순서 전체가 같은 바인딩만, 끄면 입력한 키들이 바인딩 키 순서 안에 차례로 들어 있으면 일치 | [KE#L196-L212](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L196-L212), [KE#L769-L805](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L769-L805), [KE#L496](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L496) |
| K-042 | 필터 메뉴 (`Filter` 아이콘, 툴팁 "**Filters**", 버튼 오른쪽 위 기준으로 열림) | 헤더 "**Filters**": 토글 "**Conflicts**"(Alt-C) · "**No Action**"(NoAction 바인딩 표시) / 구분선 / 헤더 "**Categories**": 토글 "**User**" · "**Default**" · "**Vim**"(출처별 표시) | [KE#L1635-L1715](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1635-L1715) |
| K-043 | 기본 필터 | 충돌 필터 꺼짐(All), NoAction 바인딩 숨김. 출처 필터의 기본값과 "Base" 출처가 어느 범주에 들어가는지는 미확인 (⑦) | [KE#L216-L222](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L216-L222), [KE#L806-L809](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L806-L809) |
| K-044 | 결과 없음 문구 | 충돌 필터 + 검색어: "**No conflicting keybinds found that match the provided query**" / 충돌 필터만: "**No conflicting keybinds found**" / 키 검색: "**No keybinds found matching the entered keystrokes**" / 일반: "**No matches found for the provided query**" | [KE#L1258-L1272](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1258-L1272) |
| K-045 | 충돌 정의 | **같은 키 순서 + 같은 컨텍스트**의 바인딩이 둘 이상이면 충돌 후보. 출처 우선순위(User > Vim > Base > Default)가 높은 쪽이 낮은 쪽을 가린다(K-033). 둘 다 User면 서로 충돌(K-032) | [KE#L264-L375](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L264-L375) |

---

## 4. 키 입력칸 (KeystrokeInput)

키 검색 줄과 편집 모달의 "Edit Keystroke"에 같은 구성요소를 쓴다.

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| K-050 | 상자 | 좌우 padding `px_3` 12px, gap 8, **최소 높이 `min_h_10` 40px**, 모서리 `rounded_md`, 테두리 1px: 포커스 있으면 **`border_focused`**, 없으면 `border_variant`. 배경: 녹화 중 = `editor_background`에 `text_accent` 10%를 섞은 색, 평소 = `editor_background` | [KI#L466-L560](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L466-L560) |
| K-051 | 안쪽 배치 | [왼쪽: 녹화 표시 (폭 64px)] [가운데: 입력한 키, 14px(0.875rem), Default 색] [오른쪽: 버튼 (폭 64px)] | [KI#L362-L384](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L362-L384), [KI#L560-L620](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L560-L620) |
| K-052 | 녹화 표시 | 녹화 중: 높이 16px 작은 상자(테두리 1px `border`, 배경 위와 같은 섞은 색, `rounded_sm`, gap 2) 안에 **빨간 원**(`IconName::Circle`, Error 색, **2초 주기로 불투명도 0.4~0.8 깜빡임**) + "**REC**"(XSmall, Error). 키 검색 모드에서는 Accent 색 원 + "**SEARCH**" | [KI#L475-L525](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L475-L525) |
| K-053 | 오른쪽 버튼 | 녹화 중: `IconName::Stop`, 툴팁 "**Stop Recording**"(키 검색이면 "**Stop Searching**"). 평소: 녹화 아이콘(편집 = `PlayFilled`, 검색 = `MagnifyingGlass`), 툴팁 "**Start Recording**"/"**Start Searching**". 포커스가 없으면 Muted 색 | [KI#L585-L620](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L585-L620) |
| K-054 | 키 | **Enter** = 녹화 시작, **Escape Escape Escape**(Esc 세 번) = 녹화 끝, **Delete** = 입력한 키 지우기 (`KeystrokeInput` 컨텍스트) | [KM#L1476-L1483](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1476-L1483) |
| K-055 | 녹화 규칙 | 키 입력을 그대로 받아 적는다(다른 단축키를 실행하지 않는다). 최대 **3타**(`KEYSTROKE_COUNT_MAX`), 넘으면 자동으로 녹화를 끝낸다. 수식키만 누른 상태도 표시한다. 마지막 Esc들이 끝내기 키(Esc×3)의 일부일 수 있어, 끝낼 때 **300ms** 안의 그 Esc들은 기록에서 뺀다(`CLOSE_KEYSTROKE_CAPTURE_END_TIMEOUT`). 포커스를 잃으면 녹화를 멈춘다 (추정) | [KI#L24-L71](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L24-L71), [KI#L147-L345](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L147-L345), [KI#L436-L450](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L436-L450) |
| K-056 | placeholder | 편집할 때 키를 아직 누르지 않았으면 기존 바인딩 키를 자리 표시로 보인다. 녹화를 시작하면 사라진다 | [KI#L108-L118](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L108-L118), [KI#L362-L372](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L362-L372) |

---

## 5. 행 우클릭 메뉴

| ID | 순서 | 항목 | 액션 | 키 (KeymapEditor) | 조건 | 출처 |
|---|---|---|---|---|---|---|
| K-060 | 0 | **Create** | `CreateBinding` | Alt-Enter | 키가 없는 액션 행일 때만 | [KE#L1108-L1112](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1108-L1112) |
| K-061 | 1 | **Edit** | `EditBinding` | Enter | 키 없는 행이면 비활성 | [KE#L1112-L1118](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1112-L1118) |
| K-062 | 2 | **Delete** | `DeleteBinding` | (키 없음) | 키 없는 행이면 비활성 | [KE#L1118-L1122](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1118-L1122) |
| | ─ | | | | | |
| K-063 | 3 | **Copy Action** | `CopyAction` | Ctrl-C | 항상 | [KE#L1124](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1124) |
| K-064 | 4 | **Copy Context** | `CopyContext` | Ctrl-Shift-C | 컨텍스트가 없으면 비활성 | [KE#L1125-L1129](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1125-L1129) |
| | ─ | | | | | |
| K-065 | 5 | **Show Matching Keybindings** | `ShowMatchingKeybinds` | Ctrl-T | 키 없는 행이면 비활성 (같은 키 순서로 키 검색을 연다) | [KE#L1130-L1136](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1130-L1136) |

---

## 6. 키 (KeymapEditor 컨텍스트)

| ID | 키 | 동작 | 출처 |
|---|---|---|---|
| K-070 | Ctrl-F | 검색 입력칸에 포커스 | [KM#L1460-L1474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1460-L1474) |
| K-071 | Alt-F | 키로 찾기 토글 (툴팁 "**Search by Keystrokes**") | 위와 같음, [KE#L2071-L2090](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2071-L2090) |
| K-072 | Alt-C | 충돌 필터 토글 | 위와 같음 |
| K-073 | Enter / Alt-Enter | 선택 행 편집 / 선택한 액션에 새 바인딩 추가 | 위와 같음 |
| K-074 | Ctrl-K | 새 키바인딩 만들기 모달(액션 직접 입력, "Create Keybinding" 버튼과 같음) | 위와 같음 |
| K-075 | Ctrl-C / Ctrl-Shift-C | 액션 이름 복사 / 컨텍스트 복사 | 위와 같음 |
| K-076 | Ctrl-T | 같은 키 바인딩 보기 | 위와 같음 |
| K-077 | Ctrl-E | keymap.json 열기 (`zed::OpenKeymapFile`, "Edit in JSON" 버튼과 같음) | 위와 같음 |
| K-078 | 정확 일치 토글 | `ToggleExactKeystrokeMatching`(기본 키 없음), 툴팁 "**Toggle Exact Match Mode**" | [KE#L1982-L1995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1982-L1995) |
| K-079 | 위·아래 등 | 표 선택 이동은 menu 계열(Up/Down/Home/End/PageUp/PageDown) (추정: 표가 menu 액션을 받음) | [KM#L5-L21](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L5-L21) |

---

## 7. 바인딩 편집·추가 모달

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| K-080 | 모달 | 폭 **34rem(544px)**, `elevation_3`(모달 표면), 화면 위쪽 가운데 모달 층 (추정) | [KE#L3075-L3090](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3075-L3090) |
| K-081 | 머리 | 편집: 액션 이름(Default) + 아래 액션 설명(Muted), 아래 1px `border_variant`. 새로 만들기(Ctrl-K): 라벨 "**Create Keybinding**" + 액션 입력칸(라벨 "**Action**", placeholder "**Type an action name**", 액션 이름 자동완성) | [KE#L3090-L3125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3090-L3125), [KE#L2585-L2590](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2585-L2590), [action_completion_provider.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/action_completion_provider.rs) |
| K-082 | 본문 (세로, gap `_2p5` 10px) | ① "**Edit Keystroke**" + 키 입력칸(K-050). 같은 키의 바인딩이 있으면 아래에 "**There is 1 binding with the same keystrokes.**" / "There are N bindings …"(Muted, Small) + "**View**" 버튼(↗ 아이콘) ② 인자가 있는 액션이면 "**Edit Arguments**" + JSON 입력(구문 강조·자동완성, placeholder "**Action Arguments**", 14px·줄 높이 1.2) ③ 컨텍스트 입력칸(라벨 "**Edit Context**", placeholder "**Keybinding Context**", 컨텍스트 문법 강조) ④ 오류가 있으면 Banner(심각도별 색) | [KE#L3124-L3200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3124-L3200), [KE#L2529-L2531](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2529-L2531), [KE#L3415-L3495](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3415-L3495) |
| K-083 | 바닥 | 오른쪽: [**Cancel**] [**Save**], gap 4 | [KE#L3200-L3215](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3200-L3215) |
| K-084 | 모달 키 | **Ctrl-Enter = 저장**(menu::Confirm), **Escape = 취소** (`KeybindEditorModal`) | [KM#L1485-L1491](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1485-L1491) |
| K-085 | 검증 오류 문구 | "**Action name is required**" · "**Action '{이름}' not found**" · "**Failed to parse action arguments as JSON**" · "**Failed to validate action arguments**" · "**Keystrokes cannot be empty**" · "**Failed to parse key context**" | [KE#L2740-L2795](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2740-L2795) |
| K-086 | 충돌 경고 | 저장 전 같은 키·컨텍스트를 확인한다: "**Your keybind would conflict with the "{액션}" action**", 여러 개면 "… action and {N} other bindings", 이름을 모르면 "**Your keybind would conflict with other actions**". 경고를 보인 뒤 다시 Save를 누르면 저장한다 (추정) | [KE#L2835-L2860](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2835-L2860) |
| K-087 | 저장 뒤 | 성공하면 모달을 닫고 토스트 "**Saved edits to the {액션} action.**"(체크 아이콘)를 보인다. 수정·추가한 행을 다시 찾아 선택한다 | [KE#L2910-L2930](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2910-L2930), [KE#L469](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L469) |

---

## 8. 저장 위치와 파일 수정 규칙

| ID | 항목 | 내용 | 출처 |
|---|---|---|---|
| K-090 | 저장 파일 | 사용자 키맵 **`%APPDATA%\Zed\keymap.json`** (Windows: RoamingAppData\Zed\keymap.json). 백업 이름 `keymap_backup.json`. **우리 앱은 `%APPDATA%\{앱 이름}\keymap.json`** (판단, 04 A-20과 같은 폴더) | [paths.rs#L122-L135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/paths/src/paths.rs#L122-L135), [paths.rs#L296-L305](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/paths/src/paths.rs#L296-L305) |
| K-091 | 쓰기 방식 | 파일을 읽어 해당 바인딩만 고친 새 내용을 만들고 파일 전체를 다시 쓴다. 읽을 수 없거나 형식이 틀린 파일은 고치지 않는다(오류: "Failed to load keymap file", "Could not save updated keybinding", "Failed to write keymap file") | [KE#L3640-L3705](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3640-L3705), [KF#L929-L940](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L929-L940) |
| K-092 | User 바인딩 수정·삭제 | keymap.json 안의 그 항목을 바꾸거나 지운다 | [KF#L895-L940](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L895-L940) |
| K-093 | Default·Base 바인딩 수정 | 기본 파일은 고치지 않는다. **새 사용자 바인딩을 추가**하고, 원래 바인딩을 끄는 **unbind 항목**을 같은 섹션의 `unbind` 객체(키 → 액션)에 쓴다 | [KF#L895-L920](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L895-L920), [KF#L77-L80](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L77-L80) |
| K-094 | Default·Base 바인딩 삭제 | unbind 항목만 추가한다 (그 행은 K-034의 "unbound" 상태가 된다) | [KF#L918-L928](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/settings/src/keymap_file.rs#L918-L928) |
| K-095 | 없는 바인딩 삭제 | 오류 "**Cannot remove a keybinding that does not exist**" | [KE#L3720-L3723](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3720-L3723) |
| K-096 | 반영 | keymap.json이 바뀌면 키맵을 다시 읽어 표가 갱신된다. 메뉴·툴팁 단축키도 새 바인딩을 따른다(사용자 바인딩이 우선, L-217) | [KE#L835-L930](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L835-L930) |
| K-097 | 우리 앱 범위 (판단) | 표의 행은 **우리 앱이 등록한 액션과 그 바인딩만** 보인다(제외 기능의 액션은 등록하지 않으므로 자연히 빠진다). Source 범주에서 "Vim"은 우리 앱에 없으므로 필터 메뉴에서 뺀다. "Base"는 VSCode 덧씌움 바인딩이다 | _decisions.md, 04 A-27 |

---

## 9. 확신 낮은 부분

- **화면 근거:** 기준 화면 두 장에는 키맵 편집 화면이 없다. 치수·색은 모두 소스 값이다.
- **줄 번호:** keymap_editor.rs가 4,500줄을 넘어 일부 범위(K-023~K-037, K-080~K-087)는 grep으로 잡은 근사치(±수십 줄)다. 파일과 커밋은 정확하다.
- **출처 필터 (K-043):** 출처 범주의 기본값, 그리고 "Base" 출처가 User·Default·Vim 중 어느 토글에 묶이는지 확인하지 못했다.
- **행 테두리 (K-030):** 선택되지 않은 행의 2px 테두리 색(투명 또는 배경색)은 확인하지 못했다.
- **충돌 경고 뒤 저장 (K-086):** 경고를 보인 뒤 두 번째 Save로 저장되는지는 코드 흐름에서 추정했다.
- **녹화 중 포커스 잃음 (K-055):** 녹화를 멈추는지 추정이다. 입력 중 IME 처리도 확인하지 않았다.
- **표 키 탐색 (K-079):** 표가 menu::SelectNext 등을 받는다는 것은 추정이다.
- **모달 위치 (K-080):** 화면 위쪽 가운데 모달 층이라는 것은 다른 모달(Go to Line)과 같다고 본 추정이다.
- **humanized_name 규칙:** 액션 이름을 사람이 읽는 형태로 바꾸는 규칙(예: `pane::GoBack` → "pane: go back" 형태로 추정)은 명령 팔레트 조사(zed-pickers-dialogs.md)와 함께 확인해야 한다.
