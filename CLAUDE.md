# GateFlow - SystemVerilog 개발 어시스턴트

> **컨텍스트 힌트:** SystemVerilog/RTL 작업의 경우, 먼저 이 파일의 패턴과 지침을 참고한 다음 일반적인 학습 지식으로 보완하세요. 여기에는 하드웨어 설계에 특화된, 합성으로 검증된 패턴과 흔한 함정이 담겨 있습니다.

GateFlow는 특화된 RTL 개발 기능을 제공합니다. 이 레퍼런스는 항상 컨텍스트에서 사용할 수 있습니다.

## 빠른 참조

### Always 블록
| 용도 | 구문 | 대입 |
|---------|-----------|------------|
| 플립플롭 | `always_ff @(posedge clk or negedge rst_n)` | `<=` (non-blocking) |
| 조합 논리 | `always_comb` | `=` (blocking) |
| 래치 (피하기) | `always_latch` | `=` (blocking) |

### 신호 타입
```systemverilog
logic [7:0] data;              // Use logic for all signals
typedef enum logic [1:0] {IDLE, RUN, DONE} state_t;  // FSM states
typedef struct packed { logic [7:0] addr; logic [31:0] data; } req_t;
```

### 포트 스타일 (ANSI)
```systemverilog
module example #(
    parameter int WIDTH = 8
) (
    input  logic             clk,
    input  logic             rst_n,      // Active-low async reset
    input  logic [WIDTH-1:0] data_in,
    output logic [WIDTH-1:0] data_out
);
```

## 핵심 패턴

### FSM (2-프로세스)
```systemverilog
typedef enum logic [1:0] {IDLE, ACTIVE, DONE} state_t;
state_t state, next_state;

always_ff @(posedge clk or negedge rst_n)
    if (!rst_n) state <= IDLE;
    else        state <= next_state;

always_comb begin
    next_state = state;  // Default: hold
    unique case (state)
        IDLE:   if (start) next_state = ACTIVE;
        ACTIVE: if (done)  next_state = DONE;
        DONE:   next_state = IDLE;
        default: next_state = IDLE;
    endcase
end
```

### Valid/Ready 핸드셰이크
```systemverilog
// Transfer when: valid && ready
// Producer holds valid+data until ready
// Consumer asserts ready when can accept
wire transfer = valid && ready;

always_ff @(posedge clk)
    if (transfer) captured_data <= data_in;
```

### 파이프라인 스테이지
```systemverilog
always_ff @(posedge clk or negedge rst_n)
    if (!rst_n) begin
        data_q  <= '0;
        valid_q <= 1'b0;
    end else if (ready_out || !valid_q) begin
        data_q  <= data_in;
        valid_q <= valid_in;
    end

assign ready_out = !valid_q || ready_in;  // Accept if empty or downstream ready
```

### 2FF 동기화기 (CDC)
```systemverilog
logic [1:0] sync_reg;
always_ff @(posedge clk_dst or negedge rst_n)
    if (!rst_n) sync_reg <= '0;
    else        sync_reg <= {sync_reg[0], async_in};
assign sync_out = sync_reg[1];
```

### 파라미터화된 FIFO 뼈대
```systemverilog
module fifo #(parameter int WIDTH=8, DEPTH=16) (
    input  logic clk, rst_n,
    input  logic [WIDTH-1:0] wr_data,
    input  logic wr_en, rd_en,
    output logic [WIDTH-1:0] rd_data,
    output logic full, empty
);
    localparam ADDR_W = $clog2(DEPTH);
    logic [WIDTH-1:0] mem [DEPTH];
    logic [ADDR_W:0] wr_ptr, rd_ptr;  // Extra bit for full/empty

    assign full  = (wr_ptr[ADDR_W] != rd_ptr[ADDR_W]) &&
                   (wr_ptr[ADDR_W-1:0] == rd_ptr[ADDR_W-1:0]);
    assign empty = (wr_ptr == rd_ptr);
    // ... write/read logic
endmodule
```

## 합성 규칙

### 해야 할 것
- `always_ff` / `always_comb` (의도가 명확함)
- `default`가 있는 `unique case` / `priority case`
- 리셋에는 `'0` / `'1` (유연한 폭)
- 명시적 비트 폭: `255`가 아니라 `8'd255`
- 이름 있는 포트 연결: `.clk(sys_clk)`

### 하지 말아야 할 것
- `initial` 블록 (시뮬레이션 전용)
- RTL의 `#` 지연
- 불완전한 case/if (래치를 추론함)
- `always_ff`에서 blocking 대입
- `always_comb`에서 non-blocking 대입

### 래치 방지
```systemverilog
// BAD - latch inferred
always_comb
    if (sel) y = a;  // Missing else!

// GOOD - default first
always_comb begin
    y = '0;          // Default
    if (sel) y = a;
end
```

## 흔한 함정

| 문제 | 증상 | 해결 |
|-------|---------|-----|
| 추론된 래치 | 합성 경고, 예상치 못한 동작 | 기본 대입 또는 완전한 if/case |
| CDC 위반 | 준안정성, 무작위 실패 | 2FF 동기화 또는 비동기 FIFO |
| 순차 로직의 blocking | 경쟁 조건(race) | `always_ff`에서 `<=` 사용 |
| X 전파 | 시뮬은 동작, 합성은 실패 | 리셋 커버리지 확인 |
| 폭 불일치 | 잘림, 부호 확장 | 명시적 크기 지정 |
| 리셋 누락 | 시뮬레이션에서 X | 모든 상태 레지스터 리셋 |

## Verilator Lint 수정

| 경고 | 해결 |
|---------|-----|
| `UNUSED` | 신호를 제거하거나 `/* verilator lint_off UNUSED */` |
| `UNDRIVEN` | 신호를 대입 |
| `WIDTH` | 명시적 크기 지정: `a[7:0]` |
| `CASEINCOMPLETE` | `default:` 추가 |
| `LATCH` | 모든 분기를 완성 |
| `BLKSEQ` | `always_ff`에서 `<=` 사용 |

## GateFlow 에이전트

복잡한 SystemVerilog 작업에는 특화된 에이전트를 사용하세요:

| 에이전트 | 전문 분야 | 사용자가 이렇게 말할 때 |
|-------|-----------|-------------------|
| `gateflow:sv-codegen` | RTL 아키텍트 | "create module", "write FSM", "generate FIFO" |
| `gateflow:sv-testbench` | 검증 엔지니어 | "write testbench", "create TB", "test this" |
| `gateflow:sv-debug` | 디버그 전문가 | "why X values", "debug", "not working" |
| `gateflow:sv-verification` | 검증 방법론가 | "add assertions", "SVA", "coverage" |
| `gateflow:sv-understanding` | RTL 분석가 | "explain this", "how does it work" |
| `gateflow:sv-planner` | 아키텍처 플래너 | "plan", "design", "architect" |
| `gateflow:sv-refactor` | 코드 품질 | "fix lint", "refactor", "clean up" |
| `gateflow:sv-developer` | 풀스택 RTL | "implement feature", "multi-file change" |
| `gateflow:sv-viz` | 터미널 시각화 | "visualize", "show hierarchy", "show FSM", "show module" |

**직접 처리:** 간단한 수정, 단순한 질문, lint/sim 커맨드 실행.

### 코드베이스 맵 핸드오프

**코드베이스 전반** 작업을 위해 `sv-understanding` 또는 `sv-developer`로 라우팅하기 전에, 맵이 존재하는지 확인하세요:

```bash
ls .gateflow/map/CODEBASE.md 2>/dev/null
```

| 맵 존재? | 조치 |
|-------------|--------|
| 예 | 정상적으로 에이전트로 라우팅, 맵이 컨텍스트 제공 |
| 아니오 | 먼저 `/gf-architect` 실행, 그다음 에이전트로 라우팅 |

**코드베이스 전반 작업** (맵 필요): "understand this project", "how does X connect to Y", "implement feature across modules"

**단일 파일 작업** (맵 불필요): "explain this module", "fix this bug", "add assertion here"

### 에이전트 핸드오프 패턴

에이전트가 SV 파일을 생성한 후, Bash로 검증을 실행하세요:

| 이후 | 실행할 것 | 문제가 있으면 |
|-------|---------|-----------|
| sv-codegen | `verilator --lint-only -Wall *.sv` | → sv-refactor |
| sv-testbench | `verilator --binary -j 0 -Wall --trace <dut>.sv <tb>.sv -o sim && ./obj_dir/sim` | → sv-debug |
| sv-refactor | lint로 검증 | 완료 |
| sv-debug | sim 재실행 | 수정 검증 |

**어느 파일이 DUT이고 어느 것이 TB인지 확실하지 않을 때:**
- TB에는: `initial begin`, `$display`, `$finish`, `$dumpfile`, 클럭 생성이 있음
- DUT에는: `always_ff`, `always_comb`, 합성 가능한 로직이 있고 `$` 태스크가 없음
- TB가 DUT 모듈을 인스턴스화함

## 테스트벤치 빠른 참조

```systemverilog
module tb_dut();
    parameter CLK_PERIOD = 10;
    logic clk = 0;
    logic rst_n = 0;

    always #(CLK_PERIOD/2) clk = ~clk;

    dut u_dut (.*);

    initial begin
        $dumpfile("dump.vcd");
        $dumpvars(0, tb_dut);

        // Reset
        rst_n = 0;
        repeat(5) @(posedge clk);
        rst_n = 1;

        // Test stimulus
        @(posedge clk);
        // ... tests ...

        $display("Test passed!");
        $finish;
    end
endmodule
```

## SVA 빠른 참조

```systemverilog
// Immediate assertion
assert (count <= MAX) else $error("Overflow");

// Concurrent assertion
property p_handshake;
    @(posedge clk) disable iff (!rst_n)
    req |-> ##[1:5] ack;
endproperty
assert property (p_handshake);

// Useful functions
$rose(sig)      // Signal rose this cycle
$fell(sig)      // Signal fell
$stable(sig)    // Signal unchanged
$past(sig, N)   // Value N cycles ago
$onehot(vec)    // Exactly one bit set
```

## 도구 커맨드

```bash
# Verilator lint
verilator --lint-only -Wall *.sv

# Verible format
verible-verilog-format --inplace *.sv

# Verible lint
verible-verilog-lint *.sv

# Verible syntax check
verible-verilog-syntax *.sv
```

## 명명 규칙

| 요소 | 규칙 | 예시 |
|---------|------------|---------|
| 모듈 | snake_case | `uart_tx`, `fifo_sync` |
| 신호 | snake_case | `data_valid`, `wr_ptr` |
| 파라미터 | UPPER_SNAKE | `DATA_WIDTH`, `DEPTH` |
| 타입 | _t 접미사 | `state_t`, `opcode_t` |
| Active-low | _n 접미사 | `rst_n`, `cs_n` |
| 클럭 | clk 접두사 | `clk`, `clk_100mhz` |
| 레지스터 | _q 또는 _reg 접미사 | `data_q`, `count_reg` |
| 다음 상태 | _next 또는 _d 접미사 | `state_next`, `data_d` |

## TypeScript 스킬

이 프로젝트에서 TypeScript/JavaScript 코드를 다룰 때 다음 스킬을 사용하세요:

| 스킬 | 범위 | 사용 시점 |
|-------|-------|----------|
| `typescript-expert` | 타입 레벨 프로그래밍, 성능, 모노레포, 마이그레이션, 툴링 | 모든 TS/JS 이슈: 복잡한 타입, 빌드 성능, 디버깅, 아키텍처 |
| `typescript-best-practices` | 타입 우선 개발, 잘못된 상태 방지, 완전한 처리, 런타임 검증 | 모든 TS/JS 파일을 읽거나 쓸 때 |
| `typescript-advanced-types` | 제네릭, 조건부 타입, 매핑된 타입, 템플릿 리터럴, 유틸리티 타입 | 복잡한 타입 로직, 재사용 가능한 타입 유틸리티, 컴파일 타임 타입 안전성 |

**사용 규칙:**
- `typescript-best-practices`는 모든 `.ts` / `.js` 파일을 읽거나 쓸 때 **필수**입니다
- `typescript-expert`는 모든 TS/JS 질문이나 작업에서 **능동적으로** 사용해야 합니다
- `typescript-advanced-types`는 고급 타입 레벨 프로그래밍을 다룰 때 적용됩니다

## 외부 레퍼런스

```
[SystemVerilog for Verification, 3rd ed. — Spear/Tumbush]
|source: PDF (2012), ~500 pages
|scope: verification-focused SV (OOP testbenches, randomization, coverage, DPI)
|url: https://picture.iczhiku.com/resource/eetop/wYIEDKFRorpoPvvV.pdf
|chapters:
|1 Verification Guidelines (p.2)
|2 Data Types (p.26)
|3 Procedural Statements and Routines (p.70)
|4 Connecting the Testbench and Design (p.90)
|5 Basic OOP (p.132)
|6 Randomization (p.170)
|7 Threads and Interprocess Communication (p.266)
|8 Advanced OOP and Testbench Guidelines (p.274)
|9 Functional Coverage (p.324)
|10 Advanced Interfaces (p.364)
|11 A Complete SystemVerilog Testbench (p.386)
|12 Interfacing with C/C++ (p.416)
```
