---
title: Minimize Serialization at RSC Boundaries
impact: HIGH
impactDescription: reduces data transfer size
tags: server, rsc, serialization, props
---

## RSC Boundaries에서의 Serialization 최소화

React Server/Client boundary는 모든 object properties를 string으로 serialize하여 HTML response와 후속 RSC requests에 포함시킵니다. 이 serialized data는 page weight와 load time에 직접적인 영향을 미치므로 **size가 매우 중요합니다**. Client가 실제로 사용하는 field만 전달하세요.

**Incorrect (50개 field 모두 serialize됨):**

```tsx
async function Page() {
  const user = await fetchUser()  // 50 fields
  return <Profile user={user} />
}

'use client'
function Profile({ user }: { user: User }) {
  return <div>{user.name}</div>  // 1개 field 사용
}
```

**Correct (1개 field만 serialize됨):**

```tsx
async function Page() {
  const user = await fetchUser()
  return <Profile name={user.name} /> // 필요한 필드에 대해서만 Props로 전달
}

'use client'
function Profile({ name }: { name: string }) {
  return <div>{name}</div>
}
```