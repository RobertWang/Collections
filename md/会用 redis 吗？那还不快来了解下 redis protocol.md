> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7119444136003895332) Client: INCR X Server: 1 Client: INCR X Server: 2 Client: INCR X Server: 3 Client: INCR X Server: 4

事实上客户端只想得到最终的结果，但是每次客户端都需要等待服务器端返回结果之后，才能发送下一次的命令。这样就会导致一个叫做 RTT(Round Trip Time) 的时间浪费。

虽然每次 RTT 的时间不长，但是累计起来也是一个非常客观的数字。

那么可不可以将所有的客户端命令放在一起发送给服务器呢? 这个优化就叫做 Pipeline。

piepline 的意思就是客户端可以在没有收到服务器端返回的时候继续向服务器端发送命令。

上面的命令可以用 pipline 进行如下改写：

```
(printf "INCR X\r\nINCR X\r\nINCR X\r\nINCR X\r\n"; sleep 1) | nc localhost 6379
:1
:2
:3
:4
复制代码

```

因为 redis 服务器支持 TCP 协议进行连接，所以我们可以直接用 nc 连到 redis 服务器中执行命令。

在使用 pipline 的时候有一点要注意，因为 redis 服务器会将请求的结果缓存在服务器端，等到 pipline 中的所有命令都执行完毕之后再统一返回，所以如果服务器端返回的数据比较多的情况下，需要考虑内存占用的问题。

那么 pipline 仅仅是为了减少 RTT 吗？

熟悉操作系统的朋友可能有听说过用户空间和操作系统空间的概念，从用户输入读取数据然后再写入到系统空间中，这里涉及到了一个用户空间的切换，在 IO 操作中，这种空间切换或者拷贝是比较耗时的，如果频繁的进行请求和响应，就会造成这种频繁的空间切换，从而降低了系统的效率。

使用 pipline 可以一次性发送多条指令，从而有效避免空间的切换行为。

Redis 中的 Pub/Sub
----------------

和 Pub/Sub 相关的命令是 SUBSCRIBE, UNSUBSCRIBE 和 PUBLISH。

为什么要用 Pub/Sub 呢？其主要的目的就是解耦，在 Pub/Sub 中消息发送方不需要知道具体的接收方的地址，同样的对于消息接收方来说，也不需要知道具体的消息发送方的地址。他们只需要知道关联的主题即可。

subscribe 和 publish 的命令比较简单，我们举一个例子，首先是客户端 subscribe topic：

```
redis-cli -h 127.0.0.1
127.0.0.1:6379> subscribe topic
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "topic"
3) (integer) 1
复制代码

```

然后在另外一个终端，调用 publish 命令：

```
redis-cli -h 127.0.0.1
127.0.0.1:6379> publish topic "what is your name?"
(integer) 1
复制代码

```

可以看到客户端会收到下面的消息：

```
1) "message"
2) "topic"
3) "what is your name?"
复制代码

```

RESP protocol
=============

RESP 协议有 5 种类型，分别是 imple Strings, Errors, Integers, Bulk Strings 和 Arrays。

不同的类型以消息中的第一个 byte 进行区分，如下所示：

<table><thead><tr><th>类型</th><th>第一个 byte</th></tr></thead><tbody><tr><td>Simple Strings</td><td>+</td></tr><tr><td>Errors</td><td>-</td></tr><tr><td>Integers</td><td>:</td></tr><tr><td>Bulk Strings</td><td>$</td></tr><tr><td>Arrays</td><td>*</td></tr></tbody></table>

protocol 中不同的部分以 "\r\n" (CRLF) 来进行区别。

Simple Strings
--------------

Simple Strings 的意思是简单的字符串。

通常用在服务器端的返回中，这种消息的格式就是 "+" 加上文本消息，最后以 "\r\n" 结尾。

比如服务器端返回 OK, 那么对应的消息就是：

```
"+OK\r\n"
复制代码

```

上面的消息是一个非二进制安全的消息，如果想要发送二进制安全的消息，则可以使用 Bulk Strings。

什么是非二进制安全的消息呢？对于 Simple Strings 来说，因为消息是以 "\r\n" 结尾，所以消息中间不能包含 "\r\n" 这两个特殊字符，否则就会产生错误的含义。

Bulk Strings
------------

Bulk Strings 是二进制安全的。这是因为 Bulk Strings 包含了一个字符长度字段，因为是根据长度来判断字符长度的，所以并不存在根据字符中某个特定字符来判断是否字符结束的缺点。

具体而言 Bulk Strings 的结构是 "$"+ 字符串长度 +"\r\n"+ 字符串 +"\r\n"。

以 OK 为例，如果以 Bulk Strings 来表示，则如下所示:

```
"$2\r\nok\r\n"
复制代码

```

Bulk Strings 还可以包含空字符串：

```
"$0\r\n\r\n"
复制代码

```

当然还可以表示不存在的 Null 值：

```
"$-1\r\n"
复制代码

```

RESP Integers
-------------

这是 redis 中的整数表示，具体的格式是 ":"+ 整数 +"\r\n"。

比如 18 这个整数就可以用下面的格式来表示：

```
":18\r\n"
复制代码

```

RESP Arrays
-----------

redis 的多个命令可以以 array 来表示，服务器端返回的多个值也可以用 arrays 来表示。

RESP Arrays 的格式是 "*"+ 数组中的元素个数 + 其他类似的数据。

所以 RESP Arrays 是一个复合结构的数据。比如一个数组中包含了两个 Bulk Strings:"redis","server" 则可以用下面的格式来表示：

```
"*2\r\n$5\r\nredis\r\n$6\r\nserver\r\n"
复制代码

```

RESP Arrays 中的原始不仅可以使用不同类型，还能包含 RESP Arrays，也就是 array 的嵌套：

```
"*3\r\n$5\r\nredis\r\n$6\r\nserver\r\n*1\r\n$4\r\ngood\r\n"
复制代码

```

为了方便观察，我们将上面的消息格式一下：

```
"*3\r\n
$5\r\nredis\r\n
$6\r\nserver\r\n
*1\r\n
$4\r\ngood\r\n"
复制代码

```

上面的消息是一个包含三个元素的数组，前面两个元素是 Bulk Strings，最后一个是包含一个元素的数组。

RESP Errors
-----------

最后，RESP 还可以表示错误消息。RESP Errors 的消息格式是 "-"+ 字符串，如下所示：

```
"-Err something wrong\r\n"
复制代码

```

一般情况下，"-" 后面的第一个单词表示的是错误类型，但是这只是一个约定俗成的规定，并不是 RESP 协议中的强制要求。

另外，经过对比，大家可能会发现 RESP Errors 和 Simple Strings 是消息格式是差不多的。

这种对不同消息类型的处理是在客户端进行区分的。

Inline commands
===============

如果完全按 RESP 协议的要求，当我们连接到服务器端的时候需要包含 RESP 中定义消息的所有格式，但是这些消息中包含了额外的消息类型和回车换行符，所以直接使用协议来执行的话会比较困惑。

于是 redis 还提供一些内联的命令，也就是协议命令的精简版本，这个精简版本去除了消息类型和回车换行符。

我们以 "get world" 这个命令为例。来看下不同方式的连接情况。

首先是使用 redis-cli 进行连接：

```
redis-cli -h 127.0.0.1
127.0.0.1:6379> get world
"hello"
复制代码

```

因为 redis-cli 是 redis 的客户端，所以可以直接使用 inline command 来执行命令。

如果使用 telnet, 我们也可以使用同样的命令来获得结果：

```
telnet 127.0.0.1 6379
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
get world
$5
hello
复制代码

```

可以看到返回的结果是 "$5\r\nhello\r\n"。

如果要使用协议消息来请求 redis 服务器应该怎么做呢？

我们要请求的命令是 "get world", 将其转换成为 RESP 的消息则是：

```
"*2\r\n$3\r\nget\r\n$5\r\nworld\r\n"
复制代码

```

我们尝试一下将上述命令使用 nc 传递到 redis server 上：

```
(printf "*2\r\n$3\r\nget\r\n$5\r\nworld\r\n"; sleep 1) |  nc localhost 6379
-ERR Protocol error: expected '$', got ' '
复制代码

```

很遗憾我们得到了 ERR，那么是不是不能直接使用 RESP 消息格式进行传输呢？当然不是，上面的问题在于`$`符号是一个特殊字符，我们需要转义一下：

```
(printf "*2\r\n\$3\r\nget\r\n\$5\r\nworld\r\n"; sleep 1) |  nc localhost 6379
$5
hello
复制代码

```

可以看到输出的结果和直接使用 redis-cli 一致。

总结
==

以上就是 RESP 协议的基本内容和手动使用的例子，有了 RESP，我们就可以根据协议中定义的格式来创建 redis 客户端。

可能大家又会问了，为什么只是 redis 客户端呢？有了协议是不是 redis 服务器端也可以创建呢？答案当然是肯定的，只需要按照协议进行消息传输即可。主要的问题在于 redis 服务器端的实现比较复杂，不是那么容易实现的。

我正在参与掘金技术社区创作者签约计划招募活动，[点击链接报名投稿](https://juejin.cn/post/7112770927082864653 "https://juejin.cn/post/7112770927082864653")。