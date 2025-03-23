# 链接和导航

在 Next.js 中有四种在路由之间导航的方法：

* 使用 `<Link>` 组件
* 使用 `useRouter` 钩子（客户端组件）
* 使用 `redirect` 函数（服务器组件）
* 使用原生History API

本页面将介绍如何使用这些选项，并深入探讨导航的工作原理。

### \<Link>组件

`<Link>` 是一个内置组件，扩展了 HTML `<a>` 标签，以提供<mark style="color:blue;">预取</mark>和路由之间的客户端导航。这是 Next.js 中在路由之间导航的主要和推荐方式。

您可以通过从 `next/link` 导入它，并向组件传递 `href` 属性来使用它：

```tsx
import Link from 'next/link'
 
export default function Page() {
  return <Link href="/dashboard">Dashboard</Link>
}
```

您可以向 `<Link>` 传递其他可选属性。有关更多信息，请参阅 API 参考。

### useRouter() 钩子

`useRouter` 钩子允许您从<mark style="color:blue;">客户端组件</mark>以编程方式更改路由。

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
}x
```

有关 `useRouter` 方法的完整列表，请参阅 API 参考。

> 建议：使用 `<Link>` 组件在路由之间导航，除非您有使用 `useRouter` 的特定要求。

### redirect 函数

对于服务器组件，请改用 `redirect` 函数。

```tsx
import { redirect } from 'next/navigation'
 
async function fetchTeam(id: string) {
  const res = await fetch('https://...')
  if (!res.ok) return undefined
  return res.json()
}
 
export default async function Profile({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  if (!id) {
    redirect('/login')
  }
 
  const team = await fetchTeam(id)
  if (!team) {
    redirect('/join')
  }
 
  // ...
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

### 使用原生 History API

Next.js 允许您使用原生 [`window.history.pushState`](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) 和 [`window.history.replaceState`](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState)方法来更新浏览器的历史堆栈，而无需重新加载页面。

`pushState` 和 `replaceState` 调用集成到 Next.js 路由器中，允许您与 `usePathname` 和 `useSearchParams` 同步。

#### `window.history.pushState`

使用它向浏览器的历史堆栈添加新条目。用户可以返回到先前的状态。例如，排序产品列表：

```tsx
'use client'
 
import { useSearchParams } from 'next/navigation'
 
export default function SortProducts() {
  const searchParams = useSearchParams()
 
  function updateSorting(sortOrder: string) {
    const params = new URLSearchParams(searchParams.toString())
    params.set('sort', sortOrder)
    window.history.pushState(null, '', `?${params.toString()}`)
  }
 
  return (
    <>
      <button onClick={() => updateSorting('asc')}>Sort Ascending</button>
      <button onClick={() => updateSorting('desc')}>Sort Descending</button>
    </>
  )
}
```

#### `window.history.replaceState`

使用它替换浏览器历史堆栈中的当前条目。用户无法返回到先前的状态。例如，切换应用程序的区域设置：

```tsx
'use client'
 
import { usePathname } from 'next/navigation'
 
export function LocaleSwitcher() {
  const pathname = usePathname()
 
  function switchLocale(locale: string) {
    // e.g. '/en/about' or '/fr/contact'
    const newPath = `/${locale}${pathname}`
    window.history.replaceState(null, '', newPath)
  }
 
  return (
    <>
      <button onClick={() => switchLocale('en')}>English</button>
      <button onClick={() => switchLocale('fr')}>French</button>
    </>
  )
}
```

### 路由和导航是如何工作的

App Router 采用混合方法进行路由和导航。在服务器上，您的应用程序代码会根据路由段自动进行代码分割。在客户端，Next.js 会预取并缓存路由段。这意味着，当用户导航到新路由时，浏览器不会重新加载页面，只有变化的路由段会重新渲染 - 提升了导航体验和性能。

#### 1.代码分割

代码分割允许您将应用程序代码拆分成更小的包，以便由浏览器下载和执行。这减少了每个请求传输的数据量和执行时间，从而提高了性能。

<mark style="color:blue;">服务器组件</mark>允许您的应用代码按路由段自动进行代码拆分。这意味着在导航时，仅加载当前路由所需的代码。

#### 2.预取

预取是一种在用户访问之前在后台预加载路由的方法。

在 Next.js 中，有两种方式预取路由：

* `<Link>` 组件：当路由在用户的视口中变为可见时，会自动进行预取。当页面首次加载或通过滚动进入视野时会进行预取。
* `router.prefetch()` : `useRouter` 钩子可以用于以编程方式预取路由。

`<Link>` 的默认预取行为（即当 `prefetch` 属性未指定或设置为 `null` 时）根据您对 `loading.js` 的使用情况而有所不同。只有共享布局，直到第一个 `loading.js` 文件的渲染“树”中的组件被预取并缓存为 `30s` 。这减少了获取整个动态路由的成本，并且意味着您可以显示即时加载状态，以便为用户提供更好的视觉反馈。

您可以通过将 `prefetch` 属性设置为 `false` 来禁用预取。或者，您可以通过将 `prefetch` 属性设置为 `true` 来预取超出加载边界的完整页面数据。

有关更多信息，请参阅 `<Link>` API 参考。

> 需要知道的是：
>
> 预取在开发中未启用，仅在生产中启用。

#### 3.缓存

Next.js 有一个名为<mark style="color:blue;">路由缓存</mark>的内存**客户端缓存**。当用户在应用中导航时，预取的路由段和访问过的路由的 React 服务器组件负载会存储在缓存中。

这意味着在导航时，缓存尽可能被重用，而不是向服务器发出新请求——通过减少请求数量和传输的数据量来提高性能。

了解<mark style="color:blue;">路由缓存</mark>的工作原理以及如何配置它。

#### 4.部分渲染

部分渲染意味着在导航时，仅有变化的路由段会在客户端重新渲染，而任何共享的段是保留的。

例如，当在两个兄弟路由 `/dashboard/settings` 和 `/dashboard/analytics` 之间导航时， `settings` 页面将被卸载， `analytics` 页面将以新的状态挂载，共享的 `dashboard`布局将被保留。此行为在同一动态段之间的两个路由之间也是存在的，例如在 `/blog/[slug]/page` 中从 `/blog/first` 导航到 `/blog/second` 时。

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

没有部分渲染的情况下，每次导航都会导致整个页面在客户端重新渲染。只渲染变化的部分减少了传输的数据量和执行时间，从而提高了性能。

#### 5.软导航

浏览器在页面之间导航时执行“硬导航”。 Next.js 应用路由器支持页面之间的“软导航”，确保仅重新渲染已更改的路由段（部分渲染）。 这使得在导航过程中客户端的 React 状态得以保留。

#### 6.后退和前进导航

默认情况下，Next.js 会保持后退和前进导航的滚动位置，并在路由器缓存中重新使用路由段。

#### 7.`pages/` 和 `app/` 之间的导航

当从 `pages/` 逐步迁移到 `app/` 时，Next.js 路由器将自动处理两者之间的硬导航。为了检测从 `pages/` 到 `app/` 的过渡，有一个客户端路由过滤器，利用对应用路由的概率检查，这有时可能导致误报。默认情况下，这种情况应该非常罕见，因为我们将误报的可能性配置为 0.01%。此可能性可以通过 `next.config.js` 中的 `experimental.clientRouterFilterAllowedRate` 选项自定义。需要注意的是，降低误报率将增加客户端包中生成的过滤器的大小。

另外，如果您更愿意完全禁用此处理并手动管理 `pages/` 和 `app/` 之间的路由，您可以在 `next.config.js` 中将 `experimental.clientRouterFilter` 设置为 false。当此功能被禁用时，任何与应用路由重叠的页面中的动态路由默认不会被正确导航。
