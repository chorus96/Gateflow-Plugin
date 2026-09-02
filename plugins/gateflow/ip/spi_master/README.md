# spi_master — SPI 마스터

구성 가능한 클럭 분주기와 함께 4가지 CPOL/CPHA 모드 전부를 지원하는 SPI 마스터.

## 파라미터

| 이름 | 기본값 | 설명 |
|------|---------|-------------|
| CLK_DIV | 4 | SCLK = clk / (2 * CLK_DIV) |
| DATA_WIDTH | 8 | 전송당 비트 |

## 인스턴스화

```systemverilog
spi_master #(.CLK_DIV(4), .DATA_WIDTH(8)) u_spi (
    .clk     (clk),
    .rst_n   (rst_n),
    .cpol    (1'b0),      // Clock polarity
    .cpha    (1'b0),      // Clock phase
    .tx_data (tx_data),
    .start   (start),
    .rx_data (rx_data),
    .busy    (busy),
    .done    (done),
    .sclk    (spi_sclk),
    .mosi    (spi_mosi),
    .miso    (spi_miso),
    .cs_n    (spi_cs_n)
);
```

## SPI 모드

| 모드 | CPOL | CPHA | 캡처 에지 |
|------|------|------|-------------|
| 0 | 0 | 0 | 상승 |
| 1 | 0 | 1 | 하강 |
| 2 | 1 | 0 | 하강 |
| 3 | 1 | 1 | 상승 |

## 검증

- **Lint**: `verilator --lint-only -Wall rtl/spi_master.sv`
- **Sim**: 루프백 테스트 (MOSI → MISO)
- **Formal**: 전송 중 CS_N 낮음, SCLK는 활성일 때만 토글
