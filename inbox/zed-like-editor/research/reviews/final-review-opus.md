# 최종 사실 리뷰 — scouter-opus (02-gpui/README.md, 03-tauri/README.md, 04-COMPARISON.md)

- 2026-10-31. 문서는 읽기만 했다. 대조 기준은 crates.io API, npm registry, Zed 커밋 `14b7b0e` 소스, 01-zed 명세(ID 존재 여부)다.
- 심각도: **높음** = 사실 오류나 구현 판단을 틀리게 만드는 것 / **중간** = 빠진 중요 항목, 잘못된 ID 참조 / **낮음** = 표기·보완.

## 결과

| 번호 | 심각도 | 파일·위치 | 문제 | 근거(URL) | 고칠 방향 |
|---|---|---|---|---|---|
| F-01 | 높음 | 02-gpui 3절 G-022, 1.3절(04) "tree-sitter `wasm` 기능 + wasmtime" | `wasmtime` 50.0.1을 따로 의존하는 것처럼 적었다. 실제로 tree-sitter의 `wasm` 기능은 **`wasmtime-c-api-impl`을 자체 버전으로 고정**해 끌어온다(tree-sitter 0.26.13 → `wasmtime-c-api-impl ^0.72.0`). 앱이 wasmtime 50을 따로 넣으면 버전이 둘이 되거나 쓸모가 없다 | [tree-sitter 0.26.13 의존성](https://crates.io/crates/tree-sitter/0.26.13/dependencies), [wasmtime-c-api-impl](https://crates.io/crates/wasmtime-c-api-impl) | G-022를 "wasmtime은 tree-sitter `wasm` 기능이 함께 가져온다(버전은 tree-sitter가 정함). 직접 의존 없음"으로 고친다. G-108 확인 방법에 "gpui-component가 고정한 tree-sitter 0.26 계열에서 `wasm` 기능을 켰을 때의 빌드 크기·시간" 추가 |
| F-02 | 높음 | 02-gpui G-019·G-042·G-125, 03-tauri R-15 | **TaskDialog는 Common Controls v6 매니페스트가 있어야 쓸 수 있다**(`TaskDialogIndirect`는 ComCtl32 6.0 전용). 두 문서 모두 앱 매니페스트(comctl32 v6, per-monitor DPI v2, longPathAware) 준비를 확인 항목에 넣지 않았다. gpui-fast에는 `windows-manifest-provided-by-host` 기능이 있어, 매니페스트를 누가 넣는지가 크레이트 설정에 걸려 있다 | [MS Learn TaskDialogIndirect](https://learn.microsoft.com/windows/win32/api/commctrl/nf-commctrl-taskdialogindirect), [gpui-component 0.8.1 의존성(gpui-fast 기능)](https://crates.io/crates/gpui-component/0.8.1/dependencies) | 02: G-125에 "앱 매니페스트(comctl32 v6·DPI·긴 경로)를 누가 넣나(gpui-fast `windows-manifest-provided-by-host`, `embed-resource` 등)" 추가. 03: R-15에 "tauri-build 기본 매니페스트에 comctl32 v6가 들어가는지(미확인), 아니면 사용자 매니페스트" 추가 |
| F-03 | 높음 | 04-COMPARISON 1.3절 "패키징·서명" Tauri 칸 | "**03-tauri에 항목 없음 (미확인)**"이라고 적었으나 03에는 **R-29**(패키징·서명·WebView2 배포)와 **R-30**(빌드 준비물)이 있다 | 03-tauri 5절 R-29, R-30 | "R-29: MSI·NSIS, WebView2 부트스트래퍼·오프라인·고정 버전, 서명" 으로 고친다. 런타임 준비물 칸에 R-30 연결 |
| F-04 | 중간 | 04 2.4절 "E-086 디스크 변경 감지 (G-115 / **R-16**)" | R-16은 "설정·키맵 외부 편집 감시와 자기 쓰기 루프"다. 열린 파일의 디스크 변경은 **R-24** | 03-tauri R-16, R-24 | `(G-115 / R-24)`로 고친다 |
| F-05 | 중간 | 04 2.7절 "K-070 ~ K-075 keymap.json 쓰기·감시", 03 R-16 관련 ID "K-070 ~ K-075" | 02-INTERACTION의 K-070~K-079는 **키맵 편집 화면의 키**다. keymap.json 쓰기·unbind·반영은 **K-090~K-097** | 02-INTERACTION K-070~K-079(키), K-090~K-097(저장) | 04·03의 해당 참조를 `K-090 ~ K-097`로 고친다. 03 4절·R-23의 "K-001 ~ K-085" 범위도 `K-001 ~ K-097`로 넓힌다 |
| F-06 | 중간 | 04 2.2절 "I-731 ~ I-739 VSCode 덧씌움 결과 7개" | I-731~I-739는 **Pane 키보드 최종 적용표**다. VSCode 덧씌움 바인딩은 **I-740~I-760(21개)**이고, 그중 채용은 I-742·I-743·I-745·I-746·I-747·I-749·I-752의 7개다 | 02-INTERACTION I-731~I-739, I-740~I-760 | "I-740 ~ I-760 중 채용 7개(I-742, I-743, I-745 ~ I-747, I-749, I-752)"로 고치고, I-731~I-739는 분할 행(2.2절 I-706~I-730 행)에 붙인다 |
| F-07 | 중간 | 03-tauri 3.1절 `windows`·`tar`·`flate2` "버전 미확인", R-21 `tauri-plugin-window-state` "버전 미확인" | 조회 가능하다: `windows` 0.62.2, `tar` 0.4.46, `flate2` 1.1.10, `tauri-plugin-window-state` 2.5.0 (02의 G-025 후보 `zip` 9.1.3) | [windows](https://crates.io/crates/windows), [tar](https://crates.io/crates/tar), [flate2](https://crates.io/crates/flate2), [tauri-plugin-window-state](https://crates.io/crates/tauri-plugin-window-state), [zip](https://crates.io/crates/zip) | 값을 채우고 "2026-10-31 조회"로 표시. 라이선스는 구현 때 확인 |
| F-08 | 중간 | 03-tauri 2절·3.2절 (web-tree-sitter in WebView) | **CSP 설정**이 빠졌다. Tauri 앱에 CSP를 걸면 `WebAssembly.instantiate`에 `'wasm-unsafe-eval'`가 필요하고, 확장 폴더의 문법·SVG를 커스텀 프로토콜·에셋 프로토콜로 줄 때 `connect-src`·`img-src`·`asset` 범위(scope)를 열어야 한다 | [Tauri CSP 문서](https://v2.tauri.app/security/csp/), [MDN CSP wasm-unsafe-eval](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/script-src) | R-03 또는 새 R- 항목: "CSP·에셋 범위에서 web-tree-sitter·문법 WASM·확장 SVG 로드" (P0, R-19와 함께) |
| F-09 | 중간 | 03-tauri R-02·R-03 | CM6에는 Tree-sitter 연동이 없다. **CM6 트랜잭션 → `Tree.edit()` 증분 파싱 → 보이는 범위만 강조 장식** 다리를 직접 만들어야 하고, 이것이 큰 파일 성능(R-10)과 구문 선택(E-090~E-093)에 직결된다. 03은 "CM6로 편집, web-tree-sitter로 강조"만 적었다 | [@codemirror/view](https://www.npmjs.com/package/@codemirror/view), [web-tree-sitter](https://www.npmjs.com/package/web-tree-sitter) | R-02 또는 R-10에 "CM6↔Tree-sitter 증분 동기화와 Lezer 기본 강조를 끄는 방법" 추가 (P0) |
| F-10 | 중간 | 02-gpui·03-tauri 5절 (둘 다) | **줄 끝(CRLF/LF)과 인코딩** 처리가 빠졌다. Windows에서 CRLF 파일을 열고 저장했을 때 줄 끝이 보존되는지, UTF-8이 아닌 파일(UTF-16, CP949)을 어떻게 다룰지. Zed는 버퍼마다 줄 끝을 기억하고, 상태 표시줄에 인코딩 버튼(`active_encoding_button: "non_utf8"`)이 있다 | [Z/assets/settings/default.json `status_bar`](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/assets/settings/default.json#L2031-L2045) | 두 문서에 G-·R- 항목 추가: "CRLF 보존·인코딩 감지·저장 시 E-058/E-059(공백 제거·끝 줄바꿈)와의 관계" (중간 / P1) |
| F-11 | 중간 | 02-gpui 5절 | **한글 대체 글꼴**이 빠졌다. `.ZedSans`(IBM Plex Sans)·`.ZedMono`(Lilex)에는 한글이 없어 gpui(DirectWrite)의 글꼴 대체가 줄 높이·기준선·폭을 바꿀 수 있다. 03은 R-32에 넣었다 | 03-THEME-ICONS T-15.5, 03-tauri R-32 | G- 항목 추가: "DirectWrite 대체 글꼴(Malgun Gothic 등)로 한글 혼합 줄 높이·거터·커서 위치 측정" (중간) |
| F-12 | 낮음 | 02-gpui G-121 "Rust 1.92 이상" | crates.io의 gpui-kit `rust_version`은 **1.90**이다. gpui-kit 설치 문서가 의존성 그래프 기준으로 1.92를 요구한다면 근거를 같이 적어야 한다 | [crates.io gpui-kit](https://crates.io/crates/gpui-kit) | "MSRV 1.90(crates.io), 설치 문서 1.92(의존성 기준)"처럼 두 값을 출처와 함께 적는다 |
| F-13 | 낮음 | 02-gpui 8절 GitHub 링크 | `github.com/longbridge/gpui-component`로 걸었으나 crates.io의 저장소는 **longbridge/gpui-kit**(이름 변경) | [crates.io gpui-kit repository](https://crates.io/crates/gpui-kit) | 링크를 `https://github.com/longbridge/gpui-kit`로 고친다 |
| F-14 | 낮음 | 03-tauri R-20, 6절 "WebView2가 `Ctrl+W`, `F1`, `F6`을 먼저 처리" | WebView2의 기본 브라우저 가속키 목록에 `Ctrl+W`·`F1`이 있는지는 근거가 없다. 확실한 것은 `Ctrl+P`·`Ctrl+F`·`Ctrl+R`/`F5`·`F3`·`Alt+Left/Right`·`Ctrl+Plus/Minus` 계열이다. 끄는 방법(`AreBrowserAcceleratorKeysEnabled`)이 Tauri에서 노출되는지도 미확인 | [MS Learn AreBrowserAcceleratorKeysEnabled](https://learn.microsoft.com/microsoft-edge/webview2/reference/win32/icorewebview2settings3) | 목록을 "확실한 것 / 시험할 것"으로 나누고, Tauri·wry에서 설정 노출 여부를 확인 방법에 넣는다 |
| F-15 | 낮음 | 04 1.1절 "Rust 코어 1개 + 창마다 WebView2 프로세스 1개" | WebView2는 브라우저 프로세스·GPU·렌더러 등 **프로세스 묶음**을 사용자 데이터 폴더 단위로 공유한다. "창마다 1개"는 단순화다 | [MS Learn WebView2 프로세스 모델](https://learn.microsoft.com/microsoft-edge/webview2/concepts/process-model) | "창마다 렌더러(브라우저 프로세스는 공유)"로 고친다 |
| F-16 | 낮음 | 04 2.3절 "L-236, L-237 비활성 버튼도 툴팁이 뜸 … gpui 쪽은 확인 못 했다고" | Zed `ButtonLike`는 disabled와 상관없이 툴팁을 붙인다(클릭만 막는다). 1차 검토 B-09에서 지적했다 | [Z/crates/ui/src/components/button/button_like.rs#L883-L895](https://github.com/zed-industries/zed/blob/14b7b0ef3764d62a85070c8b5ccea105a2ad3235/crates/ui/src/components/button/button_like.rs#L883-L895) | "Zed 소스로 확인됨(툴팁 표시), 비활성 hover 이벤트는 PoC로 확인"으로 고친다 |
| F-17 | 낮음 | 02 ↔ 03 짝 구조 | 02는 우선순위가 높음·중간·낮음, 03은 P0·P1·P2다. 02는 "6.1 확신 낮은 부분"이 있고 03에는 같은 절이 없다(5.1·5.2 요약이 있음). 04가 두 문서를 나란히 비교하므로 기준이 다르면 비교가 어렵다 | 02 5·6절, 03 5절 | 한쪽으로 통일(예: 높음=P0 …), 03에 "확신 낮은 부분" 절 추가 (checker 영역과 겹침) |
| F-18 | 낮음 | 03-tauri 3.2절 `dompurify` 라이선스 | "MPL-2.0 OR Apache-2.0"은 맞다(npm `(MPL-2.0 OR Apache-2.0)`). 표 라이선스 열은 "MIT"로 묶여 있어 읽기에 따라 헷갈린다 | [dompurify](https://www.npmjs.com/package/dompurify) | 마크다운 행의 라이선스 열을 "MIT, dompurify만 MPL-2.0 OR Apache-2.0"으로 명시 |

## 출발 문서로서 더 넣을 만한 항목 (요약)

| 대상 | 항목 | 이유 |
|---|---|---|
| 02·03 공통 | 앱 매니페스트(comctl32 v6, per-monitor DPI v2, longPathAware) | TaskDialog·고DPI·긴 경로 (F-02) |
| 02·03 공통 | CRLF·인코딩 | Windows 파일 보존 (F-10) |
| 02 | 한글 대체 글꼴 측정 | IME와 별개로 렌더링 차이 (F-11) |
| 03 | CSP·에셋 범위 | WASM·확장 자산 로드 (F-08) |
| 03 | CM6 ↔ Tree-sitter 증분 연동 | 성능·구문 선택의 핵심 (F-09) |
| 02·03 공통 | 단일 인스턴스·명령줄 인자(파일을 앱으로 열기, 이미 실행 중이면 새 창/탭) | New Window·Open File 포함 결정과 연결. Zed는 CLI로 기존 창에 연다 (판단) |

## 04의 비교·권고와 02·03의 일치

| 점검 | 결과 |
|---|---|
| 1순위 gpui-kit·2순위 Tauri 권고 | 02·03 머리 표와 같다 (일치) |
| 최대 위험 (gpui: IME·편집기·버전 / Tauri: 단축키·편집기 상태·창 장식·문법 WASM) | 02 G-130~G-132, 03 P0(R-01·R-02·R-03·R-06·R-19·R-20)와 일치 |
| Tauri 택할 조건(3.2절) | 02 G-100·G-101·G-102·G-104·G-108 ↔ 03 R-08·R-02·R-10·R-09·R-03/R-19 대응이 맞다 |
| 틀린 연결 | F-03(패키징), F-04(R-16→R-24), F-05(K-070~→K-090~), F-06(I-731~→I-740~) |
| 공통 의존 | `nucleo`(02) / `nucleo-matcher`(03) 구분은 맞다. `windows`는 03에서 미확인으로 남아 있다 (F-07) |

## 대조한 표본 (27개)

| 대상 | 문서 값 | 조회 값 | 결과 | 출처 |
|---|---|---|---|---|
| gpui-kit | 0.8.1, Apache-2.0 | 0.8.1, Apache-2.0 | 일치 | https://crates.io/crates/gpui-kit |
| gpui-component | 0.8.1, Apache-2.0 | 0.8.1, Apache-2.0 | 일치 | https://crates.io/crates/gpui-component |
| gpui-base | 0.8.1 | 0.8.1 | 일치 | https://crates.io/crates/gpui-base |
| gpui-pre | =0.3.10, Apache-2.0 | 0.3.10, Apache-2.0 | 일치 | https://crates.io/crates/gpui-pre |
| gpui-kit MSRV | Rust 1.92 | rust_version 1.90 | 다름 (F-12) | https://crates.io/crates/gpui-kit |
| tree-sitter | 0.27.1 최신 / 0.26 계열 고정 | 0.27.1 / 0.26.13 | 일치 | https://crates.io/crates/tree-sitter |
| tree-sitter `wasm` 의존 | wasmtime 50 | wasmtime-c-api-impl ^0.72 | 다름 (F-01) | https://crates.io/crates/tree-sitter/0.26.13/dependencies |
| notify | 8.2.1, CC0-1.0 | 8.2.1, CC0-1.0 | 일치 | https://crates.io/crates/notify |
| trash | 5.2.9, MIT | 5.2.9, MIT | 일치 | https://crates.io/crates/trash |
| rfd | 0.17.2, MIT | 0.17.2, MIT | 일치 | https://crates.io/crates/rfd |
| windows | 0.62.2 (02) / 미확인 (03) | 0.62.2 | 03 보완 (F-07) | https://crates.io/crates/windows |
| serde_json | 1.0.154 | 1.0.154 | 일치 | https://crates.io/crates/serde_json |
| jsonc-parser | 0.34.0 | 0.34.0 | 일치 | https://crates.io/crates/jsonc-parser |
| nucleo | 0.5.0, MPL-2.0 | 0.5.0, MPL-2.0 | 일치 | https://crates.io/crates/nucleo |
| nucleo-matcher | 0.3.1, MPL-2.0 | 0.3.1 | 일치 | https://crates.io/crates/nucleo-matcher |
| dirs | 7.0.0 | 7.0.0 | 일치 | https://crates.io/crates/dirs |
| tauri | 2.12.5, Apache-2.0 OR MIT | 2.12.5 | 일치 | https://crates.io/crates/tauri |
| tauri-plugin-dialog / fs / opener | 2.8.4 / 2.7.1 / 2.7.2 | 같음 | 일치 | https://crates.io/crates/tauri-plugin-dialog |
| toml | 1.1.8 | 1.1.8+spec-1.1.0 | 일치 | https://crates.io/crates/toml |
| @tauri-apps/api | 2.12.4 | 2.12.4, Apache-2.0 OR MIT | 일치 | https://www.npmjs.com/package/@tauri-apps/api |
| @codemirror/view·state·search | 6.43.15 / 6.7.6 / 6.7.2 | 같음, MIT | 일치 | https://www.npmjs.com/package/@codemirror/view |
| monaco-editor | 0.57.0 | 0.57.0, MIT | 일치 | https://www.npmjs.com/package/monaco-editor |
| web-tree-sitter | 0.27.1 | 0.27.1, MIT | 일치 | https://www.npmjs.com/package/web-tree-sitter |
| remark-gfm / markdown-it | 4.0.1 / 15.0.3 | 같음, MIT | 일치 | https://www.npmjs.com/package/markdown-it |
| dompurify | 3.4.17, MPL-2.0 OR Apache-2.0 | 3.4.17, (MPL-2.0 OR Apache-2.0) | 일치 (표기 F-18) | https://www.npmjs.com/package/dompurify |
| @tanstack/virtual-core | 3.19.9 | 3.19.9, MIT | 일치 | https://www.npmjs.com/package/@tanstack/virtual-core |
| solid-js / svelte / react | 1.9.18 / 5.57.3 / 19.3.1 | 같음, MIT | 일치 | https://www.npmjs.com/package/solid-js |

## 확신 낮은 부분

- **F-01:** `wasmtime-c-api-impl ^0.72.0`은 crates.io 의존성 응답을 그대로 읽은 값이다. 이 크레이트의 버전 번호가 wasmtime 본체 번호와 어떻게 대응하는지는 확인하지 않았다. 다만 "tree-sitter가 자체 고정"이라는 결론은 바뀌지 않는다.
- **F-02:** tauri-build 기본 매니페스트에 comctl32 v6가 들어 있는지, gpui-kit 앱의 기본 매니페스트 상태는 확인하지 않았다. 확인 항목으로 넣으라는 지적이다.
- **F-08, F-14:** Tauri의 CSP 기본값과 WebView2 가속키 노출 여부는 공식 문서 페이지를 링크했지만 본문을 대조하지 않았다.
- **F-15:** WebView2 프로세스 공유 단위(사용자 데이터 폴더·환경)는 Tauri 다중 창 설정에 따라 달라질 수 있다.
- **라이선스:** crates.io 조회의 `license` 값은 최신 버전 기준이고, 고정한 하위 버전의 라이선스는 따로 보지 않았다.
