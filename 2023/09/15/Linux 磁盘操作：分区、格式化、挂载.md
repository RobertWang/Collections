> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/166374034)

**目录**
------

*   [1. 磁盘分区](https://zhuanlan.zhihu.com/write)

*   [1.1 查看磁盘分区](https://zhuanlan.zhihu.com/write)
*   [1.2 使用 gdisk 给磁盘分区](https://zhuanlan.zhihu.com/write)

*   [2. 格式化](https://zhuanlan.zhihu.com/write)
*   [3. 挂载文件系统](https://zhuanlan.zhihu.com/write)

**1. 磁盘分区**
-----------

### **1.1 查看磁盘分区**

首先可以使用 lsblk（list block device）列出所有设备信息，如下图。在图中可以看到共有一个磁盘，即 sda，磁盘上有三个分区：sda1、sda2、sda3。在 sda3 中有三个文件系统都是 xfs 格式，挂载点分别为 /、/swap、/home。

MAJ:MIN 表示设备的一种编号，内核就是通过这个编号来识别设备；

RM 表示是否为可卸载（removable）设备；

RO 表示是否为只读（read only）设备；

TYPE 有几种选择：disk、part、lvm、rom。如果没有卸载安装系统的 iso 镜像文件就会出现一个只读设备，即安装盘。

使用 blkid（block id）可以查看设备的全局唯一标识符（UUID，universally unique identifier），如下图：

![](https://pic2.zhimg.com/v2-07a99131c08bb976545b6d3b23928e7d_r.jpg)

使用 parted 指令可以列出磁盘的分区信息，如下图。在图中可以看到有三个分区，实际上这三个分区为 gpt（引导扇区）、/boot、/ 和 / home 和 / swap 的数据分区。最后一个分区为 lvm 实际上里面有三个文件系统均为 xfs，挂载点为 / 和 / home 和 / swap。

![](https://pic3.zhimg.com/v2-dc331440320e554154a6af718bf1e502_r.jpg)

### **1.2 使用 gdisk 给磁盘分区**

gdisk 和 fdisk 都可以给磁盘分区，gdisk 适用于 gpt 作为引导扇区的情况，fdisk 适用与 MBR 作为引导扇区的情况。两者的使用方法基本一样，下面介绍 gdisk 的使用方法。

gdisk 基本上不用记命令，直接在终端输入 gdisk 然后根据提示就可以完成操作，如下图。注意要操作的设备就是 sda，后面没有任何数字。

![](https://pic1.zhimg.com/v2-f61a29c1bb10ab313192434af40bdedc_r.jpg)

输入? 之后可以看到 gdisk 支持的各种命令。使用 p 显示磁盘分区的信息，输出的信息和 lsblk 有点像（简略一些）其中 Code 表示在分区内可能存在的文件类型，如 Linux 为 8300，swap 为 8200。

![](https://pic3.zhimg.com/v2-ccabb264fd9c8fe76b5b9baf5b7ae3fa_r.jpg)

下面建立三个新的分区。已经在上面看到，整个磁盘最大的扇区号为 83886046，目前只使用到了 65026047。因此可以建立：

*   1G xfs 文件系统（Linux）
*   1G fat 文件系统（Windows）
*   0.5G swap

具体操作过程如下图。

Partition number 表示该分区的编号，因为是第四个分区，所以默认值为 4；

First sector 就是新分区的第一个扇区，默认就是顺序第一个空扇区；

Last sector 是新分区的最后一个扇区，默认是整个磁盘最后一个扇区，也就是分配完全部磁盘空间。这里只需要 1G，因此写 + 1G，这种偏移量的表达方式也是可以的；

最后要求选择文件系统类型，选择默认的 linux 就行。其实这里的文件系统类型不一定是真正的文件系统类型，之后格式化的时候还可以更改文件系统类型。

![](https://pic2.zhimg.com/v2-484ecad2e6166f4ae1017083cc940715_r.jpg)

分配完扇区之后，再次使用 p 显示当前的分区情况，如下图。

![](https://pic3.zhimg.com/v2-90ddb667d056ab47d84ca7ecf714c142_r.jpg)

至此就成功完成了分区的操作。要注意的是创建分区只是修改了 gpt 的内容，对磁盘上相应扇区的内容没有任何改变，之后还要给这一段磁盘空间写入文件系统，即格式化。最后将格式化的磁盘挂载到一个目录下面才能正常使用这一段存储空间。

之后再创建上文提到的其他分区，然后退出 gdisk（使用 w 退出程序）。使用 partprobe 指令即可更新系统的分区表（分区表 / proc/partitions 只是一个文本文件，会记录 gpt 的信息，在 / proc/partitions 中没有显示分区信息只是该文件还没有更新）

具体操作过程如下图：

![](https://pic4.zhimg.com/v2-596986c6cb2b1dc761dc7eb74bf0f363_r.jpg)

**2. 格式化**
----------

格式化本质上是创建文件系统，当然在创建新的文件系统的时候也会顺便把之前的文件删除掉，因此一般人就认为格式化意味着删除所有文件。

以下演示如何将刚才分配好的 4 号分区格式化为 xfs 文件系统。

格式化使用 mkfs（make file system）指令。当要格式化为 xfs 文件系统的时候就是用 mkfs.xfs，这里直接使用默认参数进行格式化，如下图。实际上和 xfs_info 显示的信息基本上一样。

![](https://pic4.zhimg.com/v2-50de7e92522c00159daffed7dced1f5b_r.jpg)

查阅其他 mkfs 可以进行格式化的文件系统如下：

```
$ mkfs[tab][tab]
mkfs         mkfs.cramfs  mkfs.ext3    mkfs.fat     mkfs.msdos   mkfs.vfat
mkfs.bfs     mkfs.ext2    mkfs.ext4    mkfs.minix   mkfs.ntfs    mkfs.xfs


```

**3. 挂载文件系统**
-------------

挂载文件系统就是将一个空的目录映射到某一个磁盘的分区上面。

在图形界面下，可以使用无脑的挂载方式，直接双击那个磁盘就行当于挂载了它，应该是被挂载到了 media 下面。就是双击下图中想挂载的磁。（原来使用双系统的时候经常从 Linux 系统直接这样读取 Windows 的文件，后来才明白这个操作叫挂载，甚至还有启动时挂载这种高级的操作）

![](https://pic2.zhimg.com/v2-70b2b49ca6f6c25d77d77879c23836d1_r.jpg)

当然，为了能挂载到指定的目录下，就必须使用 mount 这些比较麻烦的挂载方式。mount 基本使用非常直观，下图显示了如何将已经格式化的 xfs 文件系统挂载到 / data/xfs 上。首先要使用 blkid 查询 / dev/sda4 的 UUID（其实直接使用 / dev/sda4 也可以成功挂载）。最后用 df 指令查看 / data/xfs 是否成功挂载。

![](https://pic3.zhimg.com/v2-720d9ba4fc7b655ffe39f50cb4515092_r.jpg)

之后还有对特殊设备的挂载、检查文家系统一致性等比较高难度的操作，但是总的来说分区、格式化、挂载是基础，会了这些再学其他内容会很快。