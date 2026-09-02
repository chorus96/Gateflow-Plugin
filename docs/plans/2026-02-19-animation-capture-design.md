# design-brain-memory를 위한 애니메이션 캡처 시스템

**날짜:** 2026-02-19
**상태:** 승인됨
**범위:** 수동 스크롤 감지를 갖춘 전체 런타임 관찰

## 문제

현재 design-brain-memory의 모션 캡처는 원시 CSS 프로퍼티 문자열(`transition: all 0.3s ease-in-out`)을 의미론적 이해 없이 저장합니다. GSAP, Lottie, Framer Motion, 스크롤 트리거 효과, 물리 기반 모션, 애니메이션 시퀀싱, 키프레임 데이터를 감지할 수 없습니다.

## 결정 기록

- **범위**: 전체 런타임 관찰 (라이브러리 감지, Web Animations API, rAF 샘플링, 수동 스크롤 감지)
- **스크롤 처리**: 수동 감지만 (ScrollTrigger/IntersectionObserver 사용 감지, 스크롤 시뮬레이션은 하지 않음)
- **출력 형식**: 모션 의도별로 그룹화된 풍부한 마크다운 섹션
- **추가 기능**: 물리 파라미터 감지, 애니메이션 그룹화/스태거 감지

## 데이터 모델

### 핵심 타입

```typescript
type AnimationLibrary = 'css' | 'gsap' | 'lottie' | 'framer-motion' | 'react-spring' | 'motion-one' | 'unknown';
type MotionIntent = 'fade' | 'slide' | 'scale' | 'rotate' | 'color-shift' | 'spring' | 'bounce' | 'morph' | 'reveal' | 'parallax' | 'complex';
type TriggerEvent = 'load' | 'hover' | 'focus' | 'click' | 'scroll' | 'viewport-enter' | 'media-query' | 'unknown';

interface KeyframeStop {
  offset: number;
  properties: Record<string, string>;
  easing?: string;
}

interface AnimationTiming {
  duration: number;
  delay: number;
  easing: string;
  iterations: number;
  direction: 'normal' | 'reverse' | 'alternate' | 'alternate-reverse';
  fillMode: 'none' | 'forwards' | 'backwards' | 'both';
}

interface ScrollBinding {
  triggerSelector?: string;
  hasScrollTrigger: boolean;
  hasIntersectionObserver: boolean;
  scrollTimelineAxis?: 'block' | 'inline';
}

interface PhysicsParams {
  type: 'spring' | 'bounce' | 'inertia';
  mass?: number;
  stiffness?: number;
  damping?: number;
  oscillationCount?: number;
  overshootPercent?: number;
}

interface AnimationGroup {
  groupId: string;
  role: 'lead' | 'follower';
  staggerDelay?: number;
}

interface AnimationToken {
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
```

### 하위 호환성

기존 `MotionToken` 필드는 `rawTransition`, `rawAnimation`, `rawTransform`로 매핑됩니다. 렌더 레이어는 새 필드를 확인하고 기존 형식으로 폴백합니다. 기존 데이터베이스 항목은 유효하게 유지됩니다.

## 추출 아키텍처

Agent Browser `eval`을 통해 주입되는 5계층 옵저버 스크립트:

### 레이어 1: 라이브러리 감지
- `window.gsap`(+ 버전), `window.lottie`/`window.bodymovin`, `<lottie-player>` 요소 확인
- `typeof ScrollTrigger !== 'undefined'`를 통한 ScrollTrigger 감지
- Framer Motion: `data-framer-*` 속성이 있는 요소의 WAAPI 애니메이션을 통한 휴리스틱
- React Spring / Motion One: WAAPI 또는 MutationObserver 스타일 패턴을 통해 감지

### 레이어 2: Web Animations API
- `document.getAnimations()`가 CSS 애니메이션, 트랜지션, WAAPI 애니메이션을 캡처
- 전체 키프레임 데이터를 위해 `effect.getKeyframes()` 추출
- 구조화된 타이밍을 위해 `effect.getComputedTiming()` 추출

### 레이어 3: GSAP 인트로스펙션 (조건부)
- `gsap.globalTimeline.getChildren(true, true, true)`가 모든 트윈을 열거
- 추출: `targets()`, `duration()`, `vars`(properties, easing), `startTime()`
- ScrollTrigger: 스크롤 바인딩 애니메이션 메타데이터를 위한 `ScrollTrigger.getAll()`

### 레이어 4: Lottie 추출 (조건부)
- `lottie.getRegisteredAnimations()` 또는 `<lottie-player>` 요소
- 추출: `animationData` 요약 (frameRate, totalFrames, duration, 레이어 수)
- Lottie 스키마(v, fr, ip, op, layers)와 일치하는 `.json` 파일의 네트워크 요청도 감지

### 레이어 5: 수동 스크롤 감지
- `ScrollTrigger` 인스턴스와 그 구성을 감지
- 생성자가 호출되었는지 확인하여 `IntersectionObserver` 사용 감지
- `document.getAnimations()` 타임라인 타입을 통해 CSS `ScrollTimeline` / `ViewTimeline` 감지
- `scroll-snap` CSS 프로퍼티 감지

## 모션 의도 분류

키프레임에서 애니메이션되는 프로퍼티에 기반한 휴리스틱:

| 애니메이션 프로퍼티 | 의도 |
|---|---|
| `opacity`만 | `fade` |
| `translateX/Y` | `slide` |
| `scale` / `scaleX/Y` | `scale` |
| `rotate` / `rotateX/Y/Z` | `rotate` |
| `backgroundColor`, `color`, `borderColor` | `color-shift` |
| `clip-path`, 0에서 애니메이션되는 `height`/`width` | `reveal` |
| 값의 오버슈트 + 진동 | `spring` 또는 `bounce` |
| 여러 프로퍼티 타입 | `complex` |

## 물리 감지

키프레임 값 시퀀스에서 오버슈트/진동을 분석:
1. 애니메이션 값이 최종 값을 초과했다가 되돌아오는지 식별 (오버슈트)
2. 진동 교차 횟수 계산
3. 진폭이 감소하는 진동이면: `spring`
4. 단일 오버슈트 + 급격한 복귀면: `bounce`
5. 저장: 진동 횟수, 오버슈트 백분율
6. 알려진 ease 문자열(예: `elastic`, `bounce`, `back`)이 있는 GSAP 트윈은 직접 매핑

## 애니메이션 그룹화

두 가지 감지 휴리스틱:
1. **스태거**: 동일한 애니메이션 이름/키프레임을 공유하며 증분 지연을 갖는 요소들
2. **시간적 클러스터링**: DOM 형제 또는 부모를 공유하는 요소에서 50ms 창 안에 시작하는 애니메이션들

그룹 구조: 리드 요소(가장 이른 시작) + 계산된 스태거 지연을 갖는 팔로워들.

## 마크다운 렌더링

라이브러리 표기와 함께 모션 의도별로 그룹화:

```markdown
## Motion System

### Libraries Detected
- **GSAP 3.12.5** with ScrollTrigger
- **Lottie** (2 animations)

### Fade Animations (4)
| Element | Duration | Easing | Trigger | Library |
|---------|----------|--------|---------|---------|
| `.hero-title` | 600ms | ease-out | load | css |
| `.card` (x3) | 400ms | ease-out | load, stagger 100ms | css |

### Spring Animations (2)
| Element | Duration | Overshoot | Oscillations | Library |
|---------|----------|-----------|--------------|---------|
| `.modal` | 500ms | 12% | 2 | framer-motion |

### Scroll-Bound (passive)
| Element | Type | Library |
|---------|------|---------|
| `.parallax-bg` | ScrollTrigger | gsap |

### Lottie Animations
| Container | Frame Rate | Duration | Frames |
|-----------|-----------|----------|--------|
| `.spinner` | 30fps | 2.0s | 60 |

### Animation Sequences
- **hero-enter**: `.hero-title` (lead) -> `.hero-subtitle` (+200ms) -> `.hero-cta` (+400ms)
```

## 변경된 파일

| 파일 | 변경 |
|------|--------|
| `src/types.ts` | `MotionToken`을 `AnimationToken`으로 교체, 지원 타입 추가 |
| `src/extractFromUrl.ts` | 새 `ANIMATION_OBSERVER_SCRIPT`, 캡처 파이프라인에 통합 |
| `src/render.ts` | 그룹화된 마크다운을 갖춘 새 `renderMotionSection()` |
| `src/scan.ts` | `@keyframes`에 대한 향상된 정규식, 라이브러리 import 감지 |
| `src/llm.ts` | 새 토큰 구조를 위한 보강 프롬프트 갱신 |
| `src/query.ts` | 새 토큰 형식을 위한 검색 필드 갱신 |
| `src/store.ts` | 기존 MotionToken 데이터의 하위 호환 처리 |

## 구현 순서

1. 타입 (`types.ts`) - 새 인터페이스
2. 추출 스크립트 (`extractFromUrl.ts`) - 옵저버 레이어
3. 분류 로직 - 의도, 물리, 그룹화 (extractFromUrl 또는 새 `classify.ts`에 둘 수 있음)
4. 렌더링 (`render.ts`) - 새 마크다운 출력
5. 지원 갱신 (`scan.ts`, `llm.ts`, `query.ts`, `store.ts`)
6. 테스트 - 알려진 애니메이션이 있는 실제 사이트로 검증
