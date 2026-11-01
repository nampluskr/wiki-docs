# youtube-writer-notes — 유튜브 문서(R3) 작성 중 만난 모순과 처리

- 작성: rvp-writer (2026-11-01)
- 대상: `docs/guides/rust-video-player-youtube.md`
- 원칙: 판단이 필요한 모순은 멈추지 않고 여기에 적고 **보수적으로** 썼다. 묶음 A·B 본문과 기존 `research/` 파일은 고치지 않았다.

## 1. 확인 수준 (먼저 읽을 것)

| 항목 | 내용 |
| --- | --- |
| URL | 본문 출처 URL 은 세 보고서와 youtube-comparison.md 가 적은 것을 옮겼다. writer 는 웹을 다시 열지 않았다. `MPV41/...` 줄 번호(options.c 1003~1016, ytdl_hook.lua 806~811 등)는 sonnet·leader 확인값을 그대로 옮긴 것이다 |
| 로컬 | writer 가 직접 연 것은 묶음 A 본문(README, 01 일부, 05, 03·04 제목)과 묶음 B 의 03·05 일부, 목차 제목이다. 링크 대상 파일 존재는 목록으로 확인했다 |
| 판정 | youtube-comparison.md 2절 9건을 모두 반영했다. 3절(한쪽만) 채택 항목도 반영했다 |

## 2. 판정 반영 위치

| 판정 | 반영 절 |
| --- | --- |
| 1 `load-scripts` 불필요 | 3.1, 3.2 |
| 2 `ytdl-format` 기본값 빈 값 | 3.1 |
| 3 JS 런타임 최소 버전 | 4.3 |
| 4 yt-dlp.exe GPLv3+ | 4.2 |
| 5 `t=` 한 경로 | 8.4 |
| 6 VP9 정규식 `^vp0?9` | 5.1 |
| 7 자막 옵션 정식 이름 | 5.3 |
| 8 개발자 정책 서술 제외 | 10.2 |
| 9 캐시 수치는 실험값 | 5.4 |
| 3절 한쪽만 | `ignore-config`(3.1), `url` 크레이트(8.1), 재resolve(5.2), 저장 금지(7.5), `REDIRECT`(7.1), `%LOCALAPPDATA%`(4.4), PoToken(4.4·12), 코덱별 HW(6.2), argon 크기·메모리 제외(4.3·9.1) |

## 3. 판정이 없던 모순과 처리

| # | 모순 | 처리 |
| --- | --- | --- |
| 1 | zhongfly 빌드의 Lua 포함: argon 은 shinchiro·zhongfly 둘 다 포함, sonnet 은 zhongfly 미확인, sol 은 shinchiro 레시피만 확인 | **shinchiro 만 레시피로 확인**, zhongfly 는 미확인 (3.3). argon 서술은 근거 없음으로 처리 |
| 2 | Lua 없는 빌드의 증상: sonnet "스크립트가 컴파일에 안 들어간다", sol "`ytdl` 옵션 자체가 `HAVE_LUA` 조건 아래 등록" | 둘 다 소스 읽기이고 양립한다. 둘 다 적고 "실행 시험 없음" 표시. 감지법(옵션 set 오류, 로그, `user-data/mpv/ytdl/path`)은 sol·sonnet 합집합 (3.3) |
| 3 | 판정 1 의 근거에서 `ytdl` 기본값 줄 번호: sonnet 1003, leader 1004(`lua_ytdl_format`), 1016(`auto_load_scripts`) | `ytdl` 기본값은 1003, 포맷 NULL 은 1004, `auto_load_scripts` 는 1016 으로 각각 표기. 줄 번호는 재확인 안 함 |
| 4 | R1 + CPU `vf`: argon "`d3d11va` 에서 CPU `vf` 불가 또는 복사 모드로 강등" vs sonnet·sol "copy-back 성능 저하" | comparison 1절의 "copy-back 을 부른다" 표현을 따름 (6.3). argon 의 "불가" 는 싣지 않음 |
| 5 | 플레이리스트 단일 강제: argon "`no-playlist=` 를 추가", sonnet·sol "훅이 기본으로 `--no-playlist`" | 기본 동작으로 충분하다는 쪽을 채택. argon 방식은 불필요로 적음 (8.3). 순수 재생목록 URL 은 단일로 줄지 않는다는 sol 주의는 유지 |
| 6 | 썸네일: argon 이 `ytdl_hook-thumbnails=best` 를 UI 용으로 읽을 여지를 줌 | sol·sonnet·argon 모두 "비디오 트랙으로 붙는 기능"이고 UI 썸네일은 별도 조회가 안전 (8.5). argon 도 같은 결론 |
| 7 | 자막: argon `slang` 으로 선택, sol 은 `sub-langs`(추출 필터)와 `slang`(재생 트랙 선택)을 **다른 단계**로 구분 | sol 의 구분을 채택 (5.3) |
| 8 | 캐시 수치 3종류 | 판정 9 대로 실험값. sol·argon 값을 나란히 표시. sonnet 은 수치 없음 (5.4) |
| 9 | 탐색 지연 "수백 ms" (argon) | 출처 없는 추정이라 "argon 의 추정값, 출처 없음"으로 표시 (5.5) |
| 10 | 코덱별 GPU 세대: argon 에만 구체 세대, sonnet 은 큰 범주, sol 은 GPU 지원 확인 필요 | 판정(3절) 대로 "일반 지식, 공식 표 미확인" 표 (6.2). sonnet 의 "Windows AV1 확장 필요할 수 있음" 병기 |
| 11 | yt-dlp 요구 OS: sonnet "Win10+", sol "2026.10.26 부터 공식 Windows 바이너리 Win10+" | 둘이 양립. sol 의 "2026.10.26 부터" 표현 사용 (4.1) |
| 12 | D-2 표기: sonnet 은 "GPL 의존 금지"로 줄여 부름 | 실제 제목("Zed GPL 코드는 복사·의존하지 않고 …")대로 적고 05-WINDOWS.md 5절로 링크 (4.2). 묶음 A writer-notes 모순 16과 같은 처리 |
| 13 | 약관 인용: sonnet 은 한국어 원문 발췌, sol 은 영문 의역, argon 은 "4조" 번호를 붙임 | 번호는 페이지에 표시가 없다는 sonnet 서술에 맞춰 번호를 쓰지 않음. 한국어·영문 모두 짧은 의역으로 통일하고 원문 장문 인용은 하지 않음 (10.1) |
| 14 | Python·ffmpeg 필요 여부 | yt-dlp 단독 exe 는 Python 불필요(sol), ffmpeg 는 재생에 불필요(sonnet·sol). VC++ 재배포 필요성은 README 에서 주석 처리돼 있어 미확인(sonnet)이라 본문에 싣지 않음 |
| 15 | 별 수 204,244: sonnet 만 조회 | sonnet 확인값으로 표시 (10.3). 다른 보고서와 충돌 없음 |
| 16 | `loadfile` 인덱스 인자: sonnet 이 `loadfile URL replace -1 start=SEC` 를 추정으로 제시 | "추정, 문법은 mpv 버전별 확인 필요" 표시 (8.4, 12) |
| 17 | Tauri CSP 영향 | 보고서에 근거가 없다. "영향 없음(추정)" 한 줄로만 적음 (9.1) |

## 4. 제외한 것

| 내용 | 이유 |
| --- | --- |
| argon "공식 IFrame 사용 요구", DMCA 1201조, 법적 소송 선례 서술 | 판정 8. 출처 문구 미확인, 법률 판단 금지 |
| argon Deno 30~50 MB, WebView2 메모리 150~300 MB | 출처 없음 (comparison 3절) |
| argon `load-scripts=yes` 필수 | 판정 1 |
| argon 의 `sub-lang`, `write-sub` | 판정 7 |
| argon 의 Deno 2.0+/Node 20+ | 판정 3 |
| argon Rust MSVC 와 MinGW libmpv 의 ABI 우려 | 순수 C ABI 호환 서술이라 새 사실이 없고 05-WINDOWS.md 4.1 에 "MSVC·MinGW 산출물을 섞지 않는다" 가 이미 있다. 본문에서는 링크만 걸었다 |
| mpv·yt-dlp 코드 원문 | 규칙. 옵션 이름과 값만 적음 |

## 5. 사람이 확인할 것

- 본문의 mpv 소스 줄 번호와 `#anchor` 가 실제로 맞는지
- 채택 libmpv DLL 에 Lua 가 들어 있는지 (11절 1번)
- `t=` 이중 적용 여부와 `start` 우선순위 (11절 12번)
- 약관 개정판 여부 (영문·한국어 대조)
- 묶음 A·B 본문에서 이 문서로 거는 링크: A 의 README 와 01-ARCHITECTURE 1.3a 는 이미 `../rust-video-player-youtube.md` 를 가리킨다. B 는 아직 가리키는지 확인하지 않았다. 본문은 고치지 않았다
