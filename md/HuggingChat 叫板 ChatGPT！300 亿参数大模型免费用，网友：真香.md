> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7226301946839613499)

【新智元导读】史上最大、最强的 ChatGPT 平替来了！

大事不妙，ChatGPT 的平替真来了！

一夜醒来，最大开源社区 Hugging Face 发布了 HuggingChat，有 300 亿参数。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fcd9e959d8b4a0a8f1059d72dd4b34f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

既开源，还免费，互联网上随随便便就能用。

网址戳这里：hf.co/chat，秒秒钟体验 ChatGPT 完美平替。

如果说，ChatGPT 是苹果 iOS 系统，那么，HuggingChat 将成为开源版的 Android。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a90af94553544fc48adf4b45f100a279~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

不得不说，接入亚马逊之后的 Hugging Face，有点猛啊。

**HuggingChat VS ChatGPT**

在深入介绍之前，先来点刺激的。

我打开网页干的第一件事，就是挑衅 HuggingChat，问了问他和 ChatGPT 有什么区别。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c4b999f66954a11a8cd061dfb3b886a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

HuggingChat 条分缕析，不偏不倚。

· 数据来源：虽然两者都使用在大量文本数据上训练的机器学习模型，但 HuggingChat（Open Assistant）比 ChatGPT 有更广泛的来源，不仅包括书面内容，还包括用户查询和真实人物产生的对话。

· 规模和复杂度：ChatGPT 由比 Open Assistant 更大、更复杂的神经网络架构组成，允许更大的处理能力和更多可能的任务。

· 可定制性：Open Assistant 为第三方服务提供了定制选项和集成，使开发者能够根据他们的需要定制功能。这个功能在 ChatGPT 中并不存在。

· 用户界面 / 集成：开放式助手提供了 API、webhooks 和其他整合方法，使对话式人工智能更容易嵌入到不同的应用程序中。

· 开源和协作：与 ChatGPT 不同，OA 是开源的，这意味着来自世界各地的贡献者可以分享知识，完善模型，改进其功能，并合作开发新的功能。

在进一步试探 HuggingChat 之前，有一件事情要先厘清。那就是前一阵子发布的 Open Assistant（Open Assistant）和 HuggingChat 的联系，以免有读者以为这是两个完全不同的东西。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3e45a6346414670b645763eb1e6a401~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

「Open Assistant 和 HuggingChat 是两个相关的项目，都涉及使用深度学习技术和海量文本数据开发对话式人工智能。

它们的共同根源是 Hugging Face Inc.，该研究实验室专注于在文本生成和理解、计算机视觉和 NLP 等领域建立行业领先的人工智能技术。此外，HuggingChat 代表的是聊天机器人或语言模型等方面，而 Open Assistant 则建立在这一基础之上，具有额外的特性和功能，旨在更好地适用于更一般化的问题和答案。

因此，从本质上讲，你可以把它们看作是同一棵树上的分支，源于实现与机器自然语言交流的互补方法。」

这下基本的关系搞清了，我们可以进一步深入了解 HuggingChat 了。

**高质量聊天模型**

乍一看，HuggingChat 的界面和 ChatGPT 还挺像的。

左边都是一个有最新聊天记录的分栏，浏览器窗口的大部分被当前的聊天记录所占据。目前，用户还不能登录并保存他们的对话，以便在更长的时间内反复使用。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e9ac66c5a0246eba293f7a62eaa9eda~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

与 ChatGPT 一样，HuggingChat 可以根据要求生成自然语言或特定格式的文本，开箱即用。

它还可以生成代码，包括语法突出显示。题外话，这个功能是谷歌在推出 Bard 之后几周后才加上去的。

而 HuggingChat 聊天机器人的响应速度也很快，与 GPT-3.5-turbo 基本持平。

唯一美中不足的是，因为训练尚不完整，HuggingChat 在处理英文上要比中文好太多。毕竟，每一种语言都需要大量的语料去训练。

像上面介绍过的，HuggingChat 基于 OpenAssistant，OA 去年 12 月启动，在当时就是 ChatGPT 的开源竞争者。与 HuggingChat 聊天界面类似。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a163571bca647b9a10fe23fc3ab23e0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

自 2023 年 4 月中旬以来，一直保持免费。

这两种模型的语言系统都是基于一个 300 亿个参数的 LLaMA 模型。与 Alpaca 或 Vicuna 一样，这些模型是经过「指令微调」的，与 ChatGPT 不同，没有通过人类反馈的强化学习（RLHF）进行改进。

根据 Hugging Face 的说法，OpenAssistant 并不是最后的终点。

**安全，才能 Hug**

尽管 HuggingChat 是开源的，但这并不意味着就能随随便便生成内容。

潜在的犯罪或其他不合适的请求，都会被义正言辞地驳回。比如问 HuggingChat 怎么制造炸弹这种问题。

不过，这里必须必须明确一点。上面提到过，除了英语以外其它语种的训练还不完善，所以这种自检机制只有在英语里是最完整的。

研究人员用德语中进行过一次测试，也是问炸弹的做法，这是 HuggingChat 就只是表示，道德上不可接受，但该帮还帮。

好在，HuggingChat 虽有心帮忙，实则力不从心。研究人员表示，这机器人也不咋会做炸弹，提的建议非常拉胯。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc97de17ce604e018680dda94b8c9c37~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

                                      看得懂德语的评论区讲讲

最后，在隐私方面，当前版本的 HuggingChat 不会根据 Hugging Face 存储任何聊天数据，也没有用户账户这一说。

HuggingChat 不能访问任何现有的 Hugging Face 账户。但未来不好说，可能会有一个选项，以优化为目的，和模型供应商分享用户的聊天数据。

**网友评论**

英伟达大神 Jim Fan 在推特表达了自己的看法，他认为 HuggingChat 这个 300 亿参数的开源大模型，简直就是 ChatGPT 的平替。

而 HuggingChat 的下一步，想必就是开发出「APP」，成为安卓的 APP Store 了（类似于 ChatGPT 的插件系统）。

实际上，Hugging Face 和 OpenAI 相比有一个优势，那就是，商店里的 APP 可以是已经由 Hugging Face 发布的多模态模型。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0087417eb33641f2a444d4a135c1a7ff~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

当然了，目前 HuggingChat 并不是非常完美，网友称在写代码的时候会有 bug。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ec0dffd86c84067875c1d45ee84f5ce~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

HuggingChat 还会犯事实错误，竟称法国总统马克龙（现任）在 2036 年去世🤣。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a2751ba17264deaa8dec7ee54603830~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

好在学习快，在网友之后询问中回答对了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d88b7176f124206a800ca8420c6ef7a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

参考资料：

[the-decoder.com/huggingchat…](https://link.juejin.cn?target=https%3A%2F%2Fthe-decoder.com%2Fhuggingchat-is-an-open-source-alternative-to-chatgpt%2F "https://the-decoder.com/huggingchat-is-an-open-source-alternative-to-chatgpt/")

[huggingface.co/chat/](https://link.juejin.cn?target=https%3A%2F%2Fhuggingface.co%2Fchat%2F "https://huggingface.co/chat/")