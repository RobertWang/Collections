> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/eKZaQo764U2wNXfxAhTPeA)

全文大纲
====

1.  Fay- 是一个数字人开源项目
    
2.  bark - 一个基于转换器的文本到音频模型
    
3.  ChatGLM-6B- 支持中英双语的对话语言模型
    
4.  law-cn-ai - 这个开源项目被称为你的 AI 法律助手的开源项目
    
5.  gpt4-pdf-chatbot-langchain 针对 PDF 文件构建的 GPT 机器人
    
6.  MOSS - 国内首个对话式大语言模型开源
    
7.  SQL Chat 是一个基于聊天的 SQL 客户端
    
8.  DeepFloyd IF - 这是一种新颖的最先进的开源文本到图像模型
    

Fay
===

Github：https://github.com/TheRamU/Fay

Fay 是一个完整的开源项目，包含 Fay 控制器及数字人模型，可灵活组合出不同的应用场景：虚拟主播、现场推销货、商品导购、语音助理、远程语音助理、数字人互动、数字人面试官及心理测评、贾维斯、Her。

开发人员可以利用该项目简单地构建各种类型的数字人或数字助理。该项目各模块之间耦合度非常低，包括声音来源、语音识别、情绪分析、NLP 处理、情绪语音合成、语音输出和表情动作输出等模块。每个模块都可以轻松地更换。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icWFCXC5LPgdJicR1PtNNlsdJPBxnoX5zl9Ak4uHkjq0jicjff6E8TVKGg/640?wx_fmt=jpeg)

Fay 控制器用途

**Fay 控制器核心逻辑**

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93ic9DYo6LNn1L0IV3aoPIXOko5LSzicHvUqpMM8IgRwF2uqSibMsdxA4PeQ/640?wx_fmt=jpeg)

Fay 控制器核心逻辑

**使用说明**

*   抖音虚拟主播：启动 bin/Release_2.85/2.85.exe + fay 控制器（抖音输入源开启、展板播放关闭）+ 数字人 + 抖音伴侣（测试时直接通过浏览器打开别人的直播间）；
    
*   现场推销货：fay 控制器（展板播放关闭、填写商品信息）+ 数字人；
    
*   商品导购：fay 控制器（麦克风输入源开启、展板播放关闭、填写商品信息、填写商品 Q&A）+ 数字人；
    
*   语音助理：fay 控制器（麦克风输入源开启、展板播放开启）；
    
*   远程语音助理：fay 控制器（展板播放关闭）+ 远程设备接入；
    
*   数字人互动：fay 控制器（麦克风输入源开启、展板播放关闭、填写性格 Q&A）+ 数字人；
    
*   数字人面试官及心理测评：联系免费领取；
    
*   贾维斯、Her：加入我们一起完成。
    

**语音指令**

*   **关闭核心** 关闭 再见 你走吧
    
*   **静音** 静音 闭嘴 我想静静
    
*   **取消静音** 取消静音 你在哪呢？你可以说话了
    
*   **播放歌曲**（网易音乐库不可用，寻找替代中） 播放歌曲 播放音乐 唱首歌 放首歌 听音乐 你会唱歌吗？
    
*   **暂停播放** 暂停播放 别唱了 我不想听了
    

**图形界面**

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93ichdcVicdPwGrh4ay4q7BVUxMV6PB33AoUlKObG4xlM8nEnvSdHIiauVJw/640?wx_fmt=jpeg)

bark
====

Github： https://github.com/suno-ai/bark

Bark 是由 Suno 创建的一个基于转换器的文本到音频模型。Bark 可以生成高度逼真的多语言语音以及其他音频，包括音乐、背景噪音和简单的音效。该模型还可以产生非语言交流，如大笑、叹息和哭泣。为了支持研究社区，我们正在提供对预先训练的模型检查点的访问，以便进行推理。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icEicUtGxIZITudacowJ80h6dutt0hBviaib2eqTXibgyib7UW97XUYp5lY5Q/640?wx_fmt=jpeg)

ChatGLM-6B
==========

Github： https://github.com/THUDM/ChatGLM-6B

ChatGLM-6B 是一个开源的、支持中英双语的对话语言模型，基于 General Language Model (GLM) 架构，具有 62 亿参数。结合模型量化技术，用户可以在消费级的显卡上进行本地部署（INT4 量化级别下最低只需 6GB 显存）。

ChatGLM-6B 使用了和 ChatGPT 相似的技术，针对中文问答和对话进行了优化。经过约 1T 标识符的中英双语训练，辅以监督微调、反馈自助、人类反馈强化学习等技术的加持，62 亿参数的 ChatGLM-6B 已经能生成相当符合人类偏好的回答。

为了方便下游开发者针对自己的应用场景定制模型，我们同时实现了基于 P-Tuning v2 的高效参数微调方法 (使用指南) ，INT4 量化级别下最低只需 7GB 显存即刻启动微调。

不过，由于 ChatGLM-6B 的规模较小，目前已知其具有相当多的**局限性**，如事实性 / 数学逻辑错误，可能生成有害 / 有偏见内容，较弱的上下文能力，自我认知混乱，以及对英文指示生成与中文指示完全矛盾的内容。请大家在使用前了解这些问题，以免产生误解。更大的基于 1300 亿参数 GLM-130B 的 ChatGLM 正在内测开发中。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icRCrgntx7pibjycLraSqg4lENARTFXsdjQCNL0BlPaUUuQYapNmUib1Zg/640?wx_fmt=jpeg)

ChatGLM-6B Github 主页

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icJsAL72iaQCFkqaDRnCoTlofXG7Vsia7bkCylIr4ibjHwiaaDPk5cKYLtMA/640?wx_fmt=jpeg)

law-cn-ai
=========

官网：https://law-cn-ai.vercel.app/

Github： https://github.com/lvwzhen/law-cn-ai

这个开源项目被称为你的 AI 法律助手的开源项目，通过分析大量的法律文件，通过你的问题给出答案。

但该开源项目不是完全基于大模型去输出结果，而是将法律知识库进行预处理，通过向量相似性搜索来去库中匹配相似性更高的答案，将内容输入到 GPT 中进行补全，最终将结果输出到客户端。

如下图：

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icrItIwCRoTkPTcwvOOAROswSrzYbKL1DafRalbZiaRiaaT35d54Z1SiaXQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93iclVnf5GmSHGvpHv3WTqhtial9zN1ia5Qmu33icicnSPgtYn9mhM3SW4xG9Q/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icbnzYYGUIUXtjkWHMa3arIFYvLJnEAWuGqgOatN39Qod6Or04SGicx5A/640?wx_fmt=jpeg)

gpt4-pdf-chatbot-langchain
==========================

Github： https://github.com/mayooear/gpt4-pdf-chatbot-langchain  

针对 PDF 文件构建的 GPT 机器人，上传你的 PDF 文件，使用的技术堆栈包括 LangChain、Pinecone、Typescript、Openai 和 Next.js。  

基于 Open AI 和 LangChain，可以分析 PDF 文档中的文字和内容，通过 embedding API 生成向量，然后存储到数据库中。  

最后做成类似于 ChatGPT 的机器人，通过机器人快速的进行查询、输出答案。

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icvB118u4d2micY8blicVqv6sIpYsIK4nKCiaF8933sWbic9G8NMRLQrlsLA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93ic51r9eabVHwAYkglqyibH8AohlNBaf9z79ibLA4PLNIicOLebuyhB8uh8Q/640?wx_fmt=jpeg)

MOSS
====

官网：https://txsun1997.github.io/blogs/moss.html

Github： https://github.com/OpenLMLab/MOSS

国内首个对话式大语言模型开源了！复旦大学发布的大模型 MOSS 正式开源，相关代码、数据、模型参数已在 Github 平台开放，供科研人员下载。

MOSS 是一个支持中英双语和多种插件的开源对话语言模型，moss-moon 系列模型具有 160 亿参数，在 FP16 精度下可在单张 A100/A800 或两张 3090 显卡运行，在 INT4/8 精度下可在单张 3090 显卡运行。MOSS 基座语言模型在约七千亿中英文以及代码单词上预训练得到，后续经过对话指令微调、插件增强学习和人类偏好训练具备多轮对话能力及使用多种插件的能力。

**局限性**：由于模型参数量较小和自回归生成范式，MOSS 仍然可能生成包含事实性错误的误导性回复或包含偏见 / 歧视的有害内容，请谨慎鉴别和使用 MOSS 生成的内容，请勿将 MOSS 生成的有害内容传播至互联网。若产生不良后果，由传播者自负。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icpZFrQnG1PNAkWSUYIpDFHdr5TYsJ7c9RfDVkUyTM4RlnOfhAUSLliaQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93ica7UE9ibhE4s2H3RvJgEHFOw0ibsFiazX6BiaImg60Jogm3XC5qBiahsb2ibw/640?wx_fmt=jpeg)

SQL Chat  

官网：https://sqlchat.ai/  

Github： https://github.com/sqlchat/sqlchat  

SQL Chat 是一个基于聊天的 SQL 客户端，你可以像聊天一样，问数据库一些问题，让机器人帮你查询一些数据  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icIJ6CmOmF3kOxQKY2egfqkaFIiabARQaTO4yZpvYkKTajcFVlDLqeSFw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icnmrpqtcQPEOuyiavNdkCmOEibczibXkiaXibT4hYLAwtsMykzIwopRia1BMg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icJuGicVXGRcichicSqJZMIJoslicOwwXsicx4J0l5UjxcFFXWUPO8VibJlSPg/640?wx_fmt=jpeg)

DeepFloyd IF
============

Github： https://github.com/deep-floyd/IF  

这个开源项目有什么稀奇的？AI 画图不是已经有很多产品或者开源项目了吗？还真不是，像我们使用的 Midjourney 等画图软件，是没办法生成准确的文字的。

但是文字是海报上不可或缺的元素，于是 Stability AI 旗下的独立研发团队 DeepFloyd AI Research 开源了这个开源项目，这个项目能准确绘制文字，但目前不支持中文。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icVjMQbbibbzWuMqjpd4HV1jAsSUAWhwicvekQ1ichSqT7C8CSrxOBlmglQ/640?wx_fmt=jpeg)

官方介绍了 DeepFloyd IF，这是一种新颖的最先进的开源文本到图像模型，具有高度的照片真实性和语言理解能力。

DeepFloyd IF 是一个由冻结文本编码器和三个级联像素扩散模块组成的模块：一个基于文本提示生成 64x64 像素图像的基本模型和两个超分辨率模型，每个模型都设计用于生成分辨率不断提高的图像：256x256 像素和 1024x1024 像素。

模型的所有阶段都使用基于 T5 转换器的冻结文本编码器来提取文本嵌入，然后将其输入到通过交叉注意力和注意力池增强的 UNet 架构中。结果是一个高效的模型，其性能优于当前最先进的模型，在 COCO 数据集上实现了 6.66 的零样本 FID 得分。我们的工作强调了更大的 UNet 架构在级联扩散模型的第一阶段的潜力，并描绘了文本到图像合成的前景。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rWKZ4QtbuGnicaibF6xReoNHzibKNjLl93icJMX9BL0Zyd6icLq3cxC2MvcbcibWRfGB1gUI6ChExmDdyIWn3w0MYaSw/640?wx_fmt=jpeg)

最后一台电脑, 一个键盘, 尽情挥洒智慧的人生; 几行数字, 几个字母, 认真编写生活的美好;
===============================================

一 个灵感, 一段程序, 推动科技进步, 促进社会发展

关注微信公众号，免费源码、学习资料、代码获取、加入学习群。