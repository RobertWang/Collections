> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/NzQOwdntZkHwsiCJpWOH7A)

在之前的文章中我们已经学习了【[**RabbitMQ 的安装**](http://mp.weixin.qq.com/s?__biz=MzIxMDcxMjMyOA==&mid=2247484881&idx=1&sn=e572ce39f713508c02e27ec91df17faa&chksm=976127b6a016aea0011c1b1011879632951585b450af1f931a2bfe2384388c951a95aabdbf53&scene=21#wechat_redirect)】、【**rabbitmq 相关命令**】、【**PHP 扩展 amqp 安装**】。但是如何同过代码去实现队列的消费端和生成端呢？

接下来我们主要介绍 PHP 使用 amqp 扩展实现队列的消费端和生成端。

**消费端**

```
<?php
/**
 * 队列消费者
 */
class RabbitConsumer {
    //rabbitmq相关配置
    private $config;

    //交换机名
    protected $exchange_name;

    //队列名
    protected $queue_name;

    //路由key
    protected $routing_key;

    /**
     * @var AMQPConnection $conn 链接对象
     */
    protected $conn;

    //通信通道
    protected $channel;

    //交换机对象
    protected $exchange;

    /**
     * @var AMQPQueue $queue 队列对象
     */
    protected $queue;

    public function __construct()
    {
        $this->config = array(
            'host'=>'127.0.0.1', //rabbitmq服务端IP地址
            'port'=>5672, //rabbitmq服务端端口
            'login'=>'guest', //登录账号
            'password'=>'guest', //登录密码
            'vhost'=>'/', //rabbitmq服务的虚拟机
            'connect_timeout'=>30 //连接超时时间，单位：秒
        );
        $this->exchange_name = 'amq.direct';
        $this->queue_name = 'test';
        $this->routing_key = 'test';
        $this->connect();
        $this->getExChannel();
        $this->getQueue();
    }

    //连接队列服务器
    public function connect()
    {
        if(!$this->conn) {
            $this->conn = new \AMQPConnection($this->config);
            if(!$this->conn->connect()) {
                throw new Exception('rabbitmq服务连接失败');
            }
        }
        return $this->conn;
    }

    //交换机
    public function getExChannel()
    {
        if (!$this->exchange){
            //创建channel
            $this->channel = new AMQPChannel($this->connect());
            //创建交换机
            $this->exchange = new AMQPExchange($this->channel);

            //设置交换机名
            $this->exchange->setName($this->exchange_name);
        }
        return $this->exchange;
    }

    //获取队列
    public function getQueue()
    {
        if(!$this->queue){
            //创建队列
            $this->queue = new AMQPQueue($this->channel);
            //设置队列名
            $this->queue->setName($this->queue_name);
            //注意如果已经提前创建了队列可以忽略下面这三步。重复创建也不会报错，rabbitmq会自动忽略

            //设置持久化
            $this->queue->setFlags(AMQP_DURABLE);
            //创建队列，如果队列已存在不会重复创建，自动忽略
            $this->queue->declareQueue();
            //队列绑定交换机并设置路由key （如果已经绑定，rabbitmq会自动忽略，不会重复绑定）
            if(!$this->queue->bind($this->exchange_name,$this->routing_key)){
                throw new Exception('队列绑定路由失败！');
            }
        }
        return $this->queue;
    }

    /**
     * 消费队列
     */
    public function consumeQueue()
    {
        while (true){
            //获取队列消息，通过本类的logic方法处理实际的业务逻辑
            $this->queue->consume([$this,'logic']);
        }
    }

    /**
     * 业务处理逻辑
     * @param $resObj
     * @param $queue
     */
    protected function logic($resObj,$queue)
    {
        //消费队列实际业务逻辑
        var_dump(json_decode($resObj->getBody(),true));
        echo PHP_EOL;
        //手动发送ACK应答，如果设置了consume第二个参数为【AMQP_AUTOACK】自动应答，请忽略
        //实际应用中建议大家还是手动ACK应答，如果业务处理异常时可能需要重新处理一遍
        $queue->ack($resObj->getDeliveryTag());
    }

    //对象释放
    public function __destruct()
    {
        $this->conn->disconnect();
    }
}
//实例化消费端对象
$consumer = new RabbitConsumer();
//消费队列，处理实际的业务逻辑
$consumer->consumeQueue();

```

上述代码实现后我们可以通过 cli 模式执行 PHP 脚本，命令格式如下：**php RabbitConsumer.php**

**管理消费端进程**  

cli 模式启动的脚本无法常驻内存中，或者说无法以守护进程的模式在后台启动。此外如果因为一些特殊原因，导致脚本进程被杀死怎么办呢？

关于上面的问题，可以利用之前给大家讲解的【**supervisor 进程管理器**】来启动消费端的脚本。supervisor 是 python 进程管理器，它可以使 cli 模式执行的进程在后台运行，还具备进程异常退出后自动启动的能力。具体操作大家可以看看我之前的 supervisor 相关介绍，我这里给出 rabbitmq 启动脚本的配置文件信息：

```
[program:rabbit_consumer]
directory=/data/www/rabbitmq_test/
command=php rabbit_consumer.php
user=root
autostart=false
autorestart=true
startsecs=1
startretries=3
stderr_logfile=/data/www/rabbitmq_test/rabbitmq_consumer_err.log
stdout_logfile=/data/www/rabbitmq_test/rabbitmq_consumer_out.log

```

配置完成后记得使用【**systemctl restart supervisord**】重启 supervisor 使其生效。

supervisor 启动消费端：**supervisorctl start rabbit_consumer**

**生产端**

```
<?php

/**
 * 队列生成者
 */
class RabbitPublisher
{
    //rabbitmq相关配置
    private $config;

    //交换机名
    protected $exchange_name;

    //路由key
    protected $routing_key;

    /**
     * @var AMQPConnection $conn 链接对象
     */
    protected $conn;

    //通信通道
    protected $channel;

    /**
     * @var AMQPExchange $exchange 交换机对象
     */
    protected $exchange;

    public function __construct()
    {
        $this->config = array(
            'host' => '127.0.0.1', //rabbitmq服务端IP地址
            'port' => 5672, //rabbitmq服务端端口
            'login' => 'guest', //登录账号
            'password' => 'guest', //登录密码
            'vhost' => '/', //rabbitmq服务的虚拟机
            'write_timeout'=>30, //写入超时时间，单位：秒
            'connect_timeout' => 30 //连接超时时间，单位：秒
        );
        $this->exchange_name = 'amq.direct';
        $this->routing_key = 'test';
    }

    //连接队列服务器
    public function connect()
    {
        if (!$this->conn) {
            $this->conn = new \AMQPConnection($this->config);
            if (!$this->conn->connect()) {
                throw new Exception('rabbitmq服务连接失败');
            }
        }
        return $this->conn;
    }

    //交换机
    public function getExChannel()
    {
        if (!$this->exchange) {
            //创建channel
            $this->channel = new AMQPChannel($this->connect());

            //创建交换机对象
            $this->exchange = new AMQPExchange($this->channel);

            //设置交换机名
            $this->exchange->setName($this->exchange_name);
        }
        return $this->exchange;
    }

    /**
     * 发送消息
     */
    public function publish($msg_str)
    {
        if(is_array($msg_str)) $msg_str = json_encode($msg_str,JSON_UNESCAPED_UNICODE);
        $this->getExChannel()->publish($msg_str,$this->routing_key);
    }

    //释放rabbmq连接
    public function __destruct()
    {
        $this->conn->disconnect();
    }
}

$publisher = new RabbitPublisher();
//发送消息到队列中
$msg = ['id'=>'自定义ID,例如：订单id','a'=>'其他内容字段'];
$publisher->publish($msg);

```

生产端代码也很简单，就是连接 rabbitmq 之后发送一个消息即可。

注意了，上述代码中我是用的交换机是直连的方式【**amq.direct**】，这个交换机是系统默认的。这个需要根据我们自己的需求进行修改。

路由 key 是队列用于匹配的规则，队列会根据自己的路由 key 规则获取相关消息列表。其实就是告诉交换机当前消息应该发送给哪些队列的作用。

如有不对的地方，大家可以在工作号留言或者加入我们的技术交流群