> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [wizyoung.github.io](https://wizyoung.github.io/vps-selfhosted-private-cloud-setting-up/)

> 前段时间和师弟聊天，谈到网盘的问题，我们一致认为国内的网盘体验真的是太差了。

前段时间和师弟聊天，谈到网盘的问题，我们一致认为国内的网盘体验真的是太差了。网络不限速、文件不和谐、支持随意分享、支持直链下载、支持命令行上传下载，这几个刚需基本上随便一家国内网盘都有至少 3 点不满足，加上普遍会员价格昂贵，实在没有购买使用的欲望。尤其是支持命令行上传下载，简直是刚需中的刚需了，因为它确保了我能在没有图形界面的 Linux 机器上使用。

其实，只要手头有一台 VPS，可以很方便地搭建一个私有网盘，满足自己的各种需求。GitHub 上有很多成熟的开源网盘方案，如 NextCloud 等，这里不介绍它们的安装使用，因为它们配置相对复杂 (小白不友好)，系统资源消耗大 (vps 小鸡不友好)。这里推荐两个比较轻量的解决方案：[File Browser](https://github.com/filebrowser/filebrowser) 和 [Updog](https://github.com/sc0tfree/updog)，基本上都是一行命令搞定。

1. File Browser
---------------

[File Browser](https://github.com/filebrowser/filebrowser) 是一个 GO 语言编写的 WEB 文件管理器，提供了文件上传、下载、删除、预览、重命名、分享等诸多功能，同时支持多用户管理，功能非常齐全。它的界面如下图：  
![](https://wizyoung.github.io/post-images/1582697132943.gif)

File Browser 的使用非常简单，从 Github release 界面下载对应 linux 发行版安装包解压，其实就是一个名为 filebrowser 的可执行文件，**无需任何依赖**，大小仅 30 M+，非常轻量。

部署仅需一条命令行：

```
filebrowser -a 0.0.0.0 -p 8007 -r /root -d /root/filebrowser.db
```

这样就指定 `/root` 目录为网盘根目录，在 `8007` 端口下启动了一个私有网盘。`-d` 命令指定了存放用户信息的数据库的目录，第一次运行会自动生成。访问 http://your_vps_ip:8007 就可以登入刚刚搭建好的网盘，默认用户名和密码均为 `admin`，登入后在设置里面可以更改。

用 nohup 或者 screen 等可以让 File Browser 后台运行，不过为了更方便地使用，我建议使用 systemd 新建一个开机启动的守护进程服务，后台常驻 (因为确实好用)：

在 `etc/systemd/system` 下创建 `fb.service` 文件，内容填写:

```
[Unit]
Description=filebrowser service
After=network.target

[Service]
Type=simple
Restart=always

ExecStart=path_to_filebrowser -a 0.0.0.0 -p 8007 -r /root -d /root/filebrowser.db

[Install]
WantedBy=multi-user.target
```

注意上面的 `file_to_filebrowser` 填写 File Browser 的绝对路径。

执行以下两条指令使服务生效并启动:

```
sudo systemctl enable fb.service
sudo systemctl start fb.service
```

前面我提到 filebrowser 非常轻量，在我的 VPS 上，用 systemctl 查看，该服务仅仅占用 18 M 内存而已：  
![](https://wizyoung.github.io/post-images/1582698115434.png)

**文件分享**  
我非常喜欢 File Browser 的文件分享功能，它可以指定分享外链在一段时间后失效：  
![](https://wizyoung.github.io/post-images/1582698217800.png)

**直链下载**  
前面文件分享获取链接后，复制该链接在浏览器新页面打开，在文件名上右键选择`复制链接地址`即可得到直链地址。

**命令行上传下载**  
在 GitHub 的 issues 里面，作者的多次回复表明他似乎并不希望用户使用命令行进行文件上传和下载 (这点我实在想不通)，然而作者依然透露了 [api 接口](https://github.com/filebrowser/filebrowser/issues/217)，这样，我们可以通过简单的 http post 上传文件。然而我发现作者提供的 api 在版本 2.0 后就发生了变动不再有效，通过简单的分析，我获取了新版的 api 的正确使用方式。我手头上两台 VPS 之前安装的 File Browser 版本为 2.0.15 和 2.0.16，以下方法上传文件均是成功的:  
利用 curl 命令通过 post 上传文件：  
首先我们要获取用于认证的 token:

```
TOKEN=$(curl -X POST --data '{"username":"your_username", "password":"your_password"}' http://your_vps_ip:your_port/api/login)
```

假定 File Browser 指定的文件根目录下有个文件夹 test_dir，要把本机当前目录下的文件 test.png 上传到远端 test_dir 下：

```
curl -X POST \
     --header "X-Auth: $TOKEN" \
     -F "file=@test.png" \
     -o /dev/null \
     http://your_vps_ip:your_port/api/resources/test_dir/test.png
```

效果如下图：  
![](https://wizyoung.github.io/post-images/1582712169852.png)

默认设定是如果文件已存在，则不进行覆盖。可以添加参数 `override=true` 覆盖旧文件:

```
curl -X POST \
     --header "X-Auth: $TOKEN" \
     -F "file=@test.png" \
     -o /dev/null \
     http://your_vps_ip:your_port/api/resources/test_dir/test.png?override=true
```

同样，下载也可以类似进行，不过我觉得还是通过分享链接获取直链下载比较方便。

2. Updog
--------

[Updog](https://github.com/sc0tfree/updog) 是我昨天发现的一个极简 HTTP server, 官方的介绍是用用于替代 Python 内置的 SimpleHTTPServer，初步使用后觉得还不错，比 SimpleHTTPServer 多了文件上传和文件预览的功能，同时 UI 上也更好看一些。

显示效果:  
![](https://wizyoung.github.io/post-images/1582722711606.png)

最上方显示 Updog 的 logo, 以及启动服务器的根目录路径，下面是文件列表。单击文件名开始下载，单击每一项右侧的 `View in browser` 可以进行简单的文件预览。

安装方法极其简单：

```
pip3 install updog
```

部署同样只需要一条指令：

```
updog -d /your_directory -p 1234 --password 123
```

这样，就在指定的目录下启动了一个简单的 http server, 端口是 1234, 密码是 123 (用户名留空，浏览器登入时不填写)。省略 `--password` 则不设置密码。设置开机后以守护进程自启动参考 File Browser 的设定，只需修改 `ExecStart` 命令。

**命令行下载**  
Updog 没有像 File Browser 那样的文件分享功能，不过在需要分享的文件的目录下临时启动一个 Updog 服务也能达到类似的效果。

命令行下载比较简单，在需要的文件上右键拷贝链接，使用 wget 进行 http 认证下载即可:

```
wget --continue \
     --http-user="" \
     --http-password="your_password" \
     http://your_vps_ip:your_port/test
```

**命令行上传**  
对 Updog 的源码进行简单的分析发现，我们同样可以用 curl 命令实现用户认证和文件上传。

以上面显示效果图为例，假定我们要把当前本地目录下的 test.py 文件上传到远端 tools 目录下，curl 上传的命令是：

```
curl -X POST \
     -F file=@test.py \
     -F path=/home/scotfree/tools \
     -o /dev/null \
     -u :your_password http://your_vps_ip:your_port/upload
```

题外话：现如今，GFW 技术越来越强，不搞点高级的过墙方法还真容易被干扰。很多新兴协议新增了探测防御的功能，遇到主动审查直接跳转至本机的网盘服务即可，那么本文提到的 File Browser 和 Updog 就是很好的私有网盘实现方案。这样同一个域名，既可以用于过墙，又可以用于私有网盘，使用方便，安全性**相对**更好。事实上，在这套方案下，我的多台 vps 挺过了多次敏感时期，从未被墙稳如老狗，哈哈。