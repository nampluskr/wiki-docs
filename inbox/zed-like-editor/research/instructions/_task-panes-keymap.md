# 보충 주제: 탭 분할·이동(Pane) + VSCode 기본 keymap 적용 (ID 접두사 I-, I-700부터)

저장 파일: C:\Users\nampl\AppData\Local\Temp\claude\D--projects\b2c61f73-3611-43da-b80f-0f7d315ef143\scratchpad\zed-research\zed-panes-keymap.md
같은 커밋(14b7b0ef3764d62a85070c8b5ccea105a2ad3235) 기준. 규칙은 _common.md 와 같다 (GPL 코드 원문 금지).

## A. 사용자 요건: 편집기 탭 영역의 분할·이동을 Zed 와 동일하게
1. 분할: 분할 버튼(탭 바 오른쪽)·키·메뉴로 분할할 때 방향, 새 Pane 에 무엇이 열리는지(같은 항목 복제? 빈 Pane?), split_direction 관련 설정 기본값
2. 드래그 앤 드롭: 탭을 끌 때 드롭 영역(Pane 가운데/위/아래/왼쪽/오른쪽 가장자리)의 판정 규칙(가장자리 비율·px), 드롭 표시(오버레이 색 토큰·모양), 같은 Pane 안 순서 바꾸기, 다른 Pane 으로 옮기기, 가장자리에 떨어뜨려 새 분할 만들기, 핀 탭 경계 처리, 프로젝트 패널에서 파일을 끌어 Pane 에 놓을 때
3. Pane 그룹: 중첩 분할 구조(PaneGroup 축·비율), 분할선 크기·끌어서 크기 조절·최소 크기·더블클릭 시 균등화 여부, Pane 이 비면 닫히는지, 마지막 Pane 처리
4. 키보드: Pane 간 포커스 이동, 항목을 다른 Pane 으로 옮기기/복제, Pane 닫기, 분할, 줌(최대화) — **아래 B 의 적용 keymap 기준**으로
5. 활성 Pane 표시(포커스 테두리 등), 줌 상태 표시

## B. 사용자 요건: Zed 의 base_keymap 을 VSCode 로 설정했다. 우리 앱은 이 적용 keymap 을 따른다
- 사용자 설정: "base_keymap": "VSCode", 사용자 keymap.json 없음.
- leader 가 확인한 사실: Windows 에서는 default-windows.json 위에 assets/keymaps/linux/vscode.json 을 덧씌운다 (crates/settings/src/base_keymap_setting.rs asset_path, not(macos) 분기). 기본 설정 파일의 base_keymap 기본값은 "Zed".
1. keymaps/linux/vscode.json 의 모든 바인딩을 표로 (컨텍스트 / 키 / 액션). 원문 JSON 은 옮기지 말고 표로.
2. 덧씌우는 순서·우선순위 규칙: 나중에 로드된 바인딩이 이기는지, 같은 키 충돌 시 결과, 컨텍스트 우선순위, null 로 해제하는 바인딩
3. **우리 범위의 최종 적용 keymap 표**: zed-interaction.md 1절(기본 keymap)의 각 항목이 VSCode 덧씌움 후 어떻게 바뀌는지 (그대로 / 바뀜 / 추가). 메뉴·툴팁에 표시되는 단축키도 이 결과를 따르므로, zed-app-menu.md·zed-tooltips.md 에 적힌 단축키 중 바뀌는 것을 목록으로.

끝나면 파일 경로만 답해.
