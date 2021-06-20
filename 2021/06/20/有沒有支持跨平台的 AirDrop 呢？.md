> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/67112)

> 如果要我选最爱的苹果功能，必定是 AirDrop。

### **Matrix 首页推荐**

[Matrix](https://sspai.com/matrix) 是少数派的写作社区，我们主张分享真实的产品体验，有实用价值的经验与思考。我们会不定期挑选 Matrix 最优质的文章，展示来自用户的最真实的体验和观点。

文章代表作者个人观点，少数派仅对标题和排版略作修改。

如果要我选最爱的 Apple 功能，那必定是 AirDrop。在接触 AirDrop 前，我一直用着笨笨的文件传输方法——以 QQ 作为中转，需要经历「添加好友 - 上传文件 - 下载文件」的复杂流程，既耗时、又费力。

那 AirDrop 有什么好？

![](https://cdn.sspai.com/2021/06/08/8e06ef129cebcd950eeb5fecbf996040.png)简单的 AirDrop 操作

在《设计心理学》一书说过，「好的设计有两个重要特征：**可视性**（discoverability）及**易通性**（understanding）。」AirDrop 符合这两个特征：

*   可视性：在入口位于点击分享后最显眼的地方。
*   易通性：选择 AirDrop 后，立刻出现目标设备选项，设计用意明显；设计用途就是分享、发送，没有其他功能。

此外，同类竞品堆满各式各样的功能，Apple 却始终保持克制，一直在做减法；相较而言，AirDrop 操作逻辑更符合使用直觉，使用只需两步：分享文件、然后选择对象。

AirDrop 什么都好，易用、速度颇快（33Mbps+）；但谈起它，却始终绕不开**仅限 Apple 设备使用**的硬伤。所以作者群的 [@陳川端](https://sspai.com/u/5ayuq8jn/updates) 有天提出疑惑，有沒有支持跨平台的 AirDrop 呢？

![](https://cdn.sspai.com/2021/06/08/b7883b38a91f9321a097bc210baf75f3.png)鸽子群截图

有个工具不但和 AirDrop 一样好用，还开源、全平台支持并且免费。它就是我们今天的主角——Snapdrop。

开箱即用
----

受 AirDrop 启发诞生，Snapdrop 功能和界面与 AirDrop 十分相像。Snapdrop 只保留了最核心特性：

*   文件传输
*   开箱即用
*   跨平台，只要有浏览器便能用（Android、iOS、Windows、macOS、Linux）
*   离线使用

只要两个设备在同一个局域网，并且同时打开 [snapdrop.net](https://snapdrop.net) 便能使用。无需任何设置，也不用下载客户端、捣弄端口。这是我看过最**简单**的文件传输方式之一。

你会获得一个暂时名称作为当前设备的识别码。在截图中，本机的识别码是 `Lavender Marten`，在局域网发现了台 iOS iPad 叫 `Gray Hyena`。巧合的是，名字组合规律是【颜色 + 动物】。前者是薰衣草貂，后者是灰色鬣狗，或许这就是程序员的情趣吧。

![](https://cdn.sspai.com/2021/06/08/4a2e126260c8a31322515857a27299f8.png)AirDrop 既视感

类似 AirDrop，点击设备图标选取发送对象。左键在 macOS 上会弹出 Finder 文件窗口，选中文件即可发送。

如果右键点击设备图标，则会唤出文字输入框，能发送文字到目标设备。还不快发送情话给你的心仪对象，展开轰轰烈烈的爱情？

![](https://cdn.sspai.com/2021/06/08/30e58c0fd2b1887de5b5aa34dd94be81.png)土味情话大师

理论上，需要浏览器支持 WebRTC API 才能使用。Chrome、Firefox 在 2012 年的版本已经支持 WebRTC，市面主流浏览器都支持。现在应该没人还用 IE 了吧？

![](https://cdn.sspai.com/2021/06/08/a99b8bcf493eebb91cd69fc679f2fa0f.png)源：[Can I use ___ ?](https://caniuse.com/rtcpeerconnection)

以 PWA 为载体
---------

在第一次用 Snapdrop 时，Chromium 内核浏览器（e.g. Chrome、Edge）会询问你是否要**安装** Snapdrop。PWA 有着轻量化的优势，占用空间小到可以忽略。

![](https://cdn.sspai.com/2021/06/08/9a941a3f56665ab77a220ebbdc267ab5.png)PWA 安装弹窗

在测试过程中，我还发现 Snapdrop 支持 PWA 一个重要的特性——离线使用。Snapdrop 网页会自动进行**缓存**。之后就算没有连接到互联网，就像一个安装好的 App 一般，你仍然能打开 Snapdrop，进行局域网文件传输。

另外它也支持**系统通知**，当授予通知权限后，每当有新的文件传输完成，便会以原生通知形式出现。

![](https://cdn.sspai.com/2021/06/08/d7bf24682e4300822dc3e970078312c3.png)macOS 通知 Banner

速度尚可
----

这个速度和 AirDrop 相比，实在拉胯。AirDrop 能跑满 802.11n 的宽带（33 Mbps），Snapdrop 显然还差得远。不过，又不是不能用对吧？发送一些小文件非常好用。

```
65 s
```

奇怪的是，从 iPhone 发送 `.HEIC` 照片到 macOS 电脑时，会自动转换成 `.JPEG`。 好像是 Safari 为了保证兼容性做的自动转换。

要避免格式转换，把照片存到 iOS 的文档 App，在选择文件的时候，别选照片，而是选文件。

其他细节
----

Snapdrop 在隐私方面让我颇为放心：

第一，它是**开源**的。

第二，Snapdrop [承诺](https://github.com/RobinLinus/snapdrop/blob/master/docs/faq.md) 他们不会存储**任何**用户资料。

第三，技术上用户隐私得以保护。WebRTC 在文件传输时自行加密，防止其他人（包括 Snapdrop 团队）窥探用户文档。如果以上无法打动你，你甚至可以托管你的 Docker [映像](https://github.com/RobinLinus/snapdrop/blob/master/docs/local-dev.md)，建立私人的 Snapdrop 网页。

虽然我觉得 PWA 已经足够好用，你仍然能使用开源社区为 Snapdrop 开发的**第三方**客户端。

1.  基于 Electron 的 [桌面端](https://github.com/alextwothousand/snapdrop-desktop)
2.  安卓端：[GitHub](https://github.com/fm-sys/snapdrop-android)｜[Google Play](https://play.google.com/store/apps/details?id=com.fmsys.snapdrop)
3.  [Node](https://github.com/Bellisario/node-snapdrop) App

至于传输原理，Snapshare 使用了 WebRTC 协议提供的 P2P

对等式网络（英語：peer-to-peer， 简称 P2P）是无中心服务器、依靠用户群（peers）交换信息的互联网体系，它的作用在于，减低以往网路传输中的节点，以降低资料遗失的风险。与有中心服务器的中央网络系统不同，对等网络的每个用户端既是一个节点，也有服务器的功能，任何一个节点无法直接找到其他节点，必须依靠其户群进行信息交流。摘自维基百科「对等网络」词条

功能。WebRTC 受 Apple、Google、Microsoft、Mozilla、Opera 支持，是 W3C 下的开源、免费标准。开发者利用简单的 API 接口，就能在浏览器开发端对端实时通信的网络应用。

WebRTC 定义了三个常用的 Javascript API：

*   `getUserMedia` 获取用户机器上的多媒体文档。
*   `RTCPeerConnection` 建立 P2P 连接。负责信号处理、编码、端对端通信、安全、宽带管理。
*   `RTCDataChannel` 双向传输数据。

两个 peers 之间，不会「嘛哩嘛哩哄」神奇地便连接起来了。能看到原理图中，两个客户端之外，还有一个节点称作 Signaling 服务器。

![](https://cdn.sspai.com/2021/06/08/01b7bc9eb71a5afbeadf6b5a613e86ca.png)

说好的 P2P 去中心化，怎么冒出来一个服务器呢？这个 Signaling 服务器还不归 WebRTC 标准管。

Signaling 服务器的作用类似相亲里的**媒人**。一开始，两个客户端互不认识。他们需要一个中介人去介绍，才知道另一方的存在，而后使用 WebRTC 定义的 API 进行 P2P 连线及数据传输。就像相亲中，通过媒人相识，而后才能约出来吃饭。~没相亲过，应该是这样吧。（逃~

两个客户端发现对方后，文档传输的过程确实是 P2P，不像使用公有云（e.g. 百度网盘、iCloud）一般依赖中央服务器上传下载。

结语
--

Snapdrop 是我目前找到最简单的跨平台文件传输方式，过去的文件传输类应用要不需要下载客户端，要不就得要用到命令行，学习成本不低。Snapdrop 可能传输速度比较容易受路由器等网络环境因素影响，却有着最容易上手的特性，非常适合一些日常场景中小型文件的传输。

**参考**：

1.  [30-26 之 WebRTC 的 P2P 即時通信與小範例](https://mark-lin.com/posts/20180926/)
2.  [30-27 之 WebRTC 的 Signaling Server](https://mark-lin.com/posts/20180927/)

> 下载少数派 [客户端](https://sspai.com/page/client) 、关注 [少数派公众号](https://sspai.com/s/J71e) ，了解更妙的数字生活 🍃

> 想申请成为少数派作者？[冲！](https://sspai.com/apply/writing)