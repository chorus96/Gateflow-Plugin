---
name: gf-ip
description: Manage IP block library
argument-hint: "add|list|info <block>"
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
---

# GateFlow IP Library 커맨드

## 사용법

```
/gf-ip list                  # Show all available IP blocks
/gf-ip info fifo_sync        # Show block details and ports
/gf-ip add fifo_sync         # Install block into current project
```

## 실행

### list
모든 `${CLAUDE_PLUGIN_ROOT}/ip/*/block.yaml` 파일을 읽음.
표시: 이름, 설명, 검증 상태.

### info <block>
`${CLAUDE_PLUGIN_ROOT}/ip/<block>/block.yaml`을 읽음.
표시: 설명, 파라미터, 포트, formal 증명, 의존성.

### add <block>
1. 메타데이터를 위해 block.yaml을 읽음
2. `rtl/*.sv`를 프로젝트 `rtl/` 디렉터리로 복사
3. `tb/*.sv`를 프로젝트 `tb/` 디렉터리로 복사
4. `formal/*`를 프로젝트 `formal/` 디렉터리로 복사
5. `.gateflow/project.yaml` 갱신 — `ip_blocks` 목록에 추가
6. 무엇이 설치되었고 어떻게 인스턴스화하는지 보고
