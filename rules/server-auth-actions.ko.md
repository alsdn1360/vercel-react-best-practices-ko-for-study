---
title: Authenticate Server Actions Like API Routes
impact: CRITICAL
impactDescription: prevents unauthorized access to server mutations
tags: server, server-actions, authentication, security, authorization
---

## API Routes처럼 Server Actions 인증하기

**Impact: CRITICAL (Server mutations에 대한 무단 access 방지)**

Server Actions(`"use server"`가 포함된 function)는 API routes와 마찬가지로 public endpoints로 노출됩니다. Server Actions은 직접 호출될 수 있으므로, middleware, layout guards, 또는 page-level 체크에만 의존하지 말고 항상 각 Server Action **내부**에서 authentication 및 authorization을 검증하세요.

Next.js 문서는 다음과 같이 명시하고 있습니다: "Server Actions를 public-facing API endpoints와 동일한 보안 고려 사항으로 취급하고, 사용자가 mutation을 수행할 권한이 있는지 확인하세요."

**Incorrect (인증 체크 없음):**

```typescript
'use server'

export async function deleteUser(userId: string) {
  // 누구나 호출 가능! auth 체크 없음
  await db.user.delete({ where: { id: userId } })
  return { success: true }
}

```

**Correct (Action 내부에서 인증 수행):**

```typescript
'use server'

import { verifySession } from '@/lib/auth'
import { unauthorized } from '@/lib/errors'

export async function deleteUser(userId: string) {
  // 항상 action 내부에서 auth를 체크하세요
  const session = await verifySession()
  
  if (!session) {
    throw unauthorized('Must be logged in')
  }
  
  // authorization도 체크하세요
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
  // 먼저 input을 validate 하세요
  const validated = updateProfileSchema.parse(data)
  
  // 그 다음 authenticate 하세요
  const session = await verifySession()
  if (!session) {
    throw new Error('Unauthorized')
  }
  
  // 그 다음 authorize 하세요
  if (session.user.id !== validated.userId) {
    throw new Error('Can only update own profile')
  }
  
  // 마지막으로 mutation을 수행하세요
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