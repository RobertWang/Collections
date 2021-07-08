> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.coolapk.com](https://www.coolapk.com/feed/25682012?shareKey=MjdjMWU2OGQ5NmM5NjA1NWU1MTk~&shareUid=761220&shareFrom=com.coolapk.app_3.0)

> 一、问题回答 1. 我的 xxx 手机能用吗？ 现在只有 * 高通骁龙 845 * 平台受支持，未来的话大 概率也不会变。请检查你的手机 cpu 型号。

一、问题回答  
1. 我的 xxx 手机能用吗？  
现在只有 * 高通骁龙 845 * 平台受支持，未来的话大 概率也不会变。请检查你的手机 cpu 型号。

Windows 设备支持状态[查看链接](https://github.com/edk2-porting/edk2-sdm845/wiki/Windows%E7%8A%B6%E6%80%81 "https://github.com/edk2-porting/edk2-sdm845/wiki/Windows%E7%8A%B6%E6%80%81")

2. 现在项目状态怎么样？什么能用？  
只有约 50% 的 845 设备 USB 正常工作，例如：一加 6/6t，小米 mix3, 魅族 16th,USB 正常的都可以安装完整的 Windows10 触摸屏正常工作的则更少，目前只有一加 6(t)。

二、安装预准备  
Windows/Linux/Mac 电脑、U 盘、OTG hub、已解锁 845 手机并刷入 twrp recovery Windows on Arm64 镜像

三、修改分区和安装 pe  
1. 使用 parted 缩小 userdata 分区

![](http://image.coolapk.com/feed/2021/0320/17/761220_95d6d15d_4389_0891@1440x720.jpeg.m.jpg)

我的 6t 分区参考

进去 recovery, 先把 parted 文件放到 / sbin 目录下并赋予 755 权限, 复制完后 twrp 终端或者 adb shell 输入一下命令, 修改分区可能会砖！！！

先看这个 parted 教程

[查看链接](https://www.coolapk.com/feed/25327190?shareKey=NzRiYjcxMWQxY2FiNjA1NGQ4MGU~&shareUid=761220&shareFrom=com.coolapk.market_11.0.3 "https://www.coolapk.com/feed/25327190?shareKey=NzRiYjcxMWQxY2FiNjA1NGQ4MGU~&shareUid=761220&shareFrom=com.coolapk.market_11.0.3")  
分区命令示例：

-----------------------------------------------  
1. umount /data && umount /sdcard# 取消 data 分区挂载  
2. parted /dev/block/sda #选择手机硬盘  
3.p #显示分区信息  
4.rm 21 #21 指 userdata 分区，删除 21 分区，看自己的分区表删除，记住起始和和结束  
5.mkpart Windows ntfs xxx 69.6G# 创建 Window 分区，大小至少 20GB xxx 为你上一分区的 end，也是原来 userdata 的开始，出现警告⚠️输入 i 忽略  
6.mkpart userdata ext4 70G XXX  
# 创建 userdata 分区，大小看着给 xxx 128G 存储的，将 70G 到硬盘原 userdata 分区的结束我这里是 125G 的空间给安卓 data  
7.p #显示分区信息先看看自己的分区表，确认一下有没分错  
8.q 退出 parted

9. 格式化分区  
mkfs.fat -F32 /dev/block/by-name/userdata  
mkfs.ntfs -f /dev/block/by-name/Windows  
10. 重启 recovery, 点击清除 - 选择 data - 修改文件系统 - 格式为 fat  
11. 然后把群里的 20h2pe_new.zip 解压到 u 盘，插入 u 盘到手机，点击挂载把 u 盘挂载，然后使用命令复制到 data 分区  
mount /dev/block/by-name/userdata /mnt  
cp -r /usbstorage/20h2pe_new/* /mnt  
注意 u 盘下 20h2pe_new / 下的文件为

![](http://image.coolapk.com/feed/2021/0320/17/761220_d2b70f7d_4389_0893@854x339.jpeg.m.jpg)

3. 编译并刷入 uefi  
编译教程 [@梦毁我心](https://www.coolapk.com/u/%E6%A2%A6%E6%AF%81%E6%88%91%E5%BF%83) [查看链接](https://b23.tv/qcketh "https://b23.tv/qcketh")  
也可找我帮编译  
可以使用电脑  
1. fastboot boot boot-xxx.img 进入临时 uefi，魅族 16th 无法使用 fastboot boot  
也可刷入 uefi 镜像到 boot 分区，

重启即可进入 pe

四、diskpart 新建 efi 分区

⚠️一加 6/6t 必须运行在官方最新安卓 10 底层上，安卓 9 必卡 fastboot，其他 ab 分区手机可能也要最新底层

接入键盘鼠标

1. 在 CMD 输入 diskpart，回车等待。

2. 输入 list disk，可以看到硬盘编号，0,1,2,3…

3. 输入 select disk 0（0 是硬盘编号）选中你要分区的硬盘。

4. 输入 create partition efi size=xxx（xxx 是分区大小，以 MB 为单位，例如 300MB）。  
5. 输入 format quick fs=fat 格式化 efi 分区  
6.assign letter=Y 为 efi 分区分配盘符 Y  
磁盘分区命令示例：  
-----------------------------------------------

select disk 0  
list part  
create partition efi size=300  
format quick fs=fat  
assign letter=Y  
参考链接: [查看链接](https://www.bilibili.com/read/mobile/7142003 "https://www.bilibili.com/read/mobile/7142003")

五、安装 Windows10

1. 将 Windows 镜像、dism++、群里下载的驱动解压放入 u 盘，连接 u 盘  
2. 打开 dism++-arm64，文件 - 释放镜像

![](http://image.coolapk.com/feed/2021/0320/17/761220_e7896270_4389_0895@983x694.jpeg.m.jpg)

![](http://image.coolapk.com/feed/2021/0320/17/761220_d698bec7_4389_0896@984x693.jpeg.m.jpg)

使用 dism++ 释放 Windows10 镜像到 ntfs 分区并选择添加引导，选择刚创建的 efi 分区  
3. 完成后，选择驱动目录的注入全部驱动

![](http://image.coolapk.com/feed/2021/0320/17/761220_5a11ecd1_4389_0901@1043x550.jpeg.m.jpg)

如图

六、关闭驱动签名和打开测试模式  
在 cmd 输入以下命令  
注意：Y 为你的 efi 分区盘符号 / set 后面有空格  
bcdedit /store Y:\efi\microsoft\boot\bcd /set {Default} testsigning on

bcdedit /store Y:\efi\microsoft\boot\bcd /set {Default} nointegritychecks on

七、重启进入系统  
1. 建议把 data 备份到 u 盘防止系统安装失败，恢复即可进入 pe  
2. 进入 twrp 格式化 data 分区  
3. 用 parted 修改 efi 分区名，  
parted /dev/block/sda

name xxx esp #xxx 为 efi 分区的前面的编号

p 查看是否成功更改

不修改可能无法进入 fastboot  
4. 开机即可进入 Windows  
不是 ab 分区的手机可以把 uefi 刷入 recovery 分区，开机按特定进入 rec 按键即可进入 Windows  
八、开机后常见问题解决  
1. 无法在此设备安装 Window  
解决方法

![](http://image.coolapk.com/feed/2021/0320/17/761220_a02ff38e_4389_0903@720x731.jpeg.m.jpg)

其他问题群里提问，pe 和驱动文件也在群里  
uefi 项目地址：[查看链接](https://github.com/edk2-porting/edk2-sdm845 "https://github.com/edk2-porting/edk2-sdm845")  
QQ 交流群: 697666196  
[#woa 铺路#](https://www.coolapk.com/t/woa%E9%93%BA%E8%B7%AF?type=12) [#骁龙 845#](https://www.coolapk.com/t/%E9%AA%81%E9%BE%99845?type=12)