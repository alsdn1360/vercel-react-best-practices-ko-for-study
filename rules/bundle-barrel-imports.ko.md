---
title: Avoid Barrel File Imports
impact: CRITICAL
impactDescription: 200-800ms import cost, slow builds
tags: bundle, imports, tree-shaking, barrel-files, performance
---

## Barrel File Import 피하기

수천 개의 사용되지 않는 module을 로딩하는 것을 피하기 위해 barrel file 대신 source file에서 직접 import하세요. **Barrel file**은 여러 module을 다시 export하는 entry point입니다 (예: `export * from './module'`을 수행하는 `index.js`).

인기 있는 icon 및 component library는 entry file에 **최대 10,000개의 re-export**를 가질 수 있습니다. 많은 React package의 경우, **import하는 데만 200-800ms가 소요**되며, 이는 개발 속도와 production cold start 모두에 영향을 미칩니다.

**Tree-shaking이 도움이 되지 않는 이유:** library가 external로 표시되면 (bundle되지 않음), bundler가 이를 최적화할 수 없습니다. Tree-shaking을 활성화하기 위해 bundle하면 전체 module graph를 분석하느라 build가 상당히 느려집니다.

**Incorrect (전체 library를 import함):**

```tsx
import { Check, X, Menu } from 'lucide-react'
// 1,583개의 module을 로드하며, dev 환경에서 약 2.8s 추가 소요
// Runtime 비용: 모든 cold start마다 200-800ms

import { Button, TextField } from '@mui/material'
// 2,225개의 module을 로드하며, dev 환경에서 약 4.2s 추가 소요
```

**Correct (필요한 것만 import함):**

```tsx
import Check from 'lucide-react/dist/esm/icons/check'
import X from 'lucide-react/dist/esm/icons/x'
import Menu from 'lucide-react/dist/esm/icons/menu'
// 3개의 module만 로드 (~2KB vs ~1MB)

import Button from '@mui/material/Button'
import TextField from '@mui/material/TextField'
// 사용하는 것만 로드
```

**Alternative (Next.js 13.5+):**

```js
// next.config.js - optimizePackageImports 사용
module.exports = {
  experimental: {
    optimizePackageImports: ['lucide-react', '@mui/material']
  }
}

// 그러면 편의성 좋은 barrel import를 유지할 수 있습니다:
import { Check, X, Menu } from 'lucide-react'
// Build 시점에 자동으로 direct import로 변환됨
```

Direct import는 15-70% 더 빠른 dev boot, 28% 더 빠른 build, 40% 더 빠른 cold start, 그리고 현저히 빠른 HMR을 제공합니다.

일반적으로 영향을 받는 library들: `lucide-react`, `@mui/material`, `@mui/icons-material`, `@tabler/icons-react`, `react-icons`, `@headlessui/react`, `@radix-ui/react-*`, `lodash`, `ramda`, `date-fns`, `rxjs`, `react-use`.

참조: [How we optimized package imports in Next.js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)