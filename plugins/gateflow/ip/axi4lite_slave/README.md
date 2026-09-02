# axi4lite_slave — AXI4-Lite 레지스터 슬레이브

바이트 스트로브와 완전한 AXI4-Lite 인터페이스를 갖춘 파라미터화된 레지스터 파일.

## 파라미터

| 이름 | 기본값 | 설명 |
|------|---------|-------------|
| ADDR_WIDTH | 8 | 주소 버스 폭 |
| DATA_WIDTH | 32 | 데이터 버스 폭 |
| NUM_REGS | 16 | 레지스터 수 |

## 인스턴스화

```systemverilog
axi4lite_slave #(
    .ADDR_WIDTH (8),
    .DATA_WIDTH (32),
    .NUM_REGS   (16)
) u_regs (
    .clk     (clk),
    .rst_n   (rst_n),
    .awaddr  (awaddr),
    .awvalid (awvalid),
    .awready (awready),
    .wdata   (wdata),
    .wstrb   (wstrb),
    .wvalid  (wvalid),
    .wready  (wready),
    .bresp   (bresp),
    .bvalid  (bvalid),
    .bready  (bready),
    .araddr  (araddr),
    .arvalid (arvalid),
    .arready (arready),
    .rdata   (rdata),
    .rresp   (rresp),
    .rvalid  (rvalid),
    .rready  (rready)
);
```

## 검증

- **Lint**: `verilator --lint-only -Wall rtl/axi4lite_slave.sv`
- **Sim**: 쓰기 후 읽기 검증이 있는 자가 검사 테스트벤치
- **Formal**: 쓰기 완료 후 쓰기 응답이 따라옴
