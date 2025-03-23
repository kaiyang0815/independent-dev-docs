# 服务器操作和变更

服务器操作是异步函数，在服务器上执行。它们可以在服务器和客户端组件中调用，以处理 Next.js 应用程序中的表单提交和数据变更。

> 🎥 观看：了解更多关于使用服务器操作的变更 → YouTube（10 分钟）。

### 约定

可以使用 React `"use server"` 指令定义服务器操作。您可以将指令放在 `async` 函数的顶部，以将该函数标记为服务器操作，或放在单独文件的顶部，以将该文件的所有导出标记为服务器操作。

#### 服务器组件

服务器组件可以使用内联函数级别或模块级别 `"use server"` 指令。要内联一个服务器操作，请在函数体的顶部添加 `"use server"` ：

{% code title="app/page.tsx" %}
```tsx
export default function Page() {
  // Server Action
  async function create() {
    'use server'
    // Mutate data
  }
 
  return '...'
}
```
{% endcode %}

#### 客户端组件

要在客户端组件中调用服务器操作，请创建一个新文件并在其顶部添加 `"use server"` 指令。文件中的所有导出函数将被标记为可以在客户端和服务器组件中重用的服务器操作：

{% code title="app/actions.ts" %}
```typescript
'use server'
 
export async function create() {}
```
{% endcode %}

{% code title="app/button.tsx" %}
```tsx
'use client'
 
import { create } from './actions'
 
export function Button() {
  return <button onClick={() => create()}>Create</button>
}
```
{% endcode %}

#### 将动作作为 props 传递

您还可以将 Server Action 作为 prop 传递给 Client Component：

```tsx
<ClientComponent updateItemAction={updateItem} />
```

{% code title="app/client-component.tsx" %}
```tsx
'use client'
 
export default function ClientComponent({
  updateItemAction,
}: {
  updateItemAction: (formData: FormData) => void
}) {
  return <form action={updateItemAction}>{/* ... */}</form>
}
```
{% endcode %}

通常，Next.js TypeScript 插件会在 client-component.tsx 中标记 updateItemAction，因为它是一个通常无法跨客户端-服务器边界序列化的函数。然而，名为 `action` 或以 `Action` 结尾的 props 被假定为接收服务器操作。这只是一个启发式方法，因为 TypeScript 插件实际上并不知道它接收到的是服务器操作还是普通函数。运行时类型检查仍将确保您不会意外地将函数传递给客户端组件。

### 行为

* 服务器操作可以使用 `action` 属性在 [`<form>` 元素](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#forms)中调用：
  * 服务器组件默认支持渐进增强，这意味着即使 JavaScript 尚未加载或被禁用，表单也会被提交。
  * 在客户端组件中，如果 JavaScript 尚未加载，调用服务器操作的表单将排队提交，优先考虑客户端水合。
  * Hydration 后，浏览器在表单提交时不会刷新。
* 服务器操作不限于 `<form>` ，可以从事件处理程序、 `useEffect` 、第三方库和其他表单元素如 `<button>` 中调用。
* 服务器操作与 Next.js 的缓存和重新验证架构集成。当调用一个操作时，Next.js 可以在一次服务器往返中返回更新的 UI 和新数据。
* 在幕后，操作使用 `POST` 方法，只有这个 HTTP 方法可以调用它们。
* 服务器操作的参数和返回值必须可以被 React 序列化。有关[可序列化参数和值](https://react.dev/reference/react/use-server#serializable-parameters-and-return-values)的列表，请参见 React 文档。
* 服务器操作是函数。这意味着它们可以在您的应用程序中的任何地方重用。
* 服务器操作从它们所使用的页面或布局中继承运行时。
* 服务器操作从它们所使用的页面或布局中继承路由段配置，包括像 `maxDuration` 这样的字段。

### 示例

#### 表单

React 扩展了 HTML `<form>` 元素，以允许使用 `action` 属性调用服务器操作。

在表单中调用时，操作会自动接收 `FormData` 对象。您不需要使用 React `useState` 来管理字段，而是可以使用原生 `FormData` 方法提取数据：

{% code title="app/invoices/page.tsx" %}
```tsx
export default function Page() {
  async function createInvoice(formData: FormData) {
    'use server'
 
    const rawFormData = {
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    }
 
    // mutate data
    // revalidate cache
  }
 
  return <form action={createInvoice}>...</form>
}
```
{% endcode %}

> 注意：
>
> * 示例：[具有加载和错误状态的表单](https://github.com/vercel/next.js/tree/canary/examples/next-forms)
> * 当处理具有多个字段的表单时，您可能想考虑使用 `entries()` 方法与 JavaScript 的 `Object.fromEntries()` 。例如： `const rawFormData = Object.fromEntries(formData)` 。需要注意的一点是 `formData` 将包含额外的 `$ACTION_` 属性。
> * 查看 [React `<form>` 文档](https://react.dev/reference/react-dom/components/form#handle-form-submission-with-a-server-action)以了解更多信息。

#### 传递额外参数

您可以使用 JavaScript `bind` 方法向服务器操作传递额外参数。

{% code title="app/client-component.tsx" %}
```tsx
'use client'
 
import { updateUser } from './actions'
 
export function UserProfile({ userId }: { userId: string }) {
  const updateUserWithId = updateUser.bind(null, userId)
 
  return (
    <form action={updateUserWithId}>
      <input type="text" name="name" />
      <button type="submit">Update User Name</button>
    </form>
  )
}
```
{% endcode %}

服务器操作将接收 `userId` 参数，以及表单数据：

{% code title="app/actions.ts" %}
```typescript
'use server'
 
export async function updateUser(userId: string, formData: FormData) {}
```
{% endcode %}

> 注意：
>
> * 另一种选择是将参数作为隐藏输入字段传递到表单中（例如 `<input type="hidden" name="userId" value={userId} />` ）。但是，值将成为渲染的 HTML 的一部分，并且不会被编码。
> * `.bind` 在服务器和客户端组件中都有效。它还支持渐进增强。

#### 嵌套表单元素

您还可以在嵌套在 `<form>` 内的元素中调用服务器操作，例如 `<button>` 、 `<input type="submit">` 和 `<input type="image">` 。这些元素接受 `formAction` 属性或[事件处理程序](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#event-handlers)。

在您想要在表单中调用多个服务器操作的情况下，这非常有用。例如，您可以创建一个特定的 `<button>` 元素来保存帖子草稿，除了发布它。有关更多信息，请参阅 React `<form>` 文档。

#### 程序化表单提交

您可以使用 [`requestSubmit()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/requestSubmit) 方法以编程方式触发表单提交。例如，当用户使用 `⌘` + `Enter` 键盘快捷键提交表单时，您可以监听 `onKeyDown`事件：

{% code title="app/entry.tsx" %}
```tsx
'use client'
 
export function Entry() {
  const handleKeyDown = (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
    if (
      (e.ctrlKey || e.metaKey) &&
      (e.key === 'Enter' || e.key === 'NumpadEnter')
    ) {
      e.preventDefault()
      e.currentTarget.form?.requestSubmit()
    }
  }
 
  return (
    <div>
      <textarea name="entry" rows={20} required onKeyDown={handleKeyDown} />
    </div>
  )
}
```
{% endcode %}

这将触发最近的 `<form>` 祖先的提交，会调用服务器操作。

#### 服务器端表单验证

您可以使用 HTML 属性，如 `required` 和 `type="email"` 进行基本的客户端表单验证。

对于更高级的服务器端验证，您可以使用像 [zod](https://zod.dev/) 这样的库在修改数据之前验证表单字段：

{% code title="app/actions.ts" %}
```typescript
'use server'
 
import { z } from 'zod'
 
const schema = z.object({
  email: z.string({
    invalid_type_error: 'Invalid Email',
  }),
})
 
export default async function createUser(formData: FormData) {
  const validatedFields = schema.safeParse({
    email: formData.get('email'),
  })
 
  // 如果表单数据无效，则提前返回
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    }
  }
 
  // Mutate data
}
```
{% endcode %}

一旦字段在服务器上经过验证，您可以在操作中返回一个可序列化的对象，并使用 React `useActionState` 钩子向用户显示消息。&#x20;

* 通过将操作传递给 `useActionState` ，操作的函数签名更改为接收一个新的 `prevState` 或 `initialState` 参数作为其第一个参数。
* `useActionState` 是一个 React hook，因此必须在客户端组件中使用。

{% code title="app/actions.ts" %}
```typescript
'use server'
 
import { redirect } from 'next/navigation'
 
export async function createUser(prevState: any, formData: FormData) {
  const res = await fetch('https://...')
  const json = await res.json()
 
  if (!res.ok) {
    return { message: 'Please enter a valid email' }
  }
 
  redirect('/dashboard')
}
```
{% endcode %}

然后，您可以将操作传递给 `useActionState` hook，并使用返回的 `state` 来显示错误消息。

{% code title="app/ui/signup.tsx" %}
```tsx
'use client'
 
import { useActionState } from 'react'
import { createUser } from '@/app/actions'
 
const initialState = {
  message: '',
}
 
export function Signup() {
  const [state, formAction, pending] = useActionState(createUser, initialState)
 
  return (
    <form action={formAction}>
      <label htmlFor="email">Email</label>
      <input type="text" id="email" name="email" required />
      {/* ... */}
      <p aria-live="polite">{state?.message}</p>
      <button disabled={pending}>Sign up</button>
    </form>
  )
}
```
{% endcode %}

#### 待处理状态

该 `useActionState` 钩子暴露了一个 `pending` 布尔值，可以在执行操作时用于显示加载指示器。

使用 `useFormStatus` 钩子在执行操作时显示加载指示器。使用此钩子时，您需要创建一个单独的组件来渲染加载指示器。例如，当操作处于待处理状态时禁用按钮：

{% code title="app/ui/button.tsx" %}
```tsx
'use client'
 
import { useFormStatus } from 'react-dom'
 
export function SubmitButton() {
  const { pending } = useFormStatus()
 
  return (
    <button disabled={pending} type="submit">
      Sign Up
    </button>
  )
}
```
{% endcode %}

您可以将 `SubmitButton` 组件嵌套在表单内：

{% code title="app/ui/signup.tsx" %}
```tsx
import { SubmitButton } from './button'
import { createUser } from '@/app/actions'
 
export function Signup() {
  return (
    <form action={createUser}>
      {/* 其他表单元素 */}
      <SubmitButton />
    </form>
  )
}
```
{% endcode %}

> 注意：在 React 19 中， `useFormStatus` 在返回的对象上包含额外的键，如 data、method 和 action。如果您没有使用 React 19，则仅可用 `pending` 键。

#### 乐观更新

您可以使用 React `useOptimistic` 钩子在服务器操作完成执行之前乐观地更新 UI，而不是等待响应：

{% code title="app/page.tsx" %}
```tsx
'use client'
 
import { useOptimistic } from 'react'
import { send } from './actions'
 
type Message = {
  message: string
}
 
export function Thread({ messages }: { messages: Message[] }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic<
    Message[],
    string
  >(messages, (state, newMessage) => [...state, { message: newMessage }])
 
  const formAction = async (formData: FormData) => {
    const message = formData.get('message') as string
    addOptimisticMessage(message)
    await send(message)
  }
 
  return (
    <div>
      {optimisticMessages.map((m, i) => (
        <div key={i}>{m.message}</div>
      ))}
      <form action={formAction}>
        <input type="text" name="message" />
        <button type="submit">Send</button>
      </form>
    </div>
  )
}
```
{% endcode %}

#### 事件处理程序

虽然在 `<form>` 元素中使用服务器操作很常见，但它们也可以通过事件处理程序如 `onClick` 来调用。例如，增加点赞计数：

{% code title="app/like-button.tsx" %}
```tsx
'use client'
 
import { incrementLike } from './actions'
import { useState } from 'react'
 
export default function LikeButton({ initialLikes }: { initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes)
 
  return (
    <>
      <p>Total Likes: {likes}</p>
      <button
        onClick={async () => {
          const updatedLikes = await incrementLike()
          setLikes(updatedLikes)
        }}
      >
        Like
      </button>
    </>
  )
}
```
{% endcode %}

您还可以为表单元素添加事件处理程序，例如，保存表单字段 `onChange` ：

{% code title="app/ui/edit-post.tsx" %}
```tsx
'use client'
 
import { publishPost, saveDraft } from './actions'
 
export default function EditPost() {
  return (
    <form action={publishPost}>
      <textarea
        name="content"
        onChange={async (e) => {
          await saveDraft(e.target.value)
        }}
      />
      <button type="submit">Publish</button>
    </form>
  )
}
```
{% endcode %}

对于这种情况，多个事件可能会快速触发，我们建议使用**防抖**以防止不必要的服务器操作调用。

#### useEffect

您可以使用 React [`useEffect`](https://react.dev/reference/react/useEffect) 钩子在组件挂载或依赖项更改时调用服务器操作。这对于依赖于全局事件或需要自动触发的变更非常有用。例如， `onKeyDown` 用于应用快捷方式，交叉观察者钩子用于无限滚动，或在组件挂载时更新视图计数：

{% code title="app/view-count.tsx" %}
```tsx
'use client'
 
import { incrementViews } from './actions'
import { useState, useEffect } from 'react'
 
export default function ViewCount({ initialViews }: { initialViews: number }) {
  const [views, setViews] = useState(initialViews)
 
  useEffect(() => {
    const updateViews = async () => {
      const updatedViews = await incrementViews()
      setViews(updatedViews)
    }
 
    updateViews()
  }, [])
 
  return <p>Total Views: {views}</p>
}
```
{% endcode %}

记得考虑 `useEffect` 的[行为和注意事项](https://react.dev/reference/react/useEffect#caveats)。

#### 错误处理

当抛出错误时，它将被客户端最近的 `error.js` 或 `<Suspense>` 边界捕获。有关更多信息，请参见错误处理。

> 注意：
>
> * 除了抛出错误，您还可以返回一个对象，由 `useActionState` 处理。请参见服务器端验证和错误处理。

#### 重新验证数据

您可以在您的服务器操作中使用 `revalidatePath` API 重新验证 Next.js 缓存：

{% code title="app/actions.ts" %}
```typescript
'use server'
 
import { revalidatePath } from 'next/cache'
 
export async function createPost() {
  try {
    // ...
  } catch (error) {
    // ...
  }
 
  revalidatePath('/posts')
}
```
{% endcode %}

或者使用 [`revalidateTag`](https://nextjs.org/docs/app/api-reference/functions/revalidateTag) 通过缓存标签使特定数据获取失效：

{% code title="app/actions.ts" %}
```typescript
'use server'
 
import { revalidateTag } from 'next/cache'
 
export async function createPost() {
  try {
    // ...
  } catch (error) {
    // ...
  }
 
  revalidateTag('posts')
}
```
{% endcode %}

#### 重定向

如果您希望在服务器操作完成后将用户重定向到不同的路由，可以使用 `redirect` API。 `redirect` 需要在 `try/catch` 块外调用：

{% code title="app/actions.ts" %}
```typescript
'use server'
 
import { redirect } from 'next/navigation'
import { revalidateTag } from 'next/cache'
 
export async function createPost(id: string) {
  try {
    // ...
  } catch (error) {
    // ...
  }
 
  revalidateTag('posts') // Update cached posts
  redirect(`/post/${id}`) // Navigate to the new post page
}
```
{% endcode %}

#### Cookies

您可以在服务器操作中使用 `cookies` API 来 `get` 、 `set` 和 `delete`cookies：

{% code title="app/actions.ts" %}
```typescript
'use server'
 
import { cookies } from 'next/headers'
 
export async function exampleAction() {
  const cookieStore = await cookies()
 
  // Get cookie
  cookieStore.get('name')?.value
 
  // Set cookie
  cookieStore.set('name', 'Delba')
 
  // Delete cookie
  cookieStore.delete('name')
}app/actions.ts
```
{% endcode %}

查看有关从服务器操作中删除 cookies 的[其他示例](https://nextjs.org/docs/app/api-reference/functions/cookies#deleting-cookies)。

### 安全性

默认情况下，当创建并导出服务器操作时，它会创建一个公共 HTTP 端点，并应遵循相同的安全假设和授权检查。这意味着，即使服务器操作或工具函数在代码中没有被其他地方导入，它仍然是公开可访问的。

为提高安全性，Next.js 具有以下内置功能：

* 安全操作 ID：Next.js 创建加密的、非确定性 ID，以允许客户端引用和调用服务器操作。这些 ID 会定期在构建之间重新计算，以增强安全性。
* 死代码消除[^1]：未使用的服务器操作（通过其 ID 引用）从客户端包中移除，以避免第三方的公共访问。

> 注意：这些 ID 在编译时创建，并最多缓存 14 天。当启动新构建或构建缓存失效时，将重新生成它们。此安全改进减少了在缺少身份验证层的情况下的风险。然而，您仍然应该将服务器操作视为公共 HTTP 端点。

{% code title="app/actions.js" %}
```typescript
'use server'
 
// 此操作在我们的应用程序中使用，因此 Next.js
// 将创建一个安全的 ID 以允许客户端引用
// 并调用服务器操作。
export async function updateUserAction(formData) {}
 
// 此操作在我们的应用中未使用，因此 Next.js
// 会在 `next build` 期间自动删除此代码
// 不会创建公共端点。
export async function deleteUserAction(formData) {}
```
{% endcode %}

#### 身份验证和授权

您应该确保用户被授权执行该操作。例如：

{% code title="app/actions.ts" %}
```typescript
'use server'
 
import { auth } from './lib'
 
export function addItem() {
  const { user } = auth()
  if (!user) {
    throw new Error('You must be signed in to perform this action')
  }
 
  // ...
}
```
{% endcode %}

#### 闭包和加密

在组件内部定义服务器操作会创建一个闭包，其中操作可以访问外部函数的作用域。例如， `publish` 操作可以访问 `publishVersion` 变量：

{% code title="app/page.tsx" %}
```tsx
export default async function Page() {
  const publishVersion = await getLatestVersion();
 
  async function publish() {
    "use server";
    if (publishVersion !== await getLatestVersion()) {
      throw new Error('The version has changed since pressing publish');
    }
    ...
  }
 
  return (
    <form>
      <button formAction={publish}>Publish</button>
    </form>
  );
}
```
{% endcode %}

闭包在需要捕捉渲染时数据快照（例如 `publishVersion` ）时非常有用，以便在调用操作时可以稍后使用。

然而，为了实现这一点，捕获的变量在调用操作时会发送到客户端并返回到服务器。为了防止敏感数据暴露给客户端，Next.js 会自动加密闭合变量。每次构建 Next.js 应用程序时，都会为每个操作生成一个新的私钥。这意味着操作只能针对特定构建被调用。

> 注意：我们不建议仅依赖加密来防止敏感值在客户端被暴露。相反，您应该使用 React 污染 API 主动防止特定数据被发送到客户端。

#### 覆盖加密密钥（高级）

当在多个服务器上自托管您的 Next.js 应用程序时，每个服务器实例可能会拥有不同的加密密钥，从而导致潜在的不一致性。

为了缓解这个问题，您可以使用 `process.env.NEXT_SERVER_ACTIONS_ENCRYPTION_KEY` 环境变量覆盖加密密钥。指定此变量可确保您的加密密钥在构建之间保持持久，并且所有服务器实例使用相同的密钥。此变量必须是 AES-GCM 加密的。

这是一个高级用例，其中多个部署之间一致的加密行为对您的应用程序至关重要。您应该考虑标准安全实践，例如密钥轮换和签名。

> 注意：部署到 Vercel 的 Next.js 应用程序会自动处理此问题。

#### 允许的来源（高级）

由于服务器操作可以在 `<form>` 元素中被调用，这使它们容易受到 CSRF 攻击。&#x20;

在后台，服务器操作使用 `POST` 方法，并且只有这个 HTTP 方法被允许调用它们。这防止了现代浏览器中大多数 CSRF 漏洞，特别是 [SameSite cookies](https://web.dev/articles/samesite-cookies-explained) 是默认设置。

作为额外的保护，Next.js 中的服务器操作还会将 [Origin 头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin)与 [Host 头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Host)（或 `X-Forwarded-Host` ）进行比较。如果这些不匹配，请求将被中止。换句话说，服务器操作只能在与托管它的页面相同的主机上被调用。

对于使用反向代理或多层后端架构的大型应用（服务器 API 与生产域不同），建议使用配置选项 `serverActions.allowedOrigins` 来指定安全来源列表。该选项接受一个字符串数组。

{% code title="next.config.js" %}
```javascript
/** @type {import('next').NextConfig} */
module.exports = {
  experimental: {
    serverActions: {
      allowedOrigins: ['my-proxy.com', '*.my-proxy.com'],
    },
  },
}
```
{% endcode %}

了解更多关于[安全性和服务器操作](https://nextjs.org/blog/security-nextjs-server-components-actions)的信息。

### 其他资源

有关更多信息，请查看以下 React 文档：

* [Server Actions  服务器操作](https://react.dev/reference/rsc/server-actions)
* [`"use server"`](https://react.dev/reference/react/use-server)
* [`<form>`](https://react.dev/reference/react-dom/components/form)
* [`useFormStatus`](https://react.dev/reference/react-dom/hooks/useFormStatus)
* [`useActionState`](https://react.dev/reference/react/useActionState)
* [`useOptimistic`](https://react.dev/reference/react/useOptimistic)

### 下一步

学习如何在 Next.js 中配置服务器操作



[^1]: 编译器或打包工具（如 Webpack、Rollup、ESBuild）在构建时，**自动删除永远不会被执行的代码**，优化最终产物的体积和性能。
