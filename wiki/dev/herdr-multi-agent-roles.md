---
title: Herdr 멀티 에이전트 — 역할과 구성
sources:
  - raw/dev/herdr/README.md
  - raw/dev/herdr/PLANNING.md
  - raw/dev/herdr/BUILDING.md
  - raw/dev/herdr/OPERATING.md
  - raw/dev/herdr/TEMPLATE-SETUP.md
  - raw/dev/herdr/TEMPLATE-DECISIONS.md
updated: 2026-11-01
tags: [herdr, multi-agent, leader, token, claude, codex, agy]
status: draft
---

# Herdr 멀티 에이전트 — 역할과 구성

Herdr에서 여러 AI 에이전트에게 **역할을 나눠** 일을 맡기는 방법이다. Herdr의 구조와 명령은 [Herdr 기본](herdr-basics.md)에 있다.

## 1. 핵심 원칙

1. **사람은 leader하고만 대화한다.** 패널 배치·에이전트 실행·지시 전달은 leader가 `herdr` 명령으로 한다.
2. **leader는 환경을 임의로 만들지 않는다.** 구성안을 먼저 쓰고 사람의 승인을 받는다.
3. **leader는 하위 에이전트에게 사람 대신 답할 수 있지만, 근거를 기록한다.**
4. **구현은 싼 모델, 판단은 상위 모델.** 검토는 작성자와 다른 회사의 상위 모델이 한다.
5. **오래 걸리는 실행은 runner에서.**

leader는 큰 작업을 직접 하지 않는다. 직접 하면 대화창이 결과물로 차서 조율을 못 한다. 파일 하나 읽기, 명령 하나 실행 같은 짧은 확인은 직접 해도 된다.

## 2. 진입 에이전트(leader) 고르기

처음 띄운 에이전트가 leader가 된다. 셸을 쓸 수 있으면 어느 CLI든 된다.

| 진입 에이전트 | 평가 |
| --- | --- |
| claude · Opus (권장) | 토큰 여유가 크고(Max 요금제), herdr 스킬로 명령을 정확히 쓴다 |
| codex · `gpt-6.1-sol` | 조율 대화만으로 한도가 줄어든다. 가이드를 먼저 읽게 해야 한다 |
| agy · Gemini Flash | 가벼운 모델이라 여러 에이전트 조율에 약하다. 권장하지 않는다 |

leader는 대화가 길고 결과를 계속 읽어 들여서 토큰을 가장 많이 쓴다.

## 3. 역할 카탈로그

역할 이름은 그 일을 하는 **행위자 명사**로 붙이고, 에이전트 이름에 그대로 쓴다.

| 역할 | 필수 | 책임 | 파일 수정 |
| --- | --- | --- | --- |
| `leader` | 필수 | 사람과 대화, 배분, 대리 응답, 보고 | 구성안·대리 응답 기록만 |
| `writer` | 주력(문서) | 문서 작성·수정 | 맡은 파일 |
| `coder` | 주력(코드) | 코드 작성·수정 | 맡은 파일 |
| `scouter` | 선택 | 코드·문서·로그·웹을 찾아 요약 | 안 함 |
| `checker` | 선택 | 명령·사실·테스트 결과 확인 | 안 함 |
| `analyst` | 선택 | 실험 수치 해석, 다음 실험 제안 | 안 함 |
| `reviewer` | 선택 | 결함을 찾는 적대적 리뷰 | 안 함 |
| `runner` | 선택 | 테스트·서버·학습 실행(셸 패널, 에이전트 아님) | — |

reviewer는 변경이 크거나, 되돌리기 어렵거나, 작성자가 스스로 확인하기 어려운 내용(포팅 동등성, 수치 해석)일 때만 둔다. 같은 계열 모델은 같은 실수를 놓치기 쉬우므로 **작성자와 다른 회사의 상위 모델**로 고른다.

## 4. 모델과 토큰 예산

원본 기준 요금제: Claude Max(넉넉함), Codex·agy 기본(한도 작음).

| 모델 | 맡길 일 | 맡기지 않을 일 |
| --- | --- | --- |
| Claude Opus | leader 조율, reviewer 검토 | 구현, 본문 작성 |
| Claude Sonnet | 구현, 본문 작성, 긴 문맥 탐색 | — |
| Codex `gpt-6.1-terra` | 구현(Codex 주력), 단순 확인 | 저장소 전체 탐색 |
| Codex `gpt-6.1-sol` | diff·특정 파일 검토, 결과 해석 | 구현, 긴 왕복 |
| agy Gemini Flash | 가벼운 요약·조사 | 구현, 깊은 검토, 반복 왕복 |

규칙:

- reviewer·checker·analyst에게는 저장소 전체가 아니라 **파일 경로나 diff만** 넘긴다.
- 같은 에이전트와 검토 왕복은 **최대 2회**.
- coder·writer는 effort high가 기본. 나머지는 low~medium, reviewer의 high는 마지막 1회.
- agy는 Flash low로 시작하고 요약이 부실할 때만 올린다.
- 한도가 떨어지면 coder는 Sonnet ↔ terra로, reviewer도 페어를 바꾼다.

교차 페어:

| 페어 | coder(토큰 많이 씀) | reviewer(짧게 씀) |
| --- | --- | --- |
| Claude 주력(기본) | Claude Sonnet · high | Codex `gpt-6.1-sol` · medium~high |
| Codex 주력 | Codex `gpt-6.1-terra` · high | Claude Opus · medium~high |

## 5. 작업 유형별 구성

모든 유형에서 leader는 Claude Opus · medium이다.

| 유형 | 언제 | 하위 에이전트 | 비고 |
| --- | --- | --- | --- |
| 입문 | 처음 써 볼 때, 작은 작업 | coder(Sonnet · high) | Codex·agy 한도를 아낀다 |
| A. 문서·가이드 | 기술 가이드, 매뉴얼 | writer(Sonnet · high), checker(terra · low), reviewer 선택(sol · high) | 목차를 사람에게 먼저 확인 |
| A-2. 웹 조사 정리 | 출처 확인이 중요한 가이드 | scouter 2개(Opus, sol) 독립 조사 | 두 결과를 일치·한쪽만·상충으로 비교. leader가 본문을 직접 쓴다(예외) |
| B. 앱·CLI 개발 | 데스크톱 앱, CLI, 도구 | coder(Sonnet · high), scouter(Flash · low), reviewer 선택(sol), runner | 병렬 기능은 worktree로 나눈다 |
| C. 포팅·리팩터링 | 라이브러리·언어 이전 | coder, scouter(Flash · medium), reviewer **권장**(sol · high), runner ×2 | scouter에게 모듈 하나씩. 동등성은 원본·포팅본 실행 비교로 |
| D. ML·실험 | 학습, 이상 탐지, 신호 처리 | coder, scouter(로그 요약), analyst(sol), runner | 학습은 반드시 runner. 로그는 마지막 몇십 줄만 |

탭 배치(130×37 기준): 탭 1 `control`(leader만), 탭 2 `agents`(하위 에이전트), 탭 3 `run`(runner, 필요할 때). 한 탭의 열은 **최대 2개**까지이고, 3개 이상이면 왼쪽에 주력을 두고 오른쪽 열을 위아래로 나눈다.

## 6. 구성안과 승인

leader는 탭·패널·에이전트를 만들기 전에 프로젝트의 `docs/HERDR-SETUP.md`에 구성안을 쓰고 승인받는다. 승인 전에는 읽기 전용 확인만 한다.

1. 근거를 찾는다: `INTENT.md` → 사람의 이번 지시 → 둘로 정할 수 없는 것은 묻는다.
2. 구성안에 상태, 근거, 작업 유형, 탭·패널 배치, 에이전트(이름·kind·모델·effort·역할·파일 수정 범위), runner, 토큰 부담, 대리 응답 범위, 정리 방법을 쓴다.
3. 사람이 명시적으로 승인하면 상태를 `승인됨 (날짜)`로 바꾸고 구축한다.
4. 구축 뒤 `herdr agent list`를 구성안과 대조해 보고한다.

에이전트 추가·교체, 모델 변경, 워크스페이스 추가는 구성안을 고쳐 다시 승인받는다. 멈춘 에이전트를 같은 설정으로 다시 띄우는 것은 승인된 구성 안의 일이다. 구성안 템플릿: [TEMPLATE-SETUP](../../raw/dev/herdr/TEMPLATE-SETUP.md)

## 7. 대리 응답과 기록

| leader가 답한다 | 사람에게 올린다 |
| --- | --- |
| 기존 지시나 문서로 답이 정해지는 질문 | 범위를 넓히거나 줄이는 결정 |
| 작업 폴더 안의 읽기·쓰기·테스트 실행 승인 | 삭제, 되돌리기 어려운 작업 |
| 정한 방향 안의 세부 선택(이름, 순서) | 외부 전송(push·메일·업로드), 비용이 드는 작업 |
| | 승인된 구성과 달라지는 일, 근거를 댈 수 없는 판단 |

답하거나 조치할 때마다 `docs/HERDR-DECISIONS.md`에 번호(`D-001`…)를 붙여 바로 기록한다. 대리 응답, 사람에게 올린 건, 구성 안의 조치(재시작 등)를 모두 적고, 이전 기록은 고치지 않고 새 번호로 바로잡는다. 기록 템플릿: [TEMPLATE-DECISIONS](../../raw/dev/herdr/TEMPLATE-DECISIONS.md)

## 8. 여러 워크스페이스

**프로젝트 하나 = 워크스페이스 하나 = leader 하나.** 사람은 프로젝트마다 그 leader와 대화한다.

- 에이전트 이름에 프로젝트 접두어를 붙인다(`visionlab-leader`, `ytdl-coder`).
- Claude 주력과 Codex 주력 페어를 번갈아 배치해 구독 부담을 나눈다. Codex 주력 페어는 동시에 1개까지.
- 한도가 빠듯하면 동시에 돌리지 말고 차례를 둔다.
- leader는 자기 워크스페이스의 에이전트·패널만 다룬다.

## 9. 정리

작업이 끝나면: 하위 에이전트가 `idle`·`done`인지 확인 → runner의 프로세스를 `ctrl+c`로 끝냄 → 자기가 만든 패널·탭만 닫음 → worktree는 사람에게 확인받고 지움 → 대리 응답 기록을 요약 보고 → 구성안 상태를 `종료 (날짜)`로.

한 작업을 끝내고 leader 대화를 비우면 탭·에이전트는 남지만 leader의 역할 기억은 사라진다. 첫 지시로 역할과 가이드 위치를 다시 알려 준다.

## 관련 페이지

- [Herdr 기본 — 구조, 상태, 명령](herdr-basics.md)
