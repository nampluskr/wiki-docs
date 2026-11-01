---
tags: [rust, video-player, libmpv, ffmpeg, gstreamer]
created: 2026-11-01
updated: 2026-11-01
---

# 02. 디코딩 엔진 선택

> 이 파일을 읽는 때: libmpv·FFmpeg·GStreamer 중 무엇을 엔진으로 쓸지, libmpv 를 어떻게 붙일지 정할 때.

## 1. 세 엔진 비교 (2026-11-01)

| 항목 | libmpv (`libmpv2` 7.0.0) | FFmpeg (`ffmpeg-next` 9.0.0) | GStreamer (`gstreamer` 0.25.4) |
| --- | --- | --- | --- |
| 성격 | 완성형 플레이어 엔진: 디먹스·디코드·A/V 동기·자막·렌더 | 저수준 코덱·포맷 라이브러리 | 파이프라인 프레임워크 (`playbin3`) |
| 앱이 만들 것 | 창·UI·명령 연결 | 렌더러·오디오 출력·동기·탐색·자막·R1 재초기화 | 파이프라인 구성·caps 협상·플러그인 배포 |
| Windows 표시 | render API(OpenGL/SW, 0.42 부터 D3D11) 또는 `wid` | D3D11VA 프레임을 받아 앱이 그린다 | `d3d11videosink` + 창 핸들, 또는 `appsink` |
| 자막 | libass 내장 (SRT/ASS) | 디코딩만. ASS 렌더러는 따로 붙인다 | 플러그인 |
| 출처 | [crates.io](https://crates.io/crates/libmpv2), [render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h) | [crates.io](https://crates.io/crates/ffmpeg-next), [D3D11VA hwcontext](https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h_source.html) | [crates.io](https://crates.io/crates/gstreamer), [playbin3](https://gstreamer.freedesktop.org/documentation/playback/playbin3.html), [d3d11videosink](https://gstreamer.freedesktop.org/documentation/d3d11/d3d11videosink.html) |

**선택: libmpv.** R1(`hwdec` 실행 중 변경·폴백), R2(속성·필터·셰이더), 자막·트랙·탐색을 엔진이 이미 제공한다. 세 보고서의 결론이 같다.

## 2. libmpv render API 의 성격

| 사실 | 출처 |
| --- | --- |
| render API 는 디코딩된 프레임을 앱에 넘겨주지 않는다. mpv 가 앱이 준 타깃(FBO 등)에 **그려 준다** | [render.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h) |
| render context 는 재생 시작 전(VO 생성 전)에 만들어야 한다 | 같은 문서 Overview |
| core 하나에 render context 는 최대 하나다 ("only at most 1 context can exists per mpv core") | 같은 문서 |
| 새 프레임 알림은 `mpv_render_context_set_update_callback()` 으로 받는다 | 같은 문서 |
| OpenGL 백엔드: `MPV_RENDER_PARAM_OPENGL_INIT_PARAMS` 로 만들고, `MPV_RENDER_PARAM_OPENGL_FBO` 로 그린다 | [render_gl.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h) |
| OpenGL 백엔드에서도 "hardware decoding can be enabled/disabled at any time by setting the hwdec property" | 같은 문서 |

프레임 자체를 받아 직접 처리하고 싶다면 libmpv 는 맞지 않는다. 그때는 FFmpeg 나 GStreamer 로 설계해야 한다 (sol 보고서 B4).

## 3. 통합 방식 세 가지

| 방식 | 내용 | 이 묶음의 판단 |
| --- | --- | --- |
| render API (OpenGL) | eframe glow 콜백에서 mpv 가 GL 타깃에 그린다. 영상과 UI 가 한 장면에 있다 | **1단계 채택** |
| render API (D3D11, 0.42) | 앱이 만든 `ID3D11Device`·`ID3D11Texture2D` 에 mpv 가 그린다 | 2단계 ([01](01-stack-decision.md)) |
| `wid` 자식 창 | mpv 가 자식 HWND 에 직접 그린다 | 쓰지 않는다. UI 를 영상 위에 그릴 수 없고, mpv 도 임베딩의 문제를 경고한다 ([render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h), [매뉴얼 Embedding](https://mpv.io/manual/stable/#embedding-into-other-programs-libmpv)) |
| CPU 업로드 (SW render API) | `MPV_RENDER_API_TYPE_SW` 로 메모리에 그려 텍스처로 올린다 | 쓰지 않는다. 문서가 "extremely simple (but slow)" 라고 적는다 ([render.h](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render.h)) |

## 4. libmpv2 7.0.0 이 덮는 범위

| 영역 | 상태 | 출처 |
| --- | --- | --- |
| 명령·속성·이벤트 | 래핑됨 | [docs.rs libmpv2](https://docs.rs/libmpv2/7.0.0/libmpv2/) |
| OpenGL render (`OpenGLInitParams`, `FBO`, `RenderContext`) | 래핑됨 | [docs.rs render](https://docs.rs/libmpv2/7.0.0/libmpv2/render/index.html) |
| D3D11 render | 없음 | 같은 문서 |
| `SKIP_RENDERING`·`ADVANCED_CONTROL` 등 세부 파라미터 | `RenderParam` 변형 목록은 확인하지 못함 | 미확인 |

버전 주의: sol·argon 보고서는 6.0.x 를 최신으로 적었다. 하지만 crates.io 에서는 7.0.0(2026-10-30)이 최신이다. 7.0.0 은 `libmpv2-sys ^4.0.1` 에 의존한다 ([crates.io API](https://crates.io/api/v1/crates/libmpv2)). 6.x → 7.0 사이에 바뀐 API 는 확인하지 않았다. 의존성을 잠그기 전에 changelog 를 확인한다.

## 5. FFmpeg 와 GStreamer 를 고르지 않은 이유

- **FFmpeg 직접**: R1 을 만들려면 디코더 다시 열기 → 패킷 비우기 → 키프레임 탐색 → PTS 재기준 → 오디오 클럭 보정을 앱이 해야 한다. ASS 자막도 libass 를 따로 붙여야 한다. 세 보고서가 모두 가장 큰 구현량으로 평가했다 ([FFmpeg D3D11VA](https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h_source.html)).
- **GStreamer**: 동적 디코더·필터 교체, caps·장치 협상, 플러그인 배포 비용이 든다. `d3d11videosink` 문서는 공유 텍스처 접근에 과거 경로 대신 `appsink` 를 권한다 ([d3d11videosink](https://gstreamer.freedesktop.org/documentation/d3d11/d3d11videosink.html), [appsink](https://gstreamer.freedesktop.org/documentation/app/appsink.html)).

두 엔진 모두 [01-stack-decision.md](01-stack-decision.md) 의 대안으로 남겨 둔다.

---

이전: [01-stack-decision.md](01-stack-decision.md) · 다음: [03-rendering-pipeline.md](03-rendering-pipeline.md)
