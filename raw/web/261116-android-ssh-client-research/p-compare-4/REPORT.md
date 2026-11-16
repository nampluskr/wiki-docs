# Termius를 대신할 안드로이드 SSH 터미널 앱 조사 (R1-01)

- 조사·작성: p-compare-4 (herdr worker-4), 2026-11-16
- 기준 구성: 위키 `wiki/dev/web/ssh-tailscale-remote-terminal.md`(2026-11-02 갱신), `inbox/261104-web-herdr-android-password-ssh/README.md`(2026-11-04)
- 모든 가격·버전·업데이트 날짜는 2026-11-16에 확인한 값이다. 확인하지 못한 항목은 "미확인", 근거가 간접적인 항목은 "(추정)"으로 표시했다.

## 0. 결론 먼저

| 순위 | 앱 | 한 줄 이유 |
| --- | --- | --- |
| 1순위 | **ConnectBot** | 무료·오픈소스·인앱 결제 없음. Ed25519 생성→공개키 공유가 위키의 기존 절차와 똑같이 되고, Windows OpenSSH 사례가 가장 많으며 2026-10까지 활발히 업데이트된다. |
| 2순위 | **Termphin** | 완전 무료·GPL-3. 서버 에이전트가 **Windows x86_64도 지원**해서 "세션 유지"가 이 환경에서도 실제로 동작한다. 2026-09 출시라 사용자층이 얇고 한글 입력이 미확인인 점이 위험. |
| 3순위(조건부) | **Shellwave** | 폴더블·태블릿 UI가 가장 잘 만들어졌고 무료(선택적 팁). 다만 SSH 라이브러리가 sshj여서, herdr 앱이 겪은 curve25519 키교환 실패가 재발할 가능성이 있다(추정). |

Termius는 **Starter 무료 플랜이 아직 존재한다**(로컬 vault·SSH·SFTP·포트포워딩). "무료 기간 종료"는 Pro 체험 종료로 보이므로(추정), 급하면 Starter로 계속 쓸 수는 있다. 다만 동기화·세션 로그는 Pro 전용이다.

## 1. 조사 기준이 된 현재 구성

- 접속받는 쪽: Windows 11 기본 OpenSSH sshd, 포트 22, 기본 셸 PowerShell, MagicDNS `desktop.taila5bb9e.ts.net` / `100.87.213.81`, 사용자 `nampl`(관리자 그룹).
- 인증: **공개키 전용**(`PasswordAuthentication no`). 관리자 계정이라 `C:\ProgramData\ssh\administrators_authorized_keys`에 등록된 키만 통과한다.
- 그래서 새 앱의 필수 조건은 **① 앱 안에서 Ed25519 키를 만들고 ② 공개키를 꺼낼 수 있고 ③ 그 키로 공개키 인증을 할 수 있을 것**이다. 비밀번호만 넣는 앱(herdr 안드로이드 앱)은 이 구성에서 쓸 수 없다(위키 2026-11-04 시험).
- Mosh는 이 환경에서 사실상 못 쓴다. Windows에 `mosh-server`가 없다. Mosh를 내세우는 앱의 장점은 여기서는 사라진다.

## 2. 비교표 A — 가격·배포·키

| 앱 | 가격 / 무료 범위 | 배포·최근 업데이트 | 오픈소스 | Ed25519 생성 | 공개키 내보내기 | 기존 키 가져오기 |
| --- | --- | --- | --- | --- | --- | --- |
| ConnectBot | 무료, 인앱 결제 없음(기부) | Play 5M+·2026-10-14, F-Droid 1.11.1·2026-10-15 | Apache-2.0 | 가능(소프트웨어 키, 암호문구 선택) | 복사/공유 | 가능(PKCS#8/PEM, 클립보드 가져오기) |
| Termphin | 완전 무료 | Play 2026-09 출시, iOS 동시, GitHub 9 stars | GPL-3.0 | 가능(Ed25519 권장 표기) | 복사 | 가능(PEM) — 단 **개인키 내보내기는 불가** |
| Shellwave | 무료 + 선택적 일회 팁(잠금 없음) | Play 50+·2026-11-15, F-Droid 1.10.0·2026-11-04 | GPL-3.0 | 가능(Ed25519/ECDSA/RSA) | 복사, authorized_keys 자동 설치 흐름 | 가능 |
| SSHBorg | 무료(광고·추적 없음) | Play 1K+·2026-10-31, F-Droid 1.21.0·2026-11-04 | GPL-3.0 | 가능(Ed25519/ECDSA/RSA) | 미확인(공개키 표시는 있음) | 미확인 |
| Termux | 무료 | F-Droid 0.118.3 / 0.119.0-beta.3, **2025-05-22이 마지막** | GPL-3.0 | `ssh-keygen -t ed25519` | `cat ~/.ssh/id_ed25519.pub` | `~/.ssh` 복사 |
| Termius(기준) | Starter 무료(로컬 vault·SSH·SFTP·포트포워딩), Pro 월 $10(연 결제)·동기화 | Play 1M+·2026-11-04, IAP | 클로즈드 | 가능(Keystore) | 공유 | 미확인 |
| JuiceSSH | — | **Play 404(2025-12-11 내려감)**, 마지막 3.2.2·2021-02-04 | 클로즈드 | — | — | — |

## 3. 비교표 B — 이 환경(Windows OpenSSH + PowerShell)에서의 실사용 품질

| 앱 | Windows OpenSSH·PowerShell | TUI 표시(색·마우스) | 특수키(Ctrl/Esc/Tab/화살표) | 한글 입력·표시 | 세션 유지 | 폰·태블릿·폴더블·외부 키보드 |
| --- | --- | --- | --- | --- | --- | --- |
| ConnectBot | 막는 문제 없음. 과거 "빈 화면"(이슈 #1093)은 2025-12 터미널 재작성으로 종결. 단 호스트 편집에서 `Start shell session`이 꺼지면 빈 화면이 되므로 확인 필요 | 256색·트루컬러, 마우스(1.9부터 자체 termlib), 박스/이모지/결합문자 렌더 개선 | 전용 특수키 툴바(항상 표시 설정), Ctrl·Alt를 IME 대신 직접 전송하도록 선택 가능 | **1.11.0에서 한글 IME 수정**(이슈 #2352 종결, termlib 0.3.2). 다만 2026-11-11 Play 리뷰에 키보드 입력 불만이 남아 있어 실기기 확인 필요 | 포그라운드 서비스로 백그라운드 유지, 끊기면 재연결. Mosh는 Windows에서 불가 | 큰 화면 탭 UI, 외부 키보드(비US 배열 1.10.4). 폴더블 전용 대응은 문서 없음(미확인) |
| Termphin | 전용 문서는 없으나 세션 에이전트가 Windows x86_64를 지원한다고 명시. PowerShell 셸에서의 동작 세부는 미확인 | Flutter 직접 렌더(웹뷰 아님), htop/vim 박스 문자 정상 | 키 툴바(Esc·Tab·Ctrl·화살표) 배치 커스터마이즈 | 미확인 | **서버 에이전트로 세션 유지**(드롭 후 재부착), 예측 에코, 자동 재연결 | 스플릿 화면, 태블릿 스크린샷 있음. 폴더블 전용 대응 미확인 |
| Shellwave | 문서 없음(미확인). **sshj 기반**(README 토픽) → herdr 앱이 겪은 curve25519 KEX 실패 위험(추정) | 256색·트루컬러, wide 문자, alt screen, 스크롤백 검색 | 커스터마이즈 키 바(호스트별) | 미확인 | 자동 재연결, 선택적 tmux 세션(Windows에 tmux 없음) | **폴더블 우선 설계**(2-pane, 테이블탑 모드, 접고 펴도 세션 유지), 물리 키보드 감지·AltGr 전송 |
| SSHBorg | 미확인 | VT100/xterm, UTF-8, 다중 세션 | 있음(상세 미확인) | UI 언어 목록에 한국어 없음 → 미확인 | Mosh 지원(Windows 불가), 재연결 미확인 | 태블릿·TV 스크린샷 있음 |
| Termux | 일반 OpenSSH 클라이언트라 문제 없음. 대신 안드로이드 리눅스 환경(배터리·저장소 권한) 부담 | xterm 기반, 색·TUI 양호 | 볼륨키=Ctrl 등 자체 조합 | 미확인(IME 처리에 의존) | 앱이 살아 있는 동안만(wake lock 필요), 끊기면 셸 종료 | 태블릿 OK, 외부 키보드 OK |
| Termius(기준) | 위키에서 실제 접속 확인됨(Ed25519, 관리자 키 파일) | 양호(리뷰: 8.0.0에서 Claude Code 중복 출력 수정) | 가상 키보드·제스처 | 미확인 | 세션 유지 양호 | 폰·태블릿 확인됨(위키) |
| JuiceSSH | — | — | — | — | — | 설치 불가 |

## 4. 추천과 이유

### 1순위 — ConnectBot

1. **요구 조건을 그대로 만족한다.** 앱에서 Ed25519 키를 만들고(암호문구 선택, 소프트웨어 키) 공개키를 복사·공유할 수 있다. 위키 5절의 Taildrop 절차(앱 공개키 공유 → Tailscale → PC `Downloads`)를 Termius와 똑같이 쓸 수 있다.
2. **무료에 인앱 결제가 없다.** Play 목록에 인앱 결제 표기가 없고 기부로 운영된다. "무료 기간이 끝나 막히는" 일이 구조적으로 없다.
3. **유지보수가 살아 있다.** 2007년부터 한 개발자가 유지하고, 2026-10-14 Play / 2026-10-15 F-Droid에 1.11.1이 올라왔다. 1.10에서 UI·터미널을 전면 재작성했고 이후 1.11에서 Mosh, IME(Hangul), Kitty/iTerm2 인라인 이미지, 점프 호스트 등이 들어왔다.
4. **한글 입력이 공식적으로 고쳐졌다.** 2025년까지 한글 조합 문자가 유실되는 문제(#2352, 갤럭시 Z Fold 8·삼성 키보드)가 있었고, 2026-09-22 termlib 0.3.2로 종결됐다. 1.11.0 변경 내역에도 "Improve IME input handling for Hangul"이 있다.
5. **Windows OpenSSH 관련 알려진 차단 요인은 없다.** 유일하게 눈에 띄는 Windows 사례(#1093 "빈 터미널")는 2025-12에 "최신 베타에서 해결"로 닫혔고, 원인은 대개 호스트 설정의 `Start shell session`이 꺼진 것이었다.

**위험·주의**
- 2026-11-11 Play 리뷰에 "업데이트 후 키보드 입력이 어렵다, Ctrl-R이 안 된다"는 불만이 있다. 1.11.1에서 고쳐졌다는 답변이 달려 있지만, **설치 후 한글·Ctrl 조합을 직접 확인**해야 한다.
- SFTP가 없다(파일 전송이 필요하면 별도 앱). 이 사용자의 현재 용도(터미널 접속)에는 영향이 없다.
- Mosh는 Windows에 `mosh-server`가 없어 쓸 수 없다. 앱의 Mosh 기능은 이 환경에서 무의미하다.
- 폴더블 전용 레이아웃(2-pane 등)은 문서화돼 있지 않다.

### 2순위 — Termphin

1. **완전 무료 + GPL-3 + 2026년 오픈소스화.** 가격 리스크가 없다.
2. **"세션 유지"가 이 환경에서도 동작한다.** 서버에 자동 설치되는 에이전트가 Linux·macOS뿐 아니라 **Windows x86_64**도 지원한다고 문서에 명시돼 있다(파일은 `%LOCALAPPDATA%\termphin\sessions`). Windows에 tmux가 없어서 Termius·ConnectBot에서는 불가능한 "끊겨도 셸이 살아 있음"을 유일하게 제공한다.
3. 키 관리가 요구와 맞는다. Ed25519 생성 → 공개키 복사, 기존 개인키(PEM) 가져오기. 비밀번호 없이 쓸 수 있다.

**위험**
- 2026-09 Play 출시, GitHub 스타 9개. 사용자층·이슈 이력이 얇아 초기 버그를 혼자 맞을 수 있다.
- 개인키 내보내기가 안 된다(의도된 제한). 기기를 잃으면 새 키를 등록해야 한다.
- 한글 입력, 폴더블 레이아웃은 **미확인**. PowerShell 기본 셸에서 에이전트가 어떻게 붙는지도 문서에 없다.

### 3순위(조건부) — Shellwave

폴더블·태블릿 UI와 세션 재연결 설계가 가장 좋고 무료(팁 선택)다. 그러나 README 토픽이 sshj라서, 위키 2026-11-04 기록의 **"sshj 0.39가 curve25519 협상 후 끊김"** 문제가 재발할 수 있다(추정). 그때는 서버 `sshd_config`에 `KexAlgorithms -curve25519-sha256,curve25519-sha256@libssh.org`를 넣어야 하는데, 이 사용자는 그 변경을 이미 되돌려 놓았다. 그래서 2순위보다 아래에 둔다. 1·2순위가 모두 한글에서 실패할 때 시험할 후보로 남긴다.

### 검토 후 제외

- **JuiceSSH**: Play 페이지가 404(2025-12-11 내려감), 마지막 릴리스 2021-02-04. 위키 7절의 판단(설치 비권장)이 2026-11에도 유효하다.
- **Termux**: 조건은 만족하지만 앱 자체 마지막 릴리스가 2025-05-22이고, Play 버전은 구버전이라 공식 배포가 F-Droid/GitHub뿐이다. "터미널 앱 하나"를 원하는 용도에는 설정 부담이 크다. 명령행에 익숙하고 tmux·스크립트를 쓰고 싶을 때의 대안으로만 남긴다.
- **VibeTTY**: ConnectBot 포크(Apache-2.0)로 Kitty 키보드 프로토콜·가상 폭 등 매력적이지만 GitHub APK 배포만 하고 스타 1개다. 관찰 대상으로만 언급한다.
- **herdr 안드로이드 앱**: 비밀번호 입력칸만 있어 공개키 전용 구성에서 인증 단계에서 끊긴다(위키 2026-11-04, `preauth` 종료). 후보가 아니다.

## 5. 1순위(ConnectBot)로 옮기는 절차

전제: 데스크탑은 이미 `PasswordAuthentication no`, 관리자 계정(`nampl`)이며 `administrators_authorized_keys`를 쓴다. **Termius의 기존 키와 앱은 검증이 끝날 때까지 지우지 않는다.**

1. **설치** — Play 스토어에서 `ConnectBot`(개발자 Kenny Root). F-Droid를 쓰면 `org.connectbot`.
2. **키 만들기** — 앱의 키 목록 화면 → `+` → Generate. 이름은 기기별로 `fold-connectbot` / `tab-connectbot`, 종류 **Ed25519**, 원하면 암호문구 설정. 생성 후 **공개키 복사**(또는 공유).
3. **공개키 전달** — 공유 → Tailscale(Taildrop) → 데스크탑 `Downloads`. 위키 5절과 같은 방식이고, 도착 파일은 임의 이름 `.txt`, 주석 없는 한 줄일 수 있다.
4. **데스크탑 등록(관리자 PowerShell)**
   ```powershell
   $f = 'C:\ProgramData\ssh\administrators_authorized_keys'
   $k = (Get-Content "$env:USERPROFILE\Downloads\<받은 파일>.txt" -Raw).Trim()
   Add-Content -Encoding ascii -LiteralPath $f -Value "$k fold-connectbot"
   icacls.exe $f /inheritance:r /grant:r "*S-1-5-32-544:F" "*S-1-5-18:F"
   ```
5. **호스트 등록** — 새 호스트: 주소 `desktop.taila5bb9e.ts.net`(또는 `desktop`), 포트 `22`, 사용자 `nampl`, 인증에서 2단계에서 만든 키 선택. 저장 후 접속.
6. **첫 접속 지문 대조** — 데스크탑 관리자 PowerShell에서 `ssh-keygen -lf C:\ProgramData\ssh\ssh_host_ed25519_key.pub`. 앱에 뜬 지문과 같을 때만 수락. 다르면 접속하지 말고 원인 확인.
7. **설정 다듬기** — Settings → Keyboard → `Special keys always visible` 켜기(Ctrl·Esc·Tab·화살표 상시 노출). 한글은 삼성 키보드 2벌식으로 `안녕하세요 테스트`를 입력해 조합이 깨지지 않는지 확인. 빈 화면이면 호스트 편집에서 `Start shell session`이 켜져 있는지 확인(이슈 #1093의 단골 원인).
8. **정리** — 폰·태블릿 각각 위 2~6을 반복(기기마다 키를 따로). 며칠 써 보고 문제가 없으면 Termius 앱과 그 키 줄을 정리한다. Termius 키 줄은 `administrators_authorized_keys`에서 `-C` 이름으로 찾아 지운다.

## 6. 출처 (모두 2026-11-16 확인)

| 대상 | URL | 확인한 내용 |
| --- | --- | --- |
| ConnectBot 공식 | https://connectbot.org/ , https://connectbot.org/guides/ssh-keys/ , https://connectbot.org/guides/special-keys/ | 키 알고리즘별 저장 방식, 특수키 상시 표시 설정, v1.11.1 기준 문서 |
| ConnectBot 배포·버전 | https://play.google.com/store/apps/details?id=org.connectbot , https://f-droid.org/packages/org.connectbot/ | 2026-10-14 업데이트, 5M+, 인앱 결제 표기 없음, F-Droid 1.11.1(2026-10-15) |
| ConnectBot 변경 내역 | https://raw.githubusercontent.com/connectbot/connectbot/main/CHANGELOG.md | 1.11.0 한글 IME 수정, 1.11.1 키 내보내기·호스트키 관리, 1.10.4 외부 키보드 |
| ConnectBot 한글 이슈 | https://github.com/connectbot/connectbot/issues/2352 | 2026-08-30 보고 → 2026-09-22 termlib 0.3.2로 종결 |
| ConnectBot Windows 빈 화면 | https://github.com/connectbot/connectbot/issues/1093 | Windows OpenSSH 사례, 2025-12-15 "최신 베타에서 해결" |
| Termphin | https://termphin.dev/docs/connections/ssh-keys , https://termphin.dev/docs/sessions/remote-agent , https://github.com/Termphin/Termphin | Ed25519 생성·공개키 복사, 개인키 내보내기 불가, 에이전트 Windows x86_64 지원, 완전 무료·GPL-3 |
| Termphin 배포 | https://play.google.com/store/apps/details?id=dev.termphin.termphin | Play 등록, 인앱 결제 없음 |
| Shellwave | https://f-droid.org/en/packages/io.github.lordofpolls.shellwave , https://play.google.com/store/apps/details?id=io.github.lordofpolls.shellwave , https://github.com/LordOfPolls/Shellwave | 무료+선택 팁, GPLv3, Android 12+, 폴더블 2-pane, sshj 사용, 2026-11-15 업데이트, 50+ 설치 |
| SSHBorg | https://f-droid.org/en/packages/com.sshborg , https://play.google.com/store/apps/details?id=com.sshborg | 1.21.0(2026-11-04), Ed25519 생성, mosh, 지원 언어 목록 |
| Termux | https://f-droid.org/packages/com.termux/ , https://github.com/termux/termux-app/releases | 0.118.3 / 0.119.0-beta.3, 2025-05-22 마지막 |
| Termux Play | https://wiki.termux.com/wiki/Termux_Google_Play , https://github.com/termux/termux-app/discussions/4000 | Play 버전은 구버전, 공식 배포는 F-Droid/GitHub |
| Termius 가격 | https://termius.com/pricing | Starter 무료(로컬 vault·SSH·SFTP·포트포워딩), Pro 월 $10(연 결제), 동기화는 Pro |
| Termius 배포 | https://play.google.com/store/apps/details?id=com.server.auditor.ssh.client | 2026-11-04 업데이트, 1M+, IAP, Starter 무료 범위 설명 |
| JuiceSSH 종료 | https://play.google.com/store/apps/details?id=com.sonelli.juicessh (404), https://chenzong.hashnode.dev/juicessh-is-dead-what-to-move-to-and-how-to-migrate-safely , https://ericswpark.com/blog/2026/2026-06-03-juicessh | Play 404, 2025-12-11 내려감, 마지막 릴리스 2021-02-04 |
| VibeTTY | https://github.com/johnrobinsn/VibeTTY | ConnectBot 포크, Apache-2.0, GitHub APK만 배포 |
| 위키 기준 구성 | `wiki/dev/web/ssh-tailscale-remote-terminal.md`, `inbox/261104-web-herdr-android-password-ssh/README.md` | 공개키 전용, 관리자 키 파일, Termius 확인 사례, sshj curve25519 실패 |

## 7. 미확인·추정 목록

- **미확인**: Termphin·Shellwave·SSHBorg의 한글 입력·표시, 폴더블 레이아웃(Termphin·SSHBorg), SSHBorg의 공개키 내보내기 경로, 각 앱의 한국어 UI.
- **미확인**: Termphin 에이전트가 기본 셸이 PowerShell인 Windows에서 세션을 어떻게 재부착하는지(문서에 없음).
- **추정**: Shellwave의 curve25519 KEX 실패 가능성(sshj 사용 + herdr 앱 사례).
- **추정**: 사용자가 겪은 "Termius 무료 기간 종료"는 Pro 체험 종료이며 Starter는 여전히 무료. Play 리뷰에도 "계정을 만들면 체험 구독이 되는 줄 모른다"는 불만이 있다.
- **하지 않음**: 앱 설치·계정 가입·데스크탑 sshd 변경. 실기기 검증은 하지 않았다(카드에서 금지).
