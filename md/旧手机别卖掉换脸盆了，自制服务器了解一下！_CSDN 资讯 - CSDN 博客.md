> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/csdnnews/article/details/118347083)

作者 | Hannah Lee       

译者 | 弯月     责编 | 欧阳姝黎

出品 | CSDN（ID：CSDNnews）

本文将向你展示如何使用 UrBackup 和 Linux Deploy 在一台 Android 旧手机上搭建一台备份服务器。旧手机的污染问题众所周知，我有一台旧手机，虽然外壳有裂纹和磨损，但性能还很好，因此我打算废物再利用一下。

![](https://img-blog.csdnimg.cn/img_convert/4e76bed31449a1aed9b61970aa3adbcd.png)

你的旧手机很可能：

*   并没有那么旧（你会像换手机那样一两年就换一台电脑吗？）
    
*   有 4-8 个处理器和大约 4GB 的内存，以及内置 UPS。只需要再加一个外部硬盘驱动，就可以备份整个家庭的网络了！
    

警告：这只是一个尝试可能性的项目。由于我们使用的是 Android，因此必须克服一些困难，而且很多方面都会受到限制。这可能并不是最简单的备份家庭网络的方法，也不是使用 UrBackup 的最佳用途。但在设置完成，服务器可以正常运行后，你就可以轻松地管理多台机器的备份并添加存储。

下面是一些限制：

1.  文件系统只能使用 EXT4。这是唯一兼容 UrBackup 和 Android 的文件系统。因此没有文件系统级别的压缩等功能。
    
2.  从 chroot 环境下无法直接调用 systemd。我们会安装一个简单的启动脚本，启动 UrBackup 服务，并使用 pkill 停止服务。此外，我们还会添加一个 cronjob，在系统启动时启动服务。
    
3.  UrBackup 主要以 Windows 为主。虽然它提供 Linux 的完整备份功能，但其 Linux 版本的完整镜像依然是 beta 版状态。此处我们会安装稳定版，所以我们的服务器无法支持利用 Linux 客户端进行完整镜像备份。但是，你可以在客户端上创建备份镜像，然后备份含有这些镜像的目录。最后，UrBackup 团队也在开发 Mac 客户端支持，但同样是 beta 版。对于我来说，我没有 Windows 服务器，我也不想备份我的 Windows 笔记本电脑，所以我只在 Linux 服务器上使用 UrBackup。
    
4.  Linux Deploy 提供的发行版和版本支持很有限。我试验了 Centos7、Ubuntu 18.04 和 Debian 10，其中只有 Debian 10 能够毫无错误地运行。UrBackup 对 Debian 的支持也为最好（其他发行版都没有 ARM64 的 .deb 包）。
    

如果你打算与我一起尝试一下，则请看下面的行动计划：

1.  root 手机。具体做法请参考相关文档，不在此赘述。
    
2.  安装 Linux Deploy 并部署 Debian Buster。
    
3.  安装 UrBackup 服务器。
    
4.  连接客户端。
    

请记住，本教程采用了非常特殊的配置，，其中大部分是为了适应在 chroot 环境中，在 Android 上使用 Debian。

![](https://img-blog.csdnimg.cn/img_convert/d6de2fe05162c7f1945546354b0335b7.png)

![](https://img-blog.csdnimg.cn/img_convert/2d7b7741385653b54b2c31ffb7e2be67.png)

**准备工作**
========

**root Android 手机**

我的手机是 Pixel 4a（8 核 CPU，6GB 内存）。手机的特定型号应该没有太大关系，但不同的型号可能会遇到不同的问题。如果你的手机是在过去五年内发布的，那么规格上应该没问题。我建议至少 4 核 CPU 和 2GB 的内存，这对于大多数家庭网络来说应该就够了，但是你必须想清楚备份要求。此外不要忘记， Android 操作系统本身将占用一些资源。

如果你打算通过 Magisk 来 root 手机，则请注意：

*   确保你使用的 boot.img 文件与手机当前的引导程序版本相符。
    
*   如果你使用的是 Android 11，而且 Magisk 程序无法正常工作，请降级到 Android 10 再试。我在 Pixel 4a 上摆弄了一整天的 Android 11。
    
*   如果你不想使用 Magisk，请非常谨慎地使用其他工具。有很多其他应用程序可能含有恶意软件。CF-Auto-Root 也是一款很好的 root 工具，但请确认下载源的安全。
    

**Linux Deploy 应用**

该应用可以在 Debian 服务器上运行 chroot 环境。

你可以从 GitHub 下载最新版本。应用商店中的版本已经没人维护了。

**BusyBox 应用**

该应用可以为 Linux Deploy 提供 Unix 工具程序。可用的 “Busy Box” 应用有好几个，但只有这个版本与 Linux Deploy 兼容。

你可以从 GitHub 下载最新版本。应用商店中的版本已经没人维护了。

**充电线**

手机的充电线。

**从另一台机器上通过 SSH 连接到服务器**

尽管理论上可以在手机上安装 Termux 或 SSH 应用进行操作，但通过键盘进行操作肯定更容易。

![](https://img-blog.csdnimg.cn/img_convert/5b68202df62e4bd25ddce97cdb6f8022.png)

**可选设备（强烈推荐）**
==============

尽管从技术的角度来看，你可以将备份存储在 SD 卡上，但不建议这样做。SD 卡的速度较慢，可靠性较低，并且无法长时间处理持续写入。如果你计划备份到 SD 卡，那么估计一年内就会损坏（如果数据量大，甚至一个月内就会出问题）。

**带 USB 线的外置 HDD/SSD**

大小和写入速度取决于你个人，但我更关心可靠性，而不是存储和速度。如果你有大量存储空间（几百甚至几千 GB），则速度很重要。在这种情况下，写入速度很关键，因为你可以及时完成备份。为了可靠性，我们必须考虑品牌。使用廉价的驱动器，就要做好心理准备备份过程中会出现 I/O 错误。

**USB 扩展坞**

根据你的手机，可能需要支持 micro USB 或 USB-C。这个扩展坞应该至少有一个 USB 端口，可以连接到外部驱动器，而且还有一个充电的端口，但我建议选择一个带有以太网适配器的扩展坞。你可以通过 WiFi 运行该服务器，但以太网更快、更可靠。

**以太网线**

如果你在以太网上运行服务器，则需要准备一个以太网线。

**第一步：安装 Linux Deploy 并部署 Debian Buster**
=========================================

1-1. 在 root 完手机后，打开 GitHub，下载 Linux Deploy 和 BusyBox 的 .apk 软件包，安装这两个应用。

![](https://img-blog.csdnimg.cn/img_convert/14fc57631b8bbd2a4882e3a779f4132b.png)

1-2. 安装完毕后，打开 BusyBox。安装的过程中，记录下 BusyBox 的安装位置。在下图中，BusyBox 安装到了 "/system/xbin"。稍后我们会用到这个位置。

![](https://img-blog.csdnimg.cn/img_convert/aa2740642427311af0098c961458ffa7.png)

1-3. 打开 Linux Deploy，点击右下方的设置图标。

![](https://img-blog.csdnimg.cn/img_convert/4bad53e0a5ff29e5d569b6ee01c4e0f1.png)

完成如下设置：

![](https://img-blog.csdnimg.cn/img_convert/601eedcb65f9d27efc9a18055f23a1f3.png)

**架构**

所有安卓手机都是 AARCH64/ARM64。确保显示的是 “arm64”（或者是其他发行版的 “aarch64”）。

**安装路径**

默认值是 "${EXTERNAL_STORAGE}/linux.img"。这是你的 SD 卡，你可以留着它（可以在树莓派上工作）。但是，如果没有插入 SD 卡或未正确格式化，则安装将失败。我建议安装到你的内部存储中。我假设你不会使用手机干别的事情，因为它需要一直插着电。

**镜像大小（MB）**

我建议至少保留 15 GB，但请确保为 Android 留出足够的存储空间。这部分空间会占用内部存储，因此最后剩下的空间可能没有 50 GB 这么多。

**初始化系统**

如果没有设置为 “sysv”，则 cronjobs 将不会在启动时运行。

**挂载**

如果你不担心将来的存储扩展，则挂载外部块设备时只需将其路径直接添加到挂载点。如果以后有扩展存储的打算，则可以考虑逻辑分区。我们在此加载的设备，都可以在启动时直接访问，但不能用于分区和格式化。但是，请记住，重新启动手机时，块设备的名称 (/dev/block/sdX) 可能会变化，因此可能需要在重新启动时检查 / 更新此配置。如果你挂载的是逻辑卷，则名称不会变化，也不需要检查。

**如何找到外部块设备的路径**

在 Android 上，你可以通过 “/dev/block/sdX”（而不是 “/dev/sdX”）找到块设备。为了确定哪个 sdX 设备是外部块设备，你需要在插入该设备的服务器上运行 “lsblk”。然后搜索各种设备，并查看哪一个与你的设备一致（就存储容量 / 现有分区而言）。如果你在启动后插入设备，则可能是最后一个设备。

注意：挂载块设备后，你必须先解除挂载或关闭服务器，然后才能从物理上断开块设备的连接。如果在未解除挂载的情况下断开块设备的连接，则很可能会丢失所有数据。

1-4. 回到首页，并打开左上角的菜单。选择 “Settings”（设置），并一直向下滚动到 “PATH variable”（路径变量）。这就是你安装 BusyBox 的位置。设置好 “PATH variable” 后，选择 “Update ENV”（更新环境变量）。

![](https://img-blog.csdnimg.cn/img_convert/b2453b1731a57bfb194895590050fb4a.png)

1-5. 返回首页，打开右上角的菜单。点击 “Install”（安装）。你将看到安装的实时日志。完成后，日志将以 “deploy” 结尾。 选择屏幕左下角的 “START”（开始）。Android 手机上就开始运行 Debian 服务器了！

![](https://img-blog.csdnimg.cn/img_convert/0861b4aebe6e28ffdbd9e986f3d88dac.png)

![](https://img-blog.csdnimg.cn/img_convert/d92fbacdf016f62bb83cf92da1e482a9.png)

**第二步：安装 UrBackup 服务器**
=======================

2-1. SSH 到新部署的服务器。

IP 地址与手机相同，端口为 22，你可以使用步骤 1-3 中设置的凭据登录。本教程后续内容均假设你以 root 身份登录。打开 Linux Deploy 就可以看到你的 IP：

![](https://img-blog.csdnimg.cn/img_convert/ac5866e59b89408ae84e1033e147f63f.png)

2-2. 更新系统。

```
apt update && apt upgrade -y && apt install wget
```

2-3. 下载 UrBackup 的 .deb 包。

```
wget https://hndl.urbackup.org/Server/2.4.13/urbackup-server_2.4.13_arm64.deb
```

这是目前最新的稳定版本。

2-4. 创建备份目录。

在这个例子中，我将备份目录设置为 “/mnt/backup”。如果你挂载了存储，则目录已经创建好了；如果没有，请创建目录：

```
mkdir -p /mnt/backup
```

更新权限：

```
chown urbackup /mnt/backup
chgrp urbackup /mnt/backup
```

允许 UrBackup 写入此目录。

2-5. 安装启动脚本。

我们无法在 chroot 环境中调用 systemd，因此需要手动启动该服务：

```
/usr/bin/urbackupsrv run --config /etc/default/urbackupsrv --no-consoletime
```

为了避免每次都输入该命令，我们可以创建一个脚本：

```
nano /usr/bin/urbackupsrv-star
```

将其复制到下面的文件中：

```
#!/bin/sh
 
 
/usr/bin/urbackupsrv run --config /etc/default/urbackupsrv --no-consoletime
```

保存并退出。

添加执行权限：

```
chmod 755 /usr/bin/urbackupsrv-start
```

2-6. 启动服务。

```
urbackupsrv-start
```

该命令将启动服务器的日志。你可以按下 CTRL-C 停止服务，因此需要另开一个 SSH 会话。

如果想停止此服务，只需要干掉它就可以了：

```
pkill urbackup
```

2-7. 添加定时作业。

由于我们无法以传统的方式 “启用” UrBackup 服务，因此需要设置一个定时作业来启动该服务。此外，如果你断开外部块存储或重新启动手机，备份目录的权限可能会恢复。为确保在重新启动时这些设置能保留下来，我们需要添加定时作业。

打开 crontab：

```
crontab -e
```

添加作业：

```
@reboot chown urbackup [full/path/to/backup_directory] && chgrp urbackup [full/path/to/backup_directory] && urbackupsrv-start
```

保存并退出。

2-8. 打开 Web 界面。

通过 Web 浏览器导航到服务器的端口 55414：

http://YOUR_SERVER_IP:55414

![](https://img-blog.csdnimg.cn/img_convert/6c0acd2b53d74fd9f6a25f6209f4d224.png)

![](https://img-blog.csdnimg.cn/img_convert/ca28424335602658b4657122a6622846.png)

**第三步：连接客户端**
=============

3-1. 点击屏幕右下方的 “Add new client”（添加新客户端）：

![](https://img-blog.csdnimg.cn/img_convert/5e6dec6defe9f80ea191306e3b368c19.png)

3-2. 点击 “Add new Internet client/client behind NAT”，并输入新客户端的名称：

![](https://img-blog.csdnimg.cn/img_convert/b62cc0bd2fbcd8d86285f7023eb07842.png)

这是新客户端的主机名。

3-3. 安装客户端。

对于 Windows 客户端：

按照 “Download preconfigured client installer for Windows” 的说明安装客户端。

对于 Linux 客户端：

记下顶部的 “Default authentication key”，回头有需要。

![](https://img-blog.csdnimg.cn/img_convert/f48fefae7ef7baa39615ae17fd1d1663.png)

登录到客户端，并运行此安装脚本：

（不要运行服务器提供的脚本）

TF=$(mktemp) && wget "https://hndl.urbackup.org/Client/2.4.11/UrBackup%20Client%20Linux%202.4.11.sh" -O $TF && sudo sh $TF; rm -f $TF

（检查最新的客户端下载。）

在安装过程中，脚本会要求你选择快照机制。对于 “LVM - Logical Volume Manager snapshots”，请输入 “2”：

![](https://img-blog.csdnimg.cn/img_convert/ac28c0173afd75e43bae2d80be254797.png)

安装完成后，请确认客户端的正常运行：

```
service urbackupclientbackend status
```

如果客户端没有运行，请运行下述命令：

```
service urbackupclientbackend start
```

最后，通过下述命令将客户端连接到服务器：

```
urbackupclientctl set-settings \
-k internet_mode_enabled -v true \
-k internet_server -v "YOUR_SERVER_IP" \
-k internet_server_port -v "55415" \
-k computername -v "YOUR_CLIENT_NAME" \
-k internet_authkey -v "YOUR_DEFAULT_AUTHENTICATION_KEY"
```

请确保 "YOUR_CLIENT_NAME" 与 3-2 中设置的主机名相同，"YOUR_DEFAULT_AUTHENTICATION_KEY" 是服务器前面生成的键。

3-4. 配置服务器。

返回 Web 界面，你会发现客户端并不在线，点击顶部导航栏上的 “Settings”（设置）：

点击 “Settings” 页面上的“Internet”（互联网）页签，检查如下设置：

![](https://img-blog.csdnimg.cn/img_convert/7821709251f61cd49acbfadc2ec020cc.png)

（如果所有客户端都是本地的，则可以取消 “Do image backups over the internet” 以及“Do full file backups over the internet”。）

滚动到底部并单击保存。

3-5. 设置备份目录。

在 “Settings” 页面上，点击 “”Client settings（客户端设置）。选中“Separate settings for this client”，在“File Backups” 下的 “Default directories to backup” 中设置你想备份的目录。如果想添加多个目录，可以用分号 “;” 分隔。

![](https://img-blog.csdnimg.cn/img_convert/27beb63ea5da18c6df0843671157b7bd.png)

你还可以在此设置备份间隔。

点击底部的保存。

对于 Linux 客户端，你也可以从客户端的命令行设置：

```
urbackupclientctl add-backupdir -d FILE_PATH
```

重启服务器：

```
pkill urbackup
urbackupsrv-start
```

3-6. 重新登录到 Web 界面。

到此为止，客户端已经连接好了。可能 “File backup status”（文件备份状态）会显示 “No paths to backup configured”（没有设置备份路径），但没关系，在第一次完成备份之前，都会这显示。

另外请注意，如果你连接的是 Linux 客户端，则不支持镜像备份。但是，你可以通过 Linux 客户端运行镜像备份，并设置备份镜像的目录。

接下来，你就可以尝试一下备份了！

![](https://img-blog.csdnimg.cn/img_convert/3d9af86762e7d525197318f557d6a569.png)

原文链接：https://www.hannahtech.co/post/turn-your-old-cracked-android-phone-into-a-backup-server-urbackup-linux-deploy-tutorial-part-i

声明：本文由 CSDN 翻译，转载请注明来源。

> 开发者必备的知识图谱来啦！60 + 专家，13 个技术领域，CSDN 《IT 人才成长路线图》重磅来袭！直接扫码或微信搜索「CSDN」公众号，后台回复关键词「**路线图**」，即可获取完整路线图！

![](https://img-blog.csdnimg.cn/img_convert/d17f49db313272fed05f541a3b1ab23c.gif)

![](https://img-blog.csdnimg.cn/20210630092226441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NzZG5uZXdz,size_16,color_FFFFFF,t_70)