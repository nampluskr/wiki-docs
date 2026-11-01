---
tags: [rust, gui, zed, theme, icons, tauri, gpui, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 01-zed · 03 · 테마·아이콘·글꼴

> 이 파일을 읽는 때: 색 토큰, 아이콘, 글꼴, 테마 선택 동작을 정할 때. 02-gpui·03-tauri 의 테마 장(05-THEME-KEYMAP)을 쓰기 전에 읽는다.

| 항목 | 내용 |
| --- | --- |
| 근거 | Zed 커밋 [14b7b0e](https://github.com/zed-industries/zed/tree/14b7b0ef3764d62a85070c8b5ccea105a2ad3235) (`14b7b0ef3764d62a85070c8b5ccea105a2ad3235`) 의 소스·`assets`, Zed 공식 문서, 이 노트북에 설치된 Catppuccin 0.2.27·Catppuccin Icons 1.24.0 확장 |
| 검증 | 소스 대조. 색 값은 JSON 파일에서 기계로 읽어 옮겼다. 실행 검증 안 함 |
| 범위 | 01-LAYOUT·02-INTERACTION 에서 쓰는 화면 영역만. 터미널·Git·진단·AI·Vim 토큰은 뺐다 |
| ID | 조사 자료의 `T-NN` 을 그대로 쓴다. 새 항목은 T-24 부터 |
| 우선순위 | 01-LAYOUT 과 값이 다르면 **01 을 따른다.** 툴팁 모양·위치·지연(L-200 ~ L-272)은 writer-gpui 가 소스로 확정한다. 포함·제외는 사용자 결정(`_decisions.md`)이 최우선이다 |
| 정정 | 조사 자료(agy)의 "추정" 색, One 토큰 값, 기본 아이콘 규칙 일부가 소스와 달라 고쳤다. 고친 곳에 "정정" 이라고 적었다 |

기준 화면이다. 아래 값 표의 Mocha 열과 Catppuccin Latte 아이콘이 이 화면을 만든다.

![기준 화면](assets/zed-reference.png)

두 번째 기준 화면이다. Catppuccin Latte(라이트), 창을 최대화하지 않은 상태다. 아래 2.3절에서 색을 대조한다.

![기준 화면 (Latte)](assets/zed-reference-latte.png)

## 1. 테마 JSON 구조와 토큰

### 1.1 파일 구조 (T-01)

| ID | JSON 경로 | 타입 | 뜻 | 출처 |
|---| --- | --- | --- | --- |
| T-01.1 | `$schema` | string | `https://zed.dev/schema/themes/v0.2.0.json` | [one.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/themes/one/one.json) |
| T-01.2 | `name` | string | 테마 패밀리 이름. `One`, `Catppuccin` | [one.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/themes/one/one.json) |
| T-01.3 | `author` | string | 만든 이. `Zed Industries`, `Catppuccin <releases@catppuccin.com>` | 같은 파일 |
| T-01.4 | `themes[]` | array | 패밀리에 속한 테마. 설정의 `theme` 이 `themes[].name` 으로 고른다 | [공식 문서](https://zed.dev/docs/themes) |
| T-01.5 | `themes[].appearance` | `"light"` \| `"dark"` | 시스템 모드와 연결할 때 쓴다 (6절) | [theme_selector.rs L339-L349](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L339-L349) |
| T-01.6 | `themes[].style` | object | 점으로 이어 쓴 평평한 키(`border.variant`) + `players` + `accents` + `syntax` | [colors.rs](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/styles/colors.rs) |
| T-01.7 | 색 표기 | string | `#RRGGBB` 또는 `#RRGGBBAA`. 두 형식이 한 파일에 섞여 있다. One 은 대부분 8자리지만 6자리도 있다(예: One Dark `editor.line_number` `#4e5a5f`). Catppuccin 은 알파가 있을 때만 8자리다. 파서는 6·8자리를 모두 받고 6자리는 알파 1로 읽는다 | [one.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/themes/one/one.json), [catppuccin-mauve.json](https://github.com/catppuccin/zed/blob/v0.2.27/themes/catppuccin-mauve.json) |

| ID | 패밀리 | 파일 | 테마 수 | 스타일 키 수 | `syntax` 키 수 |
|---| --- | --- | --- | --- | --- |
| T-01.8 | One | `assets/themes/one/one.json` | 2 (One Dark, One Light) | 141 / 139 | 46 / 46 |
| T-01.9 | Catppuccin | `themes/catppuccin-mauve.json` | 4 (Latte, Frappé, Macchiato, Mocha) | 176 | 102 |

### 1.2 누락 토큰 규칙 (T-26)

JSON 에 키가 없으면 `—`, 키는 있는데 값이 `null` 이면 `null` 로 적었다(2.4절). Zed 는 둘을 같게 다룬다(값 없음). 아래는 **소스를 열어 확인한 규칙**이다. 구분: **확정**은 소스에서 읽은 규칙이고, **판단**은 우리 결정이다.

Zed 는 JSON 에서 읽은 값을 내장 대체 테마(`zed_default_dark`, [fallback_themes.rs L60](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L60)) 위에 덮어쓴다. 이 커밋에 대체 테마는 **dark 하나뿐**이다 ([L15-L26](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L15-L26)). 라이트 테마가 값을 빠뜨리면 이 dark 값이 그대로 남는다고 보이지만 실행해서 확인하지는 않았다 (추정).

| ID | 값 없는 토큰 | 어느 테마에서 | 최종 값 | 구분 | 근거 |
| --- | --- | --- | --- | --- | --- |
| T-26.1 | `panel.overlay_background` | One | `panel.background` (알파 1로 고정) | 확정 | [schema.rs L486-L490](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/schema.rs#L486-L490) |
| T-26.2 | `panel.overlay_hover` | 6개 테마 모두 | `panel.background` 에 `element.hover` 를 합성한 색(알파 1) | 확정 | [schema.rs L491-L498](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/schema.rs#L491-L498). 고정 위치 행(sticky scroll)을 쓸 때만 필요하다 (L-47, L-51) |
| T-26.3 | `scrollbar.thumb.active_background` | 6개 테마 모두 | `scrollbar.thumb.background` | 확정 (정정: 이전 판은 hover 였다) | [schema.rs L263-L267](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/schema.rs#L263-L267) |
| T-26.4 | `pane_group.border` | One | `border` | 확정 | [schema.rs L503-L507](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/schema.rs#L503-L507) |
| T-26.5 | `panel.focused_border` | One(`null`) | 대체 테마 값 `#62adef` (hsla 207.8°, 81%, 66%) | 확정(값은 계산) | [fallback_themes.rs L226](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L226) |
| T-26.6 | `panel.indent_guide` | One | 대체 테마 값 `#3b3d45` (hsla 228°, 8%, 25%) | 확정(값은 계산) | [fallback_themes.rs L227](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L227) |
| T-26.7 | `panel.indent_guide_hover`, `panel.indent_guide_active` | One | 대체 테마 값 `#1b1d23` (hsla 225°, 13%, 12%) | 확정(값은 계산) | [fallback_themes.rs L228-L229](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L228-L229) |
| T-26.8 | `scrollbar.thumb.border` | Catppuccin 4종(`null`) | 대체 테마 값 `#3b3d45` | 확정(값은 계산) | [fallback_themes.rs L242](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L242). 이전 판의 "투명" 은 틀렸다 |
| T-26.9 | `editor.indent_guide` | One | 대체 테마 값 `#3b3d45` | 확정(값은 계산) | [fallback_themes.rs L178](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L178) |
| T-26.10 | `editor.indent_guide_active` | One | 대체 테마 값 `#1b1d23` | 확정(값은 계산) | [fallback_themes.rs L179](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L179) |
| T-26.11 | `editor.hover_line_number` | Catppuccin 4종 | 대체 테마 값 `#8a8f97` (hsla 216°, 5.9%, 56.7%). 우리 앱은 줄 번호 hover 색을 **쓰지 않는다** | 확정(값) + 판단(미사용) | [fallback_themes.rs L174](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L174) |
| T-26.12 | `editor.document_highlight.bracket_background` | One | 대체 테마 값은 `gpui::green()` 이다([L187](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L187)). 그 정확한 색은 확인하지 못했고 눈에 띄는 색이라 이 값을 그대로 쓰지 않는다. 우리 앱은 `editor.document_highlight.read_background` 로 대신한다 | 판단 | — |
| T-26.13 | `drop_target.border` | 6개 테마 모두 | 대체 테마 값 `#d7dadf` (hsla 221°, 11%, 86%). 탭 드롭의 2px 삽입선에 쓴다 (I-715) | 확정(값은 계산) | [fallback_themes.rs L134](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L134) |
| T-26.14 | `pane.focused_border` | One(`null`) | 대체 값 `#62adef` 이지만 **미사용**. 활성 Pane 테두리 크기가 0 이다 (L-137, I-729) | 판단 | [fallback_themes.rs L232](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/fallback_themes.rs#L232) |
| T-26.15 | `editor.highlighted_line.background` | Catppuccin 4종(`null`) | **미사용**. 이동·검색으로 줄을 강조하는 기능이 없다 (One 에 값이 있어도 쓰지 않는다) | 판단 | — |
| T-26.16 | `syntax.*` 키 | One 의 `punctuation.markup`·`selector` 는 Catppuccin 에 없고, Catppuccin 의 58개 키는 One 에 없다 | T-27.1 의 탐색 규칙 | 확정 | T-27.1 |
| T-26.17 | 그 밖의 키 | — | 위 표에 없는 키는 우리가 쓰지 않는다 | 판단 | 1.3절 |

대체 테마 색 값은 소스의 hsla 를 16진수로 바꾼 계산값이다(알파 1). 화면으로 비교한 값이 아니다. 04 의 판정(A-31.8)은 "원본에 값이 있으면 그 값, 없거나 `null` 이면 이 표의 최종 값" 이다.

### 1.3 토큰이 쓰이는 곳 (T-40)

값은 2절 표에 있다(참조 열의 "값 T-0x" 가 그 표다). 이 표는 토큰이 어느 화면 요소에 쓰이는지만 적는다. 이 표의 ID(T-40.n)는 사용처 행의 ID 이고, 값 표의 T-02 ~ T-05, T-24 와 다르다. L- 은 01-LAYOUT, I- 는 02-INTERACTION 의 ID 다.

| ID | 토큰 | 쓰이는 곳 | 참조 |
| --- | --- | --- | --- |
| T-40.1 | `background` | 창 맨 바닥 | L-11, 값 T-02 |
| T-40.2 | `border`, `border.variant` | 영역 사이 1px 선, 마크다운 표·제목 밑줄 | L-16, L-105, L-107, 값 T-02 |
| T-40.3 | `border.focused` | 이름 입력란·포커스된 패널 테두리 | L-48, 값 T-02 |
| T-40.4 | `element.*` | 트리 항목 hover(`hover`), 누름(`active`), 선택(`selected`), 비활성(`disabled`) | L-47, 값 T-02 |
| T-40.5 | `ghost_element.*` | 배경 없는 아이콘 버튼. 켜진 토글(`selected`)과 사이드바 선택 항목 | L-92, L-34, 값 T-02 |
| T-40.6 | `elevated_surface.background` | ☰ 메뉴, 툴팁, 선택기 모달 | L-85, T-30, 값 T-02 |
| T-40.7 | `drop_target.background` | 탭·트리 드래그 대상 표시 | I-074, I-094, 값 T-02 |
| T-40.8 | `text` | 본문·제목, 선택된 사이드바 항목의 이름, 타이틀 바 글자 | L-103, T-08.15, 값 T-03 |
| T-40.9 | `text.muted` | 비선택 프로젝트 패널 항목(T-08.14), 비활성 탭, 인용, 사이드바 항목의 보조 글자(`13h` 같은 것은 우리 앱에 없다) | L-106, T-08.14, 값 T-03 |
| T-40.10 | `text.disabled`, `icon.disabled` | 비활성 ← → 아이콘, 이동 기록이 없을 때 (T-08.16) | L-62, 값 T-03 |
| T-40.11 | `icon.accent` | 켜진 토글 아이콘(프로젝트 패널 토글, T-08.17) | L-92, 값 T-03 |
| T-40.12 | `text.accent` | 링크 | L-108, 값 T-03 |
| T-40.13 | `text.placeholder`, `text.disabled` | 빈 입력 안내, 비활성 글자 | 값 T-03 |
| T-40.14 | `icon`, `icon.muted`, `icon.accent`, `icon.disabled` | UI 아이콘. 비활성 ← → 는 `disabled` | L-62, 값 T-03 |
| T-40.15 | `title_bar.background`, `title_bar.inactive_background` | 타이틀 바(창이 포커스를 잃으면 inactive) | L-81, 값 T-04 |
| T-40.16 | `status_bar.background` | 상태 표시줄 | L-91, 값 T-04 |
| T-40.17 | `tab_bar.background`, `tab.active_background`, `tab.inactive_background` | 탭 바 바탕, 활성 탭(편집기와 이어짐), 비활성 탭 | L-61, L-63, 값 T-04 |
| T-40.18 | `toolbar.background` | 탭 바 아래 줄 | L-14, 값 T-04 |
| T-40.19 | `panel.background` | 사이드바와 프로젝트 패널 | L-33, L-47, 값 T-04 |
| T-40.20 | `panel.indent_guide`, `_hover`, `_active` | 트리 들여쓰기 안내선 | L-38, 값 T-04 |
| T-40.21 | `scrollbar.*` | 프로젝트 패널·편집기 스크롤바 | L-45, 값 T-04 |
| T-40.22 | `editor.background`, `editor.foreground` | 편집기·마크다운 미리보기 바탕과 글자 | L-102, 값 T-05 |
| T-40.23 | `editor.gutter.background`, `editor.line_number`, `editor.active_line_number`, `editor.active_line.background` | 줄 번호 칸, 현재 줄 | 값 T-05 |
| T-40.24 | `editor.wrap_guide`, `editor.indent_guide*`, `editor.invisible` | 세로 안내선, 보이지 않는 글자 | 값 T-05 |
| T-40.25 | `editor.document_highlight.*` | 커서 아래 같은 단어 강조 | 값 T-05 |
| T-40.26 | `players[0].*` | 캐럿 색, 선택 영역 | 값 T-05 |
| T-40.27 | `error`, `warning`, `success`, `info`, `conflict`, `hint` | 수정 표시 점(Accent·Warning), 알림 | L-65, 값 T-24 |
| T-40.28 | `syntax.*` | 코드 구문 강조, 마크다운 편집 강조 | L-109, 값 T-06 |

## 2. 기본 테마와 Catppuccin: 실제 값

### 2.1 Zed 기본 (T-07)

| ID | 항목 | 값 | 출처 |
|---| --- | --- | --- |
| T-07.1 | 기본 테마 설정 | `{ "mode": "system", "light": "One Light", "dark": "One Dark" }` | [default.json L9-L13](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L9-L13) |
| T-07.2 | 기본 아이콘 테마 | `"Zed (Default)"` (문자열) | [default.json L14](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L14) |
| T-07.3 | One 의 라이선스 | MIT, Copyright (c) 2014 GitHub Inc. (Atom One UI 기반) | [assets/themes/LICENSES](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/themes/LICENSES) |
| T-07.4 | 구분 | Zed 기본값(위)과 기준 화면 설정(아래)은 다르다. 우리 앱은 둘을 따로 적는다 | 04-APP-SPEC |

기준 화면 설정(사용자 `settings.json`)은 아래다.

```json
{
  "theme": { "mode": "system", "light": "One Light", "dark": "Catppuccin Mocha" },
  "icon_theme": "Catppuccin Latte",
  "ui_font_size": 16,
  "buffer_font_size": 15
}
```

### 2.2 Catppuccin 파일 구조 (T-19)

| ID | 항목 | 내용 |
|---| --- | --- |
| T-19.1 | 출처(고정) | 저장소 https://github.com/catppuccin/zed 의 태그 [v0.2.27](https://github.com/catppuccin/zed/tree/v0.2.27) = 커밋 `83fe57ba356a3b4dc4ac78b8d98d8ae95e093e45`. 설치본 `extension.toml` 도 version 0.2.27 이다 |
| T-19.2 | 라이선스 | MIT, Copyright (c) 2021 Catppuccin. 태그의 [LICENSE](https://github.com/catppuccin/zed/blob/v0.2.27/LICENSE) 로 확인했다 (설치 폴더에는 LICENSE 파일이 없다) |
| T-19.3 | 설치본 = 태그 | 설치본 `catppuccin-mauve.json` 은 태그 v0.2.27 의 같은 파일과 바이트가 같다(`cmp` 로 확인). 이 문서의 값 표는 태그의 JSON 과 같다 |
| T-19.4 | 파일 | [`themes/catppuccin-mauve.json`](https://github.com/catppuccin/zed/blob/v0.2.27/themes/catppuccin-mauve.json) (Latte `light`, Frappé·Macchiato·Mocha `dark`), [`themes/catppuccin-no-italics-mauve.json`](https://github.com/catppuccin/zed/blob/v0.2.27/themes/catppuccin-no-italics-mauve.json) (같은 4종, 이름 뒤에 ` - No Italics`) |
| T-19.5 | 우리가 쓰는 것 | `catppuccin-mauve.json` 의 4종. 이름은 `Catppuccin Latte`, `Catppuccin Frappé`, `Catppuccin Macchiato`, `Catppuccin Mocha` |
| T-19.6 | accent | `mauve` 변형. `text.accent`·`icon.accent`·`border.selected`·`editor.active_line_number`·`panel.indent_guide_hover` 가 Mauve 색이다 (값은 2.4절) |
| T-19.7 | no-italics | 스타일·syntax 색은 같고 기울임(`font_style: "italic"`)만 `null` 로 바뀐다. 우리는 채용하지 않는다 (판단) |
| T-19.8 | 토큰 집합 | One 과 같은 v0.2.0 스키마. 키가 더 많다(176/102). 우리 범위 밖 키는 쓰지 않는다 |

### 2.3 기준 화면과의 일치 확인 (T-08)

| ID | 영역 | 기준 화면 실측 (조사 자료) | Mocha JSON 키 | JSON 값 | 일치 |
|---| --- | --- | --- | --- | --- |
| T-08.1 | 타이틀 바·탭 바·상태 표시줄 | `#11111b` | `title_bar.background`, `tab_bar.background`, `status_bar.background` | `#11111b` | 예 |
| T-08.2 | 사이드바·프로젝트 패널 | `#181825` | `panel.background` | `#181825` | 예 |
| T-08.3 | 편집기·활성 탭 | `#1e1e2e` | `editor.background`, `tab.active_background` | `#1e1e2e` | 예 |
| T-08.4 | 비활성 탭 | `#0b0b11` | `tab.inactive_background` | `#0b0b11` | 예 |
| T-08.5 | 본문 글자 | `#cdd6f4` | `text` | `#cdd6f4` | 예 |
| T-08.6 | 사이드바 선택 항목 | `#3b3c4f` | `ghost_element.selected` `#6f728d66` 을 `panel.background` `#181825` 위에 알파 합성 | (0.4×111+0.6×24, 0.4×114+0.6×24, 0.4×141+0.6×37) = `#3b3c4f` | 예 |

정정: 조사 자료는 선택 항목을 `element.selected`(실제 `#3132444d`)로 매핑했다. 사이드바 선택 항목은 `ghost_element.selected` 로 그린다.

#### 두 번째 기준 화면 (Catppuccin Latte)

`assets/zed-reference-latte.png`(2448×1480 물리 px)에서 픽셀 색을 직접 읽었다. 배율은 가정(04 의 A-30.1)이라 좌표는 물리 px 로만 적는다. JSON 값은 태그 v0.2.27 의 Latte 다.

| ID | 영역 | 읽은 색 (좌표) | Latte JSON 키 | JSON 값 | 일치 |
| --- | --- | --- | --- | --- | --- |
| T-08.7 | 타이틀 바 | `#dce0e8` (1200, 10) | `title_bar.background` | `#dce0e8` | 예 |
| T-08.8 | 사이드바·프로젝트 패널·사이드바 푸터 | `#e6e9ef` (200, 1300), (600, 1250), (300, 1440) | `panel.background` | `#e6e9ef` | 예 |
| T-08.9 | 상태 표시줄 | `#dce0e8` (1200, 1450) | `status_bar.background` | `#dce0e8` | 예 |
| T-08.10 | 비활성 탭 | `#d2d7e2` (1100, 110) | `tab.inactive_background` | `#d2d7e2` | 예 |
| T-08.11 | 활성 탭, 편집기 | `#eff1f5` (1700, 110), (1500, 1000) | `tab.active_background`, `editor.background` | `#eff1f5` | 예 |
| T-08.12 | 선택된 사이드바 항목 | `#c3c7d2` (200, 190) | `ghost_element.selected` `#8f94a766` 을 `panel.background` 위에 합성 | (0.4×143+0.6×230, 0.4×148+0.6×233, 0.4×167+0.6×239) = `#c3c7d2` | 예 |
| T-08.13 | 영역 사이 선 | `#ccd0da` 2 물리 px: 타이틀 바 아래(y 64-65), 탭 바 아래(y 128-129), 상태 표시줄 위(y 1416-1417), 사이드바 오른쪽(x 400-401) | `border` | `#ccd0da` | 예 |
| T-08.14 | 비선택 트리 항목 글자 | `#5c5f77` | `text.muted` | `#5c5f77` | 예 |
| T-08.15 | 선택된 사이드바 항목 이름·타이틀 바 글자 | `#4c4f69` | `text` | `#4c4f69` | 예 |
| T-08.16 | 비활성(←→ 중 →) 아이콘, 비활성 Preview 탭 글자 | `#9ca0b0` | `icon.disabled`, `text.disabled` | `#9ca0b0` | 예 |
| T-08.17 | 켜진 프로젝트 패널 토글 아이콘 | `#8839ef` | `icon.accent` | `#8839ef` | 예. 배경은 상태 표시줄 색 그대로(`#dce0e8`)라 선택 배경은 없고 아이콘 색만 바뀐다 |
| T-08.18 | 사이드바 위 검색 글자(제외 대상) | `#acb0be` | `text.placeholder` | `#acb0be` | 예 (우리 앱에는 없다) |

창 모양 관찰(최대화하지 않은 창):

| ID | 관찰 | 값 | 판단 |
| --- | --- | --- | --- |
| T-08.19 | 창 바깥 테두리 | 왼쪽·위·아래에 약 2 물리 px 짜리 어두운 줄(`#303234` 부근). 테마 색이 아니다 | OS 가 그리는 창 테두리로 보인다 (추정). 앱이 그릴 요소가 아니다 |
| T-08.20 | 모서리 | 네 모서리가 둥글다. 대각선으로 어두운 구간이 약 5~6 물리 px 이어서 반지름이 약 19 물리 px 로 계산된다 | Windows 11 이 둥글게 한다 (추정). 앱이 모서리 반지름을 따로 그리지 않는다. 반지름 값은 실측이 아니라 계산이다 |
| T-08.21 | 창 버튼 | 최소화 `-`, 최대화 `□`, 닫기 `×`. 최대화 상태가 아니므로 □(U+E922)가 보이고 복원(U+E923)은 보이지 않는다. 글자색 `#4c4f69` | L-83 과 같다. 최대화하면 U+E923 으로 바뀐다 |
| T-08.22 | 상태 표시줄 커서 위치 | `1:1` | 형식은 `줄:열` (L-97). 미리보기 탭이 활성인데도 원문 편집기의 위치가 보인다. 이전 기준 화면은 `9:11` |
| T-08.23 | 상태 표시줄 커서 위치 글자색 | 이 화면에서는 `#4c4f69` = `text` | Mocha 화면에서 `text.muted` 로 읽은 검토 결과와 다르다. 확정하지 않았다 (T-18.7). 01 의 L-97 이 정한다 |

### 2.4 UI 토큰 값 (T-02 ~ T-05, T-24, T-25)

모든 값은 JSON 에서 읽었다. `T-02.21`(`drop_target.border`)은 6개 테마 모두에 없어 T-26.13 의 값을 쓴다. `—` 는 그 테마에 키가 없다는 뜻이고, `null` 은 키는 있으나 값이 null 이라는 뜻이다. 둘 다 1.2절(T-26)의 규칙으로 채우거나 "미사용" 이다. One 의 값은 [one.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/themes/one/one.json), Catppuccin 의 값은 [catppuccin-mauve.json](https://github.com/catppuccin/zed/blob/v0.2.27/themes/catppuccin-mauve.json) (태그 v0.2.27) 이다.

#### T-02 표면·테두리·요소

| ID | 토큰 | One Dark | One Light | Latte | Frappé | Macchiato | Mocha |
|---|---|---|---|---|---|---|---|
| T-02.01 | `background` | `#3b414dff` | `#dcdcddff` | `#d7dce5` | `#383c52` | `#2c2f46` | `#27273b` |
| T-02.02 | `surface.background` | `#2f343eff` | `#ebebecff` | `#e6e9ef` | `#292c3c` | `#1e2030` | `#181825` |
| T-02.03 | `elevated_surface.background` | `#2f343eff` | `#ebebecff` | `#e6e9ef` | `#292c3c` | `#1e2030` | `#181825` |
| T-02.04 | `border` | `#464b57ff` | `#c9c9caff` | `#ccd0da` | `#414559` | `#363a4f` | `#313244` |
| T-02.05 | `border.variant` | `#363c46ff` | `#dfdfe0ff` | `#e7e2f5` | `#3c3c53` | `#313148` | `#2c293d` |
| T-02.06 | `border.focused` | `#47679eff` | `#7d82e8ff` | `#7287fd` | `#babbf1` | `#b7bdf8` | `#b4befe` |
| T-02.07 | `border.selected` | `#293b5bff` | `#cbcdf6ff` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-02.08 | `border.transparent` | `#00000000` | `#00000000` | `#00000000` | `#00000000` | `#00000000` | `#00000000` |
| T-02.09 | `border.disabled` | `#414754ff` | `#d3d3d4ff` | `#9ca0b0` | `#737994` | `#6e738d` | `#6c7086` |
| T-02.10 | `element.background` | `#2e343eff` | `#ebebecff` | `#dce0e8` | `#232634` | `#181926` | `#11111b` |
| T-02.11 | `element.hover` | `#363c46ff` | `#dfdfe0ff` | `#ccd0da` | `#414559` | `#363a4f` | `#313244` |
| T-02.12 | `element.active` | `#454a56ff` | `#cacacaff` | `#acb0be4d` | `#6268804d` | `#5b60784d` | `#585b704d` |
| T-02.13 | `element.selected` | `#454a56ff` | `#cacacaff` | `#ccd0da4d` | `#4145594d` | `#363a4f4d` | `#3132444d` |
| T-02.14 | `element.disabled` | `#2e343eff` | `#ebebecff` | `#9ca0b0` | `#737994` | `#6e738d` | `#6c7086` |
| T-02.15 | `ghost_element.background` | `#00000000` | `#00000000` | `#00000000` | `#00000000` | `#00000000` | `#00000000` |
| T-02.16 | `ghost_element.hover` | `#363c46ff` | `#dfdfe0ff` | `#bcc0cc4d` | `#51576d4d` | `#494d644d` | `#45475a4d` |
| T-02.17 | `ghost_element.active` | `#454a56ff` | `#cacacaff` | `#acb0be99` | `#62688099` | `#5b607899` | `#585b7099` |
| T-02.18 | `ghost_element.selected` | `#454a56ff` | `#cacacaff` | `#8f94a766` | `#7c829a66` | `#73799566` | `#6f728d66` |
| T-02.19 | `ghost_element.disabled` | `#2e343eff` | `#ebebecff` | `#9ca0b0` | `#737994` | `#6e738d` | `#6c7086` |
| T-02.20 | `drop_target.background` | `#83899480` | `#7e808780` | `#ccd0da66` | `#41455966` | `#363a4f66` | `#31324466` |
| T-02.21 | `drop_target.border` | — | — | — | — | — | — |

#### T-03 글자·아이콘

| ID | 토큰 | One Dark | One Light | Latte | Frappé | Macchiato | Mocha |
|---|---|---|---|---|---|---|---|
| T-03.01 | `text` | `#dce0e5ff` | `#242529ff` | `#4c4f69` | `#c6d0f5` | `#cad3f5` | `#cdd6f4` |
| T-03.02 | `text.muted` | `#a9afbcff` | `#58585aff` | `#5c5f77` | `#b5bfe2` | `#b8c0e0` | `#bac2de` |
| T-03.03 | `text.placeholder` | `#878a98ff` | `#7e8086ff` | `#acb0be` | `#626880` | `#5b6078` | `#585b70` |
| T-03.04 | `text.disabled` | `#878a98ff` | `#7e8086ff` | `#9ca0b0` | `#737994` | `#6e738d` | `#6c7086` |
| T-03.05 | `text.accent` | `#74ade8ff` | `#5c78e2ff` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-03.06 | `icon` | `#dce0e5ff` | `#242529ff` | `#4c4f69` | `#c6d0f5` | `#cad3f5` | `#cdd6f4` |
| T-03.07 | `icon.muted` | `#a9afbcff` | `#58585aff` | `#8c8fa1` | `#838ba7` | `#8087a2` | `#7f849c` |
| T-03.08 | `icon.disabled` | `#878a98ff` | `#7e8086ff` | `#9ca0b0` | `#737994` | `#6e738d` | `#6c7086` |
| T-03.09 | `icon.placeholder` | `#a9afbcff` | `#58585aff` | `#acb0be` | `#626880` | `#5b6078` | `#585b70` |
| T-03.10 | `icon.accent` | `#74ade8ff` | `#5c78e2ff` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-03.11 | `link_text.hover` | `#74ade8ff` | `#5c78e2ff` | `#04a5e5` | `#99d1db` | `#91d7e3` | `#89dceb` |

#### T-04 영역별

| ID | 토큰 | One Dark | One Light | Latte | Frappé | Macchiato | Mocha |
|---|---|---|---|---|---|---|---|
| T-04.01 | `title_bar.background` | `#3b414dff` | `#dcdcddff` | `#dce0e8` | `#232634` | `#181926` | `#11111b` |
| T-04.02 | `title_bar.inactive_background` | `#2e343eff` | `#ebebecff` | `#e6e9ee` | `#2a2e3e` | `#1e1f30` | `#171725` |
| T-04.03 | `status_bar.background` | `#3b414dff` | `#dcdcddff` | `#dce0e8` | `#232634` | `#181926` | `#11111b` |
| T-04.04 | `toolbar.background` | `#282c33ff` | `#fafafaff` | `#eff1f5` | `#303446` | `#24273a` | `#1e1e2e` |
| T-04.05 | `tab_bar.background` | `#2f343eff` | `#ebebecff` | `#dce0e8` | `#232634` | `#181926` | `#11111b` |
| T-04.06 | `tab.inactive_background` | `#2f343eff` | `#ebebecff` | `#d2d7e2` | `#1d202b` | `#12121c` | `#0b0b11` |
| T-04.07 | `tab.active_background` | `#282c33ff` | `#fafafaff` | `#eff1f5` | `#303446` | `#24273a` | `#1e1e2e` |
| T-04.08 | `panel.background` | `#2f343eff` | `#ebebecff` | `#e6e9ef` | `#292c3c` | `#1e2030` | `#181825` |
| T-04.09 | `panel.focused_border` | `null` | `null` | `#4c4f69` | `#c6d0f5` | `#cad3f5` | `#cdd6f4` |
| T-04.10 | `panel.indent_guide` | — | — | `#ccd0da99` | `#41455999` | `#363a4f99` | `#31324499` |
| T-04.11 | `panel.indent_guide_hover` | — | — | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-04.12 | `panel.indent_guide_active` | — | — | `#acb0be` | `#626880` | `#5b6078` | `#585b70` |
| T-04.13 | `panel.overlay_background` | — | — | `#e6e9ef` | `#292c3c` | `#1e2030` | `#181825` |
| T-04.14 | `pane.focused_border` | `null` | `null` | `#4c4f69` | `#c6d0f5` | `#cad3f5` | `#cdd6f4` |
| T-04.15 | `pane_group.border` | — | — | `#ccd0da` | `#414559` | `#363a4f` | `#313244` |
| T-04.16 | `scrollbar.thumb.background` | `#c8ccd44c` | `#383a414c` | `#acb0be80` | `#62688080` | `#5b607880` | `#585b7080` |
| T-04.17 | `scrollbar.thumb.hover_background` | `#363c46ff` | `#dfdfe0ff` | `#9ca0b0` | `#737994` | `#6e738d` | `#6c7086` |
| T-04.18 | `scrollbar.thumb.active_background` | — | — | `null` | `null` | `null` | `null` |
| T-04.19 | `scrollbar.thumb.border` | `#363c46ff` | `#dfdfe0ff` | `null` | `null` | `null` | `null` |
| T-04.20 | `scrollbar.track.background` | `#00000000` | `#00000000` | `#dce0e8` | `#232634` | `#181926` | `#11111b` |
| T-04.21 | `scrollbar.track.border` | `#2e333cff` | `#eeeeeeff` | `#4c4f6912` | `#c6d0f512` | `#cad3f512` | `#cdd6f412` |

#### T-05 편집기

| ID | 토큰 | One Dark | One Light | Latte | Frappé | Macchiato | Mocha |
|---|---|---|---|---|---|---|---|
| T-05.01 | `editor.foreground` | `#acb2beff` | `#242529ff` | `#4c4f69` | `#c6d0f5` | `#cad3f5` | `#cdd6f4` |
| T-05.02 | `editor.background` | `#282c33ff` | `#fafafaff` | `#eff1f5` | `#303446` | `#24273a` | `#1e1e2e` |
| T-05.03 | `editor.gutter.background` | `#282c33ff` | `#fafafaff` | `#eff1f5` | `#303446` | `#24273a` | `#1e1e2e` |
| T-05.04 | `editor.subheader.background` | `#2f343eff` | `#ebebecff` | `#e6e9ef` | `#292c3c` | `#1e2030` | `#181825` |
| T-05.05 | `editor.active_line.background` | `#2f343ebf` | `#ebebecbf` | `#4c4f6912` | `#c6d0f512` | `#cad3f512` | `#cdd6f412` |
| T-05.06 | `editor.highlighted_line.background` | `#2f343eff` | `#ebebecff` | `null` | `null` | `null` | `null` |
| T-05.07 | `editor.line_number` | `#4e5a5f` | `#b4b4bb` | `#8c8fa1` | `#838ba7` | `#8087a2` | `#7f849c` |
| T-05.08 | `editor.active_line_number` | `#d0d4da` | `#44454b` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-05.09 | `editor.hover_line_number` | `#acb0b4` | `#61616b` | — | — | — | — |
| T-05.10 | `editor.invisible` | `#4e5a5f` | `#b4b4bb` | `#7c7f9366` | `#949cbb66` | `#939ab766` | `#9399b266` |
| T-05.11 | `editor.wrap_guide` | `#c8ccd40d` | `#383a410d` | `#acb0be` | `#626880` | `#5b6078` | `#585b70` |
| T-05.12 | `editor.active_wrap_guide` | `#c8ccd41a` | `#383a411a` | `#acb0be` | `#626880` | `#5b6078` | `#585b70` |
| T-05.13 | `editor.document_highlight.read_background` | `#74ade81a` | `#5c78e225` | `#6c6f8529` | `#a5adce29` | `#a5adcb29` | `#a6adc829` |
| T-05.14 | `editor.document_highlight.write_background` | `#555a6366` | `#a3a3a466` | `#6c6f8529` | `#a5adce29` | `#a5adcb29` | `#a6adc829` |
| T-05.15 | `editor.document_highlight.bracket_background` | — | — | `#8839ef17` | `#ca9ee617` | `#c6a0f617` | `#cba6f717` |
| T-05.16 | `editor.indent_guide` | — | — | `#ccd0da99` | `#41455999` | `#363a4f99` | `#31324499` |
| T-05.17 | `editor.indent_guide_active` | — | — | `#acb0be` | `#626880` | `#5b6078` | `#585b70` |

#### T-25 플레이어·accents

| ID | 항목 | One Dark | One Light | Latte | Frappé | Macchiato | Mocha |
|---|---|---|---|---|---|---|---|
| T-25.01 | `players[0].cursor` | `#74ade8ff` | `#5c78e2ff` | `#dc8a78` | `#f2d5cf` | `#f4dbd6` | `#f5e0dc` |
| T-25.02 | `players[0].selection` | `#74ade83d` | `#5c78e23d` | `#7c7f934d` | `#949cbb40` | `#939ab740` | `#9399b240` |
| T-25.03 | `players[0].background` | `#74ade8ff` | `#5c78e2ff` | `#dc8a78` | `#f2d5cf` | `#f4dbd6` | `#f5e0dc` |
| T-25.04 | `players` 개수 | 8 | 8 | 8 | 8 | 8 | 8 |
| T-25.05 | `accents` | — | — | `#8839ef` `#7287fd` `#209fb5` `#40a02b` `#df8e1d` `#fe640b` `#d20f39` | `#ca9ee6` `#babbf1` `#85c1dc` `#a6d189` `#e5c890` `#ef9f76` `#e78284` | `#c6a0f6` `#b7bdf8` `#7dc4e4` `#a6da95` `#eed49f` `#f5a97f` `#ed8796` | `#cba6f7` `#b4befe` `#74c7ec` `#a6e3a1` `#f9e2af` `#fab387` `#f38ba8` |

`players[0]` 이 내 캐럿과 선택 영역이다. `accents` 는 One 에 없고 Catppuccin 에만 7색이 있다.

#### T-24 상태 색

| ID | 토큰 | One Dark | One Light | Latte | Frappé | Macchiato | Mocha |
|---|---|---|---|---|---|---|---|
| T-24.01 | `error` | `#d07277ff` | `#d36151ff` | `#d20f39` | `#e78284` | `#ed8796` | `#f38ba8` |
| T-24.02 | `error.background` | `#d072771a` | `#fbdfd9ff` | `#d20f391f` | `#e782841f` | `#ed87961f` | `#f38ba81f` |
| T-24.03 | `error.border` | `#4c2b2cff` | `#f6c6bdff` | `#d20f39` | `#e78284` | `#ed8796` | `#f38ba8` |
| T-24.04 | `warning` | `#dec184ff` | `#a48819ff` | `#df8e1d` | `#e5c890` | `#eed49f` | `#f9e2af` |
| T-24.05 | `warning.background` | `#dec1841a` | `#faf2e6ff` | `#df8e1d1f` | `#e5c8901f` | `#eed49f1f` | `#f9e2af1f` |
| T-24.06 | `warning.border` | `#5d4c2fff` | `#f4e7d1ff` | `#df8e1d` | `#e5c890` | `#eed49f` | `#f9e2af` |
| T-24.07 | `info` | `#74ade8ff` | `#5c78e2ff` | `#179299` | `#81c8be` | `#8bd5ca` | `#94e2d5` |
| T-24.08 | `info.background` | `#74ade81a` | `#e2e2faff` | `#17929933` | `#81c8be33` | `#8bd5ca33` | `#94e2d533` |
| T-24.09 | `info.border` | `#293b5bff` | `#cbcdf6ff` | `#179299` | `#81c8be` | `#8bd5ca` | `#94e2d5` |
| T-24.10 | `success` | `#a1c181ff` | `#669f59ff` | `#40a02b` | `#a6d189` | `#a6da95` | `#a6e3a1` |
| T-24.11 | `success.background` | `#a1c1811a` | `#dfeadbff` | `#40a02b1f` | `#a6d1891f` | `#a6da951f` | `#a6e3a11f` |
| T-24.12 | `success.border` | `#38482fff` | `#c8dcc1ff` | `#40a02b` | `#a6d189` | `#a6da95` | `#a6e3a1` |
| T-24.13 | `conflict` | `#dec184ff` | `#a48819ff` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-24.14 | `conflict.background` | `#dec1841a` | `#faf2e6ff` | `#fe640b26` | `#ef9f7626` | `#f5a97f26` | `#fab38726` |
| T-24.15 | `conflict.border` | `#5d4c2fff` | `#f4e7d1ff` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-24.16 | `hint` | `#788ca6ff` | `#7274a7ff` | `#acb0be` | `#626880` | `#5b6078` | `#585b70` |
| T-24.17 | `hint.background` | `#5a6f891a` | `#e2e2faff` | `#e6e9ef` | `#292c3c` | `#1e2030` | `#181825` |
| T-24.18 | `hint.border` | `#293b5bff` | `#cbcdf6ff` | `#acb0be` | `#626880` | `#5b6078` | `#585b70` |

### 2.5 구문 강조 값 (T-06)

`syntax` 의 각 항목은 `{ "color", "font_style", "font_weight" }` 다. 표에서 `i` 는 `font_style: "italic"`, `w700` 은 `font_weight: 700` 이다.

#### One 이 정의한 46개 키 (조사 자료 T-06 의 One 값과 Catppuccin 매핑을 대체한다)

정정: 조사 자료 T-06 의 Catppuccin 열은 표준 Catppuccin 팔레트 추정이었다. 아래는 설치된 JSON 값이다.

| ID | 키 | One Dark | One Light | Latte | Frappé | Macchiato | Mocha |
|---|---|---|---|---|---|---|---|
| T-06.01 | `attribute` | `#74ade8ff` | `#5c78e2ff` | `#df8e1d` | `#e5c890` | `#eed49f` | `#f9e2af` |
| T-06.02 | `boolean` | `#bf956aff` | `#ad6e25ff` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-06.03 | `comment` | `#5d636fff` | `#a2a3a7ff` | `#7c7f93` i | `#949cbb` i | `#939ab7` i | `#9399b2` i |
| T-06.04 | `comment.doc` | `#878e98ff` | `#7c7e86ff` | `#7c7f93` i | `#949cbb` i | `#939ab7` i | `#9399b2` i |
| T-06.05 | `constant` | `#dfc184ff` | `#c18401ff` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-06.06 | `constructor` | `#73ade9ff` | `#5c78e2ff` | `#dd7878` | `#eebebe` | `#f0c6c6` | `#f2cdcd` |
| T-06.07 | `embedded` | `#dce0e5ff` | `#242529ff` | `#e64553` | `#ea999c` | `#ee99a0` | `#eba0ac` |
| T-06.08 | `emphasis` | `#74ade8ff` | `#5c78e2ff` | `#e64553` i | `#ea999c` i | `#ee99a0` i | `#eba0ac` i |
| T-06.09 | `emphasis.strong` | `#bf956aff` w700 | `#ad6e25ff` w700 | `#e64553` w700 | `#ea999c` w700 | `#ee99a0` w700 | `#eba0ac` w700 |
| T-06.10 | `enum` | `#6eb4bfff` | `#3882b7ff` | `#179299` w700 | `#81c8be` w700 | `#8bd5ca` w700 | `#94e2d5` w700 |
| T-06.11 | `function` | `#73ade9ff` | `#5b79e3ff` | `#1e66f5` | `#8caaee` | `#8aadf4` | `#89b4fa` |
| T-06.12 | `hint` | `#788ca6ff` | `#7274a7ff` | `#8c8fa1` i | `#838ba7` i | `#8087a2` i | `#7f849c` i |
| T-06.13 | `keyword` | `#b477cfff` | `#a449abff` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.14 | `label` | `#74ade8ff` | `#5c78e2ff` | `#209fb5` | `#85c1dc` | `#7dc4e4` | `#74c7ec` |
| T-06.15 | `link_text` | `#73ade9ff` i | `#5b79e3ff` i | `#7287fd` | `#babbf1` | `#b7bdf8` | `#b4befe` |
| T-06.16 | `link_uri` | `#6eb4bfff` | `#3882b7ff` | `#1e66f5` i | `#8caaee` i | `#8aadf4` i | `#89b4fa` i |
| T-06.17 | `namespace` | `#dce0e5ff` | `#242529ff` | `#df8e1d` i | `#e5c890` i | `#eed49f` i | `#f9e2af` i |
| T-06.18 | `number` | `#bf956aff` | `#ad6e25ff` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-06.19 | `operator` | `#6eb4bfff` | `#3882b7ff` | `#04a5e5` | `#99d1db` | `#91d7e3` | `#89dceb` |
| T-06.20 | `predictive` | `#5a6a87ff` i | `#9b9ec6ff` i | `#9ca0b0` | `#737994` | `#6e738d` | `#6c7086` |
| T-06.21 | `preproc` | `#b477cfff` | `#a449abff` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.22 | `primary` | `#acb2beff` | `#242529ff` | `#e64553` | `#ea999c` | `#ee99a0` | `#eba0ac` |
| T-06.23 | `property` | `#d07277ff` | `#d3604fff` | `#7287fd` | `#babbf1` | `#b7bdf8` | `#b4befe` |
| T-06.24 | `punctuation` | `#acb2beff` | `#242529ff` | `#7c7f93` | `#949cbb` | `#939ab7` | `#9399b2` |
| T-06.25 | `punctuation.bracket` | `#b2b9c6ff` | `#4d4f52ff` | `#7c7f93` | `#949cbb` | `#939ab7` | `#9399b2` |
| T-06.26 | `punctuation.delimiter` | `#b2b9c6ff` | `#4d4f52ff` | `#7c7f93` | `#949cbb` | `#939ab7` | `#9399b2` |
| T-06.27 | `punctuation.list_marker` | `#d07277ff` | `#d3604fff` | `#179299` | `#81c8be` | `#8bd5ca` | `#94e2d5` |
| T-06.28 | `punctuation.markup` | `#d07277ff` | `#d3604fff` | — | — | — | — |
| T-06.29 | `punctuation.special` | `#b1574bff` | `#b92b46ff` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.30 | `selector` | `#dfc184ff` | `#669f59ff` | — | — | — | — |
| T-06.31 | `selector.pseudo` | `#74ade8ff` | `#5c78e2ff` | `#df8e1d` | `#e5c890` | `#eed49f` | `#f9e2af` |
| T-06.32 | `string` | `#a1c181ff` | `#649f57ff` | `#40a02b` | `#a6d189` | `#a6da95` | `#a6e3a1` |
| T-06.33 | `string.escape` | `#878e98ff` | `#7c7e86ff` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.34 | `string.regex` | `#bf956aff` | `#ad6e26ff` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.35 | `string.special` | `#bf956aff` | `#ad6e26ff` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.36 | `string.special.symbol` | `#bf956aff` | `#ad6e26ff` | `#dd7878` | `#eebebe` | `#f0c6c6` | `#f2cdcd` |
| T-06.37 | `tag` | `#74ade8ff` | `#5c78e2ff` | `#1e66f5` | `#8caaee` | `#8aadf4` | `#89b4fa` |
| T-06.38 | `text.literal` | `#a1c181ff` | `#649f57ff` | `#40a02b` | `#a6d189` | `#a6da95` | `#a6e3a1` |
| T-06.39 | `title` | `#d07277ff` w400 | `#d3604fff` w400 | `#d20f39` w800 | `#e78284` w800 | `#ed8796` w800 | `#f38ba8` w800 |
| T-06.40 | `type` | `#6eb4bfff` | `#3882b7ff` | `#df8e1d` | `#e5c890` | `#eed49f` | `#f9e2af` |
| T-06.41 | `variable` | `#acb2beff` | `#242529ff` | `#4c4f69` | `#c6d0f5` | `#cad3f5` | `#cdd6f4` |
| T-06.42 | `variable.parameter` | `#d07277ff` | `#d3604fff` | `#e64553` | `#ea999c` | `#ee99a0` | `#eba0ac` |
| T-06.43 | `variable.special` | `#bf956aff` | `#ad6e25ff` | `#d20f39` i | `#e78284` i | `#ed8796` i | `#f38ba8` i |
| T-06.44 | `variant` | `#73ade9ff` | `#5b79e3ff` | `#d20f39` | `#e78284` | `#ed8796` | `#f38ba8` |
| T-06.45 | `diff.plus` | `#98c379ff` | `#50a14fff` | `#40a02b` | `#a6d189` | `#a6da95` | `#a6e3a1` |
| T-06.46 | `diff.minus` | `#e06c75ff` | `#e45649ff` | `#d20f39` | `#e78284` | `#ed8796` | `#f38ba8` |

#### Catppuccin 에만 있는 58개 키

| ID | 키 | Latte | Frappé | Macchiato | Mocha |
|---|---|---|---|---|---|
| T-06.47 | `variable.builtin` | `#d20f39` | `#e78284` | `#ed8796` | `#f38ba8` |
| T-06.48 | `variable.member` | `#1e66f5` | `#8caaee` | `#8aadf4` | `#89b4fa` |
| T-06.49 | `constant.builtin` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-06.50 | `constant.macro` | `#dc8a78` | `#f2d5cf` | `#f4dbd6` | `#f5e0dc` |
| T-06.51 | `module` | `#df8e1d` i | `#e5c890` i | `#eed49f` i | `#f9e2af` i |
| T-06.52 | `string.documentation` | `#179299` | `#81c8be` | `#8bd5ca` | `#94e2d5` |
| T-06.53 | `string.regexp` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.54 | `string.special.path` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.55 | `string.special.url` | `#dc8a78` i | `#f2d5cf` i | `#f4dbd6` i | `#f5e0dc` i |
| T-06.56 | `character` | `#179299` | `#81c8be` | `#8bd5ca` | `#94e2d5` |
| T-06.57 | `character.special` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.58 | `number.float` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-06.59 | `tag.attribute` | `#df8e1d` i | `#e5c890` i | `#eed49f` i | `#f9e2af` i |
| T-06.60 | `tag.delimiter` | `#179299` | `#81c8be` | `#8bd5ca` | `#94e2d5` |
| T-06.61 | `type.builtin` | `#8839ef` i | `#ca9ee6` i | `#c6a0f6` i | `#cba6f7` i |
| T-06.62 | `type.definition` | `#df8e1d` | `#e5c890` | `#eed49f` | `#f9e2af` |
| T-06.63 | `type.interface` | `#df8e1d` i | `#e5c890` i | `#eed49f` i | `#f9e2af` i |
| T-06.64 | `type.super` | `#df8e1d` i | `#e5c890` i | `#eed49f` i | `#f9e2af` i |
| T-06.65 | `function.builtin` | `#d20f39` | `#e78284` | `#ed8796` | `#f38ba8` |
| T-06.66 | `function.call` | `#1e66f5` | `#8caaee` | `#8aadf4` | `#89b4fa` |
| T-06.67 | `function.macro` | `#dc8a78` | `#f2d5cf` | `#f4dbd6` | `#f5e0dc` |
| T-06.68 | `function.method` | `#1e66f5` | `#8caaee` | `#8aadf4` | `#89b4fa` |
| T-06.69 | `function.method.call` | `#1e66f5` | `#8caaee` | `#8aadf4` | `#89b4fa` |
| T-06.70 | `keyword.modifier` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.71 | `keyword.type` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.72 | `keyword.coroutine` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.73 | `keyword.function` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.74 | `keyword.operator` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.75 | `keyword.import` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.76 | `keyword.repeat` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.77 | `keyword.return` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.78 | `keyword.debug` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.79 | `keyword.exception` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.80 | `keyword.conditional` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.81 | `keyword.conditional.ternary` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.82 | `keyword.directive` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.83 | `keyword.directive.define` | `#ea76cb` | `#f4b8e4` | `#f5bde6` | `#f5c2e7` |
| T-06.84 | `keyword.export` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.85 | `punctuation.special.symbol` | `#dd7878` | `#eebebe` | `#f0c6c6` | `#f2cdcd` |
| T-06.86 | `comment.documentation` | `#7c7f93` i | `#949cbb` i | `#939ab7` i | `#9399b2` i |
| T-06.87 | `comment.info` | `#179299` i | `#81c8be` i | `#8bd5ca` i | `#94e2d5` i |
| T-06.88 | `comment.error` | `#d20f39` i | `#e78284` i | `#ed8796` i | `#f38ba8` i |
| T-06.89 | `comment.warning` | `#df8e1d` i | `#e5c890` i | `#eed49f` i | `#f9e2af` i |
| T-06.90 | `comment.warn` | `#df8e1d` i | `#e5c890` i | `#eed49f` i | `#f9e2af` i |
| T-06.91 | `comment.hint` | `#1e66f5` i | `#8caaee` i | `#8aadf4` i | `#89b4fa` i |
| T-06.92 | `comment.todo` | `#dd7878` i | `#eebebe` i | `#f0c6c6` i | `#f2cdcd` i |
| T-06.93 | `comment.note` | `#dc8a78` i | `#f2d5cf` i | `#f4dbd6` i | `#f5e0dc` i |
| T-06.94 | `parameter` | `#e64553` | `#ea999c` | `#ee99a0` | `#eba0ac` |
| T-06.95 | `field` | `#1e66f5` | `#8caaee` | `#8aadf4` | `#89b4fa` |
| T-06.96 | `float` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-06.97 | `symbol` | `#dd7878` | `#eebebe` | `#f0c6c6` | `#f2cdcd` |
| T-06.98 | `concept` | `#209fb5` | `#85c1dc` | `#7dc4e4` | `#74c7ec` |
| T-06.99 | `function.decorator` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-06.100 | `type.class.definition` | `#df8e1d` w700 | `#e5c890` w700 | `#eed49f` w700 | `#f9e2af` w700 |
| T-06.101 | `parent` | `#fe640b` | `#ef9f76` | `#f5a97f` | `#fab387` |
| T-06.102 | `predoc` | `#d20f39` | `#e78284` | `#ed8796` | `#f38ba8` |
| T-06.103 | `tag.doctype` | `#8839ef` | `#ca9ee6` | `#c6a0f6` | `#cba6f7` |
| T-06.104 | `string.doc` | `#179299` i | `#81c8be` i | `#8bd5ca` i | `#94e2d5` i |

| ID | 규칙 | 내용 |
| --- | --- | --- |
| T-27.1 | One 에만 있는 키 | punctuation.markup, selector 는 Catppuccin 에 없다. **탐색 규칙(확정):** 코드의 구문 이름(capture)에 대해, 테마에 등록된 키 가운데 **점 경계에서 앞부분이 일치하는 가장 긴 키**를 고른다. 예: `keyword.modifier` 가 없으면 `keyword`. 일치하는 키가 없으면 스타일을 입히지 않아 기본 글자색(`editor.foreground`)으로 나온다 ([syntax_theme.rs L87-L101](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/syntax_theme/src/syntax_theme.rs#L87-L101)). `punctuation.markup` 처럼 이름이 `punctuation` 과 점 경계로 이어지면 `punctuation` 을 고른다 |
| T-27.2 | 마크다운 편집 강조 | 제목 `title`, 굵게 `emphasis.strong`, 기울임 `emphasis`, 링크 `link_text`·`link_uri`, 인라인 코드 `text.literal`, 목록 기호 `punctuation.list_marker` |

## 3. UI 아이콘

### 3.1 출처와 규격 (T-09)

| ID | 항목 | 내용 | 출처 |
|---| --- | --- | --- |
| T-09.1 | 원본 | `assets/icons/*.svg`. `IconName` 열거형(`crates/icons/src/icons.rs`)의 이름과 snake_case 파일명이 1:1 | [icons README](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/icons/README.md) |
| T-09.2 | 규격 | 16×16 viewBox, 선 두께 1.2, 대부분 안쪽 12×12 상자 안 | 같은 문서 |
| T-09.3 | 출처 | 대부분 Lucide 에서 가져와 고쳤고, 일부는 Phosphor 나 직접 그렸다 | 같은 문서 |
| T-09.4 | 라이선스 | `assets/icons/LICENSES` 에는 Lucide(ISC)만 적혀 있다. Zed 가 직접 그린 SVG 는 따로 확인되지 않았다 | [LICENSES](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/icons/LICENSES) |
| T-09.5 | 우리 방침 | Zed 의 SVG 를 복사하지 않는다. 비슷한 의미의 Lucide 아이콘을 **대체 후보**로 쓴다 (판단). 모양이 Zed 와 같은지는 아직 비교하지 않았다 (T-18.3) | README 라이선스 원칙 |
| T-09.6 | 크기 | 탭 바·상태 표시줄 버튼 14, 트리 아이콘 16, 타이틀 바 메뉴 14 | L-06, L-40 |

### 3.2 우리 화면의 아이콘 (T-10)

Zed 파일은 커밋 14b7b0e 의 `assets/icons/` 에서 존재를 확인했다. gpui-kit 열은 `gpui_kit_assets::IconName` 의 변형 이름이다. docs.rs 의 [gpui-kit-assets 0.8.1](https://docs.rs/gpui-kit-assets/0.8.1/gpui_kit_assets/enum.IconName.html) 목록에 있음을 확인했다(1830개 변형). `gpui_component::IconName` 은 106개뿐이라 이 목록으로 쓴다. 툴팁은 Zed 가 띄우는 문구다 (T-30).

| ID | 화면 | 용도 | Zed `IconName` → 파일 | 대체 후보 (Lucide) | gpui-kit | 툴팁 (Zed) | 참조 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T-10.1 | 타이틀 바 | ☰ 앱 메뉴 | `Menu` → `menu.svg` | `menu` | `Menu` | Open Application Menu | L-85 |
| T-10.2 | 타이틀 바 | 창 최소화 | Windows 는 글꼴 글리프 U+E921 (`Segoe Fluent Icons`) | **사용 안 함.** 01 의 L-83 대로 글꼴 글리프(`Segoe Fluent Icons`, 없으면 `Segoe MDL2 Assets`)를 그린다 | — | (없음, L-228) | L-83 |
| T-10.3 | 타이틀 바 | 창 최대화 / 복원 | 글리프 U+E922 / U+E923 | **사용 안 함.** 글리프 U+E922(최대화) / U+E923(복원) | — | (없음, L-228) | L-83 |
| T-10.4 | 타이틀 바 | 창 닫기 | 글리프 U+E8BB | **사용 안 함.** 글리프 U+E8BB | — | (없음, L-228) | L-83 |
| T-10.5 | 사이드바 | 폴더 추가 | `FolderAdd` → `folder_add.svg` | `folder-plus` | `FolderPlus` | Add Project | L-96 |
| T-10.6 | 상태 표시줄 | 사이드바 토글 (닫힘) | `ThreadsSidebarLeftClosed` → `threads_sidebar_left_closed.svg` | `panel-left` | `PanelLeft` | Open Threads Sidebar (L-251). 우리 앱은 "Open Sidebar" 로 바꾼다 (판단) | L-95 |
| T-10.7 | 사이드바 | 사이드바 토글 (열림) | `ThreadsSidebarLeftOpen` → `threads_sidebar_left_open.svg` | `panel-left` + 왼쪽 면 채움 | `PanelLeft` (채움은 직접 칠함) | 두 줄: Toggle Sidebar + 키 / Focus Sidebar + 키 (L-232) | L-95 |
| T-10.8 | 상태 표시줄 | 프로젝트 패널 토글 | `FileTree` → `file_tree.svg` | `folder-tree` | `FolderTree` | Project Panel (L-253). 패널이 열려 있고 활성이면 Close Left Dock (L-254) | L-93 |
| T-10.9 | 탭 바 | 뒤로 | `ArrowLeft` → `arrow_left.svg` | `arrow-left` | `ArrowLeft` | Go Back (L-236) | L-62 |
| T-10.10 | 탭 바 | 앞으로 | `ArrowRight` → `arrow_right.svg` | `arrow-right` | `ArrowRight` | Go Forward (L-237) | L-62 |
| T-10.11 | 탭 바 | 탭 닫기 | `Close` → `close.svg` | `x` | `X` | Close Tab (L-240). 활성 탭에만 키가 붙는다 | L-66 |
| T-10.12 | 탭 바 | 마크다운 미리보기 탭 | `FileDoc` → `file_doc.svg` (책 모양) | `book-open` | `BookOpen` | — | L-69 |
| T-10.13 | 탭 바 | 새 탭 | `Plus` → `plus.svg` | `plus` | `Plus` | New… (L-243) | L-71 |
| T-10.14 | 탭 바 | 분할 | `Split` → `split.svg` | `columns-2` | `Columns2` | Split Pane (L-244) | L-72 |
| T-10.15 | 탭 바 | 최대화 | `Maximize` → `maximize.svg` | `maximize-2` | `Maximize2` | Zoom In (L-245) | L-73 |
| T-10.16 | 탭 바 | 최대화 해제 | `Minimize` → `minimize.svg` | `minimize-2` | `Minimize2` | Zoom Out (L-245) | L-73 |
| T-10.17 | 프로젝트 패널 | 트리 펼침 화살표 (`folder_indicator: "chevron"`·`"both"` 일 때만) | `chevron_right.svg`, `chevron_down.svg` | `chevron-right`, `chevron-down` | `ChevronRight`, `ChevronDown` | — | L-44 |
| T-10.18 | 탭 | 고정(pinned) 탭 표시·해제 버튼 | `Pin` → `pin.svg` | `pin` | `Pin` | Unpin Tab (활성 탭이면 키 표시, L-241) | L-66, I-065 |
| T-10.19 | 탭 | 읽기 전용 표시 | `FileLock` → `file_lock.svg` | — | — | **제외.** 읽기 전용 탭(Make Tab Read-Only / Unlock Tab) 기능은 사용자 결정(_decisions.md "확장 세부")으로 넣지 않는다. 아이콘·툴팁(L-242)도 만들지 않는다 | L-269 |
| T-10.20 | 타이틀 바 | ∨ 사용자 메뉴 버튼 | `ChevronDown` → `chevron_down.svg` | `chevron-down` | `ChevronDown` | 없음 (I-637) | I-601 |
| T-10.21 | 사이드바 | 등록 폴더 항목의 폴더 아이콘 (우리 앱이 더한 것, 04 A-12.1) | `Folder` → `folder.svg` | `folder` | `Folder` | 없음 (L-231) | A-12.1 |

- 기준 화면은 `folder_indicator: "icon"`(기본)이라 화살표가 없다 (L-44, L-126). T-10.17 은 설정 옵션을 남길 때만 쓴다.
- Lucide 이름은 **대체 후보**다. 이름과 쓰임이 가깝다는 판단일 뿐, 모양이 같다고 확인한 것이 아니다. 후보를 Zed SVG 와 나란히 놓고 비교해 일치·불일치를 이 표에 적는 일은 남아 있다 (T-18.3, T-31).
- 정정: 조사 자료는 마크다운 탭 아이콘을 `Book`/`book.svg` 로 적었다. 소스의 탭 아이콘은 `FileDoc`(L-69)다. 사이드바·패널 토글도 조사 자료의 `PanelLeft`·`list-tree` 추정 대신 위 표의 이름을 쓴다.
- 파일 트리의 폴더·파일 아이콘은 4절 아이콘 테마가 정한다. 이 표에 넣지 않는다.
- 창 버튼 세 개는 Lucide 가 아니라 글꼴 글리프를 쓴다(01 의 L-83). 03 의 이전 판이 Lucide 대체 후보를 적었던 것을 정정한다. Tauri 는 같은 글리프를 쓰되 해당 글꼴이 없는 환경에서는 같은 모양의 SVG 를 직접 그린다 (판단).
- 이 표의 아이콘은 모두 **크기·선 두께·여백**이 Zed SVG(16×16, 선 1.2)와 같아야 한다. 후보 SVG 를 가져온 뒤 viewBox 16 에 맞추고 선 두께를 1.2 로 조정한다 (판단). 실제 비교는 T-18.3 에 남아 있다.

### 3.3 툴팁 모양 (T-30)

툴팁의 값은 **01-LAYOUT 의 4.7절(L-200 ~ L-272)이 정한다.** 이 문서의 이전 판에 적었던 T-30 값(위치 미확인, 키 형식 `Ctrl+Shift+E`, 테두리 토큰)은 소스로 확정된 01 과 맞지 않아 **폐기**했다. 03 은 색 토큰이 어느 것인지만 연결한다.

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| T-30.1 | 구성·지연·사라짐·위치 | 01 의 L-200 ~ L-210 | 01-LAYOUT |
| T-30.2 | 바깥 상자(바탕·모서리·테두리·그림자)·안쪽 여백·글꼴 | 01 의 L-211 ~ L-216. 바탕은 `elevated_surface.background`, 글자는 `text`, 메타 줄은 `text.muted` 계열이다. **테두리 토큰은 01 의 값을 따른다** | 01-LAYOUT |
| T-30.3 | 단축키 표시 형식 | 01 의 L-217 ~ L-223 (하이픈, `Ctrl-Shift-E`). VSCode 를 적용한 keymap 결과를 쓴다 | 01-LAYOUT, 02 8.4 |
| T-30.4 | 버튼별 문구 | 01 의 L-225 ~ L-272, 02 8.2. 3.2절 표의 툴팁 열은 그 요약이다 | 01-LAYOUT, 02 |

## 4. 파일 아이콘 테마

### 4.1 아이콘 테마 JSON 형식 (T-11, T-12)

| ID | JSON 키 (`themes[]` 안) | 뜻 | 비고 |
|---| --- | --- | --- |
| T-11.1 | `name`, `appearance` | 선택에 쓰는 이름, light/dark | — |
| T-11.2 | `directory_icons` | 일반 폴더의 `collapsed`·`expanded` 아이콘 경로 | — |
| T-11.3 | `named_directory_icons` | 폴더 **이름**별 `collapsed`·`expanded` | 키는 대소문자를 구분한다. 양끝 공백만 자른다 |
| T-11.4 | `chevron_icons` | 펼침 화살표 `collapsed`·`expanded`. `null` 이면 안 그린다 | Catppuccin 은 둘 다 `null` |
| T-11.5 | `file_stems` | 파일 이름 전체 → 아이콘 키 | `README.md`, `Cargo.toml` 같은 이름 |
| T-11.6 | `file_suffixes` | 접미사(점 앞 부분 제외) → 아이콘 키 | `md`, `rs`, `component.ts` |
| T-11.7 | `file_icons` | 아이콘 키 → `{ "path" }` | — |

스키마: `https://zed.dev/schema/icon_themes/v0.2.0.json` (Catppuccin Icons 설치본의 `$schema`). 조사 자료의 v0.3.0 은 설치본에서 확인되지 않았다. 출처: [icon_theme.rs L20-L66](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/icon_theme.rs#L20-L66).

### 4.2 Zed (Default) 아이콘 테마 (T-28)

| ID | 항목 | 값 | 출처 |
|---| --- | --- | --- |
| T-28.1 | 정의 위치 | JSON 이 아니라 Rust 코드 안의 상수. id `zed`, 이름 `Zed (Default)` | [icon_theme.rs L424-L455](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/icon_theme.rs#L424-L455) |
| T-28.2 | 폴더 | 닫힘 `icons/file_icons/folder.svg`, 열림 `folder_open.svg` | 같은 줄 |
| T-28.3 | 화살표 | `chevron_right.svg`, `chevron_down.svg` | 같은 줄 |
| T-28.4 | `named_directory_icons` | 없음. 모든 폴더가 같은 아이콘 | 같은 줄 |
| T-28.5 | 기본 파일 | 키 `default` → `file.svg` | [icon_theme.rs L331](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/icon_theme.rs#L331) |
| T-28.6 | 마크다운 | 키 `markdown` → **`book.svg`** (접미사 `md`, `markdown`) | [L192](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/icon_theme.rs#L192), [L364](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/icon_theme.rs#L364) |
| T-28.7 | `file_stems` | `Containerfile`·`Dockerfile`·`.dockerignore`(docker), `Podfile`(ruby), `Procfile`(heroku) 뿐 | [L68-L72](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/icon_theme.rs#L68-L72) |
| T-28.8 | 정정 | 조사 자료는 `README.md` → `info` 와 `markdown.svg` 를 Zed 기본 규칙으로 적었다. 이 커밋의 기본 테마에는 README 규칙이 없고, 마크다운은 `book.svg` 다 | — |
| T-28.9 | 파일 아이콘 | 언어 로고 SVG 수십 종(`rust.svg`, `python.svg` …). 라이선스가 따로 확인되지 않았다 | [file_icons 목록](https://github.com/zed-industries/zed/tree/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/icons/file_icons) |

우리 방침(판단): README 원칙(Zed 가 직접 그린 아이콘은 개별 라이선스를 확인한 뒤에만 쓴다)에 따라 Zed 의 파일 아이콘 SVG 를 가져오지 않는다. `Zed (Default)` 아이콘 테마는 **Lucide 대체 후보로 만든 우리 버전**이다. 폴더 `folder`/`folder-open`, 기본 파일 `file`, 마크다운 `book-open`, 화살표 `chevron-*`, 그 밖의 확장자는 `file-code`·`file-text` 로 묶는다. 언어 로고는 가져오지 않는다. 이 방침이 Zed 원본과 어떻게 다른지는 4.5절(T-31)에 따로 적었고, **미해결**이다.

### 4.3 아이콘을 고르는 순서 (T-22 정정)

소스([file_icons.rs L28-L95](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/file_icons/src/file_icons.rs#L28-L95))에서 확인한 순서다. 조사 자료의 "소문자 변환" 규칙은 소스에 없다.

파일:

| ID | 순서 | 조회 | 예 |
|---| --- | --- | --- |
| T-22.1 | 1 | 파일 이름 전체를 `file_stems` 에서 찾고, 없으면 `file_suffixes` 에서 찾는다 | `README.md` → stems |
| T-22.2 | 2 | 이름에서 첫 `.` 앞을 떼고 남은 부분으로 1과 같은 조회를 반복한다 | `a.stories.tsx` → `stories.tsx` → `tsx` |
| T-22.3 | 3 | 2에서 못 찾았으면 첫 `.` 앞 부분(스템)을 `file_stems`·`file_suffixes` 에서 찾는다 | `Dockerfile.prod` → `Dockerfile` |
| T-22.4 | 4 | 여러 확장자(`multiple_extensions`)로, 그다음 확장자 또는 숨김 파일 이름으로, 마지막으로 마지막 확장자로 찾는다 | `.data.json` → `json` |
| T-22.5 | 5 | 모두 실패하면 키 `default` | — |
| T-22.6 | 6 | 얻은 키로 `file_icons[키].path` 를 찾는다. **현재 아이콘 테마에 키가 없으면 `Zed (Default)` 에서 찾는다** | — |

폴더:

| ID | 순서 | 조회 |
|---| --- | --- |
| T-22.7 | 1 | 폴더 이름(양끝 공백 제거)을 현재 테마 `named_directory_icons` 에서 찾는다. 펼침 상태에 맞는 경로를 쓴다 |
| T-22.8 | 2 | 없으면 `Zed (Default)` 의 `named_directory_icons` (비어 있다) |
| T-22.9 | 3 | 없으면 현재 테마의 `directory_icons` |
| T-22.10 | 4 | 그래도 없으면 `Zed (Default)` 의 `directory_icons` |

화살표는 `chevron_icons` 를 같은 방식으로 찾는다. 표시 여부는 `folder_indicator`(L-126)가 정한다.

확장 아이콘 테마(예: Catppuccin Icons)는 읽을 때 Zed (Default) 의 `file_stems`·`file_suffixes`·`named_directory_icons` 위에 덮어써진 표가 된다 (T-32.10). 위 순서의 "현재 테마에서 찾는다" 는 이 합친 표에서 찾는 것이다. `file_icons` 에서 키를 못 찾았을 때 Zed (Default) 로 한 번 더 찾는 규칙(6번)은 그대로다.

### 4.4 Catppuccin Icons (T-20)

| ID | 항목 | 내용 |
|---| --- | --- |
| T-20.1 | 출처(고정) | 저장소 https://github.com/catppuccin/zed-icons 의 태그 [v1.24.0](https://github.com/catppuccin/zed-icons/tree/v1.24.0) = 커밋 `fe864c992c5da1e417e5897b741b7de9b90830e0`. 설치본 `extension.toml` 도 version 1.24.0 이다 |
| T-20.2 | 라이선스 | MIT, Copyright (c) 2021 Catppuccin. 태그의 [LICENSE](https://github.com/catppuccin/zed-icons/blob/v1.24.0/LICENSE) 로 확인했다 |
| T-20.3 | 파일 | [`icon_themes/catppuccin-icons.json`](https://github.com/catppuccin/zed-icons/blob/v1.24.0/icon_themes/catppuccin-icons.json) + [`icons/<flavor>/*.svg`](https://github.com/catppuccin/zed-icons/tree/v1.24.0/icons/latte). 설치본 JSON 은 태그의 같은 파일과 바이트가 같다(`cmp` 로 확인) |
| T-20.4 | 테마 8개 | `Catppuccin Latte`(light), `Frappé`·`Macchiato`·`Mocha`(dark), 각각 ` Monochrome` 변형. 폴더는 `latte`, `frappe`, `macchiato`, `mocha` 와 `-monochrome` |
| T-20.5 | 규모 (테마마다 같다) | SVG 656개, `named_directory_icons` 421개, `file_stems` 10개, `file_suffixes` 1810개, `file_icons` 400개 |
| T-20.6 | 우리가 쓰는 것 | 일반 4종(Latte·Frappé·Macchiato·Mocha). Monochrome 은 채용하지 않는다 (판단) |
| T-20.7 | 색 | SVG 안에 flavor 색이 직접 들어 있다. 테마 색을 주입하지 않는다 |
| T-20.8 | 크기 | 16×16 viewBox. `markdown.svg` 는 359바이트, Latte 폴더 전체 약 423KB |

| ID | 키 | 값 (Latte) |
|---| --- | --- |
| T-20.9 | `directory_icons` | `./icons/latte/_folder.svg` / `_folder_open.svg` |
| T-20.10 | `chevron_icons` | `null` / `null` |
| T-20.11 | `file_stems` | `LICENSE`·`README`·`README.md`·`README.txt` → `license`/`readme`, `Makefile` → `makefile`, `Caddyfile` → `caddy`, `.DS_Store` → `macos`, `CMakeLists.txt` → `cmake`, `Cargo.lock` → `cargo-lock`, `Cargo.toml` → `cargo` |
| T-20.12 | `file_suffixes` 예 | `md`·`markdown` → `markdown`, `rs` → `rust`, `json` → `json`, `toml` → `toml`, `txt` → `text`, `png` → `image`, `js` → `javascript`, `ts` → `typescript`, `tsx` → `typescript-react`, `css`, `html`, `py` → `python`, `sh` → `bash`, `yml`·`yaml` → `yaml`, `svg` |
| T-20.13 | `named_directory_icons` 예 | `docs`, `src`, `assets`, `images`, `scripts`, `tests`, `test`, `public`, `config`, `lib`, `.git`, `.github`, `node_modules`, `workflow` 가 있다. `guides`, `herdr`, `rust-gui`, `projects`, `target` 은 없다 |

조회 결과: `workflow` 는 `folder_workflows.svg`(닫힘)·`folder_workflows_open.svg`(열림)를 가리킨다. 키는 `workflow`, 파일 이름은 `folder_workflows` 다. `guides`·`herdr`·`rust-gui` 는 키가 없어 `_folder.svg` 를 쓴다. 기준 화면에서 `workflow` 만 배지 폴더이고 `docs  guides`·`herdr`·`rust-gui` 가 일반 폴더인 것과 맞는다.

| ID | 확인 | 결과 |
|---| --- | --- |
| T-20.14 | `default` 키 | Catppuccin Icons 의 `file_icons` 에 없다 → 4.3절 6번에 따라 알 수 없는 파일은 **Zed 기본의 `file.svg`** 로 나온다. `_file.svg` 는 JSON 이 가리키지 않는다 |
| T-20.15 | `README.md` | `file_stems` → `readme` → `icons/latte/readme.svg` (기준 화면의 ⓘ 모양) |
| T-20.16 | `*.md` | `file_suffixes` → `markdown` → `icons/latte/markdown.svg` (기준 화면의 M↓ 모양) |

### 4.5 Zed (Default) 원본과의 차이 (T-31, 미해결)

3.2절과 4.2절의 대체 방침 때문에 우리의 `Zed (Default)` 는 Zed 원본과 모양이 다르다. 요건 6·7 은 "Zed 기본 아이콘" 을 고르게 하므로 이 차이를 명세에서 숨기지 않는다. 아래는 차이와 풀어야 할 조건이다.

| ID | 항목 | Zed 원본 | 우리 `Zed (Default)` | 차이 | 해결 조건 |
| --- | --- | --- | --- | --- | --- |
| T-31.1 | 폴더 닫힘·열림 | `file_icons/folder.svg`, `folder_open.svg` ([icon_theme.rs L430-L433](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/icon_theme.rs#L430-L433)) | Lucide `folder`, `folder-open` (후보) | 모양 차이 가능. 비교 전 | 두 SVG 를 나란히 비교한다. Zed SVG 의 라이선스(Lucide 파생 여부)를 확인한다 |
| T-31.2 | 기본 파일 | `file.svg` | Lucide `file` (후보) | 위와 같음 | 위와 같음 |
| T-31.3 | 마크다운·문서 | `book.svg` (키 `markdown`, `document`) | Lucide `book-open` (후보) | 위와 같음 | 위와 같음 |
| T-31.4 | 펼침 화살표 | `chevron_right.svg`, `chevron_down.svg` | Lucide `chevron-right`, `chevron-down` (후보) | 위와 같음 | 위와 같음 |
| T-31.5 | 언어·파일 종류 아이콘 | 기본 테마의 `FILE_ICONS` 92개 항목, 서로 다른 SVG 77개 (`rust`, `python`, `docker`, `toml` …) ([icon_theme.rs L314-L394](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/icon_theme.rs#L314-L394)) | 모두 `file-code`·`file-text` 로 묶는다 | **크다.** 확장자별 아이콘이 사라진다 | 각 SVG 의 출처·라이선스를 하나씩 확인한다. 언어 로고는 상표일 수 있다. 확인이 끝난 것만 추가한다 |
| T-31.6 | 색 | 단색 SVG 에 `icon` 계열 색을 입힌다 | 같은 방식 | 없음(방식) | — |
| T-31.7 | 기준 화면 | 기준 화면 두 장은 Catppuccin Latte 아이콘이라 `Zed (Default)` 의 차이는 직접 보이지 않는다. 그러나 **Catppuccin Icons 에도 `default` 키가 없어** 알 수 없는 파일은 Zed 기본의 `file.svg` 로 나온다 (T-20.14). 이 경우에도 우리 대체 아이콘이 보이므로 차이가 기준 화면 밖에 한정되지 않는다 | — | 기준 화면에서도 알 수 없는 확장자 파일에서 차이가 난다 | T-31.1, T-31.2 와 같다 |

`Zed (Default)` 를 고른 화면은 Zed 와의 일치도 판정(04-APP-SPEC 7절)에서 "원본과 다름" 으로 표시한다. T-31 이 풀리면 바꾼다.

그 밖의 `Zed (Default)` 와 `Catppuccin Icons` 의 비교는 아래와 같다.

| ID | 항목 | `Zed (Default)` | `Catppuccin Icons` |
|---| --- | --- | --- |
| T-31.8 | 색 | 단색 SVG. 현재 테마의 `icon` 계열 색을 입힌다 | SVG 에 파스텔 색이 들어 있다 |
| T-31.9 | 폴더 | 모두 같은 아이콘 | 421개 이름별 아이콘 |
| T-31.10 | 화살표 | 있다 | 없다 (`null`) |
| T-31.11 | README | 규칙 없음 (`default`) | `readme` |
| T-31.12 | 마크다운 | `book.svg` | `markdown.svg` |

### 4.6 번들 범위 (T-23 결정 필요)

| ID | 선택지 | 크기 | 비고 |
|---| --- | --- | --- |
| T-23.1 | A. 일반 4 flavor 전체 | 656×4 SVG, 약 1.7MB (Latte 423KB 를 4배한 추정) | 가장 단순. JSON 도 같이 번들 |
| T-23.2 | B. 4 flavor 모두, 우리 앱 확장자 서브셋 | 수십 개×4 | 알 수 없는 파일은 `Zed (Default)` 로 나온다 |

확장 방식(04 의 4.11절)과의 관계: Catppuccin Icons 는 **본체에 내장하기로 결정**했다(T-32.14). 그래서 내장할 범위(위 A·B)가 그대로 앱 크기가 된다. A·B 의 선택만 남아 있다.

판단: A 를 기본으로 한다. 서브셋은 04-APP-SPEC 의 번들 크기 요건이 생길 때 고른다.

## 5. 글꼴

### 5.1 설정과 기본값 (T-14)

| ID | 설정 | 기본 | 실제 글꼴 | 크기 | 쓰는 곳 | 출처 |
| --- | --- | --- | --- | --- | --- | --- |
| T-14.1 | `ui_font_family` | `.ZedSans` | **IBM Plex Sans.** default.json 주석에 `.ZedSans` 가 현재 IBM Plex Sans 의 별칭이며 바뀔 수 있다고 적혀 있다 | `ui_font_size` 16 | 타이틀 바·사이드바·트리·탭·마크다운 미리보기 본문 | [default.json L57-L59](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L57-L59), L-08, L-120 |
| T-14.2 | `buffer_font_family` | `.ZedMono` | **Lilex.** 주석에 `.ZedMono` 가 현재 Lilex 의 별칭이라고 적혀 있다 | `buffer_font_size` 15 | 편집기, 마크다운 코드 | [default.json L29-L31](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L29-L31), L-08, L-121 |
| T-14.3 | `buffer_line_height` | `comfortable` | — | 편집기 글자 크기의 1.618배 | 편집기 줄 간격 | [default.json L54](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L54), [buffer_line_height.rs L14-L20](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/buffer_line_height.rs#L14-L20) (`comfortable` = 1.618, `standard` = 1.3, `custom` 은 1.0 이상). 편집기의 줄 높이다. UI 글자의 `phi()`(L-09)와 출처가 다르다. 15 × 1.618 = 24.27 논리 px 이고 최종 반올림은 렌더러가 정한다, L-121 |
| T-14.4 | `.SystemUIFont` | `ui_font_family` 에 값으로 줄 수 있다 | 시스템 UI 글꼴 | — | UI 글꼴을 시스템 글꼴로 바꿀 때 | default.json L56 주석 |

기준 화면은 UI 16, 편집기 15다. 글꼴 이름이 우리 앱에서 어떻게 정해지는지는 5.3절(앱 판단)이다.

### 5.2 번들 글꼴 (T-15)

| ID | 글꼴 | 파일 (`assets/fonts/…`) | 저작권 | 라이선스 | 출처 |
| --- | --- | --- | --- | --- | --- |
| T-15.1 | IBM Plex Sans | `ibm-plex-sans/IBMPlexSans-Regular.ttf`, `-Italic`, `-SemiBold`, `-SemiBoldItalic`, `license.txt` | © 2017 IBM Corp., Reserved Font Name "Plex" | SIL OFL 1.1 | [폴더](https://github.com/zed-industries/zed/tree/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/fonts/ibm-plex-sans), [license.txt](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/fonts/ibm-plex-sans/license.txt) |
| T-15.2 | Lilex | `lilex/Lilex-Regular.ttf`, `-Bold`, `-Italic`, `-BoldItalic`, `OFL.txt` | 2019 The Lilex Project Authors | SIL OFL 1.1 | [폴더](https://github.com/zed-industries/zed/tree/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/fonts/lilex), [OFL.txt](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/fonts/lilex/OFL.txt) |
| T-15.3 | 글꼴 등록 | Zed 는 `assets/fonts/**/*` 를 읽어 앱에 내장 글꼴로 등록한다 | — | — | [assets.rs L15](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/assets/src/assets.rs#L15) |
| T-15.4 | 굵기 | Plex Sans 는 Regular·SemiBold 만 있다. 마크다운 제목 600(L-104)은 SemiBold 로 그린다. 굵게(700)는 SemiBold 또는 합성 굵게다 (판단) | — | — | 위 폴더 목록 |
| T-15.5 | 한글 글리프 | **없다.** 두 `Regular.ttf` 의 `cmap` 에서 U+AC00(가), U+D55C(한), U+3131(ㄱ), U+3042(あ), U+4E00(一) 을 조회했더니 모두 없었고 U+0041(A) 만 있었다. 다른 굵기·기울임 파일은 조회하지 않았다 | — | — | [IBMPlexSans-Regular.ttf](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/fonts/ibm-plex-sans/IBMPlexSans-Regular.ttf), [Lilex-Regular.ttf](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/fonts/lilex/Lilex-Regular.ttf) |
| T-15.6 | Windows 의 기본 UI 글꼴 | Zed 가 Windows 에서 시스템 UI 글꼴로 쓰는 이름은 `Segoe UI` 다 | — | — | [direct_write.rs L1980-L1981](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui_windows/src/direct_write.rs#L1980-L1981) |

### 5.3 Windows 대체 글꼴 (T-16, 앱 판단)

이 절의 순서는 Zed 소스에서 확인한 사실이 아니라 **우리 앱의 판단**이다. 근거는 T-15.5(번들 글꼴에 한글이 없다)와 T-15.6(Zed 의 Windows 기본 UI 글꼴)뿐이다. Zed 자신의 글꼴 대체 설정(`ui_font_fallbacks`, "플랫폼 기본 대체 목록과 합친다", [default.json L60-L62](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L60-L62))의 실제 동작은 열어 보지 않았다.

| ID | 용도 | 순서 | 근거 |
| --- | --- | --- | --- |
| T-16.1 | UI | IBM Plex Sans → Segoe UI → Malgun Gothic | T-15.5, T-15.6 |
| T-16.2 | 편집기 | Lilex → Cascadia Mono → Consolas → Malgun Gothic | T-15.5. Cascadia Mono·Consolas 는 Windows 에 있는 고정폭 글꼴이라는 판단 |

시스템 글꼴(Segoe UI, Cascadia Mono, Consolas, Malgun Gothic)은 이름만 부르므로 라이선스 파일이 필요 없다 (판단).

## 6. 테마·아이콘 테마 선택

### 6.1 Zed 의 선택기 (T-21)

| ID | 항목 | 테마 선택기 | 아이콘 테마 선택기 | 출처 |
|---| --- | --- | --- | --- |
| T-21.1 | 액션 | `theme_selector::Toggle` | `icon_theme_selector::Toggle` | [default-windows.json L688](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L688) |
| T-21.2 | 기본 키 (Windows) | `Ctrl+K Ctrl+T` | 없다. 명령 팔레트로만 연다 | 같은 파일 (아이콘 선택기 키 없음 확인) |
| T-21.3 | 목록 | 모달 Picker. 위에 검색 입력, 아래에 이름 목록. 현재 테마가 처음 선택돼 있다. **정렬: 어두운 테마가 먼저, 밝은 테마가 나중이고, 각각 이름 순** (문자열 비교) | 같은 구조, 같은 정렬 ([theme_selector.rs L176-L182](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L176-L182), [icon_theme_selector.rs L98-L103](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L98-L103)) | [theme_selector.rs L154-L219](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L154-L219) |
| T-21.4 | 미리 적용 | 선택 줄이 바뀔 때마다 그 테마를 바로 적용 | 같음 (미리 적용은 문자열 설정으로 덮어 쓴다) | [theme_selector.rs L228-L275](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L228-L275), [icon_theme_selector.rs L156](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L156) |
| T-21.5 | 취소 | `Escape`·바깥 클릭이면 열기 전 설정으로 되돌린다. 확정한 뒤에는 되돌리지 않는다 | 같음 | [theme_selector.rs L83-L93, L250-L257, L418-L422](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L250-L257) |
| T-21.6 | 확정 | `Enter` 로 설정 파일에 저장하고 닫는다 | 같음 | [theme_selector.rs L393-L416](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L393-L416), [icon_theme_selector.rs L198-L200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L198-L200) |

### 6.2 설정 값과 mode 규칙 (T-21, 정정)

`theme` 은 문자열(고정) 또는 `{ mode, light, dark }` 객체다. `mode` 는 `system`, `light`, `dark`.

| ID | 상황 | 결과 | 출처 |
|---| --- | --- | --- |
| T-21.10 | 문자열 설정 | 선택한 이름으로 문자열을 바꾼다 | [L292-L294](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L292-L294) |
| T-21.11 | 객체 설정, 선택한 테마가 light | `light` 칸만 새 이름으로 바꾸고 `dark` 는 열기 전 값을 유지 | [L356-L376](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L356-L376) |
| T-21.12 | 객체 설정, 선택한 테마가 dark | `dark` 칸만 바꾸고 `light` 는 유지 | 같은 줄 |
| T-21.13 | `mode` 가 `system` 이고 새 테마의 밝기가 지금 시스템 밝기와 같다 | `mode` 는 `system` 그대로 | [L339-L349](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L339-L349) |
| T-21.14 | `mode` 가 `system` 인데 새 테마의 밝기가 시스템과 다르다 | `mode` 를 새 테마 밝기(`light`/`dark`)로 바꾼다. 그래야 선택한 테마가 보인다 | 같은 줄 |
| T-21.15 | 선택기를 연 사이에 설정 파일이 다른 형태로 바뀌었다 | 미리 적용을 하지 않는다 | [L278-L329](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L278-L329) |

정정: 조사 자료는 "현재 활성 모드의 칸만 바꾼다"고 적었다. 소스는 **선택한 테마의 밝기**로 칸을 고르고, 필요하면 `mode` 도 바꾼다.

#### 저장 규칙 (T-21, 확정)

저장은 미리 적용과 별개 함수다. 고정 커밋의 [settings.rs L262-L294](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/settings.rs#L262-L294)(`set_theme`)와 [L296-L319](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/settings.rs#L296-L319)(`set_icon_theme`)에서 확인했다. 테마는 미리 적용 쪽 규칙(위 표)과 같은 결과가 나온다.

| ID | 대상 | 설정의 현재 모양 | 저장 결과 |
| --- | --- | --- | --- |
| T-21.20 | `theme` | 키가 없다 | 문자열(고정)로 새 이름을 쓴다 |
| T-21.21 | `theme` | 문자열 | 새 이름으로 바꾼다 |
| T-21.22 | `theme` | 객체 | 새 테마의 `appearance` 가 light 면 `light` 칸, dark 면 `dark` 칸을 새 이름으로 바꾼다. 다른 칸은 그대로 |
| T-21.23 | `theme` | 객체 | `mode` 가 `system` 이고 새 테마의 밝기가 시스템 밝기와 같으면 `mode` 를 그대로 둔다. 그 밖에는 `mode` 를 새 테마의 밝기(`light`/`dark`)로 바꾼다 |
| T-21.24 | `icon_theme` | 키가 없다 | 문자열(고정)로 새 이름을 쓴다 |
| T-21.25 | `icon_theme` | 문자열 | 새 이름으로 바꾼다 |
| T-21.26 | `icon_theme` | 객체 | **아이콘 테마의 밝기가 아니라 `mode`** 로 칸을 고른다. `mode` 가 `light` 면 `light` 칸, `dark` 면 `dark` 칸, `system` 이면 **창의 현재 밝기**(저장 시점의 `window.appearance()`, [icon_theme_selector.rs L196-L200](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/icon_theme_selector.rs#L196-L200))에 해당하는 칸을 바꾼다. `mode` 는 바꾸지 않는다 |

- 테마와 달리 아이콘 테마 저장은 `mode` 를 바꾸지 않는다 (T-21.26).
- `theme_settings::set_mode`(설정 화면용)는 우리 범위 밖이라 쓰지 않는다.
- 구분: 위 표는 **확정**(소스 대조)이다. 우리 앱이 이 규칙을 그대로 구현한다는 것은 **결정**이다. 이 표에 남은 **추정**은 없다.
- 확인하지 않은 것: 객체 설정에서 `light`/`dark` 키가 빠진 경우의 읽기 규칙.

`icon_theme` 도 문자열 또는 `{ mode, light, dark }` 다. 기본값은 문자열 `"Zed (Default)"` 다. 미리 적용은 문자열 설정으로 덮어 쓴다.

### 6.3 우리 앱의 선택 UI (T-29, 판단)

| ID | 항목 | 값 |
|---| --- | --- |
| T-29.1 | 테마 목록 | 6종. 표시 순서는 Zed 규칙(어두운 것 먼저, 이름 순): Catppuccin Frappé, Catppuccin Macchiato, Catppuccin Mocha, One Dark, Catppuccin Latte, One Light |
| T-29.2 | 아이콘 테마 목록 | 5종. 표시 순서는 같은 규칙: Catppuccin Frappé, Catppuccin Macchiato, Catppuccin Mocha, Zed (Default)(어두움으로 분류, [icon_theme.rs L430](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme/src/icon_theme.rs#L430)), Catppuccin Latte. `Zed (Default)` 는 Lucide 대체 후보로 만든 우리 버전이라 Zed 원본과 다르다 (T-31) |
| T-29.3 | 여는 길 | ☰ 메뉴 → Theme…, Icon Theme…. 테마는 `Ctrl+K Ctrl+T` 도 연다. 아이콘 테마는 키를 주지 않는다 (Zed 와 같음) |
| T-29.4 | 동작 | 6.1절과 같다. 이동 즉시 미리 적용, `Escape` 로 되돌림, `Enter` 로 저장 |
| T-29.8 | 선택기 외형 | 폭·높이·위치·입력칸 문구·행 높이·선택 강조·빈 결과·푸터는 Picker 공통 명세(P-001 ~, 작성 중)를 따른다. 값은 아직 정하지 않았다 |
| T-29.9 | 시스템 밝기 변경 | `mode: system` 에서 OS 밝기가 바뀌는 경우(선택기가 열려 있는 동안 포함)는 T-32.18 이 정한다. 우리 앱에서 OS 밝기를 감지하는 방법은 미확인이고 출발 문서의 검토 항목으로 넘겼다 (04 의 A-60.12) |
| T-29.5 | mode 규칙 | 6.2절과 같다. 기본 설정은 `{ mode: system, light: One Light, dark: One Dark }`, 기준 화면은 dark 가 Catppuccin Mocha |
| T-29.6 | 아이콘 테마와 밝기 | 문자열 설정이면 시스템 밝기와 무관하게 고정. 기준 화면은 Mocha 테마에 Latte 아이콘을 쓴다 |
| T-29.7 | 저장 위치와 키 | 04-APP-SPEC 의 설정 파일 형식을 따른다 |

### 6.4 확장으로 들어온 테마·아이콘 테마 (T-32)

확장으로 설치한 테마·아이콘 테마가 내장(One, Zed (Default), 내장으로 둔 Catppuccin)과 어떻게 함께 쓰이는지의 규칙이다. 확장 패키지 형식과 설치는 04-APP-SPEC 의 4.11절이 정한다. 출처는 확장 조사의 X- ID 다 (소스 커밋 14b7b0e).

| ID | 항목 | 규칙 | 구분 | 근거 |
| --- | --- | --- | --- | --- |
| T-32.1 | 하나의 등록부 | 테마와 아이콘 테마는 내장이든 확장이든 **이름을 키로 하는 하나의 목록**에 들어간다. 내장 여부를 나타내는 필드가 없다 | 확정 | X-57 |
| T-32.2 | 읽는 순서 | 내장을 먼저 등록하고, 확장은 읽기가 끝나는 대로 같은 목록에 더한다 | 확정 | X-57, X-38 |
| T-32.3 | 같은 이름 | 나중에 들어온 것이 이긴다고 보인다(덮어쓰기). **내장 Catppuccin 테마와 사용자가 설치한 `catppuccin` 확장의 테마 이름이 같으므로 확장이 이긴다.** 두 JSON 이 같은 파일이면 결과는 같다 | 추정 | X-57 |
| T-32.4 | 선택기 목록 | 두 선택기 모두 **내장과 확장을 구분하는 표시 없이** 같은 목록으로 보인다. 정렬은 6.1절과 같다: 어두운 테마가 먼저, 밝은 테마가 나중, 각각 이름 순 | 확정 | X-60 |
| T-32.5 | 현재 항목·미리 적용 | 열릴 때 지금 적용된 항목에 선택이 맞춰지고 그 행 오른쪽 끝에 체크 아이콘(`Check`, `icon.muted` 계열)이 보인다. 이동하면 바로 적용하고 `Esc` 로 원래대로, `Enter` 로 저장한다 (6.1절, 6.2절과 같다) | 확정 | X-60 |
| T-32.6 | 선택기 푸터 | Zed 는 선택기 아래에 `View Theme Docs`(왼쪽)와 `Install Themes`(오른쪽)을 둔다(아이콘 테마는 `View Icon Theme Docs`, `Install Icon Themes`). **결정(확장 세부): 확장 화면이 있고 설치는 로컬뿐이다.** 그래서 `Install Themes`·`Install Icon Themes` 는 **둔다.** 누르면 확장 화면(`Ctrl+Shift+X`, 04 의 A-58)을 테마·아이콘 테마 필터로 열고, 거기서 폴더·파일로 설치한다. `View Theme Docs`·`View Icon Theme Docs` 는 온라인 문서로 가는 링크라 **만들지 않는다.** 모양은 푸터 안쪽 여백 8, 위 테두리 1px | 확정(Zed) + 결정 | X-60, _decisions.md |
| T-32.7 | 설치 직후 선택기 | 확장 화면에서 설치(로컬 폴더·파일·개발용)가 끝나면 그 확장의 테마만 걸러 놓은 테마 선택기를(테마가 없고 아이콘 테마만 있으면 아이콘 테마 선택기를) 바로 연다. 확장 화면이 있으므로 적용한다 | 확정(Zed) + 결정 | X-51 |
| T-32.8 | 시작할 때 | 설정이 확장 테마를 가리키는데 아직 확장을 다 읽지 못했으면 **오류 없이 폴백**(T-32.19)으로 잠깐 보이고, 읽기가 끝나면 설정 값으로 바뀐다. 현재 활성 확장 테마·아이콘 테마는 창이 뜨기 전에 먼저 읽는다(04 의 A-60.2). 내장 Catppuccin 은 이 깜빡임이 없다(T-32.14) | 확정 | X-59, X-105, X-106 |
| T-32.9 | 제거된 확장의 테마를 쓰고 있을 때 | 설정 값은 그대로 두고 화면은 같은 폴백(T-32.19)으로 돌아간다. 이 경로에서 설정 파일을 고치는 코드는 확인되지 않았다. 설정 파일을 자동으로 고치지 않는다 | 확정(폴백 경로) + 판단(설정 비수정) | X-106 |
| T-32.10 | 확장 아이콘 테마의 빈 곳 채우기 | 확장 아이콘 테마를 읽을 때 **Zed (Default) 의 `file_stems`, `file_suffixes`, `named_directory_icons` 를 먼저 복사하고 그 위에 확장 쪽 매핑을 덮어쓴다.** 그래서 확장이 정의하지 않은 확장자·폴더 이름도 빈 아이콘이 되지 않는다. 4.3절의 조회 순서(T-22)는 이 합친 표를 쓴다 | 확정 | X-58 |
| T-32.11 | 아이콘 경로 | 확장 아이콘 테마 JSON 의 아이콘 `path` 는 **확장 루트 기준 상대 경로**다(앞의 `./` 를 뗀다). 확장 폴더 밖을 가리키는 경로는 거부한다 | 확정 + 판단 | X-20, X-66 |
| T-32.12 | 확장 테마 JSON | 형식은 03 의 T-01 ~ T-06 과 같다. 한 파일에 테마가 여럿 들어 있어도 테마마다 목록에 하나씩 들어간다. 파일 하나에서 이름이 겹치면 확인하지 못했다 | 확정 + 미확인 | X-18 |
| T-32.13 | 테마 종류 | 확장 테마도 `appearance`(light/dark)를 가진다. `theme.mode` 규칙(6.2절)은 내장과 같다 | 확정 | X-18, T-21.20 |
| T-32.14 | 내장하는 것 | **One Dark·One Light·Zed (Default) 아이콘 테마, Catppuccin 테마 4종(Latte·Frappé·Macchiato·Mocha), Catppuccin Icons 를 모두 본체에 내장한다.** Catppuccin 은 MIT 고지를 함께 배포한다(7절). 내장도 확장과 **같은 파일 형식·같은 파서**를 쓴다. 사전 설치형은 쓰지 않는다 | 결정 | _decisions.md "확장 세부", X-61 |
| T-32.15 | 선택기 목록의 갱신 시점 | 두 선택기 모두 **열릴 때 목록을 한 번만 복사한다.** 열려 있는 동안 확장 읽기가 끝나도 새 항목이 생기지 않는다. 닫았다 다시 열면 보인다 | 확정 | X-107, [theme_selector.rs L164-L184](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_selector/src/theme_selector.rs#L164-L184) |
| T-32.16 | 읽는 중 열었을 때의 초기 선택 | 테마 선택기는 지금 적용 중인 테마(폴백일 수 있다)에 체크·초기 선택을 둔다. 아이콘 테마 선택기는 설정에 적힌 이름 기준이어서 목록에 없으면 첫 항목을 선택하고 체크는 어디에도 없다 | 확정 | X-107 |
| T-32.17 | 읽는 중 취소 | 취소하면 열기 전 설정으로 되돌리고, 그사이 읽기가 끝났으면 원래 확장 테마가 정상 적용된다. 아이콘 테마도 원래 이름으로 되돌린다 | 확정 | X-107 |
| T-32.18 | OS 밝기 변경 (`mode: system`) | 테마·아이콘 테마를 새 밝기의 칸으로 다시 계산하고 모든 창을 다시 그리며 읽은 언어의 구문 색 표를 새 테마로 다시 만든다. 선택기가 열려 있으면 미리 적용은 열 때의 밝기를, 확정은 확정 순간의 밝기를 쓴다. 규칙은 04 의 A-60.7 ~ A-60.10 | 확정 | X-108, X-109 |
| T-32.19 | 폴백 선택 | 설정 이름이 등록부에 없으면 **OS 밝기** 기준으로 라이트면 One Light, 다크면 One Dark(설정 `mode` 가 고정이어도 OS 밝기 기준), 아이콘 테마는 Zed (Default). 오류 로그는 확장 읽기가 끝난 뒤에만 남긴다 | 확정 | X-106, [theme_settings.rs L164-L201](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/theme_settings/src/theme_settings.rs#L164-L201) |

## 7. 라이선스 고지 목록 (T-17)

배포물에 고지를 함께 넣어야 하는 것이다. 이 표는 법률 자문이 아니다.

| ID | 자산 | 출처 | 라이선스 | 고지에 넣을 것 |
|---| --- | --- | --- | --- |
| T-17.1 | One Dark / One Light 테마 JSON | zed-industries/zed `assets/themes/one/one.json` | MIT | Copyright (c) 2014 GitHub Inc. 와 MIT 허가문 ([LICENSES](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/themes/LICENSES) 의 One Dark·One Light 절) |
| T-17.2 | Catppuccin 테마 4종 JSON | [catppuccin/zed v0.2.27](https://github.com/catppuccin/zed/blob/v0.2.27/themes/catppuccin-mauve.json) | MIT | Copyright (c) 2021 Catppuccin 과 MIT 허가문 ([LICENSE](https://github.com/catppuccin/zed/blob/v0.2.27/LICENSE)) |
| T-17.3 | Catppuccin Icons SVG·JSON | [catppuccin/zed-icons v1.24.0](https://github.com/catppuccin/zed-icons/tree/v1.24.0) | MIT | Copyright (c) 2021 Catppuccin 과 MIT 허가문 ([LICENSE](https://github.com/catppuccin/zed-icons/blob/v1.24.0/LICENSE)) |
| T-17.4 | UI 아이콘 | Lucide, https://lucide.dev | ISC | Lucide 저작권 줄(Cole Bemis 2013-2022 Feather 부분, Lucide Contributors 2022)과 ISC 허가문 ([assets/icons/LICENSES](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/icons/LICENSES) 와 같은 문구) |
| T-17.5 | IBM Plex Sans (번들할 때) | [zed `assets/fonts/ibm-plex-sans`](https://github.com/zed-industries/zed/tree/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/fonts/ibm-plex-sans) | SIL OFL 1.1 | © 2017 IBM Corp. 과 OFL 전문(`license.txt`) |
| T-17.6 | Lilex (번들할 때) | [zed `assets/fonts/lilex`](https://github.com/zed-industries/zed/tree/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/fonts/lilex) | SIL OFL 1.1 | 2019 The Lilex Project Authors 와 OFL 전문(`OFL.txt`) |
| T-17.7 | 시스템 글꼴 | Segoe UI, Cascadia Mono, Consolas, Malgun Gothic | Windows 포함 | 이름만 지정하면 고지 없음 |
| T-17.8 | 가져오지 않는 것 | Zed 의 `crates/*`(GPL-3.0), 키맵 JSON 원문, Zed 가 직접 그린 SVG, 언어 로고 SVG | — | — |

고지 파일 위치는 04-APP-SPEC 에서 정한다 (예: 앱 폴더의 `NOTICE` 또는 `licenses/`).

## 8. 확인하지 못한 것 (T-18)

| ID | 내용 |
| --- | --- |
| T-18.3 | Lucide 대체 후보(3.2절)와 Zed SVG 의 모양 비교를 하지 않았다. `Zed (Default)` 의 원본과의 차이는 T-31 에서 미해결이다 |
| T-18.4 | 테마·아이콘 테마 선택기의 정렬(6.1절), 확장 읽는 중 열었을 때와 OS 밝기 변경 때의 동작(T-32.15 ~ T-32.19)은 **확정**했다. 남은 **미확인**: 입력칸 문구(placeholder), 일치 강조, 빈 결과 화면, 외형(폭·높이·위치·행)은 Picker 공통 조사(P-001 ~)가 정하고, OS 밝기를 감지하는 방법(gpui·Tauri 각각)은 02-gpui·03-tauri 출발 문서의 검토 항목으로 넘겼다 |
| T-18.5 | 한글 글리프 조회는 두 글꼴의 Regular 파일만 했다 (T-15.5). Windows 11 의 Zed 가 한글에 실제로 쓰는 글꼴과 굵기·기울임 합성은 실행 대조 전이다. 한글이 많은 기준 화면은 이 값으로 줄 높이가 달라질 수 있다 (판단) |
| T-18.7 | 상태 표시줄 커서 위치 글자색: Mocha 화면은 `text.muted` 로, Latte 화면은 `text` 로 읽혔다. 01 의 L-97 이 확정한다 |
| T-18.6 | 기준 화면 색 실측은 조사 자료의 값이다. 이 문서에서 다시 측정하지 않았다 |
| T-23 | 아이콘 번들 범위(4.6절)는 04-APP-SPEC 에서 확정한다 |
| T-18.8 | 확장 테마가 같은 파일 안에서 이름이 겹칠 때, 확장끼리 이름이 겹칠 때의 덮어쓰기 규칙은 소스로 확정하지 못했다 (T-32.3, T-32.12) |
| T-18.9 | 제거한 확장의 테마가 선택돼 있을 때 Zed 가 설정을 고치는지 여부는 확인하지 못했다 (T-32.9 는 우리 판단) |

---

이전: [02-INTERACTION.md](02-INTERACTION.md) · 다음: [04-APP-SPEC.md](04-APP-SPEC.md) · 입구: [../README.md](../README.md)
