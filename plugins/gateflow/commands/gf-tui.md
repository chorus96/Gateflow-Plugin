---
name: gf-tui
description: Open the GateFlow terminal console
argument-hint: "[--snapshot] [--json] [--plain]"
allowed-tools:
  - Bash
  - Read
---

# GateFlow TUI 커맨드

로컬 GateFlow 터미널 콘솔과 로컬 CLI를 엽니다.

## 사용법

```
/gf-tui
/gf-tui --snapshot
/gf-tui --json
```

## 실행

저장소 루트에서 커맨드 중심 CLI를 실행:

```bash
python3 tools/gateflow_cli.py status
python3 tools/gateflow_cli.py agents list
python3 tools/gateflow_cli.py agents create "CDC Reviewer" \
  --role "clock-domain crossing reviewer" \
  --description "Reviews synchronizers and CDC constraints"
python3 tools/gateflow_cli.py shell
```

키보드 대시보드 열기:

```bash
python3 tools/gateflow_tui.py
```

대시보드 내부에서 `a`를 눌러 새 에이전트를 생성.

비대화형 터미널에서 실행할 때는 스냅샷 모드 사용:

```bash
python3 tools/gateflow_tui.py --snapshot --plain
```

## 표시하는 것

- 플러그인 버전과 워크스페이스 경로
- 구성 요소 인벤토리
- 로컬 하드웨어 도구 헬스
- 맵/릴리스 준비 상태
- 대화형 에이전트 생성
- `/gf-doctor`, `/gf-map`, `/gf-viz`, `/gf-lint`, `/gf-sim`,
  `/gf-formal`, `/gf-release`를 위한 빠른 액션
