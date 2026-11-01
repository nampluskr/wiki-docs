---
tags: [herdr, multi-agent, guide]
created: 2026-10-30
updated: 2026-10-30
---

# BUILDING — 승인된 구성안대로 구축하기

> 이 파일을 읽는 때: 사람이 `docs/HERDR-SETUP.md`를 승인한 뒤, leader가 탭·패널·에이전트를 만들 때. **승인 전에는 이 파일의 명령을 실행하지 않는다** ([PLANNING.md](PLANNING.md) 3절).

입구는 [README.md](README.md)다.

## 1. 탭과 패널 배치

탭 표준

| 탭 | 이름 | 담는 것 |
| --- | --- | --- |
| 1 | `control` | leader 하나만. 사람이 보는 화면 |
| 2 | `agents` | 하위 에이전트들 |
| 3 | `run` | runner 패널: 테스트, 서버, 학습 로그 (필요할 때만) |

패널 배치 (130×37 기준)
- 열은 **최대 2개**까지. 3열로 나누면 한 열이 40칸 남짓이 되어 읽기 어렵다.
- 3개 이상이면 왼쪽에 주력(writer·coder), 오른쪽 열을 위아래로 나눈다.

```
탭 2 agents
┌──────────────────────┬──────────────────────┐
│                      │ checker / scouter    │
│  writer / coder      ├──────────────────────┤
│  (주력)               │ reviewer (선택)       │
└──────────────────────┴──────────────────────┘
```

## 2. 구축할 때 지킬 것

- **같은 파일을 두 에이전트가 동시에 고치지 않는다.** 구현을 병렬로 해야 하면 `herdr worktree create`로 작업 폴더를 나눈다.
- 에이전트를 띄울 때 **반드시 이름을 붙인다.** (`agent start <이름>`, 이미 떠 있으면 `agent rename`)
- runner 패널에도 이름을 붙인다: `herdr pane rename <패널 ID> runner`
- 백그라운드 패널은 `--no-focus`로 만든다. 사람이 보는 화면을 빼앗지 않는다.
- 구축 중에 구성안과 다르게 해야 할 일이 생기면(패널이 좁아 배치를 바꿔야 하는 등) 멈추고 구성안을 고쳐 다시 승인받는다.

## 3. 구축 명령

출발점: Herdr를 실행하고, 패널 하나에서 `claude`를 띄운 상태. 예시는 [PLANNING.md](PLANNING.md) 6.2 문서 구성이다. 사람이 직접 칠 필요는 없다. 무엇이 일어나는지 알아두는 용도다.

```bash
# 0) Herdr 안인지 확인
test "$HERDR_ENV" = 1

# 1) 현재 위치 확인, 탭 이름 바꾸기, 자기 이름 붙이기
herdr pane current --current
herdr tab rename "$HERDR_TAB_ID" control
herdr agent rename "$HERDR_PANE_ID" leader

# 2) 작업 탭 만들기 → 응답의 .result.root_pane.pane_id 가 첫 패널
herdr tab create --workspace "$HERDR_WORKSPACE_ID" --label agents --cwd "$PWD" --no-focus

# 3) 주력 에이전트 띄우기 (첫 패널)
herdr agent start writer --kind claude --pane <첫 패널 ID> -- --model sonnet --effort high

# 4) 오른쪽으로 나누고 checker 띄우기 → 응답의 .result.pane.pane_id
herdr pane split <첫 패널 ID> --direction right --cwd "$PWD" --no-focus
herdr agent start checker --kind codex --pane <새 패널 ID> -- -m gpt-6.1-terra -c model_reasoning_effort="low"

# 5) (선택) 오른쪽 패널을 아래로 나누고 reviewer 띄우기 (Claude 주력 페어 → Codex sol)
herdr pane split <checker 패널 ID> --direction down --cwd "$PWD" --no-focus
herdr agent start reviewer --kind codex --pane <새 패널 ID> -- -m gpt-6.1-sol -c model_reasoning_effort="medium"

# 6) 확인 — 결과를 docs/HERDR-SETUP.md 의 구성과 대조해 보고
herdr agent list
```

- `<...>` 자리는 실행할 때마다 Herdr가 돌려주는 값으로 채운다. 미리 짐작하지 않는다.
- 이 예시는 워크스페이스 하나 기준이라 이름에 접두어가 없다. 여러 워크스페이스를 돌릴 때는 `visionlab-writer`처럼 붙인다 (6절).
- `agent start`는 이미 있는 **빈 셸 패널**에만 쓸 수 있다. 패널을 만들어 주지는 않는다.
- 사람이 손으로 이미 띄운 에이전트에는 이름만 붙인다: `herdr agent rename <패널 ID> checker`

## 4. 일을 맡기고 결과 받기

```bash
# 지시하고 끝날 때까지 기다림 (최대 10분)
herdr agent prompt writer "docs/guides/x.md 의 3절을 작성하라. 완료 시 바뀐 파일 경로만 답하라." --wait --timeout 600000

# 결과 읽기
herdr agent read writer --source recent-unwrapped --lines 120

# 오래 걸리는 실행은 run 탭의 runner 패널에서
herdr pane run <runner 패널 ID> "python train.py"
herdr pane wait-output <runner 패널 ID> --match "epoch 10" --timeout 3600000
```

질문·승인 요청이 오면 [OPERATING.md](OPERATING.md)를 따른다.

## 5. runner 운영 규칙

runner는 에이전트가 명령을 실행하려고 쓰는 **일반 터미널 패널**이다. 에이전트가 아니므로 토큰을 쓰지 않는다.

### 5.1 언제 runner를 쓰나

| runner에서 실행한다 | 에이전트가 직접 실행한다 |
| --- | --- |
| 오래 걸리는 실행 (학습, 빌드, 긴 테스트) | 짧은 명령 (`git status`, 파일 확인, 테스트 1회) |
| 계속 떠 있어야 하는 프로세스 (서버, watch 모드 테스트) | 결과를 바로 써야 하는 명령 |
| 여러 에이전트나 사람이 같은 출력을 봐야 할 때 | |
| 출력이 많아 필요한 줄만 골라 읽어야 할 때 | |
| 대화형 프로그램 (REPL, 디버거) | |

에이전트 안에서 서버를 띄우면 그 에이전트가 붙잡히거나, 세션이 끝날 때 같이 꺼진다. runner에서 띄우면 에이전트는 다른 일을 하고 필요할 때만 출력을 읽는다.

### 5.2 쓰는 방법

```bash
herdr pane run <runner ID> "python train.py"                        # 명령 실행
herdr pane wait-output <runner ID> --match "Done" --timeout 3600000 # 특정 출력까지 대기
herdr pane read <runner ID> --source recent-unwrapped --lines 50    # 결과 읽기
herdr pane send-keys <runner ID> ctrl+c                             # 중단
```

- `pane run` 성공은 **명령이 입력됐다는 뜻**이지 실행이 끝났다는 뜻이 아니다. 결과는 `wait-output`이나 `read`로 확인한다.
- `wait-output`은 이미 화면에 있는 출력과도 맞춰 본다. 같은 명령을 다시 돌릴 때는 이전 출력과 겹치지 않는 문자열을 고르거나 `--regex`를 쓴다.
- `--lines`로 읽는 양을 줄인다. 특히 Codex·agy에게 넘길 때는 마지막 몇 십 줄만 넘긴다.

### 5.3 규칙

1. **runner 하나는 주인이 하나다.** 두 에이전트가 같은 셸에 동시에 입력하면 명령이 섞인다. 기본 주인은 leader다. 특정 에이전트에게 맡길 때는 주인을 이름에 넣는다. 예) `runner-coder`
2. **용도별로 나눈다.** 서버와 테스트를 같은 셸에서 돌리지 않는다. `run` 탭을 위아래로 나눠 `runner-server`, `runner-test`처럼 쓴다. 130×37 화면에서는 한 탭에 runner 2~3개까지가 읽기 좋다.
3. **보내기 전에 프롬프트 상태인지 확인한다.** 이전 명령이 아직 돌고 있으면 새 명령이 그 프로그램의 입력으로 들어간다. `herdr pane read <runner ID> --source visible`로 셸 프롬프트가 보이는지 먼저 본다.
4. **작업 폴더를 맞춘다.** runner를 만들 때 `--cwd`로 작업 폴더를 지정한다. worktree를 쓰면 worktree마다 runner를 따로 둔다.
5. **위험한 명령은 runner로 우회하지 않는다.** 삭제, 외부 전송, 설치 같은 명령은 runner에서 실행하더라도 [OPERATING.md](OPERATING.md) 1절 기준을 그대로 따른다. 에이전트의 승인 절차를 runner로 피해 가지 않는다.
6. **끝나면 정리한다.** 떠 있는 프로세스를 `ctrl+c`로 끝내고, leader가 만든 runner 패널만 닫는다 ([OPERATING.md](OPERATING.md) 5절).

### 5.4 runner 만들기 예시

```bash
# run 탭 만들기 → 응답의 .result.root_pane.pane_id 가 첫 runner
herdr tab create --workspace "$HERDR_WORKSPACE_ID" --label run --cwd "$PWD" --no-focus
herdr pane rename <첫 패널 ID> runner-server

# 아래로 나눠 두 번째 runner
herdr pane split <첫 패널 ID> --direction down --cwd "$PWD" --no-focus
herdr pane rename <새 패널 ID> runner-test
```

참고: `herdr terminal` 명령 그룹(`attach`, `observe`)은 사람이 터미널에 직접 붙어서 보는 용도다. 에이전트가 명령을 실행하는 데는 쓰지 않는다.

## 6. 여러 워크스페이스 동시 운영

**프로젝트 하나 = 워크스페이스 하나 = leader 하나.** 사람은 프로젝트마다 그 워크스페이스의 leader와 대화한다.

```
Herdr 세션
├─ 워크스페이스 visionlab   leader(Opus) + Claude 주력 페어
├─ 워크스페이스 inbox       leader(Opus) + Codex 주력 페어
└─ 워크스페이스 llmlabs     leader(Opus) + Claude 주력 페어
```

### 6.1 새 워크스페이스 만들기

```bash
# 응답의 .result.root_pane.pane_id 에 그 프로젝트의 leader 를 띄운다
herdr workspace create --label visionlab --cwd "D:/projects/visionlab" --no-focus
herdr agent start visionlab-leader --kind claude --pane <첫 패널 ID> -- --model opus --effort medium
```

워크스페이스와 leader를 만드는 것도 구성의 일부다. 다른 워크스페이스의 leader가 새 워크스페이스를 만들 때는 그 내용을 자기 구성안에 적고 승인받는다. 사람이 직접 만들어도 된다. 새 leader도 구성안부터 쓴다 ([PLANNING.md](PLANNING.md) 3절).

### 6.2 이름 규칙

**에이전트 이름은 Herdr 세션 전체에서 유일해야 한다.** 워크스페이스가 달라도 `coder`가 둘이면 충돌한다.
- 형식: `<프로젝트>-<역할>` — 예) `visionlab-leader`, `visionlab-coder`, `inbox-reviewer`
- 소문자로 시작하고, 소문자·숫자·`-`·`_`만, 32자 이하
- 프로젝트 이름이 길면 줄인다: `youtube-downloader` → `ytdl-coder`
- runner 패널도 같은 형식: `visionlab-runner-test`
- 워크스페이스를 하나만 쓸 때는 접두어를 생략해도 된다

### 6.3 페어를 번갈아 배치해 부담을 나눈다

| 동시 워크스페이스 | 배치 | Claude 부담 | Codex 부담 |
| --- | --- | --- | --- |
| 1개 | Claude 주력 | leader + coder | reviewer |
| 2개 | Claude 주력 + Codex 주력 | leader×2 + coder + reviewer | coder + reviewer |
| 3개 | Claude 주력 ×2 + Codex 주력 | leader×3 + coder×2 + reviewer | coder + reviewer×2 |

- 각 워크스페이스의 구성안(`docs/HERDR-SETUP.md`) "토큰 부담" 항목에 다른 워크스페이스와 겹치는지 적는다. 사람은 승인할 때 이 항목으로 전체 부담을 본다.
- Codex는 기본 요금제라 **Codex 주력 페어는 동시에 1개까지**만 둔다.
- Opus는 leader 수만큼 상시로 쓰고, reviewer로는 필요할 때 짧게만 쓴다.
- 한도가 빠듯하면 동시에 돌리지 말고 차례를 둔다. 예) 한 워크스페이스의 리뷰가 끝난 뒤 다음 워크스페이스의 리뷰를 시작한다.

### 6.4 워크스페이스 사이의 경계

- leader는 **자기 워크스페이스 안의** 에이전트·패널만 다룬다. `herdr agent list` 결과에서 `workspace_id`가 자기 `$HERDR_WORKSPACE_ID`와 같은 것만 대상으로 한다.
- 대리 응답 기록 `docs/HERDR-DECISIONS.md`는 프로젝트 폴더마다 따로 둔다.
- 사람이 다른 프로젝트 이야기를 하려면 그 워크스페이스의 leader에게 간다.

## 7. 명령어 빠른 참조

| 하고 싶은 것 | 명령 |
| --- | --- |
| 현황 보기 | `herdr workspace list` · `herdr tab list --workspace <ID>` · `herdr pane list --workspace <ID>` · `herdr agent list` |
| 내 위치 | `herdr pane current --current` |
| 탭 만들기 | `herdr tab create --workspace <ID> --label <이름> --cwd <경로> --no-focus` |
| 탭 이름 바꾸기 | `herdr tab rename <탭 ID> <이름>` |
| 패널 나누기 | `herdr pane split <패널 ID> --direction right\|down --cwd <경로> --no-focus` |
| 패널 이름 붙이기 | `herdr pane rename <패널 ID> runner` |
| 배치 확인 | `herdr pane layout --pane <패널 ID>` |
| 에이전트 띄우기 | `herdr agent start <이름> --kind claude\|codex\|agy --pane <패널 ID> -- <모델 인자>` |
| 에이전트 이름 붙이기 | `herdr agent rename <패널 ID> <이름>` |
| 지시하고 기다리기 | `herdr agent prompt <이름> "<지시>" --wait --timeout <ms>` |
| 결과 읽기 | `herdr agent read <이름> --source recent-unwrapped --lines 120` |
| 키 보내기 | `herdr agent send-keys <이름> esc` |
| 셸 명령 실행 | `herdr pane run <패널 ID> "<명령>"` |
| 출력 기다리기 | `herdr pane wait-output <패널 ID> --match "<문자열>" --timeout <ms>` |
| 셸 출력 읽기 | `herdr pane read <패널 ID> --source recent-unwrapped --lines 50` |
| 셸 실행 중단 | `herdr pane send-keys <패널 ID> ctrl+c` |
| 작업 폴더 나누기 | `herdr worktree create --workspace <ID> --branch <이름> --no-focus` |

에이전트별 모델 인자 (`--` 뒤에 붙인다)

| 에이전트 | 예시 | 확인 방법 |
| --- | --- | --- |
| claude (Opus) | `--model opus --effort medium` (effort: low·medium·high·xhigh·max) | `claude --help` |
| claude (Sonnet) | coder·writer: `--model sonnet --effort high` | 위와 같음 |
| codex (sol) | `-m gpt-6.1-sol -c model_reasoning_effort="medium"` | `codex --help`, `~/.codex/models_cache.json` |
| codex (terra) | coder·writer: `-m gpt-6.1-terra -c model_reasoning_effort="high"` · checker: `"low"` | 위와 같음 |
| agy | `--model gemini-3.8-flash-low` (`-medium`·`-high`로 effort 조절) | `agy models` |

참고: Codex 기본 설정(`~/.codex/config.toml`)은 아직 `gpt-5.6-sol`이다. 위 인자처럼 띄울 때 모델을 지정하면 설정을 바꾸지 않아도 된다.
