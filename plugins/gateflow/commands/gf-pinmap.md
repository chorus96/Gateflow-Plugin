---
name: gf-pinmap
description: Generate pin constraint file for target board
argument-hint: "<board> [peripheral] [connector]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - WebSearch
---

# GateFlow Pin Mapping 커맨드

올바른 핀 할당이 있는 FPGA 제약 파일을 생성합니다.

## 사용법

```
/gf-pinmap arty-a7-35t                          # Full constraints for board
/gf-pinmap arty-a7-35t spi pmod-ja              # Map SPI to PMOD JA
/gf-pinmap icebreaker uart                       # Map UART on iCEBreaker
```

## 출력

올바른 형식으로 제약 파일을 생성:
- Xilinx: PACKAGE_PIN + IOSTANDARD가 있는 `.xdc`
- Lattice iCE40: set_io가 있는 `.pcf`
- Gowin: IO_LOC가 있는 `.cst`
- Lattice ECP5: LOCATE + IOBUF가 있는 `.lpf`

안전: 선별 데이터베이스 우선, 웹 검색은 사용자 확인이 필요합니다.
