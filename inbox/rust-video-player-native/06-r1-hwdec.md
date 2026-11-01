---
tags: [rust, video-player, r1, hwdec, libmpv]
created: 2026-11-01
updated: 2026-11-01
---

# 06. R1 — GPU 가속 ON/OFF 와 자동 전환

> 이 파일을 읽는 때: 하드웨어 디코딩 토글, 실패 시 소프트웨어 전환, 상태 표시를 만들 때.

## 1. mpv 가 주는 것 (v0.41.0 문서)

| 사실 | 원문 요지 | 출처 |
| --- | --- | --- |
| 실행 중 전환 | "Use the Ctrl+h shortcut to toggle hardware decoding at runtime. It toggles this option between auto and no." | [options.rst `--hwdec`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst), [매뉴얼 Video](https://mpv.io/manual/stable/#video) |
| render API 에서도 가능 | "hardware decoding can be enabled/disabled at any time by setting the hwdec property" | [render_gl.h v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/include/mpv/render_gl.h) |
| 불가능하면 SW | "If hardware decoding is not possible, mpv will fall back on software decoding." | options.rst `--hwdec` |
| 실패하면 SW | `--hwdec-software-fallback=<yes\|no\|N>`: 하드웨어 디코더가 실패하면 SW 로 전환한다. 기본값 3(연속 3프레임 실패). 값을 크게 잡으면 전환 때 파일 일부를 건너뛸 수 있다 | options.rst |
| `auto` | 화이트리스트(안전한 방법)만 쓴다. 문제가 알려진 상황에서는 하드웨어 디코딩을 끌 수도 있다. 모두 실패하면 SW 로 간다 | options.rst |
| `auto-safe` | v0.41.0 에서는 `auto` 와 같다 | options.rst |
| `auto-copy` | copy-back 모드만 고른다. "will allow CPU processing with video filters" | options.rst |
| 코덱 제한 | 켜도 일부 코덱만 하드웨어로 디코딩한다(`--hwdec-codecs`) | options.rst |
| 실제 상태 | `hwdec-current`: 지금 쓰는 hwdec. `no` 면 SW. 디코더가 없으면 속성이 없다 | [input.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/input.rst), [매뉴얼 Property List](https://mpv.io/manual/stable/#property-list) |
| interop 상태 | `hwdec-interop`: 로드된 interop 드라이버 | input.rst |

"재생 중 전환"을 무중단 전환으로 보장하는 문구는 문서에 없다. 그래서 이 묶음에서는 "전환 후 복구되고 UI 가 멈추지 않는다"를 기준으로 삼는다([00-requirements.md](00-requirements.md)).

## 2. 1단계(glow)에서 실제로 일어나는 일

[01-stack-decision.md](01-stack-decision.md) 3.1 대로, eframe WGL 위의 mpv OpenGL render API 에서는 `d3d11va` 직접 모드를 쓸 수 없다. v0.41.0 `vd_lavc.c` 의 autoprobe 표에는 `d3d11va`, `nvdec` 다음에 `d3d11va-copy`, `dxva2-copy`, `nvdec-copy` 가 화이트리스트로 올라 있다. 그래서 ON(`auto`)이면 보통 copy 모드가 고려된다 ([vd_lavc.c v0.41.0](https://github.com/mpv-player/mpv/blob/v0.41.0/video/decode/vd_lavc.c), sol followup-1). 어느 PC 에서 무엇이 고를지는 실측해야 한다.

| UI 상태 | 설정할 값 | 기대하는 `hwdec-current` (확인 필요) |
| --- | --- | --- |
| GPU 가속 ON (기본) | `hwdec=auto` | `d3d11va-copy`, `nvdec` 또는 `nvdec-copy`, 실패하면 `no` |
| GPU 가속 ON (진단용 고정) | `hwdec=d3d11va-copy` | `d3d11va-copy` 또는 `no` |
| GPU 가속 OFF | `hwdec=no` | `no` |

## 3. 앱 설계

### 3.1 상태 표시

UI 에는 두 값을 따로 보인다.

- **요청**: 사용자가 고른 ON/OFF
- **실제**: `hwdec-current` 를 `observe_property` 로 받은 값 (`HW: d3d11va-copy` / `SW`)

요청이 ON 인데 실제가 `no` 면 "하드웨어 디코딩 불가 → 소프트웨어로 재생 중" 배지를 띄운다. 이유는 mpv 로그 메시지(`mpv_request_log_messages`)에서 찾는다.

### 3.2 전환 상태 기계 (설계 제안)

```text
Playing(req=On|Off, actual=…)
   │  SetHwdec(new)
   ▼
Reconfiguring  ── 위치·일시정지·속도·트랙·필터를 스냅샷
   │  set_property("hwdec", new)
   ▼
hwdec-current 변경 관찰 ──▶ Playing(req=new, actual=…)  + Event::HwdecChanged
   │  타임아웃 / 오류
   ▼
Recovering ── 스냅샷 위치로 seek, actual 을 그대로 보고
```

- mpv 가 속성 변경만으로 디코더를 다시 만드는지, 그때 위치·필터가 그대로 유지되는지는 실행해서 확인해야 한다. 그래서 스냅샷을 떠 두고, 위치가 어긋나면 다시 맞춘다.
- 자동 폴백은 mpv(`--hwdec-software-fallback`)에 맡긴다. 같은 파일에서 하드웨어 재시도를 무한히 반복하지 않게 한다.
- 앱 감시(watchdog): 일정 시간 위치가 진행되지 않으면 `hwdec=no` 로 한 번 내리고 마지막 위치로 돌아간다. GPU 장치가 통째로 사라진 경우(device lost)에도 SW 디코딩만으로 복구된다고 보장할 수는 없다. 이때는 render context 를 다시 만들어야 할 수 있다(미확인).

### 3.3 여러 탭

탭마다 mpv core 가 따로 있으므로 R1 도 탭마다 따로 적용된다. 하드웨어 디코더 생성이 실패하면 그 탭만 SW 가 된다. 고정된 세션 수 상한은 근거가 없으며, VRAM 부족이나 미지원 프로파일이 원인이다 ([Microsoft CreateVideoDecoder](https://learn.microsoft.com/en-us/windows/win32/api/d3d11/nf-d3d11-id3d11videodevice-createvideodecoder), argon followup-1). 전역 설정 하나로 모든 탭을 바꿀지, 탭마다 바꿀지는 UI 에서 정한다.

## 4. R1 과 R2 의 상호작용

- CPU 필터(lavfi: hqdn3d·gblur·cas 등)는 시스템 RAM 의 프레임이 필요하다. 직접 하드웨어 모드에서는 필터 때문에 copy-back 이 필요하다 ([options.rst `auto-copy`](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)).
- 1단계는 이미 copy-back 이므로 이 충돌이 크지 않다. 2단계(직접 디코딩)로 가면 "R2 필터 ON ⇒ copy 모드 또는 SW" 를 UI 에 알린다.
- "HW ON" 과 "zero-copy" 는 다른 말이다. UI 에는 `hwdec-current` 값을 그대로 보인다.

## 5. 2단계와 대안에서

| 경로 | R1 |
| --- | --- |
| 2단계 (D3D11 render API) | mpv 가 D3D11 장치를 쓰므로 `d3d11va` 직접 모드를 기대할 수 있다. 0.42 출시 후 확인한다 |
| Win32 + `wid` | `--gpu-api=d3d11` 이면 `d3d11va` 직접 모드 ([options.rst](https://github.com/mpv-player/mpv/blob/v0.41.0/DOCS/man/options.rst)) |
| FFmpeg 직접 | 위치 저장 → 디코더 컨텍스트 다시 만들기(D3D11VA ↔ CPU) → 키프레임 탐색 → 목표 PTS 까지 디코딩해 버리기 → 오디오 클럭 보정. 난이도가 높다 ([FFmpeg D3D11VA](https://ffmpeg.org/doxygen/trunk/hwcontext__d3d11va_8h_source.html)) |

---

이전: [05-explorer-tabs.md](05-explorer-tabs.md) · 다음: [07-r2-video-options.md](07-r2-video-options.md)
