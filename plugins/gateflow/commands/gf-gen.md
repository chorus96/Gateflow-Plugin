---
name: gf-gen
description: Generate scaffolds
argument-hint: "<type> <name> [options]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
---

# GateFlow Generate 커맨드

SystemVerilog 코드 산출물을 생성합니다: 모듈, 테스트벤치, 패키지.

## 지침

인자를 파싱해 생성 유형을 판단:
- `gf-gen module <name>` - 새 모듈 생성
- `gf-gen testbench <name>` - 모듈용 테스트벤치 생성
- `gf-gen package <name>` - 패키지 생성

### 모듈 생성

다음을 갖춘 합성 가능한 모듈 생성:
- 파일명과 설명이 있는 표준 헤더 주석
- 파라미터 선언 (필요 시)
- 올바른 방향과 타입의 포트 목록
- 내부 신호 선언
- 기본 조합/순차 로직 구조
- 올바른 `always_ff`와 `always_comb` 블록

### 테스트벤치 생성

다음과 같은 테스트벤치 생성:
- DUT(Device Under Test)를 인스턴스화
- 클럭과 리셋 신호 생성
- `$dumpfile`과 `$dumpvars`가 있는 initial 블록 포함
- 테스트 자극을 위한 플레이스홀더 보유
- 시뮬레이션 종료를 위해 `$finish` 사용
- UVM-lite 또는 간단한 지향 테스트 스타일을 따름

### 패키지 생성

다음을 갖춘 패키지 생성:
- 타입 정의 (enum, struct)
- 파라미터 상수
- 함수 선언
- 올바른 export 문

## 출력

생성된 파일을 `<name>.sv`에, 테스트벤치는 `<name>_tb.sv`에 작성.
