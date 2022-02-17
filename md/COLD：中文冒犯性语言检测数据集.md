> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODkzMzMwMQ==&mid=2650429108&idx=4&sn=e0b7812c077f5cd25d77b4a02b8d73da&chksm=becdd0ee89ba59f8b795baf6e361fe45f656795d78330ad8b2ef00e492fb9448d538933513ae&mpshare=1&scene=1&srcid=0217wT0aAddx9rpg8QWj4KTR&sharer_sharetime=1645107865802&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

```
paper：https://arxiv.org/pdf/2201.06025.pdf
```

为什么要分享这个数据集？

*   真实案例：本人在公司搞过一个**「生成式闲聊机器人」**，为了确保不出现敏感词语，通过大量敏感词典进行过滤；当自己觉得信心满满时，通过各方的测试反馈，发现虽然过滤了敏感词汇，但是依然会生成一些不友好的语言，甚至有时会**「怼」**用户；并且当测试对其进行诱导提问时，很容易生成一些奇奇怪怪的内容。
    
*   因此，如何可以使生成内容不包含冒犯性语言以及冒犯性语言的检测，在模型上线时十分重要。
    

不过数据集还没有真正的放出来，期待一下，实时关注。

```
原话：Dataset and codes will be made publicly available upon paper publication
```

介绍
--

预训练语言模型的出现，使得很多生成任务取得了很好的效果。但由于这些模型均基于大规模数据训练，不可避免地学习到一些有偏见或冒犯的内容，造成在真实场景上线时无法保证安全性。如下图所示，![](https://mmbiz.qpic.cn/mmbiz_png/iceGibVicRfib5neTQpoLvcIQ5qkZTmtLjBCe6YTnKE4nNQeDmbmqECwQNUF8OZQrYFdCQqwYHibmY9Om493Vwoib4dQ/640?wx_fmt=png)为了解决系统中的冒犯性语言问题，仅靠词典规则是远远不够的，因此很多人开始了语言毒性研究，也出现了一些数据集，例如 WTC、OLID、BAD 和 RealToxicPrompts；但这些数据都是英文数据集，无法在中文任务上直接使用，就算用过机器翻译的方法，翻译成中文，但语言习惯、语言表达、数据质量都无法得到保证。

该论文研究了中国社交平台上的冒犯性语言和流行的生成语言模型，提出了第一个可公开使用的中文侮辱性语言数据集 - COLDDateset，涵盖了种族、性别和地区等话题内容。

数据集
---

*   **「无标注数据获取」**，从社交媒体平台（微博和知乎）上抓取发布的真实数据，但由于平台均存在语言检测机制，表达冒犯性的数据比例相对较少。因此通过两种策略收集数据，(1) 从相关的子主题爬行，在知乎中搜索一些被高度讨论的子话题，并直接从后续评论中抓取数据。（2）关键字查询，从知乎和微博随机抓取大量数据，通过预先定义一些相关的关键词，从原始数据中检索出更多样化的句子。
    
*   **「数据标注」**，对上述得到的无标注数据，先进行部分人工标注，然后使用部分数据训练模型，再通过人工对不同的概率区间中选取数据标注，重新训练模型，反复五次，最终得到带有标注的数据。
    

数据集共包含 37480 个句子，其中，带冒犯性语言的句子有 18041 个，平均长度 53.69 个字符；不带冒犯性语言的句子有 19439，平均长度 44.20 个字符。![](https://mmbiz.qpic.cn/mmbiz_png/iceGibVicRfib5neTQpoLvcIQ5qkZTmtLjBCmIE3g0icMp6icBcbOWpBIkdOAaMbgeKf7a9mDyhjJibQCUIY8yE6ytLPQ/640?wx_fmt=png)

基线模型
----

论文中共提到了五种模型，如下：

*   COLDetector，使用 COLDDateset 数据，训练的 bert-base 模型。
    
*   Baidu Text Censor，百度文本审查工具，旨在识别有害内容，包括色情、暴力、恐怖主义、政治敏感性和辱骂。
    
*   Prompt-based Self-Detection，基于提示的自检模型，探索通过语言模型的内部知识来用于有害内容的自我检测。
    
*   TranslJigsaw Detector，使用翻译成中文的英文 TranslJigsaw 数据，训练的 bert-base 模型。
    
*   Random，随机预测。
    

结果如下图所示，采用 COLDDateset 数据训练的模型效果较好![](https://mmbiz.qpic.cn/mmbiz_png/iceGibVicRfib5neTQpoLvcIQ5qkZTmtLjBCvP55stsaCAsF8WAHDomQ1RVPCF7FtZI4CEhRjYV5q1pHHcjAVa3lwg/640?wx_fmt=png)对于 Baidu Text Censor，为了用户更好地体验，更倾向检测辱骂和暴力等非法内容，对于偏见的冒犯性语言检测效果较差。对于 Prompt-based Self-Detection，不同预测词语对，对结果影响较大，如下![](https://mmbiz.qpic.cn/mmbiz_png/iceGibVicRfib5neTQpoLvcIQ5qkZTmtLjBCvaricC2tEz6NYppBv50IqbDg58DOLMOHYJPhibBp3bXYKhbV82fO66Lg/640?wx_fmt=png)对于 TranslJigsaw Detector，验证了使用机器翻译过的英文数据集，较其他方法虽然效果有提示，但不如 COLDDateset 效果，如下，![](https://mmbiz.qpic.cn/mmbiz_png/iceGibVicRfib5neTQpoLvcIQ5qkZTmtLjBCwgbNMj4N5wPDy2CxuRSr3CAkx5XQMClfLb0uaFibJzbR7WsAKlamtiaA/640?wx_fmt=png)

对现有生成模型的评价
----------

对四种在 Huggingface 开源的生成模型进行冒犯性评价，结果如下，无论提示语是否包含冒犯性内容时，生成结果都会存在冒犯性内容。![](https://mmbiz.qpic.cn/mmbiz_png/iceGibVicRfib5neTQpoLvcIQ5qkZTmtLjBCF72RqTPgd6d6tWcxHRia16dWdfpklREbAfqNbVQiccCuoqQhVERlNXoQ/640?wx_fmt=png)并且，当提示语包含冒犯性内容时，生成结果存在冒犯性内容概率越高，![](https://mmbiz.qpic.cn/mmbiz_png/iceGibVicRfib5neTQpoLvcIQ5qkZTmtLjBCrvOkm6gQvB16J08fOsxkSRalpOGIuGXgH6xdrG77Itqiaf6icRNCHa9w/640?wx_fmt=png)

总结
--

个人认为，如果生成模型想要工业落地，控制生成内容是至关重要的（我组长对我说的一句话，**现在没出现问题，是还没人想搞你**。），仅仅敏感词点过滤会有些单薄，而冒犯性语言检测显得只管重要，需要从语义中检测内容。

该数据集仅从单句上进行标注，因此在对话系统中使用该数据集，还是存在一定的的弊端，如果通过联系上下文发现内容的风险。