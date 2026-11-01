# followup-1 — PotPlayer 주요 기능 이름 목록 (scouter sonnet, 확인일 2026-11-01)

## 요약
- **공식 출처로 확인된 기능은 적다.** 공식 페이지(potplayer.tv, Kakao Global PotPlayer 안내)는 마케팅 수준의 기능 목록만 제공한다. 영상 필터, 색 조정, 화면 비율·자르기·회전, 오디오 정규화·이퀄라이저는 공식 페이지에서 확인하지 못했다.
- 공식 문서·매뉴얼 페이지는 찾지 못했다 (미확인). `potplayer.daum.net`은 이 환경에서 접속되지 않았다 (DNS 오류).
- 대부분의 항목은 2차 출처(릴리스 노트 모음, 포럼, 단축키 목록)다. 신뢰도는 낮음~중간이다.
- 사용하지 않은 출처: GitHub `Potplayer-Software` 조직. 공식 도메인과 무관해 보이는 다운로드 링크가 있어 신뢰하지 않는다. 아래 표에도 넣지 않았다.
- neowin, techspot 페이지는 이 환경에서 HTTP 403이라 본문을 읽지 못했다. 검색 결과 요약으로만 인용했다.

## 출처 약어
- [OFF] https://potplayer.tv/ — 공식 사이트 안내 페이지 (WebFetch 요약으로 확인)
- [WIKI] https://en.wikipedia.org/wiki/PotPlayer — 기능 상세 없음 (일반 설명만)
- [REL] 릴리스 노트 모음. 검색 결과 요약으로만 확인:
  - https://www.neowin.net/amp/potplayer-260401/ (PotPlayer 260401)
  - https://www.techspot.com/downloads/5543-daum-potplayer.html
- [FORUM] https://forum.videohelp.com/ 의 PotPlayer 스레드. 구체 스레드 URL은 검색 결과에 있으나 내용은 직접 읽지 않음
- [OCAM] https://new.ohsoft.net/eng/ocam_help.php?id=114 — 정규화(Normalizer) 설명 (검색 요약)
- [KEYS] https://www.scribd.com/doc/152020425/Potplayer-Keyboard-Shortcut — 단축키 목록. 기능명 존재 확인용으로만 사용 (검색 요약)

## 재생
| 기능 | 상태 | 출처 |
|---|---|---|
| 북마크 (즐겨찾는 장면/챕터) | 확인 | [OFF] "Bookmarks for favorite scenes or chapters". 개수 한도가 2000으로 늘었다는 릴리스 노트도 있음 [REL] (1.5.x 시절 노트) |
| DVD/TV/HDTV 재생 | 확인 | [OFF] |
| OpenCodec(코덱 추가) 지원 | 확인 | [OFF] |
| 탐색(프레임 정확 탐색 등) | 미확인 | 공식·2차 출처 모두 직접 확인 못함 |
| 재생 속도 | 미확인 | 위와 같음 |
| 구간 반복 (A-B) | 미확인 | 제외한 출처에서만 언급됨 |
| 재생목록 | 미확인 | 위와 같음 |

## 영상
| 기능 | 상태 | 출처 |
|---|---|---|
| 하드웨어 가속: DXVA, CUDA, QuickSync | 확인 | [OFF] "Hardware acceleration via DXVA, CUDA, and QuickSync" |
| 내장 DXVA 디코더 (H.264/AVC, VC-1, MPEG2) | 확인 (오래된 정보) | [REL] 구버전 목록 설명. EVR은 Windows 7 한정이라고 되어 있어 현재와 다를 수 있음 |
| 3D 출력 (Side by Side, Top and Bottom, Page Flipping) | 확인 | [OFF] |
| Direct3D9 Ex Flip Mode, Overlay | 확인 | [OFF] |
| 디인터레이스 | 부분 확인 | [REL] "Changed default value of Deinterlace to Motion Adaptive" (1.6.x 노트), "Improved deinterlacing options" (1.5.x 노트). 공식 페이지에서는 미확인 |
| 선명 (샤프닝, adaptive sharpen) | 부분 확인 | [FORUM] 사용자 후기로 존재 언급 ("adaptive sharpen"). 공식 확인은 미확인 |
| 노이즈 제거 (3D denoise 토글) | 부분 확인 | [KEYS] 단축키 목록에 "3D denoise" 항목. 공식 확인은 미확인 |
| 흐림 (블러) | 미확인 | 출처 없음 |
| 밝기·대비·채도·색조 조절 | 부분 확인 | [KEYS] 밝기/대비/채도/색 단축키가 1% 단위라고 검색 요약에 있음. 공식 확인은 미확인 |
| 화면 비율 | 미확인 | 출처 없음 |
| 자르기 | 미확인 | 출처 없음 |
| 회전 | 미확인 | 출처 없음 |
| 픽셀 셰이더 (사용자 정의 HLSL) | 부분 확인 | [FORUM] 셰이더 팩이 PotPlayer와 함께 쓰인다는 언급. 공식 확인은 미확인 |

## 자막
| 기능 | 상태 | 출처 |
|---|---|---|
| SRT, SMI 텍스트 자막 | 확인 | [OFF] "Text subtitles in SMI and SRT formats" |
| ASS/SSA 애니메이션 자막, SMI Ruby 태그 | 확인 | [OFF] |
| DVD(VobSub), Blu-ray 자막 | 확인 | [OFF] |
| 자막 싱크 조절 | 부분 확인 | 검색 요약에 "0.5초 단위" 조절이라는 설명이 있으나 원문 URL을 특정하지 못함. [KEYS]에도 싱크 항목이 있음. 공식 확인은 미확인 |

## 오디오
| 기능 | 상태 | 출처 |
|---|---|---|
| 사운드 카드(출력 장치) 선택 | 확인 | [OFF] |
| 정규화 (Normalizer) | 부분 확인 | [OCAM] "Use Normalizer During Playback" 옵션 설명. 이 페이지는 PotPlayer가 아닌 다른 프로그램(oCam) 도움말일 가능성이 있어 신뢰도 낮음 |
| 이퀄라이저 | 부분 확인 | 소프트웨어 다운로드 사이트 설명에 언급 (검색 요약). 공식 확인은 미확인 |
| 오디오 싱크 조절 | 부분 확인 | [KEYS] 0.05초 단위 조절 항목 (검색 요약) |
| 오디오 트랙 선택 | 미확인 | 출처 없음 |

## 화면
| 기능 | 상태 | 출처 |
|---|---|---|
| 장면 스냅샷 미리보기 | 확인 | [OFF] "Snapshot previews of scenes" |
| 캡처(스크린샷) | 미확인 | 위 스냅샷 미리보기 외에는 출처 없음 |
| 전체 화면 | 미확인 | 출처 없음 |
| 항상 위 | 미확인 | 출처 없음 |

## 결론
- 공식 출처로 근거가 있는 기능: 북마크, DXVA/CUDA/QuickSync, 3D 출력, SRT/SMI/ASS/SSA/VobSub 자막, 사운드 카드 선택, 스냅샷 미리보기.
- R2에 해당하는 기능(필터, 색 조정, 디인터레이스)은 공식 근거가 없다. 위 표의 "부분 확인" 항목은 모두 2차 출처다.
- 공식 문서에 가까운 자료를 얻으려면 PotPlayer 앱의 설정 화면을 직접 보거나, 공식 포럼·카페(Daum/Kakao)를 확인해야 한다. 이 조사에서는 접근하지 못했다.
