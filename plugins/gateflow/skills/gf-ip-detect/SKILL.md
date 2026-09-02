---
name: gf-ip-detect
description: >
  Auto-detect IP blocks within FPGA and hardware codebases. Scans for
  module instantiations, identifies missing implementations, matches
  standard IP patterns, and dispatches agents to fill gaps.
  Example: "scan for missing IP blocks", "detect what needs implementing",
  "find unimplemented modules", "auto-fill IP blocks"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

# GF-IP-Detect — IP 블록 자동 감지 및 자동 채움

하드웨어 코드베이스를 스캔해 IP 블록을 찾고, 빈틈을 식별하며,
누락 부분을 구현하도록 에이전트를 파견합니다.

## 감지하는 것

### 1. 누락된 모듈 구현
인스턴스화되었으나 프로젝트에 정의되지 않은 모듈:
```bash
# Find all module instantiations
grep -rn "^\s*\w\+\s\+\w\+\s*(" rtl/ tb/ --include="*.sv" --include="*.v" --include="*.vhd"

# Find all module definitions
grep -rn "^\s*module\s\+\w\+" rtl/ --include="*.sv" --include="*.v"

# Diff = missing implementations
```

### 2. 스텁 모듈 (비어 있거나 TODO)
정의되었으나 실제 구현이 없는 모듈:
```bash
# Find modules with TODO/FIXME/stub markers
grep -rn "TODO\|FIXME\|STUB\|NOT IMPLEMENTED" rtl/ --include="*.sv"

# Find modules with empty bodies (just endmodule after ports)
```

### 3. 표준 IP 패턴 매칭
검증된 IP 블록을 쓸 수 있는 흔한 하드웨어 패턴을 감지:

| 감지된 패턴 | 일치하는 IP 블록 | 신뢰도 |
|-----------------|-------------------|------------|
| full/empty가 있는 FIFO형 read/write | `fifo_sync` 또는 `fifo_async` | High |
| 2개 이상의 플립플롭 체인 (동기화기) | `cdc_2ff` | High |
| 클럭 간 Req/ack 핸드셰이크 | `cdc_handshake` | High |
| baud가 있는 UART형 시프트 레지스터 | `uart` | Medium |
| SCLK/MOSI/MISO/CS_N의 SPI형 | `spi_master` | High |
| addr/data가 있는 valid/ready의 AXI형 | `axi4lite_slave` | Medium |
| 디바운스 로직이 있는 카운터 | `debouncer` | Medium |

### 4. 인터페이스 빈틈
톱 모듈에 선언되었으나 어떤 구현에도 연결되지 않은 포트:
```bash
# Find top module ports
# Check which ports connect to instantiated submodules
# Unconnected ports = potential missing IP
```

### 5. 벤더 IP 플레이스홀더
오픈소스 대안이 있을 수 있는 벤더 특화 IP의 인스턴스화를 감지:
```bash
# Xilinx primitives
grep -rn "IBUF\|OBUF\|BUFG\|MMCME2\|PLLE2\|BRAM" rtl/ --include="*.sv"

# Lattice primitives
grep -rn "SB_IO\|SB_GB\|SB_PLL\|SB_RAM" rtl/ --include="*.sv"
```

---

## 감지 워크플로

### 1단계: 코드베이스 스캔

```
Read all .sv/.v/.vhd files in project
        |
Extract: module definitions (name, ports, parameters)
        |
Extract: module instantiations (what's used)
        |
Extract: signal patterns (FIFO, CDC, protocol)
        |
Build dependency graph
```

### 2단계: 빈틈 식별

인스턴스화된 각 모듈에 대해:
1. 프로젝트에 정의되어 있는가? → 없으면 **MISSING**
2. GateFlow 라이브러리의 알려진 IP 블록인가? → `/gf-ip add` 제안
3. 벤더 프리미티브인가? → 리뷰 대상으로 표시
4. 정의되었으나 비어 있음/스텁인가? → **NEEDS IMPLEMENTATION**

각 신호 패턴에 대해:
1. 표준 IP 패턴과 일치하는가? → 대체 제안
2. 구현이 임시적인가? → 검증된 IP 블록 제안
3. 동기화기 없는 CDC 크로싱이 있는가? → **CRITICAL: cdc_2ff 제안**

### 3단계: 보고

```
---GATEFLOW-RESULT---
STATUS: PASS | NEEDS_ACTION
SCAN_RESULTS:
  total_modules: 15
  defined: 12
  missing: 2
  stubs: 1
  
MISSING_MODULES:
  - name: fifo_controller
    instantiated_in: rtl/top.sv:42
    ports: [clk, rst_n, wr_en, wr_data, rd_en, rd_data, full, empty]
    suggested_ip: fifo_sync (92% match)
    
  - name: spi_peripheral
    instantiated_in: rtl/top.sv:67
    ports: [clk, rst_n, sclk, mosi, miso, cs_n, tx_data, rx_data]
    suggested_ip: spi_master (88% match)

STUBS:
  - name: uart_wrapper
    file: rtl/uart_wrapper.sv:1
    status: "Module defined but body is empty (TODO marker at line 15)"
    suggested_action: "Implement using uart IP block as base"

PATTERN_MATCHES:
  - pattern: "Ad-hoc 2FF synchronizer at rtl/sync.sv:10"
    suggestion: "Replace with verified cdc_2ff IP block"
    
  - pattern: "CDC crossing without synchronizer at rtl/top.sv:55"
    severity: CRITICAL
    suggestion: "Add cdc_2ff between clk_a and clk_b domains"

IP_OPPORTUNITIES:
  - "3 modules could use GateFlow IP blocks (fifo_sync, spi_master, cdc_2ff)"
  - "1 stub module needs implementation (uart_wrapper)"
  - "1 critical CDC issue detected"
---END-GATEFLOW-RESULT---
```

### 4단계: 자동 채움 (사용자 승인)

결과를 제시하고 질문:
```
Found 2 missing modules and 1 stub:

1. fifo_controller → 92% match with fifo_sync IP block
   Action: /gf-ip add fifo_sync and rename to fifo_controller? [Y/n]

2. spi_peripheral → 88% match with spi_master IP block
   Action: /gf-ip add spi_master and adapt ports? [Y/n]

3. uart_wrapper → Empty stub, matches uart IP pattern
   Action: Generate implementation using uart IP as base? [Y/n]

4. CRITICAL: CDC crossing at top.sv:55 without synchronizer
   Action: Insert cdc_2ff synchronizer? [Y/n]
```

---

## 자동 채움 파견

사용자가 자동 채움을 승인하면, 적절한 에이전트를 파견:

### IP 블록 일치의 경우
```
Use Task tool:
  subagent_type: "gateflow:sv-codegen"
  prompt: |
    Implement module <name> based on the <ip_block> IP block.
    Adapt ports to match the instantiation in <file>:<line>.
    Original ports: <detected_ports>
    IP block ports: <ip_ports>
    Generate the module, then create a testbench.
```

### 스텁의 경우
```
Use Task tool:
  subagent_type: "gateflow:sv-codegen"
  prompt: |
    Implement the stub module at <file>.
    The module signature is already defined:
    <module_signature>
    
    Based on the port names and context, this appears to be a <type>.
    Implement full functionality, following the existing codebase patterns.
```

### CDC 문제의 경우
```
Use Task tool:
  subagent_type: "gateflow:sv-refactor"
  prompt: |
    Add CDC synchronization at <file>:<line>.
    Signal <signal> crosses from <src_clk> to <dst_clk> domain
    without synchronization.
    
    Insert a cdc_2ff synchronizer (from GateFlow IP library).
    Ensure the synchronizer is properly reset.
```

---

## 패턴 감지 규칙

### FIFO 감지
```
Confidence: HIGH if module has:
- wr_en/write + rd_en/read signals
- full + empty signals  
- data_in/wr_data + data_out/rd_data signals
- Single clock → fifo_sync
- Dual clock → fifo_async
```

### CDC 감지
```
Confidence: HIGH if:
- Signal assigned in always_ff @(posedge clk_a)
- Signal read in always_ff @(posedge clk_b) where clk_a != clk_b
- No synchronizer between domains

Confidence: MEDIUM if:
- 2+ flip-flop chain detected but not using standard sync pattern
```

### 프로토콜 감지
```
SPI: sclk + mosi + miso + cs_n (any naming variant)
UART: tx/rx + baud-related parameter
I2C: scl + sda (bidirectional)
AXI: *valid + *ready + *addr + *data patterns
```

### 벤더 IP 감지
```
Xilinx: IBUF, OBUF, BUFG, MMCME2, PLLE2, BRAM_TDP, DSP48E1
Lattice: SB_IO, SB_GB, SB_PLL40, SB_SPRAM, SB_RAM
Gowin: IBUF, OBUF, rPLL, SDPB, pROM
Intel: altpll, altsyncram, altddio
```

---

## /gf 오케스트레이터와의 통합

`/gf` 오케스트레이터가 IP 감지를 호출할 수 있습니다:
- 새 기능 구현 전에 (무엇이 존재하는지 확인)
- 코드베이스 매핑 후 (IP 분석으로 보강)
- 사용자가 "what's missing" 또는 "scan for gaps"라고 말할 때

## /gf-architect와의 통합

코드베이스 매핑과 결합:
1. `/gf-architect`가 모듈 계층 구조를 매핑
2. `/gf-ip-detect`가 맵 위에 IP 분석을 오버레이
3. 결과: 계층 구조 + IP 기회 + CDC 문제

---

## 커맨드

- "Scan for missing IP blocks" → 전체 감지 + 보고
- "Auto-fill missing modules" → 감지 + 에이전트 파견
- "What IP blocks does my project need?" → 감지 + 제안
- "Find CDC issues" → CDC 크로싱 집중 분석
- "Replace ad-hoc code with verified IP" → 패턴 매칭 + 교체

---

## 확장 벤더 IP 감지

기존 목록 외에 추가로 감지할 프리미티브:
- **Xilinx UltraScale+:** URAM288, DSP48E2, BUFGCE, MMCME4_ADV, GTHE4_CHANNEL, STARTUPE3
- **Intel Agilex:** IOPLL, RAM20K, M20K, MLAB, tennm_ph2_iopll
- **Lattice ECP5:** EHXPLLL, DP16KD, TRELLIS_FF, DCUA, EXTREFB
- **Gowin (확장):** rPLL, PLLVR, SDPB, DPB, pROM, EMCU, DHCEN
- **Microchip PolarFire:** LSRAM, uSRAM, MACC, CCC, SERDES_IF
- **Efinix:** EFX_PLL, EFX_RAM_5K, EFX_DPRAM_5K, EFX_GBUFCE

## 오탐 감소

| 패턴 | 오탐인 경우 | 조치 |
|---|---|---|
| 2FF 체인 | 시프트 레지스터 내부 (3단계 이상) | 건너뜀 |
| 2FF 체인 | 두 FF가 같은 클럭 | 건너뜀 |
| FIFO 신호 | `*fifo*` 이름의 모듈 내부 | 건너뜀 |
| valid/ready | 이미 IP 블록을 쓰는 AXI 버스 | 건너뜀 |
| SPI 신호 | 테스트벤치 파일 내부 (tb_*, *_tb.sv) | 건너뜀 |
| 벤더 프리미티브 | 주석 또는 문자열 리터럴 안 | 건너뜀 |

## 심각도 점수

| 심각도 | 기준 | 예시 |
|---|---|---|
| CRITICAL | 데이터 손상, 준안정성 | 동기화기 없는 CDC |
| HIGH | 누락 모듈, 빈 스텁 | 인스턴스화되었으나 미정의 |
| MEDIUM | 임시 재구현 | 손수 만든 FIFO, 수동 2FF |
| LOW | 스타일, 사소한 최적화 | OSS 대안이 있는 벤더 프리미티브 |
| INFO | 이미 올바름 | 검증된 IP가 제대로 사용됨 |

## 자동 제안 통합

| 감지된 패턴 | 제안 커맨드 | 파라미터 |
|---|---|---|
| 동기 FIFO 신호 | `/gf-ip add fifo_sync` | WIDTH=감지값, DEPTH=next_pow2 |
| 비동기 FIFO 신호 | `/gf-ip add fifo_async` | WIDTH=감지값, DEPTH=8 |
| 2FF CDC 크로싱 | `/gf-ip add cdc_2ff` | 기본값 |
| 멀티비트 CDC 핸드셰이크 | `/gf-ip add cdc_handshake` | WIDTH=감지값 |
| UART 패턴 | `/gf-ip add uart` | CLK_FREQ=감지값, BAUD=115200 |
| SPI 패턴 | `/gf-ip add spi_master` | CLK_DIV=계산값 |
| AXI 레지스터 패턴 | `/gf-ip add axi4lite_slave` | ADDR_WIDTH=감지값 |
| 버튼 디바운스 | `/gf-ip add debouncer` | DEBOUNCE_MS=20 |
