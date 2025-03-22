# 重定向

在 Next.js 中处理重定向有几种方法。本文将介绍每种可用选项、使用案例以及如何管理大量重定向。

<table><thead><tr><th width="223.92578125">API</th><th>目的</th><th>用于哪里</th><th>状态码</th></tr></thead><tbody><tr><td><a href="https://nextjs.org/docs/app/building-your-application/routing/redirecting#redirect-function"><code>redirect</code></a></td><td>在变更或事件后重定向用户</td><td>服务器组件、服务器操作、路由处理程序</td><td>307（临时）或 303（服务器操作）</td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/routing/redirecting#permanentredirect-function"><code>permanentRedirect</code></a></td><td>在变更或事件后重定向用户</td><td>服务器组件、服务器操作、路由处理程序</td><td>308（永久）</td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/routing/redirecting#userouter-hook"><code>useRouter</code></a></td><td>执行客户端导航</td><td>客户端组件中的事件处理程序</td><td>不适用</td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/routing/redirecting#redirects-in-nextconfigjs"><code>next.config.js</code> 中的redirects</a></td><td>根据路径重定向传入请求</td><td><code>next.config.js</code> 文件</td><td>307（临时）或 308（永久）</td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/routing/redirecting#nextresponseredirect-in-middleware"><code>NextResponse.redirect</code></a></td><td>根据条件重定向传入请求</td><td>中间件</td><td>任何</td></tr></tbody></table>

### redirect 函数

`redirect` 函数允许您将用户重定向到另一个 URL。您可以在服务器组件、路由处理程序和服务器操作中调用 `redirect` 。

`redirect` 通常在变更或事件后使用。例如，创建一个帖子：

```tsx
'use server'
 
import { redirect } from 'next/navigation'
import { revalidatePath } from 'next/cache'
 
export async function createPost(id: string) {
  try {
    // 调用数据库
  } catch (error) {
    // 处理错误
  }
 
  revalidatePath('/posts') // 更新缓存的帖子
  redirect(`/post/${id}`) // 导航到post页面
}
```

> 需要知道的是：
>
> * `redirect` 默认返回一个 307（临时重定向）状态码。当在服务器操作中使用时，它返回一个 303（查看其他），通常用于在 POST 请求的结果中重定向到成功页面。
> * `redirect` 内部抛出错误，因此应在 `try/catch` 块外调用。
> * `redirect` 可以在客户端组件的渲染过程中调用，但不能在事件处理程序中使用。您可以改用 `useRouter` 钩子。
> * `redirect` 也接受绝对 URL，并可用于重定向到外部链接。
> * 如果您希望在渲染过程之前进行重定向，请使用 `next.config.js` 或中间件。

有关更多信息，请参阅 `redirect` API 参考。

### `permanentRedirect` 函数

`permanentRedirect` 函数允许您永久性地将用户重定向到另一个 URL。您可以在服务器组件、路由处理程序和服务器操作中调用 `permanentRedirect` 。

`permanentRedirect` 通常在变更或事件后使用，这些变更会更改实体的规范 URL，例如在用户更改用户名后更新其个人资料 URL：

```tsx
'use server'
 
import { permanentRedirect } from 'next/navigation'
import { revalidateTag } from 'next/cache'
 
export async function updateUsername(username: string, formData: FormData) {
  try {
    // 调用数据库
  } catch (error) {
    // 处理错误
  }
 
  revalidateTag('username') // 更新所有对用户名的引用
  permanentRedirect(`/profile/${username}`) // 导航到新用户配置文件
}
```

> 需要知道的是：
>
> * `permanentRedirect` 默认返回一个 308 （永久重定向）状态码。
> * `permanentRedirect` 也接受绝对 URL，并可用于重定向到外部链接。
> * 如果您希望在渲染过程之前进行重定向，请使用 `next.config.js` 或中间件。

有关更多信息，请参阅 <mark style="color:blue;">`permanentRedirect`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">API</mark> 参考。

### `useRouter`钩子

如果您需要在客户端组件的事件处理程序内部进行重定向，可以使用 `useRouter` 钩子的 `push` 方法。例如：

```tsx
'use client'
 
import { useRouter } from 'next/navigation'
 
export default function Page() {
  const router = useRouter()
 
  return (
    <button type="button" onClick={() => router.push('/dashboard')}>
      Dashboard
    </button>
  )
}
```

> 需要知道的是：
>
> * 如果您不需要以编程方式导航用户，您应使用 `<Link>` 组件。

有关更多信息，请参阅 `useRouter` API 参考。

### `next.config.js` 文件中 `redirects`

`next.config.js` 文件中的 `redirects` 选项允许您将传入请求路径重定向到不同的目标路径。这在您更改页面的 URL 结构或提前知道重定向列表时非常有用。

`redirects` 支持路径、头部、 cookie 和查询匹配，使您能够根据传入请求灵活地重定向用户。

要使用 `redirects` ，请将选项添加到您的 `next.config.js` 文件中：

```typescript
import type { NextConfig } from 'next'
 
const nextConfig: NextConfig = {
  async redirects() {
    return [
      // 基本重定向
      {
        source: '/about',
        destination: '/',
        permanent: true,
      },
      // 通配符路径匹配
      {
        source: '/blog/:slug',
        destination: '/news/:slug',
        permanent: true,
      },
    ]
  },
}
 
export default nextConfigtype
```

有关更多信息，请参阅 `redirects` API 参考。

> 需要知道的是：
>
> * `redirects` 可以返回 307（临时重定向）或 308（永久重定向）状态码，带有 `permanent` 选项。
> * `redirects` 可能在平台上有一个限制。例如，在 Vercel 上，重定向的限制为 1,024。要管理大量重定向（1000+），请考虑使用中间件创建自定义解决方案。有关更多信息，请参见大规模管理重定向。
> * `redirects` 在中间件之前运行。

### 中间件里的 `NextResponse.redirect`

中间件允许您在请求完成之前运行代码。然后，根据传入的请求，使用 `NextResponse.redirect` 重定向到不同的 URL。这在您想根据条件（例如身份验证、会话管理等）重定向用户或拥有大量重定向时非常有用。

例如，如果用户未经过身份验证，则将其重定向到 `/login` 页面：

{% code title="middleware.ts" %}
```typescript
import { NextResponse, NextRequest } from 'next/server'
import { authenticate } from 'auth-provider'
 
export function middleware(request: NextRequest) {
  const isAuthenticated = authenticate(request)
 
  // 如果用户已认证，正常继续
  if (isAuthenticated) {
    return NextResponse.next()
  }
 
  // 如果未认证，则重定向到登录页面
  return NextResponse.redirect(new URL('/login', request.url))
}
 
export const config = {
  matcher: '/dashboard/:path*',
}
```
{% endcode %}

> 需要知道的是：
>
> * 中间件在 `next.config.js` 的 `redirects` 之后运行，并在渲染之前运行。

请参阅中间件文档以获取更多信息。

### 大规模重定向（高级）

要管理大量重定向（1000+）[^1]，您可以考虑使用中间件创建自定义解决方案。这使您能够以编程方式处理重定向，而无需重新部署您的应用程序。

要做到这一点，您需要考虑：

1. 创建和存储重定向映射。
2. 优化数据查找性能。

> Next.js 示例：请参见我们的中间件与 Bloom 过滤器示例，以实现以下建议。

#### 1.创建和存储重定向映射

重定向映射是您可以存储在数据库（通常是键值存储）或 JSON 文件中的重定向列表。

考虑以下数据结构：

```
{
  "/old": {
    "destination": "/new",
    "permanent": true
  },
  "/blog/post-old": {
    "destination": "/blog/post-new",
    "permanent": true
  }
}
```

在中间件中，您可以从数据库中读取数据，例如 Vercel 的 <mark style="color:blue;">Edge Config</mark> 或 <mark style="color:blue;">Redis</mark>，并根据传入的请求重定向用户：

{% code title="middleware.ts" %}
```typescript
import { NextResponse, NextRequest } from 'next/server'
import { get } from '@vercel/edge-config'
 
type RedirectEntry = {
  destination: string
  permanent: boolean
}
 
export async function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname
  const redirectData = await get(pathname)
 
  if (redirectData && typeof redirectData === 'string') {
    const redirectEntry: RedirectEntry = JSON.parse(redirectData)
    const statusCode = redirectEntry.permanent ? 308 : 307
    return NextResponse.redirect(redirectEntry.destination, statusCode)
  }
 
  // 未找到重定向，继续请求而不重定向
  return NextResponse.next()
}
```
{% endcode %}

#### 2.优化数据查找性能

对于每个传入请求读取大型数据集可能会很慢且昂贵。您可以通过两种方式优化数据查找性能：

* 使用一个针对快速读取进行优化的数据库，例如 Vercel Edge Config 或 Redis。
* 使用数据查找策略，如[<mark style="color:blue;">布隆过滤器</mark>](#user-content-fn-2)[^2]，来高效地检查重定向是否存在，而无需读取更大的重定向文件或数据库。

考虑到前面的例子，您可以将生成的布隆过滤器文件导入中间件，然后检查传入请求路径名是否存在于布隆过滤器中。

如果是这样，将请求转发给路由处理程序，该程序将检查实际文件并将用户重定向到适当的 URL。这避免了将大型重定向文件导入中间件，这可能会减慢每个传入请求的速度。

{% code title="middleware.ts" %}
```typescript
import { NextResponse, NextRequest } from 'next/server'
import { ScalableBloomFilter } from 'bloom-filters'
import GeneratedBloomFilter from './redirects/bloom-filter.json'
 
type RedirectEntry = {
  destination: string
  permanent: boolean
}
 
// 从生成的 JSON 文件初始化布隆过滤器
const bloomFilter = ScalableBloomFilter.fromJSON(GeneratedBloomFilter as any)
 
export async function middleware(request: NextRequest) {
  // 获取传入请求的路径
  const pathname = request.nextUrl.pathname
 
  // 检查路径是否在布隆过滤器中
  if (bloomFilter.has(pathname)) {
    // 将路径名转发给路由处理程序
    const api = new URL(
      `/api/redirects?pathname=${encodeURIComponent(request.nextUrl.pathname)}`,
      request.nextUrl.origin
    )
 
    try {
      // 从路由处理程序获取重定向数据
      const redirectData = await fetch(api)
 
      if (redirectData.ok) {
        const redirectEntry: RedirectEntry | undefined =
          await redirectData.json()
 
        if (redirectEntry) {
          // 确定状态码
          const statusCode = redirectEntry.permanent ? 308 : 307
 
          // 重定向到目标
          return NextResponse.redirect(redirectEntry.destination, statusCode)
        }
      }
    } catch (error) {
      console.error(error)
    }
  }
 
  // 未找到重定向，继续请求而不重定向
  return NextResponse.next()
}
```
{% endcode %}

然后，在路由处理程序中：

{% code title="app/api/redirects/route.ts" %}
```typescript
import { NextRequest, NextResponse } from 'next/server'
import redirects from '@/app/redirects/redirects.json'
 
type RedirectEntry = {
  destination: string
  permanent: boolean
}
 
export function GET(request: NextRequest) {
  const pathname = request.nextUrl.searchParams.get('pathname')
  if (!pathname) {
    return new Response('Bad Request', { status: 400 })
  }
 
  // 从 redirects.json 文件中获取重定向条目
  const redirect = (redirects as Record<string, RedirectEntry>)[pathname]
 
  // 考虑 Bloom 过滤器的假阳性
  if (!redirect) {
    return new Response('No redirect', { status: 400 })
  }
 
  // 返回重定向条目
  return NextResponse.json(redirect)
}
```
{% endcode %}

> 需要知道的是：
>
> * 要生成布隆过滤器，您可以使用像 `bloom-filters` 这样的库。
> * 您应验证发送到路由处理程序的请求，以防止恶意请求。

### 下一步



[^1]: 旧的重定向缓存可能导致无限重定向

[^2]: 布隆过滤器是一种**概率型数据结构**，用于判断一个元素是否在一个集合中。它比普通的哈希表更省内存，但会有**误判**的可能（即可能会错误地认为某个元素存在）。
