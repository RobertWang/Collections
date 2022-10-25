> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/lyshark_lyshark/article/details/125853245)

**目录**

[二、根据进程名过滤进程信息](#t1)

[三、根据用户名查询该用户的相关信息](#t2)

[五：实现磁盘分区的](#t4)

[六、使用一整块硬盘创建逻辑卷](#t5)

[七、将一块硬盘分区，然后分区设置为虚拟卷](#t6)

### 一、根据 PID 过滤进程所有信息

```
#! /bin/bash
# Author:谢公子
# Date:2018-10-10
# Function: 根据用户输入的PID，过滤出该PID所有的信息
read -p "请输入要查询的PID: " P
n=`ps -aux| awk '$2~/^'$P'$/{print $11}'|wc -l`
if [ $n -eq 0 ];then
 echo "该PID不存在！！"
 exit
fi
echo "--------------------------------"
echo "进程PID: $P"
echo "进程命令：`ps -aux| awk '$2~/^'$P'$/{print $11}'`"
echo "进程所属用户: `ps -aux| awk '$2~/^'$P'$/{print $1}'`"
echo "CPU占用率：`ps -aux| awk '$2~/^'$P'$/{print $3}'`%"
echo "内存占用率：`ps -aux| awk '$2~/^'$P'$/{print $4}'`%"
echo "进程开始运行的时刻：`ps -aux| awk '$2~/^'$P'$/{print $9}'`"
echo "进程运行的时间：`ps -aux| awk '$2~/^'$P'$/{print $10}'`"
echo "进程状态：`ps -aux| awk '$2~/^'$P'$/{print $8}'`"
echo "进程虚拟内存：`ps -aux| awk '$2~/^'$P'$/{print $5}'`"
echo "进程共享内存：`ps -aux| awk '$2~/^'$P'$/{print $6}'`"
echo "--------------------------------"
```

![](https://img-blog.csdn.net/20181013160232671?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 二、根据进程名过滤进程信息

会显示出该进程名包含的所有线程

```
#! /bin/bash
# Author:谢公子
# Date：2018-10-10
# Function: 根据输入的程序的名字过滤出所对应的PID，并显示出详细信息，如果有几个PID，则全部显示
read -p "请输入要查询的进程名：" NAME
N=`ps -aux | grep $NAME | grep -v grep | wc -l`    ##统计进程总数
if [ $N -le 0 ];then
  echo "该进程名没有运行！"
fi
i=1
while [ $N -gt 0 ]
do
  echo "进程PID: `ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $2}'`"
  echo "进程命令：`ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $11}'`"
  echo "进程所属用户: `ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $1}'`"
  echo "CPU占用率：`ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $3}'`%"
  echo "内存占用率：`ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $4}'`%"
  echo "进程开始运行的时刻：`ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $9}'`"
  echo "进程运行的时间：`  ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $11}'`"
  echo "进程状态：`ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $8}'`"
  echo "进程虚拟内存：`ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $5}'`"
  echo "进程共享内存：`ps -aux | grep $NAME | grep -v grep | awk 'NR=='$i'{print $0}'| awk '{print $6}'`"
  echo "***************************************************************"
  let N-- i++
done
```

![](https://img-blog.csdn.net/2018101316052779?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 三、根据用户名查询该用户的相关信息

```
#! /bin/bash
# Author:谢公子
# Date:2018-10-12
# Function：根据用户名查询该用户的所有信息
read -p "请输入要查询的用户名：" A
echo "------------------------------"
n=`cat /etc/passwd | awk -F: '$1~/^'$A'$/{print}' | wc -l`
if [ $n -eq 0 ];then
echo "该用户不存在"
echo "------------------------------"
else
  echo "该用户的用户名：$A"
  echo "该用户的UID：`cat /etc/passwd | awk -F: '$1~/^'$A'$/{print}'|awk -F: '{print $3}'`"
  echo "该用户的组为：`id $A | awk {'print $3'}`"
  echo "该用户的GID为：`cat /etc/passwd | awk -F: '$1~/^'$A'$/{print}'|awk -F: '{print $4}'`"
  echo "该用户的家目录为：`cat /etc/passwd | awk -F: '$1~/^'$A'$/{print}'|awk -F: '{print $6}'`"
  Login=`cat /etc/passwd | awk -F: '$1~/^'$A'$/{print}'|awk -F: '{print $7}'`
  if [ $Login == "/bin/bash" ];then
  echo "该用户有登录系统的权限！！"
  echo "------------------------------"
  elif [ $Login == "/sbin/nologin" ];then
  echo "该用户没有登录系统的权限！！"
  echo "------------------------------"
  fi
fi
```

![](https://img-blog.csdn.net/20181013160747810?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 四、加固系统的一些配置

```
#! /bin/bash
# Author:谢公子
# Date:2018-10-11
# Function:对账户的密码的一些加固
read -p  "设置密码最多可多少天不修改：" A
read -p  "设置密码修改之间最小的天数：" B
read -p  "设置密码最短的长度：" C
read -p  "设置密码失效前多少天通知用户：" D
sed -i '/^PASS_MAX_DAYS/c\PASS_MAX_DAYS   '$A'' /etc/login.defs
sed -i '/^PASS_MIN_DAYS/c\PASS_MIN_DAYS   '$B'' /etc/login.defs
sed -i '/^PASS_MIN_LEN/c\PASS_MIN_LEN     '$C'' /etc/login.defs
sed -i '/^PASS_WARN_AGE/c\PASS_WARN_AGE    '$D'' /etc/login.defs
 
echo "已对密码进行加固，新用户不得和旧密码相同，且新密码必须同时包含数字、小写字母，大写字母！！"
sed -i '/pam_pwquality.so/c\password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=  difok=1 minlen=8 ucredit=-1 lcredit=-1 dcredit=-1' /etc/pam.d/system-auth
 
echo "已对密码进行加固，如果输入错误密码超过3次，则锁定账户！！"
n=`cat /etc/pam.d/sshd | grep "auth required pam_tally2.so "|wc -l`
if [ $n -eq 0 ];then
sed -i '/%PAM-1.0/a\auth required pam_tally2.so deny=3 unlock_time=150 even_deny_root root_unlock_time300' /etc/pam.d/sshd
fi
 
echo  "已设置禁止root用户远程登录！！"
sed -i '/PermitRootLogin/c\PermitRootLogin no'  /etc/ssh/sshd_config
 
read -p "设置历史命令保存条数：" E
read -p "设置账户自动注销时间：" F
sed -i '/^HISTSIZE/c\HISTSIZE='$E'' /etc/profile
sed -i '/^HISTSIZE/a\TMOUT='$F'' /etc/profile
 
echo "已设置只允许wheel组的用户可以使用su命令切换到root用户！"
sed -i '/pam_wheel.so use_uid/c\auth            required        pam_wheel.so use_uid ' /etc/pam.d/su
n=`cat /etc/login.defs | grep SU_WHEEL_ONLY | wc -l`
if [ $n -eq 0 ];then
echo SU_WHEEL_ONLY yes >> /etc/login.defs
fi
 
echo "即将对系统中的账户进行检查...."
echo "系统中有登录权限的用户有："
awk -F: '($7=="/bin/bash"){print $1}' /etc/passwd
echo "********************************************"
echo "系统中UID=0的用户有："
awk -F: '($3=="0"){print $1}' /etc/passwd
echo "********************************************"
N=`awk -F: '($2==""){print $1}' /etc/shadow|wc -l`
echo "系统中空密码用户有：$N"
if [ $N -eq 0 ];then
 echo "恭喜你，系统中无空密码用户！！"
 echo "********************************************"
else
 i=1
 while [ $N -gt 0 ]
 do
    None=`awk -F: '($2==""){print $1}' /etc/shadow|awk 'NR=='$i'{print}'`
    echo "------------------------"
    echo $None
    echo "必须为空用户设置密码！！"
    passwd $None
    let N--
 done
 M=`awk -F: '($2==""){print $1}' /etc/shadow|wc -l`
 if [ $M -eq 0 ];then
  echo "恭喜，系统中已经没有空密码用户了！"
 else
echo "系统中还存在空密码用户：$M"
 fi
fi
 
echo "即将对系统中重要文件进行锁定，锁定后将无法添加删除用户和组"
read -p "警告：此脚本运行后将无法添加删除用户和组！！确定输入Y，取消输入N；Y/N：" i
case $i in
      [Y,y])
            chattr +i /etc/passwd
            chattr +i /etc/shadow
            chattr +i /etc/group
            chattr +i /etc/gshadow
            echo "锁定成功！"
;;
      [N,n])
            chattr -i /etc/passwd
            chattr -i /etc/shadow
            chattr -i /etc/group
            chattr -i /etc/gshadow
            echo "取消锁定成功！！"
;;
       *)
            echo "请输入Y/y or  N/n"
esac
```

![](https://img-blog.csdn.net/2018101316132251?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 五：实现磁盘分区的

只支持分配主分区和标准的 linux 文件系统 (ext4/xfs) 的分区

```
#! /bin/bash
# Author:谢公子
# Date:2018-10-13
# Function:对硬盘进行分区,得到一个标准的linux文件系统(ext4/xfs)的主分区
cat /proc/partitions > old
read -p "请输入你要分区的硬盘(写绝对路径，如：/dev/sda)：" A
if [ -e $A ];then
  echo "true"
else
  echo "该设备不存在！！"
  exit
fi
read -p "请输入你要创建的磁盘分区类型(这里只能是主分区，默认按回车即可):" B
read -p "请输入分区数字，范围1-4，默认从1开始，默认按回车即可：" C
read -p "请输入扇区起始表号，默认按回车即可：" D
read -p "请输入你要分区的分区大小(格式：如 +5G )：" E
fdisk $A << EOF
n
p
$C
$D
$E
w
EOF
echo "一个标准的linux文件系统的分区已经建立好！！"
partprobe $A
echo "-------------------------------"
cat /proc/partitions
cat /proc/partitions > new
F=`diff new old | grep "<" | awk '{print $5}'`
echo "-------------------------------"
echo $F
echo "你想对新分区设定什么类型的文件系统？有以下选项："
echo "A：ext4文件系统"
echo "B：xfs文件系统"
read -p "请输入你的选择：" G
case $G in
        a|A)
           mkfs.ext4 /dev/$F
           echo "该分区将被挂载在 "/mnt/$F" 下" 
           m=`ls /mnt/|grep $F | wc -l`
           if [ $m -eq 0 ];then
            mkdir /mnt/$F
           fi
           n=`cat /etc/fstab | grep /dev/$F| wc -l`
           if [ $n -eq 0 ];then
              echo "/dev/$F     /mnt/$F     ext4         defaults          0      0" >> /etc/fstab
           else
              sed -i '/^\/dev\/$F/c\/dev/$F     /mnt/$F     ext4         defaults          0      0' /etc/fstab
           fi
           mount -a
           df -Th
;;
        b|B)
           mkfs.xfs -f /dev/$F
           echo "该分区将被挂载在 "/mnt/$F" 下" 
           m=`ls /mnt/|grep $F | wc -l`
           if [ $m -eq 0 ];then
              mkdir /mnt/$F
           fi
           n=`cat /etc/fstab | grep /dev/$F | wc -l`
           if [ $n -eq 0 ];then
              echo "/dev/$F     /mnt/$F      xfs       defaults          0      0" >> /etc/fstab
           else
              sed -i '/^\/dev\/$F/c\/dev/$F     /mnt/$F     xfs         defaults          0      0' /etc/fstab
           fi
           mount -a
           df -Th
;;
        *)
           echo "你的输入有误！！"
esac
```

### 六、使用一整块硬盘创建逻辑卷

```
#!/bin/bash
# Author:谢公子
# Date:2018-10-12
# Function:使用一整块硬盘创建LVM逻辑卷
read -p "请输入你要做成逻辑卷的硬盘(写绝对路径，如：/dev/sda)：" path
if [ -e $path ];then
  echo "true"
else
  echo "该设备不存在！！"
  exit
fi
pvcreate $path
echo "该硬盘已做成物理卷！"
vgcreate myvg $path
echo "该物理卷已加入卷组 myvg 中"
vgs
free=`vgs| awk '$1~/myvg/{print}'|awk '{print $6}'`
echo "该物理卷剩余的空间大小为：$free "
read -p "请输入你要创建逻辑卷的大小(如：1G)：" repy2
lvcreate -L $repy2 -n mylv myvg
echo "已成功创建逻辑卷mylv"
echo "------------------------"
lvs
echo "------------------------"
echo "你想对新分区设定什么类型的文件系统？有以下选项："
echo "A：ext4文件系统"
echo "B：xfs文件系统"
read -p "请输入你的选择：" repy3
case $repy3 in
        a|A)
           mkfs.ext4 /dev/myvg/mylv
           echo "该分区将被挂载在 "/mnt/mylv" 下" 
           m=`ls /mnt/|grep mylv | wc -l`
           if [ $m -eq 0 ];then
            mkdir /mnt/mylv
           fi
           echo "/dev/myvg/mylv     /mnt/mylv     ext4         defaults          0      0" >> /etc/fstab
           mount -a
           df -Th
;;
        b|B)
           mkfs.xfs -f /dev/myvg/mylv
           echo "该分区将被挂载在 "/mnt/mylv" 下" 
           m=`ls /mnt/|grep mylv | wc -l`
           if [ $m -eq 0 ];then
              mkdir /mnt/mylv
           fi
           echo "/dev/myvg/mylv     /mnt/mylv      xfs       defaults          0      0" >> /etc/fstab
           mount -a
           df -Th
;;
        *)
           echo "你的输入有误！！"
esac
```

### 七、将一块硬盘分区，然后分区设置为虚拟卷

```
#! /bin/bash
# Author:谢公子
# Date:2018-10-13
# Function:新建一个分区，并做成逻辑卷
cat /proc/partitions > old
read -p "请输入你要分区的硬盘(写绝对路径，如：/dev/sda)：" A
if [ -e $A ];then
  echo "true"
else
  echo "该设备不存在！！"
  exit
fi
read -p "请输入你要创建的磁盘分区类型(这里只能是主分区，默认按回车即可):" B
read -p "请输入分区数字，范围1-4，默认从1开始，默认按回车即可：" C
read -p "请输入扇区起始表号，默认按回车即可：" D
read -p "请输入你要分区的分区大小(格式：如 +5G )：" E
read -p "请输入你要划分为逻辑卷的分区盘符(默认回车即可)：" Z
fdisk $A << EOF
n
p
$C
$D
$E
t
$Z
8e
p
w
EOF
echo "一个标准LVM的分区已经建立好！！"
partprobe $A
echo "-------------------------------"
cat /proc/partitions
cat /proc/partitions > new
F=`diff new old | grep "<" | awk '{print $5}'`
echo "-------------------------------"
echo $F
pvcreate /dev/$F
echo "该硬盘已做成物理卷！"
n=`vgs | grep myvg |wc -l`
if [ $n -eq 0 ];then
   vgcreate myvg /dev/$F
   echo "该物理卷已加入卷组myvg中"
else
   vgextend myvg /dev/$F
   echo  "该物理卷已加入卷组myvg中"
   vgs
   free=`vgs| awk '$1~/myvg/{print}'|awk '{print $7}'`
   echo "该卷组剩余的空间大小为：$free "
   lvs
   exit
fi
vgs
free=`vgs| awk '$1~/myvg/{print}'|awk '{print $6}'`
echo "该卷组剩余的空间大小为：$free "
read -p "请输入你要创建逻辑卷的大小(如：1G)：" repy2
lvcreate -L $repy2 -n mylv myvg
echo "已成功创建逻辑卷mylv"
echo "------------------------"
lvs
echo "------------------------"
echo "你想对新分区设定什么类型的文件系统？有以下选项："
echo "A：ext4文件系统"
echo "B：xfs文件系统"
read -p "请输入你的选择：" G
case $G in
        a|A)
           mkfs.ext4 /dev/myvg/mylv
           echo "该分区将被挂载在 "/mnt/$F" 下" 
           m=`ls /mnt/|grep $F | wc -l`
           if [ $m -eq 0 ];then
            mkdir /mnt/$F
           fi
           echo "/dev/myvg/mylv     /mnt/$F     ext4         defaults          0      0" >> /etc/fstab
           mount -a
           df -Th
;;
        b|B)
           mkfs.xfs -f /dev/myvg/mylv
           echo "该分区将被挂载在 "/mnt/$F" 下" 
           m=`ls /mnt/|grep $F | wc -l`
           if [ $m -eq 0 ];then
              mkdir /mnt/$F
           fi
           echo "/dev/myvg/mylv     /mnt/$F      xfs       defaults          0      0" >> /etc/fstab
           mount -a
           df -Th
;;
        *)
           echo "你的输入有误！！"
esac
```

更多脚本：[https://www.jb51.net/article/54488.htm](https://www.jb51.net/article/54488.htm)

相关文章：[应急响应系统之 Linux 主机安全检查](https://mp.weixin.qq.com/s?__biz=MzI5MDQ2NjExOQ==&mid=2247489990&idx=1&sn=db37670d24a97e33b58d30e8f653a909&scene=21#wechat_redirect)