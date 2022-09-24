> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/woainilixuhao/article/details/87919970)

转自：[https://www.jb51.net/article/36726.htm](https://www.jb51.net/article/36726.htm)

为了速度和正确性, 请对齐你的数据.

    概述: 对于所有直接操作[内存](https://so.csdn.net/so/search?q=%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)的程序员来说, 数据对齐都是很重要的问题. 数据对齐对你的程序的表现甚至能否正常运行都会产生影响. 就像本文章阐述的一样, 理解了对齐的本质还能够解释一些处理器的 "奇怪的" 行为.

**内存存取粒度**

   程序员通常倾向于认为内存就像一个字节. 在 C 及其衍生语言中, char * 用来指代 "一块内存", 甚至在 JAVA 中也有 byte[] 类型来指代物理内存.

![](https://img-blog.csdnimg.cn/20191108095838396.png)

   然而, 你的处理器并不是按字节块来存取内存的. 它一般会以双字节, 四字节, 8 字节, 16 字节甚至 32 字节为单位来存取内存. 我们将上述这些存取单位称为内存存取粒度.

![](https://img-blog.csdnimg.cn/2019110809585939.png)

   高层 (语言) 程序员认为的内存形态和处理器对内存的实际处理方式之间的差异产生了许多有趣的问题, 本文旨在阐述这些问题.

   如果你不理解内存对齐, 你编写的程序将有可能产生下面的问题, 按严重程度递增：

程序运行速度变慢

应用程序产生死锁

操作系统崩溃

你的程序会毫无征兆的出错, 产生错误的结果 (silently fail 如何翻译?)

**内存对齐基础**

   为了说明内存对齐背后的原理, 我们考察一个任务, 并观察内存存取粒度是如何对该任务产生影响的. 这个任务很简单: 先从地址 0 读取 4 个字节到寄存器, 然后从地址 1 读取 4 个字节到寄存器.

   首先考察内存存取粒度为 1byte 的情况:

![](https://img-blog.csdnimg.cn/20191108100006402.png)

   这迎合了那些天真的程序员的观点: 从地址 0 和地址 1 读取 4 字节数据都需要相同的 4 次操作. 现在再看看存取粒度为双字节的处理器 (像最初的 68000 处理器) 的情况:

![](https://img-blog.csdnimg.cn/20191108100029260.png)

   从地址 0 读取数据, 双字节存取粒度的处理器读内存的次数是单字节存取粒度处理器的一半. 因为每次内存存取都会产生一个固定的开销, 最小化内存存取次数将提升程序的性能.

   但从地址 1 读取数据时由于地址 1 没有和处理器的内存存取边界对齐, 处理器就会做一些额外的工作. 地址 1 这样的地址被称作非对齐地址. 由于地址 1 是非对齐的, 双字节存取粒度的处理器必须再读一次内存才能获取想要的 4 个字节, 这减缓了操作的速度.

数据结构（尤其是栈）应该尽可能地在自然边界上对齐。原因在于，为了访问未对齐的内存，处理器需要作两次内存访问；然而，对齐的内存访问仅需要一次访问，因为处理器读内存不会错过内存的自然边界。  
   最后我们再看一下存取粒度为 4 字节的处理器 (像 68030,PowerPC® 601) 的情况:

![](https://img-blog.csdnimg.cn/20191108100048392.png)

   在对齐的内存地址上, 四字节存取粒度处理器可以一次性的将 4 个字节全部读出; 而在非对齐的内存地址上, 读取次数将加倍.

   既然你理解了内存对齐背后的原理, 那么你就可以探索该领域相关的一些问题了.

**懒惰的处理器**

   处理器对非对齐内存的存取有一些技巧. 考虑上面的四字节存取粒度处理器从地址 1 读取 4 字节的情况, 你肯定想到了下面的解决方法:

![](https://img-blog.csdnimg.cn/20191108100128812.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvYWluaWxpeHVoYW8=,size_16,color_FFFFFF,t_70)

   处理器先从非对齐地址读取第一个 4 字节块, 剔除不想要的字节, 然后读取下一个 4 字节块, 同样剔除不要的数据, 最后留下的两块数据合并放入寄存器. 这需要做很多工作.

   有些处理器并不情愿为你做这些工作.

   最初的 68000 处理器的存取粒度是双字节, 没有应对非对齐内存地址的电路系统. 当遇到非对齐内存地址的存取时, 它将抛出一个异常. 最初的 Mac OS 并没有妥善处理这个异常, 它会直接要求用户重启机器. 悲剧.

   随后的 680x0 系列, 像 68020, 放宽了这个的限制, 支持了非对齐内存地址存取的相关操作. 这解释了为什么一些在 68020 上正常运行的旧软件会在 68000 上崩溃. 这也解释了为什么当时一些老 Mac 编程人员会将指针初始化成奇数地址. 在最初的 Mac 机器上如果指针在使用前没有被重新赋值成有效地址, Mac 会立即跳到调试器. 通常他们通过检查调用堆栈会找到问题所在.

   所有的处理器都使用有限的晶体管来完成工作. 支持非对齐内存地址的存取操作会消减 "晶体管预算", 这些晶体管原本可以用来提升其他模块的速度或者增加新的功能.

   以速度的名义牺牲非对齐内存存取功能的一个例子就是 MIPS. 为了提升速度, MIPS 几乎废除了所有的琐碎功能.

    PowerPC 各取所长. 目前所有的 PowPC 都硬件支持非对齐的 32 位整型的存取. 虽然牺牲掉了一部分性能, 但这些损失在逐渐减少.

   另一方面, 现今的 PowPC 处理器缺少对非对齐的 64-bit 浮点型数据的存取的硬件支持. 当被要求从非对齐内存读取浮点数时, PowerPC 会抛出异常并让操作系统来处理内存对齐这样的杂事. 软件解决内存对齐要比硬件慢得多.  
**psting 1. 每次处理一个字节**

代码如下:

  
void Munge8(void *data, uint32_t size){  
    uint8_t *data8 = (uint8_t*)data;  
    uint8_t *data8End = data8 +size;  
    while(data8 != data8End){  
        *data8++ = -*data8;  
    }  
}

  
   运行这个函数需要 67364 微秒, 现在修改成每次处理 2 个字节, 这将使存取次数减半:

**psting 2. 每次处理 2 个字节**

代码如下:

  
void Munge16(void *data, uint32_t size){  
    uint16_t *data16 = (uint16_t*)data;  
    uint16_t *data16End = data16 + (size>> 1); /* Divide size by 2. */  
    uint8_t *data8 = (uint8_t*)data16End;  
    uint8_t *data8End = data8 + (size& 0x00000001); /* Strip upper 31 bits. */  
    while(data16 != data16End){  
        *data16++ = -*data16;  
    }  
    while(data8 != data8End){  
        *data8++ = -*data8;  
    }  
}

  
   如果处理的内存地址是对齐的话, 上述函数处理同一个缓冲区需要 48765 微秒 -- 比 Munge8 快 38%. 如果缓冲区不是对齐的, 处理时间会增加到 66385 微秒 -- 比对齐情况下慢了 27%. 下图展示了对齐内存和非对齐内存之间的性能对比.

**速度**

   下面编写一些测试来说明非对齐内存对性能造成的损失. 过程很简单: 从一个 10MB 的缓冲区中读取, 取反, 并写回数据. 这些测试有两个变量:

处理缓冲区的处理粒度, 单位 bytes. 一开始每次处理 1 个字节, 然后 2 个字节, 4 个字节和 8 个字节.

缓冲区的对准. 用每次增加缓冲区的指针来交错调整内存地址, 然后重新做每个测试.

   这些测试运行在 800MHz 的 PowerBook G4 上. 为了最小化中断引起的波动, 这里取十次结果的平均值. 第一个是处理粒度为单字节的情况:  
 

**psting 1. 每次处理一个字节**

复制代码代码如下:

  
void Munge8(void *data, uint32_t size){  
    uint8_t *data8 = (uint8_t*)data;  
    uint8_t *data8End = data8 +size;  
    while(data8 != data8End){  
        *data8++ = -*data8;  
    }  
}

  
   运行这个函数需要 67364 微秒, 现在修改成每次处理 2 个字节, 这将使存取次数减半:

**psting 2. 每次处理 2 个字节**

复制代码代码如下:

  
void Munge16(void *data, uint32_t size){  
    uint16_t *data16 = (uint16_t*)data;  
    uint16_t *data16End = data16 + (size>> 1); /* Divide size by 2. */  
    uint8_t *data8 = (uint8_t*)data16End;  
    uint8_t *data8End = data8 + (size& 0x00000001); /* Strip upper 31 bits. */  
    while(data16 != data16End){  
        *data16++ = -*data16;  
    }  
    while(data8 != data8End){  
        *data8++ = -*data8;  
    }  
}

  
   如果处理的内存地址是对齐的话, 上述函数处理同一个缓冲区需要 48765 微秒 -- 比 Munge8 快 38%. 如果缓冲区不是对齐的, 处理时间会增加到 66385 微秒 -- 比对齐情况下慢了 27%. 下图展示了对齐内存和非对齐内存之间的性能对比.

![](https://img-blog.csdnimg.cn/20191108100531320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvYWluaWxpeHVoYW8=,size_16,color_FFFFFF,t_70)

   第一个让人注意到的现象是单字节存取结果很均匀, 且都很慢. 第二个是双字节存取时, 每当地址是单数时, 变慢的 27% 就会出现.

   下面加大赌注, 每次处理 4 个字节:  
**psting 3. 每次处理 4 个字节**

代码如下:

  
void Munge32(void *data, uint32_t size){  
    uint32_t *data32 = (uint32_t*)data;  
    uint32_t *data32End = data32 + (size>> 2); /* Divide size by 4. */  
    uint8_t *data8 = (uint8_t*)data32End;  
    uint8_t *data8End = data8 + (size& 0x00000003); /* Strip upper 30 bits. */  
    while(data32 != data32End){  
        *data32++ = -*data32;  
    }  
    while(data8 != data8End){  
        *data8++ = -*data8;  
    }  
}

   对于对齐的缓冲区, 函数需要 43043 微秒; 对于非对齐的缓冲区, 函数需要 55775 微秒. 因此, 在所测试的机器上, 非对齐地址的四字节存取速度比对齐地址的双字节存取速度要慢.

![](https://img-blog.csdnimg.cn/2019110810050350.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvYWluaWxpeHVoYW8=,size_16,color_FFFFFF,t_70)

现在来最恐怖的: 每次处理 8 个字节:  
**psting 4. 每次处理 8 个字节**

复制代码代码如下:

  
void Munge64(void *data, uint32_t size){  
    double *data64 = (double*)data;  
    double *data64End = data64 + (size>> 3); /* Divide size by 8. */  
    uint8_t *data8 = (uint8_t*)data64End;  
    uint8_t *data8End = data8 + (size& 0x00000007); /* Strip upper 29 bits. */  
    while(data64 != data64End){  
        *data64++ = -*data64;  
    }  
    while(data8 != data8End){  
        *data8++ = -*data8;  
    }  
}

    Munge64 处理对齐的缓冲区需要 39085 微秒 -- 大约比对齐的 Munge32 快 10%. 但是, 在非对齐缓冲区上的处理时间是让人惊讶的 1841155 微秒 -- 比对齐的慢了两个数量级, 慢了足足 4610%.

   怎么回事? 因为我们现今所使用的 PowerPC 缺少对存取非对齐内存的浮点数的硬件支持. 对每次非对齐内存的存取, 处理器都抛出一个异常. 操作系统获取该异常并软件实现内存对齐. 下图显示了非对齐内存存取带来的不利后果.

![](https://img-blog.csdnimg.cn/20191108100246327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvYWluaWxpeHVoYW8=,size_16,color_FFFFFF,t_70)

   单字节, 双字节和四字节的细节都被掩盖了. 或许去除顶部以后的图形, 如下图, 更清晰:

![](https://img-blog.csdnimg.cn/20191108100342719.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvYWluaWxpeHVoYW8=,size_16,color_FFFFFF,t_70)

   在这些数据背后还隐藏着一个微妙的现象. 比较 8 字节粒度时边界是 4 的倍数的内存的存取速度:

![](https://img-blog.csdnimg.cn/20191108100400951.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvYWluaWxpeHVoYW8=,size_16,color_FFFFFF,t_70)

   你会发现 8 字节粒度时边界为 4 和 12 字节的内存存取速度要比相同情况下的 4 和 2 字节粒度的慢. 即使 PowerPC 硬件支持 4 字节对齐的 8 字节双浮点型数据的存取, 你还是要承担额外的开销造成的损失. 诚然, 这种损失绝不会像 4610% 那么大, 但还是不能忽略的. 这个实验告诉我们: 存取非对齐内存时，大粒度的存取可能会比小粒度存取还要慢。