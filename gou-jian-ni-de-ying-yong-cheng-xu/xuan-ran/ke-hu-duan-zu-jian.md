---
description: >-
  客户端组件允许您编写在服务器上预渲染的交互式 UI，并可以使用客户端 JavaScript 在浏览器中运行。
  本页面将介绍客户端组件的工作原理、它们是如何渲染的，以及您何时可能使用它们。
---

# 客户端组件

### 客户端渲染的好处

在客户端进行渲染工作有几个好处，包括：

* 交互性：客户端组件可以使用状态、效果和事件监听器，这意味着它们可以为用户提供即时反馈并更新 UI。
* 浏览器 API：客户端组件可以访问浏览器 API，例如 [geolocation](https://developer.mozilla.org/docs/Web/API/Geolocation_API) 或 [localStorage](https://developer.mozilla.org/docs/Web/API/Window/localStorage)。

### 在 Next.js 中使用客户端组件

要使用客户端组件，您可以在文件顶部的导入之前添加 React `"use client"` 指令。

`"use client"` 用于声明服务器和客户端组件模块之间的边界。这意味着通过在文件中定义 `"use client"` ，所有导入到其中的其他模块，包括子组件，都被视为客户端包的一部分。

{% code title="app/counter.tsx" %}
```tsx
'use client'
 
import { useState } from 'react'
 
export default function Counter() {
  const [count, setCount] = useState(0)
 
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
}
```
{% endcode %}

下图显示，在嵌套组件 ( `toggle.js` ) 中使用 `onClick` 和 `useState` 将导致错误，如果未定义 `"use client"` 指令。这是因为默认情况下，App Router 中的所有组件都是服务器组件，这些 API 不可用。通过在 `toggle.js` 中定义 `"use client"` 指令，您可以告诉 React 进入这些 API 可用的客户端边界。

<figure><picture><source srcset="../../.gitbook/assets/image (32).png" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/image (31).png" alt=""></picture><figcaption></figcaption></figure>

> **定义多个 `use client` 入口点：**
>
> 您可以在 React 组件树中定义多个 "use client" 入口点。这使您能够将应用程序拆分为多个客户端包。
>
> 然而， `"use client"` 不需要在每个需要在客户端呈现的组件中定义。一旦您定义了边界，所有子组件和导入到其中的模块都被视为客户端包的一部分。

### 客户端组件是如何渲染的？

在 Next.js 中，客户端组件的渲染方式取决于请求是否是完整页面加载的一部分（首次访问您的应用程序或由浏览器刷新触发的页面重新加载）或后续导航。

#### 完整页面加载 <a href="#full-page-load" id="full-page-load"></a>

为了优化初始页面加载，Next.js 将使用 React 的 API 在服务器上为客户端和服务器组件渲染静态 HTML 预览。这意味着，当用户第一次访问您的应用程序时，他们将立即看到页面的内容，而无需等待客户端下载、解析和执行客户端组件的 JavaScript 包。

在服务器上：

1. React 将服务器组件渲染为一种特殊的数据格式，称为 React 服务器组件有效负载 (RSC Payload)，其中包括对客户端组件的引用。
2. Next.js 使用 RSC Payload 和客户端组件 JavaScript 指令在服务器上渲染路由的 **HTML**。

然后，在客户端：

1. 该 HTML 用于立即显示路由的快速非交互式初始预览。
2. React Server Components Payload 用于调和 Client 和 Server Component 树，更新 DOM。
3. JavaScript 指令用于为客户端组件注水，使其 UI 变得可交互。

> **什么是水合？**
>
> 水合是将事件监听器附加到 DOM 的过程，以使静态 HTML 具有交互性。在幕后，水合是通过 [`hydrateRoot`](https://react.dev/reference/react-dom/client/hydrateRoot) React API 完成的。

#### 后续导航

在后续导航中，客户端组件完全在客户端渲染，而不使用服务器渲染的 HTML。

这意味着客户端组件的 JavaScript 包被下载和解析。一旦包准备好，React 将使用 RSC Payload 来调和客户端和服务器组件树，并更新 DOM。

### 返回服务器环境

有时，在声明了 `"use client"` 边界后，您可能想返回服务器环境。例如，您可能希望减小客户端包的大小，在服务器上获取数据，或使用仅在服务器上可用的 API。

您可以通过交错客户端和服务器组件以及服务器操作来保留在服务器上的代码，即使它在理论上嵌套在客户端组件内部。有关更多信息，请参见组合模式页面。
