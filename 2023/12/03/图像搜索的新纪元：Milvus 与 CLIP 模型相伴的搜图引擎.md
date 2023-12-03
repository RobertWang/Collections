> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/yyblEV56q7RSZoNdC6fMLA)

*   1 背景介绍
    
*   2 CLIP 模型
    

*   2.1 关于 CLIP 模型
    
*   2.2 CLIP 模型的应用
    

*   3 Milvus
    

*   3.1 什么是 Milvus
    
*   3.2 Milvus 核心概念
    
*   3.3 相似性计算原理
    
*   3.4 Milvus 系统架构
    
*   3.5 选择 Milvus 的理由
    

*   4 转转风控的实践
    

*   4.1 Milvus 的部署方式
    
*   4.2 特征向量的生成
    
*   4.3 索引结构的选择
    
*   4.4 数据过滤的实现
    
*   4.5 一次搜索结果展示
    

1 背景介绍
------

  作为电商公司的风控部门，承担着维护平台内容安全的职责。因为政策的调整，或者一些突发情况，我们需要回溯线上历史的商品图片、用户头像信息等，确保平台的图片内容的合规性。  
  在以前我们会让算法同学离线将平台数据用相关的模型跑一遍，但是这会用到大量的计算资源，并且会花费几天甚至更长的时间。  
  我们是否有更便捷的办法对图片做搜索，比如像普通的数据库那样，通过内容甚至另一张图去搜索图片呢？可否将文本、图片等信息转换成另一种可以对比，可以计算的形式呢？  
  解决方案：可以通过深度模型提取出图像的特征向量，建立向量库，然后用目标文本或图片的特征向量进行搜索匹配，得出最接近的结果。CLIP 模型提供了生成文本和图片特征向量的能力，Milvus 向量数据库提供了对海量向量的存储、管理和检索能力。![](https://mmbiz.qpic.cn/mmbiz_png/dHUzltsJpQvL4btmZIzKSC21LZu5eEgX38s7IWyZ1EXd9ib1mxv3oNWsTlUia6MJMpvlaug3voHYGOsy9O32V9mg/640?wx_fmt=png&from=appmsg)

2 CLIP 模型
---------

### 2.1 关于 CLIP 模型

  CLIP（Contrastive Language-Image Pretraining）模型是一种由 OpenAI 开发的多模态预训练模型，结合了图像和文本的理解能力。CLIP 的目标是让模型能够理解图片和文本之间的关联关系，从而能够在语言和视觉任务上表现出色。CLIP 模型的主要作用是在图像和文本领域实现多模态的交叉理解能力，拓展了计算机视觉和自然语言处理的边界，为各种任务提供了更全面和准确的解决方案。

### 2.2 CLIP 模型的应用

  转转风控主要使用 CLIP 对图像根据文本提示词进行分类风险识别，同时因为 CLIP 模型输出为特征向量，所以同时使用 Milvus 向量数据库保存这些图片的特征向量。![](https://mmbiz.qpic.cn/mmbiz_png/dHUzltsJpQvL4btmZIzKSC21LZu5eEgXvWyrhIBLiaTkic9kyeykSZvdcLHBQAnPA3hMBWFV2XC1iaiaAc1meMYicbg/640?wx_fmt=png&from=appmsg)

3 Milvus
--------

### 3.1 什么是 Milvus

  Milvus 是一款云原生向量数据库，它具备高可用、高性能、易拓展的特点，用于海量向量数据的实时召回。Milvus 基于 FAISS、Annoy、HNSW 等向量搜索库构建，核心是解决稠密向量相似度检索的问题。

### 3.2 Milvus 核心概念

**非结构化数据**：非结构化数据指的是数据结构不规则，没有统一的预定义数据模型，不方便用数据库二维逻辑表来表现的数据，包括文本、音频、视频等。非结构化数据可以使用深度学习模型或者机器学习模型转化为向量后进行处理。  
**特征向量**：向量又称为 embedding vector，是指由 embedding 技术从离散变量转变而来的连续向量。在数学表示上，向量是一个由浮点数或者二值型数据组成的 n 维数组。  
**向量相似度检索**：相似度检索是指将目标对象与数据库中数据进行比对，并召回最相似的结果。同理，向量相似度检索返回的是最相似的向量数据。近似最近邻搜索（ANN）算法能够计算向量之间的距离，从而提升向量相似度检索的速度。如果两条向量十分相似，这就意味着他们所代表的源数据也十分相似。  
**Collection - 集合**：包含一组 Entity，可以等价于关系型数据库系统中的表。  
**Entity - 实体**：包含一组 Field。Field 与实际对象相对应。对应关系型数据库中的行。  
**Field - 字段**：Entity 的组成部分。Field 可以是结构化数据，例如数字和字符串，也可以是向量。对应关系型数据库中的表字段。  
**Partition - 分区**：分区是集合（Collection）的一个分区。Milvus 支持将收集数据划分为物理存储上的多个部分。  
**索引**：索引基于原始向量数据构建，可以提高对 Collection 数据搜索的速度。支持倒排列表、k-d 树以及高维哈希等。这种索引结构可以在大规模向量数据集中高效地定位相似向量。

### 3.3 相似性计算原理

  常用的向量相似度度量方法包括余弦相似度、欧氏距离、汉明距离等。此处以**欧氏距离**为例：中学学过的二维空间的欧氏距离：![](https://mmbiz.qpic.cn/mmbiz_png/dHUzltsJpQvL4btmZIzKSC21LZu5eEgXU73lTnU9TekTchdSF0rvVGgVOl5qfOwd0jyX97Sa4PM5uILO7pgCVQ/640?wx_fmt=png&from=appmsg)三维空间的欧式距离：![](https://mmbiz.qpic.cn/mmbiz_png/dHUzltsJpQvL4btmZIzKSC21LZu5eEgXcmic0yBEtxpfico5F64ggXk4TEuqDLgbYFugw0icQrno7kicpK8wba5TJg/640?wx_fmt=png&from=appmsg)以此类推，多维空间的欧式距离：![](https://mmbiz.qpic.cn/mmbiz_jpg/dHUzltsJpQvL4btmZIzKSC21LZu5eEgXOwcGn9MqPdWlOIQsluBoxgQS2dL2W6PQrRkdSYS6usoryk8Qx48U5g/640?wx_fmt=jpeg&from=appmsg)

### 3.4 Milvus 系统架构

Milvus 2.0 是一款云原生向量数据库，采用存储与计算分离的架构设计，所有组件均为无状态组件，极大地增强了系统弹性和灵活性。  
![](https://mmbiz.qpic.cn/mmbiz_png/dHUzltsJpQvL4btmZIzKSC21LZu5eEgXaKufhibmjZcfYBuiagzJgqemWM22MaZRB9M7SKIX3IqYqWkQr05ICkug/640?wx_fmt=png&from=appmsg)  
整个系统分为四个层次：  
**接入层（Access Layer）**：系统的门面，由一组无状态 proxy 组成。对外提供用户连接的 endpoint，负责验证客户端请求并合并返回结果。  
**协调服务（Coordinator Service）**：系统的大脑，负责分配任务给执行节点。协调服务共有四种角色，分别为 root coord、data coord、query coord 和 index coord。  
**执行节点（Worker Node）**：系统的四肢，负责完成协调服务下发的指令和 proxy 发起的数据操作语言（DML）命令。执行节点分为三种角色，分别为 data node、query node 和 index node。  
**存储服务 （Storage）**：系统的骨骼，负责 Milvus 数据的持久化，分为元数据存储（meta store）、消息存储（log broker）和对象存储（object storage）三个部分。  
各个层次相互独立，独立扩展和容灾。

### 3.5 选择 Milvus 的理由

**高性能**：性能高超，可对海量数据集进行向量相似度检索。Milvus 不但集成了业界成熟的向量搜索技术如 Faiss 和 SPTAG，Milvus 也实现了高效的 NSG 图索引。同时，Milvus 团队针对 Faiss IVF 索引进行了深度优化，实现了 CPU 与多 GPU 的融合计算，大幅提高了向量搜索性能。Milvus 可以在单机环境下完成 SIFT1b 十亿级向量搜索任务。  
**高可用、高可靠**：Milvus 支持在云上扩展，其容灾能力能够保证服务高可用。  
**混合查询**：Milvus 支持在向量相似度检索过程中进行标量字段过滤，实现混合查询。  
**开发者友好**：支持多语言、多工具的 Milvus 生态系统。Milvus 提供了向量数据管理服务，以及集成的应用开发 SDK（Java/Python/C++/RESTful API）。相比直接调用 Faiss 和 SPTAG 那样的程序库，Milvus 开发使用更便捷，数据管理更简单。

4 转转风控的实践
---------

### 4.1 Milvus 的部署方式

  Milvus 支持基于 Docker Compose 的单例部署模式，以及基于 k8s 的集群部署模式。目前我们使用的是单例部署模式，1 是对于我们的使用场景而言，单机性能目前没有问题；2 是单例部署模式更简单易上手。

### 4.2 特征向量的生成

  前文已经说过我们使用 CLIP 模型来进行特征向量的生成。原因主要有 2 个，1 是节省计算资源，CLIP 模型已经在线上应用，再用其它模型进行特征向量生成需要再进行一次计算，浪费计算资源；2 是 CLIP 模型本身提供了非常好的文本和图片交叉理解能力，为文搜图提供了基础。

### 4.3 索引结构的选择

  索引的选择对于向量召回的性能至关重要，Milvus 支持了 **Annoy，FAISS，HNSW，DiskANN** 等多种不同的索引，用户可以根据对延迟，内存使用和召回率的需求进行选择。对于我们的使用场景，我们对响应时长要求不高，主要为离线或后台使用，但是要求 100% 召回，不能漏召回，所以使用近似查找的压缩索引都不符合要求，只能使用 FAISS 的 Flat 索引。

### 4.4 数据过滤的实现

  基于产品的使用的数据过滤需求，以及需要对历史数据进行定期清理的目标，目前我们是根据时间以及数据源类型创建的分区。  为什么没有使用 Milvus 的标量过滤特性去做过滤呢？  
  主要是基于性能考量，Milvus 使用的是前过滤，即先做标量过滤生成 Bitset，在向量检索的过程中基于 Bitset 去除掉不满足条件的 Entity。在一些场景下标量过滤不仅不会加速查询反而会导致性能变差。而且目前我们的过滤场景很确定，用指定分区来实现数据过滤的方式可以获得更好的性能。

### 4.5 一次搜索结果展示

![](https://mmbiz.qpic.cn/mmbiz_png/dHUzltsJpQvL4btmZIzKSC21LZu5eEgXeoyHpuJZ9206pm9QicZe2tGz9ibahR06kUecycNRAJVYYbAwIfgZ38xw/640?wx_fmt=png&from=appmsg)  图片为以**萨摩耶**为搜索词，搜索 2023-11-19 至 2023-11-23 的商品数据得出的相似度最高的 top20 的结果。

* * *

> 关于作者

许作红，转转风控后端研发工程师，主要负责风控模型工程的开发维护。热爱运动，拥抱生活。