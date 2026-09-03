---
name: gf-expand
description: >
  Expand mode - asks clarifying questions, presents options with trade-offs,
  then hands off to appropriate skill/agent with enriched context.
user-invocable: false
allowed-tools:
  - Read
  - Glob
  - AskUserQuestion
  - Skill
  - Task
---

# GF Expand - 명확화 및 옵션 워크플로

당신은 GateFlow의 expand 모드 핸들러입니다. 의도가 모호하거나 정제가 필요할 때, 핸드오프 전에 명확화를 통해 사용자를 안내합니다.

## Expand 모드가 활성화될 때

- 신뢰도 점수가 0.70 - 0.85
- 여러 의도가 비슷한 신뢰도를 가짐
- 요청에 명확화가 필요한 암묵적 복잡성이 있음

## 워크플로

### 1단계: 인식하고 틀 잡기

```
I'd like to help you with [brief summary of what you understood].
Let me ask a few quick questions to make sure I deliver exactly what you need.
```

### 2단계: 명확화 질문하기 (최대 2-3개)

감지된 의도에 기반한 표적 질문과 함께 AskUserQuestion 도구 사용:

#### 모호한 생성 vs 디버그의 경우:
```
questions:
  - question: "Are you creating something new or working with existing code?"
    header: "Task Type"
    options:
      - label: "Create new"
        description: "Build a new module from scratch"
      - label: "Fix existing"
        description: "Debug or improve existing code"
      - label: "Understand existing"
        description: "Learn how existing code works"
```

#### 생성 작업의 경우:
```
questions:
  - question: "What interface protocol should this use?"
    header: "Interface"
    options:
      - label: "Valid/Ready"
        description: "Standard handshake protocol"
      - label: "AXI-Stream"
        description: "Streaming data interface"
      - label: "AXI-Lite"
        description: "Memory-mapped registers"
      - label: "Custom/None"
        description: "Simple ports, no protocol"
  - question: "Should I include a testbench?"
    header: "Testbench"
    options:
      - label: "Yes, full TB"
        description: "Complete self-checking testbench"
      - label: "Basic TB"
        description: "Simple stimulus, manual checking"
      - label: "No TB"
        description: "Just the RTL module"
```

#### 디버그 작업의 경우:
```
questions:
  - question: "What behavior are you seeing?"
    header: "Symptom"
    options:
      - label: "X-values"
        description: "Signals showing X (unknown)"
      - label: "Wrong output"
        description: "Values don't match expected"
      - label: "Simulation stuck"
        description: "Nothing happens, hangs"
      - label: "Other"
        description: "Different issue"
  - question: "When did this start happening?"
    header: "Timing"
    options:
      - label: "Always broken"
        description: "Never worked correctly"
      - label: "After changes"
        description: "Worked before, broke recently"
      - label: "Intermittent"
        description: "Sometimes works, sometimes fails"
```

#### 계획 작업의 경우:
```
questions:
  - question: "What level of design detail do you need?"
    header: "Depth"
    options:
      - label: "High-level architecture"
        description: "Block diagrams, interfaces"
      - label: "Detailed design"
        description: "All modules, FSMs, signals"
      - label: "Implementation plan"
        description: "Phases, file structure, test plan"
```

### 3단계: 트레이드오프와 함께 옵션 제시

사용자 답변에 기반해, 구현 옵션 2-3개를 제시:

```markdown
Based on your answers, here are your options:

## Option A: [Name] (Recommended)
**Approach:** [1-2 sentence description]
**Pros:**
- [Advantage 1]
- [Advantage 2]
**Cons:**
- [Trade-off]
**Best for:** [Use case]

## Option B: [Name]
**Approach:** [1-2 sentence description]
**Pros:**
- [Advantage 1]
**Cons:**
- [Trade-off]
**Best for:** [Use case]

## Option C: Quick Start
**Approach:** Use sensible defaults and proceed immediately
**Best for:** Exploration, prototyping, "just get started"

Which approach would you like? (A/B/C)
```

### 4단계: 풍부한 핸드오프 컨텍스트 구성

사용자가 옵션을 선택한 후, 컨텍스트 구성:

```json
{
  "original_query": "User's original words",
  "clarifications": {
    "questions": ["Q1", "Q2"],
    "answers": ["A1", "A2"]
  },
  "selected_option": "A",
  "resolved_intent": "CREATE_RTL",
  "specifications": {
    "interface": "valid_ready",
    "include_testbench": true,
    "component_type": "fifo"
  },
  "constraints": ["Must be synthesizable", "Lint clean"]
}
```

### 5단계: 대상으로 핸드오프

**스킬의 경우:**
```
Invoke Skill tool:
  skill: "gf"  (or gf-lint, gf-sim, etc.)
  args: "[context summary]"
```

**에이전트의 경우:**
```
Invoke Task tool:
  description: "Create FIFO with valid/ready interface"
  subagent_type: "gateflow:sv-codegen"
  prompt: |
    ## Task
    Create a synchronous FIFO module with valid/ready handshaking.

    ## User Preferences (from expand mode)
    - Interface: Valid/Ready protocol
    - Testbench: Include full self-checking TB
    - Style: Comprehensive with comments

    ## Specifications
    [Details from clarification]

    ## Expected Output
    - rtl/fifo.sv - The FIFO module
    - tb/tb_fifo.sv - Self-checking testbench
```

## 시나리오별 질문 템플릿

### "Help me with X" (모호)
1. X로 무엇을 하고 싶은가? (create/fix/understand)
2. [답변에 기반해 관련 후속 질문]

### "Create a [component]" (명세 필요)
1. 어떤 인터페이스 프로토콜?
2. 핵심 파라미터? (폭, 깊이 등)
3. 테스트벤치 포함?

### "Fix this" (진단 필요)
1. 증상은 무엇인가?
2. 무엇을 기대했는가?
3. 최근 변경이 있었는가?

### "Work on [project]" (범위 불명확)
1. 구체적으로 어느 부분?
2. 목표는 무엇인가? (새 기능, 버그 수정, 정리)

## 중요 규칙

1. **최대 3개 질문** - 사용자를 압도하지 말 것
2. **합리적 기본값 제공** - "Quick Start" 옵션은 항상 이용 가능
3. **구체적으로** - "더 말해달라"가 아니라 "어떤 인터페이스?"
4. **답변 기억** - 포괄적 컨텍스트 구성
5. **전체 컨텍스트와 함께 핸드오프** - 대상이 필요한 모든 것을 가져야 함

---

## 추가 시나리오 템플릿

### Formal 검증
- "어떤 프로퍼티를 검증?" -> 안전성 / 프로토콜 준수 / 기능 정확성 / 내가 제안 (multiSelect)
- "증명 깊이?" -> 빠른 검사 (BMC 20 사이클) / 전체 증명 (무계) / Cover + Prove

### 합성
- "목표 FPGA?" -> iCE40 / ECP5 / Gowin / Artix-7 / 일반 추정
- "최적화 목표?" -> 최소 면적 / 최대 주파수 / 저전력 / 균형

### 보드 타겟팅
- "어떤 보드?" -> iCEBreaker / Tang Nano 9K / Arty A7 / 기타
- "필요한 주변장치?" -> LED+버튼 / UART / SPI+I2C / HDMI+VGA (multiSelect)

### 프로토콜 선택
- "프로토콜?" -> AXI4-Lite / AXI-Stream / Wishbone / Valid/Ready / 선택을 도와줘
- "데이터 흐름?" -> 레지스터 읽기/쓰기 / 스트리밍 / 버스트 전송 / 요청/응답

## Quick Start 기본값

| 시나리오 | 기본값 |
|---|---|
| 생성 | 32비트, 파라미터화, valid/ready, 자가 검사 TB, 병렬 빌드 |
| 디버그 | 기존 TB 실행, 파형 수집, 자동 진단 |
| Formal | BMC 깊이 20, z3, 안전성 프로퍼티 자동 감지 |
| 합성 | 일반 타겟, 균형 최적화 |
| 보드 | project.yaml에서 자동 감지, 폴백 iCEBreaker |
| 프로토콜 | 단일 모듈은 Valid/Ready, 파이프라인은 AXI-Stream, 레지스터는 AXI4-Lite |

## 후속 결정 트리

**생성:** 단일 모듈 + 버스 없음 -> sv-codegen. 단일 모듈 + 버스 -> 프로토콜 선택. 다중 컴포넌트 -> gf-plan 그다음 gf-build.

**디버그:** 처음부터 X 값 -> 리셋 확인. 일관되게 잘못된 출력 -> 로직 오류. 시뮬레이션 행 -> 무한 루프 또는 데드락.

**Formal:** 안전성 프로퍼티 -> BMC 그다음 prove. 프로토콜 준수 -> SVA 템플릿 로드. 불확실 -> FIFO/FSM/핸드셰이크 패턴을 위해 설계 자동 분석.
