# 图像

<details>

<summary>示例</summary>

[图像组件](https://github.com/vercel/next.js/tree/canary/examples/image-component)

</details>

根据《网络年鉴》，图像占据了典型网站页面重量的很大一部分，并且对您网站的 LCP 性能有着显著影响。

Next.js 图像组件扩展了 HTML `<img>` 元素，提供了自动图像优化的功能：

* 尺寸优化：自动为每个设备提供正确尺寸的图像，使用现代图像格式如 WebP。
* 视觉稳定性：在图像加载时自动防止布局偏移。
* 更快的页面加载：图像仅在进入视口时加载，使用原生浏览器延迟加载，并可选地使用模糊占位符。
* 资产灵活性：按需图像调整大小，即使是存储在远程服务器上的图像。

> **🎥 观看**：了解如何使用 `next/image` → [YouTube（9 分钟）](https://youtu.be/IU_qq_c_lKA)。

### 使用

```tsx
import Image from 'next/image'
```

然后可以为您的图像（本地或远程）定义 `src` 。

#### 本地图像

要使用本地图像， `import` 您的 `.jpg` 、 `.png` 或 `.webp` 图像文件。

Next.js 将根据导入的文件自动确定您的图像的固有 `width` 和 `height` 。这些值用于确定图像比例，并在图像加载时防止累积布局偏移。

{% code title="app/page.js" %}
```tsx
import Image from 'next/image'
import profilePic from './me.png'
 
export default function Page() {
  return (
    <Image
      src={profilePic}
      alt="Picture of the author"
      // width={500} automatically provided
      // height={500} automatically provided
      // blurDataURL="data:..." automatically provided
      // placeholder="blur" // Optional blur-up while loading
    />
  )
}
```
{% endcode %}

> **警告**：动态 `await import()` 或 `require()` 不受支持。 `import` 必须是静态的，以便在构建时进行分析。

您可以选择在您的 `next.config.js` 文件中配置 `localPatterns` ，以允许特定图像并阻止所有其他图像。

{% code title="next.config.js" %}
```javascript
module.exports = {
  images: {
    localPatterns: [
      {
        pathname: '/assets/images/**',
        search: '',
      },
    ],
  },
}
```
{% endcode %}

#### 远程图像

要使用远程图像， `src` 属性应该是一个 URL 字符串。

由于 Next.js 在构建过程中无法访问远程文件，您需要手动提供 `width` 、 `height` 和可选的 `blurDataURL` 属性。

`width` 和 `height` 属性用于推断图像的正确纵横比，并避免图像加载时的布局偏移。 `width` 和 `height` 不会决定图像文件的渲染大小。了解有关图像大小调整的更多信息。

{% code title="app/page.js" %}
```jsx
import Image from 'next/image'
 
export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
```
{% endcode %}

为了安全地优化图像，在 `next.config.js` 中定义支持的 URL 模式列表。请尽可能具体，以防止恶意使用。例如，以下配置将只允许来自特定 AWS S3 存储桶的图像：

{% code title="next.config.js" %}
```javascript
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 's3.amazonaws.com',
        port: '',
        pathname: '/my-bucket/**',
        search: '',
      },
    ],
  },
}
```
{% endcode %}

了解有关 [`remotePatterns`](https://nextjs.org/docs/app/api-reference/components/image#remotepatterns) 配置的更多信息。如果您想为图像 `src` 使用相对 URL，请使用 [`loader`](https://nextjs.org/docs/app/api-reference/components/image#loader) 。

#### 域名

有时您可能希望优化远程图像，但仍然使用内置的 Next.js 图像优化 API。要做到这一点，请将 `loader` 保持在默认设置，并为图像 `src` 属性输入绝对 URL。

为了保护您的应用程序免受恶意用户的攻击，您必须定义一个您打算与 `next/image` 组件一起使用的远程主机名列表。

> 了解有关 [`remotePatterns`](https://nextjs.org/docs/app/api-reference/components/image#remotepatterns) 配置的更多信息。

#### 加载器

请注意，在之前的示例中，提供了一个部分 URL（ `"/me.png"` ）作为本地图像。这是由于加载器架构的原因。

加载器是一个生成图像 URL 的函数。它修改提供的 `src` ，并生成多个请求不同尺寸图像的 URL。这些多个 URL 用于自动 srcset 生成，以便向访问您网站的访客提供适合其视口大小的图像。

Next.js 应用程序的默认加载器使用内置的图像优化 API，它会优化来自网络任何地方的图像，然后直接从 Next.js 网络服务器提供这些图像。如果您希望直接从 CDN 或图像服务器提供图像，可以用几行 JavaScript 编写自己的加载器函数。

您可以为每个图像定义一个加载器，使用 [`loader` 属性](https://nextjs.org/docs/app/api-reference/components/image#loader)，或使用 [`loaderFile` 配置](https://nextjs.org/docs/app/api-reference/components/image#loaderfile)在应用程序级别进行定义。

### 优先级

您应该将 `priority` 属性添加到每个页面的最大内容绘制 (LCP) 元素的图像中。这样可以让 Next.js 预加载图像，从而显著提高 LCP。

LCP 元素通常是页面视口中可见的最大图像或文本块。当你运行 `next dev` 时，如果 LCP 元素是一个 `<Image>` 而没有 `priority` 属性，你会看到控制台警告。

一旦你确定了 LCP 图像，你可以像这样添加属性：

{% code title="app/page.js" %}
```jsx
import Image from 'next/image'
import profilePic from '../public/me.png'
 
export default function Page() {
  return <Image src={profilePic} alt="Picture of the author" priority />
}
```
{% endcode %}

有关优先级的更多信息，请参阅 `next/image` 组件文档。

### 图像尺寸

影响性能的图像最常见的方式之一是布局偏移，即图像在加载时推动页面上的其他元素。这种性能问题对用户来说非常烦人，以至于它有自己的核心网页指标，称为累计布局偏移。避免基于图像的布局偏移的方法是始终为图像设置尺寸。这使浏览器在图像加载之前可以准确地保留足够的空间。

因为 `next/image` 旨在保证良好的性能结果，因此不能以会导致布局偏移的方式使用，并且必须以三种方式之一进行大小设置：

1. 自动地，使用[静态导入](https://nextjs.org/docs/app/building-your-application/optimizing/images#local-images)
2. 手动地，通过包含一个 `width` 和 `height` 属性来确定图像的纵横比。
3. 隐式地，通过使用 fill，使图像扩展以填充其父元素。

> **如果我不知道我的图片大小怎么办？**
>
> 如果您从一个不知道图像大小的来源访问图像，可以采取几种措施：
>
> **使用 `fill`**
>
> `fill` 属性允许您的图像由其父元素的大小决定。考虑使用 CSS 为图像的父元素在页面上留出空间，并使用 `sizes` 属性以匹配任何媒体查询断点。您还可以与 `object-fit` 、 `fill` 、 `contain` 、 `cover` 和 `object-position` 一起使用，以定义图像应如何占用该空间。
>
> **规范化您的图像**
>
> 如果您从自己控制的源服务器上提供图像，请考虑修改您的图像处理管道以将图像规范化为特定大小。
>
> **修改您的 API 调用**
>
> 如果您的应用程序通过 API 调用（例如 CMS）检索图像 URL，您可能能够修改 API 调用以返回与 URL 一起的图像尺寸。

如果没有任何建议的方法能解决图像大小问题， `next/image` 组件被设计为能够与标准 `<img>` 元素一起良好地工作。

### 样式

为 Image 组件设置样式类似于为普通 `<img>` 元素设置样式，但有一些指导原则需要牢记：

* 使用 `className` 或 `style` ，而不是 `styled-jsx` 。
  * 在大多数情况下，我们建议使用 `className` 属性。这可以是导入的 CSS 模块、全局样式表等。
  * 您也可以使用 `style` 属性来分配内联样式。
  * 您不能使用 styled-jsx，因为它限定于当前组件（除非您将样式标记为 `global` ）。
* 当使用 `fill` 时，父元素必须具有 `position: relative` 。
  * 这对于在该布局模式下正确渲染图像元素是必要的。
* 当使用 `fill` 时，父元素必须具有 `display: block` 。
  * 这是 `<div>` 元素的默认值，但应该另行指定。

### 示例

#### 响应式

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

```jsx
import Image from 'next/image'
import mountains from '../public/mountains.jpg'
 
export default function Responsive() {
  return (
    <div style={{ display: 'flex', flexDirection: 'column' }}>
      <Image
        alt="Mountains"
        // Importing an image will
        // automatically set the width and height
        src={mountains}
        sizes="100vw"
        // Make the image display full width
        style={{
          width: '100%',
          height: 'auto',
        }}
      />
    </div>
  )
}
```

#### 填充容器

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

```jsx
import Image from 'next/image'
import mountains from '../public/mountains.jpg'
 
export default function Fill() {
  return (
    <div
      style={{
        display: 'grid',
        gridGap: '8px',
        gridTemplateColumns: 'repeat(auto-fit, minmax(400px, auto))',
      }}
    >
      <div style={{ position: 'relative', height: '400px' }}>
        <Image
          alt="Mountains"
          src={mountains}
          fill
          sizes="(min-width: 808px) 50vw, 100vw"
          style={{
            objectFit: 'cover', // cover, contain, none
          }}
        />
      </div>
      {/* And more images in the grid... */}
    </div>
  )
}
```

#### 背景图像

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

```jsx
import Image from 'next/image'
import mountains from '../public/mountains.jpg'
 
export default function Background() {
  return (
    <Image
      alt="Mountains"
      src={mountains}
      placeholder="blur"
      quality={100}
      fill
      sizes="100vw"
      style={{
        objectFit: 'cover',
      }}
    />
  )
}
```

有关与各种样式一起使用的图像组件的示例，请参阅[图像组件演示](https://image-component.nextjs.gallery/)。

### 其他属性

[**查看所有可用于 `next/image` 组件的属性。**](https://nextjs.org/docs/app/api-reference/components/image)

### **配置**

`next/image` 组件和 Next.js 图像优化 API 可以在 [`next.config.js` 文件](https://nextjs.org/docs/app/api-reference/config/next-config-js)中进行配置。这些配置允许您[启用远程图像](https://nextjs.org/docs/app/api-reference/components/image#remotepatterns)、[定义自定义图像断点](https://nextjs.org/docs/app/api-reference/components/image#devicesizes)、[改变缓存行为](https://nextjs.org/docs/app/api-reference/components/image#caching-behavior)等。

[**阅读完整的图像配置文档以获取更多信息。**](https://nextjs.org/docs/app/api-reference/components/image#configuration-options)

### **API 参考**

了解更多关于 next/image API 的信息。
