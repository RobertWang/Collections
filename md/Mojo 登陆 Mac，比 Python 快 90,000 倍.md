> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652601185&idx=1&sn=60c0c44e93187add9fe6ec9caceb8ee8&chksm=8465532bb312da3d0b1585ff41d7f5a00ad8f2aaba09e163ef5cab8c0a898ad1dcbc53c3fed2&scene=21#wechat_redirect)

> 转自：_OSC 开源社区（ID：oschina2013)_

AI 编程语言 Mojo🔥 推出了支持 Mac 平台的版本，其创始人 Chris Lattner 称 Mojo + Apple Silicon 是强强联合，强上加强。

下载地址：https://developer.modular.com/download

开发者需要先注册 Modular 账号，然后通过 Homebrew 包管理器下载 Modular CLI，接着运行 module install mojo 命令来安装 Mojo。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dkwuWwLoRKicyRF4OkVQKGicqHNNGAqic7pTQ2icBqS7TEYwNNmN4hMw8zx6icibMnHDiaB5SHQ7LjgmxPhehutgHJoXw/640?wx_fmt=png)

下面是运行 "Hello Mojo" 的示例截图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dkwuWwLoRKicyRF4OkVQKGicqHNNGAqic7p6jJ1n1I8RCsPxicLZkGAL8mJ3LkbPECtZWmpJAMYicwqVFzwV3GiatnvQ/640?wx_fmt=png)

此外，Mojo SDK 还提供了 Visual Studio Code 扩展。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dkwuWwLoRKicyRF4OkVQKGicqHNNGAqic7p3oxthLic6hsfamUPE7qpG6We9v66lglicPO2VXXG808gZhyJLjsXUhsA/640?wx_fmt=png)

Mojo 团队介绍称，Mojo 语言可以充分利用 Mac CPU 内核和矢量单元来实现加速。根据他们提供的测试 —— 使用 matmul.mojo 运行一个矩阵乘法示例。在 Apple MacBook Pro M2 Max 上，与纯 Python 实现的版本相比，Mojo 的速度大约比 Python 快 90,000 倍。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dkwuWwLoRKicyRF4OkVQKGicqHNNGAqic7pS0MmEUsNemva2XqJr9eNqeaHqJfuuszgh4ywpHJyaRMRgreMYNtFqg/640?wx_fmt=png)

开源开发者 Aydyn Tairov 上个月将 `llama2.py` 移植到 Mojo——`llama2.mojo`。当时提供的是 Linux 版本，结果非常出乎意料 —— 他表示 Mojo SIMD 原语帮助将 Python 的糟糕性能提升了近 250 倍。

现在 Mojo for Mac 发布后，Aydyn Tairov 称 `llama2.mojo`在 Mac 上的性能与 `llama2.cpp`不相上下。在许多情况下甚至优于 plain C。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dkwuWwLoRKicyRF4OkVQKGicqHNNGAqic7pReqfDggEEGpwmBM0jQY4AqakfzUYfehAicS6DUt7CMIxnGQIL0ReeKw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dkwuWwLoRKicyRF4OkVQKGicqHNNGAqic7pZxKIaXibZnGCyf6qaeg66OGWqCA6D0ZhP7QK9EVs77csQ2sLEfAMZbg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dkwuWwLoRKicyRF4OkVQKGicqHNNGAqic7pK054yufM0lkl3Z6aHf4T5pwrABlOQ1sAvPDogzP1ncfv7yON2UJw5g/640?wx_fmt=png)

相关链接：

*   https://www.modular.com/blog/mojo-is-now-available-on-mac
    
*   https://twitter.com/tairov/status/1714103695321829551
    
*   https://twitter.com/clattner_llvm/status/1715038783832526874