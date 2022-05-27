> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODc5ODgyMw==&mid=2653588146&idx=1&sn=5a194e17a5b5b5a7368a8b371dd0be1a&chksm=bd1b1e3a8a6c972cbd59e930c37c94bf51ea8464514c6484f6c244d382158b498e015dce648e&scene=21#wechat_redirect)

大家好，这次跟大家谈谈又拍云的操作系统优化方案。往简单地说，我们使用的 Linux 操作系统主要都是基于 CentOS6/7 的精简和优化。往复杂地说，则是我们有两套系统，业务上使用的定制 Linux 系统和数据中心使用的优化版 Linux 系统。

业务上我们使用裁剪过的定制 Linux 系统，目的是为了更安全、更高效、更加贴近业务需求，方便全国各点进行闪电式部署，但这套系统不具备普适性，所以我们今天暂时不谈它。今天主要分享数据中心常用的 Linux 优化版本，因为这个比较通用，适合大家在使用时进行参考。我会从以下几个方面来进行分享。

  

**主机名设定和永久生效**

  

在 CentOS 或 RHEL 中，有以下三种定义的主机名：

*   静态的（static）：“静态” 主机名也称为内核主机名，是系统在启动时从 /etc/hostname 自动初始化的主机名。
    
*   瞬态的（transient）：“瞬态” 主机名是在系统运行时临时分配的主机名如通过 DHCP 或 mDNS 服务器分配。
    
*   灵活的（pretty）：“灵活” 主机名则允许使用自由形式（包括特殊 / 空白字符）的主机名，以展示给终端用户（如 Gemini's Computer）。
    

好的主机名，可以让非运维的机房人员一目了然地了解和定位机器。比如:  用途名 + 省份 + 机房名 + 机柜号 + 编号，示例如下：

```
HOST="DBS-ZJ-FUD-009"
hostnamectl set-hostname --static $HOST
hostnamectl set-hostname --pretty $HOST
hostnamectl set-hostname --transient  $HOST
echo "$HOST" > /proc/sys/kernel/hostname
```

  

**定制远程登录界面**

  

登陆 Linux 的欢迎界面可由 /etc/issue 和 / etc/motd 控制。如下显示，不用登录系统就能一目了然的知道系统版本、CPU 和内存型号容量、运行状态、应用版本号及网络连接情况。

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUEo6xF6sojk2ZykJiap359VRS3ba5Ld2o7JhZn7hV4uQ17t5yzH12wSD7HEZBXey9Qbzia5G1k59Eibg/640?wx_fmt=png)

  

**字符集配置**

  

好的字符集，可以避免终端显示下的乱码。建议使用 en_US.utf8 字符集。

```
# 查看操作系统支持的所有字符集
# locale -a

cat > /etc/locale.conf <<EOF
LANG=en_US.utf8
LC_CTYPE=en_US.utf8
EOF
localectl set-locale LANG=en_US.UTF8
```

  

**常规基础软件安装**

  

因为使用的都是基于 CentOS 的最小化精简安装，安装后会缺少一些常规的基础软件，所以我们会适当补充一些基础软件，以帮助快速排查和定位问题。

```
yum install -y tree ntpdate  bc nc net-tools wget lsof rsync nmon bash-completion iptables-services firewalld sysstat mtr htop bind-utils yum-utils epel-release smartmontools supervisor python-setuptools python-pip pkgconfig
```

  

**时区和时间同步设定**

  

在实际生产环境中，保障服务器时区和时间的一致性非常重要。尤其是分布式系统、多机集群环境、数据库主从备份、以及依赖时间同步的定时任务等场景，时区和时间同步是非常有用的，一旦二者不一致很容易导致各种各样的问题。

```
timedatectl set-timezone Asia/Shanghai
timedatectl set-ntp 1
timedatectl set-local-rtc 0
#timedatectl set-time "2018-08-08 18:08:08"
ntpdate -u cn.pool.ntp.org
```

强烈建议同时把时间同步操作也写入 crontab 里做双保险。

```
# crontab -l
0 * * * * root(ntpdate -o3 192.168.1.10 211.115.194.21 )
```

  

**禁用 SELinux**

  

SELinux 为安全增强型 Linux（Security-Enhanced Linux），主要由美国国家安全局开发。它概念严谨、结构及配置复杂、操作严格。因此在使用时可以视情况决定是否要开启使用，可以在有机密和信息敏感机构等的特殊场景下可以启用。

```
sed -r -i  '/^SELINUX=/s^=.*^=disabled^g' /etc/selinux/config
set enforce 0
```

  

**添加普通用户并 sudo**

  

sudo 是 Linux 系统管理指令，是允许系统管理员让普通用户执行一些或者全部的 root 命令的一个工具，如 halt、reboot、su 等等。 这样不但可以减少 root 用户的登录和管理时间，也能够提高安全性。

需要注意的是，生产环境尽量不要直接用 root，建议先新建普通用户再提升权限。

```
[root@OPS-FDI-020 ~]# useradd shaohy
[root@OPS-FDI-020 ~]# usermod -G wheel  shaohy
[root@OPS-FDI-020 ~]# sed -i '/pam_wheel/s/^#//g'  /etc/pam.d/su
```

为用户 shaohy 添加 sudo，除关机外的其他所有操作：

```
[root@OPS-FDI-020 ~]# visudo
Cmnd_Alias SHUTDOWN = /sbin/halt, /sbin/shutdown, /sbin/poweroff, /sbin/reboot, /sbin/init
shaohy         ALL=(ALL)       ALL,!SHUTDOWN
%wheel         ALL=(ALL)       ALL,!SHUTDOWN    #修改wheel组的权限，禁止关机
Defaults logfile=/var/log/sudo.log
```

  

**配置防火墙规则和 iptables**

  

自 CentOS7 以后都是默认使用 firewalld 管理 netfilter 子系统，需要注意的是底层调用的命令仍然使用了  iptables。二者的区别对比如下：

*   firewalld 可以动态修改单条规则，动态管理规则集，允许更新规则而不破坏现有会话和连接。而 iptables，在修改了规则后必须得全部刷新才可以生效。
    
*   firewalld 使用区域和服务而不是链式规则。
    
*   firewalld 默认是拒绝的，需要设置以后才能放行。而 iptables 默认是允许的，需要拒绝的才去限制。
    

所以在选择时可以考虑不拒绝 firewalld，去尝试接受它。

```
#!/bin/sh
IPS="192.168.0.0/16"
firewall-cmd --zone=public --remove-service=ssh
firewall-cmd --new-zone=openssh --permanent
firewall-cmd --zone=openssh --add-port=22222/tcp --permanent
firewall-cmd --permanent --zone=public --set-target=default
for ip in  $IPS;do
        firewall-cmd --zone=openssh --add-source=$ip --permanent
done
firewall-cmd --reload
firewall-cmd --runtime-to-permanent
```

  

**GPT 分区和分区挂载**

  

又拍云每一台 CDN 服务器上都有大量硬盘，且不说 4T、6T 这类量比较小的，即便是 10T 的也有 12 块之多，所以需要自动化格式化和划分区这些硬盘的运维操作。

早期的主引导记录（Master Boot Record，缩写：MBR），又叫做主引导扇区，也就是计算机开机后访问硬盘时所必须要读取的首个扇区，无法支持大于 2T 的硬盘引导。虽然打了补丁的 MBR 也可以支持大于 2T 的分区，但 GPT 已经成为了新的趋势。相比 MBR 而言，GPT 分区方案有以下特点：

*   GPT 是 UEFI 标准的一部分（UEFI 是一种个人电脑系统规格，用来定义操作系统与系统固件之间的软件界面，作为 BIOS 的替代方案）。
    
*   GPT 分区列表支持最大 128PB（1PB=1024TB）。
    
*   可以定义 128 个分区。
    
*   没有主分区，扩展分区和逻辑分区的概念，所有分区都能格式化。
    

```
#!/bin/sh
DEV=`lsscsi | awk '/HGST/{print $NF}'` # 筛选所有的sata硬盘
i=1
for dev in $DEV;do
        label="/disk/sata0$i"
        echo $dev $label
        parted -m -s $dev rm 1
        parted -m -s $dev mklabel gpt
        parted -m -s $dev mkpart primary ext4 2048s 100%
        partx -a $dev
        ((i++))
        nohup mkfs.ext4 -L $label ${dev}1 >/dev/null &
done
```

  

**ulimit 配额设定**

  

因为 CentOS7 / RHEL7 系统中使用 Systemd 替代了之前的 SysV，导致 /etc/security/limits.conf 文件的配置只适用于通过 PAM 认证登录用户的资源限制，对 systemd 的 service 资源限制不生效。

因为 systemd service 的资源限制，所以我们将全局配置放置于  /etc/systemd/system.conf 和 /etc/systemd/user.conf 中。其中 system.conf 用于系统实例，user.conf 用于用户实例。

```
sed -r -i -e '/DefaultLimitCORE/s^.*^DefaultLimitCORE=infinity^g' -e '/DefaultLimitNOFILE/s^.*^DefaultLimitNOFILE=100000^g' -e '/DefaultLimitNPROC/s^.*^DefaultLimitNPROC=100000^g' /etc/systemd/system.conf
```

加大打开文件数的限制, 默认设置了非 root 用户的最大进程数为 4096。

```
cat > /etc/security/limits.d/20-nproc.conf <<EOF
*       soft  nproc  10240
root    soft  nproc  unlimited
EOF
```

  

**sysctl.conf 配置生效**

  

sysctl.conf 文件配置参数较多且复杂，如果需要详解每一个参数的作用需要很长时间。这里我们之间看一部分常见调优参数来感受一下。主要是集中在网络和 TCP 参数优化、 swap 禁用、文件句柄放大 。  

```
net.ipv4.ip_forward=1
net.ipv4.ip_local_port_range=1000 65535

net.ipv4.tcp_slow_start_after_idle=0
net.ipv4.tcp_no_metrics_save=1
net.ipv4.tcp_rfc1337=1
net.ipv4.tcp_timestamps=1
net.ipv4.tcp_sack=1
net.ipv4.tcp_dsack=1
net.ipv4.tcp_window_scaling=1
net.ipv4.tcp_rmem=4096 102400 16777216
net.ipv4.tcp_wmem=4096 102400 16777216
net.ipv4.tcp_mem=786432 1048576 1572864
net.ipv4.tcp_syncookies=1
net.ipv4.tcp_syn_retries=3
net.ipv4.tcp_synack_retries=5
net.ipv4.tcp_retries1=3
net.ipv4.tcp_retries2=15
net.ipv4.tcp_fin_timeout=30
net.ipv4.tcp_max_syn_backlog=262144                                  
net.ipv4.tcp_max_orphans=262144
net.ipv4.tcp_tw_recycle=0
net.ipv4.tcp_tw_reuse=1
net.ipv4.tcp_keepalive_time=30
net.ipv4.tcp_keepalive_intvl=10
net.ipv4.tcp_keepalive_probes=3
net.ipv4.tcp_max_tw_buckets=600000
net.ipv4.tcp_congestion_control=bbr

net.core.somaxconn=8192
net.core.rmem_default=131072
net.core.wmem_default=131072
net.core.rmem_max=33554432
net.core.wmem_max=33554432
net.core.dev_weight=512
net.core.optmem_max=262144
net.core.netdev_budget=1024
net.core.netdev_max_backlog=262144

vm.swappiness=0
vm.dirty_writeback_centisecs=9000
vm.dirty_expire_centisecs=18000
vm.dirty_background_ratio=5
vm.dirty_ratio=10
vm.overcommit_memory=1
vm.overcommit_ratio=50
vm.max_map_count=200000

fs.file-max=524288
fs.aio-max-nr=1048576
```

  

**/etc/passwd 安全性检查**

  

用户的 shell 权限检查，通常考虑到安全性的问题，不允许有非 root 用户拥有 shell 权限。  

```
# 过滤出有uid==0, gid==0的潜伏用户,判断有没有bash
awk -F: '($3==0||$4==0) {print $0}' /etc/passwd|grep -i bash
# 除root外的用户shell全部为nologin
sed -r -i '/^[^root]/s:/bin/bash:/sbin/nologin:g' /etc/passwd
```

  

**sshd 服务配置**

  

因为几乎所有 Linux 服务器都通过 SSH 来进行远程管理，这很容易招来许多不速之客，想方设法通过 SSH 来取得您的服务器权限。所以 SSH 安全不容忽视！强烈建议禁用口令登录，修改之前先改用公密钥的方式来 SSH 登录管理服务器。  

```
sed -r -i '/#Port 22/s^.*^Port 22222^g;/^PasswordAuthentication/s^yes^no^g' /etc/ssh/sshd_config
```

  

**关闭不必要服务**

  

Linux 服务（Linux services）对于每个应用 Linux 的用户来说都很重要。关闭不必要的服务，可以让 Linux 运行更高效，但并不是所有的 Linux 服务都可以关闭，这个要自己权衡。  

```
systemctl disable network  postfix irqbalance tuned rpcbind.target
```

如果有基于 udp 的服务，如 ntpd、dns，要注意 udp 的反射攻击。因为 udp 的反射攻击破坏力极大，最好关闭掉无用相关的服务, 或者选择高防机房去部署此类服务。

  

**logrotate 缩减轮转日志包**

  

当服务器进程较多，日志文件大小增长较快，就会不断消耗磁盘空间并触发告警。这时就需要人为定期按照各种维度去手动清理日志，如果不及时清理则很容易变成运维事故。  

通常可以使用 logrotate 日志滚动机制将日志文件按时间或大小分成多份，删除时间久远的日志文件，从而节省空间和方便整理。

```
sed -r -i 's@weekly@daily@g;s@^rotate.*@rotate 7@g;s@^#compress.*@compress@g' /etc/logrotate.conf
systemctl daemon-reload; systemctl restart rsyslog
```

  

**journalctl 调整 journal 日志**

  

在 Systemd 出现之前，Linux 系统及各应用的日志都是分别管理的，而 Systemd 统一管理了所有 Unit 启动日志。这样的好处就是可以只用一个 journalctl 命令，查看所有内核和应用的日志。  

适当的配置可以让 journal 体积可控，不至于容量爆炸。

```
sed -r -i -e '/Compress=/s@.*@Compress=yes@g; /SystemMaxUse=/s@.*@SystemMaxUse=4G@g; ' -e '/SystemMaxFileSize=/s@.*@SystemMaxFileSize=256M@g;' -e '/MaxRetentionSec=/s@.*@MaxRetentionSec=2week@g' /etc/systemd/journald.conf
```

综上所述，完成以上这些优化后，一个安全可靠的操作系统就可以正式上线提供服务了, 祝大家 Linux 之旅玩得愉快!