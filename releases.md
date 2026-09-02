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
- **gf-expand**: 정형 검증, 합성, 보드 타겟팅, 프로토콜 선택을 위한 질문 템플릿; 시나리오별 빠른 시작 기본값; 후속 결정 트리
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

## 2.3.0 (2026-03-27) — 품질 개선 + 누락 기능

### 수정
- 5개 IP 블록 README를 스텁에서 인스턴스화 예시가 포함된 전체 문서로 확장
- 5개 block.yaml 파일에 포트 추가 (axi4lite_slave, cdc_handshake, fifo_async, spi_master, uart)
- 누락된 프로토콜 레퍼런스 4개 추가 (UART, Wishbone, AXI4-Full, AXI-Stream)
- Basys 3 제약 확장 (LED 16개 전부, 스위치 16개, 7세그먼트 디스플레이)
- Tang Nano 9K 제약 확장 (UART, SPI 플래시 핀)
- CLAUDE.md 제목을 'Open-Source Hardware Development Platform'으로 갱신
- AGENTS.md를 18개 에이전트 전부로 갱신
- 오케스트레이터 라우팅 표를 Phase 3-4 에이전트로 갱신
- /gf-doctor를 계층별 도구 표시로 갱신 (Core/Formal/Synth/P&R/VHDL/PCB)
- 비어 있는 gf-learn-v2 디렉터리 제거

### 신규 커맨드
- /gf-pcb — KiCad 회로도/PCB 생성
- /gf-pinmap — 핀 제약 파일 생성
- /gf-cocotb — Python Cocotb 테스트벤치 생성
- /gf-fusesoc — FuseSoC .core 파일 생성

## 2.2.1 (2026-03-26) — IP 자동 감지 & 자동 채움

### 신규 기능
- **IP 자동 감지** (`/gf-detect`): 코드베이스에서 누락된 모듈 구현, 스텁 모듈, 표준 IP 패턴을 스캔
- **자동 채움**: 빈틈을 감지하고 검증된 IP 라이브러리 블록을 사용해 구현하도록 에이전트를 파견
- **CDC 위반 스캐닝**: 동기화기가 없는 클럭 도메인 크로싱을 식별 (CRITICAL 심각도)
- **패턴 매칭**: 임시 코드에서 FIFO, CDC, UART, SPI, AXI 패턴을 인식하고 검증된 대체물을 제안
- **서브 에이전트 기능**: sv-ip-scanner는 다른 에이전트가 작업 중에 호출할 수 있는 스킬로 동작

### 신규 에이전트
- `sv-ip-scanner` — IP 블록 감지 및 자동 채움 에이전트

### 신규 커맨드
- `/gf-detect` — 누락 IP, 스텁, CDC 문제 스캔 (`--auto-fill`로 구현)

## 2.2.0 (2026-03-26) — 커뮤니티 + KiCad + 생태계

### 신규 기능
- **KiCad 회로도/PCB 생성** (`/gf-pcb`): DRC/ERC/AI 리뷰 루프, 신뢰도 점수, 필수 고지가 포함된 AI 검증 초안 설계
- **Cocotb 지원** (`gf-cocotb`): SystemVerilog TB의 대안으로 Python 기반 테스트벤치 생성
- **FuseSoC 통합** (`gf-fusesoc`): Edalize 백엔드(Vivado, Quartus, Yosys)를 사용한 .core 파일 생성
- **CI/CD 템플릿**: lint → sim → formal → synthesis를 위한 GitHub Actions 및 GitLab CI 파이프라인
- **맥락 학습**: 워크플로 출력에 삽입된 마이크로 레슨, 개념 추적, 생성형 연습
- **생태계 통합**: F4PGA(Xilinx 7-series 오픈소스), OpenFPGA(커스텀 아키텍처), OpenLane(ASIC 테이프아웃)

### 신규 에이전트
- `pcb-designer` — 자기 개선 검증이 포함된 KiCad 회로도 및 PCB

### 커뮤니티
- 검증 체크리스트가 포함된 보드 기여 가이드
- 검증 파이프라인이 포함된 IP 블록 기여 가이드
- 하드웨어 프로젝트를 위한 CI/CD 템플릿

## 2.1.0 (2026-03-26) — 멀티 HDL + 플랫폼 + 핀 매핑

### 신규 기능
- **VHDL 지원**: GHDL 호환 VHDL-2008을 위한 vhdl-codegen 및 vhdl-testbench 에이전트
- **핀 매핑** (`/gf-pinmap`): 안전 검사가 포함된 보드 인식 제약 파일 생성
- **배치 & 라우팅** (`/gf-pnr`): iCE40/ECP5/Gowin을 위한 nextpnr 통합
- **FPGA 플래시** (`/gf-flash`): openFPGALoader 프로그래밍
- **프로토콜 스캐폴딩**: AXI4-Lite, SPI, I2C, AXI4-Full, AXI-Stream, Wishbone 레퍼런스
- **OpenClaw 통합**: 자율 하드웨어 설계를 위한 ClawHub 스킬로 게시
- **플랫폼 인덱스**: 7개 AI 코딩 플랫폼 + OpenClaw를 위한 통합 가이드

### 신규 에이전트
- `vhdl-codegen` — VHDL 엔티티/아키텍처 생성
- `vhdl-testbench` — GHDL 호환 VHDL 테스트벤치
- `sv-pinmap` — 안전 규칙이 포함된 핀 할당

### 신규 커맨드
- `/gf-pnr` — nextpnr를 사용한 배치 & 라우팅
- `/gf-flash` — openFPGALoader를 통한 FPGA 플래시

## 2.0.0 (2026-03-26) — 정형 검증 + 합성 + IP 라이브러리

### 신규 기능
- **자연어 기반 정형 검증** (`/gf-formal`): 증명할 것을 영어로 설명하면 SVA 프로퍼티 + SymbiYosys 증명을 얻습니다. 킬러 기능.
- **Yosys 합성** (`/gf-synth`): 면적/타이밍 리포트(LUT, FF, BRAM, DSP)와 함께 설계를 합성. 실패하기 전에 지원되지 않는 SV 구문을 경고.
- **IP 블록 라이브러리** (`/gf-ip`): 바로 쓰는 검증된 하드웨어 컴포넌트 8개 — 각각 RTL, 자가 검사 테스트벤치, 정형 프로퍼티, 문서 포함.
- **선별된 보드 데이터베이스** (`/gf-boards`): 전체 제약 파일(.xdc/.pcf/.cst)과 함께 Arty A7, Basys 3, iCEBreaker, Tang Nano 9K의 핀 할당.

### 포함된 IP 블록
- `fifo_sync` — 동기 FIFO (파라미터화된 폭/깊이)
- `fifo_async` — Gray 코드 포인터를 쓰는 비동기 FIFO (CDC)
- `cdc_2ff` — 2-플립플롭 동기화기
- `cdc_handshake` — 멀티비트 핸드셰이크 동기화기
- `uart` — 구성 가능한 보드레이트의 UART TX+RX
- `spi_master` — SPI 마스터 (4가지 CPOL/CPHA 모드 전부)
- `axi4lite_slave` — 바이트 스트로브가 있는 AXI4-Lite 레지스터 슬레이브
- `debouncer` — 에지 감지가 포함된 버튼 디바운서

### 신규 에이전트
- `sv-formal` — 정형 검증 전문가 (SVA + SymbiYosys)
- `sv-synth` — 합성 최적화 전문가 (Yosys)

### 신규 커맨드
- `/gf-formal` — 정형 검증 실행
- `/gf-ip` — IP 블록 라이브러리 관리 (add/list/info)
- `/gf-boards` — 보드 핀아웃 및 세부 정보 조회

## 1.6.0 (2026-03-26)

- plugin.json과 marketplace.json 전반의 모든 버전 문자열을 1.6.0으로 동기화.
- 플러그인과 루트 전반의 BSL-1.1 라이선스 일관성 확인.

## 1.5.3 (2026-02-18)

- 신뢰할 수 있는 lint 넛지를 위해 프롬프트 기반 PostToolUse 훅을 결정적 Python 스크립트로 교체.

## 1.5.2 (2026-02-15)

- 프롬프트 기반 Stop 훅을 결정적 커맨드 훅(`stop-hook.sh`)으로 교체하여 Stop 훅 JSON 검증 실패 수정.
- 플러그인 버전을 1.5.2로 상향.

## 1.5.1 (2025-02-12)

- 프롬프트 기반 훅: PreToolUse(SV 파일 안전), PostToolUse(lint 넛지), Stop(스마트 완료 게이트).
