| 번호 | 심각도(높음/중간/낮음) | 파일·위치 | 문제 | 근거 | 고칠 방향 |
| --- | --- | --- | --- | --- | --- |
| 1 | 중간 | 03-THEME-ICONS.md 1.1 T-01 | One 색상은 항상 8자리라는 설명이 틀리다. | 고정 커밋의 [one.json 원본](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/themes/one/one.json) One Dark의 `editor.line_number`는 `#4e5a5f`, `editor.active_line_number`는 `#d0d4da`다. 문서 2.4절도 6자리로 기재한다. | 6자리와 8자리가 모두 사용된다고 고친다. |
| 2 | 중간 | 1.3·2.4 T-04 | 범위에서 제외한 찾기 기능과 검색 토큰 두 개를 명세에 포함한다. | `_common.md` 상태 표시줄 제외 항목의 검색 및 README 요건 5, 본문 머리의 범위 제한. | 검색 토큰과 조건부 구현 문구를 제거한다. |
| 3 | 중간 | 1.2 T-26, 2.4 T-04·T-05 | `—`인 모든 값을 1.2절 규칙으로 채운다고 하나, `scrollbar.thumb.border`, `scrollbar.thumb.active_background`, `pane.focused_border`, `editor.indent_guide` 등은 대체값이 정의되어 있지 않다. | 2.4절 137행과 1.2절의 여섯 규칙을 비교. | 사용하는 모든 누락 키에 대체값을 정하거나 미사용임을 명시한다. |
| 4 | 중간 | 4.2 T-28, 6.3 T-29 | 선택지에 `Zed (Default)`를 제공하지만 실제로는 여러 언어별 아이콘을 `file-code`·`file-text`로 대체한다. 기본 아이콘을 그대로 채용한다는 요건과 시각적 차이가 난다. | README 요건 6·7, 4.2절의 자체 대체 방침. | 자산별 라이선스를 확인해 원래 아이콘을 재현하거나 차이와 미해결 사항을 명확히 명세한다. |
| 5 | 중간 | 6.1–6.3 T-21·T-29 | 저장 시 mode 처리의 실제 동작은 조사하지 않았는데 우리 앱은 6.2절과 동일하게 동작하도록 정한다. | 6.2절 말미와 T-18.5는 `theme_settings::set_theme` 미확인 및 동일 규칙이라는 추정을 명시한다. | 저장 구현을 고정 커밋 소스에서 확인해 확정 규칙과 추정을 분리한다. |
| 6 | 낮음 | 1.1, 2.1–2.5, 4.1–4.5, 5.1–5.3 등 | 여러 표의 첫 열이 항목 ID가 아니다. | `_check.md` 형식 요건은 항목 ID 첫 열을 요구한다. | 표마다 첫 열에 T-ID를 넣거나 각 행을 고유 하위 ID로 연결한다. |
| 7 | 낮음 | 2.2 T-19, 4.4 T-20, 7 T-17 | Catppuccin 라이선스와 확장 자산을 저장소 루트 및 로컬 파일로만 지칭해 해당 버전의 재현 가능한 고정 출처가 없다. | `_common.md`의 사실별 URL, `_check.md`의 커밋 고정 출처 요건. | 설치 버전에 대응하는 태그 또는 커밋의 JSON과 LICENSE 직접 링크를 추가한다. |
| 8 | 낮음 | 3.1 T-09, 3.2 T-10 | Lucide 아이콘을 같은 모양이라고 단정하지만 실제 비교하지 않았다고 뒤에서 밝힌다. | 3.1절 방침과 3.2절 끝·T-18.3 간의 표현 차이. | 후보 대체 아이콘으로 표현하고 시각 비교 후 일치 여부를 명시한다. |
| 9 | 낮음 | 5.1–5.3 T-14~T-16 | 실제 글꼴 연결, 한글 글리프 부재, Windows 대체 순서 등 재현에 필요한 주장에 직접 근거가 부족하다. | 5.1절의 default.json은 설정명 근거이며 글꼴 매핑 근거가 아니고, 5.2절 한글 주장에 출처가 없다. | 고정 소스의 글꼴 등록과 글리프 검사 근거를 추가하고 대체 순서는 앱의 판단으로 구분한다. |
| 10 | 낮음 | 5.2 T-15 | 폰트 폴더 링크가 GitHub `blob` 경로여서 디렉터리 탐색 링크로 적절하지 않다. | 링크 대상은 `assets/fonts/ibm-plex-sans`, `assets/fonts/lilex` 디렉터리다. | 커밋 고정 `tree` 링크나 개별 파일 링크로 바꾼다. |

## 대조한 표본 목록(일치/불일치)

아래 One Dark 10개는 [고정 커밋 raw one.json](https://raw.githubusercontent.com/zed-industries/zed/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/themes/one/one.json)을 직접 열어 문서 2.4절과 대조했다.

| 표본 | 원본 값 | 결과 |
| --- | --- | --- |
| `background` | `#3b414dff` | 일치 |
| `border` | `#464b57ff` | 일치 |
| `text` | `#dce0e5ff` | 일치 |
| `editor.line_number` | `#4e5a5f` | 표 값 일치, 1.1절의 항상 8자리 설명 불일치 |
| `editor.active_line_number` | `#d0d4da` | 표 값 일치, 1.1절 설명 불일치 |
| `editor.invisible` | `#4e5a5f` | 표 값 일치, 1.1절 설명 불일치 |
| `panel.background` | `#2f343eff` | 일치 |
| `tab.active_background` | `#282c33ff` | 일치 |
| `warning` | `#dec184ff` | 일치 |
| `icon` | `#dce0e5ff` | 일치 |

로컬 `catppuccin-mauve.json`의 Latte·Frappé·Macchiato·Mocha 각 `text.accent`, `panel.indent_guide_hover`, `editor.line_number` 12개 값은 2.4절과 일치한다. frontmatter와 읽는 때 문구, 이전·다음·입구 링크의 대상 존재를 확인했다. 여러 줄 Rust 코드 원문은 발견하지 못했다.
