> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg4MzUzNjA4NQ==&mid=2247493603&idx=1&sn=a63b00a10fb6c0e931e385eefa4964b9&chksm=cf475e59f830d74f86b7d83213633adbc81cf24a6188c803ad18ca1509d46f5b2fc8883189d8&mpshare=1&scene=1&srcid=0616o7BluAlzONPLtDTHOXpg&sharer_sharetime=1623824219104&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

什么是 springboot
--------------

> **Spring Boot** 是由 **Pivotal** 团队提供的全新框架，其设计目的是用来简化新 **Spring** 应用的初始搭建以及开发过程。

> 该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，**Spring Boot** 致力于在蓬勃发展的快速应用开发领域 (**rapid application development**) 成为领导者。

![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxVtW4R0VAsNTRSAecQgXMd5mNr6mcPoQ2icLrumvqYJdnFUXuK54Sic5aw/640?wx_fmt=png)

> **springboot** 是构建在 **spring framework** 之上的，而 **spring cloud** 的基础又是 **springboot**。所以如果后期需要走微服务线路，那 **Spring Boot** 是必不可少的路口。

学 springboot 需要什么技能
-------------------

首先你得懂基本的 **java** 语法，了解 **spring** 基础的框架体系，有这些就足够了。

### 环境准备

1.  jdk1.8 及以上
    
    > 下载地址：`https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html`
    
    下载完成后一直下一步下一步，直到完成即可。使用键盘 **WIN+R** 键打开 cmd 窗口，输入`java -version`，如下图，证明你 jdk 安装成功了。![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxVXuibIHcIK2MMkDnXTYQOPmwhX7cXHM9KAskqX3nsQKb9pHsBZ36490A/640?wx_fmt=png)
    
2.  maven 3.2.5 及以上
    
    > 下载地址：`http://maven.apache.org/download.cgi`
    
    下载后解压，并且配置环境变量 **MAVEN_HOME**，变量值是你解压后的 maven 的路径，最后在 path 变量的值上加上：**%MAVEN_HOME%\bin** 即可。
    
    在 cmd 下输入 mvn -v 如下图，证明你 maven 安装成功了![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxVqJxrpILeA2pnQwWnKqRk4NgJh6ibg2TyWKYZuCCvFbTmsOEiarpgG5yw/640?wx_fmt=png)
    
    安装完 maven 后理论上是可以直接用了，但是国外的镜像网络通过中转到国内访问的速度（你懂的），所以我们可以修改镜像源，在 maven 解压目录下找到 config 文件夹中的 setting.xml 文件，编辑添加如下内容：
    

3.  **idea** 或者 **myeclipse** 均可（本系列课程全部基于 idea 讲解）
    
    下载了 idea 后，在 file-》settings 中搜索 maven，将你刚解压的 maven 目录填上![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxVw4onJcSiahJuzbfkILokfWYh55EQOJMRfQ82KtTpyvx7PzHic9TIppqw/640?wx_fmt=png)
    
    到此，环境就准备就绪了，下面大家拿起 “毛笔” 准备写 hello world。
    

hello world
-----------

打开 idea，`file`>>`new`>>`project![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxViceMla0PeyWuCqvFsd322icwRibtHibibMPgyd38zzPXiaL4sfquqGSuod7g/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxVPf1SBJ8Ja0aAjcIFYx9bGX7UN7eIfOcEATPjLibHNyaychxfV4A8pdQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxV8kudsjY7j5yuKibgyzuxS5EOibH6FibgsyVtl6tY4grSPNBw4x4ZKoNaw/640?wx_fmt=png)`

然后一路 next 直到完成打开项目如下图：![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxViaUYvHD0C8icoJqnjMhd7BibEfC88ZsWr6caPNR8cBJxd3AYBn6b35wGw/640?wx_fmt=png)

注意右下角会出现 maven 引入 jar 包的提示，可以选择第一个（每次需要引入 jar 都会提示）或第二个（下次就不会询问你，直接引入了）都可以。PS: 第一次引入 jar 包的时候会有点慢，稍等即可。下载完后，我们看下目录结构如下：![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxV1tE9ap8afwBp4NW7sUevFUy1nBKCYyUNSymZI1dZSict0gicibsFvP5RQ/640?wx_fmt=png)

我们在 Lesson1Application.java 中添加如下代码：![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxVh619ZvgHFZS6KvaJSVdBo8c12iaMPibJ8TyiauDBPbdjPLx0IxnOeicnmg/640?wx_fmt=png)

然后运行

![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxVGD5ytib5icaAicRpZ7WEcIf6ot6f2lqTVYcrU5Ij4ichgtudVLYJncC1BQ/640?wx_fmt=png)

当控制台打印出：（从控制台可以看出服务在`8080`端口启动)

![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxVIquaXwnsFewUib4KiawOOzKux83NJB5TZZf8HRZ1ibMVCcJnN6ILkblHw/640?wx_fmt=png)

然后在浏览器输入地址：`http://localhost:8080/helloworld` 可以看到：

  
![](https://mmbiz.qpic.cn/mmbiz_png/Ts4QibG8CPiba0zTicpdyMTp4iaVrZvS8KxVe7OKhKFv5iawUtO0k0ibFSRrGwMD62hno0SjiaRnZbyV73PnOsCn2eHEQ/640?wx_fmt=png)

> 恭喜你：用 “毛笔” 写的第一个`hello world`运行成功！

有兴趣入群的同学，可长按扫描下方二维码添加微信