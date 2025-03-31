# 本地开发

Next.js 旨在提供出色的开发者体验。随着您的应用程序增长，您可能会注意到在本地开发期间编译时间变慢。此指南将帮助您识别和修复常见的编译时性能问题。

### [本地开发与生产环境](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#local-dev-vs-production) <a href="#local-dev-vs-production" id="local-dev-vs-production"></a>

使用 `next dev` 的开发过程与 `next build` 和 `next start` 不同。

`next dev` 在您打开或浏览时编译您应用程序中的路由。这使您可以在不等待应用程序中的每个路由编译的情况下启动开发服务器，这样更快且占用更少内存。运行生产构建将应用其他优化，例如压缩文件和创建内容哈希，而这些在本地开发中并不需要。

### [提高本地开发性能](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#improving-local-dev-performance) <a href="#improving-local-dev-performance" id="improving-local-dev-performance"></a>

#### [1. 检查计算机的防病毒软件](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#1-check-your-computers-antivirus) <a href="#id-1-check-your-computers-antivirus" id="id-1-check-your-computers-antivirus"></a>

防病毒软件可能会减慢文件访问速度。

尝试将您的项目文件夹添加到防病毒排除列表中。虽然这在 Windows 机器上更为常见，但我们建议在任何安装了防病毒工具的系统上执行此操作。

#### [2. 更新 Next.js 并启用 Turbopack](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#2-update-nextjs-and-enable-turbopack) <a href="#id-2-update-nextjs-and-enable-turbopack" id="id-2-update-nextjs-and-enable-turbopack"></a>

确保您正在使用最新版本的 Next.js。每个新版本通常都会包括性能改进。

Turbopack 是集成到 Next.js 中的新打包工具，可以提升本地性能。

```
npm install next@latestnpm run dev --turbopack
```

了解更多关于 Turbopack 的信息。请查看我们的升级指南和代码修改以获取更多信息。

#### [3. 检查你的导入](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#3-check-your-imports) <a href="#id-3-check-your-imports" id="id-3-check-your-imports"></a>

导入代码的方式会极大影响编译和打包时间。了解更多关于优化包打包的信息，并探索像 Dependency Cruiser 或 Madge 这样的工具。

#### [图标库](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#icon-libraries) <a href="#icon-libraries" id="icon-libraries"></a>

像 `@material-ui/icons` 或 `react-icons` 的库可以导入数千个图标，即使你只使用一些。尝试仅导入你需要的图标：

```
// Instead of this:import { Icon1, Icon2 } from 'react-icons/md' // Do this:import Icon1 from 'react-icons/md/Icon1'import Icon2 from 'react-icons/md/Icon2'
```

像 `react-icons` 的库包含许多不同的图标集。选择一个集并坚持使用该集。

例如，如果你的应用程序使用 `react-icons` 并导入所有这些：

* `pi` （Phosphor 图标）
* `md` （物料设计图标）
* `tb` （tabler-icons）
* `cg` （cssgg）

合并后，将有数以万计的模块需要编译器处理，即使你仅使用每个模块中的单个导入。

#### [桶文件](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#barrel-files) <a href="#barrel-files" id="barrel-files"></a>

""桶文件" 是从其他文件导出多个项目的文件。它们可能会减慢构建速度，因为编译器必须解析它们，以查看在模块范围内是否存在通过使用 import 引入的副作用。"

尽可能直接从特定文件导入。了解更多关于桶文件和 Next.js 中的内置优化。

#### [优化包导入](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#optimize-package-imports) <a href="#optimize-package-imports" id="optimize-package-imports"></a>

Next.js 可以自动优化某些包的导入。如果您正在使用利用桶文件的包，请将它们添加到您的 `next.config.js` :

```
module.exports = {  experimental: {    optimizePackageImports: ['package-name'],  },}
```

#### [4. 检查你的 Tailwind CSS 设置](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#4-check-your-tailwind-css-setup) <a href="#id-4-check-your-tailwind-css-setup" id="id-4-check-your-tailwind-css-setup"></a>

如果你在使用 Tailwind CSS，请确保它已正确设置。

一个常见的错误是以包含 `node_modules` 或其他不应被扫描的大目录文件的方式配置你的 `content` 数组。

Tailwind CSS 版本 3.4.8 或更高版本将警告你可能会减慢构建速度的设置。

1.  在你的 `tailwind.config.js` 中，具体说明要扫描哪些文件：

    ```
    module.exports = {  content: [    './src/**/*.{js,ts,jsx,tsx}', // Good    // This might be too broad    // It will match `packages/**/node_modules` too    // '../../packages/**/*.{js,ts,jsx,tsx}',  ],}
    ```
2.  避免扫描不必要的文件：

    ```
    module.exports = {  content: [    // Better - only scans the 'src' folder    '../../packages/ui/src/**/*.{js,ts,jsx,tsx}',  ],}
    ```

#### [5. 检查自定义 webpack 设置](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#5-check-custom-webpack-settings) <a href="#id-5-check-custom-webpack-settings" id="id-5-check-custom-webpack-settings"></a>

如果您添加了自定义的 webpack 设置，它们可能会减慢编译速度。

考虑一下您是否真的需要它们用于本地开发。您可以选择仅为生产构建包含某些工具，或者探索迁移到 Turbopack 并使用加载器。

#### [6. 优化内存使用](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#6-optimize-memory-usage) <a href="#id-6-optimize-memory-usage" id="id-6-optimize-memory-usage"></a>

如果您的应用程序非常大，它可能需要更多内存。

了解更多关于优化内存使用的信息。

#### [7. 服务器组件和数据获取](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#7-server-components-and-data-fetching) <a href="#id-7-server-components-and-data-fetching" id="id-7-server-components-and-data-fetching"></a>

对服务器组件的更改会导致整个页面在本地重新渲染，以显示新的更改，这包括为组件获取新数据。

实验性的 `serverComponentsHmrCache` 选项允许您在本地开发中的热模块替换 (HMR) 刷新期间缓存服务器组件中的 `fetch` 响应。这会导致更快的响应和减少计费 API 调用的成本。

了解更多有关实验性选项的信息。

### [查找问题的工具](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#tools-for-finding-problems) <a href="#tools-for-finding-problems" id="tools-for-finding-problems"></a>

#### [详细的抓取日志](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#detailed-fetch-logging) <a href="#detailed-fetch-logging" id="detailed-fetch-logging"></a>

使用此命令查看开发过程中发生的更详细信息：

```
next dev --verbose
```

### [仍然有问题吗？](https://nextjs.org/docs/app/building-your-application/optimizing/local-development#still-having-problems) <a href="#still-having-problems" id="still-having-problems"></a>

如果你尝试了所有方法仍然有问题：

1. 创建一个简单版本的应用程序来显示问题。
2.  生成一个特殊文件来显示发生了什么：

    ```
    NEXT_CPU_PROF=1 npm run dev
    ```
3. 在 GitHub 上作为新问题分享此文件（在项目的主文件夹中找到）和 `.next/trace` 文件。
