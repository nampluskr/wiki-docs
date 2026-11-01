---
tags: [rust, video-player, windows, d3d11va, wasapi, libass, build, license]
created: 2026-11-01
updated: 2026-11-01
---

# 05 · Windows: GPU 가속·소리·자막·빌드·라이선스

> 이 파일을 읽는 때: R1(GPU 가속 ON/OFF)을 구현할 때, 소리·자막을 정할 때, DLL 을 구해 묶고 라이선스를 정할 때.

입구는 [README.md](README.md)다. 엔진 선택은 [02-DECODING.md](02-DECODING.md), 영상 옵션(R2)은 [06-FILTERS.md](06-FILTERS.md).

## 1. R1 — GPU 가속 ON/OFF

### 1.1 mpv 로 구현

| 항목 | 방법 | 출처 |
| --- | --- | --- |
| ON | `hwdec=auto-safe`(현재 문서에서 `auto` 와 동의어) 또는 `d3d11va` | https://mpv.io/manual/stable/#video |
| OFF | `hwdec=no` | 같음 |
| 재생 중 전환 | `hwdec` 는 런타임 속성이다. mpv 의 기본 키 바인딩도 `auto` ↔ `no` 를 토글한다 | mpv `options.rst` `--hwdec` 항 https://github.com/mpv-player/mpv/blob/master/DOCS/man/options.rst (sonnet 보고서) |
| 실패 시 | 하드웨어 디코딩이 불가능하면 소프트웨어로 대체한다고 문서에 있다 | 같음 |
| 실제 상태 | 속성 `hwdec-current` 를 읽어 UI 에 표시한다 | https://mpv.io/manual/stable/#property-list |
| 복사 방식 | `d3d11va` 는 같은 D3D11 에서 무복사, `d3d11va-copy` 는 RAM 으로 복사(copy-back) | 같음 |

주의 (보고서 간 차이는 보수적으로 채택):
- 전환이 **무중단·무음성 끊김이라고 보장하지 않는다.** argon 은 "끊김 없이"라고 했으나 sol 은 보장하지 않는다고 했다. https://mpv.io/manual/stable/#video
- "ON 요청"과 "실제 하드웨어 사용"은 다른 상태다. 항상 `hwdec-current` 로 확인한다.
- D3D11VA 무복사는 출력이 호환 D3D11(또는 ANGLE) 컨텍스트일 때 성립한다. 어떤 `vo` 설정에서 무복사가 되는지는 실기 검증 항목이다.

### 1.2 하드웨어 API 선택

| API | 위치 | 비고 | 출처 |
| --- | --- | --- | --- |
| D3D11VA | **우선** | 벤더 중립, Windows 내장. H.264·HEVC·VP9·AV1 (GPU별 지원 상이) | https://learn.microsoft.com/en-us/windows/win32/medfound/direct3d-11-video-apis |
| DXVA2 | 구형 폴백 | D3D9 기반. D3D11 과 직접 공유가 어렵다 | argon 보고서 |
| NVDEC / QSV | 선택지 | NVIDIA·Intel 전용. 기본 UI 에는 노출하지 않는다 | https://github.com/intel/libvpl |

GPU별 codec/profile/bit depth/해상도 지원을 실제로 probe 한다. GPU 가 있다고 HEVC 4:4:4 나 10-bit H.264 가 되는 것은 아니다. https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videodevice-getvideodecoderprofile . 제품 UI 는 API 목록보다 **ON/OFF + 실제 엔진 표시**를 권한다 (sol).

### 1.3 자동 전환 설계

설계 제안이다 (sol, sonnet). 실측하지 않았다.

| 상황 | 동작 |
| --- | --- |
| 초기화 실패, 지원 안 하는 코덱 | 소프트웨어로 재시작 |
| 디코드 오류 N회·연속 실패·시간 초과 | 위치 기억 → HW 끄고 재오픈 |
| device removed | presenter 까지 재생성 |
| 반복 실패 | 그 세션의 HW 재시도 횟수를 제한 |
| 모든 경우 | 재생 위치·일시정지·트랙·필터 보존, UI 에 "소프트웨어로 전환됨" 표시 |

드라이버 크래시까지 엔진이 복구한다고 보장하지 않는다. https://learn.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-devices-initialize

### 1.4 무복사의 정의와 R1+R2

- "무복사"는 decoder surface 가 CPU 읽기 없이 같은 GPU 출력 파이프라인으로 넘어가는 것이다. GPU 내부 blit 은 있어도 "CPU 무복사"라고 말할 수 있다. "copy 0회"와 혼용하지 않는다. https://github.com/mpv-player/mpv/blob/master/include/mpv/render_d3d11.h
- 하드웨어 디코딩 + CPU 필터는 VRAM↔RAM 왕복이 생겨 R1 이득이 줄어든다. [06-FILTERS.md](06-FILTERS.md) 3절.
- 동시 하드웨어 디코더 세션 수는 GPU 마다 다르다. **미확인**. 초과하면 소프트웨어로 전환한다 ([01-ARCHITECTURE.md](01-ARCHITECTURE.md) 4절).

### 1.5 다른 백엔드일 때 (직접 구현 경로)

| 백엔드 | ON | OFF | 재생 중 전환 | 출처 |
| --- | --- | --- | --- | --- |
| FFmpeg | `av_hwdevice_ctx_create` 로 D3D11VA 장치를 만들어 코덱 컨텍스트에 연결, `get_format` 에서 `AV_PIX_FMT_D3D11` 선택 | `hw_device_ctx` 없이 SW 코덱 재생성 | 위치 저장 → flush → 재오픈 → 목표 PTS 까지 디코딩 (탐색 재사용) | https://ffmpeg.org/doxygen/8.1/hw_decode_8c-example.html |
| GStreamer | `d3d11*dec` 요소 랭크 상향 | `avdec_*` 선택 | pipeline 재구성·seek 복구 설계 필요 | https://gstreamer.freedesktop.org/documentation/d3d11/index.html |
| Media Foundation | hardware transform 허용, D3D device manager 설정 | HW MFT 경로 비활성 | Source Reader/MFT 재구성 | https://learn.microsoft.com/en-us/windows/win32/medfound/mf-readwrite-enable-hardware-transforms |

FFmpeg 로 D3D11VA 프레임을 직접 다루면 `ID3D11Multithread::SetMultithreadProtected` 설정과, `ID3D11Texture2D` 배열 슬라이스를 큐에 너무 많이 쌓지 않는 것이 필요하다고 argon 이 지적했다(근거 문서 미첨부, **미확인**). 이 가이드의 libmpv 경로에서는 해당 없다.

## 2. 소리

### 2.1 mpv 를 쓰면

mpv 는 자체 WASAPI 출력(ao)을 쓴다. cpal/rodio 로 다시 꺼내 재생하지 않는다. 시계·버퍼·속도·장치 변경 책임을 한 엔진에 둔다. 장치 목록·선택은 `audio-device-list`·`audio-device`. https://mpv.io/manual/stable/#audio-output-drivers , https://mpv.io/manual/stable/#audio

### 2.2 FFmpeg 직접일 때만

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 출력 | `cpal` 0.18.2 (Windows 는 WASAPI 호스트) + PCM 링버퍼 | https://docs.rs/cpal/0.18.2/cpal/ |
| 콜백 | 안에서 디코딩·할당·파일 I/O·UI 잠금 금지. underrun 은 무음으로 채우고 상태 보고 | 같음 |
| 시계 | 소비한 샘플 수 ÷ 샘플레이트 + 시작 PTS − 출력 지연 | https://docs.rs/cpal/0.18.2/cpal/struct.StreamInstant.html |
| rodio | 고수준이라 PoC 에는 편하나 PTS·seek flush·장치 시계 정밀 조회가 약하다 (평가) | https://docs.rs/rodio/0.23.0/rodio/ |
| 리샘플링 | 장치 레이트·채널 배치로 변환. FFmpeg `libswresample` 이 안전. sample-rate 변환과 tempo/pitch 변경은 다른 기능 | https://ffmpeg.org/libswresample.html |
| 장치 변경 | 스트림 오류 콜백 → 스트림 재생성, 위치 유지. cpal 0.18.2 changelog 는 WASAPI 기본 장치 변경의 잘못된 DeviceChanged 신호를 고쳤다고 한다. 자동 reroute 를 맹신하지 않는다 | https://docs.rs/crate/cpal/0.18.2/source/CHANGELOG.md |

크레이트 버전은 [02-DECODING.md](02-DECODING.md) 5.2. `rubato` 5.0.1 은 argon 단독 값이고 sonnet 은 조회하지 않았다 (**미확인**).

## 3. 자막

### 3.1 mpv 로 구현 (권장)

| 기능 | 수단 | 출처 |
| --- | --- | --- |
| 외부 자막 추가 | `sub-add <url> [flags title lang]` | mpv `input.rst` https://github.com/mpv-player/mpv/blob/master/DOCS/man/input.rst (sonnet) |
| 트랙 선택 | `sid` / `aid` / `secondary-sid`. 목록은 `track-list` | https://mpv.io/manual/stable/#property-list |
| 지연·크기 | `sub-delay`, `sub-scale` | https://mpv.io/manual/stable/#subtitles |
| ASS 렌더 | mpv 내장 libass. SRT·ASS·내장 트랙 처리 | https://mpv.io/manual/stable/#subtitles , https://github.com/libass/libass |
| 합성 위치 | blur 등 필터가 자막까지 번지지 않도록 합성 순서를 정한다 | https://mpv.io/manual/stable/#subtitles |

실제 지원은 libass·FFmpeg 포함 빌드와 자막 유형에 달렸다 (sol).

### 3.2 직접 구현일 때

| 방법 | 내용 | 출처 |
| --- | --- | --- |
| SRT | 타임코드를 파싱해 PTS 와 비교하고 텍스트를 gpui 요소나 HTML 오버레이로 그린다. 폰트·한글은 UI 가 처리한다(쉬움, 추정) | sonnet·argon |
| ASS | 위치·스타일·karaoke·font fallback·MKV 첨부 폰트까지 필요해서 libass 가 실용적 | https://github.com/libass/libass/blob/master/libass/ass.h |
| FFmpeg 필터 | `subtitles`·`ass` 필터는 libass 포함 빌드(`--enable-libass`)가 필요. 영상에 굽는 방식이라 자막 끄기는 그래프 교체 | https://github.com/FFmpeg/FFmpeg/blob/master/configure (sonnet) |
| Rust 크레이트 | `libass` 0.2.0 은 2020-11-21 이후 정지. 순수 Rust `ass-renderer` 0.1.2 는 argon 단독 후보, 품질 **미검증** | [02-DECODING.md](02-DECODING.md) 5.2 |
| WebView | WebVTT 로 ASS 를 재현하지 못한다. libass bitmap 을 네이티브 영상과 합성하거나 검증된 ASS renderer 를 넣는다 | https://html.spec.whatwg.org/multipage/media.html#text-tracks |

한글 SRT 는 CP949/UTF-8-BOM 이 흔해서 인코딩 감지(`chardetng`, `encoding_rs`)가 필요하다는 것이 sonnet 의 일반 지식이다 (**미조회, 미확인**).

## 4. 빌드·DLL·번들·크기

### 4.1 기본 규칙

| 규칙 | 내용 | 출처 |
| --- | --- | --- |
| 툴체인 | `stable-x86_64-pc-windows-msvc`. **MSVC 와 MinGW 산출물을 섞지 않는다** (GStreamer MSI, import lib 포함) | https://gstreamer.freedesktop.org/download/ , argon·sonnet·sol 공통 |
| import library | MSVC 링커에는 `.lib` 가 필요하다. MinGW `.dll.a` 는 같은 것이 아니다 | https://mpv.io/installation/ |
| 버전 한 세트 | DLL·헤더·import lib·바인딩 버전을 함께 고정한다. "DLL만 있으면 cargo 가 연결"은 틀리다 | https://github.com/zmwangx/rust-ffmpeg/wiki/Notes-on-building |
| 이 가이드의 실행 | 조사만 했다. 설치·빌드는 하지 않았다 ([README.md](README.md)) | — |

### 4.2 libmpv DLL

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 구하는 곳 | mpv 공식 installation 페이지가 shinchiro 등의 Windows 빌드를 안내한다. 포함 기능·libmpv 제공 여부·GPL/LGPL·DLL 의존성·release/hash 를 확인한다 | https://mpv.io/installation/ |
| 예 | shinchiro `mpv-dev-x86_64-*.7z` 안의 `libmpv-2.dll` + 링크용 `.lib` | https://github.com/shinchiro/mpv-winbuild-cmake/releases (argon) |
| 사례 | Cathode 는 생성한 `mpv.lib` 와 libmpv·ANGLE DLL 을 번들한다 | https://github.com/kaiserbh/cathode#building-from-source |

### 4.3 FFmpeg DLL (직접 경로일 때만)

| 항목 | 내용 | 출처 |
| --- | --- | --- |
| 공식 | 소스만 제공. Windows 빌드는 제3자 | https://ffmpeg.org/download.html#build-windows |
| 제3자 | BtbN FFmpeg-Builds, vcpkg 등. LGPL/GPL 변종이 있다 | https://github.com/BtbN/FFmpeg-Builds , https://github.com/larksuite/rsmpeg/blob/master/doc/windows.md |
| 환경 변수 | `FFMPEG_DIR`, `LIBCLANG_PATH`(bindgen). argon 의 값이며 sonnet 은 "각 README 확인 필요"라고 했다 | https://github.com/zmwangx/rust-ffmpeg/wiki/Notes-on-building |

### 4.4 GStreamer (대안일 때만)

공식 MSVC x64 runtime/devel 패키지를 Rust target 과 맞추고 해당 배포의 pkg-config 경로를 쓴다. plugin DLL·plugin scanner·종속 DLL 을 포함하고 필요한 codec plugin 집합을 명시한다. https://gstreamer.freedesktop.org/documentation/deploying/windows.html , https://github.com/GStreamer/gstreamer-rs#installation . native 최신 stable 은 확인 시 1.28.8, 릴리스 날짜는 **미확인** (sol).

### 4.5 앱에 묶기

| 앱 | 방법 | 출처 |
| --- | --- | --- |
| gpui | `exe` 옆(또는 하위 `lib/`)에 DLL 을 둔다. Windows 의 DLL 검색에서 가장 먼저 찾는 위치다 | argon·sonnet 보고서 |
| Tauri | DLL 은 `bundle.resources`, 실행형은 `externalBin`(sidecar). 로딩 위치를 명시한다 | https://v2.tauri.app/develop/resources/ , https://v2.tauri.app/develop/sidecar/ |
| 설치 | WebView2 Evergreen 의존과 Fixed Runtime 동봉은 크기·버전 정책이 다르다 | https://v2.tauri.app/distribute/windows-installer/ |
| 검증 | 깨끗한 Windows 에서 DLL 검색 경로·누락 의존성·서명·업데이트를 확인 | sol 보고서 |

### 4.6 크기

일반 수치를 보장할 수 없다. **빌드마다 다르다** (comparison.md 판정 10). 아래는 보고서 값이다.

| 대상 | 값 | 성격 | 출처 |
| --- | --- | --- | --- |
| libmpv DLL (Cathode 사례) | 약 112 MB, vendor 다운로드 약 47 MB | 특정 artifact 의 예 | https://github.com/kaiserbh/cathode#building-from-source |
| libmpv DLL (argon) | 65~85 MB, 설치본 압축 시 25~30 MB | **추정** | argon 보고서 |
| FFmpeg shared | 35~55 MB (argon), 수십~100 MB (sonnet) | **추정** | 각 보고서 |
| GStreamer | 80~150 MB 이상 (argon), 수백 MB (sonnet) | **추정**, 두 값이 다르다 | 각 보고서 |

## 5. 라이선스

### 5.1 이 가이드의 결정

**GPL 허용** (사용자 결정, 2026-11-01). 일반 배포 libmpv(GPL 빌드가 많다)와 GPL 필터(`eq`, `hqdn3d`)를 그대로 쓸 수 있다.

| 비교 | 내용 |
| --- | --- |
| workbench-gpui D-2 | Zed GPL **코드**를 복사·의존하지 않고 Apache-2.0 의 gpui·gpui-kit 만 의존한다. `D:\projects\workbench-gpui\specs\v0.1\DECISIONS.md` |
| 이 가이드 | 별도 앱이라 D-2 를 물려받지 않는다. 영상 의존에 GPL 을 허용한다 |
| 그대로인 것 | Zed 앱 코드와 mpv/FFmpeg 의 GPL·LGPL **코드 원문은 복사하지 않는다.** 라이브러리로 링크한다 |

GPL 허용이어도 **앱을 배포할 때의 조건**(소스 제공 등)은 남는다. 같은 앱에 GPL libmpv 를 동적 링크로 묶으면 앱 전체의 배포 조건을 검토해야 한다 (추정, 법률 검토 필요, sonnet). 이 문서는 법률 자문이 아니다.

### 5.2 LGPL 로 바꿀 때 (짧게)

| 영역 | 방법 | 출처 |
| --- | --- | --- |
| libmpv | LGPL 구성 빌드를 쓴다. mpv 는 기본 전체가 GPLv2+ 이고 GPL-only 파일을 제외한 LGPL 구성이 가능하다. `-Dgpl=false` 만으로 허가가 자동 완성되지는 않으므로 DLL·FFmpeg·libass 전체 closure 의 조건을 따로 기록한다 | https://github.com/mpv-player/mpv/blob/master/Copyright |
| FFmpeg | `--enable-gpl` 없는 빌드. `eq`·`hqdn3d` 등 GPL 필터는 빠진다 | https://ffmpeg.org/legal.html |
| 필터 | 대체 수단은 [06-FILTERS.md](06-FILTERS.md) 4절 | — |
| 동적 링크 | LGPL 도 고지·교체 가능한 라이브러리·소스 제공 조건을 검토한다 | https://ffmpeg.org/legal.html |

LGPL 빌드 옵션은 meson 의 `-Dgpl=false` 다. mpv `meson.options` 에 `option('gpl', type: 'boolean', value: true, ...)` 가 있다(leader 확인, 2026-11-01). https://github.com/mpv-player/mpv/blob/master/meson.options . argon 이 적은 `--enable-lgpl` 은 mpv 옵션이 아니다.

### 5.3 구성요소별 표

| 구성요소 | 라이선스 | 비고 | 출처 |
| --- | --- | --- | --- |
| `libmpv2` 바인딩 | LGPL-2.1 | 바인딩 자체만. 네이티브 의존 라이선스를 대신하지 않는다 | https://crates.io/api/v1/crates/libmpv2 |
| libmpv DLL | **빌드마다 다름** (GPLv2+ 많음) | 배포 빌드의 라이선스를 확인 | https://mpv.io/installation/ |
| FFmpeg | 기본 LGPL-2.1+. `--enable-gpl` 이면 GPL. `--enable-nonfree` 는 재배포 불가 구성 | decode 만 쓴다고 LGPL 이 자동 보장되지 않는다 | https://ffmpeg.org/legal.html |
| GStreamer | core LGPL 계열, plugin 별 상이 | `bad`/`ugly` 라는 분류명이 곧 GPL 은 아니다 | https://gstreamer.freedesktop.org/documentation/frequently-asked-questions/licensing.html |
| libass | ISC | 폰트·shaping 의존성과 묶는 폰트 자체의 권리도 확인 | https://github.com/libass/libass/blob/master/COPYING |
| 특허 코덱 | 오픈소스 라이선스와 **별개** | H.264·HEVC·AAC 등. OS 디코더 사용이나 외부 exe 분리만으로 사라진다고 단정하지 않는다. 판매 국가별 판단은 **미확인** | https://ffmpeg.org/legal.html |

---

이전: [04-TAURI.md](04-TAURI.md) · 다음: [06-FILTERS.md](06-FILTERS.md)
