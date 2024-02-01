> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg4ODUxMzE5NQ==&mid=2247485192&idx=1&sn=030bfec361f2eaf3e3cd88381453baab&chksm=cff8b0ccf88f39daf4e67be848d92da2480fb7f5175250f2913bf571a6d1fa2f96b44f4b4158&mpshare=1&scene=1&srcid=0201Nk3p5UJAAtRfL78OL99I&sharer_shareinfo=3cc6a3ee4a3772b2f817b222ffde7d0b&sharer_shareinfo_first=3cc6a3ee4a3772b2f817b222ffde7d0b#rd)

AI 大神贾扬清周末狂炫技！仅用 500 行代码打造的 AI 搜索引擎 Demo 就登顶 GitHub 热榜，告诉世界构建 AI 应用不过如此。谁说打造 AI 应用难如登天？贾扬清用实际行动告诉你：天下没有难构建的 AI 应用！![](https://mmbiz.qpic.cn/sz_mmbiz_png/qbIU4o0icv1ChARVFhUSVHNU3ZJooicIKNX8HyqrGibtuAdm4ZgC74Frff9V6TNhTicBKooB2pBRMG7mGorHFvx4iaw/640?wx_fmt=png&from=appmsg)

AI 搜索的三大流派，你知道吗？
================

如今 AI 搜索风头正劲，但你知道吗？它们在设计上其实分三大流派哦！

流派一：卡片式展示
---------

像谷歌、百度这样的老大哥，在传统搜索的页面顶部，利用卡片形式来直接给你展示 AI 生成的答案。简洁明了，一眼就能看明白！

流派二：对话式搜索
---------

必应、百度文心一言则更偏重对话。你可以像和朋友聊天一样，提出问题，AI 会帮你总结提炼答案，并在多轮对话中逐步展现。轻松自在，就像有个智能助手在身边！

流派三：新范式代表——Perplexity
---------------------

还有一类产品，它们遵循的是 Perplexity 为代表的新范式。搜索结果页面被分为 “参考链接 - AI 回答 - 相关追问” 几个部分，你可以根据需求多轮提问。最厉害的是，搜索结果还有历史记录，可以分享给朋友！而且，Copilot 增强模式下，AI 还能反向提问，引导你补充搜索条件。个性化提示词功能更是让你随心所欲地调整 AI 回答的风格和格式！

Perplexity AI 是一家成立于 2022 年 8 月的公司，总部位于旧金山。其创始人兼首席执行官 Aravind Srinivas 具有丰富的人工智能背景，曾在 OpenAI 担任研究科学家。此外，其创始团队还包括 Denis Yarats 和 Johnny Ho 等具有人工智能相关背景的人才。Perplexity AI 的工作原理是通过理解并重新构建用户查询，从实时索引中提取相关链接，然后利用大语言模型（LLM）阅读链接并整合内容，形成精准答案。

不得不说，Perplexity 的模式真的大获成功！就像它的 CEO 所说，现在这种模式几乎成了行业标准。甚至那些小细节，比如 “付费功能的免费使用次数”，都被后来的 AI 搜索产品学去了！

项目背景与特点
=======

面对 Perplexity AI 的挑衅，贾扬清进行了正面回击，强调了 LeptonAI 在 AI 开发领域的专业性和优势。他表示，LeptonAI 的焦点在于构建一个帮助开发者构建人工智能应用程序的现代云平台，而不是做一个搜索引擎。尽管如此，为了展示 LeptonAI 的实力和效率，他们还是用手搭建了一个简单的演示工具，并计划将其开源。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qbIU4o0icv1ChARVFhUSVHNU3ZJooicIKNDJX7LZlqzaswia2aTasiag2Uj4b5iaa5jstD7LsrXDQUANeUSySlzVbtQ/640?wx_fmt=png&from=appmsg)在这里插入图片描述

500 行代码背后，通过调用已有的基础架构，Lepton Search 实现了高效、精准的搜索功能。同时，内置的 LLM 支持使得搜索引擎能够理解并回答更复杂的问题，提升了用户体验。此外，美观且可分享的 UI 界面也是这个 Demo 的一大亮点，让用户在享受技术带来的便捷的同时，也能感受到视觉上的愉悦。

LeptonAI 开源项目：https://github.com/leptonai/search_with_lepton

使用环境：https://search.lepton.run/

技术亮点解析
======

目前，大语言模型主要面临两大挑战：数据陈旧和偶发幻觉。由于预训练数据集具有明确的截止日期，因此无法根据最新数据做出响应。这导致即使是当前最强大的模型，也往往会因数据过时而编造答案，即出现 “幻觉” 问题。

对于无法访问最新数据的问题，有两种主要的解决方法。第一种是通过搜索引擎执行网络搜索并向大模型提交数据来改善决策质量。Perplexity AI 更依赖于这种方法。第二种方法是使用检索增强生成（RAG）技术，这是一种成熟的技术，可以解决一定程度的 “幻觉” 问题。与动态调用搜索 API 方法不同，RAG 强调从公开数据存储中检索数据。

LeptonAI 基于 RAG 技术方式，通过调用已有基础架构的方式构建了一个简单的搜索引擎。

500 行代码的 AI 搜索引擎功能列表
--------------------

1. 大模型，调用了在自家云上部署的开源 Mixtral-8x7b 模型。2. 搜索引擎，目前用了必应搜索的 API。3. 数据存储，用自家 Lepton KV 作为无服务器存储。4. 对大模型和搜索引擎的接口支持 5. 前端 UI 界面 6. 可缓存和可分享的搜索结果

搜索引擎支持与设置
=========

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qbIU4o0icv1ChARVFhUSVHNU3ZJooicIKNaXWAr9v2xdBUTKI24y99HlaWtGa4srywjT5r6qQmDhkucwAYBp2tWw/640?wx_fmt=png&from=appmsg)**整个流程是**: 用户查询 -> 前端 -> 后端 -> Lepton LLMAPI -> 后端 -> 前端 -> 用户

Lepton Search 支持 Bing 和 Google 两大搜索引擎，用户可以根据自己的需求进行设置。对于想要快速尝试 Demo 的用户，还可以使用 Lepton Demo API 直接体验。在设置搜索引擎 API 时，用户只需按照相应的指示获取 API 密钥，并进行简单的配置即可。

部署与应用
=====

Lepton Search 的部署过程也异常简单。用户只需在 Lepton AI 平台上进行一键部署，即可将搜索引擎 Demo 快速上线。同时，用户还可以根据自己的需求进行自定义配置，如设置部署名称、资源形状等。

**一键部署**：https://dashboard.lepton.ai/workspace-redirect/explore/detail/search-by-lepton

**命令行部署**：

```
lep photon run -n search-with-lepton-modified -m search_with_lepton.py --env BACKEND=BING --env BING_SEARCH_V7_SUBSCRIPTION_KEY=YOUR_BING_SUBSCRIPTION_KEY


```

总结：
===

贾扬清通过 Lepton Search 项目不仅展示了技术实力，更激发了开发者对 AI 应用的无限期待。该项目的成功证明了构建 AI 应用并非遥不可及，而是触手可及的未来。

如果有其他疑问，欢迎朋友关注留言！

我是李孟聊 AI，独立开源软件开发者，SolidUI 作者，对于新技术非常感兴趣，专注 AI 和数据领域，如果对我的文章内容感兴趣，请帮忙关注点赞收藏，谢谢！