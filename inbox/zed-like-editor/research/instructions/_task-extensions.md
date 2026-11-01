# 조사: Zed 확장 시스템 (ID 접두사 X-) — 담당 scouter-agy2

사용자 결정: 우리 앱의 확장 방식은 **Zed 와 동일**하게 한다. 지원 범위는 테마, 아이콘 테마, 언어(Tree-sitter 문법·구문 강조)다. LSP·디버거·MCP·AI 슬래시 명령은 제외 (_decisions.md 의 '확장 방식' 절).
저장 파일: C:\Users\nampl\AppData\Local\Temp\claude\D--projects\b2c61f73-3611-43da-b80f-0f7d315ef143\scratchpad\zed-research\zed-extensions.md
같은 커밋(14b7b0ef3764d62a85070c8b5ccea105a2ad3235) 기준 + zed.dev/docs/extensions. 규칙은 _common.md 와 같다 (GPL 원문 금지, 값·이름·동작만, 출처 URL).
참고: 이 노트북에 설치된 확장 C:\Users\nampl\AppData\Local\Zed\extensions\installed\ (catppuccin, catppuccin-icons, html) — 읽기만.

1. 확장 패키지 형식: extension.toml 의 키(id, name, version, schema_version, authors, description, repository, themes/icon_themes/grammars/languages 등) 와 폴더 구조 (themes/, icon_themes/, languages/<lang>/config.toml, highlights.scm 등 쿼리 파일, grammars)
2. WASM 쪽: zed_extension_api 의 역할, wasm32-wasip2, WIT/컴포넌트 모델, 우리 범위(테마·아이콘·언어)에서 WASM 이 꼭 필요한지 (테마·아이콘 테마만 있는 확장은 WASM 없이 데이터만인지 — 설치된 catppuccin 확장으로 확인)
3. 호스트 쪽 동작: 확장 설치 위치·디렉터리 구조(installed, work, index 등), 불러오는 순서, 문법(grammar) 빌드·로드 방식(WASM 문법?), 테마·아이콘 테마 등록 방식, 확장 활성화/비활성화·제거, 업데이트
4. 확장 UI: Extensions 화면(ctrl+shift+x)의 구성·검색·설치/제거 버튼·필터, 확장 목록(레지스트리) 출처(api.zed.dev / zed-industries/extensions 저장소) — 우리 앱이 레지스트리를 쓸 수 있는지는 "판단 필요"로
5. 개발자 흐름: 로컬 개발 확장 설치(Install Dev Extension) 와 그 UI
6. 우리 앱에 옮길 때 필요한 사실: 테마·아이콘 테마 확장과 내장 테마(One, Catppuccin)의 관계, 확장으로 들어온 테마가 테마 선택기에 나타나는 규칙
7. 라이선스·보안: 확장 라이선스 요구, 샌드박스·권한(capabilities) 개념

표 위주. 끝나면 파일 경로만 답해.
