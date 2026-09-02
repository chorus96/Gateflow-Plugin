---
name: gf-detect
description: Scan codebase for missing IP blocks, stubs, and CDC issues
argument-hint: "[--auto-fill] [--cdc-only] [path]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

# GateFlow IP Detection 커맨드

하드웨어 코드베이스에서 누락된 모듈, IP 기회, CDC 위반을 스캔합니다.

## 사용법

```
/gf-detect                    # Full scan of rtl/ directory
/gf-detect --auto-fill        # Scan and auto-fill with user approval
/gf-detect --cdc-only         # Only check for CDC violations
/gf-detect src/               # Scan specific directory
```

## 찾아내는 것

1. **누락 모듈** — 인스턴스화되었으나 정의되지 않음
2. **스텁 모듈** — 정의되었으나 비어 있음 (TODO/FIXME 마커)
3. **IP 기회** — 검증된 IP 블록을 쓸 수 있는 임시 코드
4. **CDC 위반** — 동기화기가 없는 클럭 도메인 크로싱
5. **벤더 IP** — 오픈소스 대안이 있는 벤더 프리미티브

## 출력

다음을 포함한 보고서를 표시:
- 모듈 의존성 그래프
- 제안 IP 블록과 함께 누락 구현
- 심각도로 순위가 매겨진 CDC 문제
- 자동 채움 옵션 (--auto-fill 플래그가 있으면)

## 자동 채움 모드

`--auto-fill`을 쓰면, 보고서 표시 후:
1. 각 빈틈을 제안 조치와 함께 제시
2. 각각을 승인/건너뛰기하도록 사용자에게 요청
3. 승인된 조치를 구현하도록 에이전트를 파견
4. 모든 새 코드에 lint 실행
5. 최종 상태 보고
