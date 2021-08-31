> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.sohu.com](https://www.sohu.com/a/121934123_465979)

原标题：使用 NW.js 构建跨平台桌面应用程序

OSC 协作翻译

**编译自：**_Building a Cross-platform Desktop App with NW.js_

**链接：**_https://www.sitepoint.com/cross-platform-desktop-app-nw-js/_

**译者：**_Viyi, 边城, spache, Tony_

NW.js 是一个使用 Web 技术创建本地应用的框架，如 HTML、Java 和 CSS。简单地说，当你在使用普通的流程开发一个 Web 应用时，开发完成后，运行一个生成器，将所有东西编译成一个本地应用，它会像一个浏览器一样运行你的 Web 应用。这种应用就被称为 “Hybrid 应用（一种混合本地编程和 Web 编程技术的应用）”。

Hybrid 应用的伟大之处，不仅在于它可以使用你熟悉的语言（HTML、Java 和 CSS）来开发，还因为它比普通的 Web 应用更有优越性：

● 控制浏览器和浏览器版本（你知道你的应用是调用的什么浏览器）。NW.js hybrid 应用使用 Chromium 来显示— 这是一种开源浏览器，也是 Google Chrome（谷歌浏览器）的核心。因此，能在 Chrome 中运行的应用也能在 NW.js 中运行。

● 控制视窗。例如，你可以定义一个固定大小，或者最小化 / 最大化的视窗。

● 对本地文件的访问不会受同源策略的约束。如果你想在浏览器通过 打开一个不在相同目录的本地文件，请求会阻止。而 NW.js 应用中关闭了这样的行为。

它们也提供了 API，带来如下优点：

● 整合 Node.js

● 访问剪贴板

● 访问文件系统

● 访问硬件（比如获取打印机列表）

● 托盘图标

● 自定义文件选择对话框

● 整合 shell（在默认的资源管理器或浏览器中打开文件或 URL）

● 自定义主窗口的选项（关闭按钮、菜单栏）和上下文菜单

● 设置和获取绽放等级。

看起来不错？那让我们开始吧。在这篇文章中，我们会通过练习熟悉 NW.js，并学习如何创建一个 Hybrid 应用。用于这篇文章的示例应用已经在 GitHub 上准备好了。

**NW.js 与 Electron 相比之下的优势**

首先要说的是，NW.js 并不是唯一的 Bybrid 应用框架。另一个这样的框架叫 Electron。它诞生于 2013 年，比 NW.js 晚两年，不过因为它来自于 GitHub，很快就被大家所认识。现在你可能对它们之间的区别感兴趣。这里列举了与 Electron 相比，NW.js 的优势：

● 支持 chrome.* API。这些 API 可用于与浏览器交互。（你可以在 NW.js 文档中找到更多相关的信息)

● 支持 Chrome 应用。Chrome 应用是使用 Web 语句编写并打包的应用。（更多信息请参阅 Chrome 开发者文档。）这些应用与 NW.js 不同，因为它们没有整合 Node.js，而且通过 Chrome Web Store 发布。● （Chrominum 会在 2018 年 8 月取消对它的支持（参阅他们的博客）。不过因为这篇文章所说的原因，NW.js 仍然会支持 Chrome 应用。）

● 支持 NaCl (Native Client，本地客户端) 和 PNaCl (可移植的本地客户端) 应用。它们致力于性能，因此使用 C 和 C++ 编写。（参阅这篇教程了解如何在 NW.js 中使用它们。）

拥有 V8 的映像源码保护，保护你的应用程序源码。使用 nwjc 工具可以将你的代码编译为本地代码。（更多信息参考这篇文章。）

拥有一个内建的 PDF 阅读器。

● 允许打印预览。

● 支持 Node.js 整合 Web Workers。这用于编写多线程应用。

不过，Electron 也有值得一提的优点：

● 内建自动更新（你可以在这个事项里找到关于 NW.js 的自动更新）。

● 自动向远程服务器报告程序崩溃。NW.js 只会把错误信息写入一个本地文件，需要手工提交。

**还有一个重要的区别。**NW.js 应用的入口是一个 HTML 文件中的 Form。这个 HTML 文件会直接在 GUI 中打开。

另一方面，Electron 应用使用一个 Java 文件作为入口。这个 Java 文件由另一个主进程打开，然后由它在 GUI 中打开 HTML。这样的话，理论上你可以不通过 GUI 运行 Electron 应用。同样的道理，关闭 GUI 不会关闭主进程；你需要调用一个 API 来终止主进程。

虽然 Electron 不使用 GUI 来启动 Java 写的桌面应用，但 NW.js 应用却更容易建立示基于显示 HTML 的应用。

_注意: 如果你确实关心 Electron 的优势，看看 SitePoint 最新的文章使用 Electon 创建桌面应用。_

**创建一个演示应用**

来，开始创建我们的应用，稍后我们会把它编译成本地应用。因为创建 Web 应用的方式多种多样——使用不同的 JS 语言（Type、Coffe 等），模块加载器（RequireJS、webpack、SystemJS 等），框架（AngularJS、React、Vuew.js 等）和（样式表）预处理器（SCSS、LESS、Haml 等）——每个人都有自己的偏好，我们只使用基本的技术，HTML、CSS 和 JS（ES6 标准）。

NW.js 没有样板 (初始项目) 来完成初始的设置。它们都通过特定的框架、模块加载器或预处理器来创建。然而，我们从头开始实现一个简单的 NW.js 应用程序。它会比较易懂，之后你可以很容易按自己的需求定制，或者把它变成样板。

**项目结构**

首先，我们需要创建项的目录结构和文件：

nw.js-example/

├── src/

│ ├── app/

│ │ └── main.js

│ ├── assets/

│ │ └── icon.png

│ ├── styles/

│ │ └── common.css

│ ├── views/

│ │ └── main.html

│ └── package.json

└── package.json

说明：

● src/ 应用程序源文件。

● src/app/ Java 文件。

● src/assets/ 图片，在我们的例子中只有 icon.png 文件，它将作为一个窗口图标来显示。

● src/styles/ 通常包含 SCSS 或 LESS 文件，在我们的例子中，仅仅是一个简单的 CSS 文件。

● src/views/ 包含 HTML 视图文件。

● src/package.json NW.js 应用程序清单文件（参考清单文件格式），我们也可以在这个文件中为应用程序指定特殊的依赖项。

● package.json 是一个 npm 包文件，我们用它来构建工作流，也可以指定特殊的依赖，这在实际的 NW.js 应用程序中不是必须的，例如根据依赖构建。

**创建清单文件**

现在，我们已经创建了项目结构和文件，可以从 NW.js 的清单文件 src/package.json 开始了。根据文档，这个文件基本上仅需要两个属性：name，一个应用程序名称，和 main，一个作为入口的 HTML 文件的路径。但是我们需要添加更多信息，如窗口的图标的路径，以及最小宽度和高度，以确保用户不会看到任何奇怪的事情：

{

"name":"nw.js-example",

"main":"views/main.html",

"window":{

"min_width":400,

"min_height":400,

"icon":"assets/icon.png" }

}

就是这样！应用程序在开始运行后，将打开 src/views/main.html，main 的路径是以清单文件（manifest file）为参考的相对路径。

**创建主视图**

在这个时候我们可以编写一个待办事项程序。但是我们要专注于 NW.js 和它本身的功能。因此，我更倾向让你自己来决定我们的应用程序的功能。 我在 GitHub 上创建了一个示例项目 NW.js - 示例来演示几个 NW.js 功 能，例如，Node.js 集成和剪贴板访问。 您随时可以在应用程序中用它来测试和研究。当然你也可以使用其他的功能。

不管你作何决定，你都必须要创建 thesrc/views/main.html 文件。因为它是我们应用程序的入口点。文件内容如下：

<!doctype html>

<html>

<head>

<meta charset="utf-8">

<title>NW.js-example | main</title>

<link rel="stylesheet" href="../styles/common.css">

</head>

<body>

<h1>Hello World :-)</h1>

<src="../app/main.js"></>

</body>

</html>

在真正的应用程序中，你可能有个多个其他的视图文件并用 Ajax 加载它们。

为了简单起见，您还可以创建本地超链接并引用其他 HTML 文件。例如：

<a href="something.html">Something</a>

然后，在 src / views / 中创建 something.html 文件。

**安装 NW.js**

现在我们创建了 NW.js 项目，包括清单和主视图。最终我们需要一个方法直接在开发中运行 NW.js，并通过构建过程生成可运行于多个系统的本地应用。

为达此目的，我们需要两个包：

● nw (开发)

● nw-builder (生产)

既然它们与我们的应用功能无关（使用它们只是为了开发和生产目的），我们在第二个，即放在根目录下的 package.json 中创建 devDependencies 配置，加入它们。这个配置与必须的 name 和 version 字段并列：

{

"name":"nw.js-example",

"version":"1.0.0",

"devDependencies":{

"nw":"^0.18.2",

"nw-builder":"^3.1.2" }

}

现在只需要在项目的根目录下运行下面的命令来安装 devDependencies：

$ npm install

搞定！可以构建了。

**打包和发布**

我们在 package.json 中加入 npm 脚本来让打包变得简单。npm 脚本在右边定义要运行的 CLI 命令，在左边定义它的快捷方式，然后允许我们通过 npm run 来运行指定的快捷方式。现在添加两个脚本，一个用于开发，一个用于生产：

{

"name":"nw.js-example",

"version":"1.0.0",

"devDependencies":{

"nw":"^0.18.2",

"nw-builder":"^3.1.2" },

"s":{

"dev":"nw src/",

"prod":"nwbuild --platforms win32,win64,osx64,linux32,linux64 --buildDir dist/ src/" }

}

**直接运行 NW.js**

直接运行 NW.js 应用，只需要这个命令：

$ npm run dev

这个快捷方式会调用我们定义在 下 dev 对应的命令以使用 nw 包。开发机器上直接打开一个新窗口，显示着 src/views/main.html。

**生产构建**

要进行生产构建需要使用 nw-builder，它支持针对 Windows、Linux 和 macOS 进行输出。在我们的例子中，我们会针对所有这些平台打包，并且包含 32 位和 64 位版本。对于 macOS 来说，目前只能在以前的模式下构建 32 位的版本。（参阅 GitHub 上的这个 issue。）所以，我们只构建 64 位版本。

运行如下命令进行生产构建：

$ npm run prod

和直接运行 NW.js 一样，它也使用我们定义在 s 中的 CLI 命令。

然后得等一会儿 …

![](http://img.mp.itc.cn/upload/20161219/2b3ffc05378d45d7a7da4fc3f3df573f_th.jpeg)

完成之后，看看你的 dist/ 目录，就像这样：

dist/

└── nw.js-example/

├── linux32/

├── linux64/

├── osx64/

├── win32/

└── win64/

非常好，我们差不多完成了！

**测试和调试**

**手工测试**

既然 NW.js 是基于 Chrominum 的，那么它的手工测试和 Chrome 一样简单。如果你遇到一个 BUG——可视效果或者功能性的——你都可以通过快捷键 F12 或者下面的程序打开开发者工具：

nw.Window.get().showDevTools();

![](http://img.mp.itc.cn/upload/20161219/e97b24ce88f9425b8b83035e1f8ff0f3_th.jpeg)

注意这需要 SDK build flavor[译者注：flavor 可以理解为插件或者功能扩展]。如果你想在生产构建的时候去掉开发者工具，你可以使用另一个 flavor 或者直接阻止 F12 事件。

**自动化测试**

自动化的单元测试（幸好有你）广泛用于确保不同的实现能正确工作，这避免了总是进行手工测试。

![](http://img.mp.itc.cn/upload/20161219/695ee42a6cba416a89f0d87097ce3e91.png)

如果你的应用不使用 NW.js API 提供的特有方法，理论上来说你可以停留在一般的 Web 应用工作流上——例如，把 Karma 作为一个运行器与作为框架的 Jasmine 联合使用。

但是如果你使用 NW.js API 特有的方法，你需要在 NW.js 应用中运行测试，这才能确保使用到的 API 方法存在。一种方法是使用 NW.js 的 Karma 启动器插件，比如 karma-nodewebkit-launcher。它和其它浏览器的 Karma 启动器插件一样：在 NW.js 容器中打开应用并进行检查，完成之后再自动关闭应用。

不过既然 NW.js 不是 headless（就像 PhantomJS 那样的)[译者注：Headless 浏览器就是没有 GUI 的浏览器], 它总是需要显示出来。换句话说，它不能在纯 CLI 服务器上运行测试。幸运的是，这种情况下可以使用 Xvfb 来模拟显示。在 Jenkins[译者注：一个持续集成引擎] 中，你需要安装 Xvfb 插件。更多信息参阅 GitHub 上的这个讨论。

**总结**

希望本文介绍已让你熟悉 NW.js 的优点及使用环境。使用混合应用程序，要比分发包含 HTML 文件的压缩文件夹然后运行好得多，有很多因素可以证明这一点。NW.js 可以用来代替本地的应用程序，因为你不再需要专注于复杂的 GUI，并且，它有很多内置的功能，如视频播放器。 由于可以检测运行环境，你可以用 NW.js 开发既能在常规 Web 服务器上使用，又能在客户端计算机上使用的应用程序。配合使用一些小技巧，再加上有强大的 Chromium 引擎，用户体验几乎与本机应用程序无异。

当创建一个新的 NW.js 项目时，你首先要明确想使用的框架、模块加载器和预处理器——这取决于你对这些工具的熟悉度——否则只能是从头开始。做好决定之后，您可以寻找一个适合您需求的 NW.js 样板文件。 如果没有适合的样板，您可以使用基于本教程的应用程序作为开发的基础。

推荐阅读

点击 **“阅读原文”** 查看更多精彩内容[返回搜狐，查看更多](https://www.sohu.com/?strategyid=00001&spm=smpc.content.content.2.1630428234049ZsZzs6m "点击进入搜狐首页")

责任编辑：