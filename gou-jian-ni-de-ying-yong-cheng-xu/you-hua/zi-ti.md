---
cover: >-
  https://nextjs.org/api/docs-og?title=%E4%BC%98%E5%8C%96%EF%BC%9A%20%E5%AD%97%E4%BD%93
coverY: 0
---

# 字体

`next/font` 将自动优化您的字体（包括自定义字体）并移除外部网络请求，以提高隐私和性能。

> 🎥观看：了解更多关于使用 `next/font` → [YouTube（6 分钟）](https://www.youtube.com/watch?v=L8_98i_bMMA)。

`next/font` 包含针对任何字体文件的**内置自动自托管**功能。这意味着您可以以零布局位移的方式最佳加载网络字体，得益于底层的 CSS `size-adjust` 属性。

这个新的字体系统还允许您方便地使用所有 Google 字体，同时考虑性能和隐私。CSS 和字体文件在构建时下载，并与您的其余静态资产一起自托管。**浏览器不会向 Google 发送请求。**

### **谷歌字体**

自动自托管任何谷歌字体。字体包含在部署中，并从与您的部署相同的域名提供。浏览器不会向谷歌发送请求。

通过将您想要使用的字体从 `next/font/google` 导入作为函数来开始。我们建议使用[可变字体](https://fonts.google.com/variablefonts)以获得最佳性能和灵活性。

{% code title="app/layout.tsx" %}
```tsx
import { Inter } from 'next/font/google'
 
// 如果加载可变字体，您无需指定字体粗细
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
})
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  )
}
```
{% endcode %}

如果您无法使用可变字体，则需要指定一个字重：

{% code title="app/layout.tsx" %}
```tsx
import { Roboto } from 'next/font/google'
 
const roboto = Roboto({
  weight: '400',
  subsets: ['latin'],
  display: 'swap',
})
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={roboto.className}>
      <body>{children}</body>
    </html>
  )
}
```
{% endcode %}

您可以通过使用数组来指定多个权重和（或）样式：

{% code title="app/layout.js" %}
```javascript
const roboto = Roboto({
  weight: ['400', '700'],
  style: ['normal', 'italic'],
  subsets: ['latin'],
  display: 'swap',
})
```
{% endcode %}

> 注意：对于多个单词的字体名称，请使用下划线（\_）。例如， `Roboto Mono` 应该作为 `Roboto_Mono` 导入。

#### 指定子集

Google Fonts 会自动进行子集化。这减少了字体文件的大小并提高了性能。您需要定义要预加载的子集。如果在 `preload` 为 `true` 时未指定任何子集，将会产生警告。

这可以通过将其添加到函数调用中来完成：

{% code title="app/layout.tsx" %}
```tsx
const inter = Inter({ subsets: ['latin'] })
```
{% endcode %}

查看字体 API 参考以获取更多信息。

#### 使用多个字体

您可以在应用程序中导入和使用多个字体。您可以采取两种方法。

第一种方法是创建一个实用函数，该函数导出一个字体，导入它，并在需要的地方应用其 `className` 。这确保字体仅在渲染时预加载：

{% code title="app/fonts.ts" %}
```tsx
import { Inter, Roboto_Mono } from 'next/font/google'
 
export const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
})
 
export const roboto_mono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
})
```
{% endcode %}

{% code title="app/layout.tsx" %}
```tsx
import { inter } from './fonts'
 
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={inter.className}>
      <body>
        <div>{children}</div>
      </body>
    </html>
  )
}
```
{% endcode %}

{% code title="app/page.tsx" %}
```tsx
import { roboto_mono } from './fonts'
 
export default function Page() {
  return (
    <>
      <h1 className={roboto_mono.className}>My page</h1>
    </>
  )
}
```
{% endcode %}

在上面的示例中， `Inter` 将被全局应用，而 `Roboto Mono` 可以根据需要导入并应用。

另外，您可以创建一个 CSS 变量并将其与您喜欢的 CSS 解决方案一起使用：

{% code title="app/layout.tsx" %}
```tsx
import { Inter, Roboto_Mono } from 'next/font/google'
import styles from './global.css'
 
const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
  display: 'swap',
})
 
const roboto_mono = Roboto_Mono({
  subsets: ['latin'],
  variable: '--font-roboto-mono',
  display: 'swap',
})
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={`${inter.variable} ${roboto_mono.variable}`}>
      <body>
        <h1>My App</h1>
        <div>{children}</div>
      </body>
    </html>
  )
}
```
{% endcode %}

{% code title="app/global.css" %}
```css
html {
  font-family: var(--font-inter);
}
 
h1 {
  font-family: var(--font-roboto-mono);
}
```
{% endcode %}

在上面的示例中， `Inter` 将被全局应用，任何 `<h1>` 标签将使用 `Roboto Mono` 样式。

> **建议**：谨慎使用多种字体，因为每种新字体都是客户端需要下载的额外资源。

### 本地字体

导入 `next/font/local` 并指定本地字体文件的 `src`。我们建议使用可变字体以获得最佳性能和灵活性。

{% code title="app/layout.tsx" %}
```tsx
import localFont from 'next/font/local'
 
// Font files can be colocated inside of `app`
const myFont = localFont({
  src: './my-font.woff2',
  display: 'swap',
})
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={myFont.className}>
      <body>{children}</body>
    </html>
  )
}
```
{% endcode %}

如果您想为单个字体系列使用多个文件， `src` 可以是一个数组：

{% code title="" %}
```javascript
const roboto = localFont({
  src: [
    {
      path: './Roboto-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: './Roboto-Italic.woff2',
      weight: '400',
      style: 'italic',
    },
    {
      path: './Roboto-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
    {
      path: './Roboto-BoldItalic.woff2',
      weight: '700',
      style: 'italic',
    },
  ],
})
```
{% endcode %}

查看[字体 API 参考](https://nextjs.org/docs/app/api-reference/components/font)以获取更多信息。

### 配合 Tailwind CSS

`next/font` 可以通过 CSS 变量与 Tailwind CSS 一起使用。

在下面的示例中，我们使用来自 `next/font/google` 的字体 `Inter` （您可以使用来自 Google 或本地字体的任何字体）。使用 `variable` 选项加载您的字体，以定义您的 CSS 变量名称并将其分配给 `inter` 。然后，使用 `inter.variable` 将 CSS 变量添加到您的 HTML 文档中。

{% code title="app/layout.tsx" %}
```tsx
import { Inter, Roboto_Mono } from 'next/font/google'
 
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
})
 
const roboto_mono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
})
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={`${inter.variable} ${roboto_mono.variable}`}>
      <body>{children}</body>
    </html>
  )
}
```
{% endcode %}

最后，将 CSS 变量添加到您的 [Tailwind CSS 配置](https://nextjs.org/docs/app/building-your-application/styling/tailwind-css#configuring-tailwind)中：

{% code title="tailwind.config.js" %}
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
    './app/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      fontFamily: {
        sans: ['var(--font-inter)'],
        mono: ['var(--font-roboto-mono)'],
      },
    },
  },
  plugins: [],
}
```
{% endcode %}

您现在可以使用 `font-sans` 和 `font-mono` 工具类将字体应用于您的元素。

### 预加载

当在您网站的某个页面上调用字体函数时，它并不是全局可用的，也不会在所有路由上预加载。相反，字体仅在与其使用的文件类型相关的路由上预加载：

* 如果这是一个[独特的页面](https://nextjs.org/docs/app/api-reference/file-conventions/page)，它将在该页面的独特路由上预加载。
* 如果这是一个布局，它将在所有被该布局包裹的路由上预加载。
* 如果是根布局，它会在所有路由上预加载。

### 重用字体

每次调用 `localFont` 或 Google 字体函数时，该字体在您的应用程序中作为一个实例托管。因此，如果您在多个文件中加载相同的字体函数，则会托管多个相同字体的实例。在这种情况下，建议执行以下操作：

* 在一个共享文件中调用字体加载器函数
* 将其导出为常量
* 在每个希望使用此字体的文件中导入常量

