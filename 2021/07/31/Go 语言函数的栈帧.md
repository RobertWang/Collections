> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666556163&idx=3&sn=52add1782c4f0a0a8d2a27b30f40ff91&chksm=80dcafa8b7ab26be2cc0e20877f752c18a785d29574213b75e7d3b26976a3944fdd72501bfbe&mpshare=1&scene=1&srcid=0727thkpOXy8sttdBSMnYBS4&sharer_sharetime=1627373324395&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

【导读】什么是函数栈帧？这个特征是否是开发者在编程中需要关注的点？本文做了详细介绍。  

随着函数的层层调用与返回，运行时栈上的函数栈帧也随之分配和释放。实际管理栈帧的是函数自身的代码，就是编译阶段由编译器生成的指令，所以也可以说函数栈帧是由编译器管理的。

栈帧构成
----

参照下面的函数栈帧布局示意图，从空间分配的角度来看，函数的栈帧包含以下几个部分：

1.  return address：函数的返回地址，占用一个指针大小的空间。实际上是在函数被调用时由 CALL 指令自动压栈的，并非由被调用函数分配；
    
2.  caller’s BP：调用者的栈帧基址，占用一个指针大小的空间，有些情况下会被优化掉。用来将调用路径上所有的栈帧连成一个链表，方便栈回溯之类的操作。函数通过将栈指针 SP 直接向下移动指定大小，来一次性分配 caller’s BP、locals 和 args to callee 所占用的空间，在 x86 架构上就是使用 SUB 指令将 SP 减去指定大小；
    
3.  locals：局部变量区间，占用若干机器字。用来存放函数的局部变量，根据函数的局部变量占用空间大小来分配，没有局部变量的函数不分配；
    
4.  args to callee：调用传参区域，占用若干机器字。分配空间大小，根据当前函数发起的所有的函数调用中，返回值加上参数所占用的空间最大的，按此来分配。没有调用任何函数时，不需要分配该区间。在 callee 视角的 args from caller 区间，包含在 caller 视角的 args to callee 区间内，占用空间大小是小于等于的关系。
    

![](https://mmbiz.qpic.cn/mmbiz_svg/22YD2oBcVUZPr4R2P4qAdmFQnicWuUOy2xzd9xuqCKeEicByVdVI8zmf8WCHW0kP3KKO8ibceeLBR5c53wOE0RlMsXKr8HzwMnu/640?wx_fmt=svg)

Stack Frame Layout

综上所述，只有 return address 是一定会存在的，其他 3 个区间都要根据实际情况进行分析。

代码示例
----

下面就结合实际代码，具体说明函数栈帧各区间的分配和使用情况。编译运行如下示例代码：

```
package main

func main() {
        var v1, v2 int
        v3, v4 := f1(v1, v2)
        println(&v1, &v2, &v3, &v4)
        f2(v3)
}

func f1(a1, a2 int) (r1, r2 int) {
        var l1, l2, l3 int
        println(&r2, &r1, &a2, &a1, &l1, &l2, &l3)
        return
}

func f2(a1 int) {
        println(&a1)
}
```

在笔者使用的 amd64+linux 环境，得到的输出如下所示：

```
$ go build -gcflags='-l'
$ ./stack_frame
0xc000038750 0xc000038748 0xc000038740 0xc000038738 0xc000038720 0xc000038718 0xc000038710
0xc000038770 0xc000038768 0xc000038760 0xc000038758
0xc000038738
```

编译时通过指定参数来防止编译器将小函数内联优化掉，那样就不存在真正的栈帧结构了。

3 行输出依次是由 f1、main、f2 中的 println 打印的，所以可以以此为参照，画出栈帧布局图。下面先分别进行梳理：

### println

代码里之所以使用 println，而没有使用 fmt.Printf 之类的函数，是因为前者更底层更 “简单”，不会造成变量逃逸等问题，所以不会带来不必要的干扰。

实际上，代码中的 println 会被编译器转换为多次调用 runtime 包中的 printlock、printunlock、printpointer、printsp、printnl 函数，前两个函数用来进行并发同步，后 3 个用来打印指针、空格和换行，这 5 个函数均无返回值，只有 printpointer 有一个参数。

例如：

```
var a, b int
println(&a, &b)
```

会被转换为：

```
runtime.printlock()      // 获得锁
runtime.printpointer(&a) // 打印指针
runtime.printsp()        // 打印空格
runtime.printpointer(&b) // 打印指针
runtime.printnl()        // 打印换行
runtime.printunlock()    // 释放锁
```

所以这一组函数调用只需要一个机器字的空间，用来向 printpointer 传参。

### 栈帧布局

根据以上的示例代码，以及编译运行的输出，对 3 个函数的栈帧上各区间大小进行整理：

<table data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)"><thead data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)"><tr data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="font-size: 16px; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: left;"><br data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)|rgb(240, 240, 240)"></th><th data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="font-size: 16px; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: left;">caller’s BP</th><th data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="font-size: 16px; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: left;">locals</th><th data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="font-size: 16px; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: left;">args to callee</th><th data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="font-size: 16px; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: left;">分配大小</th></tr></thead><tbody data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)"><tr data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">main</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">1 个指针大小</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">4 个 int 大小：v1、v2、v3、v4</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">4 个 int 大小：调用 f1</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">0x48</td></tr><tr data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(248, 248, 248)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">f1</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(248, 248, 248)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">1 个指针大小</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(248, 248, 248)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">3 个 int 大小：l1、l2、l3</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(248, 248, 248)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">1 个 int 大小：println</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(248, 248, 248)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">0x28</td></tr><tr data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">f2</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">1 个指针大小</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">无</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">1 个 int 大小：println</td><td data-darkmode-color-16277437149895="rgb(163, 163, 163)" data-darkmode-original-color-16277437149895="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16277437149895="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16277437149895="#fff|rgb(255,255,255)" data-style="font-size: 16px; border-color: rgb(204, 204, 204); min-width: 85px;">0x10</td></tr></tbody></table>

对照以上表格，绘制栈帧布局图：

![](https://mmbiz.qpic.cn/mmbiz_svg/22YD2oBcVUZPr4R2P4qAdmFQnicWuUOy2uY5icHoJiaiaVMsQVL29vkVSHUdcsOOPjuNEU9zABgvNZKrOPKfPBoXA89UT7KXVRwx/640?wx_fmt=svg)

Real Stack Layout

左侧是调用 f1 时的运行时栈，右侧是调用 f2 时的运行时栈。

通过 f1 的调用栈，可以发现函数的返回值和参数是按照先返回值后参数，并且是从右至左的顺序在栈上分配的，与 C 语言时期的参数入栈顺序一致。这是因为 f1 的参数和返回值占满了整个 args to callee 区间。

值得注意的是 f2 的调用栈，在 a1 和 v4 之间是空了 3 个机器字的，因为 Go 语言的函数是固定栈帧大小，args to callee 是按照所需的最大空间来分配。调用函数时，参数和返回值看起来更像是按照先参数后返回值，从左到右的顺序分配在 args to callee 区间里，并且是从低地址开始使用。这点与我们对传统的栈的理解有些不同，更符合传统的栈原理的如 32 位的 VC++ 编译器，它使用 PUSH 指令动态入栈，args to callee 区间的大小不是固定的。Go 这种固定栈帧大小的分配方式，使得像调试、运行时栈扫描之类更易于实现，但是会造成更大的栈消耗。

函数的参数和返回值的传递，属于 “调用约定” 的范畴，是 compiler 和 linker 在进行构建时内部遵循的一致性规范。可以参考 C 语言的调用约定，来进行类比学习。

在函数内部访问运行时栈上的返回值、参数和局部变量时，通过栈指针 SP 加上相对偏移来寻址。函数栈帧的结构是编译器生成的，就隐含在函数的代码里面，有兴趣的同学可以自己看一下反编译后的汇编代码，自会一目了然。

> 转自：fengyoulin
> 
> fengyoulin.com/func-stack-frame.html

- EOF -