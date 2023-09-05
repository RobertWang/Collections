> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dlx7-vXv3-HX9ysj2NZmHg)

Pandas 是我们日常处理表格数据最常用的包，但是对于数据分析来说，Pandas 的 DataFrame 还不够直观，所以今天我们将介绍 4 个 Python 包，可以将 Pandas 的 DataFrame 转换交互式表格，让我们可以直接在上面进行数据分析的操作。

Pivottablejs
------------

Pivottablejs 是一个通过 IPython widgets 集成到 Python 中的 JavaScript 库，允许用户直接从 DataFrame 数据创建交互式和灵活的汇总报表。可以进行高效、清晰的数据分析和表示，帮助将数据从 Pandas DataFrame 转换为易于观察的交互式数据透视表。

![](https://mmbiz.qpic.cn/mmbiz_png/gX9JE5tiaI7iceuxQcKT3xiaLCICecbwXrATvnic68e5BRDoaXiblibnXNyiakTiauYytPvzrPKZ6Cfib6p5RXV00bQvcXw/640?wx_fmt=png)

pivot_ui 函数可以自动从 DataFrame 生成交互式用户界面，使用户可以简单地修改，检查聚合项，并快速轻松地更改数据结构。

```
 !pip install pivottablejs
 
 from pivottablejs import pivot_ui
 import pandas as pd
 
 data = pd.read_csv("D:\Data\company_unicorn.csv")
 data["Year"] = pd.to_datetime(data["Date Joined"]).dt.year
 pivot_ui(data)


```

如下图所示，我们可以直接在 notebook 中对 DataFrame 进行筛选，生成图表

![](https://mmbiz.qpic.cn/mmbiz_gif/6wQyVOrkRNIWPWBDgXJRPLc7icmbmm2SgpAKGESNK5STvaicvlPG8eBynp0DFhCPxnHz1khOmbgInOia40haC64iag/640?wx_fmt=gif)

我们还可以快速生成数据透视表

![](https://mmbiz.qpic.cn/mmbiz_gif/6wQyVOrkRNIWPWBDgXJRPLc7icmbmm2SgOBt3xibypHaGlIM8OjclRibQez5s8tvNoCrBYt3ic1h4RoRN7QfcmHRjw/640?wx_fmt=gif)

Pygwalker
---------

PyGWalker 可以把 DataFrame 变成一个表格风格的用户界面，让我们直观有效地探索数据。

![](https://mmbiz.qpic.cn/mmbiz_png/gX9JE5tiaI7iceuxQcKT3xiaLCICecbwXrA1cYR2Iqnrd4qN0c674RGDniazD1aiaZNY5GxP9SragEYiaH8nznXAM5IA/640?wx_fmt=png)

这个包的用户界面对 Tableau 用户来说很熟悉，如果你用过 Tableau 那么上手起来就很容易

```
 !pip install pygwalker
 
 import pygwalker as pyw
 walker = pyw.walk(data)


```

![](https://mmbiz.qpic.cn/mmbiz_gif/6wQyVOrkRNIWPWBDgXJRPLc7icmbmm2SgOWPN19Lol4GDIYystTx84iaqDveZmWHs0MLLOqWZSeGCjibN3ANdavEg/640?wx_fmt=gif)img

通过一些简单的拖拽，可以进行筛选和可视化，这是非常方便的

Qgrid
-----

![](https://mmbiz.qpic.cn/mmbiz_png/gX9JE5tiaI7iceuxQcKT3xiaLCICecbwXrAhuoT75SqedsHiaNSB1mxpRqibk8sqe3dA8gCDhvoTt5evZkTiccp6Rtibw/640?wx_fmt=png)

除了 PyGWalker 之外，Qgrid 也是一个很好的工具，它可以很容易地将 DataFrame 架转换为视觉上直观的交互式数据表。

```
 import qgrid
 qgridframe = qgrid.show_grid(data, show_toolbar=True)
 qgridframe


```

![](https://mmbiz.qpic.cn/mmbiz_gif/6wQyVOrkRNIWPWBDgXJRPLc7icmbmm2Sg2CZAoZ8dKREH3VF7q0Pmqe9U68h600sv1dpcoNjDzUS5odbc2noQPA/640?wx_fmt=gif)

我们还可以直接在表上添加、删除数据

Itables
-------

![](https://mmbiz.qpic.cn/mmbiz_png/gX9JE5tiaI7iceuxQcKT3xiaLCICecbwXrAwemmXwP1gNoVicON5WQn9IoeYiczPjicalbu1duXY4Ew2rnPyiaoLyJ5qg/640?wx_fmt=png)

与上面提到的 qgrid 包一样，Itables 提供了一个简单的接口。可以进行简单的操作，如过滤、搜索、排序等。

```
 from itables import init_notebook_mode, show
 init_notebook_mode(all_interactive=False)
 
 show(data)


```

![](https://mmbiz.qpic.cn/mmbiz_gif/6wQyVOrkRNIWPWBDgXJRPLc7icmbmm2SgcLZf7owCHh6SjAoPo4AXBCc3PCWr5lawcmgLXydzLkChXfXnr84kqw/640?wx_fmt=gif)

tables 和 Qgrid 包对于快速查看数据模式是必要的。然而，如果我们想要进一步理解数据并进行数据转换，它们的特征是不够的。因此，在获得更复杂的见解的情况下，使用透视表 js 和 Pygwalker 是可取的。

总结
--

上面的这些包可以在 Jupyter Notebook 中将 dataframe 转换为交互式表。

Itables 和 Qgrid 比较轻量，可以让我们快速的查看数据，但是如果你想进行更多的操作，例如生成一些简单的可视化图表，那么 Pivottablejs 和 Pygwalker 是一个很好的工具。

作者：Chi Nguyen