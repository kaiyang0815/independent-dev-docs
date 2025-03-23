# 增量静态再生（ISR）

<details>

<summary> 示例</summary>

* [Next.js ](https://vercel.com/templates/next.js/nextjs-commerce)
* [按需 ISR](https://on-demand-isr.vercel.app/)
* [Next.js 表单](https://github.com/vercel/next.js/tree/canary/examples/next-forms)

</details>

增量静态再生 (ISR) 使您能够：

* 在不重建整个站点的情况下更新静态内容
* 通过为大多数请求提供预渲染的静态页面来减少服务器负载
* 确保适当的 `cache-control` 头部自动添加到页面中
* 处理大量内容页面而不出现长时间 `next build`

这是一个最小示例：

{% code title="app/blog/[id]/page.tsx" %}
```tsx
interface Post {
  id: string
  title: string
  content: string
}
 
// 当请求到达时，Next.js 将使缓存失效
// 最多每 60 秒一次。
export const revalidate = 60
 
// 我们将在构建时仅预渲染 `generateStaticParams` 中的参数。
// 如果有请求到达尚未生成的路径，
// Next.js 将按需服务器渲染该页面。
export const dynamicParams = true // 或 false，在未知路径上返回 404
 
export async function generateStaticParams() {
  const posts: Post[] = await fetch('https://api.vercel.app/blog').then((res) =>
    res.json()
  )
  return posts.map((post) => ({
    id: String(post.id),
  }))
}
 
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const post: Post = await fetch(`https://api.vercel.app/blog/${id}`).then(
    (res) => res.json()
  )
  return (
    <main>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </main>
  )
}
```
{% endcode %}

这是该示例的工作原理：

* 在 `next build` 期间，生成所有已知的博客文章（此示例中有 25 篇）
* 对这些页面（例如 `/blog/1` ）的所有请求都是缓存的，并且是瞬时的
* 经过 60 秒后，下一个请求仍将显示缓存的（过时的）页面
* 缓存被失效，页面的新版本开始在后台生成
* 一旦成功生成，Next.js 将显示并缓存更新后的页面
* 如果请求 `/blog/26` ，Next.js 将按需生成并缓存此页面。

### 参考

#### 路由段配置

* [`revalidate`](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config#revalidate)
* [`dynamicParams`](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config#dynamicparams)

函数

* [`revalidatePath`](https://nextjs.org/docs/app/api-reference/functions/revalidatePath)
* [`revalidateTag`](https://nextjs.org/docs/app/api-reference/functions/revalidateTag)

### 示例

#### 基于时间的重新验证

这会在 `/blog` 上获取并显示博客文章列表。一个小时后，下一次访问该页面时，该页面的缓存将失效。然后，在后台，生成一个包含最新博客文章的新版本页面。

{% code title="app/blog/page.tsx" %}
```tsx
interface Post {
  id: string
  title: string
  content: string
}
 
export const revalidate = 3600 // invalidate every hour
 
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts: Post[] = await data.json()
  return (
    <main>
      <h1>Blog Posts</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </main>
  )
}
```
{% endcode %}

我们建议设置较高的重新验证时间。例如，1 小时而不是 1 秒。如果您需要更高的精度，请考虑使用按需重新验证。如果您需要实时数据，请考虑切换到[动态渲染](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-rendering)。

#### 按需重新验证 `revalidatePath`

对于更精确的重新验证方法，可以使用 `revalidatePath` 函数按需使页面失效。

例如，这个服务器操作将在添加新帖子后被调用。无论您在服务器组件中如何检索数据，无论是使用 `fetch` 还是连接到数据库，这将清除整个路由的缓存，并允许服务器组件获取新数据。

{% code title="app/actions.ts" %}
```typescript
'use server'
 
import { revalidatePath } from 'next/cache'
 
export async function createPost() {
  // Invalidate the /posts route in the cache
  revalidatePath('/posts')
}
```
{% endcode %}

[查看演示](https://on-demand-isr.vercel.app/)并[探索源代码](https://github.com/vercel/on-demand-isr)。

#### 按需重新验证`revalidateTag`

&#x20;对于大多数用例，建议重新验证整个路径。如果您需要更细粒度的控制，可以使用 `revalidateTag` 函数。例如，您可以标记单个 `fetch` 调用：

{% code title="app/blog/page.tsx" %}
```typescript
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog', {
    next: { tags: ['posts'] },
  })
  const posts = await data.json()
  // ...
}
```
{% endcode %}

如果您正在使用 ORM 或连接到数据库，您可以使用 `unstable_cache` ：

{% code title="app/blog/page.tsx" %}
```tsx
import { unstable_cache } from 'next/cache'
import { db, posts } from '@/lib/db'
 
const getCachedPosts = unstable_cache(
  async () => {
    return await db.select().from(posts)
  },
  ['posts'],
  { revalidate: 3600, tags: ['posts'] }
)
 
export default async function Page() {
  const posts = getCachedPosts()
  // ...
}
```
{% endcode %}

您可以在[服务器操作](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)或[路由处理程序](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)中使用 `revalidateTag`：

{% code title="app/actions.ts" %}
```typescript
'use server'
 
import { revalidateTag } from 'next/cache'
 
export async function createPost() {
  // 使缓存中所有标记为 'posts' 的数据失效
  revalidateTag('posts')
}
```
{% endcode %}

#### 处理未捕获的异常

如果在尝试重新验证数据时抛出错误，最后成功生成的数据将继续从缓存中提供服务。在下一个后续请求中，Next.js 将重试重新验证数据。[了解更多关于错误处理的信息](https://nextjs.org/docs/app/building-your-application/routing/error-handling)。

#### 自定义缓存位置

缓存和重新验证页面（使用增量静态再生）使用相同的共享缓存。当部署到 Vercel 时，ISR 缓存会自动持久化到耐用存储。

在自托管时，ISR 缓存存储在您的 Next.js 服务器的文件系统（磁盘）上。当使用页面路由和应用路由进行自托管时，这会自动生效。

您可以配置 Next.js 缓存位置，如果您想将缓存的页面和数据持久化到耐用存储，或在多个容器或实例之间共享缓存。[了解更多](https://nextjs.org/docs/app/building-your-application/deploying#caching-and-isr)。

### 故障排查

#### 在本地开发中调试缓存数据

如果您正在使用 `fetch` API，您可以添加额外的日志记录来了解哪些请求被缓存或未被缓存。[了解有关 `logging` 选项的更多信息](https://nextjs.org/docs/app/api-reference/config/next-config-js/logging)。

{% code title="next.config.js" %}
```javascript
module.exports = {
  logging: {
    fetches: {
      fullUrl: true,
    },
  },
}
```
{% endcode %}

#### 验证正确的生产行为

要验证您的页面在生产环境中被正确缓存和重新验证，您可以通过运行 `next build` 然后 `next start` 来在本地测试以运行生产的 Next.js 服务器。

这将允许您测试 ISR 行为，就像它在生产环境中工作一样。为了进一步调试，请将以下环境变量添加到您的 `.env` 文件中：

{% code title=".env" %}
```bash
NEXT_PRIVATE_DEBUG_CACHE=1
```
{% endcode %}

这将使 Next.js 服务器控制台记录 ISR 缓存命中和未命中情况。您可以检查输出以查看在 `next build` 期间生成了哪些页面，以及当路径按需访问时页面是如何更新的。

### 注意事项

* ISR 仅在使用 Node.js 运行时（默认）时受支持。
* 创建静态导出时不支持 ISR。
* 如果在静态渲染的路由中有多个 `fetch` 请求，并且每个请求的 `revalidate` 频率不同，则将使用最低的时间进行 ISR。然而，这些重新验证频率仍将被数据缓存所尊重。
* 如果在某条路由上使用的任何 `fetch` 请求具有 `revalidate` 时间为 `0` ，或显式 `no-store` ，则该路由将动态渲染。
* 中间件不会在按需 ISR 请求中执行，这意味着中间件中的任何路径重写或逻辑将不会被应用。确保您正在重新验证确切的路径。例如， `/post/1` 而不是重写的 `/post-1` 。

### 版本历史

| 版本        | 更改                       |
| --------- | ------------------------ |
| `v14.1.0` | 自定义 `cacheHandler` 是稳定的。 |
| `v13.0.0` | 引入了应用路由器。                |
| `v12.2.0` | 页面路由器：按需 ISR 是稳定的        |
| `v12.0.0` | 页面路由器：添加了机器人工察知 ISR 回退。  |
| `v9.5.0`  | 页面路由器：引入了稳定的 ISR。        |
