> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Iu_B_-tVV7upgNeHVg-yFw)

大家好，我是小寒。

今天给大家分享一个强大的 python 库，**latexify**。

latexify 是一个开源的 Python 库，旨在**将 Python 代码转换为 LaTeX 格式的方程**。

在本文中，我们将探讨如何利用 Latexify 让你在**显示数学表达式时变得更轻松。‍**

在深入了解 **Latexify** 之前，我们首先了解一下什么是 **LaTeX** 。

LaTeX 是一种排版系统，通常用于**创建包含复杂数学公式等的文档**。它是一种标记语言，使用户能够以可读且美观的格式**表示数学符号。‍**

### 为什么使用 Latexify ？

1.  **易于使用**：它可以将 Python 函数、方程或类似 NumPy 的表达式**直接转换为 LaTeX 代码，无需手动转换。‍**
    
2.  **节省时间**：通过以编程方式生成 LaTeX 代码来加快工作流程。
    
3.  **兼容性**：创建的 LaTeX 代码可以嵌入到 Jupyter 笔记本、论文、演示文稿或网站中。
    
4.  **可读性**：使你的方程看起来专业且易于理解，这对于学术或研究目的特别有用。
    

### 初体验

#### 库的安装

可以直接通过 pip 进行安装。

```
pip install latexify-py

```

#### 基本用法

下面是**将 Python 函数转换为 LaTeX 代码**的基本示例。

```
import latexify
import math

```

然后开始在单独的函数中编写公式，例如用于两个数字求和的函数。

```
@latexify.expression
def sum(a, b):
    return a + b

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibPqnScajrbHzsmMeCSYBZetvN0aUCcZqkn9qOc3HpxNbUPPERoaw1bdXEjicTh18M11EO57uMrJQFKcjpSNdm6A/640?wx_fmt=png)

创建指数函数。

```
@latexify.with_latex
def exponential(x):
    return math.exp(x)
exponential

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibPqnScajrbHzsmMeCSYBZetvN0aUCcZqELdQ6ZbKYv4R6NdVKYc54iaAacZ8StoATJErZnQgIBwDPBVldDvXia4A/640?wx_fmt=png)

二次方程公式。

```
@latexify.with_latex
def quadratic(a, b, c):
    return (-b + math.sqrt(b**2 - 4*a*c)) / (2*a), (-b - math.sqrt(b**2 - 4*a*c)) / (2*a)
quadratic

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibPqnScajrbHzsmMeCSYBZetvN0aUCcZqCic7xdlhe5Dowtvmloh1QWNPDsia6HaOZlJFNVCtlU62FiaQbmKEJNBQA/640?wx_fmt=png)

**为了获得不带等号的方程，需要将注释从 @latexify.with_latex 更改为 @latexify.expression。**

```
@latexify.expression
def quadratic(a, b, c):
   return (-b + math.sqrt(b**2 - 4*a*c)) / (2*a), (-b - math.sqrt(b**2 - 4*a*c)) / (2*a)
quadratic

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibPqnScajrbHzsmMeCSYBZetvN0aUCcZqZjQw4VhES8OnaxFPNhngmakgAHokg8ibHjCat5PBNSLlMnP9DsC0Kow/640?wx_fmt=png)

#### 高级功能

条件函数。

```
@latexify.with_latex
def piecewise(x):
    if x < 0:
        return -x
    else:
        return x**2
piecewise

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibPqnScajrbHzsmMeCSYBZetvN0aUCcZq6ibCZ6ibLzPVNOquOcrBNBfJ9zlOSnhoWrfCib1dqeWaAh91FH3U75lwA/640?wx_fmt=png)

```
identifiers = {
    "jaccard_similarity": "J", 
    "a": "A",
    "b": "B",
    "len": "cardinality"
}

@latexify.function(use_set_symbols=True, identifiers=identifiers)
def jaccard_similarity(a, b):
    return len(a & b) / len(a | b)

jaccard_similarity

```

![](https://mmbiz.qpic.cn/mmbiz_png/ibPqnScajrbHzsmMeCSYBZetvN0aUCcZqOgRylakrjibnrcymibibiardtZib0KAlCWLibgJ9QCT13D9Ou2m2ILu227dQ/640?wx_fmt=png)

虽然 Latexify 是一个强大的工具，但它也有一些局限性。

例如，它可能不完全支持某些高级 Python 功能或没有简单数学表示的自定义类和函数。

通过将 Python 代码无缝转换为 LaTeX 格式的方程，Latexify 实现了一种快速、有效且 Python 的方式来处理数学符号。