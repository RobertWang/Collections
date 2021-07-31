> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555749&idx=2&sn=80292e2498dbaeea9c23e647a59064fd&chksm=80dcadceb7ab24d8495f7727368f8f9fde222e10f50fa4823701c3703dbeae525f9ae0c4ca66&scene=21#wechat_redirect)

校招形势

后台开发工程师主流使用的编程语言有 C++、Java、PHP 以及目前慢慢流行的 Golang 等。本文就将以 C++ 的角度，讲讲如何学习和准备后台开发的岗位。

无论是 C++ 开发还是 Java 开发，对于一个码农而言，最重要的就是对于编程语言的熟悉。同样，无论从事哪种类型的岗位，首当其冲的就是要掌握好语言基础。

C++ 是一门博大精深的编程语言，不仅拥有继承于 C 语言的过程化程序设计思想，还包含有面对对象（OOP）的设计理念。强大而又复杂。相对来说，C++ 的学习成本较高，语言里面的坑较多。语言基础的学习路线如下：

1 语法基础

**重点掌握：（务必熟悉底层机制原理）**

*   指针和引用的概念
    
*   指针与内存关系
    
*   程序编译过程
    
*   static、const、#define 的用法和区别
    
*   C 和 C++ 区别
    
*   内存模型
    
*   内存中的栈和堆分配
    

2 面对对象基础

**（务必熟悉底层机制原理）**

*   面向对象理解
    
*   析构函数
    
*   构造函数
    
*   拷贝构造
    
*   多态
    
*   纯虚函数和虚函数
    
*   虚函数实现机制
    
*   虚函数表
    
*   访问限定符 public、private、protected
    
*   继承原理、虚继承、菱形继承
    
*   静态绑定和动态绑定
    
*   new/delete 和 malloc/free
    
*   重载、重写和隐藏
    

3 语法进阶

**（务必熟悉底层机制原理）**

*   智能指针
    
*   左值、右值引用和 move 语义
    
*   类型转换方式
    
*   常用的设计模式
    
*   线程安全的单例模式
    
*   内存溢出和内存泄漏
    
*   C++11 新特性
    
*   静态链接库和动态链接库
    

4 STL 标准模板库

**（务必能进行源码剖析）**

*   迭代器、空间配置器理解
    
*   常用容器特点、用法以及底层实现 vector、list、deque、set、map、unorderedmap
    

5 推荐书籍

*   **《C++Primer》**可作为工具书，随手查阅  
    
*   **《EffectiveC++》**深入了解 C++ 的程序设计规范
    
*   **《STL 源码剖析》**剖析 STL 的源码底层，非常具有学习价值
    
*   有精力还可以看**《深度探索 C++ 对象模型》《more EffecticeC++》**
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/TdGLaSU675ga7elH1wTp3Q9epO6U3nSZeVtKvocfCNJhUdZ7va61S6mP2Y0ZQH3ZuAFr6QoTDKaHHkd7PXmibAw/640?wx_fmt=png)

二、算法与数据结构

对于普通人而言，算法的学习最重要的是能够形成基本的算法思维，懂得从程序设计的角度对高重复性的操作做优化。这其中基本算法思想的掌握和常用数据结构的理解是必不可少。这方面的学习更倾向于多看多想多练。

1 常见算法类型

**（务必能够手撕代码）**

*   排序算法（冒泡、插入、选择、快排、希尔、堆排、归并、桶排、基数、计数）、字符串操作、数组操作、递归、回溯、分治、动态规划等  
    
*   如何准备算法可见历史文章
    
    [进入 BAT 和字节跳动最难的一关，手撕代码！](http://mp.weixin.qq.com/s?__biz=MzI2MTcxNjg5OA==&mid=2247483913&idx=1&sn=7038fe493411992bef3241336cc4fa82&chksm=ea576cc4dd20e5d2a40f846264a80669328160dbce734186b3c6e037f2c9d7871670d8d25b8f&scene=21#wechat_redirect)  
    

2 常用数据结构

**（务必熟悉底层原理和实现）**

*   链表、栈、队列、树 (二叉树、平衡二叉树、红黑树、B 树、B + 树、哈夫曼树、字典树)、跳表、图  
    

3 推荐书籍

*   **《大话数据结构》**适合入门学习  
    
*   **《剑指 offer》**必刷 66 题
    
*   **《算法导论》**尽量看，能啃完就是大神
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/TdGLaSU675ga7elH1wTp3Q9epO6U3nSZeVtKvocfCNJhUdZ7va61S6mP2Y0ZQH3ZuAFr6QoTDKaHHkd7PXmibAw/640?wx_fmt=png)

三、计算机网络

网络相关的东西不是很多，关键在于对常见网络协议簇的认识和理解，以及一些常规操作底层设计实现的剖析。比如：

 | 输入 www.baidu.com 会发生什么

 | 微信扫描登录会发生什么

1 重点掌握知识点

*   OSI 七层模型  
    
*   TCP/IP 五层模型
    
*   TCP/IP 协议总结
    
*   TCP、UDP 区别
    
*   TCP 三次握手、四次挥手
    
*   TCP 状态转换
    
*   TCP 状态中 TIME_WAIT
    
*   TCP 连接建立需要为什么不是两次握手
    
*   TCP 第三次握手失败会出现什么
    
*   TCP 长连接和短链接及优缺点
    
*   TCP 拥塞控制 - 慢启动、拥塞避免、快重传、快启动
    
*   TCP 如何保证可靠性传输
    
*   TCP 如何解决粘包、拆包问题
    
*   TCP 为什么可靠
    
*   UDP 如何实现 TCP 可靠传输
    
*   IP 地址和子网掩码
    
*   ARP 解析过程
    
*   DNS 原理
    
*   HTTP 状态码
    
*   HTTP1.0、HTTP1.1、HTTP2.0 区别
    
*   HTTP 和 HTTPS 区别
    
*   HTTPS 加密过程
    
*   非对称加密和对称加密算法
    
*   Nagle 算法
    

2 推荐书籍

*   **《计算机网络自顶向下方法》**教材书，可放手边查阅  
    
*   **《TCP/IP 详解》**重点了解 TCP、IP、UDP 协议实现
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/TdGLaSU675ga7elH1wTp3Q9epO6U3nSZeVtKvocfCNJhUdZ7va61S6mP2Y0ZQH3ZuAFr6QoTDKaHHkd7PXmibAw/640?wx_fmt=png)

四、数据库

数据库的一般使用其实不难，但是对于不同数据库的特性、实现机制、应用场景和性能优化方面却能够难倒一大批面试者。同样数据库本身也是非常好的项目实例，往往能够从中学习到许多程序设计的思想和模式。因此，对数据库要明白怎么用、为什么用、怎么用得好这几个方面的问题。

1 重点掌握

*   数据库类别
    
*   关系型数据库和非关系型数据库区别
    

 MySQL：

*   SQL 常见语句
    
*   MySQL 内链接，外链接（左链接、右链接、全链接）
    
*   MySQL 索引类型和原理
    
*   MySQL 事务实现原理 ACID
    
*   MySQL 数据存储引擎
    
*   MySQL 主从复制原理、作用和实现
    
*   MySQL 日记系统 redo log、binlog、undo log
    
*   MVCC 实现原理
    
*   Sql 优化思路
    
*   范式理论
    
*   数据库高并发解决方法
    

 Redis： 

*   Redis 支持的数据类型
    
*   Redis 持久化
    
*   Redis 架构模式
    
*   主从复制
    
*   一致性哈希算法
    

2 推荐书籍

*   **《高性能 Mysql》**能够加深对 Mysql 的理解和使用
    
*   **《Redis 设计与实现》**比较全面的书，可以多看看
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/TdGLaSU675ga7elH1wTp3Q9epO6U3nSZeVtKvocfCNJhUdZ7va61S6mP2Y0ZQH3ZuAFr6QoTDKaHHkd7PXmibAw/640?wx_fmt=png)

五、操作系统

操作系统的问题会集中在进程和线程，但是这一类的问题往往会以开放题的形式出现。主要考察的是对操作系统组件以及运行过程的理解。比如：

    | 开机登录系统发生了什么？

    | 复制粘贴是怎样操作的？

1 重点掌握

*   物理内存和虚拟内存
    
*   缓存 IO 和直接 IO
    
*   作业调度算法
    
*   线程和进程
    
*   进程和线程的调度
    
*   线程的创建和结束
    
*   线程状态
    
*   线程间通信与线程同步机制
    
*   互斥锁和信号量
    
*   线程池
    
*   消费者和生产者
    
*   死锁
    
*   并发和并行
    

2 推荐书籍

*   **《深入理解计算机系统》**很全面的书，这一本就够用了
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/TdGLaSU675ga7elH1wTp3Q9epO6U3nSZeVtKvocfCNJhUdZ7va61S6mP2Y0ZQH3ZuAFr6QoTDKaHHkd7PXmibAw/640?wx_fmt=png)

六、Linux 系统

对 Linux 系统的熟练使用是后台开发 / 服务器开发的必备技能点。这年头，不会几个 Linux 指令都不好意思说自己是敲代码的。（客户端和前端的同学表示不服）不管怎样，对于 Linux 系统的掌握无论在哪个方向上，都会有用武之地的。

1 Linux 系统操作和命令

*   top 命令
    
*   ps 命令
    
*   netstat 命令
    
*   awk 命令
    
*   find 命令
    
*   grep 命令
    
*   wc 命令
    
*   sed 命令
    
*   head 和 tail 命令
    
*   正则表达式
    
*   如何查找出现频率最高的 100 个 IP 地址
    
*   linux 如何统计文件中某个字符串出现的频率
    
*   linux 启动的第一个进程
    
*   linux 查看端口占用
    
*   linux 查看 CPU 和内存使用
    
*   Linux 查看系统负载命令
    
*   Linux 调试程序
    
*   Linux 硬链接和软连接
    
*   core dump
    
*   cmake 和 makefile
    
*   Shell 脚本基本语法和使用
    

2 推荐书籍

*   **《鸟哥私房菜》**入门足够了，多敲多写才能更快掌握
    

后台开发是离不开网络编程的，甚至简单来说，后台开发就是用厉害点的电脑去处理大规模的网络请求。所以作为一名合格的后端开发人员，对 Linux 网络编程的熟悉是必不可少的。

1 重点掌握

*   孤儿进程、僵尸进程和守护进程
    
*   进程间通信方式 signal、file、pipe、shm、sem、msg、socket
    
*   线程同步机制线程：互斥量、锁机制、条件变量、信号量、读写锁
    
*   fork 返回值
    
*   五大 IO 模型：阻塞 I/O、非阻塞 I/O、I/O 复用、信号驱动 I/O、异步 I/O
    
*   IO 复用机制
    
*   epoll 与 select/poll
    
*   LT 水平触发和 ET 边缘触发
    
*   Reactor 和 Proactor 模式
    
*   反向代理、负载均衡
    

2 推荐书籍

*   **《UNIX 环境高级编程》APUE** 比较难啃，可以挑着看
    
*   **《Unix 网络编程》UNP** 同样比较难啃，可以挑着看
    
*   **《Linux 多线程服务器端编程》**Muduo 网络库，推荐看看源码实现
    
*   **《深入理解 Nginx》**深入了解基于 C 的 web 服务器实现
    

如果以上的东西你都已经准备好了，那么相信你已经了具备 C++ 后台开发能力。但是要记得，一个大型线上项目的开发，从来都不纯粹是单一语言的设计和实现。

因此用 C++ 或者用 Java 或者用 Golang 或者用 Python 的区别或许没那么大，它们都有擅长的地方，毕竟存在即真理。所以，如果你真的有精力的话，不妨还可以了解一下更深层次的技术：

*   海量日志处理和并行计算开发
    
*   分布式技术框架、中间件等 Dubbo、Spring Cloud 、Zookeeper 、Kfaka
    
*   流媒体分发技术 CDN
    
*   ...
    

当然，这些都不是非常必要的。但是绝对是亮点！此外，你可以准备一些基础向的相关项目：

*   网络库，可参考 Muduo 或者 Nginx 实现
    
*   web 服务器 / http 服务器，可实现基本的 http 响应请求和处理
    
*   简易版 STL 库，展现 C++ 的综合代码能力
    
*   局域网聊天室开发，涉及到网络编程实现在线群聊
    
*   分布式日志系统
    
*   简易版数据库设计
    
*   可参考一些 C++ 常用库，造一些轮子或者做些有趣的小工具。
    
*   ...
    

**总结**  

在校园招聘中，对后台开发的面试大多还是针对候选人的计算机基础。大多数学生在校内接触不到太多高并发高可用的服务场景，甚至能上线的项目都很少，因此也很难要求校招生能够真正具备后台开发的能力。

所以对于 C++ 后台开发岗而言，对 C++/Linux 的充分熟悉以及扎实的计算机基础和有相关的学习经历，就已经能够满足各大公司的要求了。当然如果有在基础架构分布式开发等方面的经验，就更是各大厂抢手的香饽饽。

本文所介绍的整体学习路线可覆盖绝大多数大厂的面试题目和考察范围，如今学习资料太多，选择路线清晰的适合自己的才最重要。资料不必多，能理解掌握才是最关键的。 

- EOF -

推荐阅读  点击标题可跳转

1、[12.9k Star！这个 ZSH 的增强工具让你爱上命令行！](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555715&idx=1&sn=df47945ee1c3e3e8e02ababffe3ba3cb&chksm=80dcade8b7ab24fe10a7174343e3f67da983170d96df10471152c7eb89c3d5717d8bb7f36cd2&scene=21#wechat_redirect)

2、[“我花了 5 年时间编写自己的操作系统！”](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555710&idx=1&sn=290372f835473f0b89d62ff0146410ef&chksm=80dcad95b7ab248373e54d8f6d0a14b735a12d3c90d7611cb9c2bb8f59d8315c263c4c31c39b&scene=21#wechat_redirect)

3、[52 图初探 Linux 通用知识](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555692&idx=2&sn=42ffc8ae8e13e6b3bbd322732d042813&chksm=80dcad87b7ab2491bc472fa5db1451769fa4ced812e05dd4f9fa3ead79996680ea4506bf36f6&scene=21#wechat_redirect)