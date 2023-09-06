> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wa3Nk7ygd4D2ZQb2WI5bgA)

> Streamlit

本文主要分享一个 Python 的开源库：Streamlit，Streamlit 是一个 web 程序框架，我们可以不用学习前后端，不用去布置 Django 就可以更高效、更灵活的方式可视化数据并进行结果分析，可以帮助数据科学家和学者在短时间内开发机器学习 (ML) 可视化仪表板。只需几行代码，我们就可以构建并部署强大的数据应用程序，下面开始正文~

**1.1 Streamlit 的安装和简单使用，和配置**
------------------------------

环境前提：python3.6+

```
pip install streamlit

```

创建一个 python 文件 (streamlit_hello.py) 然后导入 Streamlit 模块

```
import streamlit as st
st.set_page_config(page_title='测试一下',layout='wide')
st.write('hello')

```

然后终端运行命令：

```
streamlit run streamlit_hello.py

```

第一次运行，会让你输入邮箱进行绑定，之后就不用了，

命令行出现

![](https://mmbiz.qpic.cn/sz_mmbiz_png/jz9F7HHiajcuIKuWUdGshASEpd7db5iboV88DiaAGvA3VDNicu6AAyJTv4BXiaUAhh0NUl6fIrln7pibaAR4A9liar2WQ/640?wx_fmt=png)

就会自动打开浏览器和网页，如果连接是蓝色的也可以自己点击，网页如下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/jz9F7HHiajcuIKuWUdGshASEpd7db5iboVbW2nc2e6eOD76gtuNovCk20vWy2bnxYxBR8NlFjlEZqJiaqplss312Q/640?wx_fmt=png)

下面简单来个实战，将我的爬虫代码，通过网页实现

需求：网页包含简介、pubmed 文献爬取，等功能通过边栏选择分别进入不同页面

准备：python 爬虫代码、streamlit 相关库

首先：安装 streamlit 的菜单组件

```
pip install streamlit_option_menu

```

然后开始编辑网页

```
from PubmedArticle_s import article_spider as sp
import streamlit as st
from streamlit_option_menu import  option_menu
import os
import time
st.set_page_config(page_title='阿尘文献爬虫网站',layout='wide')
with st.sidebar:
    choose = option_menu('阿尘的网站',['网站介绍','PubMed文献爬取','其他网站爬取'],
                         icons=['house','book-half','book-half'])

```

incon 可以访问：https://icons.bootcss.com/

然后根据不同导航定义页面, 首先定义介绍的页面

```
if choose == '网站介绍':
    st.title('欢迎来到阿尘的爬虫网站')
    st.write('这个网站用于爬取PubMed等相关医学文献，请严格按照说明使用，如果有问题请联系阿尘')
    st.write('本网站版权归阿尘微信公众号所有@阿尘blog')

```

在定义爬虫网页（仅展示部分）

```
elif choose == 'PubMed文献爬取':
    term1 =st.text_input('请输入查询关键词：',)
    sp = sp()
    if len(term1) !=0 :
        term = sp.input_term(term1)
    year = st.radio(
        '请勾选筛选年份，不勾选默认最近5年',
        ('空','1 year','5 year','10 year'),horizontal=True
    )
    if year == '1 year' and len(term1) != 0:
        year = '1'
        term = sp.input_term(term1)
        results1 = sp.send_request(term,year)[0]
        soup = sp.send_request(term,year)[1]
        total_page =int(sp.page_total(soup))
        st.write(f'查询结果一共有{total_page}页')
        if total_page > 1:
            want_page = int(st.text_input('请输入你想爬取的页数+1的数字：'))
            time.sleep(5)
            cost = want_page / 5
            st.write(f'本次爬取预计耗时{cost}分钟')
            st.write('文献爬取中请耐心等待...')
            if total_page >= want_page - 1:
                results2 = sp.next_page(term,want_page,year)
                results = results1 + results2
                results_list = sp.get_data(results)
                st.write('正在写入文件...')
                sp.write_data(results_list)
                st.write('爬取成功，点击按钮打开文件')
                if st.button('打开文件'):
                    os.system(r'D:/PythonProject/StreamlitArticle/article.csv')
            else:
                st.write('输入页码过大，请重新输入')
        else:
            results = results1
            results_list =sp.get_data(results)
            st.write('正在写入文件...')
            sp.write_data(results_list)
            st.write('爬取成功，点击按钮打开文件')
            if st.button('打开文件'):
                os.system(r'D:/PythonProject/StreamlitArticle/article.csv')

```

```
接下来，查看实际效果：

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/jz9F7HHiajcuIKuWUdGshASEpd7db5iboVViccrcH5nd9wk1zQticZNNfRwzeTic1zVuIXWDjpEgcX65aKY9UwRmuGA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/jz9F7HHiajcuIKuWUdGshASEpd7db5iboVBPBQGhvB3KriaiabH5eMibLRYRGp87PXkKwibT5zZjFS90OtYict9Lp8HZw/640?wx_fmt=png)

OK，是不是简单又很酷炫？其实这里才用了一点皮毛，官网有很多 api，可以显示数据分析图、等等功能，包含这些字体，输入框限制等等都可以进行详细定制，一个感觉，简单强大，上手容易，时间有限，就分享到这里了，感兴趣可以去研究研究官方文档。

官方文档地址：

中文：http://cw.hubwiz.com/card/c/streamlit-manual/

英文：https://docs.streamlit.io/