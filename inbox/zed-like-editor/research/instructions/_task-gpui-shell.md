# 조사: gpui-shell 기능 검토 (ID 접두사 S-) — 담당 scouter-agy2

저장 파일: C:\Users\nampl\AppData\Local\Temp\claude\D--projects\b2c61f73-3611-43da-b80f-0f7d315ef143\scratchpad\zed-research\gpui-shell.md
규칙: 공식 저장소·crates.io·docs.rs 우선, 주장마다 출처 URL, 확인 못 한 것은 "미확인", 판단은 "판단"으로 표시. 파일은 저장 파일 하나만 쓴다.

## 배경
- 우리 앱: Zed 와 같은 레이아웃·사용감의 편집기를 gpui-kit 0.8.1(1순위)로 새로 만든다 (README: D:\projects\docs\guides\zed-like-editor\README.md).
- 사용자 질문: Zed 처럼 확장(extension) 기능을 넣을 수 있나? 예: PDF 보기, 탭에서 동영상 재생.
- 알아낸 것(leader): gpui-shell 0.8.1 = "A scriptable application runtime for GPUI, built on gpui-base" (https://crates.io/crates/gpui-shell , 저장소 https://github.com/longbridge/gpui-kit). 관련: longbridge/rquickjs(quickjs-jit), native-shell(모바일 판), 예제 longbridge/longbridge-lite, huacnlee/gpui-shell-benchs.
- 이름이 같은 andre-brandao/gpui-shell(Wayland 데스크톱 셸)은 다른 프로젝트다. 섞지 마라.

## 조사할 것
1. 정체와 구조: 무엇을 하는 런타임인가. 스크립트 엔진(QuickJS?), 스크립트 언어(JS/TS), Rust 쪽과의 연결 방식(바인딩·브리지), UI 를 어떻게 그리나(gpui-kit 컴포넌트를 JS 에서 선언? 가상 DOM?), 상태·이벤트 모델
2. 제공 API: JS 에서 쓸 수 있는 컴포넌트·레이아웃·테마·키맵·파일 시스템·네트워크·창 API 목록 (문서·소스 기준)
3. 버전·의존성: gpui-shell 0.8.1 의 의존성(gpui-kit/gpui-base/gpui-pre 버전 고정 여부), Windows 지원, 빌드 준비물, 라이선스
4. 성숙도: 릴리스 이력, 예제·실사용 앱, 문서 양
5. **우리 앱의 확장 시스템으로 쓸 수 있나** (핵심):
   - 앱 본체(Rust, gpui-kit) 안에 gpui-shell 을 넣어 "확장 = JS 스크립트"로 패널·탭 뷰어·명령·메뉴를 추가할 수 있나
   - 확장 불러오기(런타임 로드·핫 리로드), 격리(샌드박스·권한), 확장 간 충돌, 성능
   - PDF 보기·동영상 재생 같은 미디어 뷰어를 JS 확장으로 만들 수 있나 (gpui-shell 이 이미지·캔버스·비디오·웹뷰를 제공하나, 아니면 Rust 쪽 네이티브 컴포넌트를 추가로 노출해야 하나)
   - Zed 확장(WASM: 언어·테마·아이콘·MCP 등, UI 확장 불가)과 비교
6. 다른 쓰임: 앱 전체를 gpui-shell(JS) 로 짜는 방식이 가능한가, 장단점(판단)
7. 위험 요소와 확신 낮은 부분

출력: 표 위주. 끝나면 파일 경로만 답해.
