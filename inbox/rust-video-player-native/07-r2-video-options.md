---
tags: [rust, video-player, r2, filters, libmpv, ffmpeg]
created: 2026-11-01
updated: 2026-11-01
---

# 07. R2 — 영상 옵션 (선명·흐림·노이즈 제거·디인터레이스·밝기/대비/채도)

> 이 파일을 읽는 때: R2 의 여섯 기능을 구현하고, 슬라이더 값을 재생 중에 반영하는 방법을 정할 때.

라이선스는 GPL 을 허용한다(사용자 결정). 그래서 FFmpeg 의 GPL 필터(`eq`, `hqdn3d`)도 그대로 쓴다. LGPL 로 바꿀 때 무엇으로 대체하는지는 [09-distribution-licensing.md](09-distribution-licensing.md) 에 있다.

## 1. 실시간 조절의 두 방법 (mpv v0.41.0)

| 방법 | 동작 | 출처 |
| --- | --- | --- |
| **속성** | `brightness`, `contrast` 처럼 mpv 속성이면 `set_property` 로 바로 바꾼다 | [options.rst Equalizer](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| **필터 명령** | `vf-command <label> <command> <argument>`. 필터에 명령을 보낸다. "currently, this only works with the lavfi filter" | [input.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst) |
| **필터 교체** | `vf add @label:...` — "using an already used label will replace the existing filter". 명령을 지원하지 않는 필터의 강도는 이 방법으로 바꾼다 | input.rst `vf` |
| **켜고 끄기** | `vf toggle @label` — 레이블 붙은 필터의 enable/disable 플래그를 바꾼다. 처음부터 `!` 를 붙여 꺼진 상태로 넣어 둘 수 있다 | input.rst `vf` |
| **실패 시 되돌림** | "the commands will check whether the video chain is recreated successfully, and will undo the operation on failure" (영상이 구성되기 전에는 이 검사를 못 한다) | input.rst `vf` |

FFmpeg 필터 중 실행 중 명령을 받는 것은 필터마다 다르다. 아래 표의 "명령" 칸은 [FFmpeg Filters](https://ffmpeg.org/ffmpeg-filters.html) 에서 각 필터의 "Commands" 절이 있는지 확인한 결과다(2026-11-01).

## 2. 기능별 구현

| 기능 | 1차 수단 | 명령 | 강도 파라미터 | 대안 | 난이도 |
| --- | --- | --- | --- | --- | --- |
| 밝기·대비·채도 | mpv 속성 `brightness`/`contrast`/`saturation` (-100~100, 기본 0). 문서에 "Not supported by all video output drivers" 라는 단서가 있다 | 속성 | 각 값 | lavfi `eq` (GPL. brightness -1~1, contrast -1000~1000, saturation 0~3) | 하 |
| 디인터레이스 | mpv 속성 `deinterlace=<yes\|no\|auto>`. 켜면 보통 `bwdif` 를 넣는다. `auto` 는 FFmpeg 가 인터레이스로 판단한 프레임에만 적용한다(오판 가능) | 속성 | — (모드는 `bwdif` 옵션) | lavfi `bwdif`/`yadif` 직접. 2단계·Win32 경로에서는 `d3d11vpp` | 하~중 |
| 선명하게 | lavfi `cas` (Contrast Adaptive Sharpen) | 있음 | `strength` (기본 0) | lavfi `unsharp` (명령 없음 → 교체), `--glsl-shaders` | 중 |
| 흐리게 | lavfi `gblur` | 있음 | `sigma` (기본 0.5), `steps` | lavfi `unsharp` 의 음수 amount(교체) | 중 |
| 노이즈 제거 | lavfi `hqdn3d` (GPL, 공간+시간 3D) | 있음 | `luma_spatial`(기본 4.0), `chroma_spatial`, `luma_tmp` 등 | lavfi `nlmeans` (명령 없음, 무거움) | 중~상 |

출처: 밝기·대비·채도와 디인터레이스는 [options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)·[매뉴얼 Video](https://mpv.io/manual/stable/#video), 필터는 [FFmpeg Filters](https://ffmpeg.org/ffmpeg-filters.html), lavfi 브리지는 [vf.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst)·[매뉴얼 Video Filters](https://mpv.io/manual/stable/#video-filters), GPL 필터 목록은 [FFmpeg LICENSE.md](https://github.com/FFmpeg/FFmpeg/blob/master/LICENSE.md).

난이도는 세 보고서의 평가를 합친 상대 추정이다. 노이즈 제거는 보고서마다 중·상·중상으로 갈려서 "중~상"으로 적었다.

### 2.1 디인터레이스 주의

- mpv 의 `deinterlace` 를 켠 상태에서 사용자가 디인터레이스 필터를 또 넣으면 충돌한다. 문서에 "will conflict with any manually inserted deinterlacing filters" 라고 적혀 있다 ([options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)). 그래서 디인터레이스는 `deinterlace` 속성 **하나로만** 제어한다.
- `bwdif` 의 기본 모드 `send_field` 는 필드마다 한 프레임을 내므로 출력 프레임률이 두 배가 된다. A/V 동기와 표시 주사율을 확인한다 ([FFmpeg bwdif](https://ffmpeg.org/ffmpeg-filters.html)).
- `d3d11vpp` 는 "Requires a D3D11 context" 이므로 1단계(glow)에서는 쓸 수 없다 ([vf.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/vf.rst)).

## 3. 필터 체인 설계 (설계 제안)

처음 열 때 레이블을 붙인 필터를 **꺼진 상태로** 미리 넣어 둔다.

```text
vf add @r2denoise:!lavfi=[hqdn3d=luma_spatial=4]
vf add @r2blur:!lavfi=[gblur=sigma=0.5]
vf add @r2sharp:!lavfi=[cas=strength=0]
```

| 사용자 동작 | mpv 명령 |
| --- | --- |
| 노이즈 제거 ON/OFF | `vf toggle @r2denoise` |
| 강도 슬라이더 (명령 지원 필터) | `vf-command r2denoise luma_spatial <값>` |
| 강도 슬라이더 (명령 미지원 필터) | `vf add @r2sharp:lavfi=[unsharp=la=<값>]` (같은 레이블이면 교체) |
| 밝기 등 | `set_property("brightness", <값>)` |
| 초기화 | 각 레이블을 끄고, 속성을 0 으로 |

- 순서는 노이즈 제거 → 흐리게 → 선명하게 로 둔다. 노이즈를 먼저 줄여야 선명 효과가 노이즈를 키우지 않는다(세 보고서 B3 공통 제안).
- 슬라이더에서 나오는 명령은 디바운스해서 마지막 값만 보낸다([04-player-core.md](04-player-core.md)).
- 적용 결과는 `vf` 속성을 관찰해서 확인하고, 실패하면 UI 를 이전 값으로 되돌린다.
- `!`(disabled) 상태의 필터가 CPU 비용 없이 통과되는지, 그리고 그 필터 때문에 copy-back 이 강제되는지는 **미확인**이다. 실측해야 한다.

## 4. R1 과의 관계

lavfi 필터는 CPU 필터다. 직접 하드웨어 디코딩 중에는 copy-back 이 필요하다 ([options.rst `auto-copy`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)). 1단계는 대부분 이미 copy-back(`d3d11va-copy`)이라 추가 비용이 작을 것으로 예상하지만, 실측해야 한다. 4K·고프레임에서 `hqdn3d` 같은 CPU 필터의 비용도 실측 대상이다 ([10-validation-and-open-questions.md](10-validation-and-open-questions.md)).

## 5. GPU 셰이더 경로 (선택)

`--glsl-shaders` 는 `vo=libmpv` 에도 적용되는 GPU 렌더러 옵션이다 ([options.rst GPU renderer options](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst), [매뉴얼](https://mpv.io/manual/stable/#gpu-renderer-options)). `--glsl-shader-opts=param=value` 로 셰이더 파라미터를 넘길 수 있다(같은 문서). CPU 비용을 줄이려면 선명·흐림을 셰이더로 옮길 수 있다. 다만 셰이더 파일(GLSL)을 직접 만들거나 구해야 하고, 파라미터를 실행 중에 바꿀 때의 동작은 **미확인**이다. 그래서 1차 구현은 lavfi 로 하고, 셰이더는 성능이 부족할 때 쓰는 선택지로 둔다.

---

이전: [06-r1-hwdec.md](06-r1-hwdec.md) · 다음: [08-playback-features.md](08-playback-features.md)
