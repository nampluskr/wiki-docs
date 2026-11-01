# 사용자 결정 (2026-10-31, 1차 검토 후) — 모든 문서의 기준

## 포함 (메뉴·키·명령·툴팁에 넣고 명세·구현한다)
| 구분 | 기능 | Zed 액션(참고) |
|---|---|---|
| 편집 보조 | 파일 안 찾기·바꾸기 (Ctrl+F / Ctrl+H, 찾기 창) | buffer_search::Deploy 등 |
| 편집 보조 | 명령 팔레트 (Ctrl+Shift+P) | command_palette::Toggle |
| 편집 보조 | 파일 찾기 Go to File (Ctrl+P) | file_finder::Toggle |
| 편집 보조 | 최근 폴더 Open Recent | projects::OpenRecent |
| 설정 | **설정 파일 열기**: 별도 설정 UI 없이 settings.json 을 탭으로 열어 편집. ☰·∨ 메뉴의 Settings 항목은 이 동작으로 연결 (Settings UI 화면은 만들지 않는다) | zed::OpenSettingsFile |
| 키맵 | 키맵 파일 열기 (keymap.json 을 탭으로) | zed::OpenKeymapFile |
| 키맵 | **키맵 편집 화면** (Zed Keymap Editor 와 같게, Ctrl+K Ctrl+S) | zed::OpenKeymap |
| 탭 | 핀 탭 (Ctrl+K Shift+Enter, 앞쪽 고정, 일반 닫기에서 보호) | pane::TogglePinTab |
| 사이드바 | 등록 폴더 순서 변경 (Move Up/Down, 드래그 포함 여부는 Zed 기준) | — |
| 창 | 새 창 New Window | workspace::NewWindow |
| 파일 | 파일 하나 열기 Open File (폴더 등록 없이) | workspace::Open (파일) |
| 기타 | About (앱 정보 창) | zed::About |
| 기타 | Toggle All Docks (Ctrl+Shift+Y) | workspace::ToggleAllDocks |
| 기타 | 구문 단위 선택 Expand/Shrink Selection, Select Next/Previous Sibling (Tree-sitter) | editor::SelectLargerSyntaxNode 등 |

## 제외
- 프로젝트 전체 검색(Find in Project), Settings UI 화면, Check for Updates, Welcome/Onboarding, 오른쪽 Dock·하단 Dock 관련 메뉴(Dock Right 등), Panel Layout(Classic/Agentic), 기존 제외 목록(Sign In, 터미널, Git, AI, 협업, 디버그, LSP 진단·정의 이동 등, 원격). 확장은 아래 '확장 방식' 절대로 포함한다(테마·아이콘 테마·언어만).

## 확인된 사실 (leader)
- autosave 기본값은 **"off"** (Zed default.json, 사용자 settings.json 에도 설정 없음). 따라서 수정 표시(●)·닫기 저장 확인창이 기본 동작이다. `on_focus_change` 라고 적힌 곳은 고친다.
- 기준 화면 설정: theme mode=system (light One Light / dark Catppuccin Mocha), icon_theme=Catppuccin Latte, ui_font_size 16, buffer_font_size 15, base_keymap=VSCode.

## 확장 방식 (사용자 결정, 2026-10-31)
- **Zed 와 동일한 확장 방식**: extension.toml 매니페스트 패키지, 로직은 Rust → WebAssembly(wasm32-wasip2), 앱 재빌드 없이 설치·제거.
- 우리 앱이 지원하는 확장 기능(leader 제안 기본값): 테마, 아이콘 테마, 언어(Tree-sitter 문법·구문 강조 쿼리). 언어 서버(LSP)·디버거·MCP 서버·AI 슬래시 명령은 앱 범위 밖이라 제외.
- 화면(패널·탭 뷰어)을 추가하는 확장은 없다 (Zed 와 같음). PDF·동영상 뷰어는 확장 대상이 아니다.
- gpui-shell(JS 스크립트 런타임) 검토는 중단.
- PDF·동영상 뷰어(사용자 결정): 확장이 아니라, 필요할 때 **본체 기능으로 구현해 새로 빌드**한다. 이번 범위에는 넣지 않는다. 대신 04-APP-SPEC 에 **내장 뷰어 등록 구조**(파일 종류 → 뷰어 탭, 탭 아이콘·제목·툴팁·메뉴 연동)를 정의하고, 마크다운 미리보기를 첫 사례로 둔다. 이미지 뷰어(Zed 내장)도 같은 구조로 설명한다.

## 02-gpui / 03-tauri 범위 (사용자 결정)
- 구현은 **별도 프로젝트**에서 한다. 02-gpui/·03-tauri/ 는 그 프로젝트의 **출발 문서**다.
- 전체 내용(구조·접근·크레이트/라이브러리 구성·01-zed 항목 연결)만 담고, 구현 전에 할 **상세 조사·검토 항목을 목록**으로 남긴다. 상세 구현 문서·코드는 쓰지 않는다.
- 파일 구성: 폴더마다 README.md 하나 (02-gpui/README.md, 03-tauri/README.md), 같은 절 구성.

## 확장 세부 (사용자 결정, A-57 해소)
- 출처: **로컬 설치만** (확장 폴더·보관 파일을 선택해 설치). 온라인 목록·api.zed.dev·미러 없음.
- 확장 관리 화면: **Zed 와 같은 화면**(Ctrl+Shift+X). 필터는 All·Themes·Icon Themes·Languages. 온라인 검색·목록 대신 설치된 확장 목록, 제거, 로컬 설치(폴더/파일), 개발용 설치.
- Catppuccin 테마·Catppuccin Icons: **본체에 내장** (MIT 고지). One Dark·One Light·Zed (Default) 아이콘도 내장.
- 확장 끄기 기능: **없음** (Zed 와 같음, 설치되어 있으면 켜짐).
- 탭 읽기 전용(Make Tab Read-Only / Unlock Tab): 결정 목록에 없으므로 **제외**.
- 문서 포맷(Format Document/Selection, Shift+Alt+F, Ctrl+K Ctrl+F): 언어 서버·외부 포맷터가 필요해 범위 밖 → **제외** (leader 기본값, 사용자가 바꿀 수 있음).
- Tauri 판 편집기(사용자 결정): **CodeMirror 6 권장안 유지**. Monaco 는 대안으로만 남긴다.
