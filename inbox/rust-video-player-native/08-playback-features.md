---
tags: [rust, video-player, potplayer, libmpv, features]
created: 2026-11-01
updated: 2026-11-01
---

# 08. 재생 기능과 PotPlayer 대응표

> 이 파일을 읽는 때: PotPlayer 의 어떤 기능을 mpv 의 무엇으로 구현할지 찾을 때.

대응표는 **기능 이름 기준**이다. 메뉴 경로·단축키는 넣지 않는다(사용자 결정). "PotPlayer 출처" 칸이 비어 있는 기능은 PotPlayer 공식 페이지에서 확인하지 못한 것이다. 이런 기능은 이 프로젝트의 요구사항으로 넣은 것이다(sonnet followup-1. 공식 페이지: [potplayer.tv](https://potplayer.tv/)).

mpv 문서의 출처는 모두 v0.41.0 이다: 옵션은 [options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst), 명령·속성은 [input.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst), 필터는 [vf.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst) / [af.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/af.rst).

## 1. 재생

| 기능 | PotPlayer 출처 | mpv 구현 | 난이도 | 비고 |
| --- | --- | --- | --- | --- |
| 탐색 (빠른·정밀) | — | `seek <target> [flags]`: `keyframes`(빠름) / `exact`(느림). 기본은 상대 탐색이면 keyframes, 절대 탐색이면 exact | 중 | 슬라이더를 끄는 동안에는 keyframes, 놓을 때 exact 로 한 번 더 ([매뉴얼 Playback Control](https://mpv.io/manual/stable/#playback-control-1)) |
| 재생 속도 | — | `speed` (0.01~100). `--audio-pitch-correction` 이 켜져 있으면(기본) `scaletempo2` 를 자동으로 넣어 음 높이를 유지한다 | 하 | [매뉴얼 Playback Control](https://mpv.io/manual/stable/#playback-control) |
| 구간 반복 (A-B) | — | `ab-loop` 명령, `ab-loop-a`/`ab-loop-b` 속성 | 하 | |
| 반복 재생 | — | `loop-file` | 하 | |
| 북마크 | [OFF] "Bookmarks for favorite scenes or chapters" | 앱이 위치를 저장하고 `seek absolute` 로 이동한다. 챕터는 `chapter-list`·`chapter` 속성 | 중 | 저장은 `settings` 크레이트 |
| 재생목록 | — | `loadfile` (append), `playlist` 속성 | 중 | 탭과의 관계를 먼저 정한다(탭 = 세션 하나) |

## 2. 영상

| 기능 | PotPlayer 출처 | mpv 구현 | 문서 |
| --- | --- | --- | --- |
| 하드웨어 가속 (R1) | [OFF] "Hardware acceleration via DXVA, CUDA, and QuickSync" | `hwdec` | [06-r1-hwdec.md](06-r1-hwdec.md) |
| 디인터레이스 (R2) | 2차 출처만 있음 | `deinterlace` | [07-r2-video-options.md](07-r2-video-options.md) |
| 선명하게·흐리게·노이즈 제거 (R2) | 2차 출처만 있음 | lavfi `cas`·`gblur`·`hqdn3d` | 07 |
| 밝기·대비·채도 (R2) | 2차 출처만 있음 | `brightness`·`contrast`·`saturation` | 07 |
| 색조·감마 | — | `hue`·`gamma` (-100~100) | options.rst Equalizer |
| 화면 비율 | — | `video-aspect-override` | options.rst |
| 자르기 | — | `video-crop` | options.rst |
| 회전 | — | `video-rotate` (0~359) | options.rst |
| 사용자 셰이더 | 2차 출처만 있음 (PotPlayer 는 HLSL) | `glsl-shaders` (GLSL. PotPlayer 셰이더와 호환되지 않음) | options.rst |
| 3D 출력 | [OFF] | 범위 밖 | [00-requirements.md](00-requirements.md) |

## 3. 자막

| 기능 | PotPlayer 출처 | mpv 구현 | 비고 |
| --- | --- | --- | --- |
| SRT·ASS/SSA | [OFF] | 내장(libass). 트랙은 `sid`, 외부 파일은 `sub-add` 명령, 자동 로드는 `sub-auto` | ASS 를 egui 로 다시 그리지 않는다 (sol 보고서 B7) |
| SMI | [OFF] "Text subtitles in SMI and SRT formats" | FFmpeg 에 SAMI 디먹서가 있다 ([samidec.c](https://github.com/FFmpeg/FFmpeg/blob/master/libavformat/samidec.c)). mpv 가 `.smi` 를 자동 로드하는지는 `--help=sub-auto-exts` 로 확인해야 한다(미확인) | 한국어 SMI 의 인코딩은 `sub-codepage` |
| VobSub·Blu-ray 자막 | [OFF] | FFmpeg 디코더 경유(그림 자막) | 검증 필요 |
| 자막 싱크 | 2차 출처만 있음 | `sub-delay` | |
| 보조 자막 (두 개 동시) | — | `secondary-sid` | |

## 4. 오디오

| 기능 | PotPlayer 출처 | mpv 구현 | 비고 |
| --- | --- | --- | --- |
| 출력 장치 선택 | [OFF] | `audio-device` | |
| 음량·음소거 | — | `volume`, `volume-max`(100~1000), `mute` | |
| 트랙 선택 | — | `aid`, `track-list` 속성 | 다시 열었을 때 같은 ID 가 재사용된다고 가정하지 않는다 |
| 오디오 싱크 | 2차 출처만 있음 | `audio-delay` | |
| 정규화 | 2차 출처 (신뢰도 낮음) | `af` 에 lavfi `loudnorm`/`dynaudnorm` | 실시간 성능 확인 필요 ([FFmpeg Filters](https://ffmpeg.org/ffmpeg-filters.html)) |
| 이퀄라이저 | 2차 출처만 있음 | `af` 에 lavfi `equalizer`/`superequalizer`, 조절은 `af-command` | 같은 문서 |

## 5. 화면

| 기능 | PotPlayer 출처 | 구현 | 비고 |
| --- | --- | --- | --- |
| 전체 화면 | — | **앱이** 처리한다: 전체 화면 + 트리·탭 숨김. mpv 의 `fullscreen` 은 쓰지 않는다(render API 에서는 앱이 창을 소유한다) | egui 0.37 `ViewportCommand::Fullscreen(bool)` ([viewport.rs](https://github.com/emilk/egui/blob/0.37.0/crates/egui/src/viewport.rs)) |
| 항상 위 | — | 앱이 창 속성으로 처리한다. mpv `ontop` 은 mpv 소유 창에만 해당한다 | egui `ViewportCommand::WindowLevel` (같은 파일) |
| 스크린샷 | — | `screenshot`, `screenshot-to-file`, `screenshot-format` | `screenshot-raw` 로 앱이 직접 저장할 수도 있다 |
| 장면 미리보기 (탐색 막대 썸네일) | [OFF] "Snapshot previews of scenes" | 별도 mpv core 하나를 숨김으로 두고, 그 core 로 탐색해서 `screenshot-raw` 를 뜬다 (설계 제안) | 비용 실측 필요 |

## 6. 난이도 요약

| 난이도 | 기능 |
| --- | --- |
| 하 | 속도, 음량, 트랙, 구간 반복, 반복, 밝기·대비·채도, 회전·비율·자르기, 자막·오디오 싱크, 스크린샷 |
| 중 | 탐색 UX, 북마크, 재생목록·탭 관계, 디인터레이스(프레임률 두 배), 선명·흐림, R1 상태 기계, 전체 화면(DPI·모니터·복원) |
| 중~상 | 노이즈 제거(화질·CPU 비용), 장면 미리보기 |

세 보고서의 B7 을 합친 상대 추정이다.

---

이전: [07-r2-video-options.md](07-r2-video-options.md) · 다음: [09-distribution-licensing.md](09-distribution-licensing.md)
