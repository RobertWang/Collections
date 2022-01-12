> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [developer.aliyun.com](https://developer.aliyun.com/article/244324?spm=a2c6h.13813017.content3.3.53e96443vVe25m)

> strace 和 gdb 是 Linux 环境下的两个常用调试工具，这里是个人在使用过程中对这两个工具常用参数的总结，留作日后查看使用。

strace 和 gdb 是 Linux 环境下的两个常用调试工具，这里是个人在使用过程中对这两个工具常用参数的总结，留作日后查看使用。  

strace 工具用于跟踪进程执行时的系统调用和所接收的信号，包括参数、返回值、执行时间。在 Linux 中，用户程序要访问系统设备，必须由用户态切换到内核态，这是通过系统调用发起并完成的。

**strace 常用参数：**

-c　　统计每种系统调用执行的时间、调用次数、出错次数，程序退出时给出报告

-p pid　　跟踪指定的进程，可以使用多个 - p 同时跟踪多个进程

-o filename　　strace 默认输出到 stdout，-o 可以将输出写入到指定的文件

-f　　跟踪由 fork 产生的子进程的系统调用

-ff　　常与 - o 选项一起使用，不同进程 (子进程) 产生的系统调用输出到各个 filename.pid 文件中

-F　　尝试跟踪 vfork 子进程系统调用，注意：与 - f 同时使用时, vfork 不被跟踪

-e expr　　输出过滤表达式，可以过滤掉不想输出的 strace 结果

-e trace=set　　指定跟踪 set 中的系统调用

-e trace=network　　跟踪与网络有关的所有系统调用

-e strace=signal　　跟踪所有与系统信号有关的系统调用

-e trace=ipc　　跟踪所有与进程通讯有关的系统调用

-e signal=set　　指定跟踪 set 中的信号

-e read=set　　输出从指定文件中读出的数据，例如 - e read=3,5

-e write=set　　输出写入到指定文件中的数据，例如 - e write=1

-r　　打印每一个系统调用的相对时间

-t　　在输出中的每一行前加上时间信息

-tt　　在输出中的每一行前加上时间信息，时间精确到微秒级

-ttt　　在输出中的每一行前加上时间信息，输出为相对时间

-s　　指定每一行输出字符串的长度（默认为 32）

**strace 使用举例：**

strace -t whoami  #跟踪 whoami 可执行程序，每行输出结果前打印执行的时间

strace -p 17151 -p 17152 -p 17153  #同时跟踪进程 17151、17152、17153  

strace -f -e trace=read,write -p 17151 -o log  #跟踪进程 17151 及子进程中 read 和 write 系统调用，输出到 log 文件

GDB 是 GNU 开源组织发布的一个强大的 UNIX 下的程序调试工具。gcc 编译时加上 - g 参数，可以使可执行程序加上 gdb 调试信息。

**（1）info**

简写：i，列出 gdb 子命令的信息，如 info break，info variables，info stack 等。

**（2）list [file:]function  
**

简写：l，查看当前行的上下文，默认为 10 行，也可以设置在某个函数处列出源码。

**（3）edit [file:]function  
**

简写：e，编辑当前所在的行，也可以编辑某个函数的源码。

**（4）break [file:]function  
**

简写：b，设置断点，可以设置在某行或某个函数处。

**（5）run [arglist]  
**

简写：r，运行程序至断点处停住，run 命令之后可以加上调试程序需要的参数。

**（6）next**

简写：n，单条语句执行。

**（7）continue**

简写：c，继续运行程序至下一个断点。

**（8）print**

简写：p，打印变量的值。

**（9）bt**

查看函数堆栈信息。

**（10）enter**

回车键，重复上一次调试命令。

**（11）help [name]**

显示指定的 gdb 命令的帮助信息。

**（12）quit**

简写：q，退出 gdb。
