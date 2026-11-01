---
tags: [zed, layout, gpui, tauri, spec, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 01-zed · 01 · 화면 구조와 치수

> 이 파일을 읽는 때: 화면을 Zed와 같은 치수로 그리기 전에. 단위, 영역 배치, 영역별 px 값, 마크다운 미리보기 스타일을 찾을 때.

| 항목 | 내용 |
| --- | --- |
| 근거 | Zed 커밋 [14b7b0e](https://github.com/zed-industries/zed/commit/14b7b0ef3764d62a85070c8b5ccea105a2ad3235) (2026-10-30). 아래 링크는 모두 이 커밋에 고정 |
| 검증 | 소스 대조 / 실행 검증 안 함 |
| 단위 | 논리 px (배율 100%). 표의 "관찰"은 기준 화면 측정값 |
| 라이선스 | Zed 앱 코드는 GPL-3.0. 값·이름·동작만 적고 코드 원문은 옮기지 않았다 |
| ID | `L-NN`은 조사 자료의 항목 ID를 그대로 쓴다. 새 항목은 L-141부터. 4.8절(앱 메뉴 모양)·4.1절(∨ 버튼)·4.5절(분할선·드롭 오버레이)·4.9절(컨텍스트 메뉴 모양)은 각 조사의 `I-NNN` ID를 그대로 쓴다 |

입구는 [../README.md](../README.md)다. 다음 문서는 [02-INTERACTION.md](02-INTERACTION.md)다.

## 1. 기준 화면

![기준 화면](assets/zed-reference.png)

- 이미지는 2880×1704 물리 px다. 타이틀 바가 64 물리 px이고 Windows 타이틀 바는 32 논리 px 고정(L-21)이므로 **배율 200%**로 보고, 논리 px = 물리 px ÷ 2로 환산했다. 배율은 역산이며 Windows 설정으로 확인하지 않았다.
- 기준 화면은 사용자 설정 결과다. 프로젝트 패널을 왼쪽에 둔 것(`project_panel.dock = "left"`)과 폭 약 200(사이드바)·약 285(패널)는 Zed 기본값이 아니다. 기본값은 8절에 있다.
- 기준 화면 설정 (Mocha 화면): 테마 Catppuccin Mocha, 아이콘 테마 Catppuccin Latte, `ui_font_size` 16, `buffer_font_size` 15, 창 최대화. 두 번째 화면(Latte)은 1.1절.

### 1.1 두 번째 기준 화면 (Catppuccin Latte, 창 최대화 안 함)

![Latte 기준 화면](assets/zed-reference-latte.png)

첫 번째 화면(Mocha)과 같은 앱·같은 레이아웃이고, 아래 차이만 다르다. 값 ID는 새로 붙였다 (L-310~L-315).

| ID | 항목 | 내용 | 출처 |
| --- | --- | --- | --- |
| L-310 | 화면 설정 | 이미지 2448×1480 물리 px. 타이틀 바가 약 64 물리 px이므로 **배율 200%**, 논리 1224×740으로 본다 (Mocha 화면과 같은 역산). 테마 **Catppuccin Latte** (라이트), 아이콘 테마 Catppuccin Latte, 창은 **최대화하지 않은 상태** | 관찰 |
| — | (참조 L-311) 창 모서리·테두리 | 최대화하지 않은 창은 바깥 모서리가 **10px 둥글고** 창 둘레에 **1px `border` 색 테두리**가 있다. Mocha 화면(최대화)에는 둥근 모서리와 바깥 테두리가 없다. 값과 규칙은 3절 "영역 경계선과 창 테두리"의 L-311 | 3절 |
| L-312 | 창 버튼 모양 | 최대화하지 않으면 가운데 버튼이 **최대화 □**(U+E922), 최대화하면 **복원**(겹친 □, U+E923)이다. Latte 화면은 □, Mocha 화면은 복원 모양이다. 닫기·최소화는 같다 (L-83) | [platform_windows.rs:45-53](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs#L45-L53) · [platform_windows.rs:78-84](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs#L78-L84) |
| L-313 | 라이트 테마 색 | 색 토큰 이름은 Mocha와 같고 값만 다르다. 설치된 Catppuccin 테마 파일(MIT)의 Latte 값: `title_bar.background`·`tab_bar.background`·`status_bar.background` **#dce0e8**, `panel.background` **#e6e9ef**, `editor.background`·`tab.active_background` **#eff1f5**, `tab.inactive_background` **#d2d7e2**, `text` **#4c4f69**, `text.muted` **#5c5f77**, `text.disabled`·`hidden` **#9ca0b0**, `text.accent` **#8839ef**. Picker 조사가 Latte 화면에서 잰 값 중 패널 `#ccd0da`, 비활성 탭 `#dce0e8`은 테마 파일 값과 다르다. **테마 파일 값을 따른다.** 전체 표는 [03-THEME-ICONS.md](03-THEME-ICONS.md) | 03, 테마 파일 |
| L-314 | 비활성 탭 글자 | 편집기 탭과 마크다운 미리보기 탭의 비활성 글자색이 **다르다.** Latte 화면에서 "Preview README.md" 탭이 "README.md" 탭보다 훨씬 옅다. 원인과 규칙은 4.11절의 L-306 | 4.11절 |
| L-315 | 커서 위치 "1:1" | 상태 표시줄 오른쪽 "1:1" (줄:열, 1부터). 마크다운 미리보기 탭이 활성이어도 커서 위치가 계속 보인다 (관찰, Mocha 화면의 "9:11"도 같다). 형식은 L-97 | 관찰 |

## 2. 단위 체계

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-01 | rem 크기 | 창의 rem = `ui_font_size` (기본 16px). 시작할 때 이 값을 창의 rem으로 넣는다. gpui 창 기본 rem도 16px | [settings.rs:605-612](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/settings.rs#L605-L612) · [window.rs:2216](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/window.rs#L2216) |
| L-02 | px → rem 변환 | `rems_from_px(px)` = px ÷ 16. 결과가 rem이라 `ui_font_size`를 바꾸면 비례해 커진다 | [units.rs:4-14](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/units.rs#L4-L14) |
| L-03 | 간격 `DynamicSpacing::BaseXX` | XX = 기본 밀도·rem 16일 때 px. 단일 값 n은 (Compact n−4, Default n, Comfortable n+4). 튜플 값은 그대로. 표는 (Compact, Default, Comfortable) | [spacing.rs:29-44](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/spacing.rs#L29-L44) · [dynamic_spacing.rs:69-83](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui_macros/src/dynamic_spacing.rs#L69-L83) |
| L-04 | UI 밀도 | `UiDensity`: Compact 0.75 / **Default 1.0** / Comfortable 1.25. 우리 앱은 Default만 쓴다 (판단) | [ui_density.rs:21-41](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/ui_density.rs#L21-L41) |
| L-05 | 글자 크기 | Large 16 / **Default 14** / Small 12 / XSmall 10 (모두 rem 변환). Ui = `ui_font_size`, Editor = `buffer_font_size`. 라벨 기본 = Default(14) | [typography.rs:134-156](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/typography.rs#L134-L156) · [label_like.rs:7-16](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/label/label_like.rs#L7-L16) |
| L-06 | 아이콘 크기 `IconSize` | Indicator 10 / XSmall 12 / **Small 14** / Medium 16 (기본) / XLarge 48 | [icon.rs:57-80](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/icon.rs#L57-L80) |
| L-07 | 버튼 높이 `ButtonSize` | Large 32 / Medium 28 / **Default 22** / Compact 18 / None 16. 좌우 안쪽 여백: Large·Medium은 Base08(8), Default·Compact는 Base04(4). 내부 gap Base04(4) | [button_like.rs:455-471](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L455-L471) · [button_like.rs:805-822](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L805-L822) |
| L-08 | 글꼴 기본값 | UI `.ZedSans` 400, **16px**. 버퍼 `.ZedMono` **15px**, `buffer_line_height: "comfortable"`. 글꼴 크기 제한 6~100px | [default.json:31-71](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L31-L71) · [settings.rs:18-19](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/settings.rs#L18-L19) |
| L-09 | 기본 줄 높이 | gpui 기본 `phi()` ≈ 1.618. 14px 라벨 한 줄 ≈ 22.6px | [style.rs:494](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/style.rs#L494) |
| L-10 | tailwind식 단위 | `_px` = 1, `_0p5` = 2, `_1` = 4 (0.25rem), `_1p5` = 6, `_2` = 8 (0.5rem), `_4` = 16, `_6` = 24 (rem 16 기준). `rounded_sm` = 0.25rem = 4px, `rounded_lg` = 0.5rem = 8px는 스타일 매크로에서 확인했다. 나머지는 같은 규칙으로 계산한 값이다 | [styles.rs:940-941](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui_macros/src/styles.rs#L940-L941) · [styles.rs:1242-1243](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui_macros/src/styles.rs#L1242-L1243) · [styles.rs:1252](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui_macros/src/styles.rs#L1252) |

`DynamicSpacing` 값 목록 (L-03). 문서에 나오는 것만 적는다.

| ID | 이름 | Compact | Default | Comfortable |
| --- | --- | --- | --- | --- |
| — | (참조 L-03) Base00 | 0 | 0 | 0 |
| — | (참조 L-03) Base01 | 1 | 1 | 2 |
| — | (참조 L-03) Base02 | 1 | 2 | 4 |
| — | (참조 L-03) Base03 | 2 | 3 | 4 |
| — | (참조 L-03) Base04 | 2 | 4 | 6 |
| — | (참조 L-03) Base06 | 3 | 6 | 8 |
| — | (참조 L-03) Base08 | 4 | 8 | 10 |
| — | (참조 L-03) Base12 | 10 | 12 | 14 |
| — | (참조 L-03) Base16 | 14 | 16 | 18 |
| — | (참조 L-03) Base20 | 18 | 20 | 22 |

Base24·Base32·Base40·Base48은 정의돼 있고 Default는 24·32·40·48이다 (L-03의 단일 값 규칙).

## 3. 전체 배치

| ID | 단위 | 내용 | 출처 |
| --- | --- | --- | --- |
| L-11 | 창 최상위 `MultiWorkspace` | 가로 배치: **[왼쪽 사이드바] → [Workspace (flex_1)] → [오른쪽 사이드바]**. 그 위에 modal·overlay 층. 사이드바 위치는 `agent.threads_sidebar.position` (기본 left). 우리 앱은 왼쪽 사이드바만 둔다 | [multi_workspace.rs:2205-2215](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/multi_workspace.rs#L2205-L2215) · [default.json:1191-1197](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1191-L1197) |
| L-12 | 사이드바는 창 전체 높이 | 사이드바 헤더 높이 = 타이틀 바 높이라서 사이드바가 타이틀 바 줄까지 차지한다. Workspace의 타이틀 바는 사이드바 오른쪽에서 시작한다 (기준 화면과 같다) | [sidebar.rs:7387-7390](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7387-L7390) |
| L-13 | `Workspace` 세로 배치 | **타이틀 바 → 본문 → 상태 표시줄 → toast 층** | [workspace.rs:9851](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L9851) · [workspace.rs:10279-10282](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L10279-L10282) |
| L-14 | 본문 (`bottom_dock_layout: "contained"`) | 가로: **왼쪽 Dock → [세로: 가운데(PaneGroup) → 아래 Dock] → 오른쪽 Dock**. 우리 앱은 왼쪽 Dock과 가운데만 둔다 | [workspace.rs:10180-10228](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L10180-L10228) · [default.json:97](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L97) |
| L-15 | 가운데 영역 | `PaneGroup`(분할 나무) → 각 `Pane` = **탭 바 + 활성 아이템(편집기·미리보기)**. 분할 방향 기본: 가로 분할은 down, 세로 분할은 right | [workspace.rs:9086](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L9086) · [default.json:99-101](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L99-L101) |
| L-16 | `Dock` | 위치 Left/Right/Bottom. 패널 여러 개를 담고 활성 패널 하나만 표시. 테두리: 왼쪽 Dock `border_r_1`, 오른쪽 `border_l_1`, 아래 `border_t_1` | [dock.rs:1356-1358](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1356-L1358) |
| L-17 | 확대 (zoom) | 확대한 Pane·패널은 본문 위 overlay로 그린다. `zoomed_padding: true`이면 사방 8px 여백 + `border_1` | [workspace.rs:10238-10275](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L10238-L10275) · [default.json:253](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L253) |

우리 앱의 나무 구조 (기준 화면, 사이드바 왼쪽, 프로젝트 패널 왼쪽 Dock). 괄호 안은 높이·폭이다.

```
창 (h_flex)
├─ 사이드바 (전체 높이, 폭 200~800, border_r_1)
│   ├─ 헤더 (높이 32 = 타이틀 바와 같음)
│   ├─ 목록 (프로젝트 헤더 행 31 + 항목)
│   └─ 푸터 (p_1, border_t_1, 높이 = 상태 표시줄): 사이드바 토글 · 폴더 추가
└─ Workspace (v_flex, flex_1)
    ├─ 타이틀 바 (32): ☰ 메뉴 · 폴더 이름 · … · ∨ 메뉴 · 창 버튼
    ├─ 본문 (h_flex)
    │   ├─ 왼쪽 Dock → 프로젝트 패널 (폭 240 기본, border_r_1, 오른쪽에 리사이즈 핸들 6)
    │   └─ 가운데 (v_flex) → PaneGroup → Pane
    │         ├─ 탭 바 (32)
    │         └─ 편집기 또는 마크다운 미리보기 (남은 높이)
    └─ 상태 표시줄 (≈30): 프로젝트 패널 토글 … 커서 위치
```

**영역 경계선과 창 테두리**

기준 화면 두 장에서 보이는 1px 선과 창 테두리다. 색은 모두 `border` 토큰이다. 값은 소스에서 확인했다.

| ID | 선 | 값 | 출처 |
| --- | --- | --- | --- |
| L-301 | 타이틀 바 아래 선, 상태 표시줄 위 선 | 작업 영역 상자(타이틀 바와 상태 표시줄 사이)가 **위·아래에 `border_t_1`, `border_b_1`(1px, `border`)**을 갖는다. 그래서 타이틀 바 아래(기준 화면 y = 32)와 상태 표시줄 위(y = 821, Mocha 화면)에 선이 생긴다. 타이틀 바와 상태 표시줄이 직접 그리는 선이 아니다 | [workspace.rs:9895-9910](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L9895-L9910) |
| L-302 | 사이드바 헤더 아래 선 | 클라이언트 장식일 때 사이드바 헤더 아래에 `border_b_1`을 둔다. 타이틀 바 아래 선과 한 줄로 이어진다 (주석: 타이틀 바의 아래 테두리와 맞춘다) | [sidebar.rs:7410-7418](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7410-L7418) |
| L-303 | 사이드바 오른쪽 선 | 사이드바가 왼쪽에 있으면 `border_r_1` | [sidebar.rs:8043-8044](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L8043-L8044) |
| L-304 | 상태 표시줄 자체 테두리 | 상태 표시줄은 `border_b(1px)`을 갖지만 색이 `status_bar_background`라 **보이지 않는다**(둥근 모서리 사이 빈틈을 막는 용도). 아래 여백 −1px. 눈에 보이는 위쪽 선은 L-301이다 | [status_bar.rs:150-183](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L150-L183) |
| L-311 | 창 모서리·테두리·그림자 (최대화하지 않음) | 클라이언트 장식에서 변이 맞닿지 않은(tiled 아닌) 쪽마다 창 둘레에 **1px 테두리**(`border`), 모서리 **10px** 둥글림, 창 바깥 **10px 그림자 영역**을 둔다. Picker 조사(P-028)는 둥글림을 8px로 적었으나 고정 커밋의 상수(`CLIENT_SIDE_DECORATION_ROUNDING`)는 **10px**이라 소스를 따른다. 타이틀 바 위쪽 모서리와 상태 표시줄 아래쪽 모서리도 같은 10px로 둥글린다 (사이드바가 그 모서리를 차지하면 사이드바가 둥글린다). 최대화하면 변이 모두 맞닿은 것으로 보아 둥글림과 테두리가 없다 (Windows 최대화의 장식 처리는 추정) | [theme.rs:52-57](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/theme.rs#L52-L57) · [workspace.rs:12232-12350](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L12232-L12350) · [platform_title_bar.rs:291-307](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platform_title_bar.rs#L291-L307) |

## 4. 영역별 치수

### 4.1 타이틀 바

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-21 | 높이 | **Windows 32 고정**. 다른 OS는 max(1.75 × rem, 34). 관찰 32 | [constants.rs:15-24](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/utils/constants.rs#L15-L24) |
| L-81 | 그리는 방식 | 기본 `window_decorations: "client"`. OS 타이틀 바 대신 앱이 직접 그린다. 바 전체가 창 이동 영역(Drag) | [default.json:282](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L282) · [platform_title_bar.rs:219-221](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platform_title_bar.rs#L219-L221) |
| L-82 | 왼쪽 여백 | 시작 padding `pl_2` = 8 (macOS 신호등 여백은 제외) | [platform_title_bar.rs:272-287](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platform_title_bar.rs#L272-L287) |
| L-87 | 왼쪽 묶음 순서 | [☰ 메뉴] → **[프로젝트 이름]**. (Zed는 제한 모드 경고·원격 호스트·Git 브랜치도 있으나 제외). 묶음 gap `_0p5` = 2 | [title_bar.rs:298-345](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L298-L345) |
| L-85 | ☰ 메뉴 | Windows에서 `show_menus: false`(기본)이면 메뉴 막대 대신 **버튼 하나**: 아이콘 `Menu`, Subtle 스타일, Small(14), tooltip "Open Application Menu". 누르면 첫 메뉴가 열리고 **열려 있는 동안 가로 메뉴 바**(앱 이름, File, Edit, …)가 나타난다. 계층형 세로 메뉴가 아니다 (I-302). 크기·색은 4.8절, 구조는 [02-INTERACTION.md](02-INTERACTION.md) 9절 | [title_bar.rs:453-462](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L453-L462) · [application_menu.rs:176-200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L176-L200) |
| L-86 | 우리 메뉴에 넣을 항목 | 확정했다. 최종 구조는 [02-INTERACTION.md](02-INTERACTION.md) 9.2절 (앱 이름, File, Edit, Selection, View, Go, Window, Help. Run은 뺀다). 뺀 항목과 이유는 9.4절 | [app_menus.rs:69-340](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/app_menus.rs#L69-L340) |
| L-88 | 프로젝트 이름 | 활성 루트 폴더 이름. **40자**에서 말줄임. 누르면 **Recent Projects 팝업**(최근 폴더 + 등록 폴더 목록)이 열린다 (사용자 결정으로 Open Recent 포함, 02 L-226) | [title_bar.rs:64](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L64) · [title_bar.rs:783-860](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L783-L860) |
| L-39 | 창 버튼 크기 | 각 **36×32**, 글리프 10px | [platform_windows.rs:134-136](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs#L134-L136) |
| L-83 | 창 버튼 그리기 | 최소화·최대화(또는 복원)·닫기, 오른쪽 끝. **글꼴 글리프**로 그린다. Windows 11(빌드 22000 이상)은 `Segoe Fluent Icons`, 그 이전은 `Segoe MDL2 Assets`. 코드포인트: 최소화 U+E921, 최대화 U+E922, 복원 U+E923, 닫기 U+E8BB. 각 버튼에 영역(Min/Max/Close)을 지정해 OS 히트테스트(스냅 레이아웃)에 연결 | [platform_windows.rs:15-95](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs#L15-L95) |
| L-84 | 창 버튼 색 | 닫기 hover 배경 **#E81120**(232,17,32) + 흰 글자. 누름은 각각 80% 불투명. 나머지 hover `ghost_element_hover`, 누름 `ghost_element_active`, 글자 `text` | [platform_windows.rs:104-128](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/platform_title_bar/src/platforms/platform_windows.rs#L104-L128) |
| L-89 | 오른쪽 묶음 | Zed는 통화·연결 상태·업데이트·Sign In·사용자 메뉴(∨)가 있다. **우리 앱은 [∨ 메뉴 버튼] + 창 버튼만** 둔다. Sign In 버튼, 통화, 연결 상태, 업데이트 알림은 뺀다 (사용자 결정). 묶음은 오른쪽 padding `pr_1` = 4, gap 4 | [title_bar.rs:367-397](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L367-L397) |

**∨ 버튼과 메뉴 모양.** 사용자 요건: Sign In은 빼고 옆의 ∨ 드롭다운 메뉴는 채용한다. 값은 ∨ 메뉴 조사(`zed-user-menu.md`)의 ID(I-600~I-612)를 그대로 쓴다. 항목 구성·툴팁·최종 구조는 [02-INTERACTION.md](02-INTERACTION.md) 10절에 있다. 계산 기준은 UI 글꼴 16px, 밀도 Default, 배율 100%다. 기준 화면에서 ∨는 Sign In 글자 오른쪽, 최소화 버튼 왼쪽에 있다.

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| I-600 | 구성 | 팝업 메뉴(식별자 `user-menu`)와, 그것을 여는 **일반 버튼**(아이콘 버튼이 아니다)으로 이루어진다. 버튼 안에 아이콘을 넣는 구조다 | [title_bar.rs:1204-1267](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1204-L1267) |
| I-601 | 아이콘 | 로그인하지 않았을 때(우리 앱은 항상) 아이콘 `ChevronDown`, 크기 Small = **14px**. 접근성 이름 "User menu", 탭 순서 0 | [title_bar.rs:1230-1263](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1230-L1263) |
| I-602 | Sign In과 독립 | Sign In 표시와 ∨ 표시는 설정(`show_sign_in`, `show_user_menu`)이 따로다. Sign In을 빼도 ∨는 혼자 성립한다. 우리 앱은 인증 코드 없이 같은 모양의 ∨ 버튼만 만든다 (판단) | [title_bar.rs:350-395](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L350-L395) |
| I-603 | 배치 | 오른쪽 묶음 안에서 (Sign In이 있으면 그 다음) ∨가 오고, 그 뒤에 창 버튼이 온다. 묶음 gap 0.25rem = **4px**, 오른쪽 padding 0.25rem = **4px**. Sign In을 빼도 그 자리에 빈 칸을 남기지 않는다 | [title_bar.rs:366-397](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L366-L397) · [units.rs:3-15](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/units.rs#L3-L15) |
| I-604 | 로그인 전 분기 | 현재 사용자가 없으면 ∨(`ChevronDown`)를 쓴다. 메뉴의 사용자 이름·조직·로그아웃 줄이 빠진다. 설정·키맵·테마 줄은 로그인과 무관하다 | [title_bar.rs:1209-1215](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1209-L1215) · [title_bar.rs:1258-1263](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1258-L1263) · [title_bar.rs:1280-1421](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1280-L1421) |
| I-605, I-606 | 로그인 후·진행 중 | 아바타(∨ 대신), 조직 이름 라벨, 업데이트 점, "Signing in…" 애니메이션 라벨은 **만들지 않는다** (계정 기능 없음) | [title_bar.rs:1216-1263](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1216-L1263) · [title_bar.rs:350-359](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L350-L359) · [title_bar.rs:380-395](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L380-L395) |
| I-607 | 설정 기본값 | `title_bar.show_user_picture` = true, `show_user_menu` = true, `show_sign_in` = true. 우리 앱은 Sign In만 제거하고 사용자 메뉴는 유지한다 (사용자 결정) | [default.json:628-636](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L628-L636) |
| I-608 | 버튼 크기 | 아이콘 14px, 버튼 높이 22px (Default 버튼), 좌우 padding Base04 = 4px씩 → 폭 14 + 4 + 4 = **22px**, 높이 **22px**. 테두리 없는 기본 스타일 기준 | [icon.rs:72-80](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/icon.rs#L72-L80) · [button_like.rs:455-471](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L455-L471) · [button_like.rs:517-531](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L517-L531) · [button_like.rs:802-825](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L802-L825) · [spacing.rs:28-39](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/spacing.rs#L28-L39) |
| I-609 | 모서리·색 | 스타일 Subtle, 모서리 0.25rem = **4px**. 배경 `ghost_element_background`, hover `ghost_element_hover`, 누름 `ghost_element_active`. 테두리 없음 | [button_like.rs:124-147](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L124-L147) · [button_like.rs:242-247](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L242-L247) · [button_like.rs:298-303](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L298-L303) · [button_like.rs:324-328](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L324-L328) · [button_like.rs:809-849](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L809-L849) · [styles.rs:1240-1243](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui_macros/src/styles.rs#L1240-L1243) |
| I-610 | 열림 상태 | 메뉴가 열려 있는 동안 버튼이 **선택된 상태**(toggle)로 그려진다. 열린 상태에서 버튼을 다시 누르면 닫히기만 하고 바로 다시 열리지 않는다. 메뉴를 닫을 때 포커스가 메뉴 안에 있었으면 열기 전의 포커스로 되돌린다 | [popover_menu.rs:182-194](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L182-L194) · [popover_menu.rs:274-312](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L274-L312) · [popover_menu.rs:480-493](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L480-L493) |
| I-611 | 팝업 위치 | 메뉴의 **오른쪽 위** 모서리를 버튼의 오른쪽 아래에 맞춘다. x 오프셋 = +(5/16 × rem) = 기준 rem 16에서 **+5px**, y 오프셋 0. 창 가장자리 여유 8px. 메뉴 내부 rem과 타이틀 바 rem이 다를 수 있어 실제 화면 좌표는 미측정 | [title_bar.rs:1424-1425](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1424-L1425) · [popover_menu.rs:245-269](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L245-L269) · [popover_menu.rs:371-384](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L371-L384) |
| I-612 | 팝업 공통 외형 | 최소 폭 **200px** (내용에 따라 증가, 테두리 포함 최소 바깥 폭 202px), 최대 높이 = 창 높이 × 0.75 (넘치면 세로 스크롤). 배경·테두리·그림자·행 여백은 ☰ 앱 메뉴와 같은 위젯이다 (4.8절 I-312~I-321). 단, ∨ 메뉴의 "Panel Layout"은 진짜 옆으로 열리는 하위 메뉴 구조를 쓴다 (우리 앱은 이 항목을 넣지 않는다) | [context_menu.rs:2218-2293](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2218-L2293) · [title_bar.rs:1388-1417](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs#L1388-L1417) |

### 4.2 사이드바 (등록 폴더 목록)

Zed에서는 threads 사이드바 자리다. 우리 앱에서는 등록 폴더 목록이다 (스레드 검색·세션·시간 제외).

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-33 | 폭 | Zed 기본 **300**, 범위 **200~800**. 두 기준 화면 모두 **200** (사용자가 최소값까지 줄인 값). **우리 앱 초기값 = 300** (소스 기본값). 끌어서 바꾸면 그 값을 저장하고 복원한다 (02 I-116). 기준 화면의 200은 사용자 조정값이라 초기값으로 쓰지 않는다 | [default.json:1191-1197](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1191-L1197) · [agent_settings.rs:37-38](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/agent_settings/src/agent_settings.rs#L37-L38) |
| — | (참조 L-12) 높이 | 창 전체. 헤더 높이 = 타이틀 바 높이(32) | [sidebar.rs:7387-7390](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7387-L7390) |
| L-34 | 프로젝트 헤더 행 | 높이 **31** (탭 내용 높이와 같음), 왼쪽 padding 8, 오른쪽 6. 관찰 약 31 | [sidebar.rs:2410-2413](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2410-L2413) |
| L-35 | 푸터 | `p_1`(4), gap 4, `border_t_1`, 아이콘 버튼 Small(14). 높이가 상태 표시줄과 같아 보인다 | [sidebar.rs:7533-7545](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7533-L7545) |
| L-95 | 사이드바 토글 | Zed는 사이드바가 **닫혀 있을 때만** 상태 표시줄 맨 왼쪽에 열기 버튼이 나오고, 열려 있으면 사이드바 푸터의 닫기 버튼을 쓴다. **우리 앱은 같은 화면 위치(맨 왼쪽 아래)에 버튼 하나를 두고 상태에 따라 아이콘만 바꾼다** (판단) | [status_bar.rs:198-202](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L198-L202) · [status_bar.rs:240-262](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L240-L262) · [sidebar.rs:7488-7500](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L7488-L7500) |
| L-96 | 폴더 추가 버튼 | 아이콘 `FolderAdd`, Small(14), tooltip "Add Project". 기준 화면에서는 사이드바 푸터 오른쪽 끝 | [sidebar.rs:6770-6778](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L6770-L6778) |

선택 강조: 선택 항목 배경은 `element_selected` 계열 토큰이다 (색은 [03-THEME-ICONS.md](03-THEME-ICONS.md)). 기준 화면의 선택 행은 높이 약 46(= 31 + 보조 줄)로 보이나, 보조 줄(시간)은 제외 대상이라 **우리 항목 행 높이는 31로 정한다 (판단)**.

### 4.3 왼쪽 Dock과 프로젝트 패널

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| — | (참조 L-16) Dock 테두리 | 왼쪽 Dock `border_r_1` | [dock.rs:1356-1358](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1356-L1358) |
| L-30 | 리사이즈 핸들 | **6px**, 테두리 위에 절반씩 걸친다(−3px 위치). 커서 `col_resize`. **더블클릭하면 기본 크기로 초기화** | [dock.rs:25](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L25) · [dock.rs:1281-1336](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1281-L1336) |
| L-31 | Dock 폭 한계 | 하한은 핸들 6px, 상한은 창 폭 − 반대쪽 Dock − 6 | [dock.rs:51-53](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L51-L53) · [workspace.rs:9192-9227](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L9192-L9227) |
| L-32 | 프로젝트 패널 기본 폭 | Zed 기본 **240**. 기준 화면은 Mocha 약 284, Latte 약 215로 **화면마다 다르다**(사용자 조정값). **우리 앱 초기값 = 240** | [default.json:888](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L888) |
| L-36 | 항목 높이 | **26 = 24 + 2.** 이름 줄 높이는 `h_6()` = 1.5rem = **24px**이고, 목록 항목(ListItem)의 안쪽 상자가 위아래 `border_1`(1px)씩 **2px**을 더한다. 간격 설정 `comfortable`이면 추가 padding 0 (`standard`는 위아래 −1px). 기준 화면 관찰 26과 일치한다. 이전 문서의 "25~26"(14 × 1.618 + 2 계산)과 "24"(`h_6`만 본 값)는 둘 다 일부만 센 값이다 | [project_panel.rs:6628-6634](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6628-L6634) · [project_panel.rs:6785-6788](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6785-L6788) · [list_item.rs:360-368](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L360-L368) |
| L-37 | 들여쓰기 | 단계당 **20** (`indent_size`). 항목 margin-left = 깊이 × 20. 항목 안쪽 좌우 padding Base06(6), gap 4. 관찰: 이미지를 다시 재면 폴더 아이콘이 단계마다 약 20씩 밀린다 (조사 자료는 17~18) | [default.json:907](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L907) · [list_item.rs:363-364](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L363-L364) |
| L-38 | 들여쓰기 안내선 | 폭 **1px**, x = 깊이 × `indent_size` + **15**, 세로 길이 = 자식 수 × 항목 높이 (위아래 4px 여백). 색 3단계: `panel_indent_guide`(기본) / `_hover` / `_active`(선택 항목이 속한 안내선) | [indent_guides.rs:22-33](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/indent_guides.rs#L22-L33) · [project_panel.rs:7815-7850](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7815-L7850) |
| L-40 | 파일·폴더 아이콘 크기 | Medium 16 (추정). 관찰 약 16 | [pane.rs:2824-2834](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2824-L2834) |
| L-41 | 루트 표시 | 워크트리 루트를 맨 위 항목으로 그린다 (기준 화면의 "projects"). `hide_root: true`이고 폴더가 하나뿐일 때만 숨긴다 (기본 false) | [project_panel.rs:4636-4665](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L4636-L4665) · [default.json:999](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L999) |
| L-42 | 경로 접기 (`auto_fold_dirs`, 기본 true) | 디렉터리의 자식이 **정확히 하나이고 그 자식도 디렉터리**이면 한 줄로 합친다. 이 규칙을 연쇄 적용 (예: `docs \ guides`) | [project_panel.rs:1468-1479](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1468-L1479) · [project_panel.rs:4677-4690](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L4677-L4690) |
| L-43 | 접힌 경로 표시 | 조각마다 따로 hover되는 요소: 좌우 padding 2, `rounded_xs`, hover 시 `element_active` 배경. 구분자는 OS 경로 구분자(Windows `\`). 조각을 누르면 그 단계를 선택. 기준 화면의 "docs \ guides" 사이 공백은 이 여백으로 보인다 (추정) | [project_panel.rs:6884-6905](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6884-L6905) |
| L-44 | 펼침 표시 | `folder_indicator` 기본 **"icon"**: 화살표 없이 폴더 아이콘(닫힘·열림)만. 기준 화면과 같다 | [default.json:896-904](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L896-L904) |
| L-45 | 가로 스크롤 | `scrollbar.horizontal_scroll` 기본 true. 가장 긴 항목 폭으로 목록 폭을 잡아 가로로 스크롤한다. 기준 화면 아래의 가로 막대 | [default.json:936-938](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L936-L938) · [project_panel.rs:7949-7954](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7949-L7954) |
| L-46 | 가상화 | 목록은 `uniform_list`(모든 행 높이 같음)로 그린다. 우리도 행 높이를 같게 고정한다 | [project_panel.rs:7732](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7732) |
| L-47 | 항목 색 | 기본 `panel_background`, hover `element_hover`, 선택 `element_selected`, 포커스 테두리 `panel_focused_border`, 드래그 대상 `drop_target_background`. sticky 행은 `panel_overlay_background` / `panel_overlay_hover` | [project_panel.rs:659-676](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L659-L676) |
| L-48 | 포커스 표시 | 항목은 모서리 없이 `border_1` + 오른쪽 `border_r_2`. 평소 테두리 색 = 배경색(안 보임). 패널에 키보드 포커스가 있고 마우스가 눌리지 않았으며 그 항목이 활성일 때만 테두리를 `panel_focused_border`로 칠한다 | [project_panel.rs:6180-6200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6180-L6200) · [project_panel.rs:6271-6277](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6271-L6277) |
| L-49 | 이름 tooltip | hover 1500ms 뒤 전체 경로 tooltip | [project_panel.rs:6254-6268](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6254-L6268) |
| L-50 | 정렬 | `sort_mode: "directories_first"`, `sort_order: "default"` (대소문자 무시 자연 정렬) | [default.json:971-990](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L971-L990) |
| L-51 | sticky scroll | 기본 true: 스크롤할 때 상위 폴더 행이 위에 붙는다. **제외** (결정 목록에 없음) | [default.json:950](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L950) |
| L-52 | 제외 (Git·진단) | `git_status`, `git_status_indicator`, `show_diagnostics`, `diagnostic_badges`는 넣지 않는다 | [default.json:905](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L905) · [default.json:948](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L948) · [default.json:993-995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L993-L995) |

### 4.4 탭 바

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-22 | 탭 바 높이 | **32** (`Base32`). 관찰 약 32 | [tab.rs:79-85](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L79-L85) · [tab_bar.rs:100](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab_bar.rs#L100) |
| L-23 | 탭 내용 높이 | **31** (= 32 − 아래 테두리 1) | [tab.rs:79-81](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L79-L81) |
| L-24 | 탭 안쪽 여백·간격 | 좌우 padding Base04 = **4**, 요소 gap Base04 = **4** | [tab.rs:172-174](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L172-L174) |
| L-25 | 탭 슬롯 | 시작 슬롯 **12×12** (수정 표시 점), 끝 슬롯 **14×14** (닫기·핀 버튼) | [tab.rs:8-9](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L8-L9) |
| L-26 | 탭 폭 | **고정 최소·최대 폭 없음** (내용 폭). 제목은 24자에서 말줄임 (`MAX_TAB_TITLE_LEN`). 관찰: "README.md" 탭 약 125 | [items.rs:74](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L74) · [markdown_preview_view.rs:1673-1683](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1673-L1683) |
| L-27 | 양 끝 버튼 영역 | gap 4, 좌우 padding Base06 = **6**, `border_b_1`. 시작 영역 `border_r_1`, 끝 영역 `border_l_1` | [tab_bar.rs:104-148](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab_bar.rs#L104-L148) |
| L-61 | 구성 | [시작 영역] + [탭 목록(가로 스크롤)] + [끝 영역]. 시작·끝 영역은 탭이 넘쳐도 고정 | [tab_bar.rs:96-150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab_bar.rs#L96-L150) |
| L-62 | 시작 영역 ← → | 이동 기록이 없으면 비활성. Small(14). tooltip에 GoBack/GoForward 단축키 (`show_nav_history_buttons` 기본 true) | [pane.rs:3508-3550](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3508-L3550) · [pane.rs:3604-3620](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3604-L3620) |
| L-63 | 탭 테두리 | 탭 위치(첫·가운데·마지막)와 활성 탭에 대한 상대 위치로 정한다. **활성 탭은 아래 테두리가 없어**(`pb_px`) 내용 영역과 이어지고, 비활성 탭은 `border_b_1`이다. 정확한 조건은 4.4절 아래 "탭 테두리 규칙" 표 | [tab.rs:145-166](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L145-L166) |
| L-64 | 탭 내용 순서 | [시작 슬롯: 수정 점] [아이콘] [제목] [끝 슬롯: 닫기]. `close_position: "left"`이면 두 슬롯을 맞바꾼다 | [tab.rs:127-141](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L127-L141) · [pane.rs:3086-3096](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3086-L3096) |
| L-65 | 수정 표시 | 점(`Indicator::dot`). 충돌은 Warning 색, 수정됨은 **Accent 색**, 깨끗하면 없음 | [pane.rs:5090-5099](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L5090-L5099) |
| L-66 | 닫기 버튼 | 아이콘 `Close`, Small, Muted, 정사각형. `show_close_button` 기본 **"hover"** (탭에 마우스를 올릴 때만). 핀 탭은 닫기 대신 Pin 아이콘 | [pane.rs:3033-3066](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3033-L3066) · [default.json:1511](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1511) |
| L-67 | 파일 아이콘 | `tabs.file_icons` 기본 **false**라 편집기 탭에는 아이콘이 없다 (기준 화면 "README.md" 탭과 일치). 아이템이 자체 아이콘을 주면 Small·Muted로 표시 | [items.rs:760-766](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L760-L766) · [pane.rs:2824-2834](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2824-L2834) |
| L-68 | 미리보기 탭 | 프로젝트 패널에서 한 번 눌러 연 파일. 제목이 **기울임꼴**이고 다른 미리보기를 열면 대체된다 | [items.rs:834](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L834) · [default.json:1534-1541](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1534-L1541) |
| L-69 | 마크다운 미리보기 탭 | 제목 "**Preview {파일명}**" (24자 말줄임), 아이콘 `FileDoc` (책 모양)이 항상 붙는다. 편집기가 없으면 "Markdown Preview". 같은 이름이 겹치면 XSmall·Muted 설명(디렉터리) 추가 | [markdown_preview_view.rs:1663-1712](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1663-L1712) |
| L-70 | 넘칠 때 | 탭 목록이 가로 스크롤이고 스크롤바는 없다. 탭을 활성화하면 그 탭이 보이게 이동. 마지막 탭 뒤에 빈 드롭 영역 24px. 기준 화면 오른쪽 끝 "Preview ("처럼 잘려 보인다 | [pane.rs:3694](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3694) · [pane.rs:1524](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L1524) · [pane.rs:3738](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3738) |
| L-71 | 끝 영역 `+` | 아이콘 `Plus`, tooltip "New…". Zed 메뉴(New File, Open File, Search Project, Search Symbols, New Terminal…) 중 **우리 앱은 New File만 남긴다** (판단) | [pane.rs:4360-4388](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4360-L4388) |
| L-72 | 끝 영역 분할 | 아이콘 `Split`, tooltip "Split Pane". 메뉴: Split Right / Left / Up / Down. 분할할 수 없으면 비활성 | [pane.rs:4388-4412](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4388-L4412) |
| L-73 | 끝 영역 최대화 | 아이콘 `Maximize`. 확대 중에는 `Minimize` 아이콘(선택 표시). tooltip "Zoom In" / "Zoom Out" | [pane.rs:4417-4430](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4417-L4430) |
| L-74 | 탭 누르기 | 가운데 클릭으로 닫기 (핀 탭 제외). 드래그로 순서 변경·다른 Pane 이동 | [pane.rs:2972-3030](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2972-L3030) |
| L-300 | 탭 바 아이콘 크기 | ←→ · + · 분할 · 최대화 · 탭의 아이콘 모두 Small(14). (이전 문서에서 L-40을 프로젝트 패널 아이콘 16과 함께 써서 번호를 나눴다) | [pane.rs:3508-3532](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3508-L3532) |

**탭 테두리 규칙 (L-63)**

탭 컨테이너의 테두리 색은 `border`다. 활성 탭은 아래 테두리 대신 `pb_px`(아래 1px 비움)이라 내용 영역과 이어진다. 탭의 위치와 활성 탭에 대한 상대 위치로 정한다. `px` = 1px padding이다.

| ID | 탭 위치 | 활성(선택) | 비활성 |
| --- | --- | --- | --- |
| — | (참조 L-63) 첫 탭 (First) | 왼쪽 padding 1, **오른쪽 테두리 1**, 아래 padding 1 | 왼쪽 padding 1, 오른쪽 padding 1, **아래 테두리 1** |
| — | (참조 L-63) 마지막 탭 (Last) | **왼쪽·오른쪽 테두리 1**, 아래 padding 1 | 왼쪽 padding 1, **아래·오른쪽 테두리 1** |
| — | (참조 L-63) 가운데 탭, 활성 탭 자신 | **왼쪽·오른쪽 테두리 1**, 아래 padding 1 | — |
| — | (참조 L-63) 가운데 탭, 활성 탭보다 **앞**(왼쪽) | — | **왼쪽 테두리 1**, 오른쪽 padding 1, **아래 테두리 1** |
| — | (참조 L-63) 가운데 탭, 활성 탭보다 **뒤**(오른쪽) | — | **오른쪽 테두리 1**, 왼쪽 padding 1, **아래 테두리 1** |

- 인접한 두 탭의 경계에는 선이 하나만 생기도록 짠 규칙이다 (활성 탭의 양옆 선이 이웃 탭의 선 역할을 한다). 첫 탭 왼쪽에 테두리가 없는 것은 시작 영역의 `border_r_1`이 대신하기 때문이다.
- 탭 바 시작 영역은 `border_b_1` + `border_r_1`, 끝 영역은 `border_b_1` + `border_l_1`이다 (L-27). 탭 목록 영역에는 전체 폭 `border_b_1` 선이 깔린다. 탭이 모자라 남는 폭에도 아래 선이 이어진다.
- 탭 안쪽: 높이 31(= `Base32` − 1), 좌우 padding 4, 요소 gap 4, 시작·끝 슬롯 12×12, 14×14 (L-23~L-25).

[tab.rs:145-166](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L145-L166) · [tab_bar.rs:98-160](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab_bar.rs#L98-L160)

### 4.5 편집기·Pane 영역

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| — | (참조 L-15) 구성 | 탭 바 아래 남은 영역 전체가 활성 아이템(편집기 또는 마크다운 미리보기) | [workspace.rs:9086](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L9086) |
| — | (참조 L-08) 편집기 글꼴 | `.ZedMono` 15px, 줄 높이 배수 **1.618** (`BufferLineHeight::Comfortable`) → 15 × 1.618 = 24.27px. 최종 반올림은 미확인. 메뉴·목록의 줄 높이도 같은 1.618을 고정으로 쓴다 | [default.json:31-54](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L31-L54) · [buffer_line_height.rs:14-21](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/buffer_line_height.rs#L14-L21) |
| L-137 | 활성 Pane 강조 | `active_pane_modifiers.border_size` 0.0, `inactive_opacity` 1.0: **강조 없음**. 우리 앱도 Pane 테두리·흐림 효과를 넣지 않는다 | [default.json:88-93](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L88-L93) |
| L-102 | 배경 | 편집기·미리보기 배경은 `editor_background` 토큰 | 5절 |

편집기 내부(거터, 줄 번호, 현재 줄, 커서, 선택, 들여쓰기 안내선, 스크롤바, 툴바, 찾기 창)의 값은 4.12절이다.

**분할선과 드롭 오버레이**

탭 분할·이동 조사(`zed-panes-keymap.md`)의 ID를 그대로 쓴다. 동작·판정 규칙은 [02-INTERACTION.md](02-INTERACTION.md) 3.1절에 있다. 색은 토큰 이름이고 값은 선택한 테마에서 가져온다.

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| I-725 | 분할선 | 두께 **1px**, 색 `pane_group_border`. 마우스 반응 영역 **4px**(경계 앞뒤 2px). 가로 나열은 `ResizeColumn` 커서, 세로 나열은 `ResizeRow` 커서 | [pane_group.rs:22-24](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L22-L24) · [pane_group.rs:1161](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1161) · [pane_group.rs:1322-1340](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1322-L1340) · [pane_group.rs:1530-1549](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1530-L1549) |
| I-726 | Pane 최소 크기 (드래그) | 폭 **80px**, 높이 **100px** | [pane_group.rs:22-24](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L22-L24) · [pane_group.rs:1224-1319](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1224-L1319) |
| I-712 | 분할 드롭 판정 폭 | min(본문 폭, 본문 높이) × **0.2** (`drop_target_size`). 탭 바는 본문에 넣지 않는다. 예: 800×600 → 120px | [pane.rs:3839-3871](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3839-L3871) · [default.json:206-210](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L206-L210) |
| I-714 | 본문 드롭 오버레이 | 배경 `drop_target_background`. 중앙이면 본문 전체, 방향 분할이면 해당 쪽 **50%**를 덮는다. **테두리·모서리 없음.** 판정 폭(20%)과 보이는 폭(50%)이 다르다 | [pane.rs:4684-4732](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L4684-L4732) |
| I-715 | 탭 위 드롭 표시 | 배경 `drop_target_background`, 경계 `drop_target_border`. 드래그 시작 위치보다 앞이면 탭 **왼쪽 2px** 선, 뒤이면 **오른쪽 2px** 선 | [pane.rs:2987-3026](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2987-L3026) · [pane.rs:3895-3987](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3895-L3987) |
| I-716 | 탭 바 끝 드롭 영역 | 최소 폭 1.5rem = **24px** (rem 16 가정), 높이는 탭 높이 32px. 놓으면 맨 끝에 넣는다 (01-LAYOUT L-70과 같은 24px) | [pane.rs:3731-3769](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L3731-L3769) |
| I-729 | 활성 Pane 표시 | 기본 테두리 0, 비활성 투명도 1.0이라 **강조 없음** (L-137). 사용자가 켜면 활성 Pane에 `border_selected` 테두리, 비활성 Pane에 `editor_background` 기반 투명 덮개 | [default.json:87-94](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L87-L94) · [pane_group.rs:1477-1525](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane_group.rs#L1477-L1525) |

### 4.6 상태 표시줄

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-28 | 높이 | 명시값 없음. padding Base04(4) × 2 + Default 버튼 22 = **30**, 아래 1px 테두리 보정. 관찰 약 31 | [status_bar.rs:150-182](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L150-L182) |
| L-29 | 간격 | 좌·우 그룹 사이 Base08 = 8, 그룹 안 gap 4 | [status_bar.rs:152](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L152) · [status_bar.rs:196](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L196) · [status_bar.rs:215](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L215) |
| L-91 | 구성 방식 | 왼쪽 목록·오른쪽 목록 두 개. 가로 `justify_between`, 배경 `status_bar_background` | [status_bar.rs:101-154](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/status_bar.rs#L101-L154) |
| L-92 | Dock 패널 버튼 | 왼쪽 Dock 버튼 묶음이 왼쪽 첫 아이템. 각 패널 버튼은 아이콘 버튼 Small(14), 열려 있으면 선택 표시 | [workspace.rs:1972-1984](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/workspace.rs#L1972-L1984) · [dock.rs:1542-1545](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/dock.rs#L1542-L1545) |
| L-93 | 프로젝트 패널 버튼 | 아이콘 `FileTree`, tooltip "Project Panel". 설정 `project_panel.button` 기본 true | [project_panel.rs:8278-8286](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L8278-L8286) |
| L-97 | 커서 위치 | 형식 "**{줄}:{열}**" (기준 화면 "9:11"). 줄·열 모두 1부터. 선택이 있으면 뒤에 설명 추가: long(기본) "2 selections, 15 lines, 32 characters", short "2 s, 15 l, 32 c". 버튼 글자 Small(12). 누르면 Go to Line/Column | [cursor_position.rs:31-60](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L31-L60) · [cursor_position.rs:181-225](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L181-L225) · [default.json:2940-2945](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2940-L2945) |
| L-98 | 우리 앱 구성 (판단) | 왼쪽: [사이드바 토글] [폴더 추가] [프로젝트 패널 토글] / 오른쪽: [커서 위치]. 기록·검색·진단·편집 예측·터미널·디버그·개요·협업·Git·AI는 넣지 않는다 | 기준 화면 규칙 |

화면 위치 정리 (기준 화면 관찰과 L-95·L-96·L-98을 합친 판단): 맨 아래 줄은 사이드바 폭 안쪽(푸터)과 Workspace(상태 표시줄)로 나뉜다. 사이드바 푸터 왼쪽 끝에 [사이드바 토글], 오른쪽 끝에 [폴더 추가]를 둔다. 상태 표시줄 왼쪽 끝에 [프로젝트 패널 토글], 오른쪽 끝에 [커서 위치]를 둔다.

### 4.7 툴팁

사용자 추가 요건: 아이콘 hover 툴팁도 Zed와 같게 한다. 값은 Zed 툴팁 조사(`zed-tooltips.md`)의 ID를 그대로 쓴다. 요소별 문구·단축키·상태별 문구는 [02-INTERACTION.md](02-INTERACTION.md) 8절에 있다. 구성 요소와 단축키 표시 형식도 그 절에 있다.

**표시·사라짐·위치**

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-206 | 표시 지연 | 기본 **500ms**. 요소별로 바꿀 수 있다 (예: 프로젝트 패널 항목 1500ms, L-247) | [div.rs:53](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L53) · [div.rs:758-761](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L758-L761) |
| L-207 | 사라지는 조건 | (1) 마우스가 요소 영역을 벗어난다 (2) **마우스 버튼을 누른다** (3) **스크롤한다**. 우리 앱의 툴팁은 모두 일반 툴팁이다. 툴팁 위로 마우스를 옮겨 유지하는 hoverable 툴팁은 쓰지 않는다 (참고: Zed의 hoverable 툴팁은 벗어난 뒤 500ms 뒤 사라진다) | [div.rs:54](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L54) · [div.rs:3744-3860](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L3744-L3860) · [div.rs:3875-3935](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L3875-L3935) |
| L-208 | 위치 기준 | **마우스 기준** (요소 기준이 아니다). 툴팁을 띄우는 순간의 마우스 위치를 기록하고 그 점에서 (+1, +1)px에 둔다. 띄운 뒤에는 마우스를 따라가지 않는다 (추정) | [div.rs:3987-3989](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs#L3987-L3989) · [window.rs:3884-3887](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/window.rs#L3884-L3887) |
| L-209 | 커서와의 간격 | 바깥 상자에 왼쪽 8px, 위 10px의 **투명 여백**이 있다. 그래서 보이는 상자는 커서에서 약 **(9, 11)px** 떨어진다 (1px + 여백) | [tooltip.rs:223-224](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L223-L224) |
| L-210 | 창 가장자리 처리 | 오른쪽이 창을 넘으면 커서 **왼쪽**(x − 폭 − 1)으로 뒤집고, 그래도 0보다 작으면 0에 붙인다. 아래쪽이 넘으면 커서 **위쪽**(y − 높이 − 1)으로 뒤집는다 | [window.rs:3893-3913](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/window.rs#L3893-L3913) |

**모양**

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-211 | 바탕 | `elevation_2`: 배경 `elevated_surface_background`, 모서리 `rounded_lg` (0.5rem = 8px), 테두리 1px `border_variant`, 그림자 | [styled_ext.rs:6-11](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/traits/styled_ext.rs#L6-L11) · [styled_ext.rs:62-64](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/traits/styled_ext.rs#L62-L64) |
| L-212 | 그림자 | 두 겹: ① y +2px, blur 3px, 검정 12% ② y +1px, blur 0, 검정 **라이트 3% / 다크 6%** | [elevation.rs:49-56](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/elevation.rs#L49-L56) |
| L-213 | 안쪽 여백 | 위아래 **4px**, 좌우 **8px** | [tooltip.rs:230-231](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L230-L231) |
| L-214 | 글꼴·크기·색 | UI 글꼴, **14px** (`TextSize::Default`), 색 `text` | [tooltip.rs:221-229](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L221-L229) · [typography.rs:139](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/typography.rs#L139) |
| L-215 | 첫 줄 배치 | 가로: [제목 (최대 폭 **288px** = 18rem, 추정)] … [단축키]. 단축키가 있으면 양끝 정렬, 둘 사이 gap **16px** | [tooltip.rs:196-203](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L196-L203) |
| L-216 | 메타 줄 | 제목 줄 아래 별도 줄. 글자 Small(12px), 색 Muted, 최대 폭 288px | [tooltip.rs:205-211](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs#L205-L211) |
| L-222 | 단축키 글자 모양 | 크기 14px, 줄 높이 1.0, 높이 = 글자 크기, 좌우 2px, 위아래 2px, 색 **Muted** (비활성이면 Disabled). 연속 키(chord)는 덩어리 사이 gap 4px (L-221) | [keybinding.rs:535-560](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L535-L560) · [keybinding.rs:259-287](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L259-L287) |

보이는 상자의 구조 (L-208~L-216을 합친 그림). 숫자는 px다.

```
마우스 (x, y)
  └ +1, +1 ─┐
            │ 투명 여백: 왼 8 / 위 10
            └─ ┌───────────────────────────────┐  테두리 1 (border_variant), 모서리 8, 그림자
               │ 제목 (최대 288) ─ 16 ─ 단축키   │  좌우 8 / 위아래 4 · 글자 14
               │ 메타 줄 (12, Muted, 선택)       │
               └───────────────────────────────┘
```

### 4.8 앱 메뉴 모양

☰ 앱 메뉴의 크기·여백·색이다. 값은 앱 메뉴 조사(`zed-app-menu.md`)의 ID(I-312~I-321)를 그대로 쓴다. 메뉴 구조와 동작은 [02-INTERACTION.md](02-INTERACTION.md) 9절에 있다. 계산 기준은 UI 글꼴 16px, 밀도 Default, 배율 100%다. **고정 높이·폭을 추정해서 적지 않았다.** 글꼴 래스터 반올림과 실제 메뉴 폭 실측은 미확인이다 (9절).

| ID | 항목 | 값·계산식 | 출처 |
| --- | --- | --- | --- |
| I-312 | 단위 | 1rem = 16px (`ui_font_size`). `rems_from_px(n)` = n ÷ 16 rem. 메뉴는 `ui_font_size`를 rem으로 쓰므로 UI 글꼴 크기를 바꾸면 **rem 값만 비례**하고 px 상수(최소 폭 200, 테두리 1)는 그대로다 | [units.rs:3-15](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/units.rs#L3-L15) · [context_menu.rs:2218-2228](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2218-L2228) · [context_menu.rs:2272-2276](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2272-L2276) |
| I-313 | 폭·최대 높이 | 최소 폭 **200px** (`CONTEXT_MENU_MIN_WIDTH`). 앱 메뉴는 폭을 고정하지 않아 내용에 따라 더 넓어진다. 테두리 좌우 1px씩을 더하면 최소 바깥 폭은 202px. 최대 높이 = 창 높이 × 0.75, 넘치면 세로 스크롤. **모든 메뉴가 200px 고정이 아니다** | [context_menu.rs:22](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L22) · [context_menu.rs:2281-2293](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2281-L2293) · [application_menu.rs:126-174](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L126-L174) |
| I-314 | 팝업 위치 | 기준점은 메뉴 이름(또는 ☰) 버튼의 **왼쪽 아래**, 팝업은 왼쪽 위 모서리를 거기에 맞춘다. x 오프셋 = −(5/16 × rem) = 기준 rem 16에서 **−5px**, y 오프셋 0. 창 가장자리 여유 8px. 타이틀 바 rem과 메뉴 내부 rem이 다를 수 있어 기준 화면 좌표 실측은 미확인 | [popover_menu.rs:150-158](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L150-L158) · [popover_menu.rs:245-269](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L245-L269) · [popover_menu.rs:371-384](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/popover_menu.rs#L371-L384) |
| I-315 | 바깥 스타일 | `elevation_2`: 배경 `elevated_surface_background`, 테두리 1px `border_variant`, 모서리 `rounded_lg` = 0.5rem = **8px**, 그림자 (I-320). 목록 위아래 padding Base04 = **4px**씩. 바깥 상자 자체에는 안쪽 padding이 없다 | [styled_ext.rs:6-11](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/traits/styled_ext.rs#L6-L11) · [styled_ext.rs:57-64](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/traits/styled_ext.rs#L57-L64) · [styles.rs:1251-1253](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui_macros/src/styles.rs#L1251-L1253) · [list.rs:79-83](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list.rs#L79-L83) |
| I-316 | 행 좌우 여백 | 행 바깥 좌우 Base04 = **4px**씩, 안쪽 좌우 Base06 = **6px**씩. 글자 시작은 테두리 안쪽에서 **10px**. 라벨과 단축키 사이 최소 1rem = **16px**. 줄바꿈 없이 라벨을 말줄임하고 폭은 내용에 따른다 | [context_menu.rs:1923-1927](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1923-L1927) · [list_item.rs:298-310](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L298-L310) · [list_item.rs:360-368](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L360-L368) · [context_menu.rs:2026-2041](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2026-L2041) |
| I-317 | 행 글꼴·높이 | 라벨 기본 **14px** (14/16 rem). 메뉴 줄 높이 1.618 → 한 줄 높이 14 × 1.618 = **22.652px**. 행에 위아래 추가 padding 없음 (Dense). 22.652px은 라벨 계산값이고 **모든 행의 고정 실측 높이가 아니다**. `buffer_font_size`(15)는 쓰지 않는다 | [typography.rs:133-142](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/typography.rs#L133-L142) · [buffer_line_height.rs:14-21](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/buffer_line_height.rs#L14-L21) · [context_menu.rs:2218-2225](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2218-L2225) · [list_item.rs:360-368](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L360-L368) |
| I-318 | 구분선 | 선 높이 **1px**, 폭 전체, 위아래 margin Base06 = **6px**씩 → 점유 높이 1 + 6 + 6 = **13px**. 색 `border_variant`. 밀도 Compact·Comfortable에서는 margin이 3·8px로 바뀐다 | [list_separator.rs:6-13](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_separator.rs#L6-L13) · [spacing.rs:28-39](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/spacing.rs#L28-L39) |
| I-319 | 색 토큰 | 라벨 `text`, 비활성 `text_disabled`. 단축키 `text_muted`, 비활성 `text_disabled`. 행 hover `ghost_element_hover`, 누름 `ghost_element_active`, 키보드 선택 `ghost_element_selected`. 값(RGBA)은 선택한 테마에서 가져온다 (Catppuccin hex를 여기서 고정하지 않는다) | [color.rs:90-105](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/color.rs#L90-L105) · [list_item.rs:379-384](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L379-L384) · [keybinding.rs:259-287](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/keybinding.rs#L259-L287) |
| I-320 | 그림자 | 두 겹: offset (0, 2px)·검정 12%·blur 3px, offset (0, 1px)·검정 (라이트 3% / 다크 6%). 다른 modal의 4겹 그림자와 다르다 | [elevation.rs:48-63](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/elevation.rs#L48-L63) |
| I-321 | ☰ 버튼·가로 메뉴 이름 | ☰는 아이콘 `Menu`, Small = **14px**, Subtle 버튼. 펼친 가로 메뉴의 이름은 글자 Small = **12px**, 버튼 사이 gap 0.25rem = **4px**. 버튼 hitbox는 아이콘 크기와 같지 않다 | [application_menu.rs:191-203](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L191-L203) · [application_menu.rs:220-227](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L220-L227) · [application_menu.rs:358-367](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs#L358-L367) · [icon.rs:72-80](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/icon.rs#L72-L80) · [typography.rs:138-141](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/typography.rs#L138-L141) |

행 구조 (I-315~I-318을 합친 그림). 숫자는 px다.

```
┌─ 테두리 1 (border_variant), 모서리 8, 그림자 ─────────────────┐
│ 위 4                                                             │
│  4 ┌ 일반 행 ───────────────────────────────────────────┐ 4     │
│    │ 6 라벨 (14px, 줄 높이 22.652) ··· 16 ··· 단축키 6 │       │
│    └────────────────────────────────────────────────────┘       │
│  ─ 구분선: 위 6 / 선 1 / 아래 6 (= 13) ─                        │
│ 아래 4                                                           │
└──────────────────────────────────────────────────────────────┘
최소 폭 200 (테두리 안쪽) · 최대 높이 = 창 높이 × 0.75, 넘치면 스크롤
```

### 4.9 컨텍스트 메뉴 모양

아이콘과 각 영역의 우클릭 메뉴의 공통 명세다. 값은 우클릭 메뉴 조사(`zed-context-menus.md`)의 ID(I-500~I-515)를 그대로 쓴다. 영역별 항목·최종 구조·키보드 조작은 [02-INTERACTION.md](02-INTERACTION.md) 11절에 있다. 바탕·여백·색은 ☰ 앱 메뉴(4.8절)·∨ 메뉴(4.1절)와 **같은 메뉴 위젯**이다 (I-504 ↔ I-315). 계산 기준은 UI 글꼴 16px, 밀도 Default이다.

**열리는 위치**

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| I-501 | 우클릭 기본 위치 | 요소를 우클릭하면 **마우스 커서 위치에 메뉴의 왼쪽 위 모서리**를 둔다. 필요하면 메뉴의 맞출 모서리와 요소의 붙일 모서리를 바꿀 수 있다. 창 밖으로 나가지 않게 **창 가장자리에서 8px 여백**을 두고 안쪽으로 민다 | [right_click_menu.rs:43-52](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/right_click_menu.rs#L43-L52) · [right_click_menu.rs:146-152](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/right_click_menu.rs#L146-L152) |
| I-502 | 프로젝트 패널 위치 | 마우스로 열면 마우스 위치. 키보드(Menu 키, Shift+F10)로 열면 **선택 항목 옆**에 (2, 2)px 띄워서 연다 (아래쪽에 붙을 때는 −2) | [project_panel.rs:1317-1336](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L1317-L1336) · [project_panel.rs:203](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L203) · [default-windows.json:1043-1044](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L1043-L1044) |
| I-503 | 편집기 위치 | 클릭한 위치에 고정한다. 클릭 지점이 기존 선택 영역 밖이면 **먼저 커서를 그 위치로 옮긴 뒤** 메뉴를 연다 | [mouse_context_menu.rs:160-200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/mouse_context_menu.rs#L160-L200) · [mouse_context_menu.rs:338-346](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/mouse_context_menu.rs#L338-L346) |

**모양**

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| I-504 | 바탕 | `elevation_2`: 배경 `elevated_surface_background`, 모서리 `rounded_lg` (8px), 테두리 1px `border_variant`, 그림자 2겹 (툴팁과 같음, 4.7절 L-211·L-212) | [context_menu.rs:2270-2276](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2270-L2276) |
| I-505 | 글꼴 | UI 글꼴. 메뉴 안 rem = `ui_font_size`. 줄 높이 **1.618** 고정 (편집기 줄 높이 설정과 무관) | [context_menu.rs:2219-2226](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2219-L2226) |
| I-506 | 크기 | **최소 폭 200px** (폭을 고정하지 않을 때), **최대 높이 = 창 높이의 75%**, 넘치면 세로 스크롤 | [context_menu.rs:22](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L22) · [context_menu.rs:2284-2293](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2284-L2293) |
| I-507 | 안쪽 여백 | 목록 위아래 Base04 = **4px**. 항목 바깥 좌우 Base04 = 4px, 항목 안 좌우 Base06 = 6px | [list.rs:80-82](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list.rs#L80-L82) · [list_item.rs:306-310](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L306-L310) |
| I-508 | 항목 높이 | 명시값 없음. 추가 padding 0 + 라벨 14 × 1.618 ≈ 22.6 + 항목 테두리 1px × 2 ≈ **약 25px** (추정) | [context_menu.rs:1923-1930](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1923-L1930) · [list_item.rs:365-368](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L365-L368) |
| I-509 | 항목 배치 | [토글 체크 아이콘 (시작 위치, 꺼져 있으면 자리만 차지)] [라벨] … [단축키]. 라벨과 단축키는 양끝 정렬, 단축키 앞 여백 16px. 하위 메뉴 항목은 오른쪽 끝에 `ChevronRight`(Muted) | [context_menu.rs:2008-2050](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2008-L2050) · [context_menu.rs:1682-1684](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1682-L1684) |
| I-510 | 단축키 표시 | 메뉴에 포커스 컨텍스트가 지정돼 있으면 그 컨텍스트에서, 아니면 전역에서 키를 찾는다. 비활성 항목은 단축키도 Disabled 색. 형식은 툴팁과 같다 (Muted, `Ctrl-X`, 4.7절·02 8.4절) | [context_menu.rs:2034-2040](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L2034-L2040) |
| I-511 | 아이콘 크기 | 토글 체크 같은 항목 아이콘 기본 Small = **14px** | [context_menu.rs:114](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L114) |
| I-512 | 구분선 | 높이 1px, 위아래 margin Base06 = **6px**, 색 `border_variant` (점유 13px, I-318) | [list_separator.rs:9-12](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_separator.rs#L9-L12) |
| I-513 | 헤더 | 글자 Small(12px), 색 Muted | [context_menu.rs:1418-1425](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1418-L1425) |
| I-514 | 색 | 라벨: 일반 `Default`, 비활성 `Disabled`. 아이콘: 비활성 Muted, 토글 Accent. hover 배경 `ghost_element_hover`, 누름 `ghost_element_active`, 키보드 선택 `ghost_element_selected`, 키보드 포커스 테두리 `border_focused` | [context_menu.rs:1812-1824](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1812-L1824) · [list_item.rs:370-385](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L370-L385) |
| I-515 | 비활성 항목 | 회색(Disabled) 라벨, hover·클릭 반응 없음, 키보드 선택에서도 건너뛴다 | [list_item.rs:379](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L379) · [context_menu.rs:1098-1135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/context_menu.rs#L1098-L1135) |

### 4.10 스크롤바

프로젝트 패널·편집기·미리보기가 같은 스크롤바 구성요소를 쓴다. 기준 화면의 프로젝트 패널 아래 가로 막대가 이 값과 맞는다 (두께 6, 아래 간격 4).

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-308 | 두께 | 일반 **6px**, 편집기 **15px** | [scrollbar.rs:29](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/scrollbar.rs#L29) · [scrollbar.rs:421-426](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/scrollbar.rs#L421-L426) |
| — | (참조 L-308) 안쪽 여백·테두리 | 막대 바깥 여백 `SCROLLBAR_PADDING` = **4px** (양쪽), 테두리 폭 1px. 전체 점유 폭 = 두께 + 4 × 2 | [scrollbar.rs:30-31](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/scrollbar.rs#L30-L31) · [scrollbar.rs:831](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/scrollbar.rs#L831) |
| — | (참조 L-308) 손잡이 최소 길이 | **25px** | [scrollbar.rs:934](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/scrollbar.rs#L934) |
| — | (참조 L-308) 색 | 토큰 `scrollbar.thumb.*`, `scrollbar.track.*` (값은 03). 표시·숨김 시점, 트랙 클릭, 바퀴 입력 규칙은 조사하지 않았다 (확신 낮은 부분) | — |

### 4.11 글자색 상태 규칙

기준 화면에서 같은 `text` 토큰이 아니라 **상태에 따라 `text` 또는 `text_muted`**가 쓰인다.

| ID | 대상 | 규칙 | 출처 |
| --- | --- | --- | --- |
| L-305 | 프로젝트 패널 항목 이름 | **표시(marked)되지 않은 항목은 `text_muted`**, 표시된(선택·다중 선택) 항목은 `text`. Git 상태 색은 쓰지 않으므로 무시한다. 기준 화면의 비선택 이름이 `text`보다 옅은 이유다 (Mocha에서 `#bac2de`) | [project_panel.rs:7202-7203](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L7202-L7203) · [items.rs:2256-2262](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L2256-L2262) · [items.rs:2284-2298](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L2284-L2298) |
| L-306 | 탭 글자색 | **편집기 탭:** 활성 `text`, 비활성 `text_muted` (Pane 포커스와 무관). **마크다운 미리보기 탭**(탭 글자색을 `TabContentParams`로 받는 항목): Pane에 포커스가 있으면 활성 `text`, 비활성 `text_muted`. **Pane에 포커스가 없으면(deemphasized)** 활성 `text_muted`, 비활성 **`status.hidden`**. 그래서 포커스가 트리·사이드바에 있을 때 비활성 미리보기 탭이 편집기 탭보다 훨씬 옅다 (Latte 화면). 탭 컨테이너의 기본 글자색(`Tab`)은 활성 `text`, 비활성 `text_muted`다 | [item.rs:141-156](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/item.rs#L141-L156) · [pane.rs:2872-2895](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/pane.rs#L2872-L2895) · [tab.rs:107-121](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L107-L121) · [color.rs:90-105](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/color.rs#L90-L105) |
| L-307 | 상태 표시줄 커서 위치 | 상태 표시줄 아이템은 `Subtle` 스타일 일반 버튼이고 라벨 색 지정이 없어 기본값인 **`text`**(`Color::Default`)다. 기본·hover·누름·포커스 모두 같고, 비활성이면 `text_disabled`다. **Latte 화면의 "1:1" 측정(`#4c4f69`)이 Latte `text`와 일치한다** (P-027). 크기 Small(12) (L-97) | [button.rs:468-474](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button.rs#L468-L474) · [cursor_position.rs:226-227](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/cursor_position.rs#L226-L227) · [button_like.rs:242-247](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L242-L247) · [button_like.rs:298-329](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L298-L329) · [button_like.rs:417-422](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L417-L422) |
| P-024 | 프로젝트 패널 (보강) | 폴더와 파일 모두 같은 규칙이다 (L-305). 마우스를 올리면 **행 배경만** `element_hover`로 바뀌고 글자·아이콘 색은 그대로다. 폴더 이름 굵게(`bold_folder_labels`)는 기본 꺼짐. 아이콘은 `icon_muted`. 무시됨(`text_ignored`)·Git 상태 색은 Git 기능이라 쓰지 않는다 | [project_panel.rs:6160-6163](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6160-L6163) · [project_panel.rs:6278](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6278) · [project_panel.rs:6808-6817](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6808-L6817) |
| P-025 | 사이드바 폴더 항목 | 활성 폴더의 이름 `text`, 비활성 폴더의 이름 `text_muted`. 선택 배경 `ghost_element_selected`, hover 배경 `ghost_element_hover`, 포커스 테두리 `panel_focused_border`. 아이콘 `icon_muted`. 항목 제목 글자는 선택 여부와 무관하게 `text`인 스레드 항목 규칙(Zed)은 폴더 목록에 쓰지 않는다 | [sidebar.rs:2377-2387](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2377-L2387) · [sidebar.rs:2550-2557](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/sidebar/src/sidebar.rs#L2550-L2557) |
| P-026 | 탭 (보강) | L-306 규칙과 같다. 추가: 탭에 마우스를 올려도 **제목 글자색은 바뀌지 않고** 닫기 버튼만 나타난다 (`_tab_hover_bg`는 선언만 되고 쓰이지 않는다). 미리보기 탭은 **기울임**, 디스크에서 지워진 파일은 **취소선**. 같은 이름이 겹칠 때의 보조 경로는 XSmall `text_muted`. 편집기 탭은 Pane 포커스와 무관하게 활성 `text`, 비활성 `text_muted`이고 deemphasized 규칙은 편집기 아닌 항목(마크다운 미리보기)에만 적용된다 | [tab.rs:107-125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tab.rs#L107-L125) · [item.rs:141-156](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/item.rs#L141-L156) · [items.rs:771-849](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/items.rs#L771-L849) |
| P-027 | 상태 표시줄 버튼 (보강) | 모든 상태 표시줄 버튼: 배경 기본 `ghost_element_background`(투명), hover `ghost_element_hover`, 누름 `ghost_element_active`, 포커스 시 테두리 `border_focused`. 글자·아이콘 `text`·`icon` (일부 아이콘 버튼은 Muted). 비활성은 배경 `ghost_element_disabled`, 글자 `text_disabled`, 아이콘 `icon_disabled`. 켜진(선택) 버튼(예: 프로젝트 패널 토글)은 Accent 틴트 | [button_like.rs:242-247](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L242-L247) · [button_like.rs:298-329](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L298-L329) · [button_like.rs:417-422](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L417-L422) |
| P-023 | 영역별 요약 | 프로젝트 패널 비선택 `text_muted` / 선택 `text`. 사이드바 활성 `text` / 비활성 `text_muted`. 탭 활성 `text` / 비활성 `text_muted` (미리보기 탭은 포커스 없음 규칙 L-306). 상태 표시줄 `text` / 비활성 `text_disabled`. 조사 문서가 가정한 "Ayu Dark" 색 값은 기준 화면(Catppuccin)과 맞지 않아 쓰지 않고 토큰 이름만 쓴다 | 위 L-305~L-307, P-024~P-027 |

상태 표시줄 커서 위치(L-307)의 글자색은 소스(`text`)와 Latte 화면 측정(`#4c4f69`)이 일치한다. 앞 조사의 Mocha 쪽 측정(`text_muted`와 비슷한 옅은 색)은 맞지 않았다.

### 4.12 편집기 화면

원문 편집기(요건 4)의 화면 값이다. 편집기 조사(`zed-editor.md`)의 ID(E-001~E-122)를 그대로 쓴다. 동작·키·확인창은 [02-INTERACTION.md](02-INTERACTION.md) 13절에 있다. 치수는 논리 px, 기준은 `buffer_font_size` 15, `ui_font_size` 16이다. **두 기준 화면은 모두 마크다운 미리보기 탭이 활성이라 편집기 본문이 보이지 않는다.** 아래 값은 모두 소스 값이고 화면 대조는 못 했다 (4.12.7).

우리 앱에서 뺀 것: Git 거터·변경 표시, 중단점·실행·북마크 아이콘, 코드 접기 동작, 진단·LSP 표시, 미니맵, sticky scroll, 줄 길이 안내선, 커서 이동 애니메이션, 편집기 컨트롤(Editor Controls) 메뉴. 포함 목록에 없거나 제외 목록에 걸린다.

#### 4.12.1 거터와 가로 구성

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| E-001 | 가로 순서 | [거터: 왼쪽 여백 · 줄 번호 · 오른쪽 여백] [margin] [본문 텍스트] … [세로 스크롤바 15]. 거터 전체 폭 = `margin + width`, `width` = 줄 번호 폭 + 왼쪽 여백 + 오른쪽 여백 | [editor.rs:1305-1329](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/editor.rs#L1305-L1329) |
| E-002 | 줄 번호 폭 | max(가장 큰 줄 번호의 자릿수 × 숫자 폭, `min_line_number_digits`(기본 **4**) × `ch_advance`). 1~9999줄은 4자리 폭으로 고정되어 자릿수가 바뀔 때 거터가 흔들리지 않는다. 15px 글꼴에서 4자리 ≈ 36px (글꼴 advance에 따라 다름, 추정) | [editor.rs:12502-12541](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/editor.rs#L12502-L12541) · [default.json:753-768](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L753-L768) |
| E-003 | 거터 왼쪽 여백 | 단일 파일 편집기에서 실행·중단점·북마크 표시 중 하나라도 켜져 있으면 **3 ch**, Git 거터 + 줄 번호면 2 ch, 하나만이면 1 ch, 없으면 0. `ch` = 글꼴 "0"의 폭. Zed 기본값은 셋 다 true라 **3 ch**(15px에서 약 27px). **우리 앱은 Zed와 같은 외형을 우선해 3 ch로 정한다** (판단: 해당 아이콘은 없어도 여백은 남긴다) | [editor.rs:12562-12576](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/editor.rs#L12562-L12576) · [default.json:755-763](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L755-L763) |
| E-004 | 거터 오른쪽 여백 | 접기 표시(`gutter.folds` 기본 true) + 줄 번호 → **4 ch**. 접기만 3 ch, 줄 번호만 1 ch. 접기 화살표가 이 4 ch 안에 그려진다. 우리 앱은 접기 동작을 만들지 않지만 **4 ch 여백은 그대로 둔다** (거터 폭을 Zed와 같게) | [editor.rs:12578-12587](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/editor.rs#L12578-L12587) |
| E-005 | 거터 margin | 버퍼 글꼴 **descent의 절댓값**. 거터와 본문 텍스트 사이 간격. 글꼴마다 다르다 (15px에서 약 3~4px, 추정) | [editor.rs:1322-1324](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/editor.rs#L1322-L1324) |
| E-006 | 줄 번호 정렬·색 | **오른쪽 정렬**: x = 거터 폭 − 숫자 폭 − 오른쪽 여백. 커서(선택 머리)가 있는 줄은 `editor_active_line_number`, 나머지는 `editor_line_number`. `relative_line_numbers` 기본 `"disabled"`. Git 변경·중단점 색은 쓰지 않는다 | [element.rs:106-140](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L106-L140) · [element.rs:2857-2950](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L2857-L2950) · [default.json:813](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L813) |
| E-007 | 거터 배경 | `editor_gutter_background` | [colors.rs:212](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L212) |
| E-008 | 접기 표시 | `gutter.folds` true: 접을 수 있는 줄에 화살표(접힘 = 오른쪽, 펼침 = 아래). **우리 앱은 만들지 않는다** (포함 목록에 없음). 표시 시점(hover 때만)은 추정이었다 | [default.json:763](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L763) |
| E-009 | 거터 클릭 | 거터를 누르면 **세 번 클릭과 같게 줄 전체 선택**. 끌면 여러 줄 선택 | [mouse.rs:595-617](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element/mouse.rs#L595-L617) |

#### 4.12.2 줄·현재 줄·커서·선택

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| E-010 | 줄 높이 | `buffer_line_height` 기본 `"comfortable"` = 글꼴 크기 × **1.618** → 15px에서 **24.27px**. `"standard"` = 1.3(19.5px), 사용자 값은 1.0 이상 | [buffer_line_height.rs:6-19](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/buffer_line_height.rs#L6-L19) · [default.json:41-54](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L41-L54) |
| E-011 | 현재 줄 강조 | `current_line_highlight` 기본 **`"all"`**: 편집기 왼쪽 끝(거터 포함)부터 오른쪽 끝까지, 높이 1줄, 색 `editor_active_line_background`. `"gutter"` = 거터만, `"line"` = 본문만, `"none"`. **커서 줄에 비어 있지 않은 선택이 있으면 그 줄은 강조하지 않는다.** 연속 줄은 한 사각형으로 합친다 | [element.rs:5099-5150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L5099-L5150) · [default.json:359](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L359) |
| E-012 | 커서 모양 | `cursor_shape` 기본 **`"bar"`**: 폭 **2px** × 줄 높이. `block`·`hollow` = 글자 폭 × 줄 높이, `underline` = 글자 폭 × 2px (줄 아래 끝). 색 `players[0].cursor` | [element.rs:11081-11094](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L11081-L11094) · [default.json:307](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L307) · [players.rs:10-12](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/players.rs#L10-L12) |
| E-013 | 커서 깜빡임 | `cursor_blink` 기본 true. 보임/숨김을 **500ms**마다 바꾼다. 입력·이동 같은 조작이 있으면 깜빡임을 멈추고 커서를 보이게 한 뒤 500ms가 지나서 다시 깜빡인다. 편집기에 포커스가 없으면 깜빡이지 않는다 (추정) | [editor.rs:305](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/editor.rs#L305) · [blink_manager.rs:46-75](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/blink_manager.rs#L46-L75) · [default.json:295](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L295) |
| E-014 | 커서 이동 애니메이션 | `cursor_animation.enabled` 기본 false. **만들지 않는다** | [default.json:308-312](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L308-L312) |
| E-015 | 선택 영역 모양 | 색 `players[0].selection`. `rounded_selection` 기본 true → 모서리 반지름 = **0.15 × 줄 높이**(24.27px에서 **3.64px**). 여러 줄 선택의 안쪽 꺾인 모서리는 그 두 배 반지름으로 계산한다 | [element.rs:5993-6008](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L5993-L6008) · [default.json:363](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L363) |
| E-016 | 같은 텍스트 강조 | `selection_highlight` 기본 true: 선택한 텍스트와 같은 다른 위치를 강조한다. 갱신 지연 **100ms**. 색 토큰은 찾지 못했다 (4.12.7) | [editor.rs:314](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/editor.rs#L314) · [default.json:361](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L361) |
| E-017 | 강조 줄 | Go to Line 미리보기 줄은 `editor_highlighted_line_background` | [go_to_line.rs:182-205](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L182-L205) |

#### 4.12.3 들여쓰기 안내선·공백·줄 바꿈

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| E-020 | 들여쓰기 안내선 | `indent_guides.enabled` true, 선 폭 **1px**, 활성 안내선 폭 **1px**, `coloring: "fixed"` → 색 `editor_indent_guide`, 커서가 속한 블록은 `editor_indent_guide_active`. `background_coloring: "disabled"` | [default.json:770-790](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L770-L790) · [element.rs:5287-5290](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L5287-L5290) |
| E-021 | 공백 표시 | `show_whitespaces` 기본 **`"selection"`**: 선택 영역 안의 공백만 보인다. 공백 `•`, 탭 `→`. 색 `editor_invisible` | [default.json:583-588](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L583-L588) · [element.rs:9924-9938](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L9924-L9938) |
| E-022 | 줄 바꿈 (soft wrap) | `soft_wrap` 기본 **`"none"`**: 줄을 접지 않고 가로 스크롤. `preferred_line_length` 80. 토글 키: VSCode 덧씌움 **Alt-Z**, 기본 키맵 Ctrl-K Z / Ctrl-K Ctrl-Z (VSCode에서는 Ctrl-K Z가 다른 동작으로 바뀐다, 02 I-820) | [default.json:1687-1696](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1687-L1696) · [default-windows.json:135-136](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L135-L136) |
| E-023 | 줄 길이 안내선 | `show_wrap_guides` true, `wrap_guides` `[]`: soft wrap가 `preferred_line_length`일 때만 세로선이 생긴다. 기본에서는 **보이지 않으므로 만들지 않는다**. 색 `editor_wrap_guide`, `editor_active_wrap_guide` | [default.json:495-500](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L495-L500) · [element.rs:2476-2500](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L2476-L2500) |
| E-024 | 미니맵·sticky scroll | 미니맵 기본 꺼짐, 편집기 `sticky_scroll.enabled` false. **만들지 않는다** | [default.json:695](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L695) · [default.json:809-812](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L809-L812) |

#### 4.12.4 스크롤바

일반 스크롤바(트리·미리보기)는 4.10절이다. 편집기 것은 아래 값이다.

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| E-030 | 두께 | 편집기 스크롤바 **15px** (`ScrollbarStyle::Editor`). 일반 UI 스크롤바는 6px | [scrollbar.rs:29](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/scrollbar.rs#L29) · [scrollbar.rs:421-428](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/scrollbar.rs#L421-L428) |
| E-031 | 위치·트랙 | 세로는 오른쪽 끝 전체 높이, 가로는 아래쪽. 트랙 배경 `scrollbar_track_background`. **세로 트랙만 왼쪽에 1px 테두리**(`scrollbar_track_border`). 모서리는 각지게 | [element.rs:6152-6185](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L6152-L6185) · [scrollbar.rs:31](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/scrollbar.rs#L31) |
| E-032 | 손잡이 색 상태 | 기본 `scrollbar_thumb_background`, hover `scrollbar_thumb_hover_background`, 끄는 중 `scrollbar_thumb_active_background`. 테두리 `scrollbar_thumb_border`. 편집기 손잡이도 각진 모서리 | [element.rs:6200-6220](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L6200-L6220) |
| E-034 | 마커 | 세로 스크롤바 위 위치 표시: 커서(`scrollbar.cursors` true, 너무 많으면 생략), 찾기 결과(`search_results`), 선택 텍스트의 다른 위치(`selected_text`). Git 변경·진단·심볼 마커는 만들지 않는다 | [default.json:669-683](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L669-L683) · [element.rs:6375-6395](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/editor/src/element.rs#L6375-L6395) |

#### 4.12.5 툴바

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| E-038 | 편집기 툴바 | 편집기 탭에는 탭 바 아래 **툴바**가 있다. 왼쪽 **브레드크럼**(`toolbar.breadcrumbs` true: 파일 아이콘 + 경로, 버퍼 글꼴, 넘치면 가로 스크롤), 오른쪽 **빠른 동작 버튼**(`quick_actions` true, gap Base01 = 1px). 툴바 상자: 위아래 padding Base06(6px), 좌우 Base08(8px), 아래 1px `border_variant`, 배경 `toolbar_background`, 줄 최소 높이 `_8` = 32px. 찾기 창(E-060~)도 이 툴바 안에 열린다. 브레드크럼은 **파일 경로만** 보이고 구조(심볼) 표시는 만들지 않는다 (개요 기능 제외) | [default.json:597-608](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L597-L608) · [toolbar.rs:120-150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/toolbar.rs#L120-L150) · [breadcrumbs.rs:49-60](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/breadcrumbs/src/breadcrumbs.rs#L49-L60) |
| E-039 | 빠른 동작 버튼 (순서) | ① 마크다운 파일이면 **미리보기 버튼**(`Eye`, Small, 툴팁 "Preview Markdown" + 키, 메타 "Alt-Click to open in a split") ② **Buffer Search** 버튼(`buffer_search::Deploy`) ③ **Selection Controls**(`CursorIBeam`, 메뉴: Select All, Select Next Occurrence, Expand Selection, Shrink Selection, Add Cursor Above/Below, Go to Line/Column, Move Line Up/Down, Duplicate Selection). 코드 액션 버튼과 Editor Controls(`Filter`) 메뉴는 만들지 않는다 (LSP·제외 기능, Line Numbers·Minimap·Vim 토글) | [quick_action_bar.rs:140-160](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/quick_action_bar.rs#L140-L160) · [quick_action_bar.rs:255-320](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/quick_action_bar.rs#L255-L320) · [quick_action_bar.rs:535-700](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/quick_action_bar.rs#L535-L700) · [preview.rs:38-80](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed/quick_action_bar/preview.rs#L38-L80) |

#### 4.12.6 찾기·바꾸기 창 (Buffer Search Bar)

동작·키·툴팁은 02 13.3절이다. 색 토큰은 4.12.8의 E-072, E-110~E-121이다.

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| E-060 | 위치 | 활성 편집기 위의 **툴바(E-038) 안**에 연다. 툴바 padding 6/8, 배경 `toolbar_background`, 아래 1px `border_variant` | [toolbar.rs:120-150](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/toolbar.rs#L120-L150) |
| E-061 | 줄 구성 | 세로로 [찾기 줄] [바꾸기 줄(바꾸기 켜짐)] [오류 줄(정규식 오류 등)], 줄 사이 gap 8px. 찾기 줄 = [입력 상자] [모드 묶음], gap 8px | [buffer_search.rs:380-480](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L380-L480) |
| E-062 | 입력 상자 | 최소 높이 **32px**, 왼쪽 padding 8, 오른쪽 4, 테두리 1px `border`(정규식 오류면 Error 색), 모서리 6px(`rounded_md`). 폭은 창 폭이 1200 미만이면 창 폭, 이상이면 1200까지. 실제 폭은 옆 묶음에 밀려 줄어든다 (추정) | [search_bar.rs:73-83](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/search_bar.rs#L73-L83) · [search_input.rs:6-20](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/utils/search_input.rs#L6-L20) · [buffer_search.rs:214-236](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L214-L236) |
| E-063 | 입력 글자 | **버퍼 글꼴**, **14px**(0.875rem), 줄 높이 1.3, 색 `text`(구문 강조 사용). 일치 결과가 없으면 입력 글자를 **Error 색**으로 바꾼다 | [search_bar.rs:101-130](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/search_bar.rs#L101-L130) · [buffer_search.rs:196-208](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L196-L208) |
| E-064 | placeholder | 찾기 "**Search…**", 바꾸기 "**Replace with…**" (말줄임표 한 글자 `…`) | [buffer_search.rs:179-185](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L179-L185) |
| E-065 | 입력 안 옵션 토글 | 오른쪽, gap 4. 순서: **대소문자**(`CaseSensitive`) · **단어**(`WholeWord`) · **정규식**(`Regex`). 정사각 아이콘 버튼, 켜지면 선택 배경. 누르면 입력칸에 포커스를 둔다 | [buffer_search.rs:236-262](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L236-L262) · [search.rs:110-185](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/search.rs#L110-L185) |
| E-066 | 모드 묶음 | 찾기 줄 오른쪽, 최소 폭 256px, gap 4: ① **바꾸기 토글**(`Replace`) ② **선택 영역 안에서 찾기**(`Quote`) ③ **모든 일치 선택**(`SelectAll`) ④ 구분: 왼쪽 1px `border_variant` 선 + padding-left 8 + margin-left 8 ⑤ **이전**(`ChevronLeft`) ⑥ **다음**(`ChevronRight`) — 일치가 없으면 ⑤⑥ 비활성 ⑦ **결과 수** 라벨 | [buffer_search.rs:264-380](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L264-L380) |
| E-067 | 결과 수 | 형식 **"{현재}/{전체}"**(예 `3/12`). 활성 일치가 없으면 "0/0". 글자 Small(12px), 왼쪽 여백 8, 최소 폭 40px. 활성 일치가 있으면 `Default` 색, 없으면 `Disabled` 색 | [buffer_search.rs:196-210](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L196-L210) · [buffer_search.rs:338-350](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L338-L350) |
| E-068 | 닫기 버튼 | `Close`, 찾기 줄 **오른쪽 끝 절대 위치**. 툴팁 "Close Search Bar" | [buffer_search.rs:443-470](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L443-L470) |
| E-069 | 좁은 모드 | 툴바 폭이 340px(rem 16 기준) 미만이면 결과 수 라벨과 닫기 버튼을 숨긴다 | [buffer_search.rs:165-166](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L165-L166) |
| E-070 | 바꾸기 줄 | [바꾸기 입력 상자(같은 스타일, 테두리 `border`)] [묶음 최소 폭 256, gap 4: **Replace Next**(`ReplaceNext`), **Replace All**(`ReplaceAll`)] | [buffer_search.rs:388-425](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L388-L425) |
| E-071 | 오류 줄 | 정규식 오류 문구를 Small·Error 색으로, 위로 −4px, 왼쪽 8px | [buffer_search.rs:436-442](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/search/src/buffer_search.rs#L436-L442) |
| E-072 | 일치 강조 색 | 일반 일치 `search_match_background`, 활성 일치 `search_active_match_background` | [colors.rs:130-131](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L130-L131) |

#### 4.12.7 편집기 색 토큰

| ID | 토큰 | 쓰는 곳 | 출처 |
| --- | --- | --- | --- |
| E-110 | `editor_background` / `editor_foreground` | 본문 배경 / 기본 글자 | [colors.rs:206-211](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L206-L211) |
| E-111 | `editor_gutter_background` | 거터 | [colors.rs:212](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L212) |
| E-112 | `editor_active_line_background` | 현재 줄 (E-011) | [colors.rs:214](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L214) |
| E-113 | `editor_highlighted_line_background` | Go to Line 미리보기 줄 | [colors.rs:215](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L215) |
| E-114 | `editor_line_number` / `editor_active_line_number` / `editor_hover_line_number` | 줄 번호 / 커서 줄 번호 / hover 줄 번호(사용처 미확인) | [colors.rs:219-223](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L219-L223) |
| E-115 | `editor_invisible` | 공백 기호 | [colors.rs:227](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L227) |
| E-117 | `editor_indent_guide` / `editor_indent_guide_active` | 들여쓰기 안내선 | [colors.rs:230-231](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L230-L231) |
| E-118 | `players[0].cursor` / `players[0].selection` | 커서 / 선택 | [players.rs:10-12](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/players.rs#L10-L12) |
| E-119 | `search_match_background` / `search_active_match_background` | 찾기 일치 | [colors.rs:130-131](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L130-L131) |
| E-120 | `scrollbar_track_background` / `_border`, `scrollbar_thumb_background` / `_hover_background` / `_active_background` / `_border` | 스크롤바 | [colors.rs:146-156](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L146-L156) |
| E-121 | `toolbar_background` | 툴바·찾기 창 바탕 | [colors.rs:126](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs#L126) |
| E-122 | 라이트 테마 | 위 토큰 이름은 같고 값만 테마 파일에서 온다. Latte(라이트)와 Mocha(다크)의 값은 [03-THEME-ICONS.md](03-THEME-ICONS.md)의 테마 값 표를 쓴다 | 03 |

(줄 길이 안내선 색 E-116은 안내선을 만들지 않으므로 쓰지 않는다.)

#### 4.12.8 편집기 화면에서 확신 낮은 부분

- **화면 근거 없음:** 두 기준 화면 모두 미리보기 탭이 활성이라 거터·줄 번호·현재 줄·커서·스크롤바를 화면으로 대조할 수 없다. 편집기 탭이 활성인 화면 캡처를 받으면 E-002~E-005의 px를 확정할 수 있다.
- **px 환산 (E-002, E-003, E-005):** 9px advance, descent 3~4px는 15px `.ZedMono`에 대한 추정이다. 실제 값은 글꼴 메트릭(`ch_width`, `ch_advance`, `descent`)으로 계산해야 한다.
- **같은 텍스트 강조 색 (E-016):** 어떤 토큰을 쓰는지 찾지 못했다 (문서 하이라이트 계열 토큰으로 추정).
- **커서 깜빡임과 포커스 (E-013):** 포커스가 없을 때 멈추는 연결은 추정이다.
- **스크롤바 자동 숨김 (E-033):** Windows에서 어떤 OS 설정을 읽는지 확인하지 않았다 (02 13.7절).
- **찾기 입력 상자 폭 (E-062):** flex 안에서 줄어드는 실제 폭은 계산하지 않았다.
- **브레드크럼 (E-038):** 경로 구분 기호와 클릭 동작의 세부를 찾지 못했다. 렌더 함수 위치는 미확인이다.

### 4.13 Picker·대화상자·인라인 입력 모양

Picker·대화상자 조사(`zed-pickers-dialogs.md`)의 ID(P-001~P-027)를 그대로 쓴다. 동작·키·문구는 [02-INTERACTION.md](02-INTERACTION.md) 14절이다. 치수는 논리 px(rem 16)이다. 조사에 있는 픽셀 실측값은 보조로만 썼고 **소스 값이 있으면 소스를 따랐다.** 포함·제외는 사용자 결정(`_decisions.md`)에 따랐다: 파일 찾기, 명령 팔레트, Open Recent, 테마·아이콘 테마 선택기, Go to Line, About은 포함이고 Git 되돌리기 확인(P-017), 파일 찾기 미리보기·다중 선택은 뺀다.

#### 4.13.1 모달 레이어와 Picker 컨테이너

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| P-001 | 위치 | 창 전체를 덮는 오버레이 위에 그린다. 모달의 위 가장자리는 **창 위에서 80px**(`top_20` = 5rem), **가로 중앙**. 일반 Picker는 **창을 어둡게 하지 않는다** (`fade_out_background` 기본 false). 어둡게 하는 모달만 `elevated_surface_background`를 80% 불투명도로 깐다. 모달 바깥을 왼쪽 클릭하면 닫히고 모달 안쪽 클릭은 막힌다 | [modal_layer.rs:50-71](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/modal_layer.rs#L50-L71) · [modal_layer.rs:298-331](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/workspace/src/modal_layer.rs#L298-L331) |
| P-002 | 기본 크기 | 폭 **544px**(34rem), 최대 높이 **384px**(24rem). 개별 Picker가 폭·높이를 따로 정하면 그 값을 쓴다 (4.13.5). 항목이 적으면 높이가 항목 수에 맞게 줄고 384까지 늘리지 않는다 | [picker.rs:45-46](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L45-L46) · [picker.rs:601-634](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L601-L634) · [render.rs:220-239](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L220-L239) |
| 〃 | 창 크기 제한 | 폭은 창 폭의 **95%** 이하, 높이는 (창 높이 − 160) × 0.95 이하. 일반 Picker는 처음 폭을 최소 폭으로 고정한다 | [shape.rs:237-253](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/shape.rs#L237-L253) · [picker.rs:623-625](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L623-L625) |
| 〃 | 바깥 모양 | `elevation_3`: 배경 `elevated_surface_background`, 모서리 **8px**, 테두리 1px `border_variant`, 안쪽 넘침 숨김. 그림자 4겹: (0, 2px, blur 3, 검정 12%) · (0, 4px, blur 8, 8%) · (0, 8px, blur 16, 4%) · (0, 12px, blur 24, 12%) | [render.rs:72-91](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L72-L91) · [styled_ext.rs:6-12](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/traits/styled_ext.rs#L6-L12) · [elevation.rs:58-90](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/styles/elevation.rs#L58-L90) |

#### 4.13.2 검색 입력칸

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| P-003 | 위치·구분선 | 모달 맨 위. 바로 아래 1px `border_variant` 가로선으로 목록과 나눈다 | [render.rs:96-159](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L96-L159) |
| 〃 | 입력칸 줄 | 높이 **36px**(`h_9`), 좌우 padding **10px**. 왼쪽은 한 줄 편집기(남는 폭 전부), 오른쪽은 Picker마다 다른 아이콘 버튼이나 로딩 표시(gap 4). placeholder는 `text_placeholder` 색 | [head.rs:17-56](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/head.rs#L17-L56) · [render.rs:96-146](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L96-L146) |
| 〃 | 다중 선택 토글 | 파일 찾기에만 있는 `FileMultiple` 버튼이다. **우리 앱은 파일 찾기 다중 선택을 만들지 않으므로 뺀다** (포함 목록에 없음) | [render.rs:108-144](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L108-L144) |

#### 4.13.3 결과 목록

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| P-004 | 목록 안쪽 여백 | 목록 위아래 **4px**. 스크롤바는 기본 숨김이고 **명령 팔레트만 항상 켠다** | [picker.rs:582](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L582) · [picker.rs:1541](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L1541) · [render.rs:251-257](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L251-L257) |
| 〃 | 항목 모양 | 항목 바깥 좌우 **4px**, 안쪽 좌우 **6px**(모달 가장자리에서 총 10px), 모서리 **4px**, 내부 gap **4px**, 위아래 padding **4px씩**(`Sparse`) | [list_item.rs:298-310](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L298-L310) · [list_item.rs:360-406](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L360-L406) |
| 〃 | 행 높이 | 1줄 항목 = 글자 줄 높이 + 8. 글자 14px × 줄 높이 1.618 = 22.652 → **약 30.65px**(소스 계산). 조사의 "28~30"은 줄 높이를 20~22로 본 추정이라 소스 계산을 따른다. 명령 팔레트는 항목 안에 위아래 1px가 더해져 약 32.65. 2줄 항목(최근 폴더 모달: 이름 + 작은 경로)은 대략 44px(추정) | [list_item.rs:365-368](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/list/list_item.rs#L365-L368) · [command_palette.rs:768-815](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/command_palette/src/command_palette.rs#L768-L815) |
| 〃 | 구분선 | 지정한 인덱스 다음에 1px `border_variant` 가로선 | [picker.rs:1491-1501](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L1491-L1501) |
| 〃 | 선택·hover | 선택 항목 배경 `ghost_element_selected`. **마우스를 움직이면 그 항목이 바로 선택된다**(`select_on_hover` 기본 true). hover 배경은 `ghost_element_hover`, 누름 `ghost_element_active`. 맞는 글자 강조는 `text_accent` | [picker.rs:213-215](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L213-L215) · [picker.rs:1452-1459](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L1452-L1459) · [highlighted_label.rs:220-237](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/label/highlighted_label.rs#L220-L237) |
| 〃 | 빈 결과 | "No matches" (Muted 색). 세로 padding 8, 안쪽은 비활성 항목 모양 | [picker.rs:227-229](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/picker.rs#L227-L229) · [render.rs:260-272](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/picker/src/render.rs#L260-L272) |

#### 4.13.4 Picker 색 토큰

| ID | 요소 | 토큰 |
| --- | --- | --- |
| P-006 | 모달 배경 / 테두리 | `elevated_surface_background` / `border_variant` (1px, 모서리 8) |
| 〃 | 입력칸 아래·푸터 위 구분선 | `border_variant` |
| 〃 | 입력 글자 / 커서 / placeholder | `text` / `players[0].cursor` / `text_placeholder` |
| 〃 | 항목 배경: 기본 / 선택 / hover | 투명 / `ghost_element_selected` / `ghost_element_hover`(누름 `ghost_element_active`) |
| 〃 | 항목 글자: 주 / 보조(경로·빈 결과) | `text` / `text_muted` |
| 〃 | 맞는 글자 강조 | `text_accent` |
| 〃 | 오른쪽 보조 아이콘 | `icon_muted` |

값은 [03-THEME-ICONS.md](03-THEME-ICONS.md)의 테마 표를 쓴다 (라이트·다크 같은 토큰).

#### 4.13.5 Picker별 치수와 문구

| ID | Picker | 폭 / 최대 높이 | placeholder | 푸터·오른쪽 요소 | 출처 |
| --- | --- | --- | --- | --- | --- |
| P-007 | 명령 팔레트 | **608px**(38rem) / 384 | "Execute a command..." | 항목 오른쪽에 키 배지, 최근 사용 명령은 `HistoryRerun`(14px)이고 hover 시 `Close`로 바뀐다. **푸터**: padding 6, gap 4, 오른쪽 정렬, 위 1px `border_variant`. 버튼 "Add Keybinding…"(또는 "Change Keybinding…") + "Run" | [command_palette.rs:147-154](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/command_palette/src/command_palette.rs#L147-L154) · [command_palette.rs:768-868](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/command_palette/src/command_palette.rs#L768-L868) |
| P-008 | 파일 찾기 (Go to File) | **544px** / 384 | "Search project files..." | 항목: 파일 아이콘(16, Muted) + 파일명 + 상대 경로(작은 글자, Muted, 앞쪽 말줄임). 오른쪽 끝에 최근 항목이면 `HistoryRerun`(14), 아니면 14px 빈 칸. 최근 항목과 검색 결과 사이 1px 구분선. **미리보기·다중 선택·"Include Ignored Files" 토글·푸터(Preview, Actions…)는 만들지 않는다** | [file_finder.rs:358-365](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L358-L365) · [file_finder.rs:2022-2034](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L2022-L2034) · [file_finder.rs:2281-2360](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_finder/src/file_finder.rs#L2281-L2360) |
| P-009 | 최근 폴더 (Open Recent), 모달 | **672px**(42rem) / 384 | "Search projects…" (한 글자 `…`) | 폴더 이름 아래에 작은 Muted 경로 줄. 구역 머리글 + 구분선 (14.4절) | [recent_projects.rs:761-784](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L761-L784) · [recent_projects.rs:877](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L877) · [recent_projects.rs:932-934](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L932-L934) |
| 〃 | 최근 폴더, 팝오버 (사이드바 폴더 추가 버튼) | **320px**(20rem) / **320px** | 같음 | 폴더 이름만 한 줄(전체 경로는 hover 툴팁). **푸터**: padding 6, gap 4, 위 1px `border_variant`, 전체 폭 버튼 "Open Local Folders" | [recent_projects.rs:668-693](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L668-L693) · [recent_projects.rs:1688-1757](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/recent_projects/src/recent_projects.rs#L1688-L1757) |
| P-010 | 테마 선택기 | **544px** / 384 | "Select Theme..." | 원래 활성 테마 항목에만 `Check`(16, Muted). **푸터**: padding 8, 위 1px `border_variant`, 오른쪽에 "**Install Themes**" 버튼 (확장 관리 화면을 Themes 필터로 연다, 02 16.1절). 왼쪽의 "View Theme Docs"(Zed 문서 링크)는 뺀다 | [theme_selector.rs:116-125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L116-L125) · [theme_selector.rs:505-569](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L505-L569) |
| P-011 | 아이콘 테마 선택기 | **544px** / 384 | "Select Icon Theme..." | 테마 선택기와 같은 구조. 푸터 오른쪽 "**Install Icon Themes**" 버튼 (확장 관리 화면을 Icon Themes 필터로 연다), 왼쪽 "View Icon Theme Docs"는 뺀다 | [icon_theme_selector.rs:51-60](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L51-L60) · [icon_theme_selector.rs:162-361](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L162-L361) |
| P-012 | 줄 이동 (Go to Line) | **384px**(24rem) / 입력칸 + 안내줄 2단 고정, 약 62px | 현재 위치 "{줄}:{열}" | Picker를 쓰지 않는 자체 모달. `elevation_2`(그림자 2겹), 위 입력칸 padding 8×4 + 아래 1px `border_variant`, 아래 안내줄 padding 8×4 (Muted). 동작은 02 13.6절(E-103~E-105) | [go_to_line.rs:329-348](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/go_to_line/src/go_to_line.rs#L329-L348) |

#### 4.13.6 대화상자

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| P-013 | 방식 (판단) | Zed는 Windows에서 `use_system_prompts` 기본 true라 **OS TaskDialog**를 띄운다 (창 제목 "Info"·"Warning"·"Critical", 아이콘 정보·경고·오류, 큰 글 = 메시지, 작은 글 = 상세, 첫 버튼이 기본, "Cancel" 버튼은 Esc에 연결). **우리 앱도 같은 방식(Windows `TaskDialogIndirect`)을 쓴다.** 그래서 이 대화상자의 외형은 OS가 정하고 문서에 치수를 적지 않는다. 메시지의 백틱은 OS 창에서 **글자 그대로 보인다** (인라인 코드로 바뀌지 않는다) | [ui_prompt.rs:20-27](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui_prompt/src/ui_prompt.rs#L20-L27) · [window.rs:723-815](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui_windows/src/window.rs#L723-L815) · [default.json:293](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L293) |
| 〃 | 내장 모달 방식 (참고, 만들지 않는다) | Zed가 `use_system_prompts: false`일 때 쓰는 직접 그린 모달: 오버레이 검정 20%, 폭 **320px**, padding 16, gap 16, `elevation_3`, 주 문구 16px `text`(마크다운, 백틱은 인라인 코드), 상세 12px `text_muted`, 버튼은 세로로 전체 폭(gap 4), 첫 버튼 강조(`info_background` + `info_border`), 나머지는 테두리형(`element_background` + `border_variant`) | [ui_prompt.rs:111-206](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui_prompt/src/ui_prompt.rs#L111-L206) |
| P-018 | About 창 | **독립 부동 창** 440×300px, 화면 중앙, 크기 조절·최소화 불가. 제목 "About {앱 이름}". 배경 `editor_background`, 글자 `text`, padding 16, gap 16, 가운데 정렬, 위·아래로 나눈다 | [zed.rs:1713-1731](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1713-L1731) · [zed.rs:1604-1689](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1604-L1689) |
| 〃 | About 안쪽 | 위(gap 8, 가운데): 앱 아이콘 **64×64**, 제목줄 "{앱 이름} {버전}"(Headline), 커밋이 있으면 "Commit"(XSmall, Muted) 아래 값(Small), "Version"(XSmall, Muted) 아래 값(Small). 아래(gap 4, 좌우 균등): **[OK]**(테두리형, 포커스 때 강조) **[Copy]**(강조형). Copy는 "{메시지}\nCommit: {커밋}\nVersion: {버전}"을 클립보드에 넣고 창을 닫는다 | [zed.rs:1584-1596](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1584-L1596) · [zed.rs:1604-1689](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/zed/src/zed.rs#L1604-L1689) |

#### 4.13.7 인라인 이름 입력칸 (프로젝트 패널)

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| P-019 | 입력칸 | 편집 중인 행의 이름 자리에 한 줄 편집기를 끼운다: 높이 **24px**(`h_6`), 폭은 아이콘 오른쪽부터 행 끝까지. 행 높이는 일반 항목과 같은 26 (24 + 위아래 테두리 1px, L-36). 들여쓰기·아이콘 칸도 그대로 | [project_panel.rs:6784-6786](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6784-L6786) · [project_panel.rs:6628-6635](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6628-L6635) |
| 〃 | 파일 아이콘 갱신 | 파일 아이콘 설정이 켜져 있고 입력한 글자가 **3자 이상**이면 입력 중인 이름으로 아이콘을 바로 바꾼다 | [project_panel.rs:6132-6137](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6132-L6137) |
| P-020 | 테두리 | 행은 모서리 0, 테두리 **위·아래·왼쪽 1px, 오른쪽 2px**. 입력칸에 포커스가 있을 때 색: 정상 `panel_focused_border`, 경고 `status.warning`, 오류 `status.error`. 포커스가 없으면 배경색(안 보임) | [project_panel.rs:659-677](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L659-L677) · [project_panel.rs:6165-6199](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6165-L6199) · [project_panel.rs:6273-6278](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6273-L6278) |
| P-021 | 오류·경고 배너 | 검증 결과가 있으면 편집 중인 행 **바로 아래**에 겹쳐 띄운다: 위치 왼쪽 −1px, 오른쪽 −0.5px, 위아래 padding **4px**, 좌우 **8px**, 테두리 1px(오류 `status.error` / 경고 `status.warning`), 배경 `background`, 글자 Small(14px)이고 같은 색 | [project_panel.rs:6842-6861](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/project_panel/src/project_panel.rs#L6842-L6861) |

### 4.14 키맵 편집 화면

사용자 결정으로 포함한 기능이다 (Zed Keymap Editor와 같게, Ctrl+K Ctrl+S). 키맵 편집 화면 조사(`zed-keymap-editor.md`)의 ID(K-001~K-097)를 그대로 쓴다. 동작·키·툴팁·저장 규칙은 [02-INTERACTION.md](02-INTERACTION.md) 15절이다. 화면은 가운데 Pane의 **일반 탭**(제목 "Keymap Editor")이다. 치수는 논리 px(rem 16)이고, 기준 화면 두 장에는 이 화면이 없어 **모두 소스 값**이다.

```
┌ Keymap Editor (p 8, 세로 gap 4, 배경 editor_background) ───────────────────────────┐
│ [검색 입력 (h 32, px 8, 테두리 1 border, rounded_md)] [⌨][⚲ 필터][Edit in JSON][Create Keybinding] │
│ (키 검색 모드일 때) [키 입력칸 KeystrokeInput (최소 높이 40)] [Aa 정확 일치] [빈 칸 min_w 320]       │
├──────┬──────────────┬──────────────┬────────────┬──────────────────────┬────────┤
│      │ Action       │ Arguments    │ Keystrokes │ Context              │ Source │
├──────┼──────────────┼──────────────┼────────────┼──────────────────────┼────────┤
│ ✎/⚠/ⓘ│ Go Back      │ <no arguments>│ Alt-Left  │ Pane                 │ Default│  ← 줄무늬
└──────┴──────────────┴──────────────┴────────────┴──────────────────────┴────────┘
```

#### 4.14.1 바깥 상자와 위쪽 줄

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| K-010 | 바깥 상자 | 안쪽 여백 **8px**, 세로 gap **4px**, 배경 `editor_background` | [keymap_editor.rs:2018-2035](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2018-L2035) |
| K-011 | 위쪽 줄 | 가로, gap **8px**: [검색 입력 상자] [버튼 묶음] | [keymap_editor.rs:2035-2130](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2035-L2130) |
| K-012 | 검색 입력 상자 | 높이 **32px**, 좌우 padding 8, 테두리 1px `border`, 모서리 6px(`rounded_md`), 남는 폭을 모두 차지. placeholder "Filter action names…" | [keymap_editor.rs:2040-2058](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2040-L2058) · [keymap_editor.rs:585](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L585) |
| K-013 | 버튼 묶음 | gap 4, **최소 폭 320px**(20rem). 순서: ① **키로 찾기**(`Keyboard`, Small, 키 검색 모드면 선택 상태) ② **필터**(`Filter`, 메뉴) ③ **Edit in JSON** 버튼 ④ **Create Keybinding** 버튼. ③④ 오른쪽에 단축키를 작게(10px) 붙인다 | [keymap_editor.rs:2059-2125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2059-L2125) · [keymap_editor.rs:1700-1715](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1700-L1715) |
| K-014 | 키 검색 줄 | 키 검색 모드에서만 위쪽 줄 아래에 나타난다: [키 입력칸 (K-050)] [정확 일치 토글 `CaseSensitive`, 정사각] [빈 칸 최소 폭 320, 위 버튼 묶음과 오른쪽 끝 정렬] | [keymap_editor.rs:1980-1995](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1980-L1995) · [keymap_editor.rs:2125-2135](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2125-L2135) |

#### 4.14.2 표

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| K-020 | 표 | 6열, **줄무늬**, 가상 목록(행 수만큼). 열 폭은 **끌어서 재분배**한다 | [keymap_editor.rs:2136-2155](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2136-L2155) · [keymap_editor.rs:58](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L58) |
| K-021 | 머리글 | "" (아이콘 열) · "Action" · "Arguments" · "Keystrokes" · "Context" · "Source" | [keymap_editor.rs:2152](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2152) |
| K-022 | 처음 열 폭 | 아이콘 열 **36px 고정**, 나머지는 비율 Action **0.25** · Arguments **0.20** · Keystrokes **0.14** · Context **0.45** · Source **0.08** | [keymap_editor.rs:653-664](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L653-L664) |
| K-030 | 행 기본 | 행 테두리 2px(평소 색은 보이지 않음, 추정), 줄무늬 배경 | [keymap_editor.rs:2364-2372](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2364-L2372) |
| K-031 | 행 선택 | 테두리 색 `panel_focused_border` | [keymap_editor.rs:2369-2372](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2369-L2372) |
| K-032 | 사용자 바인딩끼리 충돌 | 행 배경 `status.error_background` | [keymap_editor.rs:2365-2368](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2365-L2368) |
| K-033 | 다른 바인딩에 가려진 행 | 행 **불투명도 0.5** | [keymap_editor.rs:2330-2358](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2330-L2358) |
| K-036 | 아이콘 열 버튼 | `Compact` 크기(**18px**) 정사각. 선택한 행은 항상, 다른 행은 hover 때만 보인다. 종류별 아이콘은 02 15.2절 | [keymap_editor.rs:1160-1255](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L1160-L1255) · [keymap_editor.rs:2403-2407](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2403-L2407) |

#### 4.14.3 키 입력칸 (KeystrokeInput)

키 검색 줄과 편집 모달이 같은 구성요소를 쓴다.

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| K-050 | 상자 | 좌우 padding **12px**, gap 8, **최소 높이 40px**, 모서리 6px, 테두리 1px: 포커스 있으면 `border_focused`, 없으면 `border_variant`. 배경: 녹화 중 = `editor_background`에 `text_accent` 10%를 섞은 색, 평소 = `editor_background` | [keystroke_input.rs:466-560](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L466-L560) |
| K-051 | 안쪽 배치 | [왼쪽: 녹화 표시 (폭 **64px**)] [가운데: 입력한 키, **14px**, Default 색] [오른쪽: 버튼 (폭 **64px**)] | [keystroke_input.rs:362-384](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L362-L384) · [keystroke_input.rs:560-620](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L560-L620) |
| K-052 | 녹화 표시 | 녹화 중: 높이 **16px** 작은 상자(테두리 1px `border`, 배경 위와 같은 섞은 색, 모서리 4px, gap 2) 안에 **빨간 원**(`Circle`, Error 색, **2초 주기로 불투명도 0.4~0.8 깜빡임**) + "**REC**"(XSmall, Error). 키 검색 모드: Accent 색 원 + "**SEARCH**" | [keystroke_input.rs:475-525](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L475-L525) |
| K-053 | 오른쪽 버튼 | 녹화 중 `Stop`, 평소 녹화 아이콘(편집 = `PlayFilled`, 검색 = `MagnifyingGlass`). 포커스가 없으면 Muted 색. 툴팁은 02 15.9절 | [keystroke_input.rs:585-620](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/ui_components/keystroke_input.rs#L585-L620) |

#### 4.14.4 바인딩 편집·추가 모달

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| K-080 | 모달 | 폭 **34rem(544px)**, `elevation_3`(모달 표면), 화면 위쪽 가운데 모달 층 (추정) | [keymap_editor.rs:3075-3090](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3075-L3090) |
| K-081 | 머리 | 편집: 액션 이름(Default) + 아래 액션 설명(Muted), 아래 1px `border_variant`. 새로 만들기: 라벨 "Create Keybinding" + 액션 입력칸 | [keymap_editor.rs:3090-3125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3090-L3125) · [keymap_editor.rs:2585-2590](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2585-L2590) |
| K-082 | 본문 | 세로, gap **10px**: ① "Edit Keystroke" + 키 입력칸 (+ 같은 키 안내와 "View" 버튼) ② 인자가 있는 액션이면 "Edit Arguments" + JSON 입력(14px, 줄 높이 1.2) ③ 컨텍스트 입력칸 "Edit Context" ④ 오류 배너 | [keymap_editor.rs:3124-3200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3124-L3200) · [keymap_editor.rs:2529-2531](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L2529-L2531) · [keymap_editor.rs:3415-3495](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3415-L3495) |
| K-083 | 바닥 | 오른쪽 [Cancel] [Save], gap 4 | [keymap_editor.rs:3200-3215](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/keymap_editor/src/keymap_editor.rs#L3200-L3215) |

색 토큰 이름은 위 표에 있다 (`editor_background`, `border`, `border_variant`, `border_focused`, `panel_focused_border`, `text_accent`, `status.error_background`, Error·Warning 색). 값은 [03-THEME-ICONS.md](03-THEME-ICONS.md)의 테마 표를 쓴다. 라이트·다크 모두 같은 토큰이다.

#### 4.14.5 키맵 편집 화면에서 확신 낮은 부분

- **화면 근거:** 두 기준 화면에 이 화면이 없다. 치수·색은 모두 소스 값이다.
- **행 테두리 (K-030):** 선택되지 않은 행의 2px 테두리 색은 확인하지 못했다.
- **모달 위치 (K-080):** 위쪽 가운데 모달 층이라는 것은 다른 모달(줄 이동)과 같다고 본 추정이다.
- **줄 번호:** `keymap_editor.rs`가 4,500줄을 넘어 K-023~K-037, K-080~K-087의 줄 범위는 근사치(±수십 줄)다.

### 4.15 확장 관리 화면 (Extensions)

사용자 지시로 포함한 기능이다. **테마·아이콘 테마·언어 확장 3종만** 다루고, 사용자 결정(`_decisions.md` "확장 세부")에 따라 **로컬 설치만** 한다: 온라인 목록·검색·내려받기는 없고, 화면에는 **설치된 확장 목록**만 보인다. 확장 조사(`zed-extensions.md`)의 ID(X-44~X-50)를 그대로 쓰되 온라인 전용 요소(설치 상태 토글, Install·Upgrade 버튼, 내려받기 수, ⋯ 메뉴, 로딩 표시)는 뺐다. 동작·키·툴팁·설치 흐름은 [02-INTERACTION.md](02-INTERACTION.md) 16절이다. 화면은 가운데 Pane의 **일반 탭**(제목 "Extensions")이다. 치수는 논리 px(rem 16)이고 두 기준 화면에 이 화면이 없어 **모두 소스 값**이다.

```
┌ Extensions (배경 editor_background, 툴바 없음) ──────────────────────────────────────┐
│ 위·좌우 padding 16, 행 gap 16                                                       │
│ Extensions (Headline Large)               [Install Extension…] [Install Dev Extension]│
│ [🔍 Search extensions... (h 32, min 384, flex 1)]                                    │
│ [All] [Themes] [Icon Themes] [Languages]  ← 카테고리 바 (위아래 10, 아래 1px 선)       │
├──────────────────────────────────────────────────────────────────────────────────┤
│ ┌ 카드 (위 마진 16, h 110, p 12, gap 8, 테두리 1, 모서리 6) ─────────────────────────┐ │
│ │ 이름 v1.0.0 [Themes][Icon Themes]                                     [Uninstall]│ │
│ │ 설명 (1줄 말줄임)                                                                  │ │
│ │ 👤 Authors: …                                                    [저장소 아이콘]    │ │
│ └──────────────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────────┘
```

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| X-44 | 탭 | 탭 제목 "**Extensions**". 툴바 없음. 루트 배경 `editor_background`. 활성 가운데 Pane에 이미 이 탭이 있으면 그 탭을 활성화하고 없으면 새 탭으로 연다 | [extensions_ui.rs:119-169](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L119-L169) · [extensions_ui.rs:1640-1666](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L1640-L1666) |
| X-45 | 헤더 1행 | 세로 배치, 위 padding **16**, 좌우 padding **16**, 행 gap **16**. 왼쪽: 제목 "Extensions"(Headline Large). 오른쪽: **[Install Dev Extension]** 버튼 (테두리형, 크기 Medium = 28px, Zed와 같다) 그 왼쪽에 **[Install Extension…]** 버튼 (같은 모양, 로컬 설치용으로 우리가 더한 것, 판단) | [extensions_ui.rs:1478-1501](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L1478-L1501) |
| X-46 | 검색 상자 | 높이 **32px**, 최소 폭 **384px**(24rem), 남는 폭 모두 차지, 왼쪽 padding **6**, 오른쪽 **8**, 내부 gap **8**, 테두리 1px `border`, 모서리 **6px**. 왼쪽 `MagnifyingGlass`(Muted). 입력은 한 줄, 글자 **14px**, 줄 높이 **1.3**, placeholder "**Search extensions...**". 로딩 표시(`LoadCircle`)는 **만들지 않는다** (온라인 조회가 없다). 설치된 확장만 거른다 | [extensions_ui.rs:406-413](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L406-L413) · [extensions_ui.rs:535-592](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L535-L592) · [extensions_ui.rs:862-897](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L862-L897) |
| X-48 | 카테고리 바 | 가로 배치, gap **8**, 위아래 padding **10**, 좌우 **16**, 아래 1px `border_variant`, 넘치면 가로 스크롤. 버튼은 **All, Themes, Icon Themes, Languages** 4개만 (Zed의 Language Servers·MCP Servers·Snippets·Debug Adapters는 범위 밖). 선택된 버튼은 채움(`Filled`), 나머지는 `Subtle`. 위쪽 설치 상태 토글(All · Installed · Not Installed, Zed의 X-47)은 **만들지 않는다** (전부 설치된 확장이다) | [extensions_ui.rs:1550-1599](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L1550-L1599) · [extension_card.rs:427-441](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/components/extension_card.rs#L427-L441) |
| X-49 | 목록 | 가상 목록. 좌우 padding **16**, 아래 padding **16**, 오른쪽 세로 스크롤바. 빈 상태 문구 위 padding 12, 좌우 16 | [extensions_ui.rs:899-934](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L899-L934) · [extensions_ui.rs:1601-1636](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L1601-L1636) |
| 〃 | 카드 바깥 | 위 마진 **16**, 폭 100%, **고정 높이 110px**, 안쪽 padding **12**, 세로 gap **8**, 위·아래 양 끝 정렬, 배경 `elevated_surface_background` 불투명도 50%, 테두리 1px `border_variant`, 모서리 **6px** | [extension_card.rs:594-625](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/components/extension_card.rs#L594-L625) |
| X-50 | 카드 1행 | 양 끝 정렬. 왼쪽(gap 8, 글자 기준선 정렬): 이름(Headline Small) + 버전 "v{버전}"(Headline XSmall, Muted; 개발 확장은 "v{버전} (dev)") + **기능 배지**(gap 4: "Themes", "Icon Themes", "Languages"만). 오른쪽(gap 4): 상태별 버튼 (02 16.3절). "(v{설치된 버전} installed)" 표기는 업그레이드가 없으므로 쓰지 않는다 | [extensions_ui.rs:688-859](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/extensions_ui.rs#L688-L859) · [extension_card.rs:626-775](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extensions_ui/src/components/extension_card.rs#L626-L775) |
| 〃 | 카드 2행 | 설명(Small, 한 줄 말줄임)만. 오른쪽의 "Downloads: {숫자}"는 온라인 카드 전용이라 쓰지 않는다 | 위와 같음 |
| 〃 | 카드 3행 | 양 끝 정렬(gap 8). 왼쪽(gap 6): `Person` 아이콘(XSmall, Muted) + 작성자(Small, Muted, 말줄임). 오른쪽(gap 4): 저장소 아이콘 버튼(Small, 저장소 주소가 있을 때만). 온라인 카드 전용 `Ellipsis` 메뉴는 쓰지 않는다 | 위와 같음 |

내장 테마(One Dark, One Light, Catppuccin 4종)와 내장 아이콘 테마(Zed (Default), Catppuccin Icons)는 **본체에 들어 있어 이 목록에 나오지 않는다** (사용자 결정). 색 토큰은 위 표의 이름이다 (`editor_background`, `border`, `border_variant`, `elevated_surface_background`). 값은 [03-THEME-ICONS.md](03-THEME-ICONS.md)의 테마 표를 쓴다.

## 5. 마크다운 미리보기 스타일

Zed 미리보기 화면의 값이다. 기준 화면의 frontmatter 표·제목·인용·표·링크와 맞춰 봤다. 색은 토큰 이름만 적는다 (값은 [03-THEME-ICONS.md](03-THEME-ICONS.md)).

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| L-101 | 본문 폭 | `limit_content_width: true`, **`max_width: 800`px**. 넓으면 가운데 정렬 | [default.json:128-134](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L128-L134) · [markdown_preview_view.rs:2015-2018](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L2015-L2018) |
| — | (참조 L-102) 바깥 여백·배경 | 스크롤 컨테이너 사방 padding **16px**, 세로 스크롤만. 배경 `editor_background`. 미리보기 안 rem은 `markdown_preview.font_size` (없으면 `ui_font_size` = 16) | [markdown_preview_view.rs:1912-1955](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1912-L1955) · [default.json:117-130](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L117-L130) |
| L-103 | 본문 글자 | 1rem(16px), 줄 높이 1.5, 문단 간격 16px, 목록 간격 12px, 글자색 `text`. 글꼴은 UI 글꼴, 코드는 버퍼 글꼴 | [markdown.rs:356-370](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L356-L370) |
| L-104 | 제목 | 굵기 `heading_font_weight` 600, 줄 높이 1.25, 색 `text`. 크기: h1 1.75rem(28) / h2 1.4(22.4) / h3 1.2(19.2) / h4 1.0(16) / h5 0.875(14) / h6 0.85(13.6, 색 `text_muted`). 위 여백 24, 아래 12 (추정: 최종 덮어쓰기 순서 기준) | [markdown.rs:384-410](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L384-L410) · [default.json:126](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L126) |
| L-105 | 제목 밑줄 | h1: 아래 padding 8 + `border_b_1`, h2: 아래 padding 4 + `border_b_1`. 색 `border_variant` | [markdown.rs:3549-3580](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L3549-L3580) · [markdown.rs:409](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L409) |
| L-106 | 인용 | 왼쪽 padding 16, **왼쪽 테두리 4px** (색 `border`), 글자색 `text_muted` | [markdown.rs:2160-2170](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L2160-L2170) · [markdown.rs:245](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L245) |
| L-107 | 표 | grid. 바깥 테두리 **1.5px** `border` + `rounded_sm`. 셀 padding 가로 10 × 세로 4, 셀 사이 1px 선. **머리행 배경 `title_bar_background` + SEMIBOLD**. 본문 홀수 번째 행 (0부터 셀 때 1, 3, …) 배경 `panel_background` (줄무늬). 열 정렬(가운데·오른쪽) 반영 | [markdown.rs:3079-3155](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L3079-L3155) |
| L-108 | 링크 | 색 `text_accent`, 밑줄 1px (`text_accent` 50% 불투명), 미리보기에서는 배경 없음 | [markdown.rs:299-307](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L299-L307) · [markdown.rs:367](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L367) |
| L-109 | 인라인 코드·코드 블록 | 인라인: 0.875rem, 배경 `editor_foreground` 8%. 블록: padding 12, 위아래 margin 16, 모서리 6, 미리보기에서는 테두리 없음, hover 시 복사 버튼 | [markdown.rs:290-298](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L290-L298) · [markdown.rs:368-380](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L368-L380) · [markdown_preview_view.rs:1121-1125](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L1121-L1125) |
| L-110 | frontmatter → 표 | 파일 맨 앞 YAML frontmatter의 **모든 줄이 공백으로 시작하지 않는 `키: 값`** (키·값 모두 비어 있지 않음)이면 2열 표로 그린다. 하나라도 어기면 코드 블록 스타일 상자로 원문을 보인다 | [markdown_preview_view.rs:400](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown_preview/src/markdown_preview_view.rs#L400) · [parser.rs:175-208](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/parser.rs#L175-L208) |
| L-111 | frontmatter 표 스타일 | 2열 grid, 전체 폭, `border_1`(`border`) + `rounded_sm`. 키 셀: 배경 `panel_background` + SEMIBOLD. 값 셀: `border_l_1`. 두 번째 행부터 `border_t_1` (기준 화면 tags·created·updated 표와 일치) | [markdown.rs:2184-2285](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/markdown/src/markdown.rs#L2184-L2285) |
| L-309 | frontmatter 표 관찰값 | 두 열 폭 **1:1**, 행 높이 약 34, 키 열 글자색 `text_muted` (기준 화면에서 검토자가 잰 값, 소스 값은 대조하지 않았다) | 관찰 |

## 6. 기준 화면 채용·제외

| ID | 영역 | 채용 | 제외 | 관련 항목 |
| --- | --- | --- | --- | --- |
| L-141 | 타이틀 바 | ☰ 메뉴 (우리 기능만), 현재 폴더 이름, ∨ 드롭다운 메뉴 (우리 항목만), 창 버튼 | Sign In 버튼, Git 브랜치, 통화·연결 상태, 업데이트 알림 | L-85~L-89, I-600~I-612 |
| L-142 | 맨 왼쪽 사이드바 | 등록 폴더 목록: 헤더, 항목, 선택 강조, 폴더 추가 | 스레드 검색, 세션·시간 | L-12, L-33~L-35, L-96 |
| L-143 | 왼쪽 Dock (프로젝트 패널) | 루트, 접힌 경로(`docs \ guides`), 폴더·파일 아이콘, 들여쓰기 안내선, 가로 스크롤 | Git 상태 색·아이콘, 진단 표시 | L-36~L-52 |
| L-144 | 탭 바 | ← → 이동, 편집 탭, 미리보기 탭(책 아이콘 "Preview X.md"), 넘침 스크롤, 분할, 최대화, `+`(New File만) | 우리 기능이 아닌 `+` 메뉴 항목 | L-22~L-27, L-61~L-74 |
| L-145 | 편집기·미리보기 | 마크다운 미리보기(frontmatter 표, 제목, 인용, 표, 링크), 코드 구문 강조 편집기 | — | L-101~L-111 |
| L-146 | 상태 표시줄 | 사이드바 토글, 폴더 추가, 프로젝트 패널 토글, 커서 위치 | 기록, 검색, 진단, 편집 예측, 터미널, 디버그, 개요, 협업, Git, AI | L-28, L-29, L-91~L-98 |

## 7. 기준 화면 측정 대조

기준 화면(논리 px = 이미지 좌표 환산)에서 잰 값과 소스 값을 비교한다. 04-APP-SPEC의 일치도 판정이 이 표를 쓴다.

| ID | 대상 | 소스 값 | 관찰 | 비고 |
| --- | --- | --- | --- | --- |
| L-147 | 타이틀 바 높이 | 32 | 약 32 | 일치 |
| L-148 | 탭 바 높이 | 32 | 약 32 | 일치 |
| L-149 | 상태 표시줄 높이 | 30 + 보정 | 약 31 | 일치 |
| L-150 | 프로젝트 패널 항목 행 | 24 + 2 = 26 | 약 26 | 일치 (L-36) |
| L-151 | 들여쓰기 단계 | 20 | 약 20 | 조사 자료의 17~18은 재측정에서 20으로 나왔다 |
| L-152 | 사이드바 폭 | 기본 300, 하한 200 | 약 200 (두 화면 모두) | 사용자 조정값. 우리 앱 초기값 300 |
| L-153 | 프로젝트 패널 폭 | 기본 240 | 약 284 (Mocha) / 약 215 (Latte) | 화면마다 다름 = 사용자 조정값. 우리 앱 초기값 240 |

## 8. 관련 설정 기본값

`assets/settings/default.json`의 값이다. 우리 앱이 쓰는 키만 적는다.

| ID | 키 | 기본값 | 우리 앱 | 줄 |
| --- | --- | --- | --- | --- |
| L-120 | `ui_font_size` / `ui_font_family` / `ui_font_weight` | 16 / ".ZedSans" / 400 | 16 | [default.json:59-71](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L59-L71) |
| L-121 | `buffer_font_size` / `buffer_font_family` / `buffer_line_height` | 15 / ".ZedMono" / "comfortable" | 15 | [default.json:31-54](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L31-L54) |
| L-122 | `window_decorations` | "client" | client | [default.json:282](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L282) |
| L-123 | `title_bar.show_menus` / `show_project_items` | false / true | 메뉴 버튼 + 프로젝트 이름 + ∨ 사용자 메뉴(`show_user_menu`, 항목은 우리 것만). `show_branch_name`·`show_sign_in`은 쓰지 않는다 | [default.json:620-640](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L620-L640) |
| L-124 | `agent.threads_sidebar.position` / `default_width` | "left" / 300 | left / 300 | [default.json:1191-1197](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1191-L1197) |
| L-125 | `project_panel.dock` / `default_width` / `button` / `starts_open` | **"right"** / 240 / true / true | **left** / 240 / true / true | [default.json:880-913](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L880-L913) |
| L-126 | `project_panel.entry_spacing` / `indent_size` / `file_icons` / `folder_indicator` | "comfortable" / 20 / true / "icon" | 같음 | [default.json:892-907](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L892-L907) |
| L-127 | `project_panel.auto_fold_dirs` / `hide_root` / `bold_folder_labels` / `sticky_scroll` | true / false / false / true | true / false / false / 제외 (L-51) | [default.json:916-1001](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L916-L1001) |
| L-128 | `project_panel.indent_guides.show` / `scrollbar.horizontal_scroll` | "always" / true | 같음 | [default.json:920-960](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L920-L960) |
| L-129 | `resize_all_panels_in_dock` | ["left"] | — | [default.json:879](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L879) |
| L-130 | `tab_bar.show` / `show_nav_history_buttons` / `show_tab_bar_buttons` | true / true / true | 같음 | [default.json:1475-1485](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1475-L1485) |
| L-131 | `tabs.close_position` / `file_icons` / `show_close_button` / `activate_on_close` | "right" / false / "hover" / "history" | 같음. `git_status`는 쓰지 않는다 | [default.json:1495-1531](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1495-L1531) |
| L-132 | `preview_tabs.enabled` / `enable_preview_from_project_panel` / `enable_preview_from_file_finder` | true / true / false | 같음 | [default.json:1534-1543](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1534-L1543) |
| L-133 | `max_tabs` | null (제한 없음) | 같음 | [default.json:1473](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1473) |
| L-134 | `status_bar.cursor_position_button` | true | true. `active_language_button`·`show_active_file`은 쓰지 않는다 | [default.json:2031-2039](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2031-L2039) |
| L-135 | `line_indicator_format` | "long" | long | [default.json:2945](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2945) |
| L-136 | `markdown_preview.font_size` / `heading_font_weight` / `limit_content_width` / `max_width` / `open_markdown_files_in_preview` | null / 600 / true / 800 / false | 같음 | [default.json:117-134](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L117-L134) |
| — | (참조 L-137) `active_pane_modifiers.border_size` / `inactive_opacity` | 0.0 / 1.0 | 같음 | [default.json:88-93](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L88-L93) |
| L-138 | `bottom_dock_layout` / `pane_split_direction_horizontal` / `_vertical` | "contained" / "down" / "right" | 아래 Dock이 없으므로 분할 방향만 같음 | [default.json:97-101](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L97-L101) |
| L-139 | `zoomed_padding` | true | true | [default.json:253](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L253) |
| L-140 | `unstable.ui_density` | Default | Default 고정 (키 이름·공개 여부는 미확인) | [ui_density.rs:21-28](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/ui_density.rs#L21-L28) |

## 9. 확신 낮은 부분

- **줄 번호 정확도:** 일부 링크의 줄 범위는 grep으로 잡은 근사치다. L-17, L-48, L-74, L-86, L-97, L-104, L-105, L-109, L-111이 그렇다. 파일과 커밋은 정확하다.
- **들여쓰기 (L-37):** 소스값은 20이다. 조사 자료의 관찰 17~18은 이 문서 작성 때 이미지를 다시 재서 약 20으로 바꿨다. 이미지는 축소본(2509 폭)을 보고 환산했으므로 ±1px 오차가 있다.
- **행 높이 (L-36):** 24 + 2 = 26으로 소스 근거가 생겼다. 소수 px(예: 줄 높이 계산값)는 실제 렌더에서 어떻게 반올림되는지 실행 대조 전이다.
- **창 모서리·테두리 (L-311):** Latte 화면의 둥근 모서리와 테두리는 소스 값(10px, 1px)과 함께 관찰로 확인했다. Windows에서 최대화했을 때 장식이 없어지는지는 추정이다.
- **글자색 (L-307):** 상태 표시줄 커서 위치의 글자색은 소스에서 지정을 찾지 못했다. 검토자 측정값만 있다.
- **스크롤바 동작 (4.10절):** 치수는 확인했다. 표시·숨김 시점은 편집기 조사(02 13.7절 E-033)로 정했으나 Windows에서 어떤 OS 설정을 읽는지는 미확인이다. 트랙 클릭·바퀴 규칙은 조사하지 않았다.
- **Picker·대화상자 (4.13절):** 1줄 항목 행 높이 약 30.65px은 줄 높이 계산이다. OS 대화상자(TaskDialog)의 외형은 OS가 정하므로 치수를 적지 않았다. 조사 문서의 "Ayu Dark" 가정과 그 색 값은 쓰지 않았다.
- **확장 관리 화면 (4.15절):** 모두 소스 값이다. 로컬 설치만 하므로 온라인 전용 요소는 뺐다 (사용자 결정). 설치 목록의 정렬과 로컬 설치용 "Install Extension…" 버튼의 모양은 우리 판단이다.
- **편집기 화면 (4.12절):** 모두 소스 값이다. 기준 화면 두 장이 미리보기 탭이라 대조하지 못했다. 거터 폭의 px(E-002, E-003, E-005)는 글꼴 메트릭으로 계산해야 한다 (4.12.8).
- **제목 여백 (L-104):** 먼저 준 여백을 스타일 값 24/12가 덮어쓴다고 보고 최종값을 정했다 (추정).
- **사이드바 푸터 (L-95, L-96, L-98 위치 정리):** 기준 화면의 왼쪽 아래 아이콘이 모두 사이드바 푸터 소속인지는 화면 관찰과 코드 조각을 맞춘 판단이다. 순서·정렬은 미확인이다. 시계(기록) 아이콘은 제외했다.
- **프로젝트 패널 파일 아이콘 크기 (L-40):** 16으로 봤지만 크기를 따로 주는지 미확인이다.
- **사용자 조정값 (L-152, L-153):** 폭 200·285는 사용자가 바꾼 값으로 보이나 저장 위치는 확인하지 않았다.
- **배율:** 200%는 타이틀 바 32px에서 역산했다.
- **`ui_density` 설정 키:** 설정 키 이름과 공개 여부를 default.json에서 찾지 못했다.
- **단위 표 (2절):** Base24 이상의 Default 값은 "단일 값 규칙"에 따른 계산이며 정의 줄을 따로 확인하지 않았다.

---

이전: [../README.md](../README.md) · 다음: [02-INTERACTION.md](02-INTERACTION.md) · 입구: [../README.md](../README.md)
