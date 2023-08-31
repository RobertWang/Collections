> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7243725204002127932)*   写 AI 程序，训练模型和优化
*   让模型的执行 高效并稳定，这就涉及到 CUDA 优化和自定义

说到 CUDA，可以研究下

1.  内存管理
2.  程序优化: 探索并行化
3.  程序优化: GPU 内存优化策略
4.  程序优化: 指令优化

**选 N 卡（NVIDIA）还是 A 卡（AMD）？**

目前的答案是最好选 N 卡，虽然有部分 AI 绘画软件对 A 卡有支持，但大部分 AI 制图**对 N 卡的支持要更好**，部分 A 卡只能跑在兼容模式下，丧失了很大的性能，且经常出现崩溃等问题。

所以，玩深度学习、AI 绘画这些，就别考虑 A 卡了

说到这，**已经知道 要选 NVIDIA 的 GPU 了**

选什么型号的 GPU 呢
============

显卡的性能由两个部分决定，一是**核心**，二是**显存**。

核心负责处理运算图形数据，显存则负责缓存图形数据，核心在运算时要用到的数据都是在显存中调用的，所以显存的性能直接决定了核心调用数据的效率，间接影响了显卡的性能。

**RTX 3060**
------------

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/311042a36fce4a0c87bbe0bbac2317a5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

### 3060 显卡和 3050 的显卡性能差多少啊？

**3050 和 3060 的性能差距在 25% 左右**，那么目前最便宜的 3060 在某夕夕百亿补贴是 2400 多元，3050 的价格在 1800 元出头，3050 和 3060 价格查了 600 多元。在之前的对比测试中，这个性能差距仅仅低于 3060 和 3060ti 的差距。比 3060ti 和 3070，3070 和 3070ti 的对比差距大不少。比如 3070 和 3070ti 之前的差价在 500 左右，但只换来了不到 10% 的性能提升。而且 3060 和 3050 的 1080p 分辨率下，3A 游戏体验还是有比较大的差别的。3050 在 1080p 高特效下基本在 40~50 帧之间浮动，而 3060 基本可以稳定在 60 帧出头。如果是在 1080P 玩 3A 游戏，还是 3060 的性能比较稳定。但如果是玩网游的话，3050 也绝对没问题。如果主玩单机游戏，3060 比 3050 更好，3060 也比 3050 更有性价比。现在显卡价格昂贵，配置一台高性能电脑很贵不说而且后续想要升级麻烦还要再花钱购置硬件，如果也想要高性能电脑可以试试赞奇云工作站，云电脑自由度更高。赞奇云工作站拥有专业级显卡、超大内存等多种机器配置。机器显卡更新及时，提供高配机型，海量资源可按需选择，内置软件中心提供最新软件安装包，一键下载，省去搜索时间，提高工作效率。

如果不嫌慢，刚入门可以选择 3060，毕竟人家 12G 显存，哈哈

**RTX 3070**
------------

3070 相比较于 3060，显存不进则退，8G 显存，但是人家速度快

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f99aa1b434dd44a28ace89a59af64aba~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d140e5c3bffc4d7e8f29015e5c74f475~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

大数据量选 3060，小数据量想提高速度选 3070，但 3080Ti 才是你们永远的家。

**RTX 3080**
------------

为什么都说 3080 才是最终的 归属。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/085a47ac418443eb9b7b9dbfdd5e6073~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

RTX3080 采用三星 8nm 制程工艺，集成 280 亿晶体管，包含 68 个 SM 单元，总共 8704 个 CUDA 核心，搭载新一代 RT 内核、张量内核。

**RTX3080 拥有 1.71 GHz 的 Boost 频率，FP16 的最大性能达到 238TFLOPs，FP32 的最大性能达到 29.7 TFLOPs。**

RTX3080 达到了 4.5 倍 980 的性能，但价格只有 699 美元起，如果以 2080 对比，快 2 倍性能提升，真是映证了摩尔定律。

具有 8,960 个 CUDA 内核的 TU102 内核是使其成为深度学习绝佳选择的另一个因素。

**3080 有 12GB 的显存，12GB 在很多情况下都不够用，但是能支持 AI 绘画了，要运行 Transformer 模型的话，至少需要 24GB。**

如果你要 Transformer 比较推荐租用大显存的机器跑

一张表格，展示出在训练和推理过程中，一美元能买到多少算力；这在一定程度上体现了英伟达众显卡的性价比。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b080136a7df4663babc6ca5423f655f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

我们大部分情况使用 3080 已经够用了, 比 3070 更快，**而且毕竟 3080 性价比在这呢**

别告诉我 不差钱。。。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9994236fecc243528fadf02413395e86~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

推荐
==

**推荐：RTX 3080 12G**，理由：更大的显存，可以运行参数更多的深度神经网络，性价比高。

买还是租呢
=====

买的话
---

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b411a07a3624448b8a4de1b642934b2c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

租的话
---

### 第一个租的

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd854e52047f43679c6a997c9c21a0b8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

一天 30，**一个月 900**

### 第二个租的

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aeadeece699248b1b035636476ff102a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

这也是**一个月 900，还不是 windows 系统, 这什么东西**

### 第三个租的

靠，这也太贵了，不行，我去咸鱼 淘宝看看，这可能是百度出来的贵

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8af699fed1064474923acf2adbb38d12~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?) 一个月 840 还行，比前面的便宜点

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b70391e652145558ea42fa53b0169d1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

嗯嗯 720 一个月还可以，还支持向日葵，不错

### 云服务呢

看了这半天，云服务呢，我们先不看最贵的阿里云，看看腾讯云

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d1e6d610dd0471095208dfabd95f5d9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

我们不是为了生产环境使用，我们是为了个人用，性价比比较重要，3.4 * 24 * 30 = **2448**

靠，这也太贵了，不看了，看了半天，** 还是要选咸鱼 淘宝这种租服务器，用好了，然后我们自己买一个 满足自己场景的显卡 **

在哪租呢？
=====

目前最便宜的就是 720 元一个月，其实这里可以，**在本地把代码写好，准备训练的时候 临时买几天的，这种比较靠谱，性价比比较高**

**如果你感觉每次都要重新配置环境 ，有点累，也可以租一个月，可以自己找便宜点的，你就集中使用一个月，比较适合学习中的同学**

彩蛋
==

**大家可以看下评论区，评论区大哥有便宜的，我为了不让人家说是推广，我只送，来源也不会告诉，防止说是推广**

我朋友圈 随机抽一个同学送一台 RTX 3080 一个月额度 （6 月 18 号抽）

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04416051dddc43728006084b99e4735a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)