# cdc_2ff — 2-플립플롭 동기화기

단일 비트 클럭 도메인 크로싱 동기화기.

## 인스턴스화
```systemverilog
cdc_2ff #(.STAGES(2)) u_sync (
    .clk      (dest_clk),
    .rst_n    (rst_n),
    .async_in (src_signal),
    .sync_out (synced_signal)
);
```
