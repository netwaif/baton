# baton

데스크톱 앱 안의 Claude Code를 코덱스 앱·헤르메스와 **대화로 연결**한다. 한 번 연결하면 그 뒤로는 "코덱스에 이거 시켜"만으로 일이 넘어가고 답이 돌아온다. 터미널은 등장하지 않는다.

설치(앱의 Claude Code 입력창에서):
```
/plugin marketplace add netwaif/baton
/plugin install baton@baton
```
필요한 바이너리 `agentlayer`(1.12.0+)는 스킬이 첫 실행에 설치한다 — 맥은 brew, 리눅스·윈도우 WSL2는 설치 스크립트. 윈도우는 WSL2 안에서만 된다(WSL 밖 바이너리 없음).
코덱스·헤르메스 쪽에는 아무것도 설치하지 않는다. 같은 컴퓨터의 `agentlayer`만 있으면 된다.

동작 원리: 세션이 `agentlayer inbox open`으로 고유 주소를 받고, 상대(코덱스·헤르메스)는 `agentlayer send <주소> "…"`로 회신한다. 세션은 `inbox wait`를 백그라운드로 켜 두어 답이 오면 받는다. 자세한 설계: agentlayer 레포 `docs/superpowers/specs/2026-09-30-app-handoff-design.md`.
