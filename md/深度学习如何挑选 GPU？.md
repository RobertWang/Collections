> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0NjgzMDIxMQ==&mid=2247590843&idx=1&sn=a216d4b45758144e8f2051c30f6bfcba&chksm=fb548157cc23084176ce6b35ea9e13027c7965e453cb98f70c57eba37ea7679eb2705229b5c3&mpshare=1&scene=1&srcid=031447DqXCTm1xyAEKIBY5JV&sharer_sharetime=1647233040651&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**1 是什么使一个 GPU 比另一个 GPU 更快？**






=====================================

有一些可靠的性能指标可以作为人们的经验判断。以下是针对不同深度学习架构的一些优先准则：

**Convolutional networks and Transformers:** Tensor Cores > FLOPs > Memory Bandwidth > 16-bit capability

**Recurrent networks:** Memory Bandwidth > 16-bit capability > Tensor Cores > FLOPs

**2 如何选择 NVIDIA/AMD/Google**






====================================

NVIDIA 的标准库使在 CUDA 中建立第一个深度学习库变得非常容易。早期的优势加上 NVIDIA 强大的社区支持意味着如果使用 NVIDIA GPU，则在出现问题时可以轻松得到支持。但是 NVIDIA 现在政策使得只有 Tesla GPU 能在数据中心使用 CUDA，而 GTX 或 RTX 则不允许，而 Tesla 与 GTX 和 RTX 相比并没有真正的优势，价格却高达 10 倍。

AMD 功能强大，但缺少足够的支持。AMD GPU 具有 16 位计算能力，但是跟 NVIDIA GPU 的 Tensor 内核相比仍然有差距。

Google TPU 具备很高的成本效益。由于 TPU 具有复杂的并行基础结构，因此如果使用多个云 TPU（相当于 4 个 GPU），TPU 将比 GPU 具有更大的速度优势。因此，就目前来看，TPU 更适合用于训练卷积神经网络。

**3 多 GPU 并行加速**






========================

卷积网络和循环网络非常容易并行，尤其是在仅使用一台计算机或 4 个 GPU 的情况下。TensorFlow 和 PyTorch 也都非常适合并行递归。但是，包括 transformer 在内的全连接网络通常在数据并行性方面性能较差，因此需要更高级的算法来加速。如果在多个 GPU 上运行，应该先尝试在 1 个 GPU 上运行，比较两者速度。由于单个 GPU 几乎可以完成所有任务，因此，**在购买多个 GPU 时，更好的并行性（如 PCIe 通道数）的质量并不是那么重要**。

**4 性能评测**






==================

**1）来自 Tim Dettmers 的成本效益评测** **[1]**

https://timdettmers.com/2019/04/03/which-gpu-for-deep-learning/

![](https://mmbiz.qpic.cn/sz_mmbiz_png/gYUsOT36vfrPojI3DARSY7XYEBa8I928zpDJB8SFlRp1eebMgAM7y8KsjwNdoiatvdFiarympWqUzTYkh8Y56ZVQ/640?wx_fmt=png)

卷积网络（CNN），递归网络（RNN）和 transformer 的归一化性能 / 成本数（越高越好）。RTX 2060 的成本效率是 Tesla V100 的 5 倍以上。对于长度小于 100 的短序列，Word RNN 表示 biLSTM。使用 PyTorch 1.0.1 和 CUDA 10 进行基准测试。

从这些数据可以看出，RTX 2060 比 RTX 2070，RTX 2080 或 RTX 2080 Ti 具有更高的成本效益。原因是使用 Tensor Cores 进行 16 位计算的能力比仅仅拥有更多 Tensor Cores 内核要有价值得多。

**2）来自 Lambda 的评测 **[2,3]****

https://lambdalabs.com/blog/best-gpu-tensorflow-2080-ti-vs-v100-vs-titan-v-vs-1080-ti-benchmark/

https://lambdalabs.com/blog/choosing-a-gpu-for-deep-learning/

![](https://mmbiz.qpic.cn/sz_mmbiz_png/gYUsOT36vfrPojI3DARSY7XYEBa8I928mDvJbJiaeTiayvmbkYOWic7rEGicYfLpbCwgW785l5EBicHbHNRuj9to37w/640?wx_fmt=png)

GPU 平均加速 / 系统总成本  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/gYUsOT36vfrPojI3DARSY7XYEBa8I9286jbnc7reoDc1cPPUn7vH3k0u1oRsiaTYLeBOKEzibJGxCXJdGX4nDVzg/640?wx_fmt=png)

GPU 性能，以每秒处理的图像为单位

![](https://mmbiz.qpic.cn/sz_mmbiz_png/gYUsOT36vfrPojI3DARSY7XYEBa8I92889YHWS5Zg0dISTXZ3QVC4AxjtRMIibbicn8xtdMPMkg1Dj5cwoUwtOBQ/640?wx_fmt=png)

 以 Quadro RTX 8000 为基准的针对 Quadro RTX 8000 的图像模型训练吞吐量

**3) 来自知乎 @Aero 的「在线」GPU 评测 **[4]****

https://www.zhihu.com/question/299434830/answer/1010987691

大家用的最多的可能是 **Google Colab**，毕竟免费，甚至能选 TPU

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrPojI3DARSY7XYEBa8I928rydVmcZJeqJ0jXicqO81cz5VicTuh4M4siaEkNmXIiaKIbxJPiazmuOdRFQ/640?wx_fmt=jpeg)

不过现在出会员了：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrPojI3DARSY7XYEBa8I928xRJibUV0vYfx06rHDDwoiaVuLh58KjwfvLSFKlZjQZ6icu6bQmGz8SJzQ/640?wx_fmt=jpeg)

免费版主要是 K80，有点弱，可以跑比较简单的模型，有概率分到 T4，有欧皇能分到 P100。

付费就能确保是 T4 或者 P100，一个月 10 美元，说是仅限美国。

Colab 毕竟是 Google 的，那么你首先要能连得上 google，并且得网络稳定，要是掉线很可能要重新训练，综合来看国内使用体验不太好。

下一个是**百度 AI Studio**：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrPojI3DARSY7XYEBa8I928OqKBlNGVYVS68Y1ibgxB3hibeDXjMViazxTO6IpwzJbUicA4yTk3xEe9nw/640?wx_fmt=jpeg)

免费送 V100 时长非常良心，以前很多人自己装 tensorflow 用，但是现在已经不允许了，实测 tensorflow pytorch 都不给装，必须得用 paddlepaddle。那么习惯 paddlepaddle 的用户完全可以选这个，其他人不适合。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrPojI3DARSY7XYEBa8I928GvyBajsqFKPIqY1QmM9NDPLk0nGdck2AjM8rB9fdyiajuRBVZeNRylw/640?wx_fmt=jpeg)

不过似乎 GPU 不太够，白天一直提醒高峰期，真到了 22 点后才有。

国外的还有 vast.ai：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/gYUsOT36vfrPojI3DARSY7XYEBa8I928ULyP8BagvxtQ2mPwoQmJ7BekEg086ibPBc7eib5f0ES7nmh4icFMtB8TA/640?wx_fmt=jpeg)

**5 建议**






================

1）来自 Tim Dettmers 的建议

*   总体最佳 GPU：RTX 2070 GPU  
    
*   避免使用 ：任何 Tesla；任何 Quadro；任何 Founders Edition；Titan RTX，Titan V，Titan XP  
    
*   高效但价格昂贵：RTX 2070  
    
*   高效且廉价：RTX 2060，GTX 1060（6GB）  
    
*   价格实惠：GTX 1060（6GB）  
    
*   价格低廉：GTX 1050 Ti（4GB）。或者：CPU（原型设计）+ AWS / TPU（培训）；或 Colab。  
    
*   适合 Kaggle 比赛：RTX 2070  
    
*   适合计算机视觉研究人员：GTX 2080 Ti，如果训练非常大的网络，建议使用 RTX Titans
    

2）来自 Lambda 的建议

截至 2020 年 2 月，以下 GPU 可以训练所有 SOTA 语言和图像模型：

*   RTX 8000：48 GB VRAM
    
*   RTX 6000：24 GB VRAM
    
*   Titan RTX：24 GB VRAM
    

具体建议：

*   RTX 2060（6 GB）：适合业余时间探索深度学习。
    
*   RTX 2070 或 2080（8 GB）：适合深度学习专业研究者，且预算为 4-6k
    
*   RTX 2080 Ti（11 GB）：适合深度学习专业研究者，而您的 GPU 预算约为 8-9k。RTX 2080 Ti 比 RTX 2080 快 40％。
    
*   Titan RTX 和 Quadro RTX 6000（24 GB）：适合广泛使用 SOTA 型号，但没有用于 RTX 8000 足够预算的研究者。
    
*   Quadro RTX 8000（48 GB）：价格相对较高，但性能卓越，适合未来投资。
    

参考文献

[1] https://timdettmers.com/2019/04/03/which-gpu-for-deep-learning/

[2] https://lambdalabs.com/blog/best-gpu-tensorflow-2080-ti-vs-v100-vs-titan-v-vs-1080-ti-benchmark/

[3] https://lambdalabs.com/blog/choosing-a-gpu-for-deep-learning/

[4] https://www.zhihu.com/question/299434830/answer/1010987691