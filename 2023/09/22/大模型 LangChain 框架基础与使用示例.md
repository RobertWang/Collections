> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KrWM3cMywMvYUiawRZ94Gg)

作者：kevine

一图胜千言，`LangChain`已经成为当前 LLM 应用框架的事实标准，这篇文章就来对 LangChain 基本概念以及其具体使用场景做一个整理。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQ3KkichibpO1ib4FSVaRDMxvtCvg9h4ofzrZvw16vQFpQHx575YrgkVALA/640?wx_fmt=png)

### LangChain 是什么

`LangChain`是一个基于大语言模型的应用开发框架，它主要通过两种方式规范和简化了使用`LLM`的方式:

1.  **集成**：集成外部数据 (如文件、其他应用、API 数据等) 到`LLM`中；
    
2.  **Agent**：允许`LLM`通过决策与特定的环境交互，并由`LLM`协助决定下一步的操作。
    

LangChain 的优点包括:

1.  **高度抽象的组件**：规范和简化与语言模型交互所需的各种抽象和组件；
    
2.  **高度可自定义的 Chains**：提供了大量预置`Chains`的同时，支持自行继承 BaseChain 并实现相关逻辑以及各个阶段的`callback handler`等；
    
3.  **活跃的社区与生态**：`Langchain`团队迭代速度非常快，能快速使用最新的语言模型特性，该团队也有 langsmith, auto-evaluator 等其它优秀项目，并且开源社区也有相当多的支持。
    
      
    

### LangChain 的主要组件

这是一张`LangChain`的组件与架构图（`langchain python`和`langchain JS/TS`的架构基本一致，本文中以`langchain python`来完成相关介绍），基本完整描述了`LangChain`的组件与抽象层（`callback`不在这张图中，在下方我们会另外介绍），以及它们之间的相关联系。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQFwaleWj7NM44vkibG6nXP3ibiaCPdctf0oVyl9P4lHRpcgC99hI7ZdfoA/640?wx_fmt=png)

#### Model I/O

首先我们从最基本面的部分讲起，Model I/O 指的是和 LLM 直接进行交互的过程。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQxVLq0z9skicfqhTiaUrVOr2GOgicShxdTm2hg6ngdSFUZv9hOIlr69Dhg/640?wx_fmt=png)

在 Model I/O 这一流程中，LangChain 抽象的组件主要有三个：

*   [Language models](https://python.langchain.com/docs/modules/model_io/models/)
    
*   [Prompts](https://python.langchain.com/docs/modules/model_io/prompts/)
    
*   [Output parsers](https://python.langchain.com/docs/modules/model_io/output_parsers/)
    

下面我们展开介绍一下

⚠️ 注：下面涉及的所有代码示例中的`OPENAI_API_KEY`和`OPENAI_BASE_URL`需要提前配置好，`OPENAI_API_KEY`指 **OpenAI/OpenAI 代理服务**的`API Key`，`OPENAI_BASE_URL`指 OpenAI 代理服务的`Base Url`。

##### Language Model

`Language Model`是真正与 LLM / ChatModel 进行交互的组件，它可以直接被当作普通的 openai client 来使用，在`LangChain`中，主要使用到的是`LLM`，`Chat Model`和`Embedding`三类 Language Model。

*   LLM: 最基础的通过 “**text in ➡️ text out**” 模式来使用的 Language Model，另一方面，LangChain 也收录了大量的[**第三方 LLM**](https://python.langchain.com/docs/integrations/llms/)。
    
    ```
    from langchain.llms import OpenAI
    
    llm = OpenAI(model_, openai_api_key=OPENAI_API_KEY, openai_api_base=OPENAI_BASE_URL)
    
    llm("What day comes after Friday?")
    # '\n\nSaturday.'
    
    
    ```
    
*   **Chat Model**: `LLM`的变体，抽象了`Chat`这一场景下的使用模式，由 “**text in ➡️ text out**” 变成了 “**chat messages in ➡️ chat message out**”，`chat message`是指 **text + message type(System, Human, AI)**。
    
    ```
    from langchain.chat_models import ChatOpenAI
    from langchain.schema import HumanMessage, SystemMessage
    
    chat = ChatOpenAI(model_, temperature=1, openai_api_key=OPENAI_API_KEY, openai_api_base=OPENAI_BASE_URL)
    chat(
        [
            SystemMessage(content="You are an expert on large language models and can answer any questions related to large language models."),
            HumanMessage(content="What’s the difference between Generic Language Models, Instruction Tuned Models and Dialog Tuned Models")
        ]
    )
    
    # AIMessage(content='Generic Language Models, Instruction-Tuned Models, and Dialog-Tuned Models are all various types of language models that have been trained according to different datasets and methodologies. They are used in different contexts according to their specific strengths. \n\n1. **Generic Language Models**: These models are trained on a broad range of internet text. They are typically not tuned to specific tasks and thus can be used across a wide variety of applications. GPT-3, used by OpenAI, is an example of a general language model.\n\n2. **Instruction-Tuned Models**: These are language models that are fine-tuned specifically to follow instructions given in a prompt. They are trained using a procedure called Reinforcement Learning from Human Feedback (RLHF), following an initial supervised fine-tuning which consists of human AI trainers providing conversations where they play both the user and an AI assistant. This model often includes comparison data - two or more model responses are ranked by quality.\n\n3. **Dialog-Tuned Models**: Like Instruction-Tuned Models, these models are also trained with reinforcement learning from human feedback, and especially shine in multi-turn conversations. However, these are specifically designed for dialog scenarios, resulting in an ability to maintain more coherent and context-aware conversations. \n\nIn essence, the difference among the three revolves around the breadth of their training and their specific use-cases. Generic Language Models are broad but may lack specificity; Instruction-Tuned Models are better at following specific instructions given in prompts; and Dialog-Tuned Models excel in carrying out more coherent and elongated dialogues.', additional_kwargs={}, example=False)
    
    
    ```
    
    另一方面，LangChain 也收录了大量的[第三方 Chat Model](https://python.langchain.com/docs/integrations/chat/)
    

*   **System** - 告诉 AI 要做什么的背景信息上下文；
    
*   **Human** - 标识用户传入的消息类型；
    
*   **AI** - 标识 AI 返回的消息类型。以下是一个简单的`Chat Model`使用示例：
    

*   Embedding: `Embedding`将一段文字**向量化**为一个定长的向量，有了文本的向量化表示我们就可以做一些像语义搜索，聚类选择等来选择需要的文本片段，如下是将一个 embed 任意字符串的示例：
    
    ```
    from langchain.embeddings import OpenAIEmbeddings
    
    embeddings = OpenAIEmbeddings(openai_api_key=OPENAI_API_KEY, openai_api_base=OPENAI_BASE_URL)
    text_embedding = embeddings.embed_query("To embed text(it can have any length)")
    print (f"Your embedding's length: {len(text_embedding)}")
    print (f"Here's a sample: {text_embedding[:5]}...")
    
    '''
    Your embedding's length: 1536
    Here's a sample: [-0.03194352, 0.009228715, 0.00807182, 0.0077545005, 0.008256923]...
    '''
    
    
    ```
    

##### Prompts

**Prompt** 指用户的一系列指令和输入，是决定`Language Model`输出内容的唯一输入，主要用于帮助模型理解上下文并生成相关和连贯的输出，如回答问题、拓写句子和总结问题。在`LangChain`中的相关组件主要有`Prompt Template`和`Example selectors`，以及后面会提到的辅助 / 补充 **Prompt** 的一些其它组件。

Prompt Template: 预定义的一系列指令和输入参数的`prompt`模版，支持更加灵活的输入，如支持 **output instruction(输出格式指令), partial input(提前指定部分输入参数), examples(输入输出示例)** 等；`LangChain`提供了大量方法来创建`Prompt Template`，有了这一层组件就可以在不同`Language Model`和不同`Chain`下大量复用`Prompt Template`了，`Prompt Template`中也会有下面将提到的`Example selectors, Output Parser`的参与。

Example selectors: 在很多场景下，单纯的 **instruction + input** 的`prompt`不足以让`LLM`完成高质量的推理回答，这时候我们就还需要为`prompt`补充一些针对具体问题的示例，LangChain 将这一功能抽象为了`Example selectors`这一组件，我们可以基于关键字，相似度 (通常使用 **MMR/cosine similarity/ngram** 来计算相似度, 在后面的向量数据库章节中会提到)。为了让最终的`prompt`不超过`Language Model`的 token 上限（各个模型的 token 上限见下表），`LangChain`还提供了`LengthBasedExampleSelector`，根据长度来限制 example 数量，对于较长的输入，它会选择包含较少示例的提示，而对于较短的输入，它会选择包含更多示例。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQLd1IbZibag1rESyOkzVRXRGGwhcyCia51nT1R0oC0TJc5PHia77NNVTOg/640?wx_fmt=png)

  

##### Output Parser

通常我们希望`Language Model`的输出是固定的格式，以支持我们解析其输出为结构化数据，`LangChain`将这一诉求所需的功能抽象成了`Output Parser`这一组件，并提供了一系列的预定义`Output Parser`，如最常用的 **Structured output parser, List parser**，以及在`LLM`输出无法解析时发挥作用的 **Auto-fixing parser 和 Retry parser。**

`Output Parser`需要和 **Prompt Template, Chain** 组合使用：

*   Prompt Template: 在`Prompt Template`中通过指定`partial_variables`为`Output Parser`的 format，即可在`prompt`中补充让模型输出所需格式内容的指令；
    
*   Chain: 在`Chain`中指定`Output Parser`，并使用`Chain`的`predict_and_parse / apply_and_parse`方法启动`Chain`，即可直接输出解析后的数据。
    

##### 使用示例

以下是一个完整的组合 **Prompt Template, Output Parser 和 Chain** 的具体用例：

```
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(temperature=0.5, model_, openai_api_key=OPENAI_API_KEY, openai_api_base=OPENAI_BASE_URL)
template = """
## Input
{text}

## Instruction
Please summarize the piece of text in the input part above.
Respond in a manner that a 5 year old would understand.

{format_instructions}

YOUR RESPONSE:
"""

# 创建一个Output Parser，包含两个输出字段，并指定类型和说明
output_parser = StructuredOutputParser.from_response_schemas(
    [
        ResponseSchema(),
        ResponseSchema(),
    ]
)
# 创建Prompt Template，并将format_instructions通过partial_variables直接指定为Output Parser的format
prompt = PromptTemplate(
    input_variables=["text"],
    template=template,
    partial_variables={"format_instructions": output_parser.get_format_instructions()},
)
# 创建Chain并绑定Prompt Template和Output Parser(它将自动使用Output Parser解析llm输出)
summarize_chain = LLMChain(llm=llm, verbose=True, prompt=prompt, output_parser=output_parser)

to_summarize_text = 'Abstract. Text-to-SQL aims at generating SQL queries for the given natural language questions and thus helping users to query databases. Prompt learning with large language models (LLMs) has emerged as a recent approach, which designs prompts to lead LLMs to understand the input question and generate the corresponding SQL. However, it faces challenges with strict SQL syntax requirements. Existing work prompts the LLMs with a list of demonstration examples (i.e. question-SQL pairs) to generate SQL, but the fixed prompts can hardly handle the scenario where the semantic gap between the retrieved demonstration and the input question is large.'
output = summarize_chain.predict(text=to_summarize_text)

import json
print (json.dumps(output, indent=4))


```

输出如下：

```
{
    "keywords": [
        "Text-to-SQL",
        "SQL queries",
        "natural language questions",
        "databases",
        "prompt learning",
        "large language models",
        "LLMs",
        "SQL syntax requirements",
        "demonstration examples",
        "semantic gap"
    ],
    "summary": "Text-to-SQL is a method that helps users generate SQL queries for their questions about databases. One approach is to use large language models to understand the question and generate the SQL. However, this approach faces challenges with strict SQL syntax rules. Existing methods use examples to teach the language models, but they struggle when the examples are very different from the question."
}


```

#### Data connection

正如我在文章开头的 **LangChain 是什么**一节中提到的，集成外部数据到 Language Model 中是 LangChain 提供的核心能力之一，也是市面上很多优秀的大语言模型应用成功的核心之一（[Github Copilot Chat](https://docs.github.com/en/copilot/github-copilot-chat/using-github-copilot-chat)，[网页聊天助手](https://chrome.google.com/webstore/detail/sider-chatgpt-sidebar-gpt/difoiogjjojoaoomphldepapgpbgkhkb)，[论文总结助手](https://www.summarizepaper.com/)，youtube 视频总结助手…），在`LangChain`中，`Data connection`这一层主要包含以下四个抽象组件：

*   [Document loaders](https://python.langchain.com/docs/modules/data_connection/document_loaders/)
    
*   [Document transformers](https://python.langchain.com/docs/modules/data_connection/document_transformers/)
    
*   [Vector stores](https://python.langchain.com/docs/modules/data_connection/vectorstores/)
    
*   [Retrievers](https://python.langchain.com/docs/modules/data_connection/retrievers/)
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQRxp7VJykKknhMB8mqH55ia4zJWVAf0qtdEOCXc2sqw5DnxGTrMtdvaQ/640?wx_fmt=png)

下面我们展开介绍一下

##### Document loaders

为了补全 LLM 的上下文信息，给予其足够的提示，我们需要从各类数据源获取各类数据，这也就是`LangChain`抽象的`Document loaders`这一组件的功能。

使用`Document loaders`可以将源中的数据加载为`Document`。`Document`由一段文本和相关元数据组成。例如，有用于加载简单. txt 文件的，用于加载相对结构化的 markdown 文件的，用于加载任何网页文本内容，甚至用于加载解析 YouTube 视频的脚本。

同时 LangChain 还收录了海量的[第三方 Document loaders](https://python.langchain.com/docs/integrations/document_loaders/)，以下是一个使用`NotionDBLoader`来加载`notion database`中的`page`为`Document`的示例：

```
from langchain.document_loaders import NotionDBLoader
from getpass import getpass

# getpass()指引用户输入密钥
NOTION_TOKEN = getpass()
DATABASE_ID = getpass()

loader = NotionDBLoader(
    integration_token=NOTION_TOKEN,
    database_id=DATABASE_ID,
    request_timeout_sec=30,  # optional, defaults to 10
)
# 请求详情见 https://developers.notion.com/reference/post-database-query
docs = loader.load()
docs[0].page_content[:100]


```

##### Document transformers

当我们加载 Document 到内存后，我们通常还会希望将他们尽可能的结构化 / 分块，以进行更加灵活的操作。

最简单的例子是，我们很多时候都需要将一个长文档拆分成更小的块，以便放入模型的上下文窗口中；LangChain 有许多内置的 **Document transformers**(大部分都是`Text Spliter`)，可以轻松地拆分、合并、筛选和以其他方式操作文档，一些常用的 **Document transformers** 如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQcrrticGYBoNEoDHAc2YoDz29SKMaJq97wEfhib1Ao9ZnbfzArMn5G7Dw/640?wx_fmt=png)

  

同时 LangChain 也收录了很多[第三方的 **Document transformers**](https://python.langchain.com/docs/integrations/document_transformers/)（如基于爬虫中常见的 beautiful soup, 基于 OpenAI 打 metadata tag 的等等）。

##### Vector stores

在前面的 Prompt 一节中我们提到了 Example selectors，那么我们要如何找到相关示例呢？通常这个答案就是向量数据库。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQ6Dsia24w6ML4XtVic47NzNGtCTQPIpTN7ibvfG8BsNwzrarmzyg2AvFeQ/640?wx_fmt=png)

存储和搜索非结构化数据的最常见方式之一是将其向量化 (embedding) 并存储所得到的嵌入向量，然后在查询时向量化非结构化查询并检索与嵌入的查询最相似的向量。**Vector stores** 负责存储向量化数据并提供向量搜索的功能，常见的向量数据库包括 FAISS, Milvus, Pinecone, Weaviate, Chroma 等，日常使用更常用的是 FAISS。

同时 LangChain 也收录了很多[第三方的 Vector Stores](https://python.langchain.com/docs/integrations/vectorstores/)，提供更加强大的向量搜索等功能。

##### Retrievers

**Retrievers** 是 LangChain 提供的将`Document`与`Language Model`相结合的组件。

LangChain 中有许多不同类型的 Retrievers，但最广泛使用的就是 VectoreStoreRetriever，我们可以直接把它当做连接向量数据库和 Language Model 的中间层，并且 VectoreStoreRetriever 的使用也很简单，直接`retriever = db.as_retriever()`即可。

当然我们还有很多其它的 Retrievers 如 Web search Retrievers 等，LangChain 也收录了很多[第三方的 Retrievers](https://python.langchain.com/docs/integrations/retrievers/)

##### 使用示例 5

以下就是一个使用向量数据库 + VectoreStoreRetriever + QA Chain 的 QA 应用示例：

```
from langchain.chat_models import ChatOpenAI
from langchain.vectorstores import FAISS
from langchain.chains import RetrievalQA
from langchain.document_loaders import TextLoader
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 初始化LLM
llm = ChatOpenAI(temperature=0.5, model_, openai_api_key=OPENAI_API_KEY, openai_api_base=OPENAI_PROXY_URL)

# 加载文档
loader = TextLoader('path/to/related/document')
doc = loader.load()
print (f"You have {len(doc)} document")

print (f"You have {len(doc[0].page_content)} characters in that document")

# 分割字符串
text_splitter = RecursiveCharacterTextSplitter(chunk_size=3000, chunk_overlap=400)
docs = text_splitter.split_documents(doc)
num_total_characters = sum([len(x.page_content) for x in docs])
print (f"Now you have {len(docs)} documents that have an average of {num_total_characters / len(docs):,.0f} characters (smaller pieces)")

# 初始化向量化模型
embeddings = OpenAIEmbeddings(openai_api_key=OPENAI_API_KEY, openai_api_base=OPENAI_PROXY_URL)

# 向量化Document并存入向量数据库（绑定向量和对应Document元数据），这里我们选择本地最常用的FAISS数据库
# 注意: 这会向OpenAI产生请求并产生费用
doc_search = FAISS.from_documents(docs, embeddings)

qa = RetrievalQA.from_chain_type(llm=llm, chain_type="stuff", retriever=doc_search.as_retriever(), verbose=True)
query = "Specific questions to be asked"
qa.run(query)


```

执行后 verbose 输出日志如下：

> You have 1 document  
> You have 74663 characters in that document  
> Now you have 29 documents that have an average of 2,930 characters (smaller pieces)  
> 
> **  
> > Entering new chain...  
> **
> 
> Prompt after formatting:  
> 
> **_  
> System: Use the following pieces of context to answer the users question. If you don't know the answer, just say that you don't know, don't try to make up an answer.  
> ---------------_**
> 
> **_  
> Human: What does the author describe as good work?  
> _**
> 
> **_  
> The author describes working on things that aren't prestigious as a sign of good work. They believe that working on unprestigious types of work can lead to the discovery of something real and that it indicates having the right kind of motives. The author also mentions that working on things that last, such as paintings, is considered good work._**

#### Chains

接下来就是 LangChain 中的主角——Chains 了，Chains 是 LangChain 中为连接多次与 Language Model 的交互过程而抽象的重要组件，它可以将多个组件组合在一起以创建一个单一的、连贯的任务，也可以嵌套多个 Chain 组合在一起，或者将 Chain 与其他组件组合来构建更复杂的 Chain。

除了单一 Chain 外，常用的几个 Chains 如下：

##### Router Chain

在一些场景下，我们需要根据输入 / 上下文决定使用哪一条 Chain，甚至哪一个 Prompt，Router Chain 就提供了诸如 [MultiPromptChain](https://api.python.langchain.com/en/latest/chains/langchain.chains.router.multi_prompt.MultiPromptChain.html), [LLMRouterChain](https://api.python.langchain.com/en/latest/chains/langchain.chains.router.llm_router.LLMRouterChain.html) 等一系列用于决策的 Chain（这与后面我们会提到的 Agent 有类似的地方）。

Router Chain 由两部分组成：

*   Router Chain 本身：负责决定下一个目标 Chain；
    
*   目标 Chains：Router Chain 可以路由到的目标 Chain。
    

##### Sequential Chain

顾名思义，顺序执行的串行 Chain，其中最简单的 SimpleSequentialChain 非常简单粗暴，SimpleSequentialChain 的每个子 Chain 都有一个单一的输入 / 输出，并且一个步骤的输出是下一步的输入。

而高阶一些的 SequentialChain 则允许多输入输出，并且我们可以通过添加后面会提到的 Memory 等来提高其推理表现。

##### Map-reduce Chain

Map-reduce Chain 主要用于 summary 的场景，针对那些超长的文档，首先我们通过前面提到过的 TextSpliter 按一定规则分割文档为更小的 Chunks（通常使用 RecursiveCharacterTextSplitter，如果 Document 是结构化的可以考虑使用指定的 TextSpliter）, 然后对每个分割的部分执行”map-chain”，收集全部”map-chain” 的输出后，再执行”reduce-chain”，获得最终的 summary 输出。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQbfQqGUeczEEUBASeDbiaJYIeM8wOXPNibbQEMDyI8SzM1M3ibxFd96dmw/640?wx_fmt=png)

##### 使用示例

下面就是一个 Router Chain 中的 [MultiPromptChain](https://api.python.langchain.com/en/latest/chains/langchain.chains.router.multi_prompt.MultiPromptChain.html) 的具体示例：

```
from langchain.chains.router import MultiPromptChain
from langchain.llms import OpenAI
from langchain.chains import ConversationChain
from langchain.chains.llm import LLMChain
from langchain.prompts import PromptTemplate
from langchain.chains.router.llm_router import LLMRouterChain, RouterOutputParser
from langchain.chains.router.multi_prompt_prompt import MULTI_PROMPT_ROUTER_TEMPLATE

# 定义要路由的prompts
physics_template = """You are a very smart physics professor. \
You are great at answering questions about physics in a concise and easy to understand manner. \
When you don't know the answer to a question you admit that you don't know.

Here is a question:
{input}"""

math_template = """You are a very good mathematician. You are great at answering math questions. \
You are so good because you are able to break down hard problems into their component parts, \
answer the component parts, and then put them together to answer the broader question.

Here is a question:
{input}"""

# 整理prompt和相关信息
prompt_infos = [
    {
        "name": "physics",
        "description": "Good for answering questions about physics",
        "prompt_template": physics_template,
    },
    {
        "name": "math",
        "description": "Good for answering math questions",
        "prompt_template": math_template,
    },
]

llm = OpenAI(temperature=0.5, openai_api_key=OPENAI_API_KEY, openai_api_base=OPENAI_PROXY_URL+"/v1")
destination_chains = {}
for p_info in prompt_infos:
    # 以每个prompt为基础创建一个destination_chain(开启verbose)
    name = p_info["name"]
    prompt_template = p_info["prompt_template"]
    prompt = PromptTemplate(template=prompt_template, input_variables=["input"])
    chain = LLMChain(llm=llm, prompt=prompt)
    destination_chains[name] = chain
# 创建一个缺省chain，如果没有其他chain满足路由条件，则使用该chain
default_chain = ConversationChain(llm=llm, output_key="text")

destinations = [f"{p['name']}: {p['description']}" for p in prompt_infos]
destinations_str = "\n".join(destinations)
# 根据prompt_infos中的映射关系创建router_prompt
router_template = MULTI_PROMPT_ROUTER_TEMPLATE.format(destinations=destinations_str)
router_prompt = PromptTemplate(
    template=router_template,
    input_variables=["input"],
    output_parser=RouterOutputParser(),
)
# 创建router_chain(开启verbose)
router_chain = LLMRouterChain(llm_chain=LLMChain(llm=llm, prompt=router_prompt, verbose=True), verbose=True)

# 将router_chain和destination_chains以及default_chain组合成MultiPromptChain(开启verbose)
chain = MultiPromptChain(
    router_chain=router_chain,
    destination_chains=destination_chains,
    default_chain=default_chain,
    verbose=True,
)
# run
chain.run("What is black body radiation?")


```

执行后 verbose 输出日志如下：

> **Entering new chain...**Prompt after formatting: Given a raw text input to a language model select the model prompt best suited for the input. You will be given the names of the available prompts and a description of what the prompt is best suited for. You may also revise the original input if you think that revising it will ultimately lead to a better response from the language model.
> 
> <<FORMATTING>> Return a markdown code snippet with a JSON object formatted to look like:
> 
> ```
> {
>     "destination": string \ name of the prompt to use or "DEFAULT"
>     "next_inputs": string \ a potentially modified version of the original input
> }
> 
> 
> ```
> 
> REMEMBER: "destination" MUST be one of the candidate prompt names specified below OR it can be "DEFAULT" if the input is not well suited for any of the candidate prompts. REMEMBER: "next_inputs" can just be the original input if you don't think any modifications are needed.  
> 
>   
> <<CANDIDATE PROMPTS>> physics: Good for answering questions about physics math: Good for answering math questions  
> 
>   
> <<INPUT>>  
> What is black body radiation?  
> 
>   
> <<OUTPUT>>  
> 
>  **Finished chain.**
> 
> physics: {'input': 'What is black body radiation?'}  
> 
> **  
> > Entering new chain...  
> **
> 
> Prompt after formatting:  
> 
> You are a very smart physics professor. You are great at answering questions about physics in a concise and easy to understand manner. When you don't know the answer to a question you admit that you don't know.  
> 
> Here is a question:  
>  What is black body radiation?  
> 
> **  
> > Finished chain.**

#### Memory

`Memory`可以帮助`Language Model`补充历史信息的上下文，`LangChain`中的`Memory`是一个有点模糊的术语，它可以像记住你过去聊天过的信息一样简单，也可以结合向量数据库做更加复杂的历史信息检索，甚至维护相关实体及其关系的具体信息，这取决于具体的应用。

通常 Memory 用于较长的 Chain，能一定程度上提高模型的推理表现。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQe8KkibjqicNjNttRiaFiavSd9X97MkibmqSzQxRoJCbN1MO206rqcl3uW5w/640?wx_fmt=png)

常用的 Memory 类型如下：

*   **Chat Messages**
    
    最简单 Memory，将历史 Chat 记录作为补充信息放入 prompt 中.
    
*   **Vector store-backed memory**
    
    基于向量数据库的 Memory，将 memory 存储在向量数据库中，并在每次调用时查询 **TopK 的最 “重要” 的文档。**
    
    这与大多数其他内存类的不同之处在于，它不显式跟踪交互的顺序，最多基于部分元数据筛选一下向量数据库的查询范围
    
*   **Conversation buffer(window) memory** 保存一段时间内的历史 Chat 记录，它只使用**最后 K 个记录（仅保持最近交互的滑动窗口）**，这样 buffer 也就不会变得太大，避免超过 token 上限。
    
*   **Conversation summary memory**
    
    这种类型的 Memory 会随着 Chat 的进行创建对话的摘要，并将当前摘要存储在 Memory 中，用于后续对话的 history 提示；这种 memory 方案对长会话非常有用，**但频繁的总结摘要会耗费大量的 token**.
    
*   **Conversation Summary Buffer Memory**
    
    结合了`buffer memory`和`summary memory`的策略，依旧会在内存中保留最后的一些 Chat 记录作为 buffer，并在 buffer 的总 token 数达到预置的上限后，**对所有 Chat 记录总结摘要作为 SystemMessage 并清理其它历史 Messages**；这种 memory 方案结合了`buffer memory`和`summary memory`的优点，**既不会频繁地总结摘要消耗 token，也不会让 buffer 缺失过多信息**。
    

##### 使用示例

下面是一个`Conversation Summary Buffer Memory`在`ConversationChain`中的使用示例，包含了**切换会话时恢复现场 memory** 的方法以及**自定义 summary prompt** 的方法：

```
from langchain.chains import ConversationChain
from langchain.memory import ConversationSummaryBufferMemory
from langchain.llms import OpenAI
from langchain.schema import SystemMessage, AIMessage, HumanMessage
from langchain.memory.prompt import SUMMARY_PROMPT
from langchain.prompts import PromptTemplate

llm = OpenAI(temperature=0.7, openai_api_key=OPENAI_API_KEY, openai_api_base=OPENAI_PROXY_URL+"/v1")
# ConversationSummaryBufferMemory默认使用langchain.memory.prompt.SUMMARY_PROMPT作为summary的PromptTemplate
# 如果对它summary的格式/内容有特殊要求，可以自定义PromptTemplate（实测默认的summary有些流水账）
prompt_template_str = """
## Instruction
Progressively summarize the lines of conversation provided, adding onto the previous summary returning a new concise and detailed summary.
Don't repeat the conversation directly in the summary, extract key information instead.

## EXAMPLE
Current summary:
The human asks what the AI thinks of artificial intelligence. The AI thinks artificial intelligence is a force for good.

New lines of conversation:
Human: Why do you think artificial intelligence is a force for good?
AI: Because artificial intelligence will help humans reach their full potential.

New summary:
The human inquires about the AI's opinion on artificial intelligence. The AI believes that it is a force for good as it can help humans reach their full potential.

## Current summary
{summary}

## New lines of conversation
{new_lines}

## New summary

"""
prompt = PromptTemplate(
    input_variables=SUMMARY_PROMPT.input_variables, # input_variables为SUMMARY_PROMPT中的input_variables不变
    template=prompt_template_str, # template替换为上面重新编写的prompt_template_str
)
memory = ConversationSummaryBufferMemory(llm=llm, prompt=prompt, max_token_limit=60)
# 添加历史memory，其中第一条SystemMessage为历史对话中Summary的内容，第二条HumanMessage和第三条AIMessage为历史对话中最后的对话内容
memory.chat_memory.add_message(SystemMessage(content="The human asks what the AI thinks of artificial intelligence. The AI thinks artificial intelligence is a force for good because it will help humans reach their full potential. The human then asks the difference between python and golang in short. The AI responds that python is a high-level interpreted language with an emphasis on readability and code readability, while golang is a statically typed compiled language with a focus on concurrency and performance. Python is typically used for general-purpose programming, while golang is often used for building distributed systems."))
memory.chat_memory.add_user_message("Then if I want to build a distributed system, which language should I choose?")
memory.chat_memory.add_ai_message("If you want to build a distributed system, I would recommend golang as it is a statically typed compiled language that is designed to facilitate concurrency and performance.")
# 调用memory.prune()确保chat_memory中的对话内容不超过max_token_limit
memory.prune()
conversation_with_summary = ConversationChain(
    llm=llm,
    # We set a very low max_token_limit for the purposes of testing.
    memory=memory,
    verbose=True,
)
# memory.prune()会在每次调用predict()后自动执行
conversation_with_summary.predict(input="Is there any well-known distributed system built with golang?")
conversation_with_summary.predict(input="Is there a substitutes for Kubernetes in python?")


```

执行后 verbose 输出日志如下：

> **> Entering new chain...**
> 
> Prompt after formatting:
> 
> The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.
> 
> Current conversation: System: The human asks the AI about its opinion on artificial intelligence and is told that it is a force for good that can help humans reach their full potential. The human then inquires about the differences between python and golang, with the AI explaining that python is a high-level interpreted language for general-purpose programming, while golang is a statically typed compiled language often used for building distributed systems. Human: Then if I want to build a distributed system, which language should I choose? AI: If you want to build a distributed system, I would recommend golang as it is a statically typed compiled language that is designed to facilitate concurrency and performance. Human: Is there any well-known distributed system built with golang?  
> AI:
> 
> **> Finished chain.**
> 
> **> Entering new chain...**
> 
> Prompt after formatting:
> 
> The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.
> 
> Current conversation: System: The human asks the AI about its opinion on artificial intelligence and is told that it is a force for good that can help humans reach their full potential. The human then inquires about the differences between python and golang, with the AI explaining that python is a high-level interpreted language for general-purpose programming, while golang is a statically typed compiled language designed to facilitate concurrency and performance, thus better suited for distributed systems. The AI recommends golang for building distributed systems. Human: Is there any well-known distributed system built with golang? AI: Yes, there are several well-known distributed systems built with golang. These include Kubernetes, Docker, and Consul. Human: Is there a substitutes for Kubernetes in python?  
> AI:
> 
> **> Finished chain.  
> **
> 
> **_'Yes, there are several substitutes for Kubernetes in python. These include Dask, Apache Mesos and Marathon, and Apache Aurora.’_**

#### Agent

在一些场景下，我们需要根据用户输入灵活地调用`LLM`和其它工具（`LangChain`将工具抽象为 Tools 这一组件），Agent 为这样的应用程序提供了相关的支持。

`Agent`可以访问一套工具，并根据用户输入确定要使用`Chain`或是`Function`，我们可以简单的理解为他可以动态的帮我们选择和调用 Chain 或者已有的工具。

常用的`Agent`类型如下：

##### Conversational Agent

这类 Agent 可以根据 Language Model 的输出决定是否使用指定的 Tool，以及使用什么 Tool(这里的 Tool 也可以是一个 Chain)，以及时的为 Model I/O 的过程补充信息。

##### OpenAI functions Agent

类似 Conversational Agent，但它能够让 Agent 更进一步地帮忙提取指定 Tool 的参数等，甚至使用多个 Tools。

##### Plan and execute Agent

抽象 Agent“决定做什么”的过程为 “planning what to do” 和“executing the sub tasks”(这种方法来自 ["Plan-and-Solve"](https://arxiv.org/abs/2305.04091) 这一篇论文)，其中 “planning what to do” 这一步通常完全由 LLM 完成，而 “executing the sub tasks” 这一任务则通常由更多的 Tools 来完成。

### ReAct Agent

结合对 LLM 输出的归因和执行，类似 OpenAI functions Agent，提供了一个更加明确的框架以及由论文支撑的方法。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasYagBKCYYRehau4XltHicNQicdPRec14tVibJxFp48ObKfqn0Wp9QRRyxo5w5OYHlEagwiax1j9yaazA/640?wx_fmt=png)

##### **Self ask with search**

这类 Agent 会基于 LLM 的输出，自行调用 Tools 以及 LLM 来进行额外的搜索和自查，以达到拓展和优化输出的目的。

### **总结**

LangChain 中的 Data Connection 将 LLM 与万物互联，给 LangChain 构建的应用带来了无限可能，而 Agent 又为应用开发中非常常见的” 事件驱动 “这一开发框架提供了偷懒的途径，将分支决策的工作交给 LLM，这又进一步简化了应用开发的工作；结合基于高度抽象的 Model I/O 及 Memory 等组件，LangChain 让开发者能够更快，更好更灵活地实现 LLM 助手、对话机器人等应用，极大地降低了 LLM 的使用门槛。