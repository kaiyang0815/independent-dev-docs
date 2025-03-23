# 如何创建布局和页面

Next.js 使用基于文件系统的路由，这意味着您可以使用文件夹和文件来定义路由。本页面将指导您如何创建布局和页面，并在它们之间链接。

### 创建页面

页面是根据特定路由渲染的用户界面。要创建一个页面，请在 `app` 目录中添加一个 `page`文件，并默认导出一个 React 组件。例如，要创建一个索引页面（ `/` ）：

<figure><picture><source srcset="https://nextjs.org/_next/image?url=https%3A%2F%2Fh8DxKfmAPhn8O0p3.public.blob.vercel-storage.com%2Fdocs%2Fdark%2Fpage-special-file.png&#x26;w=1920&#x26;q=75" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/image (30).png" alt=""></picture><figcaption></figcaption></figure>

{% code title="app/page.tsx" %}
```tsx
export default function Page() {
  return <h1>Hello Next.js!</h1>
}
```
{% endcode %}

### 创建布局

布局是多个页面共享的用户界面。在导航时，布局保持状态，保持交互性，并且不会重新渲染。

你可以通过从 `layout` 文件中默认导出一个 React 组件来定义布局。该组件应该接受一个 `children` 属性，该属性可以是一个页面或另一个布局。

例如，要创建一个接受你的索引页面作为子项的布局，请在 `app` 目录中添加一个 `layout` 文件：

<figure><picture><source srcset="../../.gitbook/assets/image (1) (1) (1).png" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""></picture><figcaption></figcaption></figure>

```tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        {/* Layout UI */}
        {/* Place children where you want to render a page or nested layout */}
        <main>{children}</main>
      </body>
    </html>
  )
}
```

上述布局称为根布局，因为它在 `app` 目录的根部定义。根布局是必须的，并且必须包含 `html` 和 `body` 标签。

### 创建嵌套路由

嵌套路由是由多个 URL 段组成的路由。例如， `/blog/[slug]` 路由由三个段组成：

* `/` （根段）
* `blog` （段）
* `[slug]` （叶子段）

在 Next.js 中：

* 文件夹用于定义映射到 URL 段的路由段。
* 文件（如 `page` 和 `layout` ）用于创建在某个段中显示的用户界面。

要创建嵌套路由，可以将文件夹嵌套在一起。例如，要为 `/blog` 添加路由，请在 `app` 目录中创建一个名为 `blog` 的文件夹。然后，要使 `/blog` 可公开访问，请添加一个 `page`文件：

<figure><picture><source srcset="../../.gitbook/assets/image (6) (1).png" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""></picture><figcaption></figcaption></figure>

```tsx
import { getPosts } from '@/lib/posts'
import { Post } from '@/ui/post'
 
export default async function Page() {
  const posts = await getPosts()
 
  return (
    <ul>
      {posts.map((post) => (
        <Post key={post.id} post={post} />
      ))}
    </ul>
  )
}
```

您可以继续嵌套文件夹以创建嵌套路由。例如，要为特定的博客文章创建路由，请在 `blog` 内创建一个新的 `[slug]` 文件夹，并添加一个 `page` 文件：

<figure><picture><source srcset="../../.gitbook/assets/image (2) (1) (1).png" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""></picture><figcaption></figcaption></figure>

```tsx
function generateStaticParams() {}
 
export default function Page() {
  return <h1>Hello, Blog Post Page!</h1>
}
```

> 需要知道的是：
>
> 将文件夹名称用方括号括起来（例如 `[slug]` ）会创建一个特殊的动态路由段，用于从数据生成多个页面。这对博客文章、产品页等非常有用。

### 嵌套布局

默认情况下，文件夹层次结构中的布局也是嵌套的，这意味着它们通过其 `children` 属性包装子布局。您可以通过在特定路由段（文件夹）中添加 `layout` 来嵌套布局。

例如，要为 `/blog` 路由创建布局，请在 `blog` 文件夹中添加一个新的 `layout` 文件。

<figure><picture><source srcset="../../.gitbook/assets/image (3) (1) (1).png" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""></picture><figcaption></figcaption></figure>

```tsx
export default function BlogLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return <section>{children}</section>
}
```

如果将上述两个布局结合起来，根布局 ( `app/layout.js` ) 将包裹博客布局 ( `app/blog/layout.js` )，而博客布局将包裹博客 ( `app/blog/page.js` ) 和博客文章页面 ( `app/blog/[slug]/page.js` )。

### 页面之间的链接

您可以使用 `<Link>` 组件在路由之间进行导航。 `<Link>` 是一个内置的 Next.js 组件，它扩展了 HTML `<a>` 标签以提供预取和客户端导航。

例如，要生成博客文章列表，从 `next/link` 导入 `<Link>` 并将 `href` 属性传递给组件：

```tsx
import Link from 'next/link'
 
export default async function Post({ post }) {
  const posts = await getPosts()
 
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.slug}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```

`<Link>` 是在您的 Next.js 应用程序中在路由之间导航的主要和推荐方式。然而，您也可以使用 `useRouter` 钩子进行更高级的导航。

### API参考

通过阅读 API 参考，了解本页面提到的功能。

