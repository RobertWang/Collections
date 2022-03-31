> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/QuB0YL2kozbKdLWD9223WQ)

重磅干货，第一时间送达![](https://mmbiz.qpic.cn/mmbiz_jpg/ow6przZuPIENb0m5iawutIf90N2Ub3dcPuP2KXHJvaR1Fv2FnicTuOy3KcHuIEJbd9lUyOibeXqW8tEhoJGL98qOw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaNichPyr3KiadrVGic4CcdQwELhQrACyXZnYiaPSt876uvoomuYNrSunqoA/640?wx_fmt=png)

提纲

_1 Chrome 浏览器原理_
----------------

> 还记得面试过程中被问了千百遍的 "输入 URL 后发生了什么" 这个经典问题吗？因为这个问题覆盖了太多的知识点，其中包括计算机网络，操作系统，数据结构等一些列问题，对于面试官和面试者来说都更方便后续面试的进展。想必很多小伙伴都做过 web 开发，或多或少都会和各种浏览器联系在一起，最终做测试的时候也会使用多种浏览器测试以保证能很好地兼容。那么现在我们先从 Chrome 浏览器说起。

我们先想想一个问题，我们打开一个微信或者一个 XX 音乐，一个网页，到底会开几个进程。

我们实验看看，打开一个网页到底开了几个进程，又分别有什么作用

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SatJvWDHlx1MUAyn1ma5qvFFKv7s7dxkcZyIwGXzh2IzTBoEbebxD23g/640?wx_fmt=png)

打开浏览器使用的 进程数

从上图我们发现，打开一个网页，使用了四个进程，分别为 GPU 进程，Network Service 进程，当前网页进程和浏览器。到此，我们先复习进程与线程。

假设现在有这样几行伪代码，我们看看应该怎么去执行，可能分为四步

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaSPOjFhibfBbUNZAV54U2qjctvLPZRefs0W5qRqCB0BBK24TpBmibpfjg/640?wx_fmt=png)

示例伪代码

*   计算 X=5+2
    
*   计算 y=8/4
    
*   计算 z=2*5
    
*   显示出最后的结果
    

这也是采用串行的方式运行，也可说为单线程方式执行了四个任务，其好处是不用考虑诸如多线程的同步等问题。但是如果采用多线程

*   启动三个线程分别处理前面三个任务
    
*   最后一个线程显示结果
    

> 从上面这个小实验，我们可以知道使用多线程只需要两步就完成，但是单线程却使用了四步，可知使用多线程大大的提升了性能，记住：并不是多线程就一定会比单线程好，还需要从 CPU 使用率，IO 磁盘等多个因素考虑。

_进程_

> 进程是一个程序的运行实体，在上面我们比较直观的感受到了多线程并行处理提高性能的优点。一个进程可以包含多个线程，但是一个线程只能归属于一个进程，那么一个进程到底是什么样子呢 (ps 下面是在 Linux 中执行的代码，道理差不多)

_创建进程_

> 在 Linux 中使用 fork 创建进程，返回进程 id。通过 id 的不同让父子进程各干其事，然后使用 execvp 执行具体任务

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SapBVZXglUJKbg8WB7pmjoZ2W2fxicezO3U5Yg7AiazxzGwwMpSyYialAlA/640?wx_fmt=png)

创建进程

> 创建主函数来使用上面的函数，看看会出现什么情况

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaG7K0E1uicTtV3vgVDawzJPzmTKvpiaAia5XKU5wjQz3fA1YOiaCBb5iaueg/640?wx_fmt=png)

主函数

好了，现在主函数和执行函数都写完了，但是这还只是文本文件，对于计算机而言只喜欢 "01" 组合，cpu 执行的命令需要是二进制，所以需要进行「编译」，但是二进制的组合也得有一定的格式，不然定会乱套，在 Linux 中这种格式是 "ELF"Executeableand Linkable Format），后续会详细介绍。程序编译到进程的过程如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Sa8MQZuGyq7cwhD4CGQQgibIPGx2oabkqx7PVlMRCnTRhSIUfINxax87g/640?wx_fmt=png)

文本文件到二进制  

现在带参数编译两个程序

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaUO7bpNZgZVP4RYw6iadGb7R1nyPa0W4oYbTCIOAUmnTRK70EZMKiazXw/640?wx_fmt=png)

编译

> 在编译的过程中，第一步预处理，将头文件直接嵌入到文件正文中，将定义的相关宏展开，最终编译为. o 文件 (可重定文件)，那么 ELF 是什么样子呢

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaUdxfmhyI9JgDbr2WAKMDwIGmV1QQUkaQoI9fjh3YGGCyicDTFnu8ELg/640?wx_fmt=png)

ELF 头部

上图给大家准备了几个_高频面试题目_ (哪些在代码段，数据段。。)

那么在 Linux 中如何查看呢 (readelf)

可重定位什么意思呢？

> 字面意思是可以随时放在其他位置。对的，目前我们只是编译了文件，将来会被加载到内存里面，也就是加在某一个位置。可惜现在还是. o 文件 (代码片段)，不具备可执行的权限，它以后想变为函数库，哪里需要就在哪里去完成任务，搬到了哪里就重新定位了位置。要让它可重用，就得成为库文件，这个文件分为静态链接库 (.a) 和动态链接库，它能将一系列的. o 文件归档为文件。怎么创建呢

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SarLwPbww800WCWnzUQjXiaiaUtZkVLbAiaGzEOmpaCkpial8qibjtJkHpJBw/640?wx_fmt=png)

ar

这个时候其他开发人员准备使用这个功能，加上参数连接过去就好了

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SamTfTTKXNGf931XiasichjdzFXIEzUdEY34GtXFjFQIicjIROnNDbRforQ/640?wx_fmt=png)

> 上面命令中 "-L" 代表默认在当前目录寻找. a 文件，然后取出. o 文件和 creteprocess.o 做连接形成二进制执行文件 staticcreateprocess。

一旦静态链接库连接出去，它的代码和变量的 section 合并，一次程序运行不再依赖这个库。这就可能出问题了，如果同样的代码段被多个程序使用，就会导致在内存中出现多份的情况，而且代码一旦更新，二进制文件也需要重新编译才能及时的更新。所以出现了_动态链接库__，_使用这种方式的时候，程序并不在一开始就完成动态链接，而是需要到真正调用动态代码时，载入程序才会计算动态代码的逻辑地址。这种方式让程序初始化时间短，但是运行期间性能比不上静态连接的程序

说的有点远了，回来回来。刚才我们说了多线程并行计算的优势，画个图对比加深印象下

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Sa0RSIYwF0nh0HqKcbgwDAfU73s0CGL38kH4eLeFamZl6oWFTbSNVicMQ/640?wx_fmt=png)

单线程与多线程

ok 总结下进程线程有哪些特点 (面试跑不脱)

*   进程中的任意一个线程出错，将导致整个进程崩溃
    

> 假设将之前的伪代码修改为  
> X=5+2  
> Y=8/0  
> Z=5*2

此时 Y 很明显就是错的，当线程执行到 Y 的时候就会报错，进程崩溃大致其他两个线程也没有结果

*   当一个进程关闭后，操作系统会回收进程占用的资源
    

> 比如我们会使用很多不错的 Chrome 插件，当启动浏览器并打开这些插件的时候，都会占用内存，当关闭进程 Chrome 浏览器，这些内存就会被收回

*   进程之间内容相互隔离
    

> 这个机制是防止多个进程读写混乱，所以进程之间通信需要 IPC(消息队列，共享内存等)。

*   线程之间共享进程数据
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaGP9zD2y4n0BNtoqUPDAuTibuLyUMW0I82Lz313AHw2qcSqXgf8NPHXg/640?wx_fmt=png)

线程共享数据

从上图我们可以知道线程 1，2 和线程 3 分别将数据写入 ABC，线程 2 在负责处理 ABC 三种读取数据并显示

现在我们基本上了解了线程和进程。我们想象一下，某宝级别的系统架构一开始就能抵抗这么大的流量吗，当然不是，最开始小黄页的单体架构，随着需求的复杂和多样化主键演变而来。那么浏览器依然如此。我们看看最开始的 Chrome 单进程样子。

最初的浏览器单进程，意味着无论是网络，页面渲染引擎还是 js 环境都在一个进程中，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SayAhTpax6q9f5Xae6HuSd8Cw7clfOCyheTh1MjMAeOa9ichZFRVYHwng/640?wx_fmt=png)

浏览器单进程

那个时候单体结构都有什么问题？

*   不稳定 / 不流畅
    

> 以前页面中的视频等元素需要使用插件才能观看，插件在页面进程中，插件出问题很容易导致浏览器崩溃。页面中如此多模块都运行在该线程中，一旦其中一个模块独占线程，其他的就只能当观众 (ps 能不能完成了就走，别蹲着不 X)，所以也就出现卡顿现象

*   安全性很难保障
    

> 当时很多插件能够比较轻松的拿到操作系统的 shell，如果是页面脚本，可以通过浏览器爆出的漏铜来到 shell，拿了 shell 就无法想象能干啥了

_如何解决上述问题_

*   不稳定和不流畅
    

> 原因是页面模块都在一个进程，采用进程分离，这样即使某个插件崩溃也只是影响某一部分，不会导致整个浏览器挂。

*   安全性问题
    

> 使用一个箱子 (安全沙箱)，箱子里面程序可以运行且把箱子上锁，但是无法读取外部任何程序。这样的话，我把容易出错且关键的两个进程插件进程与渲染进程装进去，这样的话，即使两者之一被执行恶意程序也只是在这个箱子里瞎摆弄，无法翻越出去拿到更高的权限干坏事。

_当前架构_

> 我们最初的时候，发现使用 chrome 浏览器打开一个网页的有四个进程，下面我来看看这些都有什么功能

一共是四个进程，分别为网络进程，GPU 进程，渲染进程和浏览器主进程。

网络进程

> 作为一个单独进程，负责页面网络资源的加载。

> 由于插件容易崩溃，单独进程对其进行管理

> Chrome 中 UI 界面绘制和 3DCSS 等需要 GPU 计算密集性的帮助，从而引入 GPU 进程

> 浏览器进程负责用户交互，各个子进程等功能

乍一看全是优点，通常事物都会有两面性，进程多了，开销当然也大也就是更高的资源占用和更加复杂的体系结构。

_2 DNS 简介_
----------

> 上面之所以介绍浏览器，因为 DNS 很多时候是我们在浏览器敲下回车时开始兴奋，这也是为什么从浏览器说起的原因。现在我们看看 DNS 到底是个啥玩意

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Saep8aMM29aianLaCdw62HAZ4DcgfD8iaqn66YLGlJKyKZcVicfiaqx5JFZw/640?wx_fmt=png)

_3 DNS 报文结构_
------------

> 说了这么多，协议头部，到底有哪些字段，其含义是什么都还不知道，那怎么去分析报文，下面我们一起再看看报文什么样子

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Sa7J4wwHfSueeVSa5ibYI1rfSZNCWxVJwm62s8BAgUAa2gdibXb1CCsc3A/640?wx_fmt=png)

DNS 报文结构

> DNS 报文基础部分为 DNS 首部。其中包含了事务 ID，标志，问题计数，回答资源计数，回答计数，权威名称服务器计数和附加资源记录数。

*   事务 ID: 报文标识，用来区分 DNS 应答报文是对哪个请求进行响应
    
*   标志:DNS 报文中标志字段
    
*   问题计数：DNS 查询请求了多少次
    
*   回答资源记录数：DNS 响应了多少次
    
*   权威名称服务器计数: 权威名称服务器数目
    
*   附加资源记录数: 权威名称服务器对应 IP 地址的数目
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Sa7tLn1pqM8MFCrMsddRRR6czXzcJURLr31d9ratPMEEc4BPiaRrSAPTw/640?wx_fmt=png)

标志字段

*   QR(Response)：查询请求，值为 0；响应为 1
    
*   Opcode: 操作码。0 表示标准查询；1 表示反向查询；2 服务器状态请求
    
*   AA（Authoritative）：授权应答，该字段在响应报文中有效。通过 0,1 区分是否为权威服务器。如果值为 1 时，表示名称服务器是权威服务器；值为 0 时，表示不是权威服务器。
    
*   TC（Truncated）：表示是否被截断。当值为 1 的时候时，说明响应超过了 512 字节并已被截断，此时只返回前 512 个字节。
    
*   RD（Recursion Desired）：期望递归。该字段能在一个查询中设置，并在响应中返回。该标志告诉名称服务器必须处理这个查询，这种方式被称为一个递归查询。如果该位为 0，且被请求的名称服务器没有一个授权回答，它将返回一个能解答该查询的其他名称服务器列表。这种方式被称为迭代查询。
    
*   RA（Recursion Available）：可用递归。该字段只出现在响应报文中。当值为 1 时，表示服务器支持递归查询。
    
*   Z：保留字段，在所有的请求和应答报文中，它的值必须为 0。
    
*   rcode（Reply code）：通过返回值判断相应的状态。
    

> 当值为 0 时，表示没有错误；
> 
> 当值为 1 时，表示报文格式错误（Format error），服务器不能理解请求的报文；
> 
> 当值为 2 时，表示域名服务器失败（Server failure），因为服务器的原因导致没办法处理这个请求；
> 
> 当值为 3 时，表示名字错误（Name Error），只有对授权域名解析服务器有意义，指出解析的域名不存在；
> 
> 当值为 4 时，表示查询类型不支持（Not Implemented），即域名服务器不支持查询类型；
> 
> 当值为 5 时，表示拒绝（Refused），一般是服务器由于设置的策略拒绝给出应答，如服务器不希望对某些请求者给出应答。

> 该部分是用来显示 DNS 查询请求的问题，其中包含正在进行的查询信息，包含查询名（被查询主机名字）、查询类型、查询类。

*   查询名: 一般为查询的域名，也可能是通过 IP 地址进行反向查询
    
*   查询类型：查询请求的资源类型。常见的如果为 A 类型，表示通过域名获取 IP。具体如下图所示
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Saf7dAcImRtnYITxwBq5n8DLtB1PbjDwG0ha5RotqDDf3MxGnj6NGoRg/640?wx_fmt=png)

*   查询类：地址类型，通常为互联网地址为 1
    

_资源记录部分_

> 资源记录部分包含回答问题区域，权威名称服务器区域字段、附加信息区域字段，格式如下

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Saabg9A0LVibhiamOtYlF7dYP4nH0uasRibFmnNxRzexMMiclaECm5Vciatcg/640?wx_fmt=png)

资源记录部分

*   域名：所请求的域名
    
*   类型：与问题部分查询类型值一直
    
*   类：地址类型，和问题部分查询类值一样
    
*   生存时间：以秒为单位，表示资源记录的生命周期
    
*   资源数据长度：资源数据的长度
    
*   资源数据：按照查询要求返回的相关资源数据
    

_4 DNS 解析详解_
------------

> 知道了 DNS 大概是什么，它的域名结构和报文结构，是时候看看到底怎么解析的以及如何保证域名的解析比较稳定和可靠

_DNS 核心系统_

*   根域名服务器 (Root DNS Server), 大哥，管理顶级域名服务并放回顶级域名服务器 IP，比如 "com","cn"
    
*   顶级域名服务器 (Top-level DNS Server), 每个顶级域名服务器管理各自下属，比如 com 可以返回 baidu.com 域名服务器的 IP
    
*   权威域名服务器 (Authoritative DNS Server), 管理当前域名下的 IP 地址，比如 Tencent.com 可以返回 www.tencent.com 的 IP 地址
    

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SauF4ajWOYMzp8dyBf0n3BYULjwHicKmt0aFMoOu2pKHGcGfic3ZGVTfPQ/640?wx_fmt=png)

核心系统

举个例子，假设我们访问 "www.google.com"

*   访问根域名服务器，这样我们就会知道 "com" 顶级域名的地址
    
*   访问 "com" 顶级域名服务器，可知道 "google.com" 域名服务器的地址
    
*   最后方位 "google.com" 域名服务器，就可知道 "www.google.com" 的 IP 地址
    

嘿嘿，目前全世界 13 组根域名服务器还有上百太镜像，但是为了让它能力更强，处理任务效率更高，尽量减少域名解析的压力，通常会加一层 "缓存"，意思是如果访问过了，就缓存，下一次再访问就直接取出，也就是咱么经常配置的 "8.8.8.8" 等

操作系统中同样也对 DND 解析做缓存，比如说曾访问过 "www.google.com"，

其次，还有我们熟知的 hosts 文件，当在操作系统中没有命中则会在 hosts 中寻找。

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Sa3PRRvfVVrMFHib3MsFNcpvOAWH9jOPTs5H9qWyQH8tHrXdkfleGGmEg/640?wx_fmt=png)

DNS 解析过程

嗯？想必应该知道这个过程了，我们再举个例子，假设我们访问 www.qq.com

*   客户端发送一个 DNS 请求，请问 qq 你的 IP 的什么啊，同时会在本地域名服务器 (一般是网络服务是临近机房) 打声招呼
    
*   本地收到请求以后，服务器会有个域名与 IP 的映射表。如果存在，则会告诉你，如果想访问 qq，那么你就访问 XX 地址。不存在则会去问上级 (根域服务器):"老铁，你能告诉我 www.qq.com" 的 IP 么
    
*   根 DNS 收到本地 DNS 请求后，发现是. com，"www.qq.com 哟，这个由. com 大哥管理，我马上给你它的顶级域名地址，你去问问它就好了 "
    
*   这个时候，本地 DNS 跑去问顶级域名服务器，"老哥，能告诉下 www.qq.com" 的 ip 地址码 ", 这些顶级域名负责二级域名比如 qq.com
    
*   顶级域名回复："小本本记好，我给你 www.qq.com 区域的权威 DNS 服务器地址"，它会告诉你
    
*   本地 DNS 问权威 DNS 服务器："兄弟，能不能告诉我 www.qq.com 对应 IP 是啥"
    
*   权威 DNS 服务器查询后将响应的 IP 地址告诉了本地 DNS，本地服务器将 IP 地址返回给客户端，从而建立连接。
    

_5 DNS 进阶之新玩法_
--------------

> 这里主要分享 DNS(GSLB) 的全局负载均衡。不是所有的互联网服务都适用于 GSLB。

*   A 记录
    

> A 记录是名称解析的重要记录，它用于将特定的主机名映射到对应主机的 IP 地址上。你可以在 DNS 服务器中手动创建或通过 DNS 客户端动态更新来创建

*   NS 记录
    

> NS 记录此记录指定负责此 DNS 区域的权威名称服务器。

*   两者区别
    

> A 记录直接给出目的 IP，NS 记录将 DNS 解析任务交给特定的服务器，NS 记录中记录的 IP 即为该特定服务器的 IP 地址

在全局负载均衡解决方案中，NS 记录指向具有智能 DNS 解析功能的 GSLB 设备，通过 GSLB 设备进行 A 记录解析。为了保证高可用，如果为多地部署 GSLB，则均配置记录。另外，GSLB 设备还会对所在的后端服务器公网 IP 进行健康检查，其结果通过自有协议在不同的的 GLSB 设备间同步。解析的步骤如下图

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SacThnXWU060iacejvMvic9dLf16CYAeMFw21RzKLT6hZRCvDqFStC0Hxg/640?wx_fmt=png)

智能 DNS 解析

*   用户给本地 DNS 服务器发送查询请求，如果本地有缓存直接返回给用户，否则通过递归查询获得名服务商商处的授权 DNS 服务器
    
*   授权服务器返回 NS 记录给本地 DNS 服务器。其中 NS 记录指向一个 GSLB 设备接口地址
    
*   GSLB 设备决策最优解析结果并返回 A 记录给本地 DNS 服务器。
    
*   本地服务器将查询结果通过一条 A 记录返回给用户，并缓存这条记录。
    

_6 DNS 实战 (wireshark)_
----------------------

> 使用工具为 wireshark，访问 www.baidu.com

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Say8y2tPh13dvWbr7yRbv52OAbCPV1cickBcGlFkQtRqvqnl9OZpZL5sQ/640?wx_fmt=png)

> 分析 DNS 请求帧，如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaGS7SDquULickmjDB0MWDqrHAUyKpSVznyyaRoFmmKDGKx1BxeJ5IjZQ/640?wx_fmt=png)

DNS 请求帧

从上图我们可知道请求计数为 1，请求的域名为 dss0.bdstatic.com

> 分析 DNS 响应帧

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaVZ4AnbDT8MUe7UgTcibUAP6BmU2vsNTdhj7eYBWJpFDMs9paicsQzoEA/640?wx_fmt=png)

DNS 响应

从响应头可以知道，问题计数为 1，正好对应请求帧 1 个问题。回应了 2 个问题。分别为

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Say3xg4tbeoJoYFaFgPneb9JBEj6mLfnPkDImS3CLtYv71MtgLCCybHQ/640?wx_fmt=png)

answers![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaaNbYoAsluAECKbgmnqlzQpGJicWvDMF80zXR84ibVxvyElXJjFkyxiaEQ/640?wx_fmt=png)

权威域名服务器

从上图可以得出当前共有 13 个权威域名服务器，当然每一个的服务器地址不同，其中类型为 NS 代表权威域名服务器服务器

两个相似面试题

_7 使用 IP 地址访问浏览器的原理_
--------------------

*   打开 chrome 浏览器，输入 IP
    
*   三次握手建立连接
    
*   建立连接以后 HTTP 开始工作，通过 TCP 发送一个 "GET / HTTP/1.1", 服务端给予回应
    
*   解析请求，根据 HTTP 协议规定解析，看看那浏览器想干啥
    
*   哦，原来你想获取我的视频呀，那我读出来拼接为 HTTP 格式给你，回复 "HTTP/1.1 200 OK"
    
*   作为浏览器回复一个 TCP 的 ACK 表示确认
    
*   浏览器收到响应数据后，需要使用相应的引擎进行渲染，将更好的页面展现给用户
    

_8 使用域名访问浏览器的原理_
----------------

> 这一次从浏览器角度回答，相信大家已经了解一部分浏览器知识了，我们先看看 URL 到网页展示的完整流程是什么样子

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7Sa8GbxHYYfzCVj1y6FjF0q0D4nS5bmLb4B6iagqkYeTs6ojvWmHds9SVQ/640?wx_fmt=png)

*   用户输入
    

> 在地址栏输入相应的内容，如果为关键字，如果直接输入搜索内容，浏览器默认引擎会合成为 URL, 如果符合 URL 规则，加上协议合成完整 URL, 回车就会出现加载页面，也就是等待提交文档的阶段

*   URL 请求过程
    

> 此时浏览器进程将 URL 通过进程间通信的方式发送给网络进程，开启真正的请求流程。注意了，网络进程这里也有缓存，它会现在本地缓存查看是否缓存了资源，如果有则直接返回。如果没有，那就需要 DNS 解析获取服务器 IP 地址 (HTTPS 还少不了 TLS 连接)

此时使用 IP 和服务器建立三次握手。连接成功开始构造请求头等信息。

服务器收到请求信，根据请求信息生成响应信息给网络进程。然后开始解析响应头内容。如果返回值为 302/301, 说明需要跳转到其他 URL, 如果为 200 则继续处理该请求。

> URL 的请求数据类型多种，对于浏览器而言是怎么区分的呢

这个时候就必须强调下 Content-type 了，因为他明确服务器返回响应体数据属于什么类型，此时的浏览器也会根据 Content-type 对决定响应体是什么内容

*   进入渲染阶段
    

> 通常情况下，当前多进程架构的浏览器对于每一个页面都有一个渲染进程，前提是如果从 X 页面打开 Y 页面，x 和 y 属于同一个 "站点"(使用相同的协议和根域名)，此时 y 页面会复用 x 页面，否则 y 页面会单独对应一个渲染进程。

*   提交阶段
    

> 渲染进程收到浏览器进程的 "提交文档" 后，通过和网络进程使用 "管道" 的方式通信。一旦这些文档数据传输完成，渲染进程就会告诉浏览器进程 "确认提交"，此时浏览器进程收到 "确认提交" 就会更新地址栏的 URL，历史状态等，这就是为什么当我们在地址栏输入地址信息后需要加载一小会儿到另一个页面。over

*   渲染阶段
    

> 文档提交以后，此时就需要使用 js，css 等美化页面，并通过构建 DOM 树等让用户有更好的使用体验。

_9 DNS 劫持_
----------

> 到这里我们至少知道了 DNS 可以将域名映射为 IP，并且知道了使用了多种缓存方案来减少 DNS 访问的压力。那么 DNS 一旦出错，很可能将域名解析到其他 IP 地址，这样我们也就无法正确访问网页 (PS 是不是有的时候发现开启不了网页但是 qq 等可以使用，很可能就是 DNS 搞鬼了哟)

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SamzGVsIicsKsz9SpxKWK3kTbNc8kd6pLmTwcIKCMpARs8DwS52icUACvQ/640?wx_fmt=png)

DNS 劫持

*   利用 DNS 服务器进行 DDOS 攻击
    

> 什么是 DDOS，我们应该知道 SYN Flood，是一种 DoS(拒绝服务攻击) 与 DDOS(分布式拒绝服务攻击的方式), 利用大量的伪造 TCP 请求让被攻击方资源榨干。

![](https://mmbiz.qpic.cn/mmbiz_png/NdsdouZwicafEzHnoaNaynIh3vHS1ic7SaDoLDo2DicibW5PiaJBBq5q4xcT1OWpAOYibJJ2s7icibcT1NVdn6OZDZCfTg/640?wx_fmt=png)

DDOS

我们假设攻击者已经知道了攻击者 IP(如果需要了解这一部分内容，可以去搜索主动被动信息搜集 / sodan 等关键字)，此时攻击者使用此地址作为解析命令的源地址，当 DNS 请求的时候返回恰巧也是被攻击者。此时如果足够多的请求 (群肉鸡) 就很容易使网络崩溃。

*   缓存感染
    

> 我们已经知道了在 DNS 查询过程中，会经过操作系统的缓存，hosts 文件等，如果将数据放入有漏洞的服务器缓存中，当进行 DNS 请求的时候，就会将缓存信息返回给用户，这样用户就会莫名访问入侵者所设置的陷阱页面中。

*   DNS 信息劫持
    

> 看到这里的小伙伴，先思考一个问题，在 TCP/IP 协议栈中，三次握手中的序列号到底什么意思？

其功能之一就是避免伪装数据的插入。我们知道，如果我们知道 DNS 报文中的 ID，我们就可以知道这个 ID 请球员位置。作为攻击者，会通过旁路监听客户端和服务端的会话，拿到 DNS 中的 ID，此时相当于在 DNS 服务器之前拿到 ID，伪装 DNS 服务器回复客户端，引导客户端访问恶意的网站

_电脑小故障_

> 比如 qq 可用但是浏览器就是不好使

*   输入：http://192.168.1.1(可能是 http://192.168.0.1)，输入路由器用户名密码
    
*   DHCP 服务器 -----DHCP 服务 -，修改 DNS 为更加可靠的 DNS 服务器 IP. 保存即可
    

方法 2: 修改路由器 password

*   地址栏输入 "http://192.168.1.1", 登录并进入路由器页面
    
*   系统工具 -- 修改登录口令页面
    

_保护域名 / 尽量避免攻击_

*   备份策略。一般至少会使用两个域名，一旦其中一个被攻击，用户可以通过另一个访问
    
*   随时留意域名注册中的电子邮件
    
*   保存好所有权信息 (比如账单记录，注册信息等)
    
*   随时关注安全补丁
    

_10 本文涉及高频面试题 (自行测试)_
---------------------

*   讲讲 DNS 原理
    
*   进程与线程
    
*   递归查询和递归查询区别
    
*   DNS 解析流程
    
*   chrome 架构演变
    
*   ELF 是什么，数据段，代码段，全局变量等分别存放在哪儿
    
*   DNS 劫持
    
*   描述下 DDOS 与 DOS 攻击
    
*   使用 IP 地址访问 web 服务器
    
*   使用域名访问 web 服务器过程
    
*   可重定位什么意思？
    
*   静态库与动态库的区别
    
*   进程与线程间共享数据
    

巨人的肩膀
-----

https://baike.baidu.com/item/%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90/574285?fr=aladdin  
https://baijiahao.baidu.com/sid=1623434144833493787&wfr=spider&for=pc  
https://www.sohu.com/a/229518877_609556  
https://time.geekbang.org/column/intro/100024701?utm_source=pinpaizhuanqu&utm_medium=geektime&utm_campaign=guanwang&utm_term=guanwang&utm_content=0511