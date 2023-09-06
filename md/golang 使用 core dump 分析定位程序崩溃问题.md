> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/YiONLLc614BneyCUsHE6pw)

> 一、前言 core dump 是一个包含着意外终止的程序其内存快照的文件。这个文件可以被用来事后调试（debu

****一、前言****

core dump 是一个包含着意外终止的程序其内存快照的文件。这个文件可以被用来事后调试（debugging）以了解为什么会发生崩溃，同时了解其中涉及到的变量。通过 GOTRACEBACK，Go 提供了一个环境变量用于控制程序崩溃时生成的输出信息。这个变量同样可以强制生成 core dump，从而使调试成为可能。

**1.1** **GOTRACEBACK**
=======================

`GOTRACEBACK` 控制程序崩溃时输出的详细程度。它可以采用不同的值：

*   `none` 不显示任何 goroutine 栈 trace。
    
*   `single`, 默认选项，显示当前 goroutine 栈 trace。
    
*   `all` 显示所有用户创建的 goroutine 栈 trace。
    
*   `system` 显示所有 goroutine 栈 trace, 甚至运行时的 trace。
    
*   `crash` 类似 `system`, 而且还会生成 core dump。
    

****二、功能设置****

**2.1** **系统设置**
================

开启 core dump 功能：

```
ulimit -c unlimited

```

设置 core file size = unlimited，core dump file 的文件的大小无限制。也可以根据自己的需要把 core file 设置为特定的大小如：

```
ulimit -c 1024

```

需要注意的是 core file size 的单位是 block， block = 512 字节。因通过 ulimit 设置 core file size 只对当前的终端起作用，当你关闭终端或者重启系统设置会失效。为了避免每次都设置的麻烦，可以在~/.profile 最后添加:

```
echo "ulimit -c unlimited" >> ~/.profile

```

设置 core dump 文件的位置

```
*kernel.core_pattern=/var/core/core%t%p_%e*
%t: 生成的文件的时间戳
%p:发生断言的进程的id
%e:发生断言的的代码文件

```

执行:

```
sysctl -p /etc/sysctl.conf

```

**2.2** **GO 设置**
=================

设置 Go 环境变量

```
export GOTRACEBACK=crash

```

需要注意的是和 ulmit 命令一样，在关闭终端和重启系统后，设置的 GOTRACEBACK 变量会消失，所以要将’export GOTRACEBACK=crash’ 加入到 .profile 文件中

```
echo "export GOTRACEBACK=crash " >> ~/.profile

```

****三、案例分析****

GOTRACEBAK 变量可以控制程序在崩溃时，stack 的输出情况。下面结合具体地程序来分析。

```
package main
import (
  "time"
  "github.com/astaxie/beego/logs"
)
func main() {
  logs.Info("Start...")
  defer logs.Info("exit.")
  i := 0
  c := make(chan int, 1)
  for {
    go func(i int) {
      mem := make([]int, 100*1024*1024)
      logs.Info("i=%d,mem:%p", i, mem)
      mem[0] = <-c
    }(i)
    i++
    time.Sleep(200 * time.Microsecond)
  }
}

```

该程序将很快崩溃，产生如下报错：

```
goroutine 279 [running]:
  goroutine running on other thread; stack unavailable
created by main.main
  /opt/gopath/src/test/coredump_test/testcoredump.go:15 +0xdf
goroutine 290 [running]:
  goroutine running on other thread; stack unavailable
created by main.main
  /opt/gopath/src/test/coredump_test/testcoredump.go:15 +0xdf
Aborted (core dumped)

```

gdb 可以进行调试，查看程序运行的详细情况：

```
 gdb testcoredump core.15956
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-110.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
...
(gdb) start
Temporary breakpoint 1 at 0x618c50: file /opt/gopath/src/test/coredump_test/testcoredump.go, line 9.
Starting program: /opt/gopath/src/test/coredump_test/testcoredump
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
[New Thread 0x7ffff77f1700 (LWP 15980)]
[New Thread 0x7ffff6ff0700 (LWP 15981)]
[New Thread 0x7ffff5fee700 (LWP 15983)]
[New Thread 0x7ffff67ef700 (LWP 15982)]
[New Thread 0x7ffff57ed700 (LWP 15984)]
Temporary breakpoint 1, main.main () at /opt/gopath/src/test/coredump_test/testcoredump.go:9
9  func main() {
(gdb)

```

gdb 常用命令：

```
start    //开始调试
n    //一条一条执行
step/s    //执行下一条，如果函数进入函数
backtrace/bt    //查看函数调用栈帧
info/i locals    //查看当前栈帧局部变量
frame/f    //选择栈帧，再查看局部变量
print/p    //打印变量的值
finish    //运行到当前函数返回
set var sum=0    //修改变量值
list/l 行号或函数名    //列出源码
display/undisplay sum    //每次停下显示变量的值/取消跟踪
break/b  行号或函数名    //设置断点
continue/c    //连续运行
info/i breakpoints    //查看已经设置的断点
delete breakpoints 2    //删除某个断点
disable/enable breakpoints 3    //禁用/启用某个断点
break 7 if ok == true    //满足条件才激活断点
run/r    //重新从程序开头连续执行
watch input[7]    //设置观察点
info/i watchpoints    //查看设置的观察点
x/7b input    //打印存储器内容，b--每个字节一组，7--7组
disassemble    //反汇编当前函数或指定函数
 si    // 一条指令一条指令调试 而 s 是一行一行代码
 info registers    // 显示所有寄存器的当前值
x/20 $esp    //查看内存中开始的20个数

```

****四、总结****

程序崩溃可以通过 coredump 详细地查看程序调用栈的相关信息，可以更迅速的定位到程序的问题，特别是引起程序崩溃的 bug：内存泄漏，一些 panic 等，当然在写程序时尽量多些 log 更方便调试。golang 自带的 pprof 在涉及到 c 库的调用时，会监测不到，这时 coredump 结合 gdb 进行调试会比较有用。