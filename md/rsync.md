> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/george-guo/p/7718515.html) 一、简介

1、认识

Rsync（remote synchronize）是一个远程数据同步工具，可通过 LAN/WAN 快速同步多台主机间的文件。Rsync 使用所谓的 “Rsync 算法” 来使本地和远 程两个主机之间的文件达到同步，这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快

Rsync 支持大多数的类 Unix 系统，无论是 Linux、Solaris 还是 BSD 上都经过了良好的测试

此外，它在 windows 平台下也有相应的版本，如 cwRsync 和 Sync2NAS 等工具

2、原理

Rsync 本来是用于替代 rcp 的一个工具，目前由 rsync.samba.org 维护，所以 rsync.conf 文件的格式类似于 samba 的主配 置文件；Rsync 可以通过 rsh 或 ssh 使用，也能以 daemon 模式去运行

在以 daemon 方式运行时 Rsync server 会打开一个 873 端口，等待客户端去连接。连接时，Rsync server 会检查口令是否相符，若通过口令查核，则可以开始进行文件传输。第一次连通完成时，会把整份文件传输一次，以后则就只需进行增量备份

3、特点

1、可以镜像保存整个目录树和文件系统；

2、可以很容易做到保持原来文件的权限、时间、软硬链接等；

3、无须特殊权限即可安装；

4、优化的流程，文件传输效率高；

5、可以使用 rsh、ssh 等方式来传输文件，当然也可以通过直接的 socket 连接；

6、支持匿名传输

 

二、ssh 模式

1、本地间同步

环境： 172.16.22.12

# mkdir src

# touch src/{1,2,3,4}

# mkdir dest

# rsync -av src/ dest/ -- 将 src 目录里的所有的文件同步至 dest 目录（不包含 src 本身）

# rsync -av src dest/ -- 将 src 目录包括自己整个同步至 dest 目录

# rsync -avR src/ dest/ -- 即使 src 后面接有 / ，效果同上

2、局域网间同步

环境： 172.16.22.11

# mkdir src

# touch src/{a,b,c,d}

# mkdir dest

# rsync -av 172.16.22.12:/data/test/src/ dest/ -- 远程同步至本地，需输入 root 密码

# rsync -av src/ 172.16.22.12:/data/test/dest/ -- 本地文件同步至远程

# rsync -av src 172.16.22.12:/data/test/dest/ -- 整个目录同步过去

# rm -rf src/d -- 删除一个文件 d

# rsync -av --delete src/ 172.16.22.12:/data/test/dest/ --delete，从目标目录里面删除无关的文件

3、局域网指定用户同步

--172.16.22.12

# useradd george

# passwd george

# mkdir /home/george/test

# touch /home/george/test/g{1,2,3,4}

--172.16.22.11

# rsync -av src '-e ssh -l george' 172.16.22.12:/home/george -- 本地同步至远程

# rsync -av 172.16.22.12:/home/george/test/g* '-e ssh -l george -p 22' dest/

 

三、daemon 模式

环境：192.168.22.11

1、服务启动方式

1.1、对于负荷较重的 rsync 服务器应该使用独立运行方式

# yum install rsync xinetd -- 服务安装

# /usr/bin/rsync --daemon

1.2、对于负荷较轻的 rsync 服务器可以使用 xinetd 运行方式

# yum install rsync xinetd -- 服务安装

# vim /etc/xinetd.d/rsync -- 配置托管服务，将下项改为 no

disable = no

# /etc/init.d/xinetd start -- 启动托管服务 xinetd

# chkconfig rsync on

# netstat -ntpl | grep 873 -- 查看服务是否启动

 

2、配置详解

两种 rsync 服务运行方式都需要配置 rsyncd.conf，其格式类似于 samba 的主配置文件

全局参数

在全局参数部分也可以定义模块参数，这时该参数的值就是所有模块的默认值

address -- 在独立运行时，用于指定的服务器运行的 IP 地址；由 xinetd 运行时将忽略此参数，使用命令行上的 –address 选项替代。默认本地所有 IP

port -- 指定 rsync 守护进程监听的端口号。 由 xinetd 运行时将忽略此参数，使用命令行上的 –port 选项替代。默认 873

motd file -- 指定一个消息文件，当客户连接服务器时该文件的内容显示给客户

pid file --rsync 的守护进程将其 PID 写入指定的文件

log file -- 指定 rsync 守护进程的日志文件，而不将日志发送给 syslog

syslog facility -- 指定 rsync 发送日志消息给 syslog 时的消息级别

socket options -- 指定自定义 TCP 选项

lockfile -- 指定 rsync 的锁文件存放路径

timeout = 600 -- 超时时间

模块参数

模块参数主要用于定义 rsync 服务器哪个目录要被同步。模块声明的格式必须为 [module] 形式，这个名字就是在 rsync 客户端看到的名字，类似于 Samba 服务器提供的共享名。而服务器真正同步的数据是通过 path 来指定的

基本模块参数

path -- 指定当前模块在 rsync 服务器上的同步路径，该参数是必须指定的

comment -- 给模块指定一个描述，该描述连同模块名在客户连接得到模块列表时显示给客户

模块控制参数

use chroot = -- 默认为 true，在传输文件之前首先 chroot 到 path 参数所指定的目录下；优点，安全；缺点，需要 root 权限，不能备份指向 path 外部的符号连接所指向的目录文件

uid = -- 指定该模块以指定的 UID 传输文件；默认 nobody

gid = -- 指定该模块以指定的 GID 传输文件；默认 nobody

max connections -- 最大并发连接数，0 为不限制

lock file -- 指定支持 max connections 参数的锁文件。默认 /var/run/rsyncd.lock

list -- 指定当客户请求列出可以使用的模块列表时，该模块是否应该被列出。默认为 true，显示

read only = -- 只读选择，也就是说，不让客户端上传文件到服务器上。默认 true

write only = -- 只写选择，也就是说，不让客户端从服务器上下载文件。默认 false

ignore errors -- 忽略 IO 错误。默认 true

ignore nonreadable -- 指定 rysnc 服务器完全忽略那些用户没有访问权限的文件。这对于在需要备份的目录中有些不应该被备份者获得的文件时是有意义的。 false

timeout = -- 该选项可以覆盖客户指定的 IP 超时时间。从而确保 rsync 服务器不会永远等待一个崩溃的客户端。对于匿名 rsync 服务器来说，理想的数字是 600（单位为秒）。 0 (未限制)

dont compress -- 用来指定那些在传输之前不进行压缩处理的文件。该选项可以定义一些不允许客户对该模块使用的命令选项列表。必须使用选项全名，而不能是简称。当发生拒绝某个选项的情况时，服务器将报告错误信息然后退出。例如，要防止使用压缩，应该是：”dont compress = *”。 *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz

模块文件筛选参数

exclude -- 指定多个由空格隔开的多个文件或目录 (相对路径)，并将其添加到 exclude 列表中。这等同于在客户端命令中使用 –exclude 来指定模式

exclude from -- 指定一个包含 exclude 规则定义的文件名，服务器从该文件中读取 exclude 列表定义

include -- 指定多个由空格隔开的多个文件或目录 (相对路径)，并将其添加到 include 列表中。这等同于在客户端命令中使用 –include 来指定模式

include from -- 指定一个包含 include 规则定义的文件名，服务器从该文件中读取 include 列表定义

模块用户认证参数

auth users -- 指定由空格或逗号分隔的用户名列表，只有这些用户才允许连接该模块（和系统用户没有任何关系）。用户名和口令以明文方式存放在 secrets file 参数指定的文件中。默认为匿名方式

secrets file -- 指定一个 rsync 认证口令文件。只有在 auth users 被定义时，该文件才起作用。文件权限必须是 600

strict modes -- 指定是否监测口令文件的权限。为 true 则口令文件只能被 rsync 服务器运行身份的用户访问，其他任何用户不可以访问该文件。默认为 true

模块访问控制参数

hosts allow -- 用一个主机列表指定哪些主机客户允许连接该模块。不匹配主机列表的主机将被拒绝。默认值为 *

hosts deny -- 用一个主机列表指定哪些主机客户不允许连接该模块

模块日志参数

transfer logging -- 使 rsync 服务器将传输操作记录到传输日志文件。默认值为 false

log format -- 指定传输日志文件的字段。默认为：”%o %h [%a] %m (%u) %f %l”

设置了”log file” 参数时，在日志每行的开始会添加”%t [%p]“；

可以使用的日志格式定义符如下所示：

%o -- 操作类型：”send” 或 “recv”

%h -- 远程主机名

%a -- 远程 IP 地址

%m -- 模块名

%u -- 证的用户名（匿名时是 null）

%f -- 文件名

%l -- 文件长度字符数

%p -- 该次 rsync 会话的 PID

%P -- 模块路径

%t -- 当前时间

%b -- 实际传输的字节数

%c -- 当发送文件时，记录该文件的校验码

3、服务端配置

# vim /etc/rsyncd.conf -- 为 rsyncd 服务编辑配置文件，默认没有，需自己编辑

uid = root --rsync 运行权限为 root

gid = root --rsync 运行权限为 root

use chroot = no -- 是否让进程离开工作目录

max connections = 5 -- 最大并发连接数，0 为不限制

timeout = 600 -- 超时时间

pid file = /var/run/rsyncd.pid -- 指定 rsync 的 pid 存放路径

lockfile = /var/run/rsyncd.lock -- 指定 rsync 的锁文件存放路径

log file = /var/log/rsyncd.log -- 指定 rsync 的日志存放路径

[web1] -- 模块名称

path = /data/test/src -- 该模块存放文件的基础路径

ignore errors = yes -- 忽略一些无关的 I/O 错误

read only = no -- 客户端可以上传

write only = no -- 客户端可以下载

hosts allow = 192.168.22.12 -- 允许连接的客户端主机 ip

hosts deny = * -- 黑名单，* 表示任何主机

list = yes

auth users = web -- 认证此模块的用户名

secrets file = /etc/web.passwd -- 指定存放 “用户名：密码” 格式的文件

# mkdir /data/test/src -- 创建基础目录

# mkdir /data/test/src/george -- 再创建一个目录

# touch /data/test/src/{1,2,3}

# echo "web:123" > /etc/web.passwd -- 创建密码文件

# chmod 600 /etc/web.passwd

# service xinetd restart

 

四、测试

1、客户端

环境：192.168.22.12

# yum -y install rsync

# mkdir /data/test

2、小试参数

# rsync -avzP web@192.168.22.11::web1 /data/test/ -- 输入密码 123；将服务器 web1 模块里的文件同步至 /data/test，参数说明：

-a -- 参数，相当于 - rlptgoD，

-r -- 是递归

-l -- 是链接文件，意思是拷贝链接文件

-i -- 列出 rsync 服务器中的文件

-p -- 表示保持文件原有权限

-t -- 保持文件原有时间

-g -- 保持文件原有用户组

-o -- 保持文件原有属主

-D -- 相当于块设备文件

-z -- 传输时压缩

-P -- 传输进度

-v -- 传输时的进度等信息，和 - P 有点关系

# rsync -avzP --delete web@192.168.22.11::web1 /data/test/ -- 让客户端与服务器保持完全一致， --delete

# rsync -avzP --delete /data/test/ web@192.168.22.11::web1 -- 上传客户端文件至服务端

# rsync -avzP --delete /data/test/ web@192.168.22.11::web1/george -- 上传客户端文件至服务端的 george 目录

# rsync -ir --password-file=/tmp/rsync.password web@192.168.22.11::web1 -- 递归列出服务端 web1 模块的文件

# rsync -avzP --exclude="*3*" --password-file=/tmp/rsync.password web@192.168.22.11::web1 /data/test/ -- 同步除了路径以及文件名中包含 “3” * 的所有文件

3、通过密码文件同步

# echo "123"> /tmp/rsync.password

# chmod 600 /tmp/rsync.password

# rsync -avzP --delete --password-file=/tmp/rsync.password web@192.168.22.11::web1 /data/test/ -- 调用密码文件

4、客户端自动同步

# crontab -e

10 0 * * * rsync -avzP --delete --password-file=/tmp/rsync.password web@192.168.22.11::web1 /data/test/

# crontab -l

 

五、数据实时同步

环境：Rsync + Inotify-tools

1、inotify-tools

是为 linux 下 inotify 文件监控工具提供的一套 c 的开发接口库函数，同时还提供了一系列的命令行工具，这些工具可以用来监控文件系统的事件

inotify-tools 是用 c 编写的，除了要求内核支持 inotify 外，不依赖于其他

inotify-tools 提供两种工具：一是 inotifywait，它是用来监控文件或目录的变化，二是 inotifywatch，它是用来统计文件系统访问的次数

2、安装 inotify-tools

下载地址：http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz

# yum install –y gcc -- 安装依赖

# mkdir /usr/local/inotify

# tar -xf inotify-tools-3.14.tar.gz

# cd inotify-tools-3.14

# ./configure --prefix=/usr/local/inotify/

# make && make install

3、设置环境变量

# vim /root/.bash_profile

export PATH=/usr/local/inotify/bin/:$PATH

# source /root/.bash_profile

# echo '/usr/local/inotify/lib' >> /etc/ld.so.conf -- 加载库文件

# ldconfig

# ln -s /usr/local/inotify/include /usr/include/inotify

4、常用参数

-m -- 始终保持监听状态，默认触发事件即退出 -r -- 递归查询目录 -q -- 打印出监控事件 -e -- 定义监控的事件，可用参数： access -- 访问文件 modify -- 修改文件 attrib -- 属性变更 open -- 打开文件 delete -- 删除文件 create -- 新建文件 move -- 文件移动 --fromfile -- 从文件读取需要监视的文件或者排除的文件，一个文件一行，排除的文件以 @开头 --timefmt -- 时间格式 --format -- 输出格式 --exclude -- 正则匹配需要排除的文件，大小写敏感 --excludei -- 正则匹配需要排除的文件，忽略大小写 %y%m%d %H%M -- 年月日时钟 %T%w%f%e -- 时间路径文件名状态

5、测试一

检测源目录中是否有如下动作：modify,create,move,delete,attrib；一旦发生则发布至目标机器；方式为 ssh

src: 192.168.22.11(Rsync + Inotify-tools) dest: 192.168.22.12

两台机器需要做好 ssh 免密登录

# mdkir /data/test/dest/ --dest 机器

# mdkir /data/test/src/ --src 机器

# rsync -av --delete /data/test/src/ 192.168.22.12:/data/test/dest -- 测试下命令

# vim /data/test/test.sh

#!/bin/bash

/usr/local/inotify/bin/inotifywait -mrq -e modify,create,move,delete,attrib /data/test/src | while read events

do

rsync -a --delete /data/test/src/ 192.168.22.12:/data/test/dest

echo "`date +'%F %T'` 出现事件：$events" >> /tmp/rsync.log 2>&1

done

# chmod 755 /data/test/test.sh

# /data/test/test.sh &

# echo '/data/test/test.sh &' >> /etc/rc.local -- 设置开机自启

******* 我们可以在目标机上也写一个这样的脚本： rsync -a --delete /data/test/dest/ 192.168.22.11:/data/test/src ；这样可以实现双向同步