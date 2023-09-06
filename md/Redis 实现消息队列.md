> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6kzEjB0h7Y-vUBCbYUmLvg)

> Redis 实现消息队列的核心思想是利用 Redis 的列表数据结构，将消息作为一个个元素插入到列表中，然后通过消费者以 FIFO（先进先出）的方式逐一取出消息，从而实现队列的功能。

**一、简介**

      消息队列是一种常用的通信模式，用于解耦消息的发送者和接收者，并实现异步处理。目前基于 redis（版本大于 5.0）实现消息队列的方案如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwLv1xWbJNiat7UfJgfKh6Do1Qy5jaGjI0t9phv3P8H2V3viaoFMNiaklbA/640?wx_fmt=png)

消息队列的定义及实现方式：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmw7RUice2HPca8BZGcFDbwBqyWyZIKjbLwkq4yySkFeiaic3L8N9Eiaveofg/640?wx_fmt=png)

消息队列需要满足的要求：

<table><tbody><tr><td valign="top" width="12">1</td><td valign="top" width="98">顺序一致</td><td width="408" valign="top">即要保证消息发送的顺序和消费的顺序是一致的，不一致的话可能会导致业务上的错误。</td></tr><tr><td valign="top" width="12">2</td><td valign="top" width="128">消息确认机制</td><td width="408" valign="top">对于一个已经被消费的消息 (已经收到 ACK) 不能再次被消费。</td></tr><tr><td valign="top" width="12">3<br></td><td valign="top" width="128">消息持久化</td><td width="408" valign="top">要具有持久化的能力，避免消息丢失，这样当消费者异常宕机导致再次重启后需要重新消费消息时可以再次获取。</td></tr></tbody></table>

**二、基于 LIST 结构模拟消息队列**  

**备注：一对一模式**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwD0ETSo1vILLl6HgibQ5t6BZib0PHfexg3hrZ1cZ0w9scA8Tzz4YcTNDA/640?wx_fmt=png)**案例如下：**

**消息发送端：**

```
127.0.0.1:6379> lpush mesg 1 2
(integer) 2
127.0.0.1:6379> lpush mesg 3 4 5
(integer) 3
127.0.0.1:6379>

```

**消息接收端：**

```
127.0.0.1:6379> brpop mesg 20
1) "mesg"
2) "1"
(11.11s)
127.0.0.1:6379> brpop mesg 20
1) "mesg"
2) "2"
127.0.0.1:6379> brpop mesg 20
1) "mesg"
2) "3"
(12.37s)
127.0.0.1:6379>  brpop mesg 20
1) "mesg"
2) "4"
127.0.0.1:6379>  brpop mesg 20
1) "mesg"
2) "5"
127.0.0.1:6379>  brpop mesg 20
(nil)
(20.09s)
127.0.0.1:6379>

```

**总结如下：**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwVk55d474U1H7ibJD1S8ozSTkHeRhG3WzUMQlfURVVmQfqSjKAr4Odkw/640?wx_fmt=png)

**三、基于 PubSub 的消息队列**

****备注：一对多模式（广播模式）****

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmw0VlibyLyQm0Dawval1Kkqm9LictyPsdZxSVLVLUUbNOqApHL4ET9Tutw/640?wx_fmt=png)

**案例如下：**  

**发布端：**

```
127.0.0.1:6379> publish order.msg01 msg001
(integer) 2
127.0.0.1:6379> publish order.msg01 msg002
(integer) 2
127.0.0.1:6379> publish order.msg01 msg003
(integer) 2
127.0.0.1:6379> publish order.msg02 msg004

```

**订阅端：**

**1、指定 channel 的订阅**

```
127.0.0.1:6379> SUBSCRIBE order.msg01
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "order.msg01"
3) (integer) 1
1) "message"
2) "order.msg01"
3) "msg001"
1) "message"
2) "order.msg01"
3) "msg002"
1) "message"
2) "order.msg01"
3) "msg003"

```

****2、模糊 channel 的订阅****

```
127.0.0.1:6379> PSUBSCRIBE order.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "order.*"
3) (integer) 1
1) "pmessage"
2) "order.*"
3) "order.msg01"
4) "msg001"
1) "pmessage"
2) "order.*"
3) "order.msg01"
4) "msg002"
1) "pmessage"
2) "order.*"
3) "order.msg01"
4) "msg003"
1) "pmessage"
2) "order.*"
3) "order.msg02"
4) "msg004"

```

**运行情况如下：**![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwwvTYyS1ux4SNj2bPicMWNcIWGzX9GzMspEBpqTGwc8zxrjtXspD3ZVg/640?wx_fmt=png)  

**总结如下：**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmw9z80jn3lpfFCsHDMafv2033iaMQQCFtlHicQk5Pjv7Nruic9FpYCxKkqw/640?wx_fmt=png)

**四、基于 Stream 的消息队列**

**详细内容参考**：https://www.knowledgedict.com/tutorial/redis-streams.html

**4.1、发送消息的命令：xadd**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmwgk5uTzphrtypEEnHh9N1AfUJnbciaGR5k1jbibmb4icl2vt8fqEKjLTwg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmw1x6icOcT3Z6iah0mdLd9dUUGRxHQicqjd4h5ibWsjlWX8ibCHSna6aCw3lw/640?wx_fmt=png)

**案例如下：**  

```
10.1.50.157:0>XADD  student * name xiaoli sex 男 age 21
"1691919968658-0"
10.1.50.157:0>XLEN  student
"1"
10.1.50.157:0>

```

**效果如下：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmw9jyL5FDNmm0NsYA1w9Ivv4rpIZFe1EmclurBM8WUPGgoRfrCS4ohYQ/640?wx_fmt=png)

**4.2、读取消息的方式之一：xread**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmwib74kSrVMavr46U3uWw9E83PGzcrqpo4VndbsFWQhQoqAPGeGnhOzzQ/640?wx_fmt=png)

**案例如下：**

**发送端：  
**

```
10.1.50.157:0>XADD  student * name xiaoli sex 男 age 21
"1691919968658-0"
10.1.50.157:0>XLEN  student
"1"
10.1.50.157:0>XADD student * name xiaoli02 sex 女 age 20
"1691920556051-0"
10.1.50.157:0>XADD student * name xiaoli02 sex 女 age 28
"1691920957334-0"
10.1.50.157:0>XADD student * name xiaoli02 sex 女 age 29
"1691920975931-0"
10.1.50.157:0>

```

**消费端：**  

```
10.1.50.157:0>XREAD COUNT 1 BLOCK 0 STREAMS student $
1) 1) "student"
   2) 1) 1) "1691920851885-0"
         2) 1) "name"
            2) "xiaoli02"
            3) "sex"
            4) "女"
            5) "age"
            6) "21"

```

**数据展示：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmw9ibPapLj1Pv545425lVeHwPf5y3IScVY2KAmUcj5PTjH2OibC2YdByKQ/640?wx_fmt=png)

**总结如下：**

**![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwefKn7rXWTVTxF2m8PibSgTJia7ZN0slIdc9HgFhPbWaiaJic3AcbsXZutg/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwH1yRmtptfSwqk4CZeYcaSGGr9d8AhI8zA3tO5iaGovoiczn2m8ZNhtCw/640?wx_fmt=png)

****4.3、读取消息的方式之二：消费者组****

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmwgev0CxsTem28vqfCT5S2I9G5NUNkWicTav6HwP6HvywNE15nWnrosJQ/640?wx_fmt=png)

**常见使用命令如下：**

**4.3.1、创建消费者组命令：  
**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwkYkAINVffP7tAQVD0yvvrVicXFXib14K4c9ukiaGia8aooRHh6GB38GiceQ/640?wx_fmt=png)

**参考案例：**

```
10.1.50.157:0>XGROUP CREATE student gs01 0
"OK"

```

**4.3.2、从消费者组读取消息命令：**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwbotBk65DGKqkETC4ib7FtMicl0LcpBGMwVnq66G4YlULxnogqoFGSUeg/640?wx_fmt=png)

**参考案例：**

```
 -- 消费者c01
10.1.50.157:0>XREADGROUP GROUP gs01 c01 COUNT 1 BLOCK 2000 STREAMS student >
1) 1) "student"
   2) 1) 1) "1691920851885-0"
         2) 1) "name"
            2) "xiaoli02"
            3) "sex"
            4) "女"
            5) "age"
            6) "21"
10.1.50.157:0>XREADGROUP GROUP gs01 c01 COUNT 1 BLOCK 2000 STREAMS student >
1) 1) "student"
   2) 1) 1) "1691920917913-0"
         2) 1) "name"
            2) "xiaoli02"
            3) "sex"
            4) "女"
            5) "age"
            6) "25"
 -- 消费者c02 
10.1.50.157:0>XREADGROUP GROUP gs01 c02 COUNT 1 BLOCK 2000 STREAMS student >
1) 1) "student"
   2) 1) 1) "1691920870086-0"
         2) 1) "name"
            2) "xiaoli02"
            3) "sex"
            4) "女"
            5) "age"
            6) "23"                  

```

**XACK 命令：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwuUjRGYWnRufRHulN4YU2HD31wjVRq2Mffk6Ujqu4rt8pHveiclSJM8A/640?wx_fmt=png)

**参考案例：**

```
10.1.50.157:0>XACK student gs01 1691920917913-0
"1"

```

**XPENDING 命令：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8Ecnclgmw8KCUWmOxfY0D667uOX41mXZRZJsib3TNVeicGnmzVWOG1RWpSv46TFuA/640?wx_fmt=png)

**参考案例：**  

```
10.1.50.157:0>XPENDING student gs01 - + 10
1) 1) "1691919968658-0"
   2) "c01"
   3) "2523762"
   4) "1"
2) 1) "1691920556051-0"
   2) "c01"
   3) "2500637"
   4) "1"

```

**4.4、在 java 中的应用**  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwCvicMFhKKGeKdunDbH1NhdAuvrRP5SFibiaBK8LNtKk4nC4ToSxGOiaP0Q/640?wx_fmt=png)

**4.5、总结**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwQOPm5YDVjST7yiaeuPvjLk2n2yaPvj9uwLpDaQyawad4FTAuibo2zkJw/640?wx_fmt=png)  
**五、全文总结**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwLv1xWbJNiat7UfJgfKh6Do1Qy5jaGjI0t9phv3P8H2V3viaoFMNiaklbA/640?wx_fmt=png)

       对于中小型企业，对于消息机制要求不算太严格，推荐使用 Stream，基本上满足要求了。但是对于大型企业，对消息要求比较严格，还是推荐使用更更专业的消息中间件，像：RabbitMQ、Kafka 等等，因为 Stream 只满足消费者的 ACK 确认机制，生产者并不满足。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/CFfeGLBwm45WuicQd0CrTFKc8EcnclgmwPJOsEEbxXXYcsWqQhkz1DrOTZxJK4vLZdT3BxQdOj1SEeEdY9PJ2yg/640?wx_fmt=png)