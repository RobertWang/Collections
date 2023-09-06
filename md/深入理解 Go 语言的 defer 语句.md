> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Byt1DArDw2bowWIP5YxRlw)

> 在 Go 语言中，defer 语句用于延迟函数的调用，每次 defer 都会把一个函数压入栈中，函数返回前再把延迟的函数取出并执行。

今天我们将深入讨论 Go 语言中的一个非常重要且有趣的功能：defer 语句。我会尽力以逻辑清晰、简洁易懂的方式来解构这个主题，让我们开始吧。

### defer 语句基础

在 Go 语言中，defer 语句用于延迟函数的调用，每次 defer 都会把一个函数压入栈中，函数返回前再把延迟的函数取出并执行。

defer 语句是用于确保某个函数在当前函数执行完毕后，无论当前函数是否出现错误，都能被执行。这在处理系统资源，比如：文件句柄、网络连接关闭等等场景，非常实用。

```
func ReadTxt() {
    file, _ := os.Open("example.txt")
    defer file.Close()

    // 执行其它操作
}


```

在上述例子中，无论 "执行一些操作" 的部分是否引发错误，file.Close() 都将在 ReadTxt 函数返回前被调用，确保了文件资源的正确释放。

为了方便描述，我们把创建 defer 的函数称为主函数，defer 语句后面的函数称为延迟函数。

  
延迟函数可能有输入参数，这些参数可能来源于定义 defer 的函数，延迟函数也可能引用主函数用于返回的变量，也就是说延迟函数可能会影响主函数的一些行为。这些场景下，如果不了解 defer 的规则很容易出错，下面先介绍官方说的 defer 的三大原则。

### defer 三大规则

#### 规则一：延迟函数的参数在 defer 语句出现时就已经确定下来了

> 1.  _A deferred function’s arguments are evaluated when the defer statement is evaluated._
>     

```
package main

import "fmt"

func main() {
    i := 0
    defer fmt.Println(i)
    i++
}

// 打印 0 


```

在此示例中，当 Println 调用被延迟时，将计算表达式 “i”，延迟调用将在函数返回后打印 “0”。

也就是说参数 i 值在 defer 出现时就已经确定下来，实际上是拷贝了一份。后面对变量 i 的修改不会影响 fmt.Println() 函数的执行，因此仍然打印”0”。

注意：对于指针类型参数，规则仍然适用，只不过延迟函数的参数是一个地址值，这种情况下，defer 后面的语句对变量的修改可能会影响延迟函数。

```
package main

import "fmt"

func main() {
    arr := []int {1,2,3}
    fmt.Println(arr)
    defer fmt.Println(arr)
    arr[0] = 5
}

// 打印 [1 2 3]
// 打印 [5 2 3]


```

#### 规则二：延迟函数执行按后进先出顺序执行，即先出现的 defer 最后执行

> 2.  _Deferred function calls are executed in Last In First Out order after the surrounding function returns._
>     

```
package main

import "fmt"

func main() {
     for i := 0; i < 4; i++ {
        defer fmt.Println(i)
    }
}

// 打印
// 3
// 2
// 1
// 0


```

这个规则比较好理解，定义 defer 类似于入栈操作，执行 defer 类似于出栈操作。  
设计 defer 的初衷是简化函数返回时资源清理的动作，资源往往有依赖顺序，比如先申请 A 资源，再跟据 A 资源申请 B 资源，跟据 B 资源申请 C 资源，即申请顺序是：A—>B—>C，释放时往往又要反向进行。这就是把 deffer 设计成 LIFO 的原因。

注意：每申请到一个用完需要释放的资源时，立即定义一个 defer 来释放资源是个很好的习惯。

```
mu.Lock()
defer mu.Unlock()


```

#### 规则三：延迟函数可以读取并分配给返回函数的命名返回值

> 3.  _Deferred functions may read and assign to the returning function’s named return values._
>     

```
package main

import "fmt"

func main() {
    num := add()
    fmt.Println(num)
}

func add() (i int) {
    defer func() { i++ }()
    return 2
}

// 打印 3


```

在此示例中，延迟函数在周围函数返回后递增返回值 i 。因此，该函数返回 3 并打印出来。

定义了 defer 的函数，即主函数可能有返回值，返回值有没有名字没有关系，defer 所作用的函数，即延迟函数可能会影响到返回值。

若我们想要理解 defer（延迟函数）是如何影响主函数返回值的，先需要明白函数是如何返回的？

#### 函数返回过程

有一个事实必须要了解，关键字 return 不是一个原子操作，实际上 return 只代理汇编指令 ret，即将跳转程序执 行。例如语句 return i ，实际上分两步进行，即将 i 值存入栈中作为返回值，然后执行跳转，而 defer 的执行时机正是跳转前，所以说 defer 执行时还是有机会操作返回值的。

举个例子：

```
package main

import "fmt"

func main() {
    fmt.Println(deferFuncReturn())
}

// 说明 defer 如何影响返回值
func deferFuncReturn() (result int) { 
    i:=1

    defer func() {
        result++ 
    }()

    return i 
}


```

该函数的 return 语句可以拆分成下面两行：  

```
result = i
return


```

而延迟函数的执行正是在 return 之前，即加入 defer 后的执行过程如下：

```
package main

import "fmt"

func main() {
    fmt.Println(deferFuncReturn())
}

func deferFuncReturn() (result int) { 
    i:=1

    // return 语句 和 defer 语句的等价替换
    result = i 
    result++ 
    return
}


```

### defer 实现原理

前面介绍了三大原则，现在我们尝试了解一些 defer 的实现机制。

#### defer 数据结构

源码包 `src/src/runtime/runtime2.go:_defer` 定义了 defer 的数据结构：

```
// A _defer holds an entry on the list of deferred calls.
// If you add a field here, add code to clear it in freedefer and deferProcStack
// This struct must match the code in cmd/compile/internal/gc/reflect.go:deferstruct
// and cmd/compile/internal/gc/ssa.go:(*state).call.
// Some defers will be allocated on the stack and some on the heap.
// All defers are logically part of the stack, so write barriers to
// initialize them are not required. All defers must be manually scanned,
// and for heap defers, marked.
type _defer struct {
   siz     int32 // includes both arguments and results
   started bool
   heap    bool
   // openDefer indicates that this _defer is for a frame with open-coded
   // defers. We have only one defer record for the entire frame (which may
   // currently have 0, 1, or more defers active).
   openDefer bool
   sp        uintptr  // sp at time of defer
   pc        uintptr  // pc at time of defer
   fn        *funcval // can be nil for open-coded defers
   _panic    *_panic  // panic that is running defer
   link      *_defer

   // If openDefer is true, the fields below record values about the stack
   // frame and associated function that has the open-coded defer(s). sp
   // above will be the sp for the frame, and pc will be address of the
   // deferreturn call in the function.
   fd   unsafe.Pointer // funcdata for the function associated with the frame
   varp uintptr        // value of varp for the stack frame
   // framepc is the current pc associated with the stack frame. Together,
   // with sp above (which is the sp associated with the stack frame),
   // framepc/sp can be used as pc/sp pair to continue a stack trace via
   // gentraceback().
   framepc uintptr
}


```

*   siz：defer 函数的参数和结果的内存大小
    
*   fn：需要被延迟执行的函数
    
*   _panic：defer 的 panic 结构体
    
*   link：同一个协程里面的 defer 延迟函数，会通过该指针连接在一起
    
*   heap：是否分配在堆上面
    
*   openDefer：是否经过开放编码优化
    
*   sp：栈指针（一般会对应到汇编）
    
*   pc：程序计数器
    

通过上面的分析，我们可以发现，defer 后面一定要接一个函数的，所以 defer 的数据结构跟一般函数类似，也有栈地址、程序计数器、函数 地址等等。

与函数不同的一点是它含有一个指针，可用于指向另一个 defer，每个 goroutine 数据结构中实际上也有一个 defer 指针，该指针指向一个 defer 的单链表，每次声明一个 defer 时就将 defer 插入到单链表表头，每次执行 defer 时就从单链表表头取出一个 defer 执行。

即成员里面的 link ，它是一个 `*_defer`  类型，这时我们可以意识到了：在同一个协程里面的 defer 延迟函数，会通过该指针连接在一起。这个 link 指针，是指向的一个 defer 单链表的头，每次咱们声明一个 defer 的时候，就会将该 defer 的数据插入到这个单链表头部的位置。那么，等到 defer 执行的时候，则是从单链表的头开始，从头往后获取 defer（后进先出），直到单链表为空为止。

协程和 defer 的示意图  

![](https://mmbiz.qpic.cn/mmbiz_jpg/ib5vRCrBRAkWbHvZZdOiclSA0vRlhTFMIX6FibGw5W456gKyG5WrJZJ1Nm3WedKtibqZG0yWcI4a4ezd5Mb2Dzc6iaQ/640?wx_fmt=jpeg)

#### defer 的创建和执行

源码包 `src/runtime/panic.go` 定义了两个方法分别用于创建 defer 和执行 defer。

deferproc()：在声明 defer 处调用，其将 defer 函数存入 goroutine 的链表中;  

deferreturn() ：在 return 指令，准确的讲是在 ret 指令前调用，其将 defer 从 goroutine 链表中取出并执行。

我们可以简单理解，在编译在阶段，声明 defer 处插入了函数 deferproc()，在函数 return 前插入了函数 deferreturn()。

### 小结

总的来说，Go 语言的 defer 语句是一个强大的工具，可以帮助我们更好地管理资源并处理复杂的控制流。理解并熟练使用 defer 语句，可以大大提高我们代码的健壮性和可读性。

*   defer 定义的延迟函数参数在 defer 语句出时就已经确定下来了
    
*   defer 定义顺序与实际执行顺序相反
    
*   return 不是原子操作，执行过程是: 保存返回值 (若有)—> 执行 defer(若有)—>执行 ret 跳转
    
*   申请资源后立即使用 defer 关闭资源是好习惯
    

希望你们喜欢这篇文章，我会继续分享更多关于 Go 语言的知识和技巧，敬请期待！

如果你有任何问题或想要讨论更多关于 Go 语言的话题，欢迎留言和我交流。