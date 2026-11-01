> 버전: v0.2 · 작성일: 2026-11-01 · 상태: 진행 중
>
> 실제로 한 일과 검증 결과를 기록한다. 이번 버전의 의도는 `INTENT.md`에 있다. 지난 버전 기록은 `specs/v0.1/PROGRESS.md`에 있다.

# PROGRESS — wiki-docs v0.2

## Phase별 진행

### Phase 1 — 지침 갱신

상태: 완료

- **WIKI.md·USECASES.md 갱신** (2026-11-01)
  - 무엇을 했나: `WIKI.md`를 새 구조로 다시 썼다. 1절 폴더 구조(`wiki/<분야>/{youtube,docs,web}/`, `raw/<분야>/<YYMMDD-소스-title>/`, apps 없음), 2절 원본 폴더 이름 규칙(D-4·D-5), 3절에 "페이지를 둘 소스 폴더"(D-3)와 소스별 index 제목(D-6), 소스 폴더 사이·raw로의 링크 예시를 넣었다. 5절 저장과 6절 inbox 정리에 소스 폴더 고르기, 폴더 이름 바꾸기, 유튜브 폴더 이름(`YYMMDD-youtube-<title>`)을 반영했다. 근거 표기를 `v0.1 D-n`·`v0.2 D-n`으로 나눴다. `USECASES.md`의 UC-4·UC-6·UC-7 경로와 결정 번호를 새 규칙에 맞췄다.
  - 결과: 두 문서가 v0.2 D-2~D-7을 따른다.
  - 검증: 두 문서에서 옛 경로 형식(`youtube-<영상ID>`, `wiki/<분야>/<페이지>`, `raw/dev/herdr/`, v0.1 예시 `fft-notes`)을 grep해 0건, `WIKI.md`에서 D-2~D-7의 핵심 규칙 문구 6개가 모두 있는 것을 확인했다.
  - 특이사항: `WIKI.md` frontmatter 예시의 `raw/dev/261101-docs-herdr/PLANNING.md`는 Phase 2에서 옮긴 뒤에 생긴다.

### Phase 2 — 기존 페이지·원본 옮기기

상태: 완료

- **D-8 표대로 옮기기** (2026-11-01)
  - 무엇을 했나: `git mv`로 원본 2개(`raw/dev/herdr/` → `raw/dev/261101-docs-herdr/`, `raw/dev/youtube-pXTD2DJBlDo/` → `raw/dev/261101-youtube-claude-skill-writing-rules/`)와 페이지 3개(`herdr-basics` → `web/`, `herdr-multi-agent-roles` → `docs/`, `claude-skill-writing-rules` → `youtube/`)를 옮겼다. 페이지의 `sources` raw 경로, 페이지끼리의 링크(`../web/…`, `../docs/…`, `../youtube/…`), raw로의 링크(`../../../raw/…`)를 고쳤다. `wiki/dev/index.md`를 `## youtube`·`## docs`·`## web` 제목으로 다시 썼다.
  - 결과: `wiki/dev/`에 `index.md`와 소스 폴더 3개(각 페이지 1개), `raw/dev/`에 새 이름의 원본 2개.
  - 검증: R-1 `wiki/dev/` 바로 아래가 `index.md`와 소스 폴더뿐, 소스 폴더마다 페이지 1개. R-2 페이지별 `sources` 종류가 youtube=[youtube], docs=[docs], web=[docs, web]으로 폴더와 맞음. R-3 `raw/dev/` 폴더 이름 2개가 규칙에 맞음. R-4 `git diff --cached -M`에서 원본 10개 모두 R100(페이지 3개는 링크를 고쳐 R093~R097). R-5 소스별 제목 아래 링크와 폴더 목록이 셋 다 일치. R-6 상대 링크·`sources` 경로 깨짐 0.
  - 특이사항: inbox의 원본 3개는 아직 옛 이름이다(Phase 3에서 바꾼다). `specs/v0.1/` 문서 안의 옛 경로는 지난 버전이라 고치지 않았다.

### Phase 3 — 남은 원본 3개 정리

상태: 완료

- **rust-gui·ego-lite-guide·ssh-tailscale 정리 → dev/docs** (2026-11-01)
  - 무엇을 했나: D-8 이름으로 inbox 원본 3개를 `git mv`했다(`261101-docs-rust-gui`, `261101-docs-ego-lite-guide`, `261101-docs-ssh-tailscale-remote-terminal-guide`). leader가 세 원본(파일 11개)을 모두 읽고 `wiki/dev/docs/`에 페이지 3개를 썼다: `rust-gui-frameworks.md`(설치, UI 방식, 비교표, 고르는 방법, 프레임워크별 요점, 주의점), `ego-lite-browser-agents.md`(구조, Space·로그인, 설치, 에이전트별 설정, 요청 쓰는 법, 스크립트, 개인정보, 문제 해결), `ssh-tailscale-remote-terminal.md`(구성·순서, Tailscale, OpenSSH, 키, 공개키 등록, 비밀번호 차단, 안드로이드 앱, 접속, 전원, 보안 강화, 문제 해결). 분야 index의 `## docs` 아래에 세 줄을 넣고, 원본 3개를 `raw/dev/`로 옮겼다.
  - 결과: `inbox/`에 `.gitkeep`만 남았다. `wiki/dev/docs/` 페이지 4개, `raw/dev/` 원본 5개.
  - 검증: R-1 소스 폴더 3개 모두 페이지 있음. R-2 새 페이지 3개의 `sources`가 모두 docs 원본뿐이라 `docs/`에 맞음. R-3 `raw/dev/` 폴더 이름 5개가 모두 정규식에 맞음. R-4 이름 바꾸기·이동한 원본 11개 모두 R100. R-5 소스별 index와 폴더 목록 일치. R-6 링크·`sources` 깨짐 0, `[[` 없음.
  - 특이사항: 세 원본은 기존 페이지와 겹치는 주제가 없어 R-10(보태기)·UC-5(묻기)는 일어나지 않았다. SSH 페이지에 herdr 페이지로 가는 관련 링크를 넣었다가, 위키의 herdr 페이지에는 원격 머신 내용이 없어 뺐다. 원본의 실행 검증 범위(rust-gui는 설치만, ssh는 미시험, ego-lite는 첫 동작만)를 각 페이지 첫머리에 적었다.

## 사용자 요청 작업

### v0.2 버전 INTENT 작성 (2026-11-01)

- 요청: 위키를 `wiki/<분야>/<소스>/`로 나누고(예: `dev/youtube`), inbox·raw 폴더 이름을 `YYMMDD-{소스}-{title-in-kebab-case}`로 다시 정한다. v0.2 버전 INTENT부터 진행한다.
- 조치: 소스 종류(youtube·docs·web), 섞인 페이지 위치(web), 목적(구조 재정의 + 남은 원본 반영), 범위를 사람에게 물어 INTENT 초안을 쓰고 확정받았다. 그 직후 사용자가 앱 관련 원본 3개를 inbox에서 지웠다("앱은 독립된 레포로 관리되기 때문"). rust-gui는 남겨 dev로 정리하고, `apps` 분야를 없애고, 큰 원본 기준·이미지 처리를 제외 범위로 옮기기로 해서 INTENT를 같은 날 개정했다.
- 결과: `specs/v0.2/INTENT.md` 확정(개정). 사용자가 지운 원본 3개 폴더(102개 파일)의 삭제를 이 작업과 함께 커밋한다.
- 검증: `git status`로 지워진 파일이 rust-video-player·rust-video-player-native·zed-like-editor 아래뿐이고, inbox에 rust-gui·ego-lite-guide·ssh-tailscale이 남은 것을 확인했다.
- 상태: 완료. 다음은 `DECISIONS.md`.

### .gitkeep 삭제 (2026-11-01)

- 요청: 로컬에서 지워진 `raw/.gitkeep`, `specs/v0.1/.gitkeep`에 대해 사용자가 "파일이 존재하므로 gitkeep 불필요"라고 했다.
- 조치: 두 파일의 삭제를 커밋했다. `specs/v0.1/`은 지난 버전 폴더라 고치지 않는 것이 AGENTS 규칙이지만, 사람이 정한 것이고 문서 내용은 바뀌지 않는다. `inbox/.gitkeep`은 남겼다.
- 결과: `raw/`(dev/ 있음), `specs/v0.1/`(문서 5개 있음)은 파일이 있어 폴더가 그대로 남는다.
- 검증: `git status`로 두 파일 삭제만 남은 것을 확인했다.
- 상태: 완료.

### v0.2 DECISIONS 확정, SPEC·PLAN 초안 (2026-11-01)

- 요청: push 후 DECISIONS 진행. 확정하면서 SPEC·PLAN도 같이 쓴다.
- 조치: 날짜(inbox에 넣은 날), raw 위치(`raw/<분야>/<이름>/`), title(넣는 사람이, 없으면 에이전트), 유튜브 title(영상 제목 영문 요약), 나중에 섞일 때(web으로 옮김), index(분야 index 하나에 소스별 제목), 지금 원본·페이지의 새 이름(제안대로)을 사람에게 물어 D-1~D-8로 쓰고 확정받았다. SPEC R-1~R-8, PLAN Phase 1~4 초안을 썼다.
- 결과: `specs/v0.2/DECISIONS.md` 확정, `SPEC.md`·`PLAN.md` 초안.
- 검증: PLAN이 R-1~R-8을 모두 대응하는지, SPEC·PLAN의 v0.2 D-n이 DECISIONS에 있는지 확인했다(PLAN의 D-11·D-14는 v0.1 결정).
- 상태: 완료. SPEC·PLAN 확정 (2026-11-01).

## 버전 종료

- 사용자 종료 판단: 진행 중.
- 남은 것: DECISIONS → SPEC → PLAN → Phase 진행.
