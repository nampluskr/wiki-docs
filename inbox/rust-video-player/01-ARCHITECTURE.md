---
tags: [rust, video-player, architecture, session, tabs]
created: 2026-11-01
updated: 2026-11-01
---

# 01 · 요구와 공통 구조

> 이 파일을 읽는 때: 엔진·UI 를 고르기 전에 요구와 공통 설계를 정할 때. gpui·Tauri 가 같이 쓰는 `PlayerSession` 과 탭 정책이 여기에 있다.

입구는 [README.md](README.md)다.

## 1. 요구

### 1.1 재생 범위 (PotPlayer 기준)

PotPlayer 급 범용 재생을 목표로 한다. 기준은 **기능 이름**이다. PotPlayer 의 메뉴 경로·기본값·알고리즘은 확인하지 못했다. https://potplayer.daum.net/ 는 배포 확인용이고 메뉴 근거로 쓰지 않았다.

| 항목 | 요구 |
| --- | --- |
| 컨테이너 | MP4, MKV, WebM, TS |
| 영상 코덱 | H.264, HEVC, VP9, AV1 |
| 오디오 코덱 | AAC, AC3, Opus |
| 자막 | SRT, ASS(내장·외부) |
| 조작 | 재생·일시정지, 탐색, 속도, 음량, 오디오·자막 트랙 선택, 전체 화면 |

출처: 범위는 사용자 요구. 이 범위를 mpv 가 다루는지는 https://mpv.io/manual/stable/ 로 확인한다 (코덱 지원은 포함된 FFmpeg 빌드에 달렸다).

### 1.2 R1 — GPU 가속 ON/OFF

| 수용 기준 | 설명 |
| --- | --- |
| ON/OFF | 하드웨어 디코딩을 사용자가 켜고 끈다 |
| 재생 중 전환 | 파일을 다시 열지 않고 바꾼다. 위치·일시정지·트랙·필터를 보존한다 |
| 자동 전환 | 하드웨어 디코딩이 실패하면 소프트웨어로 바꾼다 |
| 실제 상태 표시 | "요청값"과 "실제 사용 중인 방식"을 따로 보여 준다 |

구현은 [05-WINDOWS.md](05-WINDOWS.md) 1절.

### 1.3 R2 — 영상 옵션

| 항목 | 수용 기준 |
| --- | --- |
| 선명 / 흐림 | 켜기·끄기, 강도 슬라이더 |
| 노이즈 제거 | 켜기·끄기, 강도 |
| 디인터레이스 | 켜기·끄기·자동 |
| 밝기 / 대비 / 채도 | 슬라이더, 초기화 |
| 공통 | 재생 중 실시간 반영, 끈 뒤에도 마지막 강도 보존 |

구현은 [06-FILTERS.md](06-FILTERS.md).

### 1.3a R3 — 유튜브 링크 재생 (2026-11-01 추가)

| 항목 | 수용 기준 |
| --- | --- |
| 진입 | 텍스트·미리보기 탭의 유튜브 링크를 클릭하면 새 영상 탭이 열린다 |
| 화면 | 로컬 파일과 같은 플레이어 화면·컨트롤 |
| R1·R2 | 그대로 적용된다 |
| 실패 | 로그인·지역 제한·차단 등은 사유와 재시도를 보여 준다 |

세션의 소스를 `Path` 와 `Url` 두 가지로 나눈다(2절). 나머지 세션 구조는 같다. 동작·설정·yt-dlp·약관은 묶음 A·B 공통 문서 [../rust-video-player-youtube.md](../rust-video-player-youtube.md).

### 1.4 앱 형태

| 항목 | 내용 |
| --- | --- |
| 앱 | workbench-gpui 또는 workbench-tauri 를 베이스로 하는 **별도 앱** |
| 진입 | 파일 트리에서 동영상을 열면 탭이 생기고 그 안에서 재생 |
| 베이스 현황 | 둘 다 Pane/Item 추상이 아직 없다. 새로 설계한다 ([03-GPUI.md](03-GPUI.md) 1절, [04-TAURI.md](04-TAURI.md) 1절) |

## 2. 공통 설계: `PlayerSession`

원칙: **재생 상태는 뷰(위젯)가 아니라 세션 객체에 둔다.** 뷰는 세션을 보여 주기만 한다. 탭 전환·분할·숨김에서 뷰가 사라져도 세션이 살아 있어야 위치와 설정이 유지된다. 이 원칙은 workbench-gpui D-7(상태는 gpui Entity, `cx.notify()`)과 맞는다. `D:\projects\workbench-gpui\specs\v0.1\DECISIONS.md`

세션은 엔진(libmpv)을 감싼다. gpui 와 Tauri 가 **같은 세션 코드**를 쓰고, UI 쪽 어댑터만 다르다. 권고 근거: https://mpv.io/manual/stable/#embedding-into-other-programs-libmpv (sol 보고서).

세션의 소스는 두 가지다 (R3, 설계 예시): `enum Source { Path(PathBuf), Url { original: Url, start: Option<f64> } }`. `Url` 은 원본 watch URL 만 저장하고, 추출된 signed URL·쿠키는 저장하지 않는다. 화질을 바꿀 때는 원본 URL 로 다시 load 한다. 상세는 [../rust-video-player-youtube.md](../rust-video-player-youtube.md).

### 2.1 상태 기계

| 상태 | 뜻 | 다음 |
| --- | --- | --- |
| Closed | 세션 없음 또는 정리됨 | Opening |
| Opening | 파일 열기·probe 중 | Buffering, Playing, Error |
| Buffering | 데이터 대기 | Playing |
| Playing | 재생 중 | Paused, Seeking, Ended |
| Paused | 일시정지 | Playing, Seeking |
| Seeking | 탐색 중 | Playing 또는 Paused |
| Ended | 끝까지 재생 | Seeking, Closed |
| Error / Recovering / Closing | 오류, 복구 중, 닫는 중 | 별도 상태로 둔다 |

출처: 상태 목록은 https://github.com/alhazen-rs/alhazen-core#events-and-state 같은 실제 상태 모델과 비교한 sol 보고서의 **설계 제안**이다. 에러는 `Error(kind)` 로 분리하고, `Seeking` 의 중첩 요청은 마지막 값으로 합친다 (sonnet 보고서, 설계 제안).

**독립 상태 셋.** 필터 설정, 요청한 hwdec, 실제 hwdec 는 재생 상태와 따로 저장한다. 요청과 실제가 다르기 때문이다. https://mpv.io/manual/stable/#property-list

### 2.2 명령과 이벤트

| 구분 | 이름 | 비고 |
| --- | --- | --- |
| 명령 | open, pause, seek, set_speed, set_volume | UI → 세션 |
| 명령 | select_track(audio/sub), add_subtitle | |
| 명령 | set_hwdec, set_filter | R1, R2 |
| 명령 | set_bounds, set_visible | 네이티브 표면 위치 ([03](03-GPUI.md), [04](04-TAURI.md)) |
| 명령 | close | |
| 이벤트 | state_changed, position, duration | 세션 → UI. 위치는 낮은 빈도로 |
| 이벤트 | tracks_changed, hwdec_changed, error | 모든 이벤트에 세션 ID·generation 을 붙인다 |

출처: 명령 목록 https://v2.tauri.app/develop/calling-rust/ (sol 보고서의 Tauri 명령 설계), 이벤트 모델 https://github.com/mpv-player/mpv/blob/master/include/mpv/client.h.

### 2.3 generation

seek·파일 교체·R1 전환 때마다 **generation 번호를 올린다.** 이전 generation 의 프레임·이벤트·큐 결과는 버린다. 늦게 도착한 결과가 새 화면을 덮어쓰는 사고를 막는다. 출처: sol 보고서 Q2 (설계 제안). https://github.com/alhazen-rs/alhazen-core#events-and-state

설계 예시 (구조만, 컴파일 검증 안 함):

```rust
enum State { Closed, Opening, Buffering, Playing, Paused, Seeking, Ended, Error }
struct PlayerSession {
    id: SessionId, generation: u64, state: State,
    requested_hwdec: bool,      // 사용자가 요청한 값
    actual_hwdec: Option<String>, // hwdec-current 에서 읽은 값
    filters: FilterSettings,    // ON/OFF 와 강도를 따로
}
```

## 3. 파이프라인

### 3.1 libmpv 를 쓰면

demux·디코딩·필터·동기·오디오 출력을 mpv 가 내부에서 처리한다. 세션은 명령과 속성만 다룬다. 아래 3.2~3.4 는 **직접 구현(FFmpeg) 경로에서만** 필요하고, 엔진 선택 이유 설명용으로도 둔다.

### 3.2 직접 구현 시 구조

```
file ─ demux ─┬─ video packet queue ─ decode(HW/SW) ─ filter ─ frame queue ─ present
              ├─ audio packet queue ─ decode ─ resample ─ PCM ring buffer ─ 오디오 콜백
              └─ subtitle packet queue ─ decode ─ libass ─ 영상 위에 합성
```

출처: 구조 https://ffmpeg.org/doxygen/trunk/group__lavc__encdec.html , https://github.com/FFmpeg/FFmpeg/blob/master/fftools/ffplay.c (argon 보고서). 상위 구조는 ffplay/mpv 공통이다.

| 항목 | 권장 | 출처·성격 |
| --- | --- | --- |
| 스레드 | UI, 세션 제어, demux, decode/filter, render, 오디오 콜백으로 역할을 나눈다 | sol 보고서 설계 제안 |
| 오디오 콜백 | 잠금·할당·디스크 I/O·디코딩 금지. 링버퍼만 읽는다 | https://docs.rs/cpal/0.18.2/cpal/ |
| 패킷 큐 | 바이트·시간 상한을 둔다. 가득 차면 상류가 대기(역압) | sonnet·argon 보고서, 수치는 예시 |
| 프레임 큐 | 비디오 3~8장 | argon 보고서, 예시 값 |
| 메모리 | 4K BGRA 1장 = 33.18 MB. 30장 큐만 약 995 MB | 산술값 (3840×2160×4). https://docs.rs/gpui-pre/0.3.10/gpui/struct.RenderImage.html |
| libmpv render callback | 안에서 UI 나 일반 mpv 명령을 동기 호출하지 않는다. 깨우기 신호만 보낸다. 상호 대기는 데드록 위험 | https://github.com/mpv-player/mpv/blob/master/include/mpv/render.h |

### 3.3 A/V 동기와 속도

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 기준 시계 | 실제 출력 중인 오디오의 PTS 와 장치 지연을 기준으로 한다. 오디오 없는 파일은 단조 시계 | https://gstreamer.freedesktop.org/documentation/application-development/advanced/clocks.html |
| 늦은 프레임 | 늦으면 버린다. 이르면 기다린다 | https://github.com/FFmpeg/FFmpeg/blob/master/fftools/ffplay.c |
| 시간 | PTS 를 time_base 로 환산한다. DTS·프레임 번호로 대신하지 않는다 | https://ffmpeg.org/doxygen/trunk/structAVFrame.html |
| 속도 | 시계 배율과 오디오 time-stretch 를 같이 바꾼다. 프레임 sleep 만 바꾸면 소리가 안 맞는다 | https://ffmpeg.org/ffmpeg-filters.html#atempo |
| mpv | `speed` 와 `audio-pitch-correction`(기본 켜짐)을 쓴다. 속도가 정상이 아니면 `scaletempo2` 를 자동 삽입한다고 문서에 있다 | https://mpv.io/manual/stable/#playback-control |

오디오 시계 식(argon 보고서): `C_audio = PTS_last_written − (링버퍼 잔량 + WASAPI 지연) / 초당 바이트`. 프레임 드롭 임계값(40 ms, 500 ms)은 argon 보고서의 예시 값이고 근거 문서는 없다 (추정).

### 3.4 탐색

| 종류 | 방법 | 용도 |
| --- | --- | --- |
| 빠른 탐색 | 가까운 키프레임으로 이동 | 슬라이더 드래그 |
| 정확 탐색 | 앞 키프레임으로 간 뒤 목표 PTS 까지 디코딩만 하고 버린다 | 슬라이더 놓을 때 |

직접 구현의 순서: ① generation 증가 → ② 모든 큐 비우기, 디코더 flush → ③ demux seek → ④ 새 generation 패킷만 수용 → ⑤ 오디오 링버퍼 비우고 시계 재설정. 출처: https://ffmpeg.org/doxygen/trunk/group__lavf__decoding.html (sonnet·sol·argon 공통). mpv 는 `seek` 명령의 `keyframes`/`exact` 플래그와 `hr-seek` 가 대응한다. https://mpv.io/manual/stable/#playback-control-1

## 4. 탭 동작 정책

공통 정책이다. 네이티브 표면 처리(HWND 숨김·이동)는 [03-GPUI.md](03-GPUI.md) 3.2, [04-TAURI.md](04-TAURI.md) 4절.

| 상황 | 정책 | 근거·주의 |
| --- | --- | --- |
| 탭 전환 | 숨는 탭은 **기본 일시정지**, 마지막 프레임·위치 유지. 옵션으로 "백그라운드 재생" | sonnet·sol·argon 공통. https://mpv.io/manual/stable/#property-list |
| 숨김 탭 | 영상 출력·렌더 요청을 멈춘다. 자식 HWND 는 hide | 단순 창 숨김만으로 디코더 사용량이 줄지 않는다 (sol). 디코더 해제는 N분 뒤 (sonnet 제안) |
| 백그라운드 재생 | 오디오 유지, 영상 디코딩 중단(mpv `vid=no` 등). 복귀 시 현재 위치에서 재개 | argon 제안. 복귀 시 seek·트랙 초기화 영향은 검증 필요 (sol). https://mpv.io/manual/stable/#track-selection |
| 분할 | 같은 파일의 두 번째 뷰는 같은 세션 공유 여부를 명시한다. 다른 파일은 독립 세션 | mpv core 하나에 render context 는 1개라는 제약 (sol). https://github.com/mpv-player/mpv/blob/master/include/mpv/render.h |
| 여러 탭 동시 재생 | 기본은 **보이는 탭으로 제한**, 소리는 활성 탭만. 나머지는 일시정지 또는 음소거 | 하드웨어 디코더 동시 세션 수는 GPU마다 다르다. 정확한 수 **미확인** (sonnet·sol) |
| VRAM | 4K HEVC 탭 하나가 200~400 MB 쓴다. 보이는 탭 2~4개로 제한 | **추정** (argon 보고서, 근거 문서 없음) |
| 닫기 | 이벤트 배달 차단 → 렌더러·자식 창 정리 → core 종료. libmpv render context 는 core 보다 먼저 해제 | https://github.com/mpv-player/mpv/blob/master/include/mpv/render.h |
| 전체 화면 | 활성 pane 만 확대하고 기존 분할·활성 탭·포커스를 복원한다. 호스트 창 전체 화면을 우선, HWND 재부모화는 피한다 | https://learn.microsoft.com/en-us/windows/win32/winmsg/window-features#child-windows |
| 창 최소화 | 일시정지 또는 렌더만 중단 (옵션) | sonnet 보고서 제안 |

전체 화면에서 mpv 자식 창의 독립 fullscreen 과 앱 fullscreen 을 중복 적용하지 않는다. https://mpv.io/manual/stable/#window

---

다음: [02-DECODING.md](02-DECODING.md)
