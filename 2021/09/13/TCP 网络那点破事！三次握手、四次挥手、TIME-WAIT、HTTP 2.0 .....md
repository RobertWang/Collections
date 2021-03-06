> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650816922&idx=4&sn=7477a89748ed9ed0bee5096586445ab7&chksm=bd01d4d48a765dc2c907add4cd44aee0b796e3b448c2f18608c4a5bbf62f22bfb24c951c6a88&mpshare=1&scene=1&srcid=0913RAR3xdkricJH99DRP6YZ&sharer_sharetime=1631530099934&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

出品 | 微观技术（ID：weiguanjishu）

如若转载请联系原公众号

今天主要给各位分享`TCP网络`的一些常见知识点，日常工作或面试会经常遇到。考虑内容篇幅不小，建议先收藏，慢慢咀嚼。

如果有帮助，也请转给身边的朋友们，” 独乐乐不如众乐乐 “

首先，来个目录，让大家对文章内容先有个直观了解

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwdRomH3FoIUyOnHOhxmENBvYWY9TIB7H14hiaic9lK8tjQN1gbQusSaQgWOvyYC9icVTPa0LX7W5ZKDA/640?wx_fmt=png)

**网络的七层模型，简单介绍每层的作用？**  

答案：分为 7 层，从下到上依次是：

*   应用层：计算机用户与网络之间的接口，常见的协议有：HTTP、FTP、 SMTP、TELNET
    
*   表示层：数据的表示、安全、压缩。将应用处理的信息转换为适合网络传输的格式。
    
*   会话层：建立和管理本地主机与远程主机之间的会话。
    
*   传输层：定义传输数据的`协议端口号`，以及流控和差错校验，保证报文能正确传输。协议有 TCP、UDP
    
*   网络层：路由选择算法，进行逻辑地址寻址，实现不同网络之间的最佳路径选择。协议有 IP、ICMP
    
*   数据链路层：接收来自物理层的位流形式的数据，并封装成帧，传送到上一层；同样，也将来自上层的数据帧，拆装为位流形式的数据转发到物理层。这一层的数据叫做帧。
    
*   物理层：建立、维护、断开物理连接。传输比特流（将 1、0 转化为电流强弱来进行传输，到达目的地后在转化为 1、0，也就是我们常说的数模转换与模数转换）。这一层的数据叫做比特。
    

**TCP 报文首部有哪些字段?**

答案：

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwdRomH3FoIUyOnHOhxmENBvaZ1PKzDu2mMvjvwlT1M5n9NR9F72xajuz4IicibJheSacQOFM0j3jxRQ/640?wx_fmt=png)

*   源端口、目的端口：各占 2 个字节，表示数据从哪个进程来，去往哪个进程
    
*   序号（Sequence Number）：占 4 个字节，TCP 连接中传送的数据每一个字节都会有一个序号
    
*   确认号（Acknowledgement Number）：占 4 个字节，另一方发送的 tcp 报文段的响应
    
*   数据偏移：头部长度，占 4 个字节，表示 TCP 报文段的数据距离 TCP 报文段的起始处有多远。
    
*   6 位标志位：
    

*   URG：紧急指针是否有效
    
*   ACK：表示确认号是否有效
    
*   PSH：提示接收端应用程序立刻将数据从 tcp 缓冲区读走
    
*   RST：表示要求对方重新建立连接
    
*   SYN：这是一个连接请求或连接接受的报文
    
*   FIN：告知对方本端要关闭连接
    

*   窗口大小：占 4 个字节，用于 TCP 流量控制。告诉对方本端的 TCP 接收缓冲区还能容纳多少字节的数据，这样对方就可以控制发送数据的速度。
    
*   校验和：占 2 个字节，由发送端填充，接收端对 TCP 报文段执行 CRC 算法以检验 TCP 报文段在传输过程中是否损坏。检验的范围包括头部、数据两部分，是 TCP 可靠传输的一个重要保障。
    
*   紧急指针：占 2 个字节，一个正的偏移量。它和序号字段的值相加表示最后一个紧急数据的下一个字节的序号，用于发送端向接收端发送紧急数据。
    

**TCP 三次握手过程？**

答案：目的是同步连接双方的序列号和确认号，并交换 TCP 窗口。

*   第一次握手，客户端发送 (seq=x)，客户端进入`SYN_SEND`状态
    
*   第二次握手，服务端响应 (Seq=y, Ack=x+1)，服务器端就进入`SYN_RCV`状态。
    
*   第三次握手，客户端收到服务端的确认后，发送 (Ack=y+1)，客户端进入`ESTABLISHED`状态。当服务器端接收到这个包时，也进入`ESTABLISHED`状态。
    

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwdRomH3FoIUyOnHOhxmENBvsxho6J2Z0LjgdMJibJOgO0qAT6NlQcazaaHSnWSu2Fj300Ay6PswhWg/640?wx_fmt=png)

  

**为什么是三次握手，而不是两次或四次？**

答案：

如果只有两次握手，那么服务端向客户端发送 `SYN/ACK` 报文后，就会认为连接建立。但是如果客户端没有收到报文，那么客户端是没有建立连接的，这就导致服务端会浪费资源。

使用两次握手无法建立 TCP 连接，而使用三次握手是建立连接所需要的最小次数

**TCP 四次挥手的过程？**

答案：

*   第一次挥手：客户端向服务端发送连接释放报文
    
*   第二次挥手：服务端收到连接释放报文后，立即发出确认报文。这时 TCP 连接处于半关闭状态，即客户端到服务端的连接已经释放了，但是服务端到客户端的连接还未释放。表示客户端已经没有数据发送了，但是服务端可能还要给客户端发送数据。
    
*   第三次挥手：服务端向客户端发送连接释放报文
    
*   第四次挥手：客户端收到服务端的连接释放报文后，立即发出确认报文。此时，客户端就进入了 `TIME-WAIT` 状态。注意此时客户端到 TCP 连接还没有释放，必须经过 2*MSL（最长报文段寿命）的时间后，才进入`CLOSED` 状态。
    

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwdRomH3FoIUyOnHOhxmENBv7BicFBax3vfnWmRNC5p9Hx3gYiaibfIpPprOChqFcf7iawVF369XYX6RSA/640?wx_fmt=png)

**为什么需要四次挥手？**

答案：TCP 是`全双工`。一方关闭连接后，另一方还可以继续发送数据。所以四次挥手，将断开连接分成两个独立的过程。

**客户端 TIME-WAIT ，为什么要等待 2MSL 才进入 CLOSED 状态？**

答案：MSL 是报文段在网络上最大存活时间。

确保 ACK 报文能够到达服务端，从而使服务端正常关闭连接。客户端在发送完最后一个 ACK 报文段后，再经过 2MSL，就可以保证本连接持续的时间内产生的所有报文段都从网络中消失。这样就可以使下一个连接中不会出现这种旧的连接请求报文段。

**一台 8G 内存服务器，可以同时维护多少个连接？**

答案：发送、接收缓存各 4k，还要考虑 socket 描述符，一个 tcp 连接需要占用的最小内存是 8k，那么最大连接数为：`8*1024*1024 K / 8 K = 1048576 个`，即约 100 万个 tcp 长连接。

**什么是拆包？**

答案：传输层封包不能太大，基于这个限制，往往以缓冲区大小为单位，将数据拆分成多个 TCP 段（`TCP Segment`）传输。在接收数据的时候，一个个 `TCP 段`又被重组成原来的数据。简单来讲分为几个过程：拆分——传输——重组。

**什么是粘包？**

答案：解决数据太小问题，防止`多次发送`占用资源。TCP 协议将它们合并成一个 TCP 段发送，在目的地再还原成多个数据。

**缓冲区是做什么用？**

答案：缓冲区是在内存中开辟的一块区域，目的是缓冲。当应用频繁地通过网卡收、发数据，网卡只能一个一个处理。当网卡忙不过来的时候，数据就需要排队，也就是将数据放入缓冲区。

注意：TCP Segment 的大小不能超过缓冲区大小。

**TCP 协议是如何保证数据的顺序？**

答案：

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwdRomH3FoIUyOnHOhxmENBv5FJQGXYRMxECxFc2Uzjm5vNyia2Bhg3hRFRpfsjWSHNBEfccymJc35w/640?wx_fmt=png)

大数据拆包成多个片段，发送可以保证有序，但是由于网络环境复杂，并不能保证它们到达时也是有序的，为了解决这个问题，对每个片段用`Sequence Number`编号，接收数据的时候，通过 Seq 进行排序。

注意：seq 是累计的发送字节数

**TCP 协议如何解决丢包？**

答案：丢包需要重发，关键是如何判断有没有丢包！

每一个数据包，接收方都会给发送方发响应。每个 TCP 段发送时，接收方已经接收了多少数据，用 `Acknowledgement Number`（简写 ACK） 表示。

注意：ack 是累计的接收字节数，表示这个包之前的包都已经收到了。

**什么是 MSS ?**

答案：MSS 全称 `Maximun Segment Size`。是 TCP Header 中的可选项（Options），控制了 TCP 段的大小，不能由单方决定，需要双方协商。

**TCP 协议如何控制流量传输速度？**

答案：简单讲通过`滑动窗口`。发送、接收窗口的大小可以用来控制 TCP 协议的流速。窗口越大，同时可以发送、接收的数据就越多，吞吐量也就越大。但是窗口越大，如果数据发生错误，损失也就越大，因为需要重传越多的数据。

TCP 每个请求都要有响应，如果一个请求没有收到响应，发送方就会认为这次发送出现了故障，会触发重发。为了提升吞吐量，一个 TCP 段再没有收到响应时，可以继续发送下一个段。

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwdRomH3FoIUyOnHOhxmENBvdoNu7Giaza2xicichwJvCpVK5P7pDaxbWPzYcuPWelonKzLNIzgQcibRww/640?wx_fmt=png)

  

*   窗口区域包含两类数据：已发送未确认、未发送（即将发送）
    
*   窗口中序号最小的分组如果收到 ACK，窗口就会向右滑动
    
*   滑动窗口的 size 规格可能会变化，需要从 ACK 数据包实时取最新值
    
*   如果最小序号的分组长时间没有收到 ACK，就会触发整个窗口的数据重新发送
    

**HTTP 1.0 、1.1 和 HTTP 2.0 有什么区别？**

答案：

1、HTTP 1.0

*   默认是短连接，每次与服务器交互，都需要新开一个连接。
    

2、HTTP 1.1

*   默认持久化连接，建立一次连接，多次请求均由这个连接完成。
    

3、HTTP 2.0

*   二进制分帧：在应用层和传输层之间加了一个二进制分帧层，将所有传输的信息分割为更小的消息和帧（frame），并对它们采用二进制格式的编码。减少服务端的压力，内存占用更少，连接吞吐量更大
    
*   多路复用：允许同时通过单一的 HTTP/2.0 连接发起多次的请求 - 响应消息。
    
*   头部压缩：采用了`Hpack`头部压缩算法对 Header 进行压缩，减少重复发送。
    
*   服务器推送：服务器主动将一些资源推送给浏览器并缓存起来。
    

**HTTP 与 HTTPS 的区别？**

答案：HTTPS = HTTP + SSL/TLS

*   HTTP 采用明文通讯；端口 80
    
*   HTTPS 在 HTTP 的基础上加入了`SSL/TLS协议`，SSL/TLS 依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密。端口 443
    

**HTTP 协议为什么要设计成无状态？**

答案：HTTP 是一种无状态协议，每个请求都是独立执行，请求 / 响应。这样设计的重要原因是，降低架构设计复杂度，毕竟服务器一旦带上了状态，`扩容、缩容、路由`都会受到制约。无状态协议不要求服务器在多个请求期间保留每个用户的信息。

但，你可能会问，如果有登录要求的业务怎么办？HTTP 协议提供扩展机制，Header 中增加了 Cookie，存储在客户端，每次请求时自动携带，采用空间换时间机制，满足上下请求关联。虽然浪费了些网络带宽，但是减少了复杂度。当然为了减轻网络负担，浏览器会限制 Cookie 的大小，不同浏览器的限制标准略有差异，如：Chrome 10，限制最多 180 个，每个 Cookie 大小不能超过 4096 bytes

**HTTPS 的访问流程是什么？**

答案：

*   客户端发起一个 http 请求，告诉服务器自己支持哪些 hash 算法。
    
*   服务端把自己的信息以`数字证书`的形式返回给客户端（公钥在证书里面，私钥由服务器持有）。
    
*   客户端收到服务器的响应后会先验证证书的合法性（证书中包含的地址与正在访问的地址是否一致，证书是否过期）
    
*   如果证书验证通过，就会生成一个随机的`对称密钥`，用证书的公钥加密。
    
*   客户端将证书公钥加密后的密钥发送给服务端
    
*   服务端用私钥解密，解密之后就得到客户端的密钥
    
*   然后，客户端与服务端就靠密钥完成明文加密、安全通信、对称解密
    

**对称加密与非对称加密有什么区别？**

答案：

*   对称加密。加密和解密使用同一个密钥。速度快。常用的如：AES、DES
    
*   非对称加密。公钥与私钥配对出现，公钥对数据加密，私钥对数据解密。常用的如：RSA、DSS
    

**TCP 抓包用什么工具？**

答案：Wireshark，应用最广泛的网络协议分析器。功能非常丰富

*   支持数百个协议
    
*   实时捕获、离线分析
    
*   支持 Windows、Linux、macOS、Solaris、FreeBSD、NetBSD 等平台；
    
*   界面化操作
    
*   支持 Gzip
    
*   支持 IPSec