# wiki-docs

여러 관심 분야의 문서를 하나의 위키로 모으고, LLM 에이전트(claude·codex·agy)가 정리·연결·갱신하는 개인 지식 저장소다.

- 위키 페이지: `wiki/` (분야별 → 소스별: `wiki/<분야>/{web,youtube}/`)
- 정리 전 원본: `inbox/`, 정리가 끝난 원본: `raw/{web,youtube}/`
- 에이전트 작업 지침: [WIKI.md](WIKI.md) · 사용 시나리오: [USECASES.md](USECASES.md) · 의도: [INTENT.md](INTENT.md)

## 새 컴퓨터에서 처음 할 일

이 저장소는 데스크탑·노트북 모두 **`D:\projects\wiki-docs`** 에 둔다. 경로가 같아야 `wiki` 스킬이 같은 내용으로 동작한다.

1. 저장소를 받는다.

   ```powershell
   git clone https://github.com/nampluskr/wiki-docs.git D:\projects\wiki-docs
   ```

2. `wiki` 스킬을 이 컴퓨터에 한 번 등록한다. 에이전트(Claude Code 등)를 띄우고 아래처럼 말하면 된다. 절차의 원본은 [WIKI.md 8절 "새 컴퓨터 설정"](WIKI.md#8-다른-워크스페이스에서-쓰기-wiki-스킬)이다.

   ```text
   D:\projects\wiki-docs 를 git pull 하고, WIKI.md 8절 "새 컴퓨터 설정"대로 이 컴퓨터에 wiki 스킬을 등록해줘.
   스크립트는 PowerShell 7로 실행하고, ~/.claude/settings.json 의 additionalDirectories 에 D:\projects\wiki-docs 를 넣어줘.
   끝나면 링크·skills.json·config.toml·AGENTS.md 결과를 보여줘.
   ```

   등록하는 것: `~/.claude/skills/wiki`·`~/.agents/skills/wiki` 링크(→ `skills/wiki`), agy `~/.gemini/config/skills.json`, Claude·Codex의 wiki-docs 쓰기 허용, `D:\projects\AGENTS.md`의 위키 안내 한 줄.

3. 에이전트를 새로 띄워 확인한다. Claude·agy는 `/wiki`, Codex는 `$wiki`가 자동완성되면 된다. 예) "위키에서 herdr 재시작 방법 찾아줘" → 위키 내용으로 답한다.

이후에는 `git pull`만 하면 스킬과 지침이 함께 갱신된다.

## 쓰는 법 (어느 폴더에서든)

| 하고 싶은 것 | 말하는 예 |
| --- | --- |
| 위키에서 찾기 | "위키에서 ego lite 설정 찾아줘" — 위키 내용만으로 먼저 답한다. 더 필요하면 "더 자세히"라고 하면 외부 검색 |
| 위키에 추가 | "조사해서 위키에 넣어줘", 답을 본 뒤 "위키에 추가해줘" |
| 원본 정리 | 원본 폴더를 `inbox/YYMMDD-{web\|youtube}-{title}/`에 넣고 "inbox 정리해줘" |
| 유튜브 | "이 유튜브를 위키에 넣어줘 <URL>" |

push는 매번 확인을 받는다.
