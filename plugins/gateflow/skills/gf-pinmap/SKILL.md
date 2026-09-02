---
name: gf-pinmap
description: >
  Board-aware pin mapping. Generates FPGA constraint files (.xdc/.pcf/.lpf/.cst)
  with correct pin assignments, I/O standards, and drive strength for target boards.
  Example: "map SPI to PMOD JA on Arty A7", "generate constraints for my iCEBreaker"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - Task
  - WebSearch
  - WebFetch
---

# GF-Pinmap — 보드 인식 핀 매핑

## 워크플로

1. **보드 식별** — `.gateflow/project.yaml`의 target.board를 확인하거나 사용자에게 질문
2. **선별 데이터베이스 우선 확인**:
   ```bash
   ls ${CLAUDE_PLUGIN_ROOT}/boards/<board>/board.yaml 2>/dev/null
   ```
3. **보드를 찾으면**: board.yaml을 읽고 제약 파일 생성
4. **보드를 찾지 못하면**: 폴백으로 웹 검색, 사용자 확인 필요
5. 생성된 제약을 RTL 포트 목록과 **상호 참조**
6. 목표 FPGA에 맞는 올바른 형식으로 제약 파일 **출력**

## 선별 보드 흐름

1. 핀 데이터를 위해 `boards/<board>/board.yaml`을 읽음
2. 템플릿을 위해 `boards/<board>/constraints.*`를 읽음
3. 사용자의 주변장치 선택에 기반해 RTL 포트 → 보드 핀 매핑
4. 다음을 갖춘 완전한 제약 파일 생성:
   - PACKAGE_PIN
   - IOSTANDARD (board.yaml에서)
   - DRIVE 강도 (출력 기본 12mA)
   - SLEW 레이트 (기본 SLOW)
   - Active-low 신호용 PULLUP

## 웹 검색 폴백

보드가 선별 데이터베이스에 없을 때만:

1. 검색: `"<board name>" constraint file site:github.com`
2. 검색: `"<board name>" pinout .xdc OR .pcf OR .lpf`
3. 경고와 함께 결과를 사용자에게 제시:
   ```
   Found constraint data for <board> via web search.
   Source: <url>

   WARNING: This pin data has NOT been verified against the official
   board documentation. Incorrect pin assignments can damage hardware.

   Please review before applying:
   [show pin assignments]

   Apply these constraints? [Y/n]
   ```
4. 웹 검색 데이터를 자동 적용하지 말 것

## 안전

- 핀 할당을 절대 추측하지 말 것
- 항상 IOSTANDARD 포함 (누락 = 합성 오류 또는 하드웨어 손상)
- 전압 뱅크 호환성 확인
- 같은 뱅크에서 I/O 표준을 혼용하면 경고
- Active-low 신호는 PULLUP를 받음

## GATEFLOW-RESULT 형식

```
---GATEFLOW-RESULT---
STATUS: PASS | FAIL | ERROR
FORMAT: xdc | pcf | lpf | cst
BOARD: <board name>
PINS_MAPPED: <count>
PINS_UNMAPPED: <count>
FILE: <constraint file path>
DETAILS: <summary>
---END-GATEFLOW-RESULT---
```

## I/O 표준 레퍼런스

| IOSTANDARD | 전압 | 최대 속도 | 전형적 용도 |
|---|---|---|---|
| LVCMOS33 | 3.3V | ~100 MHz | GPIO, LED, UART, SPI |
| LVCMOS25 | 2.5V | ~150 MHz | 혼합 전압 |
| LVCMOS18 | 1.8V | ~200 MHz | 최신 주변장치 |
| LVDS_25 | 2.5V diff | ~1 Gbps | 고속 시리얼 |
| SSTL15 | 1.5V | ~800 MHz | DDR3 |
| TMDS_33 | 3.3V | ~750 Mbps | HDMI/DVI |

## 흔한 핀 매핑 실수

1. **IOSTANDARD 누락** -> 기본값이 하드웨어를 손상시킬 수 있음. 항상 지정.
2. **같은 뱅크에서 I/O 표준 혼용** -> 뱅크의 모든 핀이 VCCO를 공유.
3. **플로팅 입력** -> 미사용 입력에 PULLUP/PULLDOWN 추가.
4. **고속에 종단 없음** -> DDR에는 ODT, LVDS에는 100옴 사용.
5. **잘못된 DRIVE 강도** -> LED는 4mA, SPI는 8-12mA.
6. **create_clock 누락** -> 그것 없이는 타이밍 분석이 없음.

## PMOD 매핑 패턴

표준 PMOD 핀아웃: 핀 1-4 (상단 행 I/O), 5 (GND), 6 (VCC), 7-10 (하단 행 I/O), 11 (GND), 12 (VCC).

| PMOD 타입 | 핀 1 | 핀 2 | 핀 3 | 핀 4 |
|---|---|---|---|---|
| Type 2 (SPI) | CS_N | MOSI | MISO | SCLK |
| Type 3 (UART) | CTS | TXD | RXD | RTS |
| Type 6 (I2C) | SCL | SDA | INT_N | RST_N |
