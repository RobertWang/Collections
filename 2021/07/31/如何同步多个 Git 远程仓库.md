> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488267&idx=2&sn=5233c1a5644c287067239fee16db1004&chksm=ce0da488f97a2d9e015f453d0950a8b3df1750347a697190af648865d1ab8b385a7d9c1f37f1&scene=21#wechat_redirect)

> 如何同步多个 Git 远程仓库

作者 | taadis

链接 | https://my.oschina.net/taadis/blog/3073220

日常需求

以前源码是托管在 github 的, 现在想要同步托管在 gitee, 一做备份分发, 二方便国内下载使用 (网速可观), 三防特色墙...

方式一：使用 gitee 的强制同步
------------------

之前在 github 托管了这么一个项目 mirrors-in-china,    后来国内出了 gitee, 那么想着把项目同步一份到 gitee, 方便大家查看...    正巧 gitee 提供强制同步功能, 方便操作..

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsjTXMwEiaS4NdjbHEwnbbzyOA1vFibDh2UCqXwVXwkRB2zKrhlRrTmRYKI2njNzZNKyokyUMZ7KHTnw/640)

我还是只用维护 github 那份源码, gitee 这边没忘记的话, 手搓点击下强制同步按钮即可.

但是容易忘记, 造成两边不完全同步.

不过我这个项目本身就非常简单, 这点同步时差完全没大问题, 够用, 并且没有其他任何多余的操作.

方式二：手搓 push 多次
--------------

换另一个项目来说, 我之前在 github 托管了这么一个项目 GlobalScanner.Sdk,    应广大小伙伴需求, 希望把项目在国内同步一份, 方便下载 / 参考 / 使用.

那么不外乎就是配置多个远程库地址, 多次推送咯, 那么我们先来看看现有远程库的情况:

```
$ git remote --verbose
origin  git@github.com:taadis/GlobalScanner.Sdk.git (fetch)
origin  git@github.com:taadis/GlobalScanner.Sdk.git (push)
```

可以看到目前仅有`git@github.com:taadis/GlobalScanner.Sdk.git`这个远程库地址.

我们来加一个 gitee 的远程地址, 首先在 gitee 建好同步仓库, 然后我们在本地添加一个新的远程库地址:

```
$ git remote add giteeorigin git@gitee.com:taadis/GlobalScanner.Sdk.git
```

添加完成后我们查看一下:

```
$ git remote --verbose
giteeorigin     git@gitee.com:taadis/GlobalScanner.Sdk.git (fetch)
giteeorigin     git@gitee.com:taadis/GlobalScanner.Sdk.git (push)
origin  git@github.com:taadis/GlobalScanner.Sdk.git (fetch)
origin  git@github.com:taadis/GlobalScanner.Sdk.git (push)
```

可以查看到以下 2 个远程库地址:

*   giteeorigin: 是我们新加的 gitee 的远程库地址
    
*   origin: 是我们之前在 github 的远程库地址
    

接下来同步:

```
git add .
git commit -m "add gitee"
git push -u origin master
git push -u giteeorigin master
```

有链接有真相:

*   github: add gitee
    
*   gitee: add gitee
    

比之前多个一次`git push`操作... 其他和之前没有太大区别... 没有更多的心智负担.

但是经常容易忘记...

方式三 最多跑一次
---------

不想着法偷懒的 coder 不是好程序员, 秉承 "最多跑一次" 的理念, 让我们试试怎么一次 push 统统搞定.

在本地 git 仓库里找到这个文件`.git/config`, 内容如下:

```
[core]
  repositoryformatversion = 0
  filemode = false
  bare = false
  logallrefupdates = true
  symlinks = false
  ignorecase = true
[remote "origin"]
  url = git@github.com:taadis/GlobalScanner.Sdk.git
  fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
  remote = origin
  merge = refs/heads/master
[remote "giteeorigin"]
  url = git@gitee.com:taadis/GlobalScanner.Sdk.git
  fetch = +refs/heads/*:refs/remotes/giteeorigin/*
```

改为如下:

合并 2 个 remote 配置

```
[core]
  repositoryformatversion = 0
  filemode = false
  bare = false
  logallrefupdates = true
  symlinks = false
  ignorecase = true
[remote "origin"]
  url = git@github.com:taadis/GlobalScanner.Sdk.git
  url = git@gitee.com:taadis/GlobalScanner.Sdk.git
  fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
  remote = origin
  merge = refs/heads/master
```

上面这个手动配置是为了更好的说明而已, 其实可以用以下命令简化操作, 在 origin 节点下补充了一个新的远程地址.

```
$ git remote set-url --add origin git@gitee.com:taadis/GlobalScanner.Sdk.git
```

看看补充后的远程地址情况

```
git remote --verbose
origin  git@github.com:taadis/GlobalScanner.Sdk.git (fetch)
origin  git@github.com:taadis/GlobalScanner.Sdk.git (push)
origin  git@gitee.com:taadis/GlobalScanner.Sdk.git (push)
```

注意看后面的 (fetch)(push), 相信你会明白点什么.

然后我们可以继续这样使用来实现 github & gitee 的同步推送和分发:

```
git add .
git commit -m "github & gitee 同步推送和分发"
git push origin master
```

有链接有真相:

*   github: github & gitee 同步推送和分发
    
*   gitee: github & gitee 同步推送和分发
    

可以看到, 使用上和最初没有任何区别, 只是多配置了一次, 算是实现了 "最多配 (跑) 一次".

总而言之，几种方式, 各取所需。
----------------

推荐阅读

_**1.**_ [为什么要放弃 JSP ？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488247&idx=1&sn=8f40c70e93f789211b262735ff5a0bb3&chksm=ce0da574f97a2c62d213b3a4bac5bb72dd3d34764579be08ba78836f6cb8e3205faaa12cd980&scene=21#wechat_redirect)

_**2.**_ [你还在从零搭建项目 ？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488238&idx=1&sn=253a237ec16ac1bea64a37ddc934867f&chksm=ce0da56df97a2c7b82c8849142c365a1ec01f010aa4d7982b842bd5a26c6ef312196d4c72e53&scene=21#wechat_redirect)

_**3.**_ [外行人都能看懂的 Spring Cloud](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488222&idx=1&sn=bbf8d140906aaf26c88d9b1dfb99d654&chksm=ce0da55df97a2c4b56f2cf54be5e3f18a7d8e87177311276d827bd7c24d0515f37bbcea4bcfd&scene=21#wechat_redirect)

_**4.**_ [后端接口都测试什么？怎么测？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488152&idx=2&sn=43928c7617494fa579a2624e4eb34b19&chksm=ce0da51bf97a2c0d900ba8e51e15e2824e6c675a90c23ed6c7145a78640f497b988010006c31&scene=21#wechat_redirect)

_**5.**_ [IntelliJ IDEA 2019 从入门到上瘾](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488138&idx=1&sn=aa910abad756f24a8584316882a930aa&chksm=ce0da509f97a2c1f1e8935deeeca7f5c382ec2c824b6d0ea47817439722a873ea1e585f65e18&scene=21#wechat_redirect)

_**6.**_ [如何分析一条 SQL 的性能 ?](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488106&idx=2&sn=51ea9f478004170e5c10c428b4a405c8&chksm=ce0da5e9f97a2cffe0b7af071e7fbce5a8b6fee8d0e8a485cbf114dd0cc39dc1469c0c88e4ea&scene=21#wechat_redirect)

_**7.**_ [还不懂 Java 中的多线程 ？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488085&idx=1&sn=66d41311ea6c4eeb4418a007e5d67b27&chksm=ce0da5d6f97a2cc0e4ea693f403adbd11614eedc9fbb2e3e7e6c51c3b9d03c60e8844ec6eb9d&scene=21#wechat_redirect)

_**8.**_ [为什么老外不愿意用 MyBatis？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488085&idx=3&sn=e107207958249561e7c6590e8b3abe92&chksm=ce0da5d6f97a2cc02a045987af65f05a6d28a02841280fff4179426fd61812c1e6db95a6e1db&scene=21#wechat_redirect)