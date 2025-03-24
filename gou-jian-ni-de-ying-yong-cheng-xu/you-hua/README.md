# 优化

Next.js 提供了多种内置优化，旨在提高您应用程序的速度和核心网页指标。本指南将涵盖您可以利用的优化，以增强用户体验。

### 内置组件

内置组件抽象了实现常见 UI 优化的复杂性。这些组件包括：

* 图片：基于原生的 `<img>` 元素构建。图像组件通过延迟加载和根据设备尺寸自动调整图像大小来优化图像性能。
* 链接：基于原生的 `<a>` 标签构建。链接组件在后台预取页面，以实现更快、更流畅的页面过渡。
* 脚本：基于原生的 `<script>` 标签构建。脚本组件让您控制第三方脚本的加载和执行。

### 元数据

元数据帮助搜索引擎更好地理解您的内容（这可能会导致更好的 SEO），并允许您自定义内容在社交媒体上的展示方式，帮助您在各个平台上创建更具吸引力和一致的用户体验。

Next.js 中的元数据 API 允许您修改页面的 `<head>` 元素。您可以通过两种方式配置元数据：

* 基于配置的元数据：在 `layout.js` 或 `page.js` 文件中导出[静态 `metadata` 对象](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-object)或动态 [`generateMetadata` 函数](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#generatemetadata-function)。
* 基于文件的元数据：将静态或动态生成的特殊文件添加到路由段。

此外，您可以使用 JSX 和 CSS 与 [imageResponse](https://nextjs.org/docs/app/api-reference/functions/image-response) 构造函数一起创建动态 Open Graph 图像。

### 静态资产

Next.js `/public` 文件夹可用于提供静态资产，例如图像、字体和其他文件。 `/public` 中的文件还可以被 CDN 提供商缓存，以便高效地交付。

### 分析与监控

对于大型应用程序，Next.js 与流行的分析和监控工具集成，以帮助您了解应用程序的性能。有关更多信息，请参阅 OpenTelemetry 和仪器指南。
