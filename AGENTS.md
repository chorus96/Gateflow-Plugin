# GateFlow 문서 인덱스

중요: 사전 학습 기반 추론보다 검색 기반 추론을 우선하세요.
SystemVerilog 질문에 답하기 전에 참조된 파일을 읽으세요.

## 인덱스

```
[Gateflow]|root: .
|primary:{CLAUDE.md}|SV patterns, lint fixes, Spear/Tumbush book index
|readme:{README.md}
|index:{docs/gateflow.index}|compressed file listing
|commands:{commands/gf-*.md}|slash commands
|skills:{skills/*/SKILL.md}|orchestrator, planner, architect, tb-best-practices
|agents:{agents/sv-*.md,agents/vhdl-*.md,agents/pcb-*.md}|codegen, testbench, debug, verification, understanding, refactor, developer, formal, synth, pinmap, ip-scanner, vhdl-codegen, vhdl-testbench, pcb-designer
|hooks:{hooks/hooks.json}
```

## 파일 용도

| 경로 | 용도 |
|------|---------|
| `CLAUDE.md` | SV 패턴, lint 수정, 규칙, Spear/Tumbush 책 인덱스 |
| `skills/gf/SKILL.md` | 메인 오케스트레이터 - 작업 라우팅, 검증 루프 실행 |
| `skills/gf-plan/SKILL.md` | Mermaid 다이어그램이 포함된 하드웨어 설계 플래너 |
| `skills/gf-architect/SKILL.md` | 코드베이스 매핑 및 분석 |
| `agents/sv-*.md` | 특화 에이전트 (codegen, testbench, debug 등) |
| `commands/gf-*.md` | 특정 동작을 위한 슬래시 커맨드 |

## 검색 순서

1. SV 구문/패턴은 `CLAUDE.md` 확인
2. 작업별 지침은 관련 `agents/*.md` 확인
3. 워크플로 오케스트레이션은 `skills/*/SKILL.md` 확인
4. 커맨드 구현은 `commands/*.md` 확인

## 외부 레퍼런스

```
[SystemVerilog for Verification, 3rd ed. — Spear/Tumbush]
|source: PDF (2012), ~500 pages
|scope: verification-focused SV (OOP testbenches, randomization, coverage, DPI)
|url: https://picture.iczhiku.com/resource/eetop/wYIEDKFRorpoPvvV.pdf
|chapters:
|1 Verification Guidelines (p.2)
|2 Data Types (p.26)
|3 Procedural Statements and Routines (p.70)
|4 Connecting the Testbench and Design (p.90)
|5 Basic OOP (p.132)
|6 Randomization (p.170)
|7 Threads and Interprocess Communication (p.266)
|8 Advanced OOP and Testbench Guidelines (p.274)
|9 Functional Coverage (p.324)
|10 Advanced Interfaces (p.364)
|11 A Complete SystemVerilog Testbench (p.386)
|12 Interfacing with C/C++ (p.416)

[SystemVerilog for Design, 2nd ed. — Sutherland, Davidmann, Flake, Moorby]
|source: PDF (2nd ed.)
|scope: design-focused SystemVerilog for hardware design and modeling
|url: https://www.embedic.com/uploads/files/20201009/SystemVerilog%20for%20Design%20Second%20Edition%20A%20Guide%20to%20Using%20SystemVerilog%20for%20Hardware%20Design%20and%20Modeling%20by%20Stuart%20Sutherland,%20Simon%20Davidmann,%20Peter%20Flake,%20P.%20Moorby%20(z-lib.org).pdf?srsltid=AfmBOoqo8WKljRwnv1WRMn6kvrUYSP5dba8nj-XLrlWIk5KkqLNTg-OY
```
