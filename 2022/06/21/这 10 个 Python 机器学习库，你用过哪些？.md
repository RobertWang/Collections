> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2MjE3OTA1MA==&mid=2247496707&idx=1&sn=2c81dcd8d882297006816f9e080a45f1&chksm=ea4da586dd3a2c9029f32074f38a61ff61f6d67e57335a5eb09f0207d39b2a08f73a3f87ad6d&mpshare=1&scene=1&srcid=0620RnUSkeDpof9gnhFem5DK&sharer_sharetime=1655708730645&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

##### 来源：量子位

1. Awkward Array
----------------

根据官方介绍，**Awkward Array** 用于嵌套的、大小不一的数据，包括**任意长度**的列表、记录、混合的类型和缺失数据，使用起来类似 **NumPy**。

看看是不是可以安排上了～

https://pypi.org/project/awkward/

2. Jupytext
-----------

相信大家对 Jupyter Notebook 都不陌生。

当你有了 **Jupytext** 这个小插件就可以将 Jupyter Notebook 和 **IDE** 完美结合，听起来是不是很棒！

从此 Jupyter Notebook 可以被存储为 **Markdown** 文件或多种语言的**脚本**文件。

Jupytext 可以做的事主要有：

*   Jupyter Notebook 的版本控制
    
*   在你喜欢的文本编辑器中编辑、合并或重构 Notebook
    
*   在 Notebook 上使用 Q&A 检查
    

在 Python 中使用的样子：

3. Gradio
---------

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdujfvAaVyIrw1tfsrsq6dqL9JxuXzxGXfUGj6R25Py7X1QO9ewToMmw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdvFPw5PDLeLdgGHF5K99HA91haycpibMhwlzuT6aeESDczJgHjTpUycQ/640?wx_fmt=gif)

只要将 launch() 函数中的参数设置为 share=True，还能得到一个**可分享**的**网址**，拿到链接的朋友在电脑和手机端都能打开，活脱脱就是一个**小程序**。

时常需要做 **Demo** 的小伙伴快看起来吧，此项目在 Github 上已有 4.5k+star。

https://github.com/gradio-app/gradio

4. Hub
------

这个 **Hub** 在数据管理和数据预处理上可是一把好手。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdCL8xgXMOoZlkGhYjHfrYibglrN5jVEsdvLgoas3ibuCrXLaPcRM7O6Ug/640?wx_fmt=png)

它可以处理**任何类型**，**任何大小**的数据，并且因为数据储存在云端上，所以可以无缝在**任何机器**上访问。

被压缩为二进制字节的数据可以被存储在任何地方，并且只有在需要的时候才会被获取，所以没有 TB 级硬盘也可以处理 **TB 级数据**。

Hub 贴心地提供了重要 **API**，支持数据在常用工具（PyTorch 等）上的使用，数据版本控制，数据转换等功能。

此项目在 github 上已有 4.1k+star。

https://github.com/activeloopai/Hub

5. AugLy
--------

**AugLy** 是 facebook 最新推出的数据增强库，同时支持**语音**，**文本**，**图像**和**视频**类型的数据，包含了 **100 多种**增强方式。

数据对于模型训练至关重要，而标注大规模数据十分困难。由于人力资源，和模型特性的限制，数据增强的应用越来越广泛。

AugLy 的**优点**：

*   处理类型更为**全面**。其他的数据增强库，例如 Albumentations 和 NVIDIA DALI，主要负责图像相关数据的处理，文字数据不支持。
    
*   处理方式十分**人性化**。AugLy 可以将一张图片做成备忘录，在图片 / 视频上叠加文字 / Emojis，转发社交媒体上的截图，还可以帮助你处理诸如拷贝检测、仇恨言论检测或版权侵权等问题。
    

此项目在 Github 上已有 4.1k+star。

https://github.com/facebookresearch/AugLy

6. Evidently
------------

**Evidently** 是用来监测模型效果的工具，可从 Pandas DataFrame 或 csv 文件中生成交互式**可视化报告**和 **JSON 格式**的**效果简介**。在 Jupyter Notebook 中可以使用。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdcbiacEzgNVNibILkguTt4mfkuVcahLsIgiaBXdicCBhvXzY9Tnfvm3rMsQ/640?wx_fmt=png)

目前可以提供 **6 种**报告：数据漂移、数值目标漂移、分类目标漂移、回归模型性能、分类模型性能和概率分类模型性能。

此项目在 Github 上已有 1.8k+star。

https://github.com/evidentlyai/evidently

7. YOLOX
--------

如果你熟悉 YOLO 的话，那你或许会对**旷视**今年推出的 **YOLOX** 感兴趣。

YOLO 就是那个目标检测算法，可以被使用在汽车**自动驾驶**等前沿技术中。

而 **YOLOX** 是 YOLO 的无锚版本，设计更简单，但性能更好！它的目标是在研究界和工业界之间架起一座桥梁，同时弥合两方之间的差距。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMd54j8hlrMrmxnag6z9qDg6ccYxo01kDOGGNJe6OAeiaibH8k9BWXO9jvQ/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdaasNQ7ibnmBw7Sia1lINLURCh3Yx6B8ic1MjUG9VaOk2gYNpQUxsOGAicQ/640?wx_fmt=png)

这个 Github 上的开源项目在短短半年内已获得 5.2k+star。

https://github.com/Megvii-BaseDetection/YOLOX

8. LightSeq
-----------

正如它的名字一样，**LightSeq** 是一款由字节跳动开发的支持 BERT、GPT、Transformer 等众多模型的**超快**推理引擎。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdeRn0KqfaA690vhU7ibHZ6jgGAYSSFDdbgyM1kNHy6q2jjNM98cm5F5A/640?wx_fmt=png)

可以看到它的表现，比 FasterTransformer 还要 **Fast**。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdaF1dEhz2vnBAdV6FVUqfex6icXYTD3fHPIYbuicazvatLIy2vtFll7mw/640?wx_fmt=png)

LightSeq 支持的模型也是非常**全面**。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdFWudiaicWic0dbicic2cH2tdwPHt8QkJW8rP6OwSNrDPuWDY0gxUIkybPXg/640?wx_fmt=png)

总之就是两个字 “好用”。此项目在 Github 上已有 1.9k+star。

https://github.com/bytedance/lightseq

9. Greykite
-----------

想预测 **COVID-19** 的恢复速度吗？那就来看看 LinkedIn 为了自家时间序列预测需求开发的 **Greykite** 吧。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdsW5C2mYcUVRmeMXUu9EwhrbZMlibczY30lqTlKTlH5QmTuiboOLoar2g/640?wx_fmt=png)

功能全面（多种时间趋势），界面直观，预测速度快和可扩展性强是它最大的亮点。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdaG5AylFTJaeTJugPJRDRs7wYFomDAGSCV8iaUS9sVqeABeINcXa3j8g/640?wx_fmt=png)

被应用在上面的三大算法：

*   Silverkite (Greykite’s flagship algorithm)  
    
*   Facebook Prophet  
    
*   Auto Arima
    

感兴趣的话就去研究看看吧，此项目在 Github 上已有 1.4k+star。

https://github.com/linkedin/greykite

10. Jina and Finetuner
----------------------

如今，在搜索引擎等应用上，**语义识别**的地位越来越高，因为它可以有效避免字词匹配的局限。

不过语义识别涉及的神经网络可能会让很多人感到头大，**Jina** 和 **Finetuner** 可以帮你解决这些问题。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtA3msBKkN7VFDhAviaZTzxMdMxthNB6l207u4icyyT2xck7rRlt6LvB9y9KWAhYTK3VCg77FiaAzTefQ/640?wx_fmt=png)

Jina 是一个神经搜索框架，使任何人都能在**几分钟**内建立可扩展的深度学习搜索应用程序。

Finetuner 配合 Jina 帮助你对神经网络进行**调参**，以获得神经搜索任务的最佳结果。

Jina 和 Finetuner 适合没什么经验，又想尝试的朋友。

https://github.com/jina-ai/finetuner

参考链接：

https://tryolabs.com/blog/2021/12/21/top-python-libraries-2021