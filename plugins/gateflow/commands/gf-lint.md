---
name: gf-lint
description: Run lint
argument-hint: "[files...]"
allowed-tools:
  - Bash
  - Read
  - Glob
---

# GateFlow Lint 커맨드

Verilator lint 검사를 실행해 SystemVerilog 코드의 오류와 경고를 찾습니다.

> **참고:** `/gf` 오케스트레이터는 내부적으로 `skills/gf-lint`를 사용하며, 이는 자동 처리를 위한
> 구조화된 출력(GATEFLOW-RESULT 블록)을 제공합니다. 이 커맨드 버전은 사용자가 직접 호출하기 위한
> 것이며 사람이 읽기 좋은 출력을 제공합니다.

## 지침

1. 특정 파일이 제공되면, 해당 파일을 lint:
   ```bash
   verilator --lint-only -Wall <files>
   ```

2. 파일이 지정되지 않으면, 모든 SV 파일을 찾아 lint:
   ```bash
   verilator --lint-only -Wall *.sv
   ```

3. 출력을 파싱하고 문제를 분류:
   - **오류(Errors)**: 시뮬레이션 전에 반드시 수정
   - **경고(Warnings)**: 검토하고 해결해야 함
   - **스타일(Style)**: 선택적 개선

4. 발견된 각 문제에 대해:
   - 파일과 줄 번호를 표시
   - 오류의 의미를 설명
   - 수정 방법을 제안

5. 설명할 흔한 Verilator 경고:
   - `UNUSED`: 사용되지 않는 신호 또는 변수
   - `UNDRIVEN`: 절대 대입되지 않는 신호
   - `WIDTH`: 비트 폭 불일치
   - `CASEINCOMPLETE`: 누락된 case 항목
   - `BLKSEQ`: 순차 블록의 blocking 대입
