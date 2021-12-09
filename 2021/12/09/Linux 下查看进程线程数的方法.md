> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1639033334&ver=3485&signature=SMS8kpJjmcbgWhsT*clE7EIXa6Uz27MfJ-pPoJhLNM1LpofcGoLMSd5kPZGKjMJmjLf8PDdtNsCr9V9OCHBA0XSiznl4FK23kBUh9lPReu2rrEy5rI8NCAb5OBkXk1Hm&new=1)

阅读文本大概需要 3 分钟。

**0x01：ps -ef 只打印进程，而 ps -eLf 会打印所有的线程**

```
[root@centos6 ~]# ps -ef | grep rsyslogd
root      1470     1  0  2011 ?        00:01:13 /sbin/rsyslogd -c 4
root     29865 28596  0 22:45 pts/5    00:00:00 grep rsyslogd
[root@centos6 ~]# ps -eLf | grep rsyslogd
root      1470     1  1470  0    5  2011 ?        00:00:00 /sbin/rsyslogd -c 4
root      1470     1 28631  0    5 Mar04 ?        00:00:04 /sbin/rsyslogd -c 4
root      1470     1 28632  0    5 Mar04 ?        00:00:01 /sbin/rsyslogd -c 4
root      1470     1 28633  0    5 Mar04 ?        00:00:04 /sbin/rsyslogd -c 4
root      1470     1 28636  0    5 Mar04 ?        00:00:00 /sbin/rsyslogd -c 4
root     29867 28596 29867  0    1 22:45 pts/5    00:00:00 grep rsyslogd


```

rsyslogd 这个进程有 5 个线程，所以 ps -ef 只有一行，而 ps -eLf 就有 5 行

ps -eLf 各字段含义

*   UID：用户 ID
    
*   PID：process id 进程 id
    
*   PPID: parent process id 父进程 id
    
*   LWP：表示这是个线程；要么是主线程 (进程)，要么是线程
    
*   NLWP: num of light weight process 轻量级进程数量，即线程数量
    
*   STIME: start time 启动时间
    
*   TIME: 占用的 CPU 总时间
    
*   TTY：该进程是在哪个终端运行的; pts/0255 代表虚拟终端，一般是远程连接的终端; tty1tty7 代表本地控制台终端
    
*   CMD：进程的启动命令
    

**0x02：top -H -p ${pid} 或者 top -p ${pid} 然后 shitf + H**

![](https://mmbiz.qpic.cn/mmbiz_png/gjnldtnoHOqlKWaBjSq6RbLbYbYp6ria42RoqicEGAyMuPft9w1a1EDqWDszd5licCibKF7aYpPiaDjXS9E4j2f18fQ/640?wx_fmt=png)

**0x03：cat /proc/${pid}/status  或者  ls /proc/${pid}/task**

![](https://mmbiz.qpic.cn/mmbiz_png/gjnldtnoHOqlKWaBjSq6RbLbYbYp6ria4KTPSicPsnMZBYFrNMGVbxzzw3SIUu6TafeZaFqVicujqMr2nrYx2BExw/640?wx_fmt=png)

其中 Threads 后面跟的就是线程数  

![](https://mmbiz.qpic.cn/mmbiz_png/gjnldtnoHOqlKWaBjSq6RbLbYbYp6ria4L1DKwd8fHicYRnUNj33YBo5ManekxaUvoGtxB1o7Jia643BnccVgxENw/640?wx_fmt=png)

**0x04：pstree -p ${pid}**

![](https://mmbiz.qpic.cn/mmbiz_png/gjnldtnoHOqlKWaBjSq6RbLbYbYp6ria4NYHuycRsI1aribBr17X58bWOuEXiarKvCaRvJwWQK0PIelSjjOYBb15Q/640?wx_fmt=png)

**0x05：ps -hH -p ${pid}**

```
[root@localhost ~]# ps -hH -p 1414
 1414 ?        Ssl    0:00 /usr/sbin/rsyslogd -n
 1414 ?        Ssl    0:00 /usr/sbin/rsyslogd -n
 1414 ?        Ssl    0:00 /usr/sbin/rsyslogd -n


```