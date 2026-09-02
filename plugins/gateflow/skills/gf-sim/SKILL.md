---
name: gf-sim
description: >
  SystemVerilog simulator with structured output for orchestration.
  Auto-detects DUT vs testbench, compiles with Verilator, runs simulation,
  and returns a parseable result block for /gf orchestration.
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
---

# GF Sim 스킬

구조화된 출력으로 SystemVerilog 시뮬레이션을 컴파일하고 실행합니다.

## 도구 감지

시뮬레이션을 실행하기 전에 Verilator가 사용 가능한지 확인:
```bash
which verilator
```

Verilator를 사용할 수 없으면, 즉시 반환:
```
---GATEFLOW-RESULT---
STATUS: ERROR
ERRORS: 0
WARNINGS: 0
FILES: []
DETAILS: Simulation requires Verilator. Install it to enable simulation.
  macOS: brew install verilator
  Linux: sudo apt install verilator
---END-GATEFLOW-RESULT---
```

Verilator 없이 시뮬레이션을 시도하지 말 것. ERROR 상태를 반환하고 오케스트레이터가 처리하게 함.

---

## 지침

### 1. 파일 식별

**args에 파일이 지정되면:**
제공된 경로를 사용. 첫 파일은 보통 테스트벤치.

**파일이 지정되지 않으면:**
SV 파일을 스캔하여 자동 감지:

```bash
ls *.sv rtl/*.sv tb/*.sv 2>/dev/null
```

### 2. 파일 분류: DUT vs 테스트벤치

**테스트벤치 지표** (다음 중 하나 보유):
- `initial begin`
- `$display`, `$monitor`
- `$finish`, `$fatal`
- `$dumpfile`, `$dumpvars`
- 클럭 생성: `always #N clk = ~clk`
- `tb/` 디렉터리의 파일 또는 `*_tb.sv`, `tb_*.sv` 이름

**DUT 지표** (다음 중 하나 보유):
- `always_ff`, `always_comb`
- 합성 가능한 구문만
- `$` 시스템 태스크 없음 (어서션 제외)
- `rtl/` 디렉터리의 파일

**빠른 분류:**
```bash
# Files with testbench markers
grep -l '\$display\|\$finish\|initial begin' *.sv 2>/dev/null

# Files with DUT markers
grep -l 'always_ff\|always_comb' *.sv 2>/dev/null
```

### 3. Verilator로 컴파일

```bash
verilator --binary -j 0 -Wall --trace <dut-files> <testbench> -o sim
```

**참고:**
- DUT 파일을 먼저, 테스트벤치를 마지막에 나열
- `--trace`는 VCD 파형 생성을 활성화
- `-o sim`은 출력 실행 파일 이름을 지정

**여러 top 모듈이 감지되면:**
```bash
verilator --binary -j 0 -Wall --trace --top-module <tb_name> <files> -o sim
```

### 4. 시뮬레이션 실행

```bash
./obj_dir/sim
```

또는 다르게 이름이 붙었으면:
```bash
./obj_dir/V<top_module>
```

### 5. 결과 파싱

**출력에서 확인:**
- `PASS`, `SUCCESS`, `All tests passed` -> PASS
- `FAIL`, `ERROR`, `MISMATCH`, `ASSERT` -> FAIL
- `$fatal` 또는 0이 아닌 종료 코드 -> FAIL
- 오류 없이 `$finish` 도달 -> PASS

**종료 코드 확인:**
```bash
./obj_dir/sim
echo "Exit code: $?"
```
- 종료 0: 성공
- 0이 아님: 실패

### 6. 구조화된 결과 반환

**항상 이 정확한 블록 형식으로 응답을 끝내세요:**

```
---GATEFLOW-RESULT---
STATUS: PASS|FAIL|ERROR
ERRORS: <count>
WARNINGS: <count>
FILES: <comma-separated list>
DETAILS: <one-line summary>
---END-GATEFLOW-RESULT---
```

**상태 정의:**
- `PASS`: 시뮬레이션 완료, 테스트 통과
- `FAIL`: 시뮬레이션 실패 (컴파일 오류, 어서션 실패, 테스트 실패)
- `ERROR`: 시뮬레이션을 실행할 수 없음 (파일 누락, 설정 오류)

### 7. 예시: 성공한 실행

```
## File Classification

| File | Type | Reason |
|------|------|--------|
| rtl/fifo.sv | DUT | has always_ff, no $display |
| tb/tb_fifo.sv | TB | has $display, $finish, initial |

## Compilation

$ verilator --binary -j 0 -Wall --trace rtl/fifo.sv tb/tb_fifo.sv -o sim

(compilation output...)

## Simulation

$ ./obj_dir/sim

Test 1: Write single item... PASS
Test 2: Fill FIFO... PASS
Test 3: Overflow check... PASS
All tests passed!

---GATEFLOW-RESULT---
STATUS: PASS
ERRORS: 0
WARNINGS: 0
FILES: rtl/fifo.sv,tb/tb_fifo.sv
DETAILS: All 3 tests passed
---END-GATEFLOW-RESULT---
```

### 8. 예시: 실패한 실행

```
## Simulation

$ ./obj_dir/sim

Test 1: Write single item... PASS
Test 2: Read back... FAIL
  Expected: 0xAB
  Got: 0x00
$fatal called at tb_fifo.sv:87

---GATEFLOW-RESULT---
STATUS: FAIL
ERRORS: 1
WARNINGS: 0
FILES: rtl/fifo.sv,tb/tb_fifo.sv
DETAILS: Test 2 failed - read data mismatch at line 87
---END-GATEFLOW-RESULT---
```

### 9. 예시: 컴파일 오류

```
$ verilator --binary -j 0 -Wall rtl/fifo.sv tb/tb_fifo.sv -o sim

%Error: rtl/fifo.sv:45: Cannot find: fifo_mem

---GATEFLOW-RESULT---
STATUS: FAIL
ERRORS: 1
WARNINGS: 0
FILES: rtl/fifo.sv,tb/tb_fifo.sv
DETAILS: Compile error - undefined reference to fifo_mem
---END-GATEFLOW-RESULT---
```

## 흔한 문제와 해결책

| 문제 | 증상 | 해결책 |
|-------|---------|----------|
| 다중 top | "Multiple top modules" | `--top-module <name>` 추가 |
| 모듈 누락 | "Cannot find: X" | X를 정의하는 파일 포함 |
| X 값 | 출력에 X 표시 | 리셋 커버리지 확인 |
| 타임아웃 | 시뮬레이션 행 | 타임아웃 추가 또는 FSM 수정 |
| $finish 없음 | 영원히 실행 | TB가 $finish를 호출하는지 확인 |

## Verilator v5 성능 옵션

### 멀티스레드 시뮬레이션
```bash
verilator --binary --threads N -Wall --trace <files> -o sim
```
최상의 성능을 위해 `numactl`로 물리 코어에 고정.

### 트레이스 형식
| 형식 | 플래그 | 크기 | 뷰어 |
|---|---|---|---|
| VCD | `--trace` | 큼 | 범용 |
| FST | `--trace-fst` | 작음 | GTKWave, Surfer |

큰 설계에는 `--trace-fst`를 사용. FST 쓰기를 오프로드하려면 `--trace-threads 2` 추가.

### 어서션 (SVA)
```bash
verilator --binary --assert <files>     # DEFAULT in v5.038+
verilator --binary --no-assert <files>  # Disable for performance
```
1사이클 동시 assert/cover, `$past`, `$stable`, `$rose`, `$fell`을 지원.
멀티 사이클 시퀀스(SERE)는 지원하지 않음.

### 코드 커버리지
```bash
verilator --binary --coverage <files>        # All coverage
verilator --binary --coverage-line <files>   # Line only
verilator --binary --coverage-toggle <files> # Toggle only
```

### 최대 성능
```bash
verilator --binary -O3 --x-assign fast --x-initial fast --no-assert --threads N <files>
```

## Verilator SV 지원

| 구문 | 지원 |
|---|---|
| `always_comb`/`always_ff` | 완전 |
| Interfaces와 modports | 완전 |
| Packages, structs, enums | 완전 |
| Generate | 완전 |
| DPI (C/C++ import/export) | 완전 |
| Classes | 부분 |
| 제약된 무작위화 | 부분 |
| SVA (1사이클) | 완전 |
| SVA (멀티 사이클) | 미지원 |

## 시뮬레이션 타임아웃

시뮬레이션 행 방지:
```bash
timeout 60 ./obj_dir/sim
```

또는 테스트벤치에서:
```systemverilog
initial begin
    #1000000;
    $display("TIMEOUT");
    $finish;
end
```

## /gf 오케스트레이터의 사용

`/gf` 스킬은 이 스킬을 내부적으로 사용하고 결과 블록을 파싱합니다:

```
Parse ---GATEFLOW-RESULT--- block:
- STATUS: PASS -> report success, done
- STATUS: FAIL -> spawn sv-debug agent with failure context
- STATUS: ERROR -> report setup issue to user
```
