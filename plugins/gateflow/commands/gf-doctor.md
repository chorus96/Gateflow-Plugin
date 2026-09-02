---
name: gf-doctor
description: Env check
allowed-tools:
  - Bash
---

# GateFlow Doctor 커맨드

기능 계층별로 정리하여 도구와 의존성을 검증합니다.

## 계층별 도구 표시

각 도구가 무엇을 잠금 해제하는지에 따라 그룹화하여 표시:

### Core (RTL + Lint + Sim)
- Verilator: [installed/missing] — `verilator --version`
- Verible: [installed/missing] — `verible-verilog-syntax --version`

### Formal Verification (정형 검증)
- SymbiYosys: [installed/missing] — `sby --help`
- z3 solver: [installed/missing] — `z3 --version`

### Synthesis (합성)
- Yosys: [installed/missing] — `yosys --version`

### Place & Route + Flash
- nextpnr: [installed/missing] — `nextpnr-ice40 --version` (or ecp5/gowin)
- openFPGALoader: [installed/missing] — `openFPGALoader --version`

### VHDL Support (VHDL 지원)
- GHDL: [installed/missing] — `ghdl --version`

### PCB Design
- KiCad: [installed/missing] — `kicad-cli --version`

### Python Verification (Python 검증)
- Cocotb: [installed/missing] — `python3 -c "import cocotb"`

### Build System (빌드 시스템)
- FuseSoC: [installed/missing] — `fusesoc --version`

사용자가 사용하려 시도한 계층의 도구에 대해서만 설치 안내를 표시.
요약 표시: "X/Y tools installed. Run /gf-doctor --all for full list."

## 지침

각 의존성에 대해 진단 검사를 실행:

### 1. Verilator (필수)
```bash
verilator --version
```
- lint와 시뮬레이션에 필요
- 최소 버전: 5.0

### 2. Verible (선택)
```bash
verible-verilog-syntax --version 2>/dev/null || echo "Not installed"
```
- 선택 (포매팅과 구문 검사)

## 보고 형식

요약 표를 제시:
| 도구 | 상태 | 버전 |
|------|--------|---------|
| Verilator | ✅ OK | 5.x |
| Verible (optional) | ✅ OK | v0.0-xxxx |

## 누락된 의존성

필수 도구가 누락되면, 다음 세션 시작 시 자동 설치됨(지원되는 경우)을 사용자에게 알리거나 수동 설치할 수 있음을 안내. 선택 도구가 누락되면 수동 설치 가능함을 안내:

**수동 설치 (macOS):**
```bash
brew install verilator
brew tap chipsalliance/verible && brew install verible
```

**수동 설치 (Linux - Debian/Ubuntu):**
```bash
sudo apt-get install verilator
# Verible: download from https://github.com/chipsalliance/verible/releases
```

## 사용 가능한 커맨드

검증 후, 사용 가능한 GateFlow 커맨드를 나열:
- `/gf-lint` - lint 실행
- `/gf-sim` - sim 실행
- `/gf-gen` - 스캐폴드 생성
- `/gf-scan` - 프로젝트 인덱싱
