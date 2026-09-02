---
name: gf-lint
description: >
  SystemVerilog lint checker with structured output for orchestration.
  Runs Verilator lint, categorizes errors/warnings, explains issues,
  and returns a parseable result block for /gf orchestration.
allowed-tools:
  - Bash
  - Read
  - Glob
---

# GF Lint 스킬

오케스트레이션을 위한 구조화된 출력으로 SystemVerilog 파일을 lint합니다.

## 도구 감지

lint를 실행하기 전에 Verilator가 사용 가능한지 확인:
```bash
which verilator
```

Verilator를 사용할 수 없으면, Verible을 확인:
```bash
which verible-verilog-lint
```

둘 다 사용할 수 없으면, 즉시 반환:
```
---GATEFLOW-RESULT---
STATUS: ERROR
ERRORS: 0
WARNINGS: 0
FILES: []
DETAILS: No lint tool available. Install Verilator (recommended) or Verible.
  macOS: brew install verilator
  Linux: sudo apt install verilator
---END-GATEFLOW-RESULT---
```

도구 없이 lint를 시도하지 말 것. ERROR 상태를 반환하고 오케스트레이터가 처리하게 함.

---

## 지침

### 1. Lint할 파일 식별

**args에 파일이 지정되면:**
제공된 파일 경로를 사용.

**파일이 지정되지 않으면:**
SV 파일 자동 감지:
```bash
ls *.sv rtl/*.sv 2>/dev/null | head -20
```

### 2. Verilator Lint 실행

```bash
verilator --lint-only -Wall <files>
```

### 3. 출력 파싱 및 분류

카테고리별로 문제 계산:
- **ERRORS**: `%Error:` 줄 - 반드시 수정
- **WARNINGS**: `%Warning-*:` 줄 - 검토해야 함

### 4. 흔한 경고 설명

발견된 각 경고 유형에 대해 설명:

| 경고 | 의미 | 수정 |
|---------|---------|-----|
| UNUSED | 신호가 선언되었으나 사용되지 않음 | 신호를 제거하거나 `/* verilator lint_off UNUSED */`로 억제 |
| UNDRIVEN | 신호가 값을 대입받은 적 없음 | 신호를 대입하거나 연결 |
| WIDTH | 연산에서 비트 폭 불일치 | 명시적 크기 지정 추가: `[7:0]` |
| CASEINCOMPLETE | Case 문에 항목 누락 | `default:` 분기 추가 |
| LATCH | 불완전한 if/case로 래치 추론 | always_comb 시작에 기본 대입 추가 |
| BLKSEQ | always_ff의 blocking 대입 | 순차 블록에서 `<=`(non-blocking) 사용 |
| PINCONNECTEMPTY | 모듈 포트가 미연결로 남음 | 연결하거나 명시적으로 `.*`로 표시 |
| IMPLICIT | 암시적 wire 선언 | `logic` 또는 `wire`로 선언 |

### 5. 구조화된 결과 반환

**항상 이 정확한 블록 형식으로 응답을 끝내세요:**

```
---GATEFLOW-RESULT---
STATUS: PASS|FAIL|ERROR
ERRORS: <count>
WARNINGS: <count>
FILES: <comma-separated list>
DETAILS: <one-line summary>
---END-GATEFLOW-RESULT---
```

**상태 정의:**
- `PASS`: 오류 없음, 경고는 허용 (경고 0-N개 허용됨)
- `FAIL`: 하나 이상의 오류 발견
- `ERROR`: lint를 실행할 수 없음 (파일 누락, 도구 오류)

### 6. 예시 출력

```
Running lint on: rtl/fifo.sv, rtl/uart.sv

$ verilator --lint-only -Wall rtl/fifo.sv rtl/uart.sv

%Warning-UNUSED: rtl/fifo.sv:25: Signal 'debug_flag' is not used
%Warning-WIDTH: rtl/uart.sv:42: Operator ASSIGN expects 8 bits but got 16

## Summary

**Files:** 2 | **Errors:** 0 | **Warnings:** 2

### Warnings Explained

1. **UNUSED** (rtl/fifo.sv:25): `debug_flag` is declared but never read
   - Fix: Remove if unneeded, or suppress with lint pragma

2. **WIDTH** (rtl/uart.sv:42): Assigning 16-bit value to 8-bit signal
   - Fix: Truncate explicitly `data[7:0]` or widen the target

---GATEFLOW-RESULT---
STATUS: PASS
ERRORS: 0
WARNINGS: 2
FILES: rtl/fifo.sv,rtl/uart.sv
DETAILS: Lint passed with 2 warnings
---END-GATEFLOW-RESULT---
```

### 7. 오류 케이스 예시

```
$ verilator --lint-only -Wall rtl/broken.sv

%Error: rtl/broken.sv:10: syntax error, unexpected IDENTIFIER

---GATEFLOW-RESULT---
STATUS: FAIL
ERRORS: 1
WARNINGS: 0
FILES: rtl/broken.sv
DETAILS: Syntax error on line 10
---END-GATEFLOW-RESULT---
```

## /gf 오케스트레이터의 사용

`/gf` 스킬은 이 스킬을 내부적으로 사용하고 결과 블록을 파싱합니다:

```
Parse ---GATEFLOW-RESULT--- block:
- STATUS: PASS -> proceed to next step
- STATUS: FAIL -> spawn sv-refactor agent with error context
- STATUS: ERROR -> report issue to user
```

## Verilator v5 신규 경고

### Lint 경고 (-Wall로 활성화)
| 코드 | 설명 |
|---|---|
| ASSIGNEQEXPR | 표현식 내부에 `=` 사용 (`==`를 의미할 수 있음) |
| ALWNEVER | `always @*`에 빈 이벤트 목록 |
| ALWCOMBORDER | `always_comb`에서 사용 후 변수 설정 |
| IMPLICITSTATIC | 태스크/함수 변수가 암시적으로 static |
| WIDTHEXPAND | 값이 0 확장됨 |
| WIDTHTRUNC | 값이 잘림 |
| ENUMVALUE | Enum에 호환되지 않는 타입 대입 |
| CASEOVERLAP | Case 값이 겹침 |
| CASTCONST | `$cast`가 항상 성공 또는 실패 |
| CMPCONST | 비교가 항상 상수를 산출 |
| MISINDENT | 오해를 부르는 들여쓰기 |
| MULTIDRIVEN | 여러 always 블록에서 구동되는 신호 |
| SYNCASYNCNET | sync/async 리셋이 혼용된 신호 |
| INFINITELOOP | 루프 조건이 항상 참 |
| SELRANGE | 선택 인덱스가 범위 초과 |

### 스타일 경고 (-Wall로 활성화)
| 코드 | 설명 |
|---|---|
| ASCRANGE | 오름차순 범위 `[0:7]`의 packed 벡터 |
| DECLFILENAME | 모듈 이름이 파일명과 불일치 |
| DEFPARAM | 사용 중단된 `defparam` |
| GENUNNAMED | Generate 블록에 이름 없음 |
| IMPORTSTAR | `$unit` 스코프의 `import pkg::*` |

### v5 동작 변경
- WIDTH는 이제 메타 카테고리: WIDTH 비활성화 시 WIDTHEXPAND + WIDTHTRUNC 비활성화
- v5.038부터 `--assert` 기본 활성화
- `--lint-only`가 이제 `--timing`을 함의

## 경고 억제

### 인라인 프래그마
```systemverilog
/* verilator lint_off WIDTH */
assign narrow = wide;
/* verilator lint_on WIDTH */
```

### 컨트롤 파일
```
lint_off -rule WIDTH -file "*.sv" -match "Operator ASSIGN*"
```

## 오류 메시지 형식

```
%Error: file.sv:42:5: syntax error, unexpected ';'
%Warning-WIDTH: file.sv:25:15: Operator ASSIGN expects 8 bits
```

### 종료 코드
| 코드 | 의미 |
|---|---|
| 0 | 성공 (오류나 치명적 경고 없음) |
| 1 | 오류 또는 경고 (경고는 기본적으로 치명적) |

경고를 비치명적으로 만들려면 `-Wno-fatal`을 사용.

## 기계 판독 출력

```bash
verilator --lint-only --diagnostics-sarif *.sv  # SARIF JSON output
```
