# 加载用户界面和流媒体

特殊文件 `loading.js` 帮助您使用 React Suspense 创建有意义的加载 UI。通过这种约定，您可以在路由段内容加载时立即显示来自服务器的加载状态。一旦渲染完成，新内容会自动替换。

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 即时记载状态

即时加载状态是导航时立即显示的后备 UI。您可以预渲染加载指示器，例如骨架和旋转器，或未来屏幕的一个小但有意义的部分，例如封面照片、标题等。这有助于用户理解应用程序正在响应，并提供更好的用户体验。

通过在文件夹中添加 `loading.js` 文件来创建加载状态。

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

```tsx
export default function Loading() {
  // You can add any UI inside Loading, including a Skeleton.
  return <LoadingSkeleton />
}
```

在同一文件夹中， `loading.js` 将嵌套在 `layout.js` 内部。它将自动包装 `page.js` 文件及其下面的任何子项在 `<Suspense>` 边界内。

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

> 需要知道的是：
>
> * 导航是即时的，即使使用以服务器为中心的路由。
> * 导航是可中断的，这意味着更改路由不需要等待路由的内容完全加载后再导航到另一个路由。
> * 在加载新的路由段时，共享布局仍然是互动的。

> 推荐：对于路由段（布局和页面）使用 `loading.js` 约定，因为 Next.js 会优化此功能。

### 使用 Suspense 进行流式处理

除了 `loading.js` 之外，您还可以手动为自己的 UI 组件创建 Suspense 边界。应用路由器支持使用 Suspense 进行流式处理。

> 需要知道的是：
>
> * 一些浏览器会缓冲流式响应。您可能在响应超过 1024 字节之前看不到流式响应。这通常只影响“你好，世界”应用程序，而不是真正的应用程序。

#### 什么是流式传输？

要了解流式传输在 React 和 Next.js 中是如何工作的，了解服务器端渲染（SSR）及其局限性是很有帮助的。

使用 SSR，用户在看到并与页面交互之前需要完成一系列步骤：

1. 首先，所有给定页面的数据在服务器上被获取。
2. 然后服务器渲染该页面的 HTML。
3. 页面所需的 HTML、CSS 和 JavaScript 被发送到客户端。
4. 使用生成的 HTML 和 CSS 显示一个非交互式用户界面。
5. 最后，React 为用户界面<mark style="color:blue;">提供水合</mark>，使其变得互动。

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

这些步骤是顺序和阻塞的，这意味着服务器只能在所有数据都被获取后才能渲染页面的 HTML。而在客户端，React 只能在页面中所有组件的代码下载完成后才能水合 UI。

使用 React 和 Next.js 的 SSR 通过尽快向用户显示非交互式页面来帮助提高感知加载性能。

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

然而，由于在页面显示给用户之前，所有数据获取都需要在服务器上完成，因此它仍然可能很慢。

流式传输允许您将页面的 HTML 分解为更小的块，并逐步将这些块从服务器发送到客户端。

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

这使得页面的某些部分能够更早地显示，而无需等待所有数据加载完毕后再渲染任何用户界面。

流式传输与 React 的组件模型非常契合，因为每个组件都可以视为一个块。优先级较高的组件（例如产品信息）或不依赖于数据的组件可以优先发送（例如布局），React 可以更早开始水合。优先级较低的组件（例如评论、相关产品）可以在其数据被获取后在同一服务器请求中发送。

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

流媒体特别有利于防止长时间的数据请求阻塞页面渲染，因为它可以减少首次字节时间（TTFB）和首次内容绘制时间（FCP）。它还帮助改善互动时间（TTI），尤其是在较慢的设备上。

#### 示例

`<Suspense>` 通过包装一个执行异步操作的组件（例如，获取数据），在操作进行时显示备用 UI（例如，骨架，加载动画），然后在操作完成后替换为您的组件。

```tsx
import { Suspense } from 'react'
import { PostFeed, Weather } from './Components'
 
export default function Posts() {
  return (
    <section>
      <Suspense fallback={<p>Loading feed...</p>}>
        <PostFeed />
      </Suspense>
      <Suspense fallback={<p>Loading weather...</p>}>
        <Weather />
      </Suspense>
    </section>
  )
}
```

通过使用 Suspense，您可以获得以下好处：

1. **流媒体服务器渲染** - 从服务器逐步渲染 HTML 到客户端。
2. **选择性水合** - React 根据用户交互优先考虑哪些组件首先变得可交互。

有关更多 Suspense 示例和用例，请参阅 React 文档。

#### SEO

* Next.js 将在 `generateMetadata` 内等待数据获取完成，然后再将用户界面流式传输到客户端。这保证了流式响应的第一部分包含 `<head>` 标签。
* 由于流式传输是服务器渲染的，因此不会影响 SEO。您可以使用 Google 的 Rich Results Test 工具查看您的页面在 Google 爬虫中的显示方式，并查看序列化的 HTML（源）。

#### 状态码

在流式传输时，将返回 `200` 状态码以表示请求成功。

服务器仍然可以通过流式内容本身向客户端传达错误或问题，例如，当使用 `redirect` 或 `notFound` 时。由于响应头已经发送给客户端，因此响应的状态码无法更新。这不影响 SEO。
