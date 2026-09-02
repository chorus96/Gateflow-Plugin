# AXI-Stream 프로토콜 레퍼런스

## 신호
| 신호 | 폭 | 방향 (소스) | 설명 |
|--------|-------|-------------------|-------------|
| TDATA | DATA_W | output | 데이터 페이로드 |
| TVALID | 1 | output | 데이터 valid |
| TREADY | 1 | input | 싱크가 수용 준비됨 |
| TLAST | 1 | output | 패킷의 마지막 비트 |
| TKEEP | DATA_W/8 | output | 바이트 한정자 (1=유효 바이트) |
| TSTRB | DATA_W/8 | output | 바이트가 데이터(1)인지 위치(0)인지 |
| TID | ID_W | output | 스트림 식별자 |
| TDEST | DEST_W | output | 라우팅 목적지 |
| TUSER | USER_W | output | 사이드밴드 사용자 데이터 |

## 전송 규칙
- TVALID && TREADY일 때 전송 발생
- TVALID는 TREADY에 의존해서는 안 됨 (조합 논리 루프 없음)
- TVALID가 assert되면 TREADY 핸드셰이크까지 해제할 수 없음
- TVALID가 높은 동안 TDATA, TLAST, TKEEP는 안정적이어야 함

## 흔한 패턴

### 소스 (생산자)
```systemverilog
assign m_axis_tvalid = data_available;
assign m_axis_tdata  = data_out;
assign m_axis_tlast  = is_last_beat;
// Advance when handshake: m_axis_tvalid && m_axis_tready
```

### 싱크 (소비자)
```systemverilog
assign s_axis_tready = can_accept;
// Capture when handshake: s_axis_tvalid && s_axis_tready
```

### 패스스루 (파이프라인 스테이지)
```systemverilog
// Register slice for timing closure
assign s_axis_tready = !m_axis_tvalid || m_axis_tready;
always_ff @(posedge clk)
    if (s_axis_tvalid && s_axis_tready) begin
        m_axis_tdata  <= s_axis_tdata;
        m_axis_tlast  <= s_axis_tlast;
        m_axis_tvalid <= 1'b1;
    end else if (m_axis_tready)
        m_axis_tvalid <= 1'b0;
```
