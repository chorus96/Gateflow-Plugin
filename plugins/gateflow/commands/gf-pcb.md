---
name: gf-pcb
description: Generate KiCad schematic and PCB (AI-verified draft)
argument-hint: "[description]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - Task
  - WebSearch
---

# GateFlow PCB Design 커맨드

KiCad 회로도와 PCB 레이아웃을 AI 검증 초안으로 생성합니다.

## 사용법

```
/gf-pcb "iCE40 breakout with SPI flash and 2 PMODs"
/gf-pcb "sensor board with I2C temperature sensor and OLED"
```

## 출력

- `schematic.kicad_sch` — KiCad 회로도
- `board.kicad_pcb` — PCB 레이아웃 (간단한 보드만)
- `bom.csv` — 자재 명세서(BOM)
- `verification_report.md` — DRC/ERC/AI 리뷰 결과

모든 파일에 AI 생성 고지가 포함됩니다. 수동 리뷰가 필요합니다.
