---
name: gf-protocols
description: >
  Protocol scaffolding library. Generates correct, readable protocol
  interface scaffolds (AXI4-Lite, SPI, UART, I2C, AXI4-Full, AXI-Stream,
  Wishbone) with testbench templates and integration examples.
  Example: "create an I2C master interface", "scaffold AXI-Stream source"
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
  - Task
---

# GF-Protocols — 프로토콜 스캐폴딩 라이브러리

프로덕션 IP 코어가 아닙니다. 엔지니어가 커스터마이징하는, 올바르고 읽기 쉬운 스캐폴드입니다.
그리고 테스트벤치와 통합 코드도 함께 — 진짜 시간이 드는 곳이 거기입니다.

## 사용 가능한 프로토콜

| 프로토콜 | 우선순위 | 스캐폴드 포함 항목 |
|----------|----------|-------------------|
| AXI4-Lite | 1 | 슬레이브 레지스터 인터페이스 + 마스터 BFM + TB |
| SPI | 2 | 마스터 + 슬레이브 + 루프백 TB |
| UART | 3 | TX + RX + 루프백 TB |
| I2C | 4 | 마스터 + 슬레이브 모델 + TB |
| AXI4-Full | 5 | 슬레이브 + 버스트 마스터 BFM + TB |
| AXI-Stream | 6 | 소스 + 싱크 + 패스스루 + TB |
| Wishbone | 7 | 슬레이브 + 마스터 + TB |

## 사용법

사용자가 프로토콜 인터페이스를 요청하면:
1. IP 라이브러리에 완전한 블록이 있는지 확인 (`/gf-ip list`)
2. IP로 사용 가능하면: 대신 `/gf-ip add <block>` 제안
3. 사용 불가하거나 사용자가 커스텀을 원하면: 스캐폴드 생성

## 스캐폴드 생성

각 스캐폴드는 다음을 포함:
- 올바른 포트 이름과 폭이 있는 RTL 뼈대
- 신호 타이밍 주석 (언제 assert/deassert할지)
- BFM(Bus Functional Model)이 있는 테스트벤치 템플릿
- 설계에 배선하는 방법을 보여주는 통합 예시

## 프로토콜 레퍼런스

상세 프로토콜 명세는 `references/`에 있습니다:
- `references/axi4-lite.md` — AXI4-Lite 신호 목록, 타이밍, 규칙
- `references/spi.md` — SPI 모드, 타이밍, 신호 설명
- `references/i2c.md` — I2C 프로토콜, 주소 지정, 클럭 스트레칭

스캐폴드를 생성할 때, 올바른 신호 이름, 폭, 타이밍 요구 사항을 위해
항상 레퍼런스를 먼저 읽으세요.

---

## AXI4-Lite 슬레이브 스캐폴드

```systemverilog
module axi4lite_slave_regs #(parameter int ADDR_WIDTH=4, DATA_WIDTH=32) (
    input  logic clk, rst_n,
    input  logic [ADDR_WIDTH-1:0] s_axi_awaddr, input logic s_axi_awvalid, output logic s_axi_awready,
    input  logic [DATA_WIDTH-1:0] s_axi_wdata, input logic [DATA_WIDTH/8-1:0] s_axi_wstrb,
    input  logic s_axi_wvalid, output logic s_axi_wready,
    output logic [1:0] s_axi_bresp, output logic s_axi_bvalid, input logic s_axi_bready,
    input  logic [ADDR_WIDTH-1:0] s_axi_araddr, input logic s_axi_arvalid, output logic s_axi_arready,
    output logic [DATA_WIDTH-1:0] s_axi_rdata, output logic [1:0] s_axi_rresp,
    output logic s_axi_rvalid, input logic s_axi_rready,
    output logic [DATA_WIDTH-1:0] reg0_out, reg1_out,
    input  logic [DATA_WIDTH-1:0] reg2_in, reg3_in
);
    // Write: accept AW+W, write register via wstrb, respond B
    // Read: accept AR, return register data on R
    // Customize: add registers, change address decode width
endmodule
```

## SPI 마스터 스캐폴드

```systemverilog
module spi_master_scaffold #(parameter int CLK_DIV=4) (
    input  logic clk, rst_n, cpol, cpha,
    input  logic [7:0] tx_data, input logic tx_valid, output logic tx_ready,
    output logic [7:0] rx_data, output logic rx_valid,
    output logic sclk, mosi, input logic miso, output logic cs_n
);
    // FSM: IDLE -> LEAD -> ACTIVE (8 bits MSB first) -> TRAIL -> DONE
    // Clock divider generates sclk from clk/(2*CLK_DIV)
    // CPOL/CPHA configure polarity and sample/shift edges
    // Customize: add multi-byte, variable width, DMA interface
endmodule
```

## UART TX 스캐폴드

```systemverilog
module uart_tx_scaffold #(parameter int CLK_FREQ=100_000_000, BAUD_RATE=115_200) (
    input  logic clk, rst_n,
    input  logic [7:0] tx_data, input logic tx_valid, output logic tx_ready,
    output logic tx
);
    // Baud generator: counter to CLK_FREQ/BAUD_RATE
    // FSM: IDLE -> START (1 low bit) -> DATA (8 bits LSB first) -> STOP (1 high bit)
    // tx_ready when IDLE
    // Customize: add parity, configurable data bits, RX companion
endmodule
```

## I2C 마스터 스캐폴드

```systemverilog
module i2c_master_scaffold #(parameter int CLK_FREQ=100_000_000, I2C_FREQ=100_000) (
    input  logic clk, rst_n,
    input  logic [6:0] slave_addr, input logic rw, input logic [7:0] wr_data,
    input  logic start, output logic [7:0] rd_data, output logic done, ack_err, busy,
    output logic scl_oen, sda_oen, input logic scl_i, sda_i
);
    // Quarter-period counter for I2C timing
    // FSM: IDLE -> START -> ADDR_BIT(8) -> ADDR_ACK -> WR/RD_BIT(8) -> WR/RD_ACK -> STOP
    // Open-drain: oen=0 drives low, oen=1 releases high via external pull-up
    // Customize: multi-byte transfers, clock stretching, repeated start
endmodule
```

## GATEFLOW-RESULT 통합

```
---GATEFLOW-RESULT---
STATUS: PASS | FAIL | ERROR
PROTOCOL: axi4lite | spi | uart | i2c | wishbone | axi_stream
FILES: [generated files]
DETAILS: [summary]
---END-GATEFLOW-RESULT---
```
