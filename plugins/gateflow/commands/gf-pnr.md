---
name: gf-pnr
description: Run place and route
argument-hint: "[--target ice40|ecp5|gowin]"
allowed-tools:
  - Bash
  - Read
  - Glob
---

# GateFlow Place & Route 커맨드

오픈소스 FPGA 타겟을 위해 nextpnr 배치 & 라우팅을 실행합니다.

## 사용법
```
/gf-pnr                          # Use target from project.yaml
/gf-pnr --target ice40           # Explicit target
```

## 요구 사항
- Yosys 합성 출력 (synth.json)
- 제약 파일 (.pcf/.lpf/.cst)
- 타겟 패밀리용 nextpnr 설치
