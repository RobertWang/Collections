> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/375479699)

来自：掘金，作者：ConnorZhang 链接：[https://juejin.cn/post/6944598601674784775](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6944598601674784775)

如果说要用 Python 进行 web 开发，我想你一定会告诉我 使用 Flask 或者 Django 再或者 tornado， 用来用去无非就这三种框架。可能逛 github 多的朋友还会说一个 fastapi。但是，皇上，时代变了，大清… 亡了！！！  

**速度为先**
--------

当下，Python 都已经更新到了 Python3.9.3 了，如果你还没有使用过 asyncio、和 Python3.5 新增的 async/await 语法，那说明你可能真的是桃花源人，问今是何世，不知有汉，无论魏晋了。

在当下，基于 async/await 语法的异步 Web 框架也有很多，在 github 上找一找比比皆是是，那究竟应该选哪一款呢？在 github 上有一个专门测试各种语言各种 Web 框架速度的项目，我们来看一看简单的数据：

这是所有的 Python Web 框架速度测试，有人可能会问为什么不是从 1 开始排序的，因为这个项目的测试还包含 golang、java、php 等众多语言的 Web 框架，共有 226 款。这里我们只用 Python 来做对比。

可以明显的看到，flask、django、tornado 等老牌的 Python Web 框架已经快要垫底了。

wow， 这个速度绝了。可能你们还在怀疑这个速度如何测试的，给你们看一下测试源码：

```
# Disable all logging features
import logging

logging.disable()


from flask import Flask
from meinheld import patch

patch.patch_all()

app = Flask(__name__)


@app.route("/")
def index():
    return ""


@app.route("/user/<int:id>", methods=["GET"])
def user_info(id):
    return str(id)


@app.route("/user", methods=["POST"])
def user():
    return ""
复制代码
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt


def index(request):
    return HttpResponse(status=200)


def get_user(request, id):
    return HttpResponse(id)


@csrf_exempt
def create_user(request):
    return HttpResponse(status=200)
复制代码
# Disable all logging features
import logging

logging.disable()


import tornado.httpserver
import tornado.ioloop
import tornado.web


class MainHandler(tornado.web.RequestHandler):
    def get(self):
        pass


class UserHandler(tornado.web.RequestHandler):
    def post(self):
        pass


class UserInfoHandler(tornado.web.RequestHandler):
    def get(self, id):
        self.write(id)


app = tornado.web.Application(
    handlers=[
        (r"/", MainHandler),
        (r"/user", UserHandler),
        (r"/user/(\d+)", UserInfoHandler),
    ]
)
复制代码
# Disable all logging features
import logging

logging.disable()

import multiprocessing

from sanic import Sanic
from sanic.response import text


app = Sanic("benchmark")


@app.route("/")
async def index(request):
    return text("")


@app.route("/user/<id:int>", methods=["GET"])
async def user_info(request, id):
    return text(str(id))


@app.route("/user", methods=["POST"])
async def user(request):
    return text("")


if __name__ == "__main__":
    workers = multiprocessing.cpu_count()
    app.run(host="0.0.0.0", port=3000, workers=workers, debug=False, access_log=False)
复制代码
```

就是简单的不做任何操作，只返回响应，虽然这样测试没有任何实际意义，在正常的生产环境中不可能什么都不做，但是如果所有的框架都如此测试，也算是从一定程度上在同一起跑线上了吧。

OK，就这么多，说到这里你也应该知道我想要说的这个异步框架是谁了，没错，我们今天的主角就是 **Sanic**。

![](https://pic3.zhimg.com/v2-a61f3b95e714701975179d43f06bf28e_b.jpg)

**为什么要用异步 Web 框架？**
-------------------

这可能是众多小伙伴最先想到的问题了吧？我用 Django、Flask 用的好好的，能够完成正常的任务，为什么还要用异步 Web 框架呢？

![](https://pic4.zhimg.com/v2-a4f3b20d48cca42ceabf62857c6a5bdb_b.jpg)

说到这里，首先我要反问你你一个问题，你认为在 Web 开发过程中我们最大的敌人是谁？思考 5 秒钟，然后看看我的回答：

在 Web 开发的过程中，我们最大的敌人不是用户，而是阻塞！

是的，而异步可以有效的解决 网络 I/O 阻塞，文件 I/O 阻塞。具体的阻塞相关的文章推荐查看深入理解 Python 异步编程。由于异步可以提升效率，所以对于 Python 来说，异步是最好的提升性能的方式之一。这也是为什么要选择 异步 Web 框架的原因。

**生态环境**
--------

可能有的小伙伴还是会说，你为什么不推荐 falcon 而是推荐 Sanic ？明明它的速度非常快，要比 Sanic 快了那么多，那您看一下下面的代码：

```
from wsgiref.simple_server import make_server
import falcon


class ThingsResource:
    def on_get(self, req, resp):
        """Handles GET requests"""
        resp.status = falcon.HTTP_200  # This is the default status
        resp.content_type = falcon.MEDIA_TEXT  # Default is JSON, so override
        resp.text = ('\nTwo things awe me most, the starry sky '
                     'above me and the moral law within me.\n'
                     '\n'
                     '    ~ Immanuel Kant\n\n')

app = falcon.App()

things = ThingsResource()

app.add_route('/things', things)

if __name__ == '__main__':
    with make_server('', 8000, app) as httpd:
        print('Serving on port 8000...')

        httpd.serve_forever()
```

一个状态码都要自己定义和填写的框架，我想它的速度快是值得肯定的，但是对于开发者来说，又有多少的实用价值呢？所以我们选择框架并不是要选最快的，而是要又快又好用的。

而大多数框架并不具备这样的生态环境，这应该也是为什么大多数 Python 的 Web 开发者愿意选择 Django 、 Flask 、 tornado 的原因。就是因为它们的生态相对于其他框架要丰富太多太多。

可是，如今不一样了。Sanic 框架， 从 2016 年 5 月开始 发布了第一版异步 Web 框架雏形，至今已经走过了 5 个年头，这 5 年，经过不断地技术积累，Sanic 已经由一个步履蹒跚的小框架变成了一个健步如飞的稳重框架。

在 awesome-sanic 项目中，记录了大量的第三方库，你可以找到任何常用的工具：从 API 到 Authentication，从 Development 到 Frontend，从 Monitoring 到 ORM，从 Caching 到 Queue… 只有你想不到的，没有它没有的第三方拓展。

**生产环境**
--------

以前我在国内的社区中看到过一些小伙伴在问 2020 年了，Sanic 可以用于生产环境了吗？

答案是肯定的，笔者以亲身经历来作证，从 19 年底，我们就已经将 Sanic 用于生产环境了。彼时的 Sanic 还是 19.9，笔者经历了 Sanic 19.9 -- 21.3 所有的 Sanic 的版本，眼看着 Sanic 的生态环境变得越来越棒。

还有一个问题可能你们不知道，Sanic 在创建之初目标就是创建一个可以用于生产环境的 Web 框架。可能有些框架会明确的说明框架中自带的 Run 方法仅用于测试环境，不要使用自带的 Run 方法用于部署环境。但是 Sanic 所创建的不止是一个用于测试环境的应用，更是可以直接用在生产环境中的应用。省去了使用 unicorn 等部署的烦恼！

**文档完善**
--------

想必大多数 Python 的 Web 开发者 学的第一个框架就是 Flask 或者 Django 吧，尤其是 Django 的文档，我想大多数小伙伴看了都会心塞。因为旧的版本有中文，但是新的版本，尤其是新特性，完全没有任何中文文档了！！！！这对于关注 Django 发展但英文又不是强项的同学来说，简直苦不堪言。

但 Sanic 拥有完善的 中文用户指南 和 API 文档，这些都是由贡献者自主发起的，官方承认的文档，由翻译者进行翻译贡献，由 Sanic 官方团队进行发布。或许有的小伙伴会说 Flask 也有完善的中文文档，但是那是在不同的站点上的，Sanic 的所有文档都有 Sanic 官方进行发布支持。且目前 Sanic 还在持续支持 韩语、葡萄牙语等更多的语种。

![](https://pic3.zhimg.com/v2-0d0ad4aabad2895ddd815ca6a0b3d106_b.jpg)

**社区指导**
--------

和其他框架不同，您或许能够在百度上找到论坛、频道等，但这些都是经过本地汉化的，运营者往往并不是官方，且其中夹杂了很多广告。很显然，如果是官方运营的不可能允许这种情况出现。

Sanic 不同于其他的社区，所有的论坛、频道完全由官方运营，在这里，你可以向核心开发者提问问题，Sanic 的官方发布经理也非常乐意回答各种问题。你也可以和志同道合的使用者分享自己的使用经验。这是一个完全开放的环境….

Sanic 目前常用的有 论坛、Discord、github issues、twitter、Stackoverflow

![](https://pic2.zhimg.com/v2-4aa5c8b2c98590371b47469ee663c6ad_b.jpg)

大家可以在以上的方式中关注 Sanic 的发展以及寻求社区帮助。

![](https://pic4.zhimg.com/v2-dcdc1a0ee85c3e02ded550da8c3fd71b_b.jpg)

你还在等什么？还不赶紧去试一下？最后，以 Sanic 的愿景结尾：Build Faster， Run Faster !