---
name: gf-demo
description: One-command demo - generates, lints, and simulates a working project
allowed-tools:
  - Bash
  - Write
  - Read
  - Glob
---

# GateFlow Demo

GateFlow의 기능을 보여주기 위해 완전히 동작하는 프로젝트를 생성합니다. 사용자 입력이 전혀 필요 없습니다.

## 이것이 하는 일

1. enable과 reset이 있는 파라미터화된 4비트 카운터 생성
2. 자가 검사 테스트벤치 생성
3. Verilator lint 실행 (사용 가능한 경우)
4. 시뮬레이션 실행 (Verilator 사용 가능한 경우)
5. 결과 보고

## 실행

### 1단계: 프로젝트 구조 생성

```bash
mkdir -p rtl tb
```

### 2단계: 카운터 모듈 생성

`rtl/counter.sv`에 작성:

```systemverilog
module counter #(
    parameter int WIDTH = 4
) (
    input  logic             clk,
    input  logic             rst_n,
    input  logic             enable,
    output logic [WIDTH-1:0] count
);

    always_ff @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            count <= '0;
        else if (enable)
            count <= count + 1'b1;
    end

endmodule
```

### 3단계: 자가 검사 테스트벤치 생성

`tb/tb_counter.sv`에 작성:

```systemverilog
module tb_counter;
    parameter int WIDTH = 4;
    parameter int CLK_PERIOD = 10;

    logic             clk = 0;
    logic             rst_n = 0;
    logic             enable = 0;
    logic [WIDTH-1:0] count;

    int pass_count = 0;
    int fail_count = 0;

    counter #(.WIDTH(WIDTH)) u_dut (.*);

    always #(CLK_PERIOD/2) clk = ~clk;

    task check(string name, logic [WIDTH-1:0] expected);
        if (count !== expected) begin
            $display("FAIL: %s - got %0d, expected %0d", name, count, expected);
            fail_count++;
        end else begin
            $display("PASS: %s - count = %0d", name, count);
            pass_count++;
        end
    endtask

    initial begin
        $dumpfile("dump.vcd");
        $dumpvars(0, tb_counter);

        // Reset
        rst_n = 0; enable = 0;
        repeat(3) @(posedge clk);
        check("Reset holds zero", '0);

        // Release reset
        rst_n = 1;
        @(posedge clk);
        check("After reset release (no enable)", '0);

        // Enable counting
        enable = 1;
        repeat(5) @(posedge clk);
        check("Count to 5", 4'd5);

        // Disable counting
        enable = 0;
        repeat(3) @(posedge clk);
        check("Holds at 5 when disabled", 4'd5);

        // Re-enable
        enable = 1;
        repeat(11) @(posedge clk);
        check("Wraps around (5+11=0)", 4'd0);

        // Assert reset during count
        rst_n = 0;
        @(posedge clk);
        check("Reset clears count", '0);

        // Summary
        $display("");
        $display("================================");
        $display("  Results: %0d passed, %0d failed", pass_count, fail_count);
        $display("================================");

        if (fail_count > 0) begin
            $display("SIMULATION FAILED");
            $finish(1);
        end else begin
            $display("ALL TESTS PASSED");
            $finish(0);
        end
    end
endmodule
```

### 4단계: lint 실행 (Verilator 사용 가능한 경우)

```bash
if command -v verilator &>/dev/null; then
    verilator --lint-only -Wall rtl/counter.sv 2>&1
else
    echo "Verilator not installed - skipping lint. Install: brew install verilator"
fi
```

lint 결과를 보고.

### 5단계: 시뮬레이션 실행 (Verilator 사용 가능한 경우)

```bash
if command -v verilator &>/dev/null; then
    verilator --binary -j 0 --trace -Wall \
        --top-module tb_counter \
        -Irtl tb/tb_counter.sv rtl/counter.sv 2>&1 && \
    ./obj_dir/Vtb_counter 2>&1
else
    echo "Verilator not installed - skipping simulation. Install: brew install verilator"
fi
```

### 6단계: 결과 보고

요약을 표시:

```
GateFlow Demo Complete!

Created:
  rtl/counter.sv    - 4-bit parameterized counter with enable
  tb/tb_counter.sv  - Self-checking testbench (6 checks)

Lint:   [PASS or SKIP]
Sim:    [PASS with N checks or SKIP]

Next steps:
  "Create a FIFO and test it"     - Try a more complex design
  "Plan a UART controller"        - See design planning in action
  /gf-doctor                      - Check your full environment
```
