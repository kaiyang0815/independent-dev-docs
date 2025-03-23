# Sass

Next.js 内置支持在安装包后与 Sass 集成，使用 `.scss` 和 `.sass` 扩展。您可以通过 CSS Modules 和 `.module.scss` 或 `.module.sass` 扩展使用组件级 Sass。

首先，安装 `sass` ：

```bash
npm install --save-dev sass
```

> **注意**：
>
> Sass 支持两种不同的语法，每种语法都有自己的扩展。 `.scss` 扩展要求您使用 SCSS 语法，而 `.sass` 扩展要求您使用缩进语法（“Sass”）。
>
> 如果你不确定选择哪个，可以从 `.scss` 扩展开始，它是 CSS 的超集，并且不需要你学习缩进语法（"Sass"）。

### 自定义 Sass 选项

如果你想要配置 Sass 选项，请在 `next.config` 中使用 `sassOptions` 。

{% code title="next.config.ts" %}
```typescript
import type { NextConfig } from 'next'
 
const nextConfig: NextConfig = {
  sassOptions: {
    additionalData: `$var: red;`,
  },
}
 
export default nextConfig
```
{% endcode %}

#### 实现

您可以使用 `implementation` 属性来指定要使用的 Sass 实现。默认情况下，Next.js 使用 `sass` 包。

{% code title="next.config.ts" %}
```typescript
import type { NextConfig } from 'next'
 
const nextConfig: NextConfig = {
  sassOptions: {
    implementation: 'sass-embedded',
  },
}
 
export default nextConfig
```
{% endcode %}

### Sass 变量

Next.js 支持从 CSS 模块文件导出的 Sass 变量。

例如，使用导出的 `primaryColor` Sass 变量：

{% code title="app/variables.module.scss" %}
```scss
$primary-color: #64ff00;
 
:export {
  primaryColor: $primary-color;
}
```
{% endcode %}

{% code title="app/page.js" %}
```javascript
// maps to root `/` URL
 
import variables from './variables.module.scss'
 
export default function Page() {
  return <h1 style={{ color: variables.primaryColor }}>Hello, Next.js!</h1>
}
```
{% endcode %}

