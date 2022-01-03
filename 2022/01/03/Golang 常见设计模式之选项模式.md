> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xdF2glDpvbgCx-Wk-V9glQ)

熟悉 Python 开发的同学都知道，Python 有默认参数的存在，使得我们在实例化一个对象的时候，可以根据需要来选择性的覆盖某些默认参数，以此来决定如何实例化对象。当一个对象有多个默认参数时，这个特性非常好用，能够优雅地简化代码。

而 Go 语言从语法上是不支持默认参数的，所以为了实现既能通过默认参数创建对象，又能通过传递自定义参数创建对象，我们就需要通过一些编程技巧来实现。对于这些程序开发中的常见问题，软件行业的先行者们总结了许多解决常见场景编码问题的最佳实践，这些最佳实践后来成为了我们所说的设计模式。其中选项模式在 Go 语言开发中会经常用到。

通常我们有以下三种方法来实现通过默认参数创建对象，以及通过传递自定义参数创建对象：

**通过多构造函数实现**

第一种方式是通过多构造函数实现，下面是一个简单例子：

```
package main
import "fmt"
const (
    defaultAddr = "127.0.0.1"
    defaultPort = 8000
)
type Server struct {
    Addr string
    Port int
}
func NewServer() *Server {
    return &Server{
        Addr: defaultAddr,
        Port: defaultPort,
    }
}
func NewServerWithOptions(addr string, port int) *Server {
    return &Server{
        Addr: addr,
        Port: port,
    }
}
func main() {
    s1 := NewServer()
    s2 := NewServerWithOptions("localhost", 8001)
    fmt.Println(s1)  // &{127.0.0.1 8000}
    fmt.Println(s2)  // &{localhost 8001}
}
```

这里我们为 Server 结构体实现了两个构造函数：

*   NewServer：无需传递参数即可直接返回 Server 对象
    
*   NewServerWithOptions ：需要传递 addr 和 port 两个参数来构造 Server 对象
    

如果通过默认参数创建的对象即可满足需求，不需要对 Server 进行定制时，我们可以使用 NewServer 来生成对象（s1）。而如果需要对 Server 进行定制时，我们则可以使用 NewServerWithOptions 来生成对象（s2）。

  

**通过默认参数选项实现**

  

另外一种实现默认参数的方案，是为要生成的结构体对象定义一个选项结构体，用来生成要创建对象的默认参数，代码实现如下：

```
package main
import "fmt"
const (
    defaultAddr = "127.0.0.1"
    defaultPort = 8000
)
type Server struct {
    Addr string
    Port int
}
type ServerOptions struct {
    Addr string
    Port int
}
func NewServerOptions() *ServerOptions {
    return &ServerOptions{
        Addr: defaultAddr,
        Port: defaultPort,
    }
}
func NewServerWithOptions(opts *ServerOptions) *Server {
    return &Server{
        Addr: opts.Addr,
        Port: opts.Port,
    }
}
func main() {
    s1 := NewServerWithOptions(NewServerOptions())
    s2 := NewServerWithOptions(&ServerOptions{
        Addr: "localhost",
        Port: 8001,
    })
    fmt.Println(s1)  // &{127.0.0.1 8000}
    fmt.Println(s2)  // &{localhost 8001}
}
```

我们为 Server 结构体专门实现了一个 ServerOptions 用来生成默认参数，调用 NewServerOptions 函数即可获得默认参数配置，构造函数 NewServerWithOptions 接收一个 *ServerOptions 类型作为参数。所以我们可以通过以下两种方式来完成功能：

*   直接将调用 NewServerOptions 函数的返回值传递给 NewServerWithOptions 来实现通过默认参数生成对象（s1）
    
*   通过手动构造 ServerOptions 配置来生成定制对象（s2）
    

  

**通过选项模式实现**

以上两种方式虽然都能够完成功能，但却有以下缺点：

*   通过多构造函数实现的方案需要我们在实例化对象时分别调用不同的构造函数，代码封装性不强，会给调用者增加使用负担。
    
*   通过默认参数选项实现的方案需要我们预先构造一个选项结构，当使用默认参数生成对象时代码看起来比较冗余。
    

而选项模式可以让我们更为优雅地解决这个问题。代码实现如下：

```
package main
import "fmt"
const (
    defaultAddr = "127.0.0.1"
    defaultPort = 8000
)
type Server struct {
    Addr string
    Port int
}
type ServerOptions struct {
    Addr string
    Port int
}
type ServerOption interface {
    apply(*ServerOptions)
}
type FuncServerOption struct {
    f func(*ServerOptions)
}
func (fo FuncServerOption) apply(option *ServerOptions) {
    fo.f(option)
}
func WithAddr(addr string) ServerOption {
    return FuncServerOption{
        f: func(options *ServerOptions) {
            options.Addr = addr
        },
    }
}
func WithPort(port int) ServerOption {
    return FuncServerOption{
        f: func(options *ServerOptions) {
            options.Port = port
        },
    }
}
func NewServer(opts ...ServerOption) *Server {
    options := ServerOptions{
        Addr: defaultAddr,
        Port: defaultPort,
    }
    for _, opt := range opts {
        opt.apply(&options)
    }
    return &Server{
        Addr: options.Addr,
        Port: options.Port,
    }
}
func main() {
    s1 := NewServer()
    s2 := NewServer(WithAddr("localhost"), WithPort(8001))
    s3 := NewServer(WithPort(8001))
    fmt.Println(s1)  // &{127.0.0.1 8000}
    fmt.Println(s2)  // &{localhost 8001}
    fmt.Println(s3)  // &{127.0.0.1 8001}
}
```

乍一看我们的代码复杂了很多，但其实调用构造函数生成对象的代码复杂度是没有改变的，只是定义上的复杂。

我们定义了 ServerOptions 结构体用来配置默认参数。因为 Addr 和 Port 都有默认参数，所以 ServerOptions 的定义和 Server 定义是一样的。但有一定复杂性的结构体中可能会有些参数没有默认参数，必须让用户来配置，这时 ServerOptions 的字段就会少一些，大家可以按需定义。

同时，我们还定义了一个 ServerOption 接口和实现了此接口的 FuncServerOption 结构体，它们的作用是让我们能够通过 apply 方法为 ServerOptions 结构体单独配置某项参数。

我们可以分别为每个默认参数都定义一个 WithXXX 函数用来配置参数，如这里定义的 WithAddr 和 WithPort ，这样用户就可以通过调用 WithXXX 函数来定制需要覆盖的默认参数。

此时默认参数定义在构造函数 NewServer 中，构造函数的接收一个不定长参数，类型为 ServerOption，在构造函数内部通过一个 for 循环调用每个传进来的 ServerOption 对象的 apply 方法，将用户配置的参数依次赋值给构造函数内部的默认参数对象 options 中，以此来替换默认参数，for 循环执行完成后，得到的 options 对象将是最终配置，将其属性依次赋值给 Server 即可生成新的对象。

  

**总结**

  

通过 s2 和 s3 的打印结果可以发现，使用选项模式实现的构造函数更加灵活，相较于前两种实现，选项模式中我们可以自由的更改其中任意一项或多项默认配置。

虽然选项模式确实会多写一些代码，但多数情况下这都是值得的。比如 Google 的 gRPC 框架 Go 语言实现中创建 gRPC server 的构造函数 NewServer 就使用了选项模式，感兴趣的同学可以看下其源码的实现思想其实和这里的示例程序如出一辙。

以上就是我关于 Golang 选项模式的一点经验，希望今天的分享能够给你带来一些帮助。

* * *




-----------