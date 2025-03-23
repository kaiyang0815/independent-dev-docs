# CSS

Next.js 支持多种处理 CSS 的方式，包括：

* [CSS 模块](https://app.immersivetranslate.com/markdown/#css-modules)
* [全局样式](https://app.immersivetranslate.com/markdown/#global-styles)
* [外部样式表](https://app.immersivetranslate.com/markdown/#external-stylesheets)

### CSS 模块

Next.js 内置对使用 `.module.css` 扩展名的 CSS 模块的支持。

CSS 模块通过自动创建唯一的类名来局部作用域 CSS。这使得您可以在不同的文件中使用相同的类名，而不必担心冲突。这种行为使 CSS 模块成为包含组件级 CSS 的理想方式。

#### 示例

CSS 模块可以导入到 `app` 目录中的任何文件：

```tsx
import styles from './styles.module.css'

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return <section className={styles.dashboard}>{children}</section>
}
```

```css
.dashboard {
  padding: 24px;
}
```

CSS 模块 **仅对扩展名为 `.module.css` 和 `.module.sass` 的文件启用** 。

在生产环境中，所有 CSS 模块文件将自动合并为 **多个压缩和代码分割** `.css` 文件。这些 `.css` 文件代表了您应用程序中的热执行路径，确保加载的 CSS 最小化，以便您的应用程序能够快速渲染。

### 全局样式

全局样式可以在 `app` 目录中的任何布局、页面或组件中导入。

> **注意**：
>
> * 这与 `pages` 目录不同，在那里您只能在 `_app.js` 文件中导入全局样式。
> * Next.js 不支持使用全局样式，除非它们实际上是全局的，意味着它们可以应用于所有页面，并且可以在应用程序的生命周期内存在。这是因为 Next.js 利用 React 内置的样式表支持与 Suspense 集成。当前，这种内置支持不会在您在不同路由之间导航时删除样式表。因此，我们建议使用 CSS Modules 而不是全局样式。

例如，考虑一个名为 `app/global.css` 的样式表：

{% code title="app/global.css" %}
```css
body {
  padding: 20px 20px 60px;
  max-width: 680px;
  margin: 0 auto;
}
```
{% endcode %}

在根布局(`app/layout.js`)中，导入 `global.css` 样式表，以将样式应用于您应用中的每个路由：

{% code title="app/layout.tsx" %}
```tsx
// These styles apply to every route in the application
import './global.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```
{% endcode %}

### 外部样式表

由外部包发布的样式表可以在 `app` 目录中的任何地方导入，包括共置组件：

```tsx
import 'bootstrap/dist/css/bootstrap.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className="container">{children}</body>
    </html>
  )
}
```

> **注意**：
>
> 外部样式表必须直接从 npm 包导入或下载并与您的代码库放在一起。您不能使用 `<link rel="stylesheet" />`。

### 排序与合并

Next.js 在生产构建期间通过自动分块（合并）样式表来优化 CSS。CSS 的顺序是 _由您在应用程序代码中导入样式表的顺序决定的_ 。

例如，`base-button.module.css` 将在 `page.module.css` 之前排序，因为 `<BaseButton>` 首先在 `<Page>` 中被导入：

{% code title="base-button.tsx" %}
```tsx
import styles from './base-button.module.css'

export function BaseButton() {
  return <button className={styles.primary} />
}
```
{% endcode %}

{% code title="page.ts" %}
```tsx
import { BaseButton } from './base-button'
import styles from './page.module.css'

export default function Page() {
  return <BaseButton className={styles.primary} />
}
```
{% endcode %}

为了保持一个可预测的顺序，我们推荐以下做法：

* 仅在单个 JS/TS 文件中导入 CSS 文件。
  * 如果使用全局类名，请在同一文件中按希望的应用顺序导入全局样式。
* 优先使用 CSS 模块而不是全局样式。
  * 为您的 CSS 模块使用一致的命名约定。例如，使用 `<name>.module.css` 而不是 `<name>.tsx`。
* 将共享样式提取到一个单独的共享组件中。
* 如果使用 [Tailwind](https://app.immersivetranslate.com/docs/app/building-your-application/styling/tailwind-css)，请在文件顶部导入样式表，最好在[根布局](https://app.immersivetranslate.com/docs/app/building-your-application/routing/layouts-and-templates#root-layout-required)中。
* 关闭任何自动排序导入的代码检查工具/格式化工具（例如，ESLint 的 [`sort-import`](https://eslint.org/docs/latest/rules/sort-imports)）。这可能会无意中影响您的 CSS，因为 CSS 导入顺&#x5E8F;_&#x5F88;重要_ 。

> **注意**：
>
> * CSS 排序在开发模式下可能表现不同，始终确保检查构建（`next build`）以验证最终的 CSS 顺序。
> * 您可以在 `next.config.js` 中使用 [`cssChunking`](https://app.immersivetranslate.com/docs/app/api-reference/config/next-config-js/cssChunking) 选项来控制 CSS 的分块方式。

### 附加功能

Next.js 包含额外的功能，以改善添加样式的创作体验：

* 在本地使用 `next dev` 运行时，本地样式表（无论是全局样式还是 CSS 模块）将利用 [快速刷新 ](https://app.immersivetranslate.com/docs/architecture/fast-refresh)来即时反映保存编辑后的更改。
* 在使用 `next build` 进行生产构建时，CSS 文件将被打包成更少的压缩 `.css` 文件，以减少检索样式所需的网络请求数量。
* 如果您禁用 JavaScript，样式仍将在生产构建中加载（`next start`）。然而，JavaScript 仍然是 `next dev` 所必需的，以启用[快速刷新](https://app.immersivetranslate.com/docs/architecture/fast-refresh)
