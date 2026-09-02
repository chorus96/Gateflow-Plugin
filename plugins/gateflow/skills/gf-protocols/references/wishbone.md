# Wishbone 프로토콜 레퍼런스

## 신호 (Classic Cycle)
| 신호 | 폭 | 방향 (슬레이브) | 설명 |
|--------|-------|-------------------|-------------|
| CYC_I | 1 | input | 버스 사이클 활성 |
| STB_I | 1 | input | 스트로브 (전송 요청) |
| WE_I | 1 | input | 쓰기 활성화 (1=쓰기, 0=읽기) |
| ADR_I | ADDR_W | input | 주소 |
| DAT_I | DATA_W | input | 쓰기 데이터 |
| DAT_O | DATA_W | output | 읽기 데이터 |
| ACK_O | 1 | output | 전송 확인 |
| SEL_I | DATA_W/8 | input | 바이트 선택 |

## 전송 규칙
- CYC는 전체 버스 사이클 동안 assert되어야 함
- STB는 각 개별 전송마다 assert됨
- 슬레이브는 같은 또는 이후 사이클에 ACK로 응답
- 데이터는 ACK 사이클에 valid

## 파이프라인 모드
- STB는 이전 ACK 전에 assert될 수 있음
- 버스트 전송에 더 높은 처리량
