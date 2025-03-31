# 分析

Next.js 内置支持测量和报告性能指标。您可以使用 `useReportWebVitals` 钩子自行管理报告，或者，Vercel 提供了一项托管服务，可以自动收集和可视化指标。

### 自定义构建

{% code title="app/_components/web-vitals.js" %}
```jsx
'use client'
 
import { useReportWebVitals } from 'next/web-vitals'
 
export function WebVitals() {
  useReportWebVitals((metric) => {
    console.log(metric)
  })
}
```
{% endcode %}

{% code title="" %}
```jsx
import { WebVitals } from './_components/web-vitals'
 
export default function Layout({ children }) {
  return (
    <html>
      <body>
        <WebVitals />
        {children}
      </body>
    </html>
  )
}
```
{% endcode %}

> 由于 `useReportWebVitals` 钩子需要 `"use client"` 指令，因此最有效的方法是创建一个单独的组件，根布局导入该组件。这将客户端边界专门限制在 `WebVitals` 组件内。

查看 API 参考以获取更多信息。

### 网页性能重要指标

网页重要指标是一组有用的指标，旨在捕捉网页的用户体验。以下网页重要指标均已包含：

* 首次字节时间 (TTFB)
* 首次内容绘制 (FCP)
* 最大内容绘制 (LCP)
* 首次输入延迟 (FID)
* 累积布局偏移 (CLS)
* 交互到下一个绘制 (INP)

您可以使用 `name` 属性处理这些指标的所有结果。

{% code title="app/_components/web-vitals.tsx" %}
```tsx
'use client'
 
import { useReportWebVitals } from 'next/web-vitals'
 
export function WebVitals() {
  useReportWebVitals((metric) => {
    switch (metric.name) {
      case 'FCP': {
        // handle FCP results
      }
      case 'LCP': {
        // handle LCP results
      }
      // ...
    }
  })
}
```
{% endcode %}

### 将结果发送到外部系统

您可以将结果发送到任何端点，以测量和跟踪您网站上的真实用户性能。例如：

```typescript
useReportWebVitals((metric) => {
  const body = JSON.stringify(metric)
  const url = 'https://example.com/analytics'
 
  // Use `navigator.sendBeacon()` if available, falling back to `fetch()`.
  if (navigator.sendBeacon) {
    navigator.sendBeacon(url, body)
  } else {
    fetch(url, { body, method: 'POST', keepalive: true })
  }
})
```

> 如果您使用 Google Analytics，使用 `id` 值可以让您手动构建指标分布（以计算百分位数等）。

