---
title: ego lite — 코딩 에이전트용 브라우저 (Claude Code · Codex · agy)
sources:
  - raw/web/261101-ego-lite-guide/ego-lite-guide.md
updated: 2026-11-01
tags: [ego-lite, ego-browser, browser-automation, claude-code, codex, agy, windows]
status: draft
---

# ego lite — 코딩 에이전트용 브라우저 (Claude Code · Codex · agy)

ego lite를 Windows 11에서 Claude Code, Codex CLI, Antigravity CLI(agy) 세 에이전트의 브라우저로 쓰는 방법이다. 에이전트가 사용자가 로그인해 둔 사이트에서 데이터를 읽고, 폼을 채우고, 웹앱을 점검하게 하는 것이 목적이다.

원본 기준: ego lite 0.6.2.5(Chromium 156, 내장 Node v24), 사용 가이드 v2.2.0, agy 1.4.11, 2026-10-31. 세 에이전트 모두 "스킬 호출 → example.com 제목 읽기 → Space 닫기"까지 확인했고, 사이트별 예시 작업은 시험하지 않았다. ego lite는 빠르게 바뀌는 베타라 **설치본의 `ego-browser skill` 출력이 기준**이다.

## 1. 무엇인가

- Citro Labs가 만든 무료 **Chromium 기반 브라우저**다. 사람이 평소 쓰는 브라우저이면서 AI 에이전트가 조작하도록 설계됐다. **ego lite**는 브라우저 앱, **ego-browser**는 에이전트가 쓰는 CLI·스킬 이름이다.
- **MCP 서버가 아니다.** 스킬 + CLI 구조다:
  1. 에이전트가 스킬(SKILL.md)을 읽는다.
  2. `ego-browser skill`이 설치 버전에 맞는 사용 가이드를 출력한다.
  3. 에이전트가 여러 동작을 JS 스크립트 하나로 묶어 `ego-browser nodejs`(내장 Node)로 실행한다. CDP로 브라우저를 움직인다.
- 동작마다 도구를 부르지 않으므로 MCP 방식보다 왕복·토큰이 줄어든다고 벤더는 주장한다(벤더 측정: 31개 과제 성공 93.5%·과제당 $1.75 vs Playwright MCP 71%·$4.82). 제3자 리뷰에서는 토큰 절감은 확인됐고 속도는 경우에 따라 달랐다.
- 웹 문서에는 옛 API(`useOrCreateTaskSpace`, `snapshotText`, 초 단위 timeout)가 남아 있다. 설치본 v2.2.0은 `taskSpace`, `page.*`, 밀리초 단위다. 웹 예제를 그대로 쓰지 않는다.

### Space와 로그인

- **Space**는 작업마다 하나씩 생기는 에이전트 작업 공간이다. 에이전트 탭은 Space 안에 열리고 사용자의 탭·마우스·포커스는 건드리지 않는다.
- **로그인은 공유된다.** 다른 브라우저 데이터를 가져오거나 ego lite에서 로그인하면 에이전트도 그 상태로 일한다. 비밀번호를 프롬프트에 넣지 않는다.
- **쿠키는 프로필 단위로 공유된다고 보고 쓴다.** 웹 문서는 "작업별 분리"라고 하지만 설치본 문서는 사용자 탭과 cookie jar·HTTP cache를 공유한다고 적고 있고, 관련 이슈(Space 간 쿠키 누출 등)도 열려 있다.
- 로그인·캡차·2FA·결제는 사람이 한다. 에이전트는 `handOff()`로 넘기고 멈춘다.

## 2. 설치와 확인

1. https://lite.ego.app/ 에서 Windows 설치 파일을 받아 실행한다(SmartScreen은 추가 정보 → 실행). 온보딩을 끝낸다. 온보딩이 `ego-browser`를 PATH에 넣고 각 에이전트 스킬 폴더에 `ego-browser` 링크를 만든다. 다른 브라우저 데이터 가져오기는 한 번 복사이고 이후 동기화되지 않는다.
2. **새 터미널을 열고** 확인한다. 온보딩 전에 연 터미널(과 그 안의 에이전트)은 옛 PATH라 `ego-browser`를 못 찾는다.

```powershell
ego-browser --version
ego-browser nodejs -e "console.log('ego-browser ready')"
ego-browser skill        # 마지막 줄이 "[ego-browser:skill] end of guide v2.2.0"이어야 한다
```

`Unknown command: skill`이면 `ego-browser upgrade`. 데이터 추가 가져오기는 `ego-browser import --browser chrome --profile Default`.

## 3. 에이전트별 설정

| | Claude Code | Codex | agy |
| --- | --- | --- | --- |
| 스킬 위치 | `~/.claude/skills/ego-browser` (온보딩) | `~/.agents/skills/ego-browser` (온보딩) | `~/.gemini/config/skills.json`에 **실제 경로** 등록 (직접) |
| 설치 직후 동작 | 예 | 예 | **아니요** |
| 호출 | `/ego-browser <작업>` | `$ego-browser <작업>` (`/`는 거부됨) | `/ego-browser <작업>` |
| 실행 셸 | Git Bash heredoc | PowerShell + `.mjs` 파일 | PowerShell + `.mjs` 파일 |
| 승인 줄이기 | `settings.json`에 `"allow": ["Bash(ego-browser:*)"]` | 온보딩이 `~/.codex/rules/default.rules`에 `ego-browser` 허용 규칙을 넣음 | `~/.gemini/antigravity-cli/settings.json`에 `"allow": ["command(regex:ego-browser .*)"]` |
| 업그레이드 후 | 없음 | 없음 | skills.json 다시 생성 |

### Claude Code

- `/ego`를 쳐서 `/ego-browser`가 자동완성되는지 본다.
- ego 문서는 `--dangerously-skip-permissions`를 안내하지만 모든 명령을 묻지 않고 실행하게 된다. 대신 `ego-browser`만 허용 목록에 넣는다. 허용하지 않으면 매 라운드 승인이 필요해 느리다(리뷰: 76초·12턴 → 16.4초·2턴).

### Codex

- 스킬은 `$`로 부른다. `$ego`까지 치면 `ego-browser [Skill]`이 자동완성된다. `/ego-browser`는 `Unrecognized command`로 거부된다.
- 온보딩 전에 띄운 Codex는 PATH에서 못 찾아 절대경로로 실행하고, 그러면 허용 규칙과 맞지 않아 매번 승인 창이 뜬다. 새 터미널에서 다시 띄운다.
- `--dangerously-bypass-approvals-and-sandbox`는 OpenAI가 격리된 환경에서만 쓰라고 한다. 허용 규칙이 있으면 필요 없다.

### agy

설치만으로는 스킬을 인식하지 못한다. 설치 프로그램은 agy가 보지 않는 `~/.gemini/antigravity-cli/skills/`에 링크를 만들고, agy는 Junction을 따라가지 않는다. 실제 경로를 `skills.json`에 등록한다(PowerShell 7):

```powershell
$ver  = (Get-Item "$HOME\.local\share\ego\active_version_dir").Target
$path = (Join-Path $ver 'ego-skills') -replace '\\','/'
@{ entries = @(@{ path = $path }) } | ConvertTo-Json -Depth 3 |
  Set-Content -Encoding utf8 "$HOME\.gemini\config\skills.json"
```

- 이미 `skills.json`이 있으면 덮어쓰지 말고 `entries`에 추가한다. 경로에 버전 번호가 들어가므로 **업그레이드 뒤 다시 실행**한다. PowerShell 5.1은 BOM을 붙이므로 7에서 실행한다. 고친 뒤 agy를 다시 띄운다.
- 기본 모드의 agy는 작업 한 번에 승인 창을 세 번 띄운다: ① `ego-browser skill` 실행 ② 스크립트 파일 생성 ③ `ego-browser nodejs` 실행.
  - ①·③은 `settings.json`의 `"allow": ["command(regex:ego-browser .*)"]`로 없앤다. `command(ego-browser)`는 agy 1.4.11에서 맞지 않았다. 승인 창의 "항상 허용"으로 저장하면 그 파일 경로까지 든 명령 전체가 저장되어 다음 작업에 맞지 않는다.
  - ②는 `"allowNonWorkspaceAccess": true`로 없앨 수 있지만, **워크스페이스 밖 모든 파일**(홈, `.ssh`, 다른 프로젝트)을 묻지 않고 읽고 쓰게 하는 넓은 권한이다. 로그인된 브라우저와 함께 쓰면 방문한 페이지의 프롬프트 인젝션이 파일과 세션을 한꺼번에 노출할 수 있다. 혼자 쓰는 PC에서 믿을 수 있는 사이트를 지켜보며 작업할 때만 켜고, 아니면 끄고 `f`로 스크립트를 확인한 뒤 승인한다.
  - 승인 창의 `tab`(Amend)은 허용 범위 수정이 아니라 추가 지시 입력칸이다. 그 상태에서 `esc`를 누르면 작업 전체가 멈춘다.

## 4. 요청 쓰는 법

좋은 요청에는 **목표 · 사이트와 범위 · 출력 형식 · 멈출 조건** 네 가지가 들어간다.

```text
/ego-browser <사이트>에서 <대상>을 찾아 <항목>을 표로 정리해줘. <행동>은 하지 말고, 로그인이 필요하면 멈춰.
```

- "누르지 마"는 약속이지 강제 장치가 아니다. 제출·결제·삭제처럼 되돌릴 수 없는 일은 사람이 한다.
- 비밀번호·인증 코드는 프롬프트에 쓰지 않고 브라우저에서 직접 넣는다.
- 결과 탭을 남기게 할 수 있다(`finish({ keep: ["p1"] })`). 예: 폼을 다 채워 두고 제출은 사람이 한다.
- ego lite에는 예약 실행이 없다. 반복 확인은 에이전트 쪽 기능이나 사람이 다시 요청하는 방식으로 한다.

## 5. 에이전트가 쓰는 스크립트 (읽을 때 알아 둘 것)

```js
const task = await taskSpace("check example page"); // 작업 하나에 Space 하나. 같은 이름이면 이어서 쓴다
const page = task.page("p1");
await page.goto("https://example.com");
console.log(await page.snapshot());                 // 접근성 트리 — 다음 동작을 고르는 근거
await task.finish({ keep: [] });                    // 끝나면 한 번. 에이전트 탭을 정리한다
```

- 실행할 때마다 새 Node 프로세스다. Space와 탭 라벨은 남고 JS 변수는 남지 않는다. 막혔다고 새 Space를 만들지 않고 `taskSpace(<id>)`로 이어 쓴다.
- Playwright가 아니다(`locator()`, `getByRole()` 없음). 셀렉터는 스냅샷 ref(`@21`), `text=…`, `loc=role:button[name='…']`, CSS 등이고 정확히 하나에 맞아야 한다.
- 고정 대기 대신 `waitForURL()`·`waitForSelector()`·`waitForFunction()`. 대량 추출은 `page.evaluate()`, 로그인 쿠키로 API 호출은 `page.fetch()`, 다운로드는 `waitForEvent("download")` → `saveAs(절대경로)`(같은 라운드 안에서).
- PowerShell에서는 여러 줄·큰따옴표 스크립트를 `-e`에 넣지 않고 UTF-8 `.mjs` 파일로 만들어 `ego-browser nodejs -e "await import('file:///C:/…/round-01.mjs')"`로 실행한다. 한국어는 파일 안에만 두고, 경로는 슬래시로 쓴다. `SyntaxError: await is only valid…`는 셸이 스크립트를 망가뜨린 것이다.

## 6. 개인정보와 업그레이드

- 브라우저 데이터는 로컬에 있지만, **연결한 에이전트(와 그 모델 회사)에는 페이지 내용과 작업 결과가 전달된다.**
- "Help improve ego(lite)" 옵션을 켜면 작업 내용·페이지 내용이 수집될 수 있다(Settings → Profile → Citro Labs services).
- 결과에 `[ego-browser:notice]`가 나오면 새 버전이 있다. `ego-browser upgrade`는 사람이 승인한 뒤에만 실행하고, 그 뒤 `ego-browser skill`을 다시 읽고 agy는 skills.json을 다시 만든다.

## 7. 문제 해결

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| Codex에서 `Unrecognized command '/ego-browser'` | Codex는 스킬을 `$`로 부른다 | `$ego-browser <작업>` |
| `ego-browser`를 찾을 수 없음 | 온보딩 전에 연 터미널 | 새 터미널에서 에이전트를 다시 띄운다 |
| `Unknown command: skill` | ego lite가 스킬보다 오래됨 | `ego-browser upgrade`, 안 되면 재설치 |
| agy에 `/ego-browser`가 없음 / 업그레이드 뒤 사라짐 | agy는 Junction을 따라가지 않음 / 옛 버전 경로 | 3절 skills.json 명령 실행 |
| 로그인했는데 "not logged in" | 다른 프로필의 Space | 프롬프트에 프로필을 명시 |
| 스냅샷에 원하는 요소가 없음 | 보이는 영역만 찍음, 캔버스 화면 | `{ scope: "full_page" }` 또는 스크린샷 + 좌표 |

## 관련 페이지

- [Claude 스킬 작성 규칙 7가지](../youtube/claude-skill-writing-rules.md) — ego-browser처럼 에이전트에 설치하는 스킬의 작성 규칙
- [Herdr 기본](herdr-basics.md) — claude·codex·agy를 한 화면에서 함께 돌리는 환경
