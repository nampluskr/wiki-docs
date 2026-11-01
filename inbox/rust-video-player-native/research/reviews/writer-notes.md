---
tags: [rust, video-player, writer-notes]
created: 2026-11-01
updated: 2026-11-01
---

# 본문 작성 노트 — 판단이 필요한 모순과 보수적 처리

작성: rvpn-lead, 2026-11-01. 재질의 답: `research/sources/{sol,sonnet,argon}/followup-1.md` (항목당 1회 사용).

## 1. 사용자 확인을 권하는 모순

| # | 모순 | 근거 | 본문에서 보수적으로 처리한 방식 | 위치 |
| --- | --- | --- | --- | --- |
| W1 | comparison.md B1-2 는 "glow 경로만 확인된 경로"라고 했다. 그런데 그것은 **표시 경로**에 대해서만 맞다. eframe 0.37 의 Windows 기본은 WGL 이고, mpv OpenGL render API 에서 직접 HW interop 을 쓰려면 "Windows: ANGLE is required" 이다. 그래서 1단계의 HW 디코딩은 대부분 copy-back(`d3d11va-copy`)이 된다 | [render_gl.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h) (직접 확인), [vd_lavc.c](https://github.com/mpv-player/mpv/blob/v0.41.0/video/decode/vd_lavc.c) autoprobe 표 (직접 확인), sol followup-1 (eframe 이 `FallbackEgl` 을 하드코딩) | 단계안은 유지했다. 단, "R1 은 충족되지만 1단계 HW 는 copy-back 이며 비용은 실측 필요"라고 01·03·06 에 명시했다. 개선 경로로 (a) winit+glutin 에서 ANGLE 강제, (b) 2단계를 적었다 | 01 §3.1, 03 §2.3, 06 §2, 10 Q4·Q5 |
| W2 | 1단계는 "libmpv 0.41 **안정판**"이 전제인데, Windows 용으로 바로 받을 수 있는 libmpv DLL 은 공식 CI 개발 빌드(`v0.41.0-dev-…`, master)나 서드파티 git 빌드다. 공식 v0.41.0 릴리스에 Windows libmpv 바이너리가 있다는 근거는 없다 | [git-release](https://github.com/mpv-player/mpv/releases/tag/git-release) asset 이름 (GitHub API 로 직접 확인), sol followup-1 §3 | 빌드(날짜·해시)를 고정하고 `mpv_client_api_version()` 을 기록한다. 앱은 client API 2.5(0.41) 범위만 쓴다. 안정판이 꼭 필요하면 v0.41.0 태그를 직접 빌드한다 | 09 §3, 10 Q12 |
| W3 | R2 의 기능 목록(선명·흐림·노이즈·색 조정)을 PotPlayer 기능이라고 했는데, PotPlayer 공식 페이지에서는 이 기능들을 확인하지 못했다(2차 출처만 있음) | sonnet followup-1 | R2 의 출처를 "PotPlayer 문서"가 아니라 "이 프로젝트의 요구사항"으로 적었다. 대응표의 "PotPlayer 출처" 칸에는 "2차 출처만 있음"이라고 표시했다 | 00 §5, 08 |

### 1.1 사용자 확인 결과 (2026-11-01, rvp-leader 경유)

| # | 결과 |
| --- | --- |
| W1 | **확정: 단계안을 유지한다. 1단계 copy-back 비용은 실측한 뒤 다시 검토한다.** |
| W2 | 사용자 확인: 현행 유지 (빌드 고정 + client API 2.5 범위만 사용, 필요하면 v0.41.0 태그를 직접 빌드) |
| W3 | 사용자 확인: 현행 유지 (R2 기능 목록의 출처는 이 프로젝트의 요구사항) |

## 2. 보수적으로 쓴 세부 항목 (확인 불요, 기록용)

| # | 내용 | 처리 |
| --- | --- | --- |
| W4 | argon 은 "HW 다중 세션 8개 초과 시 SW 로 전환"이라고 썼다 | argon 이 재질의에서 스스로 정정했다(고정 상한은 근거 없음, VRAM 부족·미지원 프로파일 때문). 정정한 내용으로 썼다 (05 §5.1, 06 §3.3) |
| W5 | argon 은 libmpv-2.dll 크기를 35~55MB → 110~115MB 로 바꿔 적었다 | 출처가 없어서 뺐다. GitHub API 로 확인한 압축 파일 크기 3건만 예시로 남겼다 (09 §4) |
| W6 | argon followup 은 R2 의 기본 경로로 "wgpu WGSL 자체 셰이더"를 권했다 | 1단계(glow)와 맞지 않아서 채택하지 않았다. lavfi + mpv 속성을 1차로, GLSL 셰이더를 선택지로 두었다 (07) |
| W7 | B4-4 의 libmpv 바이너리 GPL/LGPL 변형 | sol followup 으로 해소했다. 변형이 있는 것은 공식 **CI 개발 빌드**다. 안정판에 두 변형이 있다는 근거는 없다 (09 §3) |
| W8 | `d3d11vpp` 는 "Requires a D3D11 context" 이다 | 1단계에서는 쓸 수 없다고 명시했다. 디인터레이스는 `deinterlace`(bwdif)로 한다 (03, 07) |
| W9 | `vf-command` 의 label 인자에 `@` 를 붙이는지 문서가 분명하지 않다 | 예시는 `@` 없이 적고, 미확인 Q9 로 남겼다 (07 §3, 10) |
| W10 | egui paint callback 과 이미지 도형의 실행 순서 | 한 프레임 지연을 허용하는 설계로 적고 Q6 으로 남겼다 (03 §2.2) |
| W11 | libmpv2 버전 (sol·argon 은 6.0.x, 실제로는 7.0.0) | 7.0.0 으로 썼다. 6→7 의 API 변경은 Q13 으로 남겼다 |
| W12 | mpv.io 매뉴얼에는 옵션 단위 앵커가 없다 (rvp-leader 안내) | mpv.io 링크는 절 앵커(#video, #property-list 등)로 걸고, 옵션 단위 근거는 v0.41.0 rst 원문 링크로 걸었다 |
| W13 | LGPL 빌드 옵션 | `-Dgpl=false` (meson.options v0.41.0 의 `option('gpl', type: 'boolean', value: true …)` 를 직접 확인) |

## 3. outline.md 와 달라진 점

- 01 에 "1단계의 대가: copy-back" 절(§3.1)을 추가했다 (W1).
- 09 에 "DLL 구하기"와 "크기 예시" 절을 추가했다 (W2, W5, 사용자 결정 2).
- 08 은 사용자 결정 4 에 따라 PotPlayer 기능 이름 → mpv 구현 대응표로 썼다(메뉴·단축키 없음).
