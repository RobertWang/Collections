> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IGTirxb6Cg7c_uMrHyvhHw)

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7xzPgmOMoEibOVibT9e9HEbia6xkialhslY4xroxSRUj7lFrOWPLbuxHjUA/640?wx_fmt=png)

### 什么是 IM

> IM（Instant Messaging）即时通信，是一种通过网络进行实时通信的系统，允许两人或多人使用网络即时的传递文字消息、文件、语音与视频交流，通常以网站、软件或者移动 app 的方式提供服务。自从互联网的兴起，IM 就一直和我们的生活息息相关，日常聊天、工作、打车、外卖、购物等等，可以说，我们现在的生活已经几乎离不开 IM。

### IM 历史

最早人们的通信靠的是邮件，需要人去邮局寄信，然后邮递员再经过漫长的旅程送达对方。从前车马很慢，一生只够爱一个人，咳咳。从邮件到传呼机再到有线电话，无线电话，最后随着互联网的发展 IM 迎来了它的新生。

最早的即时通信软件叫做 ICQ，他是四名以色列青年于 1996 年 7 月成立的 Mirabilis 公司推出的产品。然后腾讯接着推出了 OICQ。

![](https://mmbiz.qpic.cn/mmbiz_jpg/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7YkvGPK74ytU3zhIE4spsWGfKEj8QJWyoMtGNOXmEicoQB0jpqNlXWCQ/640?wx_fmt=jpeg)

图片取自网络

以及现在主流的聊天应用

*   Whatsapp 美国
    
*   Line 日本
    
*   Kakao Talk 韩国
    
*   WeChat 中国
    
*   Facebook Messenger 美国
    

### IM 特性

IM 的四大特性，有效性、实时性、一致性以及安全性，这四点可以总结为两个字，`可靠`，那么如何实现一个可靠的 Web IM 应用呢？业界其实已有很成熟的 IM 的方案，我们会从通信协议、应用层、通信数据格式、以及相应策略方面给大家阐述一个可靠的 IM 应用都需要哪些东西。

### IM 通信协议

为了保证可靠，传输层我们一般使用 TCP 协议，它是面向连接，可靠的流协议，实行 “顺序控制” 和“重发机制”，还有“流（流量）控制”、“拥塞控制”、提高网络利用率等众多功能。

在 PC 的早期时代，IM 采用的是 http 短轮询的模式（图 1），它会定期、高频地轮询服务器端消息。

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7pke9hLXBiam9g4cBXVe3ibXxj7buQWfmG9MYibZ3F96vjot5AEUia2VbEw/640?wx_fmt=png)

图 1

它的缺点也很明显，会有大量无用的请求，用户端也会非常耗电耗流量，而服务端面对高频 QPS，内存资源压力也会非常大

对于短轮询的优化，就出现了长轮询（图 2）。相对短轮询，它大幅降低了无用轮询导致的网络与功耗开销，但是服务端悬挂住请求，只是降低了入口请求的 QPS，并没有降低服务器的资源开销，假如有 1000 个请求在等待，那就意味着有 1000 个线程挂起，被轮询占用消息存储资源。

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7Ts5LeOP6AWrftiabsyFDt1J42TpUEjJgot7PtQmFdWk20KheJklcaoQ/640?wx_fmt=png)

图 2

为了更好的解决实时性问题，IM 领域经历过几次技术的迭代升级，从简单、低效的短轮询逐步升级到相对效率可控的长轮询，然后随着 h5 的出现，`全双工（Full-duplex）`的 websocket（图 3）彻底解决了服务端推送的问题。用户侧和服务端利用 websocket 建立长连接后，双方就可以同时进行双向的数据传输了。

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7PLPibxdDibp0ibJPldNuT1Laf8z0sibSibgIbGm50icMzIhfdQTibnvUe4nyw/640?wx_fmt=png)

图 3

服务器的压力也不再是连接数，而是每一条消息事物。

### 应用层可靠

在底层协议的保障后，我们的消息就完全可靠了吗？那肯定不是，我们的服务大致是这样的，用户发送消息给服务端，服务端存储消息，再返回给用户，并把该条消息推给另一个用户，在下图（图 4）流程中，每个环节都可能存在消息丢失的风险。

*   用户 1 发送到 IM 服务的过程中
    
*   IM 服务器存储失败
    
*   用户 1 等待服务器响应超时
    
*   服务器往用户 2 推送消息时超时、错误
    

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX76RT7fIPkVA25OiaZjX1H8e2K1ZG2CevbjsPuib4F8egKcdQxicyqzuibxQ/640?wx_fmt=png)

图 4

#### ack 机制

为了解决用户 1 到服务端的可靠性问题，我们参考 TCP 协议的握手、重传机制，来保障应用层消息的可靠性，在发送消息后会有一个定时超时，在超时后根据需要，从 ack 队列中取出消息重推（图 5）。一般情况显示发送失败，交由用户手动重发（比如消息左边一个红色感叹号）。

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7uibF3L4rxHYRWYGuYgdp8szrsjcp9qsgDBfxvJkKFfwpklIiag3azrHg/640?wx_fmt=png)

图 5

用户发送消息，服务端收到消息后，生成该条消息的唯一 id，以 ack 的形式回传给用户侧，用户侧再更新该条消息 id 值，后续 IM 功能中的撤回，去重 ，重发等逻辑都会用到该 id。

#### 重发与去重机制

在服务端推送消息时，如出错或超时，会有相应的重发机制。比如，设置错误或超时重试三次。

有时因为一些网络或其他情况。服务端会有相应的重发逻辑，在推送消息出现重发时，用户端设置对应的去重逻辑。我们会对消息列表的最新的 5 条消息进行排序和去重。只取最新 5 条主要考虑到排序与去重的效率，用户的焦点主要在最新的几条消息，如果因为一些网络原因在消息列表较远处插入消息，会造成用户的困惑与遗漏，另外 5 条消息的时间差基本满足大多数异常情况的消息丢失场景。如果还有消息遗漏的情况，用户在刷新消息列表时会以 http 的形式拉取历史消息（当前会话的消息）。

#### 断线重连（Qos 机制）

websocket 有 error 和 close 事件，我们在监听这两个事件后进行相应的重连逻辑，其中在 close 事件里不对状态码 1000（正常关闭）做重连处理。

具体逻辑如下

首先我们有个重连锁，在正在进行重连时不重复触发重连逻辑，在保证 ws 完全关闭的情况下，会以重连次数的二次幂作为重连的时间间隔，并且在重试时间达到 30s 后不再递增。这样处理的逻辑一是为了保证在断线或者异常时能马上进行重连的尝试，但是会逐渐减缓重连尝试，假如是服务器负载等问题造成的断开，也避免一直频繁连接给服务器造成压力。

#### 消息就不会丢了吗

我们的 ack + 超时重传 + 消息去重，能解决大部分消息推送丢失的问题，但比如服务器宕机，电脑手机息屏，手机切换后台等等造成连接断开，通道不可用（图 6）。服务端在这个期间推送消息，那如何保证用户能收到完整的消息？

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7ic8vvYkkZBicC7iccGQoZ0bWY2jFcCNWqcuNnFPjOsibzFtgIibwAdZDzAQ/640?wx_fmt=png)

图 6

一般在这个时候，我们会在重连或者用户窗口可视时对消息进行完整性检查，会以 http 的形式拉取这段时间的消息，以最后一条消息的时间戳作为参数拉取这个时间段的消息，或者拉取后端会话（session）维度的消息。

#### 心跳机制

websocket 的连接是无感知的虚拟连接，中间链路出现一些异常情况断开时两边不会感知到，为了保证服务的可靠性，以及降低服务器的开销，我们会有对应的心跳机制（图 7），来检测连接是否正常，从而保持连接高可用。

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7HSnhiawc4RBumaWGlqZTSXbwFwXfxNliahe65ekaIyzPtZbKEU9bsMIA/640?wx_fmt=png)

图 7

在心跳的基础上，能及时支持客户端的心跳断线重连，比如两次心跳没有 ack，或者心跳超时没有收到 ack（图 8）。

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7hb1ibvLCYUGyhvoCPq9smAoMnNShxx6rZ3Xwx0KmxsRRynciaGWtAfqQ/640?wx_fmt=png)

图 8

心跳除了用于重连，还可用于及时释放服务器以及业务资源，取决于 IM 的场景与策略。比如一些客服聊天场景，客服要尽可能接待更多的用户，为及时释放客服资源，服务端在用户达到固定未收到心跳时间，及时断开客服聊天，释放相应资源。

除此之外，心跳还有连接保活的功能。有时会遇到 NAT(Network Address Translator) 超时的情况。运营商维护 NAT 映射表时，为了节约资源和降低 自身网关压力，会定时清除没有数据收发的连接，具体不在这里详情阐述。但这个过程服务端和用户端都无法感知，从而会影响消息收发。下图（图 9）是一些运营商的 NAT 超时时间

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7ic8nlfX0AHjYo8pDR5rZH7DABMkEZhsWe3S4jS1hqrRzAiaDQibNfbUrg/640?wx_fmt=png)

图 9

#### 常用心跳方案

*   TCP keepalive
    
*   应用层心跳
    
*   智能心跳
    

TCP 的 keepalive 作为系统层 TCP/IP 协议的已有实现，操作系统默认是关闭的，需要应用层开启，默认配置项周期是 2 小时，失败后重试 9 次，超时 75s，但是灵活性较差，所以我们一般不采用。应用层心跳能灵活控制，更能结合业务，具体策略如下图（图 10）

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7s5xRlpPphSttqD1vTROWbSbgOWxSWBu1iclnHzgDxpCGDJw7P0etpQg/640?wx_fmt=png)

图 10

心跳的发送间隔，最简单的就是采用固定心跳时间，另外由于 NAT 超时时间以及网络环境切换的不确定性，会有一些智能心跳方案，这里分享一下安卓版微信的智能心跳方案。

*   [MinHeat, MaxHeart]-- 心跳可选区间
    
*   successHeart-- 当前成功心跳
    
*   curHeart-- 当前心跳，初始值 successHeart
    
*   heartStep—心跳增加步长
    
*   successStep—稳定期后的探测步长
    

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7gspP14ia04zXLEjT6iaGIJ7H1A5iavZP2PjSntzkS0B3BibkKfp3bUT9eQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX760AohycV0QXpDiaDictg2XNUey5GPm02bT3HtbTUhspKztKFuKvWKzOw/640?wx_fmt=png)

### 消息协议

好，通信上已经基本没有问题了，有了上述的策略后，IM 的消息通信就基本能满足大多数场景了，现在是消息协议的选型，也就是通信的数据格式，我们需要考虑的点有以下。

*   网络数据大小：占用带宽，传输效率
    
*   网络数据安全性：敏感数据的网络安全
    
*   编码复杂度
    
*   协议通用性、大众规范
    

业界常用的数据格式有以下

*   XMPP
    
*   Protobuf（Protocol Buffer）
    
*   JSON
    
*   私有二进制
    
*   MQTT
    
*   定制化 XML
    

一般我们会选择 Protobuf，它是 Google 公司内部的混合语言数据标准，用 pb 序列化后的大小是 json 的 10 分之一，xml 格式的 20 分之一，是二进制序列化的 10 分之一，并且基本上主流语言都已支持。不过考虑到上手的简单以及易调试，json 也不是一个很坏的选择，毕竟现在 http 的数据基本都是 json，明文的数据包在调试上也会方便许多。

### 创建 websocekt 实例

websocket 现在主流浏览器早已支持很久，并且在移动端也基本没有兼容性问题，官方提供的 API 十分简单，具体不在这里阐述，基本就四个事件，我们上述说到的功能都可以基于这四个事件去做。

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNNib3nPw9vVjTLX3XiakrZoDX7lpqY2bCXgYuic9YV5K6AI6ibOeZPaqDvJIsXLic75PCTGGITcsOzXWcuw/640?wx_fmt=png)

### 展望

IM 需要考虑的还有很多，不仅是前端，还有更多的是与服务端配合的策略。其他的还有一些 IM 里面常用功能，比如撤回、已读未读，以及自定义消息类型，多媒体消息之类的实现，ws 降级，群组消息推送等问题，受篇幅限制，这里不再仔细展开。

升华一下，人类社会发展，需要协作产出，需要沟通交流。随着 5g 的发展以及普及，即时通讯必然会往更广的方向延伸，并且并不局限于简单的聊天。