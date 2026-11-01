# 묶음 A 조사 비교 (sonnet · sol · argon)

- 작성: rvp-leader (2026-11-01)
- 대상: `research/sources/sonnet/report-1.md`, `report-2.md`, `sol/report.md`, `argon/report.md`
- 표기: ●●● 셋 다 일치 · ●● 둘만 일치 · ✕ 상충(판정 포함) · ○ 한쪽만

## 1. 셋 다 일치 (본문의 뼈대)

| 항목 | 내용 |
| --- | --- |
| Q1 gpui 권장 | libmpv(`libmpv2`)가 1안. 디코딩·탐색·트랙·자막·동기화·R1·R2 를 mpv 에 맡긴다 |
| Q1 Tauri 권장 | 같은 libmpv 엔진을 Rust 쪽에 두고 네이티브 표면(자식 HWND) 사용. WebView 는 제어·위치만 |
| Q1 `<video>` 단독 | 범용(MKV/TS·AC3·HEVC·ASS)·R1·R2 를 보장하지 못한다 |
| Q4 gpui 출력 | `paint_surface` 는 macOS/iOS 전용. Windows 는 `RenderImage`(CPU BGRA → 아틀라스 업로드)만 공개 경로. 4K60 BGRA ≈ 1.99 GB/s 로 병목 |
| Q4 현황 | workbench-gpui 는 `editor_poc` 단일 슬롯·고정 탭. Pane/Item 추상이 없어 새로 설계해야 한다 |
| Q5 현황 | workbench-tauri 는 `App.tsx`·`lib.rs` 빈 껍데기 |
| Q4·Q5 자식 HWND | 탭 영역 좌표(물리 픽셀·DPI)에 맞춰 `SetWindowPos`, 숨김 탭은 hide. 영상 위 메뉴·툴팁이 가려지는 airspace 문제 → 컨트롤을 영상 밖에 둔다 |
| Q6 R1 | mpv `hwdec=auto-safe`(=auto) ↔ `no` 를 재생 중 변경, 실패 시 SW 자동 전환. D3D11VA 우선, DXVA2 는 구형 폴백. 실제 상태는 `hwdec-current` 로 표시 |
| Q7 R2 | 밝기·대비·채도는 mpv 속성(-100~100). FFmpeg `eq`·`hqdn3d` 는 GPL 전용. LGPL 쪽은 `unsharp`·`gblur`·`yadif`·`bwdif`·`nlmeans`(느림) |
| Q7 HW+CPU 필터 | HW 디코딩 + CPU 필터는 VRAM↔RAM 왕복으로 R1 이득이 줄어든다 |
| Q3 순수 Rust | symphonia(오디오)·rav1d(AV1)·openh264(H.264)만으로는 범용 재생 불가 |
| Q9 소리 | mpv/GStreamer 는 자체 WASAPI 출력 사용. FFmpeg 직접일 때만 `cpal` 0.18.2 + 링버퍼 |
| Q10 자막 | ASS 는 libass(mpv 내장)가 실용적. Rust `libass` 크레이트는 2020 이후 정지 |
| Q11 탭 | 숨김 탭 기본 일시정지, 동시 재생은 보이는 탭으로 제한, 재생 상태는 뷰가 아니라 세션 객체에 |
| Q12 | MSVC/MinGW 산출물 혼용 금지, DLL 옆 배치(Tauri 는 `bundle.resources`), 특허는 오픈소스 라이선스와 별개 |

## 2. 상충과 판정 (leader 가 출처를 직접 열어 확인, 2026-11-01)

| # | 항목 | 주장 | 판정 | 근거 |
| --- | --- | --- | --- | --- |
| 1 | `libmpv2` 최신 버전 | sonnet·sol 7.0.0 / argon 6.0.0 | **7.0.0 (2026-10-30)** | crates.io API `max_stable_version` |
| 2 | mpv D3D11 render API | sol: master 전용 / argon: 쓸 수 있는 것처럼 서술 | **master 에만 있음** (커밋 2026-10-15). 최신 정식 v0.41.0(2025-12-21)에는 없음. `libmpv2` 도 미지원 | GitHub commit `4255130d`, releases/latest |
| 3 | Tauri asset 프로토콜 Range | sol: 내장 핸들러가 Range 처리 / argon: 버그로 루프백 HTTP 서버 필요 | **내장 핸들러가 206·`Accept-Ranges`·416 처리** (argon 근거는 옛 이슈). 실기 검증 항목으로 남긴다 | `tauri/crates/tauri/src/protocol/asset.rs` (dev) |
| 4 | mpv `sharpen` | argon: 내장 속성(음수=흐림) / 나머지: 언급 없음 | **있음, 단 `--vo=gpu` 전용**. sol 권장 `vo=gpu-next` 에서는 안 됨 → 셰이더 또는 lavfi 로 대체 | mpv `options.rst` `--sharpen` |
| 5 | mpv `d3d11vpp` 노이즈 제거 | argon: 디인터레이스·노이즈 제거 / sol: 디인터레이스 | **디인터레이스·스케일·Super Resolution 만**, 노이즈 제거 옵션 없음 | mpv `vf.rst` `d3d11vpp` |
| 6 | `egui-video` 상태 | sonnet: 2024-10 정체 / argon: 활발 | **정체** (crate·push 모두 2024-10-28) | crates.io, GitHub API |
| 7 | `gpui-video-player` 별 | sonnet·sol 97 / argon ~45 | **97** | GitHub API |
| 8 | `rsmpeg` 유지 | sonnet: 1년+ 정체 / argon: 활발 | **최근 릴리스 2025-08-24** (1년+) | crates.io |
| 9 | Tauri mpv 플러그인 이름 | sonnet: 두 개 구분 / argon: `tauri-plugin-mpv` 로 섞음 | **둘 다 nini22P**: `tauri-plugin-libmpv` 0.3.2(라이브러리 임베드), `tauri-plugin-mpv` 0.5.2(프로세스 IPC) | crates.io |
| 10 | libmpv DLL 크기 | argon 65~85 MB / sol: Cathode 예 약 112 MB | **빌드마다 다름**. 범위와 예시만 적는다 | 각 보고서 출처 |
| 11 | `ffmpeg-next` 유지 | sol·argon: 유지보수 모드 / sonnet: 활발 | **유지보수 모드** (README 명시, 9.0.0 은 2026-08-05) | GitHub README |

## 3. 한쪽만 언급 (확인 후 반영 여부)

| 출처 | 내용 | 처리 |
| --- | --- | --- |
| sol | `alhazen-core`·`alhazen-gpui` 0.6.0 (gpui 어댑터 있는 신생 엔진) | 사례 표에 "검증 필요" 로 |
| sol | get-air `tauri-video-plugin`: GStreamer D3D11 → WebView2 TextureStream(실험 API) | Tauri 대안 절에 "실험" 으로 |
| sol | Windows 11 24H2 부터 AC3 가 OS 기본에서 빠짐 | WebView2/MF 한계 근거로 |
| sol | Zed 화면 공유 뷰가 현재·직전 프레임만 두고 `drop_image` | gpui 아틀라스 수명 근거로 (코드 복사 금지) |
| sonnet | `gpui-video-player` 는 `gpui-pre =0.3.9` 고정 → 로컬 0.3.10 호환 확인 필요 | 03-GPUI 주의점 |
| argon | `ID3D11Multithread::SetMultithreadProtected`, D3D11VA 텍스처 배열 슬라이스 고갈 | FFmpeg 직접 경로 위험으로 |
| argon | 4K HEVC 탭당 VRAM 200~400 MB 추정, 보이는 탭 2~4 개 제한 | 추정 표시로 |
| argon | `ass-renderer` 0.1.2 (순수 Rust ASS, wiedymi/ass-rs, 2026-07) | 대안으로만 |
| argon | `gyroflow`(별 9,660): FFmpeg D3D11 → wgpu 무복사 | 묶음 B 쪽에 더 맞음. A 에서는 사례로만 |
| argon | PotPlayer 메뉴 경로·단축키(Ctrl+I, Ctrl+R 등) | **미확인** (출처 불명). 사용자 확인 필요 |

## 4. 사용자 결정 (2026-11-01)

- GPL: **허용**. 일반 libmpv 빌드와 GPL 필터(`eq`·`hqdn3d`)를 그대로 쓸 수 있다. workbench-gpui D-2(GPL 금지)와 다르다는 점과 LGPL 로 바꿀 때의 대체 수단은 짧게 밝힌다.
- PotPlayer 대응표: **기능 이름 기준**. 메뉴 경로·단축키는 넣지 않는다.
- 목차: outline.md **그대로 작성**.

## 4-1. (당시) 사용자 판단 필요

1. **GPL 허용 여부**: workbench-gpui DECISIONS D-2 는 GPL 의존 금지. 일반 배포 libmpv(shinchiro 등)는 GPL 빌드가 많고, FFmpeg `eq`·`hqdn3d` 도 GPL. → (가) GPL 허용 (나) LGPL 만: LGPL libmpv 빌드 + 셰이더/LGPL 필터로 대체
2. **PotPlayer 메뉴 대응표의 기준**: argon 의 메뉴 경로·단축키는 출처가 불분명. 사용자 PotPlayer 의 실제 메뉴를 기준으로 할지, 기능 이름 기준 표로 둘지
