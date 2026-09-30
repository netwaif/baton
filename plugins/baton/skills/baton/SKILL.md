---
name: baton
description: 이 Claude 세션과 다른 앱의 에이전트(코덱스 앱 세션·헤르메스)를 대화로 연결해 일을 넘기고 답을 받는다. "코덱스 <세션ID>와 연결해", "코덱스에 (이미지 만들어 달라고) 시켜", "이거 코덱스로 넘겨", "헤르메스에게 이 파일(스킬) 보내", "메시지 받을 준비해", "연결 끊어", "/baton" 에 사용. 터미널 없이 데스크톱 앱 안에서만 쓴다. 한 번 연결하면 그 뒤로는 사용자가 세션 번호를 다시 말하지 않아도 이어진다.
---

# baton — 다른 앱과 대화 연결

## 준비(한 번)
`agentlayer version`이 1.12.0 이상인지 확인한다. 없거나 낮으면 앱 안 Bash로 `brew install netwaif/tap/agentlayer`(이미 있으면 `brew upgrade netwaif/tap/agentlayer`)를 실행하고 다시 확인한다. brew가 없으면 https://brew.sh 설치를 안내하고 멈춘다.

## 규칙 세 가지
- `agentlayer send`의 플래그(`--json`·`--file`·`--cwd`)는 **대상 앞에** 쓴다. 뒤에 쓰면 본문으로 들어간다.
- 답을 기다리는 `inbox wait`는 **백그라운드 Bash**로 켠다. 끝나면 그 출력이 이 세션에 들어온다. 처리한 뒤 연결이 살아 있으면 곧바로 다시 켠다.
- 짝 정보는 이 폴더의 `.baton/pair.json`에 둔다: `{"my_name","my_address","codex_session","cwd"}`. 사용자에게 세션 번호를 두 번 묻지 않는다.

## 연결하기 — "코덱스 <세션ID>와 연결해"
세션 ID(UUID)가 없으면 한 번만 안내한다: "코덱스 앱에서 '세션 번호 알려줘'라고 물어 나온 ID를 붙여 주세요."
1. 이름은 이 폴더명. `ADDR=$(agentlayer inbox open --name <폴더명>)` — 고유 주소(`al-xxxxxx`). `.baton/pair.json` 저장.
2. 백그라운드 Bash: `agentlayer inbox wait --name <폴더명> --timeout 2h`
3. 첫 메시지(플래그 먼저):
   ```
   agentlayer send --json --cwd <작업폴더> <코덱스 세션ID> "<사용자 요청 본문>

   이 대화의 회신 주소는 al-xxxxxx 이다. 결과(파일 경로 포함)는 셸에서
   agentlayer send al-xxxxxx \"<결과>\" 로 보내라. 이후 이 대화의 모든 회신도 같은 주소로."
   ```
4. "코덱스와 연결됐습니다. 답이 오면 알려 드립니다." 한 줄.

## 시키기 — "코덱스에 … 시켜"(연결된 뒤)
`.baton/pair.json`의 세션 ID로 `agentlayer send --json --cwd <cwd> <codex_session> "<요청> (회신 주소 al-xxxxxx)"`. 직전 wait가 끝나 있으면 다시 켠다. 연결 정보가 없으면 "연결하기"부터.

## 답 받기(백그라운드 wait가 끝났을 때)
출력의 `from:` 다음 본문이 답이다. 파일 경로가 있으면 열어 확인하고 사용자에게 보고한다. 처리 뒤 **곧바로** `inbox wait --name <폴더명> --timeout 2h`를 다시 켠다.
`답 없음(2h)`으로 끝났으면: "연결이 2시간 동안 조용해 대기를 멈췄습니다. 다시 기다릴까요?" 주소는 남아 있으니 다시 wait만 켜면 이어진다.

## 헤르메스 — "헤르메스에게 이 파일(스킬) 보내"
1. `agentlayer remote list`로 원격 이름을 확인한다(하나면 그것, 여럿이면 묻는다). 없으면 등록 절을 안내하고 멈춘다: `agentlayer remote add <이름> --kind hermes --local --profile <프로필> --workspace-root <절대경로>` (같은 맥의 헤르메스), VPS면 `--ssh <호스트>`.
2. 파일마다 `--file`(폴더는 zip 하나로 묶어서):
   ```
   agentlayer send --json --file <경로> <원격이름> "<본문>

   받았으면 네 디스코드 채널에 '수신 확인 — <첫 줄 요약>' 한 줄을 올려라.
   회신이 필요하면 셸에서 agentlayer send al-xxxxxx \"<회신>\" 으로 보내라."
   ```
   (`al-xxxxxx`가 없으면 먼저 `inbox open`·`wait`를 켠다.)
3. 출력의 `files`(원격 경로)·`handle`을 사용자에게 보고한다. 디스코드 게시는 사용자가 확인한다.

## 받을 준비만 — "메시지 받을 준비해"
`inbox open` + 백그라운드 `inbox wait`만 하고 주소를 알린다: "상대에게 `agentlayer send al-xxxxxx \"…\"` 로 보내라고 하세요."

## 끊기 — "연결 끊어"
`agentlayer inbox close --name <폴더명>`, `.baton/pair.json` 삭제. 남은 백그라운드 wait는 두어도 된다(편지 없이 타임아웃으로 끝난다).
