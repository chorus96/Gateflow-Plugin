# Phase 4: 대화형 정제

## 목표
인스퍼레이션 간 충돌을 감지하고 명확화 질문을 생성합니다. 사용자 결정을 적용하여 taste 프로필을 정제합니다.

**의존성**: Phase 1 타입만

## 새 파일: `src/tasteRefine.ts` (~150줄)

### 핵심 함수

**`detectConflicts(inspirations)`** → `TasteConflict[]`
- 인스퍼레이션 전반의 충돌을 감지
- Phase 2의 `buildTasteProfile`이 호출하지만, 독립적으로도 호출 가능

**`nextClarifyingQuestion(profile)`** → `{ conflict, question, options } | null`
- 해결되지 않은 충돌로부터 다음 명확화 질문을 생성
- 남은 충돌이 없으면 null 반환

**`applyDecision(options)`** → `TasteDecision`
- 충돌에 대한 사용자의 답변을 적용하여 `TasteDecision`을 생성
- 충돌을 해결됨으로 표시
- Options: `{ rootDir, projectId, conflictIndex, answer }`

### 충돌 감지 규칙

#### 색상 충돌
- 각 인스퍼레이션의 상위 5개 색상을 소스별로 그룹화
- 서로 다른 소스의 상위 3개 색상이 HSL 거리 > 40이면 → 충돌
- 설명: "인스퍼레이션들이 서로 다른 주요 색상을 보입니다"
- 옵션: 소스 이름과 함께 각 소스의 최상위 색상을 나열

#### 타이포그래피 충돌
- 각 인스퍼레이션에서 주 폰트(가장 빈번한 것)를 추출
- 서로 다른 인스퍼레이션이 서로 다른 주 폰트를 쓰면 → 충돌
- 설명: "서로 다른 인스퍼레이션이 서로 다른 주 폰트를 사용합니다"
- 옵션: 소스 이름과 함께 각 폰트를 나열

#### 간격(spacing) 충돌
- 인스퍼레이션별 기본 단위 감지 (가장 흔한 약수: 4 또는 8)
- 기본 단위가 다르면 → 충돌

#### 형태 충돌 (border-radius)
- 각 인스퍼레이션 컴포넌트에서 가장 흔한 border-radius를 추출
- 서로 다른 값 주위로 몰리면(>2px 차이) → 충돌
- 설명: "인스퍼레이션들이 서로 다른 모서리 반경을 보입니다"
- 옵션: "8px (stripe.com)", "6px (linear.app)", "12px (vercel.com)"

#### 모션 충돌
- 각 인스퍼레이션에서 주된 duration을 추출
- durations가 크게 다르면(>100ms) → 충돌

### 질문 형식
```
Your inspirations show different corner radiuses:
  1. 8px (from stripe.com) — rounded but restrained
  2. 6px (from linear.app) — tight, modern
  3. 12px (from vercel.com) — soft, approachable
Which do you prefer? [1/2/3/custom]:
```

## 재사용할 기존 함수

| 함수 | 파일 |
|----------|------|
| `loadTasteProfile()`, `saveTasteProfile()` | `src/store.ts` |
| `makeId()`, `nowIso()` | `src/util.ts` |
| `confirmPrompt()` | `src/interactive.ts` |

## 검증

```bash
npm run build && npm test
```

## 상태
- [ ] 대기 중
