> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/demingblog/p/14598676.html)

换了新的 m1 芯片的 Macboot pro, 配置环境如下：

> MacBook Pro (13-inch, M1, 2020)  
> 芯片：Apple M1  
> 系统版本：macOS Big Sur  
> jdk:  
> openjdk version "1.8.0_282"  
> OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_282-b08)  
> OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.282-b08, mixed mode)

然后启动 springboot 项目特别慢，我起初以为是 idea 和 m1 芯片的兼容问题，于是下载了针对 m1 芯片的 idea 版本，发现启动还是慢。  
仔细看了下启动日志，发现了这么一行：

> InetAddress.getLocalHost().getHostName() took 5008 milliseconds to respond. Please verify your network configuration (macOS machines may need to add entries to /etc/hosts).

惊呆了，`InetAddress.getLocalHost().getHostName()` 这一句都能耗时超过 5 秒，于是执行下面的操作：

```
1. 在命令行执行hostname，得到主机名
2. sudo vim /etc/hosts 加入 127.0.0.1 yourhostname
```

再次启动，发现没问题了。问题解决！  
继续去 stackoverflow 搜了一下，发现原来很多人都碰到了这个问题，提问都这个哥们这一句花了 30 秒，更吓人。地址在这里：  
[https://stackoverflow.com/questions/33289695/inetaddress-getlocalhost-slow-to-run-30-seconds](https://stackoverflow.com/questions/33289695/inetaddress-getlocalhost-slow-to-run-30-seconds)

在这里差一条 Git 的问题：  
gitee 里的仓库配置了 sshkey 之后，执行 ssh -T "git@gitee.com" 通过了，但是无法 push 代码，提示错误：

```
remote: You do not have permission push to this repository
fatal: unable to access 'https://gitee.com/xxx/xx.git/': The requested URL returned error: 403
```

ssh -T 都通过了，提示我没有权限，Git config 设置的也没问题，查了一通没解决，算了，弹窗让我我自己输入用户名密码吧。  
执行如下命令，可以使得 push 代码的时候提示输入用户名密码：

```
git config --system --unset credential.helper
```

本文作者：逃离沙漠

本文链接：https://www.cnblogs.com/demingblog/p/14598676.html

版权声明：本作品采用知识共享署名 - 非商业性使用 - 禁止演绎 2.5 中国大陆许可协议进行许可。