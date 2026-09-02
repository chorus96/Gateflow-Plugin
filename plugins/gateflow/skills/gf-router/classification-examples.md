# 의도 분류 예시

이 파일은 의미론적 의도 분류를 위한 few-shot 예시를 담고 있습니다.
사용자 요청을 어떻게 분류할지 이해하는 데 사용하세요.

---

## 높은 신뢰도 예시 (>= 0.85)

### CREATE_RTL 예시
```
Query: "I need a 4-stage pipeline register with valid/ready"
Intent: CREATE_RTL | Confidence: 0.95
Target: gateflow:sv-codegen
Reasoning: Explicit request for specific RTL component with clear specs

Query: "Build me an arbiter for 4 masters"
Intent: CREATE_RTL | Confidence: 0.92
Target: gateflow:sv-codegen
Reasoning: "Build me" + specific component type

Query: "Can you make a synchronous FIFO?"
Intent: CREATE_RTL | Confidence: 0.90
Target: gateflow:sv-codegen
Reasoning: "Make" implies creation, FIFO is specific component

Query: "I want a state machine that handles USB enumeration"
Intent: CREATE_RTL | Confidence: 0.93
Target: gateflow:sv-codegen
Reasoning: "I want" + FSM with specific function
```

### DEBUG 예시
```
Query: "My simulation is stuck, nothing happens after reset"
Intent: DEBUG | Confidence: 0.94
Target: gateflow:sv-debug
Reasoning: Describes failure symptom (stuck), simulation issue

Query: "The output is always X, I don't understand why"
Intent: DEBUG | Confidence: 0.92
Target: gateflow:sv-debug
Reasoning: X-value problem, needs diagnosis

Query: "This used to work but now it fails"
Intent: DEBUG | Confidence: 0.88
Target: gateflow:sv-debug
Reasoning: Regression, something broke

Query: "Why does my counter wrap at 255 instead of 1000?"
Intent: DEBUG | Confidence: 0.90
Target: gateflow:sv-debug
Reasoning: Unexpected behavior, needs root cause analysis
```

### CREATE_TB 예시
```
Query: "Write a testbench for uart_tx.sv"
Intent: CREATE_TB | Confidence: 0.95
Target: gateflow:sv-testbench
Reasoning: Explicit testbench request with target file

Query: "I need tests for this module"
Intent: CREATE_TB | Confidence: 0.88
Target: gateflow:sv-testbench
Reasoning: "Tests" implies testbench creation

Query: "Create stimulus for the memory controller"
Intent: CREATE_TB | Confidence: 0.90
Target: gateflow:sv-testbench
Reasoning: Stimulus is testbench component
```

### VERIFY 예시
```
Query: "Add assertions to verify the AXI protocol"
Intent: VERIFY | Confidence: 0.93
Target: gateflow:sv-verification
Reasoning: Explicit assertion request for protocol

Query: "I need SVA properties for this handshake"
Intent: VERIFY | Confidence: 0.95
Target: gateflow:sv-verification
Reasoning: SVA = SystemVerilog Assertions

Query: "Check that the FIFO never overflows"
Intent: VERIFY | Confidence: 0.87
Target: gateflow:sv-verification
Reasoning: Property to verify, not debug
```

### EXPLAIN 예시
```
Query: "What does the state machine in uart_tx.sv do?"
Intent: EXPLAIN | Confidence: 0.94
Target: gateflow:sv-understanding
Reasoning: "What does X do" = understanding request

Query: "Explain how the arbitration logic works"
Intent: EXPLAIN | Confidence: 0.93
Target: gateflow:sv-understanding
Reasoning: Explicit "explain" for understanding

Query: "I'm confused about this module's interface"
Intent: EXPLAIN | Confidence: 0.85
Target: gateflow:sv-understanding
Reasoning: Confusion = needs explanation

Query: "Walk me through this code"
Intent: EXPLAIN | Confidence: 0.91
Target: gateflow:sv-understanding
Reasoning: "Walk through" = explanation request
```

### REFACTOR 예시
```
Query: "This code has too many lint warnings, clean it up"
Intent: REFACTOR | Confidence: 0.92
Target: gateflow:sv-refactor
Reasoning: Lint warnings + cleanup request

Query: "Make this code more readable"
Intent: REFACTOR | Confidence: 0.88
Target: gateflow:sv-refactor
Reasoning: Readability improvement = refactor

Query: "Fix the coding style issues"
Intent: REFACTOR | Confidence: 0.90
Target: gateflow:sv-refactor
Reasoning: Style issues = refactor task

Query: "Optimize this for better timing"
Intent: REFACTOR | Confidence: 0.86
Target: gateflow:sv-refactor
Reasoning: Optimization = refactoring
```

### ORCHESTRATE 예시
```
Query: "Create a FIFO and make sure it works"
Intent: ORCHESTRATE | Confidence: 0.92
Target: gf
Reasoning: Create + verify = orchestration needed

Query: "Build and test a counter module"
Intent: ORCHESTRATE | Confidence: 0.93
Target: gf
Reasoning: Build + test = end-to-end workflow

Query: "I need a working UART with testbench"
Intent: ORCHESTRATE | Confidence: 0.90
Target: gf
Reasoning: "Working" implies verification, multiple outputs
```

### PLAN 예시
```
Query: "How should I design a DMA controller?"
Intent: PLAN | Confidence: 0.91
Target: gateflow:sv-planner
Reasoning: "How should I design" = architecture question

Query: "Plan out a cache subsystem for me"
Intent: PLAN | Confidence: 0.94
Target: gateflow:sv-planner
Reasoning: Explicit "plan out" request

Query: "What's the best architecture for a crossbar?"
Intent: PLAN | Confidence: 0.87
Target: gateflow:sv-planner
Reasoning: Architecture question before implementation
```

### LEARN 예시
```
Query: "I want to practice writing FSMs"
Intent: LEARN | Confidence: 0.95
Target: gf-learn
Reasoning: Explicit "practice" request

Query: "Give me some exercises for pipelining"
Intent: LEARN | Confidence: 0.93
Target: gf-learn
Reasoning: Exercises = learning mode

Query: "I'm learning SystemVerilog, can you help?"
Intent: LEARN | Confidence: 0.88
Target: gf-learn
Reasoning: Learning context
```

---

## 중간 신뢰도 예시 (0.70-0.85) - Expand 모드 트리거

```
Query: "Help me with the FIFO"
Intent: AMBIGUOUS | Confidence: 0.50
Possible: CREATE_RTL, DEBUG, REFACTOR, EXPLAIN
Action: EXPAND MODE
Questions:
  1. "Do you want to create a new FIFO or work with existing code?"
  2. "Is there a specific problem you're trying to solve?"

Query: "Fix this module"
Intent: AMBIGUOUS | Confidence: 0.65
Possible: DEBUG, REFACTOR
Action: EXPAND MODE
Questions:
  1. "Is there a runtime issue (simulation fails) or code quality issue (lint warnings)?"
  2. "What behavior are you seeing vs expecting?"

Query: "Work on the memory interface"
Intent: AMBIGUOUS | Confidence: 0.55
Possible: CREATE_RTL, REFACTOR, DEBUG, EXPLAIN
Action: EXPAND MODE
Questions:
  1. "Are you creating a new interface or modifying existing code?"
  2. "What's the goal - new feature, bug fix, or understanding?"

Query: "Test this"
Intent: AMBIGUOUS | Confidence: 0.70
Possible: CREATE_TB, SIMULATE
Action: EXPAND MODE
Questions:
  1. "Do you want to create a testbench or run existing tests?"
  2. "Which file/module needs testing?"

Query: "Check this code"
Intent: AMBIGUOUS | Confidence: 0.60
Possible: LINT, VERIFY, EXPLAIN, REFACTOR
Action: EXPAND MODE
Questions:
  1. "What kind of check - lint, functional verification, or code review?"
```

---

## 낮은 신뢰도 예시 (< 0.70) - 명확화 요청

```
Query: "Help"
Intent: UNCLEAR | Confidence: 0.20
Action: CLARIFY
Response: "I'd be happy to help! Could you tell me more about what you're working on? For example:
- Are you creating new RTL code?
- Debugging an issue?
- Trying to understand existing code?"

Query: "FIFO"
Intent: UNCLEAR | Confidence: 0.25
Action: CLARIFY
Response: "I see you're interested in FIFOs. What would you like to do?
- Create a new FIFO module
- Debug an existing FIFO
- Understand how a FIFO works"

Query: "This is wrong"
Intent: UNCLEAR | Confidence: 0.40
Action: CLARIFY
Response: "I'd like to help fix the issue. Could you tell me:
- Which file has the problem?
- What behavior are you seeing?"
```

---

## 컨텍스트 의존 예시

### 코드가 방금 생성된 후:
```
Query: "Test it"
Intent: SIMULATE (not CREATE_TB) | Confidence: 0.85
Reasoning: Testbench likely exists, user wants to run sim
Target: gf-sim
```

### 학습 모드에서:
```
Query: "Check my solution"
Intent: TUTOR | Confidence: 0.90
Reasoning: Learning context, wants review
Target: gateflow:sv-tutor
```

### 시뮬레이션이 실패한 후:
```
Query: "Fix it"
Intent: DEBUG | Confidence: 0.88
Reasoning: Recent failure context
Target: gateflow:sv-debug
```

---

## 모호성 해소 신호

### CREATE_RTL을 가리키는 신호:
- "new", "create", "build", "make", "generate", "need a", "want a"
- 특정 컴포넌트 이름 (FIFO, FSM, arbiter, counter)
- 인터페이스 명세 (폭, 깊이, 프로토콜)

### DEBUG를 가리키는 신호:
- 문제 설명 (stuck, wrong, fails, broken, X-value)
- "used to work", "suddenly", "after I changed"
- 오류 메시지, 예상치 못한 동작

### REFACTOR를 가리키는 신호:
- "clean", "lint", "style", "optimize", "readable"
- "warnings", "improve", "better"

### EXPLAIN을 가리키는 신호:
- "what does", "how does", "explain", "understand"
- "walk through", "confused", "why is this"

### ORCHESTRATE를 가리키는 신호:
- 여러 동작: "create and test", "build and verify"
- "working" (검증을 함의)
- "end to end", "complete"
