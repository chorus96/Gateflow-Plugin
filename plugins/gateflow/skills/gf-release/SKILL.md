---
name: gf-release
description: >
  GateFlow release readiness workflow. Validates plugin manifests, marketplace
  metadata, docs index coverage, root mirrors, release notes, and component
  counts before a version tag is created. Use when preparing, checking, or
  cutting a GateFlow plugin release.
user-invocable: true
triggers:
  - prepare GateFlow release
  - cut a GateFlow release
  - validate plugin release
  - check marketplace metadata
  - bump GateFlow version
---
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - Glob
  - Grep

# GF-Release -- 릴리스 준비 상태

GateFlow를 태깅하거나 게시하기 전에 이 워크플로를 사용하세요.

## 필수 입력

- 목표 버전, 예를 들어 `2.5.0`
- 이것이 검사 전용인지 릴리스 준비 편집인지 여부

버전이 제공되지 않으면, `plugins/gateflow/.claude-plugin/plugin.json`을
검토하고 변경 사항에 기반해 다음 semver 버전을 제안:

| 변경 유형 | 버전 상향 |
|---|---|
| 메타데이터, 문서, 패키징 수정 | Patch |
| 신규 커맨드, 스킬, 에이전트, IP, 보드 | Minor |
| 플러그인 레이아웃 또는 워크플로의 파괴적 변경 | Major |

## 릴리스 검사

저장소 루트에서 결정적 검증기를 실행:

```bash
python3 tools/validate_gateflow.py --version <target-version>
```

태그를 만들기 전에 검증기가 통과해야 합니다. 검사 항목:
- 플러그인과 마켓플레이스 JSON 버전 일관성
- 설명과 README의 구성 요소 개수
- 모든 커맨드, 스킬, 에이전트에 대한 `docs/gateflow.index` 커버리지
- 모든 커맨드 인접 스킬 및 에이전트 파일의 루트 레벨 미러
- 목표 버전에 대한 릴리스 노트 항목

## 준비 워크플로

1. 실제 구성 요소 개수 세기:

```bash
find plugins/gateflow/agents -maxdepth 1 -name '*.md' | wc -l
find plugins/gateflow/skills -maxdepth 2 -name 'SKILL.md' | wc -l
find plugins/gateflow/commands -maxdepth 1 -name '*.md' | wc -l
```

2. 버전 문자열 갱신:
- `plugins/gateflow/.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`

3. 릴리스 대상 문서 갱신:
- `README.md`
- `plugins/gateflow/README.md`
- `docs/gateflow.index`
- `releases.md`

4. 검증기와 집중 테스트 실행:

```bash
python3 -m unittest tests/test_validate_gateflow.py
python3 tools/validate_gateflow.py --version <target-version>
```

5. 검증이 통과한 후에만 태그하고 릴리스:

```bash
git tag v<target-version>
gh release create v<target-version> --title "GateFlow v<target-version>" --notes-file <notes-file>
```

## 보고 형식

```text
---GATEFLOW-RESULT---
STATUS: PASS | FAIL
VERSION: <target-version>
INVENTORY: <agents> agents, <skills> skills, <commands> commands, <ip> IP, <boards> boards
FILES: <changed release-facing files>
DETAILS: <summary or validation failures>
---END-GATEFLOW-RESULT---
```
