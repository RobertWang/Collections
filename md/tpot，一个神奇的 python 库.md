> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rtW9aoLCFrYwTJ4apyDntw)

大家好，我是小寒。

今天给大家分享一个神奇的 python 库，**tpot**。

**https://github.com/EpistasisLab/tpot**

TPOT 是一个 Python **自动化机器学习工具**，可使用**遗传编程**优化机器学习管道。

TPOT 将通过智能地**探索数千种可能的管道**来找到最适合你的数据的管道，从而自动化机器学习中**最繁琐的部分**。TPOT 可以自动化机器学习流程中的如下部分。

*   特征选择
    
*   特征预处理
    
*   特征构建
    
*   模型选择
    
*   超参数优化
    

![](https://mmbiz.qpic.cn/mmbiz_png/ibPqnScajrbFia1q1libohUS3Z9Bcd1sYRQapEpmicMoYLklUn8p2fSydFAbWWeNDfvVaqkfexrSdoK6ibZ16Q6ecGA/640?wx_fmt=png)

### TPOT 采用什么算法进行采样？

默认情况下，TPOT 搜索广泛的**预处理器、特征构造器和选择器、模型和超参数**，以最大限度地减少模型预测误差。

对于分类问题，**它会搜索以下内容**：

*   线性模型
    
*   朴素贝叶斯模型
    
*   树模型（如决策树分类器）
    
*   集成模型（如随机森林分类器）
    
*   SVM 模型（如 LinearSVC）
    
*   XGBoost 模型
    

对于回归问题，**TPOT 的搜索空间包括**：

*   线性模型（ElasticNetCV、SGDRegressor）
    
*   集成模型（如 GradientBoostingRegressor）
    
*   SVM 模型（如线性 SVR）
    
*   XGBoost 模型
    

### 初体验

#### 库的安装

我们可以直接使用 pip 进行安装。

```
pip install tpot

```

#### 加载数据集

对于此示例，我们将使用广为人知的 **鸢尾花数据集**。

该数据集根据四个属性将鸢尾花分为三个不同的物种，四个属性分别为**萼片长度、萼片宽度、花瓣长度和花瓣宽度。**

```
from sklearn.datasets import load_iris
from tpot import TPOTClassifier
from sklearn.model_selection import train_test_split

# Load the iris data
iris = load_iris()
X = iris.data
y = iris.target
X_train, X_test, y_train, y_test = train_test_split(X, y, train_size=0.75, test_size=0.25, random_state=5)


```

#### 实例化 TPOT 分类器

现在，让我们实例化 TPOT 分类器。

我们将 generations 参数设置为 5，population_size 设置为 50。

请注意，这些只是用于演示目的的基本配置，最佳设置需要更多探索。

```
tpot = TPOTClassifier(generations=5, population_size=50, verbosity=2, random_state=5)

```

#### 模型训练

接下来，我们将使用训练数据训练我们的模型。

```
tpot.fit(X_train, y_train)

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibPqnScajrbFia1q1libohUS3Z9Bcd1sYRQRO6AoJyRXpGE4pianvklia2SuQtd4r7JYsXCVZ5miasP7zb1DaVjo8SRA/640?wx_fmt=png)

在训练过程中，TPOT 将输出迄今为止发现的**最佳管道的性能，以及它已经处理了多少轮**。

让我们来分析一下这些消息的含义。

**Generations**： generations 本质上是 TPOT 优化管道所经历的迭代次数。在本例中，你将其设置为 5。

**Current best internal CV score**：这是 TPOT 在数据交叉验证上取得的最佳分数。**这里提到的分数是模型质量的衡量标准**，该分数可能是准确性，但根据你的设置，它可能是另一个指标。

**Best pipeline**：这是 TPOT 在其进化过程中发现的最好的机器学习管道。在这个例子中，它是一个 MLPClassifier 估计器。

#### 评估模型

训练完成后，我们可以在测试数据上评估最佳管道的性能。

```
print(tpot.score(X_test, y_test))

```

这将输出最佳管道的准确性。

#### 导出最佳模型

最后，如果我们对性能感到满意，**我们可以导出最佳管道的相应 Python 代码。**

```
tpot.export( 'tpot_iris_pipeline.py' )

```

这将创建一个名为 “tpot_iris_pipeline.py” 的新 Python 文件，其中包含管道的 Python 代码。

这是文件 **tpot_iris_pipeline.py** 的内容。

```
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.neural_network import MLPClassifier

# NOTE: Make sure that the outcome column is labeled 'target' in the data file
tpot_data = pd.read_csv('PATH/TO/DATA/FILE', sep='COLUMN_SEPARATOR', dtype=np.float64)
features = tpot_data.drop('target', axis=1)
training_features, testing_features, training_target, testing_target = \
            train_test_split(features, tpot_data['target'], random_state=5)

# Average CV score on the training set was: 0.990909090909091
exported_pipeline = MLPClassifier(alpha=0.001, learning_rate_init=0.001)
# Fix random state in exported estimator
if hasattr(exported_pipeline, 'random_state'):
    setattr(exported_pipeline, 'random_state', 5)

exported_pipeline.fit(training_features, training_target)
results = exported_pipeline.predict(testing_features)


```

TPOT 是一个强大的工具，**可以显着简化构建机器学习管道的过程。**

需要注意的是，虽然 TPOT 可以实现大部分流程的自动化，但它并不能替代理解基本概念。

AutoML 工具最好用作**增强数据科学家工作流程的辅助工****具**，而不是取代对它们。