# OpenFPGA 통합 — 커스텀 FPGA 아키텍처

OpenFPGA는 Verilog-to-bitstream 지원과 함께 커스터마이징 가능한 FPGA 아키텍처를
생성합니다. 이 통합으로 GateFlow가 커스텀 FPGA 패브릭을 대상으로 할 수 있습니다.

## 이것이 가능하게 하는 것

- 커스텀/학술용 FPGA 아키텍처 대상
- 아키텍처별 비트스트림 생성
- "자연어 → 커스텀 FPGA 아키텍처 → 비트스트림"

## 설정

```bash
git clone https://github.com/lnis-uofu/OpenFPGA.git
cd OpenFPGA && mkdir build && cd build
cmake .. && make -j$(nproc)
```

## GateFlow와 함께 사용

```
/gf-synth --target openfpga --arch my_architecture.xml rtl/top.sv
```

## 상태: Phase 5+ (예정)

이 통합에는 다음이 필요합니다:
- OpenFPGA 아키텍처 XML 정의
- 커스텀 배치 & 라우팅 구성
- 목표 아키텍처를 위한 비트스트림 생성
