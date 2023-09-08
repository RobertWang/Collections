> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qx6wi6eJZZnIybk1hkF1_A)

**核心点**：**sweetviz，一行代码生成一个详细数据报告！**

嗨，我是 cos 大壮！  

今天介绍一个将数据完整、快速、有效的生成漂亮的可视化图的开源库：**Sweetviz**。

将 EDA(探索性数据分析) 作为一个 HTML 应用程序启动。

Sweetviz 包是围绕快速可视化目标值和比较数据集构建的。

生成的 HTML 文件中，会显示数据集、相关性分析、分类、数字特征等等非常全面的图表。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/kibwfTuPM4lib8o3U8ibrnKyhtl4Nep8pGtDJJaQzZ20dvibArzwJ3m4lQA2e4kkhFqsXNJBSczbV6Il0RrlvQibic4Q/640?wx_fmt=png)

今天这个 Python 数据分析、数据挖掘的库，大家试着使用起来，真的非常强大！

觉得有用的朋友可以收藏、点赞、转发起来。

安装
--

安装的话，非常方面，使用`pip`即可。

```
pip install sweetviz


```

我这里使用的是 python3.11，python 3.7 + 其实就可以。

数据准备
----

数据准备的是 **titanic.csv**（kaggle 泰坦尼克号数据集）

如果想要这个数据的同学，后台回复 “**数据集**” 即可！

该数据有 11 列属性，分别是乘客的 ID、获救情况，乘客等级、姓名、性别、年龄、堂兄弟妹个数、父母与小孩的个数、船票信息、票价、船舱、登船的港口。

使用起来
----

使用方式非常简单！

当然，也有很多其他更加高级的使用方式。可以在原 github 中进行查看【文末阅读原文获取】

```
import sweetviz as sv
sweet_report = sv.analyze(pd.read_csv("titanic.csv"))
sweet_report.show_html('titanic.html')


```

之后，即可生成各维度下的数据分析情况。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/kibwfTuPM4lib8o3U8ibrnKyhtl4Nep8pGtZ8LXQyFicVgnPrmbSAaic0s3ULPvBicxseGV8RKWwXgct8lGVA8rpickAA/640?wx_fmt=png)

还可以做一个 Associations，结果展示

![](https://mmbiz.qpic.cn/sz_mmbiz_png/kibwfTuPM4lib8o3U8ibrnKyhtl4Nep8pGtWTI8jGcoib875B0G1QdGWnxNeVia3BCOZfGxYmUZ4v5C3lQaVicpwxMuw/640?wx_fmt=png)

是不是在一些简单想要看数据的情况时候，非常好用！

更多的可视化使用的方法，大家可以自己在探索一下。

github 地址：https://github.com/fbdesignpro/sweetviz

最后
--

这个项目对于数据可视化的同学来说，可以作为一个轻量级的使用场景，是极其方便的！