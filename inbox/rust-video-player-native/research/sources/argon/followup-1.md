# [rvpn-lead 재질의 1] 검증 및 정정 보고서 (확인일: 2026-11-01)

## (1) D3D11VA 동시 디코딩 세션 한계로 추가 탭을 SW로 내린다는 주장 (B5-3 / B7)

- **판정**: **"고정 세션 개수 제한(예: 8개)" 주장은 미확인(정정 대상)**이며, 실제 한계 원인은 **VRAM 디코딩 서피스 풀 고갈(`E_OUTOFMEMORY`)**이다.
- **근거 및 공식 출처**:
  1. **NVIDIA 공식 문서 (NVDEC Video Decoder API Programming Guide)**: NVIDIA GPU에서 동시 세션 개수가 인위적으로 제한(예: 3~8개)되는 것은 인코더(**NVENC**)이며, 디코더(**NVDEC / D3D11VA**)에는 드라이버가 강제하는 고정 동시 세션 개수 상한이 없다. 대신 동시 디코딩 세션 수는 **가용 VRAM 용량과 각 디코더 인스턴스가 예약하는 디코딩 서피스 풀(DPB + 여유 프레임) 메모리**에 의해 제한된다 (https://docs.nvidia.com/video-technologies/video-codec-sdk/12.2/nvdec-video-decoder-api-prog-guide/index.html, 2026-11-01 확인).
  2. **Microsoft 공식 문서 (`ID3D11VideoDevice::CreateVideoDecoder`)**: D3D11 비디오 디코더 생성 및 텍스처 배열(`D3D11_BIND_DECODER`) 할당 시 하드웨어 리소스나 VRAM이 부족하면 `E_OUTOFMEMORY` 등의 `HRESULT` 실패 코드를 반환한다 (https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videodevice-createvideodecoder, 2026-11-01 확인).
  3. **FFmpeg D3D11VA 구현**: FFmpeg `d3d11va`는 스트림마다 참조 프레임(DPB) + 스레드 수만큼의 `ID3D11Texture2D` 배열 슬라이스를 고정 할당하므로, 고해상도(4K) 탭을 다수 열면 고정 세션 제한이 아니라 VRAM 서피스 할당 실패가 발생할 수 있다 (https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8c.html, 2026-11-01 확인).
- **보고서 정정 내용**:
  - 기존 보고서 B5-3의 `"GPU의 D3D11VA 동시 디코딩 세션 한계(예: 4K 다중 스트림 8개 초과 시)"` 문구에서 **"8개 초과 시" 고정 세션 수 주장은 미확인으로 삭제**하고, **"다중 고해상도 탭 개방으로 인한 GPU VRAM 디코딩 서피스 풀 부족(`E_OUTOFMEMORY`) 또는 미지원 프로파일로 `CreateVideoDecoder`가 실패할 경우 R1 폴백 로직에 의해 SW 디코딩으로 자동 전환한다"**로 정정한다.

---

## (2) `ID3D11VideoContext::VideoProcessorSetStreamFilter`의 `NOISE_REDUCTION`·`EDGE_ENHANCEMENT` 필터 지원이 드라이버 의존인지 여부

- **판정**: **확인됨 (GPU 하드웨어 및 그래픽 드라이버 의존 맞음)**.
- **공식 Microsoft Learn 출처 URL (2026-11-01 확인)**:
  1. `ID3D11VideoContext::VideoProcessorSetStreamFilter`: https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videocontext-videoprocessorsetstreamfilter
  2. `D3D11_VIDEO_PROCESSOR_FILTER` 열거형: https://learn.microsoft.com/en-us/windows/win32/api/d3d11/ne-d3d11-d3d11_video_processor_filter
  3. `D3D11_VIDEO_PROCESSOR_FILTER_CAPS` 열거형: https://learn.microsoft.com/en-us/windows/win32/api/d3d11/ne-d3d11-d3d11_video_processor_filter_caps
  4. `ID3D11VideoProcessorEnumerator::GetVideoProcessorCaps`: https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videoprocessorenumerator-getvideoprocessorcaps
  5. `ID3D11VideoProcessorEnumerator::GetVideoProcessorFilterRange`: https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videoprocessorenumerator-getvideoprocessorfilterrange
- **상세 검증 내용**:
  - Microsoft 공식 문서에 따르면 `D3D11_VIDEO_PROCESSOR_FILTER_NOISE_REDUCTION`(`4`) 및 `D3D11_VIDEO_PROCESSOR_FILTER_EDGE_ENHANCEMENT`(`5`)는 모든 D3D11 디바이스가 의무적으로 지원하는 기능이 아니다.
  - 애플리케이션은 반드시 `ID3D11VideoProcessorEnumerator::GetVideoProcessorCaps`를 호출해 반환된 `D3D11_VIDEO_PROCESSOR_CAPS::FilterCaps` 비트필드에 `D3D11_VIDEO_PROCESSOR_FILTER_CAPS_NOISE_REDUCTION`(`0x10`) 및 `D3D11_VIDEO_PROCESSOR_FILTER_CAPS_EDGE_ENHANCEMENT`(`0x20`) 비트가 켜져 있는지 먼저 확인해야 한다.
  - 또한 지원되는 경우에도 `GetVideoProcessorFilterRange`를 호출해 해당 드라이버가 허용하는 `Minimum`, `Maximum`, `Default`, `Multiplier` 범위를 조회한 뒤 `VideoProcessorSetStreamFilter`의 `Level` 인자로 전달해야 한다.
  - 따라서 R2 요구사항을 모든 Windows 11 PC에서 일관되게 보장하려면 **`wgpu` WGSL 자체 후처리 셰이더(Bilateral Denoise, CAS/Unsharp Mask)를 기본 경로로 두고**, `FilterCaps`가 확인된 환경에서만 D3D11 비디오 프로세서 필터를 선택적으로 활용해야 한다.

---

## (3) 8개 크레이트의 crates.io 최신 버전 재확인 (2026-11-01 확인)

| 크레이트 | 기존 보고서 기재 버전 | 재확인 최신 버전 (2026-11-01) | 판정 및 정정 내역 | 공식 출처 URL |
| :--- | :---: | :---: | :--- | :--- |
| **`rfd`** | `0.17.2` | **`0.17.2`** | **일치 (확인됨)** | https://crates.io/crates/rfd |
| **`notify`** | `8.2.1` | **`8.2.1`** | **일치 (확인됨)** | https://crates.io/crates/notify |
| **`walkdir`** | `2.5.0` | **`2.5.0`** | **일치 (확인됨)** | https://crates.io/crates/walkdir |
| **`crossbeam-channel`** | `0.5.17` | **`0.5.17`** | **일치 (확인됨)** | https://crates.io/crates/crossbeam-channel |
| **`cpal`** | `0.16.0` | **`0.18.2`** | **정정 (`0.16.0` → `0.18.2`)** | https://crates.io/crates/cpal |
| **`dioxus`** | `0.7.9` | **`0.7.10`** (`0.7` 트랙 최신, 프리릴리스는 `0.8.0-alpha.1`) | **보완 (`0.7.9` → `0.7.10`)** | https://crates.io/crates/dioxus<br>https://docs.rs/dioxus/latest/dioxus/ |
| **`freya`** | `0.4.3` | **`0.4.3`** | **일치 (확인됨)** | https://crates.io/crates/freya |
| **`vizia`** | `0.4.0` | **`0.4.0`** | **일치 (확인됨)** | https://crates.io/crates/vizia |

---

## (4) `libmpv-2.dll` 크기 35~55MB의 실측 출처 유무

- **판정**: 비압축 `libmpv-2.dll` 크기가 `35~55MB`라는 기존 수치는 **공식 실측 출처 미확인(추정 오류)**이며, `.7z` 압축 아카이브 크기와 혼동된 수치다.
- **실측 근거 및 출처 (2026-11-01 확인)**:
  1. Windows용 `libmpv` 사실상 표준 프리빌드 배포소인 **`shinchiro/mpv-winbuild-cmake`** (https://github.com/shinchiro/mpv-winbuild-cmake/releases) 및 **`zhongfly/mpv-winbuild`** (https://github.com/zhongfly/mpv-winbuild/releases)의 `mpv-dev-x86_64-v3-*.7z` 패키지를 기준으로 확인하면:
     - `.7z` **압축 파일 크기**가 약 **35~45 MB** 수준이다.
     - 압축 해제 후 실제 **`libmpv-2.dll` 단일 바이너리(비압축) 크기**는 FFmpeg, `libplacebo`, `libass`, `dav1d`, `uchardet`, `luajit` 등이 모두 정적 링크되어 있어 **약 110~115 MB**이다.
  2. 만약 불필요한 인코더(`libx264`, `libx265`, `libsvtav1` 등)와 프로토콜을 제거하고 디코더만 남긴 최소 LGPL 커스텀 빌드를 직접 구성할 경우 비압축 DLL 크기를 줄일 수 있으나, 그 정확한 용량은 빌드 옵션에 좌우되므로 **미확인(추정)**으로 분류해야 한다.
- **보고서 정정 내용**:
  - 기존 보고서 B8의 `libmpv-2.dll ~35~55MB` 표기는 **`.7z` 압축 배포 크기(~35~45MB)와 비압축 실제 DLL 크기(~110~115MB, `shinchiro/mpv-winbuild-cmake` 풀 빌드 기준)를 구분하여 정정**한다. (설치 후 디스크 점유 기준 권장 1 스택의 전체 배포 크기는 약 **120~130 MB**, `.7z`/설치 프로그램 압축 배포 크기가 약 **45~55 MB**이다.)
