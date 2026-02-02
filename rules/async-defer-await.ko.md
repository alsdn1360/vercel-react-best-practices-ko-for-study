---
title: Defer Await Until Needed
impact: HIGH
impactDescription: avoids blocking unused code paths
tags: async, await, conditional, optimization
---

## 필요한 시점까지 Await 지연시키기

`await` 연산을 실제로 사용되는 branch 내부로 이동시켜, 해당 연산이 필요 없는 code path가 차단(blocking)되는 것을 방지하세요.

**Incorrect (두 branch 모두 block됨):**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId)
  
  if (skipProcessing) {
    // 즉시 return하지만 여전히 userData를 기다림
    return { skipped: true }
  }
  
  // 이 branch만 userData를 사용함
  return processUserData(userData)
}
```

**Correct (필요할 때만 block됨):**

```typescript
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // 기다리지 않고 즉시 return
    return { skipped: true }
  }
  
  // 필요한 경우에만 fetch
  const userData = await fetchUserData(userId)
  return processUserData(userData)
}
```

**또 다른 예시 (Early return optimization):**

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

// Correct: 필요한 시점에만 fetch함
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

이러한 optimization은 skip되는 branch가 자주 실행되거나, 지연된 연산의 비용이 클 때 특히 유용합니다.