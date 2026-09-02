# cdc_handshake — 멀티비트 핸드셰이크 동기화기

클럭 도메인 간에 멀티비트 데이터를 안전하게 넘기기 위한 req/ack 프로토콜.

## 파라미터

| 이름 | 기본값 | 설명 |
|------|---------|-------------|
| WIDTH | 8 | 데이터 폭 (비트) |

## 인스턴스화

```systemverilog
cdc_handshake #(.WIDTH(8)) u_cdc (
    .src_clk   (clk_a),
    .src_rst_n (rst_a_n),
    .src_data  (src_data),
    .src_valid (src_valid),
    .src_ready (src_ready),
    .dst_clk   (clk_b),
    .dst_rst_n (rst_b_n),
    .dst_data  (dst_data),
    .dst_valid (dst_valid)
);
```

## 동작 방식

1. 소스가 데이터를 제시하고 `src_valid`를 assert
2. `src_ready`가 높을 때, 데이터가 캡처되고 req가 토글됨
3. Req가 2FF 동기화기를 통해 목적지 도메인으로 넘어감
4. 목적지가 데이터를 캡처하고 `dst_valid`를 펄스하며 ack를 돌려보냄
5. Ack가 다시 넘어오고 `src_ready`가 재차 assert됨

## 검증

- **Lint**: `verilator --lint-only -Wall rtl/cdc_handshake.sv`
- **Sim**: 도메인 간 데이터 전송 테스트
- **Formal**: 백프레셔와 데이터 안정성 프로퍼티
