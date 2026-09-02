# Phase 1: 기반 (반드시 먼저 진행)

## 목표
`types.ts`에 모든 공유 타입 추가, `store.ts`에 taste 저장소 추가, `llm.ts`에서 `runCompletion` 래퍼 export, `render.ts`에서 집계 함수 export.

## 수정할 파일

### `src/types.ts` — 모든 taste 프로필 타입 추가 (~90줄)

```typescript
/* ─── Taste Profile types ─── */

export interface TasteColorPreference {
  palette: Array<{ hex: string; role: string; source: string }>;
  harmony: string;              // "monochromatic" | "complementary" | "analogous" | "triadic"
  hueRange: { min: number; max: number };
  saturationBias: 'muted' | 'vibrant' | 'neutral';
  lightnessBias: 'light' | 'dark' | 'balanced';
}

export interface TasteTypographyPreference {
  primaryFont: string;
  secondaryFont: string | null;
  scaleType: string;            // "modular" | "custom" | "tailwind-default"
  sizes: string[];
  weightRange: { min: string; max: string };
}

export interface TasteSpacingPreference {
  baseUnit: number;             // 4 or 8
  scale: number[];              // e.g. [4, 8, 12, 16, 24, 32, 48, 64]
  gridAlignmentRatio: number;
}

export interface TasteMotionPreference {
  easing: string;
  durations: string[];
  intensity: 'none' | 'subtle' | 'moderate' | 'expressive';
}

export interface TasteComponentPreference {
  cherryPicks: ComponentCherryPick[];
  borderRadius: string;
  shadowStyle: 'none' | 'subtle' | 'elevated' | 'dramatic';
}

export interface ComponentCherryPick {
  componentKind: string;
  sourceInspirationId: string;
  sourceUrl: string;
  tokens: ComponentToken[];
  styles: Record<string, string>;
  note?: string;
}

export interface TasteDecision {
  id: string;
  question: string;
  answer: string;
  dimension: 'color' | 'typography' | 'spacing' | 'motion' | 'component' | 'layout' | 'general';
  decidedAt: string;
}

export interface TasteConflict {
  dimension: string;
  description: string;
  options: string[];
  resolved: boolean;
  resolvedByDecisionId?: string;
}

export interface TasteProfile {
  id: string;
  name: string;
  version: number;
  sourceInspirationIds: string[];
  sourceUrls: string[];
  persona: PersonaMatch;
  aggregateScore: ScanScore;
  color: TasteColorPreference;
  typography: TasteTypographyPreference;
  spacing: TasteSpacingPreference;
  motion: TasteMotionPreference;
  components: TasteComponentPreference;
  decisions: TasteDecision[];
  conflicts: TasteConflict[];
  narrative?: string;
  principles?: string[];
  createdAt: string;
  updatedAt: string;
}

export interface TasteDiffResult {
  alignment: number;            // 0-100
  dimensions: {
    color: TasteDimensionDiff;
    typography: TasteDimensionDiff;
    spacing: TasteDimensionDiff;
    motion: TasteDimensionDiff;
  };
  deltas: TasteDelta[];
}

export interface TasteDimensionDiff {
  alignment: number;
  summary: string;
}

export interface TasteDelta {
  dimension: string;
  issue: string;
  suggestion: string;
  severity: 'info' | 'warning' | 'mismatch';
}
```

### `src/store.ts` — taste 프로필 저장소 추가 (+25줄)

```typescript
export function tasteDir(rootDir: string): string {
  return path.join(brainRoot(rootDir), 'taste');
}

export function tasteProfilePath(rootDir: string, projectId: string): string {
  return path.join(tasteDir(rootDir), `${projectId}.json`);
}

export async function loadTasteProfile(rootDir: string, projectId: string): Promise<TasteProfile | null> {
  const filePath = tasteProfilePath(rootDir, projectId);
  if (!await fs.pathExists(filePath)) return null;
  return fs.readJson(filePath);
}

export async function saveTasteProfile(rootDir: string, profile: TasteProfile): Promise<void> {
  const dir = tasteDir(rootDir);
  await fs.ensureDir(dir);
  await fs.writeJson(tasteProfilePath(rootDir, profile.id), profile, { spaces: 2 });
}
```

### `src/llm.ts` — `runDesignLlm` 래퍼 export (+8줄)

```typescript
export async function runDesignLlm(params: {
  llm: LlmConfig;
  system: string;
  userContent: string;
  temperature?: number;
}): Promise<string> {
  return runCompletion(params);
}
```

### `src/render.ts` — 집계 함수 export (+4줄)

각 함수 정의에 `export` 키워드 추가:
- `aggregateColors`
- `aggregateTypography`
- `aggregateComponents`
- `aggregateMotion`

### `src/index.ts` — taste export 추가 (+10줄)

```typescript
export { loadTasteProfile, saveTasteProfile } from './store.js';
export type { TasteProfile, TasteDiffResult, ComponentCherryPick, TasteDecision, TasteConflict } from './types.js';
```

## 검증

```bash
cd packages/design-brain-memory && npm run build && npm test
```

## 상태
- [ ] 대기 중
