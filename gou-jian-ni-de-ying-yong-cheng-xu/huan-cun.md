# 缓存

Next.js 通过缓存渲染工作和数据请求来提高您应用程序的性能并降低成本。此页面提供了对 Next.js 缓存机制的深入了解，您可以使用的 API 来配置它们，以及它们如何相互作用。

> 注意：此页面帮助您理解 Next.js 在背后是如何工作的，但对于高效使用 Next.js 并不是必需的知识。大多数 Next.js 的缓存启发式由您的 API 使用情况决定，并为零或最小配置提供最佳性能的默认值。如果您想跳到示例，请从[这里](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching)开始。

## 概述

这是不同缓存机制及其目的的高级概述：

|  机制                                                                                                           | 返回什么         | 何处调用 | 目的               | 持续时间       |
| ------------------------------------------------------------------------------------------------------------- | ------------ | ---- | ---------------- | ---------- |
| [请求记忆化](https://nextjs.org/docs/app/building-your-application/caching#request-memoization)                    | 函数的返回值       | 服务器  | 在 React 组件树中重用数据 | 每个请求的生命周期  |
| [数据缓存](https://nextjs.org/docs/app/building-your-application/caching#data-cache)                              | 数据           | 服务器  | 在用户请求和部署之间存储数据   | 永久（可以重新验证） |
| [完整路由缓存](https://nextjs.org/docs/app/building-your-application/caching#full-route-cache)                      | TML 和 RSC 负载 | 服务器  | 减少渲染成本并提高性能      | 永久（可以重新验证） |
| [Router Cache  路由器缓存](https://nextjs.org/docs/app/building-your-application/caching#client-side-router-cache) | RSC 有效负载     | 客户端  | 减少导航时的服务器请求      | 用户会话或基于时间  |

默认情况下，Next.js 将尽可能缓存以提高性能并降低成本。这意味着路由是静态渲染的，数据请求被缓存，除非您选择不缓存。下面的图表显示了默认的缓存行为：当路由在构建时静态渲染以及当静态路由首次访问时。

<figure><picture><source srcset="../.gitbook/assets/image (8).png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/image (3).png" alt=""></picture><figcaption></figcaption></figure>

缓存行为取决于路由是静态渲染还是动态渲染，数据是缓存还是未缓存，以及请求是初始访问的一部分还是后续导航。根据您的用例，您可以为单个路由和数据请求配置缓存行为。

### 请求记忆化

Next.js 扩展了 `fetch` API，以自动记忆具有相同 URL 和选项的请求。这意味着您可以在 React 组件树中的多个地方调用同一数据的 fetch 函数，同时只执行一次。

<figure><picture><source srcset="../.gitbook/assets/image (9).png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/image (1) (1).png" alt=""></picture><figcaption></figcaption></figure>

例如，如果您需要在一个路由中使用相同的数据（例如在 Layout、Page 和多个组件中），您不必在树的顶部获取数据，并在组件之间传递 props。相反，您可以在需要数据的组件中获取数据，而无需担心因在网络上对相同数据进行多次请求而带来的性能影响。

{% code title="app/example.tsx" %}
```tsx
async function getItem() {
  // `fetch` 函数会自动进行记忆化，结果
  // 被缓存
  const res = await fetch('https://.../item/1')
  return res.json()
}
 
// 此函数被调用了两次，但仅在第一次执行
const item = await getItem() // 缓存未命中
 
// 第二个调用可以在您的路线中的任何地方
const item = await getItem() // 缓存命中
```
{% endcode %}

**请求记忆化是如何工作的**

<figure><picture><source srcset="../.gitbook/assets/image (10).png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/image (2) (1).png" alt=""></picture><figcaption></figcaption></figure>

* 在渲染路由时，第一次调用特定请求时，其结果将不在内存中，它将是一个缓存 `MISS` 。
* 因此，函数将被执行，数据将从外部源获取，结果将存储在内存中。
* 在同一渲染过程中后续对请求的函数调用将是一个缓存 `HIT` ，数据将从内存中返回，而无需执行函数。
* 一旦路由被渲染并且渲染过程完成，内存将被“重置”，所有请求的记忆化条目将被清除。

> 注意：
>
> * 请求记忆化是一个 React 特性，而不是 Next.js 特性。它在这里包含是为了展示它如何与其他缓存机制交互。
> * 记忆化仅适用于 `GET` 方法在 `fetch` 请求中。
> * 记忆化仅适用于 React 组件树，这意味着：
>   * 它适用于 `fetch` 请求在 `generateMetadata` 、 `generateStaticParams`、布局、页面和其他服务器组件中。
>   * 它不适用于 `fetch` 请求在路由处理程序中，因为它们不是 React 组件树的一部分。
> * 对于 `fetch` 不适用的情况（例如某些数据库客户端、CMS 客户端或 GraphQL 客户端），您可以使用 React `cache` 函数来记忆函数。

#### 持续时间

缓存持续到服务器请求的生命周期，直到 React 组件树完成渲染。

#### 重新验证

由于备忘录不在服务器请求之间共享，仅在渲染期间适用，因此无需重新验证它。

#### 选择退出

备忘录仅适用于 `GET` 方法在 `fetch` 请求中，其他方法，如 `POST` 和 `DELETE` ，则不进行备忘录。这种默认行为是 React 的优化，我们不建议选择退出。

要管理单个请求，您可以使用 `AbortController` 中的 `signal` 属性。然而，这不会使请求退出备忘录，而是中止正在进行的请求。

{% code title="app/example.js" %}
```javascript
const { signal } = new AbortController()
fetch(url, { signal })
```
{% endcode %}

### 数据缓存

Next.js 具有内置的数据缓存，可以在传入的服务器请求和部署之间持久化数据获取的结果。这是可能的，因为 Next.js 扩展了原生 `fetch` API，允许服务器上的每个请求设置自己的持久缓存语义。

> 注意：在浏览器中， `fetch` 的 `cache` 选项指示请求将如何与浏览器的 HTTP 缓存交互，在 Next.js 中， `cache` 选项指示服务器端请求将如何与服务器的数据缓存交互。

您可以使用 `fetch` 的 `cache` 和 `next.revalidate` 选项来配置缓存行为。

**数据缓存的工作原理**

<figure><picture><source srcset="../.gitbook/assets/image (11).png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/image (3) (1).png" alt=""></picture><figcaption></figcaption></figure>

* 第一次在渲染期间调用带有 `'force-cache'` 选项的 `fetch` 请求时，Next.js 会检查数据缓存以获取缓存的响应。
* 如果找到缓存的响应，则立即返回并进行[记忆](huan-cun.md#qing-qiu-ji-yi-hua)。
* 如果未找到缓存的响应，则请求将发送到数据源，结果存储在数据缓存中，并进行记忆。
* 对于未缓存的数据（例如，未定义 `cache` 选项或使用 `{ cache: 'no-store' }` ），结果始终从数据源获取，并进行记忆。
* 无论数据是缓存的还是未缓存的，请求总是被记忆化，以避免在 React 渲染过程中对相同数据发出重复请求。

> **数据缓存与请求记忆化之间的区别**
>
> 虽然这两种缓存机制通过重用缓存数据来提高性能，但数据缓存在传入请求和部署之间是持久的，而记忆化仅在请求的生命周期内有效。

#### 持续时间

数据缓存在传入请求和部署之间是持久的，除非您重新验证或选择退出。

#### 重新验证

缓存数据可以通过两种方式重新验证，使用：

* **基于时间的重新验证**：在经过一定时间后并且发出新请求时重新验证数据。这对于不经常变化且新鲜度不是那么关键的数据非常有用。
* **按需重新验证**：根据事件（例如表单提交）重新验证数据。按需重新验证可以使用基于标签或基于路径的方法一次性重新验证一组数据。当您希望尽快显示最新数据时（例如，当您的无头 CMS 内容更新时），这非常有用。

**基于时间的重新验证**

要在定时间隔内重新验证数据，您可以使用 `next.revalidate` 的 `fetch`选项来设置资源的缓存生命周期（以秒为单位）。

```typescript
// 每小时最多重新验证一次
fetch('https://...', { next: { revalidate: 3600 } })
```

或者，您可以使用路由段配置选项来配置段中的所有 `fetch` 请求或在无法使用 `fetch` 的情况下。

**基于时间的重新验证如何工作**

<figure><picture><source srcset="../.gitbook/assets/image (12).png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/image (4).png" alt=""></picture><figcaption></figcaption></figure>

* 第一次调用带有 `revalidate` 的获取请求时，将从外部数据源获取数据并存储在数据缓存中。
* 在指定的时间范围内（例如 60 秒）调用的任何请求将返回缓存的数据。
* 在时间范围结束后，下一个请求仍将返回缓存的（现在过期的）数据。
  * Next.js 将在后台触发数据的重新验证。
  * 一旦数据成功获取，Next.js 将使用新数据更新数据缓存。
  * 如果后台重新验证失败，之前的数据将保持不变。

这类似于 [stale-while-revalidate](https://web.dev/articles/stale-while-revalidate) 行为。

**按需重新验证**

数据可以按需通过路径 ( `revalidatePath` ) 或缓存标签 ( `revalidateTag`) 进行重新验证。

**按需重新验证的工作原理**

<figure><picture><source srcset="../.gitbook/assets/image (13).png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/image (5).png" alt=""></picture><figcaption></figcaption></figure>

* 第一次调用 `fetch` 请求时，数据将从外部数据源获取并存储在数据缓存中。
* 当触发按需重新验证时，适当的缓存条目将从缓存中清除。
  * 这与基于时间的重新验证不同，后者在获取新数据之前会将过期数据保留在缓存中。
* 下次请求时，它将再次是缓存 `MISS` ，数据将从外部数据源获取并存储在数据缓存中。

#### 选择退出

如果您不想缓存来自 `fetch` 的响应，可以执行以下操作：

```typescript
let data = await fetch('https://api.vercel.app/blog', { cache: 'no-store' })
```

### 完整路由缓存

> **相关术语：**
>
> 您可能会看到术语**自动静态优化**、**静态网站生成**或**静态渲染**被交替使用，以指代在构建时渲染和缓存应用程序路由的过程。

Next.js 在构建时自动渲染和缓存路由。这是一种优化，允许您提供缓存的路由，而不是在每个请求时在服务器上渲染，从而实现更快的页面加载。

要理解完整路由缓存的工作原理，了解 React 如何处理渲染以及 Next.js 如何缓存结果是很有帮助的：

#### 1. 服务器上的 React 渲染

在服务器上，Next.js 使用 React 的 API 来协调渲染。渲染工作被分成多个部分：按单独的路由段和 Suspense 边界。

每个块分两步渲染：

1. React 将服务器组件渲染为一种特殊的数据格式，优化用于流式传输，称为 React 服务器组件有效载荷。
2. Next.js 使用 React 服务器组件有效载荷和客户端组件 JavaScript 指令在服务器上渲染 HTML。

这意味着我们不必等到所有内容渲染完成后再缓存工作或发送响应。相反，我们可以在工作完成时流式传输响应。

> **什么是 React 服务器组件有效负载？**
>
> React 服务器组件有效负载是渲染的 React 服务器组件树的紧凑二进制表示。它被 React 在客户端用于更新浏览器的 DOM。React 服务器组件有效负载包含：
>
> * 服务器组件的渲染结果
> * 用于标记客户端组件应该渲染的位置以及对其 JavaScript 文件的引用
> * 从服务器组件传递到客户端组件的任何 props
>
> 要了解更多信息，请参阅[服务器组件](https://nextjs.org/docs/app/building-your-application/rendering/server-components)文档。

#### 2. Next.js 服务器上的缓存（完整路由缓存）

<figure><picture><source srcset="../.gitbook/assets/image (14).png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/image (6).png" alt=""></picture><figcaption></figcaption></figure>

Next.js 的默认行为是在服务器上缓存路由的渲染结果（React 服务器组件负载和 HTML）。这适用于在构建时或重新验证期间静态渲染的路由。

#### 3. 客户端的 React Hydation 和 Reconciliation

在请求时，在客户端：

1. HTML 用于立即显示客户端和服务器组件的快速非交互式初始预览。
2. React 服务器组件有效载荷用于协调客户端和渲染的服务器组件树，并更新 DOM。
3. JavaScript 指令用于为客户端组件提供水合，并使应用程序具有交互性。

#### 4. Next.js 在客户端的缓存（路由缓存）

React 服务器组件有效载荷存储在客户端路由器缓存中 - 一个单独的内存缓存，按单个路由段分割。这个路由器缓存用于通过存储之前访问的路由和预取未来的路由来改善导航体验。

#### 5. 后续导航

在后续的导航或预取过程中，Next.js 将检查 React 服务器组件有效负载是否存储在路由器缓存中。如果是，它将跳过向服务器发送新请求。

如果路由段不在缓存中，Next.js 将从服务器获取 React 服务器组件有效负载，并在客户端填充路由器缓存。

#### 静态和动态渲染

在构建时，路由是否被缓存取决于它是静态渲染还是动态渲染。静态路由默认被缓存，而动态路由在请求时渲染，不被缓存。

此图显示了静态和动态渲染路由之间的区别，以及缓存和未缓存数据的情况：

<figure><picture><source srcset="../.gitbook/assets/image (15).png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/image (7).png" alt=""></picture><figcaption></figcaption></figure>

了解更多关于静态和动态渲染的信息。

#### 持续时间

默认情况下，完整路由缓存是持久的。这意味着渲染输出在用户请求之间被缓存。

#### 失效

您可以通过两种方式使完整路由缓存失效：

* 重新验证数据：重新验证数据缓存将通过在服务器上重新渲染组件并缓存新的渲染输出来使路由器缓存失效。
* **重新部署**：与在部署之间持续存在的数据缓存不同，完整路由缓存会在新部署时被清除。

#### 选择退出

您可以选择退出完整路由缓存，换句话说，可以通过以下方式为每个传入请求动态渲染组件：

* **使用**[**动态 API**](https://nextjs.org/docs/app/building-your-application/caching#dynamic-apis)：这将使路由从完整路由缓存中排除，并在请求时动态渲染。数据缓存仍然可以使用。
* **使用 `dynamic = 'force-dynamic'` 或 `revalidate = 0` 路由段配置选项**：这将跳过完整路由缓存和数据缓存。这意味着组件将在每个传入的服务器请求中被渲染并获取数据。路由器缓存仍然适用，因为它是客户端缓存。
* **选择退出**[**数据缓存**](https://nextjs.org/docs/app/building-your-application/caching#data-cache)：如果一个路由有一个 `fetch` 请求且未被缓存，这将使该路由退出完整路由缓存。特定 `fetch` 请求的数据将为每个传入请求获取。其他未选择退出缓存的 `fetch` 请求仍将被缓存到数据缓存中。这允许缓存和未缓存数据的混合使用。

### 客户端路由器缓存

Next.js 有一个内存中的客户端路由缓存，用于存储按布局、加载状态和页面拆分的路由段的 RSC 负载。

当用户在路由之间导航时，Next.js 会缓存访问过的路由段，并预取用户可能导航到的路由。这导致即时的前进/后退导航，在导航之间没有完整页面重新加载，并且保留 React 状态和浏览器状态。

使用路由器缓存：

* **布局**在导航时被缓存并重用（[部分渲染](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#4-partial-rendering)）。
* **加载状态**在导航时被缓存并重用，以实现[即时导航](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming#instant-loading-states)。
* **页面**默认不缓存，但在浏览器的前进和后退导航中会被重用。您可以通过使用实验性的 [`staleTimes`](https://nextjs.org/docs/app/api-reference/config/next-config-js/staleTimes) 配置选项来启用页面段的缓存。

> **注意**：此缓存专门适用于 Next.js 和服务器组件，且与浏览器的 bfcache 不同，尽管它有类似的结果。

#### 持续时间

缓存存储在浏览器的临时内存中。路由器缓存持续时间的两个因素是：

* **会话**：缓存在导航中持续存在。然而，在页面刷新时会被清除。
* **自动失效期**：布局和加载状态的缓存在特定时间后会自动失效。持续时间取决于资源的预取方式，以及资源是否是静态生成的：
  * **默认预取** ( `prefetch={null}` 或未指定)：动态页面不缓存，静态页面缓存 5 分钟。
  * **完全预取** ( `prefetch={true}` 或 `router.prefetch` ): 静态和动态页面均为 5 分钟。

虽然页面刷新会清除所有缓存的片段，但自动失效时期仅影响从预取时起的单个片段。

> **注意**：实验性的 `staleTimes` 配置选项可以用来调整上面提到的自动失效时间。

#### 失效

有两种方法可以使路由器缓存失效：

* 在服务器操作中：
  * 按需通过路径重新验证数据（ `revalidatePath` ）或通过缓存标签（ `revalidateTag` ）
  * 使用 `cookies.set` 或 `cookies.delete` 使路由器缓存无效，以防止使用 cookies 的路由过时（例如，身份验证）。
* 调用 `router.refresh` 将使路由器缓存失效，并对当前路由向服务器发出新的请求。

#### 选择退出

从 Next.js 15 开始，页面段默认是禁用的。

> **注意**：您还可以通过将 `<Link>` 组件的 `prefetch` 属性设置为 `false` 来选择不进行预取。

### 缓存交互

在配置不同的缓存机制时，了解它们之间的相互作用是很重要的：

#### 数据缓存和完整路由缓存 <a href="#data-cache-and-full-route-cache" id="data-cache-and-full-route-cache"></a>

* 重新验证或选择退出数据缓存将使完整路由缓存失效，因为渲染输出依赖于数据。
* 使完整路由缓存失效或选择退出不会影响数据缓存。您可以动态渲染同时具有缓存和未缓存数据的路由。当您的页面大部分使用缓存数据，但有一些组件依赖于需要在请求时获取的数据时，这非常有用。您可以动态渲染，而无需担心重新获取所有数据的性能影响。

#### 数据缓存和客户端路由缓存

* 要立即使数据缓存和路由器缓存失效，您可以在服务器操作中使用 `revalidatePath` 或 `revalidateTag` 。
* 在路由处理程序中重新验证数据缓存不会立即使路由器缓存失效，因为路由处理程序与特定路由没有绑定。这意味着路由器缓存将继续提供先前的有效负载，直到进行强制刷新或自动失效期限已过。

### APIs

下表概述了不同 Next.js API 如何影响缓存：

<table><thead><tr><th width="238.6171875">API</th><th width="123.25" align="center">路由缓存</th><th width="118.484375" align="center">完整路由缓存</th><th width="136.53515625" align="center">数据缓存</th><th align="center">React 缓存</th></tr></thead><tbody><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#link"><code>&#x3C;Link prefetch></code></a></td><td align="center">缓存</td><td align="center"></td><td align="center"></td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#routerprefetch"><code>router.prefetch</code></a></td><td align="center">缓存</td><td align="center"></td><td align="center"></td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#routerrefresh"><code>router.refresh</code></a></td><td align="center">重新验证</td><td align="center"></td><td align="center"></td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#fetch"><code>fetch</code></a></td><td align="center"></td><td align="center"></td><td align="center">缓存</td><td align="center">缓存</td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#fetch-optionscache"><code>fetch</code> <code>options.cache</code></a></td><td align="center"></td><td align="center"></td><td align="center">缓存或选择退出</td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnextrevalidate"><code>fetch</code></a><a href="https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnextrevalidate"><code>options.next.revalidate</code></a></td><td align="center"></td><td align="center">重新验证</td><td align="center">重新验证</td><td align="center"></td></tr><tr><td><p><a href="https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnexttags-and-revalidatetag"><code>fetch</code></a></p><p><a href="https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnexttags-and-revalidatetag"><code>options.next.tags</code></a></p></td><td align="center"></td><td align="center">缓存</td><td align="center">缓存</td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnexttags-and-revalidatetag"><code>revalidateTag</code></a></td><td align="center">重新验证（服务器操作）</td><td align="center">重新验证</td><td align="center">重新验证</td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#revalidatepath"><code>revalidatePath</code></a></td><td align="center">重新验证（服务器操作）</td><td align="center">重新验证</td><td align="center">重新验证</td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#segment-config-options"><code>const revalidate</code></a></td><td align="center"></td><td align="center">重新验证或选择退出</td><td align="center">重新验证或选择退出</td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#segment-config-options"><code>const dynamic</code></a></td><td align="center"></td><td align="center">缓存或选择退出</td><td align="center">缓存或选择退出</td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#cookies"><code>cookies</code></a></td><td align="center">重新验证（服务器操作）</td><td align="center">选择退出</td><td align="center"></td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#dynamic-apis"><code>headers</code>, <code>searchParams</code></a></td><td align="center"></td><td align="center">选择退出</td><td align="center"></td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#generatestaticparams"><code>generateStaticParams</code></a></td><td align="center"></td><td align="center">缓存</td><td align="center"></td><td align="center"></td></tr><tr><td><a href="https://nextjs.org/docs/app/building-your-application/caching#react-cache-function"><code>React.cache</code></a></td><td align="center"></td><td align="center"></td><td align="center"></td><td align="center">缓存</td></tr><tr><td><a href="https://nextjs.org/docs/app/api-reference/functions/unstable_cache"><code>unstable_cache</code></a></td><td align="center"></td><td align="center"></td><td align="center">缓存</td><td align="center"></td></tr></tbody></table>

#### `<Link>`

&#x20;默认情况下， `<Link>` 组件会自动从完整路由缓存中预取路由，并将 React 服务器组件负载添加到路由器缓存中。

要禁用预取，您可以将 `prefetch` 属性设置为 `false` 。但这不会永久跳过缓存，当用户访问该路由时，路由段仍会在客户端缓存。

了解更多关于 [`<Link>` 组件的信息](https://nextjs.org/docs/app/api-reference/components/link)。

#### `router.prefetch`

`useRouter` 钩子的 `prefetch` 选项可以用来手动预取一个路由。这将 React 服务器组件负载添加到路由缓存中。

查看 [`useRouter` 钩子](https://nextjs.org/docs/app/api-reference/functions/use-router) API 参考。

#### `router.refresh`

`useRouter` 钩子的 `refresh` 选项可用于手动刷新路由。这会完全清除路由器缓存，并对当前路由发出新的请求。 `refresh` 不会影响数据或完整路由缓存。

渲染结果将在客户端进行协调，同时保留 React 状态和浏览器状态。

查看 [useRouter 钩子](https://nextjs.org/docs/app/api-reference/functions/use-router) API 参考。

#### `fetch`

从 `fetch` 返回的数据不会自动缓存到数据缓存中。

`fetch` 的默认缓存行为（例如，当未指定 `cache` 选项时）等同于将 `cache` 选项设置为 `no-store` ：

```typescript
let data = await fetch('https://api.vercel.app/blog', { cache: 'no-store' })
```

有关更多选项，请参见 [`fetch` API 参考](https://nextjs.org/docs/app/api-reference/functions/fetch)。

#### `fetch options.cache`

您可以通过将 `cache` 选项设置为 `force-cache` 来选择单个 `fetch` 进行缓存：

```typescript
// Opt into caching
fetch(`https://...`, { cache: 'force-cache' })
```

请查看 [`fetch` API ](https://nextjs.org/docs/app/api-reference/functions/fetch)参考以获取更多选项。

#### `fetch options.next.revalidate`

您可以使用 `next.revalidate` 的 `fetch` 选项来设置单个 `fetch` 请求的重新验证周期（以秒为单位）。这将重新验证数据缓存，从而重新验证完整路由缓存。将提取新数据，并且组件将在服务器上重新渲染。

```typescript
// Revalidate at most after 1 hour
fetch(`https://...`, { next: { revalidate: 3600 } })
```

请参阅 `fetch` API 参考以获取更多选项。

#### `fetch options.next.tags` 和 `revalidateTag`

Next.js 具有用于细粒度数据缓存和重新验证的缓存标记系统。

1. 使用 `fetch` 或 [`unstable_cache`](https://nextjs.org/docs/app/api-reference/functions/unstable_cache) 时，您可以选择用一个或多个标签标记缓存条目。
2. 然后，您可以调用 `revalidateTag` 来清除与该标签相关的缓存条目。

例如，您可以在获取数据时设置一个标签：

```typescript
// Cache data with a tag
fetch(`https://...`, { next: { tags: ['a', 'b', 'c'] } })
```

然后，使用标签调用 `revalidateTag` 来清除缓存条目：

```typescript
// Revalidate entries with a specific tag
revalidateTag('a')
```

根据您想要实现的目标，您可以在两个地方使用 `revalidateTag` ：

1. [路由处理程序](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) - 在第三方事件（例如，webhook）响应中重新验证数据。这不会立即使路由器缓存失效，因为路由处理程序并未绑定到特定路由。
2. [服务器操作](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations) - 在用户操作（例如，表单提交）后重新验证数据。这将使关联路由的路由器缓存失效。

#### `revalidatePath`

`revalidatePath` 允许您手动重新验证数据并在单个操作中重新渲染特定路径下的路由段。调用 `revalidatePath` 方法会重新验证数据缓存，从而使完整路由缓存失效。

```typescript
revalidatePath('/')
```

您可以在两个地方使用 `revalidatePath` ，具体取决于您想要实现的目标：

1. [路由处理程序](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) - 以响应第三方事件（例如，Webhook）重新验证数据。
2. [服务器操作](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations) - 在用户交互后（例如，表单提交、点击按钮）重新验证数据。

请参阅 `revalidatePath` API 参考以获取更多信息。

> **`revalidatePath` 对比 `router.refresh` ：**
>
> 调用 `router.refresh` 将清除路由器缓存，并在服务器上重新渲染路由段，而不使数据缓存或完整路由缓存失效。
>
> 区别在于 `revalidatePath` 清空数据缓存和完整路由缓存，而 `router.refresh()` 不会改变数据缓存和完整路由缓存，因为它是一个客户端 API。

#### 动态 API

像 `cookies` 和 `headers` 这样的动态 API，以及 Pages 中的 `searchParams` 属性依赖于运行时的入站请求信息。使用它们将使路由从完整路由缓存中选出，换句话说，路由将被动态渲染。

**`cookies`**

在服务器操作中使用 `cookies.set` 或 `cookies.delete` 会使路由器缓存失效，以防使用 cookie 的路由变得过时（例如，反映身份验证更改）。

查看 [`cookies`](https://nextjs.org/docs/app/api-reference/functions/cookies) API 参考。

#### 段配置选项

路由段配置选项可用于覆盖路由段默认值，或在您无法使用 `fetch` API（例如数据库客户端或第三方库）时使用。

以下路由段配置选项将选择退出完整路由缓存：

* `const dynamic = 'force-dynamic'`

此配置选项将使所有提取操作选择退出数据缓存（即 `no-store` ）：

* `const fetchCache = 'default-no-store'`

请参阅 `fetchCache` 以查看更多高级选项。

请参阅路由段配置文档以获取更多选项。

#### `generateStaticParams`

对于动态段（例如 `app/blog/[slug]/page.js`），`generateStaticParams` 提供的路径在构建时被缓存到完整路由缓存中。在请求时，Next.js 还会缓存在构建时未知的路径，第一次访问时会进行缓存。

要在构建时静态渲染所有路径，请将完整的路径列表提供给 `generateStaticParams` ：

{% code title="app/blog/[slug]/page.js" %}
```javascript
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
 
  return posts.map((post) => ({
    slug: post.slug,
  }))
}
```
{% endcode %}

要在构建时静态渲染一部分路径，其余路径在运行时第一次被访问时渲染，请返回部分路径列表：

{% code title="app/blog/[slug]/page.js" %}
```javascript
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
 
  // Render the first 10 posts at build time
  return posts.slice(0, 10).map((post) => ({
    slug: post.slug,
  }))
}
```
{% endcode %}

要在首次访问时静态渲染所有路径，请返回一个空数组（在构建时不会渲染任何路径）或使用 [`export const dynamic = 'force-static'`](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config#dynamic) ：

{% code title="app/blog/[slug]/page.js" %}
```javascript
export async function generateStaticParams() {
  return []
}
```
{% endcode %}

> **注意**：您必须从 `generateStaticParams` 返回一个数组，即使它是空的。否则，路由将被动态渲染。

{% code title="app/changelog/[slug]/page.js" %}
```javascript
export const dynamic = 'force-static'
```
{% endcode %}

要在请求时禁用缓存，请在路由段中添加 `export const dynamicParams = false` 选项。当使用此配置选项时，仅提供 `generateStaticParams` 的路径将被服务，其他路由将返回 404 或匹配（在捕获所有路由的情况下）。

#### React cache 函数

React `cache` 函数允许你记忆函数的返回值，使你可以多次调用同一个函数而仅执行一次。

由于 `fetch` 请求会自动被记忆，因此你不需要将其包装在 React `cache`中。不过，你可以使用 `cache` 手动记忆数据请求，以应对 `fetch` API 不适用的用例。比如，一些数据库客户端、CMS 客户端或 GraphQL 客户端。

{% code title="utils/get-item.ts" %}
```typescript
import { cache } from 'react'
import db from '@/lib/db'
 
export const getItem = cache(async (id: string) => {
  const item = await db.item.findUnique({ id })
  return item
})
```
{% endcode %}

