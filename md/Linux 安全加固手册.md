> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650515487&idx=2&sn=054d833584bbbc1f30b80a12238935bb&chksm=83bacdfbb4cd44ed33c120084fd2415f7064f8074e6def4a70c28833aa0fd590b6205a498d4e&mpshare=1&scene=1&srcid=0623Aslj1Yz5zADAYEmQDnVz&sharer_sharetime=1624415627361&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> **文章来****源：****HACK 之道**

1  身份鉴别
=======

1.1   密码安全策略
------------

操作系统和数据库系统管理用户身份鉴别信息应具有不易被冒用的特点，口令应有复杂度要求并定期更换。

```
# awk -F: '($2 == ""){print $1}' /etc/shadow

```

可用离线破解、暴力字典破解或者密码网站查询出帐号密钥的密码是否是弱口令  

2）修改 vi /etc/login.defs 配置密码周期策略

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zZMf0uM5ThXEnlKWU6Twgk0U9kibxicsZiaIP1qL1bAjP4DSPsXPhGbUcg/640?wx_fmt=png)

此策略只对策略实施后所创建的帐号生效， 以前的帐号还是按 99999 天周期时间来算。

3）/etc/pam.d/system-auth 配置密码复杂度：

在文件中添加如下一行：

```
password requisite  pam_cracklib.so retry=3 difok=2 minlen=8 lcredit=-1 dcredit=-1

```

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zAibNpVBT8zum24cOXUHJZp4FTicjdVWIgfuC7Fe7NQiaZuicbuIF5oJLBA/640?wx_fmt=png)

参数含义如下所示：

difok：本次密码与上次密码至少不同字符数

minlen：密码最小长度，此配置优先于 login.defs 中的 PASS_MAX_DAYS

ucredit：最少大写字母

lcredit：最少小写字母

dcredit：最少数字

retry：重试多少次后返回密码修改错误

【注】用 root 修改其他帐号都不受密码周期及复杂度配置的影响。

1.2   登录失败策略
------------

 应启用登录失败处理功能，可采取结束会话、限制非法登录次数和自动退出等措施。

遭遇密码破解时，暂时锁定帐号，降低密码被猜解的可能性

1）方法一：/etc/pam.d/login 中设定控制台；/etc/pam.d/sshd 中设定 SSH

/etc/pam.d/sshd 中第二行添加下列信息

```
auth required  pam_tally2.so deny=5 lock_time=2 even_deny_root unlock_time=60

```

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zZxdLib8oV789qWxJJG4YSs0uuK3T11CcbGHoqkjXXmoc5JUicNjfhAwQ/640?wx_fmt=png)

########### 参数解释 ############

查看用户登录失败次数

# pam_tally2 --user root

解锁用户

# pam_tally2 -r -u root

even_deny_root    也限制 root 用户 (默认配置就锁定 root 帐号)；     
deny     设置普通用户和 root 用户连续错误登陆的最大次数，超过最大次数，则锁定该用户     
unlock_time     设定普通用户锁定后，多少时间后解锁，单位是秒； 

root_unlock_time      设定 root 用户锁定后，多少时间后解锁，单位是秒；

1.3 安全的远程管理方式
-------------

当对服务器进行远程管理时，应采取必要措施，防止鉴别信息在网络传输过程中被窃听。

防止远程管理过程中，密码等敏感信息被窃听

执行如下语句，查看 telnet 服务是否在运行

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zXAAyqMj3M1vEqA3yeI0HjO2ruQoDgcRCfziceThSDicL4r0bbacS8HPA/640?wx_fmt=png)

禁止 telnet 运行，禁止开机启动，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zwLsIwQZQdkMbehUnRtRZAN5wkWEpOiaMLX1sO13vSpXcIe9a5vn3ZWA/640?wx_fmt=png)

2  访问控制
=======

应及时删除多余的、过期的帐户，避免共享帐户的存在。

删除或禁用临时、过期及可疑的帐号，防止被非法利用。

主要是管理员创建的普通帐号，如：test

# usermod -L user 禁用帐号，帐号无法登录，/etc/shadow 第二栏显示为! 开头

# userdel user 删除 user 用户

# userdel -r user 将删除 user 用户，并且将 / home 目录下的 user 目录一并删除

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2z3r4f8doGpWyaicTI4bH1ZsPxPQpgqlgvU7Et9yDyvQ065bb7JYDxMtg/640?wx_fmt=png)

加固后如图

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zDNEZ2mrk6KXfUaq7aKSWNDRoxibRpL6x9xjeiawgurniaUXUJOgvF3spQ/640?wx_fmt=png)

3  安全审计
=======

3.1   审核策略开启
------------

审计范围应覆盖到服务器和重要客户端上的每个操作系统用户和数据库用户；

开启审核策略，若日后系统出现故障、安全事故则可以查看系统日志文件，排除故障、追查入侵者的信息等。

查看 rsyslog 与 auditd 服务是否开启

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zxhuZpicD0TTmib9t6bJxdrduje1plqqI3kvibMjQdaXiaY96iacdSj4umyg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zfcZHeb9icFauQvtqSyHlb6RYdbNjwXRwfL3PiaM4LwJDnib4JxxZ8FibKA/640?wx_fmt=png)

rsyslog 一般都会开启，auditd 如没开启，执行如下命令：

```
# systemctl start auditd

```

auditd 服务开机启动

```
# systemctl start auditd

```

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zH2oRXoIQJbSXZUmMgg7N71ibX0fyR2WUkjSxdbQ511DRLm7tuWBtF6g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2z9icm5pia0pNWBJcH6wLnhHLb6H7jMibTDeDgfjAe7KFAcLH1N4mdzk4Bg/640?wx_fmt=png) 

3.2   日志属性设置
------------

应保护审计记录，避免受到未预期的删除、修改或覆盖等。

防止重要日志信息被覆盖

让日志文件转储一个月，保留 6 个月的信息，先查看目前配置，

```
# more /etc/logrotate.conf | grep -v "^#\|^$"

```

需要修改配置为下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zvsicw7Rofay4y0hd0ghg7WbRWyRDZEH12rLon8aibYVS9XnibdXloUbNA/640?wx_fmt=png)

4  入侵防御
=======

操作系统遵循最小安装的原则，仅安装需要的组件和应用程序，并通过设置升级服务器等方式保持系统补丁及时得到更新。

关闭与系统业务无关或不必要的服务，减小系统被黑客被攻击、渗透的风险。

禁用蓝牙服务

```
# systemctl stop bluetooth

```

禁止蓝牙开机启动  

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2z3U55vnic4yg1Tlj55lYE7YldjzlZNnrf2WmZaRxABZVC9Sp5iakusicJQ/640?wx_fmt=png)

5  系统资源控制  

============

5.1   访问控制
----------

应通过设定终端接入方式、网络地址范围等条件限制终端登录。

对接入服务器的 IP、方式等进行限制，可以阻止非法入侵。

1) 在 / etc/hosts.allow 和 / etc/hosts.deny 文件中配置接入限制

最好的策略就是阻止所有的主机在 “/etc/hosts.deny” 文件中加入“ ALL:ALL@ALL, PARANOID ”，然后再在“/etc/hosts.allow” 文件中加入所有允许访问的主机列表。如下操作：

编辑 hosts.deny 文件（vi /etc/hosts.deny），加入下面该行：

```
# Deny access to everyone. 
ALL: ALL@ALL, PARANOID

```

编辑 hosts.allow 文件（vi /etc/hosts.allow），加入允许访问的主机列表，比如：

ftp: 202.54.15.99 foo.com    //202.54.15.99 是允许访问 ftp 服务的 IP 地址

//foo.com 是允许访问 ftp 服务的主机名称。

 ![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2za66bL5EWnFT3fftN7qeiaqS9qfdFfYxwGcSUiaiccPVibGJmwEa9J41Pjg/640?wx_fmt=png) 

2) 也可以用 iptables 进行访问控制 

5.2    超时锁定

应根据安全策略设置登录终端的操作超时锁定。

设置登录超时时间，释放系统资源，也提高服务器的安全性。

/etc/profile 中添加如下一行

```
exprot TMOUT=900  //15分钟
# source /etc/profile

```

改变这项设置后，必须先注销用户，再用该用户登录才能激活这个功能。

如果有需要，开启屏幕保护功能

设置屏幕保护：设置 -> 系统设置 -> 屏幕保护程序，进行操作

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zyDSsUAKTKW5C59xRh05tYMzstajOz3g5d47lNe5cGAT9aePF3ob3uA/640?wx_fmt=png)

6   最佳经验实践  

=============

对 Linux 系统的安全性提升有一定帮助。

6.1   DOS 攻击防御
--------------

防止拒绝服务攻击

TCP SYN 保护机制等设置

1）打开 syncookie：

# echo“1”>/proc/sys/net/ipv4/tcp_syncookies  // 默认为 1，一般不用设置

表示开启 SYN Cookies。当出现 SYN 等待队列溢出时，启用 cookies 来处理，可防范少量 SYN 攻击，默认为 0，表示关闭；

2）防 syn 攻击优化

用 vi 编辑 / etc/sysctl.conf，添加如下行：

```
net.ipv4.tcp_max_syn_backlog = 2048

```

进入 SYN 包的最大请求队列. 默认 1024. 对重负载服务器, 增加该值显然有好处. 可调整到 2048.

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2zo2SliaiciaWKbte2pN4ZhN68dtHicPH2AJLgwB157WwoK8uSWpdNXoXOCQ/640?wx_fmt=png)

6.2   历史命令  

-------------

为历史的命令增加登录的 IP 地址、执行命令时间等

1）保存 1 万条命令

```
# sed -i 's/^HISTSIZE=1000/HISTSIZE=10000/g' /etc/profile

```

2）在 / etc/profile 的文件尾部添加如下行数配置信息：

```
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}' |sed -e 's/[()]//g'`
if [ "$USER_IP" = "" ]
then
USER_IP=`hostname`
fi
export HISTTIMEFORMAT="%F %T $USER_IP `whoami` "
shopt -s histappend
export PROMPT_COMMAND="history -a"

```

##source /etc/profile 让配置生效

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2vZrVI4y1AoxH44W0PTZ2z2hxqG5fia7JJD2zIy7Hp8YUcl1IFYiaMQ9QDUAvYk0IS0zXXltfpWf0A/640?wx_fmt=png)

**如侵权请私聊公众号删文**

* [带你真正认识 Linux 系统结构](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650514864&idx=3&sn=12936d5c4f915316e5644dcfa78b6292&chksm=83bac054b4cd494201f9df2cd8a671fc4f4c7fd485d101e54bdfeceda48358b88f252f8b260b&scene=21#wechat_redirect)  

* [安全加固之 Linux&Solaris](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650514446&idx=2&sn=f7b382f068209e3d254c29f2c3fe8470&chksm=83bac1eab4cd48fc39849ab02396e7c273ad5222f64ad92731673970e92bda8b5293345d372b&scene=21#wechat_redirect)

*[Linux 内核漏洞详情及其 Exp](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650510324&idx=3&sn=186109eb0603d0c0632633461f883c50&chksm=83baf610b4cd7f0650183c5319103e857ff5dca7d704040934fb318296ef4251cc92a84c92fe&scene=21#wechat_redirect)