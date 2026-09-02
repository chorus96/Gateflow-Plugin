---
name: tb-best-practices
description: >
  Testbench verification best practices and patterns.
  This skill should be used when the user needs testbench architecture guidance,
  verification methodology, or wants to write professional-quality testbenches.
  Example requests: "testbench best practices", "how to structure TB", "verification patterns"
allowed-tools:
  - Grep
  - Glob
  - Read
  - Write
  - Edit
  - Bash
  - WebFetch
  - Task
---

# 테스트벤치 모범 사례

SystemVerilog 테스트벤치를 위한 전문 검증 패턴.

## 계층형 테스트벤치 아키텍처

```
┌─────────────────────────────────────────┐
│              Test Layer                 │  ← Test scenarios
├─────────────────────────────────────────┤
│           Environment Layer             │  ← Agent coordination
├──────────────┬──────────────────────────┤
│    Agent     │    Agent                 │  ← Protocol-specific
│  ┌────────┐  │  ┌────────┐              │
│  │ Driver │  │  │Monitor │              │
│  └────────┘  │  └────────┘              │
│  ┌────────┐  │  ┌────────┐              │
│  │Sequencer│ │  │Scoreboard│            │
│  └────────┘  │  └────────┘              │
├──────────────┴──────────────────────────┤
│           Interface Layer               │  ← Signal abstraction
├─────────────────────────────────────────┤
│               DUT                       │
└─────────────────────────────────────────┘
```

---

## 1. 테스트벤치 구조

### 기본 자가 검사 TB
```systemverilog
module tb_dut;
    // Clock and reset
    logic clk = 0;
    logic rst_n = 0;
    always #5 clk = ~clk;

    // DUT signals
    logic [7:0] data_in, data_out;
    logic valid_in, valid_out;

    // DUT instance
    dut u_dut (.*);

    // Test
    initial begin
        $dumpfile("dump.vcd");
        $dumpvars(0, tb_dut);

        // Reset
        rst_n = 0;
        repeat(5) @(posedge clk);
        rst_n = 1;

        // Stimulus + Check
        for (int i = 0; i < 100; i++) begin
            @(posedge clk);
            data_in <= $urandom();
            valid_in <= 1;

            @(posedge clk);
            valid_in <= 0;

            // Wait for output
            wait(valid_out);
            check_result(data_in, data_out);
        end

        $display("TEST PASSED");
        $finish;
    end

    // Checker
    function void check_result(input [7:0] expected, input [7:0] actual);
        assert(actual == expected)
            else $error("Mismatch: expected %0h, got %0h", expected, actual);
    endfunction
endmodule
```

---

## 2. 트랜잭션 클래스

### 모범 사례: 데이터와 타이밍 분리
```systemverilog
class Transaction;
    rand bit [31:0] addr;
    rand bit [31:0] data;
    rand bit [3:0]  burst_len;
    rand bit        write;

    // Constraints
    constraint c_aligned { addr[1:0] == 2'b00; }
    constraint c_burst   { burst_len inside {1, 4, 8, 16}; }

    // Copy
    function Transaction copy();
        Transaction t = new();
        t.addr = this.addr;
        t.data = this.data;
        t.burst_len = this.burst_len;
        t.write = this.write;
        return t;
    endfunction

    // Display
    function void display(string prefix = "");
        $display("%s addr=%08h data=%08h burst=%0d %s",
            prefix, addr, data, burst_len, write ? "WR" : "RD");
    endfunction
endclass
```

---

## 3. 드라이버

### 모범 사례: 신호가 아니라 인터페이스를 구동
```systemverilog
class Driver;
    virtual bus_if.master vif;
    mailbox #(Transaction) mbx;

    function new(virtual bus_if.master vif, mailbox #(Transaction) mbx);
        this.vif = vif;
        this.mbx = mbx;
    endfunction

    task run();
        Transaction t;
        forever begin
            mbx.get(t);
            drive(t);
        end
    endtask

    task drive(Transaction t);
        @(posedge vif.clk);
        vif.addr  <= t.addr;
        vif.data  <= t.data;
        vif.valid <= 1;

        @(posedge vif.clk);
        while (!vif.ready) @(posedge vif.clk);
        vif.valid <= 0;
    endtask
endclass
```

---

## 4. 모니터

### 모범 사례: 수동 관찰만
```systemverilog
class Monitor;
    virtual bus_if.monitor vif;
    mailbox #(Transaction) mbx;  // To scoreboard

    function new(virtual bus_if.monitor vif, mailbox #(Transaction) mbx);
        this.vif = vif;
        this.mbx = mbx;
    endfunction

    task run();
        Transaction t;
        forever begin
            @(posedge vif.clk);
            if (vif.valid && vif.ready) begin
                t = new();
                t.addr = vif.addr;
                t.data = vif.data;
                mbx.put(t);
            end
        end
    endtask
endclass
```

---

## 5. 스코어보드

### 모범 사례: 예상 vs 실제 비교
```systemverilog
class Scoreboard;
    mailbox #(Transaction) expected_mbx;
    mailbox #(Transaction) actual_mbx;
    int pass_count, fail_count;

    function new(mailbox #(Transaction) expected_mbx, mailbox #(Transaction) actual_mbx);
        this.expected_mbx = expected_mbx;
        this.actual_mbx = actual_mbx;
    endfunction

    task run();
        Transaction expected, actual;
        forever begin
            expected_mbx.get(expected);
            actual_mbx.get(actual);
            compare(expected, actual);
        end
    endtask

    function void compare(Transaction expected, Transaction actual);
        if (expected.data == actual.data) begin
            pass_count++;
        end else begin
            fail_count++;
            $error("MISMATCH: expected=%08h actual=%08h", expected.data, actual.data);
        end
    endfunction

    function void report();
        $display("=============================");
        $display("Scoreboard: PASS=%0d FAIL=%0d", pass_count, fail_count);
        $display("=============================");
    endfunction
endclass
```

---

## 6. 무작위화 모범 사례

### 제약 계층화
```systemverilog
class Transaction;
    rand bit [31:0] addr;
    rand bit [7:0] data;

    // Base constraint
    constraint c_valid_addr { addr < 32'h1000_0000; }

    // Can be overridden in extended class
    constraint c_mode { soft data > 0; }
endclass

class ErrorTransaction extends Transaction;
    // Override to inject errors
    constraint c_mode { data == 0; }
endclass
```

### 가중 분포
```systemverilog
class Packet;
    rand bit [1:0] pkt_type;

    constraint c_type_dist {
        pkt_type dist {
            0 := 60,  // 60% normal
            1 := 30,  // 30% error
            2 := 10   // 10% special
        };
    }
endclass
```

### Solve Before
```systemverilog
class Packet;
    rand bit [7:0] length;
    rand bit [7:0] payload[];

    constraint c_size {
        payload.size() == length;
        length inside {[1:64]};
        solve length before payload;
    }
endclass
```

---

## 7. 커버리지 모범 사례

### 기능 커버리지
```systemverilog
class Coverage;
    Transaction t;

    covergroup cg_trans @(posedge clk);
        // Coverpoints
        cp_addr: coverpoint t.addr[31:28] {
            bins low  = {[0:3]};
            bins mid  = {[4:11]};
            bins high = {[12:15]};
        }

        cp_write: coverpoint t.write;

        cp_burst: coverpoint t.burst_len {
            bins single = {1};
            bins burst4 = {4};
            bins burst8 = {8};
            bins burst16 = {16};
            illegal_bins bad = default;
        }

        // Cross coverage
        cross_addr_write: cross cp_addr, cp_write;
    endgroup

    function new();
        cg_trans = new();
    endfunction

    function void sample(Transaction t);
        this.t = t;
        cg_trans.sample();
    endfunction
endclass
```

### 커버리지 목표
| 커버리지 타입 | 목표 |
|---------------|--------|
| Line | >95% |
| Branch | >90% |
| FSM State | 100% |
| FSM Transition | >95% |
| Functional | >98% |

---

## 8. 스레딩 패턴

### Fork/Join
```systemverilog
// Parallel - wait for all
fork
    driver.run();
    monitor.run();
    scoreboard.run();
join

// Parallel - wait for any (with cleanup)
fork
    driver.run();
    monitor.run();
    timeout_check();
join_any
disable fork;

// Parallel - don't wait
fork
    background_task();
join_none
```

### 타임아웃 패턴
```systemverilog
task run_with_timeout(int cycles);
    fork
        begin
            run_test();
        end
        begin
            repeat(cycles) @(posedge clk);
            $error("TIMEOUT after %0d cycles", cycles);
            $finish;
        end
    join_any
    disable fork;
endtask
```

---

## 9. 인터페이스 모범 사례

### 파라미터화된 인터페이스
```systemverilog
interface axi_if #(
    parameter int ADDR_W = 32,
    parameter int DATA_W = 32
) (
    input logic clk,
    input logic rst_n
);
    logic [ADDR_W-1:0] awaddr;
    logic [DATA_W-1:0] wdata;
    logic              awvalid, awready;
    logic              wvalid, wready;
    logic [1:0]        bresp;
    logic              bvalid, bready;

    modport master (
        output awaddr, awvalid, wdata, wvalid, bready,
        input  awready, wready, bresp, bvalid
    );

    modport slave (
        input  awaddr, awvalid, wdata, wvalid, bready,
        output awready, wready, bresp, bvalid
    );

    modport monitor (
        input awaddr, awvalid, awready, wdata, wvalid, wready,
              bresp, bvalid, bready
    );

    // Clocking block for TB
    clocking cb @(posedge clk);
        default input #1 output #1;
        output awaddr, awvalid, wdata, wvalid, bready;
        input  awready, wready, bresp, bvalid;
    endclocking
endinterface
```

### 클래스의 가상 인터페이스
```systemverilog
class Agent;
    virtual axi_if vif;

    function new(virtual axi_if vif);
        this.vif = vif;
    endfunction
endclass
```

---

## 10. 테스트벤치의 어서션

### 프로토콜 검사
```systemverilog
// In interface or bind module
property p_valid_stable;
    @(posedge clk) disable iff (!rst_n)
    valid && !ready |=> valid && $stable(data);
endproperty
assert property (p_valid_stable) else $error("Valid dropped before ready");

property p_handshake;
    @(posedge clk) disable iff (!rst_n)
    valid |-> ##[1:100] ready;
endproperty
assert property (p_handshake) else $error("No ready within 100 cycles");
```

---

## 11. 테스트 구성

### 테스트 베이스 클래스
```systemverilog
virtual class BaseTest;
    Environment env;

    function new(Environment env);
        this.env = env;
    endfunction

    pure virtual task run();

    task pre_test();
        env.reset();
    endtask

    task post_test();
        env.report();
    endtask
endclass

class SmokeTest extends BaseTest;
    virtual task run();
        Transaction t = new();
        repeat(10) begin
            assert(t.randomize());
            env.driver_mbx.put(t);
        end
    endtask
endclass
```

---

## 12. 체크리스트

### 시작 전
- [ ] 검증 계획 정의
- [ ] 커버리지 목표 식별
- [ ] 테스트 시나리오 목록화
- [ ] TB 아키텍처 설계

### 개발 중
- [ ] 원시 신호가 아니라 트랜잭션 사용
- [ ] driver/monitor/scoreboard 분리
- [ ] 가상 인터페이스 사용
- [ ] 기능 커버리지 추가
- [ ] 프로토콜 어서션 포함

### 사인오프 전
- [ ] 모든 테스트 통과
- [ ] 커버리지 목표 달성
- [ ] 시뮬레이션에 X/Z 없음
- [ ] 엣지 케이스 테스트
- [ ] 오류 주입 테스트

---

## 레퍼런스

```
[SystemVerilog for Verification, 3rd ed. — Spear/Tumbush]
|url: https://picture.iczhiku.com/resource/eetop/wYIEDKFRorpoPvvV.pdf
|relevant: Ch 5 (OOP), Ch 6 (Rand), Ch 7 (Threads), Ch 8 (Advanced OOP), Ch 9 (Coverage), Ch 11 (Complete TB)
```

---

## Verilator v5용 UVM-Lite

Verilator v5는 UVM 2017 서브셋을 지원합니다. 동작: uvm_component, uvm_object, uvm_sequence_item, uvm_driver, uvm_monitor, uvm_scoreboard, uvm_env, uvm_test, uvm_tlm 포트, phase. 동작 안 함: factory override, $cast, RAL, 가상 인터페이스가 있는 uvm_config_db, 콜백.

가상 인터페이스는 config_db가 아니라 생성자/setter로 전달. factory가 아니라 직접 생성 사용. `-DUVM_NO_DPI --timing`으로 컴파일.

## Cocotb 대응물

| SV 패턴 | Cocotb 대응물 |
|---|---|
| Transaction class | Python dataclass |
| rand/constraint | random module + manual constraints |
| mailbox | collections.deque or asyncio.Queue |
| virtual interface | Direct dut handle |
| covergroup | cocotb-coverage decorators |
| fork/join | Combine(start_soon(...)) |
| fork/join_any | First(start_soon(...)) |
| assert property | async def checker coroutine |

## 커버리지 클로저 체크리스트

1. 전체 회귀 후 커버리지 보고서 추출
2. 분류: 도달 불가능한 코드 (근거와 함께 제외), 트리거되지 않은 FSM 전이 (지향 테스트 작성), 빈 크로스 빈 (제약 오버라이드), 낮은 히트 coverpoint (반복 증가 또는 가중 분포 추가)
3. 남은 각 빈틈에 대해 지향 테스트 작성
4. 병합된 커버리지로 재실행, 모든 목표 달성 검증
5. 모든 제외를 근거 주석과 함께 문서화

## 10가지 TB 안티패턴

1. **매직 넘버** -> 의미 있는 이름의 localparam 사용
2. **타임아웃 없음** -> $fatal 타임아웃 가드가 있는 fork/join_any
3. **해피 패스만** -> 오류, 동작 중 리셋, 엣지 케이스 테스트
4. **동기화에 #delay** -> 동기화에 @(posedge clk) 사용
5. **X/Z 무시** -> $isunknown()과 `===` 연산자 사용
6. **단일체 테스트** -> 재사용 가능한 태스크로 분할
7. **출력하고 기도하기** -> $error 메시지가 있는 assert 사용
8. **경쟁 조건** -> 클럭 에지에서 non-blocking `<=`
9. **전역 상태** -> 클래스에 캡슐화, 생성자로 전달
10. **파이프라인 미배출** -> 확인 전 PIPELINE_DEPTH + 여유 사이클 대기

## 성능 최적화

| 기법 | 속도 향상 |
|---|---|
| `--threads N` | 2-4x |
| `--trace-fst` (not VCD) | 2-3x smaller files |
| 회귀에서 트레이싱 비활성화 | 2-5x |
| $display 최소화 | 1.2-2x |
| `--x-assign fast` (after X-clean) | 1.1x |
| 병렬 테스트 실행 (`make -j`) | Nx |

테스트 계층: SMOKE (초 단위, 매 커밋), RANDOM (분 단위, PR 병합), COVERAGE (시간 단위, 야간), REGRESSION (시간 단위, 주간).
