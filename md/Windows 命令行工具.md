> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/61opS06xEeeKpQHAhTTpkw)

其中 cmd 引入的命令可以在 cmd 窗口中输入 help 查看命令子集：

![](https://mmbiz.qpic.cn/mmbiz_png/ulAibOLeecVu7AQBy097hr4DrsWaKg8KrSyTN1taqLmWNClBfjqZDMt8dAPP4MQtQia5spnKpsN8T5RIvGJMbgnw/640?wx_fmt=png)

系统环境变量引入的可执行文件主要在系统目录中，其中有三类可执行文件我们遇到的比较多，分别是以 exe 、cpl、msc 结尾的文件。

这些文件都在系统目录 system32 下面，我们可以在 git-bash 中对这些文件进行搜索。

```
$ cd "C:\Windows\System32"
$ find . -maxdepth 1 -iname "*.cpl"
$ find . -maxdepth 1 -iname "*.msc"

```

![](https://mmbiz.qpic.cn/mmbiz_png/ulAibOLeecVu7AQBy097hr4DrsWaKg8Kricv2EZu9zu0pC10Riasoj4GfqKpRQf8sDGHCEQMFjkPCjvfwZo3APCGg/640?wx_fmt=png)

其中 msc 结尾的命令可以不用把命令拼全，比如 devmgmt.msc 在命令行中输入 devmgmt 也可以执行。但 cpl 的命令必须输全，比如 appwiz.cpl 如果输入 appwiz 是无法找到命令的。

*   [Windows 用命令打开常用设置](http://mp.weixin.qq.com/s?__biz=Mzk0MTI4NTIzNQ==&mid=2247489134&idx=1&sn=f9651b3b467665806ccacb44107e9f23&chksm=c2d59c72f5a21564db641ec2e297541d0ceec1af086ce3ac62c9555d8eb7703fe0f2659ed89d&scene=21#wechat_redirect)  
    

```
secpol.msc  安全设置
gpedit.msc  组策略
fsmgmt.msc  共享
eventvwr.msc  事件查看器
services.msc  服务
taskschd.msc  计划任务
virtmgmt.msc  hyper-v 管理器
lusrmgr.msc   用户和组
devmgmt.msc   设备管理器
diskmgmt.msc  磁盘管理器
certmgr.msc   证书
printmanagement.msc  打印管理
perfmon.msc  性能监视器

```

```
mstsc.exe  远程桌面连接程序
msconfig.exe 引导配置
gpupdate.exe 组策略更新
DisplaySwitch.exe 切换屏幕
driverquery.exe 驱动查询
findstr.exe   windows 版的 grep
hdwwiz.exe  添加硬件向导
getmac.exe  获取mac地址
ipconfig.exe  查网络配置
label.exe   修改磁盘卷标
mountvol.exe    windows 版的 mount
services.exe 与 services.msc 功能一致
SnippingTool.exe  windows 自带的截图工具
subst.exe  虚拟卷挂载
taskkill.exe 杀进程
tasklist.exe 列出进程
Taskmgr.exe  任务管理器

```

```
C:\Users\hyang0>subst /?
将路径与驱动器号关联。
SUBST [drive1: [drive2:]path]
SUBST drive1: /D
  drive1:        指定要分配路径的虚拟驱动器。
  [drive2:]path  指定物理驱动器和要分配给虚拟驱动器的路径。
  /D             删除被替换的
(虚拟)驱动器。
不带参数键入 SUBST，以显示当前虚拟驱动器的列表。

```

```
https://www.itechtics.com/msc-files/?utm_content=expand_article

```