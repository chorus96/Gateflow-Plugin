# OpenLane 통합 — ASIC 테이프아웃

OpenLane은 Efabless를 통한 무료 칩 제작을 위한 완전한 RTL-to-GDSII 흐름입니다.
"자연어 → 검증된 RTL → 실리콘"을 가능하게 합니다.

## 이것이 가능하게 하는 것

- SKY130 (SkyWater 130nm)과 GF180 (GlobalFoundries 180nm)을 대상으로 하는 ASIC 테이프아웃
- 전체 흐름: RTL → 합성 → 플로어플랜 → 배치 → 라우팅 → 사인오프 → GDSII
- Efabless shuttle 런을 통한 무료 제작

## 궁극의 데모

"영어로 칩을 설명하고 무료로 테이프아웃했습니다."

## 설정

```bash
# Install OpenLane
pip install openlane
# Or Docker: docker pull efabless/openlane2

# Install PDK
volare enable --pdk sky130 <version>
```

## GateFlow와 함께 사용 (예정)

```
/gf-tapeout rtl/top.sv --pdk sky130 --die-area "0 0 500 500"
```

## 상태: Phase 5+ (예정)

이 통합에는 다음이 필요합니다:
- OpenLane 전문성과 신중한 검증
- PDK별 설계 규칙
- ASIC 규모의 타이밍 클로저
- 광범위한 DRC/LVS 검증
