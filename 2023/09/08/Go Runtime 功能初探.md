> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HP1075oY3xQ3CbTwZ0veeQ)

> 1. GOROOT() 获取 GOROOT 环境变量 \ x0d\x0a2. Version() 获取 GO 的版本号 \ x0d\x0a3. NumCPU() 获取本机 CPU 个数 \ x0d\x0a4. GOMAXPROC......

题图来自 Understand Compile Time && Runtime! Improving Golang Performance(1)[1]

以下内容，是对 运行时 runtime 的神奇用法 [2] 的学习与记录

目录:

*   1. 获取 GOROOT 环境变量
    
*   2. 获取 GO 的版本号
    
*   3. 获取本机 CPU 个数
    
*   4. 设置最大可同时执行的最大 CPU 数
    
*   5. 设置 cup profile 记录的速录
    
*   6. 查看 cup profile 下一次堆栈跟踪数据
    
*   7. 立即执行一次垃圾回收
    
*   8. 给变量绑定方法, 当垃圾回收的时候进行监听
    
*   9. 查看内存申请和分配统计信息
    
*   10. 查看程序正在使用的字节数
    
*   11. 查看程序正在使用的对象数
    
*   12. 获取调用堆栈列表
    
*   13. 获取内存 profile 记录历史
    
*   14. 执行一个断点
    
*   15. 获取程序调用 go 协程的栈踪迹历史
    
*   16. 获取当前函数或者上层函数的标识号、文件名、调用方法在当前文件中的行号
    
*   17. 获取与当前堆栈记录相关链的调用栈踪迹
    
*   18. 获取一个标识调用栈标识符 pc 对应的调用栈
    
*   19. 获取调用栈所调用的函数的名字
    
*   20. 获取调用栈所调用的函数的所在的源文件名和行号
    
*   21. 获取该调用栈的调用栈标识符
    
*   22. 获取当前进程执行的 cgo 调用次数
    
*   23. 获取当前存在的 go 协程数
    
*   24. 终止掉当前的 go 协程
    
*   25. 让其他 go 协程优先执行, 等其他协程执行完后, 在执行当前的协程
    
*   26. 获取活跃的 go 协程的堆栈 profile 以及记录个数
    
*   27. 将调用的 go 协程绑定到当前所在的操作系统线程，其它 go 协程不能进入该线程
    
*   28. 解除 go 协程与操作系统线程的绑定关系
    
*   29. 获取线程创建 profile 中的记录个数
    
*   30. 控制阻塞 profile 记录 go 协程阻塞事件的采样率
    
*   31. 返回当前阻塞 profile 中的记录个数
    

### 1. GOROOT() 获取 GOROOT 环境变量

GOROOT() 返回 Go 的根目录。如果存在 GOROOT 环境变量，返回该变量的值；否则，返回创建 Go 时的根目录

```
package main

import (
 "fmt"
 "runtime"
)

func main() {
 fmt.Println(runtime.GOROOT()) // /Users/fliter/.g/go
}


```

### 2. Version() 获取 GO 的版本号

Version() 返回 Go 的版本字符串。要么是提交的 hash 和创建时的日期；要么是发行标签如 "go1.20"

```
package main

import (
 "fmt"
 "runtime"
)

func main() {
 fmt.Println(runtime.Version()) // go1.19
}


```

### 3. NumCPU() 获取本机 CPU 个数

NumCPU 返回本地机器的逻辑 CPU 个数

```
package main

import (
 "fmt"
 "runtime"
)

func main() {
 fmt.Println(runtime.NumCPU()) // 8

}



```

### 4. GOMAXPROCS() 设置最大可同时执行的最大 CPU 数

GOMAXPROCS() 设置可同时执行的最大 CPU 数，并返回先前的设置。 若 n < 1，则不会更改当前设置。

本地机器的逻辑 CPU 数可通过 NumCPU 查询。该函数在调度程序优化后会去掉?（啥时候..）

```
package main

import (
 "fmt"
 "runtime"
 "time"
)

func main() {

 runtime.GOMAXPROCS(1)
 startTime := time.Now()
 var s1 chan int64 = make(chan int64)
 var s2 chan int64 = make(chan int64)
 var s3 chan int64 = make(chan int64)
 var s4 chan int64 = make(chan int64)
 go calc(s1)
 go calc(s2)
 go calc(s3)
 go calc(s4)
 <-s1
 <-s2
 <-s3
 <-s4
 endTime := time.Now()
 fmt.Println(endTime.Sub(startTime)) // 第一行注释掉 耗时: 386.954625ms； 取消注释 耗时: 1.34715s

}
func calc(s chan int64) {
 var count int64 = 0
 for i := 0; i < 1000000000; i++ {
  count += int64(i)
 }
 s <- count
}


```

### 5. SetCPUProfileRate() 设置 cup profile 记录的速录

SetCPUProfileRate() 设置 CPU profile 记录的速率为平均每秒 hz 次。如果 hz<=0，SetCPUProfileRate 会关闭 profile 的记录。如果记录器在执行，该速率必须在关闭之后才能修改

绝大多数使用者应使用 runtime/pprof 包或 testing 包的`-test.cpuprofile`选项而非直接使用 SetCPUProfileRate

### 6. CPUProfile 查看 cup profile 下一次堆栈跟踪数据

`func CPUProfile() []byte`

已废弃

### 7. GC() 立即执行一次垃圾回收

Go 三种触发 GC 的方式之一 (另外两种为 2 分钟固定一次 && 达到阈值时触发)

```
package main

import (
 "runtime"
 "time"
)

type Student struct {
 name string
}

func main() {
 var i *Student = new(Student)
 runtime.SetFinalizer(i, func(i interface{}) {
  println("垃圾回收了") // 垃圾回收了
 })
 runtime.GC() // 如果将这行注释，则上面不会输出
 time.Sleep(time.Second)
}


```

### 8. SetFinalizer() 给变量绑定方法, 当垃圾回收的时候进行监听

`func SetFinalizer(x, f interface{})`

注意 x 必须是指针类型, f 函数的参数一定要和 x 保持一致, 或者写 interface{}, 不然程序会报错

代码同上例

### 9. ReadMemStats() 查看内存申请和分配统计信息

可以获得如下信息:

```
type MemStats struct {
    // 一般统计
    Alloc      uint64 // 已申请且仍在使用的字节数
    TotalAlloc uint64 // 已申请的总字节数（已释放的部分也算在内）
    Sys        uint64 // 从系统中获取的字节数（下面XxxSys之和）
    Lookups    uint64 // 指针查找的次数
    Mallocs    uint64 // 申请内存的次数
    Frees      uint64 // 释放内存的次数
    // 主分配堆统计
    HeapAlloc    uint64 // 已申请且仍在使用的字节数
    HeapSys      uint64 // 从系统中获取的字节数
    HeapIdle     uint64 // 闲置span中的字节数
    HeapInuse    uint64 // 非闲置span中的字节数
    HeapReleased uint64 // 释放到系统的字节数
    HeapObjects  uint64 // 已分配对象的总个数
    // L低层次、大小固定的结构体分配器统计，Inuse为正在使用的字节数，Sys为从系统获取的字节数
    StackInuse  uint64 // 引导程序的堆栈
    StackSys    uint64
    MSpanInuse  uint64 // mspan结构体
    MSpanSys    uint64
    MCacheInuse uint64 // mcache结构体
    MCacheSys   uint64
    BuckHashSys uint64 // profile桶散列表
    GCSys       uint64 // GC元数据
    OtherSys    uint64 // 其他系统申请
    // 垃圾收集器统计
    NextGC       uint64 // 会在HeapAlloc字段到达该值（字节数）时运行下次GC
    LastGC       uint64 // 上次运行的绝对时间（纳秒）
    PauseTotalNs uint64
    PauseNs      [256]uint64 // 近期GC暂停时间的循环缓冲，最近一次在[(NumGC+255)%256]
    NumGC        uint32
    EnableGC     bool
    DebugGC      bool
    // 每次申请的字节数的统计，61是C代码中的尺寸分级数
    BySize [61]struct {
        Size    uint32
        Mallocs uint64
        Frees   uint64
    }
}


```

```
package main

import (
 "fmt"
 "runtime"
 "time"
)

type Student2 struct {
 name string
}

func main() {
 var list = make([]*Student2, 0)
 for i := 0; i < 100000; i++ {
  var s *Student2 = new(Student2)
  list = append(list, s)
 }
 memStatus := runtime.MemStats{}
 runtime.ReadMemStats(&memStatus)
 fmt.Printf("申请的内存:%d\n", memStatus.Mallocs) // 申请的内存:100250

 fmt.Printf("释放的内存次数:%d\n", memStatus.Frees) // 释放的内存次数:45

 time.Sleep(time.Second)
}


```

### 10. InUseBytes() 查看程序正在使用的字节数

`func (r *MemProfileRecord) InUseBytes() int64`

InUseBytes 返回正在使用的字节数（AllocBytes – FreeBytes）

### 11. InUseObjects() 查看程序正在使用的对象数

`func (r *MemProfileRecord) InUseObjects() int64`

InUseObjects 返回正在使用的对象数（AllocObjects - FreeObjects）

### 12. Stack() 获取调用堆栈列表

`func (r *MemProfileRecord) Stack() []uintptr`

Stack 返回关联至此记录的调用栈踪迹，即 r.Stack0 的前缀

### 13. MemProfile() 获取内存 profile 记录历史

`func MemProfile(p []MemProfileRecord, inuseZero bool) (n int, ok bool)`

MemProfile 返回当前内存 profile 中的记录数 n

*   `若len(p)>=n，MemProfile会将此分析报告复制到p中并返回(n, true)；`
    
*   `若len(p)<n，MemProfile则不会更改p，而只返回(n, false)`
    

如果 inuseZero 为 true，该 profile 就会包含无效分配记录（其中 r.AllocBytes>0，而 r.AllocBytes==r.FreeBytes。这些内存都是被申请后又释放回运行时环境的）

大多数调用者应当使用 runtime/pprof 包或 testing 包的`-test.memprofile`标记，而非直接调用 MemProfile

### 14. Breakpoint() 执行一个断点

`runtime.Breakpoint()`

### 15. Stack() 获取程序调用 go 协程的栈踪迹历史

`func Stack(buf []byte, all bool) int`

Stack 将调用其的 go 程的调用栈踪迹格式化后写入到 buf 中并返回写入的字节数

若 all 为 true，函数会在写入当前 go 程的踪迹信息后，将其它所有 go 程的调用栈踪迹都格式化写入到 buf 中

```
package main

import (
 "fmt"
 "runtime"
 "time"
)

func main() {
 go showRecord()
 time.Sleep(time.Second)
 buf := make([]byte, 10000000000)
 runtime.Stack(buf, true)
 fmt.Println(string(buf))
}

func showRecord() {
 ticker := time.Tick(time.Second)
 for t := range ticker {
  fmt.Println(t)
 }
}


```

输出:

```
2023-04-19 17:25:26.386522 +0800 CST m=+1.001105543
2023-04-19 17:25:27.386892 +0800 CST m=+2.001489376
2023-04-19 17:25:28.386505 +0800 CST m=+3.001116043
2023-04-19 17:25:29.38553 +0800 CST m=+4.000154334
2023-04-19 17:25:30.385618 +0800 CST m=+5.000255584
2023-04-19 17:25:31.385817 +0800 CST m=+6.000468334
2023-04-19 17:25:32.385528 +0800 CST m=+7.000192918
2023-04-19 17:25:33.385646 +0800 CST m=+8.000323543
goroutine 1 [running]:
main.main()
        /Users/fliter/runtime-demo/15Stack.go:13 +0x68

goroutine 4 [chan receive]:
main.showRecord()
        /Users/fliter/runtime-demo/15Stack.go:19 +0xac
created by main.main
        /Users/fliter/runtime-demo/15Stack.go:10 +0x24


```

### 16. Caller() 获取当前函数或者上层函数的标识号、文件名、调用方法在当前文件中的行号

`func Caller(skip int) (pc uintptr, file string, line int, ok bool)`

```
package main

import (
 "fmt"
 "runtime"
)

func main() {
 pc, file, line, ok := runtime.Caller(0)
 fmt.Println(pc)   // 4336410771
 fmt.Println(file) // /Users/fliter/runtime-demo/16Caller.go
 fmt.Println(line) //9
 fmt.Println(ok)   //true
}


```

pc = 4336410771 不是 main 函数的标识, 而是 runtime.Caller 方法的标识, line = 13 标识它在 main 方法中的第 13 行被调用

```
//package main
//
//import (
// "fmt"
// "runtime"
//)
//
//func main() {
// pc, file, line, ok := runtime.Caller(0)
// fmt.Println(pc)   // 4336410771
// fmt.Println(file) // /Users/fliter/runtime-demo/16Caller.go
// fmt.Println(line) //9
// fmt.Println(ok)   //true
//}

package main

import (
 "fmt"
 "runtime"
)

func main() {
 pc, _, line, _ := runtime.Caller(1)
 fmt.Printf("main函数的pc:%d\n", pc)      // main函数的pc:4364609931
 fmt.Printf("main函数被调用的行数:%d\n", line) // main函数被调用的行数:250
 show()
}
func show() {
 pc, _, line, _ := runtime.Caller(1)
 fmt.Printf("show函数的pc:%d\n", pc)      // show函数的pc:4364974271
 fmt.Printf("show函数被调用的行数:%d\n", line) // show函数被调用的行数:27
 // 这个是main函数的栈
 pc, _, line, _ = runtime.Caller(2)
 fmt.Printf("show的上层函数的pc:%d\n", pc)      // show的上层函数的pc:4364609931
 fmt.Printf("show的上层函数被调用的行数:%d\n", line) // show的上层函数被调用的行数:250
 pc, _, _, _ = runtime.Caller(3)
 fmt.Println(pc) //4364778899
 pc, _, _, _ = runtime.Caller(4)
 fmt.Println(pc) // 0
}



```

golang 获取调用者的方法名及所在行数 [3]

runtime.Caller 的性能问题 [4]

### 17. Callers() 获取与当前堆栈记录相关链的调用栈踪迹

`func Callers(skip int, pc []uintptr) int`

会把当前 go 程调用栈上的调用栈标识符填入切片 pc 中，返回写入到 pc 中的项数。实参 skip 为开始在 pc 中记录之前所要跳过的栈帧数，0 表示 Callers 自身的调用栈，1 表示 Callers 所在的调用栈。返回写入 p 的项数

```
package main

import (
 "fmt"
 "runtime"
)

func main() {
 pcs := make([]uintptr, 10)
 i := runtime.Callers(1, pcs)
 fmt.Println(pcs[:i]) // [4311883569 4311525404 4311694372]
}


```

获得了三个 pc 其中有一个是 main 方法自身的

### 18. FuncForPC() 获取一个标识调用栈标识符 pc 对应的调用栈

`func FuncForPC(pc uintptr) *Func`

```
package main

import (
 "runtime"
)

func main() {

 pcs := make([]uintptr, 10)
 i := runtime.Callers(1, pcs)
 for _, pc := range pcs[:i] {
  println(runtime.FuncForPC(pc))
 }
}


```

输出:

```
0x102f660f0
0x102f5c9f0
0x102f64840


```

用途见下

### 19. Name() 获取调用栈所调用的函数的名字

`func (f *Func) string`

```
package main

import (
 "runtime"
)

func main() {

 pcs := make([]uintptr, 10)
 i := runtime.Callers(1, pcs)
 for _, pc := range pcs[:i] {
  funcPC := runtime.FuncForPC(pc)
  println(funcPC.Name())
 }
}


```

输出:

```
main.main
runtime.main
runtime.goexit


```

### 20. FileLine() 获取调用栈所调用的函数的所在的源文件名和行号

`func (f *Func) FileLine(pc uintptr) (file string, line int)`

```
package main

import (
 "runtime"
)

func main() {
 pcs := make([]uintptr, 10)
 i := runtime.Callers(1, pcs)
 for _, pc := range pcs[:i] {
  funcPC := runtime.FuncForPC(pc)
  file, line := funcPC.FileLine(pc)
  println(funcPC.Name(), file, line)
 }
}


```

输出:

```
main.main /Users/fliter/runtime-demo/20FileLine.go 9
runtime.main /Users/fliter/.g/go/src/runtime/proc.go 259
runtime.goexit /Users/fliter/.g/go/src/runtime/asm_arm64.s 1166


```

### 21. Entry() 获取该调用栈的调用栈标识符

`func (f *Func) Entry() uintptr`

```
package main

import (
 "runtime"
)

func main() {
 pcs := make([]uintptr, 10)
 i := runtime.Callers(1, pcs)
 for _, pc := range pcs[:i] {
  funcPC := runtime.FuncForPC(pc)
  println(funcPC.Entry())
 }
}


```

输出:

```
4310699120
4310540672
4310690704


```

### 22. NumCgoCall() 获取当前进程执行的 cgo 调用次数

获取当前进程调用 C 方法的次数

`func NumCgoCall() int64`

```
package main

import (
 "runtime"
)

/*
#include <stdio.h>
*/
import "C"

func main() {
 println(runtime.NumCgoCall()) // 1
}


```

没有调用 C 的方法为什么是 1 呢？因为 import C 会调用 C 包中的 init 方法

```
package main

import (
 "runtime"
)

/*
   #include <stdio.h>
   // 自定义一个c语言的方法
   static void myPrint(const char* msg) {
     printf("myPrint: %s", msg);
   }
*/
import "C"

func main() {
 // 调用c方法
 C.myPrint(C.CString("Hello,C\n")) // myPrint: Hello,C
 println(runtime.NumCgoCall()) // 3
}


```

### 23. NumGoroutine() 获取当前存在的 go 协程数

`func NumGoroutine() int`

```
package main

import "runtime"

func main() {
 go print()
 print()
 println(runtime.NumGoroutine()) // 2
}
func print() {

}


```

当前程序有 2 个 go 协程 一个是 main.go 主协程， 另外一个是 go print()

### 24. Goexit() 终止掉当前的 go 协程

`func Goexit()`

```
package main

import (
 "fmt"
 "runtime"
)

func main() {
 print()
 fmt.Println("继续执行")
}
func print() {
 fmt.Println("准备结束go协程")
 runtime.Goexit()
 defer fmt.Println("结束了")
}


```

输出:

```
准备结束go协程
fatal error: no goroutines (main called runtime.Goexit) - deadlock!
exit status 2


```

Goexit 终止调用它的 go 协程, 其他协程不受影响, Goexit 会在终止该 go 协程前执行所有的 defer 函数，前提是 defer 必须在它前面定义，如下

```
package main

import (
 "fmt"
 "runtime"
)

func main() {
 print()
 fmt.Println("继续执行")
}
func print() {
 fmt.Println("准备结束go协程")
 defer fmt.Println("结束了--会输出出来")
 runtime.Goexit()
 //defer fmt.Println("结束了")
}


```

输出:

```
准备结束go协程
结束了--会输出出来
fatal error: no goroutines (main called runtime.Goexit) - deadlock!
exit status 2


```

如果在 main 主协程调用该方法, 会终止 主协程, 但不会让 main 返回, 因为 main 函数没有返回,

程序会继续执行其他 go 协程, 当其他 go 协程执行完毕后, 程序就会崩溃

```
package main

import (
 "fmt"
 "runtime"
 "time"
)

func main() {

 start := time.Now()
 go func() {
  time.Sleep(3e9)
  println("123")
 }()

 defer fmt.Println(time.Since(start))
 runtime.Goexit()

}


```

输出:

```
20.5µs
123
fatal error: no goroutines (main called runtime.Goexit) - deadlock!
exit status 2


```

### 25. Gosched() 让其他 go 协程优先执行, 等其他协程执行完后, 再执行当前的协程

`func Gosched()`

```
package main

import (
 "fmt"
)

func main() {
 go print25()
 fmt.Println("继续执行")
}
func print25() {
 fmt.Println("执行打印方法")
}


```

调用了 go print 方法, 但是还未执行, main 函数就执行完毕了（启一个协程也是需要时间的，这个时间比 for 循环，比程序继续执行要耗时多很多）

可以使用 channel，waitgroup 等，此处使用 `runtime.Gosched()`

```
package main

import (
 "fmt"
 "runtime"
)

func main() {
 go print25()
 runtime.Gosched()
 fmt.Println("继续执行")
}
func print25() {
 fmt.Println("执行打印方法")
}


```

输出:

```
执行打印方法
继续执行


```

[Rust vs Go: 常用语法对比 (13)- 将优先权让给其他线程](https://json.dashen.tech/2021/09/14/Rust-vs-Go-%E5%B8%B8%E7%94%A8%E8%AF%AD%E6%B3%95%E5%AF%B9%E6%AF%94-13/ "Rust vs Go: 常用语法对比 (13"Rust vs Go: 常用语法对比 (13)- 将优先权让给其他线程 ")- 将优先权让给其他线程")

Go 用两个协程交替打印 100 以内的奇偶数 [5]

### 26. GoroutineProfile() 获取活跃的 go 协程的堆栈 profile 以及记录个数

`func GoroutineProfile(p []StackRecord) (n int, ok bool)`

```
// GoroutineProfile returns n, the number of records in the active goroutine stack profile.
// If len(p) >= n, GoroutineProfile copies the profile into p and returns n, true.
// If len(p) < n, GoroutineProfile does not change p and returns n, false.
//
// Most clients should use the runtime/pprof package instead
// of calling GoroutineProfile directly.
func GoroutineProfile(p []StackRecord) (n int, ok bool) {

 return goroutineProfileWithLabels(p, nil)
}


```

```
package main

import (
 "fmt"
 "runtime"
)

func main() {

 for i := 0; i < 10; i++ {

  go func(k int) {
   fmt.Println(i)
  }(i)
 }

 fmt.Println("----------------")
 p := make([]runtime.StackRecord, 10000)

 fmt.Println(runtime.GoroutineProfile(p)) // 1 true
}


```

输出:

```
10
10
10
10
10
10
10
----------------
7
10
10
1 true


```

pprof 里面使用了此 func

Go 应用的性能优化 [6]

### 27. LockOSThread() 将调用的 go 协程绑定到当前所在的操作系统线程，其它 go 协程不能进入该线程

`func LockOSThread()`

将调用的 go 程绑定到它当前所在的操作系统线程。除非调用的 go 程退出或调用 UnlockOSThread，否则它将总是在该线程中执行，而其它 go 程不能进入该线程

```
package main

import (
 "fmt"
 "runtime"
 "time"
)

func main() {
 go calcSum1()
 go calcSum2()
 time.Sleep(time.Second * 10)
}

func calcSum1() {
 runtime.LockOSThread()
 start := time.Now()
 count := 0
 for i := 0; i < 10000000000; i++ {
  count += i
 }
 end := time.Now()
 fmt.Println("calcSum1耗时")
 fmt.Println(end.Sub(start))
 defer runtime.UnlockOSThread()
}

func calcSum2() {
 start := time.Now()
 count := 0
 for i := 0; i < 10000000000; i++ {
  count += i
 }
 end := time.Now()
 fmt.Println("calcSum2耗时")
 fmt.Println(end.Sub(start))
}


```

输出:

```
calcSum1耗时
3.295679583s
calcSum2耗时
3.296763125s


```

看起来没有太大的差别;

但估计在很多个协程 (涉及到频繁的调度和切换)，但是有一项重要功能需独占一个核，可使用该 func

Go LockOSThread[7]

### 28. UnlockOSThread() 解除 go 协程与操作系统线程的绑定关系

`func UnlockOSThread()`

将调用此 func 的协程，解除和其绑定的操作系统线程

若调用的协程未调用 LockOSThread，UnlockOSThread 不做操作

从 1,10 之后，调用了多少次 LockOSThread，就要使用 UnlockOSThread 接触绑定..

### 29. ThreadCreateProfile() 获取线程创建 profile 中的记录个数

`func ThreadCreateProfile(p []StackRecord) (n int, ok bool)`

返回线程创建 profile 中的记录个数。

*   `如果len(p)>=n，本func就会将profile中的记录复制到p中并返回(n, true)`
    
*   `若len(p)<n，则不会更改p，而只返回(n, false)`
    

绝大多数情况下应当使用 runtime/pprof 包，而非直接调用 ThreadCreateProfile

### 30. SetBlockProfileRate() 控制阻塞 profile 记录 go 协程阻塞事件的采样率

`func SetBlockProfileRate(rate int)`

SetBlockProfileRate 控制阻塞 profile 记录 go 程阻塞事件的采样频率。对于一个阻塞事件，平均每阻塞 rate 纳秒，阻塞 profile 记录器就采集一份样本。

*   `要在profile中包括每一个阻塞事件，需传入rate=1`
    
*   `要完全关闭阻塞profile的记录，需传入rate<=0`
    

### 31. BlockProfile() 返回当前阻塞 profile 中的记录个数

`func BlockProfile(p []BlockProfileRecord) (n int, ok bool)`

BlockProfile 返回当前阻塞 profile 中的记录个数

*   `如果len(p)>=n，本函数就会将此profile中的记录复制到p中并返回(n, true)`
    
*   `如果len(p)<n，本函数则不会修改p，而只返回(n, false)`
    

绝大多数情况应当使用 runtime/pprof 包或 testing 包的`-test.blockprofile`标记， 而非直接调用 BlockProfile

### 参考资料

[1]

Understand Compile Time && Runtime! Improving Golang Performance(1): https://levelup.gitconnected.com/improving-golang-performance-1-must-master-two-key-concepts-238a2055c926

[2]

运行时 runtime 的神奇用法: https://blog.csdn.net/u011525168/article/details/88401166

[3]

golang 获取调用者的方法名及所在行数: https://dashen.tech/2018/05/18/golang%E8%8E%B7%E5%8F%96%E8%B0%83%E7%94%A8%E8%80%85%E7%9A%84%E6%96%B9%E6%B3%95%E5%90%8D%E5%8F%8A%E6%89%80%E5%9C%A8%E8%A1%8C%E6%95%B0/

[4]

runtime.Caller 的性能问题: https://dashen.tech/2019/11/11/runtime-Caller%E7%9A%84%E6%80%A7%E8%83%BD%E9%97%AE%E9%A2%98/

[5]

Go 用两个协程交替打印 100 以内的奇偶数: https://dashen.tech/2022/04/03/Go%E7%94%A8%E4%B8%A4%E4%B8%AA%E5%8D%8F%E7%A8%8B%E4%BA%A4%E6%9B%BF%E6%89%93%E5%8D%B0100%E4%BB%A5%E5%86%85%E7%9A%84%E5%A5%87%E5%81%B6%E6%95%B0/

[6]

Go 应用的性能优化: https://zhuanlan.zhihu.com/p/406826295

[7]

Go LockOSThread: https://dashen.tech/2017/07/11/Go-LockOSThread/