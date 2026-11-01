---
title: SSH + Tailscale 원격 터미널 (Windows 11 · Android)
sources:
  - raw/dev/261101-docs-ssh-tailscale-remote-terminal-guide/ssh-tailscale-remote-terminal-guide.md
updated: 2026-11-01
tags: [ssh, openssh, tailscale, windows, android, remote]
status: draft
---

# SSH + Tailscale 원격 터미널 (Windows 11 · Android)

집의 Windows 11 PC 두 대(데스크탑·노트북)를 서로 SSH로 접속하고, 밖의 안드로이드 폰·태블릿에서도 두 PC에 접속하는 구성이다. 공유기 포트포워딩 없이 Tailscale 망 위에서 Windows 기본 OpenSSH를 쓰고, **SSH 키로만** 인증한다.

원본은 Microsoft·Tailscale·OpenSSH 공식 문서로 명령을 확인했지만 실제 기기에서는 아직 시험하지 않았다(2026-10 기준). 명령의 `<user>`, `desktop`, `laptop`, `100.x.y.z`, `<tailnet>`은 자리표시자다.

## 1. 구성과 순서

| 기기 | Tailscale | OpenSSH Server | SSH 키 | 접속 방향 |
| --- | --- | --- | --- | --- |
| 데스크탑 | 설치 | 설치 | 만듦 | ↔ 노트북 |
| 노트북 | 설치 | 설치 | 만듦 | ↔ 데스크탑 |
| 안드로이드 폰·태블릿 | 설치 | 없음 | 앱에서 기기마다 | → 데스크탑, 노트북 |

- Tailscale을 쓰면 포트포워딩·공인 IP·DDNS가 필요 없고 22번 포트를 인터넷에 열지 않는다. 기기마다 고정 이름(MagicDNS)과 주소(100.x.y.z)가 생긴다.
- **Tailscale SSH 기능은 쓰지 않는다.** Tailscale SSH 서버는 Linux·macOS만 지원한다. Windows에서는 OpenSSH Server를 Tailscale 망 위에서 쓴다.

순서: ① 모든 기기에 Tailscale(같은 계정) → ② 두 PC에 OpenSSH → ③ 접속하는 기기마다 키 → ④ 공개키 등록과 **키 접속 확인** → ⑤ 비밀번호 로그인 차단 → ⑥ 안드로이드 앱.

**⑤는 반드시 ④에서 키 접속을 확인한 뒤에 한다.** 순서가 바뀌면 원격으로 다시 들어갈 수 없다.

## 2. Tailscale

- PC: tailscale.com/download에서 설치 → 트레이 아이콘 → Log in. 모든 기기를 **같은 계정**으로 로그인한다.
- **Run unattended**(두 PC): Windows는 로그아웃하면 Tailscale도 끊긴다. 트레이 → Preferences → Run unattended, 또는 `tailscale up --unattended`.
- **키 만료 끄기**(두 PC만): 기기 키가 기본 180일마다 만료된다. 관리 콘솔 Machines → PC의 `⋯` → Disable key expiry. 잃어버릴 수 있는 폰·태블릿은 만료를 켜 둔다.
- 안드로이드: Play 스토어에서 설치 → 같은 계정 로그인 → **Always-on VPN**. 안드로이드는 VPN을 하나만 켤 수 있어서 다른 VPN을 켜면 Tailscale이 꺼진다.
- 확인: `tailscale status`, `tailscale ip -4`. MagicDNS로 `desktop` 또는 `desktop.<tailnet>.ts.net`으로 접속한다. 관리 콘솔에서 기기 이름을 짧게 바꿔 두면 편하다.

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

## 6. 비밀번호 로그인 차단 (두 PC)

시작 전에 키 접속을 확인했는지, PC 앞에 있거나 열린 SSH 세션이 하나 있는지 확인한다.

1. 관리자로 `C:\ProgramData\ssh\sshd_config`를 열고 `PubkeyAuthentication yes`, `PasswordAuthentication no`를 넣는다.
   - **파일 끝의 `Match Group administrators` 블록보다 위에** 넣는다. 아래에 쓰면 관리자 그룹에만 적용된다. `Match` 블록은 지우지 않는다.
   - Windows OpenSSH는 password와 publickey만 지원한다. Linux 가이드의 `KbdInteractiveAuthentication` 같은 옵션은 따라 넣지 않는다.
2. `& "$env:WINDIR\System32\OpenSSH\sshd.exe" -t`(출력 없으면 정상) → `Restart-Service sshd`.
3. **새 연결**로 확인한다(기존 세션은 닫지 않는다). `ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password <user>@laptop`이 비밀번호를 묻지 않고 `Permission denied (publickey)`로 끝나야 한다.

## 7. 안드로이드 앱

| 앱 | 가격 | 추천 대상 |
| --- | --- | --- |
| Termius | Starter 무료(동기화는 Pro) | 처음 쓰는 사람. Keychain에서 Ed25519 생성 → Hosts에 `desktop.<tailnet>.ts.net`·22·사용자·키 |
| ConnectBot | 무료, 오픈소스 | 무료·오픈소스. Manage Pubkeys에서 생성 → `<user>@desktop.<tailnet>.ts.net:22` |
| Termux | 무료, 오픈소스 | 명령행에 익숙한 사람. `pkg install openssh` 후 PC와 같은 `ssh-keygen`·`~/.ssh/config` |

JuiceSSH는 Play 스토어에서 확인되지 않아 새로 설치하지 않는다. 앱 메뉴 이름·키 형식은 버전마다 다르니 직접 확인한다.

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
- **첫 접속의 호스트 키 지문**은 접속받는 PC의 `ssh-keygen -lf C:\ProgramData\ssh\ssh_host_ed25519_key.pub`와 대조한 뒤 `yes`. 나중에 지문이 바뀌었다는 경고가 뜨면 접속하지 말고 원인부터 확인한다.
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

- 클라이언트 로그: `ssh -vvv <user>@laptop`. 서버는 PC 앞에서 `Stop-Service sshd` → `sshd.exe -d`(연결 하나 처리 후 종료) → `Start-Service sshd`. 로그 파일은 `sshd_config`에 `SyslogFacility LOCAL0`, `LogLevel DEBUG3`를 넣으면 `C:\ProgramData\ssh\logs\sshd.log`에 쌓인다(확인 후 지운다).
- Tailscale: `tailscale ping laptop`으로 닿는지, direct인지 relay인지 본다.
- 원본이 확정하지 못한 것: Tailscale 네트워크가 방화벽의 어느 프로필에 속하는지(`Get-NetConnectionProfile`), Windows 빌드별 OpenSSH 버전(`ssh -V`), 앱 메뉴 이름, 제조사별 배터리 메뉴.

## 원본

명령 모음과 기기별 설정 점검표, 공식 문서 링크 전체는 [원본](../../../raw/dev/261101-docs-ssh-tailscale-remote-terminal-guide/ssh-tailscale-remote-terminal-guide.md) 12절에 있다.
