# herdr 안드로이드 앱 → 데스크탑 SSH 접속 (Tailscale)

- 작성: 2026-11-04, 데스크탑 Claude Code 세션에서 확인·정리
- 관련 위키: `wiki/dev/web/ssh-tailscale-remote-terminal.md`
- 참고: https://herdr.dev/llms.txt, persistence-remote.mdx, windows-beta.mdx (herdr v0.9.5 문서, 2026-11-04) — 안드로이드 앱 이야기는 공식 문서에 없음

> **결론 (2026-11-04 시험)**: 아래 3절 설정에 키 교환 조정과 Microsoft 계정 비밀번호 갱신까지 더해서 탭(`tab`)의 herdr 앱으로 SSH 로그인에는 성공했다. 그러나 앱은 로그인 직후 리눅스 셸 명령으로 Herdr Bridge를 찾고, 데스크탑 기본 셸은 PowerShell이라 이 확인이 실패한다. 앱 오류는 `authenticated, but could not detect the Herdr Bridge install state on the server`이다. **herdr 안드로이드 앱은 아직 Windows 데스크탑을 지원하지 않는 것으로 보고, sshd 설정은 원래대로 되돌렸다.** 자세한 내용은 6절에 있다.

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
- 관리자 창이 아닌 곳에서 실행하면 첫 줄 `Copy-Item`에서 실패하고, 아무것도 바뀌지 않는다. 백업 파일 `sshd_config.bak-<시각>`이 생기지 않았다면 실행되지 않은 것이다.
- Claude가 실행할 때는 스크립트를 파일로 저장한 뒤 `Start-Process pwsh -Verb RunAs -Wait -ArgumentList '-NoProfile','-ExecutionPolicy','Bypass','-File',<스크립트>`로 띄웠고, 사람이 UAC 창에서 승인했다.

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

## 6. 시험 결과 (2026-11-04, 탭 → 데스크탑)

접속한 기기는 폰이 아니라 탭이다. `tailscale status`에서 이름은 `tab`, 주소는 `100.81.176.26`이다(위 2절 로그의 IP도 탭이다). 탭에서는 Termius로 공개키 접속이 정상적으로 된다. herdr 앱이 쓰는 SSH 라이브러리는 sshd 로그에 나온 대로 `SSHJ_0.39.0`이다.

막힌 순서대로 적는다.

| 단계 | 앱/로그에 보인 것 | 원인 | 조치 |
| --- | --- | --- | --- |
| 1. 인증 방식 | `Connection closed ... [preauth]` | `PasswordAuthentication no` | 3절 `Match Address 100.64.0.0/10` 블록 적용 |
| 2. 키 교환 | 앱: `SSH key exchange failed - check the server algorithm`. 로그: `kex: algorithm: curve25519-sha256` 협상 뒤 `expecting SSH2_MSG_KEX_ECDH_INIT`에서 16ms 만에 끊김 | 앱(sshj)이 curve25519를 지원한다고 제시해 놓고, 실제 계산 단계에서 실패한다(추정: 안드로이드에서 X25519 구현 문제) | 맨 앞에 `KexAlgorithms -curve25519-sha256,curve25519-sha256@libssh.org`를 넣었다(전역 설정이라 `Match` 안에 넣을 수 없다). 그러자 앱과 `diffie-hellman-group-exchange-sha256`으로 협상됐다. 호스트 키 `ssh-ed25519`는 그대로 써도 됐다 |
| 3. 비밀번호 | `Windows authentication failed ... error: 1326`, `Failed password for nampl` | `nampl`은 Microsoft 계정이고, "Windows Hello 로그인만 허용"이 켜져 있었다(`HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\PasswordLess\Device` `DevicePasswordLessBuildVersion = 2`). 이러면 PC에 저장된 비밀번호가 옛것으로 남아, 맞는 비밀번호도 거부된다 | 설정 → 계정 → 로그인 옵션에서 이 옵션을 끄고(값이 `0`이 됨), Win+L로 잠근 뒤 Microsoft 계정 비밀번호로 한 번 로그인해 저장된 비밀번호를 갱신했다. 그 뒤 `Accepted password for nampl` |
| 4. herdr 확인 | 앱: `authenticated, but could not detect the Herdr Bridge install state on the server` | 앱이 로그인 직후 `exec`로 POSIX 셸 명령을 보낸다. `uname -s`, `[ -x ~/.local/bin/herdr-mobile-bridge ]`, `~/.local/bin/herdr-mobile-bridge sessions --json`, `~/.config/herdr/sessions/*/herdr.sock`·`~/.config/herdr/herdr.sock` 소켓 찾기, `$HOME/.local/bin/herdr` 등에서 herdr 찾기다. 이 명령이 데스크탑 기본 셸 `powershell.exe -c`로 실행되어 `exit 1` | 해결하지 않았다. 앱은 리눅스·macOS 호스트와 `herdr-mobile-bridge`를 전제로 하는 것으로 보인다 |

진단 방법:

- 이벤트 로그(`OpenSSH/Operational`)에는 `LogLevel DEBUG3`을 켜도 디버그 줄이 남지 않았다. `SyslogFacility LOCAL0`을 함께 넣으면 `C:\ProgramData\ssh\logs\sshd.log`에 남는다. 이 폴더는 관리자만 읽을 수 있어서 관리자 권한으로 복사해서 읽었다.
- 서버가 제시하는 인증 방식은 데스크탑 자신에게 Tailscale IP로 접속해서 확인했다: `ssh -v -o BatchMode=yes -o PreferredAuthentications=none nampl@100.87.213.81 exit` → `Authentications that can continue: ...`.
- 중간에 한 번 앱에 timeout이 나면서 sshd 로그에 접속 기록 자체가 없었다. 다시 시도하자 재현되지 않았다.

마무리:

- `sshd_config`는 시험 전 백업(`sshd_config.bak-20261104183149`)으로 되돌리고 sshd를 재시작했다. 비밀번호 차단, curve25519, 상세 로그 모두 원래 상태다(같은 2294바이트이고, 다시 `curve25519-sha256`으로 협상된다).
- "Windows Hello 로그인만 허용"은 꺼진 채로 두었다. 다시 켤지는 사람이 정한다.
- `C:\ProgramData\ssh\`에 진단 중 만든 `.bak-*` 파일과 `logs\sshd.log`가 남아 있다.

## 7. 확인 안 된 것

- Herdr Bridge(`herdr-mobile-bridge`)가 Windows를 지원하지 않는다는 판단은 공식 문서에서 확인한 것이 아니다. 앱이 보낸 명령과 공식 문서(v0.9.5 llms.txt)에 관련 언급이 없다는 점으로 추정했다.
- curve25519 실패 원인(안드로이드의 X25519 구현)은 추정이다.
- WSL에 herdr와 sshd를 두고 접속하는 방법은 시험하지 않았다.
- 원래 설정으로 되돌린 뒤 탭의 Termius 접속은 다시 확인하지 않았다.
