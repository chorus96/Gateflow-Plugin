# uart — UART TX+RX

8N1 형식의 구성 가능한 보드레이트 UART.

## 인스턴스화
```systemverilog
uart_tx #(.CLK_FREQ(100_000_000), .BAUD_RATE(115200)) u_tx (
    .clk(clk), .rst_n(rst_n),
    .tx_data(tx_data), .tx_valid(tx_valid),
    .tx_ready(tx_ready), .tx_out(uart_txd)
);
```
