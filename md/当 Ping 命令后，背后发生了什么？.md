> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488502&idx=2&sn=c95854085ac0e5e510619ae46891c3d5&chksm=ce0da475f97a2d6383be98a36e7ad6b6f64c6473c5b3f7a4d3f5bcc4e4a132ba3bdad8b20181&scene=21#wechat_redirect)

作者 | 木木匠  

链接 | my.oschina.net/luozhou/blog/2992137

01 概览

对于 `ping`命令，想必只要是程序员都知道吧？当我们检查网络情况的时候，最先使用的命令肯定是 `ping`命令吧？一般我们用 `ping`查看网络情况，主要是检查两个指标：  

*   第一个是看看是不是超时
    
*   第二个看看是不是延迟太高
    

如果超时那么肯定是网络有问题（禁 `ping`情况除外）；如果延迟太高，网络情况肯定也是很糟糕的。

那么对于 `ping`命令的原理， `ping`是如何检查网络的？大家之前有了解吗？接下来我们来跟着 `ping`命令走一圈，看看 `ping`是如何工作的。

02 环境准备和抓包

*   **环境准备**
    

抓包工具：Wireshark 准备两台电脑，进行互 `ping`操作：

1.  A 电脑（IP 地址： `192.168.2.135` / MAC 地址： `98:22:EF:E8:A8:87`）
    
2.  B 电脑（IP 地址： `192.168.2.179` / MAC 地址： `90:A4:DE:C2:DF:FE`）
    

*   **抓包操作**
    

打开 Wireshark，选取指定的网卡进行抓包，进行 ping 操作，在 A 电脑上 ping B 电脑的 IP

![](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzotvBrfibDhH71pwC5kvSgLq4C3aA1dFaeMhWANdQUo7a1h3mOrwkHI3u4qqIarhWowvCTmwMqln2A/640?wx_fmt=png)

抓包情况如下：

![](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzotvBrfibDhH71pwC5kvSgLqEc2frETE7hNMsVicAleG6hN9X6LibNeiaFjr6aDj1VcDLyjb9CrQvBMWA/640?wx_fmt=png)

这里先简单的介绍下 Wireshark 的控制面板，这个面板包含 7 个字段，分别是：

*   `NO`: 编号
    
*   `Time`: 包的时间戳
    
*   `Source`: 源地址
    
*   `Destination`: 目标地址
    
*   `Protocol`: 协议
    
*   `Length`: 包长度
    
*   `Info`: 数据包附加信息
    

03 深入解析

上图中抓包编号 `54-132` 显示的就是整个 `ping`命令的过程，我们知道 `ping`命令不是依托于 TCP 或者 UDP 这种传输层协议的，而是依托于 `ICMP`协议实现的， 那么什么是 `ICMP` 协议呢？这里简单介绍下：

*   **ICMP 协议的产生背景**
    

[RFC792] 中说明了 `ICMP`产生的原因：由于互联网之间通讯会涉及很多网关和主机，为了能够报告数据错误，所以产生了 `ICMP`协议。也就是说 `ICMP` 协议就是为了更高效的转发 IP 数据报和提高交付成功的机会。

*   **ICMP 协议的数据格式**
    

![](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzotvBrfibDhH71pwC5kvSgLqtQLPPpiccxThb5xvVSqjELg9Usmvd6fR1CgjfPwSEGTushJBFicknTMQ/640?wx_fmt=png)

根据上图我们知道了 `ICMP`协议头包含 4 个字节，头部主要用来说明类型和校验 `ICMP`报文。下图是对应的类型和代码释义列表，我们后面分析抓包的时候会用到。

![](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzotvBrfibDhH71pwC5kvSgLqkCyTQLov4Rl6ThkyTfjb1SiabYMcic8gqfdQgAgmubFar9ibeGv6oLo5w/640?wx_fmt=png)

简单介绍完了 `ICMP`，那么抓包过程中出现的 `ARP`协议是什么呢？我们同样来简单解释下：

*   **ARP 协议**
    

我们知道，在一个局域网中，计算机通信实际上是依赖于 `MAC`地址进行通信的，那么 `ARP`（ `AddressResolutionProtocol`）的作用就是根据 IP 地址查找出对应的 MAC 地址。

*   **Ping 过程解析**
    

了解了上面的基础概念后，我们来分析下抓包的数据，其流程如下：

1.  A 电脑（ `192.168.2.135`）发起 `ping`请求， `ping192.168.2.179`
    
2.  A 电脑广播发起 `ARP`请求，查询 `192.168.2.179`的 MAC 地址。
    
3.  B 电脑应答 `ARP`请求，向 A 电脑发起单向应答，告诉 A 电脑自己的 MAC 地址为 `90:A4:DE:C2:DF:FE`
    
4.  知道了 MAC 地址后，开始进行真正的 ping 请求，由于 B 电脑可以根据 A 电脑发送的请求知道 **源 MAC 地址**，所以就可以根据源 MAC 地址进行响应了。
    

上面的请求过程我画成流程图比较直观一点：

![](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzotvBrfibDhH71pwC5kvSgLqlJiazUvTv3aVX3ETB6SImlPoULv8vhe37bI5y9CjSLC24gMlqO9eDXA/640?wx_fmt=png)

观察仔细的朋友可能已经发现，Ping 4 次请求和响应结束后，还有一次 B 电脑对 A 电脑的 ARP 请求，这是为什么呢？这里我猜测应该是有 2 个原因：

*   由于 `ARP`有缓存机制，为了防止 `ARP`过期，结束后重新更新下 `ARP`缓存，保证下次请求能去往正确的路径，如果 `ARP`过期就会导致出现一次错误，从而影响测试准确性。
    
*   由于 ping 命令的响应时间是根据请求包和响应包的时间戳计算出来的，所以一次 `ARP`过程也是会消耗时间。这里提前缓存最新的 `ARP`结果就是节省了下次 `ping`的 `ARP`时间。
    

为了验证我们的猜测，我再进行一次 `ping`操作，抓包看看是不是和我们猜测的一样。此时，计算机里面已经有了 ARP 的缓存，我们执行 `ARP-a` 看看缓存的 arp 列表：

![](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzotvBrfibDhH71pwC5kvSgLqHuHvbUWsMMu3LGkAjaXibRwpgI47XlabUaL4FwzHstsr7uOlVxGCI3g/640?wx_fmt=png)

我们看看第二次 `ping`的抓包

![](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzotvBrfibDhH71pwC5kvSgLq5Iic55tL2Qy5jplqFsfsmH9KMKx51R2CsetOSibib7shb2fXm0z0dCxCw/640?wx_fmt=png)

我们看到上图中在真正 `ping`之前并没有进行一次 `ARP`请求，这也就是说，直接拿了缓存中的 `ARP`来执行了，另外当 B 计算机进行响应之前还是进行了一次 `ARP`请求，它还是要确认下之前的 `ARP`缓存是否为正确的。然后结束`ping`操作之后，同样再发一次 `ARP`请求，更新下自己的 `ARP`缓存。这里和我们的猜想基本一致。

弄懂了 `ping`的流程之后我们来解析下之前解释的 `ICMP`数据结果是否和抓包的一致。我们来点击一个 `ping request`看看 `ICMP`协议详情

![](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzotvBrfibDhH71pwC5kvSgLqQU7l4oQxptJCVde9UcIrq6Bl6jscuBT1XoLTsqibJ6Tlziady4OTKH0Q/640?wx_fmt=png)

图中红框内就行 `ICMP`协议的详情了，这里的 `Type=8,code=0`, 校验是正确，且这是一个请求报文。我们再点击`Responseframe:57`，这里说明响应报文在序号 `57`。详情如下：

![](https://mmbiz.qpic.cn/mmbiz_png/xq9PqibkVAzotvBrfibDhH71pwC5kvSgLqfJ37nNMyBFicD1h3b2b1ibeyzV6f3TdXFU2S5YJZicH2jJdLj8ReffQFg/640?wx_fmt=png)

上图的响应报文， `Type=0,code=0`，这里知道就是响应报文了，然后最后就是根据请求和响应的时间戳计算出来的响应延迟。 `3379.764ms-3376.890ms=2.874ms`。

04 总结

我们分析了一次完整的 `ping`请求过程， `ping`命令是依托于 `ICMP`协议的， `ICMP`协议的存在就是为了更高效的转发 IP 数据报和提高交付成功的机会。 `ping`命令除了依托于 `ICMP`，在局域网下还要借助于 `ARP`协议， `ARP`协议能根据 IP 地址反查出计算机的 MAC 地址。另外 `ARP`是有缓存的，为了保证 `ARP`的准确性，计算机会更新 ARP 缓存。