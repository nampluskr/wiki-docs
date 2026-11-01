# YouTube 재생 연구 보고서 (R3)

### 요약 (10줄)
1. mpv는 내장 Lua 스크립트 `ytdl_hook`과 외부 `yt-dlp` CLI를 연동하여 웹 스트리밍 URL을 파싱 및 재생한다.
2. libmpv 임베딩 환경은 CLI와 달리 `load-scripts=no`가 기본값이므로, URL 훅 구동을 위해 `load-scripts=yes` 설정이 필수다.
3. Windows용 libmpv 배포판(shinchiro, zhongfly의 mpv-winbuild-cmake)은 LuaJIT를 기본 탑재하여 `ytdl_hook`을 실행할 수 있다.
4. yt-dlp는 소스 Unlicense, 단일 바이너리는 GPLv3+ 라이선스이며, 유튜브 n-challenge 해결을 위해 Deno/Node.js 런타임을 요구한다.
5. DASH 분리 스트림(영상/음성)은 mpv 내부의 `edl://` 가상 프로토콜로 실시간 결합되어 디스크 병합 없이 스트리밍된다.
6. R1(GPU 하드웨어 가속)은 스트림 디먹싱 이후 로컬 파일과 동일하게 작동하며, 4K AV1은 최신 GPU 하드웨어 디코더가 요구된다.
7. R2(영상 옵션) 중 GPU 셰이더 및 색상 보정은 온전히 적용되나, 소프트웨어 `vf` 필터는 `hwdec` 복사 모드나 CPU 디코딩이 필요하다.
8. 봇 차단 및 연령 제한 오류는 `MPV_EVENT_END_FILE`과 로그 레벨 감시로 수신하며, 브라우저 쿠키 연동으로 완화 가능하다.
9. 대안인 Tauri WebView2 YouTube IFrame API는 약관 준수에 유리하나 R1·R2 커스텀 렌더링 파이프라인 제어가 원천 불가능하다.
10. 유튜브 서비스 약관상 비공식 스트림 추출은 기술적 제한과 법적 위험이 따르며, 개인 실행과 상용 배포 간 위험도 차이가 크다.

---

## Y1. mpv의 ytdl 연동

mpv v0.41.0(2025-12-21 릴리스 확인 기준, [mpv manual](https://mpv.io/manual/master/))의 YouTube 스트리밍 연동 메커니즘은 다음과 같다.

- `--ytdl=<yes|no>`: 외부 다운로더 훅 활성화 여부를 지정한다. 기본값은 `yes`이다.
- `ytdl_hook`: mpv 소스 트리(`player/lua/ytdl_hook.lua`)에 내장된 Lua 스크립트로, 입력 URL(http/https/ytdl 등)을 가로채 `yt-dlp`를 하위 프로세스로 실행하고 메타데이터와 스트림 주소를 추출한다.
- `script-opts=ytdl_hook-ytdl_path=<path>`: mpv가 호출할 yt-dlp 실행 파일 경로를 지정한다. 앱 번들 내 특정 위치의 `yt-dlp.exe`를 지정할 때 `script-opts-add=ytdl_hook-ytdl_path=C:/path/to/yt-dlp.exe` 형태로 설정한다.
- `--ytdl-format=<string>`: yt-dlp의 포맷 선택기(`-f`)로 전달되는 문자열이다. 기본값은 `bestvideo+bestaudio/best`이다.
- `--ytdl-raw-options=<key>=<value>[,...]`: yt-dlp의 명령줄 인수를 임의로 전달한다. 인자가 없는 플래그는 `key=` 형태로 빈 값을 지정해야 한다(예: `ytdl-raw-options=sub-lang="en,ko",write-sub=,no-playlist=`).

---

## Y2. libmpv에서의 동작

C API 라이브러리인 libmpv 임베딩 시 standalone CLI mpv와 다음과 같은 동작 차이가 존재한다 ([mpv C API 문서](file:///https://github.com/mpv-player/mpv/blob/master/libmpv/client.h)).

- **내장 Lua 스크립트 기본 로드 여부**: CLI mpv는 기본적으로 내장 스크립트와 설정 파일을 로드하지만, libmpv는 호스트 애플리케이션의 독립성을 위해 기본적으로 설정을 읽지 않는다.
- **필수 옵션과 기본값 차이**:
  - `config`: libmpv 기본값은 `no`(`mpv.conf` 미로드).
  - `load-scripts`: libmpv 기본값은 `no`. 따라서 `mpv_initialize()` 호출 전에 `mpv_set_option_string(ctx, "load-scripts", "yes")`를 명시적으로 설정해야 내장 `ytdl_hook.lua`가 동작한다.
  - `ytdl`: 기본값은 `yes`이나 `load-scripts`가 꺼져 있으면 스크립트 엔진 자체가 기동하지 않아 무용지물이 된다.
- **Lua 미포함 빌드 시 문제**: `ytdl_hook`은 순수 Lua 스크립트이므로, libmpv 컴파일 시 Lua 지원(`-Dlua=enabled`)이 제외된 빌드에서는 YouTube URL 파싱 기능이 완전히 동작하지 않는다.
- **Windows 빌드(shinchiro / zhongfly)의 Lua 포함 여부**:
  - Windows 배포 툴체인인 [mpv-winbuild-cmake](https://github.com/shinchiro/mpv-winbuild-cmake) 및 활성 빌드 저장소인 [zhongfly/mpv-winbuild](https://github.com/zhongfly/mpv-winbuild)(2026년 기준 매일 빌드)는 `luajit` 패키지를 기본 의존성으로 컴파일한다.
  - 배포되는 `mpv-dev-x86_64-*.7z`에 포함된 `libmpv-2.dll`은 LuaJIT 런타임이 포함되어 있어 `ytdl_hook` 실행이 지원된다.

---

## Y3. yt-dlp

- **최신 버전 및 릴리스 주기**: [yt-dlp GitHub](https://github.com/yt-dlp/yt-dlp) 기준 날짜 기반 버전 체계(`YYYY.MM.DD`)를 사용한다. 릴리스 채널은 Stable(월간 단위), Nightly(매일 빌드, 권장), Master(커밋 단위)로 구분된다.
- **라이선스**:
  - 소스 코드 및 PyPI 배포본: The Unlicense (퍼블릭 도메인).
  - PyInstaller로 패키징된 Windows 단일 실행 파일(`yt-dlp.exe`): 번들된 의존성으로 인해 **GPLv3+** 라이선스가 적용된다([yt-dlp LICENSE](https://github.com/yt-dlp/yt-dlp/blob/master/LICENSE)). 상용 앱 번들링 시 라이선스 고지 및 소스 제공 의무 검토가 필요하다.
- **Windows 실행 파일 배포 형태 및 번들링**:
  - GitHub Releases에서 단일 실행 파일 `yt-dlp.exe`를 다운로드하여 앱의 리소스 폴더(예: `assets/bin/yt-dlp.exe`)에 포함시킨다.
  - libmpv 초기화 시 `ytdl_hook-ytdl_path` 옵션에 해당 절대 경로를 지정한다.
- **갱신 방법**:
  - `yt-dlp -U` 또는 `yt-dlp --update-to nightly` 명령으로 자체 바이너리 업데이트가 가능하다.
  - 앱 내부에서는 백그라운드 프로세스로 `yt-dlp.exe -U`를 호출하거나, GitHub Releases API를 조회하여 최신 바이너리를 교체 다운로드하는 관리 로직을 구현한다.
- **빈번한 갱신 필요성**: YouTube의 플레이어 스크립트 난독화 변경, `n` 매개변수 암호화(n-challenge), 시그니처 디사이퍼 알고리즘 변경, 봇 탐지 정책 업데이트로 인해 갱신이 누락되면 다운로드 속도가 급감(50KB/s 스로틀링)하거나 HTTP 403 오류가 발생한다.
- **JS 런타임 등 최근 추가 의존성**:
  - YouTube의 복잡한 n-challenge 해독을 위해 내부 인터프리터 대신 **외부 JavaScript 런타임(EJS)**이 필수가 되었다.
  - 지원 런타임: **Deno 2.0+**(가장 권장), **Node.js 20+**, QuickJS.
  - Deno나 Node.js가 시스템 PATH에 없거나 번들되지 않은 경우, yt-dlp의 유튜브 재생 주소 해석이 실패하거나 극심한 속도 저하가 발생한다.

---

## Y4. 재생 설정

- **화질 선택 (format 문자열 예시)**:
  - 4K VP9: `--ytdl-format="bestvideo[height<=2160][vcodec^=vp09]+bestaudio/best"`
  - 4K AV1: `--ytdl-format="bestvideo[height<=2160][vcodec^=av01]+bestaudio/best"`
  - 1080p H.264: `--ytdl-format="bestvideo[height<=1080][vcodec^=avc1]+bestaudio/best"`
  - 기본 추천: `--ytdl-format="bestvideo[height<=?1080]+bestaudio/best"`
- **DASH 분리 스트림 병합 방식**:
  - YouTube는 고화질 비디오와 오디오를 별도 DASH 스트림으로 분리 전송한다.
  - mpv는 로컬에 임시 파일을 다운로드하여 FFmpeg로 합치지 않는다. `ytdl_hook`이 yt-dlp를 통해 비디오 URL과 오디오 URL을 각각 취득한 뒤, mpv의 가상 프로토콜인 **`edl://`** 스트림 디스크립터로 변환하여 mpv demuxer에 직접 전달한다.
  - mpv의 demuxer는 단일 세션 안에서 두 네트워크 스트림을 동시에 읽어 들여 실시간으로 비디오/오디오 싱크를 맞춘다.
- **자막 (`sub-langs` 등)**:
  - 자막 메타데이터 전달: `ytdl-raw-options=sub-lang="ko,en",write-sub=,write-auto-sub=`
  - 활성 자막 선택: mpv 옵션 `slang=ko,en`을 지정하여 파싱된 원격 자막 트랙을 자동 선택한다.
- **캐시 및 버퍼**:
  - `--cache=yes`: 네트워크 스트림 캐싱 활성화.
  - `--demuxer-max-bytes=150MiB`: 전방 캐시 메모리 크기 지정.
  - `--demuxer-readahead-secs=60`: 미리 읽을 스트림 시간(초).
  - `--demuxer-max-back-bytes=50MiB`: 되감기용 후방 버퍼 크기.
- **탐색 (Seeking)**:
  - HTTP Range Request 기반으로 동작한다. 후방 버퍼 내 탐색은 즉각적이나, 캐시 범위를 벗어난 전방 탐색은 새로운 HTTP 연결 수립 및 키프레임 탐색으로 인해 수백 ms 수준의 지연이 발생한다.

---

## Y5. R1·R2와의 관계

- **R1 (GPU 가속 ON/OFF) 적용**:
  - DASH/EDL 네트워크 스트림이라도 demuxer 단계 이후의 디코딩 파이프라인은 로컬 파일과 동일하다.
  - `hwdec=auto-safe`, `hwdec=d3d11va`, `hwdec=no` 옵션이 온전히 적용된다.
  - **코덱별 Windows 11 하드웨어 디코딩 지원 현황**:
    - **H.264 (avc1)**: 구형 Intel/AMD/NVIDIA GPU 전반에서 완벽 지원. 단, YouTube는 보통 1080p까지만 H.264를 제공한다.
    - **VP9 (vp09)**: Intel 7세대(Kaby Lake)+, NVIDIA GTX 950/10시리즈+, AMD RX 400+ 이상에서 4K 하드웨어 디코딩 지원.
    - **AV1 (av01)**: Intel 11세대(Tiger Lake)+ / Arc, NVIDIA RTX 30시리즈+, AMD RX 6000(6600 이상)시리즈+에서만 하드웨어 디코딩 지원. 미지원 GPU에서는 CPU 소프트웨어 디코딩(libdav1d)으로 전환되어 4K 재생 시 CPU 점유율이 급증한다.
- **R2 (영상 옵션 및 필터) 적용 차이**:
  - **VO 레벨 영상 옵션**: `gpu`/`gpu-next` 백엔드의 셰이더 기반 처리(밝기, 대비, 채도, 감마, deband, GLSL 셰이더)는 디코딩된 비디오 텍스처에 적용되므로 로컬 파일과 완전히 동일하게 동작한다.
  - **소프트웨어 비디오 필터 (`vf`)**: `hwdec=d3d11va`(다이렉트 제로카피) 사용 시 비디오 프레임이 VRAM에 머물러 CPU 기반 `vf` 필터 적용이 불가능하거나 복사 모드로 강제 강등된다. 따라서 소프트웨어 필터 적용 시 `hwdec=d3d11va-copy` 또는 `hwdec=no`를 사용해야 한다.

---

## Y6. 오류와 상태

- **연령·지역 제한 및 로그인 요구**:
  - 로그인이나 본인 인증이 필요한 동영상은 yt-dlp 호출 단계에서 오류("Sign in to confirm your age", "Video unavailable in your country")를 반환하며 재생이 시작되지 않는다.
- **봇 차단 및 쿠키**:
  - 대량 요청이나 데이터센터 IP 사용 시 "Sign in to confirm you're not a bot" 또는 HTTP 429 에러가 발생한다.
  - 해결책: `--ytdl-raw-options=cookies-from-browser=chrome` (또는 firefox/edge) 옵션을 주어 사용자의 브라우저 쿠키를 yt-dlp에 전달하거나 파일 경로(`cookies="path/to/cookies.txt"`)를 지정한다.
- **mpv 측 오류 수신 메커니즘**:
  - libmpv C API의 `mpv_wait_event()`를 통해 `MPV_EVENT_END_FILE`을 수신한다.
  - `mpv_event_end_file` 구조체의 `reason` 값이 `MPV_END_FILE_REASON_ERROR`인 경우 로딩 실패로 판별한다.
  - 상세 오류는 `mpv_request_log_messages(ctx, "warn")`를 등록하여 `[ytdl_hook]` 및 `[ffmpeg]` 접두어가 붙은 로그 메시지에서 추출한다.
- **media-title 및 썸네일**:
  - 제목: yt-dlp 파싱 완료 시 mpv 내부 속성 `media-title`에 YouTube 영상 제목이 자동으로 채워진다.
  - 썸네일: mpv 옵션 `script-opts-append=ytdl_hook-thumbnails=best` 설정 시 커버아트 비디오 트랙으로 등록된다. 앱 UI에 썸네일을 별도 표기하려면 사전 메타데이터 질의(`yt-dlp --dump-json`)를 수행하는 편이 안전하다.

---

## Y7. 링크 → 탭 연결

- **URL 판별 유형**:
  - 일반 영상: `https://www.youtube.com/watch?v=VIDEO_ID`
  - 단축 링크: `https://youtu.be/VIDEO_ID`
  - 쇼츠: `https://www.youtube.com/shorts/VIDEO_ID`
  - 재생목록: `https://www.youtube.com/playlist?list=PLAYLIST_ID`
  - 임베드: `https://www.youtube.com/embed/VIDEO_ID`
  - Rust 정규식(`regex`) 또는 `url` 크레이트로 호스트가 `youtube.com` 또는 `youtu.be`인지 검사하고 비디오 ID와 쿼리 파라미터를 추출한다.
- **재생목록 처리**:
  - 영상 URL에 `&list=...`가 포함된 경우 yt-dlp가 전체 목록을 파싱하려고 시도할 수 있다.
  - 단일 영상 재생 원칙 시 `--ytdl-raw-options=no-playlist=` 플래그를 전달하여 단일 비디오만 로드하도록 강제한다.
  - 재생목록 탭을 명시적으로 지원할 경우 해당 옵션을 해제하여 mpv 내부 플레이리스트로 바인딩한다.
- **시작 시각 (`t=`) 처리**:
  - `ytdl_hook` 스크립트는 URL의 `?t=120` 또는 `?t=1m30s` 파라미터를 파싱하여 mpv의 초기 재생 위치로 자동 전달한다.
  - Rust 앱 차원에서 파싱하여 `loadfile <url> replace start=<seconds>` 인수로 명시적 전달하는 것도 가능하다.

---

## Y8. 대안: Tauri 웹뷰의 YouTube IFrame Player API

- **개념**: libmpv 대신 Tauri의 WebView2 창에서 [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference)를 사용하여 공식 임베디드 웹 플레이어를 띄우는 방식이다.
- **장점**:
  - YouTube 서비스 약관을 100% 준수한다.
  - yt-dlp 바이너리 번들링, 버전 업데이트 관리, n-challenge 해독 문제가 완전히 불필요하다.
  - 연령 제한, CAPTCHA, 광고 재생, DRM 콘텐츠를 공식 웹 환경에서 자연스럽게 처리한다.
- **단점**:
  - **R1 (GPU 가속 ON/OFF 제어 불가)**: WebView2(Chromium) 내부의 ANGLE/D3D 하드웨어 가속 설정에 종속되며, libmpv의 세밀한 hwdec 옵션(ON/OFF, 디코더 선택) 제어가 불가능하다.
  - **R2 (영상 옵션 불가)**: libplacebo 기반 셰이더, GLSL 셰이더, deband, mpv 커스텀 색조/대비 보정, 소프트웨어 vf 필터 적용이 원천 차단된다(CSS 필터로 흉내 내는 수준에 불과함).
  - **자원 소모 및 UI 이질감**: WebView2 렌더러 프로세스가 추가 기동되어 메모리 점유율(약 150~300MB)이 증가하며, 기존 libmpv 네이티브 플레이어 UI와 디자인 일관성이 깨진다.

---

## Y9. 약관 (YouTube Terms of Service)

- **공식 URL**:
  - YouTube 서비스 약관: [https://www.youtube.com/static?template=terms](https://www.youtube.com/static?template=terms)
  - YouTube 개발자 정책: [https://developers.google.com/youtube/terms/developer-policies](https://developers.google.com/youtube/terms/developer-policies)
- **주요 조항 내용 (서비스 약관 4조 "권한 및 제한사항")**:
  - "귀하는 서비스 또는 콘텐츠의 일부분이라도 다음에 명시된 행위를 할 수 없습니다. (1) 서비스에서 특별히 허용한 경우, YouTube 및 해당 권리자의 사전 서면 승인이 있는 경우, 또는 관련 법률에 따라 허용된 경우를 제외하고 복제, 다운로드, 배포, 전송, 방송, 전시, 판매, 라이선스 부여, 변경, 수정 또는 기타 방식으로 사용."
  - "기술적 보호 조치, 콘텐츠 복제 방지/제한 조치, 서비스 이용 제한 조치 등 보안 관련 기능을 우회, 무력화, 부정하게 개입 또는 방해하는 행위."
  - "공개 검색 엔진, 사전 서면 승인, 관련 법률 허용을 제외하고 자동화된 수단(로봇, 봇넷, 스크래퍼 등)을 사용하여 서비스에 접근하는 행위."
- **개발자 정책 기준**:
  - 스트림(오디오/비디오)을 분리하거나 광고를 인위적으로 건너뛰는 형태의 독립 플레이어 구현을 금지하고 있으며, 공식 IFrame Player API 사용을 요구한다.
- **개인 사용과 배포의 사실적 차이**:
  - **개인 사용자 로컬 실행**: 개인이 오픈소스 yt-dlp를 로컬에서 구동하는 경우 법적 소송의 대상이 된 선례는 극히 드물며, 기술적 차단(IP 임시 차단, 봇 검증) 위주로 대응된다.
  - **앱 번들 배포**: 소프트웨어 개발사가 yt-dlp를 상용/공개 앱에 번들링하여 스트림을 우회 추출하는 플레이어로 배포할 경우, 서비스 약관 위반 및 DMCA 1201조(기술적 보호조치 우회 금지) 위반을 이유로 YouTube(구글) 측의 배포 중단 요구(Cease & Desist)나 플랫폼 퇴출 조치를 받을 위험이 실질적으로 존재한다.

---

## Y10. 확신 낮은 부분

1. **PoToken 및 SABR 스트리밍 도입 여파 [추정/변동 가능]**:
   - YouTube가 봇 방지 및 서명 암호화를 위해 도입 중인 PoToken(Proof of Origin Token)과 SABR 프로토콜의 적용 강도가 지역/계정에 따라 수시로 달라진다. 향후 yt-dlp만으로 스트림 추출이 지속 가능할지 여부는 공식 보장되지 않는다.
2. **Windows 독립 배포 시 EJS 런타임 번들링 부담 [미확인]**:
   - yt-dlp의 n-challenge 대응을 위해 Deno 바이너리(약 30~50MB)를 앱에 함께 패키징해야 할지, 혹은 QuickJS 기반 경량 바인딩으로 대체 가능한지에 대한 벤치마크 검증이 필요하다.
3. **Rust MSVC 툴체인과 libmpv MinGW 빌드 간의 ABI/예외 호환성 [추정]**:
   - shinchiro/zhongfly 빌드는 GCC/MinGW-w64 기반이다. `libmpv`의 순수 C ABI는 `rust-stable-msvc`와 호환되나, DLL 내부에서 예외나 런타임 충돌 발생 시 크래시 덤프 추적에 한계가 있을 수 있다.
