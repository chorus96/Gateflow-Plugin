---
name: gf-errors
description: >
  Error translation layer for hardware tool outputs. Converts cryptic
  Verilator, Yosys, GHDL, and simulation errors into 3-layer explanations.
  Used internally by gf orchestrator — not user-invocable directly.
user-invocable: false
---

# GF Errors — 하드웨어 오류 번역

어떤 도구든 오류를 낼 때, 사용자에게 제시하기 전에 세 계층을 거쳐 번역하세요.
번역 없이 원시 도구 출력을 절대 보여주지 마세요.

## 번역 프로토콜

Verilator, Yosys, GHDL, 시뮬레이션의 모든 오류에 대해:

### 계층 1 — WHAT (한 문장, 평이한 영어)
무슨 일이 일어났는지 서술. 기술 전문 용어 없음. 도구 이름 없음.
- BAD: "%Error: counter.sv:15:17: Cannot find signal: 'clk_in'"
- GOOD: "The signal `clk_in` doesn't exist in this module."

### 계층 2 — WHY (1-2문장, 맥락)
왜 일어났는지 설명. 특정 줄과 이름을 참조.
- "Your module declares its clock input as `clk` (line 5), but the
  always block on line 15 references `clk_in`. These names must match."

### 계층 3 — FIX (실행 가능, 구체적)
정확히 무엇을 할지 사용자에게 알려줌. 파일, 줄, 변경을 포함.
- "Change `clk_in` to `clk` on line 15 of counter.sv."

## 흔한 Verilator 오류

| 오류 코드 | 계층 1 템플릿 | 계층 2 지침 |
|-----------|-----------------|-----------------|
| UNUSED | "Signal `{name}` is declared but never used." | 연결되어야 하는지, 제거 가능한지 확인. |
| UNDRIVEN | "Signal `{name}` is never assigned a value." | 선언되었으나 구동하는 로직이 없음. |
| WIDTH | "Width mismatch: `{lhs}` is {lw} bits but `{rhs}` is {rw} bits." | 명시적 크기 지정이 필요. |
| CASEINCOMPLETE | "Case statement doesn't cover all possible values." | `default:` 분기를 추가. |
| LATCH | "Unintended latch inferred for `{name}`." | 조합 블록에 `{name}`이 대입되지 않는 경로가 있음. always_comb 블록 상단에 기본 대입을 추가. |
| BLKSEQ | "Blocking assignment used in sequential block." | `always_ff`에서 `=`(blocking)이 아니라 `<=`(non-blocking)를 사용. |
| PINMISSING | "Port `{name}` on instance `{inst}` is not connected." | 연결하거나 명시적으로 미연결로 표시. |

## 흔한 시뮬레이션 오류

| 증상 | 계층 1 | 계층 2 |
|---------|---------|---------|
| 출력의 X 값 | "Output `{sig}` has unknown (X) values." | 신호가 알려진 상태로 구동된 적이 없음. 리셋 커버리지 확인 — 모든 순차 로직이 리셋되는지 확인. |
| 시뮬레이션 행 | "Simulation never reaches `$finish`." | 테스트벤치의 무한 루프나 종료 조건 누락일 가능성. 타임아웃 없는 blocking 대기를 확인. |
| 어서션 실패 | "Assertion `{name}` failed at time {t}." | 확인 중인 프로퍼티가 위반됨. 어서션 조건과 실패 시점의 신호 값을 검토. |
| 잘못된 출력 | "Output `{sig}` is {actual}, expected {expected}." | 설계의 로직 오류 또는 잘못된 테스트 기대값. 신호를 원천까지 추적. |

## 흔한 Yosys 합성 오류

| 오류 패턴 | 계층 1 | 계층 2 |
|--------------|---------|---------|
| Module not found | "Module `{name}` was not included in synthesis." | 모든 소스 파일이 Yosys에 전달되어야 함. 파일 목록을 확인. |
| Unsupported construct | "Yosys doesn't support `{construct}` in synthesis." | 이 SystemVerilog 기능은 Yosys에서 사용할 수 없음. Verilog-2005 호환 구문으로 재작성. |
| Combinational loop | "Circular dependency detected in combinational logic." | 신호 `{sig}`가 레지스터 없이 자신에게 피드백됨. 플립플롭을 추가해 루프를 끊음. |

## 오케스트레이터의 사용

`/gf` 오케스트레이터가 gf-lint 또는 gf-sim에서 FAIL 상태를 받으면,
반드시:
1. 원시 오류 출력을 파싱
2. 위 표와 대조
3. 3계층 번역을 사용자에게 제시
4. 그다음 원시 오류와 함께 수정 에이전트(sv-refactor 또는 sv-debug)를 스폰

수정 에이전트는 원시 오류를 받습니다. 사용자는 번역된 버전을 받습니다.
둘 다 같은 문제를 보지만, 사용자는 이해할 수 있는 언어로 봅니다.
