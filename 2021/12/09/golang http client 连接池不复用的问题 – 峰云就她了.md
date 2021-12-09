> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xiaorui.cc](http://xiaorui.cc/archives/7172)

> 当 httpclient 返回值为不为空，只读取 response header，但不读 body 内容就 response.Body.Close()，那么连接会被主动关闭，得不到复用。

### 摘要

当 httpclient 返回值为不为空，只读取 response header，但不读 body 内容就 response.Body.Close()，那么连接会被主动关闭，得不到复用。

**测试代码如下:**

> 正如大家所想，除了 HEAD Method 外，很少会有只读取 header 的需求吧。

话说，golang httpclient 需要注意的地方着实不少。

*   如没有 response.Body.Close()，有些小场景造成 persistConn 的 writeLoop 泄露。
*   如 header 和 body 都不管，那么会造成泄露的连接干满连接池，后面的请求只能是`短连接`。

### 上下文

由于某几个业务系统会疯狂调用各区域不同的 k8s 集群，为减少跨机房带来的时延、新老 k8s 集群 api 兼容、减少 k8s api-server 的负载，故而开发了 k8scache 服务。在部署运行后开始对该服务进行监控，发现 metrics 呈现的 QPS 跟连接数不成正比，qps 为 1500，连接数为 10 个。开始以为触发 idle timeout 被回收，但通过历史监控图分析到连接依然很少。😅

按照对 k8scache 调用方的理解，他们经常粗暴的开启不少协程来对 k8scache 进行访问。已知默认的 golang httpclient transport 对连接数是有默认限制的，连接池总大小为 100，每个 host 连接数为 2。当并发对某 url 进行请求时，当无法归还连接池，也就是超过连接池大小的连接会被主动 clsoe()。所以，我司的 golang 脚手架中会对默认的 httpclient 创建高配的 transport，不太可能出现连接池爆满被 close 的问题。

如果真的是连接池爆了? 谁主动挑起关闭，谁就有 tcp time-wait 状态，但通过 netstat 命令只发现少量跟 k8scache 相关的 time-wait。

### 排查问题

> 已知问题, 为隐藏敏感信息，索性使用简单的场景设立问题的 case

**tcpdump 抓包分析问题?**

包信息如下，通过最后一行可以确认是由客户端主动触发 `RST连接重置` 。触发 RST 的场景有很多，但常见的有 tw_bucket 满了、tcp 连接队列爆满且开启 tcp_abort_on_overflow、配置 so_linger、读缓冲区还有数据就给 close。

通过 linux 监控和内核日志可以确认不是内核配置的问题，配置 so_linger 更不可能。😅 大概率就一个可能，关闭未清空读缓冲区的连接。

通过 lsof 找到进程关联的 TCP 连接，然后使用 ss 或 netstat 查看读写缓冲区。信息如下，recv-q 读缓冲区确实是存在数据。这个缓冲区字节一直未读，直到连接关闭引发了 rst。

**strace 跟踪系统调用**

通过系统调用可分析出，貌似只是读取了 header 部分了，还未读到 body 的数据。

**逻辑代码**

那么到这里，可以大概猜测问题所在，找到业务方涉及到 httpclient 的逻辑代码。伪代码如下，跟上面的结论一样，只是读取了 header，但并未读取完 response body 数据。

> 还以为是特殊的场景，结果是使用不当，把请求投递过去后只判断 http code？ 真正的业务 code 是在 body 里的。😅

**如何解决**

不细说了，把 header length 长度的数据读完就可以了。

### 分析问题

先不管别人使用不当，再分析下为何出现短连接，连接不能复用的问题。

为什么不读取 body 就出问题？其实 http.Response 字段描述中已经有说明了。当 Body 未读完时，连接可能不能复用。

众所周知，golang httpclient 要注意 response Body 关闭问题，但上面的 case 确实有关了 body，只是非常规地没去读取 reponse body 数据。这样会造成连接异常关闭，继而引起连接池不能复用。

一般 http 协议解释器是要先解析 header，再解析 body，结合当前的问题开始是这么推测的，连接的 readLoop 收到一个新请求，然后尝试解析 header 后，返回给调用方等待读取 body，但调用方没去读取，而选择了直接关闭 body。那么后面当一个新请求被 transport roundTrip 再调度请求时，readLoop 的 header 读取和解析会失败，因为他的读缓冲区里有前面未读的数据，必然无法解析 header。按照常见的网络编程原则，协议解析失败，直接关闭连接。

想是这么想的，但还是看了 golang net/http 的代码，结果不是这样的。😅

### 分析源码

httpclient 每个连接会创建读写协程两个协程，分别使用 reqch 和 writech 来跟 roundTrip 通信。上层使用的 response.Body 其实是经过多次封装的，一次层封装的 body 是直接跟 net.conn 进行交互读取，二次封装的 body 则是加强了 close 和 eof 处理的 bodyEOFSignal。

当未读取 body 就进行 close 时，会触发 earlyCloseFn() 回调，看 earlyCloseFn 的函数定义，在 close 未见 io.EOF 时才调用。自定义的 earlyCloseFn 方法会给 readLoop 监听的 waitForBodyRead 传入 false, 这样引发 alive 为 false 不能继续循环的接收新请求，只能是退出调用注册过的 defer 方法，关闭连接和清理连接池。

bodyEOFSignal 的 Close()

最终会调用 persistConn 的 close(), 连接关闭并关闭 closech。

### 总之

同事的 httpclient 使用方法有些奇怪，除了 head method 之外，还真想不到有不读取 body 的请求。所以，大家知道 httpclient 有这么一回事就行了。

另外，一直觉得 net/http 的代码太绕，没看过一些介绍直接看代码很容易陷进去，有时间专门讲讲 http client 的实现。

* * *

大家觉得文章对你有些作用！ 如果想赏钱，可以用微信扫描下面的二维码，感谢!  
另外再次标注博客原地址  [xiaorui.cc](http://xiaorui.cc/)