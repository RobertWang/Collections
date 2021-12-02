> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6980575404809519111)*   可以提供更多的历史信息，方便快速浏览；
*   可以过滤某些 `commit`（比如文档改动），便于快速查找信息；
*   可以直接从 `commit` 生成 `Change log`；

用的什么规范？
-------

```
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
复制代码

```

其中，Header 是必需的，Body 和 Footer 可以省略。

### type

`type` 用于说明 commit 的类别。

*   `feature` A new feature
*   `fix` A bug fix
*   `docs` Documentation only changes
*   `style` Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
*   `refactor` A code change that neither fixes a bug nor adds a feature
*   `perf` A code change that improves performance
*   `test` Adding missing tests or correcting existing tests
*   `build` Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)
*   `ci` Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)
*   `chore` Other changes that don't modify src or test files
*   `revert` Reverts a previous commit

### scope

`scope` 用于说明 commit 影响的范围，比如数据层、控制层、视图层、具体模块等等，视项目不同而不同。

### subject

`subject` 是 commit 目的的简短描述，不超过 50 个字符。

### body

`Body` 部分是对本次 commit 的详细描述，可以分成多行。

### footer

`BREAKING CHANGE`，用来描述当前 commit 与上一个版本不兼容的地方。

`Issue`，用来描述当前 commit 针对的某个 issue。

### 参考文章

[Commit message 和 Change log 编写指南](https://link.juejin.cn?target=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2016%2F01%2Fcommit_message_change_log.html "http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html")

用的什么辅助工具？
---------

太教条了，太累... 给大家分享一个我使用的工具。

`JetBrains IDE` 插件，在 `GoLand`、`PhpStorm` 中 都可以在插件市场搜索 `Git Commit Message Helper`。

插件地址：[Git Commit Message Helper](https://link.juejin.cn?target=https%3A%2F%2Fplugins.jetbrains.com%2Fplugin%2F13477-git-commit-message-helper "https://plugins.jetbrains.com/plugin/13477-git-commit-message-helper")

安装后效果，在 git commit 时：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54a8ba72c7254bdaa6813ba60c750ca3~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/639700f1367a4d39b10f896a5ea666ee~tplv-k3u1fbpfcp-watermark.awebp)

咱们看一下效果：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/420e16132399408da42d08e65801fe67~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/512a116b8cf34e2fbc2d3e735ae67bf4~tplv-k3u1fbpfcp-watermark.awebp)

这时，点击 Commit 或 Commit and Push... 即可。

赶快去体验吧。

推荐阅读
----

*   [使用 Docker 快速搭建多版本 PHP 开发环境](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F-vOmWMmzkzAEPbcbJQfmJw "https://mp.weixin.qq.com/s/-vOmWMmzkzAEPbcbJQfmJw")
*   [函数的不定参数你是这样用吗？](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FjvSbZ0_g_EFqaR2TmjjO8w "https://mp.weixin.qq.com/s/jvSbZ0_g_EFqaR2TmjjO8w")
*   [优雅地处理错误真是一门学问啊！](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FW_LsZtnjGIKQ-LB6EkRgBA "https://mp.weixin.qq.com/s/W_LsZtnjGIKQ-LB6EkRgBA")