> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aEjcLtfv0d__tpcPrvqQiQ)

**导读** 

本文介绍了多个能将深度学习训练过程进行可视化的工具，帮助大家更好地理解深度学习，非常实用。

![](https://mmbiz.qpic.cn/mmbiz_png/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnX3ib3vLLTJQ7uWsn4vbMiaEhY2KwiaiaicpM7QbicaibOLz7arVibvn4zLomW4g/640?wx_fmt=png&random=0.24678089576884044)

1. 深度学习网络结构画图工具

地址：

https://cbovar.github.io/ConvNetDraw/

![](https://mmbiz.qpic.cn/mmbiz_png/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnX0oT52ibCbzfLHM98qEc1AzqTRIF4V0AwXjkcp3lpPCsabfPCr37egiag/640?wx_fmt=png&random=0.41440128798701137)

2.caffe 可视化工具

输入：caffe 配置文件 输出：网络结构

地址：

http://ethereon.github.io/netscope/#/editor

![](https://mmbiz.qpic.cn/mmbiz_png/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXypBMF17nmibpBWNCsLYbP8MmCqmYKlTZMibKDSJos2vRwhibkQQhlkZHw/640?wx_fmt=png&random=0.7570679881394264)

3. 深度学习可视化工具 Visual DL

Visual DL 是百度开发的，基于 echar 和 PaddlePaddle，支持 PaddlePaddle，PyTorch 和 MXNet 等主流框架。ps：这个是我最喜欢的，毕竟 echar 的渲染能力不错哈哈哈，可惜不支持 caffe 和 tensorflow。

地址：

https://github.com/PaddlePaddle/VisualDL

4. 结构可视化工具 PlotNeuralNet

萨尔大学计算机科学专业的一个学生开发。

地址：

https://github.com/HarisIqbal88/PlotNeuralNet

其实还有很多可视化工具，但是今天我要说的是，训练过程的可视化，与 TF 的可视化类似，但是**这个操作**更加简便！

![](https://mmbiz.qpic.cn/mmbiz_png/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXibpnXeiahKia2jBbicjGdh2Wfk6lFrNGLaic7icvDPlfnuUqQRvRiabgmMc7g/640?wx_fmt=png&random=0.8172282552924195)

**这个工具到底把训练过程展示得多么详细？**简单来说，项目作者已经给你做好了一个可以交互的界面，你只需要打开浏览器加载出这个界面就可以了。**CNN Explainer 使用 TensorFlow.js 加载预训练模型进行可视化效果，交互方面则使用 Svelte 作为框架并使用 D3.js 进行可视化。**最终的成品即使对于完全不懂的新手来说，也没有使用门槛。下面我们来看一下具体的效果。

![](https://mmbiz.qpic.cn/mmbiz_gif/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXrtaibp2ibWKgnKib2kcqZrLeBfZtnDY8whLDCNvPFMXCDvnSFVHic0ke8g/640?wx_fmt=gif&random=0.44467286820320817)

卷积

![](https://mmbiz.qpic.cn/mmbiz_gif/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXeqRvZK4gTCaibHAiaMB9gU3JRCroZF81PBvnp1ic6BzJPicbMsWyv6wABA/640?wx_fmt=gif&random=0.8463090026128743)

![](https://mmbiz.qpic.cn/mmbiz_png/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXmJr29Xvt548g85TtOltMQ4BzIaQXia3Qh7ly3xNFVB1qdK7ZW3paibrw/640?wx_fmt=png&random=0.776526741558885)

![](https://mmbiz.qpic.cn/mmbiz_gif/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXjnjVsp9OmkeALbh32HhiaT9CppEIjrcPIB64GBPcr7EHiblVfdaib6VZw/640?wx_fmt=gif&random=0.7125190086852233)

超参数

![](https://mmbiz.qpic.cn/mmbiz_png/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXBAtibsGZdiaNyUkMxJbTX2WavuRCu1w5PYicrZMBeM5IrLUouWOacgsGw/640?wx_fmt=png&random=0.32651962490333863)

softmax

![](https://mmbiz.qpic.cn/mmbiz_png/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXZOyNolz29zObDdkicydXWDwm0JBicQWibBInJYJOrxel5MibeGpziaCM3UQ/640?wx_fmt=png&random=0.09683537354281668)

![](https://mmbiz.qpic.cn/mmbiz_gif/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXzVibTl3rnnExdmlWdTqWjMUWMHjtMbCG1w9B53aDSP3UvZ4ceD6WSwQ/640?wx_fmt=gif&random=0.17294042142940813)

ReLU

![](https://mmbiz.qpic.cn/mmbiz_png/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnX4sIx9ra1kcFIQ2MnkIqRxGVbeqUIKFtIRDevlicvpdLkdz5F7v0OGPg/640?wx_fmt=png&random=0.8598823438783587)

MaxPool

![](https://mmbiz.qpic.cn/mmbiz_png/1MtnAxmWSwMiaFicFzOiaDpOszQtIlfMTnXjCEnCWoa5D0LUoO5WNpF9Ce8LylP9AVLRsAKyU2ibyOhURxaEuPlKicw/640?wx_fmt=png&random=0.9843901453503676)