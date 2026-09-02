# .sby 구성 템플릿

## BMC 템플릿
```sby
[tasks]
bmc

[options]
mode bmc
depth 20
expect pass

[engines]
smtbmc z3

[script]
read -formal design.sv
prep -top top_module

[files]
design.sv
```

## Prove (귀납) 템플릿
```sby
[tasks]
prove

[options]
mode prove
depth 40
expect pass

[engines]
smtbmc z3
abc pdr

[script]
read -formal design.sv
prep -top top_module

[files]
design.sv
```

## Cover 템플릿
```sby
[tasks]
cover

[options]
mode cover
depth 30
expect pass

[engines]
smtbmc z3

[script]
read -formal design.sv
prep -top top_module

[files]
design.sv
```

## 멀티 태스크 템플릿 (BMC + Prove + Cover)
```sby
[tasks]
bmc
prove
cover

[options]
bmc: mode bmc
bmc: depth 20
prove: mode prove
prove: depth 40
cover: mode cover
cover: depth 30
expect pass

[engines]
bmc: smtbmc z3
prove: smtbmc z3
prove: abc pdr
cover: smtbmc z3

[script]
read -formal design.sv
prep -top top_module

[files]
design.sv
```

## .sby 옵션 레퍼런스

| 옵션 | 모드 | 기본값 | 설명 |
|--------|-------|---------|-------------|
| `mode` | all | (필수) | `bmc`, `prove`, `cover`, `live` |
| `depth` | bmc, cover | 20 | 확인할 사이클 수 |
| `timeout` | all | none | 초 단위 타임아웃 |
| `multiclock` | all | off | 다중 클럭 / 비동기 로직 |
| `expect` | all | pass | 예상 결과: pass, fail, unknown |
