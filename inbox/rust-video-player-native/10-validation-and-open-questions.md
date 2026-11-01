---
tags: [rust, video-player, validation, open-questions]
created: 2026-11-01
updated: 2026-11-01
---

# 10. 검증 계획과 미확인 목록

> 이 파일을 읽는 때: PoC 를 시작하기 전, 그리고 버전을 올리거나 2단계로 갈지 다시 정할 때.

이 묶음의 내용은 문서·소스 조사 결과다. **빌드·실행·계측은 하지 않았다.** 아래 항목을 PoC 로 확인해야 한다.

## 1. PoC 순서

| 순서 | PoC | 성공 기준 | 관련 파일 |
| --- | --- | --- | --- |
| P1 | eframe 0.37 glow + libmpv2 7.0.0 로 탭 하나에 영상 표시 | 탭 FBO 에 영상이 나오고, 그 위에 OSD 가 그려진다. `mpv_render_context_render() not being called or stuck` 로그가 없다 | [03](03-rendering-pipeline.md), [04](04-player-core.md) |
| P2 | R1 토글 | `hwdec=auto` ↔ `no` 를 재생 중에 바꾸고, `hwdec-current` 를 기록한다. 위치·트랙·필터가 유지되는지 본다 | [06](06-r1-hwdec.md) |
| P3 | R2 체인 | 레이블 필터 toggle·`vf-command`·교체가 재생 중 동작하고, 프레임이 멈추는 시간을 잰다 | [07](07-r2-video-options.md) |
| P4 | 다중 탭 | 탭 2~4개, 숨김 탭 일시정지, 분할 화면 동시 재생, 오디오 포커스 | [05](05-explorer-tabs.md) |
| P5 | 배포 | MSVC 링크, DLL 을 exe 옆에 두고 다른 PC 에서 실행 | [09](09-distribution-licensing.md) |
| P6 | R3 유튜브 링크 재생 | Lua 포함 DLL 에서 `ytdl_hook` 이 yt-dlp 를 실행해 새 탭에 재생한다. 4K VP9/AV1 에서 copy-back 비용을 잰다. R1 토글·R2 필터가 로컬 파일과 같이 동작한다 (Q18, Q19) | [04](04-player-core.md), [05](05-explorer-tabs.md), [공통 문서](../rust-video-player-youtube.md) |

## 2. 측정 corpus 와 지표 (제안)

| 축 | 값 |
| --- | --- |
| 컨테이너 | MP4, MKV, WebM, TS |
| 코덱 | H.264, HEVC(8/10bit), VP9, AV1 / AAC, AC3, Opus |
| 해상도·프레임률 | 1080p·4K, 23.976/24/25/29.97/50/59.94/60, CFR·VFR, 인터레이스 원본 |
| 자막 | 외부 SRT·ASS·SMI, 한글 경로·파일명 |
| GPU | Intel 내장, AMD, NVIDIA (하이브리드 노트북 포함) |
| 모니터 | 60/120/144Hz, DPI 100%/150% |

지표: `hwdec-current`, CPU·GPU 사용률, 프레임 drop, A/V drift, 메모리·VRAM, 전력, R1 전환 시간, R2 적용 지연. (sol 보고서 B7 의 검증 제안을 정리했다. 수행한 테스트는 아니다.)

## 3. 미확인 목록

| ID | 질문 | 영향 | 다시 확인할 때 | 출처 |
| --- | --- | --- | --- | --- |
| Q1 | mpv 0.42.0 출시 시기, 안정판에 D3D11 render API 가 들어가는지 | 2단계 착수 | 분기마다 | [Releases](https://github.com/mpv-player/mpv/releases), [client-api-changes](https://github.com/mpv-player/mpv/blob/master/DOCS/client-api-changes.rst) |
| Q2 | libmpv2 가 D3D11 render 를 래핑하는지 | 2단계 FFI 범위 | 0.42 출시 후 | [libmpv2](https://crates.io/crates/libmpv2) |
| Q3 | PR #17764 리뷰어가 말한 D3D11 render API 의 "제한"이 무엇인지 | 2단계 vs Win32 `wid` | 0.42 출시 전 | [PR #17764](https://github.com/mpv-player/mpv/pull/17764) |
| Q4 | eframe WGL 에서 `auto` 가 실제로 무엇을 고르는지, copy-back 의 4K CPU·전력 비용 | 1단계 성능 | P2 | [vd_lavc.c](https://github.com/mpv-player/mpv/blob/v0.41.0/video/decode/vd_lavc.c), sol followup-1 |
| Q5 | eframe 대신 winit+glutin 으로 ANGLE(EGL)을 강제하면 `d3d11va` 직접 interop 이 되는지 | 1단계 개선 경로 | Q4 결과가 나쁠 때 | [render_gl.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h), [glutin](https://docs.rs/glutin/0.32.3/glutin/display/enum.DisplayApiPreference.html) |
| Q6 | egui paint callback 과 이미지 도형의 실행 순서(같은 프레임 반영 여부) | 한 프레임 지연 | P1 | [painter.rs](https://github.com/emilk/egui/blob/0.37.0/crates/egui_glow/src/painter.rs) |
| Q7 | `hwdec` 변경 때 위치·필터 유지, 정지 시간 | R1 UX | P2 | [options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| Q8 | disabled(`!`) 필터의 CPU 비용, 필터 교체 때 정지 시간 | R2 UX | P3 | [input.rst `vf`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst) |
| Q9 | `vf-command` 의 label 인자에 `@` 를 붙이는지 | R2 구현 | P3 | input.rst |
| Q10 | 4K 다중 탭에서 R2(특히 `hqdn3d`) CPU 부하 | 다중 탭 정책 | P4 | argon 보고서 B10 |
| Q11 | mingw 빌드 libmpv DLL 을 MSVC 로 링크할 import library | 배포 | P5 | [git-release](https://github.com/mpv-player/mpv/releases/tag/git-release) |
| Q12 | 공식 v0.41.0 안정판 Windows libmpv 바이너리가 있는지 | 1단계 "안정판" 전제 | 지금 | sol followup-1 |
| Q13 | libmpv2 6.x → 7.0 의 API 변경 | 코드 작성 | 의존성 고정 전 | [libmpv2](https://crates.io/crates/libmpv2) |
| Q14 | HDR end-to-end (glow 경로) | HDR 이 필수가 되면 재평가 | 요구 변경 시 | [Microsoft HDR](https://learn.microsoft.com/en-us/windows/win32/direct3darticles/high-dynamic-range) |
| Q15 | 2단계 브리지: D3D11 공유 텍스처 → D3D12 → wgpu-hal 의 동기화·포맷 | 2단계 | 2단계 PoC | [wgpu-hal](https://docs.rs/wgpu-hal/latest/wgpu_hal/), [D3D12 interop](https://learn.microsoft.com/en-us/windows/win32/direct3d12/direct3d-12-with-direct3d-11--direct-2d-and-gdi) |
| Q16 | `.smi` 자동 로드 여부 | 한국어 자막 | P1 | [options.rst `--sub-auto-exts`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) |
| Q17 | 개발 기간·배포 크기·전력 | 계획 | 각 PoC 후 | 수치 없음 (사용자 결정: 실측 필요) |
| Q18 | (R3) 유튜브 4K VP9/AV1 재생 때 1단계 glow copy-back 비용: `hwdec-current`, CPU·GPU 사용률, 프레임 drop. GPU 가 코덱을 지원하지 않으면 SW 디코딩이 되는지도 기록한다 | R3 성능, 1.5단계 재검토 | P6 | Q4, [render_gl.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h), [공통 문서](../rust-video-player-youtube.md) |
| Q19 | (R3) 배포할 libmpv DLL 이 Lua 를 넣어 빌드됐는지, 그 DLL 에서 `ytdl=yes` + `ytdl_hook-ytdl_path` 로 내장 `ytdl_hook` 이 동작하는지 | R3 가능 여부 | P6 | [04-player-core.md](04-player-core.md) 6절, [공통 문서](../rust-video-player-youtube.md) |

## 4. 결정 기록

| 날짜 | 결정 | 근거 | 기각한 것 |
| --- | --- | --- | --- |
| 2026-11-01 | 1단계 eframe+glow+libmpv 0.41, 2단계 wgpu+0.42 D3D11 | 안정판 render API 는 OpenGL/SW 뿐이다 ([render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)) | 처음부터 wgpu+libmpv (미출시 API 에 의존) |
| 2026-11-01 | 기간·크기 수치를 넣지 않는다 | 세 보고서의 추정치가 서로 달랐고 근거가 없었다 | 범위 추정치 |
| 2026-11-01 | GPL 허용 | 사용자 결정 | LGPL 전용 빌드 |
| 2026-11-01 | 탭·분할은 egui_tiles | egui 0.37 호환 (egui_dock 0.21.1 은 ^0.36) | egui_dock (egui 0.36 으로 내려야 함) |
| 2026-11-01 | `wid` 자식 창을 쓰지 않는다 | airspace 문제, mpv 의 임베딩 경고 | Win32 + `wid` (대안으로만 남김) |
| 2026-11-01 | W1 확정: 단계안을 유지한다. 1단계 copy-back 비용은 실측(Q4, P2) 후 다시 검토한다 | 사용자 확인. WGL 에서 직접 interop 에 ANGLE 이 필요하다 ([render_gl.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h)) | 1단계부터 ANGLE 강제 셸 (Q5 로 보류) |
| 2026-11-01 | W2 사용자 확인: 현행 유지. libmpv 빌드를 고정하고 client API 2.5 범위만 쓴다 | Windows 용 안정판 libmpv 바이너리의 근거가 없다 (Q12) | — |
| 2026-11-01 | W3 사용자 확인: 현행 유지. R2 기능 목록은 이 프로젝트의 요구사항으로 본다 | PotPlayer 공식 페이지에서 R2 기능을 확인하지 못했다 | — |

비교 근거 전체는 [research/reviews/comparison.md](research/reviews/comparison.md) 에 있다.

---

이전: [09-distribution-licensing.md](09-distribution-licensing.md) · 다음: [11-comparison.md](11-comparison.md)
