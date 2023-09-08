> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTg2NTU0Ng==&mid=2656637964&idx=2&sn=20602b514b034ef0e68e082ae8e66a8c&chksm=bd5c42a98a2bcbbf026198f73c979a6995169638488265f6342bc044165a2551cdd5a9db69a1&scene=21#wechat_redirect)

> 每隔一段时间，就有关于哪种编程语言最好，哪种不好的讨论。我认为对这样的话题不会有最终答案，因为它取决于很多因

每隔一段时间，就有关于哪种编程语言最好，哪种不好的讨论。我认为对这样的话题不会有最终答案，因为它取决于很多因素，例如应用程序或脚本的目的、要解决的问题，甚至是业务需求的差别等。

为了简化问题，我们不会考虑任何以上因素。

我们将关注一个更普遍的问题，例如通过使用冒泡算法 BubbleSort 算法对每种编程语言的数据数组进行排序，从而达到测试这些流行编程语言的性能。

只选择冒泡算法的原因是它是最慢的算法，时间复杂度为 O(n²)。本例中将使用的数据样本是一个包含 10000 个随机生成数字的数组。

该基准测试在具有 16G 内存、Intel Core I7 CPU 5 核 — 2.6Ghz 和 250Gb SSD 硬盘的机器上执行。

大家都是对结果很看重的人，下表将向大家展示测试后的数据。

![](https://mmbiz.qpic.cn/mmbiz_png/X1wOHbVRDnzLcRS1a3cq8zvyicws0MAxbdShQic2Cdej8GWCfyjdZ20MoIty5sFia1f0ibKr1m9GibQaSGsZXgP41ug/640?wx_fmt=png)

表 1：每种语言的执行时间（以毫秒为单位）

![](https://mmbiz.qpic.cn/mmbiz_png/X1wOHbVRDnzLcRS1a3cq8zvyicws0MAxbrYqNlcBOMyJiaIynD1GCPEIDI1rD0sXAQJEEK2V7gZR4Yr0mzxSicMSQ/640?wx_fmt=png)

图 2：每种语言的执行时间（以毫秒为单位）

如图所示，我们看到某些语言中存在一些重大差异，其中 Python 是迄今为止性能比较差的，而 Golang 以巨大的性能优势获胜，即使它与 Node.js 相比也是如此。

各位可以在 GitHub 上通过以下链接中的 docker 设置找到此测试基准：

![](https://mmbiz.qpic.cn/mmbiz_png/X1wOHbVRDnyLYZribJgjSh2tsJV1269WgwaqAib9lIAic1vpcaCA3091wC08M0jEQOUdFqYkFYLjYs9yxPp77D5UA/640?wx_fmt=png)

https://github.com/abame/bubble-sort-benchmark

值得一提的是，即使你的配置参数与我的相同，你也可能会从我那里得到不同的结果，因为数据数组是随机生成的。

希望这个简单的基准测试能够指导大家在未来的项目中选择合适的语言。

> 编译：艾多多