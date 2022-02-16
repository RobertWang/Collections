> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651170209&idx=1&sn=060ee1c374258db50ba2434f32674fbf&chksm=806474feb713fde8ecefd555d231dde9e559eb15666c6d99b8d3fb26b98933baead8bd3271b0&mpshare=1&scene=1&srcid=0214pENoszLY6Q6bBgJpVdI1&sharer_sharetime=1644812808498&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

一、什么是系统调用
---------

`系统调用` 是内核提供给应用程序使用的功能函数，由于应用程序一般运行在 `用户态`，处于用户态的进程有诸多限制（如不能进行 I/O 操作），所以有些功能必须由内核代劳完成。而内核就是通过向应用层提供 `系统调用`，来完成一些在用户态不能完成的工作。

说白了，系统调用其实就是函数调用，只不过调用的是内核态的函数。但与普通的函数调用不同，系统调用不能使用 `call` 指令来调用，而是需要使用 `软中断` 来调用。在 Linux 系统中，系统调用一般使用 `int 0x80` 指令（x86）或者 `syscall` 指令（x64）来调用。

下面我们以 `int 0x80` 指令（x86）调用方式为例，来说明系统调用的原理。

二、系统调用原理
--------

在 Linux 内核中，使用 `sys_call_table` 数组来保存所有系统调用，`sys_call_table` 数组每一个元素代表着一个系统调用的入口，其定义如下：

```
typedef void (*sys_call_ptr_t)(void);

const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] = {
    ...
};
```

当应用程序需要调用一个系统调用时，首先需要将要调用的系统调用号（也就是系统调用所在 `sys_call_table` 数组的索引）放置到 `eax` 寄存器中，然后通过使用 `int 0x80` 指令触发调用 `0x80` 号软中断服务。

`0x80` 号软中断服务，会通过以下代码来调用系统调用，如下所示：

```
...
call *sys_call_table(,%eax,8)
...
```

上面的代码会根据 `eax` 寄存器中的值来调用正确的系统调用，其过程如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/ciab8jTiab9J7ogndHy01HFemoyqaibibicExYx2lATS8NDPydaicw5wEKEjasDwWUgnKzWibnCoicw55Xrs4okkFDfia7A/640?wx_fmt=jpeg)

整体流程  

![](https://mmbiz.qpic.cn/mmbiz_png/cYSwmJQric6lqibOYp8ENMHnMLo8WdbOA6e3aVJ0oeKf4pbK2iade8fVtibsE8cLXCX5g3xxpAFwu65XM7b4Z57l5w/640?wx_fmt=png)

三、系统调用拦截
--------

了解了系统调用的原理后，要拦截系统调用就很简单了。那么如何拦截呢？

做法就是：我们只需要把 `sys_call_table` 数组的系统调用换成我们自己编写的函数入口即可。比如，我们想要拦截 `write()` 系统调用，那么只需要将 `sys_call_table` 数组的第一个元素换成我们编写好的函数（因为 `write()` 系统调用在 `sys_call_table` 数组的索引为 1）。

要修改 `sys_call_table` 数组元素的值，步骤如下：

### 1. 获取 `sys_call_table` 数组的地址

> 要修改 `sys_call_table` 数组元素的值，一般需要通过内核模块来完成。因为用户态程序由于内存保护机制，不能改写内核态的数据。而内核模块运行在内核态，所以能够跳过这个限制。

要修改 `sys_call_table` 数组元素的值，首先要获取 `sys_call_table` 数组的虚拟内存地址（由于 `sys_call_table` 变量不是一个导出符号，所以内核模块不能直接使用）。

要获取 `sys_call_table` 数组的虚拟内存地址有两种方法：

#### 第一种方法：从 `System.map` 文件中读取

`System.map` 是一份内核符号表，包含了内核中的变量名和函数名地址，在每次编译内核时，自动生成。获取 `sys_call_table` 数组的虚拟地址使用如下命令：

```
sudo cat /boot/System.map-`uname -r` | grep sys_call_table
```

结果如下图所示：  
  

![](https://mmbiz.qpic.cn/mmbiz_png/ciab8jTiab9J7ogndHy01HFemoyqaibibicEx4NbYibjjxuAL92ias1ia9K6n2fnUpQjy1ym6c42Hk8d4UHFUP8Atp4ZCg/640?wx_fmt=png)

  
从上图可知，`sys_call_table` 数组的虚拟地址为：`ffffffff818001c0`。

#### 第二种方法：通过 `kallsyms_lookup_name()` 函数来获取

从 `System.map` 文件中读取的方法不是很优雅，所以内核提供了一个名为 `kallsyms_lookup_name()` 的函数来获取内核变量和内核函数的虚拟内存地址。

`kallsyms_lookup_name()` 函数的使用很简单，只需要传入要获取虚拟内存地址的变量名即可，如下代码所示：

```
#include <linux/kallsyms.h>

void func() {
    ...
    unsigned long *sys_call_table;

    // 获取 sys_call_table 的虚拟内存地址
    sys_call_table = (unsigned long *)kallsyms_lookup_name("sys_call_table");
    ...
}
```

### 2. 设置 sys_call_table 数组为可写状态

是不是获取到 `sys_call_table` 数组的虚拟地址就可以修改其元素的值呢？没那么简单。

由于 `sys_call_table` 数组处于写保护区域，并不能直接修改其内容。但有两种方法可以将写保护暂时关闭，如下：

#### 第一种方法：将 `cr0` 寄存器的第 16 位设置为零

`cr0` 控制寄存器的第 16 位是写保护位，若设置为零，则允许超级权限往内核中写入数据。这样我们可以在修改 `sys_call_table` 数组的值前，将 `cr0` 寄存器的第 16 位清零，使其可以修改 `sys_call_table` 数组的内容。当修改完后，又将那一位复原即可。

代码如下：

```
/*
 * 设置cr0寄存器的第16位为0
 */
unsigned int clear_and_return_cr0(void)
{
    unsigned int cr0 = 0;
    unsigned int ret;

    /* 将cr0寄存器的值移动到rax寄存器中，同时输出到cr0变量中 */
    asm volatile ("movq %%cr0, %%rax" : "=a"(cr0));

    ret = cr0;
    cr0 &= 0xfffeffff;  /* 将cr0变量值中的第16位清0，将修改后的值写入cr0寄存器 */

    /* 读取cr0的值到rax寄存器，再将rax寄存器的值放入cr0中 */
    asm volatile ("movq %%rax, %%cr0" :: "a"(cr0));

    return ret;
}

/*
 * 还原cr0寄存器的值为val
 */
void setback_cr0(unsigned int val)
{
    asm volatile ("movq %%rax, %%cr0" :: "a"(val));
}
```

#### 第二种方法：设置虚拟地址对应页表项的读写属性

由于 `x86 CPU` 的内存保护机制是通过虚拟内存页表来实现的，所以我们只需要把 `sys_call_table` 数组的虚拟内存页表项中的保护标志位清空即可，代码如下：

```
/*
 * 把虚拟内存地址设置为可写
 */
int make_rw(unsigned long address)
{
    unsigned int level;

    //查找虚拟地址所在的页表地址
    pte_t *pte = lookup_address(address, &level);

    if (pte->pte & ~_PAGE_RW)  //设置页表读写属性
        pte->pte |=  _PAGE_RW;

    return 0;
}

/*
 * 把虚拟内存地址设置为只读
 */
int make_ro(unsigned long address)
{
    unsigned int level;

    pte_t *pte = lookup_address(address, &level);
    pte->pte &= ~_PAGE_RW;  //设置只读属性

    return 0;
}
```

### 3. 修改 `sys_call_table` 数组的内容

万事俱备，只欠东风。前面我们把准备工作都做完了，现在只需要把 `sys_call_table` 数组中的系统调用入口替换成我们编写的函数入口即可。

我们可以在内核模块初始化函数修改 `sys_call_table` 数组的值，然后在内核模块退出函数改回成原来的值即可，完整代码如下：

```
/*
 * File: syscall.c
 */

#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/unistd.h>
#include <linux/time.h>
#include <asm/uaccess.h>
#include <linux/sched.h>
#include <linux/kallsyms.h>

unsigned long *sys_call_table;

unsigned int clear_and_return_cr0(void);
void setback_cr0(unsigned int val);
static int sys_hackcall(void);

unsigned long *sys_call_table = 0;

/* 定义一个函数指针，用来保存原来的系统调用*/
static int (*orig_syscall_saved)(void);

/*
 * 设置cr0寄存器的第16位为0
 */
unsigned int clear_and_return_cr0(void)
{
    unsigned int cr0 = 0;
    unsigned int ret;

    /* 将cr0寄存器的值移动到rax寄存器中，同时输出到cr0变量中 */
    asm volatile ("movq %%cr0, %%rax" : "=a"(cr0));

    ret = cr0;
    cr0 &= 0xfffeffff;  /* 将cr0变量值中的第16位清0，将修改后的值写入cr0寄存器 */

    /* 读取cr0的值到rax寄存器，再将rax寄存器的值放入cr0中 */
    asm volatile ("movq %%rax, %%cr0" :: "a"(cr0));

    return ret;
}

/*
 * 还原cr0寄存器的值为val
 */
void setback_cr0(unsigned int val)
{
    asm volatile ("movq %%rax, %%cr0" :: "a"(val));
}

/*
 * 自己编写的系统调用函数
 */
static int sys_hackcall(void)
{
    printk("Hack syscall is successful!!!\n");
    return 0;
}

/*
 * 模块的初始化函数，模块的入口函数，加载模块时调用
 */
static int __init init_hack_module(void)
{
    int orig_cr0;

    printk("Hack syscall is starting...\n");

    /* 获取 sys_call_table 虚拟内存地址 */
    sys_call_table = (unsigned long *)kallsyms_lookup_name("sys_call_table");

    /* 保存原始系统调用 */
    orig_syscall_saved = (int(*)(void))(sys_call_table[__NR_perf_event_open]);

    orig_cr0 = clear_and_return_cr0(); /* 设置cr0寄存器的第16位为0 */
    sys_call_table[__NR_perf_event_open] = (unsigned long)&sys_hackcall; /* 替换成我们编写的函数 */
    setback_cr0(orig_cr0); /* 还原cr0寄存器的值 */

    return 0;
}

/*
 * 模块退出函数，卸载模块时调用
 */
static void __exit exit_hack_module(void)
{
    int orig_cr0;

    orig_cr0 = clear_and_return_cr0();
    sys_call_table[__NR_perf_event_open] = (unsigned long)orig_syscall_saved; /* 设置为原来的系统调用 */
    setback_cr0(orig_cr0);

    printk("Hack syscall is exited....\n");
}

module_init(init_hack_module);
module_exit(exit_hack_module);
MODULE_LICENSE("GPL");
```

在上面代码中，我们将 `perf_event_open()` 系统调用替换成了我们自己实现的函数。

> 注意：测试时最好使用冷门的系统调用，否则可能会导致系统崩溃。

### 4. 编写 Makefile 文件

为了编译方便，我们编写一个 Makefile 文件来进行编译，如下所示：

```
obj-m:=syscall.o
PWD:= $(shell pwd)
KERNELDIR:= /lib/modules/$(shell uname -r)/build
EXTRA_CFLAGS= -O0

all:
    make -C $(KERNELDIR)  M=$(PWD) modules
clean:
    make -C $(KERNELDIR) M=$(PWD) clean
```

要注意添加 `EXTRA_CFLAGS= -O0` 关闭 gcc 优化选项，避免插入模块出错。

### 5. 测试程序

现在，我们编写一个测试程序来测试一下系统调用拦截是否成功，代码如下：

```
#include <syscall.h>
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    unsigned long ret = syscall(__NR_perf_event_open, NULL, 0, 0, 0, 0);
    printf("%d\n", (int)ret);
    return 0;
}
```

### 6. 运行结果

#### 第一步：安装拦截内核模块

使用以下命令安装内核模块：

```
root# insmod syscall.ko
```

然后通过 `dmesg` 命令来观察系统日志，可以看到以下输出：

```
...
[  133.564652] Hack syscall is starting...
```

这说明我们的内核模块安装成功。

#### 第二步：运行测试程序

接着，我们运行刚才编写的测试程序，然后观察系统日志，输出如下：

```
...
[  532.243714] Hack syscall is successful!!!
```

这说明拦截系统调用成功了。