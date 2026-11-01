| 번호 | 심각도(높음/중간/낮음) | 파일·위치 | 문제 | 근거 | 고칠 방향 |
| --- | --- | --- | --- | --- | --- |
| 1 | 높음 | 04-APP-SPEC.md 2절 A-05.1 | 사용자 메뉴 `∨`를 제외하고 창 버튼만 둔다고 해 필수 요건과 모순된다. | README 1절과 `_common.md` 추가 요건은 Sign In을 빼고 `∨` 메뉴를 채용하도록 명시한다. 01-LAYOUT L-89도 같은 결론이다. | `∨` 버튼과 사용 가능한 항목만 담은 메뉴를 명세·검증 기준에 포함한다. 작성 중인 01·02 메뉴 절의 빈자리는 지적하지 않는다. |
| 2 | 중간 | 4.2 A-21, 5절 | 모든 설정 이름·기본값이 Zed와 같고 프로젝트 패널 위치만 다르다고 하나 글꼴 기본값이 다르다. | [고정 커밋 raw default.json](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json)은 `ui_font_family: .ZedSans`, `buffer_font_family: .ZedMono`다. 문서 JSON은 각각 `IBM Plex Sans`, `Lilex`다. | Zed 설정 문자열과 실제 대응 글꼴을 구별하고 앱 자체 기본값임을 밝힌다. |
| 3 | 중간 | 4.2 A-21.7, 5절 | `sidebar.default_width: 300`을 Zed와 같은 설정 키·기본값으로 제시하나 원본 기본 설정에 `sidebar` 항목이 없다. | 위 default.json 원본과 A-21.7. | 자체 설정 키라고 표시하고 300의 실제 근거를 별도 제시한다. |
| 4 | 중간 | 7.5 A-34.4 | 화면 가장자리 밖으로 나가지 않는 것을 필수 통과 조건으로 단정하나 근거는 그 처리가 미확인이라고 밝힌다. | 03-THEME-ICONS T-30.5, T-18.2. | 앱의 별도 판단으로 표시하거나 Zed 동작을 확인한 후 비교 기준으로 정한다. |
| 5 | 중간 | 1절 A-04, 4.3 A-22, 8절 A-40.7 | 분할 버튼을 사용하도록 참조하고 검증하면서 구현 자체는 선택 사항으로 남겨 구현 범위를 확정하지 못한다. | A-04의 L-72, 4.3절 분할 조건, A-40.7. `_common.md`는 탭 바 분할을 채용한다. | 분할을 필수로 정하고 분할 상태·복원·조작을 완성한다. |
| 6 | 중간 | 3.5 A-14.1~2, 3.4 A-13.4 | 트리 루트에서 등록 해제를 제공한다고 하나 메뉴명·단축키와 3.4절 동작은 사이드바 항목만 기준으로 적혀 있어 루트의 실행 결과와 Delete 키의 범위가 불명확하다. | A-14.1은 트리 루트를 포함하지만 A-13.4는 사이드바 우클릭·포커스 후 Delete만 명시한다. | 루트 메뉴 동작, 포커스별 Delete 키, 활성 폴더 및 저장 질문 처리까지 구체적으로 연결한다. |
| 7 | 중간 | 4.3 A-22.3 | 모든 경로의 끝 역슬래시와 `\\?\` 접두사를 단순 제거하면 드라이브 루트 `C:\`가 `C:`가 되고 확장 UNC 경로도 잘못 바뀐다. | Windows 루트·확장 경로 형식과 A-22.3의 무조건 제거 규칙. | 드라이브 루트와 UNC를 보존하는 정규화·비교 규칙과 예를 명시한다. |
| 8 | 중간 | 4.5 A-24.1, A-24.4 | 임시 파일 이름 바꾸기만으로 Windows에서 기존 파일 덮어쓰기와 충돌 없는 복구를 보장하지 못한다. 기존 `state.json.bad`가 있을 때 처리도 없다. | A-24.1은 중단 시 옛 파일 보존을 단정하고 A-24.4는 고정 이름으로 이동하도록 한다. | 원자적 교체 API·실패 처리·기존 백업 충돌 규칙을 명세한다. |
| 9 | 중간 | 7.2 A-31.8, 03 1.2·2.4 | 사용하는 모든 토큰이 원본 JSON 값과 완전히 같아야 한다지만 One에 없는 토큰은 03에서 대체값을 사용하도록 한다. | 03 T-26 및 `panel.indent_guide`, `panel.focused_border`의 누락 표시. | 원본에 있는 값은 원본과, 없는 값은 합의한 대체 규칙과 비교한다. |
| 10 | 낮음 | 1절 A-05, 2절 A-05.5 | 존재하지 않는 `L-94`를 참조한다. | 현재 01-LAYOUT의 관련 항목은 L-93, L-95~L-98이며 L-94 정의와 언급이 없다. | 실제 관련 ID로 교체하거나 해당 정의를 복원한다. |
| 11 | 낮음 | 1절 A-04 | `T-27`을 단독 항목처럼 참조하나 03에는 T-27.1, T-27.2만 정의되어 있다. | 03 구문 강조 규칙 표와 T-18.1의 총칭. | 필요한 하위 ID를 명시한다. |
| 12 | 낮음 | 문서 끝 다음 링크 | `../02-gpui/01-SETUP.md`가 현재 존재하지 않는다. | 실제 경로 존재 검사 실패. | 파일 생성 후 링크를 확인하거나 작성 예정임을 명시한다. |
| 13 | 낮음 | 3.9 A-18, 5절, 6절 | 표의 첫 열이 항목 ID가 아니다. | `_check.md` 형식 요건은 항목 ID 첫 열을 요구한다. | A 하위 ID 열을 추가한다. |
| 14 | 낮음 | 3.7 A-16.4 | OS 폴더 선택 창이 시스템 폴더 선택을 막는다고 단정하나 출처가 없다. | 폴더 선택과 접근 권한은 별개이며 문서에 해당 OS 동작 근거가 없다. | 경로 검증·접근 실패 처리를 앱 책임으로 명세하고 근거 없는 단정을 뺀다. |
| 15 | 낮음 | 7.1 A-30.1~2 | 기준 이미지 배율은 미확인인데 200%를 확정값처럼 사용해 ±1 논리 px 판정의 기준으로 삼는다. | A-30.1은 타이틀 바 높이에서 역산한 배율이 미확인이라고 명시한다. | 배율을 확인하거나 200%를 가정으로 표시하고 판정 불확실성을 반영한다. |
| 16 | 낮음 | 7.2 A-31.9 | 아이콘 경로 선택 근거를 이 문서의 4.3절로 지칭하지만 4.3절은 state.json이다. | 아이콘 선택 규칙은 03-THEME-ICONS 4.3절 T-22다. | 03의 4.3절로 명시한다. |

## 대조한 표본 목록(일치/불일치)

[고정 커밋 raw default.json](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json)을 직접 열어 4.2절 예제와 대조했다.

| 표본 | 원본 | 결과 |
| --- | --- | --- |
| `theme.mode` | `system` | 일치 |
| `theme.light` | `One Light` | 일치 |
| `theme.dark` | `One Dark` | 일치 |
| `icon_theme` | `Zed (Default)` | 일치 |
| `ui_font_family` | `.ZedSans` | 불일치 |
| `ui_font_size` | 16 | 일치 |
| `buffer_font_family` | `.ZedMono` | 불일치 |
| `buffer_font_size` | 15 | 일치 |
| `buffer_line_height` | `comfortable` | 일치 |
| `project_panel.default_width` | 240 | 일치 |
| `project_panel.indent_size` | 20 | 일치 |
| `project_panel.auto_fold_dirs` | true | 일치 |
| `project_panel.folder_indicator` | `icon` | 일치 |
| `tabs.close_position` | `right` | 일치 |
| `tabs.show_close_button` | `hover` | 일치 |
| `tabs.file_icons` | false | 일치 |
| `preview_tabs.enabled` | true | 일치 |
| `preview_tabs.enable_preview_from_project_panel` | true | 일치 |
| `markdown_preview.limit_content_width` | true | 일치 |
| `markdown_preview.max_width` | 800 | 일치 |
| `sidebar.default_width` | 원본에 해당 `sidebar` 객체 없음 | 불일치 |

로컬 Catppuccin `catppuccin-mauve.json`에서 네 테마 이름과 밝기를 확인했다: Latte는 light, Frappé·Macchiato·Mocha는 dark로 일치한다. 01~03에 대한 문서의 명시적 ID 참조를 실제 표 첫 열·제목 정의와 대조한 결과 L-94와 총칭 T-27을 제외하고 참조 ID가 존재한다. 범위 표기의 중간 번호가 모두 필요한 항목이라는 뜻은 아니므로 범위 끝점과 명시된 ID를 검사했다. frontmatter, 읽는 때 문구, 이전·입구 링크를 확인했으며 다음 링크는 위와 같다. 여러 줄 Rust 원문은 발견하지 못했다.
