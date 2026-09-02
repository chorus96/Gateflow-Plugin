# 증명 전략과 엔진

## 증명 전략

```
What to verify?
  |
  +-- Bugs in first N cycles? --> BMC (mode bmc)
  +-- Always true, forever?   --> Prove (mode prove)
  +-- Can state be reached?   --> Cover (mode cover)
  +-- Eventually happens?     --> Live (mode live)
```

| 프로퍼티 유형 | 접근법 | 엔진 |
|---|---|---|
| 단순 경계 | BMC 먼저, 그다음 prove | `smtbmc z3` |
| 프로토콜 준수 | BMC + prove | `smtbmc z3`, `abc pdr` |
| FSM 정확성 | 불변식으로 prove | `abc pdr` |
| 활성성 | Live 모드 | `aiger suprove` |
| 복잡한 산술 | BMC (prove는 타임아웃 가능) | `smtbmc bitwuzla` |

## 엔진 비교

| 엔진 | 모드 | 강점 |
|---|---|---|
| `smtbmc` | bmc, prove, cover | 사람이 읽기 쉬운 트레이스, k-귀납법 |
| `abc pdr` | prove | 강력한 무계 증명, 자동 불변식 |
| `abc bmc3` | bmc | 빠른 비트 수준 유계 검사 |
| `aiger suprove` | prove, live | 활성성 검증 |

## SMT 솔버 옵션

| 솔버 | 최적 용도 |
|---|---|
| `z3` | 좋은 기본값, 관대한 라이선스 |
| `yices` | 빠른 비트 벡터 문제 |
| `bitwuzla` | 복잡한 산술 |
| `boolector` | 하드웨어 특화 |
