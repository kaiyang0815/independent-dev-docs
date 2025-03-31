# 懒加载

在 Next.js 中，懒加载通过减少渲染路由所需的 JavaScript 数量来帮助提高应用程序的初始加载性能。

它允许您推迟加载客户端组件和导入的库，并仅在需要时将它们包含在客户端捆绑中。例如，您可能希望推迟加载一个模态框，直到用户点击打开它。

您可以在 Next.js 中实现懒加载的两种方式：

1. 使用动态导入与 `next/dynamic`
2. 与 Suspense 一起使用 `React.lazy()`

默认情况下，服务器组件会自动进行代码拆分，您可以使用流式传输逐步将 UI 的部分从服务器发送到客户端。懒加载适用于客户端组件。

### `next/dynamic`

`next/dynamic` 是 `React.lazy()` 和 Suspense 的组合。它在 `app` 和 `pages` 目录中表现相同，以便进行增量迁移。

### 示例

#### 导入客户端组件

{% code title="app/page.js" %}
```javascript
'use client'
 
import { useState } from 'react'
import dynamic from 'next/dynamic'
 
// 客户端组件：
const ComponentA = dynamic(() => import('../components/A'))
const ComponentB = dynamic(() => import('../components/B'))
const ComponentC = dynamic(() => import('../components/C'), { ssr: false })
 
export default function ClientComponentExample() {
  const [showMore, setShowMore] = useState(false)
 
  return (
    <div>
      {/* Load immediately, but in a separate client bundle */}
      <ComponentA />
 
      {/* Load on demand, only when/if the condition is met */}
      {showMore && <ComponentB />}
      <button onClick={() => setShowMore(!showMore)}>Toggle</button>
 
      {/* Load only on the client side */}
      <ComponentC />
    </div>
  )
}
```
{% endcode %}

> **注意**：当服务器组件动态导入客户端组件时，当前不支持自动代码分割。

#### 跳过 SSR

当使用 `React.lazy()` 和 Suspense 时，客户端组件将默认进行预渲染 (SSR)。

> 注意： `ssr: false` 选项仅适用于客户端组件，将其移入客户端组件以确保客户端代码分割正常工作。

如果您想禁用客户端组件的预渲染，可以使用 `ssr` 选项设置为 `false` ：

```typescript
const ComponentC = dynamic(() => import('../components/C'), { ssr: false })ype
```

#### 导入服务器组件

如果动态导入一个服务器组件，只有服务器组件的子客户端组件会被延迟加载 - 而不是服务器组件本身。当在服务器组件中使用时，它还将帮助预加载静态资产，如 CSS。

{% code title="app/page.js" %}
```javascript
import dynamic from 'next/dynamic'
 
// 服务器组件：
const ServerComponent = dynamic(() => import('../components/ServerComponent'))
 
export default function ServerComponentExample() {
  return (
    <div>
      <ServerComponent />
    </div>
  )
}
```
{% endcode %}

> **注意**： `ssr: false` 选项在服务器组件中不受支持。如果您尝试在服务器组件中使用它，将会看到一个错误。 `ssr: false` 在服务器组件中与 `next/dynamic` 不允许一起使用。请将其移动到客户端组件中。

#### 加载外部库

外部库可以使用 `import()` 函数按需加载。此示例使用外部库 `fuse.js` 进行模糊搜索。该模块仅在用户输入搜索内容后在客户端加载。

{% code title="app/page.jsx" %}
```jsx
'use client'
 
import { useState } from 'react'
 
const names = ['Tim', 'Joe', 'Bel', 'Lee']
 
export default function Page() {
  const [results, setResults] = useState()
 
  return (
    <div>
      <input
        type="text"
        placeholder="Search"
        onChange={async (e) => {
          const { value } = e.currentTarget
          // Dynamically load fuse.js
          const Fuse = (await import('fuse.js')).default
          const fuse = new Fuse(names)
 
          setResults(fuse.search(value))
        }}
      />
      <pre>Results: {JSON.stringify(results, null, 2)}</pre>
    </div>
  )
}
```
{% endcode %}

#### 添加自定义加载组件

{% code title="app/page.jsx" %}
```jsx
'use client'
 
import dynamic from 'next/dynamic'
 
const WithCustomLoading = dynamic(
  () => import('../components/WithCustomLoading'),
  {
    loading: () => <p>Loading...</p>,
  }
)
 
export default function Page() {
  return (
    <div>
      {/* The loading component will be rendered while  <WithCustomLoading/> is loading */}
      <WithCustomLoading />
    </div>
  )
}
```
{% endcode %}

#### 导入命名导出

要动态导入命名导出，可以从 `import()` 函数返回的 Promise 中返回它：

{% code title="components/hello.js" %}
```jsx
'use client'
 
export function Hello() {
  return <p>Hello!</p>
}
```
{% endcode %}

{% code title="app/page.js" %}
```jsx
import dynamic from 'next/dynamic'
 
const ClientComponent = dynamic(() =>
  import('../components/hello').then((mod) => mod.Hello)
)
```
{% endcode %}

