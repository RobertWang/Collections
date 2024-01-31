> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4MzMyNjQwMw==&mid=2247486043&idx=1&sn=e867441995b3ca2ee410dffb1e13809a&chksm=eb8d28f7dcfaa1e11b0d3650fbb17091d2b95349d998cbd49a40384b065b383fc0b7453ba453&mpshare=1&scene=1&srcid=0129h7IdA5i166kWlvRLmBHf&sharer_shareinfo=b51e4a0dcf2fbfe7282fd8ab824cd348&sharer_shareinfo_first=b51e4a0dcf2fbfe7282fd8ab824cd348#rd)

在现代的互联网世界中，Web 应用程序的开发变得越来越重要和普遍。无论是开发个人博客、电子商务网站，还是构建大型企业级应用，都需要一个高效、稳定的 Web 框架来提供支持。

今天我要介绍的是 CherryPy，它是一个极简、稳定且功能强大的 Web 框架，可以帮助开发者快速构建高性能的 Web 应用程序。使用 CherryPy，你可以轻松地创建 RESTful API、静态网站、异步任务和 WebSocket 等应用。

接下来，让我们一起探索 CherryPy 带给我们的魅力吧。

### 什么是 CherryPy

CherryPy 是一个基于 Python 的开源 Web 框架，由一个专注于性能和易用性的小型核心驱动。它由 Sylvain Hellegouarch 创建，并在 2002 年首次发布。

CherryPy 的主要设计目标是提供一种简单的、快速的方法来构建 Web 应用程序。

CherryPy 提供了一个高效的 HTTP/1.1 WSGI Web 服务器，可以与任何兼容 WSGI 的 Web 服务器集成，如 Apache、Nginx 等。

它使用了一个简洁的 API 和清晰的结构，让开发者能够以最少的代码量实现各种 Web 应用程序。

*   项目地址：**https://cherrypy.dev/**
    
*   文档地址：**https://docs.cherrypy.dev/en/latest**
    

### 与其他类似库的对比

与其他类似的 Python Web 框架相比，CherryPy 有以下几个优点：

1.  **简单易用**：CherryPy 的 API 非常简单，容易上手。它使用类和方法来组织应用程序逻辑，可以快速创建路由、处理请求和响应。
    
2.  **稳定可靠**：CherryPy 经过多年的发展和实战检验，已经非常稳定和可靠。它的核心代码非常小巧，不会引入太多不必要的依赖和复杂性。
    
3.  **高性能**：CherryPy 是一个高性能的 Web 框架，它使用了异步 I/O 和协程技术，能够处理大量并发请求，提供出色的响应速度。
    
4.  **灵活可扩展**：CherryPy 支持插件机制和中间件，可以轻松扩展功能。你可以使用现有的插件或编写自己的插件来增加框架的功能。
    

### 安装

你可以使用 pip 命令来安装 CherryPy：

```
pip install cherrypy


```

安装完成后，你就可以开始使用 CherryPy 来构建 Web 应用程序了。

### 构建一个简单的 Web 应用程序

让我们从一个简单的示例开始，构建一个 Hello World 的 Web 应用程序。首先，创建一个名为`app.py`的文件，并添加以下代码：

```
import cherrypy

class HelloWorld:
    @cherrypy.expose
    def index(self):
        return "Hello, World!"

if __name__ == '__main__':
    cherrypy.quickstart(HelloWorld())


```

上述代码定义了一个名为 `HelloWorld` 的类，该类包含一个名为 `index` 的方法。通过在方法上添加 `@cherrypy.expose` 装饰器，我们将该方法暴露为一个 Web 页面。在 `index` 方法中，我们返回了一个简单的字符串 `Hello, World!`。

在 `if __name__ == '__main__'` 语句中，我们使用 `cherrypy.quickstart` 方法启动 CherryPy 服务器，并传递 `HelloWorld` 类的实例作为根应用程序。

保存文件后，运行以下命令启动 CherryPy 服务器：

```
python app.py


```

打开浏览器，访问 `http://localhost:8080`，你将看到一个简单的页面显示 `Hello, World!`。

### 处理请求和响应

CherryPy 使用类和方法来处理 Web 请求和生成响应。通过在方法上添加 `@cherrypy.expose` 装饰器，我们可以将该方法暴露为一个 URL 处理程序。

下面是一个处理 GET 和 POST 请求的示例：

```
import cherrypy

class FormHandler:
    @cherrypy.expose
    def index(self):
        return '''
            <form method="POST" action="submit">
                <input type="text" >
                <input type="submit" value="Submit">
            </form>
        '''

    @cherrypy.expose
    def submit(self, name):
        return f'Hello, {name}!'

if __name__ == '__main__':
    cherrypy.quickstart(FormHandler())


```

上述代码定义了一个名为 `FormHandler` 的类，该类包含一个 `index` 方法和一个 `submit` 方法。在 `index` 方法中，我们返回了一个简单的 HTML 表单，用于输入名字。在 `submit` 方法中，我们接收到名字参数，并返回一个包含名字的问候消息。

保存文件后，运行以下命令启动 CherryPy 服务器：

```
python app.py


```

打开浏览器，访问 `http://localhost:8080`，你将看到一个包含输入框和提交按钮的表单。在输入框中输入你的名字并点击提交按钮，页面将显示一个个性化的问候消息。

### 静态文件和模板

CherryPy 支持处理静态文件和使用模板引擎渲染动态内容。通过配置静态文件目录和使用模板引擎，我们可以更好地组织和管理 Web 应用程序。

首先，创建一个名为 `static` 的文件夹，并将静态文件放入其中。

然后，安装一个模板引擎，比如 Jinja2[1]。你可以使用 pip 命令来安装：

```
pip install jinja2


```

下面是一个处理静态文件和使用 Jinja2 模板引擎的示例：

```
import cherrypy
from jinja2 import Environment, FileSystemLoader

class WebApp:
    def __init__(self):
        self.env = Environment(loader=FileSystemLoader('templates'))

    @cherrypy.expose
    def index(self):
        template = self.env.get_template('index.html')
        return template.render()

if __name__ == '__main__':
    cherrypy.quickstart(WebApp(), '', config={
        '/static': {
            'tools.staticdir.on': True,
            'tools.staticdir.dir': 'static'
        }
    })


```

上述代码定义了一个名为 `WebApp` 的类，该类使用 Jinja2 模板引擎来渲染动态内容。在 `index` 方法中，我们加载名为 `index.html` 的模板文件，并使用模板引擎渲染。

在 CherryPy 的配置中，我们将 `/static` 路径映射到 `static` 文件夹，以便 CherryPy 能够处理静态文件。

创建一个名为 `templates/index.html` 的文件，并添加以下内容：

```
<!DOCTYPE html>
<html>
<head>
    <title>CherryPy WebApp</title>
    <link rel="stylesheet" type="text/css" href="/static/style.css">
</head>
<body>
    <h1>Welcome to CherryPy WebApp</h1>
    <p>This is a static file and template example.</p>
</body>
</html>


```

在 `templates` 文件夹中创建一个名为 `style.css` 的文件，并添加以下内容：

```
h1 {
    color: blue;
}


```

保存文件后，运行以下命令启动 CherryPy 服务器：

```
python app.py


```

打开浏览器，访问`http://localhost:8080`，你将看到一个包含静态文件和渲染模板的页面。标题为蓝色，页面显示一段文字。

### 实践

#### 练习 1：创建一个 RESTful API

使用 CherryPy 创建一个简单的 RESTful API，包含以下几个请求：

*   `GET /api/books`：获取所有图书
    
*   `POST /api/books`：创建一本新书
    
*   `GET /api/books/{id}`：获取指定 ID 的图书
    
*   `PUT /api/books/{id}`：更新指定 ID 的图书
    
*   `DELETE /api/books/{id}`：删除指定 ID 的图书
    

提示：可以使用 Python 的字典来保存图书数据，并使用 CherryPy 的`cherrypy.request.json`属性来获取 JSON 请求体中的数据。

#### 练习 2：使用中间件增加功能

通过编写一个自定义中间件，实现请求计时功能。中间件应该记录每个请求的处理时间，并在响应中添加一个`X-Processing-Time`标头，指示请求的处理时间。

### 总结

CherryPy 是一个极简、稳定且功能强大的 Web 框架，它通过简洁的 API 和清晰的结构，提供了一种简单快速的方法来构建 Web 应用程序。

使用 CherryPy，你可以轻松地创建高性能的 Web 应用程序，处理请求和生成响应。

在本教程中，我们简要介绍了 CherryPy 的基本用法，包括构建简单的 Web 应用程序、处理请求和响应、处理静态文件和使用模板引擎等。同时，我们也提供了一些实践题目供你尝试，以进一步探索 CherryPy 的功能和灵活性。

希望这个教程能帮助你了解和学习 CherryPy，多一个 Web 开发工具，为工作提供帮助和灵感。

参考资料

[1]

Jinja2: _https://jinja.palletsprojects.com/_