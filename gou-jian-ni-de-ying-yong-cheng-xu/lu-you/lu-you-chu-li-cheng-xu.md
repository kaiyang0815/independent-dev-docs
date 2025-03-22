# 路由处理程序

路由处理程序允许您使用 Web 请求和响应 API 为特定路由创建自定义请求处理程序。

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

> 注意：路由处理程序仅在 `app` 目录中可用。它们是 `pages` 目录中 API 路由的等效项，这意味着您不需要同时使用 API 路由和路由处理程序。

### 约定

路由处理程序在 `app` 目录中的 `route.js|ts` 文件中定义：

{% code title="app/api/route.ts" %}
```tsx
export async function GET(request: Request) {}
```
{% endcode %}

路由处理程序可以嵌套在 `app` 目录中的任何位置，类似于 `page.js` 和 `layout.js` 。但是在与 `page.js` 相同的路由段级别上不能有 `route.js`文件。

#### 支持的HTTP方法

支持以下 HTTP 方法： `GET` 、 `POST` 、 `PUT` 、 `PATCH` 、 `DELETE` 、 `HEAD` 和 `OPTIONS` 。如果调用不支持的方法，Next.js 将返回 `405 Method Not Allowed` 响应。

#### 扩展的`NextRequest`和`NextResponse`

除了支持原生的请求和响应 API，Next.js 还通过 `NextRequest` 和 `NextResponse` 扩展了它们，以提供方便的助手用于高级用例。

### 行为

#### 缓存

路由处理程序默认情况下不进行缓存。然而，您可以选择对 `GET` 方法进行缓存。其他支持的 HTTP 方法不进行缓存。要缓存 `GET` 方法，请在您的路由处理程序文件中使用<mark style="color:blue;">路由配置选项</mark>，例如 `export const dynamic = 'force-static'` 。

{% code title="app/items/route.ts" %}
```tsx
export const dynamic = 'force-static'
 
export async function GET() {
  const res = await fetch('https://data.mongodb-api.com/...', {
    headers: {
      'Content-Type': 'application/json',
      'API-Key': process.env.DATA_API_KEY,
    },
  })
  const data = await res.json()
 
  return Response.json({ data })
}
```
{% endcode %}

> 注意：其他支持的 HTTP 方法不会被缓存，即使它们与一个被缓存的 `GET` 方法放在同一个文件中。

#### 特殊路由处理程序

特殊路由处理程序如 `sitemap.ts` 、 `opengraph-image.tsx` 和 `icon.tsx` 以及其他元数据文件默认保持静态，除非它们使用动态 API 或动态配置选项。

#### 路由解析

您可以将 `route` 视为最低级别的路由原语。

* 它们**不参与**布局或客户端导航，如 `page` 。
* 在与 `page.js` 相同的路由下**不能有** `route.js` 文件。

| 页面                   | 路由                 | 结果 |
| -------------------- | ------------------ | -- |
| `app/page.js`        | `app/route.js`     | 冲突 |
| `app/page.js`        | `app/api/route.js` | 有效 |
| `app/[user]/page.js` | `app/api/route.js` | 有效 |

每个 `route.js` 或 `page.js` 文件会接管该路由的所有 HTTP 动作。

{% code title="app/page.ts" %}
```typescript
export default function Page() {
  return <h1>Hello, Next.js!</h1>
}
 
// 冲突
// `app/route.ts`
export async function POST(request: Request) {}
```
{% endcode %}

### 示例

以下示例展示了如何将路由处理程序与其他 Next.js API 和功能结合使用。

#### 重新验证缓存数据

您可以使用增量静态再生（ISR）重新验证缓存数据：

{% code title="app/posts/route.ts" %}
```typescript
export const revalidate = 60
 
export async function GET() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()
 
  return Response.json(posts)
}
```
{% endcode %}

#### Cookies

您可以使用 `cookies` 从 `next/headers` 读取或设置 cookies。此服务器功能可以在路由处理程序中直接调用，或嵌套在另一个函数中。

或者，您可以使用 `Set-Cookie` 头返回一个新的 `Response` 。

{% code title="app/api/route.ts" %}
```typescript
import { cookies } from 'next/headers'
 
export async function GET(request: Request) {
  const cookieStore = await cookies()
  const token = cookieStore.get('token')
 
  return new Response('Hello, Next.js!', {
    status: 200,
    headers: { 'Set-Cookie': `token=${token.value}` },
  })
}
```
{% endcode %}

您还可以使用底层 Web API 从请求中读取 cookies ( `NextRequest` ):

{% code title="app/api/route.ts" %}
```typescript
import { type NextRequest } from 'next/server'
 
export async function GET(request: NextRequest) {
  const token = request.cookies.get('token')
}
```
{% endcode %}

#### 请求头

您可以从 `next/headers` 中读取带有 `headers` 的头部。此服务器功能可以在路由处理程序中直接调用，也可以嵌套在另一个函数内部。

此 `headers` 实例是只读的。要设置头部，您需要返回一个新的 `Response` 和新的 `headers` 。

{% code title="app/api/route.ts" %}
```typescript
import { headers } from 'next/headers'
 
export async function GET(request: Request) {
  const headersList = await headers()
  const referer = headersList.get('referer')
 
  return new Response('Hello, Next.js!', {
    status: 200,
    headers: { referer: referer },
  })
}
```
{% endcode %}

您还可以使用底层 Web API 从请求中读取头信息（ `NextRequest` ）：

{% code title="app/api/route.ts" %}
```typescript
import { type NextRequest } from 'next/server'
 
export async function GET(request: NextRequest) {
  const requestHeaders = new Headers(request.headers)
}
```
{% endcode %}

#### 重定向

{% code title="app/api/route.ts" %}
```typescript
import { redirect } from 'next/navigation'
 
export async function GET(request: Request) {
  redirect('https://nextjs.org/')
}
```
{% endcode %}

#### 动态路由段

路由处理程序可以使用<mark style="color:blue;">动态段</mark>从动态数据创建请求处理程序。

{% code title="app/items/[slug]/route.ts" %}
```typescript
typetype
export async function GET(
  request: Request,
  { params }: { params: Promise<{ slug: string }> }
) {
  const { slug } = await params // 'a', 'b', or 'c'
}
```
{% endcode %}

<table><thead><tr><th width="276.4296875">路由</th><th width="212">示例 URL</th><th>params</th></tr></thead><tbody><tr><td><code>app/items/[slug]/route.js</code></td><td><code>/items/a</code></td><td><code>Promise&#x3C;{ slug: 'a' }></code></td></tr><tr><td><code>app/items/[slug]/route.js</code></td><td><code>/items/b</code></td><td><code>Promise&#x3C;{ slug: 'b' }></code></td></tr><tr><td><code>app/items/[slug]/route.js</code></td><td><code>/items/c</code></td><td><code>Promise&#x3C;{ slug: 'c' }></code></td></tr></tbody></table>

#### URL 查询参数

传递给路由处理程序的请求对象是一个 `NextRequest` 实例，它具有[一些额外的便利方法](https://nextjs.org/docs/app/api-reference/functions/next-request#nexturl)，包括更轻松地处理查询参数。

{% code title="app/api/search/route.ts" %}
```typescript
import { type NextRequest } from 'next/server'
 
export function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  const query = searchParams.get('query')
  // 对于 /api/search?query=hello 来说 query 是 "hello" 
}
```
{% endcode %}

#### 流媒体

流媒体通常与大型语言模型（LLMs）结合使用，例如 OpenAI，用于 AI 生成的内容。了解更多关于 AI SDK 的信息。

{% code title="app/api/chat/route.ts" %}
```typescript
import { openai } from '@ai-sdk/openai'
import { StreamingTextResponse, streamText } from 'ai'
 
export async function POST(req: Request) {
  const { messages } = await req.json()
  const result = await streamText({
    model: openai('gpt-4-turbo'),
    messages,
  })
 
  return new StreamingTextResponse(result.toAIStream())
}
```
{% endcode %}

这些抽象使用 Web API 来创建流。您也可以直接使用底层 Web API。

{% code title="app/api/route.ts" %}
```typescript
// https://developer.mozilla.org/docs/Web/API/ReadableStream#convert_async_iterator_to_stream
function iteratorToStream(iterator: any) {
  return new ReadableStream({
    async pull(controller) {
      const { value, done } = await iterator.next()
 
      if (done) {
        controller.close()
      } else {
        controller.enqueue(value)
      }
    },
  })
}
 
function sleep(time: number) {
  return new Promise((resolve) => {
    setTimeout(resolve, time)
  })
}
 
const encoder = new TextEncoder()
 
async function* makeIterator() {
  yield encoder.encode('<p>One</p>')
  await sleep(200)
  yield encoder.encode('<p>Two</p>')
  await sleep(200)
  yield encoder.encode('<p>Three</p>')
}
 
export async function GET() {
  const iterator = makeIterator()
  const stream = iteratorToStream(iterator)
 
  return new Response(stream)
}
```
{% endcode %}

#### 请求体

您可以使用标准 Web API 方法读取 `Request` 主体：

{% code title="app/items/route.ts" %}
```typescript
export async function POST(request: Request) {
  const res = await request.json()
  return Response.json({ res })
}
```
{% endcode %}

#### 请求体 FormData

您可以使用 `request.formData()` 函数读取 `FormData` ：

{% code title="app/items/route.ts" %}
```typescript
export async function POST(request: Request) {
  const formData = await request.formData()
  const name = formData.get('name')
  const email = formData.get('email')
  return Response.json({ name, email })
}
```
{% endcode %}

由于 `formData` 数据都是字符串，您可能想使用 [`zod-form-data`](https://www.npmjs.com/zod-form-data) 来验证请求并以您喜欢的格式检索数据（例如 `number` ）。

#### CORS

您可以使用标准 Web API 方法为特定路由处理程序设置 CORS 头

{% code title="app/api/route.ts" %}
```typescript
export async function GET(request: Request) {
  return new Response('Hello, Next.js!', {
    status: 200,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    },
  })
}
```
{% endcode %}

> 注意：
>
> * 要为多个路由处理程序添加 CORS 头，您可以使用[中间件](https://nextjs.org/docs/app/building-your-application/routing/middleware#cors)或 [`next.config.js` 文件](https://nextjs.org/docs/app/api-reference/config/next-config-js/headers#cors)。
> * 或者，请参阅我们的 [CORS 示例](https://github.com/vercel/examples/blob/main/edge-functions/cors/lib/cors.ts)包。

#### Webhooks

您可以使用路由处理程序接收来自第三方服务的 Webhook：

{% code title="app/api/route.ts" %}
```typescript
export async function POST(request: Request) {
  try {
    const text = await request.text()
    // 处理 webhook 负载
  } catch (error) {
    return new Response(`Webhook error: ${error.message}`, {
      status: 400,
    })
  }
 
  return new Response('Success!', {
    status: 200,
  })
}
```
{% endcode %}

值得注意的是，与使用页面路由的 API 路由不同，您无需使用 `bodyParser` 来使用任何额外的配置。

#### 非 UI 响应

您可以使用路由处理程序返回非 UI 内容。请注意， `sitemap.xml` 、 `robots.txt` 、 `app icons` 和开放图像都具有内置支持。

{% code title="app/rss.xml/route.ts" %}
```typescript
export async function GET() {
  return new Response(
    `<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
 
<channel>
  <title>Next.js Documentation</title>
  <link>https://nextjs.org/docs</link>
  <description>The React Framework for the Web</description>
</channel>
 
</rss>`,
    {
      headers: {
        'Content-Type': 'text/xml',
      },
    }
  )
}
```
{% endcode %}

#### 段配置选项

路由处理程序使用与页面和布局相同的[路由段配置](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config)。

{% code title="app/items/route.ts" %}
```typescript
export const dynamic = 'auto'
export const dynamicParams = true
export const revalidate = false
export const fetchCache = 'auto'
export const runtime = 'nodejs'
export const preferredRegion = 'auto'
```
{% endcode %}

有关更多详细信息，请参阅 API 参考。

### API 参考

