---
tags: [rust, gui, windows, install, guide]
created: 2026-10-31
updated: 2026-10-31
---

# 01 · Rust 설치 (Windows 11)

> 이 파일을 읽는 때: Rust를 처음 설치할 때, 빌드한 앱이 실행되지 않을 때.

입구는 [README.md](README.md)다.

| 항목 | 내용 |
| --- | --- |
| 결과 | MSVC 툴체인의 Rust, rustfmt·clippy·rust-analyzer, Zed에서 Rust 코드 편집 |
| 기준 | Windows 11 x64, winget, rustup 1.29.1, rustc·cargo 1.99.1 |
| 검증 | 이 노트북에서 아래 순서대로 설치하고 `cargo run`으로 "Hello, world!"를 확인했다 |

## 1. 설치 순서 요약

| 단계 | 하는 일 | 이유 |
| --- | --- | --- |
| 1 | Visual Studio 2022 Build Tools (C++ 워크로드) | Windows용 Rust(MSVC 툴체인)는 MSVC 링커 `link.exe`와 Windows SDK를 쓴다 |
| 2 | rustup | Rust 툴체인 설치·관리 도구 |
| 3 | 구성 요소 추가 | 포매터·린터·언어 서버 |
| 4 | 스마트 앱 컨트롤 확인 | 켜져 있으면 직접 빌드한 `.exe`가 실행되지 않는다 |
| 5 | Zed 재시작과 확인 | 새 PATH를 읽게 한다 |

명령은 모두 PowerShell에서 실행한다.

## 2. Visual Studio 2022 Build Tools

```powershell
winget install --id Microsoft.VisualStudio.2022.BuildTools -e `
  --accept-package-agreements --accept-source-agreements `
  --override "--quiet --wait --norestart --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"
```

- `VCTools` 워크로드에 `--includeRecommended`를 붙이면 MSVC 컴파일러·링커와 Windows SDK가 같이 들어간다.
- 2~5GB를 내려받는다. 관리자 권한(UAC) 창이 뜨면 승인한다.
- 이 옵션은 **CMake도 같이 설치한다**. gpui-kit 빌드에 필요하다 ([02-GPUI.md](02-GPUI.md) 2절). 위치는 아래와 같고, PATH에는 들어가지 않는다.

```
C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin
```

설치 확인 (MSVC와 SDK 폴더가 보이면 된다)

```powershell
Get-ChildItem "${env:ProgramFiles(x86)}\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC" -Name
Get-ChildItem "${env:ProgramFiles(x86)}\Windows Kits\10\Lib" -Name
```

## 3. rustup

```powershell
winget install --id Rustlang.Rustup -e --accept-package-agreements --accept-source-agreements
```

- 기본 툴체인 `stable-x86_64-pc-windows-msvc`가 함께 설치된다.
- `%USERPROFILE%\.cargo\bin`이 사용자 PATH에 추가된다. **이미 열린 터미널에는 반영되지 않는다.** 새 터미널을 열거나 6절처럼 Zed를 재시작한다.

## 4. 구성 요소

```powershell
rustup default stable
rustup component add rustfmt clippy rust-analyzer rust-src
```

| 구성 요소 | 용도 |
| --- | --- |
| rustfmt | 코드 포매터 (`cargo fmt`) |
| clippy | 린터 (`cargo clippy`) |
| rust-analyzer | 언어 서버. Zed가 자동 완성·정의 이동에 쓴다 |
| rust-src | 표준 라이브러리 소스. rust-analyzer가 표준 라이브러리를 분석할 때 쓴다 |

## 5. 스마트 앱 컨트롤

빌드는 되는데 실행에서 아래 오류가 나면 스마트 앱 컨트롤 때문이다.

```
error: could not execute process `target\debug\hello_rust.exe` (never executed)
Caused by:
  애플리케이션 제어 정책에서 이 파일을 차단했습니다. (os error 4551)
```

- 스마트 앱 컨트롤은 서명되지 않은 프로그램을 막는다. 직접 빌드한 `.exe`는 서명이 없어서 폴더와 상관없이 막힌다. 크레이트의 빌드 스크립트(`build.rs`)도 실행 파일이라 함께 막힌다.
- 상태 확인: `(Get-MpComputerStatus).SmartAppControlState` → `On`이면 막힌다.
- 끄는 곳: Windows 보안 → 앱 및 브라우저 컨트롤 → 스마트 앱 컨트롤 설정 → **끄기**.
- 보안 설정이므로 본인이 판단한다. Windows 빌드에 따라 한 번 끄면 Windows를 다시 설치하기 전에는 켤 수 없는 경우가 있다.

## 6. Zed 설정과 확인

1. **Zed를 완전히 종료하고 다시 연다.** Zed 안의 터미널은 Zed가 시작될 때의 PATH를 물려받는다.
2. Herdr 같은 터미널 멀티플렉서 서버가 계속 살아 있으면 새 패널도 옛 PATH를 받을 수 있다. 그때는 그 셸에서 PATH를 다시 읽는다.

   ```powershell
   $env:Path = [Environment]::GetEnvironmentVariable('Path','Machine') + ';' + [Environment]::GetEnvironmentVariable('Path','User')
   ```

3. 확인

   ```powershell
   cargo --version      # cargo 1.99.1 (...)
   rustc --version      # rustc 1.99.1 (...)
   cargo new hello_rust
   cd hello_rust
   cargo run            # Hello, world!
   ```

4. Zed에서 `Cargo.toml`이 있는 폴더를 열면 Rust 지원이 켜진다. rust-analyzer는 PATH에 있는 것(4절에서 설치한 것)을 쓴다.

---

이전: [README.md](README.md) · 다음: [02-GPUI.md](02-GPUI.md) · 입구: [README.md](README.md)
