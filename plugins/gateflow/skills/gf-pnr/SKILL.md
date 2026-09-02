---
name: gf-pnr
description: >
  Place and route with nextpnr for open-source FPGA targets.
  Supports iCE40, ECP5, and Gowin devices.
  Example: "place and route for iCEBreaker", "run P&R targeting ice40"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - AskUserQuestion
---

# GF-PNR -- Place & Route 스킬

## 지원 타겟

| FPGA 패밀리 | 도구 | 커맨드 |
|------------|------|---------|
| Lattice iCE40 | nextpnr-ice40 | `nextpnr-ice40 --up5k --package sg48 --json synth.json --pcf constraints.pcf --asc output.asc` |
| Lattice ECP5 | nextpnr-ecp5 | `nextpnr-ecp5 --85k --package CABGA381 --json synth.json --lpf constraints.lpf --textcfg output.config` |
| Gowin | nextpnr-gowin | `nextpnr-gowin --device GW1NR-LV9QN88PC6/I5 --json synth.json --cst constraints.cst` |

**미지원**: Xilinx (Vivado 사용), Intel (Quartus 사용). 이 경우 GateFlow는 대신 TCL 스크립트를 생성합니다.

## 도구 감지

```bash
which nextpnr-ice40 || which nextpnr-ecp5 || which nextpnr-gowin
```

찾을 수 없으면:
```
---GATEFLOW-RESULT---
STATUS: ERROR
DETAILS: nextpnr not installed. Install for place & route.
  macOS: brew install nextpnr
  Linux: see https://github.com/YosysHQ/nextpnr
---END-GATEFLOW-RESULT---
```

## 워크플로

1. 프로젝트 타겟 확인 (board.yaml -> pnr_target)
2. 합성 출력이 존재하는지 검증 (Yosys의 synth.json)
3. 제약 파일이 존재하는지 검증
4. 올바른 플래그로 nextpnr 실행
5. 타이밍과 사용률 보고
6. P&R 성공 시 비트스트림 생성

## 비트스트림 생성

P&R 후:
- iCE40: `icepack output.asc output.bin`
- ECP5: `ecppack output.config output.bit`
- Gowin: nextpnr-gowin 출력에 내장됨

## 결과 형식

```
---GATEFLOW-RESULT---
STATUS: PASS | FAIL | ERROR
TIMING:
  Fmax: 125.3 MHz
  Slack: +2.1 ns
UTILIZATION:
  LUTs: 142 / 5280 (2.7%)
  FFs: 87 / 5280 (1.6%)
FILES: [output files]
DETAILS: [summary or timing violations]
---END-GATEFLOW-RESULT---
```

## 흔한 플래그

| 플래그 | 목적 |
|---|---|
| `--freq <mhz>` | 목표 주파수 |
| `--seed <n>` / `-r` | RNG 시드 / 무작위화 |
| `--opt-timing` | 배치 후 타이밍 최적화 |
| `--report <file>` | JSON 타이밍/사용률 |
| `--timing-allow-fail` | 위반에도 진행 |
| `--router2-heatmap <prefix>` | 혼잡 히트맵 |
| `--placer heap` | Heap 배치기 (기본) |

## 타겟 세부 사항

### iCE40
디바이스: `--lp1k`, `--lp8k`, `--hx1k`, `--hx8k`, `--up5k`
출력: `--asc <file>` | 제약: `--pcf <file>`
전체 파이프라인: `yosys -p "synth_ice40 -json out.json" design.v && nextpnr-ice40 --hx8k --package ct256 --json out.json --pcf pins.pcf --asc out.asc && icepack out.asc out.bin`

### ECP5
디바이스: `--12k`, `--25k`, `--45k`, `--85k`
출력: `--textcfg <file>` | 제약: `--lpf <file>`, `--sdc <file>`
전체 파이프라인: `yosys -p "synth_ecp5 -json out.json" design.v && nextpnr-ecp5 --25k --package CABGA256 --json out.json --lpf pins.lpf --textcfg out.config && ecppack out.config out.bit`

### Gowin
디바이스: `--device <part>` (예: `GW1NR-LV9QN88PC6/I5`)
패밀리: `--vopt family=<fam>` (C-silicon의 경우, 예: `GW1N-9C`)
제약: `--vopt cst=<file>` (필수)
전체 파이프라인: `yosys -p "synth_gowin -json out.json" design.v && nextpnr-himbaechel --device GW1NR-LV9QN88PC6/I5 --vopt family=GW1N-9C --vopt cst=pins.cst --json out.json --write routed.json && gowin_pack -d GW1N-9C -o out.fs routed.json`

## 타이밍 분석

### Fmax
- 최악의 critical path에서 나온 최대 주파수
- 자동 결정은 `--freq 0`, 목표는 `--freq N`

### Slack
- 양수 = 타이밍 충족 | 음수 = 위반 | 0 = 정확
- Critical path = slack이 가장 나쁜 경로

### Fmax 개선
1. 긴 조합 논리 경로를 파이프라인화
2. 배치 후 최적화를 위한 `--opt-timing`
3. 더 나은 배치를 위한 다른 시드 (`-r`)
4. `--placer-heap-timingweight` 증가 (20-50)
5. 타이밍 주도 재라우팅을 위한 `--router2-tmg-ripup`
6. Yosys: ECP5용 `-nowidelut` (큰 Fmax 개선)

## 사용률 임계값

| 사용률 | 상태 |
|---|---|
| < 50% | 여유로움 |
| 50-70% | 보통 |
| 70-80% | 빡빡함 |
| > 80% | 경고: 혼잡 가능성 |
| > 95% | 라우팅 실패 가능 |

## 흔한 실패

| 실패 | 수정 |
|---|---|
| 라우팅 혼잡 | 더 큰 디바이스, 합성 최적화, 다른 시드 |
| 타이밍 위반 | 파이프라인, `--opt-timing`, 다른 시드 |
| 배치 실패 | 제약 확인, `--placer sa` 시도 |
| 조합 논리 루프 | 레지스터로 끊기, 또는 `--ignore-loops` |

## 제약 형식

### PCF (iCE40)
```
set_io clk 21
set_io led[0] 99
set_frequency clk 48
```

### LPF (ECP5)
```
LOCATE COMP "clk" SITE "P6";
IOBUF PORT "clk" IO_TYPE=LVCMOS33;
FREQUENCY PORT "clk" 48000000.0 HZ;
```

### CST (Gowin)
```
IO_LOC "clk" 52;
IO_PORT "clk" IO_TYPE=LVCMOS33;
```
