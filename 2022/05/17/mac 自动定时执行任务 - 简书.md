> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/44313b350b70)

1、launchctl
-----------

> launchctl: 是一个统一的服务管理框架，可以启动、停止和管理守护进程、应用程序、进程和脚本等。

> launchctl 是通过配置文件来指定执行周期和任务的。

当然 mac 也可以像 linux 系统一样，使用 crontab 命令来添加定时任务，这里就不赘述，具体可参见：[OS X 添加定时任务](http://codingpub.github.io/2016/10/27/OS-X-%E6%B7%BB%E5%8A%A0%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1/)

### 配置文件（plist 文件）

launchctl 将根据 plist 文件的信息来启动任务。

##### plist 脚本一般存放在以下目录：

*   `/Library/LaunchDaemons` --> 只要系统启动了，哪怕用户不登陆系统也会被执行
*   `/Library/LaunchAgents` --> 当用户登陆系统后才会被执行

##### 更多的 plist 存放目录：

> ~/Library/LaunchAgents 由用户自己定义的任务项

> /Library/LaunchAgents 由管理员为用户定义的任务项

> /Library/LaunchDaemons 由管理员定义的守护进程任务项

> /System/Library/LaunchAgents 由 Mac OS X 为用户定义的任务项

> /System/Library/LaunchDaemons 由 Mac OS X 定义的守护进程任务项

### plist 部分参数说明：

*   Label：对应的需要保证全局唯一性；
*   Program：要运行的程序；
*   ProgramArguments：命令语句
*   StartCalendarInterval：运行的时间，单个时间点使用 dict，多个时间点使用 array <dict>
*   StartInterval：指定脚本每间隔多长时间执行一次，与 StartCalendarInterval 使用其一，单位为秒
*   StandardInPath、StandardOutPath、StandardErrorPath：标准的输入输出错误文件，这里建议不要使用 .log 作为后缀，会打不开里面的信息。
*   定时启动任务时，如果涉及到网络，但是电脑处于睡眠状态，是执行不了的，这个时候，可以定时的启动屏幕就好了

任务相关命令
------

```
# 加载任务, -w选项会将plist文件中无效的key覆盖掉，建议加上
$ launchctl load -w com.test.plist

# 删除任务
$ launchctl unload -w com.test.plist

# 查看任务列表, 使用 grep '任务部分名字' 过滤
$ launchctl list | grep 'test.demo'

# 开始任务
$ launchctl start  com.test.plist

# 结束任务
$ launchctl stop   com.test.plist
```

**注意：**  
如果任务被修改了，那么必须先 unload，然后重新 load  
start 可以测试任务，这个是立即执行，不管时间到了没有  
执行 start 和 unload 前，任务必须先 load 过，否则报错  
stop 可以停止任务

### 实战

创建一个定时任务：

任务目标：没一分钟执行一次 shell 脚本

#### 创建脚本文件，以及一些接收文件

1、创建一个简单的 Shell 脚本。

脚本任务：输出时间到 time.txt 文件中

```
vim ~/Desktop/testShell.sh
```

并在`testShell.sh`输入

```
time=$(date "+%Y-%m-%d %H：%M：%S")
echo "$time" >> ~/Desktop/time.txt
```

如图：

![](http://upload-images.jianshu.io/upload_images/6165105-8657b4a7cb624528.png) F1E2709D-F898-474F-9B8A-9E20155315FD.png

2、创建 time.txt 文件

```
touch ~/Desktop/time.txt
```

3、创建一个标准输出文件

```
touch ~/Desktop/stdout
```

#### 2、创建任务描述文件

```
vim ~/Library/LaunchAgents/com.test.plist
```

并在 plist 文件中输入

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- Label唯一的标识 -->
    <key>Label</key>
    <string>com.test</string>
    <!-- 指定要运行的脚本 -->
    <key>ProgramArguments</key>
    <array>
        <string>/Users/jamalping/Desktop/testShell.sh</string>
    </array>
    <!-- 指定运行的时间 -->
    <key>StartCalendarInterval</key>
    <dict>
        <key>Minute</key>
        <integer>1</integer>
        <key>Hour</key>
        <integer>0</integer>
    </dict>
    <!-- 时间间隔 -->
    <key>StartInterval</key>
    <integer>3</integer>
    <key>StandardOutPath</key>
    <!-- 标准输出文件 -->
    <string>/Users/jamalping/Desktop/stdout</string>
    <!-- 标准错误输出文件，错误日志 -->
    <key>StandardErrorPath</key>
    <string>/Users/jamalping/Desktop/error.txt</string>
</dict>
</plist>
```

*   加载任务

```
launchctl load -w com.test.plist
```

*   开始任务

```
launchctl start -w com.test.plist
```

接下来就可以在 time.txt 文件中看到输出时间了。  
如图：

![](http://upload-images.jianshu.io/upload_images/6165105-12d1ebdc8bc359f4.png) 6D1D2D9C-BA5E-4AC3-BE53-8A3163C5E131.png