> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.php.cn](https://www.php.cn/java-article-419750.html)

> Java 中的 BIO 是同步阻塞式，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。

本篇所写的 io 均为 java bio 体系（即 jdk1.0 发布的 io），JDK1.4 以前的唯一选择，但程序直观简单易理解。

**BIO:** 同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。BIO 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中。

**BIO**  
同步阻塞式 IO，相信每一个学习过操作系统网络编程或者任何语言的网络编程的人都很熟悉，在 while 循环中服务端会调用 accept 方法等待接收客户端的连接请求，一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成。  
如果 BIO 要能够同时处理多个客户端请求，就必须使用多线程，即每次 accept 阻塞等待来自客户端请求，一旦受到连接请求就建立通信套接字同时开启一个新的线程来处理这个套接字的数据读写请求，然后立刻又继续 accept 等待其他客户端连接请求，即为每一个客户端连接请求都创建一个线程来单独处理，大概原理图就像这样：

![](https://img.php.cn/upload/image/573/302/607/1558496168791046.jpg)

然此时服务器具备了高并发能力，即能够同时处理多个客户端请求了，但是却带来了一个问题，随着开启的线程数目增多，将会消耗过多的内存资源，导致服务器变慢甚至崩溃。

IO 方式适用于连接数目比较小且固定的场景，这种方式对服务器资源要求比较高，并发局限于应用中。

以上就是 java 中 bio 是什么的详细内容，更多请关注 php 中文网其它相关文章！