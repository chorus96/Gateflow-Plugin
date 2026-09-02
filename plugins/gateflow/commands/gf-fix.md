---
name: gf-fix
description: Fix lint
argument-hint: "<file>"
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
---

# GateFlow Fix 커맨드

반복적인 lint-fix 사이클을 사용해 SystemVerilog 파일의 lint 오류를 자동으로 수정합니다.

## 지침

1. **초기 lint**: 지정된 파일에 Verilator 실행:
   ```bash
   verilator --lint-only -Wall <file> 2>&1
   ```

2. **오류 파싱**: 출력에서 모든 오류와 경고를 추출

3. **수정 루프** (최대 5회 반복):
   - 파일 내용을 읽음
   - 각 오류에 대해 적절한 수정 적용:
     - `UNUSED`: 미사용 신호를 제거하거나 `/* verilator lint_off UNUSED */` 추가
     - `UNDRIVEN`: 신호를 초기화하거나 올바르게 연결
     - `WIDTH`: 비트 폭을 일치하도록 조정
     - `CASEINCOMPLETE`: 누락된 case 항목이나 default 추가
     - `BLKSEQ`: always_ff 블록에서 `=`를 `<=`로 변경
   - Edit 도구로 수정 적용
   - lint를 재실행해 검증

4. **결과 보고**:
   - 이전/이후 오류 개수 표시
   - 적용된 모든 수정 나열
   - 수동 리뷰가 필요한 남은 문제 기록

5. **안전**: 변경 적용 전에 항상 diff를 표시. 파괴적 변경을 자동 승인하지 말 것.
