> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [molunerfinn.com](https://molunerfinn.com/Electron-vs-nwjs/#%E6%9E%84%E5%BB%BA%E8%BE%93%E5%87%BA)

> Electron vs nwjs[译]

写在前头：最近要写点东西，准确来说是用前端技术写桌面应用。之前曾经对 Electron 有所了解，最近在查阅资料的时候又发现了一个 “新的东西”，叫做 nwjs——前身是 node-webkit。作为一个还未入坑的人来说，到底选择哪个东西作为我们开发的工具会更好呢？我看了这个作者写的两篇文章，发现应该能给另外想要从事同样开发的或者有兴趣的人一些启发。文章不错，而且没发现有中文译版，所以我给出自己的翻译。只是兴趣之作，有错误之处还请指出。  
以下是译文：  

* * *

我最近一直在对 [Electron](https://github.com/atom/electron) 和 [nwjs](https://github.com/nwjs/nw.js) 进行深入研究，它们是能将 web 应用部署到 PC 电脑上成为可安装文件的一类办法。它们的背后都采用了相似的技术——Node.js 和 Chromium（Chrome 浏览器的引擎），而我自己也已经发现了一些细微的区别，这些区别也影响了我的开发工作（超出了纯粹的技术差异）。

_更新_：我写了一篇文章作为这篇文章的[跟进](http://www.akawebdesign.com/2015/11/02/electron-vs-nwjs-part-2/)。

> 译者注：跟进的文章的翻译在[这里](https://molunerfinn.com/Electron-vs-nwjs-part2/)

### [](#公平比较 "公平比较")公平比较

出于公平公开的考虑，我将这两个工具的准确版本公布如下：

*   Electron v0.25.2
*   nwjs v0.12.0

在写这篇文章的时候，它们都是最新的版本。

> 译者注：原文发表于 2015 年 5 月 6 号

Electron 官方有篇很好的文章阐述了[他们所认为的 electron 和 nwjs 的差异](https://github.com/atom/electron/blob/master/docs/development/atom-shell-vs-node-webkit.md)——要确保你读过它。至于我，那些技术性的差异并不是真正影响我对于这两个工具所持看法的原因，不过（对于你们来说）还是绝对值得关注的。

### [](#构建输出 "构建输出")构建输出

Electron 和 nwjs 都是十分容易上手的，我曾经各自用它们在几分钟之内就成功运行了应用。  
然而我注意到了我构建出来的 “产品” 它们在文件的体积上有着巨大的差异。二者的工程在多平台下转译成二进制文件都有不少不错的说明文档——对于我的应用（18M 的压缩过的静态网页内容），从 electron 里输出的构建文件大小（在 OSX 下总共 117MB，没有 asar 压缩）明显小于从 nwjs 里输的构建文件的大小——在 OSX 下 220MB（没有 V8 引擎的快照版本）。  
我并没有对于这些工具进行科学的研究，并且很有可能是我自己并没有使一些东西最优化…… 但是 18MB 的压缩过的静态网页文件（我的应用）确实被打包成了一个更大的体积（100/200MB）的文件。  
此外，我采用 electron 制作的应用貌似启动地更快——这可能只是把从 Node 背景下启动（electron)和 Web 背景下启动 (nwjs) 的二者比较的结果罢了。

### [](#构建过程 "构建过程")构建过程

另一方面，用 nwjs 来创建跨平台的应用貌似更加容易。  
我使用 grunt 为我本地应用进行构建，并且 nwjs 有着成堆的 grunt 插件让构建过程平稳而简单。（尽管我实际上并没有尝试为 Windows 系统构建可执行文件）  
Electron 有些 grunt 插件——但是它们并不频繁更新，并且使用上来说也有着不少困惑。特别地，[grunt-build-atom-shell](https://github.com/paulcbetts/grunt-build-atom-shell) 是一个超级奇怪的插件，你必须告诉它 electron 使用什么 tag/commit 以及 tag/commit 所依赖的 Node 的定制版本。因为我清楚地知道它们。(所以为什么还要告诉它令我很困惑)

### [](#保护源码 "保护源码")保护源码

利用 nwjs 你可以将源文件打包成一个利用某种形式的 V8 快照版本的可执行文件。我并没有真正深入它是什么或者它是怎么实现的，但是说明里明确地告诉我说你将会因此而造成 30% 的性能损失。真疼！  
对于 electron，你可以将你的应用打包成一个 ASAR 文件。这个操作真的是非常，非常简单——因为有个非常棒的 grunt 插件为你做了这些事！不过让我并不怎么开心的是，我已经能想象到一个 ASAR 文件将会相对地容易被破解——不过也已经比什么都不做的强。

### [](#所以胜者是…… "所以胜者是……")所以胜者是……

在这篇文章结束之际，我个人更倾向 electron。我喜欢它有一部分原因还来自于 Github 官方赞助了它——并且 **Visual Studio Code** 也基于它发布了 release 版本，所以这是相当赞的支持。  
定制的二进制文件对我来说将会有点麻烦，但是剩下的构建过程都挺好。当然，最后的更小体积的可执行文件也是加分项啊！

### [](#你的看法是？ "你的看法是？")你的看法是？

有没有谁用了这些工具？你们的看法是什么呢？  
请随意提供你们的一些观点——我喜欢了解一下是否有其他的注意事项我应该要考虑到的，亦或是我没有做到最好的地方。

请分享这篇文章！

* * *

> 原文地址：[http://www.akawebdesign.com/2015/05/06/electron-vs-nwjs/](http://www.akawebdesign.com/2015/05/06/electron-vs-nwjs/)

Copyright Notice: All articles in this blog are licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) unless stating additionally.