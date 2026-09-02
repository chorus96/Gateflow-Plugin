---
name: gf-audit
description: Audit plugin quality and optionally auto-fix issues
argument-hint: "[--fix] [--category agents|skills|ip|boards|docs]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - Task
---

# GateFlow Plugin Audit 커맨드

플러그인 전체를 스캔해 빈틈, 불일치, 누락된 콘텐츠를 찾습니다.

## 사용법

```
/gf-audit                    # Full audit, report only
/gf-audit --fix              # Audit and auto-fix all issues
/gf-audit --category ip      # Audit only IP blocks
/gf-audit --category agents  # Audit only agents
/gf-audit --category docs    # Audit only documentation
```

## 실행

### 보고 모드 (기본)
1. gf-auditor 에이전트를 스폰해 모든 것을 스캔
2. 우선순위가 매겨진 보고서 제시 (Critical → Low)
3. 어느 문제를 수정할지 사용자에게 질문

### 수정 모드 (--fix)
1. gf-auditor 에이전트를 스폰해 스캔
2. 결과와 함께 gf-pluginfixer 에이전트를 스폰
3. 모든 문제를 자동으로 수정
4. 변경 요약 표시
5. 수정 커밋

### 카테고리 모드
한 영역에 감사를 집중:
- `agents` — 에이전트 파일, 라우팅, 상호 참조 확인
- `skills` — 스킬 파일, 트리거, 반환 형식 확인
- `ip` — IP 블록 확인 (RTL, TB, formal, 메타데이터, 문서)
- `boards` — 보드 데이터베이스 확인 (yaml, 제약, 완전성)
- `docs` — README, releases, CLAUDE.md, AGENTS.md 일관성 확인

## 출력

```
GateFlow Plugin Audit Results
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Critical:  0
  High:      2
  Medium:    5
  Low:       3
  ─────────────
  Total:     10

[H1] README says 26 skills but 27 SKILL.md files found
     Fix: Update README count to 27

[H2] sv-formal agent not in orchestrator routing table
     Fix: Add to gf/SKILL.md Agent Routing section

...

Fix all issues? [Y/n/select]
```
