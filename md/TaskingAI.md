> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2NzkxNDY0Ng==&mid=2247487060&idx=1&sn=4efba423a2016963e4d296f97b100e2c&chksm=fc94b7adcbe33ebb475a7eb9ee58340a1d59222fd01a75c5f213a0ca05d8c563bb7add68c9ce&mpshare=1&scene=1&srcid=0201HoHOidfmdxnDMndzx9hM&sharer_shareinfo=edc15e6ad600185113bbd8cbbf40b118&sharer_shareinfo_first=edc15e6ad600185113bbd8cbbf40b118#rd)

前言

在人工智能盛起的当下，AI 正以非常迅猛的速度重塑着很多行业。可以预见的是 2024 将是 AI 原生应用开发元年，将会涌现出数不清的 AI 原生应用来重塑我们的工作和生活的方方面面。而在 AI 原生应用里面将会以 AI Agent 即 AI 智能体为主要代表，将会有很多个像 [crewAI—用于编排角色扮演的 AI agent（超级智能体）](http://mp.weixin.qq.com/s?__biz=MzU2NzkxNDY0Ng==&mid=2247486972&idx=1&sn=745ea174b66bd31ddfe0124bf902c133&chksm=fc94b405cbe33d13bad87f75593bda15707d5f22e584867a1585208228fdcaabda1a6d310bfb&scene=21#wechat_redirect)一样的 Agent 出现在我们的面前。今天要重点介绍的便是一款 AI 原生应用开发工具—TaskingAI。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/8wiaQU8PwroKs0jBV1bhZ2ia1yArNRjiaUS3YR7Tpia4lbEEeRsIpt2MDnHCGSP4s7qZASScCQ3fCu7N0QvlCSRcFQ/640?wx_fmt=png&from=appmsg)

TaskingAI
=========

TaskingAI 的协调设计确保了 AI 应用开发中的高效、智能和用户友好体验。

主要特点：
-----

1. **全能 LLM 平台**：使用统一的 API 访问数百种 AI 模型。2. **直****观的 UI 控制台**：简化项目管理并允许在控制台内进行工作流测试。3.**BaaS 灵感****的工作流程**：将 AI 逻辑（服务器端）与产品开发（客户端）分开，通过 RESTful API 和客户端 SDK 提供从控制台原型设计到可扩展解决方案的清晰路径。4. **可定制集成**：使用可定制工具和先进的检索增强生成（RAG）系统增强 LLM 功能。5. **异步效率**：利用 Python FastAPI 的异步特性进行高性能、并发计算，提高应用程序的响应性和可扩展性。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/8wiaQU8PwroKs0jBV1bhZ2ia1yArNRjiaUSia6ew3gRRcibjo2IBPibtn3icLibkcdPOCbYXllATK6b9FQH0On7Ufmw6nw/640?wx_fmt=png&from=appmsg)

使用 TaskingAI 能建造什么？
-------------------

• **交互式应用程序演示**：使用 TaskingAI 的 UI 控制台快速创建并部署引人入胜的应用程序演示。这是展示 AI 本地应用潜力的理想环境，具有实时互动和用户参与。• **团队协作的 AI 代理**：开发利用集体知识和工具的 AI 代理，增强团队合作和效率。TaskingAI 促进创建支持协作和组织内部支持的共享 AI 资源。• **面向商业的多租户 AI 本地应用程序**：使用 TaskingAI 构建适用于生产的强大多租户 AI 本地应用程序。它非常适合处理各种客户需求，同时保持个性化定制、安全性和可扩展性。

为什么选择 TaskingAI？
----------------

### 现有产品的问题

OpenAI 的助手 API 虽然在类似 GPT 的功能上很强大，但由于其设计将关键功能（如工具和文档检索）绑定在单个助手上，这种结构可能限制了多租户应用程序的灵活性，其中共享数据至关重要。

### TaskingAI 如何解决问题

TaskingAI 通过解耦关键模块，提供更广泛的模型支持和一个开源框架来克服这些障碍。其适应性使其成为需要更多样化、能够共享数据的 AI 解决方案的开发人员的更好选择，尤其是对于复杂、可定制的项目。

### 对比

下面是主流代理开发框架与 TaskingAI 之间的比较表：

<table width="335"><thead data-style="color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 15px; background: rgba(0, 0, 0, 0.05);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; font-size: 12px; border-color: rgb(223, 223, 223); text-align: justify;">特征</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; font-size: 12px; border-color: rgb(223, 223, 223); text-align: justify;">LangChain</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; font-size: 12px; border-color: rgb(223, 223, 223); text-align: justify;">TaskingAI</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">LLM 提供商</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">多个提供商</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">多个提供商</td></tr><tr><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">检索系统</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">需要第三方</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">解耦；灵活</td></tr><tr><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">工具集成</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">需要第三方</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">解耦；灵活</td></tr><tr><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">代理记忆</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">可配置</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">可定制</td></tr><tr><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">开发方法</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">基于 Python 的 SDK</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">RESTful API 和 SDK</td></tr><tr><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">异步支持</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">选择模型支持</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">全面</td></tr><tr><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">多租户支持</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">复杂设置</td><td data-style="padding: 0.25em 0.5em; color: rgb(63, 63, 63); line-height: 1.75; font-family: -apple-system-font, BlinkMacSystemFont, &quot;Helvetica Neue&quot;, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei UI&quot;, &quot;Microsoft YaHei&quot;, Arial, sans-serif; font-size: 11.2px; border-color: rgb(223, 223, 223); text-align: justify;">简化设置</td></tr></tbody></table>

### 架构

TaskingAI 的架构以模块化和灵活性为核心设计，使其能够与广泛的 LLMs 兼容。这种适应性使它能够轻松支持各种应用程序，从简单的演示到复杂的多租户 AI 系统。TaskingAI 建立在开源原则的基础上，集成了众多开源工具，确保平台不仅多功能，而且可定制。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/8wiaQU8PwroKs0jBV1bhZ2ia1yArNRjiaUS3Ql4OpAiaDmZUzr7ibNtmLkiaiaHSy13k1fCnmsziaJwXd6EcTF2Ac2IPbw/640?wx_fmt=png&from=appmsg)

•**Nginx**：作为前端 Web 服务器，有效地将流量路由到架构内的指定服务。• **前端（TypeScript + React）**：使用 TypeScript 和 React 构建的交互式和响应式用户界面，允许用户顺畅地与后端 API 交互。• **后端（Python + FastAPI）**：后端采用 Python 和 FastAPI 构建，其异步设计带来高性能。它管理业务逻辑、数据处理，并作为前端和 AI 推理服务之间的桥梁。Python 的广泛使用邀请更广泛的贡献，促进持续改进和创新的协作环境。•**TaskingAI - 推理**：专用于 AI 模型推理，这个组件熟练处理响应生成和自然语言输入处理等任务。它是 TaskingAI 开源套件中的另一个亮点项目。•**TaskingAI 核心服务**：包括模型、助手、检索和工具等各种服务，每个服务都对平台的运行至关重要。•**PostgreSQL + PGVector**：作为主要数据库，PGVector 通过增强嵌入比较的向量操作，对 AI 功能至关重要。•**Redis**：提供高性能数据缓存，对加快响应时间和提高数据检索效率至关重要。

通过 Docker 快速开始
==============

使用 Docker 是启动自托管的 TaskingAI 社区版的一种简单方法。

先决条件
----

• 在您的机器上安装了 Docker 和 Docker Compose。• 安装了 Git 用于克隆仓库。•Python 环境（Python 3.8 以上）用于运行客户端 SDK。

安装
--

首先，从 GitHub 克隆 TaskingAI（社区版）仓库。

```
git clone https://github.com/taskingai/taskingai.git
cd taskingai

```

在克隆的仓库内，进入 docker 目录并使用 Docker Compose 启动服务。

```
cd docker
docker-compose -p taskingai up -d

```

一旦服务启动，通过浏览器使用 URL `[http://localhost:8080]`访问 TaskingAI 控制台。默认的用户名和密码是`admin`和 `TaskingAI321`。

TaskingAI UI 控制台
----------------

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/8wiaQU8PwroKs0jBV1bhZ2ia1yArNRjiaUSpJHZ8w2gV9vfduWEeZmERUkCdprysf8g4d7xPjoojqGqibro75LuibyA/640?wx_fmt=jpeg&from=appmsg)

点击上面的图片查看 TaskingAI 控制台演示视频

TaskingAI 客户端 SDK
-----------------

一旦控制台启动，您可以使用 TaskingAI 客户端 SDK 以编程方式与 TaskingAI 服务器进行交互。

确保安装了 Python 3.8 或以上版本，并设置了虚拟环境（可选但推荐）。使用 pip 安装 TaskingAI Python 客户端 SDK。

```
pip install taskingai

```

这里有一个客户端代码示例：

```
import taskingai
from taskingai.assistant.memory import AssistantNaiveMemory
taskingai.init(api_key='YOUR_API_KEY', host='http://localhost:8080')
# 创建一个新的助手
assistant = taskingai.assistant.create_assistant(
    model_id="YOUR_MODEL_ID",
    memory=AssistantNaiveMemory(),
)
# 创建一个新的聊天
chat = taskingai.assistant.create_chat(
    assistant_id=assistant.assistant_id,
)
# 发送用户消息
taskingai.assistant.create_message(
    assistant_id=assistant.assistant_id,
    chat_id=chat.chat_id,
    text="Hello!",
)
# 生成助手响应
assistant_message = taskingai.assistant.generate_message(
    assistant_id=assistant.assistant_id,
    chat_id=chat.chat_id,
)
print(assistant_message)

```

请注意，`YOUR_API_KEY` 和`YOUR_MODEL_ID`应该替换为您在控制台中创建的实际 API 密钥和聊天完成模型 ID。

您可以在文档 [1] 中了解更多信息。

资源
==

• 文档 [2]•API 参考 [3]• 联系我们 [4]

其他
==

本文由山行翻译整理自：https://github.com/TaskingAI/TaskingAI，核心目的是为大家科普更多 AI 相关的知识，如果对您有帮助，请帮忙点赞、收藏、关注，谢谢～  

### References

`[1]` documentation: _https://docs.tasking.ai/docs/guide/getting_started/self_hosting/overview_  
`[2]` Documentation: _https://docs.tasking.ai/_  
`[3]` API Reference: _https://docs.tasking.ai/api_  
`[4]` Contact Us: _https://www.tasking.ai/contact-us_