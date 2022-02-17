> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTU5OTI4MA==&mid=2247504124&idx=2&sn=e36beab9cad5f38d538135f5b6a63067&chksm=fbaff1a3ccd878b545f946e7f2f58914722aca846f86bb04b230cdcdd4d7d730e22b3fb63b5c&mpshare=1&scene=1&srcid=0206MtG5ssA7eo0WH0Ocv03t&sharer_sharetime=1644126312919&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

我们都知道，Pandas 擅长处理大量数据并以多种文本和视觉表示形式对其进行总结，它支持将结构输出到 CSV、Excel、HTML、json 等。但是如果我们想将多条数据合并到一个文档中，就有些复杂了。例如，如果要将两个 DataFrames 放在一张 Excel 工作表上，则需要使用 Excel 库手动构建输出。虽然可行，但并不简单。本文将介绍一种将多条信息组合成 HTML 模板，然后使用 Jinja 模板和 WeasyPrint 将其转换为独立 PDF 文档的方法，一起来看看吧~

总体流程
----

如报告文章所示，使用 Pandas 将数据输出到 Excel 文件中的多个工作表或从 pandas DataFrames 创建多个 Excel 文件都非常方便。但是，如果我们想将多条信息组合到一个文件中，那么直接从 Pandas 中完成的简单方法却并不多，下面我们来探索一条可行的简单方法

在本文中，我将使用以下流程来创建多页 PDF 文档

![图片](https://mmbiz.qpic.cn/mmbiz_png/4J9DtZ0rhjBYTgXnSw2iaYeYNP2xdRSfJnFScjiaibjVMQCpQfJIBF7Xk4EmibGPsRZ4aibib7LWREyb7Xbuvb3ibqlGA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

这种方法的好处是我们可以将自己的工具替换到此工作流程中。不喜欢用 Jinja？那么可以插入 mako 或其他任何模板工具

工具选择
----

首先，我们使用 HTML 作为模板语言，因为它可能是生成结构化数据并允许设置相对丰富的格式的最简单方法

其次，选择 Jinja 是因为我有使用 Django/Flask 的经验，上手比较容易

这个工具链中最困难的部分是弄清楚如何将 HTML 呈现为 PDF。我觉得目前还没有非常好的解决方案，我这里选择了 WeasyPrint，大家也可以尝试一下其他的工具

数据处理
----

导入模块，读取销售信息

```
from __future__ import print_function
import pandas as pd
import numpy as np
df = pd.read_excel("sales-funnel.xlsx")
df.head()


```

Output:

![图片](https://mmbiz.qpic.cn/mmbiz_png/4J9DtZ0rhjBYTgXnSw2iaYeYNP2xdRSfJvXCHPjejnA4hsT2qO0icoib6Okib4Z5NRAQuQO2nGxv6snnarVCqllS9Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

将数据进行透视表汇总处理

```
sales_report = pd.pivot_table(df, index=["Manager", "Rep", "Product"], values=["Price", "Quantity"],
                           aggfunc=[np.sum, np.mean], fill_value=0)
sales_report.head()


```

Output:

![图片](https://mmbiz.qpic.cn/mmbiz_png/4J9DtZ0rhjBYTgXnSw2iaYeYNP2xdRSfJU1QTJqAHcoXkyialkVSCEDp9UOSOxDCONYOOzKD8bq4va7rglwAOLRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

模板
--

Jinja 模板非常强大，支持许多高级功能，例如沙盒执行和自动转义等等

Jinja 的另一个不错的功能是它包含多个内置过滤器，这将允许我们以在 Pandas 中难以做到的方式格式化我们的一些数据

为了在我们的应用程序中使用 Jinja，我们需要做 3 件事：

*   创建模板
    
*   将变量添加到模板上下文中
    
*   将模板渲染成 HTML
    

我们先创建一个简单的模板 myreport.html

```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
    <h2>Sales Funnel Report - National</h2>
     {{ national_pivot_table }}
</body>
</html>


```

此代码的两个关键部分是 {{ title }} 和 {{ national_pivot_table }}。它们本质上是我们在渲染文档时将提供的变量的占位符

要填充这些变量，我们需要创建一个 Jinja 环境并获取我们的模板：

```
from jinja2 import Environment, FileSystemLoader
env = Environment(loader=FileSystemLoader('.'))
template = env.get_template("myreport.html")


```

在上面的示例中，我们假设模板位于当前目录中

另一个关键组件是 env 的创建，这个变量是我们将内容传递给模板的方式。我们创建一个名为 template_var 的字典，其中包含我们要传递给模板的所有变量

变量的名称与我们的模板匹配

```
template_vars = {"title" : "Sales Funnel Report - National",
                 "national_pivot_table": sales_report.to_html()}


```

最后一步是使用输出中包含的变量来呈现 HTML，这将创建一个字符串，我们最终将传递给我们的 PDF 创建引擎

```
html_out = template.render(template_vars)


```

生成 PDF
------

PDF 创建部分也相对简单，我们需要做一些导入并将一个字符串传递给 PDF 生成器

```
from weasyprint import HTML
HTML(string=html_out).write_pdf("report.pdf")


```

此命令会创建一个如下所示的 PDF 报告：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4J9DtZ0rhjBYTgXnSw2iaYeYNP2xdRSfJ02zeAUsG7AgBtw5CpAxLkq6hHEl8YiaPD4xbU5bLmkwxqI4zgartF9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

虽然报告生成了，但是看起来很难看啊，我们来优化下，添加 CSS

这里使用 blue print 的 typography.css 作为我们的 style.css 的基础，它有以下几个优点：

*   它比较小且易于理解
    
*   它可以在 PDF 引擎中工作而不会引发错误和警告
    
*   它包括看起来相当不错的基本表格格式
    

```
HTML(string=html_out).write_pdf(args.outfile.name, stylesheets=["style.css"])


```

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/4J9DtZ0rhjBYTgXnSw2iaYeYNP2xdRSfJFFBZTwKc4RbyD9y5QywqzbNvXDzIaFxYJJIMvOTicXT7FMqVYg2fKGg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

可以看到，仅仅添加一行代码，产生的效果却大大不同

更复杂的模板
------

为了生成更有用的报告，我们将结合上面显示的汇总统计数据，并将报告拆分为每个经理包含一个单独的 PDF 页面

让我们从更新的模板（myreport.html）开始：

```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>{{ title }} </title>
</head>
<body>
<div class="container">
    <h2>Sales Funnel Report - National</h2>
     {{ national_pivot_table }}
    {% include "summary.html" %}
</div>
<div class="container">
    {% for manager in Manager_Detail %}
        <p style="page-break-before: always" ></p>
        <h2>Sales Funnel Report - {{manager.0}}</h2>
        {{manager.1}}
        {% include "summary.html" %}
    {% endfor %}
</div>
</body>
</html>


```

我们注意到的第一件事是有一个包含语句，它提到了另一个文件。包含允许我们引入一段 HTML 并在代码的不同部分重复使用它。在这种情况下，摘要包含一些我们希望在每个报告中包含的简单的国家级统计数据，以便管理人员可以将他们的绩效与全国平均水平进行比较。

以下是 summary.html 的样子：

```
<h3>National Summary: CPUs</h3>
    <ul>
        <li>Average Quantity: {{CPU.0|round(1)}}</li>
        <li>Average Price: {{CPU.1|round(1)}}</li>
    </ul>
<h3>National Summary: Software</h3>
    <ul>
        <li>Average Quantity: {{Software.0|round(1)}}</li>
        <li>Average Price: {{Software.1|round(1)}}</li>
    </ul>


```

在此代码段中，看到我们可以访问一些其他变量：CPU 和 Software 。其中每一个都是一个 python 列表，其中包括 CPU 和软件销售的平均数量和价格

还注意到我们使用管道`|`将每个值四舍五入到小数点后 1 位。这是使用 Jinja 过滤器的一个具体示例

还有一个 `for` 循环允许我们在报告中显示每个经理的详细信息。Jinja 的模板语言只包含一个非常小的代码子集，它会改变控制流

附加统计信息
------

下面编写供模板调用的函数和代码

一个简单的汇总函数

```
def get_summary_stats(df,product):
    """
    For certain products we want National Summary level information on the reports
    Return a list of the average quantity and price
    """
    results = []
    results.append(df[df["Product"]==product]["Quantity"].mean())
    results.append(df[df["Product"]==product]["Price"].mean())
    return results


```

创建经理详细信息

```
manager_df = []
for manager in sales_report.index.get_level_values(0).unique():
    manager_df.append([manager, sales_report.xs(manager, level=0).to_html()])


```

最后，使用以下变量调用模板

```
template_vars = {"title" : "National Sales Funnel Report",
                 "CPU" : get_summary_stats(df, "CPU"),
                 "Software": get_summary_stats(df, "Software"),
                 "national_pivot_table": sales_report.to_html(),
                 "Manager_Detail": manager_df}
# Render our file and create the PDF using our css style file
html_out = template.render(template_vars)
HTML(string=html_out).write_pdf("report.pdf",stylesheets=["style.css"])


```

这样我们的 pdf 报表就完成了，整体效果如下

![图片](https://mmbiz.qpic.cn/mmbiz_gif/4J9DtZ0rhjBYTgXnSw2iaYeYNP2xdRSfJNzRITicyGLYkcfNTkJXiaiaTC3cSaXPiayK3GjkhgbJ8JuDwJf3PgCYluQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

  

完整代码：

```
from __future__ import print_function
import pandas as pd
import numpy as np
import argparse
from jinja2 import Environment, FileSystemLoader
from weasyprint import HTML


def create_pivot(df, infile, index_list=["Manager", "Rep", "Product"], value_list=["Price", "Quantity"]):
    """
    Create a pivot table from a raw DataFrame and return it as a DataFrame
    """
    table = pd.pivot_table(df, index=index_list, values=value_list,
                           aggfunc=[np.sum, np.mean], fill_value=0)
    return table

def get_summary_stats(df,product):
    """
    For certain products we want National Summary level information on the reports
    Return a list of the average quantity and price
    """
    results = []
    results.append(df[df["Product"]==product]["Quantity"].mean())
    results.append(df[df["Product"]==product]["Price"].mean())
    return results

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='Generate PDF report')
    parser.add_argument('infile', type=argparse.FileType('r'),
    help="report source file in Excel")
    parser.add_argument('outfile', type=argparse.FileType('w'),
    help="output file in PDF")
    args = parser.parse_args()

    df = pd.read_excel(args.infile.name)
    sales_report = create_pivot(df, args.infile.name)

    manager_df = []
    for manager in sales_report.index.get_level_values(0).unique():
        manager_df.append([manager, sales_report.xs(manager, level=0).to_html()])

    env = Environment(loader=FileSystemLoader('.'))
    template = env.get_template("myreport.html")
    template_vars = {"title" : "National Sales Funnel Report",
                     "CPU" : get_summary_stats(df, "CPU"),
                     "Software": get_summary_stats(df, "Software"),
                     "national_pivot_table": sales_report.to_html(),
                     "Manager_Detail": manager_df}

    html_out = template.render(template_vars)
    HTML(string=html_out).write_pdf(args.outfile.name,stylesheets=["style.css"])


```