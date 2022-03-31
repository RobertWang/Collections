> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/K_wEqhemhnvn8l5qqBW4Rg)

现在，来自康奈尔大学和 Meta AI 等机构，通过 Prompt 来调整基于 Transformer 的视觉模型，结果发现：

**完全可以！**

比起全面微调，Prompt 性能提升显著。无论模型的规模和训练数据怎么变，24 种情况中有 20 种都完全胜出。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBl4cupcrpHHzJVObicLvAtaibm3rmNf1I7ujqpgtAJkMrshN0ReHtxoCCJUMtnMzbztXzkEOxic9PqA/640?wx_fmt=png)

与此同时，它还能大幅降低每项任务所需的存储成本。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBl4cupcrpHHzJVObicLvAtaVy4CjwYHNneqhMMh1zy5ERQka4G9v9EW9Ap5X18Ef69geGu01jv3Yw/640?wx_fmt=png)

**只使用不到 1% 的模型参数**

大家一贯使用的**全面微调**（full fine-tuning），需要为每个下游任务存储和部署单独的主干参数副本，成本太高，尤其是现在基于 Transformer 的模型越来越大，已经超过 CNN 架构。

所谓 Prompt，最初指的是在输入文本中预编语言指令，以便预培训的语言模型后续可以直接理解各种下游任务。

它曾让 GPT-3 即使在少样本或零样本的情况下表现出很强的泛化能力。

最近一些成果则表明，Prompt 与完全微调的性能相当，参数存储量还减少了 1000 倍。

NLP 中的高超性能让不少人开始在 CV 领域中探索 Prompt 的魔力，不过都只局限于跨模态任务中文本编码器的输入。

在实操中，这些附加参数只用预先加入到每个 Transformer 层的输入序列中，并在微调期间与线性 head 一起学习。

他们一共探索出两种变体：

**VPT-Deep** 变体为 Transformer 编码器每层的输入预先设置一组可学习的参数；

**VPT-Shallow** 变体则仅将提示参数插入第一层的输入。

两者在下游任务的训练过程中，只有特定于任务的提示和线性头的参数会更新，而整个 Transformer 编码器被冻结。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBl4cupcrpHHzJVObicLvAtaxiaLiccJxicDTQBGpZraOcdSrx9O37AmMBU5n1hCYEfnbIb8CvrY1xl3w/640?wx_fmt=png)

接下来，是骡子是马？拉出来溜溜～

实验涉及两种在 ImageNet-21k 上预训练好的主干，**一个来自 Vision Transformer，一个来自 Swin Transformer**。

进行对比的**微调方法有三大种，7 小种，**包括：

（2）以分类头为重点的微调，包括 Linear、Partial-k 和 Mlp-k 三种；

（1）由 5 个基准细粒度视觉分类任务组成的 FGVC；

和其他微调方法相比（b、c 组），VPT-Deep 的性能则全部胜出。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBl4cupcrpHHzJVObicLvAtaUIHZjHDHHO2ibT1zwp8mIOq5icqxCqWQwicCpLU8dmBendeNdk54KbibMA/640?wx_fmt=png)

此外，选择**不同主干参数规模和模型规模**的 ViT（ViT-B、ViT-L 和 ViT-H）进行测试还发现，VPT 方法不会受影响，依然基本保持性能领先。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBl4cupcrpHHzJVObicLvAtaMzeZZKUhVRJ9Dj9WZOe2yicBtR3mBWJNOKHegch4va313am5d8d2iciaA/640?wx_fmt=png)

而在 Swin Transformer 中，全面微调法的平均准确度虽然更高，但也付出了巨大的参数代价。

其他微调方法则全部不敌 VPT。

**一作贾梦霖**，康奈尔大学信息科学（Information Science）博士生，主要研究方向为视觉和文本信息的细粒度识别，截至目前共发表过 4 篇顶会。

**共同一作为唐路明**，也是康奈尔大学的一位计算机博士在读学生，本科毕业于清华大学数学与物理专业。

他的主要研究方向为机器学习和计算机视觉的交叉领域。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBl4cupcrpHHzJVObicLvAtavOtQribFFkBh2tHXkGicMhe29I5ZMkN1JyQ4icR7icGGu2hgpIu4H9vKRA/640?wx_fmt=png)

论文地址：

https://arxiv.org/abs/2203.12119