---
tags: [rust, video-player, license, gpl, distribution, windows]
created: 2026-11-01
updated: 2026-11-01
---

# 09. 배포와 라이선스

> 이 파일을 읽는 때: libmpv DLL 을 어디서 구해 어떻게 링크하고, 어떤 라이선스 의무가 생기는지 확인할 때.

이 파일은 법률 자문이 아니다. 공식 라이선스 문서의 내용을 정리한 것이다.

## 1. 결정

**GPL 을 허용한다**(사용자 결정, 2026-11-01). 일반 libmpv 빌드(GPL)와 FFmpeg GPL 필터(`eq`, `hqdn3d` 등)를 그대로 쓴다. 앱을 배포할 때는 GPL 의무(소스 제공 등)를 지킨다.

## 2. 라이선스 사실

| 대상 | 내용 | 출처 |
| --- | --- | --- |
| mpv 기본 | "mpv as a whole is licensed under the GNU General Public License GPL version 2 or later" | [mpv Copyright](https://github.com/mpv-player/mpv/blob/master/Copyright) |
| mpv LGPL | meson 옵션 `-Dgpl=false` 로, GPL 전용 파일 없이 빌드하면 LGPLv2.1+. 다만 이 스위치가 "does not in itself create a LGPLv2.1+ license grant" | 같은 문서, [meson.options v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/meson.options) (`option('gpl', type: 'boolean', value: true …)`) |
| FFmpeg 기본 | LGPL v2.1+. GPL 부분을 켜면 FFmpeg 전체가 GPL v2+ 가 된다 | [FFmpeg LICENSE.md](https://github.com/FFmpeg/FFmpeg/blob/master/LICENSE.md), [FFmpeg Legal](https://ffmpeg.org/legal.html) |
| FFmpeg GPL 필터 (일부) | `vf_eq`, `vf_hqdn3d`, `vf_boxblur`, `vf_smartblur`, `vf_owdenoise`, `vf_vaguedenoiser`, `vf_mcdeint`, `vf_nnedi`, `vf_spp`/`uspp`/`pp7`/`fspp` 등 | FFmpeg LICENSE.md |
| Rust 크레이트 | egui·eframe 등은 MIT/Apache-2.0. 그렇다고 앱 배포 전체가 permissive 가 되지는 않는다 | [eframe](https://crates.io/crates/eframe), sol 보고서 B8 |
| Slint (참고) | Royalty-free / GPLv3 / 상용 중에서 고른다 | [Slint LICENSE](https://github.com/slint-ui/slint/blob/master/LICENSE.md?plain=1) |

인코더(x264·x265 등)는 디코딩에 필요 없다. 넣을 이유가 없다 (sol 보고서 B8).

## 3. libmpv DLL 구하기

| 출처 | 내용 | 주의 |
| --- | --- | --- |
| mpv 공식 CI 개발 빌드 ([git-release](https://github.com/mpv-player/mpv/releases/tag/git-release)) | `libmpv-…-x86_64-w64-mingw32-gpl.zip` 과 `…-lgpl.zip` 을 따로 제공한다. GPL 변형과 LGPL 변형이 나뉘어 있다 | **안정판이 아니다**(master 개발 빌드). 이름도 `v0.41.0-dev-…` 이다. mingw 로 빌드했다 |
| 서드파티 빌드 ([shinchiro/mpv-winbuild-cmake](https://github.com/shinchiro/mpv-winbuild-cmake/releases)) | `mpv-dev-x86_64-<날짜>-git-<해시>.7z` | git 빌드다. 서드파티다 |
| 직접 빌드 (meson) | v0.41.0 태그로 빌드한다 | 빌드 체인 준비 비용 |

**모순 주의**: 1단계는 "0.41 안정판 API" 를 전제로 한다. 그런데 Windows 용으로 바로 받을 수 있는 libmpv 는 주로 **개발 빌드**다. 공식 v0.41.0 릴리스에 Windows libmpv DLL 이 들어 있다는 근거는 찾지 못했다(sol followup-1). 그래서 이렇게 한다.

1. 개발 빌드를 쓰면 특정 빌드(날짜·해시)로 고정하고, `mpv_client_api_version()` 값을 기록한다.
2. 앱은 0.41 API(client API 2.5) 범위만 쓴다. D3D11 render API(2.6)는 2단계 전까지 쓰지 않는다 ([client-api-changes.rst](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst)).
3. 안정판 바이너리가 꼭 필요하면 v0.41.0 태그를 직접 빌드한다.

## 4. 크기 예시 (출처가 있는 수치만)

개발 기간과 앱 배포 크기 추정은 넣지 않는다(사용자 결정: **실측 필요**). 아래 값은 다운로드 압축 파일 크기이며, 설치 후 크기가 아니다.

| 파일 | 크기 (바이트) | 확인 |
| --- | --- | --- |
| `libmpv-v0.41.0-dev-g9e2ed65eb-…-x86_64-w64-mingw32-gpl.zip` | 65,629,702 | GitHub API, [git-release](https://github.com/mpv-player/mpv/releases/tag/git-release), 2026-11-01 |
| `libmpv-v0.41.0-dev-g9e2ed65eb-…-x86_64-w64-mingw32-lgpl.zip` | 56,163,691 | 같음 |
| `mpv-dev-x86_64-20261101-git-48fca90e8c.7z` | 31,681,465 | GitHub API, [shinchiro 20261101](https://github.com/shinchiro/mpv-winbuild-cmake/releases/tag/20261101) |

압축을 푼 `libmpv-2.dll` 의 크기는 확인하지 못했다. argon 이 적은 "약 110~115 MB" 는 출처가 없다(argon followup-1).

## 5. stable-msvc 와 링크

- 앱은 `x86_64-pc-windows-msvc` 로 빌드하고, libmpv 는 DLL 의 C ABI 로 연결한다.
- 공식 CI 빌드와 shinchiro 빌드는 mingw 로 만든 것이다. MSVC 링커에 필요한 import library(`.lib`)가 들어 있는지, `.dll.a` 를 쓸 수 있는지는 **미확인**이다. 없으면 `.def` 에서 `.lib` 를 만들어야 할 수 있다.
- libmpv2-sys 가 DLL·import library 를 어떻게 찾는지(환경 변수, 빌드 스크립트)는 확인하지 않았다 ([libmpv2](https://crates.io/crates/libmpv2)).
- 배포할 때는 `libmpv-2.dll` 을 exe 옆에 둔다. DLL 검색 순서 문제를 피하려면 앱 폴더에 두는 것이 안전하다.

## 6. LGPL 로 바꿀 때의 대체 수단

GPL 을 피해야 하는 상황이 오면 다음과 같이 바꾼다.

| GPL 요소 | LGPL 대체 | 근거 |
| --- | --- | --- |
| libmpv GPL 빌드 | `-Dgpl=false` + LGPL FFmpeg 로 빌드한 libmpv (공식 CI 의 `-lgpl.zip`) | [mpv Copyright](https://github.com/mpv-player/mpv/blob/master/Copyright), [git-release](https://github.com/mpv-player/mpv/releases/tag/git-release) |
| lavfi `eq` (밝기·대비·채도) | mpv 속성 `brightness`/`contrast`/`saturation` (렌더러에서 처리하므로 FFmpeg 필터가 아님) | [options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| lavfi `hqdn3d` (노이즈 제거) | lavfi `nlmeans`(무겁고 실행 중 명령 없음), 또는 GLSL 셰이더 | FFmpeg LICENSE.md 의 GPL 목록에 nlmeans 는 없다. [FFmpeg Filters](https://ffmpeg.org/ffmpeg-filters.html) |
| lavfi `boxblur`/`smartblur` | `gblur` (GPL 목록에 없음) | 같음 |

`-Dgpl=false` 일 때 mpv 안에서 빠지는 기능 목록은 확인하지 않았다. LGPL 로 바꿀 때 다시 조사한다.

---

이전: [08-playback-features.md](08-playback-features.md) · 다음: [10-validation-and-open-questions.md](10-validation-and-open-questions.md)
