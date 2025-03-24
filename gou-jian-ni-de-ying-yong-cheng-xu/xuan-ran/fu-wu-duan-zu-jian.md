# 服务端组件

React 服务端组件允许您编写可以在服务器上渲染并可选地缓存的 UI。在 Next.js 中，渲染工作进一步按路由段拆分，以启用流式传输和部分渲染，并且有三种不同的服务器渲染策略：

* [静态渲染](https://nextjs.org/docs/app/building-your-application/rendering/server-components#static-rendering-default)
* [动态渲染](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-rendering)
* [流式传输](https://nextjs.org/docs/app/building-your-application/rendering/server-components#streaming)

本页面将介绍服务端组件的工作原理、何时使用它们以及不同的服务器渲染策略。

### 服务端组件的好处

在服务器上进行渲染工作有几个好处，包括：

* **数据获取**：服务器组件允许您将数据获取移至服务器，更接近您的数据源。这可以通过减少获取渲染所需数据的时间以及客户端需要发出的请求数量来提高性能。
* **安全性**：服务器组件允许您将敏感数据和逻辑保留在服务器上，例如令牌和 API 密钥，而无需将其暴露给客户端。
* **缓存**：通过在服务器上呈现，结果可以被缓存并在后续请求和用户之间重用。这可以通过减少每次请求的渲染和数据获取量来提高性能并降低成本。
* **性能**：服务器组件为您提供额外的工具，从基线优化性能。例如，如果您开始时是完全由客户端组件组成的应用程序，将 UI 中非交互式部分移至服务器组件可以减少所需的客户端 JavaScript 的数量。这对于拥有较慢互联网或较弱设备的用户是有利的，因为浏览器需要下载、解析和执行的客户端 JavaScript 较少。
* **初始页面加载和**[**首次内容绘制（FCP）**](https://web.dev/fcp/)：在服务器上，我们可以生成 HTML 以允许用户立即查看页面，而无需等待客户端下载、解析和执行所需的 JavaScript 来渲染页面。
* **搜索引擎优化和社交网络易共享性**：渲染后的 HTML 可以被搜索引擎机器人用于索引您的页面，并被社交网络机器人用于为您的页面生成社交卡片预览。
* **流式传输**：服务端组件允许您将渲染工作拆分成块，并在它们准备好时将其流式传输到客户端。这使得用户能够更早地看到页面的部分内容，而无需等待整个页面在服务器上渲染完成。

### 在 Next.js 中使用服务端组件

默认情况下，Next.js 使用服务端组件。这使您能够自动实现服务器渲染，无需额外配置，您可以在需要时选择使用客户端组件，参见客户端组件。

### 服务端组件是如何渲染的？

在服务器上，Next.js 使用 React 的 API 来协调渲染。渲染工作被分成多个块：按单个路由段和 [Suspense 边界](https://react.dev/reference/react/Suspense)。

每个块分两步渲染：

* React 将服务器组件渲染为一种特殊的数据格式，称为 **React 服务器组件负载（RSC Payload）**。
* Next.js 使用 RSC Payload 和客户端组件 JavaScript 指令在服务器上渲染 HTML。

然后，在客户端：

1. HTML 用于立即显示路由的快速非交互式预览 - 这仅用于初始页面加载。
2. React Server Components Payload 用于调和 Client 和 Server Component 树，更新 DOM。
3. JavaScript 指令用于为客户端组件提供水合，并使应用程序具有交互性。

> [**什么是 React Server Component Payload (RSC)？**](https://nextjs.org/docs/app/building-your-application/rendering/server-components#what-is-the-react-server-component-payload-rsc)
>
> RSC Payload 是渲染的 React 服务器组件树的紧凑二进制表示。它被 React 在客户端用于更新浏览器的 DOM。RSC Payload 包含：
>
> * 服务器组件的渲染结果
> * 用于标记客户端组件应该渲染的位置以及对其 JavaScript 文件的引用
> * 从服务器组件传递到客户端组件的任何 props

### 服务端组件策略

服务器渲染有三个子集：静态、动态和流式。

#### 静态渲染（默认）

通过静态渲染，路由在构建时或在数据重新验证后后台渲染。结果被缓存并可以推送到内容分发网络（CDN）。这种优化允许您在用户和服务器请求之间共享渲染工作的结果。

静态渲染在路由的数据不针对用户且可以在构建时已知时非常有用，例如静态博客文章或产品页面。

#### 动态渲染

通过动态渲染，路由在**请求时**为每个用户渲染。

动态渲染在路由具有个性化数据或在请求时才能知道的信息（如 Cookies 或 URL 的搜索参数）时非常有用。

> **带缓存数据的动态路由**
>
> 在大多数网站中，路由既不是完全静态的，也不是完全动态的 - 这是一个光谱。例如，您可以拥有一个电子商务页面，该页面使用在一定时间间隔内重新验证的缓存产品数据，但也有未缓存的个性化客户数据。
>
> 在 Next.js 中，您可以拥有动态渲染的路由，这些路由同时具有缓存和未缓存的数据。这是因为 RSC Payload 和数据是单独缓存的。这使您能够选择动态渲染，而无需担心在请求时获取所有数据的性能影响。
>
> 了解更多关于全路由缓存和数据缓存的信息。

[**切换到动态渲染**](https://nextjs.org/docs/app/building-your-application/rendering/server-components#switching-to-dynamic-rendering)

在渲染过程中，如果发现了[动态 API](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-apis) 或 `{ cache: 'no-store' }` 的 [fetch](https://nextjs.org/docs/app/api-reference/functions/fetch) 选项，Next.js 将切换到动态渲染整个路由。下表总结了动态 API 和数据缓存如何影响路由的静态或动态渲染：

| 动态 API | 数据  | 路由   |
| ------ | --- | ---- |
| 否      | 缓存  | 静态渲染 |
| 是      | 缓存  | 动态渲染 |
| 否      | 未缓存 | 动态渲染 |
| 是      | 未缓存 | 动态渲染 |

在上表中，要使路由完全静态，所有数据必须被缓存。然而，您可以拥有一个动态渲染的路由，它同时使用缓存和未缓存的数据获取。

作为开发者，您无需在静态渲染和动态渲染之间进行选择，因为 Next.js 会根据使用的功能和 API 自动为每个路由选择最佳渲染策略。相反，您可以选择何时缓存或重新验证特定数据，并且您可以选择流式传输 UI 的部分内容。

动态 API 依赖于在请求时才能知道的信息（而不是在预渲染时提前知道）。使用任何这些 API 都会表明开发者的意图，并将在请求时将整个路由选择为动态渲染。这些 API 包括：

* [`cookies`](https://nextjs.org/docs/app/api-reference/functions/cookies)
* [`headers`](https://nextjs.org/docs/app/api-reference/functions/headers)
* [`connection`](https://nextjs.org/docs/app/api-reference/functions/connection)
* [`draftMode`](https://nextjs.org/docs/app/api-reference/functions/draft-mode)
* [`searchParams` prop](https://nextjs.org/docs/app/api-reference/file-conventions/page#searchparams-optional)
* [`unstable_noStore`](https://nextjs.org/docs/app/api-reference/functions/unstable_noStore)

#### 流式传输

<figure><picture><source srcset="../../.gitbook/assets/image (4) (1).png" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""></picture><figcaption></figcaption></figure>

流式传输使您能够从服务器逐步渲染用户界面。工作被分成块，并在准备好时流式传输到客户端。这使得用户可以立即看到页面的部分内容，而无需等待整个内容渲染完成。

<figure><picture><source srcset="../../.gitbook/assets/image (5) (1).png" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""></picture><figcaption></figcaption></figure>

流式传输默认内置于 Next.js 应用路由中。这有助于提高初始页面加载性能，以及依赖于较慢数据获取的 UI，这些数据获取会阻塞整个路由的渲染。例如，产品页面上的评论。

您可以使用 `loading.js` 开始流式传输路由段，并使用 React Suspense 组件。有关更多信息，请参阅加载 UI 和流式传输部分。

### 下一步

了解 Next.js 如何缓存数据和静态渲染的结果。
