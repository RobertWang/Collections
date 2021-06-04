> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/60599237)

**数据说话**
--------

![](https://pic3.zhimg.com/v2-25aa69f88c12f0e111d7f8e5f7b3d382_r.jpg)

从图中可以看出 RoadRunner 对比 Nginx+FPM，运行效率是有数量级上的提升。

**一般 PHP 服务器**
--------------

**传统 CGI 协议服务器**

客户端访问某个 URL 地址之后，通过 GET/POST/PUT 等方式提交数据，并通过 HTTP 协议向 Web 服务器发出请求，服务器端将 HTTP 请求里描述的信息通过标准输入 (stdin) 和环境变量 (environment variable) 传递给新建的 CGI 进程。处理完成后，进程立即关闭。

**Nginx + PHP-FPM 模式**

现在流行的 PHP web 程序一般都是运行在 Nginx + PHP-FPM 模式下的。`PHP-FPM`就是 PHP 对`FastCGI`的实现。  
master 创建并监听多个 worker 进程，通过共享内存获取 worker 的状态，进而通过信号控制 worker 进程。每一个 worker 进程就类似一个 CGI 进程，收到 CGI 请求后会执行相应的 PHP 文件，并把请求内容作为 PHP 进程状态的一部分 (_GET, _POST, _SERVER 等等)。结束请求后，worker 不会立刻结束，而是继续留在 worker pool. 这就节省了频繁创建结束子进程的开支。

**RoadRunner**
--------------

**为什么**

现在很多 PHP 的企业级框架都要求你加载至少十几个文件，构造多个类并解析一些配置，以便处理简单的用户请求或查询数据库。每个任务完成后，你不得不抛弃这些代码。收到下一个 HTTP 请求时，PHP-FPM 会创建一个新的 PHP 子进程来处理这个请求，所有的文件都要重新加载一遍，即便文件可以有缓存，所有的代码也要重新运行。

如果我们可以避免对每个请求都重启一次 PHP 子进程，我们就可以节约很多的资源。

**基本原理**

`RoadRunner`可以看作一个升级版的 Nginx + PHP-FPM. 它直接把长时运行的 PHP 进程作为 worker, 直接对 PHP worker 进行监控和维护，每次收到 http 请求时，就发给 php worker 来处理。这样，我们就不再需要对每个请求重启一遍 PHP 了。

![](https://pic2.zhimg.com/v2-986070c001fe3d34255cde6656e250e1_r.jpg)

**一些实现细节**

整个项目都是开源的，RoadRunner 的代码在[这里](https://link.zhihu.com/?target=https%3A//github.com/spiral/roadrunner)。

RoaderRunner 通过 Socket/Pipe 上的二进制流完成和 PHP 子进程之间的通信。为此，他们创建了一个轻量的二进制协议，这个协议的包头长[这样](https://link.zhihu.com/?target=https%3A//github.com/spiral/goridge/blob/master/prefix.go)。

不同的通信方式，创建 worker 时就会有所区别。RoadRunner 实现了两套 factory 和 relay：

当使用 pipe 时，work 是从`pipe_factory.go`创建的。PHP 方面对应的 Class 是`StreamRelay.php`  
其中读取包头的部分代码是这样的

```
$prefixBody = fread($this->in, 17);
if ($prefixBody === false) {
    throw new Exceptions\PrefixException("unable to read prefix from the stream");
}

$result = unpack("Cflags/Psize/Jrevs", $prefixBody);
if (!is_array($result)) {
    throw new Exceptions\PrefixException("invalid prefix");
}

if ($result['size'] != $result['revs']) {
    throw new Exceptions\PrefixException("invalid prefix (checksum)");


```

当使用 socket 时，对应的 PHP Class 是`SocketRelay.php`, 代码是类似的。只不过 PHP 读取的部分从 stdin/out 变成了 socket.

RoadRunner 实现了`StaticPool`和`Worker`来对所有的 PHP worker 进行管理。本质上就是一个进程池。当需要 PHP 对数据进行处理时，StaticPool 的`allocateWorker()`方法会从进程池的 free worker 中分配一个进程，把数据发送给这个 PHP 进程。StaticPool 不会在 PHP 进程返回数据后关闭进程，而是把这个进程放回进程池。如果某个 PHP 进程意外关闭，staticPool 会主动丢弃并在需要的时候创建新的进程。

### **举个栗子**

下面是一个实际应用 RoadRunner 的例子。我们创建了一个最多拥有 4 个 PHP 进程的 StaticPool，Golang 会把 0 到 9 的数字依次传给 PHP 处理，PHP 把数字加 1 并返回。

Golang 部分：

```
//创建RoadRunner server
srv := roadrunner.NewServer(
    &roadrunner.ServerConfig{
        Command: "php " + phpScriptName,
        Relay:   "pipes", // 选择用pipe通信
        Pool: &roadrunner.Config{
            NumWorkers:      4, // worker的数量是4
            AllocateTimeout: time.Second,
            DestroyTimeout:  time.Second,
        },
    })
defer srv.Stop()

err := srv.Start()
if err != nil {
    panic(err)
}

for i := 0; i < 10; i++ {
    // 把i作为payload传给PHP进程
    res, err = srv.Exec(&roadrunner.Payload{Body: i})
    if err != nil {
        panic(err)
    }

    fmt.Print(res)
}


```

PHP 部分

```
<?php

use Spiral\Goridge;
use Spiral\RoadRunner;

// 创建Worker实例，选择用pipe通信
$rr = new RoadRunner\Worker(new Spiral\Goridge\StreamRelay(STDIN, STDOUT));

// 长时运行，阻塞式接收go传来的数据
while ($i = $this->rr->receive($context)) {
    try {
        $i = $i + 1;

        $rr->send($i, (string) $context);
    } catch (\Throwable $e) {
        $rr->error((string) $e);
    }
}


```

### **为什么要 go + php?**

> “有些人仍然坚持认为 PHP 是一种缓慢，笨重的语言，只能用来编写 WordPress 插件。他们甚至可能会说 PHP 有一个限制：一旦你的应用程序变得比较大，你就必须切换到更 “成熟” 的语言并取代之前的 PHP 代码。”

基本上 php 脚本也都可以用 go 来写，那为什么不直接用 100% 的 Go 来开发项目呢？  
我会认为这种模式还是有一些存在的意义的：

1.  开发效率高。尤其是 PHP 对 html 渲染有很好的支持，这又正是 Go 的痛点。
2.  用工成本的考虑。让大牛用 Go 写出框架，每个模块再由 php 分别实现，也许是个好主意。（这也算 PHP 的优势吗 -。-？）
3.  PHP 作为弱类型的脚本语言，用来实现一些小模块，真的很爽~！

### **参考文献**

[https://blog.spiralscout.com/php-was-never-meant-to-die-830de87915ee](https://link.zhihu.com/?target=https%3A//blog.spiralscout.com/php-was-never-meant-to-die-830de87915ee)