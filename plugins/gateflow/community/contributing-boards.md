# 보드 정의 기여

당신의 FPGA 보드를 GateFlow의 선별 데이터베이스에 추가하세요.

## 요구 사항

1. **검증된 핀 데이터** — 공식 보드 문서와 일치해야 함
2. **완전한 제약 파일** — PACKAGE_PIN과 IOSTANDARD 포함
3. **board.yaml** — boards/README.md의 스키마를 따름

## 단계

1. 저장소 포크
2. `plugins/gateflow/boards/<your-board>/` 생성
3. 보드 메타데이터가 있는 `board.yaml` 추가
4. 제약 파일 추가 (.xdc/.pcf/.lpf/.cst)
5. PR에 공식 보드 문서 링크 포함
6. PR 제출

## 검증 체크리스트

- [ ] 핀 할당이 공식 회로도와 일치
- [ ] IOSTANDARD가 보드 전압 레일과 일치
- [ ] 클럭 주파수와 핀이 올바름
- [ ] 모든 LED/버튼/스위치 핀 검증됨
- [ ] PMOD/커넥터 핀이 올바른 순서
