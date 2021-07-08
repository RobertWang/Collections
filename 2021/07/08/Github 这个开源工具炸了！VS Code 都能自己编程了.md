> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650808857&idx=4&sn=70319f27a86625d8554c373c53ab1652&chksm=bd01b7578a763e411e64524386953ef0913f8d9033bef072e7d1cce6e94b9daacdfb650de482&mpshare=1&scene=1&srcid=0708yaSv4xbiezxp1V43KjyB&sharer_sharetime=1625740257580&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> **开源最前线（ID：OpenSourceTop） 猿妹编译**
> 
> 链接：https://www.theverge.com/2021/6/29/22555777/github-openai-ai-tool-autocomplete-code

近日，GitHub 和 OpenAI 发布了一个名为 Copilot 的新 AI 工具，该工具能够在 isual Studio Code 编辑器中自动完成代码片段。

根据 GitHub 的说法，Copilot 所做的不仅仅是模仿以前见过的代码。相反，它会分析你已经编写的代码并生成新的匹配代码，包括之前调用的特定函数。示例如下：

根据 GitHub 首席执行官 Nat Friedman 的介绍，它最适合 Python、JavaScript、TypeScript、Ruby 和 Go 。GitHub 将其视为结对编程的演变，其中两名程序员将在同一个项目上工作，以发现彼此的错误并加快开发过程。只是使用 Copilot，其中一位程序员是虚拟的。

该项目是微软向 OpenAI 投资 10 亿美元的第一个主要成果，该研究公司现在由 Y Combinator 总裁 Sam Altman 领导。自从 Altman 上任以来，OpenAI 已经从非营利状态转向了 “利润上限” 模式，接受了微软的投资，并开始授权其 GPT-3 文本生成算法。

Copilot 建立在一种名为 OpenAI Codex 的新算法之上，OpenAI 首席技术官 Greg Brockman 将其描述为 GPT-3 的后代。OpenAI Codex 在人们如何使用代码方面十分成熟，并且在代码生成方面比 GPT-3 强大得多，部分原因是它是在包含大量公共源代码的数据集上进行训练的。

GitHub Copilot 可以根据上下文自动写代码，包括文档字符串、注释、函数名称、代码，只要用户给出提示，就可以写出完整的函数，将程序员从重复的编写中解放出来虽然网站上的评价对 Copilot 是赞不绝口，但是 Github 也提示并非所有使用的代码都经过了错误、不安全实践或个人数据的审查，虽然他们已经设置了一些过滤器来防止 Copilot 出错。

GitHub Copilot 运作流程如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/nXtTCRG74nKAnQXze8mz70J3BgkS3G1CugRvUpoHHHSZ0RlMJTeyEX9ntXTbxFzibNjzWIcEBfaY4E8oLf4GsUQ/640?wx_fmt=png)

了解更多 Copilot：https://copilot.github.com/