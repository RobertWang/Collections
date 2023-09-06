> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/OkGrtYGqrigrAoiRBbALyA)

> image.png 我们知道，PHP 最初是为了支持同步开发而创建的，因此大多数 PHP 开发人员习惯于仅使用

  
我们知道，PHP 最初是为了支持同步开发而创建的，因此大多数 PHP 开发人员习惯于仅使用 php 编写同步代码，那么我们能用 Php 来进行异步编程吗。答案是可以的。

什么是异步编程
=======

当我们用 php 编写的应用程序 I/O 任务时，程序会在执行某个任务之前，**一定要等待之前的任务完成**，这时 CPU 会有很多时间处于空闲状态，这不仅会降低应用程序性能，还会降低硬件利用率。比如，当程序需要从数据库中读取大量的数据时，由于需要等待 I/O 操作完成，程序的执行速度会非常缓慢。因此，我们通过 Swoole 扩展或者一些如 Reactphp 的库，在程序执行的过程中，**不需要等待某个任务完成才能执行下一个任务**。这种编程模式可以极大地提高程序的效率和响应速度，尤其在处理复杂的 I/O 操作时表现得更为出色，而这就是**异步编程**。

Php pcntl 扩展
============

通过异步编程的概念可知，异步编程的可以通过多进程来实现，在 php 中，pcntl 扩展是首选的用来处理多进程的库，我们先测一段同步的代码

```
<?php
class AsyncPhp{
    public function curl($url)
    {
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_HEADER, 0);
        $output = curl_exec($ch);
        curl_close($ch);
        return $output;
    }
    public function testSync(){
        $url = 'https://www.baidu.com/?i=';
        $start=microtime(true);    
        for($i=0;$i<5;$i++){
            $this->curl($url.$i);
            echo '访问第'.($i+1).'次百度'."\n";
        }
        $end=microtime(true);

        echo "\n总用时",$end-$start."\n";        
    }
}

$tester = new AsyncPhp();
$tester->testSync();

```

当我们同步请求 5 次百度时，是 0.9 秒  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialK7EqI60rhqgM3BRiaBJmCZkYmWagWftWxDufUdqyAwibqIfGYHz93c1VA/640?wx_fmt=png)

  
这时为了提高性能，pcntl 就可以派上用场。  
pcntl 是一个可以利用操作系统的 fork 系统调用在 PHP 中实现多线程的进程控制扩展，当使用 fork 系统调用后执行的代码将会是并行的。**pcntl 的缺点是只适用于 Linux 平台的 CLI 模式下使用**  
我们在刚才的测试来新增如下方法来测试

```
    public function testAsync(){
        $url = 'https://www.baidu.com/?i=';
        $start=microtime(true); 
        $i=0;
        while($i<5){
            $pids[$i] = pcntl_fork(); //创建子进程
             if($pids[$i] == 0){ //返回0表示在子进程
                 $this->curl($url.$i);; //子进程执行代码
                 exit(0);
             }
             $i++;
         }
         //等待进程关闭
         for($i=0;$i<5;$i++){
             pcntl_waitpid($pids[$i],$status,WUNTRACED);//等待进程结束
             if(pcntl_wifexited($status)){
                 //子进程完成退出
                 echo '第'.$i.'次访问百度，用时:'.microtime(true)-$start."\n";
             }
         } 

        $end=microtime(true);

        echo "\n总用时",$end-$start."\n";        
    } 

```

在上面的代码中，当程序运行 pcntl_fork 时，Linux 系统立 fork 出一个子进程，并在子进程中返回 0，所以 $this->curl 是在子进程中运行的，exit(0）是终止了这个子进程，为什么要终止？如果不终止，子进程就会继续执行 while 这样最终导致产生很多次访问，而我们只要 5 次。pcntl_waitpid 用于等待子进程结束，这样就可以计算子进程运行时间了。代码执行结果如下  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialK2PZK8MRRhFQtoic1mNmaY62QhAdNa365rpy7WIBZibuAAFrK3WPq6FAQ/640?wx_fmt=png)image.png

  
通过对比可知，php 使用 pcntl 扩展实现异步总用时比传统同步方式少用时 0.7 秒，性能提升了 3 倍多。但是 pcntl 只能跑在 cli 环境下，传统 php-fpm 环境下是无法使用的，这时我们可以在网络请求这边想办法，于是有了 **curl_multi**。

使用 curl_multi 函数组
=================

**curl_multi 函数组允许异步处理多个 cURL 句柄,** 下面是具体代码

```
    public function getCurl($url)
    {
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_HEADER, 0);
        return $ch;
    }    
    public function testCurlMulti(){
        $url = 'https://www.baidu.com/?i=';
        $start=microtime(true); 
        $mh = curl_multi_init();

        $curl_array = array();
        for ($i=0; $i < 5; $i++) { 
            $curl_array[$i] = $this->getCurl($url.$i);
            curl_multi_add_handle($mh, $curl_array[$i]);
        }
        $running = NULL;
        do {
            curl_multi_exec($mh,$running);
        } while($running > 0);

        foreach($curl_array as $i=>$item){
            $cotnent = curl_multi_getcontent($curl_array[$i]);
            echo '第'.$i.'次访问百度，用时:'.microtime(true)-$start."\n";
            curl_multi_remove_handle($mh, $curl_array[$i]);
        }
        curl_multi_close($mh);  

        $end=microtime(true);

        echo "\n总用时",$end-$start."\n";            
    }

```

在上面的代码中，curl_multi_exec 函数将并行执行每个 curl 句柄。最终总用时如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialKACUvhEgmrOxXgxSEamMoa3RukcXz2EZhsHM1LvKiaHkwJUVEamf727A/640?wx_fmt=png)

  
**也比同步的代码快了一倍了**。  
curl_multi 函数组在很多场景下都有使用到，**特别是在网络爬虫开发中会经常使用到**。

Parallel 扩展
===========

同 pcntl 一样，Parallel[1] 也是一个 php 扩展，安装方法：https://github.com/krakjoe/parallel/blob/develop/INSTALL.md，如果安装时报如下错误，  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialKciaq9GQUWmZn2HqvGib6MMQ7xnJxL97vR2nZ86DxMEZhWRFPiaDVzsEeQ/640?wx_fmt=png)

  
则需要重新编译 php，编译方式如下：

```
wget https://www.php.net/distributions/php-8.1.1.tar.gz
tar -zxvf ./php-8.1.1.tar.gz
cd php-8.1.1
# 编译
./configure --prefix=/usr/local/php81 --enable-debug --enable-zts --with-config-file-path=/usr/local/php81/etc --with-config-file-scan-dir=/usr/local/php81/conf.d --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-iconv=/usr/local --with-freetype=/usr/local/freetype --with-jpeg --with-zlib --enable-xml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --with-curl --enable-mbregex --enable-mbstring --enable-intl --enable-pcntl --enable-ftp --enable-gd --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-zip --enable-soap --with-gettext --enable-opcache --with-xsl --with-pear --disable-mbregex
make && make install
# 搞定
ln -s /usr/local/php81/bin/php /usr/bin/php81
php -v

```

以上安装过程中有可能会出错，出错的解决方法请看我的另一篇文章[《为了编译启用 ZTS 的 php，我一路披荆斩棘》](https://mp.weixin.qq.com/s?__biz=MzI5MTM3NTE0MQ==&mid=2247484347&idx=1&sn=f15235680f0a8d388d78104a79cefa72&chksm=ec10d2f1db675be733285a86136a2274f6bd644332ae1e509b91e437921bd73993165e841681&scene=21#wechat_redirect "《为了编译启用ZTS的php，我一路披荆斩棘》")。  
成功安装 php 之后, 会打印如下结果  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialKDIQEm3iapC03kSIE5KkIclVnUovUc4oI3U3DsPuKFtSAjvDUtSEC3CA/640?wx_fmt=png)

  
ZTS 的 php 环境有了，就可以安装 parallel 了

```
git clone https://github.com/krakjoe/parallel.git
cd parallel
phpize
./configure --enable-parallel  [ --enable-parallel-coverage ] [ --enable-parallel-dev ]
make
make test
make install

```

安装成功后是这样子的  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialKDmbaaZLKJDlD59RmHjlW8qSGI0GboydDgRFEo0ibp8RsrWsOCBEwBiaQ/640?wx_fmt=png)

  
我们继续新增 testParallel 方法。代码如下

```
    public function testParallel(){        
        $ch = new Channel();
        
        // executed within thread
        $start=microtime(true); 
        $task = function (Channel $channel, int $i){
            $url = 'https://www.baidu.com/?i=';
            $url = $url.$i;
            $ch = curl_init();
            curl_setopt($ch, CURLOPT_URL, $url);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
            curl_setopt($ch, CURLOPT_HEADER, 0);
            $output = curl_exec($ch);
            curl_close($ch);
            // echo $output;
            echo "第".$i."次访问百度\n";
              $channel->send($output);       
        };
        
        // creating a few threads
        $runtimeList = [];
        for ($i = 0; $i < 5; $i++) {
            $runtimeList[] = new Runtime();
        }
        // run all threads
        $futureList = [];
        foreach ($runtimeList as $i => $runtime) {
            $futureList[] = $runtime->run($task, [$ch, $i]);
        }
        
        $ch->close();
        
        // echo $queue . PHP_EOL;
    }

```

Parallel 底层用的也是多进程机制，**但术语一般是说成线程，上面的代码中，我们定义了 5 个线程**，并发运行结果如下  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialKW9202iaibOIorDFrsNvkdZRNc0cJnX1pXPiaXlV2z1Eto9HJiaXarIzlfw/640?wx_fmt=png)

ReactPHP
========

  
与前面几个扩展库不同，ReactPHP[2] 是一个用 PHP 编写的基于事件驱动编程（EventLoop) 的低级库。其核心是事件循环，在其之上提供低级实用程序，例如：流抽象、异步 DNS 解析器、网络客户端 / 服务器、HTTP 客户端 / 服务器以及与进程的交互。  
安装 ReactPHP 很简单，直接用 composer 安装

```
composer require react/react

```

ReactPHP 有各种组件，例如事件循环、promise 和流。当我们安装 ReactPHP 时，这些组件和其他一些组件都会被安装，因此您不必单独安装  
ReactPHP 异步编程的写法采用了类 Promise 的设计，熟悉 ES6 Promise 的话上手很快。

```
require __DIR__ . '/vendor/autoload.php';

$browser = new React\Http\Browser();
$url = 'https://www.baidu.com/?i=';
$start = microtime(true);

for ($i=0; $i < 500; $i++) { 
  $browser->get($url.$i)->then(function (Psr\Http\Message\ResponseInterface $response) use($i,$start){
    $result = $response->getBody();
    echo "第{$i}次访问百度完成，用时：".microtime(true)-$start."\n";
  }, function (Exception $e) {
    echo 'Error: ' . $e->getMessage() . PHP_EOL;
  });
}


```

上面的代码确实实现了异步编程。**但我们发现用时并没有多大的提升**  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialKiaHj9WexwTR9QZOZC415eUkGScVVn5EQZ9z4uWTB4SUuMWz87CUOAhw/640?wx_fmt=png)

  
**总用时为 0.63 秒？为什么？**  
我们把 5 次访问改为 500 次，运行程序时，我们新开个命令行窗口，输入 ps -ef | grep php 来观察 php 进程  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialKmCr06oc7LTGIIH656CmCc0uiaibZqSR3vicR5tCNsw6ARRuoib809jmUTg/640?wx_fmt=png)

  
我们发现始终都只有一个 php ReactPhp2.php 的进程，**说明 Reactphp 没有采用多进程机制，但它的优点大于它的缺点。**

1.  1. ReactPHP 安装非常方便，直接用 composer require 就行，不需要其他 php 扩展的支持
    
2.  2. ReactPHP 对异步编程进行了非常好的支持，书写语法非常友好。具体查看官方文档就知道了
    

Fibers
======

2021 年 11 月发布的 PHP 8.1 开始支持了一个新特性： Fibers[3]，用于实现轻量级协程。由于它是一个非常底层的 API ，并不是直接可以使用于应用层，更多的，要使用 Fibers，只需把 php 版本更新到 8.1 即可。

```
<?php
declare(ticks=1);//Zend引擎每执行1条低级语句就去执行一次 register_tick_function() 注册的函数

class Thread {
  protected static $names = [];
  protected static $fibers = [];
  protected static $params = [];

  public static function register(string|int $name, callable $callback, array $params)
  {
    self::$names[]  = $name;
    self::$fibers[] = new Fiber($callback);
    self::$params[] = $params;
  }

  public static function run() {
    $output = [];

    while (self::$fibers) {
      foreach (self::$fibers as $i => $fiber) {
          try {
              if (!$fiber->isStarted()) {
                  // Register a new tick function for scheduling this fiber
                  register_tick_function('Thread::scheduler');
                  $fiber->start(...self::$params[$i]);
              } elseif ($fiber->isTerminated()) {
                  $output[self::$names[$i]] = $fiber->getReturn();
                  unset(self::$fibers[$i]);
              } elseif ($fiber->isSuspended()) {
                $fiber->resume();
              }                
          } catch (Throwable $e) {
              $output[self::$names[$i]] = $e;
          }
      }
    }

    return $output;
  }

  public static function scheduler () {
    if(Fiber::getCurrent() === null) {
      return;
    }
    // running Fiber::suspend() in this if condition will prevent an infinite loop!
    if(count(self::$fibers) > 1)
    {
      Fiber::suspend();
    }
  }
}


$start = microtime(true);
function curl($url)
{
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    curl_setopt($ch, CURLOPT_HEADER, 0);
    $output = curl_exec($ch);
    curl_close($ch);
    return $output;
}
function callback($i){
    $url = 'https://www.baidu.com/?i=';
    return curl($url.$i);
};
// registering 6 Threads (A, B, C, D, E, and F)
foreach(range(1, 5) as $id) {
  Thread::register($id, function($id,$start){
    $result = callback($id);
    return microtime(true) - $start;//放回用时。
  }, [$id,$start]);
}

// run threads and wait until execution finishes
$outputs = Thread::run();


print_r($outputs);

```

以上代码通过 declare(ticks=1) 来实现 Fibers 协程的切换。通过 register_tick_function 函数来让程序没运行一行就执行 Fiber::suspend()，这样就可以达到协程切换的目的，最后 $outputs 打印的时每个协程的耗时。也因为频繁 suspend，耗时比较长。但我们异步编程的目录已达到了。  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialKfSYCOOyticbZm3zVjmW77icFTfKiaDhf0RVIPBqR4PSNYdEMhcZ6QmsYg/640?wx_fmt=png)

Amphp
=====

前面的例子只是使用 Fiber 完成了一个最简单但不太合理的协程。实际上 Fiber 扩展进入内核后，它是一个非常底层的 API ，并不是直接用于应用层的。而 **Amphp**[4] 利用了 Fibers 的特殊，完成一个完整的协程框架，里面包含了一大堆组件，可以根据需要 composer 对应的组件就行。  
安装：

```
composer require amphp/amp

```

Amphp 的使用和 Reactphp 很像，使用方法大概如下：

```
<?php

use Amp\Future;

$httpClient = HttpClientBuilder::buildDefault();
$uris = [
    "google" => "https://www.google.com",
    "news"   => "https://news.google.com",
    "bing"   => "https://www.bing.com",
    "yahoo"  => "https://www.yahoo.com",
];

try {
    $responses = Future\await(array_map(function ($uri) use ($httpClient) {
        return Amp\async(fn () => $httpClient->request(new Request($uri, 'HEAD')));
    }, $uris));

    foreach ($responses as $key => $response) {
        printf(
            "%s | HTTP/%s %d %s\n",
            $key,
            $response->getProtocolVersion(),
            $response->getStatus(),
            $response->getReason()
        );
    }
} catch (Exception $e) {
    // If any one of the requests fails the combo will fail
    echo $e->getMessage(), "\n";
}

```

Swoole
======

我们最后介绍 Swoole，**swoole 目前应该是 php 中异步编程的首选框架了**，在国内有很高的知名度，甚至在有些 php 招聘中要求必会 swoole。

```
use Swoole\Coroutine\Channel;
use function Swoole\Coroutine\go;
use function Swoole\Coroutine\run;
run(function() {
    $chan = new Channel(5);

    go(function () use ($chan) {
        $cli = new Swoole\Coroutine\Http\Client('www.qq.com', 80);
        $ret = $cli->get('/');
        $chan->push(['key' => 'www.qq.com', 'content' => '访问www.qq.com成功！']);
    });

    go(function () use ($chan) {
        $cli = new Swoole\Coroutine\Http\Client('www.163.com', 80);
        $ret = $cli->get('/');
        $chan->push(['key' => 'www.163.com', 'content' => '访问www.qq.com成功！']);
    });

    for ($i = 0; $i < 2; $i++) {
        $result = $chan->pop();
        var_dump($result);
    }
});

```

上面的代码执行结果如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfcpuNFswSLGE2oXI23EialKGFyva8YPKTEzIyF2wVbQXoib1BGV4rTZ5pI1NskXfNVl53HWCu2DQ1Q/640?wx_fmt=png)

  
Swoole 具体使用可以看一下老俊写的教程[《Swoole 入门教程》](https://mp.weixin.qq.com/s?__biz=MzI5MTM3NTE0MQ==&mid=2247483951&idx=1&sn=cb051298485fcb10cab725d8a07a0851&chksm=ec10d365db675a735a86b0e7c516b4741fea6a57c1d342a2cd6c72137c263ff1b761768c6112&scene=21#wechat_redirect "《Swoole入门教程》")。

#### 引用链接

`[1]` Parallel: _https://www.php.net/parallel_  
`[2]` ReactPHP: _https://reactphp.org/_  
`[3]` Fibers: _https://www.php.net/manual/zh/class.fiber.php_  
`[4]` **Amphp**: _https://amphp.org/_