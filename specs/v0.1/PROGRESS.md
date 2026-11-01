> 버전: v0.1 · 작성일: 2026-11-01 · 상태: 진행 중
>
> 실제로 한 일과 검증 결과를 기록한다. 요구는 `SPEC.md`에, 변하지 않는 의도는 루트 `INTENT.md`에 있다.

# PROGRESS — wiki-docs v0.1

## Phase별 진행

없음.

## 사용자 요청 작업

### herdr scout-trio 환경 구성 (2026-11-01)

- 요청: `D:\projects\HERDR.md`를 읽고 서브에이전트 3개로 이뤄진 herdr 환경을 만든다.
- 조치: 워크스페이스 w7에 control·agents·runner 탭을 만들고 leader와 scouter 3개(claude·codex·agy)를 띄웠다. 이름이 w6의 기존 scouter와 겹쳐 사용자 지시로 `wd-` 접두사를 붙였다(`wd-leader`, `wd-scouter-claude`·`-codex`·`-agy`). 사용자 요청으로 w6의 scouter 3개 창을 닫았다. `D:\projects\HERDR.md`에 워크스페이스 접두사 규칙(워크스페이스 이름 각 단어의 첫 글자)을 넣었다.
- 결과: w7에 탭 3개와 `wd-*` 에이전트 4개가 있다. HERDR.md 1·3·5절이 접두사 규칙을 따른다.
- 검증: `herdr tab list`로 탭 3개, `herdr agent list`로 scouter 셋 idle/done, `herdr pane layout`으로 agents 탭 너비 35/35/34를 확인했다. 접두사 계산 코드를 w6(`projects`→`p`)·w7(`wiki-docs`→`wd`, 중복 감지)과 예시 `abc-def-hij`→`adh`로 실행해 확인했다.
- 상태: 완료.

### v0.1 SPEC 초안 작성 (2026-11-01)

- 요청: 현재 워크스페이스(wiki-docs)의 문서 작업을 시작한다. 첫 작업으로 v0.1 스펙을 대화로 정하고, leader가 혼자 진행한다.
- 조치: 위키 구조·이름 규칙·원본 보관·frontmatter·링크·목차·수식·inbox 정리 시점·동기화를 사람에게 물어 정하고 `specs/v0.1/SPEC.md`에 R-1~R-9로 썼다. 분야는 미리 정하지 않고 필요할 때 폴더를 추가한다는 사용자 지시를 R-1에 반영했다.
- 결과: `specs/v0.1/SPEC.md` 초안이 있다. `wiki/`·`inbox/`·`raw/` 폴더는 아직 만들지 않았다.
- 검증: 요구마다 출처가 사람의 답이나 INTENT·AGENTS 문장인지 다시 읽어 확인했다.
- 상태: 보류. 사용자가 "INTENT.md부터 처음 시작"하라고 해서, 워크플로 순서(버전 INTENT → DECISIONS → SPEC)를 건너뛴 초안으로 보고 참고용으로 남겼다.

### 루트 INTENT 다시 정하기 (2026-11-01)

- 요청: 이 프로젝트의 INTENT.md부터 작업을 처음 시작한다(사용자가 버전 INTENT가 아닌 루트 INTENT를 골랐다).
- 조치: 절마다 사람에게 물었다. "무엇을 하는가"에서 고정 분야 목록 문장을 지웠다. "변하지 않는 원칙"에서 "GitHub 웹에서 inbox로 파일을 바로 올릴 수도 있다"를 지웠다(버전 문서에서 다루기로 함). "왜 하는가"·"다루지 않는 것"은 그대로 두었다.
- 결과: 루트 `INTENT.md`에 폴더 이름·분야 목록이 없다.
- 검증: 고친 INTENT를 DOCUMENTS 3절 기준(기술·폴더 구조를 담지 않음)으로 다시 읽었다.
- 상태: 완료. 다음은 `specs/v0.1/INTENT.md`(버전 INTENT).

### 루트 INTENT에 위키 사용 방식 반영 (2026-11-01)

- 요청: 사용 목적·방법을 알려 줌 — 질문하면 1차 위키 검색, 2차 외부 검색, 3차 위키에 정리해 저장. 새로 조사한 내용은 위키 추가를 문의하거나 사용자가 지시.
- 조치: 저장 시점("물어본 뒤 저장"), 원본 원칙("외부 자료도 출처를 남겨 허용"), 위치("루트 INTENT")를 사람에게 물어 정했다. "무엇을 하는가"에 사용 방식 두 문장을, "변하지 않는 원칙"에 위키 우선 검색·확인 후 추가 원칙을 넣고 원본 원칙을 고쳤다.
- 결과: 루트 `INTENT.md`에 질문 → 위키 → 외부 검색 → 확인 후 저장 흐름이 있다. 검색 도구·저장 절차 같은 구체 방법은 버전 문서로 넘겼다.
- 검증: 고친 INTENT를 다시 읽어 사람이 고른 답만 들어갔는지 확인했다.
- 상태: 완료.

### v0.1 버전 INTENT 작성 (2026-11-01)

- 요청: v0.1 버전 INTENT 작업 시작.
- 조치: 목적("흐름이 한 번 돌아가게"), 범위 4개, 제외 범위 4개를 사람에게 물어 정했다. 배경은 대화 내용으로 초안을 보여 주고 사용자가 "그대로 적어 주세요"라고 해서 그대로 적었다.
- 결과: `specs/v0.1/INTENT.md` 초안이 있다.
- 검증: 템플릿(project-workflow-compact `docs/templates/INTENT.md`)의 절 구성과 같은지, 사람이 고른 답만 들어갔는지 다시 읽어 확인했다.
- 상태: 확정 (2026-11-01).

### v0.1 DECISIONS 작성과 SPEC 동시 개정 (2026-11-01)

- 요청: 버전 INTENT를 확정하고 DECISIONS 작업 시작. 기존 SPEC.md 참조 또는 동시 작업은 사용자에게 묻는다.
- 조치: 보류했던 SPEC 초안의 답 9가지를 사용자 지시("그대로 D-n으로 옮기기")로 옮겼다. 외부 원본(URL만), 지침 위치(WIKI.md + AGENTS.md 한 줄), 위키 검색(index → 본문), 답 형식(출처 구분·확인 후 저장), 기존 페이지(보태기·다르면 묻기), 새 페이지 status(draft)를 새로 물어 D-1~D-14로 썼다. 사용자 답("동시에")대로 SPEC.md를 R-1~R-13으로 다시 써 출처를 D-n으로 바꿨다.
- 결과: `specs/v0.1/DECISIONS.md` 초안(D-1~D-14, 미결 3개), `specs/v0.1/SPEC.md` 초안(R-1~R-13)이 있다.
- 검증: 결정마다 근거가 사용자 답이나 INTENT·AGENTS 문장인지, SPEC의 출처 D-n이 DECISIONS에 있는지 다시 읽어 확인했다.
- 상태: 완료. 이후 "미결 사항 결정, DECISIONS·SPEC 확정" 작업에서 두 문서를 확정했다.

### inbox 원본 종류(마크다운 폴더·유튜브) 반영 (2026-11-01)

- 요청: inbox에는 사람이 넣거나 사람의 요청으로 에이전트가 넣는다. 원본은 마크다운 문서 폴더, 유튜브 영상 내용(yt-dlp 자막 또는 ego-lite의 YouTube 요약). `C:\winpython` 환경과 상위 프로젝트의 yt-dlp 스크립트를 확인할 것.
- 조치: `C:\winpython`의 cpu·gpu 환경과 `D:\projects`의 yt-dlp 사용 코드를 확인했다. 유튜브 받을 것(요청할 때 고름), raw 이름(폴더는 영어, 파일 이름은 한국어·영어), 영상 원본 문장 보강, v0.1 범위 포함을 사람에게 물었다. 루트 INTENT 다루지 않는 것에 한 줄, 버전 INTENT 범위 항목, DECISIONS D-2·D-3 개정과 D-15 추가, SPEC R-2·R-3 개정과 R-14 추가를 했다.
- 결과: 자막은 cpu 환경의 youtube-kit, 요약은 ego-lite로 정했다. 유튜브 inbox 파일 모양은 미결로 남겼다.
- 검증: cpu 환경 `python -m yt_dlp --version` → 2026.08.19, `import youtube_kit` → `D:\projects\youtube-kit\src`, `Scripts\youtube-kit.exe` 있음. gpu 환경은 `No module named yt_dlp`. ego-lite 0.6.2.5 설치 폴더 확인. youtube-kit README에서 `info`·`subtitle` 명령을 확인했다. 실제 자막 다운로드와 ego-lite 요약은 아직 해 보지 않았다.
- 상태: 완료.

### 미결 사항 결정, DECISIONS·SPEC 확정, USECASES 작성 (2026-11-01)

- 요청: 미결 사항을 지금 정하고 확정한 뒤, wiki 사용 시나리오(유스케이스)를 문서로 저장.
- 조치: 유튜브 원본 모양(영상마다 폴더), sources 표기(문자열 목록), 외부 검색 도구(에이전트 기본 검색), 시나리오 문서 위치(루트 USECASES.md)를 사람에게 물어 D-16~D-19로 썼다. youtube-kit README의 파일 규칙에 맞춰 자막 파일 이름을 `<영상ID>.<종류>.<키>.srt`로 적었다. SPEC R-4·R-8·R-14에 반영하고 DECISIONS·SPEC을 확정했다. 루트 `USECASES.md`에 UC-1~UC-9를 썼다.
- 결과: DECISIONS D-1~D-19(미결: 흐름 확인용 질문 1개, PLAN에서 정함), SPEC R-1~R-14 확정. `USECASES.md` 작성.
- 검증: USECASES의 단계마다 해당 D-n이 있는지, SPEC 출처 D-n이 DECISIONS에 있는지 다시 읽어 확인했다.
- 상태: 완료. 다음은 PLAN.

### v0.1 PLAN 작성 (2026-11-01)

- 요청: 다음 진행(PLAN).
- 조치: Phase 4단계(뼈대·지침 → 질문 흐름 → inbox 원본 정리 → 전체 점검·동기화)를 제안해 사람이 골랐다. 흐름 확인용 질문과 Phase 3 원본은 Phase를 시작할 때 사람이 주기로 했다. `.serena/`는 `.gitignore`에 넣고 지금까지의 문서를 커밋하기로 했다.
- 결과: `specs/v0.1/PLAN.md` 확정. `.gitignore` 추가.
- 검증: R-1~R-14가 모두 어느 Phase의 대응에 있는지 확인했다.
- 상태: 완료. 다음은 Phase 1.

### GitHub public 저장소 생성과 push (2026-11-01)

- 요청: GitHub에 public으로 저장소를 만들고 커밋·푸시한다.
- 조치: 추적 파일에 토큰·비밀번호·이메일 같은 문자열이 없는지 확인한 뒤 `gh repo create nampluskr/wiki-docs --public --source . --remote origin --push`로 만들고 main을 push했다.
- 결과: https://github.com/nampluskr/wiki-docs (public). 로컬 main이 `origin/main`을 추적한다.
- 검증: `git branch -vv`에서 `[origin/main]`, `git status -sb`에서 `main...origin/main`을 확인했다.
- 상태: 완료.

## 버전 종료

- 사용자 종료 판단: 진행 중.
- 남은 것: PLAN Phase 1~4.
