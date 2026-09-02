# OpenClaw 통합 가이드

GateFlow는 OpenClaw 스킬로 제공되어, OpenClaw AI 에이전트 프레임워크를 통해
자율 하드웨어 설계를 가능하게 합니다.

## 이것이 가능하게 하는 것

- **자율 하드웨어 설계**: "Design me an SPI controller for my Arty A7"
- **상시 하드웨어 CI**: OpenClaw가 저장소를 모니터링하고 lint/sim/formal을 자동 실행
- **다중 에이전트 워크플로**: OpenClaw 오케스트레이션 + GateFlow 하드웨어 에이전트

## ClawHub에 게시

GateFlow는 ClawHub에서 검증된 스킬로 게시됩니다 (보안을 위해 커뮤니티 제출이 아님
— 커뮤니티 스킬에 대한 Bitdefender 감사 결과 참고).

### 스킬 구성

```yaml
# openclaw-skill.yaml
name: gateflow
description: AI-powered hardware development — RTL, verification, synthesis, FPGA deployment
version: 2.1.0
author: codejunkie99
category: development
tags: [hardware, fpga, rtl, systemverilog, vhdl, synthesis, formal-verification]

triggers:
  - "design * hardware"
  - "create * module"
  - "synthesize"
  - "formal verify"
  - "FPGA"
  - "SystemVerilog"
  - "VHDL"

mcp_server:
  transport: stdio
  command: claude
  args: ["--plugin-dir", "~/.claude-plugins/gateflow/plugins/gateflow", "--mcp"]

capabilities:
  - rtl_generation
  - testbench_generation
  - formal_verification
  - synthesis
  - pin_mapping
  - board_targeting
```

### MCP 인터페이스

OpenClaw는 MCP(Model Context Protocol)를 통해 GateFlow와 통신합니다:

```json
{
  "method": "tools/call",
  "params": {
    "name": "gateflow_create",
    "arguments": {
      "description": "Create an SPI controller with testbench",
      "board": "arty-a7-35t",
      "hdl": "systemverilog",
      "options": {
        "formal": true,
        "synthesize": true
      }
    }
  }
}
```

## 보안

- GateFlow 스킬은 ClawHub에서 **검증됨/공식**으로 게시됨
- 모든 하드웨어 파괴적 작업은 명시적 사용자 확인이 필요:
  - `/gf-flash` (FPGA 프로그래밍)
  - 웹 검색에서의 핀 매핑
  - 파일 덮어쓰기
- OpenClaw의 샌드박스가 GateFlow 실행을 격리

## 생태계 조합

| OpenClaw + GateFlow + ... | 결과 |
|---------------------------|--------|
| KiCad | 자연어 → 회로도 |
| openFPGALoader | 자연어 → 동작하는 하드웨어 |
| GitHub Actions | 자동화된 하드웨어 CI/CD |
| Ollama (로컬 LLM) | 완전 오프라인 하드웨어 개발 |
