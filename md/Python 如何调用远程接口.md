> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzU0MzU0Nw==&mid=2651398413&idx=4&sn=c5ca9533627a539759aa70fa821c8e38&chksm=bd25ea198a52630f4df7de83ed9c2bfbd7184b00cbb7bca66dba8ee6685997e2aab13203cc3e&mpshare=1&scene=1&srcid=0713EVpFNI1o8lhBLnsd31ym&sharer_sharetime=1626142047480&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

在 python 中我们可以使用 requests 模块来实现调用远程接口

### 一、安装 requests 模块

```
pip install requests


```

### 二、使用 requests 模块实现 get 方式调用远程接口

使用 get 方式调用远程接口主要使用了 requests 模块的 get 方法

```
requests.get()


```

get 方法常见的参数有 url,params 和 headers

*   url：表示远程接口的地址
    
*   params 表示 get 参数
    
*   headers 表示 get 传参的 headers 参数信息
    

使用 requests 模块实现 get 方式调用远程接口的简单实现如下

```
# -*- coding: utf-8 -*-
import requests
import ast
#接口地址
url = 'XXX'
#get传参
data = {'type':'0'}
#headers信息
headers = {
  'Content-Type': 'application/x-www-form-urlencoded',
  'Authorization': 'Bearer XXX'
}
#
r = requests.get(url, params=data, headers = headers)
# 接口返回的状态码
print(r.status_code)
# 接口返回的字符串内容
content = r.text
# #将字符串转字典型
content_list = ast.literal_eval(content)
print(content_list)
# 接口返回的json格式内容
print(r.json())


```

根据如上就可以实现使用 get 方式调用远程接口

### 三、使用 requests 模块实现 post 方式调用远程接口

使用 post 方式调用远程接口主要使用了 requests 模块的 post 方法

```
requests.post()


```

post 方法常见的参数有 url,data 和 headers

*   url：表示远程接口的地址
    
*   data：表示 post 参数
    
*   headers：表示 post 传参的 headers 参数信息
    

使用 requests 模块实现 post 方式调用远程接口的简单实现如下

```
# -*- coding: utf-8 -*-
import requests
import ast
#接口地址
url = 'XXX'
#header信息
headers = {
  'Content-Type': 'application/x-www-form-urlencoded',
  'Authorization': 'Bearer XXX'
}
#post传参
data = {
  'nickname': '111',
  'gender': 1,
  'city': 'ce',
  'avatar': '111'
}
r = requests.post(url, data=data,headers=headers)
# 接口返回的状态码
print(r.status_code)
# 接口返回的字符串内容
content = r.text
# #将字符串转字典型
content_list = ast.literal_eval(content)
print(content_list)
# 接口返回的json格式内容
print(r.json())


```

入门 Python 的最强三件套《ThinkPython》、《简明 Python 教程》、《Python 进阶》的 PDF 电子版已打包提供给大家，**关注下方公众号，在后台回复关键字**「**P3**」即可获取。