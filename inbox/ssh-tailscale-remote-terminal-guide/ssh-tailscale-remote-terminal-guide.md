---
tags: [ssh, tailscale, windows, android, remote, guide]
created: 2026-10-31
updated: 2026-10-31
---

# SSH + Tailscale 원격 터미널 접속 가이드

집의 Windows 11 PC 두 대(데스크탑·노트북)를 서로 SSH로 접속하고, 밖에 있는 안드로이드 스마트폰·태블릿에서도 두 PC에 접속하게 만든다. 공유기 포트포워딩 없이 Tailscale 망 위에서 SSH를 쓴다.

| 항목 | 내용 |
| --- | --- |
| 대상 | SSH와 Tailscale을 처음 설정하는 사람 |
| 결과 | 데스크탑 ↔ 노트북 양방향 접속, 안드로이드 → 각 PC 접속 (집 밖 LTE에서도) |
| 인증 | SSH 키만 쓴다. 비밀번호 로그인은 막는다 |
| 기준 | Windows 11, OpenSSH (Windows 기본 제공판), Tailscale 2026-10 기준 문서 |

> 명령의 `<user>`, `desktop`, `laptop`, `100.x.y.z`, `<tailnet>`은 자리표시자다. 자기 환경의 값으로 바꾼다.
>
> 이 문서의 명령은 공식 문서로 확인했지만 실제 기기에서의 시험은 아직 거치지 않았다. 시험하면서 다른 점이 나오면 해당 절을 고친다.

## 1. 개요

### 1.1 구성도

```
  tailnet (Tailscale 가상 사설망, 100.x.y.z) — 모든 연결이 이 안에서만 오간다

  ┌──────────────────────┐    SSH (22)    ┌──────────────────────┐
  │ 데스크탑 (Windows 11) │ ◀────────────▶ │ 노트북 (Windows 11)   │
  │ 서버 + 클라이언트     │                │ 서버 + 클라이언트      │
  └──────────▲───────────┘                └──────────▲───────────┘
             │ SSH                                   │ SSH
             └──────────────┬────────────────────────┘
                            │
                 ┌──────────┴───────────┐
                 │ 안드로이드 폰·태블릿  │   집 밖 LTE·다른 Wi-Fi
                 │ 클라이언트만          │
                 └──────────────────────┘
```

### 1.2 기기별 역할

| 기기 | Tailscale | OpenSSH Server | SSH 키 | 접속 방향 |
| --- | --- | --- | --- | --- |
| 데스크탑 | 설치 | 설치 | 만듦 (노트북 접속용) | ↔ 노트북 |
| 노트북 | 설치 | 설치 | 만듦 (데스크탑 접속용) | ↔ 데스크탑 |
| 안드로이드 폰·태블릿 | 설치 | 없음 | 앱에서 기기마다 만듦 | → 데스크탑, 노트북 |

PC 두 대는 같은 설정을 양쪽에 똑같이 한다. 이 문서의 2~6절은 **두 PC 모두에서** 한다.

### 1.3 왜 Tailscale인가

- 포트포워딩·공인 IP·DDNS가 필요 없다. 같은 tailnet에 들어온 기기끼리만 통신한다.
- SSH 포트(22)를 인터넷에 열지 않는다.
- 기기마다 바뀌지 않는 이름(MagicDNS)과 주소(100.x.y.z)가 생긴다.

**Tailscale SSH 기능은 쓰지 않는다.** Tailscale SSH 서버는 Linux·macOS만 지원하고 Windows는 지원하지 않는다. 그래서 Windows에서는 OpenSSH Server를 Tailscale 망 위에서 쓴다. ([Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh))

### 1.4 진행 순서

| 순서 | 할 일 | 절 |
| --- | --- | --- |
| 1 | 모든 기기에 Tailscale 설치, 같은 계정 로그인 | 2 |
| 2 | 두 PC에 OpenSSH 설치 | 3 |
| 3 | 접속하는 쪽 기기마다 키 만들기 | 4 |
| 4 | 공개키 등록, **키 접속 확인** | 5 |
| 5 | 비밀번호 로그인 차단 | 6 |
| 6 | 안드로이드 앱 설정, 접속 | 7, 8 |

**5번(비밀번호 차단)은 반드시 4번에서 키 접속을 확인한 뒤에 한다.** 순서가 바뀌면 원격으로 다시 들어갈 수 없다.

## 2. Tailscale 설치와 설정

### 2.1 Windows (데스크탑·노트북)

1. [tailscale.com/download](https://tailscale.com/download)에서 Windows 설치 파일을 받아 실행한다
2. 작업 표시줄 트레이의 Tailscale 아이콘 → **Log in** → 브라우저에서 로그인한다
3. **두 PC와 안드로이드 기기 모두 같은 계정**으로 로그인한다. 계정이 다르면 같은 tailnet에 초대해야 한다

### 2.2 Run unattended (두 PC)

Windows에서는 사용자가 로그아웃하면 Tailscale 연결도 끊긴다. 로그아웃·재부팅 뒤에도 접속하려면 켠다.

- 트레이 아이콘 → **Preferences** → **Run unattended**
- 또는 PowerShell: `tailscale up --unattended`

이 설정을 켠 Windows 계정이 그 PC의 Tailscale 설정을 소유한다. ([Run unattended](https://tailscale.com/docs/how-to/run-unattended))

### 2.3 키 만료 끄기 (두 PC만)

Tailscale 기기 키는 기본 180일마다 만료되어 다시 로그인해야 한다. 원격으로 쓰는 PC가 만료되면 집에 갈 때까지 접속할 수 없다.

- [관리 콘솔](https://login.tailscale.com/admin/machines) → **Machines** → 해당 PC의 `⋯` → **Disable key expiry**
- **안드로이드 폰·태블릿은 만료를 켜 둔다.** 잃어버릴 수 있는 기기이기 때문이다

이것은 Tailscale 기기 키 이야기이고, SSH 키와는 별개다. ([Key expiry](https://tailscale.com/kb/1028/key-expiry))

### 2.4 안드로이드 (폰·태블릿)

1. Play 스토어에서 **Tailscale** 설치
2. **Get Started** → VPN 연결 요청 허용 → 같은 계정으로 로그인
3. 백그라운드에서 끊기지 않게 **Always-on VPN**을 켠다
   - 설정 → 네트워크 및 인터넷 → VPN → Tailscale 옆 톱니바퀴 → 상시 사용 VPN (메뉴 이름은 제조사마다 조금 다르다)

안드로이드는 VPN을 **하나만** 켤 수 있다. 다른 VPN 앱을 쓰고 있으면 Tailscale이 꺼진다. ([Android 설치](https://tailscale.com/docs/install/android), [다른 VPN과 병용](https://tailscale.com/docs/reference/faq/other-vpns))

### 2.5 이름과 주소 확인

PC의 PowerShell에서:

```powershell
tailscale status     # tailnet 안의 기기 목록과 연결 상태
tailscale ip -4      # 이 기기의 100.x.y.z 주소
```

- **MagicDNS**가 켜져 있으면 기기 이름으로 접속할 수 있다. 짧은 이름 `desktop`, 전체 이름 `desktop.<tailnet>.ts.net` 둘 다 된다. 2022-10 이후 만든 tailnet은 기본으로 켜져 있다
- 전체 이름은 관리 콘솔 Machines에서 기기를 눌러 확인한다
- 이름으로 안 되면 100.x.y.z 주소로 시험한다

관리 콘솔에서 기기 이름을 `desktop`, `laptop`처럼 짧게 바꿔 두면 이후 명령이 편하다. ([MagicDNS](https://tailscale.com/kb/1081/magicdns), [CLI](https://tailscale.com/docs/reference/tailscale-cli))

## 3. OpenSSH 설치와 기본 설정 (두 PC)

아래 명령은 모두 **관리자 PowerShell**에서 실행한다 (시작 메뉴에서 PowerShell 우클릭 → 관리자 권한으로 실행).

### 3.1 설치

```powershell
# 설치 상태 확인
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

# Client(접속하는 쪽)와 Server(접속받는 쪽) 둘 다 설치
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

GUI로 하려면: 설정 → 시스템 → 선택적 기능 → 기능 보기 → **OpenSSH 서버** 추가. 메뉴 이름은 Windows 빌드에 따라 조금 다르다.

### 3.2 서비스 자동 시작

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

### 3.3 방화벽 규칙 확인

설치할 때 TCP 22 인바운드 규칙이 자동으로 만들어진다. 확인만 한다.

```powershell
Get-NetFirewallRule -Name OpenSSH-Server-In-TCP | Select-Object Name, Enabled, Profile
```

**규칙이 없을 때만** 만든다.

```powershell
New-NetFirewallRule -Name OpenSSH-Server-In-TCP -DisplayName 'OpenSSH Server (sshd)' `
  -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

### 3.4 기본 셸을 PowerShell로

기본 셸은 `cmd.exe`다. PowerShell로 바꾼다.

```powershell
New-ItemProperty -Path 'HKLM:\SOFTWARE\OpenSSH' -Name DefaultShell `
  -Value 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' -PropertyType String -Force
```

PowerShell 7을 쓰려면 `-Value`를 `pwsh.exe`의 실제 경로(보통 `C:\Program Files\PowerShell\7\pwsh.exe`)로 바꾼다.

### 3.5 확인

```powershell
Get-Service sshd                               # Running 이어야 한다
Get-NetTCPConnection -LocalPort 22 -State Listen
ssh -V                                         # OpenSSH 버전
```

출처: [OpenSSH 설치·최초 사용](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse), [OpenSSH 서버 설정](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh-server-configuration)

## 4. SSH 키 만들기

**키는 접속하는 기기마다 따로 만든다.** 데스크탑 키, 노트북 키, 폰 키, 태블릿 키가 각각 있다. 한 기기를 잃어버리면 그 기기의 키만 지우면 된다.

| 파일 | 뜻 | 어디에 두나 |
| --- | --- | --- |
| `id_ed25519` | 개인키 | 만든 기기에만. **절대 복사·전송하지 않는다** |
| `id_ed25519.pub` | 공개키 | 접속할 PC에 등록한다 (5절) |

### 4.1 PC에서 키 만들기

SSH 클라이언트로 쓸 Windows 사용자로(관리자 권한 필요 없음) PowerShell에서:

```powershell
ssh-keygen -t ed25519 -C "desktop"      # 노트북에서는 -C "laptop"
```

- 저장 위치는 Enter로 기본값(`%USERPROFILE%\.ssh\id_ed25519`)을 쓴다. 이미 키가 있으면 덮어쓰지 않는다
- **암호문구(passphrase)를 건다.** 개인키가 유출돼도 바로 쓰이지 않는다

공개키 내용 보기:

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
# ssh-ed25519 AAAA...... desktop  ← 이 한 줄이 공개키다
```

### 4.2 ssh-agent로 암호문구 한 번만 입력

ssh-agent는 기본으로 꺼져 있다. **관리자 PowerShell**에서 켠다.

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

일반 PowerShell에서 키를 넣는다. 이후 접속할 때 암호문구를 묻지 않는다.

```powershell
ssh-add "$env:USERPROFILE\.ssh\id_ed25519"
```

### 4.3 안드로이드에서 키 만들기

앱 안에서 기기마다 만든다. 앱별 방법은 7절에 있다. 만든 뒤 **공개키를 복사**해 두면 5절에서 PC에 등록한다.

출처: [OpenSSH 키 관리](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement)

## 5. 공개키 등록 (두 PC)

접속받는 PC에, 접속할 기기들의 공개키를 등록한다.

| 접속받는 PC | 등록할 공개키 |
| --- | --- |
| 데스크탑 | 노트북, 폰, 태블릿 |
| 노트북 | 데스크탑, 폰, 태블릿 |

### 5.1 접속 계정 확인

먼저 **접속받는 PC**에서 SSH로 들어갈 Windows 계정 이름과 관리자 여부를 확인한다.

```powershell
whoami                                   # 예: desktop\<user>
net localgroup administrators            # 목록에 그 계정이 있으면 관리자
```

- SSH 사용자 이름은 `whoami`의 `\` 뒤 부분이다. 마이크로소프트 계정으로 로그인하는 PC는 이메일이 아니라 이 로컬 이름을 쓴다
- **관리자 계정인지에 따라 공개키를 넣는 파일이 다르다.** 관리자 계정이 일반 계정 위치에 키를 넣으면 키 인증이 안 되고 비밀번호를 묻는다

| 계정 종류 | 공개키 파일 |
| --- | --- |
| 일반 사용자 | `%USERPROFILE%\.ssh\authorized_keys` |
| 관리자 그룹 구성원 | `C:\ProgramData\ssh\administrators_authorized_keys` |

### 5.2 공개키를 접속받는 PC로 옮기기

공개키는 한 줄짜리 텍스트라서 유출돼도 위험하지 않다. 편한 방법으로 옮긴다.

- **Taildrop**: Tailscale의 파일 보내기. 보내는 기기에서 공유 → Tailscale → 받을 PC 선택 (`.pub` 파일이나 텍스트 파일)
- 메모 앱·메신저·이메일로 공개키 한 줄을 보내 복사한다
- (PC끼리) 아직 비밀번호 로그인이 켜져 있는 지금, `ssh <user>@laptop`으로 들어가 붙여 넣는다

### 5.3 일반 사용자 계정

**접속받는 PC**에서, 그 계정으로 로그인한 PowerShell에서 실행한다. `ssh-ed25519 AAAA... laptop`은 실제 공개키 한 줄로 바꾼다.

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.ssh"
Add-Content -Encoding ascii -LiteralPath "$env:USERPROFILE\.ssh\authorized_keys" `
  -Value "ssh-ed25519 AAAA... laptop"
```

- 키 하나에 한 줄. 여러 기기의 키는 줄을 더해 넣는다
- 파일 이름에 `.txt`가 붙지 않게 한다 (메모장으로 만들 때 주의)

권한 정리: 본인·SYSTEM·Administrators만 남기고 다른 사용자 권한을 뺀다.

```powershell
$f = "$env:USERPROFILE\.ssh\authorized_keys"
icacls.exe $f /inheritance:r /grant:r "$(whoami):F" "*S-1-5-18:F" "*S-1-5-32-544:F"
icacls.exe $f                            # 결과 확인
```

`*S-1-5-18`은 SYSTEM, `*S-1-5-32-544`는 Administrators 그룹의 SID다. 한국어 Windows에서 그룹 이름이 달라도 그대로 쓸 수 있다.

- 이 세 주체(본인·SYSTEM·Administrators)는 sshd 권한 검사를 통과한다. 그 밖의 계정·그룹(Users, Authenticated Users 등)에 쓰기 권한이 있으면 sshd가 키 파일을 거부한다
- `/inheritance:r`은 상속 권한만, `/grant:r`은 지정한 주체의 권한만 바꾼다. 다른 계정에 따로 준 권한이 남아 있으면 `icacls.exe $f /remove <계정>`으로 지운다
- 소유자가 본인인지도 확인한다. 다른 관리자 계정으로 연 PowerShell에서는 `whoami`가 접속 대상 계정이 아니다. 반드시 **접속 대상 계정**으로 실행한다

### 5.4 관리자 계정

**접속받는 PC**의 **관리자 PowerShell**에서 실행한다.

```powershell
$f = 'C:\ProgramData\ssh\administrators_authorized_keys'
Add-Content -Encoding ascii -LiteralPath $f -Value "ssh-ed25519 AAAA... laptop"
icacls.exe $f /inheritance:r /grant:r "*S-1-5-32-544:F" "*S-1-5-18:F"
icacls.exe $f                            # Administrators 와 SYSTEM 만 남아야 한다
```

- 5.3과 마찬가지로, 다른 계정에 따로 준 권한이 남아 있으면 `icacls.exe $f /remove <계정>`으로 지운다
- 이 파일은 **관리자 그룹 전원이 함께 쓴다.** 여기 넣은 키로는 그 PC의 어떤 관리자 계정으로도 들어갈 수 있다
- 일상 접속은 일반 계정으로 하는 편이 안전하다. 관리자 일은 접속 후 관리자 PowerShell을 따로 연다

### 5.5 키 접속 확인

**접속하는 기기**에서 시험한다. 지금은 비밀번호 로그인이 아직 켜져 있으므로, 비밀번호를 묻지 않는지 본다.

```powershell
ssh -o PreferredAuthentications=publickey <user>@laptop
```

- 접속되면 성공. `Permission denied (publickey)`이면 11절을 본다
- **데스크탑→노트북, 노트북→데스크탑 둘 다 확인한 뒤** 6절로 간다

출처: [OpenSSH 서버 설정](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh-server-configuration), [OpenSSH 키 관리](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement), [Win32-OpenSSH 파일 권한](https://github.com/PowerShell/Win32-OpenSSH/wiki/Security-protection-of-various-files-in-win32-openssh)

## 6. 비밀번호 로그인 차단 (두 PC)

**시작 전**: 5.5에서 키 접속을 확인했는지, PC 앞에 있거나 이미 연결된 SSH 세션이 하나 있는지 확인한다. 설정이 틀리면 원격으로 다시 들어갈 수 없다.

### 6.1 sshd_config 수정

관리자 권한으로 `C:\ProgramData\ssh\sshd_config`를 연다.

```powershell
notepad C:\ProgramData\ssh\sshd_config      # 관리자 PowerShell 에서
```

아래 두 줄을 넣는다. 이미 `#`으로 주석 처리된 줄이 있으면 `#`을 지우고 값을 바꾼다.

```
PubkeyAuthentication yes
PasswordAuthentication no
```

**위치 주의**: 파일 맨 끝에 다음 블록이 있다.

```
Match Group administrators
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

두 줄은 반드시 **이 `Match` 블록보다 위에** 넣는다. 아래에 쓰면 관리자 그룹에만 적용된다. `Match` 블록 자체는 지우지 않는다.

Windows OpenSSH는 password와 publickey 인증만 지원한다. Linux 가이드의 `KbdInteractiveAuthentication` 같은 옵션은 그대로 따라 넣지 않는다.

### 6.2 설정 검사와 재시작

```powershell
& "$env:WINDIR\System32\OpenSSH\sshd.exe" -t     # 아무것도 출력되지 않으면 정상
Restart-Service sshd
```

### 6.3 다시 확인

**새 연결**로 확인한다. 기존 세션은 닫지 않는다.

```powershell
ssh <user>@laptop                                              # 키로 접속되어야 한다
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password <user>@laptop
# → 비밀번호를 묻지 않고 Permission denied (publickey) 로 끝나야 한다
```

- 비밀번호를 묻는다면 차단이 적용되지 않은 것이다. 6.1의 위치(`Match` 블록보다 위)와 `Restart-Service sshd`를 다시 확인한다
- 서버가 어떤 인증 방식을 제시하는지 보려면 앞에 `-vvv`를 붙인다. 로그의 `Authentications that can continue: publickey`에 `password`가 없으면 된다

## 7. 안드로이드 앱 추천

### 7.1 비교

| 앱 | 가격 | 키 지원 | 상태 (2026-10) | 추천 |
| --- | --- | --- | --- | --- |
| **Termius** | Starter 무료 (SSH·SFTP·키 생성). 기기 간 동기화는 Pro 유료 | 앱 안에서 생성·가져오기 | 활발히 업데이트 | **처음 쓰는 사람** — 화면이 가장 편하다 |
| **ConnectBot** | 무료, 오픈소스 | Ed25519 생성·가져오기 | 활발히 개발 | **무료·오픈소스를 원하는 사람** |
| **Termux** | 무료, 오픈소스 | 일반 OpenSSH 그대로 (`ssh-keygen`) | 개발 지속. F-Droid·GitHub 배포 권장 | **명령행에 익숙한 사람** |
| JuiceSSH | — | — | Play 스토어에서 확인되지 않음 | 새로 설치하지 않는다 |

- 위 셋 모두 무료 범위에서 이 가이드의 구성이 된다
- Termius의 키 가져오기 메뉴 이름·지원 형식은 버전마다 다르다. 설치한 버전에서 직접 확인한다

출처: [Termius 요금](https://termius.com/pricing), [ConnectBot](https://connectbot.org/), [ConnectBot 릴리스](https://github.com/connectbot/connectbot/releases), [Termux](https://github.com/termux/termux-app)

### 7.2 Termius

1. **Keychain** → `+` → **Generate Key** → 종류 Ed25519, 이름 `phone`(태블릿은 `tablet`)
2. 만든 키 → 공개키 복사 → 5절대로 두 PC에 등록
3. **Hosts** → `+` → 호스트 추가
   - Label: `desktop`
   - Address: `desktop.<tailnet>.ts.net` (또는 100.x.y.z)
   - Port: `22`
   - Username: 5.1에서 확인한 이름
   - Key: 1에서 만든 키 선택 (Password는 비워 둔다)
4. 노트북도 같은 방법으로 `laptop` 호스트를 추가한다

### 7.3 ConnectBot

1. 메뉴 → **Manage Pubkeys** → `+` → 종류 Ed25519로 생성 (암호 설정 권장)
2. 키를 길게 눌러 **Copy public key** → 5절대로 두 PC에 등록
3. 첫 화면 주소창에 `<user>@desktop.<tailnet>.ts.net:22` 입력 → 접속
4. 호스트 설정에서 **Use pubkey authentication**에 1의 키를 지정한다

([ConnectBot 호스트 추가](https://connectbot.org/guides/add-host/))

### 7.4 Termux

```bash
pkg install openssh
ssh-keygen -t ed25519 -C "phone"
cat ~/.ssh/id_ed25519.pub          # 이 줄을 두 PC에 등록
ssh <user>@desktop
```

8.2의 `~/.ssh/config`도 Termux 안에서 그대로 쓸 수 있다.

## 8. 접속 방법 상세

### 8.1 데스크탑 ↔ 노트북

기본 형식:

```powershell
ssh <user>@laptop                        # 데스크탑에서 노트북으로
ssh <user>@desktop                       # 노트북에서 데스크탑으로
ssh <user>@100.x.y.z                     # 이름이 안 될 때
```

### 8.2 `~/.ssh/config`로 줄이기

`%USERPROFILE%\.ssh\config` 파일(확장자 없음)을 만든다. **데스크탑**의 예:

```
Host laptop
    HostName laptop.<tailnet>.ts.net
    User <user>
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
    ServerAliveInterval 30
    ServerAliveCountMax 3
```

이제 `ssh laptop`만 입력하면 된다. 노트북에는 `Host desktop` 항목을 같은 구조로 만든다.

| 옵션 | 뜻 |
| --- | --- |
| `IdentitiesOnly yes` | 지정한 키만 쓴다. 키가 여러 개일 때 엉뚱한 키로 시도하다 실패하는 것을 막는다 |
| `ServerAliveInterval 30` | 30초마다 연결 확인 신호를 보낸다 (기본 0 = 끔) |
| `ServerAliveCountMax 3` | 3번 응답이 없으면 연결을 끊는다 |

### 8.3 안드로이드 → 각 PC

1. **Tailscale이 연결돼 있는지** 먼저 본다 (상단에 VPN 아이콘)
2. SSH 앱에서 7절에 저장한 `desktop` 또는 `laptop` 호스트를 누른다
3. 처음 접속이면 호스트 키 확인 화면이 뜬다 (8.4)
4. PowerShell 프롬프트가 나오면 성공

**집 밖 접속 시험**: 폰의 Wi-Fi를 끄고 LTE만 켠 상태로 두 PC에 한 번씩 접속해 본다. PC를 재부팅한 뒤, 로그아웃한 뒤에도 한 번씩 시험한다.

### 8.4 첫 접속 때 호스트 키 지문 대조

처음 접속하면 `The authenticity of host ... can't be established` 와 지문(`SHA256:...`)이 뜬다. 이 지문이 접속하려는 PC의 것인지 대조한 뒤 `yes`를 입력한다.

접속받는 PC에서 지문 보기:

```powershell
ssh-keygen -lf C:\ProgramData\ssh\ssh_host_ed25519_key.pub
```

한 번 받아들이면 `known_hosts`에 저장되고 다음부터는 묻지 않는다. 나중에 **지문이 바뀌었다는 경고**가 뜨면 접속하지 말고 원인(PC 재설치 등)을 먼저 확인한다.

출처: [ssh_config(5)](https://man.openbsd.org/ssh_config), [ssh-keygen(1)](https://man.openbsd.org/ssh-keygen)

## 9. 연결 유지와 전원

### 9.1 keepalive와 세션 유지는 다르다

- keepalive(8.2)는 쉬고 있는 연결이 끊기지 않게 하고, 죽은 연결을 빨리 알아채게 한다
- **끊어진 세션을 되살리지는 않는다.** Windows에는 tmux가 없어서, 연결이 끊기면 그 PowerShell 세션과 그 안에서 돌던 명령도 같이 끝난다
- 휴대폰은 네트워크가 자주 바뀐다. 오래 걸리는 작업은 접속을 끊어도 계속 돌도록 따로 띄운다. 예) `Start-Process` 로 별도 프로세스 실행, 또는 작업 스케줄러

### 9.2 PC 절전 끄기

**절전·최대 절전 중인 PC에는 Tailscale로도 접속할 수 없다.** 원격으로 깨우는 기능도 없다. 원격으로 쓸 PC는 전원 연결 상태에서 절전을 끈다.

```powershell
powercfg /change standby-timeout-ac 0      # 전원 연결 시 자동 절전 타이머: 사용 안 함
powercfg /change hibernate-timeout-ac 0    # 전원 연결 시 자동 최대 절전 타이머: 사용 안 함
```

- 값의 단위는 분이고, 0은 "사용 안 함"이다. **현재 쓰는 전원 구성표에만** 적용된다
- 이 두 명령은 **유휴 타이머만** 바꾼다. 덮개·전원 버튼·배터리 부족·수동으로 들어가는 절전은 그대로다. 최대 절전 타이머를 0으로 해도 Windows의 적응형 최대 절전이 따로 동작할 수 있다
- 적용 결과는 설정 → 시스템 → 전원에서 확인한다
- 화면 꺼짐(monitor-timeout)은 꺼져도 괜찮다
- 노트북은 덮개를 닫을 때의 동작(제어판 → 전원 옵션 → 덮개를 닫으면 수행되는 작업)도 전원 연결 시 "아무 것도 안 함"으로 바꾼다

### 9.3 안드로이드

- Always-on VPN을 켠다 (2.4)
- 제조사 배터리 최적화가 Tailscale을 끄면, 설정 → 앱 → Tailscale → 배터리 → **제한 없음**으로 바꾼다 (메뉴는 제조사마다 다르다)

출처: [Modern Standby 네트워크](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/networking-power-management-for-modern-standby-platforms), [Tailscale 모바일 배터리](https://tailscale.com/docs/reference/troubleshooting/mobile/battery-drains)

## 10. 보안 강화 (선택)

여기까지로도 SSH 포트는 인터넷에 열려 있지 않다. 아래는 더 좁히고 싶을 때 한다.

### 10.1 Tailscale grants로 22번 포트만 허용

새 tailnet의 기본 정책은 tailnet 안 모든 기기 사이의 통신을 허용한다. 두 PC에 태그를 붙여 SSH만 허용하도록 좁힐 수 있다.

1. 관리 콘솔 → **Access controls**에서 태그 소유자를 정하고, Machines에서 두 PC에 `tag:sshserver` 태그를 붙인다
2. 정책에 grant를 넣는다

```json
{
  "tagOwners": { "tag:sshserver": ["autogroup:admin"] },
  "grants": [
    { "src": ["autogroup:member", "tag:sshserver"], "dst": ["tag:sshserver"], "ip": ["tcp:22"] }
  ]
}
```

- **`src`에 `tag:sshserver`도 넣는다.** 태그가 붙은 기기는 사용자 소유가 아니게 되어 `autogroup:member`에 들어가지 않는다. 빼면 폰·태블릿 → PC는 되지만 **PC ↔ PC 접속이 막힌다**
- `tagOwners`는 태그를 붙일 권한만 정한다. 태그는 Machines에서 직접 붙여야 한다
- 태그를 붙인 기기는 키 만료가 기본으로 꺼진다
- 정책은 허용한 것만 통한다(deny-by-default). 이 파일로 기본 정책(전체 허용)을 바꾸면 **22번 외의 tailnet 통신은 모두 막힌다.** 파일 공유(Taildrop 등) 같은 다른 통신이 필요하면 그 규칙도 넣는다
- 허용 규칙은 **합쳐진다.** 기존의 전체 허용 규칙을 남겨 두면 제한되지 않는다
- 이 정책은 네트워크 접근만 정한다. Windows 계정 인증(키)은 따로 거친다. 일반 OpenSSH를 쓰므로 Tailscale SSH용 `ssh` 정책은 필요 없다

([Grants](https://tailscale.com/kb/1324/grants), [grants 문법](https://tailscale.com/kb/1538/grants-syntax), [정책 문법](https://tailscale.com/kb/1337/policy-syntax), [Tags](https://tailscale.com/kb/1068/tags))

### 10.2 Windows 방화벽에서 Tailscale 주소만 허용

Tailscale 정책은 집 LAN에서의 직접 접속까지 막지는 않는다. 같은 Wi-Fi의 다른 기기도 막으려면 SSH 방화벽 규칙의 원격 주소를 Tailscale 범위로 좁힌다.

```powershell
Set-NetFirewallRule -Name OpenSSH-Server-In-TCP -RemoteAddress 100.64.0.0/10
```

이렇게 하면 LAN 주소(192.168.x.x)로는 SSH 접속이 안 되고 Tailscale 이름·주소로만 된다. ([Set-NetFirewallRule](https://learn.microsoft.com/en-us/powershell/module/netsecurity/set-netfirewallrule))

### 10.3 휴대폰·태블릿을 잃어버렸을 때

1. 관리 콘솔 Machines에서 그 기기를 **Remove** 한다 → tailnet에서 즉시 빠진다
2. 두 PC의 `authorized_keys`(또는 `administrators_authorized_keys`)에서 그 기기의 공개키 줄을 지운다. 4절에서 키마다 `-C "phone"`처럼 이름을 붙여 둔 것이 여기서 쓰인다
3. 새 기기에서는 새 키를 만든다. 예전 개인키를 옮겨 쓰지 않는다

평소에 앱 잠금이나 키 암호를 걸어 두면 잃어버렸을 때의 위험이 줄어든다.

## 11. 문제 해결

### 11.1 증상별 점검

| 증상 | 점검 순서 |
| --- | --- |
| `Connection timed out` | 1) 양쪽 `tailscale status`에서 상대가 online인지 2) 상대 PC가 절전 중인지 3) 안드로이드면 Tailscale이 켜져 있는지, 다른 VPN이 켜져 있지 않은지 4) 10.1 grants로 막지 않았는지 |
| `Connection refused` | 1) `Get-Service sshd`가 Running인지 2) `Get-NetTCPConnection -LocalPort 22 -State Listen` 3) 방화벽 규칙이 켜져 있는지 (3.3) |
| 키가 있는데 비밀번호를 묻는다 / `Permission denied (publickey)` | 1) 관리자 계정인데 `authorized_keys`에 넣지 않았는지 (5.1) 2) 파일 권한 (5.3·5.4의 `icacls` 결과) 3) 공개키가 한 줄인지, 줄이 깨지지 않았는지 4) 파일 이름에 `.txt`가 붙지 않았는지 5) 사용자 이름이 맞는지 (`whoami`) |
| 개인키 암호문구를 묻는다 | 정상이다. 매번 묻지 않게 하려면 4.2 ssh-agent |
| 호스트 키가 바뀌었다는 경고 | 접속하지 말고 원인을 먼저 확인한다 (8.4). PC를 재설치했다면 `ssh-keygen -R <호스트>`로 예전 기록을 지우고, 새 지문을 8.4대로 다시 대조한다. `laptop`, `laptop.<tailnet>.ts.net`, `100.x.y.z`는 따로 기록되므로 접속에 쓴 이름마다 지운다 |
| 재부팅·로그아웃 뒤 접속 안 됨 | 1) Run unattended (2.2) 2) sshd 자동 시작 (3.2) 3) 키 만료 (2.3) |
| 몇 달 뒤 갑자기 안 됨 | Tailscale 키 만료 (2.3). 관리 콘솔에서 기기 상태 확인 |
| 자주 끊긴다 | keepalive (8.2), 안드로이드 배터리 제한 (9.3). `tailscale status`에 relay로 나오는 것은 정상 연결 방식이고 그 자체로 고장이 아니다 |

### 11.2 디버그 방법

**클라이언트 쪽 자세한 로그**:

```powershell
ssh -vvv <user>@laptop
```

어떤 키를 내밀었고 서버가 무엇을 거절했는지 보인다.

**서버 쪽**: 관리자 PowerShell에서 서비스를 멈추고 디버그 모드로 직접 띄운다. PC 앞에서만 한다.

```powershell
Stop-Service sshd                                  # 먼저 멈춰야 한다. 안 그러면 22번 포트가 사용 중이라 실패한다
& "$env:WINDIR\System32\OpenSSH\sshd.exe" -d      # 연결 하나를 처리하며 로그를 보여 주고, 그 연결이 끝나면 종료한다
Start-Service sshd                                 # 끝나면 반드시 다시 켠다
```

이 방식으로는 **지금 PC에 로그인한 사용자의 키 접속만** 시험할 수 있다. 서비스와 똑같은 조건(다른 계정 등)으로 진단하려면 SYSTEM 권한으로 실행해야 한다 ([Win32-OpenSSH 문제 해결](https://github.com/PowerShell/Win32-OpenSSH/wiki/Troubleshooting-Steps)).

**로그 파일로 남기기**: `sshd_config`에 넣고 `Restart-Service sshd`. 확인이 끝나면 지운다.

```
SyslogFacility LOCAL0
LogLevel DEBUG3
```

로그는 `C:\ProgramData\ssh\logs\sshd.log`에 쌓인다. 이벤트 뷰어 → 응용 프로그램 및 서비스 로그 → OpenSSH 에서도 볼 수 있다.

**Tailscale 쪽**:

```powershell
tailscale status
tailscale ping laptop          # 상대에게 닿는지, direct 인지 relay 인지
```

출처: [Win32-OpenSSH 문제 해결](https://github.com/PowerShell/Win32-OpenSSH/wiki/Troubleshooting-Steps), [OpenSSH 문제 진단](https://learn.microsoft.com/en-us/troubleshoot/windows-server/system-management-components/troubleshoot-sftp-issues-using-openssh), [Tailscale 연결 방식](https://tailscale.com/docs/reference/connection-types), [Tailscale 모바일 문제 해결](https://tailscale.com/docs/reference/troubleshooting/mobile)

### 11.3 직접 확인이 필요한 것

자료 조사로 확정하지 못한 부분이다. 자기 환경에서 확인한다.

- Tailscale 네트워크가 Windows 방화벽의 어느 프로필(공용·개인)에 속하는지, SSH 규칙이 그 프로필에 적용되는지: `Get-NetConnectionProfile`, 3.3의 `Profile` 열
- Windows 빌드마다 기본 OpenSSH 버전과 지원 옵션이 다르다: `ssh -V`
- Termius·ConnectBot의 키 가져오기 메뉴 이름과 지원 형식
- 안드로이드 제조사별 배터리 제한 메뉴

## 12. 부록

### 12.1 명령 모음 (두 PC, 관리자 PowerShell)

```powershell
# Tailscale
tailscale up --unattended
tailscale status
tailscale ip -4

# OpenSSH 설치·시작
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
Get-NetFirewallRule -Name OpenSSH-Server-In-TCP

# 기본 셸
New-ItemProperty -Path 'HKLM:\SOFTWARE\OpenSSH' -Name DefaultShell `
  -Value 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' -PropertyType String -Force

# ssh-agent
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent

# 비밀번호 차단 후 (sshd_config 수정)
& "$env:WINDIR\System32\OpenSSH\sshd.exe" -t
Restart-Service sshd

# 절전 끄기
powercfg /change standby-timeout-ac 0
powercfg /change hibernate-timeout-ac 0
```

### 12.2 설정 점검표

| 항목 | 데스크탑 | 노트북 | 폰 | 태블릿 |
| --- | --- | --- | --- | --- |
| Tailscale 로그인 (같은 계정) | ☐ | ☐ | ☐ | ☐ |
| Run unattended | ☐ | ☐ | — | — |
| 키 만료 끔 | ☐ | ☐ | 켜 둠 | 켜 둠 |
| Always-on VPN | — | — | ☐ | ☐ |
| OpenSSH Server 자동 시작 | ☐ | ☐ | — | — |
| 기본 셸 PowerShell | ☐ | ☐ | — | — |
| 자기 키 만듦 | ☐ | ☐ | ☐ | ☐ |
| 다른 기기 공개키 등록 | ☐ | ☐ | — | — |
| 키 접속 확인 후 비밀번호 차단 | ☐ | ☐ | — | — |
| 전원 연결 시 절전 끔 | ☐ | ☐ | — | — |
| LTE에서 두 PC 접속 확인 | — | — | ☐ | ☐ |

### 12.3 출처

- Microsoft Learn: [OpenSSH 설치·최초 사용](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse) · [OpenSSH 서버 설정](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh-server-configuration) · [OpenSSH 키 관리](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement) · [OpenSSH 문제 진단](https://learn.microsoft.com/en-us/troubleshoot/windows-server/system-management-components/troubleshoot-sftp-issues-using-openssh) · [Set-NetFirewallRule](https://learn.microsoft.com/en-us/powershell/module/netsecurity/set-netfirewallrule) · [powercfg](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/powercfg-command-line-options) · [적응형 최대 절전](https://learn.microsoft.com/en-us/windows-hardware/customize/power-settings/adaptive-hibernate)
- Win32-OpenSSH 소스: [파일 권한 검사](https://github.com/PowerShell/openssh-portable/blob/latestw_all/contrib/win32/win32compat/w32-sshfileperm.c)
- Win32-OpenSSH: [문제 해결](https://github.com/PowerShell/Win32-OpenSSH/wiki/Troubleshooting-Steps) · [파일 권한](https://github.com/PowerShell/Win32-OpenSSH/wiki/Security-protection-of-various-files-in-win32-openssh)
- OpenSSH man page: [ssh_config(5)](https://man.openbsd.org/ssh_config) · [sshd_config(5)](https://man.openbsd.org/sshd_config) · [ssh-keygen(1)](https://man.openbsd.org/ssh-keygen)
- Tailscale: [Windows 설치](https://tailscale.com/docs/install/windows) · [Android 설치](https://tailscale.com/docs/install/android) · [MagicDNS](https://tailscale.com/kb/1081/magicdns) · [Key expiry](https://tailscale.com/kb/1028/key-expiry) · [Run unattended](https://tailscale.com/docs/how-to/run-unattended) · [Tailscale SSH](https://tailscale.com/kb/1193/tailscale-ssh) · [Grants](https://tailscale.com/kb/1324/grants) · [grants 문법](https://tailscale.com/kb/1538/grants-syntax) · [정책 문법](https://tailscale.com/kb/1337/policy-syntax) · [Tags](https://tailscale.com/kb/1068/tags) · [CLI](https://tailscale.com/docs/reference/tailscale-cli) · [연결 방식](https://tailscale.com/docs/reference/connection-types) · [다른 VPN과 병용](https://tailscale.com/docs/reference/faq/other-vpns)
- 안드로이드 앱: [Termius 요금](https://termius.com/pricing) · [ConnectBot](https://connectbot.org/) · [ConnectBot 릴리스](https://github.com/connectbot/connectbot/releases) · [Termux](https://github.com/termux/termux-app)
