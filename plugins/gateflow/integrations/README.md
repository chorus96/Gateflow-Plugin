# GateFlow 통합

여러 플랫폼과 생태계에서 GateFlow를 사용하기 위한 가이드.

## AI 코딩 플랫폼

| 플랫폼 | 지원 수준 | 가이드 |
|----------|-------------|-------|
| Claude Code | 완전 (네이티브 플러그인) | 내장 |
| OpenCode | MCP + Skills | README 크로스 툴 섹션 참고 |
| Cursor | Rules + Skills | README 크로스 툴 섹션 참고 |
| Cline | Rules | README 크로스 툴 섹션 참고 |
| Windsurf | Rules + Workflows | README 크로스 툴 섹션 참고 |
| Codex CLI | Skills | README 크로스 툴 섹션 참고 |
| Copilot CLI | Instructions | README 크로스 툴 섹션 참고 |

## AI 에이전트 프레임워크

| 프레임워크 | 통합 | 가이드 |
|-----------|------------|-------|
| OpenClaw | MCP를 통한 ClawHub 스킬 | [openclaw.md](openclaw.md) |

## 하드웨어 생태계

| 도구 | 통합 | 상태 |
|------|------------|--------|
| openFPGALoader | `/gf-flash` 커맨드 | 포함됨 |
| Yosys | `/gf-synth` 스킬 | 포함됨 |
| nextpnr | `/gf-pnr` 스킬 | 포함됨 |
| SymbiYosys | `/gf-formal` 스킬 | 포함됨 |
| GHDL | VHDL 시뮬레이션/합성 | Phase 3 |
| Icarus Verilog | Verilog 시뮬레이션 | Phase 3 |
| F4PGA | 대체 FPGA 툴체인 | Phase 4 |
| OpenFPGA | 커스텀 FPGA 아키텍처 | Phase 4 |
| OpenLane | ASIC 테이프아웃 (SKY130/GF180) | Phase 5 |
| KiCad | 회로도/PCB 생성 | Phase 4 |
| Cocotb | Python 테스트벤치 | Phase 4 |
| FuseSoC | 빌드 시스템 통합 | Phase 4 |
