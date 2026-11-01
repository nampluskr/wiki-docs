---
tags: [rust, video-player, egui, explorer, tabs]
created: 2026-11-01
updated: 2026-11-01
---

# 05. 파일 탐색기 트리와 탭

> 이 파일을 읽는 때: 왼쪽 트리, 오른쪽 탭·분할, 탭 안 영상 표면, 숨김 탭 정책을 만들 때.

## 1. 크레이트와 버전 (2026-11-01 crates.io)

| 용도 | 크레이트 | 버전 | egui 요구 | 출처 |
| --- | --- | --- | --- | --- |
| 앱 셸 | eframe | 0.37.0 (2026-10-15) | — | [crates.io](https://crates.io/crates/eframe) |
| 탭·분할 (기본) | egui_tiles | 0.18.0 (2026-10-19) | `egui ^0.37.0` | [crates.io](https://crates.io/crates/egui_tiles), [docs.rs](https://docs.rs/egui_tiles/0.18.0/egui_tiles/) |
| 트리 | egui_ltreeview | 0.10.0 (2026-10-25) | `egui ^0.37` | [crates.io](https://crates.io/crates/egui_ltreeview), [docs.rs](https://docs.rs/egui_ltreeview/0.10.0/egui_ltreeview/) |
| 탭·도킹 (대안) | egui_dock | 0.21.1 (2026-08-06) | `egui ^0.36` | [crates.io](https://crates.io/crates/egui_dock) |
| 파일 대화상자 | rfd | 0.17.2 | — | [crates.io](https://crates.io/crates/rfd) |
| 폴더 감시 | notify | 8.2.1 | — | [crates.io](https://crates.io/crates/notify) |

egui 요구 버전은 crates.io 의존성 API(`/api/v1/crates/<이름>/<버전>/dependencies`)로 확인했다. **eframe 0.37 + egui_tiles 0.18 + egui_ltreeview 0.10** 은 모두 egui 0.37 을 요구하므로 함께 쓸 수 있다. egui_dock 0.21.1 은 egui 0.36 을 요구하므로, egui_dock 을 쓰려면 eframe 0.36.2 와 egui_ltreeview 0.9.0 으로 내려야 한다. cargo 로 빌드해 보지는 않았다.

## 2. 탐색기 트리

| 항목 | 설계 |
| --- | --- |
| 위젯 | egui_ltreeview `TreeView`. 디렉터리는 `dir(id, name)` … `close_dir()`, 파일은 `leaf(id, name)` 로 넣는다. 다중 선택, 화살표 키 탐색, 드래그 앤 드롭, 노드 컨텍스트 메뉴를 지원한다 ([docs.rs](https://docs.rs/egui_ltreeview/0.10.0/egui_ltreeview/)) |
| 노드 ID | 경로 문자열 대신 정수 ID 를 쓰고, 경로는 앱 모델(`ExplorerModel`)에 둔다. 이름 변경·정렬·비동기 열거 중에도 선택이 꼬이지 않는다 (설계 제안, sol 보고서 B5) |
| 열거 | 펼친 노드만 워커 스레드에서 비동기로 열거한다. 권한 오류·순환 링크·확장자 필터는 앱이 처리한다 |
| 감시 | 펼친 폴더만 `notify` 로 감시하고, 변경은 디바운스한 뒤 UI 에 스냅샷으로 넘긴다 ([notify](https://docs.rs/notify/latest/notify/)) |
| 열기 | 파일을 활성화하면 `Command::Open` 을 보내 새 `SessionId` 탭을 만든다. 같은 탭에서 열지, 새 탭에서 열지는 설정으로 둔다 |
| 대화상자 | 자막 파일 추가·폴더 열기는 `rfd` 로 한다. 내장 트리를 대신하는 것은 아니다 ([rfd](https://docs.rs/rfd/latest/rfd/)) |

## 3. 탭과 분할 (egui_tiles)

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 상태 | `Tree<Pane>` 하나에 모든 배치가 들어 있다. `Tiles` 저장소와 루트 `TileId` 로 이루어진다 | [docs.rs](https://docs.rs/egui_tiles/0.18.0/egui_tiles/) |
| 컨테이너 | `Tabs`(한 번에 하나만 보임), `Linear`(가로·세로 분할), `Grid` | 같은 문서 |
| 화면 | `Behavior` 트레이트의 `tab_title_for_pane`, `pane_ui` 를 구현한다 | 같은 문서 |
| 저장 | serde 기능을 켜면 트리 전체를 직렬화할 수 있다 → 탭 배치 복원 | 같은 문서 |

`Pane` 에는 `SessionId` 만 넣는다. 재생 상태는 `player-api` 쪽에 둔다([04-player-core.md](04-player-core.md)). 첫 버전에서는 탭을 별도 OS 창으로 떼어내지 않는다. GL 컨텍스트와 mpv render context 가 창마다 다르기 때문이다 (설계 판단, [render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)).

## 4. 탭 안 영상 표면

1. `pane_ui` 에서 탭 영역의 rect 를 얻는다.
2. 영상 종횡비를 지켜 letterbox 된 `video_rect` 를 계산하고, 물리 픽셀로 바꾼다(DPI 배율).
3. 그 rect 에 paint callback 을 등록한다. 콜백은 mpv 가 그린 FBO 를 그리거나, mpv 가 그 영역에 직접 그리게 한다 ([03-rendering-pipeline.md](03-rendering-pipeline.md)).
4. 콜백 다음에 같은 egui 장면에서 OSD(탐색 막대·재생 버튼·R1/R2 패널)를 그린다. 영상과 UI 가 한 GL 장면에 있으므로 영상 위에 반투명 UI 를 그릴 수 있다.
5. 로딩·오류·마지막 프레임 상태는 따로 표시한다.

자식 HWND(`wid`)에 영상을 그리는 방식은 쓰지 않는다. 부모 창의 UI 를 영상 위에 그릴 수 없고(airspace 문제, 세 보고서 공통), mpv 문서도 "window embedding can cause various issues, especially with GUI toolkits" 라며 render API 를 권한다 ([render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h), [매뉴얼 Embedding](https://mpv.io/manual/stable/#embedding-into-other-programs-libmpv)).

## 5. 숨김 탭과 동시 재생

| 상황 | 기본 정책 (설계 제안) | 근거 |
| --- | --- | --- |
| 탭이 가려짐 | 일시정지한다. 위치·트랙·필터는 보존한다 | 세 보고서 공통 |
| "백그라운드 오디오" 설정 ON | 재생은 계속하고 렌더만 건너뛴다. mpv 의 `MPV_RENDER_PARAM_SKIP_RENDERING` 은 프레임을 그린 것으로 처리하고 타이밍도 똑같이 진행한다 | [render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h) |
| 분할 화면에서 여러 탭이 보임 | 탭마다 mpv core 를 따로 둔다. 오디오는 마지막으로 클릭한 탭 하나만 내고, 나머지는 음소거한다 | core 당 render context 1개 ([render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)) |
| 오래 숨겨진 탭 | 메모리 예산을 넘으면 세션을 닫고, 위치만 저장했다가 다시 보일 때 복원한다 | 설계 제안 |

숨김 탭을 렌더하지 않고 방치하면 안 된다. update 콜백이 오면 `mpv_render_context_update()` 를 처리하거나 skip rendering 으로 프레임을 소비해야 한다. 그러지 않으면 core 가 기다리며 막힐 수 있다 ([render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h) `ADVANCED_CONTROL` 설명).

### 5.1 하드웨어 디코더가 모자랄 때

탭을 여러 개 열면 하드웨어 디코더 생성이 실패할 수 있다. 고정된 "세션 수 상한"은 근거를 찾지 못했다. 원인은 VRAM 디코딩 표면 풀 부족이나 미지원 프로파일이다(`ID3D11VideoDevice::CreateVideoDecoder` 가 실패 HRESULT 를 반환) ([Microsoft](https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videodevice-createvideodecoder), argon followup-1). 이때는 R1 폴백으로 그 탭만 소프트웨어 디코딩이 된다 ([06-r1-hwdec.md](06-r1-hwdec.md)).

4K 8bit RGBA 한 장은 3840×2160×4 바이트 ≈ 31.6 MiB 다(산술). 디코더 표면 풀·필터 이력·탭 수에 따라 메모리가 곱해지므로, 탭 수만으로 예산을 잡지 말고 **보이는 표면 수**로 잡는다.

## 6. 유튜브 링크 클릭 → 새 영상 탭 (R3)

상세(yt-dlp 배포·갱신, 화질, 오류, 약관 사실)는 묶음 A·B 공통 문서 [../rust-video-player-youtube.md](../rust-video-player-youtube.md) 에 있다.

| 단계 | 내용 |
| --- | --- |
| 1. 클릭 | 링크를 클릭하면 URL 을 검증한다 |
| 2. 검증 | [`url`](https://docs.rs/url/latest/url/) 크레이트로 파싱하고, scheme 이 `https` 이고 `host_str()` 이 허용 목록(예: `www.youtube.com`, `youtube.com`, `m.youtube.com`, `youtu.be`)과 **정확히 같을 때만** 통과시킨다. 문자열 `contains` 로 비교하지 않는다 (`youtube.com.example.net` 같은 주소가 통과한다) (sol 조사 채택) |
| 3. 탭 | 통과하면 새 `SessionId` 탭을 만들고 `Command::Open { source: MediaSource::Url(..) }` 을 보낸다 ([04-player-core.md](04-player-core.md) 6절). 탭 표면·R1/R2 패널은 로컬 파일 탭과 같다 |
| 4. 그 밖 | 허용 목록 밖의 링크는 플레이어로 열지 않는다. 기본 브라우저로 넘길지는 설정으로 둔다 (설계 제안) |

**범위 공백**: 지금 묶음 B 의 UI 는 파일 탐색기 트리와 영상 탭뿐이다. **링크를 클릭할 텍스트·미리보기 화면은 아직 범위에 없다.** 최소 방안은 다음과 같다 (설계 제안).

| 방안 | 내용 |
| --- | --- |
| 텍스트·마크다운 미리보기 탭 | 트리에서 `.txt`·`.md` 를 열면 미리보기 탭을 띄운다. egui 의 링크 위젯(`Hyperlink`)은 클릭 때 `OutputCommand::OpenUrl` 을 내보낸다. 앱은 프레임 끝에서 이 명령을 가로채 2단계 검증으로 넘기고, 유튜브 링크면 브라우저로 넘기지 않는다 ([egui Hyperlink](https://docs.rs/egui/0.37.0/egui/widgets/struct.Hyperlink.html), [PlatformOutput](https://docs.rs/egui/0.37.0/egui/struct.PlatformOutput.html)). eframe 이 그 명령을 처리하기 전에 앱이 지울 수 있는지는 **미확인**이다. 막히면 링크를 `Hyperlink` 대신 버튼 모양 위젯으로 그려 클릭을 직접 받는다 |
| URL 붙여넣기 | "URL 열기" 입력 칸(또는 클립보드 붙여넣기)으로 같은 검증 경로를 탄다. 미리보기 탭 없이도 R3 를 시험할 수 있다 |

마크다운 렌더 크레이트를 쓰면 그 크레이트의 링크 hook 을 써야 한다. 어떤 크레이트를 쓸지는 정하지 않았다(egui 0.37 호환 확인 필요).

---

이전: [04-player-core.md](04-player-core.md) · 다음: [06-r1-hwdec.md](06-r1-hwdec.md)
