---
tags: [rust, video-player, research, archive]
created: 2026-11-01
updated: 2026-11-01
---

# research · 조사 원자료 보관

> 이 파일을 읽는 때: 본문의 값·동작이 어디서 왔는지 원자료를 확인할 때, 보고서끼리 어긋난 값의 판정을 볼 때.

입구는 [../README.md](../README.md) 다.

이 폴더는 문서 묶음 B 를 만들며 Herdr 멀티에이전트(rvpn-scouter 3명, rvpn-lead)가 남긴 **작업 원자료**다. 고치지 않고 그대로 보관한다.
- **기준은 본문(README, 00~10)이다.** 원자료와 본문이 다르면 본문이 맞다. 보고서끼리 어긋난 값은 `reviews/comparison.md` 의 판정을 따랐다. 예: sol·argon 보고서의 `libmpv2` 6.0.x 는 7.0.0 이 맞다. argon 의 "libmpv 로 D3D11 Zero-Copy" 는 안정판 기준으로 틀렸다.
- 모든 조사는 확인일 2026-11-01 기준이다. 조사만 했고 패키지 설치·빌드·실행은 하지 않았다.
- 상충 항목은 rvpn-lead 가 출처(crates.io API, mpv v0.41.0 rst·헤더·소스, FFmpeg 문서, GitHub API)를 직접 열어 판정했다. 확인 수준은 [reviews/comparison.md](reviews/comparison.md) 끝의 "직접 확인한 출처" 와 [reviews/writer-notes.md](reviews/writer-notes.md) 에 있다.
- 라이선스가 걸린 코드 원문은 없다. 사실(값·이름·동작)만 있다.

## 1. sources · 조사 원자료

| 파일 | 담당 | 내용 |
| --- | --- | --- |
| [sonnet/report.md](sources/sonnet/report.md) | rvpn-scouter-sonnet | B1~B10. libmpv2 7.0.0, mpv D3D11 render API PR #17764, D3D11→wgpu 공유 핸들 사례. 상당 부분이 검색 요약과 크레이트 메타데이터에 기댄다 |
| [sonnet/followup-1.md](sources/sonnet/followup-1.md) | rvpn-scouter-sonnet | 재질의: PotPlayer 기능 이름 목록과 출처 (공식 확인은 일부뿐) |
| [sol/report.md](sources/sol/report.md) | rvpn-scouter-sol | B1~B10. eframe+glow+libmpv 0.41 권고, egui 위젯 버전 호환, mpv 스레드 규칙. 출처 URL 이 가장 촘촘하다 |
| [sol/followup-1.md](sources/sol/followup-1.md) | rvpn-scouter-sol | 재질의: WGL 에서 쓸 수 있는 hwdec(copy-back), `egui_glow::CallbackFn`, eframe 의 ANGLE 선택, mpv CI 빌드 GPL/LGPL 변형 |
| [argon/report.md](sources/argon/report.md) | rvpn-scouter-argon | B1~B10. wgpu+libmpv 권고, R2 셰이더 배치, 기능별 난이도. 일부 값(libmpv2 버전, Zero-Copy, iced 0.14.1, HDR "완벽 지원")이 판정과 다르다 |
| [argon/followup-1.md](sources/argon/followup-1.md) | rvpn-scouter-argon | 재질의: HW 디코더 세션 수 주장 정정, D3D11 VideoProcessor 필터의 드라이버 의존, 크레이트 버전 재확인, DLL 크기(출처 없음) |

## 2. reviews · 검토·판정·작성 기록

| 파일 | 담당 | 내용 |
| --- | --- | --- |
| [comparison.md](reviews/comparison.md) | rvpn-lead | B1~B10 항목별 비교(셋 다/둘만/상충/한쪽만), 상충 9건의 판정, 사용자 결정 4건 |
| [outline.md](reviews/outline.md) | rvpn-lead | 목차 초안 (파일 목록, 절 제목, 근거 항목 ID) |
| [writer-notes.md](reviews/writer-notes.md) | rvpn-lead | 본문을 쓰며 만난 모순(W1~W3)과 사용자 확인 결과, 보수적으로 처리한 항목 |
| [summary-for-leader.md](reviews/summary-for-leader.md) | rvpn-lead | rvp-leader 에게 보내는 요약 |
| [agy-approvals.log](reviews/agy-approvals.log) | rvp-leader | agy 승인 기록 |

## 3. 어떤 값이 어디서 왔나

| 본문 내용 | 주 근거 |
| --- | --- |
| 단계안(glow → wgpu), 안정판/개발판 경계 | sol B1·B4, sonnet B4, comparison.md B1-2·B4-2 판정 |
| WGL 의 copy-back, ANGLE, `CallbackFn` | sol followup-1, rvpn-lead 가 `render_gl.h`·`vd_lavc.c` 확인 |
| egui 위젯 버전 조합 | sol B1·B5, comparison.md B5-2 판정 (crates.io 의존성 API) |
| 코어·스레드·종료 순서 | sol B6, mpv `render.h` |
| 탐색기·탭·숨김 탭 | sol B5, argon B5, argon followup-1 (세션 수 정정) |
| R1 | sol B7, sonnet B7, argon B7, mpv `options.rst`·`input.rst` |
| R2·GPL 필터·실행 중 명령 | sol B7, argon B3·B7, rvpn-lead 가 FFmpeg Filters·LICENSE.md 확인 |
| PotPlayer 대응 | sonnet followup-1, mpv `options.rst`·`input.rst` |
| 배포·라이선스·크기 예시 | sol B8·followup-1, mpv Copyright·meson.options, GitHub API |
| 미확인 목록 | 세 보고서 B10, writer-notes |
