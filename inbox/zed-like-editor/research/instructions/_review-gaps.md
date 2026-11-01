# 01-zed 누락·구현 가능성 검토 지시

너는 Herdr 워크스페이스 w8 의 scouter 다. 이번에는 **검토자**다. 문서를 읽기만 하고 고치지 않는다. 결과를 지정한 파일에 쓰고, 끝나면 그 파일 경로만 답해.

## 목적 (사용자 요청)
01-zed 에서 정리한 Zed 의 기능·화면·아이콘 등에 대해 아래 두 가지를 검토한다.
1. **누락**: 조사에서 빠진 항목이 있는가. 기준 화면(zed-reference.png)에 보이는데 문서에 없는 요소, 요건(README 1~7과 추가 요건)에 필요한데 명세가 없는 동작·상태·치수·색·아이콘·툴팁·메뉴 항목.
2. **구현 가능성**: 조사된 항목이 **새 코드로 똑같이 다시 만들 수 있을 만큼** 확인되었는가. 값이 "추정/미확인"이거나, 출처가 없거나, 상태(hover·active·focus·disabled·선택·드래그 중)가 빠졌거나, 치수가 계산식 없이 대략치만 있거나, 동작의 조건·예외가 빠진 항목.

## 읽을 것
- D:\projects\docs\guides\zed-like-editor\README.md (요건)
- D:\projects\docs\guides\zed-like-editor\01-zed\01-LAYOUT.md, 02-INTERACTION.md, 03-THEME-ICONS.md, 04-APP-SPEC.md
- 기준 화면: D:\projects\docs\guides\zed-like-editor\01-zed\assets\zed-reference.png (직접 본다)
- 공통 규칙: C:\Users\nampl\AppData\Local\Temp\claude\D--projects\b2c61f73-3611-43da-b80f-0f7d315ef143\scratchpad\zed-research\_common.md
- 필요하면 Zed 소스(커밋 14b7b0ef3764d62a85070c8b5ccea105a2ad3235)를 직접 열어 확인한다. GPL 코드 원문은 옮기지 않는다.
- 메뉴(☰·∨·우클릭) 절은 지금 채우는 중일 수 있다. 빈 자리라면 "채우는 중"으로만 적고 누락으로 세지 않는다.

## 출력 형식
### A. 누락 항목
| 번호 | 영역 | 빠진 것 | 왜 필요한가(요건·기준 화면 근거) | 확인할 곳(Zed 소스 파일·설정) | 우선순위(높음/중간/낮음) |
### B. 구현 가능성 부족 항목
| 번호 | 파일·ID | 문제(추정·미확인·상태 누락·조건 누락 등) | 구현에 필요한 정보 | 확인할 곳 | 우선순위 |
### C. 종합 판정
담당 범위가 "구현 가능 수준"인지 한 문단으로. 높음 우선순위 개수.

## 담당 (교차 검토: 자기가 조사하지 않은 범위를 본다)
- scouter-sol: 01-LAYOUT.md 와 03-THEME-ICONS.md → 저장: review-gaps-sol.md
- scouter-opus: 02-INTERACTION.md 와 04-APP-SPEC.md → 저장: review-gaps-opus.md
- scouter-agy: 기준 화면 전수 대조 — 화면에 보이는 모든 요소(아이콘·글자·선·간격·색 영역)를 목록으로 만들고, 01~04 에 대응 명세가 있는지 표시 → 저장: review-gaps-agy.md
(모두 같은 폴더 scratchpad\zed-research\ 에 저장)
