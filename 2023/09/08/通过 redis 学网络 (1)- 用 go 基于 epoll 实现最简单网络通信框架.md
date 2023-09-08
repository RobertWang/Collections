> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vEwXRCPaa7X1pmMLVW_cNw)

> ❝本系列主要是为了对 redis 的网络模型进行学习，我会用 golang 实现一个 reactor 网络模型，并实现对

> ❝
> 
> 本系列主要是为了对 redis 的网络模型进行学习，我会用 golang 实现一个 reactor 网络模型，并实现对 redis 协议的解析。
> 
> ❞

系列源码已经上传 github

```
https://github.com/HobbyBear/tinyredis/tree/chapter1


```

redis 的网络模型是基于 epoll 实现的，所以这一节让我们先基于 epoll，实现一个最简单的服务端客户端通信模型。在实现前，先来简单的了解下 epoll 的原理。

> ❝
> 
> 为什么不用 golang 的原生的 netpoll 网络框架呢，这是因为 netpoll 框架虽然底层也是基于 epoll 实现，但是它提供给开发人员使用网络 io 方式依然是同步阻塞模式，一个连接单独的拿给一个协程去处理，为了更加真实的感受下 redis 的网络模型，我们不用 netpoll 框架，而是自己写一个非阻塞的网络模型。
> 
> ❞

epoll 网络通信原理
------------

通常情况下服务端的处理客户端请求的逻辑是客户端每发起一个连接，服务端就单独起一个线程去处理这个连接的请求，对于 go 应用程序而言，则是启用一个协程去处理这个连接。 而采用 epoll 相关的 api 后，能够让我们在一个线程或者协程里去处理多个连接的请求。

一个套接字连接对应一个文件描述符，当收到客户端的连接请求时，可以将对应的文件描述符加入到 epoll 实例关注的事件中去。

在 golang 里，可以通过 syscall.EpollCreate1 去创建一个 epoll 实例。

```
func EpollCreate1(flag int) (fd int, err error) 


```

其返回结果的 fd 就代表 epoll 实例的 fd，当收到客户端的连接请求时，便可以将客户端连接的 fd，通过 EpollCtl 加入到 epoll 实例感兴趣的事件当中。

```
func EpollCtl(epfd int, op int, fd int, event *EpollEvent) (err error) 


```

EpollCtl 方法参数的 epfd 则是 EpollCreate1 返回的 fd，EpollCtl 的第二个参数则是代表客户端连接的 fd，通过我们在获取到客户端连接后，后续的行为便是查看客户端是否有数据发送过来或者往客户端发送数据，这些在 epoll api 里用 event 事件去表示，分别对应了读 event 和写 event，这便是 EpollCtl 第三个参数所代表的含义。

将这些感兴趣事件添加到 epoll 实例中后，就代表 epoll 实例后续会监听这些连接的读写事件的到达，那么读写事件到达后，用户程序又是如何知道的呢，这就要提到 epoll 相关的另一个 api，EpollWait。

```
func EpollWait(epfd int, events []EpollEvent, msec int) (n int, err error) 


```

EpollWait 的第二个参数是一个事件数组，用户应用程序调用 EpollWait 时传入一个固定长度的事件数组，然后 EpollWait 会将这个数组尽可能填满，这样用户程序便能知道有哪些事件类型到达了，EpollEvent 类型如下所示:

```
type EpollEvent struct {
 Events uint32
 Fd     int32
 Pad    int32
}


```

其中 fd 则代表这些事件所关联的客户端连接的 fd，通过这个 fd，我们便可以对对应连接进行读写操作了。

而 Events 是个枚举类型, 比较常用的枚举以及含义如下:

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">类型</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px; text-align: left;">解释</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">EPOLLIN</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;" class="">表示文件描述符可读。</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">EPOLLRDHUP</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;" class="">表示 TCP 连接的远程端点关闭或半关闭连接</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">EPOLLET</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;" class="">表示使用边缘触发模式来监听事件</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">EPOLLOUT</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">表示文件描述符可写</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">EPOLLERR</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">表示文件描述符发生错误时发生，这个事件不通过 EpollCtl 添加也能触发</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">EPOLLHUP</td><td data-style="border-color: rgb(204, 204, 204); font-size: 14px; color: rgb(89, 89, 89); min-width: 85px;">与 EPOLLRDHUP 类似同样表示连接关闭，在不支持 EPOLLRDHUP 的 linux 版本会触发，这个事件不通过 EpollCtl 添加也能触发</td></tr></tbody></table>

虽然 epoll event 还有其他类型，不过一般情况下监控这几种类型就足够了，golang 的 netpoll 框架在添加连接的文件描述符时事件时也只添加了这几种类型。netpoll 的部分源码如下:

```
func netpollopen(fd uintptr, pd *pollDesc) int32 {
 var ev epollevent
 ev.events = _EPOLLIN | _EPOLLOUT | _EPOLLRDHUP | _EPOLLET
 *(**pollDesc)(unsafe.Pointer(&ev.data)) = pd
 return -epollctl(epfd, _EPOLL_CTL_ADD, int32(fd), &ev)
}


```

如何用 golang 创建基于 epoll 的网络框架
---------------------------

了解完 epoll 的一些概念以后，现在来看下我们需要实现的网络框架模型是怎样的。我们先实现一个最简单的网络通信框架，客户端发送来消息，然后服务端打印收到的消息。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/81178hAUFMiczcqcBP9sqdLIPtac8QRuG0FXZibgCb83V1I3W6reQOcXt53bZ8NSuasA1mYeWibtic8dgdWhJHDPlg/640?wx_fmt=png)Pasted image 20230605160424.png

如上图所示，我们收到新的连接后，会调用 epoll 实例的 EpollCtl 方法将连接的可读事件添加到 epoll 实例中，接着调用 EpollWait 方法等待客户端再次发送消息时，让连接变为可读。

下面是程序的效果测试结果

#### 效果测试

![](https://mmbiz.qpic.cn/sz_mmbiz_png/81178hAUFMiczcqcBP9sqdLIPtac8QRuGicWN426nnEVdGmKD1dQcoWN2UBUhw4iaJO87OXMzhMdyYyOoTKAwV0Lw/640?wx_fmt=png)效果演示. png

启动了两个终端，其中右边的终端连接上 redis 以后，发送了 1231，然后左边的终端收到后将收到的消息打印出来。

#### go 代码实现

接着，我们来看看实际代码编写逻辑。

我们定义一个 Server 的结构体来代表 epoll 的 server。

Conn 是对 golang 原生连接类型 net.Conn 的包装，。

poll 结构体是封装了对 epoll api 的调用。

```
type Server struct {  
   Poll     *poll  
   addr     string  
   listener net.Listener  
   ConnMap  sync.Map  
}

type Conn struct {  
   s    *Server  
   conn *net.TCPConn  
   nfd  int  
}


type poll struct {
 EpollFd int
}


```

接着来看下如何启动一个 Server，NewServer 是返回一个 Server 实例，Server 调用 Run 方法后，才算 Server 正式启动了起来。

在 Run 方法里，构建监听连接的 listener，构建一个 epoll 实例，用于后续对事件的监听，同时把监听握手连接和处理连接可读数据分成了两个协程分别用 accept 方法，和 handler 方法执行。

```
func NewServ(addr string) *Server {  
   return &Server{addr: addr, ConnMap: sync.Map{}}  
}  
  
func (s *Server) Run() error {  
   listener, err := net.Listen("tcp", s.addr)  
   if err != nil {  
      return err  
   }  
   s.listener = listener  
   epollFD, err := syscall.EpollCreate1(0)  
   if err != nil {  
      return err  
   }  
   s.Poll = &poll{EpollFd: epollFD}  
   go s.accept()  
   go s.handler()  
   ch := make(chan int)  
   <-ch  
   return nil  
}


```

accept 方法里执行的逻辑就是将握手完成的链接从全连接队列里取出来，将其连接的文件描述符和连接存储到一个 map 里， 然后将对应的文件描述符通过 epoll 的 epollCtl 系统调用监听它的可读事件，后续客户端再使用这个连接发送数据时，epoll 就能监听到了。

```
func (s *Server) accept() {  
   for {  
      acceptConn, err := s.listener.Accept()  
      if err != nil {  
         return  
      }  
      var nfd int  
      rawConn, err := acceptConn.(*net.TCPConn).SyscallConn()  
      if err != nil {  
         log.Error(err.Error())  
         continue  
      }  
      rawConn.Control(func(fd uintptr) {  
         nfd = int(fd)  
      })  
      // 设置为非阻塞状态  
      err = syscall.SetNonblock(nfd, true)  
      if err != nil {  
         return  
      }  
      err = s.Poll.AddListen(nfd)  
      if err != nil {  
         log.Error(err.Error())  
         continue  
      }  
      c := &Conn{  
         conn: acceptConn.(*net.TCPConn),  
         nfd:  nfd,  
         s:    s,  
      }  
      s.ConnMap.Store(nfd, c)  
   }  
}


```

handler 里的逻辑则是通过 epoll Wait 系统调用等待可读事件产生，到达后，根据事件的文件描述符找到对应连接，然后读取对应连接的数据。

```
func (s *Server) handler() {  
   for {  
      events, err := s.Poll.WaitEvents()  
      if err != nil {  
         log.Error(err.Error())  
         continue  
      }  
      for _, e := range events {  
         connInf, ok := s.ConnMap.Load(int(e.FD))  
         if !ok {  
            continue  
         }  
         conn := connInf.(*Conn)  
         if IsClosedEvent(e.Type) {  
            conn.Close()  
            continue  
         }  
         if IsReadableEvent(e.Type) {  
            buf := make([]byte, 1024)  
            rd, err := conn.Read(buf)  
            if err != nil && err != syscall.EAGAIN {  
               conn.Close()  
               continue  
            }  
            fmt.Println("收到消息", string(buf[:rd]))  
         }  
      }  
   }  
}


```

主干代码是比较容易理解的，但是用 golang 使用 epoll 时有几个点 需要注意下:

**「第一点是 IsReadableEvent 的判断方式」**，epoll 的每个 event 都有一个位掩码，位掩码是什么意思呢？比如 EPOLLIN 的值 是 0x1，二进制就是 00000001,EPOLLHUP 的值是 0x10，二进制表示是 00010000，那么 epoll wait 系统调用的 event 要如何同时表示同一个文件描述符同时拥有这两个事件呢? epoll 的 event 会将对应的位掩码设置为和对应事件一致，比如同时拥有 EPOLLIN 和 EPOLLHUP，那么 event 的值将会是 00010001，所以利用与位运算是不是就能判断 event 是否具有某个事件了。因为 1 只有与 1 进行与运算结果才为 1。

```
func IsReadableEvent(event uint32) bool {
 if event&syscall.EPOLLIN != 0 {
  return true
 }
 return false
}


```

**「第二点是如何读取连接的数据」**, 我们后续要达到的目的是在同一个事件循环里能处理多个连接，所以要保证读取连接中的数据时不能阻塞，通过调用 golang 的 net.Conn 下的 read 方法是阻塞的，其 read 实现最终会调用到下面👇🏻👇🏻👇🏻的这个方法。

```
func (fd *FD) Read(p []byte) (int, error) {  
   if err := fd.readLock(); err != nil {  
      return 0, err  
   }  
   defer fd.readUnlock()  
   if len(p) == 0 {  
      // If the caller wanted a zero byte read, return immediately  
      // without trying (but after acquiring the readLock).      // Otherwise syscall.Read returns 0, nil which looks like      // io.EOF.      // TODO(bradfitz): make it wait for readability? (Issue 15735)      return 0, nil  
   }  
   if err := fd.pd.prepareRead(fd.isFile); err != nil {  
      return 0, err  
   }  
   if fd.IsStream && len(p) > maxRW {  
      p = p[:maxRW]  
   }  
   for {  
      n, err := ignoringEINTRIO(syscall.Read, fd.Sysfd, p)  
      if err != nil {  
         n = 0  
         if err == syscall.EAGAIN && fd.pd.pollable() {  
            if err = fd.pd.waitRead(fd.isFile); err == nil {  
               continue  
            }  
         }  
      }  
      err = fd.eofError(n, err)  
      return n, err  
   }  
}


```

这个方法会在 for 循环中判断系统调用 syscall.Read 的返回，如果是 syscall.EAGAIN 那么会让当前协程睡眠，等待被唤醒。

> ❝
> 
> syscall.EAGAIN 错误是在非阻塞 io 进行读写时才有可能产生的，在读取数据时，如果发现读缓冲区没有数据到达，则返回这个 syscall.EAGAIN 错误，在写入数据时，如果写缓冲区满了，也会返回这个错误。
> 
> ❞

既然 golang 的 net.Conn 下的 read 方法是阻塞的，那么我们就自己实现下 conn 的 Read 方法。

```
func (c *Conn) Read(p []byte) (n int, err error) {  
   rawConn, err := c.conn.SyscallConn()  
   if err != nil {  
      return 0, err  
   }  
   rawConn.Read(func(fd uintptr) (done bool) {  
      n, err = syscall.Read(int(fd), p)  
      if err != nil {  
         return true  
      }  
      return true  
   })  
   return  
}


```

👆🏻👆🏻的 Read 方法是我们自定义的 Conn 类型实现的 Read 方法，原生的连接类型是 net.Conn，它有一个 SyscallConn 能够获取到更加底层的连接类型，从这个类型能够获取到该网络连接的文件描述符 fd，我们通过直接调用系统调用 syscall.Read 来从该网络连接读取数据。 并且碰到错误则直接返回。后续 syscall.EAGAIN 错误会交给上层 handler 方法去进行处理。

总结
--

这节算是用 golang 去演示了下如何对 epoll api 的调用，并且能够实现最简单的客户端服务端通信，下一节我会讲解 redis 的网络模型是怎么样的，你可以从中了解到经常说的 redis 的单线程具体是指什么，了解到 reactor 网络模型是怎样的？