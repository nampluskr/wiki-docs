# R1-01 — Termius를 대신할 안드로이드 SSH 터미널 앱 조사

- 역할: worker-N (병렬 동일 작업. 다른 worker의 결과는 보지 않는다)
- 작업 폴더: `D:\projects`
- 고칠 수 있는 파일: `D:\projects\.herdr\261116-android-ssh-client-research\<자기 에이전트 이름>\` 폴더 안만. 그 밖은 읽기만 한다.
  - 예: `p-compare-1`은 `D:\projects\.herdr\261116-android-ssh-client-research\p-compare-1\`

## 배경

사용자는 안드로이드 폰·태블릿에서 Tailscale + Termius로 집 데스크탑(Windows 11)에 SSH 접속해 왔다.
Termius 무료 사용 기간이 끝나 더 쓸 수 없다. Termius를 대신할 안드로이드 SSH 터미널 앱을 제안해야 한다.

**먼저 위키의 아래 두 문서를 읽고 지금 구성을 파악한다.** 조사와 제안은 이 구성을 기준으로 한다.

1. `D:\projects\wiki-docs\wiki\dev\web\ssh-tailscale-remote-terminal.md` — 전체 구성(Tailscale, Windows OpenSSH, SSH 키, 비밀번호 차단, 7절 안드로이드 앱 표)
2. `D:\projects\wiki-docs\inbox\261104-web-herdr-android-password-ssh\README.md` — 데스크탑 접속 정보, herdr 안드로이드 앱이 안 된 이유

위키에서 확인할 핵심(요약. 원문이 우선):
- 접속받는 쪽: Windows 11 기본 OpenSSH sshd, 포트 22, 기본 셸 PowerShell, Tailscale MagicDNS 이름으로 접속
- 인증: **공개키만**(`PasswordAuthentication no`). 접속하는 기기마다 Ed25519 키를 따로 만들고 공개키를 데스크탑 `administrators_authorized_keys`에 등록
- 앱 안에서 Ed25519 키를 만들고 공개키를 내보낼 수 있어야 한다(Termius는 Keychain에서 했다)
- 위키 7절에 ConnectBot, Termux가 후보로 있고 JuiceSSH는 Play 스토어에서 확인되지 않았다고 적혀 있다

## 할 일

1. 위 두 위키 문서를 읽는다.
2. 웹에서 2026년 11월 기준 정보를 조사해 안드로이드 SSH 터미널 앱 후보를 찾는다(위키의 ConnectBot·Termux 포함, 그 밖의 앱도).
3. 후보마다 아래를 확인한다. 확인하지 못한 항목은 "미확인"으로 적는다.
   - 가격과 무료 범위(구독인지, 일회 결제인지, 무료로 막히는 기능)
   - Play 스토어(또는 F-Droid) 배포 여부, 최근 업데이트 시점, 오픈소스 여부
   - 앱 안에서 Ed25519 키 생성, 공개키 내보내기, 기존 키 가져오기
   - Windows OpenSSH + PowerShell 셸 접속에 알려진 문제
   - 터미널 품질: TUI 표시(색, 마우스), 특수키(Ctrl, Esc, Tab, 화살표) 입력, 한글 입력·표시, 세션 유지(백그라운드, 재연결)
   - 폰과 태블릿(폴더블 포함) 화면 대응, 외부 키보드
4. 비교표를 만들고, 이 사용자에게 맞는 추천 1순위와 2순위를 이유와 함께 고른다. 옮겨 가는 절차(키 생성 → 공개키 등록 → 호스트 등록)를 1순위 앱 기준으로 짧게 적는다.

## 완료 조건

- `<자기 폴더>\REPORT.md`에 비교표, 추천(1·2순위와 이유), 1순위 앱 이전 절차, 출처(URL과 확인 날짜)가 있다.
- 사실마다 출처가 있다. 추정은 "(추정)"으로 표시한다.
- `<자기 폴더>\R1-01-report.md`가 있다.

## 하지 않는 것

- 커밋, push, 삭제. 자기 폴더 밖 파일 수정(위키 문서도 고치지 않는다).
- 다른 worker 폴더를 읽거나 고치기.
- 데스크탑의 sshd 설정 변경, 앱 설치, 계정 가입.
- 백그라운드 실행. herdr 명령 호출.

## 보고

`<자기 폴더>\R1-01-report.md`에 아래 형식으로, 한국어로 쓰고, 답으로 그 경로만 말한다.

```markdown
# R1-01 보고

- 결과: 완료 / 일부 완료 / 막힘
- 바꾼 파일: 없음
- 만든 파일: <경로 목록>
- 검증: <출처 확인 방법. 하지 않았으면 "하지 않음">
- 막힌 점: <없으면 "없음">
- 확인이 필요한 점: <추정한 것, 카드와 다르게 한 것. 없으면 "없음">
```
