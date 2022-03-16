> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/323254081)

代码库地址：[https://github.com/garyburd/redigo](https://link.zhihu.com/?target=https%3A//github.com/garyburd/redigo)

*   1：连接池
*   2：发送命令
*   3：解析结果

1：连接池

连接池结构体如下：

```
type Pool struct {
   // Dial is an application supplied function for creating and configuring a
   // connection.
   //
   // The connection returned from Dial must not be in a special state
   // (subscribed to pubsub channel, transaction started, ...).
   Dial func() (Conn, error)  //生成网络连接对象

   // TestOnBorrow is an optional application supplied function for checking
   // the health of an idle connection before the connection is used again by
   // the application. Argument t is the time that the connection was returned
   // to the pool. If the function returns an error, then the connection is
   // closed.
   TestOnBorrow func(c Conn, t time.Time) error  //测试连接是否通畅

   // Maximum number of idle connections in the pool.
   MaxIdle int  //最大空闲连接数

   // Maximum number of connections allocated by the pool at a given time.
   // When zero, there is no limit on the number of connections in the pool.
   MaxActive int //最大活动（正在执行任务）连接数

   // Close connections after remaining idle for this duration. If the value
   // is zero, then idle connections are not closed. Applications should set
   // the timeout to a value less than the server's timeout.
   IdleTimeout time.Duration //空闲连接超时时间，超时会释放

   // If Wait is true and the pool is at the MaxActive limit, then Get() waits
   // for a connection to be returned to the pool before returning.
   Wait bool //当到达最大活动连接时，是否阻塞

   chInitialized uint32 // set to 1 when field ch is initialized 初始化标记

   mu     sync.Mutex    // mu protects the following fields 锁
   closed bool          // set to true when the pool is closed. 连接池关闭标记
   active int           // the number of open connections in the pool 连接总数
   ch     chan struct{} // limits open connections when p.Wait is true 用于实现阻塞逻辑
   idle   idleList      // idle connections 双向链表，存放空闲连接
}

type idleList struct { //空闲连接链表
   count       int //空闲连接数
   front, back *idleConn //空闲连接信息
}

type idleConn struct {  //空闲连接信息
   c          Conn      //连接接口
   t          time.Time //加入空闲队列的时间，用于判断空闲超时
   next, prev *idleConn //双向链表指针
}

```

1：空闲连接池实现

空闲连接池存在一个双向链表中，一个连接用完后回收，就会从表头插入这个链表，当需要一个连接时也是从链表的表头取，从表头插入的时候会写入当前时间，所以链表是一个按时间倒序的链表，判断一个连接有没有空闲超时，就从链表表尾开始判断，如果空闲超时，就从表尾移除并关闭连接。从表头插入一个元素后，如果空闲数量超过阈值，会从表尾移除一个元素，保证空闲的连接数不超过指定的值，防止空闲的连接过多浪费系统资源

获取可用连接过程

对应方法：Pool.Get

1：删除超时空闲连接。从链表的表尾开始往表头判断，如果到达空闲超时时间，从链表中移除并释放连接

2：从空闲链表中获取可用连接。从链表表头往表尾查找可用的连接，如果连接能 ping 通，将当前连接从链表中移除，返回当前连接

3：建立一个新的连接。如果空闲连接为空，或者空闲链表中的所有连接都不可用，则从新建立一个新的连接并返回

在获取连接的时候删除超时空闲连接，这是一种惰性删除的方式

以上实现方式同时解决了断开重连的问题。

如在某一时刻 redis server 重启了，那么空闲链表中的连接都会变得不可用，由于在获取可用连接前会先 ping 一下，但是所有连接都 ping 不通，最后只能重新建立，而空闲连接会在空闲时间超时后自动释放，于是很好的解决了断开重连的问题，同时也做了一些牺牲，不 ping 一下怎么知道连接是否可用，每次获取可用连接的时候都 ping 了一下，但是大部分时候连接都是可用的。

回收可用连接：

对应方法：pooledConnection.Close -> Pool.put

1：终止相关操作，如果是事务 / 监听 / 订阅，停止相关操作

2：将连接放入空闲链表。将连接存入链表表头，如果空闲连接数量超过阈值，就将表尾元素移除并关闭连接

2：发送命令 & 接收回复并解析

```
func (c *conn) Do(cmd string, args ...interface{}) (interface{}, error) {
   return c.DoWithTimeout(c.readTimeout, cmd, args...)
}

func (c *conn) DoWithTimeout(readTimeout time.Duration, cmd string, args ...interface{}) (interface{}, error) {
   c.mu.Lock()
   pending := c.pending
   c.pending = 0
   c.mu.Unlock()

   if cmd == "" && pending == 0 {
      return nil, nil
   }

   //设置写超时时间
   if c.writeTimeout != 0 {
      c.conn.SetWriteDeadline(time.Now().Add(c.writeTimeout))
   }
   //发送命令内容
   if cmd != "" {
      if err := c.writeCommand(cmd, args); err != nil {
         return nil, c.fatal(err)
      }
   }

   if err := c.bw.Flush(); err != nil {
      return nil, c.fatal(err)
   }

   var deadline time.Time
   if readTimeout != 0 {
      deadline = time.Now().Add(readTimeout)
   }
   //设置读超时时间
   c.conn.SetReadDeadline(deadline)

   if cmd == "” {
      //获取server回复信息并解析
      reply := make([]interface{}, pending)
      for i := range reply {
         r, e := c.readReply()
         if e != nil {
            return nil, c.fatal(e)
         }
         reply[i] = r
      }
      return reply, nil
   }

   var err error
   var reply interface{}
   for i := 0; i <= pending; i++ {
      var e error
      if reply, e = c.readReply(); e != nil {
         return nil, c.fatal(e)
      }
      if e, ok := reply.(Error); ok && err == nil {
         err = e
      }
   }
   return reply, err
}

```

redis 请求协议格式

```
set命令消息格式：
*3\r\n$3\r\nSET\r\n$4\r\nhlxs\r\n$28\r\nhttps://www.cnblogs.com/hlxs\r\n

注释如下：
*3        //参数个数是*开头，3个参数
$3        //参数长度是$开头，命令长度
SET       //命令名称SET
$5        //参数长度是$开头，key长度
mykey     //key的内容
$28       //参数长度是$开头，value长度
https://www.cnblogs.com/hlxs      //value内容

```

参数个数是 * 开头，参数长度是 $ 开头，每个参数通过 \ r\n 隔开

redis 协议格式特点：1 易于实现，2 可以高效地被计算机分析，3 可以很容易地被人类读懂

发送命令其实就是构造请求协议格式，以二进制的方式发送出去

redis 回复协议格式

```
* 状态回复（status reply）的第一个字节是 “+”，如：+ok\r\n
* 错误回复（error reply）的第一个字节是 “-“，如：-ERR unknown command xxx\r\n
  在 "-" 之后，直到遇到第一个空格或新行为止，这中间的内容表示所返回错误的类型
* 整数回复（integer reply）的第一个字节是 “:”，如：:1000\r\n
* 批量回复（bulk reply）的第一个字节是 “$”，如：$6\r\nfoobar\r\n，也是长度加内容的风格
* 多条批量回复（multi bulk reply）的第一个字节是 “*”，如：*5\r\n:1\r\n:2\r\n:3\r\n:4\r\n$6\r\nfoobar\r\n，前面多了数量

```

接收命令其实就是解析以上格式