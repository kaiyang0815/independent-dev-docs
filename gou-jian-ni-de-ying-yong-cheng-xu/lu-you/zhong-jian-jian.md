# 中间件

中间件允许您在请求完成之前运行代码。然后，根据传入的请求，您可以通过重写、重定向、修改请求或响应头，或直接响应来修改响应。

中间件在缓存内容和路由匹配之前运行。有关更多详细信息，请参见[匹配路径](https://nextjs.org/docs/app/building-your-application/routing/middleware#matching-paths)。

### 用例

将中间件集成到您的应用程序中可以显著提高性能、安全性和用户体验。一些中间件特别有效的常见场景包括：

* 身份验证和授权：确保用户身份并在授予访问特定页面或 API 路由之前检查会话 cookie。
* 服务器端重定向：根据某些条件（例如，区域设置、用户角色）在服务器级别重定向用户。
* 服务器端重定向：根据某些条件（例如，区域设置、用户角色）在服务器级别重定向用户。
* 机器人检测：通过检测和阻止机器人流量来保护您的资源。
* 日志记录和分析：捕获和分析请求数据，以便在页面或 API 处理之前获得见解。
* 功能标记：动态启用或禁用功能，以实现无缝的功能发布或测试。

识别中间件可能不是最佳方法的情况同样重要。以下是需要注意的一些场景：

* 复杂的数据获取和处理：中间件并不设计用于直接的数据获取或处理，这应该在路由处理程序或服务器端工具中完成。
* 重计算任务：中间件应该是轻量级的并快速响应，否则可能会导致页面加载延迟。重计算任务或长时间运行的过程应该在专用的路由处理程序中完成。
* 广泛的会话管理：虽然中间件可以管理基本的会话任务，但广泛的会话管理应该由专用的身份验证服务或在路由处理程序中管理。
* 直接数据库操作：在中间件中执行直接的数据库操作并不推荐。数据库交互应该在路由处理程序或服务器端工具中进行。

### 约定

在项目根目录中使用文件 `middleware.ts` （或 `.js` ）来定义中间件。例如，在 `pages` 或 `app` 的同一级别，或者在适用的情况下在 `src` 内部。

> 注意：虽然每个项目只支持一个 `middleware.ts` 文件，但您仍然可以模块化地组织中间件逻辑。将中间件功能拆分到单独的 `.ts` 或 `.js` 文件中，并将它们导入到您的主 `middleware.ts` 文件中。这允许更清晰地管理特定路由的中间件，集中在 `middleware.ts` 中进行集中控制。通过强制使用单个中间件文件，它简化了配置，防止潜在冲突，并通过避免多个中间件层来优化性能。

### 示例

```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
 
// This function can be marked `async` if using `await` inside
export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}
 
// See "Matching Paths" below to learn more
export const config = {
  matcher: '/about/:path*',
}
```

### 匹配路径

中间件将在项目中的每个路由上被调用。因此，使用匹配器精确地定位或排除特定路由至关重要。以下是执行顺序：

1. 来自 `next.config.js` 的 `headers`&#x20;
2. 来自 `next.config.js` 的 `redirects`
3. 中间件 ( `rewrites` , `redirects` , 等)
4. 来自 `next.config.js` 的 `beforeFiles` ( `rewrites` )&#x20;
5. 文件系统路由 ( `public/` , `_next/static/` , `pages/` , `app/` , 等等)
6. 来自 `next.config.js` 的 `afterFiles` ( `rewrites` )&#x20;
7. 动态路由 ( `/blog/[slug]` )
8. 来自 `next.config.js` 的 `fallback` ( `rewrites` )&#x20;



有两种方法来定义中间件将运行的路径：

1. [自定义匹配器配置](https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher)
2. [条件语句](https://nextjs.org/docs/app/building-your-application/routing/middleware#conditional-statements)

#### 匹配器

`matcher` 允许您过滤在特定路径上运行的中间件。

{% code title="middleware.js" %}
```javascript
export const config = {
  matcher: '/about/:path*',
}
```
{% endcode %}

您可以使用数组语法匹配单个路径或多个路径：

{% code title="middleware.js" %}
```javascript
export const config = {
  matcher: ['/about/:path*', '/dashboard/:path*'],
}
```
{% endcode %}

`matcher` 配置允许完整的正则表达式，因此支持像负向前瞻或字符匹配这样的匹配。负向前瞻的一个示例，用于匹配所有路径，除了特定路径，可以在这里看到：

```javascript
export const config = {
  matcher: [
    /*
     * 匹配所有请求路径，除了以以下内容开头的路径：
     * - api (API 路由)
     * - _next/static (静态文件)
     * - _next/image（图像优化文件）
     * - favicon.ico, sitemap.xml, robots.txt（元数据文件）
     */
    '/((?!api|_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)',
  ],
}javasc

```

您还可以通过使用 `missing` 或 `has` 数组，或两者的组合，来绕过某些请求的中间件：

{% code title="middleware.js" %}
```javascript
export const config = {
  matcher: [
    /*
     * 匹配所有请求路径，除了以以下内容开头的路径：
     * - api (API 路由)
     * - _next/static (静态文件)
     * - _next/image（图像优化文件）
     * - favicon.ico, sitemap.xml, robots.txt（元数据文件）
     */
    {
      source:
        '/((?!api|_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)',
      missing: [
        { type: 'header', key: 'next-router-prefetch' },
        { type: 'header', key: 'purpose', value: 'prefetch' },
      ],
    },
 
    {
      source:
        '/((?!api|_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)',
      has: [
        { type: 'header', key: 'next-router-prefetch' },
        { type: 'header', key: 'purpose', value: 'prefetch' },
      ],
    },
 
    {
      source:
        '/((?!api|_next/static|_next/image|favicon.ico|sitemap.xml|robots.txt).*)',
      has: [{ type: 'header', key: 'x-present' }],
      missing: [{ type: 'header', key: 'x-missing', value: 'prefetch' }],
    },
  ],
}
```
{% endcode %}

> 注意：`matcher` 值需要是常量，以便可以在构建时进行静态分析。动态值如变量将被忽略。

配置的匹配器：

1. 必须以 `/` 开头
2. 可以包含命名参数: `/about/:path` 匹配 `/about/a` 和 `/about/b` 但不匹配 `/about/a/c`&#x20;
3. 可以对命名参数使用修饰符（以 `:` 开头）: `/about/:path*` 匹配 `/about/a/b/c` 因为 `*` 是零个或多个。 `?` 是零个或一个， `+` 是一个或多个
4. 可以使用括号中的正则表达式： `/about/(.*)` 与 `/about/:path*` 相同

在 [path-to-regexp](https://github.com/pillarjs/path-to-regexp#path-to-regexp-1) 文档中查看更多详细信息。

> 注意：为了向后兼容，Next.js 始终将 `/public` 视为 `/public/index` 。因此， `/public/:path` 的匹配器将匹配。

#### 条件语句

{% code title="middleware.ts" %}
```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
 
export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/about')) {
    return NextResponse.rewrite(new URL('/about-2', request.url))
  }
 
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.rewrite(new URL('/dashboard/user', request.url))
  }
}
```
{% endcode %}

### NextResponse

`NextResponse` API 允许您：

* `redirect` 将传入请求转发到另一个 URL
* `rewrite` 通过显示给定的 URL 进行响应
* 为 API 路由设置请求头， `getServerSideProps` 和 `rewrite` 目标
* 设置响应 cookies
* 设置响应头部

要从中间件生成响应，您可以：

1. `rewrite` 到一个路由（[页面](https://nextjs.org/docs/app/api-reference/file-conventions/page)或[路由处理程序](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)），该路由生成响应
2. 直接返回 a `NextResponse` 。请参见[生成响应](https://nextjs.org/docs/app/building-your-application/routing/middleware#producing-a-response)

### 使用 Cookies

Cookies 是常规头部。在 `Request` 中，它们存储在 `Cookie` 头部。在 `Response` 中，它们存储在 `Set-Cookie` 头部。Next.js 提供了一种方便的方式，通过 `cookies` 扩展在 `NextRequest` 和 `NextResponse` 上访问和操作这些 cookies。

1. 对于传入请求， `cookies` 提供以下方法： `get` 、 `getAll` 、 `set` 和 `delete` cookies。您可以使用 `has` 检查 cookie 是否存在，或使用 `clear` 删除所有 cookies。
2. 对于外发响应， `cookies` 有以下方法 `get` 、 `getAll` 、 `set` 和 `delete` 。

{% code title="middleware.ts" %}
```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
 
export function middleware(request: NextRequest) {
  // 假设在传入请求中存在 "Cookie:nextjs=fast" 头部
  // 使用 `RequestCookies` API 从请求中获取 cookies
  let cookie = request.cookies.get('nextjs')
  console.log(cookie) // => { name: 'nextjs', value: 'fast', Path: '/' }
  const allCookies = request.cookies.getAll()
  console.log(allCookies) // => [{ name: 'nextjs', value: 'fast' }]
 
  request.cookies.has('nextjs') // => true
  request.cookies.delete('nextjs')
  request.cookies.has('nextjs') // => false
 
  // 使用 `ResponseCookies` API 在响应中设置 cookies
  const response = NextResponse.next()
  response.cookies.set('vercel', 'fast')
  response.cookies.set({
    name: 'vercel',
    value: 'fast',
    path: '/',
  })
  cookie = response.cookies.get('vercel')
  console.log(cookie) // => { name: 'vercel', value: 'fast', Path: '/' }
  // 响应将包含 `Set-Cookie:vercel=fast;path=/` 头。
 
  return response
}
```
{% endcode %}

### 设置头部

您可以使用 `NextResponse` API 设置请求和响应头（自 Next.js v13.0.0 起可设置请求头）。

{% code title="middleware.ts" %}
```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
 
export function middleware(request: NextRequest) {
  // 克隆请求头并设置新头部 `x-hello-from-middleware1`
  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-hello-from-middleware1', 'hello')
 
  // 您还可以在 NextResponse.next 中设置请求头。
  const response = NextResponse.next({
    request: {
      // 新请求头部
      headers: requestHeaders,
    },
  })
 
  // 设置新的响应头 `x-hello-from-middleware2`
  response.headers.set('x-hello-from-middleware2', 'hello')
  return response
}
```
{% endcode %}

> 注意：避免设置较大的头部，因为这可能会导致 431 Request Header Fields Too Large 错误，具体取决于您的后端 web 服务器配置。

### CORS

您可以在中间件中设置 CORS 头以允许跨源请求，包括简单请求和预检请求。

{% code title="middleware.ts" %}
```typescript
import { NextRequest, NextResponse } from 'next/server'
 
const allowedOrigins = ['https://acme.com', 'https://my-app.org']
 
const corsOptions = {
  'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
}
 
export function middleware(request: NextRequest) {
  // 检查请求的来源
  const origin = request.headers.get('origin') ?? ''
  const isAllowedOrigin = allowedOrigins.includes(origin)
 
  // 处理预检请求
  const isPreflight = request.method === 'OPTIONS'
 
  if (isPreflight) {
    const preflightHeaders = {
      ...(isAllowedOrigin && { 'Access-Control-Allow-Origin': origin }),
      ...corsOptions,
    }
    return NextResponse.json({}, { headers: preflightHeaders })
  }
 
  // 处理简单请求
  const response = NextResponse.next()
 
  if (isAllowedOrigin) {
    response.headers.set('Access-Control-Allow-Origin', origin)
  }
 
  Object.entries(corsOptions).forEach(([key, value]) => {
    response.headers.set(key, value)
  })
 
  return response
}
 
export const config = {
  matcher: '/api/:path*',
}
```
{% endcode %}

> 注意：您可以为路由处理程序中的单个路由配置 CORS 头。

### 生成响应

您可以通过直接返回一个 `Response` 或 `NextResponse` 实例从中间件响应。（自 Next.js v13.1.0 起可用）

{% code title="middleware.ts" %}
```typescript
import type { NextRequest } from 'next/server'
import { isAuthenticated } from '@lib/auth'type
 
// Limit the middleware to paths starting with `/api/`
export const config = {
  matcher: '/api/:function*',
}
 
export function middleware(request: NextRequest) {
  // Call our authentication function to check the request
  if (!isAuthenticated(request)) {
    // Respond with JSON indicating an error message
    return Response.json(
      { success: false, message: 'authentication failed' },
      { status: 401 }
    )
  }
}
```
{% endcode %}

#### waitUntill[^1] 和 `NextFetchEvent`

`NextFetchEvent` 对象扩展了原生 `FetchEvent` 对象，并包含 `waitUntil()` 方法。

`waitUntil()` 方法接受一个 promise 作为参数，并在 promise 解决之前延长 Middleware 的生命周期。这对于在后台执行工作非常有用。

{% code title="middleware.ts" %}
```typescript
import { NextResponse } from 'next/server'
import type { NextFetchEvent, NextRequest } from 'next/server'
 
export function middleware(req: NextRequest, event: NextFetchEvent) {
  event.waitUntil(
    fetch('https://my-analytics-platform.com', {
      method: 'POST',
      body: JSON.stringify({ pathname: req.nextUrl.pathname }),
    })
  )
 
  return NextResponse.next()
}
```
{% endcode %}

### 高级中间件标志

在 Next.js 的 `v13.1` 中，引入了两个额外的标志用于中间件， `skipMiddlewareUrlNormalize` 和 `skipTrailingSlashRedirect` 以处理高级用例。

`skipTrailingSlashRedirect` 禁用 Next.js 重定向以添加或删除尾部斜杠。这允许在中间件内部进行自定义处理，以便为某些路径保留尾部斜杠而不为其他路径保留，这可以使增量迁移更容易。

{% code title="next.config.js" %}
```javascript
module.exports = {
  skipTrailingSlashRedirect: true,
}
```
{% endcode %}

{% code title="middleware.js" %}
```javascript
const legacyPrefixes = ['/docs', '/blog']
 
export default async function middleware(req) {
  const { pathname } = req.nextUrl
 
  if (legacyPrefixes.some((prefix) => pathname.startsWith(prefix))) {
    return NextResponse.next()
  }
 
  // apply trailing slash handling
  if (
    !pathname.endsWith('/') &&
    !pathname.match(/((?!\.well-known(?:\/.*)?)(?:[^/]+\/)*[^/]+\.\w+)/)
  ) {
    return NextResponse.redirect(
      new URL(`${req.nextUrl.pathname}/`, req.nextUrl)
    )
  }
}
```
{% endcode %}

`skipMiddlewareUrlNormalize` 允许在 Next.js 中禁用 URL 规范化，以使直接访问和客户端过渡的处理相同。在某些高级情况下，此选项通过使用原始 URL 提供完全控制。

{% code title="next.config.js" %}
```javascript
module.exports = {
  skipMiddlewareUrlNormalize: true,
}
```
{% endcode %}

{% code title="middleware.js" %}
```javascript
export default async function middleware(req) {
  const { pathname } = req.nextUrl
 
  // GET /_next/data/build-id/hello.json
 
  console.log(pathname)
  // 有标志的情况下值为 /_next/data/build-id/hello.json
  // 没有标志的情况下值会规范为 /hello
}
```
{% endcode %}

### 单元测试（实验性）

从 Next.js 15.1 开始， `next/experimental/testing/server` 包含用于帮助单元测试中间件文件的工具。单元测试中间件可以帮助确保它仅在所需路径上运行，并且在代码进入生产环境之前，自定义路由逻辑按预期工作。

`unstable_doesMiddlewareMatch` 函数可用于断言中间件是否会针对提供的 URL、头部和 cookies 运行。

```typescript
import { unstable_doesMiddlewareMatch } from 'next/experimental/testing/server'
 
expect(
  unstable_doesMiddlewareMatch({
    config,
    nextConfig,
    url: '/test',
  })
).toEqual(false)
```

整个中间件函数也可以进行测试。

```typescript
import { isRewrite, getRewrittenUrl } from 'next/experimental/testing/server'
 
const request = new NextRequest('https://nextjs.org/docs')
const response = await middleware(request)
expect(isRewrite(response)).toEqual(true)
expect(getRewrittenUrl(response)).toEqual('https://other-domain.com/docs')
// getRedirectUrl 也可以用于响应是重定向的情况
```

### 运行时

中间件默认使用 Edge 运行时。从 v15.2（canary）开始，我们对使用 Node.js 运行时提供实验性支持。要启用，请将标志添加到您的 `next.config` 文件中：

{% code title="next.config.ts" %}
```typescript
import type { NextConfig } from 'next'
 
const nextConfig: NextConfig = {
  experimental: {
    nodeMiddleware: true,
  },
}
 
export default nextConfig
```
{% endcode %}

然后在你的中间件文件中，将 `config` 对象中的运行时设置为 `nodejs` ：

{% code title="middleware.ts" %}
```typescript
export const config = {
  runtime: 'nodejs',
}
```
{% endcode %}

> 注意：此功能尚不建议用于生产环境。因此，Next.js 将抛出错误，除非您使用的是 next@canary 版本，而不是稳定版本。

### 版本历史

| 版本        | 更改                         |
| --------- | -------------------------- |
| `v15.2.0` | 中间件现在可以使用 Node.js 运行时（实验性） |
| `v13.1.0` | 添加了高级中间件标志                 |
| `v13.0.0` | 中间件可以修改请求头、响应头，并发送响应       |
| `v12.2.0` | 中间件稳定，请查看升级指南              |
| `v12.0.9` | 在边缘运行时强制使用绝对 URL（PR）       |
| `v12.0.0` | 中间件（Beta）已添加               |



[^1]: 
