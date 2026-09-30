# baton

데스크톱 앱 안의 Claude Code를 코덱스 앱·헤르메스와 **대화로 연결**한다. 한 번 연결하면 그 뒤로는 "코덱스에 이거 시켜"만으로 일이 넘어가고 답이 돌아온다. 터미널은 등장하지 않는다.

설치(앱의 Claude Code 입력창에서):
```
/plugin marketplace add netwaif/baton
/plugin install baton@baton
```
필요한 바이너리 `agentlayer`(1.12.0+)는 스킬이 첫 실행에 brew로 설치한다.

동작 원리: 세션이 `agentlayer inbox open`으로 고유 주소를 받고, 상대(코덱스·헤르메스)는 `agentlayer send <주소> "…"`로 회신한다. 세션은 `inbox wait`를 백그라운드로 켜 두어 답이 오면 받는다. 자세한 설계: agentlayer 레포 `docs/superpowers/specs/2026-09-30-app-handoff-design.md`.
