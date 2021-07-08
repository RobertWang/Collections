> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [silime.gitee.io](https://silime.gitee.io/2021/05/20/Windows10-on-arm64/)

> 下载这些文件到你的 U 盘

#### [](#first "first")first[](#first)

下载这些文件到你的 U 盘

1.  下载 PE  
    [20h2pe_new.zip](https://www.dropbox.com/s/5c98gpqehe4n3p0/20h2pe_new.zip?dl=0)  
    或者 QQ 交流群: 697666196 里下载
    
2.  下载 dism++
    
    [Dism++](http://www.chuyu.me/zh-Hans/index.html)
    
3.  下载 SDM845 驱动
    
    [GitHub -WOA-Drivers](https://github.com/edk2-porting/WOA-Drivers)
    
4.  下载 windows10 arm64 iso
    
    [UUP dump](https://uupdump.net/?lang=zh-cn)
    
5.  下载 uefi
    
    [Releases · edk2-porting/edk2-sdm845 · GitHub](https://github.com/edk2-porting/edk2-sdm845/releases)
    
6.  下载 parted
    
    [parted](https://pwdx.lanzoux.com/iUgSEmkrlmh)
    
7.  新建 new.txt 文件
    
    ```
    bcdedit /store Y:\efi\microsoft\boot\bcd /set {Default} testsigning on
    bcdedit /store Y:\efi\microsoft\boot\bcd /set {Default} nointegritychecks on
    ```
    

#### [](#second "second")second[](#second)

​ 电脑连接手机进入 TWRP

1.  分区（仅限一加 6T 复制粘贴）
    
    ```
    cp /sdcard/parted /sbin/ && chmod 755 /sbin/parted
    umount /data && umount /sdcard
    parted /dev/block/sda
    rm 17 
    mkpart esp fat32 6559MB 7000MB
    mkpart pe fat32 7000MB 10000MB
    mkpart win ntfs 10000MB 70GB
    mkpart userdata ext4 70GB 125GB
    set 17 esp on
    ```
    
    2.  格式化新分区
    
    ```
    mkfs.fat -F32 -s1 /dev/block/by-name/pe
    mkfs.fat -F32 -s1 /dev/block/by-name/esp
    mkfs.ntfs -f /dev/block/by-name/win
    mke2fs -t ext4 /dev/block/by-name/userdata
    ```
    
    挂载 PE 分区到 /mnt
    
    ```
    mount /dev/block/by-name/pe /mnt
    ```
    
    1.  OTG 连接 U 盘，复制 pe 文件到 PE 分区
    
    ```
    cp -r /usbstorage/20h2pe_new/* /mnt
    ```
    
    2.  重启进入 Android

#### [](#Third "Third")Third[](#Third)

1.  boot uefi
    
    ```
    fastboot boot uefi.img
    ```
    
2.  进入 PE 系统
    
    挂载 ESP 分区
    
    ```
    diskpart
    select disk 0
    list part
    select part 17 //17为你的esp分区号
    assign letter=Y
    ```
    
3.  安装 windows arm64
    
    1.  打开 dism++ 释放镜像到 D 盘，并选择释放引导分区
    2.  安装驱动
4.  关闭驱动签名
    
    ```
    bcdedit /store Y:\efi\microsoft\boot\bcd /set {Default} testsigning on
    bcdedit /store Y:\efi\microsoft\boot\bcd /set {Default} nointegritychecks on
    ```
    
5.  重启，boot uefi 进入完整 Windows 系统
    
6.  参考[骁龙 845 安装 Windows10 指南](https://www.coolapk.com/feed/25682012?shareKey=MjdjMWU2OGQ5NmM5NjA1NWU1MTk~&shareUid=761220&shareFrom=com.coolapk.app_3.0)