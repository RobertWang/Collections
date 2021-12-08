> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/xiangxiaolin/p/12258022.html)

多线程程序在单核上运行，就是并发

多线程程序在多核上运行，就是并行

**Go 协程和 Go 主线程**

　　Go 主线程（线程）：一个 Go 线程上，可以起多个协程 ，你可以这样理解，协程是轻量级的线程

　　Go 协程的特点：

　　　　1）有独立的栈空间

　　　　2）共享程序堆空间

　　　　3） 调度由用户控制

　　　　4）协程是轻量级的线程 3

**goroutine 快速入门**

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
func test() {
    for i := 1; i <= 10; i++ {
        fmt.Println("test() hello, world " + strconv.Itoa(i))
        time.Sleep(time.Second)
    }
}

func main() {
    go test()

    for i := 1; i <= 10; i++ {
        fmt.Println("main() hello, world " + strconv.Itoa(i))
        time.Sleep(time.Second)
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

　　小结：

　　　　1）主线程是一个物理线程，直接作用在 cpu 上的，是重量级的，非常耗费 cpu 资源；

　　　　2）协程从主线程开启的，是轻量级的线程，是逻辑态。对资源消耗相对小；

　　　　3）Golang 的协程机制是重要的特点，可以轻松的开启上万个协程。其它编程语言的并发机制是一般基于线程的，开启过多的线程，资源耗费大，这里就突显 Golang 在并发上的优势了。

**MPG 模式基本介绍**

　　1）M：操作系统的主线程（是物理线程）

　　2）P：协程执行需要的上下文

　　3）G：协程

设置运行的 cpu 数目：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
func main() {
    cupNum := runtime.NumCPU()
    fmt.Println(cupNum)

    // 设置运行的cpu数目
    runtime.GOMAXPROCS(cupNum)
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 **channel**

　　1）channel 本质就是一个数据结构 - 队列

　　2）数据是先进先出

　　3）线程安全，多 goroutine 访问时，不需要加锁，就是说 channel 本身就是线程安全的

　　4）channel 有类型的，一个 string 的 channel 只能存放 string 数据

　　5）示意图：

　　![](https://img2018.cnblogs.com/i-beta/1675901/202002/1675901-20200211215930762-1131336470.png)

 **定义声明 channel**

　　　　var 变量名 chan 数据类型

　　　　举例：

　　　　　　var intChan chan int

　　　　　　var mapChan chan map[int]string

　　　　　　var perChan chan Person

　　　　　　var perChan2 chan *Person

　　　　1）channel 是引用类型

　　　　2）channel 必须初始化才能写入数据，即 make 后才能使用

　　　　3）管道是有类型的，intChan 只能写入整数 int

　　**管道的初始化，写入与读取数据及基本的注意事项**　　

```
func main() {
    var intChan chan int
    intChan = make(chan int, 4)

    fmt.Printf("intChan 的值=%v inChan本身的地址=%p\n", intChan, &intChan)

    // 存数据
    intChan <- 10
    num := 23
    intChan <- num
    intChan <- 30
    intChan<- 22
    fmt.Printf("channel len=%v cap=%v \n", len(intChan), cap(intChan))

    // 读取数据
    var num2 int
    num2 = <-intChan
    fmt.Println("num2=",num2)
    fmt.Printf("channel len=%v cap=%v \n", len(intChan), cap(intChan))
}

```

 **channel 的关闭**

　　使用内置函数 close 可以关闭 channel，当 channel 关闭后，就不能再向 channel 写数据了，但是仍然可以从该 channel 读取数据。

**channel 的遍历**

　　channel 支持 for-range 的方式进行遍历，请注意两个细节

　　　　1）在遍历时，如果 channel 没有关闭，则会出现 deadlock 的错误

　　　　2）在遍历时，如果 channel 已经关闭，则会正常遍历数据，遍历完后，就会退出遍历。

**goroutine 和 channel 结合使用案例**

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
func writeData(intChan chan int) {
    for i := 0; i < 50; i++ {
        intChan<- i
        fmt.Println("writeData ", i)
    }
    close(intChan)
}

func readData(intChan chan int, exitChan chan bool) {
    for {
        v, ok := <-intChan
        if !ok {
            break
        }
        fmt.Println("readData ", v)
    }
    exitChan<- true
    close(exitChan)
}

func main() {
    intChan := make(chan int, 50)
    exitChan := make(chan bool, 1)

    go writeData(intChan)
    go readData(intChan, exitChan)
    for {
        _, ok := <-exitChan
        if !ok {
            break
        }
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 **channel 阻塞**

　　如果只是向管道写入数据，而没有读取，就会出现阻塞而 dead lock，原因是容量不足

　　如果，编译器运行，发现一个管道只有写，而没有读，则该管道会阻塞

 **channel 注意事项**

　　1）channel 可以声明为只读（var chan2 chan<- int），或者只写（var chan2 <-chan int）性质

　　2）使用 select 可以解决从管道取数据的阻塞问题

　　3）goroutine 中使用 recover，解决协程中出现 panic，导致程序崩溃问题

　　　　如果我们起了一个协程，但是这个协程出现了 panic，如果我们没有捕获这个 panic，就会造成整个程序崩溃，这时我们可以在 goroutine 中使用 recover 来捕获 panic 进行处理，这样即使这个协程发生的问题，但是主线程仍然不受影响，可以继续执行。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
func sayHello() {
    for i := 0; i < 10; i++ {
        time.Sleep(time.Second)
        fmt.Println("hello world!")
    }
}

func test() {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("test() 发生了错误！")
        }
    }()
    var myMap map[int]string
    myMap[1] = "golang"
}

func main() {
    go sayHello()
    go test()

    for i := 0 ;i < 10; i++ {
        fmt.Println("main() ok=", i)
        time.Sleep(time.Second)
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")