# GateFlow

**오픈소스 AI 하드웨어 플랫폼.** 자연어로 동작하는 RTL을 설계, 검증, 합성, 릴리스, 배포합니다. 에이전트 20개. 스킬 27개. 커맨드 21개. 검증된 IP 블록 8개.

```bash
claude plugin add codejunkie99/Gateflow-Plugin
```

> 만들고 싶은 것을 말하세요. GateFlow가 계획하고, 병렬로 빌드하고, lint하고, 시뮬레이션하고, 동작하는 코드를 건네줍니다.

![GateFlow v2.3.1 Update](assets/update-2.3.1.svg)

---

## 하는 일

**종단 간 RTL 개발.** 단순한 코드 생성이 아니라 — 설계가 실제로 동작할 때까지 반복하는 완전한 검증 루프.

```
"Create a FIFO with AXI-Stream interface and test it"

  /gf asks requirements → plans architecture → spawns parallel agents
    → generates RTL → lints (Verilator) → fixes warnings automatically
    → creates self-checking testbench → simulates → fixes failures
    → delivers lint-clean, sim-passing code
```

**영어로 하는 정형 검증.** 증명할 것을 설명하면, SVA 프로퍼티 + SymbiYosys 증명을 얻습니다.

```
"Prove the FIFO never overflows" → SVA assertions + .sby config + proof result
```

**합성 + 배치 & 라우팅.** 오픈소스 도구로 실제 하드웨어를 대상으로 합니다.

```
"Synthesize for iCEBreaker" → Yosys → nextpnr → bitstream → flash
```

---

## 스택

### 에이전트 20개

| 에이전트 | 하는 일 |
|-------|-------------|
| `sv-codegen` | 명세로부터 합성 가능한 RTL |
| `sv-testbench` | 자가 검사 테스트벤치 |
| `sv-debug` | 시뮬레이션 실패의 근본 원인 |
| `sv-verification` | SVA 어서션 + 커버리지 |
| `sv-formal` | 정형 증명 (SymbiYosys) |
| `sv-synth` | Yosys 합성 최적화 |
| `sv-refactor` | lint 수정, 코드 정리 |
| `sv-planner` | 코드 전 아키텍처 계획 |
| `sv-developer` | 복잡한 다중 파일 변경 |
| `sv-orchestrator` | 병렬 컴포넌트 빌드 |
| `sv-understanding` | 코드 설명 + 분석 |
| `sv-tutor` | 대화형 SV 교육 |
| `sv-viz` | 터미널 계층/FSM 다이어그램 |
| `sv-pinmap` | 보드 인식 핀 제약 |
| `sv-ip-scanner` | 누락 IP + CDC 문제 자동 감지 |
| `vhdl-codegen` | VHDL-2008 생성 (GHDL) |
| `vhdl-testbench` | VHDL 테스트벤치 |
| `pcb-designer` | KiCad 회로도 + PCB |
| `gf-auditor` | 플러그인 헬스 체크 |
| `gf-pluginfixer` | 플러그인 문제 자동 수정 |

### 스킬 27개

| 카테고리 | 스킬 |
|----------|--------|
| **오케스트레이션** | `/gf` (메인), `/gf-build` (병렬), `/gf-router`, `/gf-expand` |
| **계획** | `/gf-plan`, `/gf-architect`, `/gf-project` |
| **검증** | `/gf-lint`, `/gf-sim`, `/gf-formal`, `/gf-cocotb` |
| **IP & 프로토콜** | `/gf-ip`, `/gf-ip-detect`, `/gf-protocols` |
| **하드웨어** | `/gf-pcb`, `/gf-pnr`, `/gf-pinmap`, `/gf-fusesoc` |
| **학습** | `/gf-learn`, `/gf-learn-ctx` |
| **시각화** | `/gf-viz` |
| **터미널** | `/gf-tui` |
| **릴리스** | `/gf-release` |
| **레퍼런스** | `tb-best-practices`, `/gf-build` |

### 커맨드 21개

| 커맨드 | 설명 |
|---------|-------------|
| `/gf-lint` | 구조화된 출력의 Verilator lint |
| `/gf-sim` | Verilator로 컴파일 + 시뮬레이션 |
| `/gf-fix` | lint 경고 자동 수정 |
| `/gf-gen` | 모듈과 테스트벤치 스캐폴드 |
| `/gf-scan` | 프로젝트 파일 인덱싱 |
| `/gf-map` | 코드베이스 아키텍처 매핑 |
| `/gf-doctor` | 환경 + 의존성 점검 |
| `/gf-formal` | 정형 검증 (SymbiYosys) |
| `/gf-ip` | 검증된 IP 라이브러리 관리 |
| `/gf-detect` | 누락 IP + CDC 문제 스캔 |
| `/gf-boards` | 보드 핀아웃 조회 |
| `/gf-pinmap` | 제약 파일 생성 |
| `/gf-pnr` | 배치 & 라우팅 (nextpnr) |
| `/gf-flash` | FPGA 프로그래밍 (openFPGALoader) |
| `/gf-pcb` | KiCad 회로도/PCB 생성 |
| `/gf-cocotb` | Python 테스트벤치 (Cocotb) |
| `/gf-fusesoc` | FuseSoC .core 파일 |
| `/gf-demo` | 대화형 데모 |
| `/gf-audit` | 플러그인 헬스 감사 |
| `/gf-tui` | 로컬 터미널 콘솔 |
| `/gf-release` | 릴리스 준비 상태 검증 |

### 로컬 CLI

저장소 루트에서:

```bash
python3 tools/gateflow_cli.py status
python3 tools/gateflow_cli.py agents list
python3 tools/gateflow_cli.py agents create "CDC Reviewer" \
  --role "clock-domain crossing reviewer" \
  --description "Reviews synchronizers and CDC constraints"
python3 tools/gateflow_cli.py shell
```

키보드 대시보드 내부에서 `a`를 눌러 새 에이전트를 생성.

### 검증된 IP 블록 8개

모든 블록은 RTL + 테스트벤치 + formal 프로퍼티 + 문서와 함께 제공됩니다.

| 블록 | 설명 |
|-------|-------------|
| `fifo_sync` | 동기 FIFO (파라미터화됨) |
| `fifo_async` | Gray 코드 CDC의 비동기 FIFO |
| `cdc_2ff` | 2-플립플롭 동기화기 |
| `cdc_handshake` | 멀티비트 핸드셰이크 동기화기 |
| `uart` | UART TX+RX (구성 가능한 보드레이트) |
| `spi_master` | SPI 마스터 (4가지 모드 전부) |
| `axi4lite_slave` | AXI4-Lite 레지스터 슬레이브 |
| `debouncer` | 버튼 디바운서 + 에지 감지 |

### 보드 구성 4개

전체 제약 파일과 함께 사전 검증된 핀 할당.

| 보드 | FPGA | 제약 |
|-------|------|-----------|
| Arty A7-35T | Xilinx XC7A35T | `.xdc` |
| Basys 3 | Xilinx XC7A35T | `.xdc` |
| iCEBreaker | Lattice iCE40UP5K | `.pcf` |
| Tang Nano 9K | Gowin GW1NR-9C | `.cst` |

---

## 동작 방식

```
User request
     |
     v
  /gf (orchestrator)
     |
     +-- Ask clarifying questions
     +-- Plan with sv-planner (architecture, FSMs, interfaces)
     +-- Build with sv-orchestrator (parallel agent dispatch)
     |      |
     |      +-- sv-codegen (module 1) --+
     |      +-- sv-codegen (module 2) --+-- parallel
     |      +-- sv-testbench -----------+
     |
     +-- Verify
     |      +-- gf-lint --> if FAIL --> sv-refactor --> re-lint
     |      +-- gf-sim  --> if FAIL --> sv-debug --> sv-refactor --> re-sim
     |
     +-- Deliver working, lint-clean, tested code
```

**모든 수정은 에이전트를 거칩니다.** 직접 편집 없음. 구조화된 결과 블록(`GATEFLOW-RESULT`)이 루프를 구동합니다. 사람의 지침을 요청하기 전 최대 3회 재시도.

---

## 지원 도구

| 도구 | 목적 | 설치 |
|------|---------|---------|
| **Verilator** | lint + 시뮬레이션 | `brew install verilator` |
| **Yosys** | 합성 | `brew install yosys` |
| **nextpnr** | 배치 & 라우팅 | `brew install nextpnr` |
| **SymbiYosys** | 정형 검증 | `pip install symbiyosys` |
| **openFPGALoader** | FPGA 프로그래밍 | `brew install openfpgaloader` |
| **KiCad** | PCB 설계 | `brew install --cask kicad` |
| **Cocotb** | Python 테스트벤치 | `pip install cocotb` |
| **GHDL** | VHDL 시뮬레이션 | `brew install ghdl` |

`/gf-doctor`를 실행해 무엇이 설치되었는지 확인하세요.

---

## 빠른 시작

```bash
# Install
claude plugin add codejunkie99/Gateflow-Plugin

# Check environment
/gf-doctor

# Build something
/gf create a 4-bit counter with testbench

# Plan first, build second
/gf-plan design a UART controller

# Verify formally
/gf-formal prove the counter never overflows

# Target real hardware
/gf-pnr synthesize and place for iCEBreaker

# Prepare plugin release
/gf-release --check-only

# Open terminal console
/gf-tui

# Use the local CLI
python3 tools/gateflow_cli.py shell
```

---

## 라이선스

[BSL-1.1](LICENSE)

[@Av1dlive](https://x.com/Av1dlive)가 만들었습니다
