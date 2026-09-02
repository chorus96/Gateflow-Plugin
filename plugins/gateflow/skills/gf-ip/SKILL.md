---
name: gf-ip
description: >
  IP block library manager. Install, list, and query verified drop-in
  hardware components. Each block includes RTL, testbench, formal
  properties, and documentation.
  Example: "add a FIFO to my project", "/gf-ip add uart"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
---

# GF-IP — IP 블록 라이브러리

## 커맨드

- `add <block>` — IP 블록을 현재 프로젝트로 복사
- `list` — 사용 가능한 모든 IP 블록 표시
- `info <block>` — 블록 세부 정보, 포트, 파라미터 표시

## 사용 가능한 블록

| 블록 | 설명 | 검증 |
|-------|-------------|----------|
| fifo_sync | 동기 FIFO (파라미터화된 폭/깊이) | lint + sim + formal |
| fifo_async | Gray 코드 포인터를 쓰는 비동기 FIFO (CDC) | lint + sim + formal |
| cdc_2ff | 2-플립플롭 동기화기 | lint + sim + formal |
| cdc_handshake | 멀티비트 핸드셰이크 동기화기 | lint + sim + formal |
| uart | 구성 가능한 보드레이트의 UART TX+RX | lint + sim + formal |
| spi_master | SPI 마스터 (4가지 CPOL/CPHA 모드 전부) | lint + sim + formal |
| axi4lite_slave | AXI4-Lite 레지스터 슬레이브 | lint + sim + formal |
| debouncer | 에지 감지가 포함된 버튼 디바운서 | lint + sim + formal |

## 블록 구조

각 블록은 `${CLAUDE_PLUGIN_ROOT}/ip/<name>/`에 위치:
```
<name>/
  rtl/<name>.sv           # RTL source
  tb/tb_<name>.sv         # Self-checking testbench
  formal/<name>_props.sv  # SVA properties
  formal/<name>.sby       # SymbiYosys config
  block.yaml              # Metadata
  README.md               # Usage guide
```

## Add 흐름

사용자가 "add a FIFO"라고 말하거나 `/gf-ip add fifo_sync`를 실행하면:
1. 메타데이터와 파라미터를 위해 block.yaml을 읽음
2. RTL을 `rtl/`(또는 사용자 지정 디렉터리)로 복사
3. 테스트벤치를 `tb/`로 복사
4. formal 프로퍼티를 `formal/`로 복사
5. `.gateflow/project.yaml` 갱신 — `ip_blocks`에 추가
6. README.md의 인스턴스화 예시를 표시

## 블록 메타데이터 스키마 (block.yaml)

```yaml
name: fifo_sync
version: 1.0.0
description: Synchronous FIFO with parameterized width and depth
parameters:
  WIDTH: { type: int, default: 8, description: "Data width in bits" }
  DEPTH: { type: int, default: 16, description: "FIFO depth (entries)" }
ports:
  - { name: clk, dir: input, width: 1 }
  - { name: rst_n, dir: input, width: 1 }
  - { name: wr_en, dir: input, width: 1 }
  - { name: wr_data, dir: input, width: WIDTH }
  - { name: rd_en, dir: input, width: 1 }
  - { name: rd_data, dir: output, width: WIDTH }
  - { name: full, dir: output, width: 1 }
  - { name: empty, dir: output, width: 1 }
formal_proofs:
  - p_no_overflow: "FIFO never accepts writes when full"
  - p_no_underflow: "FIFO never allows reads when empty"
dependencies: []
```

## 인스턴스화 예시

### fifo_sync
```systemverilog
fifo_sync #(.WIDTH(8), .DEPTH(32)) u_fifo (.clk(sys_clk), .rst_n(sys_rst_n), .wr_en(wr_valid && !fifo_full), .wr_data(wr_data), .rd_en(rd_consume), .rd_data(rd_out), .full(fifo_full), .empty(fifo_empty));
```

### fifo_async
```systemverilog
fifo_async #(.WIDTH(32), .DEPTH(16)) u_cdc (.wr_clk(clk_fast), .wr_rst_n(rst_fast_n), .wr_en(producer_valid), .wr_data(producer_data), .wr_full(full), .rd_clk(clk_slow), .rd_rst_n(rst_slow_n), .rd_en(consumer_ready), .rd_data(consumer_data), .rd_empty(empty));
```

### cdc_2ff
```systemverilog
cdc_2ff u_sync (.clk(clk_dst), .rst_n(rst_dst_n), .d(async_signal), .q(synced_signal));
```

### uart
```systemverilog
uart #(.CLK_FREQ(100_000_000), .BAUD_RATE(115200)) u_uart (.clk, .rst_n, .tx_data(tx_byte), .tx_valid(tx_start), .tx_ready(tx_idle), .tx(uart_tx_pin), .rx(uart_rx_pin), .rx_data(rx_byte), .rx_valid(rx_ready));
```

### spi_master
```systemverilog
spi_master #(.CLK_DIV(8)) u_spi (.clk, .rst_n, .cpol(1'b0), .cpha(1'b0), .tx_data(spi_tx), .tx_valid(spi_start), .tx_ready(spi_idle), .rx_data(spi_rx), .rx_valid(spi_done), .sclk(spi_sclk), .mosi(spi_mosi), .miso(spi_miso), .cs_n(spi_cs_n));
```

## IP 블록 비교

| 필요 | 사용 | 비사용 | 이유 |
|---|---|---|---|
| 동일 클럭 버퍼링 | fifo_sync | fifo_async | 비동기는 Gray 코드 오버헤드가 있음 |
| 도메인 간 스트림 | fifo_async | cdc_2ff | 2FF는 1비트만 처리 |
| 도메인 간 1비트 플래그 | cdc_2ff | fifo_async | 1비트에 FIFO는 과함 |
| 도메인 간 멀티비트 (드묾) | cdc_handshake | fifo_async | 핸드셰이크가 더 작음 |
| 도메인 간 멀티비트 (스트리밍) | fifo_async | cdc_handshake | 핸드셰이크는 블로킹됨 |

## GATEFLOW-RESULT 형식

```
---GATEFLOW-RESULT---
STATUS: PASS | FAIL | NEEDS_ACTION
OPERATION: add | list | info
BLOCK: <name>
VERSION: <version>
FILES: [copied files]
DETAILS: [summary]
---END-GATEFLOW-RESULT---
```
