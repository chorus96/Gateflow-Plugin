# 반례 해석

## 트레이스 위치

| 실패 | 위치 |
|---|---|
| BMC 실패 | `<task>/engine_0/trace.vcd` |
| 귀납 실패 | `<task>/engine_0/trace_induct.vcd` |
| Cover 트레이스 | `<task>/engine_0/trace<N>.vcd` |

## 디버깅 워크플로

```
Assertion Fails
  +-- BMC mode? --> Counterexample is REACHABLE --> Fix design
  +-- Prove mode?
        +-- BMC also fails? --> Real bug, fix design
        +-- BMC passes? --> Unreachable induction state
              --> Add invariant assertions
              --> Or try `abc pdr` (builds invariants automatically)
```

## 흔한 실패 패턴

| 패턴 | 수정 |
|---|---|
| 초기값 누락 | 리셋 로직 추가 |
| 제약되지 않은 입력 | `assume` 프로퍼티 추가 |
| 도달 불가능한 귀납 상태 | 불변식 강화 또는 `abc pdr` 사용 |
| 과도하게 제약됨 (cover 실패) | 가정을 완화 |
