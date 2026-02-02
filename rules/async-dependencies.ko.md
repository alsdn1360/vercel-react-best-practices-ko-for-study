---
title: Dependency-Based Parallelization
impact: CRITICAL
impactDescription: 2-10× improvement
tags: async, parallelization, dependencies, better-all
---

## Dependency-Based Parallelization

부분적인 의존성(partial dependencies)이 있는 작업의 경우, `better-all`을 사용하여 parallelism을 극대화하세요. 각 task를 가능한 가장 이른 시점에 자동으로 시작해 줍니다.

**Incorrect (profile이 불필요하게 config를 기다림):**

```typescript
const [user, config] = await Promise.all([
  fetchUser(),
  fetchConfig()
])
const profile = await fetchProfile(user.id)

```

**Correct (config와 profile이 병렬로 실행됨):**

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

**추가 dependency가 없는 대안:**

모든 promise를 먼저 생성한 뒤, 마지막에 `Promise.all()`을 수행할 수도 있습니다.

```typescript
const userPromise = fetchUser()
const profilePromise = userPromise.then(user => fetchProfile(user.id))

const [user, config, profile] = await Promise.all([
  userPromise,
  fetchConfig(),
  profilePromise
])

```

Reference: [https://github.com/shuding/better-all](https://github.com/shuding/better-all)