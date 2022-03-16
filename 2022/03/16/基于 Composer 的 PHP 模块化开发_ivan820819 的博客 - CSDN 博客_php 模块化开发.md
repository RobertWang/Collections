> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/ivan820819/article/details/76130669?locationNum=5&fps=1)

转载自：https://zhuanlan.zhihu.com/p/27943241

[安正超](https://www.zhihu.com/people/overtrue)  

基于 [GitHub](https://so.csdn.net/so/search?q=GitHub&spm=1001.2101.3001.7020) 或者其它平台托管的开源项目的引入大家应该都已经非常熟悉了，但是公司内部项目的模块化应该怎么做呢？这或许是不少朋友头疼的问题。

![](https://pic3.zhimg.com/v2-fb823b6679a5fae047dff8ff50c6cd4a_b.jpg)

我们先聊聊 PHP 模块化开发演进的过程，在没有 GitHub 之前，我们大家获取与分享代码的方式主要是博客，国内的 CSDN 或者博客园还有很多很多，大家都是从文章内复制到自己项目里面使用。现在看起来真的是相当原始粗暴，但是那个时代也没有太多可选的方案。导致的现象就是一段代码在 N 个项目里出现，可能见得最多的就是获取客户端 IP 的那几行了，在互联网上不止出现了几万遍。现在很多项目里都还是这段：

![](https://pic2.zhimg.com/v2-85263a5f6bb14562df7d56a426010195_b.png)

是不是很熟悉？

这种引入代码的方式有很多弊端：比如不安全，因为很多人是直接复制粘贴就用上了，根本没花时间去考证它是否真的是安全的。另外一个问题就是不同步，你今天在别人那里复制过来就用上了，后来作者发现了 bug 并修复更新了文章也不会通知你，你也不可能记得这段代码来自哪里去检查更新。

![](https://pic3.zhimg.com/v2-68d457c525fda3c1edee523ce14e708a_b.jpg)

在没有 [Composer](https://so.csdn.net/so/search?q=Composer&spm=1001.2101.3001.7020) 之前我们是如何引入代码的呢？除了上面说的复制粘贴以外，在 PHP 中还有 pear，不过自从用过两次我就再也不用它了，一种说不出来的感觉。

不信你可以找一些旧的项目看看，在没有 Composer 之前的项目中，你会发现大量的重复代码，以及各种花样的组织格式，各种规范的写法。这也是 Composer 诞生的原因之一。

![](https://pic4.zhimg.com/v2-a425750201ed63c074de9bf4c1ccb25f_b.jpg)

Composer 给我们带来了诸多的好处：

*   模块化，降低代码重用成本
*   统一的第三方代码组织方式
*   更科学的版本更新

这三个是比较重要的特征了，基于 GitHub 的共享代码方式解决了传统引入方式带来了各种问题。

我们先来了解一点 Composer 基础。

![](https://pic4.zhimg.com/v2-eaf8e3cbe80f8c5673a328a6e8399903_b.jpg)

Composer 的实现结构相对比较简单，[http://Packagist.org](http://link.zhihu.com/?target=http%3A//Packagist.org) 是 Composer 官方数据源，它的数据基于 GitHub 等代码托管平台，你在本地使用 Composer 命令行工具，基于 [http://Packagist.org](http://link.zhihu.com/?target=http%3A//Packagist.org) 的数据信息安装与更新依赖。本地安装 Composer 非常简单，主要有以下几种方式：

![](https://pic2.zhimg.com/v2-1b187901da23c20903515a133af40969_b.jpg)

新手同学需要注意的是，这里一定要确定 composer 安装目录在环境变量 $PATH 内才能全局使用 composer 命令。

那接下来我们聊一下如何创建一个 Composer 包。

![](https://pic1.zhimg.com/v2-03affee0fe572adb32f222f3254047ec_b.jpg)

步骤很简单，创建目录，然后在目录内使用命令 `composer init` 按照提示完成包的初始化。

接着就是完成你的代码编写，然后在 composer.json 文件配置你的引入方式等信息。然后我们如何对已经写好的代码进行测试呢？

![](https://pic2.zhimg.com/v2-ef14c401d0c9c907e7d0963fe7a1aa35_b.jpg)

我们需要在其它任何地方建立一个测试项目（不要在刚才创建的包目录就可以），比如这里我们创建一个叫'my-package-test' 的目录，然后在目录里 composer init 完成项目初始化。接着就是声明项目依赖，我们这里要依赖的就是刚才建立好的包，由于我们的包还没有发布到 packagist，所以是无法直接 composer require 来安装的，我们需要告诉 composer 从哪里加载我们的包信息：

```
$ composer config repositories.foo path /Users/overtrue/www/foo/bar


```

我们通过这个命令在 composer.json 中 repositories 区块添加了一个项目源。

然后我们添加包依赖：

```
$ composer require foo/bar:dev-master -vvv


```

这样就完成了包的安装，你会发现这样的安装方式它只是创建了一个软链接到包目录，所以，你在测试的时候就可以直接在 vendor/foo/bar 下修改代码，这样就加快了你的开发速度。

更多细节这里你就自己去研究了，我们来看看 composer.json 文件：

![](https://pic1.zhimg.com/v2-488a649c8d0978688e68b0a9030e760c_b.jpg)

我们最需要关心的就是图里上面的三个部分了，包名、依赖、以及自动加载，是必不可少的部分。

刚才我们提到了包的安装，安装依赖包的方式主要有以下两种：

![](https://pic1.zhimg.com/v2-7e3b1bb38ba84886d64aee6b42e067e0_b.jpg)

手动方式是不太推荐的，容易写错，比如后面多一个逗号之类的，不过你可以写完以后使用以下命令来验证：

```
$ composer validate


```

更新依赖就非常简单了：

![](https://pic2.zhimg.com/v2-9f9b70f829f0f7644c6cd718cbb426e1_b.jpg)

虽然在上一篇文章我们已经讲了语义化版本，这里再提一次：

![](https://pic4.zhimg.com/v2-04f8ab18ffb320a5bb1e2fabe71cab37_b.jpg)

![](https://pic2.zhimg.com/v2-6e419f125bc09dfe08d0ef5384faa4f5_b.jpg)

我们在依赖一个包的时候，很多同学对于依赖的版本一直处于蒙逼状态，那看完下一页你就恍然大悟了，首先是两个符号："~" 与 "^"

![](https://pic2.zhimg.com/v2-1fb7831bcce55ae8221fcf694f658ef1_b.jpg)

接着是版本号的范围的各种写法：

![](https://pic3.zhimg.com/v2-5297855551ee5badd3cca8e7aa1dd0a6_b.jpg)

还有包含稳定性的标识：

这里需要说一下生产环境最重要也一直是好多同学不清楚的一个东西：**版本锁定**，很多人在纠结，要不要把 composer.lock 上传到代码库啊。我可以给你一个特别简单的判断方法：

**如果你的代码是一个项目，就上传，如果是一个工具包，给大家用的，就别上传。**

![](https://pic4.zhimg.com/v2-acb250980355d34ce18cb09cd43c8f4b_b.jpg)

在已经存在 composer.lock 的目录执行 composer install 的时候，是不会分析包依赖的，它只是按 composer.lock 中描述的下载地址直接下载，所以会快很多，而且版本号是具体的。那怕包已经发了新版，只要 composer.lock 没动过，它就会按 composer.lock 里的版本来安装。composer update 时会更新 composer.lock，所以不要乱用 composer update。

包开发好了怎么发布？开源的方式是这样的：

![](https://pic4.zhimg.com/v2-3027a262741ef82083d6c73c28357d13_b.jpg)

最后一句请酌情考虑。

另外一种发布方式是闭源，公司内部用的包，上传到 GitLab 或者其它私有的代码托管平台，有两种玩法：

1.  最容易的玩法，在 composer.json 中添加 repositories 直接用 vcs 指定代码库地址

这样做有一个缺点，当你的包很多的时候，你全都得在 composer.json 中加上才行。

2. 自已架设 Packagist 类似的服务，Packagist 官方提供了两款： toran，收费，开源方案是 Satis，不过它偏手动一些，自己酌情选择即可。

私有包有一个点需要注意：授权，私有包肯定都是需要授权才能访问的，这里由于方案不太通用，大家根据自己的场景来解决就好了。

另外，有一些痛点不晓得啥时候能够解决：

![](https://pic3.zhimg.com/v2-0d85631bb7c78aac03f3a6aa23386f56_b.jpg)

![](https://pic4.zhimg.com/v2-b9d674269430f006073d38f9c61657ab_b.jpg)

![](https://pic4.zhimg.com/v2-47dc4bb1407d09687c03b70aa53583a7_b.jpg)

好在 Laravel China 已经为了我提供了国内目前最稳定最好用的镜像源，[Composer 中文镜像 / Packagist 中国全量镜像正式发布！](http://link.zhihu.com/?target=https%3A//laravel-china.org/topics/4484/composer-mirror-use-help)

最后总结一下：

微信并不适合聊一些代码细节的东西，我更多的倾向于提供思路。

在 PHP 现代开发中，Composer 已经是离不开的东西了，它的确加快了我们的开发速度节省了开发成本，如果你还在纠结用不用 Composer，那你真得反思一下了。

本文标题是模块化开发，内容主要介绍了包的创建与测试，以及公有包与私有包的发布方案。但是无法帮你解决，如何拆分项目这类问题，这得基于你的长期经验积累，但是有一些经验可以分享一下：

1.  **不要过度设计**，很多自以为很 NB 我不把学到的东西用上就是不爽的同学，上来就分库分表，uuid 做主键之类，项目运营了好几年一个表还没到 100 万条记录，也是够厉害的。
2.  **不要过早设计**，真正 NB 的架构是演进而来的，不是前期设计出来的，当然不是说完全不需要设计哈，恰当的根据实际情况来就好，不要立项就把 “千万”、“亿级”、“百亿” 这些单位挂在嘴边，也许到你项目倒闭那天你都没到过任何一个量级。随着项目逐渐改进即可。
3.  **优先关注成本**，很多同学以为是性能，No! 技术团队真正的成本是人力，所以开发效率才是优先需要关注的。上次文章在发在知乎有人就强制把我的 “不要首先关注性能” 解读成了“性能不重要”，也是够厉害的，语文也许跟养殖场动物学的。数学正常一点的人都会算，一台服务器多少钱？一个技术员工多少钱？服务器你花 20 万是永久资产，一个员工 20 万呢？半年工资？一年工资？