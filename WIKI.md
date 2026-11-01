# WIKI — 위키 작업 지침

> 에이전트(claude·codex·agy)가 이 저장소에서 위키 작업을 할 때 따르는 지침이다.
> 근거는 `specs/v0.2/DECISIONS.md`(v0.2 D-n)와 그것이 이어받은 `specs/v0.1/DECISIONS.md`(v0.1 D-n)에, 시나리오 예시는 `USECASES.md`(UC-n)에 있다.
> 이 지침과 AGENTS.md가 다르면 AGENTS.md를 따르고 사람에게 알린다.
> 다른 워크스페이스에서 `wiki` 스킬로 불렸을 때도 이 지침을 따른다. 경로는 이 저장소(`D:\projects\wiki-docs`) 기준이다(8절).

## 1. 폴더 구조 (v0.2 D-2, D-5, D-7, D-10, D-12)

```text
wiki/                              위키. 하나뿐이다.
  index.md                         분야 목록
  <분야>/                           분야 폴더. 그 분야의 첫 페이지를 저장할 때 만든다.
    index.md                       그 분야의 페이지 목록 (소스별 제목)
    youtube/<페이지>.md             유튜브 영상에서 만든 페이지
    web/<페이지>.md                 웹에서 가져온 정보로 만든 페이지(에이전트가 정리한 문서, 질문에 답하며 한 웹 검색), 또는 두 소스가 섞인 페이지
inbox/
  <YYMMDD-소스-title>/             정리 전 원본
raw/                               분야로 나누지 않는다
  youtube/<YYMMDD-title>/          정리가 끝난 유튜브 원본
  web/<YYMMDD-title>/              정리가 끝난 웹 정보 문서 원본
                                   파일 내용·이름은 고치지 않는다.
```

- 분야 폴더와 소스 폴더는 미리 만들지 않는다. 그 분야·소스의 첫 페이지를 저장할 때 만든다. 빈 폴더를 두지 않는다. 새 분야는 이름을 사람에게 확인받는다. (v0.1 D-1, v0.2 D-2)
- 지금 분야는 `dev`(개발 툴, AI, 소프트웨어, 하드웨어) 하나다. 앱은 독립 레포에서 관리하므로 `apps` 분야는 두지 않는다. (v0.2 D-7)
- 분야 폴더 바로 아래에는 `index.md`와 소스 폴더만 둔다. `raw/` 바로 아래에는 소스 폴더(`youtube`·`web`)만 둔다. 원본의 분야는 그 원본을 가리키는 위키 페이지의 위치로 알 수 있다. (v0.2 D-12)
- 여러 분야에 걸치는 내용은 한 분야에 두고 다른 분야 페이지에서 링크로 잇는다. (v0.1 D-1)

## 2. 이름 규칙

### 위키 페이지 (v0.1 D-2, v0.2 D-2)

- `wiki/` 아래 폴더·파일 이름은 영문 소문자·숫자·하이픈(`[a-z0-9-]`)과 확장자만 쓴다. 페이지 파일 이름에는 날짜를 붙이지 않는다. 예) `wiki/dev/web/herdr-multi-agent-roles.md`
- 한글 제목은 frontmatter `title`과 본문 제목에 쓴다.

### 원본 폴더 (v0.2 D-4, D-10, D-12)

- inbox: `inbox/YYMMDD-{소스}-{title-in-english}/`. 예) `inbox/261101-web-rust-gui/`
- raw: `raw/{소스}/YYMMDD-{title-in-english}/`. 소스는 상위 폴더에 있으므로 이름에서 뺀다. 예) `raw/web/261101-rust-gui/`, `raw/youtube/261101-claude-skill-writing-rules/`
- inbox에서 raw로 옮길 때 이름에서 `-{소스}`만 빼고 날짜와 title은 그대로 둔다.
- `{title-in-english}`: 영문 소문자·숫자·하이픈(kebab-case).
- `YYMMDD`: 원본을 inbox에 넣은 날. raw로 옮겨도 바꾸지 않는다.
- `{소스}`: `youtube` 또는 `web`. 소스는 이 둘뿐이다.
- `{title}`: 넣는 사람이 짓는다. 규칙과 다른 이름으로 들어오면 원래 폴더 이름이나 제목으로 새 이름을 제안하고 확인받아 `git mv`로 바꾼다.
- 유튜브의 `{title}`은 영상 제목을 영문으로 요약한 것이다. 영상 ID는 폴더 안 파일 이름에 남는다.
- 원본이 파일 하나여도 폴더로 둔다.
- 폴더 안의 파일 이름(한국어·영어)과 내용은 바꾸지 않는다. 폴더 이름·위치만 이 규칙에 맞추려고 바꿀 수 있다. (v0.2 D-5)

## 3. 페이지 양식

### frontmatter (v0.1 D-5, D-17)

`index.md`를 뺀 모든 페이지는 아래 다섯 항목으로 시작한다.

```yaml
---
title: Herdr 멀티 에이전트 — 역할과 구성
sources:
  - raw/web/261101-herdr/PLANNING.md
  - https://herdr.dev/llms-full.txt (2026-11-01)
updated: 2026-11-01
tags: [herdr, multi-agent]
status: draft
---
```

- `sources`: 문자열 목록. `raw/` 경로는 그대로, URL은 뒤에 ` (YYYY-MM-DD)` 확인 날짜를 붙인다. 유튜브 원본은 `raw/` 폴더 경로를 적는다. 직접 겪은 일을 적을 때는 그 일을 기록한 저장소 안 문서(예: `specs/v0.1/PROGRESS.md`)의 경로를 적는다. 내용이 있는 페이지의 `sources`는 비우지 않는다.
- `updated`: 마지막으로 내용을 고친 날짜.
- `status`: 새 페이지는 `draft`. `stable`로는 사람이 지시할 때만 바꾼다. (UC-8)

### 페이지를 둘 소스 폴더 (v0.2 D-3, D-10)

- `sources`가 한 소스에서만 왔으면 그 소스 폴더에 둔다: 유튜브 원본·YouTube URL → `youtube/`, 웹 정보 원본(`raw/web/`)·웹 URL → `web/`.
- 두 소스가 섞였으면 `web/`에 둔다. 저장소 안 기록 문서(직접 겪은 일)는 소스 종류로 세지 않는다.
- `youtube/` 페이지에 웹 내용을 보태 섞이게 되면, 같은 커밋에서 페이지를 `web/`으로 옮기고(`git mv`) 그 페이지를 가리키는 링크와 분야 index를 함께 고친다.

### 유튜브 페이지 양식 (v0.2 D-11)

유튜브 페이지도 주제 단위다(영상 하나에 페이지 하나가 아니다). 유튜브는 최신 정보가 중요하므로 페이지 안을 **영상별 섹션**으로 나누고 **최신 등록일이 위**로 오게 둔다.

```markdown
# <주제 제목>

<페이지 요약: 최신 영상 기준으로 지금 유효한 내용 몇 줄>

## 점검 목록 (최신 기준)          ← 있는 페이지만. 최신 내용으로 갱신

## YYYY-MM-DD · <영상 제목>        ← 최신 등록일이 맨 위
- 채널 · 길이 · URL
- 원본: raw 링크
<그 영상에서 정리한 내용. 하위 절은 ###, 타임스탬프 유지>
> YYYY-MM-DD 영상과 다름: …      ← 예전 섹션과 내용이 다를 때만

## YYYY-MM-DD · <예전 영상 제목>
...

## 관련 페이지
```

- 등록일은 그 영상 `<영상ID>.meta.json`의 `upload_date`(YYYYMMDD)를 `YYYY-MM-DD`로 쓴다. 원본 폴더 이름의 날짜(inbox에 넣은 날)와 다르다.
- 같은 주제의 새 영상은 요약·점검 목록 바로 아래에 섹션을 더하고, 요약·점검 목록을 최신 기준으로 고친다.
- 나중 영상이 예전 영상과 다르면 예전 섹션은 지우지 않고, 최신 섹션에 `> YYYY-MM-DD 영상과 다름: …`을 적는다. 다른 내용을 넣기 전에는 5절 4단계대로 묻는다.
- 웹 내용을 보태 `web/`으로 옮긴 페이지도 영상 섹션 양식은 그대로 둔다.

### 본문 (v0.1 D-6, D-8)

- 링크는 표준 마크다운 상대 링크만 쓴다. `[[…]]`는 쓰지 않는다. 링크 대상은 실제 파일이어야 한다.
  - 같은 소스 폴더: `[제목](other-page.md)` / 다른 소스 폴더: `[제목](../youtube/other-page.md)` / raw: `[제목](../../../raw/web/261101-herdr/README.md)`
- 수식은 인라인 `$…$`, 블록 `$$…$$`로 쓴다.

### 목차 (v0.1 D-7, v0.2 D-6)

- `wiki/index.md`: 분야마다 한 줄 `- [분야 이름](<분야>/index.md)`.
- `wiki/<분야>/index.md`: 페이지가 있는 소스만 `## youtube`·`## web` 제목을 두고, 그 아래에 페이지마다 한 줄 `- [제목](<소스>/<페이지>.md)`.
- 소스 폴더에는 index를 두지 않는다.
- 페이지를 추가·삭제·이름변경·이동하면 분야 `index.md`를, 분야를 추가하면 `wiki/index.md`도 같은 커밋에서 고친다. 폴더의 페이지 목록과 index의 링크 목록이 같아야 한다.

## 4. 질문에 답하기 (UC-1, UC-2 / v0.1 D-9, D-10, D-18)

1. `git pull`로 최신 위키를 받는다.
2. 위키를 먼저 찾는다: `wiki/index.md` → 분야 `index.md`에서 관련 페이지를 찾고, 못 찾으면 `wiki/` 본문을 키워드로 검색(grep)한다.
3. **먼저 위키 내용만으로 답한다.** 찾은 내용을 요약해 보여 주고 페이지 경로를 붙인다. 위키에 일부만 있어도 외부 검색으로 채우지 않고, 위키에 없는 부분은 "위키에 없음"이라고 적는다. 답 끝에 "더 자세한 내용이 필요하면 외부에서 찾아보겠다"고 안내한다. 위키에 전혀 없으면 "위키에 없음"을 알리고 외부 검색을 할지 묻는다.
4. **외부 검색은 사람이 추가로 요청할 때만 한다**(예: "더 자세히", "외부에서 찾아줘", 3단계 물음에 예). 에이전트 기본 웹 검색을 쓰고, 로그인이 필요한 페이지는 ego-lite로 연다. 위키에 있는 부분은 다시 검색하지 않는다.
5. 답은 출처를 나눈다: 위키에서 온 부분은 페이지 경로, 외부 검색 부분은 URL.
6. 외부 검색을 했으면 답 끝에 **"위키에 추가할까요?"**를 묻는다.
7. 사람이 예라고 하거나, 처음부터 "조사해서 위키에 넣어줘"처럼 저장을 지시했으면(UC-3) 5절로 간다. 그 밖에는 위키를 고치지 않는다.

## 5. 위키에 저장하기 (UC-4, UC-5 / v0.1 D-1, D-11, v0.2 D-3)

1. 같은 주제의 페이지가 있는지 분야 `index.md`와 본문 검색으로 본다.
2. **없으면** 새 페이지를 만든다.
   - 분야 폴더가 없으면 분야 이름을 사람에게 확인받고 `wiki/<분야>/`와 `index.md`를 만든 뒤 `wiki/index.md`에 넣는다.
   - 3절 "페이지를 둘 소스 폴더"대로 소스 폴더를 고른다. 없으면 만든다.
   - 3절 양식대로 쓰고 분야 `index.md`의 그 소스 제목 아래에 한 줄을 넣는다.
3. **있으면** 없던 내용만 보태고 `sources`·`updated`를 고친다. 같은 주제의 페이지를 둘 만들지 않는다. 보태서 소스가 섞이면 페이지를 `web/`으로 옮긴다. 유튜브 영상이면 3절 "유튜브 페이지 양식"대로 영상 섹션을 맨 위(요약·점검 목록 다음)에 더하고 요약·점검 목록을 최신 기준으로 고친다.
4. 새 내용이 기존 내용과 **다르면** 고치기 전에 기존 내용·새 내용과 각각의 출처를 나란히 보여 주고 묻는다: 기존 유지 / 새 내용으로 바꾸기 / 둘 다 적기. 답 없이 기존 내용을 바꾸지 않는다.
5. 7절대로 커밋한다.

## 6. inbox 원본 정리 (UC-6, UC-7 / v0.1 D-3, D-4, D-13, D-15, D-16, v0.2 D-4, D-5, D-10, D-12)

사람이 "inbox 정리해줘"처럼 **요청할 때만** 한다. 스스로 inbox를 처리하지 않는다.

### 원본이 들어오는 길

- 사람이 데스크탑·노트북에서 넣거나 GitHub 웹(`inbox/` → Add file → Upload files)에서 올린다. 폴더 이름은 2절 원본 폴더 규칙을 따른다.
- 사람이 요청하면 에이전트가 넣는다(유튜브 등).
- 원본 종류는 두 가지다: 웹에서 가져온 정보를 에이전트가 정리한 마크다운 문서 폴더(`web`), 유튜브 영상의 자막·요약(`youtube`).
- 질문에 답하며 한 웹 검색 결과는 inbox·raw에 넣지 않는다. 출처 URL만 페이지 `sources`에 남긴다(v0.1 D-4).

### 유튜브 원본 넣기 (UC-7)

1. 무엇을 받을지 사람에게 묻는다: 자막 전체 / YouTube 요약 / 둘 다.
2. 영상 제목을 영문으로 요약해 폴더 이름 `inbox/YYMMDD-youtube-<title>/`을 정하고(날짜는 오늘), 메타 정보를 받는다. 메타는 항상 둔다.

   ```bash
   PY='C:\winpython\WPy64-31180_cpu\python-3.11.8.amd64\python.exe'
   D='inbox/YYMMDD-youtube-<title>'
   "$PY" -m youtube_kit.cli info '<URL>' -o "$D"
   ```

3. 자막을 골랐으면 트랙을 이 순서로 고른다: **1순위 원본 언어, 2순위 한국어 번역.** 같은 언어면 수동(`manual:`)이 자동(`auto:`)보다 먼저다. 예) 한국어 영상 → `auto:ko-orig`. 두 순위의 트랙이 모두 없으면 트랙 목록을 사람에게 보여 주고 묻는다.

   ```bash
   "$PY" -m youtube_kit.cli subtitle '<URL>' --track <트랙ID> --format srt -o "$D"
   ```

4. 요약을 골랐으면 ego-lite에서 YouTube 요약 기능으로 가져와 `summary.md`에 저장한다. 맨 위에 영상 URL과 가져온 날짜를 적는다.
5. 받지 못한 것이 있으면 다른 것으로 대신하지 않고 사람에게 알린다. 영상·음성 파일은 받지 않는다.

```text
inbox/YYMMDD-youtube-<title>/
  <영상ID>.meta.json             항상
  <영상ID>.<종류>.<키>.srt        자막을 고른 경우 (youtube-kit 파일 이름 그대로)
  summary.md                     요약을 고른 경우
```

- youtube-kit은 cpu 환경(`C:\winpython\WPy64-31180_cpu`)에만 설치되어 있다. gpu 환경에는 yt-dlp가 없다.
- youtube-kit이 저장한 파일은 이름·내용을 바꾸지 않는다.

### 정리 절차

1. `git pull` 후 `inbox/`를 본다.
2. 폴더 이름이 2절 원본 폴더 규칙과 다르면 새 이름을 제안하고 확인받아 `git mv`로 바꾼다.
3. 원본마다 분야를 정한다. 정할 수 없으면 그 원본은 그대로 두고 사람에게 묻는다. 새 분야면 이름을 확인받는다.
4. 원본 내용을 위키 페이지로 정리한다(5절). 소스 폴더는 원본의 소스(`web`·`youtube`)를 따른다. `sources`에는 옮길 `raw/` 경로를 적는다. 유튜브는 `meta.json`의 `upload_date`로 영상 섹션 제목의 등록일을 쓴다(3절).
5. 원본 폴더를 `raw/<소스>/YYMMDD-<title>/`로 옮긴다(`git mv`, 이름에서 `-<소스>`만 뺀다). 소스 폴더가 없으면 만든다. 폴더 안의 구조·파일 이름·내용은 그대로 두고, 옮긴 뒤에도 고치지 않는다.
6. 7절대로 커밋한다.

## 7. 커밋과 동기화 (UC-9 / v0.1 D-14)

- 작업 전에 `git pull`을 한다.
- 작업이 끝나면 커밋한다. 커밋 메시지 예) `wiki: dev/web/rust-gui-frameworks 추가`, `inbox: 261101-web-rust-gui 정리 → raw/web/261101-rust-gui`
- push는 매번 **"push할까요?"**를 묻고, 사람이 예라고 하면 한다.
- 작업을 마치면 `specs/<현재 버전>/PROGRESS.md`에 기록한다(AGENTS).

## 8. 다른 워크스페이스에서 쓰기 (wiki 스킬)

`D:\projects`와 그 아래 어느 폴더에서 연 세션이든 "위키에서 찾아줘", "위키에 추가해줘", "inbox 정리해줘"처럼 말하면 `wiki` 스킬이 이 지침으로 안내한다.

- 스킬 원본: 이 저장소의 `skills/wiki/SKILL.md`. 규칙은 이 WIKI.md에만 두고, 스킬은 어느 절을 읽을지만 안내한다.
- 스킬과 지침은 git으로 동기화된다. 데스크탑·노트북 모두 저장소 경로가 `D:\projects\wiki-docs`이므로 같은 내용으로 동작한다.
- 다른 폴더에서 불려도 경로는 `D:/projects/wiki-docs/` 기준 절대경로로, git은 `git -C D:/projects/wiki-docs`로 한다. 시작 전에 `status --short`가 비어 있지 않으면(다른 세션이 작업 중일 수 있음) 손대지 않고 묻는다.
- 호출: Claude·agy는 `/wiki`, Codex는 `$wiki`. 위키를 말하면 스킬 설명이 맞아 자동으로 쓰이기도 한다.

### 새 컴퓨터 설정 (컴퓨터마다 한 번)

스킬을 찾는 폴더, 권한 설정, `D:\projects\AGENTS.md`는 git 밖에 있어서 컴퓨터마다 한 번 등록한다. 저장소를 `D:\projects\wiki-docs`에 받은(`git clone` 또는 `git pull`) 뒤 PowerShell 7에서 실행한다. 이미 된 것은 건너뛴다. 저장소 첫 화면의 `README.md`에도 이 절을 가리키는 안내가 있다.

에이전트에게 맡길 때의 프롬프트 예:

```text
D:\projects\wiki-docs 를 git pull 하고, WIKI.md 8절 "새 컴퓨터 설정"대로 이 컴퓨터에 wiki 스킬을 등록해줘.
스크립트는 PowerShell 7로 실행하고, ~/.claude/settings.json 의 additionalDirectories 에 D:\projects\wiki-docs 를 넣어줘.
끝나면 링크·skills.json·config.toml·AGENTS.md 결과를 보여줘.
```

직접 실행할 때의 명령:

```powershell
$repo  = 'D:\projects\wiki-docs'
$skill = "$repo\skills\wiki"

# 1) Claude·Codex 스킬 폴더에 링크 (관리자 권한 필요 없음)
foreach ($dir in "$HOME\.claude\skills", "$HOME\.agents\skills") {
  New-Item -ItemType Directory -Force $dir | Out-Null
  if (-not (Test-Path "$dir\wiki")) { New-Item -ItemType Junction -Path "$dir\wiki" -Target $skill | Out-Null }
}

# 2) agy: 링크를 따라가지 않으므로 실제 경로를 skills.json에 등록 (있는 항목은 보존)
$sj = "$HOME\.gemini\config\skills.json"
New-Item -ItemType Directory -Force (Split-Path $sj) | Out-Null
$cfg = if (Test-Path $sj) { Get-Content $sj -Raw | ConvertFrom-Json } else { [pscustomobject]@{ entries = @() } }
$p = "$repo\skills" -replace '\','/'
if (-not ($cfg.entries | Where-Object path -eq $p)) { $cfg.entries = @($cfg.entries) + @([pscustomobject]@{ path = $p }) }
$cfg | ConvertTo-Json -Depth 5 | Set-Content -Encoding utf8NoBOM $sj

# 3) Codex: wiki-docs 쓰기 허용 (블록이 없을 때만 추가)
$ct = "$HOME\.codex\config.toml"
if (-not (Select-String -Path $ct -Pattern 'wiki-docs' -Quiet)) {
  Add-Content -Encoding utf8NoBOM $ct "`n[sandbox_workspace_write]`nwritable_roots = ['D:\projects\wiki-docs']"
}

# 4) D:\projects\AGENTS.md에 위키 안내 (없을 때만)
$ag = 'D:\projects\AGENTS.md'
if (-not (Select-String -Path $ag -Pattern 'wiki 스킬' -Quiet)) {
  Add-Content -Encoding utf8NoBOM $ag "`n## 위키`n`n- 위키 검색·요약·추가·수정·inbox 정리 요청은 ``wiki`` 스킬로 하고, 규칙은 ``D:\projects\wiki-docs\WIKI.md``를 따른다. 질문엔 위키 내용만으로 먼저 답한다."
}
```

5) Claude 쓰기 허용: `~/.claude/settings.json`의 `permissions.additionalDirectories` 목록에 `"D:\projects\wiki-docs"`를 한 줄 넣는다(다른 설정이 많아 직접 고친다).

확인: 각 에이전트를 새로 띄워 Claude·agy는 `/wiki`, Codex는 `$wiki`가 자동완성되는지 본다. `[sandbox_workspace_write]` 블록이 config.toml에 이미 있으면 새로 넣지 말고 그 블록의 `writable_roots`에 경로를 더한다.

이후에는 `git pull`만 하면 스킬과 지침이 갱신된다.
