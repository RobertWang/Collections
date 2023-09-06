> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rJhGdX5xGpr4Kg9k7I2wPQ)

BLOOMZ 简介
---------

BLOOM[3] 是一个拥有 1760 亿参数的自回归模型，训练后可用于生成文本序列。它可以处理 46 种语言和 13 种编程语言。

作为 BigScience[4] 计划中的一个开放科学项目，BLOOM 的设计和训练吸引了世界各地众多研究人员和工程师的共同参与。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDichgzrNibxuIFRxBsEZdCDtybPCTDhYIamZhfpd9I8kTiaQHAdianEal1PJriaNN9SaI5yYfH5u8R5yw/640?wx_fmt=png)

BLOOMZ[5] 是最近发布的、与 BLOOM 架构完全相同的模型，它是 BLOOM 基于多个任务的调优版本，具有更出色的泛化和零样本 [6] 能力。

无论是训练 [7] 还是推理 [8] 场景，这类大模型都对内存和速度提出了新挑战。

即便使用 16 位精度，一个实例所需的内存仍高达 352 GB！目前具有如此大内存的设备可能很难找到，但像 Habana® Gaudi®2 这样的先进硬件却足以让 BLOOM 和 BLOOMZ 模型以更低的时延执行推理。

Habana® Gaudi®2
---------------

Gaudi®2 是 Habana Labs 设计的第二代 AI 硬件加速器。单个服务器包含 8 张加速卡（称为 Habana 处理单元，即 Habana Processing Unit，简称为 HPU），每张卡内存高达 96 GB，可提供足够的空间来容纳大模型。

然而，如果计算速度很慢，那么为大模型提供大内存的意义也不大。所幸，Gaudi®2 的计算速度也非常出色。

Gaudi®2 与 GPU 的主要区别在于它的架构能让加速器并行执行通用矩阵乘法 (GeMM) 和其他运算，从而加快深度学习工作流。这些特性使 Gaudi®2 成为 LLM 训练和推理的理想选择。

Habana 的 SDK SynapseAITM 支持使用 PyTorch 和 DeepSpeed 来加速 LLM 训练和推理。SynapseAI 图形编译器 [9] 可优化图形中所累积的操作的执行（如算子融合、数据布局管理、并行化、流水线、内存管理、图优化等）。

以上所有功能均已集成至 Optimum Habana[12] 库，因此在 Gaudi® 上部署模型非常简单。

访问此链接 https://huggingface.co/docs/optimum/habana/quickstart，查看快速入门页面。

如欲试用 Gaudi®2，请登录英特尔 ® Developer Cloud[13] 并按照本指南 [14] 操作。

基准测试
----

本节将提供 BLOOMZ 在 Gaudi®2 和第一代 Gaudi® 上的基准测试结果。

虽然 Gaudi®2 和第一代 Gaudi® 的内存都不小，但由于模型过大，单个设备仍无法容纳单个 BLOOMZ 实例。

为解决这一问题，本文使用了深度学习优化库 DeepSpeed[15] 来实现多种内存和速度优化，进而加速模型推理并使模型与设备适配。

本文方案需依赖 DeepSpeed-inference[16]：它引入了诸如模型（或流水线）并行 [17] 等多个功能特性，可充分利用可用设备。

对于 Gaudi®2，则使用了已添加 HPU 支持的 Habana 的 DeepSpeed[18] 分支。

### 时延

本文基于两种不同规模但参数均达数十亿的 BLOOMZ 模型（批大小为 1 个样本）进行了实验测试，两种模型的参数大小分别为：

*   1760 亿 [19] 参数 (BLOOMZ-176B)
    
*   70 亿 [20] 参数 (BLOOMZ-7B)
    

本文使用 DeepSpeed-inference 以 16 位精度在 8 个设备上运行推理，并且使用 key-value 缓存。值得注意的是，尽管 CUDA Graph 目前与 DeepSpeed 中的模型并行不兼容（DeepSpeed v0.8.2，参见文末 [21]），但 Habana 的 DeepSpeed 分支是支持 HPU Graph 的。

所有基准测试都使用贪心搜索 (Greedy Search)[22] 生成 100 个词元。输入提示为：

BLOOM 分词器会将该提示分为 7 个词元。

推理时延测试结果如下图所示（单位为秒）：

###### **![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtDichgzrNibxuIFRxBsEZdCDtTDVKZKOtCFOWeDenOKqibtvHEqtaBncWFh4HWB3U7yabxZ7R2oPZcyw/640?wx_fmt=png)**

###### **△**图 1. BLOOMZ 在 Gaudi®2 和第一代 Gaudi® 上的推理时延测试结果

Habana 团队最近在 SynapseAI 1.8 中引入了 DeepSpeed-inference 支持，可快速赋能 1000 多亿参数模型的推理。

根据图 1 测试结果可知：对于参数量达 1760 亿的模型 BLOOMZ，Gaudi®2 性能表现出色，时延仅为约 3.7 秒 ；对于参数量为 70 亿的较小模型 BLOOMZ-7B，Gaudi®2 的时延优势更加显著，单设备约为第一代 Gaudi® 的 37.21%，而当设备数量都增加为 8 后，这一百分比进一步下降至约 24.33%。

Habana 团队会在后续发布的新 SynapseAI 版本中继续优化这些模型的性能。例如，在 SynapseAI 1.9 的预览版中，BLOOMZ-176B 的时延从 3.7 秒进一步降低到 3.5 秒。

在完整数据集上运行推理
-----------

我们编写的脚本支持模型在完整的数据集上完成所有句子的推理。这尤其适用于想在自有数据集上尝试使用 Gaudi®2 进行 BLOOMZ 推理的情况。

这里我们以 tldr_news 数据集为例。该数据集包含多篇文章的标题和内容（均可在 Hugging Face Hub 中对其进行可视化处理）。前几个样本如下所示：

```
Input: 'Businesses Will Not Be Able to Hide': Spy Satellites May Give Edge From Above ;
Output: 'Businesses Will Not Be Able to Hide': Spy Satellites to Track Illegal Logging in Indonesia
The Indonesian government has announced that it will use spy satellites to track illegal logging in the country. 
Input: Alphabet and SoftBank’s solar-powered drone provides first LTE connection ;
Output: Alphabet and SoftBank’s solar-powered drone provides first LTE connection connection from the stratosphere. The drone, which is about the size of a small car, is equipped with a solar panel and can fly for up 
Input: SQLFlow (GitHub Repo);
Output: SQLFlow (GitHub Repo):
https://github.com/davidfowl/SQLFlow
The code is written in Java and is available on GitHub. It is a simple, lightweight
Input: Secret to keeping ice cream creamy (not crunchy);
Output: Secret to keeping ice cream creamy (not crunchy) is to freeze it in a container that is airtight. 
Input: Tesla's giant battery saved $40 million during its first year, report says ;
Output: Tesla's giant battery saved $40 million during its first year, report says
Tesla's giant battery saved $40 million during its first year, report says
Tesla's giant battery saved $40 million during its first year,
Input: Python 3.9: Cool New Features for You to Try (28 minute read);
Output: Python 3.9: Cool New Features for You to Try (28 minute read): This is a great article for those 
Input: A company aims to power the world for millions of years by digging the deepest holes ever ;
Output: A company aims to power the world for millions of years by digging the deepest hole ever made in 
Input: In Nevada desert, a technology firm aims to be a government ;
Output: In Nevada desert, a technology firm aims to be a government Introduction
The use of the Internet has become a common practice in the daily life of people. The Internet has become

△若代码显示不全，请左右滑动

```

下一节将展示如何用该脚本来执行基准测试，以及如何将其应用于 Hugging Face Hub 中任何您喜欢的数据集。

如何复现这些结果？
---------

访问文末 [23] 获取在 Gaudi®2 和第一代 Gaudi® 上对 BLOOMZ 进行基准测试的脚本。

在运行上述脚本之前，请确保按照 Habana 提供的指南 [24] 安装了最新版本的 SynapseAI 和 Gaudi® 驱动程序。

然后，运行以下命令：

```
git clone https://github.com/huggingface/optimum-habana.git
cd optimum-habana && pip install . && cd examples/text-generation
pip install git+https://github.com/HabanaAI/DeepSpeed.git@1.8.0

△若代码显示不全，请左右滑动

```

最后，按如下方式运行脚本：

```
git clone https://github.com/huggingface/optimum-habana.git
cd optimum-habana && pip install . && cd examples/text-generation
pip install git+https://github.com/HabanaAI/DeepSpeed.git@1.8.0

△若代码显示不全，请左右滑动

```

关于多节点推理，请查看和遵循 Optimum Habana 文档中的指南 [25]。

使用参数 —dataset_name my_dataset_name 即可加载来自 Hugging Face Hub 的任何数据集以获取用于文本生成的提示。

此基准测试基于 Transformers v4.27.1、SynapseAI v1.8.0，和源码安装的 Optimum Habana。

对于 GPU，此代码库 [26] 包含了可用于复现本文 [27] 前述测试结果的脚本。静态形状 (static shape) 是使用 CUDA Graph 的必要条件，而 Transformers 并不支持静态形状。因此，您需使用 Habana 团队编写的代码 [28] 来启用静态形状。

结论
--

从本文可以看出，Habana® Gaudi®2 在执行 BLOOMZ 推理时，具有较优的速度优势，且无需编写复杂的脚本，因为 Optimum Habana[29] 提供了易于使用的工具，来支持在 HPU 上运行数十亿参数模型的推理。

Habana 的 SynapseAI SDK 将于后续版本实现进一步的性能提升。随着 SynapseAI 上大语言模型推理优化的不断推进，我们也将定期对此基准测试进行更新，同时也期待 Gaudi®2 为 FP8 推理带来更多性能优势。

如有兴趣使用最新 AI 硬件加速器和软件库来加速机器学习训练和推理工作流，请查看 Hugging Face 的专家加速计划 [30]。

如需了解有关 Habana 解决方案的更多信息，请阅读了解 Habana 与 Hugging Face 的合作关系 [31] 并联系 Habana[32]。

如需详细了解 Hugging Face 如何让 AI 硬件加速器更易于使用，请查看 Hugging Face 的硬件合作伙伴计划 [33]。

英特尔研究院认知 AI 团队研究科学家 Philip Howard 和 Anahita Bhiwandiwalla 还介绍了 Gaudi®2 与 BLOOMZ 的相关测试。

可点击观看视频 [34]，了解如何在 Gaudi®2 上轻松部署 BLOOMZ 等大语言模型。

参考链接：  

[1]https://huggingface.co/blog/zh/habana-gaudi-2-bloom

[2]https://habana.ai/products/gaudi2/

[3]https://arxiv.org/abs/2211.05100

[4]https://bigscience.huggingface.co/

[5]https://arxiv.org/abs/2211.01786

[6]“零样本” 是指模型基于新输入数据或无准备输入数据（即未提供任何训练示例的数据）完成任务的能力。我们向模型提供提示和以自然语言描述的指令（即我们希望模型做什么）。零样本分类不包括与正在完成的任务相关的任何示例。这区别于单样本或少样本分类，因为这些任务包括特定任务的一个或多个示例。

[7]https://huggingface.co/blog/bloom-megatron-deepspeed

[8]https://huggingface.co/blog/bloom-inference-optimization

[9]https://docs.habana.ai/en/latest/Gaudi_Overview/SynapseAI_Software_Suite.html#graph-compiler-and-runtime

[10]https://docs.habana.ai/en/latest/PyTorch/Inference_on_PyTorch/Inference_Using_HPU_Graphs.html

[11]https://docs.habana.ai/en/latest/PyTorch/DeepSpeed/Inference_Using_DeepSpeed.html

[12]https://github.com/huggingface/optimum-habana

[13]https://huggingface.co/docs/optimum/habana/quickstart

[14]https://huggingface.co/blog/habana-gaudi-2-benchmark#how-to-get-access-to-gaudi2

[15]https://www.deepspeed.ai/

[16]https://arxiv.org/abs/2207.00032

[17]https://huggingface.co/blog/bloom-megatron-deepspeed#pipeline-parallelism

[18]https://github.com/HabanaAI/deepspeed

[19]bigscience/bloomz · Hugging Face

[20]bigscience/bloomz-7b1 · Hugging Face

[21]https://github.com/microsoft/DeepSpeed/blob/v0.8.2/deepspeed/inference/engine.py#L158

[22]https://huggingface.co/blog/how-to-generate#greedy-search

[23]https://github.com/huggingface/optimum-habana/tree/main/examples/text-generation

[24]https://docs.habana.ai/en/latest/Installation_Guide/index.html

[25]https://huggingface.co/docs/optimum/habana/usage_guides/multi_node_training

[26]transformers-bloom-inference/bloom-inference-scripts at main · huggingface /transformers-bloom-inference · GitHub

[27]IncrediblyFast BLOOM Inference with DeepSpeed and Accelerate (huggingface.co)

[28]Model-References/PyTorch/nlp/bloom at 1.8.0 · HabanaAI/Model-References · GitHub

[29]https://huggingface.co/docs/optimum/habana/index

[30]https://huggingface.co/support

[31]https://huggingface.co/hardware/habana

[32]https://habana.ai/contact-us/

[33]https://huggingface.co/hardware

[34]https://videos.sproutvideo.com/embed/799fd9b8141be0c6f0/79c248b5ecd76231?playerColor=0270c1&endFrame=posterFrame&autoplay=true&lightbox=true

* 本文系量子位获授权刊载，观点仅为作者所有。

**最 “in” 大模型专栏**

**1**

[十亿参数，一键瘦身！「模型减重」神器让大模型狂掉 3/4](http://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247689770&idx=5&sn=ec104278d2c91d013ad72183ff651ffa&chksm=e8df5158dfa8d84eba51ad2f70504f39f5379e4a5f02c300866fe54d264b6c5835b2a26f34c3&scene=21#wechat_redirect)

**2**

[保护大模型应用安全，现在不需要拿性能做代价了](http://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247690127&idx=5&sn=e324bda2a55f74a0988ccb4c02f60cbe&chksm=e8df50fddfa8d9ebb6e518e3ea475ec9b906a04f6607e03c42712f9f5f9a72205d4c498c46f0&scene=21#wechat_redirect)

**3**

[如何优化 ChatGLM-6B？一行代码就行](http://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247690618&idx=2&sn=65587070da3273fa96ccb353ed407335&chksm=e8df5e08dfa8d71e157456b3419dc15bcdec92447a31092b15ab45ae3d8cb4f8a52dea160e08&scene=21#wechat_redirect)

**4**

[一个简单模型就让 ChatGLM 性能大幅提升](http://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247691483&idx=2&sn=6a0e25af4f1f0aa05f6c42db5bac6ec8&chksm=e8df5ba9dfa8d2bfe31a04e145dce89829a10612ffcc0522820c98b964f0035a065b40892baa&scene=21#wechat_redirect)

— **完** —