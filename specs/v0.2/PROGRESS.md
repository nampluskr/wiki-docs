> 버전: v0.2 · 작성일: 2026-11-01 · 상태: 종료 (2026-11-01)
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

### Phase 4 — 전체 점검과 종료

상태: 대체됨 — Phase 5 이전 판정. 최종 판정은 아래 "Phase 4 최종 판정"

- **R-1~R-8과 이어받은 v0.1 요구 판정** (2026-11-01)

  | 요구 | 판정 | 근거 |
  | --- | --- | --- |
  | R-1 `wiki/<분야>/<소스>/`, 빈 폴더·apps 없음 | 충족 | `wiki/`에 `dev/`만, `wiki/dev/`에 `index.md`와 소스 폴더 3개(docs 4·web 1·youtube 1), 소스 폴더에 index 없음 |
  | R-2 소스와 위치 일치 | 충족 | youtube=[youtube], docs 4개=[docs], web=[docs, web] |
  | R-3 원본 폴더 이름, raw는 분야 아래 | 충족 | `raw/dev/` 5개 모두 `^[0-9]{6}-(youtube\|docs)-[a-z0-9-]+$`, `raw/` 바로 아래는 `dev/`뿐, inbox는 `.gitkeep`뿐 |
  | R-4 원본 파일 불변 | 충족 | Phase 2·3에서 옮긴 21개 모두 R100. 점검 스크립트가 `pXTD2DJBlDo.meta.json`을 수정 의심으로 걸었으나 이력이 추가 1회 + R100 이동 2회이고 처음과 지금의 차이가 0이라 오탐 |
  | R-5 소스별 index | 충족 | 소스별 제목 아래 링크와 폴더 목록이 셋 다 일치, `wiki/index.md`와 분야 목록 일치 |
  | R-6 링크·sources | 충족 | 상대 링크·`sources` 경로 깨짐 0, `[[` 없음 |
  | R-7 남은 원본 3개 정리 | 충족 | Phase 3 기록, inbox 비어 있음 |
  | R-8 WIKI.md·USECASES.md | 충족 | 옛 경로 형식 0건, D-2~D-7 규칙 문구 있음(Phase 1) |
  | 이어받은 v0.1 요구 (D-1) | 충족 | 모든 페이지에 frontmatter 다섯 항목·`status: draft`, URL마다 확인 날짜, `wiki/` 이름이 `[a-z0-9.-]`, inbox·raw에 영상·음성 파일 없음, 병합 커밋 없음, push는 모두 사용자 지시 뒤 |

  - 검증: R-1~R-6·R-8과 이어받은 요구는 스크립트로, R-7은 Phase 3 기록과 `inbox/` 목록으로 확인했다. 판정 직전 push(`162ad28`)로 로컬=원격.
  - 결과: R-1~R-8 충족, 미충족 없음.
  - 남은 판단: 버전 종료 여부. 사용자가 "아직 종료하지 않음"을 골랐다(2026-11-01). v0.2에서 더 할 일을 정한 뒤 종료한다.
  - 이후: 이 판정은 Phase 5 이전 구조 기준이다. Phase 5 뒤 다시 판정한다.

### Phase 5 — raw 소스 구분과 docs → web 통합

상태: 완료

- **raw를 분야/소스로, docs를 web으로** (2026-11-01)
  - 무엇을 했나: 사용자 지시 "raw 폴더도 분야\소스로 구분"으로 D-9를 쓰고 `raw/dev/{docs,youtube}/`로 옮기던 중, 사용자 지시 "docs 소스도 웹에서 가져온 정보를 AI 에이전트가 정리한 것입니다. docs 소스를 제거하고 web으로 변경. 즉, 소스는 web, youtube 2개입니다."가 와서 D-10을 쓰고 방향을 바꿨다. `git mv`로 docs 원본 4개를 `raw/dev/web/261101-web-…/`로(폴더 이름의 소스도 web으로), 유튜브 원본을 `raw/dev/youtube/`로, `wiki/dev/docs/` 페이지 4개를 `wiki/dev/web/`으로 옮겼다. 페이지의 `sources`·raw 링크·페이지끼리 링크(`../docs/` → 같은 폴더)를 고치고, 분야 index를 `## youtube`·`## web`으로 다시 썼다. WIKI.md(1·2·3·6·7절), USECASES.md(UC-4·UC-6·UC-7), INTENT 범위, DECISIONS(D-9·D-10, D-1 표, 개정된 D-2~D-6·D-8에 개정 표시), SPEC(R-1·R-2·R-3·R-5·R-7), PLAN(Phase 5 추가, Phase 2·3 완료 조건에 당시 기준 표시)을 고쳤다.
  - 결과: 소스는 `youtube`·`web` 두 개. `wiki/dev/web/` 5개, `wiki/dev/youtube/` 1개. `raw/dev/web/` 4개, `raw/dev/youtube/` 1개.
  - 검증: R-1 `wiki/dev/`에 index와 `web/`·`youtube/`만. R-2 web 5개=[web], youtube 1개=[youtube]. R-3 `raw/dev/`에 `web/`·`youtube/`만, 원본 폴더 이름의 소스가 소스 폴더와 같음, inbox는 `.gitkeep`뿐. R-4 원본 21개 모두 R100(페이지 4개는 링크 수정으로 R092~R098). R-5 index 제목 2개, 링크와 폴더 목록 일치. R-6 링크·`sources` 깨짐 0. WIKI.md·USECASES.md·`wiki/`에 `docs` 소스 잔여 0(외부 URL·`docs/HERDR-*` 같은 원본 속 경로 제외).
  - 특이사항: D-9를 먼저 `youtube·docs`로 썼다가 커밋 전에 `youtube·web`으로 고쳤다. "웹 정보 문서 원본은 raw/web에 둔다"가 v0.1 D-4("외부 검색 자료는 raw에 두지 않는다")와 부딪혀, 질문 중 웹 검색은 그대로 URL만 남긴다고 D-10에 나눠 적었다.

### Phase 4 재판정 (Phase 5 이후)

상태: 대체됨 — Phase 6 이전 판정. 최종 판정은 아래 "Phase 4 최종 판정"

- **R-1~R-8과 이어받은 v0.1 요구 재판정** (2026-11-01)

  | 요구 | 판정 | 근거 |
  | --- | --- | --- |
  | R-1 구조, 빈 폴더·apps 없음 | 충족 | `wiki/`에 `dev/`만, `wiki/dev/`에 `index.md`·`web/`(5)·`youtube/`(1), 소스 폴더에 index 없음 |
  | R-2 소스와 위치 일치 | 충족 | web 5개=[web], youtube 1개=[youtube] |
  | R-3 원본 폴더 이름, raw 분야/소스 | 충족 | `raw/`에 `dev/`만, `raw/dev/`에 `web/`·`youtube/`만, 원본 5개 이름의 소스가 소스 폴더와 같음, inbox는 `.gitkeep`뿐 |
  | R-4 원본 파일 불변 | 충족 | raw 21개 파일 모두 처음 추가된 blob 해시와 HEAD 해시가 같다. 첫 점검 스크립트가 README.md 2개를 "변경"으로 걸었으나, 같은 커밋의 다른 README.md와 비교한 오탐이었다 |
  | R-5 소스별 index | 충족 | `## youtube`·`## web` 아래 링크와 폴더 목록 일치, `wiki/index.md`와 분야 목록 일치 |
  | R-6 링크·sources | 충족 | 깨짐 0, `[[` 없음 |
  | R-7 남은 원본 정리 | 충족 | inbox 비어 있음, 세 원본이 `raw/dev/web/`에 있고 `wiki/dev/web/` 페이지가 가리킴 |
  | R-8 WIKI.md·USECASES.md | 충족 | 옛 경로 형식·`docs` 소스 잔여 0 |
  | 이어받은 v0.1 요구 | 충족 | frontmatter 다섯 항목·`status: draft`, URL 확인 날짜, 이름 규칙, 영상·음성 파일 없음, 병합 커밋 없음 |

  - 검증: 스크립트로 확인하고, R-4는 `git log --follow --raw`의 추가 시점 blob과 `git rev-parse HEAD:<파일>`을 21개 모두 비교했다. 판정 직전 push(`f9aacb8`)로 로컬=원격.
  - 결과: R-1~R-8 충족, 미충족 없음.
  - 남은 판단: 버전 종료 여부. 사용자가 "종료"를 골랐으나, 종료 기록 전에 유튜브 페이지 질문과 수정 지시가 와서 Phase 6을 했다. 이 판정은 Phase 6 이전 기준이다.

### Phase 6 — 유튜브 페이지 양식과 raw 폴더 규칙

상태: 완료

- **유튜브 페이지 영상별 섹션 (D-11)** (2026-11-01)
  - 무엇을 했나: 사용자가 "유튜브는 동영상 1개마다 1개의 문서인가"를 물어, 원본은 1:1이고 페이지는 주제 단위라고 답했다. 사용자가 "지금 규칙 그대로 적용"하고 "섹션별로 유튜브 등록일이 포함되고 최신 정보 순"을 지시했다. 섹션 단위("영상별 섹션"), 내용 충돌("둘 다 두고 최신에 표시"), 버전("v0.2에 추가")을 물어 D-11·R-9를 쓰고, WIKI.md 3절에 "유튜브 페이지 양식", 5·6절과 USECASES UC-4·UC-7에 영상 섹션 규칙을 넣었다. `wiki/dev/youtube/claude-skill-writing-rules.md`를 요약 → `## 점검 목록 (최신 기준)` → `## 2026-10-31 · 앤트로픽 피셜…`(채널·길이·URL·raw 링크, 기존 절을 `###`로) 순서로 재배치했다.
  - 결과: 유튜브 페이지가 영상별 섹션 양식이다.
  - 검증: R-9 스크립트로 섹션 날짜가 내림차순이고 `meta.json`의 `upload_date`(20261031)와 같음을 확인했다. `git diff --word-diff`로 바뀐 것이 제목 단계, 맨 위 요약 줄, 영상 정보 줄뿐이고 규칙 본문 문장은 그대로임을 확인했다.
  - 특이사항: 영상이 하나뿐이라 "영상과 다름" 표시와 섹션 정렬은 실제로 여러 영상에서 확인하지 못했다.

- **inbox·raw 폴더 규칙 (D-12)** (2026-11-01)
  - 무엇을 했나: 사용자 지시 "inbox: YYYYMMDD-{소스}-{title-in-english}, raw: raw\{소스}\YYYMMDD-{title-in-english}"로 원본을 `raw/{소스}/20261101-…`로 옮기다가, "날짜 접두사를 YYMMDD로 유지" 지시로 `raw/{소스}/261101-{title}/`로 고쳤다. raw는 분야로 나누지 않고, 폴더 이름에서 `-{소스}`를 뺐다(`raw/dev/web/261101-web-herdr/` → `raw/web/261101-herdr/` 등 5개). 페이지 `sources`·raw 링크, WIKI.md(1·2·3·6·7절), USECASES(UC-6·UC-7), DECISIONS(D-12, D-1 표, D-5·D-8·D-9 개정 표시), SPEC R-3, PLAN Phase 6, INTENT 범위를 고쳤다.
  - 결과: `raw/`에 `web/`(4)·`youtube/`(1)만 있다.
  - 검증: R-3 `raw/` 바로 아래가 `web/`·`youtube/`뿐이고 그 아래 이름이 `^[0-9]{6}-[a-z0-9-]+$`. R-4 커밋 전 `git diff --cached -M`에서 raw 21개 모두 R100. R-6 링크·`sources` 깨짐 0. R-8 WIKI.md·USECASES.md에 `raw/<분야>`·`raw/dev` 잔여 0(`261101-web-…`은 inbox 이름 예시라 맞음).
  - 특이사항: `YYYMMDD`는 오타로 보고, title은 kebab-case로 유지했다. 커밋된 이력 기준 blob 비교 스크립트는 커밋 전 새 경로를 따라가지 못해 21개를 모두 "다름"으로 냈고, rename 판정(R100)으로 대신 확인했다. 커밋 뒤 blob 비교로 다시 확인해 21개 모두 같았다.

### Phase 4 최종 판정 (Phase 6 이후)

상태: 완료 (2026-11-01)

- **R-1~R-9와 이어받은 v0.1 요구 판정** (2026-11-01)

  | 요구 | 판정 | 근거 |
  | --- | --- | --- |
  | R-1 구조 | 충족 | `wiki/`에 `dev/`·`index.md`만, `wiki/dev/`에 `index.md`·`web/`(5)·`youtube/`(1)만 |
  | R-2 소스와 위치 | 충족 | `youtube/` 페이지는 유튜브 출처만, `web/` 페이지는 유튜브만이 아님 |
  | R-3 폴더 이름 | 충족 | `raw/`에 `web/`·`youtube/`만, 그 아래 5개 모두 `^[0-9]{6}-[a-z0-9-]+$`, inbox는 `.gitkeep`뿐 |
  | R-4 원본 불변 | 충족 | raw 21개 모두 처음 추가된 blob과 HEAD blob이 같음 |
  | R-5 index | 충족 | `## youtube`·`## web` 아래 링크와 폴더 목록 일치 |
  | R-6 링크·sources | 충족 | 깨짐 0, `[[` 없음 |
  | R-7 남은 원본 정리 | 충족 | inbox 비어 있음, 세 원본이 `raw/web/`에 있고 `wiki/dev/web/` 페이지가 가리킴 |
  | R-8 WIKI.md·USECASES.md | 충족 | `raw/<분야>`·`raw/dev`·`youtube-<영상ID>` 잔여 0 |
  | R-9 유튜브 영상 섹션 | 충족 | 섹션 날짜 2026-10-31이 내림차순이고 `meta.json` `upload_date`(20261031)와 같음 |
  | 이어받은 v0.1 요구 | 충족 | frontmatter 다섯 항목, URL 확인 날짜, 영상·음성 파일 없음, 병합 커밋 없음 |

  - 검증: 스크립트로 확인했다. 판정 직전 push(`ad6d1cd`)로 로컬=원격.
  - 결과: R-1~R-9 충족, 미충족 없음.

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

> 아래 두 건은 v0.2 종료 뒤 사용자 지시("이 내용을 적용하고, v0.2 PROGRESS.md에 사용자 요청에 의한 작업 내용을 기록합니다")로 덧붙였다.

### 질문엔 위키 내용만으로 먼저 답하기 (2026-11-01)

- 요청: "claude 스킬에 대해 설명해줘"에 leader가 위키에 일부만 있다고 보고 바로 웹 검색을 하려 하자, 사용자가 막고 "위키에 저장된 내용으로만 먼저 대답하고, 이후에 사용자가 추가적인 내용을 요청하면 그 때 외부 검색해야 합니다", 이어서 "지침에도 반영할 것"을 지시했다.
- 조치: `WIKI.md` 4절 3·4단계를 "먼저 위키 내용만으로 답하고, 없는 부분은 '위키에 없음'으로 적고, 외부 검색은 추가 요청 때만"으로 고쳤다. `USECASES.md` UC-1에 같은 내용과 끝맺음 안내를, UC-2 시작 조건에 "위키에 전혀 없으면 외부 검색 여부를 묻는다"를 넣었다. 커밋 `7d92a88`.
- 결과: 질문 흐름이 "위키 답 → (사용자 추가 요청) → 외부 검색"이 됐다.
- 검증: 4절과 UC-1·UC-2를 다시 읽었다. 이후 다른 레포에서 wiki 스킬 시험(아래)에서 Claude가 위키 내용만으로 답했다.
- 상태: 완료.

### D:\projects 어디서든 위키 쓰기 — wiki 스킬 (2026-11-01)

- 요청: 위키 작성·사용·지침이 이 워크스페이스뿐 아니라 `D:\projects`와 그 아래 워크스페이스에서도 프롬프트로 동작하게 한다. 데스크탑·노트북에서 같은 내용으로 동작해야 하고(두 곳 모두 `D:\projects\wiki-docs`), 노트북 등록 방법은 레포를 클론하면 확인할 수 있어야 한다.
- 조치: 사용자가 방식(wiki 스킬 + 상위 AGENTS 한 줄), 대상(claude·codex·agy), 권한(wiki-docs만 쓰기 허용)을 골랐다. 레포 안에 `skills/wiki/SKILL.md`를 만들었다(WIKI.md 어느 절을 읽을지만 안내, 위키 먼저 답하기·확인 후 저장 두 줄). `WIKI.md`에 머리 한 줄과 8절 "다른 워크스페이스에서 쓰기"(새 컴퓨터 설정 PowerShell 절차, 프롬프트 예)를 넣고, 루트 `README.md`를 새로 만들어 클론 → 등록 프롬프트 → 확인 순서를 적었다. 이 데스크탑에는 8절대로 등록했다: `~/.claude/skills/wiki`·`~/.agents/skills/wiki` → `D:\projects\wiki-docs\skills\wiki` Junction, `~/.gemini/config/skills.json`(새로 만듦, `D:/projects/wiki-docs/skills`), `~/.codex/config.toml`의 `[sandbox_workspace_write] writable_roots`, `~/.claude/settings.json`의 `additionalDirectories`, `D:\projects\AGENTS.md`의 "## 위키".
- 결과: 레포 안 스킬·지침은 git으로 동기화되고, 컴퓨터마다 한 번 등록하면 pull만으로 갱신된다.
- 검증: Junction 두 개가 레포 폴더를 가리키고, `skills.json`이 BOM 없는 JSON이며, `settings.json`이 JSON으로 읽히는 것을 확인했다. 이 세션에 `wiki` 스킬이 바로 인식됐다. 다른 레포(`D:\projects\youtube-kit`)에서 `claude -p`로 "위키에서 herdr 하위 에이전트 재시작 방법"을 물으니 `wiki/dev/web/herdr-basics.md` 6절을 읽고 위키 내용만으로 답했고 권한 거부는 0건이었다.
- 특이사항: Git Bash에서 `claude -p "/wiki …"`로 보내면 `/wiki`가 `C:/Program Files/Git/wiki`로 바뀐다(경로 변환). 스킬 설명이 맞아 위키는 찾았다. youtube-kit 자체의 Stop hook(`progress-check.mjs`)이 답 뒤에 PROGRESS 기록을 계속 요구해 반복 출력이 났는데, wiki 스킬과는 관계없는 그 레포 설정이다. **Codex·agy 동작과 Codex `writable_roots`가 Windows `elevated` 샌드박스에서 적용되는지는 사용자가 시험을 멈춰 확인하지 않았다.** 노트북 등록은 사용자가 노트북에서 한다.
- 상태: 완료(Codex·agy 확인은 남음 → 아래 "Codex·agy에서 wiki 스킬 확인"에서 확인).

### Codex·agy에서 wiki 스킬 확인 (2026-11-01)

- 요청: agents 탭의 codex, agy를 재시작하고 wiki 스킬 테스트를 실행해 leader가 결과를 확인한다.
- 조치: `wd-scouter-codex`·`wd-scouter-agy`를 Ctrl+C로 종료하고(agy는 세 번째 Ctrl+C에서 종료) 같은 창·이름·설정으로 `agent start`했다. 두 scouter(작업 폴더 `D:\projects`, wiki-docs 밖)에 같은 읽기 전용 질문 "위키에서 herdr 하위 에이전트를 재시작하는 방법을 3줄로, 위키는 고치지 말고, 사용한 스킬과 읽은 위키 파일을 적어"를 보냈다. Codex는 `$wiki`, agy는 `/wiki`(Git Bash 경로 변환을 막으려 `MSYS_NO_PATHCONV=1`)로 시작했다.
- 결과:
  - Codex: 스킬 `wiki`, `wiki/dev/web/herdr-basics.md`를 읽고 위키 6절 내용(종료 → 셸 확인 → 같은 설정으로 `agent start`)으로 답했다. 승인 창 없음.
  - agy: 스킬 `wiki`, WIKI.md·index·`herdr-basics.md` 등 5개 파일을 읽고 같은 내용으로 답했다. 셸 명령 `git -C D:/projects/wiki-docs status --short`와 `git -C D:/projects/wiki-docs pull`에 승인 창이 떠서, leader가 읽기·동기화 명령이라 각각 "이번 한 번 허용"(1번)으로 대신 답했다. 설정에 저장하는 선택지는 고르지 않았다. 파일 읽기에는 승인 창이 없었다.
  - 세 에이전트(claude·codex·agy) 모두 다른 폴더에서 wiki 스킬로 위키 내용만으로 답했다.
- 검증: `herdr agent read`로 두 scouter의 답 전체를 읽었다. 시험 뒤 `git -C D:/projects/wiki-docs status --short`가 비어 있어 위키가 바뀌지 않았다.
- 특이사항: 쓰기 시험은 하지 않았다. 그래서 Codex `writable_roots`(Windows `elevated` 샌드박스)와 agy의 위키 파일 쓰기 승인 여부는 아직 모른다. agy는 셸 명령마다 승인을 물으므로, 승인을 줄이려면 ego lite처럼 `~/.gemini/antigravity-cli/settings.json`에 `command(regex:git -C D:/projects/wiki-docs .*)` 같은 허용 규칙을 넣는 방법이 있다(사람이 정할 일).

### Tailscale 키 만료 끄기 절차 보강 (2026-11-01)

- 요청: 데스크탑 원격 접속 환경을 위키 `wiki/dev/web/ssh-tailscale-remote-terminal.md` 순서대로 사용자가 직접 실행하던 중, 1단계 "키 만료 끄기"에서 사용자가 "홈페이지에 접속하니 영어로 표시됩니다"라고 했고, 이어 "키 만료 끄기 내용을 위키에 업데이트 해 주세요"라고 지시했다.
- 조치: 페이지 2절 "키 만료 끄기"를 하위 목록으로 나눠 관리 콘솔이 영어로만 나온다는 점, 기기 목록 바로가기 주소 `https://login.tailscale.com/admin/machines`, 행 맨 오른쪽 `⋯` → Disable key expiry, 확인 표시 "Expiry disabled"를 넣었다. `sources`에 이 기록을 더했다.
- 결과: 영어 화면에서 메뉴를 찾는 절차가 페이지에 생겼다.
- 검증: 페이지를 다시 읽어 frontmatter와 2절 목록 형식을 확인했다. **바로가기 주소와 "Expiry disabled" 표시는 leader가 안내한 내용이고, 사용자 화면에서 확인됐는지는 아직 보고받지 않았다.**
- 상태: 완료.

## 버전 종료

- 사용자 종료 판단: 종료 (2026-11-01). 사용자 지시 "모든 과정을 종료하고 v0.2 마무리". Phase 4 최종 판정 R-1~R-9 충족.
- 남은 것 (다음 버전으로 넘긴다):
  - 아직 실제로 일어나지 않은 규칙: 기존 페이지에 보태기(v0.1 R-10), 내용이 다를 때 묻기(UC-5), 유튜브 페이지에 웹 내용을 보태 `web/`으로 옮기기(D-3·D-10), 수식(v0.1 R-7), 영상 여러 개일 때의 섹션 정렬과 "영상과 다름" 표시(D-11).
  - 제외 범위로 둔 것: 큰 원본을 페이지로 나누는 기준, 마크다운이 아닌 파일(이미지), 여러 에이전트 분담.
  - herdr `agent send-keys`가 scouter 입력칸에 반영되지 않은 원인.
  - 페이지는 모두 `status: draft`다. `stable`은 사람이 지시할 때 바꾼다.
