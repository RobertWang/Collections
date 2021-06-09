> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247580887&idx=4&sn=87b13ec2e6807bf7e2658d9c60bd4408&chksm=e8d10ba5dfa682b37cf20fe2da8373e14b83aa89f337a4e88c1acf763d5f2b074106b4916b98&mpshare=1&scene=1&srcid=0608mUIe6cFnvyRqbvr3yqQU&sharer_sharetime=1623114661153&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

##### 丰色 发自 凹非寺  
量子位 报道 | 公众号 QbitAI

不得不说，自发布以来，苹果 M1 芯片的各项测评表现都令人印象深刻。甚至此前有人发现 **M1** Mac Mini 在某项 TensorFlow 速度测试中的得分**高于英伟达 RTX 2080Ti**。

所以一位从事**光线追踪** （Ray tracing）技术的程序员，就对 M1 产生了兴趣。

他发现，M1 比他的 Haswell（英特尔第四代酷睿处理器）旧电脑 Cinebench 得分高 1.6 倍，比 Tiger Lake（第 11 代）新电脑得到的分数高 2 倍！

于是他又自己动手测了个新的跑分，看一看 M1 **纯粹的**光线追踪性能。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2ia6ibN47GGs4nFxd7iczjsSJbmxDPPb98KmkQSE4GmReZ4HalFY5kmZpA/640?wx_fmt=png)

###### **△** 图源维基百科，光线追踪可以制作照片级真实感的图像。  

那 M1 芯片的光追性能可以打英伟达 RTX 吗？

往下看。

测试基准 ChameleonRT
----------------

这位测试者买了一台 Mac Mini 来测试他自己的光线追踪项目 **ChameleonRT**。

也就是此次测评采用的基准，一个开源的光线追踪器，可在多个光线追踪后端（Embree/DXR/OptiX/Vulkan/Metal/OSPRay）上运行。

这和文章开头提到的很流行的光线追踪基准程序 **CineBench** 有点不一样。

AnandTech 的 CineBench 跑分也使用了 Embree 进行光线追踪。这是一个由英特尔开发的 CPU 光线追踪库，提供优化的加速结构遍历和原始交叉内核。Embree 已广泛应用于电影、科学可视化和其他领域。所以 ChameleonRT 也实现了一个 Embree 后端。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2Hywx3h8PnlHZyfsOUPah1qN52auEs8NnJCNQkfhO9b734P56q0StUw/640?wx_fmt=png)

接下来就切入正题看看 M1 在 ChameleonRT 基准上的光线追踪性能评测：

M1 的光线追踪性能比较
------------

测试使用以下两个场景：Sponza 和 San Miguel。

Sponza 是一个有 26 万个三角形的小场景，San Miguel 有 996 万个，分别对应左右两图：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2LvhLI8F2dTx5ibibiaNsvglkMeyqgE2pepDUicJH4N78gLReIyharzq1zw/640?wx_fmt=png)

比较方法：使用基准运行渲染 1280x720 像素图像并运行约 200 帧，然后记录**平均帧速率** (FPS) 和**每秒追踪的百万光线数** (MRay/s)。

下面是使用 Embree CPU 后端渲染两种场景的 “公平” 比较结果：

Sponza

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2GaPIyqzWRmiaicmhrpafvXIUKM8mfx0iafmCvbkEichDwHMb69cv0L2NFg/640?wx_fmt=png)

San Miguel

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2AoicC8meBGjqLLxD4CVTB0ADibicEDXG8HBzqU18bRb75MBmAPHBnawLQ/640?wx_fmt=png)

苹果 M1 芯片都居于中间水平。

此外，出于好奇心，测评人员还进行了 “**极其不公平**” 的比较，将 M1 上的 Metal GPU 光线追踪后端与 英伟达 RTX 2070 上的 DirectX 光线追踪、Embree CPU 后端与 i9-9920X 进行比较。

“不公平” 比较结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2ba5SCibHDVZcSWGic4GabNRQiaC6O4MdfLgutUvtw2gu8FnEB37ibpaWibg/640?wx_fmt=png)

**△** Sponza 使用 Embree CPU 后端进行的基准测试结果

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x22ciazRMQojbfqDHcJ3oKNxE6fTsYYplibV6ZUtefF2ybnctGdQ09YOaA/640?wx_fmt=png)

###### **△** San Miguel 使用 Embree CPU 后端进行的基准测试结果

可以发现，i9-9920X 在使用 AVX2 指令集时表现最好。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2cFFbKzorm41h3mxZeXibJbz4ibcU0CS1ic3VaTNicg4BcnxX41wAAtCY4g/640?wx_fmt=png)

**△** Sponza 使用 GPU 后端进行的基准测试结果

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2bnzdwvMhs1YFcwtrfB9taDRTwL1BTHMa9wHe7dvh23vlvjODbwGhQA/640?wx_fmt=png)

**△** San Miguel 使用 GPU 后端进行的基准测试结果

可以看出，分数差距较大，但评测人员本身也没有期待它能超过**极具竞争力**的英伟达 RTX 2070，只是为了看看 M1 能排在什么位置。

最后，评测人员总结道:

> 但即使是当前这样的性能水平，对于轻量级芯片来说也令人印象深刻，因为它不会遇到与 XPS 13 相同的热问题（做这些基准测试时风扇很安静），并且可以在 1/4 SIMD 宽度的 CPU 上提供更好的性能，还有一个 GPU 光线追踪 API，可以在这些基准测试中提供比 CPU 快 1.6-2 倍的加速。
> 
> 很期待在未来的 M 系列芯片中看到对 8-wide 的 SIMD 和硬件加速光线追踪的支持。

“毫不奇怪 M1 的表现不是很好”
-----------------

而对于以上 M1 芯片的光线追踪性能评测，有网友用一句 “太长不看 “总结道：

基本上是 7 年前一台拥有 i7-4790k 处理器的台式机的性能。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x25Zx9Nj0jwulCzo3ps2DAEXxhNnkgu8Jckx9Hdy0Sib1kicJ7yMtdSbKw/640?wx_fmt=png)

评论区看法基本一致，另一位网友总结道：任何支持光线追踪的东西都有专门的硬件来处理，毫不奇怪 M1 的表现不是很好。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2fB7grzJNDla8cb28c2Mx8YufeTfbA1znviagxQhhD4yyd9EGdPg3iboA/640?wx_fmt=png)

也就是说，“如果你想要一个 M1 Mac 来处理光线追踪，性能好不了。但这并不是什么大事，因为图形并不是 M1 真正的卖点。”

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA4utQTXRkCibv3icyVMQe5x2EN7LO8cKIiaZoWmsLibxeVhvNBI8icBlEsnGeM3RibT2aDaNC3gdiatFRaA/640?wx_fmt=png)

参考链接：

[1]https://www.willusher.io/graphics/2020/12/20/rt-dive-m1  
[2]https://www.reddit.com/r/apple/comments/nomun4/a_dive_into_ray_tracing_performance_on_the_apple/