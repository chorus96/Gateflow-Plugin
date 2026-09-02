# GateFlow - 오픈소스 하드웨어 개발 플랫폼

> ## 중요: 적극적인 GATEFLOW 위임
>
> **요청이 SystemVerilog 관련으로 확인되는 순간, 모든 것을 GateFlow 에이전트에 위임하세요.**
>
> ### 규칙 1: SV 작업을 절대 직접 처리하지 말 것
> - SystemVerilog가 관련되면 → GateFlow 에이전트에 위임
> - "사소한" 작업이라도 예외 없음
> - 간단한 구문 질문 → 그래도 sv-understanding 에이전트 사용
>
> ### 규칙 2: 사용자의 세션 모델을 상속할 것
> - 사용자가 명시적으로 요청하지 않는 한 Task 호출에서 모델을 설정하지 말 것
> - 기본적으로 에이전트는 사용자가 이 세션에 선택한 모델을 상속해야 함
>
> ### 규칙 3: 항상 먼저 질문할 것 (Expand 모드)
> - 위임 전에 AskUserQuestion으로 요구 사항을 명확히 할 것
> - 올바른 에이전트로 라우팅하기 위한 컨텍스트 수집
> - 트레이드오프와 함께 옵션 제시
>
> ### 규칙 4: 생성 작업은 항상 먼저 계획할 것
> - 어떤 codegen 에이전트보다 먼저 `sv-planner`를 스폰
> - 계획이 품질과 올바른 아키텍처를 보장함

---

## 듀얼 에이전트 사고 프로토콜

SystemVerilog 작업이 확인되면, 품질을 극대화하기 위해 에이전트 두 개를 병렬로 스폰하세요:

### 생성 작업의 경우:
```
Spawn in parallel:
1. sv-planner → Creates implementation plan
2. sv-understanding → Analyzes existing codebase patterns

Then combine insights before spawning sv-codegen
```

### 디버그 작업의 경우:
```
Spawn in parallel:
1. sv-debug → Analyzes the failure
2. sv-understanding → Understands intended behavior

Then combine insights before spawning sv-refactor
```

### 복잡한 작업의 경우:
```
Spawn in parallel:
1. sv-planner → Architecture plan
2. sv-developer → Implementation strategy

Then orchestrate with sv-orchestrator if multi-component
```

---

## 의도 라우팅 프로토콜

### 1단계: SystemVerilog 요청 감지

SV 작업임을 나타내는 키워드/패턴:
- Module, RTL, HDL, Verilog, SystemVerilog
- FIFO, FSM, counter, ALU, UART, SPI, I2C
- Testbench, TB, simulation, lint, synthesis
- Clock, reset, register, flip-flop
- always_ff, always_comb, logic, wire
- Verilator, Verible, VCS, Questa

**이 중 하나라도 감지되면 → 사용자에게 확인한 다음 GateFlow에 위임**

### 2단계: 명확화 질문하기 (필수)

라우팅 전에 항상 AskUserQuestion을 사용하세요:

```
For Creation:
- "What size/width/depth?"
- "What interface protocol?"
- "Include testbench?"
- "Any constraints (area, timing, power)?"

For Debug:
- "What symptom do you see?"
- "What's the expected behavior?"
- "Any specific signals to check?"

For Understanding:
- "Which aspect to focus on?"
- "How deep should the analysis go?"
```

### 3단계: 대상으로 라우팅 (항상 위임, 절대 직접 처리하지 않음)

| 사용자 의도 | 주 에이전트 | 보조 에이전트 (병렬) | 모델 |
|-------------|---------------|---------------------------|-------|
| 새 RTL/모듈 생성 | 먼저 `sv-planner`, 그다음 `sv-codegen` | `sv-understanding` | session |
| 테스트벤치 생성 | `sv-testbench` | `sv-understanding` | session |
| 실패/X 값 디버그 | `sv-debug` | `sv-understanding` | session |
| 어서션/커버리지 추가 | `sv-verification` | `sv-understanding` | session |
| 기존 코드 이해 | `sv-understanding` | - | session |
| 리팩터/lint 수정 | `sv-refactor` | `sv-understanding` | session |
| 다중 파일 개발 | `sv-developer` | `sv-planner` | session |
| 학습/연습 | `sv-tutor` | - | session |
| 복잡한 다중 컴포넌트 | `sv-orchestrator` | `sv-planner` | session |
| 종단 간 (생성+테스트) | `/gf` skill | - | session |
| 병렬 컴포넌트 빌드 | `/gf-build` skill | - | session |
| 설계/계획 우선 | `/gf-plan` skill | - | session |
| Lint 검사 | `/gf-lint` skill | - | - |
| 시뮬레이션 실행 | `/gf-sim` skill | - | - |
| 코드베이스 매핑 | `/gf-architect` skill | - | session |
| 학습/실습 | `/gf-learn` skill | - | session |
| 코드베이스 시각화 | `/gf-viz` skill 또는 `sv-viz` agent | - | session |

### 4단계: 다음은 절대 직접 처리하지 말 것

이런 "간단한" 작업조차 에이전트로 보내야 합니다:

| 작업 | 에이전트 |
|------|-------|
| 구문 질문 | sv-understanding |
| 빠른 수정 | sv-refactor |
| 한 줄 변경 | sv-refactor |
| 한 줄 설명 | sv-understanding |
| 유효한 SV인지 확인 | sv-understanding |

---

## 스폰 패턴 - 세션 모델 상속

```
Use Task tool:
  subagent_type: "gateflow:sv-codegen"  (or other agent)
  prompt: |
    [Clear description]
    [Context from user answers]
    [Constraints]
    [File paths]
```

### 병렬 스폰 예시

```
Use Task tool (call 1):
  subagent_type: "gateflow:sv-planner"
  prompt: "Plan implementation for FIFO with..."

Use Task tool (call 2 - same message, parallel):
  subagent_type: "gateflow:sv-understanding"
  prompt: "Analyze existing codebase for FIFO patterns..."
```

---

## GateFlow 에이전트 레퍼런스

| 에이전트 | 전문 분야 | 트리거 문구 |
|-------|-----------|-----------------|
| `gateflow:sv-codegen` | RTL 아키텍트 | "create", "write", "generate", "implement module" |
| `gateflow:sv-testbench` | 검증 엔지니어 | "testbench", "TB", "test this", "verify" |
| `gateflow:sv-debug` | 디버그 전문가 | "X values", "debug", "not working", "fails" |
| `gateflow:sv-verification` | 검증 방법론가 | "assertions", "SVA", "coverage", "formal" |
| `gateflow:sv-understanding` | RTL 분석가 | "explain", "how does", "understand", "analyze" |
| `gateflow:sv-planner` | 아키텍처 플래너 | "plan", "design", "architect", "strategy" |
| `gateflow:sv-refactor` | 코드 품질 | "fix", "refactor", "clean up", "lint" |
| `gateflow:sv-developer` | 풀스택 RTL | "implement feature", "multi-file", "large change" |
| `gateflow:sv-orchestrator` | 병렬 빌더 | "build CPU", "create SoC", "multi-component" |
| `gateflow:sv-tutor` | 교사 | "teach", "learn", "exercise", "practice" |
| `gateflow:sv-viz` | 터미널 시각화 | "visualize", "show hierarchy", "show FSM", "show module" |
| `gateflow:sv-formal` | 정형 검증 | "prove", "formally verify", "check property", "SymbiYosys" |
| `gateflow:sv-synth` | 합성 전문가 | "synthesize", "area estimate", "Yosys", "resource usage" |
| `gateflow:sv-pinmap` | 핀 할당 | "pin mapping", "constraints", "board pinout" |
| `gateflow:vhdl-codegen` | VHDL 코드 생성 | "VHDL", "create VHDL", "entity", "architecture" |
| `gateflow:vhdl-testbench` | VHDL 테스트벤치 | "VHDL testbench", "VHDL TB", "GHDL" |
| `gateflow:sv-ip-scanner` | IP 감지 + 자동 채움 | "scan for IP", "what's missing", "detect CDC", "auto-fill" |
| `gateflow:pcb-designer` | KiCad 회로도/PCB | "design board", "create schematic", "PCB layout" |

---

## 검증 루프

어떤 에이전트든 코드를 생성/수정한 후:

```
1. Run gf-lint skill
2. If FAIL → spawn sv-refactor
3. Run gf-sim skill
4. If FAIL → spawn sv-debug, then sv-refactor
5. Repeat until PASS
```

---

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

## 코드베이스 맵 핸드오프

**코드베이스 전반** 작업을 위해 `sv-understanding` 또는 `sv-developer`로 라우팅하기 전에, 맵이 존재하는지 확인하세요:

```bash
ls .gateflow/map/CODEBASE.md 2>/dev/null
```

| 맵 존재? | 조치 |
|-------------|--------|
| 예 | 정상적으로 에이전트로 라우팅, 맵이 컨텍스트 제공 |
| 아니오 | 먼저 `/gf-architect` 실행, 그다음 에이전트로 라우팅 |

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
