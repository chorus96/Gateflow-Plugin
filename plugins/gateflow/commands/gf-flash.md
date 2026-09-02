---
name: gf-flash
description: Flash bitstream to FPGA board
argument-hint: "[bitstream-file]"
allowed-tools:
  - Bash
  - Read
  - Glob
---

# GateFlow Flash 커맨드

openFPGALoader를 사용해 FPGA 보드를 프로그래밍합니다.

## 사용법
```
/gf-flash                        # Auto-detect board and bitstream
/gf-flash output.bin             # Flash specific file
/gf-flash --board arty output.bit  # Specify board
```

## 실행

1. openFPGALoader 확인: `which openFPGALoader`
2. `.gateflow/project.yaml` 또는 인자에서 보드를 읽음
3. 비트스트림 파일 찾기 (.bin, .bit, .config)
4. 플래시: `openFPGALoader -b <board> <bitstream>`

## 도구 감지

openFPGALoader가 설치되지 않았으면:
```
openFPGALoader not found. Install to flash FPGA boards.
  macOS: brew install openfpgaloader
  Linux: see https://github.com/trabucayre/openFPGALoader
```
