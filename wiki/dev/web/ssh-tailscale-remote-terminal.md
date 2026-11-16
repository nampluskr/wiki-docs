---
title: SSH + Tailscale 원격 터미널 (Windows 11 · Android)
sources:
  - raw/web/261101-ssh-tailscale-remote-terminal-guide/ssh-tailscale-remote-terminal-guide.md
  - specs/v0.2/PROGRESS.md
  - raw/web/261116-android-ssh-client-research/COMPARE.md
updated: 2026-11-16
tags: [ssh, openssh, tailscale, windows, android, remote]
status: draft
---

# SSH + Tailscale 원격 터미널 (Windows 11 · Android)

집의 Windows 11 PC 두 대(데스크탑·노트북)를 서로 SSH로 접속하고, 밖의 안드로이드 폰·태블릿에서도 두 PC에 접속하는 구성이다. 공유기 포트포워딩 없이 Tailscale 망 위에서 Windows 기본 OpenSSH를 쓰고, **SSH 키로만** 인증한다.

원본은 Microsoft·Tailscale·OpenSSH 공식 문서로 명령을 확인했다. 2026-11-01~02에 데스크탑·노트북(Windows 11)·안드로이드 폰·태블릿에서 1~7절대로 실제 구성했고, 그때 확인한 사실은 **(확인됨)**으로 표시했다. 집 밖(LTE) 접속, 9·10절은 아직 시험하지 않았다. 명령의 `<user>`, `desktop`, `laptop`, `100.x.y.z`, `<tailnet>`은 자리표시자다.

## 1. 구성과 순서

| 기기 | Tailscale | OpenSSH Server | SSH 키 | 접속 방향 |
| --- | --- | --- | --- | --- |
| 데스크탑 | 설치 | 설치 | 만듦 | ↔ 노트북 |
| 노트북 | 설치 | 설치 | 만듦 | ↔ 데스크탑 |
| 안드로이드 폰·태블릿 | 설치 | 없음 | 앱에서 기기마다 | → 데스크탑, 노트북 |

- Tailscale을 쓰면 포트포워딩·공인 IP·DDNS가 필요 없고 22번 포트를 인터넷에 열지 않는다. 기기마다 고정 이름(MagicDNS)과 주소(100.x.y.z)가 생긴다.
- **접속받는 PC 설정에는 IP를 넣는 곳이 없다.** Tailscale이 주소와 이름을 정하고, sshd는 모든 네트워크의 22번 포트에서 받는다. IP·이름은 접속하는 쪽 명령(`ssh <user>@desktop` 또는 `ssh <user>@100.x.y.z`)과, 방화벽을 Tailscale 대역으로 제한할 때(10절)만 쓴다.
- **Tailscale SSH 기능은 쓰지 않는다.** Tailscale SSH 서버는 Linux·macOS만 지원한다. Windows에서는 OpenSSH Server를 Tailscale 망 위에서 쓴다.

순서: ① 모든 기기에 Tailscale(같은 계정) → ② 두 PC에 OpenSSH → ③ 접속하는 기기마다 키 → ④ 공개키 등록과 **키 접속 확인** → ⑤ 비밀번호 로그인 차단 → ⑥ 안드로이드 앱.

**⑤는 반드시 ④에서 키 접속을 확인한 뒤에 한다.** 순서가 바뀌면 원격으로 다시 들어갈 수 없다.

## 2. Tailscale

- PC: tailscale.com/download에서 설치 → 트레이 아이콘 → Log in. 모든 기기를 **같은 계정**으로 로그인한다.
- **Run unattended**(두 PC): Windows는 로그아웃하면 Tailscale도 끊긴다. 트레이 → Preferences → Run unattended, 또는 `tailscale up --unattended`.
- **키 만료 끄기**(두 PC만): 기기 키가 기본 180일마다 만료된다. 잃어버릴 수 있는 폰·태블릿은 만료를 켜 둔다.
  - 관리 콘솔은 영어로만 나온다. 메뉴 이름은 영어 그대로 찾는다.
  - 주소창에 `https://login.tailscale.com/admin/machines`를 넣으면 기기 목록(Machines)으로 바로 간다. 로그인을 물으면 Tailscale을 설치할 때 쓴 계정으로 로그인한다.
  - 목록에서 그 PC 행을 찾아 맨 오른쪽 `⋯` → **Disable key expiry**.
  - 확인: PC 이름 아래에 **Expiry disabled**가 보인다.
- 안드로이드: Play 스토어에서 설치 → 같은 계정 로그인 → **Always-on VPN**. 안드로이드는 VPN을 하나만 켤 수 있어서 다른 VPN을 켜면 Tailscale이 꺼진다.
- 확인: `tailscale status`, `tailscale ip -4`. MagicDNS로 `desktop` 또는 `desktop.<tailnet>.ts.net`으로 접속한다. 관리 콘솔에서 기기 이름을 짧게 바꿔 두면 편하다.
  - **(확인됨)** 설치 전에 열어 둔 터미널은 `tailscale`을 찾지 못한다(`'tailscale' 용어는 … 인식되지 않습니다`). 설치가 시스템 PATH에 `C:\Program Files\Tailscale\`을 넣으므로 새 창을 열면 된다.
  - **(확인됨)** 관리 콘솔에서 기기 이름을 바꾸면 MagicDNS 이름도 바로 바뀐다(`book-…` → `laptop`, `z-fold5` → `fold`).
- **(확인됨)** 안드로이드 Always-on VPN 위치: 설정 앱 검색창에 `VPN` → Tailscale 오른쪽 톱니 → **상시 접속 VPN**(삼성 갤럭시). 같은 화면의 "VPN 없는 연결 차단"은 꺼 둔다. 목록에 Tailscale이 없으면 앱에서 한 번 연결하면 나타난다.

## 3. OpenSSH 서버 (두 PC, 관리자 PowerShell)

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
Get-NetFirewallRule -Name OpenSSH-Server-In-TCP | Select-Object Name, Enabled, Profile   # 설치 때 자동 생성됨
# 기본 셸을 PowerShell로 (기본은 cmd.exe)
New-ItemProperty -Path 'HKLM:\SOFTWARE\OpenSSH' -Name DefaultShell `
  -Value 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' -PropertyType String -Force
```

방화벽 규칙은 없을 때만 `New-NetFirewallRule … -LocalPort 22`로 만든다. PowerShell 7을 기본 셸로 하려면 `pwsh.exe` 실제 경로를 넣는다.

- **(확인됨)** Windows 업데이트 재시작이 걸려 있으면 Server 설치에 재부팅이 필요하다(노트북). 걸려 있지 않으면 `RestartNeeded : False`로 바로 끝난다(데스크탑). Server 설치는 몇 분 걸린다.
- **(확인됨)** 자동 생성된 방화벽 규칙의 Profile은 `Private`뿐이었고, `Get-NetConnectionProfile`로 보면 Tailscale 어댑터도 `Private`로 잡혀 있어 추가 조치 없이 Tailscale로 접속됐다(두 PC).

## 4. SSH 키

- **접속하는 기기마다 따로 만든다.** 한 기기를 잃어버리면 그 기기 키만 지운다. 개인키(`id_ed25519`)는 만든 기기 밖으로 옮기지 않는다.
- PC: `ssh-keygen -t ed25519 -C "desktop"`(노트북은 `"laptop"`). 암호문구를 건다.
- 암호문구를 한 번만 넣으려면 ssh-agent를 켠다(기본 꺼짐):

  ```powershell
  Get-Service ssh-agent | Set-Service -StartupType Automatic   # 관리자
  Start-Service ssh-agent
  ssh-add "$env:USERPROFILE\.ssh\id_ed25519"                   # 일반 PowerShell
  ```

## 5. 공개키 등록 (접속받는 PC)

데스크탑에는 노트북·폰·태블릿의 공개키를, 노트북에는 데스크탑·폰·태블릿의 공개키를 등록한다. 공개키는 한 줄 텍스트라 Taildrop·메모·메신저로 옮겨도 된다.

**관리자 계정이면 넣는 파일이 다르다.** `whoami`(사용자 이름은 `\` 뒤 부분, 마이크로소프트 계정이어도 로컬 이름)와 `net localgroup administrators`로 먼저 확인한다. 관리자 계정이 일반 위치에 키를 넣으면 키 인증이 안 되고 비밀번호를 묻는다.

| 계정 | 공개키 파일 | 권한 정리 |
| --- | --- | --- |
| 일반 사용자 | `%USERPROFILE%\.ssh\authorized_keys` | `icacls.exe $f /inheritance:r /grant:r "$(whoami):F" "*S-1-5-18:F" "*S-1-5-32-544:F"` |
| 관리자 그룹 | `C:\ProgramData\ssh\administrators_authorized_keys` | `icacls.exe $f /inheritance:r /grant:r "*S-1-5-32-544:F" "*S-1-5-18:F"` |

- 키 하나에 한 줄. `Add-Content -Encoding ascii -LiteralPath $f -Value "ssh-ed25519 AAAA... laptop"`. 파일 이름에 `.txt`가 붙지 않게 한다.
- `*S-1-5-18`(SYSTEM), `*S-1-5-32-544`(Administrators)는 SID라 한국어 Windows에서도 그대로 쓴다. 다른 계정·그룹에 쓰기 권한이 남아 있으면 sshd가 키 파일을 무시하므로 `icacls.exe $f /remove <계정>`으로 지운다. 일반 계정 파일은 **접속 대상 계정**으로 실행한다.
- 관리자 파일은 관리자 그룹 전원이 함께 쓴다. 일상 접속은 일반 계정이 더 안전하다.
- 확인: `ssh -o PreferredAuthentications=publickey <user>@laptop`. **양방향 모두** 확인한 뒤 6절로 간다.
- **(확인됨)** 관리자 파일은 **관리자 PowerShell**에서만 만들고 고칠 수 있다. 일반 창에서는 `Access to the path … is denied`가 난다.
- 접속받는 PC 자신의 공개키는 넣지 않는다. 넣으면 자기 자신으로의 접속만 열릴 뿐, 기기별로 줄을 관리하기 번거로워진다.

### 공개키 옮기기: Taildrop (확인됨)

- PC → PC: 보내는 PC에서 `tailscale file cp "$env:USERPROFILE\.ssh\id_ed25519.pub" laptop:`, 받는 PC에서 `tailscale file get --wait <받을 폴더>`. `--wait` 없이 실행하면 아직 안 온 파일은 받지 않는다(`moved 0/0 files`).
- 안드로이드 → PC: SSH 앱의 공개키 공유 → Tailscale → 받을 PC. PC의 `Downloads`에 **임의 이름의 `.txt`**, **이름표(주석) 없는 한 줄**로 도착했다. 등록할 때 이름을 붙인다.

  ```powershell
  # 관리자 PowerShell
  $f = 'C:\ProgramData\ssh\administrators_authorized_keys'
  $k = (Get-Content "$env:USERPROFILE\Downloads\<받은 파일>.txt" -Raw).Trim()
  Add-Content -Encoding ascii -LiteralPath $f -Value "$k phone"
  icacls.exe $f /inheritance:r /grant:r "*S-1-5-32-544:F" "*S-1-5-18:F"
  ```

## 6. 비밀번호 로그인 차단 (두 PC)

**왜 막나.** 필수는 아니지만 키로만 접속하는 편이 안전하다.

- 설치 때 생긴 방화벽 규칙은 `Private` 프로필 전체에 열려 있어(3절), 같은 공유기 안의 다른 기기(가족 PC, IoT, 감염된 기기)도 22번 포트로 비밀번호를 대입해 볼 수 있다.
- SSH 비밀번호는 Windows 로그인 비밀번호다. 마이크로소프트 계정이면 메일·클라우드와 같은 비밀번호라 다른 곳에서 새면 SSH도 열린다.
- Ed25519 키는 대입으로 뚫을 수 없고, 개인키는 만든 기기에만 있다.
- 단점: 키가 없는 새 기기는 공개키를 먼저 등록해야 접속되고, 키를 모두 잃으면 PC 앞에서 고쳐야 한다.
- 막지 않을 때의 차선은 10절 방화벽 제한(Tailscale 대역만)이다. 그래도 tailnet 안의 기기에서는 비밀번호 시도가 가능하다.

시작 전에 키 접속을 확인했는지, PC 앞에 있거나 열린 SSH 세션이 하나 있는지 확인한다.

1. 관리자로 `C:\ProgramData\ssh\sshd_config`를 열고 `PubkeyAuthentication yes`, `PasswordAuthentication no`를 넣는다.
   - **파일 끝의 `Match Group administrators` 블록보다 위에** 넣는다. 아래에 쓰면 관리자 그룹에만 적용된다. `Match` 블록은 지우지 않는다.
   - Windows OpenSSH는 password와 publickey만 지원한다. Linux 가이드의 `KbdInteractiveAuthentication` 같은 옵션은 따라 넣지 않는다.
2. `& "$env:WINDIR\System32\OpenSSH\sshd.exe" -t`(출력 없으면 정상) → `Restart-Service sshd`.
3. **새 연결**로 확인한다(기존 세션은 닫지 않는다). `ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password <user>@laptop`이 비밀번호를 묻지 않고 `Permission denied (publickey)`로 끝나야 한다.

**(확인됨)** 기본 `sshd_config`에서는 `#PubkeyAuthentication yes`(34행), `#PasswordAuthentication yes`(51행)가 이미 `Match` 블록(87행)보다 위에 있어서, 1단계를 주석만 바꾸는 명령으로 할 수 있었다(두 PC 같음).

```powershell
# 관리자 PowerShell
$c = 'C:\ProgramData\ssh\sshd_config'
Copy-Item $c "$c.bak"
(Get-Content $c) -replace '^#PubkeyAuthentication yes$','PubkeyAuthentication yes' -replace '^#PasswordAuthentication yes$','PasswordAuthentication no' | Set-Content -Encoding ascii $c
Select-String -Path $c -Pattern 'PasswordAuthentication|PubkeyAuthentication|^Match'   # Pubkey yes, Password no, Match 순서
& "$env:WINDIR\System32\OpenSSH\sshd.exe" -t
Restart-Service sshd
```

**(확인됨)** 3단계의 실제 메시지는 `Permission denied (publickey,keyboard-interactive)`였다. 서버가 `keyboard-interactive`를 계속 내걸지만 `ssh -v`로 보면 비밀번호 질의가 오지 않아 비밀번호로는 들어갈 수 없다. 되돌릴 때는 `Copy-Item "$c.bak" $c -Force; Restart-Service sshd`.

## 7. 안드로이드 앱

| 앱 | 가격 | 추천 대상 |
| --- | --- | --- |
| ConnectBot | 무료, 인앱 결제 없음, 오픈소스(Apache-2.0) | **지금 쓰는 앱**(2026-11-16 태블릿). Termius와 같은 흐름(앱에서 Ed25519 생성 → 공개키 공유) |
| Termius | Starter 무료(동기화는 Pro) | 2026-11 사용자의 무료 사용 기간이 끝나 ConnectBot으로 옮겼다. Starter 플랜 자체는 가격 페이지에 남아 있다(조사, 2026-11-16) |
| Termux | 무료, 오픈소스(GPLv3) | 명령행에 익숙한 사람. `pkg install openssh` 후 PC와 같은 `ssh-keygen`·`~/.ssh/config`. Play 판은 실험판이라 F-Droid·GitHub 판을 쓴다 |

JuiceSSH는 Play 스토어에서 2025-12-11 내려갔고 마지막 릴리스가 2021-02라 쓰지 않는다(조사, 2026-11-16). 앱 메뉴 이름·키 형식은 버전마다 다르니 직접 확인한다.

**(확인됨)** Termius(폰·태블릿): Keychain에서 Ed25519 키 생성 → 공개키 공유로 Taildrop 전송(5절) → 데스크탑에 등록 → Hosts에 `desktop`·22·사용자·키를 넣고 접속했다. 배터리 설정에서 Tailscale을 "제한 없음"으로 둔다.

### 7.1 ConnectBot 설치와 설정

2026-11-16 태블릿에서 아래 순서로 데스크탑에 접속했다. **(확인됨)**이 없는 줄은 조사 결과(웹 문서)이고 아직 시험하지 않았다. 폰도 같은 순서로 하되 **기기마다 키를 따로** 만든다(4절).

1. **설치** **(확인됨)**: Play 스토어에서 `ConnectBot`(`org.connectbot`)을 설치한다. F-Droid에도 `1.11.1-oss`가 있다(Google Play 서비스 불필요). 2026-10-14에 1.11.1이 나왔다.
2. **키 만들기** **(확인됨)**: `⋮` → **Manage Pubkeys** → `+` → 이름(태블릿은 `tab-connectbot`), 형식 **Ed25519**, 암호문구는 선택 → **Generate**. 암호문구를 걸면 **Load key on start**를 켜 두면 편하다.
3. **공개키 보내기** **(확인됨)**: 키의 **공개키 내보내기** → 폴더에 저장 → 저장된 파일을 공유 → **Tailscale** → `desktop`. 데스크탑 `Downloads`에 `tab-connectbot.pub.txt`로 도착했다. Termius와 달리 **파일 이름과 줄 끝의 이름표(`tab-connectbot`)가 그대로** 들어 있다.
4. **데스크탑 등록** **(확인됨)**: 관리자 PowerShell. 이름표가 이미 있으므로 덧붙이지 않는다.

   ```powershell
   $f = 'C:\ProgramData\ssh\administrators_authorized_keys'
   $k = (Get-Content "$env:USERPROFILE\Downloads\tab-connectbot.pub.txt" -Raw).Trim()
   Add-Content -Encoding ascii -LiteralPath $f -Value $k
   icacls.exe $f /inheritance:r /grant:r "*S-1-5-32-544:F" "*S-1-5-18:F"
   Get-Content $f | ForEach-Object { ($_ -split ' ')[0,2] -join ' ' }   # 키 종류와 이름표만 표시
   ```

5. **호스트 등록** **(확인됨)**: 새 호스트 화면에서 다섯 칸을 채우고, 인증에 2번의 키를 고른다.

   | 칸 | 값 |
   | --- | --- |
   | 닉네임 | `desktop`(화면에 보일 이름) |
   | 프로토콜 | `ssh` |
   | 사용자 이름 | `<user>` |
   | 호스트 | `desktop.<tailnet>.ts.net`(안 되면 `100.x.y.z`) |
   | 포트 | `22` |

6. **첫 접속** **(확인됨: 접속)**: 호스트 키 지문은 8절대로 대조한 뒤 수락한다.
7. **다듬기**(조사):
   - Settings → **Special keys always visible**을 켜면 Ctrl·Esc·Tab·화살표가 늘 보인다.
   - 접속했는데 화면이 비면 호스트 편집에서 **Start shell session**이 켜져 있는지 본다(이슈 #1093의 흔한 원인).
   - 백그라운드 유지는 설정의 **Persist connections**와 알림 권한, 배터리 최적화 예외로 한다.
8. **정리**: Termius 키 줄은 ConnectBot을 며칠 써 보고 `administrators_authorized_keys`에서 그 이름표의 줄을 지운다(10절).

ConnectBot에서 아직 확인하지 않은 것(조사, 2026-11-16):
- **한글 입력**: 1.11.0 변경 내역에 "Improve IME input handling for Hangul"이 있고, 조합 중 글자가 빠지는 이슈 #2352는 2026-09-22에 닫혔다는 보고가 있다. 열린 이슈로 본 보고도 있었다. 삼성 키보드로 직접 확인한다.
- TUI 마우스 트래킹(이슈 #2351), Android 14+ 터치패드 두 손가락 스크롤(#2549), 폴더블 접기·펴기.
- 지원하지 않는 것: SFTP는 없다. 앱에 Mosh가 있지만 Windows에는 `mosh-server`가 없어 이 구성에서는 쓸 수 없다(9절).

다른 후보(ZestSSH, Termphin, Shellwave, SSHBorg 등)와 네 에이전트의 비교는 [안드로이드 SSH 앱 조사](../../../raw/web/261116-android-ssh-client-research/COMPARE.md)에 있다.

## 8. 접속

`%USERPROFILE%\.ssh\config`(확장자 없음)로 줄인다. 데스크탑 예:

```text
Host laptop
    HostName laptop.<tailnet>.ts.net
    User <user>
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

- 이제 `ssh laptop`. `IdentitiesOnly yes`는 엉뚱한 키로 시도하다 실패하는 것을 막는다.
- **첫 접속의 호스트 키 지문**은 접속받는 PC의 `ssh-keygen -lf C:\ProgramData\ssh\ssh_host_ed25519_key.pub`와 대조한 뒤 `yes`. 나중에 지문이 바뀌었다는 경고가 뜨면 접속하지 말고 원인부터 확인한다. **(확인됨)** 이 지문 명령은 일반 창에서 `Permission denied`가 나므로 관리자 PowerShell에서 실행한다.
- 집 밖 시험: 폰 Wi-Fi를 끄고 LTE로 두 PC에 접속해 본다. PC 재부팅·로그아웃 뒤에도 한 번씩.

## 9. 연결 유지와 전원

- keepalive는 쉬는 연결을 유지할 뿐 **끊긴 세션을 되살리지 않는다.** Windows에는 tmux가 없어서 연결이 끊기면 그 PowerShell과 돌던 명령도 끝난다. 오래 걸리는 작업은 `Start-Process` 등으로 따로 띄운다.
- **절전 중인 PC에는 Tailscale로도 접속할 수 없고 원격으로 깨울 수도 없다.** 전원 연결 시 `powercfg /change standby-timeout-ac 0`, `powercfg /change hibernate-timeout-ac 0`. 이 명령은 현재 전원 구성표의 유휴 타이머만 바꾼다. 노트북은 "덮개를 닫으면"도 전원 연결 시 "아무 것도 안 함"으로.
- 안드로이드는 Always-on VPN, 배터리 최적화에서 Tailscale을 "제한 없음"으로.

## 10. 보안 강화 (선택)

- **Tailscale grants로 22번만 허용**: 두 PC에 `tag:sshserver` 태그를 붙이고 정책에 `{"src": ["autogroup:member", "tag:sshserver"], "dst": ["tag:sshserver"], "ip": ["tcp:22"]}`. `src`에 `tag:sshserver`도 넣어야 PC끼리 접속된다(태그 기기는 `autogroup:member`가 아니다). 기본 전체 허용 정책을 바꾸면 22번 외 tailnet 통신(Taildrop 등)은 막힌다. 허용 규칙은 합쳐지므로 전체 허용 규칙을 남기면 제한되지 않는다.
- **Windows 방화벽에서 Tailscale 주소만**: `Set-NetFirewallRule -Name OpenSSH-Server-In-TCP -RemoteAddress 100.64.0.0/10`. 같은 LAN의 직접 접속도 막힌다.
- **폰을 잃어버렸을 때**: 관리 콘솔에서 기기 Remove → 두 PC의 공개키 파일에서 그 기기 줄 삭제(키에 `-C "phone"` 이름을 붙여 둔 이유) → 새 기기는 새 키.

## 11. 문제 해결

| 증상 | 점검 |
| --- | --- |
| `Connection timed out` | 양쪽 `tailscale status`에서 상대가 online인지, 상대 PC가 절전 중인지, 안드로이드 Tailscale·다른 VPN |
| `Connection refused` | `Get-Service sshd`, `Get-NetTCPConnection -LocalPort 22 -State Listen`, 방화벽 규칙 |
| 키가 있는데 비밀번호를 묻는다 / `Permission denied (publickey)` | 관리자 계정인데 `authorized_keys`에 넣었는지, 파일 권한(`icacls`) |
| 재부팅·로그아웃 뒤 안 됨 | Run unattended, sshd 자동 시작, 키 만료 |
| 몇 달 뒤 갑자기 안 됨 | Tailscale 키 만료 |
| 자주 끊긴다 | keepalive, 안드로이드 배터리 제한. `tailscale status`의 relay 표시는 고장이 아니다 |
| `'tailscale' 용어는 … 인식되지 않습니다` | 설치 전에 연 터미널이다. 새 창을 연다 (확인됨) |
| `Could not resolve hostname … \276\313…` | 호스트 이름 오타(예: `destop`). 뒤의 깨진 글자는 한국어 오류 메시지의 인코딩 문제다 (확인됨) |
| 키 파일·지문 명령에서 `Access … denied` / `Permission denied` | 관리자 PowerShell이 아닌 창이다 (확인됨) |

- 클라이언트 로그: `ssh -vvv <user>@laptop`. 서버는 PC 앞에서 `Stop-Service sshd` → `sshd.exe -d`(연결 하나 처리 후 종료) → `Start-Service sshd`. 로그 파일은 `sshd_config`에 `SyslogFacility LOCAL0`, `LogLevel DEBUG3`를 넣으면 `C:\ProgramData\ssh\logs\sshd.log`에 쌓인다(확인 후 지운다).
- Tailscale: `tailscale ping laptop`으로 닿는지, direct인지 relay인지 본다.
- 원본이 확정하지 못한 것: Windows 빌드별 OpenSSH 버전(`ssh -V`), 앱 메뉴 이름, 제조사별 배터리 메뉴.

## 원본

명령 모음과 기기별 설정 점검표, 공식 문서 링크 전체는 [원본](../../../raw/web/261101-ssh-tailscale-remote-terminal-guide/ssh-tailscale-remote-terminal-guide.md) 12절에 있다.
