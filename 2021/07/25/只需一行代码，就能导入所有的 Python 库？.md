> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2MDY2NzQwMQ==&mid=2247492323&idx=2&sn=6f34a13af073978a1d0403603640a8e7&chksm=ea648385dd130a93a5369ae251f3edfa6869954c1aa4c0da4831824277edc5f81764083d5286&mpshare=1&scene=1&srcid=07244s6IB9PlYOMB8m6vvOJt&sharer_sharetime=1627115599629&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

GitHub 地址：_https://github.com/8080labs/pyforest_

Python 因为有着成千上万个功能强大的开源库，备受大家的欢迎。

而且每当新建一个程序文件时，都需要根据自己的需求导入相关的库。

如果是相同类型的任务，比如想做一个数据可视化的小项目，可能会一直使用到某个库。

如此，反复编写同一条 import 语句，就算是复制粘贴，也会感觉到麻烦，这时 Pyforest 库就可以上场了。

Pyforest 是一个开源的 Python 库，可以自动导入代码中使用到的 Python 库。

在进行数据可视化的时候，一般都需要导入多个库，比如 pandas、numpy、matplotlib 等等。

使用了 Pyforest，每个程序文件中就不需要导入相同的 Python 库，而且也不必使用确切的导入语句。

比如下面这行代码，就可以省略掉。

在你使用 import 语句导入 Pyforest 库后，你就可以直接使用所有的 Python 库。

你使用的任何库都不需要使用 import 语句导入，Pyforest 会为你自动导入。

只有在代码中调用库或创建库的对象后，才会导入库。如果一个库没有被使用或调用，Pyforest 将不会导入它。

**/ 02 / 使用**

安装，使用以下命令安装 Pyforest。

安装成功后，使用 import 语句导入它。

现在，你可以直接使用相关的 Python 库，无需编写 import 导入。

先以 jupiter notebook 为例，我们没有导入 pandas、seaborn 和 matplotlib 库，但是我们可以通过导入 Pyforest 库直接使用它们。

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhansl7ibJm1OzvJ3k6ha8BDFzXm4SqyJwsutjTA0WFzCkcHSzyzKLXZ8pmcS4OduJ3NU9RdgIj0QIFQ/640?wx_fmt=png)

读取数据，这个是国内棉花产量排行前三的省份，新疆全国第一 (数据来源：国家统计局)。  

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhansl7ibJm1OzvJ3k6ha8BDFzCDuicpso1EoibFAuhnPuTibeQz8Mhicqezu2ypbqRvibVn2DZXyGVKtgZAQ/640?wx_fmt=png)

那么 Pyforest 可以导入所有库吗？

目前这个包包含了大部分流行的 Python 库，比如

```
pandas as pd
NumPy as np
matplotlob.pyplot as plt
seaborn as sns 
```

除了这些库之外，它还提供了一些辅助的 Python 库，如 os、tqdm、re 等。  

如果你想查看库列表，可以使用 dir(pyforest) 进行查看，内置的是 68 个库。

```
import pyforest

print(len(dir(pyforest)))
for i in dir(pyforest):
    print(i)

-------------------------
GradientBoostingClassifier
GradientBoostingRegressor
LazyImport
OneHotEncoder
Path
RandomForestClassifier
RandomForestRegressor
SparkContext
TSNE
TfidfVectorizer
...
```

如果没有的话，可以进行自定义添加，在主目录中的文件写入 import 语句。

示例如下。

```
vim ~/.pyforest/user_imports.py
```

添加语句，此处便能在代码中使用 requests 这个库。

```
# Add your imports here, line by line
# e.g
# import pandas as pd
# from pathlib import Path
# import re

import requests as req
~                                                                               
~                                                                                                                                                                                                      
"~/.pyforest/user_imports.py" 7L, 129C
```

这回我们在 PyCharm 中来实验一下。

发现 PyCharm 的自动补全的功能失效了，看来这个库还是比较适合 jupyter notebook(自动补全代码还可以使用)。

除了上面这个地方可以自定义添加，还可以在库的_import.py 文件中添加。

此处以 Pyechars 为例，缩写为 chart。

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhansl7ibJm1OzvJ3k6ha8BDFzb5vA6tibkZCNwYlRy04vI7XclSqsknLlAAQgRSx5qggJ7YBiamKcib4FA/640?wx_fmt=png)

可视化代码如下。

![](https://mmbiz.qpic.cn/mmbiz_png/y0SBuxfLhansl7ibJm1OzvJ3k6ha8BDFzmAY3vKMT3C0oiakCAic1h3iaGATXGbsx91revW6W9czBj6CtyyIicSAkeg/640?wx_fmt=png)

新疆棉花产量年年上升，其它省份年年下降...  

最后 Pyforest 还提供了一些函数来了解库的使用情况。

```
# 返回已导入并且正在使用的库列表
print(pyforest.active_imports())
--------------------------------
['import pandas as pd', 'import requests as req', 'import pyg2plot']


# 返回pyforest中所有Python库的列表
print(pyforest.lazy_imports())
--------------------------------
['import glob', 'import numpy as np', 'import matplotlib.pyplot as plt'...]
```

只有代码中有使用到的库，程序才会 import 进去，否则不会导入的哦！

**/ 03 / 总结**

好了，到此本期的分享就结束了。

使用 Pyforest 库有时候确实是可以节省一些时间，不过也是有弊端存在的。

比如调试的时候 (大型项目)，可能会很痛苦，不知道是哪里来的库。  

所以建议大家，在一些独立的脚本程序中使用，效果应该还是不错的。

****人生苦短，快学 Python **👍******