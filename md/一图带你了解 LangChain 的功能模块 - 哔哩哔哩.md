> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bilibili.com](https://www.bilibili.com/read/cv25333824)

> Model I/O：管理大语言模型（Models），及其输入（Prompts）和格式化输出（Output Parsers）Data Connection：管理主要用于建设私域知识（库）的向量数据存储（V......

*   **Model I/O**：管理大语言模型（Models），及其输入（Prompts）和格式化输出（Output Parsers）
    
*   **Data Connection**：管理主要用于建设私域知识（库）的向量数据存储（Vector Stores）、内容数据获取（Document Loaders）和转化（Transformers），以及向量数据查询（Retrievers）
    
*   **Memory**：用于存储和获取 对话历史记录 的功能模块
    
*   **Chains**：用于串联 Memory ↔️ Model I/O ↔️ Data Connection，以实现 串行化 的连续对话、推测流程
    
*   **Agents**：基于 Chains 进一步串联工具（Tools），从而将大语言模型的能力和本地、云服务能力结合
    
*   **Callbacks**：提供了一个回调系统，可连接到 LLM 申请的各个阶段，便于进行日志记录、追踪等数据导流
    

![](http://i0.hdslb.com/bfs/article/f96fcda6b35886d22c413e7c5415e68af1b80109.png@1256w_1110h_!web-article-pic.avif)LangChain 功能模块概览![](http://i0.hdslb.com/bfs/article/card/7f37f7c211baa93756265b8ffa8126aba728abfc.png)