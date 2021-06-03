> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/27be62fffc00)

**来源：公众号【很酷的程序员 / RealCoolEngineer】**

> 自动登录网站后，可以保存 cookie，在有效期内可以直接加载 cookie 保持登录状态，而无需使用登录 api 重新进行登录。

本文主要介绍 cookie 的保存和加载，是上一篇文章：[Python 爬虫：自动登录下载实践](https://www.jianshu.com/p/b4f58217160d) 的扩展，后续示例代码基于这篇文章已有内容修改。

一 HTTP 客户端 Cookie 处理库
---------------------

Python3 中 Cookie 处理使用的库为`http.cookiejar`，处理 Cookie 相关的常用的几个类如下：

![](http://upload-images.jianshu.io/upload_images/3829088-592445015c1a8faf.png)

其中`MozillaCookieJar`和`LWPCookieJar`都是对`FileCookieJar`的封装，提供对不同标准的兼容而已，按需使用即可。

二 使用 Cookie 保持登录状态
------------------

使用 Cookie 保持登录实现比较简单：

1.  在初次登录成功后，将 Cookie 保存下来
2.  再次登录时，直接加载 Cookie，如果 Cookie 没有过期，直接返回即可；否则重新登录

### 1 实现分析

因为不同用户的 Cookie 信息不同，后续可能需要自动登录多个用户，同时抓取多个用户的信息，所以要将不同用户的 Cookie 分开保存。所以可以初步设计为：

1.  将 Cookie 统一保存在系统用户目录下：**`~/.polar_spider/`**；
2.  将用户的**账号邮箱或者用户 ID** 作为次级目录保存每个用户各自的信息；
3.  将缓存保存为文件**`.cookie`**。

对于`requests.sessions.Session`类，Cookie 保存在`cookies`属性中，源码如下：

```
class Session(SessionRedirectMixin):
    ...
    def __init__(self):
        ...
        #: A CookieJar containing all currently outstanding cookies set on this
        #: session. By default it is a
        #: :class:`RequestsCookieJar <requests.cookies.RequestsCookieJar>`, but
        #: may be any other ``cookielib.CookieJar`` compatible object.
        self.cookies = cookiejar_from_dict({})


```

所以可以创建一个`CookieJar`对象覆盖默认值，`Session`对象会自己解析登录请求响应中的 Cookie 信息。

> 具体怎么从 HTTP 响应中提取 Cookie 这里暂不展开，感兴趣可以查看`requests`的源码。

### 2 实现

基于前面的分析设计，重构登录的逻辑，Cookie 处理这里使用`LWPCookieJar`，核心代码:

```
# 创建CookieJar对象
cookies = LWPCookieJar(filename=cookie_file)
# 保存Cookie
cookies.save()
# 加载Cookie
cookies.load()


```

修改后的登录逻辑如下：

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author: Farmer Li，公众号：很酷的程序员(RealCoolEngineer)
# @Date: 2021-04-06

from http.cookiejar import LWPCookieJar
import json
from pathlib import Path

import click
from requests.sessions import Session

USER_AGENT = 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36'

DEAFULT_DOWNLOAD_DIR = Path.home() / 'Downloads/polar/'
DEFAULT_CACHE_DIR = Path.home() / '.polar_spider/'


class PolarSpider:
    def __init__(self, cache_dir: Path = DEFAULT_CACHE_DIR) -> None:
        self._session = Session()
        self._cache_dir: Path = cache_dir
        if not self._cache_dir.exists():
            self._cache_dir.mkdir(parents=True)

    def login(self, email, password):
        print("Login Polar Flow")
        url = "https://flow.polar.com/login"
        header = {
            'Referer': 'https://flow.polar.com/',
            'Rser-Agent': USER_AGENT
        }
        postData = {
            'returnUrl': '/',
            "email": email,
            "password": password,
        }

        cookie_file: Path = self._cache_dir / email / '.cookie'
        if not cookie_file.parent.exists():
            cookie_file.parent.mkdir(parents=True)
        self._session.cookies = LWPCookieJar(filename=cookie_file)
        if cookie_file.exists():
            self._session.cookies.load()
            # 判断缓存是否过期
            expired = False
            for cookie in self._session.cookies:
                expired = cookie.is_expired()
                if expired:
                    break
            if not expired:
                print('Cookie is not expired, not need to login again')
                return True

            # 或者检查登录后才能访问的API是否可以正常访问，如果可以则表示Cookie没有过期
            # if self.get_user_id() > 0:
            #     return True

        res = self._session.post(url, data=postData, headers=header)
        if res.status_code == 200:
            print('Login succeed')
            # 登录成功保存Cookie
            self._session.cookies.save()
            return True
        else:
            print(f"statusCode = {res.status_code}")
            return False


```

注意这里判断 Cookie 是否过期的方式，对于不同的网站，Cookie 的具体形式可能有所不同，所这个方式可能并不适用于所有的网站，需要根据具体情况分析。

> 在上一篇文章中，创建`Session`对象使用的是`requests.session()`，但是因为这个函数是已经弃用了的，所以本篇文章修正为直接使用`requests.sessions.Session`进行实例化。

以上便是 Cookie 保存和加载的一个示例，对于 Cookie 的操作还有一系列的 api 支持，在使用的过程中按实际需求查阅官方文档即可。

其他相关细节，可以阅读上一篇文章：  
[Python 爬虫：自动登录下载实践](https://www.jianshu.com/p/b4f58217160d)