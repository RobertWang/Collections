> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247514219&idx=4&sn=48cb472082269913b094a825f706d20d&chksm=e8791d11df0e94073b7f0831a350c7279e48df5c833bbebd1b55dbfeff0d694258d63fe6b546&mpshare=1&scene=1&srcid=0220t5EG8XbQ3E6BRdCe9tE1&sharer_sharetime=1645322326738&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> Linux 设备驱动 - 内核管理设备号的实现细节

### 开篇

本文引用的内核代码参考来自版本 linux-5.15.4 。

在 Linux 系统中，每个注册到系统的设备都有一个编号，这个编号便是 Linux 系统中的设备号。

设备号作为一种系统资源，需要加以管理。否则，如果设备号与驱动程序对应关系错误，就会引起混乱或引起潜在的问题。

通过查看 /proc/devices 文件可以得到系统中注册的设备，第一列为主设备号，第二列为设备名称

```
$ cat /proc/devices

Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  5 ttyprintk
  6 lp
  7 vcs
 10 misc
 13 input
 21 sg
...

Block devices:
  7 loop
  8 sd
  9 md
 11 sr
 65 sd
 66 sd
 ...
```

### 设备号的构成

一个设备号由主设备号和次设备号构成。

主设备号对应设备驱动程序，同一类设备一般使用相同的主设备号。

次设备号由驱动程序使用，驱动程序用来描述使用该驱动的设备的序号，序号一般从 0 开始。

Linux 设备号用 dev_t 类型的变量进行标识，这是一个 32 位 无符号整数，内核源码定义为：

```
/* <include/linux/types.h> */

typedef u32 __kernel_dev_t;
typedef __kernel_dev_t        dev_t;
```

主设备号用 dev_t 的高 12 位表示，次设备号用 dev_t 低 20 位表示。

内核提供了几个宏定义，供驱动程序操作设备号时使用：

```
/* <include/linux/kdev_t.h> */

#define MINORBITS    20
#define MINORMASK    ((1U << MINORBITS) - 1)

#define MAJOR(dev)    ((unsigned int) ((dev) >> MINORBITS))
#define MINOR(dev)    ((unsigned int) ((dev) & MINORMASK))
#define MKDEV(ma,mi)    (((ma) << MINORBITS) | (mi))
```

宏 MAJOR 从设备号 dev 中提取主设备号。宏 MINOR 用来从设备号 dev 中提取次设备号。宏 MKDEV 用来将主设备号 ma 和 次设备号 mi 组合成 dev_t 类型的设备号。

另外，内核也提供了从设备文件 i - 节点结构（inode 结构体）中获取主次设备号的函数，如下：

```
/* <include/linux/fs.h> */

/* 获取次设备号 */
static inline unsigned iminor(const struct inode *inode)
{
    return MINOR(inode->i_rdev);
}

/* 获取主设备号 */
static inline unsigned imajor(const struct inode *inode)
{
    return MAJOR(inode->i_rdev);
}
```

通过函数源码可知，获取主设备号和次设备号最终是通过宏定义完成的。

### 内核管理设备号

以字符设备为例，向内核中注册设备号，内核是如何分配和管理设备号的呢？

在编写字符设备驱动时，可以通过如下两个系统调用向内核注册设备号：

*   **register_chrdev_region()**
    

注册一系列连续的字符设备号，主设备号需要函数调用者指定。此函数的原型为：

```
int register_chrdev_region(dev_t from, unsigned count, const char *name)
```

参数 from 为设备编号，包含主设备号和次设备号。参数 count 用于指定连续设备号的个数，即当前驱动程序所管理的同类设备的个数。参数 name 为设备或驱动的名字。

执行成功，返回 0。失败，则返回一个负值的错误码。

*   **alloc_chrdev_region()**
    

注册一系列连续的字符设备号，主设备号是由内核动态分配得到的。此函数的原型为：

```
int alloc_chrdev_region(dev_t *dev, unsigned baseminor, unsigned count, const char *name)
```

参数 dev 为函数的传出参数，用于记录动态分配的设备号，如果申请多个设备号，则此参数记录这些连续设备号的起始值。

参数 baseminor 指定首个次设备号。参数 count 用于指定连续设备号的个数。参数 name 为设备或驱动的名字。

执行成功，返回 0。失败，则返回一个负值的错误码。

接下来，看看这两个函数的内部实现流程。

### register_chrdev_region()

该函数的内核源码为，关键部分已加注释：

```
/* <fs.char_dev.c> */

int register_chrdev_region(dev_t from, unsigned count, const char *name)
{
    struct char_device_struct *cd;
    dev_t to = from + count;
    dev_t n, next;

    /* 循环，注册多个连续的设备号 */
    for (n = from; n < to; n = next) 
    {
        /* 计算得到下一个设备号 */
        next = MKDEV(MAJOR(n)+1, 0);
        /* 判断是否超限 */
        if (next > to)
            next = to;
        /* 向内核注册指定的设备号 */
        cd = __register_chrdev_region(MAJOR(n), MINOR(n), next - n, name);
        if (IS_ERR(cd))
            goto fail;
    }
    return 0;
fail:
    /* 如果失败，则释放已申请的设备号资源 */
    to = n;
    for (n = from; n < to; n = next) 
    {
        next = MKDEV(MAJOR(n)+1, 0);
        kfree(__unregister_chrdev_region(MAJOR(n), MINOR(n), next - n));
    }
    return PTR_ERR(cd);
}
```

由代码内容可知，这个函数的核心处理流程是通过内部调用 `__register_chrdev_region()`实现的。

这个函数的主要功能是，将要使用的设备号注册到内核的设备号管理体系中，避免多个驱动程序使用相同的设备号，而引起的混乱。

如果注册设备号已经被使用，则会返回错误码告知调用者，即调用失败。如果成功，则函数返回 0。

**struct char_device_struct**

在调用过程中，会涉及到一个关键的数据结构 `struct char_device_struct`，其定义如下：

```
#define CHRDEV_MAJOR_HASH_SIZE 255

static struct char_device_struct {
    struct char_device_struct *next;  /* 链表指针 */
    unsigned int major;  /* 主设备号 */
    unsigned int baseminor;  /* 次设备号 */
    int minorct;   /* 此设备号个数 */
    char name[64];  /* 设备名称 */
    struct cdev *cdev;      /* will die */
} *chrdevs[CHRDEV_MAJOR_HASH_SIZE];
```

定义结构体的同时，还定义了一个全局性的指针数组 chrdevs，是内核用来分配和管理设备号的。数组中的每一个元素都是指向 `struct char_device_struct` 类型的指针。

函数 `register_chrdev_region()` 的主要功能是将驱动程序要使用的设备号记录到 chrdevs 数组中。

**__register_chrdev_region()**

核心处理函数 __register_chrdev_region() 内部，首先会分配一个 `struct char_device_struct`  类型的指针 cd，然后对其进行初始化（已经去除无关代码）：

```
static struct char_device_struct *
__register_chrdev_region(unsigned int major, unsigned int baseminor,
               int minorct, const char *name)
{
    ...
    cd = kzalloc(sizeof(struct char_device_struct), GFP_KERNEL);
    if (cd == NULL)
        return ERR_PTR(-ENOMEM);

    /* 根据主设备号计算索引，搜索 chrdevs 数组，判断主设备号是否可用 */
    i = major_to_index(major);
    for (curr = chrdevs[i]; curr; prev = curr, curr = curr->next) 
    {
        if (curr->major < major)
            continue;
        if (curr->major > major)
            break;
        if (curr->baseminor + curr->minorct <= baseminor)
            continue;
        if (curr->baseminor >= baseminor + minorct)
            break;
        goto out;
    }
    /* 初始化信息 */
    cd->major = major;
    cd->baseminor = baseminor;
    cd->minorct = minorct;
    strlcpy(cd->name, name, sizeof(cd->name));

    /* 将分配的 cd 加入到 chrdevs[i] 中 */
    if (!prev) {
        cd->next = curr;
        chrdevs[i] = cd;
    } else {
        cd->next = prev->next;
        prev->next = cd;
    }
    ...
}
```

函数申请完内存资源后，开始扫描 chrdevs 数组，确保当前注册的设备号可用。如果设备号占用，函数返回错误码，即调用失败。

如果设备号可用，则用设备号和名字信息初始化。初始化完成后，将 `struct char_device_struct` 加入到内核管理设备号的链表中。

### alloc_chrdev_region()

此函数由内核动态分配设备号，该函数的内核源码如下，关键部分已加注释：

```
/* <fs.char_dev.c> */

int alloc_chrdev_region(dev_t *dev, unsigned baseminor, unsigned count,
            const char *name)
{
    struct char_device_struct *cd;

    /* 向内核注册设备号 */
    cd = __register_chrdev_region(0, baseminor, count, name);
    if (IS_ERR(cd))
        return PTR_ERR(cd);

    /* 得到动态获取的首个设备号 */
    *dev = MKDEV(cd->major, cd->baseminor);
    return 0;
}
```

这个函数的核心处理也是由函数 `__register_chrdev_region()` 实现的。

与 `register_chrdev_region()` 相比，`alloc_chrdev_region()` 在调用 `__register_chrdev_region()` 时，第一个参数为 0。此时 `__register_chrdev_region()` 处理流程代码如下，

```
static struct char_device_struct *
__register_chrdev_region(unsigned int major, unsigned int baseminor,
               int minorct, const char *name)
{
    ...
    cd = kzalloc(sizeof(struct char_device_struct), GFP_KERNEL);
    if (cd == NULL)
        return ERR_PTR(-ENOMEM);

    /* 查找可用的主设备号 */
    f (major == 0) {
        ret = find_dynamic_major();
        if (ret < 0) {
            pr_err("CHRDEV \"%s\" dynamic allocation region is full\n",
                   name);
            goto out;
        }
        major = ret;
    }
    ...
}
```

在分配完成 `struct char_device_struct` 内存资源之后，通过 `find_dynamic_major()` 查找可用的主设备号。后续处理与 `register_chrdev_region()` 函数调用处理相同。

设备号分配成功后，将 `struct char_device_struct` 类型指针返回给 `alloc_chrdev_region()` 函数。然后再通过如下代码将新分配的设备号返回给 `alloc_chrdev_region()` 调用者：

```
*dev = MKDEV(cd->major, cd->baseminor);
```

### 小结

本文主要介绍了以下几点内容：

*   设备号是如何构成的，以及对其操作的宏定义。
    
*   register_chrdev_region() 和 alloc_chrdev_region() 实现细节。
    
*   记录设备号相关信息的关键数据结构 `struct char_device_struct`。
    
*   内核通过 chrdevs 数组来跟踪系统中设备号的使用情况。