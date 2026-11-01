| 번호 | 심각도(높음/중간/낮음) | 파일·위치 | 문제 | 근거 | 고칠 방향 |
| --- | --- | --- | --- | --- | --- |
| 1 | 높음 | 02-INTERACTION 9.2 I-368·Go 메뉴 | 최종 메뉴에 Find, Replace, Command Palette, Go to File을 넣어 제외 범위 및 앱 명세와 충돌한다. | `_check.md`는 검색을 제외 기능으로 명시하고 `_common.md`도 쓰지 않는 기능의 메뉴를 빼도록 한다. 04-APP-SPEC A-05.6은 명령 팔레트·파일 찾기를 제외한다. | 해당 기능의 채택 범위를 확정하고 제외 기능과 메뉴 진입점을 일관되게 제거한다. |
| 2 | 높음 | 02-INTERACTION 10.2~10.5 I-620·I-621 | Settings와 Keymap을 구현하고 최종 메뉴에 넣는다고 확정하면서 같은 절에서 두 화면의 구현 여부가 결정 대기이며 최종 구조에 넣지 않았다고 쓴다. | 10.2 I-647 및 10.3은 두 항목 채용, 10.5는 화면 구현 여부 미결과 최종 구조 제외를 명시한다. 9.4 I-326·I-332도 미결이다. | 구현 여부와 두 메뉴 진입점을 하나의 결정으로 통일한다. |
| 3 | 중간 | 02-INTERACTION 10절과 11절 I-608~I-655 | 두 독립 조사에서 동일한 I-ID를 재사용해 참조가 모호하다. 예를 들어 I-610은 사용자 메뉴 열기·닫기와 편집기 Copy and Trim을 각각 가리킨다. | 10.1 I-610과 11.4 I-610; I-620은 Settings와 미리보기 Copy, I-647은 설정 범위와 Dock 메뉴를 각각 뜻한다. | 한 조사에 새 ID 범위를 할당하고 01·02 및 다른 문서의 참조를 함께 갱신한다. |
| 4 | 중간 | 02-INTERACTION 11.3·11.9 I-577 | Pin Tab을 구현 여부 결정 대기로 두고 탭 메뉴에서 제거하지만 앞부분과 앱 명세는 핀 탭을 명세·복원·검증한다. | 02 I-065~I-066, 04 A-04·A-22.13·A-31.4 및 11.9 I-577. | 핀 기능과 메뉴 진입점을 채택 여부에 맞춰 통일한다. |
| 5 | 중간 | 02-INTERACTION 11.2 I-662·I-555 | 루트 메뉴의 Remove from Project를 결정 대기로 빼지만 앱 명세는 루트에 Remove from Sidebar를 제공한다고 확정한다. | 04-APP-SPEC A-14.1 및 3.5절, 02 11.9 I-555. | 루트 등록 해제 동작과 문구·구분선을 일관되게 확정한다. |
| 6 | 중간 | 02-INTERACTION 11.6 I-634·I-635 | 폴더 Move Up/Down을 유지하고 Remove라고 표시하지만 앱 명세는 등록 순서 고정 및 Remove from Sidebar라고 정한다. | 04 A-13.4·A-13.6, 02 I-667 표. | 순서 변경 지원 및 삭제 메뉴 문구를 양쪽에서 일치시킨다. |
| 7 | 중간 | 02-INTERACTION 11.7 I-668 | 오른쪽 Dock이 없는데 최종 메뉴 예시에 Dock Right를 표시하고 실행 가능 여부는 나중에 결정하도록 한다. | 01 L-14는 왼쪽 Dock만, 02 11.7 및 11.9 I-647은 Dock Right가 미결임을 명시한다. | 지원하지 않으면 최종 메뉴에서 제거하고 지원 시 레이아웃·저장 동작을 명세한다. |
| 8 | 중간 | 02-INTERACTION 9.2 I-343·I-345·I-350 | New Window, Open File, Add Folder to Project 등을 최종 채택했으나 04의 단일 등록 폴더 모델 및 3.9 메뉴는 Open File을 제외하고 폴더 추가 동작을 다르게 정한다. | 04 A-18 및 3.1·3.9, 02 I-343·I-345·I-350. | 창·파일 열기·폴더 등록의 지원 범위와 실제 메뉴를 명세 전반에서 통일한다. |
| 9 | 낮음 | 01-LAYOUT 4.7 L-211, 03-THEME-ICONS T-30.6 | 툴팁 테두리를 01은 `border.variant`, 03은 `border`라고 적어 테마 적용 결과가 달라진다. | 01 L-211은 고정 소스 styled_ext.rs를 근거로, 03 T-30.6은 내부 미확인 판단으로 제시한다. | 해당 elevation 구현을 대조해 하나로 정정하고 관련 참조를 갱신한다. |
| 10 | 낮음 | 02-INTERACTION 11.6 I-634 | Move Up과 Move Down에 같은 ID가 배정되어 개별 참조가 불가능하다. | 11.6 항목표의 두 연속 행이 모두 I-634다. | 구분 가능한 하위 ID를 배정한다. |
| 11 | 낮음 | 01-LAYOUT 2절의 DynamicSpacing 표, 02-INTERACTION 9.2 메뉴 도식 | 별도 데이터 표 또는 메뉴 도식의 항목별 ID 연결이 빠진 곳이 있다. | `_check.md`는 항목 ID 첫 열을 요구한다. 메뉴 표 구분선 행도 첫 열에 `—`를 쓰고 ID는 마지막 열에 둔다. | 각 표의 첫 열에 ID를 넣고 해당되는 조사 ID를 연결한다. |
| 12 | 낮음 | 01-LAYOUT 4.8 및 02-INTERACTION 9.1 I-302 | 02의 정정 문구는 01 L-85가 여전히 계층형 팝업이라고 주장하지만 현재 L-85는 이미 가로 메뉴 바로 수정돼 있다. | 01 L-85와 02 I-302의 괄호 안 정정 설명 비교. | 오래된 정정 문구를 현재 내용에 맞게 바꾼다. |
| 13 | 낮음 | 01-LAYOUT 1절 | 기준 화면 아이콘 테마를 막연히 Catppuccin 계열이라고만 적어 기준 화면 재현에 필요한 flavor가 빠졌다. | README 1절은 Catppuccin Latte를 지정한다. | `Catppuccin Latte`로 명시한다. |
| 14 | 낮음 | 02-INTERACTION 9.4 I-368 | 프로젝트 검색을 사용자 결정 대기로 남겨 제외 범위를 다시 열어 둔다. | `_check.md`와 `_common.md`의 검색 제외, 02 11절에서도 프로젝트 검색을 제외한다. | 제외 판정으로 옮긴다. |

## 대조한 표본 목록(일치/불일치)

고정 커밋의 raw 원본을 직접 열어 확인했다. 단축키 표시는 요청에 따라 VSCode keymap 차이를 지적하지 않았다.

| 표본 | 원본 근거 | 결과 |
| --- | --- | --- |
| Windows 타이틀 바 높이 32px | [constants.rs](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/utils/constants.rs) | 일치 |
| Windows 외 높이 max(1.75rem, 34px) | 같은 constants.rs | 일치 |
| 툴팁 기본 지연 500ms | [div.rs](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/gpui/src/elements/div.rs) | 일치 |
| hoverable 툴팁 숨김 지연 500ms | 같은 div.rs | 일치 |
| 사용자 메뉴 아이콘 ChevronDown | [title_bar.rs](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/title_bar.rs) | 일치 |
| 사용자 메뉴 접근성 이름 User menu | 같은 title_bar.rs | 일치 |
| 사용자 메뉴 아이콘 Small | 같은 title_bar.rs | 일치 |
| 툴팁 제목과 키 가로 배치 | [tooltip.rs](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/tooltip.rs) | 일치 |
| 툴팁 설명 Small·Muted | 같은 tooltip.rs | 일치 |
| 툴팁 표면 elevation_2 | 같은 tooltip.rs | 일치 |
| 툴팁 바깥 왼쪽·위 여백 `pl_2`, `pt_2p5` | 같은 tooltip.rs | 일치 |
| 툴팁 내부 `py_1`, `px_2` | 같은 tooltip.rs | 일치 |
| 앱 메뉴의 1단계 하위 항목 평탄화 및 더 깊은 메뉴 제외 | [application_menu.rs](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/title_bar/src/application_menu.rs) | 일치 |
| UI 글꼴 크기 16, 버퍼 15, 줄 높이 comfortable | [default.json](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json) | 일치 |
| 프로젝트 패널 기본 폭 240, dock right, 들여쓰기 20 | 같은 default.json | 일치 |
| 미리보기 탭 두 기본 설정 true | 같은 default.json | 일치 |

로컬 `catppuccin-mauve.json`에서 Latte·Frappé·Macchiato·Mocha의 `panel.background`, `editor.background`, `text` 12개 값을 확인했다. 두 파일의 frontmatter·읽는 때 문구·머리 표와 이전/다음/입구 대상은 확인했다. 여러 줄 Rust 코드 원문은 발견하지 못했다.
