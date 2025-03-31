# 脚本

### 布局脚本

要为多个路由加载第三方脚本，请导入 `next/script` 并直接在您的布局组件中包含该脚本：

{% code title="app/dashboard/layout.tsx" %}
```tsx
import Script from 'next/script'
 
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <>
      <section>{children}</section>
      <Script src="https://example.com/script.js" />
    </>
  )
}
```
{% endcode %}

当用户访问文件夹路由（例如 `dashboard/page.js` ）或任何嵌套路由（例如 `dashboard/settings/page.js` ）时，将获取第三方脚本。Next.js 将确保脚本只加载一次，即使用户在同一布局中在多个路由之间导航。

### 应用程序脚本

要为所有路由加载第三方脚本，请导入 `next/script` 并直接在根布局中包含脚本：

{% code title="app/layout.tsx" %}
```tsx
import Script from 'next/script'
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
      <Script src="https://example.com/script.js" />
    </html>
  )
}
```
{% endcode %}

当访问您应用程序中的任何路由时，此脚本将加载并执行。Next.js 将确保该脚本仅加载一次，即使用户在多个页面之间导航。

> **注意**：
>
> 我们建议仅在特定页面或布局中包含第三方脚本，以尽量减少对性能的任何不必要影响。

### 策略

尽管 `next/script` 的默认行为允许您在任何页面或布局中加载第三方脚本，但您可以通过使用 `strategy` 属性来微调其加载行为：

* `beforeInteractive` : 在任何 Next.js 代码和任何页面水合发生之前加载脚本。
* `afterInteractive` : (默认) 早期加载脚本，但在页面发生一些水合之后。
* `lazyOnload` : 在浏览器空闲时间稍后加载脚本。
* `worker` : (实验性) 在 Web Worker 中加载脚本。

请参阅 `next/script` API 参考文档以了解每种策略及其用例的更多信息。

### 将脚本装载到 Web Workter 中（实验性）

> **警告**：`worker` 策略尚不稳定，并且尚不支持 App Router。请谨慎使用。

使用 `worker` 策略的脚本被卸载并在 Partytown 的 Web Worker 中执行。这可以通过将主线程专用于其余应用程序代码来提高您网站的性能。

此策略仍处于实验阶段，仅在 `nextScriptWorkers` 标志在 `next.config.js` 中启用时可使用：

{% code title="next.config.js" %}
```javascript
module.exports = {
  experimental: {
    nextScriptWorkers: true,
  },
}
```
{% endcode %}

然后，运行 `next` （通常是 `npm run dev` 或 `yarn dev` ），Next.js 将引导您完成所需软件包的安装以完成设置：

```bash
npm run dev
```

您将看到类似这样的说明：请通过运行 `npm install @builder.io/partytown` 安装 Partytown

一旦设置完成，定义 `strategy="worker"` 将自动在您的应用程序中实例化 Partytown，并将脚本卸载到 Web Worker。

{% code title="pages/home.tsx" %}
```tsx
import Script from 'next/script'
 
export default function Home() {
  return (
    <>
      <Script src="https://example.com/script.js" strategy="worker" />
    </>
  )
}
```
{% endcode %}

在 Web Worker 中加载第三方脚本时，需要考虑许多权衡。有关更多信息，请参阅 Partytown 的权衡文档。

### 内联脚本

内联脚本，或未从外部文件加载的脚本，也受到脚本组件的支持。它们可以通过将 JavaScript 放在大括号内来编写：

```tsx
<Script id="show-banner">
  {`document.getElementById('banner').classList.remove('hidden')`}
</Script>
```

或者通过使用 `dangerouslySetInnerHTML` 属性：

```tsx
<Script
  id="show-banner"
  dangerouslySetInnerHTML={{
    __html: `document.getElementById('banner').classList.remove('hidden')`,
  }}
/>
```

> **警告**：必须为内联脚本分配一个 `id` 属性，以便 Next.js 跟踪和优化该脚本。

### 执行附加代码

事件处理程序可以与脚本组件一起使用，以在某个事件发生后执行额外的代码：

* `onLoad` : 在脚本加载完成后执行代码。
* `onReady` : 在脚本加载完成后以及每次组件挂载时执行代码。
* `onError` : 如果脚本加载失败，则执行代码。

这些处理程序将仅在 `next/script` 被导入并在定义 `"use client"` 作为第一行代码的客户端组件内使用时有效：

{% code title="app/page.tsx" %}
```tsx
'use client'
 
import Script from 'next/script'
 
export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        onLoad={() => {
          console.log('Script has loaded')
        }}
      />
    </>
  )
}
```
{% endcode %}

请参考 `next/script` API 参考文档以了解每个事件处理程序的更多信息并查看示例。

### 附加属性

有许多 DOM 属性可以分配给 `<script>` 元素，这些属性不被 Script 组件使用，比如 `nonce` 或自定义数据属性。包括任何附加属性将自动转发到包含在 HTML 中的最终优化 `<script>` 元素。

{% code title="app/page.tsx" %}
```tsx
import Script from 'next/script'
 
export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        id="example-script"
        nonce="XUENAJFW"
        data-test="script"
      />
    </>
  )
}
```
{% endcode %}

### API 参考

了解有关 next/script API 的更多信息。
