> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzk0MjU3ODc5OQ==&mid=2247485272&idx=1&sn=ff6a1b126047553b99212b9714927f15&chksm=c2c04a58f5b7c34e889a29f2aa9712ca6b5426c29d32dfaabb2a8755e36072db1eb89b1c4770&mpshare=1&scene=1&srcid=0129C0nEFriuQm2aGkaqNF7A&sharer_shareinfo=3ae142f2d20d7bd99d48a6c4d60509ef&sharer_shareinfo_first=3ae142f2d20d7bd99d48a6c4d60509ef#rd)

**了解如何设计、开发、部署和迭代生产级 ML 应用程序。**

资源：翻译版本  

http://www.gitpp.com/farsoft/made-with-ml-cn

构建基于 RAG 的 LLM 生产应用程序
=====================

以下为 www.gitpp.com 翻译版本，水平有限  

大型语言模型（LLM）无疑改变了我们与信息交互的方式。然而，对于我们可以向他们提出的要求，它们也有相当多的限制。基础 LLM（例如`Llama-2-70b`，，`gpt-4`等）只知道他们接受过培训的信息，当我们要求他们了解除此之外的信息时，他们就会达不到要求。基于检索增强生成（RAG）的 LLM 应用程序解决了这个确切的问题，并将 LLM 的实用性扩展到我们的特定数据源。 

![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m3iadfJibTAtRDlMbwQhWZcxibOr2FewQWNl3Pc0FxwxLtXTeW0Ux3FNnSw/640?wx_fmt=png&from=appmsg)

1、矢量数据库创建
---------

在开始构建 RAG 应用程序之前，我们需要首先创建包含已处理数据源的向量数据库。

![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m3EPDvBJrEhj2EAyqnMYAEaOiaO7uz6U5qgduanVValcO7vb7jKGZVsGQ/640?wx_fmt=png&from=appmsg)

###  数据

我们现在有一个部分列表（包含每个部分的文本和源代码），但我们还不应该直接将其用作 RAG 应用程序的上下文。每个部分的文本长度各不相同，并且许多都是相当大的块。 

![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m3HMJOznbZNIVdIAjzmS5DUibXAfAaFtETy7fcICWYWNuyDDSyJBp5v9g/640?wx_fmt=png&from=appmsg)

2、查询检索
------

通过在矢量数据库中索引嵌入的块，我们准备好对给定的查询执行检索。我们将首先使用用于嵌入文本块的相同嵌入模型来嵌入传入的查询。

![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m3FoxmO8ibhW6sN4zEXXbeRdCpuPnlXiavzv8OfKghJUHKDucPlc1KMGsQ/640?wx_fmt=png&from=appmsg)

3、响应生成
------

现在，我们可以使用上下文来生成 LLM 的响应。如果没有我们检索到的相关背景，LLM 可能无法准确回答我们的问题。随着数据的增长，我们可以轻松地嵌入和索引任何新数据，并能够检索它来回答问题。

![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m3AvVj7MmJDjMz1yC337ryFzJ7RSxAAGaibpPdLDc82YnibTu1j7Npuibjg/640?wx_fmt=png&from=appmsg)

### 代理  Agent

**让我们将上下文检索和响应生成结合到一个方便的查询代理中，我们可以使用它轻松生成响应**。这将负责设置我们的代理（嵌入和 LLM 模型）以及上下文检索，并将其传递给我们的 LLM 以生成响应。

搭建一个 LLM 应用很简单，是不是？？  

![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m30amZ21tYxLyyVVfib9vwLQ38Cbs3mr9ZLu6Aq54ibKQd3wX1cRBCGYuA/640?wx_fmt=png&from=appmsg)

评估
--

到目前为止，我们已经为 RAG 应用程序的各个部分选择了典型 / 任意值。但如果我们要改变一些东西，比如我们的分块逻辑、嵌入模型、LLM 等，我们怎么知道我们的配置比以前更好呢？像这样的生成任务很难定量评估，因此我们需要开发可靠的方法来做到这一点。

因为我们的应用程序中有许多移动部件，所以我们需要执行单元 / 组件和端到端评估。逐个组件的评估可以涉及单独评估我们的检索（是我们检索到的块集中的最佳来源）和评估我们的 LLM 响应（给定最佳来源，LLM 是否能够产生高质量的答案）。对于端到端评估，我们可以评估整个系统的质量（给定数据源，响应的质量如何）。

我们将要求我们的评估员 LLM 使用上下文对响应的质量进行评分 1-5 之间，但是，我们也可以让它为其他维度（例如幻觉）生成分数（是仅使用所提供上下文中的信息生成的答案） ）、毒性等。

注意：我们可以将分数限制为二进制 (0/1)，这可能更容易解释（例如，响应要么正确，要么不正确）。然而，我们在分数中引入了更高的方差，以便更深入、更细致地理解 LLM 如何对回答进行评分（例如 LLM 对回答的偏见）。

![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m3YiaOkJmszUrBLKficpdnW6oIu7zxOmXACXcxfEsVHBtzbFiaSZqfuXqsg/640?wx_fmt=png&from=appmsg)

### 冷启动

我们可能并不总是有准备好的问题数据集和随时可用的回答该问题的最佳来源。为了解决这个冷启动问题，我们可以使用 LLM 来查看我们的文本块并生成特定块将回答的问题。这为我们提供了质量问题以及答案的确切来源。但是，这种数据集生成方法可能有点嘈杂。生成的问题可能并不总是与我们的用户可能提出的问题高度一致。我们所说的最佳来源的特定块也可能在其他块中具有确切的信息。尽管如此，当我们收集 + 手动标记高质量数据集时，这是开始我们的开发过程的好方法。

![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m3zoo0Esn6adtEicl54v7r6JkCe2wRogulwNOptiauV8OpSuUlf002rxNg/640?wx_fmt=png&from=appmsg)

有了我们的评估器集，我们就可以开始试验 LLM 申请中的各个组件了。虽然我们可以将其作为大型超参数调整实验来执行，我们可以在其中搜索有希望的值 / 决策组合，但我们将一次评估一个决策，并为下一个实验设置最佳值。

注意：这种方法有点不完美，因为我们的许多决策不是独立的（例如，`chunk_size`理想`num_chunks`情况下应该在许多值组合中进行评估）。

![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m36HtBNvUiboSZeAXsTy1aXttFD8HE4aP4weIvZfcQaC8dRNNtNibxlEQA/640?wx_fmt=png&from=appmsg)

微调
--

到目前为止，我们探索的所有内容都涉及优化数据预处理方式以及按原样使用我们的模型（嵌入、LLM 等）。然而，还值得探索使用我们用例特有的数据来微调我们的模型。这可以帮助我们更好地表示我们的数据，并最终提高我们的检索和质量得分。在本节中，我们将微调我们的嵌入模型。这里的直觉是，学习比默认嵌入模型更符合上下文的令牌表示可能是值得的。如果我们有很多：

*   默认标记化过程创建子标记的新标记失去了标记的重要性
    
*   在我们的用例中具有上下文不同含义的现有标记
    
*   ![](https://mmbiz.qpic.cn/mmbiz_png/qt08A648NlPsrdZApbdNiar0I01ayj5m3SXhFoAibY3k1gvBVsrLBnX7UEfQPAzdjiaNsdATwbpbYsGoia8JgzmHxw/640?wx_fmt=png&from=appmsg)
    

### 综合数据集

我们的第一步是创建一个数据集来微调我们的嵌入模型。我们当前的嵌入模型已经通过自监督学习（word2vec、GloVe、下一个 / 屏蔽标记预测等）进行了训练，因此我们将继续通过自监督工作流程进行微调。我们将重复使用与之前的冷启动 QA 数据集部分非常相似的方法，以便我们可以将数据中的部分映射到问题。这里的微调任务是让模型确定数据集中的哪些部分最适合输入查询。此优化任务将使我们的嵌入模型能够学习数据集中标记的更好表示。

注意：虽然我们可以创建一个将部分标题与部分文本映射的数据集，但我们正在创建一个综合问答数据集，因为它将最能代表我们想要学习如何嵌入的数据类型。

我们的提示会有所不同，因为我们想要生成各种不同的问题，并且我们将`llama-70b`在此处使用，以便我们可以扩展此 QA 生成过程（并避免任何速率限制）。为了彻底起见，我们将从数据集中的每个部分生成一个问题，以便我们可以尝试捕获尽可能多的独特标记。

### 训练数据

我们现在将把数据集分成训练和验证部分。

### 验证

我们的验证评估标准涉及一个信息检索 (IR) 评估器，该评估器将从每个查询的语料库中检索前 k 个相似文档。InformationRetrievalEvaluator 需要以下输入：

*   查询：`Dict[str, str]`  # qid => 查询
    
*   语料库：`Dict[str, str]`  # cid => doc
    
*   related_docs: `Dict[str, Set[str]]`  # qid => 设置 [cid]
    

注意：虽然我们的数据集可能有针对特定查询的多个有效部分，但我们会将除用于生成查询的部分之外的所有其他部分视为负样本。这不是一个理想的场景，但引入的噪声很小，特别是因为我们使用它来调整表示层（而不是用于分类任务）。

数据飞轮
----

创建这样的应用程序不是一次性任务。继续迭代并使我们的应用程序保持最新状态非常重要。这包括不断重新索引我们的数据，以便我们的应用程序能够使用最新的信息。以及重新运行我们的实验，看看是否有任何决定需要改变。这个持续迭代的过程可以通过将我们的工作流程映射到 CI/CD 管道来实现。

除了自动重新索引、评估等之外，迭代的一个关键部分涉及修复我们的数据本身。事实上，我们发现这是我们可以控制的最有影响力的杠杆（远远超出我们上面的检索和生成优化）。以下是我们确定的工作流程示例：

1.  用户使用 RAG 应用程序询问有关产品的问题。
    
2.  使用反馈（👍/👎、访问过的源页面、top-k 余弦分数等）来识别表现不佳的查询。
    
3.  检查检索的资源、标记化等，以确定是否是检索、生成或底层数据源的缺陷。
    
4.  如果数据中的某些内容可以改进，分为部分 / 页面等→修复它！
    
5.  对以前表现不佳的查询进行评估（并添加到测试套件中）。
    
6.  重新索引并部署一个新的、可能进一步优化的应用程序版本。
    

影响
--

### 产品和生产力

建立这样的 LLM 申请对我们的产品和公司产生了巨大的影响。预期对我们产品的整体开发人员和用户采用产生一阶影响。以自助和即时的方式交互和解决用户遇到的问题的能力是可以改善任何产品体验的功能类型。它使人们更容易取得成功，并将对 LLM 申请的看法从 “可有可无” 提升为 “必备”。 

### 基础代理

然而，还有一些我们没有立即意识到的二阶影响。例如，当我们进一步检查得分较低的用户查询时，通常由于我们的文档中存在空白而存在问题。当我们进行修复时（例如，在我们的文档中添加适当的部分），这改进了我们的产品和 LLM 应用程序本身 - 创建了一个非常有价值的反馈飞轮。此外，当内部团队了解我们的 LLM 应用程序的功能时，这就产生了高度有价值的 LLM 应用程序的开发，这些应用程序依赖于 Ray docs LLM 应用程序作为其用于执行其任务的 基础代理之一。

以下为 www.gitpp.com 翻译版本，水平有限

**了解如何设计、开发、部署和迭代生产级 ML 应用程序。**

资源： 翻译版本  

http://www.gitpp.com/farsoft/made-with-ml-cn