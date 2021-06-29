> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NjY5NzI0NA==&mid=2247500500&idx=2&sn=b1b016bf56e2c04e42a1695d0cb7717a&chksm=ea88ada7ddff24b172808e58e209373cce3dde655c9bfc1c4f1b6a48a08d95bff2823a58dd78&mpshare=1&scene=1&srcid=0629u6vPD6YscO8tbu9pdge8&sharer_sharetime=1624939652867&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> 文章来自：数仓宝贝库

  

**导读：**SQFlow 利用 SQL 语言构建机器学习和深度学习，致力于 “Make AI as simple as SQL”，愿景是推进人工智能大众化、普及化，也就是只要懂商业逻辑就能用上人工智能，让懂业务的人能自由地使用人工智能。本文带你快速了解开源的工程化的自助式数据科学平台 SQLFlow。

SQLFlow 利用 SQL 语言构建机器学习和深度学习，致力于 “Make AI as simple as SQL”，愿景是推进人工智能大众化、普及化，也就是只要懂商业逻辑就能用上人工智能，让懂业务的人能自由地使用人工智能。**SQLFlow 具备三大核心要素，即数据描述商业逻辑、AI 赋能深度数据分析和易于使用。**

**0****1**

**什么是 SQLFlow**

  

**0****2**

**SQLFlow 的定位和目标**

  

作为连接数据引擎和 AI 引擎的桥梁，SQLFlow 目前支持的数据引擎包括 MySQL、Hive 和 MaxCompute，支持的 AI 引擎不仅包括业界流行的 TensorFlow，还包括 XGBoost、Scikit-Learn 等传统机器学习框架，如下表所示。

表 SQLFlow 支持的数据引擎和 AI 引擎简介

![](https://mmbiz.qpic.cn/mmbiz_jpg/oeqMOybW2r0VXUKWttOoatjKTuptNKmgR9oCr3pojwuvx8UmYJm3D35w9hPIFoq1Xa75zj1L83kzg7WMAsuasg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/oeqMOybW2r0VXUKWttOoatjKTuptNKmghQwQ3u9pfj9xCBpHhA8ZezM9NBLwuNiaSKVgdsvWJe9ecnYVww1DYSg/640?wx_fmt=jpeg)

**0****3**

**SQLFlow 的工作原理**

  

接下来我们使用 Docker 镜像中的 Iris 案例来说明 SQLFlow 的工作原理。

**Iris 数据集**也称为鸢尾花数据集，是机器学习领域常用的分类实验数据集。该数据集包含 150 个数据样本，分为 3 类，每类包含 50 个样例，每条样例包含花萼长度、花萼宽度、花瓣长度和花瓣宽度。数据集通过这 4 个属性判断鸢尾花样例属于山鸢尾（setosa）、杂色鸢尾（versicolour）、维吉尼亚鸢尾（virginica）中的哪一类。

我们预先将数据存储至 iris.train 表中，前 4 列表示训练样例的特征，最后一列代表训练样本的标签。

分类模型以 DNN 分类器为例。DNN 分类器默认具有双隐藏层，每层的隐藏节点数均为 10，分类数为 3，默认优化器和学习率分别为 Adagrad 和 0.1，损失函数则默认配置为 tf.keras.losses.sparse_categorical_crossentropy。

从 iris.train 表中获取数据并训练对应 DNN 分类器模型的训练语句如代码清单 1 所示。

```
代码清单1　基于iris数据集训练DNN分类器
SELECT * FROM iris.train
TO TRAIN DNNClassifer
WITH hidden_units = [10, 10], n_classes = 3, EPOCHS = 10
COLUMN sepal_length, sepal_width, petal_length, petal_width
LABEL class
INTO sqlflow_models.my_dnn_model;

```

SQLFlow 解析接收到的 SQL 命令，其中 SELECT 语句传递给对应数据引擎获取数据，而 TRAIN 和 WITH 语句则分别指定了使用的模型种类、模型结构和训练所需的超参数，COLUMN 和 LABEL 部分分别用于训练的各特征列名称和标签列名称。

SQLFlow 将 TRAIN 和 WITH 语句中的内容解析为对应的 Python 程序，整体流程如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_jpg/oeqMOybW2r0VXUKWttOoatjKTuptNKmgPnw4nbWGfsqjskTNn8KXPibDBO1ib6YHhBPoiaMMEVpPqqrnryl2oibpQQ/640?wx_fmt=jpeg)

图 SQLFlow 工作原理  

本文摘编于《数据科学工程实践》，经出版方授权发布。