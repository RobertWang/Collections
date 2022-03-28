> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1648462701&ver=3704&signature=tq4*lThWPp1WiRDD-ue5ppMT-rlgKcL1VoagZSnG0ZrLvmUKs1rkCa8RqrEcIUItL6aV3z0yl6t3xmY-c4xbUa0djmqFxFso7ZUwRyYY7k*X5qP7Ca84-43u45z9vu0p&new=1)

MQTT 是一种基于发布 / 订阅（publish/subscribe）模式的 "轻量级" 通讯协议，作为一种低开销、低带宽占用的即时通讯协议，已经成为物联网的重要组成部分

Swoole 也给 PHP 提供了开发物联网项目的能力，只需要设置一个 open_mqtt_protocol[1] 选项，启用后就会解析 MQTT 包头，在 Worker 进程的 onReceive 事件每次都会返回一个完整的 MQTT 数据包

当然其他的也有，例如 Workerman 之前提供的 异步 mqtt 客户端库 [2] ，还有其他的开源库，这里就不一一介绍了

Simps 的第一个版本 MQTT 库 [3] 就是参考了 Workerman 的实现，使其能够使用 Swoole 的协程能力，同时也修复了一些问题

在此也要感谢 @walkor[4] 对 PHP 生态作出的贡献

第一个版本的实现是放在了框架当中，限制了一些用户的使用。于是又开始了重构，将 MQTT 独立为一个 library[5] ，方便用户使用的同时也丰富了 PHP 生态，让 PHP 程序员不再局限于 Web 开发

在第一个版本发布之后，Simps 的交流群中也有不少用户询问 MQTT 的问题，Swoole 也修复了一些相关的 Bug，现在使用 PHP + Swoole 去开发物联网相关的项目应该是如虎添翼

同时第一个版本的 MQTT 库，只支持 MQTT 3.x，不支持 MQTT 5.0，在 GitHub 上也没有找到相关支持的类库，所以在重构了 3.x 版本之后，也支持了一下 MQTT 5.0[6]

> 也许这是第一个支持 MQTT v5.0 协议的 PHP library...

支持 MQTT 协议 3.1、3.1.1 和 5.0 版本，支持 QoS 0、QoS 1、QoS 2，那么它来了，使用 composer 来安装

```
composer require simps/mqtt


```

安装成功之后我们来看一下订阅和发布的使用，以 MQTT5.0 为例

订阅
--

首先应该是订阅，订阅成功之后才能收到对应主题的发布消息，创建一个`subscribe.php`写入以下内容

```
include __DIR__ . '/vendor/autoload.php';

use Simps\MQTT\Hex\ReasonCode;
use Swoole\Coroutine;
use Simps\MQTT\Client;
use Simps\MQTT\Types;

$config = [
    'host' => 'broker.emqx.io',
    'port' => 1883,
    'time_out' => 5,
    'user_name' => 'user001',
    'password' => 'hLXQ9ubnZGzkzf',
    'client_id' => Client::genClientID(),
    'keep_alive' => 10,
    'properties' => [
        'session_expiry_interval' => 60,
        'receive_maximum' => 200,
        'topic_alias_maximum' => 200,
    ],
    'protocol_level' => 5,
];

Coroutine\run(function () use ($config) {
    $client = new Client($config, ['open_mqtt_protocol' => true, 'package_max_length' => 2 * 1024 * 1024]);
    while (!$data = $client->connect()) {
        Coroutine::sleep(3);
        $client->connect();
    }
    $topics['simps-mqtt/user001/get'] = [
        'qos' => 1,
        'no_local' => true,
        'retain_as_published' => true,
        'retain_handling' => 2,
    ];
    $timeSincePing = time();
    $res = $client->subscribe($topics);
    // 订阅的结果
    var_dump($res);
    while (true) {
        $buffer = $client->recv();
        if ($buffer && $buffer !== true) {
            $timeSincePing = time();
            // 收到的数据包
            var_dump($buffer);
        }
        if (isset($config['keep_alive']) && $timeSincePing < (time() - $config['keep_alive'])) {
            $buffer = $client->ping();
            if ($buffer) {
                echo 'send ping success' . PHP_EOL;
                $timeSincePing = time();
            } else {
                $client->close();
                break;
            }
        }
        // QoS1 发布回复
        if ($buffer['type'] === Types::PUBLISH && $buffer['qos'] === 1) {
            $client->send(
                [
                    'type' => Types::PUBACK,
                    'message_id' => $buffer['message_id'],
                    'code' => ReasonCode::SUCCESS
                ]
            );
        }
    }
});


```

执行`php subscribe.php`，就会得到这样的输出

```
array(3) {
  ["type"]=>
  int(9)
  ["message_id"]=>
  int(1)
  ["codes"]=>
  array(1) {
    [0]=>
    int(1)
  }
}


```

表示订阅成功，codes 对应的是对应订阅主题的 QoS 等级

发布
--

订阅成功之后，创建一个`publish.php`来测试发布

```
include __DIR__ . '/vendor/autoload.php';

use Swoole\Coroutine;
use Simps\MQTT\Client;

$config = [
    'host' => 'broker.emqx.io',
    'port' => 1883,
    'time_out' => 5,
    'user_name' => 'user002',
    'password' => 'adIJS1D482sd',
    'client_id' => Client::genClientID(),
    'keep_alive' => 20,
    'properties' => [
        'session_expiry_interval' => 60,
        'receive_maximum' => 200,
        'topic_alias_maximum' => 200,
    ],
    'protocol_level' => 5,
];

Coroutine\run(function () use ($config) {
    $client = new Client($config, ['open_mqtt_protocol' => true, 'package_max_length' => 2 * 1024 * 1024]);
    while (!$client->connect()) {
        Coroutine::sleep(3);
        $client->connect();
    }
    while (true) {
        $response = $client->publish(
            'simps-mqtt/user001/get',
            '{"time":' . time() . '}',
            1,
            0,
            0,
            ['topic_alias' => 1]
        );
        var_dump($response);
        Coroutine::sleep(3);
    }
});


```

代码的意思是每隔 3 秒给订阅的主题`simps-mqtt/user001/get`发布一次消息

打开一个新的终端窗口，执行`php publish.php`就会得到输出：

```
array(4) {
  ["type"]=>
  int(4)
  ["message_id"]=>
  int(1)
  ["code"]=>
  int(0)
  ["message"]=>
  string(7) "Success"
}


```

这里增加了 message，为了用户可读，不需要去查找对应的 code 含义是什么

返回到订阅的窗口，就会看到所打印的发布信息

```
array(8) {
  ["type"]=>
  int(3)
  ["topic"]=>
  string(0) ""
  ["message"]=>
  string(19) "{"time":1608017156}"
  ["dup"]=>
  int(1)
  ["qos"]=>
  int(1)
  ["retain"]=>
  int(0)
  ["message_id"]=>
  int(4)
  ["properties"]=>
  array(1) {
    ["topic_alias"]=>
    int(1)
  }
}


```

这样一个简单的发布订阅功能就实现了

在这个库中还有一些值得优化和还未完成的部分，如还没有支持 MQTT5 的`Auth` type，以及部分的`properties`还未支持

想参与的同学可以提交 PR，如果有问题也可以提交 Issue，让我们共同去建设 PHP 的生态

仓库地址：simps/mqtt[7] ，支持记得点个 Star 哦

### 参考资料

[1]

open_mqtt_protocol: _https://wiki.swoole.com/#/server/setting?id=open_mqtt_protocol_

[2]

异步 mqtt 客户端库: _https://github.com/walkor/mqtt_

[3]

MQTT 库: _https://github.com/simple-swoole/simps/blob/master/src/Server/Protocol/MQTT.php_

[4]

@walkor: _https://github.com/walkor_

[5]

library: _https://github.com/simps/mqtt_

[6]

MQTT 5.0: _https://github.com/simps/mqtt/pull/5_

[7]

simps/mqtt: _https://github.com/simps/mqtt_