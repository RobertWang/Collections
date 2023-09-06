> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/byouCtNulnjYDluDaO9wIw)

**一、介绍**

当数组越界、访问非法空间或者主动调用 panic 时，panic 会停掉当前正在执行的程序，包括所有协程，比起 exit 直接退出，panic 的退出更有秩序，它会先处理完当前 goroutine 已经 defer 挂上去的任务，执行完毕后再退出整个程序。

**二、执行 panic 底层到底发生什么？**

接下来我们通过一个案例汇编执行得出 panic 底层执行哪些操作，如下：

```
func main() {
  panic("hello panic")
}

```

执行输出:  

```
➜  go tool compile -S main.go
"".main STEXT size=66 args=0x0 locals=0x18
        0x0000 00000 (main.go:108)      TEXT    "".main(SB), ABIInternal, $24-0
        0x0000 00000 (main.go:108)      MOVQ    (TLS), CX
        0x0009 00009 (main.go:108)      CMPQ    SP, 16(CX)
        ...
        0x002f 00047 (main.go:120)      PCDATA  $2, $0
        0x002f 00047 (main.go:120)      MOVQ    AX, 8(SP)
        0x0034 00052 (main.go:120)      CALL    runtime.gopanic(SB)
        0x0039 00057 (main.go:120) 
              ...
        0x0024 00036 (main.go:111)      LEAQ    "".main.func1·f(SB), AX
        0x002b 00043 (main.go:111)      PCDATA  $2, $0
        0x002b 00043 (main.go:111)      MOVQ    AX, 8(SP)
        0x0030 00048 (main.go:111)      CALL    runtime.deferproc(SB)
        0x0035 00053 (main.go:111)      TESTL   AX, AX
        ...
        0x004b 00075 (main.go:116)      MOVQ    AX, 8(SP)
        0x0050 00080 (main.go:116)      CALL    runtime.gopanic(SB)
        0x0055 00085 (main.go:116)      UNDEF
        0x0057 00087 (main.go:111)      XCHGL   AX, AX
        0x0058 00088 (main.go:111)      CALL    runtime.deferreturn(SB)
        0x005d 00093 (main.go:111)      MOVQ    16(SP), BP
       ...
        0x0026 00038 (main.go:112)      MOVQ    AX, (SP)
        0x002a 00042 (main.go:112)      CALL    runtime.gorecover(SB)
        ... 
        0x0072 00114 ($GOROOT/src/fmt/print.go:208)     LEAQ    go.string."recovered:%v\n"(SB), AX
        ...
        0x00a3 00163 ($GOROOT/src/fmt/print.go:208)     CALL    fmt.Fprintf(SB)

```

**2.1 panic 结构**

```
type _panic struct {
  argp      unsafe.Pointer  //一个指针，指向defer调用的参数的指针
  arg       interface{}   //panic传入的参数
  link      *_panic   //是一个链表结构，指向上一个调用的_panic
  recovered bool  //是否已被recover 恢复
  aborted   bool   //panic是否被强行中止
}

```

说明：

*   link 字段：一个指向 _panic 结构体的指针，表明 _panic 和 _defer 类似，_panic 可以是一个单向链表，就跟 _defer 链表一样；
    
*   recovered 字段：重点来了，所谓的 _panic 是否恢复其实就是看这个字段是否为 true，recover( ) 其实就是修改这个字段；
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/caBJ5fWjGkuvRSCAxTIkg434libL7dhXBichVQgaUZfLwbJ8VKK6wucwLobLmT8IOZgw5N7RYlDynAExWXonzuibg/640?wx_fmt=png)

再看一下 goroutine 的两个重要字段：

从这里我们看出：_defer 和 _panic 链表都是挂在 goroutine 之上的

通过汇编得知 panic 的底层实现是 gopanic 的函数，位于 runtime/panic.go 文件。panic 机制**最重要**的就是 gopanic 函数了，所有的 panic 细节尽在此。为什么 panic 会显得晦涩，主要有两个点：

1.  嵌套 panic 的时候，gopanic 会有递归执行的场景
    
2.  程序指令跳转并不是常规的函数压栈，弹栈，在 recovery 的时候，是直接修改指令寄存器的结构体，从而直接越过了 gopanic 后面的逻辑，甚至是多层 gopanic 递归的逻辑
    

**2.2 gopanic 实现**

```
// The implementation of the predeclared function panic.
func gopanic(e interface{}) {
  gp := getg() //获取指向当前goroutine的指针
  ...
  //初始化一个panic的基本单元_panic
  var p _panic
  p.arg = e
   // 把当前最新的 _panic 挂到链表最前面
  p.link = gp._panic
  gp._panic = (*_panic)(noescape(unsafe.Pointer(&p)))
  //遍历当前goroutinue的defer链表
  for {
    //获取当前goroutinue上挂的defer
    d := gp._defer
    if d == nil {
      break
    }
    //下面是一些对此defer的判断和处理，是否开放，是否可执行等
    ...
    if d.started {
      if d._panic != nil {
        d._panic.aborted = true
      }
      d._panic = nil
      if !d.openDefer {
        // For open-coded defers, we need to process the
        // defer again, in case there are any other defers
        // to call in the frame (not including the defer
        // call that caused the panic).
        d.fn = nil
        gp._defer = d.link
        freedefer(d)
        continue
      }
    }
    ...
    d._panic = (*_panic)(noescape(unsafe.Pointer(&p)))
    //当前goroutinue若存在有效的defer调用，
    //就会调下面的reflectcall方法来处理defer后面的操作，
    //当然，若defer里面进行了recover处理，则会调用gorecover函数，
    p.argp = unsafe.Pointer(getargp(0))
    reflectcall(nil, unsafe.Pointer(d.fn), deferArgs(d), uint32(d.siz), uint32(d.siz))
    }
    p.argp = nil
    //在pc、sp中记录当前defer的pc、sp
    pc := d.pc
    sp := unsafe.Pointer(d.sp) // must be pointer so it gets adjusted during stack copy
    ...
    //已经有recover被调用
    if p.recovered {
         // 摘掉当前的 _panic
         gp._panic = p.link
         // 如果前面还有 panic，并且是标记了 aborted 的，那么也摘掉；
         for gp._panic != nil && gp._panic.aborted {
             gp._panic = gp._panic.link
         }
         // panic 的流程到此为止，恢复到业务函数堆栈上执行代码；
         gp.sigcode0 = uintptr(sp)
         gp.sigcode1 = pc
         // 注意：恢复的时候 panic 函数将从此处跳出，本 gopanic 调用结束，后面的代码永远都不会执行。
         mcall(recovery)
         throw("recovery failed") // mcall should not return
    }    
  }
  //准备panic程序退出时要打印的参数
  preprintpanics(gp._panic)
  //递归打印所有panic中的参数信息，并执行exit(2)来退出程序
  fatalpanic(gp._panic) // should not return
  *(*int)(nil) = 0      // not reached
}

```

上面逻辑可以拆分为循环内和循环外两部分去理解：

*   循环内：程序执行 defer，是否恢复正常的指令执行，一切都在循环内决定。循环内的事情拆解成：
    

1.  遍历 goroutine 的 defer 链表，获取到一个 _defer 延迟函数；
    
2.  获取到 _defer 延迟函数，设置标识 d.started，绑定当前 d._panic（用以在递归的时候判断）；
    
3.  执行 _defer 延迟函数；
    
4.  摘掉执行完的 _defer 函数；
    
5.  判断 _panic.recovered 是否设置为 true，进行相应操作；
    
6.  如果是 true 那么重置 pc，sp 寄存器（一般从 deferreturn 指令前开始执行），goroutine 投递到调度队列，等待执行；  
    
7.  重复以上步骤；
    

*   循环外：一旦走到循环外，说明 _panic 没人处理，认命吧，程序即将退出
    

调用 gopanic 函数，接着遍历当前 goroutinue 上挂载的 defer 链表，在执行 reflectcall 时碰到 recover，就执行 gorecover 函数处理，接下来看下 gorecover。

**2.3 gorecover 实现以及如何恢复 panic**

```
func gorecover(argp uintptr) interface{} {
  gp := getg()
  p := gp._panic
  if p != nil && !p.goexit && !p.recovered && argp == uintptr(p.argp) {
    p.recovered = true
    return p.arg
  }
  return nil
}

```

通过如上代码，得知 gorecover 得知当前 goroutinue 是否在 panic 的流程中，在的话就将 panic 的 recovered 字段置微为 true，否则直接返回 nil，所以说 recover 在普通的流程被调用时时没有任何作用的。

gorecover 函数仅仅改了 recovered 标记，那么 gouroutine 是怎么从 panic 返回的呢？

*   在 gopanic 函数中，执行 defer 链中所有的延迟函数，执行时遇到 recover 会使得_panic.recovered=true
    
*   调用 recovery 通过_defer.sp 和_defer.pc 跳转到 deferproc 的上下文中来执行，同时将返回值设置为 1
    

```
func recovery(gp *g) {
  // Info about defer passed in G struct.
  sp := gp.sigcode0
  pc := gp.sigcode1
  // d's arguments need to be in the stack.
  if sp != 0 && (sp < gp.stack.lo || gp.stack.hi < sp) {
    print("recover: ", hex(sp), " not in [", hex(gp.stack.lo), ", ", hex(gp.stack.hi), "]\n")
    throw("bad recovery")
  }
  // Make the deferproc for this d return again,
  // this time returning 1. The calling function will
  // jump to the standard return epilogue.
  gp.sched.sp = sp
  gp.sched.pc = pc
  gp.sched.lr = 0
  gp.sched.ret = 1
  //切换上下文
  gogo(&gp.sched)
}

```

*   最后根据 deferproc 的返回值，若为 0 的话，说明 deferproc 成功，可以继续处理后面的逻辑；当返回值为非 0 的情况时（就是 panic 恢复时从 recovery 函数跳转过来的），此时就会跳转到函数的最后，return 之前，继续执行 deferreturn。由于调用 recover 的 defer 已经从 defer 链表上摘掉了，所以可以继续执行之前没完成的 defer，并最终返回当前函数的调用者。
    

```
func deferproc(siz int32, fn *funcval) {
    ...
  sp := getcallersp()
  argp := uintptr(unsafe.Pointer(&fn)) + unsafe.Sizeof(fn)
  callerpc := getcallerpc()
  //newdefer从_defer池中找是否有可复用的_defer基本单元，若没有，就会malloc一个新的
  //并且将当前goroutinue的defer指向这个d,  新的defer总是会出现在最前面
  //这就是defer先入后出属性的原因所在，
  d := newdefer(siz)
    ...
  d.fn = fn
  d.pc = callerpc
  d.sp = sp
  switch siz {
  case 0:
    // Do nothing.
  case sys.PtrSize:
    *(*uintptr)(deferArgs(d)) = *(*uintptr)(unsafe.Pointer(argp))
  default:
    memmove(deferArgs(d), unsafe.Pointer(argp), uintptr(siz))
  }
  return0()
}

```

**三、Q&A**

**3.1 为什么 recover 一定要放在 defer 才生效？**

因为 _panic.recovered 字段的的修改时机，只能在 for 循环内的第三步执行 _defer 延迟函数。

**3.2 为什么 recover 只能当前协程有效？**

上面我们提到 gopanic 函数，里面 for 循环有如下代码：

```
    //获取当前goroutinue上挂的defer
    d := gp._defer
    // 步骤：执行 defer 函数
    reflectcall(nil, unsafe.Pointer(d.fn), deferArgs(d), uint32(d.siz), uint32(d.siz))
    // 步骤：执行完成，把这个 defer 从链表里摘掉；
    gp._defer = d.link

```

在 gopanic 里，只遍历执行当前 goroutine 上的 _defer 函数链条。所以，如果挂在其他 goroutine 的 defer 函数做了 recover ，那么没有丝毫用途。