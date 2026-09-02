# F4PGA 통합 — "FPGA의 GCC"

F4PGA는 완전한 오픈소스 FPGA 툴체인으로, Project X-Ray를 통해
GateFlow의 합성 커버리지를 Xilinx 7-series로 확장합니다.

## 이것이 가능하게 하는 것

- Vivado 없이 Xilinx 7-series 지원 (Artix-7, Spartan-7)
- 통합 오픈소스 흐름: Yosys → nextpnr-xilinx → bitstream
- 더 넓은 디바이스 지원을 위한 `/gf-synth --backend f4pga`

## 설정

```bash
# Install F4PGA
conda install -c litex-hub f4pga
# Or from source: https://f4pga.org/

# Set environment
export F4PGA_INSTALL_DIR=~/f4pga
export FPGA_FAM=xc7
```

## GateFlow와 함께 사용

목표 보드가 Xilinx 7-series를 사용하고 사용자가 오픈소스를 선호할 때:
```
/gf-synth --backend f4pga rtl/top.sv
```

GateFlow는 F4PGA 가용성을 자동 감지하고 지원 디바이스에 대해
Vivado의 대안으로 제공합니다.

## 지원 디바이스 (Project X-Ray를 통해)

- Artix-7: xc7a35t, xc7a50t, xc7a100t, xc7a200t
- Spartan-7: xc7s6, xc7s15, xc7s25, xc7s50
- 부분 지원: Kintex-7 (실험적)
