> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODkzMzMwMQ==&mid=2650433749&idx=3&sn=388123913eb29902c5333eed3aeb1c87&chksm=becdee8f89ba67997e203261f3c2ed8a02da90fa0ba0b3dae663b61492a7b1458146b2b368fe&mpshare=1&scene=1&srcid=1009mDexn3CD4R1aFY3J0qx1&sharer_sharetime=1665330159293&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

关于中台，有太多讨论的角度。比如，最近读过的一本书，钟华的《企业 IT 架构转型之道》。这篇文章仅从个人作为用户的角度做一个小的讨论。

首先，为什么需要算法中台？

（1）用一种机制设计相对清晰地划分了算法和工程的边界。这种划分并不是绝对意义上的划分，而是用专业的人做专业的事情。  

（2）算法研发效能提升。算法研发的流程整体上看是串行的，形成闭环之后就进入一个正常的迭代流程。各个依赖组件，工具的打通，能够较大限度的提效。  

总而言之，算法中台通过对生产工具的组合，容纳多种生产关系，从而带来生产力的较大提升。

什么时候才需要一个算法中台？

算法需求较多的时候。业务侧的推进带来产品形态的繁荣，进而带来较多的算法需求。这个时候，对于研发效能的提升提出了新的要求，就需要组合生成工具，变革生产关系，从而带来生产力的提升。

生产工具的组合是单点效能提升的方式，一个算法同学不需要同时了解多种工具组件。生产关系是多点效能提升的关键，不仅协调算法团队内部的关系，同时处理算法和工程，以及产品或者运营同学的协作关系。

所以，在算法需求不多且团队较小的时候，刀耕火种就可以。但是当团队变大，需求变多的时候，也许就是考虑算法中台能力建设的时候。

理想中的算法中台是什么样子的？一种架构方式如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/GcdHgGa9niciaShKBlia9icWegibVvicRCc6EjHTHicpIb4eZw0DtsU8lHJ4lpNj9OicV8wqHSRIHbIeLkteicR0iaXEhFzQ/640?wx_fmt=png)

采用自底向上的角度来看，数据管理平台内联 ETL，数据版本管理等各种基础功能，外联标注平台和知识图谱能力，整体是数据侧工具的整合打通。pipeline 平台连接数据平台和代码平台，沉淀基础算子，通用组件，算法 pipeline 等。  

对于单个算法服务，需要方便高效的部署方式，支持服务能力的水平扩展。此外，和传统工程类似，需要性能测试。除了性能测试，也需要对应的回归测试能力，服务版本管理以及指标评估能力，监控能力等。  

整体上，对于多框架的支持也是必须的。最后，通过统一的 serving 层提供对外的服务能力。

综上可知，算法中台的建设是工程主导，面向算法同学，结合算法研发特色的系统性工作。相比于两年前，建设一个算法中台的各个组件的开源工作已经丰富了很多，基本可以利用现有组件，搭建一个基本可用的算法中台。

相关参考：  

(1)grafana 的 mac 版单机环境搭建

https://www.cnblogs.com/yjmyzz/p/how-to-install-prometheus-and-grafana.html

(2) 性能测试工具 Locust 简介及安装使用

https://testerhome.com/topics/23649

(3) 数据版本管理 dvc

https://github.com/iterative/dvc

(4)BentoML

https://github.com/bentoml/BentoML，笔者个人非常喜欢的工作

(5)MLOps.toys

https://mlops.toys/，一个 MLOps 相关项目的 list，包括：Data Versioning, Training Orchestration, Feature Store, Experiment Tracking, Model Serving, Model Monitoring, Explainability.

(6)service-streamer

https://github.com/ShannonAI/service-streamer/blob/master/README_zh.md

(7)https://github.com/ahkarami/Deep-Learning-in-Production

(8)https://github.com/samueldobbie/markup，基于主动学习的 web 版标注工具，笔者对标注工具也是极其的喜欢，比较重要的 feature 包括但不限于：多人权限，质检，预标注，主动学习。