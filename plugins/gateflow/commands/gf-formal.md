---
name: gf-formal
description: Run formal verification
argument-hint: "[files...] [--property 'description']"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
---

# GateFlow Formal Verification 커맨드

## 사용법

```
/gf-formal rtl/fifo.sv                            # Auto-generate and prove properties
/gf-formal rtl/fifo.sv --property "never overflows"  # Prove specific property
/gf-formal formal/fifo.sby                          # Run existing .sby config
```

## 실행

1. 인자가 `.sby` 파일이면: `sby -f <file>.sby`로 직접 실행
2. 인자가 `.sv` 파일이면:
   a. 모듈을 읽음
   b. SVA 프로퍼티를 생성 (또는 --property 힌트 사용)
   c. `.sby` 구성을 생성
   d. SymbiYosys 실행
   e. 3계층 오류 번역과 함께 보고
3. 인자가 없으면: `formal/`에서 `.sby` 구성을 찾아 전부 실행
