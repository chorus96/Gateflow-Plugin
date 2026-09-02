# fifo_sync — 동기 FIFO

full/empty 플래그를 갖춘 파라미터화된 동기 FIFO.

## 인스턴스화

```systemverilog
fifo_sync #(.WIDTH(8), .DEPTH(16)) u_fifo (
    .clk     (clk),
    .rst_n   (rst_n),
    .wr_en   (wr_en),
    .wr_data (wr_data),
    .rd_en   (rd_en),
    .rd_data (rd_data),
    .full    (full),
    .empty   (empty)
);
```

## 검증

- **Lint**: `verilator --lint-only -Wall rtl/fifo_sync.sv`
- **Sim**: `verilator --binary -j0 --trace -Irtl tb/tb_fifo_sync.sv rtl/fifo_sync.sv && ./obj_dir/Vtb_fifo_sync`
- **Formal**: `sby -f formal/fifo_sync.sby`
