> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7137650081401929759)

原创：小姐姐味道（微信公众号 ID：xjjdog），欢迎分享，非公众号转载保留此声明。

HTTPS 是安全通道。如果浏览器导航栏前面有一个绿如 A 股的小锁，那么感觉就会非常的放心。

把自己见不得人的小心思和污言秽语，统统用这个小锁锁起来，为所欲为，想想就让人激动。

但是等等，Charles 为什么能抓到 HTTPS 的包呢？

HTTPS 简单原理
----------

我们希望数据传输过程，对用户来说是个黑盒，对攻击者来说也是个黑盒。这主要体现在两方面。

1.  用户不希望自己的敏感数据被获取到。比如自己的账号密码，比如自己发给女友的聊骚数据。
2.  开发者不希望自己的数据被用户获取到。比如自己的验签方式，被破解了用户就能干很多非法的事情。

HTTP 协议属于一问一答的协议，在传输过程中是以明文方式传递的。

如果攻击者截取了 Web 浏览器和网站服务器之间的传输报文，就可以直接读懂其中的信息。比如通过代理方式或者局域网嗅探方式获取了报文的内容。

相关的工具有很多，比如 wireshark、tcpdump、dsniff 等。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/502adf8c1a3d4ea3b207ef9ce12a7437~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?) 但如果传输的内容是加密的，那么即使你把所有的数据报文都抓到了，那么也没有什么价值。

HTTPS 就是一种传输加密数据的协议。如果我们在 TCP 与 HTTP 中间，加入一个`TLS/SSL`层，那么就会变成`HTTPS`。

HTTPS 包括握手阶段和传输阶段。其中握手阶段是最重要的协商阶段。

握手的目标，是安全的交换对称密钥，全程需要 3 个随机数的参与。在`Change Cipher Spec`之后，传输的就都是加密后的内容了。

这个过程可以使用 WireShark 抓包工具轻易抓取，有大量的文章分析握手过程，在此不再赘述。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4dc6ffa49e7d4511a01f682ff7037147~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?) 当然，HTTPS 的效率是非常低的。这里稍微扩展一下。

HTTP3，也就是谷歌的 QUIC，除了解决了队头阻塞问题，还可以作为`TCP+TLS+HTTP/2`的一种替代方案。HTTP3 默认就是安全通道，采用 UDP 协议。在 DH 秘钥交换算法的加持下，它可以减少连接建立时间 - 在常见情况下为 0 次`RTT`往返。

这比 HTTPS 的握手速度快多了。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ae1de90c4ca40f9afef1744fc91ae8b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

Charles 抓包
----------

虽然 HTTPS 的传输过程是加密的，但如果我们就是请求的发起方，设备也在自己手里，去抓包 HTTPS 连接中的内容，也是非常容易的。

这让开发者很头疼。比如我使用云平台提供的 AK、SK 直接发起 HTTPS 调用，用户是能够抓到这两个关键密码的。所以一般开发者并不能直接把 AK、SK 在网络上传递，即使这样在功能上行得通。

启动 Charles 后，我们需要把它设置成系统代理。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/993bcd4c23f741bf9a8a7af13c76b47f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

然后，在 Help / 菜单下，找到 Root 证书进行安装。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff9d022ac8f54954b41455d86d22f56d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

安装完毕之后，我们还要信任这个证书。这样，当你的浏览器访问我们的 Charles 代理时，就可以畅通无阻。

安装到 System Keychains 中，而且一定要信任它哦。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32dfbd5d0bd54e1a929febc7f5663790~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

通常情况下，我访问一个 HTTPS 连接，抓到的内容都是一团糟。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7abaee2152c4422389a850e5fe7d6400~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

我们还差最后一步。默认情况下，Charles 并没有任何过滤，我么还需要把要抓包的网址，加入 HTTPS 的代理配置中才可以。

右键找到这个连接，然后选择启动 SSL 代理即可。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db1cf0a9d2f944c7adb7c7c5fb5b304a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

此时，我们再看一下这些连接的内容，就能够变成人眼能够识别的了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0035a94da2964706919edd74b5094920~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

当然，电脑上的代理没有什么意义。我们做代理，一般是想要抓取手机上的应用产生的请求。

但方法是一样的，你只需要把这个 Root 证书，安装到你的手机中，然后信任它就可以了。

为什么能够抓到数据？
----------

在这个案例中，Charles 是作为中间人而存在的。对于 Charles 来说，对于服务端的请求，是由它发起的。

你可以把它想象成一个浏览器，它发出的请求和返回的内容，对于 Charles 自身来说自然是可见的。欺骗服务器很容易，重要的欺骗客户端。

Charles 通过伪造一个 CA 证书，来冒充一个服务端。当浏览器或者移动手机访问 Charles 冒充的服务端时，Charles 会携带 CA 证书返回给客户端。

对于普通的 CA 证书来说，浏览器和客户端是不信任的。这也是为什么要进行 HTTPS 抓包，必须安装 CA 证书的原因 --- 我们需要把这个信任关系建立起来。

这两部分是割裂的，可以说是由两条完全不同的 SSL 通道。请求报文在全程是加密的，除了一个非常薄弱的交接点。

在通道的粘合处，所有的信息却是明文的。Charles 掌控了这个过程，自然就能够把原始信息展示出来。

End
---

可以看到，Charles 是可以抓取到 HTTPS 的明文信息的。在中间人场景中，它既作为客户端发起请求，也作为服务端接收请求，然后在请求的转发处获取数据。

作为用户，我们千万不能随意信任来历不明的证书，否则你的很多隐私数据将暴露在阳光之下。

作为开发者，也不能把敏感数据直接放在 URL 或者请求体里，防止用户抓包获取到这些信息，对服务造成破坏。

当然，在 CN，隐私可能是个伪命题。就比如 xjjdog，虽然我一直在隐藏自己，但还是有很多朋友知道我到底是不是带把的。

这个时候，HTTPS 就没什么用。

> 作者简介：**小姐姐味道** (xjjdog)，一个不允许程序员走弯路的公众号。聚焦基础架构和 Linux。十年架构，日百亿流量，与你探讨高并发世界，给你不一样的味道。我的个人微信 xjjdog0，欢迎添加好友，​进一步交流。​