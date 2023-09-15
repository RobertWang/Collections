> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/llife/p/11474491.html)

在`Linux`中，删除`rm`命令使用需谨慎，有时候可能由于误操作，导致重要文件删除了，这时不要太紧张，操作得当的话，还是可以恢复的。

EXT 类型文件恢复[#](#ext-类型文件恢复)
==========================

删除一个文件，实际上并不清除`inode`节点和`block`的数据，只是在这个文件的父目录里面的`block`中，删除这个文件的名字。`Linux`是通过`Link`的数量来控制文件删除的，只有当一个文件不存在任何`Link`的时候，这个文件才会被删除。

当然，这里所指的是彻底删除，即已经不能通过`回收站`找回的情况，比如使用`rm -rf`来删除数据。针对`Linux`下的`EXT`文件系统，可用的恢复工具有`debugfs`、`ext3grep`、`extundelete`等。 其中`extundelete`是一个开源的`Linux`数据恢复工具，支持`ext3`、`ext4`文件系统。

在数据被误删除后，第一时间要做的就是卸载被删除数据所在的分区，如果是根分区的数据遭到误删，就需要将系统进入单用户模式，并且将根分区以只读模式挂载。这样做的原因很简单，因为将文件删除后，仅仅是将文件的`inode`节点中的扇区指针清零，实际文件还存储在磁盘上，如果磁盘继续以读写模式挂载，这些已删除的文件的数据块就可能被操作系统重新分配出去，在这些数据库被新的数据覆盖后，这些数据就真的丢失了，恢复工具也回天无力。所以以只读模式挂载磁盘可以尽量降低数据库中数据被覆盖的风险，以提高恢复数据成功的比例。

Demo[#](#demo)
--------------

在编译安装`extundelete`之前需要先安装两个依赖包`e2fsprogs-libs`和`e2fsprogs-devel`，这两个包在系统安装光盘的`/Package`目录下就有，使用`rpm`或`yum`命令将其安装。`e2fsprogs-devel`安装依赖于`libcom_err-devel`包。

1. 系统使用的是`rhel6.5`，挂载光盘，安装依赖包，这里使用的是`rpm`安装方式。

```
[root@localhost ~]# mkdir /mnt/cdrom
[root@localhost ~]# mount /dev/cdrom /mnt/cdrom/
mount: block device /dev/sr0 is write-protected, mounting read-only


```

```
[root@localhost ~]# cd /mnt/cdrom/Packages/


```

```
[root@localhost Packages]# rpm -ivh e2fsprogs-libs-1.41.12-18.el6.x86_64.rpm
warning: e2fsprogs-libs-1.41.12-18.el6.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY
Preparing...                ########################################### [100%]
	package e2fsprogs-libs-1.41.12-18.el6.x86_64 is already installed


```

```
[root@localhost Packages]# rpm -ivh libcom_err-devel-1.41.12-18.el6.x86_64.rpm
warning: libcom_err-devel-1.41.12-18.el6.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY
Preparing...                ########################################### [100%]
   1:libcom_err-devel       ########################################### [100%]


```

```
[root@localhost Packages]# rpm -ivh e2fsprogs-devel-1.41.12-18.el6.x86_64.rpm
warning: e2fsprogs-devel-1.41.12-18.el6.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY
Preparing...                ########################################### [100%]
   1:e2fsprogs-devel        ########################################### [100%]


```

2. 创建本地`yum`源，安装编译环境。

```
[root@localhost ~]# yum install gcc gcc-c++ -y


```

3. 解压`extundelete`软件包。

```
[root@localhost ~]# tar jxvf extundelete-0.2.4.tar.bz2 -C ~
extundelete-0.2.4/
extundelete-0.2.4/acinclude.m4
extundelete-0.2.4/missing
extundelete-0.2.4/autogen.sh
extundelete-0.2.4/aclocal.m4
extundelete-0.2.4/configure
extundelete-0.2.4/LICENSE
extundelete-0.2.4/README
extundelete-0.2.4/install-sh
extundelete-0.2.4/config.h.in
extundelete-0.2.4/src/
extundelete-0.2.4/src/extundelete.cc
extundelete-0.2.4/src/block.h
extundelete-0.2.4/src/kernel-jbd.h
extundelete-0.2.4/src/insertionops.cc
extundelete-0.2.4/src/block.c
extundelete-0.2.4/src/cli.cc
extundelete-0.2.4/src/extundelete-priv.h
extundelete-0.2.4/src/extundelete.h
extundelete-0.2.4/src/jfs_compat.h
extundelete-0.2.4/src/Makefile.in
extundelete-0.2.4/src/Makefile.am
extundelete-0.2.4/configure.ac
extundelete-0.2.4/depcomp
extundelete-0.2.4/Makefile.in
extundelete-0.2.4/Makefile.am


```

4. 配置、编译、安装`extundelete`软件包

```
[root@localhost ~]# cd extundelete-0.2.4
[root@localhost extundelete-0.2.4]# ls
acinclude.m4  aclocal.m4  autogen.sh  config.h.in  configure  configure.ac  depcomp  install-sh  LICENSE  Makefile.am  Makefile.in  missing  README  src
[root@localhost extundelete-0.2.4]# ./configure
Configuring extundelete 0.2.4
Writing generated files to disk
[root@localhost extundelete-0.2.4]# make
make -s all-recursive
Making all in src
extundelete.cc:571: 警告：未使用的参数‘flags’
[root@localhost extundelete-0.2.4]# make install
Making install in src
  /usr/bin/install -c extundelete '/usr/local/bin'


```

5. 准备好用于测试的分区，`/dev/sdb1`为`ext4`格式，挂载到`/mnt/ext4`目录下。

```
[root@localhost ~]# mkdir /mnt/ext4
[root@localhost ~]# mount /dev/sdb1 /mnt/ext4/
[root@localhost ~]# df -hT /mnt/ext4/
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sdb1      ext4   20G  172M   19G   1% /mnt/ext4


```

6. 创建测试文件。

```
[root@localhost ~]# cd /mnt/ext4/
[root@localhost ext4]# echo 1 > a
[root@localhost ext4]# echo 2 > b
[root@localhost ext4]# echo 3 > c
[root@localhost ext4]# ls
a  b  c  lost+found


```

7. 删除测试文件。

```
[root@localhost ext4]# rm -f a b
[root@localhost ext4]# ls
c  lost+found


```

8. 卸载对应的分区。

```
[root@localhost ext4]# cd
[root@localhost ~]# umount /mnt/ext4/


```

9. 恢复删除的内容。

```
[root@localhost ~]# extundelete /dev/sdb1 --restore-all
NOTICE: Extended attributes are not restored.
Loading filesystem metadata ... 160 groups loaded.
Loading journal descriptors ... 24 descriptors loaded.
Searching for recoverable inodes in directory / ...
2 recoverable inodes found.
Looking through the directory structure for deleted files ...
0 recoverable inodes still lost.


```

10. 恢复的文件会在在当前目录下的`RECOVERED_FILES`文件夹内。

```
[root@localhost ~]# ls RECOVERED_FILES/
a  b


```

XFS 类型文件备份和恢复[#](#xfs-类型文件备份和恢复)
================================

`extundelete`工具仅可以恢复`EXT`类型的文件，无法恢复`CentOS 7`系统默认采用`xfs`类型的文件。针对`xfs`文件系统目前也没有比较成熟的文件恢复工具，所以建议提前做好数据备份，以避免数据丢失。

`xfs`类型的文件可使用`xfsdump`与`xfsrestore`工具进行备份恢复。若系统中未安装`xfsdump`与`xfsrestore`工具，可以通过`yum install -y xfsdump`命令安装。`xfsdump`按照`inode`顺序备份一个`xfs`文件系统。

`xfsdump`的备份级别有两种：`0`表示完全备份；`1-9`表示增量备份。默认为`0`。

```
xfsdump -f 备份存放位置 要备份路径或设备文件


```

`-f`：指定备份文件目录  
`-L`：指定标签`session label`  
`-M`：指定设备标签`media label`  
`-s`：备份单个文件，`-s`后面不能直接跟路径。

*   使用`xfsdump`时，需要注意以下的几个限制：

1.`xfsdump`不支持没有挂载的文件系统备份，所以只能备份已挂载的；  
2.`xfsdump`必须使用`root`的权限才能操作 (涉及文件系统的关系)；  
3.`xfsdump`只能备份`XFS`文件系统；  
4.`xfsdump`备份下来的数据 (档案或储存媒体) 只能让`xfsrestore`解析；  
5.`xfsdump`是透过文件系统的`UUID`来分辨各个备份档的，因此不能备份两个具有相同`UUID`的文件系统。

```
xfsrestore -f 恢复文件的位置 存放恢复后文件的路径


```

Demo[#](#demo-1)
----------------

1. 准备好用于测试的分区，`/dev/sdb1`为`ext4`格式，挂载到`/mnt/ext4`目录下。

```
[root@localhost ~]# mkdir /mnt/xfs
[root@localhost ~]# mount /dev/sdb1 /mnt/xfs/
[root@localhost ~]# df -hT /mnt/xfs/
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sdb1      xfs    20G   33M   20G   1% /mnt/xfs


```

2. 创建测试文件。

```
[root@localhost ~]# cd /mnt/xfs/
[root@localhost xfs]# mkdir test
[root@localhost xfs]# touch a.txt
[root@localhost xfs]# touch test/b.txt


```

3. 可以使用`tree`查看目录结构。

```
[root@localhost ~]# yum install tree -y
[root@localhost ~]# tree /mnt/xfs/
/mnt/xfs/
├── a.txt
└── test
    └── b.txt

1 directory, 2 files


```

4. 使用`xfsdump`命令备份整个分区。

```
[root@localhost ~]# xfsdump -f /opt/dump_sdb1 /dev/sdb1
xfsdump: using file dump (drive_simple) strategy
xfsdump: version 3.1.4 (dump format 3.0) - type ^C for status and control

 ============================= dump label dialog ==============================

please enter label for this dump session (timeout in 300 sec)
 -> dump_sdb1      //指定备份会话标签
session label entered: "dump_sdb1"

 --------------------------------- end dialog ---------------------------------

xfsdump: level 0 dump of localhost.localdomain:/mnt/xfs
xfsdump: dump date: Fri Sep  6 13:36:12 2019
xfsdump: session id: 74232f85-124c-4486-8d91-f35208534f74
xfsdump: session label: "dump_sdb1"
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 21760 bytes
xfsdump: /var/lib/xfsdump/inventory created

 ============================= media label dialog =============================

please enter label for media in drive 0 (timeout in 300 sec)
 -> sdb1      //指定设备标签，就是对要备份的设备做一个描述
media label entered: "sdb1"

 --------------------------------- end dialog ---------------------------------

xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsdump: dumping non-directory files
xfsdump: ending media file
xfsdump: media file size 22952 bytes
xfsdump: dump size (non-dir files) : 0 bytes
xfsdump: dump complete: 46 seconds elapsed
xfsdump: Dump Summary:
xfsdump:   stream 0 /opt/dump_sdb1 OK (success)
xfsdump: Dump Status: SUCCESS


```

5. 查看备份信息与内容。

```
[root@localhost ~]# xfsdump -I
file system 0:
        fs id:          f8805a3e-089e-4875-ad54-d31e5dc98835
        session 0:
                mount point:    localhost.localdomain:/mnt/xfs
                device:         localhost.localdomain:/dev/sdb1
                time:           Fri Sep  6 13:36:12 2019
                session label:  "dump_sdb1"
                session id:     74232f85-124c-4486-8d91-f35208534f74
                level:          0
                resumed:        NO
                subtree:        NO
                streams:        1
                stream 0:
                        pathname:       /opt/dump_sdb1
                        start:          ino 68 offset 0
                        end:            ino 70 offset 0
                        interrupted:    NO
                        media files:    1
                        media file 0:
                                mfile index:    0
                                mfile type:     data
                                mfile size:     22952
                                mfile start:    ino 68 offset 0
                                mfile end:      ino 70 offset 0
                                media label:    "sdb1"
                                media id:       cc32446f-42e8-489b-867f-84a55949c1fa
xfsdump: Dump Status: SUCCESS


```

6. 删除创建的测试文件，模拟数据丢失。

```
[root@localhost ~]# rm -rf /mnt/xfs/*
[root@localhost ~]# tree /mnt/xfs/
/mnt/xfs/

0 directories, 0 files


```

7. 恢复文件丢失的文件。

```
[root@localhost ~]# xfsrestore -f /opt/dump_sdb1 /mnt/xfs/
xfsrestore: using file dump (drive_simple) strategy
xfsrestore: version 3.1.4 (dump format 3.0) - type ^C for status and control
xfsrestore: searching media for dump
xfsrestore: examining media file 0
xfsrestore: dump description:
xfsrestore: hostname: localhost.localdomain
xfsrestore: mount point: /mnt/xfs
xfsrestore: volume: /dev/sdb1
xfsrestore: session time: Fri Sep  6 13:36:12 2019
xfsrestore: level: 0
xfsrestore: session label: "dump_sdb1"
xfsrestore: media label: "sdb1"
xfsrestore: file system id: f8805a3e-089e-4875-ad54-d31e5dc98835
xfsrestore: session id: 74232f85-124c-4486-8d91-f35208534f74
xfsrestore: media id: cc32446f-42e8-489b-867f-84a55949c1fa
xfsrestore: using online session inventory
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsrestore: 2 directories and 3 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsrestore: restore complete: 0 seconds elapsed
xfsrestore: Restore Summary:
xfsrestore:   stream 0 /opt/dump_sdb1 OK (success)
xfsrestore: Restore Status: SUCCESS
[root@localhost ~]# tree /mnt/xfs/
/mnt/xfs/
├── a.txt
└── test
    └── b.txt

1 directory, 2 files


```