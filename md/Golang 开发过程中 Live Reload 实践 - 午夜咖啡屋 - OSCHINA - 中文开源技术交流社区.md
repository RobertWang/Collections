> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [my.oschina.net](https://my.oschina.net/barat/blog/5524746)

> 几个月前开始尝试用 Golang 写 Web 应用时，第一次意识到：对于一个像我这样过去总写 Java 代码的老程序员来说，因各类 IDE 带来的便利性，几乎忽略了热加载（Live Reload）这个问题的存在。

几个月前开始尝试用 Golang 写 Web 应用时，第一次意识到：对于一个像我这样过去总写 Java 代码的老程序员来说，因各类 IDE 带来的便利性，几乎忽略了热加载（Live Reload）这个问题的存在。开始使用 Golang 之后，特别是是编写 Web 应用时，因为改写页面上一个 JavaScript 函数而需要暂停当前运行的程序，再重新运行程序实在太令人沮丧了。好在，Golang 开发相关的资料和方案都已经相当齐全，只需要找到合适的解决方案就可以愉快地编码。

任何一门编程语言，在开发过程中总是需要「编写代码」，之后「编译运行」这两个步骤。「编写代码」的过程，可以使用任何文本编辑器。「编译并运行」的过程需要编程语言提供的开发工具包来支持。而好的 IDE 可以将上述两个步骤紧密有效的链接起来，方便开发人员专注于实现业务本身。

Golang 开发过程中常见的 IDE 包括：免费的 Microsoft Visual Studio Code 以及商业收费的 JetBrains Goland。这篇文章讲解过程中，也将围绕着前述这两种 IDE 展开。

简单讲解一下关于「热加载」的概念。我们用中文书写的「热加载」相关的概念，源自于英文中的「Live Reload」以及 「Hot Reload」。而在英文中，这两者有着本质区别的：

1.  **Live Reload** ：当程序中的某一个或若干个文件发生变化时，重新加载或刷新整个程序。这个过程会导致程序状态丢失，比如：在界面上点击进入某个字界面等操作，会全部失效。最终的结果是，程序会重新启动并会是最初的样子。
    
2.  **Hot Reload** ：当程序中的某一个或若干个文件发生变化时，在不丢失程序运行状态的前提下更新修改的部分。比如：flutter 开发过程中就支持这种刷新方式。
    

![](https://oscimg.oschina.net/oscnet/up-078cfaf28435f7385e32c6a9ff4b2204771.png)

截图内容源自 stackoverflow.com

https://stackoverflow.com/questions/41428954/what-is-the-difference-between-hot-reloading-and-live-reloading-in-react-native

1**Visual Studio Code 实践方案**

使用 Visual Studio Code 作为 IDE 开发时，可以借助 cosmtrek/air 来实现热加载。air 本身也是用 Golang 实现的开发辅助工具，其主要的功能包括：

1.  监听应用文件夹内文件的变化并重新启动程序
    
2.  通过配置文件实现定制化需求
    
3.  可以设置忽略监听的文件以及其他辅助选项
    

https://github.com/cosmtrek/air

**安装 Air 的过程相当简单（请注意上网姿势）：**

```
# binary will be $(go env GOPATH)/bin/aircurl -sSfL https://raw.githubusercontent.com/cosmtrek/air/master/install.sh | sh -s -- -b $(go env GOPATH)/bin
# or install it into ./bin/curl -sSfL https://raw.githubusercontent.com/cosmtrek/air/master/install.sh | sh -s
```

安装完成之后，在项目根目录执行 **air init** 命令，可初始化 air 配置并生成默认配置文件，其内容可参考 [ohUrlShortener](https://gitee.com/barat/ohurlshortener) 项目使用的 air 配置：

```
root = "."testdata_dir = "testdata"tmp_dir = "tmp"
[build]  bin = "./tmp/ohurlshortener"  cmd = "go build -o ./tmp/ohurlshortener ."  delay = 1000  exclude_dir = ["tmp", "vendor", "testdata","docker"]  exclude_file = []  exclude_regex = ["_test.go"]  exclude_unchanged = false  follow_symlink = false  full_bin = ""  include_dir = []  include_ext = ["go", "tpl", "tmpl", "html","css","js"]  kill_delay = "0s"  log = "build-errors.log"  send_interrupt = false  stop_on_error = true
[color]  app = ""  build = "yellow"  main = "magenta"  runner = "green"  watcher = "cyan"
[log]  time = false
[misc]  clean_on_exit = true
[screen]  clear_on_rebuild = false
```

通过 **air -c .air.toml** 命令启动应用程序，air 将会监听上述配置文件描述的内容，当有文件修改或删除、更新等操作时，会重新加载整个应用程序，从而实现 Live Reload 。

2**JetBrains Goland 实践方案**

如果使用的是 JetBrains Goland 作为 IDE 的话，就可以不必使用 air 来做 Live Reload。毕竟，air 是直接使用 go build -o 编译并运行，做 Debug 调试是不支持的。Goland 实践方案的思路：**利用 Save Actions 插件监听保存操作，当保存操作被出发之后，执行 Goland 配置好的 Run 或者 Debug 选项启动应用。**

**2.1 配置应用的 Debug 或 Run 选项**

找到程序入口 main 函数，开启 Debug 或者 Run 选项，根据弹出框配置好应用启动必要的参数，可参考：

![](https://oscimg.oschina.net/oscnet/up-1614563f6e52ee48df50fc43cd20ac44bb2.png)

![](https://oscimg.oschina.net/oscnet/up-97034a55873a360a8ccd527f83751cd312f.png)

**2.2 下载安装 Save Actions 插件**

在插件中心，搜索并安装「Save Actions」插件。可以看到该插件的说明中，其实并不包含对 Goland 的支持。但实践过程中发现，该插件能很好的运行在 Goland 环境中。

![](https://oscimg.oschina.net/oscnet/up-64b37900eca353a5c08752b8267d84f199d.png)

**2.3 配置 Save Actions 插件**

在首选项中配置好 Save Actions 插件，主要在 「Build Actions」处选择第一步编辑好的 Debug 或 Run 选项即可，参考如下：

**![](https://oscimg.oschina.net/oscnet/up-3dbf6acf2a3a9d8871c88d361e2ddb64d30.png)**

通过上述的配置之后，在 Goland 中编辑某个文件并保存，即可自动执行 Debug 或者 Run 选项。该插件的界面上，还可以指定「不监控文件」的正则表达式，一些不必要监听的文件也可以排除在外。

自从使用 Golang 以来，写了两个小系统，感兴趣的朋友可以来看看开源代码，非常欢迎吐槽。

**ohUrlShortener 短链接系统** 

**https://github.com/barats/ohUrlShortener**

**RepoStats 开源代码仓库数据可视化工具**

**https://github.com/barats/RepoStats**

关注我的公众号，来和我一起聊聊开源。

![](https://oscimg.oschina.net/oscnet/up-286ec70ac8a3fabdbdcc4c4c3e88f7c6e7b.jpg)