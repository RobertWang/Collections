> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247507075&idx=3&sn=7068a1ff207f4cd8832cc41e1fde726f&chksm=e87979f9df0ef0efe92d7d53e7cba1f83b1c9aabf6d66dbef8fa1c31fc67efa290e18f3623e1&mpshare=1&scene=1&srcid=0619itCo7P82fsGB1ckyCx74&sharer_sharetime=1624113300187&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> **作者** |  i 校长
> 
> **地址** |  mp.weixin.qq.com/s/VhtIcuBaSGEHcuiG95myeg

简述
==

环境要求
====

*   Flutter  
    需要切换 beta 版本来支持 web 开发  
    环境搭建跳至之前博客：Flutter 系列之环境搭建
    
*   Node  
    下载跳至：下载 | Node.js  
    入门跳至：指南  
    环境配置：Node.js 安装配置 | 菜鸟教程  
    脚手架：Express 生成器  
    具体操作步骤请往下看
    
    Flutter 项目创建
    ------------
    
    假装你已经搭建好环境
    
*   step 1  
    打开终端，切换 Flutter 分支
    
    ```
    flutter channel beta
    flutter upgrade
    flutter config --enable-web
    flutter devices
    Chrome     • chrome     • web-javascript • Google Chrome 78.0.3904.108
    ```
    
    一行一行执行命令，最后看到 Chrome，祝贺你成功了。
    
*   step 2  
    打开 Android Studio![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6ELunccALhUNVLI3X1Hmr6WNE4QcfclCS3mps00vibrQIREr55x5kiaic8Q/640?wx_fmt=png)  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6E6ELkfHEpzRl55bobfKLkycIfdMGt144KkoRgTBAnI6VpibNDcCDWTAg/640?wx_fmt=png)  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6EibTQIt9Ted5ICuH1WwQoC0RJkrurmFCFV7sJ4XibibQR62n0N8PvnUvIA/640?wx_fmt=png)  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6ER9Sqvnq137wzHpXCOWFickHO8GuCOf5wVe019U4tVoUWLuO55lySHeA/640?wx_fmt=png)  
    项目名字、描述简单修改一下，next 下一步  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6EUGBBfHa7yiaFDKUicWL0BqVYYQf5xbbIIRzHUfic0ibqMcCeafOPnqM2Kg/640?wx_fmt=png)  
    修改一下包名，然后 Finish，需要等待一会儿。  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6Evp8Y3Uya2aGlAHo4mpJ6lLZSbFemlOCuITRlOPSL3z08iasYm3yGbAQ/640?wx_fmt=png)  
    项目创建成功了。这里就到这，后期博客慢慢介绍每次开发的细节。
    
    Node 项目创建
    ---------
    
    我们直接打开 Flutter 项目的 Terminal![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6EDVULIJkID4hiadEyNxxNR1ICcX6w6Emrzl2efWbicwGrghN8n7ytcNWQ/640?wx_fmt=png)
    
    ```
    mkdir node
    mkdir server
    cd node/server
    ```
    
    进入 server 目录，现在你的 node 环境应该也可以了吧，好开始用 Express 生成器生成项目
    
    ```
    npm install express-generator -g //安装好了略过
    express --view=pug myapp
    ```
    
    修改 myapp 为你自己的项目名。执行完你会看到![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6ExibBIq8m1f38Pib2xdrzHFgmrswIVaIX9RMSrKuQbaAwtqjiaogJfJgBw/640?wx_fmt=png)  
    接下来
    
    ```
    cd myapp
    npm i
    npm start
    ```
    
    浏览器试下 http://localhost:3000 看到如下就 ok 了  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6EicnyYuK2KdMtvw9g9Uq2uNzvWibsrlTzLsZkkuuSR9HC5TZzeYuPDdrw/640?wx_fmt=png)
    
    开始项目关联
    ------
    
*   step 1  
    在 Flutter 项目中执行
    
    ```
    flutter build web
    ```
    
    构建 web 包，最终会在 build 文件夹下生成 web 包，web 包下就是网站的相关文件。![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6EBFP8zJOqzQvLX7cLMCJaNd2hiazwOBGFFLGfeaJe94cKPjzkJ1QGuMw/640?wx_fmt=png)
    
*   step 2  
    copy web 包下的文件到 node 项目的 public 文件下  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6EgQATcUhL8tdCqW0FhZl5LAm2pqzC8yHFECXkMVLB7GKF2owmR4foJw/640?wx_fmt=png)  
    我创建了一个 public_flutter_web，为了是以后文件区分，也建议你做一样的操作
    
*   step 3  
    改造 express，因为默认 express 是展示 views 包下的网页的，而且默认不是 html 实现。将下图中文件全部删除即可  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6EptibBPL15IIwb9GzMnBibJM6B9XSeicjxOR6s9cgwRQtvJH14C0j7eJaQ/640?wx_fmt=png)  
    打开 app.js 文件，删除 delete 标记部分，添加 add 标记部分  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6EULK8iaXqPBibcB7fzxThItL7gkGRL0IqEgcsTicAJE51fjMKiaQ7rldCIw/640?wx_fmt=png)
    
*   step 4  
    保存修改，重新将服务 npm start，再打开 http://localhost:3000  
    看到如下：  
    ![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6EgCL6P5yonOa2r6GBhRf02diaFYDeq1CYzzfgMADCvzOQwqveciaSWxsw/640?wx_fmt=png)  
    大功告成，这样就行了吗，nono，对于一个懒惰的人来说，我们要写一些脚本，辅助项目自动构建。
    
*   step 5  
    由于 node 项目目录太深，在命令行运行也很麻烦，我们写个 shell 脚本，来帮我搞定。在 flutter 项目根目录创建 bin 文件夹，用来放置我们的脚本![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQnw4zH55GkbADOm8b5Ds6E7OicxRcqawDdWtm8GfROhRofCq3771ribQ53ttuNA87wxGmbsLiaW14LQ/640?wx_fmt=png)  
    右键 New File 命名为 test_start_node.sh，内容如下
    
    ```
    #!/usr/bin/env bash
    node node/server/bin/www
    ```
    
    也很简单。这个脚本就是辅助我们开启 node 服务。当然我们还会有 flutter 项目构建的一些脚本，自动 copy 文件到指定目录等等，这些之后慢慢补全哦。
    
    最后
    --
    
    将代码上传至 github
    
    ibaozi-cn/**flutter-jetpack**
    =============================
    
    最后的最后
    -----
    
    登上你的云服务器，通过 git 将项目下载到服务器上，这里我们需要工具辅助我们服务部署  
    我选择 pm2+nginx 来将我的服务启动起来  
    pm2：环境搭建  
    nginx：环境搭建  
    这里不详细说了，网上有一片大海，需要你去浪。有问题的留言我，我可以协助你。  
    最终通过 pm2 和 nginx ，项目完美运行  
    jetpack.net.cn，没错你看到的是 jetpack.ibaozi.cn，哈哈，域名还没下来，先用了之前的 ibaozi.cn, 后面我们会迁移到 jetpack.net.cn。
    
    总结
    ==
    
    下期我们就开发 Flutter 主页，遇到什么，需要借助什么，怎么写，为什么这么写，我们将在未来的博客中，带你一步步实现一个完整的网站，随我写下去。