---
tags: [herdr, multi-agent, guide]
created: 2026-10-30
updated: 2026-10-30
---

# OPERATING — 운영과 정리

> 이 파일을 읽는 때: 구축 뒤 하위 에이전트가 질문하거나 멈췄을 때, 문제가 생겼을 때, 작업을 정리할 때.

입구는 [README.md](README.md)다. 기록 템플릿은 [TEMPLATE-DECISIONS.md](TEMPLATE-DECISIONS.md)에 있다.

## 1. 대리 응답과 근거 기록

하위 에이전트가 질문하거나 `blocked`(승인·선택 요청)가 되면 **leader가 사람 대신 답한다.** 답할 때마다 **반드시 근거를 문서로 남긴다.**

### 1.1 대리 응답 범위

| leader가 답한다 | 사람에게 올린다 |
| --- | --- |
| 사람의 기존 지시나 문서로 답이 정해지는 질문 | 요구 범위를 넓히거나 줄이는 결정 |
| 작업 폴더 안의 읽기·쓰기·테스트 실행 승인 | 파일·브랜치 삭제, 되돌리기 어려운 작업 |
| 이미 정한 방향 안에서의 세부 선택 (이름, 순서 등) | 외부 전송 (push, 메일, 업로드), 비용이 드는 작업 |
| | 승인된 구성안과 달라지는 일 ([PLANNING.md](PLANNING.md) 3.4) |
| | 근거를 댈 수 없는 판단 |

**근거를 댈 수 없으면 답하지 않고 올린다.** 구성안(`docs/HERDR-SETUP.md`)의 "대리 응답 범위"에 프로젝트별로 따로 정한 것이 있으면 그것도 따른다.

### 1.2 기록

기록 위치: 작업 폴더의 `docs/HERDR-DECISIONS.md`. leader만 쓰고, 하위 에이전트는 고치지 않는다. 한 건마다 번호를 붙여 덧붙인다. 대리 응답뿐 아니라 **사람에게 올린 건과 구성 안의 조치(에이전트 재시작 등)도 함께 기록**한다. 쓰는 규칙·템플릿·작성 예시는 [TEMPLATE-DECISIONS.md](TEMPLATE-DECISIONS.md)에 있다.

```markdown
### D-001 · 2026-10-30 14:20 · writer

- 상황: 질문
- 질문: 예시 명령에 패널 ID를 실제 값으로 넣을까, 자리표시자로 둘까?
- 구분: 대리 응답
- 답변: 자리표시자(<새 패널 ID>)로 둔다.
- 근거: 사용자 지시 "초보자용 실용 매뉴얼" — ID는 실행 때마다 달라진다 (가이드 README.md 3.1).
- 보고: 미보고
```

작업을 마치고 사람에게 보고할 때 **대리 응답 목록을 함께 보고한다.**

## 2. blocked / unknown 대응

1. 화면을 먼저 읽는다: `herdr agent read <이름> --source recent-unwrapped --lines 80`
2. 무엇을 묻는지 확인한다
3. 1절 기준에 따라 대리 응답하고 `docs/HERDR-DECISIONS.md`에 기록하거나, 사람에게 올린다
4. 승인 화면은 화면에 안내된 키로 답한다: 예) `herdr agent send-keys <이름> esc`

`unknown`은 끝났다는 뜻이 아니다. 화면을 읽어 직접 확인한다.

## 3. 자주 겪는 문제

| 증상 | 원인 | 대응 |
| --- | --- | --- |
| `agent prompt`가 `timeout`·`agent_prompt_stalled` | 지시는 갔을 수 있다 | **같은 지시를 다시 보내지 않는다.** 먼저 `agent read`로 확인 |
| `agent_blocked` 오류 | 승인 화면이 떠 있음 | 2절 절차 |
| `agent start` 실패 | 패널이 빈 셸 프롬프트가 아님 | 패널 화면을 읽고 실행 중인 것을 확인 |
| 사람 화면이 갑자기 바뀜 | `--no-focus`를 빠뜨림 | 패널·탭을 만들 때 항상 `--no-focus` |
| agy 업데이트 실패 | 실행 중인 agy가 파일을 잠금 | agy 패널을 끝낸 뒤 `agy update` |

## 4. 하지 않을 것

- leader가 만들지 않은 탭·패널은 닫지 않는다. 사람이 시킬 때만 닫는다.
- `herdr server stop`은 실행하지 않는다. 세션의 모든 패널이 같이 꺼진다.
- bare `herdr`는 에이전트 안에서 실행하지 않는다. TUI를 띄운다.
- 하위 에이전트 화면을 사람이 직접 건드리지 않는다. 필요하면 leader에게 말한다.

## 5. 정리 절차

작업이 끝나면 사람이 leader에게 말한다.

```
작업 끝났어. 네가 만든 탭과 패널을 정리하고, 대리 응답 목록을 보고해줘.
```

leader가 하는 일
1. 하위 에이전트가 `idle`·`done`인지 확인한다 (`herdr agent list`)
2. runner에서 아직 돌고 있는 프로세스를 `herdr pane send-keys <runner ID> ctrl+c`로 끝낸다
3. 자기가 만든 패널·탭만 닫는다: `herdr pane close <패널 ID>`, `herdr tab close <탭 ID>`
4. worktree를 만들었다면 사람에게 확인받고 지운다: `herdr worktree remove --workspace <ID>`
5. `docs/HERDR-DECISIONS.md`의 이번 작업 기록을 요약해 보고한다
6. `docs/HERDR-SETUP.md`의 상태를 `종료 (YYYY-MM-DD)`로 바꾼다
