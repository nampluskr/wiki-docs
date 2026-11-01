# Zed 테마·아이콘·글꼴 역공학 조사 (ID 접두사: T-)

> 본 문서는 `zed-industries/zed` 공식 저장소, 공식 테마·아이콘 스키마 문서(`zed.dev/docs`), 기준 화면(`D:\projects\docs\guides\zed-like-editor\01-zed\assets\zed-reference.png`) 픽셀 측정을 바탕으로 우리 말로 요약한 명세다. GPL-3.0 앱 코드는 원문을 복사하지 않고 식별자·토큰명·치수·동작 사실만 기술했다.

---

## 1. 테마 JSON 구조 및 주요 스타일 토큰 (`T-01` ~ `T-08`)

### T-01 · 테마 파일 최상위 구조 (Theme Family)
- **스키마 URL**: `https://zed.dev/schema/themes/v0.2.0.json` (출처: https://zed.dev/docs/themes , https://github.com/zed-industries/zed/blob/main/crates/theme/src/schema.rs)
- **구조 요약**: 하나의 테마 JSON 파일은 **Theme Family(테마 패밀리)** 단위로 정의되며, 최상위에 패밀리 메타데이터(`$schema`, `name`, `author`)와 개별 테마 배열인 `themes`를 가진다 (출처: https://zed.dev/docs/themes).
- **색상 표기 형식**: `#RRGGBB` 또는 알파 채널을 포함한 8자리 16진수 문자열 `#RRGGBBAA`를 사용한다 (출처: https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json).

| JSON 경로 | 타입 | 필수 | 설명 | 출처 |
| --- | --- | --- | --- | --- |
| `$schema` | `string` | 권장 | `"https://zed.dev/schema/themes/v0.2.0.json"` | https://zed.dev/docs/themes |
| `name` | `string` | 필수 | 테마 패밀리 이름 (예: `"One"`, `"Catppuccin"`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `author` | `string` | 필수 | 제작자 이름 또는 조직명 | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `themes` | `array` | 필수 | 패밀리에 속한 개별 테마 객체 배열 (다크·라이트 변종 포함) | https://zed.dev/docs/themes |
| `themes[].name` | `string` | 필수 | 설정(`"theme"`)에서 선택하는 개별 테마 이름 (예: `"One Dark"`) | https://zed.dev/docs/themes |
| `themes[].appearance` | `"dark" \| "light"` | 필수 | 다크/라이트 구분. 시스템 외형 연동 및 기본 폴백 계산에 사용 | https://github.com/zed-industries/zed/blob/main/crates/theme/src/schema.rs |
| `themes[].style` | `object` | 필수 | 평탄화된 점 표기법(dot-notation) UI 토큰 + `players` 배열 + `syntax` 맵 | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |

### T-02 · 배경·표면·경계선·요소 상태 토큰 (`style.*`)
`crates/theme/src/styles/colors.rs`의 `ThemeColors` 구조체 필드가 serde 직렬화를 통해 점 표기법(`border.variant`, `element.hover` 등) 키로 매핑된다 (출처: https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs).

| 토큰 이름 (`style` 내부 키) | 의미 및 우리 범위 적용 영역 | One Dark 기본값 (`one.json`) | 스크린샷(`zed-reference.png`) 측정/매핑값 | 출처 |
| --- | --- | --- | --- | --- |
| `background` | 창 전체 최하단 기본 배경색 (타이틀 바·상태 표시줄의 기반 표면) | `#3b414dff` | `#11111bff` (Crust) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `surface.background` | 패널·모달 하부 등 기본 표면 배경색 | `#2f343eff` | `#181825ff` (Mantle) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `elevated_surface.background` | 드롭다운 메뉴(`☰`), 컨텍스트 메뉴, 대화상자 등 떠 있는 표면 배경 | `#2f343eff` | `#1e1e2eff` / `#313244ff` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `border` | 사이드바·프로젝트 패널·탭 바·상태 표시줄 사이의 1px 기본 구분선 | `#464b57ff` | `#181825ff` ~ `#313244ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `border.variant` | 보조 경계선, 마크다운 표 셀 테두리, 입력창 비활성 테두리 | `#363c46ff` | `#313244ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `border.focused` | 포커스된 패널·입력창(폴더/파일 이름 변경 입력란 등)의 강조 테두리 | `#47679eff` | `#89b4faff` / `#b4befeff` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `border.selected` | 선택된 항목의 테두리 | `#293b5bff` | `#45475aff` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `border.transparent` | 레이아웃 공간만 차지하는 투명 테두리 | `#00000000` | `#00000000` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `element.background` | 버튼·목록 아이템 등 인터랙티브 요소의 기본 배경 | `#2e343eff` | `#313244ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `element.hover` | 트리 항목·사이드바 항목·아이콘 버튼에 마우스를 올렸을 때 배경 | `#363c46ff` | `#31324480` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `element.active` | 마우스 클릭(press) 중인 요소의 배경 | `#454a56ff` | `#45475aff` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `element.selected` | 사이드바의 선택된 폴더 항목, 프로젝트 트리의 선택된 파일/폴더 배경 | `#454a56ff` | `#3b3c4fff` (실측) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `element.disabled` | 비활성화된 버튼(`←` `→` 히스토리 없음 등) 배경 | `#2e343eff` | `#181825ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `ghost_element.background` | 평소에는 배경이 투명한 아이콘 버튼(`☰`, `+`, 분할, 토글 등)의 기본 상태 | `#00000000` | `#00000000` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `ghost_element.hover` | 투명 아이콘 버튼 위에 마우스를 올렸을 때의 반투명 배경 | `#363c46ff` | `#31324480` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `ghost_element.active` | 투명 아이콘 버튼을 누르는 순간의 배경 | `#454a56ff` | `#45475aff` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `ghost_element.selected` | 토글이 켜진 상태 표시줄 아이콘(프로젝트 패널 열림 등)의 배경 | `#454a56ff` | `#3b3c4fff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `drop_target.background` | 탭 드래그 분할 또는 폴더 이동 시 드롭 영역 오버레이 색 | `#83899480` | `#89b4fa33` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |

### T-03 · 글자·아이콘·강조 토큰 (`text.*`, `icon.*`, `link_text_hover`)
| 토큰 이름 | 의미 및 우리 범위 적용 영역 | One Dark 기본값 | 스크린샷(`zed-reference.png`) 측정/매핑값 | 출처 |
| --- | --- | --- | --- | --- |
| `text` | 기본 본문 글자, 활성 탭 제목, 트리 파일명, 마크다운 제목·본문 | `#dce0e5ff` | `#cdd6f4ff` (실측) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `text.muted` | 비활성 탭 제목, 사이드바 보조 텍스트(`12h`), 마크다운 인용문, 상태 표시줄 숫자 | `#a9afbcff` | `#a6adc8ff` (Subtext0) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `text.placeholder` | 빈 입력창 안내 문구 | `#878a98ff` | `#6c7086ff` (Overlay0) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `text.disabled` | 비활성 메뉴·버튼 글자 | `#878a98ff` | `#585b70ff` (Surface2) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `text.accent` | 강조 텍스트, 마크다운 링크 텍스트(`[01-INSTALL.md]`) | `#74ade8ff` | `#89b4faff` / `#b4befeff` (실측 보라·파랑 계열) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `icon` | 기본 UI 아이콘 색 | `#dce0e5ff` | `#cdd6f4ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `icon.muted` | 트리 기본 폴더 아이콘, 비활성 탭 아이콘, 보조 버튼 아이콘 | `#a9afbcff` | `#a6adc8ff` / `#7f849cff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `icon.disabled` | 비활성 화살표(`→`) 등의 아이콘 색 | `#878a98ff` | `#585b70ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `icon.accent` | 선택·활성 상태 토글 아이콘 강조색 | `#74ade8ff` | `#89b4faff` / `#cba6f7ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `link_text_hover` | 마크다운 미리보기 링크 호버 색상 | `#74ade8ff` | `#89b4faff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |

### T-04 · 구조 영역별 전용 토큰 (타이틀 바·패널·탭 바·툴바·상태 표시줄·스크롤바)
| 토큰 이름 | 의미 및 우리 범위 적용 영역 | One Dark 기본값 | 스크린샷(`zed-reference.png`) 실측값 | 출처 |
| --- | --- | --- | --- | --- |
| `title_bar.background` | 상단 타이틀 바(`☰ projects`, 창 버튼) 배경 | `#3b414dff` | `#11111bff` (실측 `(1000,20)`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `title_bar.inactive_background` | 창이 포커스를 잃었을 때 타이틀 바 배경 | `#2e343eff` | `#11111bff` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `panel.background` | 맨 왼쪽 사이드바(폴더 목록) 및 왼쪽 Dock(프로젝트 패널) 배경 | `#2f343eff` | `#181825ff` (실측 `(100,600)`, `(600,600)`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `panel.focused_border` | 패널이 키보드 포커스를 가질 때 외곽선 색 | `null` (기본 `border.focused` 폴백) | 미표시 또는 `#89b4faff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `panel.indent_guide` | 프로젝트 패널 트리의 세로 들여쓰기 안내선 기본 색 | `#464b5766` (border 변형) | `#313244ff` (실측) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `panel.indent_guide_hover` | 마우스가 위치한 폴더 하위 세로 들여쓰기 안내선 색 | `#a9afbc80` | `#45475aff` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `panel.indent_guide_active` | 현재 선택된 항목이 속한 부모 폴더의 세로 안내선 색 | `#dce0e580` | `#585b70ff` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `tab_bar.background` | 탭 바 빈 영역 및 탭 컨테이너 뒷배경 | `#2f343eff` | `#11111bff` (실측 `(450,55)`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `tab.active_background` | 현재 선택된 활성 탭(`Preview README.md`) 배경 (편집기 배경과 동일하게 이어짐) | `#282c33ff` | `#1e1e2eff` (실측 `(1500,85)`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `tab.inactive_background` | 비활성 탭(`README.md`, `Preview 05-SLINT.md`) 배경 | `#2f343eff` | `#11111bff` / `#0b0b11ff` (실측 `(1250,85)`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `toolbar.background` | 탭 바 아래 브레드크럼/보조 툴바 배경 (편집기 배경과 동일) | `#282c33ff` | `#1e1e2eff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `status_bar.background` | 하단 상태 표시줄(토글 아이콘, 커서 줄:칸 표시) 배경 | `#3b414dff` | `#11111bff` (실측 `(1500,1680)`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `scrollbar.thumb.background` | 스크롤바 막대(thumb) 반투명 배경 | `#4c566a4c` | `#585b704c` (추정) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `scrollbar.thumb.hover_background` | 스크롤바 막대 호버 시 배경 | `#363c46ff` | `#6c708680` (추정) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `scrollbar.thumb.border` | 스크롤바 막대 테두리 | `#363c46ff` | `#00000000` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `scrollbar.track.background` | 스크롤바 트랙 배경 | `#00000000` | `#00000000` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `scrollbar.track.border` | 스크롤바 트랙 왼쪽 구분선 | `#2e333cff` | `#181825ff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |

### T-05 · 편집기·미리보기 및 커서/선택 토큰 (`editor.*`, `players[]`)
Zed는 로컬 사용자의 텍스트 선택 영역과 커서 색상을 `editor.*`가 아닌 **`style.players[0]`(첫 번째 플레이어 슬롯)** 에서 가져온다 (출처: https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/players.rs).

| 토큰 이름 | 의미 및 우리 범위 적용 영역 | One Dark 기본값 | 스크린샷(`zed-reference.png`) 측정/매핑값 | 출처 |
| --- | --- | --- | --- | --- |
| `editor.background` | 코드 편집기 및 마크다운 미리보기 본문 캔버스 배경 | `#282c33ff` | `#1e1e2eff` (실측 `(1500,500)`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `editor.foreground` | 코드 편집기 기본 텍스트 색 | `#acb2beff` | `#cdd6f4ff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `editor.gutter.background` | 줄 번호가 표시되는 거터(gutter) 배경 | `#282c33ff` | `#1e1e2eff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `editor.line_number` | 일반 줄 번호 색 | `#4e5a5f75` | `#6c7086ff` (추정) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `editor.active_line_number` | 현재 커서가 위치한 줄의 번호 색 | `#d0d4daff` | `#cdd6f4ff` / `#b4befeff` (추정) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `editor.active_line.background` | 현재 커서 줄 전체의 배경 하이라이트 | `#2f343ebf` | `#31324466` (추정) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `editor.subheader.background` | 편집기 내부 소제목 바 배경 | `#2f343eff` | `#181825ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `editor.Highlighted_line.background` | 이동/검색으로 강조된 줄 배경 | `#2f343eff` | `#313244ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `editor.invisible` | 공백·탭 기호 등 비가시 문자 표시 색 | `#525761ff` | `#45475aff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `editor.wrap_guide` | 줄 바꿈 안내선(80/100자 세로선) 기본 색 | `#c8ccd40d` | `#31324480` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `editor.active_wrap_guide` | 활성 줄 바꿈 안내선 색 | `#c8ccd41a` | `#45475a80` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `editor.document_highlight.read_background` | 커서 아래 식별자와 동일한 단어 자동 강조 배경 | `#74ade81a` | `#585b7040` (추정) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `editor.indent_guide` | 코드 편집기 내부 들여쓰기 세로 안내선 | `#c8ccd40d` | `#313244ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `editor.indent_guide_active` | 현재 블록의 활성 들여쓰기 세로 안내선 | `#c8ccd433` | `#585b70ff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/colors.rs |
| `players[0].cursor` | 로컬 사용자의 편집기 텍스트 커서(캐럿) 색 | `#74ade8ff` | `#f5e0dcff` (Rosewater, Catppuccin Mocha 기본값) | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/players.rs |
| `players[0].selection` | 로컬 사용자의 텍스트 드래그 선택 영역 배경색 | `#74ade83d` | `#585b7066` / `#9399b240` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/players.rs |
| `players[0].background` | 플레이어 배지 배경 | `#74ade8ff` | `#f5e0dcff` | https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/players.rs |

### T-06 · 구문 강조 토큰 (`style.syntax.*`)
`style.syntax` 객체는 Tree-sitter 하이라이트 캡처 이름을 키로 가지며, 각 항목은 `{ "color": "#RRGGBBAA", "font_style": "normal" | "italic" | "oblique" | null, "font_weight": 100..900 | null }` 형식의 `HighlightStyle` 객체다 (출처: https://github.com/zed-industries/zed/blob/main/crates/theme/src/styles/syntax.rs , https://zed.dev/docs/themes).

| `syntax` 토큰 키 | 의미 (코드 구문 강조 및 마크다운 편집 시 역할) | One Dark (`one.json`) 색상 / 스타일 | Catppuccin Mocha 표준 매핑 색상 | 출처 |
| --- | --- | --- | --- | --- |
| `attribute` | Rust `#[derive(...)]`, HTML 속성 | `#74ade8ff` | `#f9e2afff` (Yellow) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `boolean` | `true`, `false` | `#bf956aff` | `#fab387ff` (Peach) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `comment` | 일반 주석 (`//`, `<!-- -->`) | `#5d636fff` (`italic`) | `#6c7086ff` (`italic`, Overlay0) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `comment.doc` | 문서화 주석 (`///`, `//!`) | `#878e98ff` (`italic`) | `#7f849cff` (`italic`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `constant` | 상수 (`const`, 대문자 식별자) | `#dfc184ff` | `#fab387ff` (Peach) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `constructor` | 생성자·열거형 배리언트 (`Some`, `Ok`) | `#73ade9ff` | `#74c7ecff` (Sapphire) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `embedded` | 문자열 보간·임베디드 코드 블록 | `#dce0e5ff` | `#cdd6f4ff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `emphasis` | 마크다운 기울임꼴 (`*italic*`) | `#74ade8ff` (`italic`) | `#eba0acff` (`italic`, Maroon) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `emphasis.strong` | 마크다운 굵은 글씨 (`**bold**`, 스크린샷 본문 `#cdd6f4` + `700`) | `#bf956aff` (`weight: 700`) | `#eba0acff` (`weight: 700`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `enum` | `enum` 타입명 | `#d07277ff` | `#f9e2afff` (Yellow) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `function` | 함수·메서드 이름 (`fn main`) | `#73ade9ff` | `#89b4faff` (Blue) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `hint` | 인레이 힌트·보조 표기 | `#5a6f89ff` (`weight: 700`) | `#6c7086ff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `keyword` | 예약어 (`fn`, `let`, `pub`, `impl`, `match`) | `#b477cfff` | `#cba6f7ff` (Mauve) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `label` | 루프 레이블·마크다운 참조 레이블 | `#74ade8ff` | `#74c7ecff` (Sapphire) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `link_text` | 마크다운 링크 표시 텍스트 (`[text](url)`) | `#73ade9ff` (`normal`) | `#89b4faff` (Blue) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `link_uri` | 마크다운 링크 URL (`(https://...)`) | `#6eb4bfff` (`italic`) | `#b4befeff` (`italic`, Lavender) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `number` | 숫자 리터럴 | `#bf956aff` | `#fab387ff` (Peach) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `operator` | 연산자 (`+`, `=`, `->`, `::`) | `#6eb4bfff` | `#89dcebff` (Sky) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `predictive` | 자동 완성 고스트 텍스트 | `#5a6a87ff` (`italic`) | `#585b70ff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `preproc` | 전처리기·매크로 지시어 | `#dce0e5ff` | `#f5c2e7ff` (Pink) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `primary` | 기본 코드 식별자 | `#acb2beff` | `#cdd6f4ff` (Text) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `property` | 구조체 필드·JSON 키 (`tags`, `created`) | `#d07277ff` | `#b4befeff` / `#89b4faff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `punctuation` | 일반 구두점 (`,`, `;`, `.`) | `#acb2beff` | `#9399b2ff` (Overlay2) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `punctuation.bracket` | 괄호 (`()`, `{}`, `[]`) | `#b2b9c6ff` | `#9399b2ff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `punctuation.delimiter` | 구분자 (`:`, `,`) | `#b2b9c6ff` | `#9399b2ff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `punctuation.list_marker` | 마크다운 목록 기호 (`-`, `*`, `1.`) | `#d07277ff` | `#94e2d5ff` (Teal) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `punctuation.special` | 문자열 보간 괄호, 마크다운 인용 기호 (`>`) | `#b1574bff` | `#f5c2e7ff` (Pink) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `string` | 문자열 리터럴 | `#a1c181ff` | `#a6e3a1ff` (Green) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `string.escape` | 이스케이프 시퀀스 (`\n`, `\t`) | `#878e98ff` | `#f5c2e7ff` (Pink) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `string.regex` | 정규표현식 리터럴 | `#bf956aff` | `#fab387ff` (Peach) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `string.special` | 심볼·경로·특수 문자열 | `#bf956aff` | `#f2cdcdff` (Flamingo) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `string.special.symbol` | 루비/엘릭서 심볼 등 | `#bf956aff` | `#f2cdcdff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `tag` | HTML/XML 태그, 마크다운 태그 | `#74ade8ff` | `#cba6f7ff` / `#89b4faff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `text.literal` | 마크다운 인라인 코드 (`` `01-INSTALL.md` ``) 및 코드 블록 | `#a1c181ff` | `#a6e3a1ff` / 배경 `#313244ff` | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `title` | 마크다운 제목 (`# H1`, `## H2`) | `#d07277ff` (`weight: 700`) | `#cdd6f4ff` / `#89b4faff` (`weight: 700`) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `type` | 타입·트레이트·구조체 이름 (`String`, `Result`) | `#dfc184ff` | `#f9e2afff` (Yellow) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `variable` | 일반 변수명 | `#dce0e5ff` | `#cdd6f4ff` (Text) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `variable.special` | `self`, `super`, `this` | `#bf956aff` | `#f38ba8ff` (Red) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |
| `variant` | 열거형 값 | `#73ade9ff` | `#94e2d5ff` (Teal) | https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json |

---

## 2. 기본 테마 및 기준 스크린샷(`zed-reference.png`) 테마 분석 (`T-07` ~ `T-08`)

### T-07 · Zed 공식 기본 다크·라이트 테마
- **기본 설정(`assets/settings/default.json`)**:
  - 기본 다크 테마: **`"One Dark"`** (`appearance: "dark"`)
  - 기본 라이트 테마: **`"One Light"`** (`appearance: "light"`)
  - 기본 모드: `"mode": "system"` (OS 다크/라이트 설정에 따라 위 두 테마 중 선택) (출처: https://github.com/zed-industries/zed/blob/main/assets/settings/default.json)
- **파일 위치**: `assets/themes/one/one.json` (출처: https://github.com/zed-industries/zed/blob/main/assets/themes/one/one.json)
- **저작권자 및 라이선스**: `"author": "Zed Industries"`, Atom 편집기의 One Dark/Light를 기반으로 Zed Industries가 작성했으며 **`assets/themes/LICENSES`에 따라 MIT 라이선스**로 배포된다 (출처: https://github.com/zed-industries/zed/tree/main/assets/themes).

### T-08 · 기준 화면(`zed-reference.png`)의 테마 판별 (픽셀 실측 기반)
`D:\projects\docs\guides\zed-like-editor\01-zed\assets\zed-reference.png`(해상도 `2880×1704`)의 주요 영역 픽셀 RGB를 직접 추출한 결과, **Catppuccin Mocha** 팔레트와 16진수 코드가 100% 일치한다 (추정 근거가 명확한 실측 확인).

| 측정 영역 | 이미지 픽셀 좌표 (`2880×1704`) | 실측 RGB Hex | Catppuccin Mocha 공식 컬러 토큰 | One Dark(`one.json`)와 비교 |
| --- | --- | --- | --- | --- |
| 타이틀 바 (`title_bar.background`) | `(1000, 20)` | **`#11111b`** | `crust` (`#11111b`) | One Dark는 `#3b414d` |
| 상태 표시줄 (`status_bar.background`) | `(1500, 1680)` | **`#11111b`** | `crust` (`#11111b`) | One Dark는 `#3b414d` |
| 탭 바 배경 (`tab_bar.background`) | `(450, 55)` | **`#11111b`** | `crust` (`#11111b`) | One Dark는 `#2f343e` |
| 비활성 탭 (`tab.inactive_background`) | `(1250, 85)` | **`#0b0b11`** / **`#11111b`** | `crust` 어두운 변형 | One Dark는 `#2f343e` |
| 맨 왼쪽 사이드바 (`panel.background`) | `(100, 600)` | **`#181825`** | `mantle` (`#181825`) | One Dark는 `#2f343e` |
| 왼쪽 Dock 프로젝트 패널 (`panel.background`) | `(600, 600)` | **`#181825`** | `mantle` (`#181825`) | One Dark는 `#2f343e` |
| 편집기·미리보기 본문 (`editor.background`) | `(1500, 500)` | **`#1e1e2e`** | `base` (`#1e1e2e`) | One Dark는 `#282c33` |
| 활성 탭 (`tab.active_background`) | `(1500, 85)` | **`#1e1e2e`** | `base` (`#1e1e2e`) | One Dark는 `#282c33` |
| 사이드바 선택 항목 (`element.selected`) | `(100, 190)` | **`#3b3c4f`** | `surface0`/`surface1` 알파 블렌딩 | One Dark는 `#454a56` |
| 기본 본문 글자 (`text`) | `(50, 105)` | **`#cdd6f4`** | `text` (`#cdd6f4`) | One Dark는 `#dce0e5` |

- **Catppuccin 테마 저장소·라이선스**: 공식 확장 저장소 `catppuccin/zed`(또는 `zed-industries/zed` 초기 내장본 `assets/themes/catppuccin/`), 저작권자 **Catppuccin Org**, 라이선스 **MIT License** (출처: https://github.com/catppuccin/zed/blob/main/LICENSE).
- **구현 지침**: 우리 앱(`02-gpui`, `03-tauri`)은 기본 테마로 **Catppuccin Mocha**(기준 화면과 동일한 색상)와 **One Dark / One Light**(Zed 공식 기본 테마)를 모두 MIT 라이선스 하에 안전하게 내장할 수 있다 (`gpui-kit`에도 Catppuccin과 One 테마가 기본 포함되어 있음, 출처: https://github.com/longbridge/gpui-kit).

---

## 3. UI 아이콘 세트 출처 및 우리 범위 아이콘 목록 (`T-09` ~ `T-10`)

### T-09 · UI 아이콘 세트 출처와 관리 구조
- **파일 위치**:
  - SVG 원본 파일: `assets/icons/*.svg` 및 `assets/icons/file_icons/*.svg` (출처: https://github.com/zed-industries/zed/tree/main/assets/icons)
  - Rust 열거형 매핑: `crates/icons/src/icons.rs`의 `IconName` enum (PascalCase 배리언트가 snake_case 파일명과 1:1 대응) (출처: https://github.com/zed-industries/zed/blob/main/crates/icons/src/icons.rs)
- **아이콘 규격**: `16×16` viewBox, 외곽선(stroke) 두께 `1.2px` 기준 (출처: https://github.com/zed-industries/zed/blob/main/crates/icons/README.md).
- **출처 및 라이선스**: `crates/icons/README.md`에 따르면 Zed UI 아이콘의 대다수는 **Lucide Icons (`https://lucide.dev`, ISC 라이선스)** 에서 가져와 16×16 뷰박스와 1.2px 선 두께로 조정한 것이며, 일부는 Phosphor Icons(MIT) 참조 또는 Zed 자체 제작이다 (출처: https://github.com/zed-industries/zed/blob/main/crates/icons/README.md).

### T-10 · 우리 범위 화면에서 쓰는 아이콘 상세 목록
| 화면 영역 | 용도 (기준 화면 표기) | Zed `IconName` (`crates/icons/src/icons.rs`) | Zed SVG 파일 경로 (`assets/icons/`) | 동등한 Lucide 원본 아이콘 이름 | 라이선스 구분 | `gpui-kit` (`0.8.1`) 대응 `IconName` |
| --- | --- | --- | --- | --- | --- | --- |
| **타이틀 바** | 왼쪽 햄버거 앱 메뉴 (`☰`) | `IconName::Menu` | `assets/icons/menu.svg` | `menu` | Lucide (ISC) | `IconName::Menu` |
| **타이틀 바** | 창 최소화 (`-`) | `IconName::Dash` / OS 캡션 버튼 | `assets/icons/dash.svg` | `minus` | Lucide (ISC) | `IconName::Minus` |
| **타이틀 바** | 창 최대화 (`□`) | `IconName::Maximize` / OS 캡션 | `assets/icons/maximize.svg` | `square` / `maximize-2` | Lucide (ISC) | `IconName::Maximize` |
| **타이틀 바** | 창 닫기 (`×`) | `IconName::Close` | `assets/icons/close.svg` | `x` | Lucide (ISC) | `IconName::Close` |
| **사이드바 / 상태바** | 작업 폴더 추가 (폴더 `+` 아이콘) | `IconName::FolderPlus` / `NewFolder` (추정) | `assets/icons/folder_plus.svg` (또는 `plus.svg`) | `folder-plus` | Lucide (ISC) | `IconName::FolderPlus` / `Plus` |
| **왼쪽 Dock (트리)** | 닫힌 폴더 아이콘 | `IconName::Folder` | `assets/icons/file_icons/folder.svg` | `folder` | Lucide (ISC) 기반 | `IconName::Folder` |
| **왼쪽 Dock (트리)** | 열린 폴더 아이콘 (`projects`, `rust-gui`) | `IconName::FolderOpen` | `assets/icons/file_icons/folder_open.svg` | `folder-open` | Lucide (ISC) 기반 | `IconName::FolderOpen` |
| **왼쪽 Dock (트리)** | 일반 파일 기본 아이콘 | `IconName::File` / `FileGeneric` | `assets/icons/file_icons/file.svg` (`page.svg`) | `file` | Lucide (ISC) 기반 | `IconName::File` |
| **왼쪽 Dock (트리)** | 마크다운 파일 아이콘 (`[M↓]` `01-INSTALL.md` 등) | `IconName::Markdown` (파일 아이콘 매핑) | `assets/icons/file_icons/markdown.svg` | (언어 아이콘, Lucide 외 별도) | Zed/아이콘 테마 (MIT/CC0 확인 필요, 아래 4절 참조) | 커스텀 SVG 등록 권장 |
| **왼쪽 Dock (트리)** | `README.md` 정보 아이콘 (`ⓘ`) | `IconName::Info` (파일 아이콘 매핑) | `assets/icons/file_icons/info.svg` (`assets/icons/info.svg`) | `info` | Lucide (ISC) | `IconName::Info` |
| **탭 바** | 뒤로 이동 (`←`) | `IconName::ArrowLeft` | `assets/icons/arrow_left.svg` | `arrow-left` | Lucide (ISC) | `IconName::ArrowLeft` |
| **탭 바** | 앞으로 이동 (`→`) | `IconName::ArrowRight` | `assets/icons/arrow_right.svg` | `arrow-right` | Lucide (ISC) | `IconName::ArrowRight` |
| **탭 바** | 마크다운 미리보기 탭 아이콘 (열린 책 `Preview README.md`) | `IconName::Book` (또는 `BookOpen`) | `assets/icons/book.svg` | `book-open` / `book` | Lucide (ISC) | `IconName::BookOpen` |
| **탭 바** | 탭 닫기 (`×`, 호버 시 표시) | `IconName::Close` | `assets/icons/close.svg` | `x` | Lucide (ISC) | `IconName::Close` |
| **탭 바 우측** | 새 파일/탭 추가 (`+`) | `IconName::Plus` | `assets/icons/plus.svg` | `plus` | Lucide (ISC) | `IconName::Plus` |
| **탭 바 우측** | 패널 분할 (세로 분할선 있는 사각형) | `IconName::Split` | `assets/icons/split.svg` | `columns-2` / `square-split-horizontal` | Lucide (ISC) | `IconName::SplitVertical` / `PanelRight` |
| **탭 바 우측** | 패널 최대화/복원 (대각선 양방향 화살표 `⤢`) | `IconName::Maximize` / `ExpandVertical` | `assets/icons/maximize.svg` | `maximize-2` | Lucide (ISC) | `IconName::Maximize` / `Minimize` |
| **상태 표시줄** | 맨 왼쪽 사이드바 토글 (사이드바 영역 표시 사각형) | `IconName::PanelLeft` / `Sidebar` (추정) | `assets/icons/panel_left.svg` (추정) | `panel-left` | Lucide (ISC) | `IconName::PanelLeft` |
| **상태 표시줄** | 프로젝트 패널(왼쪽 Dock) 토글 (계층 트리 모양 아이콘) | `IconName::FileTree` | `assets/icons/file_tree.svg` | `list-tree` / `folder-tree` | Lucide (ISC) | `IconName::FolderTree` / `ListTree` |

출처: https://github.com/zed-industries/zed/blob/main/crates/icons/src/icons.rs , https://github.com/zed-industries/zed/blob/main/crates/icons/README.md , https://docs.rs/gpui-component/0.8.1/gpui_component/enum.IconName.html

---

## 4. 파일 아이콘 매핑 및 아이콘 테마 구조 (`T-11` ~ `T-13`)

### T-11 · 아이콘 테마(Icon Theme) 개념과 기본값
- **아이콘 테마 개념**: Zed는 색상 테마(`theme`)와 별도로 프로젝트 패널과 탭의 파일/폴더 아이콘 세트를 교체할 수 있는 **아이콘 테마(`icon_theme`)** 시스템을 지원한다 (출처: https://zed.dev/docs/icon-themes).
- **기본 아이콘 테마 설정(`assets/settings/default.json`)**:
  ```json
  "icon_theme": {
    "mode": "system",
    "light": "Zed (Default)",
    "dark": "Zed (Default)"
  }
  ```
  기본 아이콘 테마 이름은 **`"Zed (Default)"`** 이다 (출처: https://github.com/zed-industries/zed/blob/main/assets/settings/default.json).
- **기준 화면(`zed-reference.png`)의 파일 아이콘**:
  - `01-INSTALL.md` ~ `08-COMPARISON.md` 옆에 청록색 `[M↓]` 아이콘, `README.md` 옆에 청록색 `(i)` 아이콘, `workbench-kit` 폴더에 파란색, `workflow` 폴더에 주황색 배지가 표시되어 있다.
  - 파일명이 `README.md`일 때 일반 마크다운 아이콘 대신 **`info`(`ⓘ`) 아이콘을 우선 매핑**하는 것은 Zed의 기본 `file_stems` 규칙(`"README.md": "info"`)과 **Catppuccin Icons** 확장 모두에서 사용하는 대표 규칙이다 (색상이 입혀진 폴더/파일 아이콘은 **Catppuccin Icons** 확장 또는 컬러 아이콘 테마 적용 상태로 추정, 출처: https://zed.dev/docs/icon-themes).

### T-12 · 확장자·파일명 → 아이콘 매핑 JSON 형식
- **스키마 URL**: `https://zed.dev/schema/icon_themes/v0.3.0.json` (출처: https://zed.dev/docs/icon-themes)
- **Zed 내장 파일 위치**: `assets/icon_themes/zed-default.json` (및 레거시/내부 매핑 `assets/icons/file_icons/file_types.json`) (출처: https://github.com/zed-industries/zed/tree/main/assets/icons/file_icons).
- **매핑 우선순위 규칙**:
  1. **폴더인 경우**: `named_directory_icons[폴더이름]` 일치 시 해당 아이콘(`collapsed`/`expanded`) 사용 → 없으면 기본 `directory_icons`(`collapsed`/`expanded`) 사용.
  2. **파일인 경우**: `file_stems[정확한파일명]`(예: `"README.md"` → `"info"`, `"Cargo.toml"` → `"rust"`) 최우선 확인 → 없으면 `file_suffixes[확장자]`(예: `"md"` → `"markdown"`, `"rs"` → `"rust"`, `"json"` → `"json"`) 확인 → 없으면 기본 파일 아이콘(`file` / `default`) 사용 (출처: https://zed.dev/docs/icon-themes).

| JSON 키 (`themes[]` 내부) | 역할 | 우리 앱 필수 매핑 예시 | 출처 |
| --- | --- | --- | --- |
| `directory_icons` | 기본 폴더 닫힘/열림 SVG 경로 | `{ "collapsed": "icons/folder.svg", "expanded": "icons/folder_open.svg" }` | https://zed.dev/docs/icon-themes |
| `named_directory_icons` | 특정 폴더명 전용 아이콘 (선택) | `{ ".git": { ... } }` (우리 앱은 기본 폴더 아이콘만 써도 충분) | https://zed.dev/docs/icon-themes |
| `chevron_icons` | 트리 접기/펼치기 화살표 아이콘 | `{ "collapsed": "icons/chevron_right.svg", "expanded": "icons/chevron_down.svg" }` | https://zed.dev/docs/icon-themes |
| `file_stems` | 파일 전체 이름(또는 stem) → 아이콘 키 매핑 (`file_suffixes`보다 우선) | `{ "README": "info", "README.md": "info", "Cargo.toml": "toml", "Cargo.lock": "lock" }` | https://zed.dev/docs/icon-themes |
| `file_suffixes` | 파일 확장자(점 제외) → 아이콘 키 매핑 | `{ "md": "markdown", "markdown": "markdown", "rs": "rust", "toml": "toml", "json": "json", "js": "javascript", "ts": "typescript", "css": "css", "html": "html" }` | https://zed.dev/docs/icon-themes |
| `file_icons` | 아이콘 키 → SVG 경로 및 색상 정의 | `{ "markdown": { "path": "icons/markdown.svg" }, "info": { "path": "icons/info.svg" }, "rust": { "path": "icons/rust.svg" } }` | https://zed.dev/docs/icon-themes |

---

## 5. 글꼴 설정·번들 글꼴·Windows 대체 글꼴 (`T-14` ~ `T-16`)

### T-14 · Zed 기본 UI 글꼴·버퍼 글꼴 설정과 크기
`assets/settings/default.json` 및 `crates/theme/src/default_theme.rs`에 정의된 기본 타이포그래피 설정은 다음과 같다 (출처: https://github.com/zed-industries/zed/blob/main/assets/settings/default.json , https://zed.dev/docs/configuring-zed).

| 설정 키 | 기본값 (`default.json`) | 실제 매핑되는 번들 글꼴 이름 | 논리 px 크기 (배율 100%) | 용도 | 출처 |
| --- | --- | --- | --- | --- | --- |
| `ui_font_family` | `".ZedSans"` (또는 `".SystemUIFont"`) | **IBM Plex Sans** (Zed 커스텀/번들 산세리프) | 기본 **`16px`** (`1rem` 기준 단위) | 타이틀 바, 사이드바, 프로젝트 트리, 탭, 상태 표시줄, 마크다운 미리보기 본문 | https://zed.dev/docs/configuring-zed |
| `ui_font_size` | `16` | — | **`16px`** (트리·탭 등은 `rems` 비율로 12~14px 렌더링) | UI 요소 전체의 기준(`rem`) 크기 결정 | https://zed.dev/docs/configuring-zed |
| `buffer_font_family` | `".ZedMono"` | **Lilex** (IBM Plex Mono 기반 개발자용 고정폭 글꼴) | 기본 **`15px`** | 코드 편집기 버퍼, 마크다운 원본 편집기, 마크다운 미리보기의 코드 블록 | https://zed.dev/docs/configuring-zed |
| `buffer_font_size` | `15` | — | **`15px`** | 편집기 텍스트 기본 크기 | https://zed.dev/docs/configuring-zed |
| `buffer_line_height` | `"comfortable"` | 배수 약 `1.618` (황금비, `"standard"`는 `1.3`) | 약 **`24.3px`** (`15px × 1.618`) | 편집기 줄 간격 | https://zed.dev/docs/configuring-zed |

### T-15 · 번들된 글꼴 파일 위치와 라이선스
| 내부 별칭 | 글꼴 패밀리 | Zed 저장소 경로 | 저작권자 | 라이선스 | 출처 |
| --- | --- | --- | --- | --- | --- |
| `.ZedSans` | **IBM Plex Sans** (`Zed Plex Sans`) | `assets/fonts/plex-sans/` (또는 `zed-sans`) | IBM Corp. | **SIL Open Font License 1.1 (OFL-1.1)** | https://github.com/zed-industries/zed/tree/main/assets/fonts |
| `.ZedMono` | **Lilex** (및 `IBM Plex Mono` / `Zed Plex Mono`) | `assets/fonts/lilex/` (및 `assets/fonts/plex-mono/`) | Mikhael Khrustik (Lilex) / IBM Corp. | **SIL Open Font License 1.1 (OFL-1.1)** | https://github.com/mishamyrt/Lilex , https://github.com/zed-industries/zed/tree/main/assets/fonts |

### T-16 · Windows 11 대체(Fallback) 글꼴 구성
별도 글꼴 파일을 앱에 번들하지 않거나 한글(CJK) 글리프를 렌더링할 때 Windows 11 DirectWrite가 사용하는 권장 폴백 체인은 다음과 같다 (출처: https://zed.dev/docs/configuring-zed 및 Windows 11 표준 글꼴 기준).
- **UI 글꼴 체인 (`ui_font_family`)**: `"IBM Plex Sans"` (번들 시) → **`"Segoe UI"`** (Windows 11 기본 영문 UI 글꼴) → **`"Malgun Gothic"` (`맑은 고딕`)** (Windows 11 기본 한글 UI 글꼴).
- **버퍼/코드 글꼴 체인 (`buffer_font_family`)**: `"Lilex"` 또는 `"JetBrains Mono"` (번들 시) → **`"Cascadia Mono"` / `"Consolas"`** (Windows 11 기본 내장 고정폭 글꼴) → **`"Malgun Gothic"`** (한글 폴백).

---

## 6. 라이선스 요약표 (`T-17`)

우리 가이드 문서(`02-gpui`, `03-tauri`)와 예제 앱에서 가져올 자산 후보별 라이선스 및 준수 조건 요약표다.

| 자산 구분 | 구체적 대상 및 출처 | 라이선스 | 앱에 그대로 가져와도 되는가? | 필요한 고지 및 준수 조건 | 출처 |
| --- | --- | --- | --- | --- | --- |
| **테마 JSON (Zed 기본)** | `zed-industries/zed`의 `assets/themes/one/one.json` 등 내장 테마 JSON | **MIT License** | **가능 (예)** | 배포 시 `NOTICE` 또는 테마 폴더의 `LICENSE`에 Zed Industries 저작권 및 MIT 허가문 포함 (`crates/theme` Rust 코드는 GPL-3.0이므로 복사 금지, JSON 데이터만 사용) | https://github.com/zed-industries/zed/tree/main/assets/themes |
| **테마 JSON (Catppuccin)** | `catppuccin/zed` (기준 화면의 **Catppuccin Mocha** 팔레트) | **MIT License** | **가능 (예)** | Catppuccin Org 저작권 및 MIT 라이선스 고지 포함 (`gpui-kit` 내장 테마를 그대로 써도 됨) | https://github.com/catppuccin/zed/blob/main/LICENSE |
| **UI 아이콘 (Lucide)** | `lucide-icons/lucide` 원본 또는 `gpui-kit-assets` 내장 아이콘 (`menu`, `folder`, `folder-open`, `file`, `info`, `book-open`, `arrow-left/right`, `plus`, `columns-2`, `maximize-2`, `panel-left`, `folder-tree`) | **ISC License** (Lucide) / **Apache-2.0·MIT** (`gpui-kit-assets`) | **가능 (예)** | Lucide 원본 또는 `gpui-kit-assets`에서 가져올 경우 ISC/MIT 저작권 고지만 포함하면 상용·독점·오픈소스 모두 자유롭게 사용 가능 | https://lucide.dev/license , https://crates.io/crates/gpui-kit-assets |
| **파일 아이콘 (마크다운 `[M↓]` 및 언어 아이콘)** | `assets/icons/file_icons/markdown.svg` 또는 `catppuccin/vscode-icons` / `lucide` / `devicon` | Zed 저장소 개별 SVG는 라이선스 혼재 가능 / **Catppuccin Icons**·**Devicon**·**VSCode Icons**는 **MIT** | **원본 MIT 아이콘 세트에서 가져오면 가능 (예)** | Zed 저장소의 `assets/icons/file_icons/`를 무분별하게 일괄 복사하기보다, `info.svg`는 Lucide(`ISC`)를 쓰고 `markdown.svg`·언어 아이콘은 MIT가 명시된 아이콘 세트(예: Catppuccin Icons / Devicon / 자체 제작 SVG)를 사용 후 MIT 고지 첨부 | https://github.com/catppuccin/vscode-icons/blob/main/LICENSE |
| **번들 글꼴** | `IBM Plex Sans`, `Lilex` (`assets/fonts/`) 또는 Windows 11 내장 `Segoe UI` / `Cascadia Mono` / `맑은 고딕` | **SIL OFL 1.1** (번들 시) / Windows 시스템 폰트 (호출만 할 때 무료) | **가능 (예)** | 폰트 파일(`.ttf`)을 앱에 번들할 경우 SIL OFL 1.1 전문을 함께 배포하고 폰트 단독 판매 금지. 시스템 폰트(`Segoe UI`, `Cascadia Mono`)를 이름만 지정해 호출할 경우 별도 라이선스 파일 불필요 | https://openfontlicense.org/ , https://github.com/mishamyrt/Lilex/blob/main/LICENSE.md |

---

## 7. 확신 낮은 부분 (`T-18`)

1. **기준 화면(`zed-reference.png`)의 아이콘 테마 확장 여부**:
   - 색상 테마가 **Catppuccin Mocha**(`#11111b`, `#181825`, `#1e1e2e`, `#cdd6f4`)인 것은 이미지 픽셀 실측으로 100% 확인되었다.
   - 다만 왼쪽 프로젝트 패널의 파란색 폴더(`workbench-kit`), 주황색 배지 폴더(`workflow`), 청록색 `[M↓]`·`ⓘ` 아이콘이 Zed 기본 `"Zed (Default)"` 아이콘 테마에 테마의 `icon.accent` 색상이 입혀진 것인지, 아니면 **Catppuccin Icons** 확장(`catppuccin-icons`)이 함께 활성화된 것인지는 설정 파일을 직접 열지 않고 이미지 외형만으로는 추정에 해당한다 (`README.md` → `ⓘ`, `.md` → `[M↓]` 매핑 규칙 자체는 두 경우 모두 동일하다).
2. **상태 표시줄 맨 왼쪽 두 번째·세 번째 아이콘의 정확한 내부 액션명**:
   - 기준 화면 하단 상태 표시줄 좌측의 `[사이드바 토글]` 옆 동그라미 아이콘과 `[폴더+]` 모양 아이콘 중 일부는 Zed 최신 버전의 스레드/에이전트 워크스페이스 버튼일 가능성이 있다. 그러나 `_common.md` 지시에 따라 우리 앱에서는 이를 각각 **사이드바 토글(`panel-left`)**, **폴더 추가(`folder-plus`)**, **프로젝트 패널 토글(`list-tree`/`file_tree`)** 3개 버튼으로 명확히 고정해 구현하면 된다.

---

## 보충: 테마·아이콘 테마 선택 (`T-19` ~ `T-23`)

> 이 절은 새 요건 7(Zed 기본 테마/아이콘과 Catppuccin 테마/아이콘 중 선택 기능)을 위해 로컬에 설치된 확장 파일(`C:\Users\nampl\AppData\Local\Zed\extensions\installed\catppuccin` `0.2.27`, `catppuccin-icons` `1.24.0`, 둘 다 MIT 라이선스)과 Zed 공식 문서를 직접 조사해 덧붙인 명세다.

### T-19 · Catppuccin 테마 파일 구조 (`catppuccin` 확장 `0.2.27`)
- **설치 경로 및 매니페스트**: `C:\Users\nampl\AppData\Local\Zed\extensions\installed\catppuccin\extension.toml` (원본 저장소: https://github.com/catppuccin/zed , 라이선스: MIT, Copyright (c) 2021 Catppuccin).
- **포함된 테마 JSON 파일 2종과 내부 테마 목록**:
  1. `themes/catppuccin-mauve.json` (총 2,975줄, `$schema: "https://zed.dev/schema/themes/v0.2.0.json"`):
     - `"Catppuccin Latte"` (`"appearance": "light"`)
     - `"Catppuccin Frappé"` (`"appearance": "dark"`)
     - `"Catppuccin Macchiato"` (`"appearance": "dark"`)
     - `"Catppuccin Mocha"` (`"appearance": "dark"`)
  2. `themes/catppuccin-no-italics-mauve.json` (총 2,975줄):
     - `"Catppuccin Latte - No Italics"` (`"appearance": "light"`)
     - `"Catppuccin Frappé - No Italics"` (`"appearance": "dark"`)
     - `"Catppuccin Macchiato - No Italics"` (`"appearance": "dark"`)
     - `"Catppuccin Mocha - No Italics"` (`"appearance": "dark"`)
- **4가지 Flavor별 주요 표면·글자·강조 색상 실측표 (`catppuccin-mauve.json`)**:

| 토큰 키 | Catppuccin Latte (`light`) | Catppuccin Frappé (`dark`) | Catppuccin Macchiato (`dark`) | Catppuccin Mocha (`dark`) |
| --- | --- | --- | --- | --- |
| `background` | `#d7dce5` | `#383c52` | `#2c2f46` | `#27273b` |
| `title_bar.background` / `status_bar.background` / `tab_bar.background` | `#dce0e8` (Crust) | `#232634` (Crust) | `#181926` (Crust) | `#11111b` (Crust) |
| `panel.background` / `surface.background` | `#e6e9ef` (Mantle) | `#292c3c` (Mantle) | `#1e2030` (Mantle) | `#181825` (Mantle) |
| `editor.background` / `tab.active_background` / `toolbar.background` | `#eff1f5` (Base) | `#303446` (Base) | `#24273a` (Base) | `#1e1e2e` (Base) |
| `tab.inactive_background` | `#d2d7e2` | `#1d202b` | `#12121c` | `#0b0b11` |
| `text` / `editor.foreground` | `#4c4f69` | `#c6d0f5` | `#cad3f5` | `#cdd6f4` |
| `text.muted` | `#5c5f77` | `#b5bfe2` | `#b8c0e0` | `#bac2de` |
| `text.accent` / `icon.accent` / `border.selected` | `#8839ef` (Mauve) | `#ca9ee6` (Mauve) | `#c6a0f6` (Mauve) | `#cba6f7` (Mauve) |
| `border` | `#ccd0da` | `#414559` | `#363a4f` | `#313244` |
| `border.focused` | `#7287fd` (Lavender) | `#babbf1` (Lavender) | `#b7bdf8` (Lavender) | `#b4befe` (Lavender) |
| `element.hover` / `element.selected` | `#ccd0da` / `#ccd0da4d` | `#414559` / `#4145594d` | `#363a4f` / `#363a4f4d` | `#313244` / `#3132444d` |
| `players[0].cursor` / `players[0].selection` | `#dc8a78` / `#7c7f934d` | `#f2d5cf` / `#949cbb4d` | `#f4dbd6` / `#939ab74d` | `#f5e0dc` / `#9399b24d` |

- **Accent(`mauve`) 변형과 `no-italics` 변형의 차이**:
  - **Accent(`mauve`)의 의미**: Catppuccin은 14가지 파스텔 액센트 색상을 가지며, 공식 확장에 기본 번들된 `catppuccin-mauve.json`은 시그니처 색상인 **`mauve`(보라)** 를 `text.accent`, `icon.accent`, `border.selected`, `editor.active_line_number`, `panel.indent_guide_hover`, `minimap.thumb.*`에 매핑한 버전이다 (나머지 13개 액센트 변형은 GitHub Releases에서 개별 JSON으로 제공, 출처: https://github.com/catppuccin/zed).
  - **`no-italics`와의 차이**: 두 파일의 176개 UI 스타일 키와 102개 `syntax` 색상 값은 100% 동일하다. 유일한 차이는 `catppuccin-mauve.json`에서 `"font_style": "italic"`으로 지정된 22개 구문 강조 키(`variable.special`, `module`, `string.special.url`, `tag.attribute`, `type.builtin`, `type.interface`, `type.super`, `comment`, `comment.doc`, `comment.documentation`, `comment.info`, `comment.error`, `comment.warning`, `comment.warn`, `comment.hint`, `comment.todo`, `comment.note`, `namespace`, `emphasis`, `hint`, `link_uri`, `string.doc`)가 `catppuccin-no-italics-mauve.json`에서는 모두 `"font_style": null`로 설정되어 기울임꼴을 사용하지 않는다는 점이다 (출처: 로컬 `themes/catppuccin-mauve.json` 대 `themes/catppuccin-no-italics-mauve.json` 비교).
- **Zed 기본 테마(`One`)와의 토큰 집합 비교**:
  - 동일한 `v0.2.0` 스키마를 따르므로 1절(`T-02`~`T-06`)에 정리한 모든 핵심 UI·편집기·구문 토큰 이름이 정확히 호환된다.
  - 단, `catppuccin-mauve.json`(스타일 키 176개, `syntax` 키 102개)은 `one.json`보다 확장 토큰(`accents` 7색 배열, `vim.*` / `vim.helix_*`, `minimap.thumb.*`, `debugger.accent`, `version_control.conflict_marker.*`, 세분화된 Tree-sitter `syntax` 하위 키 `variable.member`, `comment.todo` 등)을 더 많이 정의하고 있다.

### T-20 · 아이콘 테마 JSON 형식 및 Catppuccin Icons (`1.24.0`) 분석
- **설치 경로 및 스키마**: `C:\Users\nampl\AppData\Local\Zed\extensions\installed\catppuccin-icons\icon_themes\catppuccin-icons.json` (총 37,799줄, `$schema: "https://zed.dev/schema/icon_themes/v0.2.0.json"`, 원본 저장소: https://github.com/catppuccin/zed-icons , 라이선스: MIT).
- **Catppuccin Icons의 8개 테마(4 Flavor × 일반/Monochrome) 및 아이콘 파일 수**:
  - `icons/` 디렉터리 아래 8개 폴더가 있으며, **각 폴더마다 정확히 656개의 SVG 파일**(전체 5,248개 SVG)이 들어 있다.
  - `icon_themes/catppuccin-icons.json`의 `themes` 배열에는 아래 8개 아이콘 테마가 정의되어 있다:

| 번호 | 아이콘 테마 `name` | `appearance` | SVG 폴더 경로 (`./icons/...`) | 파일 수 | 특징 |
| --- | --- | --- | --- | --- | --- |
| 1 | `"Catppuccin Latte"` | `"light"` | `./icons/latte/` | 656개 | 라이트 파스텔 다색 아이콘 (**기준 화면 `zed-reference.png`에서 사용 중인 설정**) |
| 2 | `"Catppuccin Frappé"` | `"dark"` | `./icons/frappe/` | 656개 | 중간 다크 파스텔 다색 아이콘 |
| 3 | `"Catppuccin Macchiato"` | `"dark"` | `./icons/macchiato/` | 656개 | 짙은 다크 파스텔 다색 아이콘 |
| 4 | `"Catppuccin Mocha"` | `"dark"` | `./icons/mocha/` | 656개 | 가장 밝은 대비의 Mocha 파스텔 다색 아이콘 |
| 5 | `"Catppuccin Latte Monochrome"` | `"light"` | `./icons/latte-monochrome/` | 656개 | Latte 단색(모노크롬) 변형 (`#5c5f77` 계열 단일 톤) |
| 6 | `"Catppuccin Frappé Monochrome"` | `"dark"` | `./icons/frappe-monochrome/` | 656개 | Frappé 단색 변형 |
| 7 | `"Catppuccin Macchiato Monochrome"` | `"dark"` | `./icons/macchiato-monochrome/` | 656개 | Macchiato 단색 변형 |
| 8 | `"Catppuccin Mocha Monochrome"` | `"dark"` | `./icons/mocha-monochrome/` | 656개 | Mocha 단색 변형 |

- **각 테마 객체 내부의 매핑 구조 (`catppuccin-icons.json` 실측)**:
  1. `directory_icons`: 기본 닫힌 폴더 `"collapsed": "./icons/<flavor>/_folder.svg"`, 열린 폴더 `"expanded": "./icons/<flavor>/_folder_open.svg"`. (폴더 내에 기본 파일용 `_file.svg`, 루트용 `_root.svg`, `_root_open.svg`도 함께 존재)
  2. `named_directory_icons`: 약 **420개**의 특수 폴더 이름을 전용 폴더 아이콘에 매핑한다.
     - 예: `"docs"` → `folder_docs.svg` / `folder_docs_open.svg`, `"workflow"` → `folder_workflow.svg` / `folder_workflow_open.svg`, `"assets"` → `folder_assets.svg`, `".cargo"` → `folder_cargo.svg`.
     - **기준 화면(`zed-reference.png`) 설명**: 왼쪽 프로젝트 트리에서 `docs \ guides`와 `workflow` 폴더 아이콘에 특수 배지가 그려져 있던 것은 바로 이 `named_directory_icons` 매핑 때문이다.
  3. `chevron_icons`: `{ "collapsed": null, "expanded": null }` (트리 접기/펼치기 화살표를 그리지 않고 폴더 아이콘 모양으로만 열림/닫힘을 표현).
  4. `file_stems` (정확히 **10개**):
     - `"LICENSE": "license"`, `"README": "readme"`, `"README.md": "readme"`, `"README.txt": "readme"`, `"Makefile": "makefile"`, `"Caddyfile": "caddy"`, `".DS_Store": "macos"`, `"CMakeLists.txt": "cmake"`, `"Cargo.lock": "cargo-lock"`, `"Cargo.toml": "cargo"`.
  5. `file_suffixes` (약 **1,810개**):
     - 단일 확장자(`"md": "markdown"`, `"rs": "rust"`, `"toml": "toml"`, `"json": "json"`)뿐 아니라 점이 포함된 복합 접미사(`"component.ts": "angular-component"`, `"androidmanifest.xml": "android"`, `".adonisrc.json": "adonis"`)도 키로 포함한다.
  6. `file_icons` (약 **400개** 아이콘 키):
     - 각 키가 `{ "path": "./icons/<flavor>/<key>.svg" }` 객체를 가리킨다 (예: `"markdown"` → `./icons/latte/markdown.svg`, `"readme"` → `./icons/latte/readme.svg`).
- **`"Zed (Default)"` 기본 아이콘 테마와의 차이점 요약**:

| 비교 항목 | `"Zed (Default)"` (Zed 내장 기본값) | `"Catppuccin Icons"` (확장 `1.24.0`) |
| --- | --- | --- |
| **색상 방식** | 단색 SVG에 현재 테마의 `icon` / `icon.muted` 색상을 런타임 주입 | SVG 파일 내부에 flavor별 파스텔 색상(`stroke`/`fill`)이 직접 지정됨 (Monochrome 변형만 단색) |
| **아이콘 규모** | 기본 폴더·화살표·주요 언어 수십 종 | Flavor당 **656개** SVG (약 400개 파일 타입 + 250여 개 특수 폴더 열림/닫힘) |
| **특수 폴더 (`named_directory_icons`)** | 미사용 (모든 폴더가 동일한 기본 폴더 아이콘) | `docs`, `workflow`, `src`, `assets` 등 420여 개 폴더명별 전용 배지 폴더 아이콘 표시 |
| **`README.md` 매핑** | `file_stems`에서 `"info"` 키(`info.svg`, `ⓘ`)로 매핑 | `file_stems`에서 `"readme"` 키(`readme.svg`, 원형 `ⓘ` 아이콘)로 매핑 |
| **트리 화살표 (`chevron_icons`)** | 기본 화살표(`chevron_right` / `chevron_down`) 사용 | `collapsed: null, expanded: null`로 화살표 생략 |

### T-21 · Zed의 테마·아이콘 테마 선택 UI 동작 명세
출처: https://zed.dev/docs/themes , https://zed.dev/docs/icon-themes , https://github.com/zed-industries/zed/tree/main/crates/theme_selector

| 항목 | 테마 선택기 (Theme Selector) | 아이콘 테마 선택기 (Icon Theme Selector) |
| --- | --- | --- |
| **실행 액션 식별자** | `theme_selector::Toggle` | `icon_theme_selector::Toggle` |
| **기본 키 바인딩** | `ctrl-k ctrl-t` (Windows/Linux), `cmd-k cmd-t` (macOS) | 기본 단축키 없음 (명령 팔레트 `ctrl-shift-p`에서 `icon theme selector: toggle` 검색 실행) |
| **메뉴 진입 경로** | 타이틀 바 좌측 `☰` 메뉴 → `Theme` 또는 명령 팔레트 | 명령 팔레트 (`ctrl-shift-p`) |
| **목록 UI 형태** | 편집기 상단 중앙에 뜨는 모달 `Picker` (상단 퍼지 검색 입력창 + 하단 테마 리스트, 현재 활성 테마에 초기 선택 위치) | 동일한 모달 `Picker` 구조 (설치된 아이콘 테마 리스트 표시) |
| **실시간 미리 적용 (Live Preview)** | **지원 (예)** — `↑`/`↓` 방향키로 목록 항목을 이동하거나 마우스로 가리키면 `Enter`를 누르기 전이라도 창 전체에 해당 테마가 즉시 렌더링된다 | **지원 (예)** — `↑`/`↓` 이동 즉시 프로젝트 패널과 탭 바의 모든 폴더·파일 아이콘이 해당 아이콘 테마로 실시간 교체된다 |
| **취소 시 되돌림 (Revert on Cancel)** | **지원 (예)** — `Escape`를 누르거나 모달 바깥을 클릭해 닫으면, 선택기 열기 직전의 원래 테마로 즉시 되돌아간다 | **지원 (예)** — `Escape` 또는 외부 클릭 시 원래 아이콘 테마로 즉시 되돌아간다 |
| **확정 및 저장 (`Enter`)** | `Enter` 입력 시 선택이 확정되고 사용자 `settings.json`(`%APPDATA%\Zed\settings.json`)의 `"theme"` 설정이 디스크에 저장된다 | `Enter` 입력 시 사용자 `settings.json`의 `"icon_theme"` 설정이 디스크에 저장된다 |

- **`settings.json` 저장 형식과 `mode: system / light / dark` 동작**:
  - `"theme"`와 `"icon_theme"` 모두 **단일 문자열** 또는 **`mode` 객체** 두 가지 형태를 가질 수 있다 (출처: https://zed.dev/docs/themes , https://zed.dev/docs/icon-themes):
    ```json
    {
      "theme": {
        "mode": "system",
        "light": "One Light",
        "dark": "Catppuccin Mocha"
      },
      "icon_theme": "Catppuccin Latte"
    }
    ```
  - **동작 규칙**:
    1. 값이 **단일 문자열**(예: `"icon_theme": "Catppuccin Latte"`)이면 OS 다크/라이트 모드와 무관하게 항상 해당 테마를 고정 사용한다. 선택기에서 새 항목을 `Enter`로 고르면 이 문자열 값이 새 이름으로 덮어써진다.
    2. 값이 **객체(`{ mode, light, dark }`)** 이면:
       - `mode: "system"`: Windows 11 시스템 앱 모드가 다크일 때는 `dark` 필드의 테마(`"Catppuccin Mocha"`), 라이트일 때는 `light` 필드의 테마(`"One Light"`)를 활성화한다. 선택기에서 테마를 변경해 `Enter`를 누르면 현재 활성 모드에 해당하는 하위 키(`dark` 또는 `light`)가 업데이트된다.
       - `mode: "dark"` 또는 `"light"`: OS 설정과 무관하게 각각 `dark` 또는 `light` 필드에 지정된 테마를 사용한다.

### T-22 · 우리 앱(`02-gpui`, `03-tauri`)에 옮길 때 필요한 구현 규칙
1. **번들할 선택 후보 목록 (모두 MIT 라이선스)**:
   - **색상 테마 (6종)**: `"One Dark"` (`dark`), `"One Light"` (`light`), `"Catppuccin Latte"` (`light`), `"Catppuccin Frappé"` (`dark`), `"Catppuccin Macchiato"` (`dark`), `"Catppuccin Mocha"` (`dark`). (기본값: `light = "One Light"`, `dark = "Catppuccin Mocha"`)
   - **아이콘 테마 (2~5종)**: `"Zed (Default)"`, `"Catppuccin Latte"`, `"Catppuccin Frappé"`, `"Catppuccin Macchiato"`, `"Catppuccin Mocha"`. (기본값: `"Catppuccin Latte"` 또는 `"Catppuccin Mocha"`)
2. **테마 JSON 로드 및 토큰 적용 순서**:
   - ① 앱 시작 시 테마 JSON(`one.json`, `catppuccin-mauve.json`)을 파싱해 `themes[]` 배열의 각 테마를 `name` 키로 레지스트리에 등록한다.
   - ② 설정 파일의 `theme` 값(`mode` + `light`/`dark` 또는 단일 문자열)을 읽고, `mode == "system"`이면 Windows 다크 모드 여부를 확인해 활성 테마 이름을 결정한다.
   - ③ 선택기 모달을 열 때 현재 테마 이름을 `previous_theme`에 백업하고, 목록에서 `↑`/`↓` 포커스가 바뀔 때마다 메모리상의 활성 테마만 즉시 교체해 화면을 다시 그린다(Live Preview). `Escape` 시 `previous_theme`으로 복원하고, `Enter` 시 설정 파일에 기록한다.
3. **아이콘 테마에서 폴더·파일 아이콘을 결정하는 알고리즘 (우선순위 규칙)**:
   - **폴더(`is_dir == true`)**:
     1. 폴더 이름(소문자 변환 `name.to_lowercase()`)이 현재 아이콘 테마의 `named_directory_icons`에 존재하면, 열림 상태(`is_expanded`)에 따라 `expanded` 또는 `collapsed` SVG 경로를 반환한다.
     2. 존재하지 않으면 `directory_icons.expanded`(`_folder_open.svg` / `folder_open.svg`) 또는 `directory_icons.collapsed`(`_folder.svg` / `folder.svg`)를 반환한다.
   - **파일(`is_dir == false`) — 우선순위: `정확한 파일명(file_stems) > 복합 확장자 > 단일 확장자(file_suffixes) > 기본 아이콘`**:
     1. **1순위 (`file_stems`)**: 파일 전체 이름(`file_name`, 예: `"README.md"`, `"Cargo.toml"`, `"Cargo.lock"`, `"LICENSE"`)이 `file_stems`에 있으면 해당 아이콘 키를 얻는다. 없으면 확장자를 뺀 스템(예: `"README"`)으로 한 번 더 `file_stems`를 조회한다.
     2. **2순위 (`file_suffixes`)**: 파일 이름에서 첫 번째 `.` 이후의 복합 접미사(예: `"component.ts"`)부터 마지막 `.` 이후의 단일 확장자(예: `"md"`, `"rs"`, `"toml"`, `"json"`, 소문자 기준) 순서로 `file_suffixes` 맵을 조회해 일치하는 아이콘 키를 얻는다.
     3. **3순위 (`file_icons` 조회 및 기본 폴백)**: 1·2순위에서 얻은 아이콘 키로 `file_icons[key].path`를 찾는다. 매칭된 키가 없거나 경로가 없으면 기본 파일 아이콘(`_file.svg` 또는 `file.svg`)을 반환한다.

### T-23 · 보충 확신 낮은 부분
- **우리 예제 앱에 포함할 Catppuccin Icons SVG 서브셋 범위**:
  - 원본 `catppuccin-icons` 확장에는 flavor당 656개(8개 테마 합계 5,248개)의 SVG와 37,799줄짜리 JSON이 들어 있다. 가이드 문서의 예제 앱에 5,248개 SVG를 모두 복사할지, 아니면 파서·우선순위 알고리즘(`named_directory_icons` → `file_stems` → `file_suffixes` → 기본)은 그대로 구현하되 실제 자주 쓰는 핵심 아이콘 서브셋(또는 선택된 1~2개 flavor 폴더 전체)만 번들할지는 `01-zed/04-APP-SPEC.md` 작성 단계에서 결정이 필요하다 (**판단**).
