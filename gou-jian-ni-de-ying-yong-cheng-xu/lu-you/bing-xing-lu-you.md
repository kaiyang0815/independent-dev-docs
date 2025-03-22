# 并行路由

并行路由允许您在同一布局中同时或有条件地渲染一个或多个页面。它们对于应用程序的高度动态部分非常有用，例如仪表板和社交网站上的信息流。

例如，考虑一个仪表板，您可以使用并行路由同时渲染 `team` 和 `analytics` 页面：\


<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### 插槽

并行路由是使用命名插槽创建的。插槽使用 `@folder` 约定定义。例如，以下文件结构定义了两个插槽： `@analytics` 和 `@team` ：

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

插槽作为属性传递给共享的父布局。对于上面的示例， `app/layout.js`中的组件现在接受 `@analytics` 和 `@team` 插槽属性，并可以与 `children` 属性平行渲染它们：

{% code title="app/layout.tsx" %}
```tsx
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <>
      {children}
      {team}
      {analytics}
    </>
  )
}
```
{% endcode %}

然而，插槽不是路由段，并不影响 URL 结构。例如，对于 `/@analytics/views` ，URL 将是 `/views` ，因为 `@analytics` 是一个插槽。插槽与常规页面组件结合，以形成与路由段关联的最终页面。因此，您不能在同一路由段级别上同时拥有独立的静态和动态插槽。如果一个插槽是动态的，那么该级别上的所有插槽必须都是动态的。

> 注意：`children` 属性是一个隐式插槽，无需映射到文件夹。这意味着 `app/page.js` 等同于 `app/@children/page.js` 。

### 活动状态和导航

默认情况下，Next.js 会跟踪每个插槽的活动状态（或子页面）。然而，插槽中渲染的内容将取决于导航的类型：

* 软导航：在客户端导航期间，Next.js 将执行部分渲染，改变插槽中的子页面，同时保持其他插槽的活动子页面，即使它们与当前 URL 不匹配。
* 硬导航：在完整页面加载（浏览器刷新）后，Next.js 无法确定与当前 URL 不匹配的插槽的活动状态。相反，它将为不匹配的插槽渲染一个 `default.js` 文件，或者如果 `default.js` 不存在，则渲染 `404` 。

> 注意：不匹配路由的 `404` 有助于确保您不会意外地在不适合的页面上渲染平行路由。

#### `default.js`

您可以定义一个 `default.js` 文件，以在初始加载或全页面重新加载期间作为未匹配插槽的后备渲染。

请考虑以下文件夹结构。 `@team` 插槽有一个 `/settings` 页面，但 `@analytics` 没有。

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

当导航到 `/settings` 时， `@team` 插槽将渲染 `/settings` 页面，同时保持 `@analytics` 插槽的当前活动页面。

刷新时，Next.js 将为 `@analytics` 渲染一个 `default.js` 。如果 `default.js` 不存在，则渲染一个 `404` 作为替代。

此外，由于 `children` 是一个隐式插槽，您还需要创建一个 `default.js`文件，以便在 Next.js 无法恢复父页面的活动状态时为 `children` 呈现回退。

#### [`useSelectedLayoutSegment(s)`](https://nextjs.org/docs/app/building-your-application/routing/parallel-routes#useselectedlayoutsegments) <a href="#useselectedlayoutsegments" id="useselectedlayoutsegments"></a>

`useSelectedLayoutSegment` 和 `useSelectedLayoutSegments` 都接受一个 `parallelRoutesKey` 参数，这允许您在插槽内读取活动路由段。

{% code title="app/layout.tsx" %}
```tsx
'use client'
 
import { useSelectedLayoutSegment } from 'next/navigation'
 
export default function Layout({ auth }: { auth: React.ReactNode }) {
  const loginSegment = useSelectedLayoutSegment('auth')
  // ...
}
```
{% endcode %}

当用户导航到 `app/@auth/login` （或在 URL 栏中出现 `/login` ）， `loginSegment` 将等于字符串 `"login"` 。

### 示例

#### 条件路由

您可以使用并行路由根据某些条件（例如用户角色）有条件地渲染路由。例如，为 `/admin` 或 `/user` 角色渲染不同的仪表板页面：

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

{% code title="app/dashboard/layout.tsx" %}
```tsx
import { checkUserRole } from '@/lib/auth'
 
export default function Layout({
  user,
  admin,
}: {
  user: React.ReactNode
  admin: React.ReactNode
}) {
  const role = checkUserRole()
  return role === 'admin' ? admin : user
}
```
{% endcode %}

#### 标签组

您可以在插槽中添加一个 `layout` 以允许用户独立导航该插槽。这对于创建标签非常有用。

例如， `@analytics` 插槽有两个子页面： `/page-views` 和 `/visitors`。

在 `@analytics` 中，创建一个 `layout` 文件以在两个页面之间共享标签：

{% code title="app/@analytics/layout.tsx" %}
```tsx
import Link from 'next/link'
 
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Link href="/page-views">Page Views</Link>
        <Link href="/visitors">Visitors</Link>
      </nav>
      <div>{children}</div>
    </>
  )
}
```
{% endcode %}

#### 模态

并行路由可以与拦截路由一起使用，以创建支持深度链接的模态。这使您能够解决构建模态时常见的挑战，例如：

* 通过 URL 共享模态内容。
* 在页面刷新时保留上下文，而不是关闭模态框。
* 在向后导航时关闭模态框，而不是返回到上一个路由。
* 在前进导航时重新打开模态框。

考虑以下 UI 模式，用户可以通过客户端导航从布局中打开登录模态框，或访问单独的 `/login` 页面：

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

要实现此模式，首先创建一个 `/login` 路由，以渲染您的主登录页面。

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

{% code title="app/login/page.tsx" %}
```tsx
import { Login } from '@/app/ui/login'
 
export default function Page() {
  return <Login />
}
```
{% endcode %}

然后，在 `@auth` 插槽内，添加 `default.js` 文件，返回 `null` 。这确保了当模态不活跃时不会被渲染。

{% code title="app/@auth/default.tsx" %}
```tsx
export default function Default() {
  return null
}
```
{% endcode %}

在您的 `@auth` 插槽内，通过更新 `/(.)login` 文件夹来拦截 `/login` 路由。将 `<Modal>` 组件及其子组件导入到 `/(.)login/page.tsx` 文件中：

{% code title="app/@auth/(.)login/page.tsx" %}
```tsx
import { Modal } from '@/app/ui/modal'
import { Login } from '@/app/ui/login'
 
export default function Page() {
  return (
    <Modal>
      <Login />
    </Modal>
  )
}
```
{% endcode %}

> 注意：
>
> * 拦截路由所使用的约定，例如 `(.)` ，取决于您的文件系统结构。请参见拦截路由约定。
> * 通过将 `<Modal>` 功能与模态内容 ( `<Login>` ) 分开，您可以确保模态内部的任何内容，例如表单，都是服务器组件。有关更多信息，请参见交错客户端和服务器组件。

**打开模态框**

现在，您可以利用 Next.js 路由器来打开和关闭模态框。这确保了在模态框打开时，URL 会正确更新，并在前后导航时保持一致。

要打开模态框，将 `@auth` 插槽作为 prop 传递给父布局，并与 `children` prop 一起渲染。

{% code title="app/layout.tsx" %}
```tsx
import Link from 'next/link'
 
export default function Layout({
  auth,
  children,
}: {
  auth: React.ReactNode
  children: React.ReactNode
}) {
  return (
    <>
      <nav>
        <Link href="/login">Open modal</Link>
      </nav>
      <div>{auth}</div>
      <div>{children}</div>
    </>
  )
}
```
{% endcode %}

当用户点击 `<Link>` 时，模态框将打开，而不是导航到 `/login` 页面。然而，在刷新或初始加载时，导航到 `/login` 将把用户带到主登录页面。

**关闭模态框**

您可以通过调用 `router.back()` 或使用 `Link` 组件来关闭模态框。

{% code title="app/ui/modal.tsx" %}
```tsx
'use client'
 
import { useRouter } from 'next/navigation'
 
export function Modal({ children }: { children: React.ReactNode }) {
  const router = useRouter()
 
  return (
    <>
      <button
        onClick={() => {
          router.back()
        }}
      >
        Close modal
      </button>
      <div>{children}</div>
    </>
  )
}
```
{% endcode %}

当使用 `Link` 组件导航离开一个不应该再渲染 `@auth` 插槽的页面时，我们需要确保并行路由匹配到一个返回 `null` 的组件。例如，当导航回根页面时，我们创建一个 `@auth/page.tsx` 组件：

{% code title="app/ui/modal.tsx" %}
```tsx
import Link from 'next/link'
 
export function Modal({ children }: { children: React.ReactNode }) {
  return (
    <>
      <Link href="/">Close modal</Link>
      <div>{children}</div>
    </>
  )
}
```
{% endcode %}

{% code title="app/@auth/page.tsx" %}
```tsx
export default function Page() {
  return null
}
```
{% endcode %}

或者如果导航到任何其他页面（例如 `/foo` ， `/foo/bar` 等），您可以使用一个通用插槽：

{% code title="app/@auth/[...catchAll]/page.tsx" %}
```tsx
export default function CatchAll() {
  return null
}
```
{% endcode %}

> 注意：
>
> * 我们在 `@auth` 插槽中使用一个通用路由来关闭模态框，这是由于在活动状态和导航中描述的行为。由于客户端导航到不再匹配插槽的路由将保持可见，因此我们需要将插槽匹配到一个返回 `null` 的路由以关闭模态框。
> * 其他示例可能包括在画廊中打开照片模态，同时拥有专门的 `/photo/[id]`页面，或在侧模态中打开购物车。
> * 查看带有拦截和并行路由的模态示例。

#### 加载和错误界面

并行路由可以独立流式传输，允许您为每个路由定义独立的错误和加载状态：

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

有关更多信息，请参阅加载 UI 和错误处理文档。

### 下一步

