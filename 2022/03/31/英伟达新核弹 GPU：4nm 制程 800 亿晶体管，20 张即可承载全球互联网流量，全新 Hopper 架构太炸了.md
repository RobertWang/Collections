> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/mLl_smTLXXYsI3-K4EcyVw)

##### 丰色 萧箫 发自 凹非寺  
量子位 | 公众号 QbitAI

他来了他来了，老黄带着英伟达的最新一代 GPU 来了。

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWxrL9umibKSZ2gmR3N4QxAqQcKBS5s3nT30PSKtStHIqH6BbmQ6LINog/640?wx_fmt=gif)

之前大家猜的 5nm 错了，一手大惊喜，老黄直接上了**台积电 4nm** 工艺。

新卡取名 H100，采用全新 Hopper 架构，直接集成了 800 亿个晶体管，比上一代 A100 足足多了 **260 亿个**。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWOHmtGgS79tlv9qh348MEeibE4t87pXTcicCgzvdxTTKCE7ibFUXmkCY8w/640?wx_fmt=png)

内核数量则飙到了前所未有的 **16896 个**，达到上一代 A100 卡的 2.5 倍。

浮点计算和张量核心运算能力也随之翻了至少 3 倍，比如 FP32 就达到了达到 60 万亿次 / 秒。

特别注意的是，H100 面向 AI 计算，**针对 Transformer** 搭载了优化引擎，让大模型训练速度直接 ×6。

（可算知道 5300 亿参数的威震天 - 图灵背后的秘诀了。）

作为一款性能爆炸的全新 GPU，不出意外，H100 将与前辈 V100、A100 一样成为 AI 从业者心心念念的大宝贝。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicW7edpicB7Vo90DxDxSKggHcXa7D9F6YMcYacDXSM9a6EhFMTuLibic7F5g/640?wx_fmt=png)

不过不得不提，它的功耗也爆炸了，达到了史无前例的 **700W**，重回核弹级别。

关于自研的 Grace CPU，这次大会也公布了更多细节。

没想到，老黄从库克那里学来一手 **1+1=2**，两块 CPU“粘” 在一起组成了 CPU 超级芯片——Grace CPU Superchip。

Grace CPU 采用最新 Arm v9 架构，两块总共拥有 144 个核心，拥有 1TB/s 的内存带宽，比苹果最新 M1 Ultra 的 800GB/s 还高出一截。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWSibqTmTPEcyUgcUWibQltz5KlcZmUHaM0Eb8pd5yhudBnbOP3QiajZoCw/640?wx_fmt=png)

基于全新 CPU、GPU 基础硬件，这次发布会也带来了下一代企业级 AI 基础设施 DXG H100、全球最快 AI 超算 Eos。

当然，英伟达作为真正的元宇宙先驱，也少不了 Omniverse 上的新进展。

下面具体来看看。

首款 Hopper 架构 GPU，性能暴增
---------------------

作为上一代 GPU 架构 A100（安培架构）的继承者，搭载了全新 Hopper 架构的 H100 有多突飞猛进？

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWgedaxL7ov3XjHwApVYyA5HRCOvJ5wicAWeEubBtgiaAKfq5AUgbia905A/640?wx_fmt=png)

话不多说，先上参数：

老黄可谓下血本，先是直接采用了**台积电 4nm** 工艺，晶体管一口气集成了 **800 亿**个。

要知道，上一代 A100 还只是 7nm 架构，这次发布会出来前，外界不少声音猜测老黄会用 5nm 制程，结果一发布就给大家来了个大惊喜。

最恐怖的是 CUDA 核心直接飙升到了 **16896 个**，直接达到了 A100 的近 2.5 倍。（要知道从 V100 到 A100 的时候，核心也不过增加那么一丝丝）

这次可不能感慨老黄刀法精准了。

再看浮点运算和 INT8/FP16/TF32/FP64 的张量运算，性能基本全部提升 **3 倍**不止，相比来看，前两代的架构升级也显得小打小闹。

这也使得 H100 的热功耗（TDP）直接达到了前所未有的 **700w**，英伟达 “核弹工厂” 名副其实（手动狗头）。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWvQkuHLNVXMxdg5vORXSP8xDEovDEHwibdYwbBibLy6SQny6Xy2FE45tQ/640?wx_fmt=png)

话又说回来，这次 H100 也是首款支持 PCle 5.0 和 HBM3 的 GPU，数据处理速度进一步飞升——内存带宽达到了 3TB/s。

这是什么概念？

老黄在发布会上神秘一笑：只需要 20 个 H100 在手，全球互联网流量我有。

整体参数细节究竟如何，与前代 A100 和 V100 对比一下就知道了：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWIibnoBxYiaNXHCTV3YFakBuR3lDW5VtIAvRVwelA1DIvH5fraN4Cic2nA/640?wx_fmt=png)

###### **△**图源 @anandtech  

值得一提的是，Hopper 架构的新 GPU 和英伟达 CPU Grace 名字组在一起，就成了著名女性计算机科学家 **Grace Hopper** 的名字，这也被英伟达用于命名他们的超级芯片。

Grace Hopper 发明了世界上第一个编译器和 COBOL 语言，有 “计算机软件工程第一夫人” 之称。

训练 3950 亿参数大模型仅 1 天
-------------------

当然，Hopper 的新特性远不止体现在参数上。

这次，老黄特意在发布会上着重提到了 Hopper 首次配备的 **Transformer 引擎**。

嗯，专为 Transformer 打造，让这类模型在训练时保持精度不变、性能提升 **6 倍**，意味着训练时间从几周缩短至几天。

怎么表现？

现在，无论是训练 **1750 亿**参数的 **GPT-3** （19 小时），还是 **3950 亿**参数的 Transformer 大模型（21 小时），H100 都能将训练时间从一周缩短到 1 天之内，速度提升高达 9 倍。

推理性能也是大幅提升，像英伟达推出的 **5300 亿** Megatron 模型，在 H100 上推理时的吞吐量比 A100 直接高出 30 倍，响应延迟降低到 1 秒，可以说是完美 hold 住了。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWhV6wSuJGfxW4tLIGfPhlRzglJ3cUVQIZ4j6WCr9bYKnqsLbhNCBEyA/640?wx_fmt=png)

不得不说，英伟达这波确实突入了 Transformer 阵营。

在此之前，英伟达一系列 GPU 优化设计基本都是针对**卷积**架构进行的，接近要把 “I love 卷积” 这几个字印在脑门上。

要怪只怪 Transformer 最近实在太受欢迎。（手动狗头）

当然，H100 的亮点不止如此，伴随着它以及英伟达一系列芯片，随后都会引入 NVIDIA **NVLink** 第四代互连技术。

也就是说，芯片堆堆乐的效率更高了，I/O 带宽更是扩展至 900GB/s。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWVQ85c7ibLXXfiasAVcn1tJzTQBnSA2y9yS5XS5nK42PzicyJLxNwfmJdw/640?wx_fmt=png)

这次，老黄还着重提到了 GPU 的**安全性**，包括实例之间具有隔离保护、新 GPU 具有机密计算功能等。

当然，数学计算能力也提升了。

这次 H100 上新的 DPX 指令可以加速动态规划，在运算路径优化和基因组学在内的一系列动态规划算法时速度提升了 7 倍。

据老黄介绍，H100 会在今年**第三季度**开始供货，网友调侃 “估计也便宜不了”。

目前，H100 有两个版本可选：

一个就是功率高达 700W 的 SXM，用于高性能服务器；另一个是适用于更主流的服务器 PCIe，功耗也比上一代 A100 的 300W 多了 50W。

4608 块 H100，打造全球最快 AI 超算
------------------------

H100 都发布了，老黄自然不会放过任何一个搭建超级计算机的机会。

基于 H100 推出的最新 DGX H100 计算系统，与上一代 “烤箱” 一样，同样也是配备 8 块 GPU。

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWjQ4ykj5As5UaMg22nLicHn3TnraGoc9OyNPsmc97ekM2KmDKt6B8e8w/640?wx_fmt=gif)

不同的是，DGX H100 系统在 FP8 精度下达到了 32 Petaflop 的 AI 性能，比上一代 DGX A100 系统整整**高了 6 倍**。

各 GPU 之间的连接速度也变得更快，900GB/s 的速度接近上一代的 **1.5 倍**。

最关键的是，这次英伟达还在 DGX H100 基础上，搭建了一台 **Eos 超级计算机**，一举成为 AI 超算界的性能 TOP 1——

光就 18.4 Exaflops 的 AI 计算性能，就比日本的 “富岳”（Fugaku）超级计算机**快了 4 倍**。

这台超算配备了 576 个 DGX H100 系统，直接用了 **4608 块 H100**。

即使是传统科学计算，算力也能达到 **275 Petaflops** （富岳是 442 Petaflops），跻身前 5 的超算是没什么问题。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWqCR4sJ5GPpSvme8mDTzwFibT0XwciayibTwTTAcxTESMxMgbIEC1YCz0g/640?wx_fmt=png)

“拼装”CPU，跑分成了 TOP1
-----------------

本次 GTC 大会，老黄仍然 “提了几嘴” 超级服务器芯片 Grace。

它在去年 4 月份的 GTC 大会就已经有所亮相，和当时一样，老黄表示：**有望** 2023 年可以开始供货，反正今年是不可能碰上了。

不过，Grace 的性能倒是值得一提，有了 “惊人进展”。

它被用在两个超级芯片中：

一个是 **Grace Hopper 超级芯片**，单 MCM，由一个 Grace CPU 和一个 Hopper 架构的 GPU 组成。

一个是 **Grace CPU 超级芯片**，由两个 Grace CPU 组成，通过 NVIDIA NVLink-C2C 技术互连，包括 144 个 Arm 核心，并有着高达 1TB/s 的内存带宽——带宽提升 2 倍的同时，能耗 “只要”500w。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicW73YChwSh4pbsR2picAOic4GffYxUD5z3gKibebGecvrrI7COoO57I9ZRg/640?wx_fmt=png)

很难不让人联想到苹果刚发的 M1 Ultra，看来片间互连技术的进展，让 “拼装” 成了芯片行业一大趋势。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWTDdj3LVmMiapbbibj7QEtZKZFUT6Sy4VMoMhX1VkL6xfBELLf7kZ7tpA/640?wx_fmt=png)

Grace 超级芯片在 SPECrate®2017_int_base 基准测试中的模拟性能达到了 740 分，是当前 DGX A100 搭载的 CPU 的 1.5 倍（460 分）。

Grace 超级芯片可以运行在所有的 NVIDIA 计算平台，既可作为独立的纯 CPU 系统，也可作为 GPU 加速服务器，利用 NVLink-C2C 技术搭载一块至八块基于 Hopper 架构的 GPU。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWzTN2RM1B8Fm6QtYafatkkpzSTKFxOUXrdal0dm7Tziagibx1sviaYP2dQ/640?wx_fmt=png)

（嗯，刚说完，老黄的芯片堆堆乐就堆上了。）

值得一提的是，英伟达**对第三方定制芯片开放了 NVLink-C2C**。

它是一种超快速的芯片到芯片、裸片到裸片的互连技术，将支持定制裸片与 NVIDIA GPU、CPU、DPU、NIC 和 SOC 之间实现一致的互连。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWFfrAfwkNicuiatV1skrG29MT9W8r5daA2Z6SO9celYXia7g7FGg8yHEJQ/640?wx_fmt=png)

或许，**任天堂新掌机**可以期待一波？

连工业也要在元宇宙里搞
-----------

当然，除了上述内容之外，这次英伟达也透露了不少与工业应用相关的案例。

而无论是自动驾驶、还是包括虚拟工厂的数字孪生等场景，都与计算机渲染和仿真技术有着密不可分的关系。

英伟达认为，工业上同样能通过在虚拟环境中模拟的方式，来增加 AI 训练的数据量，换而言之就是 “**在元宇宙里搞大训练**”。

例如，让 AI 智能驾驶在元宇宙里 “练车”，利用仿真出来的数据搞出半真实环境，增加一些可能突发故障的环境模拟：

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWaiaQwklbj4SLia1Tlk29wNtct15PpET8ZR5oHXYDornAa9zEdpXTuCsQ/640?wx_fmt=gif)

又例如，搞出等比例、与现实环境中材料等参数完全一样的 “数字工厂”，在建造前先提前开工试运行，以及时排查可能出现问题的环境。

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicW6zRxpEp4tSd8nCFSic73EpYzy1MiaU94H8AsWERZqau6Coxl8RSOwIjQ/640?wx_fmt=gif)

除了数字孪生，数字资产的生产也是元宇宙早期建设阶段需要着重考虑的部分。

在这方面，英伟达推出了随时随地能在云端协作的 **Omniverse Cloud**。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWAPTSnVVyQjU9iacItkVukPovrYcgRoEz2GCND4hcvlvmj3V8tO3NHIg/640?wx_fmt=png)

最有意思的是，这次发布会上还演示了一套 AI 驱动虚拟角色系统。

现实中 3 天，虚拟角色在元宇宙里**靠****强化学习苦练 10 年功夫**。

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWAKWYU9icm0Mq9mTibibkxWibsQDvRPcIuib9UyOCOh6vO5PSgfUo7BxDWCQ/640?wx_fmt=gif)

等练成一身本领，出来无论到游戏还是动画里都是个好 “动作演员”。

用它生成动画无需再绑定骨骼、k 帧，用自然语言下指令即可，就像导演和真人演员一样沟通，大大缩短开发流程。

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtBbeUZUEAYAfNldQpwRSmicWkgvdwUqkuKricicPecfxhSibvbjwHUg3icwqTxyibXkzrJlJYzLDzKyzo6Q/640?wx_fmt=gif)

要论**元宇宙基建**还得看老黄啊。

Venturebeat 对此评价称，“这些案例给元宇宙赋予了真正的意义”。

那么，你看好英伟达的 omniverse 前景吗？

更多详情，可以戳完整演讲地址（带中字哦）：  
https://www.nvidia.cn/gtc-global/keynote/?nvid=nv-int-bnr-223538&sfdcid=Internal_banners

参考链接：  
[1]https://www.anandtech.com/show/17327/nvidia-hopper-gpu-architecture-and-h100-accelerator-announced  
[2]https://venturebeat.com/2022/03/22/nvidia-gtc-how-to-build-the-industrial-metaverse/