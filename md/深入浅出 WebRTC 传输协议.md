> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XMhSDABc74dpALIHrPPt7w)

大厂技术 坚持周更 精选好文
==============

RTC 是 Real-Time Communication 的简写，正如其中文名称 “即时通讯” 的意思一样，RTC 协议被广泛用于各种即时通讯领域，诸如：

*   在线教育；
    
*   直播中的主播连麦 PK；
    
*   日常生活的音视频电话；
    
*   ......
    

WebRTC 则是 Google 基于 RTC 协议实现的一个开源项目，为 Web 页面提供了实时音视频传输所需的能力（前端部分）；

RTC 有一个非常重要的特性，它是一个支持点对点直接传输的 P2P 协议；

**P2P**
-------

P2P 是 “Peer to Peer” 的简写，在金融领域大家应该都听过这个名词（P2P 暴雷），金融中 P2P 可以指代 “个人对个人的网络放贷”；在互联网中也有着类似的意思，表示数据 “点对点传输” ，指数据可以在两个互联网用户之间直接传输，无需服务器在中间进行转发；

举个例子：IM 聊天软件中有 A、B 两个正在聊天的用户，聊天过程中，用户 A 的文字信息是没办法直接通过网络发送给用户 B 的，而是需要一个服务器 S 在中间做转发，“A -> S -> B”；但采用 RTC 协议的视频电话就不一样了，电话的音视频数据可以通过网络直接在两个用户之间传输，无需中间服务器进行转发：“A -> B”；

采用 RTC 协议能带来两个非常大的优势：

*   大幅度降低服务端的负载，减少成本；
    
*   用户间直接进行数据传输，延迟上能带来不小的提升；
    

NAT “墙”
=======

从上文知道，RTC 是一个 P2P 协议，数据可以直接在用户之间传输；但现实往往比理论来的复杂，实际用户的网络环境并没有那么简单，如果不做一些特殊处理，数据很大概率无法在两个用户之间传输，之所以无法直接传输，为了大家更好的理解需要从头说起；

随着互联网的用户逐年增多，接入互联网的设备也越来越多，公网 IPv4 的地址池慢慢见底，新接入互联网的设备很难再分配到单独公网的 IPv4 地址，为了解决这个问题，引入了一个叫 NAT（Network address translation）的协议；新接入的设备不再直接分配公网的 IPv4 地址，而是躲在 NAT 设备（路由器等）之后，NAT 会给后面的每一个设备都分配一个单独的内网地址，NAT 内部将维护一个内外网的地址映射表格，举个例子：

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIprdVsJwia60665StiaMyS6BTwUBFBicWicXqKBicQHCERkIrOf4OEibSqQBWBnWzmqJyCYgFmo0tzlLefQ/640?wx_fmt=png)

NAT 设备会修改发出去和收到的数据包，比如把上面 “192.168.0.1:8088” 发出去的包改成 “220.181.38.149:1111” ，这样外部的设备就会以为自己是在跟 “220.181.38.149:1111” 通信，接收到响应包之后，NAT 会把目标地址 “220.181.38.149:1111” 修改为 “192.168.0.1:8088”，随后转发到内网；这样就实现了一个公网的 IPv4 地址给多个设备共同使用的效果；

NAT 在实现上述功能之后，为了内网设备不被攻击，还使用了两个安全策略：

*   NAT 超时：NAT 维护的内外网地址映射表存在超时时间，一段时间内没有数据传输，对应的映射就会被取消，造成连接链路中断；
    
*   NAT 墙：NAT 还实现了类似防火墙的能力，外部主动发送给内部设备的数据包到了 NAT 之后可能会被丢弃；
    

根据 NAT 墙策略的不同，最常见的 NAT 可以分为四种：

1.  Full Cone NAT（完全锥形）：表示映射表中所有的地址，外网设备都可以直接访问到，是最宽松的策略了；
    

2.  Restricted Cone NAT（IP 限制锥形）：没被访问过的外网地址发的数据包都会被丢弃，用上面的例子来解释，假如 “192.168.0.1:8088” 访问过外网 “103.15.99.89:80” 这个地址，那外网 “103.15.99.89” 这个 IP 的所有端口都将可以访问通内网，但其他外网地址的访问会被阻止；
    

3.  Port Restricted Cone NAT（端口限制锥形）：类似 2，不过除了限制外网的 IP 地址外还会限制端口；
    

4.  Symmetric NAT（对称形）：这种 NAT 的丢包策略和 3 一样，只要外网的 IP 和端口有一个没被访问过，数据包就会被丢弃；但是该类型的 NAT 内外网地址映射的策略不一样，对称型 NAT 不会直接给一个内网设备分配固定的 IP 和端口，而是根据访问的外网地址分配不同的 IP 和端口；举个例子，假设内网设备 A 访问外网 B 时的映射为 “192.168.0.1:8088 <--> 220.181.38.149:1111”，那么内网 A 访问外网 C 时的映射可能会变成 “192.168.0.1:8088 <--> 220.181.38.149:2222”；
    

为什么要叫锥形和对称形？

![](https://mmbiz.qpic.cn/mmbiz_jpg/ndgH50E7pIprdVsJwia60665StiaMyS6BTtXh9aicfPfqTh1o0IYxHvqOmDMsGNQ0Cibg8kdf7P3bAqd80fQ8JLyUQ/640?wx_fmt=jpeg)

ICE -- NAT “打洞”
===============

知道 NAT 的存在之后，再举一个例子🌰：用户 A 知道用户 B 的网络地址，并且 A 和 B 在不同的 NAT 之后；某一时刻 A 想主动联系 B，然后 A 经过自己 NAT 发一个请求给 B，请求到达 B 的 NAT 时，因为 B 没联系过 A，所以 B 的 NAT 便会将 A 的请求丢弃；

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIprdVsJwia60665StiaMyS6BTVjWOticJKYtMUibDLVBPU2fzvjQgLGodYTMaM9rbL646icMic5cRNibclHA/640?wx_fmt=png)

上面简单的例子就可以看出，虽然 RTC 是一个 P2P 的协议，但因为 NAT 墙的存在，就算通讯的双方知道对方的网络地址，也没办法直接沟通......

所以，需要引入一个机制对这个沟通过程进行协调，帮助通讯双方能够越过 NAT 并成功建立连接，这套机制就是 ICE（Interactive Connectivity Establishment），ICE 是一个框架协议，可以让互联网中两个设备进行点对点的连接，ICE 框架包含的两个主要工具协议：

*   STUN
    
*   TURN
    

STUN
----

STUN（Session Traversal Utilities for NAT）是一个工具协议，这个协议的主要目的是协调帮助两个在 NAT 之后的设备建立 UDP （也可以是 TCP）传输；既然 STUN 是一个协议，那我们就可以采用任意技术栈来开发实现一个 STUN 服务及 STUN 客户端，实现的 STUN 主要有两个作用：

1.  帮助获取客户端的公网地址，并通过复杂的策略，探测出客户端所处的 NAT 类型；
    

2.  STUN 还可以帮助两个客户端之间进行 NAT “打洞” 或者协调 TURN 在两个客户端中间充当中继服务；
    

### NAT 探测

服务端可以非常轻松的在数据包中获取请求的来源 IP 和端口，但是并没有办法知道对应的请求是客户端直发还是 NAT 转发的，更没办法知道是哪种类型的 NAT，客户端也一样无法知道自己的 NAT 情况；但只要 STUN 客户端及 STUN 服务齐心协力，就可以一步步探测出 NAT 情况；

STUN 服务和 STUN 客户端会按照下面的逻辑进行配合，一步步探测客户端所处的 NAT 情况；

> ps：一个 STUN 服务需要拥有两个 IP ，通过两个 IP 的服务互相配合来探测 NAT 的情况

1.  第一步，判断是否存在 NAT，客户端主动发一个请求到 STUN 服务的 “IP1 端口 1” 上，STUN 服务把收到的请求的 IP 和端口直接返回给客户端，客户端会将 STUN 服务返回的 IP 和端口跟自己的 IP 和端口进行比较，
    

1.  如果一致，则表明客户端处在公网中，或者说客户端没有在 NAT 之后；
    
2.  如果不一致，则表明客户端处在 NAT 之后，需要往下走继续探测 NAT 类型；
    

2.  第二步，判断 NAT 是不是 Full Cone NAT（完全锥形），客户端发送请求到 STUN 服务的 “IP1 端口 1”，STUN 服务收到请求之后用 “IP2 端口 2” 主动往客户端发送一个请求，
    

1.  如果客户端能够收到 STUN 服务 IP2 的请求，则表明 NAT 策略非常宽松来者不拒，是完全锥形；
    
2.  如果客户端没办法收到 STUN 服务 IP2 的请求，则数据包被 NAT 丢弃了，NAT 不是完全锥形，需要往下走继续探测 NAT 类型；
    

3.  第三步，判断 NAT 是不是 Symmetric NAT（对称形），客户端主动往 STUN 服务的 “IP2 端口 2” 发送请求，STUN 服务收到请求之后把请求的来源 IP 和端口直接返回给客户端，客户端用收到的 IP 和端口跟 “第一步” 中的 IP 和端口进行比较，
    

1.  如果两次收到的端口不一致，则表明 NAT 是对称形的；
    
2.  如果一致，则表明 NAT 不是对称形的，需要进一步探测 NAT 类型；
    

4.  第四步，判断 NAT 对端口的限制，客户端主动往 STUN 服务的 “IP2 端口 2” 发请求，要求 STUN 服务用 “IP2 端口 3” 往客户端发请求，
    

1.  如果客户端收到了 “IP2 端口 3” 的请求，则表明 NAT 没有对端口进行限制，是 Restricted Cone NAT（IP 限制锥形）；
    
2.  如果没收到请求，则表明 NAT 限制了端口，是 Port Restricted Cone NAT（端口限制锥形）；
    

### NAT “打洞”

经过上面四个步骤之后，便知道了客户端的公网地址以及所在的 NAT 情况，光知道 NAT 情况还没用，NAT 依旧会对请求进行拦截，STUN 还需要协调两个客户端对各自的 NAT 进行打洞，客户端才能穿越 NAT 完成连接建立，下面从简单到复杂举几个例子来说明 NAT 的打洞流程；

#### 只有一方在 NAT 后

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIprdVsJwia60665StiaMyS6BTNfRicTaCcrAcibicMmiaB1nPcCMoQibJlIInz0JJDfcp281rE7Ing4qJGIg/640?wx_fmt=png)

假设：客户端 A 和客户端 B 需要建立 P2P 连接，客户端 A 直接拥有一个公网 IP，而客户端 B 在 NAT 之后；

这种情况下如果客户端 A 直接与客户端 B 通信，通信将会失败，客户端 A 发送的数据包会被客户端 B 的 NAT 丢弃；此时，STUN 服务端便会进行协调，让客户端 B 主动先往客户端 A 发送数据包，客户端 B 的 NAT 便记录了客户端 A 的通信记录，客户端 A 后续便可以与客户端 B 进行通信了；

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIprdVsJwia60665StiaMyS6BTlZCY3dLh3DnWcic733SicLUiaDeSsialUPg7F5DIScbcPjHKRqJian3JCAQ/640?wx_fmt=png)

“客户端 B 主动连接客户端 A” 这个过程就被形象的称为 “给客户端 B 的 NAT 打洞”；

#### 双方都在非对称形 NAT 后

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIprdVsJwia60665StiaMyS6BT6EhMRpEzdBHQ6VQaiclyutozRELEf1rJRpsBW6JFEclVBiaE6VCOqqRg/640?wx_fmt=png)

假设：客户端 A 和客户端 B 需要建立 P2P 连接，客户端 A 和客户端 B 在不同的 NAT 之后；

这种情况下客户端 A 和客户端 B 往对方发送的数据都会被 NAT 丢弃，STUN 服务便会协调两个客户端，让它们先主动都往对方发送数据，在自己的 NAT 上留下对方的 “洞”，后续两个客户端便可以完成连接的建立了；

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIprdVsJwia60665StiaMyS6BT3ic0TQhjiaWRhMibVKB0SIlVNohwWU8opJl3gicDontkMCrjdicI2E5icibgg/640?wx_fmt=png)

#### 双方在对称形 NAT 后

假设：客户端 A 和 B 的 NAT 均为 Symmetric NAT（对称形）；

> 这种情况下，先说结论，STUN 服务将无法协调客户端 B 的 NAT 打洞；

由于对称形 NAT 的特性，STUN 服务端看到的客户端 A “ip 、 端口”，将会和客户端 B 看到的客户端 A 的 “ip、 端口” 不一样，此时如果 STUN 服务强行协调客户端 B 给 NAT 进行打洞，打的洞客户端 A 并没办法使用；

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIprdVsJwia60665StiaMyS6BTOtfJTMNaia72j4yDE5WtibWYgkJOrDS8q9pib2CPANhL3LTLOtdKa29dQ/640?wx_fmt=png)

所以这种情况下是没有办法建立 P2P 连接的，也因为这种情况的存在，才引入了 TURN 中继协议；

TURN
----

TURN 全称 “Traversal Using Relays around NAT（TURN）：Relay Extensions to Session Traversal Utilities for NAT（STUN）” ，从全称就可以看出，TURN 是 STUN 的一个补充协议，当 STUN 无法完成两个客户端的 P2P 直连时，TURN 便会充当一个 “中继服务器” 的角色，对两个客户端之间的信息进行转发；

TURN 服务功能示意图

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIprdVsJwia60665StiaMyS6BT8NrmKNx24IicDhaPdSgyWElEoG7jYLQZTllDzibMp7ibvm6GkibdfTpYww/640?wx_fmt=png)

如何快速判断是否能打洞？
------------

给 NAT 类型进行一个排序，从宽松到严格的顺序如下：

1.  Full Cone NAT（完全锥形）
    

2.  Restricted Cone NAT（IP 限制锥形）
    

3.  Port Restricted Cone NAT（端口限制锥形）
    

4.  Symmetric NAT（对称形）
    

如果两个客户端的 NAT 类型的序号相加大于等于 7 ，则无法打洞成功，需要引入 TURN 服务；举个例子，如果两个客户端分别是 “4. Symmetric NAT（对称形）” 和 “2. Restricted Cone NAT（IP 限制锥形）”，则这两个客户端能打洞成功，因为他们的序号相加为 6 ，小于 7；

SDP
---

SDP 全称 “Session Description Protocol” 会话描述协议，虽然名字上看起来是一个协议，但本质上只是一种数据格式，这种数据格式可以用来让 RTC 通信双方互相交换信息， SDP 数据格式如下图所示：

```
Session description（会话级别描述）     v= (protocol version)     o= (originator and session identifier)    s= (session name)     c=* (connection information -- not required if included in all media) One or more Time descriptions ("t=" and "r=" lines; see below)     a=* (zero or more session attribute lines) Zero or more Media descriptions     Time description     t= (time the session is active) Media description（媒体级别描述）, if present     m= (media name and transport address)     c=* (connection information -- optional if included at session level)     a=* (zero or more media attribute lines)
```

教室中一个实际的 SDP

```
v=0<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">o=- 6307461294211741498 2 IN IP4 127.0.0.1<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">s=-<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">t=0 0<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=group:BUNDLE 0 1<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=msid-semantic: WMS<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">m=audio 9 UDP/TLS/RTP/SAVPF 111 63 103 104 9 0 8 106 105 13 110 112 113 126<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">c=IN IP4 0.0.0.0<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=rtcp:9 IN IP4 0.0.0.0<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=ice-ufrag:USPp<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=ice-pwd:qDZxlC3pdC2JJLUhEGDTh71+<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=ice-options:trickle<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">m=video 9 UDP/TLS/RTP/SAVPF 96 97 98 99 100 101 122 102 121 127 120 125 107 108 109 124 119 123 117 35 36 114 115 116 62 118 37<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">c=IN IP4 0.0.0.0<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=rtcp:9 IN IP4 0.0.0.0<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=ice-ufrag:USPp<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=ice-pwd:qDZxlC3pdC2JJLUhEGDTh71+<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=ice-options:trickle<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=rtpmap:102 H264/90000<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=fingerprint:sha-256 AE:C2:C3:BC:8E:87:E1:32:3F:D4:C1:C4:B4:78:0A:10:03:E1:02:9D:2D:8C:8A:E5:DC:D0:60:A4:0A:3B:C6:C3<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=setup:actpass<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=mid:1<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=extmap:3 urn:ietf:params:rtp-hdrext:toffset<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">a=extmap:4 urn:3gpp:video-orientation<br data-darkmode-color-16527997830401="rgb(171, 178, 191)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)|rgb(171, 178, 191)" data-darkmode-bgcolor-16527997830401="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(40, 44, 52)">
```

链接建立完整流程
--------

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIprdVsJwia60665StiaMyS6BTHibh1yNJJKEibPT6qDrEHc1AXUJZIP6JQzmHibD6w1ibhVaWVYHBFNOuMA/640?wx_fmt=png)

RTP
===

经过面的流程，我们已经把 “传输层” 打通，最常见的传输层协议有 TCP 和 UDP ，应该用 TCP 还是 UDP 来传输音视频呢？

<table data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)"><thead data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)"><tr data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">UDP</th><th data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">TCP</th></tr></thead><tbody data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)"><tr data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">无连接，传输数据前无需建立连接，双方可以随时发送数据；</td><td data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">面向连接，数据传输前需要经过三次握手建立一个连接；</td></tr><tr data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">面向报文，UDP 只会对应用层传过来的数据增加或拆除 UDP 报文头；</td><td data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">面向字节流，应用层数据对 TCP 来说只是一串没结构的字节流；</td></tr><tr data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">不可靠传输，出现丢包的情况，发送方并不会做特殊处理；</td><td data-darkmode-color-16527997830401="rgb(163, 163, 163)" data-darkmode-original-color-16527997830401="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16527997830401="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16527997830401="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;" class="">可靠传输，存在丢包重试的机制，保证传输过程中接收方能够收到对应的数据包；</td></tr></tbody></table>

总的来说，UDP 效率更高，TCP 更稳定，对于即时通讯来说，音视频丢失一两帧的影响是可以接受的，但是高延迟会带来非常差的用户体验，所以大部分实时音视频的传输都会选择 UDP 作为传输层协议；

视频一帧的数据需要拆成多个 UDP 数据包进行传递，UDP 并不具备视频帧拆分及组装的能力，所以还需要在其之上构建一个应用层协议来完成音视频传输；

RTP （Real-time Transport Protocol）是一个基于 UDP 的应用层协议，包含两个子协议：

*   RTP：数据传输协议，主要负责视频帧拆分及组装；
    
*   RTCP：控制协议，QoS 反馈相关，拥塞控制；
    

总结
==

RTC 虽然是一个 P2P 的协议，但是由于 NAT 的存在，需要引入 ICE 协议进行网络穿透，才能在两个设备之间建立 UDP 连接，连接建立完成之后，会采用 RTP 作为应用层协议保证音视频的正常传输。

- END -