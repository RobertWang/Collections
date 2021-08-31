> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/35439412)

**这是一篇零基础环境搭建教程，适合没有接触过 PC 应用编写的前端开发。写这篇文章有两个原因，一个前端众热衷于用 Js 进行移动端开发，PC 相关资料较少，话题更少。另一个是现存资料大多是大佬所作，多少会有专家盲点，无法从初学者的角度理解问题。而我恰好又是个初学者… 如教程有纰漏，望大佬指教**

Electron 介绍
-----------

> [Electron](https://link.zhihu.com/?target=https%3A//electronjs.org/) 是由 Github 开发，用 HTML，CSS 和 JavaScript 来构建跨平台桌面应用程序的一个开源库。 Electron 通过将 [Chromium](https://link.zhihu.com/?target=https%3A//www.chromium.org/Home) 和 [Node.js](https://link.zhihu.com/?target=https%3A//nodejs.org/) 合并到同一个运行时环境中，并将其打包为 Mac，Windows 和 Linux 系统下的应用来实现这一目的。

安装 Node
-------

首先下载安装 Node.js，安装时会自动安装 NPM 包管理工具

安装完毕后检测安装结果。

打开命令行工具，执行 node -v，出现以下信息表示安装成功：

再执行 npm -v，出现以下信息表示安装成功：

安装 Electron
-----------

采用 npm 安装，国内用户建议使用 cnpm，避免出现网络错误。打开命令行工具，执行下面的命令安装 cnpm：

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

安装完毕，继续在命令行工具执行下面的命令安装 Electron：

```
cnpm install electron -g
```

最后还需要安装一个打包工具，将编写的代码打包成 exe 安装文件及可执行文件：

```
cnpm install electron-builder -g
```

至此，简单的开发环境搭建完毕。

可以进行检测是否安装成功，在命令行工具执行（检测 electron）：

```
electron
```

![](https://pic1.zhimg.com/v2-7df1972f197d201cbb2ee1a389b08358_b.jpg)

命令行工具执行（检测 electron-builder）：

```
build -v
```

![](https://pic3.zhimg.com/v2-2b5b91067bd4a6d656726991679c08da_b.jpg)

小试牛刀
----

接下来我们编写一个简单的应用尝试一下。

新建一个目录，目录下需要下列三个文件，这是一个最基本的 Electron 项目结构目录：

![](https://pic2.zhimg.com/v2-8131525b8418504c7b12ba12196e5629_b.jpg)

package.json 文件内容

```
{
    "name": "your-app",
    "version": "0.1.0",
    "main": "main.js",
    "scripts": {
      "start": "electron ."
    }
  }
```

main.js 文件内容

```
const {app, BrowserWindow} = require('electron')
  const path = require('path')
  const url = require('url')
  
  // 保持一个对于 window 对象的全局引用，如果你不这样做，
  // 当 JavaScript 对象被垃圾回收， window 会被自动地关闭
  let win
  
  function createWindow () {
    // 创建浏览器窗口。
    win = new BrowserWindow({width: 800, height: 600})
  
    // 然后加载应用的 index.html。
    win.loadURL(url.format({
      pathname: path.join(__dirname, 'index.html'),
      protocol: 'file:',
      slashes: true
    }))
  
    // 打开开发者工具。
    win.webContents.openDevTools()
  
    // 当 window 被关闭，这个事件会被触发。
    win.on('closed', () => {
      // 取消引用 window 对象，如果你的应用支持多窗口的话，
      // 通常会把多个 window 对象存放在一个数组里面，
      // 与此同时，你应该删除相应的元素。
      win = null
    })
  }
  
  // Electron 会在初始化后并准备
  // 创建浏览器窗口时，调用这个函数。
  // 部分 API 在 ready 事件触发后才能使用。
  app.on('ready', createWindow)
  
  // 当全部窗口关闭时退出。
  app.on('window-all-closed', () => {
    // 在 macOS 上，除非用户用 Cmd + Q 确定地退出，
    // 否则绝大部分应用及其菜单栏会保持激活。
    if (process.platform !== 'darwin') {
      app.quit()
    }
  })
  
  app.on('activate', () => {
    // 在macOS上，当单击dock图标并且没有其他窗口打开时，
    // 通常在应用程序中重新创建一个窗口。
    if (win === null) {
      createWindow()
    }
  })
  
  // 在这文件，你可以续写应用剩下主进程代码。
  // 也可以拆分成几个文件，然后用 require 导入。
```

index.html 文件内容

```
<!DOCTYPE html>
  <html>
    <head>
      <meta charset="UTF-8">
      <title>Hello World!</title>
    </head>
    <body>
      <h1>Hello World!</h1>
      We are using node <script>document.write(process.versions.node)</script>,
      Chrome <script>document.write(process.versions.chrome)</script>,
      and Electron <script>document.write(process.versions.electron)</script>.
    </body>
  </html>
```

将以上内容保存到自己的文件。接下来打开命令行工具，进入当前目录，我的文件存放在 e 盘的 js_win 文件夹下，命令行切换至该目录。

![](https://pic1.zhimg.com/v2-adcb460d12b6454d51e38dd922e39de8_b.jpg)

然后执行命令运行程序：

```
electron .
```

就能看到自己的程序已经运行出来了，修改 html 代码，重新运行即可看到变化。

开发完毕，同样在该目录，执行以下命令打包（首次执行失败可二次尝试）：

```
build
```

该命令执行成功会自动在项目目录生成多个文件：

![](https://pic4.zhimg.com/v2-1eccb071589b4bacd45c88c841296037_b.jpg)

其中 dist 文件夹下的一个 exe 文件便是项目安装包。

![](https://pic4.zhimg.com/v2-38996e896ac5fcf591ecffdcdb8949f3_b.jpg)

至此，项目环境搭建完毕。接下来编写业务代码即可，详细可参考：

完。