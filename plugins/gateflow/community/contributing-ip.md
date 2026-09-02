# IP 블록 기여

검증된 IP 블록을 GateFlow 라이브러리에 제출하세요.

## 요구 사항

모든 IP 블록은 반드시 다음을 포함해야 합니다:
1. `rtl/*.sv` — lint 클린 RTL (Verilator -Wall, 경고 0)
2. `tb/tb_*.sv` — 자가 검사 테스트벤치 (pass/fail 카운터)
3. `formal/*_props.sv` — SVA formal 프로퍼티
4. `formal/*.sby` — SymbiYosys 구성
5. `block.yaml` — 메타데이터 (name, version, parameters, ports)
6. `README.md` — 인스턴스화 예시가 있는 사용 가이드

## 검증 파이프라인

제출된 블록은 다음을 통과해야 합니다:
```bash
# Lint
verilator --lint-only -Wall rtl/*.sv

# Simulation
verilator --binary -j0 --trace --top-module tb_<name> -Irtl tb/*.sv rtl/*.sv
./obj_dir/Vtb_<name>

# Formal (if SymbiYosys available)
sby -f formal/<name>.sby
```

수락 전에 세 가지 모두 PASS해야 합니다.

## block.yaml 스키마

```yaml
name: block_name
version: 1.0.0
description: One-line description
parameters:
  PARAM_NAME: { type: int, default: 8, description: "..." }
ports:
  - { name: clk, dir: input, width: 1 }
formal_proofs:
  - property_name: "What it proves"
dependencies: []  # Other IP blocks required
```
