# 안드로이드 SSH 터미널 대안 조사

확인일: 2026-11-16. 조사자: p-compare-2. 이 문서는 웹 문서 조사 결과이며 앱 설치·결제·실제 SSH 연결은 하지 않았다. **1순위 ConnectBot, 2순위 Termux**를 추천한다. 앱 기능은 문서상 확인과 실기 확인을 구분했고, 근거가 부족한 항목은 미확인으로 적었다.

## 기준이 되는 현재 구성

- Windows 11 기본 OpenSSH, TCP 22, 기본 셸 PowerShell, Tailscale을 통한 접속이다. 호스트는 `desktop.taila5bb9e.ts.net`, 사용자 `nampl`, 보조 주소 `100.87.213.81`이다. 관리자 그룹 계정이라 새 공개키는 `C:\ProgramData\ssh\administrators_authorized_keys`에 추가한다. 폰·태블릿마다 별도 Ed25519 키를 만든다. [로컬 위키 W1·W2]
- 비밀번호 인증은 차단되어 있다. herdr 안드로이드 앱 시험에서 비밀번호와 키 교환 설정을 임시로 바꿨지만 원상복구했다. 인증 이후 POSIX 명령으로 Bridge를 찾는 단계가 PowerShell에서 실패했다. 따라서 단순히 키 교환 알고리즘을 낮추거나 비밀번호를 다시 허용하는 것은 해결책으로 추천하지 않는다. Bridge의 Windows 미지원이라는 결론 자체는 W2의 추정이며 공식 지원표로 확정된 사실은 아니다. [W2]
- 위키에는 Termius Starter가 무료라고 되어 있지만 이번 카드는 무료 기간 종료로 사용하지 못한다는 사용자 상황을 전제로 한다. Termius 계정의 실제 제한 원인은 이 조사에서 확인하지 않았다. [W1·카드]

## 가격·배포·유지보수 비교

| 앱 | 가격·무료 범위 | 배포·최근 확인된 업데이트 | 오픈소스 |
| --- | --- | --- | --- |
| **ConnectBot** | 무료, 기부형. 구독·유료 기능 제한을 공식 페이지에서 찾지 못함. [C1] | Play·F-Droid·GitHub. Play 업데이트 2026-10-13, F-Droid **1.11.1-oss**, 추가일 2026-10-15. Android 7 이상. [C2·C3] | Apache-2.0. F-Droid 빌드는 Google Play Services 불필요. [C3] |
| **Termux + OpenSSH** | 무료. SSH·키 생성의 결제 제한 없음. [T1·T2·T3] | F-Droid·GitHub, Play도 존재. 공식 저장소가 표시하는 안정판 **0.118.3**, 릴리스 2025-05-22. F-Droid 페이지 상단에는 **0.119.0-beta.3**, 2025-05-29가 별도 표시됨. Play는 Android 11 이상용 실험 브랜치로 기능 누락·버그 경고. [T1·T2·T4] | GPLv3 앱, OpenSSH 패키지는 BSD. [T2·T3] |
| **SSHBorg** | 무료·기부형, 광고·추적·클라우드 없음. 무료 기능 차단 미기재. [B1] | F-Droid 확인. **1.21.0**, 프로젝트 릴리스 2026-10-31 / F-Droid 추가 2026-11-04. Android 9 이상. Play의 현재 배포 여부는 미확인. [B2·B3] | GPLv3. [B1·B3] |
| **Terminalko** | 현재 Play 본문은 **모든 기능 무료**, 구독·체험·잠금 없음이라고 명시. 다만 페이지에 인앱 결제 배지가 남아 있고 과거 구독자는 직접 해지하라는 안내가 있음. 한국 계정 결제 화면은 미확인. [K1] | Play 확인, 업데이트 2026-10-12. F-Droid는 미확인. [K1] | 터미널·SSH 라이브러리는 Apache-2.0. **앱 전체 소스 공개 여부는 미확인**이며 라이브러리 공개만으로 앱 전체를 오픈소스라 판단하지 않음. [K1] |

릴리스 날짜와 스토어 추가일은 서로 다른 날짜다. 검색 인덱스에는 SSHBorg 1.15.1이나 Terminalko 유료 Pro 설명도 남아 있었다. 표는 직접 연 현재 배포 페이지 본문을 우선했다. [B3·K1]

## 키 관리·Windows 적합성

| 앱 | 앱 내부 Ed25519 생성 | 공개키 내보내기·기존 키 가져오기 | Windows OpenSSH + PowerShell |
| --- | --- | --- | --- |
| **ConnectBot** | 가능. Ed25519는 소프트웨어 키이며 암호문구 사용 가능. 생체인식 Keystore 키와는 별개. [C4·C5] | 키 관리자에서 공개키 복사 경로 사용. 기존 개인키 가져오기 지원, Ed25519 PKCS#8/PEM 가져오기와 클립보드 가져오기를 변경 이력에서 확인. [C5·C6·C8] | 일반 SSH 셸 연결이므로 현재 구성에 맞을 것으로 판단 **(추정)**. 이 데스크탑 실기 및 Windows 전용 호환성 보증은 미확인. 예전 Ed25519 기기별 오류 수정 이력이 있으므로 최신 안정판 사용. [C4·C8] |
| **Termux** | 앱 안의 로컬 셸에서 OpenSSH `ssh-keygen -t ed25519`로 가능. GUI 키 관리자는 아님. [T3·T5] | `.pub` 파일을 표시·복사하거나 파일로 전달. 기존 OpenSSH 개인키 파일을 앱 내부 SSH 디렉터리에 두는 방식. 키 파일 읽기·생성 기능은 OpenSSH 근거, Android 파일 선택 UI의 세부 절차는 미확인. [T3·T5] | OpenSSH 클라이언트의 대화형 SSH로 PowerShell에 접속하는 구성이므로 적합할 것으로 판단 **(추정)**. Linux 전용 `ssh-copy-id` 명령을 Windows 관리자 계정에 그대로 쓰지 않음. [T3·W1] |
| **SSHBorg** | 가능. Ed25519·ECDSA·RSA. [B1·B4] | 키 화면에서 공개키 복사 가능, 기존 키 가져오기 지원. `.ppk`와 일부 암호화 컨테이너는 제약 있음. 개인키 내보내기는 의도적으로 미지원. [B4] | 일반 SSH 로그인 자체의 Windows 실기는 미확인. 명령 추천은 Bash·Zsh·Fish 이력 파일 기반이므로 PowerShell 이력 추천은 기대하기 어려움 **(추정)**. Bridge 설치가 필요하다는 설명은 없음. [B4] |
| **Terminalko** | 폰에서 Ed25519 생성 가능. [K1] | 개인키 붙여넣기 가능. **공개키 복사·공유 UI는 미확인**. 따라서 카드의 필수 조건을 전부 확인한 후보는 아님. [K1] | 공식 설명 대상이 Linux/Unix이며 Windows 실기·관련 알려진 문제 모두 미확인. 자동 tmux 복귀 설명을 Windows 기능으로 해석하지 않음. [K1·W1] |

## 터미널·입력·기기·연결 유지

| 앱 | TUI·색·마우스 | 특수키·한글 | 폰·태블릿·폴더블·외부 키보드 | 백그라운드·재연결 |
| --- | --- | --- | --- | --- |
| **ConnectBot** | 색·글꼴 프로필과 터미널 마우스·휠 지원 이력. 실제 herdr TUI 화면·마우스 동작은 미확인. [C4·C7] | Ctrl·Esc·화살표 도구막대, 키 사용자화. Tab 실기 미확인. **1.11.0에 Hangul IME·백스페이스·넓은 문자 수정 명시**, 정상 한글 조합을 모든 키보드에서 보증하는 뜻은 아님. [C4·C6] | 폰·태블릿 사용 명시, 과거 큰 화면 탭 지원. 외부 키보드의 비미국 배열·dead key 수정 이력. 최신 UI의 폴더블 접기/펴기·멀티윈도우 실기 미확인. [C3·C4] | 백그라운드 연결 유지와 VPN 감지·재연결 개선 이력. OS 강제 종료·LTE 전환에서 무손실 보증 없음. Mosh는 서버 구성 필요. [C2·C3·C4] |
| **Termux** | 강력한 터미널과 vim 사용은 배포 설명에서 확인. 실제 원격 TUI 색·마우스 동작은 미확인. [T2] | extra-keys와 Ctrl·Alt·Shift 잠금 지원 이력. Esc·Tab·화살표의 현재 기본 배열은 공식 위키 열람 제한으로 미확인. **한글 조합·표시 실기 미확인**. [T6] | 터미널 앱으로 사용 가능하나 폴더블 대응·외부 키보드 한영 전환 실기 미확인. | 포그라운드 서비스·절전 방지 권한 있음. Android 12 이상 프로세스 종료 관련 공식 경고가 있어 장시간 세션 생존 보증 불가. SSH 재접속은 가능하나 원격 PowerShell 복구는 별개. [T1·T2·W1] |
| **SSHBorg** | VT100/xterm·UTF-8. 마우스 보고와 복잡한 TUI 재현은 미확인. [B1] | Ctrl·Alt·Esc·Tab·화살표·F키 도구막대, 자연어용 word mode. UTF-8은 한글 조합 성공의 근거가 아니므로 한글 입력·표시는 실기 미확인. [B4] | 좁은 화면용 키 배열, 화면 회전 잘림 수정. USB/Bluetooth 물리 키보드는 Android TV 문서에서 확인되나 폰·태블릿·폴더블별 품질은 미확인. [B2·B4] | 1.19.0에 백그라운드 장시간 중단 수정. Mosh 지원은 서버 필요. PowerShell 세션 복구는 미확인. [B2·B3] |
| **Terminalko** | xterm-256color, vim 등 TUI·회전 시 크기 변경을 개발자가 명시. 마우스는 미확인. [K1] | Ctrl·Alt·Esc·Tab·화살표·F1~F10. 한글 표시·조합은 미확인. UI 번역 목록에 한국어 없음은 한글 입력 불가라는 뜻이 아님. [K1] | 폰 회전 대응 명시, 안드로이드 태블릿·폴더블·외부 키보드는 미확인. [K1] | 자동 재연결·tmux 복귀 명시. 백그라운드 생존 보증과 Windows 복원은 미확인. [K1] |

**공통 제한:** 현재 Windows sshd + PowerShell에서 앱의 keepalive나 자동 재연결은 끊긴 원격 명령을 복원한다는 뜻이 아니다. Mosh 기능은 `mosh-server`, tmux 복귀 기능은 원격 tmux가 필요하다. 현재 구성에 이 서버 구성요소가 있다는 근거가 없으므로 앱 교체만으로 세션 영속성이 해결된다고 추천하지 않는다. [W1·C2·B3·K1]

Windows 전용 문제 검색에서 충분한 재현 조건을 갖춘 공통 결함을 확보하지 못했다. 이를 '알려진 문제 없음' 또는 '완벽 호환'으로 바꾸어 적지 않았다. 실제 승인 전에는 폰과 태블릿 각각에서 한글 조합, Ctrl-C/Ctrl-R/Esc/Tab/화살표, TUI 색·마우스, 회전·접기, 화면 잠금 후 복귀, Wi-Fi→LTE 전환을 확인할 필요가 있다. 이는 후속 검증 제안이며 이번에 수행한 시험이 아니다.

## 추천과 이유

**1순위: ConnectBot 최신 안정판.** 무료 전용 SSH 앱이고 앱 내부 Ed25519 생성·키 가져오기, 저장 호스트, 특수키를 갖췄다. 2026년 10월 배포와 한글 IME 수정 근거까지 확인되어 이 사용자에게 가장 작은 이전 비용으로 맞는다. F-Droid 1.11.1-oss는 Google Play Services에 의존하지 않는 배포 경로다. 다만 한글·외부 키보드의 실제 품질, PowerShell TUI, 장시간 백그라운드는 검증이 남는다. 추천은 문서 기반 판단이다. [C1~C6]

**2순위: Termux의 OpenSSH.** 사용자는 Python/C++와 명령행을 사용하므로 GUI 키 관리자 대신 표준 `ssh-keygen`·`ssh`를 쓰는 부담을 감당할 가능성이 높다 **(추정)**. 기존 PC SSH 방식과 가까우며 새 유료 계정이 필요 없다. 앱·패키지 설치와 파일 설정이 필요해 ConnectBot보다 초기 작업이 많다. 공식 경고에 따라 F-Droid/GitHub 배포를 우선하고 Play 실험판을 기본 추천하지 않는다. [T1~T5]

SSHBorg는 무료 GUI 대안으로 유망하지만 PowerShell 이력 기능과 한글 실기 근거가 부족해 두 후보 뒤에 둔다. Terminalko는 현재 무료 설명이 확인되어 후보에는 포함하되 공개키 내보내기와 Windows 검증이 미확인이어서 우선 추천하지 않는다. [B4·K1]

## ConnectBot으로 옮기는 절차

아래는 실행한 작업이 아니라 현재 위키 설정에 맞춘 이전 안내다. 메뉴명은 설치 버전에 따라 달라질 수 있다. [W1·C5·C8]

1. 폰·태블릿 각각 공식 경로의 ConnectBot을 설치한다. 최신 안정판을 확인하고 같은 tailnet의 Tailscale 연결을 켠다. [C2·C3·W1]
2. ConnectBot 키 관리자(기존 이름 Manage Pubkeys/Public Keys)에서 새 키를 생성한다. 알고리즘은 **Ed25519**, 이름은 기기별로 구분하고 암호문구를 설정한다. Ed25519에 RSA/EC용 Fingerprint protection을 적용할 수 있다고 가정하지 않는다. [C4·C5]
3. 그 키의 **공개키 한 줄**을 복사한다. PC로 공개키만 전달한다. Taildrop은 기존 위키에서 확인된 전달 경로이나 ConnectBot 자체의 공유 버튼·Taildrop 연동 UI는 미확인이다. 필요하면 공개키 텍스트 파일을 통해 전달한다. 개인키를 전달하지 않는다. [W1·C5]
4. 데스크탑의 관리자 PowerShell에서 기존 `C:\ProgramData\ssh\administrators_authorized_keys` 내용을 유지한 채 새 공개키를 새 줄로 추가한다. `phone-connectbot`·`tablet-connectbot`처럼 주석으로 구분하고, 위키 5절의 Administrators·SYSTEM 권한 기준을 유지한다. 비밀번호 인증을 다시 켤 필요는 없다. [W1·W2]
5. Host List의 `+`에서 SSH 호스트를 저장한다. Quick connect 값은 `nampl@desktop.taila5bb9e.ts.net:22`, 해당 호스트의 키로 새 Ed25519 키를 선택한다. 첫 접속 지문은 데스크탑의 호스트 키 지문과 대조한다. [C8·W1·W2]
6. `whoami`로 계정·PowerShell을 확인하고 한글·특수키·TUI·LTE를 시험한다. 새 연결 성공을 확인하기 전에는 기존 Termius 키를 지우지 않는다. 두 기기는 각각 별도 키로 반복한다. [C8·W1]

## 출처

아래 모든 웹 URL 확인일은 **2026-11-16**이다. 기능 표의 문서 근거는 개발자·배포자 공식 페이지이며 사용자 리뷰의 호불호는 추천 근거로 사용하지 않았다.

- W1: `D:\projects\wiki-docs\wiki\dev\web\ssh-tailscale-remote-terminal.md` (로컬 읽기, 문서 updated 2026-11-02).
- W2: `D:\projects\wiki-docs\inbox\261104-web-herdr-android-password-ssh\README.md` (로컬 읽기, 2026-11-04 실험 기록).
- C1: [개발자·무료 유지·기부](https://connectbot.org/about/)
- C2: [Google Play 배포·업데이트](https://play.google.com/store/apps/details?id=org.connectbot)
- C3: [F-Droid 버전·라이선스·지원 OS](https://f-droid.org/packages/org.connectbot/)
- C4: [공식 변경 이력](https://github.com/connectbot/connectbot/blob/main/CHANGELOG.md)
- C5: [키 알고리즘·암호문구·Keystore 구분](https://connectbot.org/guides/ssh-keys/)
- C6: [특수키 도구막대](https://connectbot.org/guides/special-keys/)
- C7: [색·폰트 프로필](https://connectbot.org/guides/terminal-appearance/)
- C8: [호스트·키 선택·지문 확인](https://connectbot.org/guides/add-host/)
- T1: [Termux 공식 저장소·배포 경로·Play 실험판·Android 제한](https://github.com/termux/termux-app)
- T2: [Termux F-Droid](https://f-droid.org/en/packages/com.termux/)
- T3: [공식 OpenSSH 패키지](https://github.com/termux/termux-packages/blob/master/packages/openssh/build.sh)
- T4: [안정판 0.118.3 릴리스](https://github.com/termux/termux-app/releases/tag/v0.118.3)
- T5: [OpenSSH ssh-keygen 공식 매뉴얼](https://man.openbsd.org/ssh-keygen)
- T6: [Termux 공식 릴리스 변경 이력·extra-keys](https://github.com/termux/termux-app/releases)
- B1: [SSHBorg 공식 소개·가격](https://sshborg.com/)
- B2: [SSHBorg 변경 이력](https://sshborg.com/changelog.html)
- B3: [SSHBorg F-Droid 현재 배포](https://f-droid.org/packages/com.sshborg/)
- B4: [SSHBorg 키·키보드·서버 의존 기능 안내](https://sshborg.com/docs.html)
- K1: [Terminalko Google Play 개발자 설명](https://play.google.com/store/apps/details?id=com.pixelsorter.terminalko)

## 확인 한계

Termux 공식 위키의 Touch Keyboard·Remote Access 페이지는 자동 접근 거부 화면이 나와 내용을 확인하지 못했다. 접근 제한을 우회하지 않았다. ConnectBot 릴리스 목록도 한 번 fetch timeout이 났으며 버전은 F-Droid 페이지로 확인했다. 브라우저에서 직접 확인한 일부 본문은 같은 폴더의 `browser-evidence.json`에 보관했다. 검색 도구의 캐시와 실제 페이지가 다를 수 있어 '현재 최신'을 모든 채널 전체의 최종 버전으로 보증하지 않는다.

앱 설치, 계정 가입, 결제, sshd 변경, 실제 폰·태블릿·폴더블·외부 키보드 검증은 하지 않았다. 남은 위험은 기기별 한글 입력·TUI 표시·백그라운드 종료와 현재 Windows 세션의 복원 불가다.
