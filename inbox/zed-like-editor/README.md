---
tags: [rust, gui, zed, gpui, tauri, editor, guide]
created: 2026-10-31
updated: 2026-10-31
---

# Zed 같은 편집기 만들기

> 이 파일을 읽는 때: 처음, 그리고 무엇을 읽을지 모를 때. 이 문서 묶음의 입구다.

Zed의 기본 편집기 기능만 가진, 확장 가능한 기반 앱을 만든다. 사용감과 레이아웃은 **Zed와 거의 같은 수준**이어야 한다.

**방식은 포크가 아니라 reverse engineering 재구현이다.** Zed 소스를 가져와 불필요한 기능을 걷어내지 않는다. Zed의 화면·치수·동작·테마를 관찰과 문서·소스 분석으로 명세화하고, 그 명세대로 새 코드로 다시 만든다. 그래서 먼저 Zed의 화면·동작·테마를 명세로 정리하고(01-zed), 그 명세를 gpui-kit(1순위)과 Tauri 2(2순위)로 구현한다.

**구현은 별도 프로젝트에서 한다.** 이 묶음의 `02-gpui/`·`03-tauri/`는 그 프로젝트의 **출발 문서**다. 전체 구조와 접근 방법만 담고, 구현 전에 해야 할 **상세 조사·검토 항목**을 목록으로 남긴다. 상세 구현 문서는 각 프로젝트에서 쓴다.

| 항목 | 내용 |
| --- | --- |
| 대상 | Rust GUI 기초([../rust-gui/README.md](../rust-gui/README.md))를 읽은 사람 |
| 결과 | Zed와 비슷한 레이아웃·사용감의 멀티 폴더 편집기 (gpui-kit 판, Tauri 판) |
| 기준 | Windows 11, Rust 1.99.1, gpui-kit 0.8.1, Tauri 2.12.5, 2026-10 기준 Zed |
| 상태 | 1차 완료 (2026-10-31): 01-zed 명세 완료, 02-gpui·03-tauri 출발 문서, 04 비교·체크리스트 |

## 1. 만들 앱의 요건

| 번호 | 요건 |
| --- | --- |
| 1 | 멀티 세션 대신 **멀티 폴더** 관리 |
| 2 | 사이드바에서 작업 폴더 추가·등록·삭제 |
| 3 | 왼쪽 Dock에 선택한 폴더의 폴더 트리 — 폴더·파일 선택, 생성, 이름 변경, 삭제 |
| 4 | 트리에서 고른 파일을 편집기 영역에 **탭**으로 열어 보기·편집 — 마크다운 편집·미리보기, 코드 구문 강조 |
| 5 | 터미널 등 다른 기능은 넣지 않는다 |
| 6 | Zed의 레이아웃, 아이콘, 테마를 그대로 채용 |
| 7 | 테마·아이콘 테마 선택: Zed 기본(One Dark·One Light, Zed 기본 아이콘)과 Catppuccin 테마(Latte·Frappé·Macchiato·Mocha)·Catppuccin Icons 중에서 고른다 |

아이콘 위에 마우스를 올렸을 때 나오는 **툴팁**(문구, 단축키 표시, 지연, 위치, 스타일)도 Zed와 같게 한다. 타이틀 바 ☰ **앱 메뉴**도 Zed와 같은 구조로 하되, 우리 앱에서 쓸 수 있는 항목만 남긴다. 타이틀 바 오른쪽의 Sign In 버튼은 빼고, 옆의 **∨ 드롭다운 메뉴**는 같은 기준(쓸 수 있는 항목만)으로 채용한다. 아이콘과 각 영역(프로젝트 패널, 탭, 편집기, 미리보기, 사이드바 등)의 **우클릭 메뉴**도 Zed와 같게 하되, 쓸 수 없는 항목은 뺀다. 편집기 탭 영역의 **분할·이동**(분할 버튼·키, 탭 드래그, Pane 그룹과 크기 조절)도 Zed와 같게 한다.

**포함하는 보조 기능** (사용자 결정): 파일 안 찾기·바꾸기, 명령 팔레트, 파일 찾기(Go to File), 최근 폴더(Open Recent), 설정 파일 열기(설정 UI 없이 settings.json을 탭에서 편집), 키맵 파일 열기, 키맵 편집 화면, 핀 탭, 사이드바 폴더 순서 변경, 새 창, 파일 하나 열기, About, Toggle All Docks, 구문 단위 선택. **제외**: 프로젝트 전체 검색, 설정 UI 화면, 업데이트 확인, 오른쪽·하단 Dock.

**확장**(사용자 결정): Zed와 같은 방식이다. `extension.toml` 패키지, 로직은 Rust → WebAssembly(`wasm32-wasip2`), 앱을 다시 빌드하지 않고 설치한다. 지원하는 확장 기능은 테마, 아이콘 테마, 언어(문법·구문 강조)다. 화면을 추가하는 확장은 없다. 확장은 **로컬 설치만** 지원하고(온라인 목록 없음), Zed와 같은 **확장 관리 화면**(Ctrl+Shift+X)을 둔다. 확장 끄기 기능은 없다(Zed와 같음). Catppuccin 테마·아이콘은 본체에 내장한다. PDF·동영상 뷰어는 확장이 아니라, 필요할 때 본체 기능으로 구현해 새로 빌드한다. 이를 위해 명세에 **내장 뷰어 등록 구조**(파일 종류 → 뷰어 탭)를 두고, 마크다운 미리보기를 첫 사례로 삼는다.

**keymap**: 사용자 Zed 설정은 `"base_keymap": "VSCode"`다. Windows의 Zed는 기본 keymap(`default-windows.json`) 위에 `keymaps/linux/vscode.json`을 덧씌우므로, 우리 앱은 **이 덧씌운 결과**를 적용 keymap으로 쓴다. 메뉴·툴팁에 표시하는 단축키도 같다.

기준 화면은 두 장이다. 사용자가 실제로 쓰는 Zed 화면이다.
- [01-zed/assets/zed-reference.png](01-zed/assets/zed-reference.png): 다크(Catppuccin Mocha), 창 최대화, 2880×1704
- [01-zed/assets/zed-reference-latte.png](01-zed/assets/zed-reference-latte.png): 라이트(Catppuccin Latte), 창을 줄인 상태(최대화 아님), 2448×1480

 쓰지 않는 기능의 아이콘·메뉴·옵션은 넣지 않는다. 기준 화면의 설정은 아래와 같다.
- 테마: 시스템 모드를 따른다. 라이트는 One Light, 다크는 Catppuccin Mocha
- 아이콘 테마: Catppuccin Latte
- 글꼴 크기: UI 16, 편집기 15

## 2. 읽기 지도

| 지금 할 일 | 읽을 곳 |
| --- | --- |
| Zed가 어떻게 생겼고 어떻게 동작하는지 안다 | [01-zed/01-LAYOUT.md](01-zed/01-LAYOUT.md)부터 차례로 |
| gpui-kit으로 만든다 (1순위) | [02-gpui/README.md](02-gpui/README.md) |
| Tauri 2로 만든다 (2순위) | [03-tauri/README.md](03-tauri/README.md) |
| 두 구현을 비교하고 Zed와 얼마나 같은지 확인한다 | [04-COMPARISON.md](04-COMPARISON.md) |

| 폴더·파일 | 담은 것 |
| --- | --- |
| [01-zed/01-LAYOUT.md](01-zed/01-LAYOUT.md) | 화면 구조와 치수: 타이틀 바, 사이드바, 왼쪽 Dock, 탭 바, 편집기, 상태 표시줄, 툴팁·메뉴·선택 창·대화상자·키맵 편집·확장 화면의 모양 |
| [01-zed/02-INTERACTION.md](01-zed/02-INTERACTION.md) | 사용감: 키맵(VSCode 적용), 탭·분할·드래그, 트리 조작, ☰·∨·우클릭 메뉴, 툴팁, 편집기 동작, 선택 창, 키맵 편집, 확장 관리 |
| [01-zed/03-THEME-ICONS.md](01-zed/03-THEME-ICONS.md) | 테마 토큰 구조, One·Catppuccin 실제 값, 아이콘·파일 아이콘 매핑, 글꼴, 테마 선택, 라이선스 고지 |
| [01-zed/04-APP-SPEC.md](01-zed/04-APP-SPEC.md) | 우리 앱 명세: 요건 1~7을 Zed 분석에 연결, 멀티 폴더, 데이터 모델·설정 저장 형식, 일치도 판정 기준, 확장, 내장 뷰어 등록 구조 |
| [02-gpui/README.md](02-gpui/README.md) | gpui-kit 프로젝트 출발 문서: 전체 구조, 크레이트 구성, 01-zed 항목 ↔ 구성 요소 연결, 상세 조사·검토 항목 목록, 위험, 시작 순서 |
| [03-tauri/README.md](03-tauri/README.md) | Tauri 2 프로젝트 출발 문서: 02-gpui와 같은 절 구성 |
| [04-COMPARISON.md](04-COMPARISON.md) | 두 방식 비교, Zed 일치도 체크리스트(01-zed 항목 ID 기준) |
| [research/README.md](research/README.md) | 조사 원자료·검토 기록·지시문·사용자 결정 보관 (기준은 01-zed 명세) |

**짝 규칙**: `02-gpui/README.md`와 `03-tauri/README.md`는 같은 절 구성으로 쓴다. 두 문서는 `01-zed`의 같은 항목 ID를 참조한다.

## 3. 라이선스 원칙

Zed 저장소는 대부분 GPL-3.0-or-later이고, gpui 같은 일부만 Apache-2.0이다. https://github.com/zed-industries/zed

| 구분 | 대상 | 방법 |
| --- | --- | --- |
| 명세로 적는다 | Zed 공식 문서, 화면 관찰, 키맵·동작의 **사실**(어떤 키가 무엇을 하는지, 치수) | 우리 말로 다시 쓴다 |
| 가져온다 | `assets/themes`(MIT), `assets/icons`의 Lucide 아이콘(ISC) | 저작권 고지를 함께 배포한다 |
| 가져온다 | Catppuccin 테마 https://github.com/catppuccin/zed , Catppuccin Icons https://github.com/catppuccin/zed-icons (둘 다 MIT, Copyright (c) 2021 Catppuccin) | 저작권 고지를 함께 배포한다 |
| 가져오지 않는다 | `workspace`·`ui`·`editor`·`theme` 등 앱 코드(GPL-3.0), 키맵 JSON 파일 원문 | 동작을 이해하는 데만 참고한다. 복사하지 않는다 |

Zed가 직접 그린 아이콘은 개별 라이선스를 확인한 뒤에만 쓴다. 이 원칙은 법률 자문이 아니다.

---

다음: [01-zed/01-LAYOUT.md](01-zed/01-LAYOUT.md) · 관련: [../rust-gui/README.md](../rust-gui/README.md)
