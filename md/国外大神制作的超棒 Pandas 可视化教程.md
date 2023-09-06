> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652570851&idx=2&sn=b204af97ccd263d6551bb69e8adab576&chksm=84652ca9b312a5bf16cf12b87421381348b6511d163bd7e69a1e953a149b1532045c76812c17&scene=21#wechat_redirect)

> 作者：Jay Alammar，编译：极客猴

Pandas 有个核心类型叫 DataFrame。DataFrame 是表格型的数据结构。因此，我们可以将其当做表格。DataFrame 是以表格类似展示，而且还包含行标签、列标签。另外，每列可以是不同的值类型 (数值、字符串、布尔型等)。

我们可以使用 read_csv() 来加载 CSV 文件。

```
# 加载音乐流媒体服务的 CSV 文件
df = pandas.read_csv('music.csv')

```

其中变量 DF 是 Pandas 的 DataFrame 类型。

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicpQuoya57ia9OsUkWyiaheicvT70jJw5rUT4czPMgPTibsKM31xQKApsRxNA/640?wx_fmt=png)

Pandas 同样支持操作 Excel 文件，使用 read_excel() 接口能从 EXCEL 文件中读取数据。

**2. 选择数据**

我们能使用列标签来选择列数据。比如，我们想获取 Artist 所在的整列数据, 可以将 artists 当做下标来获取。

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicpibSmdVHNZKCOxicFufrRDjWSWtjuwAACCVFOyicOiaxCuwhiakuj8hAriabQ/640?wx_fmt=png)

同样，我们可以使用行标签来获取一列或者多列数据。表格中的下标是数字，比如我们想获取第 1、2 行数据，可以使用 df[1:3] 来拿到数据。

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicp5NRdKoYIEOQn5mIib9IuNcPrCxicvzFjfxPE5ibxJKEZV8wlOJicq2aNwA/640?wx_fmt=png)

Pandas 的利器之一是索引和数据选择器。我们可以随意搭配列标签和行标签来进行切片，从而得到我们所需要的数据。比如，我们想得到第 1, 2, 3 行的 Artist 列数据。

```
import pandas as pd

df.loc[1:3, ['Artist']]
# loc(这里会包含两个边界的行号所在的值)


```

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicpARXbIAdboiaAr5XYrH9sEyG9icDF1BVorS2vWl0H3zBjC3IoicsKeaickw/640?wx_fmt=png)

**3. 过滤数据**

过滤数据是最有趣的操作。我们可以通过使用特定行的值轻松筛选出行。比如我们想获取音乐类型 (Genre) 为值为 Jazz 行。

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicpujd0mlvdsibGTZSsicaYXYg6QRKCj0oJ0bsv3svIibz4MzibA2ehWy3hrA/640?wx_fmt=png)

再比如获取超过 180 万听众的 艺术家。

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicpYUADwaxsDibR1XyicO2OyE8eegh941GLw3RU14ML1doOSw0BiaeJvGvHg/640?wx_fmt=png)

**4. 处理空值**

数据集来源渠道不同，可能会出现空值的情况。我们需要数据集进行预处理时。

如果想看下数据集有哪些值是空值，可以使用 isnull() 函数来判断

```
import pandas as pd

df = pd.read_csv('music.csv')
print(df.isnull())

```

假设我们之前的音乐数据集中 有空值 (NaN) 的行。

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicpzlZT4qtUCwzJv77bWqb4ficrroDrWIsichCdLicia06y3osKfzPjcn4N1Q/640?wx_fmt=png)

我们对之前的音乐. csv 文件进行判断，得到结果如下:

![](https://mmbiz.qpic.cn/mmbiz_jpg/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicpbuicRR5CdZOJ9ulAgq4JXdqAdKUGOcCicBIEka88DVMSCj1EZdjeFcLw/640?wx_fmt=jpeg)

如果我想知道哪列存在空值，可以使用 df.isnull().any()

结果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicp0t0InvLU5bBeHjNKujBZ7TuvXBPciaNuKGCR4d7evFRB4RKTdr8WPpQ/640?wx_fmt=jpeg)

处理空值，Pandas 库提供很多方式。最简单的办法就是删除空值的行。

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicp4yrzhiaBPXws34FjD0tq052OFp6ichXtiaz2Y11KsCjiaibS1WicZ6Gf3YDA/640?wx_fmt=png)

除此之外，还可以使用取其他数值的平均值，使用出现频率高的值进行填充缺失值。

```
import pandas as pd

# 将值填充为 0
pd.fillna(0)

```

**5. 分组**

我们使用特定条件进行分组并聚它们的数据，也是很有意思的操作。比如，我们需要将数据集以音乐类型进行分组，以便我们能更加方便、清晰了解每个音乐类型有多少听众和播放量。

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicpPiah4lgYgxZUW9EY9cqqffCdgyBKticPcKmYnXGK0tdCy1WgZVVibtaDA/640?wx_fmt=png)

上述代码的的执行过程是：Pandas 会将 Jazz 音乐类型的两行数据聚合一组；我们调用了 sum() 函数，Pandas  还会将这两行数据端的 Listeners(听众) 和 Plays (播放量) 相加在一起，然后组合在 Jazz 列中显示总和。

这也是 Pandas 库强大之处，能将多个操作进行组合，然后显示最终结果。

**6. 从现有列中创建新列**

通常在数据分析过程中，我们发现自己需要从现有列中创建新列，使用 Pandas 也是能轻而易举搞定。

![](https://mmbiz.qpic.cn/mmbiz_png/RvCRMgjJ1SOl6YgXzHukO3LdGHSoMSicphQRhJk4vePkShbiaGrhoWSEjBvzSeM6fg943JtwbnEtYDW2PK2qpy4w/640?wx_fmt=png)