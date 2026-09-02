---
name: gf-build
description: >
  Parallel build orchestrator for SystemVerilog creation tasks.
  Decomposes designs into independent modules, builds in parallel phases,
  runs verification on each component, then integrates.
  Example: "/gf-build RISC-V CPU with ALU, regfile, decoder, control FSM"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - Skill
  - AskUserQuestion
---

# GF-Build: 병렬 설계 오케스트레이터

당신은 설계를 분해하고 병렬 에이전트를 스폰하여 RTL 빌드를 오케스트레이션합니다.

## 호출

사용자가 말함: `/gf-build <design description>`

## 워크플로

### 1단계: 분해

요청을 분석하고 식별:

```markdown
## Design Decomposition: [Name]

### Shared Resources (Phase 0)
| File | Purpose |
|------|---------|
| pkg.sv | Common types, opcodes |

### Independent Components (Phase 1 - Parallel)
| Component | File | Agent | Dependencies |
|-----------|------|-------|--------------|
| ALU | alu.sv | sv-codegen | pkg |
| RegFile | regfile.sv | sv-codegen | pkg |
| ImmGen | imm_gen.sv | sv-codegen | pkg |

### Dependent Components (Phase 2 - Parallel)
| Component | File | Agent | Dependencies |
|-----------|------|-------|--------------|
| Decoder | decoder.sv | sv-codegen | pkg |
| Control | control.sv | sv-codegen | pkg, decoder |

### Integration (Phase 3)
| File | Purpose |
|------|---------|
| top.sv | Connects all components |

### Verification (Phase 4 - Parallel)
| Testbench | Tests |
|-----------|-------|
| tb_alu.sv | ALU operations |
| tb_top.sv | Integration |
```

### 2단계: 승인 요청

```
I've decomposed your design into [N] components across [M] phases.
Phase 1 will spawn [X] parallel agents.

Proceed with parallel build?
```

### 단일 모듈 요청

설계가 단일 모듈로 분해되면:
- 컴포넌트 하나가 있는 Phase 1로 취급
- 하나의 sv-codegen 작업을 스폰 (여전히 병렬 패턴 사용)
- 평소처럼 lint/testbench/sim 진행

### 3단계: 단계 실행

#### Phase 0: 설정
```bash
mkdir -p rtl tb
```
공유 패키지를 만들기 위해 sv-codegen을 스폰 (에이전트 전용 규칙을 일관되게 유지).

#### Phase 1: 병렬 컴포넌트 빌드

**중요: 하나의 메시지에 여러 Task 호출로 모든 Phase 1 에이전트를 스폰.**

```
<Task 1>
subagent_type: gateflow:sv-codegen
prompt: |
  ## Component: ALU
  [full spec...]
  Write to: rtl/alu.sv
</Task 1>

<Task 2>
subagent_type: gateflow:sv-codegen
prompt: |
  ## Component: Register File
  [full spec...]
  Write to: rtl/regfile.sv
</Task 2>

<Task 3>
subagent_type: gateflow:sv-codegen
prompt: |
  ## Component: Immediate Generator
  [full spec...]
  Write to: rtl/imm_gen.sv
</Task 3>
```

#### Phase 2: 병렬 Lint

모든 컴포넌트에 lint 실행:
```
Skill: gf-lint
args: rtl/alu.sv rtl/regfile.sv rtl/imm_gen.sv
```

결과를 파싱. 실패가 있으면 sv-refactor 에이전트를 병렬로 스폰.

#### Phase 3: 통합

다음 중:
- 컴포넌트 인터페이스로 톱레벨용 sv-codegen 스폰
- 또는 간단하면 직접 작성

#### Phase 4: 병렬 테스트벤치

테스트벤치 에이전트를 병렬로 스폰:
```
<Task 1> sv-testbench for ALU
<Task 2> sv-testbench for RegFile
<Task 3> sv-testbench for top
```

#### Phase 5: 시뮬레이션

시뮬레이션 실행 (병렬 가능):
```
Skill: gf-sim tb/tb_alu.sv rtl/alu.sv
Skill: gf-sim tb/tb_top.sv rtl/*.sv
```

### 4단계: 보고

```markdown
## Build Complete

### Files Created
| Phase | File | Status |
|-------|------|--------|
| 0 | rtl/pkg.sv | ✓ |
| 1 | rtl/alu.sv | ✓ lint-clean |
| 1 | rtl/regfile.sv | ✓ lint-clean |
| 1 | rtl/imm_gen.sv | ✓ lint-clean |
| 3 | rtl/cpu.sv | ✓ lint-clean |
| 4 | tb/tb_cpu.sv | ✓ sim-pass |

### Parallel Efficiency
- Phase 1: 3 agents parallel (vs 3 sequential)
- Phase 4: 2 agents parallel (vs 2 sequential)

### Next Steps
- Run full simulation: `verilator --binary rtl/*.sv tb/tb_cpu.sv`
- Add assertions: `/gf add assertions to cpu.sv`
```

---

## 에이전트 프롬프트 템플릿

### sv-codegen 컴포넌트 프롬프트

```markdown
## Component: [NAME]

## Context
Part of: [parent design]
Package: [package to import]

## Specification
[Detailed functional spec]

## Interface
```systemverilog
module [name] #(
    parameter int PARAM = VALUE
) (
    input  logic        clk,
    input  logic        rst_n,
    // [grouped ports]
);
```

## Requirements
1. [Requirement]
2. [Requirement]

## Output
Write to: [path]
```

### sv-testbench 컴포넌트 프롬프트

```markdown
## Testbench for: [DUT_NAME]

## DUT Location
rtl/[dut].sv

## Test Scenarios
1. [Test case 1]
2. [Test case 2]
3. [Edge case]

## Self-Checking
- Compare expected vs actual
- Use assertions
- Report pass/fail

## Output
Write to: tb/tb_[dut].sv
```

---

## 병렬성 규칙

1. **같은 단계 = 병렬** - 의존성이 없는 컴포넌트는 함께 스폰
2. **다른 단계 = 순차** - 이전 단계 완료를 대기
3. **Lint = 배치** - 모든 파일에 한 번에, 또는 파일당 병렬
4. **수정 = 병렬** - 실패한 각 파일은 자체 sv-refactor 에이전트를 받음
5. **Sim = 병렬** - 각 테스트벤치는 독립적으로 실행 가능

---

## 오류 복구

| 오류 | 조치 |
|-------|--------|
| 에이전트 타임아웃 | 한 번 재시도, 그다음 사용자에게 질문 |
| Lint 실패 | sv-refactor 스폰, 재lint |
| Sim 실패 | sv-debug 스폰, 그다음 sv-refactor |
| 통합 불일치 | 인터페이스 확인, 수동 수정 |
| 연속 2회 실패 | 사용자에게 지침 요청 |

---

## 의존성 그래프 알고리즘

병렬 단계를 식별하기 위해 Kahn 알고리즘(BFS 위상 정렬)을 사용:
1. 각 컴포넌트의 인접 리스트와 in-degree 구성
2. 모든 in-degree=0 컴포넌트로 큐 초기화
3. 레벨별로 큐를 비움 -- 각 레벨이 병렬 단계
4. 모든 레벨 후에도 in_degree>0인 컴포넌트가 있으면: 사이클 감지됨

---

## 자원 경합 규칙

| 규칙 | 설명 |
|---|---|
| 파일당 하나의 작성자 | 각 파일은 정확히 한 에이전트가 소유 |
| 읽기 전 쓰기 | 작성자는 독자보다 이른 단계에 |
| 공유 패키지 패턴 | 공유 타입은 Phase 0의 pkg.sv로 |
| 암시적 의존성 없음 | 에이전트 프롬프트는 CREATE, READ, NEVER MODIFY할 파일을 나열 |

---

## 증분 빌드

`.gateflow/cache/hashes.json`에 파일 해시를 추적:
```json
{"rtl/alu.sv": {"sha256": "a1b2...", "last_lint": "PASS", "last_sim": "PASS"}}
```

결정: 해시가 변경되지 않았고 모든 의존성 해시가 변경되지 않았으면 -> 건너뜀 (캐시 히트). 그렇지 않으면 재실행.

의존성 인식 무효화: pkg.sv가 변경되면, 모든 importer를 무효화.

---

## 빌드 캐시 구조

```
.gateflow/cache/
  hashes.json          # hash -> result mapping
  deps.json            # dependency graph
  lint/<file>.lint     # cached lint output
  sim/<tb>/result.json # cached sim result
```

---

## 진행 상황 시각화

```
[Phase 0] Setup         [====================] DONE  0:04
[Phase 1] Components    [==========>         ] 62%   0:30
  alu.sv                [====================] DONE
  regfile.sv            [====================] DONE
  decoder.sv            [=========>          ] 55%
[Phase 2] Dependent     [                    ] WAIT
```

---

## 반환 형식

```
---GATEFLOW-RETURN---
STATUS: complete
SUMMARY: Built [design] with [N] components in [M] parallel phases
PARALLEL_SPEEDUP: [X]x (spawned N agents vs sequential)
FILES_CREATED:
  - rtl/pkg.sv
  - rtl/alu.sv
  - rtl/regfile.sv
  - rtl/cpu.sv
  - tb/tb_cpu.sv
VERIFICATION:
  - Lint: PASS (0 errors, 0 warnings)
  - Sim: PASS (all tests)
---END-GATEFLOW-RETURN---
```
