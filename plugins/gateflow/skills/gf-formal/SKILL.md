---
name: gf-formal
description: >
  Formal verification from natural language. Generates SVA properties,
  configures SymbiYosys, runs proofs, and explains results.
  Example: "formally verify the FIFO never overflows"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - Task
  - WebSearch
  - AskUserQuestion
---

# GF-Formal -- Formal 검증 스킬

## 도구 감지

```bash
which sby
```

찾을 수 없으면:
```
---GATEFLOW-RESULT---
STATUS: ERROR
DETAILS: SymbiYosys not installed. Install to enable formal verification.
  pip install symbiyosys
  Also need: yosys, z3 (or yices2)
  macOS: brew install yosys z3
  Linux: sudo apt install yosys z3
---END-GATEFLOW-RESULT---
```

## 워크플로

1. 요청 파싱 -- 어떤 프로퍼티를 검증? 어느 모듈?
2. 설계 읽기 -- 포트, 신호, 동작을 이해
3. sv-formal 에이전트 스폰 -- 프로퍼티 + .sby 구성 생성
4. SymbiYosys 실행: `sby -f <config>.sby`
5. 결과 파싱 -- pass/fail/반례를 위해 sby 출력을 읽음
6. 보고 -- 실패 시 3계층 오류 번역, 통과 시 명확한 요약

## 결과 형식

```
---GATEFLOW-RESULT---
STATUS: PASS | FAIL | ERROR
PROOFS: N proved, M failed, K covers
FILES: [generated files]
DETAILS: [proof results or counterexample explanation]
---END-GATEFLOW-RESULT---
```

## /gf 오케스트레이터와의 통합

Formal 검증은 선택적 향상 단계입니다:
- 안전 필수 설계의 경우, 시뮬레이션 통과 후
- 사용자가 Formal 검증을 명시적으로 요청할 때
- CDC, FIFO, 프로토콜 설계의 경우

## .sby 구성 템플릿

### BMC 템플릿
```sby
[tasks]
bmc
[options]
mode bmc
depth 20
expect pass
[engines]
smtbmc z3
[script]
read -formal design.sv
prep -top top_module
[files]
design.sv
```

### Prove 템플릿
```sby
[tasks]
prove
[options]
mode prove
depth 40
expect pass
[engines]
smtbmc z3
abc pdr
[script]
read -formal design.sv
prep -top top_module
[files]
design.sv
```

### Cover 템플릿
```sby
[tasks]
cover
[options]
mode cover
depth 30
expect pass
[engines]
smtbmc z3
[script]
read -formal design.sv
prep -top top_module
[files]
design.sv
```

### 멀티 태스크 (BMC + Prove + Cover)
```sby
[tasks]
bmc
prove
cover
[options]
bmc: mode bmc
bmc: depth 20
prove: mode prove
prove: depth 40
cover: mode cover
cover: depth 30
expect pass
[engines]
bmc: smtbmc z3
prove: smtbmc z3
prove: abc pdr
cover: smtbmc z3
[script]
read -formal design.sv
prep -top top_module
[files]
design.sv
```

### .sby 옵션
| 옵션 | 모드 | 설명 |
|--------|-------|-------------|
| `mode` | all | `bmc`, `prove`, `cover`, `live` |
| `depth` | bmc, cover | 확인할 사이클 (기본 20) |
| `timeout` | all | 초 단위 타임아웃 |
| `multiclock` | all | 다중 클럭 / 비동기 |
| `expect` | all | 예상: pass, fail, unknown |

## SVA 프로퍼티 패턴

### 오버플로 없음
```systemverilog
a_no_overflow: assert property (
    @(posedge clk) disable iff (rst) full |-> !wr_en);
```

### 언더플로 없음
```systemverilog
a_no_underflow: assert property (
    @(posedge clk) disable iff (rst) empty |-> !rd_en);
```

### Valid/Ready 핸드셰이크
```systemverilog
a_valid_stable: assert property (
    @(posedge clk) disable iff (rst) (valid && !ready) |=> valid);
a_data_stable: assert property (
    @(posedge clk) disable iff (rst) (valid && !ready) |=> $stable(data));
```

### One-Hot
```systemverilog
a_onehot: assert property (
    @(posedge clk) disable iff (rst) $onehot(state));
```

### 활성성
```systemverilog
a_req_granted: assert property (
    @(posedge clk) disable iff (rst) req |-> ##[1:MAX_LATENCY] grant);
```

### 리셋 동작
```systemverilog
a_reset: assert property (
    @(posedge clk) rst |-> (data_out == '0) && (count == '0));
```

### FIFO 카운트
```systemverilog
a_count_inc: assert property (
    @(posedge clk) disable iff (rst)
    (wr_en && !rd_en && !full) |=> (count == $past(count) + 1));
```

## 증명 전략

| 프로퍼티 유형 | 접근법 | 엔진 |
|---|---|---|
| 단순 경계 | BMC 그다음 prove | `smtbmc z3` |
| 프로토콜 준수 | BMC + prove | `smtbmc z3`, `abc pdr` |
| FSM 정확성 | Prove | `abc pdr` |
| 활성성 | Live 모드 | `aiger suprove` |
| 복잡한 산술 | BMC | `smtbmc bitwuzla` |

## 엔진

| 엔진 | 모드 | 강점 |
|---|---|---|
| `smtbmc` | bmc, prove, cover | 읽기 쉬운 트레이스, k-귀납법 |
| `abc pdr` | prove | 무계 증명, 자동 불변식 |
| `abc bmc3` | bmc | 빠른 비트 수준 검사 |
| `aiger suprove` | prove, live | 활성성 검증 |

### SMT 솔버
| 솔버 | 최적 용도 |
|---|---|
| `z3` | 좋은 기본값 |
| `yices` | 빠른 비트 벡터 |
| `bitwuzla` | 복잡한 산술 |
| `boolector` | 하드웨어 특화 |

## 반례 해석

| 실패 | 트레이스 위치 |
|---|---|
| BMC | `<task>/engine_0/trace.vcd` |
| 귀납 | `<task>/engine_0/trace_induct.vcd` |
| Cover | `<task>/engine_0/trace<N>.vcd` |

### 디버깅
- BMC 실패: 반례가 도달 가능, 설계 수정
- Prove 실패하나 BMC 통과: 도달 불가능한 귀납 상태, 불변식 추가 또는 `abc pdr` 사용
- Cover 실패: 과도하게 제약됨, 가정 완화

| 패턴 | 수정 |
|---|---|
| 초기값 누락 | 리셋 로직 추가 |
| 제약되지 않은 입력 | `assume` 프로퍼티 추가 |
| 도달 불가능한 귀납 | 불변식 강화 또는 `abc pdr` 사용 |

## Formal 확장

| 지시자 | 목적 |
|---|---|
| `assert(expr)` | 항상 참이어야 함 |
| `assume(expr)` | 솔버 입력을 제약 |
| `cover(expr)` | 도달 가능성 대상 |

| 속성 | 동작 |
|---|---|
| `(* anyconst *)` | 솔버가 상수를 선택 |
| `(* anyseq *)` | 솔버가 사이클마다 선택 |

```systemverilog
`ifdef FORMAL
    initial assume(rst);
    always @(posedge clk) begin
        a_example: assert(count <= MAX);
        c_reach: cover(count == MAX);
    end
`endif
```
