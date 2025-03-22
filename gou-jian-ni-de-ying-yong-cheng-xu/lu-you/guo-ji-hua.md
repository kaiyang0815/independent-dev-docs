# 国际化

Next.js 使您能够配置内容的路由和渲染，以支持多种语言。使您的网站适应不同的地区包括翻译内容（本地化）和国际化路由。

### 术语

* **Locale**：一组语言和格式偏好的标识符。这通常包括用户的首选语言以及他们的地理区域。
  * `en-US` : 美国英语
  * `nl-NL` : 荷兰人所说的荷兰语
  * `nl` : 荷兰语，无特定地区

### 路由概述

建议使用用户在浏览器中的语言偏好来选择使用哪个区域设置。更改您的首选语言将修改传入的 `Accept-Language` 头部到您的应用程序。

例如，使用以下库，您可以查看传入的 `Request` 以确定选择哪个区域设置，基于您计划支持的 `Headers` 和默认区域设置。

{% code title="middleware.js" %}
```javascript
import { match } from '@formatjs/intl-localematcher'
import Negotiator from 'negotiator'
 
let headers = { 'accept-language': 'en-US,en;q=0.5' }
let languages = new Negotiator({ headers }).languages()
let locales = ['en-US', 'nl-NL', 'nl']
let defaultLocale = 'en-US'
 
match(languages, locales, defaultLocale) // -> 'en-US'
```
{% endcode %}

路由可以通过子路径（ `/fr/products` ）或域名（ `my-site.fr/products` ）进行国际化。有了这些信息，您现在可以根据中间件中的区域设置重定向用户。

{% code title="middleware.js" %}
```javascript
import { NextResponse } from "next/server";
 
let locales = ['en-US', 'nl-NL', 'nl']
 
// 获取首选区域设置，类似于上述内容或使用库
function getLocale(request) { ... }
 
export function middleware(request) {
  // 检查路径名中是否存在任何支持的区域设置
  const { pathname } = request.nextUrl
  const pathnameHasLocale = locales.some(
    (locale) => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  )
 
  if (pathnameHasLocale) return
 
  // Redirect if there is no locale
  const locale = getLocale(request)
  request.nextUrl.pathname = `/${locale}${pathname}`
  // e.g. incoming request is /products
  // 新的 URL 现在是 /en-US/products
  return NextResponse.redirect(request.nextUrl)
}
 
export const config = {
  matcher: [
    // 跳过所有内部路径 (_next)
    '/((?!_next).*)',
    // 可选：仅在根目录 (/) URL 上运行
    // '/'
  ],
}
```
{% endcode %}

最后，确保所有特殊文件都嵌套在 `app/[lang]` 下的 `app/` 中。这使得 Next.js 路由器能够动态处理路由中不同的区域设置，并将 `lang` 参数转发到每个布局和页面。例如：

{% code title="app/[lang]/page.tsx" %}
```typescript
// 您现在可以访问当前区域设置
// 例如 /en-US/products -> `lang` 是 "en-US"
export default async function Page({
  params,
}: {
  params: Promise<{ lang: string }>
}) {
  const { lang } = await params
  return ...
}
```
{% endcode %}

根布局也可以嵌套在新文件夹中（例如 `app/[lang]/layout.js` ）。

### 本地化

基于用户首选语言或本地化的显示内容更改并不是 Next.js 特有的。下面描述的模式在任何 Web 应用程序中都可以使用。

假设我们想在应用程序中支持英语和荷兰语内容。我们可能会维护两个不同的“词典”，即将某个键映射到本地化字符串的对象。例如：

{% code title="dictionaries/en.json" %}
```json
{
  "products": {
    "cart": "Add to Cart"
  }
}
```
{% endcode %}

{% code title="dictionaries/nl.json" %}
```json
{
  "products": {
    "cart": "Toevoegen aan Winkelwagen"
  }
}
```
{% endcode %}

然后我们可以创建一个 `getDictionary` 函数来加载请求的区域的翻译：

{% code title="app/[lang]/dictionaries.ts" %}
```typescript
import 'server-only'
 
const dictionaries = {
  en: () => import('./dictionaries/en.json').then((module) => module.default),
  nl: () => import('./dictionaries/nl.json').then((module) => module.default),
}
 
export const getDictionary = async (locale: 'en' | 'nl') =>
  dictionaries[locale]()
```
{% endcode %}

根据当前选择的语言，我们可以在布局或页面中获取字典。

{% code title="app/[lang]/page.tsx" %}
```typescript
import { getDictionary } from './dictionaries'
 
export default async function Page({
  params,
}: {
  params: Promise<{ lang: 'en' | 'nl' }>
}) {
  const { lang } = await params
  const dict = await getDictionary(lang) // en
  return <button>{dict.products.cart}</button> // Add to Cart
}
```
{% endcode %}

因为 `app/` 目录中的所有布局和页面默认使用服务器组件，所以我们不需要担心翻译文件的大小会影响我们的客户端 JavaScript 包的大小。此代码只会在服务器上运行，只有生成的 HTML 会发送到浏览器。

### 静态生成

要为给定的语言环境生成静态路由，我们可以在任何页面或布局中使用 `generateStaticParams` 。这可以是全局的，例如，在根布局中：

{% code title="app/[lang]/layout.tsx" %}
```typescript
export async function generateStaticParams() {
  return [{ lang: 'en-US' }, { lang: 'de' }]
}
 
export default async function RootLayout({
  children,
  params,
}: Readonly<{
  children: React.ReactNode
  params: Promise<{ lang: 'en-US' | 'de' }>
}>) {
  return (
    <html lang={(await params).lang}>
      <body>{children}</body>
    </html>
  )
}
```
{% endcode %}

### 资源

* [最小化的国际化路由和翻译](https://github.com/vercel/next.js/tree/canary/examples/i18n-routing)
* [`next-intl`](https://next-intl.dev/)
* [`next-international`](https://github.com/QuiiBz/next-international)
* [`next-i18n-router`](https://github.com/i18nexus/next-i18n-router)
* [`paraglide-next`](https://inlang.com/m/osslbuzt/paraglide-next-i18n)
* [`lingui`](https://lingui.dev/)
