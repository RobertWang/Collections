> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/DJNMb2BI17kHaLj0Cjxnxw)

这不，客户在笔者开发完一个基于浏览器的 Web 应用程序之后说：程序需要在内（无）部（网）环境中运行……

这就意味着无法安装 Python 环境！

谁叫咱是程序员呢，不就开发一个 GUI 版本吗，难不倒我……

可是听到给的时间后，就不淡定了……

为了不影响客户的评测，只能给出一周时间！

构思
--

GUI 虽然也不难，不过需要梳理一遍服务以及与用户的交互接口，弄不好就得为 GUI 单独编写接口，这点时间显然不够呀。

不行，就再想想办法……

神器出场
----

建立虚拟环境 [3] 或者在原来的 Web 项目环境中，执行：

```
pip install pywebview


```

> 在 Windows 系统中，需要 .Net 4.0 以上

小试牛刀：

```
import webview

window = webview.create_window('Hello!', 'http://http://www.justdopython.com')
webview.start()


```

*   引用 webview 库
    
*   启动一个窗口，设置标题为 Hello!，指定页面地址
    
*   启动 webview
    

小试牛刀

**简单模式** 就相当于一个定制流浏览器，指定一个地址，就可以实现浏览了，如上面的例子。

**服务器模式** 相当于包装了一个 Web 应用，就是会启动一个本地服务器，在定制的浏览器中浏览。

**线程模式** 比较高级，就是需要自己手动维护线程状态，实现更高级的玩法。

对于现在的需求，我们选择服务器模式，即包装本地的一个 Web 应用。

对接 Flask
--------

服务器模式会为我们提供一个 HTTP Server，只要把 Web 应用部署上去就好了。

因为无非展示实际项目的代码，这里写一个简单的 Flask 应用：

> 关于 Flask Web 应用开发，可以参考笔者之前写的 [Flask 文章](https://mp.weixin.qq.com/s?__biz=MzkxNDI3NjcwMw==&mid=2247493321&idx=1&sn=3a57327ad6de499e27b17a20c4ee6159&scene=21#wechat_redirect)

创建一个 app.py 文件：

```
from flask import Flask, render_template, jsonify, request

app = Flask(__name__) # 创建一个应用

@app.route('/') 
def index():    # 定义根目录处理器
    return render_template('index.html')

@app.route('/detail')
def detail():
    return render_template('detail.html') 

if __name__ == '__main__':
    app.run() # 启动服务


```

这个应用很简单，只有两个页面，分别通过 `/` 和 `/detail` 来访问。

如果运营这段代码，就会启动一个 Flask 应用，通过 http://120.0.0.1:5000 来访问。

如何套在 Pywebview 中呢？

很简单：

```
import webview
from app import app

if __name__ == '__main__':
    window = webview.create_window('Pywebview', app, height=600, width=1000)
    webview.start()


```

*   引入 `webview`
    
*   引入 刚才创建的 app
    
*   创建一个 webview window，并将 app 作为 url 参数传入
    
*   然后启动 webview 就可以了
    

这里的关键是，将 Flask 应用作为 url 参数，Webview 发现传入的参数是 flask 应用，就会启动服务模式。

运行程序后，可以看到和在浏览器中的效果一样的：

![](https://mmbiz.qpic.cn/mmbiz_jpg/pbRNVEA1d2z5eK9aXsBo5fic5U5x0K3EnLTXPRxHcVIzjYNnibEvZRgZwNfxxAr8vtfk0kHmmu2Hh1Mbdu8FlxBQ/640?wx_fmt=jpeg)

对接 Flask

目录问题
----

现在就可以将这个项目打包成 exe 了。

首先需要安装 pyinstaller[4]

```
pip install pyinstaller


```

然后进入程序目录执行：

```
pyinstall -F -w main.py


```

*   F 参数表示将程序打包成一个可执行文件，不加这个参数就会打包成一个文件夹夹
    
*   w 参数表示执行打包好的可执行程序时，不显示命令行窗口，这个特性只有在 Windows 系统中有
    

很快在程序目录下，就会生成一个 dist 文件夹，其中就会有个 `main.exe` 可执行文件，这就是打包好的结果。

双击运行，可以看到效果……

等等，好像并不是想象中的那样！

![](https://mmbiz.qpic.cn/mmbiz_jpg/pbRNVEA1d2z5eK9aXsBo5fic5U5x0K3En3tbI1YPhmQm5Ec1icnV71qyDknZsr0FsrsSCYicdUBCsnoWahzbjicKXQ/640?wx_fmt=jpeg)

对接 Flask

这是怎么回事呢？

根据提示来看，是因为找不到页面的模板文件。

我们在前面创建 Flask app 时，使用的是默认的模板路径，即 app.py 文件所在目录的 **templates** 目录，为啥打包之后就找不见了呢？

这是因为在 windows 中，可执行文件的运行时，会被解压到一个特定的目录下，而我们的模板文件并没有被打包进入 exe 文件中，所以导致运行时找不见模板文件。

完美呈现
----

如何解决这个问题呢？

作为不使用外部数据或文件的程序，只需要将程序本身打包就可以了，但大部分程序都需要外部数据，比如我们的 Flask 应用，就需要用到静态文件等。

那么如何将它们打包进可执行文件呢？

只需要在打包时多加一个参数就可以了：

```
pyinstaller main.py -F -w --add-data "./templates/*;templates"


```

-- add-data 参数表示添加额外的数据 -- `./templates/*` 表示需要添加当前目录的 `templates` 目录中的所有文件 -- `;`为分隔符，其后的 `templates` 表示解压是这些数据所在的目录，这个目录名必须和 创建 app 时 template_folder 参数一致 -- 如果需要用到静态文件，需要额外添加，比如 --add-data "./static/*;static"

这样就能将外部数据一起打包进来了。

打包好后，双击执行，就会发现网页得以完美呈现了。

> **注意：**
> 
> 如果使用了虚拟环境，必须在虚拟环境中单独安装 pyinstaller，而不能用其他环境中已经安装好的，这是为了包装打包是可以链接所以程序引用的模块
> 
> 因为 pyinstaller 打包时，找不到被引用的模块时并不报错，而打包好的程序可能会无法执行。

总结
--

经过一番折腾，终于在客户要求的时间之前将工作完成了，特别高兴。

回头一想，多亏用了 Python 作为主要的开发语言，因为 Python 强悍的社区支持没有找不到的解决方法。

这次经历的另一个启示就是，遇到问题，不要着急就做，可以先想一想，是否有更好的方法，特别在使用 Python 的时候。

比心！

参考代码
----

> https://github.com/JustDoPython/python-examples/tree/master/taiyangxue/mazegame

[1]

Electron: _https://www.electronjs.org/_

[2]

Pywebview: _https://pywebview.flowrl.com/_

[3]

虚拟环境: _https://mp.weixin.qq.com/s/WflK5pOKhvPg8zrf_W5mfw_

[4]

pyinstaller: _https://pyinstaller.readthedocs.io/en/stable/_