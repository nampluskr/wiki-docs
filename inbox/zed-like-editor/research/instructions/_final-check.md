# 최종 확인 지시 (문서는 읽기만, 고치지 않는다)

대상: D:\projects\docs\guides\zed-like-editor\ 의 README.md, 02-gpui\README.md, 03-tauri\README.md, 04-COMPARISON.md
기준: _decisions.md (사용자 결정 전부), _common.md, 01-zed\01~04 (명세)
명령에 삭제·쓰기(Remove-Item, Set-Content 등)를 넣지 않는다. 결과는 지정 파일 하나에만 쓴다.

## checker (agy Flash) → final-check-checker.md
1. 형식: frontmatter, "> 이 파일을 읽는 때:", 머리 표, 이전/다음/입구 링크가 실제 파일을 가리키는지
2. 짝 규칙: 02-gpui 와 03-tauri 의 절 번호·제목이 같은지, 5절 체크리스트 열(ID/질문/왜/관련 ID/확인 방법/우선순위)이 같은지
3. 결정 반영: _decisions.md 의 포함/제외(예: Settings UI 없음, 프로젝트 검색 제외, 확장=Zed 방식·로컬 설치만·확장 화면 있음·끄기 없음, Catppuccin 내장, PDF·동영상은 향후 본체 기능, Tauri 편집기=CM6, 포맷 제외)와 어긋나는 문장
4. README 의 읽기 지도·파일 표가 실제 파일 구성과 맞는지
출력: | 번호 | 심각도 | 파일·위치 | 문제 | 근거 | 고칠 방향 |

## scouter-opus → final-review-opus.md
02-gpui\README.md, 03-tauri\README.md, 04-COMPARISON.md 의 **사실** 리뷰:
1. 크레이트·라이브러리 버전과 라이선스 (crates.io·npm·공식 저장소로 표본 10개 이상 대조)
2. 기술적으로 틀린 주장 (예: gpui/gpui-kit/Tauri/WebView2/CM6/web-tree-sitter/wasmtime 능력)
3. 출발 문서로서 빠진 중요한 조사·검토 항목 (02 와 03 각각)
4. 04 의 비교·권고가 02·03 내용과 맞는지
출력: | 번호 | 심각도 | 파일·위치 | 문제 | 근거(URL) | 고칠 방향 | + 대조한 표본 목록
