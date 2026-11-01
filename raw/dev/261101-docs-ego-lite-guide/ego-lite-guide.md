---
tags: [ego-lite, ego-browser, browser-automation, claude-code, codex, antigravity, windows, guide]
created: 2026-10-31
updated: 2026-10-31
---

# ego lite 사용 가이드 (Claude Code · Codex · agy)

Windows 11에 설치한 ego lite를 Claude Code, Codex CLI, Antigravity CLI(agy) 세 에이전트에서 브라우저로 쓰는 방법을 정리한다. 에이전트가 사용자의 로그인된 사이트에서 데이터를 읽고, 폼을 채우고, 웹앱을 점검하게 만드는 것이 목표다.

| 항목 | 내용 |
| --- | --- |
| 대상 | ego lite를 설치했고, 코딩 에이전트에서 브라우저 작업을 시키려는 사람 |
| 결과 | 세 에이전트 모두 `ego-browser` 스킬로 ego lite를 조작한다 |
| 기준 | Windows 11, ego lite 0.6.2.5 (Chromium 156, 내장 Node v24), 사용 가이드 v2.2.0, agy 1.4.11 — 2026-10-31 기준 |
| 시험 | 세 에이전트 모두 이 PC에서 스킬 호출 → example.com 제목 읽기 → Space 닫기까지 확인했다. 4.1~4.8의 사이트별 작업은 시험하지 않았다 |

> `<name>`, `<작업>`은 자리표시자다. 자기 환경의 값으로 바꾼다.
>
> ego lite는 빠르게 바뀌는 베타 제품이다. 웹 문서와 설치본의 API가 다를 때는 **설치본의 `ego-browser skill` 출력이 기준**이다(1.2절).

## 1. 소개

### 1.1 ego lite란

- Citro Labs(싱가포르)가 만든 **Chromium 기반 브라우저**다. 사람이 평소 쓰는 브라우저이면서, 동시에 AI 에이전트가 조작하도록 설계됐다. 무료다.
- **ego lite**는 브라우저 앱의 이름이고, **ego-browser**는 에이전트가 그 브라우저를 다루는 CLI와 스킬의 이름이다.
- 에이전트는 사용자가 이미 로그인한 사이트를 그대로 쓴다. 비밀번호를 프롬프트에 넣을 필요가 없다.
- Windows판은 베타다. 0.6.2.5(2026-10-29)에서 Windows 사용 가이드가 추가됐다.

공식 채널

| 항목 | 주소 |
| --- | --- |
| 사이트·다운로드 | https://lite.ego.app/ |
| 문서 | https://lite.ego.app/document/en/docs/quick-start (Windows: `/document/en/docs/windows/quick-start`) |
| GitHub (스킬·문서, MIT) | https://github.com/citrolabs/ego-lite — 브라우저 앱은 별도 배포 |
| 변경 이력 | https://lite.ego.app/changelog |

### 1.2 동작 구조

ego lite는 **MCP 서버가 아니다**. 스킬 + CLI 구조다.

```
 에이전트 (Claude Code / Codex / agy)
   │ ① 스킬(SKILL.md)을 읽는다
   │ ② ego-browser skill  → 설치 버전에 맞는 사용 가이드 출력
   │ ③ JS 스크립트를 쓴다 (TaskSpace·Page API)
   ▼
 ego-browser nodejs  ── 내장 Node.js가 스크립트 실행
   │ CDP
   ▼
 ego lite 브라우저
   ├─ 사용자 탭 (사람이 쓰는 그대로)
   └─ Space (에이전트 작업 공간, 작업마다 하나)
```

- 에이전트는 동작 하나마다 도구를 부르지 않는다. 여러 동작을 JS 스크립트 하나에 묶어 한 번에 실행한다. 그래서 MCP 방식보다 왕복 횟수와 토큰이 줄어든다(벤더 주장, 1.4절).
- 스킬 파일(SKILL.md)은 짧다. 실제 사용법은 `ego-browser skill`이 설치 버전에 맞춰 출력한다. 웹 문서에는 예전 API(`useOrCreateTaskSpace`, `snapshotText`, 초 단위 timeout)가 남아 있고, 설치본 v2.2.0은 `taskSpace`, `page.*`, 밀리초 단위다. 웹 예제 코드를 그대로 쓰지 않는다.

### 1.3 Space와 로그인 공유

- **Space**는 에이전트의 작업 공간이다. 에이전트의 탭은 Space 안에 열리고, 사용자의 탭·마우스·포커스는 건드리지 않는다. 여러 Space를 병렬로 돌릴 수 있다.
- **로그인은 공유된다.** 온보딩에서 Chrome·Edge 등의 데이터를 가져오거나 ego lite에서 직접 로그인하면, 에이전트도 그 로그인 상태로 작업한다.
- **쿠키는 프로필 단위로 공유된다고 보고 쓴다.** 웹 Space 문서는 "작업별 분리"라고 하지만, 설치본 문서(`references/clearing-state.md`)는 "cookie jar and HTTP cache are shared with the user's own tabs"라고 적고 있다. 관련 이슈(#319 Space 간 쿠키 누출, #303 Space에서 쿠키 삭제 시 메인 로그인도 삭제)도 열려 있다.
- 로그인·캡차·2FA·결제는 사람이 한다. 에이전트는 `handOff()`로 제어를 넘기고 기다린다(3.6절).

### 1.4 다른 도구와 비교

| 도구 | 연결 방식 | 로그인 재사용 | 특징 |
| --- | --- | --- | --- |
| **ego lite** | 스킬 + CLI, 전용 Chromium | 예 (프로필 공유) | 여러 CLI 에이전트 공용, Space로 사람과 분리 |
| Playwright MCP | MCP 서버 | 예 (`--extension` 모드) | 동작마다 tool call. 벤더 비교표의 "로그인 재사용 불가"는 틀렸다 ([Microsoft](https://github.com/microsoft/playwright/blob/main/packages/extension/README.md)) |
| Chrome DevTools MCP | MCP 서버 | 예 (`--autoConnect`) | 성능·네트워크 디버깅 중심 |
| Claude in Chrome | Chrome 확장 | 예 | Claude 전용, 연결된 Chrome 창 안에서 동작 |

- 벤더 벤치마크(31개 과제): ego 93.5% / 과제당 $1.75, Playwright MCP 71% / $4.82. **벤더 자체 측정**이다 ([비교](https://lite.ego.app/compare/ego-lite-vs-playwright-mcp)).
- 제3자 리뷰(macOS, 2026-09)에서는 토큰 절감은 확인됐지만 속도는 경우에 따라 달랐다 ([리뷰](https://www.heyuan110.com/posts/ai/2026-09-10-ego-lite-browser-review/)).

## 2. 설정

진행 순서: 2.1 설치 → 2.2 확인 → 쓰려는 에이전트만 2.3~2.5 → 2.6·2.7은 필요할 때.

### 2.1 설치와 온보딩

1. https://lite.ego.app/ 에서 Windows 설치 파일을 받아 실행한다. Windows용 설치 스크립트는 없다.
   - SmartScreen이 막으면 **추가 정보 → 실행**을 누른다.
2. ego lite를 열고 온보딩을 끝낸다.
   - 다른 브라우저 데이터 가져오기는 필요한 것만 고른다. 가져오기는 **한 번 복사**이고, 이후 Chrome과 동기화되지 않는다.
   - 온보딩이 하는 일: `ego-browser` 명령을 PATH에 등록하고, 각 에이전트의 스킬 폴더에 `ego-browser` 스킬 링크를 만든다.
3. 나중에 다른 브라우저 데이터를 더 가져오려면:
   ```powershell
   ego-browser import list
   ego-browser import --browser chrome --profile Default   # edge, brave 등
   ```

설치 후 생기는 것 (이 PC에서 확인)

| 항목 | 경로 |
| --- | --- |
| 앱 | `C:\Program Files\Citro Labs\ego lite\Application\ego.exe` |
| CLI | `...\Application\0.6.2.5\ego-browser.exe` |
| PATH 항목 | `%USERPROFILE%\.local\share\ego\active_version_dir` → 현재 버전 폴더를 가리키는 Junction |
| 스킬 원본 | `%USERPROFILE%\.local\share\ego\ego-skills` → `active_version_dir\ego-skills\ego-browser` (Junction) |
| 브라우저 데이터 | `%LOCALAPPDATA%\Citro Labs\ego lite\User Data` |

### 2.2 설치 확인

**새 터미널을 연다.** 온보딩 전에 열려 있던 터미널(그리고 그 안에서 실행 중인 에이전트)은 예전 PATH를 쓰므로 `ego-browser`를 찾지 못한다. 이 PC에서도 온보딩 전에 연 셸에서는 실패하고, 새로 연 셸에서는 바로 됐다.

PowerShell:

```powershell
Get-Command ego-browser                                    # cmd: where ego-browser / Git Bash: command -v ego-browser
ego-browser --version                                      # ego-browser 0.6.2.5, chromium ..., node ...
ego-browser nodejs -e "console.log('ego-browser ready')"   # ego-browser ready
ego-browser skill                                          # 사용 가이드 출력
```

- `ego-browser skill` 출력의 마지막 줄은 `[ego-browser:skill] end of guide v2.2.0`이다. 이 줄이 없으면 출력이 잘린 것이다.
- `Unknown command: skill`이 나오면 ego lite가 스킬보다 오래된 것이다. `ego-browser upgrade`를 실행한다(2.7절).

### 2.3 Claude Code

| 항목 | 내용 |
| --- | --- |
| 스킬 위치 | `~/.claude/skills/ego-browser` (온보딩이 만듦) |
| 호출 | `/ego-browser <작업>` — 또는 브라우저 작업을 요청하면 스킬 설명이 맞아 자동으로 쓴다 |
| 실행 셸 | Claude Code의 Bash 도구(Git Bash)에서 heredoc으로 실행한다 (3.3절) |

1. 스킬이 보이는지 확인한다. Claude Code를 새 터미널에서 다시 띄우고 `/ego`를 입력하면 `/ego-browser`가 자동완성 목록에 나와야 한다.
2. 권한을 정한다. ego 공식 문서는 `claude --dangerously-skip-permissions`(모든 승인 생략)를 안내하지만, 이 모드는 에이전트의 **모든** 셸 명령을 묻지 않고 실행한다. 대신 `ego-browser` 명령만 허용 목록에 넣는다:

   `~/.claude/settings.json`

   ```json
   {
     "permissions": {
       "allow": ["Bash(ego-browser:*)"]
     }
   }
   ```

   승인 화면에서 "don't ask again"을 골라도 같은 효과다. 허용하지 않으면 매 라운드마다 승인을 눌러야 해서 느리다(리뷰: 기본 권한 76s·12턴 → 허용 후 16.4s·2턴).

### 2.4 Codex

| 항목 | 내용 |
| --- | --- |
| 스킬 위치 | `~/.agents/skills/ego-browser` (온보딩이 만듦. Codex의 사용자 스킬 경로와 같다) |
| 호출 | `$ego-browser <작업>` — `$ego`까지 치면 `ego-browser [Skill] Automate logged-in web tasks`가 자동완성된다. `/skills`에서 골라도 된다. **`/ego-browser`는 안 된다**(Codex CLI는 `Unrecognized command '/ego-browser'`로 거부한다. 이 PC에서 확인) |
| 기본 프롬프트 | `Use $ego-browser to complete a browser automation task.` (스킬의 `agents/openai.yaml`) |

1. 새 터미널에서 `codex`를 띄우고 입력창에 `$ego`를 쳐서 `ego-browser [Skill]`이 자동완성되는지 본다. Codex에서 스킬은 `/`가 아니라 `$`로 부른다. ego 문서의 `/ego` 안내는 Codex 앱 기준으로 보인다.
2. 권한은 따로 정하지 않아도 된다. **온보딩이 `~/.codex/rules/default.rules`에 `prefix_rule(pattern=["ego-browser"], decision="allow")`를 넣어 둔다**(이 PC에서 확인, 온보딩 시각과 파일 시각이 같다). 그래서 `ego-browser`로 시작하는 명령은 승인 없이 실행된다.
   - 단, 온보딩 전에 띄운 Codex는 PATH에서 `ego-browser`를 못 찾아 `& 'C:\Program Files\…\ego-browser.exe' …`처럼 **절대경로로 실행**한다. 그러면 규칙과 맞지 않아 매번 승인 화면이 뜬다. 새 터미널에서 Codex를 다시 띄운다.
   - ego 문서는 `codex --dangerously-bypass-approvals-and-sandbox`도 안내하지만, OpenAI는 이 옵션을 **외부에서 격리된 환경에서만** 쓰라고 한다. 위 규칙이 있으면 필요 없다.

### 2.5 agy (Antigravity CLI)

**설치만 해서는 agy가 ego-browser 스킬을 인식하지 못한다.** 이 PC에서 확인한 원인과 해결법은 다음과 같다.

| 확인한 것 | 결과 |
| --- | --- |
| 설치 프로그램이 만든 링크 | `~/.gemini/antigravity-cli/skills/ego-browser` — agy가 **탐색하지 않는** 경로 |
| agy 전역 스킬 경로 | `~/.gemini/config/skills/` (agy 내장 `agy-customizations` 문서) |
| `config/skills/`에 Junction 추가 | ❌ 인식 안 됨 |
| `config/skills/`에 실제 폴더로 복사 | ✅ 인식됨 → **agy는 Junction을 따라가지 않는다** |
| `config/skills.json`에 링크 경로 등록 | ❌ 인식 안 됨 (`ego-skills`, `active_version_dir` 모두 Junction) |
| `config/skills.json`에 **실제 경로** 등록 | ✅ 인식됨, `/ego-browser` 호출과 실제 작업까지 성공 |

설정 방법 (PowerShell 7)

```powershell
# 현재 버전 폴더의 실제 경로를 찾아 skills.json 에 등록한다
$ver  = (Get-Item "$HOME\.local\share\ego\active_version_dir").Target
$path = (Join-Path $ver 'ego-skills') -replace '\\','/'
@{ entries = @(@{ path = $path }) } | ConvertTo-Json -Depth 3 |
  Set-Content -Encoding utf8 "$HOME\.gemini\config\skills.json"
Get-Content "$HOME\.gemini\config\skills.json"
```

결과 (이 PC):

```json
{
  "entries": [
    { "path": "C:/Program Files/Citro Labs/ego lite/Application/0.6.2.5/ego-skills" }
  ]
}
```

- `skills.json`이 이미 있고 다른 항목이 있으면 덮어쓰지 말고 `entries`에 한 줄을 추가한다.
- entries의 경로는 **스킬 폴더들을 담은 폴더**다. agy는 그 아래를 한 단계만 훑어 `ego-browser/SKILL.md`를 찾는다.
- 경로에 버전 번호가 들어가므로 **ego lite를 업그레이드한 뒤에는 위 명령을 다시 실행**한다.
- Windows PowerShell 5.1의 `-Encoding utf8`은 BOM을 붙인다. PowerShell 7에서 실행한다.

확인

1. agy를 다시 띄운다(실행 중인 세션은 스킬을 다시 읽지 않는다).
2. 입력창에 `/ego`를 쳐서 `/ego-browser  When you need a browser, ...`가 자동완성되는지 본다.
3. 기본 모드의 agy는 작업 한 번에 승인 창을 세 번 띄운다(이 PC에서 확인).

| 순서 | 승인 창 | 선택지와 주의 |
| --- | --- | --- |
| ① | `ego-browser skill` 실행 | 3번 "always allow … (Persist to settings.json)"을 고르면 저장된다 |
| ② | 스크립트 파일 생성 (`C:/Users/<name>/ego/…/round-01.mjs`) | 워크스페이스 밖이면 "always allow non-workspace access"가 있지만 **워크스페이스 밖 전체**를 여는 넓은 권한이다. 요청에 "스크립트는 `D:/projects/.ego/` 아래에 만들어"를 붙이면 워크스페이스 안에 만들지만, **그래도 파일 생성 승인은 매번 뜬다**(항상 허용 선택지 없음). `f`로 스크립트 전체를 보고 승인한다 |
| ③ | `ego-browser nodejs -e "await import('file:///…/round-01.mjs')"` 실행 | 3번을 고르면 **이번 파일 경로까지 포함한 명령 전체**가 저장돼 다음 작업에 맞지 않는다 |

- 승인 창의 `tab`(Amend)은 허용 범위를 고치는 기능이 아니다. **에이전트에게 추가 지시를 붙이는 입력칸**이다. 이 상태에서 `esc`를 누르면 작업 전체가 중단된다.
- 3번으로 저장한 규칙은 `~/.gemini/antigravity-cli/settings.json`에 이렇게 들어간다:

  ```json
  "permissions": {
    "allow": ["command(ego-browser skill)"]
  }
  ```

- ①·③을 한 번에 없애려면 `settings.json`에 **정규식 규칙**을 직접 넣고 agy를 다시 띄운다:

  ```json
  "permissions": {
    "allow": ["command(regex:ego-browser .*)"]
  }
  ```

  이 PC에서 시험한 결과:

  | 규칙 | `ego-browser skill` | `ego-browser nodejs …` |
  | --- | --- | --- |
  | `command(ego-browser skill)` (3번으로 저장) | 묻지 않음 | 물음 |
  | `command(ego-browser)` | **물음** | 물음 |
  | `command(regex:ego-browser .*)` | 묻지 않음 | 묻지 않음 ✅ |

  공식 문서([permissions](https://antigravity.google/docs/permissions))는 `command(git)`처럼 쓰면 "단어 단위 앞부분 일치"라고 하지만, agy 1.4.11에서 `command(ego-browser)`는 맞지 않았다(로그에는 규칙이 `Allow:[command(ego-browser)]`로 읽힌 것까지 확인). 정규식 형식을 쓴다.
- agy가 실행 중일 때 이 파일을 고치면 반영되지 않는다. 고친 뒤 agy를 다시 띄운다.
- 규칙을 넣어도 ② 파일 생성 승인은 남는다. 작업 한 번에 승인 한 번이다.
- ②까지 없애려면 같은 `settings.json` 최상위에 `"allowNonWorkspaceAccess": true`를 넣는다(승인 창의 "always allow non-workspace access"를 고른 것과 같다). 정규식 규칙과 함께 쓰면 **승인 창이 하나도 뜨지 않는다**(이 PC에서 naver.com 접속 시험으로 확인).

  ```json
  {
    "allowNonWorkspaceAccess": true,
    "permissions": {
      "allow": ["command(regex:ego-browser .*)"]
    }
  }
  ```

  > **위험**: 이 설정은 ego 스크립트만이 아니라 **워크스페이스 밖의 모든 파일**(홈 폴더, `.ssh`, 다른 프로젝트, 각종 설정 파일)을 agy가 묻지 않고 읽고 쓰게 한다. 게다가 ego-browser는 로그인된 브라우저를 다루므로, 방문한 페이지의 프롬프트 인젝션이 agy를 움직이면 파일 접근과 로그인 세션이 한꺼번에 노출된다(5.2절).
  >
  > - 켤 때: 혼자 쓰는 PC에서, 믿을 수 있는 사이트만 다루고, 작업 중 화면을 지켜볼 때.
  > - 켜지 않을 때: 처음 보는 사이트를 돌아다니는 작업, 민감한 계정이 로그인된 프로필, 무인 반복 작업.
  > - 대안: 끄고 두고, 파일 생성 승인 창에서 `f`로 스크립트를 확인한 뒤 승인한다. 작업당 한 번이다.
- `shift+tab`으로 바꾸는 auto 모드에서는 셸 명령 승인(①·③)이 뜨지 않는다. 대신 ego 외의 **모든** 명령도 묻지 않는다. 워크스페이스 밖 파일 접근(②)은 auto 모드에서도 묻는다.

### 2.6 개인정보·텔레메트리

- 브라우저 데이터는 로컬에 저장된다. 그러나 **연결한 에이전트(그리고 그 모델 회사)에는 페이지 내용과 작업 결과가 전달된다.**
- 벤더 정책상 저장된 비밀번호·인증 쿠키·세션 토큰은 브라우저 인터페이스로 읽거나 내보낼 수 없다. 독립 검증은 아니다.
- "Help improve ego(lite)'s features and performance"를 켜면 작업 내용·실행 기록·페이지 내용이 수집될 수 있다. `Settings → Profile → Citro Labs services`에서 확인한다 ([개인정보처리방침](https://lite.ego.app/privacy)).

### 2.7 업그레이드

- 작업 결과에 `[ego-browser:notice]`가 나오면 새 버전이 있다는 뜻이다. 에이전트는 사용자 승인 후에만 `ego-browser upgrade`를 실행한다.
- 업그레이드 뒤에는 `ego-browser skill`을 다시 읽고, **agy를 쓴다면 2.5절의 skills.json 명령을 다시 실행**한다.

## 3. 사용법

### 3.1 요청 쓰는 법

에이전트에게는 자연어로 맡긴다. 스크립트는 에이전트가 쓴다. 좋은 요청에는 네 가지가 들어간다.

| 요소 | 예 |
| --- | --- |
| 목표 | "지난달 인보이스 합계를 알고 싶다" |
| 사이트·범위 | "Stripe 대시보드, 2026-09만" |
| 출력 형식 | "표로, 금액·날짜·링크" |
| **멈출 조건** | "제출·결제·삭제 버튼은 누르지 마", "로그인이 필요하면 멈추고 알려줘" |

```
/ego-browser <사이트>에서 <대상>을 찾아 <항목>을 표로 정리해줘. <행동>은 하지 말고, 로그인이 필요하면 멈춰.
```

- 프롬프트의 "누르지 마"는 약속이지 강제 장치가 아니다. 되돌릴 수 없는 작업은 사람이 직접 한다.
- 비밀번호·인증 코드는 프롬프트에 쓰지 않는다. 브라우저에서 직접 넣는다.

### 3.2 에이전트가 하는 일: TaskSpace와 Page

에이전트가 쓰는 스크립트의 기본 형태다. 직접 쓸 일은 드물지만, 에이전트의 진행을 읽을 때 알아 두면 좋다.

```js
const task = await taskSpace("check example page"); // 작업 하나에 Space 하나. 같은 이름이면 이어서 쓴다
const page = task.page("p1");                        // 새 Space는 p1 페이지로 시작한다
await page.goto("https://example.com");
console.log(await page.snapshot());                 // 접근성 트리 — 다음 동작을 고르는 근거
await task.finish({ keep: [] });                    // 끝나면 한 번 호출. 에이전트 탭을 정리한다
```

- 실행할 때마다 새 Node 프로세스다. Space와 탭 라벨(`p1`, `p2`…)은 남고, JS 변수는 남지 않는다.
- 다음 라운드는 `taskSpace(<id>)`로 같은 Space를 이어 쓴다. 막혔다고 새 Space를 만들지 않는다.
- `finish({ keep: ["p2"] })`처럼 결과 탭을 사용자에게 남길 수도 있다. 기본은 모두 닫는다.
- Playwright가 아니다. `locator()`, `getByRole()` 같은 메서드는 없다.

### 3.3 셸별 실행 형식

같은 스크립트라도 셸에 따라 넘기는 방법이 다르다. 에이전트는 `ego-browser skill` 출력을 보고 맞는 방법을 고른다.

| 셸 | 방법 | 쓰는 에이전트 |
| --- | --- | --- |
| Git Bash | heredoc: `ego-browser nodejs <<'EOF' … EOF` | Claude Code (Bash 도구) |
| PowerShell | UTF-8 `.mjs` 파일을 만들고 `ego-browser nodejs -e "await import('file:///C:/Users/<name>/ego/task.mjs')"` | Codex, agy |
| cmd | `ego-browser nodejs < C:\Users\<name>\ego\task.mjs` | — |

PowerShell에서 지킬 것

- 여러 줄이나 큰따옴표가 든 스크립트를 `-e`에 직접 넣지 않는다. PowerShell 5.1이 큰따옴표를 지워 스크립트가 깨진다.
- 한국어 등 비ASCII 텍스트는 `.mjs` 파일 안에만 둔다.
- `Get-Content`로 파이프하지 않는다(5.1에서 인코딩이 깨진다).
- 라운드마다 새 파일(`round-01.mjs`, `round-02.mjs`…)을 쓴다. agy는 이 PC에서 `C:\Users\<name>\ego\<작업이름>-<날짜시각>\round-01.mjs`를 스스로 만들어 썼다.
- 스크립트 안의 Windows 경로는 슬래시로 쓴다: `C:/Users/<name>/ego-shots/a.png`.

`SyntaxError: await is only valid in async functions…`가 나오면 셸이 스크립트를 망가뜨린 것이다. `.mjs` 파일 방식으로 바꾼다.

### 3.4 스냅샷·셀렉터·폼 입력

- **스냅샷 먼저.** `page.snapshot()`은 화면에 보이는 영역의 접근성 트리를 준다. 전체 페이지는 `{ scope: "full_page" }`.
- 셀렉터는 스냅샷 ref(`@21`), `text=저장`, `loc=role:button[name='Sign in']`, `loc=css:#id`, `xpath=…`, CSS를 받는다. **정확히 하나**에 맞아야 한다.
- 입력: `fill()`, `selectOption()`, `press()`, `setInputFiles()`(절대경로).
- 결과는 고정 대기 대신 조건으로 기다린다: `waitForURL()`, `waitForSelector()`, `waitForFunction()`.
- 캔버스·스프레드시트·지도처럼 DOM이 빈약한 화면은 스크린샷 + `mouse`·`keyboard`로 다룬다.

### 3.5 스크린샷·데이터 추출·다운로드

| 할 일 | API | 주의 |
| --- | --- | --- |
| 스크린샷 | `page.screenshot({ path: "C:/…/a.png" })` | 절대경로. 생략하면 임시 폴더 |
| 대량 추출 | `page.evaluate(fn, arg)` | 페이지 안에서 실행, JSON으로 돌려받는다 |
| 페이지 쿠키로 API 호출 | `page.fetch(url, options)` | 그 페이지의 로그인·CORS를 따른다 |
| 다운로드 | `page.waitForEvent("download")` → 클릭 → `download.saveAs(절대경로)` | 같은 라운드 안에서 저장해야 한다 |

### 3.6 사람에게 넘기기

- 로그인·캡차·2FA·권한 팝업·기기 선택창이 나오면 에이전트는 `task.handOff()`를 부르고 멈춘다.
- 사람이 ego lite의 해당 Space에서 처리한 뒤 에이전트에게 "계속해"라고 말한다. 에이전트는 `takeOverTaskSpace(<id>)`로 같은 Space를 다시 잡는다.
- 사람이 Space를 직접 조작하기 시작하면 에이전트는 멈추고 우회하지 않는다.

### 3.7 에이전트별 차이

| | Claude Code | Codex | agy |
| --- | --- | --- | --- |
| 스킬 경로 | `~/.claude/skills` | `~/.agents/skills` | `~/.gemini/config/skills.json` (2.5절) |
| 설치 직후 바로 동작 | 예 | 예 | **아니요** — skills.json 필요 |
| 호출 | `/ego-browser <작업>` | `$ego-browser <작업>` (`/`는 안 됨) | `/ego-browser <작업>` |
| 실행 셸 | Git Bash (heredoc) | PowerShell (`.mjs`) | PowerShell (`.mjs`) |
| 승인 줄이기 | `Bash(ego-browser:*)` 허용 (직접 추가) | 온보딩이 `default.rules`에 자동 추가 | `settings.json`의 `permissions.allow`에 `command(regex:ego-browser .*)` 직접 추가. 파일 생성 승인은 남음 (`allowNonWorkspaceAccess`로 없앨 수 있으나 위험, 2.5절) |
| 업그레이드 후 할 일 | 없음 | 없음 | skills.json 다시 생성 |

## 4. 예시

각 예시는 **목적 · 에이전트 · 프롬프트 · 진행 흐름 · 결과와 주의점** 순서다. 출처 표시: **[시험함]** 이 PC에서 실행, **[공식]** 벤더의 실연·게시 예시, **[응용]** 스킬이 밝힌 용도를 바탕으로 만든 예시.

### 4.0 첫 동작 확인 — 세 에이전트 [시험함]

같은 요청을 세 에이전트에 보내 모두 `Example Domain`을 받았다. 호출만 다르다: Claude Code·agy는 `/ego-browser …`, Codex는 `$ego-browser …`. 아래는 agy의 진행이다.

- **목적**: 설정이 끝났는지 가장 가벼운 작업으로 확인한다.
- **프롬프트**
  ```
  /ego-browser https://example.com 을 열어 페이지 제목만 알려줘. 다른 사이트로 이동하지 말고 끝나면 Space를 닫아.
  ```
- **진행 흐름** (실제 관찰)
  1. `ego-browser skill` 실행 → 승인
  2. `C:\Users\<name>\ego\example-title-<날짜시각>\round-01.mjs` 작성
  3. `ego-browser nodejs -e "await import('file:///…/round-01.mjs')"` 실행 → 승인
  4. 답: `Example Domain`
- **주의**: 승인 화면이 두 번 뜬다. 2.5절처럼 항상 허용을 고르면 다음부터 바로 진행한다.

### 4.1 로그인된 SNS 게시물 추출 — Claude Code [공식]

- **목적**: 로그인이 필요한 사이트에서 데이터를 표로 뽑는다.
- **프롬프트**
  ```
  /ego-browser x.com에서 @AnthropicAI 프로필을 열고 보이는 게시물 5개의 본문 요약, 링크, 좋아요 수를 표로 정리해줘. 좋아요·리트윗·답글은 하지 마.
  ```
- **진행 흐름**: Space 생성 → 프로필 이동 → 스냅샷으로 게시물 확인 → `evaluate()`로 5개 추출 → `finish()`
- **결과와 주의**: X에 로그인돼 있지 않으면 `handOff()`로 멈춘다. ego lite에서 로그인한 뒤 "계속해"라고 한다. Chrome과 Edge를 둘 다 가져왔다면 로그인이 없는 다른 프로필을 쓸 수 있다(5.1절). 출처: [claude-code 문서](https://lite.ego.app/document/en/docs/claude-code)

### 4.2 커뮤니티 인기 글 리서치 — Codex [공식]

- **목적**: 커뮤니티의 최근 흐름을 빠르게 요약한다.
- **프롬프트**
  ```
  $ego-browser Reddit r/LocalLLaMA에서 지난 1주일 Top 글 5개를 찾아 제목·추천수·댓글 수·링크·한 줄 요약을 표로 정리해줘. 보이지 않는 값은 "없음"으로 쓰고, 투표·댓글은 하지 마.
  ```
- **진행 흐름**: Top/Week 정렬 URL로 이동 → 스냅샷 → 상위 5개 추출 → 필요하면 각 글을 같은 탭에서 열어 요약 → `finish()`
- **결과와 주의**: Codex는 PowerShell에서 `.mjs` 파일 방식으로 실행한다. 출처: [Windows Codex 실연](https://lite.ego.app/document/en/docs/windows/codex)

### 4.3 여러 사이트 비교 요약 — agy [공식]

- **목적**: 여러 페이지의 값을 모아 한 표로 비교한다.
- **프롬프트**
  ```
  /ego-browser Yahoo Finance에서 AAPL, MSFT, NVDA, GOOGL, AMZN의 현재가·일간 변동·시가총액을 모아 표로 정리하고, 조회 시각과 출처 링크를 적어줘. 로그인·주문은 하지 마.
  ```
- **진행 흐름**: 한 탭(`p1`)을 `goto()`로 재사용하며 종목 페이지를 차례로 연다 → 각 페이지에서 `evaluate()`로 값 추출 → 표 작성 → `finish()`
- **결과와 주의**: 시세는 조회 시각이 중요하다. 표에 시각을 같이 남기게 한다. 다른 예: "OpenAI와 Anthropic 블로그의 최근 글을 비교해 날짜·핵심·링크로 정리" ([quick-start](https://lite.ego.app/document/en/docs/quick-start)). 출처: [use-cases](https://lite.ego.app/use-cases)

### 4.4 지원서 폼 채우기, 제출 직전에 멈춤 — Claude Code [공식]

- **목적**: 긴 폼 입력을 맡기고, 최종 제출은 사람이 한다.
- **프롬프트**
  ```
  /ego-browser 지금 열려 있는 채용 공고의 지원 폼을 D:/docs/resume.pdf 내용으로 채우고 이력서 파일도 첨부해. 제출 버튼은 절대 누르지 말고, 다 채운 화면을 스크린샷으로 C:/Users/<name>/ego-shots/apply.png 에 저장한 뒤 결과 탭을 남겨줘.
  ```
- **진행 흐름**: `task.tabs()`로 사용자가 연 탭을 찾아 `adopt()` → 스냅샷 → `fill()`·`selectOption()`·`setInputFiles()` → 스크린샷 → `finish({ keep: ["p1"] })`
- **결과와 주의**: 결과 탭이 남아 사람이 내용을 확인하고 직접 제출한다. 항공권 승객 정보 입력에서 "결제 단계에서 멈춤"도 같은 형태다. 출처: [use-cases](https://lite.ego.app/use-cases)

### 4.5 로그인된 주문 페이지에서 CSV 받기 — Codex [공식]

- **목적**: 관리 화면에서 필터를 걸고 파일을 내려받는다.
- **프롬프트**
  ```
  $ego-browser 쇼핑몰 관리자 페이지에서 어제 주문만 필터링해 CSV를 내려받아 D:/exports/orders-2026-10-30.csv 로 저장하고 행 수를 알려줘. 내보내기 확인 창 외의 설정은 바꾸지 마.
  ```
- **진행 흐름**: 주문 목록 이동 → 날짜 필터 → `waitForEvent("download")`를 걸고 내보내기 클릭 → `saveAs()` → Node `fs`로 행 수 계산 → `finish()`
- **결과와 주의**: 다운로드는 같은 라운드 안에서 `saveAs()`해야 한다. 인보이스 PDF 일괄 수집(Stripe 등)도 같은 형태다. 출처: [snapshot 문서 예시](https://lite.ego.app/document/en/docs/snapshot)

### 4.6 로컬 웹앱 QA와 스크린샷 — Claude Code [응용]

- **목적**: 개발 중인 앱을 로그인된 상태에서 클릭해 보며 점검한다.
- **프롬프트**
  ```
  /ego-browser http://localhost:3000 에 로그인된 상태로 /settings 에서 프로필 이름을 "QA Test"로 바꾸고 저장해. 성공 토스트가 뜨는지 확인하고, 저장 전후 스크린샷을 C:/Users/<name>/ego-shots/ 에 저장해. 이상한 점이 있으면 재현 단계를 적어줘.
  ```
- **진행 흐름**: 이동 → 스크린샷(전) → `fill()`·`click()` → `waitForSelector()`로 토스트 대기 → 스크린샷(후) → `finish()`
- **결과와 주의**: 코드 수정과 브라우저 확인을 한 세션에서 오간다. 브라우저 콘솔 로그를 보려면 `page.cdp("egoAutomated.disable")`이 필요하고, 이때는 사이트가 자동화를 감지할 수 있으니 로컬 앱에서만 쓴다.

### 4.7 버그 재현 기록 — Codex [응용]

- **목적**: 보고된 버그를 재현하고 이슈에 붙일 기록을 만든다.
- **프롬프트**
  ```
  $ego-browser staging 사이트에서 검색 → 필터 "가격 낮은 순" → 첫 상품 상세 순서로 진행해 "상세 페이지 가격이 목록과 다르다"는 버그를 재현해줘. 재현 단계, 기대 결과, 실제 결과, 각 단계 스크린샷 경로를 Markdown으로 정리해. 장바구니·구매는 하지 마.
  ```
- **진행 흐름**: 단계마다 행동 → 스냅샷/스크린샷 → 값 비교 → 기록 작성
- **결과와 주의**: 재현이 안 되면 "재현 안 됨"과 시도한 조건을 그대로 적게 한다. 공식 실연은 아직 없고, 스킬 설명의 "bug investigation" 용도를 바탕으로 한 예시다.

### 4.8 반복 모니터링 — agy [응용]

- **목적**: 정해진 페이지의 변화를 주기적으로 확인한다.
- **프롬프트**
  ```
  /ego-browser https://github.com/citrolabs/ego-lite/issues 에서 열린 이슈 중 Windows 관련 이슈의 번호·제목·생성일을 읽고, D:/notes/ego-issues.md 에 있는 이전 목록과 비교해 새로 생긴 것만 알려줘. 이슈에 댓글이나 반응은 남기지 마.
  ```
- **진행 흐름**: 목록 추출 → 이전 기록과 비교 → 변경점 보고 → 기록 파일 갱신
- **결과와 주의**: ego lite 자체에는 예약 실행 기능이 없다. 반복은 에이전트 쪽 기능(예: Claude Code의 `/loop`)이나 사람이 같은 프롬프트를 다시 보내는 방식으로 한다. 공식 문서는 "monitor mentions"를 용도로 들지만 예약 실행 실연은 없다.

## 5. 문제 해결과 주의점

### 5.1 문제 해결

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| Codex에서 `Unrecognized command '/ego-browser'` | Codex CLI는 스킬을 `$`로 부른다 | `$ego-browser <작업>` (2.4절) |
| `ego-browser`를 찾을 수 없음 | 온보딩 전에 연 터미널 | 새 터미널을 연다. 에이전트도 새 터미널에서 다시 띄운다. 그래도 안 되면 ego lite를 다시 열어 온보딩을 끝낸다 |
| `Unknown command: skill` | ego lite가 스킬보다 오래됨 | `ego-browser upgrade`. `upgrade`도 없으면 재설치 |
| agy에서 `/ego-browser`가 없음 | agy는 설치 프로그램이 만든 경로를 보지 않고, Junction도 따라가지 않음 | 2.5절의 skills.json 설정 |
| 업그레이드 후 agy에서 스킬이 사라짐 | skills.json에 예전 버전 경로 | 2.5절 명령을 다시 실행 |
| 로그인했는데 "not logged in" | 다른 프로필(Chrome/Edge)의 Space | 프롬프트에 사용할 프로필을 명시 |
| `SyntaxError: await is only valid…` | PowerShell이 스크립트를 망가뜨림 | `.mjs` 파일 방식(3.3절) |
| 스냅샷에 원하는 요소가 없음 | 보이는 영역만 찍음, 또는 캔버스 화면 | `{ scope: "full_page" }` 또는 스크린샷 + 좌표 |
| SmartScreen이 설치를 막음 | 서명 평판 | 추가 정보 → 실행 |
| 업데이트가 막힘 | 일부 보안 SW (이슈 #592) | 보안 SW 예외 처리 |

### 5.2 보안

- **권한 우회는 기본값으로 쓰지 않는다.** 공식 문서의 `--dangerously-…` 옵션은 브라우저 명령뿐 아니라 에이전트의 모든 명령을 승인 없이 실행한다. `ego-browser` 명령만 허용한다(2.3~2.5절). agy의 auto 모드와 `allowNonWorkspaceAccess`도 같은 이유로 범위가 넓다.
- **프롬프트 인젝션.** 방문한 페이지에 숨은 지시문이 로그인된 세션에 닿을 수 있다. 벤더도 면역이 아니라고 밝혔다. 민감한 계정(은행, 업무 관리자)과 처음 보는 사이트를 한 작업에 섞지 않는다.
- **쿠키 정리는 범위를 확인한다.** `Network.clearBrowserCookies`, `Network.clearBrowserCache`, `Storage.clearCookies`는 **프로필 전체**를 지우고 되돌릴 수 없다. 사이트 하나만 지울 때는 `Storage.clearDataForOrigin`을 쓴다(설치본 `references/clearing-state.md`). 에이전트가 이 명령을 쓰려 하면 승인하기 전에 확인한다.
- **스킬 출처.** 스킬은 에이전트에게 명령 실행을 지시하는 파일이다. 공식 설치본이나 `npx skills add citrolabs/ego-lite`처럼 믿을 수 있는 출처만 쓴다.
- **열린 이슈** (2026-10-31 기준): #315 스크립트의 자격 증명 접근, #319 Space 간 쿠키 누출, #303 Space에서 쿠키 삭제 시 메인 로그인도 삭제, #594 Windows에서 TaskSpace 목록 누락. https://github.com/citrolabs/ego-lite/issues

## 부록: 출처

| 내용 | 출처 |
| --- | --- |
| 제품·다운로드 | https://lite.ego.app/ |
| Windows 시작 | https://lite.ego.app/document/en/docs/windows/quick-start |
| Claude Code 연결 | https://lite.ego.app/document/en/docs/claude-code |
| Codex 연결 | https://lite.ego.app/document/en/docs/codex , https://lite.ego.app/document/en/docs/windows/codex |
| Space | https://lite.ego.app/document/en/docs/space |
| 스냅샷 | https://lite.ego.app/document/en/docs/snapshot |
| 사례 | https://lite.ego.app/use-cases |
| 비교 | https://lite.ego.app/compare/ego-lite-vs-playwright-mcp , https://lite.ego.app/compare/ego-lite-vs-claude-chrome-extension |
| 개인정보 | https://lite.ego.app/privacy |
| 스킬·설치 레퍼런스 | https://github.com/citrolabs/ego-lite (`skills/ego-browser/`) |
| Codex 스킬 | https://learn.chatgpt.com/docs/build-skills |
| agy 스킬 경로 | agy 내장 `~/.gemini/antigravity-cli/builtin/skills/agy-customizations/` (SKILL.md, docs/json_configs.md) |
| 제3자 리뷰 | https://www.heyuan110.com/posts/ai/2026-09-10-ego-lite-browser-review/ |
| 사용 API | 설치본 `ego-browser skill` 출력 v2.2.0 (이 PC) |
