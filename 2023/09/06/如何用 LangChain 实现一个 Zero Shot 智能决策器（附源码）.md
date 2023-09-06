> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0hfbgp50GQSJwPlTJYhhnw)

> 最近一直在研究 Agent 和 Tool 的使用，今天给大家带来，如何用 LangChain 实现一个 Zero Shot 智能决策器

写在前面
====

大家好，我是刘聪 NLP。

最近一直在研究 Agent 和 Tool 的使用，今天给大家带来一篇何枝大佬（知乎 @何枝）的文章《如何用 LangChain 实现一个 Zero Shot 智能决策器》，并附上源码。

知乎：https://zhuanlan.zhihu.com/p/627333499

LangChain 是当下非常热门的一个库，其通过融合 LLM 的强大能力能够让我们快速地搭建一个具备 “思考能力” 的 AI 助手。在今天的文章中，我们将侧重于讲解：LangChain 是如何进行自我思考并做出决策的。

Github：https://github.com/hwchase17/langchain

官方文档：LangChain 0.0.160

假设今天我们有一个朋友过生日，你很头疼，你不知道送什么礼物给他。这时候，你希望有一个机器人能告诉你一些推荐，如何实现这样一个机器人呢？

人在分析问题时候的思维
===========

**如果今天需要你自己去思考送什么礼物，你可能会这么想：**

我这个朋友是女生。

女生可能会喜欢一些口红、护肤品等等。

好！那我就送她一只口红吧！

可以看出，人们在得到一个问题的最终答案前，通常是会有若干个「中间步骤（intermediate steps）」，LangChain 也是这样！在人们给出一个问题后，LangChain 会引导 LLM 进行一步一步的分析，最终得到一个确定的答案。

**下面，我们就将使用 LangChain + ChatGPT 来复现上述的思考步骤。当我们问两个不同的朋友时，模型会给出不同的推荐礼物：**

**朋友 1**

User: 帮我推荐几个送给小红的礼物

Model: 为小红推荐的礼物有：夏季碎花裙、轻盈帆布包、琉璃唇釉。

**朋友 2**

User: 我想送点礼物给张三

Model: 根据张三的性别和兴趣，我可以推荐 Steam 爆款、RTX-3090 或 iPhone 14 作为礼物。

思维链中的工具设计
=========

我们先分析一下，上述思维链中一共需要两个功能：

找出人物的性别（张三是男的，小红是女生）

根据人物的性别推荐对应的商品（男生喜欢数码产品，女生喜欢化妆品）

尽管 LLM 本身就存在一定的性别推理、商品推荐的能力，但为了更好的利用本地的数据，LangChain 提供了 Tool 类，能够更方便我们使用本地数据，以便于更好的对齐业务需求。

我们可以通过以下方式来定义这两个工具：

```
from langchain.agents import Tool


def find_gender(name: str):
    """
    模拟本地数据库查询。

    Args:
        name (str): 人物名称，由LLM提取。

    Returns:
        _type_: _description_
    """
    info = {                     # 模拟数据库，存放人物对应的性别
        '张三': '男',
        '小红': '女'
    }
    return info.get(name, f'未找到{name}的性别信息，我应该直接返回 Observation: 未知')

tool_of_find_gender = Tool(
    name = "查询人物性别",                         # 工具的名称
    func=find_gender,                            # 工具的实现函数
    description="通过人名查找该人物的性别时用的工具，输入应该是人物的名字"   # 【重要】描述工具的用途，LLM 会根据描述来决定是否使用这个工具
)


def recommend_item(gender: str):
    """
    根据人物性别推荐不同的商品。

    Args:
        gender (str): 人物的性别，由 LLM 提取。

    Returns:
        _type_: _description_
    """
    recommend = {
        '男': ['Steam爆款', 'RTX-9090', 'iPhone 80'],
        '女': ['夏季碎花裙', '轻盈帆布包', '琉璃唇釉'],
        '未知': ['AJ新品', '手冲咖啡']
    }
    return recommend.get(gender, f'未找到合适的推荐商品，我应该返回 Final Answer: 随便买些什么吧，只要消费就能快乐！')

tool_of_recommend_item = Tool(
    name = "根据性别推荐商品",
    func=recommend_item,
    description="当知道了一个人性别后想进一步获得他可能感兴趣的商品时用的工具，输入应该是人物的性别"
)

```

其中，description 和 name 非常重要，这将帮助 LLM 参考在何时应该调用这个工具。

思维链的 Prompt 设计
==============

在有了工具后，我们就要正式开始引导模型进行分析了。

首先我们定义整个 prompt 模板：

```
"""
按照给定的格式回答以下问题。
你可以使用下面这些工具：

{tools}

回答时需要遵循以下用---括起来的格式：

---
Question: 我需要回答的问题
Thought: 回答这个上述我需要做些什么
Action: ”{tool_names}“ 中的其中一个工具名
Action Input: 选择工具所需要的输入
Observation: 选择工具返回的结果
...（这个思考/行动/行动输入/观察可以重复N次）
Thought: 我现在知道最终答案
Final Answer: 原始输入问题的最终答案
---

现在开始回答，记得在给出最终答案前多按照指定格式进行一步一步的推理。

Question: {input}
{agent_scratchpad}
"""

```

这个模版中，最重要的是：

*   Thought
    
*   Action
    
*   Action Input
    

这 3 个部分是 AI 在思维链中进行自我思考的过程，我们可以直观来看看 LLM 具体是怎么思考的：

```
>>> 按照给定的格式回答以下问题，你可以使用下面这些工具：

查询人物性别: 通过人名查找该人物的性别时用的工具，输入应该是人物的名字
根据性别推荐商品: 当知道了一个人性别后想进一步获得他可能感兴趣的商品时用的工具，输入应该是人物的性别

回答时需要遵循以下用---括起来的格式：

---
Question: 我需要回答的问题
Thought: 回答这个上述我需要做些什么
Action: ”查询人物性别, 根据性别推荐商品“ 中的其中一个工具名
Action Input: 选择工具所需要的输入
Observation: 选择工具返回的结果
...（这个思考/行动/行动输入/观察可以重复N次）
Thought: 我现在知道最终答案
Final Answer: 原始输入问题的最终答案
---

现在开始回答，记得在给出最终答案前多按照指定格式进行一步一步的推理。

Question: 我想送点礼物给张三


LLM Output >>> Thought: 我需要知道张三的性别，才能更好地推荐礼物          # 模型的第一次输出
Action: 查询人物性别
Action Input: 张三

现在开始回答，记得在给出最终答案前多按照指定格式进行一步一步的推理。

Question: 我想送点礼物给张三


LLM Output >>> Thought: 我需要知道张三的性别，才能更好地推荐礼物          # 模型的第一次输出
Action: 查询人物性别
Action Input: 张三

```

可以看到，模型进行了第一次思考，它认为解决这个问题首先需要 “确定人物性别”。

而模型之所以能够想到先 “确认性别”，其实是因为我们在 prompt 的最开始告诉了模型：

```
你可以使用下面这些工具：
查询人物性别: 通过人名查找该人物的性别时用的工具，输入应该是人物的名字
...

```

模型通过可用工具的描述来推测出自己当前应该怎么决策，所以文章前面也有提到：工具名 + 工具描述非常的重要！

现在，模型已经决定了要调用 find_gender 函数（Action），并且决定了函数的输入（Action Input）为「张三」，那么，LangChain 就会调用 find_gender("张三") 这个函数，得到结果为：男。

于是，我们的 prompt 模板就需要更新啦：

```
>>> 照给定的格式回答以下问题，你可以使用下面这些工具：

查询人物性别: 通过人名查找该人物的性别时用的工具，输入应该是人物的名字
根据性别推荐商品: 当知道了一个人性别后想进一步获得他可能感兴趣的商品时用的工具，输入应该是人物的性别

回答时需要遵循以下用---括起来的格式：

---
Question: 我需要回答的问题
Thought: 回答这个上述我需要做些什么
Action: ”查询人物性别, 根据性别推荐商品“ 中的其中一个工具名
Action Input: 选择工具所需要的输入
Observation: 选择工具返回的结果
...（这个思考/行动/行动输入/观察可以重复N次）
Thought: 我现在知道最终答案
Final Answer: 原始输入问题的最终答案
---

现在开始回答，记得在给出最终答案前多按照指定格式进行一步一步的推理。

Question: 我想送点礼物给张三

Thought: 我需要知道张三的性别，才能更好地推荐礼物       # 新添加的内容，这部分内容会随着思考不断增加
Action: 查询人物性别                               # 上一步使用的工具
Action Input: 张三                                # 上一步使用工具的输入
Observation: 男                                  # 上一步使用工具后得到的结果
Thought:

```

可以看到，在模板的最后添加了上一步模型思考的内容（Thought）、调用的工具（Action）、以及调用工具得到的结果（Observation）。

现在我们知道「性别」了，让模型接着往下思考吧：

>>> 照给定的格式回答以下问题，...

```
>>> 照给定的格式回答以下问题，...
...
Thought: 我需要知道张三的性别，才能更好地推荐礼物       # 新添加的内容，这部分内容会随着思考不断增加
Action: 查询人物性别                               # 上一步使用的工具
Action Input: 张三                                # 上一步使用工具的输入
Observation: 男                                  # 上一步使用工具后得到的结果
Thought:

LLM Output >>> 现在我知道了张三的性别，我可以根据性别推荐商品   # 接着上面的 prompt 做续写
Action: 根据性别推荐商品
Action Input: 男

```

模型现在决定调用 recommend_item() 函数，并且决定函数输入为「男」，调用函数 recommend_item('男') 得到推荐结果：['Steam 爆款', 'RTX-9090', 'iPhone 80']

随后更新 prompt 模版，并喂给模型进行下一步决策：

```
>>> 按照给定的格式回答以下问题。你可以使用下面这些工具：
...
现在开始回答，记得在给出最终答案前多按照指定格式进行一步一步的推理。

Question: 我想送点礼物给张三
Thought: 我需要知道张三的性别，才能更好地推荐礼物

Action: 查询人物性别
Action Input: 张三
Observation: 男
Thought: 现在我知道了张三的性别，我可以根据性别推荐商品

Action: 根据性别推荐商品
Action Input: 男
Observation: ['Steam爆款', 'RTX-9090', 'iPhone 80']
Thought: 

LLM Output >>> 现在我知道了可以推荐给张三的礼物，我可以给他送一个             # 模型根据已有的历史记录，得出了最终答案
Final Answer: 我可以给张三送一个Steam爆款、RTX-9090或者iPhone 80作为礼物。

```

现在已经得到足够多的信息，模型认为可以输出最终的答案了，因此输出「Final Answer：我可以送...」

而当我们解析到模型生成的内容中包含「Final Answer」关键字时，便可以让 Agent 停止思考并输出答案了。

最终的示例结果是这样：

```
User  >>> 我想送点礼物给张三
Model >>> 根据张三的性别和兴趣，我可以推荐Steam爆款、RTX-3090或iPhone 14作为礼物。

```

完整源码
====

```
import os
import re
import json
from typing import List, Union

from langchain.agents import Tool, AgentExecutor, LLMSingleActionAgent, AgentOutputParser
from langchain.prompts import StringPromptTemplate
from langchain import OpenAI, SerpAPIWrapper, LLMChain
from langchain.schema import AgentAction, AgentFinish

os.environ["OPENAI_API_KEY"] = "你的OpenAI key"

def find_person(name: str):
    """
    模拟本地数据库查询。

    Args:
        name (str): 人物名称，由LLM提取。

    Returns:
        _type_: _description_
    """
    info = {
        '张三': '男',
        '小红': '女'
    }
    return info.get(name, f'未找到{name}的性别信息，我应该直接返回 Observation: 未知')


def recommend_item(gender: str):
    """
    根据人物性别推荐不同的商品。

    Args:
        gender (str): 人物的性别，由 LLM 提取。

    Returns:
        _type_: _description_
    """
    recommend = {
        '男': ['Steam爆款', 'RTX-9090', 'iPhone 80'],
        '女': ['夏季碎花裙', '轻盈帆布包', '琉璃唇釉'],
        '未知': ['AJ新品', '手冲咖啡']
    }
    return recommend.get(gender, f'未找到合适的推荐商品，我应该返回 Final Answer: 随便买些什么吧，只要消费就能快乐！')

tools = [
    Tool(
        name = "查询人物性别",
        func=find_person,
        description="通过人名查找该人物的性别时用的工具，输入应该是人物的名字"
    ),
    Tool(
        name = "根据性别推荐商品",
        func=recommend_item,
        description="当知道了一个人性别后想进一步获得他可能感兴趣的商品时用的工具，输入应该是人物的性别"
    )
]


template_zh = """按照给定的格式回答以下问题。你可以使用下面这些工具：

{tools}

回答时需要遵循以下用---括起来的格式：

---
Question: 我需要回答的问题
Thought: 回答这个上述我需要做些什么
Action: ”{tool_names}“ 中的其中一个工具名
Action Input: 选择工具所需要的输入
Observation: 选择工具返回的结果
...（这个思考/行动/行动输入/观察可以重复N次）
Thought: 我现在知道最终答案
Final Answer: 原始输入问题的最终答案
---

现在开始回答，记得在给出最终答案前多按照指定格式进行一步一步的推理。

Question: {input}
{agent_scratchpad}
"""


class CustomPromptTemplate(StringPromptTemplate):
    template: str           # 标准模板
    tools: List[Tool]       # 可使用工具集合
    
    def format(
            self, 
            **kwargs
        ) -> str:
        """
        按照定义的 template，将需要的值都填写进去。

        Returns:
            str: 填充好后的 template。
        """
        intermediate_steps = kwargs.pop("intermediate_steps")       # 取出中间步骤并进行执行
        thoughts = ""
        for action, observation in intermediate_steps:
            thoughts += action.log
            thoughts += f"\nObservation: {observation}\nThought: "
        kwargs["agent_scratchpad"] = thoughts                       # 记录下当前想法
        kwargs["tools"] = "\n".join([f"{tool.name}: {tool.description}" for tool in self.tools])    # 枚举所有可使用的工具名+工具描述
        kwargs["tool_names"] = ", ".join([tool.name for tool in self.tools])                        # 枚举所有的工具名称
        cur_prompt = self.template.format(**kwargs)
        print(cur_prompt)
        return cur_prompt


prompt = CustomPromptTemplate(
    template=template_zh,
    tools=tools,
    input_variables=["input", "intermediate_steps"]
)


class CustomOutputParser(AgentOutputParser):
    
    def parse(
            self, 
            llm_output: str
        ) -> Union[AgentAction, AgentFinish]:
        """
        解析 llm 的输出，根据输出文本找到需要执行的决策。

        Args:
            llm_output (str): _description_

        Raises:
            ValueError: _description_

        Returns:
            Union[AgentAction, AgentFinish]: _description_
        """
        if "Final Answer:" in llm_output:       # 如果句子中包含 Final Answer 则代表已经完成
            return AgentFinish(
                return_values={"output": llm_output.split("Final Answer:")[-1].strip()},
                log=llm_output,
            )
        
        regex = r"Action\s*\d*\s*:(.*?)\nAction\s*\d*\s*Input\s*\d*\s*:[\s]*(.*)"   # 解析 action_input 和 action
        match = re.search(regex, llm_output, re.DOTALL)
        if not match:
            raise ValueError(f"Could not parse LLM output: `{llm_output}`")
        action = match.group(1).strip()
        action_input = match.group(2)
        
        return AgentAction(tool=action, tool_input=action_input.strip(" ").strip('"'), log=llm_output)
    
output_parser = CustomOutputParser()

llm = OpenAI(temperature=0)
llm_chain = LLMChain(
    llm=llm, 
    prompt=prompt
)

tool_names = [tool.name for tool in tools]
agent = LLMSingleActionAgent(
    llm_chain=llm_chain, 
    output_parser=output_parser,
    stop=["\nObservation:"], 
    allowed_tools=tool_names
)

agent_executor = AgentExecutor.from_agent_and_tools(
    agent=agent, 
    tools=tools, 
    verbose=True
)

res = agent_executor.run(
    "我想送点礼物给张三"
)
print(res)

```

总结
==

当然上面只是一个简单的用例，使用本地化定义的工具，大家可以尝试更多的工具。Tools 的使用以及 Agent 应该是未来大模型应用的落地点，希望大家紧跟时代的齿轮。