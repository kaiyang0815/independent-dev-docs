# 数据获取和缓存

<details>

<summary>示例</summary>

* [Next.js Commerce](https://vercel.com/templates/next.js/nextjs-commerce)
* [On-Demand ISR](https://on-demand-isr.vercel.app/)
* [Next.js Form](https://github.com/vercel/next.js/tree/canary/examples/next-forms)

</details>

本指南将带您了解 Next.js 中数据获取和缓存的基础知识，提供实用示例和最佳实践。

这是 Next.js 中数据获取的一个最小示例：

{% code title="app/page.tsx" %}
```tsx
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```
{% endcode %}

此示例演示了在异步 React 服务器组件中使用 `fetch` API 进行基本的服务器端数据获取。

### 参考

* [`fetch`](https://nextjs.org/docs/app/api-reference/functions/fetch)
* React [`cache`](https://react.dev/reference/react/cache) &#x20;
* Next.js [`unstable_cache`](https://nextjs.org/docs/app/api-reference/functions/unstable_cache)&#x20;

### 示例

#### 使用 `fetch` API 在服务器上获取数据

该组件将获取并显示博客文章列表。 `fetch` 的响应默认不被缓存。

{% code title="app/page.tsx" %}
```tsx
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```
{% endcode %}

如果您在此路由的其他地方没有使用任何动态 API，它将在下一个构建期间预渲染为静态页面。然后可以使用增量静态再生更新数据。

要防止页面预渲染，您可以将以下内容添加到您的文件中：

```typescript
export const dynamic = 'force-dynamic'
```

然而，您通常会使用像 `cookies` 、 `headers` 或从页面属性中读取传入的 `searchParams` 的函数，这将自动使页面动态渲染。在这种情况下，您不需要显式使用 `force-dynamic` 。

#### 使用 ORM 或数据库在服务器上获取数据

该组件将获取并显示博客文章列表。数据库的响应默认不被缓存，但可以通过额外配置进行缓存。

{% code title="app/page.tsx" %}
```tsx
import { db, posts } from '@/lib/db'
 
export default async function Page() {
  const allPosts = await db.select().from(posts)
  return (
    <ul>
      {allPosts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```
{% endcode %}

如果您在此路由的其他地方没有使用任何动态 API，它将在下一个构建期间预渲染为静态页面。然后可以使用增量静态再生更新数据。

要防止页面预渲染，您可以将以下内容添加到您的文件中：

```typescript
export const dynamic = 'force-dynamic'
```

然而，您通常会使用像 `cookies` 、 `headers` 或从页面属性中读取传入的 `searchParams` 的函数，这将自动使页面动态渲染。在这种情况下，您不需要显式使用 `force-dynamic` 。

#### 在客户端获取数据

我们建议首先尝试在服务器端获取数据。

然而，在某些情况下，客户端数据获取是有意义的。在这些场景中，您可以手动在 `useEffect` 中调用 `fetch` （不推荐），或者依赖社区中流行的 React 库（如 SWR 或 React Query）进行客户端获取。

{% code title="app/page.tsx" %}
```tsx
'use client'
 
import { useState, useEffect } from 'react'
 
export function Posts() {
  const [posts, setPosts] = useState(null)
 
  useEffect(() => {
    async function fetchPosts() {
      const res = await fetch('https://api.vercel.app/blog')
      const data = await res.json()
      setPosts(data)
    }
    fetchPosts()
  }, [])
 
  if (!posts) return <div>Loading...</div>
 
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```
{% endcode %}

### 使用 ORM 或数据库缓存数据

您可以使用 `unstable_cache` API 在运行 `next build` 时缓存响应。

{% code title="app/page.tsx" %}
```tsx
import { unstable_cache } from 'next/cache'
import { db, posts } from '@/lib/db'
 
const getPosts = unstable_cache(
  async () => {
    return await db.select().from(posts)
  },
  ['posts'],
  { revalidate: 3600, tags: ['posts'] }
)
 
export default async function Page() {
  const allPosts = await getPosts()
 
  return (
    <ul>
      {allPosts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```
{% endcode %}

此示例将数据库查询的结果缓存 1 小时（3600 秒）。它还添加了缓存标签 `posts` ，该标签可以通过增量静态再生进行失效。

### 在多个函数之间重用数据

Next.js 使用像 `generateMetadata` 和 `generateStaticParams` 的 API，您需要在 `page` 中使用相同获取的数据。

如果您使用 `fetch` ，可以通过添加 `cache: 'force-cache'` 来缓存请求。这意味着您可以安全地使用相同的选项调用相同的 URL，并且只会发出一个请求。

> 注意：在之前的 Next.js 版本中，使用 `fetch` 的默认 `cache` 值为 `force-cache`。在版本 15 中，这一点发生了变化，默认值为 `cache: no-store` 。

{% code title="app/blog/[id]/page.tsx" %}
```tsx
import { notFound } from 'next/navigation'
 
interface Post {
  id: string
  title: string
  content: string
}
 
async function getPost(id: string) {
  const res = await fetch(`https://api.vercel.app/blog/${id}`, {
    cache: 'force-cache',
  })
  const post: Post = await res.json()
  if (!post) notFound()
  return post
}
 
export async function generateStaticParams() {
  const posts = await fetch('https://api.vercel.app/blog', {
    cache: 'force-cache',
  }).then((res) => res.json())
 
  return posts.map((post: Post) => ({
    id: String(post.id),
  }))
}
 
export async function generateMetadata({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const post = await getPost(id)
 
  return {
    title: post.title,
  }
}
 
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const post = await getPost(id)
 
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}
```
{% endcode %}

如果您没有使用 `fetch` ，而是直接使用 ORM 或数据库，您可以使用 React `cache` 函数包装您的数据获取。这将去重并只进行一次查询。

```tsx
import { cache } from 'react'
import { db, posts, eq } from '@/lib/db' // Example with Drizzle ORM
import { notFound } from 'next/navigation'
 
export const getPost = cache(async (id) => {
  const post = await db.query.posts.findFirst({
    where: eq(posts.id, parseInt(id)),
  })
 
  if (!post) notFound()
  return post
})
```

#### 重新验证缓存数据

了解有关使用增量静态再生重新验证缓存数据的更多信息。

### 模式

#### 并行和顺序数据获取

在组件内部获取数据时，您需要了解两种数据获取模式：并行和顺序。

<figure><picture><source srcset="../../.gitbook/assets/image (5).png" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/image (4).png" alt=""></picture><figcaption></figcaption></figure>

* 顺序：组件树中的请求相互依赖。这可能导致更长的加载时间。
* 并行：路由中的请求被积极发起，并将同时加载数据。这减少了加载数据所需的总时间。

**顺序数据获取**

如果您有嵌套组件，并且每个组件获取自己的数据，那么如果这些数据请求没有被记忆化，数据获取将按顺序发生。

可能会有一些情况需要这种模式，因为一个获取依赖于另一个的结果。例如， `Playlists` 组件只有在 `Artist` 组件完成数据获取后才会开始获取数据，因为 `Playlists` 依赖于 `artistID` 属性：

{% code title="app/artist/[username]/page.tsx" %}
```tsx
export default async function Page({
  params,
}: {
  params: Promise<{ username: string }>
}) {
  const { username } = await params
  // 获取艺术家信息
  const artist = await getArtist(username)
 
  return (
    <>
      <h1>{artist.name}</h1>
      {/* 在播放列表组件加载时显示备用用户界面 */}
      <Suspense fallback={<div>Loading...</div>}>
        {/* 将 artistID 传递给播放列表组件 */}
        <Playlists artistID={artist.id} />
      </Suspense>
    </>
  )
}
 
async function Playlists({ artistID }: { artistID: string }) {
  // 使用 artistID 获取播放列表
  const playlists = await getArtistPlaylists(artistID)
 
  return (
    <ul>
      {playlists.map((playlist) => (
        <li key={playlist.id}>{playlist.name}</li>
      ))}
    </ul>
  )
}
```
{% endcode %}

您可以使用 `loading.js` （用于路由段）或 React `<Suspense>` （用于嵌套组件）来显示即时加载状态，同时 React 正在流入结果。

这将防止整个路由被数据请求阻塞，用户将能够与页面上已准备好的部分进行交互。

**并行数据获取**

默认情况下，布局和页面段以并行方式呈现。这意味着请求将并行发起。

然而，由于 `async` / `await` 的特性，同一段或组件内的等待请求将阻塞其下的任何请求。

要并行获取数据，您可以通过在使用数据的组件外部定义请求来主动发起请求。这样做可以节省时间，通过并行发起两个请求，然而，用户在两个承诺被解决之前不会看到呈现的结果。

在下面的示例中， `getArtist` 和 `getAlbums` 函数在 `Page` 组件外部定义，并在组件内部使用 `Promise.all` 初始化：

{% code title="app/artist/[username]/page.tsx" %}
```tsx
import Albums from './albums'
 
async function getArtist(username: string) {
  const res = await fetch(`https://api.example.com/artist/${username}`)
  return res.json()
}
 
async function getAlbums(username: string) {
  const res = await fetch(`https://api.example.com/artist/${username}/albums`)
  return res.json()
}
 
export default async function Page({
  params,
}: {
  params: Promise<{ username: string }>
}) {
  const { username } = await params
  const artistData = getArtist(username)
  const albumsData = getAlbums(username)
 
  // 并行初始化所有请求
  const [artist, albums] = await Promise.all([artistData, albumsData])
 
  return (
    <>
      <h1>{artist.name}</h1>
      <Albums list={albums} />
    </>
  )
}
```
{% endcode %}

此外，您可以添加一个 Suspense 边界来分割渲染工作，并尽快显示部分结果。

#### 预加载数据

另一种防止瀑布流的方法是使用预加载模式，通过创建一个工具函数，在阻塞请求之前主动调用它。例如， `checkIsAvailable()` 阻止 `<Item/>`渲染，因此您可以在它之前调用 `preload()` 来主动初始化 `<Item/>` 数据依赖关系。当 `<Item/>` 被渲染时，它的数据已经被获取。

请注意， `preload` 函数不会阻止 `checkIsAvailable()` 运行。

{% code title="components/Item.tsx" %}
```tsx
import { getItem } from '@/utils/get-item'
 
export const preload = (id: string) => {
  // void 评估给定的表达式并返回 undefined
  // https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void
  void getItem(id)
}
export default async function Item({ id }: { id: string }) {
  const result = await getItem(id)
  // ...
}
```
{% endcode %}

{% code title="app/item/[id]/page.tsx" %}
```tsx
import Item, { preload, checkIsAvailable } from '@/components/Item'
 
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  // 开始加载项目数据
  preload(id)
  // 执行另一个异步任务
  const isAvailable = await checkIsAvailable()
 
  return isAvailable ? <Item id={id} /> : null
}
```
{% endcode %}

> 注意：“preload” 函数可以有任何名称，因为它是一个模式，而不是一个 API。

**使用 `React cache` 和 `server-only` 结合预加载模式**

您可以结合 `cache` 函数、 `preload` 模式和 `server-only` 包来创建一个可以在整个应用中使用的数据获取工具。

{% code title="utils/get-item.ts" %}
```tsx
import { cache } from 'react'
import 'server-only'
 
export const preload = (id: string) => {
  void getItem(id)
}
 
export const getItem = cache(async (id: string) => {
  // ...
})
```
{% endcode %}

通过这种方法，您可以提前获取数据，缓存响应，并确保此数据获取仅在服务器上发生。

`utils/get-item` 导出可以被布局、页面或其他组件使用，以便让它们控制何时获取项目的数据。

> 注意：
>
> * 我们建议使用 `server-only` 包，以确保服务器数据获取函数永远不会在客户端使用。

#### 防止敏感数据暴露给客户端

我们建议使用 React 的 taint API， [`taintObjectReference`](https://react.dev/reference/react/experimental_taintObjectReference) 和 [`taintUniqueValue`](https://react.dev/reference/react/experimental_taintUniqueValue) ，以防止整个对象实例或敏感值被传递给客户端。

要在您的应用程序中启用 taint，设置 Next.js 配置 `experimental.taint`选项为 `true` ：

{% code title="next.config.js" %}
```javascript
module.exports = {
  experimental: {
    taint: true,
  },
}
```
{% endcode %}

然后将您想要 taint 的对象或值传递给 `experimental_taintObjectReference` 或 `experimental_taintUniqueValue` 函数：

```typescript
import { queryDataFromDB } from './api'
import {
  experimental_taintObjectReference,
  experimental_taintUniqueValue,
} from 'react'
 
export async function getUserData() {
  const data = await queryDataFromDB()
  experimental_taintObjectReference(
    'Do not pass the whole user object to the client',
    data
  )
  experimental_taintUniqueValue(
    "Do not pass the user's address to the client",
    data,
    data.address
  )
  return data
}
```

{% code title="app/page.tsx" %}
```tsx
import { getUserData } from './data'
 
export async function Page() {
  const userData = getUserData()
  return (
    <ClientComponent
      user={userData} // 这将导致错误，因为 taintObjectReference
      address={userData.address} // 这将导致错误，因为 taintObjectReference
    />
  )
}
```
{% endcode %}

