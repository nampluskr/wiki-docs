---
tags: [rust, video-player, architecture, libmpv, threading]
created: 2026-11-01
updated: 2026-11-01
---

# 04. 플레이어 코어와 UI 분리

> 이 파일을 읽는 때: 크레이트를 나누고, 스레드와 채널을 정할 때. 교착(deadlock)을 피하는 규칙도 여기 있다.

이 파일의 크레이트 구성과 채널 설계는 **설계 제안**이다(세 scouter 보고서 B6 의 공통점을 정리). 사실로 적은 것은 mpv `render.h` 의 규칙뿐이며, 각 사실에 출처를 달았다.

## 1. 크레이트 구성

```text
crates/
├── player-api     SessionId, Command, Event, PlayerState, Capabilities (UI·엔진 타입 없음)
├── player-mpv     libmpv2 래퍼: 명령·속성 관찰·이벤트 루프·엔진 수명
├── render-gl      GL 컨텍스트 / FBO / mpv render API / egui paint callback 연결
├── library-fs     디렉터리 비동기 열거·파일 감시·경로 처리
├── settings       키맵·탭 복원·R2 프리셋·버전 이전
└── app            eframe 앱: 트리·탭·OSD·전체 화면 정책
```

| 규칙 | 이유 |
| --- | --- |
| `player-api` 에는 egui·libmpv 타입을 넣지 않는다 | UI 를 바꾸거나(예: Win32), 엔진을 바꾸거나(예: FFmpeg 직접), 렌더를 바꿔도(0.42 이후 wgpu) API 가 유지된다 |
| 렌더 크레이트는 경로마다 하나씩 둔다 (`render-gl`, 나중에 `render-wgpu`) | [01-stack-decision.md](01-stack-decision.md) 의 단계안에서 렌더만 갈아 끼운다 |
| 엔진 포인터(`mpv_handle`, render context)는 `player-mpv`·`render-gl` 밖으로 내보내지 않는다 | 수명과 스레드 규칙을 한곳에서 지킨다 |

크레이트 수는 보고서마다 4~7개로 다르다. 이것은 설계 선택일 뿐 정답이 있는 문제가 아니다. 위 구성은 sol 보고서의 7개 구성에서 Win32 대안 크레이트를 뺀 것이다.

## 2. 스레드 모델

| 스레드 | 하는 일 | 하지 않는 일 |
| --- | --- | --- |
| UI 스레드 (eframe) | 위젯·입력·레이아웃. glow 경로에서는 paint callback 안에서 `mpv_render_context_render` 를 호출한다 | 블로킹 I/O, mpv 일반 API 동기 호출, 제어 스레드 기다리기 |
| 세션 제어 워커 (탭마다) | Command 를 받아 mpv 명령·속성을 설정하고, mpv 이벤트를 Event 로 바꾼다 | 렌더 |
| mpv 내부 스레드 | 디먹스·디코드·오디오 출력 (mpv 가 관리한다) | — |
| 파일 열거 워커 | 디렉터리 열거·`notify` 이벤트 디바운스 | UI 상태 직접 수정 |

### 2.1 mpv render API 의 스레드 규칙 (사실)

mpv 0.41.0 `render.h` 에 이렇게 적혀 있다 ([render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)).

| 규칙 | 원문 요지 |
| --- | --- |
| 렌더는 별도 스레드가 좋다 | "Preferably rendering should be done in a separate thread. If you call normal libmpv API functions on the renderer thread, deadlocks can result" |
| 동시 호출 금지 | mpv_render_* 함수는 한 번에 하나만 호출한다(서로 다른 core 에 속하면 예외) |
| 콜백 안에서 호출 금지 | wakeup 콜백·update 콜백 안에서 mpv_render_* 를 호출하면 안 된다 |
| GL 컨텍스트 | OpenGL 백엔드면 호출 스레드에서 **같은** GL 컨텍스트가 current 여야 한다 |
| 대기 관계 금지 | 렌더 스레드가 "안전하지 않은" libmpv 함수의 반환을 기다리는 상황이 생기면 안 된다. 어기면 타임아웃 후 품질이 떨어지고, `mpv_render_context_render() not being called or stuck.` 로그가 남는다 |
| update 콜백 | `MPV_RENDER_PARAM_ADVANCED_CONTROL` 을 켜면 update 콜백이 새 프레임 없이도 불릴 수 있다. 이때는 `mpv_render_context_update()` 의 반환값으로 그릴지 정한다 |

### 2.2 이 규칙에서 나온 설계

eframe 의 glow paint callback 은 UI 스레드에서 실행된다. 그래서 **UI 스레드가 곧 렌더 스레드**가 된다. 2.1 의 규칙을 지키려면 이렇게 한다.

1. UI 스레드는 mpv 명령·속성 설정을 **직접 부르지 않는다**. 모두 Command 채널로 세션 제어 워커에 넘긴다.
2. update 콜백은 원자 플래그를 세우고 `egui::Context::request_repaint()` 만 호출한다. 다른 일은 하지 않는다.
3. 렌더 락과 명령 락은 서로를 기다리지 않게 한다.
4. 렌더 스레드를 따로 두는 구성(공유 텍스처 필요)은 첫 버전 범위에서 뺀다. 필요해지면 [03-rendering-pipeline.md](03-rendering-pipeline.md) 의 wgpu 경로에서 다시 설계한다.

`libmpv2` 7.0.0 래퍼가 이 규칙을 어디까지 대신 지켜 주는지는 **미확인**이다 ([libmpv2 7.0.0 render](https://docs.rs/libmpv2/7.0.0/libmpv2/render/index.html)).

## 3. Command / Event

```rust
pub enum Command {
    Open { source: MediaSource },    // Path 또는 Url (6절, R3)
    SetPaused(bool),
    Seek { target: Duration, exact: bool },
    SetSpeed(f64),
    SetVolume(f64),
    SelectTrack { kind: TrackKind, id: Option<i64> },
    SetHwdec(HwdecRequest),          // R1
    SetVideoOptions(VideoOptions),   // R2
    Close,
}

pub enum Event {
    Loaded { duration: Option<Duration> },
    Tracks(Vec<TrackInfo>),
    Position(Duration),
    HwdecChanged { requested: HwdecRequest, actual: String }, // actual = hwdec-current
    VideoOptionsApplied { result: Result<(), String> },
    Error(String),
    Ended,
}
```

| 규칙 | 내용 |
| --- | --- |
| 채널 | UI→코어는 bounded 채널, 코어→UI 는 채널 + `request_repaint()` |
| 병합 | `Seek`, `SetVideoOptions` 처럼 슬라이더에서 연달아 나오는 명령은 마지막 값만 남긴다 |
| 유실 금지 | `Open`, `Close`, `Error`, `Ended` 는 병합하지 않는다 |
| 늦은 응답 | 이벤트에 `SessionId` 와 세대 번호(generation)를 붙인다. 닫힌 탭이나 다시 연 파일에 늦게 도착한 응답은 버린다 |
| 상태 원천 | 실제 값은 mpv 속성 관찰(`observe_property`)로 받는다. UI 가 보낸 값을 그대로 "적용됨"으로 표시하지 않는다 |

`hwdec-current` 는 실제 사용 중인 디코딩을 알려 주고, `no` 면 소프트웨어 디코딩이다 ([input.rst v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst), [매뉴얼 Property List](https://mpv.io/manual/stable/#property-list)).

## 4. 수명과 종료 순서

1. 탭 닫기 요청 → 세션 제어 워커에 `Close`
2. UI 스레드: 다음 프레임부터 그 세션의 렌더를 멈춘다
3. UI 스레드(GL current 상태): render context 해제
4. 제어 워커: mpv core 종료(`mpv_destroy` 계열)
5. UI: 탭 표면 정리

render context 는 mpv core 보다 **먼저** 해제한다. `render.h` 는 `mpv_render_context_free()` 가 core 에 비디오 체인을 끝내라고 요청하고 기다릴 수 있다고 적는다 ([render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)). 그래서 3번 단계는 제어 워커와 락을 공유하지 않게 한다.

## 5. 탭마다 core 하나

mpv render API 는 core 하나에 render context 하나다. 여러 파일을 동시에 재생하려면 `mpv_create()` 로 core 를 따로 만든다 ([render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)). 숨김 탭 정책은 [05-explorer-tabs.md](05-explorer-tabs.md) 에 있다.

## 6. 세션 소스: 로컬 파일과 URL (R3)

R3(유튜브 링크 재생)의 상세는 묶음 A·B 공통 문서 [../rust-video-player-youtube.md](../rust-video-player-youtube.md) 에 있다. 이 절은 코어에서 달라지는 점만 적는다.

```rust
pub enum MediaSource {
    Path(PathBuf),
    Url(Url),   // url 크레이트. 원본 watch URL 만 담는다
}
```

| 규칙 | 내용 |
| --- | --- |
| 저장 | `Url` 에는 사용자가 클릭한 **원본 watch URL** 만 저장한다. yt-dlp 가 푼 signed 스트림 URL·쿠키는 저장하지도, 로그에 남기지도 않는다 (sol 조사 채택, 공통 문서) |
| 화질 변경 | 같은 원본 URL 을 `ytdl-format` 을 바꿔 **다시 load** 한다 (sol 조사 채택) |
| 재생 경로 | `loadfile <URL>` → mpv 내장 `ytdl_hook` 이 yt-dlp 를 실행 → 로컬 파일과 같은 디코딩·렌더 경로. 그래서 R1·R2 가 그대로 적용된다 |

libmpv 초기 옵션 (core 를 만들고 `mpv_initialize` 전에 설정):

| 옵션 | 값 | 이유 |
| --- | --- | --- |
| `ytdl` | `yes` | 내장 ytdl hook 을 켠다 |
| `script-opts` | `ytdl_hook-ytdl_path=<yt-dlp.exe 절대 경로>` | PATH 검색에 기대지 않고 앱이 배포·갱신하는 yt-dlp 를 쓴다 |
| `ytdl-raw-options` | `ignore-config=` | mpv `config=no` 여도 yt-dlp 사용자 설정 파일은 막히지 않으므로 따로 끈다 (sol 조사 채택) |

옵션 설명은 [options.rst v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst) 의 `--ytdl`, `--ytdl-raw-options`, `--script-opts` 와 [매뉴얼 Program Behavior 절](https://mpv.io/manual/stable/#program-behavior) 에 있다.

`load-scripts=yes` 는 **설정하지 않는다.** `load-scripts` 의 기본값은 true 이고 libmpv 프로필에서 바뀌지 않는다. 내장 ytdl 스크립트는 사용자 스크립트와 별개로 `ytdl` 옵션에 따라 로드된다 ([options.c v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/options/options.c) `.auto_load_scripts = true`, [builtin.conf](https://github.com/mpv-player/mpv/blob/v0.41.0/etc/builtin.conf) `[libmpv]`, [scripting.c](https://github.com/mpv-player/mpv/blob/v0.41.0/player/scripting.c) `mp_load_builtin_scripts`; rvp-leader 판정 1). 이 hook 은 Lua 스크립트이므로 **Lua 를 넣어 빌드한 libmpv DLL** 이 필요하다 (확인 항목: [10-validation-and-open-questions.md](10-validation-and-open-questions.md) Q19).

---

이전: [03-rendering-pipeline.md](03-rendering-pipeline.md) · 다음: [05-explorer-tabs.md](05-explorer-tabs.md)
