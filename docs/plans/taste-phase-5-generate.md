# Phase 5: LLM 생성 + 스킬 + CLI 연결

## 목표
taste 프로필로부터 디자인 토큰/컴포넌트를 생성하고, 모든 taste 커맨드로 CLI를 갱신하며, 스킬을 갱신합니다.

**의존성**: Phase 1 타입 + `llm.ts`의 `runDesignLlm`

**CLI 연결**은 Phase 2-4에서 import하므로, 2-4가 완료된 후에 마무리해야 합니다. `tasteGenerate.ts` 파일 자체는 Phase 2-4 의존성이 없습니다.

## 새 파일: `src/tasteGenerate.ts` (~200줄)

### 핵심 함수

**`generateDesignTokens(taste)`** → `string`
- taste 프로필로부터 CSS 커스텀 프로퍼티를 생성 (LLM 불필요, 결정적)

**`generateFromTaste(options)`** → `{ code, explanation }`
- LLM + taste 프로필을 컨텍스트로 사용해 컴포넌트나 페이지를 생성
- Options: `{ taste, target, componentKind?, llm, framework? }`
- Target: `'component' | 'page' | 'tokens'`

### 토큰 생성 출력 (결정적, LLM 없음)

```css
:root {
  /* Colors — from taste profile */
  --color-primary: #635BFF;
  --color-background: #0A2540;
  --color-accent: #5E6AD2;
  --color-text: #000000;
  --color-surface: #FFFFFF;

  /* Typography */
  --font-primary: 'Inter', sans-serif;
  --font-secondary: 'GT America', sans-serif;
  --text-sm: 14px;
  --text-base: 16px;
  --text-lg: 20px;
  --text-xl: 24px;
  --text-2xl: 32px;
  --text-3xl: 48px;

  /* Spacing */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;
  --space-16: 64px;

  /* Motion */
  --duration-fast: 150ms;
  --duration-normal: 300ms;
  --easing-default: cubic-bezier(0.4, 0, 0.2, 1);

  /* Shape */
  --radius-default: 8px;
}
```

### LLM 생성 프롬프트 템플릿

```
You are a frontend developer. Generate production-ready {framework} code matching this taste profile.

Persona: {persona.name} — "{persona.tagline}"
Palette: {palette list}
Typography: Primary={primaryFont}, Secondary={secondaryFont}
Spacing: {baseUnit}px base, scale=[{scale}]
Motion: {intensity}, {durations}, {easing}
Shape: radius={borderRadius}, shadow={shadowStyle}
Principles: {principles list}

Cherry-picked references:
{for each cherry-pick: kind from source with key styles}

Generate: {target} {componentKind?}
Requirements: Use exact token values. Semantic HTML. Accessible.
Output: Single code block, then a 2-sentence explanation.
```

---

## CLI 연결 (`src/cli.ts` 수정)

서브커맨드가 있는 `taste` 커맨드 그룹 추가:

### `taste build <urls...>`
- 하나 이상의 URL로부터 taste 프로필을 생성
- Options: `--project`, `--name`, `--headed`, `--llm-base-url`, `--llm-api-key`, `--llm-model`
- `buildTasteProfile()` 호출

### `taste show`
- 현재 프로필을 표시
- Options: `--project`, `--json`
- taste 프로필을 로드 + 렌더링

### `taste pick`
- 컴포넌트를 체리픽
- 필수: `--component`, `--from`
- Options: `--project`, `--note`
- `cherryPickComponent()` 호출

### `taste score [path]`
- taste 대비 코드베이스를 점수화
- Options: `--project`
- `scoreTaste()` 호출

### `taste ask`
- 명확화 질문에 답변
- Options: `--project`, `--answer`
- `nextClarifyingQuestion()` + `applyDecision()` 호출

### `taste generate`
- taste로부터 코드를 생성
- Options: `--project`, `--target`, `--component`, `--framework`, `--llm-*`
- `generateFromTaste()` 호출

### 기본 커맨드 라우팅
아무 인자 없는 `npx design-brain-memory stripe.com linear.app`는 `taste build`로 라우팅되어야 함.

---

## 스킬 갱신 (`skills/design-brain/SKILL.md`)

섹션 추가:

```markdown
## Taste Profile Context

Before generating UI code, check for a taste profile:
\`\`\`bash
cat .design-brain/taste/*.json 2>/dev/null
\`\`\`

If a taste profile exists, ALL generated UI code MUST use:
- Colors from the palette (exact hex values)
- The specified font families
- Spacing values from the scale
- Motion parameters (durations, easing)
- Cherry-picked component styles where applicable

Commands:
- `design-brain-memory taste show --project <id> --json` — get taste as JSON
- `design-brain-memory taste score . --project <id>` — check alignment
- `design-brain-memory taste generate --target tokens --project <id>` — emit CSS vars
```

## 검증

```bash
npm run build && npm test
# Full flow:
node dist/cli.js taste build stripe.com linear.app --project demo
node dist/cli.js taste show --project demo
node dist/cli.js taste score . --project demo
node dist/cli.js taste generate --target tokens --project demo
```

## 상태
- [ ] 대기 중
