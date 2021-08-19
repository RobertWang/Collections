> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2MjE3OTA1MA==&mid=2247492926&idx=1&sn=ac098dd7017151c9547892bf9632874b&chksm=ea4db4bbdd3a3dad4a29be98519ae11cffeb59215abb6d6a274b6432aa2f7690b389cdbd2273&mpshare=1&scene=1&srcid=0819mK9qJiviUpU4rrrSGUl5&sharer_sharetime=1629345233456&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

大家都知道 Python，但是应该很少有人听过 dython，dython 是 python 中的一款数据建模库。尽管已经有了 scikit-learn、statsmodels、seaborn 等非常优秀的数据建模库，但实际数据分析过程中常用到的一些功能场景仍然需要编写数十行以上的代码才能实现。  

而今天要给大家推荐的`dython`就是一款集成了诸多实用功能的数据建模工具库，帮助我们更加高效地完成数据分析过程中的诸多任务：

![](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmdOwmdUhyCOibrxvJTCGp7r5FLgQbNhqjSdCeEeYOGNQxTLspqQNsLialpJx5iaTuZKyMjbXWm2kOQIQ/640?wx_fmt=png)

通过下面两种方式均可完成对`dython`的安装：

```
pip install dython
```

或：

```
conda install -c conda-forge dython
```

`dython`中目前根据功能分类划分为以下几个子模块：

*   **「data_utils」**
    

`data_utils`子模块集成了一些基础性的数据探索性分析相关的 API，如`identify_columns_with_na()`可用于快速检查数据集中的缺失值情况：

```
>> df = pd.DataFrame({'col1': ['a', np.nan, 'a', 'a'], 'col2': [3, np.nan, 2, np.nan], 'col3': [1., 2., 3., 4.]})
>> identify_columns_with_na(df)
  column  na_count
1   col2         2
0   col1         1
```

`identify_columns_by_type()`可快速选择数据集中具有指定数据类型的字段：

```
>> df = pd.DataFrame({'col1': ['a', 'b', 'c', 'a'], 'col2': [3, 4, 2, 1], 'col3': [1., 2., 3., 4.]})
>> identify_columns_by_type(df, include=['int64', 'float64'])
['col2', 'col3']
```

`one_hot_encode()`可快速对数组进行**「独热编码」**：

```
>> one_hot_encode([1,0,5])
[[0. 1. 0. 0. 0. 0.]
 [1. 0. 0. 0. 0. 0.]
 [0. 0. 0. 0. 0. 1.]]
```

`split_hist()`则可以快速绘制分组直方图，帮助用户快速探索数据集特征分布：

```
import pandas as pd
from sklearn import datasets
from dython.data_utils import split_hist

# Load data and convert to DataFrame
data = datasets.load_breast_cancer()
df = pd.DataFrame(data=data.data, columns=data.feature_names)
df['malignant'] = [not bool(x) for x in data.target]

# Plot histogram
split_hist(df, 'mean radius', split_by='malignant', bins=20, figsize=(15,7))
```

![](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmdOwmdUhyCOibrxvJTCGp7r5hLibAvNfkg1erpmDcKMEt9E1Opt0AAoI3oWMlkgKfOtHBVJ3DUkayJA/640?wx_fmt=png)

*   **「nominal」**
    

`nominal`子模块包含了一些进阶的特征相关性度量功能，例如其中的`associations()`可以自适应由连续型和类别型特征混合的数据集，并自动计算出相应的`Pearson`、`Cramer's V`、`Theil's U`、条件熵等多样化的系数；`cluster_correlations()`可以绘制出基于层次聚类的相关系数矩阵图等实用功能：

![](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmdOwmdUhyCOibrxvJTCGp7r5FuVib8MQ0yZw0r64ePLVBQIeOUT28101EetBkXMibuh2cViagsuUfnHjQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmdOwmdUhyCOibrxvJTCGp7r5HX5HU2TljHdwnVRZ9gzOtj9hJpibJr7V6VGeGRljbMUmEYQKSp70gOw/640?wx_fmt=png)

*   **「model_utils」**
    

`model_utils`子模块包含了诸多对机器学习模型进行性能评估的工具，如`ks_abc()`：

```
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from dython.model_utils import ks_abc

# Load and split data
data = datasets.load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(data.data, data.target, test_size=.5, random_state=0)

# Train model and predict
model = LogisticRegression(solver='liblinear')
model.fit(X_train, y_train)
y_pred = model.predict_proba(X_test)

# Perform KS test and compute area between curves
ks_abc(y_test, y_pred[:,1])
```

![](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmdOwmdUhyCOibrxvJTCGp7r5z20PDia1owvsUsruL4icz5cL4hK7xnYXnia6KRMja5lXz4l1JwWD4wFeA/640?wx_fmt=png)

`metric_graph()`：

```
import numpy as np
from sklearn import svm, datasets
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import label_binarize
from sklearn.multiclass import OneVsRestClassifier
from dython.model_utils import metric_graph

# Load data
iris = datasets.load_iris()
X = iris.data
y = label_binarize(iris.target, classes=[0, 1, 2])

# Add noisy features
random_state = np.random.RandomState(4)
n_samples, n_features = X.shape
X = np.c_[X, random_state.randn(n_samples, 200 * n_features)]

# Train a model
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=.5, random_state=0)
classifier = OneVsRestClassifier(svm.SVC(kernel='linear', probability=True, random_state=0))

# Predict
y_score = classifier.fit(X_train, y_train).predict_proba(X_test)

# Plot ROC graphs
metric_graph(y_test, y_score, 'pr', class_names=iris.target_names)
```

![](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmdOwmdUhyCOibrxvJTCGp7r5M4NiaIWjOFCdc87Em0gO5nNOVnCz5YrFHKlNMRgv7VWTdHeInDib2Liaw/640?wx_fmt=png)

```
import numpy as np
from sklearn import svm, datasets
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import label_binarize
from sklearn.multiclass import OneVsRestClassifier
from dython.model_utils import metric_graph

# Load data
iris = datasets.load_iris()
X = iris.data
y = label_binarize(iris.target, classes=[0, 1, 2])

# Add noisy features
random_state = np.random.RandomState(4)
n_samples, n_features = X.shape
X = np.c_[X, random_state.randn(n_samples, 200 * n_features)]

# Train a model
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=.5, random_state=0)
classifier = OneVsRestClassifier(svm.SVC(kernel='linear', probability=True, random_state=0))

# Predict
y_score = classifier.fit(X_train, y_train).predict_proba(X_test)

# Plot ROC graphs
metric_graph(y_test, y_score, 'roc', class_names=iris.target_names)
```

![](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmdOwmdUhyCOibrxvJTCGp7r5Vy64I7GvviaqgZTn4uj2Ve1OiaDLI4zvC9FwB8h7OTKiafgfbrd5SSMjA/640?wx_fmt=png)

*   **「sampling」**
    

`sampling`子模块则包含了`boltzmann_sampling()`和`weighted_sampling()`两种数据采样方法，简化数据建模流程。

![](https://mmbiz.qpic.cn/mmbiz_png/g64sbb6FfmdOwmdUhyCOibrxvJTCGp7r5yQLuPp0ibSwfQSewRFF4XkezJstqxAO9Dg7V1ickEaJGOEodh5zqaTiag/640?wx_fmt=png)

`dython`作为一个处于快速开发迭代过程的`Python`库，陆续会有更多的实用功能引入，感兴趣的朋友们可以前往`https://github.com/shakedzy/dython`查看更多内容或对此项目保持关注。

* * *

以上就是本文的全部内容，欢迎在评论区与我进行讨论~