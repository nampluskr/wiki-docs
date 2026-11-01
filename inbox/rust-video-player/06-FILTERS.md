---
tags: [rust, video-player, filters, mpv, ffmpeg, d3d11, potplayer]
created: 2026-11-01
updated: 2026-11-01
---

# 06 · 영상 옵션(R2)과 PotPlayer 대응

> 이 파일을 읽는 때: 선명·흐림·노이즈 제거·디인터레이스·밝기/대비/채도를 어떻게 만들지 정할 때. GPL 필터를 피해야 할 때. PotPlayer 기능과 대응시킬 때.

입구는 [README.md](README.md)다. R1(GPU 가속)은 [05-WINDOWS.md](05-WINDOWS.md) 1절.

## 1. 기능별 구현표

mpv 를 기준으로 쓴다. 판정 반영: mpv `sharpen` 속성은 `--vo=gpu` 전용이다. sol 이 권장한 `vo=gpu-next` 에서는 동작하지 않는다. `d3d11vpp` 필터에는 노이즈 제거 옵션이 없다.

| 기능 | mpv 속성·명령 | lavfi 필터 (CPU) | 셰이더 | D3D11 비디오 프로세서 |
| --- | --- | --- | --- | --- |
| 선명 | `sharpen` 속성은 **`vo=gpu` 전용**. `gpu-next` 에서는 아래로 대체 | `unsharp` (양수=선명). 런타임 명령 섹션 없음 → 그래프 재구성 | `glsl-shaders` 사용자 셰이더. 과하면 ringing·노이즈 증가 | edge enhancement 필터 enum. 지원·강도는 드라이버 의존 |
| 흐림 | `vf-command` 로 `gblur` 의 sigma 변경 (lavfi 필터 한정) | `gblur` (명령 지원). `boxblur` 는 GPL | 분리형 Gaussian 2-pass 셰이더 | 표준 enum 없음 |
| 노이즈 제거 | `vf add lavfi=[...]` | `hqdn3d` (**GPL**, 명령 지원), `nlmeans` (LGPL, 매우 느림) | 검증된 denoise 셰이더 (시간축은 이전 프레임 처리 필요) | mpv `d3d11vpp` 필터에는 **노이즈 제거 옵션이 없다.** (Windows API 의 NOISE_REDUCTION 필터 enum 은 별개이며 자체 파이프라인용, 드라이버 의존) |
| 디인터레이스 | `deinterlace=yes/no/auto`, 속성 `deinterlace-active` | `yadif`, `bwdif` | — | `d3d11vpp` 필터가 **디인터레이스·스케일·Super Resolution** 을 제공 |
| 밝기·대비·채도 | 속성 `brightness`/`contrast`/`saturation` (**-100~100**). 색조 `hue` 도 있다 (argon 보고) | `eq` (**GPL**) | 출력 단계 색 행렬 | brightness/contrast/saturation 필터 enum (지원 caps 조회 필요) |

출처:
- mpv 속성·`deinterlace`·`hwdec`: https://mpv.io/manual/stable/ , `--sharpen` 항 https://github.com/mpv-player/mpv/blob/master/DOCS/man/options.rst (comparison.md 판정 4)
- `d3d11vpp`: https://github.com/mpv-player/mpv/blob/master/DOCS/man/vf.rst (comparison.md 판정 5)
- `unsharp` 양수=선명·`gblur` 명령: https://ffmpeg.org/ffmpeg-filters.html#unsharp , https://ffmpeg.org/ffmpeg-filters.html#gblur
- `hqdn3d`·`nlmeans`: https://ffmpeg.org/ffmpeg-filters.html#hqdn3d , https://ffmpeg.org/ffmpeg-filters.html#nlmeans
- `yadif`·`bwdif`: https://ffmpeg.org/ffmpeg-filters.html#bwdif
- 셰이더: https://mpv.io/manual/stable/#gpu-renderer-options , https://libplacebo.org/custom-shaders/
- D3D11 필터 enum: https://learn.microsoft.com/en-us/windows/win32/api/d3d11/ne-d3d11-d3d11_video_processor_filter

### 1.1 D3D11 비디오 프로세서 주의

- brightness/contrast/hue/saturation/noise reduction/edge enhancement enum 이 있으나, 지원 caps 와 `GetVideoProcessorFilterRange` 결과를 조회해야 한다. enum 이 있다고 모든 GPU 에서 R2 를 충족하지 않는다. https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videoprocessorenumerator-getvideoprocessorfilterrange
- denoise·edge enhancement 품질은 드라이버에 따라 다르다. 셰이더/CPU fallback 을 둔다. 일반 Gaussian blur 를 대신하는 표준 enum 은 없다.
- 이 표는 mpv 를 쓰는 이 가이드의 기본 경로에서 직접 호출하지 않는다. FFmpeg + 자체 D3D11 파이프라인을 만들 때의 선택지다.

### 1.2 선명의 대체 (gpu-next)

`vo=gpu-next` 를 쓰면 `sharpen` 속성이 안 되므로 둘 중 하나를 쓴다.
- lavfi `unsharp` 를 `vf` 에 라벨을 붙여 넣는다 (CPU 필터 비용은 3절).
- 선명 셰이더를 `glsl-shaders` 로 건다. 파일 형식은 WGSL·HLSL 과 호환되지 않는다 (mpv hook shader 형식).

mpv 셰이더 파일 형식과 WGSL/HLSL 의 비호환: https://libplacebo.org/custom-shaders/ , https://www.w3.org/TR/WGSL/ (sol).

## 2. 실시간 조절

| 방법 | 내용 | 출처 |
| --- | --- | --- |
| 속성 | `brightness` 등은 `set_property` 로 즉시 반영 | https://mpv.io/manual/stable/#command-interface |
| 필터 | 라벨을 붙인 `vf add / remove / toggle / set` | https://mpv.io/manual/stable/#video-filters |
| 필터 파라미터 | `vf-command` 는 lavfi 필터만. 필터와 옵션이 런타임 변경을 지원해야 한다 (`gblur`·`hqdn3d` 는 지원, 모든 필터가 그런 것은 아니다) | https://github.com/mpv-player/mpv/blob/master/DOCS/man/input.rst , https://github.com/FFmpeg/FFmpeg/blob/master/libavfilter/vf_gblur.c |
| 직접 FFmpeg | `avfilter_graph_send_command` 는 옵션에 "T" 표시가 있는 항목만. 아니면 그래프 재구성 | https://ffmpeg.org/ffmpeg-filters.html#Changing-options-at-runtime-with-a-command |

UI 규칙 (sol, 설계 제안):
- ON/OFF 와 강도를 따로 저장한다. OFF 에서도 마지막 강도를 보존한다.
- 슬라이더 입력은 합친다. 마우스 이벤트마다 파이프라인을 재생성하지 않는다.
- 적용이 실패하면 이전 유효 그래프로 돌아간다.
- 일시정지 중에 바꿔도 현재 프레임을 다시 출력한다.
- 시간축 필터(노이즈 제거)는 seek·트랙 교체·필터 구성 변경 시 이력을 초기화한다.

mpv 의 `vf` 변경이 파이프라인을 다시 만들 때 순간적으로 프레임이 끊기는지는 **미확인**이다. 실기 검증 항목 ([07-COMPARISON.md](07-COMPARISON.md) 4절).

## 3. CPU 필터 vs GPU 처리

| 사실 | 출처 |
| --- | --- |
| 하드웨어 디코딩 프레임에 CPU 필터를 걸려면 VRAM → RAM → VRAM 왕복이 생긴다. R1 의 무복사 이득이 줄고 4K60 에서 프레임이 드롭될 수 있다 | https://ffmpeg.org/doxygen/8.1/hw_decode_8c-example.html (sonnet·sol·argon 일치) |
| 단순 공간·색 필터는 GPU 에 두면 readback 을 줄인다. `nlmeans` 의 탐색·temporal denoise 는 비용이 크다 | https://ffmpeg.org/ffmpeg-filters.html#nlmeans |
| 4K60 품질 프리셋별 성능은 **미확인** | sol |

권장 배치 (설계 제안):

| 기능 | 먼저 쓸 것 | 이유 |
| --- | --- | --- |
| 밝기·대비·채도 | mpv 속성 | 출력 단계에서 처리, 지연 없음, GPL 아님 |
| 선명·흐림 | 셰이더 (또는 `unsharp`·`gblur`) | 셰이더가 무복사를 유지. lavfi 는 CPU 필터 |
| 디인터레이스 | mpv `deinterlace`, 하드웨어 디코딩이면 `d3d11vpp` | 같은 GPU 에서 처리 |
| 노이즈 제거 | 셰이더 또는 `hqdn3d`/`nlmeans` | `d3d11vpp` 에 없다. CPU 필터면 R1 이득이 줄어든다는 점을 UI 에 알린다 |

**미확인:** lavfi 필터를 `vf` 에 넣을 때 mpv 가 하드웨어 프레임을 자동으로 내려받는지, `hwdec` 를 copy 방식으로 바꿔야 하는지는 보고서에서 확인하지 못했다. 구현 전 `hwdec-current` 와 CPU/GPU 사용률로 확인한다.

## 4. GPL 필터 표

GPL 은 이 가이드에서 **허용**이다 ([05-WINDOWS.md](05-WINDOWS.md) 5절). 이 표는 GPL 허용 시 그대로 쓰는 값과, LGPL 로 바꿀 때의 대체를 함께 보여 준다.

근거: FFmpeg `configure` 의 `*_filter_deps` 에 `gpl` 이 있으면 `--enable-gpl` 이 필요하다. https://github.com/FFmpeg/FFmpeg/blob/master/configure . 주의: configure 에 의존 표기가 없다는 것이 소스 라이선스가 LGPL 이라는 보장은 아니다 (sonnet). 최종 구성 전체로 판단한다.

| 필터 | 용도 | configure 상 GPL | LGPL 쪽 대체 |
| --- | --- | --- | --- |
| `eq` | 밝기·대비·채도 | **GPL** | mpv 속성 `brightness`/`contrast`/`saturation` (필터 아님, 대체가 아니라 우선 수단). `colorlevels`/`colorbalance`/`lut` 는 sonnet 이 미확인 |
| `hqdn3d` | 노이즈 제거 | **GPL** | `nlmeans` (느림) 또는 셰이더 |
| `boxblur` | 흐림 | **GPL** | `gblur` |
| `histeq`, `nnedi`, `owdenoise`, `vaguedenoiser`, `kerndeint`, `pullup`, `colormatrix` | — | **GPL** (sonnet 이 configure 에서 확인) | 이 가이드는 쓰지 않는다 |
| `pp` | 후처리 | **GPL** (argon 보고, sonnet 목록에는 없음) | **미확인** |
| `unsharp` | 선명 | 없음 (sol: 소스 LGPL 표기 확인) | — |
| `gblur` | 흐림 | 없음 (sol: 소스가 다른 허용적 고지를 쓴다. 한 라이선스로 뭉뚱그리지 않는다) | — |
| `yadif`, `bwdif` | 디인터레이스 | 없음 (sol: LGPL 표기 확인) | — |
| `nlmeans` | 노이즈 제거 | 없음 (sol: LGPL 표기 확인) | — |
| `atadenoise`, `cas` | 노이즈 제거, 선명 | argon 이 LGPL 이라고만 적음. **미확인** | — |

출처 파일: https://github.com/FFmpeg/FFmpeg/blob/master/libavfilter/vf_unsharp.c , https://github.com/FFmpeg/FFmpeg/blob/master/libavfilter/vf_bwdif.c , https://github.com/FFmpeg/FFmpeg/blob/master/libavfilter/vf_nlmeans.c , https://github.com/FFmpeg/FFmpeg/blob/master/libavfilter/vf_gblur.c

## 5. PotPlayer 대응표

**기능 이름 기준**이다 (사용자 결정). PotPlayer 의 메뉴 경로·단축키·기본값·알고리즘은 확인하지 못해 넣지 않았다. 아래 왼쪽은 사용자 요구의 기능명에 대응시킨 것이고, PotPlayer 내부 구현과 같다는 뜻이 아니다. https://potplayer.daum.net/

| 기능 | 구현 (mpv 기준) | 차이·검증 포인트 | 출처 |
| --- | --- | --- | --- |
| 하드웨어 디코더 선택 (DXVA·D3D11 등) | `hwdec=auto-safe` / `d3d11va` / `d3d11va-copy` | 요청과 실제를 분리 표시. decode 가속과 renderer GPU 는 별개 | https://mpv.io/manual/stable/#video |
| 하드웨어 사용 안 함 | `hwdec=no` | — | 같음 |
| 비디오 렌더러 | `vo=gpu-next` + D3D11 컨텍스트 | PotPlayer 렌더러 명칭·화질과 같음을 보장하지 않음 | https://mpv.io/manual/stable/#video-output-drivers |
| 선명하게 | 셰이더 또는 `unsharp` (`vo=gpu` 면 `sharpen`) | 강도 범위는 독립 보정 | 1절 |
| 흐리게 | `gblur` sigma 또는 셰이더 | PotPlayer 의 blur kernel 동일성 **미확인** | 1절 |
| 노이즈 제거 | `hqdn3d`(GPL)/`nlmeans`/셰이더 | spatial/temporal 구분, 잔상 | 1절, 4절 |
| 디인터레이스 | `deinterlace`, `bwdif`/`yadif`, `d3d11vpp` | field order, 25i→50p·30i→60p, progressive 우회 확인 | https://ffmpeg.org/ffmpeg-filters.html#bwdif |
| 밝기·대비·채도·색조 | `brightness`/`contrast`/`saturation`/`hue` 속성 | limited/full range, HDR 적용 위치에 따라 효과 상이 | https://mpv.io/manual/stable/#equalizer |
| 스케일러·Super Resolution | `d3d11vpp` 의 스케일·Super Resolution, mpv `scale` 옵션 | PotPlayer 는 D3D11 내장 렌더러에 "GPU Super Resolution" 을 추가했다 (230405) https://www.neowin.net/software/potplayer-230405/ . mpv `scale`·libplacebo 세부는 **미확인** | comparison.md 판정 5 |
| 재생 속도 | `speed` + 피치 보정 (`scaletempo2` 자동 삽입) | 음정 보정 정책 | https://mpv.io/manual/stable/#playback-control |
| 탐색·음량 | `seek`, `volume` | exact/keyframe 구분 | https://mpv.io/manual/stable/#command-interface |
| 자막·오디오 트랙 | `sid`/`aid`/`sub-add`/`track-list` | 외부 SRT/ASS, MKV 첨부 폰트, 자막 지연 | https://mpv.io/manual/stable/#property-list |
| 전체 화면 | 호스트 창 fullscreen + pane 복원 | mpv 자식 창의 독립 fullscreen 과 중복 적용하지 않음 | https://mpv.io/manual/stable/#window |

argon 보고서가 적은 PotPlayer 메뉴 경로·단축키(Ctrl+I, Ctrl+R 등)는 출처가 없어 **미확인**이고, 사용자 결정에 따라 싣지 않았다.

---

이전: [05-WINDOWS.md](05-WINDOWS.md) · 다음: [07-COMPARISON.md](07-COMPARISON.md)
