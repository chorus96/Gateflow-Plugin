# Claude Code를 위한 GateFlow 플러그인

> AI 기반 하드웨어 개발 플랫폼 — 자연어로 동작하는 RTL을 설계, 검증, 합성, 배포합니다. 오픈소스 FPGA 툴체인 전반에 걸쳐 SystemVerilog, Verilog, VHDL을 지원합니다.

[![GitHub Stars](https://img.shields.io/github/stars/codejunkie99/Gateflow-Plugin?style=social)](https://github.com/codejunkie99/Gateflow-Plugin/stargazers)
[![Version](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fraw.githubusercontent.com%2Fcodejunkie99%2FGateflow-Plugin%2Fmain%2Fplugins%2Fgateflow%2F.claude-plugin%2Fplugin.json&query=%24.version&label=version&color=blue)](https://github.com/codejunkie99/Gateflow-Plugin/releases)
[![License: BSL-1.1](https://img.shields.io/badge/License-BSL%201.1-orange.svg)](LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-purple.svg)](https://code.claude.com/)

![GateFlow Demo](https://github.com/user-attachments/assets/73115231-5dac-473c-8f6b-5ed88a780135)

![GateFlow v2.3.1 Update](plugins/gateflow/assets/update-2.3.1.svg)

---

**하드웨어를 개발하는 데 진입 장벽이 있어서는 안 됩니다.**

GateFlow는 전문적인 하드웨어 개발 도구를 Claude Code로 가져옵니다. 만들고 싶은 것을 — 목표 보드까지 포함해 — 설명하면, 올바른 핀 할당과 함께 lint 검사, 시뮬레이션, Formal 검증, 합성 준비가 완료된 코드를 얻을 수 있습니다.

첫 `always_ff`를 작성하든, FIFO가 절대 오버플로하지 않음을 정형적으로(formally) 증명하든, Arty A7용 SPI 컨트롤러를 합성하든, 도구는 당신과 싸우는 것이 아니라 당신을 도와야 합니다.

당신이 무엇을 만들지 무척 기대됩니다. ❤️

---

- [빠른 시작](#빠른-시작) — 명령 하나로 설치
- [사용법](#사용법) — 자연어, 스킬, 커맨드
- [특징](#특징) — GateFlow가 특별한 이유
- [구성 요소](#구성-요소) — 모든 스킬, 에이전트, 커맨드
- [개별 다운로드](#개별-다운로드) — 단일 구성 요소 받기
- [크로스 툴 호환성](#크로스-툴-호환성) — Codex, Cursor, Copilot, Cline, Windsurf와 함께 사용
- [설정](#설정) — 프로젝트별 설정
- [프로젝트 구조](#프로젝트-구조) — 저장소 레이아웃
- [문제 해결](#문제-해결) — 흔한 문제들
- [업데이트](#업데이트) — 변경 이력
- [릴리스](releases.md) — 상세 릴리스 노트
- [기여](#기여) | [라이선스](#라이선스) | [링크](#링크)

---

## 빠른 시작

### 설치

```bash
# Option 1: Marketplace (recommended)
claude plugin marketplace add codejunkie99/Gateflow-Plugin
claude plugin install gateflow

# Option 2: Clone and run
git clone https://github.com/codejunkie99/Gateflow-Plugin.git
claude --plugin-dir ./Gateflow-Plugin/plugins/gateflow

# Option 3: Persistent (add to ~/.claude/settings.json)
git clone https://github.com/codejunkie99/Gateflow-Plugin.git ~/.claude-plugins/gateflow-marketplace
```

Option 3의 경우 `~/.claude/settings.json` 또는 `.claude/settings.json`에 다음을 추가하세요:
```json
{
  "plugins": [
    "~/.claude-plugins/gateflow-marketplace/plugins/gateflow"
  ]
}
```

### 사전 요구 사항

| 도구 | 필요 여부 | macOS | Linux |
|------|----------|-------|-------|
| [Claude Code](https://code.claude.com/) | 필수 | 웹사이트 참고 | 웹사이트 참고 |
| [Verilator](https://verilator.org/) | 권장 | `brew install verilator` | `sudo apt install verilator` |
| Verible | 선택 | `brew tap chipsalliance/verible && brew install verible` | [releases](https://github.com/chipsalliance/verible/releases) 참고 |
| [Yosys](https://github.com/YosysHQ/yosys) | 합성용 | `brew install yosys` | `sudo apt install yosys` |
| [SymbiYosys](https://github.com/YosysHQ/sby) | Formal 검증용 | `pip install symbiyosys` | `pip install symbiyosys` |
| [GHDL](https://github.com/ghdl/ghdl) | VHDL용 | `brew install ghdl` | `sudo apt install ghdl` |
| [nextpnr](https://github.com/YosysHQ/nextpnr) | P&R용 | `brew install nextpnr` | GitHub 참고 |
| [openFPGALoader](https://github.com/trabucayre/openFPGALoader) | 플래시용 | `brew install openfpgaloader` | GitHub 참고 |

### 확인 및 업데이트

```bash
# Verify installation
/gf-doctor

# Update (marketplace)
# /plugin → Marketplaces → gateflow → Update → restart Claude Code

# Update (local)
# git pull in your plugin folder, then restart Claude Code
```

---

## 사용법

### 그냥 요청하세요

GateFlow는 맥락을 이해합니다. 필요한 것을 평범한 말로 설명하세요:

```
"Create a FIFO and test it"
 → Generates FIFO, creates testbench, runs simulation, fixes issues, delivers working code

"Formally verify that the FIFO never overflows"
 → Generates SVA properties, configures SymbiYosys, runs proof, reports results

"Synthesize my design for the iCEBreaker board"
 → Runs Yosys synthesis, reports LUT/FF/BRAM usage, generates constraint file

"Add an SPI controller to my project"
 → Installs verified SPI master IP block with testbench and formal proofs

"What pins does the Arty A7 PMOD JA have?"
 → Looks up curated board database, shows pin assignments with I/O standards

"Why is my output X?"
 → Analyzes code, traces signal path, identifies root cause

"Plan a DMA controller"
 → Creates detailed design plan with block diagrams, FSMs, interfaces, verification strategy
```

### 스킬 (자동 활성화)

스킬은 맥락에 따라 자동으로 활성화됩니다:

| 스킬 | 트리거 | 하는 일 |
|-------|---------|--------------|
| `/gf` | 모든 SV 작업 | **메인 오케스트레이터** — 계획 우선, 병렬 빌드, 동작할 때까지 검증 |
| `/gf-plan` | "plan", "design", "architect" | ASCII 다이어그램이 포함된 RTL 구현 계획 |
| `/gf-build` | "build", "multi-component", "SoC" | 병렬 컴포넌트 빌드 오케스트레이션 |
| `/gf-formal` | "formally verify", "prove", "check property" | SymbiYosys를 통한 **자연어 기반 Formal 검증** |
| `/gf-synth` | "synthesize", "area estimate", "resource usage" | LUT/FF/BRAM/DSP 리포트가 포함된 **Yosys 합성** |
| `/gf-architect` | "map codebase", "analyze project" | 계층 구조, FSM, 클럭, CDC를 담은 코드베이스 맵 |
| `/gf-viz` | "visualize", "show hierarchy" | RTL 아키텍처의 터미널 ASCII 시각화 |
| `/gf-learn` | "teach me", "exercise", "practice" | 연습 문제와 피드백이 있는 학습 모드 |

### 커맨드

| 커맨드 | 하는 일 |
|---------|-------------|
| `/gf-lint` | 구조화된 출력으로 Verilator lint 실행 |
| `/gf-fix` | lint 오류 자동 수정 |
| `/gf-sim` | 컴파일 후 시뮬레이션 실행 |
| `/gf-formal` | SymbiYosys 정형 검증 실행 |
| `/gf-gen` | 모듈/테스트벤치 스캐폴드 생성 |
| `/gf-ip` | 검증된 IP 블록 라이브러리 관리 (add/list/info) |
| `/gf-boards` | 지원 FPGA 보드 목록 및 핀아웃 조회 |
| `/gf-demo` | 명령 한 줄로 실행되는 무설정 쇼케이스 프로젝트 |
| `/gf-scan` | 프로젝트 파일 인덱싱 |
| `/gf-map` | 코드베이스 아키텍처 매핑 |
| `/gf-doctor` | 환경 및 의존성 점검 |
| `/gf-tui` | 로컬 GateFlow 터미널 콘솔 열기 |
| `/gf-release` | 플러그인 릴리스 준비 상태 검증 |

### 로컬 CLI

GateFlow는 플러그인 유지 관리와 에이전트 생성을 위한 커맨드 중심 로컬 CLI도 함께 제공합니다:

```bash
python3 tools/gateflow_cli.py status
python3 tools/gateflow_cli.py agents list
python3 tools/gateflow_cli.py agents create "CDC Reviewer" \
  --role "clock-domain crossing reviewer" \
  --description "Reviews synchronizers and CDC constraints"
python3 tools/gateflow_cli.py shell
python3 tools/gateflow_cli.py tui
```

TUI 내부에서는 `a` 키를 눌러 대시보드를 벗어나지 않고 새 에이전트를 만들 수 있습니다.

### 예시 세션

```
$ claude --plugin-dir ./Gateflow-Plugin/plugins/gateflow

You: Create a parameterized counter with enable and test it

Claude: Creating counter module...
✓ Created counter.sv

Running lint check...
✓ Lint clean

Creating testbench...
✓ Created tb_counter.sv

Running simulation...
✓ All tests pass (12 checks)

Done! Created:
- rtl/counter.sv (8-bit parameterized counter with enable)
- tb/tb_counter.sv (Self-checking testbench)
```

---

## 특징

### 그냥 생성된 코드가 아니라, 동작하는 코드
`/gf` 오케스트레이터는 생성에서 멈추지 않고 검증합니다:
```
Create → Lint → Fix → Test → Fix → Formally Verify → Synthesize → Deliver
```

### 자연어 기반 Formal 검증
증명하고 싶은 것을 평범한 말로 GateFlow에 알려주세요:
```
"Formally verify that the FIFO never overflows and the pointers are always consistent"
 → Generates SVA assert property statements
 → Configures SymbiYosys (.sby file)
 → Runs bounded model checking
 → Reports proof results or explains counterexamples in English
```

### 리소스 리포트가 포함된 Yosys 합성
```
"Synthesize my design for the iCEBreaker"
 → Checks for unsupported SV constructs (warns before failing)
 → Runs Yosys synthesis targeting iCE40
 → Reports: LUTs: 142, FFs: 87, BRAM: 1, DSP: 0
```

### 바로 쓰는 IP 라이브러리
바로 사용할 수 있는, 정형적으로(formally) 증명된 검증 IP 블록 8종:
```
/gf-ip add fifo_sync    → Installs FIFO with RTL + testbench + formal proofs
/gf-ip add uart          → UART TX+RX with configurable baud rate
/gf-ip add axi4lite_slave → AXI4-Lite register interface
```
모든 블록은 lint 클린이며 시뮬레이션 테스트와 정형 검증을 거쳤습니다.

### 보드를 인식하는 개발
GateFlow는 당신의 FPGA 보드 핀아웃을 알고 있습니다:
```
/gf-boards arty-a7-35t pmod-ja  → Shows exact pin assignments
```
인기 있는 보드용으로 선별된 제약 파일(.xdc, .pcf, .cst)이 포함되어 있습니다.

### 하드웨어 설계 계획
`/gf-plan`은 전문적인 설계 문서를 만듭니다:
- 블록 다이어그램 (ASCII 및 Mermaid)
- 모듈 계층 구조와 인터페이스 명세
- FSM 상태 다이어그램
- 클럭 도메인 분석
- 검증 전략과 구현 단계

### 코드베이스 인텔리전스
`/gf-architect`는 프로젝트 전체를 매핑합니다:
- 모듈 계층 구조와 의존성
- 신호 흐름 분석과 FSM 추출
- 클럭 도메인 크로싱 탐지
- 패키지 및 타입 정의

### 폭넓은 커버리지
- **메모리**: FIFO, 듀얼 포트 RAM, 레지스터 파일
- **오류 처리**: ECC, 워치독, TMR
- **DFT**: 스캔 체인, JTAG, BIST
- **타이밍**: 리타이밍, 파이프라이닝, SDC
- **검증**: SVA, 커버리지, Formal

### 스마트 훅
GateFlow는 당신의 워크플로를 관찰하며 능동적으로 돕습니다:
- **SV 편집 후** — lint 하라고 상기시킴
- **파괴적인 커맨드 전** — SV 파일을 삭제하려 할 때 경고
- **세션 종료 시** — 수정된 파일을 lint 또는 시뮬레이션하는 것을 잊지 않았는지 확인
- **점진적 팁** — 아직 사용하지 않은 슬래시 커맨드를 안내

---

## 구성 요소

### 스킬 (27)

| 스킬 | 설명 | 소스 |
|-------|-------------|--------|
| `gf` | 메인 오케스트레이터 — 계획, 빌드, 동작할 때까지 검증 | [SKILL.md](plugins/gateflow/skills/gf/SKILL.md) |
| `gf-plan` | 다이어그램이 포함된 RTL 구현 계획 | [SKILL.md](plugins/gateflow/skills/gf-plan/SKILL.md) |
| `gf-build` | 병렬 컴포넌트 빌드 오케스트레이션 | [SKILL.md](plugins/gateflow/skills/gf-build/SKILL.md) |
| `gf-formal` | **자연어 기반 정형 검증** (SymbiYosys) | [SKILL.md](plugins/gateflow/skills/gf-formal/SKILL.md) |
| `gf-synth` | 면적/타이밍 리포트가 포함된 **Yosys 합성** | [SKILL.md](plugins/gateflow/skills/gf-synth/SKILL.md) |
| `gf-ip` | **IP 블록 라이브러리** — 바로 쓰는 검증 컴포넌트 | [SKILL.md](plugins/gateflow/skills/gf-ip/SKILL.md) |
| `gf-architect` | 계층 구조, FSM, 클럭, CDC를 담은 코드베이스 맵 | [SKILL.md](plugins/gateflow/skills/gf-architect/SKILL.md) |
| `gf-lint` | 구조화된 Verilator lint 검사 | [SKILL.md](plugins/gateflow/skills/gf-lint/SKILL.md) |
| `gf-sim` | DUT/TB 자동 감지가 포함된 시뮬레이션 | [SKILL.md](plugins/gateflow/skills/gf-sim/SKILL.md) |
| `gf-viz` | RTL 아키텍처의 터미널 시각화 | [SKILL.md](plugins/gateflow/skills/gf-viz/SKILL.md) |
| `gf-learn` | 학습 모드 — 연습, 리뷰, 힌트 | [SKILL.md](plugins/gateflow/skills/gf-learn/SKILL.md) |
| `gf-errors` | 하드웨어 도구를 위한 3계층 오류 번역 | [SKILL.md](plugins/gateflow/skills/gf-errors/SKILL.md) |
| `gf-project` | 프로젝트 컨텍스트(.gateflow/project.yaml) 관리 | [SKILL.md](plugins/gateflow/skills/gf-project/SKILL.md) |
| `gf-router` | 의도 분류 및 라우팅 | [SKILL.md](plugins/gateflow/skills/gf-router/SKILL.md) |
| `gf-expand` | 트레이드오프를 담은 명확화 질문 | [SKILL.md](plugins/gateflow/skills/gf-expand/SKILL.md) |
| `gf-summary` | Verilator/lint 출력 요약 | [SKILL.md](plugins/gateflow/skills/gf-summary/SKILL.md) |
| `tb-best-practices` | 테스트벤치 모범 사례 레퍼런스 | [SKILL.md](plugins/gateflow/skills/tb-best-practices/SKILL.md) |
| `gf-pinmap` | 제약 생성이 포함된 **보드 인식 핀 매핑** | [SKILL.md](plugins/gateflow/skills/gf-pinmap/SKILL.md) |
| `gf-pnr` | nextpnr를 통한 배치 & 라우팅 (iCE40/ECP5/Gowin) | [SKILL.md](plugins/gateflow/skills/gf-pnr/SKILL.md) |
| `gf-protocols` | 프로토콜 스캐폴딩 (AXI4, SPI, I2C, Wishbone) | [SKILL.md](plugins/gateflow/skills/gf-protocols/SKILL.md) |
| `gf-pcb` | AI 검증 루프가 포함된 **KiCad 회로도/PCB** | [SKILL.md](plugins/gateflow/skills/gf-pcb/SKILL.md) |
| `gf-cocotb` | Cocotb를 통한 Python 테스트벤치 | [SKILL.md](plugins/gateflow/skills/gf-cocotb/SKILL.md) |
| `gf-fusesoc` | FuseSoC 빌드 시스템 통합 | [SKILL.md](plugins/gateflow/skills/gf-fusesoc/SKILL.md) |
| `gf-learn-ctx` | 맥락 학습 — 워크플로 속 마이크로 레슨 | [SKILL.md](plugins/gateflow/skills/gf-learn-ctx/SKILL.md) |
| `gf-ip-detect` | **IP 블록 자동 감지** — 스캔, 매칭, 빈틈 자동 채움 | [SKILL.md](plugins/gateflow/skills/gf-ip-detect/SKILL.md) |
| `gf-tui` | **터미널 콘솔** — 로컬 OpenClaw 스타일 GateFlow 대시보드 | [SKILL.md](plugins/gateflow/skills/gf-tui/SKILL.md) |
| `gf-release` | **릴리스 준비 상태** — 매니페스트, 문서, 인덱스, 미러 검증 | [SKILL.md](plugins/gateflow/skills/gf-release/SKILL.md) |

### 에이전트 (20)

| 에이전트 | 전문 분야 | 소스 |
|-------|-----------|--------|
| `sv-codegen` | RTL 아키텍트 — 합성 가능한 모듈 | [sv-codegen.md](plugins/gateflow/agents/sv-codegen.md) |
| `sv-testbench` | 검증 엔지니어 — 테스트벤치와 자극 | [sv-testbench.md](plugins/gateflow/agents/sv-testbench.md) |
| `sv-debug` | 디버그 전문가 — 시뮬레이션 실패, X 값 | [sv-debug.md](plugins/gateflow/agents/sv-debug.md) |
| `sv-formal` | **정형 검증** — SVA 프로퍼티, SymbiYosys 증명 | [sv-formal.md](plugins/gateflow/agents/sv-formal.md) |
| `sv-synth` | **합성 전문가** — Yosys, 면적/타이밍 최적화 | [sv-synth.md](plugins/gateflow/agents/sv-synth.md) |
| `sv-verification` | 검증 방법론가 — SVA, 커버리지, 정형 | [sv-verification.md](plugins/gateflow/agents/sv-verification.md) |
| `sv-understanding` | RTL 분석가 — 코드 설명 및 문서화 | [sv-understanding.md](plugins/gateflow/agents/sv-understanding.md) |
| `sv-planner` | 아키텍처 플래너 — 설계 계획과 다이어그램 | [sv-planner.md](plugins/gateflow/agents/sv-planner.md) |
| `sv-orchestrator` | 병렬 빌더 — 다중 컴포넌트 설계 | [sv-orchestrator.md](plugins/gateflow/agents/sv-orchestrator.md) |
| `sv-refactor` | 코드 품질 — lint 수정, 정리, 최적화 | [sv-refactor.md](plugins/gateflow/agents/sv-refactor.md) |
| `sv-developer` | 풀스택 RTL — 복잡한 다중 파일 기능 | [sv-developer.md](plugins/gateflow/agents/sv-developer.md) |
| `sv-tutor` | 교사 — 솔루션 리뷰, 힌트 제공 | [sv-tutor.md](plugins/gateflow/agents/sv-tutor.md) |
| `sv-viz` | RTL 아키텍처의 터미널 시각화 | [sv-viz.md](plugins/gateflow/agents/sv-viz.md) |
| `sv-pinmap` | **핀 할당 전문가** — 제약 파일 | [sv-pinmap.md](plugins/gateflow/agents/sv-pinmap.md) |
| `vhdl-codegen` | **VHDL 코드 생성** — 엔티티와 아키텍처 | [vhdl-codegen.md](plugins/gateflow/agents/vhdl-codegen.md) |
| `vhdl-testbench` | **VHDL 테스트벤치** — GHDL 호환 검증 | [vhdl-testbench.md](plugins/gateflow/agents/vhdl-testbench.md) |
| `pcb-designer` | **KiCad PCB** — AI 검증 회로도 및 레이아웃 | [pcb-designer.md](plugins/gateflow/agents/pcb-designer.md) |
| `sv-ip-scanner` | **IP 스캐너** — 누락 모듈 감지, 자동 채움 | [sv-ip-scanner.md](plugins/gateflow/agents/sv-ip-scanner.md) |
| `gf-auditor` | **플러그인 감사** — 패키지 빈틈과 오래된 문서 탐지 | [gf-auditor.md](plugins/gateflow/agents/gf-auditor.md) |
| `gf-pluginfixer` | **플러그인 수정** — 감사된 플러그인 빈틈 복구 | [gf-pluginfixer.md](plugins/gateflow/agents/gf-pluginfixer.md) |

### 커맨드 (21)

| 커맨드 | 설명 | 소스 |
|---------|-------------|--------|
| `/gf-doctor` | 환경 점검 | [gf-doctor.md](plugins/gateflow/commands/gf-doctor.md) |
| `/gf-demo` | 무설정 쇼케이스 프로젝트 | [gf-demo.md](plugins/gateflow/commands/gf-demo.md) |
| `/gf-formal` | 정형 검증 실행 | [gf-formal.md](plugins/gateflow/commands/gf-formal.md) |
| `/gf-ip` | IP 블록 라이브러리 관리 | [gf-ip.md](plugins/gateflow/commands/gf-ip.md) |
| `/gf-boards` | 보드 목록 및 핀아웃 조회 | [gf-boards.md](plugins/gateflow/commands/gf-boards.md) |
| `/gf-scan` | 프로젝트 인덱싱 | [gf-scan.md](plugins/gateflow/commands/gf-scan.md) |
| `/gf-map` | 코드베이스 매핑 | [gf-map.md](plugins/gateflow/commands/gf-map.md) |
| `/gf-lint` | lint 실행 | [gf-lint.md](plugins/gateflow/commands/gf-lint.md) |
| `/gf-fix` | lint 수정 | [gf-fix.md](plugins/gateflow/commands/gf-fix.md) |
| `/gf-gen` | 스캐폴드 생성 | [gf-gen.md](plugins/gateflow/commands/gf-gen.md) |
| `/gf-sim` | 시뮬레이션 실행 | [gf-sim.md](plugins/gateflow/commands/gf-sim.md) |
| `/gf-pnr` | 배치 & 라우팅 (nextpnr) | [gf-pnr.md](plugins/gateflow/commands/gf-pnr.md) |
| `/gf-flash` | FPGA 보드 플래시 | [gf-flash.md](plugins/gateflow/commands/gf-flash.md) |
| `/gf-detect` | 누락 IP 블록 및 CDC 문제 스캔 | [gf-detect.md](plugins/gateflow/commands/gf-detect.md) |
| `/gf-pcb` | KiCad 회로도/PCB 생성 (AI 검증) | [gf-pcb.md](plugins/gateflow/commands/gf-pcb.md) |
| `/gf-pinmap` | 보드용 핀 제약 파일 생성 | [gf-pinmap.md](plugins/gateflow/commands/gf-pinmap.md) |
| `/gf-cocotb` | Python 테스트벤치 생성 (Cocotb) | [gf-cocotb.md](plugins/gateflow/commands/gf-cocotb.md) |
| `/gf-fusesoc` | FuseSoC .core 파일 생성 | [gf-fusesoc.md](plugins/gateflow/commands/gf-fusesoc.md) |
| `/gf-audit` | 플러그인 품질 감사 및 선택적 자동 수정 | [gf-audit.md](plugins/gateflow/commands/gf-audit.md) |
| `/gf-tui` | 로컬 GateFlow 터미널 콘솔 열기 | [gf-tui.md](plugins/gateflow/commands/gf-tui.md) |
| `/gf-release` | 플러그인 릴리스 준비 상태 검증 | [gf-release.md](plugins/gateflow/commands/gf-release.md) |

### IP 라이브러리 (검증된 블록 8종)

| 블록 | 설명 | 포함 항목 |
|-------|-------------|----------|
| `fifo_sync` | 동기 FIFO (파라미터화된 폭/깊이) | RTL + TB + formal |
| `fifo_async` | Gray 코드 포인터를 쓰는 비동기 FIFO (CDC) | RTL + TB + formal |
| `cdc_2ff` | 2-플립플롭 동기화기 | RTL + TB + formal |
| `cdc_handshake` | 멀티비트 핸드셰이크 동기화기 | RTL + TB + formal |
| `uart` | 구성 가능한 보드레이트의 UART TX+RX | RTL + TB + formal |
| `spi_master` | SPI 마스터 (4가지 CPOL/CPHA 모드 전부) | RTL + TB + formal |
| `axi4lite_slave` | AXI4-Lite 레지스터 슬레이브 | RTL + TB + formal |
| `debouncer` | 에지 감지가 포함된 버튼 디바운서 | RTL + TB + formal |

설치: `/gf-ip add fifo_sync` 또는 "add a FIFO to my project"

### 보드 데이터베이스 (보드 4종)

| 보드 | FPGA | 툴체인 |
|-------|------|-----------|
| Digilent Arty A7-35T | Xilinx xc7a35t | Vivado / Yosys |
| Digilent Basys 3 | Xilinx xc7a35t | Vivado / Yosys |
| 1BitSquared iCEBreaker | Lattice iCE40UP5K | Yosys + nextpnr |
| Sipeed Tang Nano 9K | Gowin GW1NR-9 | Yosys + nextpnr |

조회: `/gf-boards arty-a7-35t` 또는 "what pins does the Arty A7 have?"

에이전트는 요청에 따라 `/gf`가 자동으로 호출합니다 — 직접 호출할 필요가 없습니다.

---

## 개별 다운로드

전체 플러그인이 필요 없나요? 개별 구성 요소를 받으세요.

<details>
<summary><b>단일 구성 요소 다운로드 방법</b></summary>

각 구성 요소는 독립적인 `.md` 파일입니다. 다운로드하여 플러그인 디렉터리에 넣으세요:

```
your-plugin/
├── .claude-plugin/
│   └── plugin.json
├── agents/          ← agent .md files
├── commands/        ← command .md files
└── skills/
    └── skill-name/  ← SKILL.md files
        └── SKILL.md
```

### curl 예시

```bash
# Download an agent
curl -O https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/agents/sv-codegen.md

# Download a skill
mkdir -p skills/gf-plan
curl -o skills/gf-plan/SKILL.md https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/skills/gf-plan/SKILL.md

# Download a command
curl -O https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/commands/gf-lint.md
```

> **참고:** 일부 스킬(예: `gf-plan`)은 `references/` 하위 디렉터리에 레퍼런스 파일을 포함합니다. 전체 기능을 사용하려면 스킬 폴더 전체를 다운로드하세요.

</details>

---

## 크로스 툴 호환성

GateFlow의 스킬과 에이전트는 순수 Markdown이므로 여러 AI 코딩 도구에서 동작합니다.

<details>
<summary><b>Codex, Cursor, Copilot, Cline, Windsurf 설정 방법</b></summary>

### OpenAI Codex CLI

Codex는 동일한 `SKILL.md` 형식을 사용합니다:

```bash
# User-global
mkdir -p ~/.codex/skills/gf-plan
curl -o ~/.codex/skills/gf-plan/SKILL.md \
  https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/skills/gf-plan/SKILL.md

# Repo-level
mkdir -p .agents/skills/gf-lint
curl -o .agents/skills/gf-lint/SKILL.md \
  https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/skills/gf-lint/SKILL.md
```

| 위치 | 범위 |
|----------|-------|
| `.agents/skills/` | 현재 저장소 |
| `~/.codex/skills/` | 사용자 전역 |
| `/etc/codex/skills/` | 시스템 전체 |

### Cursor

```bash
# Append to .cursorrules
curl -s https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/agents/sv-codegen.md \
  >> .cursorrules

# Or: Settings → Agent Modes → Add Custom Mode → paste agent content
```

### GitHub Copilot CLI

```bash
mkdir -p .github
curl -s https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/agents/sv-codegen.md \
  >> .github/copilot-instructions.md
```

### Cline

```bash
curl -s https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/agents/sv-codegen.md \
  >> .clinerules
```

### Windsurf

```bash
# As a rule
mkdir -p .windsurf/rules
curl -o .windsurf/rules/sv-codegen.md \
  https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/agents/sv-codegen.md

# As a workflow
mkdir -p .windsurf/workflows
curl -o .windsurf/workflows/gf-plan.md \
  https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/skills/gf-plan/SKILL.md
```

### OpenCode

OpenCode는 MCP 서버와 스킬 파일을 지원합니다:

```bash
# Add GateFlow skills to your OpenCode config
mkdir -p .opencode/skills/gf-plan
curl -o .opencode/skills/gf-plan/SKILL.md \
  https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/skills/gf-plan/SKILL.md

# Add agent definitions
curl -O https://raw.githubusercontent.com/codejunkie99/Gateflow-Plugin/main/plugins/gateflow/agents/sv-codegen.md
```

### 빠른 참조

| 도구 | 파일 위치 | 형식 |
|------|-------------------|--------|
| **Claude Code** | 플러그인 `skills/`, `agents/`, `commands/` | 네이티브 |
| **Codex CLI** | `~/.codex/skills/` 또는 `.agents/skills/` | SKILL.md |
| **Cursor** | `.cursorrules` 또는 커스텀 에이전트 모드 | 추가(append) |
| **Copilot CLI** | `.github/copilot-instructions.md` | 추가(append) |
| **Cline** | `.clinerules` | 추가(append) |
| **Windsurf** | `.windsurf/rules/` 또는 `.windsurf/workflows/` | 개별 .md |
| **OpenCode** | `.opencode/skills/` 또는 MCP | SKILL.md / Agent .md |

</details>

---

## 설정

프로젝트별 설정을 위해 프로젝트에 `.claude/gateflow.local.md`를 만드세요:

```yaml
---
verilator_flags: ["-Wall", "-Wno-UNUSED"]
top_module: chip_top
clock_freq: 100MHz
---

# Project Notes
- Memory mapped registers at 0x1000
- AXI4-Lite interface for config
```

---

## 프로젝트 구조

```
Gateflow-Plugin/
├── plugins/gateflow/          # Main plugin source
│   ├── .claude-plugin/        #   Plugin manifest
│   ├── agents/                #   20 specialized AI agents
│   ├── commands/              #   21 slash commands
│   ├── skills/                #   27 auto-activating skills
│   ├── hooks/                 #   Automation hooks + session tracking
│   ├── boards/                #   Curated FPGA board database (4 boards)
│   ├── ip/                    #   Verified IP block library (8 blocks)
│   └── CLAUDE.md              #   SystemVerilog reference
├── agents/                    # Mirrored agent entrypoints (symlinks)
├── skills/                    # Mirrored skill entrypoints (symlinks)
├── docs/                      # Compressed docs index
├── CLAUDE.md                  # SV reference (repo-level)
└── AGENTS.md                  # Docs index for non-Claude agents
```

| 파일 | 대상 |
|------|-----|
| `CLAUDE.md` | Claude Code (주 레퍼런스) |
| `AGENTS.md` | 기타 AI 에이전트 (Cursor, Copilot 등) |

---

## 문제 해결

<details>
<summary><b>"Verilator not found"</b></summary>

```bash
verilator --version            # Check if installed
brew install verilator         # macOS
sudo apt install verilator     # Linux (Debian/Ubuntu)
```
</details>

<details>
<summary><b>"Plugin not loading"</b></summary>

```bash
claude --plugin-dir /path/to/Gateflow-Plugin/plugins/gateflow
ls /path/to/Gateflow-Plugin/plugins/gateflow/.claude-plugin/plugin.json
```
</details>

<details>
<summary><b>"Agent not found"</b></summary>

에이전트를 수동으로 스폰할 때는 `gateflow:` 접두사를 사용하세요:
```
gateflow:sv-codegen
gateflow:sv-testbench
```
</details>

---

## 업데이트

상세한 릴리스 노트는 [`releases.md`](releases.md)를 참고하세요.

| 버전 | 날짜 | 변경 사항 |
|---------|------|-------------|
| **2.5.3** | 2026-05-21 | 커맨드 중심 로컬 CLI, 대화형 에이전트 생성, 더 풍부한 터미널 색상 |
| **2.5.2** | 2026-05-21 | 좁은 터미널 창을 위한 반응형 TUI 레이아웃 |
| **2.5.1** | 2026-05-21 | 커서 및 무색 PTY를 위한 TUI 터미널 호환성 수정 |
| **2.5.0** | 2026-05-21 | OpenClaw 스타일 CLI/TUI, 릴리스 준비 워크플로, 결정적 검증기, 마켓플레이스/문서/인덱스/미러 동기화 |
| **2.4.0** | 2026-04-11 | 검증, 합성, 오케스트레이션, 아키텍처, 학습, IP, 계획 전반의 심층 스킬 보강 |
| **2.3.0** | 2026-03-27 | 품질 개선, IP 문서 확장, 신규 커맨드, 구성 요소 레퍼런스 수정 |
| **2.2.1** | 2026-03-26 | IP 자동 감지, 자동 채움, CDC 스캐닝, `sv-ip-scanner` |
| **2.2.0** | 2026-03-26 | 커뮤니티 가이드, KiCad, Cocotb, FuseSoC, CI 템플릿, 생태계 통합 |
| **2.1.0** | 2026-03-26 | VHDL, 핀 매핑, 배치 및 라우팅, FPGA 플래시, 프로토콜 스캐폴딩 |
| **2.0.0** | 2026-03-26 | 정형 검증, 합성, IP 라이브러리, 보드 데이터베이스 |
| **1.6.0** | 2026-03-26 | plugin.json과 marketplace.json 간 버전 동기화; BSL-1.1 라이선스 확정 |
| **1.5.3** | 2026-02-18 | 프롬프트 기반 PostToolUse 훅을 결정적 Python 스크립트로 교체 |
| **1.5.2** | 2026-02-15 | Stop 훅 JSON 검증 수정: 프롬프트 훅을 결정적 커맨드 훅(논블로킹 리마인더)으로 교체 |
| **1.5.0** | 2025-02-11 | `/gf-viz` 스킬과 `sv-viz` 에이전트로 터미널 시각화 |
| **1.4.4** | 2025-02-11 | 개별 구성 요소 다운로드, 크로스 툴 설치 안내 |
| **1.4.3** | 2025-02-10 | `gf-plan` 레퍼런스 분리, 검증 수정, 문서 개선 |

---

## 기여

기여를 환영합니다! 도움을 주시면 좋을 분야:
- **보드 정의**: `plugins/gateflow/boards/`에 당신의 FPGA 보드를 추가
- **IP 블록**: `plugins/gateflow/ip/`에 검증된 블록 제출
- **프로토콜 지원**: AXI4-Full, PCIe, USB, Ethernet, Wishbone
- **도구 통합**: Cocotb, FuseSoC, Edalize
- **문서 및 예제**

---

## 라이선스

BSL-1.1 (Business Source License) — 자세한 내용은 [LICENSE](LICENSE)를 참고하세요.

**가능한 것:** 비상업적/개인적/교육적 목적의 사용, 포크, 기여.
**상업적 사용:** 라이선스는 문의해 주세요.
**2028년 이후:** Apache 2.0으로 전환됩니다.

---

## 링크

- [Claude Code Documentation](https://code.claude.com/docs)
- [Verilator](https://verilator.org/)
- [SystemVerilog LRM](https://ieeexplore.ieee.org/document/8299595)

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=codejunkie99/Gateflow-Plugin&type=date&legend=top-left)](https://www.star-history.com/#codejunkie99/Gateflow-Plugin&type=date&legend=top-left)

---

<p align="center">
  <b>더 빠르게 나아가고 싶은 하드웨어 엔지니어를 위해 만들었습니다.</b><br>
  <i>Design. Verify. Ship.</i>
</p>
