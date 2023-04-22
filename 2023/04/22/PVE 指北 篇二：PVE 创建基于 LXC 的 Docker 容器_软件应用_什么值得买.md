> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/a4pw9qrk/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

PVE 指北 篇二：PVE 创建基于 LXC 的 Docker 容器
==================================

2022-11-06 14:27:58 16 点赞 120 收藏 7 评论

[![](https://qnam.smzdm.com/202211/05/636646843a9308483.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_2/)

点击 PVE，选择 Shlle  

[![](https://am.zdmimg.com/202211/05/636646842d1ac2863.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_3/)

在 Shell 中执行以下命令，用来给 CT 模板换源

cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm_back

sed -i 's|http://download.proxmox.com|https://mirrors.tuna.tsinghua.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm

重启服务

systemctl restart pvedaemon.service

[![](https://qnam.smzdm.com/202211/05/6366468430cbd5142.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_4/)

这时候会报一个错误，TA[SK](https://pinpai.smzdm.com/20247/)

[](https://pinpai.smzdm.com/20247/)

ERROR: command '/usr/bin/termproxy 5900, 我也不知道为啥会报错，我的解决方法, 使用 SSH 软件连接 PVE，重新执行换源操作。

输入 ip，账号，密码，连接 SSH

[![](https://qnam.smzdm.com/202211/05/63664684aa3505362.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_5/)

重新执行，以上两条命令即可。

PVE_WEB 中, 点击 local-CT 模板 - 模板

[![](https://am.zdmimg.com/202211/05/63664684b62209901.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_7/)

选择 debian-11, 单击下载

[![](https://am.zdmimg.com/202211/05/636646854740b8180.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_8/)

等待下载完成，出现 TASK OK 就是下载完成了。

[![](https://qnam.smzdm.com/202211/05/636646853bcde7923.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_9/)

点击创建 CT，输入主机名，密码，取消无特权的容器勾选

[![](https://qnam.smzdm.com/202211/05/63664685716133116.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_10/)

选择 debian-11 做为模板

[![](https://qnam.smzdm.com/202211/05/63664685a9c074177.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_11/)

根据实际情况给[磁盘](https://www.smzdm.com/ju/s2441lm/)大小，建议不低于 50G

[![](https://qnam.smzdm.com/202211/05/636646861ac8a7360.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_12/)

根据实际情况给 [CPU](https://www.smzdm.com/fenlei/cpu/) 核心数量

[![](https://am.zdmimg.com/202211/05/636646861c7896683.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_13/)

根据实际情况给[内存](https://www.smzdm.com/fenlei/neicun/)数量

[![](https://qnam.smzdm.com/202211/05/636646863597b4837.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_14/)

根据实际情况，分配静态 ip 或者使用 DHCP，我这边关闭了 iPV6

[![](https://qnam.smzdm.com/202211/05/63664686855e84276.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_15/)

DNS 设置主路由 ip，或者使用主机设置都行

[![](https://am.zdmimg.com/202211/05/636646868707e7070.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_16/)

点击完成, 进入 PVE 的 shell 中，关闭 LXC 容器的 apparmor 保护，开启状态下会造成 docker 无法安装！

[![](https://am.zdmimg.com/202211/05/63664686bf9b52125.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_17/)

输入以下命令，模板 ID，替换为自己的 id，图中为 100。

nano /etc/pve/lxc/[模板 ID].conf

[![](https://am.zdmimg.com/202211/05/63664686eb22d131.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_18/)

粘贴以下内容，Shell 中粘贴的快捷键为 SHIFT+CTRL+V, 或者[鼠标](https://www.smzdm.com/fenlei/shubiao/)右键粘贴

lxc.apparmor.profile: unconfined

[![](https://qnam.smzdm.com/202211/05/636646874e94e561.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_19/)

粘贴完成后，使用 CTRL+X, 输入 Y 后按回车退出。  

启动 Docker

[![](https://qnam.smzdm.com/202211/05/636646876e6bc6618.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_20/)

更换 Debian 源

mv /etc/apt/sources.list /etc/apt/sources.list.bk

nano /etc/apt/sources.list

[![](https://qnam.smzdm.com/202211/05/63664687b8a6b3651.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_21/)

粘贴

deb https://mirrors.ustc.edu.cn/debian/ bullseye main non-free contrib

deb-src https://mirrors.ustc.edu.cn/debian/ bullseye main non-free contrib

deb https://mirrors.ustc.edu.cn/debian-security/ bullseye-security main

deb-src https://mirrors.ustc.edu.cn/debian-security/ bullseye-security main

deb https://mirrors.ustc.edu.cn/debian/ bullseye-updates main non-free contrib

deb-src https://mirrors.ustc.edu.cn/debian/ bullseye-updates main non-free contrib

deb https://mirrors.ustc.edu.cn/debian/ bullseye-backports main non-free contrib

deb-src https://mirrors.ustc.edu.cn/debian/ bullseye-backports main non-free contrib

[![](https://qnam.smzdm.com/202211/05/63664687ecfce9620.png_e1080.jpg)](https://post.smzdm.com/p/a4pw9qrk/pic_22/)

粘贴完成后，使用 CTRL+X, 输入 Y 后按回车退出。

更新

apt update

apt upgrade -y

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～