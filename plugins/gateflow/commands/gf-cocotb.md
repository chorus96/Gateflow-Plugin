---
name: gf-cocotb
description: Generate Python testbench using Cocotb
argument-hint: "<module> [test-description]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
---

# GateFlow Cocotb 커맨드

Cocotb를 사용해 Python 기반 테스트벤치를 생성합니다.

## 사용법

```
/gf-cocotb rtl/counter.sv                      # Auto-generate tests
/gf-cocotb rtl/fifo.sv "test full and empty"    # Specific test focus
```

## 출력

- `test_<module>.py` — Cocotb 테스트 파일
- `Makefile` — Cocotb 시뮬레이션 makefile

요구 사항: `pip install cocotb`
