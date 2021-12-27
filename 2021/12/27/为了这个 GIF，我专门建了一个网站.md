> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/cGGDL0cSpkASqHItgbMKMg)

![](https://mmbiz.qpic.cn/mmbiz_gif/njjfaJS7c9qRUnuwEn0qMpaoicmZ1LYcGRHbQlSmzFQhQsauAsibWu5lAC8y7K0jMzYacXKPNFr9cMUbR8edaiciag/640?wx_fmt=gif)这个叫条形竞赛图，非常适合制作随时间变动的数据可视化动图。

我已经用 streamlit+bar_chart_race 实现了，然后白嫖了 heroku 的服务器，大家通过下面的网址上传 csv 格式的表格就可以轻松制作条形竞赛图，生成的视频可以保存本地。

https://bar-chart-race-app.herokuapp.com/

本文我将`实现过程`介绍一下，白嫖服务器 + 部署留在下期再讲。

纯 matplotlib 实现
---------------

注：以下所有实现方式都需要提前安装 ffmpeg, 安装方式我之前在[决策树可视化](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2648939588&idx=2&sn=1dc10660a667f425d2477ecba042b9d1&chksm=87940e6eb0e38778f501f91cd3c2a80f83d034d8431641d5ab3e7e45185eb417c2d744fd87f5&token=748838886&lang=zh_CN&scene=21#wechat_redirect)一文中有介绍

matplotlib 实现 bar-chart-race 很简单，直接上代码

```
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import matplotlib.animation as animation
from IPython.display import HTML
url = 'https://gist.githubusercontent.com/johnburnmurdoch/4199dbe55095c3e13de8d5b2e5e5307a/raw/fa018b25c24b7b5f47fd0568937ff6c04e384786/city_populations'
df = pd.read_csv(url, usecols=['name', 'group', 'year', 'value'])
colors = dict(zip(
    ["India", "Europe", "Asia", "Latin America", "Middle East", "North America", "Africa"],
    ["#adb0ff", "#ffb3ff", "#90d595", "#e48381", "#aafbff", "#f7bb5f", "#eafb50"]
))
group_lk = df.set_index('name')['group'].to_dict()
fig, ax = plt.subplots(figsize=(15, 8))

def draw_barchart(current_year):
    dff = df[df['year'].eq(current_year)].sort_values(by='value', ascending=True).tail(10)
    ax.clear()
    ax.barh(dff['name'], dff['value'], color=[colors[group_lk[x]] for x in dff['name']])
    dx = dff['value'].max() / 200
    for i, (value, name) in enumerate(zip(dff['value'], dff['name'])):
        ax.text(value-dx, i,     name,           size=14, weight=600, ha='right', va='bottom')
        ax.text(value-dx, i-.25, group_lk[name], size=10, color='#444444', ha='right', va='baseline')
        ax.text(value+dx, i,     f'{value:,.0f}',  size=14, ha='left',  va='center')
    ax.text(1, 0.4, current_year, transform=ax.transAxes, color='#777777', size=46, ha='right', weight=800)
    ax.text(0, 1.06, 'Population (thousands)', transform=ax.transAxes, size=12, color='#777777')
    ax.xaxis.set_major_formatter(ticker.StrMethodFormatter('{x:,.0f}'))
    ax.xaxis.set_ticks_position('top')
    ax.tick_params(axis='x', colors='#777777', labelsize=12)
    ax.set_yticks([])
    ax.margins(0, 0.01)
    ax.grid(which='major', axis='x', linestyle='-')
    ax.set_axisbelow(True)
    ax.text(0, 1.15, 'The most populous cities in the world from 1500 to 2018',
            transform=ax.transAxes, size=24, weight=600, ha='left', va='top')
    ax.text(1, 0, 'by @pratapvardhan; credit @jburnmurdoch', transform=ax.transAxes, color='#777777', ha='right',
            bbox=dict(facecolor='white', alpha=0.8, edgecolor='white'))
    plt.box(False)
    
fig, ax = plt.subplots(figsize=(15, 8))
animator = animation.FuncAnimation(fig, draw_barchart, frames=range(1900, 2019))
HTML(animator.to_jshtml())


```

核心是定义`draw_barchart`函数绘制当前图表的样式，然后用`animation.FuncAnimation`重复调用`draw_barchart`来制作动画, 最后用`animator.to_html5_video()` 或 `animator.save()`保存 GIF / 视频。

xkcd 手绘风格
---------

![](https://mmbiz.qpic.cn/mmbiz_gif/njjfaJS7c9qRUnuwEn0qMpaoicmZ1LYcGlyvL0xr6z7picGbYw681MZfBPb99UFqj4zovMavicia7lBVZbzB9GU2VQ/640?wx_fmt=gif)我们也可以用`matplotlib.pyplot.xkcd`函数绘制 XKCD 风格的图表，方法也很简单，只需把上面的代码最后一段加上一行

```
with plt.xkcd():
    fig, ax = plt.subplots(figsize=(15, 8))
    animator = animation.FuncAnimation(fig, draw_barchart, frames=range(1900, 2019))
    HTML(animator.to_jshtml())


```

bar_chart_race 库极简实现
--------------------

如果嫌麻烦，还可以使用一个库「Bar Chart Race」，堪称 Python 界最强的动态可视化包。

GitHub 地址：https://github.com/dexplo/bar_chart_race

目前主要有 0.1 和 0.2 两个版本，0.2 版本添加动态曲线图以及 Plotly 实现的动态条形图。

通过 pip install bar_chart_race 也只能到 0.1 版本，因此需要从 GitHub 上下载下来，再进行安装。

```
git clone https://github.com/dexplo/bar_chart_race


```

使用起来就是极简了，三行代码即可实现

```
import bar_chart_race as bcr
# 获取数据
df = bcr.load_dataset('covid19_tutorial')
# 生成GIF图像
bcr.bar_chart_race(df, 'covid19_horiz.gif')


```

![](https://mmbiz.qpic.cn/mmbiz_png/njjfaJS7c9qRUnuwEn0qMpaoicmZ1LYcGy4SBYIqbZxcSua5wN5fbfia6iaolYeGyBVAo3gTRsL7VjD8zfAZWzpeg/640?wx_fmt=png)

实际上 bar_chart_race 还有很多参数可以输出不同形态的 gif

```
bcr.bar_chart_race(
    df=df,
    filename='covid19_horiz.mp4',
    orientation='h',
    sort='desc',
    n_bars=6,
    fixed_order=False,
    fixed_max=True,
    steps_per_period=10,
    interpolate_period=False,
    label_bars=True,
    bar_size=.95,
    period_label={'x': .99, 'y': .25, 'ha': 'right', 'va': 'center'},
    period_fmt='%B %d, %Y',
    period_summary_func=lambda v, r: {'x': .99, 'y': .18,
                                      's': f'Total deaths: {v.nlargest(6).sum():,.0f}',
                                      'ha': 'right', 'size': 8, 'family': 'Courier New'},
    perpendicular_bar_func='median',
    period_length=500,
    figsize=(5, 3),
    dpi=144,
    cmap='dark12',
    title='COVID-19 Deaths by Country',
    title_size='',
    bar_label_size=7,
    tick_label_size=7,
    shared_fontdict={'family' : 'Helvetica', 'color' : '.1'},
    scale='linear',
    writer=None,
    fig=None,
    bar_kwargs={'alpha': .7},
    filter_column_colors=False)  


```

比如以下几种

![](https://mmbiz.qpic.cn/mmbiz_gif/njjfaJS7c9qRUnuwEn0qMpaoicmZ1LYcGj3ozKKuXGNXCcCcQr6YDg9pXf0t20icZpHibcaXiak47PwMm1aFGBY5RA/640?wx_fmt=gif)![](https://mmbiz.qpic.cn/mmbiz_gif/njjfaJS7c9qRUnuwEn0qMpaoicmZ1LYcGczylSMSArXbSHiaB1TfX1OzCS7jhjcRYKhVsRyC1ZVWnPLbW6GHicNbA/640?wx_fmt=gif)![](https://mmbiz.qpic.cn/mmbiz_gif/njjfaJS7c9qRUnuwEn0qMpaoicmZ1LYcG6Mo7cUH8CZE2Dg8tnVnibgd3x0zDicr0dEhCzpmuibeTiaXMcNCAh3UdXw/640?wx_fmt=gif)

更详细的用法大家可以查阅官方文档

地址：https://www.dexplo.org/bar_chart_race/

streamlit+bar_chart_race
------------------------

streamlit 是我最近特别喜欢玩的一个机器学习应用开发框架，它能帮你不用懂得复杂的 HTML,CSS 等前端技术就能快速做出来一个炫酷的 Web APP。

我之前开发的[决策树挑西瓜](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2648961437&idx=1&sn=b8704d462e9d764a8ed54564b2558802&chksm=879463b7b0e3eaa1a5e6c0a678a7c0c70236e7f338ac5397282ecf92359cd4867043f8fd22ac&token=1014330116&lang=zh_CN&scene=21#wechat_redirect)就是使用了 streamlit

下面是 streamlit+bar_chart_race 整体结构![](https://mmbiz.qpic.cn/mmbiz_png/njjfaJS7c9qRUnuwEn0qMpaoicmZ1LYcGGLYI7JzcMdZ6IibM7XnL3BhbDkh0I9bN0DG29E7J3afUA1wVDG9FeDA/640?wx_fmt=png)

核心是 app.py，代码如下：

```
from bar_chart_race import bar_chart_race as bcr
import pandas as pd
import streamlit as st
import streamlit.components.v1 as components

st.title('Bar Chart Race', anchor=None)
uploaded_file = st.file_uploader("", type="csv")

if uploaded_file is not None:
    df = pd.read_csv(uploaded_file,sep=',', encoding='gbk')
    df = df.set_index("date")
    st.write(df.head(6))
    bcr_html = bcr.bar_chart_race(df=df, n_bars=10)
    components.html(bcr_html.data, width=800, height=600)


```

最终效果大家亲自体验吧：

https://bar-chart-race-app.herokuapp.com/

三连在看，年入百万。下期开讲白嫖服务器 + 部署，敬请期待。