# 조사 보고서 (sonnet) — 유튜브 재생 R3 (libmpv, Windows 11)

- 작성: rvp-scouter-sonnet / 확인일 2026-11-01. 저장 폴더는 지시문의 `sources$n` 을 `sources\sonnet` 으로 해석(추정).
- 표기: **[확인]** 이번에 원문을 직접 열어 봄, **[일반지식]** 기억 기반 미검증, **추정**, **미확인**.
- 출처 약칭: MPV41 = `https://github.com/mpv-player/mpv/tree/v0.41.0` (태그 v0.41.0, 릴리스 2025-12-21 [확인 `gh api repos/mpv-player/mpv/releases`]; 저장소에는 `git-release` 2026-06-04 표지도 있음). 아래 raw 경로는 모두 `https://raw.githubusercontent.com/mpv-player/mpv/v0.41.0/…`. YTDLP = `https://github.com/yt-dlp/yt-dlp` (README master, 2026-11-01 받음).
- 코드 원문은 옮기지 않았다. 옵션 이름·값 정도만 인용.

## 요약 (10줄)

1. mpv 는 **내장 Lua 스크립트 `ytdl_hook.lua` 로 유튜브 URL 을 yt-dlp 에 위임**한다. 옵션 `--ytdl` 기본값은 yes 이고, 소스상 기본값도 `lua_load_ytdl = true` 다 [확인 options/options.c, DOCS/man/options.rst:885].
2. **libmpv 에서도 같은 내장 스크립트가 로드된다** — `mp_load_builtin_scripts()` 는 `@ytdl_hook.lua` 를 옵션 `ytdl` 로 켜고 끄며, 파일 시스템의 `scripts` 폴더를 읽는 `load-scripts` 와 별개다 [확인 player/scripting.c:262-273]. libmpv 는 설정 파일을 읽지 않는 것이 기본이다 [확인 include/mpv/client.h:447-451]. 즉 코드로 `ytdl=yes` 를 명시하면 안전하다(권장, 추정).
3. **Lua 없이 빌드된 libmpv 는 유튜브가 안 된다**(meson 옵션 `lua` 가 꺼지면 스크립트가 없음) [확인 meson.options:22-25, meson.build:698-]. Windows 빌드 `shinchiro/mpv-winbuild-cmake` 는 `-Dlua=enabled`, `-Dlibmpv=true` 로 빌드한다 [확인 packages/mpv.cmake]. zhongfly 빌드도 `mpv-dev-*` 아카이브를 제공(Lua 포함 여부는 미확인).
4. **yt-dlp 가 필수 외부 프로그램**. 최신 `2026.10.26`(2026-10-26), 라이선스 Unlicense, Windows 배포는 `yt-dlp.exe`(Win10+ 단독 실행 파일) [확인 YTDLP README, `gh api`]. 단 **PyInstaller 로 만든 exe 는 GPLv3+ 코드를 포함해 결합물이 GPLv3+** 라고 README 가 밝힌다 [확인].
5. **유튜브 전체 지원에 JS 런타임이 필요**하다: `yt-dlp-ejs` + deno(권장, 최소 2.3.0)/node(22+)/quickjs/bun. yt-dlp.exe 는 ejs 스크립트를 내장하지만 **런타임(deno 등)은 별도 설치** [확인 `https://github.com/yt-dlp/yt-dlp/wiki/EJS`, README]. 런타임이 없으면 `web`·`tv_samsung` 클라이언트가 빠진다 [확인 README 1862행].
6. 화질은 `--ytdl-format` 에 yt-dlp 포맷 문자열을 넣어 고른다. DASH 분리 스트림은 ytdl_hook 이 영상·음성 URL 을 **EDL 로 합쳐** mpv 가 두 스트림을 동시에 연다(ffmpeg 병합 불필요) [확인 ytdl_hook.lua requested_formats / edl 처리].
7. **R1·R2 는 그대로 적용**된다: 유튜브는 mpv 에 일반 HTTP 스트림으로 보일 뿐이라 `hwdec`·`vf`·속성 변경이 같다. 달라지는 점은 코덱 선택(VP9/AV1 은 GPU 세대 의존)과 `--ytdl-format` 으로 H.264 를 강제하면 HW 디코딩 성공률이 오른다는 것(일반지식).
8. 오류는 `end-file` 이벤트(`reason=error`, `error` 코드)와 **로그 메시지**로 받는다. yt-dlp 의 stderr 는 `msg.error` 로 mpv 로그에 나오고, 결과는 속성 `user-data/mpv/ytdl/json-subprocess-result` 에 남는다 [확인 ytdl_hook.lua:960-1000, input.rst:3831].
9. 링크 처리: `youtu.be`, `youtube.com` 은 기본 `include` 패턴에 있다 [확인 options.rst:~929]. `t=` 시작 시각은 yt-dlp 가 `start_time` 으로 돌려주면 훅이 `start` 로 설정한다 [확인 ytdl_hook.lua:806-811].
10. **약관**: YouTube 서비스 약관에 콘텐츠의 다운로드·복제 금지, 자동화된 수단 접근 금지, 우회 금지 조항이 있다 [확인 `https://www.youtube.com/t/terms`, 발효 2022-01-05]. 법률 판단은 하지 않는다. 대안인 IFrame Player API 는 공식 경로지만 R1·R2 는 불가(Y8).

---

## Y1 mpv 의 ytdl 연동 (v0.41.0)

| 항목 | 내용 | 출처 |
|---|---|---|
| `--ytdl=<yes|no>` | youtube-dl 훅 스크립트를 켠다. 입력 URL 을 보고 사이트의 영상을 재생. "recent version of youtube-dl 가 설치돼 있어야" 하며 기본 yes | options.rst:885-892 [확인] |
| `ytdl_hook-ytdl_path` | 실행 파일 경로(들). `:`(Unix) / `;`(Windows) 구분. mpv 는 PATH 와 **mpv 설정 디렉터리** 순으로 찾는다. 기본 탐색 이름: `yt-dlp`, `yt-dlp_x86`, `youtube-dl`. Windows 는 `.exe` 만 허용 | options.rst:996-1001, ytdl_hook.lua:13-18 [확인] |
| 설정 방법 | `--script-opts=ytdl_hook-ytdl_path=C:/app/bin/yt-dlp.exe` | options.rst:894 [확인] |
| `--ytdl-format` | yt-dlp 의 `--format` 에 그대로 전달. 빈 값/`ytdl` 이면 `--format` 을 넘기지 않음(yt-dlp 기본 선택). `--vid=no` 이면 `bestaudio/best` 로 자동 변경 | options.rst:1007-1015, 115 [확인] |
| `--ytdl-raw-options` | yt-dlp 옵션을 `키=값` 목록으로 전달. 값 없는 옵션은 `=` 필요. 검증 없음. 예: `proxy=...`, `username=...` | options.rst:1017-1037 [확인] |
| `ytdl_hook-try_ytdl_first` | yes 면 mpv 가 열기 전에 yt-dlp 먼저 시도. 기본 no 이면 `include` 패턴 URL 만 먼저 시도 | options.rst:896-899 [확인] |
| `ytdl_hook-include` | 기본값 `^%w+%.youtube%.com/|^youtube%.com/|^youtu%.be/|^%w+%.twitch%.tv/|^twitch%.tv/` | options.rst:927-929 [확인] |
| `ytdl_hook-exclude` | 제외할 URL 패턴(`\|` 구분, Lua 패턴) | options.rst:901-918 [확인] |
| `ytdl_hook-all_formats` / `use_manifests` / `thumbnails` | 모든 포맷을 트랙으로 노출 / DASH·HLS manifest 사용(기본 no) / 썸네일을 비디오 트랙으로 추가(기본 none) | options.rst:930-995 [확인] |

호출 방식: 훅은 `yt-dlp --no-warnings -J --flat-playlist [--format …]` 형태로 실행하고 JSON 을 파싱한다. 재생목록이 아니면 `--no-playlist` 를 붙인다 [확인 ytdl_hook.lua:879-916]. 훅은 JSON 안의 http 헤더(User-Agent)·쿠키를 파일 로컬 옵션으로 mpv 에 넘긴다 [확인 ytdl_hook.lua:134-224].

## Y2 libmpv 에서의 동작

- **기본 로드 여부**: 내장 스크립트 목록(`osc`, `ytdl_hook`, `stats`, `console`, …)은 C 소스에 문자열로 포함되고(`player/lua.c` 의 `builtin_lua_scripts`), `mp_load_builtin_scripts()` 가 각 옵션(`lua_load_ytdl` 등)이 true 일 때 `@ytdl_hook.lua` 를 로드한다 [확인 player/lua.c:52-75, player/scripting.c:262-273]. `ytdl` 의 기본값은 true [확인 options/options.c:1003].
- **`load-scripts`**: 사용자 `scripts` 폴더 자동 로드 여부만 제어한다(기본 yes) [확인 options.rst:817-820, scripting.c:276-300]. 내장 `ytdl_hook` 은 `ytdl` 옵션이 제어하므로 `load-scripts=no` 로 해도 ytdl 은 영향 없음(소스 구조상, **추정** — 실행 시험 안 함).
- **`config`**: libmpv 는 기본적으로 설정 파일을 읽지 않는다(`--config=no` 와 비슷). 켜려면 `config-dir` 도 함께 지정하라고 문서가 권고 [확인 client.h:447-454]. 그래서 사용자의 `mpv.conf` 의 `ytdl-format` 등은 libmpv 앱에서 적용되지 않는다 → 앱이 코드로 옵션을 설정해야 한다.
- **libmpv 기본값 차이**: `input-default-bindings` 가 libmpv 에서는 꺼짐 [확인 options.rst:4580-4583]; 단말 접근 없음·idle 모드 [확인 client.h:444-458]. 유튜브와 직접 관련된 차이는 `config` 비로드와 `terminal` 없음 정도.
- **yt-dlp 위치 문제**: 훅은 PATH 와 mpv 설정 디렉터리에서 찾는다. 앱이 `yt-dlp.exe` 를 앱 폴더에 두고 `ytdl_hook-ytdl_path` 에 절대 경로를 주거나, `config-dir` 을 앱 폴더로 지정(추정)해야 한다. 찾지 못하면 속성 `user-data/mpv/ytdl/path` 가 빈 문자열이다 [확인 input.rst:3834-3838].
- **Lua 미포함 빌드**: meson 옵션 `lua` 가 `disabled` 이면 Lua 스크립트가 컴파일에 들어가지 않는다(소스: `lua_opt != 'disabled'` 일 때만 의존 탐색) [확인 meson.build:698-711]. 그러면 ytdl_hook 이 없어 유튜브 URL 이 열리지 않는다(추정 — 빌드로 시험 안 함). 대안: 앱이 직접 yt-dlp 를 호출해 직접 URL(또는 EDL)을 `loadfile` 에 넘긴다.
- **Windows 빌드**: shinchiro 빌드 스크립트에 `luajit` 의존과 `-Dlua=enabled`, `-Djavascript=enabled`, `-Dlibmpv=true` 가 있다 [확인 `https://raw.githubusercontent.com/shinchiro/mpv-winbuild-cmake/master/packages/mpv.cmake`]. 저장소 별 1846, push 2026-11-01, 릴리스 `20261101` [확인 `gh api`]. zhongfly/mpv-winbuild 는 `mpv-dev-x86_64-…7z`, `mpv-dev-lgpl-x86_64-…7z` 를 올린다(별 967, 최신 2026-10-31) [확인]. **LGPL 변형은 Lua 를 포함하는지 미확인**.
- 직접 확인 필요(미확인): 실제 DLL 에 Lua 가 들어 있는지는 DLL 로드 후 `mpv_get_property("user-data/mpv/ytdl/path")` 또는 로그의 `Lua`/`ytdl_hook` 줄로 시험해야 한다. 이번 조사는 DLL 을 내려받거나 실행하지 않았다.

## Y3 yt-dlp

- **최신/주기**: GitHub 릴리스(stable) `2026.10.26`, `2026.10.21`, `2026.08.19`, `2026.07.04`, `2026.06.09`, `2026.03.17` (각 2026-10-26, 10-21, 08-19, 07-04, 06-09, 03-17) [확인 `gh api repos/yt-dlp/yt-dlp/releases`]. 간격이 불규칙(5일~약 2.5개월). README: stable 은 "(mostly) monthly" 이며 외부 변화로 "stale/prone to breakage", **nightly 채널이 일반 사용자 권장**(변경 있는 날 자정 직전 UTC 발행), master 는 push 마다 [확인 README "update channels"].
- **라이선스**: 저장소 Unlicense [확인 GH]. 단 **PyInstaller exe 는 GPLv3+ 코드를 포함 → 결합물 GPLv3+**; zipimport 유닉스 실행 파일·tar.gz 는 ISC/MIT 코드 포함 [확인 README "Licensing"].
- **Windows 배포 형태** [확인 README 98-118행 및 release assets]: `yt-dlp.exe`(Win10+ x64, 권장), `yt-dlp_x86.exe`, `yt-dlp_arm64.exe`; 압축형 `yt-dlp_win.zip` 등은 "자동 업데이트 없음". 체크섬 `SHA2-256SUMS(.sig)` 제공.
- **앱과 묶기**: `yt-dlp.exe` 를 앱 폴더(예: `bin\yt-dlp.exe`)에 복사하고 `ytdl_hook-ytdl_path` 로 지정(추정). 재배포 시 GPLv3+ 결합물이라는 점 검토(Y9와 별도, 법률 판단 아님). `.exe` 는 첫 실행 시 PyInstaller 압축 해제로 시작이 느릴 수 있음(일반지식, 미시험).
- **갱신**: `yt-dlp -U`(현재 채널의 최신으로), `--update-to nightly|master|stable@TAG` [확인 README 172-186행]. 자동 업데이트 불가인 zip 형은 직접 교체. 앱 설계: 시작 시/오류 시 `-U --update-to nightly` 실행, 실패하면 이전 파일 보존(추정). 설치 폴더가 쓰기 불가(Program Files)면 업데이트 불가 → 사용자 데이터 폴더(`%LOCALAPPDATA%`)에 두는 것이 안전(추정).
- **갱신이 잦은 이유**: 유튜브가 서명/`n` 파라미터 해독 코드와 클라이언트 구조를 자주 바꾸기 때문. README: 기본 `player_client` 는 `tv_samsung,visionos,web` 이고, JS 런타임이 없으면 `web`·`tv_samsung` 이 제외, 일부 클라이언트는 `po_token` 이 필요 [확인 README 1862행, 1879행].
- **최근 추가 의존성 (중요)**: ① `yt-dlp-ejs`(Unlicense; MIT/ISC 포함) "Required for full YouTube support" ② **JS 런타임**: deno(권장, 기본 활성, ≥2.3.0), node(≥22.0.0), QuickJS(≥2023-12-9), bun(Deprecated, ≥1.2.11) [확인 `https://github.com/yt-dlp/yt-dlp/wiki/EJS`, README 213-215행, 367-397행]. 표준 exe 는 ejs 스크립트를 **내장**하지만 런타임은 별도 설치 필요. `--remote-components ejs:npm|ejs:github` 로 ejs 를 원격에서 받을 수도 있다(deno/bun 한정 npm). 이전 PhantomJS 방식은 "유튜브에는 더 이상 사용하지 않음" [확인 README 241행]. 이 요구가 도입된 yt-dlp 버전은 **미확인**.
  - 앱 영향: deno 를 함께 배포하거나(`--js-runtimes deno:C:\path\deno.exe`) 사용자에게 설치 안내해야 한다(`ytdl-raw-options=js-runtimes=deno:…` 로 전달 가능 — 추정, 값에 `:` 가 있는 경우의 인용은 미시험).
  - ffmpeg: yt-dlp 자체 병합용이며, mpv 로 재생할 때는 yt-dlp 가 병합하지 않으므로 불필요(Y4). README 는 "병합과 후처리에 필요"라고만 한다 [확인 README 203-208행].
  - 실행 요건: Windows exe 는 Win10+ [확인 README 98행]. VC++ 재배포 필요성은 주석 처리돼 있음 [확인 README 198-200행].

## Y4 재생 설정

- **format 문자열 예** (yt-dlp 문법 [확인 README "Format Selection" 1696, 1706, 1716 행 예]):
  - 1080p 이하 + 최선: `--ytdl-format="bv*[height<=1080]+ba/b[height<=1080]"` (README 1706행 480p 예의 변형, 추정).
  - H.264 우선(HW 디코딩 호환): `bv*[vcodec^=avc1]+ba/b` 또는 정렬: 포맷 정렬 필드 `codec` / `+codec:avc:m4a` 가 README 에 있다 [확인 README 1641행]. mpv 에서 `-S` 를 쓰려면 `--ytdl-raw-options=format-sort=…` (키=값 규칙, 값에 쉼표가 있으면 인용 필요 — **미시험, 추정**).
  - VP9 우선(4K): `bv*[vcodec^=vp09]+ba/b` / AV1: `bv*[vcodec^=av01]+ba/b` (코드 이름은 yt-dlp 의 `vcodec` 값, 일반지식).
  - 4K 포함 최고 화질: `bv*+ba/b` (기본 선택과 유사; 기본 정렬은 `lang,quality,res,fps,hdr:12,vcodec,…` [확인 README 1643행]).
- **DASH 분리 스트림 합치기**: yt-dlp JSON 의 `requested_formats`(영상, 음성)를 ytdl_hook 이 **EDL** 로 엮어 `stream-open-filename` 에 넣는다. 영상·음성이 별도 URL 이어도 mpv 가 한 파일처럼 열고 `external audio` 처럼 동기시킨다 [확인 ytdl_hook.lua:465, 633-703]. 분리 EDL 에서는 오디오·비디오 각각 다른 서버라서 한쪽 만료 시 둘 다 실패(일반지식).
- **자막**: yt-dlp `--write-subs`, `--sub-langs "en.*,ko"` 가 존재하고 [확인 README 891, 901행], 훅은 JSON 의 `requested_subtitles` 를 `sub-add` 로 붙인다 [확인 ytdl_hook.lua:725-753]. 따라서 mpv 에서는 `--ytdl-raw-options=write-subs=,sub-langs=ko.*,en.*` (값 없는 플래그는 `=`) 로 요청해야 자막 트랙이 생긴다(추정). 자동 생성 자막(`write-auto-subs`)은 별개 옵션(일반지식).
- **캐시/버퍼**: `--cache=<yes|no|auto>`(기본 auto; 네트워크 스트림이면 켜짐), `--cache-secs`, `--demuxer-max-bytes`(앞으로 읽는 최대량), `--demuxer-max-back-bytes`(뒤로 탐색용), `--demuxer-readahead-secs` [확인 options.rst:5382-5400, 4175-4270]. 기본값 수치는 이번에 확인하지 않음(미확인). 4K 를 위해 `demuxer-max-bytes` 를 올릴 필요가 있을 수 있음(추정).
- **탐색**: 캐시에 있는 구간은 빠르고 아니면 HTTP Range 재요청. 유튜브 DASH 스트림은 Range 지원(일반지식, 미시험). 라이브 스트림은 구현이 다르다 — 이번 조사 범위 밖.

## Y5 R1·R2 와의 관계

- mpv 입장에서 유튜브 스트림은 HTTPS 의 MP4/WebM 일 뿐이므로 **`hwdec`·`vf`·밝기/대비/채도·`deinterlace` 설정이 그대로 적용** [확인 report-1 의 mpv 문서 근거와 동일: options.rst:1340-1420, input.rst:1660]. 변경 방법은 report-1 참조.
- 코덱별 HW 디코딩(일반지식 — 이번엔 공식 표 미확인):
  - **H.264(avc1)**: D3D11VA 가 사실상 모든 GPU 에서 가능. 가장 안전.
  - **VP9**: 최근 NVIDIA/Intel/AMD 에서 대부분 가능하나 구세대 GPU 는 불가 → 자동 소프트 전환(R1).
  - **AV1**: 비교적 최신 GPU(NVIDIA RTX30+, AMD RDNA2+, Intel Arc/Gen12 이후)만 HW, Windows 의 AV1 확장 필요할 수 있음. 미지원이면 dav1d 소프트 디코딩이 CPU 부담(4K 에서 특히).
  - 그래서 "유튜브 화질 선택"과 R1 이 연결된다: HW 가 안 되는 환경에서 4K AV1/VP9 를 고르면 끊김 → 앱이 `hwdec-current` 와 프레임 드롭 속성으로 판단해 화질을 낮추는 UX 가 필요(추정, 설계 제안).
- **필터 적용 차이**: 없다. 단 4K 에서 `vf` 가 CPU 필터면 `hwdec=d3d11va`(무복사)에서는 필터 때문에 프레임이 CPU 로 내려와 성능 저하 가능(일반지식, mpv 문서의 `-copy` 설명과 일치 [확인 options.rst:1420]). 유튜브 특유 문제는 아님.

## Y6 오류와 상태

- **유튜브 쪽 오류 유형** (yt-dlp 가 stderr 로 보고하는 메시지; README 의 구체적 오류 문구 목록은 이번에 확인 안 함 → **미확인**): 연령 제한, 지역 제한, 로그인 필요, "Sign in to confirm you're not a bot", 비공개/삭제, 라이브/프리미어 전. 쿠키가 도움이 되는 경우가 많다.
- **쿠키**: yt-dlp `--cookies-from-browser BROWSER[+KEYRING][:PROFILE][::CONTAINER]` 와 `--cookies FILE` [확인 README 721행, 2278행]. mpv 에서는 `--ytdl-raw-options=cookies-from-browser=chrome` 또는 `cookies=C:/…/cookies.txt` (추정, 미시험). 로그인 쿠키를 넘기면 클라이언트 선택이 달라진다 [확인 README 1862행]. 보안: 쿠키 파일은 계정 자격과 같으므로 앱이 저장하지 말 것(권고).
- **mpv 가 오류를 알려주는 방법** (libmpv API):
  - `MPV_EVENT_END_FILE`: `reason` 이 `ERROR`(4) 이고 `error` 필드에 mpv 에러 코드(예: `MPV_ERROR_LOADING_FAILED` -13, `MPV_ERROR_UNKNOWN_FORMAT` -17) [확인 client.h:347, 366, 1461-1520; input.rst:1735-1760]. 단 이것은 "대략적" 이유뿐이라 yt-dlp 의 실패 사유(연령 제한 등)는 담기지 않는다.
  - **로그 메시지**: ytdl_hook 은 yt-dlp stderr 를 `msg.error` 로 낸 뒤 "youtube-dl failed: …(not found or not enough permissions / failed to parse JSON data / unexpected error occurred)" 를 낸다 [확인 ytdl_hook.lua:975-1000]. 앱은 `mpv_request_log_messages("error")` 로 받아 사용자에게 표시(추정). 접두사는 스크립트 이름(예: `ytdl_hook`)일 것이나 정확한 접두사는 **미확인**.
  - **속성** `user-data/mpv/ytdl/json-subprocess-result`: yt-dlp 실행 결과(status/stdout/stderr)를 그대로 담는다 [확인 input.rst:3831-3840]. 오류 분류(문자열 매칭)에 가장 직접적.
  - `user-data/mpv/ytdl/path` 가 빈 문자열이면 yt-dlp 를 못 찾음 [확인 input.rst:3834-3838].
- **제목/썸네일**: `media-title` 속성 [확인 input.rst:2176], `metadata` 속성 [확인 input.rst:2389]. 훅이 yt-dlp JSON 의 `title` 등을 mpv 메타데이터로 올리는 매핑 표가 있다(`ytdl_playlist` 등 [확인 ytdl_hook.lua:54-57]). 썸네일은 옵션 `ytdl_hook-thumbnails=best|all` 로 비디오 트랙으로 붙일 수 있으나 [확인 options.rst], 탭 아이콘용 썸네일 URL 은 JSON 직접 파싱이 더 적합(추정).

## Y7 링크 → 탭 연결

- **URL 판별(앱 쪽)**: 호스트가 `youtube.com`/`www.`/`m.`/`music.` 또는 `youtu.be`; 경로 `/watch?v=`, `/shorts/ID`, `/live/ID`, `/embed/ID`, `/playlist?list=`. mpv 훅의 기본 `include` 는 `^%w+%.youtube%.com/|^youtube%.com/|^youtu%.be/` 로 위 대부분을 잡는다 [확인 options.rst:927-929]. 일반 `https://…` 링크 전체를 훅에 맡기려면 `try_ytdl_first=yes`(느려짐) [확인 options.rst:896].
- **앱 설계**: 링크 → 새 탭 생성 → 탭 안의 세션이 `loadfile URL`. 파일 탭과 같은 `VideoSession` 을 쓰고 소스 종류만 `Url` (추정, 설계).
- **재생목록**: URL 에 `list=` 가 있으면 훅이 `--no-playlist` 를 빼고 `--yes-playlist` 처리를 한다(훅은 `ytdl_raw_options` 에 `yes-playlist` 가 있으면 `use_playlist`) [확인 ytdl_hook.lua:904-916]. 기본(옵션 없음)에서는 영상 하나만 재생. 재생목록 사용 시 mpv 가 M3U 형태로 항목을 채우고 `playlist-start` 를 맞춘다 [확인 ytdl_hook.lua:1085-1135]. 항목 URL 은 `https://youtu.be/ID` 로 구성. 앱이 "재생목록 열기" 를 지원하려면 `ytdl-raw-options=yes-playlist=` 를 켜고 `playlist` 속성으로 항목 목록을 표시(추정).
- **시작 시각 `t=`**: 훅은 yt-dlp JSON 의 `start_time`(또는 `section_start`)이 있고 `start` 옵션이 사용자가 안 정했을 때 `file-local-options/start` 로 설정한다 [확인 ytdl_hook.lua:806-811]. 유튜브 URL 의 `t=` 가 yt-dlp 의 `start_time` 로 해석되는지는 **미확인**(yt-dlp 추출기 동작; 일반지식으로는 URL 의 `t=` 를 `start_time` 으로 처리하는 것으로 알려져 있음). 안전한 방법: 앱이 URL 에서 `t=`/`start=` 를 파싱해 `loadfile URL replace -1 start=SEC` 로 직접 지정(추정).

## Y8 대안: Tauri 웹뷰의 YouTube IFrame Player API

출처: `https://developers.google.com/youtube/iframe_api_reference` [확인].
- 지원: `loadVideoById`/`cueVideoById` 에 `startSeconds`, `seekTo()`; 재생목록 `loadPlaylist`/`cuePlaylist`, `nextVideo` 등; `onError` 코드 2, 5, 100(없음/비공개), 101·150(소유자가 임베드 금지), 153(Referer 없음); 플레이어 최소 200×200, 권장 16:9 480×270 이상; 자동재생 차단 이벤트 `onAutoplayBlocked` [확인].
- 장점: 공식 경로, yt-dlp·JS 런타임·ffmpeg 불필요, 약관상 합법 임베드 방식, 웹 표준 재생(구현 쉬움), 소리/자막/화질 선택이 유튜브 UI 로 제공. 갱신이 필요 없다.
- 단점: ① **R1·R2 불가**: 페이지에 하드웨어 디코딩·필터·셰이더를 제어하는 문서가 없다(문서에 해당 항목 없음 [확인]). 웹뷰(WebView2)의 디코딩은 Chromium 내부 결정이고 사용자 정의 필터는 iframe 안의 `<video>` 에 접근 불가(교차 출처, 추정). CSS 필터를 iframe 요소 전체에 거는 정도는 가능하나 sharpen/denoise/deinterlace 는 불가(추정). ② 임베드 금지 영상 재생 불가(101/150). ③ 광고·추천 UI 가 보이고, "같은 플레이어 화면"이라는 R3 와 불일치(자체 컨트롤 불가). ④ gpui 앱에는 웹뷰가 없어 쓸 수 없음(묶음 B egui+glow 도 마찬가지, 추정).
- 결론: R3 의 "같은 플레이어 화면 + R1·R2" 요구와는 맞지 않는 보조 옵션. "R1·R2 없는 안전한 대체 재생" 용으로만 의미.

## Y9 약관·법적 위험 (사실만, 판단 없음)

- 출처 `https://www.youtube.com/t/terms` (발효일 2022-01-05, 한국어 페이지로 열림) [확인 WebFetch]. 인용(한국어 원문 발췌, 1~2문장):
  - "서비스나 콘텐츠의 어떤 부분에 대해서도 액세스, 복제, 다운로드, 배포, 전송, 방송, 표시, 판매, 라이선스 부여, 변경, 수정 또는 그 밖의 사용을 금지합니다." — 섹션 "권한 및 제한사항"(번호는 페이지에 표시 없음). 다만 YouTube 또는 해당 권리자의 사전 서면 승인이 있는 경우는 예외로 서술된다.
  - "우회, 무력화, 기망적인 관여 또는 그 밖의 방해 행위(또는 이러한 행위들에 대한 시도)를 금지합니다." — 복제·사용을 막는 기능을 우회하는 행위를 포함.
  - "자동화된 수단(로봇, 봇넷 또는 스크래퍼 등)을 사용해 서비스에 액세스하면 안 됩니다." — 공개 검색엔진의 robots.txt 준수 및 사전 서면 승인 예외.
- **API 서비스 약관(별도 문서)**: IFrame Player API 사용에는 별도의 "YouTube API 서비스 약관" 이 적용된다 — 이번에 해당 문서는 열지 않음(**미확인**). `https://developers.google.com/youtube/terms/api-services-terms-of-service` 를 확인해야 한다.
- **개인 사용 vs 배포**: 약관 본문은 "서비스를 이용하는 개인"에 적용되며, 앱을 배포하는 쪽에 대한 책임은 본 조항만으로는 판단할 수 없다. yt-dlp 같은 도구를 앱이 호출하거나 번들하는 경우의 위험은 법률 검토 사항 — 이번 조사는 판단하지 않는다. 사실로 확인된 것: ① yt-dlp 저장소가 활성(별 204,244, push 2026-10-29) [확인 `gh api`] ② yt-dlp 가 유튜브용 JS 해독 컴포넌트를 쓴다 [확인 README]. ③ 약관의 "다운로드" 금지 문구와 mpv 의 스트리밍 재생(저장하지 않음) 사이의 해석 차이는 **법률 판단이 필요해 이 문서에서 다루지 않음**.
- 참고 사실: `yt-dlp.exe` 결합물이 GPLv3+ 라는 점은 앱 라이선스(예: workbench-gpui 의 D-2 "GPL 의존 금지" [확인 DECISIONS.md])와 충돌할 수 있다. **별도 프로세스로 실행**하는 것은 일반적으로 "정적/동적 링크" 와 다르게 취급된다고 알려져 있으나(일반지식) 이것도 법률 검토 대상이다.

## Y10 확신 낮은 부분

1. **내장 ytdl_hook 이 libmpv 에서 실제로 동작하는지 실행 시험을 하지 않았다**(DLL 을 받지 않음). 소스 근거(`mp_load_builtin_scripts`, 기본 `ytdl=yes`)만으로 판단.
2. `load-scripts=no` 가 내장 `ytdl_hook` 에 영향을 주지 않는다는 결론은 소스 구조로 추정한 것(시험 없음).
3. Windows libmpv DLL 별 Lua 포함 여부: shinchiro 는 빌드 스크립트로 확인, zhongfly(특히 LGPL 변형)와 mpv.io 공식 안내 빌드는 **미확인**.
4. yt-dlp 의 JS 런타임 요구가 도입된 정확한 버전, 현재 유튜브의 `po_token` 필요 빈도 — **미확인**. 이 영역은 가장 빨리 바뀐다(README 가 수시로 변경).
5. `t=` 를 yt-dlp 가 `start_time` 으로 해석하는지, `ytdl-raw-options` 의 `format-sort` 처럼 쉼표/콜론이 든 값의 인용법 — **미시험**.
6. 오류 종류별 yt-dlp 메시지 원문(연령 제한, 봇 확인 등)과 쿠키 사용 효과 — 공식 문서에서 목록을 확인하지 못함.
7. 유튜브 코덱별 HW 디코딩 지원 GPU 세대 표 — 공식 표를 열지 않았고 일반지식으로만 서술.
8. 약관은 한국어로 열렸으며 발효일 2022-01-05 판이 현재 유효한 최신인지 **미확인**(2026-11-01 시점에 개정판이 있을 수 있음). 영어 원문과 대조하지 않았다.
9. YouTube API 서비스 약관, 개발자 정책 — 열지 않음.
10. 다른 scouter 2명과 달리 이 보고서는 mpv **v0.41.0 태그의 실제 소스**(options.rst, ytdl_hook.lua, scripting.c, client.h, meson)를 1차 근거로 썼다. `git-release` 2026-06-04 표지에서 변경이 있었을 수 있다(미확인).
