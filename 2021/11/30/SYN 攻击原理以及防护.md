> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/157312196)

### SYN 攻击

### 简介

```
攻击者短时间伪造不同IP地址的SYN报文，快速占满backlog队列，使服务器不能为正常用户服务。
SYN攻击属于DOS攻击的一种， 它利用TCP协议缺陷，通过发送大量的半连接请求，耗费CPU和内存资源

```

### TCP 连接

```
TCP基础知识

TCP标志位

    SYN  建立连接  
    ACK  表示响应  
    FIN  关闭连接


TCP连接传输数据

    seq 是随机生成的一个数。另一端收到后会返回ack(seq+1)数据让发出端进行验证
    ack ack就是收到的上一次另一端发送的seq+1的数据

TCP,可靠的数据传输协议。

一个TCP生命周期主要经历三个步骤
    1建立连接
        三次握手
            1,client->server. SYN=1,seq=x
            2,server->client. SYN=1,ACK=1,seq=y,ack=x+1
            3,client->server. ACK=1,seq=x+1,ack=y+1
    2传输数据
        数据通信
    3关闭连接
        四次挥手
            1,client->server. FIN=1,seq=x
            2,server->client. ACK=1,ack=x+1,seq=y
            3,server->client. FIN=1,ACK=1,seq=z,ack=x+1
            4,client->server. ACK=1,seq=x+1,ack=z+1

```

### 原理

```
在TCP建立连接的时候,按照上面建立连接的步骤，第2步,服务器响应客户端的时候,
如果步骤3没有给服务器响应，则服务器会一直执行步骤2,发送数据进行尝试连接。
直到步骤3客户端给服务器响应后，停止尝试。
    这样就给黑客创造了可乘之机，用不存在的地址访问服务器,服务器会一直执行上面的步骤2，
给客户端发送连接数据，由于地址是不存在的，不会执行第三步，所以服务器就会在超时时间内，
一直尝试发送，这个就是半连接请求。如果模拟成千上万个不存在的地址进行访问，则会快速占满
处理队列，不能给正常用户服务。这就是SYN攻击

```

### SYN 攻击预防

1，减少 SYN 重试次数

```
net.ipv4.tcp_syn_retries = 6 
sysctl -w net.ipv4.tcp_synack_retries=3     
sysctl -w net.ipv4.tcp_syn_retries=3

```

2, 增大 [backlog 队列](https://www.zhihu.com/search?q=backlog%E9%98%9F%E5%88%97&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22article%22%2C%22sourceId%22%3A157312196%7D)

```
net.core.netdev_max_backlog  接收自网卡、但未被内核协议栈处理的报文队列长度
net.ipv4.tcp_max_syn_backlog -SYN_RCVD状态连接的最大个数

```

3，超出处理能力时，对新来的 SYN 丢弃连接 (影响用户体验)

```
net.ipv4.tcp_abort_on_overflow

```

4，生成验证 cookie，重连

```
net.ipv4.tcp_syncookies = 1
 当syn的队列满员后,新的syn连接不会进入A队列，计算出Cookie再以SYN+ACK返回客户端，不再尝试连接
 正常客户端发报文时,服务端根据报文中携带的cookie重新恢复连接

 synccookies是妥协版的TCP协议，失去了很多功能，三次握手也放弃了。不到万不得已，不用这种方法

```