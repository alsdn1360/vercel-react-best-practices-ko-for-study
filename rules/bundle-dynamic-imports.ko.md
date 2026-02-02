---
title: Dynamic Imports for Heavy Components
impact: CRITICAL
impactDescription: directly affects TTI and LCP
tags: bundle, dynamic-import, code-splitting, next-dynamic
---

## Heavy Components를 위한 Dynamic Imports

초기 render 시점에 필요하지 않은 대규모 component들을 lazy-load하기 위해 `next/dynamic`을 사용하세요.

**Incorrect (Monaco가 main chunk와 함께 번들링됨 ~300KB):**

```tsx
import { MonacoEditor } from './monaco-editor'

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}

```

**Correct (Monaco를 필요할 때 load):**

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