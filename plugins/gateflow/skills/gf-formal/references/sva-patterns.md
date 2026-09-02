# SVA 프로퍼티 패턴

## 오버플로 없음 (FIFO/카운터)
```systemverilog
a_no_overflow: assert property (
    @(posedge clk) disable iff (rst)
    full |-> !wr_en
);
```

## 언더플로 없음
```systemverilog
a_no_underflow: assert property (
    @(posedge clk) disable iff (rst)
    empty |-> !rd_en
);
```

## Valid/Ready 핸드셰이크
```systemverilog
a_valid_stable: assert property (
    @(posedge clk) disable iff (rst)
    (valid && !ready) |=> valid
);

a_data_stable: assert property (
    @(posedge clk) disable iff (rst)
    (valid && !ready) |=> $stable(data)
);
```

## One-Hot (상호 배제)
```systemverilog
a_state_onehot: assert property (
    @(posedge clk) disable iff (rst)
    $onehot(state)
);
```

## 활성성 (요청이 승인됨)
```systemverilog
a_req_granted: assert property (
    @(posedge clk) disable iff (rst)
    req |-> ##[1:MAX_LATENCY] grant
);
```

## 리셋 동작
```systemverilog
a_reset_outputs: assert property (
    @(posedge clk)
    rst |-> (data_out == '0) && (count == '0)
);
```

## FIFO 카운트 추적
```systemverilog
a_fifo_count_inc: assert property (
    @(posedge clk) disable iff (rst)
    (wr_en && !rd_en && !full) |=> (count == $past(count) + 1)
);
```
