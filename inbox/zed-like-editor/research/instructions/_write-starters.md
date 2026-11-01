# 작성 지시: 출발 문서 02-gpui/README.md, 03-tauri/README.md (+ 04-COMPARISON.md)

사용자 결정: 구현은 별도 프로젝트에서 한다. 이 두 README 는 그 프로젝트의 **출발(참고) 문서**다. **전체 내용만** 담고, 구현 전에 할 **상세 조사·검토 항목을 목록**으로 남긴다. 코드·상세 구현 절차는 쓰지 않는다.
기준: README(D:\projects\docs\guides\zed-like-editor\README.md), 01-zed/01~04(항목 ID), _decisions.md, _common.md.
재료: scratchpad\zed-research\impl-gpui.md (scouter-opus), impl-tauri.md (scouter-agy2). 재료의 사실·URL 은 그대로 쓰되, 확인 안 된 것은 그대로 "미확인"·"판단"으로 둔다.
형식: frontmatter(tags, created/updated 2026-10-31), "# 02-gpui · gpui-kit 프로젝트 출발 문서" 같은 제목, "> 이 파일을 읽는 때:" 한 줄, 머리 표(기준 버전, 상태: 출발 문서), 한국어·짧은 문장·표 위주, 끝에 이전/다음/입구 링크.
분량: 각 250~450줄 정도.

## 두 README 공통 절 구성 (짝 규칙: 같은 절 번호·제목)
1. 이 문서의 역할 (별도 프로젝트의 출발점, 01-zed 가 명세 기준)
2. 전체 구조 (모듈·프로세스 그림, 상태·렌더링·파일 I/O·설정/키맵·확장 호스트 위치)
3. 기술 스택 (크레이트/라이브러리 후보·역할·버전·라이선스)
4. 01-zed 항목 ↔ 구성 요소 (영역별: 타이틀 바, 사이드바, Dock·트리, 탭·Pane, 편집기, 미리보기·내장 뷰어, 상태 표시줄, 메뉴(☰·∨·우클릭), 툴팁, Picker·대화상자, 키맵 편집, 테마·아이콘, 확장 관리) — 기본 제공 / 일부 / 직접 구현
5. **상세 조사·검토 항목** (핵심 절): 체크리스트 표 — ID(gpui: G-, tauri: R-) / 질문 / 왜 / 관련 01-zed ID / 확인 방법(PoC·문서·소스) / 우선순위. 01-zed 04 의 A-57.3(개발용 문법 빌드·링크)·A-57.7(Tree-sitter WASM 로더)도 여기 넣는다.
6. 위험 요소와 대응
7. 시작 순서 (PoC 단계 5~8개)
8. 참고 링크

## 담당
- writer-gpui → D:\projects\docs\guides\zed-like-editor\02-gpui\README.md (재료 impl-gpui.md)
  - 이전: ../01-zed/04-APP-SPEC.md · 다음: ../03-tauri/README.md · 입구: ../README.md
- writer-tauri → D:\projects\docs\guides\zed-like-editor\03-tauri\README.md (재료 impl-tauri.md)
  - 이전: ../02-gpui/README.md · 다음: ../04-COMPARISON.md · 입구: ../README.md
- 두 사람 모두 끝나면, writer-gpui 가 D:\projects\docs\guides\zed-like-editor\04-COMPARISON.md 를 쓴다 (leader 가 따로 지시).

끝나면 쓴 파일 경로만 답해.
