---
name: gf-sim
description: Run sim
argument-hint: "<testbench> [dut-files...]"
allowed-tools:
  - Bash
  - Read
  - Glob
---

# GateFlow Simulate 커맨드

Verilator를 사용해 SystemVerilog 시뮬레이션을 컴파일하고 실행합니다.

> **참고:** `/gf` 오케스트레이터는 내부적으로 `skills/gf-sim`을 사용하며, 이는 자동 처리를 위한
> 구조화된 출력(GATEFLOW-RESULT 블록)을 제공합니다. 이 커맨드 버전은 사용자가 직접 호출하기 위한
> 것이며 사람이 읽기 좋은 출력을 제공합니다.

## 지침

1. **파일 식별**:
   - 테스트벤치 파일 (보통 `*_tb.sv`)
   - DUT 파일 (테스트 대상 모듈)
   - 지정되지 않으면 테스트벤치 include에서 자동 감지

2. **Verilator로 컴파일**:
   ```bash
   verilator --binary -Wall <dut-files> <testbench>
   ```

3. **시뮬레이션 실행**:
   ```bash
   ./obj_dir/V<top_module>
   ```

4. **결과 확인**:
   - $display 출력을 찾음
   - $error 또는 $fatal 호출 확인
   - $finish에 도달했는지 검증

5. **결과 보고**:
   - PASS: 오류 없이 시뮬레이션 완료
   - FAIL: 오류 감지, 관련 출력 표시

## 흔한 문제

- **해결되지 않은 모듈**: 컴파일에 파일 누락
- **다중 top**: --top-module 지정
