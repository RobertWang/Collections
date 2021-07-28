> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/liuxiao723846/article/details/107051209)

零、前言
----

为什么要去看 VSCode？  
因为我们团队在做的中后台快速研发平台[云凤蝶](https://zhuanlan.zhihu.com/p/90238943)也是一款类似 Web IDE 形态的产品：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi1kNmMyY2E5ZjZjMWFhMmIzYjRmYTZkZmZhMTBkOTkwM183MjB3LmpwZw?x-oss-process=image/format,png)

而谈起 Web IDE，没人能绕开 VSCode，它非常流行，同时又完全开源，总共 350000 行 TypeScript 代码的巨大工程，使用了 142 个开源库。市面上选择基于 VSCode 去修改定制的 IDE 比比皆是：[Weex Studio](https://link.zhihu.com/?target=http%3A//emas.weex.io/zh/tools/ide.html)、[白鹭 Egret Wing](https://link.zhihu.com/?target=https%3A//www.egret.com/products/wing.html)、[快应用 IDE](https://link.zhihu.com/?target=https%3A//www.quickapp.cn/docCenter/IDEPublicity)...

我希望从 VSCode 身上看到什么？

*   大型复杂 GUI 软件（如 IDE 类）如何组织功能模块代码
*   如何使用 Electron 技术将 Web 软件桌面化
*   如何在打造插件化开放生态的同时保证软件整体质量与性能
*   如何打造一款好用的、流行的工具软件

一、VSCode 是什么
------------

### 官方定义

> [https://code.visualstudio.com/](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMyLnpoaW1nLmNvbS84MC92Mi01MjNmODM0NGFmODYzMDMzYTcwNGJjMzBlYjZlYzI2ZF83MjB3LmpwZw?x-oss-process=image/format,png)

关键词：

*   代码编辑（工具属性）
*   跨平台运行、开源
*   核心功能：
    *   IntelliSense（代码提示）、 debugging（代码调试）、 git（代码管理） 都是围绕代码编辑的核心链路
    *   extensions （插件）则肩负着打造开放生态的责任

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMyLnpoaW1nLmNvbS84MC92Mi1mNTJjYWYyZWVmM2UyMzEwMjRhNzk4ZDY3ZjJhYzk4OV83MjB3LnBuZw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi03YTc2MmYxODMzZmEzYjk2MzhiZjlmZWU1NjJhZTE2Zl83MjB3LmpwZw?x-oss-process=image/format,png)

```
点评：
对于工具软件而言，需要内心能想清楚边界。
哪些是自己应该专注去做的，哪些可以外溢到交给第三方扩展来满足。 
```

### 发展历程

*   2015-04 （4 年前） 发布
*   2015-11 （发布之后半年）开源
*   2019-05 发布 VSCode Remote Development

团队负责人：[Erich Gamma](https://link.zhihu.com/?target=https%3A//zh.wikipedia.org/zh/%25E5%259F%2583%25E9%2587%258C%25E5%25B8%258C%25C2%25B7%25E4%25BC%25BD%25E7%2591%25AA) . [JUnit](https://link.zhihu.com/?target=https%3A//junit.org/junit5/) 作者之一，[《设计模式》](https://link.zhihu.com/?target=https%3A//book.douban.com/subject/1052241//)作者之一， Eclipse 架构师。2011 加入微软，在瑞士苏黎世组建团队开发基于 web 技术的编辑器，也就是后来的 monaco-editor。VSCode 开发团队从 10 来个人开始，早期成员大多有 Eclipse 开发团队的背景。

> [Visual Studio Code 有哪些工程方面的亮点](https://zhuanlan.zhihu.com/p/35303567)  
> [维护一个大型开源项目是怎样的体验？](https://www.zhihu.com/question/36292298)  
> [「Shape Up」 适合中小团队的一种工作方式](https://link.zhihu.com/?target=https%3A//rebornix.com/work/2019/10/18/Shape-Up/)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi04NmMwYTdhNzEzZTg4ZmZlNDMxOWVjY2M4OTgwOTlmZl83MjB3LmpwZw?x-oss-process=image/format,png)

Erich Gamma 在 GOTO 2016 发表了主题为 《[The journey of visual studio code: Building an App Using JS/TypeScript, Node, Electron & 100 OSS Components](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DuLrnQtAq5Ec)》的演讲，详细讲解了这个项目的发展历程：

> 没时间观看视频的同学可以选择下载完整  [PDF](https://link.zhihu.com/?target=https%3A//gotocon.com/dl/goto-amsterdam-2016/slides/ErichGamma_TheJourneyOfALargeScaleApplicationBuiltUsingJavaScriptTypeScriptNodeElectron100OSSComponentsAtMicrosoft.pdf)

PPT 的第一页，就是 Erich Gamma 截取自己正式加入微软之后收到的工作内容描述的邮件：

**” 探索一种全新的和桌面 IDE 一样成功的在线开发工具模式 “**

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMxLnpoaW1nLmNvbS84MC92Mi02M2UxNGYwY2RlMmNhYWQ5ZDYyOTcyOWRlNDNlYTIzMF83MjB3LmpwZw?x-oss-process=image/format,png)

整个团队从大致 10 个人开始，混合老中新三代不同水平的程序员，在微软这个巨无霸的商业公司里面想要落地这样一个宏大的愿景是不容易的，团队一开始定下的思路就是像 start up 一样工作，每月每年都要 ship 东西。

同时他也提出早期会疯狂的在公司内部寻找落地场景，比如 Visual Studio Online 的在线 Code DIff 页面，TypeScript 的官网的 Playground 编辑器，OneDrive 代码文件，Edge 浏览器 Dev Tool 的代码浏览等。

一个重要转折点是微软本身发生的巨大变化：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMyLnpoaW1nLmNvbS84MC92Mi01MjI0MGQzOWE2MDY3NzRkMDk5MTE5ZTE4NGFjMTNmMV83MjB3LmpwZw?x-oss-process=image/format,png)

伴随微软整个的开放开源跨平台风潮，Erich Gamma 敏锐的决定将产品从 Browser Based IDE 转向跨平台的 Desktop IDE，但仍然使用 Web 技术，于是 electron 完美契合，VSCode 团队花了六个月使用 Electron 将 Web 编辑器桌面化，又花了六个月将整个 IDE 插件化，最终 VSCode 成为一个流行的产品同时也成为一个典型的 Electron 客户端开源项目。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMyLnpoaW1nLmNvbS84MC92Mi02MjM5YTc2OWZjMDU3M2YyZDM4YzcxN2IzZDA1ODI5ZF83MjB3LmpwZw?x-oss-process=image/format,png)

### 产品定位

Erich Gamma 在 2017 SpringOne Platform 上有一个 [关于 VSCode 的分享](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DVs3AGfeuNKU)，讲解了在他开发 Eclipse 的过往经验基础上，对 VSCode 进行顶层设计时的诸多思路与决策，其中提到过对于 VSCode 的产品定位：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi02MjNhNjNiZTJmZGU3YjU0OWEzYzQ3MWQ0OGZjNTg1Yl83MjB3LmpwZw?x-oss-process=image/format,png)

从图中可以看出 VSCode 定位是处于编辑器和 IDE 的中间并且偏向轻量编辑器一侧的。  
VSCode 的核心是 “编辑器 + 代码理解 + 调试 “，围绕这个关键路径做深做透，其他东西非常克制，产品保持轻量与高性能。

```
点评：
生产力工具类的软件一定要守住主线，否则很可能会变成不收门票的游乐园
牛逼的产品背后一定有牛逼的团队，比如微软挖到 Anders Hejlsberg，接连创造了 C# 和 TypeScript
挖到 Erich Gamma，接连诞生了 monaco 和 vscode 这些明珠
```

二、Electron 是什么
--------------

上文提到 VSCode 有一个特性是跨平台，它的跨平台实质是通过 electron 实现的。所以我们需要先简单了解下 [electron](https://link.zhihu.com/?target=https%3A//electronjs.org/)

### 官方定义

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMzLnpoaW1nLmNvbS84MC92Mi03OTE4M2I0ZjU3NWRkZWNkZWYyM2Y5ZjU2ZmJiODRmNl83MjB3LmpwZw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMxLnpoaW1nLmNvbS84MC92Mi0yN2Q4OWM0ZTlhOWFkNDMyMWZhZWU5YjQwNTc3M2MyNF83MjB3LmpwZw?x-oss-process=image/format,png)

### 核心技术

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMxLnpoaW1nLmNvbS84MC92Mi04MmQ2Y2RlYWVkNDBhM2RjYjRkYWNjN2M2ZjczNzQ1NF83MjB3LmpwZw?x-oss-process=image/format,png)

*   使用 Web 技术来编写 UI，用 chrome 浏览器内核来运行
*   使用 NodeJS 来操作文件系统和发起网络请求
*   使用 NodeJS C++ Addon 去调用操作系统的 native API

### 应用架构

> [https://electronjs.org/docs/tutorial/application-architecture](https://link.zhihu.com/?target=https%3A//electronjs.org/docs/tutorial/application-architecture)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi02MTYzNWY2MGM3MzM1YzllNzMyYmMyMDlmNjFiYjFmZl83MjB3LmpwZw?x-oss-process=image/format,png)

*   1 个主进程：一个 Electron App 只会启动一个主进程，它会运行 package.json 的 main 字段指定的脚本
*   N 个渲染进程：主进程代码可以调用 Chromium API 创建任意多个 web 页面，而 Chromium 本身是多进程架构，每个 web 页面都运行在属于它自己的渲染进程中

进程间通讯：

*   Render 进程之间的通讯本质上和多个 Web 页面之间通讯没有差别，可以使用各种浏览器能力如 localStorage
*   Render 进程与 Main 进程之间也可以通过 API 互相通讯 (ipcRenderer/ipcMain)

```
点评：
普通 web 页面无法调用 native api，因此缺少一些能力
electron 的 web 页面所处的 Render 进程可以将任务转发至运行在 NodeJS 环境的 Main 进程，从而实现 native API
这套架构大大扩展了 electron app 相比 web app 的能力丰富度
但同时又保留了 web 快捷流畅的开发体验，再加上 web 本身的跨平台优势
结合起来让 electron 成为性价比非常高的方案
```

三、VSCode 技术架构
-------------

### 多进程架构

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi1hOTNiYTAyYzVkOTRiNGQwZDM3Y2M1OWU2ODIxZTkwN183MjB3LmpwZw?x-oss-process=image/format,png)

*   主进程：VSCode 的入口进程，负责一些类似窗口管理、进程间通信、自动更新等全局任务
*   渲染进程：负责一个 Web 页面的渲染
*   插件宿主进程：每个插件的代码都会运行在一个独属于自己的 NodeJS 环境的宿主进程中，插件不允许访问 UI
*   Debug 进程：Debugger 相比普通插件做了特殊化
*   Search 进程：搜索是一类计算密集型的任务，单开进程保证软件整体体验与性能

### 开发流程

*   开发文档：[https://github.com/Microsoft/vscode/wiki/How-to-Contribute](https://link.zhihu.com/?target=https%3A//github.com/Microsoft/vscode/wiki/How-to-Contribute)
*   主仓库：[https://github.com/microsoft/vscode](https://link.zhihu.com/?target=https%3A//github.com/microsoft/vscode)
*   其它关联项目：[https://github.com/Microsoft/vscode/wiki/Related-Projects](https://link.zhihu.com/?target=https%3A//github.com/Microsoft/vscode/wiki/Related-Projects)

```
# 检出代码
git clone git@github.com:microsoft/vscode.git
cd vscode
# 安装依赖
yarn
# 启动 web 版
yarn watch && yarn web
# 启动 桌面 版
yarn watch && ./scripts/code.sh
# 打包
yarn run gulp vscode-[platform]
yarn run gulp vscode-[platform]-min
# platforms: win32-ia32 | win32-x64 | darwin | linux-ia32 | linux-x64 | linux-arm
```

### 源码组织

> [https://github.com/microsoft/vscode/wiki/Source-Code-Organization](https://link.zhihu.com/?target=https%3A//github.com/microsoft/vscode/wiki/Source-Code-Organization)

下面是整个 VSCode project 的一些顶级的重点文件夹，后文会重点关注 src 与 extensions:

```
├── build         # 构建脚本
├── extensions    # 内置插件
├── scripts       # 工具脚本
├── out           # 产物目录
├── src           # 源码目录
├── test          # 测试代码
```

VSCode 的代码架构也是随着产品阶段演进而不断更迭的：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMyLnpoaW1nLmNvbS84MC92Mi0xZDRjYjI1MmM4ZjI1YTM0OWE1YmVhNzk3MGExMGRhMV83MjB3LmpwZw?x-oss-process=image/format,png)

下文会分享一些整个 VScode 源码组织的一些亮点与特色：

### 1. 隔离内核 (src) 与插件 (extensions)，内核分层模块化

*   /src/vs：分层和模块化的 core
*   /src/vs/base: 通用的公共方法和公共视图组件
*   /src/vs/code: VSCode 应用主入口
*   /src/vs/platform：可被依赖注入的各种纯服务
*   /src/vs/editor: 文本编辑器
*   /src/vs/workbench：整体视图框架
*   /src/typings: 公共基础类型
*   /extensions：内置插件

### 2. 每层按环境隔离

内核里面每一层代码都会遵守 electron 规范，按不同环境细分文件夹:

*   common: 公共的 js 方法，在哪里都可以运行的
*   browser: 只使用浏览器 API 的代码，可以调用 common
*   node: 只使用 NodeJS API 的代码，可以调用 common
*   electron-browser: 使用 electron 渲染线程和浏览器 API 的代码，可以调用 common，browser，node
*   electron-main: 使用 electron 主线程和 NodeJS API 的代码，可以调用 common， node
*   test: 测试代码

```
点评：
云凤蝶也遇到了类似问题，作为一个 低代码 + 可视化 的研发平台，许多功能模块的实现都需要横跨编辑态和运行态
如果代码不加以限制和区分，很容易导致错误的依赖关系和预期之外的 bug
因此云凤蝶最终也决定采用 (editor/runtime/common) 类似的隔离架构
```

### 3. 内核代码本身也采用扩展机制: Contrib

可以看到 /src/vs/workbench/contrib 这个目录下存放着非常多的 VSCode 的小的功能单元：

```
├── backup
├── callHierarchy
├── cli
├── codeActions
├── codeEditor
├── comments
├── configExporter
├── customEditor
├── debug
├── emmet
├──....中间省略无数....
├── watermark
├── webview
└── welcome
```

Contrib 有一些特点：

*   Contrib 目录下的所有代码不允许依赖任何本文件夹之外的文件
*   Contrib 主要是使用 Core 暴露的一些扩展点来做事情
*   每一个 Contrib 如果要对外暴露，将 API 在一个出口文件里面导出 eg: contrib/search/common/search.ts
*   一个 Contrib 如果要和另一个 Contrib 发生调用，不允许使用除了出口 API 文件之外的其它文件
*   接上一条，即使 Contrib 事实上可以调用另一个 Contrib 的出口 API，也要审慎的考虑并尽量避免两个 Contrib 互相依赖

VSCode 开发团队做这个设计的目的我猜大概是因为重型的工具软件功能点实在太多，而且非常多的地方都是采用相似的模式去横向扩展，如果这些功能代码直接采用原始的模块引用的方式在 core 里面硬编码聚合拼装起来，是一个自顶向下的架构，对维护性的挑战比较大。

而采用暴露扩展点的方式，可以将依赖关系反转，依附于扩展点协议，独立的小功能的代码实现可以单独聚合，核心模块无需硬编码和集成所有判断，整体是一个松散式的架构，降低了代码信息密度与提升维护性，也更好扩展。

但是 VSCode Contrib 的具体业务代码组织其实看起来没有太多范式，而且这个内核代码的扩展机制 Contrib 和 VSCode 开放给外界的插件化机制 extension 是有差异的，读起来十分头疼。

通过和兄弟团队 CloudIDE 开发组的专家交流，我得到两条主要差异性：

*   extension 每一个都是运行在归宿于自己的独立宿主进程，而 contrib 的功能基本是要运行在主进程的
*   extension 只能依附于 core 开放的扩展点而活，但是 contrib 可以通过依赖注入拿到所有 core 内部实现的 class （虽然官方不推荐）

### 4. 依赖注入

上一小节提到了 VSCode 的代码大量使用了依赖注入，这项技术的具体实现细节本文不会展开细讲，感兴趣的可以阅读一些好的实现：

*   云凤蝶团队同学写的 [power-di](https://link.zhihu.com/?target=https%3A//github.com/zhang740/power-di)
*   社区开源的强大的 [http://inversify.io](https://link.zhihu.com/?target=http%3A//inversify.io/)

TS 依赖注入常见的实现原理是使用 [reflect-metadata](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/reflect-metadata) 设置与获取元信息，从而可以实现在运行时拿到本来属于编辑态的 TypeScript 类型相关元信息，具体来说就是下面这些 API：

*   Reflect.getMetadata("design:type", , target, key): 获取 class 属性类型元信息
*   Reflect.getMetadata("design:paramtypes", target, key): 获取 class 方法参数元信息
*   Reflect.getMetadata("design:returntype", target, key)：获取 class 方法返回值元信息
*   Reflect.defineMetadata(metadataKey, metadataValue, target, propertyKey)：设置元信息
*   Reflect.getMetadata(metadataKey, target, propertyKey): 获取元信息

不过具体到 VSCode 的依赖注入，它没有使用 reflect-metadata 这一套，而是基于 decorator 去标注元信息，整个实现了一套自己的依赖注入方式，具体可以参考 [vscode 源码解析 - 依赖注入](https://zhuanlan.zhihu.com/p/96902077) 这篇文章，大致包含如下几类角色：

*   Service：服务的实现逻辑
*   Interface：服务的接口描述
*   Client：服务使用方
*   Manger：服务管理器

举个例子来看，在 /src/core/platform 里面定义了大量 service，其他地方消费者 Client 都可以用依赖注入的方式使用到，伪代码如下：

```
class Client {
  // 构造函数参数注入（依赖注入方式的一种）
  constructor(
    // 必选
    @IModelService modelService: IModelService,
    // 可选
    @optional(IEditorService) editorService: IEditorService
  ) {
    // use services
  }
}
 
// 实例化
instantiationService.createInstance(Client);
```

### 5. 绝对路径 import

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi00MjAxNWJkYzQ2NjM4ZWRiMTU5OGE4YmQ2NzZiNDFhZl83MjB3LmpwZw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi0zMzQ4MDNlYmYzOTgwM2E5YTUxNGE3NjNmNTUwNGM2Yl83MjB3LmpwZw?x-oss-process=image/format,png)

```
点评：
绝对路径 import 是一个非常值得学习的技巧，具体的方式是配置 TypeScript compilerOptions.paths
相对路径 import 对阅读者的大脑负担高，依赖当前文件位置上下文信息才能理解
假设修改代码的时候移动文件位置，相对路径需要修改本文件的所有 import，绝对路径不需要
```

### 6. 命令系统

VSCode 和 monaco-editor 都有自己的命令系统，蚂蚁 [CloudIDE](https://zhuanlan.zhihu.com/p/83860143) 团队的同学也曾经对命令系统的优势做过总结：

传统的模块调用是个网状，不太好找到一个切面来理解或治理：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi1mYTE3NmU3NzE5NjAxNzdiOGY2NGUzMzRiOWRjNTBlN183MjB3LmpwZw?x-oss-process=image/format,png)

而命令系统是中心化的，各功能末端变成了扁平化的结构：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMyLnpoaW1nLmNvbS84MC92Mi03NjAyYWRiYTJmYmJlNjI4YWE0MzQyOTRlY2ZjM2NhNV83MjB3LmpwZw?x-oss-process=image/format,png)

```
点评：
云凤蝶编辑器也遇到这类问题，云凤蝶的自由画布编辑器有非常多的功能，参见 《打造媲美 Sketch 的自由画布》
而且很多情况下一个功能的触发方式是多种多样的，比如大纲树右键菜单触发，顶部工具栏触发，画布右键菜单触发，键盘快捷键触发等
这就要求该功能的实现函数能跨区域在多个位置调用，这很容易导致代码依赖关系异常混乱。
而命令系统就是一种解决这个问题的很好思路
```

### 7. TypeScript

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMxLnpoaW1nLmNvbS84MC92Mi01MDcwODg5ZjhkMzI1YTA2NDIyZTUyYzQwNWI5ODM4NF83MjB3LmpwZw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi1hOTNmNjMzZjcwNjEzZDEyZDRjNzEzOTg2ZDkzODAzYl83MjB3LmpwZw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi1kNTA2ZmVjZDhjZjJmYzY4MWFjYTRkZTVkZGEzNjBlYl83MjB3LmpwZw?x-oss-process=image/format,png)

### 启动流程 （TLDR）

上文初步了解了 vscode 的技术架构与源码组织，手痒的同学估计有点等不及尝试走一遍 vscode 的启动流程了。

然后在正式发车之前，我需要给大家一点友情提醒，如果你没耐心看完下面的 VSCode 的启动流程，应该知道，人生得过且过 o(╥﹏╥)o

总体来看，VSCode 的启动代码真正 show 给我们看了一个复杂的客户端软件的代码会工程化到什么地步，这其中掺杂了大量的基于 TypeScript 的 OOP 式的代码组织，各种对边界，宿主环境，上下文的处理，本来简单的启动 APP 渲染一个页面流程变得极其复杂。

下面精简抽取核心启动链路的文件和方法看一看：

*   入口：package.json {"main": "./out/main"} : electron 的标准启动入口
*   '/out/main.js': 是构建产物的入口文件，它对应源码 '/src/main.js'

```
// /src/main.js 的精简核心链路
const { app, protocol } = require('electron');
 
app.once('ready', function () {
  // electron 启动好之后，调用 vscode 的入口
    onReady();
});
 
async function onReady() {
    // 获取缓存文件目录地址和语言配置，执行启动
        const [cachedDataDir, nlsConfig] = await Promise.all([nodeCachedDataDir.ensureExists(), resolveNlsConfiguration()]);
        startup(cachedDataDir, nlsConfig);
}
 
function startup(cachedDataDir, nlsConfig) {
  // 先加载 vscode 自己开源的 AMD Loader https://github.com/Microsoft/vscode-loader/
  // 再使用这个 loader 去加载 VSCode 的主入口文件
    require('./bootstrap-amd').load('vs/code/electron-main/main');
}
// /src/vs/code/electron-main/main.ts 精简核心链路
 
// 初始化主类
const code = new CodeMain();
// 执行主入口函数
code.main();
 
class CodeMain {
  main() {
    // vscode 的 class public 入口一般只是空壳，真正的都在 private 逻辑里面
    this.startUp();
  }
  private async startup() {
    // 先创建依赖的初始化 service
        const [instantiationService, instanceEnvironment] = this.createServices();
    // 创建编辑器实例并调用 startUp 方法
    return instantiationService.createInstance(CodeApplication).startup();
  }
}
// /src/vs/code/electron-main/app.ts
 
export class CodeApplication extends Disposable {
    async startup(): Promise<void> {
        // IPC Server
        const electronIpcServer = new ElectronIPCServer();
        // SharedProcess
        const sharedProcess = this.instantiationService.createInstance(SharedProcess, machineId, this.userEnv);
        // 创建一大堆依赖的 service
        // IUpdateService IWindowsMainService IDialogMainService IMenubarService IStorageMainService......
        const appInstantiationService = await this.createServices(machineId, trueMachineId, sharedProcess, sharedProcessClient);
        // 打开一个窗口
            const windows = appInstantiationService.invokeFunction(accessor => this.openFirstWindow(accessor, electronIpcServer, sharedProcessClient));
    }
      private openFirstWindow() {
      const windowsMainService = this.windowsMainService = accessor.get(IWindowsMainService);
            return windowsMainService.open();
    }
}
// /src/vs/platform/windows/electron-main/windowsMainService.ts
export class WindowsMainService extends Disposable implements IWindowsMainService {
  open() {
    // 执行 open
    this.doOpen();
  }
  private doOpen() {
    // 打开浏览器窗口
    this.openInBrowserWindow();
  }
  private openInBrowserWindow() {
            // 创建窗口
            const createdWindow = window = this.instantiationService.createInstance(CodeWindow, {
                state,
                extensionDevelopmentPath: configuration.extensionDevelopmentPath,
                isExtensionTestHost: !!configuration.extensionTestsPath
            });
  }
    private doOpenInBrowserWindow() {
        // 加载页面
    window.load(configuration);
  }
}
// /src/vs/code/electron-main/window.ts
export class CodeWindow extends Disposable implements ICodeWindow {
  load() {
        // 调用 electron 的 api 加载一个 url 的 html 页面
        this._win.loadURL(this.getUrl(configuration));
  }
    private getUrl() {
    // 获取要打开的 url
    let configUrl = this.doGetUrl(config);
    return configUrl;
  }
    private doGetUrl(config: object): string {
    // 终于看到 html 了！！泪流满面〒▽〒
    // 打开 VSCode 的工作台，也就是 workbench  
        return `${require.toUrl('vs/code/electron-browser/workbench/workbench.html')}?config=${encodeURIComponent(JSON.stringify(config))}`;
    }
}
```

### 代码编辑器技术

因为本文关注的重点并不在真正的代码编辑器技术而是在调研一下大型软件的工程化，因此本文只会简要介绍一下代码编辑相关的的一些核心技术：

*   monaco-editor 文本编辑器，非常精深的领域，不展开聊，可以看一些有意思的细节：
    *   [Text Buffer 性能优化](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/blogs/2018/03/23/text-buffer-reimplementation)
    *   [MVVM 架构](https://link.zhihu.com/?target=https%3A//github.com/microsoft/vscode/wiki/%255BWIP%255D-Code-Editor-Design-Doc)
*   language server protocol 语言提示， 也是 vscode 的一大创举：
    *   不再关注 AST 和 Parser，转而关注 Document 和 Position，从而实现语言无关。
    *   将语言提示变成 CS 架构，核心抽象成当点击了文档的第几行第几列位置需要 server 作出什么响应的一个简单模型，基于 JSON RPC 协议传输，每个语言都可以基于协议实现通用后端

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMxLnpoaW1nLmNvbS84MC92Mi1kZjE1ZGZiNzYxODBkOTJkNmY4MWVjYjYxOWYyZjQxNF83MjB3LmpwZw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi1hNDhlZDY1YzQwY2NiYzc0YTNhOWEzMDM5ODY4NDg1N183MjB3LmpwZw?x-oss-process=image/format,png)

*   [Debug Adaptor Prototal](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/api/extension-guides/debugger-extension): 调试协议

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMzLnpoaW1nLmNvbS84MC92Mi01OWI3MTg3YmIyNmRkMmZmNjhlMTc4MmMwM2I3ZTIwNl83MjB3LmpwZw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi05M2VmNjg5ZWIzZTM1OWRkOWUxOWI3MjI0ZDFlMWU2Zl83MjB3LmpwZw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMxLnpoaW1nLmNvbS84MC92Mi1iMTIyZWZlNzE5ZTU2YzcyZTFjZWEwZTY1NjI0ZmQwNF83MjB3LmpwZw?x-oss-process=image/format,png)

```
点评：
monaco-editor 可以看出专家级人物的领域积累，属于 VSCode 的核心竞争力
language server protocol 和 Debug Adaptor Prototal 的设计就属于高屋建瓴
可以看出思想层次，把自己的东西做成标准，做出生态，开放共赢
```

四、VSCode 插件系统
-------------

### 理念差异

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMzLnpoaW1nLmNvbS84MC92Mi0xZGEzYmNkMzUxMjM2ZjNlNmRiZjVkZWE2ZDBjMTY5Nl83MjB3LmpwZw?x-oss-process=image/format,png)

对比几大 IDE：

*   Visual Studio / IntelliJ：不需要插件，all in one （不够开放）
*   Eclipse: 一切皆插件 （臃肿、慢、不稳定、体验差）
*   VSCode：中庸之道

### VSCode 插件的强隔离

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi1jNDkzYTUyYjg5Yzk2OGQ1ZWZiZjIwOGQzOGQyZGZlN183MjB3LmpwZw?x-oss-process=image/format,png)

*   独立进程：VSCode plugin 代码运行在只属于自己的独立 Extension Host 宿主进程里
*   逻辑与视图隔离：插件完全无法访问 DOM 以及操作 UI，插件只能响应 VSCode Core 暴露的扩展点
*   视图扩展能力非常弱：VSCode 有非常稳定的交互与视觉设计，提供给插件的 UI 上的洞（component slot）非常少且稳定
*   只能使用限制的组件来扩展：VSCode 对视图扩展的能力限制非常强，洞里面的 UI 是并不能随意绘制的，只能使用一些官方提供的内置组件，比如 TreeView 之类

> vscode 有哪些扩展能力？ [https://code.visualstudio.com/api/extension-capabilities/overview](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/api/extension-capabilities/overview)

```
点评： 
视图扩展的克制带来统一的视觉与交互风格，带来好的用户体验，便于建立稳定的用户心智 
插件独立进程，与视图隔离，保证整体软件的质量、性能、安全性
```

### Workbench 视图结构

> [https://code.visualstudio.com/docs/getstarted/userinterface](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/docs/getstarted/userinterface)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWM0LnpoaW1nLmNvbS84MC92Mi02MzFjMDk0ZjBiMmU5YTEwNjAxM2EzYjY5YzkxMTg2Yl83MjB3LmpwZw?x-oss-process=image/format,png)

*   标题栏: Title Bar
*   活动栏: Activity Bar
*   侧边栏: Side Bar
*   面板: Panal
*   编辑器: Editor
*   状态栏: Status Bar

在这个视图结构里面有哪些可扩展呢？详见 [extending workbench](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/api/extension-capabilities/extending-workbench)：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9waWMxLnpoaW1nLmNvbS84MC92Mi1iNDg1OTU1OWE2YTMxNzIzNzRmYTQwZDAzNzhkNmY1Y183MjB3LmpwZw?x-oss-process=image/format,png)

### 插件 API 注入

插件开发者调用 core 能力时需要引入名为 vscode 的 npm 模块

```
import * as vscode from 'vscode';

```

而实际上这只是一个 vscode.d.ts 类型声明文件，它声明了所有插件可用的 API 类型。  
这些 API 的具体实现在 src/vs/workbench/api/common/extHost.api.impl.ts createApiFactoryAndRegisterActors

那么具体这些 API 在 plugin 执行上下文是何时注入的呢？其实是在插件 import 语句执行的时候动了手脚。

```
// /src/vs/workbench/api/common/extHostRequireInterceptor.ts
 
class VSCodeNodeModuleFactory implements INodeModuleFactory {
    public load(_request: string, parent: URI): any {
 
        // get extension id from filename and api for extension
        const ext = this._extensionPaths.findSubstr(parent.fsPath);
        if (ext) {
            let apiImpl = this._extApiImpl.get(ExtensionIdentifier.toKey(ext.identifier));
            if (!apiImpl) {
                apiImpl = this._apiFactory(ext, this._extensionRegistry, this._configProvider);
                this._extApiImpl.set(ExtensionIdentifier.toKey(ext.identifier), apiImpl);
            }
            return apiImpl;
        }
 
        // fall back to a default implementation
        if (!this._defaultApiImpl) {
            let extensionPathsPretty = '';
            this._extensionPaths.forEach((value, index) => extensionPathsPretty += `\t${index} -> ${value.identifier.value}\n`);
            this._logService.warn(`Could not identify extension for 'vscode' require call from ${parent.fsPath}. These are the extension path mappings: \n${extensionPathsPretty}`);
            this._defaultApiImpl = this._apiFactory(nullExtensionDescription, this._extensionRegistry, this._configProvider);
        }
        return this._defaultApiImpl;
    }
}
```

vscode plugin 的 require 全部被 [Microsoft/vscode-loader](https://link.zhihu.com/?target=https%3A//github.com/Microsoft/vscode-loader) 劫持了，通过对 require 的 hack 将插件 API 注入到了运行环境。

### 插件开发与配置

脚手架： [https://github.com/Microsoft/vscode-generator-code](https://link.zhihu.com/?target=https%3A//github.com/Microsoft/vscode-generator-code)  
官方 demo: [https://github.com/Microsoft/vscode-eslint](https://link.zhihu.com/?target=https%3A//github.com/Microsoft/vscode-eslint)

```
npm install -g yo generator-code
yo code
```

一个插件核心就是一个配置文件：Extension Manifest JSON (package.json 里面的一个字段)

> [https://code.visualstudio.com/api/references/extension-manifest](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/api/references/extension-manifest%25EF%25BC%2589)

一些关键配置如下：

### main：主文件入口，比如导出一个 activate 方法，可以接受 ctx 做一些事情

```
"main": "./src/extension.js",
// extension.js
const vscode = require("vscode");
 
function activate(context) {
  console.log('Congratulations, your extension "helloworld" is now active!');
 
  let disposable = vscode.commands.registerCommand(
    "extension.helloWorld",
    function() {
      vscode.window.showInformationMessage("Hello World!");
    }
  );
 
  context.subscriptions.push(disposable);
}
 
exports.activate = activate;
```

### Activation Events 激活时机

*   onLanguage：包含该语言类型的文件被打开
*   onLanguage:json
*   onCommand：某个命令
*   onCommand:extension.sayHello
*   onDebug：开始调试
*   onDebugInitialConfigurations
*   onDebugResolve
*   workspaceContains：有匹配规则的文件被打开
*   workspaceContains:**/.editorconfig
*   onFileSystem：打开某个特殊协议的文件
*   onFileSystem:sftp
*   onView：某个 id 的视图被显示
*   onView:nodeDependencies
*   onUri：向操作系统注册的 schema
*   vscode://vscode.git/init
*   onWebviewPanel：某种 viewType 的 webview 打开时
*   onWebviewPanel:catCoding
*   *：启动就立即打开

### Contribution Points 扩展点

*   contributes.configuration：本插件有哪些可供用户配置的选项
*   contributes.configurationDefaults：覆盖 vscode 默认的一些编辑器配置
*   contributes.commands：向 vscode 的命令系统注册一些可供用户调用的命令
*   contributes.menus：扩展菜单
*   ...

五、感想
----

Eric Raymond 有一本非常知名的著作 [《大教堂与集市》](https://link.zhihu.com/?target=https%3A//book.douban.com/subject/25881855/)，其中提到过一些有意思的观点：

*   健壮的结构远比精巧的设计来得重要。换句话说，结构是第一位的，功能是第二位的。
*   保持项目的简单性。设计达到完美的时候，不是无法再增加东西了，而是无法再减少东西了。

VSCode 的一些工程上的优秀设计，比如依赖注入、绝对路径引用、命令系统对于云凤蝶来说是可以马上学以致用的，而 contrib 与 extension 的扩展系统，则非一日之功，也并不宜盲目下手。

而事实上在尝试打造每一个开发者都梦想的万物皆 plugin 式的工具软件之前，有一些通用的问题需要先冷静下来思考：

*   用户核心在操作的资源是什么？
*   用户的关键路径是什么？
*   这个软件的整体功能形态，交互与视觉设计已经稳定了吗？
*   内核功能区和第三方扩展的功能域之间的界限在哪里？
*   哪些环节可能会出现外溢需求需要第三方扩展才能被满足，不适宜官方动手做吗？

对 VSCode 而言：

*   核心操作的资源是文件
*   关键路径是：打开文件 - 编辑文件 - 保存文件
*   整体功能设计，交互与视觉设计非常稳定
*   内核是文件管理与代码编辑，多样性的编程语言生态，CICD 等衍生研发链路等可能会出现扩展需求

对云凤蝶而言：

*   核心操作的资源是页面：（分为视图和代码）
*   关键路径是：打开页面 - 编辑页面（拖拽视图，配置属性，编写代码）- 保存页面
*   整体功能设计还在快速迭代中
*   内核是页面的制作，多样性的资产生态，CI CD 等衍生研发链路等可能会出现扩展需求

围绕这个思考，云凤蝶将持续吸纳优秀的思想与架构，持续将编辑器的核心功能链路打磨通透，底层架构搭建稳定。

六、相关资料
------

*   [https://www.youtube.com/watch?v=uLrnQtAq5Ec](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DuLrnQtAq5Ec)
*   [https://www.youtube.com/watch?v=Vs3AGfeuNKU](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DVs3AGfeuNKU)
*   [https://github.com/JChehe/blog/issues/5](https://link.zhihu.com/?target=https%3A//github.com/JChehe/blog/issues/5)
*   [https://electronjs.org/docs/tutorial/application-architecture](https://link.zhihu.com/?target=https%3A//electronjs.org/docs/tutorial/application-architecture)
*   [https://zhuanlan.zhihu.com/p/54289476](https://zhuanlan.zhihu.com/p/54289476)
*   [http://phosphorjs.github.io/](https://link.zhihu.com/?target=http%3A//phosphorjs.github.io/)
*   [https://developer.egret.com/cn/docs/page/778](https://link.zhihu.com/?target=https%3A//developer.egret.com/cn/docs/page/778)
*   [https://developer.chrome.com/extensions/getstarted](https://link.zhihu.com/?target=https%3A//developer.chrome.com/extensions/getstarted)
*   [https://developer.alipay.com/article/8708](https://link.zhihu.com/?target=https%3A//developer.alipay.com/article/8708)
*   [https://zhaomenghuan.js.org/blog/vscode-custom-development-basic-preparation.html](https://link.zhihu.com/?target=https%3A//zhaomenghuan.js.org/blog/vscode-custom-development-basic-preparation.html)