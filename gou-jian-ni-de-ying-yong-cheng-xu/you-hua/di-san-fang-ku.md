# 第三方库

`@next/third-parties` 是一个库，提供了一系列组件和工具，改善在您的 Next.js 应用程序中加载流行第三方库的性能和开发者体验。

`@next/third-parties` 提供的所有第三方集成都已针对性能和易用性进行了优化。

### [入门](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#getting-started) <a href="#getting-started" id="getting-started"></a>

要开始，请安装 `@next/third-parties` 库：

&#x20; 终端

```
npm install @next/third-parties@latest next@latest
```

`@next/third-parties` 目前是一个正在积极开发的实验性库。我们建议在我们努力添加更多第三方集成时，使用最新或金丝雀标志安装。

### [谷歌第三方](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#google-third-parties) <a href="#google-third-parties" id="google-third-parties"></a>

所有来自谷歌的支持的第三方库可以从 `@next/third-parties/google` 导入。

#### [谷歌标签管理器](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#google-tag-manager) <a href="#google-tag-manager" id="google-tag-manager"></a>

`GoogleTagManager` 组件可用于在您的页面上实例化谷歌标签管理器容器。默认情况下，它会在页面水合后获取原始内联脚本。

要在所有路由中加载谷歌标签管理器，请直接在您的根布局中包含该组件并传入您的 GTM 容器 ID：

{% code title="app/layout.tsx" %}
```tsx
import { GoogleTagManager } from '@next/third-parties/google'
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <GoogleTagManager gtmId="GTM-XYZ" />
      <body>{children}</body>
    </html>
  )
}
```
{% endcode %}

要加载 Google Tag Manager 以支持单个路由，请在您的页面文件中包含该组件：

{% code title="app/page.js" %}
```
import { GoogleTagManager } from '@next/third-parties/google' export default function Page() {  return <GoogleTagManager gtmId="GTM-XYZ" />}
```
{% endcode %}

[  **发送事件**](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#sending-events)

`sendGTMEvent` 函数可以通过使用 `dataLayer` 对象发送事件来跟踪用户在您页面上的交互。要使此功能正常工作， `<GoogleTagManager />` 组件必须包含在父布局、页面或组件中，或直接在同一文件中。

{% code title="app/page.js" %}
```
'use client' import { sendGTMEvent } from '@next/third-parties/google' export function EventButton() {  return (    <div>      <button        onClick={() => sendGTMEvent({ event: 'buttonClicked', value: 'xyz' })}      >        Send Event      </button>    </div>  )}
```
{% endcode %}

请参考标签管理器开发文档，以了解可以传递到函数中的不同变量和事件。

[**服务器端标记**](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#server-side-tagging)

如果您正在使用服务器端标签管理器，并从您的标签服务器提供 `gtm.js` 脚本，您可以使用 `gtmScriptUrl` 选项来指定脚本的 URL。

[**选项**](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#options)

传递给 Google Tag Manager 的选项。有关选项的完整列表，请阅读 Google Tag Manager 文档。

|   名称            |   类型 |   描述                                                       |
| --------------- | ---- | ---------------------------------------------------------- |
| `gtmId`         |   必需 | 您的 GTM 容器 ID。通常以 `GTM-` 开头。                                |
| `gtmScriptUrl`  |   可选 | GTM 脚本 URL。默认为 `https://www.googletagmanager.com/gtm.js` 。 |
| `dataLayer`     |   可选 | 用于实例化容器的数据层对象。                                             |
| `dataLayerName` |   可选 | 数据层的名称。默认为 `dataLayer` 。                                   |
| `auth`          |   可选 | 环境代码片段的认证参数值 ( `gtm_auth` )。                               |
| `preview`       |   可选 | 环境片段的预览参数值 ( `gtm_preview` )。                              |

#### [谷歌分析](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#google-analytics) <a href="#google-analytics" id="google-analytics"></a>

`GoogleAnalytics` 组件可以通过 Google 标签 ( `gtag.js` ) 将 Google Analytics 4 包含到您的页面中。默认情况下，它会在页面水合后获取原始脚本。

> 建议：如果您的应用程序中已经包含 Google Tag Manager，您可以直接使用它配置 Google Analytics，而无需将 Google Analytics 作为单独的组件包含。请参考文档以了解 Tag Manager 和 `gtag.js` 之间的区别。

要在所有路由中加载 Google Analytics，请直接在根布局中包含该组件并传入您的测量 ID：

{% code title="app/layout.tsx" %}
```
import { GoogleAnalytics } from '@next/third-parties/google' export default function RootLayout({  children,}: {  children: React.ReactNode}) {  return (    <html lang="en">      <body>{children}</body>      <GoogleAnalytics gaId="G-XYZ" />    </html>  )}
```
{% endcode %}

要为单一路由加载 Google Analytics，请在您的页面文件中包含该组件：

{% code title="app/page.js" %}
```
import { GoogleAnalytics } from '@next/third-parties/google' export default function Page() {  return <GoogleAnalytics gaId="G-XYZ" />}
```
{% endcode %}

[**发送事件**](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#sending-events-1)

`sendGAEvent` 函数可用于通过使用 `dataLayer` 对象发送事件来测量用户在页面上的交互。为了使此函数正常工作，必须在父布局、页面或组件中包含 `<GoogleAnalytics />` 组件，或直接包含在同一个文件中。

{% code title="app/page.js" %}
```
'use client' import { sendGAEvent } from '@next/third-parties/google' export function EventButton() {  return (    <div>      <button        onClick={() => sendGAEvent('event', 'buttonClicked', { value: 'xyz' })}      >        Send Event      </button>    </div>  )}
```
{% endcode %}

请参阅 Google Analytics 开发者文档以了解有关事件参数的更多信息。

[**页面浏览量追踪**](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#tracking-pageviews)

谷歌分析在浏览器历史状态更改时会自动跟踪页面浏览量。这意味着在 Next.js 路由之间的客户端导航将发送页面浏览数据而无需任何配置。

为了确保客户端导航被正确测量，请在您的管理面板中验证“增强测量”属性已启用，并选中“基于浏览器历史事件的页面更改”复选框。

> 注意：如果您决定手动发送页面浏览事件，请确保禁用默认的页面浏览测量，以避免重复数据。有关更多信息，请参阅 Google Analytics 开发者文档。

[**选项**](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#options-1)

传递给 `<GoogleAnalytics>` 组件的选项。

| 名称              | 类型   | 描述                       |
| --------------- | ---- | ------------------------ |
| `gaId`          |   必需 | 您的测量 ID。通常以 `G-` 开头。     |
| `dataLayerName` |   可选 | 数据层的名称。默认为 `dataLayer` 。 |
| `nonce`         |   可选 |   一个随机数。                 |

#### [Google Maps 嵌入](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#google-maps-embed) <a href="#google-maps-embed" id="google-maps-embed"></a>

`GoogleMapsEmbed` 组件可以用于在您的页面上添加 Google Maps Embed。默认情况下，它使用 `loading` 属性来延迟加载下面的嵌入内容。

{% code title="app/page.js" %}
```
import { GoogleMapsEmbed } from '@next/third-parties/google' export default function Page() {  return (    <GoogleMapsEmbed      apiKey="XYZ"      height={200}      width="100%"      mode="place"      q="Brooklyn+Bridge,New+York,NY"    />  )}
```
{% endcode %}

[**选项**](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#options-2)

传递给 Google Maps Embed 的选项。有关完整选项列表，请阅读 Google Map Embed 文档。

| 名称                | 类型   | 描述                                                                                                |
| ----------------- | ---- | ------------------------------------------------------------------------------------------------- |
| `apiKey`          |   必需 |   您的 API 密钥。                                                                                      |
| `mode`            |   必需 | [  地图模式](https://developers.google.com/maps/documentation/embed/embedding-map#choosing_map_modes) |
| `height`          |   可选 | 嵌套的高度。默认为 `auto` 。                                                                                |
| `width`           |   可选 | 嵌套的宽度。默认为 `auto` 。                                                                                |
| `style`           |   可选 | 将样式传递给 iframe。                                                                                    |
| `allowfullscreen` |   可选 | 允许某些地图部分全屏的属性。                                                                                    |
| `loading`         |   可选 | 默认为懒加载。如果您知道您的嵌入内容会在可视区域内，请考虑更改。                                                                  |
| `q`               |   可选 | 定义地图标记位置。这可能根据地图模式而有所要求。                                                                          |
| `center`          |   可选 | 定义地图视图的中心。                                                                                        |
| `zoom`            |   可选 | 设置地图的初始缩放级别。                                                                                      |
| `maptype`         |   可选 | 定义要加载的地图瓦片类型。                                                                                     |
| `language`        |   可选 | 定义用于用户界面元素和地图图层上标签显示的语言。                                                                          |
| `region`          |   可选 | 根据地缘政治敏感性定义要显示的适当边界和标签。                                                                           |

#### [YouTube 嵌入](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#youtube-embed) <a href="#youtube-embed" id="youtube-embed"></a>

`YouTubeEmbed` 组件可以用来加载和显示 YouTube 嵌入。这个组件通过在底层使用 `lite-youtube-embed` 来加快加载速度。

{% code title="app/page.js" %}
```
import { YouTubeEmbed } from '@next/third-parties/google' export default function Page() {  return <YouTubeEmbed videoid="ogfYd705cRs" height={400} params="controls=0" />}
```
{% endcode %}

[**选项**](https://nextjs.org/docs/app/building-your-application/optimizing/third-party-libraries#options-3)

|   名称        |   类型 |   描述                                                                                                       |
| ----------- | ---- | ---------------------------------------------------------------------------------------------------------- |
| `videoid`   |   必需 |   YouTube 视频 ID。                                                                                           |
| `width`     |   可选 | 视频容器的宽度。默认为 `auto`                                                                                         |
| `height`    |   可选 | 视频容器的高度。默认为 `auto`                                                                                         |
| `playlabel` |   可选 | 用于可访问性的播放按钮的视觉隐藏标签。                                                                                        |
| `params`    |   可选 | <p>这里定义的视频播放器参数。<br>参数作为查询参数字符串传递。<br>  例如: <code>params="controls=0&#x26;start=10&#x26;end=30"</code></p> |
| `style`     |   可选 | 用于为视频容器应用样式。                                                                                               |

\
