> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/u_14449524/2433036)

> Linux Centos7 xfs 文件误删了怎么办——快速恢复 xfs 文件教学，一. xfs 文件恢复 xfs 类型的文件可使用 xfsdump 与 xfsrestore 工具进行备份恢复。

© 著作权归作者所有：来自 51CTO 博客作者 23trl 的原创作品，请联系作者获取转载授权，否则将追究法律责任

**一. xfs 文件恢复**
---------------

> **xfs 类型的文件可使用 xfsdump 与 xfsrestore 工具进行备份恢复。若系统中未安装 xfsdump 与 xfsrestore 工具，可以通过 yum install -y xfsdump 命令安装。xfsdump 按照 inode 顺序备份一个 xfs 文件系统。xfsdump 的备份级别有两种：0 表示完全备份；1-9 表示 增量备份。xfsdump 的备份级别默认为 0。xfsdump 的命令格式为：xfsdump -f 备份存放 位置 要备份路径或设备文件。**

![](https://s1.51cto.com/images/blog/201908/27/0a59db6eac23759eb2df5f995e5d660c.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)![](https://s1.51cto.com/images/blog/201908/27/88fc893540ea9729affd4dfcd2c36298.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 我们来实验一下 xfs 文件恢复

#### 1. 我们先添加一块磁盘并且让其有效

![](https://s1.51cto.com/images/blog/201908/27/d2341fa6d81ca7d3492616f3908d9541.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)![](https://s1.51cto.com/images/blog/201908/27/7f8a88274ad8cbd6f2bda7f7b2edaa08.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 2. 创建主分区

![](https://s1.51cto.com/images/blog/201908/27/45c66f6886efca47c94acb9fce804dbd.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 3. 格式化，创建挂载点，挂载

![](https://s1.51cto.com/images/blog/201908/27/2f1d136e89cf219166364dc611f5abeb.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=) ![](https://s1.51cto.com/images/blog/201908/27/6d9a7ca9ecb6b0cad144c69c3fc706f0.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 4. 在 data 挂载点下面创建文件和目录

![](https://s1.51cto.com/images/blog/201908/27/a9c6520ea96bc64eb3ea21e3fbe5ccd0.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 5. 树状结构去查看文件

![](https://s1.51cto.com/images/blog/201908/27/ecfb82282969255437a4745b09d37c25.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 6. 使用 xfsdump -f 命令去 opt 目录下面起一个名字备份 ，设备是 sdb1

![](https://s1.51cto.com/images/blog/201908/27/f9bc24f451144e987e22692054c5de95.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)![](https://s1.51cto.com/images/blog/201908/27/66de58c148762321cf073eb29166cfb5.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 7. 删除 data 所有文件，使用 xfsrestore -f 命令去恢复

![](https://s1.51cto.com/images/blog/201908/27/f8fc3bb6d2741b02416fa7521ef7f4e1.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

#### 8. 查看恢复

![](https://s1.51cto.com/images/blog/201908/27/5416b102e8376a37793b7181efa6b65d.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)