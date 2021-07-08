> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zhihu.com](https://www.zhihu.com/question/30712664) ![](https://pic1.zhimg.com/v2-34bc4e0f217255dbfa920842ecd19dc6_xs.jpg?source=1940ef5c)taokongcn​

直接搬一下

@魏秀参主页的 trick&tip，感觉总结的很好  
[Must Know Tips/Tricks in Deep Neural Networks (by](https://link.zhihu.com/?target=http%3A//lamda.nju.edu.cn/weixs/project/CNNTricks/CNNTricks.html)  
以及  
[Top 10 Deep Learning Tips and Tricks](https://link.zhihu.com/?target=http%3A//www.ithome.com.tw/video/101624)

这个东东最忌讳的就是纸上谈兵，作为渣渣过一段时间自己总结下跳过的坑。![](https://pic3.zhimg.com/da8e974dc_xs.jpg?source=1940ef5c)知乎用户

还是蛮多的，之前总结过一次，在这里搬运一下，

[DOTA：大道至简：算法工程师炼丹 Trick 手册​zhuanlan.zhihu.com![](https://pic4.zhimg.com/v2-d1182d60fbc886ca288b794440b7c6b3_180x120.jpg)](https://zhuanlan.zhihu.com/p/352971645)

**Focal Loss**
--------------

![](https://pic3.zhimg.com/50/v2-6741afcf4819774e313ae3a2cac2d190_hd.jpg?source=1940ef5c)

针对类别不平衡问题，用预测概率对不同类别的 loss 进行加权。Focal loss 对 CE loss 增加了一个调制系数来降低容易样本的权重值，使得训练过程更加关注困难样本。

```
loss = -np.log(p) 
loss = (1-p)^G * loss

```

Dropout
-------

![](https://pic1.zhimg.com/50/v2-2f3a5e2842073ea699fad5d7b3c5d785_hd.jpg?source=1940ef5c)

随机丢弃，抑制过拟合，提高模型鲁棒性。

**Normalization**
-----------------

Batch Normalization 于 2015 年由 Google 提出，开 Normalization 之先河。其规范化针对单个神经元进行，利用网络训练时一个 mini-batch 的数据来计算该神经元的均值和方差, 因而称为 Batch Normalization。

```
x = (x - x.mean()) / x.std()

```

relu
----

![](https://pic1.zhimg.com/50/v2-c984bb0e4d58d0b7f7a5ba90cf2006cd_hd.jpg?source=1940ef5c)

用极简的方式实现非线性激活，缓解梯度消失。

```
x = max(x, 0)

```

**Cyclic LR**
-------------

![](https://pic3.zhimg.com/50/v2-0a0bb3099d71845a290e481447995c72_hd.jpg?source=1940ef5c)

每隔一段时间重启学习率，这样在单位时间内能收敛到多个局部最小值，可以得到很多个模型做集成。

```
scheduler = lambda x: ((LR_INIT-LR_MIN)/2)*(np.cos(PI*(np.mod(x-1,CYCLE)/(CYCLE)))+1)+LR_MIN

```

**With Flooding**
-----------------

![](https://pic1.zhimg.com/50/v2-dfff1c8958c99433be3d98a19d0dee9f_hd.jpg?source=1940ef5c)

当 training loss 大于一个阈值时，进行正常的梯度下降；当 training loss 低于阈值时，会反过来进行梯度上升，让 training loss 保持在一个阈值附近，让模型持续进行 “random walk”，并期望模型能被优化到一个平坦的损失区域，这样发现 test loss 进行了 double decent。

```
flood = (loss - b).abs() + b

```

Group Normalization
-------------------

![](https://pic3.zhimg.com/50/v2-bc3a9f69253ac87e0950500119d47c87_hd.jpg?source=1940ef5c)

Face book AI research（FAIR）吴育昕 - 恺明联合推出重磅新作 Group Normalization（GN），提出使用 Group Normalization 替代深度学习里程碑式的工作 Batch normalization。一句话概括，Group Normbalization（GN）是一种新的深度学习归一化方式，可以替代 BN。

```
def GroupNorm(x, gamma, beta, G, eps=1e-5):
    # x: input features with shape [N,C,H,W]
    # gamma, beta: scale and offset, with shape [1,C,1,1]
    # G: number of groups for GN
    N, C, H, W = x.shape
    x = tf.reshape(x, [N, G, C // G, H, W])
    mean, var = tf.nn.moments(x, [2, 3, 4], keep dims=True)
    x = (x - mean) / tf.sqrt(var + eps)
    x = tf.reshape(x, [N, C, H, W])
    return x * gamma + beta

```

**Label Smoothing**
-------------------

![](https://pic3.zhimg.com/50/v2-79d513b2b9993223cdc4a53a38d4c2d9_hd.jpg?source=1940ef5c)![](https://pic2.zhimg.com/50/v2-736fe8a9d9f79dd0db990ba3a64a9c8a_hd.jpg?source=1940ef5c)

abel smoothing 将 hard label 转变成 soft label，使网络优化更加平滑。标签平滑是用于深度神经网络（DNN）的有效正则化工具，该工具通过在均匀分布和 hard 标签之间应用加权平均值来生成 soft 标签。它通常用于减少训练 DNN 的过拟合问题并进一步提高分类性能。

```
targets = (1 - label_smooth) * targets + label_smooth / num_classes

```

![](https://pic2.zhimg.com/50/v2-f6614a7035b1823d35aa6a4e14aeeefd_hd.jpg?source=1940ef5c)

Wasserstein GAN
---------------

![](https://pic3.zhimg.com/50/v2-5061e1f191f791e1d3ce8a9093530fd8_hd.jpg?source=1940ef5c)

*   彻底解决 GAN 训练不稳定的问题，不再需要小心平衡生成器和判别器的训练程度
*   基本解决了 Collapse mode 的问题，确保了生成样本的多样性
*   训练过程中终于有一个像交叉熵、准确率这样的数值来指示训练的进程，数值越小代表 GAN 训练得越好，代表生成器产生的图像质量越高
*   不需要精心设计的网络架构，最简单的多层全连接网络就可以做到以上 3 点。

**Skip Connection**
-------------------

一种网络结构，提供恒等映射的能力，保证模型不会因网络变深而退化。

```
F(x) = F(x) + x

```

参考文献
----

*   [https://www.zhihu.com/question/427088601](https://www.zhihu.com/question/427088601)
*   [https://arxiv.org/pdf/1701.07875.pdf](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/1701.07875.pdf)
*   [https://zhuanlan.zhihu.com/p/25071913](https://zhuanlan.zhihu.com/p/25071913)
*   [https://www.zhihu.com/people/yuconan/posts](https://www.zhihu.com/people/yuconan/posts)

![](https://pic3.zhimg.com/50/v2-071e132c7353c2be34deb925be1bf617_hd.jpg?source=1940ef5c)![](https://pic2.zhimg.com/v2-915c1063dc8de2d157d94a2bbce8a616_xs.jpg?source=1940ef5c)永无止境

在噪声较强的时候，可以考虑采用软阈值化作为激活函数：

![](https://pic1.zhimg.com/50/v2-e1b60d5d60dfbb47507a7d93998e26cb_hd.jpg?source=1940ef5c)

软阈值化几乎是降噪的必备步骤，但是阈值τ该怎么设置呢？

阈值τ不能太大，否则所有的输出都是零，就没有意义了。而且，阈值不能为负。

针对这个问题，深度残差收缩网络 [[1]](#ref_1)[[2]](#ref_2) 提供了一个思路，设计了一个特殊的子网络来自动设置：

![](https://pic1.zhimg.com/50/v2-ff5ad94a5326900e453440db1296b9c7_hd.jpg?source=1940ef5c)

Zhao M, Zhong S, Fu X, Tang B, Pecht M. Deep residual shrinkage networks for fault diagnosis. IEEE Transactions on Industrial Informatics. 2019 Sep 26;16(7):4681-90.

参考
--

1.  [^](#ref_1_0) 深度残差收缩网络：从删除冗余特征的灵活度进行探讨 [https://zhuanlan.zhihu.com/p/118493090](https://zhuanlan.zhihu.com/p/118493090)
2.  [^](#ref_2_0) 深度残差收缩网络：一种面向强噪声数据的深度学习方法 [https://zhuanlan.zhihu.com/p/115241985](https://zhuanlan.zhihu.com/p/115241985)

 ![](https://pic2.zhimg.com/da8e974dc_xs.jpg?source=1940ef5c) 知乎用户

世上本没有 trick, 只是用了别人的 (任何类型的) 好方法 却不说的人多了，也就有了 trick

YOLOv4: 我对比了一堆 trick

![](https://pic2.zhimg.com/50/v2-b29a81408afac498a82db8147f491f40_hd.jpg?source=1940ef5c)![](https://pic1.zhimg.com/50/v2-011d9079da6086ee0d003f1bb9e36223_hd.jpg?source=1940ef5c)![](https://pic1.zhimg.com/50/v2-eb0d67213a9c3ed94bfad837eb4ee6e5_hd.jpg?source=1940ef5c)

mish 比 swish 好

![](https://pic2.zhimg.com/50/v2-2cda8124ffef750f4f767d388aa56594_hd.jpg?source=1940ef5c)![](https://pic3.zhimg.com/50/v2-d337d799593883cecf60e81b05c10156_hd.jpg?source=1940ef5c)![](https://pic2.zhimg.com/50/v2-a5eb1797c49044e7750ac8b98a3a6982_hd.jpg?source=1940ef5c)![](https://pic2.zhimg.com/50/v2-8389605bb004719c4b3b93a3f0244270_hd.jpg?source=1940ef5c) ![](https://pic1.zhimg.com/da8e974dc_xs.jpg?source=1940ef5c) 冯迁

赞比较多的给了些，损失函数 (focal)，模型结构（identity skip)，训练方面（strategy on learning rate), 稳定性方面（normalization）, 复杂泛化性（drop out），宜优化性（relu,smooth label) 等，这些都可以扩展。

focal 可以扩展到 centernet loss，结构有尺度 fpn，重复模块，堆叠 concatenate，splite，cross fusion 等，训练方面有 teaching，step learning，对抗（本身是个思想），多阶段优化，progress learning，稳定性方面，batch normal，instance normal，group normal 之外，还有谱范数等，复杂性还有正则 l1/2 等，宜优化性，可以扩展到检测的 anchor 等。

dl 你得说优化器吧，把动量，一二阶考虑进来，梯度方向和一阶动量的折中方向，把随机考虑进来 sgd

以上可能带来最多 10 的收益，你得搞数据啊

数据方面的处理 clean，various ，distribution,aug data 等更重要 (逃…

“理论” 在收敛速度，稳定，泛化，通用，合理等方面着手，性能在数据方面着手，也许 ![](https://pic1.zhimg.com/63d59eaa1702dc7e36a24766eea32cab_xs.jpg?source=1940ef5c) DLing

说到深度学习的 trick，大家第一反应都是各种网络结构上的模块（attention，BN，各种激活函数等），各种数据预处理函数，各种损失函数，各种训练超参的调整这些。这些技巧当然可以提升网络训练的效果，但是今天我这边另辟蹊径，从快速有效的处理添加与清洗数据的角度，给大家说一说，怎么快速的，高效的，节省人工成本的得到一个高质量数据集，进而得到一个好模型。

众所周知，我们目前的深度学习任务，第一靠数据，后面才是模型结构，但是有数据和有一份高质量的数据集却不是一回事。我们日常工作中，因数据集不干净，数据集数据量不够，数据集里场景范围单一导致的模型性能不好的例子比比皆是，而得到一个数据干净，数据量庞大，数据场景多样的数据集却需要耗费大量的人力与时间成本。

今天，给大家提供一种通过多模型融合的方式去预处理与清洗数据的方法，这个方法可以节省人工标注数据的时间；提升标注质量；让每次添加的数据都更有针对性，减少无用数据的累积。大量实验证明，这种方法在分类任务，重识别任务中效果很好，在检测任务中也可以做相关迁移，具体思路，步骤和代码可见我的这篇文章：

[https://zhuanlan.zhihu.com/p/359960152​zhuanlan.zhihu.com![](https://pic4.zhimg.com/v2-436d51f4746d17e75e8ef416fb67d713_ipico.jpg)](https://zhuanlan.zhihu.com/p/359960152) ![](https://pic3.zhimg.com/da8e974dc_xs.jpg?source=1940ef5c) 知乎用户 caffe 有一个参数 rand_skip  
用了和不用，差别是非常明显的，算是一个 trick 吧![](https://pic1.zhimg.com/v2-66bfc499d6cd8fcd29f647d987186c8f_xs.jpg?source=1940ef5c)白皮书

参数初始化的方法，总结，希望对你有用 [http://blog.csdn.net/BVL10101111/article/details/70787683](https://link.zhihu.com/?target=http%3A//blog.csdn.net/BVL10101111/article/details/70787683)

最近总结了一下 Deep Learning 的最优化方法，包括 Adam,RMSProp,SGD,Momentum，adaGrad, 以及一些传统的二阶优化方法等等，欢迎一起交流  
博文地址:  
[http://blog.csdn.net/BVL10101111/article/details/72614711](https://link.zhihu.com/?target=http%3A//blog.csdn.net/BVL10101111/article/details/72614711) ![](https://pic3.zhimg.com/aadd7b895_xs.jpg?source=1940ef5c) 匿名用户个人觉得主要有参数初始化，学习率改变，激活函数的选择这些 trick, 参数初始化非常坑，自己以前在上面栽过，学习率更改方法的选择和激活函数的选择很重要，会直接影响到你收敛的速度，最后网络的结构也很重要。![](https://pic3.zhimg.com/v2-c378e7ae5442bea74d06cd532f335844_xs.jpg?source=1940ef5c)CJEQ 看 10 遍 trick 不如踩一遍坑