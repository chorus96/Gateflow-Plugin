# Phase 3: 체리피킹 + Taste Diff

## 목표
taste 프로필 대비 코드베이스를 점수화하고, 인스퍼레이션에서 특정 컴포넌트를 체리픽합니다.

**의존성**: Phase 1 타입만 (디스크에서 TasteProfile을 읽으며 Phase 2에서 import하지 않음)

## 새 파일: `src/tasteDiff.ts` (~200줄)

### 핵심 함수

**`computeTasteDiff(codebaseTokens, taste)`** → `TasteDiffResult`
- 코드베이스 `ScanTokens`를 `TasteProfile`과 비교
- 정렬 점수(0-100)와 실행 가능한 델타를 반환

**`scoreTaste(options)`** → `{ scanResult, diff }`
- `scanDesignSystem` + `computeTasteDiff`를 결합
- Options: `{ rootDir, projectId, scanPath }`

**`cherryPickComponent(options)`** → `ComponentCherryPick`
- 인스퍼레이션에서 컴포넌트를 체리픽
- Options: `{ rootDir, projectId, componentKind, sourceUrlOrId, index?, note? }`

**`colorDistance(hex1, hex2)`** → `number` (비공개)
- taste 매칭을 위한 단순화된 HSL 델타

**`closestTasteColor(hex, palette)`** → `{ hex, distance }` (비공개)
- 주어진 hex에 가장 가까운 taste 팔레트 색상을 찾음

### Diff 알고리즘 세부 사항

#### 색상 정렬 (0-100)
- 각 코드베이스 색상마다 HSL 거리로 가장 가까운 taste 팔레트 색상을 찾음
- 거리 < 15 = 정렬됨, 15-30 = 근접, >30 = 불일치
- 점수 = (정렬 + 근접*0.5) / 전체 * 100
- 델타: 가장 가까운 제안과 함께 불일치 색상을 나열

#### 타이포그래피 정렬 (0-100)
- 코드베이스 폰트가 {primaryFont, secondaryFont}에 속하는지 확인 → 매치당 100
- 추가 폰트에는 페널티: 추가 폰트 패밀리당 -20
- 크기 정렬: 코드베이스 크기가 taste 스케일에 속하는지 확인 → 보너스

#### 간격(spacing) 정렬 (0-100)
- 코드베이스 간격 값을 px로 파싱
- 각각을 taste `spacing.scale`과 대조 (±2px 허용)
- 점수 = 스케일에 맞는 값의 %

#### 모션 정렬 (0-100)
- Duration 매치: 코드베이스 durations가 taste durations에 속함 → 50점
- Easing 매치: 코드베이스가 유사한 easing을 쓰는지 확인 → 30점
- Intensity 매치: 모션 개수가 강도 레벨과 일치 → 20점

### `sourceUrlOrId`의 체리픽 해석 체인
1. `InspirationRecord.id`와 정확히 일치
2. `InspirationRecord.url`과 정확히 일치
3. 호스트명 일치 (도메인 추출, 같은 도메인의 인스퍼레이션을 찾음)

### TUI 렌더링 (`src/tasteRenderer.ts`에 추가)

**`renderTasteDiff(diff)`** — 정렬 막대와 델타를 렌더링:

```
  ┌─ Taste Alignment ────────────────────────────┐
  │                                                │
  │  Color           ████████████░░░░  78          │
  │  Typography      ████████████████  100         │
  │  Spacing         ██████████░░░░░░  65          │
  │  Motion          ████████░░░░░░░░  50          │
  │                                                │
  │  Overall         ████████████░░░░  73          │
  └────────────────────────────────────────────────┘

  Deltas:
  ⚠ Color #FF5733 not in taste palette → closest: #635BFF
  ⚠ Font "Roboto" not in taste → expected: Inter, GT America
  ✔ Spacing 92% on taste grid
```

## 재사용할 기존 함수

| 함수 | 파일 |
|----------|------|
| `scanDesignSystem()` | `src/scan.ts` |
| `loadTasteProfile()` | `src/store.ts` |
| `loadDatabase()`, `findProject()` | `src/store.ts` |

## 검증

```bash
npm run build && npm test
# Manual: node dist/cli.js taste score . --project demo
```

## 상태
- [ ] 대기 중
