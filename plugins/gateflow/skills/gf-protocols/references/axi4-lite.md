# AXI4-Lite 프로토콜 레퍼런스

## 신호 목록

### 쓰기 주소 채널 (Write Address Channel)
| 신호 | 폭 | 방향 (슬레이브) | 설명 |
|--------|-------|-------------------|-------------|
| AWADDR | ADDR_WIDTH | input | 쓰기 주소 |
| AWPROT | 3 | input | 보호 타입 (보통 3'b000) |
| AWVALID | 1 | input | 쓰기 주소 valid |
| AWREADY | 1 | output | 쓰기 주소 ready |

### 쓰기 데이터 채널 (Write Data Channel)
| 신호 | 폭 | 방향 (슬레이브) | 설명 |
|--------|-------|-------------------|-------------|
| WDATA | DATA_WIDTH | input | 쓰기 데이터 |
| WSTRB | DATA_WIDTH/8 | input | 쓰기 바이트 스트로브 |
| WVALID | 1 | input | 쓰기 데이터 valid |
| WREADY | 1 | output | 쓰기 데이터 ready |

### 쓰기 응답 채널 (Write Response Channel)
| 신호 | 폭 | 방향 (슬레이브) | 설명 |
|--------|-------|-------------------|-------------|
| BRESP | 2 | output | 쓰기 응답 (00=OKAY) |
| BVALID | 1 | output | 쓰기 응답 valid |
| BREADY | 1 | input | 쓰기 응답 ready |

### 읽기 주소 채널 (Read Address Channel)
| 신호 | 폭 | 방향 (슬레이브) | 설명 |
|--------|-------|-------------------|-------------|
| ARADDR | ADDR_WIDTH | input | 읽기 주소 |
| ARPROT | 3 | input | 보호 타입 |
| ARVALID | 1 | input | 읽기 주소 valid |
| ARREADY | 1 | output | 읽기 주소 ready |

### 읽기 데이터 채널 (Read Data Channel)
| 신호 | 폭 | 방향 (슬레이브) | 설명 |
|--------|-------|-------------------|-------------|
| RDATA | DATA_WIDTH | output | 읽기 데이터 |
| RRESP | 2 | output | 읽기 응답 |
| RVALID | 1 | output | 읽기 데이터 valid |
| RREADY | 1 | input | 읽기 데이터 ready |

## 핵심 규칙
- Valid는 ready에 의존해서는 안 됨 (조합 논리 루프 없음)
- Valid는 ready 핸드셰이크까지 assert된 상태를 유지해야 함
- BRESP는 AWVALID/AWREADY와 WVALID/WREADY 둘 다 이후에
- 모든 채널은 독립적 (어떤 순서로든 핸드셰이크 가능)
