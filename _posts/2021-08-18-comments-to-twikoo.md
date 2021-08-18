---
layout: post
title: "评论系统迁移到 Twikoo"
author: maxelbk
categories: [ "日常" ]
---

今天将博客的评论系统从 Disqus 迁移到了完全自由的 Twikoo 。~~整个博客终于纯 Markdown 化~~

[![Twikoo](https://github.com/imaegoo/twikoo/blob/dev/docs/static/logo.png)](https://twikoo.js.org/)

在网页上添加的过程不是很难，三下五除二注释掉 Disqus ，在 _includes 中添加 `twikoo.html` 然后在评论位置引入这个文件，顺带还做了个简单的评论系统切换。（感觉这个修改版主题可配置项越来越多了）

最简单的方式就是将这段代码添加到文章页面中评论的部分<!--MORE-->（Jekyll 一般在 `_layout/page.html` 中）：

```html
<section>
    <div id="tcomment"></div>
    <script src="https://cdn.jsdelivr.net/npm/twikoo@1.4.4/dist/twikoo.all.min.js"></script>
    <script>
        twikoo.init({
            envId: '', // 环境ID，Vercel 方案中就是 Vercel 给你的那个地址(包括开头的https)
            el: '#tcomment',
            //region: 'ap-guangzhou', // 环境地域，默认为 ap-shanghai，环境地域不是上海需传此参数
            path: 'window.location.pathname', // 一般不需要更改
            lang: '{{ site.twikoo.lang }}', // 评论区语言
        })
    </script>
</section>
```

其中支持的语言可以在这里查看: [支持的语言列表](https://github.com/imaegoo/twikoo/blob/dev/src/js/utils/i18n/index.js)

然后就是 Twikoo 服务器系统的搭建，这里我选择了 Vercel 的方案（~~是的继续白嫖~~），配合 MongoDB 的免费方案完全可以代替常规的服务器。具体如何操作在 [Twikoo 官方文档](https://twikoo.js.org/quick-start.html#vercel-%E9%83%A8%E7%BD%B2) 写的很清楚，这里不再赘述。

一切就绪，提交推送构建一条龙。

构建成功后，随便进一个文章，找到评论区，点右下角小齿轮图标，会提示设置密码，设置之后就配置完成了。

Twikoo 还支持从 Disqus 导入评论数据，不过我这边 Disqus 导出时直接给了我一个排队中...所以没办法...建议谨慎使用 Disqus 的免费版本...

另外这个博客由 GitHub Pages 切换到了 Vercel 的网页托管服务，由此域名也变成了 [https://maxelbk.vercel.app](https://maxelbk.vercel.app)。~~DP7.LINK域名准备中~~
