> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [cloud.tencent.com](https://cloud.tencent.com/developer/article/1965271)

> PHP 项目使用 Nginx 时，一般通过 php-fpm Nginx+PHP-FPM 形式访问交互，本文将详细解读 Nginx 配置文件、PHP-FPM、PHP-CGI 和 fastCGI 的概念。

PHP 项目使用 Nginx 时，一般通过 php-fpm `Nginx`+`PHP-FPM` 形式访问交互，本文将详细解读 Nginx 配置文件、PHP-FPM、PHP-CGI 和 fastCGI 的概念。

**一. 背景：**
----------

![](https://ask.qcloudimg.com/http-save/7284958/2de8d3e72ad77dfaf3af85a84984b5c4.webp)

在开发中碰到一个问题，项目以 `nginx`+`php-fpm` 形式访问交互，结果访问项目时报错如上图：

**二. 分析：**
----------

提示很明确嘛，去看 error.log（在 nginx.conf 或者 vhost 里头配置的，找到你对应路径即可）

错误信息如下：

```
2017/09/18 10:46:21 [error] 3880#0: *92 connect() failed (111: Connection refused)  
while connecting to upstream, client: 192.168.33.10, server: local.helios.com,  
request: "GET /v1/room/detail.json HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "local.helios.com"



2017/09/18 14:30:42 [crit] 5375#0: *43 connect() to unix:/tmp/php-cgi.sock failed (2: No such file or directory) 
while connecting to upstream, client: 192.168.33.10, server: www.lnmp.org,  
request: "GET /index.php?uri=look/index HTTP/1.1", upstream: "fastcgi://unix:/tmp/php-cgi.sock:", host: "local.helios.com"

```

关于问题：`connect() failed (111: Connection refused) while connecting to upstream` 网上也有很多描述了，

### 1.php-fpm 是否运行

```
ps aux |grep php-fpm 
/etc/init.d/php-fpm   或  /usr/local/php/sbin/php-fpm  

```

### 2.php-fpm 队列是否足够

```
cat /proc/meminfo -u 
vi /usr/local/php/etc/php-fpm.conf  
pm.max_children = 33 

```

### 3.php-fpm.conf 与 fastcgi.conf 的监听是否相同

*   1). 查看 php-fpm 监听方式：

```
vim /usr/local/php/etc/php-fpm.conf 
listen = 127.0.0.1:9000 
listen = /tmp/php-cgi.sock

```

*   2). 查看 nginx 指定与 php-fpm 通信方式：

```
vi /usr/local/nginx/conf/vhost/vhost.conf 
location ~\.php$ { 
  fastcgi_pass unix:/tmp/php-cgi.sock;【unxi domain socket形式】 
  fastcgi_index index.php;include fastcgi_params;         
  fastcgi_param  SCRIPT_FILENAME $document_root/$fastcgi_script_name; 
}

```

**三. 解决：**
----------

根据上边配置显示，情况已经很明显了，本次问题属于第三种情况:

*   我的 php-fpm 用 `ip:port` 方式建立链接
*   nginx 却用 `unix socket` 方式建立链接

这是两种不同方式，好比暗号不同，怎么可能接上头嘛！

修改操作如下：

用哪种取决于你的 PHP-FPM 配置:，以下二选一即可。

### 方式 1，统一成 ip+port 的形式:

```
php-fpm.conf:  
listen = 127.0.0.1:9000 
vhost.conf:  
fastcgi_pass 127.0.0.1:9000;

```

### 方式 2，统一成. sock 的形式:

```
php-fpm.conf:  
listen = /tmp/php-fpm.sock 
vhost.conf: 
fastcgi_pass unix:/tmp/php-fpm.sock;

```

重启 nginx 与 php-fpm

```
/etc/init.d/nginx reload 
/etc/init.d/php-fpm reload

```

搞定，访问再试试。

其中 php-fpm.sock 是一个文件, 由 php-fpm 生成, 类型是 srw-rw----. UNIX Domain Socket 可用于两个没有亲缘关系的进程, 是目前广泛使

**四. 知识延伸：**
------------

上边问题说到了是因为 nginx 与 php-fpm 进程通信不匹配造成的，那他们有什么不同呢？

### unix socket 方式

*   优点：

unix socket 方式要比 tcp 的方式快，而且消耗资源少，因为 socket 之间在 nginx 和 php-fpm 的进程之间通信，而 tcp 需要经过本地回环驱动，还要申请临时端口和 tcp 相关资源。

会有很多 linux 傻瓜面板，他们可能会有很多中 php-fpm 的版本，那么如果是不同版本去开不同的端口，然后 nginx 去配置的话，你要记住很多端口，sock 文件便是解决这个问题最简单的方法。

Socket 是使用 unix domain socket 连接套接字 / dev/shm/php-cgi.sock（很多教程使用路径 / tmp，而路径 / dev/shm 是个 tmpfs，速度比磁盘快得多）

*   缺点：

unix socket 会显得不是那么稳定，当并发连接数爆发时，会产生大量的长时缓存，在没有面向连接协议支撑的情况下，[大数据](https://cloud.tencent.com/solution/bigdata?from_column=20065&from=20065)包很有可能就直接出错并不会返回异常。

虽然 sock 有更少的数据拷贝和上下文切换，更少的资源占用，但是如果数据都是错的，那还有什么用呢。另外使用 sock 的话，必须 nginx 和 fpm 在同一台机器上

### tcp 方式

*   优点：

从稳妥的考虑肯定是使用 tcp，tcp 协议能保证数据的正确性，sock 不能保证。

可以跨[服务器](https://cloud.tencent.com/act/pro/promotion-cvm?from_column=20065&from=20065)，当 nginx 和 php-fpm 不在同一台机器上时，只能使用这种方式

*   缺点：

性能不如 unix socket

更多知识：[nginx、php-fpm 默认配置与性能–TCP socket 还是 unix domain socket](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fblog.csdn.net%2Fhemengsi123%2Farticle%2Fdetails%2F47787913)

**五、php-fpm 的理解**
-----------------

首先，CGI 是干嘛的？CGI 是为了保证 web server 传递过来的数据是标准格式的，方便 CGI 程序的编写者。

web server（比如说 nginx）只是内容的分发者。比如，如果请求 / index.html，那么 web server 会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态数据。好了，如果现在请求的是 / index.php，根据配置文件，nginx 知道这个不是静态文件，需要去找 PHP 解析器来处理，那么他会把这个请求简单处理后交给 PHP 解析器。

Nginx 会传哪些数据给 PHP 解析器呢？url 要有吧，查询字符串也得有吧，POST 数据也要有，HTTP header 不能少吧，好的，CGI 就是规定要传哪些数据、以什么样的格式传递给后方处理这个请求的协议。仔细想想，你在 PHP 代码中使用的用户从哪里来的。

当 web server 收到 / index.php 这个请求后，会启动对应的 CGI 程序，这里就是 PHP 的解析器。接下来 PHP 解析器会解析 php.ini 文件，初始化执行环境，然后处理请求，再以规定 CGI 规定的格式返回处理后的结果，退出进程。web server 再把结果返回给浏览器。

好了，CGI 是个协议，跟进程什么的没关系。那 fastcgi 又是什么呢？Fastcgi 是用来提高 CGI 程序性能的。

提高性能，那么 CGI 程序的性能问题在哪呢？"PHP 解析器会解析 php.ini 文件，初始化执行环境"，就是这里了。标准的 CGI 对每个请求都会执行这些步骤（不闲累啊！启动进程很累的说！），所以处理每个时间的时间会比较长。这明显不合理嘛！那么 Fastcgi 是怎么做的呢？

首先，Fastcgi 会先启一个 master，解析配置文件，初始化执行环境，然后再启动多个 worker。当请求过来时，master 会传递给一个 worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当 worker 不够用时，master 可以根据配置预先启动几个 worker 等着；当然空闲 worker 太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是 fastcgi 的对进程的管理。

那 PHP-FPM 又是什么呢？是一个实现了 Fastcgi 的程序，被 PHP 官方收了。

大家都知道，PHP 的解释器是 php-cgi。php-cgi 只是个 CGI 程序，他自己本身只能解析请求，返回结果，不会进程管理（皇上，臣妾真的做不到啊！）所以就出现了一些能够调度 php-cgi 进程的程序，比如说由 lighthttpd 分离出来的 spawn-fcgi。好了 PHP-FPM 也是这么个东东，在长时间的发展后，逐渐得到了大家的认可（要知道，前几年大家可是抱怨 PHP-FPM 稳定性太差的），也越来越流行。

好了，最后来回来你的问题。网上有的说，fastcgi 是一个协议，php-fpm 实现了这个协议

对。

有的说，php-fpm 是 fastcgi 进程的管理器，用来管理 fastcgi 进程的

对。php-fpm 的管理对象是 php-cgi。但不能说 php-fpm 是 fastcgi 进程的管理器，因为前面说了 fastcgi 是个协议，似乎没有这么个进程存在，就算存在 php-fpm 也管理不了他（至少目前是）。 有的说，php-fpm 是 php 内核的一个补丁

以前是对的。因为最开始的时候 php-fpm 没有包含在 PHP 内核里面，要使用这个功能，需要找到与源码版本相同的 php-fpm 对内核打补丁，然后再编译。后来 PHP 内核集成了 PHP-FPM 之后就方便多了，使用 --enalbe-fpm 这个编译参数即可。

有的说，修改了 php.ini 配置文件后，没办法平滑重启，所以就诞生了 php-fpm

是的，修改 php.ini 之后，php-cgi 进程的确是没办法平滑重启的。php-fpm 对此的处理机制是新的 worker 用新的配置，已经存在的 worker 处理完手上的活就可以歇着了，通过这种机制来平滑过度。

还有的说 PHP-CGI 是 PHP 自带的 FastCGI 管理器，那这样的话干吗又弄个 php-fpm 出

不对。php-cgi 只是解释 PHP 脚本的程序而已。

fastCGI

FastCGI 是一个可伸缩地、高速地在 HTTP Server 和动态脚本语言间通信的接口，它采用 C/S 结构，可以将 HTTP 服务和脚本解析服务器分开，当 HTTP 服务器遇到动态请求时，会将请求转发给 FastCGI 进程，FastCGI 进程执行动态脚本后再将结果返回给 HTTP 服务器，HTTP 服务器最后将结果输出给浏览器，这在很大程度上提高了请求的相应速度。

实现 FastCGI 有几种方式：php-CGI、php-FPM、Spawn-FCGI

nginx 结合 FastCGI 工作原理如下图：

![](https://ask.qcloudimg.com/http-save/7284958/636f65c2cdd11b670a5cb54e0760334f.webp)

php-CGI 是 PHP 自带的 fastCGI 管理器，有两个问题，

*   ① 变更 php.ini 时必须重启 php-cgi 才能生效
*   ② php-cgi 进程崩溃或者被杀死后 php 就不能运行

php-fpm 是从 php5.3.3 之后新加入的管理器，在更改 php 配置之后不需要重启，且由于加入守护进程，所以即使被杀死之后也能快速重启。

Spawn-fcgi 是一个通用的 fastcgi 管理器，而不仅仅只针对 php 一种脚本语言。但它在效率、cup 占用方面都不如 php-fpm.

php-fpm

php-fpm 是一个独立的进程，所以需要与 nginx 进行通信，有两种通信方式：

①tcp ②socket

这两种配置方式都需要修改 nginx 配置文件（/etc/nginx/sites-available/default）和 fpm 配置文件 (/etc/php/7.0/fpm/pool.d/www/conf)

① tcp

```
#nginx配置文件：
fastcgi_pass 127.0.0.1:9000;
#php-fpm配置文件：
listen = 127.0.0.1:9000
#重启nginx
service nginx restart

```

② socket

```
#nginx配置文件：
fastcgi_pass unix:/run/php/php7.0-fpm.sock;
#php-fpm配置文件：
listen = /run/php/php7.0-fpm.sock
#重启php-fpm
service php-fpm restart

```

lnmp 组合调用逻辑关系图：

![](https://ask.qcloudimg.com/http-save/7284958/e0e12f68fdd2cd533d376b24daba3cdd.webp)

lnmp FastCGI 调用 PHP MYSQL 关系逻辑图：

![](https://ask.qcloudimg.com/http-save/7284958/f52f0d6a4c93cf8d800e0d38a18082b5.webp)