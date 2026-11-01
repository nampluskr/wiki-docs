---
tags: [herdr, multi-agent, guide]
created: 2026-10-30
updated: 2026-10-31
---

# Herdr 멀티 에이전트 실용 매뉴얼

> 이 파일을 읽는 때: 처음, 그리고 무엇을 읽을지 모를 때. 이 묶음의 입구다.

Herdr 하나에서 Claude·Codex·agy를 함께 돌리는 방법을 정리한다. 에이전트마다 **역할을 나누는 것**이 이 문서 묶음의 중심이다.

## 1. 읽기 지도

필요한 파일만 읽는다. 한 단계에서 보통 1~2개면 된다.

| 지금 할 일 | 읽는 사람 | 읽을 파일 |
| --- | --- | --- |
| Herdr를 처음 써 본다 | 사람 | 이 파일 → [PLANNING.md](PLANNING.md) 3절 "구성안 작성과 승인" |
| 구성안을 쓴다 | leader | [PLANNING.md](PLANNING.md), [TEMPLATE-SETUP.md](TEMPLATE-SETUP.md) |
| 승인 후 구축한다 | leader | [BUILDING.md](BUILDING.md) |
| 하위 에이전트를 운영한다 | leader | [OPERATING.md](OPERATING.md), [TEMPLATE-DECISIONS.md](TEMPLATE-DECISIONS.md) |
| 작업을 정리한다 | leader | [OPERATING.md](OPERATING.md) 5절 "정리 절차" |

| 파일 | 담은 것 |
| --- | --- |
| README.md | 읽기 지도, 기준 환경과 모델, Herdr 기본 개념, 핵심 원칙 |
| PLANNING.md | 진입 에이전트, 구성안 작성과 승인, 토큰 예산, 역할 카탈로그, 작업 유형별 구성 |
| BUILDING.md | 탭·패널 배치, 구축 명령, runner, 여러 워크스페이스, 명령어 참조 |
| OPERATING.md | 대리 응답과 기록, blocked 대응, 문제 해결, 정리 절차 |
| TEMPLATE-SETUP.md | HERDR-SETUP.md 구성안 템플릿과 작성 예시 |
| TEMPLATE-DECISIONS.md | HERDR-DECISIONS.md 기록 규칙, 템플릿과 작성 예시 |

| 항목 | 내용 |
| --- | --- |
| 대상 | Herdr를 처음 쓰는 사람 |
| 출발점 | Herdr를 실행하고, 패널 하나에서 에이전트 하나만 띄운 상태 |
| 목표 | 그 에이전트에게 구성안을 받아 승인하고, 탭·패널·에이전트를 구축해 일을 나눠 맡긴다 |

## 2. 기준 환경과 모델

- Herdr 0.9.5, Windows 11, 터미널 130×37
- Claude Code (Max 요금제) — Opus 5.5 / Sonnet 5.5
- Codex (기본 요금제) — `gpt-6.1-sol` / `gpt-6.1-terra`
- agy = Antigravity CLI 1.4.11 (Google AI Pro) — **Gemini 3.8 Flash만 쓴다**

여러 워크스페이스를 동시에 돌리는 것을 전제로 한다. 그래서 **구현은 싼 모델, 판단은 상위 모델**로 나눈다.

| 에이전트 | 모델 | 맡는 역할 | 고르는 기준 |
| --- | --- | --- | --- |
| claude | Opus 5.5 | leader, reviewer, scouter (교차 조사, [PLANNING.md](PLANNING.md) 6.6) | 상위 모델. 조율과 검토처럼 판단이 필요한 곳에만 |
| claude | Sonnet 5.5 | coder, writer | 구현·작성의 기본 모델 |
| codex | `gpt-6.1-sol` | reviewer, analyst | 상위 모델. 검토·해석에만 |
| codex | `gpt-6.1-terra` | coder, checker | 구현의 대안 모델, 단순 확인·대조 |
| agy | Gemini 3.8 Flash | scouter | 요약·조사. effort는 모델 ID(`-low`·`-medium`·`-high`)로 고른다 |

effort와 역할별 배치의 기준은 [PLANNING.md](PLANNING.md) 4·5절이다.

## 3. Herdr 기본 개념

### 3.1 구조

```
Herdr 세션
└─ 워크스페이스 (w8)          보통 프로젝트 하나
   ├─ 탭 (w8:t1)              화면 하나
   │  ├─ 패널 (w8:p1)         터미널 하나 — 셸, 에이전트, 테스트 등
   │  └─ 패널 (w8:p2)
   └─ 탭 (w8:t2)
```

- ID는 Herdr가 정한다. 닫힌 탭·패널의 ID는 다시 쓰이지 않는다. 그래서 `w8:p4` 다음에 만든 패널이 `w8:p2`가 아닐 수 있다.
- 패널 ID보다 **에이전트 이름**(`writer`, `reviewer` 등)으로 부르는 것이 안전하다. 이름은 패널이 바뀌어도 그 에이전트를 따라간다.

### 3.2 에이전트 상태

Herdr는 패널 안의 에이전트를 알아보고 상태를 붙인다.

| 상태 | 뜻 | leader가 할 일 |
| --- | --- | --- |
| `idle` | 입력을 기다림 | 다음 지시를 보낼 수 있다 |
| `working` | 일하는 중 | 기다린다 |
| `blocked` | 승인·질문 화면에서 멈춤 | 화면을 읽고 대리 응답하거나 사람에게 올린다 ([OPERATING.md](OPERATING.md) 1절) |
| `done` | 일을 끝냄 | 결과를 읽는다 |
| `unknown` | 상태를 판단하지 못함 | 화면을 직접 읽어 확인한다. 끝났다는 뜻이 아니다 |

### 3.3 에이전트와 일반 셸의 구분

- **에이전트 패널**: Claude·Codex·agy가 떠 있는 패널. `herdr agent ...`로 다룬다.
- **일반 셸 패널**: 테스트, 서버, 학습 스크립트를 돌리는 패널. `herdr pane ...`로 다룬다. 토큰을 쓰지 않는다. 이 패널의 역할 이름이 `runner`다.

오래 걸리는 실행(학습, 서버)은 에이전트에게 시키지 말고 runner에서 돌린다. 에이전트가 출력을 지켜보는 동안 토큰이 계속 들기 때문이다. 짧은 명령(`git status`, 테스트 1회 등)은 에이전트가 자기 안에서 직접 실행한다. runner를 언제 어떻게 쓰는지는 [BUILDING.md](BUILDING.md) 5절에 있다.

## 4. 핵심 원칙

1. **사람은 leader하고만 대화한다.** 사람은 구성안을 검토·승인하고, 패널 배치·에이전트 실행·지시 전달은 승인된 구성안대로 leader가 `herdr` 명령으로 한다.
2. **leader는 환경을 임의로 구축하지 않는다.** 구성안(`docs/HERDR-SETUP.md`)을 먼저 쓰고 승인받는다.
3. **leader는 하위 에이전트에게 사람 대신 답할 수 있지만, 근거를 반드시 `docs/HERDR-DECISIONS.md`에 남긴다.**
4. **구현은 싼 모델, 판단은 상위 모델.** 검토는 작성자와 다른 회사의 상위 모델이 한다.
5. **오래 걸리는 실행은 runner에서.** 에이전트가 출력을 지켜보게 하지 않는다.
