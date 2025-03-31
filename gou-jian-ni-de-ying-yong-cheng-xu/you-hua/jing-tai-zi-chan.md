# 静态资产

Next.js 可以在根目录下的一个名为 `public` 的文件夹中提供静态文件，例如图片。文件 `public` 中的内容可以通过从基本 URL ( `/` ) 开始的代码进行引用。

例如，通过访问 `/avatars/me.png` 路径可以查看文件 `public/avatars/me.png` 。显示该图像的代码可能如下所示：

{% code title="avatar.js" %}
```java
import Image from 'next/image'
 
export function Avatar({ id, alt }) {
  return <Image src={`/avatars/${id}.png`} alt={alt} width="64" height="64" />
}
 
export function AvatarOfMe() {
  return <Avatar id="me" alt="A portrait of me" />
}
```
{% endcode %}

### [缓存](https://nextjs.org/docs/app/building-your-application/optimizing/static-assets#caching) <a href="#caching" id="caching"></a>

Next.js 不能安全地缓存 `public` 文件夹中的资产，因为它们可能会更改。应用的默认缓存头是：

```
Cache-Control: public, max-age=0
```

### [机器人、网站图标和其他](https://nextjs.org/docs/app/building-your-application/optimizing/static-assets#robots-favicons-and-others) <a href="#robots-favicons-and-others" id="robots-favicons-and-others"></a>

对于静态元数据文件，如 `robots.txt` 、 `favicon.ico` 等，您应该在 `app` 目录中使用特殊的元数据文件。

> &#x20; 注意：
>
> * 目录必须命名为 `public` 。名称不能更改，并且这是用于提供静态资产的唯一目录。
> * 仅在构建时位于 `public` 目录中的资产将由 Next.js 提供服务。在请求时添加的文件将不可用。我们建议使用像 Vercel Blob 这样的第三方服务进行持久文件存储。
