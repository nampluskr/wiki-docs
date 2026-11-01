---
tags: [rust, gui, zed, tauri, gpui, editor, spec, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 01-zed · 04 · 우리 앱 명세

> 이 파일을 읽는 때: 무엇을 만들지 확정할 때, 데이터·설정 파일 형식을 정할 때, 만든 앱이 Zed 와 얼마나 같은지 판정할 때.

| 항목 | 내용 |
| --- | --- |
| 근거 | Zed 커밋 [14b7b0e](https://github.com/zed-industries/zed/tree/14b7b0ef3764d62a85070c8b5ccea105a2ad3235) 의 소스·문서를 정리한 01-LAYOUT(`L-`), 02-INTERACTION(`I-`), 03-THEME-ICONS(`T-`) |
| 검증 | 소스 대조. 실행 검증 안 함. Zed 의 사실과 우리의 결정을 구분한다: 결정은 "(판단)" 이라고 적는다 |
| 이 파일의 ID | `A-NN`. 01~03 의 ID 는 그대로 참조한다 |
| 주의 | 01·02·03 은 다른 writer 가 함께 고친다. 이 문서는 ID 로 참조한다. 02 의 ID 대역: ☰ 메뉴 I-300 ~ I-488(9절), ∨ 메뉴 I-600 ~ I-655(10절), 우클릭 메뉴 I-500 ~ I-668(11절. ∨ 메뉴와 겹치던 옛 I-600 ~ I-652 는 **I-900 ~ I-952 로 옮겨졌다**, id-renumber.md), 탭 분할·이동 I-706 ~ I-739(3.1절), 툴팁 L-200 ~ L-272(01 4.7절). 편집기(E-001 ~), 선택기·대화상자(P-001 ~), 키맵 편집 화면(K-001 ~), 조사는 아직 문서에 들어오지 않았다 (확장 X- 조사는 4.11절에 반영했다) |
| 확장 방식 | 사용자 결정(_decisions.md "확장 방식")과 확장 조사(`zed-extensions.md`, ID `X-`)를 4.11절(A-46, A-48 ~ A-57)에 반영했다. 확장은 Zed 와 같은 방식이고 지원 종류는 테마·아이콘 테마·언어(문법·구문 강조)뿐이다. 내장 뷰어는 확장이 아니라 본체의 등록 구조다 (3.12절) |
| 사용자 결정 | `_decisions.md`(2026-10-31)가 최우선이다. 찾기·바꾸기, 명령 팔레트, Go to File, Open Recent, 설정·키맵 파일 열기, 키맵 편집 화면, 핀 탭, 사이드바 순서 변경, 새 창, Open File, About, Toggle All Docks, 구문 단위 선택을 **포함**한다. 프로젝트 전체 검색, Settings UI 화면, Check for Updates, Welcome, 오른쪽·아래 Dock 메뉴, Panel Layout 은 **제외**한다 |
| autosave | 기본 `off` (Zed default.json L1471). 수정 표시(●)와 닫기 저장 확인이 기본 동작이다 |

기준 화면이다. 일치도를 판정할 때 이 화면과 겹쳐 본다 (7절).

![기준 화면](assets/zed-reference.png)

두 번째 기준 화면이다. Catppuccin Latte(라이트), 창을 최대화하지 않은 상태다.

![기준 화면 (Latte)](assets/zed-reference-latte.png)

## 1. 요건 1~7 과 항목 ID

README 의 요건 번호를 그대로 쓴다. 빈 칸은 해당 문서에 그 요건의 항목이 없다는 뜻이다.

| ID | 요건 | 우리 앱의 결정 | 01-LAYOUT | 02-INTERACTION | 03-THEME-ICONS | 이 문서 |
| --- | --- | --- | --- | --- | --- | --- |
| A-01 | 1. 멀티 세션 대신 멀티 폴더 | 등록한 폴더마다 사이드바에 항목 하나. 선택한 폴더 하나만 작업 영역에 보인다. 순서를 바꿀 수 있다 | L-11, L-12, L-33 ~ L-35 | I-105, I-108, I-111, I-934, I-940 | — | 3절, 4절 |
| A-02 | 2. 사이드바에서 폴더 추가·등록·삭제 | 추가 = 등록. 삭제 = 등록 해제(디스크는 그대로). 디스크 삭제는 트리에서만 한다 | L-34, L-35, L-95, L-96 | I-106, I-107, I-109, I-110, I-093, I-935 | T-10.5 ~ T-10.7 | 3절 |
| A-03 | 3. 왼쪽 Dock 폴더 트리, 선택·생성·이름 변경·삭제 | 선택한 폴더의 트리만 프로젝트 패널에 그린다 | L-14, L-16, L-30 ~ L-32, L-36 ~ L-50, L-125 ~ L-128 | I-030 ~ I-043, I-087 ~ I-097 | T-10.8, T-20, T-28 | 3.5절, 3.6절 |
| A-04 | 4. 탭으로 열어 보기·편집, 마크다운 편집·미리보기, 구문 강조 | 탭·미리보기 탭·**핀 탭**·**탭 분할·이동(필수)** 은 Zed 규칙 그대로. 찾기·바꾸기, 구문 단위 선택 포함 | L-15, L-22 ~ L-26, L-61 ~ L-74, L-101 ~ L-111, L-130 ~ L-133, L-136 | I-044 ~ I-047, I-060 ~ I-086, I-706 ~ I-739, I-367, I-372, I-373, I-479 | T-05, T-06, T-10.9 ~ T-10.16, T-10.18, T-27.1, T-27.2 | 4.3절, 4.7절, 3.11절 |
| A-05 | 5. 터미널 등 다른 기능은 넣지 않는다 | 2절의 제외 목록 | L-52, L-87, L-89, L-98 | — | — | 2절 |
| A-06 | 6. Zed 의 레이아웃, 아이콘, 테마를 그대로 채용 | 치수·색·아이콘·툴팁·메뉴(☰, ∨, 우클릭)를 Zed 와 같게 한다. 쓸 수 없는 항목만 뺀다 | L-01 ~ L-10, L-21 ~ L-40, L-200 ~ L-272 | I-300 ~ I-488, I-500 ~ I-668, I-900 ~ I-952, I-600 ~ I-655 | T-01 ~ T-17, T-26, T-30, T-31 | 3.9절, 3.10절, 7절 |
| A-07 | 7. 테마·아이콘 테마 선택 | 테마 6종, 아이콘 테마 5종. 미리 적용·취소·저장은 Zed 규칙 | L-85 | I-336, I-337, I-622, I-623 | T-19 ~ T-23, T-29 | 3.9절, 3.10절, 4.2절 |
| A-08 | (추가 요건) 탭 분할·이동을 Zed 와 같게 | 분할 버튼·키·드래그·분할선. **필수** | L-15, L-17, L-72, L-137 ~ L-139, L-244, L-245 | I-706 ~ I-739 | T-02 (`drop_target.*`), T-26.13 | 4.7절 |
| A-46 | (추가 요건) 확장: Zed 와 같은 방식 | `extension.toml` 패키지, 재빌드 없이 설치·제거. 지원: 테마·아이콘 테마·언어(문법·구문 강조). 제외: LSP·디버거·MCP·AI·스니펫 등. **설치는 로컬뿐**(폴더·보관 파일·개발용). 확장 관리 화면은 있다(`Ctrl+Shift+X`). 확장을 끄는 기능은 없다. 설치된 Zed 확장 패키지를 그대로 쓸 수 있어야 한다 | — | I-339, I-624 (메뉴 항목) | T-01, T-11, T-12, T-19, T-20, T-32, X-01 ~ X-66 | 4.11절 |
| A-47 | (추가 요건) 내장 뷰어 등록 구조 | 파일 종류 → 뷰어 탭. 마크다운 미리보기가 첫 사례, 이미지 뷰어가 둘째. PDF·동영상 뷰어는 향후 본체 기능 | L-69 | I-044 ~ I-047, I-077 ~ I-086, I-060 | T-10.12 | 3.12절 |
| A-09 | (추가 요건) keymap 은 `base_keymap: VSCode` 적용 결과 | 메뉴·툴팁의 단축키 표시도 같은 결과 | L-217 ~ L-223 | 02 1절, 3.1.4절(I-731 ~ I-739), 8.4절 | — | 4.8절 |

### 1.1 사용자 결정으로 포함된 기능 (A-42)

_decisions.md 의 "포함" 표를 이 문서의 어디에서 정하는지 연결한다. 값이 비어 있는 곳은 조사 ID 를 적었다.

| ID | 기능 | 우리 앱의 결정 | 근거 ID | 이 문서 |
| --- | --- | --- | --- | --- |
| A-42.1 | 파일 안 찾기·바꾸기 | `Ctrl+F`(찾기), `Ctrl+H`(바꾸기). 창 모양·옵션·결과 수 표시는 E-001 ~ 조사 대기 | I-367, I-479 | 3.11절 |
| A-42.2 | 명령 팔레트 | `Ctrl+Shift+P`, `F1`. 항목은 우리 앱에 있는 명령만 | I-416, I-028 | 3.11절 |
| A-42.3 | Go to File | `Ctrl+P`. 활성 폴더의 파일만 찾는다 | I-418, I-007 | 3.11절 |
| A-42.4 | Open Recent | 최근 폴더 이력을 저장한다. 이력이 폴더 사이드바와 겹치지 않게 모델을 나눈다 | I-347, L-226 | 4.3절 A-22.23, 3.11절 |
| A-42.5 | 설정 파일 열기 | `settings.json` 을 편집기 탭으로 연다. 별도 Settings 화면은 없다 | I-326, I-327, I-620 | 3.11절 |
| A-42.6 | 키맵 파일 열기 | `keymap.json` 을 편집기 탭으로 연다 | I-333 | 3.11절, 4.1절 |
| A-42.7 | 키맵 편집 화면 | `Ctrl+K Ctrl+S`. 화면 구성은 K-001 ~ 조사 대기. 저장 위치는 `keymap.json` | I-332, I-621 | 3.11절 |
| A-42.8 | 핀 탭 | `Ctrl+K Shift+Enter`, 탭 우클릭 Pin Tab. 상태 복원 포함 | I-014, I-065, I-066, I-577, T-10.18 | 4.3절, 4.6절 |
| A-42.9 | 사이드바 순서 변경 | 우클릭 Move Up / Move Down. 저장 순서는 `folders` 배열 순서다 | I-934, I-940, I-667 | 3.4절, 4.3절 |
| A-42.10 | 새 창 | `Ctrl+Shift+N`. 창마다 폴더 목록·탭·Dock 상태가 따로다 | I-343 | 3.11절, 4.3절 |
| A-42.11 | Open File | `Ctrl+O`. 폴더 등록 없이 파일 하나를 탭으로 연다 | I-345, I-003 | 3.11절 |
| A-42.12 | About | 앱 정보 창. 문구·크기는 P-001 ~ 조사 대기 | I-322 | 3.11절 |
| A-42.13 | Toggle All Docks | `Ctrl+Shift+Y`. 왼쪽 Dock 이 하나뿐이라 `Ctrl+B` 와 같은 동작이지만 메뉴에 둘 다 둔다 | I-394 | 3.11절 |
| A-42.15 | 확장 | A-46, A-48 ~ A-58 (확장 조사 X- 와 확장 세부 결정 반영) | 결정 문서, X-01 ~ X-66 | 4.11절 |
| A-42.16 | 내장 뷰어 등록 | A-47 | 결정 문서 | 3.12절 |
| A-42.14 | 구문 단위 선택 | Expand / Shrink Selection, Select Next / Previous Sibling. Tree-sitter 구문 트리를 쓴다. 키는 VSCode 적용 결과 | I-372, I-373 | 3.11절 |

## 2. 넣지 않는 것 (A-05)

사용자 결정(_decisions.md)을 반영한 제외 목록이다. 아이콘·메뉴·옵션 모두 만들지 않고 설정 파일에도 키를 두지 않는다.

| ID | 제외 | 근거 |
| --- | --- | --- |
| A-05.1 | 타이틀 바 오른쪽의 **Sign In**, 통화 컨트롤, 연결 상태, 업데이트 알림. 사용자 메뉴 `∨` 와 창 버튼은 **둔다** (3.10절) | L-89, I-602, I-607 |
| A-05.2 | 타이틀 바의 Git 브랜치, 원격 호스트, 제한 모드 경고 | L-87 |
| A-05.3 | 사이드바의 스레드 검색, 세션 이름, 경과 시간(`12h`), 기록 버튼 | L-12, L-95 |
| A-05.4 | 프로젝트 패널의 Git 상태 색·아이콘, 진단 배지 | L-52 |
| A-05.5 | 상태 표시줄의 기록·검색(프로젝트 검색)·진단·편집 예측·터미널·디버그·개요·협업·Git·AI·언어·인코딩·줄 끝 | L-98 |
| A-05.6 | 프로젝트 전체 검색(Find in Project), Settings UI 화면, Check for Updates, Welcome·Onboarding | _decisions.md |
| A-05.7 | 오른쪽 Dock·아래 Dock 과 그 메뉴(Dock Left/Right 전환 포함), Panel Layout(Classic·Agentic) | _decisions.md, L-14 |
| A-05.8 | 원격(Remote), 터미널, Git, AI, 협업, 디버그, LSP 진단·정의 이동. **확장은 포함**이다(테마·아이콘 테마·언어만, A-46). 확장으로 LSP·디버거·MCP 서버·AI 슬래시 명령은 더하지 못한다 | _decisions.md |
| A-05.13 | 화면(패널·탭 뷰어)을 더하는 확장. PDF·동영상 뷰어를 확장으로 만드는 것 | _decisions.md, A-47.9 |
| A-05.9 | 탭 전환기(`Ctrl+Tab`). 사용자 결정에 없어서 넣지 않는다 (판단) | I-011 |
| A-05.10 | 탭 `+` 메뉴의 Search·Terminal 항목 (New File·Open File 은 남긴다) | L-71 |
| A-05.11 | Vim 모드, 미니맵, 인레이 힌트, 멀티플레이어 `players[1..]` | 03 의 토큰 중 쓰지 않음 |
| A-05.12 | 이전 판이 제외했던 명령 팔레트, Go to File, 최근 폴더, 키맵 편집은 **이제 포함**이다 (1.1절) | _decisions.md |

## 3. 멀티 폴더 사이드바 명세

### 3.1 Zed 의 어느 자리인가 (A-10)

| ID | 항목 | Zed | 우리 앱 |
|---| --- | --- | --- |
| A-10.1 | 자리 | 창 맨 왼쪽, 창 전체 높이. 타이틀 바 줄까지 차지한다 (L-11, L-12) | 같다 |
| A-10.2 | 내용 | 프로젝트 헤더 아래에 스레드 목록 | **등록 폴더 목록**. 폴더 하나가 항목 하나다 |
| A-10.3 | 층 | 창의 여러 프로젝트(workspace) ≠ 한 프로젝트의 여러 루트(worktree) ≠ 스레드 (I-105) | 우리의 "폴더" 는 Zed 의 **프로젝트 한 개**에 해당한다. 폴더 하나에 루트 하나 (판단) |
| A-10.4 | 헤더 클릭 | 스레드 묶음 접기·펼치기 (I-109) | 클릭 = 그 폴더를 활성화 (판단, I-109 가 권장) |
| A-10.5 | 추가 버튼 | 최근 프로젝트·로컬 폴더 추가 (I-110) | 폴더 선택 창을 연다. 최근 폴더는 ☰ File → Open Recent… 와 타이틀 바의 프로젝트 이름 버튼이 연다 (A-13.9) |
| A-10.6 | 한 번에 여러 폴더 추가 | multi-root 프로젝트 하나로 묶인다 (I-110) | **폴더마다 독립 항목**으로 등록한다 (판단). 선택 창은 폴더 하나만 고른다 |

### 3.2 배치와 치수 (A-11)

| ID | 항목 | 값 | 출처 |
| --- | --- | --- | --- |
| A-11.1 | 폭 | 기본 300, 최소 200, 최대 800. 끌어서 조절. 사용자가 바꾼 폭은 `state.json` 에 저장 | L-33, I-116 |
| A-11.2 | 맨 위 줄 | 높이 32(= 타이틀 바 높이). 내용 없음. 창 끌기 영역으로 쓴다 (스레드 검색 칸은 뺀다). 아래에 1px `border` 선이 있고 타이틀 바 아래 선과 한 줄로 이어진다 | L-12, L-21, A-05.3, T-08.13 |
| A-11.3 | 폴더 항목 줄 | 높이 31, 왼쪽 padding 8, 오른쪽 padding 6 | L-34 |
| A-11.4 | 푸터 | 높이는 상태 표시줄과 같은 30, padding 4, 버튼 사이 gap 4, 위 테두리 1px, 아이콘 버튼 14 | L-35, L-28 |
| A-11.5 | 오른쪽 테두리 | 1px `border`. 사이드바 오른쪽에서 Workspace 가 시작한다 | L-16 |
| A-11.6 | 배경 | `panel.background` | T-04 |
| A-11.7 | 열림 상태 시작값 | 열림 | 판단 |
| A-11.10 | 영역 사이 1px 선 (모두 `border`) | ① 타이틀 바 아래, ② 사이드바 맨 위 줄 아래, ③ 사이드바 푸터 위, ④ 사이드바 오른쪽, ⑤ 탭 바 아래(활성 탭 아래는 끊김), ⑥ 상태 표시줄 위. 두 번째 기준 화면에서 읽은 색이 `#ccd0da` 로 Latte `border` 와 같다 | T-08.13 |
| A-11.11 | 푸터와 상태 표시줄의 바탕색이 다르다 | 푸터는 `panel.background`, 상태 표시줄은 `status_bar.background` | T-08.8, T-08.9 |
| A-11.12 | 폭 조절 핸들 | 사이드바 폭을 끄는 핸들의 위치·너비·커서·더블클릭 초기화는 확인하지 못했다. Dock 의 6px 핸들(L-30)과 같다고 가정하지 않는다. 값은 01 에서 조사가 들어오면 채운다 (A-40.12) | L-30, L-33 |

푸터의 버튼 순서(왼쪽 → 오른쪽 정렬 규칙)는 Zed 에서 미확인이다 (L-95, L-96 의 "확신 낮은 부분"). 우리는 아래로 정한다 (판단).

| ID | 사이드바 | 푸터(사이드바 안) | 상태 표시줄 왼쪽 |
| --- | --- | --- | --- |
| A-11.8 | 열림 | 왼쪽에 사이드바 토글(T-10.7), 오른쪽 끝에 폴더 추가(T-10.5) | 프로젝트 패널 토글(T-10.8) |
| A-11.9 | 닫힘 | 보이지 않는다 | 사이드바 토글(T-10.6), 폴더 추가(T-10.5), 프로젝트 패널 토글(T-10.8) 순서 |

상태 표시줄 왼쪽의 세 버튼 순서는 사용자 결정(사이드바 토글, 폴더 추가, 프로젝트 패널 토글)이다.

### 3.3 폴더 항목의 모양 (A-12)

| ID | 상태 | 모양 |
| --- | --- | --- |
| A-12.1 | 기본 | [폴더 아이콘 14][이름]. 이름은 한 줄, 넘치면 말줄임. 글자색은 비활성 `text.muted`, 활성·hover `text`(프로젝트 패널의 비선택 `text.muted` 와 선택 `text` 를 두 기준 화면에서 읽은 값에 맞춘다, T-08.14, T-08.15). 아이콘은 `icon.muted`. **기준 화면의 폴더 행에는 아이콘이 없으므로 아이콘은 우리 앱이 더한 것이다 (우리 앱 값, 03 T-10.21)** |
| A-12.2 | 활성(선택) | 배경 `ghost_element.selected` (기준 화면의 `#3b3c4f`, T-08) |
| A-12.3 | hover | 배경 `ghost_element.hover` |
| A-12.4 | 누름 | 배경 `ghost_element.active` |
| A-12.5 | 키보드 포커스 | 항목 둘레 `border.focused` 1px (판단, L-48 의 트리 규칙을 따른다) |
| A-12.6 | 이름이 같은 폴더가 둘 이상 | 이름 뒤에 `text.muted` 로 바로 위 폴더 이름을 덧붙인다. 예: `docs · guides` (판단, 탭의 구분 설명 L-69 와 같은 방식) |
| A-12.7 | 경로가 없어졌다 | 이름과 아이콘을 `text.disabled`·`icon.disabled` 로 그린다. 클릭하면 활성화는 되지만 트리 자리에 안내를 보인다 (3.7절) |
| A-12.8 | tooltip | **없다.** Zed 의 폴더 헤더 행에는 툴팁이 없다 (L-231). 전체 경로를 보려면 프로젝트 패널의 루트 항목에 마우스를 올린다 (1500ms, L-247) |
| A-12.10 | 우클릭 메뉴 | `Move Up`, `Move Down`(폴더가 둘 이상일 때, 맨 위·맨 아래면 비활성), 구분선, `Remove`. Zed 문구 그대로 (I-667, I-934, I-940, I-935). 메뉴를 여는 ⋯ 버튼은 두지 않는다 (판단) |
| A-12.11 | 끌어서 순서 바꾸기 | Zed 에서 사이드바 헤더를 끌어 옮기는지는 확인하지 못했다. 우리 앱은 우클릭 Move Up/Down 만 만든다 (판단, A-40.13) |
| A-12.9 | 기준 화면 대응 | 기준 화면의 두 줄 항목(`BOOK-RH00M2K19S: p…`, `12h`)은 스레드 항목이라 가져오지 않는다. 폴더 항목 한 줄로 대체한다 |

### 3.4 등록·선택·해제 동작 (A-13)

| ID | 동작 | 방법 | 결과 | 디스크 |
| --- | --- | --- | --- | --- |
| A-13.1 | 폴더 추가(등록) | 푸터·상태 표시줄의 폴더 추가 버튼(툴팁 `Add Project`, L-230), ☰ File → `Open Folder...`, `Ctrl+K Ctrl+O`. 이 키는 Zed 에서 `workspace::Open` 이고 우리는 "폴더 등록"에 쓴다 (Zed 키를 다른 동작에 재사용, 판단, I-004) | OS 폴더 선택 창(폴더 하나)을 연다. 고르면 목록 맨 아래에 항목을 붙이고 **활성화**한다. 취소하면 아무 일도 없다 | 읽기만 한다 |
| A-13.2 | 이미 등록된 폴더를 다시 고름 | 같은 경로(Windows 는 대소문자 무시, 끝 `\` 무시) | 새 항목을 만들지 않고 기존 항목을 활성화한다 | 없음 |
| A-13.3 | 선택(활성화) | 항목 클릭, 포커스 후 `Enter`, `Up`/`Down` 으로 이동 후 `Enter` | `activeFolderId` 를 바꾼다. 트리·탭·타이틀 바 이름이 그 폴더 것으로 바뀐다 | 없음 |
| A-13.4 | 등록 해제 | ① 사이드바 항목 우클릭 → **Remove** (I-935), ② 사이드바에 포커스가 있을 때 `Delete` (Zed 에 없는 키, 판단), ③ 프로젝트 패널의 **루트 항목** 우클릭 → **Remove from Project** (I-555) | 대상은 ①②는 선택한 항목, ③은 지금 활성 폴더다. 저장 안 한 탭이 있으면 먼저 Save / Don't Save / Cancel (I-069)을 폴더의 탭마다 묻는다. 하나라도 Cancel 이면 해제하지 않는다. 그 밖의 확인 창은 없다 | **그대로** |
| A-13.5 | 활성 폴더를 해제 | 위와 같음 | 아래 항목, 없으면 위 항목을 활성화한다. 하나도 없으면 빈 상태(3.7절) | 그대로 |
| A-13.6 | 순서 | 처음에는 등록 순서. 우클릭 Move Up / Move Down 으로 한 칸씩 바꾼다. 바꾼 순서는 `folders` 배열 순서로 저장한다 (A-22.2) | `folders` 배열을 바꾼다 | 없음 |
| A-13.8 | 새 폴더를 놓는 자리 | 목록의 맨 아래. 이미 있으면 그 자리를 유지한다 | — | — |
| A-13.9 | 타이틀 바의 프로젝트 이름 버튼 | 활성 폴더 이름(40자 말줄임, L-88). 누르면 Open Recent 목록(P-001 ~ 조사 대기)을 연다. 툴팁 `Recent Projects` (L-226). 폴더가 없으면 글자가 `Open Recent Project`(Muted, L-227) | 최근 이력만 읽는다 | 없음 |
| A-13.7 | 이름 바꾸기 | 없다. 이름은 폴더 이름이다 (Zed 도 폴더 이름을 쓴다, L-88) | — | — |

Zed 의 대응: 프로젝트 루트의 **Remove from Project** 는 `remove_worktree` 로 등록만 뺀다 (I-107). 루트의 우클릭 메뉴에는 Trash·Delete 가 없다 (I-093). 우리는 이 규칙을 그대로 쓴다.

### 3.5 등록 해제와 디스크 삭제를 구분한다 (A-14)

메뉴 문구는 Zed 그대로 쓴다. 사이드바의 `Remove` 와 프로젝트 루트의 `Remove from Project` 는 같은 동작(등록 해제)이다 (I-555, I-935).

| ID | 구분 | 등록 해제 | 파일·폴더 삭제 (휴지통) | 파일·폴더 삭제 (영구) |
| --- | --- | --- | --- | --- |
| A-14.1 | 어디서 | 사이드바 항목, **트리의 루트 항목**(활성 폴더를 해제) | 트리의 루트가 아닌 항목 | 같음 |
| A-14.2 | 메뉴 글자·키 | `Remove`(사이드바), `Remove from Project`(루트). 키는 **사이드바 포커스일 때 `Delete`** 뿐이다 | `Trash`. 키는 **프로젝트 패널 포커스일 때** `Delete`·`Backspace` | `Delete`. 키는 **프로젝트 패널 포커스일 때** `Shift+Delete`·`Ctrl+Delete`·`Ctrl+Backspace` |
| A-14.3 | 디스크 | 변화 없음 | 휴지통으로 옮긴다 | 지운다 |
| A-14.4 | 확인창 | 없음(저장 안 한 탭이 있을 때만 저장 질문) | 있음 | 있음 |
| A-14.5 | 되돌림 | 폴더를 다시 추가하면 된다. 열었던 탭은 복구되지 않는다 | 휴지통에서 복원 | 불가 |
| A-14.6 | 근거 | I-107, I-935, I-555 | I-039, I-093, I-552 | I-040, I-093, I-553 |

확인창 문구는 Zed 소스(project_panel.rs L2838-L2880, L3039-L3042)를 따른다. 이 문구는 검토 자료에서 옮겼고 이 문서에서 소스를 다시 열어 대조하지는 않았다.

| ID | 경우 | 본문 | 버튼 |
| --- | --- | --- | --- |
| A-14.7 | 휴지통, 항목 하나 | ``Do you want to trash `name`?`` | `Trash` 와 취소 |
| A-14.8 | 영구 삭제, 항목 하나 | ``Are you sure you want to permanently delete `name`?`` + 보조 문구 `This cannot be undone.` | `Delete` 와 취소 |
| A-14.9 | 여러 항목 | 본문이 `… the following N files?` 로 바뀌고 목록을 보인다. 넘치면 `.. N files not shown` | 위와 같음 |
| A-14.10 | 수정된 버퍼가 있다 | 본문 끝에 `It has unsaved changes, which will be lost.` 를 덧붙인다 | 위와 같음 |
| A-14.11 | 실패 | `Failed to trash/delete X of Y file(s).` 알림 | — |

- 트리 루트 항목의 우클릭 메뉴에는 `Trash`·`Delete`·`Rename` 을 넣지 않는다 (I-091, I-093, I-095, I-551 ~ I-553). 루트가 선택된 채 `Delete` 키를 눌러도 아무 일도 하지 않는다 (판단).
- 포커스별 `Delete`: 사이드바 → 등록 해제(A-13.4), 프로젝트 패널의 루트가 아닌 항목 → 휴지통(I-039), 편집기 → 글자 삭제(I-057). 같은 키라도 포커스 영역이 정한다 (I-002).
- 휴지통·영구 삭제는 키맵 편집 화면에서 키를 바꿔도 확인창을 건너뛰지 않는다 (`skip_prompt` 는 만들지 않는다, 판단).

### 3.6 폴더를 바꾸면 무엇이 바뀌나 (A-15)

폴더마다 작업 문맥을 따로 보관한다 (I-108, I-111). 전환해도 이전 폴더의 문맥은 메모리에 남는다.

| ID | 항목 | 폴더마다 따로 | 전체 공유 |
| --- | --- | --- | --- |
| A-15.1 | 열린 탭, 활성 탭, 임시(preview) 탭, 핀 탭, 이동 기록(←/→), Pane 분할 구조, 활성·확대 Pane | 예 | — |
| A-15.2 | 프로젝트 패널의 펼침 상태, 선택 항목, 스크롤 | 예 (저장은 안 한다. A-25.7) | — |
| A-15.3 | 저장 안 한 버퍼(dirty) | 예. 전환해도 유지한다 | — |
| A-15.4 | 사이드바 폭, 왼쪽 Dock 폭·열림, 테마, 아이콘 테마, 글꼴 | — | 예 |
| A-15.5 | 타이틀 바의 폴더 이름 | 활성 폴더 이름 (40자 말줄임) | L-88 |
| A-15.6 | 포커스 | 전환한 뒤 포커스는 사이드바에 그대로 둔다. 편집기로 가려면 `Esc` 또는 클릭 (판단) | I-099 |

활성 폴더 안의 트리는 그 폴더 루트 하나다. 루트를 맨 위 항목으로 그린다 (L-41). `docs\guides` 처럼 자식이 하나뿐인 폴더는 한 줄로 합친다 (L-42).

### 3.7 예외 상태 (A-16)

| ID | 상황 | 동작 |
| --- | --- | --- |
| A-16.1 | 등록 폴더가 없다 (처음 실행·모두 해제) | 사이드바 목록은 비어 있다. 프로젝트 패널은 Zed 의 빈 화면(ProjectEmptyState)을 따른다: 문구 `Choose one of the options below to use the Project Panel` 와 버튼. 버튼은 **`Open Project` 하나만** 둔다. 누르면 폴더 선택 창을 연다(A-13.1). Zed 의 `Clone Repository` 는 뺀다. 편집기 영역은 `editor.background` 빈 화면. 타이틀 바 이름은 앱 이름 (판단, 소스: project_empty_state.rs, 검토 자료) |
| A-16.2 | 시작할 때 경로가 없다 | 항목을 `text.disabled` 로 그린다(A-12.7). 트리 자리에 `This folder no longer exists.` 와 Remove 버튼. 탭 상태는 보관하되 열지 않는다 |
| A-16.3 | 실행 중 폴더가 사라졌다 | A-16.2 와 같다. 열린 탭은 dirty 이면 저장 질문 없이 유지하고, 저장하려 하면 오류를 보인다 (판단) |
| A-16.4 | 읽을 수 없는 폴더를 고름 | 선택 창이 무엇을 막는지는 OS·프레임워크에 달려 있어 근거가 없다. 앱이 직접 검사한다: 경로가 존재하고, 폴더이고, `read_dir` 이 성공하는지. 하나라도 실패하면 등록하지 않고 `Cannot open this folder.` 알림만 보인다 (판단) |
| A-16.5 | 한 폴더가 다른 등록 폴더의 안쪽·바깥 | 허용한다. 독립 항목이다 (판단) |
| A-16.6 | 같은 폴더를 두 번 | A-13.2 |
| A-16.7 | 등록 폴더 수 | 제한 없음. 목록은 세로 스크롤 (판단) |

### 3.8 사이드바 키 (A-17)

이 표의 키는 02 의 키맵 표(`1절, VSCode 덧씌움 적용 뒤)와 같아야 한다 (4.8절). 02 의 표와 다르면 02 를 따른다. Zed 의 `ThreadsSidebar` 컨텍스트 키([default-windows.json L755-L777](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/keymaps/default-windows.json#L755-L777))는 스레드 목록용이라 쓰지 않고, A-17.3 ~ A-17.5 는 폴더 목록에 맞춘 우리 앱 값이다 (판단).

| ID | 키 | 범위 | 동작 | 근거 |
| --- | --- | --- | --- | --- |
| A-17.1 | `Ctrl+Alt+J` | 창 | 사이드바 열기·닫기. 열 때 포커스를 옮기고, 닫을 때 사이드바에 포커스가 있었으면 이전 위치로 돌아간다 | I-024, I-100 |
| A-17.2 | `Ctrl+Alt+;` | 창 | 사이드바 포커스 토글. 닫혀 있으면 열고 포커스한다. 열려 있고 포커스가 있으면 이전 포커스로 돌아가되 닫지 않는다 | I-024, I-099 |
| A-17.3 | `Up` / `Down` | 사이드바 | 항목 이동 (판단) | — |
| A-17.4 | `Enter` | 사이드바 | 선택한 폴더 활성화 (판단) | I-109 |
| A-17.5 | `Delete` | 사이드바 | Remove (A-13.4) (Zed 에 없는 키, 판단) | I-107 |
| A-17.6 | `Ctrl+K Ctrl+O` | 창 | 폴더 추가 (A-13.1) | I-004 |
| A-17.7 | `Ctrl+B` | 창 | 왼쪽 Dock(프로젝트 패널) 표시 토글. 사이드바와 다르다 | I-022, I-098 |
| A-17.8 | `Ctrl+Shift+E` | 창 | 프로젝트 패널 포커스 토글 | I-023, I-098 |

### 3.9 ☰ 메뉴 (A-18)

구조·항목·키·문구는 **02 의 9절(I-300 ~ I-488)이 정한다.** 이 문서는 사용자 결정으로 바뀐 점과 우리 앱 고유 항목만 적는다.

| ID | 항목 | 내용 | 근거 |
| --- | --- | --- | --- |
| A-18.1 | 표시 방식 | ☰ 아이콘 하나가 닫힌 상태다. 누르면 첫 메뉴(앱 이름)가 열리고 그동안 가로 메뉴 바(Zed, File, Edit, …)가 나타난다. 다른 이름에 마우스를 올리면 그 메뉴로 바뀐다. `F10` 으로 연다 | I-302, I-303, I-310 |
| A-18.2 | 하위 메뉴 | 없다. 묶음 안의 항목은 부모 메뉴에 펴서 넣고 구분선은 Zed 그대로 둔다 | I-304, I-306 |
| A-18.3 | 앱 이름 메뉴 | About <앱 이름>(포함, A-42.12), **Settings 묶음**(Open Settings File, Open Keymap, Open Keymap File, Select Theme..., Select Icon Theme...), **Extensions**(확장 화면을 연다, A-58), Quit. 제외: Check for Updates, Open Settings(화면), Open Project Settings, Open Default Settings | I-322 ~ I-341, _decisions.md |
| A-18.4 | File | New, **New Window**, **Open File...**, **Open Folder...**(= 폴더 등록, A-13.1), **Open Recent…**, Save, Save As…, Save All, Close Editor, Close Window. 제외: Add Folder to Project…(한 프로젝트에 루트를 더하는 기능이라 multi-root 이고 우리 모델과 다르다, 판단), Open Remote… | I-342 ~ I-358, I-350 |
| A-18.5 | Edit | Undo, Redo, Cut, Copy, Copy and Trim, Paste, **Find**, **Replace**, Toggle Line Comment | I-359 ~ I-370, I-479 |
| A-18.6 | Selection | Select All, Expand Selection, Shrink Selection(구문 단위 선택 포함), Add Cursor Above/Below, Select Next/Previous Occurrence, Select All Occurrences, Move Line Up/Down, Duplicate Selection | I-371 ~ I-385 |
| A-18.7 | View | Zoom In/Out, Reset Zoom, Toggle Left Dock, **Toggle All Docks**, Toggle Folder Sidebar †, Focus Folder Sidebar †, Split Up/Down/Left/Right, Maximize Pane †, Project Panel, Open Markdown Preview † | I-386 ~ I-401, I-472 ~ I-476 |
| A-18.8 | Go | Back, Forward, Previous Tab †, Next Tab †, **Command Palette...**, **Go to File...**, Go to Line/Column... | I-413 ~ I-420, I-477, I-478 |
| A-18.9 | Window, Help | Minimize, Zoom. Help 에는 View Dependency Licenses 정도만 남긴다 (이 항목은 03 의 7절 고지 파일을 보여 주는 용도로 연결, 판단) | I-445, I-446, I-450 |
| A-18.10 | 우리 앱 문구 | † 는 02 가 우리 앱 기능에 맞게 추가한 항목이다. `Zed` 가 들어간 문구는 앱 이름으로 바꾼다 | 02 9.2 |
| A-18.11 | 단축키 표시 | 02 9.2 의 열. VSCode 적용 결과다 (4.8절) | I-307 |
| A-18.12 | 02 9.4 의 결정 대기 항목 | _decisions.md 가 해소했다: Find, Replace, Command Palette, Go to File, Open Recent, Settings 파일, Keymap(파일·편집 화면), 핀 탭, 새 창, Open File, About, Toggle All Docks, 구문 단위 선택은 **포함**. 프로젝트 검색·Settings 화면·Check for Updates 는 **제외** | _decisions.md |

### 3.10 ∨ 사용자 메뉴 (A-19)

사용자 요건: Sign In 버튼은 없애고 옆의 `∨` 드롭다운은 둔다. 항목은 우리 앱에서 쓸 수 있는 것만 남긴다. 조사는 I-600 ~ I-650 (작성 중)이다.

| ID | 항목 | 값 | 근거 |
| --- | --- | --- | --- |
| A-19.1 | 자리 | 타이틀 바 오른쪽, 창 버튼 바로 앞. 그룹 gap 4, 오른쪽 padding 4. Sign In 이 있던 자리를 비워 두지 않는다 | I-603 |
| A-19.2 | 버튼 | `ChevronDown` 아이콘 14, 버튼 22×22(14 + 좌우 4), 모서리 4, 스타일 Subtle(배경 `ghost_element.*`). 접근성 이름 `User menu` | I-601, I-608, I-609 |
| A-19.3 | 툴팁 | 없다 (Zed 가 지정하지 않는다) | I-637 |
| A-19.4 | 메뉴 열림 | 버튼이 선택 상태가 된다. 열린 상태에서 버튼을 다시 누르면 닫고 곧바로 다시 열리지 않는다. 메뉴가 닫히면 이전 포커스로 돌아간다 | I-610 |
| A-19.5 | 메뉴 위치 | 오른쪽 위를 버튼 오른쪽 아래에 붙이고 x 로 +5. 창 안쪽 여백 8 | I-611 |
| A-19.6 | 메뉴 모양 | 최소 폭 200, 최대 높이 창의 75%, 넘치면 세로 스크롤. 바탕·테두리·그림자는 ☰ 메뉴와 같다 | I-612 |
| A-19.7 | 항목 (확정) | **Settings**(= `settings.json` 파일을 탭으로 연다, A-42.5), **Keymap**(= 키맵 편집 화면, A-42.7, 키 `Ctrl+K Ctrl+S`), **Themes…**, **Icon Themes…**, **Extensions**(확장 화면, `Ctrl+Shift+X`, A-58). ☰ 메뉴의 같은 명령과 같은 상태를 쓴다 (선택기를 따로 만들지 않는다) | I-620 ~ I-624, I-647 ~ I-649, _decisions.md |
| A-19.8 | Settings 항목의 동작 | 별도 Settings 화면을 만들지 않는다. 항목을 누르면 `settings.json` 이 편집기 탭으로 열린다 (이전 판의 "화면 범위 미정" 은 결정으로 해소됐다) | _decisions.md |
| A-19.9 | 항목 순서 | Settings, Keymap, Themes…, Icon Themes…, Extensions (원본 순서, 구분선 없음) | I-620 ~ I-624, I-650 |
| A-19.10 | 뺀 것 | 사용자 이름, 업데이트 안내(Restart to update), Organization, Panel Layout(Classic·Agentic·Custom), Sign Out. 로그인·AI 가 없다 | I-613 ~ I-619, I-625 ~ I-631, I-635 |
| A-19.11 | 키 표시 | 메뉴 오른쪽에 키를 보인다. 조사한 기본 keymap 에서는 Settings `Ctrl+,`, Keymap `Ctrl+K Ctrl+S`, Themes… `Ctrl+K Ctrl+T` 이고 Icon Themes… 는 키가 없다. VSCode 적용(4.8절) 뒤 바뀔 수 있다 | I-632, I-633 |

### 3.11 그 밖의 포함 기능 (A-43)

사용자 결정으로 포함된 기능의 우리 앱 규칙이다. Zed 의 동작·치수·문구는 02 와 조사 대기 목록이 정한다. 아래는 이 문서가 정할 수 있는 것만 적는다.

| ID | 기능 | 규칙 | 근거 |
| --- | --- | --- | --- |
| A-43.1 | 새 창 | `Ctrl+Shift+N` 은 **빈 창**을 연다(등록 폴더 없음). 창마다 폴더 목록·활성 폴더·탭·Dock·사이드바 상태가 따로다 (4.3절의 `windows[]`). 한 폴더를 두 창에 등록해도 된다. 같은 파일을 두 창에서 편집하는 충돌은 디스크 변경 감지(A-44.6)에 맡긴다 | I-343, _decisions.md |
| A-43.2 | Open File | `Ctrl+O`. 파일 선택 창에서 고른 파일을 **활성 폴더의 탭 영역**에 연다. 폴더가 없으면 빈 폴더 없는 작업 영역(`workspaces["_loose"]`)에 연다. 탭의 `path_kind` 는 `absolute` 다 (4.3절) | I-345, I-003 |
| A-43.3 | Open Recent | 최근 폴더 이력은 `recent` 에 저장한다(4.3절 A-22.23). 이력에 넣는 때는 폴더를 **등록하거나 활성화**했을 때다. 최근 파일은 이력에 넣지 않는다 (판단). 목록 화면은 P-001 ~ 조사 대기 | I-347 |
| A-43.4 | Command Palette | `Ctrl+Shift+P`, `F1`. 우리 앱에 있는 명령만 보인다. 명령 목록은 메뉴에 있는 액션과 같다 (판단). 모양·검색·키 표시는 P-001 ~ 조사 대기 | I-416, I-028 |
| A-43.5 | Go to File | `Ctrl+P`(VSCode 덧씌움 결과는 02 `1). 활성 폴더의 파일을 이름으로 찾아 연다. 같은 폴더 밖 파일은 찾지 않는다 | I-418, I-007 |
| A-43.6 | 찾기·바꾸기 | 편집기 위쪽 찾기 막대. 옵션: 대소문자, 단어, 정규식. `seed_search_query_from_cursor` 기본 `always` (A-21.20). 막대 모양·결과 수·이전/다음 키는 E-001 ~ 조사 대기. 마크다운 미리보기 탭에서는 열지 않는다 (판단) | I-367, I-479, 검토 자료 |
| A-43.7 | settings.json 탭 | `Open Settings File` 은 `%APPDATA%\ZedLike\settings.json` 을 **일반 편집기 탭**으로 연다. 저장하면 앱이 바로 다시 읽는다 (A-23.7). 파일이 깨지면 알림(A-45)을 보이고 마지막으로 읽은 값을 유지한다 | I-327, _decisions.md |
| A-43.8 | keymap.json 탭 | `Open Keymap File` 은 `%APPDATA%\ZedLike\keymap.json` 을 탭으로 연다. 형식은 4.1절 A-20.4. 저장하면 바로 다시 읽는다. 파일이 없으면 `[]` 로 만든다 | I-333 |
| A-43.9 | 키맵 편집 화면 | `Ctrl+K Ctrl+S`. Zed Keymap Editor 와 같게 한다. 화면 구성·열·검색·충돌 표시·키 입력 기록·치수·색은 K-001 ~ 조사 대기. 저장은 `keymap.json` 에 한다 | I-332, _decisions.md |
| A-43.10 | About | 앱 이름·버전·저작권·고지(03 의 7절)를 보인다. 크기·버튼·문구는 P-001 ~ 조사 대기 | I-322 |
| A-43.11 | Toggle All Docks | `Ctrl+Shift+Y`. 왼쪽 Dock 을 열고 닫는다. 사이드바는 건드리지 않는다 (I-483). `Ctrl+B` 와 결과가 같지만 둘 다 메뉴에 둔다 | I-394, I-483 |
| A-43.12 | 구문 단위 선택 | Expand / Shrink Selection, Select Next / Previous Sibling. 문법(Tree-sitter)을 지원하는 언어에서만 동작하고 지원하지 않으면 아무 일도 하지 않는다. 키는 VSCode 적용 결과 (02 `1) | I-372, I-373 |
| A-43.13 | 핀 탭 | A-42.8. 상태 복원 포함 (A-25.3). 핀은 Pane 앞쪽 연속 구간이다 (I-719) | I-065, I-719 |

### 3.12 내장 뷰어 등록 구조 (A-47)

**뷰어**는 파일의 원문 편집기 대신(또는 곁에) 그 파일을 다른 모양으로 보여 주는 탭이다. 뷰어는 **본체에 내장**하고 확장으로 더하지 않는다(사용자 결정). 새 뷰어는 본체를 고쳐 다시 빌드해서 더한다. 이 절은 그 등록 구조를 정한다. 구조는 우리 설계(판단)이고, 마크다운 미리보기 값은 Zed 소스에서 확인한 것(01·02 의 ID)이다.

#### 3.12.1 등록 항목의 필드 (A-47.1)

뷰어 하나는 등록 항목 `ViewerSpec` 하나다. 앱이 시작할 때 코드로 등록한다.

| ID | 필드 | 뜻 | 마크다운 미리보기 | 이미지 뷰어 |
| --- | --- | --- | --- | --- |
| A-47.1a | `id` | 탭 종류 이름. `state.json` 의 `pane.tabs[].kind` 에 저장되는 값이다 | `markdown_preview` | `image_viewer` |
| A-47.1b | `matchers` | 이 뷰어가 맡는 파일: 확장자(소문자, 점 제외)와 MIME 의 목록 | 확장자 `md`, `markdown` 등. 목록은 I-044 의 `extension == md` 조건과 같다 (정확한 목록은 02 I-044 가 정한다) | 이미지 확장자·MIME. **목록은 조사 대기** (A-47.6) |
| A-47.1c | `open_mode` | 파일을 열 때의 방식: `replace`(편집기 대신 뷰어만), `companion`(편집기는 그대로 두고 뷰어 탭을 더함) | `companion`. 원문 편집기가 있고 미리보기는 명령으로 더한다 (I-044, I-077) | **조사 대기.** 편집기가 없는 형식이라 `replace` 일 가능성이 높다 (추정) |
| A-47.1d | `title` | 탭 제목을 만드는 규칙 | `Preview {파일명}`, 24자에서 말줄임. 원문이 없으면 `Markdown Preview` (L-69, I-080) | 조사 대기 (파일 이름일 것으로 추정) |
| A-47.1e | `icon` | 탭 아이콘 | `FileDoc`(책 모양, T-10.12). **항상** 붙는다 (L-69) | 조사 대기 |
| A-47.1f | `tooltip` | 탭 위에 마우스를 올렸을 때 | **원본 편집기의 툴팁 그대로**(원본 파일 전체 경로, L-239) | 조사 대기 |
| A-47.1g | `tab_menu` | 탭 우클릭 메뉴 | 02 의 11.3절(미리보기 탭 항목) | 조사 대기 |
| A-47.1h | `body_menu` | 본문 우클릭 메뉴 | 02 의 11.5절 (Copy, Copy as Markdown, Copy Link, 조건이 없으면 열지 않음) | 조사 대기 |
| A-47.1i | `commands` | 뷰어를 여는·닫는 명령과 키 | `Ctrl+Shift+V`(같은 Pane), `Ctrl+K V`(옆 Pane), 미리보기에서 `Ctrl+Shift+V` 는 원문으로 돌아가기 (I-044, I-045, I-086). ☰ View 의 Open Markdown Preview(I-474, I-475) | 조사 대기 |
| A-47.1j | `source` | 다른 항목에 묶이는지 | 원본 버퍼에 묶인다. 같은 Pane 에 같은 원본의 미리보기가 있으면 새로 만들지 않고 그 탭을 활성화한다 (I-079). 내용은 원본 버퍼의 현재 내용으로 실시간 갱신한다 (I-082) | 파일 자체. 원본 버퍼 없음 (추정) |
| A-47.1k | `serializable` | 상태 복원 대상인가 | 예 (I-114). 복원 중에는 "자동 미리보기 열기" 가 동작하지 않는다 | 조사 대기 |
| A-47.1l | `view_state` | 뷰어가 저장하는 화면 상태(스크롤·확대 등) | 저장하지 않는다 (I-083 의 스크롤 동기화는 약속하지 않는다) | 조사 대기 (확대·위치) |
| A-47.1m | `preview_tab` | 임시(preview) 탭 규칙과의 관계 | A-47.3 | 조사 대기 |

#### 3.12.2 파일을 뷰어로 정하는 규칙 (A-47.2)

| ID | 규칙 | 내용 |
| --- | --- | --- |
| A-47.2a | 매칭 | 파일 이름의 확장자를 소문자로 바꿔 각 `matchers` 와 비교한다. MIME 은 확장자로 알 수 없을 때만 본다 (판단) |
| A-47.2b | 여럿이 맞을 때 | 등록 순서가 앞선 뷰어가 이긴다. 지금은 겹치는 확장자가 없다 (판단) |
| A-47.2c | `replace` 뷰어 | 프로젝트 패널·Go to File·Open File 에서 파일을 열면 원문 편집기가 아니라 그 뷰어 탭이 열린다. 원문으로 보는 명령(Open As Text 같은 것)은 Zed 에 있는지 확인하지 못했다 (A-40.20) |
| A-47.2d | `companion` 뷰어 | 파일을 열면 원문 편집기가 열린다. 뷰어는 명령(키·메뉴·우클릭)으로 더한다. 이 설정을 바꿔 자동으로 뷰어를 열게 할 수 있다: `markdown_preview.open_markdown_files_in_preview` (Zed 기본 false, I-081). 우리 앱은 이 키를 만들지 않고 false 로 고정한다 (판단) |
| A-47.2e | 등록되지 않은 종류 | 일반 원문 편집기로 연다 |
| A-47.2f | 열 수 없는 파일 | 뷰어가 읽기에 실패하면 탭을 만들지 않고 알림(A-45.1)을 보인다 (판단) |

#### 3.12.3 임시(preview) 탭 규칙과의 관계 (A-47.3)

이름이 겹치니 구분한다 (I-060).

| ID | 개념 | 뜻 | 규칙 |
| --- | --- | --- | --- |
| A-47.3a | 임시(preview) 탭 | 프로젝트 패널에서 한 번 클릭으로 연 파일이 다음 탐색 때 교체되는 상태. 제목이 기울임꼴 (I-061, I-062, I-064) | Pane 마다 하나. 편집이나 더블클릭으로 일반 탭이 된다 |
| A-47.3b | "Preview X" 탭 | **뷰어 종류**(`markdown_preview`). 임시 탭 상태와 별개다 (I-060) | 일반 탭이다. 임시 탭을 교체하지 않는다 (판단, I-060) |
| A-47.3c | 뷰어 탭의 `preview` 필드 | `state.json` 의 `pane.tabs[].preview` | 마크다운 미리보기 탭은 항상 `false` 다. `replace` 뷰어(이미지)가 임시 탭이 될 수 있는지는 조사 대기 (A-47.6) |
| A-47.3d | 핀 | 뷰어 탭도 핀할 수 있다. 핀은 Pane 앞쪽 연속 구간의 경계다 (I-719) | 같음 |
| A-47.3e | 분할·이동 | 뷰어 탭도 드래그로 옮기고 분할할 수 있다. 복제 가능 여부는 뷰어마다 다르다 (I-718). 마크다운 미리보기를 복제하면 같은 원본의 두 번째 미리보기가 된다 (I-079 의 "같은 Pane 재사용" 은 Pane 안에서만) | 판단 |

#### 3.12.4 뷰어와 나머지 기능의 연동 (A-47.4)

| ID | 연동 | 규칙 |
| --- | --- | --- |
| A-47.4a | 탭 바 | 제목·아이콘·툴팁은 `ViewerSpec` 이 만든다. 수정 표시(●)는 원본 버퍼가 있는 뷰어에서 원본의 상태를 따르지 않는다(미리보기 탭에는 점이 없다, 판단) |
| A-47.4b | 상태 표시줄 | 미리보기 탭이 활성이어도 커서 위치는 원본 편집기의 위치를 보인다 (기준 화면 두 장: `9:11`, `1:1`, T-08.22) |
| A-47.4c | 툴팁 | A-47.1f. 일반 규칙은 A-34 (지연 500ms 등) 와 같다. 탭 본체 툴팁은 01 의 L-238, L-239 |
| A-47.4d | 우클릭 메뉴 | A-47.1g, A-47.1h. 항목·문구는 02 11절 |
| A-47.4e | 상태 복원 | `serializable` 이 참인 뷰어만 `pane.tabs[]` 에 저장한다 (4.3절). 저장한 `kind` 가 등록되어 있지 않으면(예: 뷰어를 뺀 버전) 그 탭을 건너뛴다 (A-25.5) |
| A-47.4f | 명령 팔레트 | `commands` 의 액션은 메뉴·팔레트에 같은 이름으로 나타난다 (A-43.4) |
| A-47.4g | 파일 아이콘 | 프로젝트 패널의 파일 아이콘은 아이콘 테마가 정한다. 뷰어와 무관하다 (T-22) |

#### 3.12.5 첫째 사례: 마크다운 미리보기 (A-47.5)

| ID | 항목 | 규칙 | 근거 |
| --- | --- | --- | --- |
| A-47.5a | 여는 법 | `Ctrl+Shift+V` = 같은 Pane, `Ctrl+K V` = 옆 Pane(없으면 오른쪽에 분할) | I-044, I-077, I-078 |
| A-47.5b | 중복 | 같은 Pane 에 같은 원본의 기본 모드 미리보기가 있으면 그 탭을 활성화한다 | I-079 |
| A-47.5c | 제목·아이콘·툴팁 | `Preview {파일명}` / `FileDoc` / 원본의 툴팁 | L-69, L-239, T-10.12 |
| A-47.5d | 내용 | 원본 버퍼의 편집·더티 변화를 관찰해 실시간 갱신한다 | I-082 |
| A-47.5e | 본문 클릭 | 원문 위치 매핑이 있는 본문을 누르면 원문 선택 위치를 옮긴다 | I-084 |
| A-47.5f | 링크 | 로컬 마크다운 링크는 기본 렌더링 미리보기로 연다 | I-085 |
| A-47.5g | 원문으로 | 미리보기에서 `Ctrl+Shift+V` 는 원문으로 돌아간다. 원문 탭이 닫혔어도 원문 편집기를 다시 연다 | I-086 |
| A-47.5h | 스타일 | 본문 폭·여백·제목·표·인용·frontmatter 표는 01 의 5절(L-101 ~ L-111) | L-101 ~ L-111 |
| A-47.5i | 복원 | 저장·복원한다. 복원 중에는 자동 대체를 하지 않는다 | I-114 |

#### 3.12.6 둘째 사례: 이미지 뷰어 (A-47.6)

Zed 에는 이미지를 보여 주는 내장 뷰어가 있다. 기준 화면의 프로젝트 트리에도 이미지 파일(`zed-reference.png`)이 있다. **동작·치수·문구는 아직 조사하지 않았다.** 아래는 이 구조에 넣을 때 채울 칸이다.

| ID | 채울 칸 | 상태 |
| --- | --- | --- |
| A-47.6a | 확장자·MIME 목록(png, jpg, gif, webp, svg 등 어느 것까지) | 조사 대기 |
| A-47.6b | 열기 방식(`replace` 인지), 임시 탭이 되는지 | 조사 대기 |
| A-47.6c | 탭 제목·아이콘·툴팁 | 조사 대기. 기준 화면의 트리에서 이미지 파일 아이콘은 Catppuccin Icons 의 `image` 로 보인다 (T-20) |
| A-47.6d | 확대·축소·맞춤, 배경(체커보드 등), 크기 정보 표시 | 조사 대기 |
| A-47.6e | 우클릭 메뉴, 키 | 조사 대기 |
| A-47.6f | 상태 복원 | 조사 대기 |
| A-47.6g | 마크다운 미리보기 안의 이미지 표시(상대 경로 해석)는 마크다운 뷰어의 일이다 | 02 4절 미조사 항목 |

#### 3.12.7 향후 본체 기능 (A-47.9)

| ID | 뷰어 | 상태 |
| --- | --- | --- |
| A-47.9a | PDF 뷰어 | 이번 범위에 넣지 않는다. 필요해지면 **본체에 구현하고 다시 빌드**해서 `ViewerSpec` 으로 등록한다. 확장으로 만들지 않는다 (사용자 결정) |
| A-47.9b | 동영상 뷰어 | 위와 같다 |
| A-47.9c | 등록 구조의 쓰임 | 위 둘은 이 절의 필드(`matchers`, `open_mode`, 제목·아이콘·툴팁, 메뉴, 복원)를 그대로 채우면 된다. 새 필드가 필요하면 이 표에 더한다 |

## 4. 데이터 모델과 설정 파일

### 4.1 파일과 위치 (A-20)

Zed 는 설정(`settings.json`)과 작업 상태(DB)를 나눈다 (I-116, I-112 ~ I-119). 우리도 나눈다.

| ID | 파일 | 위치 (Windows) | 누가 쓰나 | 내용 |
| --- | --- | --- | --- | --- |
| A-20.1 | `settings.json` | `%APPDATA%\ZedLike\settings.json` | 사용자와 앱 | 테마·글꼴 등 Zed 와 같은 이름의 설정 키 |
| A-20.2 | `state.json` | `%APPDATA%\ZedLike\state.json` | 앱만 | 등록 폴더, 활성 폴더, 열린 탭, Dock 폭, 창 위치 |
| A-20.4 | `keymap.json` | `%APPDATA%\ZedLike\keymap.json` | 사용자와 앱(키맵 편집 화면) | 형식은 Zed 와 같다: 객체 배열 `[{ "context": "Editor", "bindings": { "ctrl-k ctrl-t": "theme_selector::Toggle" } }]`. 바인딩 값은 우리 앱에 있는 액션 이름만 쓴다. 이 파일의 바인딩이 VSCode 덧씌움 위에 마지막으로 얹힌다 (A-27.1) |
| A-20.5 | `unsaved/` | `%APPDATA%\ZedLike\unsaved\` | 앱만 | 저장 안 한 버퍼의 본문(A-22.25). 폴더 안에 `{id}.txt` 로 둔다 |
| A-20.6 | `extensions/` | `%LOCALAPPDATA%\ZedLike\extensions\` | 앱 | 확장 루트. 설정(`%APPDATA%`)과 데이터(`%LOCALAPPDATA%`)를 나누는 Zed 의 배치와 같다 (A-50.1) |
| A-20.7 | `extensions/index.json` | 위 폴더 안 | 앱만 | 설치된 확장·테마·아이콘 테마·언어의 색인 (A-54). 지워도 다시 만든다 |
| A-20.3 | 앱 이름 | `ZedLike` 는 임시 이름이다. 구현 때 바꾼다 | — | — |

- 두 구현(gpui-kit, Tauri)은 **같은 파일 형식**을 읽고 쓴다. 한 구현이 쓴 파일을 다른 구현이 열 수 있어야 한다 (판단).
- 형식은 JSON 이다. Zed 의 `settings.json` 은 주석을 허용하지만 우리는 허용하지 않는다 (판단, 파서 부담을 줄이려고). 사용자가 직접 편집하는 `settings.json`·`keymap.json` 은 끝의 쉼표도 허용하지 않는다. 읽기 오류는 A-45 의 알림으로 알린다.
- 알 수 없는 키는 읽을 때 무시하고, 쓸 때 보존한다.

### 4.2 settings.json (A-21)

앱이 구현한 설정만 둔다. 키 이름과 기본값은 Zed 와 같고, 다른 것은 아래 표의 "Zed 와" 열에 **우리 앱 값** 또는 **우리 키** 로 적었다. 기본값 출처는 L-120 ~ L-139 와 [default.json](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json) 이다.

```json
{
  "theme": { "mode": "system", "light": "One Light", "dark": "One Dark" },
  "icon_theme": "Zed (Default)",
  "ui_font_family": ".ZedSans",
  "ui_font_size": 16,
  "buffer_font_family": ".ZedMono",
  "buffer_font_size": 15,
  "buffer_line_height": "comfortable",
  "autosave": "off",
  "tab_size": 4,
  "hard_tabs": false,
  "auto_indent": "syntax_aware",
  "use_autoclose": true,
  "remove_trailing_whitespace_on_save": true,
  "ensure_final_newline_on_save": true,
  "format_on_save": "off",
  "cursor_blink": true,
  "cursor_shape": "bar",
  "current_line_highlight": "all",
  "show_whitespaces": "selection",
  "soft_wrap": "none",
  "scroll_beyond_last_line": "one_page",
  "search": { "whole_word": false, "case_sensitive": false, "regex": false },
  "seed_search_query_from_cursor": "always",
  "sidebar": { "default_width": 300 },
  "project_panel": {
    "default_width": 240,
    "indent_size": 20,
    "auto_fold_dirs": true,
    "folder_indicator": "icon"
  },
  "tabs": { "close_position": "right", "show_close_button": "hover", "file_icons": false },
  "preview_tabs": { "enabled": true, "enable_preview_from_project_panel": true },
  "markdown_preview": { "limit_content_width": true, "max_width": 800 }
}
```

기준 화면의 사용자 설정은 위에서 아래만 다르다 (T-08, README).

```json
{
  "theme": { "mode": "system", "light": "One Light", "dark": "Catppuccin Mocha" },
  "icon_theme": "Catppuccin Latte"
}
```

| ID | 키 | 값 | 기본 | Zed 와 | 뜻 | 참조 |
| --- | --- | --- | --- | --- | --- | --- |
| A-21.1 | `theme` | 문자열 또는 `{ mode, light, dark }`. `mode`: `system`, `light`, `dark` | 위 | 같음 | 활성 테마. 이름은 03 의 6종 | T-21, T-22 |
| A-21.2 | `icon_theme` | 문자열 (또는 같은 `mode` 객체) | `Zed (Default)` | 같음 | 파일·폴더 아이콘 테마. 5종 | T-29 |
| A-21.3 | `ui_font_size` | 숫자 | 16 | 같음 | UI 글자 크기이자 rem 크기. 바꾸면 rem 으로 정한 치수가 비례해 변한다 | L-01, L-02, L-120 |
| A-21.4 | `buffer_font_size` | 숫자 | 15 | 같음 | 편집기 글자 크기 | L-121 |
| A-21.5 | `ui_font_family`, `buffer_font_family` | 글꼴 이름 또는 별칭 | `.ZedSans`, `.ZedMono` | 문자열은 같음. 별칭이 가리키는 글꼴이 우리 앱 값이다 | 별칭 `.ZedSans` = IBM Plex Sans, `.ZedMono` = Lilex (Zed 도 같은 별칭을 쓴다, T-14.1, T-14.2). `.SystemUIFont` 는 시스템 UI 글꼴. 이름이 없으면 03 5.3절의 앱 판단 순서(T-16) | T-14, T-16 |
| A-21.6 | `buffer_line_height` | `comfortable` (약 1.618), `standard` (1.3) | `comfortable` | 같음 | 편집기 줄 높이 | L-121 |
| A-21.7 | `sidebar.default_width` | 200 ~ 800 | 300 | **우리 키.** Zed 에는 `sidebar` 객체가 없고, 같은 값이 `agent.threads_sidebar.default_width` 에 있다 | 사용자가 폭을 저장하기 전의 폭. 300 은 Zed 의 `agent.threads_sidebar.default_width` 기본값([default.json L1191-L1197](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1191-L1197)), 범위 200~800 은 [agent_settings.rs L37-L38](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/agent_settings/src/agent_settings.rs#L37-L38) 의 최소·최대다 | L-33, L-124 |
| A-21.8 | `project_panel.default_width` | 숫자 | 240 | 같음 | 사이드바와 같은 뜻 | L-32, L-125 |
| A-21.9 | `project_panel.indent_size` | 숫자 | 20 | 같음 | 들여쓰기 단계 폭 | L-37, L-126 |
| A-21.10 | `project_panel.auto_fold_dirs` | bool | true | 같음 | 자식이 폴더 하나뿐인 폴더를 한 줄로 합친다 | L-42, L-127 |
| A-21.11 | `project_panel.folder_indicator` | `icon`, `chevron`, `both` | `icon` | 같음 | 폴더 펼침 표시 | L-44, T-10.17 |
| A-21.12 | `tabs.close_position`, `tabs.show_close_button`, `tabs.file_icons` | `right`/`left`, `hover`/`always`/`hidden`, bool | `right`, `hover`, `false` | 같음 | 탭 닫기 버튼·아이콘 | L-131 |
| A-21.13 | `preview_tabs.enabled`, `preview_tabs.enable_preview_from_project_panel` | bool | true, true | 같음 | 임시(preview) 탭 | L-132, I-061 |
| A-21.14 | `markdown_preview.limit_content_width`, `max_width` | bool, 숫자 | true, 800 | 같음 | 미리보기 본문 폭 | L-136, L-101 |
| A-21.15 | `autosave` | `off` \| `on_focus_change` \| `on_window_change` \| `after_delay` | `off` | 같음 | 기본은 자동 저장이 없다. 그래서 수정하면 탭에 ●(Accent)가 붙고 닫을 때 저장 확인이 뜬다 (I-069, I-071). 값을 바꾸면 그 동작이 따른다 | [default.json L1471](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L1471), I-069 |
| A-21.16 | 저장할 때의 변환 | `remove_trailing_whitespace_on_save` true, `ensure_final_newline_on_save` true, `format_on_save` off | 위 | 같음 | 저장할 때 줄 끝 공백을 지우고 마지막 줄바꿈을 보장한다. 같은 파일을 저장한 결과가 Zed 와 같아야 한다 | default.json L1617, L1626, L1653 |
| A-21.17 | 들여쓰기·입력 | `tab_size` 4, `hard_tabs` false, `auto_indent` `syntax_aware`, `use_autoclose` true | 위 | 같음 | 편집기 기본 입력 규칙 | default.json L1701, L1699, L524, L515 |
| A-21.18 | 커서·줄 표시 | `cursor_blink` true, `cursor_shape` `bar`, `current_line_highlight` `all`, `show_whitespaces` `selection` | 위 | 같음 | 모양·치수는 E-001 ~ 조사 대기 | default.json L295, L307, L359, L583 |
| A-21.19 | 줄 바꿈·스크롤 | `soft_wrap` `none`, `scroll_beyond_last_line` `one_page` | 위 | 같음 | 줄 바꿈 없음, 마지막 줄 아래로 한 쪽 분량 | default.json L1687, L792 |
| A-21.20 | 찾기 | `search.whole_word` false, `search.case_sensitive` false, `search.regex` false, `seed_search_query_from_cursor` `always` | 위 | 같음 | 찾기 막대의 처음 옵션 | default.json L817-L842 |



| A-21.21 | `auto_install_extensions`, `auto_update_extensions` | — | **만들지 않는다** | **Zed 와 다름.** Zed 기본은 `{"html": true}` 와 `{}` (X-42) | 설치가 로컬뿐이라 자동 설치·자동 업데이트가 없다 (결정: 로컬 설치만). 설정에도 키를 두지 않는다 | [default.json L2366](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2366), X-42, _decisions.md |

값이 범위를 벗어나면 가까운 값으로 맞춘다. 위 키 밖의 Zed 설정(`window_decorations`, `bottom_dock_layout` 등)은 만들지 않고 기본값을 상수로 고정한다 (A-05, 판단). `base_keymap` 키도 만들지 않는다. 적용 keymap 은 VSCode 덧씌움 결과로 고정한다 (4.8절).

### 4.3 state.json (A-22)

앱이 쓰는 작업 상태다. **창마다** 상태가 따로이므로 최상위에 `windows[]` 가 있다. 예시는 창 하나에 폴더 둘, 첫 폴더에 Pane 둘(왼쪽: 탭 셋 중 핀 하나, 오른쪽: 탭 하나)이 있고, 저장 안 한 버퍼가 하나 있는 경우다. 분할은 필수라서 폴더의 탭 영역은 항상 `layout` 트리다. 분할이 없으면 `layout` 이 `pane` 하나다.

```json
{
  "version": 1,
  "recent": [
    { "path": "D:\\projects", "last_opened": "2026-10-31T09:12:44+09:00" },
    { "path": "D:\\work\\notes", "last_opened": "2026-10-30T18:03:10+09:00" }
  ],
  "windows": [
    {
      "id": "w1",
      "window": { "x": 120, "y": 80, "width": 1440, "height": 852, "maximized": false },
      "folders": [
        { "id": "7c1e2b94-5a30-4f0e-9d3b-2f6a8c1d4e10", "path": "D:\\projects", "added_at": "2026-10-31T09:12:44+09:00" },
        { "id": "e04b7a62-91cd-4b58-8a77-0c3d5f2e9a31", "path": "D:\\work\\notes", "added_at": "2026-10-31T09:15:02+09:00" }
      ],
      "active_folder_id": "7c1e2b94-5a30-4f0e-9d3b-2f6a8c1d4e10",
      "sidebar": { "open": true, "width": 300 },
      "left_dock": { "open": true, "width": 285 },
      "workspaces": {
        "7c1e2b94-5a30-4f0e-9d3b-2f6a8c1d4e10": {
          "layout": {
            "type": "split",
            "axis": "horizontal",
            "flexes": [1.0, 1.0],
            "children": [
              {
                "type": "pane",
                "id": "p1",
                "tabs": [
                  { "kind": "editor",           "path": "docs\\guides\\README.md", "path_kind": "relative", "preview": false, "pinned": true },
                  { "kind": "markdown_preview", "path": "docs\\guides\\README.md", "path_kind": "relative", "preview": false, "pinned": false },
                  { "kind": "editor",           "path": "docs\\guides\\notes.md",  "path_kind": "relative", "preview": false, "pinned": false, "unsaved": "u-3f9a" }
                ],
                "active": 1,
                "pinned_count": 1
              },
              {
                "type": "pane",
                "id": "p2",
                "tabs": [
                  { "kind": "editor", "path": "docs\\guides\\rust-gui\\05-SLINT.md", "path_kind": "relative", "preview": true, "pinned": false }
                ],
                "active": 0,
                "pinned_count": 0
              }
            ]
          },
          "active_pane": "p2",
          "zoomed_pane": null
        }
      }
    }
  ]
}
```

| ID | 필드 | 타입 | 뜻 |
| --- | --- | --- | --- |
| A-22.1 | `version` | 정수 | 파일 형식 버전. 지금 1 |
| A-22.2 | `windows[].folders[]` | 배열 | 창의 등록 폴더. **배열 순서가 사이드바 표시 순서**다. Move Up / Move Down 은 이 배열의 두 칸을 맞바꾼다 (A-13.6) |
| A-22.3 | `folders[].id` | 문자열(UUID v4) | 폴더 식별자. 한 번 정하면 바뀌지 않는다. `workspaces` 의 키 |
| A-22.4 | `folders[].path` | 문자열 | 정규화한 절대 경로. 규칙과 예는 4.10절(A-28). 비교는 정규화한 값을 대소문자 무시로 한다 (A-13.2) |
| A-22.5 | `folders[].added_at` | ISO 8601 | 등록한 시각. 화면에는 쓰지 않는다 (판단) |
| A-22.6 | `windows[].active_folder_id` | 문자열 또는 `null` | 창의 활성 폴더. `folders` 에 없으면 첫 폴더, 폴더가 없으면 `null` |
| A-22.7 | `sidebar.open`, `sidebar.width` | bool, 숫자 | 사이드바 열림·폭 (L-33, I-119). 창마다 따로 |
| A-22.8 | `left_dock.open`, `left_dock.width` | bool, 숫자 | 왼쪽 Dock 열림·폭 (L-32, I-115, I-116). 폭은 폴더마다가 아니라 창마다 하나 (판단) |
| A-22.9 | `windows[].window` | 객체 | 창 위치·크기·최대화. 화면 밖이면 복원하지 않는다. 최대화하지 않은 창의 위치·크기를 `maximized: true` 일 때도 보존한다 (판단) |
| A-22.10 | `workspaces.{id}.layout` | 노드 | 폴더의 탭 영역 트리. 노드는 `pane` 또는 `split` |
| A-22.11 | `pane.id` | 문자열 | Pane 식별자. 폴더 안에서 유일. `active_pane`·`zoomed_pane` 이 가리킨다 |
| A-22.12 | `pane.tabs[].kind` | 문자열 | `editor`(원문 편집) 또는 **등록된 뷰어의 `id`**(`markdown_preview`, `image_viewer` 등, 3.12절 A-47.1a), 또는 `extensions_page`(확장 화면, A-58). 등록되어 있지 않은 값이면 그 탭을 건너뛴다 (I-114). `extensions_page` 는 복원하지 않는다 (판단) |
| A-22.13 | `pane.tabs[].path`, `path_kind` | 문자열, `relative` \| `absolute` | `relative` 는 **폴더 루트 기준 상대 경로**로, 폴더를 옮겨도 열 수 있다. `absolute` 는 Open File 로 연 폴더 밖 파일, `settings.json`·`keymap.json` 탭이다 (판단) |
| A-22.14 | `pane.tabs[].preview` | bool | 임시(preview) 탭. **Pane 마다** 하나만 true (I-062, I-113) |
| A-22.15 | `pane.tabs[].pinned` | bool | 핀 탭. `pinned_count` 만큼 앞쪽에 모인다 (I-065, I-719) |
| A-22.16 | `pane.active`, `pane.pinned_count` | 정수 | 활성 탭의 위치, 핀 탭 수 (I-113) |
| A-22.17 | `split.axis` | `horizontal` \| `vertical` | 자식이 놓이는 방향. `horizontal` 은 좌우로 나란히 (I-723) |
| A-22.18 | `split.flexes` | 숫자 배열 | 자식 크기 비율. 길이는 `children` 과 같다. 길이나 합이 올바르지 않으면 모두 1 로 복구한다 (I-724) |
| A-22.19 | `split.children` | 노드 배열 | 둘 이상. 한 자식만 남으면 부모가 그 자식으로 바뀌고 그 그룹의 비율은 초기화한다 (I-728) |
| A-22.20 | `active_pane` | 문자열 | 활성 Pane 의 `id` |
| A-22.21 | `zoomed_pane` | 문자열 또는 `null` | 확대(zoom)한 Pane 의 `id` (L-17, L-73, I-730) |
| A-22.22 | `workspaces["_loose"]` | 노드 | 등록 폴더가 없는 창에서 Open File 로 연 탭을 담는 작업 영역 (A-43.2) |
| A-22.23 | `recent[]` | 배열 | 최근 폴더 이력. 전역(창 공통)이고 최신이 앞이다. 최대 개수는 Zed 에서 확인하지 못했다. 우리는 20개로 한다 (판단). 같은 경로는 한 번만 둔다 |
| A-22.24 | `windows[].id` | 문자열 | 창 식별자. 창을 닫으면 그 창의 항목을 지운다. 마지막 창을 닫을 때(앱 종료)는 지우지 않는다 (복원 대상, A-25.10) |
| A-22.25 | `pane.tabs[].unsaved` | 문자열(선택) | 저장 안 한 버퍼의 본문 파일 이름(`unsaved/{값}.txt`). 있으면 시작할 때 그 본문으로 연다 (A-25.8) |

### 4.4 메모리 모델 (A-23)

두 구현이 같은 이름·같은 규칙을 쓰도록 타입을 적는다. 언어는 정하지 않는다.

| ID | 타입 | 필드 | 규칙 |
| --- | --- | --- | --- |
| A-23.1 | `Folder` | `id`, `path`, `added_at` | `id` 는 불변 |
| A-23.2 | `Workspace` (창·폴더마다 하나) | `folder_id`, `tree_state`, `layout: PaneNode`, `active_pane`, `zoomed_pane` | 폴더를 등록할 때 만들고 해제할 때 버린다 |
| A-23.3 | `TreeState` | `expanded: set<path>`, `selected: path[]`, `scroll` | 저장하지 않는다 (A-25.7) |
| A-23.4 | `PaneNode` = `Pane` \| `Split` | `Pane`: `id`, `tabs: Tab[]`, `active: index`, `pinned_count`, `preview_index: option`. `Split`: `axis`, `flexes: number[]`, `children: PaneNode[]` | 임시 탭은 Pane 마다 최대 하나 (I-062) |
| A-23.5 | `Tab` | `kind`, `path`, `path_kind`, `dirty`, `pinned`, `preview`, `buffer`, `unsaved_ref: option` | `dirty` 는 저장하지 않는다. 본문이 바뀐 상태만 `unsaved_ref` 로 저장한다 |
| A-23.6 | `Window` | `id`, `bounds`, `folders: Folder[]`, `active_folder_id`, `workspaces: map<id, Workspace>`, `sidebar`, `left_dock` | 창마다 하나. 상태가 서로 섞이지 않는다 |
| A-23.7 | `Settings` | 4.2절 키 | 파일이 바뀌면(저장·외부 편집) 다시 읽는다 (판단) |
| A-23.8 | `App` | `windows: Window[]`, `recent: RecentEntry[]`, `settings`, `keymap` | — |
| A-23.9 | `RecentEntry` | `path`, `last_opened` | 최신이 앞. 최대 20개 |

### 4.5 읽기·쓰기 규칙 (A-24)

| ID | 규칙 | 값 |
| --- | --- | --- |
| A-24.1 | 쓰기 순서 | ① 같은 폴더에 `state.json.tmp` 를 새로 만들어 전체를 쓴다. ② 파일을 디스크로 내린다(`sync_all` 에 해당). ③ 기존 `state.json` 이 있으면 `ReplaceFileW`, 없으면 `MoveFileExW`(`MOVEFILE_REPLACE_EXISTING \| MOVEFILE_WRITE_THROUGH`)로 `state.json` 자리에 놓는다. 출처: Microsoft Learn [ReplaceFileW](https://learn.microsoft.com/windows/win32/api/winbase/nf-winbase-replacefilew), [MoveFileExW](https://learn.microsoft.com/windows/win32/api/winbase/nf-winbase-movefileexw). 이 API 를 쓴다는 것은 앱 판단이고, 정전 같은 비정상 종료까지 이 순서가 보장하는지는 검증하지 않았다 |
| A-24.2 | 쓰는 때 | 폴더 등록·해제·순서 변경·활성 변경, 최근 이력 변경, 창 열기·닫기, 저장 안 한 버퍼의 본문 변경(해당 `unsaved/{id}.txt` 만 따로 쓴다), 탭 열기·닫기·핀·활성 변경, 분할·Pane 이동·크기 변경, Dock·사이드바 열림·폭 변경, 창 이동·크기 변경. 변경 뒤 500ms 안에 또 변경이 없으면 쓴다 (판단) |
| A-24.3 | 종료 | 종료 직전에 한 번 더 쓴다 |
| A-24.4 | 교체 실패 | 백신·인덱서가 파일을 잡고 있어 실패할 수 있다. 50ms 간격으로 3번 다시 시도한다. 그래도 실패하면 `state.json.tmp` 를 그대로 두고, 메모리 상태를 유지하고, 알림 한 줄(`Could not save the window state.`)을 보이고, 다음 쓰기 때 처음부터 다시 한다 |
| A-24.5 | 읽을 때 복구 | `state.json` 이 없거나 JSON 이 깨졌고 `state.json.tmp` 가 올바른 JSON 이면 `.tmp` 를 쓴다. 둘 다 안 되면 빈 상태로 시작한다 |
| A-24.6 | 깨진 파일 보관 | 깨진 `state.json` 은 `state.json.bad-YYYYMMDD-HHMMSS` 로 옮긴다. 같은 이름이 있으면 `-2`, `-3` 을 붙인다. 최근 3개만 남기고 오래된 것은 지운다. 알림 한 줄을 보인다 |
| A-24.7 | 버전 | `version` 이 앱보다 크면 읽기만 하고 쓰지 않는다 |
| A-24.8 | 설정 저장 | 테마·아이콘 테마 선택기의 확정만 `settings.json` 을 쓴다(쓰기 순서는 A-24.1 과 같다). 미리 적용은 메모리에서만 바꾼다 (T-21) |

### 4.6 시작할 때 복원 (A-25)

Zed 의 기본은 `restore_on_startup: last_session` 이다 (I-112). 우리는 항상 마지막 상태를 복원한다 (판단).

| ID | 대상 | 규칙 | 근거 |
| --- | --- | --- | --- |
| A-25.1 | 폴더 목록·활성 폴더 | 복원 | I-119 |
| A-25.2 | 탭·분할·핀 | 활성 폴더의 `layout`(분할 구조, 비율, 각 Pane 의 탭)을 복원한다. 다른 폴더는 그 폴더를 처음 활성화할 때 연다 (판단) | I-113 |
| A-25.3 | 임시·핀·활성 Pane·확대 | 복원. 복원할 수 없는 탭을 뺀 뒤 비어 버린 Pane 은 닫고, 한 자식만 남은 `split` 은 그 자식으로 바꾼다. 모든 Pane 이 비면 빈 Pane 하나 | I-113 |
| A-25.4 | 뷰어 탭(마크다운 미리보기 등) | `kind: markdown_preview` 로 복원한다. 복원 중에는 "마크다운을 미리보기로 열기" 같은 자동 대체를 하지 않는다 | I-114 |
| A-25.5 | 존재하지 않는 파일 탭 | 건너뛴다. `active` 는 남은 탭 범위로 맞춘다 | — |
| A-25.6 | Dock·사이드바 | 열림과 폭 복원. 저장된 폭이 없으면 `default_width` | I-115, I-116 |
| A-25.7 | 트리 펼침 | 복원하지 않는다. 루트만 펼쳐 시작하고, 활성 파일이 있으면 그 위치까지 펼친다 (auto reveal) | I-096, I-120 |
| A-25.8 | 저장 안 한 변경 | **Zed 와 같게 복원한다.** 앱을 끌 때 저장 여부를 묻지 않고 본문을 `unsaved/` 에 두었다가 다시 켜면 그 탭에 되살린다 (`session.restore_unsaved_buffers` 기본 true). 탭 하나를 닫을 때의 Save / Don't Save / Cancel 확인은 그대로다 (I-069). 이전 판은 반대로 정했었다 | I-118, [default.json L2727-L2734](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2727-L2734) |
| A-25.10 | 창 | 마지막 세션에 열려 있던 **모든 창**을 복원한다 (`restore_on_startup` `last_session`, I-112). 창 하나만 닫으면 그 창은 복원 대상에서 빠진다 (판단) | I-112 |
| A-25.11 | 최근 이력 | `recent` 는 창과 무관하게 항상 유지한다 | A-22.23 |
| A-25.9 | 파일 다시 열기 시 커서·스크롤 | 같은 세션 안에서만 기억한다 (판단) | I-117 |

### 4.7 탭 분할·이동 (A-26)

사용자 요건: 편집기 탭 영역의 분할·이동(드래그, Pane 그룹, 키)을 Zed 와 같게 한다. **필수**다. 값과 규칙은 02 의 3.1절(I-706 ~ I-739)과 01 의 4.5절이 정한다. 아래는 이 문서가 맡는 연결이다.

| ID | 항목 | 규칙 | 상세 ID |
| --- | --- | --- | --- |
| A-26.1 | 구조 | 루트는 `Pane` 하나이거나 재귀적인 `Split`. 상태는 4.3절 `layout` | I-723, I-724 |
| A-26.2 | 분할 버튼 | 탭 바 오른쪽 끝, 팝업 순서 Split Right → Left → Up → Down. 기본 방식은 ClonePane(활성 항목 하나의 복제본) | I-706, I-707, I-708, L-72, L-244 |
| A-26.3 | 분할 방향 설정 | `pane_split_direction_horizontal`(기본 down), `pane_split_direction_vertical`(기본 right)는 **고정값**으로 둔다. 명시적 Split Right/Left/Up/Down 은 설정과 무관하다 | I-710, L-138 |
| A-26.4 | 드래그 판정 | 가장자리 판정은 탭 바를 뺀 본문의 `min(폭,높이) × 0.2`. 오버레이는 `drop_target.background` 로 본문 전체 또는 해당 쪽 절반 | I-712 ~ I-714 |
| A-26.5 | 탭 위·탭 바 빈 곳 | 탭 위: 그 위치에 넣기(`drop_target.border` 2px 삽입선). 빈 곳: 맨 끝, 분할 안 함 | I-715, I-716, T-26.13 |
| A-26.6 | 이동·복제 | 다른 Pane 으로 이동. Windows 에서 `Ctrl` 을 누르고 놓으면 복제. 드래그한 임시 탭은 일반 탭이 된다 | I-717, I-718 |
| A-26.7 | 핀 경계 | 핀 여부는 새 위치로 정한다 | I-719, I-720 |
| A-26.8 | 프로젝트 패널에서 끌어 놓기 | 선택 항목 하나만. 일반 탭으로 연다. 핀 구역이면 핀 | I-721, I-722 |
| A-26.9 | 분할선 | 두께 1(`pane_group.border`, 없으면 `border`), 마우스 반응 4. 최소 폭 80·높이 100. 더블클릭하면 균등화 | I-725 ~ I-727 |
| A-26.10 | Pane 이 비면 | 그룹에서 빠지고 형제가 하나면 부모를 접는다. 유일한 루트 Pane 은 빈 편집 영역으로 남는다 | I-728 |
| A-26.11 | 활성 Pane 표시 | 따로 강조하지 않는다 (테두리 0, 투명도 1.0) | I-729, L-137 |
| A-26.12 | 확대(zoom) | `Shift+Escape`, 탭 바 최대화 버튼 | I-730, I-737 |
| A-26.13 | 키 | `Ctrl+\`(Split Right), `Ctrl+K` 방향키(Split), `Ctrl+K Ctrl+` 방향키(Pane 포커스). Pane 배치 교환(`Ctrl+K Shift+` 방향키)은 제외 | I-731 ~ I-733 |
| A-26.14 | 실행 대조 | 소스 정적 조사다. 크기 조절 수식, 거리가 같을 때의 방향, 항목 고유 드롭 처리의 세부는 검증 전이다 | I-839 ~ I-843 |

### 4.8 키맵 기준 (A-27)

사용자 요건: Zed 설정이 `base_keymap: VSCode` 이므로 우리 앱의 keymap 은 그 **적용 결과**다. 메뉴·툴팁의 단축키 표시도 같은 결과를 따른다.

| ID | 항목 | 내용 | 상세 ID |
| --- | --- | --- | --- |
| A-27.1 | 적용 순서 | Windows 기본 keymap → VSCode 덧씌움 → 우리 `keymap.json` (A-20.4). 나중 것이 이긴다 | 02 1.1절 |
| A-27.2 | 우리 앱 | `base_keymap` 설정을 만들지 않는다. 항상 VSCode 덧씌움 결과를 쓴다 (판단) | — |
| A-27.3 | 이 문서의 키 | A-17, A-18, A-19.11, A-33 의 키는 02 의 최종 키 표와 같아야 한다. 다르면 02 를 따르고 이 문서를 고친다 | 02 1절, 3.1.4절 |
| A-27.4 | 표시 방식 | 메뉴·툴팁은 액션에 묶인 키 가운데 우선순위가 가장 높은 하나를 `Ctrl-Shift-E` 형식(하이픈, 순서 Fn → Ctrl → Win → Alt → Shift → 키)으로 그린다 | L-217 ~ L-221 |
| A-27.5 | 판정 | 키 판정(A-31.3)과 툴팁 단축키 판정(A-34.2)은 이 결과로 한다 | A-31.3, A-34.2 |
| A-27.6 | 키 편집 | 키맵 편집 화면(A-43.9)이 `keymap.json` 을 고친다. 편집 결과가 메뉴·툴팁 표시에 바로 반영된다 | K-001 ~ |

### 4.9 저장·닫기·디스크 변경·알림 (A-44, A-45)

autosave 기본이 `off` 이므로 저장 확인창이 기본 동작이다. 문구는 Zed 소스(pane.rs L2011-L2066, L2334-L2375, L5075-L5076)에서 읽은 검토 자료의 값이고 이 문서에서 다시 대조하지는 않았다.

| ID | 경우 | 본문 | 버튼 (순서대로) |
| --- | --- | --- | --- |
| A-44.1 | 수정된 탭 하나를 닫는다 | `{name} contains unsaved edits. Do you want to save it?` | `Save`, `Don't Save`, `Cancel` |
| A-44.2 | 수정된 탭 여럿을 한꺼번에 닫는다 | `Do you want to save changes to the following files?` + 목록 | `Save all`, `Discard all`, `Cancel` |
| A-44.3 | 디스크에서 지워진 파일을 저장하려 한다 | Zed 의 `DELETED_MESSAGE` (본문은 확인하지 못했다) | `Save`, `Close`, `Cancel` |
| A-44.4 | 디스크 내용과 충돌한다 | Zed 의 `CONFLICT_MESSAGE` (본문은 확인하지 못했다) | `Overwrite`, `Discard Edits`, `Cancel` |
| A-44.5 | 저장에 실패했다 | `Unable to save file: {err}` | `Close Without Saving`, `Cancel` |
| A-44.6 | 디스크가 바뀌었다 | 깨끗한 버퍼는 자동으로 다시 읽는다. 수정된 버퍼는 충돌로 표시한다(탭 점 `warning` 색, I-071). 지워진 파일의 탭은 표시를 바꾼다. 세부 동작은 E-001 ~ 조사 대기 | I-071, 검토 자료 |
| A-44.7 | 앱을 끈다 | 묻지 않는다. 저장 안 한 버퍼는 복원한다 (A-25.8) | I-118 |
| A-45.1 | 알림(토스트) | 이 문서가 쓰는 "알림 한 줄"(A-16.4, A-24.4, A-24.6 등)의 위치·모양·닫기·자동 사라짐은 Zed 의 알림 규칙을 따른다. 규칙은 P-001 ~ 조사 대기. 그때까지 모양은 정하지 않는다 | 검토 자료 |

모든 확인창은 경고 수준이고, 기본 버튼과 `Esc` 의 결과는 P-001 ~ 조사가 정한다.

### 4.10 경로 정규화 (A-28)

폴더 경로는 저장·비교하기 전에 아래 규칙으로 정규화한다. 드라이브 루트와 UNC 경로는 바뀌지 않아야 한다.

| ID | 규칙 | 내용 |
| --- | --- | --- |
| A-28.1 | 구분자 | `/` 는 `\` 로 바꾼다 |
| A-28.2 | 확장 접두사 | `\\?\C:\x` 는 `C:\x` 로, `\\?\UNC\server\share\x` 는 `\\server\share\x` 로 바꾼다. 그 밖의 `\\?\` 형식(장치 경로 등)은 바꾸지 않고 등록하지 않는다 |
| A-28.3 | 끝의 `\` | 드라이브 루트(`C:\`)와 UNC 루트(`\\server\share\`)에서는 **남긴다.** 그 밖의 경로에서는 모두 지운다 |
| A-28.4 | 드라이브 문자 | 대문자로 바꾼다 (`d:\projects` → `D:\projects`) |
| A-28.5 | 비교 키 | 정규화한 문자열을 대소문자 무시(유니코드 단순 케이스 폴딩)로 비교한다. 대소문자를 구분하는 NTFS 폴더는 구분하지 않는다 (판단) |
| A-28.6 | 하지 않는 것 | 심볼릭 링크·정션·8.3 짧은 이름을 해석하지 않는다. `..`, `.` 구간은 지운다. 현재 드라이브 상대 경로(`C:foo`)와 앞이 `\` 하나인 경로는 절대 경로가 아니므로 등록하지 않는다 (판단) |

| ID | 입력 | 저장·비교 값 |
| --- | --- | --- |
| A-28.7 | `D:\projects\` | `D:\projects` |
| A-28.8 | `d:/projects` | `D:\projects` |
| A-28.9 | `C:\` | `C:\` (루트는 그대로) |
| A-28.10 | `\\?\C:\work\notes` | `C:\work\notes` |
| A-28.11 | `\\?\UNC\nas\share\docs\` | `\\nas\share\docs` |
| A-28.12 | `\\nas\share\` | `\\nas\share\` (UNC 루트는 그대로) |
| A-28.13 | `D:\Projects` 와 `d:\projects` | 같은 폴더로 본다 (A-13.2) |

### 4.11 확장 (A-46, A-48 ~ A-57)

사용자 결정(_decisions.md "확장 방식")과 Zed 확장 조사(`zed-extensions.md`, ID `X-`)를 반영한다. 값의 출처는 모두 X- 의 ID 로 적는다(소스 커밋 14b7b0e, 이 노트북에 설치된 Catppuccin·Catppuccin Icons·HTML 확장과 대조). 구분: **확정**은 조사에서 소스·설치본으로 확인한 사실, **결정**은 사용자 결정, **판단**은 우리 앱 선택, **기술 확인**은 구현 단계에서 확인할 것, **조사 중**은 후속 조사 대기다.

#### 4.11.1 범위와 원칙 (A-46)

| ID | 항목 | 내용 | 구분 | 근거 |
| --- | --- | --- | --- | --- |
| A-46.1 | 방식 | Zed 와 같은 확장 패키지: `extension.toml` 매니페스트와 폴더 규약. 앱을 다시 빌드하지 않고 설치·제거·다시 읽기를 한다 | 결정 | X-01 ~ X-16 |
| A-46.2 | 지원 종류 | **테마**(JSON), **아이콘 테마**(JSON + SVG), **언어**(Tree-sitter 문법 `grammars/*.wasm` + 설정 `config.toml` + 쿼리 `.scm`) | 결정 | X-09 ~ X-12 |
| A-46.3 | 지원하지 않는 종류 | 언어 서버(LSP), 디버거(`debug_adapters`, `debug_locators`), MCP 서버(`context_servers`), AI 슬래시 명령(`slash_commands`), 에이전트 서버, 문서 인덱서, 언어 모델 공급자, 스니펫. 매니페스트에 이 키가 있어도 **오류 없이 읽고 무시**한다 | 결정 + 판단 | X-13 ~ X-15 |
| A-46.4 | 화면을 더하는 확장 | 없다. 패널·탭 뷰어는 확장으로 더하지 못한다 (3.12절) | 결정 | _decisions.md |
| A-46.5 | WASM 이 필요한 곳 | ① **문법 `grammars/<이름>.wasm` 은 필요하다.** Tree-sitter 파서를 WASM 으로 컴파일한 것이고 Tree-sitter 의 WASM 로더가 읽는다. ② **`extension.wasm` 은 읽지도 실행하지도 않는다.** Zed 도 `lib.kind` 가 없으면 로드하지 않고, `extension.wasm` 의 export 는 LSP·슬래시 명령·MCP·문서 인덱싱·DAP 용뿐이라 테마·아이콘 테마·언어에는 쓰이지 않는다. ③ 따라서 `wasm32-wasip2` 실행 환경(`zed_extension_api` 호스트)은 **만들지 않는다** | 확정(X-28, X-29) + 판단(만들지 않음) | [extension.wit](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_api/wit/since_v0.8.0/extension.wit#L94-L178), [extension_host.rs L1660-L1737](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/extension_host/src/extension_host.rs#L1660-L1737) |
| A-46.6 | 기존 Zed 패키지와의 호환 | 설치본 `html` 확장처럼 문법·언어와 LSP 용 `extension.wasm` 을 함께 담은 패키지를 **그대로 설치할 수 있어야 한다.** `[lib]`, `[language_servers.*]` 는 파싱만 하고 무시하며 `extension.wasm` 은 건드리지 않는다 | 확정(로컬 html 설치본) + 판단 | X-21, X-29 |
| A-46.7 | 구현 가능성 | 문법 WASM 을 읽는 Tree-sitter WASM 로더를 gpui-kit 판(Rust)과 Tauri 판(웹뷰)이 각각 쓸 수 있는지는 **02-gpui·03-tauri 출발 문서의 검토 항목으로 넘긴다** (A-57.7) | 기술 확인 | X-40 |
| A-46.8 | 내장하는 것 | **One Dark·One Light·Zed (Default) 아이콘 테마와 Catppuccin 테마 4종·Catppuccin Icons 를 모두 본체에 내장한다**(Catppuccin 은 MIT 고지를 함께 배포, 03 의 7절). 내장도 확장과 같은 파일 형식·같은 파서를 쓴다. 설치된 확장의 같은 이름이 나중에 등록되면 그쪽이 이긴다 (A-51.6) | 결정 | _decisions.md "확장 세부", X-57, X-61, 03 T-32.14 |

#### 4.11.2 extension.toml 키 (A-48)

| ID | 키 | 타입 | 필수 | 우리 앱 | 규칙 | 근거 |
| --- | --- | --- | --- | --- | --- | --- |
| A-48.1 | `id` | 문자열 | 예 | 채택 | 소문자 영숫자와 하이픈. 앞뒤 하이픈 금지. 설치 폴더 이름(`installed/<id>/`)이 된다. 레지스트리에 올릴 때는 `zed-` 로 시작하거나 `-zed` 로 끝나면 안 된다 | X-01 |
| A-48.2 | `name` | 문자열 | 예 | 채택 | 사람이 읽는 이름 | X-02 |
| A-48.3 | `version` | 문자열 | 예 | 채택 | SemVer. 업데이트 비교에 쓴다 | X-03 |
| A-48.4 | `schema_version` | 정수 | 예 | 채택 | **1 만 받는다.** 더 큰 값이면 그 확장을 건너뛰고 알림(A-45.1). 구형 `extension.json` 은 읽지 않는다 (판단) | X-04 |
| A-48.5 | `description`, `repository`, `authors` | 문자열, 문자열, 문자열 배열 | 아니오 | 채택 | 확장 목록 화면이 있을 때 표시한다 | X-05 ~ X-07 |
| A-48.6 | `[lib]` (`kind`, `version`) | 표 | 아니오 | **읽고 무시** | `kind` 가 비어 있어도 있어도 `extension.wasm` 을 로드하지 않는다 (A-46.5) | X-08 |
| A-48.7 | `themes` | 상대 경로 배열 | 아니오 | **채택** | 예: `["themes/catppuccin-mauve.json"]`. 비어 있고 `themes/*.json` 이 있으면 개발용 설치 때 채운다 (A-53.4) | X-09 |
| A-48.8 | `icon_themes` | 상대 경로 배열 | 아니오 | **채택** | 예: `["icon_themes/catppuccin-icons.json"]` | X-10 |
| A-48.9 | `languages` | 상대 폴더 배열 | 아니오 | **채택** | 예: `["languages/html"]`. 폴더마다 `config.toml` 이 있어야 한다 | X-11 |
| A-48.10 | `[grammars.<id>]` | 표 | 아니오 | **채택** | `repository`(Git 주소), `rev`(커밋 SHA, `commit` 별칭), `path`(저장소 안 하위 폴더, 선택). 배포된 패키지에는 컴파일된 `grammars/<id>.wasm` 이 들어 있다. 이 표의 `repository`·`rev` 는 설치할 때 쓰지 않고 개발용 설치(A-53)에서만 쓴다 | X-12 |
| A-48.11 | `capabilities` | 배열 | 아니오 | **읽되 모두 거부** | A-55.1 | X-16 |
| A-48.12 | `[language_servers.*]`, `[context_servers]`, `[slash_commands]`, `[agent_servers.*]`, `[indexed_docs_providers.*]`, `language_model_providers`, `snippets`, `[debug_adapters.*]`, `[debug_locators.*]` | 표·배열 | 아니오 | **읽고 무시** | 알 수 없는 키 오류로 설치가 막히지 않게 한다 | X-13 ~ X-15 |

설치된 `catppuccin` 확장의 `extension.toml` 은 위 키 가운데 `id`, `name`, `version`, `schema_version`, `description`, `repository`, `authors`, `themes`, `icon_themes`, `languages`, `capabilities` 와 빈 `[lib]`·`[grammars]`·`[language_servers]`·`[context_servers]`·`[slash_commands]` 표를 가진다 (로컬 설치본, X-17). `html` 확장은 `[lib] kind = "Rust"` 와 `[grammars.html]`, `[language_servers.vscode-html-language-server]` 를 가진다 (X-21).

#### 4.11.3 폴더 구조 (A-49)

경로는 모두 확장 루트(`installed/<id>/`) 기준 상대 경로다. 절대 경로, `..` 이 들어간 경로, 확장 폴더 밖을 가리키는 심볼릭 링크는 거부한다 (A-55.2).

| ID | 종류 | 구성 | 로컬 설치본 | 근거 |
| --- | --- | --- | --- | --- |
| A-49.1 | 테마 확장 | `extension.toml` + `themes/<이름>.json` (`extension.wasm` 없음) | `catppuccin`: `themes/catppuccin-mauve.json`, `catppuccin-no-italics-mauve.json`. WASM 파일 0개 | X-17, X-18 |
| A-49.2 | 아이콘 테마 확장 | `extension.toml` + `icon_themes/<이름>.json` + `icons/<변종>/*.svg` (`extension.wasm` 없음). JSON 안의 아이콘 경로(`./icons/...`)는 확장 루트 기준이다 | `catppuccin-icons`: 변종 8개 폴더. WASM 파일 0개 | X-19, X-20 |
| A-49.3 | 언어 확장 | `extension.toml` + `grammars/<문법>.wasm` + `languages/<언어>/config.toml` + `languages/<언어>/*.scm`. `extension.wasm` 은 LSP 를 함께 줄 때만 들어 있고 우리는 쓰지 않는다 | `html`: `grammars/html.wasm`, `languages/html/{config.toml, highlights.scm, brackets.scm, indents.scm, injections.scm, outline.scm, overrides.scm}`, `extension.wasm`(309,560 바이트, 무시) | X-21 |
| A-49.4 | 테마 JSON | 03 의 T-01 ~ T-06. `$schema` 는 `https://zed.dev/schema/themes/v0.2.0.json`. 한 파일에 테마 여럿 | 03 | X-18 |
| A-49.5 | 아이콘 테마 JSON | 03 의 4.1절(T-11, T-12). `$schema` 는 `icon_themes/v0.2.0.json` 또는 `v0.3.0.json` | 03 | X-20 |
| A-49.6 | 언어 `config.toml` | 필수 `name`. 문법 연결 `grammar`(`[grammars.<id>]` 의 키), 매칭 `path_suffixes`(점 없는 확장자), `first_line_pattern`(첫 줄 정규식), `modeline_aliases`. 편집 규칙 `brackets`, `line_comments`, `block_comment`, `documentation_comment`, `autoclose_before`, `word_characters`, `increase_indent_pattern`, `decrease_indent_pattern(s)`, `auto_indent_on_paste`, `hard_tabs`, `tab_size`, `soft_wrap`, `jsx_tag_auto_close`, `overrides`, `code_fence_block_name`, `hidden`. **전부 읽고, 쓰이지 않는 키는 무시한다.** 어떤 편집 규칙을 실제로 쓰는지는 편집기 조사(E-001 ~)가 정한다 | 확정(키 목록) + 판단 | X-22 |
| A-49.7 | **쓰는** 쿼리 파일 | `highlights.scm`(필수, 구문 강조), `brackets.scm`(괄호 짝), `indents.scm`(자동 들여쓰기, `auto_indent: syntax_aware` 가 쓴다), `injections.scm`(다른 언어 끼워 넣기, 예: 마크다운 코드 블록), `overrides.scm`(범위별 설정) | 판단 | X-23 |
| A-49.8 | **쓰지 않는** 쿼리 파일 | `outline.scm`(아웃라인 화면이 없다), `redactions.scm`(화면 공유), `runnables.scm`(실행), `debugger.scm`(디버거), `textobjects.scm`(Vim), `tasks.json`, `semantic_token_rules.json`(LSP). 있어도 읽지 않는다 | 판단 | X-23, X-24 |
| A-49.9 | 구문 강조 이름 매칭 | `highlights.scm` 의 캡처 이름(`@keyword.modifier` 등)은 03 의 T-27.1 규칙(가장 긴 점 접두사)으로 테마의 `syntax` 키를 찾는다 | 확정 | X-23, T-27.1 |
| A-49.10 | 파일 이름 → 언어 | 규칙은 4.11.13절(A-59)이 정한다. 요약: 뒤에 등록된 언어부터 훑고, 일치한 후보 문자열이 **더 긴** 언어가 이긴다. 같은 이름의 언어는 본체 내장이 이기고 확장 언어는 거부된다 (A-51.5) | 확정 | X-100 ~ X-104 |

#### 4.11.4 설치 위치와 색인 (A-50)

| ID | 항목 | 내용 | 구분 | 근거 |
| --- | --- | --- | --- | --- |
| A-50.1 | 확장 루트 | `%LOCALAPPDATA%\ZedLike\extensions\`. Zed 는 `%LOCALAPPDATA%\Zed\extensions\`(데이터 폴더)이고 사용자 테마 폴더는 `%APPDATA%\Zed\themes`(설정 폴더)다. 우리 앱은 앱 이름만 바꾼다 | 판단 | X-30 |
| A-50.2 | `installed/<id>/` | 확장 하나의 패키지가 풀려 있는 폴더. 폴더 이름이 `id`. 이 폴더가 있으면 그 확장은 **켜져 있는 것**이다 (A-52.1). 개발용 확장은 원본 소스 폴더를 가리키는 링크다 (A-53.5) | 확정 | X-31 |
| A-50.3 | `staging/` | 내려받은 압축 파일을 임시로 푸는 폴더. 다 풀린 뒤 `installed/<id>` 로 이름을 바꿔 옮긴다(교체 전에 기존 폴더를 지운다). 설치 중에 끊겨도 `installed/` 는 온전하다 | 확정 | X-32, X-43 |
| A-50.4 | `work/<id>/` | Zed 는 WASM 확장이 쓰는 작업 폴더다. `extension.wasm` 을 실행하지 않는 우리 앱은 **만들지 않는다** | 확정 + 판단 | X-33 |
| A-50.5 | `build/` | Zed 는 개발용 빌드의 문법 컴파일에 필요한 `wasi-sdk` 를 받아 두는 폴더다. 개발용 문법 빌드를 하지 않으면 만들지 않는다 (A-53.3) | 확정 + 판단 | X-34 |
| A-50.6 | `index.json` | `extensions/index.json`. 최상위 키 4개: `extensions`(id → `manifest` + `dev`), `themes`(테마 이름 → `extension`, `path`), `icon_themes`(이름 → `extension`, `path`), `languages`(언어 이름 → `extension`, `path`, `matcher`, `hidden`, `grammar`, `query_files`). `query_files` 는 어떤 쿼리 파일이 있는지를 비트로 담은 정수다: highlights 1, brackets 2, outline 4, indents 8, injections 16, overrides 32, redactions 64, runnables 128, debugger 256, textobjects 512. HTML 설치본은 63 (앞 여섯 개) | 확정 | X-23, X-35, 로컬 `index.json` |
| A-50.7 | 색인의 성격 | **캐시다.** 지우거나 깨져도 `installed/` 를 훑어 다시 만든다. 앱이 시작할 때 색인이 없거나 JSON 이 깨졌거나 `installed/` 폴더의 수정 시각이 색인보다 새로우면 다시 만든다 | 확정 | X-36 |
| A-50.8 | 폴더 감시 | `installed/` 를 감시한다(감시 지연 100ms). 바뀌면 200ms 안에 또 변경이 없을 때 다시 읽는다. 그래서 폴더를 직접 넣거나 지워도 앱이 켜진 채로 반영된다 | 확정 | X-39 |

`index.json` 예시다. 이 노트북의 Zed `index.json` 에서 `catppuccin` 과 `html` 만 옮기고 값을 줄였다 (아이콘 테마 항목은 같은 모양으로 `extension`, `path` 만 있다).

```json
{
  "extensions": {
    "catppuccin": {
      "manifest": {
        "id": "catppuccin", "name": "Catppuccin", "version": "0.2.27", "schema_version": 1,
        "description": "Soothing pastel theme for Zed",
        "repository": "https://github.com/catppuccin/zed",
        "authors": ["Catppuccin <releases@catppuccin.com>"],
        "lib": { "kind": null, "version": null },
        "themes": ["themes/catppuccin-no-italics-mauve.json", "themes/catppuccin-mauve.json"],
        "icon_themes": [], "languages": [], "grammars": {}, "capabilities": []
      },
      "dev": false
    },
    "html": {
      "manifest": {
        "id": "html", "name": "HTML", "version": "0.3.2", "schema_version": 1,
        "description": "HTML support.",
        "lib": { "kind": "Rust", "version": "0.7.0" },
        "themes": [], "icon_themes": [], "languages": ["languages/html"],
        "grammars": { "html": { "repository": "https://github.com/tree-sitter/tree-sitter-html", "rev": "bfa075d83c6b97cd48440b3829ab8d24a2319809", "path": null } },
        "language_servers": { "vscode-html-language-server": { "language": "HTML", "languages": [], "language_ids": { "CSS": "css", "HTML": "html" }, "code_action_kinds": null } },
        "capabilities": []
      },
      "dev": false
    }
  },
  "themes": {
    "Catppuccin Latte": { "extension": "catppuccin", "path": "themes/catppuccin-mauve.json" },
    "Catppuccin Mocha": { "extension": "catppuccin", "path": "themes/catppuccin-mauve.json" }
  },
  "icon_themes": {
    "Catppuccin Latte": { "extension": "catppuccin-icons", "path": "icon_themes/catppuccin-icons.json" }
  },
  "languages": {
    "HTML": {
      "extension": "html", "path": "languages/html",
      "matcher": { "path_suffixes": ["html", "htm", "shtml"], "first_line_pattern": null, "modeline_aliases": [] },
      "hidden": false, "grammar": "html", "query_files": 63
    }
  }
}
```

우리 앱의 `index.json` 도 이 모양을 따른다. 우리가 쓰지 않는 `language_servers` 같은 키는 설치된 매니페스트를 그대로 옮겨 적어도 되고 빼도 된다 (읽을 때 무시, A-48.12).

#### 4.11.5 시작할 때 읽는 순서 (A-51)

| ID | 단계 | 동작 | 근거 |
| --- | --- | --- | --- |
| A-51.1 | 1. 내장 등록 | One Dark·One Light(03 T-07), Zed (Default) 아이콘 테마(03 T-28), 본체 내장 언어를 먼저 등록한다. Catppuccin 을 내장으로 두면(A-46.8) 이때 함께 등록한다 | X-57 |
| A-51.2 | 2. 색인 읽기 | `index.json` 을 읽고 `installed/` 수정 시각과 비교한다(A-50.7). 유효하면 바로 등록해 **첫 화면을 지연시키지 않는다**. 아니면 다시 만든다 | X-36 |
| A-51.3 | 3. 색인 다시 만들기 | `installed/` 의 항목마다 `extension.toml` 을 읽고(링크이면 `dev: true`), `languages/*/config.toml` 과 쿼리 파일, `themes/*.json` 안의 테마 이름, `icon_themes/*.json` 안의 아이콘 테마 이름을 찾아 `index.json` 을 통째로 덮어쓴다 | X-37 |
| A-51.4 | 4. 차이 적용 | 이전 색인과 새 색인을 비교해 ① 없어졌거나 바뀐 확장의 테마·아이콘 테마·언어·문법을 먼저 내리고 ② 새로 생긴 것을 등록한다. 문법은 `grammars/<이름>.wasm` 경로만 등록하고 **그 언어가 실제로 필요할 때** 파일을 읽어 로드한다. 언어는 `config.toml` 과 쿼리를 처음 쓸 때 읽는다. 테마·아이콘 테마 JSON 은 백그라운드에서 읽는다 | X-38, X-40 |
| A-51.5 | 언어 이름 충돌 | 본체 내장 언어와 이름이 같은 확장 언어는 **등록이 거부된다.** 확장끼리의 충돌 규칙은 A-59.8 | 확정. X-38, X-100 |
| A-51.6 | 테마·아이콘 테마 이름 충돌 | 내장과 확장을 가리지 않고 **이름을 키로 하나의 목록**에 넣는다. 같은 이름이 있으면 나중에 넣은 것이 이전 것을 덮어쓴다(Zed 가 맵 삽입으로 처리하는 것으로 보이며 덮어쓰기 규칙을 따로 확인하지는 않았다, 추정). 확장은 내장 뒤에 읽으므로 확장이 이긴다 | X-57 |
| A-51.7 | 5. 현재 테마 다시 적용 | 확장 읽기가 끝나면 설정에 적힌 테마·아이콘 테마를 다시 찾아 적용하고 창을 새로 그린다 | X-38, X-59 |
| A-51.8 | 읽기 전의 임시 대체 | 설정이 가리키는 테마·아이콘 테마가 확장에 있고 아직 안 읽혔으면 **OS 밝기에 맞는 내장 기본값**(라이트면 One Light, 다크면 One Dark, 아이콘 테마는 Zed (Default))으로 잠깐 보였다가 읽기가 끝나면 설정 값으로 바뀐다. 규칙과 시스템 밝기 변경은 4.11.14절(A-60)이 정한다. Catppuccin 은 내장이라 이 깜빡임이 없다 | 확정. X-59, X-106 |

#### 4.11.6 설치·제거·업데이트·켜기 (A-52)

| ID | 동작 | 규칙 | 구분 | 근거 |
| --- | --- | --- | --- | --- |
| A-52.1 | 켜기·끄기 | **개별 켜기·끄기가 없다.** `installed/<id>/` 가 있으면 켜진 것이고, 끄려면 제거한다. 이전 판의 `enabled` 필드는 두지 않는다 | 확정(Zed) + 판단 | X-41 |
| A-52.2 | 제거 | `installed/<id>` 를 통째로 지운다 → 색인을 다시 만들고 메모리의 테마·아이콘 테마·언어·문법을 내린다 → (Zed 는 `work/<id>` 도 지운다) → 완료 알림. Windows 에서 프로세스가 아직 파일을 잡고 있을 수 있어 Zed 는 0ms, 100ms, 200ms 간격으로 최대 3번 시도한다 | 확정 | X-41 |
| A-52.3 | 제거한 확장의 테마가 지금 선택돼 있다 | 설정 값은 그대로 두고 화면은 내장 기본값으로 돌아간다(A-51.8 의 "찾을 수 없음" 과 같은 처리). 설정을 자동으로 고치지는 않는다 | 판단 | X-59 |
| A-52.4 | 설치 (로컬) | **폴더**: 선택한 폴더의 `extension.toml` 을 읽어 검증하고(A-48, A-55.2) `staging/` 에 복사한 뒤 `installed/<id>` 로 옮긴다. **보관 파일(`.tar.gz`)**: `staging/` 에 풀고 같은 검증 후 옮긴다. 기존 같은 `id` 가 있으면 지우고 교체한다. 끝나면 색인을 다시 읽는다. 개발용은 A-53 | 결정(로컬만) + 판단(폴더 복사) | X-43, X-32 |
| A-52.5 | 같은 id 를 다시 설치 | 새 버전으로 교체한다. 개발용 확장이 이미 같은 id 로 설치돼 있으면 일반 설치는 오류다 | 확정 | X-54, X-43 |
| A-52.6 | 업데이트 | **온라인 확인이 없다.** 새 버전은 같은 `id` 의 더 새 폴더·파일을 다시 설치해서 교체한다. 더 낮은 버전도 막지 않는다 (판단) | 결정(로컬만) + 판단 | _decisions.md |
| A-52.7 | 자동 설치·자동 업데이트 | **없다.** `auto_install_extensions`, `auto_update_extensions` 설정도 없다 (A-21.21). Zed 가 본체에 흡수한 확장 목록(`SUPPRESSED_EXTENSIONS`)을 지우는 동작도 하지 않는다 | 결정 | X-42, _decisions.md |
| A-52.8 | 설치 직후 | 테마 확장이면 그 확장의 테마만 걸러 놓은 테마 선택기를, 아이콘 테마 확장이면 같은 방식의 아이콘 테마 선택기를 바로 연다. 확장 화면이 있으므로 적용한다 (03 의 T-32.7) | 확정(Zed) + 결정 | X-51 |
| A-52.9 | 오류 | 매니페스트가 깨졌거나 `schema_version` 이 1 보다 크거나 경로 검증에 실패한 확장은 그 확장만 건너뛰고 알림(A-45.1)을 보인다. 나머지는 정상으로 읽는다 | 판단 | — |

#### 4.11.7 개발용 확장 설치 (A-53)

Zed 의 `Install Dev Extension` 흐름이다 (X-53 ~ X-56). 개발용 설치는 **포함**이다(결정). 빌드와 링크 방식의 기술 확인은 02-gpui·03-tauri 출발 문서의 검토 항목으로 넘긴다.

| ID | 단계 | Zed 의 동작 | 우리 앱 | 구분 |
| --- | --- | --- | --- | --- |
| A-53.1 | 진입 | 확장 화면의 `Install Dev Extension` 버튼 또는 명령 팔레트(A-43.4)의 같은 이름 명령. 폴더 선택 창(폴더 하나)을 연다. `Rebuild Dev Extension` 은 개발용 확장이 하나도 없으면 팔레트에서 숨긴다 | 같다. 확장 화면은 A-58 | 확정 |
| A-53.2 | 매니페스트 읽기 | 선택한 폴더의 `extension.toml` 을 읽는다. 같은 id 의 일반 확장이 설치돼 있으면 먼저 제거한다. 이미 개발용으로 설치돼 있으면 오류 | 같다 | 확정 |
| A-53.3 | 빌드 | 순수 테마·아이콘 테마는 **빌드를 건너뛴다.** `lib.kind = Rust` 이면 `wasm32-wasip2` 로 `cargo build` 해서 `extension.wasm` 을 만든다. `[grammars]` 가 있으면 문법 저장소를 `grammars/<이름>/` 에 받아 `wasi-sdk`(v34.0, 자동 다운로드)의 `clang` 으로 `grammars/<이름>.wasm` 을 만든다 | **Rust 빌드는 하지 않는다**(`extension.wasm` 을 안 쓴다). 문법 컴파일은 외부 도구·내려받기가 필요해서 **포함 범위와 방법을 기술 확인으로 넘긴다** (A-57.3). 넣지 않으면 `grammars/*.wasm` 이 이미 들어 있는 폴더만 개발용으로 설치된다 | 기술 확인 |
| A-53.4 | 기본값 채우기 | 매니페스트에 목록이 없으면 `themes/*.json`, `icon_themes/*.json`, `languages/<폴더>/config.toml` 을 훑어 채운다 | 같다 | 확정 |
| A-53.5 | 설치 | `installed/<id>` 를 원본 소스 폴더를 가리키는 **심볼릭 링크**로 만들고 다시 읽는다. 색인에는 `dev: true` | 같은 효과를 내는 디렉터리 정션 사용을 제안한다. Windows 권한 문제는 **기술 확인으로 넘긴다** (A-57.3) | 기술 확인 |
| A-53.6 | 라이브 반영 | 링크된 소스 폴더의 테마·아이콘 테마 JSON 과 `.scm` 을 고치면 폴더 감시(A-50.8)로 다시 읽는다. 문법·`extension.wasm` 이 바뀌면 `Rebuild` 로 다시 빌드해 다시 읽는다 | 문법 빌드(A-53.3)를 넣을 때만 `Rebuild` 를 둔다 | 기술 확인 |
| A-53.7 | 표시 | 확장 목록(A-58)에서 개발용은 버전 뒤에 `(dev)` 가 붙고 `Rebuild`(해당할 때)와 `Uninstall` 단추가 Accent 색이다 | 같다 | 확정 |

#### 4.11.8 데이터 모델 (A-54)

| ID | 저장 위치·구조 | 내용 | 비고 |
| --- | --- | --- | --- |
| A-54.1 | 설치된 확장 목록 | `extensions/index.json` 의 `extensions` (A-50.6). **`state.json` 에는 넣지 않는다.** 창과 폴더에 따라 달라지지 않는 전역 값이다 | 이전 판의 `extensions.json` 은 없앴다 |
| A-54.2 | 활성 상태 | 따로 저장하지 않는다. `installed/<id>/` 가 있으면 활성이다 (A-52.1). **끄기 기능은 없다** (결정, Zed 와 같음) | 확정 |
| A-54.3 | 개발용 여부 | `index.json` 의 `extensions.<id>.dev` | X-37 |
| A-54.4 | 테마·아이콘 테마·언어의 출처 | `index.json` 의 `themes`, `icon_themes`, `languages` 가 각 이름이 어느 확장의 어느 파일에서 오는지 가진다 (A-50.6) | 선택기와 구문 강조가 이 색인으로 찾는다 |
| A-54.5 | 선택한 테마·아이콘 테마 | `settings.json` 의 `theme`, `icon_theme` (A-21.1, A-21.2). 확장 테마 이름도 그대로 적는다 | 03 의 T-21 |
| A-54.6 | 메모리 모델 | `ExtensionStore`: `extensions: map<id, {manifest, dev}>`, `themes: map<이름, {extension, path}>`, `icon_themes: map<이름, {extension, path}>`, `languages: map<이름, {extension, path, matcher, hidden, grammar, query_files}>`. 테마·아이콘 테마 등록부(03 의 T-32)는 내장과 확장을 같은 맵에 둔다 | X-57 |
| A-54.7 | 쓰기 | `index.json` 도 A-24.1 의 쓰기 순서(임시 파일 → 교체)로 쓴다. 깨진 색인은 보관하지 않고 다시 만든다 | 판단 |
| A-54.8 | 설정 키 | `auto_install_extensions`, `auto_update_extensions` 는 만들지 않는다 (A-21.21, A-52.7) | A-57.1 |

#### 4.11.9 보안·권한·라이선스 (A-55)

| ID | 항목 | 규칙 | 구분 | 근거 |
| --- | --- | --- | --- | --- |
| A-55.1 | 권한(`capabilities`) | Zed 의 권한은 `process:exec`(프로세스 실행), `download_file`(파일 내려받기), `npm:install` 셋이다. 우리는 매니페스트를 읽되 **전부 허용하지 않는다.** 지원하는 세 종류는 권한 없이 동작한다 | 확정(Zed 모델) + 판단 | X-16, X-65, X-66 |
| A-55.2 | 경로 검증 | 매니페스트·JSON·TOML 이 가리키는 모든 파일(`themes`, `icon_themes`, `languages`, 아이콘 `path`, 문법 `wasm`, `.scm`)은 **확장 폴더 안의 상대 경로**여야 한다. 절대 경로·`..` 은 거부한다. 개발용 링크의 원본 폴더는 그 확장의 루트로 본다 | 확정(상대 경로 규칙) + 판단 | X-20, X-66 |
| A-55.3 | WASM 샌드박스 | Zed 는 `extension.wasm` 에게 `work/<id>` 하나만 열어 주고 그 밖으로 나가는 경로를 막는다. 우리는 `extension.wasm` 을 실행하지 않아 이 장치가 필요 없다. **문법 WASM** 은 Tree-sitter 로더 안에서만 실행하고 파일·네트워크를 열어 주지 않는다 | 판단 | X-64 |
| A-55.4 | 확장 라이선스 | 공식 레지스트리는 2025-10-01 부터 확장 루트에 `LICENSE*` 가 있어야 하고 허용 라이선스 9종을 정했다(Apache-2.0, BSD-2, BSD-3, CC BY 4.0, GPLv3, LGPLv3, MIT, Unlicense, zlib). 이것은 **그 레지스트리에 올릴 때의 규칙**이고 로컬 설치에는 해당하지 않는다. 우리 앱이 내장·번들하는 확장의 고지는 03 의 7절 | 확정 | X-62 |
| A-55.5 | 클린룸 | Zed 의 확장 호스트·확장 화면 코드(GPL-3.0)는 복사하지 않는다. 이 문서의 명세와 스키마만 쓴다. `zed_extension_api` 는 Apache-2.0 이지만 우리는 쓰지 않는다 | 확정 | X-63 |

#### 4.11.10 선택기와의 연동 (A-56)

확장에서 온 테마·아이콘 테마가 내장과 함께 선택기에 나타나는 규칙은 03 의 6.4절(T-32)이 정한다. 요약: 내장과 확장을 구분 없이 하나의 목록으로, 어두운 것 먼저 이름 순으로 보이고, 미리 적용·취소·저장은 내장과 같다. 확장 아이콘 테마는 Zed (Default) 의 매핑 위에 덮어써서 비어 있는 곳이 생기지 않는다. 선택기 푸터의 `Install Themes`·`Install Icon Themes` 는 확장 화면을 연다.

#### 4.11.11 확장 화면 (A-58)

사용자 결정(확장 세부): **Zed 와 같은 화면**이다. 단축키 `Ctrl+Shift+X`, 필터는 All·Themes·Icon Themes·Languages. 온라인 검색·목록 대신 **설치된 확장 목록**, 제거, 로컬 설치(폴더·파일), 개발용 설치를 둔다. 치수·문구는 Zed 값(X-44 ~ X-50)이고 온라인에만 있던 부분은 뺐다. 이 화면에 마우스를 올렸을 때의 툴팁·우클릭 메뉴는 아직 조사하지 않았다 (A-57.9).

| ID | 영역 | 규칙 | 구분 | 근거 |
| --- | --- | --- | --- | --- |
| A-58.1 | 진입 | 액션 `Extensions`. 키 `Ctrl+Shift+X`(Zed 기본, VSCode 덧씌움 뒤의 표시 키는 02 의 1절을 따른다). ☰ 앱 이름 메뉴의 `Extensions`(I-339)와 ∨ 메뉴의 `Extensions`(I-624)도 같은 액션이다. 선택기 푸터의 `Install Themes` 는 같은 화면을 테마 필터로 연다 | 확정 + 결정 | X-44, X-60 |
| A-58.2 | 탭 | 활성 Pane 에 이미 이 화면 탭이 있으면 활성화하고 없으면 새 탭을 더한다. 제목 `Extensions`, 탭 도구 줄 없음, 바탕 `editor.background`. 탭 종류는 `extensions_page` (A-22.12) | 확정 | X-44 |
| A-58.3 | 머리 1행 | 세로 배치, 위 16, 좌우 16, 행 사이 16. 왼쪽 `Extensions` 큰 제목. 오른쪽 단추 둘: `Install Extension`(Outlined, Medium)과 `Install Dev Extension`(Outlined, Medium). `Install Extension` 은 누르면 작은 메뉴 `Folder…`, `Archive (.tar.gz)…` 를 열고 OS 선택 창으로 연결한다. Zed 에는 이 단추가 없다 | 확정(제목·Dev 단추) + 결정(로컬 설치 단추, 문구는 판단) | X-45 |
| A-58.4 | 머리 2행: 검색창 | 높이 32, 최소 폭 384, 남는 폭을 채움, 왼쪽 안쪽 6, 오른쪽 8, 안 간격 8, 테두리 1(`border`), 모서리 6. 왼쪽 돋보기 아이콘(Muted), 한 줄 입력, 안내 `Search extensions...`, 글자 14, 줄 높이 1.3. 설치된 확장을 입력 즉시 퍼지 필터. `id:<확장 id>` 로 쓰면 그 id 만. **온라인 조회·250ms 지연·로딩 회전 아이콘은 없다** | 확정(치수·문구) + 결정(온라인 없음) | X-46 |
| A-58.5 | 머리 2행: 설치 상태 토글 | Zed 의 `All`·`Installed`·`Not Installed` 토글은 **두지 않는다**(보이는 것이 설치된 확장뿐이다) | 판단 | X-47 |
| A-58.6 | 머리 3행: 필터 줄 | 간격 8, 위아래 10, 좌우 16, 아래 테두리 1(`border_variant`), 가로 스크롤. 단추 `All`, `Themes`, `Icon Themes`, `Languages`. 선택은 Filled, 나머지는 Subtle | 확정 + 결정(네 개) | X-48 |
| A-58.7 | 목록 | 가상 목록, 좌우 16, 아래 16, 세로 스크롤바. 순서는 이름 순 (판단). 빈 상태: 위 12, 좌우 16, 문구 `No extensions` (검색어가 있으면 `No extensions that match "<검색어>"`). 색인을 읽는 동안은 `Loading extensions...` | 확정(치수) + 판단(순서·문구 조합) | X-49 |
| A-58.8 | 카드 바깥 | 위 여백 16, 폭 100%, **높이 110 고정**, 안쪽 12, 안 간격 8, 양끝 정렬. 바탕 `elevated_surface.background` 50% 불투명, 테두리 1(`border_variant`), 모서리 6 | 확정 | X-49 |
| A-58.9 | 카드 1행 | 왼쪽: 이름(작은 제목) + `v{version}`(개발용은 `v{version} (dev)`, 가장 작은 글자, Muted) + 기능 배지(`Themes`, `Icon Themes`, `Languages`, 간격 4). 오른쪽: `Uninstall`(OutlinedGhost). 개발용은 `Rebuild`(해당할 때)와 `Uninstall` 이 Accent 색 | 확정 + 결정 | X-50 |
| A-58.10 | 카드 2행 | 왼쪽에 `description`(작은 글자, 한 줄 말줄임). 오른쪽의 다운로드 수는 **없다** | 확정 + 결정 | X-50 |
| A-58.11 | 카드 3행 | 왼쪽: 사람 아이콘(가장 작게, Muted) + 작성자(작은 글자, Muted, 말줄임. 여럿이면 `Authors: ` 뒤에 쉼표로). 오른쪽: 저장소 아이콘 단추(`repository` 가 있을 때. 툴팁 `Visit Extension Repository` 와 주소, 누르면 외부에서 연다). **Zed 의 ⋯ 메뉴(`Install Another Version...`, `Copy Extension ID`, `Copy Author Info`)는 온라인 카드에만 있었으므로 두지 않는다** | 확정 + 결정 | X-50 |
| A-58.12 | 없는 카드 요소 | `Install`, `Upgrade` 단추, `Overridden by dev extension.` 덮개, 호환 안 됨 툴팁은 온라인 카드 전용이라 없다 | 결정 | X-50 |
| A-58.13 | 제거 | `Uninstall` 을 누르면 A-52.2 대로 제거하고 카드가 사라진다. 확인 창은 두지 않는다 (판단, Zed 도 묻지 않는 것으로 보이나 확인하지 못했다) | 판단 | X-41 |
| A-58.14 | 로컬 설치 흐름 | 메뉴에서 폴더나 파일을 고르면 A-52.4 로 설치한다. 설치하는 동안 단추를 비활성으로 하고, 끝나면 목록에 카드가 나타난다. 실패하면 알림(A-45.1)에 이유를 보인다 | 판단 | A-52.4 |
| A-58.15 | 상태 복원 | 복원하지 않는다 (A-22.12) | 판단 | — |

#### 4.11.12 결정과 남은 확인 사항 (A-57)

사용자 결정(_decisions.md "확장 세부")으로 A-57.1 ~ A-57.5 를 확정했다. 남은 것은 기술 확인과 후속 조사다.

| ID | 내용 | 결과 | 구분 |
| --- | --- | --- | --- |
| A-57.1 | 확장 설치 출처 | **로컬 설치만.** 확장 폴더·보관 파일을 골라 설치한다. 온라인 목록·`api.zed.dev`·미러는 쓰지 않는다. 자동 설치·자동 업데이트도 없다 (A-52.6, A-52.7) | 결정 |
| A-57.2 | 확장 관리 화면 | **Zed 와 같은 화면**을 둔다 (A-58). 필터 네 개, 설치된 목록, 제거, 로컬 설치, 개발용 설치 | 결정 |
| A-57.3 | 개발용 문법 빌드(A-53.3)와 링크 방식(A-53.5) | 외부 도구(`wasi-sdk`)·내려받기 필요 여부, 심볼릭 링크와 정션 중 무엇을 쓸지 | **02-gpui·03-tauri 출발 문서의 검토 항목으로 넘김** (기술 확인) |
| A-57.4 | Catppuccin 의 내장 여부 | **본체에 내장** (MIT 고지). One Dark·One Light·Zed (Default) 도 내장 (A-46.8) | 결정 |
| A-57.5 | 확장 끄기 | **없다.** 설치되어 있으면 켜진 것이다 (A-52.1, A-54.2) | 결정 |
| A-57.6 | 같은 파일 이름에 맞는 언어가 여럿일 때의 우선 규칙, 같은 이름 확장 언어끼리의 충돌 | **확정.** 4.11.13절(A-59). 우리 앱의 내장 언어 목록·순서는 아직 정하지 않았으므로(A-40.17) 그 부분만 남는다 | 확정 (조사 X-100 ~ X-104) |
| A-57.7 | Tree-sitter WASM 로더의 구현 (A-46.7) | gpui 판은 Rust 크레이트, Tauri 판은 웹 쪽 로더가 같은 `grammars/*.wasm` 을 읽을 수 있는지 | **02-gpui·03-tauri 출발 문서의 검토 항목으로 넘김** (기술 확인) |
| A-57.8 | 확장을 읽는 중에 선택기를 열었을 때, 시스템 밝기가 바뀔 때의 동작 | **확정.** 4.11.14절(A-60). 우리 앱에서 OS 밝기 변경을 어떻게 감지하는지(gpui·Tauri 각각)는 **미확인**이며 02-gpui·03-tauri 출발 문서의 검토 항목으로 넘긴다 (A-60.12) | 확정 (조사 X-105 ~ X-109) + 기술 확인 |
| A-57.9 | 확장 화면의 툴팁·우클릭 메뉴·키 | 아직 조사하지 않았다. 01(L-200 ~)·02(I-500 ~)에 항목이 생기면 연결한다 | 조사 중 |

#### 4.11.13 언어 선택 규칙 (A-59)

확장 조사 보충(`zed-ext-followup.md`, ID `X-100 ~ X-104`)에서 소스로 확정했다. 모든 값은 고정 커밋의 `available_languages.rs`, `language_registry.rs`, `extension_host.rs` 에서 읽었다.

| ID | 항목 | 규칙 | 구분 | 근거 |
| --- | --- | --- | --- | --- |
| A-59.1 | 등록 순서 | 목록은 ① `Plain Text`(확장자 `txt`, 맨 앞) ② 본체 내장 언어(코드에 적힌 순서) ③ 확장 언어(색인의 **언어 이름 사전순**) 이다. 세션 중에 설치·다시 읽은 확장 언어는 목록 끝에 붙는다. 이 목록을 **뒤에서부터** 훑는다 (나중에 등록된 것이 먼저 평가된다) | 확정 | X-100, X-102 |
| A-59.2 | 후보 문자열 | 파일 하나에서 후보 배열을 만든다: ① 확장자(마지막 `.` 뒤. `.env` 처럼 점으로 시작하는 파일도 `env`) ② 파일 이름 전체 ③ 작업 영역 기준 상대 경로 ④ 절대 경로 ⑤ Windows 에서 `\` 가 있으면 `/` 로 바꾼 경로 | 확정 | X-101 |
| A-59.3 | 사용자 설정 `file_types` | Zed 는 설정의 `file_types`(언어 이름 → 글로브 목록)를 가장 먼저 본다. 우리 앱은 이 설정을 **만들지 않는다**(A-21 에 없다). 그래서 이 단계는 없다 | 확정(Zed) + 판단(없앰) | X-101 |
| A-59.4 | 기본 `path_suffixes` | 언어의 `path_suffixes` 각 항목에 대해 `.` + 항목 을 만들고, 후보 중 `그것으로 끝나거나 항목과 같은` **첫 후보의 길이**를 구한다. 그 언어의 모든 항목 가운데 **가장 긴 길이**가 그 언어의 점수다(1 이상이면 일치) | 확정 | X-101 |
| A-59.5 | `first_line_pattern` | 버퍼 첫 줄(최대 256열)을 정규식으로 검사한다. **아직 일치한 언어가 없을 때만** 검사하고 점수는 1 이다 | 확정 | X-101 |
| A-59.6 | 비교 | 뒤에서부터 훑으며 현재 최고 점수보다 **엄격히 큰** 점수일 때만 바꾼다. 점수가 같으면 바꾸지 않아서 **먼저 만난 것(나중에 등록된 언어)** 이 이긴다. 경로 일치(점수 1 이상)가 있으면 `first_line_pattern` 일치는 절대 이기지 못한다 | 확정 | X-102 |
| A-59.7 | 예 | `.env` 파일: `Env`(항목 `.env` 가 파일 이름 `.env` 와 맞아 길이 4)가 `Shell Script`(항목 `env` 가 확장자 `env` 와 맞아 길이 3)를 이긴다. 같은 확장자(`rs`, `md`)를 서로 다른 이름의 내장 언어와 확장 언어가 가지면 **확장 언어**가 이긴다(나중에 등록). 서로 다른 두 확장 언어가 같은 확장자를 가지면 **언어 이름 사전순에서 뒤쪽**이 이긴다 | 확정 | X-103 |
| A-59.8 | 같은 언어 이름의 충돌 | ① 내장이 이미 있고 확장이 같은 이름을 등록하려 하면 확장을 **거부**하고 경고를 남긴다(내장 유지). ② 확장이 먼저 있고 내장이 나중에 등록되면 내장이 그 자리를 덮어쓴다. ③ 두 확장이 같은 이름을 주면 색인을 만들 때 **나중에 읽힌 확장의 항목이 덮어쓴다.** Zed 는 이 순서를 폴더 열거 순서에 맡기므로 결정적이지 않다. **우리 앱은 확장 `id` 의 사전순에서 뒤쪽이 이긴다**고 정한다. ④ 이미 읽은 같은 이름의 언어가 있으면 그 자리를 새 내용으로 갈아 끼우고 읽은 인스턴스를 버린다 | 확정(①②④) + 판단(③) | X-100, X-38 |
| A-59.9 | 문법 이름 충돌 | 내장 문법과 이름이 같은 확장의 WASM 문법은 경고를 남기고 **무시**한다. 내장 문법이 항상 이긴다 | 확정 | X-100 |
| A-59.10 | 다른 조회 함수 | 코드 블록 이름·확장자로 언어를 찾을 때(마크다운 코드 펜스, `code_fence_block_name`)는 **뒤에서부터**(나중 등록 우선). 확장자만으로 언어 이름을 찾는 조회와 모델라인(`ft=`, `mode:`) 조회는 **앞에서부터**(먼저 등록된 내장 우선)다. 파일을 열 때는 위 A-59.1 ~ A-59.6 | 확정 | X-104 |
| A-59.11 | 우리 앱의 내장 언어 | 본체에 내장할 언어의 목록과 순서는 정하지 않았다 (A-40.17). 목록이 정해지면 위 규칙이 그대로 적용된다. Zed 는 Plain Text 와 내장 23종(bash, c, cpp, css, diff, env, go, gomod, gowork, json, jsonc, markdown, markdown-inline, python, rust, tsx, typescript, javascript, jsdoc, regex, yaml, gitcommit, zed-keybind-context)을 가진다 | 확정(Zed) + 미확인(우리 목록) | X-100 |
| A-59.12 | 구현 확인 | 위 규칙을 어떤 자료구조로 구현하든 같은 결과가 나와야 한다. 점검용 사례(`.env`, `README.md`, `Dockerfile`, `foo.d.ts`, 첫 줄만 단서인 스크립트, 같은 확장자를 주장하는 확장 둘)를 02-gpui·03-tauri 출발 문서의 검토 항목으로 넘긴다 | 기술 확인 | — |

#### 4.11.14 확장을 읽는 중과 시스템 밝기 변경 (A-60)

확장 조사 보충(ID `X-105 ~ X-109`)에서 소스로 확정했다. 03 의 6.4절(T-32.15 ~ T-32.19)이 같은 내용을 테마·아이콘 테마 쪽에서 적는다.

| ID | 항목 | 규칙 | 구분 | 근거 |
| --- | --- | --- | --- | --- |
| A-60.1 | 시작 단계 | Zed 는 ① 기본 등록부를 만들고 ② 번들 테마를 읽어 **설정에 맞는 테마를 먼저 적용**하고 ③ 색인에 있는 **현재 활성 확장 테마·아이콘 테마 하나씩만 동기로 먼저 읽어** 즉시 적용하고(같은 파일의 형제 테마도 함께 등록) ④ 사용자 테마 폴더를 백그라운드로 읽고 ⑤ 모든 확장을 백그라운드로 읽은 뒤 "확장 읽기 끝" 표시를 켜고 현재 테마·아이콘 테마를 다시 적용하고 창을 새로 그린다 | 확정 | X-105 |
| A-60.2 | 우리 앱의 시작 단계 | ①②는 같다(One, Zed (Default), Catppuccin 은 본체 내장이라 ①에서 등록). ③은 같이 한다: 색인(A-50.6)에 있는 현재 활성 확장 테마·아이콘 테마만 창이 뜨기 전에 먼저 읽는다. ④의 사용자 테마 폴더(`%APPDATA%\Zed\themes` 에 해당)는 **만들지 않는다** (사용자 테마는 확장으로 설치, 판단). ⑤는 같다 | 확정 + 판단 | X-105 |
| A-60.3 | 폴백 | 설정이 가리키는 이름이 등록부에 없으면 **OS 밝기**를 보고 라이트면 `One Light`, 다크면 `One Dark` 를 쓴다. 설정의 `mode` 가 `light`/`dark` 로 고정이어도 폴백은 OS 밝기 기준이다. 그것도 없으면 `One Dark`. 아이콘 테마는 `Zed (Default)`. 이름이 없다는 **오류 로그는 확장 읽기가 끝난 뒤에만** 남긴다 | 확정 | X-106 |
| A-60.4 | 읽는 중 선택기 열기: 목록 | 테마·아이콘 테마 선택기는 열릴 때 등록부를 **한 번만 복사**한다(어두운 것 먼저, 이름 순). 열려 있는 동안 확장 읽기가 끝나도 목록이 갱신되지 않는다. 닫았다 다시 열어야 보인다 | 확정 | X-107 |
| A-60.5 | 읽는 중 선택기 열기: 초기 선택·체크 | 테마 선택기는 지금 **적용 중인 테마**(폴백일 수 있다)에 체크와 초기 선택을 둔다. 아이콘 테마 선택기는 **설정에 적힌 이름** 기준이라 그 이름이 목록에 없으면 첫 항목이 선택되고 체크는 어디에도 없다 | 확정 | X-107 |
| A-60.6 | 읽는 중 선택기 열기: 미리 적용·취소 | 열 때 선택된 항목을 메모리에서 바로 적용한다. 취소하면 열기 전 설정으로 되돌린다. 그사이 확장 읽기가 끝났으면 되돌린 **원래 확장 테마가 정상 적용된다**. 아이콘 테마는 원래 이름을 문자열 고정 형식으로 메모리에서 되돌린다 | 확정 | X-107 |
| A-60.7 | OS 밝기가 바뀔 때 (`mode: system`) | 창의 밝기 변화를 감지하면 ① 전역 시스템 밝기를 갱신하고 ② 테마·아이콘 테마를 다시 계산해(`Dynamic` 이면 라이트 → `light` 칸, 다크 → `dark` 칸. 문자열(고정)이면 그대로) ③ 모든 창을 다시 그리고 ④ 이미 읽은 모든 언어의 구문 색 표를 새 테마로 다시 만든다. `mode` 가 `system` 이면 창 틀(타이틀 바 등 OS 장식)의 밝기를 강제하지 않고 OS 를 따르고, `light`/`dark` 고정일 때만 강제한다 | 확정 | X-108 |
| A-60.8 | 바뀐 밝기의 테마가 아직 안 읽혔을 때 | 같은 확장 파일(패밀리)에 속하면(예: `catppuccin-mauve.json` 안의 Mocha 와 Latte) 이미 등록되어 즉시 바뀐다. 다른 확장 패밀리이고 읽기 전이면 폴백(A-60.3)을 잠시 쓰다가 읽기가 끝나는 순간 설정 테마를 적용한다 | 확정 | X-108 |
| A-60.9 | 선택기가 열린 채 OS 밝기가 바뀔 때 | 미리 적용은 선택기를 열 때 저장한 밝기를 쓰고, 확정(`Enter`)은 **확정하는 순간의 밝기**를 쓴다. OS 밝기가 바뀌면 화면에는 바로 반영된다. 취소하면 열기 전 설정 전체로 복원한다 | 확정 | X-109 |
| A-60.10 | 선택기 확정 때의 `mode` 규칙 | `Dynamic { mode: system }` 이고 고른 테마의 밝기가 지금 OS 밝기와 **같으면** `mode` 는 `system` 그대로 두고 그 밝기의 칸만 새 이름으로 바꾼다. **다르면** 새 테마가 보이도록 `mode` 를 그 밝기(`light`/`dark` 고정)로 바꾸고 그 밝기의 칸을 바꾼다. 어느 쪽이든 반대 칸은 선택기를 열기 전 값을 유지한다 (03 의 T-21.22, T-21.23 과 같다) | 확정 | X-109, 03 T-21 |
| A-60.11 | 내장 테마와의 관계 | One·Catppuccin 은 ①에서 등록되므로 읽는 중 폴백이 일어나지 않는다. 폴백(A-60.3, A-60.8)은 설치한 확장의 테마·아이콘 테마에만 해당한다 | 판단 | A-46.8 |
| A-60.12 | 우리 앱에서 OS 밝기 변경 감지 | gpui 는 창의 밝기 관찰 기능, Tauri 는 창 테마 변경 이벤트나 웹의 `prefers-color-scheme` 로 같은 흐름(A-60.7)을 얻을 수 있는지 **미확인**이다. 02-gpui·03-tauri 출발 문서의 검토 항목으로 넘긴다 | 기술 확인 (넘김) | — |

## 5. 설정에서 기본값과 기준 화면 값

| ID | 항목 | Zed 기본 | 기준 화면 | 우리 앱 출고 기본값 |
| --- | --- | --- | --- | --- |
| A-29.1 | 테마 | One Light / One Dark | One Light / Catppuccin Mocha | Zed 기본 |
| A-29.2 | 아이콘 테마 | Zed (Default) | Catppuccin Latte | Zed 기본 (단, 원본과 모양이 다르다, T-31) |
| A-29.3 | 글꼴 크기 | UI 16, 편집기 15 | UI 16, 편집기 15 | 같음 |
| A-29.4 | 글꼴 | `.ZedSans`(= IBM Plex Sans), `.ZedMono`(= Lilex) | 같음 | 같음. 한글은 시스템 글꼴 (T-15.5, 앱 판단 T-16) |
| A-29.5 | 프로젝트 패널 위치 | 오른쪽 (L-125) | **왼쪽** (사용자 설정) | **왼쪽**. 우리는 위치를 바꿀 수 없게 고정한다 (**우리 앱 값**) |
| A-29.6 | 사이드바 폭 | 300 (`agent.threads_sidebar.default_width`, L-124) | 약 200 (사용자가 조정, L-33) | 300. 설정 키는 `sidebar.default_width` (**우리 키**) |
| A-29.7 | 프로젝트 패널 폭 | 240 | 약 285 (사용자가 조정, L-32) | 240 |
| A-29.9 | autosave | `off` | `off` (사용자 설정에 없음) | `off` |
| A-29.10 | 창 | 단일 창 시작 | 창 하나 | 마지막 세션의 모든 창을 복원 (A-25.10) |
| A-29.8 | keymap | `base_keymap: Zed` | `base_keymap: VSCode` | VSCode 덧씌움 결과로 고정 (**우리 앱 값**, 4.8절) |

우리 앱의 출고 기본값은 위 마지막 열이다. Zed 와 다른 것은 A-29.5, A-29.6(키 이름), A-29.8 이다. 기준 화면과 겹쳐 볼 때는 4.2절의 두 번째 JSON 을 쓴다.

## 6. 구현 순서 제안 (판단)

두 구현이 같은 순서로 쌓이도록 02-gpui·03-tauri 의 번호(01-SETUP … 05-THEME-KEYMAP)와 맞춘다.

| ID | 단계 | 만드는 것 | 이 문서의 항목 |
|---| --- | --- | --- |
| A-36.1 | 1 | 빈 창, 사이드바·Dock·탭 바·상태 표시줄 뼈대 | A-11, L-11 ~ L-17 |
| A-36.2 | 2 | 폴더 등록·선택·해제, 트리 | A-13 ~ A-16, A-22 |
| A-36.3 | 3 | 탭·편집기·마크다운 미리보기, 핀, 탭 분할·이동 | A-04, A-26, A-22.10 ~ A-22.21 |
| A-36.4 | 4 | 테마·아이콘 테마·키맵·툴팁, 메뉴 | A-21, A-17, A-18, A-19, T-30 |
| A-36.6 | 4a | 찾기·바꾸기, 명령 팔레트, Go to File, Open Recent, 키맵 편집 화면, About | A-43 (조사 E-, P-, K- 가 들어온 뒤) |
| A-36.5 | 5 | 상태 복원, 일치도 판정 | A-25, 7절 |

## 7. 일치도 판정 기준

"Zed 와 거의 같다" 를 숫자와 예·아니오로 판정한다. 같은 표를 gpui-kit 판과 Tauri 판에 똑같이 적용한다 (README 의 짝 규칙). 결과는 04-COMPARISON.md 의 체크리스트에 옮긴다.

### 7.1 비교 방법 (A-30)

| ID | 항목 | 내용 |
| --- | --- | --- |
| A-30.1 | 기준 | 기준 화면 `assets/zed-reference.png` (2880×1704). **배율 200% 는 가정이다.** 타이틀 바 높이 32 논리 px 가 이미지에서 64 물리 px 로 보여서 역산했을 뿐, Windows 디스플레이 설정으로 확인하지 않았다. 가정이 맞으면 논리 px = 물리 px ÷ 2 |
| A-30.2 | 맞춰 놓을 설정과 불확실성 | 4.2절 두 번째 JSON(Catppuccin Mocha, Catppuccin Latte 아이콘), UI 16, 편집기 15, 창 크기 1440×852 논리 px(= 2880×1704 ÷ 2, 가정), 배율 200% 로 그려 같은 크기에서 비교한다. 배율 가정 때문에 **이미지로 잰 치수는 보조 증거**이고, 치수 판정의 기준은 소스 값(7.3절의 L- 값)이다. 이미지 측정은 물리 px 1(= 논리 0.5)의 읽기 오차를 따로 인정한다 |
| A-30.6 | 두 번째 기준 화면 | `assets/zed-reference-latte.png` (2448×1480 물리 px). Catppuccin Latte(라이트), 창을 최대화하지 않았다. 이 화면으로 라이트 테마의 색(03 T-08.7 ~ T-08.18), 최대화 전 창 버튼(최대화 □), 창 가장자리·모서리(T-08.19, T-08.20), 상태 표시줄 커서 위치 `1:1` 을 대조한다. 앱이 그리지 않는 창 테두리·모서리는 판정 대상이 아니다(OS 몫) |
| A-30.3 | 맞춰 놓을 내용 | 기준 화면과 같은 폴더 구조(`projects` 아래 `docs\guides\…`), 같은 탭(README.md, Preview README.md, …), 같은 스크롤 |
| A-30.4 | 비교 도구 | 화면 캡처 + 눈금 측정, 요소 검사(Tauri 는 개발자 도구, gpui 는 로그로 치수 출력). 색은 캡처 픽셀과 토큰 표(03)를 비교 |
| A-30.5 | 판정 | 항목마다 **통과/실패**. 필수 항목은 하나라도 실패하면 그 범주가 실패다 |

### 7.2 범주별 기준 (A-31)

| ID | 범주 | 통과 기준 | 필수 | 기준 ID |
| --- | --- | --- | --- | --- |
| A-31.1 | 치수 | 아래 7.3절 모든 값이 **±1 논리 px** 안 | 예 | L-21 ~ L-40, A-11 |
| A-31.2 | 글꼴 크기 | UI 16, 편집기 15, 마크다운 제목 28·22.4·19.2·16·14 가 ±0.5 안 | 예 | L-05, L-08, L-104, T-14 |
| A-31.3 | 키 | **VSCode 를 적용한 keymap**(4.8절)의 채택 키 전부가 같은 동작. 연속 키(`Ctrl+K …`)도 같다. 표는 02 의 1절·3.1.4절을 쓴다 | 예 | 02 1절, I-731 ~ I-739, A-17, A-27 |
| A-31.4 | 동작: 탭 | 임시 탭 교체, 핀, 닫을 때 다음 활성 탭, 더블클릭 승격이 Zed 와 같다 | 예 | I-060 ~ I-076 |
| A-31.4a | 동작: 탭 분할·이동 | 분할(버튼·키·메뉴), 탭 드래그(같은 Pane·다른 Pane·새 분할·복제), 분할선, 포커스 이동, 확대가 Zed 와 같다 (4.7절) | 예 | A-26, I-706 ~ I-739 |
| A-31.4b | 동작: 핀 탭 | 핀 지정·해제, 앞쪽 고정, 일반 닫기에서 보호, 복원 | 예 | I-065, I-066, I-719, A-42.8 |
| A-31.16 | 포함 기능 | 1.1절(A-42.1 ~ A-42.14)의 기능이 02·조사 문서가 정한 대로 동작한다. 조사(E-, P-, K-)가 안 들어온 기능은 "판정 불가" 로 둔다 | 예 | A-42, A-43 |
| A-31.18 | 내장 뷰어 | 마크다운 미리보기가 A-47.5 대로 동작하고, 새 뷰어를 `ViewerSpec` 로 등록하면 제목·아이콘·툴팁·메뉴·복원이 같은 규칙으로 따라온다. 이미지 뷰어는 조사(A-47.6)가 들어온 뒤 판정한다 | 예 | A-47, A-47.5 |
| A-31.20 | 확장 화면 | A-58 대로: `Ctrl+Shift+X` 로 열림, 필터 4개, 설치된 확장 카드(110 높이, 이름·버전·배지·Uninstall), 로컬 설치(폴더·파일)·개발용 설치, 제거, 검색 | 예 | A-58 |
| A-31.19 | 확장 | ① 설치된 Catppuccin·Catppuccin Icons·HTML 확장 폴더를 `installed/` 에 넣으면 재빌드 없이 선택기·파일 아이콘·구문 강조에 나타난다. ② 제거하면 사라지고 선택돼 있던 테마는 화면이 내장 기본값으로 돌아간다. ③ HTML 확장의 `extension.wasm` 과 `[language_servers]` 는 오류 없이 무시된다. ④ `schema_version` 2 짜리·경로가 `..` 인 확장은 그 확장만 건너뛴다. ⑤ `index.json` 을 지워도 다시 만들어진다 | 예 | A-46, A-48 ~ A-56 |
| A-31.17 | 저장·닫기 | autosave `off` 에서 수정 점, 닫기 확인(A-44.1 ~ A-44.5), 앱 종료 때 묻지 않고 복원 | 예 | A-44, A-25.8 |
| A-31.5 | 동작: 트리 | 클릭·방향키·펼침·이름 변경·생성·휴지통·영구 삭제·끌어 놓기 | 예 | I-087 ~ I-097 |
| A-31.6 | 동작: 마크다운 | 미리보기 열기, 같은 탭 재사용, 실시간 반영, 링크, 원문으로 돌아가기 | 예 | I-077 ~ I-086 |
| A-31.7 | 동작: 포커스 | 표시 토글과 포커스 토글을 구분한다 | 예 | I-098 ~ I-104 |
| A-31.8 | 색 토큰 | 03 의 값 표 6개 테마 전부에서 우리가 쓰는 토큰이 다음과 같다: **원본 JSON 에 값이 있으면 그 값과 완전히 같다**(알파 포함). 원본에 값이 없거나 `null` 이면 03 의 T-26 규칙으로 정한 값과 같다("미사용" 으로 정한 토큰은 대상이 아니다). 화면 샘플은 알파 합성 뒤 채널당 ±1 | 예 | T-02 ~ T-06, T-24, T-25, T-26 |
| A-31.9 | 아이콘 | UI 아이콘이 03 의 T-10 표와 같은 의미·같은 크기. 파일·폴더 아이콘이 **03-THEME-ICONS 4.3절(T-22)** 규칙으로 고른 경로와 같다. `Zed (Default)` 는 원본과 모양이 달라 "원본과 다름" 으로 표시한다 (T-31) | 예 | T-10, T-20, T-22, T-28, T-31 |
| A-31.10 | 테마 선택 | 미리 적용, 취소 시 되돌림, 확정 시 저장, `mode` 규칙이 같다 | 예 | T-21 |
| A-31.11 | 폴더 사이드바 | 3절의 등록·선택·해제·구분 규칙 | 예 | A-10 ~ A-17 |
| A-31.12 | 상태 복원 | 4.6절 규칙. 분할 구조 복원 포함 | 예 | A-25, I-112 ~ I-120 |
| A-31.13 | **툴팁** | 7.5절 | 예 | L-200 ~ L-272, T-30 |
| A-31.14 | 범위 밖 | 2절에 적은 것이 화면·메뉴·설정에 하나도 없다 | 예 | A-05 |
| A-31.15 | 메뉴 | ☰ 메뉴(3.9절), ∨ 메뉴(3.10절), 우클릭 메뉴의 구조·항목·문구·키 표시·모양이 Zed 와 같고, 쓸 수 없는 항목만 빠졌다. 사이드바 우클릭은 `Move Up`, `Move Down`, `Remove` | 예 | A-18, A-19, I-300 ~ I-488, I-600 ~ I-655, I-500 ~ I-668, I-900 ~ I-952 |

### 7.3 치수 점검표 (A-32)

100% 배율의 논리 px 다. 측정 방법은 요소의 바깥 상자(테두리 포함)다. 기대 값은 소스 값(L-)이고, 기준 화면으로는 보조 확인만 한다 (A-30.2).

| ID | 점검 | 기대 값 | 기준 ID |
| --- | --- | --- | --- |
| A-32.1 | 타이틀 바 높이 | 32 | L-21 |
| A-32.2 | 창 버튼 | 각 36×32, 글리프 10 | L-39 |
| A-32.3 | 탭 바 높이 / 탭 내용 높이 | 32 / 31 | L-22, L-23 |
| A-32.4 | 탭 안쪽 padding·gap | 4 / 4 | L-24 |
| A-32.5 | 탭 슬롯 | 시작 12×12, 끝 14×14 | L-25 |
| A-32.6 | 탭 바 양 끝 영역 padding | 좌우 6, gap 4 | L-27 |
| A-32.7 | 상태 표시줄 높이 | 30 | L-28 |
| A-32.8 | 상태 표시줄 gap | 그룹 사이 8, 그룹 안 4 | L-29 |
| A-32.9 | Dock 리사이즈 핸들 | 6 | L-30 |
| A-32.10 | 프로젝트 패널 기본 폭 | 240 | L-32 |
| A-32.11 | 사이드바 폭 | 기본 300, 범위 200~800 | L-33 |
| A-32.12 | 사이드바 폴더 줄 | 높이 31, 좌 8, 우 6 | L-34 |
| A-32.13 | 사이드바 푸터 | padding 4, gap 4, 위 테두리 1 | L-35 |
| A-32.14 | 트리 항목 높이 | 약 25~26 (소스에 명시값 없음, 기준 화면 26) | L-36 |
| A-32.15 | 트리 들여쓰기 | 단계당 20. 안내선 폭 1, x = 깊이×20 + 15 | L-37, L-38 |
| A-32.16 | 아이콘 크기 | 탭 바·상태 표시줄 14, 트리 16 | L-06, L-40 |
| A-32.17 | 마크다운 본문 | 폭 최대 800, 바깥 여백 16, 글자 16, 줄 높이 1.5 | L-101 ~ L-103 |
| A-32.18 | 마크다운 표 | 바깥 테두리 1.5, 셀 padding 10×4 | L-107 |
| A-32.19 | 마크다운 인용 | 왼쪽 테두리 4, padding-left 16 | L-106 |
| A-32.20 | ∨ 버튼 | 22×22, 아이콘 14, 모서리 4 | I-608, I-609 |
| A-32.21 | 메뉴 최소 폭 | 200 (☰, ∨ 메뉴 공통) | I-313, I-612 |

### 7.4 키·동작 점검 방법 (A-33)

| ID | 방법 | 비고 |
| --- | --- | --- |
| A-33.1 | 키 표(I-003 ~ I-059, 그리고 VSCode 덧씌움 결과 I-731 ~ I-739, 02 1절)의 각 줄을 입력해 동작을 확인한다 | 컨텍스트가 다르면 같은 키가 다른 동작이다 (I-002). 포커스 위치를 같이 적는다 |
| A-33.2 | 시나리오: 트리에서 `README.md` 한 번 클릭 → 임시 탭(제목 기울임) → 다른 파일 한 번 클릭 → 같은 자리에서 교체 → 더블클릭 → 일반 탭 | I-061, I-062, I-064, I-087 |
| A-33.3 | 시나리오: 마크다운 탭에서 `Ctrl+Shift+V` → `Preview README.md` 탭(책 아이콘), 다시 `Ctrl+Shift+V` → 원문으로 | I-044 ~ I-045, I-086, L-69 |
| A-33.4 | 시나리오: 폴더 둘 등록 → 각각 탭을 연다 → 전환해도 탭이 따로 유지된다 → 하나 해제 → 디스크에 그대로 있다 | A-13, A-15 |
| A-33.5 | 시나리오: 종료 후 다시 실행 → 폴더·활성·탭·임시·핀·Dock 폭이 복원, 트리 펼침은 복원 안 됨 | A-25 |
| A-33.6 | 시나리오: 테마 선택기에서 `Down` 으로 이동 → 즉시 바뀜 → `Esc` → 원래 테마 → 다시 열어 `Enter` → 저장 | T-21 |
| A-33.7 | 시나리오: 탭 바의 분할 버튼 → Split Right → 같은 폴더 안에 Pane 둘 → 탭을 끌어 다른 Pane 으로 → 가장자리에 놓아 새 분할 → 종료 후 재실행해 구조 복원 | A-26, A-22.9 |
| A-33.9 | 시나리오: 폴더 둘 등록 → 우클릭 Move Down → 순서가 바뀐다 → 종료 후 재실행해도 순서 유지 → Remove → 디스크에 그대로 | A-13.6, A-22.2 |
| A-33.10 | 시나리오: 파일 수정(●) → 탭 닫기 → Save / Don't Save / Cancel → 수정한 채 앱 종료 → 다시 실행하면 수정한 내용이 그 탭에 있고 묻지 않았다 | A-44, A-25.8 |
| A-33.11 | 시나리오: `Ctrl+Shift+N` → 빈 창 → 폴더 등록 → 첫 창과 상태가 섞이지 않는다 | A-43.1 |
| A-33.12 | 시나리오: 마크다운 파일을 연다 → `Ctrl+Shift+V` → `Preview X.md` 탭(책 아이콘, 원본과 같은 툴팁) → 원본을 고치면 미리보기가 따라온다 → 임시 탭은 교체되지 않는다 | A-47.3, A-47.5 |
| A-33.15 | 시나리오: `Ctrl+Shift+X` → 확장 화면 → `Install Extension` → `Folder…` 로 `catppuccin` 폴더를 고른다 → 카드가 나타나고 테마 선택기가 그 확장의 테마만 걸러서 열린다 → `Uninstall` → 카드가 사라진다 | A-58, A-52.4, A-52.8 |
| A-33.13 | 시나리오: `catppuccin` 폴더를 `installed/` 에 넣는다 → 200ms 뒤 선택기에 테마 4종(와 No Italics 4종)이 나타난다 → 선택·저장 → 폴더를 지운다 → 화면은 내장 기본값, `settings.json` 의 `theme` 은 그대로 | A-50.8, A-52.3 |
| A-33.14 | 시나리오: `html` 확장 폴더를 넣는다 → `.html` 파일에 구문 강조가 붙는다 → `extension.wasm` 을 건드리지 않는다 | A-46.5, A-49.3 |
| A-33.8 | 시나리오: ∨ 버튼 → 메뉴 열림(버튼 선택 표시) → Themes… 선택 → 선택기가 열림 → `Esc` 로 닫으면 이전 포커스 복귀 | A-19 |

### 7.5 툴팁 점검 (A-31.13, A-34)

사용자 요건: 아이콘에 마우스를 올렸을 때 나오는 툴팁도 Zed 와 같게 한다(문구, 단축키 표시, 지연, 위치, 스타일). 조사는 L-200 ~ L-272 (작성 중)에 있다. 03 의 T-30.5·T-18.2 는 위치 규칙을 확인하지 못했다고 적었으나, 이후 조사(L-208 ~ L-210)에서 확인됐다. 판정은 L- 값을 따른다.

| ID | 점검 | 통과 기준 | 근거 | 상태 |
| --- | --- | --- | --- | --- |
| A-34.1 | 문구 | 버튼마다 Zed 와 같은 문구가 뜬다. 예: ☰ `Open Application Menu`(L-225), 폴더 추가 `Add Project`(L-230), ← `Go Back`·→ `Go Forward`(L-236, L-237), 탭 닫기 `Close Tab`(L-240), `+` `New…`(L-243), 분할 `Split Pane`(L-244), 최대화 `Zoom In`/`Zoom Out`(L-245), 프로젝트 패널 `Project Panel`(L-253)·열려 있을 때 `Close Left Dock`(L-254), 사이드바 닫힘 `Open Threads Sidebar`(L-251)·열림은 두 줄 `Toggle Sidebar` / `Focus Sidebar`(L-232). **창 버튼과 ∨ 버튼에는 툴팁이 없다**(L-228, I-637). 문구에 "Threads" 가 들어간 것은 우리 앱에 맞게 바꾼다(`Open Sidebar`, 판단) | L-225 ~ L-255, I-637 | 확인됨(버튼별). 일부 L- 값은 작성 중 |
| A-34.2 | 단축키 표시 | 액션에 키가 묶인 버튼은 툴팁 오른쪽 끝에 키를 `Ctrl-Shift-E` 형식(하이픈, 순서 Fn → Ctrl → Win → Alt → Shift → 키)으로 보인다. 키가 없으면 그리지 않는다. 같은 액션에 키가 여럿이면 우선순위가 가장 높은 하나만. 키는 VSCode 를 적용한 keymap(4.8절)의 결과다. 03 의 T-30.7 은 `+` 형태로 적었으나 실제 형식은 L-219 의 하이픈 형식이다 | L-201, L-202, L-217 ~ L-223, A-27 | 형식 확인됨. VSCode 적용 결과는 I-700 ~ 확정 뒤 |
| A-34.3 | 지연·사라짐 | 마우스가 올라간 뒤 **500ms** 에 보인다. 트리 항목 이름만 1500ms 다. 마우스가 요소를 벗어나거나 버튼을 누르거나 스크롤하면 사라진다. 메뉴를 여는 버튼은 메뉴가 열려 있는 동안 툴팁이 없다 | L-206, L-207, L-205, L-247 | 확인됨 |
| A-34.4 | 위치 | 툴팁을 띄우는 순간의 마우스 위치에서 (+1, +1) 에 놓고, 바깥 상자의 왼쪽 8·위 10 은 투명 여백이다. 오른쪽이 창을 넘으면 커서 왼쪽으로(x − 폭 − 1) 뒤집고 그래도 0 보다 작으면 0, 아래쪽이 넘으면 커서 위쪽으로(y − 높이 − 1) 뒤집는다 | L-208 ~ L-210 | 확인됨 |
| A-34.5 | 모양 | 떠 있는 표면(`elevated_surface.background`, 모서리·테두리·두 겹 그림자는 L-211, L-212), 안쪽 여백 위아래 4·좌우 8, UI 글꼴 14px, 글자색 `text`. 제목은 최대 폭 288, 메타 줄은 Small(12)·Muted | L-211 ~ L-216, T-30.2 | 확인됨 |
| A-34.6 | 비활성 버튼 | 비활성(예: 이동 기록이 없는 ← →) 이어도 툴팁이 뜬다. Zed 의 버튼은 비활성이어도 툴팁을 붙인다 (button_like.rs L883-L895, 검토 자료). 비활성 요소에 hover 이벤트가 도달하는지는 gpui 쪽을 확인하지 못했다 | L-236, L-237 | 확인됨(버튼), gpui 쪽 미확인 |
| A-34.7 | 사이드바 폴더 항목 | 툴팁이 **없다** (A-12.8, L-231). 이전 판의 "전체 경로 툴팁" 은 Zed 와 달라 뺐다 | L-231 | 확인됨 |

검증 방법: 버튼 위에 마우스를 올리고 500ms 를 재서 보인다 → 문구·키를 읽는다 → 캡처해 위치·여백을 잰다 → 창 가장자리에서 뒤집히는지 본다. Zed 와 우리 화면을 같은 버튼에서 비교한다.

### 7.6 합격 선 (A-35)

| ID | 선 |
| --- | --- |
| A-35.1 | 필수 범주(A-31.1 ~ A-31.15, A-31.4a 포함)가 모두 통과하면 "Zed 와 같은 수준" 이다 |
| A-35.2 | 실패한 항목은 원인이 두 구현 모두의 문제인지, 한쪽 프레임워크의 한계인지 적는다 (04-COMPARISON) |
| A-35.3 | 실행 검증은 아직 하지 않았다. 이 문서의 값은 소스·JSON 대조 결과다 |

## 8. 확인하지 못한 것·결정이 필요한 것 (A-40)

| ID | 내용 |
| --- | --- |
| A-40.1 | 사이드바 푸터·상태 표시줄 왼쪽의 버튼 정렬 순서는 Zed 에서 미확인이다. A-11.8~9 는 우리 결정이다 (L-95, L-96) |
| A-40.2 | 확인창의 일부 본문(A-44.3, A-44.4), 기본 버튼·`Esc` 결과, 알림(A-45.1)의 모양은 P-001 ~ 조사가 정한다 |
| A-40.3 | 한 번에 여러 폴더를 고르는 경우를 막은 것은 우리 결정이다 (3.1절). 필요하면 multi-root 항목을 추가한다 |
| A-40.4 | 툴팁 L-200 ~ L-272 의 줄 번호·문구는 01·02 가 최종이다. 이 문서의 인용(A-34)이 다르면 01 을 따른다 |
| A-40.5 | 앱 이름·설정 폴더 이름(A-20.3)과 고지 파일 위치(T-17)는 정하지 않았다 |
| A-40.6 | 프로젝트 패널 항목 높이(L-36), 들여쓰기 단계(L-37)의 관찰값 차이는 01-LAYOUT 의 확신 낮은 부분을 따른다 |
| A-40.7 | 분할·이동 세부(크기 조절 수식, 거리가 같을 때의 방향, 항목 고유 드롭)는 검증 전이다 (A-26.14) |
| A-40.8 | 키맵 편집 화면(A-43.9)은 K-001 ~ 조사가 들어와야 명세할 수 있다 |
| A-40.9 | 우클릭 메뉴 ID 는 id-renumber.md 대로 반영했다(사이드바 Move Up/Down/Remove 는 I-934, I-940, I-935). 우클릭 메뉴의 나머지 항목(프로젝트 패널·탭·편집기)은 아직 이 문서에 옮기지 않았다. 02 11절이 정한다 |
| A-40.10 | VSCode 덧씌움으로 바뀌는 키는 02 의 표가 정한다. 이 문서에 적은 키(A-17, A-18, A-19.11, A-33.1)는 02 와 대조해야 한다 |
| A-40.11 | 배율 200% 는 가정이다 (A-30.1) |
| A-40.12 | 사이드바 폭 조절 핸들(A-11.12)은 01 조사 대기다 |
| A-40.13 | 사이드바 폴더 항목을 끌어서 옮기는지는 확인하지 못했다 (A-12.11) |
| A-40.14 | 찾기·바꾸기(E-001 ~), Command Palette·Go to File·Open Recent·About·삭제 확인창의 외형(P-001 ~)은 조사 대기다 (A-43) |
| A-40.15 | `recent` 최대 개수 20, 새 창이 빈 창으로 열린다는 점, 저장 안 한 버퍼 파일 위치는 우리 결정이다 (A-22.23, A-43.1, A-20.5) |
| A-40.16 | 최대화한 창의 복원 위치·크기 보존(A-22.9)과 창 모서리·테두리(T-08.19, T-08.20)는 OS 가 그리는 것으로 보이며 앱이 정하지 않는다 (추정) |
| A-40.19 | 확장 세부 결정(로컬 설치만, 확장 화면, Catppuccin 내장, 끄기 없음)은 4.11절에 반영했다. 남은 것은 A-57.3, A-57.7(02-gpui·03-tauri 로 넘김)과 A-57.6, A-57.8, A-57.9(조사 중)이다 |
| A-40.20 | 이미지 뷰어 조사(A-47.6)가 필요하다. Zed 에 "원문으로 열기" 명령이 있는지(A-47.2c)도 같이 본다 |
| A-40.21 | 확장 후속 조사(X-100 ~ X-109)로 A-57.6, A-57.8 은 확정했다. 남은 **미확인**: 우리 앱의 내장 언어 목록(A-40.17), OS 밝기 변경 감지 방법(A-60.12), 규칙 구현의 점검 사례(A-59.12). 뒤의 둘은 02-gpui·03-tauri 출발 문서의 검토 항목으로 넘겼다 |
| A-40.17 | 코드 구문 강조를 내장할 문법(Tree-sitter) 언어 목록은 정하지 않았다. 03 의 아이콘 매핑 범위와 별개다. 04-COMPARISON 전에 정한다 |
| A-40.18 | 창 최소 크기(검토 자료: 360×240)와 타이틀 바 더블클릭 최대화는 이 문서에 넣지 않았다. 01 의 조사가 들어오면 따른다 |

---

이전: [03-THEME-ICONS.md](03-THEME-ICONS.md) · 다음: [../02-gpui/README.md](../02-gpui/README.md) · 입구: [../README.md](../README.md)
