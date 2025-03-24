# 视频

本页面概述了如何在 Next.js 应用程序中使用视频，展示了如何存储和显示视频文件而不影响性能。

### 使用 `<video>` 和 `<iframe>`

可以使用 HTML `<video>` 标签将视频嵌入页面，以便直接使用视频文件，以及使用 `<iframe>` 标签嵌入外部平台托管的视频。

#### `<video>`

HTML `<video>` 标签可以嵌入自托管或直接提供的视频内容，允许对播放和外观进行完全控制。

{% code title="app/ui/video.jsx" %}
```jsx
export function Video() {
  return (
    <video width="320" height="240" controls preload="none">
      <source src="/path/to/video.mp4" type="video/mp4" />
      <track
        src="/path/to/captions.vtt"
        kind="subtitles"
        srcLang="en"
        label="English"
      />
      Your browser does not support the video tag.
    </video>
  )
}
```
{% endcode %}

#### 常见 \<video> 标签属性

<table><thead><tr><th width="184.546875">属性</th><th width="226.875">描述</th><th>示例值</th></tr></thead><tbody><tr><td><code>src</code></td><td>指定视频文件的来源。</td><td><code>&#x3C;video src="/path/to/video.mp4" /></code></td></tr><tr><td><code>width</code></td><td>设置视频播放器的宽度。</td><td><code>&#x3C;video width="320" /></code></td></tr><tr><td><code>height</code></td><td>设置视频播放器的高度。</td><td><code>&#x3C;video height="240" /></code></td></tr><tr><td><code>controls</code></td><td>如果存在，它将显示默认的播放控制集。</td><td><code>&#x3C;video controls /></code></td></tr><tr><td><code>autoPlay</code></td><td>页面加载时自动开始播放视频。注意：自动播放政策在不同浏览器中有所不同。</td><td><code>&#x3C;video autoPlay /></code></td></tr><tr><td><code>loop</code></td><td>循环播放视频。</td><td><code>&#x3C;video loop /></code></td></tr><tr><td><code>muted</code></td><td>默认情况下静音音频。通常与 <code>autoPlay</code> 一起使用。</td><td><code>&#x3C;video muted /></code></td></tr><tr><td><code>preload</code></td><td>指定视频的预加载方式。值： <code>none</code> ， <code>metadata</code> ， <code>auto</code> 。</td><td><code>&#x3C;video preload="none" /></code></td></tr><tr><td><code>playsInline</code></td><td>在 iOS 设备上启用内联播放，通常是使自动播放在 iOS Safari 上正常工作的必要条件。</td><td><code>&#x3C;video playsInline /></code></td></tr></tbody></table>

> **注意**：使用 `autoPlay` 属性时，重要的是还要包括 `muted` 属性，以确保视频在大多数浏览器中自动播放，并且 `playsInline` 属性以兼容 iOS 设备。

有关视频属性的全面列表，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video#attributes)。

#### 视频最佳实践

* 回退内容：使用 `<video>` 标签时，请在标签内包含回退内容，以便不支持视频播放的浏览器使用。
* 字幕或说明文字：为听障或听力受限的用户提供字幕或说明文字。使用 [`<track>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track) 标签和 `<video>` 元素来指定字幕文件源。
* 可访问的控件：建议使用标准的 HTML5 视频控件，以便于键盘导航和屏幕阅读器兼容性。对于更高级的需求，可以考虑使用第三方播放器，如 [react-player](https://github.com/cookpete/react-player) 或 video.js，它们提供可访问的控件和一致的浏览器体验。

#### `<iframe>`

HTML `<iframe>` 标签允许您从外部平台（如 YouTube 或 Vimeo）嵌入视频。

{% code title="app/page.jsx" %}
```jsx
export default function Page() {
  return (
    <iframe src="https://www.youtube.com/embed/19g66ezsKAg" allowFullScreen />
  )
}
```
{% endcode %}

#### 常见的 `<iframe>` 标签属性

<table><thead><tr><th width="168.83984375">属性</th><th width="307.23046875">描述</th><th> 示例值</th></tr></thead><tbody><tr><td><code>src</code></td><td>要嵌入的页面的 URL。</td><td><code>&#x3C;iframe src="https://example.com" /></code></td></tr><tr><td><code>width</code></td><td>设置 iframe 的宽度。</td><td><code>&#x3C;iframe width="500" /></code></td></tr><tr><td><code>height</code></td><td>设置 iframe 的高度。</td><td><code>&#x3C;iframe height="300" /></code></td></tr><tr><td><code>allowFullScreen</code></td><td>允许 iframe 内容以全屏模式显示。</td><td><code>&#x3C;iframe allowFullScreen /></code></td></tr><tr><td><code>sandbox</code></td><td>对 iframe 内的内容启用额外的限制。</td><td><code>&#x3C;iframe sandbox /></code></td></tr><tr><td><code>loading</code></td><td>优化加载行为（例如，延迟加载）。</td><td><code>&#x3C;iframe loading="lazy" /></code></td></tr><tr><td><code>title</code></td><td>为 iframe 提供标题以支持无障碍访问。</td><td><code>&#x3C;iframe title="Description" /></code></td></tr></tbody></table>

有关 iframe 属性的完整列表，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#attributes)。

#### 选择视频嵌入方法

在您的 Next.js 应用程序中，可以使用两种方法嵌入视频：

* 自托管或直接视频文件：使用 `<video>` 标签嵌入自托管视频，以便在需要详细控制播放器的功能和外观的情况下使用。此集成方法允许您在 Next.js 中定制和控制您的视频内容。
* 使用视频托管服务（YouTube、Vimeo 等）：对于像 YouTube 或 Vimeo 这样的在线视频托管服务，您将使用 `<iframe>` 标签嵌入它们基于 iframe 的播放器。虽然这种方法限制了对播放器的某些控制，但它提供了这些平台所提供的易用性和功能。

选择与您的应用程序需求和您希望提供的用户体验相符的嵌入方法。

#### 嵌入外部托管的视频

要从外部平台嵌入视频，您可以使用 Next.js 获取视频信息，并使用 React Suspense 处理加载时的回退状态。

**1. 创建一个用于视频嵌入的服务器组件**

第一步是创建一个服务器组件，该组件生成适当的 iframe 来嵌入视频。此组件将获取视频的源 URL 并渲染 iframe。

{% code title="app/ui/video-component.jsx" %}
```jsx
export default async function VideoComponent() {
  const src = await getVideoSrc()
 
  return <iframe src={src} allowFullScreen />
}
```
{% endcode %}

**2. 使用 React Suspense 流式传输视频组件**

在创建了嵌入视频的服务器组件后，下一步是使用 React Suspense [流式传输](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming)该组件。

{% code title="app/page.jsx" %}
```jsx
import { Suspense } from 'react'
import VideoComponent from '../ui/VideoComponent.jsx'
 
export default function Page() {
  return (
    <section>
      <Suspense fallback={<p>Loading video...</p>}>
        <VideoComponent />
      </Suspense>
      {/* Other content of the page */}
    </section>
  )
}
```
{% endcode %}

> **注意**：
>
> 在嵌入来自外部平台的视频时，请考虑以下最佳实践：
>
> * 确保视频嵌入是响应式的。使用 CSS 使 iframe 或视频播放器适应不同的屏幕尺寸。
> * 根据网络条件实施视频加载策略，特别是对于数据计划有限的用户。

这种方法可以提供更好的用户体验，因为它防止页面阻塞，这意味着用户可以在视频组件流式传输时与页面进行交互。

为了提供更具吸引力和信息性的加载体验，考虑使用加载骨架作为后备 UI。因此，您可以显示一个类似于视频播放器的骨架，而不是简单的加载消息。

{% code title="app/page.jsx" %}
```jsx
import { Suspense } from 'react'
import VideoComponent from '../ui/VideoComponent.jsx'
import VideoSkeleton from '../ui/VideoSkeleton.jsx'
 
export default function Page() {
  return (
    <section>
      <Suspense fallback={<VideoSkeleton />}>
        <VideoComponent />
      </Suspense>
      {/* Other content of the page */}
    </section>
  )
}
```
{% endcode %}

### 自托管视频

自托管视频可能出于几个原因更为可取：

* 完全控制和独立性：自托管使您能够直接管理视频内容，从播放到外观，确保完全拥有和控制，摆脱外部平台的限制。
* 针对特定需求的定制：非常适合独特的需求，例如动态背景视频，它允许根据设计和功能需求进行量身定制。
* 性能和可扩展性考虑：选择既高性能又可扩展的存储解决方案，以有效支持不断增加的流量和内容大小。
* 成本与集成：平衡存储和带宽的成本与在您的 Next.js 框架及更广泛的技术生态系统中实现简单集成的需求。

#### 使用 Vercel Blob 进行视频托管

[Vercel Blob](https://vercel.com/docs/storage/vercel-blob?utm_source=next-site\&utm_medium=docs\&utm_campaign=next-website) 提供了一种高效的视频托管方式，提供了一个可扩展的云存储解决方案，适用于 Next.js。以下是如何使用 Vercel Blob 托管视频的方法：

**1. 将视频上传到 Vercel Blob**

在您的 Vercel 控制面板中，导航到“存储”选项卡并选择您的 Vercel Blob 存储。在 Blob 表的右上角，找到并点击“上传”按钮。然后，选择您希望上传的视频文件。上传完成后，视频文件将出现在 Blob 表中。

另外，您可以使用服务器操作上传视频。有关详细说明，请参阅 Vercel 文档中的服务器端上传部分。Vercel 还支持客户端上传。这种方法在某些用例中可能更可取。

**2. 在 Next.js 中显示视频**

一旦视频上传并存储，您可以在您的 Next.js 应用程序中显示它。以下是如何使用 `<video>` 标签和 React Suspense 来实现这一点的示例：

{% code title="app/page.jsx" %}
```jsx
import { Suspense } from 'react'
import { list } from '@vercel/blob'
 
export default function Page() {
  return (
    <Suspense fallback={<p>Loading video...</p>}>
      <VideoComponent fileName="my-video.mp4" />
    </Suspense>
  )
}
 
async function VideoComponent({ fileName }) {
  const { blobs } = await list({
    prefix: fileName,
    limit: 1,
  })
  const { url } = blobs[0]
 
  return (
    <video controls preload="none" aria-label="Video player">
      <source src={url} type="video/mp4" />
      Your browser does not support the video tag.
    </video>
  )
}
```
{% endcode %}

在这种方法中，页面使用视频的 `@vercel/blob` URL 来通过 `VideoComponent` 显示视频。使用 React Suspense 来显示一个后备内容，直到视频 URL 被获取并且视频准备好显示。

#### 为视频增加字幕

如果您有视频的字幕，可以通过在您的 `<video>` 标签内使用 `<track>` 元素轻松添加它们。您可以以与视频文件类似的方式从 Vercel Blob 获取字幕文件。以下是如何更新 `<VideoComponent>` 以包含字幕。

{% code title="app/page.jsx" %}
```jsx
async function VideoComponent({ fileName }) {
  const { blobs } = await list({
    prefix: fileName,
    limit: 2,
  })
  const { url } = blobs[0]
  const { url: captionsUrl } = blobs[1]
 
  return (
    <video controls preload="none" aria-label="Video player">
      <source src={url} type="video/mp4" />
      <track src={captionsUrl} kind="subtitles" srcLang="en" label="English" />
      Your browser does not support the video tag.
    </video>
  )
}
```
{% endcode %}

通过采用这种方法，您可以有效地自我托管并将视频集成到您的 Next.js 应用程序中。

### 资源

要继续学习有关视频优化和最佳实践的更多信息，请参考以下资源：

* 理解视频格式和编解码器：选择合适的格式和编解码器，例如 MP4 以获得兼容性，或 WebM 以进行网络优化，以满足您的视频需求。有关更多详细信息，请参阅 [Mozilla 关于视频编解码器的指南](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Video_codecs)。
* 视频压缩：使用像 FFmpeg 这样的工具有效压缩视频，平衡质量与文件大小。了解压缩技术，请访问 [FFmpeg 的官方网站](https://www.ffmpeg.org/)。
* 分辨率和比特率调整：根据观看平台调整分辨率和比特率，移动设备使用较低的设置。
* 内容分发网络（CDN）：利用 CDN 来提高视频传输速度并管理高流量。当使用一些存储解决方案时，例如 Vercel Blob，CDN 功能会自动为您处理。了解更多关于 CDN 及其好处的信息。

探索这些视频流媒体平台，以将视频集成到您的 Next.js 项目中：

#### 开源的 `next-video` 组件

* 为 Next.js 提供了一个 `<Video>` 组件，兼容包括 Vercel Blob、S3、Backblaze 和 Mux 在内的各种托管服务。
* 使用 `next-video.dev` 与不同托管服务的详细文档。

#### Cloudinary 集成

* 使用 Cloudinary 与 Next.js 的[官方文档和集成指南](https://next.cloudinary.dev/)。
* 引入一个 `<CldVideoPlayer>` 组件以支持嵌入视频。
* 找到将 Cloudinary 与 Next.js 集成的示例，包括[自适应比特率流](https://github.com/cloudinary-community/cloudinary-examples/tree/main/examples/nextjs-cldvideoplayer-abr)。
* 其他 [Cloudinary 库](https://cloudinary.com/documentation)，包括 Node.js SDK，也可用。

#### Mux 集成 API

* Mux 提供了一个用于创建视频课程的入门模板，适用于 Mux 和 Next.js。
* 了解 Mux 对于在您的 Next.js 应用程序中嵌入高性能视频的建议。
* 探索一个示例项目，展示 Mux 与 Next.js 的结合。

#### Fastly

* 了解更多关于将 Fastly 的视频点播和流媒体解决方案集成到 Next.js 中的信息。

#### ImageKit.io 集成

* 查看官方快速入门指南，以了解如何将 ImageKit 与 Next.js 集成。
* 此集成提供 `<IKVideo>` 组件，提供无缝的视频支持。
* 您还可以探索其他 ImageKit 库，例如也可用的 Node.js SDK。
