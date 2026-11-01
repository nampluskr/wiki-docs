# 유튜브 R3 조사 비교 (sonnet · sol · argon(Flash))

- 작성: rvp-leader (2026-11-01)
- 대상: `research/sources/sonnet/youtube.md`, `sol/youtube.md`, `argon/youtube.md`
- argon 은 Gemini 4 Argon 사용량 한도 때문에 **Gemini 3.8 Flash** 로 바꿔 조사했다.
- 결과는 묶음 A·B 공통 문서 `docs/guides/rust-video-player-youtube.md` 의 근거다.

## 1. 셋 다 일치

| 항목 | 내용 |
| --- | --- |
| 구조 | 링크 → 새 영상 탭 → libmpv `loadfile URL` → 내장 `ytdl_hook.lua` 가 yt-dlp 를 subprocess 로 실행 → 기존 디코딩·출력 경로 |
| yt-dlp 경로 | `script-opts=ytdl_hook-ytdl_path=<절대 경로>` 로 지정. Windows 는 `.exe` |
| Lua | Lua 없는 libmpv 빌드에서는 안 된다. shinchiro 빌드 레시피는 `-Dlua=enabled` |
| DASH | 영상·음성 분리 스트림을 hook 이 EDL 로 묶어 mpv 가 동시에 재생. ffmpeg.exe 병합 불필요 |
| JS 런타임 | 유튜브 전체 지원에 EJS + 외부 JS 런타임(Deno 권장)이 필요. 공식 exe 는 EJS 를 내장하지만 런타임은 따로 둔다 |
| 갱신 | stable 대략 월간, nightly 권장. `-U`, `--update-to` |
| R1·R2 | 로컬 파일과 같은 mpv 경로라 그대로 적용. 코덱(VP9/AV1)별 하드웨어 지원은 GPU 에 달림. CPU `vf` 필터는 copy-back 을 부른다 |
| 오류 | `END_FILE` reason=error 는 대략적 코드뿐. 상세 사유는 로그 메시지·`user-data/mpv/ytdl/json-subprocess-result` |
| 제목 | `media-title` |
| 쿠키 | `ytdl-raw-options=cookies-from-browser=...` 로 완화 가능, 보장 없음 |
| iframe 대안 | 공식 경로지만 R1·R2 불가 → R3 대체안으로 부적합 |
| 약관 | 다운로드·자동화 접근·우회 제한 조항이 있다. 법률 판단은 하지 않는다 |

## 2. 상충과 판정 (leader 가 원문 확인, 2026-11-01)

| # | 항목 | 주장 | 판정 | 근거 |
| --- | --- | --- | --- | --- |
| 1 | libmpv 에서 `load-scripts=yes` 필요 여부 | argon: libmpv 기본 no 라 yes 필수 / sonnet·sol: 내장 hook 은 별개 | **필요 없음.** `load-scripts` 기본값은 true 이고 libmpv 프로필에 없다. 내장 스크립트는 `mp_load_builtin_scripts()` 가 `ytdl` 옵션으로 따로 로드 | v0.41.0 `options/options.c` 1016 `.auto_load_scripts = true`, `etc/builtin.conf` `[libmpv]`, `player/scripting.c` 262-265 |
| 2 | `--ytdl-format` 기본값 | argon: `bestvideo+bestaudio/best` / sol: 빈 값 | **빈 값(NULL).** 비면 yt-dlp 기본 선택 | `options/options.c` 1004 `.lua_ytdl_format = NULL` |
| 3 | JS 런타임 최소 버전 | argon: Deno 2.0+, Node 20+ / sonnet·sol: Deno 2.3.0+, Node 22.0.0+ | **Deno 2.3.0, Node 22.0.0, QuickJS 2023-12-9, Bun 1.2.11~1.3.14(deprecated)** | yt-dlp wiki EJS |
| 4 | yt-dlp.exe 라이선스 | sonnet: 결합물 GPLv3+ / sol: 여러 라이선스 혼합 | **PyInstaller exe 결합물은 GPLv3+.** 소스·PyPI 는 Unlicense | yt-dlp 2026.10.26 README "Licensing" |
| 5 | URL 의 `t=` 처리 | argon: hook 이 URL 을 직접 파싱 / sonnet·sol: yt-dlp JSON `start_time` 이 있으면 반영(t= 해석 여부 미확인) | **sonnet·sol 서술 채택.** 앱이 `t=` 를 파싱해 `start=` 로 넘기는 한 경로로 정하고 이중 적용을 피한다 | `ytdl_hook.lua` 806-811 (sonnet), sol Y7 |
| 6 | VP9 코덱 문자열 | sonnet·argon `vp09` / sol `vp9` | 둘 다 나올 수 있어 **`vcodec~='^vp0?9'`** 로 쓴다 (실기 확인 필요) | yt-dlp Format Selection 문법 |
| 7 | 자막 옵션 이름 | argon `sub-lang`, `write-sub` / sonnet·sol `sub-langs`, `write-subs` | **`sub-langs`, `write-subs`, `write-auto-subs`** (정식 이름). 값의 쉼표는 mpv 리스트 escaping 주의(sol) | yt-dlp README Subtitle Options |
| 8 | 개발자 정책 | argon: "공식 IFrame 사용을 요구", DMCA 위험 서술 | **싣지 않음.** 출처 문구 미확인, 법률 판단 금지 규칙. sol 의 III.I 사실 정리만 쓴다 | sol Y9 |
| 9 | 캐시 수치 | 셋 다 다른 제안값 | 공식 기본값이 아니라 **실험 시작값**으로만 적는다 | — |

## 3. 한쪽만

| 출처 | 내용 | 처리 |
| --- | --- | --- |
| sol | `config=no` 여도 yt-dlp 사용자 설정은 안 막힌다 → `ytdl-raw-options=ignore-config=` | 채택 |
| sol | URL 검증은 `url` 크레이트로 host 를 정확히 비교(`contains` 금지) | 채택 |
| sol | 화질 변경은 재resolve 후 다시 load (`all_formats` 는 느림) | 채택 |
| sol | 원본 watch URL 만 저장, signed URL·쿠키는 저장·로그 금지 | 채택 |
| sol | `END_FILE` 의 `REDIRECT`(재생목록 전환)는 실패가 아님 | 채택 |
| sonnet | Program Files 에서는 `-U` 불가 → `%LOCALAPPDATA%` 쪽 도구 폴더 | 채택 |
| argon | PoToken·SABR 로 앞으로 추출이 막힐 수 있음 | 위험으로 (추정) |
| argon | 코덱별 하드웨어 지원 GPU 세대 | "일반 지식, 공식 표 미확인" 으로 |
| argon | Deno 크기 30~50MB, WebView2 메모리 150~300MB | 출처 없음 → 싣지 않음 |
