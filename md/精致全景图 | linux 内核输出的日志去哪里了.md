> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTkwMTE4Nw==&mid=2247497420&idx=2&sn=c8fb061bef6471f3b338637ca42d68a8&chksm=fc730d20cb048436974d6ddb1352b470a3c2156e9effe0a1dbb8258e7bfd1c010d46e207ea2b&mpshare=1&scene=1&srcid=1206HyAsnjxj6oWRgE5bU4IS&sharer_sharetime=1638772295315&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

![](https://mmbiz.qpic.cn/mmbiz_png/z79xARvmm9hTIrt1BeJhwTtqzyTPPZPP02ulsH0DAL31MO4oaWibNkEchKo1icZWRK5aDHmRb0CCibr9TKwjs3OQw/640?wx_fmt=png)

因为图片比较大，微信公众号上压缩的比较厉害，所以很多细节都看不清了，我单独传了一份到 github 上，想要原版图片的，可以点击下方的阅读原文，或者直接使用下面的链接，来访问 github：  

https://github.com/wangyuntao/linux-kernel-illustrated

另外，精致全景图系列文章，以及之后的 linux 内核分析文章，我都会整理到这个 github 仓库里，欢迎大家 star 收藏。

熟悉 linux 内核，或者看过 linux 内核源码的同学就会知道，在内核中，有一个类似于 c 语言的输出函数，叫做 printk，使用它，我们可以打印各种我们想要的信息，比如内核当前的运行状态，又或者是我们自己的调试日志等，非常方便。

那当我们调用 printk 函数后，这些输出的信息到哪里去了呢？我们又如何在 linux 下的用户态，查看这些信息呢？  

为了解答这些疑问，我画了一张 printk 全景图，放在了文章开始的部分，这张图既包含了 printk 在内核态的实现，又包含了其输出的信息在用户态如何查看。

我们可以根据这张图，来理解 printk 的整体架构。

在内核编码时，如果想要输出一些信息，通常并不会直接使用 printk，而是会使用其衍生函数，比如 pr_err / pr_info / pr_debug 等，这些衍生函数附带了日志级别、所属模块等其他信息，比较友好，但其最终还是调用了 printk。

printk 函数会将每次输出的日志，放到内核为其专门分配的名为 ring buffer 的一个槽位里。

ring buffer 其实就是一个用数组实现的环形队列，不过既然是环形队列，就会有一个问题，即当 ring buffer 满了的时候，下一条新的日志，会覆盖最开始的旧的日志。

ring buffer 的大小，可以通过内核参数来修改。

printk 在将日志放到 ring buffer 后，会再调用系统 console 的相关方法，将还未输出到系统控制台的消息，继续输出到控制台，这个后面会详细说，这里就暂不赘述。

以上就是 printk 在内核态的实现。  

在用户态，我们有几个方式，可以查看 printk 输出的内核日志，比如使用 dmesg 命令，cat /proc/kmsg 文件，或者是使用 klogctl 函数等，这些方式分别对应于全景图中用户态的橙色、绿色、和蓝色的部分。

dmesg 命令，在默认情况下，是通过读取 / dev/kmsg 文件，来实现查看内核日志的。  

当该命令运行时，dmesg 会先调用 open 函数，打开 / dev/kmsg 文件，该打开操作在内核中的逻辑，会为 dmesg 分配一个 file 实例，在这个 file 实例里，会有一个 seq 变量，该变量记录着下一条要读取的内核日志在 ring buffer 中的位置。

刚打开 / dev/kmsg 文件时，这个 seq 指向的就是 ring buffer 中最开始的那条日志。

之后，dmesg 会以打开的 / dev/kmsg 文件为媒介，不断的调用 read 函数，从内核中读取日志消息，每读取出一条，seq 的值都会加一，即指向下一条日志的位置，依次往复，直到所有的内核日志读取完毕，dmesg 退出。

以上就是 dmesg 的主体实现。

第二种查看内核日志的方式，是通过 cat /proc/kmsg 命令。

该命令和 dmesg 命令的实现机制基本类似，都是通过读文件，只不过 cat 读取的是 / proc/kmsg 文件，而 dmesg 读取的是 / dev/kmsg 文件。

读取这两个文件最大的区别是，/dev/kmsg 文件每次打开时，内核都会为其分配一个单独的 seq 变量，而 / proc/kmsg 文件每次打开时，用的都是同一个全局的静态 seq 变量，叫做 syslog_seq。

syslog_seq 指向的也是下一条要读取的内核日志在 ring buffer 中的位置，但因为它是一个全局的静态变量，当有多个进程要读取 / proc/kmsg 文件时，就会有一个比较严重的问题，即内核日志会被这几个进程随机抢占读取，也就是说，每个进程读到的都是整个内核日志的一部分，是不完整的，这也是 dmesg 命令默认不使用 / proc/kmsg 文件的原因。

第三种查看内核日志的方式，是通过 klogctl 函数。

该函数是 glibc 对 syslog 系统调用的一个简单封装，其具体使用方式，可以参考全景图中用户态的蓝色部分。

klogctl 函数可以指定很多命令，在上图的示例中，我们使用的是 SYSLOG_ACTION_READ 命令，以此来模拟 cat /proc/kmsg 行为。

其实在内核层面，cat /proc/kmsg 命令，使用的就是 klogctl 对应的 syslog 系统调用的 SYSLOG_ACTION_READ 命令的处理逻辑，所以示例中的 klogctl 函数相关代码，和 cat /proc/kmsg 命令其实是等价的。

也就是说，klogctl 函数在内核里使用的也是 syslog_seq 变量，它也有和 / proc/kmsg 文件同样的问题。

其实还有一种方式可以查看内核日志，就是通过系统控制台。

但这种方式和前面讲的三种方式都不一样，它是完全被动的，是内核在调用 printk 函数，将日志信息放到 ring buffer 后，再去通知系统控制台，告知其可以输出这些日志。

系统控制台也是通过一个 console_seq 变量，记录下一条要输出内核日志的所在位置。

这里说的系统控制台，是指我们在开机的时候，黑色屏幕输出的那些内容，但当我们进入图形化界面后，我们就看不到系统控制台的输出了，除非我们再用 ctrl + alt + f1/f2/f3 等方式，切换成系统控制台。  

系统控制台输出的内容，是被日志级别过滤过的，内核默认的日志过滤级别是 7，即 debug 级别以上的日志，比如 info / err 等，这些都会输出，但 debug 级别不会输出。

该日志过滤级别，可以通过很多方式改变，比如说，可以通过内核参数 loglevel，所以，如果发现系统控制台没有输出想要的日志信息，先看下其是否被过滤掉了。

以上就是 printk 生态的完整实现。  

了解 printk 函数的实现，对于内核开发者或研究者来说，意义非常大，但对于普通的应用开发人员来说，又有什么帮助呢？

其实，随着技术的深入，我们不应该再只关心应用层面的行为，而且还要关心系统层面的行为，这样我们才能更好的去定位问题，更好的去保证我们应用的健康运行。  

比如，当我们的应用需要内存时，会向操作系统申请，操作系统此时给我们的，其实是虚拟内存，只有当我们的进程真正的在使用这些内存时，比如读 / 写，操作系统才会为其分配物理内存。

但假设此时物理内存没有了，那操作系统会怎么办？

对于 linux 内核来说，它会选择一个使用内存最多的进程，然后将其 kill 掉，以此来释放内存，保证后续的内存分配操作能够成功，这个我在之前文章 [为什么我的进程被 kill 掉了](http://mp.weixin.qq.com/s?__biz=MzUxNDUwOTc0Nw==&mid=2247485098&idx=1&sn=46297f0b18fde11d8bc9335bb46eb611&chksm=f9459876ce321160d9dbfae34766783309b180d2837fb14808b17d199c02ba78ba2bd6105469&scene=21#wechat_redirect) 有详细讲过。

对于内核的这种行为，我们就应该多加关注，而关注的方式，就是查看内核日志。  

比如，linux 内核在 kill 掉进程时，会用 pr_err 记录一行日志：

![](https://mmbiz.qpic.cn/mmbiz_png/z79xARvmm9jdeqnzZzRapYdzqCltetc65tWm6zf7sqknDX3oFGhEVrXVaRxopFmOSZ02Uo1Dy9boIeHtTXYEeQ/640?wx_fmt=png)

如果我们发现一个进程跑着跑着就没有了，就可以通过 dmesg 命令，查看是否有这个日志，如果有，说明该进程因为系统内存不足，被操作系统 kill 掉了。

类似的，内核里还有很多 error 级别，甚至更高级别的日志需要我们关注，通过这些日志，我们可以及时的发现系统的异常情况，必要时可以人工介入进行干预。

总之，对系统了解的越深，内核日志对我们的帮助就越大。

就这些，希望你喜欢。