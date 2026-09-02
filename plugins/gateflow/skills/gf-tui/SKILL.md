---
name: gf-tui
description: >
  GateFlow terminal console inspired by OpenClaw local TUI workflows. Shows
  workspace status, component inventory, tool health, map readiness, release
  readiness, and command shortcuts from one terminal surface.
user-invocable: true
triggers:
  - open GateFlow TUI
  - show GateFlow dashboard
  - terminal console
  - CLI TUI
  - OpenClaw-style TUI
---
allowed-tools:
  - Bash
  - Read

# GF-TUI -- 터미널 콘솔

GateFlow를 위한 로컬 터미널 콘솔을 실행합니다.

## 모드

| 모드 | 커맨드 | 사용 시점 |
|---|---|---|
| CLI | `python3 tools/gateflow_cli.py status` | 일반 커맨드 화면을 원할 때 |
| Shell | `python3 tools/gateflow_cli.py shell` | 로컬 `gateflow>` 프롬프트를 원할 때 |
| Agent create | `python3 tools/gateflow_cli.py agents create "Name"` | 새 커스텀 에이전트가 필요할 때 |
| Interactive | `python3 tools/gateflow_tui.py` | 실제 TTY에 있고 키보드 내비게이션을 원할 때 |
| Snapshot | `python3 tools/gateflow_tui.py --snapshot --plain` | 로그, CI, 비대화형 터미널 |
| JSON | `python3 tools/gateflow_tui.py --json` | 스크립트가 기계 판독 상태가 필요할 때 |

## OpenClaw 영감 동작

- 기본적으로 로컬 워크스페이스 모드; 게이트웨이가 필요 없음.
- 일반 및 JSON 폴백을 갖춘 TTY 인식 스타일링.
- 상태 및 에이전트 관리를 위한 커맨드 중심 로컬 CLI.
- 대시보드에서 `a`를 눌러 새 에이전트를 대화형으로 생성.
- 헬스/상태 화면이 조치 전에 보임.
- 커맨드가 숨겨진 문서가 아니라 오퍼레이터 단축키로 표시됨.
- 릴리스 및 구성 복구 루프가 터미널 워크플로 내부에 머무름.

## 콘솔 섹션

1. **Workspace** — 루트 경로와 플러그인 버전.
2. **Inventory** — 에이전트, 스킬, 커맨드, IP 블록, 보드.
3. **Health** — doctor 커맨드, 릴리스 검증, 맵 준비 상태, Verilator,
   Yosys, SymbiYosys 가용성.
4. **Actions** — doctor, map, viz, lint, sim, formal, release 워크플로의
   실행 지점.
5. **Agent creation** — `a`가 이름, 역할, 설명 프롬프트를 엶.

## 가드레일

- TUI는 파괴적인 하드웨어 동작을 조용히 실행하지 않음.
- `/gf-flash`는 보드 프로그래밍이 명시적으로 유지되어야 하므로 기본 액션
  목록 밖에 남음.
- 누락된 도구는 실패가 아니라 경고로 표시됨. 모든 선택 도구가 설치되지
  않아도 GateFlow는 여전히 RTL을 생성하고 리뷰할 수 있기 때문.

## 검증

실행:

```bash
python3 -m unittest tests/test_gateflow_tui.py
python3 -m unittest tests/test_gateflow_cli.py
python3 tools/gateflow_cli.py --plain status
python3 tools/gateflow_tui.py --snapshot --plain
python3 tools/gateflow_tui.py --json
```
