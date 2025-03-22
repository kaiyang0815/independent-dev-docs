# 路由组

在 `app` 目录中，嵌套文件夹通常映射到 URL 路径。然而，您可以将文件夹标记为**路由组**，以防止该文件夹包含在路由的 URL 路径中。

这使您能够将路由段和项目文件组织成逻辑组，而不影响 URL 路径结构。

路由组的用途包括：

* 将路由组织成组，例如按网站部分、意图或团队。
* 在同一路由段级别启用嵌套布局：
  * [在同一段中创建多个嵌套布局，包括多个根布局](https://nextjs.org/docs/app/building-your-application/routing/route-groups#creating-multiple-root-layouts)
  * [向公共段中的一组路由添加布局](https://nextjs.org/docs/app/building-your-application/routing/route-groups#opting-specific-segments-into-a-layout)
* [向公共段中的特定路由添加加载骨架](https://nextjs.org/docs/app/building-your-application/routing/route-groups#opting-for-loading-skeletons-on-a-specific-route)

### 约定

可以通过将文件夹名称用括号括起来来创建路由组： `(folderName)`&#x20;

### 示例

#### 组织路由而不影响URL路径

为了在不影响 URL 的情况下组织路由，创建一个组以将相关路由放在一起。括号中的文件夹将从 URL 中省略（例如 `(marketing)` 或 `(shop)` ）。

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

尽管 `(marketing)` 和 `(shop)` 内的路由共享相同的 URL 层次结构，但您可以通过在它们的文件夹中添加 `layout.js` 文件来为每个组创建不同的布局。

#### 将特定段落纳入布局中

要将特定路由纳入到布局中，请创建一个新的路由组（例如 `(shop)` ），并将共享相同布局的路由移动到该组中（例如 `account` 和 `cart` ）。组外的路由将不共享该布局（例如 `checkout` ）。

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

#### 在路由上选择加载骨架屏

要通过 `loading.js` 文件在特定路由上应用加载骨架，请创建一个新的路由组（例如， `/(overview)` ），然后将您的 `loading.tsx` 移动到该路由组内。

现在， `loading.tsx` 文件将仅适用于您的 dashboard → overview 页面，而不是所有 dashboard 页面，并且不会影响 URL 路径结构。

#### 创建多个根布局

要创建多个根布局，请删除顶级 `layout.js` 文件，并在每个路由组内添加 `layout.js` 文件。这对于将应用程序划分为具有完全不同的 UI 或体验的部分非常有用。需要将 `<html>`和 `<body>` 标签添加到每个根布局中。

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

在上面的示例中， `(marketing)` 和 `(shop)` 都有自己的根布局。

> 需要知道的是：
>
> * 路由组的命名除了用于组织之外没有特殊意义。它们不影响 URL 路径。
> * 包含路由组的路由不应解析为与其他路由相同的 URL 路径。例如，由于路由组不会影响 URL 结构， `(marketing)/about/page.js` 和 `(shop)/about/page.js` 都会解析为 `/about` ，并导致错误。
> * 如果您使用多个根布局而没有顶级 `layout.js` 文件，您的主页 `page.js` 文件应在其中一个路由组中定义，例如： `app/(marketing)/page.js` 。
> * 在多个根布局之间进行导航将导致整个页面重新加载（与客户端导航相对）。例如，从使用 `app/(shop)/layout.js` 的 `/cart` 导航到使用 `app/(marketing)/layout.js` 的 `/blog` 将导致页面完全加载。这仅适用于多个根布局。

