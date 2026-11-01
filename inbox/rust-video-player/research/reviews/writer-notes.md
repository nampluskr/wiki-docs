# writer-notes — 묶음 A 본문 작성 중 만난 모순과 처리

- 작성: rvp-writer (2026-11-01)
- 원칙: 판단이 필요한 모순은 멈추지 않고 여기에 적고 **보수적으로** 썼다. 기존 `sources/`·`reviews/` 파일은 고치지 않았다.

## 1. 확인 수준 (먼저 읽을 것)

| 항목 | 내용 |
| --- | --- |
| URL | 본문의 출처 URL 은 보고서·comparison.md 가 적은 것을 옮겼다. writer 는 웹을 다시 열지 않았다. mpv 매뉴얼 `#anchor` 와 `DOCS/man/*.rst` 경로는 보고서 표기를 따른 것이라 앵커가 어긋났을 수 있다 |
| 로컬 소스 | writer 가 직접 연 것은 `workbench-gpui\specs\v0.1\DECISIONS.md`·`workbench-tauri\specs\v0.1\DECISIONS.md` 의 제목(D 번호)뿐이다. 나머지 로컬 소스 사실(`mod.rs`, `Cargo.toml`, gpui-pre 소스 줄 번호)은 sonnet·sol 보고서 값이다 |
| 판정 | comparison.md 2절 11건을 모두 반영했다. 반영 위치는 07-COMPARISON.md 7절 |

## 2. 모순과 처리

| # | 모순 | 처리 |
| --- | --- | --- |
| 1 | argon: `libmpv2` 6.0.0 / sonnet·sol: 7.0.0 | comparison 판정 1 → 7.0.0 |
| 2 | mpv LGPL 빌드 옵션: argon `--enable-lgpl` / sol·sonnet `-Dgpl=false`(sonnet 은 미확인) | 05 5.2 에 `-Dgpl=false` 를 sol 근거로 쓰되 "확정하지 않음"으로 표시. argon 표현은 mpv 옵션 이름과 안 맞는 것으로 보이나 확인 못 함 |
| 3 | hwdec 전환: argon "끊김 없이" / sol "무중단 보장 안 함" | sol 쪽(보수적) 채택. 05 1.1 |
| 4 | Tauri asset Range: argon 버그 주장(옛 이슈 #4775) / sonnet 미확인 / sol 처리함 | comparison 판정 3 → 처리함. 04 3.1 에 "실기 검증 항목" |
| 5 | mpv `sharpen`: argon 내장 속성(범위 0~2, 음수=흐림) | 판정 4 → `vo=gpu` 전용. argon 의 강도 범위·"음수=흐림"은 싣지 않음 (같은 보고서 안에서도 범위가 `0.0~2.0` 과 `-1.0~0.0` 로 어긋남) |
| 6 | `d3d11vpp`: argon 이 노이즈 제거 포함, 옵션 문법 `deint=yes` 사용 | 판정 5 → 노이즈 제거 없음. 옵션 문법은 확인 안 돼 싣지 않음. Windows API 의 NOISE_REDUCTION enum 은 별개라고 구분해 06 1절에 적음 |
| 7 | argon: `cas`, `atadenoise` 를 LGPL 로 단정 | 단독 주장이라 06 4절에 "미확인"으로 표시 |
| 8 | argon: `pp` 필터 GPL / sonnet 목록에는 없음 | argon 단독 값으로 표시, 대체 수단 "미확인" |
| 9 | argon: mpv `hue` 속성 포함 / sonnet 은 세 속성만 | 06 1절에 "argon 보고"로 표시 |
| 10 | `rubato` 5.0.1: argon 만 조회, sonnet "미확인" | 02 5.2 에 "argon 단독 값"으로 표시 |
| 11 | `ass-renderer` 0.1.2, `srtlib` 0.2.0: argon 단독 | 대안·단독 값으로만 표시 (comparison.md 3절 처리와 동일) |
| 12 | 크기: argon libmpv 65~85 MB / sol Cathode 112 MB; GStreamer argon 80~150 MB / sonnet 수백 MB | 판정 10 → 범위·예시만. 두 값이 다른 것은 그대로 병기 (05 4.6) |
| 13 | video-rs 별: sonnet 428 / argon ~500+ (추정) | 428 (확인값) 채택 |
| 14 | gyroflow 별: argon ~9,700+ / comparison 9,660 | 9,660 채택 |
| 15 | tauri-plugin-mpv 별: argon ~30 / sonnet 미조회 | 04 4.3 표에는 별을 적지 않음. 라이선스는 sonnet "미확인" 유지 |
| 16 | D-2 의 뜻: 보고서·comparison 은 "GPL 의존 금지"로 줄여 부름 | 실제 DECISIONS.md 제목은 "Zed GPL 코드는 복사·의존하지 않고 Apache-2.0 의 gpui·gpui-kit 만 의존". 본문은 이 제목대로 적고 "GPL 허용과 다르다"는 점을 05 5.1 에 정리 |
| 17 | alhazen-core README "Windows AC3 설치 불필요" vs Windows 11 24H2 자료 | 충돌 가능으로 표시, 검증 필요 (03 5절) |
| 18 | `egui-video` 상태, `gpui-video-player` 별 등 11건 | comparison 판정 그대로 |
| 19 | argon: `ID3D11Multithread`·텍스처 배열 슬라이스 고갈 | 근거 문서 미첨부라 "미확인"으로 05 1.5 에 한 줄 (libmpv 경로에서는 해당 없음) |
| 20 | argon: VRAM 200~400 MB, 보이는 탭 2~4개 | "추정"으로 유지 (01 4절) |
| 21 | 프레임 드롭 임계값(40 ms, 500 ms) 등 argon 수치 | "예시 값, 근거 문서 없음"으로 표시 (01 3.3) |
| 22 | gpui 에서 외부 texture 공개 API: sol "별도 renderer extension 필요" / sonnet "찾지 못함(미확인)" | "찾지 못했다(미확인)"로 표현 (03 3.3) |
| 23 | sol Q16(목차 13항) / argon Q16(7편) / sonnet Q16 | outline.md 확정 목차를 그대로 따름 |
| 24 | 동영상 사례 중 Kenichi Preview: sonnet 이 검색 요약만 확인 | 04 6절에 그 사실 표시 |

## 3. 제외한 것

| 내용 | 이유 |
| --- | --- |
| PotPlayer 메뉴 경로·단축키 (argon Q8) | 사용자 결정(기능 이름 기준). 출처 없음 |
| mpv 쪽 GPL/LGPL 코드, Zed 앱 코드 | 규칙. 사실(값·이름·동작)만 서술 |
| 수식(LaTeX) | 문서 형식에 맞춰 문장으로 풀어 씀 (01 3.3) |
| argon Q16 의 파일 구성 | outline.md 가 우선 |

## 4. 구현 전에 사람이 확인할 것

- mpv 매뉴얼 앵커와 `DOCS/man/*.rst` 링크가 실제로 열리는지
- LGPL libmpv 를 만들 경우의 정확한 빌드 옵션
- lavfi 필터를 `vf` 에 넣을 때 하드웨어 프레임 처리 (06 3절)
- gpui 창 HWND 노출 방법 (03 3.2)

## 5. leader 검토 반영 (2026-11-01, rvp-leader)

| 항목 | 처리 |
| --- | --- |
| 판정 11건 반영 | 본문 grep 으로 확인. 모두 반영됨 |
| 상대 링크 | 깨진 링크 없음 |
| mpv 매뉴얼 앵커 | `#options-hwdec` 같은 옵션 단위 앵커는 실제 페이지에 없다. 절 단위 앵커로 바꿈: hwdec·deinterlace → `#video`, vid → `#track-selection`, wid·fullscreen → `#window`, glsl-shaders → `#gpu-renderer-options`, speed → `#playback-control`, audio-device → `#audio`, blend-subtitles → `#subtitles`, video-sync → `#video-sync`, seek → `#playback-control-1` |
| 외부 URL 145개 | 200 이 아닌 것: potplayer.daum.net(응답 없음), neowin(403, 봇 차단), crates.io 웹 페이지 2개(봇 응답 404, API 로 존재 확인). 본문 수정 없음 |
| mpv LGPL 빌드 옵션 (모순 2) | `meson.options` 의 `gpl` boolean 확인 → 05-WINDOWS.md 5절을 `-Dgpl=false` 로 확정 |
