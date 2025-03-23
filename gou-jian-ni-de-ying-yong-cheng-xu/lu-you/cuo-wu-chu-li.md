# 错误处理

错误可以分为两类：期望的错误和未捕获的异常：

* **将期望的错误建模为返回值**：在服务器操作中避免使用 `try` / `catch` 来表示期望的错误。使用 `useActionState` 来管理这些错误并将其返回给客户端。
* **使用错误边界处理意外错误**：使用 `error.tsx` 和 `global-error.tsx` 文件实现错误边界，以处理意外错误并提供后备用户界面。

### 处理期望的错误

预期错误是指在应用程序正常操作过程中可能发生的错误，例如来自服务器端表单验证或请求失败的错误。这些错误应明确处理并返回给客户端。

#### 处理来自服务器操作的预期错误

使用 `useActionState` 钩子来管理服务器操作的状态，包括处理错误。这种方法避免了对预期错误使用 `try` / `catch` 块，这些错误应建模为返回值而不是抛出异常。

```tsx
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

然后，您可以将您的操作传递给 `useActionState` 钩子，并使用返回的 `state` 显示错误消息。

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

您还可以使用返回的状态从客户端组件显示 toast 消息。

#### 处理来自服务器组件的预期错误

在服务器组件内部获取数据时，可以使用响应条件性地呈现错误消息或 `redirect` 。

```tsx
export default async function Page() {
  const res = await fetch(`https://...`)
  const data = await res.json()
 
  if (!res.ok) {
    return 'There was an error.'
  }
 
  return '...'
}
```

### 未捕获的异常

未捕获的异常是意外错误，表示在应用程序的正常流程中不应发生的 bug 或问题。这些应该通过抛出错误来处理，随后将由错误边界捕获。

* 常见问题：使用 `error.js` 处理根布局下的未捕获错误。
* 可选：使用嵌套的 `error.js` 文件（例如 `app/dashboard/error.js` ）处理细粒度的未捕获错误
* 不常见：在根布局中使用 `global-error.js` 处理未捕获的错误。

#### 使用错误边界

Next.js 使用错误边界来处理未捕获的异常。错误边界捕获其子组件中的错误，并显示回退 UI，而不是崩溃的组件树。

通过在路由段中添加一个 `error.tsx` 文件并导出一个 React 组件来创建错误边界：

```tsx
'use client' // Error boundaries must be Client Components
 
import { useEffect } from 'react'
 
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error(error)
  }, [error])
 
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </button>
    </div>
  )
}
```

如果您希望错误向上冒泡到父错误边界，您可以在渲染 `error` 组件时 `throw` 。

#### 处理嵌套路由中的错误

错误将冒泡到最近的父错误边界。这允许通过在路由层次结构的不同级别放置 `error.tsx`文件来进行细粒度的错误处理。

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### 处理全局错误

虽然不太常见，但您可以使用位于根应用目录中的 `app/global-error.js` 在根布局中处理错误，即使在利用国际化时也是如此。全局错误 UI 必须定义自己的 `<html>` 和 `<body>`标签，因为它在激活时会替换根布局或模板。

```tsx
'use client' // Error boundaries must be Client Components
 
export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    // global-error must include html and body tags
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  )
}
```

### 下一步

