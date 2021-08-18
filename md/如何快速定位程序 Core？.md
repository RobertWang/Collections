> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653556773&idx=1&sn=1a30b9449fdbfec949fc3215e7c9f78e&chksm=813981bdb64e08ab5d5a635bc0c76d2e15d04c030106dbcc328c17fc54daab3b4c6445ef73df&mpshare=1&scene=1&srcid=0817Pc4sc5xTrI3GGQFLAMjO&sharer_sharetime=1629189190084&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

导读：程序 core 是指应用程序无法保持正常 running 状态而发生的崩溃行为。程序 core 时会生成相关的 core-dump 文件，是程序崩溃时程序状态的数据备份。core-dump 文件中包含内存、处理器、寄存器、程序计数器、栈指针等状态信息。本文将介绍一些利用 core-dump 文件定位程序 core 原因的方法和技巧。  

_全文 7023 字，预计阅读时间 13 分钟。_  

**一、程序 Core 定义及分类**
===================

程序 core 是指应用程序无法保持正常 running 状态而发生的崩溃行为。程序 core 时会生成相关的 core-dump 文件，core-dump 文件是程序崩溃时程序状态的状态数据备份。core-dump 文件包含内存、处理器、寄存器、程序计数器、栈指针等状态信息。我们可以借助 core-dump 文件来分析定位程序 Core 的原因。

这里我们从三个方面对程序 Core 进行分类：机器、资源、程序 Bug。下表对常见的 Core 原因进行了分类：

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0mDUyYTnGM1ic0nZ1MrBzBxaaERjZc4YbtR8OmbfiavEbGXTHXdGNVPow/640?wx_fmt=png)

**二、函数栈介绍**
===========

当我们打开 core 文件时，首先关注的是程序崩溃时的函数调用栈状态，为了方便理解后续定位 core 的一些技巧，这里先简单介绍一下函数栈。

**2.1 寄存器介绍**
-------------

目前生产环境都为 64 位机，这里只介绍 64 位机的寄存器，如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0o2qUxK1Jj1Pgqdyw0AWsnhLiaHP3ibuNn7gBdPjl3az9krHKScNqe6MA/640?wx_fmt=jpeg)

对于 x86-64 架构，共有 16 个 64 位寄存器，每个寄存器的用途并不单一，如 %rax 通常保存函数返回结果，但也被应用于 imul 和 idiv 指令。这里重点关注 %rsp（栈顶指针寄存器）、%rbp（栈底指针寄存器）、%rdi、%rsi、%rdx、%rcx、%r8、%r9（分别对应第 1～6 函数参数）。

Callee Save 说明是否需要被调用者保存寄存器的值。

**2.2 函数调用**
------------

![](https://mmbiz.qpic.cn/mmbiz_jpg/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0NQoDTyy3T8W3rxIJaylKMHibF182MrFwvPxpgh4qbviaIHtE3523Vujw/640?wx_fmt=jpeg)

### **2.2.1 调用函数栈帧：**

在调用一个函数时首先进行的是**参数压栈**，参数压栈的顺序跟参数定义的顺序相反。注意，**并不是参数一定会压栈**，在 x86-64 架构中会针对可以使用寄存器传递的变量，直接通过寄存器传值，如数字、指针、引用等。

接着是**返回地址压栈，**返回地址为被调用函数执行完后，调用函数执行的下一个指令地址。这里牢记返回地址的位置，后续章节会利用到这个返回地址的特性。

针对上面的介绍举个例子说明：

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0D1qmAyw1llQNd4hIoVWk7XOez400Soib6qYNn72U5Sm9cPk1ryBWY7A/640?wx_fmt=png)

如上图，在 main 函数中调用了 foo 函数，首先对参数压栈，三个参数都可以直接用寄存器传递（分别对应 %edi、%esi、%edx），然后 call 指令将下一个指令压栈。

### 2.2.2 被调用函数栈帧：

被调用函数首先会将上一个函数的栈底指针（%rbp）保存，即 %rbp 压栈。然后再保存需要被保存的寄存器值，即 Callee Save 为 True 的寄存器。接着为临时变量、局部变量申请栈空间。

针对被调用函数，举个例子说明：

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0JVibKGcaCgPp3XWSNycST4AsoEc5VYNZclZv9Kog6drpXVqdBbsm7sA/640?wx_fmt=png)

如上图，在 foo 函数执行时，先对 main 函数的 %rbp 压栈，再把寄存器中的参数值存放到局部变量 (a, b, c) 中。

**2.3 总结**
----------

通过对函数调用的简单介绍，我们可以发现函数栈是一个缜密且脆弱的结构，内存结构必须按照严格的方式被访问，如稍有不慎就可能导致程序崩溃。  

**三、GDB 定位 Core**
=================

这一节将介绍从 core 文件打开到定位全流程中可能会遇到的问题以及解决技巧。

**3.1 Core 文件**
---------------

### core 文件在哪里？

查看 “/proc/sys/kernel/core_pattern” 确定 core 文件生成规则。

**3.2 变量打印**  

程序 debug 过程中常常要查看各种变量（内存、寄存器、函数表等）的值是否正确，维持单独用一节介绍下常用的变量打印方法以及一些冷门小技巧。

### **3.2.1 print 命令**

```
print [Expression]
print $[Previous value number]
print {[Type]}[Address]
print [First element]@[Element count]
print /[Format] [Expression]
Format格式：
o - 8进制
x - 16进制
u - 无符号十进制
t - 二进制
f - 浮点数
a - 地址
c - 字符
s - 字符串
```

### **3.2.2 x 命令**

```
x /<n/f/u>  <addr>
n:是正整数，表示需要显示的内存单元的个数，即从当前地址向后显示n个内存单元的内容，
一个内存单元的大小由第三个参数u定义。
f:表示addr指向的内存内容的输出格式，s对应输出字符串，此处需特别注意输出整型数据的格式：
  x 按十六进制格式显示变量.
  d 按十进制格式显示变量。
  u 按十进制格式显示无符号整型。
  o 按八进制格式显示变量。
  t 按二进制格式显示变量。
  a 按十六进制格式显示变量。
  c 按字符格式显示变量。
  f 按浮点数格式显示变量。
u:就是指以多少个字节作为一个内存单元-unit,默认为4。u还可以用被一些字符表示:
  如b=1 byte, h=2 bytes,w=4 bytes,g=8 bytes.
<addr>:表示内存地址。
```

### **3.2.3 容器对象打印**

利用上面的 print 和 x 命令，再结合容器的数据结构，我们就能知道容器的详细信息。这里举个完整打印二进制 string 的例子，string 的数据结构如下：

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI03wcUZt1ibVLAZUl8RRyFBQIuoxOibLU2cgjwxqYEPNibj7V3IRwI5eyAg/640?wx_fmt=png)

string 为空时，_M_dataplus._M_p 是指向 nullptr 的。当赋值后会在堆上申请一段内存，分为两段，前半段是 meta 信息（类型为 std::string::_Rep），如 length、capacity、refcount，后半段为数据区，_M_p 指向数据区。

通常情况下非二进制的 string，直接 print 即可显示数据内容，但当数据为二进制时，'\0'会截断打印内容。因此，打印二进制 string 的首要任务是确认 string 的 size。

string 的 size 信息保存在 std::string::_Rep 结构体中，根据上面的数据结构可以发现，_Rep 与_M_dataplus._M_p 相差一个结构体大小，因此打印_Rep 结构体的命令为：

```
#先把_M_p转成_Rep指针，再让指针向低地址偏移一个结构体大小
p *((std::string::_Rep*)(s._M_dataplus._M_p) - 1)
```

找到 string 的 size（_M_length）后，再通过 x 命令打印相关的内存区即可，命令为：

```
#这里的n是_Rep._M_length
x /ncb s._M_dataplus._M_p
```

运行效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0Jt5X0t4JNgI5aCZTIicYS6q3WkibgFs5cRj18yhLQ72MqNT2LzLsdClg/640?wx_fmt=png)

为了方便，这里推荐一个方便的脚本：stl-views.gdb（链接：https://sourceware.org/gdb/wiki/STLSupport?action=AttachFile&do=view&target=stl-views-1.0.3.gdb，直接在 gdb 终端 **source stl-views.gdb** 即可，支持常见的容器打印，如 vector、map、list、string 等。

### **3.2.4 静态变量打印**

程序中经常会使用到静态变量，有时我们需要查看某个静态对象的值是否正确，就涉及到静态对象的打印。看如下例子：

```
void foo() {
    static std::string s_foo("foo");
}
```

这里可以借助 **nm -C** ./bin  | grep xx 找到静态变量的**内存地址**，再通过 gdb 的 print 打印。  

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI03zMma4Mdcyib98FkWJhV4NIwKRVGq5g6kqUYMyBYE0SRycLkFiaia0gJw/640?wx_fmt=png)

### 3.2.5 内存 dump

```
dump [format] memory filename start_addr end_addr
dump [format] value filename expr
format一般使用binary，其他的可以查看gdb手册。
比如我们可以结合上面查看string内容的例子dump整个string数据到文件中。
dump binary memory file1 s._M_dataplus._M_p s._M_dataplus._M_p + length
如果想查看文件内容的话可把vim -b和xxd结合使用。
```

接上面 string 的例子，举一个 dump string 内存数据到文件的例子：

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI07ohgu7gicDVAAzI9SgFOz9RbOsibGI0e411Rt9GJaV3zD5s4icYnFVia6g/640?wx_fmt=png)

**3.3 定位代码行**
-------------

定位 core 的原因，首先要定位崩溃时正在执行的代码行，这一节主要介绍一些定位代码行的方法。通常情况下直接通过 gdb 的 breaktrace 即可一览整个函数栈，但有时候函数栈信息并非如此清晰明了，这时就可利用一些小技巧来查看函数栈。

**3.3.1 去编译优化**
---------------

有时候会发现 core 的函数栈跟实际的代码行不匹配，如果是在线下环境中，可以尝试把编译优化设置成 **-O0**，然后再重新复现 core 问题。

### **3.3.2 程序****计数器 + addr2line**

对于线上 core 问题，一般没法再对程序进行去编译优化操作，只能在现有的 core 文件基础上进行代码定位。这一节我们采用一个例子来介绍如何使用**程序计数器 + addr2line** 来定位代码行。

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0L17u9vCtzEaXxF0UBTN9kb8hqGibknaGzMgeKTUz8wVTH9WcslHJLyQ/640?wx_fmt=png)

从截图可以发现 frame 20 指示的代码行与实际的代码行是不匹配的，定位步骤如下：

```
# 跳转到第20号栈
frame 20
# 使用display命令显示程序计数器
display /i $rip
# 使用addrline工具做地址转换
shell /opt/compiler/gcc-8.2/bin/addr2line -e bin address
```

**3.3.3 函数栈修复**

有时候我们会发现函数调用栈里面会出现很多？？的情况，这常发生于栈被写花，某些情况下手动进行修复。函数栈的修复利用的函数栈内存分布知识，见第一节。

```
-----------------------------------
Low addresses
-----------------------------------
0(%rsp)  | top of the stack frame 
         | (this is the same as -n(%rbp))
---------|-------------------------
-n(%rbp) | variable sized stack frame
-8(%rbp) | varied
0(%rbp)  | previous stack frame address
8(%rbp)  | return address
-----------------------------------
High addresses
```

从上面的栈示意图可以发现，利用 %rbp 寄存器即可找到上一个**函数的返回地址**和**栈底指针，**再利用 **addr2line** 命令找到对应的代码行。这里举一个例子：

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0J9zbGVNicqouwC15Kc49IcKBwE62WpZhESwJxYQlfufIQfG9oltAhGg/640?wx_fmt=png)

```
#首先找到当前被调用栈上一个栈的栈底指针值和返回地址
x /2ag $rbp # 2个单位，a=十六进制，g=8字节单元
#使用上一条命令得到的栈底指针值依次递归
x /2ag address
```

### **3.3.4 无规律 core 栈**

无规律 core 栈问题一般发生于堆内存写坏。函数调用是一个非常精密的过程，任何一个位置发生非预期的读写都会导致程序崩溃。这里可以举个小例子来说明：

```
int main(int argc, char* argv[]) {
    std::string s("abcd");
    *reinterpret_cast<uint64_t*>(&s) = 0x11;
    return 0;
}
```

上面的例子 core 在 string 析构上，原因是因为 string 的_M_ptr 被改写成了 0x11，析构流程变成了非法内存操作。

同理，由于进程堆空间是共享的，一个线程对堆的非法操作就可能会影响另一个线程的正常操作，由于堆分配的随机性，表现出来的现象就是无规律 core 栈。

针对无规律 core 栈最好的方式还是借助 AddressSanitizer。

```
#设置编译参数CXXFLAGS
CXXFLAGS="-fPIC -fsanitize=address  -fno-omit-frame-pointer"
#设置链接参数
LDFLAGS="-lasan"
# 设置启动环境变量
export ASAN_OPTIONS=halt_on_error=0:abort_on_error=1:disable_coredump=0
# 启动
LD_PRELOAD=/opt/compiler/gcc-8.2/lib/libasan.so ./bin/xxx
```

**3.3.5 总结**

上面提到的几种方法都是为了找到具体的问题代码行，为后续分析 core 的具体原因提供线索。

**3.4 定位 Core 原因**  

---------------------

这一节主要介绍定位 Core 原因的方法以及一些常见原因的介绍。

**3.4.****1 确认信号量**
-------------------

从上面的 Core 分类我们可以发现某些场景的 core 是由于机器故障导致的，如 SIGBUS，因此可以先通过信号量排除掉一些 core 原因。

**3.4.****2 定位异常汇编指令**
----------------------

通过上面的代码行定位我们可以大致找到程序 core 在哪一行，比较简单的 core 直接 print 程序上下文即可找到 core 的原因。

但有些场景下，通过排查上下文无任何异常，这个时候就需要准确定位具体的异常汇编指令，根据指令找原因。

查看汇编指令比较简单的方法是使用 **layout asm** 命令，frame 指向那个栈，就显示对应栈的汇编。这里举个 core 例子，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0X9leJemkDxl0Pcjibp0okq9I5dDuwxuia6UFYSf44KO5xbuA425lib5NA/640?wx_fmt=png)

程序显示 core 在 start 函数，查看相关上下文变量均无异常。使用 layout asm 打开正在执行的汇编指令，如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0gvicLiawcbE0aR8oN5XbxCkeh1AnCibIKNAgwcm2ibF10iaaibjEicKWHdVTw/640?wx_fmt=png)

查看汇编定位到程序 core 在 mov 指令，mov 指令上一个指令为 sub，为栈申请了 3M 空间，怀疑是栈空间不足。采用 frame 0 的 %rsp - frame N 的 %rbp 排查为栈空间不足。

通过上面的例子，可以发现定位异常汇编指令位置后，我们能够把异常点进一步压缩，定位到是哪个**指令、变量、地址**导致的 core 问题。

### **3.4.3 排查异常变量**

通过上面的操作我们可以准确定位到具体是哪一行代码的哪一条指令出现了问题，根据异常指令我们可以排查相关的变量，确定变量值是否符合预期。

这里举一个比较经典的空指针例子，如下：

```
int main(int argc, char* argv[]) {
    int* a = nullptr;
    *a = 1;
    return *a;
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0JXuCA9dg4mqnWqDqJbUEbiaKKuwc3IT2XKY6GC5G8n6May197spIAQA/640?wx_fmt=png)

通过汇编指令我们可以发现是 **movl $0x1, (%rax)** 出现了问题，**%rax** 的值来自于 **0x8(%rbp)**，**x 命令**打印相关的地址就可以发现为空指针错误。

### **3.4.4 查看被优化变量**

通常情况下程序都是开启了编译优化的，就会出现变量无法被 print，提示变量被优化，有时可利用**汇编 + 寄存器**的方式查看被优化的变量。

这里举一个例子说明下：

```
void foo(char const* str) {
    char buf[1024] = {'\0'};
    memcpy(buf, str, sizeof(buf));
}
int main(int argc, char* argv[]) {
    foo("abcd");
    return 0;
}
```

通常情况下在 foo 函数内部，str 变量是会直接别优化掉的，因为可以直接利用 %rdi 寄存器传递参数。为了能够打印出 str 的值，这个时候我们可以借助**汇编 + 寄存器**的方式找到具体的变量值，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0hCdQib1WLTBDVNB1ia6icKmk0kGfv1ia1b99lKH19J9OWfq6PSxlicV9kuw/640?wx_fmt=png)

首先找到 main 函数调用 foo 函数的参数压栈汇编：**mov $0x402011, %edi，**这里的 **0x402011** 即为 str 的内存地址，通过 x 命令即可显示 str 的值了。

比较复杂的场景可能没法直接找到被优化变量，这时可以采用汇编回溯的方式找到变量。

### **3.4.5 异常函数地址排查**

有时的 core 问题是因为数据异常导致，有时也可能是优化函数地址导致，如调用虚函数地址错误、函数返回地址错误、函数指针值错误。

异常函数地址排查同理于异常变量排查，根据汇编指令确认调用是否异常即可。这里举一个虚函数地址异常的例子，如下：

```
class
A {
public:
    virtual ~A() = default;
    virtual void foo() = 0;
};
class
B : public A {
public:
    void foo() {}
};
int main(int argc, char* argv[])
{
    A* a = new B;
    a->foo();
    A* b = new B;
    *reinterpret_cast<void**>(b) = 0x0;
    b->foo();
    return 0;
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/5p8giadRibbOibDX6NibdPZkzcg8bTFC8yI0Cg36zLkgUFHDagyan4HX32ERZibzFk6tkR25lpcdX4NgvvmjQiapmniaw/640?wx_fmt=png)

从汇编指令看是 core 在了 mov (%rax), %rax，结合指令上下文可发现是在虚函数地址寻址操作，对比两个变量的虚函数表即可发现是函数地址 load 错误导致的 core。

### **3.4.6 总结**

定位 core 的基本流程可总结为以下几步：

1.  明确 core 的大致触发原因。机器问题？自身程序问题？
    
2.  定位代码行。哪一行代码出现了问题。
    
3.  定位执行指令。哪一行指令干了什么事。
    
4.  定位异常变量。指令不会有问题，是指令操作的变量不符合预期。
    

善于利用**汇编指令**以及**打印指令（x、print、display）**可以更有效的定位 Core。

**参考资料：**
=========

汇编查看工具：https://godbolt.org/ https://cppinsights.io/  
标准 GDB 文档：https://sourceware.org/gdb/current/onlinedocs/gdb/