# debouncer — 버튼 디바운서

깔끔한 출력과 단일 사이클 에지 감지를 갖춘 카운터 기반 디바운서.

## 파라미터

| 이름 | 기본값 | 설명 |
|------|---------|-------------|
| CLK_FREQ | 100_000_000 | 클럭 주파수 (Hz) |
| DEBOUNCE_MS | 20 | 디바운스 시간 (밀리초) |

## 인스턴스화

```systemverilog
debouncer #(
    .CLK_FREQ    (100_000_000),
    .DEBOUNCE_MS (20)
) u_btn (
    .clk      (clk),
    .rst_n    (rst_n),
    .btn_in   (btn_raw),
    .btn_out  (btn_clean),
    .btn_rise (btn_pressed),   // Single-cycle pulse on press
    .btn_fall (btn_released)   // Single-cycle pulse on release
);
```

## 동작 방식

1. 입력이 2FF 동기화기를 통과 (CDC 안전)
2. 입력이 출력과 다른 동안 카운터가 증가
3. 카운터가 임계값에 도달하면 출력이 전환됨
4. 에지 감지가 단일 사이클 rise/fall 펄스를 생성

## 검증

- **Lint**: `verilator --lint-only -Wall rtl/debouncer.sv`
- **Sim**: 빠른 토글이 있는 튀는 입력 테스트
- **Formal**: 에지는 단일 사이클, 동시 rise+fall 없음
