> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488486&idx=2&sn=209f40f0a8682fbb99ba45d28bf5c15a&chksm=ce0da465f97a2d7391fcfb226d1310f750de7fdc48ad1eed549f84dfad3641d7fd81b7491eb4&scene=21#wechat_redirect)

作者 | greenbird6  

链接 | nowcoder.com/discuss/151562

一、文件和目录

**1. cd 命令**

它用于切换当前目录，它的参数是要切换到的目录的路径，可以是绝对路径，也可以是相对路径

cd /home    进入 '/ home' 目录

cd ..            返回上一级目录

cd ../..         返回上两级目录

cd               进入个人的主目录

cd ~user1   进入个人的主目录

cd -             返回上次所在的目录

**2. pwd 命令**

pwd 显示工作路径

**3. ls 命令** 

查看文件与目录的命令，list 之意

ls 查看目录中的文件

ls -l 显示文件和目录的详细资料

ls -a 列出全部文件，包含隐藏文件

ls -R 连同子目录的内容一起列出（递归列出），等于该目录下的所有文件都会显示出来  

ls *[0-9]* 显示包含数字的文件名和目录名

**4. cp 命令**

用于复制文件，copy 之意，它还可以把多个文件一次性地复制到一个目录下

-a ：将文件的特性一起复制

-p ：连同文件的属性一起复制，而非使用默认方式，与 - a 相似，常用于备份

-i ：若目标文件已经存在时，在覆盖时会先询问操作的进行

-r ：递归持续复制，用于目录的复制行为

-u ：目标文件与源文件有差异时才会复制

**5.  mv 命令**（用于移动文件、目录或更名，move 之意）

-f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖

-i ：若目标文件已经存在，就会询问是否覆盖

-u ：若目标文件已经存在，且比目标文件新，才会更新

**6.  rm 命令**（用于删除文件或目录，remove 之意）

-f ：就是 force 的意思，忽略不存在的文件，不会出现警告消息

-i ：互动模式，在删除前会询问用户是否操作

-r ：递归删除，最常用于目录删除，它是一个非常危险的参数

二、查看文件内容

**7. cat 命令**（用于查看文本文件的内容，后接要查看的文件名，通常可用管道与 more 和 less 一起使用）

cat file1 从第一个字节开始正向查看文件的内容

tac file1 从最后一行开始反向查看一个文件的内容

cat -n file1 标示文件的行数

more file1 查看一个长文件的内容

head -n 2 file1 查看一个文件的前两行

tail -n 2 file1 查看一个文件的最后两行

tail -n +1000 file1  从 1000 行开始显示，显示 1000 行以后的

cat filename | head -n 3000 | tail -n +1000  显示 1000 行到 3000 行

cat filename | tail -n +3000 | head -n 1000  从第 3000 行开始，显示 1000(即显示 3000~3999 行)

三、文件搜索

**8. find 命令**

find / -name file1 从 '/' 开始进入根文件系统搜索文件和目录

find / -user user1 搜索属于用户'user1' 的文件和目录

find /usr/bin -type f -atime +100 搜索在过去 100 天内未被使用过的执行文件

find /usr/bin -type f -mtime -10 搜索在 10 天内被创建或者修改过的文件

whereis halt 显示一个二进制文件、源码或 man 的位置

which halt 显示一个二进制文件或可执行文件的完整路径

删除大于 50M 的文件：

find /var/mail/ -size +50M -exec rm {} ＼;

四、文件的权限 

- 使用 "+" 设置权限，使用 "-" 用于取消

**9. chmod 命令**

ls -lh 显示权限

chmod ugo+rwx directory1 设置目录的所有人 (u)、群组(g) 以及其他人 (o) 以读（r，4 ）、写 (w，2) 和执行 (x，1) 的权限

chmod go-rwx directory1  删除群组 (g) 与其他人 (o) 对目录的读写执行权限

**10. chown 命令**（改变文件的所有者）

chown user1 file1 改变一个文件的所有人属性

chown -R user1 directory1 改变一个目录的所有人属性并同时改变改目录下所有文件的属性

chown user1:group1 file1 改变一个文件的所有人和群组属性

**11. chgrp 命令**（改变文件所属用户组）

chgrp group1 file1 改变文件的群组

五、文本处理

**12. grep 命令**（分析一行的信息，若当中有我们所需要的信息，就将该行显示出来，该命令通常与管道命令一起使用，用于对一些命令的输出进行筛选加工等等）

grep Aug /var/log/messages  在文件 '/var/log/messages'中查找关键词 "Aug" 

grep ^Aug /var/log/messages 在文件 '/var/log/messages'中查找以 "Aug" 开始的词汇

grep [0-9]  /var/log/messages 选择 '/var/log/messages' 文件中所有包含数字的行

grep Aug -R /var/log/* 在目录 '/var/log' 及随后的目录中搜索字符串 "Aug"

sed 's/stringa1/stringa2/g' example.txt 将 example.txt 文件中的 "string1" 替换成 "string2"

sed '/^$/d' example.txt 从 example.txt 文件中删除所有空白行

**13. paste 命令**

paste file1 file2 合并两个文件或两栏的内容

paste -d '+' file1 file2 合并两个文件或两栏的内容，中间用 "+" 区分

**14. sort 命令**

sort file1 file2 排序两个文件的内容

sort file1 file2 | uniq 取出两个文件的并集 (重复的行只保留一份)

sort file1 file2 | uniq -u 删除交集，留下其他的行

sort file1 file2 | uniq -d 取出两个文件的交集 (只留下同时存在于两个文件中的文件)

**15. comm 命令**

comm -1 file1 file2 比较两个文件的内容只删除'file1' 所包含的内容

comm -2 file1 file2 比较两个文件的内容只删除'file2' 所包含的内容

comm -3 file1 file2 比较两个文件的内容只删除两个文件共有的部分

六、打包和压缩文件

**16. tar 命令**（对文件进行打包，默认情况并不会压缩，如果指定了相应的参数，它还会调用相应的压缩程序（如 gzip 和 bzip 等）进行压缩和解压）

-c ：新建打包文件

-t ：查看打包文件的内容含有哪些文件名

-x ：解打包或解压缩的功能，可以搭配 - C（大写）指定解压的目录，注意 - c,-t,-x 不能同时出现在同一条命令中

-j ：通过 bzip2 的支持进行压缩 / 解压缩

-z ：通过 gzip 的支持进行压缩 / 解压缩

-v ：在压缩 / 解压缩过程中，将正在处理的文件名显示出来

-f filename ：filename 为要处理的文件

-C dir ：指定压缩 / 解压缩的目录 dir

压缩：tar -jcv -f filename.tar.bz2 要被处理的文件或目录名称

查询：tar -jtv -f filename.tar.bz2

解压：tar -jxv -f filename.tar.bz2 -C 欲解压缩的目录

bunzip2 file1.bz2 解压一个叫做'file1.bz2'的文件

bzip2 file1 压缩一个叫做'file1' 的文件

gunzip file1.gz 解压一个叫做'file1.gz'的文件

gzip file1 压缩一个叫做'file1'的文件

gzip -9 file1 最大程度压缩

rar a file1.rar test_file 创建一个叫做'file1.rar' 的包

rar a file1.rar file1 file2 dir1 同时压缩'file1', 'file2' 以及目录'dir1'

rar x file1.rar 解压 rar 包

zip file1.zip file1 创建一个 zip 格式的压缩包

unzip file1.zip 解压一个 zip 格式压缩包

zip -r file1.zip file1 file2 dir1 将几个文件和目录同时压缩成一个 zip 格式的压缩包

七、系统和关机 (系统的关机、重启以及登出)

shutdown -h now 关闭系统 (1)

init 0 关闭系统 (2)

telinit 0 关闭系统 (3)

shutdown -h hours:minutes & 按预定时间关闭系统

shutdown -c 取消按预定时间关闭系统

shutdown -r now 重启 (1)

reboot 重启 (2)

logout 注销

time 测算一个命令（即程序）的执行时间

八、进程相关的命令

jps 命令 （显示当前系统的 java 进程情况，及其 id 号）

jps(Java Virtual Machine Process Status Tool) 是 JDK 1.5 提供的一个显示当前所有 java 进程 pid 的命令，简单实用，非常适合在 linux/unix 平台上简单察看当前 java 进程的一些简单情况。

ps 命令 （用于将某个时间点的进程运行情况选取下来并输出，process 之意）

-A ：所有的进程均显示出来

-a ：不与 terminal 有关的所有进程

-u ：有效用户的相关进程

-x ：一般与 a 参数一起使用，可列出较完整的信息

-l ：较长，较详细地将 PID 的信息列出

ps aux # 查看系统所有的进程数据

ps ax # 查看不与 terminal 有关的所有进程

ps -lA # 查看系统所有的进程数据

ps axjf # 查看连同一部分进程树状态

kill 命令 （用于向某个工作（%jobnumber）或者是某个 PID（数字）传送一个信号，它通常与 ps 和 jobs 命令一起使用）

killall 命令  （向一个命令启动的进程发送一个信号）

top 命令 是 Linux 下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于 Windows 的任务管理器。

如何杀死进程：

（1）图形化界面的方式

（2）kill -9 pid  （-9 表示强制关闭）

（3）killall -9 程序的名字

（4）pkill 程序的名字

查看进程端口号：

netstat -tunlp|grep 端口号

九、普通文件和目录文件的区别

**9.1 文件的类型**

Linux 下面一切皆文件，配置是文件，设备是文件，目录也是特殊的文件，文件有如下几种：

d：目录文件的标识是，

-：普通文件标识，

l：软连接文件，亦称符号链接文件；

b，块文件，是设备文件的一种（还有另一种），b 是 block 的简写。

c，字符文件，也是设备文件的一种，c 是 character 的文件。

**9.2 普通文件和目录文件**

普通文件：存储普通数据，一般就是字符串。

目录文件：存储了一张表，该表就是该目录文件下，所有文件名和 inode 的映射关系。

**9.3 权限的区别**

对于普通文件来说，rwx 的意义是：

r：可以获得这个普通文件的名字和内容。

w：可以修改这个文件的内容和文件名。可以删除该文件。

x：该文件是否具有被执行的权限。

对于目录文件来说，rwx 的意义是：

r：表示具有读取目录结构列表的权限，所以当你具有读取 (r) 一个目录的权限时，表示你可以查询该目录下的文件名。 就可以利用 ls 这个命令将该目录的内容列表显示出来， 必须这个目录有 x 的权限，才可以进入这个目录。

w：移动该目录结构列表的权限（建立新的文件与目录、删除已经存在的文件与目录、更名、移动位置）。

x：目录不可以被执行，目录的 x 代表的是用户能否进入该目录成为工作目录。

如果喜欢本篇文章，欢迎转发、点赞。关注订阅号「Web 项目聚集地」，回复「技术博文」即可获取更多图文教程、技术博文。

推荐阅读

_**1.**_ [因 转账失败 引发的思考...](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488455&idx=1&sn=7dcd97f1824cf791c7e0f2c4232e6bbf&chksm=ce0da444f97a2d5285e9244fdd38b5ee404c1e71fc19fba486835a57ce08f0d0794e31f87546&scene=21#wechat_redirect)

_**2.**_ [如何优雅地编码？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488455&idx=2&sn=3e7956ce8909a4cc5526867af0ff8ec4&chksm=ce0da444f97a2d524dbe79eb845c94cbe8a1f45829f73e1923d2b0f2f81cfb510fcec6fd9c7f&scene=21#wechat_redirect)

_**3.**_ [Java 程序员 “吃完饭直接走”](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=1&sn=4cef717f2e51f8c8e969d549e014cbed&chksm=ce0da43bf97a2d2d8a572536f7c2e92525e5f374f0298b2d85925c2243f84129c7a2d47ce21e&scene=21#wechat_redirect)

_**4.**_ [一键登陆了解一下？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=2&sn=818107afc6c7e62ffec4bbecc0f11359&chksm=ce0da43bf97a2d2d1810a6c565e6bc9f38bd757b758a9637a20ff246bba0f9a54a2c3589198e&scene=21#wechat_redirect)

_**5.**_ [9 张 Java 技术流程图](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=3&sn=6d2c3de5fd95236e471c1ed0361843df&chksm=ce0da43bf97a2d2d25473580e1571b0b182cd62c4f9a0eba92c4c0a8b43ea6c1049a06abc776&scene=21#wechat_redirect)

_**6.**_ [MySQL 请不要使用 “utf-8”](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=4&sn=e2970c3db59752acf282741c4aed0364&chksm=ce0da43bf97a2d2df92bcf57aaf8e444e6631035e60c828bfb7416465fc5a14f7978c2fba60e&scene=21#wechat_redirect)

_**7.**_ [还不懂 Java 中的多线程 ？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488085&idx=1&sn=66d41311ea6c4eeb4418a007e5d67b27&chksm=ce0da5d6f97a2cc0e4ea693f403adbd11614eedc9fbb2e3e7e6c51c3b9d03c60e8844ec6eb9d&scene=21#wechat_redirect)

_**8.**_ [如何编写轻量级 CSS 框架](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488286&idx=1&sn=08d7d117bac2c520e9847c03c70f576e&chksm=ce0da49df97a2d8b7c0fc13620eef9542e5bbec1de7d85454aedc9eb64483985bca7ff1ab0bc&scene=21#wechat_redirect)