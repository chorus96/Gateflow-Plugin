---
name: gf-boards
description: List supported boards and query pinouts
argument-hint: "[board-name] [connector]"
allowed-tools:
  - Bash
  - Read
  - Glob
---

# GateFlow Boards 커맨드

## 사용법

```
/gf-boards                        # List all supported boards
/gf-boards arty-a7-35t            # Show board details and pinout
/gf-boards arty-a7-35t pmod-ja    # Show specific connector pins
```

## 실행

1. **인자 없음**: `${CLAUDE_PLUGIN_ROOT}/boards/`의 모든 디렉터리를 나열하고, 각 `board.yaml`을 읽어 이름 + FPGA + 클럭 표시
2. **보드 이름**: `boards/<name>/board.yaml`을 읽어 클럭, 커넥터, LED, 버튼을 포함한 전체 세부 정보 표시
3. **보드 + 커넥터**: 해당 커넥터의 I/O 표준과 함께 핀 수준 세부 정보 표시

## 출력 형식

```
Arty A7-35T (Digilent)
  FPGA:    xc7a35ticsg324-1L (Xilinx 7-series)
  Clock:   E3 @ 100MHz (LVCMOS33)
  LEDs:    H5, J5, T9, T10
  Buttons: D9, C9, B9, B8
  PMODs:   JA, JB, JC, JD
  Synth:   synth_xilinx
  Flash:   openFPGALoader -b arty
```
