# 包捆绑

捆绑外部包可以显著提高您应用程序的性能。默认情况下，在服务器组件和路由处理程序内部导入的包会被 Next.js 自动捆绑。此页面将指导您如何分析和进一步优化包捆绑。

### 分析 Javascript 包

`@next/bundle-analyzer` 是一个 Next.js 插件，帮助您管理应用程序包的大小。它生成每个包及其依赖项大小的可视化报告。您可以使用这些信息来删除大型依赖项、拆分或懒加载您的代码。

#### 安装

通过运行以下命令安装插件：

```bash
npm i @next/bundle-analyzer
# or
yarn add @next/bundle-analyzer
# or
pnpm add @next/bundle-analyz
```

然后，将打包分析器的设置添加到您的 `next.config.js` 中。

{% code title="next.config.js" %}
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {}
 
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
})
 
module.exports = withBundleAnalyzer(nextConfig)
```
{% endcode %}

#### 生成报告

运行以下命令以分析您的包：

```bash
ANALYZE=true npm run build
# or
ANALYZE=true yarn build
# or
ANALYZE=true pnpm build
```

报告将在您的浏览器中打开三个新标签，您可以进行检查。定期评估您应用程序的包可以帮助您在一段时间内保持应用程序性能。

### 优化包导入

某些包，例如图标库，可以导出数百个模块，这可能会导致开发和生产中的性能问题。

您可以通过将 `optimizePackageImports` 选项添加到您的 `next.config.js` 来优化这些包的导入方式。此选项将仅加载您实际使用的模块，同时仍然为您提供使用多个命名导出编写导入语句的便利。

{% code title="next.config.js" %}
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    optimizePackageImports: ['icon-library'],
  },
}
 
module.exports = nextConfig
```
{% endcode %}

Next.js 还会自动优化某些库，因此它们不需要包含在 optimizePackageImports 列表中。查看完整列表。

### 选择特定包不进行打包

由于在服务器组件和路由处理程序中导入的包会被 Next.js 自动打包，您可以使用 `serverExternalPackages` 选项在您的 `next.config.js` 中选择特定包不进行打包。

{% code title="next.config.js" %}
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  serverExternalPackages: ['package-name'],
}
 
module.exports = nextConfig
```
{% endcode %}

Next.js 包含一个当前正在兼容性工作并自动选择退出的流行包列表。查看完整列表。

### 下一步

了解更多关于优化您的应用程序以用于生产的信息。
