---
tags: [herdr, multi-agent, guide]
created: 2026-10-30
updated: 2026-10-30
---

# TEMPLATE-SETUP — HERDR-SETUP.md 구성안 템플릿

> 이 파일을 읽는 때: leader가 구성안(`docs/HERDR-SETUP.md`)을 쓸 때. 작성 예시(2절)는 처음 한 번, 또는 막힐 때만 읽는다.

입구는 [README.md](README.md)다.

leader가 구성안을 쓸 때 쓰는 틀이다. 프로젝트 폴더의 `docs/HERDR-SETUP.md`로 저장한다. 항목의 뜻은 [PLANNING.md](PLANNING.md) 3.2에 있다.

## 1. 빈 템플릿

`<...>`를 채우고, 해당 없는 항목은 지우지 말고 "없음"이라고 적는다. 빠뜨린 것인지 필요 없는 것인지 구분하기 위해서다.

````markdown
# HERDR-SETUP — <프로젝트> 멀티 에이전트 구성안

- 상태: 초안
- 작성: <leader 이름> (YYYY-MM-DD)
- 가이드: D:\projects\docs\guides\herdr\README.md

## 1. 근거

- INTENT.md: "<해당 부분 인용>" (없으면 "INTENT.md 없음")
- 사용자 지시: "<지시 인용>"
- 사용자에게 확인한 것: <질문과 답, 없으면 "없음">

## 2. 작업 유형

- 바탕 구성: 가이드 PLANNING.md <6.x> <유형>
- 페어: Claude 주력 | Codex 주력
- 바꾼 점: <가이드 구성에서 달라진 점과 이유, 없으면 "없음">

## 3. 탭·패널 배치

| 탭 | 이름 | 패널 |
| --- | --- | --- |
| 1 | control | <leader> |
| 2 | agents | <왼쪽>, <오른쪽 위>, <오른쪽 아래> |
| 3 | run | <runner 목록, 없으면 "없음"> |

```
<배치 그림>
```

## 4. 에이전트

| 이름 | kind | 모델 · effort | 위치 | 역할 | 파일 수정 범위 |
| --- | --- | --- | --- | --- | --- |
| <프로젝트>-leader | claude | Opus 5.5 · medium | 1 control | 소통·조율 | docs/HERDR-SETUP.md, docs/HERDR-DECISIONS.md |
| <프로젝트>-<역할> | <kind> | <모델 · effort> | <탭 · 위치> | <역할> | <경로 또는 "안 함"> |

## 5. runner

| 이름 | 용도 | 주인 | 작업 폴더 |
| --- | --- | --- | --- |
| <프로젝트>-runner-<용도> | <용도> | <에이전트 이름> | <경로> |

## 6. 토큰 부담

| 구독 | 이 워크스페이스 | 동시에 도는 다른 워크스페이스 |
| --- | --- | --- |
| Claude | <●◐○ 와 역할> | <워크스페이스 이름과 부담, 없으면 "없음"> |
| Codex | | |
| agy | | |

## 7. 대리 응답 범위

- 가이드 OPERATING.md 1절 기준을 따른다
- 이 프로젝트에서 따로 정한 것: <예: "의존성 추가는 사용자에게 올린다", 없으면 "없음">

## 8. 정리 방법

- 닫을 탭·패널: <목록>
- 지울 worktree: <목록, 없으면 "없음">
- 남길 것: <예: runner-server 는 사용자가 닫을 때까지 유지>

## 9. 변경 이력

| 날짜 | 내용 | 승인 |
| --- | --- | --- |
| YYYY-MM-DD | 초안 작성 | — |
````

## 2. 작성 예시

youtube-downloader에 기능을 추가하는 경우다. visionlab 워크스페이스가 Claude 주력 페어로 이미 돌고 있다고 가정해, 부담을 나누려고 Codex 주력 페어를 골랐다 ([BUILDING.md](BUILDING.md) 6.3). 인용문과 날짜는 예시다.

````markdown
# HERDR-SETUP — youtube-downloader 멀티 에이전트 구성안

- 상태: 승인됨 (2026-11-02)
- 작성: ytdl-leader (2026-11-02)
- 가이드: D:\projects\docs\guides\herdr\README.md

## 1. 근거

- INTENT.md: "재생목록 단위 다운로드를 큐로 관리하는 데스크톱 앱"
- 사용자 지시: "다운로드 큐에 일시정지·재개 기능을 추가해줘. 테스트도 같이."
- 사용자에게 확인한 것: reviewer 를 둘지 → "둬. diff 리뷰 1회만."

## 2. 작업 유형

- 바탕 구성: 가이드 PLANNING.md 6.3 B. 앱·CLI 개발
- 페어: Codex 주력 (visionlab 이 Claude 주력으로 돌고 있어 부담 분산)
- 바꾼 점: scouter 없음 — 기존 코드가 작고 외부 라이브러리 조사가 필요 없음

## 3. 탭·패널 배치

| 탭 | 이름 | 패널 |
| --- | --- | --- |
| 1 | control | ytdl-leader |
| 2 | agents | ytdl-coder (왼쪽), ytdl-reviewer (오른쪽) |
| 3 | run | ytdl-runner-test (위), ytdl-runner-app (아래) |

```
탭 2 agents                                       탭 3 run
┌──────────────────────┬──────────────────────┐  ┌──────────────────────┐
│ ytdl-coder           │ ytdl-reviewer        │  │ ytdl-runner-test     │
│                      │                      │  ├──────────────────────┤
│                      │                      │  │ ytdl-runner-app      │
└──────────────────────┴──────────────────────┘  └──────────────────────┘
```

## 4. 에이전트

| 이름 | kind | 모델 · effort | 위치 | 역할 | 파일 수정 범위 |
| --- | --- | --- | --- | --- | --- |
| ytdl-leader | claude | Opus 5.5 · medium | 1 control | 소통·조율 | docs/HERDR-SETUP.md, docs/HERDR-DECISIONS.md |
| ytdl-coder | codex | gpt-6.1-terra · high | 2 agents 왼쪽 | 구현, 테스트 작성 | src/, tests/ |
| ytdl-reviewer | claude | Opus 5.5 · medium | 2 agents 오른쪽 | diff 적대적 리뷰 1회 | 안 함 |

## 5. runner

| 이름 | 용도 | 주인 | 작업 폴더 |
| --- | --- | --- | --- |
| ytdl-runner-test | pytest 실행 | ytdl-leader | D:/projects/youtube-downloader |
| ytdl-runner-app | 앱 실행 확인 | ytdl-leader | D:/projects/youtube-downloader |

## 6. 토큰 부담

| 구독 | 이 워크스페이스 | 동시에 도는 다른 워크스페이스 |
| --- | --- | --- |
| Claude | ◐ leader, ○ reviewer 1회 | visionlab: leader + coder(Sonnet) |
| Codex | ● coder | visionlab: reviewer(sol) 필요시 |
| agy | 없음 | 없음 |

## 7. 대리 응답 범위

- 가이드 OPERATING.md 1절 기준을 따른다
- 이 프로젝트에서 따로 정한 것: 새 패키지 설치(pip install)는 사용자에게 올린다

## 8. 정리 방법

- 닫을 탭·패널: 탭 2 agents, 탭 3 run
- 지울 worktree: 없음
- 남길 것: 탭 1 control (ytdl-leader)

## 9. 변경 이력

| 날짜 | 내용 | 승인 |
| --- | --- | --- |
| 2026-11-02 | 초안 작성 | — |
| 2026-11-02 | reviewer 추가 (사용자 확인 반영) | 사용자 승인 |
````

