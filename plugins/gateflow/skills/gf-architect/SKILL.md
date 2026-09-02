---
name: gf-architect
description: >
  Codebase architect - Maps and documents SystemVerilog projects.
  This skill should be used when the user wants to understand a codebase structure,
  generate architecture documentation, or onboard to a new RTL project.
  Example requests: "map this codebase", "document the architecture", "show module hierarchy"
allowed-tools:
  - Grep
  - Glob
  - Read
  - Write
  - Bash
  - Task
---

# GF Architect

병렬 서브에이전트를 사용해 SystemVerilog 코드베이스를 매핑합니다.

**중요: 당신은 오케스트레이션하고, Sonnet이 읽습니다.** 코드베이스 파일을 직접 읽지 마세요. 파일 읽기는 항상 Sonnet 서브에이전트에 위임하세요 - 작은 코드베이스라도. 당신은 작업을 계획하고, 서브에이전트를 스폰하며, 그들의 보고서를 종합합니다.

## 에이전트 수 전략

| 코드베이스 토큰 | 에이전트 | 근거 |
|-----------------|--------|-----------|
| < 50k | 2 | 병렬성을 위한 최소 |
| 50k-300k | 3 | 부하 균형, 관련 파일을 함께 |
| 300k-600k | 4-5 | 효율적 병렬 분석 |
| 600k-1M | 6-8 | 에이전트당 150k 미만 유지 |
| > 1M | 8-10 | 10개로 상한, 증분 갱신 사용 |

**규칙:**
- **최소: 2 에이전트** (아주 작은 코드베이스라도 항상 병렬화)
- **최대: 10 에이전트** (수익 체감, 종합 오버헤드)
- **큰 파일 (>80k 토큰):** Grep 우선 전략의 전용 에이전트

## 빠른 시작

1. 기존 맵 확인 (있으면 증분 갱신)
2. 코드베이스를 스캔해 토큰 수가 있는 파일 목록 획득
3. 위 표를 사용해 **에이전트 수 결정**
4. 서브에이전트 배정 계획 (파일 그룹화, 큰 파일 처리)
5. Sonnet 서브에이전트를 병렬로 스폰 (하나의 메시지에 전부)
6. 서브에이전트 보고서를 `.gateflow/map/` 파일로 종합
7. 요약으로 `CLAUDE.md` 갱신

## 출력 구조

```
.gateflow/map/
├── CODEBASE.md          # Main summary (AI-friendly index)
├── hierarchy.md         # Module tree diagram
├── signals.md           # Port and signal flow
├── clock-domains.md     # CDC analysis, resets
├── fsm.md              # State machine diagrams
├── packages.md         # Package dependencies
├── types.md            # Structs, unions, typedefs
├── functions.md        # Functions and tasks
├── macros.md           # Preprocessor directives
├── verification.md     # SVA, coverage, checkers
├── interfaces.md       # Interfaces, modports (if found)
├── classes.md          # UVM/OOP classes (if found)
├── generate.md         # Generate blocks (if found)
├── dpi.md              # DPI imports/exports (if found)
├── recipe.md           # Compile order, filelists
└── modules/            # Per-module detail pages
    └── <module_name>.md
```

---

## 워크플로

### 1단계: 기존 맵 확인

```bash
ls .gateflow/map/CODEBASE.md 2>/dev/null
```

**존재하면:** 마지막 맵 이후 변경 사항 확인:
```bash
# Read last commit from metadata
last_commit=$(cat .gateflow/map/.last_scan_commit 2>/dev/null)
git diff --name-only $last_commit HEAD -- "*.sv" "*.svh" 2>/dev/null
```
- 변경 없음: "Map is up to date"
- 변경 있음: 증분 갱신으로 진행 (변경된 파일만 재매핑)

**존재하지 않으면:** 전체 매핑으로 진행.

### 2단계: 코드베이스 스캔 & 토큰 예산 배정

```bash
mkdir -p .gateflow/map/modules
```

**토큰 수와 함께 파일 스캔:**
```bash
find . \( -name "*.sv" -o -name "*.svh" \) -not -path "./.gateflow/*" | while read f; do
  tokens=$(wc -c < "$f" | awk '{print int($1/4)}')
  echo "$tokens $f"
done | sort -rn
```

**배정 표 구성:**
| 파일 | 토큰 | 배정 |
|------|--------|------------|
| top.sv | 50000 | Agent 1 |
| uart_tx.sv | 8000 | Agent 1 |
| hmac_core.sv | 120000 | Agent 2 (LARGE - use Grep) |

### 3단계: 큰 파일 처리 (>80k 토큰)

**80k 토큰을 초과하는 파일은 청크 분석 사용:**

1. **Grep으로 구조 추출** (전체 파일을 읽지 말 것):
```bash
# Get module declaration
grep -n "^\s*module\s" large_file.sv

# Get ports
grep -n "(input|output|inout)" large_file.sv

# Get instances
grep -n "^\s*\w\+\s\+\w\+\s*(" large_file.sv
```

2. offset/limit을 사용해 **섹션별로 읽기**:
```
Read file with offset=0, limit=500 (header, ports)
Read file with offset=500, limit=500 (logic section 1)
... continue until covered
```

3. Grep 우선 전략으로 **전용 서브에이전트에 배정**

### 4단계: 병렬 서브에이전트 스폰

**중요: 모든 서브에이전트를 하나의 메시지에 스폰.**

Task 도구를 다음과 함께 사용:
- `subagent_type: "Explore"`
- (사용자의 세션 모델을 상속하려면 model 생략)

**예시 - 하나의 메시지에 3개 에이전트 스폰:**

```
Task 1:
  description: "Analyze UART files"
  subagent_type: "Explore"
  prompt: |
    Read and analyze these SystemVerilog files:
    - rtl/uart_pkg.sv
    - rtl/uart_tx.sv
    - rtl/uart_rx.sv

    For EACH file, extract:
    1. Module/Package name
    2. Purpose (one-line)
    3. Ports table: name, direction, width
    4. Parameters: name, type, default
    5. Instances: what it instantiates
    6. FSM states (if any)
    7. Clock/Reset signals
    8. Package imports

    Return structured markdown.

Task 2:
  description: "Analyze SHA files"
  subagent_type: "Explore"
  prompt: |
    Read and analyze these SystemVerilog files:
    - rtl/sha2_pad.sv
    - rtl/sha2_core.sv

    [Same extraction request...]

Task 3:
  description: "Analyze large file with Grep"
  subagent_type: "Explore"
  prompt: |
    This file is large. Use Grep to extract structure first:
    - rtl/hmac_core.sv (120k tokens)

    1. Grep for module declaration
    2. Grep for ports
    3. Grep for instances
    4. Read specific sections if needed

    Return structured markdown.
```

### 5단계: 보고서 종합

모든 서브에이전트 완료 후:

1. 모든 보고서 **병합**
2. 인스턴스 데이터로 **계층 구성**
3. **다이어그램 생성** (Mermaid)
4. **교차 관심사 식별** (클럭, CDC)
5. **출력 파일 작성**

---

## 출력 파일 명세

### CODEBASE.md (메인 인덱스)

```markdown
---
last_mapped: YYYY-MM-DDTHH:MM:SSZ
total_files: N
total_tokens: N
commit: abc123
---

# Codebase Map: [Project Name]

> Auto-generated by GateFlow Architect

## Quick Stats
| Modules | Packages | Interfaces | FSMs | Clocks |
|---------|----------|------------|------|--------|
| N       | N        | N          | N    | N      |

## Module Index
| Module | Type | File | Ports |
|--------|------|------|-------|
| uart_ctrl | top | rtl/uart_ctrl.sv | clk,rst_n,tx_*,rx_* |
| uart_tx | leaf | rtl/uart_tx.sv | clk,rst_n,data[7:0] |

## Package Index
| Package | File | Exports |
|---------|------|---------|
| uart_pkg | rtl/uart_pkg.sv | state_t, BAUD_RATE |

## Key Files
- [Hierarchy](hierarchy.md) - Module tree
- [Signals](signals.md) - Port connections
- [Clock Domains](clock-domains.md) - CDC analysis
- [FSMs](fsm.md) - State machines
- [Packages](packages.md) - Dependencies
- [Types](types.md) - Structs, enums
- [Verification](verification.md) - Assertions

## Navigation Guide
**To trace data flow**: Start at top module, follow instances
**To add new register**: Modify [module]_reg_top.sv
**To add assertion**: See verification.md for patterns
```

### hierarchy.md

```markdown
# Module Hierarchy

## Top Modules
Modules never instantiated by others: [list]

## Hierarchy Tree

\`\`\`mermaid
flowchart TD
    top[hmac]
    top --> core[u_core: hmac_core]
    top --> regs[u_regs: hmac_reg_top]
    core --> sha[u_sha: sha2_multimode]
\`\`\`

## Instance Table
| Parent | Instance | Module | Parameters |
|--------|----------|--------|------------|
| hmac | u_core | hmac_core | - |
| hmac | u_regs | hmac_reg_top | - |
```

### signals.md

```markdown
# Signal Flow Analysis

## Port Summary by Module

### hmac_core
| Port | Dir | Width | Connected To |
|------|-----|-------|--------------|
| clk_i | input | 1 | top.clk |
| data_o | output | [31:0] | regs.wdata |

## Data Flow Diagram

\`\`\`mermaid
flowchart LR
    subgraph Input
        msg[msg_fifo]
    end
    subgraph Core
        pad[sha2_pad]
        hash[sha2_core]
    end
    subgraph Output
        digest[digest_reg]
    end
    msg --> pad --> hash --> digest
\`\`\`

## Unconnected Ports
- [none or list]
```

### clock-domains.md

```markdown
# Clock Domain Analysis

## Clocks Detected
| Clock | Modules |
|-------|---------|
| clk_i | all |

## Resets Detected
| Reset | Type | Modules |
|-------|------|---------|
| rst_ni | async active-low | all |

## Clock Domain Map

\`\`\`mermaid
flowchart LR
    subgraph clk_i_domain["clk_i domain"]
        core[hmac_core]
        regs[hmac_reg_top]
    end
\`\`\`

## CDC Crossings
| Source | Dest | Signal | Sync Type |
|--------|------|--------|-----------|
| [none or list] |
```

### fsm.md

```markdown
# State Machines

## FSM: tx_state in uart_tx

**States:** IDLE, START, DATA, STOP
**Encoding:** 2-bit

\`\`\`mermaid
stateDiagram-v2
    [*] --> IDLE
    IDLE --> START: tx_valid
    START --> DATA: 1 cycle
    DATA --> DATA: bit_cnt < 7
    DATA --> STOP: bit_cnt == 7
    STOP --> IDLE: 1 cycle
\`\`\`

**Transitions:**
| From | To | Condition |
|------|-----|-----------|
| IDLE | START | tx_valid |
| START | DATA | always |
| DATA | STOP | bit_cnt == 7 |
```

### packages.md

```markdown
# Packages

## uart_pkg
**File:** rtl/uart_pkg.sv

**Exports:**
- Types: state_t, config_t
- Parameters: BAUD_RATE, DATA_BITS
- Functions: calc_divisor()

## Import Graph

\`\`\`mermaid
flowchart TD
    pkg[uart_pkg]
    tx[uart_tx.sv] -->|import| pkg
    rx[uart_rx.sv] -->|import| pkg
\`\`\`
```

### types.md

```markdown
# Type Definitions

## Structs

### request_t
**File:** pkg/types_pkg.sv:15
\`\`\`systemverilog
typedef struct packed {
    logic [7:0] addr;
    logic [31:0] data;
} request_t;
\`\`\`
**Width:** 40 bits

## Enums

| Name | Values | Width | File |
|------|--------|-------|------|
| state_t | IDLE,RUN,DONE | 2-bit | uart_pkg.sv:10 |

## Typedefs

| Alias | Base Type | File |
|-------|-----------|------|
| data_t | logic[31:0] | types.sv:5 |
```

### functions.md

```markdown
# Functions and Tasks

## Functions

### calc_crc
**File:** rtl/crc_pkg.sv:20
**Return:** logic [15:0]
**Args:** input logic [7:0] data
**Used by:** uart_tx, uart_rx

## Tasks

### send_byte
**File:** tb/uart_tb.sv:50
**Args:** input byte data
```

### macros.md

```markdown
# Preprocessor Directives

## Defines
| Macro | Value | File |
|-------|-------|------|
| DATA_WIDTH | 32 | defines.svh:1 |

## Include Graph

\`\`\`mermaid
flowchart TD
    top[top.sv] -->|include| defs[defines.svh]
    uart[uart.sv] -->|include| defs
\`\`\`

## Conditional Compilation
| Condition | Files |
|-----------|-------|
| \`ifdef SIMULATION | tb/*.sv |
```

### verification.md

```markdown
# Verification Constructs

## Assertions (SVA)

### uart_tx assertions
| Property | Type | Description |
|----------|------|-------------|
| p_no_overflow | assert | FIFO never overflows |
| p_valid_stable | assert | valid held until ready |

## Covergroups

### cg_opcodes
**File:** tb/coverage.sv:30
**Coverpoints:** opcode, size
**Crosses:** opcode x size

## Bind Statements
| Target | Checker | File |
|--------|---------|------|
| uart_tx | uart_sva | bind.sv:5 |
```

### interfaces.md (if found)

```markdown
# Interfaces

## axi_if
**File:** rtl/axi_if.sv
**Parameters:** ADDR_WIDTH, DATA_WIDTH

### Signals
| Name | Type | Width |
|------|------|-------|
| awvalid | logic | 1 |
| awaddr | logic | [ADDR_WIDTH-1:0] |

### Modports
| Name | Signals |
|------|---------|
| master | output: aw*, w*, input: b* |
| slave | input: aw*, w*, output: b* |
```

### classes.md (if UVM found)

```markdown
# Classes

## Class Hierarchy

\`\`\`mermaid
classDiagram
    uvm_driver <|-- uart_driver
    uvm_monitor <|-- uart_monitor
\`\`\`

## uart_driver
**File:** tb/uart_driver.sv
**Extends:** uvm_driver
```

### generate.md (if found)

```markdown
# Generate Blocks

## uart_fifo generate loop
**File:** rtl/uart_fifo.sv:50

\`\`\`systemverilog
genvar i;
generate
    for (i = 0; i < DEPTH; i++) begin : gen_mem
        // memory slice
    end
endgenerate
\`\`\`
```

### dpi.md (if found)

```markdown
# DPI Functions

## Imports
| SV Name | C Name | Return | Args |
|---------|--------|--------|------|
| c_calc | calc | int | int a, int b |

## Exports
| SV Name | C Name |
|---------|--------|
| sv_notify | notify |
```

### recipe.md

```markdown
# Build Recipe

## Compile Order
1. Packages (no deps)
2. Interfaces
3. Leaf modules
4. Top modules

\`\`\`mermaid
flowchart LR
    pkg[uart_pkg.sv] --> leaf[uart_tx.sv]
    pkg --> leaf2[uart_rx.sv]
    leaf --> top[uart_ctrl.sv]
    leaf2 --> top
\`\`\`

## Filelists Found
| File | Entries |
|------|---------|
| rtl.f | 15 files |

## Include Paths
- ./rtl
- ./include
```

### 모듈별 페이지 (modules/*.md)

```markdown
# Module: uart_tx

## Overview
UART transmitter with configurable baud rate.

## Location
- **File:** rtl/uart_tx.sv
- **Lines:** 1-145

## Parameters
| Name | Type | Default | Description |
|------|------|---------|-------------|
| CLK_FREQ | int | 100000000 | System clock Hz |
| BAUD | int | 115200 | Baud rate |

## Ports
| Name | Dir | Width | Description |
|------|-----|-------|-------------|
| clk | input | 1 | System clock |
| rst_n | input | 1 | Active-low reset |
| tx_data | input | [7:0] | Byte to send |
| tx_valid | input | 1 | Data valid |
| tx_ready | output | 1 | Ready for data |
| tx | output | 1 | Serial output |

## Instances
None (leaf module)

## FSM
State type: tx_state_t {IDLE, START, DATA, STOP}
[Link to fsm.md#uart_tx]

## Clock/Reset
- Clock: clk (single domain)
- Reset: rst_n (async active-low)

## Assertions
- p_valid_hold: valid held until ready
```

---

## 정규식 패턴 레퍼런스

```
# Design Units
^\s*module\s+(\w+)              # Module
^\s*interface\s+(\w+)           # Interface
^\s*package\s+(\w+)             # Package
^\s*program\s+(\w+)             # Program
^\s*class\s+(\w+)               # Class
^\s*checker\s+(\w+)             # Checker

# Ports & Parameters
(input|output|inout)\s+(logic|wire|reg)?\s*(\[.*?\])?\s*(\w+)
parameter\s+(int|logic|integer)?\s*(\w+)\s*=
localparam\s+

# Instances
^\s*(\w+)\s*(#\s*\([^)]*\))?\s+(\w+)\s*\(

# Always blocks
always_ff\s*@|always_comb|always_latch|always\s*@

# Types
typedef\s+enum
typedef\s+struct\s+packed
typedef\s+union

# Functions/Tasks
function\s+(automatic\s+)?
task\s+(automatic\s+)?

# Preprocessor
`define\s+(\w+)
`include\s+"([^"]+)"
`ifdef|`ifndef|`elsif

# Verification
assert\s+property|assume\s+property|cover\s+property
covergroup\s+(\w+)
sequence\s+(\w+)
property\s+(\w+)
bind\s+\w+

# Generate
generate|genvar

# Interface features
modport\s+\w+
clocking\s+\w+

# OOP/UVM
class\s+\w+|extends\s+\w+|virtual\s+class
constraint\s+\w+
rand\s+|randc\s+

# DPI
import\s+"DPI|export\s+"DPI
```

---

## 매핑 후 메타데이터 저장

```bash
# Save commit hash
git rev-parse HEAD > .gateflow/map/.last_scan_commit

# Save timestamp
date -u +"%Y-%m-%dT%H:%M:%SZ" > .gateflow/map/.last_scan

# Save file hashes for non-git change detection
find . -name "*.sv" -o -name "*.svh" | xargs md5 > .gateflow/map/.file_hashes
```

---

## 증분 갱신 모드

기존 맵을 갱신할 때:

1. 변경된 파일 획득: `git diff --name-only <last_commit> HEAD -- "*.sv"`
2. 변경된 파일 그룹에 대해서만 서브에이전트 스폰
3. 새 분석을 기존 맵 파일과 병합
4. 프론트매터 타임스탬프 갱신
5. 영향받은 다이어그램 재생성

---

## 품질 경고

CODEBASE.md의 "## Warnings" 아래에 보고:
- 추론된 래치 (default 없는 always)
- 순차 로직의 리셋 누락
- 미연결 포트
- 동기화기 없는 CDC 크로싱
- 미사용 신호/파라미터
- 톱 모듈 누락

---

## 토큰 예산 레퍼런스

| 모델 | 컨텍스트 | 안전 예산 |
|-------|---------|-------------|
| Sonnet | 200k | 에이전트당 150k |
| Haiku | 200k | 에이전트당 100k |

분석 품질을 위해 **항상 Sonnet 사용**. 150k 예산은 에이전트 추론과 출력을 위해 50k 여유를 남깁니다.

---

## 문제 해결

**단일 에이전트에 파일이 너무 큼:**
- 먼저 Grep으로 구조 추출
- offset/limit으로 청크 단위로 읽기
- 전용 서브에이전트 배정

**파일이 너무 많음:**
- 서브에이전트 수 늘리기
- RTL에 집중, 테스트벤치 건너뛰기
- glob 패턴으로 필터링

**git을 사용할 수 없음:**
- 파일 해시 비교로 폴백
- 변경 감지를 위해 .file_hashes 저장

---

## Verilator JSON 모드

Verilator가 사용 가능하면, 더 정확한 매핑을 위해 `--json-only` 사용:
```bash
verilator --json-only --json-only-output design.tree.json --no-json-edit-nums -Wall <files>.sv
```

주요 AST 노드: MODULE (모듈), VAR (ioDirection이 있는 신호), CELL (인스턴스), ASSIGNW/ASSIGNDLY (연결). 정규식 대비 이점: 파라미터/generate 해결, 엘라보레이션된 계층 캡처, 타입 폭 정보. Verilator가 없으면 정규식으로 폴백.

## 복잡도 지표

각 모듈 페이지와 CODEBASE.md 요약에 추가:

| 지표 | 측정하는 것 |
|---|---|
| CC (순환 복잡도) | 조합 논리의 결정 지점 |
| FSM_STATES | FSM 상태 수 |
| ALWAYS | always 블록 수 |
| PORTS | 포트 수 |
| INSTANCES | 서브모듈 수 |
| SLOC | 공백/주석이 아닌 줄 |

복합: `(CC*3) + (FSM_STATES*2) + ALWAYS + (PORTS/5) + (INSTANCES*2)`. 등급: 1-10 Low, 11-30 Medium, 31-60 High, 61+ Critical.

## 신호 추적

방향성 연결 그래프 구성 (노드=신호, 엣지=대입+포트 바인딩). 소스에서 BFS 순방향, 목적지에서 BFS 역방향. 모듈 크로싱과 파이프라인 스테이지 마커가 있는 추적 경로 출력.

## Diff 인식 매핑

각 맵 후, 스냅샷 저장. 다음 맵에서 모듈 수준 비교: 구체적 변경 타입(port, instance, FSM, parameter)과 함께 ADDED/MODIFIED/REMOVED. `.gateflow/map/CHANGES.md`에 출력.

## 의존성 그래프 출력

Mermaid로 `.gateflow/map/dependencies.md` 생성: 인스턴스화에는 실선 화살표, import에는 점선, 클럭 도메인에는 subgraph, top/mid/leaf/package에는 classDef.
