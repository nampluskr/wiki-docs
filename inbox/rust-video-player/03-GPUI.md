---
tags: [rust, video-player, gpui, gpui-kit, windows, hwnd]
created: 2026-11-01
updated: 2026-11-01
---

# 03 · gpui 앱에 붙이기

> 이 파일을 읽는 때: workbench-gpui 위에 동영상 탭을 만들 때. 화면 출력 방식(RenderImage·자식 HWND·D3D11)을 고를 때.

입구는 [README.md](README.md)다. 세션·탭 정책은 [01-ARCHITECTURE.md](01-ARCHITECTURE.md), 엔진은 [02-DECODING.md](02-DECODING.md).

## 1. workbench-gpui 현황

조사자가 로컬 소스를 읽은 결과다 (읽기만 함).

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 의존 | `gpui-kit = "=0.8.1"`. `Cargo.lock` 은 `gpui-pre 0.3.10` | `D:\projects\workbench-gpui\Cargo.toml`, `Cargo.lock` |
| 활성 아이템 | `Workbench` 구조체가 `editor_poc: Option<Entity<EditorPoc>>` **하나**만 갖고, `body()` 에서 `AnyElement` 로 바꿔 `active-item` 영역에 넣는다 | `src/ui/mod.rs` |
| 탭 바 | `layout.rs` 의 `tab_bar()` 는 고정 탭 `untitled (placeholder)` 와 정적 버튼. 프로젝트 패널은 자리 표시 | `src/ui/layout.rs` |
| 없는 것 | Pane/Item 추상, 파일 열기 라우터, 탭 목록 모델 | 같음 |
| 결정 | D-2 Zed GPL 코드는 복사·의존하지 않고 Apache-2.0 의 gpui·gpui-kit 만 의존. D-3 gpui-kit 이 진입점. D-5 gpui-kit 의존을 얇은 래퍼 계층에 가둠. D-7 상태는 Entity + `cx.notify()` | `D:\projects\workbench-gpui\specs\v0.1\DECISIONS.md` |

이 가이드의 앱은 **별도 앱**이다. 베이스의 D-2 를 그대로 물려받지 않는다. GPL 허용 결정과 D-2 의 차이는 [05-WINDOWS.md](05-WINDOWS.md) 5절. D-3·D-5·D-7 은 그대로 따른다. 공식 `gpui`(0.2.2)를 따로 추가하지 않는다. 타입이 섞인다. https://crates.io/crates/gpui-kit/0.8.1

## 2. `PaneItem` 설계

새로 만든다. 기존 API 인용이 아니라 설계다. 근거 지점은 위 `src/ui/mod.rs` 의 `body()`.

| 단계 | 내용 |
| --- | --- |
| 1 | 탭을 `Pane { tabs, active_id }` 로 일반화한다. 탭 공통 메타: 경로, 표시명, dirty 여부, close 정책 |
| 2 | 항목 종류를 열거형으로 둔다: 텍스트 편집기, 동영상 뷰어 |
| 3 | 프로젝트 트리의 열기 이벤트에서 **확장자 + probe** 로 분기한다. 동영상은 텍스트 읽기를 거치지 않는다 |
| 4 | `body()` 의 활성 아이템 선택부를 열거형 분기로 바꾼다 |

설계 예시:

```rust
enum PaneItem {
    Text(Entity<EditorPoc>),
    Video(Entity<VideoViewer>),   // VideoViewer 가 PlayerSession 을 보여 준다
}
// body(): match active_item { Text(e) => e.into_any(), Video(v) => v.into_any() }
```

probe 는 FFmpeg stream info 등으로 한다. https://ffmpeg.org/doxygen/trunk/group__lavf__decoding.html . 확장자만 믿지 않는다 (sol 보고서 Q4).

**유튜브 링크 (R3, 2026-11-01 추가)**: 텍스트 편집기·마크다운 미리보기에서 링크 클릭을 받아 같은 `Video` 항목을 `Source::Url` 로 연다. 링크 판별은 `url` 크레이트로 host 를 정확히 비교한다(`youtube.com`·하위 도메인·`youtu.be`). 미리보기의 링크 클릭 처리가 아직 없다면 그것이 이 기능의 가장 큰 작업이다. 상세는 [../rust-video-player-youtube.md](../rust-video-player-youtube.md).

## 3. 화면 출력 3단계

### 3.0 gpui 가 줄 수 있는 것

| 사실 | 출처 |
| --- | --- |
| Windows 렌더러는 Direct3D 11. 스프라이트 아틀라스는 `B8G8R8A8_UNORM` | gpui-pre 0.3.10 소스 `directx_atlas.rs:185`, `directx_devices.rs` (sonnet 보고서, 로컬 레지스트리 소스 확인) |
| `Window::paint_surface` 는 **macOS/iOS 전용**(CVPixelBuffer). Windows 의 `draw_surfaces` 는 빈 함수 | 같은 소스 `window.rs:5170-5176`, `directx_renderer.rs:816-821`. https://docs.rs/gpui-pre/0.3.10/gpui/struct.Window.html |
| 공개 경로는 `RenderImage`(BGRA CPU 버퍼) → `paint_image` → 아틀라스 `UpdateSubresource` 업로드뿐 | `directx_atlas.rs:279-304`. https://docs.rs/gpui-pre/0.3.10/gpui/struct.RenderImage.html |
| 새 `RenderImage` 마다 새 `ImageId`. 오래된 것은 `Window::drop_image` 로 해제해야 한다 | `window.rs:5193`. https://docs.rs/gpui-pre/0.3.10/gpui/struct.Window.html#method.drop_image |
| 사용자 `Element::paint` 가 임의 D3D 명령 주입이나 외부 texture import 를 허용하지는 않는다 | https://docs.rs/gpui-pre/0.3.10/gpui/trait.Element.html |

upstream main 에는 `crates/gpui_windows/src/directx_renderer.rs` 로 이동해 있고, 0.3.10 과 내용이 같다고 단정하지 않는다 (sol). **미확인**: upstream master 에서 Windows `paint_surface` 가 추가됐는지 (sonnet).

### 3.1 1단계 — `RenderImage` PoC

| 항목 | 내용 |
| --- | --- |
| 방식 | 디코딩 → CPU 에서 BGRA 변환(swscale) → `Arc<RenderImage>` → `img`/`paint_image` |
| 갱신 | 디코딩은 백그라운드. 가장 최근 프레임 슬롯에 넣고 UI 엔티티에서 `cx.notify()`. 재생 중에는 `window.request_animation_frame()` 로 예약할 수 있으나 60 Hz 를 영상 시계로 대신하지 않는다. 일시정지·숨김에서는 예약을 멈춘다 |
| 수명 | 현재·직전 프레임만 보관하고 더 오래된 것을 `drop_image`. Zed 의 원격 영상 뷰가 이렇게 한다(동작 이해용. **Zed 앱 코드는 복사하지 않는다**) |
| 한계 | CPU 복사 + 변환. GPU 디코딩을 켜도 최종 CPU 회수가 있으면 무복사가 아니다 |

출처: https://docs.rs/gpui-pre/0.3.10/gpui/struct.Window.html#method.request_animation_frame , https://github.com/zed-industries/zed/blob/14628482a3007b014043b73507a5a1437e13c2b2/crates/livekit_client/src/remote_video_track_view.rs (sol 보고서, 동작 참고용).

대역폭 (모두 **산술값**, alignment·staging 제외):

| 해상도 | 형식 | 초당 |
| --- | --- | --- |
| 1080p60 | BGRA | 약 0.50 GB/s (497.7 MB/s). 가능할 것으로 **추정** |
| 4K60 | BGRA | 약 1.99 GB/s (1.85 GiB/s). 병목 후보 |
| 4K60 | NV12 8-bit | 약 0.75 GB/s |

`RenderImage` 를 프레임마다 새로 만들고 `drop_image` 를 반복하는 방식의 안정성·성능은 **실측하지 않았다** (sonnet, sol). 1080p 까지 PoC 로 쓰고 4K60 주력 경로로 채택하기 전에 측정한다. YUV→RGB 는 성능 경로에서 GPU(셰이더 또는 D3D11 video processor)가 해야 하지만, 이는 gpui 가 외부 텍스처를 받아야 가능하다 (3.3).

### 3.2 2단계 — 탭 전용 자식 HWND (실용안)

mpv 의 `wid` 에 **탭 영상 영역 전용 자식 HWND** 를 준다. gpui 최상위 창 HWND 를 `wid` 로 주지 않는다. mpv 가 부모 전체를 채운다. https://mpv.io/manual/stable/#window

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 좌표 | layout 이후 영상 영역 bounds 에 창 DPI 배율을 곱해 **물리 픽셀** 로 `SetWindowPos`. DPI 변경·분할·사이드바 드래그 때 갱신 | https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowpos |
| 숨김 | 숨는 탭은 `ShowWindow(SW_HIDE)`, 보이면 SHOW | sonnet·sol |
| 겹침 | 자식 HWND 는 gpui 가 그린 버튼·메뉴·툴팁 위에 덮인다(airspace). **컨트롤은 영상 밖(아래)** 에 둔다 | https://learn.microsoft.com/en-us/windows/win32/winmsg/window-features#child-windows |
| 전체 화면 | 호스트 창 전체 화면 + 원래 pane layout 복원 | [01-ARCHITECTURE.md](01-ARCHITECTURE.md) 4절 |
| 얻기 | gpui 가 창 HWND 를 어떻게 노출하는지(`raw-window-handle`)는 **미확인** | sonnet 보고서 Q4·Q17 |

장점: `vo=gpu-next`, D3D11VA 무복사, R1·R2 를 mpv 가 그대로 처리한다. 4K60·HDR 에서 CPU 복사가 없다. 단점은 위 겹침·포커스·분할마다 창 하나를 만들어 위치를 맞추는 일이다.

### 3.3 3단계 — D3D11 render API (장래)

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| mpv | D3D11 render API 는 master 에만 있다 (커밋 2026-10-15). 정식 v0.41.0 에는 없다 | https://github.com/mpv-player/mpv/commit/4255130de410e0cbdab694b951085409d466b704 |
| 계약 | 앱이 준 **같은 device** 의 render-target texture 에 출력한다. 형식·usage 제한, context 상태 복원, 다른 device 동기화가 필요 | https://github.com/mpv-player/mpv/blob/master/include/mpv/render_d3d11.h |
| 바인딩 | `libmpv2` 7.0.0 은 render enum 이 OpenGL 만 | https://github.com/kohsine/libmpv-rs/blob/master/src/mpv/render.rs |
| gpui | gpui D3D11 renderer 에 **외부 texture 합성 경로를 직접 추가**해야 한다. 외부 텍스처를 받는 공개 API 는 조사에서 찾지 못했다(미확인) | https://github.com/zed-industries/zed/blob/main/crates/gpui_windows/src/directx_renderer.rs |

판단: master 기능 수용, FFI 추가, gpui renderer 변경을 모두 받아들일 때만 검토한다 (sol의 "통합 개발 제안"). 지금은 하지 않는다.

### 3.4 단계 비교

| 단계 | 4K60 | 겹침 | 구현량 | R1·R2 |
| --- | --- | --- | --- | --- |
| ① RenderImage | 병목(추정) | 없음. gpui 안 | 중 | 직접 구현 필요 |
| ② 자식 HWND | 가능(mpv 무복사) | 있음. 컨트롤을 영상 밖에 | 작음 | mpv 위임 |
| ③ D3D11 render API | 가능(이론) | 없음 | 큼 | mpv 위임 |

## 4. 컨트롤 UI

gpui-kit 의 버튼·슬라이더, 또는 직접 `div`·팝오버로 만든다. gpui-kit 은 gpui 버전이 같아야 한다. https://github.com/longbridge/gpui-kit

| 컨트롤 | 내용 |
| --- | --- |
| 재생·일시정지, 탐색 바 | 탐색 바 드래그는 빠른 탐색, 놓을 때 정확 탐색 ([01-ARCHITECTURE.md](01-ARCHITECTURE.md) 3.4) |
| 속도, 음량 | |
| 오디오·자막 트랙 | `track-list` 를 UI 로 ([05-WINDOWS.md](05-WINDOWS.md) 3절) |
| 영상 옵션 패널 | R2 슬라이더 ([06-FILTERS.md](06-FILTERS.md)) |
| R1 토글 | 요청값 + 실제 엔진 표시 |
| 전체 화면 | |

규칙: 컨트롤은 **영상 영역 밖**. 재생 위치 텍스트는 낮은 빈도로 갱신하고 영상은 PTS 기준으로 갱신한다. 키보드 포커스와 접근성을 포함한다. 출처: sol 보고서 Q4 (설계 제안). https://github.com/cijiugechu/gpui-video-player/tree/main/examples

## 5. 사례

| 사례 | 구성 | 별 | 최근 | 참고점·주의 |
| --- | --- | --- | --- | --- |
| gpui-video-player https://github.com/cijiugechu/gpui-video-player | GStreamer + gpui | **97** (comparison.md 판정 7. argon 의 ~45 는 틀렸다) | 0.1.3, 2026-10-23 | Element·컨트롤·프레임 수명. macOS 는 CVPixelBuffer, **Windows/Linux 는 sprite atlas 소프트 경로**. 예제가 `gpui-pre =0.3.9` 고정이라 로컬 0.3.10 과 호환 확인 필요 |
| alhazen-gpui / alhazen-core https://github.com/alhazen-rs/alhazen-core | MF·Rust 코덱·ffmpeg CLI + gpui | 0 / 2 | 0.6.0, 2026-10-10 | 오디오 시계·fallback·generation 참고. **검증 필요**: README 의 "Windows AC3 설치 불필요" 류 표현은 24H2 자료와 충돌 가능. R1 완전 OFF·ASS·R2 전체 **미확인** |
| syncplay-gpui https://github.com/everpcpc/syncplay-gpui | gpui + 외부 플레이어 연동 | 미확인 | — | gpui 상태 모델과 미디어 상태 동기화 구조. argon 보고서 단독, 별 수 미확인 |
| Zed 화면 공유 뷰 | gpui `img` + 백그라운드 프레임 갱신 + `notify` | — | 2026-01-16 커밋 | 영상 파일 플레이어가 아니라 프레임 표시·수명 사례. 동작 이해용, 코드 복사 금지 |

Windows 에서 무복사로 영상을 gpui 에 올린 제품 사례는 조사에서 찾지 못했다 (**미확인**, sol).

## 6. 주의점

| 주의 | 내용 |
| --- | --- |
| 버전 고정 | `gpui-kit`·`gpui-pre`·`libmpv2` 를 `=` 로 고정. 업그레이드 시 같이 검증 |
| 타입 혼용 | 공식 `gpui` 0.2.2 와 `gpui-pre` 를 섞지 않는다 |
| 상태 위치 | 재생 상태는 `Entity<VideoViewer>` 가 아니라 세션에 ([01-ARCHITECTURE.md](01-ARCHITECTURE.md) 2절) |
| 수명 | 닫을 때 `drop_image`, 자식 창, mpv core 를 순서대로 해제 |
| 라이선스 | Zed 앱 코드·GPL 코드는 복사하지 않는다. 두 베이스 저장소의 AGENTS 가 Zed 앱 코드 복사 금지를 명시한다 (sol) |

---

이전: [02-DECODING.md](02-DECODING.md) · 다음: [04-TAURI.md](04-TAURI.md)
