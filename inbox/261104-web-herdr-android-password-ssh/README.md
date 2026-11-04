# herdr 안드로이드 앱 → 데스크탑 SSH 접속 (Tailscale)

- 작성: 2026-11-04, 데스크탑 Claude Code 세션에서 확인·정리
- 관련 위키: `wiki/dev/web/ssh-tailscale-remote-terminal.md`
- 참고: https://herdr.dev/llms.txt, persistence-remote.mdx, windows-beta.mdx (herdr v0.9.5 문서, 2026-11-04) — 안드로이드 앱 이야기는 공식 문서에 없음

## 1. 접속 정보 (데스크탑)

| 항목 | 값 |
| --- | --- |
| Host (IP) | `100.87.213.81` |
| Host (MagicDNS) | `desktop.taila5bb9e.ts.net` |
| Port | `22` (OpenSSH sshd, Running / Automatic) |
| User | `nampl` (`desktop\nampl`) |

- `sshd_config`에 `Port` 줄이 없어서 기본값 22를 쓴다. sshd는 `0.0.0.0:22`, `[::]:22`에서 기다린다.
- 폰에서도 Tailscale이 켜져 있고 같은 tailnet에 로그인되어 있어야 한다.
- 확인 명령: `tailscale ip -4`, `(tailscale status --json | ConvertFrom-Json).Self.DNSName`, `Get-NetTCPConnection -State Listen | ? LocalPort -eq 22`

## 2. 증상: Termius는 되는데 herdr 앱은 안 됨

sshd 로그(`Get-WinEvent -LogName OpenSSH/Operational`, 폰 IP `100.81.176.26`):

```
오전 9:12  Accepted publickey for nampl ... ED25519      ← Termius
오후 3:17  Connection closed by 100.81.176.26 [preauth]  ← herdr 앱
오후 3:19  Connection closed by 100.81.176.26 [preauth]  ← herdr 앱
```

- `[preauth]`는 로그인하기 전에 연결이 끊겼다는 뜻이다. 네트워크·Tailscale·포트는 문제가 없고, **인증에서 막혔다**.
- 데스크탑 `sshd_config`는 `PasswordAuthentication no`라서 공개키 접속만 받는다.
- Termius는 앱 안에서 키를 만들 수 있고, 그 공개키가 데스크탑에 등록되어 있어 접속된다.
- 이 계정은 관리자 그룹이라 `Match Group administrators` → `C:\ProgramData\ssh\administrators_authorized_keys` 파일에 등록된 키만 받는다.
- herdr 안드로이드 앱의 SSH connection 화면에는 **비밀번호 입력칸만** 있다. 키를 만들거나 붙여넣는 기능을 찾지 못했다.

## 3. 해결: Tailscale 대역에서만 비밀번호 허용

관리자 권한으로 `C:\ProgramData\ssh\sshd_config`를 열고, **`Match Group administrators` 줄 바로 위에** 아래를 넣는다. Match 블록은 다음 Match 줄까지 이어지므로 맨 끝에 넣으면 안 된다.

```
Match Address 100.64.0.0/10
       PasswordAuthentication yes
```

그다음 `sshd -t`로 문법을 검사하고 `Restart-Service sshd`를 실행한다.

- `100.64.0.0/10`은 Tailscale 주소 대역이다. tailnet 기기에서 오는 접속만 비밀번호가 허용되고, 그 밖의 접속은 계속 키 인증만 받는다.
- 비밀번호는 Windows 로그인 PIN이 아니라 **Microsoft 계정 비밀번호**(로컬 계정이면 로컬 비밀번호)다.

자동 스크립트(백업 → 블록 삽입 → `sshd -t` 검사, 실패하면 되돌림 → sshd 재시작)를 관리자 창에서 실행하는 방식:

```powershell
$cfg = 'C:\ProgramData\ssh\sshd_config'
$bak = "$cfg.bak-$(Get-Date -Format yyyyMMddHHmmss)"
Copy-Item $cfg $bak
$text = Get-Content $cfg -Raw
if ($text -notmatch 'Match Address 100\.64\.0\.0/10') {
    $block = "Match Address 100.64.0.0/10`r`n       PasswordAuthentication yes`r`n`r`n"
    $text = $text -replace '(?m)^Match Group administrators', ($block + 'Match Group administrators')
    Set-Content $cfg $text -NoNewline -Encoding ascii
}
& "$env:WINDIR\System32\OpenSSH\sshd.exe" -t
if ($LASTEXITCODE -ne 0) { Copy-Item $bak $cfg -Force } else { Restart-Service sshd }
```

- Claude Code 자동 모드는 보안을 약하게 만드는 변경이라며 이 실행을 막았다. 사람이 직접 실행해야 한다.

## 4. herdr 앱 설정

- Host `100.87.213.81` (또는 `desktop.taila5bb9e.ts.net`)
- Port `22`
- User `nampl`
- Password: Microsoft 계정 비밀번호

## 5. 다른 방법 (앱에 키 입력칸이 있을 때)

데스크탑에서 앱 전용 키를 만들어 공개키를 등록하고, 개인키를 앱에 붙여넣는다(관리자 창).

```powershell
ssh-keygen -t ed25519 -f $HOME\.ssh\herdr_android -N '""' -C herdr-android
Get-Content $HOME\.ssh\herdr_android.pub | Add-Content C:\ProgramData\ssh\administrators_authorized_keys
```

## 6. 확인 안 된 것

- 2026-11-04 기록 시점에는 3절의 설정을 아직 실행하지 않았다. 바꾼 뒤 herdr 앱으로 접속에 성공했는지는 확인하지 않았다.
- herdr 앱이 접속한 뒤 Windows 호스트(기본 셸 `powershell.exe`, herdr는 사용자 PATH에만 있음)에서 herdr 서버에 제대로 붙는지는 확인하지 않았다.
