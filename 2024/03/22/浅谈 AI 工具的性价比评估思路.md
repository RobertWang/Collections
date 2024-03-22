> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/prime/story/ai-tools-eval)

> 坚持场景导向、需求导向的原则，而不是陷入对于新热的无尽追求中，浪费不必要的时间和成本。

引言
--

ChatGPT 的火热给行业带来了很多商机。一年多来，各种带着 AI 标签的新产品不断出现，老产品也争先恐后地引入与 AI 相关的功能，让人眼花缭乱。而且，只要和 AI 稍微沾上边的产品，价格都不会便宜。于是，如何鉴别良莠、把钱花在刀刃上，就成了对 AI 感兴趣用户需要做的功课。

为此，本文准备从模型选择、功能设计、定价方式等角度，为读者做出合理选择提供一个思考框架。

**注：**本文主要讨论输出文本的 AI 工具或功能，图片、视频生成等暂不考虑。由于 AI 领域处于高速发展阶段，文中的信息和结论随时间推移可能发生变化，请读者留意。

模型选择
----

模型是 AI 工具的核心。截至 2024 年年初，总的竞争局势仍然是 OpenAI 凭借 GPT-4 领跑，竞争者在陆续实现了追赶和超越 GPT-3.5 的基础上，近两个月还推出了号称与 GPT-4 不分伯仲的模型，例如 2 月的 [Google Gemini 1.5](https://sspai.com/link?target=https%3A%2F%2Fblog.google%2Ftechnology%2Fai%2Fgoogle-gemini-next-generation-model-february-2024%2F)、[Mistral Large](https://sspai.com/link?target=https%3A%2F%2Fmistral.ai%2Fnews%2Fmistral-large%2F) 和 3 月的 [Claude 3 Opus](https://sspai.com/link?target=https%3A%2F%2Fwww.anthropic.com%2Fnews%2Fclaude-3-family) 等。

但是，这些性能主张都是基于基准测试（俗称「跑分」）的结果做出的。一般而言，这些「跑分」都是一组经过挑选的问题集，用来测试模型掌握事实、算数、推理等方面的能力，跑分分数则是模型能够正确作答的百分比。例如，在近期发布 [Mistral Large](https://sspai.com/link?target=https%3A%2F%2Fmistral.ai%2Fnews%2Fmistral-large%2F)、[Claude 3](https://sspai.com/link?target=https%3A%2F%2Fwww.anthropic.com%2Fnews%2Fclaude-3-family) 等模型时，相应厂商都援引了 [MMLU](https://sspai.com/link?target=https%3A%2F%2Fpaperswithcode.com%2Fdataset%2Fmmlu)（测试人文、社科、理工等领域几十个不同主题的知识掌握程度）、[HellaSwag](https://sspai.com/link?target=https%3A%2F%2Fhuggingface.co%2Fdatasets%2FRowan%2Fhellaswag)（测试理解自然语言并根据常识做出推理的能力）、[ARC](https://sspai.com/link?target=https%3A%2F%2Fallenai.org%2Fdata%2Farc)（测试小学水平的科学问题）等测试集作为性能依据。

不难理解，和任何领域的跑分一样，模型跑分也只能作为一个参考：方便看出大致水平区间，但并不适合决出肩并肩的胜负。同时，模型跑分的内容决定了它无法反映个人的实际使用场景，分数更高并不意味着自己的使用体验也会更好。退一步说，即使一些新模型确有 GPT-4 级别的能力，仍然有待下游开发者花时间来适应它们的「脾性」、做针对性调优，才能让潜力充分发挥出来。

因此，在短期内，采用 GPT-4 的工具仍然值得优先考虑。如果感兴趣的工具使用其他模型，则可以参照跑分测试集的思路，收集一组能代表自己兴趣领域和使用场景的问题，作为评估工具的依据。

当然，模型性能也只是一个维度的标准。在实际使用中，还有很多可能影响体验的非性能因素：

*   风格倾向：这看起来是一个很玄乎的说法，但从网上讨论和个人体验看，不同厂商模型的输出确实呈现出不同的风格倾向。这一定程度上是厂商执行内容安全策略尺度不同导致的。就我观察而言，GPT 较为平衡和寡淡；Claude 比较严谨准确，但有时过于保守；Mistral 则更具「创造力」，但经常收放无度。
*   响应速度：这是最能反映模型不是越强大越好的一个维度。在使用 GPT-4 等较为复杂的模型时，可以观察到其思考时间也较长，输出是逐字往外蹦出来的，而较为精简的模型的则可以一口气答复大段内容。量化地看，GPT-3.5 Turbo 的输出效率在每秒 60 个 token 左右，是 GPT-4 Turbo（约 20t/s）的三倍快，而 Gemini Pro 1.0 的输出速度更是可以达到近 150t/s。

![](https://cdn.sspai.com/2024/03/20/1aa42bcece02efc5bd6a40ce440f29d3.png?imageView2/2/format/webp) Claude 3「胆小」到不敢复述简短台词引文

*   长度限制：这就是术语中所说的「上下文窗口」（context window），对于内容总结、信息检索等功能的效果影响很大。如果允许一次性输入的内容太少，在处理长文本时就只能先做切分，从而影响输出的质量和准确性。目前，在长度支持方面具有优势的是 Anthropic，其模型普遍支持 200K tokens 输入，意味着可以一次装进大部头的书；相比之下，GPT-4 和 GPT-3.5 目前分别能支持到 128K 和 16K 输入，分别大致相当于比较简短的书籍和长篇报告。（支持 1M 输入的 Gemini Pro 1.5 目前尚未开放接口。）

![](https://cdn.sspai.com/2024/03/20/08f26d02e9713338d4eea965350fb334.png?imageView2/2/format/webp) Gemini Pro 1.5 宣称可以通过长上下文窗口处理能力，用自然语言从大型代码库中找出特定文件（来源：Google）

下表总结了本文写作时，一些流行模型在上述几个维度的表现情况：

<table><thead><tr><th>模型</th><th class="">长度限制</th><th>输入价格</th><th>输出价格</th><th>输出速度</th></tr></thead><tbody><tr><td>GPT-3.5 Turbo</td><td>16K tkns</td><td>$0.5/M tkns</td><td>$1.5/M tkns</td><td>~60t/s</td></tr><tr><td>GPT-4 Turbo</td><td>128K tkns</td><td>$10/M tkns</td><td>$30/M tkns</td><td>~20t/s</td></tr><tr><td>Claude 3 Haiku</td><td>200K tkns</td><td>$0.25/M tkns</td><td>$1.25/M tkns</td><td>~60t/s</td></tr><tr><td>Claude 3 Opus</td><td>200K tkns</td><td>$15/M tkns</td><td>$75/M tkns</td><td>~20t/s</td></tr><tr><td>Claude 2.1</td><td class="">200K tkns</td><td>$8/M tkns</td><td>$24/M tkns</td><td>~20t/s</td></tr><tr><td>Claude Instant 1.2</td><td>100K tkns</td><td>$0.8/M tkns</td><td>$2.4/M tkns</td><td>~60t/s</td></tr><tr><td>Mistral Small</td><td>32K tkns</td><td>$2/M tkns</td><td>$6/M tkns</td><td>~60t/s</td></tr><tr><td>Mistral Large</td><td class="">32K tkns</td><td>$8/M tkns</td><td>$24/M tkns</td><td>~30t/s</td></tr><tr><td>Gemini Pro 1.0</td><td>128K chars</td><td>$0.125/M chars</td><td>$0.375/M chars</td><td>~80t/s</td></tr></tbody></table>

综合以上，在评价 AI 工具使用的模型时，不应一概而论、追求用上最新最好的模型，而是应该根据任务类型具体判断：如果目的是完成比较简单的网页总结、信息提取、语法检测等任务，那么 GPT-3.5、Claude Instant 这种级别的模型通常就可以胜任了，响应速度也有保证。如果需要处理超长文件、理解复杂需求、辅助创意写作等，使用 GPT-4、Claude 2/3 等复杂模型则更能保证输出质量。

功能设计
----

模型是 AI 工具质量的决定性因素，但并不是唯一因素。事实上，除了 ChatGPT 这种背靠自家模型的第一方产品，其他 AI 工具都面临着一个共同的存在主义难题：最关键的内核不属于自己，某种程度上只是模型输出的搬运工。因此，一个好的 AI 工具不能停留在 API 套壳的肤浅层次（正如市面上无数廉价版、山寨版 ChatGPT 所做的那样），而应该在功能与设计方面反映出独立于下层模型的价值。

**首先，好的 AI 工具应当具备自主技术能力。**虽然大多数 AI 工具开发商不具有从头训练一个模型的条件，但即使在现有模型的基础上，能优化的空间也很大。需要指出的是，仅仅预置一段自己编写的提示词模版（无论多么花哨）并不算是什么独家技术。实践已经反复表明，纯粹基于「提示词工程」获得的优化效果是不稳定且易失效的，其重要性随着模型的进化在逐渐下降，而且任何试图隐藏内置提示词的努力几乎[注定要失败](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fjujumilk3%2Fleaked-system-prompts%2Ftree%2Fmain)。

什么是好的技术能力呢？代码生成工具 GitHub Copilot 可以提供一些示范。尽管基于 OpenAI 的 Codex 模型，但 Copilot 在此之上搭建了复杂的「脚手架」。根据[逆向工程](https://sspai.com/link?target=https%3A%2F%2Fthakkarparth007.github.io%2Fcopilot-explorer%2Fposts%2Fcopilot-internals.html)，Copilot 并不是简单地向模型发送用户指令，而是会动态地提取当前项目中最近打开的文件、相似文件、导入文件、文件路径等信息，与当前光标所在位置的上下文一起发出。对于模型返回的结果，Copilot 还会根据括号是否闭合、用户近期「满意率」等维度判断其质量，以便预先过滤低质量结果。

![](https://cdn.sspai.com/2024/03/20/531372ac158ed8a01b8f01cbc5a49237.png?imageView2/2/format/webp) Copilot 构建的提示词（来源：Joas Pambou）

开发商还可以对现有模型进行「微调」（fune-tuning），即使用自行提供的数据进一步训练模型，使其更符合特定场景或任务的需要。例如，人气很旺的 AI 搜索工具 Perplexity 就基于 Mistral 和 Llama 模型微调出了 [PPLX 模型](https://sspai.com/link?target=https%3A%2F%2Fwww.perplexity.ai%2Fhub%2Fblog%2Fintroducing-pplx-online-llms)，能更好地应对在线搜索和问答场景。

![](https://cdn.sspai.com/2024/03/20/3dda4449d40da88c2b7c21d5b843f669.png?imageView2/2/format/webp) Perplexity 自主微调的模型在搜索相关场景中具有优于通用模型的效果（来源：Perplexity）

**同时，好的 AI 工具应当根据使用场景，有针对性地优化体验。**这意味着工具开发者对于目标用户的使用场景和工作流程有深入的认识。

例如，OpenAI Whisper 是一个很受欢迎的语音转文本模型，理论上可以完全免费使用，GitHub 上也不乏在其基础上做包装的开源项目。但是，同样依赖于该模型的应用 [MacWhisper](https://sspai.com/link?target=https%3A%2F%2Fgoodsnooze.gumroad.com%2Fl%2Fmacwhisper) 即使收费不低，仍然收获了不少称赞和销路。这是因为它充分考虑到了 Whisper 的常见使用场景，把原本需要手动转码的音频输入做得足够简单，并且支持在线视频音轨提取、系统音频内录和语音录制等多种来源；同时，整合了移除填充词、对话者识别、导出字幕、AI 总结和翻译等后期处理功能。正是这些模型之外的辅助功能让 MacWhisper 成了一个值得付费的产品。

![](https://cdn.sspai.com/2024/03/20/7169e731dd6694dd850bdfe4770e6e4a.png?imageView2/2/format/webp)

近期另一个比较好的场景化案例是 Arc 浏览器的移动版 Arc Search。AI 搜索和网页总结并不新鲜，但 Arc Search 找到的切入点直白、明确：「为我浏览」。根据[提取的提示词](https://sspai.com/link?target=https%3A%2F%2Fgist.github.com%2Fdan-palmer%2Fd1e3469c7d4173efd0aeb0a209795f08)可知，Arc Search 首先对用户的关键词做联想和改写，然后抓取六个网络结果汇总，呈现为一份图文并茂、具有总分结构的搜索「报告」。再加上与桌面端 Arc 一脉相承的动效设计，说 Arc Search 是同功能产品中最有说服力、打磨最精致的选项之一，应当是恰如其分的。

![](https://cdn.sspai.com/2024/03/20/22e07ac290a1cbc7f3fabe890fd618f0.png?imageView2/2/format/webp) Arc Search 提示词要求的输出结构

**最后，隐私问题也值得特别关注。**这可能是 AI 工具目前最令人顾虑的方面。作为用户，虽然不能直接对抗这一现实情况，但至少应该了解一般实践，并将隐私承诺纳入选择工具时的主要考虑因素：