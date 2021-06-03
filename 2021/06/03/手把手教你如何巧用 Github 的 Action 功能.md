> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bilibili.com](https://www.bilibili.com/read/cv9190554/)

概念
==

GitHub Actions 是 GitHub 于 2018 年 10 月推出的持续集成服务。

那么何谓持续集成呢？

持续集成
====

持续集成（Continuous integration），也就是我们经常说的 CI。它是一种软件开发实践，可以让团队在持续的基础上收到反馈并进行改进，不必等到开发后期才寻找和修复缺陷，常运用于软件的敏捷开发中。Jenkins 就是我们常用的持续集成平台工具。

理解了持续集成的概念之后，下面我简单讲一下使用持续集成的好处：

*   提高效率，减少了重复性工作：一些重复性的工作写成脚本交给持续集成服务执行。
    
*   减少了人工带来的错误：机器通过预先写好的脚本执行犯错的几率比人工低很多。
    
*   减少等待的时间：一套完备的持续集成服务涵盖了开发、集成、测试、部署等各个环节。
    
*   提高产品质量：很多大公司在代码提交后都会有一套代码检视脚本（俗称门禁）来检查代码的提交是否符合规范，从而从源头遏制问题的产生。
    

Actions
=======

相比较持续集成这个大概念，GitHub 推出的 Actions 就显得非常轻量和巧妙了。Actions 就相当于持续集成中的某个特定功能的脚本，通过多个 actions 的自由组合，便可实现自己特定功能的持续集成服务。

同时，Github 为了方便大家使用 Actions，还专门做了一个 Actions 市场, 真的是非常方便！

![](http://i0.hdslb.com/bfs/article/46dc21895f94a1b3064ff690652192b75aafdea2.png@1320w_658h.webp)

GitHub Actions 有一些自己的术语:

*   1.`workflow（工作流程）`：持续集成一次运行的过程，就是一个`workflow`。
    
*   2.`job（任务）`：一个`workflow`由一个或多个`jobs`构成，含义是一次持续集成的运行，可以完成多个任务。
    
*   3.`step（步骤）`：每个`job`由多个`step`构成，一步步完成。
    
*   4.`action（动作）`：每个`step`可以依次执行一个或多个命令（action）。
    

workflow 文件
===========

GitHub Actions 的配置文件叫做`workflow`文件，存放在代码仓库的`.github/workflows`目录, 如下图所示：

![](http://i0.hdslb.com/bfs/article/a3aaf839cbe60adf4e1c5f4919e5188f1c61a41b.png@1320w_578h.webp)

`workflow`文件采用`YAML`格式，文件名可以任意取，但是后缀名统一为`.yml`，比如上图的`package.yml`。

`workflow`文件的配置字段非常多，详见官方文档 。下面是一些基本字段：

*   1.`name`: workflow 的名称。如果省略该字段，默认为当前 workflow 的文件名。
    
*   2.`on`: 触发 workflow 的条件，通常是某些事件，例如：`release`、`push`、`pull_request`等。详细内容可以参照 官方文档 。
    
*   3.`jobs`: workflow 文件的主体内容，表示要执行的一项或多项任务。
    

*   `jobs.<job_id>.name`: `job_id`是任务的 id，`name`是任务的描述。
    
*   `jobs.<job_id>.runs-on`: `runs-on`运行所需要的虚拟机环境, 它是必填字段。
    
*   `jobs.<job_id>.needs`: `needs`指定当前任务的依赖关系，即运行顺序。
    
*   `jobs.<job_id>.steps`: `steps`指定每个任务的运行步骤，可以包含一个或多个步骤。
    

Actions 的应用
===========

如何使用 Action 发布 flutter 插件
=========================

之前我写过一篇《Flutter Plugin 插件开发填坑指南》 ，讲的就是如何开发一个 flutter 插件并进行发布。但由于我们发布插件到 flutter 插件平台 需要访问外网，而且还需要给命令终端设置代理，所以每次的发布都非常的麻烦。

一个偶然的机会，我就在 Action 市场中发现了一个`publish-dart-flutter-package`插件，可以一键把自己的插件发布到 flutter 插件平台 ，脚本如下:

`name: Pub Publish plugin  
on: workflow_dispatch  
jobs:  
  publish:  
    runs-on: ubuntu-latest  
    steps:  
      - name: Checkout  
        uses: actions/checkout@v1  
      - name: Publish  
        uses: sakebook/actions-flutter-pub-publisher@v1.3.0  
        with:  
          credential: ${{ secrets.CREDENTIAL_JSON }}  
          flutter_package: true  
          skip_test: true  
          dry_run: false  
`

当然你也可以参考我的 flutter_xupdate ，它就是利用这个 Action 进行发布的。

这里我们可以看到，我们定义的触发条件是`workflow_dispatch`，也就是手动触发任务执行：需要我们点击 `Run workflow` => 选择 `master` 分支 => 点击 `Run workflow`，入下图所示：

![](http://i0.hdslb.com/bfs/article/3056b5b62b7f54a088268030378adfb123d36c04.png@1320w_460h.webp)

这里我们注意到定义了一个`secrets.CREDENTIAL_JSON`常量，也就是我们的 google 账号认证证书，这里需要我们在项目的`Settings` => 选择 `Secrets` => 点击 `New repository secret`来创建一个属性名为`CREDENTIAL_JSON`的常量。对应的值你可以到你的用户 Home 目录下的`.pub-cache`文件夹下找到`credentials.json`文件。

![](http://i0.hdslb.com/bfs/article/08d59531874dc4ea2950c16c6f6acd42e587a219.png@1320w_670h.webp)

下图是我执行了一次发布 action 的结果，只需 2 分钟，无需 **** 工具和给命令终端配置代理，即可完成 flutter 插件的发布，真的是非常方便！

![](http://i0.hdslb.com/bfs/article/e2f3f152c69bb6c37a5dbc65f295d44a106474a6.png@1320w_406h.webp)

如何使用 Action 打包 apk
==================

作为一名 Android 开发，你有没有想过每次提交代码或者发布版本的时候，github 能够对应帮你自动打包出一个 apk？

这样你既可以省去打包 apk 的时间，还免去了应用包的管理，岂不美哉？

那么我们应该怎么做呢？下面就是我实现的一个`workflow`脚本，主要的功能就是：在提交代码或者发布版本的时候，自动构建脚本打包出 apk，同时直接上传至`Artifacts`存储。

`name: Android CI  
on:  
  release:  
    types: [published]  
  push:  
    branches:  
      - master  
    tags:  
      - '2.*'  
  pull_request:  
    branches:  
      - master  
jobs:  
  build:  
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/checkout@v2  
      - name: set up JDK 1.8  
        uses: actions/setup-java@v1  
        with:  
          java-version: 1.8  
      - name: release apk sign  
        run: |  
          echo "给apk增加签名"  
          cp $GITHUB_WORKSPACE/.github/workflows/android.keystore $GITHUB_WORKSPACE/app/android.keystore  
          sed '$a\RELEASE_STORE_FILE=./android.keystore'          $GITHUB_WORKSPACE/gradle.properties -i  
      - name: build with gradle  
        run: |  
          echo "开始进行release构建"  
          chmod +x gradlew  
          ./gradlew app:assembleRelease  
      - name : upload apk  
        uses: actions/upload-artifact@master  
        if: always()  
        with:  
          name: xupdate_apk  
          path: ${{ github.workspace }}/app/build/outputs/apk/release  
`

详细配置可以参考我的 XUpdate 中的配置。

这里我们可以看到，我们定义的触发条件是`release`，`push`和`pull_request`，触发的分支是`master`，tags 是`2.*`开头的。

整个任务主要分为 4 个步骤：

*   1.`set up JDK 1.8`： 构建 java1.8 的环境。
    
*   2.`release apk sign`：配置应用的签名。这里需要注意的是，这个地方的签名配置还是需要结合着 build.gradle 文件的配置来编写的。
    
*   3.`build with gradle`：编译构建 apk。运行`assembleRelease`命令打 release 包。
    
*   4.`upload apk`：上传 apk 至`Artifacts`。
    

最后执行的效果如下：

![](http://i0.hdslb.com/bfs/article/0315a6186556f64fcb6e18b212518ada45765d70.png@1320w_636h.webp)

如何使用 Action 来反击白嫖党
==================

> 我在做开源项目的时候，经常能够碰到一些个无名小号（白嫖党），项目看都不看就提一些没有任何价值的`issues`，然后你好心好意地回复了，他却消失不见了... 真的是让人恨得牙痒痒的！

是的，你没有看错，Action 居然还可以用来反击白嫖党！这也是之前我在逛掘金的时候偶然看到一篇文章《❌ 对白嫖怪 SAY NO !!! —— 如何在 GitHub 上阻止无耻白嫖》 发现的。

那么他是怎么做的呢？其实也很简单，就是设置触发的条件是`issues`的创建，在创建的时候去查询一下`issues`的创建者是否`star`或者`fork`了该仓库，如果满足条件则不做处理，否则将自动锁住并关闭`issues`。

当然，这位作者也是把这个非常骚的 Action 做成了一个插件，插件的地址是：https://github.com/marketplace/actions/no-free-usage-action ，使用起来非常简单。

以下是我简单使用的脚本案例：

`name: No Free usage issue checker  
on:  
  issues:  
    types: [opened, reopened]  
jobs:  
  build:  
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/checkout@v2  
      - name: Check issue actor  
        uses: fluttercandies/no-free-usage-action@v1.0.1  
        with:  
          token: ${{ secrets.GITHUB_TOKEN }}  # 由GitHub提供的临时Token，必须在此处进行传递，且必须为这个值。  
          forked: '--no-forked'  
          words:  To support our project, please file the issue after you starred the repo. Thanks! 🙂  
`

这里，我设置的触发条件是`issues`的打开和重新打开事件，设置不强制`fork`，但是需要`star`。当一只野生的白嫖党出没并在你的项目上提`issues`的时候，就会触发下图的效果：

![](http://i0.hdslb.com/bfs/article/f045869c325ea0da3a3edb4d4bcbb5a16bfa6531.png@1320w_760h.webp)

看到上图的效果，是不是感到很惊喜，很刺激？你以为你做白嫖党我就没有办法治你？哈哈，给我老实点！

最后
==

都看到这儿了，还不赶紧三连支持一下，难道你也想做白嫖党吗？

> 更多资讯内容，欢迎搜索我的微信公众号:【我的 Android 开源之旅】