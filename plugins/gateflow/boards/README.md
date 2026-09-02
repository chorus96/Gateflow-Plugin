# GateFlow 보드 데이터베이스

제약 파일이 포함된 선별 FPGA 보드 정의.

## 지원 보드

| 보드 | FPGA | 제약 형식 |
|-------|------|-------------------|
| Arty A7-35T | Xilinx xc7a35t | `.xdc` |
| Basys 3 | Xilinx xc7a35t | `.xdc` |
| iCEBreaker | Lattice iCE40UP5K | `.pcf` |
| Tang Nano 9K | Gowin GW1NR-9 | `.cst` |

## 보드 추가

`plugins/gateflow/boards/<board-name>/`를 다음과 함께 생성:
- `board.yaml` — 보드 메타데이터 (FPGA, 클럭, 핀, 커넥터)
- `constraints.<ext>` — 핀 제약 파일 (.xdc/.pcf/.lpf/.cst)

### board.yaml 스키마

```yaml
name: Board Display Name
vendor: Manufacturer
fpga:
  family: xilinx-7series | ice40 | ecp5 | gowin
  device: exact part number
  package: package code
synth_target: synth_xilinx | synth_ice40 | synth_ecp5 | synth_gowin
pnr_target: nextpnr command (or null for vendor-only)
programmer: openFPGALoader command
clock:
  pin: pin name/number
  frequency: clock speed
  iostandard: LVCMOS33 (for XDC boards)
leds: [pin list]
buttons: [pin list]
switches: [pin list]  # optional
pmod:
  connector_name: [pin list]
```

## 기여

1. yaml + constraints로 보드 디렉터리 생성
2. 공식 문서 대비 핀 할당 검증
3. 보드 레퍼런스 매뉴얼 링크와 함께 PR 제출
