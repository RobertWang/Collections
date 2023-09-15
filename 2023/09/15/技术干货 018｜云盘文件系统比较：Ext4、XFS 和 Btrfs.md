> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/342837972)

0x00 背景
-------

在上一篇[云硬盘性能分析](https://zhuanlan.zhihu.com/p/336910081)的教程中，为大家介绍了如何评测云硬盘的读写性能。但是，我们使用硬盘，从来不是直接读写裸设备，而是通过文件系统来管理和访问硬盘上地文件。不少朋友询问，文件系统该如何对比，又该如何选择呢？

文件系统的选择，其重要性不言而喻，可能仅次于 Linux 发行版的选择。其实，各个文件系统在功能及性能方面是有不小的差异的。本文中，我们将一起探索 Linux 中主流的三个文件系统——Ext4、XFS 以及 Btrfs——的功能特点，并基于腾讯云高性能云硬盘，做一个初步的性能对比。

0x01 文件系统
---------

首先，我们简单感受下什么是文件系统。文件系统（File System 或 fs），定义并实现了数据在存储介质（如硬盘等）上的存储方式和结构，以及其是如何被访问的，如索引、读取等。操作系统里，硬盘中的数据被抽象为文件的形式，并对其进行管理，比如为一块或多块数据关联一个文件名等，因此，我们称这些用于管理文件的数据结构（以及其对应的规则）为文件系统，就再自然不过了。Unix 中更是讲一切皆文件，可见文件系统的地位非常之高。

当然，随着技术的发展，现代的文件系统特性也越来越多，如 Btrfs 等文件系统，支持快照、子卷、校验和自检、软 RAID 甚至透明压缩等，这些虽不是传统文件系统所做的事，但却是一种不可阻挡的趋势。

文件系统通常作为操作系统的一部分，如 Linux 中是作为子系统而实现在内核里的。我们接下来要介绍的 Ext4、XFS、Btrfs 都是实现在 Kernel 代码中的 fs 目录下。文件系统需要实现操作系统所定义的对象和接口，如 inode、dentry 等。

如上图所示，super block 用于描述挂载文件系统的元信息，即文件系统控制块；inode 对象就是存储一个文件的通用信息，如时间戳、所对应的数据块位置等；dentry 对象存储一个文件的目录项的链接信息，也就是文件的路径名，因为一个文件可能有多个链接。以上三者都是由文件系统各自实现并存储在硬盘上的。那么，什么时候存上的呢？对于一些文件系统如 Ext4 等，在硬盘格式化时就全部确定了，而对于 XFS 则是动态生成的，BtrfS 则是更特别的动态实现。但无论如何，各个文件系统都需要存储这三类信息，因为这是内核规定的（见下）。另外，我们常说的 file 对象，它用于关联进程和 dentry 对象的，可以简单理解成正在打开的文件，它是存在内存里的，千万不要搞混，file 对象并不存在 file system 里。

Linux 中的 VFS（Virtural File System 或 Virtual FS Switch）作为具体的文件系统的抽象层，是非常重要的。它为各类文件系统提供了一个一致的接口，如必须支持哪些 POSIX 兼容的系统调用等，用户态的应用无需关注底层具体文件系统的区别，通过相同的系统调用请求内核即可。还是得益于 VFS，我们甚至可以通过 FUSE 等机制来实现用户态的文件系统，比如 GlusterFS、IPFS 等。

![](https://pic1.zhimg.com/v2-64d44c3a9b6d9b584cdfecac31f7f19c_r.jpg)

0x02 各文件系统简介
------------

本章简单介绍 Linux 三类文件系统：**Ext4、XFS、Btrfs**。

**Ext4**

Ext4（ext4 日志文件系统、第四代扩展文件系统）文件系统是 Linux 用途最广泛的日志文件系统。Ext4 稳定版本发布于 2008 年，即 Linux 2.6.28 版本。但它的历史最早可以追溯到 1992 年的 Ext2 文件系统，那是 Linux 最早使用的文件系统，而后 2001 年出现的 Ext3 在 Ext2 基础上增加了日志功能，并最终由 Ext4 替代，而且 Ext4 可向前兼容 Ext2/Ext3。Ext4 是很多发行版如 Debian、Ubuntu 等的默认文件系统。

什么是日志（Journaling）文件系统？就是在数据更改正式提交至硬盘之前，先在日志区域（也是存在文件系统上的）记录变更日志，这样可以在系统崩溃或掉电后能快速恢复。现代的文件系统，甚至各类可靠的存储系统，日志存储都是必须的。

Ext4 在 Ext3 的基础上，又增加了不少新特性。第一，大文件支持，最大卷 **1EiB**，最大文件 **16TiB**（对于 4KiB 块）。Ext4 中实现了基于 extent 的数据管理。extents 可以简单理解为连续的块，有了它管理大文件就更方便，大大降低了用于索引大文件的元数据量——原来用间接块索引（Indirect Block）的方法是非常浪费的——自然访问也性能也提升。而且，Ext4 的 inode 比 Ext3 大一倍，达到 256 字节。不过话说回来，2008 年那时所谓的 “大” 文件，到今天看来也就一般，如今谁还没有个几百兆甚至上 GiB 的视频呢。第二，增加并优化了日志校验和（journal check summing）功能，就是 fsck 过程更快了。这两个特性也是必须要有的，其实 XFS 和 Btrfs 也都支持。另外还有些特性如无日志模式、多块分配、延迟分配、在线去除碎片（defragmentation）等。

![](https://pic2.zhimg.com/v2-f0677ab40e3ac798bb8af567bf5fc2b1_r.jpg)

上图是 Ext 系列文件系统的数据存储 schema，硬盘分区被等分成 N 个同样大小的块组，方便索引。每个快组又分为 super block、group descriptors、data block bitmap、inode table，以及最后数据块区域。bitmap 块表示对应的块 / inode 是否被分配。全部文件系统的元信息可以通过`dumpe2fs`工具查看，注意需要 umount 后才是最新的。

![](https://pic4.zhimg.com/v2-184d98b9f33e7350fd9378c8dde3020b_r.jpg)

可以看出，数据块总量受制于 Bitmap 块的大小，所以每个块组最多也就 4096*8=**32768 个数据块**。另外 inode 数一般是数据块的**四分之一**，其数量也是在格式化时固定的，理论上 inode 耗尽时，即便还有空间，也是无法再创建新文件的。查看 inode 占用量可以通过`df -ih`来查看。

_为什么是 4KiB 的块大小（Block Size），这个数字怎么总出现？块是文件系统操作设备的最小单元。硬盘寻址的最小单位是 512B，文件系统操作的块大小需要是它的整数倍且是 2 的幂次，而且不能超过内存页帧大小（通常是 4096B），可选的就是 512、1024、2048 和 4096，为了提升吞吐率，所以基本就都是 4KiB 的块。_

**XFS**

XFS 是 1993 年由 SiliconGraphic Inc(SGI) 公司创建的高性能的 64 位文件系统。2001 年进入 Linux 内核，如今已被大多数 Linux 发行版支持。最支持的要数红帽公司，其下主打的操作系统 RHEL（Red Hat Enterprise Linux）7 和 8——即最近两个稳定版——都选 XFS 为默认的文件系统，红帽的很多工程师也深度参与了 XFS 的开发和维护。

![](https://pic3.zhimg.com/v2-2c045b8145b28d25f37534682c507dd6_r.jpg)

XFS 的文件系统结构如上，不同于 Ext4，它通过 B + 树来索引 inode 和数据块。用树结构的文件系统通常相比 Ext4 用表结构，如链表、直接 / 间接 Block 以及 extent，能更好地支持大文件，如视频 / 数据库文件等。另外其元数据规模少，使得硬盘可用空间更多，实测 XFS、Btrfs 多平均至少 1.5% 以上的可用空间。

XFS 能支持多大文件呢？单卷可达 **8EiB**，最大文件也到 **8EiB**，相比 Ext4 的 16TiB 可高了几个数量级。而且是其实动态分配 inode 的实现机制，只要有空间，就不会耗尽 inode。通过 df 命令看出，其 inode 初始值就是 ext4 的 10 倍左右。

![](https://pic2.zhimg.com/v2-a228aadf25ced8a397bb7002072a46c5_r.jpg)

另外，XFS 可以更高效支持并行 IO 操作，RAID 上的扩展性更好，多线程并行读写时相比 Ext4 有优势。

**Btrfs**

Btrfs，是 B-Tree File System 的缩写，可以读作 “butter fs” 或“b-tree fs”，是一个支持 copy-on-write (COW)的文件系统，由 Oracle 公司于 2007 年设计并使用，2013 年进入 Linux 内核稳定发布。 目前除了 Oracle 外，SUSE Linux Enterprise Server 将其用作默认文件系统，而在工作站领域中，Fedora 33 也将其作为默认文件系统，Facebook 公司也大量应用了 Btrfs 文件系统。

Btrfs 的强大之处，在于实现了很多先进特性的同时，还保持了很高的容错能力、可扩展性以及可靠性。其最早的 COW B-tree 的数据结构也是 2007 年才提出的，Btrfs 比 Ext4、XFS 小了近 20 年，的确是个现代文件系统，那么它不一样在哪里，有哪些面向未来的特性呢？

![](https://pic4.zhimg.com/v2-8b0aa1c420d02086363924d8c16ae083_r.jpg)

那么，什么是 COW 的文件系统？你或许听说过创建子进程时的内存分页中的 COW 机制：进程创建时并不申请真正内存，而是在写内存是通过触发 page fault 动态申请，on-demand 分配提升了资源利用率和系统响应性能。COW 的文件系统也是用了类似的机制，这种机制比日志，能更高效地保证了数据的一致性（consistency）和完整性（Integrity），同时提升了系统自修复、快照功能的性能。

![](https://pic1.zhimg.com/v2-21a0aacdda0db3b1c2d0f4c61b24e200_r.jpg)

特性上，Btrfs 支持得更多：大文件支持是必须的，**16EiB** 最大卷和文件大小；集成了卷管理功能，可以以卷的方式动态地增减设备，实现硬盘资源池化；高效的数据完整性 check，如基于 cow 的自恢复（self-healing）、基于 checksum 的数据清理（data scrubbing）；高性能的读写 / 只读快照，得益于 cow，增量快照和备份非常直接、灵活且低成本；软 raid 支持，数据和元数据的 stripe 和 mirror 生产环境级别支持；透明的压缩，支持 lzo 和 zlib；在线去碎片（online defragmentation）；数据去重（data deduplication）。

这方面，与其说是更好的文件系统，倒不如说它把传统的逻辑卷管理（如 LVM）、软 RAID（如 mdadm）等工具做的工作集成在了文件系统层面上，方便了系统管理员，确实是一大进步。

0x03 文件系统的使用
------------

**格式化**

对于分区格式化，通过 mkfs 命令即可，不过对于某些发行版，需要安装对应的文件系统 utility 工具。而这 3 个文件系统，现在的主流内核早已全部稳定支持。

```
# 格式化成 Ext4
mkfs.ext4 /dev/vdb1
​
# 格式化为 XFS
apt install xfsprogs
mkfs.xfs /dev/vdb2
​
# 格式化为 Btrfs
apt install btrfs-progs
mkfs.btrfs /dev/vdb3

```

**用量查看**

对于 Ext4 和 XFS 用 df 命令即可查看数据和 inode 的用量

```
df -h   # 查看硬盘用量
df -ih  # 查看硬盘inode用量

```

Btrfs 则相对特殊一些，因为它是 COW 的文件系统，最好不要用`df`查看容量，因为不准；我们也无法用`df -i`查看 inode 数量的，会显示 0，因为 Btrfs 根本没有 inode 预设限制。通过`btrfs`命令来查看用量。

```
btrfs filesystem usage /your/btrfs/directory

```

命令执行如下图：

![](https://pic4.zhimg.com/v2-787b27d1cb324e1520f9432432a6c9a3_r.jpg)

当然，Btrfs 可玩的东西比较多，如子卷、快照、raid、数据恢复等，这些特性后续值得专门再单独分析介绍。

不过在云硬盘场景下，以上各中工作其实都已经完全托管实现了，[腾讯云的 CBS 云硬盘产品](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/product/cbs%3Ffrom%3D10680)，通过多副本的机制保障了数据的可靠性、可用性等问题，同时并发性能也得到特别的优化。

0x04 云盘上各文件系统性能对比
-----------------

实验环境选择 x86_64 体系下的 Debian 10 (Buster) 稳定版的 OS，每种文件系统均为 1T 的高性能云硬盘。

**格式化**

Ext4: 7.137s, XFS: 5.871s, Btrfs: 1.428s （第二次即以后仅要 0.046s） **Btrfs > XFS > Ext4**

**顺序读**

```
fio -name=write-throughput -readwrite=read -blocksize=128k -numjobs=2 -ioengine=libaio -iodepth=1 -direct=1 -runtime=60 -refill_buffers -norandommap -randrepeat=0 -group_reporting --size=10G -filename=/data/ext4/a

```

Ext4: IOPS=3039, BW=380MiB/s

XFS: IOPS=3145, BW=393MiB/s

Btrfs: IOPS=3132, BW=392MiB/s

**读吞吐量：XFS、Btrfs 差不多，相比 Ext4 稍有优势。**

**顺序写**

```
fio -name=write-throughput -readwrite=write -blocksize=128k -numjobs=2 -ioengine=libaio -iodepth=1 -direct=1 -runtime=60 -refill_buffers 
-norandommap -randrepeat=0 -group_reporting --size=10G -filename=/data/xfs/a

```

Ext4: IOPS=1056, BW=132MiB/s

Btrfs: IOPS=1007, BW=126MiB/s

**写吞吐量：线程少的时候，总体差不多，Ext4 略快一些。**

**4K 随机读**

```
fio -name=read-latency-iops -readwrite=randread -blocksize=4k -numjobs=2 -ioengine=libaio -iodepth=1 -direct=1 -runtime=60 -refill_buffers -norandommap -randrepeat=0 -group_reporting --size=10G -filename=/data/btrfs/a

```

Ext4: IOPS=7254, BW=28.3MiB/s

XFS: IOPS=8202, BW=32.0MiB/s

Btrfs: IOPS=7923, BW=30.0MiB/s

**4K 随机读，XFS > Btrfs > Ext4。**

性能测试总结，在腾讯云的 CBS 云硬盘上，对于以上三种成熟稳定的文件系统，不论选用其中哪一种，性能都比较优异，且差异相对不大。

0x05 小结
-------

至此，我们一起熟悉了文件系统的概念、以及常见主流的三大文件系统（Ext4、XFS、Btrfs）的各自特点及使用方法、场景以及各自的比较，并实测了其在云服务器中的性能。那么综合以上的分析，你的业务产经更适合倾向于哪种文件系统呢？欢迎关注机构号，后续我们将持续地讲解存储相关选型的具体实践~

0x06 参考资料
---------

*   [腾讯云硬盘](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/product/cbs%3Ffrom%3D10680)
*   [https://en.wikipedia.org/wiki/Filesystem_in_Userspace](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Filesystem_in_Userspace)
*   [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/managing_file_systems/index](https://link.zhihu.com/?target=https%3A//access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/managing_file_systems/index)
*   [XFS manual](https://link.zhihu.com/?target=https%3A//man7.org/linux/man-pages/man5/xfs.5.html)

更多腾讯云服务器技术干货、优惠活动、交流社区，敬请关注「腾讯云服务器」微信公众号（TencentCVM）