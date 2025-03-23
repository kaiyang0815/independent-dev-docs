# CSS-in-JS

> **警告**：在使用 CSS-in-JS 与较新的 React 特性（如服务器组件和流式传输）时，需要库的作者支持最新版本的 React，包括并发渲染。

以下库在 `app` 目录中的客户端组件中受支持（按字母顺序）：

* [`ant-design`](https://ant.design/docs/react/use-with-next#using-app-router)
* [`chakra-ui`](https://chakra-ui.com/getting-started/nextjs-app-guide)
* [`@fluentui/react-components`](https://react.fluentui.dev/?path=/docs/concepts-developer-server-side-rendering-next-js-appdir-setup--page)
* [`kuma-ui`](https://kuma-ui.com/)
* [`@mui/material`](https://mui.com/material-ui/guides/next-js-app-router/)
* [`@mui/joy`](https://mui.com/joy-ui/integrations/next-js-app-router/)
* [`pandacss`](https://panda-css.com/)
* [`styled-jsx`](https://nextjs.org/docs/app/building-your-application/styling/css-in-js#styled-jsx)
* [`styled-components`](https://nextjs.org/docs/app/building-your-application/styling/css-in-js#styled-components)
* [`stylex`](https://stylexjs.com/)
* [`tamagui`](https://tamagui.dev/docs/guides/next-js#server-components)
* [`tss-react`](https://tss-react.dev/)
* [`vanilla-extract`](https://vanilla-extract.style/)

目前正在努力支持的有：

* [`emotion`](https://github.com/emotion-js/emotion/issues/2928)

> **注意**：我们正在测试不同的 CSS-in-JS 库，并将为支持 React 18 特性和/或 `app` 目录的库添加更多示例。

如果您想为服务器组件添加样式，我们建议使用 CSS 模块或其他输出 CSS 文件的解决方案，如 PostCSS 或 Tailwind CSS。

### 在应用中配置 CSS-in-JS

配置 CSS-in-JS 是一个三步的选择过程，涉及：

* 一个**样式注册表**，用于收集渲染中的所有 CSS 规则。
* 新的 `useServerInsertedHTML` 钩子，用于在任何可能使用它们的内容之前注入规则。
* 一个客户端组件，在初始服务器端渲染期间用样式注册表包装您的应用程序。

#### `styled-jsx`

&#x20;在客户端组件中使用 `styled-jsx` 需要使用 `v5.1.0` 。首先，创建一个新的注册表：

{% code title="app/registry.tsx" %}
```tsx
'use client'
 
import React, { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { StyleRegistry, createStyleRegistry } from 'styled-jsx'
 
export default function StyledJsxRegistry({
  children,
}: {
  children: React.ReactNode
}) {
  // Only create stylesheet once with lazy initial state
  // x-ref: https://reactjs.org/docs/hooks-reference.html#lazy-initial-state
  const [jsxStyleRegistry] = useState(() => createStyleRegistry())
 
  useServerInsertedHTML(() => {
    const styles = jsxStyleRegistry.styles()
    jsxStyleRegistry.flush()
    return <>{styles}</>
  })
 
  return <StyleRegistry registry={jsxStyleRegistry}>{children}</StyleRegistry>
}
```
{% endcode %}

然后，用注册表包装你的根布局：

{% code title="app/layout.tsx" %}
```tsx
import StyledJsxRegistry from './registry'
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        <StyledJsxRegistry>{children}</StyledJsxRegistry>
      </body>
    </html>
  )
}
```
{% endcode %}

在[这里](https://github.com/vercel/app-playground/tree/main/app/styling/styled-jsx)查看示例。

#### 样式组件

以下是如何配置 `styled-components@6` 或更高版本的示例：

首先，在 `next.config.js` 中启用 styled-components。

{% code title="next.config.js" %}
```javascript
module.exports = {
  compiler: {
    styledComponents: true,
  },
}
```
{% endcode %}

然后，使用 `styled-components` API 创建一个全局注册组件，以收集在渲染过程中生成的所有 CSS 样式规则，并返回这些规则的函数。接着，使用 `useServerInsertedHTML` 钩子将注册中收集的样式注入到根布局中的 `<head>` HTML 标签中。

{% code title="lib/registry.tsx" %}
```tsx
'use client'
 
import React, { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { ServerStyleSheet, StyleSheetManager } from 'styled-components'
 
export default function StyledComponentsRegistry({
  children,
}: {
  children: React.ReactNode
}) {
  // Only create stylesheet once with lazy initial state
  // x-ref: https://reactjs.org/docs/hooks-reference.html#lazy-initial-state
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet())
 
  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement()
    styledComponentsStyleSheet.instance.clearTag()
    return <>{styles}</>
  })
 
  if (typeof window !== 'undefined') return <>{children}</>
 
  return (
    <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>
      {children}
    </StyleSheetManager>
  )
}
```
{% endcode %}

用样式注册组件包裹根布局的 `children` ：

{% code title="app/layout.tsx" %}
```tsx
import StyledComponentsRegistry from './lib/registry'
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        <StyledComponentsRegistry>{children}</StyledComponentsRegistry>
      </body>
    </html>
  )
}
```
{% endcode %}

在[这里](https://github.com/vercel/app-playground/tree/main/app/styling/styled-components)查看示例。

> 注意：
>
> * 在服务器渲染过程中，样式将被提取到一个全局注册表中，并刷新到您的 HTML 的 `<head>` 中。这确保样式规则在任何可能使用它们的内容之前被放置。在未来，我们可能会使用即将推出的 React 功能来确定在哪里注入样式。
> * 在流式传输过程中，将收集每个块的样式并附加到现有样式中。在客户端水合完成后， `styled-components` 将像往常一样接管并注入进一步的动态样式。
> * 我们特别在树的顶层使用客户端组件作为样式注册表，因为这样提取 CSS 规则更高效。这避免了在后续服务器渲染时重新生成样式，并防止它们被发送到服务器组件负载中。
> * 对于需要配置 styled-components 编译的单独属性的高级用例，您可以查看我们的 Next.js styled-components API 参考以了解更多信息。
