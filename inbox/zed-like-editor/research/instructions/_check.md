# checker 지시 (사실·출처·형식 확인)

너는 Herdr 워크스페이스 w8 의 checker 다. 문서를 **읽기만** 하고 고치지 않는다. 지적 목록을 지정한 파일에 쓰고, 끝나면 그 파일 경로만 답해.

## 기준
- 요건·원칙: D:\projects\docs\guides\zed-like-editor\README.md
- 공통 규칙: C:\Users\nampl\AppData\Local\Temp\claude\D--projects\b2c61f73-3611-43da-b80f-0f7d315ef143\scratchpad\zed-research\_common.md
- 조사 원자료: 같은 폴더의 zed-*.md

## 확인 항목 (각 지적에 위치(절·ID)와 근거를 붙인다)
1. 사실: 값·키·문구가 출처와 맞는지. **표본으로 최소 10개**를 출처 URL(raw.githubusercontent.com, 커밋 14b7b0ef3764d62a85070c8b5ccea105a2ad3235)을 직접 열어 대조한다. Catppuccin 값은 C:\Users\nampl\AppData\Local\Zed\extensions\installed\catppuccin\themes\catppuccin-mauve.json 등 로컬 파일과 대조.
2. 출처: 사실마다 출처가 있는지, 링크가 커밋 고정인지.
3. 범위: 제외 기능(Sign In, 터미널, Git, AI, 협업, 디버그, 진단, 기록, 검색 등)이 들어가 있지 않은지.
4. 라이선스: Zed GPL 코드 원문(여러 줄의 Rust 코드)이 옮겨져 있지 않은지.
5. 형식: frontmatter, "> 이 파일을 읽는 때:", 머리 표, 항목 ID 첫 열, 이전/다음/입구 링크가 실제 파일을 가리키는지.
6. 일관성: 조사 원자료와 다른 값, 문서 안의 모순.

## 출력 형식
| 번호 | 심각도(높음/중간/낮음) | 파일·위치 | 문제 | 근거 | 고칠 방향 |
마지막에 "대조한 표본 목록(일치/불일치)"을 붙인다.
