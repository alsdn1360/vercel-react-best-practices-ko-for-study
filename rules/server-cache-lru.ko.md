---
title: Cross-Request LRU Caching
impact: HIGH
impactDescription: caches across requests
tags: server, cache, lru, cross-request
---

## Cross-Request LRU Caching

`React.cache()`는 단일 request 내에서만 작동합니다. 순차적인 request(사용자가 버튼 A를 누른 후 버튼 B를 누르는 경우) 사이에서 공유되는 데이터의 경우, LRU cache를 사용하세요.

**Implementation:**

```typescript
import { LRUCache } from 'lru-cache'

const cache = new LRUCache<string, any>({
  max: 1000, // 캐시에 저장할 최대 항목 수
  ttl: 5 * 60 * 1000  // 캐시 유지 시간 (밀리초), 5분
})

export async function getUser(id: string) {
  const cached = cache.get(id)
  if (cached) return cached

  const user = await db.user.findUnique({ where: { id } })
  cache.set(id, user)
  return user
}

// Request 1: DB 쿼리 실행 → 결과를 캐시에 저장
// Request 2: 캐시에서 바로 반환 → DB 쿼리 실행 안해도 됨
```

순차적인 사용자 action이 몇 초 내에 동일한 데이터를 필요로 하는 여러 endpoint를 hit할 때 사용하세요.

**Vercel의 [Fluid Compute](https://vercel.com/docs/fluid-compute) 사용 시:** 여러 concurrent request가 동일한 function instance와 cache를 공유할 수 있으므로 LRU caching이 특히 효과적입니다. 이는 Redis와 같은 외부 storage 없이도 request 간에 cache가 유지됨을 의미합니다.

**Traditional serverless 환경:** 각 invocation이 격리되어 실행되므로, cross-process caching을 위해 Redis 고려를 권장합니다.

Reference: [https://github.com/isaacs/node-lru-cache](https://github.com/isaacs/node-lru-cache)