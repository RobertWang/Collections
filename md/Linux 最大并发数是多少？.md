> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651167793&idx=1&sn=f44c8c206391dfbe61f4e19a1f249ab9&chksm=80644d6eb713c478d23a4b166903c9dd33ed6dfadc0d73b783b1b24eb638c0c0346161d287c7&scene=21#wechat_redirect)

1. 开场白
------

在开始今天的文章之前，先抛一个面试题出来：

> 你接触过的单机最大并发数是多少？  
> 你认为当前正常配置的服务器物理机最大并发数可以到多少？  
> 说说你的理解和分析。

思考几分钟，如果你可以**有理有据地说出答案**，那确实就不用再往下看了，关上手机去陪陪家人是个不错的选择。

思考几分钟，如果你**没有头绪或者对答案不确定**，那么你先不用着急关闭页面去玩耍，你应该继续往下看，因为这个问题很不错。

对于后端开发人员来说，并发数往往和技术难度是呈正相关的，实际上也确实如此：**体量决定架构**。

服务端根据不同业务场景会有不同的侧重点，单纯追求高并发其实并不是根本目的，**高可用 & 稳定性更重要**。

所以最终我们的目的是：**保证高可用高稳定的基础上追求高并发，降本增效**。

高可用 & 高并发是我们直观感受到的，本质上这是个**复杂的系统工程**，每个环节都会影响结果，每一块都值得研究和深入。

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7ic9xdUEcyPezq66oDWiaHc0Dewu9A48duaFMn2rt84n7PmWKg92ibcx7icA/640?wx_fmt=png)

2. C10K 问题和 C10M 问题
-------------------

在 2000 年初的时候，全球互联网的规模并不大，但是当时就已经提出了 C10K 问题，所谓 **C10K 就是单机 1w 并发问题**，虽然现在不觉得是个难题了，但是这在当初是很有远见和挑战的问题。

C10K 问题最早由 **Dan Kegel** 发布于其个人站点，原文链接如下:

> **http://www.kegel.com/c10k.html**

相关资料显示 Dan Kegel 目前工作于 **Google**，从 1978 年起开始接触计算机编程，是 Winetricks 和 Crosstool 的作者，大佬年轻时的照片：

Dan Kegel 这篇文章阅读难度并不大，大白建议从事服务端开发或者对高性能网络开发有兴趣的读者尝试读一读。  

在 APUE 第三版都没有提到 epoll，所以**我们解决 C10K 问题的时间并不长**，其中 IO 复用 epoll/kqueue/iocp 等技术对于 C10k 问题的解决起到了非常重要的作用。

开源大神们基于 epoll/kqueue 等开发了诸如 libevent/libuv 等网络库，从而大幅提高了高并发网络的开发效率，对于 C/C++ 程序员来说并不陌生。

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7icwrVRF3kSwvMdsge6KrdiaEQGVibzic7uBOZw851iclbQ2cm1tXssWpwuQw/640?wx_fmt=png)

这里简单提一下针对下一个 10 年的展望和挑战：**C10M 问题**。

站在浪尖的那一批人早就开始思考**让单机达到 1000w 并发**，现在听起来感觉不可思议，但是要达到这个目标，除了硬件上的提升，更重要的是**对系统软件和协议栈的改造**。

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7icQlvML1DLvvwrULdzdjUqkpDwmWhmmKZNJ4picZX3mv5Mx06tt2zaU8Q/640?wx_fmt=png)

Errata Security 的 CEO Robert Graham 在 Shmoocon 2013 大会上的演讲，大佬重要的观点是：

> **不要让 OS 内核执行所有繁重的任务：将数据包处理、内存管理、处理器调度等任务从内核转移到应用程序高效地完成，让诸如 Linux 这样的 OS 只处理控制层，数据层完全交给应用程序来处理。**

确实也是如此，**难道你不觉得 Linux 内核做了太多不该自己做的事情了吗**？

近几年出现的 DPDK、PFRING、NETMAP 等技术也是类似的思想，现在流行的协处理器 + CPU 的架构也是这样的：

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7icL04E6OJbCnpnibQ3BTYEpNs46Q3kpcSfMqK13x4duN69oR9gn2GV4Uw/640?wx_fmt=png)

3. 服务器最大并发数分析
-------------

前面提到的 C10K 和 C10M 问题都是围绕着提升服务器并发能力展开的，但是难免要问：**服务器最大的并发上限是多少**？

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7icib5m03SiayEibcsHCaYnVlb03af2JmDdEeGHA9JfEYHaxk4z3ARuxhNQA/640?wx_fmt=png)

### 3.1 五元组

做过通信的盆友们一定听过**五元组**这个概念，一个五元组可以唯一标记一个网络连接，所以要理解和分析最大并发数，就必须理解五元组：

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7icnBd55Fem053fREQZ1wOzr4wviaBYrs3sjxtiaOd7WznhA7JnjZ1RSGkA/640?wx_fmt=png)

这样的话，就可以基本认为：**理论最大并发数 = 服务端唯一五元组数**。

### 3.2 端口 & IP 组合数

那么对于服务器来说，服务端唯一五元组数最大是多少呢？

**有人说是 65535**，显然不是，但是之所以会有这类答案是因为当前 Linux 的端口号是 2 字节大小的 short 类型，总计 2^16 个端口，除去一些系统占用的端口，可用端口确实只剩下 64000 多了。

对于服务端本身来说，DestPort 数量确实有限，假定有多张网卡，每个网卡绑定多个 IP，**服务端的 Port 端口数和 IP 数的组合类型也是有限的**。

对于客户端来说，本身的端口和 IP 也是一样有限的，虽然这是个**组合问题**，但是数量还是有限的：

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7icNcicTDq68qlIoeZu5cINXz3EQIHGKUy4FaOVKUpC2UqdJAPeFVFInxw/640?wx_fmt=png)

### 3.3 并发数理论极限

看了前面的端口 & IP 的组合数计算，好像并发数并不会特别大。

**错了，是真的会很大。**

分析一下，前面的计算都是针对单个服务器或者客户端的，但是**实际上每个服务器会应对全网的所有客户端**，那么从服务端看，源 IP 和源 Port 的数量是非常大的。

**理论上服务端可以接受的客户端 IP 是 2^32(按照 IPv4 计算）, 端口数是 2^16，目前端口号仍然是 16bit 的，所有这个理论最大值是 2^48**，果然很大！

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7icxJDaILeLKoAdBa4t7AK4w5h0eF9OITKyrZFnDRtLRkibKBUGIcPw4tA/640?wx_fmt=png)

### 3.4 实际情况

天下没有免费的午餐。

**每一条连接都是要消耗系统资源的**，所以实际中可能会设置最大并发数来保证服务器的安全和稳定，所以**这个理论最大并发数是不可能达到的**。

**实际中并发数和业务是直接相关的**，像 Redis 这种内存型的服务端并发十几万都是没问题的，大部分来讲几十 / 几百 / 几千 / 几万等是存在的。  

4. 客户端最大连接数  

--------------

理解了服务器的最大并发数是 2^48，那么**客户端最多可以连接多少服务器呢**？

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7icrGoKbWnGGN6z6IavlPJMcMjzgWiaCJ3LTs1vz1ia1icT9zWjTJia1rhIFw/640?wx_fmt=png)

对于客户端来说，当然可以借助于多网卡多 IP 来增加连接能力，我们仍然假定客户端只有 1 张网卡 1 个 IP，由于端口数的限制到 2^16，再去掉系统占用的端口，剩下可用的差不多 64000。

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7icVMZ1MToDCeG3icPopDLezsuqkNuQicEewTqPk4dA7HTNmAEcGJiathRDw/640?wx_fmt=png)

也就是说，客户端虽然可以连接任意的目的 IP 和目的端口，但是客户端自身端口是有限的，所以**客户端的理论最大连接数是 2^16**，含系统占用端口。

5. NAT 环境下的客户端
--------------

> **一个公网出口 NAT 服务设备最多可同时支持多少内网 IP 并发访问外网服务？**

![](https://mmbiz.qpic.cn/mmbiz_png/wAkAIFs11qZibu6QZauU9zUcre7RvLf7ic4vCCiaibz3fagWs7UsWxQWUjfc8HTb6jsSe00icoka7WPZAkeLM9au4lA/640?wx_fmt=png)

6. 小结
-----

虽然理论服务端并发数非常大，但是我们也没有必要觉得并发数高就厉害，服务复杂程度不一样，**切忌唯并发数来判断业务和开发者水平**。

试想 echo 服务和订单交易服务显然是不一样的，我们**应该做的是在服务稳定和高可用的前提下去从缓存 / 网络 / 数据库等多个角度来优化提高性能**。  

最后感谢各位老铁的倾情阅读。

- EOF -

推荐阅读  点击标题可跳转

1、[这篇 CPU Cache，估计也没人看](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651167790&idx=1&sn=457108d59a01c1683f667f5ab793e2db&chksm=80644d71b713c46708a05930b72259b679afdfc537b675421f48b98a58e0acf3e02625c87cf9&scene=21#wechat_redirect)

2、[深入理解 Linux 内存子系统](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651167786&idx=1&sn=afb37aa02790fa95de818fd2875f7e0c&chksm=80644d75b713c46381cdc5151c3253b0eda60381be02b0b52edc24c1afd9dfb76581c03573ed&scene=21#wechat_redirect)

3、[研究了一波 Android Native C++ 内存泄漏的调试](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651167785&idx=2&sn=1f9c30e3c0689456e760cf06f788120c&chksm=80644d76b713c460316cfc1463065c3f17568d67e8ae4dc60c142b84114ad8bce8ba8ecb595a&scene=21#wechat_redirect)