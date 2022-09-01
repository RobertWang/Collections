> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/40075)

> 通过之前的安装篇，我们已经成功安装了 HASS 和 Homebridge。在这篇文章中，将一步步带领大家接入智能家居设备。

相信大家通过之前的 [安装篇](https://sspai.com/post/38849 "安装篇") 已经成功安装了 Home Assistant (HASS) 和 Homebridge，在这篇文章中，我将带领大家接入智能家居设备。

大多数人初接触 HASS 的时候经常一头雾水，原因是 HASS 的配置体系十分混乱，一个设备的完美接入需要涉及多个配置文件。实际上，系统架构不清晰也是 HASS 的最大缺点，因此，在开始配置教程前，我先帮助大家捋一捋 HASS 的配置框架。

HASS 配置框架
---------

HASS 的核心配置围绕 `configuration.yaml` 文件展开, 在这里你可以进行时区、度量单位、开发者模式、主题选择等等基础配置。当然，最为重要的，你将在该文件内完成所有设备的接入。这也是**本篇教程的重点**。

HASS 的运行依赖于一个个相对独立的功能组件 (Components），比如小米米家平台就可以视作一个组件。有些时候，部分设备或者功能仍未得到 HASS 的官方支持，你必须在主目录下新建自定义组件`custom_components`文件夹，添加相关的设备支持文件。

完成上述的文件修改，加上 HB 的配置，你就可以自如控制所有智能家居设备了。

然而，使用一段时间后，你可能会觉得设备太多显示凌乱，想给界面换个风格，或者`configuration.yaml`文件看起来要炸了。此时，你会考虑把部分配置剥离出去形成独立的文件，以满足你的强迫症。例如你开启了 “设备追踪功能”（Device Tracker），那么 HASS 将在主文件下自动生成 `known_devices.yaml`文件，你将在这里配置需要追踪的设备。**本篇教程也将涉及相关内容**。至于主题设置、群组设置等其他非功能性设置，我将在之后的” 个性化配置 “中详细介绍。

主文件设置
-----

上一篇教程中我们已经打开了树莓派的 SMB 服务，现在我们通过 SMB 打开 HASS 主目录。  
（macOS 在 FInder 左侧栏 “共享的” 接入，Windows 在 计算机 - 地址栏 里直接输入 `//树莓派地址` 即可跳转)  
打开`configuration.yaml`，文件默认包含如下内容，我们按需修改：

```
homeassistant:
  #经纬度
  latitude: 32.87336
  longitude: 117.22743
  #海拔
  elevation: 430
  #度量单位，默认米
  unit_system: metric
  #时区
  time_zone:Asia/Shanghai
  #系统昵称，显示在主界面顶部
  name: Home


```

正常情况下，剩下的部分便无须变动了。现在添加雅虎天气服务小试牛刀一下吧：

```
weather:
  - platform: yweather
    woeid:2151849


```

其中，woeid 是城市代码，打开雅虎天气官网输入城市后搜索，url 的最后几位数字便是  

![](https://cdn.sspai.com/2017/07/19/47e7e98415e250b485839cedc5773fad.jpg)woeid

保存，重启 HASS。

恭喜你，接入了第一个 HASS 组件~ 现在你大概明白 HASS 是怎么个操作原理了，HASS 支持上千款智能家居设备，你可以[到此](https://home-assistant.io/components/ "到此")寻找你的设备按上述方法接入。

如果你是果家用户，需要 Homekit 服务，那么我们还需要转到 Homebridge 进行相关设置。

Homebridge 设置
-------------

执行指令前，请先运行一次 Homebridge。**注意：如果添加了开机自启任务，勿重复运行 Homebridge，否则会出现端口占用错误。**  
**Homebridge - homeassistant 插件版本为 2.3.0 以上的，特别注意添加最后一行配置，否则家庭 app 内设备为空。**

```
cd /home/pi/.homebridge
sudo nano config.json
{
"bridge": {
"name": "Homebridge",
"username": "CC:22:3D:E3:CE:30",     //树莓派 mac 地址
"port": 51826,     //运行端口
"pin": "123-45-678"    //连接密码，自行设定
},
"platforms": [
{
"platform": "HomeAssistant",
"name": "HomeAssistant",
"host": "http://127.0.0.1:8123",     //HA 运行的网址，可以是 ip 也可以是域名
"password": "raspberry",     //HA 的 api_password，及密码，如有设置请添加
"supported_types": ["automation", "binary_sensor", "climate", "cover", "device_tracker", "fan", "group", "input_boolean", "light", "lock", "media_player", "remote", "scene", "sensor", "switch"],
"default_visibility": "visible"    //特别注意此项
}
]
}


```

ctrl+x，y，回车。之后清除 Homebridge 的缓存：

```
sudo rm -rf /home/pi/.homebridge/persist/


```

**请大家记住此步指令，今后若出现重新配置 HA、HB 导致 iOS 设备无法识别新设备或树莓派的，大部分情况均可以使用此指令解决。**

这样我们就完成了 Homebridge 的设置，重启 Homebridge：

```
sudo systemctl restart homebridge


```

经过上述设置，我相信你的智能家居设备已经在 HA 和 HB 里稳定运转了，现在不妨尝试使用 Siri 操控你的设备。

鹬蚌相争？
-----

在之前的 [安装篇](https://sspai.com/post/38849 "安装篇") 中我们知道 Homebridge 本身可以通过安装插件的方式将智能设备接入 Apple Home 平台，有的时候设备同时支持 HA 和 Apple Homekit 2 个平台，这时我们就面临平台选择。

首先，这个问题只存在 iOS 以及未来的 macOS 用户身上，因为只有你们可以使用 Siri，才可以痛并快乐着。 对于其他终端系统的用户而言，要么和设备厂家的 app 斗智斗勇，要么享受 HA 网页操控的快感，至于语音控制，就还需等待了。

其次，以米家设备为例，同是网关，HA 可以控制夜灯功能，获取光感数据，Homekit 则不能；而针对扫地机器人，HASS 的接入方法十分复杂，且现阶段只能实现开关功能，HB 的插件安装配置更为容易，且可以控制吸力…… 可以说两个平台之间没有绝对的胜者，都需要具体情况具体分析。个人认为大多数情况下，HA 对设备的支持更好，并且基于社群支持，跟进速度也更快。

对于已经将设备通过 HB 接入，但想转入 HA 平台的派友，这里提供如下兼容方案：

1.  在 HA 接入相关设备，详见上文。
2.  在 HA 主页面侧边栏，点击下方 “<>”，获取设备的 entity ID：  
    ![](https://cdn.sspai.com/2017/07/30/a12570a56afa72143ea842790a4dbcb3.jpg)States 面板
3.  在 HA `configuration.yaml` 配置文件中添加如下设置：  
    ![](https://cdn.sspai.com/2017/07/30/769d4a78065db8ce46fdf7e546d6b21d.jpg)设备个性化  
    则该设备将在 Homebridge 中被隐藏，重启后不会被 家庭 App 重复识别添加。  
    此时，你可以通过 Apple Homekit 和 Hass 平台同时控制该设备。

设备追踪设置
------

HA 可以追踪同一路由器内网设备联网状态，我们转换一下思路，便可以利用这个系统判断家人是否在家等，非常方便。

![](https://cdn.sspai.com/2017/07/19/9da22ed3c7e53ae1d939888e292f4ef2.jpg)家人追踪

我的路由器是 ASUS AC66U-B1，HASS 原生支持大部分品牌的路由器，包括小米路由器、TP-LINK 等。不同的路由器对应配置方法不同，具体请大家前往 [官网](https://home-assistant.io/components/ "官网") 查询。设置前需要**打开路由器的 SSH 模式**，请大家设置好密码等，保护数据安全。建议设置仅限 LAN 用户拥有 SSH 权限。

打开`configuration.yaml`文件，末尾添加：

```
device_tracker:
  - platform: asuswrt （按需填写，小米为“xiaomi”，Netgear为“netgear”，TP-LINK为“tplink”）
    host: 192.168.xx.1 #路由器Ip
    username: ***** #管理员账号
    password: ****** #管理员密码
    track_new_devices: no #是否自动添加新设备


```

不出意外，重启 HASS 后，主文件夹下便会自动生成 `known_devices.yaml` 文件，打开之后你会发现，系统已经自动为嗅探到的连接到路由器的设备添加了默认配置：

```
devicename: 
  name: Friendly Name #昵称
  mac: EA:AA:55:E7:C6:94 #mac地址
  picture: https://home-assistant.io/images/favicon-192x192.png #图片icon: mdi:human-female #图标，和图片取一个设置
  track: yes #是否追踪
  hide_if_away: no #离开后是否自动隐藏


```

在接下来的教程中，我将指导大家如何进行自动化配置，真正实现人工智能，摆脱遥控的束缚、更换主题，展现个性、设置简洁大方的控制面板等等等等  

![](https://cdn.sspai.com/2017/07/26/a03014830a2fb8e4341d130c87ad0171.jpeg)HADashboard

敬请期待~

欢迎继续阅读系列其他文章：

小米设备特别篇：

Home Assistant + 树莓... 小米的智能家居设备物美价廉，博得了国内外不少用户的喜爱。本... [sspai.com](https://sspai.com/post/40113) ![](https://cdn.sspai.com/article/b700b183-56aa-fb9e-861b-dc049732f464.jpg)

个性化配置篇：

Home Assistant + 树莓... 通过之前的篇章，相信大家已经能够较舒适地使用 HA 这个平... [sspai.com](https://sspai.com/post/40272) ![](https://cdn.sspai.com/2017/08/24/899b1c96238f2ea68494eacf671e93cf.jpg)

更新日志
----

*   09.28 修改 HB 配置；
*   08.27 发布『[答疑篇](https://sspai.com/post/40623 "答疑篇")』，开放 Q&A，HA 最新版本为 0.52.0；
*   08.22 配合 homebridge-homeassistant v2.3.0，修改配置；
*   08.08 关闭评论区；
*   08.05 更改 config.json 设置路径，增加删除 HB 缓存方法。

作者的话
----

距离我发布本系列首篇文章至今已经快半年了，系列至今已有 10 篇教程。无独有偶，JailbreakHum 在 [《少数派季度作者颁奖礼开场发言》](https://sspai.com/post/41257 "《少数派季度作者颁奖礼开场发言》")所阐述的『内容调整』的观点恰恰与我的初衷十分契合。可以说从一开始，我就在整个系列中尽量保证内容的入门性，也在现实中帮助了很多派友成功搭建系统。

在和派友的互动中我意识到碍于英文水平的局限，很多人无法进一步地享受 Home Assistant 带来的便利。鉴于此，我个人独立制作了一份更接地气的 [中文文档](https://home-assistant.cc "中文文档") ，目前仍处于雏形阶段，还在快马加鞭中，希望可以帮助到更多的人，欢迎大家阅读 + 收藏。

良好的体验需要良性的互动维持，为了保持版面的整洁，请大家不要在系列所有文章下方评论区 PO 整段的错误代码。遇到问题请至[『维护答疑篇』](https://sspai.com/post/40623 "『维护答疑篇』")集中评论。

Home Assistant + 树莓...

小米的智能家居设备物美价廉，博得了国内外不少用户的喜爱。本...

[sspai.com](https://sspai.com/post/40113)

Home Assistant + 树莓...

通过之前的篇章，相信大家已经能够较舒适地使用 HA 这个平...

[sspai.com](https://sspai.com/post/40272)