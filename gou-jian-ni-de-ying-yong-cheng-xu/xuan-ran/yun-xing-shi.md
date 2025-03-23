# 运行时

Next.js 有两个可以在您的应用程序中使用的服务器运行时：

* **Node.js 运行时**（默认），可以访问所有 Node.js API 和生态系统中的兼容包。
* [**边缘运行时**](#user-content-fn-1)[^1]，它包含[一组更有限的 API](https://nextjs.org/docs/app/api-reference/edge)。

边缘运行时是[中间件](https://nextjs.org/docs/app/building-your-application/routing/middleware)的默认运行时。然而，这可以更改为 Node.js 运行时。有关更多详细信息，请参阅[中间件文档](https://nextjs.org/docs/app/building-your-application/routing/middleware#runtime)。

### 用例

* Node.js 运行时用于渲染您的应用程序。
* 边缘运行时用于中间件（路由规则，如重定向、重写和设置头部）。

### 注意事项

* 边缘运行时不支持所有 Node.js API。一些软件包可能无法按预期工作。了解有关边缘运行时中不支持的 API 的更多信息。
* 边缘运行时不支持增量静态再生 (ISR)。
* 根据您的部署基础设施，两种运行时均可支持流式传输。

### 下一步

查看边缘运行时 API 参考。



[^1]: **Edge Runtime** 指的是在 **边缘计算环境** 中运行的**计算资源和代码执行环境**，而不是传统的中心化服务器。
