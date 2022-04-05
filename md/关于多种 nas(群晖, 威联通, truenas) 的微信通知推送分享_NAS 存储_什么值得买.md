> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [post.m.smzdm.com](https://post.m.smzdm.com/p/amxkz8g4/?send_by=3547620770&invite_code=zdm4ptnapkinv&zdm_ss=iOS_3547620770_&from=singlemessage)

> 目前的 nas 都自带了邮件通知及短信通知等方式，但我们可能更习惯的是微信接收通知，我这边因为几个 nas 都在用，对多种 nas 的微信通知推送进行了一些测试

目前的 nas 都自带了邮件通知及短信通知等方式，但我们可能更习惯的是微信接收通知，我这边因为几个 nas 都在用，对多种 nas 的微信通知推送进行了一些测试，现在分享一下自己这几天折腾的体会![](https://res.smzdm.com/images/emotions/22.png) 。

目前的微信通知用得最多的可能是 server 酱，不过有些问题，目前 server 酱升级后对免费推送消息进行了限制，并且还需要关注他的公众号才行，这就用着有点不舒服，![](https://res.smzdm.com/images/emotions/46.png) 虽然用别人的服务收费是天经地义的![](https://res.smzdm.com/images/emotions/45.png) 。

再经过一番搜索之后，发现可以通过企业微信 api 来曲线救国实现消息推送![](https://res.smzdm.com/images/emotions/35.png)

使用企业微信推送的流程大概就是

本地搭建 web 服务并开放接口 --> nas 使用自定义短信服务商指向本地 web 服务 -->web 服务转发请求到企业微信

--> 企业微信推送 --> 微信接收

大概找了下，目前用得比较多的是使用 nodered 来实现，图形化编程，看上去实现起来还算简单![](https://res.smzdm.com/images/emotions/33.png)

关于企业微信及应用的申请和 nodered 的搭建我就不再赘述，大家可以看站里其他文章

例如：[https://post.m.smzdm.com/p/a9g4r4me/](https://post.m.smzdm.com/p/a9g4r4me/)

不过可以注意到的是目前看到的文章使用了企业微信的相关应用的 API 搭配 nodered 插件接收消息，这个功能本意是在对外提供服务时更有安全保障，缺点是需要一个域名，相应的还需要公网 ip，![](https://res.smzdm.com/images/emotions/58.png) 对于没有公网的同学就不太友好![](https://res.smzdm.com/images/emotions/48.png)

不过我们作为拥有者其实不必这么麻烦，使用 corpId 和 corpSecret 就可实现自己发送消息，就是需要处理下 token 问题，所以我对该方式稍微做了一些改动，在没有公网 ip 的情况下也能使用

提前说明下我对 nodered 及 js 不太熟悉，下面的方式是我自己摸索了一天测试出来的

吐槽下资料真不好找![](https://res.smzdm.com/images/emotions/48.png) ，估计 js 开发的会很熟悉

我自己测试使用通过，不过其中可能有 bug 或待优化的地方，各位有更好地欢迎分享

使用方法如下

我把 nodered 配置导出了 json

百度网盘 [https://pan.baidu.com/s/1G7HLXEs6mR4f6kPwk_XQtA?pwd=q29u](https://pan.baidu.com/s/1G7HLXEs6mR4f6kPwk_XQtA?pwd=q29u)

密码 q29u

复制上述 json，

修改获取 token 节点中的 corpid 和 corpsecret 为你企业微信的相应值

![](https://qnam.smzdm.com/202201/11/61dd533cd0d9f6410.png_e600.jpg)

  
修改构造发送数据节点中的你的 agentid 为对应的 agentid，注意 agentid 为数字

![](https://qnam.smzdm.com/202201/11/61dd535615c842254.png_e600.jpg)

在 nodered 中选择导入

![](https://qnam.smzdm.com/202201/11/61dd53083acb16263.png_e600.jpg)导入 json

然后粘贴导入即可![](https://res.smzdm.com/images/emotions/34.png)

![](https://qnam.smzdm.com/202201/11/61dd536b60a173321.png_e600.jpg)

导入成功后如下，直接部署即可

![](https://qnam.smzdm.com/202201/11/61dd5376491a64368.png_e600.jpg)

能力原因，这个流程有个小问题![](https://res.smzdm.com/images/emotions/89.png)

每次发送消息都会先获取 token，然后再发送，也就是两次 http 请求，所以耗时会长一些，我测试大概在 500ms 左右，同时我也没有做异常处理，有需要的自行修改。。（实在不想改了。。。。 ![](https://res.smzdm.com/images/emotions/91.png) ）

还有个潜在问题是微信获取 token 的频率是有限制的，不过我看了下估计我们单人使用正常情况下基本不可能超出限制

说下我这边的服务 url 配置，有两个 http 入口

get： [http://ip:1880/push?from=a&to=b&text=test2](http://ip:1880/push?from=a&to=b&text=test2)

其中 ip 为你安装 nodered 的 ip，端口 1880 为 nodered 的端口，[如果](https://pinpai.m.smzdm.com/115291/)你不一样需要修改

from 为发送方，在群晖和[威联通](https://pinpai.m.smzdm.com/3063/)里对应手机号，truenas 中可自定义，

理论上在多台设备下可以分辨哪台设备的通知

to 为接收者在企业微信中的 id，向关注该应用所有人发送的话使用 @all（一般用这个就行）

text 为数据内容，由 nas 填充

post： [http://ip:1880/push?from = 测试 & to=b](http://ip:1880/push?from=%E6%B5%8B%E8%AF%95&to=b)

post 主要针对 truenas，url 意义同上，数据体在 body 中，由 nas 填充，不需要[关心](https://pinpai.m.smzdm.com/291412/)

关于 nodered 方面的配置就到此为止了![](https://res.smzdm.com/images/emotions/60.png)

下面说下 nas 方面的配置

群晖方面，到群晖控制面板 -- 短信 -- 新增短信服务提供商，如图

![](https://qnam.smzdm.com/202201/11/61dd53d35886a1094.png_e600.jpg)

注意 text 后面需要写入 hello world，群晖要求的

然后下一步，

请求标题可以不填，下一步

各参数类型如下，这儿实际上就是群晖会把对应的数据填充到参数里

其中电话和内容是必须的

![](https://qnam.smzdm.com/202201/11/61dd53ec35e9b6247.png_e600.jpg)

然后保存, 填入电话号码（可以乱填），点击寄送测试短信，正常的话就能收到微信推送了

我这里使用的是普通消息，卡片消息需要修改发送消息模块里的 json 模板

如图

![](https://qnam.smzdm.com/202201/11/61dd5418f05b76282.png_e600.jpg)

然后是威联通，我们需要到通知中心 -- 服务账户和设备配对 -- 短信，如图

![](https://qnam.smzdm.com/202201/11/61dd53e4ea8161238.png_e600.jpg)

点击添加 SMSC 服务，然后在 SMS 提供商处选择 custom 自定义

![](https://qnam.smzdm.com/202201/11/61dd54312d2155360.png_e600.jpg)

由于[服务器](https://m.smzdm.com/fenlei/fuwuqi/)为我们自定义内网服务，所以不需要用户名和密码，只需要关注 url 模板这个地方

qnap 和群晖不一样，填写的 url 如下

[http://127.0.0.1:1880/push?from=@@PhoneNumber@@&text=@@Text@@&to=@all](http://127.0.0.1:1880/push?from=@@PhoneNumber@@&text=@@Text@@&to=@all)

其中的 @@PhoneNumber@@ 和 @@Text@@ 是 qnap 自己的占位标志

在发送时系统会自动替换为手机号 和 具体内容点击提供商旁边的小飞机按钮发送测试短信，随便输入电话号码，点击发送就能收到了如图

![](https://qnam.smzdm.com/202201/11/61dd5444ef2b58608.png_e600.jpg)

收到如图

![](https://qnam.smzdm.com/202201/11/61dd546d1c44d1005.png_e600.jpg)

由于我自己在使用 truenas scale ，所以也想着将 turenas scale 的通知也推送到微信上![](https://res.smzdm.com/images/emotions/43.png)

方便进行统一查看，可惜找了一下，truenas 有很多通知服务，不过大多是国外得到服务提供商![](https://res.smzdm.com/images/emotions/36.png)

并且 truenas 还没有提供自定义短信供应商的服务![](https://res.smzdm.com/images/emotions/53.png)

![](https://qnam.smzdm.com/202201/11/61dd547c8bc0b3796.png_e600.jpg)

不过在我一个一个尝试这些服务的过程中，发现在 Slack 的服务那用的是 webhook 的形式发送的数据![](https://res.smzdm.com/images/emotions/51.png)

理论上我们应该可以通过 Slack 的 webhook 指向我们自己的服务再转发到微信了![](https://res.smzdm.com/images/emotions/39.png)

稍微测试了下，找到了发送数据的真正格式，类似于

{"text":"数据内容"}

请求方式是 POST，这也就是为什么我在 nodered 中有两个 http 输入了![](https://res.smzdm.com/images/emotions/43.png)

具体使用方式如下 truenas 主页面 -- 通知服务

![](https://qnam.smzdm.com/202201/11/61dd5497663ff6448.png_e600.jpg)

通知设置 -- 通知服务

![](https://qnam.smzdm.com/202201/11/61dd54a20094f6666.png_e600.jpg)

默认有 email 和 snmp，我们添加一个服务

![](https://qnam.smzdm.com/202201/11/61dd54ade71972217.png_e600.jpg)

填写如下

![](https://qnam.smzdm.com/202201/11/61dd54c4675222657.png_e600.jpg)

然后点击 发送测试通知，配置正确的情况下就能收到通知了 truenas 发出的通知如图![](https://res.smzdm.com/images/emotions/118.png)

![](https://qnam.smzdm.com/202201/11/61dd54cfcd4145906.png_e600.jpg)

总共花了大概一天的时间，基本达到这三种 nas 都能实现微信推送的目的，![](https://res.smzdm.com/images/emotions/106.png)

由于对 nodered 的不了解导致有些小瑕疵，不过基本算是能用了

**彩蛋**

最近上班疯狂摸鱼，已经没心思工作了![](https://res.smzdm.com/images/emotions/80.png)

![](https://qnam.smzdm.com/202201/12/61de379c3a9882113.jpg_e600.jpg)

想到反正也是搭建服务转发请求，为什么不自己写一个呢，

于是我就用 java 实现了一个 web 服务来转发，由于是 java 写的，所以不可避免的稍微占用内存一点。。。。

服务很简单，代码也是随便写的，能用就行。。。也是两个接口，转发对应的数据到企业微信

摸鱼搞的，各位轻喷，目前使用没问题，后面没变动的话基本不会去改它了。。。。

这里实现时我用的是卡片消息，看上去好看点![](https://res.smzdm.com/images/emotions/90.png)

由于[腾讯](https://pinpai.m.smzdm.com/19011/)要求必须添加跳转 url，**默认我就跳转了百度首页**![](https://res.smzdm.com/images/emotions/79.png)

各位不点击详情就行

git 地址 [lepark2/wxpush (github.com)](https://github.com/lepark2/wxpush)

最近网络疯狂抽风。。。本来不想搞 git 的，不过想来不上传你们怕是不敢用 (反正来源不明的我是不敢)![](https://res.smzdm.com/images/emotions/90.png)

为了方便大家使用，学习了下 docker，打包了一个 docker 服务到 dockerhub，

大家可以下载来试下 dockerhub 搜 lepark/wxpush 那个就是

使用方法也说下

先 docker pull lepark/wxpush:latest 拉取镜像

sudo docker run -p 50000:50000 -e corpid= 企业微信的 corpid -e corpsecret = 企业微信的 corpsecret

-e agentid = 企业微信应用的 agentid -e wxpush:latest

群晖如图，添加端口，默认 50000

![](https://qnam.smzdm.com/202201/11/61dd54f6270c75059.png_e600.jpg)

然后修改环境变量对应即可

![](https://qnam.smzdm.com/202201/11/61dd55200b96c7560.png_e600.jpg)

威联通类似，使用 container station 搜索 wxpush 转发好端口，设置好环境变量即可

安装好后使用方法同上面的 nodered，只需要修改 url 对应 ip 和端口为容器使用的即可

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～