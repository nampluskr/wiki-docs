# Termius 대체 안드로이드 SSH 터미널 앱 조사 (p-compare-1)

- 작성: 2026-11-16, 웹 확인 날짜도 모두 2026-11-16
- 기준 구성(위키 `ssh-tailscale-remote-terminal.md`, inbox `261104-web-herdr-android-password-ssh`): Tailscale MagicDNS(`desktop.<tailnet>.ts.net`) → Windows 11 OpenSSH sshd:22, 기본 셸 PowerShell, **공개키만**(`PasswordAuthentication no`), 관리자 계정이라 `C:\ProgramData\ssh\administrators_authorized_keys`, 기기마다 Ed25519 키를 앱 안에서 생성
- 표기: [S1] 같은 번호는 아래 출처. 출처 없이 판단한 것은 "(추정)", 찾지 못한 것은 "미확인"

## 0. 먼저 확인할 점 — Termius 무료 플랜은 남아 있다

Termius 가격 페이지에는 무료 **Starter** 플랜이 있고 SSH·SFTP·로컬 볼트·포트포워딩이 들어 있다. Pro($10/월, 연 결제)에 묶인 것은 클라우드 볼트·기기 간 동기화·세션 로그·스니펫이다 [S1]. 끝난 것이 Pro 체험이라면 Starter로 계속 쓸 수 있을 수 있다(추정 — 앱 안에서 실제로 막힌 화면을 사용자에게 확인해야 한다). 기기마다 키를 따로 만드는 위키 구성은 동기화가 필요 없어서 Starter와 맞는다(추정).

## 1. 후보

| 앱 | 포함 이유 |
| --- | --- |
| ConnectBot | 위키 7절 후보. 무료·오픈소스, 2026년에 활발히 갱신 |
| Termux (+ `pkg install openssh`) | 위키 7절 후보. PC와 같은 OpenSSH |
| Server Box | 오픈소스 Flutter 앱. SSH 터미널 + 서버 상태 |
| JuiceSSH | 위키 7절에서 제외. 제외가 맞는지 확인 |
| (참고) TermAI, Moshi | 2026년 비교 글에 나오지만 출처가 전부 자사 글이라 비교표에는 넣지 않음 [S10][S11] |

## 2. 비교표

| 항목 | ConnectBot | Termux + OpenSSH | Server Box | JuiceSSH |
| --- | --- | --- | --- | --- |
| 가격 | 무료, 결제 없음 [S2][S6] | 무료 [S6] | 미확인(후원 링크만) [S8] | Pro 잠금 해제 구매 불가 [S6] |
| 배포 | Play(`org.connectbot`, 제목 "SSH & Mosh client") [S3], F-Droid 1.11.1-oss [S2] | F-Droid·GitHub가 안정판. Play 판은 Android 11+ 실험 브랜치로 기능 누락·버그가 있다고 공식 README가 밝힘 [S5] | F-Droid·GitHub [S8], Play 미확인 | Play에서 2025-12-11 내려감 [S6][S7] |
| 최근 업데이트 | v1.11.1, 2026-10-14(APKMirror)·2026-10-15(F-Droid) [S2][S4]. 2026-11-13에도 알파 빌드 [S9] | 안정판 v0.118.3, 2025-05-22 [S12]. Play 판 2026-06-21 [S13] | 미확인 | 마지막 릴리스 2021-02 [S7] |
| 오픈소스 | Apache-2.0 [S2] | GPLv3 [S12] | AGPL-3.0 [S8] | 미확인 |
| Ed25519 생성 | 됨. Manage Pubkeys → + → Ed25519 선택 [S14] | 됨. `ssh-keygen -t ed25519` [S6] | 미확인 | — |
| 공개키 내보내기 | 키 길게 누르기 → Copy public key [S14]. v1.10.9에서 Ed25519 복사·내보내기 버그 수정 [S15] | `~/.ssh/id_ed25519.pub` 파일 그대로 | 미확인 | — |
| 기존 키 가져오기 | v1.10.0부터 Ed25519 PKCS#8/PEM 가져오기 [S16]. 앱이 만든 개인키는 꺼낼 수 없음 [S15] | 됨(일반 OpenSSH) [S6] | 미확인 | — |
| Windows OpenSSH + PowerShell 문제 | 알려진 보고 못 찾음 [S17]. v1.11.1에서 Curve25519 키 교환 실패를 충돌 대신 오류로 표시 [S9] — herdr 앱의 curve25519 실패(inbox 6절)와 같은 증상이 날 수 있는지는 미확인 | 알려진 보고 못 찾음. PC와 같은 OpenSSH 클라이언트라 위키 8절 `config`를 그대로 쓸 수 있음(추정) | Windows 서버 상태 모니터링을 지원한다고 적음 [S8], 터미널은 미확인 | — |
| TUI(색·마우스) | v1.10.0에서 libvterm 기반 termlib로 교체, OSC 52·OSC 8 [S16]. v1.10.7에서 마우스·휠 지원 [S18] | 미확인(실제 Linux 터미널 에뮬레이터라 대체로 됨, 추정) | 미확인 | — |
| 특수키(Ctrl·Esc·Tab·화살표) | 화면 키 막대 미확인(옛 버전 자료뿐) [S18] | extra-keys 행, `~/.termux/termux.properties`로 바꿀 수 있음 [S6][S19] | 폰에 가상 키 [S8] | — |
| 한글 입력·표시 | 넓은 문자·결합 문자 처리 개선(v1.11.1) [S9]. 한글 IME 조합 보고 못 찾음 → 미확인 | 미확인 | 미확인(17개 언어 UI) [S8] | — |
| 세션 유지 | 앱 전환 중 연결 유지 [S2], 백그라운드 서비스 시작 거부 처리(v1.11.1) [S9]. 재연결은 미확인. Mosh는 Windows에 mosh-server가 없어 못 씀(추정) | 미확인. 끊기면 Windows 쪽 PowerShell도 끝남(위키 9절) | 미확인 | — |
| 폰·태블릿·폴더블 | 미확인(짧은 창에서 드롭다운 충돌 수정 [S9]) | 미확인 | 미확인 | — |
| 외부 키보드 | v1.10.7 도움말에 키보드 단축키 항목 [S18]. 옛 버전엔 BT 키보드 Ctrl·Esc 문제 보고 [S18], 지금은 미확인 | 미확인 | 미확인 | — |

## 3. 추천

### 1순위: ConnectBot

- Termius처럼 **앱 안에서 Ed25519 키를 만들고 공개키를 복사**할 수 있다 [S14][S15]. 위키 구성(기기마다 키, 공개키만 등록)을 바꿀 필요가 없다.
- 무료·결제 없음·오픈소스이고 [S2], Play 스토어에 있으며 2026-10에 안정판, 2026-11에도 개발 빌드가 나온다 [S4][S9]. 다시 유료 벽에 막힐 일이 없다.
- 2026년 판(v1.10~1.11)은 UI·터미널 엔진을 새로 짜고 마우스·OSC 52를 넣어 [S16][S18] "오래된 UI"라는 평 [S6]은 옛 버전 기준일 수 있다(추정).
- 남는 위험: 한글 IME 조합, 화면 특수키 막대, 태블릿·폴더블 화면, Windows 데스크탑의 curve25519 협상은 웹에서 확인하지 못했다. 설치 후 폰·탭에서 직접 확인해야 한다.

### 2순위: Termux + OpenSSH

- PC와 똑같은 `ssh-keygen`·`~/.ssh/config`·`ssh -v`를 써서 위키 4·8·11절을 그대로 따를 수 있고, extra-keys로 Ctrl·Esc·Tab·화살표를 둘 수 있다 [S6][S19].
- 2순위인 이유: 안정판은 F-Droid·GitHub로 받아야 하고 Play 판은 공식적으로 실험판이다 [S5]. 호스트 목록 같은 GUI가 없고, 공개키를 Taildrop으로 보내려면 파일 공유 단계가 하나 더 든다(추정).

### 고르지 않은 것

- JuiceSSH: Play에서 내려갔고 2021년 뒤로 갱신이 없다 [S6][S7]. 위키 판단과 같다.
- Server Box: 오픈소스지만 키 생성·한글·Play 배포를 확인하지 못했다 [S8].
- TermAI·Moshi: 유료 구독 중심이고 근거가 자사 글뿐이다 [S10][S11].
- Termius Starter: 0절대로 무료로 계속 쓸 수 있다면 가장 바꿀 게 적다. 사용자 확인이 먼저다.

## 4. ConnectBot으로 옮기는 절차 (폰·탭 각각)

메뉴 이름은 커뮤니티 문서 기준이라 버전에 따라 다를 수 있다 [S14].

1. **설치**: Play 스토어에서 `ConnectBot`(org.connectbot) 설치 [S3]. Tailscale은 그대로 Always-on VPN(위키 2절).
2. **키 생성**: ⋮ → Manage Pubkeys → + → 별명(`phone` 또는 `tab`) → 형식 **Ed25519** → 암호문구 → Load key on start 체크 → Generate → 화면을 문질러 난수 채우기 [S14].
3. **공개키 내보내기**: 새 키를 길게 누르기 → Copy public key [S14]. 메모 앱 등에 붙여 `.txt`로 저장한 뒤 Tailscale 공유(Taildrop)로 `desktop`에 보낸다(위키 5절 방식, 추정 — ConnectBot에서 바로 공유 메뉴가 있는지는 미확인).
4. **데스크탑 등록**(관리자 PowerShell, 위키 5절 그대로):
   ```powershell
   $f = 'C:\ProgramData\ssh\administrators_authorized_keys'
   $k = (Get-Content "$env:USERPROFILE\Downloads\<받은 파일>.txt" -Raw).Trim()
   Add-Content -Encoding ascii -LiteralPath $f -Value "$k phone"
   icacls.exe $f /inheritance:r /grant:r "*S-1-5-32-544:F" "*S-1-5-18:F"
   ```
   기존 Termius 키 줄은 ConnectBot 접속을 확인한 뒤 지운다(위키 10절 원칙).
5. **호스트 등록·접속**: 주 화면 주소칸에 `ssh` / `nampl@desktop.<tailnet>.ts.net:22`, 호스트 설정에서 위 키 지정. 첫 접속의 호스트 키 지문은 데스크탑 `ssh-keygen -lf C:\ProgramData\ssh\ssh_host_ed25519_key.pub`(관리자)와 대조(위키 8절).
6. **확인**: PowerShell 프롬프트, 한글 입력·표시, Ctrl+C·Esc·Tab·화살표, 화면 회전·폴더블 펼침, 백그라운드 후 복귀. 접속이 키 교환에서 끊기면 inbox 6절 2단계(curve25519)와 같은 원인인지 sshd 로그로 본다.

## 5. 출처 (모두 2026-11-16 확인)

- [S1] Termius 가격 — https://termius.com/pricing
- [S2] F-Droid ConnectBot — https://f-droid.org/en/packages/org.connectbot/
- [S3] Play 스토어 ConnectBot — https://play.google.com/store/apps/details?id=org.connectbot (제목만 확인, 본문은 불러오지 못함)
- [S4] APKMirror ConnectBot 업로드 목록 — https://www.apkmirror.com/apk/kenny-root/connectbot/
- [S5] Termux README(Google Play Version 절) — https://github.com/termux/termux-app
- [S6] Botmonster, Android SSH clients compared — https://botmonster.com/coding/android-ssh-client-termius-compared/
- [S7] Moshi, Best Android Terminal App for AI Coding Agents (2026-08-06, Moshi 자사 글) — https://getmoshi.app/articles/best-android-terminal-app-coding-agent
- [S8] Server Box GitHub — https://github.com/lollipopkit/flutter_server_box
- [S9] ConnectBot 릴리스(v1.11.1 노트, alpha-3324~3338) — https://github.com/connectbot/connectbot/releases , https://github.com/connectbot/connectbot/releases/latest
- [S10] TermAI 비교(자사 글) — https://termai-website.pages.dev/compare , https://termai.sh/blog/termius-alternatives
- [S11] [S7]과 같음
- [S12] Termux GitHub 릴리스 — https://github.com/termux/termux-app/releases , F-Droid https://f-droid.org/en/packages/com.termux/
- [S13] Termux Play 판 릴리스 — https://github.com/termux-play-store/termux-apps/releases , https://www.apkmirror.com/?p=14495920
- [S14] ConnectBot 키 만들기(커뮤니티 문서) — https://pop.ircnow.org/connectbot/keys/
- [S15] 검색 요약: v1.10.9 "Fixed Ed25519 key copying/exporting"(https://www.apkmirror.com/?p=14226302, 직접 열기는 403), 개인키 꺼낼 수 없음(ConnectBot 이슈, https://lightrun.com/answers/connectbot-connectbot-support-ed25519-openssh-standard-formats-)
- [S16] ConnectBot v1.10.0 릴리스 노트 — https://github.com/connectbot/connectbot/releases/tag/v1.10.0
- [S17] Windows OpenSSH·ConnectBot 관련 검색(직접 일치 없음) — https://github.com/PowerShell/Win32-OpenSSH/issues
- [S18] 검색 요약: v1.10.7 "Terminal mouse support including mouse wheel", "Keyboard shortcuts … in help menu"(https://www.apkmirror.com/?p=13734868), 옛 BT 키보드 문제(https://groups.google.com/g/connectbot-commits/c/le0r0_Ph3f4)
- [S19] Termux extra keys — https://mobile-coding-hub.github.io/termux/customisation/extra_keys
