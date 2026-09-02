---
name: gf-synth
description: >
  Synthesize SystemVerilog/Verilog with Yosys. Reports area, timing,
  and resource utilization. Warns about unsupported SV constructs.
  Example: "synthesize my design for iCE40"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - Task
---

# GF-Synth — Yosys 합성 스킬

## 도구 감지

```bash
which yosys
```

찾을 수 없으면:
```
---GATEFLOW-RESULT---
STATUS: ERROR
DETAILS: Yosys not installed. Install to enable synthesis.
  macOS: brew install yosys
  Linux: sudo apt install yosys
---END-GATEFLOW-RESULT---
```

## 합성 전 SV 서브셋 검사

합성 전에 미지원 구문을 스캔:
```bash
grep -rn "^\s*interface\s\|^\s*modport\s\|^\s*class\s\|^\s*bind\s" <files>
```

발견되면 사용자에게 경고. 진행하지 말 것 — 혼란스러운 오류를 유발함.

## 워크플로

1. 타겟을 위해 프로젝트 컨텍스트(`.gateflow/project.yaml`) 확인
2. 미지원 SV 구문에 대한 합성 전 lint
3. 보드를 Yosys synth 타겟에 매핑
4. sv-synth 에이전트를 통해 또는 직접 합성 실행
5. stat 출력을 LUT/FF/BRAM/DSP로 파싱
6. 구조화된 결과 보고

## 결과 형식

```
---GATEFLOW-RESULT---
STATUS: PASS | FAIL | ERROR
RESOURCES:
  LUTs: N
  FFs: N
  BRAM: N
  DSP: N
TARGET: ice40 | ecp5 | gowin | xilinx | generic
FILES: [synth output files]
DETAILS: [summary or error explanation]
---END-GATEFLOW-RESULT---
```
