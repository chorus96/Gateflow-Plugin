# SPI 프로토콜 레퍼런스

## 신호
| 신호 | 방향 (마스터) | 설명 |
|--------|-------------------|-------------|
| SCLK | output | 시리얼 클럭 |
| MOSI | output | Master Out Slave In |
| MISO | input | Master In Slave Out |
| CS_N | output | 칩 셀렉트 (active low) |

## 모드
| 모드 | CPOL | CPHA | 클럭 유휴 | 데이터 캡처 | 데이터 시프트 |
|------|------|------|------------|-------------|------------|
| 0 | 0 | 0 | Low | 상승 에지 | 하강 에지 |
| 1 | 0 | 1 | Low | 하강 에지 | 상승 에지 |
| 2 | 1 | 0 | High | 하강 에지 | 상승 에지 |
| 3 | 1 | 1 | High | 상승 에지 | 하강 에지 |

## 타이밍
- 첫 SCLK 에지 전에 CS_N assert (low)
- 캡처 에지 전에 데이터 valid
- 마지막 SCLK 에지 후에 CS_N deassert (high)
- MSB 먼저 (기본) 또는 LSB 먼저 (구성 가능)
