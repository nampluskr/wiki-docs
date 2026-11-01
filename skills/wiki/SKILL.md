---
name: wiki
description: 개인 위키(D:\projects\wiki-docs)를 다룬다. 위키에서 찾기·요약하기, 위키에 추가·수정·저장하기, inbox 원본 정리, 유튜브 원본을 위키에 넣기를 요청할 때 사용한다. "위키에서 찾아줘", "위키에 있어?", "위키에 추가해줘", "위키 정리해줘", "inbox 정리해줘"처럼 위키를 말할 때 쓴다. 어느 폴더에서 연 세션이든 쓸 수 있다.
---

# wiki

대상: claude · codex · agy 공통. 따로 설치할 것은 없다.

위키 저장소는 **`D:/projects/wiki-docs`** 하나다. 규칙은 그 저장소의 `WIKI.md`에만 있다. 이 파일은 그곳으로 안내만 한다.

## 순서

1. 지금 작업 폴더가 어디든, 위키 파일 경로는 `D:/projects/wiki-docs/` 기준 절대경로로 쓰고 git은 `git -C D:/projects/wiki-docs …`로 한다.
2. 시작 전에 `git -C D:/projects/wiki-docs status --short`가 비어 있는지 본다. 커밋하지 않은 변경이 있으면 다른 세션이 작업 중일 수 있으니 손대지 말고 사람에게 묻는다. 비어 있으면 `git -C D:/projects/wiki-docs pull`.
3. `D:/projects/wiki-docs/WIKI.md`에서 할 일에 맞는 절만 읽고 그대로 따른다.
   - 질문에 답하기 → 4절
   - 위키에 저장하기 → 3절(페이지 양식)·5절
   - inbox 원본 정리, 유튜브 원본 넣기 → 2절(이름 규칙)·3절·6절
   - 커밋과 push → 7절
   - 처음이면 1절(폴더 구조)도 읽는다.
4. 작업을 마치면 `D:/projects/wiki-docs/AGENTS.md`의 PROGRESS 기록 규칙을 따른다.

## 꼭 지킬 두 가지

- **질문엔 위키 내용만으로 먼저 답한다.** 외부 검색은 사람이 추가로 요청할 때만 한다.
- **위키는 사람이 확인하거나 지시한 것만 저장한다.** push는 매번 묻는다.
