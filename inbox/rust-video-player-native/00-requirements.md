---
tags: [rust, video-player, requirements, windows]
created: 2026-11-01
updated: 2026-11-01
---

# 00. 요구사항과 범위

> 이 파일을 읽는 때: 무엇을 만들지, 무엇을 "다 됐다"로 볼지 정할 때. 다른 파일의 기준이 여기 있다.

## 1. 만들 것

| 항목 | 내용 |
| --- | --- |
| 앱 | Windows 11 데스크톱 동영상 플레이어 |
| UI 형태 | 왼쪽에 파일 탐색기 트리, 오른쪽 탭 영역. 동영상은 탭 안에서 재생한다 |
| 기능 기준 | PotPlayer (기능 이름 기준. 메뉴 경로·단축키는 다루지 않는다) |
| 필수 R1 | GPU 가속(하드웨어 디코딩) ON/OFF. 재생 중에 전환하고, 실패하면 소프트웨어 디코딩으로 자동 전환 |
| 필수 R2 | 선명하게·흐리게·노이즈 제거·디인터레이스·밝기/대비/채도. 재생 중 실시간 조절 |
| 필수 R3 | 유튜브 링크를 클릭하면 새 탭의 같은 플레이어 화면에서 재생한다. R1·R2 가 그대로 적용된다. **2026-11-01 추가, 공통 문서 작성 중**: 상세는 묶음 A·B 공통 문서 [../rust-video-player-youtube.md](../rust-video-player-youtube.md) |
| 언어 | 앱 코드는 Rust. 디코딩 C 라이브러리(FFmpeg·libmpv·GStreamer)는 Rust 바인딩으로 허용 |
| 기준 | Windows 11, Rust stable-msvc, 2026-11-01 확인 문서·crates.io |
| 라이선스 | GPL 허용 (사용자 결정). 자세한 내용은 [09-distribution-licensing.md](09-distribution-licensing.md) |

이 묶음은 기존 앱 구조를 전제하지 않는 **원점 설계**다. 검증 범위: 실제 빌드·실행은 하지 않았다. 모든 내용은 공식 문서·소스·crates.io 를 기준으로 하며, 추정은 "추정"으로 표시한다.

## 2. "Rust 만" 조건의 해석

| 대상 | 이 묶음의 해석 | 근거 |
| --- | --- | --- |
| 앱 로직·UI 코드 | Rust 만 쓴다 | 요구사항 |
| 디코딩 엔진 | C 라이브러리를 Rust 바인딩으로 쓴다 (libmpv2 등) | 요구사항 |
| GPU 셰이더 (GLSL·WGSL·HLSL) | GPU 자산으로 보고 허용한다. 다만 처음에는 엔진 내장 필터를 먼저 쓴다 | sol 보고서 B3 |
| UI DSL (`.slint`, makepad `live_design!`) | Rust 문법이 아니므로 이 조건에 맞지 않을 수 있다. 그래서 후보에서 뒤로 뺀다 | sol 보고서 B2, [Slint API](https://docs.rs/slint/latest/slint/) |

## 3. R1 인수 기준

| 번호 | 기준 | 확인 방법 |
| --- | --- | --- |
| R1-a | 재생 중에 ON/OFF 할 수 있다. 앱을 다시 시작하거나 파일을 다시 열 필요가 없다 | mpv 는 `hwdec` 를 실행 중 바꿀 수 있다. 공식 문서에 "toggle hardware decoding at runtime"(`auto` ↔ `no`)이 있다 ([options.rst v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst), [매뉴얼 Video 절](https://mpv.io/manual/stable/#video)) |
| R1-b | 하드웨어 디코딩이 불가능하거나 실패하면 소프트웨어로 전환된다 | mpv: "If hardware decoding is not possible, mpv will fall back on software decoding." `--hwdec-software-fallback` 의 기본값은 3이다(연속 3프레임 실패 시 전환) (같은 문서) |
| R1-c | UI 에 **요청한 값**과 **실제로 쓰는 값**을 따로 보인다 | `hwdec-current` 속성: "no indicates software decoding" ([input.rst v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst)) |
| R1-d | 전환 중에도 UI 가 멈추지 않는다. 위치·속도·트랙·필터가 유지된다 | 앱 설계로 보장한다 ([06-r1-hwdec.md](06-r1-hwdec.md)) |

"재생 중 전환"이 **무중단·프레임 유실 0** 을 뜻하지는 않는다. 그런 보장은 공식 문서에 없다. 오히려 폴백 때 파일 일부를 건너뛸 수 있다는 경고가 있다 ([options.rst `--hwdec-software-fallback`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)).

## 4. R2 인수 기준

| 기능 | 기준 | 1차 구현 수단 (자세한 내용은 07) |
| --- | --- | --- |
| 밝기·대비·채도 | 슬라이더를 움직이면 다음 프레임부터 반영된다 | mpv `brightness`/`contrast`/`saturation` (-100~100) ([options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)) |
| 디인터레이스 | ON/OFF/자동 | mpv `deinterlace=<yes\|no\|auto>` (같은 문서) |
| 선명하게 | ON/OFF + 강도 | FFmpeg `cas` 또는 `unsharp` (lavfi) |
| 흐리게 | ON/OFF + 강도 | FFmpeg `gblur` (lavfi) |
| 노이즈 제거 | ON/OFF + 강도 | FFmpeg `hqdn3d` (lavfi, GPL) |

"실시간 조절"은 이렇게 정의한다. **(a)** 실행 중 명령으로 값을 바꿀 수 있는 필터는 명령으로 바꾼다. **(b)** 명령을 지원하지 않는 필터는 같은 레이블로 필터를 교체한다. 두 경우 모두 재생이 이어져야 한다. 필터마다 지원 여부가 다르며, 그 목록은 [07-r2-video-options.md](07-r2-video-options.md) 에 있다 ([FFmpeg Filters](https://ffmpeg.org/ffmpeg-filters.html), [input.rst `vf-command`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst)).

## 5. PotPlayer 기능 대응 범위

PotPlayer 공식 페이지로 확인한 기능은 적다(하드웨어 가속 DXVA/CUDA/QuickSync, SMI/SRT/ASS/SSA/VobSub 자막, 북마크, 사운드 카드 선택, 장면 미리보기 등) ([potplayer.tv](https://potplayer.tv/)). 영상 필터·색 조정은 공식 페이지에서 확인되지 않았다. 그래서 R2 의 기능 목록은 **PotPlayer 문서가 아니라 이 프로젝트의 요구사항**에서 온 것으로 본다. 대응표는 [08-playback-features.md](08-playback-features.md) 에 있다.

| 등급 | 기능 |
| --- | --- |
| 필수 | R1, R2, 파일 탐색기 트리, 탭 재생, 탐색, 속도, 음량, 트랙 선택, SRT/ASS 자막, 전체 화면 |
| 권장 | 구간 반복(A-B), 자막·오디오 싱크, 북마크, 스크린샷, 화면 비율·자르기·회전, 항상 위 |
| 범위 밖 (이번 묶음) | 3D 출력, DVD/TV, 스트리밍·방송, 녹화, 사용자 HLSL 셰이더 팩 |

## 6. HDR

HDR 출력은 필수가 아니다. 후보 비교에서는 평가 항목으로만 쓴다. 권장 경로(eframe+glow)에서 HDR 을 처음부터 끝까지 지원하는지는 **미확인**이다 ([Microsoft HDR 지침](https://learn.microsoft.com/en-us/windows/win32/direct3darticles/high-dynamic-range)). 미확인 목록은 [10-validation-and-open-questions.md](10-validation-and-open-questions.md) 에 있다.

---

다음: [01-stack-decision.md](01-stack-decision.md)
