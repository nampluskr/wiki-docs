---
tags: [rust, video-player, summary]
created: 2026-11-01
updated: 2026-11-01
---

# 묶음 B 요약 — rvp-leader 에게 (rvpn-lead, 2026-11-01)

**결론**: eframe+glow+libmpv 0.41(OpenGL render API)로 지금 시작한다. mpv 0.42 의 D3D11 render API 가 출시되면 wgpu(DX12)로 렌더만 옮긴다. R1·R2 는 mpv 속성·lavfi 필터로 충족한다.

## 후보별 장단점

| 후보 | 장점 | 단점 |
| --- | --- | --- |
| 1단계 eframe+glow + libmpv 0.41 | 출시된 API 만 쓴다. 영상 위 OSD 를 그릴 수 있다. Rust 한 언어 | WGL 이라 HW 디코딩이 대부분 copy-back. `d3d11vpp` 불가. UI 스레드 = 렌더 스레드 |
| 2단계 wgpu + libmpv 0.42 D3D11 | `d3d11va` 직접 디코딩 기대. 렌더 크레이트만 교체 | 0.42 미출시, Rust 래퍼 없음, D3D11→D3D12→wgpu 브리지 |
| 대안 Win32 + libmpv `wid` | 직접 디코딩·`d3d11vpp`. 지금 된다 | airspace(영상 위 UI 불가), UI 를 Win32 로 직접 만든다 |
| 대안 wgpu + FFmpeg 직접 | 디코더·셰이더 완전 통제 | A/V 동기·자막·탐색·R1 을 모두 직접 구현. 구현량이 가장 크다 |

## 최종 선택과 단계 (01-stack-decision.md 7.2)

| 단계 | 끝내는 조건 |
| --- | --- |
| 0. PoC: glow 탭에 영상 + OSD, R1 토글 | stuck 로그 없이 표시·토글. 막히면 Win32 대안 검토 |
| 1. 1단계 구현 (코어·탭·트리·R1·R2·재생 기능) | 00 의 R1·R2 인수 기준 + PoC P3·P4 통과 |
| 1.5 copy-back 실측 후 재검토 (W1 확정) | 허용 범위면 유지. 아니면 ANGLE 강제 셸 또는 2단계를 앞당긴다 |
| 2. 0.42 출시 후 wgpu 전환 | 0.42 안정판 + 래퍼/FFI + 브리지 PoC 성공 |

크레이트(2026-11-01): eframe 0.37, egui_tiles 0.18, egui_ltreeview 0.10 (모두 egui 0.37), libmpv2 7.0.0. egui_dock 0.21.1 은 egui 0.36 이라 쓰지 않는다.

## 가장 큰 위험 3가지

1. **copy-back 비용**: eframe WGL 에서는 mpv 의 직접 HW interop 에 ANGLE 이 필요하다(render_gl.h). 4K 고프레임의 CPU·전력 비용은 미측정이다.
2. **스레드 규칙**: glow 콜백이 UI 스레드에서 돌기 때문에 mpv render API 규칙(렌더 스레드에서 일반 API 호출·대기 금지)을 어기면 stuck·교착이 생긴다.
3. **libmpv 바이너리**: Windows 용으로 바로 받을 수 있는 것은 CI·서드파티 **개발 빌드**(mingw)다. 안정판 근거가 없고, MSVC import library 도 미확인이다. 그래서 빌드를 고정하고 client API 2.5 만 쓴다(W2 현행 유지).

## 남은 실측·미확인 상위 5개 (10-validation-and-open-questions.md)

1. Q4 — WGL 에서 `hwdec=auto` 가 실제로 고르는 값, copy-back 의 4K CPU·전력 비용
2. Q7 — `hwdec` 를 바꿀 때 위치·필터가 유지되는지, 정지 시간
3. Q8·Q9 — R2 필터 toggle·교체 때의 정지 시간, disabled 필터의 비용, `vf-command` 의 label 표기
4. Q11·Q12 — mingw libmpv DLL 의 MSVC 링크, 공식 안정판 Windows 바이너리 유무
5. Q1·Q3 — mpv 0.42.0 출시 시기, D3D11 render API 의 "제한" 내용

사용자 결정 반영: GPL 허용, 기간·크기 수치 없음(실측 필요), PotPlayer 대응은 기능 이름 기준, W1~W3 확정.
