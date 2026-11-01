# 공통 지시 (1단계: Zed 분석 조사)

너는 Herdr 워크스페이스 w8 의 scouter 다. leader 의 지시로 Zed 를 reverse engineering 수준으로 조사한다.

## 목적
docs/guides/zed-like-editor/ 문서 묶음을 쓴다. Zed 의 기본 편집기 기능만 가진 앱을 gpui-kit(1순위)과 Tauri 2(2순위)로 만드는 문서다. 사용감·레이아웃이 Zed 와 비슷한 수준이어야 한다. 먼저 읽기: D:\projects\docs\guides\zed-like-editor\README.md

## 기준 화면
사용자가 쓰는 Zed 화면: D:\projects\docs\guides\zed-like-editor\01-zed\assets\zed-reference.png (이미지를 열어 볼 수 있으면 직접 본다)
최종 구현은 이 화면과 비슷해야 하고, 쓰지 않는 기능의 아이콘·메뉴·옵션은 넣지 않는다.

| 영역 | 채용 | 제외 |
|---|---|---|
| 타이틀 바 | ☰ 메뉴(우리 기능만), 현재 폴더 이름, 창 버튼 | Sign In |
| 맨 왼쪽 사이드바 (Zed 에서는 threads 사이드바) | 등록 폴더 목록: 헤더, 항목, 선택 강조, 폴더 추가 | 스레드 검색, 세션·시간 |
| 왼쪽 Dock (프로젝트 패널) | 루트, 접힌 경로(docs \ guides), 폴더·파일 아이콘, 들여쓰기 안내선, 가로 스크롤 | Git 상태 색·아이콘 |
| 탭 바 | ← → 이동, 편집 탭, 미리보기 탭(책 아이콘 "Preview X.md"), 넘침, 분할, 최대화 | 우리 기능이 아닌 메뉴 |
| 편집기·미리보기 | 마크다운 미리보기(frontmatter 표, 제목, 인용, 표, 링크), 코드 구문 강조 편집기 | — |
| 상태 표시줄 | 사이드바 토글, 폴더 추가, 프로젝트 패널 토글, 커서 위치 | 기록, 검색, 진단, 편집 예측, 터미널, 디버그, 개요, 협업, Git, AI |

## 출처와 라이선스 규칙
- 출처: Zed 공식 문서(zed.dev/docs), GitHub zed-industries/zed 소스(읽기만), 기본 설정·키맵 파일.
- Zed 앱 코드는 GPL-3.0 이다. **코드 원문을 옮기지 말고 사실만 우리 말로 요약**한다 (값, 이름, 동작). 짧은 식별자(함수·설정·액션·토큰 이름)는 써도 된다.
- 모든 사실 옆에 출처 URL(가능하면 파일 경로와 줄)을 붙인다. 확인 못 한 것은 "미확인", 추정은 "추정"이라고 표시한다.
- 치수는 논리 px(배율 100% 기준)로 적는다. 소스의 상수·rems 값과 계산식을 함께 적는다.

## 출력
- 결과를 지정한 Markdown 파일 하나에 저장하고, 화면에는 **파일 경로만** 답한다.
- 그 파일 외에는 아무것도 만들거나 고치지 않는다. 시스템 설정도 바꾸지 않는다.
- 각 항목에 임시 ID 를 붙인다 (주제 접두사 + 번호, 예: L-01). writer 가 이 ID 를 문서의 항목 ID 로 쓴다.
- 마지막 절은 "확신 낮은 부분" 으로 끝낸다.

## 추가 요건 (사용자, 조사 중 추가)
- 요건 7: 테마·아이콘 테마 선택. Zed 기본(One Dark·One Light, Zed 기본 아이콘)과 Catppuccin 테마(Latte·Frappé·Macchiato·Mocha)·Catppuccin Icons 중에서 고른다. 둘 다 MIT.
- 기준 화면 설정: theme mode=system (light One Light / dark Catppuccin Mocha), icon_theme=Catppuccin Latte, ui_font_size 16, buffer_font_size 15.
- 방식(사용자 확인): Zed 를 포크해 기능을 걷어내는 것이 아니다. Zed 와 거의 같은 수준으로 reverse engineering 해서 gpui-kit / Tauri 로 새로 구현한다. 그래서 명세는 '새 코드로 똑같이 다시 만들 수 있을 만큼' 구체적이어야 한다 (치수·색 토큰·동작 규칙·키).
- leader 대조 메모: L-21 타이틀 바 높이 값(Windows 32px)은 맞음. 출처는 crates/ui/src/utils/constants.rs:15-24 로 고칠 것.
- 요건(사용자 추가): 아이콘 위에 마우스를 올렸을 때 나오는 툴팁(hover 메시지)도 Zed 와 동일하게 한다 (문구, 단축키 표시, 지연, 위치, 스타일). 조사: zed-tooltips.md (scouter-opus).
- 요건(사용자 추가): 타이틀 바 ☰ 앱 메뉴는 Zed 와 같은 구조로 하되, 우리 앱에서 쓸 수 있는 항목만 남기고 나머지는 뺀다. 조사: zed-app-menu.md (scouter-sol).
- 요건(사용자 추가): 타이틀 바 오른쪽 Sign In 버튼은 제거, 옆의 '∨' 드롭다운(사용자 메뉴)은 채용. 메뉴 항목은 우리 앱에서 쓸 수 있는 것만. 조사: zed-user-menu.md (scouter-sol, 앱 메뉴 다음).
- 요건(사용자 추가): 아이콘과 각 영역의 우클릭 컨텍스트 메뉴도 Zed 와 동일하게 하되 쓸 수 없는 항목은 뺀다. 조사: zed-context-menus.md (scouter-opus).
- ID 대역: I-3xx~I-45x ☰ 앱 메뉴(zed-app-menu.md), I-500~ 우클릭 메뉴(zed-context-menus.md), I-600~ ∨ 메뉴(zed-user-menu.md), L-200~ 툴팁(zed-tooltips.md).
- 요건(사용자 추가): 편집기 탭 영역의 분할·이동(드래그, Pane 그룹, 키)도 Zed 와 동일하게.
- 요건(사용자 추가): keymap 은 사용자의 Zed 설정 base_keymap=VSCode 를 따른다. Windows 에서는 default-windows.json 위에 keymaps/linux/vscode.json 을 덧씌운 결과가 적용 keymap 이다. 메뉴·툴팁의 단축키 표시도 이 결과를 따른다. 조사: zed-panes-keymap.md (I-700~).
- 사용자 결정(포함/제외 목록): C:\Users\nampl\AppData\Local\Temp\claude\D--projects\b2c61f73-3611-43da-b80f-0f7d315ef143\scratchpad\zed-research\_decisions.md — 모든 문서의 기준.
- 기준 화면 2: D:\projects\docs\guides\zed-like-editor\01-zed\assets\zed-reference-latte.png (Catppuccin Latte 라이트 테마, 창 최대화 안 함, 2448x1480). 확인 가능: Latte 색, 최대화 아닐 때 창 버튼(□ 최대화) vs 화면1(⧉ 복원), 창 테두리·모서리, 상태 표시줄 커서 위치 '1:1', 이미지 파일·assets 폴더 아이콘, 비활성 탭 흐림, 깊은 트리 안내선.
