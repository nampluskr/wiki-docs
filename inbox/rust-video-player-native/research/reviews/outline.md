---
tags: [rust, video-player, outline]
created: 2026-11-01
updated: 2026-11-01
---

# 문서 묶음 B 목차 초안

제목: **Rust 동영상 플레이어 원점 설계 가이드 — 파일 탐색기·탭 포함 (Windows 11)**
위치: `D:\projects\docs\guides\rust-video-player-native\`
근거 ID 는 `research/reviews/comparison.md` 의 항목 ID 다.

> 전제: `comparison.md` 의 "사용자 판단 필요" 1번(B1-2)의 답에 따라 01·03·04 의 주 경로가 바뀐다. 아래 초안은 **(다) 단계안**을 가정한다. glow+libmpv 0.41 로 시작하고, 0.42 D3D11 render API 를 이전 경로로 둔다.

| 파일 | 절 제목 | 근거 ID |
|---|---|---|
| `README.md` | 이 묶음의 목적과 전제 (원점 재검토, Rust 앱 코드 + C 디코더 바인딩, 2026-11 기준) | B9-1 |
| | 읽기 지도 (상황별로 읽을 파일 표) | B9-1 |
| | 한눈에 보는 결론 (권장 스택·대안·주요 위험 5줄) | B1-1, B1-2, B1-4 |
| | 용어 (hwdec, render API, copy-back, zero-copy, airspace) | B3-2, B4-6, B7-5 |
| `00-requirements.md` | PotPlayer 기능 기준과 범위 | B7-11, B7-12 |
| | R1 인수 기준 (재생 중 전환, 실패 시 SW, 실제 상태 표시) | B7-1, B7-2, B7-4 |
| | R2 인수 기준 (6종, 실시간 조절의 정의) | B7-6~B7-10 |
| | "Rust 만" 조건의 해석 (셰이더·DSL·C 바인딩의 경계) | B2-6, B3-8 |
| | HDR 은 필수가 아님 (평가 항목으로만) | B3-5, B10-3 |
| `01-stack-decision.md` | UI 후보 비교표 (버전·확인일) | B2-1~B2-11 |
| | 권장 스택과 이유 | B1-1, B1-2 |
| | 대안 스택: Win32+libmpv, FFmpeg 직접 | B1-3, B8-3, B8-4 |
| | 스택 비교표 (기간·성능·R1·R2·크기·위험) | B8-1~B8-5 |
| | 안정판과 개발판의 경계 (libmpv 0.41 vs 0.42) | B4-2, B10-1 |
| | 버전 고정 표 (egui 0.37 + egui_tiles 0.18 + egui_ltreeview 0.10, libmpv2 7.0.0) | B4-1, B5-2 |
| `02-decoding-engine.md` | libmpv / FFmpeg / GStreamer 비교 | B4-3, B4-8 |
| | libmpv render API 의 성격 (앱 타깃에 그린다, core 당 context 하나) | B4-7 |
| | 통합 방식 3가지 (render API / wid / CPU 업로드) | B4-6 |
| | libmpv2 7.0.0 의 커버 범위와 직접 FFI 가 필요한 곳 | B4-1, B4-2 |
| `03-rendering-pipeline.md` | glow 경로: egui paint callback 과 mpv FBO | B1-2, B6-2 |
| | wgpu 경로: D3D11 공유 핸들 → D3D12 → wgpu-hal | B3-1, B3-3, B10-4, B10-8 |
| | 복사 경로 구분 (CPU 왕복 / GPU 복사 / zero-copy) | B3-2 |
| | YUV→RGB 와 색 메타데이터 (NV12/P010, 601/709/2020, range) | B3-4, B3-6 |
| | R2 패스 순서와 UI 합성 위치 | B3-7, B3-8 |
| `04-player-core.md` | 크레이트 구성 | B6-1 |
| | 스레드 모델과 mpv 콜백 규칙 | B6-2, B6-4 |
| | Command / Event 설계 (Seek 병합, SessionId+generation) | B6-3 |
| | 수명과 종료 순서 | B6-5 |
| `05-explorer-tabs.md` | 탐색기 트리 (egui_ltreeview, 지연 열거, notify, rfd) | B5-1, B5-5, B5-8 |
| | 탭·분할 (egui_tiles 를 기본으로, egui_dock 을 쓸 때의 버전 조건) | B5-1, B5-2 |
| | 탭 안 영상 표면과 OSD 오버레이 | B4-6, B5-6 |
| | 숨김 탭·동시 재생·오디오 포커스 정책 | B5-3, B5-4, B5-7 |
| `06-r1-hwdec.md` | hwdec 값과 의미 (auto, auto-safe=auto, auto-copy, no) | B7-3, B7-5 |
| | 재생 중 전환 (속성 변경, "무중단"이 아님) | B7-1 |
| | 자동 폴백 (`--hwdec-software-fallback`, 건너뜀 주의) | B7-2 |
| | 상태 표시와 전환 상태 기계 | B7-4 |
| `07-r2-video-options.md` | 밝기·대비·채도 | B7-6 |
| | 디인터레이스 | B7-7 |
| | 선명·흐림·노이즈 제거 (GLSL hook vs lavfi) | B7-8, B7-9 |
| | 실시간 조절의 한계 (필터 교체 vs 파라미터 변경, stall) | B7-10, B10-5 |
| | R1 과의 상호작용 (CPU 필터 → copy-back) | B7-5 |
| `08-playback-features.md` | 탐색·속도·음량 | B7-11 |
| | 트랙과 SRT/ASS 자막 | B7-11 |
| | 전체 화면 | B7-12 |
| `09-distribution-licensing.md` | stable-msvc 와 DLL 링크 | B4-8 |
| | libmpv 라이선스 (GPLv2+ / LGPLv2.1+) | B4-4, B10-9 |
| | FFmpeg·GStreamer·Slint 라이선스 | B4-5, B2-6 |
| `10-validation-and-open-questions.md` | 검증 계획 (파일·GPU·모니터 corpus, 측정 지표) | B10-2, B10-7 |
| | 미확인 목록과 재확인 날짜 | B10-1~B10-10, B9-3 |
| | 결정 기록 (채택·기각 근거) | B1-2, B8-1, B8-2 |
