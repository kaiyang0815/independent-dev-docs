# 组合模式

在构建 React 应用程序时，您需要考虑应用程序的哪些部分应该在服务器或客户端上渲染。 本页面涵盖了使用服务器和客户端组件时的一些推荐组合模式。

### 何时使用服务端和客户端组件

以下是服务器和客户端组件的不同用例的快速总结：

<table><thead><tr><th width="504.78515625">你需要做什么？</th><th width="111.78515625" align="center">服务端组件</th><th align="center">客户端组件</th></tr></thead><tbody><tr><td>获取数据</td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="2705">✅</span></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="274c">❌</span></td></tr><tr><td>直接访问后端资源</td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="2705">✅</span></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="274c">❌</span></td></tr><tr><td>将敏感信息保留在服务器上（访问令牌，API 密钥等）</td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="2705">✅</span></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="274c">❌</span></td></tr><tr><td>将大型依赖项保留在服务器上 / 减少客户端 JavaScript</td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="2705">✅</span></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="274c">❌</span></td></tr><tr><td>添加交互性和事件监听器（ <code>onClick()</code> , <code>onChange()</code> , 等）</td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="274c">❌</span></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="2705">✅</span></td></tr><tr><td>使用状态和生命周期效果 ( <code>useState()</code> , <code>useReducer()</code> , <code>useEffect()</code> , 等等)</td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="274c">❌</span></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="2705">✅</span></td></tr><tr><td>使用仅浏览器的 API</td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="274c">❌</span></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="2705">✅</span></td></tr><tr><td>使用依赖于状态、效果或仅浏览器的 API 的自定义钩子</td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="274c">❌</span></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="2705">✅</span></td></tr><tr><td>使用 <a href="https://react.dev/reference/react/Component">React 类组件</a></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="274c">❌</span></td><td align="center"><span data-gb-custom-inline data-tag="emoji" data-code="2705">✅</span></td></tr></tbody></table>

### 服务端组件模式

在选择客户端渲染之前，您可能希望在服务器上进行一些工作，例如获取数据或访问您的数据库或后端服务。

在使用服务器组件时的一些常见模式：

#### 在组件之间共享数据

在服务器上获取数据时，可能会有需要在不同组件之间共享数据的情况。例如，您可能有一个布局和一个依赖于相同数据的页面。

您可以使用 `fetch` 或 React 的 `cache` 函数在需要数据的组件中获取相同的数据，而无需担心对相同数据进行重复请求，而不是使用 [React Context](https://react.dev/learn/passing-data-deeply-with-context)（在服务器上不可用）或将数据作为 props 传递。这是因为 React 扩展了 `fetch` 以自动记忆数据请求，并且可以在 `fetch` 不可用时使用 `cache` 函数。

查看此模式的[示例](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching#reusing-data-across-multiple-functions)。

#### 将仅服务端可用的代码排除在客户端环境之外

由于 JavaScript 模块可以在服务器和客户端组件模块之间共享，因此原本只打算在服务器上运行的代码可能会悄悄地进入客户端。

例如，考虑以下数据获取函数：

{% code title="lib/data.ts" %}
```typescript
export async function getData() {
  const res = await fetch('https://external-service.com/data', {
    headers: {
      authorization: process.env.API_KEY,
    },
  })
 
  return res.json()
}
```
{% endcode %}

乍一看， `getData` 似乎同时在服务器和客户端上工作。然而，这个函数包含一个 `API_KEY` ，其编写意图是它只会在服务器上执行。

由于环境变量 `API_KEY` 没有以 `NEXT_PUBLIC` 为前缀，它是一个只能在服务器上访问的私有变量。为了防止您的环境变量泄露给客户端，Next.js 将私有环境变量替换为空字符串。

因此，即使 `getData()` 可以在客户端导入和执行，它也不会按预期工作。虽然将变量设为公共可以使函数在客户端正常工作，但您可能不想向客户端暴露敏感信息。

为防止这种意外的客户端使用服务器代码，我们可以使用 `server-only` 包来给其他开发者在构建时提供错误，如果他们不小心将其中一个模块导入到客户端组件中。

要使用 `server-only` ，首先安装该软件包：

```bash
npm install server-only
```

然后将该包导入任何包含仅限服务器代码的模块中：

{% code title="lib/data.js" %}
```javascript
import 'server-only'
 
export async function getData() {
  const res = await fetch('https://external-service.com/data', {
    headers: {
      authorization: process.env.API_KEY,
    },
  })
 
  return res.json()
}
```
{% endcode %}

现在，任何导入 `getData()` 的客户端组件都会收到一个构建时错误，说明该模块只能在服务器上使用。

对应的包 `client-only` 可用于标记包含仅客户端代码的模块 – 例如，访问 `window` 对象的代码。

#### 使用第三方包和提供者

由于 Server Components 是一个新的 React 特性，生态系统中的第三方包和提供者刚刚开始将 `"use client"` 指令添加到使用仅客户端功能的组件中，例如 `useState` 、 `useEffect` 和 `createContext` 。

今天，许多来自 `npm` 包的仅使用客户端功能的组件尚未具有该指令。这些第三方组件在客户端组件中将按预期工作，因为它们具有 `"use client"`指令，但在服务器组件中将无法工作。

例如，假设您已安装了假设的 `acme-carousel` 包，该包具有 `<Carousel />` 组件。该组件使用 `useState` ，但尚未具有 `"use client"` 指令。

如果您在客户端组件中使用 `<Carousel />` ，它将按预期工作：

{% code title="app/gallery.tsx" %}
```tsx
'use client'
 
import { useState } from 'react'
import { Carousel } from 'acme-carousel'
 
export default function Gallery() {
  const [isOpen, setIsOpen] = useState(false)
 
  return (
    <div>
      <button onClick={() => setIsOpen(true)}>View pictures</button>
 
      {/* Works, since Carousel is used within a Client Component */}
      {isOpen && <Carousel />}
    </div>
  )
}
```
{% endcode %}

但是，如果您尝试在服务器组件中直接使用它，您会看到一个错误：

{% code title="app/page.tsx" %}
```tsx
import { Carousel } from 'acme-carousel'
 
export default function Page() {
  return (
    <div>
      <p>View pictures</p>
 
      {/* Error: `useState` can not be used within Server Components */}
      <Carousel />
    </div>
  )
}
```
{% endcode %}

这是因为 Next.js 不知道 `<Carousel />` 正在使用仅限客户端的功能。

要解决此问题，您可以将依赖于仅客户端功能的第三方组件包装在您自己的客户端组件中：

{% code title="app/carousel.tsx" %}
```tsx
'use client'
 
import { Carousel } from 'acme-carousel'
 
export default Carousel
```
{% endcode %}

现在，您可以在服务器组件中直接使用 `<Carousel />` ：

{% code title="app/page.tsx" %}
```tsx
import Carousel from './carousel'
 
export default function Page() {
  return (
    <div>
      <p>View pictures</p>
 
      {/*  Works, since Carousel is a Client Component */}
      <Carousel />
    </div>
  )
}
```
{% endcode %}

我们不期望您需要包装大多数第三方组件，因为您可能会在客户端组件中使用它们。然而，有一个例外是提供者，因为它们依赖于 React 状态和上下文，并且通常在应用程序的根部需要。[了解更多关于第三方上下文提供者的信息](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns#using-context-providers)。

**使用上下文提供者**

上下文提供者通常在应用程序的根部渲染，以共享全局关注点，例如当前主题。由于 React 上下文不支持服务器组件，因此在应用程序根部尝试创建上下文将导致错误：

{% code title="app/layout.tsx" %}
```tsx
import { createContext } from 'react'
 
//  createContext is not supported in Server Components
export const ThemeContext = createContext({})
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
      </body>
    </html>
  )
}
```
{% endcode %}

要解决此问题，请在客户端组件中创建您的上下文并渲染其提供者：

{% code title="app/theme-provider.tsx" %}
```tsx
'use client'
 
import { createContext } from 'react'
 
export const ThemeContext = createContext({})
 
export default function ThemeProvider({
  children,
}: {
  children: React.ReactNode
}) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
}
```
{% endcode %}

您的服务器组件现在可以直接渲染您的提供者，因为它已被标记为客户端组件：

{% code title="app/layout.tsx" %}
```tsx
import ThemeProvider from './theme-provider'
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  )
}
```
{% endcode %}

在根部渲染的提供者使得您应用中的所有其他客户端组件都能够使用此上下文。

> 注意：你应该尽可能深地在树中渲染提供者——注意 `ThemeProvider` 仅包装 `{children}` 而不是整个 `<html>` 文档。这使得 Next.js 更容易优化你的服务器组件的静态部分。

**给库作者的建议**

以类似的方式，创建供其他开发者使用的包的库作者可以使用 `"use client"` 指令来标记其包的客户端入口点。这允许包的用户将包组件直接导入到他们的服务器组件中，而无需创建包装边界。

您可以通过在树中更深处使用 'use client' 来优化您的包，从而允许导入的模块成为服务器组件模块图的一部分。

值得注意的是，一些打包工具可能会剥离 `"use client"` 指令。您可以在 [React Wrap Balancer](https://github.com/shuding/react-wrap-balancer/blob/main/tsup.config.ts#L10-L13) 和 [Vercel Analytics](https://github.com/vercel/analytics/blob/main/packages/web/tsup.config.js#L26-L30) 存储库中找到如何配置 esbuild 以包含 `"use client"` 指令的示例。

### 客户端组件

#### 将客户端组件移动到组件树下

为了减少客户端 JavaScript 包的大小，我们建议将客户端组件移动到您的组件树下。

例如，您可能有一个具有静态元素（如 logo、链接等）和一个使用状态的交互式搜索栏的布局。

将整个布局作为客户端组件的做法改为将交互逻辑移动到客户端组件（例如 `<SearchBar />` ），并将布局保持为服务器组件。这意味着您不必将布局的所有组件 JavaScript 发送到客户端。

{% code title="app/layout.tsx" %}
```tsx
// SearchBar is a Client Component
import SearchBar from './searchbar'
// Logo is a Server Component
import Logo from './logo'
 
// Layout is a Server Component by default
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Logo />
        <SearchBar />
      </nav>
      <main>{children}</main>
    </>
  )
}
```
{% endcode %}

#### 从服务器传递 props 到客户端组件（序列化）

如果您在服务器组件中提取数据，您可能想要将数据作为 props 传递给客户端组件。 从服务器传递到客户端组件的 props 需要可以被 React 序列化。

如果您的客户端组件依赖于不可序列化的数据，您可以使用第三方库在客户端获取数据，或者在服务器上通过路由处理程序获取数据。

### 交错服务器和客户端组件

在交错客户端和服务器组件时，可能有助于将您的用户界面可视化为组件树。从根布局开始，这是一个服务器组件，然后您可以通过添加 `"use client"` 指令在客户端渲染某些组件子树。

在这些客户端子树中，您仍然可以嵌套服务器组件或调用服务器操作，但有一些事项需要注意：

* 在请求-响应生命周期中，您的代码从服务器移动到客户端。如果您需要在客户端访问服务器上的数据或资源，您将向服务器发出新的请求，而不是来回切换。
* 当向服务器发出新请求时，所有服务器组件首先被渲染，包括嵌套在客户端组件内部的那些。渲染结果（RSC Payload）将包含对客户端组件位置的引用。然后，在客户端，React 使用 RSC Payload 将服务器组件和客户端组件协调成一个单一的树。
* 由于客户端组件在服务器组件之后渲染，因此您无法将服务器组件导入客户端组件模块（因为这将需要向服务器发出新的请求）。相反，您可以将服务器组件作为 `props` 传递给客户端组件。请参阅下面的未支持模式和支持模式部分。

#### 不支持的模式：将服务器组件导入客户端组件

以下模式不受支持。您不能将服务器组件导入到客户端组件中：

{% code title="app/client-component.tsx" %}
```tsx
'use client'
 
// You cannot import a Server Component into a Client Component.
import ServerComponent from './Server-Component'
 
export default function ClientComponent({
  children,
}: {
  children: React.ReactNode
}) {
  const [count, setCount] = useState(0)
 
  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
 
      <ServerComponent />
    </>
  )
}
```
{% endcode %}

#### 支持的模式：将服务器组件作为属性传递给客户端组件

支持以下模式。您可以将服务器组件作为道具传递给客户端组件。

一个常见的模式是使用 React `children` 属性在您的客户端组件中创建一个“插槽”。

在下面的示例中， `<ClientComponent>` 接受一个 `children` 属性：

{% code title="app/client-component.tsx" %}
```jsx
'use client'
 
import { useState } from 'react'
 
export default function ClientComponent({
  children,
}: {
  children: React.ReactNode
}) {
  const [count, setCount] = useState(0)
 
  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      {children}
    </>
  )
}
```
{% endcode %}

`<ClientComponent>` 并不知道 `children` 最终会被一个服务器组件的结果填充。 `<ClientComponent>` 唯一的责任是决定 `children` 最终将被放置的位置。

在父 Server 组件中，您可以导入 `<ClientComponent>` 和 `<ServerComponent>` ，并将 `<ServerComponent>` 作为 `<ClientComponent>` 的子组件传递：

{% code title="app/page.tsx" %}
```tsx
// This pattern works:
// You can pass a Server Component as a child or prop of a
// Client Component.
import ClientComponent from './client-component'
import ServerComponent from './server-component'
 
// Pages in Next.js are Server Components by default
export default function Page() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  )
}
```
{% endcode %}

通过这种方法， `<ClientComponent>` 和 `<ServerComponent>` 被解耦，可以独立渲染。在这种情况下，子组件 `<ServerComponent>` 可以在服务器上渲染，远在客户端渲染 `<ClientComponent>` 之前。

> 注意：
>
> * “提升内容”模式已被用来避免在父组件重新渲染时重新渲染嵌套的子组件。
> * 您不限于 `children` 属性。您可以使用任何属性来传递 JSX。

