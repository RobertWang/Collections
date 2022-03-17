> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651168422&idx=1&sn=9c467edbeca10b29ba983c24fdc08f29&chksm=80644ff9b713c6ef8a8ffd8908e24538791544af0b1f5a8a7fb69a25ac89432196bed482a0fa&scene=21#wechat_redirect)

在逛知乎的时候，看到篇帖子，如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9AAGq6Vqp50R6onibERQV75sBiaIyuI8LEOUVNWQicQLJs99FOuVcVHTdw/640?wx_fmt=png)

看了下面所有的回答，要么是没有回答到点上，要么是回答不够深入，所以，借助本文，深入讲解 C/C++ 内存管理。

1 写在前面
------

源码分析本身就很枯燥乏味，尤其是要将其写成通俗易懂的文章，更是难上加难。

本文尽可能的从读者角度去进行分析，重点写大家关心的点，必要的时候，会贴出部分源码，以加深大家的理解，尽可能的通过本文，让大家理解内存分配释放的本质原理。

接下来的内容，干货满满，对于你我都是一次收获的过程。主要从内存布局、glibc 内存管理、malloc 实现以及 free 实现几个点来带你领略 glibc 内存管理精髓。最后，针对项目中的问题，指出了解决方案。大纲内容如下：

‍‍‍‍‍‍![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9dZ1JJj2DFWTnGU48hgevRDBDnwvdX3EKdaE731pb9WuOZ4AQFhpQXQ/640?wx_fmt=png)主要内容

2 背景
----

几年前，在上家公司做了一个项目，暂且称之为 SeedService 吧。SeedService 从 kafka 获取 feed 流的转、评、赞信息，加载到内存。因为每天数据不一样，所以 kafka 的 topic 也是按天来切分，整个 server 内部，类似于双指针实现，当天 0 点释放头一天的数据。

项目上线，一切运行正常。

但是几天之后，进程开始无缘无故的消失。开始定位问题，最终发现是因为内存暴增导致 OOM，最终被操作系统 kill 掉。

弄清楚了进程消失的原因之后，开始着手分析内存泄漏。在解决了几个内存泄露的潜在问题以后，发现系统在高压力高并发环境下长时间运行仍然会发生内存暴增的现象，最终进程因 OOM 被操作系统杀掉。

由于内存管理不外乎三个层面，用户管理层，C 运行时库层，操作系统层，在操作系统层发现进程的内存暴增，同时又确认了用户管理层没有内存泄露，因此怀疑是 C 运行时库的问题，也就是 Glibc 的内存管理方式导致了进程的内存暴增。

问题缩小到 glibc 的内存管理方面，把下面几个问题弄清楚，才能解决 SeedService 进程消失的问题：

*   glibc 在什么情况下不会将内存归还给操作系统？
    
*   glibc 的内存管理方式有哪些约束? 适合什么样的内存分配场景？
    
*   我们的系统中的内存管理方式是与 glibc 的内存管理的约束相悖的？
    
*   glibc 是如何管理内存的？
    

带着上面这些问题，大概用了将近一个月的时间分析了 glibc 运行时库的内存管理代码，今天将当时的笔记整理了出来，希望能够对大家有用。

3 基础
----

Linux 系统在装载 elf 格式的程序文件时，会调用 loader 把可执行文件中的各个段依次载入到从某一地址开始的空间中。

用户程序可以直接使用系统调用来管理 heap 和 mmap 映射区域，但更多的时候程序都是使用 C 语言提供的 malloc() 和 free() 函数来动态的分配和释放内存。stack 区域是唯一不需要映射，用户却可以访问的内存区域，这也是利用堆栈溢出进行攻击的基础。

### 3.1 进程内存布局

计算机系统分为 32 位和 64 位，而 32 位和 64 位的进程布局是不一样的，即使是同为 32 位系统，其布局依赖于内核版本，也是不同的。

在介绍详细的内存布局之前，我们先描述几个概念：

*   栈区（Stack）— 存储程序执行期间的本地变量和函数的参数，从高地址向低地址生长
    
*   堆区（Heap）动态内存分配区域，通过 malloc、new、free 和 delete 等函数管理
    
*   未初始化变量区（BSS）— 存储未被初始化的全局变量和静态变量
    
*   数据区（Data）— 存储在源代码中有预定义值的全局变量和静态变量
    
*   代码区（Text）— 存储只读的程序执行代码，即机器指令
    

#### 3.1.1 32 位进程内存布局

###### 经典布局

在 Linux 内核 2.6.7 以前，进程内存布局如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9nqflj6Y23PPeF8RdZkk2ruAcZGS8BXM9vo59UpVgrGos5Or1asvic8w/640?wx_fmt=png)32 位经典布局

在该内存布局示例图中，mmap 区域与栈区域相对增长，这意味着堆只有 1GB 的虚拟地址空间可以使用，继续增长就会进入 mmap 映射区域， 这显然不是我们想要的。这是由于 32 模式地址空间限制造成的，所以内核引入了另一种虚拟地址空间的布局形式。但对于 64 位系统，因为提供了巨大的虚拟地址空间，所以 64 位系统就采用的这种布局方式。

###### 默认布局

如上所示，由于经典内存布局具有空间局限性，因此在内核 2.6.7 以后，就引入了下图这种默认进程布局方式。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9PPOxmlUEuRFcXXw6nK7X28rfDUw21J43APpr7J73OrD29fDic3XPBmA/640?wx_fmt=png)32 位默认布局

从上图可以看到，栈至顶向下扩展，并且栈是有界的。堆至底向上扩展，mmap 映射区域至顶向下扩展，mmap 映射区域和堆相对扩展，直至耗尽虚拟地址空间中的剩余区域，这种结构便于 C 运行时库使用 mmap 映射区域和堆进行内存分配。

#### 3.1.2 64 位进程内存布局

如之前所述，64 位进程内存布局方式由于其地址空间足够，且实现方便，所以采用的与 32 位经典内存布局的方式一致，如下图所示:

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9LicOauqXria0Zvq9kK3no1N0iaic9ibAOxHPvW4xDQUbQJew2m6eatjibSibQ/640?wx_fmt=png)64 位进程布局

### 3.2 操作系统内存分配函数

在之前介绍内存布局的时候，有提到过，heap 和 mmap 映射区域是可以提供给用户程序使用的虚拟内存空间。那么我们该如何获得该区域的内存呢？

操作系统提供了相关的系统调用来完成内存分配工作。

*   对于 heap 的操作，操作系统提供了 brk() 函数，c 运行时库提供了 sbrk() 函数。
    
*   对于 mmap 映射区域的操作，操作系统提供了 mmap() 和 munmap() 函数。
    

sbrk()，brk() 或者 mmap() 都可以用来向我们的进程添加额外的虚拟内存。而 glibc 就是使用这些函数来向操作系统申请虚拟内存，以完成内存分配的。

> ❝
> 
> 这里要提到一个很重要的概念，内存的延迟分配，只有在真正访问一个地址的时候才建立这个地址的物理映射，这是 Linux 内存管理的基本思想之一。Linux 内核在用户申请内存的时候，只是给它分配了一个线性区（也就是虚拟内存），并没有分配实际物理内存；只有当用户使用这块内存的时候，内核才会分配具体的物理页面给用户，这时候才占用宝贵的物理内存。内核释放物理页面是通过释放线性区，找到其所对应的物理页面，将其全部释放的过程。
> 
> ❞

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9aicSVGvPVLzClpX6qLJDm0sLpJc4Wjl2nJiavMJ1HZ9xsia2RVMltnKbg/640?wx_fmt=png)内存分配

进程的内存结构，在内核中，是用 mm_struct 来表示的，其定义如下：

```
struct mm_struct {
 ...
 unsigned long (*get_unmapped_area) (struct file *filp,
 unsigned long addr, unsigned long len,
 unsigned long pgoff, unsigned long flags);
 ...
 unsigned long mmap_base; /* base of mmap area */
 unsigned long task_size; /* size of task vm space */
 ...
 unsigned long start_code, end_code, start_data, end_data;
 unsigned long start_brk, brk, start_stack;
 unsigned long arg_start, arg_end, env_start, env_end;
 ...
}
```

在上述 mm_struct 结构中：

*   [start_code,end_code) 表示代码段的地址空间范围。
    
*   [start_data,end_start) 表示数据段的地址空间范围。
    
*   [start_brk,brk) 分别表示 heap 段的起始空间和当前的 heap 指针。
    
*   [start_stack,end_stack) 表示 stack 段的地址空间范围。
    
*   mmap_base 表示 memory mapping 段的起始地址。
    

C 语言的动态内存分配基本函数是 malloc()，在 Linux 上的实现是通过内核的 brk 系统调用。brk() 是一个非常简单的系统调用， 只是简单地改变 mm_struct 结构的成员变量 brk 的值。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9f5Ap2I19JsBy0mIjKyKWNOGe5Gh67HdvKjSlKgWsTGNS7L2chB51SQ/640?wx_fmt=png)mm_struct

#### 3.2.1 Heap 操作

在前面有提过，有两个函数可以直接从堆 (Heap) 申请内存，brk()函数为系统调用，sbrk()为 c 库函数。

系统调用通常提过一种最小的功能，而库函数相比系统调用，则提供了更复杂的功能。在 glibc 中，malloc 就是调用 sbrk() 函数将数据段的下界移动以来代表内存的分配和释放。sbrk() 函数在内核的管理下，将虚拟地址空间映射到内存，供 malloc() 函数使用。

下面为 brk() 函数和 sbrk() 函数的声明。

```
#include <unistd.h>
int brk(void *addr);

void *sbrk(intptr_t increment);
```

> ❝
> 
> 需要说明的是，当 sbrk() 的参数 increment 为 0 时候，sbrk() 返回的是进程当前 brk 值。increment 为正数时扩展 brk 值，当 increment 为负值时收缩 brk 值。
> 
> ❞

#### 3.2.2 MMap 操作

在 LINUX 中我们可以使用 mmap 用来在进程虚拟内存地址空间中分配地址空间，创建和物理内存的映射关系。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9Xuico2o1NkHt9wDVqBGw0oE5hKK9vEic1Ou6E83hYC2ZZHGjNh3H1WkA/640?wx_fmt=png)共享内存

mmap() 函数将一个文件或者其它对象映射进内存。文件被映射到多个页上，如果文件的大小不是所有页的大小之和，最后一个页不被使用的空间将会清零。

munmap 执行相反的操作，删除特定地址区域的对象映射。

函数的定义如下：

```
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset); 

int munmap(void *addr, size_t length);
```

· 映射关系分为以下两种：

*   文件映射: 磁盘文件映射进程的虚拟地址空间，使用文件内容初始化物理内存。
    
*   匿名映射: 初始化全为 0 的内存空间
    

映射关系是否共享，可以分为:

*   私有映射 (MAP_PRIVATE)
    

*   多进程间数据共享，修改不反应到磁盘实际文件，是一个 copy-on-write（写时复制）的映射方式。
    

*   共享映射 (MAP_SHARED)
    

*   多进程间数据共享，修改反应到磁盘实际文件中。
    

因此，整个映射关系总结起来分为以下四种:

*   私有文件映射多个进程使用同样的物理内存页进行初始化，但是各个进程对内存文件的修改不会共享，也不会反应到物理文件中
    
*   私有匿名映射
    

*   mmap 会创建一个新的映射，各个进程不共享，这种使用主要用于分配内存 (malloc 分配大内存会调用 mmap)。例如开辟新进程时，会为每个进程分配虚拟的地址空间，这些虚拟地址映射的物理内存空间各个进程间读的时候共享，写的时候会 copy-on-write。
    

*   共享文件映射
    

*   多个进程通过虚拟内存技术共享同样的物理内存空间，对内存文件的修改会反应到实际物理文件中，也是进程间通信 (IPC) 的一种机制。
    

*   共享匿名映射
    

*   这种机制在进行 fork 的时候不会采用写时复制，父子进程完全共享同样的物理内存页，这也就实现了父子进程通信 (IPC)。
    

这里值得注意的是，mmap 只是在虚拟内存分配了地址空间，只有在第一次访问虚拟内存的时候才分配物理内存。

在 mmap 之后，并没有在将文件内容加载到物理页上，只有在虚拟内存中分配了地址空间。当进程在访问这段地址时，通过查找页表，发现虚拟内存对应的页没有在物理内存中缓存，则产生 "缺页"，由内核的缺页异常处理程序处理，将文件对应内容，以页为单位 (4096) 加载到物理内存，注意是只加载缺页，但也会受操作系统一些调度策略影响，加载的比所需的多。

> ❝
> 
> 下面的内容将是本文的重点中的重点，对于了解内存布局以及后面 glibc 的内存分配原理至关重要，必要的话，可以多阅读几次。
> 
> ❞

4 概述
----

在前面，我们有提到在堆上分配内存有两个函数，分别为 brk() 系统调用和 sbrk()c 运行时库函数，在内存映射区分配内存有 mmap 函数。

现在我们假设一种情况，如果每次分配，都直接使用 brk(),sbrk() 或者 mmap() 函数进行多次内存分配。如果程序频繁的进行内存分配和释放，都是和操作系统直接打交道，那么性能可想而知。

这就引入了一个概念，「内存管理」。

本节大纲如下：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9DA9KWZuKpYnocamicGtfz3pibrczAJjXBUBWvjsk03sbcCbKtKcro5oQ/640?wx_fmt=png)

### 4.1 内存管理

内存管理是指软件运行时对计算机内存资源的分配和使用的技术。其最主要的目的是如何高效，快速的分配，并且在适当的时候释放和回收内存资源。

一个好的内存管理器，需要具有以下特点：1、跨平台、可移植通常情况下，内存管理器向操作系统申请内存，然后进行再次分配。所以，针对不同的操作系统，内存管理器就需要支持操作系统兼容，让使用者在跨平台的操作上没有区别。

2、浪费空间小内存管理器管理内存，如果内存浪费比较大，那么显然这就不是一个优秀的内存管理器。通常说的内存碎片，就是浪费空间的罪魁祸首，若内存管理器中有大量的内存碎片，它们是一些不连续的小块内存，它们总量可能很大，但无法使用，这显然也不是一个优秀的内存管理器。

3、速度快之所以使用内存管理器，根本原因就是为了分配 / 释放快。

4、调试功能作为一个 C/C++ 程序员，内存错误可以说是我们的噩梦，上一次的内存错误一定还让你记忆犹新。内存管理器提供的调试功能，强大易用，特别对于嵌入式环境来说，内存错误检测工具缺乏，内存管理器提供的调试功能就更是不可或缺了。

### 4.2 管理方式

内存管理的管理方式，分为 手动管理 和 自动管理 两种。

所谓的手动管理，就是使用者在申请内存的时候使用 malloc 等函数进行申请，在需要释放的时候，需要调用 free 函数进行释放。一旦用过的内存没有释放，就会造成内存泄漏，占用更多的系统内存；如果在使用结束前释放，会导致危险的悬挂指针，其他对象指向的内存已经被系统回收或者重新使用。

自动管理内存由编程语言的内存管理系统自动管理，在大多数情况下不需要使用者的参与，能够自动释放不再使用的内存。

#### 4.2.1 手动管理

手动管理内存是一种比较传统的内存管理方式，C/C++ 这类系统级的编程语言不包含狭义上的自动内存管理机制，使用者需要主动申请或者释放内存。经验丰富的工程师能够精准的确定内存的分配和释放时机，人肉的内存管理策略只要做到足够精准，使用手动管理内存的方式可以提高程序的运行性能，也不会造成内存安全问题。

但是，毕竟这种经验丰富且能精准确定内存和分配释放实际的使用者还是比较少的，只要是人工处理，总会带来一些错误，内存泄漏和悬挂指针基本是 C/C++ 这类语言中最常出现的错误，手动的内存管理也会占用工程师的大量精力，很多时候都需要思考对象应该分配到栈上还是堆上以及堆上的内存应该何时释放，维护成本相对来说还是比较高的，这也是必然要做的权衡。

#### 4.2.2 自动管理

自动管理内存基本是现代编程语言的标配，因为内存管理模块的功能非常确定，所以我们可以在编程语言的编译期或者运行时中引入自动的内存管理方式，最常见的自动内存管理机制就是垃圾回收，不过除了垃圾回收之外，一些编程语言也会使用自动引用计数辅助内存的管理。

自动的内存管理机制可以帮助工程师节省大量的与内存打交道的时间，让使用者将全部的精力都放在核心的业务逻辑上，提高开发的效率；在一般情况下，这种自动的内存管理机制都可以很好地解决内存泄漏和悬挂指针的问题，但是这也会带来额外开销并影响语言的运行时性能。

### 4.1 常见的内存管理器

1 ptmalloc：ptmalloc 是隶属于 glibc(GNU Libc)的一款内存分配器，现在在 Linux 环境上，我们使用的运行库的内存分配 (malloc/new) 和释放 (free/delete) 就是由其提供。

2 BSD Malloc：BSD Malloc 是随 4.2 BSD 发行的实现，包含在 FreeBSD 之中，这个分配程序可以从预先确实大小的对象构成的池中分配对象。它有一些用于对象大小的 size 类，这些对象的大小为 2 的若干次幂减去某一常数。所以，如果您请求给定大小的一个对象，它就简单地分配一个与之匹配的 size 类。这样就提供了一个快速的实现，但是可能会浪费内存。

3 Hoard：编写 Hoard 的目标是使内存分配在多线程环境中进行得非常快。因此，它的构造以锁的使用为中心，从而使所有进程不必等待分配内存。它可以显著地加快那些进行很多分配和回收的多线程进程的速度。

4 TCMalloc：Google 开发的内存分配器，在不少项目中都有使用，例如在 Golang 中就使用了类似的算法进行内存分配。它具有现代化内存分配器的基本特征：对抗内存碎片、在多核处理器能够 scale。据称，它的内存分配速度是 glibc2.3 中实现的 malloc 的数倍。

5 glibc 之内存管理 (ptmalloc)
------------------------

因为本次事故就是用的运行库函数 new/delete 进行的内存分配和释放，所以本文将着重分析 glibc 下的内存分配库 ptmalloc。

本节大纲如下：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9fNWPEzI0y6B6M7zqN3CfSuE9YWiaocibjq4U7gY8mdo8uCm0qAIH3jhg/640?wx_fmt=png)

在 c/c++ 中，我们分配内存是在堆上进行分配，那么这个堆，在 glibc 中是怎么表示的呢？

我们先看下堆的结构声明:

```
typedef struct _heap_info
{
  mstate ar_ptr;            /* Arena for this heap. */
  struct _heap_info *prev;  /* Previous heap. */
  size_t size;              /* Current size in bytes. */
  size_t mprotect_size;     /* Size in bytes that has been mprotected
                             PROT_READ|PROT_WRITE.  */
  /* Make sure the following data is properly aligned, particularly
     that sizeof (heap_info) + 2 * SIZE_SZ is a multiple of
     MALLOC_ALIGNMENT. */
  char pad[-6 * SIZE_SZ & MALLOC_ALIGN_MASK];
```

在堆的上述定义中，ar_ptr 是指向分配区的指针，堆之间是以链表方式进行连接，后面我会详细讲述进程布局下，堆的结构表示图。

在开始这部分之前，我们先了解下一些概念。

### 5.1 分配区 (arena)

> ❝
> 
> ptmalloc 对进程内存是通过一个个 Arena 来进行管理的。
> 
> ❞

在 ptmalloc 中，分配区分为主分配区 (arena) 和非主分配区(narena)，分配区用 struct malloc_state 来表示。主分配区和非主分配区的区别是 主分配区可以使用 sbrk 和 mmap 向 os 申请内存，而非分配区只能通过 mmap 向 os 申请内存。

当一个线程调用 malloc 申请内存时，该线程先查看线程私有变量中是否已经存在一个分配区。如果存在，则对该分配区加锁，加锁成功的话就用该分配区进行内存分配；失败的话则搜索环形链表找一个未加锁的分配区。如果所有分配区都已经加锁，那么 malloc 会开辟一个新的分配区加入环形链表并加锁，用它来分配内存。释放操作同样需要获得锁才能进行。

需要注意的是，非主分配区是通过 mmap 向 os 申请内存，一次申请 64MB，一旦申请了，该分配区就不会被释放，为了避免资源浪费，ptmalloc 对分配区是有个数限制的。

> ❝
> 
> 对于 32 位系统，分配区最大个数 = 2 * CPU 核数 + 1
> 
> 对于 64 位系统，分配区最大个数 = 8 * CPU 核数 + 1
> 
> ❞

堆管理结构：

```
struct malloc_state {
 mutex_t mutex;                 /* Serialize access. */
 int flags;                       /* Flags (formerly in max_fast). */
 #if THREAD_STATS
 /* Statistics for locking. Only used if THREAD_STATS is defined. */
 long stat_lock_direct, stat_lock_loop, stat_lock_wait;
 #endif
 mfastbinptr fastbins[NFASTBINS];    /* Fastbins */
 mchunkptr top;
 mchunkptr last_remainder;
 mchunkptr bins[NBINS * 2];
 unsigned int binmap[BINMAPSIZE];   /* Bitmap of bins */
 struct malloc_state *next;           /* Linked list */
 INTERNAL_SIZE_T system_mem;
 INTERNAL_SIZE_T max_system_mem;
 };
```

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9mGuflfOSmC6ickDC0MKtUBBbIMaVn5kzZCBUx578QXgr1znHj6JDIUA/640?wx_fmt=png)malloc_state

每一个进程只有一个主分配区和若干个非主分配区。主分配区由 main 线程或者第一个线程来创建持有。主分配区和非主分配区用环形链表连接起来。分配区内有一个变量 mutex 以支持多线程访问。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9yBFe7bLBKmZDslerCrbdkySmtJ5WZOtxtSpgBIo14DsPtL81Xp4Iyw/640?wx_fmt=png)环形链表链接的分配区

在前面有提到，在每个分配区中都有一个变量 mutex 来支持多线程访问。每个线程一定对应一个分配区，但是一个分配区可以给多个线程使用，同时一个分配区可以由一个或者多个的堆组成，同一个分配区下的堆以链表方式进行连接，它们之间的关系如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9hl6X66GtgZgWsXae7ONWN26XshibcUn3EPdZSN6DbGO2XOGbPT9phQw/640?wx_fmt=png)线程 - 分配区 - 堆

一个进程的动态内存，由分配区管理，一个进程内有多个分配区，一个分配区有多个堆，这就组成了复杂的进程内存管理结构。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX95kv8ZqdhPTfZyByqTQyUpAibWJnRBFWrXQGPibPYW5by4fSSiaA729bzQ/640?wx_fmt=png)

需要注意几个点：

*   主分配区通过 brk 进行分配，非主分配区通过 mmap 进行分配
    
*   非主分配区虽然是 mmap 分配，但是和大于 128K 直接使用 mmap 分配没有任何联系。大于 128K 的内存使用 mmap 分配，使用完之后直接用 ummap 还给系统
    
*   每个线程在 malloc 会先获取一个 area，使用 area 内存池分配自己的内存，这里存在竞争问题
    
*   为了避免竞争，我们可以使用线程局部存储，thread cache（tcmalloc 中的 tc 正是此意），线程局部存储对 area 的改进原理如下：
    
*   如果需要在一个线程内部的各个函数调用都能访问、但其它线程不能访问的变量（被称为 static memory local to a thread 线程局部静态变量），就需要新的机制来实现。这就是 TLS。
    
*   thread cache 本质上是在 static 区为每一个 thread 开辟一个独有的空间，因为独有，不再有竞争
    
*   每次 malloc 时，先去线程局部存储空间中找 area，用 thread cache 中的 area 分配存在 thread area 中的 chunk。当不够时才去找堆区的 area。
    

### 5.2 chunk

ptmalloc 通过 malloc_chunk 来管理内存，给 User data 前存储了一些信息，使用边界标记区分各个 chunk。

chunk 定义如下:

```
struct malloc_chunk {  
  INTERNAL_SIZE_T      prev_size;    /* Size of previous chunk (if free).  */  
  INTERNAL_SIZE_T      size;         /* Size in bytes, including overhead. */  
  
  struct malloc_chunk* fd;           /* double links -- used only if free. */  
  struct malloc_chunk* bk;  
  
  /* Only used for large blocks: pointer to next larger size.  */  
  struct malloc_chunk* fd_nextsize;      /* double links -- used only if free. */  
  struct malloc_chunk* bk_nextsize; 
};  
```

*   prev_size: 如果前一个 chunk 是空闲的，则该域表示前一个 chunk 的大小，如果前一个 chunk 不空闲，该域无意义。
    

一段连续的内存被分成多个 chunk，prev_size 记录的就是相邻的前一个 chunk 的 size，知道当前 chunk 的地址，减去 prev_size 便是前一个 chunk 的地址。prev_size 主要用于相邻空闲 chunk 的合并。

*   size ：当前 chunk 的大小，并且记录了当前 chunk 和前一个 chunk 的一些属性，包括前一个 chunk 是否在使用中，当前 chunk 是否是通过 mmap 获得的内存，当前 chunk 是否属于非主分配区。
    
*   fd 和 bk ：指针 fd 和 bk 只有当该 chunk 块空闲时才存在，其作用是用于将对应的空闲 chunk 块加入到空闲 chunk 块链表中统一管理，如果该 chunk 块被分配给应用程序使用，那么这两个指针也就没有用（该 chunk 块已经从空闲链中拆出）了，所以也当作应用程序的使用空间，而不至于浪费。
    
*   fd_nextsize 和 bk_nextsize: 当前的 chunk 存在于 large bins 中时， large bins 中的空闲 chunk 是按照大小排序的，但同一个大小的 chunk 可能有多个，增加了这两个字段可以加快遍历空闲 chunk ，并查找满足需要的空闲 chunk ， fd_nextsize 指向下一个比当前 chunk 大小大的第一个空闲 chunk ， bk_nextszie 指向前一个比当前 chunk 大小小的第一个空闲 chunk 。（同一大小的 chunk 可能有多块，在总体大小有序的情况下，要想找到下一个比自己大或小的 chunk，需要遍历所有相同的 chunk，所以才有 fd_nextsize 和 bk_nextsize 这种设计） 如果该 chunk 块被分配给应用程序使用，那么这两个指针也就没有用（该 chunk 块已经从 size 链中拆出）了，所以也当作应用程序的使用空间，而不至于浪费。
    

正如上面所描述，在 ptmalloc 中，为了尽可能的节省内存，使用中的 chunk 和未使用的 chunk 在结构上是不一样的。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9BkTHFsCNMVzqWlRKurA3VquhYrgzcYFR7RtcyIUR3hJmlXibZmhUu9A/640?wx_fmt=png)在上图中：

*   chunk 指针指向 chunk 开始的地址
    
*   mem 指针指向用户内存块开始的地址。
    
*   p=0 时，表示前一个 chunk 为空闲，prev_size 才有效
    
*   p=1 时，表示前一个 chunk 正在使用，prev_size 无效 p 主要用于内存块的合并操作；ptmalloc 分配的第一个块总是将 p 设为 1, 以防止程序引用到不存在的区域
    
*   M=1 为 mmap 映射区域分配；M=0 为 heap 区域分配
    
*   A=0 为主分配区分配；A=1 为非主分配区分配。
    

与非空闲 chunk 相比，空闲 chunk 在用户区域多了四个指针，分别为 fd,bk,fd_nextsize,bk_nextsize，这几个指针的含义在上面已经有解释，在此不再赘述。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9ZsbGia2Mpn459FDQa0VMmNSeiayISZDFJAVdmhaFN6vicOSl3ric0Scr3w/640?wx_fmt=png)空闲 chunk

### 5.3 空闲链表 (bins)

用户调用 free 函数释放内存的时候，ptmalloc 并不会立即将其归还操作系统，而是将其放入空闲链表 (bins) 中，这样下次再调用 malloc 函数申请内存的时候，就会从 bins 中取出一块返回，这样就避免了频繁调用系统调用函数，从而降低内存分配的开销。

在 ptmalloc 中，会将大小相似的 chunk 链接起来，叫做 bin。总共有 128 个 bin 供 ptmalloc 使用。

根据 chunk 的大小，ptmalloc 将 bin 分为以下几种：

*   fast bin
    
*   unsorted bin
    
*   small bin
    
*   large bin
    

从前面 malloc_state 结构定义，对 bin 进行分类，可以分为 fast bin 和 bins，其中 unsorted bin、small bin 以及 large bin 属于 bins。

在 glibc 中，上述 4 中 bin 的个数都不等，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9G55MWGnxWX5Gx09Lf9MB3wmFRV5aBiaOh1Rh7cG0lbd19xicCENds77w/640?wx_fmt=png)bin

#### 5.3.1 fast bin

程序在运行时会经常需要申请和释放一些较小的内存空间。当分配器合并了相邻的几个小的 chunk 之后, 也许马上就会有另一个小块内存的请求, 这样分配器又需要从大的空闲内存中切分出一块, 这样无疑是比较低效的, 故而, malloc 中在分配过程中引入了 fast bins。

在前面 malloc_state 定义中

```
mfastbinptr fastbins[NFASTBINS]; // NFASTBINS  = 10
```

1.  fast bin 的个数是 10 个
    
2.  每个 fast bin 都是一个单链表 (只使用 fd 指针)。这是因为 fast bin 无论是添加还是移除 chunk 都是在链表尾进行操作，也就是说，对 fast bin 中 chunk 的操作，采用的是 LIFO(后入先出) 算法：添加操作 (free 内存) 就是将新的 fast chunk 加入链表尾，删除操作 (malloc 内存) 就是将链表尾部的 fast chunk 删除。
    
3.  chunk size：10 个 fast bin 中所包含的 chunk size 以 8 个字节逐渐递增，即第一个 fast bin 中 chunk size 均为 16 个字节，第二个 fast bin 的 chunk size 为 24 字节，以此类推，最后一个 fast bin 的 chunk size 为 80 字节。
    
4.  不会对 free chunk 进行合并操作。这是因为 fast bin 设计的初衷就是小内存的快速分配和释放，因此系统将属于 fast bin 的 chunk 的 P(未使用标志位) 总是设置为 1，这样即使当 fast bin 中有某个 chunk 同一个 free chunk 相邻的时候，系统也不会进行自动合并操作，而是保留两者。
    
5.  malloc 操作：在 malloc 的时候，如果申请的内存大小范围在 fast bin 的范围内，则先在 fast bin 中查找，如果找到了，则返回。否则则从 small bin、unsorted bin 以及 large bin 中查找。
    
6.  free 操作：先通过 chunksize 函数根据传入的地址指针获取该指针对应的 chunk 的大小；然后根据这个 chunk 大小获取该 chunk 所属的 fast bin，然后再将此 chunk 添加到该 fast bin 的链尾即可。
    

下面是 fastbin 结构图：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9YqKOuI1F40QKlEiarRNvP1z8uYkhNZMZHgnRAtHJIEQRGrtyLduoSxQ/640?wx_fmt=png)fastbin

#### 5.3.2 unsorted bin

unsorted bin 的队列使用 bins 数组的第一个，是 bins 的一个缓冲区，加快分配的速度。当用户释放的内存大于 max_fast 或者 fast bins 合并后的 chunk 都会首先进入 unsorted bin 上。

在 unsorted bin 中，chunk 的 size 没有限制，也就是说任何大小 chunk 都可以放进 unsorted bin 中。这主要是为了让 “glibc malloc 机制” 能够有第二次机会重新利用最近释放的 chunk(第一次机会就是 fast bin 机制)。利用 unsorted bin，可以加快内存的分配和释放操作，因为整个操作都不再需要花费额外的时间去查找合适的 bin 了。　　

用户 malloc 时，如果在 fast bins 中没有找到合适的 chunk, 则 malloc 会先在 unsorted bin 中查找合适的空闲 chunk，如果没有合适的 bin，ptmalloc 会将 unsorted bin 上的 chunk 放入 bins 上，然后到 bins 上查找合适的空闲 chunk。

与 fast bin 所不同的是，unsortedbin 采用的遍历顺序是 FIFO。

unsorted bin 结构图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9oL9ia2a4uV7XVHA3e9sjAhPUWkDoJYxgfnhfAKXdvawlXo1fWlibicic7w/640?wx_fmt=png)unsorted bin

#### 5.3.3 small bin

大小小于 512 字节的 chunk 被称为 small chunk，而保存 small chunks 的 bin 被称为 small bin。数组从 2 开始编号，前 62 个 bin 为 small bins，small bin 每个 bin 之间相差 8 个字节，同一个 small bin 中的 chunk 具有相同大小。　　

每个 small bin 都包括一个空闲区块的双向循环链表（也称 binlist）。free 掉的 chunk 添加在链表的前端，而所需 chunk 则从链表后端摘除。　　

两个毗连的空闲 chunk 会被合并成一个空闲 chunk。合并消除了碎片化的影响但是减慢了 free 的速度。　　

分配时，当 samll bin 非空后，相应的 bin 会摘除 binlist 中最后一个 chunk 并返回给用户。

在 free 一个 chunk 的时候，检查其前或其后的 chunk 是否空闲，若是则合并，也即把它们从所属的链表中摘除并合并成一个新的 chunk，新 chunk 会添加在 unsorted bin 链表的前端。

small bin 也采用的是 FIFO 算法，即内存释放操作就将新释放的 chunk 添加到链表的 front end(前端)，分配操作就从链表的 rear end(尾端) 中获取 chunk。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9EeRPyzctf31dPqQNDj9vCDq3nleOsziaficjMbUTDHAbIicRDGRUMFjEQ/640?wx_fmt=png)small bin

#### 5.3.4 large bin

大小大于等于 512 字节的 chunk 被称为 large chunk，而保存 large chunks 的 bin 被称为 large bin，位于 small bins 后面。large bins 中的每一个 bin 分别包含了一个给定范围内的 chunk，其中的 chunk 按大小递减排序，大小相同则按照最近使用时间排列。

两个毗连的空闲 chunk 会被合并成一个空闲 chunk。

small bins 的策略非常适合小分配，但我们不能为每个可能的块大小都有一个 bin。对于超过 512 字节（64 位为 1024 字节）的块，堆管理器改为使用 “large bin”。

63 large bin 中的每一个都与 small bin 的操作方式大致相同，但不是存储固定大小的块，而是存储大小范围内的块。每个 large bin 的大小范围都设计为不与 small bin 的块大小或其他 large bin 的范围重叠。换句话说，给定一个块的大小，这个大小对应的正好是一个 small bin 或 large bin。

在这 63 个 largebins 中：第一组的 32 个 largebin 链依次以 64 字节步长为间隔，即第一个 largebin 链中 chunksize 为 1024-1087 字节，第二个 large bin 中 chunk size 为 1088~1151 字节。第二组的 16 个 largebin 链依次以 512 字节步长为间隔；第三组的 8 个 largebin 链以步长 4096 为间隔；第四组的 4 个 largebin 链以 32768 字节为间隔；第五组的 2 个 largebin 链以 262144 字节为间隔；最后一组的 largebin 链中的 chunk 大小无限制。

在进行 malloc 操作的时候，首先确定用户请求的大小属于哪一个 large bin，然后判断该 large bin 中最大的 chunk 的 size 是否大于用户请求的 size。如果大于，就从尾开始遍历该 large bin，找到第一个 size 相等或接近的 chunk，分配给用户。如果该 chunk 大于用户请求的 size 的话，就将该 chunk 拆分为两个 chunk：前者返回给用户，且 size 等同于用户请求的 size；剩余的部分做为一个新的 chunk 添加到 unsorted bin 中。

如果该 large bin 中最大的 chunk 的 size 小于用户请求的 size 的话，那么就依次查看后续的 large bin 中是否有满足需求的 chunk，不过需要注意的是鉴于 bin 的个数较多 (不同 bin 中的 chunk 极有可能在不同的内存页中)，如果按照上一段中介绍的方法进行遍历的话 (即遍历每个 bin 中的 chunk)，就可能会发生多次内存页中断操作，进而严重影响检索速度，所以 glibc malloc 设计了 Binmap 结构体来帮助提高 bin-by-bin 检索的速度。Binmap 记录了各个 bin 中是否为空，通过 bitmap 可以避免检索一些空的 bin。如果通过 binmap 找到了下一个非空的 large bin 的话，就按照上一段中的方法分配 chunk，否则就使用 top chunk（在后面有讲）来分配合适的内存。

large bin 的 free 操作与 small bin 一致，此处不再赘述。

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9aKGXI0nbmqEabKvPBk08QCGbTXxaLnkk5rLOYBiazpc7lxzvms49Icw/640?wx_fmt=png)large bin

上述几种 bin，组成了进程中最核心的分配部分：bins，如下图所示：![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX93icYNGPuPlMDFEkHVWTSdxxiatveFHsekFgLgtyAicIJAGP4VPHz5ibyxA/640?wx_fmt=png)

### 5.4 特殊 chunk

上节内容讲述了几种 bin 以及各种 bin 内存的分配和释放特点，但是，仅仅上面几种 bin 还不能够满足，比如假如上述 bins 不能满足分配条件的时候，glibc 提出了另外几种特殊的 chunk 供分配和释放，分别为 top chunk，mmaped chunk 和 last remainder chunk。

#### 5.4.1 top trunk

top chunk 是堆最上面的一段空间，它不属于任何 bin，当所有的 bin 都无法满足分配要求时，就要从这块区域里来分配，分配的空间返给用户，剩余部分形成新的 top chunk，如果 top chunk 的空间也不满足用户的请求，就要使用 brk 或者 mmap 来向系统申请更多的堆空间（主分配区使用 brk、sbrk，非主分配区使用 mmap）。

在 free chunk 的时候，如果 chunk size 不属于 fastbin 的范围，就要考虑是不是和 top chunk 挨着，如果挨着，就要 merge 到 top chunk 中。

#### 5.4.2 mmaped chunk

当分配的内存非常大（大于分配阀值，默认 128K）的时候，需要被 mmap 映射，则会放到 mmaped chunk 上，当释放 mmaped chunk 上的内存的时候会直接交还给操作系统。（chunk 中的 M 标志位置 1）

#### 5.4.3 last remainder chunk

Last remainder chunk 是另外一种特殊的 chunk，这个特殊 chunk 是被维护在 unsorted bin 中的。

如果用户申请的 size 属于 small bin 的，但是又不能精确匹配的情况下，这时候采用最佳匹配（比如申请 128 字节，但是对应的 bin 是空，只有 256 字节的 bin 非空，这时候就要从 256 字节的 bin 上分配），这样会 split chunk 成两部分，一部分返给用户，另一部分形成 last remainder chunk，插入到 unsorted bin 中。

当需要分配一个 small chunk, 但在 small bins 中找不到合适的 chunk，如果 last remainder chunk 的大小大于所需要的 small chunk 大小，last remainder chunk 被分裂成两个 chunk，其中一个 chunk 返回给用户，另一个 chunk 变成新的 last remainder chunk。

last remainder chunk 主要通过提高内存分配的局部性来提高连续 malloc（产生大量 small chunk）的效率。

### 5.5 chunk 切分

chunk 释放时，其长度不属于 fastbins 的范围，则合并前后相邻的 chunk。首次分配的长度在 large bin 的范围，并且 fast bins 中有空闲 chunk，则将 fastbins 中的 chunk 与相邻空闲的 chunk 进行合并，然后将合并后的 chunk 放到 unsorted bin 中，如果 fastbin 中的 chunk 相邻的 chunk 并非空闲无法合并，仍旧将该 chunk 放到 unsorted bin 中，即能合并的话就进行合并，但最终都会放到 unsorted bin 中。

fastbins，small bin 中都没有合适的 chunk，top chunk 的长度也不能满足需要，则对 fast bin 中的 chunk 进行合并。

### 5.6 chunk 合并

前面讲了相邻的 chunk 可以合并成一个大的 chunk，反过来，一个大的 chunk 也可以分裂成两个小的 chunk。chunk 的分裂与从 top chunk 中分配新的 chunk 是一样的。需要注意的一点是：分裂后的两个 chunk 其长度必须均大于 chunk 的最小长度（对于 64 位系统是 32 字节），即保证分裂后的两个 chunk 仍旧是可以被分配使用的，否则则不进行分裂，而是将整个 chunk 返回给用户。

6 内存分配 (malloc)
---------------

glibc 运行时库分配动态内存，底层用的是 malloc 来实现 (new 最终也是调用 malloc)，下面是 malloc 函数调用流程图：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9pxMalYAla7bvpXq014GOgeHNjhTibeXGMx0EIH3kSCNZDTjDoKvrx8w/640?wx_fmt=gif)malloc

在此，将上述流程图以文字形式表示出来，以方便大家理解：

1.  获取分配区的锁，为了防止多个线程同时访问同一个分配区，在进行分配之前需要取得分配区域的锁。线程先查看线程私有实例中是否已经存在一个分配区，如果存在尝试对该分配区加锁，如果加锁成功，使用该分配区分配内存，否则，该线程搜索分配区循环链表试图获得一个空闲（没有加锁）的分配区。如果所有的分配区都已经加锁，那么 ptmalloc 会开辟一个新的分配区，把该分配区加入到全局分配区循环链表和线程的私有实例中并加锁，然后使用该分配区进行分配操作。开辟出来的新分配区一定为非主分配区，因为主分配区是从父进程那里继承来的。开辟非主分配区时会调用 mmap() 创建一个 sub-heap，并设置好 top chunk。
    
2.  将用户的请求大小转换为实际需要分配的 chunk 空间大小。
    
3.  判断所需分配 chunk 的大小是否满足 chunk_size <= max_fast (max_fast 默认为 64B)， 如果是的话，则转下一步，否则跳到第 5 步。
    
4.  首先尝试在 fast bins 中取一个所需大小的 chunk 分配给用户。如果可以找到，则分配结束。否则转到下一步。
    
5.  判断所需大小是否处在 small bins 中，即判断 chunk_size < 512B 是否成立。如果 chunk 大小处在 small bins 中，则转下一步，否则转到第 6 步。
    
6.  根据所需分配的 chunk 的大小，找到具体所在的某个 small bin，从该 bin 的尾部摘取一个恰好满足大小的 chunk。若成功，则分配结束，否则，转到下一步。
    
7.  到了这一步，说明需要分配的是一块大的内存，或者 small bins 中找不到合适的 chunk。于是，ptmalloc 首先会遍历 fast bins 中的 chunk，将相邻的 chunk 进行合并，并链接到 unsorted bin 中，然后遍历 unsorted bin 中的 chunk，如果 unsorted bin 只有一个 chunk，并且这个 chunk 在上次分配时被使用过，并且所需分配的 chunk 大小属于 small bins，并且 chunk 的大小大于等于需要分配的大小，这种情况下就直接将该 chunk 进行切割，分配结束，否则将根据 chunk 的空间大小将其放入 small bins 或是 large bins 中，遍历完成后，转入下一步。
    
8.  到了这一步，说明需要分配的是一块大的内存，或者 small bins 和 unsorted bin 中都找不到合适的 chunk，并且 fast bins 和 unsorted bin 中所有的 chunk 都清除干净了。从 large bins 中按照 “smallest-first，best-fit” 原则，找一个合适的 chunk，从中划分一块所需大小的 chunk，并将剩下的部分链接回到 bins 中。若操作成功，则分配结束，否则转到下一步。
    
9.  如果搜索 fast bins 和 bins 都没有找到合适的 chunk，那么就需要操作 top chunk 来进行分配了。判断 top chunk 大小是否满足所需 chunk 的大小，如果是，则从 top chunk 中分出一块来。否则转到下一步。
    
10.  到了这一步，说明 top chunk 也不能满足分配要求，所以，于是就有了两个选择: 如果是主分配区，调用 sbrk()，增加 top chunk 大小；如果是非主分配区，调用 mmap 来分配一个新的 sub-heap，增加 top chunk 大小；或者使用 mmap() 来直接分配。在这里，需要依靠 chunk 的大小来决定到底使用哪种方法。判断所需分配的 chunk 大小是否大于等于 mmap 分配阈值，如果是的话，则转下一步，调用 mmap 分配， 否则跳到第 12 步，增加 top chunk 的大小。
    
11.  使用 mmap 系统调用为程序的内存空间映射一块 chunk_size align 4kB 大小的空间。然后将内存指针返回给用户。
    
12.  判断是否为第一次调用 malloc，若是主分配区，则需要进行一次初始化工作，分配一块大小为 (chunk_size + 128KB) align 4KB 大小的空间作为初始的 heap。若已经初始化过了，主分配区则调用 sbrk() 增加 heap 空间，分主分配区则在 top chunk 中切割出一个 chunk，使之满足分配需求，并将内存指针返回给用户。
    

将上面流程串起来就是：

> ❝
> 
> 根据用户请求分配的内存的大小，ptmalloc 有可能会在两个地方为用户分配内存空间。在第一次分配内存时，一般情况下只存在一个主分配区，但也有可能从父进程那里继承来了多个非主分配区，在这里主要讨论主分配区的情况，brk 值等于 start_brk，所以实际上 heap 大小为 0，top chunk 大小也是 0。这时，如果不增加 heap 大小，就不能满足任何分配要求。所以，若用户的请求的内存大小小于 mmap 分配阈值， 则 ptmalloc 会初始 heap。然后在 heap 中分配空间给用户，以后的分配就基于这个 heap 进行。若第一次用户的请求就大于 mmap 分配阈值，则 ptmalloc 直接使用 mmap()分配一块内存给用户，而 heap 也就没有被初始化，直到用户第一次请求小于 mmap 分配阈值的内存分配。第一次以后的分配就比较复杂了，简单说来，ptmalloc 首先会查找 fast bins，如果不能找到匹配的 chunk，则查找 small  bins。若仍然不满足要求，则合并 fast bins，把 chunk 加入 unsorted bin，在 unsorted bin 中查找，若仍然不满足要求，把 unsorted bin 中的 chunk 全加入 large bins 中，并查找 large bins。在 fast bins 和 small bins 中的查找都需要精确匹配， 而在 large  bins 中查找时，则遵循 “smallest-first，best-fit” 的原则，不需要精确匹配。若以上方法都失败了，则 ptmalloc 会考虑使用 top chunk。若 top chunk 也不能满足分配要求。而且所需 chunk 大小大于 mmap 分配阈值，则使用 mmap 进行分配。否则增加 heap，增大 top chunk。以满足分配要求。
> 
> ❞

当然了，glibc 中 malloc 的分配远比上面的要复杂的多，要考虑到各种情况，比如指针异常ΩΩ越界等，将这些判断条件也加入到流程图中，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhj9IYTlypoT70yB70bWpJfe9xTz1bEmEaFAKp4Dnb4RBnibUialibiafiagARPw1sk33HLovR50bEC9nw/640?wx_fmt=gif)

malloc

7 内存释放 (free)
-------------

malloc 进行内存分配，那么与 malloc 相对的就是 free，进行内存释放，下面是 free 函数的基本流程图：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9gXibFHj4CPV7HhBBK10XIsHlMUDibgFGpeZgkV7SZ6xEgQguaC2g55ZA/640?wx_fmt=gif)free

对上述流程图进行描述，如下：

1.  获取分配区的锁，保证线程安全。
    
2.  如果 free 的是空指针，则返回，什么都不做。
    
3.  判断当前 chunk 是否是 mmap 映射区域映射的内存，如果是，则直接 munmap() 释放这块内存。前面的已使用 chunk 的数据结构中，我们可以看到有 M 来标识是否是 mmap 映射的内存。
    
4.  判断 chunk 是否与 top chunk 相邻，如果相邻，则直接和 top chunk 合并（和 top chunk 相邻相当于和分配区中的空闲内存块相邻）。否则，转到步骤 8
    
5.  如果 chunk 的大小大于 max_fast（64b），则放入 unsorted bin，并且检查是否有合并，有合并情况并且和 top chunk 相邻，则转到步骤 8；没有合并情况则 free。
    
6.  如果 chunk 的大小小于 max_fast（64b），则直接放入 fast bin，fast bin 并没有改变 chunk 的状态。没有合并情况，则 free；有合并情况，转到步骤 7
    
7.  在 fast bin，如果当前 chunk 的下一个 chunk 也是空闲的，则将这两个 chunk 合并，放入 unsorted bin 上面。合并后的大小如果大于 64B，会触发进行 fast bins 的合并操作，fast bins 中的 chunk 将被遍历，并与相邻的空闲 chunk 进行合并，合并后的 chunk 会被放到 unsorted bin 中，fast bin 会变为空。合并后的 chunk 和 topchunk 相邻，则会合并到 topchunk 中。转到步骤 8
    
8.  判断 top chunk 的大小是否大于 mmap 收缩阈值（默认为 128KB），如果是的话，对于主分配区，则会试图归还 top chunk 中的一部分给操作系统。free 结束。
    

如果将 free 函数内部各种条件加入进去，那么 free 调用的详细流程图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/p3sYCQXkuHhRXIkKJ7KWooq2f0YibyXX9LMAxjZnicyhfuThoonVddcpL2Rlxv4nWMkCocJib7smjuxDqe9lmbAew/640?wx_fmt=gif)free

8 问题分析以及解决
----------

通过前面对 glibc 运行时库的分析，基本就能定位出原因，是因为我们调用了 free 进行释放，但仅仅是将内存返还给了 glibc 库，而 glibc 库却没有将内存归还操作系统，最终导致系统内存耗尽，程序因为 OOM 被系统杀掉。

有以下两种方案：

*   禁用 ptmalloc 的 mmap 分配 阈 值 动 态 调 整 机 制 。通 过 mallopt() 设置 M_TRIM_THRESHOLD, M_MMAP_THRESHOLD, M_TOP_PAD 和 M_MMAP_MAX 中的任意一个，关闭 mmap 分配阈值动态调整机制，同时需要将 mmap 分配阈值设置为 64K，大于 64K 的内存分配都使用 mmap 向系统分配，释放大于 64K 的内存将调用 munmap 释放回系统。但是，这种方案的 缺点是每次内存分配和申请，都是直接向操作系统申请，效率低。
    
*   预 估 程 序 可 以 使 用 的 最 大 物 理 内 存 大 小 ， 配 置 系 统 的 /proc/sys/vm/overcommit_memory，/proc/sys/vm/overcommit_ratio，以及使用 ulimt –v 限制程序能使用虚拟内存空间大小，防止程序因 OOM 被杀掉。这种方案的 缺点是如果预估的内存小于进程实际占用，那么仍然会出现 OOM，导致进程被杀掉。
    
*   tcmalloc
    

最终采用 tcmalloc 来解决了问题，后面有机会的话，会写一篇 tcmalloc 内存管理的相关文章。

9 结语
----

业界语句说法，是否了解内存管理机制，是辨别 C/C++ 程序员和其他的高级语言程序员的重要区别。作为 C/C++ 中的最重要的特性，指针及动态内存管理在给编程带来极大的灵活性的同时，也给开发人员带来了许多困扰。

了解底层内存实现，有时候会有意想不到的效果哦。

先写到这里吧。

10 参考
-----

1、https://sourceware.org/glibc/wiki/MallocInternals

2、https://titanwolf.org/Network/Articles/Article?AID=99524c69-cb90-4d61-bb28-01c0864d0ccc

3、https://blog.fearcat.in/a?ID=00100-535db575-0d98-4287-92b6-4d7d9604b216

4、https://sploitfun.wordpress.com/2015/02/10/understanding-glibc-malloc/

5、https://sploitfun.wordpress.com/2015/02/11/syscalls-used-by-malloc

- EOF -

推荐阅读  点击标题可跳转

1、[介绍一个 C++ 中非常有用的设计模式](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651168199&idx=1&sn=9f1f5be983a9141ddb46237248e0de55&chksm=80644c98b713c58ef010f0584b8feca1b68d8d9ca26fcccd0a404edb6b32024fd18f38b80d63&scene=21#wechat_redirect)

2、[Effective C++ 高阶笔记](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651168193&idx=1&sn=de52028d24539944930f1a00b2d82f6d&chksm=80644c9eb713c58862bd086fa08cbc7a9a881fb01745110122b0202b25119815de997b01ba7a&scene=21#wechat_redirect)

3、[字节一面：“为什么网络要分层？每一层的职责、包含哪些协议？”](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651168170&idx=1&sn=e31e011d9442d7d316a0f7a036dbbf94&chksm=80644cf5b713c5e366eebcbbe7ec9e1a4c08bc2b52d9510e2faaf01695c0ab9d86a2f6b3942b&scene=21#wechat_redirect)