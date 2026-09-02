---
name: gf-pcb
description: >
  KiCad schematic and PCB generation from natural language.
  AI-verified drafts with DRC/ERC/AI review loop.
  Example: "design a breakout board for iCE40 with SPI flash and 2 PMODs"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - Task
  - WebSearch
  - WebFetch
---

# GF-PCB — KiCad 회로도 & PCB 생성

## 도구 감지

```bash
which kicad-cli
```

찾을 수 없으면:
```
---GATEFLOW-RESULT---
STATUS: ERROR
DETAILS: KiCad not installed. Install for PCB design.
  macOS: brew install --cask kicad
  Linux: sudo apt install kicad
  Download: https://www.kicad.org/download/
---END-GATEFLOW-RESULT---
```

## 워크플로

1. **요청 파싱** — 어떤 보드? 부품? 제약?
2. **웹 검색** — 레퍼런스 설계, 데이터시트, 풋프린트
3. **pcb-designer 에이전트 스폰** — 회로도 + PCB 생성
4. **검증 루프 실행**:
   a. KiCad DRC (Design Rule Check)
   b. KiCad ERC (Electrical Rule Check)
   c. AI 리뷰 (전원, 디커플링, 플로팅 핀, 전압)
   d. 오류가 있으면 → 수정 후 재검증 (최대 3회 반복)
5. 검증 상태와 신뢰도로 출력 **라벨링**
6. 필수 고지와 함께 **전달**

## 출력 파일

```
project/
├── schematic.kicad_sch     # Schematic (S-expression)
├── board.kicad_pcb         # PCB layout
├── bom.csv                 # Bill of materials
└── verification_report.md  # DRC/ERC/AI review results
```

## 결과 형식

```
---GATEFLOW-RESULT---
STATUS: PASS | FAIL | ERROR
CONFIDENCE: high | medium | low
VERIFICATION:
  DRC: PASS/FAIL (N errors, M warnings)
  ERC: PASS/FAIL (N errors, M warnings)
  AI_REVIEW: PASS/FAIL (N/M checks passed)
FILES: [generated files]
DETAILS: [summary]
---END-GATEFLOW-RESULT---
```

## KiCad CLI 빠른 참조

```bash
# DRC
kicad-cli pcb drc --format json --severity-all --exit-code-violations --output drc.json board.kicad_pcb

# ERC
kicad-cli sch erc --format json --severity-all --exit-code-violations --output erc.json design.kicad_sch

# BOM
kicad-cli sch export bom --fields "Reference,Value,Footprint,${QUANTITY},Manufacturer,MPN" \
  --group-by "Value,Footprint" --exclude-dnp --output bom.csv design.kicad_sch

# Gerbers + Drill
kicad-cli pcb export gerbers --output gerbers/ board.kicad_pcb
kicad-cli pcb export drill --output gerbers/ --format excellon board.kicad_pcb

# 3D Model
kicad-cli pcb export step --output board.step board.kicad_pcb
```

종료 코드: 0 = 클린, 5 = 위반 발견.

## DRC/ERC 해석

### 흔한 DRC 오류
| 오류 | 의미 | 수정 |
|---|---|---|
| clearance_violation | 항목이 너무 가까움 | 간격 늘리기 |
| shorting_items | 다른 네트가 접촉 | 트레이스 재라우팅 |
| courtyards_overlap | 부품이 너무 가까움 | 떨어뜨리기 |
| track_width | 허용 범위 밖 | 폭 조정 |
| annular_width | Via 링이 너무 작음 | Via 크기 늘리기 |
| copper_sliver | 얇은 구리 (제조 위험) | pour 조정 |
| hole_near_hole | 드릴 홀이 너무 가까움 | 간격 늘리기 |

### 흔한 ERC 오류
| 오류 | 수정 |
|---|---|
| Input power pin not driven | PWR_FLAG 추가 |
| Pin not connected | 연결하거나 no-connect X 추가 |
| Conflicting pin types | 심볼 핀 타입 확인 |
| Duplicate reference | 회로도 재주석 |

## AI 리뷰 체크리스트

### 디커플링
- [ ] 모든 IC: 전원 핀 5mm 이내에 100nF 캡
- [ ] 전원 진입부 근처에 벌크 캡
- [ ] 전류 경로: 공급 -> 캡 -> IC 핀

### 전원
- [ ] 연속적인 그라운드 플레인
- [ ] 고속 신호 아래 그라운드를 자르는 신호 트레이스 없음
- [ ] 전류에 충분한 구리 두께

### 고속
- [ ] >50MHz 신호에 임피던스 제어
- [ ] 차동 쌍의 길이 매칭
- [ ] 고속 경로를 따라 via 스티칭

### 열
- [ ] 노출 패드 아래 열 via (최소 4-9)
- [ ] 열원을 온도 민감 부품에서 멀리

### 제조
- [ ] Edge.Cuts에 보드 외곽선 닫힘
- [ ] 최소 트레이스 >= 0.1mm, 최소 드릴 >= 0.2mm
- [ ] 실크스크린이 패드 위에 없음
- [ ] 피듀셜 존재 (최소 3)

## 신뢰도 점수

| 레벨 | 기준 | DRC 기대 |
|---|---|---|
| HIGH | 부품 <50, 2층, <10MHz | 위반 0 |
| MEDIUM | 부품 50-200, 2-4층, <100MHz | 조정 필요 가능 |
| LOW | 부품 >200, 4층 이상, 고속/RF/전력 | 전문가 리뷰 필요 |

다음의 경우 LOW로 재정의: RF/안테나, DDR/PCIe/USB3+, SMPS >5W, BGA >100 핀.

## 제조 패키지 스크립트

```bash
mkdir -p output/{gerbers,assembly,docs}
kicad-cli pcb export gerbers --output output/gerbers/ board.kicad_pcb
kicad-cli pcb export drill --output output/gerbers/ --format excellon board.kicad_pcb
kicad-cli pcb export pos --format csv --output output/assembly/positions.csv board.kicad_pcb
kicad-cli sch export bom --exclude-dnp --output output/assembly/bom.csv design.kicad_sch
kicad-cli sch export pdf --output output/docs/schematic.pdf design.kicad_sch
kicad-cli pcb export step --output output/docs/board.step board.kicad_pcb
```
