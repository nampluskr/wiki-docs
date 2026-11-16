# Termius 대체 안드로이드 SSH 터미널 앱 조사 보고서

- 작성일: 2026-11-16
- 조사 기준 구성: [ssh-tailscale-remote-terminal.md](file:///D:/projects/wiki-docs/wiki/dev/web/ssh-tailscale-remote-terminal.md), [README.md](file:///D:/projects/wiki-docs/inbox/261104-web-herdr-android-password-ssh/README.md)

## 1. 기준 환경 및 요구사항 요약

위키 문서 두 편에서 확인한 현재 구성과 제약은 다음과 같다[1][2].

- **접속 대상(서버)**: Windows 11 데스크탑(`desktop.taila5bb9e.ts.net` / `100.87.213.81`, 사용자 `nampl`, 포트 `22`) 및 노트북(`laptop`) 총 2대. Windows 기본 OpenSSH `sshd`, 기본 셸 `powershell.exe`.
- **네트워크**: 모든 기기에 Tailscale 설치 및 상시 VPN(안드로이드는 Always-on VPN).
- **인증 방식**: **공개키 전용**(`PasswordAuthentication no`). 기기마다 Ed25519 키를 따로 만들고 개인키는 기기 밖으로 옮기지 않는다. 접속받는 PC가 관리자 계정이므로 공개키는 `C:\ProgramData\ssh\administrators_authorized_keys`에 등록한다.
- **클라이언트 기기**: 안드로이드 폴더블 폰(Galaxy Z Fold 5)과 안드로이드 태블릿(`tab`).
- **핵심 필수 조건**:
  1. 앱 안에서 Ed25519 키 생성 및 공개키 내보내기(Taildrop 등 공유 시트 연동 가능).
  2. Windows OpenSSH + PowerShell 셸과 호환.
  3. 무료 또는 구독 부담 없는 가격 구조(Termius는 Pro 체험 종료 후 Starter 플랜에서 클라우드 동기화·스니펫·고급 기능이 잠김[3]).
  4. 폴더블·태블릿 화면과 외부 키보드, TUI·한글 입력·백그라운드 세션 유지.
- **사전 제외 대상**:
  - **JuiceSSH**: 2021년 초 이후 업데이트 중단, 2025년 말~2026년 초 최신 Android API 미준수로 Google Play 스토어에서 제거됨[4][5].
  - **herdr 안드로이드 앱**: 비밀번호 입력 칸만 확인되고 키 관리가 없으며, 로그인 직후 리눅스 POSIX 셸 명령으로 `herdr-mobile-bridge`를 탐지해 PowerShell 데스크탑에서 실패함(2026-11-04 시험)[2].
  - **Termark**: 데스크탑·iOS 중심이며 안드로이드는 Play 스토어 정식 출시가 아닌 Beta/사이드로드 APK 형태[6].

---

## 2. 안드로이드 SSH 터미널 앱 후보 비교표

> **표기 원칙**: 확인하지 못한 항목은 `미확인`으로, 직접 관찰된 문서에 명시되지 않고 정황상 추론한 내용은 `(추정)`으로 표기했다.

| 항목 | ZestSSH | Termux (+ OpenSSH) | ConnectBot | ServerBox | Admin Hands | Moshi |
| --- | --- | --- | --- | --- | --- | --- |
| **가격 / 무료 범위** | **무료 티어**(계정 불필요, SFTP·포트포워딩·자가 스토리지 E2EE 동기화 포함; 플랫폼별 2세션 제한 또는 무제한 안내 혼재)[7][8][9]<br>Pro **~$9.99 일회 결제**(세션 무제한·위젯·단축키), Cloud Sync ~$24.99 일회 결제. **구독 없음**[7][8] | **완전 무료**, 광고·구독 없음[10] | **완전 무료**, 광고·구독 없음[11] | **무료**(GitHub/F-Droid 배포 기준, 핵심 기능 페이월 없음; 일부 스토어 후원/프리미엄 옵션)[12][13] | Play 스토어 **무료** 다운로드, **광고·트래킹 없음**[14][15]<br>무료 저장 호스트 **5개 제한** 및 프리미엄 인앱 결제 안내[16] vs 핵심 무료 안내 혼재(정확한 인앱 금액 **미확인**) | **무료 티어**(저장 호스트 **2개**, 활성 세션 무제한, 키 인증·생체 잠금)<br>Pro **$9.99/월**, $89.99/년, $199 평생(최대 3대; 무제한 호스트·Mosh·tmux/herdr 자동 재연결)[17][18] |
| **배포처 / 최신 업데이트 / 오픈소스** | Play 스토어 배포[9][19]<br>v3.4 / 3.4.2 (2026년 업데이트)[20][21]<br>**클로즈드 소스**[8] | **GitHub Releases / F-Droid 권장**(Play 스토어 버전은 기능·권한 축소로 비추천)[22]<br>v0.118.1(2024), v0.119.0 베타(2025-03 `beta.2` ~ 2026년 개발 지속)[23][24]<br>**오픈소스(GPLv3)**[10] | Play 스토어(`google`) & F-Droid/GitHub(`oss`)[11][25]<br>**v1.11.1 (2026년 10월)**[26]<br>**오픈소스(Apache 2.0)**[11] | GitHub Releases / F-Droid 배포(**Play 스토어 2026년 기준 미확인/미노출 가능성**)[12][27]<br>v1.0.19xx (2026년 업데이트)[27][28]<br>**오픈소스(AGPLv3)**[12][29] | Play 스토어(`com.arpaplus.adminhands`) 배포[14][30]<br>v3.5 (2026년 후반)[31]<br>**클로즈드 소스**(추정: 소스 저장소 미확인) | Play 스토어 배포(*Moshi: tmux/herdr SSH terminal*)[32]<br>2026년 활성 업데이트<br>**클로즈드 소스**[33] |
| **Ed25519 키 (생성 / 내보내기 / 가져오기)** | 앱 내 **생성 가능**, `More > Keys` 길게 눌러 `Share public key`로 **내보내기 가능**, OpenSSH/PuTTY **가져오기 가능**[7][34] | `ssh-keygen -t ed25519`로 **생성**, `cat ~/.ssh/id_ed25519.pub` 복사·공유로 **내보내기**, `~/.ssh/` 복사로 **가져오기**[10][35] | `Manage Pubkeys`에서 **생성·복사·가져오기** 지원[11][36]. 과거 Android 13~16 복사/인증 버그 있었으나 v1.10.9·v1.11.x에서 수정 반영[37][38] | 앱 내 **생성·공개키 확인** 및 **개인키 가져오기** 지원[28][39]. 단, 줄바꿈/PEM 헤더 관련 키 포맷 이슈(`srvbox_pk_v1: invalid format` 등) 보고 이력 있음[40] | 앱 내 **생성**(Ed25519/RSA/ECDSA/DSA) 및 관리 지원[30]. 공개키 내보내기 UI 상세 경로·가져오기 포맷 **미확인** | 앱 내 **네이티브 생성** 및 **공개키 제공**, 기존 개인키 **가져오기** 지원, 생체인증 키 보호[18][41] |
| **Windows OpenSSH + PowerShell 이슈** | Windows 로컬 PowerShell/WSL·OpenSSH 에이전트 공식 지원[42]. 원격 PowerShell 접속 특이 장애 **미확인**(표준 SSH 호환 예상(추정)) | 표준 OpenSSH 클라이언트 기반이라 접속 호환성 우수[1]. 단, PowerShell 측 UTF-8 미설정 시 한글 깨짐, `>>`로 공개키 추가 시 UTF-16 BOM 트랩 주의[35][43] | 기본 접속 가능[1]. 단, `TERM`(`xterm-256color` vs `screen`) 불일치 시 표시 오류 가능[44] | Windows 서버 지원 표방 및 v1.0.x 호환성 개선[12][28]. 본래 Linux `/proc` 모니터링 중심 앱(추정) | Linux/Unix/Windows 서버 지원 표방[30]. PowerShell 특화 이슈 **미확인** | 접속 직후 PowerShell 명령으로 멀티플렉서 탐지하므로 **`DefaultShell` PowerShell 설정 필수**(WSL 권장)[45]. 일반 PowerShell 단독 세션 UX 세부 **미확인** |
| **터미널 품질: TUI (색 / 마우스)** | `xterm 4.0.0` 기반 `xterm-256color`, 물리 마우스 지원[7][46]. 풀스크린 TUI 마우스 트래킹 세부 **미확인** | `xterm-256color`, 256색·TUI 마우스(`tmux`, `htop`, `lazygit`) 완전 지원[47] | 256색 지원. **xterm 마우스 트래킹/스크롤 휠 미지원(Issue #2351)** 및 Android 14+ 터치패드 2핑거 스크롤 미작동(#2549)[48][49] | `xterm.dart` 기반 `xterm-256color`, `tmux` 등 TUI 호환 우수[12][50] | 컬러 터미널 지원[30]. TUI 마우스 트래킹 **미확인** | `tmux`·`Zellij`·`herdr` 연동 특화[32]. 순수 PowerShell TUI 세부 **미확인** |
| **터미널 품질: 특수키 (Ctrl, Esc, Tab, 화살표)** | 소프트웨어 키보드 위 전용 유틸리티 키 행(`Esc, Tab, Ctrl, Alt, 화살표`) 제공[7] | `~/.termux/termux.properties`의 `extra-keys`로 자유롭게 커스터마이징 가능[51] | 가상 특수키 바 제공(`Special keys always visible` 설정 가능)[52] | 전용 특수키 바 제공(추정: `xterm.dart` UI 및 모바일 단축키)(세부 **미확인**) | **미확인** | 모바일 UI 툴바 및 단축키 제공[32] |
| **터미널 품질: 한글 입력·표시** | UTF-8/이모지/CJK(한·중·일) 렌더링 및 UI 한국어 지원[7][53], SwiftKey용 Incognito 토글 제공[21]. 삼성 키보드 조합 세부 동작 **미확인** | UTF-8 출력 가능(PowerShell 측 UTF-8 설정 필요)[43]. **안드로이드 IME 조합 중 글자(초·중·종성) 씹힘·분리 이슈 보고**(Hacker's Keyboard 우회 권장)[51] | **한글 IME 조합 입력 시 조합 중 글자 조용히 드롭됨(Issue #2352)**(`commitText()`만 전달, `setComposingText()` 미연결)[54] | `xterm.dart` 특성상 **한글 등 조합형 IME 입력 시 글자 밀림·줄바꿈 오작동 보고**(`KeyboardType` `visiblePassword` 우회 시 한글 직접 입력 제약)[55] | 앱 UI 한국어 지원[56]. 터미널 한글 IME 조합 입력 품질 **미확인** | **미확인** |
| **터미널 품질: 세션 유지 (백그라운드 / 재연결)** | Android **포그라운드 서비스**로 백그라운드 세션·포트포워딩 유지[34]. 자동 재연결 여부 **미확인** | 알림창 `Acquire wakelock` + Android 배터리 제한 해제 필수[57]. 자체 자동 재연결 없음(`ServerAliveInterval` / `mosh` 필요) | 설정 `Persist connections` + 알림 권한 허용(포그라운드 서비스) + 배터리 최적화 예외 필요[58] | 백그라운드 모니터링·알림은 서버 측 `ServerBox Monitor` 에이전트 방식[28]. 일반 SSH 백그라운드 유지 세부 **미확인** | 백그라운드 로컬 포트포워딩 지원[31]. 터미널 백그라운드 유지 세부 **미확인** | Mosh UDP 연결 유지 및 자동 재연결은 **Pro 전용**(무료는 표준 SSH만 제공)[17][18] |
| **폰 / 태블릿(폴더블) / 외부 키보드** | **500dp 이상 폴더블(Z Fold 펼친 화면)·태블릿에서 분할 터미널 무료 지원**[34][59]. 하드웨어 키보드/마우스 지원(이중 입력 버그 수정 이력)[21][46] | v0.119 베타에서 **태블릿·폴더블 포커스/전체화면 회전 버그 수정**, 디스플레이별 글꼴 배율 지원. 하드웨어 키보드 완벽 지원[24][47] | 물리 키보드 `Ctrl/Tab/Esc/화살표` 동작[60]. 단, Android 14+ 터치패드 2핑거 스크롤 미작동(#2549)[49] | Flutter 반응형 UI. 폴더블 전용 분할·외부 키보드 세부 **미확인** | **미확인** | 태블릿 UI 및 하드웨어 키보드 단축키(`⌘K`, `⌘1-9` 등) 지원[61][62] |

---

## 3. 후보별 핵심 평가 요약

### 1) ZestSSH
- Termius와 가장 닮은 모던 UX에 **구독 모델이 없는**("no subscription, ever") 크로스플랫폼 SSH 클라이언트다[8].
- 앱 안에서 Ed25519 키를 생성하고 길게 눌러 `Share public key`로 바로 공유할 수 있어[34], 위키 5절의 Taildrop 공개키 전달 흐름과 딱 맞는다.
- Z Fold 5 펼친 화면과 태블릿에서 **무료로 분할 터미널(Split terminal)**을 쓸 수 있고[34][59], 포그라운드 서비스로 백그라운드 세션을 유지하며[34], CJK(한·중·일) UTF-8 렌더링과 유틸리티 키 행을 갖춘 점이 강점이다[7].
- **유일한 주의점**: 무료 티어에 저장 세션/연결 2개 제한이 걸리는 경우(플랫폼·버전에 따라 2세션 제한 또는 무제한 안내가 혼재함[7][8]) 현재 데스크탑·노트북 2대는 무료 범위 안이지만, 호스트를 늘리거나 테마·위젯·단축키를 풀려면 약 **$9.99 일회 결제(Pro)**가 필요하다[7]. 클로즈드 소스 상용 앱이다.

### 2) Termux + OpenSSH
- 100% 무료·오픈소스이며[10], 위키 7절에도 수록된 정석 조합이다[1].
- `pkg install openssh`로 PC와 완전히 동일한 `ssh-keygen -t ed25519`와 `~/.ssh/config`(`IdentitiesOnly yes`, `ServerAliveInterval 30`)를 그대로 쓸 수 있다[1][35].
- TUI(256색, 마우스, `tmux`)와 태블릿·폴더블 대응(v0.119 베타 개선), 하드웨어 키보드 호환성이 매우 뛰어나다[24][47].
- **주의점**: 반드시 Google Play 스토어가 아닌 **GitHub Releases 또는 F-Droid**에서 받아야 하며[22], 안드로이드 기본 IME(삼성 키보드·Gboard)로 한글을 직접 칠 때 자모가 분리·누락되는 문제가 알려져 있어 Hacker's Keyboard 등을 병행하거나 영문 명령어 위주로 써야 한다[51].

### 3) ConnectBot (v1.11.1)
- 2026년 10월 v1.11.1이 나올 만큼 여전히 활발한 정통 무료·오픈소스 앱이다[11][26]. 과거 발목을 잡던 Ed25519 공개키 복사·인증 버그도 v1.10.9·v1.11.x에서 대폭 수정되었다[37][38].
- **추천 순위에서 밀린 이유**: GitHub 이슈에 **한글 IME 조합 입력 드롭(#2352)**[54]과 **xterm 마우스 트래킹 미지원(#2351)**[48], Android 14+ 터치패드 2핑거 스크롤 미작동(#2549)[49]이 보고되어 있어, 한국어 환경과 TUI·태블릿 사용성이 중요한 이 사용자에게 1·2순위로 권하기 어렵다. 영문 CLI와 물리 키보드 위주라면 가벼운 무료 대안으로 쓸 수 있다.

### 4) ServerBox
- Flutter + Rust 기반의 뛰어난 오픈소스 서버 대시보드 앱이다[12]. 다만 주 목적이 Linux 서버 상태 모니터링(`/proc` 지표·Docker 관리)이고, **2026년 기준 Play 스토어 공식 배포 여부가 불확실**하며(GitHub/F-Droid 중심)[27], `xterm.dart` 기반 **한글 IME 조합 입력 문제**[55]와 키 포맷 관련 이슈(`srvbox_pk_v1`)[40] 이력이 있어 단순 Windows PowerShell SSH 클라이언트 용도로는 과하고 번거롭다.

### 5) Admin Hands
- Play 스토어에서 v3.5로 활발히 업데이트되고 광고·트래킹이 없으며 Ed25519 키 생성·UI 한국어를 지원하는 점은 좋으나[14][15][30][31][56], 무료 호스트 제한(5개) 및 인앱 결제 범위[16], 공개키 내보내기 방식, TUI 마우스·한글 IME 입력 품질 등 핵심 세부 정보가 공개 문서로 확인되지 않아 우선 추천에서 제외했다.

### 6) Moshi (`getmoshi.app`)
- `herdr`·`tmux`·`Zellij`와 Claude Code 등 AI 코딩 에이전트를 모바일로 다루는 데 특화된 최신 앱으로[32], 무료 티어(호스트 2개)에서도 Ed25519 키 생성·생체 잠금을 제공한다[17][18][41].
- 다만 Windows 환경에서는 WSL 사용을 권장하며(네이티브 Windows 접속 시에도 멀티플렉서 탐지를 실행하므로 `DefaultShell` PowerShell 설정 필요)[45], 핵심 장점인 Mosh 연결 유지와 멀티플렉서 자동 재연결이 **Pro 구독($9.99/월 또는 $89.99/년, 평생 $199)**에 묶여 있어[17][18], 일반적인 Windows PowerShell 원격 터미널 대체 목적에는 과금 부담이 크다. (단, 추후 WSL/Linux 기반 `herdr` 원격 조종을 모바일에서 하고 싶다면 별도 검토 가치가 있다.)

---

## 4. 최종 추천

### 1순위: **ZestSSH**
- **선정 이유**:
  1. **Termius와 가장 매끄러운 대체**: Play 스토어에서 바로 설치할 수 있고, 앱 내 Ed25519 키 생성 + `Share public key`[34]로 기존 위키 5절의 Taildrop 공개키 등록 절차[1]를 거의 그대로 유지할 수 있다.
  2. **구독 없는 비용 구조**: 계정 가입 없이 무료로 시작하며(데스크탑·노트북 2대 구성에 적합(추정)), 기능 제한 없이 쓰고 싶어도 월 구독이 아닌 **$9.99 일회 결제(Pro)**로 끝난다[7][8].
  3. **폴더블·태블릿·한글 출력 친화적**: Z Fold 5 펼친 화면과 태블릿에서 분할 터미널을 무료 지원하고[34][59], `xterm 4.0.0` CJK/UTF-8 렌더링과 유틸리티 키 행·포그라운드 세션 유지를 모두 제공한다[7][34].

### 2순위: **Termux (+ OpenSSH)**
- **선정 이유**:
  1. **완전 무료·오픈소스 대안**: 비용을 전혀 들이지 않고 구독·제한 걱정 없이 오래 쓰려면 가장 검증된 선택이다[1][10].
  2. **최상위 터미널 품질·확장성**: PC와 동일한 OpenSSH 클라이언트(`~/.ssh/config`)로 Windows OpenSSH에 가장 안정적으로 붙고, TUI 256색·마우스·외부 키보드·태블릿/폴더블 대응(v0.119 베타)이 가장 완벽하다[24][47].
  3. **감수할 점**: Play 스토어가 아닌 F-Droid/GitHub에서 설치해야 하며[22], 폰/태블릿 소프트웨어 키보드 한글 조합 이슈는 Hacker's Keyboard 등을 쓰거나 영문 명령 위주로 타협해야 한다[51].

---

## 5. 1순위 앱(ZestSSH) 이전 절차 (폰 · 태블릿 각각 수행)

> 위키 4절 원칙대로 **기기마다 새 Ed25519 키를 생성**하고 개인키는 기기 밖으로 내보내지 않는다[1]. 아래는 데스크탑(`desktop`) 기준이며, 노트북(`laptop`)도 같은 방식으로 공개키 줄을 추가한다.

### 1단계: ZestSSH 설치 및 Ed25519 키 생성
1. 폰(또는 태블릿)의 Google Play 스토어에서 **ZestSSH**를 설치한다[9]. (계정 로그인 없이 로컬 모드로 시작 가능[7][20])
2. 앱 메뉴에서 **More → Keys**로 이동해 새 키를 만든다[34] (`Type: Ed25519`, 필요 시 암호문구 및 앱 생체 잠금 설정[7]).

### 2단계: 공개키 내보내기 및 데스크탑 등록
1. **More → Keys**에서 방금 만든 키를 **길게 눌러 `Share public key`**를 선택하고[34], 공유 시트에서 **Tailscale(Taildrop)**을 골라 데스크탑으로 보낸다[1].
2. 데스크탑의 **관리자 PowerShell**을 열어 받은 공개키(`Downloads` 폴더의 `.txt` 파일)를 `administrators_authorized_keys`에 등록하고 권한을 정리한다[1]:
   ```powershell
   $f = 'C:\ProgramData\ssh\administrators_authorized_keys'
   $k = (Get-Content "$env:USERPROFILE\Downloads\<받은 파일>.txt" -Raw).Trim()
   Add-Content -Encoding ascii -LiteralPath $f -Value "$k phone-zestssh"
   icacls.exe $f /inheritance:r /grant:r "*S-1-5-32-544:F" "*S-1-5-18:F"
   ```
   *(태블릿 키를 등록할 때는 끝의 주석을 `tab-zestssh`처럼 기기명으로 바꾼다. 일반 PowerShell의 `>>` 리다이렉션은 UTF-16 BOM이 붙어 OpenSSH가 무시하므로 반드시 `-Encoding ascii` 또는 UTF-8 no-BOM을 사용한다[1][43].)*

### 3단계: 호스트 등록 및 첫 접속 확인
1. ZestSSH에서 새 호스트를 추가한다(위키 7·8절 및 데스크탑 접속 정보 기준[1][2]):
   - **Host**: `desktop.taila5bb9e.ts.net` (또는 `100.87.213.81`)
   - **Port**: `22`
   - **Username**: `nampl`
   - **Auth / Key**: 1단계에서 만든 Ed25519 키 선택
2. 첫 접속 시 뜨는 **호스트 키 지문**을 데스크탑 관리자 PowerShell에서 `ssh-keygen -lf C:\ProgramData\ssh\ssh_host_ed25519_key.pub` 실행 결과와 대조한 뒤 수락한다[1].
3. PowerShell 프롬프트가 뜨면 한글 깨짐 방지를 위해 필요 시 PowerShell 프로필에 UTF-8 인코딩(`[Console]::InputEncoding = [Console]::OutputEncoding = [System.Text.Encoding]::UTF8`)을 확인·적용한다[43].
4. 안드로이드 설정에서 **ZestSSH**와 **Tailscale**의 배터리 사용량을 "제한 없음"으로 두고(Tailscale은 Always-on VPN), 백그라운드 전환 시 연결 유지를 확인한다[1][34].

---

## 6. 출처 (최종 확인일: 2026-11-16)

- [1] 로컬 위키 문서 — [ssh-tailscale-remote-terminal.md](file:///D:/projects/wiki-docs/wiki/dev/web/ssh-tailscale-remote-terminal.md) (확인일: 2026-11-16)
- [2] 로컬 위키 문서 — [README.md](file:///D:/projects/wiki-docs/inbox/261104-web-herdr-android-password-ssh/README.md) (확인일: 2026-11-16)
- [3] Termius Starter vs Pro 체험 만료 제한 — `termius.com` / `ctrlops.io` / `reddit.com` (Google 검색 확인: 2026-11-16)
- [4] JuiceSSH Play 스토어 제거 및 미유지보수 — `zestssh.com` / `termax.app` / `reddit.com` (Google 검색 확인: 2026-11-16)
- [5] JuiceSSH Android API 미준수 제거 언급 — `csdn.net` / `appbrain.com` (Google 검색 확인: 2026-11-16)
- [6] Termark 안드로이드 Beta/APK 배포 현황 — `termark.app` (Google 검색 확인: 2026-11-16)
- [7] ZestSSH 기능, 가격(Pro $9.99 / Cloud Sync $24.99), CJK·xterm 4.0.0·키 생성 — `apple.com` / `zestssh.com` (https://zestssh.com, 확인일: 2026-11-16)
- [8] ZestSSH 비구독 일회 결제 모델 및 자가 스토리지 E2EE 동기화 — `nicheloom.com` / `microsoft.com` (확인일: 2026-11-16)
- [9] ZestSSH Google Play 배포(Affluent Labs) — `appbrain.com` / `zestssh.com` (확인일: 2026-11-16)
- [10] Termux 오픈소스·무료·OpenSSH(`ssh-keygen -t ed25519`) — `termux.dev` / `coden-app.com` / `reddit.com` (확인일: 2026-11-16)
- [11] ConnectBot 공식 사이트 및 기능·오픈소스 — `connectbot.org` (https://github.com/connectbot/connectbot, 확인일: 2026-11-16)
- [12] ServerBox 공식 저장소·AGPLv3·`xterm.dart`·`dartssh2` — `github.com` (https://github.com/lollipopkit/flutter_server_box, 확인일: 2026-11-16)
- [13] ServerBox APKMirror/오픈 배포 — `apkmirror.com` / `sourceforge.net` (확인일: 2026-11-16)
- [14] Admin Hands Google Play 목록 — `play.google.com` (https://play.google.com/store/apps/details?id=com.arpaplus.adminhands, 확인일: 2026-11-16)
- [15] Admin Hands 광고/트래킹 SDK 없음 — `chrome-stats.com` (확인일: 2026-11-16)
- [16] Admin Hands 무료 호스트 5개 제한·인앱 결제 안내 — `apk.cafe` / `4pda.to` (확인일: 2026-11-16)
- [17] Moshi 가격표(Free 호스트 2개, Pro $9.99/월·$89.99/년·$199 평생) — `getmoshi.app` (https://getmoshi.app, 확인일: 2026-11-16)
- [18] Moshi 무료 티어 기능 및 생체인증·키 인증 — `getmoshi.app` (확인일: 2026-11-16)
- [19] ZestSSH 문서 및 Android 8.0+ 요구사항 — `zestssh.com` (`zestssh.com/docs`, 확인일: 2026-11-16)
- [20] ZestSSH v3.4 릴리즈 노트(FTPS, 탭 파일 에디터, 계정 없이 로컬 시작) — `apple.com` (확인일: 2026-11-16)
- [21] ZestSSH v3.4.2 등 2026 업데이트(물리 키보드 이중 입력 수정, Incognito 토글) — `zestssh.com` (확인일: 2026-11-16)
- [22] Termux Play Store vs F-Droid/GitHub 공식 권고 — `github.com` / `reddit.com` (https://github.com/termux/termux-app, 확인일: 2026-11-16)
- [23] Termux v0.118.1 및 v0.119.0-beta.2(2025-03-29) 릴리즈 이력 — `github.com` / `sourceforge.net` (확인일: 2026-11-16)
- [24] Termux v0.119 태블릿·폴더블 포커스·전체화면 회전·디스플레이별 배율 개선 — `apkmirror.com` / `termux.dev` / `github.com` (확인일: 2026-11-16)
- [25] ConnectBot Play 스토어(`google`) vs F-Droid/GitHub(`oss`) 플레이버 — `github.com` (https://github.com/connectbot/connectbot/releases, 확인일: 2026-11-16)
- [26] ConnectBot v1.11.1(2026년 10월) 릴리즈 노트 — `newreleases.io` / `openapk.net` / `github.com` (확인일: 2026-11-16)
- [27] ServerBox F-Droid·GitHub Releases(`v1.0.1925+`) 및 Play 스토어 미노출 상태 — `openapk.net` / `github.com` (https://github.com/lollipopkit/flutter_server_box/releases, 확인일: 2026-11-16)
- [28] ServerBox v1.0.x 2026 업데이트(키 생성·공개키 확인, Monitor v0.4.0, Windows 호환) — `github.com` / `4pda.to` / `apkmirror.com` (확인일: 2026-11-16)
- [29] ServerBox 공식 웹사이트 및 AGPLv3 CLA 안내 — `serverbox.lollipopkit.com` (https://serverbox.lollipopkit.com/, 확인일: 2026-11-16)
- [30] Admin Hands 지원 기능(SSH/SFTP, Ed25519 키 생성, Windows/Linux 서버) — `play.google.com` (`com.arpaplus.adminhands`, 확인일: 2026-11-16)
- [31] Admin Hands v3.5(2026년 후반) 업데이트(생체 잠금, 백그라운드 포트포워딩) — `cafebazaar.ir` / `apkpure.net` (확인일: 2026-11-16)
- [32] Moshi Google Play 등록 및 `tmux`/`Zellij`/`herdr` 연동 — `play.google.com` / `getmoshi.app` (확인일: 2026-11-16)
- [33] Moshi 클로즈드 소스 안내 — `getmoshi.app` (확인일: 2026-11-16)
- [34] ZestSSH 모바일 공개키 내보내기(`More > Keys > Share public key`), 포그라운드 서비스, 500dp+ 폴더블 분할 터미널 — `zestssh.com` (확인일: 2026-11-16)
- [35] Termux에서 Windows OpenSSH 접속 시 `ssh-keygen -t ed25519` 및 `authorized_keys` 권한·인코딩 가이드 — `zenn.dev` / `benheater.com` / `microsoft.com` (확인일: 2026-11-16)
- [36] ConnectBot `Manage Pubkeys` OpenSSH 키 지원 — `apkmirror.com` / `askubuntu.com` (확인일: 2026-11-16)
- [37] ConnectBot v1.10.9 Ed25519 공개키 복사 버그(#1508 등) 수정 — `apkmirror.com` (확인일: 2026-11-16)
- [38] ConnectBot v1.11.x Ed25519 255비트 생성 및 `sshlib`/Conscrypt 호환성 수정 — `apkmirror.com` / `github.com` (확인일: 2026-11-16)
- [39] ServerBox 개인키 등록·암호화 백업 — `github.com` / `apkmirror.com` (확인일: 2026-11-16)
- [40] ServerBox SSH 키 포맷 이슈(#7, #387, #499, #501) — `github.com` (https://github.com/lollipopkit/flutter_server_box/issues, 확인일: 2026-11-16)
- [41] Moshi 앱 내 Ed25519 키 생성·가져오기 — `getmoshi.app` (확인일: 2026-11-16)
- [42] ZestSSH Windows 로컬 PowerShell·WSL·OpenSSH 에이전트 지원 — `microsoft.com` (확인일: 2026-11-16)
- [43] Termux → Windows PowerShell SSH 접속 시 UTF-8 인코딩 및 `>>` UTF-16 BOM 트랩 — `github.com` / `zenn.dev` (확인일: 2026-11-16)
- [44] ConnectBot `TERM`(`xterm-256color` / `screen`) 관련 이슈 — `github.com` (확인일: 2026-11-16)
- [45] Moshi Windows 호스트 `HKLM\SOFTWARE\OpenSSH\DefaultShell` PowerShell 설정 필수 및 WSL 권장 — `getmoshi.app` (확인일: 2026-11-16)
- [46] ZestSSH 모바일 물리 마우스·하드웨어 키보드 지원 — `zestssh.com` (확인일: 2026-11-16)
- [47] Termux 태블릿 TUI(`tmux`, `htop`, `lazygit`) 및 하드웨어 키보드 지원 — `termux.dev` / `sourceforge.net` (확인일: 2026-11-16)
- [48] ConnectBot Issue #2351 (xterm mouse-tracking 미지원) — `github.com` (https://github.com/connectbot/connectbot/issues/2351, 확인일: 2026-11-16)
- [49] ConnectBot Issue #2549 (Android 14+ 터치패드 두 손가락 스크롤 미작동) — `github.com` (확인일: 2026-11-16)
- [50] ServerBox `xterm.dart` 256색 및 `tmux` TUI 호환 — `github.com` / `sourceforge.net` (확인일: 2026-11-16)
- [51] Termux 한글 IME 조합 입력 이슈 및 `Hacker's Keyboard`·`extra-keys` 우회 — `github.com` / `note.com` (확인일: 2026-11-16)
- [52] ConnectBot `Special keys always visible` 설정 및 가상 특수키 바 — `github.com` / `connectbot.org` (확인일: 2026-11-16)
- [53] ZestSSH UI 한국어 포함 다국어 지원 — `zestssh.com` (확인일: 2026-11-16)
- [54] ConnectBot Issue #2352 (한글 입력 조합 텍스트 드롭) — `github.com` (https://github.com/connectbot/connectbot/issues/2352, 확인일: 2026-11-16)
- [55] ServerBox `xterm.dart` 한글 IME 입력 이슈 및 `KeyboardType`(`visiblePassword`) 우회 제약 — `github.com` (확인일: 2026-11-16)
- [56] Admin Hands UI 한국어 지원 — `apkpure.net` (확인일: 2026-11-16)
- [57] Termux 백그라운드 세션 유지(`Acquire wakelock`, 배터리 최적화 예외) — `github.com` (확인일: 2026-11-16)
- [58] ConnectBot `Persist connections` 및 알림 권한 필수 안내 — `github.com` / `connectbot.org` (확인일: 2026-11-16)
- [59] ZestSSH Split terminal 메뉴 안내 — `zestssh.com` (확인일: 2026-11-16)
- [60] ConnectBot 물리 키보드 `Ctrl/Tab/Esc/화살표` 매핑 — `superuser.com` / `stackexchange.com` (확인일: 2026-11-16)
- [61] Moshi 태블릿 지원 — `getmoshi.app` (확인일: 2026-11-16)
- [62] Moshi 하드웨어 키보드 단축키 — `getmoshi.app` (확인일: 2026-11-16)
