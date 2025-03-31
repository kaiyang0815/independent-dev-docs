# 仪表盘

仪表盘是使用代码将监控和日志工具集成到您的应用程序中的过程。这使您能够跟踪应用程序的性能和行为，并在生产中调试问题。

### [约定](https://nextjs.org/docs/app/building-your-application/optimizing/instrumentation#convention) <a href="#convention" id="convention"></a>

要设置仪表盘，请在项目的根目录中创建 `instrumentation.ts|js` 文件（如果使用一个，请在 `src` 文件夹内创建）。

然后，在文件中导出 `register` 函数。此函数将在新的 Next.js 服务器实例启动时调用一次。

例如，要使用 Next.js 和 OpenTelemetry 以及 @vercel/otel：

{% code title="instrumentation.ts" %}
```typescript
import { registerOTel } from '@vercel/otel' 
export function register() {  
    registerOTel('next-app')
}
```
{% endcode %}

查看 Next.js 与 OpenTelemetry 示例以获取完整实现。

> 注意：
>
> * `instrumentation` 文件应位于项目的根目录中，而不是在 `app` 或 `pages` 目录内。如果您使用 `src` 文件夹，则应将文件放在 `src` 中，与 `pages` 和 `app` 一起。
> * 如果您使用 `pageExtensions` 配置选项添加后缀，则还需要更新 `instrumentation` 文件名以匹配。

### [示例](https://nextjs.org/docs/app/building-your-application/optimizing/instrumentation#examples) <a href="#examples" id="examples"></a>

#### [导入具有副作用的文件](https://nextjs.org/docs/app/building-your-application/optimizing/instrumentation#importing-files-with-side-effects) <a href="#importing-files-with-side-effects" id="importing-files-with-side-effects"></a>

有时，在代码中导入一个文件可能是有用的，因为它会引起副作用。例如，您可能会导入一个定义了一组全局变量的文件，但在代码中从未明确使用导入的文件。您仍然可以访问该包声明的全局变量。

我们建议在您的 `register` 函数中使用 JavaScript `import` 语法导入文件。以下示例演示了在 `register` 函数中 `import` 的基本用法：

{% code title="instrumentation.ts" %}
```typescript
export async function register() {  
    await import('package-with-side-effect')
}
```
{% endcode %}

> **注意：**
>
> 我们建议在 `register` 函数内导入文件，而不是在文件顶部。通过这样做，您可以将所有副作用集成到代码的一个地方，并避免因在文件顶部全局导入而导致的任何意外后果。

#### [导入特定运行时的代码](https://nextjs.org/docs/app/building-your-application/optimizing/instrumentation#importing-runtime-specific-code) <a href="#importing-runtime-specific-code" id="importing-runtime-specific-code"></a>

Next.js 在所有环境中调用 `register` ，因此条件性地导入任何不支持特定运行时的代码（例如 Edge 或 Node.js）非常重要。您可以使用 `NEXT_RUNTIME` 环境变量来获取当前环境：

{% code title="instrumentation.ts" %}
```typescript
export async function register() {  
    if (process.env.NEXT_RUNTIME === 'nodejs') {    
        await import('./instrumentation-node')  
    }   
    if (process.env.NEXT_RUNTIME === 'edge') {    
        await import('./instrumentation-edge')  
    }
}
```
{% endcode %}

### 了解有关仪器的更多信息 <a href="#learn-more-about-instrumentation" id="learn-more-about-instrumentation"></a>
