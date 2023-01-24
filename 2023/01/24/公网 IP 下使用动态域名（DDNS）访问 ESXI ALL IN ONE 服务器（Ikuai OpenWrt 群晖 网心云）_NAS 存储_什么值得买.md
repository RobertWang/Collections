> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/axlq7nwd/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

公网 IP 下使用动态域名（DDNS）访问 ESXI ALL IN ONE 服务器（Ikuai OpenWrt 群晖 网心云）
===============================================================

2022-02-18 16:59:50 88 点赞 1110 收藏 79 评论

前言
--

搞定公网 IP 后过年期间果断入了 ALL IN ONE 的坑，下面是我目前的配置和成本：

[![](https://qnam.smzdm.com/202202/18/620f185cd51574951.jpg_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_2/)硬件信息

QQLT 是笔记本 U i7-9850HKES 的魔改，45W TDP 6C12T，支持超频![](https://res.smzdm.com/images/emotions/68.png) 追上 10600K（超频后无法正常进入 esxi 系统，百度不到相关信息![](https://res.smzdm.com/images/emotions/78.png) )

这个[机箱](https://www.smzdm.com/fenlei/jixiang/)除了大和便宜以外，一无是处，强烈不推荐
----------------------------------------------------------------

目前 esxi 里装了这些个虚拟机：

[![](https://qnam.smzdm.com/202202/18/620f2244d17896210.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_3/)虚拟机列表

OpenWrt1 单独挂了 WXY1、2，Ikuai+OpenWrt2 作为主力使用，下挂 WXY3、Win10、黑色裙子和无线 AP（[京东云](https://pinpai.smzdm.com/44504/)

[](https://pinpai.smzdm.com/44504/)

无线宝 鲁班），

[群晖](https://pinpai.smzdm.com/2315/)

挂了 webdav（通过 Raidrive 挂载网络盘到 win、ES 文件管理器、Nplayer 视频播放器挂载到

[手机](https://www.smzdm.com/fenlei/zhinengshouji/)

实现云视频和

[音乐](https://www.smzdm.com/fenlei/yinyue/)

等等）、qBittorrent 下载、transmission 保种、jellyfin 影音库

![](https://res.smzdm.com/images/emotions/33.png)

都可以通过端口映射来实现远程访问（远程下载香不香

![](https://res.smzdm.com/images/emotions/34.png)

）。网心云用来赚电费，江苏电信千兆宽带，春节期间每天 4 元左右

![](https://res.smzdm.com/images/emotions/43.png)

一、获取免费域名
--------

首先默认大家都有公网 IP，电信的话只需要和在线客服沟通一下就可以了。  

打开 [http://www.pubyun.com/](http://www.pubyun.com/) 3322 公云，注册一个账号或者到阿里云等买一个域名，登陆后点击创建一个动态域名

[![](https://qnam.smzdm.com/202202/18/620f268fc24706202.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_4/)创建动态域名 1

[![](https://qnam.smzdm.com/202202/18/620f2716ee8a58066.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_5/)创建动态域名 2

选择一个没有被创建的域名，点击创建，这里我创建的是 js14a，加上免费的域名后缀，红框里的就是我创建的二级域名了：js14a.f3322.net

[![](https://qnam.smzdm.com/202202/18/620f2808dfd925727.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_6/)创建动态域名 3

点击小齿轮进入设置界面

[![](https://qnam.smzdm.com/202202/18/620f290dc15512161.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_7/)创建动态域名 4

设置一下更新密码

[![](https://qnam.smzdm.com/202202/18/620f298fc75814875.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_8/)创建动态域名 5

设置好后点击确认，再点修改域名设置  

[![](https://qnam.smzdm.com/202202/18/620f2a1db3e092273.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_9/)创建动态域名 6

到这里动态域名就注册好了，下面到我们的路由上操作，我演示一下 ikuai 和 openwrt 上的操作方法，理解之后用于其他的环境也很简单![](https://res.smzdm.com/images/emotions/26.png)

二、设置动态域名和端口转发
-------------

Ikuai
-----

进入 ikuai 的后台，在高级应用中打开动态域名，添加一条

[![](https://qnam.smzdm.com/202202/18/620f2c29f20d08793.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_10/)ikuai1

这里用户名为 root，填写上你注册的域名和更新解析的密码，其它选择对应的就 OK，保存退出。刷新一下我们可以看到已经发送解析结果，显示更新成功。每隔一段时间就会发送你最新的公网 IP 到域名解析，我们就可以使用这个域名来代替我们的公网 IP 了，只需要记住自己的域名即可，不需要关注公网 IP 的变动了。

[![](https://qnam.smzdm.com/202202/18/620f2cc0aeaa26953.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_11/)ikuai2

OpenWrt
-------

进入后台后在服务中找到动态 dns，输入名称后点击添加

[![](https://qnam.smzdm.com/202202/18/620f302867da42187.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_12/)op1

更改 ddns 服务商为 3322，然后点击更改

[![](https://qnam.smzdm.com/202202/18/620f31517104b9733.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_13/)op2

[![](https://qnam.smzdm.com/202202/18/620f32b7537044200.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_14/)op3

接口选择 wan 口，因为我是多播所以有多个 wan 口

[![](https://qnam.smzdm.com/202202/18/620f32cb10d8e1446.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_15/)op4

这里是设置更新的间隔，10 分钟就够用了，点击右下角的保存并应用，接着点击启动就配置完成了

[![](https://qnam.smzdm.com/202202/18/620f33e2bdd3d7443.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_16/)op5

[![](https://qnam.smzdm.com/202202/18/620f34509cac99506.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_17/)op6

配置端口转发
------

OpenWrt
-------

打开网络找到防火墙，设置转发为接受  

[![](https://qnam.smzdm.com/202202/18/620f2f03eac323879.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_18/)op7

接下来以 esxi 的后台为例，设置端口转发，百度查到 esxi web 界面端口为 443 和 902, 所以内部端口就是 443 和 902  

按图示设置分别设置好 443 和 902 端口

[![](https://qnam.smzdm.com/202202/18/620f35c3c0a6f1635.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_19/)op8

设置好的样子如下，因为 443 端口的特殊性所以外网端口必须设置为非 443 端口，我设置的是外网 29999 转发内网 443，访问外网的 29999 就相当于访问内网的 443 端口![](https://res.smzdm.com/images/emotions/77.png)

[![](https://qnam.smzdm.com/202202/18/620f3648aba651816.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_20/)op9

保存并应用后，在外网打开试试，访问地址为域名: 端口，也就是 https:js14a.f3322.net:29999 ，点开高级选择继续访问，OK 进入熟悉的后台，如果你转发的端口是 http 端口就不需要加这个 https:，直接输入后半部分即可访问，如果不确定就都试一下即可

[![](https://qnam.smzdm.com/202202/18/620f375bd1c8f1557.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_21/)op10

[![](https://qnam.smzdm.com/202202/18/620f37c94dacd7182.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_22/)op11

Ikuai  

--------

打开后台找到端口映射，点击添加，可以看到我已经添加了很多常用应用的端口，大家可以参考一下  

[![](https://qnam.smzdm.com/202202/18/620f39067b652475.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_23/)ikuai3

以群晖为例，内网地址填入群晖的 IP，群晖的 HTTPS 端口为 5001，协议为了方便我就统一选择 TCP+UDP，外网地址选择全部线路，外网端口选择你想用的端口即可，保存后就可以访问了

[![](https://qnam.smzdm.com/202202/18/620f39a1e06223359.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_24/)ikuai4

我这里的访问地址为 https:js14a.f3322.net:29991 , 打开成功![](https://res.smzdm.com/images/emotions/77.png)

[![](https://qnam.smzdm.com/202202/18/620f3ab46713c3609.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_25/)ikuai5

三、外网远程下载
--------

以 Qbit 为例，在群晖 docker 中找到 Qb 的 web 端口为 8989，并且在 web 设置中关闭验证，和之前一样映射出 8989 端口即可远程访问，实现远程下载管理![](https://res.smzdm.com/images/emotions/68.png)

[![](https://qnam.smzdm.com/202202/18/620f3b54388e89064.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_26/)qb1

[![](https://qnam.smzdm.com/202202/18/620f3b9d7b818680.png_e1080.jpg)](https://post.smzdm.com/p/axlq7nwd/pic_27/)qb2

总结一下，不同的应用的外网访问都是相似的，设置的差别都可以百度到，学会举一反三![](https://res.smzdm.com/images/emotions/45.png)

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～