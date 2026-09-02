---
name: gf-learn-ctx
description: >
  Contextual learning — micro-lessons embedded in workflow output.
  Explains hardware concepts on first encounter, tracks what has been
  taught, generates practice exercises on demand.
  Used internally by orchestrator — not typically invoked directly.
user-invocable: false
---

# GF-Learn-Ctx — 맥락 학습

## 철학

교육은 별도 모드가 아니라 워크플로 안에서 일어납니다.
사용자가 개념을 처음 만날 때 간략히 설명합니다.
같은 개념을 두 번 설명하지 않습니다.

## 첫 만남 설명

GateFlow가 어떤 개념을 사용하는 코드를 처음 생성할 때,
[Learn]으로 표시된 2-3문장 설명을 포함:

```
Generated sync_2ff.sv

[Learn] This is a 2-flip-flop synchronizer. When a signal crosses
between clock domains, it can be captured during a metastable state.
Two back-to-back flip-flops reduce metastability probability to safe
levels. For multi-bit signals, use a Gray code FIFO or handshake instead.
```

## 개념 추적

설명 전에 `~/.gateflow/profile.json`을 확인:

```json
{
  "concepts_introduced": ["cdc", "fsm", "pipeline", "fifo", "axi"]
}
```

개념이 목록에 있으면 설명을 건너뜀.
설명 후, 개념을 목록에 추가.

## 개념 라이브러리

| 개념 | 트리거 | 설명 초점 |
|---------|---------|-------------------|
| cdc | 2FF sync, async FIFO | 준안정성, 왜 FF 2개인지 |
| fsm | 상태 머신 코드 | 2-프로세스 패턴, 인코딩 |
| pipeline | 파이프라인 레지스터, 스테이지, valid_q | 더 높은 클럭 주파수를 위해 긴 경로를 스테이지로 분할 |
| fifo | FIFO 인스턴스화 | full/empty, 포인터 연산 |
| axi | AXI 인터페이스 | Valid/ready 핸드셰이크 규칙 |
| formal | SVA 프로퍼티 | 유계 증명, 반례 |
| synthesis | Yosys 출력 | LUT vs FF, 리소스 의미 |
| constraints | 핀 매핑 | IOSTANDARD, 전압 레일 |
| vhdl | VHDL 코드 | SystemVerilog와의 비교 |
| backpressure | ready 신호, stall | 소비자가 ready 해제로 생산자에게 멈추라고 알림 |
| clock_gating | gclk, ICG 셀 | 유휴 레지스터로의 클럭을 멈춰 동적 전력 절약 |
| reset_synchronizer | rst_sync, 비동기 assert/동기 deassert | 리셋 해제 시 준안정성 방지 |
| gray_code | bin2gray, gray2bin | 증가당 한 비트만 변경, CDC 포인터에 안전 |
| dual_port_ram | sdp_ram, true dual-port | 독립 접근 포트 2개, FPGA BRAM 네이티브 |
| arbitration | arbiter, grant, round-robin | 공유 자원에 대한 경합 해결 |
| bus_protocol | AXI, Wishbone, APB | 마스터/슬레이브 통신 규칙 정의 |
| timing_closure | slack, critical path | 모든 경로가 목표 주파수에서 setup/hold 충족 |
| register_file | regfile, read/write 포트 | 다중 읽기 포트가 있는 작고 빠른 메모리 |
| shift_register | LFSR, serial-to-parallel | 클럭당 한 위치씩 데이터 이동 |
| priority_encoder | casez, leading zero | 최우선 활성 비트의 인덱스를 출력 |
| barrel_shifter | rotate, variable shift | mux 캐스케이드를 통한 가변 거리 시프트 |
| interrupt_controller | IRQ, pending, mask | 인터럽트 소스를 집계하고 우선순위 지정 |
| dma | descriptor, scatter-gather | CPU 없이 메모리 간 데이터 전송 |

## 생성형 연습

사용자가 연습을 요청할 때:
1. 최근 설계의 버그 있는 버전을 생성
2. 사용자에게 오류를 찾으라고 요청
3. 설명과 함께 답을 공개

예시:
```
Want to practice? Here's a FIFO with a subtle bug.
Find what's wrong:

[buggy code]

Hint: Look at the full/empty logic...
```

## 크로스 HDL 설명

SV 사용자가 VHDL을 처음 볼 때:
```
[Learn] VHDL uses 'signal' instead of 'logic', and 'process'
instead of 'always_ff'. The semantics are similar — sequential
logic with sensitivity lists. VHDL is more verbose but equally
capable for synthesis.
```

## 상호 참조 링크

개념을 설명할 때, 관련 GateFlow 링크를 덧붙임:
- CDC -> "Use `/gf-ip add cdc_2ff` for a drop-in synchronizer."
- FIFO -> "Use `/gf-ip add fifo_async` for a verified async FIFO."
- Protocol -> "See `/gf-protocols` for AXI/SPI/UART references."
- Practice -> "Try `/gf-learn <topic>` for exercises."

## 간격 반복

| 마지막 사용 이후 세션 수 | 조치 |
|---|---|
| 3-5 | 간단한 리마인더 (1문장) |
| 6-10 | 중간 리마인더 (2문장 + 링크) |
| 11+ | 전체 재설명 |

## 통합 훅

/gf 오케스트레이션의 다음 지점에 마이크로 레슨을 주입:
- 코드 생성 후: 생성된 코드에서 새 개념을 스캔
- ip-add 후: IP 블록 배후의 개념을 설명
- lint 실패 후: 오류가 개념과 관련되면 그 개념을 가르침 (예: latch = 불완전한 case)
- sim 실패 후: 실패가 X 전파 또는 CDC와 관련되면 설명
- 계획 후: 사용자가 보지 못한 개념을 소개
