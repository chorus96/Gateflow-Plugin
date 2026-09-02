---
name: gf-cocotb
description: >
  Python-based testbench generation using Cocotb. Alternative to
  SystemVerilog testbenches for Python-native hardware engineers.
  Example: "create a cocotb test for the FIFO", "write Python testbench"
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

# GF-Cocotb -- Python 테스트벤치 생성

SystemVerilog TB의 대안으로 Cocotb 테스트벤치를 생성합니다.

## 도구 감지

```bash
python3 -c "import cocotb" 2>/dev/null
```

찾을 수 없으면, `pip install cocotb`와 함께 GATEFLOW-RESULT ERROR를 반환.

## 테스트 템플릿

```python
import cocotb
from cocotb.clock import Clock
from cocotb.triggers import RisingEdge

@cocotb.test()
async def test_reset(dut):
    clock = Clock(dut.clk, 10, units="ns")
    cocotb.start_soon(clock.start())
    dut.rst_n.value = 0
    for _ in range(3):
        await RisingEdge(dut.clk)
    dut.rst_n.value = 1
    await RisingEdge(dut.clk)
    assert dut.count.value == 0
```

## 사용 시점

| Cocotb | SV 테스트벤치 |
|--------|-------------|
| Python (낮은 진입 장벽) | SystemVerilog |
| 복잡한 자극 (Python 라이브러리) | 프로토콜 + 커버리지 |
| 느림 (공동 시뮬레이션) | 빠름 (네이티브) |

## Makefile 템플릿

```makefile
SIM ?= icarus
TOPLEVEL_LANG ?= verilog
VERILOG_SOURCES += $(PWD)/rtl/$(DUT).sv
TOPLEVEL = $(DUT)
MODULE = test_$(DUT)
COCOTB_HDL_TIMEUNIT = 1ns
COCOTB_HDL_TIMEPRECISION = 1ps

ifeq ($(SIM),verilator)
    EXTRA_ARGS += --trace --trace-structs
endif

SIM_BUILD = sim_build/$(SIM)
include $(shell cocotb-config --makefiles)/Makefile.sim
```

## 테스트 템플릿

### FIFO 테스트
```python
import cocotb
from cocotb.clock import Clock
from cocotb.triggers import RisingEdge, ClockCycles
import random

async def reset_fifo(dut):
    dut.rst.value = 1
    dut.wr_en.value = 0
    dut.rd_en.value = 0
    await ClockCycles(dut.clk, 5)
    dut.rst.value = 0
    await RisingEdge(dut.clk)

@cocotb.test()
async def test_fifo_write_read(dut):
    clock = Clock(dut.clk, 10, units="ns")
    cocotb.start_soon(clock.start())
    await reset_fifo(dut)

    assert dut.empty.value == 1
    written = []
    for i in range(16):
        data = random.randint(0, 255)
        written.append(data)
        dut.din.value = data
        dut.wr_en.value = 1
        await RisingEdge(dut.clk)
    dut.wr_en.value = 0
    await RisingEdge(dut.clk)
    assert dut.full.value == 1

    read = []
    for i in range(16):
        dut.rd_en.value = 1
        await RisingEdge(dut.clk)
        read.append(int(dut.dout.value))
    dut.rd_en.value = 0
    assert read == written
```

### Valid/Ready 핸드셰이크 테스트
```python
@cocotb.test()
async def test_handshake(dut):
    clock = Clock(dut.clk, 10, units="ns")
    cocotb.start_soon(clock.start())
    # ... reset ...

    async def driver():
        for data in range(20):
            dut.s_data.value = data
            dut.s_valid.value = 1
            while True:
                await RisingEdge(dut.clk)
                if dut.s_ready.value == 1:
                    break
        dut.s_valid.value = 0

    async def backpressure():
        while True:
            dut.m_ready.value = random.randint(0, 1)
            await ClockCycles(dut.clk, random.randint(1, 4))

    cocotb.start_soon(driver())
    cocotb.start_soon(backpressure())
    await ClockCycles(dut.clk, 200)
```

### FSM 테스트
```python
from enum import IntEnum

class State(IntEnum):
    IDLE = 0
    ACTIVE = 1
    DONE = 2

@cocotb.test()
async def test_fsm(dut):
    clock = Clock(dut.clk, 10, units="ns")
    cocotb.start_soon(clock.start())
    # reset...
    assert int(dut.state.value) == State.IDLE
    dut.start.value = 1
    await RisingEdge(dut.clk)
    dut.start.value = 0
    await RisingEdge(dut.clk)
    assert int(dut.state.value) == State.ACTIVE
```

## Python 러너 (pytest)

```python
import os
from pathlib import Path
from cocotb_tools.runner import get_runner

def test_module():
    sim = os.getenv("SIM", "icarus")
    runner = get_runner(sim)
    runner.build(
        sources=[Path("rtl/my_module.sv")],
        hdl_toplevel="my_module",
    )
    runner.test(
        hdl_toplevel="my_module",
        test_module="test_my_module",
    )
```

## cocotbext 프로토콜 라이브러리

| 패키지 | 프로토콜 | 설치 |
|---|---|---|
| `cocotbext-axi` | AXI4, AXI-Lite, AXI-Stream | `pip install cocotbext-axi` |
| `cocotbext-wishbone` | Wishbone B4 | `pip install cocotbext-wishbone` |
| `cocotbext-spi` | SPI | `pip install cocotbext-spi` |
| `cocotbext-uart` | UART | `pip install cocotbext-uart` |
| `cocotbext-eth` | Ethernet | `pip install cocotbext-eth` |
| `cocotbext-pcie` | PCIe | `pip install cocotbext-pcie` |

### AXI-Lite 예시
```python
from cocotbext.axi import AxiLiteBus, AxiLiteMaster

axil = AxiLiteMaster(AxiLiteBus.from_prefix(dut, "s_axi"), dut.clk, dut.rst)
await axil.write(0x0000, b'\x42\x00\x00\x00')
data = await axil.read(0x0000, 4)
```

## 결정 매트릭스: Cocotb vs SV TB

| 요인 | Cocotb 사용 | SV/UVM 사용 |
|---|---|---|
| 팀이 Python을 앎 | 예 | - |
| FPGA 프로젝트 | 예 | - |
| UVM 인프라가 있는 대형 ASIC | - | 예 |
| 오픈소스 CI 필요 | 예 | - |
| 표준 프로토콜 (AXI, SPI) | 예 (cocotbext) | - |
| 깊은 제약 무작위 | - | 예 |
| 빠른 반복 | 예 | - |

## GATEFLOW-RESULT 통합

```
---GATEFLOW-RESULT---
STATUS: PASS | FAIL | ERROR
ERRORS: <count>
WARNINGS: <count>
FILES: <test files>
DETAILS: <summary>
---END-GATEFLOW-RESULT---
```
