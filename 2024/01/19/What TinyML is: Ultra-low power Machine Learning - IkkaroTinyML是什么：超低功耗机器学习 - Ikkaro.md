> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.ikkaro.net](https://www.ikkaro.net/what-tinyml-is/)

> Explanation of TinyML, what it is used for and the main techniques it usesTinyML的解释，它的用途和它使用的主要技术

![](https://www.ikkaro.net/wp-content/uploads/2024/01/what-is-tinyml-1024x320.jpg)

TinyML or Tiny Machine Learning refers to the **use of Machine Learning in microcontrollers**. In systems that unlike those used in traditional ML have few resources, are systems that have little CPU, little RAM and extremely low power consumption in the order of magnitude of milliwatts or microwatts.  
TinyML 或 Tiny Machine Learning 是指在微控制器中使用机器学习。与传统ML中使用的系统不同，这些系统资源很少，CPU很少，RAM很小，功耗极低，数量级为毫瓦或微瓦。

Its official website is the [TinyML Foundation](https://www.tinyml.org/).  
它的官方网站是TinyML基金会。

What is done is to reduce large models for use with equipment with very few resources and microcontrollers. The preferred field of the Makers.  
所做的是减少大型模型，以便与资源和微控制器很少的设备一起使用。Makers 的首选字段。

I have started a series of 3 courses offered by Harvard for free  
我已经开始了哈佛大学免费提供的 3 门课程

1.  _[Fundamentals of TinyML](https://www.edx.org/learn/machine-learning/harvard-university-fundamentals-of-tinyml)_ (What do I build, what for and what are the problems)  
    TinyML的基础知识（我构建什么，为什么以及存在什么问题）
2.  _Applications of TinyML_ (data-driven, bias, etc)  
    TinyML的应用（数据驱动、偏差等）
3.  _Deploying TinyML_ (where do we put our models, security and privacy)  
    部署 TinyML（我们将模型、安全性和隐私放在哪里）

The following notes are from the first Fundamentals of TinyML where they explain what it is, when it is applied, the different techniques that are used, etc, etc.  
以下注释来自TinyML的第一个基础，其中解释了它是什么，何时应用，使用的不同技术等。

Embedded systems using microcontrollers cannot work with the large models, as they have memories up to 256kB. Here are some examples of operating systems that can be used with microcontrollers  
使用微控制器的嵌入式系统无法与大型模型一起使用，因为它们具有高达256kB的存储器。以下是一些可与微控制器一起使用的操作系统示例

*   [FreeRTOS](https://www.freertos.org/index.html)
*   [Mbed OS Mbed 操作系统](https://os.mbed.com/mbed-os/)

[Machine Learning](https://www.ikkaro.net/machine-learning/) consists of algorithms that search for patterns in data.  
机器学习由在数据中搜索模式的算法组成。

With TinyML, techniques are used to compress these algorithms so that they remain effective in finding patterns in data.  
使用 TinyML，使用技术来压缩这些算法，以便它们在查找数据模式时保持有效。

There are 5 quintillion bytes of data produced daily by IoT and only less than 1% is analyzed.  
物联网每天产生 5 万亿字节的数据，只有不到 1% 的数据被分析。

Algorithm compression techniques  
算法压缩技术
-----------------------------------------

Some algorithm compression techniques are:  
一些算法压缩技术包括：

### Pruning 修剪

**Pruning Synapsis:** We remove network connections from the model. Sometimes it can decrease the accuracy.  
修剪突触：我们从模型中删除网络连接。有时它会降低准确性。

**Pruning Neurons:** We can also eliminate entire neurons from our model which reduces the computational demand of the network.  
修剪神经元：我们还可以从模型中消除整个神经元，从而减少网络的计算需求。

### Quantization 量化

It consists of discretizing the values within a small range. For example if we discretize a float within the range -128 to 127 we only have to traverse 256 values. Going from a float point value that is stored in 4 bytes to an integer value that is stored in 1 byte implies a x4 reduction in size.  
它包括在一个小范围内离散化值。例如，如果我们离散化 -128 到 127 范围内的浮点数，我们只需要遍历 256 个值。从以 4 个字节存储的浮点值到以 1 个字节存储的整数值意味着大小减小 4 倍。

Quantization is going to be critical in TinyML due to the limited resources available.  
由于可用资源有限，量化在 TinyML 中至关重要。

### Knowledge distillation 知识提炼

Apply our knowledge and know how to make the model small.  
运用我们的知识，知道如何使模型变小。

Tools 工具
--------

We use Tensor Flow Lite. While tensorFlow is focused on ML Researcher, Tensor Flow Lite is for Application Developer.  
我们使用 Tensor Flow Lite。tensorFlow 专注于 ML Researcher，而 Tensor Flow Lite 则面向 Application Developer。

Uses of TinyML TinyML的用途
------------------------

Although they are not cited, of course being on this website we can find **uses of TinyML dedicated to the DIY, Maker and Hacker world**.  
虽然它们没有被引用，但当然在这个网站上，我们可以找到专门用于 DIY、Maker 和 Hacker 世界的 TinyML 的用途。

### Uses of TinyML in Industry  
TinyML在工业中的应用

In Industry, in maintenance, to warn us when there are vibrations that indicate that there will be breakage, etc, etc. increases efficiency and reduces costs. The negative points are the accuracy that can give us false alarms. In case of false alarm whose responsibility is the operator or the system.  
在工业中，在维护中，当有振动表明会有破损等时警告我们，可以提高效率并降低成本。缺点是可以给我们误报的准确性。在误报的情况下，由操作员或系统负责。

### TinyML in the environment  
环境中的 TinyML

Instead of collecting data that then has to be processed, with TinyML we have real-time answers about changes in the environment, for example in the life of wild animals.  
使用TinyML，我们无需收集数据，然后必须进行处理，而是可以实时了解环境变化，例如野生动物的生活变化。

### TinyML for humans 面向人类的 TinyML

Helps people with disabilities to perform more tasks without having to use their hands. Improving the UI and UX of applications to make them easier to use.  
帮助残障人士在不动手的情况下执行更多任务。改进应用程序的 UI 和 UX，使其更易于使用。

We build technology to improve our experience as humans. Technology has to help people  
我们开发技术来改善我们作为人类的体验。技术必须帮助人们

Risks and downsides 风险和缺点
-------------------------

*   Will it work well across all population groups?  
    它是否适用于所有人群？
*   Is the privacy of our data assured?  
    我们数据的隐私是否得到保证？
*   Can we protect this data?  
    我们能保护这些数据吗？

We have to create technology based on human-centered AI. Design, development and deployment  
我们必须创造基于以人为本的人工智能的技术。设计、开发和部署