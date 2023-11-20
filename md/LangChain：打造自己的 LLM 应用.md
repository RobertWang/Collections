> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU1MzE2NzIzMg==&mid=2247493179&idx=1&sn=b389c412064c2a9d8e64f9d9f32fef60&chksm=fbf456d4cc83dfc2cecfa62f9331f3b2fa75d2081fd47961600fd0605aadff80a2dd5e0660af&scene=21#wechat_redirect)

> 本文通过介绍 LangChain 是什么，LangChain 的核心组件以及 LangChain 在实际场景下的使用方式，希望大家能快速上手 LLM 应用的开发。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1UibHPAanTaZmNVW2yn5bYF7O5pGDH1QmUNa2XCUichFvjlOqV5sfnlHaKRALYlrlRCpZygon1I3PIg/640?wx_fmt=png)

Tech

导读
--

随着 LLM 的技术发展，其在业务上的应用越来越关键，通过 LangChain 大大降低了 LLM 应用开发的门槛。本文通过介绍 LangChain 是什么，LangChain 的核心组件以及 LangChain 在实际场景下的使用方式，希望帮助大家能快速上手 LLM 应用的开发。
















==========================================================================================================================================================================================================================================================================================================================

  

  

**01** 

**LangChain 是什么**

  

  

在今年的敏捷团队建设中，我通过 Suite 执行器实现了一键自动化单元测试。Juint 除了 Suite 执行器还有哪些执行器呢？由此我的 Runner 探索之旅开始了！

LangChain 是一个框架，用于开发由 LLM 驱动的应用程序。可以简单认为是 LLM 领域的 Spring，以及开源版的 ChatGPT 插件系统。核心的 2 个功能为：

1）可以将 LLM 模型与外部数据源进行连接。

2）允许与 LLM 模型与环境进行交互，通过 Agent 使用工具。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVs5xye484dlOp8j6DjIbLkxaJglXZ8YhKx8aN7ENVaI5arUaRugvocg/640?wx_fmt=png)图 1.

  

  

**02** 

 

**LangChain 核心组件**
==================

 

  

  

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目标页面展示到屏幕。

LangChain 提供了各种不同的组件帮助使用 LLM，如下图所示，核心组件有 Models、Indexes、Chains、Memory 以及 Agent。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFV3BGkHgCtXsoePfbw9NKsYIKmF8PpxHg3jm8C1596CVZXYibQia3To2HQ/640?wx_fmt=png)﻿图 2.

**2.3  Models**

  

LangChain 本身不提供 LLM，提供通用的接口访问 LLM，可以很方便的更换底层的 LLM 以及自定义自己的 LLM。主要有 2 大类的 Models：

1）LLM：将文本字符串作为输入并返回文本字符串的模型，类似 OpenAI 的 text-davinci-003；

2）Chat Models：由语言模型支持但将聊天消息列表作为输入并返回聊天消息的模型。一般使用的 ChatGPT 以及 Claude 为 Chat Models。

与模型交互的，基本上是通过给与 Prompt 的方式，LangChain 通过 PromptTemplate 的方式方便构建以及复用 Prompt。

```
from langchain import PromptTemplate
prompt_template = '''作为一个资深编辑，请针对 >>> 和 <<< 中间的文本写一段摘要。
>>> {text} <<<
'''
prompt = PromptTemplate(template=prompt_template, input_variables=["text"])
print(prompt.format_prompt(text="我爱北京天安门"))

```

**2.2  Indexes**

  

索引和外部数据进行集成，用于从外部数据获取答案。如下图所示，主要的步骤有：

1）通过 Document Loaders 加载各种不同类型的数据源,

2）通过 Text Splitters 进行文本语义分割

3）通过 Vectorstore 进行非结构化数据的向量存储

4）通过 Retriever 进行文档数据检索

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFV10tKkn1jkDPZG6Ws113ic5dxNWcIiboCmwKiaolgyq01yOViaUWia0HtLPA/640?wx_fmt=png)图 3.

#### **2.2.1 Document Loaders**

LangChain 通过 Loader 加载外部的文档，转化为标准的 Document 类型。Document 类型主要包含两个属性：page_content 包含该文档的内容。meta_data 为文档相关的描述性数据，类似文档所在的路径等。

如下图所示：LangChain 目前支持结构化、非结构化以及公开以及私有的各种数据

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVpMWxaYnribt3ibto0SDicBwqsTWef8YPG0EMwOdWNw9S4YzjdT9QeZsiaQ/640?wx_fmt=png)图 4.

#### **2.2.2 Text Splitters**

LLM 一般都会限制上下文窗口的大小，有 4k、16k、32k 等。针对大文本就需要进行文本分割，常用的文本分割器为 RecursiveCharacterTextSplitter，可以通过 separators 指定分隔符。其先通过第一个分隔符进行分割，不满足大小的情况下迭代分割。

文本分割主要有 2 个考虑：

1）将语义相关的句子放在一块形成一个 chunk。一般根据不同的文档类型定义不同的分隔符，或者可以选择通过模型进行分割。

2）chunk 控制在一定的大小，可以通过函数去计算。默认通过 len 函数计算，模型内部一般都是使用 token 进行计算。token 通常指的是将文本或序列数据划分成的小的单元或符号，便于机器理解和处理。使用 OpenAI 相关的大模型，可以通过 tiktoken 包去计算其 token 大小。

```
from langchain.text_splitter import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    model_name="gpt-3.5-turb
    allowed_special="all",
    separators=["\n\n", "\n", "。", "，"],
    chunk_size=7000,
    chunk_overlap=0
)
docs = text_splitter.create_documents(["文本在这里"])
print(docs)

```

#### **2.2.3 Vectorstore**

通过 Text Embedding models，将文本转为向量，可以进行语义搜索，在向量空间中找到最相似的文本片段。目前支持常用的向量存储有 Faiss、Chroma 等。

Embedding 模型支持 OpenAIEmbeddings、HuggingFaceEmbeddings 等。通过 HuggingFaceEmbeddings 加载本地模型可以节省 embedding 的调用费用。

```
#通过cache_folder加载本地模型
embeddings = HuggingFaceEmbeddings(model_)
embeddings = embeddings_model.embed_documents(
    [
        "我爱北京天安门!",
        "Hello world!"
    ]
)

```

#### **2.2.4 Retriever**

Retriever 接口用于根据非结构化的查询获取文档，一般情况下是文档存储在向量数据库中。可以调用 get_relevant_documents 方法来检索与查询相关的文档。

```
from langchain import FAISS
from langchain.document_loaders import WebBaseLoader
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
loader = WebBaseLoader("https://in.m.jd.com/help/app/register_info.html")
data = loader.load()
text_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    model_,
    allowed_special="all",
    separators=["\n\n", "\n", "。", "，"],
    chunk_size=800,
    chunk_overlap=0
)
docs = text_splitter.split_documents(data)
#通过cache_folder设置自己的本地模型路径
embeddings = HuggingFaceEmbeddings(model_)
vectorstore = FAISS.from_documents(docs, embeddings)
result = vectorstore.as_retriever().get_relevant_documents("用户注册资格")
print(result)
print(len(result))

```

**2.3  Chains**

  

Langchain 通过 chain 将各个组件进行链接，以及 chain 之间进行链接，用于简化复杂应用程序的实现。其中主要有 LLMChain、Sequential Chain 以及 Route Chain。

#### **2.3.1 LLMChain**

最基本的链为 LLMChain，由 PromptTemplate、LLM 和 OutputParser 组成。LLM 的输出一般为文本，OutputParser 用于让 LLM 结构化输出并进行结果解析，方便后续的调用。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVK4ANNc4KwVC9hjR5lGyYuicxyLTTgfC9fVX5xAvpSIHytwU71wRv6tQ/640?wx_fmt=png)图 5.

类似下面的示例，给评论进行关键词提前以及情绪分析，通过 LLMChain 组合 PromptTemplate、LLM 以及 OutputParser，可以很简单的实现一个之前通过依赖小模型不断需要调优的事情。

```
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate
from langchain.output_parsers import ResponseSchema, StructuredOutputParser
from azure_chat_llm import llm
#output parser
keyword_schema = ResponseSchema()
emotion_schema = ResponseSchema(评论的情绪，正向为1，中性为0，负向为-1")
response_schemas = [keyword_schema, emotion_schema]
output_parser = StructuredOutputParser.from_response_schemas(response_schemas)
format_instructions = output_parser.get_format_instructions()
#prompt template
prompt_template_txt = '''
作为资深客服，请针对 >>> 和 <<< 中间的文本识别其中的关键词，以及包含的情绪是正向、负向还是中性。
>>> {text} <<<
RESPONSE:
{format_instructions}
'''
prompt = PromptTemplate(template=prompt_template_txt, input_variables=["text"],
                        partial_variables={"format_instructions": format_instructions})
#llmchain
llm_chain = LLMChain(prompt=prompt, llm=llm)
comment = "京东物流没的说，速度态度都是杠杠滴！这款路由器颜值贼高，怎么说呢，就是泰裤辣！这线条，这质感，这速度，嘎嘎快！以后妈妈再也不用担心家里的网速了！"
result = llm_chain.run(comment)
data = output_parser.parse(result)
print(f"type={type(data)}, keyword={data['keyword']}, emotion={data['emotion']}")

```

输出：

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVbjpp7u4X84ceQHYXkHhxjMzZiaiaUcDnTxrvApgQf9ohSPEzQN5SF0mQ/640?wx_fmt=png)图 6.

#### **2.3.2 Sequential Chain**

SequentialChains 是按预定义顺序执行的链。SimpleSequentialChain 为顺序链的最简单形式，其中每个步骤都有一个单一的输入 / 输出，一个步骤的输出是下一个步骤的输入。SequentialChain 为顺序链更通用的形式，允许多个输入 / 输出。

```
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate
from langchain.chains import SimpleSequentialChain
first_prompt = PromptTemplate.from_template(
    "翻译下面的内容到中文:"
    "\n\n{content}"
)
# chain 1: 输入：Review 输出：英文的 Review
chain_trans = LLMChain(llm=llm, prompt=first_prompt, output_key="content_zh")
second_prompt = PromptTemplate.from_template(
    "一句话总结下面的内容:"
    "\n\n{content_zh}"
)
chain_summary = LLMChain(llm=llm, prompt=second_prompt)
overall_simple_chain = SimpleSequentialChain(chains=[chain_trans, chain_summary],verbose=True)
content = '''In a blog post authored back in 2011, Marc Andreessen warned that, “Software is eating the world.” Over a decade later, we are witnessing the emergence of a new type of technology that’s consuming the world with even greater voracity: generative artificial intelligence (AI). This innovative AI includes a unique class of large language models (LLM), derived from a decade of groundbreaking research, that are capable of out-performing humans at certain tasks. And you don’t have to have a PhD in machine learning to build with LLMs—developers are already building software with LLMs with basic HTTP requests and natural language prompts.
In this article, we’ll tell the story of GitHub’s work with LLMs to help other developers learn how to best make use of this technology. This post consists of two main sections: the first will describe at a high level how LLMs function and how to build LLM-based applications. The second will dig into an important example of an LLM-based application: GitHub Copilot code completions.
Others have done an impressive job of cataloging our work from the outside. Now, we’re excited to share some of the thought processes that have led to the ongoing success of GitHub Copilot.
'''
result = overall_simple_chain.run(content)
print(f'result={result}')

```

输出：

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVUXdNZoxx1h9k6aVS6zGSRLGhf7w9jzoAL1nL9NyQr3YFjfqhTc93LQ/640?wx_fmt=png)图 7.

#### **2.3.3 Router Chain**

RouterChain 是根据输入动态的选择下一个链，每条链处理特定类型的输入。

RouterChain 由两个组件组成：

1）路由器链本身，负责选择要调用的下一个链，主要有 2 种 RouterChain，其中 LLMRouterChain 通过 LLM 进行路由决策，EmbeddingRouterChain 通过向量搜索的方式进行路由决策。

2）目标链列表，路由器链可以路由到的子链。

初始化 RouterChain 以及 destination_chains 完成后，通过 MultiPromptChain 将两者结合起来使用。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVf0j69OlJrhblFHYVFx2V3oyp7Hb3TMdhbqfHYFLrlGjCdgGI9Y0UnQ/640?wx_fmt=png)图 8.

#### **2.3.4 Documents Chain**

下面的 4 种 Chain 主要用于 Document 的处理，在基于文档生成摘要、基于文档的问答等场景中经常会用到，在后续的落地实践里也会有所体现。

##### **2.3.4.1 Stuff**

StuffDocumentsChain 这种链最简单直接，是将所有获取到的文档作为 context 放入到 Prompt 中，传递到 LLM 获取答案。

这种方式可以完整的保留上下文，调用 LLM 的次数也比较少，建议能使用 stuff 的就使用这种方式。其适合文档拆分的比较小，一次获取文档比较少的场景，不然容易超过 token 的限制。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVriciaZ9dhicG9ONZze3icExmo5NxWxsE5nOjSKjDOrAJkICeg0EiaEDYoaQ/640?wx_fmt=png)图 9.

##### **2.3.4.2 Refine**

RefineDocumentsChain 是通过迭代更新的方式获取答案。先处理第一个文档，作为 context 传递给 llm，获取中间结果 intermediate answer。然后将第一个文档的中间结果以及第二个文档发给 llm 进行处理，后续的文档类似处理。

Refine 这种方式能部分保留上下文，以及 token 的使用能控制在一定范围。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFV7Yj0qh1lOID3mmfSkOEkiam0kqHH5PwNdd0xLTZp85B9VxGVWP5ndRA/640?wx_fmt=png)图 10.

##### **2.3.4.3 MapReduce**

MapReduceDocumentsChain 先通过 LLM 对每个 document 进行处理，然后将所有文档的答案在通过 LLM 进行合并处理，得到最终的结果。

MapReduce 的方式将每个 document 单独处理，可以并发进行调用。但是每个文档之间缺少上下文。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVMcWO44ZdialMWHwia9hzBicTj7jbkNOYNzzR0mYqPibKeRezxwGvaKr70w/640?wx_fmt=png)图 11.

##### **2.3.4.4 MapRerank**

MapRerankDocumentsChain 和 MapReduceDocumentsChain 类似，先通过 LLM 对每个 document 进行处理，每个答案都会返回一个 score，最后选择 score 最高的答案。

MapRerank 和 MapReduce 类似，会大批量的调用 LLM，每个 document 之间是独立处理。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVe4je110roaLibx6ibxLCmonfONsHwgvcC7p1yzFxia7ARfgiadbDAibd61A/640?wx_fmt=png)图 12.﻿

**2.4  Memory**

正常情况下 Chain 无状态的，每次交互都是独立的，无法知道之前历史交互的信息。LangChain 使用 Memory 组件保存和管理历史消息，这样可以跨多轮进行对话，在当前会话中保留历史会话的上下文。Memory 组件支持多种存储介质，可以与 Monogo、Redis、SQLite 等进行集成，以及简单直接形式就是 Buffer Memory。常用的 Buffer Memory 有：

1）ConversationSummaryMemory ：以摘要的信息保存记录

2）ConversationBufferWindowMemory：以原始形式保存最新的 n 条记录

3）ConversationBufferMemory：以原始形式保存所有记录

通过查看 chain 的 prompt，可以发现 {history} 变量传递了从 memory 获取的会话上下文。下面的示例演示了 Memory 的使用方式，可以很明细看到，答案是从之前的问题里获取的。

```
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory
from azure_chat_llm import llm
memory = ConversationBufferMemory()
conversation = ConversationChain(llm=llm, memory=memory, verbose=True)
print(conversation.prompt)
print(conversation.predict(input="我的姓名是tiger"))
print(conversation.predict(input="1+1=?"))
print(conversation.predict(input="我的姓名是什么"))

```

输出：

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFVRslFx7QAuBvCStg5Gq6YiaqkoE0YyLicoZYhVUkel3rdMnSiaYENLHuHQ/640?wx_fmt=png)图 13.

**2.5  Agent**

Agent 字面含义就是代理，如果说 LLM 是大脑，Agent 就是代理大脑使用工具 Tools。目前的大模型一般都存在知识过时、逻辑计算能力低等问题，通过 Agent 访问工具，可以去解决这些问题。目前这个领域特别活跃，诞生了类似 AutoGPT（https://github.com/Significant-Gravitas/AutoGPT）、BabyAGI（https://github.com/yoheinakajima/babyagi）、AgentGPT（https://github.com/reworkd/AgentGPT）等一些优秀的项目。传统使用 LLM，需要给定 Prompt 一步一步的达成目标，通过 Agent 是给定目标，其会自动规划并达到目标。

#### **2.5.1 Agent 核心组件**

Agent：代理，负责调用 LLM 以及决定下一步的 Action。其中 LLM 的 prompt 必须包含 agent_scratchpad 变量，记录执行的中间过程。

Tools：工具，Agent 可以调用的方法。LangChain 已有很多内置的工具，也可以自定义工具。注意 Tools 的 description 属性，LLM 会通过描述决定是否使用该工具。

ToolKits：工具集，为特定目的的工具集合。类似 Office365、Gmail 工具集等。

Agent Executor：Agent 执行器，负责进行实际的执行。

#### **2.5.2 Agent 的类型**

一般通过 initialize_agent 函数进行 Agent 的初始化，除了 llm、tools 等参数，还需要指定 AgentType。

```
agent = initialize_agent(agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
                tools=tools,
                llm=llm,
                verbose=True)
print(agent.agent.llm_chain.prompt.template)

```

该 Agent 为一个 zero-shot-react-description 类型的 Agent，其中 zero-shot 表明只考虑当前的操作，不会记录以及参考之前的操作。react 表明通过 ReAct 框架进行推理，description 表明通过工具的 description 进行是否使用的决策。

其他的类型还有 chat-conversational-react-description、conversational-react-description、react-docstore、self-ask-with-search 等，类似 chat-conversational-react-description 通过 memory 记录之前的对话，应答会参考之前的操作。

可以通过 agent.agent.llm_chain.prompt.template 方法，获取其推理决策所使用的模板。

#### **2.5.3 自定义 Tool**

有多种方式可以自定义 Tool，最简单的方式是通过 @tool 装饰器，将一个函数转为 Tool。注意函数必须得有 docString，其为 Tool 的描述。

```
from azure_chat_llm import llm
from langchain.agents import load_tools, initialize_agent, tool
from langchain.agents.agent_types import AgentType
from datetime import date
@tool
def time(text: str) -> str:
    """
    返回今天的日期。
    """
    return str(date.today())
tools = load_tools(['llm-math'], llm=llm)
tools.append(time)
agent_math = initialize_agent(agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
                                   tools=tools,
                                   llm=llm,
                                   verbose=True)
print(agent_math("计算45 * 54"))
print(agent_math("今天是哪天？"))

```

输出为：

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1VXibTbBqf2xCh1tIWXeiaaFV7LCjK2ibfQjXlfxDPkDt4WfichWRauqALSiaAXlpPQtmaeI4PNT9NgOFA/640?wx_fmt=png)图 14.

  

  

**03** 

 **LangChain 落地实践** 

  

  

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目标页面展示到屏幕。

**3.1  文档生成总结**

  

1）通过 Loader 加载远程文档

2）通过 Splitter 基于 Token 进行文档拆分

3）加载 summarize 链，链类型为 refine，迭代进行总结

```
from langchain.prompts import PromptTemplate
from langchain.document_loaders import PlaywrightURLLoader
from langchain.chains.summarize import load_summarize_chain
from langchain.text_splitter import RecursiveCharacterTextSplitter
from azure_chat_llm import llm
loader = PlaywrightURLLoader(urls=["https://content.jr.jd.com/article/index.html?pageId=708258989"])
data = loader.load()
text_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    model_,
    allowed_special="all",
    separators=["\n\n", "\n", "。", "，"],
    chunk_size=7000,
    chunk_overlap=0
)
prompt_template = '''
作为一个资深编辑，请针对 >>> 和 <<< 中间的文本写一段摘要。
>>> {text} <<<
'''
refine_template = '''
作为一个资深编辑，基于已有的一段摘要：{existing_answer}，针对 >>> 和 <<< 中间的文本完善现有的摘要。
>>> {text} <<<
'''
PROMPT = PromptTemplate(template=prompt_template, input_variables=["text"])
REFINE_PROMPT = PromptTemplate(
    template=refine_template, input_variables=["existing_answer", "text"]
)
chain = load_summarize_chain(llm, chain_type="refine", question_prompt=PROMPT, refine_prompt=REFINE_PROMPT, verbose=False)
docs = text_splitter.split_documents(data)
result = chain.run(docs)
print(result)

```

**3.2  基于外部文档的问答**

  

1）通过 Loader 加载远程文档

2）通过 Splitter 基于 Token 进行文档拆分

3）通过 FAISS 向量存储文档，embedding 加载 HuggingFace 的 text2vec-base-chinese 模型

4）自定义 QA 的 prompt，通过 RetrievalQA 回答相关的问题

```
from langchain.chains import RetrievalQA
from langchain.document_loaders import WebBaseLoader
from langchain.embeddings.huggingface import HuggingFaceEmbeddings
from langchain.prompts import PromptTemplate
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import FAISS
from azure_chat_llm import llm
loader = WebBaseLoader("https://in.m.jd.com/help/app/register_info.html")
data = loader.load()
text_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    model_,
    allowed_special="all",
    separators=["\n\n", "\n", "。", "，"],
    chunk_size=800,
    chunk_overlap=0
)
docs = text_splitter.split_documents(data)
#设置自己的模型路径
embeddings = HuggingFaceEmbeddings(model_)
vectorstore = FAISS.from_documents(docs, embeddings)
template = """请使用下面提供的背景信息来回答最后的问题。 如果你不知道答案，请直接说不知道，不要试图凭空编造答案。
回答时最多使用三个句子，保持回答尽可能简洁。 回答结束时，请一定要说"谢谢你的提问！"
{context}
问题: {question}
有用的回答:"""
QA_CHAIN_PROMPT = PromptTemplate(input_variables=["context", "question"], template=template)
qa_chain = RetrievalQA.from_chain_type(llm, retriever=vectorstore.as_retriever(),
                                       return_source_documents=True,
                                       chain_type_kwargs={"prompt": QA_CHAIN_PROMPT})
result = qa_chain({"query": "用户注册资格"})
print(result["result"])
print(len(result['source_documents']))

```

  

  

**04** 

  **未来发展方向**  

  

  

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将

随着大模型的发展，LangChain 应该是目前最火的 LLM 开发框架，能和外部数据源交互、能集成各种常用的组件等等，大大降低了 LLM 应用开发的门槛。其创始人 Harrison Chase 也和 Andrew Ng 联合开发了 2 门短课程，帮忙大家快速掌握 LangChain 的使用。

目前大模型的迭代升级特别快，作为一个框架，LangChain 也得保持特别快的迭代速度。其开发特别拼，每天都会提交大量的 commit，基本隔几天就会发布一个新版本，其 Contributor 也达到了 1200 多人，特别活跃。

个人认为，除了和业务结合落地 LLM 应用外，还有 2 个大的方向可以进一步去探索：

1）通过低代码的形式进一步降低 LLM 应用的开发门槛。类似 langflow 这样的可视化编排工具发展也很快；

2）打造更加强大的 Agent。Agent 之于大模型，个人觉得类似 SQL 之于 DB，能大幅度提升 LLM 的应用场景。

  

  

**05** 

  **参考资料**  

  

  

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将

1、https://python.langchain.com/docs/get_started/introduction.html﻿

2、https://github.com/liaokongVFX/LangChain-Chinese-Getting-Started-Guide﻿

3、https://www.deeplearning.ai/short-courses/langchain-for-llm-application-development/﻿

4、https://lilianweng.github.io/posts/2023-06-23-agent/﻿

5、[https://mp.weixin.qq.com/s/3coFhAdzr40tozn8f9Dc-w](https://mp.weixin.qq.com/s?__biz=Mzg2OTY0MDk0NQ==&mid=2247501117&idx=1&sn=e860ac5e259a969f62b05d080bf42d14&scene=21#wechat_redirect)﻿

6、https://github.com/langchain-ai/langchain