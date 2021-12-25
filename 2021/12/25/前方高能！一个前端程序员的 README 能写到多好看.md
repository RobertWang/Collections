> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7022299474458312718) <div align=center> # 你的 Markdown 内容 </div> 复制代码

### 2.2 Badges

在我的 Markdown 中，有两处用到了五彩缤纷的 `badges`，分别是 **联系方式** 和 **前端技术栈**。

两处的 `badges` 图标均由 [shields.io](https://link.juejin.cn?target=https%3A%2F%2Fshields.io "https://shields.io") 网站提供。在这个网站上，你只需要提供需要显示的内容，就能得到一个精美的 `badge`。你还可以自定义样式，可以说非常灵活。

联系方式部分，还使用到了 [SubStats](https://link.juejin.cn?target=https%3A%2F%2Fsubstats.spencerwoo.com%2F%23why-i-did-this "https://substats.spencerwoo.com/#why-i-did-this")。这是一个致力于提供社交帐号粉丝数统计服务的开源项目，你只需要按照指定格式要求，就能从他们的开放接口获得你在某个平台的粉丝数量。然后在 [shields.io](https://link.juejin.cn?target=https%3A%2F%2Fshields.io "https://shields.io") 的 `Dynamic` 部分，提供固定格式的查询信息，就能 **将动态查询到的数据与 `badges` 结合起来使用**。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da9c910b83b74e48b8341c88eadf176d~tplv-k3u1fbpfcp-watermark.awebp)

### 2.3 仓库卡片 & 分析统计

这两个部分使用到了 [GitHub Readme Stats](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fanuraghazra%2Fgithub-readme-stats "https://github.com/anuraghazra/github-readme-stats") 和 [GitHub Profile Trophy](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fryo-ma%2Fgithub-profile-trophy "https://github.com/ryo-ma/github-profile-trophy") 开源项目。

你只需复制你想要的卡片链接，在 Markdown 中插入对应图片，修改 **用户名**、**仓库名** 等信息即可。这个开源项目的 README 文件中由具体的使用方式。

### 2.4 阅读数统计

阅读数统计功能使用了 [GitHub Profile Views Counter](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fantonkomarev%2Fgithub-profile-views-counter "https://github.com/antonkomarev/github-profile-views-counter") 开源项目。

只需在 Markdown 中插入 `View Counter` 提供的 `badge`，然后在 Markdown 的末尾插入一个由该开源项目提供的 `hitter` （一个大小为 1 平方像素的透明图片）。

这样，每当有人访问你的 `profile`，`hitter` 都会自动被请求，这会帮你为你的 `view count` 数量加 1。

三、作用
----

GitHub 在之前的一次更新中，为每位 GitHub 用户的仓库加入了魔法——只需要创建一个命名和自己的 **账户名称** 完全一样的仓库，并在这个仓库的根目录放入一个 `README.md` 文件，该文件将展示在个人 GitHub 首页。

如果这篇文章对你编写个人 GitHub 主页的介绍文件有帮助的话，烦请帮我点一个宝贵的 👍 你的支持是我创作的最大动力~

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5f795f4705441d5be43717bdd55ea4c~tplv-k3u1fbpfcp-watermark.awebp)

> 外观花哨起来了，下一步就是充实内容了，加油！