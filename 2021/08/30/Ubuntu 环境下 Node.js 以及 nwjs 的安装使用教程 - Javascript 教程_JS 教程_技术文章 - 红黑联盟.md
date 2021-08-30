> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.2cto.com](https://www.2cto.com/kf/201710/688665.html)

> Ubuntu 环境下 Node js 以及 nwjs 的安装使用教程。需求：通过 nwjs 实现一个可以全屏显示，防止用户退出浏览器的演示用 app。

* * *

Ubuntu 环境下 Node.js 以及 nwjs 的安装使用教程。需求：通过 nwjs 实现一个可以全屏显示，防止用户退出[浏览器](https://www.2cto.com/os/liulanqi/)的演示用 app。

* * *

### 一、安装 Node.js

```
sudo apt-get install nodejs  
sudo apt-get install npm

```

这种方法安装的版本可能不是最新的，可以尝试直接从官网 (https://nodejs.org/en/download/) 下载。  
解压下载的 node-v6.11.4-linux-x64.tar.gz 文件之后建立软链接即可：

```
sudo ln -s ~/node-v6.11.4-linux-x64/bin/node /usr/local/bin/node
sudo ln -s ~/node-v6.11.4-linux-x64/bin/npm /usr/local/bin/npm
sudo ldconfig 

```

如果下载的是 Source Code(node-v6.11.4.tar.gz)，那么需要在解压后的目录自行编译：

```
./configure  
make  
make install  

```

> 如果遇到所需环境版本问题，在 sudo apt-get update && apt-get upgrade 后用 apt-get remove 旧版本后 install 就好

### 二、安装 nwjs

在官网 (http://nwjs.io/) 上下载 nwjs-sdk-v0.25.4-linux-x64.tar.gz，解压后建立软链接：

```
sudo ln -s ~/nwjs-v0.18.8-linux-x64/nw /usr/local/bin/nw
sudo ldconfig

```

进入目录后可以看到 nw 文件，运行./nw 命令查看是否可以正常运行。

### 三、nwjs 使用示例

#### 1. 创建 html 文件

首先创建一个简单的 Demo.html 文件：

 

[Visit Baidu](http://www.baidu.com/)

#### 2. 创建 package.json 文件

用来进行初始化配置：

```
{
    "name": "Demo",
    "main": "Demo.html",
    "window": {
        "title": "Demo",
        "toolbar": false,
        "frame": true,
        "position": "center",
        "always-on-top": true,
        "fullscreen": true,
        "width": 1920,
        "heigth": 1080
    }
}

```

> 这里实现了一个去掉了工具栏的全屏效果

#### 3. 打包文件运行

创建好两个文件之后将其打包：

```
cat package.json Demo.html > Demo.nw

```

这时新打包出来的 nw 文件就可以运行了：

```
sudo ./nw Demo.nw

```

### 创建桌面图标

这里想要实现一个双击启动的效果，类似. exe 文件的运行效果。根据官方教程使用 cat `which nw` app.nw > app && chmod +x app 创建的 app 一直无法使用，所以想出了一个取巧的办法：  
1. 创建一个 bash 脚本 start.sh 启动 Demo.nw：

```
#!/bin/bash
cd /home/ubuntu/Desktop/nwjs-v0.18.8-linux-x64
./nw demo.nw

```

找一个 app 图标，命名为 icon.jpg 在桌面创建一个 Demo.desktop 图标，使用 sudo nano Demo.desktop 命令打开后写入：

```
[Desktop Entry]
Encoding=UTF-8
Name=Demo
Exec=sh /home/ubuntu/Desktop/nwjs-v0.18.8-linux-x64/start.sh
Icon=/home/ubuntu/Desktop/nwjs-v0.18.8-linux-x64/icon.jpg
Info="Spark"
Categories=GTK;Network;message;
Comment="demo_nwjs"
Terminal=false
Type=Application
StartupNotify=true
Name[zh_CN]=Demo

```

这时双击就能看到运行结果了

### 屏蔽按键

package.json 中配置了全屏并且去掉工具栏，就是为了防止用户退出浏览器，具体操作步骤如下：  
1. 在设置中可以关闭快捷键，并自定义一个快捷键呼出 teminal 用来退出浏览器，例如 Shift_R + Ctrl_R + Q。  
2. 将键盘左侧的 Shift、Ctrl、Super(win) 键改到 CapsLock 键上，这可以通过 xmodmap 实现：

```
#super_l -> capslock
xmodmap -e "remove mod4 = Super_L"
xmodmap -e "keycore 133 = Caps_Lock NoSymbol Caps_Lock"
xmodmap -e "add lock = Caps_Lock"
#super_r -> capslock
xmodmap -e "remove mod4 = Super_R"
xmodmap -e "keycore 134 = Caps_Lock NoSymbol Caps_Lock"
xmodmap -e "add lock = Caps_Lock"
#alt_l -> capslock
xmodmap -e "remove mod1 = Alt_L"
xmodmap -e "keycore 64 = Caps_Lock NoSymbol Caps_Lock"
xmodmap -e "add lock = Caps_Lock"
#ctrl_l -> capslock
xmodmap -e "remove control = Control_L"
xmodmap -e "keycore 37 = Caps_Lock NoSymbol Caps_Lock"
xmodmap -e "add lock = Caps_Lock"
#shift_l -> capslock
xmodmap -e "remove shift = Shift_L"
xmodmap -e "keycore 50 = Caps_Lock NoSymbol Caps_Lock"
xmodmap -e "add lock = Caps_Lock"
#alt_r -> capslock
xmodmap -e "remove mod1 = Alt_R"
xmodmap -e "keycore 108 = Caps_Lock NoSymbol Caps_Lock"
xmodmap -e "add lock = Caps_Lock"

```

将上述代码添加到 start.sh 的启动命令之前，这样在双击启动的时候就可以修改掉按键了。