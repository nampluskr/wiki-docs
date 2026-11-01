# 2단계 작성 지시: 01-zed (Zed 분석 명세)

너는 Herdr 워크스페이스 w8 의 writer 다. leader 의 지시로 문서를 쓴다.

## 먼저 읽을 것
- 목표·요건·라이선스 원칙: D:\projects\docs\guides\zed-like-editor\README.md
- 공통 규칙(끝의 추가 요건·방식 포함): C:\Users\nampl\AppData\Local\Temp\claude\D--projects\b2c61f73-3611-43da-b80f-0f7d315ef143\scratchpad\zed-research\_common.md
- 기준 화면: D:\projects\docs\guides\zed-like-editor\01-zed\assets\zed-reference.png (직접 열어 본다)
- 형식 견본: D:\projects\docs\guides\rust-gui\02-GPUI.md

## 조사 자료 (scratchpad\zed-research\)
- zed-layout.md (L-, scouter-opus) / zed-interaction.md (I-, scouter-sol) / zed-theme-icons.md (T-, scouter-agy, 끝에 보충 T-19~T-23)
- leader 가 소스로 대조해 맞다고 확인한 것: 타이틀 바 32px(Windows), 탭 바 32/내용 31, IconSize 10/12/14/16/48, preview_tabs 기본, 키(Ctrl+B, Ctrl+Shift+E, Ctrl+Shift+V, Ctrl+K V, Ctrl+Shift+T, 트리 Enter·F2=Rename, Delete=Trash, Shift+Delete=Delete, Ctrl+N, Alt+N)
- 고칠 것: L-21 의 출처는 crates/ui/src/utils/constants.rs:15-24

## 결정 사항 (사용자)
- 방식: 포크가 아니라 reverse engineering 재구현. 명세는 "새 코드로 똑같이 다시 만들 수 있을 만큼" 구체적으로.
- 쓰지 않는 기능의 아이콘·메뉴·옵션은 넣지 않는다.
- 상태 표시줄: **우리 앱에 있는 기능의 아이콘만** 남긴다 → 왼쪽: 사이드바 토글, 폴더 추가, 프로젝트 패널 토글 / 오른쪽: 커서 위치. 기록·검색·진단·편집 예측·터미널·디버그·개요·협업·Git·AI 는 제외.
- 테마 선택: Zed 기본(One Dark·One Light, Zed 기본 아이콘) + Catppuccin(Latte·Frappé·Macchiato·Mocha) + Catppuccin Icons. 기준 화면 = Catppuccin Mocha + Catppuccin 아이콘.
- Catppuccin 토큰 색은 agy 의 "추정" 값을 쓰지 말고, 설치된 확장 JSON 에서 실제 값을 읽어 적는다:
  C:\Users\nampl\AppData\Local\Zed\extensions\installed\catppuccin\themes\catppuccin-mauve.json
  C:\Users\nampl\AppData\Local\Zed\extensions\installed\catppuccin-icons\icon_themes\catppuccin-icons.json
  (읽기만. 이 파일들은 MIT 라 값 인용 가능)

## 작성 규칙
- frontmatter(tags, created/updated 2026-10-31), 제목 "# 01-zed · NN · 제목", "> 이 파일을 읽는 때:" 한 줄, 머리 표(근거: Zed 커밋 14b7b0e, 검증: 소스 대조 / 실행 검증 안 함).
- 한국어, 짧은 문장, 표 위주. 모든 사실에 출처 URL (커밋 고정 링크 유지).
- **항목 ID**: 조사 자료의 ID(L-NN, I-NNN, T-NN)를 그대로 쓴다. 명세 표의 첫 열이 ID 다. 새 항목이 필요하면 같은 접두사로 이어 번호.
- 범위 밖 항목(터미널·Git·AI 등)은 쓰지 않는다. 조사 자료에 있어도 뺀다.
- Zed GPL 코드 원문을 옮기지 않는다. 값·이름·동작만.
- 기준 화면은 ![기준 화면](assets/zed-reference.png) 로 참조한다.
- 끝에 "이전 / 다음 / 입구" 링크. 입구는 ../README.md. 순서: ../README.md → 01-LAYOUT → 02-INTERACTION → 03-THEME-ICONS → 04-APP-SPEC → ../02-gpui/01-SETUP.md
- 쓰는 파일은 맡은 두 개뿐이다. 다른 파일은 고치지 않는다.

## 담당
### writer-gpui
1. 01-zed\01-LAYOUT.md: 단위 체계(rem·간격·밀도·글자/아이콘/버튼 크기) → 전체 배치(나무 구조 그림, 사이드바가 창 전체 높이) → 영역별 치수(타이틀 바·사이드바·Dock·프로젝트 패널·탭 바·편집기·상태 표시줄) → 마크다운 미리보기 스타일 → 기준 화면 채용/제외 표 → 관련 설정 기본값
2. 01-zed\02-INTERACTION.md: 키맵 표(범위 안) → 미리보기 탭·핀 탭 → 탭 동작 → 마크다운 미리보기 → 프로젝트 패널 조작 → 포커스 → 상태 복원

### writer-tauri
3. 01-zed\03-THEME-ICONS.md: 테마 JSON 구조와 토큰(범위 안) → 기본 테마 One Dark/Light 와 Catppuccin 4종(실제 값) → UI 아이콘 목록(Zed IconName·Lucide 이름·라이선스·gpui-kit 대응) → 파일 아이콘 테마 규칙(우선순위) → 글꼴 → 테마·아이콘 테마 선택 UI → 라이선스 고지 목록
4. 01-zed\04-APP-SPEC.md: 요건 1~7 ↔ 항목 ID 표 → 멀티 폴더 사이드바 명세(Zed threads 사이드바 자리, 등록·선택·삭제, 등록 해제와 디스크 삭제 구분) → 데이터 모델·설정 파일 형식(JSON 예: 등록 폴더, 활성 폴더, 열린 탭, Dock 폭, 테마, 아이콘 테마, 글꼴 크기) → 일치도 판정 기준(치수 ±1px, 키, 동작, 색 토큰) — 01~03 의 ID 를 참조. 01·02 는 writer-gpui 가 동시에 쓰고 있으니 조사 자료의 ID 기준으로 참조한다.

끝나면 쓴 파일 경로만 답해.
