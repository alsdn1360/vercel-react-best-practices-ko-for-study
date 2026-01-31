# React Best Practices

**Version 1.0.0**  
Vercel Engineering  
January 2026

> **Note:**  
> 이 문서는 주로 agent와 LLM이 React 및 Next.js codebase를 유지 관리, 생성 또는 refactoring할 때 따르기 위한 것입니다.  
> 사람에게도 유용할 수 있지만, 여기에 기재된 지침은 AI-assisted workflow를 통한 자동화와 consistency에 최적화되어 있습니다.  
> 하지만 이 문서는 저의 학습을 위한 것입니다.

---

## Abstract

AI agent와 LLM을 위해 설계된 React 및 Next.js application용 종합 performance optimization guide입니다. Critical(waterfall 제거, bundle size 감소)부터 incremental(advanced pattern)까지 영향력에 따라 우선순위가 지정된 8개 category, 40개 이상의 rule을 포함하고 있습니다. 각 rule에는 상세한 설명, incorrect와 correct 구현을 비교하는 실제 예시, 그리고 자동화된 refactoring 및 code generation을 안내하기 위한 구체적인 impact metric이 포함되어 있습니다.

---

## Table of Contents

1. [Waterfall 제거](#1-waterfall-제거) — **CRITICAL**
   - 1.1 [필요한 시점까지 await 지연](#11-필요한-시점까지-await-지연)
   - 1.2 [의존성 기반 병렬화 (Dependency-Based Parallelization)](#12-의존성-기반-병렬화-dependency-based-parallelization)
   - 1.3 [API Route에서 Waterfall Chain 방지](#13-api-route에서-waterfall-chain-방지)
   - 1.4 [독립적인 작업에 Promise.all() 사용](#14-독립적인-작업에-promiseall-사용)
   - 1.5 [전략적 Suspense Boundary 사용](#15-전략적-suspense-boundary-사용)
2. [Bundle Size Optimization](#2-bundle-size-optimization) — **CRITICAL**
   - 2.1 [Barrel File Import 피하기](#21-barrel-file-import-피하기)
   - 2.2 [조건부 Module Loading](#22-조건부-module-loading)
   - 2.3 [비임계(Non-Critical) 제3자 Library 지연](#23-비임계non-critical-제3자-library-지연)
   - 2.4 [무거운 Component에 Dynamic Import 사용](#24-무거운-component에-dynamic-import-사용)
   - 2.5 [사용자 의도 기반 Preload](#25-사용자-의도-기반-preload)
3. [Server-Side Performance](#3-server-side-performance) — **HIGH**
   - 3.1 [Server Action을 API Route처럼 인증](#31-server-action을-api-route처럼-인증)
   - 3.2 [RSC Prop에서 중복 Serialization 피하기](#32-rsc-prop에서-중복-serialization-피하기)
   - 3.3 [Cross-Request LRU Caching](#33-cross-request-lru-caching)
   - 3.4 [RSC Boundary에서 Serialization 최소화](#34-rsc-boundary에서-serialization-최소화)
   - 3.5 [Component Composition을 통한 병렬 Data Fetching](#35-component-composition을-통한-병렬-data-fetching)
   - 3.6 [React.cache()를 이용한 Request별 Deduplication](#36-reactcache를-이용한-request별-deduplication)
   - 3.7 [Non-Blocking 작업을 위한 after() 사용](#37-non-blocking-작업을-위한-after-사용)
4. [Client-Side Data Fetching](#4-client-side-data-fetching) — **MEDIUM-HIGH**
   - 4.1 [Global Event Listener Deduplication](#41-global-event-listener-deduplication)
   - 4.2 [Scrolling 성능을 위해 Passive Event Listener 사용](#42-scrolling-성능을-위해-passive-event-listener-사용)
   - 4.3 [자동 Deduplication을 위해 SWR 사용](#43-자동-deduplication을-위해-swr-사용)
   - 4.4 [localStorage Data 버전 관리 및 최소화](#44-localstorage-data-버전-관리-및-최소화)
5. [Re-render Optimization](#5-re-render-optimization) — **MEDIUM**
   - 5.1 [Rendering 중에 파생 State 계산](#51-rendering-중에-파생-state-계산)
   - 5.2 [State 읽기를 사용 시점으로 지연](#52-state-읽기를-사용-시점으로-지연)
   - 5.3 [Primitive 결과 타입을 갖는 단순 표현식에 useMemo 사용 금지](#53-primitive-결과-타입을-갖는-단순-표현식에-usememo-사용-금지)
   - 5.4 [Memoized Component의 Non-primitive Parameter Default Value를 상수로 추출](#54-memoized-component의-non-primitive-parameter-default-value를-상수로-추출)
   - 5.5 [Memoized Component로 추출](#55-memoized-component로-추출)
   - 5.6 [Effect Dependency 좁히기](#56-effect-dependency-좁히기)
   - 5.7 [Interaction Logic을 Event Handler에 배치](#57-interaction-logic을-event-handler에-배치)
   - 5.8 [파생 State에 Subscribe](#58-파생-state에-subscribe)
   - 5.9 [Functional setState Update 사용](#59-functional-setstate-update-사용)
   - 5.10 [Lazy State Initialization 사용](#510-lazy-state-initialization-사용)
   - 5.11 [비긴급(Non-Urgent) 업데이트에 Transition 사용](#511-비긴급non-urgent-업데이트에-transition-사용)
   - 5.12 [일시적인 값(Transient Value)에 useRef 사용](#512-일시적인-값transient-value에-useref-사용)
6. [Rendering Performance](#6-rendering-performance) — **MEDIUM**
   - 6.1 [SVG Element 대신 SVG Wrapper 애니메이션화](#61-svg-element-대신-svg-wrapper-애니메이션화)
   - 6.2 [긴 List를 위한 CSS content-visibility](#62-긴-list를-위한-css-content-visibility)
   - 6.3 [Static JSX Element 호이스팅](#63-static-jsx-element-호이스팅)
   - 6.4 [SVG Precision Optimize](#64-svg-precision-optimize)
   - 6.5 [Flicker 없는 Hydration Mismatch 방지](#65-flicker-없는-hydration-mismatch-방지)
   - 6.6 [예상된 Hydration Mismatch 억제](#66-예상된-hydration-mismatch-억제)
   - 6.7 [Show/Hide에 Activity Component 사용](#67-showhide에-activity-component-사용)
   - 6.8 [명시적 Conditional Rendering 사용](#68-명시적-conditional-rendering-사용)
   - 6.9 [수동 Loading State보다 useTransition 사용](#69-수동-loading-state보다-usetransition-사용)
7. [JavaScript Performance](#7-javascript-performance) — **LOW-MEDIUM**
   - 7.1 [Layout Thrashing 피하기](#71-layout-thrashing-피하기)
   - 7.2 [반복된 Lookup을 위해 Index Map 구축](#72-반복된-lookup을-위해-index-map-구축)
   - 7.3 [Loop 내 Property Access 캐싱](#73-loop-내-property-access-캐싱)
   - 7.4 [반복되는 Function Call 캐싱](#74-반복되는-function-call-캐싱)
   - 7.5 [Storage API Call 캐싱](#75-storage-api-call-캐싱)
   - 7.6 [여러 번의 Array Iteration 결합](#76-여러-번의-array-iteration-결합)
   - 7.7 [Array 비교 시 Early Length Check](#77-array-비교-시-early-length-check)
   - 7.8 [Function에서 Early Return](#78-function에서-early-return)
   - 7.9 [RegExp 생성 호이스팅](#79-regexp-생성-호이스팅)
   - 7.10 [Min/Max를 위해 Sort 대신 Loop 사용](#710-minmax를-위해-sort-대신-loop-사용)
   - 7.11 [O(1) Lookup을 위해 Set/Map 사용](#711-o1-lookup을-위해-setmap-사용)
   - 7.12 [Immutability를 위해 sort() 대신 toSorted() 사용](#712-immutability를-위해-sort-대신-tosorted-사용)
8. [Advanced Patterns](#8-advanced-patterns) — **LOW**
   - 8.1 [Mount 시가 아닌, App당 한 번 초기화](#81-mount-시가-아닌-app당-한-번-초기화)
   - 8.2 [Event Handler를 Ref에 저장](#82-event-handler를-ref에-저장)
   - 8.3 [Stable한 Callback Ref를 위한 useEffectEvent](#83-stable한-callback-ref를-위한-useeffectevent)

---

## 1. Waterfall 제거

**Impact: CRITICAL**

Waterfall은 성능의 최대 적입니다. 순차적인 await는 각각 전체 network latency를 추가합니다. 이를 제거하는 것이 가장 큰 성능 향상을 가져옵니다.

### 1.1 필요한 시점까지 await 지연

**Impact: HIGH (사용되지 않는 code path 블로킹 방지)**

`await` 작업을 실제로 사용되는 branch 안으로 이동시켜, 해당 작업이 필요 없는 code path가 블로킹되는 것을 방지하세요.

**Incorrect: 두 branch를 모두 블로킹함**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId)
  
  if (skipProcessing) {
    // 즉시 return하지만 여전히 userData를 기다렸음
    return { skipped: true }
  }
  
  // 이 branch만 userData를 사용함
  return processUserData(userData)
}

```

**Correct: 필요할 때만 블로킹함**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // 기다리지 않고 즉시 return
    return { skipped: true }
  }
  
  // 필요할 때만 fetch
  const userData = await fetchUserData(userId)
  return processUserData(userData)
}

```

**또 다른 예시: early return optimization**

```typescript
// Incorrect: 항상 permissions를 fetch함
async function updateResource(resourceId: string, userId: string) {
  const permissions = await fetchPermissions(userId)
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}

// Correct: 필요할 때만 fetch함
async function updateResource(resourceId: string, userId: string) {
  const resource = await getResource(resourceId)
  
  if (!resource) {
    return { error: 'Not found' }
  }
  
  const permissions = await fetchPermissions(userId)
  
  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }
  
  return await updateResourceData(resource, permissions)
}

```

이 optimization은 skip되는 branch가 자주 실행되거나, 지연된 작업이 expensive할 때 특히 유용합니다.

### 1.2 의존성 기반 병렬화 (Dependency-Based Parallelization)

**Impact: CRITICAL (2-10배 향상)**

부분적인 의존성이 있는 작업의 경우, `better-all`을 사용하여 병렬성을 극대화하세요. 각 task를 가능한 가장 빠른 시점에 자동으로 시작합니다.

**Incorrect: profile이 불필요하게 config를 기다림**

```typescript
const [user, config] = await Promise.all([
  fetchUser(),
  fetchConfig()
])
const profile = await fetchProfile(user.id)

```

**Correct: config와 profile이 병렬로 실행됨**

```typescript
import { all } from 'better-all'

const { user, config, profile } = await all({
  async user() { return fetchUser() },
  async config() { return fetchConfig() },
  async profile() {
    return fetchProfile((await this.$.user).id)
  }
})

```

**추가 의존성 없는 대안:**

```typescript
const userPromise = fetchUser()
const profilePromise = userPromise.then(user => fetchProfile(user.id))

const [user, config, profile] = await Promise.all([
  userPromise,
  fetchConfig(),
  profilePromise
])

```

모든 promise를 먼저 생성한 뒤, 마지막에 `Promise.all()`을 수행할 수도 있습니다.

Reference: [https://github.com/shuding/better-all](https://github.com/shuding/better-all)

### 1.3 API Route에서 Waterfall Chain 방지

**Impact: CRITICAL (2-10배 향상)**

API route와 Server Action에서는 독립적인 작업을 즉시 시작하세요. 아직 await하지 않더라도 마찬가지입니다.

**Incorrect: config는 auth를 기다리고, data는 둘 다 기다림**

```typescript
export async function GET(request: Request) {
  const session = await auth()
  const config = await fetchConfig()
  const data = await fetchData(session.user.id)
  return Response.json({ data, config })
}

```

**Correct: auth와 config가 즉시 시작됨**

```typescript
export async function GET(request: Request) {
  const sessionPromise = auth()
  const configPromise = fetchConfig()
  const session = await sessionPromise
  const [config, data] = await Promise.all([
    configPromise,
    fetchData(session.user.id)
  ])
  return Response.json({ data, config })
}

```

복잡한 의존성 체인이 있는 작업은 `better-all`을 사용하여 병렬성을 자동으로 극대화하세요 (Dependency-Based Parallelization 참고).

### 1.4 독립적인 작업에 Promise.all() 사용

**Impact: CRITICAL (2-10배 향상)**

async 작업 간에 상호 의존성이 없을 경우, `Promise.all()`을 사용하여 동시에 실행하세요.

**Incorrect: 순차 실행, 3번의 round trip**

```typescript
const user = await fetchUser()
const posts = await fetchPosts()
const comments = await fetchComments()

```

**Correct: 병렬 실행, 1번의 round trip**

```typescript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])

```

### 1.5 전략적 Suspense Boundary 사용

**Impact: HIGH (더 빠른 initial paint)**

async component에서 JSX를 return하기 전에 data를 await하는 대신, Suspense boundary를 사용하여 data가 로드되는 동안 wrapper UI를 더 빨리 보여주세요.

**Incorrect: data fetching에 의해 wrapper가 블로킹됨**

```tsx
async function Page() {
  const data = await fetchData() // 페이지 전체를 블로킹
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  )
}

```

중간 섹션만 data가 필요함에도 불구하고 전체 layout이 data를 기다립니다.

**Correct: wrapper가 즉시 표시되고 data는 streaming됨**

```tsx
function Page() {
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <Suspense fallback={<Skeleton />}>
          <DataDisplay />
        </Suspense>
      </div>
      <div>Footer</div>
    </div>
  )
}

async function DataDisplay() {
  const data = await fetchData() // 이 component만 블로킹
  return <div>{data.content}</div>
}

```

Sidebar, Header, Footer는 즉시 렌더링됩니다. DataDisplay만 data를 기다립니다.

**대안: component 간 promise 공유**

```tsx
function Page() {
  // 즉시 fetch를 시작하지만, await하지 않음
  const dataPromise = fetchData()
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <Suspense fallback={<Skeleton />}>
        <DataDisplay dataPromise={dataPromise} />
        <DataSummary dataPromise={dataPromise} />
      </Suspense>
      <div>Footer</div>
    </div>
  )
}

function DataDisplay({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // promise를 unwrap함
  return <div>{data.content}</div>
}

function DataSummary({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // 동일한 promise를 재사용
  return <div>{data.summary}</div>
}

```

두 component가 동일한 promise를 공유하므로 fetch는 한 번만 발생합니다. Layout은 즉시 렌더링되고 두 component는 함께 기다립니다.

**이 pattern을 사용하지 말아야 할 때:**

* Layout 결정에 필수적인 data인 경우 (위치에 영향을 줄 때)
* Above the fold에 위치한 SEO에 중요한 content인 경우
* Suspense overhead의 가치가 없는 작고 빠른 query인 경우
* Layout shift(loading → content 점프)를 피하고 싶을 때

**Trade-off:** 더 빠른 initial paint vs 잠재적인 layout shift. UX 우선순위에 따라 선택하세요.

---

## 2. Bundle Size Optimization

**Impact: CRITICAL**

Initial bundle size를 줄이면 Time to Interactive와 Largest Contentful Paint가 개선됩니다.

### 2.1 Barrel File Import 피하기

**Impact: CRITICAL (200-800ms import 비용, 느린 build)**

수천 개의 사용되지 않는 module이 로드되는 것을 피하기 위해 barrel file 대신 source file에서 직접 import하세요. **Barrel file**은 여러 module을 다시 export하는 entry point입니다 (예: `export * from './module'`를 수행하는 `index.js`).

인기 있는 icon 및 component library는 entry file에 **최대 10,000개의 re-export**를 가질 수 있습니다. 많은 React package의 경우 **import하는 데만 200-800ms가 소요**되어, 개발 속도와 production cold start 모두에 영향을 줍니다.

**Tree-shaking이 도움이 되지 않는 이유:** Library가 external(bundled 아님)로 표시되면 bundler가 이를 optimize할 수 없습니다. Tree-shaking을 위해 bundle에 포함시키면 전체 module graph를 분석하느라 build가 상당히 느려집니다.

**Incorrect: library 전체를 import함**

```tsx
import { Check, X, Menu } from 'lucide-react'
// 1,583개 module 로드, dev에서 약 2.8초 추가 소요
// Runtime 비용: 매 cold start마다 200-800ms

import { Button, TextField } from '@mui/material'
// 2,225개 module 로드, dev에서 약 4.2초 추가 소요

```

**Correct: 필요한 것만 import함**

```tsx
import Check from 'lucide-react/dist/esm/icons/check'
import X from 'lucide-react/dist/esm/icons/x'
import Menu from 'lucide-react/dist/esm/icons/menu'
// 오직 3개 module만 로드 (~2KB vs ~1MB)

import Button from '@mui/material/Button'
import TextField from '@mui/material/TextField'
// 사용하는 것만 로드

```

**대안: Next.js 13.5+**

```js
// next.config.js - optimizePackageImports 사용
module.exports = {
  experimental: {
    optimizePackageImports: ['lucide-react', '@mui/material']
  }
}

// 그러면 편리한 barrel import를 유지할 수 있습니다:
import { Check, X, Menu } from 'lucide-react'
// Build time에 직접 import로 자동 변환됩니다.

```

직접 import는 15-70% 빠른 dev boot, 28% 빠른 build, 40% 빠른 cold start, 그리고 훨씬 빠른 HMR을 제공합니다.

주로 영향을 받는 library: `lucide-react`, `@mui/material`, `@mui/icons-material`, `@tabler/icons-react`, `react-icons`, `@headlessui/react`, `@radix-ui/react-*`, `lodash`, `ramda`, `date-fns`, `rxjs`, `react-use`.

Reference: [https://vercel.com/blog/how-we-optimized-package-imports-in-next-js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)

### 2.2 조건부 Module Loading

**Impact: HIGH (필요할 때만 대용량 data 로드)**

Feature가 활성화될 때만 대용량 data나 module을 로드하세요.

**예시: animation frame lazy-load**

```tsx
function AnimationPlayer({ enabled, setEnabled }: { enabled: boolean; setEnabled: React.Dispatch<React.SetStateAction<boolean>> }) {
  const [frames, setFrames] = useState<Frame[] | null>(null)

  useEffect(() => {
    if (enabled && !frames && typeof window !== 'undefined') {
      import('./animation-frames.js')
        .then(mod => setFrames(mod.frames))
        .catch(() => setEnabled(false))
    }
  }, [enabled, frames, setEnabled])

  if (!frames) return <Skeleton />
  return <Canvas frames={frames} />
}

```

`typeof window !== 'undefined'` 체크는 SSR 시 이 module이 bundling되는 것을 방지하여, server bundle size와 build speed를 optimize합니다.

### 2.3 비임계(Non-Critical) 제3자 Library 지연

**Impact: MEDIUM (hydration 후 로드)**

Analytics, logging, error tracking은 user interaction을 블로킹하지 않습니다. Hydration 후에 로드하세요.

**Incorrect: initial bundle을 블로킹함**

```tsx
import { Analytics } from '@vercel/analytics/react'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}

```

**Correct: hydration 후 로드**

```tsx
import dynamic from 'next/dynamic'

const Analytics = dynamic(
  () => import('@vercel/analytics/react').then(m => m.Analytics),
  { ssr: false }
)

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}

```

### 2.4 무거운 Component에 Dynamic Import 사용

**Impact: CRITICAL (TTI 및 LCP에 직접적인 영향)**

Initial render에 필요하지 않은 대형 component는 `next/dynamic`을 사용해 lazy-load하세요.

**Incorrect: Monaco가 main chunk에 포함되어 약 300KB 차지**

```tsx
import { MonacoEditor } from './monaco-editor'

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}

```

**Correct: Monaco를 필요할 때 로드**

```tsx
import dynamic from 'next/dynamic'

const MonacoEditor = dynamic(
  () => import('./monaco-editor').then(m => m.MonacoEditor),
  { ssr: false }
)

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}

```

### 2.5 사용자 의도 기반 Preload

**Impact: MEDIUM (체감 latency 감소)**

Perceived latency를 줄이기 위해 무거운 bundle이 필요하기 전에 미리 로드하세요.

**예시: hover/focus 시 preload**

```tsx
function EditorButton({ onClick }: { onClick: () => void }) {
  const preload = () => {
    if (typeof window !== 'undefined') {
      void import('./monaco-editor')
    }
  }

  return (
    <button
      onMouseEnter={preload}
      onFocus={preload}
      onClick={onClick}
    >
      Open Editor
    </button>
  )
}

```

**예시: feature flag 활성화 시 preload**

```tsx
function FlagsProvider({ children, flags }: Props) {
  useEffect(() => {
    if (flags.editorEnabled && typeof window !== 'undefined') {
      void import('./monaco-editor').then(mod => mod.init())
    }
  }, [flags.editorEnabled])

  return <FlagsContext.Provider value={flags}>
    {children}
  </FlagsContext.Provider>
}

```

`typeof window !== 'undefined'` 체크는 preload된 module이 SSR 시 bundling되는 것을 방지합니다.

---

## 3. Server-Side Performance

**Impact: HIGH**

Server-side rendering과 data fetching을 optimize하면 server-side waterfall이 제거되고 response time이 단축됩니다.

### 3.1 Server Action을 API Route처럼 인증

**Impact: CRITICAL (server mutation에 대한 무단 접근 방지)**

Server Action(`"use server"`가 선언된 함수)은 API route와 마찬가지로 public endpoint로 노출됩니다. Server Action은 직접 호출될 수 있으므로, middleware, layout guard, page-level 체크에만 의존하지 말고 **각 Server Action 내부에서** 항상 authentication과 authorization을 확인하세요.

Next.js 문서는 다음과 같이 명시합니다: "Server Action을 외부로 노출된 API endpoint와 동일한 보안 고려 사항으로 취급하고, 사용자가 mutation을 수행할 권한이 있는지 확인하십시오."

**Incorrect: authentication 체크 없음**

```typescript
'use server'

export async function deleteUser(userId: string) {
  // 누구나 호출 가능! 인증 체크 없음
  await db.user.delete({ where: { id: userId } })
  return { success: true }
}

```

**Correct: action 내부에서 authentication 수행**

```typescript
'use server'

import { verifySession } from '@/lib/auth'
import { unauthorized } from '@/lib/errors'

export async function deleteUser(userId: string) {
  // 항상 action 내부에서 auth 체크
  const session = await verifySession()
  
  if (!session) {
    throw unauthorized('Must be logged in')
  }
  
  // authorization도 체크
  if (session.user.role !== 'admin' && session.user.id !== userId) {
    throw unauthorized('Cannot delete other users')
  }
  
  await db.user.delete({ where: { id: userId } })
  return { success: true }
}

```

**Input validation 포함:**

```typescript
'use server'

import { verifySession } from '@/lib/auth'
import { z } from 'zod'

const updateProfileSchema = z.object({
  userId: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email()
})

export async function updateProfile(data: unknown) {
  // 먼저 input validation
  const validated = updateProfileSchema.parse(data)
  
  // 그다음 authenticate
  const session = await verifySession()
  if (!session) {
    throw new Error('Unauthorized')
  }
  
  // 그다음 authorize
  if (session.user.id !== validated.userId) {
    throw new Error('Can only update own profile')
  }
  
  // 최종적으로 mutation 수행
  await db.user.update({
    where: { id: validated.userId },
    data: {
      name: validated.name,
      email: validated.email
    }
  })
  
  return { success: true }
}

```

Reference: [https://nextjs.org/docs/app/guides/authentication](https://nextjs.org/docs/app/guides/authentication)

### 3.2 RSC Prop에서 중복 Serialization 피하기

**Impact: LOW (중복 serialization을 피하여 network payload 감소)**

RSC→client serialization은 value가 아닌 object reference로 중복을 제거(deduplication)합니다. 동일 reference = 1회 serialize; 새 reference = 다시 serialize. 변환 작업(`.toSorted()`, `.filter()`, `.map()`)은 server가 아닌 client에서 수행하세요.

**Incorrect: array를 중복시킴**

```tsx
// RSC: 6개의 string 전송 (2개 array × 3개 item)
<ClientList usernames={usernames} usernamesOrdered={usernames.toSorted()} />

```

**Correct: 3개의 string 전송**

```tsx
// RSC: 한 번만 전송
<ClientList usernames={usernames} />

// Client: 거기서 변환
'use client'
const sorted = useMemo(() => [...usernames].sort(), [usernames])

```

**중첩된 deduplication 동작:**

```tsx
// string[] - 모든 것이 중복됨
usernames={['a','b']} sorted={usernames.toSorted()} // 4개 string 전송

// object[] - array 구조만 중복됨
users={[{id:1},{id:2}]} sorted={users.toSorted()} // 2개 array + 2개 고유 object 전송 (4개 아님)

```

Deduplication은 재귀적으로 작동합니다. Data type에 따라 영향이 다릅니다:

* `string[]`, `number[]`, `boolean[]`: **HIGH impact** - array와 모든 primitive가 완전히 중복됨
* `object[]`: **LOW impact** - array는 중복되지만, 중첩된 object는 reference로 deduplicated됨

**Deduplication을 깨뜨리는 작업: 새 reference 생성**

* Arrays: `.toSorted()`, `.filter()`, `.map()`, `.slice()`, `[...arr]`
* Objects: `{...obj}`, `Object.assign()`, `structuredClone()`, `JSON.parse(JSON.stringify())`

**추가 예시:**

```tsx
// ❌ Bad
<C users={users} active={users.filter(u => u.active)} />
<C product={product} productName={product.name} />

// ✅ Good
<C users={users} />
<C product={product} />
// Filtering/destructuring은 client에서 수행

```

**예외:** 변환 비용이 매우 비싸거나 client가 원본 data를 필요로 하지 않는 경우에는 파생 data를 전달하세요.

### 3.3 Cross-Request LRU Caching

**Impact: HIGH (여러 request에 걸쳐 캐싱)**

`React.cache()`는 단일 request 내에서만 작동합니다. 연속적인 request(사용자가 버튼 A를 누른 후 버튼 B를 누름) 간에 공유되는 data의 경우 LRU cache를 사용하세요.

**구현:**

```typescript
import { LRUCache } from 'lru-cache'

const cache = new LRUCache<string, any>({
  max: 1000,
  ttl: 5 * 60 * 1000  // 5분
})

export async function getUser(id: string) {
  const cached = cache.get(id)
  if (cached) return cached

  const user = await db.user.findUnique({ where: { id } })
  cache.set(id, user)
  return user
}

// Request 1: DB query 수행, 결과 캐싱됨
// Request 2: cache hit, DB query 없음

```

수 초 내에 동일한 data를 필요로 하는 여러 endpoint에 연속적인 사용자 작업이 도달할 때 사용하세요.

**Vercel의 [Fluid Compute](https://vercel.com/docs/fluid-compute) 사용 시:** 여러 동시 request가 동일한 function instance와 cache를 공유할 수 있으므로 LRU caching이 특히 효과적입니다. 이는 Redis와 같은 외부 storage 없이도 request 간에 cache가 유지됨을 의미합니다.

**전통적인 serverless 환경:** 각 호출이 격리되어 실행되므로, process 간 caching을 위해 Redis 고려가 필요할 수 있습니다.

Reference: [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)

### 3.4 RSC Boundary에서 Serialization 최소화

**Impact: HIGH (data 전송 크기 감소)**

React Server/Client boundary는 모든 object property를 string으로 serialize하여 HTML response와 후속 RSC request에 포함시킵니다. 이 serialize된 data는 page weight와 load time에 직접적인 영향을 주므로 **크기가 매우 중요합니다**. Client가 실제로 사용하는 field만 전달하세요.

**Incorrect: 50개 field 전체를 serialize함**

```tsx
async function Page() {
  const user = await fetchUser()  // 50개 field
  return <Profile user={user} />
}

'use client'
function Profile({ user }: { user: User }) {
  return <div>{user.name}</div>  // 1개 field만 사용
}

```

**Correct: 1개 field만 serialize함**

```tsx
async function Page() {
  const user = await fetchUser()
  return <Profile name={user.name} />
}

'use client'
function Profile({ name }: { name: string }) {
  return <div>{name}</div>
}

```

### 3.5 Component Composition을 통한 병렬 Data Fetching

**Impact: CRITICAL (server-side waterfall 제거)**

React Server Component는 tree 내에서 순차적으로 실행됩니다. Composition 구조로 재구성하여 data fetching을 병렬화하세요.

**Incorrect: Sidebar가 Page의 fetch 완료를 기다림**

```tsx
export default async function Page() {
  const header = await fetchHeader()
  return (
    <div>
      <div>{header}</div>
      <Sidebar />
    </div>
  )
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}

```

**Correct: 두 가지가 동시에 fetch됨**

```tsx
async function Header() {
  const data = await fetchHeader()
  return <div>{data}</div>
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}

export default function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
    </div>
  )
}

```

**children prop을 사용한 대안:**

```tsx
async function Header() {
  const data = await fetchHeader()
  return <div>{data}</div>
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}

function Layout({ children }: { children: ReactNode }) {
  return (
    <div>
      <Header />
      {children}
    </div>
  )
}

export default function Page() {
  return (
    <Layout>
      <Sidebar />
    </Layout>
  )
}

```

### 3.6 React.cache()를 이용한 Request별 Deduplication

**Impact: MEDIUM (request 내 중복 제거)**

Server-side request deduplication을 위해 `React.cache()`를 사용하세요. Authentication과 database query에서 가장 큰 이점이 있습니다.

**Usage:**

```typescript
import { cache } from 'react'

export const getCurrentUser = cache(async () => {
  const session = await auth()
  if (!session?.user?.id) return null
  return await db.user.findUnique({
    where: { id: session.user.id }
  })
})

```

단일 request 내에서 `getCurrentUser()`를 여러 번 호출해도 query는 한 번만 실행됩니다.

**Argument로 inline object 사용 피하기:**

`React.cache()`는 cache hit 여부를 판단하기 위해 shallow equality(`Object.is`)를 사용합니다. Inline object는 호출할 때마다 새 reference를 생성하여 cache hit를 방해합니다.

**Incorrect: 항상 cache miss 발생**

```typescript
const getUser = cache(async (params: { uid: number }) => {
  return await db.user.findUnique({ where: { id: params.uid } })
})

// 호출마다 새 object 생성, 절대 cache hit 안 됨
getUser({ uid: 1 })
getUser({ uid: 1 })  // Cache miss, 다시 query 실행

```

**Correct: cache hit 발생**

```typescript
const params = { uid: 1 }
getUser(params)  // Query 실행
getUser(params)  // Cache hit (동일 reference)

```

Object를 전달해야 한다면 동일한 reference를 전달하세요.

**Next.js 전용 참고:**

Next.js에서 `fetch` API는 request memoization 기능이 자동으로 확장되어 있습니다. URL과 option이 동일한 request는 단일 request 내에서 자동으로 deduplicated되므로 `fetch` 호출에는 `React.cache()`가 필요하지 않습니다. 하지만 그 외의 async 작업에는 여전히 `React.cache()`가 필수적입니다:

* Database query (Prisma, Drizzle 등)
* Heavy computation
* Authentication check
* File system operation
* 모든 non-fetch async 작업

Component tree 전체에서 이러한 작업들을 deduplicate하려면 `React.cache()`를 사용하세요.

Reference: [https://react.dev/reference/react/cache](https://react.dev/reference/react/cache)

### 3.7 Non-Blocking 작업을 위한 after() 사용

**Impact: MEDIUM (더 빠른 response time)**

Response가 전송된 후 실행되어야 하는 작업을 스케줄링하려면 Next.js의 `after()`를 사용하세요. 이를 통해 logging, analytics 및 기타 side effect가 response를 블로킹하는 것을 방지할 수 있습니다.

**Incorrect: response를 블로킹함**

```tsx
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // Mutation 수행
  await updateDatabase(request)
  
  // Logging이 response를 블로킹함
  const userAgent = request.headers.get('user-agent') || 'unknown'
  await logUserAction({ userAgent })
  
  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}

```

**Correct: non-blocking**

```tsx
import { after } from 'next/server'
import { headers, cookies } from 'next/headers'
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // Mutation 수행
  await updateDatabase(request)
  
  // Response 전송 후 log 기록
  after(async () => {
    const userAgent = (await headers()).get('user-agent') || 'unknown'
    const sessionCookie = (await cookies()).get('session-id')?.value || 'anonymous'
    
    logUserAction({ sessionCookie, userAgent })
  })
  
  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}

```

Logging이 백그라운드에서 진행되는 동안 response는 즉시 전송됩니다.

**주요 Use case:**

* Analytics tracking
* Audit logging
* Notification 전송
* Cache invalidation
* Cleanup task

**중요 사항:**

* `after()`는 response가 실패하거나 redirect되어도 실행됩니다.
* Server Action, Route Handler, Server Component에서 작동합니다.

Reference: [https://nextjs.org/docs/app/api-reference/functions/after](https://nextjs.org/docs/app/api-reference/functions/after)

---

## 4. Client-Side Data Fetching

**Impact: MEDIUM-HIGH**

자동 deduplication과 효율적인 data fetching pattern은 불필요한 network request를 줄여줍니다.

### 4.1 Global Event Listener Deduplication

**Impact: LOW (N개 component에 대해 단일 listener)**

Component instance 간에 global event listener를 공유하려면 `useSWRSubscription()`을 사용하세요.

**Incorrect: N개 instance = N개 listener**

```tsx
function useKeyboardShortcut(key: string, callback: () => void) {
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && e.key === key) {
        callback()
      }
    }
    window.addEventListener('keydown', handler)
    return () => window.removeEventListener('keydown', handler)
  }, [key, callback])
}

```

`useKeyboardShortcut` hook을 여러 번 사용하면 각 instance마다 새 listener가 등록됩니다.

**Correct: N개 instance = 1개 listener**

```tsx
import useSWRSubscription from 'swr/subscription'

// key별 callback을 추적하는 Module-level Map
const keyCallbacks = new Map<string, Set<() => void>>()

function useKeyboardShortcut(key: string, callback: () => void) {
  // Map에 이 callback 등록
  useEffect(() => {
    if (!keyCallbacks.has(key)) {
      keyCallbacks.set(key, new Set())
    }
    keyCallbacks.get(key)!.add(callback)

    return () => {
      const set = keyCallbacks.get(key)
      if (set) {
        set.delete(callback)
        if (set.size === 0) {
          keyCallbacks.delete(key)
        }
      }
    }
  }, [key, callback])

  useSWRSubscription('global-keydown', () => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && keyCallbacks.has(e.key)) {
        keyCallbacks.get(e.key)!.forEach(cb => cb())
      }
    }
    window.addEventListener('keydown', handler)
    return () => window.removeEventListener('keydown', handler)
  })
}

function Profile() {
  // 여러 shortcut이 동일한 listener를 공유함
  useKeyboardShortcut('p', () => { /* ... */ }) 
  useKeyboardShortcut('k', () => { /* ... */ })
  // ...
}

```

### 4.2 Scrolling 성능을 위해 Passive Event Listener 사용

**Impact: MEDIUM (event listener로 인한 scroll 지연 제거)**

즉각적인 scrolling이 가능하도록 touch 및 wheel event listener에 `{ passive: true }`를 추가하세요. 브라우저는 원래 `preventDefault()` 호출 여부를 확인하기 위해 listener가 끝날 때까지 기다리며, 이로 인해 scroll 지연이 발생합니다.

**Incorrect:**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)
  
  document.addEventListener('touchstart', handleTouch)
  document.addEventListener('wheel', handleWheel)
  
  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])

```

**Correct:**

```typescript
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)
  
  document.addEventListener('touchstart', handleTouch, { passive: true })
  document.addEventListener('wheel', handleWheel, { passive: true })
  
  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])

```

**Passive 사용 시기:** tracking/analytics, logging, `preventDefault()`를 호출하지 않는 모든 listener.

**Passive 미사용 시기:** 커스텀 swipe 제스처, 커스텀 zoom 제어 등 `preventDefault()`가 필요한 listener.

### 4.3 자동 Deduplication을 위해 SWR 사용

**Impact: MEDIUM-HIGH (자동 중복 제거)**

SWR은 component instance 간의 request deduplication, caching, revalidation을 가능하게 합니다.

**Incorrect: deduplication 없음, 각 instance가 각각 fetch함**

```tsx
function UserList() {
  const [users, setUsers] = useState([])
  useEffect(() => {
    fetch('/api/users')
      .then(r => r.json())
      .then(setUsers)
  }, [])
}

```

**Correct: 여러 instance가 하나의 request를 공유함**

```tsx
import useSWR from 'swr'

function UserList() {
  const { data: users } = useSWR('/api/users', fetcher)
}

```

**Immutable data의 경우:**

```tsx
import { useImmutableSWR } from '@/lib/swr'

function StaticContent() {
  const { data } = useImmutableSWR('/api/config', fetcher)
}

```

**Mutation의 경우:**

```tsx
import { useSWRMutation } from 'swr/mutation'

function UpdateButton() {
  const { trigger } = useSWRMutation('/api/user', updateUser)
  return <button onClick={() => trigger()}>Update</button>
}

```

Reference: [https://swr.vercel.app](https://swr.vercel.app)

### 4.4 localStorage Data 버전 관리 및 최소화

**Impact: MEDIUM (schema 충돌 방지, storage 크기 감소)**

Key에 버전 prefix를 추가하고 필요한 field만 저장하세요. Schema 충돌과 민감한 data의 실수에 의한 저장을 방지합니다.

**Incorrect:**

```typescript
// 버전 없음, 모든 것을 저장, error handling 없음
localStorage.setItem('userConfig', JSON.stringify(fullUserObject))
const data = localStorage.getItem('userConfig')

```

**Correct:**

```typescript
const VERSION = 'v2'

function saveConfig(config: { theme: string; language: string }) {
  try {
    localStorage.setItem(`userConfig:${VERSION}`, JSON.stringify(config))
  } catch {
    // Incognito mode, quota 초과, 또는 비활성화 시 throw됨
  }
}

function loadConfig() {
  try {
    const data = localStorage.getItem(`userConfig:${VERSION}`)
    return data ? JSON.parse(data) : null
  } catch {
    return null
  }
}

// v1에서 v2로 migration
function migrate() {
  try {
    const v1 = localStorage.getItem('userConfig:v1')
    if (v1) {
      const old = JSON.parse(v1)
      saveConfig({ theme: old.darkMode ? 'dark' : 'light', language: old.lang })
      localStorage.removeItem('userConfig:v1')
    }
  } catch {}
}

```

**Server response 중 최소한의 field만 저장:**

```typescript
// User object에 20개 이상의 field가 있어도 UI에 필요한 것만 저장
function cachePrefs(user: FullUser) {
  try {
    localStorage.setItem('prefs:v1', JSON.stringify({
      theme: user.preferences.theme,
      notifications: user.preferences.notifications
    }))
  } catch {}
}

```

**항상 try-catch로 감싸기:** `getItem()`과 `setItem()`은 incognito mode(Safari, Firefox), quota 초과, 또는 비활성화 상태에서 throw될 수 있습니다.

**이점:** 버전을 통한 schema 발전 가능, storage 크기 감소, token/PII/internal flag 저장 방지.

---

## 5. Re-render Optimization

**Impact: MEDIUM**

불필요한 re-render를 줄이면 낭비되는 연산을 최소화하고 UI responsiveness를 향상시킬 수 있습니다.

### 5.1 Rendering 중에 파생 State 계산

**Impact: MEDIUM (중복 렌더링 및 state drift 방지)**

현재 props/state로부터 계산될 수 있는 값이라면, state에 저장하거나 effect에서 업데이트하지 마세요. Extra render와 state drift를 피하기 위해 render 중에 파생(derive)시키세요. Prop 변경에 반응하기 위해서만 effect에서 state를 설정하지 마세요. 대신 파생된 값을 사용하거나 keyed reset을 선호하세요.

**Incorrect: 중복된 state와 effect**

```tsx
function Form() {
  const [firstName, setFirstName] = useState('First')
  const [lastName, setLastName] = useState('Last')
  const [fullName, setFullName] = useState('')

  useEffect(() => {
    setFullName(firstName + ' ' + lastName)
  }, [firstName, lastName])

  return <p>{fullName}</p>
}

```

**Correct: render 중에 파생**

```tsx
function Form() {
  const [firstName, setFirstName] = useState('First')
  const [lastName, setLastName] = useState('Last')
  const fullName = firstName + ' ' + lastName

  return <p>{fullName}</p>
}

```

Reference: [https://react.dev/learn/you-might-not-need-an-effect](https://react.dev/learn/you-might-not-need-an-effect)

### 5.2 State 읽기를 사용 시점으로 지연

**Impact: MEDIUM (불필요한 subscription 방지)**

Dynamic state(`searchParams`, `localStorage`)를 callback 내부에서만 읽는다면 해당 state에 직접 subscribe하지 마세요.

**Incorrect: 모든 searchParams 변경에 subscribe함**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const searchParams = useSearchParams()

  const handleShare = () => {
    const ref = searchParams.get('ref')
    shareChat(chatId, { ref })
  }

  return <button onClick={handleShare}>Share</button>
}

```

**Correct: 필요할 때 읽기, subscription 없음**

```tsx
function ShareButton({ chatId }: { chatId: string }) {
  const handleShare = () => {
    const params = new URLSearchParams(window.location.search)
    const ref = params.get('ref')
    shareChat(chatId, { ref })
  }

  return <button onClick={handleShare}>Share</button>
}

```

### 5.3 Primitive 결과 타입을 갖는 단순 표현식에 useMemo 사용 금지

**Impact: LOW-MEDIUM (매 render마다 낭비되는 연산)**

표현식이 단순하고(논리/산술 연산자 몇 개 수준) 결과 타입이 primitive(boolean, number, string)인 경우 `useMemo`로 감싸지 마세요.

`useMemo`를 호출하고 hook dependency를 비교하는 것이 표현식 자체를 계산하는 것보다 더 많은 리소스를 소모할 수 있습니다.

**Incorrect:**

```tsx
function Header({ user, notifications }: Props) {
  const isLoading = useMemo(() => {
    return user.isLoading || notifications.isLoading
  }, [user.isLoading, notifications.isLoading])

  if (isLoading) return <Skeleton />
  // ...
}

```

**Correct:**

```tsx
function Header({ user, notifications }: Props) {
  const isLoading = user.isLoading || notifications.isLoading

  if (isLoading) return <Skeleton />
  // ...
}

```

### 5.4 Memoized Component의 Non-primitive Parameter Default Value를 상수로 추출

**Impact: MEDIUM (default value에 상수를 사용하여 memoization 복구)**

Memoized component가 array, function, object와 같은 non-primitive optional parameter에 대해 default value를 가질 때, 해당 parameter 없이 component를 호출하면 memoization이 깨집니다. 매 rerender마다 새로운 value instance가 생성되어 `memo()`의 strict equality 비교를 통과하지 못하기 때문입니다.

이 문제를 해결하려면 default value를 상수로 추출하세요.

**Incorrect: 매 rerender마다 `onClick`이 다른 값을 가짐**

```tsx
const UserAvatar = memo(function UserAvatar({ onClick = () => {} }: { onClick?: () => void }) {
  // ...
})

// optional onClick 없이 사용됨
<UserAvatar />

```

**Correct: stable한 default value**

```tsx
const NOOP = () => {};

const UserAvatar = memo(function UserAvatar({ onClick = NOOP }: { onClick?: () => void }) {
  // ...
})

// optional onClick 없이 사용됨
<UserAvatar />

```

### 5.5 Memoized Component로 추출

**Impact: MEDIUM (early return 가능)**

비싼 작업을 memoized component로 추출하여 계산 전에 early return이 가능하도록 하세요.

**Incorrect: loading 중에도 avatar를 계산함**

```tsx
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => {
    const id = computeAvatarId(user)
    return <Avatar id={id} />
  }, [user])

  if (loading) return <Skeleton />
  return <div>{avatar}</div>
}

```

**Correct: loading 중에는 계산을 건너뜀**

```tsx
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const id = useMemo(() => computeAvatarId(user), [user])
  return <Avatar id={id} />
})

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />
  return (
    <div>
      <UserAvatar user={user} />
    </div>
  )
}

```

**참고:** 프로젝트에 [React Compiler](https://react.dev/learn/react-compiler)가 활성화되어 있다면 `memo()`와 `useMemo()`를 사용한 수동 memoization은 필요하지 않습니다. Compiler가 re-render를 자동으로 optimize합니다.

### 5.6 Effect Dependency 좁히기

**Impact: LOW (effect 재실행 최소화)**

Effect 재실행을 최소화하기 위해 object 대신 primitive dependency를 지정하세요.

**Incorrect: user의 어떤 field라도 바뀌면 재실행됨**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user])

```

**Correct: id가 바뀔 때만 재실행됨**

```tsx
useEffect(() => {
  console.log(user.id)
}, [user.id])

```

**파생 State의 경우 effect 밖에서 계산:**

```tsx
// Incorrect: width가 767, 766, 765...일 때마다 실행됨
useEffect(() => {
  if (width < 768) {
    enableMobileMode()
  }
}, [width])

// Correct: boolean transition 시에만 실행됨
const isMobile = width < 768
useEffect(() => {
  if (isMobile) {
    enableMobileMode()
  }
}, [isMobile])

```

### 5.7 Interaction Logic을 Event Handler에 배치

**Impact: MEDIUM (effect 재실행 및 중복 side effect 방지)**

Side effect가 특정 사용자 action(submit, click, drag)에 의해 트리거된다면, 해당 event handler에서 실행하세요. Action을 state + effect로 모델링하지 마세요. 그렇게 하면 관련 없는 변경에도 effect가 재실행되거나 action이 중복될 수 있습니다.

**Incorrect: action이 state + effect로 모델링됨**

```tsx
function Form() {
  const [submitted, setSubmitted] = useState(false)
  const theme = useContext(ThemeContext)

  useEffect(() => {
    if (submitted) {
      post('/api/register')
      showToast('Registered', theme)
    }
  }, [submitted, theme])

  return <button onClick={() => setSubmitted(true)}>Submit</button>
}

```

**Correct: handler에서 처리**

```tsx
function Form() {
  const theme = useContext(ThemeContext)

  function handleSubmit() {
    post('/api/register')
    showToast('Registered', theme)
  }

  return <button onClick={handleSubmit}>Submit</button>
}

```

Reference: [https://react.dev/learn/removing-effect-dependencies#should-this-code-move-to-an-event-handler](https://react.dev/learn/removing-effect-dependencies#should-this-code-move-to-an-event-handler)

### 5.8 파생 State에 Subscribe

**Impact: MEDIUM (re-render 빈도 감소)**

Re-render 빈도를 줄이기 위해 연속적인 값 대신 파생된 boolean state에 subscribe하세요.

**Incorrect: pixel이 변할 때마다 re-render됨**

```tsx
function Sidebar() {
  const width = useWindowWidth()  // 지속적으로 업데이트됨
  const isMobile = width < 768
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}

```

**Correct: boolean이 변할 때만 re-render됨**

```tsx
function Sidebar() {
  const isMobile = useMediaQuery('(max-width: 767px)')
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}

```

### 5.9 Functional setState Update 사용

**Impact: MEDIUM (stale closure 방지 및 불필요한 callback 생성 제거)**

현재 state 값을 기반으로 state를 업데이트할 때는 state 변수를 직접 참조하는 대신 setState의 functional update 형태를 사용하세요. 이는 stale closure를 방지하고, 불필요한 dependency를 제거하며, stable한 callback reference를 생성합니다.

**Incorrect: dependency로 state가 필요함**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)
  
  // Callback이 items에 의존해야 하므로, items가 바뀔 때마다 재생성됨
  const addItems = useCallback((newItems: Item[]) => {
    setItems([...items, ...newItems])
  }, [items])  // ❌ items dependency로 인한 재생성
  
  // Dependency를 잊으면 stale closure 위험
  const removeItem = useCallback((id: string) => {
    setItems(items.filter(item => item.id !== id))
  }, [])  // ❌ items dependency 누락 - stale items 사용!
  
  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}

```

첫 번째 callback은 `items`가 바뀔 때마다 재생성되어 하위 component의 불필요한 re-render를 유발할 수 있습니다. 두 번째 callback은 stale closure bug가 있어 항상 초기 `items` 값을 참조하게 됩니다.

**Correct: stable callback, stale closure 없음**

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)
  
  // Stable callback, 절대 재생성되지 않음
  const addItems = useCallback((newItems: Item[]) => {
    setItems(curr => [...curr, ...newItems])
  }, [])  // ✅ Dependency 필요 없음
  
  // 항상 최신 state 사용, stale closure 위험 없음
  const removeItem = useCallback((id: string) => {
    setItems(curr => curr.filter(item => item.id !== id))
  }, [])  // ✅ 안전하고 stable함
  
  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}

```

**이점:**

1. **Stable callback reference** - State 변경 시 callback을 재생성할 필요가 없음
2. **Stale closure 없음** - 항상 최신 state 값에 대해 동작함
3. **적은 dependency** - Dependency array가 단순해지고 memory leak 감소
4. **Bug 방지** - React의 가장 흔한 closure bug 원인 제거

**Functional update 사용 시기:**

* 현재 state 값에 의존하는 모든 setState
* State가 필요한 useCallback/useMemo 내부
* State를 참조하는 event handler
* State를 업데이트하는 async 작업

**Direct update가 괜찮은 경우:**

* Static value로 설정할 때: `setCount(0)`
* Props/argument로만 설정할 때: `setName(newName)`
* State가 이전 값에 의존하지 않을 때

**참고:** 프로젝트에 [React Compiler](https://react.dev/learn/react-compiler)가 활성화되어 있다면 일부 case는 자동 optimize되지만, 정확성과 stale closure bug 방지를 위해 functional update 사용이 여전히 권장됩니다.

### 5.10 Lazy State Initialization 사용

**Impact: MEDIUM (매 render마다 낭비되는 연산)**

비싼 초기값을 위해 `useState`에 function을 전달하세요. Function 형태가 아니면 초기화가 한 번만 필요함에도 불구하고 매 render마다 initializer가 실행됩니다.

**Incorrect: 매 render마다 실행됨**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex()는 초기화 후에도 매 render마다 실행됨
  const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items))
  const [query, setQuery] = useState('')
  
  // query가 바뀔 때 buildSearchIndex가 불필요하게 다시 실행됨
  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse가 매 render마다 실행됨
  const [settings, setSettings] = useState(
    JSON.parse(localStorage.getItem('settings') || '{}')
  )
  
  return <SettingsForm settings={settings} onChange={setSettings} />
}

```

**Correct: 한 번만 실행됨**

```tsx
function FilteredList({ items }: { items: Item[] }) {
  // buildSearchIndex()는 initial render 시에만 실행됨
  const [searchIndex, setSearchIndex] = useState(() => buildSearchIndex(items))
  const [query, setQuery] = useState('')
  
  return <SearchResults index={searchIndex} query={query} />
}

function UserProfile() {
  // JSON.parse가 initial render 시에만 실행됨
  const [settings, setSettings] = useState(() => {
    const stored = localStorage.getItem('settings')
    return stored ? JSON.parse(stored) : {}
  })
  
  return <SettingsForm settings={settings} onChange={setSettings} />
}

```

LocalStorage/sessionStorage에서 초기값을 계산하거나, data structure(index, map)를 구축하거나, DOM을 읽거나, 무거운 변환을 수행할 때 lazy initialization을 사용하세요.

단순 primitive(`useState(0)`), direct reference(`useState(props.value)`), 또는 가벼운 literal(`useState({})`)에는 function 형태가 필요하지 않습니다.

### 5.11 비긴급(Non-Urgent) 업데이트에 Transition 사용

**Impact: MEDIUM (UI responsiveness 유지)**

UI responsiveness를 유지하기 위해 빈번하고 긴급하지 않은 state update를 transition으로 표시하세요.

**Incorrect: scroll 시마다 UI를 블로킹함**

```tsx
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0)
  useEffect(() => {
    const handler = () => setScrollY(window.scrollY)
    window.addEventListener('scroll', handler, { passive: true })
    return () => window.removeEventListener('scroll', handler)
  }, [])
}

```

**Correct: non-blocking update**

```tsx
import { startTransition } from 'react'

function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0)
  useEffect(() => {
    const handler = () => {
      startTransition(() => setScrollY(window.scrollY))
    }
    window.addEventListener('scroll', handler, { passive: true })
    return () => window.removeEventListener('scroll', handler)
  }, [])
}

```

### 5.12 일시적인 값(Transient Value)에 useRef 사용

**Impact: MEDIUM (빈번한 업데이트 시 불필요한 re-render 방지)**

값이 자주 바뀌지만 매 업데이트마다 re-render가 필요하지 않은 경우(예: mouse tracker, interval, 일시적인 flag) `useState` 대신 `useRef`에 저장하세요. UI용은 state로 유지하고, DOM에 인접한 일시적인 값은 ref를 사용하세요. Ref를 업데이트해도 re-render는 트리거되지 않습니다.

**Incorrect: 매 업데이트마다 렌더링됨**

```tsx
function Tracker() {
  const [lastX, setLastX] = useState(0)

  useEffect(() => {
    const onMove = (e: MouseEvent) => setLastX(e.clientX)
    window.addEventListener('mousemove', onMove)
    return () => window.removeEventListener('mousemove', onMove)
  }, [])

  return (
    <div
      style={{
        position: 'fixed',
        top: 0,
        left: lastX,
        width: 8,
        height: 8,
        background: 'black',
      }}
    />
  )
}

```

**Correct: tracking 시 re-render 없음**

```tsx
function Tracker() {
  const lastXRef = useRef(0)
  const dotRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const onMove = (e: MouseEvent) => {
      lastXRef.current = e.clientX
      const node = dotRef.current
      if (node) {
        node.style.transform = `translateX(${e.clientX}px)`
      }
    }
    window.addEventListener('mousemove', onMove)
    return () => window.removeEventListener('mousemove', onMove)
  }, [])

  return (
    <div
      ref={dotRef}
      style={{
        position: 'fixed',
        top: 0,
        left: 0,
        width: 8,
        height: 8,
        background: 'black',
        transform: 'translateX(0px)',
      }}
    />
  )
}

```

---

## 6. Rendering Performance

**Impact: MEDIUM**

Rendering 프로세스를 optimize하면 브라우저가 해야 할 작업량이 줄어듭니다.

### 6.1 SVG Element 대신 SVG Wrapper 애니메이션화

**Impact: LOW (hardware acceleration 활성화)**

많은 브라우저가 SVG element에 대한 CSS3 animation의 hardware acceleration을 지원하지 않습니다. SVG를 `<div>`로 감싸고 대신 wrapper를 애니메이션화하세요.

**Incorrect: SVG 직접 애니메이션 - hardware acceleration 없음**

```tsx
function LoadingSpinner() {
  return (
    <svg 
      className="animate-spin"
      width="24" 
      height="24" 
      viewBox="0 0 24 24"
    >
      <circle cx="12" cy="12" r="10" stroke="currentColor" />
    </svg>
  )
}

```

**Correct: wrapper div 애니메이션 - hardware accelerated**

```tsx
function LoadingSpinner() {
  return (
    <div className="animate-spin">
      <svg 
        width="24" 
        height="24" 
        viewBox="0 0 24 24"
      >
        <circle cx="12" cy="12" r="10" stroke="currentColor" />
      </svg>
    </div>
  )
}

```

이는 모든 CSS transform 및 transition(`transform`, `opacity`, `translate`, `scale`, `rotate`)에 적용됩니다. Wrapper div를 통해 브라우저가 GPU acceleration을 사용하여 더 부드러운 animation을 구현할 수 있게 합니다.

### 6.2 긴 List를 위한 CSS content-visibility

**Impact: HIGH (더 빠른 initial render)**

화면 밖(off-screen) rendering을 지연시키기 위해 `content-visibility: auto`를 적용하세요.

**CSS:**

```css
.message-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px;
}

```

**Example:**

```tsx
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="overflow-y-auto h-screen">
      {messages.map(msg => (
        <div key={msg.id} className="message-item">
          <Avatar user={msg.author} />
          <div>{msg.content}</div>
        </div>
      ))}
    </div>
  )
}

```

1,000개의 message가 있을 때, 브라우저는 약 990개의 off-screen item에 대해 layout/paint를 건너뜁니다 (initial render 약 10배 향상).

### 6.3 Static JSX Element 호이스팅

**Impact: LOW (재생성 방지)**

재생성을 피하기 위해 static JSX를 component 밖으로 추출하세요.

**Incorrect: 매 render마다 element를 재생성함**

```tsx
function LoadingSkeleton() {
  return <div className="animate-pulse h-20 bg-gray-200" />
}

function Container() {
  return (
    <div>
      {loading && <LoadingSkeleton />}
    </div>
  )
}

```

**Correct: 동일 element를 재사용함**

```tsx
const loadingSkeleton = (
  <div className="animate-pulse h-20 bg-gray-200" />
)

function Container() {
  return (
    <div>
      {loading && loadingSkeleton}
    </div>
  )
}

```

이는 매 render마다 재생성하기 비싼 크고 정적인 SVG node에 특히 도움이 됩니다.

**참고:** 프로젝트에 [React Compiler](https://react.dev/learn/react-compiler)가 활성화되어 있다면, compiler가 자동으로 static JSX element를 hoisting하고 component re-render를 optimize하므로 수동 hoisting은 필요하지 않습니다.

### 6.4 SVG Precision Optimize

**Impact: LOW (파일 크기 감소)**

파일 크기를 줄이기 위해 SVG 좌표 precision(정밀도)을 낮추세요. 최적의 정밀도는 viewBox 크기에 따라 다르지만 일반적으로 정밀도를 낮추는 것을 고려해야 합니다.

**Incorrect: 과도한 정밀도**

```svg
<path d="M 10.293847 20.847362 L 30.938472 40.192837" />

```

**Correct: 소수점 1자리**

```svg
<path d="M 10.3 20.8 L 30.9 40.2" />

```

**SVGO로 자동화:**

```bash
npx svgo --precision=1 --multipass icon.svg

```

### 6.5 Flicker 없는 Hydration Mismatch 방지

**Impact: MEDIUM (시각적 flicker 및 hydration error 방지)**

Client-side storage(`localStorage`, `cookies`)에 의존하는 content를 렌더링할 때, React가 hydrate하기 전에 DOM을 업데이트하는 synchronous script를 주입하여 SSR 중단과 post-hydration flickering을 모두 방지하세요.

**Incorrect: SSR 중단**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  // localStorage는 server에서 사용할 수 없음 - error 발생
  const theme = localStorage.getItem('theme') || 'light'
  
  return (
    <div className={theme}>
      {children}
    </div>
  )
}

```

`localStorage`가 undefined이므로 server-side rendering이 실패합니다.

**Incorrect: 시각적 flickering**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState('light')
  
  useEffect(() => {
    // Hydration 후에 실행됨 - 시각적 flash 유발
    const stored = localStorage.getItem('theme')
    if (stored) {
      setTheme(stored)
    }
  }, [])
  
  return (
    <div className={theme}>
      {children}
    </div>
  )
}

```

Component가 먼저 default value(`light`)로 렌더링된 후 hydration 이후 업데이트되어, 잘못된 content가 잠깐 보이는 flash 현상이 발생합니다.

**Correct: flicker 없음, hydration mismatch 없음**

```tsx
function ThemeWrapper({ children }: { children: ReactNode }) {
  return (
    <>
      <div id="theme-wrapper">
        {children}
      </div>
      <script
        dangerouslySetInnerHTML={{
          __html: `
            (function() {
              try {
                var theme = localStorage.getItem('theme') || 'light';
                var el = document.getElementById('theme-wrapper');
                if (el) el.className = theme;
              } catch (e) {}
            })();
          `,
        }}
      />
    </>
  )
}

```

Inline script는 element가 보이기 전에 synchronous하게 실행되어 DOM이 이미 올바른 값을 갖도록 보장합니다. Flickering도, hydration mismatch도 없습니다.

이 pattern은 theme toggle, user preference, authentication state 등 default value가 flash되는 것을 방지해야 하는 client-only data에 특히 유용합니다.

### 6.6 예상된 Hydration Mismatch 억제

**Impact: LOW-MEDIUM (알려진 차이에 대한 노이즈성 hydration 경고 방지)**

SSR framework(예: Next.js)에서 일부 값은 의도적으로 server와 client가 다를 수 있습니다(random ID, date, locale/timezone formatting 등). 이러한 *예상된* mismatch의 경우, dynamic text를 `suppressHydrationWarning` 속성이 있는 element로 감싸 노이즈성 경고를 방지하세요. 실제 bug를 숨기는 데 사용하지 마세요. 남용하지 마세요.

**Incorrect: 알려진 mismatch 경고 발생**

```tsx
function Timestamp() {
  return <span>{new Date().toLocaleString()}</span>
}

```

**Correct: 예상된 mismatch만 억제**

```tsx
function Timestamp() {
  return (
    <span suppressHydrationWarning>
      {new Date().toLocaleString()}
    </span>
  )
}

```

### 6.7 Show/Hide에 Activity Component 사용

**Impact: MEDIUM (state/DOM 유지)**

가시성이 자주 바뀌는 비싼 component의 state/DOM을 보존하려면 React의 `<Activity>`를 사용하세요.

**Usage:**

```tsx
import { Activity } from 'react'

function Dropdown({ isOpen }: Props) {
  return (
    <Activity mode={isOpen ? 'visible' : 'hidden'}>
      <ExpensiveMenu />
    </Activity>
  )
}

```

비싼 re-render와 state 유실을 방지합니다.

### 6.8 명시적 Conditional Rendering 사용

**Impact: LOW (0 또는 NaN 렌더링 방지)**

Condition이 `0`, `NaN` 또는 렌더링될 수 있는 다른 falsy value일 수 있는 경우, `&&` 대신 명시적인 삼항 연산자(`? :`)를 사용하세요.

**Incorrect: count가 0일 때 "0"을 렌더링함**

```tsx
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count && <span className="badge">{count}</span>}
    </div>
  )
}

// count = 0일 때: <div>0</div> 렌더링
// count = 5일 때: <div><span class="badge">5</span></div> 렌더링

```

**Correct: count가 0일 때 아무것도 렌더링하지 않음**

```tsx
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count > 0 ? <span className="badge">{count}</span> : null}
    </div>
  )
}

// count = 0일 때: <div></div> 렌더링
// count = 5일 때: <div><span class="badge">5</span></div> 렌더링

```

### 6.9 수동 Loading State보다 useTransition 사용

**Impact: LOW (re-render 감소 및 코드 명확성 향상)**

Loading state를 위해 수동으로 `useState`를 사용하는 대신 `useTransition`을 사용하세요. 이는 내장된 `isPending` state를 제공하고 transition을 자동으로 관리합니다.

**Incorrect: 수동 loading state**

```tsx
function SearchResults() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isLoading, setIsLoading] = useState(false)

  const handleSearch = async (value: string) => {
    setIsLoading(true)
    setQuery(value)
    const data = await fetchResults(value)
    setResults(data)
    setIsLoading(false)
  }

  return (
    <>
      <input onChange={(e) => handleSearch(e.target.value)} />
      {isLoading && <Spinner />}
      <ResultsList results={results} />
    </>
  )
}

```

**Correct: 내장 pending state와 함께 useTransition 사용**

```tsx
import { useTransition, useState } from 'react'

function SearchResults() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isPending, startTransition] = useTransition()

  const handleSearch = (value: string) => {
    setQuery(value) // input은 즉시 업데이트
    
    startTransition(async () => {
      // Results fetch 및 업데이트
      const data = await fetchResults(value)
      setResults(data)
    })
  }

  return (
    <>
      <input onChange={(e) => handleSearch(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  )
}

```

**이점:**

* **자동 pending state**: `setIsLoading(true/false)`를 수동으로 관리할 필요 없음
* **Error resilience**: Transition에서 에러가 발생해도 pending state가 올바르게 reset됨
* **Better responsiveness**: 업데이트 중에 UI responsiveness를 유지함
* **Interrupt handling**: 새로운 transition이 진행 중인 transition을 자동으로 취소함

Reference: [https://react.dev/reference/react/useTransition](https://react.dev/reference/react/useTransition)

---

## 7. JavaScript Performance

**Impact: LOW-MEDIUM**

Hot path에 대한 micro-optimization이 모여 유의미한 개선을 만들어냅니다.

### 7.1 Layout Thrashing 피하기

**Impact: MEDIUM (forced synchronous layout 방지 및 성능 병목 현상 감소)**

Style 쓰기(write)와 layout 읽기(read)를 번갈아 수행하지 마세요. Style 변경 사이에 layout property(`offsetWidth`, `getBoundingClientRect()`, `getComputedStyle()` 등)를 읽으면 브라우저는 synchronous reflow를 강제로 트리거합니다.

**괜찮은 예: 브라우저가 style 변경을 batch 처리함**

```typescript
function updateElementStyles(element: HTMLElement) {
  // 각 줄이 style을 무효화하지만, 브라우저는 recalculation을 batch 처리함
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'
}

```

**Incorrect: interleaved read/write가 reflow를 강제함**

```typescript
function layoutThrashing(element: HTMLElement) {
  element.style.width = '100px'
  const width = element.offsetWidth  // Reflow 강제
  element.style.height = '200px'
  const height = element.offsetHeight  // 또 다른 Reflow 강제
}

```

**Correct: batch write 후 한 번만 read**

```typescript
function updateElementStyles(element: HTMLElement) {
  // 모든 write를 한 번에 처리
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'
  
  // 모든 write가 끝난 후 read (단일 reflow)
  const { width, height } = element.getBoundingClientRect()
}

```

**Correct: batch read 후 write**

```typescript
function updateElementStyles(element: HTMLElement) {
  element.classList.add('highlighted-box')
  
  const { width, height } = element.getBoundingClientRect()
}

```

**더 나은 방법: CSS class 사용**

**React 예시:**

```tsx
// Incorrect: style 변경과 layout query를 번갈아 수행
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  const ref = useRef<HTMLDivElement>(null)
  
  useEffect(() => {
    if (ref.current && isHighlighted) {
      ref.current.style.width = '100px'
      const width = ref.current.offsetWidth // Layout 강제
      ref.current.style.height = '200px'
    }
  }, [isHighlighted])
  
  return <div ref={ref}>Content</div>
}

// Correct: class toggle
function Box({ isHighlighted }: { isHighlighted: boolean }) {
  return (
    <div className={isHighlighted ? 'highlighted-box' : ''}>
      Content
    </div>
  )
}

```

가능하다면 inline style보다 CSS class를 선호하세요. CSS file은 브라우저에 캐싱되며, class는 관심사 분리가 더 잘 되고 유지보수가 쉽습니다.

Layout을 유발하는 작업에 대한 자세한 내용은 [이 gist](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)와 [CSS Triggers](https://csstriggers.com/)를 참고하세요.

### 7.2 반복된 Lookup을 위해 Index Map 구축

**Impact: LOW-MEDIUM (1M ops에서 2K ops로)**

동일한 key로 여러 번 `.find()`를 호출해야 한다면 Map을 사용하세요.

**Incorrect (lookup당 O(n)):**

```typescript
function processOrders(orders: Order[], users: User[]) {
  return orders.map(order => ({
    ...order,
    user: users.find(u => u.id === order.userId)
  }))
}

```

**Correct (lookup당 O(1)):**

```typescript
function processOrders(orders: Order[], users: User[]) {
  const userById = new Map(users.map(u => [u.id, u]))

  return orders.map(order => ({
    ...order,
    user: userById.get(order.userId)
  }))
}

```

Map을 한 번 구축(O(n))하면 모든 lookup은 O(1)입니다.
1,000개 order × 1,000개 user인 경우: 1M ops → 2K ops.

### 7.3 Loop 내 Property Access 캐싱

**Impact: LOW-MEDIUM (lookup 감소)**

Hot path에서 object property lookup을 캐싱하세요.

**Incorrect: N번 iteration × 3번 lookup**

```typescript
for (let i = 0; i < arr.length; i++) {
  process(obj.config.settings.value)
}

```

**Correct: 총 1번 lookup**

```typescript
const value = obj.config.settings.value
const len = arr.length
for (let i = 0; i < len; i++) {
  process(value)
}

```

### 7.4 반복되는 Function Call 캐싱

**Impact: MEDIUM (중복 연산 방지)**

Rendering 중에 동일한 input으로 function이 반복적으로 호출될 때, module-level Map을 사용하여 결과를 캐싱하세요.

**Incorrect: 중복 연산**

```typescript
function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // 동일한 프로젝트 이름에 대해 slugify()가 100번 이상 호출됨
        const slug = slugify(project.name)
        
        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}

```

**Correct: 캐싱된 결과**

```typescript
// Module-level cache
const slugifyCache = new Map<string, string>()

function cachedSlugify(text: string): string {
  if (slugifyCache.has(text)) {
    return slugifyCache.get(text)!
  }
  const result = slugify(text)
  slugifyCache.set(text, result)
  return result
}

function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        // 고유한 프로젝트 이름당 한 번만 계산됨
        const slug = cachedSlugify(project.name)
        
        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}

```

**단일 값 Function을 위한 더 단순한 pattern:**

```typescript
let isLoggedInCache: boolean | null = null

function isLoggedIn(): boolean {
  if (isLoggedInCache !== null) {
    return isLoggedInCache
  }
  
  isLoggedInCache = document.cookie.includes('auth=')
  return isLoggedInCache
}

// auth 변경 시 cache 초기화
function onAuthChange() {
  isLoggedInCache = null
}

```

React component뿐만 아니라 utility, event handler 등 어디서나 작동하도록 (hook이 아닌) Map을 사용하세요.

Reference: [https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)

### 7.5 Storage API Call 캐싱

**Impact: LOW-MEDIUM (비싼 I/O 감소)**

`localStorage`, `sessionStorage`, `document.cookie`는 synchronous하며 비쌉니다. 읽기 작업을 memory에 캐싱하세요.

**Incorrect: 호출할 때마다 storage를 읽음**

```typescript
function getTheme() {
  return localStorage.getItem('theme') ?? 'light'
}
// 10번 호출 = 10번 storage 읽기

```

**Correct: Map cache**

```typescript
const storageCache = new Map<string, string | null>()

function getLocalStorage(key: string) {
  if (!storageCache.has(key)) {
    storageCache.set(key, localStorage.getItem(key))
  }
  return storageCache.get(key)
}

function setLocalStorage(key: string, value: string) {
  localStorage.setItem(key, value)
  storageCache.set(key, value) // cache 동기화
}

```

어디서나 작동하도록 Map을 사용하세요.

**Cookie 캐싱:**

```typescript
let cookieCache: Record<string, string> | null = null

function getCookie(name: string) {
  if (!cookieCache) {
    cookieCache = Object.fromEntries(
      document.cookie.split('; ').map(c => c.split('='))
    )
  }
  return cookieCache[name]
}

```

**중요: 외부 변경 시 무효화**

```typescript
window.addEventListener('storage', (e) => {
  if (e.key) storageCache.delete(e.key)
})

document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'visible') {
    storageCache.clear()
  }
})

```

Storage가 외부(다른 tab, server에서 설정한 cookie 등)에서 변경될 수 있다면 cache를 무효화하세요.

### 7.6 여러 번의 Array Iteration 결합

**Impact: LOW-MEDIUM (iteration 감소)**

여러 번의 `.filter()` 또는 `.map()` 호출은 array를 여러 번 순회합니다. 하나의 loop로 결합하세요.

**Incorrect: 3번 순회**

```typescript
const admins = users.filter(u => u.isAdmin)
const testers = users.filter(u => u.isTester)
const inactive = users.filter(u => !u.isActive)

```

**Correct: 1번 순회**

```typescript
const admins: User[] = []
const testers: User[] = []
const inactive: User[] = []

for (const user of users) {
  if (user.isAdmin) admins.push(user)
  if (user.isTester) testers.push(user)
  if (!user.isActive) inactive.push(user)
}

```

### 7.7 Array 비교 시 Early Length Check

**Impact: MEDIUM-HIGH (길이가 다를 때 비싼 연산 방지)**

비싼 연산(sorting, deep equality, serialization)으로 array를 비교할 때는 먼저 길이를 확인하세요. 길이가 다르면 두 array는 같을 수 없습니다.

실제 application에서 이 optimization은 hot path(event handler, render loop)에서 비교를 실행할 때 특히 가치 있습니다.

**Incorrect: 항상 비싼 비교를 실행함**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 길이가 다르더라도 항상 sort와 join을 실행함
  return current.sort().join() !== original.sort().join()
}

```

`current.length`가 5이고 `original.length`가 100일 때도 두 번의 O(n log n) sort가 실행됩니다. 또한 array를 join하고 string을 비교하는 overhead도 발생합니다.

**Correct (먼저 O(1) length check 수행):**

```typescript
function hasChanges(current: string[], original: string[]) {
  // 길이가 다르면 즉시 return
  if (current.length !== original.length) {
    return true
  }
  // 길이가 같을 때만 sort
  const currentSorted = current.toSorted()
  const originalSorted = original.toSorted()
  for (let i = 0; i < currentSorted.length; i++) {
    if (currentSorted[i] !== originalSorted[i]) {
      return true
    }
  }
  return false
}

```

이 방식이 더 효율적인 이유:

* 길이가 다를 때 sorting 및 joining overhead를 피함
* Joined string에 대한 memory 소모를 피함 (특히 대형 array에서 중요)
* 원본 array를 mutate하지 않음
* 차이점을 발견하는 즉시 early return함

### 7.8 Function에서 Early Return

**Impact: LOW-MEDIUM (불필요한 연산 방지)**

결과가 결정되면 일찍 return하여 불필요한 처리를 건너뛰세요.

**Incorrect: 에러를 발견한 후에도 모든 item을 처리함**

```typescript
function validateUsers(users: User[]) {
  let hasError = false
  let errorMessage = ''
  
  for (const user of users) {
    if (!user.email) {
      hasError = true
      errorMessage = 'Email required'
    }
    if (!user.name) {
      hasError = true
      errorMessage = 'Name required'
    }
    // 에러를 찾은 후에도 모든 사용자를 계속 체크함
  }
  
  return hasError ? { valid: false, error: errorMessage } : { valid: true }
}

```

**Correct: 첫 번째 에러 발생 시 즉시 return**

```typescript
function validateUsers(users: User[]) {
  for (const user of users) {
    if (!user.email) {
      return { valid: false, error: 'Email required' }
    }
    if (!user.name) {
      return { valid: false, error: 'Name required' }
    }
  }

  return { valid: true }
}

```

### 7.9 RegExp 생성 호이스팅

**Impact: LOW-MEDIUM (재생성 방지)**

Render 내부에서 RegExp를 생성하지 마세요. Module scope로 hoisting하거나 `useMemo()`로 memoize하세요.

**Incorrect: 매 render마다 새 RegExp 생성**

```tsx
function Highlighter({ text, query }: Props) {
  const regex = new RegExp(`(${query})`, 'gi')
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}

```

**Correct: memoize 또는 hoisting**

```tsx
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

function Highlighter({ text, query }: Props) {
  const regex = useMemo(
    () => new RegExp(`(${escapeRegex(query)})`, 'gi'),
    [query]
  )
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}

```

**경고: Global regex는 mutable state를 가짐**

```typescript
const regex = /foo/g
regex.test('foo')  // true, lastIndex = 3
regex.test('foo')  // false, lastIndex = 0

```

Global regex(`/g`)는 가변적인 `lastIndex` state를 갖습니다.

### 7.10 Min/Max를 위해 Sort 대신 Loop 사용

**Impact: LOW (O(n log n) 대신 O(n))**

가장 작거나 큰 element를 찾는 데는 array를 한 번만 순회하면 됩니다. Sorting은 낭비이며 더 느립니다.

**Incorrect (O(n log n) - 최신 항목을 찾기 위해 sort):**

```typescript
interface Project {
  id: string
  name: string
  updatedAt: number
}

function getLatestProject(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => b.updatedAt - a.updatedAt)
  return sorted[0]
}

```

Maximum value 하나를 찾기 위해 전체 array를 정렬합니다.

**Incorrect (O(n log n) - 가장 오래된 항목과 최신 항목을 위해 sort):**

```typescript
function getOldestAndNewest(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => a.updatedAt - b.updatedAt)
  return { oldest: sorted[0], newest: sorted[sorted.length - 1] }
}

```

Min/max만 필요한데도 불필요하게 정렬합니다.

**Correct (O(n) - 단일 loop):**

```typescript
function getLatestProject(projects: Project[]) {
  if (projects.length === 0) return null
  
  let latest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt > latest.updatedAt) {
      latest = projects[i]
    }
  }
  
  return latest
}

function getOldestAndNewest(projects: Project[]) {
  if (projects.length === 0) return { oldest: null, newest: null }
  
  let oldest = projects[0]
  let newest = projects[0]
  
  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt < oldest.updatedAt) oldest = projects[i]
    if (projects[i].updatedAt > newest.updatedAt) newest = projects[i]
  }
  
  return { oldest, newest }
}

```

Copy나 sorting 없이 array를 단 한 번 순회합니다.

**대안: 작은 array의 경우 Math.min/Math.max**

```typescript
const numbers = [5, 2, 8, 1, 9]
const min = Math.min(...numbers)
const max = Math.max(...numbers)

```

작은 array에서는 작동하지만, spread operator의 제한으로 인해 매우 큰 array에서는 느려지거나 에러가 발생할 수 있습니다 (Chrome 143 기준 약 124,000개, Safari 18 기준 약 638,000개). 안정성을 위해 loop 방식을 사용하세요.

### 7.11 O(1) Lookup을 위해 Set/Map 사용

**Impact: LOW-MEDIUM (O(n)에서 O(1)로)**

반복적인 멤버십 확인(membership check)을 위해 array를 Set/Map으로 변환하세요.

**Incorrect (확인당 O(n)):**

```typescript
const allowedIds = ['a', 'b', 'c', ...]
items.filter(item => allowedIds.includes(item.id))

```

**Correct (확인당 O(1)):**

```typescript
const allowedIds = new Set(['a', 'b', 'c', ...])
items.filter(item => allowedIds.has(item.id))

```

### 7.12 Immutability를 위해 sort() 대신 toSorted() 사용

**Impact: MEDIUM-HIGH (React state의 mutation bug 방지)**

`.sort()`는 원본 array를 직접 mutate하여 React state 및 props와 관련된 bug를 유발할 수 있습니다. Mutation 없이 새로운 정렬된 array를 생성하려면 `.toSorted()`를 사용하세요.

**Incorrect: 원본 array를 mutate함**

```typescript
function UserList({ users }: { users: User[] }) {
  // users prop array를 직접 mutate함!
  const sorted = useMemo(
    () => users.sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}

```

**Correct: 새 array 생성**

```typescript
function UserList({ users }: { users: User[] }) {
  // 새 정렬 array 생성, 원본은 유지
  const sorted = useMemo(
    () => users.toSorted((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}

```

**React에서 이것이 중요한 이유:**

1. Props/state mutation은 React의 immutability 모델을 깨뜨립니다 - React는 props와 state를 read-only로 간주합니다.
2. Stale closure bug 유발 - Closure(callback, effect) 내부에서 array를 mutate하면 예기치 않은 동작이 발생할 수 있습니다.

**Browser support: 구형 브라우저용 fallback**

```typescript
// 구형 브라우저용 fallback
const sorted = [...items].sort((a, b) => a.value - b.value)

```

`.toSorted()`는 모든 현대 브라우저(Chrome 110+, Safari 16+, Firefox 115+, Node.js 20+)에서 사용 가능합니다. 구형 환경에서는 spread operator를 사용하세요.

**기타 immutable array method:**

* `.toSorted()` - immutable sort
* `.toReversed()` - immutable reverse
* `.toSpliced()` - immutable splice
* `.with()` - immutable element replacement

---

## 8. Advanced Patterns

**Impact: LOW**

신중한 구현이 필요한 특정 케이스를 위한 고급 pattern입니다.

### 8.1 Mount 시가 아닌, App당 한 번 초기화

**Impact: LOW-MEDIUM (개발 중 중복 초기화 방지)**

App 로드당 한 번만 실행되어야 하는 app-wide 초기화 작업을 component의 `useEffect([])` 안에 넣지 마세요. Component는 다시 mount될 수 있고 effect는 재실행됩니다. Module-level guard를 사용하거나 entry module에서 top-level 초기화를 수행하세요.

**Incorrect: dev에서 두 번 실행됨, remount 시 재실행됨**

```tsx
function Comp() {
  useEffect(() => {
    loadFromStorage()
    checkAuthToken()
  }, [])

  // ...
}

```

**Correct: app 로드당 한 번**

```tsx
let didInit = false

function Comp() {
  useEffect(() => {
    if (didInit) return
    didInit = true
    loadFromStorage()
    checkAuthToken()
  }, [])

  // ...
}

```

Reference: [https://react.dev/learn/you-might-not-need-an-effect#initializing-the-application](https://react.dev/learn/you-might-not-need-an-effect#initializing-the-application)

### 8.2 Event Handler를 Ref에 저장

**Impact: LOW (stable subscription)**

Callback이 변경되어도 re-subscribe하면 안 되는 effect에서 사용할 때, callback을 ref에 저장하세요.

**Incorrect: 매 render마다 re-subscribe함**

```tsx
function useWindowEvent(event: string, handler: (e) => void) {
  useEffect(() => {
    window.addEventListener(event, handler)
    return () => window.removeEventListener(event, handler)
  }, [event, handler])
}

```

**Correct: stable subscription**

```tsx
import { useEffectEvent } from 'react'

function useWindowEvent(event: string, handler: (e) => void) {
  const onEvent = useEffectEvent(handler)

  useEffect(() => {
    window.addEventListener(event, onEvent)
    return () => window.removeEventListener(event, onEvent)
  }, [event])
}

```

**대안: 최신 React를 사용 중이라면 `useEffectEvent` 사용:**

`useEffectEvent`는 동일한 pattern에 대해 더 깔끔한 API를 제공합니다. 항상 최신 버전의 handler를 호출하는 stable한 function reference를 생성합니다.

### 8.3 Stable한 Callback Ref를 위한 useEffectEvent

**Impact: LOW (effect 재실행 방지)**

Dependency array에 추가하지 않고도 callback 내부에서 최신 값에 접근하세요. Stale closure를 피하면서 effect 재실행을 방지합니다.

**Incorrect: callback이 바뀔 때마다 effect가 재실행됨**

```tsx
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')

  useEffect(() => {
    const timeout = setTimeout(() => onSearch(query), 300)
    return () => clearTimeout(timeout)
  }, [query, onSearch])
}

```

**Correct: React의 useEffectEvent 사용**

```tsx
import { useEffectEvent } from 'react';

function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')
  const onSearchEvent = useEffectEvent(onSearch)

  useEffect(() => {
    const timeout = setTimeout(() => onSearchEvent(query), 300)
    return () => clearTimeout(timeout)
  }, [query])
}

```

---

## References

1. [https://react.dev](https://react.dev)
2. [https://nextjs.org](https://nextjs.org)
3. [https://swr.vercel.app](https://swr.vercel.app)
4. [https://github.com/shuding/better-all](https://github.com/shuding/better-all)
5. [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)
6. [https://vercel.com/blog/how-we-optimized-package-imports-in-next-js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
7. [https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)