> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/f8R87wZ2_dInyFQE9379Lw)

> 一、打桩介绍 桩或称桩代码，是指用来代替关联代码或者未实现代码的代码。如果函数 B 用 B1 来代替，那么，B 称为原

**一、打桩介绍**

 桩或称桩代码，是指用来代替关联代码或者未实现代码的代码。如果函数 B 用 B1 来代替，那么，B 称为原函数，B1 称为桩函数。打桩就是编写或生成桩代码。

**1.****1 目的**

函数打桩的目的一般是隔离、占位和控制。

1.   隔离是指将测试任务从产品项目中分离出来，使之能够独立编译、链接，并独立运行。隔离的基本方法就是打桩，将测试任务之外的，并且与测试任务相关的代码，用桩来代替，从而实现分离测试任务。例如函数 A 调用了函数 B，函数 B 又调用了函数 C 和 D，如果函数 B 用桩来代替，函数 A 就可以完全割断与函数 C 和 D 的关系
    
2.  补齐是指用桩来代替未实现的代码，例如，函数 A 调用了函数 B，而函数 B 由其他程序员编写，且未实现，那么，可以用桩来代替函数 B，使函数 A 能够运行并测试。补齐在并行开发中很常用。
    
3.  控制是指在测试时，人为设定相关代码的行为，使之符合测试需求。
    

Golang 里打桩的库也有很多，这边只介绍 gomonkey。

**二**、gomonkey****

gomonkey 是 golang 的一款打桩框架，目标是让用户在单元测试中低成本的完成打桩，从而将精力聚焦于业务功能的开发。gomonkey 接口友好，功能强大，目前已被很多项目使用，用户遍及世界多个国家。

**2.****1 特性**

1.  支持为一个函数打一个桩
    
2.  支持为一个成员方法打一个桩
    
3.  支持为一个全局变量打一个桩
    
4.  支持为一个函数变量打一个桩
    
5.  支持为一个函数打一个特定的桩序列
    
6.  支持为一个成员方法打一个特定的桩序列
    
7.  支持为一个函数变量打一个特定的桩序列
    

**2.2** **方法说明**

<table width="802" class=""><tbody><tr><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p><strong>ApplyFunc(target, double interface{}</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>为<strong>函数</strong> / <strong>接口</strong>打一个桩</p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>target 表示函数名，第二个参数表示桩函数。</p></td></tr><tr><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);" class=""><p><strong>ApplyFuncSeq(target interface{}, outputs []OutputCell）</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>为<strong>函数</strong> / <strong>接口</strong>打一个特定的<strong>桩序列</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>target 表示函数名，第二个参数表示桩序列参数（返回值需序列）。</p></td></tr><tr><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p><strong>ApplyMethod(target reflect.Type, methodName string, double interface{}）</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>为<strong>成员方法</strong>打一个桩</p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>target 表示对象类型，对象的方法名，第三个参数表示桩函数。</p></td></tr><tr><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p><strong>ApplyMethodSeq(target reflect.Type, methodName string, outputs []OutputCell）</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>为<strong>成员方法</strong>打一个特定的<strong>桩序列</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>target 表示对象类型，对象的方法名，第三个参数表示桩序列参数（返回值需序列）。</p></td></tr><tr><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p class=""><strong>ApplyFuncVar(target, double interface{}）</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>为<strong>函数变量</strong>打一个桩</p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>target 表示函数变量，第二个参数表示桩函数。</p></td></tr><tr><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p><strong>ApplyFuncVarSeq(target interface{}, outputs []OutputCell）</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>为<strong>函数变量</strong>打一个特定的<strong>桩序列</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>target 表示函数变量，第二个参数表示桩序列参数（返回值需序列）。</p></td></tr><tr><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p><strong>ApplyGlobalVar(target, double interface{})</strong></p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>为<strong>全局变量</strong>打一个桩</p></td><td data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>target 表示函数变量，第二个参数表示桩函数。</p></td></tr><tr><td height="27" data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);" class=""><p><strong>Reset()</strong></p></td><td height="27" data-style="padding: 12px 16px; list-style: inherit; border-color: rgb(225, 230, 240);"><p>删除桩</p></td><td height="27"><br></td></tr></tbody><thead><tr><th data-style="padding: 12px 16px; list-style: inherit; border-top-width: 1px; border-color: rgb(225, 230, 240); background-color: rgb(245, 247, 250); color: rgb(103, 116, 137); text-align: left; font-weight: normal; word-break: keep-all;"><p class="">方法</p></th><th data-style="padding: 12px 16px; list-style: inherit; border-top-width: 1px; border-color: rgb(225, 230, 240); background-color: rgb(245, 247, 250); color: rgb(103, 116, 137); text-align: left; font-weight: normal; word-break: keep-all;"><p>作用</p></th><th data-style="padding: 12px 16px; list-style: inherit; border-top-width: 1px; border-color: rgb(225, 230, 240); background-color: rgb(245, 247, 250); color: rgb(103, 116, 137); text-align: left; font-weight: normal; word-break: keep-all;"><p>函数使用说明</p></th></tr></thead><tbody></tbody></table>

****2.3** **工作原理****

gomonkey 是为函数、变量打桩，但是对于函数以及方法的模拟替换，在 Go 这种静态强类型语言中不太容易，因为我们的代码逻辑已经是声明好的，因此，我们很难通过编码的方式将其替换掉。

所以，gomonkey 提供了让我们在运行时替换原函数 / 方法的能力。虽然说我们在语言层面很难去替换运行中的函数体，但是代码最终都会转换成机器可以理解的汇编指令，因此，我们可以通过创建汇编指令来改写函数。

在 gomonkey 打桩的过程中，其核心函数其实是 ApplyCore。不管是对函数打桩还是对方法打桩，实际上最后都会调用这个 ApplyCore 函数，如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/caBJ5fWjGku7FeGEOwNURUIia0GnrUbmdPwLB9zhxIOlHQNwDjp7ZxWmhopd580OsfJBF51iaGqPU64bpC7IVC7A/640?wx_fmt=png)

ApplyCore 函数的具体实现如下：

```
func (this *Patches) ApplyCore(target, double reflect.Value) *Patches {
    this.check(target, double)
    if _, ok := this.originals[target]; ok {
        panic("patch has been existed")
    }
    this.valueHolders[double] = double
    original := replace(*(*uintptr)(getPointer(target)), uintptr(getPointer(double)))
    this.originals[target] = original
    return this
}

```

可以看到，获取到传入的原始函数和替换函数做了一个 replace 的操作，这里就是替换的逻辑所在了。replace 函数原型如下：

```
func replace(target, double uintptr) []byte {
    code := buildJmpDirective(double)
    bytes := entryAddress(target, len(code))
    original := make([]byte, len(bytes))
    copy(original, bytes)
    modifyBinary(target, code)
    return original
}

```

buildJmpDirective 构建了一个函数跳转的指令，把目标函数指针移动到寄存器 rdx 中，然后跳转到寄存器 rdx 中函数指针指向的地址。之后通过 modifyBinary 函数，先通过 entryAddress 方法获取到原函数所在的内存地址，之后通过 syscall.Mprotect 方法打开内存保护，将函数跳转指令以 bytes 数组的形式调用 copy 方法写入到原函数所在内存之中，最终达到替换的目的。此外，这里 replace 方法还保留了原函数的副本，方便后续函数 mock 的恢复。

****2.4** **限制****

gomonkey 作为一个打桩的工具，使用场景还是比较广泛，可以使用我们大部分的应用场景。但是，它依然还是有很多限制，它必须要找到该方法对应的真实的类（结构体）：

*   gomonkey 必须禁用 golang 编译器的内联优化，不然函数被内联之后，就找不到接缝了，stub 无法进行。一般我们是通过 go test 的时候带上 '-gcflags=all=-N -l' 来禁用内联优化。
    

1.  内联优化一般用于能够快速执行的函数，因为在这种情况下函数调用的时间消耗显得更为突出，同时内联体量小的函数也不会明显增加编译后的执行文件占用的空间。Go 中，函数体内包含：闭包调用，select ，for ，defer，go 关键字的的函数不会进行内联。并且除了这些，还有其它的限制。
    

*   gomonkey 需要很高的系统权限，因为在运行时替换函数入口是一个权限要求较高的事情，在一个安全的系统上，比如在 10.15 + 的 macos 上，这一点就是很难做到的。
    
*   gomonkey 不支持异包未导出函数的打桩、不支持同包未导出方法的打桩