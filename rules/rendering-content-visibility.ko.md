---
title: CSS content-visibility for Long Lists
impact: HIGH
impactDescription: faster initial render
tags: rendering, css, content-visibility, long-lists
---

## 긴 리스트를 위한 CSS content-visibility

off-screen 렌더링을 지연시키려면 `content-visibility: auto`를 적용하세요.

> Lazy Loading은 데이터 요청 자체도 화면에 보이기 전까지 지연시키지만, content-visibility는 데이터 요청을 한 번에 하고, layout/paint 과정만 지연시킴.

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

1,000개의 메시지가 있을 때, 브라우저는 약 990개의 off-screen 아이템에 대한 layout/paint 과정을 건너뜁니다 (초기 render 속도 10배 향상).
