---
name: gf-scan
description: Index project
allowed-tools:
  - Bash
  - Read
  - Glob
---

# GateFlow Scan 커맨드

현재 프로젝트의 모든 SystemVerilog 파일을 인덱싱하여 모듈 데이터베이스를 구축합니다.

## 지침

1. Verilator를 사용해 모든 SV 파일을 빠르게 파싱하고 모듈을 표면화:
   ```bash
   verilator --lint-only -Wall *.sv 2>&1 | head -50
   ```

2. 또는 SV 파일을 수동으로 발견:
   - Glob으로 모든 `**/*.sv` 및 `**/*.svh` 파일 찾기
   - 각 파일을 읽어 모듈 선언 추출
   - 모듈 계층 구조 보고

3. 다음을 보여주는 결과 제시:
   - 발견된 총 파일 수
   - 모듈 이름과 위치
   - include 파일 의존성
   - 발생한 파싱 오류
