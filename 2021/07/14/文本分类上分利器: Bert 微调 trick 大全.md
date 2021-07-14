> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI3ODgwODA2MA==&mid=2247503785&idx=2&sn=46c47f361456a9b3d322d09bfb915346&chksm=eb53db3adc24522ce1fb76eb9e7ce17963294792544b69391bc291ccab27d1d6cf769ef41836&mpshare=1&scene=1&srcid=07146DXLmiCqnSsnJioRF0Lu&sharer_sharetime=1626229185598&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

点击下面**卡片**，**关注**我呀，每天给你送来 AI 技术干货！

来自：ChallengeHub

论文标题：How to Fine-Tune BERT for Text Classification? 中文  

中文标题：如何微调 BERT 进行文本分类？ 

论文作者：复旦大学**邱锡鹏**老师课题组 

实验代码：https://github.com/xuyige/BERT4doc-Classification

**1**

**前言**

大家现在打比赛对预训练模型非常喜爱，基本上作为 NLP 比赛基线首选（图像分类也有预训练模型）。预训练模型虽然很强，可能通过简单的微调就能给我们带来很大提升，但是大家会发现比赛做到后期，bert 等预训练模型炼丹一定程度的时候很难有所提升，分数达到了瓶颈，这个时候需要针对具体的任务如何进行微调使用，就涉及到了考经验积累的 tricks，这篇论文做了非常大的充足实验，为我们提供了宝贵的 BERT 微调经验及方法论，当需要应用 BERT 到具体的现实任务上时，可以参照这篇论文提供的调参路线进行优化，我在 NLP 比赛中也屡试不爽，总有一个 trick 是你的菜，推荐大家读一读这篇论文！

这篇论文的主要目的在于在文本分类任务上探索不同的 BERT 微调方法并提供一种通用的 BERT 微调解决方法。这篇论文从三种路线进行了探索：(1) BERT 自身的微调策略，包括长文本处理、学习率、不同层的选择等方法；(2) 目标任务内、领域内及跨领域的进一步预训练 BERT；(3) 多任务学习。微调后的 BERT 在七个英文数据集及搜狗中文数据集上取得了当前最优的结果。有兴趣的朋友可以点击上面的实验代码，跑一跑玩一玩~

**3**

**论文背景与研究动机**

文本分了是 NLP 中非常经典的任务，就是判断给定的一个文本所属的具体类别，比如判断文本情感是正向还是负向。尽管已经有相关的系研究工作表明基于**大语料**预训练模型可以对文本分类以及其他 NLP 任务有非常不错的效果收益和提升，这样做的一个非常大的好处我们不需要从头开始训练一个新的模型，节省了很大资源和时间。一种常见的预训练模型就是我们常见的词嵌入，比如 Word2Vec，Glove 向量，或者一词多义词向量模型 Cove 和 ELMo，这些词向量经常用来当做 NLP 任务的附加特征。另一种预训练模型是句子级别上的向量化表示，如 ULMFiT。其他的还有 OpenAI GPT 及 BERT。

虽然 BERT 在许多自然语言理解任务上取得了惊人的成绩，但是它的潜力还尚未被完全探索出来。很少有研究来进一步改进 BERT 在目标任务上的性能。这篇论文的主要目的就是通过探索多种方式最大化地利用 BERT 来增强其在文本分类任务上的性能。本篇论文的主要贡献如下：

（1）提出了一个通用的解决方案来微调预训练的 BERT 模型，它包括三个步骤：（1）进一步预训练 BERT 任务内训练数据或领域内数据；(2) 如果有多个相关任务可用，可选用多任务学习微调 BERT；(3) 为目标任务微调 BERT。

（2）本文研究了 BERT 在目标任务上的微调方法，包括长文本预处理、逐层选择、逐层学习率、灾难性遗忘

（3）我们在七个广泛研究的英文文本分类数据集和一个中文新闻分类数据集上取得了 SOTA 成果

**4**

**论文核心**

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXylEIGDgSl9raKlfW6weRlPbHUIibXFCFbgxeCJ3lZKmqhPlKQ3Xw6T2g/640)

*   **Fine-Tuning Strategies：当我们为目标任务微调 BERT 时，有很多方法可以 使用 BERT。例如，BERT 的不同层捕获不同级别的语义和句法信息，哪一层更适合目标任务？我们如何选择更好的优化算法和学习率？**
    
*   **Further Pre-training：BERT 在通用域中训练，其数据分布与目标域不同。一个自然的想法是使用目标域数据进一步预训练 BERT。这个真的非常有效，在微调达到一定瓶颈之后，可以尝试下在比赛语料上 ITPT，也就是继续预训练。在****海华阅读理解比赛****以及****基于文本挖掘的企业隐患排查质量分析模型****都得到了成功验证~**
    
*   **Multi-Task Fine-Tuning：在没有预先训练的 LM 模型的情况下，多任务学习已显示出其利用多个任务之间共享知识优势的有效性。当目标域中有多个可用任务时，一个有趣的问题是，在所有任务上同时微调 BERT 是否仍然带来好处。**
    

**5**

**微调策略**

**1.  处理长文本**我们知道 BERT 的最大序列长度为 512，BERT 应用于文本分类的第一个问题是如何处理长度大于 512 的文本。本文尝试了以下方式处理长文章。

**Truncation methods 截断法**文章的关键信息位于开头和结尾。我们可以使用三种不同的截断文本方法来执行 BERT 微调。

1.  **head-only: keep the first 510 tokens 头部 510 个字符，加上两个特殊字符刚好是 512 ;**
    
2.  **tail-only: keep the last 510 tokens; 尾部 510 个字符，同理加上两个特殊字符刚好是 512 ;**
    
3.  **head+tail: empirically select the first 128and the last 382 tokens.：尾部结合**
    

**Hierarchical methods 层级法**输入的文本首先被分成 k = L/510 个片段，喂入 BERT 以获得 k 个文本片段的表示向量。每个分数的表示是最后一层的 [CLS] 标记的隐藏状态，然后我们使用均值池化、最大池化和自注意力来组合所有分数的表示。

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyXUEvBVHnhT2HxYPcPDiaFHneLm3jiblmKXjK5j7XtRcfmgvoWBypur7Q/640)

上表的结果显示，head+tail 的截断法在 IMDb 和 Sogou 数据集上表现最好。后续的实验也是采用这种方式进行处理。

**2. 不同层的特征** BERT 的每一层都捕获输入文本的不同特征。文本研究了来自不同层的特征的有效性, 然后我们微调模型并记录测试错误率的性能。

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyRYRTMI3C4L9afibx7BPAmCph7FuDd6UUQ7obiaVGkEh9ib86X3GNiakjnQ/640)

我们可以看到：最后一层表征效果最好；最后 4 层进行 max-pooling 效果最好 **3. 灾难性遗忘** Catastrophic forgetting (灾难性遗忘) 通常是迁移学习中的常见诟病，这意味着在学习新知识的过程中预先训练的知识会被遗忘。因此，本文还研究了 BERT 是否存在灾难性遗忘问题。我们用不同的学习率对 BERT 进行了微调，发现需要较低的学习率，例如 2e-5，才能使 BERT 克服灾难性遗忘问题。在 4e-4 的较大学习率下，训练集无法收敛。

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXycw4Hg7m5v1NIqiaeaZNPyja757kYPhLhDlRvEYyYPqlEnZprlrNFjrg/640)

这个也深有体会，当预训练模型失效不能够收敛的时候多检查下超参数是否设置有问题。**4. Layer-wise Decreasing Layer Rate 逐层降低学习率**下表 显示了不同基础学习率和衰减因子在 IMDb 数据集上的性能。我们发现为下层分配较低的学习率对微调 BERT 是有效的，比较合适的设置是 ξ=0.95 和 lr=2.0e-5

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyLic5mibLnylwLFmK2aAnuXt02IfDX2IQGZgnNrHdQfKfRf15UXLGicCGA/640)

为不同的 BERT 设置不同的学习率及衰减因子，BERT 的表现如何？把参数θ \thetaθ划分成 {θ 1 , … , θ L} {\theta^1,\dots,\theta^L}{θ 1 ,…,θ L }，其中θ l \theta^lθ l

**6**

**ITPT：继续预训练**

Bert 是在通用的语料上进行预训练的，如果要在特定领域应用文本分类，数据分布一定是有一些差距的。这时候可以考虑进行深度预训练。

Within-task pre-training：Bert 在训练语料上进行预训练 In-domain pre-training：在同一领域上的语料进行预训练 Cross-domain pre-training：在不同领域上的语料进行预训练

1.  **Within-task pretraining**
    
      
    

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXycPjoRDJWIj7r0u9CaaBRF61lDUCjdT82OKiaH5EqZRS1LmEEAthcCtA/640)

**BERT-ITPT-FiT 的意思是 “BERT + with In-Task Pre-Training + Fine-Tuning”，上图表示 IMDb 数据集上进行不同步数的继续预训练是有收益的。2 In-Domain 和 Cross-Domain Further Pre-Training**

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXy1Zzc1gatQ8xyFhAX7nHiaFGxfyic7bor3bORUKxdc6XyicT6TdePfl8Vw/640)

**我们发现几乎所有进一步的预训练模型在所有七个数据集上的表现都比原始 BERT 基础模型。一般来说，域内预训练可以带来比任务内预训练更好的性能。在小句子级 TREC 数据集上，任务内预训练会损害性能，而在使用 Yah 的领域预训练中。Yah. A. 语料库可以在 TREC 上取得更好的结果。**

这篇论文与其他模型进行了比较，结果如下表所示：

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyDhIEyLzI9NMb2Dxj13QaBTUCKWKaMuVUZHm8GaNILUhodib7Sjqz3Xw/640)

我们可以看到 ITPT 和 IDPT 以及 CDPT 的错误率相比其他模型在不同数据集有不同程度下降。

**7**

**多任务微调**

所有任务都会共享 BERT 层及 Embedding 层，唯一不共享的层就是最终的分类层，每个任务都有各自的分类层。

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyL7EiaGpUB0LVsKuzC1ByI6hGVg6AvTYRegp9jhwlWm9v8AUJFrWMplA/640)

上表表明对于基于 BERT 多任务微调，效果有所提升，但是对于 CDPT 的多任务微调是有所下降的，所以说多任务学习对于改进对相关文本分类子任务的泛化可能不是必要的。

**8**

**小样本学习 Few-Shot Learning**

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXygiaaUycGzlsvNeAOm7iaY4AOH9icjZv7ibBeThdeec2FgqRLdIQTbOiauQg/640)

实验表明：BERT 能够为小规模数据带来显著的性能提升。

**9**

**BERT Large 模型上进一步预训练**

![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyflHryRIhwXmRYBrvE6qLprexGe8OY9DbZ0iaq2JUNGohWJ6tSEibCA3Q/640)

实验结果表明：在特定任务上微调 BERT Large 模型能够获得当前最优的结果。

接下来给大家带来干货部分：**不同学习率策略**的使用

**不同学习率策略**
-----------

完整代码 ChallengeHub 后台回复 “**学习率**” 获取

1.  **Constant Schedule**![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXygUy8Yq6iaOcr5x2XfFYjte2icQAvVFBDOk1YMicwL7tfpIuibVdKPRMG5w/640)
    
2.  **Constant Schedule with Warmup**![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyiblOlBSvRwfmcgJZFzRG9wYAsORXu7aCWl2PpBoc9FFxI6gGCy17iabw/640)
    
3.  **Cosine with Warmup**![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyhsAaP2SyvmkKLkjFXKcMAwb5AODuWPYYE2x1rMI34tcYEkjkbpTrww/640)
    
4.  **Cosine With Hard Restarts**![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyFB4FDwJKibB5kXTCQJ8PRtLYlrlNFGic3Ap5ibUmnX2TiaLJdiaTSq055WA/640)
    
5.  **Linear Schedule with Warmup**![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyq3Tfy0bo0EjF4JwfAkibm4tB8J2DFuA9Gm9I25J5rFUPZqrIrCwouBw/640)
    
6.  **Polynomial Decay with Warmup**![](https://mmbiz.qpic.cn/mmbiz_png/1FD1x61uYVdHZbIr7PUVOkURq9hgkjXyO7J7NU8sFqoialaWGs0qQBGjgI8djbB558X4fPu9YjP3MbwCUYM1MPA/640)
    

**参考资料**
--------

*   **一起读论文 | 文本分类任务的 BERT 微调方法论**
    
*   **NLP 重铸篇之 BERT 如何微调文本分类**
    
*   **【论文解析】如何将 Bert 更好地用于文本分类（How to Fine-Tune BERT for Text Classification?）**
    
*   **How to Fine-Tune BERT for Text Classification 论文笔记**
    
*   **Bert 微调技巧实验大全**
    
*   **论文阅读笔记：这篇文章教你在文本分类任务上微调 BERT**
    
*   **How to Fine-Tune BERT for Text Classification? 读论文** **-** **如何让 Bert 在 finetune 小数据集时更 “稳” 一点**