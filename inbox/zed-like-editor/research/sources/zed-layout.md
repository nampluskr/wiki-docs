# Zed 레이아웃과 치수 조사 (ID 접두사 L-)

- 담당: scouter-opus · 조사일 2026-10-31
- 소스 기준: zed-industries/zed `main` 커밋 `14b7b0ef3764d62a85070c8b5ccea105a2ad3235` (링크는 모두 이 커밋에 고정). 코드 원문은 옮기지 않고 값·이름·동작만 요약했다 (GPL-3.0).
- 기준 화면: `D:\projects\docs\guides\zed-like-editor\01-zed\assets\zed-reference.png` (2880×1704 물리 px). 타이틀 바가 64 물리 px이고 Windows 타이틀 바는 소스상 32px 고정(L-21)이므로 **배율 200%**로 보고 논리 px = 물리 px ÷ 2로 환산했다. 이 환산값은 "관찰"로 표시한다.
- 사용자 Zed 설정(`%APPDATA%\Zed\settings.json`, 읽기만): `project_panel.dock = "left"`, `agent.dock = "right"`, `ui_font_size 16`, `buffer_font_size 15`, 테마 `Catppuccin Mocha`(다크)/`One Light`, 아이콘 테마 `Catppuccin Latte`. **기본값은 프로젝트 패널이 오른쪽**이므로(L-80), 기준 화면의 왼쪽 프로젝트 패널은 사용자 설정 결과다.

---

## 0. 단위 체계

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| L-01 | rem 크기 | 창의 rem = `ui_font_size`(기본 16px). `setup_ui_font`가 `window.set_rem_size(ui_font_size)`를 호출한다. gpui 창 기본 rem도 16px | [theme_settings/settings.rs:605-612](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/settings.rs#L605-L612), [gpui/window.rs:2216](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/window.rs#L2216) |
| L-02 | `rems_from_px` | px ÷ `BASE_REM_SIZE_IN_PX`(16). 결과는 rem이라 `ui_font_size`를 바꾸면 비례해 커진다 | [ui/styles/units.rs:4-14](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/units.rs#L4-L14) |
| L-03 | `DynamicSpacing::BaseXX` | XX = 기본 밀도·rem 16일 때 px. 단일 값 n은 (Compact n−4, Default n, Comfortable n+4), 튜플은 그대로. 정의 목록: Base00 (0,0,0), Base01 (1,1,2), Base02 (1,2,4), Base03 (2,3,4), Base04 (2,4,6), Base06 (3,6,8), Base08 (4,8,10), Base12 (10,12,14), Base16 (14,16,18), Base20 (18,20,22), Base24, Base32, Base40, Base48 | [ui/styles/spacing.rs:29-44](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/spacing.rs#L29-L44), [ui_macros/dynamic_spacing.rs:69-83](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui_macros/src/dynamic_spacing.rs#L69-L83) |
| L-04 | UI 밀도 | `UiDensity` Compact 0.75 / **Default 1.0** / Comfortable 1.25 | [theme/ui_density.rs:21-41](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/ui_density.rs#L21-L41) |
| L-05 | 글자 크기 단계 (`TextSize`) | Large 16 / **Default 14** / Small 12 / XSmall 10 (모두 `rems_from_px`), Ui = `ui_font_size`, Editor = `buffer_font_size`. `Label` 기본 크기 = Default(14px) | [ui/styles/typography.rs:134-156](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/typography.rs#L134-L156), [label_like.rs:7-16](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/label/label_like.rs#L7-L16) |
| L-06 | 아이콘 크기 (`IconSize`) | Indicator 10 / XSmall 12 / **Small 14** / Medium 16(기본) / XLarge 48 | [ui/components/icon.rs:57-80](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/icon.rs#L57-L80) |
| L-07 | 버튼 높이 (`ButtonSize`) | Large 32 / Medium 28 / **Default 22** / Compact 18 / None 16. 좌우 안쪽 여백: Large·Medium은 Base08(8), Default·Compact는 Base04(4), 내부 gap Base04 | [button_like.rs:455-471, 805-822](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L455-L471) |
| L-08 | 글꼴 기본값 | UI `.ZedSans`(현재 IBM Plex Sans 별칭) 400, **16px** / 버퍼 `.ZedMono` **15px**, `buffer_line_height: "comfortable"` / 글꼴 크기는 6~100px로 제한 | [default.json:31-71](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L31-L71), [theme_settings/settings.rs:18-19](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/settings.rs#L18-L19) |
| L-09 | gpui 기본 줄 높이 | `phi()`(≈1.618). 그래서 14px 라벨 한 줄 ≈ 22.6px | [gpui/style.rs:494](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/style.rs#L494) |
| L-10 | tailwind식 단위 | `_1` = 0.25rem = 4px, `_2` = 8px, `_4` = 16px, `_6` = 24px, `_0p5` = 2px, `_px` = 1px (rem 16 기준). gpui 스타일 메서드 규칙 | 추정 (gpui `styled` 매크로 관례, 위 값과 화면 관찰로 일관 확인) |

---

## 1. 전체 배치 (나무 구조)

| ID | 단위 | 내용 | 출처 |
|---|---|---|---|
| L-11 | 창 최상위 `MultiWorkspace` | 가로 배치: **[왼쪽 사이드바] → [Workspace (flex_1)] → [오른쪽 사이드바]**, 그 위에 modal layer와 overlay. 사이드바 위치는 `agent.threads_sidebar.position`(기본 left) | [workspace/multi_workspace.rs:2205-2215](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/multi_workspace.rs#L2205-L2215), [default.json:1191-1197](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1191-L1197) |
| L-12 | 사이드바는 창 전체 높이 | 사이드바 헤더 높이 = 타이틀 바 높이(`platform_title_bar_height`)라서, 사이드바가 타이틀 바 줄까지 차지하고 Workspace의 타이틀 바는 사이드바 오른쪽에서 시작한다 (기준 화면과 일치) | [sidebar.rs:7387-7390](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7387-L7390) |
| L-13 | `Workspace` 세로 배치 | **타이틀 바(`titlebar_item`) → 본문 → 상태 표시줄(`status_bar`) → toast layer** | [workspace.rs:9851](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L9851), [workspace.rs:10279-10282](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L10279-L10282) |
| L-14 | 본문 (`bottom_dock_layout: "contained"` 기본) | 가로: **왼쪽 Dock → [세로: 가운데(PaneGroup) → 아래 Dock] → 오른쪽 Dock**. 다른 모드: full, left_aligned, right_aligned | [workspace.rs:10180-10228](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L10180-L10228), [default.json:97](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L97) |
| L-15 | 가운데 영역 | `render_center` → `PaneGroup`(분할 나무) → 각 `Pane` = **탭 바 + 활성 아이템(편집기·미리보기)**. 분할 방향 기본: 가로 분할은 down, 세로 분할은 right | [workspace.rs:9086](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L9086), [default.json:99-101](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L99-L101) |
| L-16 | `Dock` | 위치 Left/Right/Bottom, 패널(Panel) 여러 개를 담고 활성 패널 하나만 표시. 테두리: 왼쪽 Dock은 `border_r_1`, 오른쪽은 `border_l_1`, 아래는 `border_t_1` | [dock.rs:1356-1358](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1356-L1358) |
| L-17 | 확대(zoom) | 확대된 Pane이나 패널은 본문 위 overlay로 그린다. `zoomed_padding: true`이면 사방 `_2`(8px) 여백과 `border_1` | [workspace.rs:10238-10275](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L10238-L10275), [default.json:253](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L253) |

정리하면 (기준 화면, 사이드바 왼쪽, 프로젝트 패널 왼쪽 Dock):

```
MultiWorkspace (h_flex)
├─ Sidebar (전체 높이, border_r_1)
│   ├─ 헤더 (32px, 타이틀 바 높이와 같음)
│   ├─ 목록 (프로젝트 헤더 31px + 항목)
│   └─ 푸터 (p_1, border_t_1: 사이드바 토글·기록·폴더 추가)
└─ Workspace (v_flex, flex_1)
    ├─ TitleBar (32px)
    ├─ 본문 (h_flex)
    │   ├─ Left Dock → ProjectPanel (border_r_1, 오른쪽 가장자리에 리사이즈 핸들)
    │   ├─ 가운데 (v_flex) → PaneGroup → Pane (TabBar 32px + 아이템)
    │   └─ Right Dock
    └─ StatusBar
```

---

## 2. 치수 요약

| ID | 대상 | 값 (논리 px, 배율 100%) | 계산 근거 | 기준 화면 관찰 | 출처 |
|---|---|---|---|---|---|
| L-21 | 타이틀 바 높이 | **Windows 32 고정**. 다른 OS는 max(1.75×rem, 34) | Windows 전용 함수가 `px(32.)` 반환 | 32 | [ui/utils.rs:14-24](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/utils.rs#L14-L24) |
| L-22 | 탭 바 높이 | **32** (`Tab::container_height` = Base32) | Base32 Default = 32/16 rem → 32 | 약 32 | [tab.rs:79-85](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L79-L85), [tab_bar.rs:100](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab_bar.rs#L100) |
| L-23 | 탭 내용 높이 | **31** (`content_height` = Base32 − 1, 아래 테두리 1px 몫) | 32 − 1 | — | [tab.rs:79-81](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L79-L81) |
| L-24 | 탭 안쪽 여백·간격 | 좌우 padding Base04 = **4**, 요소 gap Base04 = **4** | — | — | [tab.rs:172-174](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L172-L174) |
| L-25 | 탭 슬롯 | 시작 슬롯 **12×12**(수정 표시 점), 끝 슬롯 **14×14**(닫기·고정 버튼) | 상수 `START_TAB_SLOT_SIZE`, `END_TAB_SLOT_SIZE` | — | [tab.rs:8-9](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L8-L9) |
| L-26 | 탭 최소·최대 폭 | **고정 최소·최대 폭 없음** (내용 폭). 제목은 `MAX_TAB_TITLE_LEN = 24`자에서 말줄임 | — | "README.md" 탭 ≈ 125 | [items.rs:74](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L74), [markdown_preview_view.rs:1673-1683](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1673-L1683) |
| L-27 | 탭 바 양 끝 버튼 영역 | gap Base04 = 4, 좌우 padding Base06 = **6**, `border_b_1`. 시작 영역은 `border_r_1`, 끝 영역은 `border_l_1` | — | — | [tab_bar.rs:104-148](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab_bar.rs#L104-L148) |
| L-28 | 상태 표시줄 높이 | 명시적 높이 없음. padding Base04(4) ×2 + Default 버튼 22 = **30**, 아래 1px 테두리 보정 | 4 + 22 + 4 | 약 31 | [status_bar.rs:150-182](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L150-L182) |
| L-29 | 상태 표시줄 간격 | 좌·우 그룹 사이 gap Base08 = 8, 그룹 안 gap `_1` = 4 | — | — | [status_bar.rs:152, 196, 215](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L152) |
| L-30 | Dock 리사이즈 핸들 | **6px**(`RESIZE_HANDLE_SIZE`), 테두리 위에 절반씩 걸친다(−3px 위치). 커서 `col_resize`/`row_resize`. **더블클릭 시 기본 크기로 초기화** | — | — | [dock.rs:25](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L25), [dock.rs:1281-1336](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1281-L1336) |
| L-31 | Dock 최소 폭 | 패널 기본 `min_size`는 None. 크기 하한은 리사이즈 핸들(6px), 상한은 창 폭 − 반대쪽 Dock − 6 | — | — | [dock.rs:51-53, 413](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L51-L53), [workspace.rs:9192-9227](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L9192-L9227) |
| L-32 | 프로젝트 패널 기본 폭 | **240** (`project_panel.default_width`) | — | 약 285 (사용자가 조정한 것으로 추정) | [default.json:888](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L888) |
| L-33 | 사이드바 폭 | 기본 **300**, 범위 **200~800** (`THREADS_LIST_MIN/MAX_WIDTH`) | — | 약 200 (최소값) | [default.json:1191-1197](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1191-L1197), [agent_settings.rs:37-38](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/agent_settings/src/agent_settings.rs#L37-L38) |
| L-34 | 사이드바 프로젝트 헤더 행 | 높이 `Tab::content_height` = **31**, 왼쪽 padding `_2`(8), 오른쪽 `_1p5`(6) | — | 약 31 | [sidebar.rs:2410-2413](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2410-L2413) |
| L-35 | 사이드바 푸터 | `p_1`(4), gap `_1`(4), `border_t_1`, 아이콘 버튼 Small(14) | — | 상태 표시줄과 같은 높이로 보임 | [sidebar.rs:7533-7545](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7533-L7545) |
| L-36 | 프로젝트 패널 항목 높이 | 명시 없음. 라벨 14px × 줄 높이 1.618 ≈ 22.6 + 위아래 `border_1` 2 ≈ **25~26**. `entry_spacing: "comfortable"`이면 `ListItemSpacing::Dense`(추가 padding 0), "standard"이면 ExtraDense(−1px) | 추정 계산 | 약 26 | [project_panel.rs:6628-6634](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6628-L6634), [list_item.rs:363-368](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L363-L368) |
| L-37 | 프로젝트 패널 들여쓰기 | 단계당 **20** (`indent_size`), 항목 margin-left = 깊이 × 20. ListItem 안쪽 좌우 padding Base06(6), gap `_1`(4) | — | 약 17~18 (추정 차이) | [default.json:907](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L907), [list_item.rs:363-364, 408](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L363-L364) |
| L-38 | 들여쓰기 안내선 | 폭 **1px**, x = 깊이 × indent_size + **15**(`LIST_ITEM_INDENT_GUIDE_LEFT_OFFSET`), 세로 길이 = 자식 수 × 항목 높이(위아래 4px 여백). 색 3단계: `panel_indent_guide`(기본) / `_hover` / `_active`(선택 항목이 속한 안내선) | — | — | [indent_guides.rs:22-33](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/indent_guides.rs#L22-L33), [project_panel.rs:7815-7850](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7815-L7850) |
| L-39 | 창 버튼 (Windows) | 각 **36×32**, 글리프 10px | — | — | [platform_windows.rs:134-136](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs#L134-L136) |
| L-40 | 아이콘 크기 실사용 | 탭 바 ←→·+·분할·최대화 = Small(14) / 상태 표시줄 패널 버튼 = Small(14) / 탭의 파일 아이콘 = Small(14) / 프로젝트 패널 파일 아이콘 = 기본 Medium(16, 추정) | — | 프로젝트 패널 아이콘 ≈ 16 | [pane.rs:3508-3532](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3508-L3532), [dock.rs:1542-1544](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1542-L1544), [pane.rs:2824-2834](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2824-L2834) |

---

## 3. 프로젝트 패널 구조

| ID | 항목 | 동작 | 출처 |
|---|---|---|---|
| L-41 | 루트 표시 | 워크트리 루트를 맨 위 항목으로 그린다(기준 화면의 "projects"). `hide_root: true`이고 폴더가 하나뿐이면 루트를 숨긴다 (기본 false) | [project_panel.rs:4636-4665](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L4636-L4665), [default.json:999](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L999) |
| L-42 | 경로 접기 규칙 (`auto_fold_dirs`, 기본 true) | 디렉터리의 자식이 **정확히 하나이고 그 자식도 디렉터리**이면 한 줄로 합친다. 이 규칙을 연쇄 적용 (예: `docs\guides`) | [project_panel.rs:1468-1479](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1468-L1479), [project_panel.rs:4677-4690](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L4677-L4690) |
| L-43 | 접힌 경로 표시 | 각 경로 조각이 따로 hover되는 요소(좌우 padding `_0p5` = 2px, `rounded_xs`, hover 시 `element_active` 배경). 구분자는 OS 경로 구분자(`primary_separator`, Windows는 `\`). 조각을 클릭하면 그 단계를 선택. 기준 화면의 "docs \ guides" 사이 공백은 이 여백으로 보인다 (추정) | [project_panel.rs:6884-6905](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6884-L6905) |
| L-44 | 펼침 화살표 | `folder_indicator` 기본 **"icon"**: 화살표 없이 폴더 아이콘(닫힘·열림)만. "chevron"은 화살표만, "both"는 화살표와 아이콘 (기준 화면은 화살표 없음 = 기본값) | [default.json:896-904](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L896-L904) |
| L-45 | 가로 스크롤 | `scrollbar.horizontal_scroll` 기본 true. 가장 긴 항목(`max_width_item_index`) 폭으로 목록 폭을 잡아 가로로 스크롤한다. false이면 왼쪽 고정, 긴 이름은 잘린다 | [default.json:936-938](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L936-L938), [project_panel.rs:7539, 7949-7954](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7949-L7954) |
| L-46 | 가상화 | 항목 목록은 `uniform_list`(모든 행 높이 같음)로 그린다 | [project_panel.rs:7732](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7732) |
| L-47 | 항목 색 | 기본 배경 `panel_background`, hover `element_hover`, 선택(marked) `element_selected`, 포커스 테두리 `panel_focused_border`, 드래그 대상 `drop_target_background`. sticky 행은 `panel_overlay_background`/`panel_overlay_hover` | [project_panel.rs:659-676](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L659-L676) |
| L-48 | 포커스 표시 | 항목은 모서리 없이(`rounded_none`) **`border_1` + 오른쪽 `border_r_2`**. 평소 테두리 색 = 배경색(안 보임). 패널에 키보드 포커스가 있고 마우스가 눌리지 않았으며 그 항목이 활성일 때만 테두리를 `panel_focused_border`로 칠한다 | [project_panel.rs:6180-6200, 6271-6277](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6271-L6277) |
| L-49 | 이름 tooltip | 항목에 마우스를 올리면 1500ms 뒤 전체 경로 tooltip (`title_tooltip_delay`) | [project_panel.rs:6254-6268](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6254-L6268) |
| L-50 | 정렬 | `sort_mode: "directories_first"`, `sort_order: "default"`(대소문자 무시 자연 정렬) | [default.json:971-990](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L971-L990) |
| L-51 | sticky scroll | 기본 true: 스크롤할 때 상위 폴더 행이 위에 붙는다 (채용 여부는 판단 필요) | [default.json:950](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L950) |
| L-52 | 제외 (Git) | `git_status`, `git_status_indicator`, `show_diagnostics`, `diagnostic_badges`는 채용하지 않음 (기준 화면 규칙) | [default.json:905, 948, 993-995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L905) |

---

## 4. 탭 바 구조

| ID | 위치·요소 | 동작 | 출처 |
|---|---|---|---|
| L-61 | 구성 | `TabBar` = [시작 영역] + [탭 목록(가로 스크롤)] + [끝 영역]. 시작·끝 영역은 탭 바가 넘쳐도 고정 | [tab_bar.rs:96-150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab_bar.rs#L96-L150) |
| L-62 | 시작 영역 ← → | `navigate_backward`(ArrowLeft), `navigate_forward`(ArrowRight), Small(14). 이동할 기록이 없으면 비활성(disabled). tooltip에 액션 단축키(GoBack/GoForward). `tab_bar.show_nav_history_buttons` 기본 true | [pane.rs:3508-3550](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3508-L3550), [pane.rs:3604-3620](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3604-L3620) |
| L-63 | 탭 테두리 | 탭 위치(첫·마지막·활성 기준 왼쪽/오른쪽)에 따라 좌우 테두리를 다르게 그린다. **활성 탭은 아래 테두리가 없어** 내용 영역과 이어진다. 비활성 탭은 `border_b_1` | [tab.rs:145-166](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L145-L166) |
| L-64 | 탭 내용 순서 | [시작 슬롯: 수정 표시 점] [아이콘] [제목(+구분용 설명)] [끝 슬롯: 닫기]. `tabs.close_position: "left"`이면 시작·끝 슬롯을 맞바꾼다 | [tab.rs:127-141](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L127-L141), [pane.rs:3086-3096](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3086-L3096) |
| L-65 | 수정 표시 | `Indicator::dot`. 충돌(conflict)은 Warning 색, 수정됨(dirty)은 **Accent 색**, 깨끗하면 없음 | [pane.rs:5090-5099](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L5090-L5099) |
| L-66 | 닫기 버튼 | `IconName::Close`, Small, Muted, 정사각형. `tabs.show_close_button` 기본 **"hover"**(탭에 마우스를 올릴 때만), "always", "hidden". 고정(pinned) 탭은 닫기 대신 Pin 아이콘 | [pane.rs:3033-3066](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3033-L3066), [default.json:1511](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1511) |
| L-67 | 파일 아이콘 | `tabs.file_icons` 기본 **false**라 편집기 탭에는 아이콘 없음 (기준 화면 "README.md" 탭과 일치). 아이템이 자체 아이콘을 주면 Small·Muted로 표시 | [items.rs:760-766](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L760-L766), [pane.rs:2824-2834](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2824-L2834) |
| L-68 | 미리보기 탭 (Preview tab) | 프로젝트 패널에서 한 번 클릭으로 연 파일은 **미리보기 탭**: 제목이 **기울임꼴(italic)**이고, 다른 미리보기를 열면 대체된다. `preview_tabs.enabled`, `enable_preview_from_project_panel` 기본 true | [items.rs:834](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L834), [default.json:1534-1541](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1534-L1541) |
| L-69 | 마크다운 미리보기 탭 | 제목 "**Preview {파일명}**"(24자 말줄임), 아이콘 `IconName::FileDoc`(책 모양, `assets/icons/file_doc.svg`)이 항상 붙는다. 편집기가 없으면 "Markdown Preview". 같은 이름이 겹치면 XSmall·Muted 설명(디렉터리) 추가 | [markdown_preview_view.rs:1663-1712](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1663-L1712) |
| L-70 | 넘칠 때 | 탭 목록 영역이 `overflow_x_scroll`(가로 스크롤)이고 스크롤바는 없다. 탭을 활성화하면 `scroll_to_item`으로 그 탭이 보이게 이동. 마지막 탭 뒤에 빈 드롭 영역(`min_w_6` = 24px). 기준 화면 오른쪽 끝 "Preview (" 처럼 잘려 보인다 | [pane.rs:3694](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3694), [pane.rs:1524](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1524), [pane.rs:3738](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3738) |
| L-71 | 끝 영역 `+` | `IconName::Plus`, tooltip "New…", 메뉴(오른쪽 위 기준): New File, Open File / Search Project, Search Symbols / New Terminal… → **우리 앱은 New File만 남긴다** (판단) | [pane.rs:4360-4388](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4360-L4388) |
| L-72 | 끝 영역 분할 | `IconName::Split`, tooltip "Split Pane", 메뉴: Split Right / Left / Up / Down. 분할할 수 없으면 비활성 | [pane.rs:4388-4412](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4388-L4412) |
| L-73 | 끝 영역 최대화 | `IconName::Maximize`, 확대 상태에서는 `Minimize` 아이콘(selected). `ToggleZoom`, tooltip "Zoom In"/"Zoom Out" | [pane.rs:4417-4430](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4417-L4430) |
| L-74 | 탭 클릭 | 가운데 클릭으로 닫기(고정 탭 제외), 드래그로 순서 변경·다른 Pane으로 이동 | [pane.rs:2972-3030](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2972-L3030) |

---

## 5. 타이틀 바

| ID | 항목 | 내용 | 출처 |
|---|---|---|---|
| L-81 | 그리는 방식 | 기본 `window_decorations: "client"`: OS 타이틀 바 대신 Zed가 직접 그린다. 타이틀 바 전체가 `WindowControlArea::Drag`(끌어서 창 이동) | [default.json:282](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L282), [platform_title_bar.rs:219-221](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platform_title_bar.rs#L219-L221) |
| L-82 | 왼쪽 여백 | 내용 시작 padding `pl_2` = 8px (macOS만 신호등 여백) | [platform_title_bar.rs:272-287](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platform_title_bar.rs#L272-L287) |
| L-83 | Windows 창 버튼 | 최소화·최대화(또는 복원)·닫기, 오른쪽 끝. **글꼴 글리프**로 그린다: Windows 11(빌드 ≥ 22000)은 `Segoe Fluent Icons`, 그 이전은 `Segoe MDL2 Assets`. 코드포인트 최소화 U+E921, 최대화 U+E922, 복원 U+E923, 닫기 U+E8BB. 각 버튼에 `WindowControlArea`(Min/Max/Close)를 지정해 OS 히트테스트(스냅 레이아웃 등)에 연결 | [platform_windows.rs:15-95](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs#L15-L95) |
| L-84 | 창 버튼 색 | 닫기 hover 배경 **#E81120**(232,17,32) + 흰 글자, 누름은 각각 80% 불투명. 나머지 버튼 hover `ghost_element_hover`, 누름 `ghost_element_active`, 글자 `text` | [platform_windows.rs:104-128](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs#L104-L128) |
| L-85 | ☰ 메뉴 | Windows·Linux에서 `title_bar.show_menus: false`(기본)이면 메뉴 막대 대신 **햄버거 버튼 하나**(`IconName::Menu`, Subtle, Small, tooltip "Open Application Menu"). 누르면 최상위 메뉴(Zed, File, Edit, Selection, View, Go, Run, Window, Help)가 하위 메뉴로 펼쳐지는 팝업 | [title_bar.rs:453-462](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L453-L462), [application_menu.rs:176-200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L176-L200), [app_menus.rs:71-291](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L71-L291) |
| L-86 | 우리 기능에 해당하는 Zed 메뉴 항목 | File: New, Open File…, Add Folder to Project…, Save, Save As…, Save All, Close Window / View: Toggle Left Dock, Editor Layout(Split Up/Down/Left/Right), Project Panel / Go: Back, Forward, Command Palette…, Go to File…, Go to Line/Column… (선별은 판단) | [app_menus.rs:28-43, 116-149, 227-241](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L116-L149) |
| L-87 | 왼쪽 묶음 순서 | [☰ 메뉴] → [제한 모드 경고] → [원격 호스트] → **[프로젝트 이름]** → [Git 브랜치]. 묶음 gap `_0p5` = 2px | [title_bar.rs:298-345](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L298-L345) |
| L-88 | 프로젝트 이름 | 활성 워크트리 루트 폴더 이름(Git 저장소면 저장소 이름 기준), **40자**에서 말줄임 (`MAX_PROJECT_NAME_LENGTH`). 누르면 최근 프로젝트 팝업. `title_bar.show_project_items`로 켜고 끈다 | [title_bar.rs:64, 233-292, 783-860](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L783-L860) |
| L-89 | 오른쪽 묶음 (제외 대상) | `pr_1`, gap `_1`: 통화 컨트롤, 연결 상태, 업데이트 알림, Sign In, 사용자 메뉴 → **우리 앱은 창 버튼만** (기준 화면 규칙) | [title_bar.rs:367-397](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L367-L397) |

---

## 6. 상태 표시줄

| ID | 항목 | 내용 | 출처 |
|---|---|---|---|
| L-91 | 구성 방식 | `StatusBar`는 `left_items`·`right_items` 두 목록. 각 아이템은 `StatusItemView`이고 `add_left_item`/`add_right_item`으로 등록. 가로 `justify_between`, 배경 `status_bar_background` | [status_bar.rs:101-154](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L101-L154) |
| L-92 | Dock 패널 버튼 | Workspace가 만들 때 **왼쪽 Dock 버튼 묶음을 왼쪽 첫 아이템**으로, 오른쪽 Dock·아래 Dock 버튼을 오른쪽 아이템으로 넣는다. 각 패널 버튼은 IconButton Small, 열려 있으면 `toggle_state`(선택 표시) | [workspace.rs:1972-1984](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L1972-L1984), [dock.rs:1542-1545](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1542-L1545) |
| L-93 | 프로젝트 패널 버튼 | 아이콘 `IconName::FileTree`, tooltip "Project Panel". `project_panel.button` 기본 true | [project_panel.rs:8278-8286](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8278-L8286) |
| L-94 | Zed 기본 아이템 순서 | 왼쪽: [Dock 버튼] 검색, LSP, 진단, 활성 파일 이름, Git blame, 병합 충돌, 활동 표시 / 오른쪽: 편집 예측, 인코딩, 언어, 툴체인, 줄 끝, **커서 위치**, 이미지 정보, [오른쪽·아래 Dock 버튼], Vim 모드, 대기 키 입력 | [zed.rs:636-652](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L636-L652) |
| L-95 | 사이드바 토글 | 사이드바가 **닫혀 있을 때만** 상태 표시줄 맨 왼쪽(사이드바가 오른쪽이면 맨 오른쪽)에 `ThreadsSidebarLeftClosed` 아이콘 버튼(Small, tooltip "Open Threads Sidebar")이 나온다. 열려 있으면 사이드바 푸터의 닫기 토글(`ThreadsSidebarLeftOpen`)을 쓴다 | [status_bar.rs:198-202, 240-262](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L240-L262), [sidebar.rs:7488-7500](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7488-L7500) |
| L-96 | 폴더 추가 버튼 | 사이드바 안 `IconButton("open-project", IconName::FolderAdd)`, Small, tooltip "Add Project"(OpenRecent 액션). 기준 화면에서는 사이드바 하단 오른쪽(상태 표시줄 높이 줄)에 보인다 | [sidebar.rs:6770-6778](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L6770-L6778) |
| L-97 | 커서 위치 형식 | "**{줄}:{열}**"(구분자 `:`; 기준 화면 "9:11"). 줄·열 모두 1부터. 선택이 있으면 뒤에 " (…)" 추가: long(기본) "2 selections, 15 lines, 32 characters", short "2 s, 15 l, 32 c". Button LabelSize::Small(12), 누르면 Go to Line/Column. `status_bar.cursor_position_button` 기본 true | [cursor_position.rs:31-60, 181-225](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L181-L225), [default.json:2039, 2940-2945](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2940-L2945) |
| L-98 | 우리 앱 구성 (판단) | 왼쪽: [사이드바 토글] [폴더 추가] [프로젝트 패널 토글] / 오른쪽: [커서 위치]. 기록·검색·진단·편집 예측·터미널·디버그·개요·협업·Git·AI는 제외 | 기준 화면 규칙 |

---

## 7. 마크다운 미리보기 화면

| ID | 항목 | 값 | 출처 |
|---|---|---|---|
| L-101 | 본문 폭 | `markdown_preview.limit_content_width: true`, **`max_width: 800`px**. 넓으면 가운데 정렬(`mx_auto`) | [default.json:128-134](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L128-L134), [markdown_preview_view.rs:2015-2018](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L2015-L2018) |
| L-102 | 바깥 여백·배경 | 스크롤 컨테이너 사방 `p_4` = **16px**, 세로 스크롤만. 배경 `editor_background`. 미리보기 안 rem은 미리보기 글꼴 크기(`markdown_preview.font_size`, 없으면 `ui_font_size` = 16) | [markdown_preview_view.rs:1912-1955](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1912-L1955), [default.json:117-130](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L117-L130) |
| L-103 | 본문 글자 | 1rem(16px), 줄 높이 1.5, 문단 간격 16px, 목록 간격 12px, 글자색 `text`. 글꼴은 UI 글꼴, 코드는 버퍼 글꼴 | [markdown.rs:356-370](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L356-L370) |
| L-104 | 제목 | 굵기 `heading_font_weight` 600, 줄 높이 1.25, 색 `text`. 크기 h1 1.75rem(28) / h2 1.4(22.4) / h3 1.2(19.2) / h4 1.0(16) / h5 0.875(14) / h6 0.85(13.6, `text_muted`). 위 여백 24, 아래 12 | [markdown.rs:384-410](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L384-L410), [default.json:126](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L126) |
| L-105 | 제목 밑줄 | h1: 아래 padding 8 + `border_b_1`, h2: 아래 padding 4 + `border_b_1`, 색 `border_variant` (기준 화면의 h1·h2 아래 선) | [markdown.rs:3549-3580](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L3549-L3580), [markdown.rs:409](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L409) |
| L-106 | 인용(blockquote) | 왼쪽 padding `pl_4` 16, **왼쪽 테두리 4px**(`border_l_4`, 색 `border`), 글자색 `text_muted` | [markdown.rs:2160-2170](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L2160-L2170), [markdown.rs:245, 366](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L245) |
| L-107 | 표 | grid, 바깥 테두리 **1.5px** `border` + `rounded_sm`. 셀 padding 가로 10 × 세로 4, 셀 사이 1px 선. **머리행 배경 `title_bar_background` + SEMIBOLD**, 본문 홀수 번째 행(0부터 셀 때 1, 3, …) 배경 `panel_background`(줄무늬). 열 정렬(가운데·오른쪽) 반영 | [markdown.rs:3079-3155](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L3079-L3155) |
| L-108 | 링크 | 색 `text_accent`, 밑줄 1px(`text_accent` 50% 불투명), 미리보기에서는 배경 없음 | [markdown.rs:299-307](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L299-L307), [markdown.rs:367](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L367) |
| L-109 | 인라인 코드·코드 블록 | 인라인: 0.875rem, 배경 `editor_foreground` 8%. 블록: padding 12, 위아래 margin 16, 모서리 6, 미리보기에서는 테두리 없음, hover 시 복사 버튼 | [markdown.rs:290-298, 368-380](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L368-L380), [markdown_preview_view.rs:1121-1125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1121-L1125) |
| L-110 | frontmatter → 표 | 미리보기는 `render_metadata_blocks: true`. 파일 맨 앞 YAML frontmatter의 **모든 줄이 공백으로 시작하지 않는 `키: 값`**(키·값 모두 비어 있지 않음)이면 2열 표로 그린다. 하나라도 어기면 코드 블록 스타일 상자로 원문 표시 | [markdown_preview_view.rs:400](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L400), [parser.rs:175-208](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/parser.rs#L175-L208) |
| L-111 | frontmatter 표 스타일 | 2열 grid, 전체 폭, `border_1`(`border`) + `rounded_sm`. 키 셀: 배경 `panel_background`, SEMIBOLD. 값 셀: `border_l_1`. 두 번째 행부터 `border_t_1` (기준 화면의 tags·created·updated 표와 일치) | [markdown.rs:2184-2285](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L2184-L2285) |

---

## 8. 관련 설정 키와 기본값 (`assets/settings/default.json`)

| ID | 키 | 기본값 | 줄 |
|---|---|---|---|
| L-120 | `ui_font_size` / `ui_font_family` / `ui_font_weight` | 16 / ".ZedSans" / 400 | [59-71](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L59-L71) |
| L-121 | `buffer_font_size` / `buffer_font_family` / `buffer_line_height` | 15 / ".ZedMono" / "comfortable" | [31-54](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L31-L54) |
| L-122 | `window_decorations` | "client" | [282](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L282) |
| L-123 | `title_bar.show_menus` / `show_project_items` / `show_branch_name` / `show_sign_in` / `show_user_menu` | false / true / true / true / true (우리 앱: 메뉴 버튼·프로젝트 이름만) | [620-640](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L620-L640) |
| L-124 | `agent.threads_sidebar.position` / `default_width` | "left" / 300 | [1191-1197](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1191-L1197) |
| L-125 | `project_panel.dock` / `default_width` / `button` / `starts_open` | **"right"** / 240 / true / true | [880-913](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L880-L913) |
| L-126 | `project_panel.entry_spacing` / `indent_size` / `file_icons` / `folder_indicator` | "comfortable" / 20 / true / "icon" | [892-907](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L892-L907) |
| L-127 | `project_panel.auto_fold_dirs` / `hide_root` / `bold_folder_labels` / `sticky_scroll` | true / false / false / true | [916-1001](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L916-L1001) |
| L-128 | `project_panel.indent_guides.show` / `scrollbar.show` / `scrollbar.horizontal_scroll` | "always" / null(편집기 설정 상속) / true | [920-960](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L920-L960) |
| L-129 | `resize_all_panels_in_dock` | ["left"] | [879](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L879) |
| L-130 | `tab_bar.show` / `show_nav_history_buttons` / `show_tab_bar_buttons` | true / true / true | [1475-1485](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1475-L1485) |
| L-131 | `tabs.close_position` / `file_icons` / `show_close_button` / `activate_on_close` / `git_status` | "right" / false / "hover" / "history" / false | [1495-1531](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1495-L1531) |
| L-132 | `preview_tabs.enabled` / `enable_preview_from_project_panel` / `enable_preview_from_file_finder` | true / true / false | [1534-1543](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1534-L1543) |
| L-133 | `max_tabs` | null (제한 없음) | [1473](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1473) |
| L-134 | `status_bar.experimental.show` / `cursor_position_button` / `active_language_button` / `show_active_file` | true / true / true / false | [2031-2039](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2031-L2039) |
| L-135 | `line_indicator_format` | "long" | [2945](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2945) |
| L-136 | `markdown_preview.font_size` / `heading_font_weight` / `limit_content_width` / `max_width` / `open_markdown_files_in_preview` | null / 600 / true / 800 / false | [117-134](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L117-L134) |
| L-137 | `active_pane_modifiers.border_size` / `inactive_opacity` | 0.0 / 1.0 (활성 Pane 강조 없음) | [88-93](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L88-L93) |
| L-138 | `bottom_dock_layout` / `pane_split_direction_horizontal` / `_vertical` | "contained" / "down" / "right" | [97-101](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L97-L101) |
| L-139 | `zoomed_padding` | true | [253](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L253) |
| L-140 | `unstable.ui_density` | 기본 Default (설정 키 이름과 노출 여부는 미확인) | [ui_density.rs:21-28](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/ui_density.rs#L21-L28) |

---

## 9. 확신 낮은 부분

- **줄 번호 정확도:** 일부 링크의 줄 범위는 grep 결과로 잡은 근사치다. 특히 L-17, L-48, L-74, L-86, L-97, L-104, L-105, L-109, L-111이 그렇다. 파일과 커밋은 정확하다.
- **프로젝트 패널 항목 높이(L-36):** 소스에 명시값이 없다. "14px × phi + 테두리 2 ≈ 25~26"은 추정이고, 기준 화면 관찰값 26과만 맞춰 봤다.
- **들여쓰기 관찰값(L-37):** 화면에서 잰 단계 폭(약 17~18)이 설정값 20과 다르다. 측정 오차인지, 사용자 설정이나 다른 보정 때문인지는 미확인이다.
- **제목 여백(L-104):** 요소에 먼저 `mt_4`/`mb_2`(H1 외에는 `mt_6`)를 준 뒤 스타일 margin 24/12로 덮어쓴다. 최종값을 24/12로 본 것은 refine 순서에 근거한 추정이다.
- **tailwind 단위(L-10):** `_1` = 4px 같은 gpui 단위 대응은 관례에 근거한 추정이다. gpui `styles` 매크로 원문은 확인하지 않았다.
- **사이드바 푸터 배치(L-95, L-96):** 기준 화면 왼쪽 아래 아이콘들(토글·시계·폴더 추가)이 모두 사이드바 푸터 소속인지는 화면 관찰과 코드 조각으로 맞춘 것이다. 순서와 정렬은 미확인이다.
- **프로젝트 패널 파일 아이콘 크기(L-40):** 16px로 봤지만, 프로젝트 패널에서 `Icon::from_path`에 크기를 따로 주는지는 확인하지 못했다.
- **사용자 조정값(L-32, L-33):** 기준 화면 사이드바(약 200)와 프로젝트 패널(약 285) 폭은 사용자가 조정한 값으로 보인다. 저장 위치(DB `dock_panel_size`)는 확인하지 않았다.
- **기준 화면 아이콘:** 폴더·MD 아이콘은 사용자 아이콘 테마 `Catppuccin Latte`(확장)의 것이다. Zed 기본 아이콘 테마와 모양이 다르다 (테마·아이콘 담당 주제와 교차 확인 필요).
- **배율:** 기준 화면 배율 200%는 타이틀 바 32px에서 역산한 것이다. Windows 디스플레이 설정으로 확인하지 않았다.
- **`ui_density` 설정 키:** 설정 키 이름과 공개 여부를 default.json에서 찾지 못했다.
