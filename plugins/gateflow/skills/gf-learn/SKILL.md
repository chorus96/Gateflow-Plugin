---
name: gf-learn
description: "SystemVerilog learning mode — generates exercises, reviews solutions, and teaches RTL design patterns. Use when the user wants to learn SystemVerilog, practice hardware design, get exercises, or understand verification methodology."
user-invocable: true
triggers:
  - teach me SystemVerilog
  - give me an exercise
  - learning mode
  - practice RTL design
  - review my solution
  - SystemVerilog tutorial
  - help me learn hardware design
  - generate a learning exercise
---
allowed-tools:
  - Read
  - Write
  - Bash
  - Task
  - AskUserQuestion

# GateFlow 학습 모드

연습 문제와 솔루션 리뷰가 있는 대화형 SystemVerilog 학습.

## 사용법

```
/gf-learn                    # Start learning session, pick topic
/gf-learn <topic>            # Get exercises on specific topic
/gf-learn check <file>       # Submit solution for review
/gf-learn hint               # Get hint for current exercise
/gf-learn solution           # Show solution (gives up)
```

## 주제

| 주제 | 연습 |
|-------|-----------|
| `basics` | 신호, always 블록, 대입 |
| `fsm` | 상태 머신, 인코딩, 전이 |
| `fifo` | 동기 FIFO, 포인터, full/empty |
| `pipeline` | Valid/ready, 백프레셔, 스테이지 |
| `cdc` | 클럭 도메인 크로싱, 동기화기 |
| `arbiter` | 라운드 로빈, 우선순위, grant 로직 |
| `memory` | RAM, ROM, 레지스터 파일 |
| `protocol` | AXI-lite, Wishbone, 핸드셰이크 |
| `verification` | 어서션, SVA 프로퍼티, 커버리지 |
| `optimization` | 타이밍 클로저, 자원 공유, 파이프라이닝 트레이드오프 |

## 워크플로

### 1단계: 연습 생성

사용자가 `/gf-learn <topic>`을 실행하면:

1. 난이도가 점점 높아지는 연습 3-5개를 제시
2. 각 연습을 다음과 같이 형식화:

```markdown
## Exercise X: <Title>

**Difficulty:** Beginner/Intermediate/Advanced

**Requirements:**
- [ ] Requirement 1
- [ ] Requirement 2
- [ ] Requirement 3

**Interface:**
```systemverilog
module exercise_name (
    input  logic clk,
    input  logic rst_n,
    // ... define ports
);
```

**Test Cases:**
1. When X happens, Y should occur
2. Edge case: ...

**Starter File:** `exercises/exercise_X.sv`
```

3. `exercises/` 디렉터리에 스타터 파일 생성
4. **멈추고 사용자가 솔루션을 시도할 때까지 대기**

### 2단계: 사용자 대기

연습을 제시한 후, 안내:

```
Your turn! Edit the file and run `/gf-learn check exercises/exercise_X.sv` when ready.

Need help? Run `/gf-learn hint` for a hint.
```

**사용자가 `/gf-learn check`로 제출할 때까지 진행하지 말 것**

### 3단계: 솔루션 리뷰

사용자가 `/gf-learn check <file>`을 실행하면:

1. `gateflow:sv-tutor` 에이전트를 스폰해 리뷰
2. 답을 알려주지 않고 피드백 제시
3. 솔루션이 통과하면 다음 연습 제안

### 4단계: 힌트 (요청 시)

사용자가 `/gf-learn hint`를 실행하면:

1. 점진적 힌트 제공 (힌트 1은 막연, 힌트 3은 구체적)
2. 연습당 힌트 개수 추적
3. 힌트에서 완전한 솔루션을 절대 주지 않음

## 연습 템플릿

### 초급: 4비트 카운터
```
Create a 4-bit counter with:
- Synchronous reset
- Enable signal
- Wrap-around at max value
```

### 중급: 동기 FIFO
```
Create a synchronous FIFO with:
- Parameterized WIDTH and DEPTH
- Full and empty flags
- No overflow/underflow
```

### 고급: AXI-Lite 슬레이브
```
Create an AXI-Lite slave with:
- 4 read/write registers
- Proper handshaking
- Address decoding
```

## 핵심 규칙

1. 연습을 제시한 후 **항상 사용자를 대기**
2. `/gf-learn solution`으로 명시적으로 요청받지 않는 한 **솔루션을 절대 보이지 말 것**
3. `.gateflow/learn/progress.json`에 **진행 상황 추적**
4. **격려하라** - 학습은 어렵다, 지지해 주라

## 난이도 스케일링

| 레벨 | 점수 범위 | 기준 |
|---|---|---|
| Beginner | 0-99 | 단일 always 블록, 기본 신호 |
| Intermediate | 100-299 | 다중 블록, FSM, 파라미터화 |
| Advanced | 300-499 | 다중 클럭, 프로토콜, 최적화 |
| Expert | 500+ | 전체 서브시스템, 교차 관심사 |

승급: 힌트 없음 +30, 힌트 1개 +20, 힌트 2개 이상 +10, lint 클린 보너스 +10, 솔루션 공개 -10.

## 채점 루브릭

| 등급 | 기준 |
|---|---|
| A (우수) | lint 클린, 올바른 리셋, 파라미터화, 어서션 포함 |
| B (양호) | 정확, 사소한 lint 경고, 합리적 명명 |
| C (허용) | 기본 케이스는 정확, 여러 경고, 하드코딩 값 |
| D (개선 필요) | 기능 오류, 리셋 누락, 폭 불일치 |

자동 검사: lint (20%), 기능 정확성 (40%), 스타일 (15%), 파라미터화 (10%), 엣지 케이스 (10%), 어서션 (5%).

## 진행 상황 지속

`.gateflow/learn/progress.json`에 저장:
```json
{"user_level": "intermediate", "total_score": 185, "topics": {"basics": {"level": "intermediate", "score": 90, "exercises_completed": 3}}}
```

## 챌린지 모드

`/gf-learn challenge <topic>` -- 채점이 있는 시간 제한 연습.

| 난이도 | 시간 제한 | 보너스 임계값 |
|---|---|---|
| Beginner | 15 min | 8 min (2x points) |
| Intermediate | 25 min | 15 min (2x points) |
| Advanced | 40 min | 25 min (2x points) |
