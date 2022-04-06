> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTU5OTI4MA==&mid=2247505113&idx=1&sn=22598a216a0d91ae3415f04a5734a77e&chksm=fbaff586ccd87c908dbc01364af5d42f8845410f429f2d4308587569f600d6ceff621dd8bdd7&mpshare=1&scene=1&srcid=0403tyEamdHAuWwYsIvkEgSF&sharer_sharetime=1648982492232&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

> 编辑：乐乐 | 来自：SegmentFault

  

**正文**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0eF7aUpibzQyB5gciaRIaJ8hdQEKVJyG2656iaB8G9K8vEmVDeXw3U2zCT1aur2c6vdOLTOHs2rk6KVg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)  

技术编辑：宗恩丨发自 思否编辑部

* * *

最近，微软开源了一个项目叫「playwright-python」，作为一个兴起项目，出现后受到了大家热烈的欢迎，那它到底是什么样的存在呢？今天为你介绍一下这个传说中的小白神器。

Playwright 是针对 Python 语言的纯自动化工具，它可以通过单个API自动执行 Chromium，Firefox 和 WebKit 浏览器，连代码都不用写，就能实现自动化功能。

虽然测试工具 selenium 具有完备的文档，但是其学习成本让一众小白们望而却步，对比之下 playwright-python 简直是小白们的神器。

Playwright真的适用于Python吗？答案是肯定的，微软对于适用于Python的Playwright已准备就绪。可能会发生API重大更改。但大概率是这种情况不会发生，微软还表示仅在他们知道它可以改善您使用新库的体验时，才会可能这样做。不过微软也提醒尚不支持特定于供应商的API的某些极端情况，例如收集Chromium跟踪，覆盖率报告等。

* * *

### **1、Playwright介绍**

Playwright是一个强大的Python库，仅用一个API即可自动执行Chromium、Firefox、WebKit等主流浏览器自动化操作，并同时支持以无头模式、有头模式运行。

Playwright提供的自动化技术是绿色的、功能强大、可靠且快速，支持Linux、Mac以及Windows操作系统。

还有朋友这么夸：这个项目作为针对 Python 语言纯自动化的工具，解放了代码，实现了自动化功能，我们来看看怎么用它吧。

* * *

### **2、Playwright使用**

#### **安装**

Playwright的安装非常简单，两步解决。

```
安装playwright库
pip install playwright
安装浏览器驱动文件（安装过程稍微有点慢）
python -m playwright install
复制代码


```

上面两个pip操作分别安装：

*   安装Playwright依赖库，需要Python3.7+
    
*   安装Chromium、Firefox、WebKit等浏览器的驱动文件
    

#### **录制**

使用Playwright无需写一行代码，我们只需手动操作浏览器，它会录制我们的操作，然后自动生成代码脚本。

下面就是录制的命令codegen，仅仅一行。

  

```
命令行键入 --help 可看到所有选项
python -m playwright codegen
复制代码


```

codegen的用法可以使用--help查看，如果简单使用就是直接在命令后面加上url链接，如果有其他需要可以添加options。

  

```
python -m playwright codegen --help
Usage: index codegen [options] [url]


open page and generate code for user actions


Options:
  -o, --output <file name>  saves the generated script to a file
  --target <language>       language to use, one of javascript, python, python-async, csharp (default: "python")
  -h, --help                display help for command


Examples:


  $ codegen
  $ codegen --target=python
  $ -b webkit codegen https://example.com

复制代码


```

options含义：

  

*   -o：将录制的脚本保存到一个文件
    
*   --target：规定生成脚本的语言，有JS和Python两种，默认为Python
    
*   -b：指定浏览器驱动
    

  

比如，我要在baidu.com搜索，用chromium驱动，将结果保存为my.py的python文件。

  

```
python -m playwright codegen --target python -o 'my.py' -b chromium https://www.baidu.com
复制代码


```

命令行输入后会自动打开浏览器，然后可以看见在浏览器上的一举一动都会被自动翻译成代码，如下所示。

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0eF7aUpibzQyB5gciaRIaJ8hdzTGhsWb6PWr2snuYrk5HtZS0ba4pNty4OTfOXIU4ic62CicbNy7DqKDw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

结束后自动关闭浏览器，保存生成的自动化脚本到py文件。

```
from playwright import sync_playwright


def run(playwright):
browser = playwright.chromium.launch(headless=False)
context = browser.newContext()

# Open new page
page = context.newPage()


page.goto("https://www.baidu.com/")


page.click("input[]")


page.fill("input[]", "jingdong")


page.click("text="京东"")

# Click //a[normalize-space(.)='京东JD.COM官网 多快好省 只为品质生活']
with page.expect_navigation():
    with page.expect_popup() as popup_info:
        page.click("//a[normalize-space(.)='京东JD.COM官网 多快好省 只为品质生活']")
    page1 = popup_info.value
# ---------------------
context.close()
browser.close()
with sync_playwright() as playwright:
run(playwright


```

此外，playwright还提供了同步和异步的API接口，文档如下。另外，搜索公众号顶级架构师后台回复“面试”，获取一份惊喜礼包。

> 链接：https://microsoft.github.io/playwright-python/index.html

  

#### **同步**

下面示例代码：依次打开三个浏览器，前往baidu搜索，截图后退出。

  

```
from playwright import sync_playwright
with sync_playwright() as p:
for browser_type in [p.chromium, p.firefox, p.webkit]:
    browser = browser_type.launch()
    page = browser.newPage()
    page.goto('https://baidu.com/')
    page.screenshot(path=f'example-{browser_type.name}.png')
    browser.close()
    复制代码


```

#### **异步**

异步操作可结合asyncio同时进行三个浏览器操作。

  

```
import asyncio
from playwright import async_playwright
async def main():
async with async_playwright() as p:
    for browser_type in [p.chromium, p.firefox, p.webkit]:
        browser = await browser_type.launch()
        page = await browser.newPage()
        await page.goto('http://baidu.com/')
        await page.screenshot(path=f'example-{browser_type.name}.png')
        await browser.close()
        asyncio.get_event_loop().run_until_complete(main())
       复制代码


```

#### **移动端**

更厉害的是，playwright还可支持移动端的浏览器模拟。下面是官方文档提供的一段代码，模拟在给定地理位置上手机iphone 11 pro上的Safari浏览器，首先导航到maps.google.com，然后执行定位并截图。

  

```
from playwright import sync_playwright
with sync_playwright() as p:
iphone_11 = p.devices['iPhone 11 Pro']
browser = p.webkit.launch(headless=False)
context = browser.newContext(
    **iphone_11,
    locale='en-US',
    geolocation={ 'longitude': 12.492507, 'latitude': 41.889938 },
    permissions=['geolocation']
)
page = context.newPage()
page.goto('https://maps.google.com')
page.click('text="Your location"')
page.screenshot(path='colosseum-iphone.png')
browser.close()
复制代码


```

另外，还可以配合pytest插件一起使用，感兴趣可以自己试一下。

* * *

  

**3、总结**
--------

playwright相比已有的自动化测试工具有很多优势，其中有：

  

**支持****所有浏览器的**
----------------

  

---

*   在Chromium，Firefox和WebKit上进行测试。Playwright拥有适用于所有现代浏览器的完整API覆盖，包括Google Chrome和Microsoft Edge（带有Chromium），Apple Safari（带有WebKit）和Mozilla Firefox。
    
*   跨平台的WebKit测试。使用Playwright，使用适用于Windows，Linux和macOS的WebKit构建，测试您的应用程序在Apple Safari中的行为。在本地和CI上进行测试。
    
*   测试手机。使用设备仿真在移动Web浏览器中测试您的自适应Web应用程序。
    
*   无报文头与有报文头。Playwright支持所有浏览器和所有平台的无头（无浏览器UI）和有头（有浏览器UI）模式。有报文头模式适用于调试，而无报文头适用于CI / cloud执行。
    

**拥有快速可靠的执行**
-------------

*   自动等待APIs。Playwright交互会自动等待直到元素准备就绪。这样可以提高可靠性并简化测试编写流程。
    
*   无超时自动化。Playwright会接收浏览器信号，例如网络请求，页面导航和页面加载事件，以消除导致睡眠中断的烦恼。
    
*   与浏览器上下文保持并行。对于多个并行孤立的浏览器上下文可执行环境重复使用一个单独的浏览器实例。
    
*   弹性元素选择器。Playwright可以依靠面向用户的字符串（例如文本内容和可访问性标签）来选择元素。这些字符串比紧耦合到DOM结构的选择器更具弹性。
    

**拥有强大的自动化功能**
--------------

*   多个域，页面和框架。Playwright是一种进程外自动化驱动程序，不受页面内JavaScript执行范围的限制，并且可以自动执行具有多个页面的方案。
    
*   强大的网络控制。Playwright引入上下文范围的网络拦截以便进行终止或者模拟网络请求。
    
*   现代网络功能。Playwright通过插入阴的选择器，地理位置，权限，Web Worker和其他现代Web API支持Web组件。
    
*   涵盖所有场景的能力。支持文件下载和上传，进程外iframe，原生输入事件，甚至是深色模式。
    

**但它也有局限性**
-----------

*   旧版Edge和IE11支持。Playwright不支持旧版Microsoft Edge或IE11（弃用通知）。支持新的Microsoft Edge（在Chromium上）。
    
*   Java语言绑定：Playwright API目前无法在Java或Ruby中使用。这是暂时的限制，因为Playwright旨在支持任何语言的绑定。
    
*   在真实的移动设备上进行测试：Playwright使用桌面浏览器来模拟移动设备。
    

虽然有一些局限，但现在playwright 已经更新到了 1.7.0 版本，随着一代代的更新，系统也会更为完善，作为一款小白神器，为大家省了那么多事情，我们相信它的未来会越来越好。

****你还有什****么想要补充的吗？****  

> 免责声明：本文内容来源于网络，文章版权归原作者所有，意在传播相关技术知识&行业趋势，供大家学习交流，若涉及作品版权问题，请联系删除或授权事宜。  

END