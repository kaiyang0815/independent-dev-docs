# 动态路由

当你事先不知道确切的段名称并想要从动态数据创建路由时，可以使用在请求时填充或在构建时预渲染的动态段。

### 约定

动态段可以通过将文件夹名称用方括号括起来创建： `[folderName]` 。例如， `[id]` 或 `[slug]` 。

动态段被作为 `params` 属性传递给 `layout` 、 `page` 、 `route` 和 `generateMetadata` 函数。

### 示例

例如，一个博客可以包含以下路由 `app/blog/[slug]/page.js` ，其中 `[slug]` 是用于博客文章的动态段。

{% code title="app/blog/[slug]/page.tsx" %}
```typescript
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  return <div>My Post: {slug}</div>
}
```
{% endcode %}

| 路由                        | 示例 URL    | params          |
| ------------------------- | --------- | --------------- |
| `app/blog/[slug]/page.js` | `/blog/a` | `{ slug: 'a' }` |
| `app/blog/[slug]/page.js` | `/blog/b` | `{ slug: 'b' }` |
| `app/blog/[slug]/page.js` | `/blog/c` | `{ slug: 'c' }` |

查看 generateStaticParams() 页面以了解如何生成该段的参数。

### 需要注意的点

* 由于 `params` 属性是一个 Promise。您必须使用 async/await 或 React 的 use 函数来访问这些值。
  * 在 14 及更早版本中， `params` 是一个同步属性。为了帮助向后兼容，您仍然可以在 Next.js 15 中以同步方式访问它，但这种行为将在未来被弃用。‘
* 动态段等同于 `pages` 目录中的动态路由。

### 生成静态参数

`generateStaticParams` 函数可以与动态路由段结合使用，以便在构建时静态生成路由，而不是在请求时按需生成。

{% code title="app/blog/[slug]/page.tsx" %}
```tsx
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
 
  return posts.map((post) => ({
    slug: post.slug,
  }))
}
```
{% endcode %}

`generateStaticParams` 函数的主要好处是其智能数据检索。如果在 `generateStaticParams` 函数中使用 `fetch` 请求获取内容，请求将被自动记忆。这意味着在多个 `generateStaticParams` 、布局和页面中，使用相同参数的 `fetch` 请求只会发送一次，从而缩短构建时间。

如果您要从 `pages` 目录迁移，请使用迁移指南。

有关更多信息和高级用例，请参阅 `generateStaticParams` 服务器函数文档。

### 捕获所用段

动态段可以通过在括号 `[...folderName]` 内添加省略号来扩展为捕获所有段。

例如， `app/shop/[...slug]/page.js` 将匹配 `/shop/clothes` ，但也会匹配 `/shop/clothes/tops` 、 `/shop/clothes/tops/t-shirts` 等等。

<table><thead><tr><th width="274.84765625">路由</th><th width="164.7265625">示例 URL</th><th>params</th></tr></thead><tbody><tr><td><code>app/shop/[...slug]/page.js</code></td><td><code>/shop/a</code></td><td><code>{ slug: ['a'] }</code></td></tr><tr><td><code>app/shop/[...slug]/page.js</code></td><td><code>/shop/a/b</code></td><td><code>{ slug: ['a', 'b'] }</code></td></tr><tr><td><code>app/shop/[...slug]/page.js</code></td><td><code>/shop/a/b/c</code></td><td><code>{ slug: ['a', 'b', 'c'] }</code></td></tr></tbody></table>

### 可选的捕获所有段

通过在双方括号中包含参数，可以使捕获所有段变为可选： `[[...folderName]]` 。

例如， `app/shop/[[...slug]]/page.js` 除了 `/shop/clothes` 、 `/shop/clothes/tops` 、 `/shop/clothes/tops/t-shirts` ，还将匹配 `/shop` 。

catch-all 和可选 catch-all 段之间的区别在于，使用可选时，没有参数的路由也会被匹配（如上例中的 `/shop` ）。

<table><thead><tr><th width="284.80859375">路由</th><th width="176.44140625">示例 UR</th><th>params</th></tr></thead><tbody><tr><td><code>app/shop/[[...slug]]/page.js</code></td><td><code>/shop</code></td><td><code>{ slug: undefined }</code></td></tr><tr><td><code>app/shop/[[...slug]]/page.js</code></td><td><code>/shop/a</code></td><td><code>{ slug: ['a'] }</code></td></tr><tr><td><code>app/shop/[[...slug]]/page.js</code></td><td><code>/shop/a/b</code></td><td><code>{ slug: ['a', 'b'] }</code></td></tr><tr><td><code>app/shop/[[...slug]]/page.js</code></td><td><code>/shop/a/b/c</code></td><td><code>{ slug: ['a', 'b', 'c'] }</code></td></tr></tbody></table>

### TypeScript

在使用 TypeScript 时，您可以根据配置的路由段为 `params` 添加类型。

{% code title="app/blog/[slug]/page.tsx" %}
```tsx
export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  return <h1>My Page</h1>
}
```
{% endcode %}

| 路由                                  |  params 类型定义                             |
| ----------------------------------- | ---------------------------------------- |
| `app/blog/[slug]/page.js`           | `{ slug: string }`                       |
| `app/shop/[...slug]/page.js`        | `{ slug: string[] }`                     |
| `app/shop/[[...slug]]/page.js`      | `{ slug?: string[] }`                    |
| `app/[categoryId]/[itemId]/page.js` | `{ categoryId: string, itemId: string }` |

> 需要知道的是：这可能在以后由 TypeScript 插件自动完成。

### 下一步

