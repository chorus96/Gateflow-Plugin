---
name: gf-fusesoc
description: "FuseSoC build system integration for GateFlow. Generates .core files and drives synthesis/simulation through Edalize backends (Vivado, Quartus, open-source tools). Use when the user needs to create a FuseSoC core file, build with Edalize, or integrate RTL into a FuseSoC project."
user-invocable: true
triggers:
  - create a FuseSoC core file
  - build with Edalize
  - FuseSoC integration
  - generate .core file
  - Vivado build via FuseSoC
  - Quartus synthesis with FuseSoC
  - set up FuseSoC project
  - fusesoc build
---
allowed-tools:
  - Bash
  - Read

# GF-FuseSoC -- 빌드 시스템 통합

FuseSoC .core 파일을 생성하고 Edalize 백엔드를 통해 빌드를 구동합니다.

## .core 파일 템플릿

```yaml
CAPI=2:
name: ::my_project:1.0.0
filesets:
  rtl:
    files: [rtl/top.sv, rtl/fifo.sv]
    file_type: systemVerilogSource
  tb:
    files: [tb/tb_top.sv]
    file_type: systemVerilogSource
  constraints:
    files:
      - constraints/arty_a7.xdc: {file_type: xdc}
targets:
  sim:
    filesets: [rtl, tb]
    toplevel: tb_top
    default_tool: verilator
  synth:
    filesets: [rtl, constraints]
    toplevel: top
    default_tool: vivado
    tools:
      vivado:
        part: xc7a35ticsg324-1L
```

## 완전한 .core 스키마

```yaml
CAPI=2
name: vendor:library:name:version    # VLNV identifier (required)
description: "Brief description"

filesets:
  rtl:
    file_type: systemVerilogSource
    files:
      - rtl/top.sv
      - rtl/core.sv
    depend:
      - ">=vendor:lib:dep:1.0"
  tb:
    files: [tb/tb_top.sv]
    file_type: systemVerilogSource
  constraints_ice40:
    files: [constraints/ice40.pcf]
    file_type: PCF
  constraints_ecp5:
    files: [constraints/ecp5.lpf]
    file_type: LPF

targets:
  default:
    filesets: [rtl]
    toplevel: top
  sim:
    default_tool: verilator
    filesets: [rtl, tb]
    toplevel: tb_top
    tools:
      verilator:
        mode: cc
        verilator_options: [--trace, --coverage]
      icarus:
        iverilog_options: [-g2012]
  synth_ice40:
    default_tool: icestorm
    filesets: [rtl, constraints_ice40]
    toplevel: top
    tools:
      icestorm:
        pnr: next
        nextpnr_options: [--hx8k, --package, ct256, --freq, "48"]
  synth_ecp5:
    default_tool: trellis
    filesets: [rtl, constraints_ecp5]
    toplevel: top
    tools:
      trellis:
        nextpnr_options: [--25k, --package, CABGA256]
  synth_vivado:
    default_tool: vivado
    filesets: [rtl]
    toplevel: top
    tools:
      vivado:
        part: xc7a35tcpg236-1
  lint:
    default_tool: verilator
    filesets: [rtl]
    toplevel: top
    tools:
      verilator:
        mode: lint-only

parameters:
  WIDTH:
    datatype: int
    paramtype: vlogparam
    default: 8
    description: "Data width"
```

## 파일 타입

| 타입 | 확장자 |
|---|---|
| `verilogSource` | .v, .vh |
| `systemVerilogSource` | .sv, .svh |
| `vhdlSource` | .vhd, .vhdl |
| `xdc` | Vivado 제약 |
| `PCF` | iCE40 제약 |
| `LPF` | ECP5 제약 |
| `CST` | Gowin 제약 |

## Edalize 백엔드

| 백엔드 | 도구 | 용도 |
|---------|------|----------|
| verilator | Verilator | 시뮬레이션 + lint |
| icarus | Icarus Verilog | 시뮬레이션 |
| vivado | Xilinx Vivado | Synth + P&R |
| quartus | Intel Quartus | Synth + P&R |
| yosys | Yosys | 오픈소스 synth |

## 의존성 관리

```yaml
depend:
  - vendor:lib:uart:1.0         # Exact
  - ">=vendor:lib:spi:2.0"     # Minimum
  - "^vendor:lib:i2c:1.2"      # Semver compatible
  - "~vendor:lib:fifo:1.2.3"   # Patch only
```

| 연산자 | 의미 |
|---|---|
| `=` (기본) | 정확한 일치 |
| `>=` | 최소 |
| `^` | Semver 호환 (>=X.Y.Z, <X+1.0.0) |
| `~` | Patch만 (>=X.Y.Z, <X.Y+1.0) |

## FuseSoC 실행

```bash
fusesoc run --target=sim vendor:lib:design           # Simulate
fusesoc run --target=sim --tool=icarus vendor:lib:design  # Specific simulator
fusesoc run --target=synth_ice40 vendor:lib:design    # Synthesize
fusesoc run --target=lint vendor:lib:design           # Lint only
fusesoc core list                                      # List cores
fusesoc library add name https://github.com/org/repo  # Add library
```

## 도구 백엔드 옵션

### icestorm
- `pnr`: `next` (nextpnr) 또는 `arachne`
- `nextpnr_options`: nextpnr-ice40의 CLI 인자
- `yosys_synth_options`: 추가 synth_ice40 옵션

### trellis (ECP5)
- `nextpnr_options`: nextpnr-ecp5의 CLI 인자
- `yosys_synth_options`: 추가 synth_ecp5 옵션

### verilator
- `mode`: `binary`, `cc`, `lint-only`
- `verilator_options`: 추가 CLI 인자

### vivado
- `part`: FPGA 부품 번호
- `synth`: `vivado` 또는 `yosys`

## 자동 생성

프로젝트를 스캔하고, `.gateflow/project.yaml`을 읽어 .core 파일을 생성.

## GATEFLOW-RESULT 통합

```
---GATEFLOW-RESULT---
STATUS: PASS | FAIL | ERROR
FILES: [generated .core file]
DETAILS: [summary]
---END-GATEFLOW-RESULT---
```
