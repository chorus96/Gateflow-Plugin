---
name: gf-map
description: Map codebase
allowed-tools:
  - Glob
  - Read
  - Write
  - Task
  - Bash
  - Grep
---

# GateFlow Map 커맨드

병렬 분석 에이전트로 SystemVerilog 코드베이스의 포괄적인 맵을 생성합니다.

## 지침

1. 프로젝트의 모든 SystemVerilog 파일을 찾음:
   ```
   Use Glob to find **/*.sv and **/*.svh files
   ```

2. 출력 디렉터리 생성:
   ```bash
   mkdir -p .gateflow/map/modules
   ```

3. Task 도구를 사용해 `gf-architect` 에이전트를 스폰:
   - 발견된 파일 목록을 전달
   - 에이전트가 분석을 위해 10개의 서브 에이전트를 병렬로 스폰
   - 결과는 `.gateflow/map/`에 작성됨

4. 요약과 함께 완료 보고:
   - 발견된 모듈 수
   - 패키지 수
   - 식별된 톱 모듈
   - 경고 (누락 파일, 파싱 오류)

## 출력 구조

```
.gateflow/map/
├── CODEBASE.md          # AI-friendly summary (AGENTS.md style)
├── hierarchy.md         # Module tree with Mermaid
├── signals.md           # Signal flow diagrams
├── fsm.md              # State machine diagrams
├── clock-domains.md    # CDC analysis
├── packages.md         # Package dependencies
├── types.md            # Structs, unions, typedefs
├── functions.md        # Functions and tasks
├── macros.md           # Preprocessor directives
├── verification.md     # SVA, coverage, assertions
├── recipe.md           # Filelists, compile order
├── interfaces.md       # Interfaces, modports, clocking (if found)
├── classes.md          # Classes, UVM hierarchy (if found)
├── generate.md         # Generate blocks (if found)
├── dpi.md              # DPI imports/exports (if found)
└── modules/            # Per-module detail pages
    └── <module_name>.md
```

## 분석 방법

하이브리드 접근법 사용:
1. **토큰 예산 배정** - 파일당 토큰을 세어 ~150k 청크로 그룹화
2. **병렬 에이전트** - 코드베이스 크기에 따라 2-10개 에이전트를 스폰
3. **정규식 파싱** - 포트, 인스턴스, 타입의 빠른 구조적 추출
4. **병합** - 에이전트 컨텍스트 + 정규식 구조를 결합

## 증분 갱신

이후 실행 시:
- 기존 `.gateflow/map/CODEBASE.md`를 감지
- 마지막 스캔 이후 변경 사항을 git 이력에서 확인
- 수정된 파일만 재분석
- 갱신을 기존 맵과 병합
- 다음 증분 실행을 위해 커밋 해시를 저장

첫 실행: 전체 스캔 (큰 코드베이스는 상당한 토큰을 사용할 수 있음)
이후 실행: 빠른 증분 갱신 (변경된 파일만)
