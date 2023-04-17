> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/JMLiu/p/10318379.html)

前置知识：你必须知道 grub 的启动过程以及 bios 和 uefi 的相关基础知识，可以参考：[《Unified Extensible Firmware Interface Wikipedia》](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface)、[《linux 启动过程简介》](https://opensource.com/article/17/2/linux-boot-and-startup)

先说说三个变量是干嘛的：

cmdpath
=======

当前被加载的 "core.img"（bios 的 core.img，uefi 的 BOOTX64.EFI 或 grubx64.efi 等镜像）所在目录的绝对路径。例如：UEFI 启动可能是'(hd0,gpt1)/EFI/GRUB 或'(cd0)/EFI/BOOT'，BIOS 启动可能是'(hd0)'

prefix
======

这个变量是指 GRUB2 的安装目录，即绝对路径形式的'/boot/grub'目录位置，如例如'(hd0,gpt1)/grub'或'(hd0,msdos2)/boot/grub'。初始值由 GRUB 在启动时根据 "grub-install" 在安装时提供的信息自动设置，这些信息是由 grub-install 脚本直接写入到 grub 镜像（bios 的 core.img，uefi 的 BOOTX64.EFI 或 grubx64.efi 等镜像）的，当然，也可以在 grub 的 rescue 模式下用 grub 的命令直接修改，如: set prefix=(hd1,gpt2)/grub。

GRUB2 的安装目录中存在着这么几个文件夹和文件

文件夹：

fonts: 存在着一些字体

locale: 区域化相关

x86_64-efi: 模块的二进制文件，以. mod 为后缀。你 insmod 所操作的模块，都在这里面有，比如 ext4.mod、fat.mod 这些文件系统模块，gzio 等解压支持的模块。

themes: 一些主题，就是你 grub 界面的背景啥的。

文件：

grub.cfg: 这是 grub 的启动 shell 脚本，对于用户来说，是最重要的文件，几乎是用户配置 grub 的唯一的配置文件。通过这个文件用户可以控制 grub 加载操作系统的行为，比如添加一个 menuentry 就是添加一个操作系统启动选项，每个选项中可以指定操作系统内核尽享和 initramfs 镜像等等。下面是一个 grub.cfg 的实例：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
  1 #由于"$prefix"的值是在"grub-install"安装时确定的，并且嵌入'core.img'中的模块也是随硬件变化的，
  2 #所以不要只是简单的复制'grub'目录到处使用，而应该在每一个介质上都使用"grub-install"进行安装。
  3 ###############################################################################
  4 #(1)本配置文件要求"grub"目录所在分区的卷标必须是"GRUB2"。
  5 #(2)为了保持最大程度的BIOS兼容性，"GRUB2"分区必须位于磁盘的前 137GB 范围。
  6 ###############################################################################
  7 # 如果要在windows上安装GRUB2的话，必须首先以管理员身份打开命令提示符，然后运行
  8 # wmic diskdrive list brief
  9 # 命令查看安装目标，最后再使用例如下面这样的命令进行安装：
 10 # grub-install.exe --boot-directory=g: --recheck --target=x86_64-efi --efi-directory=g: --no-nvram --removable \\.\PHYSICALDRIVE5
 11 # grub-install.exe --boot-directory=g: --recheck --target=i386-pc \\.\PHYSICALDRIVE5
 12 ###############################################################################
 13 
 14 #################
 15 ## (1)特殊变量 ##
 16 #################
 17 #禁止验证签名
 18 set check_signatures=no
 19 #默认启动第一个菜单项
 20 set default=0
 21 #如果第一个菜单项启动失败，转而启动第二个菜单项
 22 set fallback=1
 23 #优先使用最常规的1024x768分辨率，以保证在不同的屏幕上拥有一致的菜单效果，如果失败再自动匹配分辨率
 24 set gfxmode=1024x768,auto
 25 #使用自己制作的24px的大号字体以避免默认字体太小看不清
 26 set gfxterm_font=WenQuanYiMicroHeiMono24px
 27 #将GRUB2设置为简体中文界面
 28 set lang=zh_CN
 29 #指定翻译文件(*.mo)的目录，若未明确设置此目录，则无法显示中文界面。
 30 set locale_dir=$prefix/locale
 31 #每一满屏后暂停输出，以免信息太多一闪而过看不清
 32 set pager=1
 33 #开启密码验证功能，并设置一个名为'root'的超级用户
 34 set superusers=root
 35 #设置菜单的超时时间为5秒
 36 set timeout=5
 37 
 38 #################
 39 ## (2)公共模块 ##
 40 #################
 41 #两种最流行的磁盘分区格式(partmap.lst)
 42 insmod part_gpt
 43 insmod part_msdos
 44 #常见文件系统驱动(fs.lst)
 45 insmod fat
 46 insmod exfat
 47 insmod ntfs
 48 insmod iso9660
 49 insmod ext2
 50 insmod xfs
 51 #一次性加载所有可用的视频驱动
 52 insmod all_video
 53 #图形模式终端
 54 insmod gfxterm
 55 #背景图片支持
 56 insmod png
 57 
 58 #########################################
 59 ## (3)公共命令(必须放在模块和变量之后) ##
 60 #########################################
 61 #激活图形模式的输出终端，以允许使用中文和背景图
 62 terminal_output  gfxterm
 63 #设置背景图片
 64 background_image $prefix/themes/1024x768.png
 65 #加载自己制作的24px的大号字体文件($prefix/fonts/WenQuanYiMicroHeiMono24px.pf2)
 66 loadfont WenQuanYiMicroHeiMono24px
 67 #设置'root'用户的哈希密码[通过"grub-mkpasswd-pbkdf2"工具生成]
 68 password_pbkdf2 root grub.pbkdf2.sha512.69.7DBCA469F80EA1C0A8A1E2FEBC4F8463.B073C1C89EC1E85309C3D6A1BAFF4356
 69 
 70 #################
 71 ## (4)菜单项   ##
 72 #################
 73 
 74 menuentry '正常启动(Windows)' --unrestricted {
 75     if [ 'pc' == $grub_platform ] ; then
 76         if search --file --set /bootmgr ; then
 77             chainloader +1
 78         elif search --file --set /ntldr ; then
 79             chainloader +1
 80         fi
 81     elif [ 'efi' == $grub_platform ] ; then
 82         if search --file --set /EFI/Microsoft/Boot/bootmgfw.efi ; then
 83             chainloader /EFI/Microsoft/Boot/bootmgfw.efi
 84         fi
 85     fi
 86 }
 87 
 88 #http://www.wepe.com.cn/download.html
 89 menuentry '系统救援(WinPE)' --users=root {
 90     if [ 'pc' == $grub_platform -a -f $prefix/winpe/memdisk ] ; then
 91         if [ -f $prefix/winpe/WinPE.iso ] ; then
 92             linux16  $prefix/winpe/memdisk iso raw
 93             initrd16 $prefix/winpe/WinPE.iso
 94         fi
 95     elif [ 'efi' == $grub_platform -a -f $prefix/winpe/bootmgfw.efi -a -f $prefix/winpe/BCD ] ; then
 96         if [ -f $prefix/winpe/WinPE.wim -a -f $prefix/winpe/boot.sdi ] ; then
 97             chainloader $prefix/winpe/bootmgfw.efi
 98         fi
 99     fi
100 }
101 
102 #https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/
103 if [ -f $prefix/linux/archlinux.iso ] ; then
104     menuentry 'Arch Linux LiveCD [root/空]' --unrestricted {
105         loopback loop0 $prefix/linux/archlinux.iso
106         linux  (loop0)/arch/boot/x86_64/vmlinuz img_label=GRUB2 img_loop=/grub/linux/archlinux.iso   systemd.wants=sshd.service
107         initrd (loop0)/arch/boot/x86_64/archiso.img
108     }
109 fi
110 
111 #https://mirrors.163.com/gentoo/releases/amd64/autobuilds/current-install-amd64-minimal/
112 #因为Gentoo官方的最小安装CD不支持UEFI启动，所以不可用于在UEFI环境中安装Gentoo
113 if [ -f $prefix/linux/install-amd64-minimal.iso ] ; then
114     menuentry 'Mini Gentoo LiveCD [root/123]' --unrestricted {
115         loopback loop0 $prefix/linux/install-amd64-minimal.iso
116         linux  (loop0)/isolinux/gentoo cdroot isoboot=/grub/linux/install-amd64-minimal.iso  dosshd nokeymap passwd=123
117         initrd (loop0)/isolinux/gentoo.igz
118     }
119 fi
120 
121 #https://mirror.umd.edu/calculate/release/
122 #https://www.calculate-linux.org/main/en/download
123 # Calculate Linux Scratch 是基于Gentoo制作的 LiveCD ，
124 #包含编译工具(GCC,Binutils,...)、支持UEFI、支持WiFi，
125 #既可用作LFS(Linux From Scratch)的宿主系统、也可用于在UEFI环境中安装 Gentoo
126 if [ -f $prefix/linux/cls.iso ] ; then
127     menuentry 'Live GCC x64 [root/root]' --unrestricted {
128         loopback loop0 $prefix/linux/cls.iso
129         linux  (loop0)/boot/vmlinuz root=live iso-scan/filename=/grub/linux/cls.iso   vga=current nodevfs noresume nodmraid
130         initrd (loop0)/boot/initrd
131     }
132 fi
133 
134 #https://mirrors.huaweicloud.com/centos/7/isos/x86_64/
135 if [ -f $prefix/linux/CentOS-LiveGNOME.iso ] ; then
136     menuentry 'CentOS 7.6 GNOME LiveCD [root/空](可退出liveuser后再用root登录)' --unrestricted {
137         loopback loop0 $prefix/linux/CentOS-LiveGNOME.iso
138         linux  (loop0)/isolinux/vmlinuz0 rd.live.image root=live:CDLABEL=CentOS-7-x86_64-LiveGNOME-1810 iso-scan/filename=/grub/linux/CentOS-LiveGNOME.iso   systemd.wants=sshd.service inst.lang=zh_CN
139         initrd (loop0)/isolinux/initrd0.img
140     }
141 fi
142 
143 #https://mirrors.ustc.edu.cn/debian-cd/current-live/amd64/iso-hybrid/
144 #也适用于'Kali LiveCD [root/toor]' https://cdimage.kali.org/current/
145 if [ -f $prefix/linux/debian-live-kde.iso ] ; then
146     menuentry 'Debian 9.6 KDE LiveCD (NO SSH)' --unrestricted {
147         loopback loop0 $prefix/linux/debian-live-kde.iso
148         linux  (loop0)/live/vmlinuz-4.9.0-8-amd64 boot=live findiso=/grub/linux/debian-live-kde.iso   components username=root locales=zh_CN.UTF-8
149         initrd (loop0)/live/initrd.img-4.9.0-8-amd64
150     }
151 fi
152 
153 #https://mirrors.aliyun.com/fedora/releases/29/Spins/x86_64/iso/
154 #只有 Xfce LiveCD 可以设置中文界面
155 if [ -f $prefix/linux/Fedora-Xfce-Live.iso ] ; then
156     menuentry 'Fedora 29 Xfce LiveCD [root/空](可退出liveuser后再用root登录)' --unrestricted {
157         loopback loop0 $prefix/linux/Fedora-Xfce-Live.iso
158         linux  (loop0)/isolinux/vmlinuz rd.live.image root=live:CDLABEL=Fedora-Xfce-Live-29-20181029-1 iso-scan/filename=/grub/linux/Fedora-Xfce-Live.iso   systemd.wants=sshd.service locale.LANG=zh_CN.utf8 inst.lang=zh_CN
159         initrd (loop0)/isolinux/initrd.img
160     }
161 fi
162 
163 #https://ftp.sjtu.edu.cn/opensuse/distribution/openSUSE-stable/live/
164 if [ -f $prefix/linux/openSUSE-Leap-KDE-Live.iso ] ; then
165     menuentry 'openSUSE Leap 15.0 KDE LiveCD [root/空]' --unrestricted {
166         loopback loop0 $prefix/linux/openSUSE-Leap-KDE-Live.iso
167         linux  (loop0)/boot/x86_64/loader/linux root=live:CDLABEL=openSUSE_Leap_15.0_KDE_Live iso-scan/filename=/grub/linux/openSUSE-Leap-KDE-Live.iso   systemd.wants=sshd.service lang=zh_CN
168         initrd (loop0)/boot/x86_64/loader/initrd
169     }
170 fi
171 
172 #https://mirrors.shu.edu.cn/ubuntu-cdimage/lubuntu/releases/
173 #也适用于 KDE neon LiveCD https://files.kde.org/neon/images/neon-userltsedition/current/neon-userltsedition-current.iso
174 if [ -f $prefix/linux/lubuntu.iso ] ; then
175     menuentry 'Ubuntu LXQt LiveCD (NO SSH)' --unrestricted {
176         loopback loop0 $prefix/linux/lubuntu.iso
177         linux  (loop0)/casper/vmlinuz boot=casper iso-scan/filename=/grub/linux/lubuntu.iso   username=root locale=zh_CN keyboard-configuration/layoutcode=us
178         initrd (loop0)/casper/initrd
179     }
180 fi
181 
182 menuentry '关机' --unrestricted { halt ; }
183 menuentry '重新启动' --unrestricted { reboot ; }
184 
185 #https://rhinstaller.github.io/anaconda/boot-options.html
186 #硬盘安装通用于所有包含"Packages"目录的 CentOS/Fedora ISO映像(Minimal,DVD,Everything)
187 #网络安装通用于所有包含"images"目录的 CentOS/Fedora ISO映像(Minimal,DVD,Everything,NetInstall)
188 #https://mirrors.zju.edu.cn/centos/7/isos/x86_64/
189 if [ -f $prefix/linux/CentOS-Minimal.iso ] ; then
190     menuentry '硬盘安装 CentOS [最小安装]' --unrestricted {
191         loopback loop0 $prefix/linux/CentOS-Minimal.iso
192         linux  (loop0)/isolinux/vmlinuz inst.repo=hd:LABEL=GRUB2:/grub/linux/CentOS-Minimal.iso   inst.lang=zh_CN
193         initrd (loop0)/isolinux/initrd.img
194     }
195     menuentry '网络安装 CentOS 7.x [不支持WiFi]' --unrestricted {
196         loopback loop0 $prefix/linux/CentOS-Minimal.iso
197         linux  (loop0)/images/pxeboot/vmlinuz inst.repo=https://mirrors.aliyun.com/centos/7/os/x86_64/   inst.lang=zh_CN ip=dhcp nameserver=223.6.6.6
198         initrd (loop0)/images/pxeboot/initrd.img
199     }
200 fi
201 
202 #https://www.debian.org/releases/stable/amd64/ch05s03.html.zh-cn
203 #https://www.debian.org/releases/stable/amd64/ch06s03.html.zh-cn
204 #最好将用于硬盘安装的ISO映像放在文件系统的根目录或第一层子目录中(可简化ISO映像的搜索)
205 #https://mirrors.163.com/debian/dists/stable/main/installer-amd64/current/images/hd-media/
206 #https://mirrors.163.com/ubuntu/dists/bionic/main/installer-amd64/current/images/hd-media/
207 #硬盘安装也适用于 Alternative Ubuntu Server ISO [ http://cdimage.ubuntu.com/releases/18.04/release/ubuntu-18.04.1-server-amd64.iso ]
208 if [ -f $prefix/linux/debian/vmlinuz -a -f $prefix/linux/debian/initrd.gz ] ; then
209     menuentry '硬盘安装 Debian [自动搜索ISO映像(最好放在根目录)]' --unrestricted {
210         linux  $prefix/linux/debian/vmlinuz   priority=low vga=791 locale=zh_CN
211         initrd $prefix/linux/debian/initrd.gz
212     }
213 fi
214 #https://mirrors.163.com/debian/dists/stable/main/installer-amd64/current/images/netboot/
215 #https://mirrors.163.com/ubuntu/dists/bionic/main/installer-amd64/current/images/netboot/
216 #网络安装也适用于 Ubuntu 的 mini.iso 映像(支持WiFi网络)
217 if [ -f $prefix/linux/debian/mini.iso ] ; then
218     menuentry '网络安装 Debian [不支持WiFi]' --unrestricted {
219         loopback loop0 $prefix/linux/debian/mini.iso
220         linux  (loop0)/linux   priority=low vga=791 locale=zh_CN
221         initrd (loop0)/initrd.gz
222     }
223 fi
224 
225 #https://doc.opensuse.org/documentation/leap/startup/html/book.opensuse.startup/cha.boot_parameters.html
226 #https://ftp.sjtu.edu.cn/opensuse/distribution/openSUSE-stable/iso/
227 if [ -f $prefix/linux/openSUSE-Leap-DVD.iso ] ; then
228     menuentry '硬盘安装 openSUSE Leap' --unrestricted {
229         loopback loop0 $prefix/linux/openSUSE-Leap-DVD.iso
230         linux  (loop0)/boot/x86_64/loader/linux install=hd:/grub/linux/openSUSE-Leap-DVD.iso   lang=zh_CN
231         initrd (loop0)/boot/x86_64/loader/initrd
232     }
233 fi
234 #https://mirrors.nju.edu.cn/opensuse/distribution/openSUSE-stable/iso/
235 if [ -f $prefix/linux/openSUSE-Leap-NET.iso ] ; then
236     menuentry '网络安装 openSUSE Leap [支持WiFi]' --unrestricted {
237         loopback loop0 $prefix/linux/openSUSE-Leap-NET.iso
238         linux  (loop0)/boot/x86_64/loader/linux install=https://mirrors.aliyun.com/opensuse/distribution/openSUSE-stable/repo/oss/   lang=zh_CN netsetup=dhcp nameserver=223.6.6.6
239         initrd (loop0)/boot/x86_64/loader/initrd
240     }
241 fi

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

不过，grub.cfg 一般都不会手动编辑的，而是用过 grub-mkconfig -o /boot/grub/grub.cfg 去生成。

grubenv: 预设的一些环境变量可以放到这，是一个文本文件

root
====

root 变量一般在 grub.cfg 执行的过程中设定的，指定了后续脚本中所有文件的其实位置，比如 root=(hd0,gpt3)，那么 / boot/xxx.img 就是在第一块硬盘的第三个分区的 boot 目录下，如果 boot 是单独的分区，比如是第 2 块硬盘的第 2 个分区，那么就是 set root=(hd1,gpt2)； linux vmlinuz-linux； 这两行代码来启动一个 linux 内核了。

几点疑问：
=====

1.grub 是如何确定镜像的安装位置的？

　　如果是 bios/mbr 启动，那么镜像的安装位置是固定的，boot.img 在 mbr 上，core.img 在 post-MBR gap 上。你只需要在安装时指定哪个盘就可以了，比如：

```
grub-install /dev/sda，

```

它就会安装到第一块盘上。

　　如果是 bios/gpt 方式，那么必须要分一个 bootable partition（或者叫 bios boot，不同分区工具，叫法不一样），这个分区不要格式化任何文件格式，留着就好了，一般 1-2M 大就可以，镜像就安装在上面。

　　如果是 uefi/gpt 方式，那么必须要有一个 efi system partition（叫做 esp）的专门分区，分区格式是 esp，文件系统是 flat32。在安装的时候，先把这个分区挂载上，然后用 --efi-directory 指定一下他的位置，如果它是挂载到 / boot/efi 上，那么不用指定，grub-install 会默认 esp 在那。uefi 安装的示例如下：  
  

```
# grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB

```

```
--target指定了系统架构，--bootloader-id指定了grub的efi启动点的名字，到时候你可以在主板设置上看到它。

```

 　　至于 uefi/mbr 方式，那种太奇怪了，你不会碰到的。

2.grub-install 是在安装 grub 时，是如何确定 prefix 的？

你在哪个操作系统上运行它，这个操作系统的 / boot/grub 所对应的路径就是 prefix。比如你的操作系统的 boot 分区在第 1 块盘的第 3 个分区上，那么 prefix 就是 (hd0,gpt3)/grub。这就是网上经常说的默默操作系统去引导啥，比如说 Ubuntu 系统引导 windows，其实就是说启动点是 grub，prefix 是 Ubuntu 的 / boot/grub。

说句题外话，一个操作系统是不可能引导另一个操作系统的，只有引导器（BootLoader）如 grub 才能引导操作系统启动，或者 uefi 可以直接引导操作系统启动。并且，grub 还不能直接应道 dos（windows）启动，而是先引导 windows 系统的引导器（如 EasyBCD），再让这个引导器去引导 windows，这种方式成为链式引导（chainload），比如：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
menuentry 'Windows'{
insmod part_msdos
insmod ntfs
set root='(hd0,msdos1)'
chainloader +1
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

3. 是不是一定要挂载 esp 分区？

如果你确定不会在去动 grub 了，那么可以不挂载，efi 分区只在你操作引导器的时候才有用，平常它是没有用的。所以看到有些教程说把它挂载到 / efi 上，有的则是 / boot/efi 上，甚至有的直接说挂到 / boot 上这样方便直接启动。其实都可以，挂载到不同位置，只影响你 grub-install 的操作结果，操作过后，他就没用了。

4.linux 内核是如何确定文件挂载树的？我能不能把 / etc 也搞到一个单独的分区上？

有人会说有 / etc/fstab，对，没错，但是，内核启动的时候，它首先得知道 / etc 在那个盘对吧？不然它怎么知道到 fstab 呀。看来这就是个先有鸡还是先有蛋的问题，怎么解决呢？

如果你仔细看上面的 grub.cfg，你就会发现，在启动 Linux 内核的语句中有许多参数，比如这句： linux (loop0)/boot/x86_64/loader/linux root=live:CDLABEL=openSUSE_Leap_15.0_KDE_Live iso-scan/filename=/grub/linux/openSUSE-Leap-KDE-Live.iso systemd.wants=sshd.service lang=zh_CN ，有一个 root=xxx，这就是指定了 root 参数，所以就能知道 / 在哪，也就知道了 / etc 在哪。所以，一般来说你的 / etc 是不能挂载到单独的盘的，除非你自己编译一下内核，加入一个 etc=xxx 之内的参数，在运行 linux 命令的适合指定这个 etc 的盘，这样你就可以把 etc 单独搞到别的盘了，但一般没有人有这么蛋疼的需求。如果你一定要这么做，可以参考一下这个老哥《[Moving /etc to separate partition](https://unix.stackexchange.com/questions/77681/moving-etc-to-separate-partition)》。