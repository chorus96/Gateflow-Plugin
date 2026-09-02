---
name: gf-summary
description: "Summarize Verilator, lint, or simulation output into a readable, actionable format. Use when the user wants to understand build output, lint errors, or simulation results from a Verilator or EDA tool run."
user-invocable: true
triggers:
  - summarize lint output
  - summarize Verilator output
  - what do these errors mean
  - parse build output
  - summarize simulation results
  - explain these lint warnings
  - make this output readable
  - summarize EDA tool output
---
allowed-tools:
  - Bash
  - Read

# GateFlow Summary 스킬

Verilator lint 및 시뮬레이션 출력을 깔끔하고 읽기 쉬운 보고서로 요약합니다.

## 지침

호출되면, 가장 최근 Verilator 출력을 분석하거나 새 lint 검사를 실행하고 결과를 다음과 같이 제시:

### 요약 형식

```
## Lint Summary: <filename>

| Status | Count |
|--------|-------|
| Errors | X |
| Warnings | X |
| Clean | Yes/No |

### Issues Found

#### Errors
- **Line X**: <error description> - <fix suggestion>

#### Warnings
- **Line X**: <warning description> - <fix suggestion>

### Recommendations
1. <prioritized fix>
2. <next fix>
```

## 트리거 패턴

사용자가 다음과 같이 말할 때 활성화:
- "summarize lint"
- "lint summary"
- "what's wrong with"
- "show lint results"

## 예시

**입력:** WIDTH 및 UNUSED 경고가 있는 Verilator 출력

**출력:**
```
## Lint Summary: sync_fifo.sv

| Status | Count |
|--------|-------|
| Errors | 0 |
| Warnings | 2 |
| Clean | No |

### Warnings
- **Line 15**: WIDTH - Signal 'count' width mismatch (8 vs 4 bits)
  - Fix: Change `logic [7:0] count` to `logic [3:0] count`

- **Line 32**: UNUSED - Signal 'debug_flag' is never used
  - Fix: Remove the signal or add `/* verilator lint_off UNUSED */`

### Recommendations
1. Fix WIDTH warning first (affects functionality)
2. Clean up UNUSED signals
```

## 사용법

```
/gf-summary                    # Summarize last lint output
/gf-summary sync_fifo.sv       # Run lint and summarize for specific file
```

파일 경로가 주어지면, 실행:
```bash
verilator --lint-only -Wall <file> 2>&1
```

그다음 파싱하고 요약을 제시.
