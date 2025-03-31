# 元数据

Next.js 有一个元数据 API，可以用来定义您的应用程序元数据（例如 `meta` 和 `link` 标签在您的 HTML `head` 元素内），以改善 SEO 和网页分享性。

您可以通过两种方式将元数据添加到您的应用程序：

* 基于配置的元数据：在 `layout.js` 或 `page.js` 文件中导出一个静态 `metadata` 对象或一个动态 `generateMetadata` 函数。
* 基于文件的元数据：将静态或动态生成的特殊文件添加到路由段。

使用这两个选项，Next.js 将自动为您的页面生成相关的 `<head>` 元素。您还可以使用 `ImageResponse` 构造函数创建动态 OG 图像。

### 静态元数据

要定义静态元数据，请从 `layout.js` 或静态 `page.js` 文件中导出一个 `Metadata` 对象。

{% code title="layout.tsx | page.tsx" %}
```tsx
import type { Metadata } from 'next'
 
export const metadata: Metadata = {
  title: '...',
  description: '...',
}
 
export default function Page() {}
```
{% endcode %}

有关所有可用选项，请参阅 API 参考。

### 动态元数据

您可以使用 `generateMetadata` 函数来 `fetch` 需要动态值的元数据。

{% code title="app/products/[id]/page.tsx" %}
```tsx
import type { Metadata, ResolvingMetadata } from 'next'
 
type Props = {
  params: Promise<{ id: string }>
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}
 
export async function generateMetadata(
  { params, searchParams }: Props,
  parent: ResolvingMetadata
): Promise<Metadata> {
  // read route params
  const { id } = await params
 
  // fetch data
  const product = await fetch(`https://.../${id}`).then((res) => res.json())
 
  // optionally access and extend (rather than replace) parent metadata
  const previousImages = (await parent).openGraph?.images || []
 
  return {
    title: product.title,
    openGraph: {
      images: ['/some-specific-page-image.jpg', ...previousImages],
    },
  }
}
 
export default function Page({ params, searchParams }: Props) {}
```
{% endcode %}

有关所有可用参数，请参阅 API 参考。

> **注意**：
>
> * 通过 `generateMetadata` 的静态和动态元数据仅在服务器组件中受支持。
> * `fetch` 请求会自动缓存相同数据在 `generateMetadata` 、 \{{2 \}} 、布局、页面和服务器组件中。如果 `fetch` 不可用，可以使用 React `cache` 。
> * Next.js 将等待 `generateMetadata` 中的数据获取完成，然后再将 UI 流式传输到客户端。这保证了流式响应的第一部分包含 `<head>` 标签。

### 基于文件的元数据

这些特殊文件可用于元数据：

* [favicon.ico, apple-icon.jpg 和 icon.jpg](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/app-icons)
* [opengraph-image.jpg 和 twitter-image.jpg](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image)
* [robots.txt](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/robots)
* [sitemap.xml](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/sitemap)

您可以将这些用于静态元数据，或者您可以通过代码程序生成这些文件。

有关实现和示例，请参阅元数据文件 API 参考和动态图像生成。

### 行为

基于文件的元数据具有更高的优先级，并将覆盖任何基于配置的元数据。

#### 默认字段

即使路由未定义元数据，也始终会添加两个默认的 `meta` 标签：

* meta charset 标签设置网站的字符编码。
* meta 视口标签设置网站的视口宽度和缩放，以适应不同的设备。

```html
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

> **注意**：您可以覆盖默认的 `viewport` meta 标签。

#### 排序

元数据按顺序评估，从根段开始，到最接近最终 `page.js` 段的段落。例如：

1. `app/layout.tsx` (根布局)
2. `app/blog/layout.tsx` (嵌套博客布局)
3. `app/blog/[slug]/page.tsx` (博客页面)

#### 合并

根据评估顺序，从同一路由中的多个段导出的元数据对象被浅合并在一起，以形成路由的最终元数据输出。重复的键根据其顺序被替换。

这意味着在早期段中定义的具有嵌套字段的元数据，如 `openGraph` 和 `robots` ，会被最后一个段覆盖以重新定义它们。

**覆盖字段**

{% code title="app/layout.js" %}
```javascript
export const metadata = {
  title: 'Acme',
  openGraph: {
    title: 'Acme',
    description: 'Acme is a...',
  },
}
```
{% endcode %}

{% code title="app/blog/page.js" %}
```javascript
export const metadata = {
  title: 'Blog',
  openGraph: {
    title: 'Blog',
  },
}
 
// 输出：
// <title>Blog</title>
// <meta property="og:title" content="Blog" />
```
{% endcode %}

在上面的例子中：

* `title` 从 `app/layout.js` 被 `title` 替换在 `app/blog/page.js` 。
* 所有 `openGraph` 字段来自 `app/layout.js` 将在 `app/blog/page.js` 中被替换，因为 `app/blog/page.js` 设置了 `openGraph` 元数据。请注意缺少 `openGraph.description` 。

如果您想在不同段落之间共享一些嵌套字段，同时覆盖其他字段，可以将它们提取到一个单独的变量中：

{% code title="app/shared-metadata.js" %}
```javascript
export const openGraphImage = { images: ['http://...'] }
```
{% endcode %}

{% code title="app/page.js" %}
```javascript
import { openGraphImage } from './shared-metadata'
 
export const metadata = {
  openGraph: {
    ...openGraphImage,
    title: 'Home',
  },
}
```
{% endcode %}

{% code title="app/about/page.js" %}
```javascript
import { openGraphImage } from '../shared-metadata'
 
export const metadata = {
  openGraph: {
    ...openGraphImage,
    title: 'About',
  },
}
```
{% endcode %}

在上面的示例中，OG 图像在 `app/layout.js` 和 `app/about/page.js` 之间共享，而标题则不同。

**继承字段**

{% code title="app/layout.js" %}
```javascript
export const metadata = {
  title: 'Acme',
  openGraph: {
    title: 'Acme',
    description: 'Acme is a...',
  },
}
```
{% endcode %}

{% code title="app/about/page.js" %}
```javascript
export const metadata = {
  title: 'About',
}
 
// 输出：
// <title>About</title>
// <meta property="og:title" content="Acme" />
// <meta property="og:description" content="Acme is a..." />
```
{% endcode %}

**笔记**

* `title` 从 `app/layout.js` 被 `title` 替换在 `app/about/page.js` 。
* 所有 `openGraph` 字段来自 `app/layout.js` 在 `app/about/page.js` 中被继承，因为 `app/about/page.js` 没有设置 `openGraph` 元数据。

### 动态图像生成

`ImageResponse` 构造函数允许您使用 JSX 和 CSS 生成动态图像。这对于创建社交媒体图像，如 Open Graph 图像、Twitter 卡片等非常有用。

要使用它，您可以从 `next/og` 导入 `ImageResponse` ：

{% code title="app/about/route.js" %}
```javascript
import { ImageResponse } from 'next/og'
 
export async function GET() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 128,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          textAlign: 'center',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        Hello world!
      </div>
    ),
    {
      width: 1200,
      height: 600,
    }
  )
}
```
{% endcode %}

`ImageResponse` 与其他 Next.js API（包括路由处理程序和基于文件的元数据）良好集成。例如，您可以在 `opengraph-image.tsx` 文件中使用 `ImageResponse` 在构建时或请求时动态生成 Open Graph 图像。

`ImageResponse` 支持包括 flexbox 和绝对定位、自定义字体、文本换行、居中和嵌套图像在内的常见 CSS 属性。查看支持的 CSS 属性的完整列表。

> **注意**：
>
> * 示例可在 Vercel OG Playground 中找到。
> * `ImageResponse` 使用 @vercel/og、Satori 和 Resvg 将 HTML 和 CSS 转换为 PNG。
> * 仅支持 flexbox 和一部分 CSS 属性。高级布局（例如 `display: grid` ）将无法工作。
> * 最大捆绑大小为 `500KB` 。捆绑大小包括您的 JSX、CSS、字体、图片和其他任何资源。如果超出限制，请考虑减少任何资源的大小或在运行时获取。
> * 仅支持 `ttf` ， `otf` 和 `woff` 字体格式。为了最大化字体解析速度，建议使用 `ttf` 或 `otf` 而不是 `woff` 。

### JSON-LD

JSON-LD 是一种结构化数据格式，可以被搜索引擎用来理解您的内容。例如，您可以使用它来描述一个人、一个事件、一个组织、一部电影、一本书、一份食谱和许多其他类型的实体。

我们目前对 JSON-LD 的建议是将结构化数据呈现为 `<script>` 标签，在您的 `layout.js` 或 `page.js` 组件中。例如：

{% code title="app/products/[id]/page.tsx" %}
```tsx
export default async function Page({ params }) {
  const { id } = await params
  const product = await getProduct(id)
 
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    image: product.image,
    description: product.description,
  }
 
  return (
    <section>
      {/* Add JSON-LD to your page */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      {/* ... */}
    </section>
  )
}
```
{% endcode %}

您可以使用 Google 的丰富结果测试工具或通用 Schema 标记验证器来验证和测试您的结构化数据。

您可以使用社区包如 `schema-dts` 以 TypeScript 输入您的 JSON-LD：

```typescript
import { Product, WithContext } from 'schema-dts'
 
const jsonLd: WithContext<Product> = {
  '@context': 'https://schema.org',
  '@type': 'Product',
  name: 'Next.js Sticker',
  image: 'https://nextjs.org/imgs/sticker.png',
  description: 'Dynamic at the speed of static.',
}
```

### 下一步

查看所有元数据 API 选项。
