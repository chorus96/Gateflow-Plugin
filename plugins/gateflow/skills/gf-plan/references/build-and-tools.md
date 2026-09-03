## 합성 계획

### 합성 전략

```markdown
## Synthesis Plan

### Target
| Attribute | Value |
|-----------|-------|
| Target | FPGA / ASIC |
| Device | Xilinx Artix-7 / ASIC 28nm |
| Clock | 100 MHz |
| Area budget | 10k LUTs / 50k gates |

### Synthesis Tool
| Tool | Command |
|------|---------|
| Yosys (open) | `yosys -p "read_verilog *.sv; synth_xilinx -top module"` |
| Vivado | `vivado -mode batch -source synth.tcl` |
| Design Compiler | `dc_shell -f synth.tcl` |

### Timing Constraints (SDC)
| Constraint | Value |
|------------|-------|
| Clock period | 10ns (100MHz) |
| Input delay | 2ns |
| Output delay | 2ns |
| False paths | async resets, CDC |
| Multicycle | [if any] |
```

**SDC 템플릿:**
```tcl
# Clock definition
create_clock -name clk -period 10.0 [get_ports clk]

# Input/output delays
set_input_delay -clock clk 2.0 [all_inputs]
set_output_delay -clock clk 2.0 [all_outputs]

# Reset is async - false path
set_false_path -from [get_ports rst_n]

# CDC false paths
set_false_path -from [get_clocks clk_a] -to [get_clocks clk_b]

# Multicycle path (if needed)
set_multicycle_path 2 -setup -from [get_pins */slow_reg*/Q]
```

### 리소스 추정

```markdown
| Resource | Estimate | Budget | Notes |
|----------|----------|--------|-------|
| LUTs | ~2000 | 10000 | FSM + logic |
| FFs | ~500 | 5000 | State + pipeline |
| BRAM | 2 | 10 | FIFOs |
| DSP | 0 | 4 | No math |
```

### 합성 체크리스트
- [ ] 모든 코드가 합성 가능 (`initial`, `#delays` 없음)
- [ ] 래치가 추론되지 않음
- [ ] 전력을 위한 클럭 게이팅 셀
- [ ] 리셋 전략이 타겟과 일치
- [ ] 타이밍 제약 정의
- [ ] 리소스 추정치 허용 가능

---

## 파형 & 디버그 계획

### 디버그 인프라

```markdown
## Debug Plan

### Simulation Dumps
| Format | Tool | Command |
|--------|------|---------|
| VCD | All | `$dumpfile("dump.vcd"); $dumpvars(0, tb);` |
| FST | Verilator | `--trace-fst` flag |
| FSDB | VCS | `$fsdbDumpfile` |

### Waveform Viewing
| Tool | Platform | Usage |
|------|----------|-------|
| GTKWave | All | `gtkwave dump.vcd` |
| Surfer | All (Rust) | `surfer dump.vcd` |
| DVE | VCS | `dve -vpd dump.vpd` |
| Vivado | Xilinx | Integrated |

### Key Signals to Observe
| Signal | Module | Why |
|--------|--------|-----|
| state | FSM | State transitions |
| valid/ready | interfaces | Handshake |
| data | datapath | Values |
| error | all | Failures |
```

**파형 덤프 템플릿:**
```systemverilog
initial begin
    // VCD dump
    $dumpfile("dump.vcd");
    $dumpvars(0, tb_top);         // All signals
    // Or selective:
    // $dumpvars(1, tb_top.u_dut); // One level
    // $dumpvars(0, tb_top.u_dut.state); // Specific signal
end

// FST for Verilator (faster, smaller)
// Use: verilator --trace-fst

// Conditional dump (large sims)
initial begin
    $dumpfile("dump.vcd");
    #1000;  // Skip reset
    $dumpvars(0, tb_top);
    #100000;
    $dumpoff;  // Stop dumping
end
```

### 디버그 어서션

```systemverilog
// Debug helpers
always @(posedge clk) begin
    if (state == ERROR)
        $display("[%0t] ERROR state entered, cause=%b", $time, error_cause);
    if (fifo_overflow)
        $warning("[%0t] FIFO overflow!", $time);
end

// Transaction logging
always @(posedge clk) begin
    if (valid && ready)
        $display("[%0t] Transfer: addr=%h data=%h", $time, addr, data);
end
```

---

## Formal 검증 계획

### Formal 전략

```markdown
## Formal Verification Plan

### Tool Selection
| Tool | Type | Usage |
|------|------|-------|
| SymbiYosys | Open source | Property checking |
| JasperGold | Commercial | Full formal |
| VC Formal | Commercial | Full formal |

### Properties to Prove
| Property | Type | Bounded/Unbounded |
|----------|------|-------------------|
| No deadlock | Safety | Unbounded |
| Request→Ack | Liveness | Bounded (100 cycles) |
| FIFO no overflow | Safety | Unbounded |
| FSM reachability | Cover | Bounded |

### Assumptions (Constraints)
| Assumption | Purpose |
|------------|---------|
| Valid inputs only | Constrain input space |
| Reset sequence | Proper initialization |
| Protocol compliance | Assume valid stimulus |
```

**SymbiYosys 템플릿 (.sby):**
```
[options]
mode bmc        # Bounded model checking
depth 100       # 100 cycles

[engines]
smtbmc z3

[script]
read -formal top.sv
prep -top top

[files]
top.sv
```

**Formal 프로퍼티:**
```systemverilog
// Assume valid inputs
assume property (@(posedge clk) disable iff (!rst_n)
    cmd inside {CMD_READ, CMD_WRITE, CMD_NOP});

// Assert safety property
assert property (@(posedge clk) disable iff (!rst_n)
    fifo_count <= FIFO_DEPTH);

// Assert liveness
assert property (@(posedge clk) disable iff (!rst_n)
    req |-> ##[1:100] ack);

// Cover reachability
cover property (@(posedge clk) state == DONE);
```

---

## 빌드 시스템 계획

### 빌드 인프라

```markdown
## Build System

### Makefile Targets
| Target | Action |
|--------|--------|
| lint | Run Verilator lint |
| sim | Compile and simulate |
| synth | Run synthesis |
| formal | Run formal checks |
| clean | Remove generated files |
| all | lint + sim |

### File Lists
| File | Purpose |
|------|---------|
| rtl.f | RTL source files |
| tb.f | Testbench files |
| pkg.f | Packages (compile first) |
```

**Makefile 템플릿:**
```makefile
# GateFlow Makefile
TOP = dma_top
TB = tb_$(TOP)

# Tool selection
SIM ?= verilator
SYNTH ?= yosys

# File lists
RTL_FILES = $(shell cat rtl.f)
TB_FILES = $(shell cat tb.f)

#--- Targets ---

.PHONY: lint sim synth formal clean

lint:
	verilator --lint-only -Wall $(RTL_FILES)

sim: lint
ifeq ($(SIM),verilator)
	verilator --binary -j 0 -Wall --trace \
		$(RTL_FILES) $(TB_FILES) \
		--top-module $(TB) -o sim
	./obj_dir/sim
else
	iverilog -g2012 -o sim.vvp $(RTL_FILES) $(TB_FILES)
	vvp sim.vvp
endif

synth:
	yosys -p "read_verilog -sv $(RTL_FILES); \
		synth_xilinx -top $(TOP); \
		write_json $(TOP).json"

formal:
	sby -f $(TOP).sby

waves:
	gtkwave dump.vcd &

clean:
	rm -rf obj_dir *.vvp *.vcd *.fst *.json

# Regression
test: lint
	@for t in tests/*.sv; do \
		echo "Running $$t..."; \
		$(MAKE) sim TB=$$t || exit 1; \
	done
	@echo "All tests passed!"
```

**Filelist 템플릿 (rtl.f):**
```
# Packages first (order matters)
rtl/pkg/dma_pkg.sv

# RTL modules (leaf to top)
rtl/dma_channel.sv
rtl/dma_arbiter.sv
rtl/dma_engine.sv
rtl/dma_reg_if.sv
rtl/dma_top.sv
```

**FuseSoC core 파일 (IP 관리용):**
```yaml
CAPI=2:
name: ::dma:1.0.0
description: DMA Controller

filesets:
  rtl:
    files:
      - rtl/pkg/dma_pkg.sv: {is_include_file: false}
      - rtl/dma_channel.sv
      - rtl/dma_top.sv
    file_type: systemVerilogSource

  tb:
    files:
      - tb/tb_dma_top.sv
    file_type: systemVerilogSource

targets:
  default: &default
    filesets: [rtl]
    toplevel: dma_top

  sim:
    <<: *default
    filesets: [rtl, tb]
    toplevel: tb_dma_top
    default_tool: verilator
```

---

## FPGA 특화 계획

### FPGA 고려 사항

```markdown
## FPGA Implementation Plan

### Target Device
| Attribute | Value |
|-----------|-------|
| Family | Xilinx Artix-7 |
| Part | xc7a35tcpg236-1 |
| Speed grade | -1 |
| Package | cpg236 |

### Resource Usage
| Resource | Available | Estimated | % |
|----------|-----------|-----------|---|
| LUT | 20800 | 2000 | 10% |
| FF | 41600 | 1000 | 2% |
| BRAM | 50 | 4 | 8% |
| DSP | 90 | 0 | 0% |

### Clocking
| Clock | Source | Frequency |
|-------|--------|-----------|
| clk_sys | PLL from 100MHz | 100 MHz |
| clk_mem | PLL from 100MHz | 200 MHz |

### Pin Assignments
| Signal | Pin | Standard | Notes |
|--------|-----|----------|-------|
| clk | E3 | LVCMOS33 | Board oscillator |
| rst_n | C2 | LVCMOS33 | Button |
| led[0] | H5 | LVCMOS33 | Status |
```

**Vivado 제약 (XDC):**
```tcl
# Clock
set_property -dict {PACKAGE_PIN E3 IOSTANDARD LVCMOS33} [get_ports clk]
create_clock -period 10.0 -name sys_clk [get_ports clk]

# Reset
set_property -dict {PACKAGE_PIN C2 IOSTANDARD LVCMOS33} [get_ports rst_n]

# LEDs
set_property -dict {PACKAGE_PIN H5 IOSTANDARD LVCMOS33} [get_ports {led[0]}]

# Configuration
set_property CFGBVS VCCO [current_design]
set_property CONFIG_VOLTAGE 3.3 [current_design]
```

### FPGA 특화 코딩

```systemverilog
// Block RAM inference
(* ram_style = "block" *)
logic [31:0] mem [0:1023];

// Distributed RAM
(* ram_style = "distributed" *)
logic [7:0] small_mem [0:15];

// DSP inference
(* use_dsp = "yes" *)
logic [31:0] product;
assign product = a * b;

// Register for timing
(* KEEP = "TRUE" *)
logic [31:0] pipeline_reg;
```

### FPGA 디버그 (ILA)

```markdown
## Debug Cores

### ILA Probes
| Signal | Width | Trigger |
|--------|-------|---------|
| state | 3 | state == ERROR |
| data | 32 | valid && ready |
| count | 16 | count > 1000 |

### VIO Controls
| Signal | Width | Purpose |
|--------|-------|---------|
| force_reset | 1 | Manual reset |
| test_mode | 2 | Select test |
```

```tcl
# ILA insertion (Vivado)
create_debug_core u_ila ila
set_property C_DATA_DEPTH 4096 [get_debug_cores u_ila]
set_property C_TRIGIN_EN false [get_debug_cores u_ila]

connect_debug_port u_ila/probe0 [get_nets {state[*]}]
connect_debug_port u_ila/probe1 [get_nets {data[*]}]
```

