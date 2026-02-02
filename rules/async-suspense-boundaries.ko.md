---
title: Strategic Suspense Boundaries
impact: HIGH
impactDescription: faster initial paint
tags: async, suspense, streaming, layout-shift
---

## 전략적 Suspense Boundaries

JSX를 return하기 전 async component에서 data를 await하는 대신, Suspense boundary를 사용하여 data가 로드되는 동안 wrapper UI를 더 빠르게 보여주세요.

**Incorrect (data fetching에 의해 wrapper가 차단됨):**

```tsx
async function Page() {
  const data = await fetchData() // 전체 페이지를 block함
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  )
}
```

중간 섹션만 data가 필요함에도 불구하고 전체 layout이 data를 기다립니다.

**Correct (wrapper는 즉시 표시되고, data는 streaming됨):**

```tsx
function Page() {
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <Suspense fallback={<Skeleton />}>
          <DataDisplay />
        </Suspense>
      </div>
      <div>Footer</div>
    </div>
  )
}

async function DataDisplay() {
  const data = await fetchData() // 이 component만 block함
  return <div>{data.content}</div>
}
```

Sidebar, Header, Footer는 즉시 render됩니다. 오직 DataDisplay만 data를 기다립니다.

**대안 (component 간 promise 공유):**

```tsx
function Page() {
  // 즉시 fetch를 시작하되, await하지 않음
  const dataPromise = fetchData()
  
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <Suspense fallback={<Skeleton />}>
        <DataDisplay dataPromise={dataPromise} />
        <DataSummary dataPromise={dataPromise} />
      </Suspense>
      <div>Footer</div>
    </div>
  )
}

function DataDisplay({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // promise를 unwrap함
  return <div>{data.content}</div>
}

function DataSummary({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // 동일한 promise를 재사용함
  return <div>{data.summary}</div>
}
```

두 component가 동일한 promise를 공유하므로 한 번의 fetch만 발생합니다. Layout은 즉시 render되며 두 component는 함께 기다립니다.

**이 패턴을 사용하지 말아야 할 때:**

* Layout 결정에 필요한 critical data (위치에 영향을 주는 경우)
* SEO에 중요한 above the fold 콘텐츠
* Suspense overhead의 가치가 없는 작고 빠른 query
* Layout shift를 피하고 싶을 때 (loading → content 점프)

**Trade-off:** 더 빠른 initial paint vs 잠재적인 layout shift. UX 우선순위에 따라 선택하세요.

* Suspense의 가장 큰 단점은 CLS(Cumulative Layout Shift)가 증가하는 것입니다. Fallback UI에서 실제 컨텐츠로 전환될 때, 높이나 너비가 달라지면 페이지의 다른 요소가 밀려나며 layout shift가 발생합니다.

**Layout Shift를 피해야 하는 상황(Suspense를 사용하면 안좋은 상황):**

* E-commerce checkout flow (버튼 위치가 중요)
* Form 제출 화면 (사용자가 입력 중)
* Landing page의 CTA(Call-to-Action) 버튼 근처
* 광고나 promotional banner 영역

**Suspense를 사용해도 되는 상황:**

* Page 하단의 "더보기" 콘텐츠
* Modal이나 drawer 내부
* Infinite scroll로 추가되는 항목들
* 사용자 action 이후의 결과 (검색, 필터링 등)