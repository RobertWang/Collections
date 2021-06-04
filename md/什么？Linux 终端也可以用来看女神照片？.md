> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3NTgyODQ1Nw==&mid=2247485577&idx=1&sn=481418e5efdfd0b06095c6e07d890628&chksm=fd1c700fca6bf9194a6657ab06562f4f0890967d7491880804d0c9aa08a983ee1ab14a4daf34&mpshare=1&scene=24&srcid=&key=89d12b870c1b66b5776696fd5e0ca8ca40eaf2c5b8f59e275ec8b2b66d31d80e1511e5aaaf6452883e235aee138375957634612765e269e0495dfb95b7210430939cb9eb66aec9c0a61c3d53eddc1a0a&ascene=14&uin=MTA1OTczMzYxNA==&devicetype=Windows%2010&version=62060739&lang=zh_CN&pass_ticket=N6qf9AQdQt8Tag6A3EbRWR7riqctCu6PBGWhO/AO3LiQ77ioWkh/BDrjY2i/uOaK)

Linux 终端给人的感觉就是黑漆漆一片，里面只能显示一些字符，而从来没见过显示图片的，如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/IsrmVA0RIYPZtnvhxlx4tiaFF2VHQ6iaoCubibYTiaLwcXRa5eSPbZ0bmtYkHHSW2aTwbnARYmS48X9luDNvfhDlZQ/640?wx_fmt=png)

但是，实际上，Linux 终端除了显示字符外，当然也可以显示图片（然后就可以用来看女神照片）。具体怎么操作呢？一起跟良许来操作吧。

为了显示图片，我们使用了一个工具—— `lsix` 。这个工具的名称很像是 ls 命令，但它只用来显示图片。那么，这个工具有什么特色呢？

*   自动根据你的终端的前景色与背景色，以最优的方式来显示图像
    
*   不仅在电脑终端里可以直接用，还可以通过 SSH 的方式来远程使用
    
*   除了支持图像，还支持一些非图像格式，如：.svg, .eps, .pdf, .xcf 等等
    
*   工具是用 BASH 写的，所以大部分 Linux 发行版都可以用
    

#### lsix 工具的安装

lsix 会使用到 `ImageMagick` 这个工具，所以在此之前需要先安装好 ImageMagick 。大部分 Linux 发行版都已经默认安装了 ImageMagick ，如果没有的话就需要自行安装了。

对于 Arch Linux 以及它的延伸版本（如 Antergos， Manjaro Linux），安装命令如下：

```
sudo pacman -S imagemagick
```

对于 Debian，Ubuntu，Linux Mint 这个系列的，安装命令为：

```
sudo apt-get install imagemagick
```

lsix 本身其实就是个 BASH 脚本，所以无需进行安装，只需将它下载下来，并移动到 `$PATH` 环境变量里。就这么简单！

首先将它下载到本地计算机：

```
wget https://github.com/hackerb9/lsix/archive/master.zip
```

然后再将它解压：

```
unzip lsix-master.zip
```

解压之后，将得到一个 `lsix-master` 的目录。将目录里的 lsix 文件拷备到环境变量 $PATH 里，比如 /usr/local/bin/ ：

```
sudo cp lsix-master/lsix /usr/local/bin/
```

最后，再赋予它可执行权限：

```
sudo chmod +x /usr/local/bin/lsix
```

接下来，就可以愉快地使用这个工具啦。

但在使用之前，要先确保你的终端支持 Sixel 格式。开发人员在 Xterm 上以 vt340 仿真模式来开发了 lsix ，但 Xterm 并不默认支持 Sixel 。启动支持 Sixel 的方式如下：

```
xterm -ti vt340
```

运行这条命令之后，将弹出另外一个窗口，即 Xterm ，它已经支持了 Sixel 。

如果你想要 Xterm 默认开启 Sixel ，需要修改它的 **.Xresources** 文件（如果没有这个文件，直接创建一个即可）：

```
vim .Xresources
```

在文件里添加这么一句：

```
xterm*decTerminalID    :   vt340
```

再之后，按 **ESC** 后输入 **:wq** 保存退出。

最后，运行以下命令来应用这个改动：

```
xrdb -merge .Xresources
```

这样， Xterm 就默认开启了 Siexl 模式，以后机器关机后再开机也不受影响。

#### 在终端里显示图像

开启一个 Xterm 终端，这个终端长得和系统自带的终端差不多，如下图示：

![](https://mmbiz.qpic.cn/mmbiz_png/IsrmVA0RIYPZtnvhxlx4tiaFF2VHQ6iaoCASKsyGhQQWyl2o1KXuAxPtV1jY1yZx1v3ejI6YoMMj2vFOXxbuWqXw/640?wx_fmt=png)

然后就可以玩 lsix 这个工具啦，比如我现在在终端里显示我的 logo ，只需在 lsix 后面跟上 logo 的绝对或相对路径即可：

```
lsix logo.jpg
```

![](https://mmbiz.qpic.cn/mmbiz_png/IsrmVA0RIYPZtnvhxlx4tiaFF2VHQ6iaoCpIrkianianHqqB4vf9IKiaHUKg9e7A8RGrnicI9vwiaH2sTQrUiaCjlSE2qA/640?wx_fmt=png)

如果要显示当前目录下所有的文件，那更简单，只需一个 lsix 命令就可以：

```
lsix
```

![](https://mmbiz.qpic.cn/mmbiz_png/IsrmVA0RIYPZtnvhxlx4tiaFF2VHQ6iaoCibZtE3yZH8BibOL1eXJBY2XQdCYtHoBJWdXRBCzkWuv05Icw0VeIdsfw/640?wx_fmt=png)

当然它也支持通配符，比如要显示当前目录下所有的 jpg 文件，可以这样：

```
lsix *.jpg
```

![](https://mmbiz.qpic.cn/mmbiz_png/IsrmVA0RIYPZtnvhxlx4tiaFF2VHQ6iaoCZniczLm8fsibz2RoEgd4MYwFiawvmsUaibAYLZhyQN7CkTQiczzSks7e5gA/640?wx_fmt=png)

如果是通过 ssh 到服务器的，也是一样会弹出 Xterm 窗口来显示图片。

怎么样，是不是很简单？以后代码写累的时候可以偷偷用终端来看保存在硬盘里的女神照片！