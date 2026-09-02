---
name: gf-project
description: >
  Manages .gateflow/project.yaml for project-specific configuration.
  Auto-detects project settings or prompts user for board, HDL, and target.
  Used internally by other skills — not typically invoked directly.
user-invocable: false
---

# GF Project — 프로젝트 컨텍스트 관리

## 프로젝트 파일 위치

프로젝트 루트의 `.gateflow/project.yaml`.

## 스키마

```yaml
name: my-project
top_module: top
hdl: systemverilog  # systemverilog | verilog | vhdl
target:
  board: null       # arty-a7-35t, icebreaker, etc.
  device: null      # xc7a35ticsg324-1L, ice40hx8k, etc.
  clock_freq: null  # 100MHz, 12MHz, etc.
sources: []         # auto-populated by gf-scan
constraints: null   # path to constraint file
ip_blocks: []       # installed IP block names
```

## 자동 감지

이 스킬이 호출되면:

1. `.gateflow/project.yaml`이 존재하는지 확인
2. 없으면, 스캔하여 기본값으로 생성:
   - HDL: 프로젝트의 파일 확장자를 확인 (`.sv` = systemverilog, `.v` = verilog, `.vhd` = vhdl)
   - Sources: `**/*.sv`, `**/*.v`, `**/*.vhd`를 glob
   - Top module: 다른 것에 의해 인스턴스화되지 않은 모듈을 찾음
3. 보드가 설정되지 않았고 사용자가 보드를 언급하면, 갱신

## 다른 스킬의 사용

프로젝트 컨텍스트가 필요한 스킬은:

```bash
cat .gateflow/project.yaml 2>/dev/null
```

파일이 없으면, 이 스킬을 호출해 생성.

## 보드 기억

사용자가 자연어로 보드 이름을 언급하면(예: "I'm using an Arty A7"),
`.gateflow/project.yaml`의 `target.board`에 지속. 사용자가 보드를 바꾸거나
새 프로젝트를 시작하지 않는 한 다시 묻지 말 것.

보드 기억은 전역이 아니라 프로젝트별입니다. 서로 다른 프로젝트는 서로 다른 보드를 대상으로 합니다.

## GATEFLOW-RESULT 형식

```
---GATEFLOW-RESULT---
STATUS: CREATED | UPDATED | VALID | INVALID | ERROR
PROJECT: <name>
FILE: .gateflow/project.yaml
BOARD: <board or null>
HDL: systemverilog | verilog | vhdl
TOP_MODULE: <module>
SOURCES: <count>
DETAILS: <summary>
---END-GATEFLOW-RESULT---
```

## 확장 스키마

```yaml
simulation:
  tool: verilator        # verilator | iverilog | vcs
  timeout: 100000        # max cycles
  trace_format: fst      # vcd | fst
  defines: []            # compile-time defines

synthesis:
  tool: yosys            # yosys | vivado | quartus
  target_family: null    # ice40 | ecp5 | gowin | xilinx
  optimization: area     # area | speed | balanced

verification:
  formal_engine: sby
  formal_depth: 50
  coverage_goal: 90
  lint_tool: verilator
```

## 프로젝트 템플릿

### iCE40 FPGA
```yaml
name: my-project
top_module: top
hdl: systemverilog
target: {board: icebreaker, device: ice40up5k-sg48, clock_freq: 12MHz}
simulation: {tool: verilator, trace_format: fst}
synthesis: {tool: yosys, target_family: ice40, optimization: area}
```

### 시뮬레이션 전용
```yaml
name: sim-project
top_module: top
hdl: systemverilog
target: {board: null}
simulation: {tool: verilator, timeout: 500000, trace_format: fst, defines: [SIMULATION, DEBUG]}
synthesis: {tool: null}
```

## 헬스 체크

| 검사 | 통과 조건 |
|---|---|
| 스키마 유효 | name, top_module, hdl 존재 |
| Sources 존재 | 나열된 모든 파일이 발견됨 |
| Top module 발견 | sources에서 `module <top>`를 grep |
| 제약이 타겟과 일치 | Xilinx는 .xdc, iCE40은 .pcf, ECP5는 .lpf, Gowin은 .cst |
| 도구 설치됨 | which <sim_tool>, which <synth_tool> |
