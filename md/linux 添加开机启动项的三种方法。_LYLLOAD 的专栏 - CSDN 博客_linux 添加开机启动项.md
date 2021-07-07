> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/lylload/article/details/79488968)

linux 添加开机启动项的三种方法。

（1）编辑文件 /etc/rc.local

输入命令：vim /etc/rc.local 将出现类似如下的文本片段：

#!/bin/sh  
#  
# This script will be executed *after* all the other init scripts.  
# You can put your own initialization stuff in here if you don't  
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local  
/etc/init.d/mysqld start #mysql 开机启动  
/etc/init.d/nginx start #nginx 开机启动  
/etc/init.d/php-fpm start #php-fpm 开机启动  
/etc/init.d/memcached start #memcache 开机启动

#在文件末尾（exit 0 之前）加上你开机需要启动的程序或执行的命令即可（执行的程序需要写绝对路径，添加到系统环境变量的除外），如：

/usr/local/thttpd/sbin/thttpd  -C /usr/local/thttpd/etc/thttpd.conf  
   
（2）自己写一个 shell 脚本

将写好的脚本（.sh 文件）放到目录 /etc/profile.d/ 下，系统启动后就会自动执行该目录下的所有 shell 脚本。

（3）通过 chkconfig 命令设置

将启动文件 cp 到 /etc/init.d / 或者 / etc/rc.d/init.d/（前者是后者的软连接）下

vim 启动文件，文件前面务必添加如下三行代码，否侧会提示 chkconfig 不支持

#!/bin/sh 告诉系统使用的 shell, 所以的 shell 脚本都是这样  
#chkconfig: 35 20 80 分别代表运行级别，启动优先权，关闭优先权，此行代码必须  
#description: http server（自己随便发挥）// 两行都注释掉！！！，此行代码必须

chkconfig --add 脚本文件名 操作后就已经添加了