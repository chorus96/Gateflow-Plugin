# Phase 2: Taste 빌더 코어

## 목표
`buildTasteProfile()` 생성 — 여러 인스퍼레이션을 하나의 TasteProfile로 집계합니다.

**의존성**: Phase 1 타입 + `render.ts`에서 export된 집계 함수

## 새 파일

### `src/taste.ts` (~250줄)

```typescript
import { aggregateColors, aggregateTypography, aggregateComponents, aggregateMotion } from './render.js';
import { designAnalysisToScanTokens } from './scan.js';
import { assignPersona } from './persona.js';
import { loadDatabase, ensureProject, saveTasteProfile, loadTasteProfile } from './store.js';
import { theatricalScan } from './theatrical.js';
import { looksLikeUrl, normalizeToUrl } from './scan.js';
import type { TasteProfile, InspirationRecord, DesignAnalysis, ... } from './types.js';
```

#### 핵심 함수

**`buildTasteProfile(options)`** — 여러 URL/인스퍼레이션 소스로부터 taste 프로필을 생성하거나 갱신.

Options:
```typescript
{
  rootDir: string;
  projectId: string;
  projectName?: string;
  urls: string[];
  headed?: boolean;
  llm?: LlmConfig;
}
```

반환: `Promise<TasteProfile>`

#### `buildTasteProfile` 알고리즘:
1. 각 URL마다: `theatricalScan(url)`(또는 headless 변형) 호출 → `DesignAnalysis` 획득
2. 각 항목마다 `ingestInspiration()`도 호출 → 프로젝트에 저장 (기존 파이프라인 재사용)
3. 데이터베이스에서 모든 프로젝트 인스퍼레이션 로드
4. 모든 인스퍼레이션에 대해 `aggregateColors()`, `aggregateTypography()`, `aggregateComponents()`, `aggregateMotion()` 호출
5. 각 `derive*Preference()` 함수 실행
6. 집계 데이터로부터 `ScanTokens` 생성 → `computeScore()` → `assignPersona()`
7. 충돌 감지 (Phase 4 인터페이스 참고)
8. LLM 사용 가능 시: 내러티브 + 원칙 생성
9. TasteProfile 저장
10. 반환

#### Headless vs Headed
- 기본: `extractFromUrl.ts`의 `captureDesignFromUrl()` 사용 (headless, 빠름)
- `--headed` 사용 시: `theatrical.ts`의 `theatricalScan()` 사용 (보이는 브라우저)
- 둘 다 `DesignAnalysis`를 생성 — 동일한 다운스트림 파이프라인

#### 비공개 derive 함수

**`deriveColorPreference(colors, inspirations)`** → `TasteColorPreference`
- 하모니 타입, 색상(hue) 범위, 채도/명도 편향을 결정

**`deriveTypographyPreference(typography)`** → `TasteTypographyPreference`
- 빈도로 주/보조 폰트 선택, 스케일 타입 감지

**`deriveSpacingPreference(components)`** → `TasteSpacingPreference`
- 기본 단위(4 또는 8) 감지, 스케일 생성, 그리드 정렬 계산

**`deriveMotionPreference(motion)`** → `TasteMotionPreference`
- 주된 easing/durations 선택, 강도 분류

**`deriveComponentPreference(components)`** → `TasteComponentPreference`
- 컴포넌트 토큰에서 border-radius, 그림자 스타일 도출

---

### `src/tasteRenderer.ts` (~200줄)

taste 프로필을 위한 TUI 렌더링 (`scanRenderer.ts` 같은 ANSI 박스).

#### 함수

**`renderTasteProfile(profile)`** — 전체 taste 카드 출력
**`renderTasteProfileCompact(profile)`** — 한 줄 요약

#### 출력 형식

```
  ┌─ Taste Profile ──────────────────────────────┐
  │                                                │
  │  design-brain  Taste Engine                    │
  │  Project: my-app (3 sources)                   │
  │                                                │
  └────────────────────────────────────────────────┘

  ✔ Analyzed stripe.com, linear.app, vercel.com
  ✔ Persona: Jony Ive — "Less, but better"
  ✔ Aggregate score: 78/100

  ┌─ Color Palette ──────────────────────────────┐
  │  ■ #635BFF  primary (stripe.com)              │
  │  ■ #0A2540  background (stripe.com)           │
  │  ■ #5E6AD2  accent (linear.app)               │
  │  ■ #000000  text (vercel.com)                  │
  │  ■ #FFFFFF  surface (all)                      │
  │                                                │
  │  Harmony: analogous · Vibrant · Dark           │
  └────────────────────────────────────────────────┘

  ┌─ Typography ─────────────────────────────────┐
  │  Primary: Inter                                │
  │  Secondary: GT America                         │
  │  Scale: 14px, 16px, 20px, 24px, 32px, 48px   │
  └────────────────────────────────────────────────┘

  ┌─ Spacing ────────────────────────────────────┐
  │  Base: 8px grid · 94% aligned                  │
  │  Scale: 4 8 12 16 24 32 48 64                  │
  └────────────────────────────────────────────────┘

  ┌─ Motion ─────────────────────────────────────┐
  │  Intensity: subtle                             │
  │  Durations: 150ms, 300ms                       │
  │  Easing: cubic-bezier(0.4, 0, 0.2, 1)         │
  └────────────────────────────────────────────────┘

  ⚠ 2 conflicts detected. Run: taste ask --project my-app
```

## 재사용할 기존 함수 (재구현하지 말 것)

| 함수 | 파일 |
|----------|------|
| `theatricalScan()` | `src/theatrical.ts` |
| `captureDesignFromUrl()` | `src/extractFromUrl.ts` |
| `ingestInspiration()` | `src/commands.ts` |
| `designAnalysisToScanTokens()` | `src/scan.ts` |
| `computeScore()` | `src/scan.ts` |
| `assignPersona()` | `src/persona.ts` |
| `aggregateColors()` | `src/render.ts` |
| `aggregateTypography()` | `src/render.ts` |
| `aggregateComponents()` | `src/render.ts` |
| `aggregateMotion()` | `src/render.ts` |
| `enrichWithLlm()` | `src/llm.ts` |
| `makeId()`, `nowIso()`, `slugify()` | `src/util.ts` |
| `loadDatabase()`, `findProject()` | `src/store.ts` |

## 검증

```bash
npm run build && npm test
# Manual: node dist/cli.js taste stripe.com --project demo
```

## 상태
- [ ] 대기 중
