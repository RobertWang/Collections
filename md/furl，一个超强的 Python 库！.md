> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg4ODUzNTcxMw==&mid=2247498587&idx=1&sn=36ab164563b621ba763417b58046ffa5&chksm=cffb1495f88c9d83a26310cf0e17c5c38b7d080f6434d38c4066158e0eb85ddf371a2f0dd224&mpshare=1&scene=1&srcid=0201uhAp74GuYtETQ7kpvk5q&sharer_shareinfo=9bb3bd93ebee60536ae41debf133abca&sharer_shareinfo_first=9bb3bd93ebee60536ae41debf133abca#rd)

更多 Python 学习内容：ipengtao.com

大家好，今天为大家分享一个超强的 Python 库 - furl。

Github 地址：https://github.com/gruns/furl

* * *

在现代 Web 应用程序和网络爬虫中，对 URL 进行操作是一个常见而关键的任务。Python Furl 是一个强大的 URL 处理库，它提供了简单而高性能的 URL 构建、解析和操作功能。本文将提供关于 Python Furl 的全面指南，包括安装和配置、基本概念、URL 解析、URL 构建、查询参数操作、片段处理、实际应用场景以及丰富的示例代码。

什么是 Python Furl？
----------------

Python Furl 是一个用于处理 URL 的 Python 库，它旨在提供高性能且易于使用的 URL 操作功能。

Furl 的主要特点包括：

*   **简单易用**：Furl 提供了简单而直观的 API，使 URL 操作变得轻松。
    
*   **高性能**：Furl 经过优化，执行速度快，适用于处理大量 URL。
    
*   **功能丰富**：Furl 支持 URL 的解析、构建、查询参数操作、片段处理等多种功能。
    
*   **不可变性**：Furl 的 URL 对象是不可变的，可以确保线程安全性。
    

安装和配置
-----

可以使用 pip 来安装 Furl：

```
pip install furl


```

安装完成后，可以在 Python 中导入 Furl 库：

```
import furl


```

URL 解析
------

Furl 可以将 URL 字符串解析为其各个组成部分，如协议、主机、路径、查询参数和片段。

以下是一个示例：

```
url = furl.furl("https://example.com/path?)

print("Scheme:", url.scheme)
print("Host:", url.host)
print("Path:", url.path)
print("Query Parameters:", url.args)
print("Fragment:", url.fragment)


```

输出结果如下：

```
Scheme: https
Host: example.com
Path: /path
Query Parameters: {'name': ['John'], 'age': ['30']}
Fragment: section1


```

可以使用 Furl 的属性来访问 URL 的不同部分，使 URL 解析变得简单而直观。

URL 构建
------

除了解析 URL 外，Furl 还可以构建 URL，将各个组成部分组合成一个完整的 URL。

以下是一个构建 URL 的示例：

```
url = furl.furl()
url.scheme = "https"
url.host = "example.com"
url.path = "/path"
url.args['name'] = "John"
url.args['age'] = 30
url.fragment = "section1"

print(url.url)


```

输出结果是：

```
https://example.com/path?name=John&age=30#section1


```

通过设置 Furl 对象的属性，可以轻松地构建复杂的 URL。

查询参数操作
------

Furl 还提供了强大的查询参数操作功能，包括添加、删除、修改和获取查询参数。

以下是一些示例：

```
url = furl.furl("https://example.com/search?q=python&lang=en")

# 添加查询参数
url.args.add("page", 2)

# 删除查询参数
url.args.remove("lang")

# 修改查询参数
url.args['q'] = "programming"

# 获取查询参数值
print("Page:", url.args.get("page"))


```

查询参数操作能够轻松地处理 URL 中的参数，无需手动解析和构建查询字符串。

片段处理
----

Furl 还支持片段处理，可以轻松地获取和设置 URL 中的片段。

以下是一些示例：

```
url = furl.furl("https://example.com/page#section1")

# 获取片段
fragment =

 url.fragment

# 设置片段
url.fragment = "section2"


```

片段通常用于在 Web 页面内部进行导航，Furl 使其操作变得简单。

实际应用场景
------

Python Furl 可以在许多实际应用场景中发挥作用。

### 1. Web 爬虫

在 Web 爬虫中，可以使用 Furl 来构建和解析 URL，以便在不同的页面之间导航、抓取数据和处理查询参数。

```
base_url = "https://example.com"
url = furl.furl(base_url)

# 构建下一页的URL
next_page = url.copy()
next_page.args['page'] = 2


```

### 2. Web 应用程序

在 Web 应用程序中，可以使用 Furl 来处理用户提交的 URL，解析其中的查询参数，进行页面路由等。

```
from flask import request

# 从请求中获取URL并解析查询参数
url = furl.furl(request.url)
search_query = url.args.get("q")


```

### 3. URL 重写和路由

在 URL 重写和路由中，可以使用 Furl 来构建和修改 URL，以实现友好的 URL 结构和路由规则。

```
from werkzeug.routing import Map, Rule
from werkzeug.test import Client

url_map = Map([
    Rule('/page/<int:page>', endpoint='page'),
    Rule('/post/<slug>', endpoint='post'),
])

# 构建URL
url = furl.furl()
url.path = url_map.build("page", values={"page": 2})


```

### 4. API 请求

在与 Web API 进行通信时，可以使用 Furl 来构建 API 请求的 URL，并处理 API 响应中的数据。

```
import requests

base_url = "https://api.example.com"
url = furl.furl(base_url)
url.path.segments.append("users")
url.args['page'] = 1

response = requests.get(url.url)
data = response.json()


```

总结
--

Python Furl 是一个高性能的 URL 处理库，用于解析、构建和操作 URL。本文提供了有关 Furl 的全面指南，包括安装和配置、基本概念、URL 解析、URL 构建、查询参数操作、片段处理以及实际应用场景。通过使用 Furl，可以轻松地处理 URL 相关的任务，从而简化 Web 开发、爬虫和 API 请求等工作。希望本文能帮助大家更好地理解 Python Furl，并开始使用它来处理 URL 操作。