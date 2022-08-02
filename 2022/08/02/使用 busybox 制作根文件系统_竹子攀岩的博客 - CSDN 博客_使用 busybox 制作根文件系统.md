> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/lzkzls/article/details/96423154)

1、Linux 內核啓動之後，第一個掛載的文件系統，称为根文件系统。根文件系统由基本的 she'll 命令、各种库、[字符设备](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E8%AE%BE%E5%A4%87&spm=1001.2101.3001.7020)、配置脚本组成。它提供了根目录 /。RFS（root file system）可以放在 nor、nand flash、SD 卡、磁盘、网络空间上。

2、在 Linux 文件系统中，计算机对应的就是根文件系统。bin 里面存放了各种命令文件，lib 里面存放了各种库，dev 目录存放了各种设备。etc 文件下面由配置文件和配置脚本等。

嵌入式文件系统相对简单。busybox 工具来制作。

3、安装过程：

上一篇启动 Linux 内核成功后，提示根文件系统挂载失败。本篇制作根文件系统。

获得编译源文件 wget https://busybox.net/downloads/busybox-1.27.2.tar.bz2  
解压缩 tar xjvf busybox-1.27.2.tar.bz2  
进入 busybox 解压后的目录 cd busybox-1.27.2

依次执行以下命令：

expert ARCH=arm

expert CROSS_COMPILE=arm-linux-gnueabi-

make defconfig #配置开发板  
make CROSS_COMPILE=arm-linux-gnueabi- #编译 busybox  
make install CROSS_COMPILE=arm-linux-gnueabi- #编译 busybox

在_isntall 文件夹下面查看是否生成相应的需要的文件  
![](https://img-blog.csdnimg.cn/20190718135716843.png)

_注：cp 命令的相关说明_

_-p ：连同档案的属性一起复制过去，而非使用预设属性；  
-r ：递归持续复制，用于目录的复制行为；_

_在 / home/lzk / 下创建 rootfs 文件夹：_

sudo mkdir rootfs

将上述_install 文件夹下生成的命令文件等拷贝到 rootfs 文件夹下。

sudo cp -r busybox-1.2.7.1/_isntall/* rootfs/

从工具链中复制运行库到 lib 目录下（首先创建 lib 目录）

sudo cp -p /usr/arm-linux-gnueabi/lib/* /rootfs/lib

创建 4 个 tty 终端设备（c 代表字符设备，4 是主设备号，1~4 分别是次设备号）

sudo mkdir -p rootfs/dev  
sudo mknod rootfs/dev/tty1 c 4 1  
sudo mknod rootfs/dev/tty2 c 4 2  
sudo mknod rootfs/dev/tty3 c 4 3  
sudo mknod rootfs/dev/tty4 c 4 4

创建终端和回收站

sudo mknod -m 666 console c 5 1

sudo mknod -m 666 null 1 3

使用 dd 命令制作文件系统镜像，然后格式化生成的 ext3 文件系统，【注：生成系统镜像时，一定不要在 rootfs 文件夹下面，否则后面 cp 时会报错】，接着将文件系统挂载到 tmpfs 文件夹下，最后 rootfs 下的各种文件拷贝到刚刚挂载了文件镜像系统的 tmpfs 文件夹中。

注：mount 命令 用于挂载 Linux 系统外的文件

![](https://img-blog.csdnimg.cn/20190718151830419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x6a3pscw==,size_16,color_FFFFFF,t_70)

最后，检查ｑｅｍｕ模拟系统是否启动成功及挂载成功：

在 / home/lzk / 目录下执行一下命令：

qemu-system-arm -M vexpress-a9 -m 512 -kernel /home/lzk/linux_kernel_iso/zImage -dtb /home/lzk/linux_kernel_iso/vexpress-v2p-ca9.dtb  -nographic -append "root=/dev/mmcblk0 rw console=ttyAMA0" -sd a9rootfs.ext3

![](https://img-blog.csdnimg.cn/20190718163652881.png)

第一次运行时报错，找不到 / etc/init.d/rcS

直接在ｃｏｎｓｏｌｅ下，创建该目录和文件，并在 rcS 下输入：

echo "---------------------------------"

echo ''welcome to A9 vexpress"

echo ''---------------------------------"

然后，重新运行截图中的命令。可以看到，运行成功，根文件系统挂载成功。

![](https://img-blog.csdnimg.cn/20190718164157109.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x6a3pscw==,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20190718164223790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x6a3pscw==,size_16,color_FFFFFF,t_70)