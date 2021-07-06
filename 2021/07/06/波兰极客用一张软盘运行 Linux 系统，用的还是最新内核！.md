> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzNjc1NzUzMw==&mid=2247584762&idx=4&sn=6e9b0e90baa95a9446e43c590e7b83a0&chksm=e8d13c88dfa6b59eb9787291ecdea9bcc6477e10250d189bf2ba6d14ff00d9b661583a6fab11&mpshare=1&scene=1&srcid=0706WtKgcf6IKScHSSvp9MiN&sharer_sharetime=1625552533690&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**用软盘启动 Linux 系统**曾经很 “家常便饭”，当然那都是 90-00 年代的事了。

一张软盘装下现代 Linux 系统  

--------------------

**Diskmag** 这个远古东西不知道有人了解吗？

它的全称叫 disk magazine，也就是磁盘杂志，是一种在上世纪 80-90 年代，以软盘形式发行的电子杂志。90 年代后就被在线出版物所取代了。

小哥已经用 bash 脚本搞定了前端界面，就差封面、目录和 cat 每个文件的正文了。

为了运行他写的脚本，需要一个可用的 Linux 发行版，也就是一个可以在软盘上运行的系统。

动手！

因为在 64 位系统上编译 32 位代码有点棘手。为了更简单，小哥用他的 32 位 CPU 的旧笔记本来做这一切。

可以使用 32 位系统的 VirtualBox，如果要用 64 位，添加命令 “ARCH=x86”，例如：make ARCH=x86 tinyconfig。

下面就是把现代 Linux 操作系统装进一张 1.44MB 软盘的大概过程：

**1、创建并进入你想要保存文件的目录**

**2、配置和构建定制内核**

使用最新 Linux 内核（版本 5.13.0-rc2）：  

```
git clone --depth=1 https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git


```

进行最小配置：make tinyconfig

添加额外配置：make menuconfig

从菜单中选择以下选项：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCrhibCScM2EYTr9761agwCBQKvyicdo9BbspibfeT7Oj1UI7RxUA6ib4sCfCAGrCbr3YNb8zYPwOShVw/640?wx_fmt=png)

将设置保存并退出，等待编译完成，最后内核将在 arch/x86/boot/bzImage 中构建，把它移到主目录。

**3、 添加工具**

如果没有工具，内核只会启动，无法执行任何操作。小哥使用 BusyBox（最流行的轻量级工具之一），下载并解压：

```
wget https://busybox.net/downloads/busybox-1.33.1.tar.bz2

```

进入目录，进行启动配置：make allnoconfig

然后选择你想要的工具：make menuconfig

每个菜单项都显示各工具需占用多少 KB，合理选择哦。

小哥的选择：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCrhibCScM2EYTr9761agwCBXKbqK4ETJ98mWUnGKvcT4L6AJQ3Siavu1MCW8UXBmUTV06DcnITteRA/640?wx_fmt=png)

保存配置并退出，编译完成后**_install** 目录下会创建一个包含所有文件的文件系统，把它移到主目录。

**4、添加目录结构**

有了内核和基本工具，仍然需要一些额外的目录结构：

```
cd ../filesystem
mkdir -pv {dev,proc,etc/init.d,sys,tmp}
sudo mknod dev/console c 5 1
sudo mknod dev/null c 1 3

```

接下来创建几个配置文件，启动后显示欢迎消息：

```
cat >> welcome << EOF
Some welcome text...
EOF


```

然后配置处理启动、退出和重启的 **Inittab 文件** & 实际的**初始化脚本**，并使初始化脚本可执行，并将所有文件的所有者设置为 root。（限于篇幅命令已省略，具体可查看文末链接 [1]）

**5、下面就是把这一切放进软盘了**

创建指向内核和文件系统的 Syslinux 引导文件（boot file）：

```
cat >> syslinux.cfg << EOF
DEFAULT linux
LABEL linux
SAY [ BOOTING FLOPPINUX VERSION 0.1.0 ]
KERNEL bzImage
APPEND initrd=rootfs.cpio.gz
EOF

chmod +x syslinux.cfg

```

创建空软盘映像：

```
dd if=/dev/zero of=floppinux.img bs=1k count=1440
mkdosfs floppinux.img
syslinux --install floppinux.img

```

Mount it ! 并将 syslinux、内核和文件系统复制到软盘映像：

```
sudo mount -o loop floppinux.img /mnt
sudo cp bzImage /mnt
sudo cp rootfs.cpio.gz /mnt
sudo cp syslinux.cfg /mnt
sudo umount /mnt

```

完成！

现在你就有了自己的发行版映像 floppinux.img，你可以烧录到软盘，然后在真正的硬件上启动它了！

启动耗时 1 分多
---------

小哥花了不到 3 分钟烧录成功，然后开始了首次启动：

成功！大概只花了 1 分多钟。

啊，从屏幕上看了小哥似乎不年轻，头发也秃得让人落泪。

小哥（老哥）表示，在这种裸机的现代硬件上，唯一能阻止启动速度的就是软驱的实际速度。它们最大原始速度为 125KB/s。实际上可能会更慢。

下面是软盘占有空间总结，可以看到还剩 272KiB。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCrhibCScM2EYTr9761agwCBGI3SOibgUbl4tmxHB9HnJzayExlrKnY5qwvVEuicSLJgPFZ0KAR9RC1Q/640?wx_fmt=png)

网友热议：“92 年的时候我可是需要两张 5.25” 的软盘”
-------------------------------

硬件开源项目网站 Hackaday 对小哥的创造进行了报道，并点评道：

> 当然，为了将最新的 Linux 内核和 BusyBox 构建到大约 1MB 的空间，必须做出一些让步，所以 Floppinux 肯定不是任何人所说的日常驱动程序。一旦系统启动，除了编写一些 shell 脚本之外，就没有什么可做的了。
> 
> 即使你没有软盘，也值得跟着他的教程，在 QEMU 中启动映像，看看如何从零开始正式构建一个 Linux 系统。这事不仅可以用来吹牛，这样一个最小安装的所有组件如何组合在一起的知识，对学习嵌入式 Linux 设备也很有用。

而在 Hacker News 论坛上很多人纷纷对小哥竖起大拇指，有人表示最令他惊讶的就是用的最新版的 Linux 内核和 BusyBox。而且这对其他嵌入式系统也很有用。

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCrhibCScM2EYTr9761agwCBtmY2HuJlnNarkOmcEwK9hODAoUGcyf0aaqaf7dAAlk2C9PV5XU1UbQ/640?wx_fmt=png)

有人说，92 年的时候我可是需要两张 5.25 英寸的软盘来运行 Linux！

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCrhibCScM2EYTr9761agwCB3gWREHNdzANHFxkGGUicZQiaW1jhKh3ia6YkTdKuZD3Jza1ibk9CxPeIGQ/640?wx_fmt=png)

开发者介绍
-----

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtCrhibCScM2EYTr9761agwCBiaLicU3FzgqygE23A9tibfCQjWjicWFs6pN6Cm3Y4sKgfKGibSnRKfIasVg/640?wx_fmt=png)

文中的主角 “小哥” 叫 Krzysztof Jankowski，来自波兰，85 后，是一名专业的游戏开发者和数字艺术家。

25 年前就开始用 QBASIC 编程，喜欢 FOSS、像素画（pixel art）、树莓派,、游戏引擎等。

去年，他创办了自己的公司 Cyfrowy Nomada，与 beffio 签订了高级游戏引擎开发合同。他成为一名专业的游戏开发商的梦想成为现实。

他和他的伙伴们开发的游戏 “自由坦克”（Tanks of Freedom）不知道有人玩过没？

GitHub 传送门：https://github.com/w84death/floppinux

参考链接：

[1]https://bits.p1x.in/floppinux-an-embedded-linux-on-a-single-floppy/

[2]https://hackaday.com/2021/05/24/running-modern-linux-from-a-single-floppy-disk/

[3]https://news.ycombinator.com/item?id=27247612

[4]https://krzysztofjankowski.com/