> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1628779129&ver=3248&signature=dnprV7ElNNg4KT4lcvde-rgqGk*GCZFt4mZK2DQ9nwwVu3MMhLaGN0TjTXVP3LTqngh2s-DYZ1HAWHECxn7glv7Yp00ax*Si2xcoORPMf*9WJey3osyfgwK7e4JttE-8&new=1)

上午刚工作 10 分左右，同事说在使用 jira 时出现问题，具体截图如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTiaLORMsAsFicyLgt7HCoVdhOWs7o3kD9XI5Bddn32nqle3koUC7qjLoBibQl4TG70UDRRWrz7kV4Bg/640?wx_fmt=jpeg)

通过上图的报错信息：**定位为 mysql 数据库连接数的问题**

**解决方法：**

**1. 登录 mysql 进行查看**

```
Mysql –uroot –p123456
mysql> show variables like'%max_connections%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151  |
+-----------------+-------+
1 row in set (0.00 sec)
```

很奇怪，最大连接数怎么是 151 呢，mysql 默认的最大连接数不是 100 么？

后来想一下可能是版本不同的问题，默认连接数也不同。

为了确认 mysql5.5.3 默认的最大连接数为 151，去 mysql 官网查看了一下：mysql 默认的最大连接数为 151，上限为 100000。另外，MySQL 系列面试题和答案我都整理好了，关注公众号 Java 技术栈回复 "面试" 获取。

**2. 修改 mysql 默认的最大连接数为 1000**

在 / etc/my.cnf 文件中 [mysqld] 部分增加 max_connections=1000，重启 mysql 服务，问题解决。

**补充 1:mysql 其他版本默认的最大连接数**

Mysql5.5 mysql5.6  mysql5.7：默认的最大连接数都是 151，上限为：100000

![](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTiaLORMsAsFicyLgt7HCoVdhOWs7o3kD9XI5Bddn32nqle3koUC7qjLoBibQl4TG70UDRRWrz7kV4Bg/640?wx_fmt=jpeg)

Mysql5.1 根据其小版本的不同，默认的最大连接数和可修改的连接数上限也有所不同

![](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTiaLORMsAsFicyLgt7HCoVdhqA5ASOd9ZHrdMoLywAVktbNa3v89J8hhVLCGtPsGjic4W9BHu4ZSMug/640?wx_fmt=jpeg)

Mysql5.0 版本：默认的最大连接数为 100，上限为 16384

![](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTiaLORMsAsFicyLgt7HCoVdhOWs7o3kD9XI5Bddn32nqle3koUC7qjLoBibQl4TG70UDRRWrz7kV4Bg/640?wx_fmt=jpeg)

**补充 2：修改 mysql 数据库默认的最大连接数**

[方法一：修改 mysql 的主配置文件 / etc/my.cnf，[mysqld] 部分添加 “max_connections=1000（这个根据实际的需要来进行设置即可）”，重启 mysql 服务。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247486173&idx=2&sn=0603756b36279b5f5f036151af85f9fd&chksm=eb538febdc2406fd70748604c500eeaea584306c137cb2020ab5a72187c9359926015e00a46d&scene=21#wechat_redirect)

[方法二：mysql 客户端登录，通过命令行修改全局变量来进行修改](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247486173&idx=2&sn=0603756b36279b5f5f036151af85f9fd&chksm=eb538febdc2406fd70748604c500eeaea584306c137cb2020ab5a72187c9359926015e00a46d&scene=21#wechat_redirect)

```
mysql -uroot -p123456
mysql> set global_max_connections = 200;
mysql> show processlist;
mysql> show status;
```

修改完成后进行查看，mysql 的最大连接数

```
mysql> show variables like '%max_connections%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 1000  |
+-----------------+-------+
1 row in set (0.00 sec)
```

方法三：解开 mysql 源代码，进入里面的 SQL 目录修改 mysqld.cc 找到下面一行：

```
{"max_connections", OPT_MAX_CONNECTIONS,
  "The number of simultaneous clients allowed.", (gptr*) &max_connections,
  (gptr*) &max_connections, 0, GET_ULONG, REQUIRED_ARG, 100, 1, 16384, 0, 1,
  0},
```

把它改为：

```
{"max_connections", OPT_MAX_CONNECTIONS,
  "The number of simultaneous clients allowed.", (gptr*) &max_connections,
  (gptr*) &max_connections, 0, GET_ULONG, REQUIRED_ARG, 1500, 1, 16384, 0, 1,
  0},
```

保存退出，然后./configure ;make;make install 可以获得同样的效果

方法四：通过修改 mysqld_safe 来修改 mysql 的连接数

编辑 mysqld_safe 配置文件，找到如下内容：

```
then $NOHUP_NICENESS $ledir/$MYSQLD
  $defaults --basedir=$MY_BASEDIR_VERSION
  --datadir=$DATADIR $USER_OPTION
  --pid-file=$pid_file
  --skip-external-locking
  -O max_connections=1500
  >> $err_log 2>&1 else
  eval "$NOHUP_NICENESS $ledir/$MYSQLD
  $defaults --basedir=$MY_BASEDIR_VERSION
  --datadir=$DATADIR $USER_OPTION
  --pid-file=$pid_file
  --skip-external-locking $args
  -O max_connections=1500 >>
  $err_log 2>&1"
```

保存退出并重启 mysql 服务。

参考文章：

> http://blog.chinaunix.net/uid-20592013-id-94956.html  
> http://blog.csdn.net/tongle_deng/article/details/6932733

作者：mohan87821000  
来源：https://blog.51cto.com/5250070/1672803