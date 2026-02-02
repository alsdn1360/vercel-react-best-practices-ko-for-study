---
title: Prevent Waterfall Chains in API Routes
impact: CRITICAL
impactDescription: 2-10× improvement
tags: api-routes, server-actions, waterfalls, parallelization
---

## API Routes의 Waterfall Chains 방지

API routes와 Server Actions에서는 독립적인 작업들을 나중에 await 하더라도 즉시 시작(start)시키세요.

**Incorrect (config는 auth를 기다리고, data는 둘 다 기다림):**

```typescript
export async function GET(request: Request) {
  const session = await auth()
  const config = await fetchConfig()
  const data = await fetchData(session.user.id)
  return Response.json({ data, config })
}
```

**Correct (auth와 config가 즉시 시작됨):**

```typescript
export async function GET(request: Request) {
  const sessionPromise = auth() // auth() 즉시 실행(백그라운드)
  const configPromise = fetchConfig() // fetchConfig() 즉시 실행(백그라운드)

  const session = await sessionPromise // auth()가 이미 완료됐으면 즉시 값을 받고, 아직 진행 중이면 기다림

  const [config, data] = await Promise.all([
    configPromise, // 이미 진행 중
    fetchData(session.user.id) // config 안기다리고, 병렬로 같이 실행
  ])

  return Response.json({ data, config })
}
```

더 복잡한 의존성 체인(dependency chains)이 있는 작업의 경우, `better-all`을 사용하여 parallelism을 자동으로 극대화하세요 (Dependency-Based Parallelization 참고).