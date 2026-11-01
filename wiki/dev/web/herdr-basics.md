---
title: Herdr 기본 — 구조, 상태, 명령
sources:
  - raw/web/261101-herdr/README.md
  - raw/web/261101-herdr/BUILDING.md
  - raw/web/261101-herdr/OPERATING.md
  - raw/web/261101-herdr/herdr-multi-agent-guide.md
  - https://code.claude.com/docs/en/interactive-mode (2026-11-01)
  - https://wmedia.es/en/tips/claude-code-how-to-quit-session (2026-11-01)
  - https://www.octavehq.com/post/claude-code-cli-cheatsheet-essential-commands (2026-11-01)
  - https://herdr.dev/llms-full.txt (2026-11-01)
  - https://docs.cloudbees.com/docs/cloudbees-cd-kb/latest/cloudbees-cd-kb/kbec-00420-stopping-path-conversion-for-git-bash (2026-11-01)
  - specs/v0.1/PROGRESS.md
updated: 2026-11-01
tags: [herdr, multi-agent, terminal, claude, codex, agy]
status: draft
---

# Herdr 기본 — 구조, 상태, 명령

Herdr는 터미널 멀티플렉서다. 한 화면에서 Claude·Codex·agy 같은 AI 에이전트 여러 개를 띄우고, 에이전트 하나(leader)가 `herdr` 명령으로 다른 에이전트에게 일을 나눠 맡긴다. 역할을 나누고 구성하는 방법은 [Herdr 멀티 에이전트 — 역할과 구성](herdr-multi-agent-roles.md)에 있다.

원본의 기준 환경: Herdr 0.9.5, Windows 11, 터미널 130×37.

## 1. 구조

```text
Herdr 세션
└─ 워크스페이스 (w8)          보통 프로젝트 하나
   ├─ 탭 (w8:t1)              화면 하나
   │  ├─ 패널 (w8:p1)         터미널 하나 — 셸, 에이전트, 테스트 등
   │  └─ 패널 (w8:p2)
   └─ 탭 (w8:t2)
```

- ID는 Herdr가 정하고, 닫힌 탭·패널의 ID는 다시 쓰이지 않는다. 그래서 다음에 만든 패널의 ID를 짐작하면 안 된다. 명령의 응답에서 읽는다.
- 패널 ID보다 **에이전트 이름**으로 부르는 것이 안전하다. 이름은 에이전트를 따라간다.
- 에이전트 이름은 **세션 전체에서 유일**해야 한다. 소문자로 시작, 소문자·숫자·`-`·`_`만, 32자 이하. 워크스페이스가 여럿이면 `<프로젝트>-<역할>`처럼 접두어를 붙인다(예: `visionlab-coder`).

## 2. 에이전트 상태

| 상태 | 뜻 | leader가 할 일 |
| --- | --- | --- |
| `idle` | 입력을 기다림 | 다음 지시를 보낼 수 있다 |
| `working` | 일하는 중 | 기다린다 |
| `blocked` | 승인·질문 화면에서 멈춤 | 화면을 읽고 대리 응답하거나 사람에게 올린다 |
| `done` | 일을 끝냄 | 결과를 읽는다 |
| `unknown` | 상태를 판단하지 못함 | 화면을 직접 읽는다. 끝났다는 뜻이 아니다 |

## 3. 에이전트 패널과 셸 패널

- **에이전트 패널**: Claude·Codex·agy가 떠 있는 패널. `herdr agent …`로 다룬다.
- **셸 패널(runner)**: 테스트·서버·학습을 돌리는 일반 터미널. `herdr pane …`로 다룬다. 토큰을 쓰지 않는다.

오래 걸리는 실행은 runner에서 돌린다. 에이전트가 출력을 지켜보면 그동안 토큰이 계속 든다. 짧은 명령(`git status`, 테스트 1회)은 에이전트가 직접 실행한다.

runner 규칙:

- runner 하나는 주인이 하나다. 두 에이전트가 같은 셸에 입력하면 명령이 섞인다.
- 서버와 테스트처럼 용도가 다르면 runner를 나눈다(`runner-server`, `runner-test`).
- 보내기 전에 `pane read --source visible`로 셸 프롬프트가 보이는지 확인한다.
- `pane run` 성공은 명령이 **입력됐다**는 뜻이다. 결과는 `pane wait-output`이나 `pane read`로 확인한다.
- 위험한 명령(삭제·외부 전송·설치)을 runner로 돌려 승인 절차를 피하지 않는다.

## 4. 명령 빠른 참조

| 하고 싶은 것 | 명령 |
| --- | --- |
| 현황 보기 | `herdr workspace list` · `herdr tab list --workspace <ID>` · `herdr agent list` |
| 내 위치 | `herdr pane current --current` |
| 탭 만들기 | `herdr tab create --workspace <ID> --label <이름> --cwd <경로> --no-focus` |
| 패널 나누기 | `herdr pane split <패널 ID> --direction right\|down --cwd <경로> --no-focus` |
| 배치 확인 | `herdr pane layout --pane <패널 ID>` |
| 에이전트 띄우기 | `herdr agent start <이름> --kind claude\|codex\|agy --pane <패널 ID> -- <모델 인자>` |
| 이름 붙이기 | `herdr agent rename <패널 ID> <이름>` |
| 지시하고 기다리기 | `herdr agent prompt <이름> "<지시>" --wait --timeout <ms>` |
| 결과 읽기 | `herdr agent read <이름> --source recent-unwrapped --lines 120` |
| 키 보내기 | `herdr agent send-keys <이름> esc` |
| 셸 명령 실행·대기·읽기 | `herdr pane run <ID> "<명령>"` · `herdr pane wait-output <ID> --match "<문자열>" --timeout <ms>` · `herdr pane read <ID> --source recent-unwrapped --lines 50` |
| 작업 폴더 나누기 | `herdr worktree create --workspace <ID> --branch <이름> --no-focus` |

- `agent start`는 이미 있는 **빈 셸 패널**에만 쓸 수 있다. 패널을 만들어 주지 않는다.
- 패널·탭을 만들 때는 항상 `--no-focus`를 붙인다. 빠뜨리면 사람이 보던 화면이 바뀐다.

모델 인자(`--` 뒤):

| 에이전트 | 예시 | 확인 방법 |
| --- | --- | --- |
| claude | `--model opus --effort medium` · `--model sonnet --effort high` | `claude --help` |
| codex | `-m gpt-6.1-sol -c model_reasoning_effort="medium"` | `codex --help`, `~/.codex/models_cache.json` |
| agy | `--model gemini-3.8-flash-low` (`-medium`·`-high`로 effort) | `agy models` |

## 5. 자주 겪는 문제

| 증상 | 원인 | 대응 |
| --- | --- | --- |
| `agent prompt`가 timeout·`agent_prompt_stalled` | 지시는 갔을 수 있다 | 같은 지시를 다시 보내지 말고 먼저 `agent read`로 확인 |
| `agent_blocked` 오류 | 승인 화면이 떠 있음 | 화면을 읽고 대리 응답하거나 사람에게 올린다 |
| `agent start` 실패 | 패널이 빈 셸 프롬프트가 아님 | 패널 화면을 읽어 무엇이 돌고 있는지 확인 |
| 슬래시 명령(`/clear`, `/new`)이 엉뚱하게 들어감 | Git Bash가 `/clear`를 `C:/Program Files/Git/clear` 같은 경로로 바꿈 | `MSYS_NO_PATHCONV=1`을 붙여 보낸다 |
| claude 결과가 `agent read`로 앞부분까지 안 읽힘 | 결과가 길다 | 결과를 임시 Markdown 파일로 저장하게 하고 경로만 답하게 한다 |
| agy 업데이트 실패 | 실행 중인 agy가 파일을 잠금 | agy 패널을 끝낸 뒤 `agy update` |

## 6. 에이전트 재시작

멈춘 에이전트나 새 스킬·설정을 읽혀야 하는 에이전트는 같은 패널·같은 이름·같은 설정으로 다시 띄운다. 이것은 승인된 구성 안의 일이고, 대리 응답 기록에 "구성 안 조치"로 남긴다([역할과 구성](herdr-multi-agent-roles.md) 6·7절).

```bash
# 1) 종료 — Ctrl+C 두 번이 가장 확실하다
herdr agent send-keys <이름> ctrl+c ctrl+c
#    또는 /exit. Git Bash에서는 경로 변환을 꺼야 한다
MSYS_NO_PATHCONV=1 herdr agent prompt <이름> "/exit"

# 2) 셸 프롬프트로 돌아왔는지 확인 (agent start는 빈 셸 패널에만 된다)
herdr pane read <패널 ID> --source visible

# 3) 같은 패널·이름·설정으로 다시 시작
herdr agent start <이름> --kind claude --pane <패널 ID> --timeout 60000 -- --model sonnet --effort high
```

### claude 종료 키

| 방법 | 동작 |
| --- | --- |
| `Ctrl+C` | 실행 중인 작업이 있으면 그 작업만 멈춘다. 없으면 첫 번째는 입력칸을 비우고, 두 번째에 종료한다 |
| `Ctrl+D` | 첫 번째는 확인 안내, 800ms 안에 두 번째를 누르면 종료한다. 입력칸에 글이 있으면 커서 뒤 글자를 지운다 |
| `/exit` | 세션을 끝내고 셸로 돌아간다. 대화는 남아 `claude --resume`으로 다시 열 수 있다 |

### Git Bash의 경로 변환

Git Bash(MSYS2)는 `/`로 시작하는 인자를 Windows 경로로 바꾼다. 그래서 `herdr agent prompt <이름> "/exit"`는 `/exit`가 아니라 `C:/Program Files/Git/exit` 같은 글로 들어갈 수 있다. 명령 앞에 `MSYS_NO_PATHCONV=1`을 붙이면 그 명령에서만 변환이 꺼진다. `//exit`처럼 슬래시를 두 번 쓰는 방법도 있지만 항상 통하지는 않는다.

### agent start 참고

- 기존 셸 패널이 있어야 하고, 패널을 만들거나 나누지 않는다.
- 에이전트가 입력 준비가 됐다고 감지되면 성공으로 돌아온다. 기본 대기는 30초이고, `--timeout`은 3000ms 초과 300000ms 이하로 준다.
- `--` 뒤의 인자는 그대로 실행 파일에 넘어간다.

### 겪은 일 (2026-11-01, wiki-docs 워크스페이스)

- `wd-scouter-claude`를 재시작하려고 `herdr agent prompt … "/exit"`와 `herdr pane run … "/exit"`를 `MSYS_NO_PATHCONV=1` 없이 보냈는데, 두 번 다 종료되지 않았다. 위 경로 변환이 원인으로 보이지만 다시 해 보지는 않았다.
- `herdr agent send-keys wd-scouter-claude ctrl+c ctrl+c`로 종료됐고, 셸 프롬프트를 확인한 뒤 `agent start`로 같은 이름·설정으로 다시 띄웠다. 재시작한 뒤 새로 설치한 스킬(youtube-batch)이 인식됐다.
- 그 뒤 scouter 입력칸에 보내지 않은 글이 남았을 때 `send-keys`로 보낸 Esc·Ctrl+U·Ctrl+C·Enter가 화면에 반영되지 않았다. 원인은 확인하지 못했고, 사람이 그 창에서 직접 Enter를 눌러 해결했다.

## 7. 하지 않을 것

- `herdr server stop` — 세션의 모든 패널이 같이 꺼진다.
- 에이전트 안에서 bare `herdr` — TUI가 뜬다. `herdr <그룹>`으로 도움말을 본다.
- 자기가 만들지 않은 탭·패널 닫기 — 사람이 시킬 때만.
- 다른 워크스페이스의 에이전트·패널 다루기 — `agent list`의 `workspace_id`가 자기 것인 것만.

## 관련 페이지

- [Herdr 멀티 에이전트 — 역할과 구성](herdr-multi-agent-roles.md)
- [Claude 스킬 작성 규칙 7가지](../youtube/claude-skill-writing-rules.md) — herdr 스킬처럼 에이전트에 설치하는 스킬을 만들 때
