> 버전: v0.2 · 작성일: 2026-11-01 · 상태: 진행 중
>
> 실제로 한 일과 검증 결과를 기록한다. 이번 버전의 의도는 `INTENT.md`에 있다. 지난 버전 기록은 `specs/v0.1/PROGRESS.md`에 있다.

# PROGRESS — wiki-docs v0.2

## Phase별 진행

없음.

## 사용자 요청 작업

### v0.2 버전 INTENT 작성 (2026-11-01)

- 요청: 위키를 `wiki/<분야>/<소스>/`로 나누고(예: `dev/youtube`), inbox·raw 폴더 이름을 `YYMMDD-{소스}-{title-in-kebab-case}`로 다시 정한다. v0.2 버전 INTENT부터 진행한다.
- 조치: 소스 종류(youtube·docs·web), 섞인 페이지 위치(web), 목적(구조 재정의 + 남은 원본 반영), 범위를 사람에게 물어 INTENT 초안을 쓰고 확정받았다. 그 직후 사용자가 앱 관련 원본 3개를 inbox에서 지웠다("앱은 독립된 레포로 관리되기 때문"). rust-gui는 남겨 dev로 정리하고, `apps` 분야를 없애고, 큰 원본 기준·이미지 처리를 제외 범위로 옮기기로 해서 INTENT를 같은 날 개정했다.
- 결과: `specs/v0.2/INTENT.md` 확정(개정). 사용자가 지운 원본 3개 폴더(102개 파일)의 삭제를 이 작업과 함께 커밋한다.
- 검증: `git status`로 지워진 파일이 rust-video-player·rust-video-player-native·zed-like-editor 아래뿐이고, inbox에 rust-gui·ego-lite-guide·ssh-tailscale이 남은 것을 확인했다.
- 상태: 완료. 다음은 `DECISIONS.md`.

## 버전 종료

- 사용자 종료 판단: 진행 중.
- 남은 것: DECISIONS → SPEC → PLAN → Phase 진행.
