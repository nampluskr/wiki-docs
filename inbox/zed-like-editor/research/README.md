---
tags: [zed, research, archive]
created: 2026-10-31
updated: 2026-10-31
---

# research · 조사 원자료 보관

> 이 파일을 읽는 때: 01-zed 명세의 값·동작이 어디서 왔는지 원자료를 확인할 때, 별도 프로젝트에서 더 깊이 조사할 때.

입구는 [../README.md](../README.md)다.

이 폴더는 문서 묶음을 만들며 Herdr 멀티에이전트(scouter·writer·checker)가 남긴 **작업 원자료**다. 고치지 않고 그대로 보관한다.
- **기준은 `01-zed/` 명세다.** 원자료와 명세가 다르면 명세가 맞다. 검토·수정을 거치며 바로잡힌 내용이 원자료에는 옛 값으로 남아 있을 수 있다.
- Zed 소스 근거는 모두 커밋 `14b7b0ef3764d62a85070c8b5ccea105a2ad3235` 기준이다. GPL 코드 원문은 없고 사실(값·이름·동작)만 있다.
- 파일 안의 스크래치패드 경로(`C:\Users\...\scratchpad\zed-research\...`)는 작업 당시 위치다. 지금은 이 폴더로 옮겨졌다.

## 1. sources · 조사 원자료

| 파일 | 담당 | ID | 내용 |
| --- | --- | --- | --- |
| [zed-layout.md](sources/zed-layout.md) | scouter-opus | L- | 레이아웃·치수 |
| [zed-interaction.md](sources/zed-interaction.md) | scouter-sol | I- | 키맵·탭·트리·포커스·상태 복원 |
| [zed-theme-icons.md](sources/zed-theme-icons.md) | scouter-agy | T- | 테마 토큰·아이콘·글꼴·Catppuccin·테마 선택 |
| [zed-tooltips.md](sources/zed-tooltips.md) | scouter-opus | L-2xx | 툴팁 |
| [zed-app-menu.md](sources/zed-app-menu.md) | scouter-sol | I-3xx~45x | ☰ 앱 메뉴 |
| [zed-user-menu.md](sources/zed-user-menu.md) | scouter-sol | I-6xx | 타이틀 바 ∨ 메뉴 |
| [zed-context-menus.md](sources/zed-context-menus.md) | scouter-opus | I-5xx (명세에서 일부 I-9xx 로 변경) | 우클릭 메뉴 |
| [zed-panes-keymap.md](sources/zed-panes-keymap.md) | scouter-sol | I-7xx | 탭 분할·이동, VSCode keymap 덧씌움 |
| [zed-editor.md](sources/zed-editor.md) | scouter-opus | E- | 원문 편집기·찾기 바·저장 |
| [zed-pickers-dialogs.md](sources/zed-pickers-dialogs.md) | scouter-agy | P- | 선택 창·대화상자·인라인 입력·글자 상태색 |
| [zed-keymap-editor.md](sources/zed-keymap-editor.md) | scouter-opus | K- | 키맵 편집 화면 |
| [zed-extensions.md](sources/zed-extensions.md) | scouter-agy2 | X- | Zed 확장 시스템 |
| [zed-ext-followup.md](sources/zed-ext-followup.md) | scouter-agy | X-100~ | 언어 우선 규칙, 테마 로드 중 동작 |
| [impl-gpui.md](sources/impl-gpui.md) | scouter-opus | — | 02-gpui 출발 문서 재료 |
| [impl-tauri.md](sources/impl-tauri.md) | scouter-agy2 | — | 03-tauri 출발 문서 재료 |

## 2. reviews · 검토·확인·수정 기록

| 파일 | 내용 |
| --- | --- |
| [review-gaps-opus.md](reviews/review-gaps-opus.md), [review-gaps-sol.md](reviews/review-gaps-sol.md), [review-gaps-agy.md](reviews/review-gaps-agy.md) | 01-zed 누락·구현 가능성 교차 검토 (agy 는 기준 화면 전수 대조) |
| [check-03-THEME-ICONS.md](reviews/check-03-THEME-ICONS.md), [check-04-APP-SPEC.md](reviews/check-04-APP-SPEC.md), [check-01-02.md](reviews/check-01-02.md) | checker 의 사실·출처·형식 확인 |
| [final-check-checker.md](reviews/final-check-checker.md), [final-review-opus.md](reviews/final-review-opus.md) | 최종 형식 확인, 최종 사실 리뷰(버전·라이선스 표본 27개) |
| [fix-log-gpui.md](reviews/fix-log-gpui.md), [fix-log-tauri.md](reviews/fix-log-tauri.md) | writer 가 반영하지 않은 지적과 이유 |
| [id-renumber.md](reviews/id-renumber.md) | 우클릭 메뉴 ID 재번호 대응표 (옛 → 새) |
| [agy-approvals.log](reviews/agy-approvals.log) | agy 승인 기록 (자동 승인·leader 승인, 명령 원문) |

## 3. instructions · 지시문과 결정

| 파일 | 내용 |
| --- | --- |
| [_decisions.md](instructions/_decisions.md) | **사용자 결정 전부** (포함·제외 기능, 확장, 뷰어, 편집기 등) |
| [_common.md](instructions/_common.md) | 모든 에이전트 공통 규칙, 추가 요건, ID 대역 |
| `_task-*.md`, `_write-*.md`, `_check.md`, `_review-gaps.md`, `_final-check.md` | 단계별로 에이전트에게 준 지시문 |

`_task-gpui-shell.md` 는 확장 방식을 Zed 와 같게 정하면서 **중단한 조사**의 지시문이다 (결과 파일 없음).
