# 묶음 A 목차 초안

제목: **Rust 동영상 플레이어 구현 가이드 — gpui·Tauri 기반 (Windows 11)**
폴더: `D:\projects\docs\guides\rust-video-player\`
근거: `research/reviews/comparison.md`, 세 보고서의 Q16 권장 목차

| 파일 | 담을 내용 | 근거 항목 |
| --- | --- | --- |
| README.md | 입구, 결론 요약(권장 조합 한 장), 대상·기준·검증 범위, 읽기 지도 | Q1, Q16 |
| 01-ARCHITECTURE.md | 요구(PotPlayer 기준, R1·R2 정의), 공통 `PlayerSession`(상태 기계·명령·이벤트·generation), 파이프라인·A/V 동기·탐색, 탭 동작 정책(전환·분할·숨김·동시 재생·전체 화면) | Q2, Q11 |
| 02-DECODING.md | 백엔드 비교(libmpv·FFmpeg·GStreamer·Media Foundation·순수 Rust), libmpv 를 권하는 이유와 대안, 크레이트 표 | Q3, Q13 |
| 03-GPUI.md | workbench-gpui 현황, `PaneItem` 설계, 출력 3단계(RenderImage PoC → 자식 HWND 실용안 → D3D11 render API 장래), 컨트롤 UI, 사례 | Q4, Q14 |
| 04-TAURI.md | workbench-tauri 현황, 탭 모델·Rust 세션 레지스트리·IPC, `<video>` 한계, 자식 HWND 좌표 동기화, 투명 WebView·TextureStream 대안, 사례 | Q5, Q14 |
| 05-WINDOWS.md | R1(hwdec·D3D11VA·자동 전환·실제 상태 표시), 소리, 자막, 빌드·DLL·번들·크기 | Q6, Q9, Q10, Q12 |
| 06-FILTERS.md | R2 기능별 구현표(mpv 속성·lavfi·셰이더·D3D11 VPP), 실시간 조절, CPU vs GPU, GPL 필터 표, PotPlayer 대응표 | Q7, Q8, Q12 |
| 07-COMPARISON.md | 방식별 비교와 최종 선택, 위험 3가지·자주 틀리는 점, 검증 체크리스트, 확신 낮은 부분 | Q15, Q17 |
| research/ | 조사 원자료(sources·reviews) 보관, README 로 안내 | — |

형식: `docs/guides/rust-gui/README.md` 와 같게 (frontmatter `tags`·`created`·`updated`, "이 파일을 읽는 때" 인용, 읽기 지도 표). 모든 사실에 출처 URL.
