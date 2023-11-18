> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.smzdm.com](https://post.smzdm.com/p/a3x63wrr/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

不止是反向代理！3 分钟在 NAS 上搭建国内大佬开发的免费开源神器『Lucky』
=========================================

2023-11-17 16:47:13 267 点赞 1904 收藏 88 评论

[![](https://qnam.smzdm.com/202311/17/655720064e8941488.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_2/)

所以，这么好的神器不敢独自分享，特地来分享给大家！

关于 Lucky
--------

我原来以为评论区的小伙伴说出 Lucky 的时候，我只是单纯的觉得它可能也只是一个单纯的反代工具，搭建之前我其实并没有对它抱有很大的希望，毕竟这类很多工具都是免费开源的，有些编程基础的其实都可以借鉴源码编译出另外一个自己的版本。

不过等我体验之后，才发现它的功能非常丰富，并且因为是国人开发，操作逻辑和易用性也更符合我们国人的使用习惯。

[![](https://qnam.smzdm.com/202311/17/655720061b9931488.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_3/)

🔺该工具的作者为 **@gdy666**，再此再次对互联网所有的无私奉献者表示感谢！！！

至于该工具的作用，我直接引用作者自己的介绍：

看了介绍之后，我原本以为 NAS 有了 ddns-go 就天下无阻了，没想到 Lucky 比它更猛！

到底怎么样，盘起来再说吧！

Lucky 的安装和部署
------------

**👉安装前的准备**

[![](https://qnam.smzdm.com/202311/17/6557200772a183525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_4/)

🔺打开 NAS 的文件管理器，在 docker 文件夹中（[威联通](https://pinpai.smzdm.com/3063/)

默认为 Container 文件夹），创建一个新文件夹【goodluck】，用于存放它的相关数据。

**👉正式安装和部署**

今天使用的安装方式为 SSH 终端部署，演示的 NAS 为威联通。至于 SSH 工具请自行解决，Putty，XShell，FinalShell 等都可以，我个人使用的是 FinalShell。

[![](https://am.zdmimg.com/202311/17/65572006b22141488.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_5/)

🔺使用 SSH 终端工具连接到 NAS 之后先改用管理员模式登录，输入命令 “ **sudo -i** ” 回车，提示输入密码，密码就是我们 NAS 的登录密码，输入的时候不会有显示，输入完成后直接点回车即可。

[![](https://am.zdmimg.com/202311/17/655720086e0555090.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_6/)[![](https://qnam.smzdm.com/202311/17/655720069ba6b1488.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_7/)

🔺然后还需要在出现上图界面的时候输入 “Q” 和“Y”（[群晖](https://pinpai.smzdm.com/2315/)

[](https://pinpai.smzdm.com/2315/)[![](https://am.zdmimg.com/202311/17/655720078e1453525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_8/)

🔺接着输入 Docker run 命令：

> docker run -d --name lucky --restart always --net=host -p 16601:16601 -v /docker/goodluck:/goodluck gdy666/lucky

以上命令需要改动的为：

*   -p 16601:16601：冒号前面改为本地没被占用的端口；
    
*   -v /docker/goodluck:/goodluck：冒号前面对应我们前面新建 “goodluck” 文件夹的本地实际路径
    

[![](https://qnam.smzdm.com/202311/17/6557200738b243525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_9/)

🔺如果没有问题在 NAS 的 Docker 容器列表中就能看到 lucky 容器已经正在运行了，说明部署成功。

Lucky 的使用体验
-----------

直接在浏览器中输入 【**http:// NAS 的局域网 IP: 端口号**】 就能看到 OneNav 的登录界面了。

**第一步：设置账号和密码**

[![](https://qnam.smzdm.com/202311/17/65572006c13431488.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_10/)

🔺默认的账号和密码均为 “666”，先“登录” 进去。

[![](https://am.zdmimg.com/202311/17/65572007f1bb83525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_11/)

🔺主界面确实做的挺简洁的，还有帮助说明。并且功能也远远比我想象的更要多，并且几乎都是我们在折腾 NAS 的时候需要的，比如除了今天要说的 DDNS 与反向代理，还有端口转发，内网穿透，网络唤醒等多个功能。

[![](https://qnam.smzdm.com/202311/17/6557200706bd63525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_12/)

🔺为了安全起见，我们进入容器的第一步就是在 “设置” 中更改默认的账号和密码。

**第二步：设置 DDNS(动态域名解析)**

[![](https://qnam.smzdm.com/202311/17/655720076de263525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_13/)

🔺在左侧点击 “动态域名”，选择 “添加 DDNS 任务”。

[![](https://am.zdmimg.com/202311/17/655720070b9963525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_14/)

🔺这里默认你已经有了自己的域名，以下信息是你需要填写的：

*   DDNS 任务名称：自己随意填
    
*   DNS 服务商：选择自己域名的服务商，我这里是腾讯云，如果你是阿里云就选择阿里的
    
*   Token：这个就填入自己在域名服务商那里获取到的数据，不知道的可以去下面文章里学习下~
    
*   公网 IP 类型：有 IPv4 的当然选 IPv4，没有的选 IPv6
    
*   域名列表：需要填两行，第一行是主域名，第二行是泛域名，也就是在前面加上 “*.” 即可。
    

其它的直接保持默认即可，最后点 “添加”。

关于域名获取相关 Token 数据，教程在这里：

[![](https://am.zdmimg.com/202311/17/655720075653c3525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_15/)

🔺返回到页面之后，在 “任务列表” 这里看到 “同步结果” 显示为“成功”，就说明我们前面的设置没有问题。

[![](https://am.zdmimg.com/202311/17/65572008342ef5090.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_16/)

🔺这个时候我们就能用自己的这个域名远程访问自己的 NAS 了！不过这里显示的是 “不安全”，其实是我们还没建立更安全的 HTTPS 连接，也就是没有部署 SSL 证书。

其实到这里，我们就已经实现了 DDNS-GO 容器的效果了，并且通过这次体验，有一说一，我个人觉得它的体验确实好过 DDNS-GO。

**第三步：部署 SSL 证书**

[![](https://qnam.smzdm.com/202311/17/655720074cb833525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_17/)

🔺点击左侧的 “安全管理”，选择 “SSL 证书添加”。

[![](https://am.zdmimg.com/202311/17/6557200715e673525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_18/)

🔺以下信息是你需要填写的：

*   备注：随意填一个
    
*   添加方式：选择 “ACME”
    
*   证书颁发机构：选择 “Let's Encrypt”，选择它的目的是因为虽说它的 SSL 证书有效期仅为三个月，但是 Lucky 会一直自动帮我们续签，这个够意思吧！
    
*   DNS 服务商：和前面一样，选择自己域名的服务商
    
*   API 秘钥：这里人家提示的很清楚，是腾讯云 API 秘钥，而不是前面的 Token
    
*   域名列表：和前面一样，需要填两行，第一行是主域名，第二行是泛域名
    
*   勾选：“DNS 查询强制 IPv4” 和 “DNS 查询仅使用 TCP 通道”
    

其它的直接保持默认即可，最后点 “添加”。

[![](https://qnam.smzdm.com/202311/17/65572007b4f233525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_19/)

🔺证书的申请可能需要等一会，一般在一分钟到三分钟之间。我们可以点开 “日志” 看它申请的过程，当出现 “证书申请成功” 和“已保存到配置” 的显示的时候，就说明证书部署成功。

[![](https://qnam.smzdm.com/202311/17/65572007e2da83525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_20/)

🔺部署成功的证书会显示 ACME 信息，以及颁发时间和到期时间。

**第四步：设置安全的反向代理**

[![](https://qnam.smzdm.com/202311/17/655720071f5283525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_21/)

🔺反代就比较简单了，选择左侧的 “Web 服务”，点击 “添加 Web 服务规则”。

[![](https://qnam.smzdm.com/202311/17/655720075ff2e3525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_22/)

🔺以下信息是你需要填写的：

*   Web 服务规则名称：随意填
    
*   监听类型：同理，有 IPv4 的当然选 IPv4，没有的选 IPv6（其实两个都选也没问题）
    
*   监听端口：自己随意改，只要本地端口没被占用即可
    
*   TLS：因为我们前面设置好了 SSL，为了安全的反向代理，这里必须启用
    

其它的直接保持默认即可，接着点 “添加 Web 服务子规则”。

[![](https://qnam.smzdm.com/202311/17/6557200778b683525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_23/)

🔺其实这个子规则就是我们外网访问 NAS 的某项服务时，我们需要隐藏的这个服务。需要填写的信息为：

*   Web 服务子规则名称：随意填
    
*   Web 服务类型：当然是 “反向代理” 了
    
*   前端域名 / 地址：填写从外网访问该服务的二级域名，简单来说就是在我们前面填写的自己域名前面随意添加一个自己能记下来的前缀即可
    
*   后端地址：这个就必须是 NAS 中该服务的局域网页面地址了，一般就是 “内网 IP + 端口号” 组成，直接在页面的地址栏复制粘贴过来即可。注意，前面是有“http://” 的。
    
*   BasicAuth 认证：这个和 NginxWebUI 那个加密功能是一样的，就是我们外网访问的时候需要先输入账号和密码才能打开，相当于在安全上又多了一层屏障，看个人需求吧~
    

如果这个时候你只需要这一个反向代理服务，这个时候你就可以直接点击右下角的 “添加” 按钮即可。如果你有其它服务需要反向代理，则需要直接点击 “添加 Web 服务子规则” 继续。

我这里顺便也演示一遍吧。

[![](https://am.zdmimg.com/202311/17/65572007e674c3525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_24/)

🔺比如说我这里添加了第二个需要反代的服务，其实操作和前面的一模一样，需要更改的只是前端域名的前缀即可，只要和前面的不冲突随意改。

[![](https://am.zdmimg.com/202311/17/65572007eb5853525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_25/)

🔺设置好之后再 “服务列表” 就能看到我们前面设置好的规则。反代之后的好处就是我们以后访问家里 NAS 中的所有服务都不需要记端口号了（因为端口号都是一样的，或者说有那位老 6 测试 80 和 443 端口可以使用，端口号就不用记了，直接使用域名访问），我们只需要记下域名的前缀即可（前缀我们尽量设置简单点，并且和对应的服务有关联性）。

[![](https://qnam.smzdm.com/202311/17/65572007fc9603525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_26/)

🔺最后别忘了在[路由器](https://www.smzdm.com/fenlei/luyouqi/)将反代的端口号做一个端口转发。

[![](https://qnam.smzdm.com/202311/17/65572007640bf3525.png_e1080.jpg)](https://post.smzdm.com/p/a3x63wrr/pic_27/)

🔺看下效果：已经可以 HTTPS 外网访问，并且前面也正常显示安全小锁，此次折腾成功！

通过前面的折腾可以看出，Lucky 这款工具确实不错！可以说是它集中了 NAS 中两大神器：DDNS 神器【DDNS-GO】和反向代理神器【Nginx Proxy Manager】，并且在使用体验上也是有所优化。可能是我折腾太多有了一些经验，我个人完成今天教程所有的操作真的还没用到 10 分钟，真的是快捷又方便！

再次感谢 **@gdy666** 大佬，分享出这么好的工具！

好了，以上就是今天给大家分享的内容，我是爱分享的 Stark-C，如果今天的内容对你有帮助请记得收藏，顺便点点关注，我会经常给大家分享各类有意思的软件和免费干货，咱们下期再见！谢谢大家~

![](https://res.smzdm.com/pc/pc_shequ/dist/img/the-end.png)