> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/c-BYbUr1aQt8oYveCF2uYQ)

编程是一项充满挑战和创造力的工作，而 Python 作为一种简洁、高效的编程语言，为开发者提供了许多强大的工具和库。笔者在这里将推荐 10 个必备的 Python 库，无论你是初学者还是经验丰富的开发者，这些库都将成为你编程必备的利器！

数据采集
----

### TuShare

TuShare 是一个基于 Python 的金融数据接口包，提供了丰富的金融数据获取、处理和分析功能。它可以方便地获取股票、指数、基金等金融数据，并提供了各种数据处理和分析的方法。 使用 TuShare，你可以轻松地获取股票的历史行情数据、财务指标数据、交易数据等。它还提供了一些常用的技术指标计算方法，如移动平均线、MACD 等，帮助你进行技术分析。

以下是一个使用 TuShare 获取股票历史行情数据的示例代码：

```
import tushare as ts

# 设置TuShare的token，需要在https://tushare.pro注册获取
ts.set_token('your_token')

# 初始化pro API
pro = ts.pro_api()

# 获取股票历史行情数据
df = pro.daily(ts_code='000001.SZ', start_date='2022-01-01', end_date='2022-12-31')

# 打印数据
print(df)


```

### GoPUP

https://github.com/justinzm/gopup

GoPUP 项目所采集的数据皆来自公开的数据源，数据接口包括百度、谷歌、头条、微博指数, 宏观数据，利率数据，货币汇率，千里马、独角兽公司，新闻联播文字稿，影视票房数据，高校名单，疫情数据等等，不涉及任何个人隐私数据和非公开数据。同样地，部分接口需要注册 TOKEN 才能使用。

```
import gopup as gp
df_index = gp.weibo_index(word="疫情", time_type="3month")
print(df_index)


```

爬虫
--

### playwright-python

https://github.com/microsoft/playwright-python

playwright-python 是一个 Python 的自动化测试工具，用于模拟用户在浏览器中的交互行为。它基于 Playwright 项目，提供了一组 API 和工具，可用于编写可靠和可维护的浏览器自动化测试脚本。

以下是一个简单的示例代码，演示了如何使用 playwright-python 来打开浏览器、导航到网页并获取页面标题：

```
from playwright import sync_playwright

# 初始化playwright
with sync_playwright() as playwright:
    # 在Chrome浏览器中创建一个浏览器实例
    browser = playwright.chromium.launch()

    # 在浏览器中创建一个新的页面
    page = browser.new_page()

    # 导航到指定的网页
    page.goto('https://example.com')

    # 获取页面标题
    title = page.title()

    # 打印页面标题
    print('Page Title:', title)

    # 关闭浏览器
    browser.close()


```

### awesome-python-login-model

https://github.com/Kr1s77/awesome-python-login-model

该项目收集了一些各大网站登陆方式和一些网站的爬虫程序，用于研究和分享各大网站的模拟登陆方式和爬虫程序。

### ProxyPool

https://github.com/Python3WebSpider/ProxyPool

ProxyPool 是基于 Python 的代理池项目，用于获取、存储和检验代理 IP，以供其他爬虫程序使用。它提供了一个简单的 API 接口，让你可以方便地获取可用的代理 IP，从而提高爬虫的效率和稳定性。

以下是一个简单的示例代码，演示了如何使用 ProxyPool 项目获取代理 IP：

```
import requests

# 发起请求的URL
url = 'http://httpbin.org/ip'

# ProxyPool的API接口
api_url = 'http://localhost:5555/random'

# 设置代理
proxy = requests.get(api_url).text.strip()

proxies = {
    'http': f'http://{proxy}',
    'https': f'https://{proxy}'
}

# 使用代理发起请求
response = requests.get(url, proxies=proxies)

# 打印响应内容
print(response.text)


```

Web 相关
------

### streamlit

Streamlit 是一个用于构建数据科学和机器学习应用的开源 Python 库。它简化了应用程序的创建过程，使开发者可以快速构建交互式的 Web 应用程序，无需编写大量的前端代码。

以下是一个简单的示例代码，展示了如何使用 Streamlit 创建一个简单的 Web 应用程序：

```
import streamlit as st
# 标题
st.title('欢迎使用Streamlit')
# 文本输入框
name = st.text_input('请输入您的姓名')
# 按钮
button = st.button('点击打招呼')
# 打招呼逻辑
if button:
    st.write(f'你好，{name}！欢迎使用Streamlit。')
# 数据展示
data = [1, 2, 3, 4, 5]
st.write('数据展示：', data)


```

### PyWebIO

PyWebIO 是一个基于 Python 的交互式 Web 应用框架，它提供了简单而强大的工具来构建具有交互性的 Web 应用程序。使用 PyWebIO，你可以在浏览器中构建并展示用户界面，无需编写 HTML、CSS 或 JavaScript 代码。

以下是一个简单的示例代码，演示了如何使用 PyWebIO 创建一个交互式的 Web 应用程序：

```
from pywebio import start_server
from pywebio.input import input, TEXT
from pywebio.output import put_text

def greet_user():
    name = input("请输入你的名字：", type=TEXT)
    put_text(f"你好，{name}！欢迎使用PyWebIO。")

if __name__ == '__main__':
    start_server(greet_user, port=8080)


```

### wagtail

https://github.com/wagtail/wagtail

Wagtail 是一个基于 Django 的开源内容管理系统（CMS），它提供了一种简单、灵活和现代的方式来构建和管理网站内容。Wagtail 专注于内容管理和编辑体验，同时提供了丰富的功能和扩展性。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Zvl9ickIYtdLIv4BER0iaJxkjkxDWbvaZialE3fLEW4Bhp3rBVt2t2Ouaz7spxaBI906icfqayp3zZ9AxzzC5tjgUA/640?wx_fmt=png)

python 教程
---------

### practical-python

https://github.com/hantichao/practical-python

一个人气超高的 Python 学习资源项目，是 MarkDown 格式的教程，非常友好。

### learn-python3