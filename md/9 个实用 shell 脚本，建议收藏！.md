> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247505876&idx=1&sn=735f0536586273623e83485c345e049f&chksm=e885b736dff23e20528d011a6bba9e9d1cb05d07b8b43ead0c38634aede01940d0a18efb70b1&mpshare=1&scene=1&srcid=0317dZ5cdWOLtgn4IggQgAz0&sharer_sharetime=1647504423666&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

```
今天跟大家分享一下 9 个实用的 shell 脚本。
```

1.Dos 攻击防范 ，收藏起来（自动屏蔽攻击 IP）

```
#!/bin/bash
DATE=$(date +%d/%b/%Y:%H:%M)
LOG_FILE=/usr/local/nginx/logs/demo2.access.log
ABNORMAL_IP=$(tail -n5000 $LOG_FILE |grep $DATE |awk '{a[$1]++}END{for(i in a)if(a[i]>10)print i}')
for IP in $ABNORMAL_IP; do
    if [ $(iptables -vnL |grep -c "$IP") -eq 0 ]; then
        iptables -I INPUT -s $IP -j DROP
        echo "$(date +'%F_%T') $IP" >> /tmp/drop_ip.log
    fi
done
```

  

2.Linux 系统发送告警脚本

```
# yum install mailx
# vi /etc/mail.rc
set from=baojingtongzhi@163.com smtp=smtp.163.com
set smtp-auth-user=baojingtongzhi@163.com smtp-auth-password=123456
set smtp-auth=login
```

  

3.MySQL 数据库备份单循环

```
#!/bin/bash
DATE=$(date +%F_%H-%M-%S)
HOST=localhost
USER=backup
PASS=123.com
BACKUP_DIR=/data/db_backup
DB_LIST=$(mysql -h$HOST -u$USER -p$PASS -s -e "show databases;" 2>/dev/null |egrep -v "Database|information_schema|mysql|performance_schema|sys")

for DB in $DB_LIST; do
    BACKUP_NAME=$BACKUP_DIR/${DB}_${DATE}.sql
    if ! mysqldump -h$HOST -u$USER -p$PASS -B $DB > $BACKUP_NAME 2>/dev/null; then
        echo "$BACKUP_NAME 备份失败!"
    fi
done
```

  

4.MySQL 数据库备份多循环

```
#!/bin/bash
DATE=$(date +%F_%H-%M-%S)
HOST=localhost
USER=backup
PASS=123.com
BACKUP_DIR=/data/db_backup
DB_LIST=$(mysql -h$HOST -u$USER -p$PASS -s -e "show databases;" 2>/dev/null |egrep -v "Database|information_schema|mysql|performance_schema|sys")

for DB in $DB_LIST; do
    BACKUP_DB_DIR=$BACKUP_DIR/${DB}_${DATE}
    [ ! -d $BACKUP_DB_DIR ] && mkdir -p $BACKUP_DB_DIR &>/dev/null
    TABLE_LIST=$(mysql -h$HOST -u$USER -p$PASS -s -e "use $DB;show tables;" 2>/dev/null)
    for TABLE in $TABLE_LIST; do
        BACKUP_NAME=$BACKUP_DB_DIR/${TABLE}.sql
        if ! mysqldump -h$HOST -u$USER -p$PASS $DB $TABLE > $BACKUP_NAME 2>/dev/null; then
            echo "$BACKUP_NAME 备份失败!"
        fi
    done
done
```

  

5.Nginx 访问访问日志按天切割

```
#!/bin/bash
LOG_DIR=/usr/local/nginx/logs
YESTERDAY_TIME=$(date -d "yesterday" +%F)
LOG_MONTH_DIR=$LOG_DIR/$(date +"%Y-%m")
LOG_FILE_LIST="default.access.log"

for LOG_FILE in $LOG_FILE_LIST; do
    [ ! -d $LOG_MONTH_DIR ] && mkdir -p $LOG_MONTH_DIR
    mv $LOG_DIR/$LOG_FILE $LOG_MONTH_DIR/${LOG_FILE}_${YESTERDAY_TIME}
done

kill -USR1 $(cat /var/run/nginx.pid)
```

  

6.Nginx 访问日志分析脚本

```
#!/bin/bash
# 日志格式: $remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"
LOG_FILE=$1
echo "统计访问最多的10个IP"
awk '{a[$1]++}END{print "UV:",length(a);for(v in a)print v,a[v]}' $LOG_FILE |sort -k2 -nr |head -10
echo "----------------------"

echo "统计时间段访问最多的IP"
awk '$4>="[01/Dec/2018:13:20:25" && $4<="[27/Nov/2018:16:20:49"{a[$1]++}END{for(v in a)print v,a[v]}' $LOG_FILE |sort -k2 -nr|head -10
echo "----------------------"

echo "统计访问最多的10个页面"
awk '{a[$7]++}END{print "PV:",length(a);for(v in a){if(a[v]>10)print v,a[v]}}' $LOG_FILE |sort -k2 -nr
echo "----------------------"

echo "统计访问页面状态码数量"
awk '{a[$7" "$9]++}END{for(v in a){if(a[v]>5)print v,a[v]}}'
```

  

7. 查看网卡实时流量脚本

```
#!/bin/bash
NIC=$1
echo -e " In ------ Out"
while true; do
    OLD_IN=$(awk '$0~"'$NIC'"{print $2}' /proc/net/dev)
    OLD_OUT=$(awk '$0~"'$NIC'"{print $10}' /proc/net/dev)
    sleep 1
    NEW_IN=$(awk  '$0~"'$NIC'"{print $2}' /proc/net/dev)
    NEW_OUT=$(awk '$0~"'$NIC'"{print $10}' /proc/net/dev)
    IN=$(printf "%.1f%s" "$((($NEW_IN-$OLD_IN)/1024))" "KB/s")
    OUT=$(printf "%.1f%s" "$((($NEW_OUT-$OLD_OUT)/1024))" "KB/s")
    echo "$IN $OUT"
    sleep 1
done
```

  

8. 服务器系统配置初始化脚本

```
#/bin/bash
# 设置时区并同步时间
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
if ! crontab -l |grep ntpdate &>/dev/null ; then
    (echo "* 1 * * * ntpdate time.windows.com >/dev/null 2>&1";crontab -l) |crontab
fi

# 禁用selinux
sed -i '/SELINUX/{s/permissive/disabled/}' /etc/selinux/config

# 关闭防火墙
if egrep "7.[0-9]" /etc/redhat-release &>/dev/null; then
    systemctl stop firewalld
    systemctl disable firewalld
elif egrep "6.[0-9]" /etc/redhat-release &>/dev/null; then
    service iptables stop
    chkconfig iptables off
fi

# 历史命令显示操作时间
if ! grep HISTTIMEFORMAT /etc/bashrc; then
    echo 'export HISTTIMEFORMAT="%F %T `whoami` "' >> /etc/bashrc
fi

# SSH超时时间
if ! grep "TMOUT=600" /etc/profile &>/dev/null; then
    echo "export TMOUT=600" >> /etc/profile
fi

# 禁止root远程登录
sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# 禁止定时任务向发送邮件
sed -i 's/^MAILTO=root/MAILTO=""/' /etc/crontab

# 设置最大打开文件数
if ! grep "* soft nofile 65535" /etc/security/limits.conf &>/dev/null; then
    cat >> /etc/security/limits.conf << EOF
    * soft nofile 65535
    * hard nofile 65535
EOF
fi

# 系统内核优化
cat >> /etc/sysctl.conf << EOF
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_tw_buckets = 20480
net.ipv4.tcp_max_syn_backlog = 20480
net.core.netdev_max_backlog = 262144
net.ipv4.tcp_fin_timeout = 20
EOF

# 减少SWAP使用
echo "0" > /proc/sys/vm/swappiness

# 安装系统性能分析工具及其他
yum install gcc make autoconf vim sysstat net-tools iostat if
```

  

9. 监控 100 台服务器磁盘利用率脚本

```
#!/bin/bash
HOST_INFO=host.info
for IP in $(awk '/^[^#]/{print $1}' $HOST_INFO); do
    USER=$(awk -v ip=$IP 'ip==$1{print $2}' $HOST_INFO)
    PORT=$(awk -v ip=$IP 'ip==$1{print $3}' $HOST_INFO)
    TMP_FILE=/tmp/disk.tmp
    ssh -p $PORT $USER@$IP 'df -h' > $TMP_FILE
    USE_RATE_LIST=$(awk 'BEGIN{OFS="="}/^\/dev/{print $NF,int($5)}' $TMP_FILE)
    for USE_RATE in $USE_RATE_LIST; do
        PART_NAME=${USE_RATE%=*}
        USE_RATE=${USE_RATE#*=}
        if [ $USE_RATE -ge 80 ]; then
            echo "Warning: $PART_NAME Partition usage $USE_RATE%!"
        fi
    done
done
```

  

转载：我的小碗汤