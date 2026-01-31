## About This Repository

이 레포지토리는 AI가 작성해주는 코드를 조금이라도 더 이해하기 위한 학습을 목적으로 [react-best-practices](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices)의 rules를 한국어로 번역했습니다.

## Rule File Structure

각 rules는 다음 구조를 따릅니다:

```markdown
---
title: Rule Title Here
impact: MEDIUM
impactDescription: Optional description
tags: tag1, tag2, tag3
---

## Rule Title Here

Brief explanation of the rule and why it matters.

**Incorrect (description of what's wrong):**

// Bad code example

**Correct (description of what's right):**

// Good code example
```

## Impact Levels

### `CRITICAL`
  - [`bundle-barrel-imports.md`](rules/bundle-barrel-imports.md) (Avoid Barrel File Imports)
  - [`server-auth-actions.md`](rules/server-auth-actions.md) (Authenticate Server Actions Like API Routes)
  - [`bundle-dynamic-imports.md`](rules/bundle-dynamic-imports.md) (Dynamic Imports for Heavy Components)
  - [`async-parallel.md`](rules/async-parallel.md) (Promise.all() for Independent Operations)
  - [`server-parallel-fetching.md`](rules/server-parallel-fetching.md) (Parallel Data Fetching with Component Composition)
  - [`async-dependencies.md`](rules/async-dependencies.md) (Dependency-Based Parallelization)
  - [`async-api-routes.md`](rules/async-api-routes.md) (Prevent Waterfall Chains in API Routes)
### `HIGH`
  - [`server-cache-lru.md`](rules/server-cache-lru.md) (Cross-Request LRU Caching)
  - [`rendering-content-visibility.md`](rules/rendering-content-visibility.md) (CSS content-visibility for Long Lists)
  - [`server-serialization.md`](rules/server-serialization.md) (Minimize Serialization at RSC Boundaries)
  - [`async-defer-await.md`](rules/async-defer-await.md) (Defer Await Until Needed)
  - [`async-suspense-boundaries.md`](rules/async-suspense-boundaries.md) (Strategic Suspense Boundaries)
  - [`bundle-conditional.md`](rules/bundle-conditional.md) (Conditional Module Loading)
### `MEDIUM-HIGH`
  - [`js-length-check-first.md`](rules/js-length-check-first.md) (Early Length Check for Array Comparisons)
  - [`js-tosorted-immutable.md`](rules/js-tosorted-immutable.md) (Use toSorted() Instead of sort() for Immutability)
  - [`client-swr-dedup.md`](rules/client-swr-dedup.md) (Use SWR for Automatic Deduplication)
### `MEDIUM`
  - [`bundle-preload.md`](rules/bundle-preload.md) (Preload Based on User Intent)
  - [`rerender-memo-with-default-value.md`](rules/rerender-memo-with-default-value.md) (Extract Default Non-primitive Parameter Value from Memoized Component to Constant)
  - [`rerender-derived-state-no-effect.md`](rules/rerender-derived-state-no-effect.md) (Calculate Derived State During Rendering)
  - [`rerender-derived-state.md`](rules/rerender-derived-state.md) (Subscribe to Derived State)
  - [`rendering-activity.md`](rules/rendering-activity.md) (Use Activity Component for Show/Hide)
  - [`rerender-move-effect-to-event.md`](rules/rerender-move-effect-to-event.md) (Put Interaction Logic in Event Handlers)
  - [`rerender-defer-reads.md`](rules/rerender-defer-reads.md) (Defer State Reads to Usage Point)
  - [`bundle-defer-third-party.md`](rules/bundle-defer-third-party.md) (Defer Non-Critical Third-Party Libraries)
  - [`rerender-use-ref-transient-values.md`](rules/rerender-use-ref-transient-values.md) (Use useRef for Transient Values)
  - [`client-passive-event-listeners.md`](rules/client-passive-event-listeners.md) (Use Passive Event Listeners for Scrolling Performance)
  - [`rerender-functional-setstate.md`](rules/rerender-functional-setstate.md) (Use Functional setState Updates)
  - [`rerender-lazy-state-init.md`](rules/rerender-lazy-state-init.md) (Use Lazy State Initialization)
  - [`server-after-nonblocking.md`](rules/server-after-nonblocking.md) (Use after() for Non-Blocking Operations)
  - [`rerender-transitions.md`](rules/rerender-transitions.md) (Use Transitions for Non-Urgent Updates)
  - [`server-cache-react.md`](rules/server-cache-react.md) (Per-Request Deduplication with React.cache())
  - [`rendering-hydration-no-flicker.md`](rules/rendering-hydration-no-flicker.md) (Prevent Hydration Mismatch Without Flickering)
  - [`rerender-memo.md`](rules/rerender-memo.md) (Extract to Memoized Components)
  - [`js-batch-dom-css.md`](rules/js-batch-dom-css.md) (Avoid Layout Thrashing)
  - [`js-cache-function-results.md`](rules/js-cache-function-results.md) (Cache Repeated Function Calls)
  - [`client-localstorage-schema.md`](rules/client-localstorage-schema.md) (Version and Minimize localStorage Data)
### `LOW-MEDIUM`
  - [`js-cache-storage.md`](rules/js-cache-storage.md) (Cache Storage API Calls)
  - [`js-combine-iterations.md`](rules/js-combine-iterations.md) (Combine Multiple Array Iterations)
  - [`rendering-hydration-suppress-warning.md`](rules/rendering-hydration-suppress-warning.md) (Suppress Expected Hydration Mismatches)
  - [`js-set-map-lookups.md`](rules/js-set-map-lookups.md) (Use Set/Map for O(1) Lookups)
  - [`js-cache-property-access.md`](rules/js-cache-property-access.md) (Cache Property Access in Loops)
  - [`js-early-exit.md`](rules/js-early-exit.md) (Early Return from Functions)
  - [`js-index-maps.md`](rules/js-index-maps.md) (Build Index Maps for Repeated Lookups)
  - [`advanced-init-once.md`](rules/advanced-init-once.md) (Initialize App Once, Not Per Mount)
  - [`js-hoist-regexp.md`](rules/js-hoist-regexp.md) (Hoist RegExp Creation)
  - [`rerender-simple-expression-in-memo.md`](rules/rerender-simple-expression-in-memo.md) (Do not wrap a simple expression with a primitive result type in useMemo)
### `LOW`
  - [`advanced-use-latest.md`](rules/advanced-use-latest.md) (useEffectEvent for Stable Callback Refs)
  - [`server-dedup-props.md`](rules/server-dedup-props.md) (Avoid Duplicate Serialization in RSC Props)
  - [`rerender-dependencies.md`](rules/rerender-dependencies.md) (Narrow Effect Dependencies)
  - [`rendering-hoist-jsx.md`](rules/rendering-hoist-jsx.md) (Hoist Static JSX Elements)
  - [`js-min-max-loop.md`](rules/js-min-max-loop.md) (Use Loop for Min/Max Instead of Sort)
  - [`rendering-animate-svg-wrapper.md`](rules/rendering-animate-svg-wrapper.md) (Animate SVG Wrapper Instead of SVG Element)
  - [`rendering-usetransition-loading.md`](rules/rendering-usetransition-loading.md) (Use useTransition Over Manual Loading States)
  - [`advanced-event-handler-refs.md`](rules/advanced-event-handler-refs.md) (Store Event Handlers in Refs)
  - [`client-event-listeners.md`](rules/client-event-listeners.md) (Deduplicate Global Event Listeners)
  - [`rendering-svg-precision.md`](rules/rendering-svg-precision.md) (Optimize SVG Precision)
  - [`rendering-conditional-render.md`](rules/rendering-conditional-render.md) (Use Explicit Conditional Rendering)

## Acknowledgments

Originally created by [@shuding](https://x.com/shuding) at [Vercel](https://vercel.com).
