> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MDExMDEyMw==&mid=2247498338&idx=2&sn=bbe1d9d2739dbf26a7b7393f0db0a5be&chksm=9fab8d26a8dc043030f54e226ad1b34335284bdad5757550668b3862861eaa195b57eaf5f994&mpshare=1&scene=1&srcid=0912vI4Mz3l4GTd9VQ2jXf1l&sharer_sharetime=1662963444074&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

众所周知，Python 的简单和易读性是**靠牺牲性能**为代价的——

尤其是在计算密集的情况下，比如多重 for 循环。

不过现在，大佬**胡渊鸣**说了：

> 只需 import 一个叫做 “Taichi” 的库，就可以把代码速度**提升 100 倍**！

不信？

来看三个例子。

计算素数的个数，速度 x120
---------------

第一个例子非常非常简单，求所有小于给定正整数 N 的素数。

标准答案如下：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2T6haO3kPkMQX6kuQ1UmvrodrQiaj7niasURut8gSPNWqGcicHRwZfM1ic2w/640?wx_fmt=png)

我们将上面的代码保存，运行。

当 N 为 100 万时，需要 2.235s 得到结果：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TA7uRAY0Dql9ib1nOozKrIwG73RnFiaKA3nHibMjVaZE8LhFFcXWwCf9nw/640?wx_fmt=png)

现在，我们开始施魔法。

不用更改任何函数体，import“taichi” 库，然后再加两个装饰器：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TdINPQPicfIiap2n6qOAMJtvv4SFYZXGTVf6slCfQm6AKRy6zUXAllR7A/640?wx_fmt=png)

Bingo！**同样的结果只要 0.363s，快了将近 6 倍。**

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TFavtBTwcy7eIahOeicCfA1Vx730nqMS0dFLj1p8RjjsJ136iaElLiaxPw/640?wx_fmt=png)

如果 N=1000 万，则只要 0.8s；要知道，不加它可是 55s，一下子又快了 **70 倍**！

不止如此，我们还可以在 ti.init() 中加个参数变为 ti.init(arch=ti.gpu) ，让 taich 在 **GPU** 上进行计算。

那么此时，计算所有小于 1000 万的素数就只耗时 0.45s 了，与原来的 Python 代码相比速度就**提高了 120 倍**！

厉不厉害？

什么？你觉得这个例子太简单了，说服力不够？我们再来看一个稍微复杂一点的。

动态规划，速度 x500
------------

动态规划不用多说，作为一种优化算法，通过动态存储中间计算结果来减少计算时间。

我们以经典教材《算法导论》中的经典动态规划案例 **“最长公共子序列问题（LCS）”** 为例。

比如对于序列 a = [0, 1, 0, 2, 4, 3, 1, 2, 1] 和序列 b = [4, 0, 1, 4, 5, 3, 1, 2]，它们的 LCS 就是：

> LCS(a, b) = [0, 1, 4, 3, 1, 2]。

用动态规划的思路计算 LCS，就是先求解序列 a 的前 i 个元素和序列 b 的前 j 个元素的最长公共子序列的长度，然后逐步增加 i 或 j 的值，重复过程，得到结果。

我们用 f[i, j] 来指代这个子序列的长度，即 LCS((prefix(a, i), prefix(b, j)。其中 prefix(a, i) 表示序列 a 的前 i 个元素，即 a[0], a[1], …, a[i - 1]，得到如下递归关系：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TqqscuUiaNUugbofrq6tkvBibicTlfGkfeQjvpTtqs13Wr43I2eTvkLAlA/640?wx_fmt=png)

完整代码如下：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TDGFJzCOBnzsyx1pecGlVyQK7Daiayq3KgQHvsN7UaQST9VlmLBYuaoA/640?wx_fmt=png)

现在，我们用 Taichi 来加速：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TZtNtDoeUYD3HdQy28Tdw5bzyDoCKWPCv0zn1Q12ia6eu9YpL9lWIGGA/640?wx_fmt=png)

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TfLZ5jUtFtyuFjRm9rYkmMSkdlkKpyBFcwPeln3F25ztjjVADQxtPRg/640?wx_fmt=png)

胡渊鸣电脑上的程序**最快做到了 0.9 秒内**完成，而**换成用 NumPy 来实现，则需要 476 秒，差异达到了超 500 倍！**

最后，我们再来一个不一样的例子。

反应 - 扩散方程，效果惊人
--------------

自然界中，总有一些动物身上长着一些看起来无序但实则并非完全随机的花纹。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TPDvU1Elkpthcu8W7R46YuCaR364TU6sZT0NRKZjnWPQcIFdemsSPmg/640?wx_fmt=png)

图灵机的发明者艾伦 · 图灵是第一个提出模型来描述这种现象的人。

在该模型中，两种化学物质（U 和 V）来模拟图案的生成。这两者之间的关系类似于猎物和捕食者，它们自行移动并有交互：

1.  最初，U 和 V 随机分布在一个域上；
    
2.  在每个时间步，它们逐渐扩散到邻近空间；
    
3.  当 U 和 V 相遇时，一部分 U 被 V 吞噬。因此，V 的浓度增加；
    
4.  为了避免 U 被 V 根除，我们在每个时间步添加一定百分比 (f) 的 U 并删除一定百分比 (k) 的 V。
    

上面这个过程被概述为 “反应 - 扩散方程”：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TLJakmSicRIctE1zV1aXRgfqMuuHuiaPlaAxGeMg6GS7bBxY0KIp1qmdA/640?wx_fmt=png)  

其中有四个关键参数：Du（U 的扩散速度），Dv（V 的扩散速度），f（feed 的缩写，控制 U 的加入）和 k（kill 的缩写，控制 V 的去除）。

如果 Taichi 中实现这个方程，首先创建网格来表示域，用 vec2 表示每个网格中 U, V 的浓度值。

拉普拉斯算子数值的计算需要访问相邻网格。为了避免在同一循环中更新和读取数据，我们应该创建两个形状相同的网格 W×H×2。

每次从一个网格访问数据时，我们将更新的数据写入另一个网格，然后切换下一个网格。那么数据结构设计就是这样：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TQMNNTiaIDcVH3kSKmt5bpuoXVKHgniaVJhRP9KLQjGmHlBOr7BkgTzMA/640?wx_fmt=png)

一开始，我们将 U 在网格中的浓度设置为 1，并将 V 放置在 50 个随机选择的位置：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDTI85uUotpAT854bauicq2TTAbiaKRgC6S8aWx1jW2tj0z7o1Y2nEm3q8VAOM8OKfhsG3GGvn5e9nw/640?wx_fmt=png)

那么实际计算就可以用不到 10 行代码完成：

```
@ti.kernel
def compute(phase: int):
    for i, j in ti.ndrange(W, H):
        cen = uv[phase, i, j]
        lapl = uv[phase, i + 1, j] + uv[phase, i, j + 1] + uv[phase, i - 1, j] + uv[phase, i, j - 1] - 4.0 * cen
        du = Du * lapl[0] - cen[0] * cen[1] * cen[1] + feed * (1 - cen[0])
        dv = Dv * lapl[1] + cen[0] * cen[1] * cen[1] - (feed + kill) * cen[1]
        val = cen + 0.5 * tm.vec2(du, dv)
        uv[1 - phase, i, j] = val

```

在这里，我们使用整数相位（0 或 1）来控制我们从哪个网格读取数据。

最后一步就是根据 V 的浓度对结果进行染色，就可以得到这样一个**效果惊人的图案**：‍

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtDTI85uUotpAT854bauicq2TCZhxmRY8OtDIHansX9BhYtIzWW6gYWLY9mViak25oRMaGZ35ycsnBiaw/640?wx_fmt=gif)

‍有趣的是，胡渊鸣介绍，即使 V 的初始浓度是随机设置的，但每次都可以得到相似的结果。

而且和只能达到 30fps 左右的 Numba 实现比起来，Taichi 实现由于可以选择 GPU 作为后端，轻松超过了 300fps。

pip install 即可安装
----------------

看完上面三个例子，你这下相信了吧？

其实，Taichi 就是一个嵌入在 Python 中的 DSL（动态脚本语言），它通过自己的编译器将被 @ti.kernel 装饰的函数编译到各种硬件上，包括 CPU 和 GPU，然后进行高性能计算。

有了它，你无需再羡慕 C++/CUDA 的性能。

正如其名，Taichi 就出自太极图形胡渊鸣的团队，现在你只需要用 pip install 就能安装这个库，并与其他 Python 库进行交互，包括 NumPy、Matplotlib 和 PyTorch 等等。

当然，Taichi 用起来和这些库以及其他加速方法有什么差别，胡渊鸣也给出了详细的优缺点对比，感兴趣的朋友可以戳下面的链接详细查看：

https://docs.taichi-lang.org/blog/accelerate-python-code-100x