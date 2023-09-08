> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wvyO8laP7iokI8ZTeGjdcQ)

> 探索那些优雅的异步编程库——Python 异步库！

**欢迎来到本篇文章，今天我们要探索那些优雅的异步编程库——Python 异步库！**

**asyncio：官方异步库**

**首先，让我们认识一下 Python 的官方异步库 asyncio。它提供了一套基于协程的异步编程框架，让你可以轻松编写高效的异步代码。**

**让我们看看 asyncio 的魔法：**

```
import asyncio
# 定义异步函数
async def greet(name):
    await asyncio.sleep(1)  # 模拟耗时操作
    return f"Hello, {name}!"
# 异步调用函数
async def main():
    tasks = [greet("Alice"), greet("Bob"), greet("Charlie")]
    result = await asyncio.gather(*tasks)
    print(result)
# 运行异步代码
if __name__ == "__main__":
    asyncio.run(main())

```

**运行上述代码，你将看到异步调用的结果输出在终端上，asyncio 让异步编程变得如此简单！**

**Trio：友好的异步库**

**接下来，我们要介绍 Trio 这个友好的异步库。它提供了清晰简洁的 API，让你可以更加直观地编写异步代码。**

**Trio 的安装非常简单，使用以下命令：**

```
pip install trio

```

**让我们看看 Trio 的魔法：**

```
import trio
# 定义异步函数
async def greet(name):
    await trio.sleep(1)  # 模拟耗时操作
    return f"Hello, {name}!"
# 异步调用函数
async def main():
    async with trio.open_nursery() as nursery:
        nursery.start_soon(greet, "Alice")
        nursery.start_soon(greet, "Bob")
        nursery.start_soon(greet, "Charlie")
# 运行异步代码
if __name__ == "__main__":
    trio.run(main)

```

**运行上述代码，你将看到异步调用的结果输出在终端上，Trio 让你的异步编程更加友好！**

**Aiohttp：异步网络请求**

**最后，让我们来认识 Aiohttp 这个异步网络请求库。它基于 asyncio，提供了高效的异步 HTTP 请求功能，让你的网络请求变得更加快速。**

**Aiohttp 的安装非常简单，使用以下命令：**

```
pip install aiohttp

```

**让我们看看 Aiohttp 的魔法：**

```
import aiohttp
import asyncio
# 异步发送 GET 请求
async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()
# 异步调用函数
async def main():
    url = "https://www.example.com"
    result = await fetch(url)
    print(result)
# 运行异步代码
if __name__ == "__main__":
    asyncio.run(main())

```

**运行上述代码，你将看到异步请求的结果输出在终端上，Aiohttp 让网络请求更加高效！**

**异步编程案例**

**下面我们来看一个实际应用的案例：使用 asyncio 和 aiohttp 构建一个简单的异步爬虫，抓取网页内容。**

```
import aiohttp
import asyncio
async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()
async def main():
    urls = ["https://www.example.com", "https://www.google.com", "https://www.python.org"]
    tasks = [fetch(url) for url in urls]
    results = await asyncio.gather(*tasks)
    for url, content in zip(urls, results):
        print(f"Content of {url}: {content[:100]}...")
if __name__ == "__main__":
    asyncio.run(main())

```

**运行上述代码，你可以看到多个网页内容被异步爬取，让你的异步编程更加高效！**

**— **完** —**