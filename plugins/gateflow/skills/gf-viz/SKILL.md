---
name: gf-viz
description: >
  Terminal visualization for GateFlow codebase maps. Renders module hierarchies,
  FSM state diagrams, and module detail cards as interactive ASCII/Unicode art.
  Example requests: "visualize the codebase", "show hierarchy", "show FSM", "show module detail"
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

# GF-Viz: 터미널 시각화

`.gateflow/map/` 데이터를 터미널에서 대화형 ASCII/Unicode 다이어그램으로 렌더링합니다.

## 사전 요구 사항

코드베이스 맵 확인:

```bash
ls .gateflow/map/CODEBASE.md 2>/dev/null
```

- **맵 존재:** 렌더링으로 진행
- **맵 없음:** 사용자에게 안내: "No codebase map found. Run `/gf-map` first to generate one."

## 진입점

호출되면, **개요 대시보드**를 렌더링하고 내비게이션 메뉴를 제시.

인자와 함께 호출되면(예: `/gf-viz uart_tx`), 해당 모듈의 **모듈 상세 카드**로 바로 이동.

---

## 뷰 1: 개요 대시보드

**데이터 소스:** `CODEBASE.md` (통계, 모듈 인덱스), `hierarchy.md` (트리), `fsm.md` (FSM 목록), `clock-domains.md` (클럭/CDC)

이 파일들을 읽고, 데이터를 추출한 뒤, 렌더링:

```
╔══ CODEBASE: <project_name> ═══════════════════════════════╗
║                                                            ║
║  ● N modules  ● N packages  ● N FSMs  ● N interfaces       ║
║  ● N clocks   ● N CDC       ● N ports   ● N warnings       ║
║                                                            ║
╠══ HIERARCHY (compact) ════════════════════════════════════╣
║                                                            ║
║  ◆ <top> ──┬── <child1> ──┬── <grandchild1>               ║
║            │              └── <grandchild2>                ║
║            └── <child2> ── <grandchild3>                   ║
║                                                            ║
╠══ FSMs ═══════════════════════════════════════════════════╣
║  ↻ <fsm_name> (<module>)  N states: S1→S2→S3→S4           ║
║  ↻ <fsm_name> (<module>)  N states: S1→S2→S3              ║
║                                                            ║
╠══ HEALTH ═════════════════════════════════════════════════╣
║  <✓ or ⚠> lint status  <✓ or ⚠> undriven  <✓ or ⚠> CDC   ║
║                                                            ║
╠═══════════════════════════════════════════════════════════╣
║  [1] Hierarchy  [2] FSMs  [3] Module detail                ║
║  Or ask anything: "show uart_tx", "trace data path"        ║
╚═══════════════════════════════════════════════════════════╝
```

**규칙:**
- 대시보드에서 계층은 **최대 2레벨**로 평탄화
- FSM은 상태 체인과 함께 **한 줄 요약**으로 표시
- 통계는 CODEBASE.md 프론트매터와 모듈 인덱스에서 가져옴
- 헬스는 warnings 섹션에서 가져옴
- 섹션에 데이터가 없으면 생략하지 말고 "none detected" 표시

---

## 뷰 2: 계층 탐색기

**데이터 소스:** `hierarchy.md`, `modules/*.md`

```
══ MODULE HIERARCHY ════════════════════════════════════════

◆ <top_module>                                       TOP
├── ■ <inst> : <module> [PARAM=VAL]                   MID
│   ├── ■ <inst> : <module>                           MID
│   │   ├── □ <inst> : <module>                       LEAF
│   │   └── □ <inst> : <module>                       LEAF
│   └── □ <inst> : <module> [W=32, D=16]              LEAF
└── ■ <inst> : <module>                               MID
    └── □ <inst> : <module>                           LEAF

── STATS ──────────────────────────────────────────────
  Total: N modules │ Max depth: N │ Leaf count: N

── INSTANCE TABLE ─────────────────────────────────────
  ┃ Parent       │ Instance   │ Module          │ Params       ┃
  ┃ ...          │ ...        │ ...             │ ...          ┃

═══════════════════════════════════════════════════════════
  [H] Home  [2] FSMs  [3] Module detail: <name>
  Or: "show <module>", "which modules use <module>?"
```

**모듈 타입 배지:**
- `◆` **TOP** - 굵게, 톱레벨 모듈 (다른 것에 의해 인스턴스화되지 않음)
- `■` **MID** - 표준 두께, 자식 있음
- `□` **LEAF** - 가벼운 두께, 자식 없음

**깊이 단서:** 더 깊은 모듈은 더 가벼운 시각 두께로 렌더링. 톱은 두드러지고 리프는 흐려짐.

**동작:**
- 모든 깊이 레벨의 전체 트리 표시
- 파라미터는 `[PARAM=VAL]`로 인라인 표시
- 인스턴스 표는 모든 부모→자식 관계를 표시
- "show <module>"는 해당 모듈을 루트로 하는 트리를 재렌더링
- "which modules use <module>?"는 인스턴스 표를 검색하여 부모를 나열

---

## 뷰 3: FSM 뷰어

**데이터 소스:** `fsm.md`, 모듈별 페이지

**여러 FSM이 존재하면, 선택기를 먼저 표시:**

```
══ STATE MACHINES ══════════════════════════════════════════

  [1] ↻ <fsm_name>  (<module>)   N states
  [2] ↻ <fsm_name>  (<module>)   N states
  [3] ↻ <fsm_name>  (<module>)   N states

  Pick a number, or: "show <fsm_name>"
```

**단일 FSM 렌더링:**

```
══ FSM: <fsm_name> ═════════════════════════════════════════
   Module: <module> │ Encoding: N-bit │ Reset: → <reset_state>

                    <condition>
   ┌──────┐  ─────────────►  ┌───────┐
   │      │                  │       │
   │  S1  │                  │  S2   │
   │  ◉   │                  │       │
   └──────┘                  └───┬───┘
      ▲                          │ <condition>
      │                          ▼
   ┌──────┐                  ┌───────┐
   │      │  ◄───────────   │       │──┐
   │  S4  │   <condition>   │  S3   │  │ <self-loop cond>
   │      │                  │       │◄─┘
   └──────┘                  └───────┘

── TRANSITIONS ────────────────────────────────────────────
  ┃ From  │ To    │ Condition     │ Output          ┃
  ┃ S1    │ S2    │ ...           │ ...             ┃
  ┃ S2    │ S3    │ ...           │ ...             ┃
  ┃ ...   │ ...   │ ...           │ ...             ┃

── STATE DETAILS ──────────────────────────────────────────
  ◉ S1   Reset state. <description>
    S2   <description>
    S3   <description>
    S4   <description>

═══════════════════════════════════════════════════════════
  [H] Home  [1] Hierarchy  [3] Module: <parent_module>
  Or: "show another FSM", "explain the S2→S3 transition"
```

**FSM 박스 다이어그램의 레이아웃 규칙:**
- **2-4 상태:** 일렬 또는 L자 형태로 배치
- **4-6 상태:** 2x2 또는 2x3 그리드로 배치
- **7개 이상 상태:** 전이 표만 사용 (ASCII 박스에는 너무 복잡)
- 리셋 상태는 항상 `◉`로 표시
- 셀프 루프는 같은 박스로 돌아가는 `──┐` / `◄─┘`로 표시
- 전이 화살표는 조건 라벨과 함께 `──►` 사용

**동작:**
- "explain <from>→<to> transition"은 RTL 소스를 사용한 분석을 트리거
- "show module"은 부모 모듈의 상세 카드로 상호 링크
- 맵 데이터가 포함하면 전이 표의 Output 열을 채움

---

## 뷰 4: 모듈 상세 카드

**데이터 소스:** `modules/<module_name>.md` (주), `hierarchy.md`, `fsm.md`, `signals.md`

```
╔══════════════════════════════════════════════════════════╗
║  <module_name>                              <TYPE_BADGE> ║
║  <file_path>:<line_range>                                ║
╠══ PARAMETERS ════════════════════════════════════════════╣
║  ┃ Name     │ Type │ Default     │ Description       ┃  ║
║  ┃ ...      │ ...  │ ...         │ ...               ┃  ║
╠══ PORTS ═════════════════════════════════════════════════╣
║  → <name>     input   <width>   <description>           ║
║  → <name>     input   <width>   <description>           ║
║  ← <name>     output  <width>   <description>           ║
║  ← <name>     output  <width>   <description>           ║
╠══ INTERNALS ═════════════════════════════════════════════╣
║                                                          ║
║  Clock : <clock_name> (<domain info>)                    ║
║  Reset : <reset_name> (<type>)                           ║
║  FSM   : ↻ <fsm_name> → <state_list>                    ║
║  Inst  : <instance_count> (<list or "none (leaf)")>      ║
║                                                          ║
╠══ CONNECTIONS ═══════════════════════════════════════════╣
║                                                          ║
║  Instantiated by:                                        ║
║    ■ <parent_module> as <instance_name>                  ║
║      .<port>(<signal>) .<port>(<signal>)                 ║
║      .<port>(<signal>) .<port>(<signal>)                 ║
║                                                          ║
╠══ HEALTH ════════════════════════════════════════════════╣
║  <✓ or ⚠> port connection status                         ║
║  <✓ or ⚠> lint status                                    ║
║  <✓ or ⚠> assertion coverage                             ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
  [H] Home  [1] Hierarchy  [2] FSM: <fsm_name>
  [↑] Parent: <parent_module>
  Or: "show ports", "explain the handshake", "add assertions"
```

**포트 방향 기호:**
- `→` 입력
- `←` 출력
- `↔` 양방향 (inout)

**타입 배지:** `TOP`, `MID`, `LEAF`

**동작:**
- Connections 섹션은 부모 인스턴스화의 **실제 신호 바인딩**을 표시
- 모듈이 여러 번 인스턴스화되면, 각 인스턴스를 표시
- FSM 줄은 FSM 뷰어로 상호 링크
- 부모 이름은 부모의 상세 카드로 상호 링크
- "add assertions"는 `sv-verification` 에이전트로 핸드오프 가능
- "explain the handshake"는 RTL 소스를 읽어 프로토콜을 추론

---

## 색상/강조 어휘

모든 뷰에서 일관되게 적용:

| 요소 | 기호 | 스타일 |
|---------|--------|-------|
| Top module | `◆` | **굵게** |
| Mid module | `■` | 표준 |
| Leaf module | `□` | 가볍게 |
| Input port | `→` | 초록 강조 |
| Output port | `←` | 노랑 강조 |
| Bidir port | `↔` | 청록 강조 |
| FSM indicator | `↻` | 표준 |
| Reset state | `◉` | **굵게/강조** |
| Clean/pass | `✓` | 초록 |
| Warning | `⚠` | 노랑/호박색 |
| Info/stat | `●` | 표준 |
| Transition | `──►` | 표준 |

**계층의 깊이 단서:** 톱레벨 굵게, 미드 표준, 리프 흐리게.

---

## 상호작용 모델

### 메뉴 내비게이션

매 렌더 후, 번호 옵션이 있는 내비게이션 푸터 표시:
- `[H]` Home - 대시보드로 복귀
- `[1]` `[2]` `[3]` - 뷰 간 전환
- `[↑]` Parent - 계층에서 위로 이동 (상세 카드에서만)

### 자유 형식 질의

메뉴와 함께 항상 자연어를 수용:
- "show <module_name>" → 모듈 상세 카드
- "show <fsm_name>" → 해당 FSM의 FSM 뷰어
- "which modules use <module>?" → 필터링된 계층
- "trace <signal> from <module_a> to <module_b>" → 신호 경로 분석
- "explain <aspect>" → RTL 소스를 읽어 추론
- "add assertions to <module>" → sv-verification 에이전트로 핸드오프
- "back" → 이전 뷰
- "home" → 대시보드

### 에이전트 핸드오프

시각화를 넘어서는 질의의 경우:
- "explain" / "why" → Task 도구로 `sv-understanding` 에이전트 스폰
- "add assertions" → Task 도구로 `sv-verification` 에이전트 스폰
- "fix" / "refactor" → Task 도구로 `sv-refactor` 에이전트 스폰

핸드오프할 때, 에이전트가 전체 컨텍스트를 갖도록 현재 시각화 컨텍스트(어느 모듈, 어느 뷰)를 전달.

---

## /gf-map 이후 자동 트리거

`gf-architect`의 마지막 단계로 사용될 때, 간결한 요약으로 개요 대시보드(뷰 1)만 렌더링. 전체 대화형 메뉴는 표시하지 말고 - 대시보드와 함께 참고만:

```
Run /gf-viz to explore interactively.
```

---

## 데이터 추출

### CODEBASE.md 읽기

프론트매터에서 추출:
- `total_files`, `total_tokens`, `commit`, `last_mapped`

모듈 인덱스 표에서 추출:
- 모듈 이름, 타입, 파일, 포트 요약

Warnings 섹션에서 추출:
- lint 경고, 미구동 신호, CDC 문제

### hierarchy.md 읽기

Mermaid 플로차트에서 추출:
- 부모→자식 관계
- 인스턴스 이름

인스턴스 표에서 추출:
- 전체 부모, 인스턴스, 모듈, 파라미터 데이터

### fsm.md 읽기

각 FSM에 대해 추출:
- FSM 이름, 부모 모듈
- 인코딩이 있는 상태 목록
- 전이 표 (from, to, condition, output)
- 리셋 상태

### modules/*.md 읽기

모듈별로 추출:
- 파라미터 표
- 포트 표 (이름, 방향, 폭, 설명)
- 클럭/리셋 정보
- 인스턴스 목록
- 어서션/커버리지 정보

---

## 엣지 케이스

- **빈 맵:** "No codebase map found. Run `/gf-map` first."
- **FSM 미감지:** FSM 섹션에 "No state machines detected in this codebase." 표시
- **단일 모듈:** 계층 뷰가 하나의 모듈만 표시. 인스턴스 표 건너뜀.
- **모듈을 찾을 수 없음:** "Module '<name>' not found in map. Available modules: <list>"
- **매우 깊은 계층 (>6레벨):** 전체 트리를 렌더링하되 참고: "Deep hierarchy detected. Use 'show <module>' to focus on a subtree."
- **매우 넓은 계층 (형제 >10):** 처음 8개를 표시한 뒤 "... and N more. Use 'show <parent>' to see all."

---

## 뷰 5: 신호 경로 추적

트리거: "trace data_in from top to digest_out"

모듈 경계를 넘나드는 신호 경로를 ASCII로 렌더링 - 모듈에는 박스, 신호에는 화살표, 레지스터된 경계(파이프라인 스테이지)에는 `◈` 마커. 홉 수와 파이프라인 스테이지 요약을 표시.

## 뷰 6: 타이밍 다이어그램

트리거: "timing uart_tx" 또는 "timing fsm tx_state"

ASCII 파형: 클럭에 `┌─┐└─┘`, high에 `───`, low에 `___`, 버스/enum 값에 `╡val╞`. FSM 데이터나 알려진 프로토콜 패턴에서 자동 생성. 커스텀 다이어그램용 WaveJSON 입력 수용.

## 뷰 7: Diff 뷰

트리거: "diff" 또는 "what changed"

맵 스냅샷 간 구조 변화 표시: `+ ADDED`, `~ MODIFIED` (구체적 변경: port/instance/FSM/parameter 포함), `- REMOVED`. `.gateflow/map/.prev_*`에 이전 스냅샷 필요.

## 뷰 8: 포트 연결 매트릭스

트리거: "matrix uart_ctrl" 또는 "connections"

어느 부모 신호가 어느 인스턴스 포트에 연결되는지 보여주는 표. 하단 행에 연결됨/미연결 개수 표시. 별도 섹션에 모든 미연결 포트를 `⚠` 경고와 함께 나열. 큰 설계용 간결한 점 매트릭스 변형: `●` 연결됨, `○` 미연결.

## 검색

트리거: "find modules with FSM", "find signals named *_valid"

| 질의 | 찾는 것 |
|---|---|
| `find modules with <clock>` | 특정 클럭을 쓰는 모듈 |
| `find modules with fsm` | FSM을 포함하는 모든 모듈 |
| `find modules with >20 ports` | 큰 인터페이스 모듈 |
| `find signals named <glob>` | 신호 이름 패턴 일치 |
| `find instances of <module>` | 모든 인스턴스화 |
| `find unconnected ports` | 플로팅 포트 |
| `find cdc crossings` | 클럭 도메인 크로싱 |
