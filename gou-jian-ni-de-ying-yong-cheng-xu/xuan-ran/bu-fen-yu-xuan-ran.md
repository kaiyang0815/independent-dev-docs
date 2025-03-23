# 部分预渲染

> **注意**：部分预渲染是一个仅在 canary 上可用的实验性功能，可能会发生变化。它尚未准备好用于生产环境。

部分预渲染 (PPR) 使您能够在同一路由中结合静态和动态组件。

在构建期间，Next.js 尽可能多地预渲染该路由。如果检测到[动态](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-rendering)代码，例如从传入请求中读取，您可以用 [React Suspense](https://react.dev/reference/react/Suspense) 边界包装相关组件。然后，Suspense 边界的回退将包含在预渲染的 HTML 中。

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

> **🎥观看**：为什么 PPR 以及它是如何工作的 → YouTube (10 分钟)。

### 背景

PPR 使您的 Next.js 服务器能够立即发送预渲染的内容。

为了防止客户端到服务器的瀑布流，动态组件开始从服务器并行流式传输，同时提供初始预渲染。这确保了动态组件可以在客户端 JavaScript 加载到浏览器之前开始渲染。

为了防止为每个动态组件创建多个 HTTP 请求，PPR 能够将静态预渲染和动态组件组合成一个单一的 HTTP 请求。这确保了每个动态组件不需要多个网络往返。

### 使用部分预渲染

#### 渐进采用（版本 15 Canary 版本）

在 Next.js 15 Canary 版本中，PPR 作为实验性功能可用。它尚未在稳定版本中提供。要安装：

```bash
npm install next@canary
```

您可以通过将 `ppr` 选项设置为 `incremental` 在 `next.config.js` 中逐步采用部分预渲染，并在文件顶部导出 `experimental_ppr` 路由配置选项：

{% code title="next.config.ts" %}
```typescript
import type { NextConfig } from 'next'
 
const nextConfig: NextConfig = {
  experimental: {
    ppr: 'incremental',
  },
}
 
export default nextConfig
```
{% endcode %}

{% code title="app/page.tsx" %}
```tsx
import { Suspense } from 'react'
import { StaticComponent, DynamicComponent, Fallback } from '@/app/ui'
 
export const experimental_ppr = true
 
export default function Page() {
  return (
    <>
      <StaticComponent />
      <Suspense fallback={<Fallback />}>
        <DynamicComponent />
      </Suspense>
    </>
  )
}
```
{% endcode %}

> 注意：
>
> * 没有 `experimental_ppr` 的路由将默认为 `false` ，并且不会使用 PPR 进行预渲染。您需要为每个路由显式选择 PPR。
> * `experimental_ppr` 将适用于路由段的所有子项，包括嵌套的布局和页面。您不必将其添加到每个文件中，只需添加到路由的顶层段即可。
> * 要禁用子段的 PPR，您可以在子段中将 `experimental_ppr` 设置为 `false`。

### 动态组件

在 `next build` 期间为您的路由创建预渲染时，Next.js 要求动态 API 被 React Suspense 包装。然后 `fallback` 被包含在预渲染中。

例如，使用像 `cookies` 或 `headers` 这样的函数：

{% code title="app/user.tsx" %}
```tsx
import { cookies } from 'next/headers'
 
export async function User() {
  const session = (await cookies()).get('session')?.value
  return '...'
}
```
{% endcode %}

该组件需要查看传入请求以读取 cookies。要将其与 PPR 一起使用，您应该用 Suspense 包裹该组件：

{% code title="app/page.tsx" %}
```typescript
import { Suspense } from 'react'
import { User, AvatarSkeleton } from './user'
 
export const experimental_ppr = true
 
export default function Page() {
  return (
    <section>
      <h1>This will be prerendered</h1>
      <Suspense fallback={<AvatarSkeleton />}>
        <User />
      </Suspense>
    </section>
  )
}
```
{% endcode %}

组件仅在访问值时选择动态渲染。

例如，如果您从 `page` 读取 `searchParams` ，您可以将此值作为 prop 转发到另一个组件：

{% code title="app/page.tsx" %}
```typescript
import { Table } from './table'
 
export default function Page({
  searchParams,
}: {
  searchParams: Promise<{ sort: string }>
}) {
  return (
    <section>
      <h1>This will be prerendered</h1>
      <Table searchParams={searchParams} />
    </section>
  )
}
```
{% endcode %}

在表格组件内部，从 `searchParams` 访问值将使组件动态运行：

{% code title="app/table.tsx" %}
```typescript
export async function Table({
  searchParams,
}: {
  searchParams: Promise<{ sort: string }>
}) {
  const sort = (await searchParams).sort === 'true'
  return '...'
}
```
{% endcode %}

