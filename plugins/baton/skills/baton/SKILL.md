---
name: baton
description: 이 Claude 세션과 다른 앱의 에이전트(코덱스 앱 세션·헤르메스)를 대화로 연결해 일을 넘기고 답을 받는다. "코덱스 <세션ID>와 연결해", "코덱스에 (이미지 만들어 달라고) 시켜", "이거 코덱스로 넘겨", "헤르메스에게 이 파일(스킬) 보내", "메시지 받을 준비해", "헤르메스(디스코드) 편지도 받아", "연결 끊어", "/baton" 에 사용. 터미널 없이 데스크톱 앱 안에서만 쓴다. 한 번 연결하면 그 뒤로는 사용자가 세션 번호를 다시 말하지 않아도 이어진다.
---

# baton — 다른 앱과 대화 연결

## 준비(한 번)
`agentlayer version`이 1.17.0 이상인지 확인한다(`--skill`은 1.15.0부터, 윈도우 코덱스 앱 연결은 1.17.0부터). 없거나 낮으면 앱 안 Bash로 설치하고 다시 확인한다.
- **맥**: `brew install netwaif/tap/agentlayer`(이미 있으면 `brew upgrade netwaif/tap/agentlayer`). brew가 없으면 아래 스크립트를 쓴다.
- **리눅스·윈도우 WSL2**(brew 없는 맥도): `curl -fsSL https://raw.githubusercontent.com/netwaif/agentlayer/main/install.sh | bash` — 릴리즈 파일을 `~/.local/bin/agentlayer`에 놓는다.
- **윈도우(WSL 밖)**: 바이너리가 없다. "WSL2 안에서 연 세션에서만 됩니다"라고 알리고 멈춘다.

설치했는데 `agentlayer`를 못 찾으면 PATH 문제다. `~/.local/bin/agentlayer`·`/opt/homebrew/bin/agentlayer`·`/usr/local/bin/agentlayer` 중 있는 절대 경로로 부르고, 상대에게 보내는 회신 명령에도 그 절대 경로를 적는다.
상대(코덱스·헤르메스)에는 이 스킬을 설치하지 않는다. 같은 컴퓨터의 `agentlayer`를 셸로 한 줄 치기만 한다.

## 규칙 세 가지
- `agentlayer send`의 플래그(`--json`·`--file`·`--skill`·`--cwd`)는 **대상 앞에** 쓴다. 뒤에 쓰면 본문으로 들어간다.
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
4. "코덱스와 연결됐습니다. 답이 오면 알려 드립니다." 한 줄. 윈도우(WSL 세션)면 "코덱스 앱에 WSL 접근 승인창이 뜨면 Approve for session을 누르세요"를 덧붙인다 — 코덱스 앱은 WSL 밖이라 답을 보낼 때 한 번 묻는다.

## 시키기 — "코덱스에 … 시켜"(연결된 뒤)
`.baton/pair.json`의 세션 ID로 `agentlayer send --json --cwd <cwd> <codex_session> "<요청> (회신 주소 al-xxxxxx)"`. 직전 wait가 끝나 있으면 다시 켠다. 연결 정보가 없으면 "연결하기"부터.

## 답 받기(백그라운드 wait가 끝났을 때)
출력의 `from:` 다음 본문이 답이다. 파일 경로가 있으면 열어 확인하고 사용자에게 보고한다. 처리 뒤 **곧바로** `inbox wait --name <폴더명> --timeout 2h`를 다시 켠다.
`답 없음(2h)`으로 끝났으면: "연결이 2시간 동안 조용해 대기를 멈췄습니다. 다시 기다릴까요?" 주소는 남아 있으니 다시 wait만 켜면 이어진다.

## 헤르메스 — "헤르메스에게 이 스킬(파일) 보내"
사용자는 "헤르메스에게 <스킬 이름> 보내" 한마디면 된다. 묶기(폴더면 zip)·설치 위치·회신 규칙은 `agentlayer send`가 정해서 본문에 붙인다 — 스킬이 본문을 지어내지 않는다.
1. `agentlayer remote list`로 원격 이름을 확인한다(하나면 그것, 여럿이면 묻는다). 없으면 등록 절을 안내하고 멈춘다: `agentlayer remote add <이름> --kind hermes --local --profile <프로필> --workspace-root <절대경로>` (같은 컴퓨터의 헤르메스), VPS면 `--ssh <호스트>`. 등록이 헤르메스에 스킬을 깔아 주는데, **헤르메스가 이미 돌고 있으면 한 번 다시 띄워야 그 스킬을 본다**(등록 출력에도 나온다) — 사용자에게 그 한 줄을 알린다.
2. 스킬이면 `--skill`, 그냥 파일이면 `--file`. 본문은 사용자가 시킨 일만. 시킨 일이 없으면 본문을 생략한다 — 기본은 "헤르메스가 자기 환경에서 쓸 수 있는지 판단해, 되면 설치하고 경로를, 안 되면 이유를 답한다":
   ```
   agentlayer send --json --skill <스킬 이름> <원격이름> "<사용자가 시킨 일>"
   agentlayer send --json --file <경로> <원격이름> "<사용자가 시킨 일>"
   ```
   stderr에 "개인 정보일 수 있는 내용" 경고가 나오면 그 파일 이름을 사용자에게 알린다(전송은 이미 됐다).
3. 출력의 `handle`(카드, `t_…`)로 답을 기다린다(백그라운드 Bash): `agentlayer inbox wait --name <폴더명> --timeout 2h --remote <원격이름>:<handle>`. 카드가 끝나면 `from: <원격이름>` 다음에 결과 전문이 오고, 질문이면 `[WAITING] …`이 온다. 헤르메스의 답은 카드 완료 결과로 돌아온다(회신 주소 `al-…`는 헤르메스와 무관).
- 디스코드에는 헤르메스가 수신 확인 한 줄과 결과 첫 줄을 올린다. 사용자가 디스코드에서 먼저 "내 pc에서 … 가져올게"라고 예고했으면 그 대화(스레드)에, 아니면 헤르메스 홈 채널에 올라간다 — 이것도 `agentlayer send`가 정하므로 스킬이 신경 쓸 일은 없다.

## 디스코드에서 온 요청 받기 — "헤르메스(디스코드) 편지도 받아"
사용자가 디스코드에서 헤르메스에게 "Claude에게 … 전해 줘"라고 하면 헤르메스가 이 세션 앞으로 편지를 보낸다.
1. 원격 이름을 `agentlayer remote list`로 확인한다. 처음 한 번은 `agentlayer remote setup <원격이름>`으로 헤르메스 쪽에 편지 명령(`claude-letter`)을 깐다.
2. 백그라운드 Bash로 기다린다(연결돼 있으면 같은 대기에 옵션만 더한다):
   `agentlayer inbox wait --name <폴더명> --timeout 2h --remote <원격이름> --app-mailbox`
3. 편지가 오면 출력이 `from: <원격이름>/<보낸이>`, `letter: <원격이름>:<카드ID>`, 빈 줄, 본문 순서다. 본문의 요청을 수행한다.
4. 답은 `letter:` 줄의 값 그대로 보낸다(파일은 `--file`, 대상 앞에):
   `agentlayer inbox reply --file <경로> <원격이름>:<카드ID> "<답>"`
   답하면 디스코드의 그 대화에 알림이 뜬다. **알림에는 답의 첫 줄(160자)만 보인다.** 인사말·머리말 없이 핵심을 첫 줄에 쓴다. 항목이 여럿이면 첫 줄에 ` / `로 이어 쓰고, 자세한 내용은 둘째 줄부터 쓴다(전문은 사용자가 헤르메스에게 "Claude 답 보여 줘"라고 하면 읽어 준다).
   답한 뒤 **곧바로** 2번 대기를 다시 켠다.

## 받을 준비만 — "메시지 받을 준비해"
`inbox open` + 백그라운드 `inbox wait`만 하고 주소를 알린다: "상대에게 `agentlayer send al-xxxxxx \"…\"` 로 보내라고 하세요."

## 끊기 — "연결 끊어"
`agentlayer inbox close --name <폴더명>`, `.baton/pair.json` 삭제. 남은 백그라운드 wait는 두어도 된다(편지 없이 타임아웃으로 끝난다).
