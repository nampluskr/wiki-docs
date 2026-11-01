---
tags: [herdr, multi-agent, guide]
created: 2026-10-30
updated: 2026-10-31
---

# PLANNING — 구성안 쓰기

> 이 파일을 읽는 때: leader가 구성안(`docs/HERDR-SETUP.md`)을 쓸 때. 사람은 3절 "구성안 작성과 승인"만 읽으면 된다.

입구는 [README.md](README.md)다. 구성안 템플릿은 [TEMPLATE-SETUP.md](TEMPLATE-SETUP.md)에 있다.

## 1. 진입 에이전트 고르기

처음 띄우는 에이전트가 **leader**가 된다. claude·codex·agy 중 무엇이든 될 수 있다. `herdr`는 셸 명령이라 셸을 쓸 수 있는 에이전트라면 다 조작할 수 있다.

| 진입 에이전트 | 장점 | 주의 |
| --- | --- | --- |
| **claude · Opus 5.5 (권장)** | Max 요금제라 토큰 여유가 크다. herdr 스킬이 설치돼 있어 명령을 정확히 쓴다 | — |
| codex · `gpt-6.1-sol` | 기본 요금제 | 조율 대화만으로도 한도가 줄어든다. 이 문서 묶음을 먼저 읽게 해야 한다 |
| agy · Gemini 3.8 Flash | 기본 요금제 | 가벼운 모델이라 여러 에이전트 조율에는 약하다. 권장하지 않는다 |

leader는 대화가 길어지고 결과를 계속 읽어 들이기 때문에 토큰을 가장 많이 쓴다. 그래서 토큰이 넉넉한 Claude를 권장한다.

codex·agy를 진입으로 쓸 때 첫 지시
```
너는 이 Herdr 세션의 leader 다.
D:\projects\docs\guides\herdr\README.md 를 읽고, 읽기 지도에 따라 필요한 파일만 읽는다.
herdr 명령은 `herdr <그룹>`(예: herdr pane)으로 도움말을 확인한 뒤 쓴다.
bare `herdr`는 실행하지 않는다.
```

## 2. leader는 유일한 창구다

- 사람은 탭 1의 leader하고만 대화한다.
- leader는 지시를 쪼개서 하위 에이전트에게 맡기고, 결과를 모아 보고한다.
- leader는 **직접 큰 작업을 하지 않는다.** 직접 하면 대화창이 결과물로 가득 차서 조율을 못 하게 된다. 짧은 확인(파일 하나 읽기, 명령 하나 실행)은 직접 해도 된다.
- leader는 **환경을 임의로 구축하지 않는다.** 구성안을 문서로 먼저 쓰고 사람의 승인을 받은 뒤에만 만든다 (3절).

## 3. 구성안 작성과 승인

leader는 탭·패널·에이전트를 만들기 전에 **구성안 문서를 쓰고 사람의 검토와 승인을 받는다.** 승인 전에는 읽기 전용 확인(`herdr ... list`, `pane layout`, 파일 읽기)만 한다.

### 3.1 근거 찾기

구성안은 사람의 의도에 맞아야 한다. leader는 아래 순서로 근거를 찾는다.
1. 프로젝트 폴더에 `INTENT.md`가 있으면 먼저 읽는다
2. 사람의 이번 지시
3. 둘 다로 정할 수 없는 것(작업 유형, reviewer를 둘지, 동시에 돌릴 워크스페이스 수 등)은 짐작하지 않고 사람에게 묻는다

### 3.2 구성안 문서

프로젝트 폴더의 `docs/HERDR-SETUP.md`. 아래 항목을 담는다. 템플릿과 작성 예시는 [TEMPLATE-SETUP.md](TEMPLATE-SETUP.md)에 있다.

| 항목 | 내용 |
| --- | --- |
| 상태 | `초안` → `승인됨 (YYYY-MM-DD)` → `종료 (YYYY-MM-DD)` |
| 근거 | `INTENT.md`의 해당 부분, 사람의 지시를 인용 |
| 작업 유형 | 6절의 어느 구성을 바탕으로 했는지, 바꾼 점과 이유 |
| 탭·패널 배치 | 탭 목록과 패널 배치 그림 |
| 에이전트 | 이름 · kind · 모델 · effort · 역할 · 파일 수정 범위 |
| runner | 이름 · 용도 · 주인 |
| 토큰 부담 | 구독별 부담, 다른 워크스페이스와 겹치는지 |
| 대리 응답 범위 | [OPERATING.md](OPERATING.md) 1절 기준 외에 이 프로젝트에서 따로 정할 것 |
| 정리 방법 | 끝났을 때 닫을 것 |

### 3.3 승인 절차

1. leader가 `docs/HERDR-SETUP.md`를 `초안`으로 쓰고, 요약과 파일 경로를 사람에게 보고한다
2. 사람이 검토하고 고칠 점을 말한다. leader는 고쳐서 다시 보고한다
3. 사람이 명시적으로 승인하면("승인", "진행해" 등) 상태를 `승인됨 (날짜)`로 바꾸고 구축한다 ([BUILDING.md](BUILDING.md))
4. 구축이 끝나면 `herdr agent list` 결과를 구성안과 대조해 보고한다

사람이 구성안을 요청하는 지시 예시
```
D:\projects\docs\guides\herdr\README.md 를 읽고 이 프로젝트의 Herdr 구성안을 써줘.
- 할 일: "<주제>" 가이드 문서를 docs/<파일>.md 로 작성
- INTENT.md 가 있으면 먼저 읽고 근거로 삼아
- 구성안은 docs/HERDR-SETUP.md 에 초안으로 쓰고, 정할 수 없는 건 나에게 물어봐
- 내가 승인하기 전에는 탭·패널·에이전트를 만들지 마
```

승인 지시 예시
```
docs/HERDR-SETUP.md 승인. 구성안대로 구축해줘.
```

### 3.4 승인된 구성을 바꿀 때

에이전트 추가·교체, 모델 변경, 워크스페이스 추가처럼 구성안과 달라지는 일은 **구성안을 고치고 다시 승인받는다.** 같은 역할의 에이전트를 같은 설정으로 다시 띄우는 것(멈춘 에이전트 재시작)은 승인된 구성 안의 일이다. 단, [OPERATING.md](OPERATING.md) 1절에 따라 기록한다.

대리 응답 범위([OPERATING.md](OPERATING.md) 1절)는 구축 후 하위 에이전트의 질문에 적용된다. **구성 자체는 대리 응답으로 정하지 않는다.**

## 4. 토큰 예산 규칙

요금제가 다르다. Claude는 Max 요금제로 넉넉하고, Codex·agy는 기본 요금제로 한도가 작다. 그래도 여러 워크스페이스를 동시에 돌리면 Claude도 한도에 닿는다. 그래서 **토큰을 가장 많이 쓰는 구현은 싼 모델에게, 상위 모델은 짧고 판단이 필요한 곳에만** 쓴다.

| 모델 | 맡길 일 | 맡기지 않을 일 |
| --- | --- | --- |
| Claude Opus 5.5 | leader의 조율, reviewer의 검토 | 구현, 본문 작성 |
| Claude Sonnet 5.5 | 구현, 문서 본문 작성, 긴 문맥 탐색 | — |
| Codex `gpt-6.1-terra` | 구현(Codex 주력 페어), 단순 확인 | 저장소 전체 탐색 |
| Codex `gpt-6.1-sol` | diff·특정 파일 검토, 결과 해석 | 구현, 긴 왕복 |
| agy Gemini 3.8 Flash | 가벼운 요약·조사 | 구현, 깊은 검토, 반복 왕복 |

규칙
1. reviewer·checker·analyst에게는 저장소 전체가 아니라 **파일 경로나 diff만** 넘긴다.
2. 같은 에이전트와 검토 왕복은 **최대 2회**까지만 한다.
3. **coder·writer는 effort high가 기본**이다. 구현·작성 품질이 결과를 좌우하기 때문이다. 나머지 역할은 low~medium으로 두고, reviewer의 high는 마지막 검토 1회에만 쓴다.
4. agy는 Flash low로 시작하고, 요약이 부실할 때만 medium으로 올린다.
5. 한도가 떨어지면 coder는 Sonnet ↔ terra로 서로 대신하고, reviewer도 페어를 바꾼다 (5.2 교차 구성). Codex·agy가 모두 떨어지면 그 역할을 Sonnet 패널로 바꾼다.

## 5. 역할 카탈로그

### 5.1 역할

역할 이름은 **그 일을 하는 행위자**를 가리키는 명사로 붙인다. 에이전트 이름(`agent start <이름>`)과 패널 이름(`pane rename`)에 그대로 쓴다.

| 역할 이름 | 필수 여부 | 책임 | 하지 않을 일 | 파일 수정 |
| --- | --- | --- | --- | --- |
| `leader` | **필수** | 사람과 대화, 작업 배분, 대리 응답, 결과 보고 | 큰 구현·작성 | `docs/HERDR-SETUP.md`, `docs/HERDR-DECISIONS.md`만 |
| `writer` | 주력 (문서) | 문서 본문 작성·수정 | 범위 밖 변경 | 맡은 파일 |
| `coder` | 주력 (코드) | 코드 작성·수정 | 범위 밖 변경 | 맡은 파일 |
| `scouter` | 선택 | 코드·문서·로그를 찾아 요약 | 수정 | 안 함 |
| `checker` | 선택 | 명령·사실·테스트 결과 확인 | 수정 | 안 함 |
| `analyst` | 선택 | 실험 수치 해석, 다음 실험 제안 | 수정 | 안 함 |
| `runner` | 선택 | 테스트·서버·학습 실행 (일반 셸 패널, 에이전트 아님) | — | — |
| `reviewer` | **선택** | 결과물의 결함을 찾는 적대적 리뷰 | 수정, 근거 없는 "통과" | 안 함 |

**reviewer는 필수가 아니다.** 아래 경우에만 둔다.
- 변경이 크거나 여러 모듈에 걸칠 때
- 되돌리기 어렵거나 다른 사람이 쓸 결과물일 때
- 작성자가 스스로 확인하기 어려운 내용일 때 (포팅 동등성, 수치 해석 등)

둘 때는 작성자와 **다른 회사의 상위 모델**로 고른다. 같은 계열 모델은 같은 실수를 놓치기 쉽고, 싼 모델이 쓴 결과는 상위 모델이 봐야 빈틈을 찾는다.

### 5.2 교차 구성 (coder ↔ reviewer 페어)

| 페어 | coder (구현, 토큰 많이 씀) | reviewer (검토, 짧게 씀) |
| --- | --- | --- |
| Claude 주력 | Claude Sonnet 5.5 · high | Codex `gpt-6.1-sol` · medium~high |
| Codex 주력 | Codex `gpt-6.1-terra` · high | Claude Opus 5.5 · medium~high |

- 기본은 Claude 주력 페어다. 구현량이 많은 쪽을 한도가 넉넉한 Claude에 둔다.
- Codex 주력 페어는 Claude 한도를 아껴야 할 때, 또는 여러 워크스페이스에 부담을 나눌 때 쓴다 ([BUILDING.md](BUILDING.md) 6절).
- writer는 coder와 같은 모델·effort를 쓴다 (Sonnet 5.5 · high 또는 `gpt-6.1-terra` · high).

## 6. 작업 유형별 구성

GitHub 작업 이력(github.com/nampluskr)을 네 가지 유형으로 나눴다. 하려는 일에 맞는 구성을 고른다.

| 유형 | 대표 저장소 |
| --- | --- |
| A. 문서·가이드 | project-workflow, techbase, doccamp, project-templates |
| B. 앱·CLI 개발 | youtube-downloader, speech_transcriber, backlog, inbox-office, markdown-viewer, workbench-kit |
| C. 포팅·리팩터링 | visionlab (anomalib 포팅), defectvad-refactor |
| D. ML·실험 | defectvad, roi-corner-detection, mnist-from-scratch, fft-analysis, llm-labs |

6.6은 A의 변형으로, 웹 조사 비교가 중심인 문서에 쓴다.

이 절의 구성은 **구성안을 쓸 때의 출발점**이다. leader는 이 중 하나를 골라 `INTENT.md`와 사람의 지시에 맞게 고쳐 구성안을 쓰고, 승인받은 뒤에 구축한다 (3절).

모든 유형에서 leader는 **Claude Opus 5.5, effort medium**이다. 판단이 어려운 국면에서만 high로 올린다.

표는 기본인 **Claude 주력 페어**로 적었다. Codex 주력 페어로 돌릴 때는 coder를 `gpt-6.1-terra` · high로, reviewer를 Claude Opus 5.5로 바꾼다 (5.2).

토큰 부담 표시: ● 큼 · ◐ 보통 · ○ 작음

### 6.1 입문 구성 — 에이전트 2개

처음 Herdr를 써 볼 때, 또는 작은 작업일 때 쓴다.

| 이름 | 에이전트 | 모델 · effort | 탭 · 위치 | 역할 | 토큰 |
| --- | --- | --- | --- | --- | --- |
| `leader` | claude | Opus 5.5 · medium | 1 control | 소통 | Claude ◐ |
| `coder` | claude | Sonnet 5.5 · high | 1 control 오른쪽 | 작성·구현 | Claude ● |

```
탭 1 control
┌──────────────────────┬──────────────────────┐
│ leader               │ coder                │
└──────────────────────┴──────────────────────┘
```

Codex·agy를 쓰지 않아 기본 요금제 한도를 아낀다. 익숙해지면 6.2~6.6으로 넘어간다.

### 6.2 A. 문서·가이드

**언제**: 기술 가이드, 매뉴얼, 학습 노트를 쓸 때.

| 이름 | 에이전트 | 모델 · effort | 탭 · 위치 | 역할 | 토큰 |
| --- | --- | --- | --- | --- | --- |
| `leader` | claude | Opus 5.5 · medium | 1 control | 소통, 목차 확정 | Claude ◐ |
| `writer` | claude | Sonnet 5.5 · high | 2 agents 왼쪽 | 본문 작성·수정 | Claude ● |
| `checker` | codex | `gpt-6.1-terra` · low | 2 agents 오른쪽 위 | 명령·사실 확인 (바뀐 절만) | Codex ○ |
| `reviewer` (선택) | codex | `gpt-6.1-sol` · high | 2 agents 오른쪽 아래 | 적대적 리뷰 (완성본 1회) | Codex ◐ |

Codex 주력 페어라면 writer는 `gpt-6.1-terra` · high, reviewer는 Claude Opus 5.5가 된다. 이때 checker는 생략하거나 agy Flash로 바꿔 Codex 부담을 줄인다.

흐름
1. leader가 목차를 정해 사람에게 확인받는다
2. writer가 절 단위로 작성한다
3. checker가 바뀐 절의 명령·사실만 확인한다
4. (선택) 완성본을 reviewer가 한 번 검토한다
5. leader가 맞는 지적만 골라 writer에게 넘기고, 끝나면 보고한다

시작 지시문
```
PLANNING.md 6.2 문서 구성을 바탕으로 구성안을 docs/HERDR-SETUP.md 에 써줘. 주제는 "<주제>", 결과는 docs/<파일>.md. reviewer 는 생략.
승인하면 구축하고, 목차를 먼저 나에게 확인받은 뒤 writer 에게 맡겨줘.
```

### 6.3 B. 앱·CLI 개발

**언제**: pywebview 데스크톱 앱, Python CLI, JS/TS 도구를 만들거나 기능을 추가할 때.

| 이름 | 에이전트 | 모델 · effort | 탭 · 위치 | 역할 | 토큰 |
| --- | --- | --- | --- | --- | --- |
| `leader` | claude | Opus 5.5 · medium | 1 control | 소통, 작업 분해 | Claude ◐ |
| `coder` | claude | Sonnet 5.5 · high | 2 agents 왼쪽 | 구현, 테스트 수정 | Claude ● |
| `scouter` | agy | Gemini 3.8 Flash · low | 2 agents 오른쪽 위 | 라이브러리 문서·기존 코드 조사 요약 | agy ○ |
| `reviewer` (선택) | codex | `gpt-6.1-sol` · medium | 2 agents 오른쪽 아래 | diff 적대적 리뷰 | Codex ◐ |
| `runner` | — | — | 3 run | 테스트, 앱 실행 | 없음 |

```
탭 2 agents                                       탭 3 run
┌──────────────────────┬──────────────────────┐  ┌──────────────────────┐
│                      │ scouter              │  │ runner               │
│  coder               ├──────────────────────┤  │ pytest / 앱 실행      │
│                      │ reviewer (선택)       │  │                      │
└──────────────────────┴──────────────────────┘  └──────────────────────┘
```

흐름
1. leader가 기능을 작업 단위로 쪼갠다
2. 필요하면 scouter에게 조사를 맡긴다 (coder와 동시에 진행)
3. coder가 구현한다. 테스트는 runner에서 돌리고 결과만 coder에게 넘긴다
4. (선택) 큰 변경이면 `git diff`만 reviewer에게 넘긴다
5. 기능을 병렬로 만들려면 `herdr worktree create --branch <이름>`으로 작업 폴더를 나누고 coder를 하나 더 띄운다

### 6.4 C. 포팅·리팩터링

**언제**: 다른 라이브러리·언어의 코드를 옮기거나 구조를 크게 바꿀 때 (예: anomalib → Lightning 포팅).

| 이름 | 에이전트 | 모델 · effort | 탭 · 위치 | 역할 | 토큰 |
| --- | --- | --- | --- | --- | --- |
| `leader` | claude | Opus 5.5 · medium | 1 control | 소통, 모듈 순서 결정 | Claude ◐ |
| `coder` | claude | Sonnet 5.5 · high | 2 agents 왼쪽 | 원본 분석 + 구현 | Claude ● |
| `scouter` | agy | Gemini 3.8 Flash · medium | 2 agents 오른쪽 위 | 원본 모듈 하나씩 동작 요약 | agy ○ |
| `reviewer` (**권장**) | codex | `gpt-6.1-sol` · high | 2 agents 오른쪽 아래 | 모듈 단위 동등성 리뷰 | Codex ◐ |
| `runner` ×2 | — | — | 3 run (위아래 2개) | 원본·포팅 결과 비교 실행 | 없음 |

포인트
- 포팅은 원본과 새 코드를 함께 읽어야 해서 문맥이 크다. 원본 분석의 중심은 coder가 맡고, scouter는 보조 요약만 한다.
- coder가 싼 모델이므로 상위 모델 reviewer의 모듈 단위 검토를 **권장**한다. 핵심 모듈 하나가 계속 막히면 leader 판단으로 그 모듈만 Opus coder를 잠시 띄운다. 이것도 구성 변경이므로 구성안을 고쳐 승인받는다 (3.4).
- scouter에게는 **모듈 하나**씩만 맡긴다. 저장소 전체를 넘기지 않는다.
- 동등성은 말보다 실행으로 확인한다. `run` 탭을 위아래로 나눠 원본과 포팅본을 같은 입력으로 돌리고 출력을 비교한다.
- 이 유형은 reviewer를 둘 가치가 가장 크다. 작성자 스스로 놓친 차이를 찾기 어렵기 때문이다. 리뷰 권장 표시를 붙인 이유다.

### 6.5 D. ML·실험

**언제**: 모델 학습, 이상 탐지 실험, 신호 처리 분석, 노트북 작업.

| 이름 | 에이전트 | 모델 · effort | 탭 · 위치 | 역할 | 토큰 |
| --- | --- | --- | --- | --- | --- |
| `leader` | claude | Opus 5.5 · medium | 1 control | 소통, 실험 계획 | Claude ◐ |
| `coder` | claude | Sonnet 5.5 · high | 2 agents 왼쪽 | 실험 코드·설정 작성 | Claude ● |
| `scouter` | agy | Gemini 3.8 Flash · low | 2 agents 오른쪽 위 | 학습 로그 요약 | agy ○ |
| `analyst` | codex | `gpt-6.1-sol` · medium | 2 agents 오른쪽 아래 | 결과 수치 해석, 다음 실험 제안 | Codex ◐ |
| `runner` | — | — | 3 run | 학습 실행 | 없음 |

포인트
- 학습은 반드시 runner에서 돌린다. leader는 `herdr pane wait-output`으로 끝나기를 기다린다. 기다리는 동안 토큰을 쓰지 않는다.
- 로그 전체를 에이전트에게 넘기지 않는다. 마지막 몇 십 줄이나 지표 파일만 넘긴다.
- reviewer는 보통 필요 없다. 실험 결과가 곧 검증이다.

### 6.6 A-2. 웹 조사 자료 정리

**언제**: 웹 자료를 모아 문서 하나로 정리할 때. 최신 설치 절차·설정처럼 출처 확인이 중요한 가이드 (예: SSH + Tailscale 원격 접속).

| 이름 | 에이전트 | 모델 · effort | 탭 · 위치 | 역할 | 토큰 |
| --- | --- | --- | --- | --- | --- |
| `leader` | claude | Opus 5.5 · medium (작성 시 high) | 1 control | 소통, 조사 지시, 두 결과 비교, **본문 작성** | Claude ● |
| `scouter-opus` | claude | Opus 5.5 · medium | 2 agents 왼쪽 | 웹 독립 조사 → 의견 | Claude ◐ |
| `scouter-sol` | codex | `gpt-6.1-sol` · medium | 2 agents 오른쪽 | 웹 독립 조사 → 의견 | Codex ◐ |

```
탭 2 agents
┌──────────────────────┬──────────────────────┐
│ scouter-opus         │ scouter-sol          │
│ 웹 독립 조사          │ 웹 독립 조사          │
└──────────────────────┴──────────────────────┘
```

흐름
1. leader가 두 scouter에 **같은 조사 지시**를 동시에 보낸다. 서로의 결과는 보여주지 않는다
2. leader가 두 결과를 **일치 / 한쪽만 / 상충**으로 나눠 비교한다. 상충은 출처를 직접 열어 판정하고, 못 하면 사람에게 올린다
3. 비교 요약과 목차를 사람에게 확인받는다
4. leader가 본문을 쓴다. 상충·확신 낮은 항목만 해당 scouter에 다시 묻는다 (최대 2회)
5. 비교 판정과 채택 근거를 `docs/HERDR-DECISIONS.md`에 남기고 보고한다

포인트
- **6.2와 다른 점**: writer·checker·reviewer 대신, 서로 다른 회사 모델 2개의 독립 조사를 비교해 검증한다. 문서가 하나이고 판단이 곧 작성이라 leader가 직접 쓴다. 2절 "leader는 큰 작성을 하지 않는다"의 예외이므로 구성안에 적는다.
- 조사에 상위 모델을 쓰는 것은 4절 기본(조사 = Flash)과 다르다. 출처 판단 품질이 결과를 좌우하는 문서에만 쓰고, 가벼운 요약이면 한쪽을 agy Flash로 바꾼다.
- 조사 지시의 **출력 형식을 고정**한다: ① 권장 절차 요약 ② 항목별 명령·설정 + 출처 URL ③ 권장 목차 ④ 주의점·논쟁점 ⑤ 확신 낮은 부분. 형식이 같아야 비교가 쉽다.
- 출처는 공식 문서 우선. 출처 없는 주장은 반영하지 않는다.
- scouter는 파일을 쓰지 않고 화면에 출력만 한다. 분량을 제한(예: 150줄)해 leader 문맥을 지킨다.
- 대리 응답: 웹 검색·페이지 열람 승인은 대리 승인하고, 파일 쓰기·시스템 설정 변경 요청은 거절한다.

시작 지시문
```
PLANNING.md 6.6 웹 조사 구성을 바탕으로 구성안을 docs/HERDR-SETUP.md 에 써줘.
주제는 "SSH + Tailscale 원격 터미널 접속", 결과는 docs/guides/ssh-tailscale-remote-terminal-guide.md.
승인하면 구축하고, 두 조사 결과 비교 요약과 목차를 먼저 나에게 확인받은 뒤 작성해줘.
```

기존 에이전트로 다음 작업 시작하기

한 문서를 끝내고 leader 대화를 `/clear`로 비우면, 탭·패널·에이전트 이름·모델 설정은 남지만 leader의 역할 기억은 사라진다. 아래처럼 첫 지시로 역할을 다시 알려 준다. scouter의 대화는 따로 비우기 전까지 남아 있다.

```
너는 이 Herdr 워크스페이스(<워크스페이스 ID>)의 leader 다.
D:\projects\docs\guides\herdr\README.md 를 읽고, 읽기 지도에 따라 필요한 파일만 읽어.
이번에도 PLANNING.md 6.6 웹 조사 구성으로 한다.
- 탭 2의 기존 에이전트를 그대로 쓴다: <scouter-opus 이름>(Opus·medium), <scouter-sol 이름>(sol·medium)
- 두 에이전트의 이름을 이번 작업에 맞게 바꾸고, 대화를 비운 뒤 일을 맡겨
- 슬래시 명령(/clear, /new)을 Git Bash 로 보낼 때는 MSYS_NO_PATHCONV=1 을 붙여
- 할 일: "<새 주제>" 가이드를 docs/guides/<파일>.md 로 작성
- 조사 결과 비교 요약과 목차를 먼저 나에게 확인받고 작성해
```

leader가 하는 일
1. `herdr agent list`로 탭 2의 에이전트를 확인한다
2. 이름을 바꾼다: `herdr agent rename <기존 이름> <새 이름>`. 이름은 세션 전체에서 유일해야 한다 ([BUILDING.md](BUILDING.md) 6.2)
3. scouter 대화를 비운다: claude는 `/clear`, codex는 `/new`
4. 새 주제의 조사 지시를 두 scouter에 같은 내용으로 보낸다

- Git Bash는 `/clear`·`/new`를 Windows 경로(`C:/Program Files/Git/clear`)로 바꿔 보낸다. `MSYS_NO_PATHCONV=1`을 붙이지 않으면 엉뚱한 지시가 들어간다.
- 구성안(`docs/HERDR-SETUP.md`)을 쓸지는 사람이 정한다. 쓰지 않기로 했다면 지시에 "HERDR-SETUP.md·HERDR-DECISIONS.md 는 만들지 마"를 넣는다.
- claude scouter의 긴 결과는 `agent read`로 앞부분까지 다 읽히지 않을 때가 있다. 그때는 결과를 임시 폴더의 Markdown 파일로 저장하게 하고 경로만 답하게 한다.
