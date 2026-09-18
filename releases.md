# 릴리스

## 2.5.3 (2026-05-21) — 커맨드 CLI + 에이전트 생성

로컬 터미널 화면을 진짜 CLI처럼 동작하게 만드는 패치 릴리스.

### 신규 기능
- `status`, `tui`, `agents list`, `agents create`, `shell` 서브커맨드가 포함된
  `tools/gateflow_cli.py` 추가.
- `plugins/gateflow/agents/` 아래에 Claude 호환 에이전트 마크다운을 작성하는
  로컬 에이전트 생성 기능 추가.
- 대시보드를 벗어나지 않고 새 에이전트를 만드는 TUI 내부의 `a` 키 추가.

### 다듬기
- 터미널 색상을 의미론적 상태 중심으로 재정비: copper 정체성, cyan
  헤딩/내비게이션, green 준비 상태, amber 경고, 그리고 흐린 구분선.
- 에이전트 생성, CLI JSON 상태, 셸 도움말, 풍부한 curses 색상 쌍 설정,
  적층된 헬스 로우 스타일에 대한 회귀 테스트 커버리지 추가.

## 2.5.2 (2026-05-21) — 반응형 TUI 레이아웃

좁은 터미널 창을 위한 패치 릴리스.

### 수정
- 100컬럼 미만 터미널을 위한 적층형 대화 레이아웃 추가로, 액션 목록이 더 이상
  워크스페이스, 인벤토리, 헬스 패널과 겹치지 않음.
- 액션 설명 및 기타 터미널 텍스트를 사용 가능한 컬럼 폭에 맞게 잘라냄.
- 좁은 터미널 레이아웃 선택과 텍스트 말줄임에 대한 회귀 테스트 커버리지 추가.

## 2.5.1 (2026-05-21) — TUI 터미널 호환성

GateFlow 터미널 콘솔을 위한 패치 릴리스.

### 수정
- 대화형 `/gf-tui` curses 경로가 `curs_set(0)`을 거부하는 터미널도 견디도록 함.
- 사용 가능한 curses 색상 쌍이 없는 PTY를 위한 색상 폴백 처리 추가.
- 커서 및 색상 기능 실패에 대한 회귀 테스트 커버리지 추가.
- 릴리스 검증기 기본값과 TUI 릴리스 검사가 플러그인 매니페스트 버전을 따르도록 갱신.

## 2.5.0 (2026-05-21) — CLI/TUI + 릴리스 준비 상태

이제 GateFlow는 결정적 릴리스 준비 경로를 제공하여 버전이 매겨진 플러그인
업데이트를 태깅 전에 검사할 수 있으며, 사용자가 현대적인 에이전트 CLI에서
기대하는 상태 우선 워크플로를 위한 로컬 터미널 콘솔도 함께 제공합니다.

### 신규 기능
- 로컬 OpenClaw 스타일 터미널 콘솔을 위한 `/gf-tui` 커맨드 추가.
- 대화형, 스냅샷, JSON 콘솔 모드를 다루는 `gf-tui` 스킬 추가.
- TTY 인식 렌더링, 일반 출력, JSON 출력, 컴포넌트 인벤토리, 도구 헬스, 맵 준비 상태,
  릴리스 준비 상태, 커맨드 단축키를 갖춘 stdlib Python TUI인 `tools/gateflow_tui.py` 추가.
- 릴리스 검증 및 버전 준비 워크플로를 위한 `/gf-release` 커맨드 추가.
- semver 지침, 필수 릴리스 검사, 구조화된 `GATEFLOW-RESULT` 리포팅을 갖춘
  `gf-release` 스킬 추가.
- 플러그인/마켓플레이스 버전, README 개수, 문서 인덱스 커버리지, 루트 미러,
  릴리스 노트를 검증하는 `tools/validate_gateflow.py` 추가.
- 릴리스 검증기와 터미널 콘솔을 위한 집중 unittest 스위트 추가.

### 패키지 일관성
- 플러그인 및 마켓플레이스 메타데이터를 `2.5.0`으로 동기화.
- 현재의 모든 커맨드, 스킬, 에이전트, IP 블록, 보드, 훅, 릴리스 도구를 발견할 수 있도록
  `docs/gateflow.index` 갱신.
- 크로스 툴 설치가 Claude 플러그인과 동일한 진입점 커버리지를 갖도록 20개 에이전트와
  27개 스킬 전부에 대한 루트 레벨 미러 완성.
- 루트 및 플러그인 README를 20개 에이전트, 27개 스킬, 21개 커맨드, 8개 IP 블록,
  4개 보드를 반영하도록 갱신.

## 2.4.0 (2026-04-11) — 심층 스킬 보강

GateFlow 역사상 가장 큰 콘텐츠 업데이트. 플러그인의 모든 스킬이 연구 기반 레퍼런스 자료, 실행 가능한 템플릿, 구조화된 반환 형식으로 보강되었습니다. 37개 파일 변경, +3,139줄 추가.

### 핵심 수정
- 오래된 `gateflow.index` 재구축 (누락된 스킬 11개 이상 추가, 존재하지 않는 gf-summary 제거)
- gf-architect에서 하드코딩된 "Sonnet" 모델 참조 제거 (이제 세션 모델을 상속)
- 누락된 gf-errors 3계층 번역 프로토콜을 gf/SKILL.md에 인라인 삽입
- gf-router의 끊긴 라우팅 대상 수정 (gf-synth -> gateflow:sv-synth agent, gf-boards -> command)
- CLAUDE.md 콘텐츠 중복 제거 (세션당 약 3K 토큰 절약)

### 스킬 보강 — 검증 & 합성 (스킬 7개)
- **gf-formal**: SVA 프로퍼티 패턴(오버플로, 핸드셰이크, one-hot, liveness, 리셋, FIFO), SymbiYosys .sby 템플릿(BMC/prove/cover/multi-task), 증명 전략 결정 트리, 엔진 비교, 반례 해석 가이드, 레퍼런스 파일 4개
- **gf-cocotb**: FIFO/핸드셰이크/FSM 테스트 템플릿, 다중 시뮬레이터 Makefile, pytest 러너, cocotbext 프로토콜 라이브러리(AXI/Wishbone/SPI/UART), Cocotb vs SV 결정 매트릭스, GATEFLOW-RESULT 형식
- **gf-pcb**: 전체 KiCad CLI 레퍼런스(DRC/ERC/BOM/gerbers/drill/STEP), DRC/ERC 오류 해석 표, AI 리뷰 체크리스트(디커플링/전원/고속/열/제조), 신뢰도 점수 프레임워크, 제조 패키지 스크립트
- **gf-pnr**: 완전한 nextpnr 플래그 레퍼런스(iCE40/ECP5/Gowin), 전체 비트스트림 파이프라인, 타이밍 분석(Fmax/slack/critical path), 사용률 임계값, 흔한 실패와 수정, 제약 파일 형식 템플릿(PCF/LPF/CST)
- **gf-fusesoc**: 다중 타겟 구성이 포함된 완전한 CAPI=2 .core 스키마, 파일 타입 레퍼런스, 버전 연산자를 사용한 의존성 관리, FuseSoC CLI 커맨드, 도구 백엔드 옵션
- **gf-lint**: Verilator v5 신규 경고 코드 20개 이상, 경고 억제(인라인 + 컨트롤 파일), SARIF 기계 판독 출력, 오류 메시지 형식과 종료 코드
- **gf-sim**: Verilator v5 멀티스레드 시뮬레이션, FST vs VCD 추적, SVA 어서션 지원, 코드 커버리지 플래그, SV 구문 지원 매트릭스, 타임아웃 패턴

### 스킬 보강 — 오케스트레이션 (스킬 4개)
- **gf**: Verilator/Yosys 오류 사전(평이한 영어 번역과 수정이 있는 패턴 20개)
- **gf-router**: 신뢰도 부스트를 갖춘 컨텍스트 의존 라우팅, 복합 질의를 위한 다중 의도 감지, 적응형 신뢰도 보정 공식
- **gf-expand**: Formal 검증, 합성, 보드 타겟팅, 프로토콜 선택을 위한 질문 템플릿; 시나리오별 빠른 시작 기본값; 후속 결정 트리
- **gf-build**: 최적 병렬 단계 배정을 위한 Kahn 알고리즘, 리소스 경합 방지 규칙, 의존성 인식 캐시 무효화를 갖춘 SHA256 기반 증분 빌드, 진행 상황 시각화

### 스킬 보강 — 아키텍처 & 시각화 (스킬 2개)
- **gf-architect**: 정확한 매핑을 위한 Verilator v5 JSON AST 모드, RTL 복잡도 지표(순환 복잡도 + 복합 점수), 모듈 간 신호 추적 알고리즘, CHANGES.md 출력을 갖춘 diff 인식 매핑, Mermaid 의존성 그래프 생성
- **gf-viz**: 신호 경로 추적 뷰, ASCII 타이밍 다이어그램, 구조적 diff 뷰, 포트 연결 매트릭스, 검색 기능(클럭/FSM/포트로 모듈 찾기, glob로 신호 찾기, 미연결 포트, CDC 크로싱)

### 스킬 보강 — 학습 & 검증 실무 (스킬 3개)
- **gf-learn**: 신규 주제 카테고리 5개(arbiter, memory, protocol, verification, optimization), 주제 간 전이가 있는 난이도 스케일링 알고리즘, 자동 검사가 포함된 4단계 채점 루브릭, JSON 진행 상황 지속, 시간 채점이 있는 챌린지 모드
- **gf-learn-ctx**: 신규 개념 라이브러리 항목 15개(pipeline, backpressure, clock gating, Gray code, arbitration, DMA 등), IP 블록 및 스킬로의 상호 참조 링크, 간격 반복 알고리즘, 오케스트레이션 통합 훅 5개
- **tb-best-practices**: Verilator v5용 UVM-lite 호환성, 모든 SV TB 패턴의 완전한 Cocotb 대응물, 4단계 커버리지 클로저 체크리스트, 수정이 포함된 흔한 TB 안티패턴 10개, 시뮬레이션 성능 최적화 가이드

### 스킬 보강 — IP & 프로토콜 (스킬 3개)
- **gf-protocols**: AXI4-Lite slave, SPI master, UART TX, I2C master의 실제 스캐폴드 코드(이전에는 코드가 전혀 없는 빈 스텁이었음), GATEFLOW-RESULT 형식
- **gf-ip**: 8개 IP 블록 전부의 인스턴스화 예시, IP 블록 비교/결정 표, IP 작업용 GATEFLOW-RESULT 형식
- **gf-ip-detect**: 확장된 벤더 IP 감지(Xilinx UltraScale+, Intel Agilex, Microchip PolarFire, Efinix), 오탐 감소 규칙, 5단계 심각도 점수, 패턴을 `/gf-ip add` 커맨드에 매핑하는 자동 제안 통합

### 스킬 보강 — 하드웨어 & 계획 (스킬 3개)
- **gf-pinmap**: GATEFLOW-RESULT 형식, I/O 표준 레퍼런스 표(LVCMOS33부터 TMDS_33까지), 흔한 핀 매핑 실수 6개, PMOD 커넥터 매핑 패턴(Type 2/3/6)
- **gf-project**: GATEFLOW-RESULT 형식, 확장된 project.yaml 스키마(simulation/synthesis/verification/CI 섹션), 프로젝트 템플릿(iCE40/ECP5/sim-only/multi-board), 헬스 체크 검증
- **gf-plan**: 활동 계수와 클럭 게이팅 후보를 사용한 전력 추정, RTL 구문별 면적 추정, 지연 예산 규칙, 심각도 매트릭스가 포함된 위험 평가 템플릿

### README
- 전체 범위(20개 에이전트, 25개 스킬, 19개 커맨드, 8개 IP 블록, 4개 보드)를 보여주는 대담한 빌더 목소리로 전면 재작성
- v2.4.0 업데이트 SVG 그래픽 추가
