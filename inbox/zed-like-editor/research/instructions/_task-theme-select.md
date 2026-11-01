# 보충 주제: 테마·아이콘 테마 선택 (ID 접두사 T-, 이어 번호) — 담당 scouter-agy

새 요건 7: 사용자가 Zed 기본 테마/아이콘과 Catppuccin 테마/아이콘 중에서 고를 수 있어야 한다.
기존 결과 파일 zed-theme-icons.md 끝에 "## 보충: 테마·아이콘 테마 선택" 절로 **덧붙인다** (기존 내용은 고치지 않는다).

이 노트북에 설치된 확장 (읽기만):
- C:\Users\nampl\AppData\Local\Zed\extensions\installed\catppuccin\ (Catppuccin 0.2.27, themes\catppuccin-mauve.json, catppuccin-no-italics-mauve.json)
- C:\Users\nampl\AppData\Local\Zed\extensions\installed\catppuccin-icons\ (Catppuccin Icons 1.24.0, icon_themes\catppuccin-icons.json, icons\<flavor>\...)
- 사용자 설정: theme = { mode: system, light: "One Light", dark: "Catppuccin Mocha" }, icon_theme = "Catppuccin Latte"
- 라이선스: 두 저장소 모두 MIT (Copyright (c) 2021 Catppuccin) — leader 가 확인함

조사할 것
1. Catppuccin 테마 파일 구조: 한 파일 안의 테마 목록(Latte·Frappé·Macchiato·Mocha 이름, appearance), accent(mauve) 변형과 no-italics 변형의 차이, Zed 기본 테마(One)와 토큰 집합이 같은지
2. 아이콘 테마 JSON 형식: Zed 의 icon theme 스키마(파일 확장자·파일명·폴더 열림/닫힘 → 아이콘 경로), Catppuccin Icons 의 flavor 별 구성, 몇 개의 아이콘, monochrome 변형. Zed 기본 아이콘 테마("Zed (Default)")와의 차이
3. Zed 의 선택 UI: theme selector / icon theme selector 를 여는 액션·키, 목록 표시 방식, 미리 적용(hover/이동 시 바로 바뀜) 여부, 취소 시 되돌림, settings.json 에 저장되는 키, mode: system/light/dark 동작
4. 우리 앱에 옮길 때 필요한 사실: 테마 JSON 을 읽어 토큰을 쓰는 순서, 아이콘 테마에서 파일 아이콘을 고르는 규칙(우선순위: 파일명 > 확장자 > 기본), 폴더 아이콘

규칙은 _common.md 와 같다. 끝나면 파일 경로만 답해.
