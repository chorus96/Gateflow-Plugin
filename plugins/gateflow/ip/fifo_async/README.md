# fifo_async — 비동기 FIFO

안전한 CDC를 위한 Gray 코드 포인터 동기화를 갖춘 듀얼 클럭 FIFO.

## 파라미터

| 이름 | 기본값 | 설명 |
|------|---------|-------------|
| WIDTH | 8 | 데이터 폭 (비트) |
| DEPTH | 8 | FIFO 깊이 (2의 거듭제곱) |

## 인스턴스화

```systemverilog
fifo_async #(.WIDTH(8), .DEPTH(8)) u_afifo (
    .wr_clk   (clk_fast),
    .wr_rst_n (rst_fast_n),
    .wr_en    (wr_en),
    .wr_data  (wr_data),
    .full     (full),
    .rd_clk   (clk_slow),
    .rd_rst_n (rst_slow_n),
    .rd_en    (rd_en),
    .rd_data  (rd_data),
    .empty    (empty)
);
```

## 동작 방식

- 쓰기와 읽기 포인터가 Gray 코드로 변환됨
- Gray 코드 포인터가 2FF를 통해 도메인 간에 동기화됨
- Gray 코드 포인터를 비교하여 full/empty를 감지
- Gray 코드는 클럭당 1비트만 변경되도록 보장 — CDC에 안전

## 검증

- **Lint**: `verilator --lint-only -Wall rtl/fifo_async.sv`
- **Sim**: 서로 다른 주파수의 듀얼 클럭 테스트벤치
- **Formal**: full일 때 오버플로 없음
