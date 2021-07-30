> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247491228&idx=1&sn=1b355141ab14795650d4a73370de79d3&chksm=e87312efdf049bf9e731cbda09f81e4a86bc02754ac0dfb16241b6be242c2074a21f82cb5c64&mpshare=1&scene=1&srcid=0730x5YZdrfolORVQ6Vfguay&sharer_sharetime=1627635272926&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

大家好，我是轩辕。

前几天，我在读者群里提了一个问题：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlWfWX8W1icMEXBwLu7NzURqYHdTEKIUj4jWCrblp13mlrMicoUibicj2hBg/640?wx_fmt=png)

这一下，大家总算停止了灌水（这群人都不用上班的，天天划水摸鱼），开始讨论起这个问题来。

有的说通过 **User-Agent** 可以看，我直接给了一个狗头。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlhlVzz5V7W5C9HeQn0Q0Wqa1fXcAQclFDq28pq7VLALYzTkB8sKf4Ng/640?wx_fmt=png)

然后发现不对劲，改口说可以通过 HTTP 响应的 Server 字段看，比如看到像这种的，那肯定 Windows 无疑了。

还有的说可以通过 URL 路径来判断，如果大小写敏感就是 Linux，不敏感就是 Windows。

于是我进一步提高了难度，如果连 Web 服务也没有，只有一个 TCP Server 呢？

这时又有人说：可以通过 ping 这个 IP，查看 ICMP 报文中的 **TTL** 值，如果是 xxx 就是 xx 系统，如果是 yyy 就是 yy 系统 ···（不过有些情况下也不是太准确）

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSltSxLvUbng5Ribvb4GvWoKfPL7f22qHZME9m2CseiayiadG4F9jay2jgFQ/640?wx_fmt=png)

从 TCP 重传说起
----------

今天想跟大家探讨的是另外一种方法，这个方法的思路来源于前几天被删掉的那篇文章。就是日本网络环境下访问不了极客时间的问题，当时抓包看到的情况是这个图的样子：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlSLjbEtLOfBibqMnAp2oAhTITkcLwm6vKsqJ1sXrk48w6n6LDsaPGO8A/640?wx_fmt=png)

看到了服务器后面在不断的尝试重发了吗？当时我就想到了一个问题：

**服务器到底会重传好多次呢？**

众所周知，TCP 是一种面向连接的、可靠的、基于字节流的传输层通信协议。

其中，可靠性的一个重要体现就是它的**超时重传机制**。

TCP 的通信中有一个确认机制，我发给你了数据，你得告诉我你收到没，这样双方才能继续通信下去，这个确认机制是通过序列号 SEQ 和确认号 ACK 来实现的。

简单来说，当发送方给接收方发送了一个报文，而接收方在规定的时间里没有给出应答，那发送方将认为有必要重发。

那具体最多重发多少次呢？关于这一点，RFC 中关于 TCP 的文档并未明确规定出来，只是给了一些在总超时时间上的参考，这就导致不同的操作系统在实现这一机制的时候可能会有一些差异。于是我进一步想到了另一个问题：

**会不会不同操作系统重传次数不一样，这样就能通过这一点来判断操作系统了呢？**

然后我翻看了《TCP/IP 详解 · 卷 1》，试图在里面寻找答案，果然，这本神书从来没有让我失望过：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlENFkHkQSHNefcWmkf72C2q9JzUNXbXpchk7iclYetd3tuicmlDJMMChA/640?wx_fmt=png)

这一段说了个什么事情呢？大意是说 RFC 标准中建议有两个参数 R1 和 R2 来控制重传的次数，Linux 中，这俩参数可以这样看：

```
cat /proc/sys/net/ipv4/tcp_retries1
cat /proc/sys/net/ipv4/tcp_retries2
```

`tcp_retries1`默认值是 3，`tcp_retries2`默认值是 15。

但需要特别注意的是，并不是最多重传 3 次或者 15 次，Linux 内部有一套算法，这两个值是算法中非常重要的参数，而不是重传次数本身。具体的重传次数还与`RTO`有关系，具体的算法有兴趣的朋友可以看看这篇文章：聊一聊重传次数（http://perthcharles.github.io/2015/09/07/wiki-tcp-retries/）

总体来说，在 Linux 上重传的次数不是一个固定值，而是不同的连接根据`tcp_retries2`和`RTO`计算出来的一个动态值，不固定。

而在 Windows 上，也有一个变量来控制重传次数，可以在注册表中设定它：

```
键值路径：
HKLM\System\CurrentControlSet\Services\Tcpip\Parameters

键值名：
TcpMaxDataRetransmissions

默认值：5
```

我手里有一份 Windows XP 的源码，在实现协议栈的驱动`tcpip.sys`的部分中，也印证了这个信息：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlCKjMjMianwqFDTmzM41hcTJ7YYic3h6l0ahEG59SZG22yg6mQ7KMIYHQ/640?wx_fmt=png)

从注册表中读取键值

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlMFyVuNicxhHp3YYYkV9SfbEBqjNOEzTHD5L6o90Wxb9j0DP8TicUWZPA/640?wx_fmt=png)

没有读到的默认值

不过就目前的信息来看，由于 Linux 的重传次数是不固定的，还没法用这个重传次数来判断操作系统。

TCP 之 SYN+ACK 的重传
-----------------

就在我想要放弃的时候，我再一次品读《TCP/IP 详解 · 卷 1》中的那段话，发现另一个信息：**TCP 的重传在建立连接阶段和数据传输阶段是不一样的！**

上面说到的重传次数限制，是针对的是 TCP 连接已经建立完成，在数据传输过程中发生超时重传后的重传次数情况描述。

而在 TCP 建立连接的过程中，也就是三次握手的过程中，发生超时重传，它的次数限定是有另外一套约定的。

**Linux：**

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSliavp6Ob9r4u9ayxCxuxrsV3kngc9Lh7mf1gxg8CDV5BZmzLM5Gm8dXg/640?wx_fmt=png)

在 Linux 中，另外还有两个参数来限定建立连接阶段的重传次数：

```
cat /proc/sys/net/ipv4/tcp_syn_retries
cat /proc/sys/net/ipv4/tcp_synack_retries
```

`tcp_syn_retries`限定作为客户端的时候发起 TCP 连接，最多重传 SYN 的次数，Linux3.10 中默认是 6，Linux2.6 中是 5。

`tcp_synack_retries`限定作为服务端的时候收到 SYN 后，最多重传 SYN+ACK 的次数，默认是 5

重点来关注这个`tcp_synack_retries`，它指的就是 TCP 的三次握手中，服务端回复了第二次握手包，但客户端一直没发来第三次握手包时，服务端会重发的次数。

我们知道正常情况下，TCP 的三次握手是这个样子的：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlicAciaibUMc4tuIwqe7BYK6jJmBOkb2unoJia2L8KseBuUjgC4Qd74wfIA/640?wx_fmt=png)

但如果客户端不给服务端发起第三个包，那服务端就会重发它的第二次握手包，情况就会变成下面这样：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlVJPekIsgkPhMfxCgycxy5libgd2gSfB9Br6CNiacyJIQjmmbKPHgbDfg/640?wx_fmt=png)

所以，这个`tcp_synack_retries`实际上规定的就是上面这种情况下，服务端会重传 SYN+ACK 的次数。

为了进一步验证，我使用 Python 写了一段代码，用来手动发送 TCP 报文，里面使用的发包库是 scapy，这个我之前写过一篇文章介绍它：[面向监狱编程，就靠它了！](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247490184&idx=1&sn=271fee2bc91e33dcdd780957675e2b98&scene=21#wechat_redirect)。

下面的这段代码，我向目标 IP 的指定端口只发送了一个 SYN 包，：

```
def tcp_syn_test(ip, port):

    # 第一次握手，发送SYN包
    # 请求端口和初始序列号随机生成
    # 使用sr1发送而不用send发送，因为sr1会接收返回的内容
    ans = sr1(IP(dst=ip) / TCP(dport=port, sport=RandShort(), seq=RandInt(), flags='S'), verbose=False)
```

用上面这段代码，向一台 Linux 的服务器发送，抓包来看一下：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlLHVA05vbOV3hbqu5wYt0iaHiboDyVcHMC5AfxTNwrxINOfYIwDEKGn3Q/640?wx_fmt=png)

实际验证，服务器确实重传了 5 次 SYN+ACK 报文。

一台服务器说明不了问题，我又多找了几个，结果都是 5 次。

再来看一下 Linux 的源码中关于这个次数的定义：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlN5zUXMwPNGbtSC9hhQonhD53hiaERNMh3AicxTub8CljibS50J3GVyYaw/640?wx_fmt=png)

接下来看一下 Windows 上的情况。

Windows
-------

前面说过，在注册表`HKLM\System\CurrentControlSet\Services\Tcpip\Parameters`目录下有一个叫`TcpMaxDataRetransmissions`的参数可以用来控制数据重传次数，不过那是限定的数据传输阶段的重传次数。

根据`MSDN`上的介绍，除了这个参数，还有另一个参数用来限制上面 SYN+ACK 重传的次数，它就是`TcpMaxConnectResponseRetransmissions`。

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSl3X3dUpqwNLvlIGIeF4WCYPFkq5icia1bMP7icEtBAqN6oGfIq91FNX3IA/640?wx_fmt=png)

而且有趣的是，和 Linux 上的默认值不一样，Windows 上的默认值是 **2**。

这就有意思了，通过这一点，就能把 Windows 和 Linux 区分开来。

我赶紧用虚拟机中的 XP 上跑了一个 nginx，测试了一下：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlHSW0yqUDuRKQ8cN71R0Xnyr3ia01BZ4JfVIGl4DVobdyiaHTxCBqIGKw/640?wx_fmt=png)

果然是 2 次，随后我又换了一个 Windows Server 2008，依旧是 2 次。

为了进一步验证，我通过注册表把这个值设定成了 4：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlyBsEVyo6TSTbYDzTW0zNmrjFDc9iafjbNBJfiadVv8XYytCw4tZeHaUw/640?wx_fmt=png)

再来试一下：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlWiaXpPQzKp60SdPjBQR8Hribs85jYELhicxLvI9PpL3GUXiaiawibFhqPolA/640?wx_fmt=png)

重传次数果然变成了 4 次了。

接下来在手中的 Windows XP 源码中去印证这个信息：

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSl7PkWdRiaN9eRawrwaVkzL5dMNcrn7sZOFBJ7tgYibrx18GEXVC4tTgicQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlU9lUPIibGGGx5bFDm0ECAYE8fQH9UROibrkS2GqL5cTcDUiahWSMDptgA/640?wx_fmt=png)

果然，不管是从实验还是从源码中都得到了同一个结论：

> Linux 上，SYN+ACK 默认重传 5 次。
> 
> Windows 上，SYN+ACK 默认重传 2 次。

总结
--

如果一个 IP 开启了基于 TCP 的服务，不管是不是 HTTP 服务，都可以通过向其发送 SYN 包，观察其回应来判断对方是一个 Linux 操作系统还是一个 Windows 操作系统。

当然，这种方法的局限性还是挺大的。

首先，本文只介绍了一些默认的情况，但 TCP 的重传次数是可以更改的，如果网络管理员更改了这个数值，判断的结果就不准确了。

其次，对于有些网络服务器开启了防 DDoS 功能，测试发现，其根本不会重传 SYN+ACK 包，比如我用百度的 IP 测试就得到了这样的结果。

最后，没有测试其他操作系统上的情况，比如 Unix 和 MAC OSX，为什么呢？

![](https://mmbiz.qpic.cn/mmbiz_jpg/jXQDbLkGBYW8GR2Vo9DHrWWUWrj3ZdSlZ5xmUAX1B3vBdTpXcFddA8xzWhyddy28ENbTAszQzPmugXwQs4XmLw/640?wx_fmt=jpeg)

因此，文中介绍的这种方法只能作为一种辅助手段，仅供参考，大家能顺便了解一些关于 TCP 重传的知识也是很有意义的。

好了，以上就是今天的分享了，写作不易，大家看完给个三连支持呀~

往期相关推荐
------

[抓包技术哪家强？](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247489821&idx=1&sn=58039da4499f1c12da0ba153490c1c02&scene=21#wechat_redirect)

[三次握手变成了两次！](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247490319&idx=1&sn=8d389077e528948fd9ca5939ed63239b&scene=21#wechat_redirect)

[面向监狱编程，就靠它了！](https://mp.weixin.qq.com/s?__biz=MzIyNjMxOTY0NA==&mid=2247490184&idx=1&sn=271fee2bc91e33dcdd780957675e2b98&scene=21#wechat_redirect)