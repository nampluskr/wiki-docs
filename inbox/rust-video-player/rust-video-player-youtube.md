---
tags: [rust, video-player, youtube, mpv, yt-dlp, libmpv, windows, r3]
created: 2026-11-01
updated: 2026-11-01
---

# 유튜브 링크 재생 (R3) — 묶음 A·B 공통

> 이 파일을 읽는 때: 플레이어 앱에서 유튜브 링크를 클릭해 탭으로 재생하게 만들 때. 묶음 A(gpui·Tauri)와 묶음 B(egui+glow)가 함께 쓴다.

묶음 A 입구는 [rust-video-player/README.md](rust-video-player/README.md), 묶음 B 입구는 [rust-video-player-native/README.md](rust-video-player-native/README.md)다. 요구 정의는 A 의 [01-ARCHITECTURE.md](rust-video-player/01-ARCHITECTURE.md) 1.3a 절에 있다.

| 항목 | 내용 |
| --- | --- |
| 대상 | 링크 클릭으로 유튜브 영상을 탭에서 재생하려는 사람 |
| 결과 | 동작 구조, libmpv 설정, yt-dlp·JS 런타임 배포, 화질·자막·캐시 설정, 오류 처리, 링크 판별, 약관 사실 정리, 검증 체크리스트 |
| 기준 | mpv v0.41.0 태그 소스·문서, yt-dlp 2026.10.26(stable), Windows 11. 확인일 2026-11-01 |
| 검증 범위 | **조사만 했다.** 패키지 설치, 빌드, DLL 실행, 실제 유튜브 재생, 쿠키 접근은 하지 않았다. 소스에서 읽은 동작과 실제 바이너리의 재생 성공은 구분한다 |
| 판정 | 세 보고서(sonnet·sol·argon)가 어긋난 값은 `rust-video-player/research/reviews/youtube-comparison.md` 의 leader 판정을 따랐다 |
| 코드 | 아래 조각은 모두 **설계 예시**다. mpv·yt-dlp 코드 원문은 싣지 않았고 옵션 이름과 값만 적었다 |

표기: **확인**(조사자가 원문을 연 값, 출처 URL 병기), **추정**, **미확인**(구현 전에 확인), **설계 예시**(이 문서의 제안). mpv 소스 약칭은 `MPV41` = https://github.com/mpv-player/mpv/blob/v0.41.0 , yt-dlp 약칭은 `YTDLP` = https://github.com/yt-dlp/yt-dlp/blob/2026.10.26 이다.

## 1. 요구 R3 와 범위

R3: 유튜브 링크를 클릭하면 **새 탭**에서 로컬 파일과 **같은 플레이어 화면**으로 재생한다. R1(GPU 가속 ON/OFF)·R2(영상 옵션)가 그대로 적용된다.

### 1.1 공통과 묶음별

| 영역 | 공통 (이 문서) | 묶음 A | 묶음 B |
| --- | --- | --- | --- |
| 링크 → 재생 | `loadfile URL` → ytdl_hook → yt-dlp → EDL (2~5절) | 같음 | 같음 |
| yt-dlp·JS 런타임 | 선택·배포·갱신 (4절) | 설치본에 동봉 (Tauri 는 resource/sidecar) | 설치본에 동봉 |
| 오류·상태 | 이벤트·로그·표시 정책 (7절) | 세션 이벤트 규약은 [01-ARCHITECTURE.md](rust-video-player/01-ARCHITECTURE.md) 2.2 | 코어 Event 는 [04-player-core.md](rust-video-player-native/04-player-core.md) 3절 |
| 링크 판별 | `url` 크레이트 검증, URL 종류 (8절) | 같음 | 같음 |
| 링크를 클릭하는 자리 | — | gpui·Tauri 각각 (9.1) | egui (9.2) |
| 화면 출력·R1 interop | — | [03-GPUI.md](rust-video-player/03-GPUI.md), [04-TAURI.md](rust-video-player/04-TAURI.md) | [03-rendering-pipeline.md](rust-video-player-native/03-rendering-pipeline.md) |

### 1.2 이 문서가 다루지 않는 것

- 법률 판단. 약관은 사실만 적는다 (10절).
- 광고 우회·로그인 자동화·다운로드 저장. 재생만 다룬다.
- 라이브 스트림의 탐색·지연. 조사 범위 밖이다 (sonnet).

## 2. 동작 구조

### 2.1 흐름

| 단계 | 일어나는 일 | 출처 |
| --- | --- | --- |
| 1 | 사용자가 링크를 클릭한다. 앱이 URL 을 검증·정규화한다 (8절) | 설계 |
| 2 | 새 영상 탭과 `PlayerSession`(소스 `Url`)을 만든다 | 설계, [01-ARCHITECTURE.md](rust-video-player/01-ARCHITECTURE.md) 2절 |
| 3 | 세션이 libmpv 에 `loadfile URL` 을 보낸다 | https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst |
| 4 | mpv 내장 Lua 스크립트 `ytdl_hook.lua` 가 URL 을 보고 yt-dlp 를 **하위 프로세스**로 실행한다. 결과는 JSON 이다 | `MPV41/player/lua/ytdl_hook.lua` |
| 5 | 훅이 JSON 에서 영상·음성 URL, 자막, 제목, 헤더·쿠키를 꺼내 mpv 에 넘긴다 | 같음 |
| 6 | 영상·음성이 분리된 DASH 면 훅이 둘을 **EDL** 로 묶는다. mpv 가 두 스트림을 한 파일처럼 동시에 읽는다 | 같음 (sonnet, sol, argon 일치) |
| 7 | 이후는 로컬 파일과 같은 디코딩·출력 경로다 (R1·R2 그대로, 6절) | — |

- 유튜브 페이지 URL 을 FFmpeg 디코더가 직접 해석하는 것이 아니다. 훅이 yt-dlp 에 위임한다. `MPV41/player/lua/ytdl_hook.lua` (sol).
- 훅 호출은 `yt-dlp --no-warnings -J --flat-playlist [--format …]` 형태다. 재생목록이 아니면 `--no-playlist` 가 붙는다. 소스를 읽은 값이고 실행 시험은 없다. `MPV41/player/lua/ytdl_hook.lua` (sonnet).
- 분리 스트림은 `ffmpeg.exe` 로 병합하지 않는다. 병합은 yt-dlp 가 파일로 저장·후처리할 때의 의존성이다. https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#dependencies (sol, sonnet).
- 분리 EDL 은 영상·음성 URL 이 따로라서 한쪽이 만료되면 둘 다 실패한다. 일반 지식, 미시험 (sonnet).

### 2.2 한눈에 보는 구성요소

| 구성요소 | 역할 | 배포 |
| --- | --- | --- |
| libmpv DLL | 재생 엔진. **Lua 포함 빌드**여야 한다 (3.3) | [05-WINDOWS.md](rust-video-player/05-WINDOWS.md) 4.2 |
| `yt-dlp.exe` | URL → 스트림 주소·자막·제목 추출 | 앱 `tools\` 또는 사용자 데이터 폴더 (4절) |
| JS 런타임 (Deno 등) | 유튜브 서명·`n` 파라미터 challenge 풀이 | 같은 `tools\` (4.3) |

## 3. libmpv 설정

### 3.1 옵션 표

libmpv 옵션 setter 에는 이름 앞 `--` 를 붙이지 않는다. 공백이 든 경로는 문자열로 조립하지 말고 API 값으로 넘긴다. https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/client.h (sol).

| 옵션 | 값 | 이유 | 출처 |
| --- | --- | --- | --- |
| `ytdl` | `yes` | 기본이 yes 다. 계약을 고정하려고 명시한다 (권장) | `MPV41/options/options.c` 1003~1016 (sonnet, 판정 1), `MPV41/DOCS/man/options.rst` |
| `script-opts` | `ytdl_hook-ytdl_path=<yt-dlp.exe 절대 경로>` | 앱 동봉본을 쓰게 한다. 기본 탐색은 PATH 와 mpv 설정 디렉터리이고 이름은 `yt-dlp`, `yt-dlp_x86`, `youtube-dl` 이다. Windows 는 `.exe` 만 허용하고 여러 후보는 `;` 로 구분한다 | `MPV41/DOCS/man/options.rst` (sonnet, sol) |
| `ytdl-raw-options` | `ignore-config=` 외 앱이 정한 옵션 | `config=no` 여도 **yt-dlp 의 사용자 설정은 안 막힌다.** 훅이 `--ignore-config` 를 기본으로 넣지 않는다 (판정, sol) | `MPV41/player/lua/ytdl_hook.lua`, https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#configuration |
| `ytdl-format` | 5.1 절 | 기본값은 **빈 값**이다. 비면 `--format` 을 넘기지 않아 yt-dlp 기본 선택을 따른다 (판정 2) | `MPV41/options/options.c` 1004 |
| `config` | `no`(기본) | libmpv 는 기본적으로 설정 파일을 읽지 않는다. 사용자 `mpv.conf` 의 `ytdl-format` 은 적용되지 않으므로 앱이 코드로 정한다 | https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/client.h (sonnet, sol) |
| `load-scripts` | **필요 없음** | 아래 3.2 | 판정 1 |

`script-opts` 는 키=값 목록이다. `script-opts` 는 목록 전체를 대체하고 `script-opt`(단수)는 항목을 더한다. 값에 쉼표가 있으면 mpv 목록 이스케이프가 필요하다 (sol). Windows `;` 는 후보 구분자라서 경로 이스케이프 정책을 확인한다 (**미확인**).

설계 예시 (옵션 이름과 값만):

```text
ytdl=yes
script-opts=ytdl_hook-ytdl_path=C:/MyPlayer/tools/yt-dlp.exe
ytdl-raw-options=ignore-config=
ytdl-format=          # 빈 값: yt-dlp 기본 선택
```

### 3.2 `load-scripts` 는 필요 없다

| 보고서 | 주장 |
| --- | --- |
| argon | libmpv 기본이 `no` 라서 `yes` 가 필수 |
| sonnet·sol | 내장 hook 은 별개 경로 |
| **판정** | **필요 없다.** `load-scripts` 의 기본은 true 이고 libmpv 프로필에 없다. 내장 스크립트는 `mp_load_builtin_scripts()` 가 `ytdl` 옵션으로 따로 로드한다 |

근거: v0.41.0 `options/options.c` 1016 `.auto_load_scripts = true`, `etc/builtin.conf` 의 `[libmpv]` 항, `player/scripting.c` 262~265 (leader 확인). https://github.com/mpv-player/mpv/blob/v0.41.0/player/scripting.c

- `load-scripts` 는 사용자 `scripts` 폴더의 자동 로딩만 제어한다. 사용자 스크립트를 안 쓰면 `no` 로 둬도 된다 (소스 구조상 추정, 실행 시험 없음).
- 초기화 순서: create → 초기 옵션(yt-dlp 경로 포함) → initialize → 렌더러 연결·이벤트 펌프 → 비동기 `loadfile`. `config`·`config-dir`·`load-scripts` 는 initialize 전에 정한다. https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/client.h (sol).

### 3.3 Lua 포함 빌드 확인

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| Lua 없는 빌드 | meson 옵션 `lua` 가 꺼지면 ytdl_hook 이 없다. `ytdl` 옵션 자체가 `HAVE_LUA` 조건 아래 등록된다는 소스 읽기가 있다. 실행 시험은 없다 | `MPV41/meson.build` 698~711, `MPV41/options/options.c` (sonnet, sol) |
| 감지 | `ytdl` 옵션 set 이 오류를 돌려주는지 확인한다. 로그에 `ytdl_hook` 줄이 나오는지 본다 | sol (추정) |
| 속성 | `user-data/mpv/ytdl/path` 가 **빈 문자열**이면 yt-dlp 를 못 찾은 것이다 | `MPV41/DOCS/man/input.rst` (sonnet) |
| 외부 Lua 파일 | Lua 없는 DLL 에 Lua 파일을 복사해도 동작하지 않는다 | sol |
| shinchiro 빌드 | 레시피가 `luajit` 의존과 `-Dlua=enabled`, `-Dlibmpv=true` 를 명시한다. 레시피는 Lua 포함을 **의도**하지만 과거·변형 바이너리를 보증하지 않는다 | https://github.com/shinchiro/mpv-winbuild-cmake/blob/0ee575368129b7656bde32532711c6d17906af6a/packages/mpv.cmake (sonnet, sol) |
| zhongfly 빌드 | `mpv-dev-*`, `mpv-dev-lgpl-*` 아카이브를 제공한다. **Lua 포함 여부는 미확인**이다. argon 은 둘 다 포함이라 했으나 확인한 것은 shinchiro 레시피뿐이다 | https://github.com/zhongfly/mpv-winbuild (sonnet) |
| 대안 | 앱이 yt-dlp JSON 을 직접 받아 영상·음성 URL 을 libmpv 에 연결하는 resolver. 훅이 하던 EDL·재생목록·메타데이터·start 처리를 앱이 떠맡는다. 초기 권고는 **Lua 포함 libmpv + 내장 hook** | sol (설계 평가) |

채택할 DLL 은 받은 뒤 확인한다 (11절 체크리스트): 정상 URL 한 개 재생, `user-data/mpv/ytdl/path` 값, 로그의 `ytdl_hook` 줄.

## 4. yt-dlp 와 JS 런타임

### 4.1 yt-dlp

| 항목 | 값 | 출처 |
| --- | --- | --- |
| 최신 stable | **2026.10.26** (2026-10-26), 2026-11-01 확인 | https://github.com/yt-dlp/yt-dlp/releases/tag/2026.10.26 |
| 최신 nightly | 2026.10.29.232701 (2026-10-29). 확인 시점의 스냅샷이다 | https://github.com/yt-dlp/yt-dlp-nightly-builds/releases/tag/2026.10.29.232701 (sol) |
| 주기 | stable 은 대체로 월간, nightly 는 변경이 있는 날 UTC 자정 직전, master 는 push 마다. upstream 은 일반 사용자에게 nightly 를 권장한다. 90일 넘은 버전에는 갱신 권고 경고가 있다 | https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#update |
| Windows 파일 | `yt-dlp.exe`(x64, 권장), `yt-dlp_arm64.exe`, `yt-dlp_x86.exe`. onedir zip 도 있다. 2026.10.26 부터 공식 Windows 바이너리는 Win10+ | https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#release-files |
| Python | 단독 exe 는 인터프리터를 포함한다. 별도 설치 불필요 | https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#dependencies |
| 무결성 | 릴리스에 SHA2-256SUMS·SHA2-512SUMS 와 서명 파일이 있다 | https://api.github.com/repos/yt-dlp/yt-dlp/releases/latest (sol) |
| 갱신이 잦은 이유 | 유튜브가 서명·`n` 파라미터 풀이 코드, 클라이언트 구조를 자주 바꾼다. 기본 `player_client` 는 `tv_samsung,visionos,web` 이고 JS 런타임이 없으면 `web`·`tv_samsung` 이 빠진다. 일부 클라이언트는 `po_token` 이 필요하다 | https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md (sonnet) |

### 4.2 라이선스

| 대상 | 라이선스 | 출처 |
| --- | --- | --- |
| 소스·PyPI | Unlicense | https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/LICENSE |
| **`yt-dlp.exe`**(PyInstaller 결합물) | **GPLv3+**. 포함된 GPLv3+ 코드 때문이다 (판정 4). Python 등 다른 의존성 고지도 함께 들어간다 | https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#licensing , https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/THIRD_PARTY_LICENSES.txt |

- **GPL 은 허용한다** (사용자 결정, 2026-11-01). 동봉에 라이선스 장벽이 없다. 이 결정은 workbench-gpui 의 D-2("Zed GPL 코드는 복사·의존하지 않고 …")와 다르다. 차이와 LGPL 로 바꿀 때는 [05-WINDOWS.md](rust-video-player/05-WINDOWS.md) 5절, B 는 [09-distribution-licensing.md](rust-video-player-native/09-distribution-licensing.md).
- yt-dlp 는 **별도 프로세스**로 실행한다. 링크가 아니다. 이것이 앱 전체 배포 조건에 어떤 영향을 주는지는 법률 검토 사항이다 (일반 지식, sonnet). 이 문서는 판단하지 않는다.
- 배포할 때는 yt-dlp 와 번들 의존성의 라이선스 고지, 버전, 해시를 목록으로 같이 둔다 (설계 제안, sol).

### 4.3 JS 런타임 (EJS)

유튜브 전체 지원에는 `yt-dlp-ejs`(challenge 풀이)와 **외부 JS 런타임**이 필요하다. 공식 `yt-dlp.exe` 는 EJS 스크립트를 **내장**하지만 런타임은 따로 둔다. https://github.com/yt-dlp/yt-dlp/wiki/EJS

| 런타임 | 최소 버전 (판정 3) | 비고 |
| --- | --- | --- |
| **Deno** | **2.3.0** | 권장·기본 활성 |
| Node | 22.0.0 | `--js-runtimes node` 로 지정 |
| QuickJS | 2023-12-9 | QuickJS-NG 도 지원 |
| Bun | 1.2.11 ~ 1.3.14 | deprecated |

- argon 의 "Deno 2.0+, Node 20+" 는 쓰지 않는다 (판정 3).
- 이 숫자는 **지원 최소·범위**다. 각 런타임의 최신 릴리스는 미확인이다 (sol).
- 이 요구가 도입된 yt-dlp 버전은 **미확인**이다 (sonnet). 이전 PhantomJS 방식은 유튜브에 쓰지 않는다. README 241행 (sonnet).
- mpv 의 Lua/MuJS 는 이 외부 런타임을 **대신하지 않는다** (구조적 판단, sol).
- 런타임이 PATH 나 `yt-dlp.exe` 와 같은 폴더에 있으면 찾는다. 명시하려면 `ytdl-raw-options` 에 `js-runtimes=deno:C:/MyPlayer/tools/deno.exe` 를 넣는 구성을 검증한다. 값에 `:` 가 있을 때의 인용은 **미시험**이다 (sol, sonnet).
- 동봉 exe 에는 EJS 가 있어 `--remote-components` 로 추가 다운로드할 필요는 없다 (sol).
- Deno 의 크기는 출처가 없어 적지 않는다 (argon 의 30~50 MB 제외).

### 4.4 배포 위치와 갱신 설계

설계 제안이다. 검증된 yt-dlp 기능이 아니다 (sol, sonnet).

| 항목 | 설계 |
| --- | --- |
| 초기 동봉 | 설치본 `tools\yt-dlp.exe` + Deno + 라이선스 고지·버전·해시 목록. 절대 경로를 `ytdl_hook-ytdl_path` 에 준다 |
| 갱신본 위치 | `Program Files` 는 쓰기 권한 때문에 `-U` 가 안 된다. **`%LOCALAPPDATA%` 아래 버전별 tools 폴더**에 검증 후 새 버전을 놓고 **다음 세션부터** 전환한다 (sonnet, sol) |
| 갱신 수단 | `yt-dlp -U`(현재 채널 최신), `--update-to nightly`(채널 전환), `--update-to stable@2026.10.26`(고정·복귀). 자동 갱신이 안 되는 zip 형은 직접 교체한다 |
| 출처 제한 | 다른 저장소로 `--update-to` 도 가능하지만 upstream 이 검증 부재를 경고한다. 앱 updater 는 **공식 저장소만** 허용한다 |
| 시점 | 시작 시 또는 추출 오류 시. 이전 버전을 남겨 회귀 시 되돌린다. 탭마다 updater 를 중복 실행하지 않는다 |
| 첫 실행 | 단독 exe 는 첫 실행 때 압축 해제로 느릴 수 있다 (일반 지식, 미시험, sonnet) |

출처: https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#update . Tauri 는 sidecar/resource 로 묶을 수 있다. https://v2.tauri.app/develop/sidecar/ , https://v2.tauri.app/develop/resources/ . 앞으로 PoToken·SABR 같은 서버 쪽 변화로 추출이 막힐 수 있다 (**추정**, argon). 아래 12절.

## 5. 재생 설정

### 5.1 화질 format 예

모두 yt-dlp 문법으로 구성한 **앱용 제안 예**다. 영상·지역·로그인에 따라 제공 format 이 다르므로 전부 **미시험**이다. `+` 는 영상+음성 조합, `/` 는 왼쪽이 안 될 때의 대안이다. https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#format-selection

| UI 선택 | `ytdl-format` 값 예 | 의미·제약 |
| --- | --- | --- |
| 자동 | (빈 값) | yt-dlp 기본 선택. yt-dlp 가 바뀌면 결과도 달라질 수 있다 |
| 1080p 이하 | `bv*[height<=1080]+ba/b[height<=1080]` | 호환·저부하용 |
| 4K 이하 | `bv[height<=2160]+ba/b[height<=2160]` | 4K 제공 보장 아님 |
| VP9 | `bv[height<=2160][vcodec~='^vp0?9']+ba/b` | 코드 이름이 `vp09` 로도 `vp9` 로도 보고돼서 **정규식 `^vp0?9`** 를 쓴다 (판정 6, 실기 확인 필요) |
| AV1 | `bv[height<=2160][vcodec^=av01]+ba/b` | 없을 수 있다 |
| H.264 | `bv[height<=2160][vcodec^=avc1]+ba/b` | 4K AVC 는 항상 있지 않다. 호환 모드는 `height<=1080` 도 검토 |

- 필터 문법: https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#filtering-formats
- **고정과 선호를 구분한다.** 코덱을 엄격히 고정하면 없을 때 "format unavailable" 로 끝난다. 선호만 하려면 뒤에 `/bv[height<=2160]+ba/b` 를 더하고, UI 에 "VP9 선호"로 쓰고 실제 `video-codec`·`hwdec-current` 를 보여 준다 (sol).
- `best` 하나를 최고 분리 스트림과 같게 취급하지 않는다 (sol).
- 정렬(`format-sort`)을 `ytdl-raw-options` 로 넘길 때 값의 쉼표·콜론 인용은 **미시험**이다 (sonnet).
- `--vid=no` 이면 mpv 가 `bestaudio/best` 로 자동 바꾼다 (sonnet). `MPV41/DOCS/man/options.rst`

### 5.2 화질 변경은 재load

`ytdl-format` 변경은 이미 열린 스트림에 새 화질을 붙이지 않는다. 다음 순서로 재resolve 한다 (판정, sol).

1. 현재 위치·일시정지·R1 요청값·R2 필터를 세션에 저장한다.
2. 원본 watch URL 을 새 `ytdl-format` 으로 다시 `loadfile` 한다.
3. 로드 후 위치로 탐색하고 R1·R2 를 다시 적용한다.

`ytdl_hook-all_formats=yes` 로 화질들을 트랙으로 노출할 수 있지만, mpv 문서가 런타임 전환이 느리고 이 목적에 맞지 않는다고 한다. `MPV41/DOCS/man/options.rst` (sol). 기본 `reset-on-next-file` 정책에 우연히 의존하지 않는다 (sol).

### 5.3 자막

| 단계 | 수단 | 출처 |
| --- | --- | --- |
| yt-dlp 에 요청 | `ytdl-raw-options` 에 `write-subs=`, `sub-langs=…`, 자동 생성은 `write-auto-subs=` | https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md#subtitle-options |
| mpv 에 연결 | 훅이 JSON 의 `requested_subtitles` 를 `sub-add` 로 붙인다. 기본 요청은 모든 언어 | `MPV41/player/lua/ytdl_hook.lua` 725~753 (sonnet, sol) |
| 재생 트랙 선택 | mpv `slang=ko,en` / `sid`. **위 요청과 다른 단계**다 | `MPV41/DOCS/man/options.rst` (sol) |

- 정식 옵션 이름은 `sub-langs`, `write-subs`, `write-auto-subs` 다. argon 의 `sub-lang`, `write-sub` 는 쓰지 않는다 (판정 7).
- 값에 쉼표가 있는 `sub-langs=ko,en` 은 mpv 목록의 경계로 오해된다. 이스케이프하거나 node 전달을 쓴다 (sol, **미시험**).
- flag 는 값 없는 키가 아니라 `=` 를 붙인다. 예 `write-subs=`. `MPV41/DOCS/man/options.rst`
- 한국어 수동·자동 자막은 수용 시험 대상이다 (sol).

### 5.4 캐시·버퍼

공식 기본값이 **아니라** 실험 시작값이다 (판정 9). 4K 보장 수치도 아니다.

| 옵션 | sol 제안 | argon 제안 | 비고 |
| --- | --- | --- | --- |
| `cache` | `yes` | `yes` | 기본은 `auto`(네트워크면 켜짐) |
| `cache-secs` | `20` | — | |
| `demuxer-max-bytes` | `128MiB` | `150MiB` | 앞쪽 패킷 캐시 |
| `demuxer-readahead-secs` | — | `60` | |
| `demuxer-max-back-bytes` | 따로 정함 | `50MiB` | 뒤로 탐색용 |

- 패킷 캐시와 디코드·GPU 프레임 메모리는 다르다. 탭 메모리 상한과 혼동하지 않는다 (sol). 탭 정책은 [01-ARCHITECTURE.md](rust-video-player/01-ARCHITECTURE.md) 4절.
- 옵션 설명: `MPV41/DOCS/man/options.rst` 의 `--cache`, `--demuxer-max-bytes` 등 (sonnet, sol). 기본값 수치는 확인하지 않았다 (**미확인**).
- 로딩 표시는 `paused-for-cache`, `demuxer-cache-state`, `cache-buffering-state` 를 관찰한다. `cache-secs` 를 "재생 전 전량 채우기 시간"과 같게 보지 않는다 (sol). `MPV41/DOCS/man/input.rst`

### 5.5 탐색

- VOD: 기존 mpv `seek` 정책을 그대로 쓴다. 캐시 안은 빠르고 밖은 HTTP Range 재요청이 든다 (sonnet). 유튜브 DASH 가 Range 를 지원하는지는 일반 지식, 미시험이다.
- 캐시 밖 앞쪽 탐색의 지연("수백 ms")은 argon 의 추정값이다. 출처 없음.
- 라이브·종료된 라이브·서명 만료 URL 의 탐색은 별도 수용 시험이다 (sol).

## 6. R1·R2 와의 관계

### 6.1 공통

mpv 에는 유튜브 스트림이 HTTPS 의 MP4/WebM 로 보일 뿐이다. 그래서 `hwdec`, `vf`, 밝기·대비·채도, `deinterlace` 설정이 로컬 파일과 같다. https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst (sonnet, sol). 구현은 [05-WINDOWS.md](rust-video-player/05-WINDOWS.md) 1절과 [06-FILTERS.md](rust-video-player/06-FILTERS.md), B 는 [06-r1-hwdec.md](rust-video-player-native/06-r1-hwdec.md)·[07-r2-video-options.md](rust-video-player-native/07-r2-video-options.md).

### 6.2 코덱별 하드웨어 디코딩

**일반 지식이다. 공식 지원 표는 열지 않았다 (미확인).** 이 표는 설계 판단의 참고용이다.

| 코덱 | 하드웨어 디코딩 (argon, 일반 지식) | 미지원 GPU 에서 |
| --- | --- | --- |
| H.264 (`avc1`) | D3D11VA 가 거의 모든 GPU 에서 가능. 가장 안전. 유튜브는 보통 1080p 까지만 제공한다는 서술이 있다 (argon) | — |
| VP9 (`vp09`) | Intel 7세대+, NVIDIA GTX 950·10 시리즈+, AMD RX 400+ | 소프트웨어로 전환 (R1 폴백) |
| AV1 (`av01`) | Intel 11세대+·Arc, NVIDIA RTX 30+, AMD RX 6000+. Windows 의 AV1 확장이 필요할 수 있다 (sonnet) | dav1d 소프트웨어. 4K 에서 CPU 부하가 크다 |

- 코덱을 AV1 로 고정하고 HW ON 이어도 GPU 가 지원하지 않으면 소프트웨어가 된다. 요청값·`hwdec-current`·코덱·해상도를 함께 표시한다 (sol). https://mpv.io/manual/stable/#video
- 설계 제안: 하드웨어가 안 되는 환경에서 4K AV1·VP9 를 고르면 끊긴다. `hwdec-current` 와 프레임 드롭 속성으로 판단해 화질을 낮추도록 안내한다 (sonnet).
- "화질 선택"과 R1 이 연결된다. 화질을 바꾸는 재load 는 5.2 절의 절차를 따른다.

### 6.3 CPU 필터와 copy-back

- `vf` 가 CPU 필터이면 `hwdec=d3d11va`(무복사)에서도 프레임이 CPU 로 내려와 성능이 떨어질 수 있다. mpv 문서의 `-copy` 설명과 일치한다 (일반 지식, sonnet). `MPV41/DOCS/man/options.rst`
- argon 은 `d3d11va` 에서 CPU `vf` 가 "불가능하거나 복사 모드로 강등된다"고 했다. 이 문서는 mpv 문서에 맞춰 **copy-back 이 생긴다**는 표현으로 쓴다 (판정 문서 1절의 "CPU `vf` 필터는 copy-back 을 부른다").
- 고해상도 소프트웨어 AV1 + 노이즈 제거의 실시간 성능은 **미확인**이다 (sol).
- 유튜브라서 FFmpeg GPL 필터 조건이 바뀌지는 않는다. `hqdn3d`·`eq` 는 GPL 의존을 확인한다 ([06-FILTERS.md](rust-video-player/06-FILTERS.md) 4절).
- 재load 때 세션에 저장할 값: `requested_hwdec`, 필터, `quality_policy`, `original_url`, `last_position`, pause. 요청값을 실제 적용 성공으로 오인하지 않는다 (sol).

### 6.4 묶음 B 1단계(glow)의 copy-back — 실측 항목

B 의 1단계는 eframe glow + libmpv OpenGL render API 다. eframe 0.37 의 Windows GL 은 기본 WGL 이고 mpv OpenGL API 에서 직접 하드웨어 interop 을 쓰려면 ANGLE 이 필요하므로 대부분 `d3d11va-copy`(copy-back)가 된다. [03-rendering-pipeline.md](rust-video-player-native/03-rendering-pipeline.md) 2.3절, https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h (sol).

| 실측 항목 | 이유 |
| --- | --- |
| 4K VP9·AV1 유튜브를 glow 경로에서 재생할 때 CPU 사용률·프레임 드롭 | 유튜브는 고화질에서 VP9·AV1 이 주력이다. 계산상 4K60 NV12 8-bit 는 약 0.75 GB/s 의 프레임 데이터량이다 (계산값, [03-GPUI.md](rust-video-player/03-GPUI.md) 3.1절 표) |
| `d3d11va` vs `d3d11va-copy` 차이 | glow 에서 무복사가 되는지 |
| 1080p H.264 선택으로 낮춘 경우의 여유 | 5.1 절 "1080p 이하"가 완화책인지 |

## 7. 오류·상태 처리

### 7.1 libmpv 이벤트

| 이벤트·속성 | 의미 | 처리 | 출처 |
| --- | --- | --- | --- |
| `START_FILE` → `FILE_LOADED` | `FILE_LOADED` 는 헤더를 읽고 디코딩이 시작되는 시점이다. 끝까지 재생 성공을 보증하지 않는다 | "로딩" → "재생" 전환 표시 | https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/client.h (sol) |
| `END_FILE` reason=`ERROR` | `error` 필드는 **대략적인** mpv 오류 코드다. 연령·지역·봇 같은 사유를 담지 않는다 (예 `LOADING_FAILED` -13, `UNKNOWN_FORMAT` -17) | 실패 상태로. 사유는 로그·속성에서 찾는다 | 같음 (sonnet) |
| `END_FILE` reason=`REDIRECT` | 재생목록 전환이다. **실패가 아니다** | 실패 팝업을 띄우지 않는다. `playlist_entry_id` 와 insert id 로 연관시킨다 | 같음 (판정, sol) |
| `mpv_command_async` 완료 | 명령 접수·완료이지 재생 성공이 아니다 | 구분한다 | 같음 (sol) |
| `media-title` | 훅이 yt-dlp 의 `title` 을 `force-media-title` 로 넣는다 | 탭 제목을 교체한다. 실패 전에는 없을 수 있으니 URL·video ID 를 대체값으로 | `MPV41/DOCS/man/input.rst` 2176 (sonnet, sol) |

- UI 스레드에서 네트워크 로드를 동기 대기하지 않는다. wakeup 콜백은 신호만 전하고 이벤트 루프에서 읽는다. generation 이 지난 로그·메타데이터가 새 탭에 붙지 않게 한다 ([01-ARCHITECTURE.md](rust-video-player/01-ARCHITECTURE.md) 2.3). 재시도는 횟수를 제한하고 무한 reload 는 하지 않는다 (sol, 설계 제안).

### 7.2 상세 사유는 로그와 속성에서

| 수단 | 내용 | 출처 |
| --- | --- | --- |
| 로그 메시지 | `mpv_request_log_messages` 로 받는다. 훅은 yt-dlp 의 stderr 를 오류 로그로 내고 "youtube-dl failed: …"(실행 파일 없음·권한, JSON 파싱 실패, 예기치 않은 오류)를 낸다 | `MPV41/player/lua/ytdl_hook.lua` 975~1000 (sonnet) |
| 로그 접두사 | 스크립트 이름(예 `ytdl_hook`)일 것이나 정확한 접두사는 **미확인**이다. argon 은 `[ytdl_hook]`·`[ffmpeg]` 라고 했다 (미확인) | — |
| 속성 `user-data/mpv/ytdl/json-subprocess-result` | yt-dlp 실행 결과(status/stdout/stderr). 오류 분류에 가장 직접적이다 | `MPV41/DOCS/man/input.rst` 3831~3840 (sonnet) |
| 속성 `user-data/mpv/ytdl/path` | 비어 있으면 yt-dlp 를 못 찾은 것 | 같음 |

- 로그 문자열로 사유를 분류하는 것은 **추정적 진단**이다. `end-file` 이 연령·지역·봇을 안정된 enum 으로 주지 않는다 (sol). 정제한 원문과 "다시 시도" 버튼을 같이 보인다.
- `json-subprocess-result` 의 stdout 에는 서명된 미디어 URL 이 들어 있을 수 있다 (추정). 화면·로그·이슈 보고에 그대로 남기지 않는다 (설계 제안).

### 7.3 사유별 표시

| 경우 | 알려진 한계 | UI 처리 제안 | 출처 |
| --- | --- | --- | --- |
| 연령 제한·비공개·멤버 전용 | 계정이 필요한 일부 콘텐츠. 쿠키를 넣어도 권한 없는 계정이 허용되는 것은 아니다 | "로그인·권한 필요" 와 "없는 영상" 을 구분한다 | https://github.com/yt-dlp/yt-dlp/wiki/Extractors#youtube |
| 지역 제한 | 제한 우회를 자동 재시도 정책으로 만들지 않는다 | 사유만 표시 | https://developers.google.com/youtube/terms/developer-policies (sol) |
| 봇 차단·rate limit·PO token | 최신 yt-dlp·EJS 여도 실패할 수 있다. EJS 가 모든 token 문제를 풀지 않는다 | 갱신 안내, 쿠키 옵션 안내 | https://github.com/yt-dlp/yt-dlp/wiki/EJS |
| 추출 성공 후 403·전송 실패 | 추출 URL 은 IP·헤더·쿠키 조건이 있다 | **원본 watch URL** 로 제한된 횟수만 재resolve | https://github.com/yt-dlp/yt-dlp/wiki/FAQ |
| yt-dlp 없음·실행 실패 | 경로·권한·백신 | 설치 상태 점검 화면 | `ytdl_hook.lua` (sonnet) |

yt-dlp 의 구체적 오류 문구 목록은 공식 문서에서 확인하지 못했다 (**미확인**). "Sign in to confirm you're not a bot" 같은 문구는 보고서(sonnet·argon)가 든 예다.

### 7.4 쿠키

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 전달 | `ytdl-raw-options=cookies-from-browser=edge`(chrome·firefox 가능), 또는 `cookies=<파일>`. mpv 로 넘기는 방식은 추정, 미시험 | https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/README.md (sonnet, sol) |
| 효과 | 완화될 수 있으나 **보장 없음.** Windows 의 암호화·프로필 잠금·권한별 성공은 **미확인** | https://github.com/yt-dlp/yt-dlp/wiki/FAQ#how-do-i-pass-cookies-to-yt-dlp (sol) |
| 위험 | upstream 이 쿠키를 쓰는 계정의 차단 위험과 쿠키 회전을 경고한다. 로그인 쿠키를 넘기면 클라이언트 선택도 달라진다 | https://github.com/yt-dlp/yt-dlp/wiki/Extractors#exporting-youtube-cookies |
| 정책 | **사용자가 명시적으로 켠 때만** 사용한다 (설계 제안) | sol |

### 7.5 저장·로그 금지 항목

| 저장하는 것 | 저장·로그하지 않는 것 |
| --- | --- |
| 원본(정규) watch URL, video ID, 시작 위치, 화질 정책 | 서명된 미디어 URL (재개는 원본 URL 재추출 + 저장 위치로 탐색) |
| | 쿠키 내용, 토큰, 브라우저 프로필 경로 |
| | 인증이 든 `ytdl-raw-options` 값 |

근거: 추출 URL 은 IP·헤더·쿠키 조건이 있을 수 있어 영구 식별자로 쓰지 않는다. https://github.com/yt-dlp/yt-dlp/wiki/FAQ (sol, 설계 제안).

## 8. 링크 판별과 탭 연결

### 8.1 URL 검증

- Rust `url` 크레이트로 scheme·host·path·query 를 분리해 **host 를 정확히 비교한다.** 문자열 `contains("youtube.com")` 는 `youtube.com.attacker.example` 을 잘못 받는다 (판정, sol). https://docs.rs/url/latest/url/struct.Url.html
- 허용: HTTPS, `youtube.com`·`www.`·`m.`·`music.` 하위, `youtu.be` (sonnet 의 목록 + sol 의 "정확한 허용 subdomain"). 앱이 허용한 URL 만 ytdl 에 보낸다. 알 수 없는 scheme 과 임의 extractor 옵션은 받지 않는다.
- mpv 훅의 기본 `include` 패턴은 `^%w+%.youtube%.com/`, `^youtube%.com/`, `^youtu%.be/` 와 twitch 다. 훅은 이름과 달리 다른 사이트도 처리한다. 앱이 유튜브만 허용할지 정책으로 정한다. `MPV41/DOCS/man/options.rst` 927~929 (sonnet, sol).
- `url` 크레이트 검증은 **구현 설계**다. yt-dlp 추출기가 모든 입력을 안전한 유튜브 링크로 제한한다는 보장은 아니다 (sol).
- 모든 `https://…` 링크를 훅에 맡기려면 `try_ytdl_first=yes` 가 있으나 느려진다. 유튜브만 다루면 필요 없다. `MPV41/DOCS/man/options.rst` 896 (sonnet).

### 8.2 URL 종류

앱 라우팅 **설계 제안**이며 유튜브 URL 규격을 새로 정의하지 않는다. 실제 지원은 yt-dlp 버전에 따라 바뀐다. https://github.com/yt-dlp/yt-dlp/blob/2026.10.26/yt_dlp/extractor/youtube/_video.py (sol)

| 입력 | 라우팅 | 보존·주의 |
| --- | --- | --- |
| `youtube.com/watch?v=ID` | 새 영상 탭, 소스 `Url` | 원본 URL·ID·시작·화질·R1·R2 |
| `youtu.be/ID` | 같은 경로로 정규화 | query 의 `t`·`list` 를 버리기 전에 해석 |
| `youtube.com/shorts/ID` | 같은 플레이어 탭 | 세로 영상은 디코더 메타데이터로 표시 |
| `youtube.com/live/ID`, `/embed/ID` | 같은 경로 | 라이브는 seek 범위가 다르다 |
| `watch?v=ID&list=LIST` | **클릭한 영상 1개** | 재생목록은 사용자가 따로 선택 |
| `youtube.com/playlist?list=LIST` | 탭 하나 + 큐 또는 목록 UI | 수백 항목을 수백 세션으로 만들지 않는다 |

### 8.3 재생목록: 기본은 단일

- 훅은 기본으로 `--no-playlist` 를 넘긴다. 영상+재생목록 URL 에서는 영상 하나만 재생한다. 순수 재생목록 URL 까지 단일로 줄인다고 가정하면 안 된다 (sol). `MPV41/player/lua/ytdl_hook.lua` 879~916 (sonnet).
- argon 의 "`no-playlist=` 를 추가해 단일 강제"는 이미 기본 동작이라 쓰지 않는다 (sonnet·sol 의 소스 읽기).
- 재생목록 지원은 `ytdl-raw-options` 에 `yes-playlist=` 를 켠다. 훅이 mpv 내부 재생목록(M3U 형태)으로 바꾸므로 앱의 탭 목록과 구조가 다르다. 항목 URL 은 `https://youtu.be/ID` 로 만들어진다 (sonnet).
- 정책 제안: **한 탭 안의 큐**를 첫 선택으로 한다. 여러 탭 열기를 주더라도 목록을 먼저 보이고 제한된 수만 지연 open 한다. 최대 항목 수·취소·비공개/삭제 항목·현재 인덱스를 설계한다 (sol).

### 8.4 시작 시각 `t=`: 한 경로

| 보고서 | 주장 |
| --- | --- |
| argon | 훅이 URL 의 `t=` 를 직접 파싱한다 |
| sonnet·sol | 훅은 yt-dlp JSON 의 `start_time`/`section_start` 가 있고 사용자가 `start` 를 정하지 않았을 때 `file-local-options/start` 로 반영한다. URL 의 `t=` 를 yt-dlp 가 `start_time` 으로 해석하는지는 미확인 |
| **판정 5** | **sonnet·sol 서술을 채택한다.** 앱이 `t=` 를 파싱해 `start=` 로 넘기는 **한 경로**로 정하고 이중 적용을 피한다 |

- 앱은 `t=90`, `t=1m30s`, `start=` 를 초로 바꿔 load 때 `start` 로 넘긴다. 예: `loadfile URL replace -1 start=SEC` (sonnet, 문법은 mpv 버전별 확인 필요, **추정**). `MPV41/DOCS/man/input.rst`
- 명시한 mpv `start` 가 있으면 훅이 덮어쓰지 않는다 (sol). 그래도 두 곳이 함께 적용돼 90초가 더해지는 일이 없는지는 **시험 대상**이다. 확실히 하려면 load 에 쓰는 URL 에서 `t=` 를 뺀다 (설계 제안, 미시험).
- 훅의 start 반영 위치: `MPV41/player/lua/ytdl_hook.lua` 806~811 (sonnet).

### 8.5 탭 연결

- 같은 `PlayerSession` 을 쓰고 소스 종류만 `Url` 이다. `tab_id`·`session_id`·generation 을 나눈다. 제목이 늦게 와도 처음 연 탭에 붙인다 (sol).
- 같은 영상을 다시 클릭할 때 새 탭을 열지 기존 탭을 활성화할지는 앱 정책이다. R3 문구대로 기본은 **새 탭**을 제안한다 (sol).
- 최근 목록에는 원본 URL·ID·시작 위치만 둔다 (7.5).
- 탭 썸네일은 `ytdl_hook-thumbnails=best|all` 로 얻지 못한다. 그 옵션은 이미지를 영상 트랙으로 붙인다. 필요하면 별도 메타데이터 조회(`thumbnail` 필드)를 쓰되 추출 요청이 중복되지 않게 한다 (sol, argon 일치).

## 9. 묶음별 적용

### 9.1 묶음 A — gpui·Tauri

링크 클릭 자리 (설계 제안, A 의 01-ARCHITECTURE 1.3a 는 "텍스트·미리보기 탭의 유튜브 링크"를 진입으로 정했다).

| 앱 | 클릭 자리 | 처리 | 참고 |
| --- | --- | --- | --- |
| gpui | 텍스트 탭(`EditorPoc`)의 링크, 주소 입력, 붙여넣기 | 링크 판별(8절) 후 `PaneItem::Video` 를 새 탭에 추가하고 세션이 `loadfile` | [03-GPUI.md](rust-video-player/03-GPUI.md) 2절 |
| Tauri | 웹(Solid) 쪽 링크 클릭 이벤트 | 프런트가 URL 을 검증하지 않고 **Rust 쪽에서 `url` 크레이트로 검증**한 뒤 세션 생성. IPC 는 세션 ID 만 오간다 | [04-TAURI.md](rust-video-player/04-TAURI.md) 2절 |

- 화면 출력(자식 HWND 등)은 로컬 파일과 같다. 유튜브라고 출력 경로가 바뀌지 않는다 ([03-GPUI.md](rust-video-player/03-GPUI.md) 3절, [04-TAURI.md](rust-video-player/04-TAURI.md) 4절).
- Tauri 에서 `<video>` 로 유튜브 주소를 직접 재생할 수 없다. 추출 URL 은 만료·헤더 조건이 있고 `<video>` 한계는 [04-TAURI.md](rust-video-player/04-TAURI.md) 3절에 있다.
- 앱의 CSP 에는 영향이 없다. 네트워크는 Rust 쪽 libmpv·yt-dlp 가 하기 때문이다 (추정).

#### Tauri iframe 대안 — R3 대체안으로는 부적합

공식 YouTube IFrame Player API 를 WebView2 에 넣는 방식이다. https://developers.google.com/youtube/iframe_api_reference

| 구분 | 내용 |
| --- | --- |
| 장점 | 공식 경로. yt-dlp·JS 런타임·ffmpeg 가 필요 없다. 갱신 부담이 없다. 소리·자막·화질 UI 를 유튜브가 준다 (sonnet, sol) |
| **R1·R2 불가** | 하드웨어 디코딩·필터·셰이더를 앱이 제어하는 항목이 API 에 없다. 디코딩은 Chromium 이 정한다. 교차 출처 iframe 의 `<video>` 에 필터를 걸 수 없다. iframe 요소 전체에 CSS 필터를 거는 것은 sharpen·denoise·deinterlace 를 해결하지 못한다 (sonnet, sol, argon 일치) |
| 제약 | 소유자가 임베드를 금지한 영상은 재생 불가(오류 101·150). Referer 가 없으면 오류 153. 비공개·삭제·자동재생 정책으로 막힐 수 있다. `setPlaybackQuality` 등은 더 이상 지원되지 않는다 |
| UI | 광고·추천 UI 가 보이고 자체 컨트롤 통일이 어렵다. "같은 플레이어 화면"이라는 R3 와 맞지 않는다 |
| 식별 | 임베드 플레이어는 HTTP Referer 나 동등한 클라이언트 식별을 요구한다. Tauri 의 dev/prod origin·CSP·Referer 는 배포에서 검증한다 (**미확인**) |

결론: "R1·R2 없는 안전한 대체 재생" 용 보조 모드로만 의미가 있다. 필요하면 "공식 플레이어로 열기/브라우저에서 열기"를 별도 모드로 두고 재생 엔진이 다름을 보여 준다 (sol). 플레이어의 광고·브랜딩·컨트롤을 가리거나 바꾸는 방식은 API 정책 확인이 먼저다 (sol). WebView2 메모리 증가량은 출처가 없어 적지 않는다. https://developers.google.com/youtube/terms/required-minimum-functionality#api-client-identity-and-credentials

### 9.2 묶음 B — egui

| 항목 | 내용 | 참고 |
| --- | --- | --- |
| 링크 클릭 자리 | egui 의 텍스트·`Hyperlink` 류 위젯, 주소 입력, 붙여넣기. 탐색기 트리 활성화는 `Command::Open` 으로 새 `SessionId` 탭을 만든다. 링크도 같은 `Command` 경로로 보낸다 (설계 제안) | [05-explorer-tabs.md](rust-video-player-native/05-explorer-tabs.md) 2절 |
| 코어 | 링크 판별(8절)은 코어 크레이트 안에서 한다. UI 는 URL 문자열만 보낸다 | [04-player-core.md](rust-video-player-native/04-player-core.md) |
| 표면 | 탭마다 mpv core 하나, glow paint callback 으로 그린다. 유튜브라도 같다 | [03-rendering-pipeline.md](rust-video-player-native/03-rendering-pipeline.md) |
| iframe | egui 앱에는 WebView 가 없어 쓸 수 없다 (추정, sonnet) | — |
| 숨김 탭 | 가려진 유튜브 탭은 일시정지한다. 네트워크 스트림을 계속 읽는 비용에 주의한다 (설계 제안) | [05-explorer-tabs.md](rust-video-player-native/05-explorer-tabs.md) 5절 |
| 1단계 copy-back | 6.4 절 | [06-r1-hwdec.md](rust-video-player-native/06-r1-hwdec.md) 2절 |

## 10. 약관 (사실만, 법률 판단 없음)

이 절은 공식 문서의 관련 항목을 짧게 옮긴 **사실 목록**이다. 이 앱이 적법한지 위반인지, 개인 사용과 배포의 법적 차이는 **판단하지 않는다.**

### 10.1 서비스 약관

| 항목 | 내용 |
| --- | --- |
| 출처 | https://www.youtube.com/t/terms (한국어, 발효일 2022-01-05, sonnet), https://www.youtube.com/t/terms?gl=GB (영문, sol) |
| 이 판이 최신인지 | **미확인** (2026-11-01 개정판이 있을 수 있다). 영문 원문과 한국어 판을 대조하지 않았다 |
| 권한·제한 (1) | 서비스·콘텐츠의 접근·복제·다운로드·배포·전송·변경 등의 사용은 서비스가 허용한 경우, YouTube 와 필요한 권리자의 사전 서면 허가가 있는 경우, 또는 적용 법률이 허용하는 경우를 제외하고 금지한다 |
| 권한·제한 (2) | 복제·사용을 막는 기능을 포함해 보안·사용 제한 기능의 우회·무력화·방해(시도 포함)를 금지한다 |
| 권한·제한 (3) | 자동화된 수단(로봇, 봇넷, 스크래퍼 등)으로 서비스에 접근하는 것을 제한한다. 공개 검색엔진의 robots.txt 준수, 사전 서면 승인, 적용 법률 등의 예외가 있다 |
| 개인 사용 | 본문은 개인적·비상업적 시청·청취와 embeddable player 표시를 설명한다. 개인 사용자에게 주어진 설명과 앱 개발자의 배포·상업화·자동 추출에 관한 조항은 **구분해서 기록한다** |

### 10.2 개발자 정책(API 서비스)

- 출처: https://developers.google.com/youtube/terms/developer-policies III.I (sol). 별도 문서인 API 서비스 약관(https://developers.google.com/youtube/terms/api-services-terms-of-service)은 열지 않았다 (**미확인**).
- III.I 에는 플레이어·광고의 수정이나 차단, 영상·음성 구성요소의 분리·변경, 사용자가 보지 않는 탭·화면의 백그라운드 플레이어, 지역 제한 우회 등에 관한 제한이 있다.
- 이 조항이 libmpv+yt-dlp 앱에 어떻게 적용되는지는 별도 판단 대상이며 **확정하지 않는다.** 확인한 것은 조항의 존재뿐이다.
- argon 의 "독립 플레이어를 금지하고 IFrame 사용을 요구한다", DMCA 1201조 위반 위험, 소송 선례에 관한 서술은 출처 문구를 확인하지 못해 **싣지 않는다** (판정 8).

### 10.3 라이선스와의 구분

- 앱을 배포할 때는 약관 외에 yt-dlp·번들 의존성·libmpv/FFmpeg 의 고지·소스·재배포 조건을 기록한다 (sol). [05-WINDOWS.md](rust-video-player/05-WINDOWS.md) 5절.
- 오픈소스 라이선스는 유튜브 콘텐츠 사용 허가를 대신하지 않는다 (sol, 사실 구분).
- 약관의 "다운로드" 금지 문구와 mpv 의 스트리밍 재생(저장하지 않음)의 해석 차이는 법률 판단이 필요해 다루지 않는다 (sonnet).
- 확인된 배경: yt-dlp 저장소는 활성이다 (별 204,244, 2026-10-29 push, sonnet 확인값). 유튜브용 JS 해독 컴포넌트를 쓴다.

## 11. 검증 체크리스트 (실행하지 않았다)

| # | 항목 | 확인 방법 | 기대·판정 |
| --- | --- | --- | --- |
| 1 | 채택 libmpv DLL 에 Lua 포함 | 정상 URL 재생, `user-data/mpv/ytdl/path` 값, 로그의 `ytdl_hook` 줄 | 값이 비어 있지 않다 |
| 2 | `ytdl_hook-ytdl_path` 절대 경로 | 동봉 `yt-dlp.exe` 로 watch URL 재생 | 재생 시작 |
| 3 | `load-scripts` 기본값에서 동작 | 옵션을 건드리지 않고 재생 | 3.2 절 판정과 일치 |
| 4 | JS 런타임 없음/있음 | Deno 제거 후 재생 시도 | 실패 사유가 로그에 남는다 |
| 5 | `js-runtimes=deno:C:/…` 인용 | raw option 으로 경로 전달 | `:` 가 든 값이 전달된다 |
| 6 | `ignore-config=` 효과 | 사용자 yt-dlp config 를 만들어 두고 재생 | config 가 섞이지 않는다 |
| 7 | 화질 format 6종 | 같은 영상에서 5.1 절 값 비교 | 제공 format·코덱 확인 |
| 8 | VP9 정규식 | `vcodec~='^vp0?9'` | VP9 format 이 잡힌다 |
| 9 | 자막 | `sub-langs`·`write-auto-subs`, 쉼표 이스케이프 | 한국어 트랙 생성 |
| 10 | 캐시 시작값 | 4K 재생, 앞/뒤 탐색 | 끊김·메모리 기록 |
| 11 | 화질 변경 재load | 위치·pause·R1·R2 보존 | 보존됨 |
| 12 | `t=` 한 경로 | `t=90`, `t=1m30s` 링크 | 90초 위치, 이중 적용 없음 |
| 13 | 재생목록 | `watch?v=…&list=…` / `playlist?list=` | 단일 / 큐 |
| 14 | 오류 분류 | 연령 제한·비공개·지역 제한 영상 | `json-subprocess-result`·로그에서 분류 |
| 15 | `REDIRECT` | 재생목록 전환 | 실패로 처리되지 않는다 |
| 16 | 쿠키 | `cookies-from-browser=edge` | 성공률, 오류 문구 기록 |
| 17 | R1 | 유튜브 H.264/VP9/AV1 에서 `hwdec-current` | 요청값과 실제값 구분 표시 |
| 18 | R2 CPU 필터 | 4K VP9 + denoise | copy-back 비용 기록 |
| 19 | 갱신 | 버전 폴더에 새 yt-dlp, 다음 세션 전환·롤백 | 동작 |
| 20 | Program Files 설치본 | `-U` 가 막히는지 | 막힘 확인 |
| 21 | 로그 | 서명 URL·쿠키가 로그에 남는지 | 남지 않는다 |
| 22 | B 1단계 | glow 에서 4K 유튜브 재생 | 6.4 절 항목 |

## 12. 확신 낮은 부분

| 항목 | 내용 |
| --- | --- |
| 실행 시험 없음 | 내장 hook 이 libmpv DLL 에서 실제 동작하는지, `load-scripts=no` 의 영향이 없는지는 소스 구조로 판단한 것이다 (sonnet, sol) |
| DLL 별 Lua | shinchiro 는 레시피로 확인. zhongfly(특히 LGPL 변형)와 mpv.io 안내 빌드는 **미확인** |
| 서비스 변화 | PoToken·SABR, 로그인·지역·봇 제한, 정상 작동 기간은 보장되지 않는다 (**추정**). JS 런타임 최소 버전도 바뀔 수 있다. nightly 최신값은 확인 시점의 스냅샷이다 |
| 요구 도입 버전 | yt-dlp 의 JS 런타임 요구가 도입된 버전, 현재 PO token 필요 빈도는 **미확인** |
| 코덱별 HW 표 | 공식 표 없이 일반 지식이다 (6.2) |
| 쉼표·콜론 인용 | `sub-langs`, `format-sort`, `js-runtimes` 값의 mpv 목록 인용은 **미시험** |
| `t=` | 유튜브 URL 의 `t=` 를 yt-dlp 가 어떻게 해석하는지 **미확인** |
| 로그 접두사 | `ytdl_hook` 로그의 정확한 접두사 **미확인** |
| `loadfile` 문법 | `replace -1 start=SEC` 의 인덱스 인자는 mpv 버전 확인 필요 (**추정**) |
| 쿠키 | Windows 의 Chrome/Edge 쿠키 읽기 성공률 **미확인**. 계정 접근은 하지 않았다 |
| 약관 | 개정판 여부, 영문·한국어 대조, API 서비스 약관은 **미확인**. 법률·배포 적법성은 판단하지 않았다 |
| 성능 | R1 전환 지연, 4K60 소프트웨어 폴백, 필터·자막 동시 성능, 동시 유튜브 탭 수, 적정 캐시, 긴 VOD 의 만료 복구는 모두 **미측정** |
| IFrame 문서 | 로케일별 reference 에 차이가 있었다 (sol). 최종 확인은 영문 최신 문서와 실제 오류로 한다 |
| mpv 태그 이후 | v0.41.0 이후 `git-release` 2026-06-04 표지 이후의 변경은 확인하지 않았다 (sonnet) |
