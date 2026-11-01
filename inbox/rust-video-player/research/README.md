---
tags: [rust, video-player, research, archive]
created: 2026-11-01
updated: 2026-11-01
---

# research · 조사 원자료 보관

> 이 파일을 읽는 때: 본문의 값·동작이 어디서 왔는지 원자료를 확인할 때, 보고서끼리 어긋난 값의 판정을 볼 때.

입구는 [../README.md](../README.md)다.

이 폴더는 문서 묶음을 만들며 Herdr 멀티에이전트(scouter·leader·writer)가 남긴 **작업 원자료**다. 고치지 않고 그대로 보관한다.
- **기준은 본문(01~07)이다.** 원자료와 본문이 다르면 본문이 맞다. 보고서끼리 어긋난 값은 `reviews/comparison.md` 의 판정을 따랐다. 예를 들어 argon 보고서의 `libmpv2` 6.0.0 은 7.0.0 이 맞다.
- 모든 조사는 확인일 2026-11-01 기준이다. 조사만 했고 패키지 설치·빌드·실행은 하지 않았다.
- 보고서의 출처 URL 은 조사자가 연 것이다. 본문 작성자는 URL 을 다시 열지 않았다. 확인 수준은 [reviews/writer-notes.md](reviews/writer-notes.md)에 있다.
- GPL/LGPL 코드 원문과 Zed 앱 코드는 없다. 사실(값·이름·동작)만 있다.

## 1. sources · 조사 원자료

| 파일 | 담당 | 내용 |
| --- | --- | --- |
| [sonnet/report-1.md](sources/sonnet/report-1.md) | rvp-scouter-sonnet | 요약, Q1~Q8: 권장안, 파이프라인, 백엔드 비교, gpui·Tauri 통합, R1, R2, PotPlayer 대응 |
| [sonnet/report-2.md](sources/sonnet/report-2.md) | rvp-scouter-sonnet | Q9~Q17: 소리, 자막, 탭 동작, 빌드·라이선스, 크레이트, 사례, 위험, 목차, 확신 낮은 부분 |
| [sol/report.md](sources/sol/report.md) | rvp-scouter-sol | Q1~Q17 전체. 출처 URL 이 가장 촘촘하다 (mpv D3D11 render API, WebView2 코덱, TextureStream) |
| [argon/report.md](sources/argon/report.md) | rvp-scouter-argon | Q1~Q17 전체. 수식·표 위주. 일부 값(libmpv2 버전, Tauri Range, d3d11vpp)이 판정과 다르다 |

## 2. reviews · 검토·판정·작성 기록

| 파일 | 담당 | 내용 |
| --- | --- | --- |
| [outline.md](reviews/outline.md) | rvp-leader | 확정 목차 (파일 구성과 각 파일 내용) |
| [comparison.md](reviews/comparison.md) | rvp-leader | 세 보고서 비교, 상충 11건의 판정, 한쪽만 언급한 항목의 처리, 사용자 결정 (GPL 허용, PotPlayer 대응표 기준) |
| [writer-notes.md](reviews/writer-notes.md) | rvp-writer | 본문을 쓰며 만난 모순과 보수적으로 처리한 방법 |
| [agy-approvals.log](reviews/agy-approvals.log) | rvp-leader | agy 승인 기록 (명령 원문) |

## 3. 어떤 값이 어디서 왔나

| 본문 내용 | 주 근거 |
| --- | --- |
| 엔진 권고, 세션·상태 기계, 탭 정책 | sol Q1·Q2·Q11, sonnet Q2·Q11 |
| gpui 현황·출력 3단계 | sonnet Q4 (로컬 gpui-pre 0.3.10 소스), sol Q4 |
| Tauri 현황·`<video>` 한계·HWND·대안 | sol Q5, sonnet Q5, comparison.md 판정 3 |
| R1 | sol Q6, sonnet Q6, argon Q6 |
| R2·GPL 필터 | sonnet Q7·Q12, sol Q7, comparison.md 판정 4·5 |
| 소리·자막·빌드·크기 | sonnet Q9·Q10·Q12, sol Q9·Q10·Q12, argon Q9·Q10·Q12 |
| 크레이트·사례 | sonnet Q13·Q14, sol Q13·Q14, comparison.md 판정 1·6~9·11 |
| 위험·검증 체크리스트 | sol Q15, sonnet Q15, argon Q15 |
