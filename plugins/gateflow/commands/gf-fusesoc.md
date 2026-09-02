---
name: gf-fusesoc
description: Generate FuseSoC .core file for the project
argument-hint: "[--target sim|synth]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
---

# GateFlow FuseSoC 커맨드

현재 프로젝트로부터 FuseSoC `.core` 파일을 생성합니다.

## 사용법

```
/gf-fusesoc                   # Auto-detect and generate
/gf-fusesoc --target synth    # Generate with synthesis target
```

## 출력

- `<project>.core` — filesets, targets, 도구 구성이 포함된 FuseSoC core 파일

`rtl/`, `tb/`, 제약, `.gateflow/project.yaml`을 스캔합니다.
