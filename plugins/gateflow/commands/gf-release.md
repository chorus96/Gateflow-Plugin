---
name: gf-release
description: Validate GateFlow plugin readiness and prepare a versioned release
argument-hint: "[--version X.Y.Z] [--check-only]"
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - Glob
  - Grep
---

# GateFlow Release 커맨드

GateFlow 플러그인 릴리스를 준비하고 검증합니다.

## 사용법

```
/gf-release --check-only
/gf-release --version 2.5.0
```

## 워크플로

1. `gf-release` 스킬을 호출.
2. 결정적 검증기를 실행:

```bash
python3 tools/validate_gateflow.py --version <version>
```

3. 검증이 실패하면, 태깅 전에 보고된 패키지 연결 문제를 수정.
4. `plugins/gateflow/.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`,
   `README.md`, `plugins/gateflow/README.md`, `docs/gateflow.index`, `releases.md`이
   모두 동일한 버전과 구성 요소 개수를 기술하는지 확인.
5. 실제 릴리스의 경우, 검증이 통과한 후에만 git 태그와 GitHub 릴리스를 생성.

## 출력

보고:
- 준비 중인 버전
- 구성 요소 인벤토리
- 검증 실패(있는 경우)
- 릴리스 전에 변경해야 하는 파일
- 준비되면 최종 태그 커맨드
