---
title: Promise.all() for Independent Operations
impact: CRITICAL
impactDescription: 2-10× improvement
tags: async, parallelization, promises, waterfalls
---

## 독립적인 작업을 위한 Promise.all()

비동기 작업(async operations) 사이에 상호 의존성이 없는 경우, `Promise.all()`을 사용하여 이를 동시에(concurrently) 실행하세요.

**Incorrect (순차적 실행, 3번의 round trips 발생):**

```typescript
const user = await fetchUser()
const posts = await fetchPosts()
const comments = await fetchComments()

```

**Correct (병렬 실행, 1번의 round trip 발생):**

```typescript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])

```