> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666552971&idx=3&sn=debacb8bdfb83ddf31c3191b066af9cb&chksm=80dc9a20b7ab1336da60dc85a5314953c169e68b0c409aad5ba706abf50ef6a627c8ed4d1b24&scene=21#wechat_redirect)

【导读】围观大佬操作！php＋go怎么用？性能如何？本文详细介绍了这种开发模式的适用场景、性能等细节。  

**数据说话**
--------

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2KgthibB09L38ibH1p42jO90ibkKCqYt6jSHRwiaqSsic3921PC1gw64L3iaB2CvLicBvVt64qAmND1YyxmA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从图中可以看出RoadRunner对比Nginx+FPM，运行效率是有数量级上的提升。

**一般PHP服务器**
------------

**传统CGI协议服务器**

客户端访问某个URL地址之后，通过 GET/POST/PUT等方式提交数据，并通过HTTP协议向Web服务器发出请求，服务器端将HTTP请求里描述的信息通过标准输入(stdin)和环境变量(environment variable)传递给新建的CGI进程。处理完成后，进程立即关闭。

**Nginx + PHP-FPM模式**

现在流行的PHP web程序一般都是运行在Nginx + PHP-FPM模式下的。`PHP-FPM`就是PHP对`FastCGI`的实现。

  
master创建并监听多个worker进程，通过共享内存获取worker的状态，进而通过信号控制worker进程。

每一个worker进程就类似一个CGI进程，收到CGI请求后会执行相应的PHP文件，并把请求内容作为PHP进程状态的一部分(_GET, _POST, _SERVER等等)。

结束请求后，worker不会立刻结束，而是继续留在worker pool. 这就节省了频繁创建结束子进程的开支。

**RoadRunner**
--------------

**为什么**

现在很多PHP的企业级框架都要求你加载至少十几个文件，构造多个类并解析一些配置，以便处理简单的用户请求或查询数据库。每个任务完成后，你不得不抛弃这些代码。

收到下一个HTTP请求时，PHP-FPM会创建一个新的PHP子进程来处理这个请求，所有的文件都要重新加载一遍，即便文件可以有缓存，所有的代码也要重新运行。

如果我们可以避免对每个请求都重启一次PHP子进程，我们就可以节约很多的资源。

**基本原理**

`RoadRunner`可以看作一个升级版的Nginx + PHP-FPM. 它直接把长时运行的PHP进程作为worker, 直接对PHP worker进行监控和维护，每次收到http请求时，就发给php worker来处理。这样，我们就不再需要对每个请求重启一遍PHP了。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/IgylNib7ZE2KgthibB09L38ibH1p42jO90ibSJyxgpGdIMRzTRneUNHphVkFmyr8YWjg7w6mgmP0DRg3wJSsqiay8Vw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**一些实现细节**

整个项目都是开源的，RoadRunner的代码在这里。

RoaderRunner 通过 Socket/Pipe 上的二进制流完成和PHP子进程之间的通信。为此，他们创建了一个轻量的二进制协议，这个协议的包头长这样。

不同的通信方式，创建worker时就会有所区别。RoadRunner实现了两套factory和relay：

当使用pipe时，work是从`pipe_factory.go`创建的。PHP方面对应的Class是`StreamRelay.php`  
其中读取包头的部分代码是这样的

```
$prefixBody = fread($this->in, 17);  
if ($prefixBody === false) {  
    throw new Exceptions\PrefixException("unable to read prefix from the stream");  
}  
  
$result = unpack("Cflags/Psize/Jrevs", $prefixBody);  
if (!is_array($result)) {  
    throw new Exceptions\PrefixException("invalid prefix");  
}  
  
if ($result['size'] != $result['revs']) {  
    throw new Exceptions\PrefixException("invalid prefix (checksum)");  

```

当使用socket时，对应的PHP Class是`SocketRelay.php`, 代码是类似的。只不过PHP读取的部分从stdin/out变成了socket.

RoadRunner实现了`StaticPool`和`Worker`来对所有的PHP worker进行管理。本质上就是一个进程池。当需要PHP对数据进行处理时，StaticPool的`allocateWorker()`方法会从进程池的free worker中分配一个进程，把数据发送给这个PHP进程。StaticPool不会在PHP进程返回数据后关闭进程，而是把这个进程放回进程池。如果某个PHP进程意外关闭，staticPool会主动丢弃并在需要的时候创建新的进程。

### **举个栗子**

下面是一个实际应用RoadRunner的例子。我们创建了一个最多拥有4个PHP进程的StaticPool，Golang会把0到9的数字依次传给PHP处理，PHP把数字加1并返回。

Golang部分：

```
//创建RoadRunner server  
srv := roadrunner.NewServer(  
    &roadrunner.ServerConfig{  
        Command: "php " + phpScriptName,  
        Relay:   "pipes", // 选择用pipe通信  
        Pool: &roadrunner.Config{  
            NumWorkers:      4, // worker的数量是4  
            AllocateTimeout: time.Second,  
            DestroyTimeout:  time.Second,  
        },  
    })  
defer srv.Stop()  
  
err := srv.Start()  
if err != nil {  
    panic(err)  
}  
  
for i := 0; i < 10; i++ {  
    // 把i作为payload传给PHP进程  
    res, err = srv.Exec(&roadrunner.Payload{Body: i})  
    if err != nil {  
        panic(err)  
    }  
  
    fmt.Print(res)  
}  

```

PHP部分

```
<?php  
  
use Spiral\Goridge;  
use Spiral\RoadRunner;  
  
// 创建Worker实例，选择用pipe通信  
$rr = new RoadRunner\Worker(new Spiral\Goridge\StreamRelay(STDIN, STDOUT));  
  
// 长时运行，阻塞式接收go传来的数据  
while ($i = $this->rr->receive($context)) {  
    try {  
        $i = $i + 1;  
  
        $rr->send($i, (string) $context);  
    } catch (\Throwable $e) {  
        $rr->error((string) $e);  
    }  
}  

```

### **为什么要go + php?**

> “有些人仍然坚持认为 PHP 是一种缓慢，笨重的语言，只能用来编写 WordPress 插件。他们甚至可能会说 PHP 有一个限制：一旦你的应用程序变得比较大，你就必须切换到更“成熟”的语言并取代之前的 PHP 代码。”

基本上php脚本也都可以用go来写，那为什么不直接用100%的Go来开发项目呢？

  
我会认为这种模式还是有一些存在的意义的：

1.  开发效率高。尤其是PHP对html渲染有很好的支持，这又正是Go的痛点。
    
2.  用工成本的考虑。让大牛用Go写出框架，每个模块再由php分别实现，也许是个好主意。（这也算PHP的优势吗-。-？）
    
3.  PHP作为弱类型的脚本语言，用来实现一些小模块，真的很爽~！
    

  

> 转自：无敌的CF
> 
> 链接：https://zhuanlan.zhihu.com/p/60599237

  

- EOF -

推荐阅读  点击标题可跳转

1、[揭晓 Go 语言真实现状！](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666552454&idx=3&sn=4d0fe06a0cb011e717ed8787f1968c42&chksm=80dc982db7ab113b95433fec8b1f4cca3d58636a6657362135eae60e982f9eef260b042f1d0c&scene=21#wechat_redirect)

2、[使用 Go 构建 Kubernetes 应用](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666551110&idx=3&sn=ef9b6910535c15744b960400e7b566cc&chksm=80dc93edb7ab1afb0c19f1a5e176479730a786f76a668c7c2d76f609217bdea33707b54674ff&scene=21#wechat_redirect)

3、[作为多年 PHP 的开发者，在使用了 Go 语言之后......](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666545424&idx=1&sn=61ee1de8d21a692b32930aa2fa48ce31&chksm=80dc85bbb7ab0cad7231373e82a66960d87b2d9bc46b89bf04a93ee7cac64c93c70461835f85&scene=21#wechat_redirect)

  

看完本文有收获？请分享给更多人  

推荐关注「Linux 爱好者」，提升Linux技能

 ![Linux爱好者](http://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb3sjicd8JxDra10FRIqT54Zke2sfhibTDdtdnVhv5Qh3wLHZmKPjiaD7piahMAzIH6Cnltd1Nco17Ihjw/0?wx_fmt=png) ** Linux爱好者 ** 点击获取《每天一个Linux命令》系列和精选Linux技术资源。「Linux爱好者」日常分享 Linux/Unix 相关内容，包括：工具资源、使用技巧、课程书籍等。 72篇原创内容   公众号