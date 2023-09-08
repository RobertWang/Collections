> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/X4e0IvC4XpcM99U2fvsUkg)

> Pyforest 是一个开源的 Python 库，可以自动导入代码中使用到的 Python 库安装 pip instal

Pyforest 是一个开源的 Python 库，可以自动导入代码中使用到的 Python 库

安装
--

```
pip install pyforest -i https://pypi.tuna.tsinghua.edu.cn/simple

```

pyforest 库的使用非常简单，我们只需要导入该包（无需导入其他包），然后就可以直接进行代码的编写。

```
# 仅需导入pyforest
import pyforest

# 无需导入以下包
# import numpy as np
# import os
# import pandas as pd
# ...

s = pd.Series([1,3,5,7,9])

print(s)

```

> <IPython.core.display.Javascript object> 0 1 1 3 2 5 3 7 4 9 dtype: int64

可以查看一下 pyforest 导入了哪些 python 标准库（也就是 python 中的内置库）

```
list_ = [n for n in dir(pyforest)]

print(f'python内置库的总数是：{str(len(list_))}')
# python内置库的总数是：105

print(list_)

# ['ARIMA', 'CountVectorizer', 'ElasticNet', 'ElasticNetCV', 'GradientBoostingClassifier',
# 'GradientBoostingRegressor', 'GridSearchCV', 'Image', 'KFold', 'KMeans', 'LabelEncoder',
# 'Lasso', 'LassoCV', 'LazyImport', 'LinearRegression', 'LogisticRegression', 'MinMaxScaler',
# 'OneHotEncoder', 'PCA', 'Path', 'PolynomialFeatures', 'Prophet', 'RandomForestClassifier',
# 'RandomForestRegressor', 'RandomizedSearchCV', 'Ridge', 'RidgeCV', 'RobustScaler', 'SimpleImputer',
# 'SparkContext', 'StandardScaler', 'StratifiedKFold', 'TSNE', 'TfidfVectorizer', '__builtins__',
# '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__path__',
# '__spec__', '__version__', '_importable', '_imports', '_jupyter_labextension_paths',
# '_jupyter_nbextension_paths', 'active_imports', 'alt', 'bokeh', 'cross_val_score', 'cv2', '
# dash', 'dd', 'dt', 'fastai', 'fbprophet', 'gensim', 'get_user_symbols', 'glob', 'go',
# 'import_symbol', 'imutils', 'install_extensions', 'install_labextension', 'install_nbextension',
# 'keras', 'lazy_imports', 'lgb', 'load_workbook', 'metrics', 'mpl', 'nltk', 'np', 'open_workbook',
# 'os', 'pd', 'pickle', 'plt', 'px', 'py', 'pydot', 'pyforest_imports', 're', 'sg', 'skimage',
# 'sklearn', 'sm', 'sns', 'spacy', 'statistics', 'stats', 'svm', 'sys', 'textblob', 'tf', 'torch',
# 'tqdm', 'train_test_split', 'user_specific_imports', 'user_symbols', 'utils', 'wr', 'xgb']

```