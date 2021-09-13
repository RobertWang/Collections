> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.m.smzdm.com](https://post.m.smzdm.com/p/a5d0n308/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=other)

> &nbsp; &nbsp; &nbsp; &nbsp; 市面上流行的密码方案比较多，LastPass、1passwor

市面上流行的密码方案比较多，LastPass、1password、Enpass、Keeper、Bitwarden 都各有有点，但在开源免费方面 Bitwarden 应该是无人能及吧；Bitwarden 对我这种白嫖佬来说全身都是有点，免费、速度快、体积小、数据自己保管、支持 TOTP；同时 Bitwarden 还具有多种客户端，支持浏览器插件、手机 app 等，几乎能覆盖所有的应用环境；废话不多说，现在咱们就开始使用群晖的 docker 部署一款属于自己的密码库吧（注以下内容需要环境包括：公网 IP、你会路由器端口映射、你有域名、你的域名有 ssl；如果没有环境的话可以分项百度自行配置）。

### bitwarden 的搭建主要分为 1. docker 部署 2. 设置二级域名 3. 路由器端口映射 4. 群晖反向代理 5. bitwarden 设置

一、Docker 部署 bitwarden
---------------------

1. 首先打开群晖，打开 file station，选中 docker 文件夹，新建文件夹取名为 bitwarden ；（名字其实无所谓爱用啥用啥）

![](https://qnam.smzdm.com/202108/06/610d078a0095a1810.png_e600.jpg)

2. 首先打开群晖 docker，选中注册表，搜索 vaultwarden，找到 vaultwarden/server

为什么要使用 vaultwarden/server 呢，因为 bitwardenrs/server 改名了~~~~~

![](https://am.zdmimg.com/202108/06/610d07b9c5b633885.png_e600.jpg)

3. 影响下载好以后选中映像，在映像中找到 vaultwarden/server ，双击进入配置，修改容器名称（随意修改，我一般用默认值），然后进入高级设置；

![](https://am.zdmimg.com/202108/06/610d088c1bfca5259.png_e600.jpg)

4. 高级设置中，首页有自动启动，开不开看你自己需求；然后选中 存储空间 ，添加文件夹，宿主机路径为刚刚创建的 docker/bitwarden ， 容器路径一定要挂载 /data ；然后进入端口设置；

![](https://qnam.smzdm.com/202108/06/610d0d372821e2394.png_e600.jpg)

5. 进入端口设置，首先删除第一行设置，剩下一行设置如下，本地端口使用任意未被暂用的端口，

我这里作为演示使用 16666 ， 容器端口不变 保持 80 ；设置好后进入 环境 ；  

![](https://qnam.smzdm.com/202108/06/610d0e2825b373750.png_e600.jpg)

6. 环境配置，环境配置可以配置注册是否开放，时区，邮件验证功能；

![](https://res.smzdm.com/images/emotions/162.gif) 没有特殊需求设置 SIGNUPS_ALLOWED 、 TZ 即可，设置如下：

![](https://qnam.smzdm.com/202108/06/610d20c6919b04816.png_e600.jpg)

# SIGNUPS_ALLOWED 控制注册是否开放，true 开放注册，false 关闭注册

可变：SIGNUPS_ALLOWED 值：true

# TZ 控制时区，默认时区是 utc，TZ 设置为 Asia/Shanghai 可以修改时区上海

可变：TZ 值：Asia/Shanghai

docker 配置到这里基本上就能用了，下列几项配置，是因为我这边因为时区问题，手机 app 没法用谷歌验证器配置的 totp 登录，会提示我时区错误，所以我单独设置了邮件两步验证；

![](https://res.smzdm.com/images/emotions/170.gif) 如果你谷歌验证器不会报时区错误的话，可以直接使用谷歌验证器，设置更简单，如果你使用 Authy 或 Google Authenticator 的两步验证器，也提示存在时区错误的话，就可以配置邮件验证：

邮件验证需要配置 SMTP_PASSWORD、SMTP_USERNAME、SMTP_SSL、SMTP_PORT、SMTP_FROM、SMTP_HOST ；配置如下：

![](https://am.zdmimg.com/202108/08/610f8b39dfd317711.png_e600.jpg)

# SMTP_PASSWORD 我用的 QQ 邮箱，QQ 邮箱有专用的 smtp 密码；

# SMTP_USERNAME 使用自己的 QQ 邮箱就可以了；

# SMTP_FROM 同上，使用自己的 QQ 邮箱就可以了；

# SMTP_SSL 使用 SSL 连接，值为 true；

# SMTP_PORT smtp 端口号，QQ 邮箱 smtp 端口号是 587；

# SMTP_HOST smtp 主机，QQ 邮箱 smtp 主机位 smtp.qq.com ；

7. 全部设置好以后，应用→下一步→应用

配置完毕以后把 bitwarden 容器运行起来，这个时候使用 [群晖 ip:16666] 即可访问到 bitwarden 的登录页面了，但是应该是无法登录的，因此我们要开始这是反向代理。

二、二级域名设置
--------

不同的 dns 服务商大同小异，我这里以 cloudflare 设置为例：

进入 dns 服务商，将顶级域名（比如：abc.com）的 A 记录指向群晖的公网 IP，然后增加 CNAME 记录，设置二级域名为 vault ，内容为顶级域名 abc.com；

![](https://am.zdmimg.com/202108/06/610d23274fe0f9998.png_e600.jpg)

上述步骤设置好以后，我们进入路由器设置端口映射，这里正好说一个思路，群晖里面有很多套件跟 docker 都有自己的 webui，他们都有不同的端口，我们可以在路由器里面映射一个特定的端口到群晖，然后群晖在根据二级域名设置反向代理，即可做到使用不同的二级域名控制不用的应用；  

比如群晖路由器分配的 ip 地址为 192.168.1.100 ，我们用路由器映射一个端口 88 给群晖， 然后群晖设置来源为 vault.abc.com:88 ， 目的地为 192.168.1.100:16666 的反向代理，设置好了以后，我们就可以通过 vault.abc.com:88 访问 vault 的 webui 了，而且这样省去了为每项应用配置 ssl 的过程，只需要在群晖控制面板→安全性→证书 里面添加证书，反向代理的其他 webui 全都能用了；

三、路由器端口映射
---------

打开路由器，端口映射设置不限制来源，端口范围为 88 ， 内网 IP 为 192.168.1.100 ， 本地端口为 88 ；

设置完后保存应用，进入下一步；

![](https://qnam.smzdm.com/202108/06/610d25c7f26352188.png_e600.jpg)

四、群晖反向代理设置
----------

打开群晖控制面板，选中 应用程序门户 ， 选中 反向代理服务器 ，按照下图以此设置：

![](https://qnam.smzdm.com/202108/06/610d2713ec83d783.png_e600.jpg)

上面设置中 启用 hsts 启用 http/2 选不选择都可以，看你心情；目的地协议 本地如果没有配置 ssl 的话 就可以直接用 http 即可；主机名 如果跟群晖同 ip 可以直接使用 localhost ，如果是其他 ip 直接输入需要反代的 ip 即可；这里延伸出，群晖的这个功能 ，除了能给群晖的应用进行反向代理，还能给其他服务端下的应用进行反向代理，比如我路由器 webui ip 为 192.168.1.1，我开启了共享，端口为 80，就可以用群晖把特定的二级域名（比如 router.abc.com:88），指向路由器 ip，端口号 80，这样我们就可以在外网用 router.abc.com:88 访问家里的路由器了。

群晖反向代理服务器，其实就是基于 nginx 的一个 ui，如果有用其他 nas 的，可以直接配置 nginx，也可以达到同样的效果；

群晖反向代理设置完成以后，就可以用 vault.abc.com:88 打开 bitwarden 的主页了；

五、bitwarden 设置
--------------

1. 创建账号提交，创建后主要记住主密码，最好是用物理的方式保存好，比如写在笔记本上；

![](https://qnam.smzdm.com/202108/06/610d2ac817cb71771.png_e600.jpg)

2. 注册完成以后登录进去，点击 设置 ， 两步登录 （不设置也无所谓），这里推荐第一种或第四种，第一种是可以通过下载谷歌验证器或者微软的 Authenticator，第四种直接使用邮箱来接收验证码。  

![](https://qnam.smzdm.com/202108/06/610d2c53432678335.png_e600.jpg)

3. 浏览器应用下载， 实测使用 chrome 或者 edge、360 浏览器、360 急速浏览器可以直接在应用商店找到 bitwarden 应用（其他浏览器应该也有，但是我没用过，你需要自己找。），

[edge 扩展中心](https://go.smzdm.com/a8bffc191c635d22/ca_aa_ot_163_84377424_11335_2315_173_0)

[360 急速扩展中心](https://go.smzdm.com/802049787303d8f2/ca_aa_ot_163_84377424_11335_2315_173_0)

进入扩展中心，搜索 bitwarden，找到以后安装插件，插件安装好以后会在浏览器上显示，然后点击插件进行配置；配置只需要在设置→服务器 URL 里面填入反代以后的 url 点击保存即可；  

![](https://qnam.smzdm.com/202108/06/610d2f2d6ac0c2976.png_e600.jpg)

保存登录以后就即可正常使用拉；另外 bitwarden 抢眼功能，支持 totp，在新建网页信息的时候，取得 totp 字符串，填入 totp 里面就可以使用 totp 功能了，赞~~~~  

![](https://qnam.smzdm.com/202108/06/610d3031b76b03197.png_e600.jpg)

4. 手机 app 设置，先到手机下载 bitwarden 客户端，完成后打开 app，你会发现跟前面图片一样，照着设置→保存→登录 即可使用。

### bitwarden 配置就到这里拉，里面的各项功能各位小伙伴可以根据需要自己设置，如果有设置不明白的话可以给我留言哟~~~