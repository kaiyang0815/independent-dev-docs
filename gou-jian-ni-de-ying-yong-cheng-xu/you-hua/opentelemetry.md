# OpenTelemetry

可观察性对于理解和优化您的 Next.js 应用程序的行为和性能至关重要。

随着应用程序变得越来越复杂，识别和诊断可能出现的问题变得越来越困难。通过利用可观察性工具，如日志记录和指标，开发人员可以深入了解其应用程序的行为，并识别优化的领域。通过可观察性，开发人员可以主动解决问题，以防它们成为重大问题，并提供更好的用户体验。因此，强烈建议在您的 Next.js 应用程序中使用可观察性，以提高性能、优化资源并增强用户体验。

我们推荐使用 OpenTelemetry 来为您的应用程序进行监控。这是一种与平台无关的监控应用程序的方法，允许您在不更改代码的情况下更改可观察性提供者。有关 OpenTelemetry 及其工作原理的更多信息，请阅读官方 OpenTelemetry 文档。

本文件使用了 Span、Trace 或 Exporter 等术语，这些术语都可以在 OpenTelemetry 可观察性入门中找到。

Next.js 开箱即用地支持 OpenTelemetry 监控，这意味着我们已经对 Next.js 本身进行了监控。当您启用 OpenTelemetry 时，我们将自动将您的所有代码（如 `getStaticProps` ）包装在具有有用属性的 spans 中。

### 入门 <a href="#getting-started" id="getting-started"></a>

OpenTelemetry 是可扩展的，但正确设置它可能会非常冗长。这就是为什么我们准备了一个包 `@vercel/otel` 来帮助您快速入门。

#### [使用 `@vercel/otel` ](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#using-vercelotel) <a href="#using-vercelotel" id="using-vercelotel"></a>

要开始，请安装以下软件包：

```
npm install @vercel/otel @opentelemetry/sdk-logs @opentelemetry/api-logs @opentelemetry/instrumentation
```

接下来，在项目的根目录中创建一个自定义 `instrumentation.ts` (或 `.js` ) 文件（如果使用 `src` 文件夹，则在其中创建）：

{% code title="your-project/instrumentation.ts" %}
```typescript
import { registerOTel } from '@vercel/otel' 
export function register() {  
    registerOTel({ serviceName: 'next-app' })
}
```
{% endcode %}

请参阅 `@vercel/otel` 文档以获取其他配置选项。

> &#x20; 值得知道：
>
> * `instrumentation` 文件应位于项目的根目录中，而不是在 `app` 或 `pages` 目录内。如果您使用 `src` 文件夹，则应将文件放在 `src` 中，与 `pages` 和 `app` 一起。
> * 如果您使用 `pageExtensions` 配置选项添加后缀，则还需要更新 `instrumentation` 文件名以匹配。
> * 我们创建了一个基本的 with-opentelemetry 示例供您使用。

#### [手动 OpenTelemetry 配置](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#manual-opentelemetry-configuration) <a href="#manual-opentelemetry-configuration" id="manual-opentelemetry-configuration"></a>

`@vercel/otel` 包提供了许多配置选项，应该可以满足大多数常见用例。但如果不适合你的需求，你可以手动配置 OpenTelemetry。

首先，你需要安装 OpenTelemetry 包：

```bash
npm install @opentelemetry/sdk-node @opentelemetry/resources @opentelemetry/semantic-conventions @opentelemetry/sdk-trace-node @opentelemetry/exporter-trace-otlp-http
```

现在您可以在您的 `instrumentation.ts` 中初始化 `NodeSDK` 。与 `@vercel/otel` 不同， `NodeSDK` 与边缘运行时不兼容，因此您需要确保仅在 `process.env.NEXT_RUNTIME === 'nodejs'` 时导入它们。我们建议创建一个新文件 `instrumentation.node.ts` ，该文件仅在使用 node 时有条件地导入：

{% code title="instrumentation.ts" %}
```typescript
export async function register() {  
    if (process.env.NEXT_RUNTIME === 'nodejs') {    
        await import('./instrumentation.node.ts')  
    }
}
```
{% endcode %}

TypeScriptJavaScriptTypeScript

{% code title="instrumentation.node.ts" %}
```typescript
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http'i
mport { Resource } from '@opentelemetry/resources'
import { NodeSDK } from '@opentelemetry/sdk-node'
import { SimpleSpanProcessor } from '@opentelemetry/sdk-trace-node'
import { ATTR_SERVICE_NAME } from '@opentelemetry/semantic-conventions' 

const sdk = new NodeSDK({  
    resource: new Resource({    
        [ATTR_SERVICE_NAME]: 'next-app',  
    }),  
    spanProcessor: new SimpleSpanProcessor(new OTLPTraceExporter()),
})

sdk.start()
```
{% endcode %}

这样做相当于使用 `@vercel/otel` ，但可以修改和扩展一些 `@vercel/otel` 未公开的功能。如果需要边缘运行时支持，您将必须使用 `@vercel/otel` 。

### [测试您的仪器](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#testing-your-instrumentation) <a href="#testing-your-instrumentation" id="testing-your-instrumentation"></a>

您需要一个与兼容后端的 OpenTelemetry 收集器，以便在本地测试 OpenTelemetry 跟踪。我们建议使用我们的 OpenTelemetry 开发环境。

如果一切正常，您应该能够看到标记为 `GET /requested/pathname` 的根服务器跨度。该特定跟踪的所有其他跨度将嵌套在其下。

Next.js 追踪的跨度比默认发出的更多。要查看更多跨度，您必须设置 `NEXT_OTEL_VERBOSE=1` 。

### [部署](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#deployment) <a href="#deployment" id="deployment"></a>

#### [使用 OpenTelemetry Collector](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#using-opentelemetry-collector) <a href="#using-opentelemetry-collector" id="using-opentelemetry-collector"></a>

当您使用 OpenTelemetry Collector 部署时，可以使用 `@vercel/otel` 。它将在 Vercel 和自托管时都能工作。

[**在 Vercel 上部署**](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#deploying-on-vercel)

我们确保 OpenTelemetry 可以在 Vercel 上开箱即用。

按照 Vercel 文档将您的项目连接到可观察性提供商。

[**自托管**](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#self-hosting)

将应用部署到其他平台也很简单。您需要启动自己的 OpenTelemetry Collector，以接收和处理来自您的 Next.js 应用的遥测数据。

要做到这一点，请按照 OpenTelemetry Collector 入门指南的指示操作，该指南将引导您设置收集器并配置它以从您的 Next.js 应用程序接收数据。

一旦你的收集器启动并运行，你可以按照各自的部署指南将你的 Next.js 应用部署到你选择的平台。

#### [自定义导出器](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#custom-exporters) <a href="#custom-exporters" id="custom-exporters"></a>

OpenTelemetry Collector 不是必要的。您可以使用自定义的 OpenTelemetry 导出器与 `@vercel/otel` 或手动配置 OpenTelemetry。

### [自定义](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#custom-spans) Spans <a href="#custom-spans" id="custom-spans"></a>

您可以使用 OpenTelemetry API 添加自定义跨度。

```bash
npm install @opentelemetry/api
```

以下示例演示了一个函数，该函数获取 GitHub 星标并添加一个自定义 `fetchGithubStars` span 来跟踪获取请求的结果：

```typescript
import { trace } from '@opentelemetry/api'
 
export async function fetchGithubStars() {
  return await trace
    .getTracer('nextjs-example')
    .startActiveSpan('fetchGithubStars', async (span) => {
      try {
        return await getValue()
      } finally {
        span.end()
      }
    })
}
```

`register` 函数将在您的代码在新环境中运行之前执行。您可以开始创建新的跨度，它们应该被正确添加到导出的追踪中。

### [Next.js 中的默认跨度](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#default-spans-in-nextjs) <a href="#default-spans-in-nextjs" id="default-spans-in-nextjs"></a>

Next.js 自动为您仪器化多个跨度，以提供有关您应用程序性能的有用见解。

跨度上的属性遵循 OpenTelemetry 语义约定。我们还在 `next` 命名空间下添加了一些自定义属性：

* `next.span_name` - 重复的跨度名称
* `next.span_type` - 每个跨度类型都有一个唯一标识符
* `next.route` - 请求的路由模式 (例如， `/[param]/user` )。
* `next.rsc` (true/false) - 请求是否为 RSC 请求，例如预取。
* `next.page`
  * 这是应用路由器使用的内部值。
  * 你可以把它看作是通往特殊文件的路径（如 `page.ts` 、 `layout.ts` 、 `loading.ts` 等）
  * 仅在与 `next.route` 配对时才能用作唯一标识符，因为 `/layout` 可用于标识 `/(groupA)/layout.ts` 和 `/(groupB)/layout.ts`

#### [`[http.method] [next.route]`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#httpmethod-nextroute) <a href="#httpmethod-nextroute" id="httpmethod-nextroute"></a>

* `next.span_type`: `BaseServer.handleRequest`

这个跨度表示每个进入您的 Next.js 应用程序的请求的根跨度。它跟踪请求的 HTTP 方法、路由、目标和状态码。

&#x20; 属性:

* [常见的 HTTP 属性 ](https://opentelemetry.io/docs/reference/specification/trace/semantic_conventions/http/#common-attributes)
  * `http.method`
  * `http.status_code`
* [服务器 HTTP 属性](https://opentelemetry.io/docs/reference/specification/trace/semantic_conventions/http/#http-server-semantic-conventions)
  * `http.route`
  * `http.target`
* `next.span_name`
* `next.span_type`
* `next.route`

#### [`render route (app) [next.route]`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#render-route-app-nextroute) <a href="#render-route-app-nextroute" id="render-route-app-nextroute"></a>

* `next.span_type`: `AppRender.getBodyResult`.

这个跨度表示在应用路由器中渲染路由的过程。

&#x20; 属性:

* `next.span_name`
* `next.span_type`
* `next.route`

#### [`fetch [http.method] [http.url]`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#fetch-httpmethod-httpurl) <a href="#fetch-httpmethod-httpurl" id="fetch-httpmethod-httpurl"></a>

* `next.span_type`: `AppRender.fetch`

此跨度表示在您的代码中执行的获取请求。

&#x20; 属性:

* [常见的 HTTP 属性 ](https://opentelemetry.io/docs/reference/specification/trace/semantic_conventions/http/#common-attributes)
  * `http.method`
* [客户端 HTTP 属性 ](https://opentelemetry.io/docs/reference/specification/trace/semantic_conventions/http/#http-client)
  * `http.url`
  * `net.peer.name`
  * `net.peer.port` （仅在指定时）
* `next.span_name`
* `next.span_type`

通过在您的环境中设置 `NEXT_OTEL_FETCH_DISABLED=1` 可以关闭此跨度。当您想使用自定义的提取仪器库时，这非常有用。

#### [`executing api route (app) [next.route]`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#executing-api-route-app-nextroute) <a href="#executing-api-route-app-nextroute" id="executing-api-route-app-nextroute"></a>

* `next.span_type`: `AppRouteRouteHandlers.runHandler`.

该跨度表示应用路由器中 API 路由处理程序的执行。

&#x20; 属性:

* `next.span_name`
* `next.span_type`
* `next.route`

#### [`getServerSideProps [next.route]`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#getserversideprops-nextroute) <a href="#getserversideprops-nextroute" id="getserversideprops-nextroute"></a>

* `next.span_type`: `Render.getServerSideProps`.

此跨度表示特定路由的 `getServerSideProps` 执行。

&#x20; 属性:

* `next.span_name`
* `next.span_type`
* `next.route`

#### [`getStaticProps [next.route]`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#getstaticprops-nextroute) <a href="#getstaticprops-nextroute" id="getstaticprops-nextroute"></a>

* `next.span_type`: `Render.getStaticProps`.

此跨度表示特定路由的 `getStaticProps` 执行。

&#x20; 属性:

* `next.span_name`
* `next.span_type`
* `next.route`

#### [`render route (pages) [next.route]`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#render-route-pages-nextroute) <a href="#render-route-pages-nextroute" id="render-route-pages-nextroute"></a>

* `next.span_type`: `Render.renderDocument`.

此跨度表示为特定路由渲染文档的过程。

&#x20; 属性:

* `next.span_name`
* `next.span_type`
* `next.route`

#### [`generateMetadata [next.page]`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#generatemetadata-nextpage) <a href="#generatemetadata-nextpage" id="generatemetadata-nextpage"></a>

* `next.span_type`: `ResolveMetadata.generateMetadata`.

这个跨度表示为特定页面生成元数据的过程（一个单一的路由可以有多个这样的跨度）。

&#x20; 属性:

* `next.span_name`
* `next.span_type`
* `next.page`

#### [`resolve page components`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#resolve-page-components) <a href="#resolve-page-components" id="resolve-page-components"></a>

* `next.span_type`: `NextNodeServer.findPageComponents`.

此跨度表示为特定页面解析页面组件的过程。

&#x20; 属性:

* `next.span_name`
* `next.span_type`
* `next.route`

#### [`resolve segment modules`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#resolve-segment-modules) <a href="#resolve-segment-modules" id="resolve-segment-modules"></a>

* `next.span_type`: `NextNodeServer.getLayoutOrPageModule`.

此跨度表示为布局或页面加载代码模块。

&#x20; 属性:

* `next.span_name`
* `next.span_type`
* `next.segment`

#### [`start response`](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry#start-response) <a href="#start-response" id="start-response"></a>

* `next.span_type`: `NextNodeServer.startResponse`.

这个零长度的跨度表示响应中第一个字节被发送的时间。
