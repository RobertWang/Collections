> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_42057852/article/details/81129096)

###### 我相信不少人有这个需求，但是每次命令记不住啊，怎么办？还能怎么办，好记性不如烂笔头，脑子记不住，就写下来啊

一、为什么要使程序在后台执行
--------------

1：我们这边是否关机不影响日本那边的程序运行。（不会像以前那样，我们这网络一断开，或一关机，程序就断掉或找不到数据，跑了几天的程序只能重头再来，很是烦恼）

2：不影响计算效率

3：让程序在后台跑后，不会占据终端，我们可以用终端做别的事情。

二、怎么样使程序在后台执行
-------------

方法有很多，这里主要列举两种。假如我们有程序`pso.cpp`, 通过编译后产生可执行文件`pso`，我们要使`pso`在`linux`服务器后台执行。当客户端关机后重新登入服务器后继续查看本来在终端输出的运行结果。（假设操作都在当前目录下）

方法 1 在终端输入命令：

```
$ ./pso > log.file 2>&1 &
```

解释：将`pso`直接放在后台运行，并把终端输出存放在当前目录下的`log.file`文件中。

当客户端关机后重新登陆服务器后，直接查看`pso.file`文件就可看执行结果（命令：$ `cat pso.file` ）。  
方法 2 在终端输入命令：

```
$ nohup  ./pso > pso.file 2>&1 &
```

解释：`nohup`就是不挂起的意思，将`pso`直接放在后台运行，并把终端输出存放在当前

目录下的`pso.file`文件中。当客户端关机后重新登陆服务器后，直接查看`pso.file`

文件就可看执行结果（命令：`#cat pso.file` ）。

三、常用任务管理命令
----------

```
$ jobs      //查看任务，返回任务编号n和进程号

$ bg  %n   //将编号为n的任务转后台运行

$ fg  %n   //将编号为n的任务转前台运行

$ ctrl+z    //挂起当前任务

$ ctrl+c    //结束当前任务
```

注：如果要使在前天执行任务放到后台运行，则先要用`ctrl+z`挂起该任务，然后用`bg`使之后台执行。  
附：  
在`Linux`中，如果要让进程在后台运行，一般情况下，我们在命令后面加上`&`即可，实际上，这样是将命令放入到一个作业队列中了：

```
$ ./test.sh &
[1] 17208

$ jobs -l
[1]+ 17208 Running                 ./test.sh &
```

对于已经在前台执行的命令，也可以重新放到后台执行，首先按`ctrl+z`暂停已经运行的进程，然后使用`bg`命令将停止的作业放到后台运行：

```
$ ./test.sh
[1]+  Stopped                 ./test.sh

$ bg %1
[1]+ ./test.sh &

$ jobs -l
[1]+ 22794 Running                 ./test.sh &
```

但是如上方到后台执行的进程，其父进程还是当前终端`shell`的进程，而一旦父进程退出，则会发送`hangup`信号给所有子进程，子进程收到`hangup`以后也会退出。如果我们要在退出`shell`的时候继续运行进程，则需要使用`nohup`忽略`hangup`信号，或者`setsid`将将父进程设为`init`进程 (进程号为`1`)

```
$ echo $$

$ nohup ./test.sh &
[1] 29016

$ ps -ef | grep test
515      29710 21734  0 11:47 pts/12   00:00:00 /bin/sh ./test.sh
515      29713 21734  0 11:47 pts/12   00:00:00 grep test
$ setsid ./test.sh &
[1] 409

$ ps -ef | grep test
515        410     1  0 11:49 ?        00:00:00 /bin/sh ./test.sh
515        413 21734  0 11:49 pts/12   00:00:00 grep test
```

上面的试验演示了使用`nohup/setsid`加上`&`使进程在后台运行，同时不受当前`shell`退出的影响。那么对于已经在后台运行的进程，该怎么办呢？可以使用`disown`命令：

```
$ ./test.sh &
[1] 2539

$ jobs -l
[1]+  2539 Running                 ./test.sh &

$ disown -h %1

$ ps -ef | grep test
515        410     1  0 11:49 ?        00:00:00 /bin/sh ./test.sh
515       2542 21734  0 11:52 pts/12   00:00:00 grep test
```

另外还有一种方法，即使将进程在一个`subshell`中执行，其实这和`setsid`异曲同工。方法很简单，将命令用括号 () 括起来即可：

```
$ (./test.sh &)

$ ps -ef | grep test
515        410     1  0 11:49 ?        00:00:00 /bin/sh ./test.sh
515      12483 21734  0 11:59 pts/12   00:00:00 grep test
```

注：本文试验环境为`Red Hat Enterprise Linux AS release 4 (Nahant Update 5)`, `shell`为`/bin/bash`，不同的`OS`和`shell`可能命令有些不一样。例如`AIX`的`ksh`，没有`disown`，但是可以使用`nohup -p PID`来获得`disown`同样的效果。

还有一种更加强大的方式是使用`screen`，首先创建一个断开模式的虚拟终端，然后用`-r`选项重新连接这个虚拟终端，在其中执行的任何命令，都能达到`nohup`的效果，这在有多个命令需要在后台连续执行的时候比较方便：

```
$ screen -dmS screen_test

$ screen -list
There is a screen on:
        27963.screen_test       (Detached)
1 Socket in /tmp/uscreens/S-jiangfeng.

$ screen -r screen_test
```
