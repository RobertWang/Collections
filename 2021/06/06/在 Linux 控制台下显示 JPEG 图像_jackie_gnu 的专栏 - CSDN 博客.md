> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/jackie_gnu/article/details/7033624?utm_medium=distribute.pc_relevant_download.none-task-blog-baidujs-2.nonecase&depth_1-utm_source=distribute.pc_relevant_download.none-task-blog-baidujs-2.nonecase)

在 Linux 控制台下显示 JPEG 图像

（陈云川

[ybc2084@163.com](mailto:ybc2084@163.com)

UESTC, 成都）

1、引言

通常情况下，在 Linux 控制台下是无法查看图像文件的，要想查看图像文件，比如要查看 JPEG 格式的图像文件，可能必须启动 X-Windows，通过 GNOME 或者 KDE 之类的桌面管理器提供的图像查看工具查看图片内容。那么，能不能有办法在控制台下面简单地浏览图像内容呢。实际上，这是完全可以的。在 Linux 下有一个名为 zgv 的看图软件就是工作在控制台下的。不过，由于它所使用的底层图形库 svgalib 已经是一个比较 “古老” 的图形库了，所以现在知道 zgv 的人并不是很多，用的人就更少了。

目前 Linux 上的底层图形支持通常是由 Framebuffer 提供的，因此，作者试图在本文中说明如何通过 Framebuffer 和 libjpeg 在控制台上显示 JPEG 图像。需要说明的是，本文中所编写的程序 fv 并非 zgv 的替代品，而只是一个出于验证想法的简单程序（fv 的含义是 Framebuffer Vision）。本文将先对 Framebuffer 和 libjpeg 的编程做一个简略的说明，然后再给出程序 fv 的具体实现。

2、Framebuffer 介绍

Framebuffer 在 Linux 中是作为设备来实现的，它是对图形硬件的一种抽象 [1]，代表着显卡中的帧缓冲区（Framebuffer）。通过 Framebuffer 设备，上层软件可以通过一个良好定义的软件接口访问图形硬件，而不需要关心底层图形硬件是如何工作的，比如，上层软件不用关心应该如何读写显卡寄存器，也不需要知道显卡中的帧缓冲区从什么地址开始，所有这些工作都由 Framebuffer 去处理，上层软件只需要集中精力在自己要做的事情上就是了。

Framebuffer 的优点在于它是一种低级的通用设备，而且能够跨平台工作，比如 Framebuffer 既可以工作在 x86 平台上，也能工作在 PPC 平台上，甚至也能工作在 m68k 和 SPARC 等平台上，在很多嵌入式设备上 Framebuffer 也能正常工作。诸如 Minigui 之类的 GUI 软件包也倾向于采用 Framebuffer 作为硬件抽象层（HAL）。

从用户的角度来看，Framebuffer 设备与其它设备并没有什么不同。Framebuffer 设备位于 / dev 下，通常设备名为 fb*，这里 * 的取值从 0 到 31。对于常见的计算机系统而言，32 个 Framebuffer 设备已经绰绰有余了（至少作者还没有看到过有 32 个监视器的计算机）。最常用到的 Framebuffer 设备是 / dev/fb0。通常，使用 Framebuffer 的程序通过环境变量 FRAMEBUFFER 来取得要使用的 Framebuffer 设备，环境变量 FRAMEBUFFER 通常被设置为”/dev/fb0”。

从程序员的角度来看，Framebuffer 设备其实就是一个文件而已，可以像对待普通文件那样读写 Framebuffer 设备文件，可以通过 mmap() 将其映射到内存中，也可以通过 ioctl() 读取或者设置其参数，等等。最常见的用法是将 Framebuffer 设备通过 mmap() 映射到内存中，这样可以大大提高 IO 效率。

要在 PC 平台上启用 Framebuffer，首先必须要内核支持，这通常需要重新编译内核。另外，还需要修改内核启动参数。在作者的系统上，为了启用 Framebuffer，需要将 / boot/grub/menu.lst 中的下面这一行：

kernel /boot/vmlinuz-2.4.20-8 ro root=LABEL=/1

修改为

kernel /boot/vmlinuz-2.4.20-8 ro root=LABEL=/1 vga=0x0314

即增加了 vga=0x0314 这样一个内核启动参数。这个内核启动参数表示的意思是：Framebuffer 设备的大小是 800x600，颜色深度是 16bits / 像素。

下面，来了解一下如何编程使用 Framebuffer 设备。由于对 Framebuffer 设备的读写应该是不缓冲的，但是标准 IO 库默认是要进行缓冲的，因此通常不使用标准 IO 库读写 Framebuffer 设备，而是直接通过 read()、write() 或者 mmap() 等系统调用来完成与 Framebuffer 有关的 IO 操作。又由于 mmap() 能够大大降低 IO 的开销，因此与 Framebuffer 设备有关的 IO 通常都是通过 mmap() 系统调用来完成的。mmap() 的函数原型如下（Linux 系统上的定义）：

#include <sys/mman.h>

#ifdef _POSIX_MAPPED_FILES

void  *  mmap(void *start, size_t length, int prot , int flags, int fd,

off_t offset);

int munmap(void *start, size_t length);

#endif

系统调用 mmap() 用来实现内存映射 IO。所谓内存映射 IO，是指将一个磁盘文件的内容与内存中的一个空间相映射。当从这个映射内存空间中取数据时，就相当于从文件中读取相应的字节，而当向此映射内存空间写入数据时，就相当于向磁盘文件中写入数据。这就是内存映射 IO 的含义。

具体到对 mmap() 而言，当调用成功时，返回值就是与磁盘文件建立了映射关系的内存空间的起始地址，当调用失败时，mmap() 的返回值是 - 1。第一个参数 start 通常设置为 0，表示由系统选择映射内存空间；第二个参数 length 指定了要映射的字节数；第三个参数指明了映射内存空间的保护属性，对于 Framebuffer 通常将其设置为 PROT_READ | PROT_WRITE，表示既可读也可写；第四个参数 flags 指明了影响映射内存空间行为的标志，对于 Framebuffer 编程而言，要将 flags 设置为 MAP_SHARED，表明当向映射内存空间写入数据时，将数据写入磁盘文件中；第五个参数 fd 是要映射的文件的文件描述符；第六个参数 offset 指明了要映射的字节在文件中的偏移量。

如果 mmap() 调用成功，就可以在程序中对得到的映射内存空间进行读写操作了。所有的读写都将由操作系统内核转换成 IO 操作。

在使用完映射内存空间之后，应当将其释放，这是通过 munmap() 系统调用完成的。munmap() 的第一个参数是映射内存空间的起始地址，第二个参数 length 是映射内存空间的长度，单位为字节。如果释放成功，munmap() 返回 0，否则返回 - 1。

如果应用程序需要知道 Framebuffer 设备的相关参数，必须通过 ioctl() 系统调用来完成。在头文件 < linux/fb.h> 中定义了所有的 ioctl 命令字，不过，最常用的 ioctl 命令字是下面这两个：FBIOGET_FSCREENINFO 和 FBIOGET_VSCREENINFO，前者返回与 Framebuffer 有关的固定的信息，比如图形硬件上实际的帧缓存空间的大小、能否硬件加速等信息；而后者返回的是与 Framebuffer 有关的可变信息，之所以可变，是因为对同样的图形硬件，可以工作在不同的模式下，简单来讲，一个支持 1024x768x24 图形模式的硬件通常也能工作在 800x600x16 的图形模式下，可变的信息就是指 Framebuffer 的长度、宽度以及颜色深度等信息。这两个命令字相关的结构体有两个：struct fb_fix_screeninfo 和 struct fb_var_screeninfo，这两个结构体都比较大，前者用于保存 Framebuffer 设备的固定信息，后者用于保存 Framebuffer 设备的可变信息。在调用 ioctl() 的时候，要用到这两个结构体。应用程序中通常要用到 struct fb_var_screeninfo 的下面这几个字段：xres、yres、bits_per_pixel，分别表示 x 轴的分辨率、y 轴的分辨率以及每像素的颜色深度（颜色深度的单位为 bit/pixel），其类型定义都是无符号 32 位整型数。

3、libjpeg 函数库介绍

JPEG 是 CCITT 和 ISO 定义的一种连续色调图像压缩标准 [2]。JPEG 是一种有损图像压缩标准，其基础是 DCT 变换（离散余弦变换）。JPEG 图像的压缩过程分为三步：DCT 计算，量化，变长编码分配。尽管 CCITT 定义了 JPEG 图像压缩标准，但是却并没有为 JPEG 定义标准的文件格式。这导致了现实世界中出现了各种各样的 JPEG 文件格式，而一种被称为 JFIF 的 JPEG 文件格式逐渐成为 JPEG 文件格式的主流。

libjpeg 是一个被广泛使用的 JPEG 压缩 / 解压缩函数库（至少在 Unix 类系统下是广泛使用的），它能够读写 JFIF 格式的 JPEG 图像文件，通常这类文件是以. jpg 或者. jpeg 为后缀名的。通过 libjpeg 库，应用程序可以每次从 JPEG 压缩图像中读取一个或多个扫描线（scanline，所谓扫描线，是指由一行像素点构成的一条图像线条），而诸如颜色空间转换、降采样 / 增采样、颜色量化之类的工作则都由 libjpeg 去完成了。

要使用 libjpeg，需要读者对数字图像的基本知识有初步的了解。对于 libjpeg 而言，图像数据是一个二维的像素矩阵。对于彩色图像，每个像素通常用三个分量表示，即 R（Red）、G（Green）、B（Blue）三个分量，每个分量用一个字节表示，因此每个分量的取值范围从 0 到 255；对于灰度图像，每个像素通常用一个分量表示，一个分量同样由一个字节表示，取值范围从 0 到 255。由于本文不会涉及到索引图像，因此这里略去对索引图像的说明。

在 libjpeg 中，图像数据是以扫描线的形式存放的。每一条扫描线由一行像素点构成，像素点沿着扫描线从左到右依次排列。对于彩色图像，每个分量由三个字节组成，因此这三个字节以 R、G、B 的顺序构成扫描线上的一个像素点。一个典型的扫描线形式如下：

R,G,B,R,G,B,R,G,B,…

通过 libjpeg 解压出来的图像数据也是以扫描线的形式存放的。

在本文中，只涉及到 JPEG 的解压缩，因此只对 libjpeg 的解压过程进行说明，有关 libjpeg 的压缩过程和其它高级用法，请参考 [3]。一般地，libjpeg 的解压过程如下：

1、分配并初始化一个 JPEG 解压对象（本文中将 JPEG 解压对象命名为 cinfo）：

struct jpeg_decompress_struct cinfo;

struct jpeg_error_mgr jerr;

...

cinfo.err = jpeg_std_error(&jerr);

jpeg_create_decompress(&cinfo);

2、指定要解压缩的图像文件：

FILE * infile;

...

if ((infile = fopen(filename, "rb")) == NULL) {

fprintf(stderr, "can't open %s/n", filename);

exit(1);

}

jpeg_stdio_src(&cinfo, infile);

3、调用 jpeg_read_header() 获取图像信息：

jpeg_read_header(&cinfo, TRUE);

4、这是一个可选步骤，用于设置 JPEG 解压缩对象 cinfo 的一些参数，本文可忽略；

5、调用 jpeg_start_decompress() 开始解压过程：

jpeg_start_decompress(&cinfo);

调用 jpeg_start_decompress() 函数之后，JPEG 解压缩对象 cinfo 中的下面这几个字段将会比较有用：

l output_width                这是图像输出的宽度

l output_height                这是图像输出的高度

l output_components              每个像素的分量数，也即字节数

这是因为在调用 jpeg_start_decompress() 之后往往需要为解压后的扫描线上的所有像素点分配存储空间，这个空间的大小可以通过 output_width * output_componets 确定，而要读取的扫描线的总数为 output_height 行。

6、读取一行或者多行扫描线数据并处理，通常的代码是这样的：

while (cinfo.output_scanline < cinfo.ouput_height) {

jpeg_read_scanlines();

/* deal with scanlines */

}

对扫描线的读取是按照从上到下的顺序进行的，也就是说图像最上方的扫描线最先被 jpeg_read_scanlines() 读入存储空间中，紧接着是第二个扫描线，最后是图像底边的扫描线被读入存储空间中。

7、调用 jpeg_finish_decompress() 完成解压过程：

jpeg_finish_decompress(&cinfo);

8、调用 jpeg_destroy_decompress() 释放 JPEG 解压对象 cinfo：

jpeg_destroy_decompress(&cinfo);

以上就是通过 libjpeg 函数解压 JPEG 压缩图像的基本过程，由于本文不涉及 libjpeg 的高级特性和用法，因此，上面的介绍对于说明本文中要用到的 libjpeg 的功能已经足够了。

另外一个需要说明地方是：由于作者所用的 Framebuffer 设备的颜色深度为 16 位，颜色格式为 5-6-5 格式——即 R（红色）在 16bit 中占据高 5 位，G（绿色）在 16bit 中占据中间 6 位，B（蓝色）在 16bit 中占据低 5 位；而 libjpeg 解压出来的图像数据为 24 位 RGB 格式，因此必须进行转换。对于 24 位的 RGB，每个字节表示一个颜色分量，因此转换的方式为：对于 R 字节，右移 3 位，对于 G 字节，右移 2 位，对于 B 字节，右移 3 位，然后将右移得到的值拼接起来，就得到了 16 位的颜色值。在后面的程序中，将把 24 位的颜色称为 RGB888，而把 16 位颜色值称为 RGB565，这种命名方式可能不太规范，不过无论如何，在本文中就这样称呼了。另外，读者可能会想到，上面这种直接将颜色分量的低位丢弃的方式不是会导致图像细节的丢失吗？比如，对于 24 位颜色的 R 字节，假如原来低 3 位的值在 0~7 之间均匀分布，转换之后，所有这低 3 位的值全部都变成了 0，这就是颜色细节的丢失。为了处理这个问题，可以采用误差扩散算法或者抖动算法来完成颜色转换。为了保持程序的简单，本文中仍然只采用上面提到的最简单的转换方式——而且事实表明，这样做得到的结果并不差。