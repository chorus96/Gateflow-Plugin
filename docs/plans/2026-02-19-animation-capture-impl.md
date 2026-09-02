# 애니메이션 캡처 시스템 구현 계획

> **Claude에게:** 필수 서브 스킬: superpowers:executing-plans를 사용하여 이 계획을 작업 단위로 구현하세요.

**목표:** 얕은 MotionToken(원시 CSS 문자열 4개)을, 애니메이션 라이브러리(GSAP, Lottie, Framer Motion)를 감지하고 Web Animations API로 키프레임을 추출하며 모션 의도를 분류하고 물리 기반 애니메이션을 감지하며 스태거된 시퀀스를 그룹화하는 풍부한 AnimationToken 시스템으로 교체합니다.

**아키텍처:** 새 `ANIMATION_OBSERVER_SCRIPT`가 기존 추출과 함께 Agent Browser `eval`을 통해 페이지에 주입됩니다. 5개의 감지 레이어(라이브러리 감지, WAAPI, GSAP 인트로스펙션, Lottie 추출, 수동 스크롤 감지)를 실행합니다. 결과는 모션 의도 분류, 물리 감지, 애니메이션 그룹화를 처리하는 새 `classify.ts` 모듈에 의해 Node에서 후처리됩니다. 렌더 레이어는 풍부하게 그룹화된 마크다운 섹션을 출력합니다.

**기술 스택:** TypeScript (strict, ESM, NodeNext), Node.js 20+, 테스트용 node:test, 브라우저 자동화용 Agent Browser CLI.

**프로젝트 루트:** `/Users/arnavdas/packages/design-brain-memory`

**빌드:** `npm run build` (tsc)
**테스트:** `npm run build && node --test tests/*.test.mjs`

---

### 작업 1: AnimationToken 타입 추가

**파일:**
- 수정: `src/types.ts`

**1단계: 새 타입 작성**

`MotionToken`(26-31행)을 교체하고 모든 지원 타입을 추가합니다. 하위 호환성을 위해 `MotionToken`을 사용 중단(deprecated) 별칭으로 유지합니다.

`src/types.ts`에서 26-31행을 교체:

```typescript
// OLD:
export interface MotionToken {
  selector: string;
  transition: string;
  animation: string;
  transform: string;
}
```

다음으로 교체:

```typescript
/* ─── Animation capture types ─── */

export type AnimationLibrary = 'css' | 'gsap' | 'lottie' | 'framer-motion' | 'react-spring' | 'motion-one' | 'unknown';
export type MotionIntent = 'fade' | 'slide' | 'scale' | 'rotate' | 'color-shift' | 'spring' | 'bounce' | 'morph' | 'reveal' | 'parallax' | 'complex';
export type TriggerEvent = 'load' | 'hover' | 'focus' | 'click' | 'scroll' | 'viewport-enter' | 'media-query' | 'unknown';

export interface KeyframeStop {
  offset: number;
  properties: Record<string, string>;
  easing?: string;
}

export interface AnimationTiming {
  duration: number;
  delay: number;
  easing: string;
  iterations: number;
  direction: 'normal' | 'reverse' | 'alternate' | 'alternate-reverse';
  fillMode: 'none' | 'forwards' | 'backwards' | 'both';
}

export interface ScrollBinding {
  triggerSelector?: string;
  hasScrollTrigger: boolean;
  hasIntersectionObserver: boolean;
  scrollTimelineAxis?: 'block' | 'inline';
}

export interface PhysicsParams {
  type: 'spring' | 'bounce' | 'inertia';
  mass?: number;
  stiffness?: number;
  damping?: number;
  oscillationCount?: number;
  overshootPercent?: number;
}

export interface AnimationGroup {
  groupId: string;
  role: 'lead' | 'follower';
  staggerDelay?: number;
}

export interface AnimationToken {
  selector: string;
  library: AnimationLibrary;
  motionIntent: MotionIntent;
  timing?: AnimationTiming;
  keyframes?: KeyframeStop[];
  trigger: TriggerEvent;
  scrollBinding?: ScrollBinding;
  physics?: PhysicsParams;
  group?: AnimationGroup;
  gsapVars?: Record<string, unknown>;
  lottieMetadata?: { frameRate: number; totalFrames: number; duration: number };
  rawTransition?: string;
  rawAnimation?: string;
  rawTransform?: string;
}

/** @deprecated Use AnimationToken instead */
export interface MotionToken {
  selector: string;
  transition: string;
  animation: string;
  transform: string;
}
```

**2단계: DesignAnalysis 갱신**

`DesignAnalysis`(78행)의 `motion` 필드를 기존 및 새 토큰 모두 지원하도록 변경:

```typescript
// In DesignAnalysis, change line 78:
motion: (MotionToken | AnimationToken)[];
```

**3단계: 타입이 컴파일되는지 빌드로 확인**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build`

예상: MotionToken 필드를 직접 참조하는 파일에서 타입 오류. 이는 예상된 것으로 — 이후 작업에서 수정합니다.

**4단계: 커밋**

```bash
git add src/types.ts
git commit -m "feat(types): add AnimationToken with library detection, physics, and grouping"
```

---

### 작업 2: 테스트와 함께 classify.ts 생성 (TDD)

**파일:**
- 생성: `src/classify.ts`
- 생성: `tests/classify.test.mjs`

**1단계: 실패하는 테스트 작성**

`tests/classify.test.mjs` 생성:

```javascript
import test from 'node:test';
import assert from 'node:assert/strict';
import {
  classifyMotionIntent,
  detectPhysics,
  detectAnimationGroups,
  classifyGsapEasing,
} from '../dist/classify.js';

/* ─── classifyMotionIntent ─── */

test('classifyMotionIntent: opacity-only → fade', () => {
  const result = classifyMotionIntent([
    { offset: 0, properties: { opacity: '0' } },
    { offset: 1, properties: { opacity: '1' } },
  ]);
  assert.equal(result, 'fade');
});

test('classifyMotionIntent: translateX → slide', () => {
  const result = classifyMotionIntent([
    { offset: 0, properties: { transform: 'translateX(-100px)' } },
    { offset: 1, properties: { transform: 'translateX(0px)' } },
  ]);
  assert.equal(result, 'slide');
});

test('classifyMotionIntent: translateY → slide', () => {
  const result = classifyMotionIntent([
    { offset: 0, properties: { transform: 'translateY(50px)' } },
    { offset: 1, properties: { transform: 'translateY(0px)' } },
  ]);
  assert.equal(result, 'slide');
});

test('classifyMotionIntent: scale → scale', () => {
  const result = classifyMotionIntent([
    { offset: 0, properties: { transform: 'scale(0)' } },
    { offset: 1, properties: { transform: 'scale(1)' } },
  ]);
  assert.equal(result, 'scale');
});

test('classifyMotionIntent: rotate → rotate', () => {
  const result = classifyMotionIntent([
    { offset: 0, properties: { transform: 'rotate(0deg)' } },
    { offset: 1, properties: { transform: 'rotate(360deg)' } },
  ]);
  assert.equal(result, 'rotate');
});

test('classifyMotionIntent: backgroundColor → color-shift', () => {
  const result = classifyMotionIntent([
    { offset: 0, properties: { backgroundColor: 'rgb(255,0,0)' } },
    { offset: 1, properties: { backgroundColor: 'rgb(0,0,255)' } },
  ]);
  assert.equal(result, 'color-shift');
});

test('classifyMotionIntent: clip-path → reveal', () => {
  const result = classifyMotionIntent([
    { offset: 0, properties: { clipPath: 'inset(0 100% 0 0)' } },
    { offset: 1, properties: { clipPath: 'inset(0 0 0 0)' } },
  ]);
  assert.equal(result, 'reveal');
});

test('classifyMotionIntent: opacity + translate → complex', () => {
  const result = classifyMotionIntent([
    { offset: 0, properties: { opacity: '0', transform: 'translateY(20px)' } },
    { offset: 1, properties: { opacity: '1', transform: 'translateY(0px)' } },
  ]);
  assert.equal(result, 'complex');
});

test('classifyMotionIntent: empty keyframes → complex', () => {
  const result = classifyMotionIntent([]);
  assert.equal(result, 'complex');
});

/* ─── detectPhysics ─── */

test('detectPhysics: spring oscillation detected', () => {
  // Values: 0 → 1.15 → 0.95 → 1.02 → 1.0 (overshoot + oscillation)
  const keyframes = [
    { offset: 0, properties: { transform: 'translateX(0px)' } },
    { offset: 0.3, properties: { transform: 'translateX(115px)' } },
    { offset: 0.5, properties: { transform: 'translateX(95px)' } },
    { offset: 0.7, properties: { transform: 'translateX(102px)' } },
    { offset: 1, properties: { transform: 'translateX(100px)' } },
  ];
  const result = detectPhysics(keyframes, 'translateX');
  assert.ok(result);
  assert.equal(result.type, 'spring');
  assert.ok(result.oscillationCount >= 2);
  assert.ok(result.overshootPercent > 0);
});

test('detectPhysics: bounce detected (single overshoot)', () => {
  const keyframes = [
    { offset: 0, properties: { transform: 'translateY(0px)' } },
    { offset: 0.5, properties: { transform: 'translateY(110px)' } },
    { offset: 1, properties: { transform: 'translateY(100px)' } },
  ];
  const result = detectPhysics(keyframes, 'translateY');
  assert.ok(result);
  assert.equal(result.type, 'bounce');
});

test('detectPhysics: linear motion returns null', () => {
  const keyframes = [
    { offset: 0, properties: { transform: 'translateX(0px)' } },
    { offset: 0.5, properties: { transform: 'translateX(50px)' } },
    { offset: 1, properties: { transform: 'translateX(100px)' } },
  ];
  const result = detectPhysics(keyframes, 'translateX');
  assert.equal(result, null);
});

/* ─── classifyGsapEasing ─── */

test('classifyGsapEasing: elastic → spring', () => {
  const result = classifyGsapEasing('elastic.out(1, 0.3)');
  assert.ok(result);
  assert.equal(result.type, 'spring');
});

test('classifyGsapEasing: bounce → bounce', () => {
  const result = classifyGsapEasing('bounce.out');
  assert.ok(result);
  assert.equal(result.type, 'bounce');
});

test('classifyGsapEasing: back → spring', () => {
  const result = classifyGsapEasing('back.out(1.7)');
  assert.ok(result);
  assert.equal(result.type, 'spring');
});

test('classifyGsapEasing: power1 → null (not physics)', () => {
  const result = classifyGsapEasing('power1.out');
  assert.equal(result, null);
});

/* ─── detectAnimationGroups ─── */

test('detectAnimationGroups: staggered elements grouped', () => {
  const tokens = [
    {
      selector: '.item-1', library: 'css', motionIntent: 'fade', trigger: 'load',
      timing: { duration: 300, delay: 0, easing: 'ease-out', iterations: 1, direction: 'normal', fillMode: 'none' },
      keyframes: [{ offset: 0, properties: { opacity: '0' } }, { offset: 1, properties: { opacity: '1' } }],
    },
    {
      selector: '.item-2', library: 'css', motionIntent: 'fade', trigger: 'load',
      timing: { duration: 300, delay: 100, easing: 'ease-out', iterations: 1, direction: 'normal', fillMode: 'none' },
      keyframes: [{ offset: 0, properties: { opacity: '0' } }, { offset: 1, properties: { opacity: '1' } }],
    },
    {
      selector: '.item-3', library: 'css', motionIntent: 'fade', trigger: 'load',
      timing: { duration: 300, delay: 200, easing: 'ease-out', iterations: 1, direction: 'normal', fillMode: 'none' },
      keyframes: [{ offset: 0, properties: { opacity: '0' } }, { offset: 1, properties: { opacity: '1' } }],
    },
  ];

  const result = detectAnimationGroups(tokens);
  assert.equal(result.length, 3);
  assert.equal(result[0].group.role, 'lead');
  assert.equal(result[1].group.role, 'follower');
  assert.equal(result[1].group.staggerDelay, 100);
  assert.equal(result[2].group.role, 'follower');
  assert.equal(result[2].group.staggerDelay, 200);
  // All share same groupId
  assert.equal(result[0].group.groupId, result[1].group.groupId);
});

test('detectAnimationGroups: unrelated animations not grouped', () => {
  const tokens = [
    {
      selector: '.a', library: 'css', motionIntent: 'fade', trigger: 'load',
      timing: { duration: 300, delay: 0, easing: 'ease-out', iterations: 1, direction: 'normal', fillMode: 'none' },
      keyframes: [{ offset: 0, properties: { opacity: '0' } }, { offset: 1, properties: { opacity: '1' } }],
    },
    {
      selector: '.b', library: 'gsap', motionIntent: 'slide', trigger: 'hover',
      timing: { duration: 600, delay: 0, easing: 'ease-in', iterations: 1, direction: 'normal', fillMode: 'none' },
      keyframes: [{ offset: 0, properties: { transform: 'translateX(0)' } }, { offset: 1, properties: { transform: 'translateX(100px)' } }],
    },
  ];

  const result = detectAnimationGroups(tokens);
  assert.equal(result.length, 2);
  assert.equal(result[0].group, undefined);
  assert.equal(result[1].group, undefined);
});
```

**2단계: 테스트가 실패하는지 실행하여 확인**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build && node --test tests/classify.test.mjs`

예상: FAIL — `../dist/classify.js`가 존재하지 않음.

**3단계: classify.ts 구현 작성**

`src/classify.ts` 생성:

```typescript
import type { AnimationToken, KeyframeStop, PhysicsParams, AnimationGroup } from './types.js';

/**
 * Classify the motion intent of an animation based on its keyframe properties.
 */
export function classifyMotionIntent(keyframes: KeyframeStop[]): AnimationToken['motionIntent'] {
  if (keyframes.length === 0) return 'complex';

  const allProps = new Set<string>();
  for (const kf of keyframes) {
    for (const prop of Object.keys(kf.properties)) {
      allProps.add(prop);
    }
  }

  const hasOpacity = allProps.has('opacity');
  const hasTransform = allProps.has('transform');
  const hasColor = allProps.has('backgroundColor') || allProps.has('color') || allProps.has('borderColor');
  const hasClipPath = allProps.has('clipPath');
  const hasWidthHeight = allProps.has('width') || allProps.has('height');

  // Count distinct categories
  const categories: string[] = [];
  if (hasOpacity) categories.push('opacity');
  if (hasTransform) categories.push('transform');
  if (hasColor) categories.push('color');
  if (hasClipPath || hasWidthHeight) categories.push('reveal');

  if (categories.length > 1) return 'complex';
  if (categories.length === 0) return 'complex';

  if (hasOpacity && !hasTransform) return 'fade';
  if (hasColor) return 'color-shift';
  if (hasClipPath || hasWidthHeight) return 'reveal';

  if (hasTransform) {
    return classifyTransformIntent(keyframes);
  }

  return 'complex';
}

function classifyTransformIntent(keyframes: KeyframeStop[]): AnimationToken['motionIntent'] {
  const transforms = keyframes
    .map((kf) => kf.properties.transform ?? '')
    .filter(Boolean);

  const hasTranslate = transforms.some((t) => /translate[XY]?\s*\(/.test(t));
  const hasScale = transforms.some((t) => /scale[XY]?\s*\(/.test(t));
  const hasRotate = transforms.some((t) => /rotate[XYZ]?\s*\(/.test(t));

  const count = [hasTranslate, hasScale, hasRotate].filter(Boolean).length;
  if (count > 1) return 'complex';

  if (hasTranslate) return 'slide';
  if (hasScale) return 'scale';
  if (hasRotate) return 'rotate';

  return 'complex';
}

/**
 * Extract a numeric value from a transform function at a specific keyframe.
 * e.g., "translateX(100px)" → 100
 */
function extractNumericFromTransform(transformStr: string, fnName: string): number | null {
  const re = new RegExp(`${fnName}\\s*\\(\\s*(-?[\\d.]+)`);
  const match = transformStr.match(re);
  return match ? parseFloat(match[1]) : null;
}

/**
 * Detect physics-based motion (spring/bounce) from keyframe value progressions.
 * Looks for overshoot and oscillation around the final value.
 */
export function detectPhysics(
  keyframes: KeyframeStop[],
  trackProperty: string,
): PhysicsParams | null {
  if (keyframes.length < 3) return null;

  const values: number[] = [];
  for (const kf of keyframes) {
    const transformStr = kf.properties.transform ?? kf.properties[trackProperty] ?? '';
    const numericVal = extractNumericFromTransform(transformStr, trackProperty)
      ?? parseFloat(transformStr);
    if (Number.isNaN(numericVal)) return null;
    values.push(numericVal);
  }

  if (values.length < 3) return null;

  const finalValue = values[values.length - 1];
  const startValue = values[0];
  const range = Math.abs(finalValue - startValue);
  if (range === 0) return null;

  // Count zero-crossings relative to final value
  let crossings = 0;
  let maxOvershoot = 0;
  for (let i = 1; i < values.length - 1; i++) {
    const prevDelta = values[i - 1] - finalValue;
    const currDelta = values[i] - finalValue;
    if (prevDelta * currDelta < 0) {
      crossings++;
    }
    const overshoot = Math.abs(currDelta);
    if (overshoot > maxOvershoot) {
      maxOvershoot = overshoot;
    }
  }

  // Check if any intermediate value overshoots the final value
  const overshoots = values.some((v, i) => {
    if (i === 0 || i === values.length - 1) return false;
    if (finalValue > startValue) return v > finalValue;
    return v < finalValue;
  });

  if (!overshoots) return null;

  const overshootPercent = Math.round((maxOvershoot / range) * 100);

  if (crossings >= 2) {
    return {
      type: 'spring',
      oscillationCount: crossings,
      overshootPercent,
    };
  }

  return {
    type: 'bounce',
    oscillationCount: crossings || 1,
    overshootPercent,
  };
}

/**
 * Map GSAP easing string to physics params when applicable.
 */
export function classifyGsapEasing(easeString: string): PhysicsParams | null {
  const lower = easeString.toLowerCase();

  if (lower.includes('elastic')) {
    return { type: 'spring', oscillationCount: 3 };
  }
  if (lower.includes('bounce')) {
    return { type: 'bounce', oscillationCount: 4 };
  }
  if (lower.includes('back')) {
    return { type: 'spring', oscillationCount: 1, overshootPercent: 10 };
  }

  return null;
}

/**
 * Detect animation groups (staggered sequences) by matching tokens
 * with the same motionIntent, library, duration, and easing but different delays.
 */
export function detectAnimationGroups<T extends Pick<AnimationToken, 'selector' | 'motionIntent' | 'library' | 'timing' | 'keyframes' | 'trigger'>>(
  tokens: T[],
): Array<T & { group?: AnimationGroup }> {
  if (tokens.length < 2) {
    return tokens.map((t) => ({ ...t }));
  }

  // Group candidates by signature (same intent, library, duration, easing)
  const buckets = new Map<string, Array<{ index: number; delay: number }>>();

  for (let i = 0; i < tokens.length; i++) {
    const t = tokens[i];
    if (!t.timing) continue;

    const sig = `${t.motionIntent}|${t.library}|${t.timing.duration}|${t.timing.easing}|${keyframeSignature(t.keyframes)}`;
    const bucket = buckets.get(sig) ?? [];
    bucket.push({ index: i, delay: t.timing.delay });
    buckets.set(sig, bucket);
  }

  const result: Array<T & { group?: AnimationGroup }> = tokens.map((t) => ({ ...t }));

  let groupCounter = 0;
  for (const [, bucket] of buckets) {
    if (bucket.length < 2) continue;

    // Sort by delay
    bucket.sort((a, b) => a.delay - b.delay);

    // Check if delays form a stagger pattern (roughly equal increments)
    const deltas: number[] = [];
    for (let i = 1; i < bucket.length; i++) {
      deltas.push(bucket[i].delay - bucket[i - 1].delay);
    }

    const avgDelta = deltas.reduce((sum, d) => sum + d, 0) / deltas.length;
    const isStagger = avgDelta > 0 && deltas.every((d) => Math.abs(d - avgDelta) <= avgDelta * 0.5);

    if (!isStagger) continue;

    groupCounter++;
    const groupId = `group-${groupCounter}`;
    const leadDelay = bucket[0].delay;

    for (let i = 0; i < bucket.length; i++) {
      const entry = bucket[i];
      result[entry.index] = {
        ...result[entry.index],
        group: {
          groupId,
          role: i === 0 ? 'lead' : 'follower',
          staggerDelay: i === 0 ? 0 : entry.delay - leadDelay,
        },
      };
    }
  }

  return result;
}

function keyframeSignature(keyframes?: KeyframeStop[]): string {
  if (!keyframes || keyframes.length === 0) return '';
  return keyframes.map((kf) => Object.keys(kf.properties).sort().join(',')).join('|');
}
```

**4단계: 빌드 후 테스트 실행**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build && node --test tests/classify.test.mjs`

예상: 모든 테스트 PASS.

**5단계: 커밋**

```bash
git add src/classify.ts tests/classify.test.mjs
git commit -m "feat: add motion classification, physics detection, animation grouping"
```

---

### 작업 3: ANIMATION_OBSERVER_SCRIPT 작성

**파일:**
- 수정: `src/extractFromUrl.ts`

이것은 핵심 브라우저 주입 스크립트입니다. Agent Browser `eval`을 통해 페이지 내부에서 실행되며 Node 측 classify.ts가 처리할 원시 애니메이션 데이터를 반환합니다.

**1단계: EXTRACTION_SCRIPT 뒤에 스크립트 상수 추가**

`src/extractFromUrl.ts`에서 `EXTRACTION_SCRIPT`(324행 뒤)에 다음을 추가:

```typescript
const ANIMATION_OBSERVER_SCRIPT = String.raw`(() => {
  const maxAnimations = 200;
  const result = {
    libraries: [],
    webAnimations: [],
    gsapTweens: [],
    lottiePlayers: [],
    scrollBindings: {
      hasScrollTrigger: false,
      hasIntersectionObserver: false,
      scrollTriggers: [],
      hasScrollSnap: false,
      hasScrollTimeline: false,
    },
  };

  /* ─── Layer 1: Library Detection ─── */

  if (typeof gsap !== 'undefined') {
    result.libraries.push({
      name: 'gsap',
      version: typeof gsap.version === 'string' ? gsap.version : 'unknown',
    });
  }

  if (typeof ScrollTrigger !== 'undefined') {
    result.libraries.push({ name: 'scrolltrigger', version: 'detected' });
    result.scrollBindings.hasScrollTrigger = true;
  }

  if (typeof lottie !== 'undefined' || typeof bodymovin !== 'undefined') {
    result.libraries.push({ name: 'lottie', version: 'detected' });
  }

  if (document.querySelector('lottie-player, dotlottie-player')) {
    if (!result.libraries.some(l => l.name === 'lottie')) {
      result.libraries.push({ name: 'lottie', version: 'web-component' });
    }
  }

  // Framer Motion heuristic: data-framer-* attributes
  if (document.querySelector('[data-framer-name], [data-framer-component-type]')) {
    result.libraries.push({ name: 'framer-motion', version: 'detected' });
  }

  /* ─── Layer 2: Web Animations API ─── */

  try {
    const animations = document.getAnimations();
    for (const anim of animations.slice(0, maxAnimations)) {
      const effect = anim.effect;
      if (!effect || !effect.target) continue;

      const target = effect.target;
      const selector = buildSelector(target);

      let keyframes = [];
      try { keyframes = effect.getKeyframes(); } catch {}

      let computedTiming = {};
      try { computedTiming = effect.getComputedTiming(); } catch {}

      let timelineType = 'document';
      try {
        if (anim.timeline && anim.timeline.constructor) {
          const name = anim.timeline.constructor.name;
          if (name === 'ScrollTimeline') timelineType = 'scroll';
          else if (name === 'ViewTimeline') timelineType = 'view';
        }
      } catch {}

      if (timelineType !== 'document') {
        result.scrollBindings.hasScrollTimeline = true;
      }

      result.webAnimations.push({
        selector,
        playState: anim.playState,
        animationName: anim.animationName || anim.id || '',
        keyframes: keyframes.map(kf => {
          const props = {};
          for (const [k, v] of Object.entries(kf)) {
            if (k !== 'offset' && k !== 'computedOffset' && k !== 'easing' && k !== 'composite') {
              props[k] = String(v);
            }
          }
          return {
            offset: kf.computedOffset ?? kf.offset ?? 0,
            properties: props,
            easing: kf.easing || 'linear',
          };
        }),
        timing: {
          duration: typeof computedTiming.duration === 'number' ? computedTiming.duration : 0,
          delay: computedTiming.delay || 0,
          easing: computedTiming.easing || 'linear',
          iterations: computedTiming.iterations ?? 1,
          direction: computedTiming.direction || 'normal',
          fillMode: computedTiming.fill || 'none',
        },
        timelineType,
      });
    }
  } catch {}

  /* ─── Layer 3: GSAP Introspection ─── */

  try {
    if (typeof gsap !== 'undefined' && gsap.globalTimeline) {
      const children = gsap.globalTimeline.getChildren(true, true, false);
      for (const tween of children.slice(0, maxAnimations)) {
        const targets = tween.targets ? tween.targets() : [];
        if (targets.length === 0) continue;

        const selector = buildSelector(targets[0]);
        const vars = {};
        if (tween.vars) {
          for (const [k, v] of Object.entries(tween.vars)) {
            if (typeof v === 'number' || typeof v === 'string') {
              vars[k] = v;
            }
          }
        }

        result.gsapTweens.push({
          selector,
          duration: typeof tween.duration === 'function' ? tween.duration() : 0,
          delay: typeof tween.delay === 'function' ? tween.delay() : 0,
          ease: tween.vars?.ease || 'power1.out',
          vars,
          startTime: typeof tween.startTime === 'function' ? tween.startTime() : 0,
        });
      }
    }
  } catch {}

  /* ─── Layer 4: Lottie Extraction ─── */

  try {
    const lottieRef = typeof lottie !== 'undefined' ? lottie : (typeof bodymovin !== 'undefined' ? bodymovin : null);
    if (lottieRef && typeof lottieRef.getRegisteredAnimations === 'function') {
      const registered = lottieRef.getRegisteredAnimations();
      for (const anim of registered.slice(0, 20)) {
        const container = anim.wrapper || anim.container;
        result.lottiePlayers.push({
          selector: container ? buildSelector(container) : 'lottie-unknown',
          frameRate: anim.frameRate || anim.animationData?.fr || 0,
          totalFrames: anim.totalFrames || anim.animationData?.op || 0,
          duration: typeof anim.getDuration === 'function' ? anim.getDuration(false) : 0,
        });
      }
    }

    // Web component detection
    const players = document.querySelectorAll('lottie-player, dotlottie-player');
    for (const player of Array.from(players).slice(0, 20)) {
      const selector = buildSelector(player);
      if (result.lottiePlayers.some(l => l.selector === selector)) continue;
      result.lottiePlayers.push({
        selector,
        frameRate: 0,
        totalFrames: 0,
        duration: 0,
      });
    }
  } catch {}

  /* ─── Layer 5: Passive Scroll Detection ─── */

  try {
    if (typeof ScrollTrigger !== 'undefined' && typeof ScrollTrigger.getAll === 'function') {
      const triggers = ScrollTrigger.getAll();
      for (const st of triggers.slice(0, 50)) {
        result.scrollBindings.scrollTriggers.push({
          triggerSelector: st.trigger ? buildSelector(st.trigger) : '',
          isActive: !!st.isActive,
        });
      }
    }
  } catch {}

  try {
    const scrollSnap = window.getComputedStyle(document.documentElement).scrollSnapType;
    if (scrollSnap && scrollSnap !== 'none') {
      result.scrollBindings.hasScrollSnap = true;
    }
  } catch {}

  /* ─── Helper ─── */

  function buildSelector(el) {
    if (!el || !el.tagName) return 'unknown';
    const tag = el.tagName.toLowerCase();
    if (el.id) return tag + '#' + el.id;
    if (typeof el.className === 'string' && el.className.trim()) {
      const firstClass = el.className.trim().split(/\s+/).slice(0, 2).join('.');
      if (firstClass) return tag + '.' + firstClass;
    }
    return tag;
  }

  return result;
})();`;
```

**2단계: 스크립트 문자열이 컴파일되는지 빌드로 확인**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build`

예상: 컴파일됨 (스크립트는 단지 문자열 상수임).

**3단계: 커밋**

```bash
git add src/extractFromUrl.ts
git commit -m "feat: add ANIMATION_OBSERVER_SCRIPT with 5-layer detection"
```

---

### 작업 4: 애니메이션 옵저버를 추출 파이프라인에 연결

**파일:**
- 수정: `src/extractFromUrl.ts`

**1단계: classify.ts와 AnimationToken에 대한 import 추가**

`src/extractFromUrl.ts` 상단에서 import 갱신:

```typescript
import type {
  AnimationToken,
  DesignAnalysis,
  MotionToken,
  ResponsiveSnapshot,
  StateStyleToken,
  JourneyStep,
} from './types.js';
import { classifyMotionIntent, detectPhysics, detectAnimationGroups, classifyGsapEasing } from './classify.js';
```

**2단계: raw→AnimationToken 변환 함수 추가**

`ANIMATION_OBSERVER_SCRIPT` 뒤에 추가:

```typescript
interface RawAnimObserverResult {
  libraries: Array<{ name: string; version: string }>;
  webAnimations: Array<{
    selector: string;
    playState: string;
    animationName: string;
    keyframes: Array<{ offset: number; properties: Record<string, string>; easing?: string }>;
    timing: {
      duration: number;
      delay: number;
      easing: string;
      iterations: number;
      direction: string;
      fillMode: string;
    };
    timelineType: string;
  }>;
  gsapTweens: Array<{
    selector: string;
    duration: number;
    delay: number;
    ease: string;
    vars: Record<string, unknown>;
    startTime: number;
  }>;
  lottiePlayers: Array<{
    selector: string;
    frameRate: number;
    totalFrames: number;
    duration: number;
  }>;
  scrollBindings: {
    hasScrollTrigger: boolean;
    hasIntersectionObserver: boolean;
    scrollTriggers: Array<{ triggerSelector: string; isActive: boolean }>;
    hasScrollSnap: boolean;
    hasScrollTimeline: boolean;
  };
}

function convertObserverResult(raw: RawAnimObserverResult): AnimationToken[] {
  const tokens: AnimationToken[] = [];
  const libraryNames = new Set(raw.libraries.map((l) => l.name));

  // Convert Web Animations API results
  for (const wa of raw.webAnimations) {
    const intent = classifyMotionIntent(wa.keyframes);
    const isCssAnim = wa.animationName && wa.animationName !== '';
    let library: AnimationToken['library'] = 'css';
    if (libraryNames.has('framer-motion')) {
      // Heuristic: Framer Motion uses WAAPI internally
      library = 'framer-motion';
    }

    const token: AnimationToken = {
      selector: wa.selector,
      library,
      motionIntent: intent,
      timing: {
        duration: wa.timing.duration,
        delay: wa.timing.delay,
        easing: wa.timing.easing,
        iterations: wa.timing.iterations === Infinity ? Infinity : wa.timing.iterations,
        direction: wa.timing.direction as AnimationToken['timing'] extends undefined ? never : NonNullable<AnimationToken['timing']>['direction'],
        fillMode: wa.timing.fillMode as AnimationToken['timing'] extends undefined ? never : NonNullable<AnimationToken['timing']>['fillMode'],
      },
      keyframes: wa.keyframes,
      trigger: wa.timelineType === 'scroll' || wa.timelineType === 'view' ? 'scroll' : 'load',
    };

    if (wa.timelineType !== 'document') {
      token.scrollBinding = {
        hasScrollTrigger: false,
        hasIntersectionObserver: false,
        scrollTimelineAxis: 'block',
      };
    }

    // Detect physics from keyframes
    const allProps = new Set<string>();
    for (const kf of wa.keyframes) {
      for (const prop of Object.keys(kf.properties)) allProps.add(prop);
    }

    if (allProps.has('transform')) {
      for (const fnName of ['translateX', 'translateY', 'scale', 'rotate']) {
        const physics = detectPhysics(wa.keyframes, fnName);
        if (physics) {
          token.physics = physics;
          token.motionIntent = physics.type === 'spring' ? 'spring' : 'bounce';
          break;
        }
      }
    }

    tokens.push(token);
  }

  // Convert GSAP tweens
  for (const tween of raw.gsapTweens) {
    const token: AnimationToken = {
      selector: tween.selector,
      library: 'gsap',
      motionIntent: 'complex',
      timing: {
        duration: tween.duration * 1000,
        delay: tween.delay * 1000,
        easing: tween.ease,
        iterations: 1,
        direction: 'normal',
        fillMode: 'none',
      },
      trigger: 'load',
      gsapVars: tween.vars,
    };

    const physics = classifyGsapEasing(tween.ease);
    if (physics) {
      token.physics = physics;
      token.motionIntent = physics.type === 'spring' ? 'spring' : 'bounce';
    } else {
      // Infer intent from vars
      const varKeys = Object.keys(tween.vars);
      if (varKeys.some((k) => ['x', 'y', 'xPercent', 'yPercent'].includes(k))) {
        token.motionIntent = 'slide';
      } else if (varKeys.some((k) => ['scale', 'scaleX', 'scaleY'].includes(k))) {
        token.motionIntent = 'scale';
      } else if (varKeys.some((k) => ['rotation', 'rotationX', 'rotationY'].includes(k))) {
        token.motionIntent = 'rotate';
      } else if (varKeys.includes('opacity')) {
        token.motionIntent = 'fade';
      }
    }

    // Check if this tween is scroll-bound
    if (raw.scrollBindings.hasScrollTrigger) {
      const matchingTrigger = raw.scrollBindings.scrollTriggers.find(
        (st) => st.triggerSelector === tween.selector
      );
      if (matchingTrigger) {
        token.trigger = 'scroll';
        token.scrollBinding = {
          triggerSelector: matchingTrigger.triggerSelector,
          hasScrollTrigger: true,
          hasIntersectionObserver: false,
        };
      }
    }

    tokens.push(token);
  }

  // Convert Lottie players
  for (const lp of raw.lottiePlayers) {
    tokens.push({
      selector: lp.selector,
      library: 'lottie',
      motionIntent: 'complex',
      trigger: 'load',
      lottieMetadata: {
        frameRate: lp.frameRate,
        totalFrames: lp.totalFrames,
        duration: lp.duration,
      },
    });
  }

  return tokens;
}
```

**3단계: captureDesignFromUrl에 통합**

`captureDesignFromUrl`에서 뷰포트 루프 뒤, 대화형 상태 캡처 전(653행 부근)에 애니메이션 옵저버 호출을 추가:

```typescript
    // Run animation observer script (once, at desktop viewport)
    let animationTokens: AnimationToken[] = [];
    try {
      const animResult = await runAgentBrowserJson(['eval', ANIMATION_OBSERVER_SCRIPT], {
        session: sessionName,
        cwd: workingDir,
      });
      if (animResult.success && animResult.data.result) {
        const rawAnimData = animResult.data.result as RawAnimObserverResult;
        animationTokens = convertObserverResult(rawAnimData);
        animationTokens = detectAnimationGroups(animationTokens);
      }
    } catch {
      // Animation capture is best-effort; don't fail the whole extraction
    }
```

그다음 return 블록(673행 부근)에서 애니메이션 토큰을 레거시 motion 배열과 병합:

```typescript
    // Merge: prefer new AnimationTokens, fall back to legacy MotionTokens
    const mergedMotion: (MotionToken | AnimationToken)[] = [
      ...animationTokens,
      ...merged.motion.filter((legacyToken) =>
        !animationTokens.some((at) => at.selector === legacyToken.selector)
      ),
    ];

    return {
      ...merged,
      motion: mergedMotion,
      accessibilitySnapshot: (snapshotResult.data.snapshot as string | undefined) ?? undefined,
      responsiveSnapshots,
      stateStyles,
      journey,
    };
```

**4단계: 빌드**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build`

예상: 컴파일됨. `motion`을 참조하는 다른 파일에서 일부 타입 오류가 발생할 수 있음 — 작업 6에서 처리.

**5단계: 커밋**

```bash
git add src/extractFromUrl.ts
git commit -m "feat: integrate animation observer into extraction pipeline"
```

---

### 작업 5: 풍부한 모션 섹션을 위해 render.ts 갱신

**파일:**
- 수정: `src/render.ts`

**1단계: AnimationToken import와 타입 가드 추가**

상단의 import 갱신:

```typescript
import type {
  AnimationToken,
  ColorToken,
  ComponentToken,
  DesignBrainDatabase,
  InspirationRecord,
  MotionToken,
  ProjectRecord,
  TypographyToken,
} from './types.js';
```

타입 가드 함수 추가:

```typescript
function isAnimationToken(token: MotionToken | AnimationToken): token is AnimationToken {
  return 'library' in token && 'motionIntent' in token;
}
```

**2단계: aggregateMotion을 새 그룹화 렌더링으로 교체**

`aggregateMotion` 함수(71-84행)를 다음으로 교체:

```typescript
function aggregateMotion(records: InspirationRecord[]): Array<(MotionToken | AnimationToken) & { count: number }> {
  const map = new Map<string, (MotionToken | AnimationToken) & { count: number }>();

  for (const record of records) {
    for (const token of record.analysis.motion) {
      const key = isAnimationToken(token)
        ? `${token.selector}|${token.library}|${token.motionIntent}`
        : `${token.selector}|${token.transition}|${token.animation}|${token.transform}`;
      const existing = map.get(key) ?? { ...token, count: 0 };
      existing.count += 1;
      map.set(key, existing);
    }
  }

  return [...map.values()].sort((a, b) => b.count - a.count).slice(0, 140);
}
```

**3단계: renderRichMotionSection 헬퍼 추가**

```typescript
function renderRichMotionSection(tokens: Array<(MotionToken | AnimationToken) & { count: number }>): string {
  const animTokens = tokens.filter(isAnimationToken);
  const legacyTokens = tokens.filter((t) => !isAnimationToken(t)) as Array<MotionToken & { count: number }>;

  if (animTokens.length === 0 && legacyTokens.length > 0) {
    // All legacy — fall back to old format
    const rows = legacyTokens.map((t) => [
      truncate(t.transition || 'none'),
      truncate(t.animation || 'none'),
      `\`${t.selector}\``,
      `${t.count}`,
    ]);
    return mdTable(['Transition', 'Animation', 'Selector', 'Occurrences'], rows);
  }

  const sections: string[] = [];

  // Libraries
  const libraries = new Set<string>();
  for (const t of animTokens) {
    if (t.library !== 'css' && t.library !== 'unknown') {
      libraries.add(t.library);
    }
  }
  if (libraries.size > 0) {
    sections.push('### Libraries Detected\n' + [...libraries].map((l) => `- **${l}**`).join('\n'));
  }

  // Group by motionIntent
  const groups = new Map<string, Array<AnimationToken & { count: number }>>();
  for (const t of animTokens) {
    const intent = t.motionIntent;
    const group = groups.get(intent) ?? [];
    group.push(t as AnimationToken & { count: number });
    groups.set(intent, group);
  }

  const intentOrder = ['fade', 'slide', 'scale', 'rotate', 'spring', 'bounce', 'color-shift', 'reveal', 'morph', 'parallax', 'complex'];

  for (const intent of intentOrder) {
    const group = groups.get(intent);
    if (!group || group.length === 0) continue;

    const label = intent.charAt(0).toUpperCase() + intent.slice(1);

    if (intent === 'spring' || intent === 'bounce') {
      const rows = group.map((t) => [
        `\`${t.selector}\``,
        t.timing ? `${t.timing.duration}ms` : '-',
        t.physics?.overshootPercent != null ? `${t.physics.overshootPercent}%` : '-',
        t.physics?.oscillationCount != null ? `${t.physics.oscillationCount}` : '-',
        t.library,
      ]);
      sections.push(`### ${label} Animations (${group.length})\n\n` + mdTable(['Element', 'Duration', 'Overshoot', 'Oscillations', 'Library'], rows));
    } else {
      const rows = group.map((t) => [
        `\`${t.selector}\``,
        t.timing ? `${t.timing.duration}ms` : '-',
        t.timing?.easing ?? '-',
        t.trigger,
        t.library,
      ]);
      sections.push(`### ${label} Animations (${group.length})\n\n` + mdTable(['Element', 'Duration', 'Easing', 'Trigger', 'Library'], rows));
    }
  }

  // Lottie
  const lotties = animTokens.filter((t) => t.lottieMetadata);
  if (lotties.length > 0) {
    const rows = lotties.map((t) => [
      `\`${t.selector}\``,
      t.lottieMetadata ? `${t.lottieMetadata.frameRate}fps` : '-',
      t.lottieMetadata ? `${t.lottieMetadata.duration.toFixed(1)}s` : '-',
      t.lottieMetadata ? `${t.lottieMetadata.totalFrames}` : '-',
    ]);
    sections.push('### Lottie Animations\n\n' + mdTable(['Container', 'Frame Rate', 'Duration', 'Frames'], rows));
  }

  // Scroll-bound
  const scrollBound = animTokens.filter((t) => t.trigger === 'scroll' || t.scrollBinding);
  if (scrollBound.length > 0) {
    const rows = scrollBound.map((t) => [
      `\`${t.selector}\``,
      t.scrollBinding?.hasScrollTrigger ? 'ScrollTrigger' : 'CSS ScrollTimeline',
      t.library,
    ]);
    sections.push('### Scroll-Bound (passive)\n\n' + mdTable(['Element', 'Type', 'Library'], rows));
  }

  // Animation sequences
  const groupedTokens = animTokens.filter((t) => t.group);
  const sequenceMap = new Map<string, Array<AnimationToken & { count: number }>>();
  for (const t of groupedTokens) {
    const gid = t.group!.groupId;
    const seq = sequenceMap.get(gid) ?? [];
    seq.push(t as AnimationToken & { count: number });
    sequenceMap.set(gid, seq);
  }
  if (sequenceMap.size > 0) {
    const seqLines: string[] = [];
    for (const [gid, seq] of sequenceMap) {
      const sorted = seq.sort((a, b) => (a.group?.staggerDelay ?? 0) - (b.group?.staggerDelay ?? 0));
      const parts = sorted.map((t) => {
        const delay = t.group?.staggerDelay ?? 0;
        return t.group?.role === 'lead'
          ? `\`${t.selector}\` (lead)`
          : `\`${t.selector}\` (+${delay}ms)`;
      });
      seqLines.push(`- **${gid}**: ${parts.join(' → ')}`);
    }
    sections.push('### Animation Sequences\n\n' + seqLines.join('\n'));
  }

  // Legacy tokens at the end
  if (legacyTokens.length > 0) {
    const rows = legacyTokens.map((t) => [
      truncate(t.transition || 'none'),
      truncate(t.animation || 'none'),
      `\`${t.selector}\``,
      `${t.count}`,
    ]);
    sections.push('### CSS Transitions (legacy)\n\n' + mdTable(['Transition', 'Animation', 'Selector', 'Occurrences'], rows));
  }

  return sections.join('\n\n');
}
```

**4단계: 새 함수를 사용하도록 renderTokens 갱신**

`renderTokens`(261행 부근)에서 모션 렌더링을 교체:

```typescript
// OLD:
const motionRows = motion.map((token) => [truncate(token.transition || 'none'), truncate(token.animation || 'none'), `\`${token.selector}\``, `${token.count}`]);

// ... and later:
motion:
  `# ${project.name} Motion Brain\n\n` + mdTable(['Transition', 'Animation', 'Selector', 'Occurrences'], motionRows) + '\n',
```

다음으로 교체:

```typescript
motion:
  `# ${project.name} Motion Brain\n\n` + renderRichMotionSection(motion) + '\n',
```

(`motionRows` 변수를 제거.)

**5단계: AnimationToken을 처리하도록 renderInspiration 갱신**

`renderInspiration`(135-140행 부근)에서 교체:

```typescript
  const motionRows = record.analysis.motion.slice(0, 80).map((motion) => [
    `\`${motion.selector}\``,
    truncate(motion.transition || 'none'),
    truncate(motion.animation || 'none'),
    truncate(motion.transform || 'none'),
  ]);
```

다음으로 교체:

```typescript
  const motionContent = renderRichMotionSection(
    record.analysis.motion.slice(0, 80).map((t) => ({ ...t, count: 1 }))
  );
```

그리고 Motion 테이블 출력(202행 부근)을 교체:

```typescript
// OLD:
+ `## Motion\n\n${mdTable(['Selector', 'Transition', 'Animation', 'Transform'], motionRows)}\n\n`

// NEW:
+ `## Motion\n\n${motionContent}\n\n`
```

**6단계: 빌드**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build`

예상: 컴파일됨.

**7단계: 커밋**

```bash
git add src/render.ts
git commit -m "feat: rich grouped markdown rendering for animation tokens"
```

---

### 작업 6: 지원 파일 갱신 (scan, llm, query, commands)

**파일:**
- 수정: `src/scan.ts`
- 수정: `src/llm.ts`
- 수정: `src/query.ts`
- 수정: `src/commands.ts`

**1단계: scan.ts 갱신 — @keyframes 정규식과 애니메이션 라이브러리 감지 추가**

`src/scan.ts`에서 60행 뒤에 새 정규식 패턴을 추가:

```typescript
const KEYFRAME_RE = /@keyframes\s+([\w-]+)/g;
const GSAP_IMPORT_RE = /(?:from\s+['"]gsap|require\s*\(\s*['"]gsap)/g;
const FRAMER_IMPORT_RE = /(?:from\s+['"]framer-motion|from\s+['"]motion)/g;
const LOTTIE_IMPORT_RE = /(?:from\s+['"]lottie-web|from\s+['"]@lottiefiles)/g;
```

키프레임 이름을 추적하도록 `scanCssContent`를 갱신 (return 전에 함수에 추가):

```typescript
  // Keyframe definitions
  const keyframeNames: string[] = [];
  for (const m of css.matchAll(KEYFRAME_RE)) {
    keyframeNames.push(m[1].trim());
  }
  // Include keyframe names in transitions array for scoring
  for (const name of keyframeNames) {
    transitions.push(`@keyframes:${name}`);
  }
```

두 토큰 타입을 모두 처리하도록 `designAnalysisToScanTokens`(314-327행)를 갱신:

```typescript
export function designAnalysisToScanTokens(analysis: DesignAnalysis): ScanTokens {
  const transitionValues: string[] = [];
  for (const m of analysis.motion) {
    if ('library' in m && 'motionIntent' in m) {
      // AnimationToken
      const at = m as AnimationToken;
      if (at.timing) {
        transitionValues.push(`${at.motionIntent} ${at.timing.duration}ms ${at.timing.easing}`);
      }
    } else {
      // Legacy MotionToken
      const mt = m as MotionToken;
      if (mt.transition && mt.transition !== 'all 0s ease 0s') {
        transitionValues.push(mt.transition);
      }
    }
  }

  return {
    colors: analysis.colors.map(c => c.hex),
    fontFamilies: [...new Set(analysis.typography.map(t => t.fontFamily.toLowerCase()))],
    fontSizes: [...new Set(analysis.typography.map(t => t.fontSize))],
    transitions: transitionValues,
    spacingValues: analysis.components
      .flatMap(c => [c.styles.padding, c.styles.margin].filter(Boolean) as string[]),
    cssVariableCount: Object.keys(analysis.cssVariables).length,
    framework: null,
  };
}
```

scan.ts 상단에 `AnimationToken`과 `MotionToken` import를 추가.

**2단계: llm.ts 갱신 — 프롬프트에 더 풍부한 모션 컨텍스트**

`src/llm.ts`에서 `buildPrompt` 함수의 motion 줄(175행)을 갱신:

```typescript
// OLD:
const motionList = input.analysis.motion.slice(0, 12).map((motion) => `${motion.selector}:${motion.transition || motion.animation || motion.transform}`).join(', ');

// NEW:
const motionList = input.analysis.motion.slice(0, 12).map((m) => {
  if ('library' in m && 'motionIntent' in m) {
    const at = m as AnimationToken;
    const dur = at.timing ? `${at.timing.duration}ms` : '';
    return `${at.selector}:${at.motionIntent}(${at.library}${dur ? ' ' + dur : ''})`;
  }
  const mt = m as MotionToken;
  return `${mt.selector}:${mt.transition || mt.animation || mt.transform}`;
}).join(', ');
```

`AnimationToken` import를 추가.

또한 `enrichImageWithLlmVision`(327행)을 갱신 — 비전에서의 모션 구성은 MotionToken(레거시)을 반환해야 하며, 이미지가 런타임 애니메이션을 감지할 수 없으므로 문제없음.

**3단계: query.ts 갱신 — 새 토큰 필드 검색**

`src/query.ts`에서 인스퍼레이션 텍스트 빌더(102행)를 갱신:

```typescript
// OLD:
inspiration.analysis.motion.map((motion) => `${motion.transition} ${motion.animation}`).join(' '),

// NEW:
inspiration.analysis.motion.map((m) => {
  if ('library' in m && 'motionIntent' in m) {
    const at = m as { library: string; motionIntent: string; selector: string; trigger: string };
    return `${at.motionIntent} ${at.library} ${at.selector} ${at.trigger}`;
  }
  const mt = m as { transition: string; animation: string };
  return `${mt.transition} ${mt.animation}`;
}).join(' '),
```

**4단계: commands.ts 갱신 — fingerprint가 두 타입을 모두 처리**

`src/commands.ts`에서 `computeFingerprint` 함수(34행)를 갱신:

```typescript
// OLD:
motion: input.analysis.motion.slice(0, 20).map((motion) => `${motion.transition}|${motion.animation}`),

// NEW:
motion: input.analysis.motion.slice(0, 20).map((m) => {
  if ('library' in m && 'motionIntent' in m) {
    const at = m as { library: string; motionIntent: string; selector: string };
    return `${at.library}|${at.motionIntent}|${at.selector}`;
  }
  const mt = m as { transition: string; animation: string };
  return `${mt.transition}|${mt.animation}`;
}),
```

**5단계: 빌드 후 모든 테스트 실행**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build && node --test tests/*.test.mjs`

예상: 기존의 모든 테스트 통과. `designAnalysisToScanTokens` 테스트는 MotionToken 형식을 사용하고 코드가 둘 다 처리하므로 여전히 통과해야 함.

**6단계: 커밋**

```bash
git add src/scan.ts src/llm.ts src/query.ts src/commands.ts
git commit -m "feat: update scan, llm, query, commands for AnimationToken support"
```

---

### 작업 7: 하위 호환성을 위해 기존 scan 테스트 갱신

**파일:**
- 수정: `tests/scan.test.mjs`

**1단계: 기존 designAnalysisToScanTokens 테스트가 여전히 동작하는지 확인**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build && node --test tests/scan.test.mjs`

예상: PASS — 테스트는 여전히 지원되는 레거시 MotionToken 형식을 사용함.

**2단계: AnimationToken 변환에 대한 테스트 추가**

`tests/scan.test.mjs`에 추가:

```javascript
test('designAnalysisToScanTokens handles AnimationToken format', () => {
  const mockAnalysis = {
    colors: [{ hex: '#FF0000', count: 5, samples: [] }],
    typography: [{ fontFamily: 'Inter', fontSize: '16px', fontWeight: '400', lineHeight: '1.5', count: 10 }],
    components: [],
    motion: [
      {
        selector: '.hero',
        library: 'css',
        motionIntent: 'fade',
        trigger: 'load',
        timing: { duration: 600, delay: 0, easing: 'ease-out', iterations: 1, direction: 'normal', fillMode: 'none' },
        keyframes: [],
      },
      {
        selector: '.card',
        library: 'gsap',
        motionIntent: 'slide',
        trigger: 'scroll',
        timing: { duration: 400, delay: 100, easing: 'power2.out', iterations: 1, direction: 'normal', fillMode: 'none' },
      },
    ],
    layout: [],
    cssVariables: {},
  };

  const tokens = designAnalysisToScanTokens(mockAnalysis);
  assert.equal(tokens.transitions.length, 2);
  assert.ok(tokens.transitions[0].includes('fade'));
  assert.ok(tokens.transitions[0].includes('600ms'));
  assert.ok(tokens.transitions[1].includes('slide'));
});
```

**3단계: 테스트 실행**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build && node --test tests/scan.test.mjs`

예상: 모두 PASS.

**4단계: 커밋**

```bash
git add tests/scan.test.mjs
git commit -m "test: add AnimationToken backward-compatibility test for scan"
```

---

### 작업 8: 최종 빌드, 전체 테스트 스위트, 검증

**파일:** 새 파일 없음

**1단계: 전체 빌드**

실행: `cd /Users/arnavdas/packages/design-brain-memory && npm run build`

예상: 오류 없음.

**2단계: 전체 테스트 스위트 실행**

실행: `cd /Users/arnavdas/packages/design-brain-memory && node --test tests/*.test.mjs`

예상: 모든 테스트 통과 (util, query, persona, scan, classify).

**3단계: export된 타입 확인**

실행: `cd /Users/arnavdas/packages/design-brain-memory && grep -r "AnimationToken" dist/types.d.ts`

예상: `AnimationToken`이 선언 파일에 export됨.

**4단계: 커밋**

수정이 필요했다면 커밋:

```bash
git add -A
git commit -m "fix: resolve any remaining type/build issues from animation capture"
```
