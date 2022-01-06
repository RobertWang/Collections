> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [yafeilee.com](https://yafeilee.com/blogs/100)

> ShowMeBug 是一款远程面试工具，双方可通过在线面试板进行实时沟通技术。所以关键技术要点在于 “实时同步”。关于实时同步，ShowMeBug 采用了以下技术。 OT 转换算法

ShowMeBug 核心技术内幕
----------------

rails

ShowMeBug 是一款远程面试工具，双方可通过在线面试板进行实时沟通技术。所以关键技术要点在于 “实时同步”。关于实时同步，ShowMeBug 采用了以下技术。

OT 转换算法
-------

本质上，ShowMeBug 核心就是多人同时在线实时编辑，难点即在这里。因为网络原因，操作可能是异步到达，丢失，与他人操作冲突。想想这就是个复杂的问题。

经过研究，最好用户体验的方式是 OT 转换算法。此算法由 1989 年 C. Ellis 和 S. Gibbs 首次提出，目前像 quip，google docs 均用的此法。

OT 算法允许用户自由编辑任意行，包括冲突的操作也可以很好支持，不用锁定。它的核心算法如下：

文档的操作统一为以下三种类型的操作（ Operation ）：

1.  `retain(n)`: 保持 n 个字符
2.  `insert(s)`: 插入字符串 s
3.  `delete(s)`: 删除字符串 s

然后客户端与服务端各记录历史版本，每次操作都经过一定的转换后，推送给另一端。

转换的核心是

S(o_1, o_2) = S(o_2, o_1)

换言之，把正在并发的操作进行转换合并，形成新的操作，然后应用在历史版本上，就可以实现无锁化同步编辑。

下图演示了对应的操作转换过程。

![](https://daotestimg.dao42.com/ipic/070918.jpg)

这个算法的难点在于分布式的实现。客户端服务端均需要记录历史，并且保持一定的序列。还要进行转换算法处理。

OT Rails 侧的处理
-------------

本质上，这是一个基于 websocket 的算法应用。所以我们没有怀疑就选用 ActionCable 作为它的基础。想着应该可以为我们节省大量的时间。实际上，我们错了。

ActionCable 实际上与 NodeJS 版本的 socket.io 一样，不具备任何可靠性的保障，做一些玩意性的聊天工具还可以，或者做消息通知允许丢失甚至重复推送的弱场景是可以的。但像 OT 算法这种强要求的就不可行了。

因为网络传输的不可靠性，我们必须按次序处理每一个操作。所以首先，我们实现了一个互斥锁，也就是针对某一个面试板，准备一个锁，同时只有一个操作可以进行操作。锁采用了 Redis 锁。实现如下：

```
def unlock_pad_history(lock_key)
    logger.debug "[padable] unlock( lock_key: #{lock_key} )..."
    old_lock_key = REDIS.get(_pad_lock_history_key)
    if old_lock_key == lock_key
      REDIS.del(_pad_lock_history_key)
    else
      log = "[FIXME] unlock_pad_history expired: lock_key=#{lock_key}, old_lock_key=#{old_lock_key}"
      logger.error(log)
      e = RuntimeError.new(log)
      ExceptionNotifier.notify_exception(e, lock_key: lock_key, old_lock_key: old_lock_key)
    end
  end

  # 为防止死锁，锁的时间为5分钟，超时自动解锁，但在 unlock 时会发出异常
  def lock_pad_history(lock_key)
    return REDIS.set(_pad_lock_history_key, lock_key, nx: true, ex: 5*60)
  end

  def wait_and_lock_pad_history(lock_key, retry_times = 200)
    total_retry_times = retry_times
    while !lock_pad_history(lock_key)
      sleep(0.05)
      logger.debug '[padable] locked, waiting 50ms...'
      retry_times-=1
      raise "wait_and_lock_pad_history(in #{total_retry_times*0.1}s) #{lock_key} failed" if retry_times == 0
    end
    logger.debug "[padable] locking it(lock_key: #{lock_key})..."
  end
```

服务端的并发控制完毕后，客户端通过 “状态队列” 技术一个个排队发布操作记录，核心如下：

```
class PadChannelSynchronized {
  sendHistory(channel, history){
    channel._sendHistory(history)
    return new PadChannelAwaitingConfirm(history)
  }
}

class PadChannelAwaitingConfirm {
  constructor(outstanding_history) {
    this.outstanding_history = outstanding_history
  }

  sendHistory(channel, history){
    return new PadChannelAwaitingWithHistory(this.outstanding_history, history)
  }

  receiveHistory(channel, history){
    return new PadChannelAwaitingConfirm(pair_history[0])
  }

  confirmHistory(channel, history) {
    if(this.outstanding_history.client_id !== history.client_id){
      throw new Error('confirmHistory error: client_id not equal')
    }
    return padChannelSynchronized
  }
}

class PadChannelAwaitingWithHistory {
  sendHistory(channel, history){
    let newHistory = composeHistory(this.buffer_history, history)
    return new PadChannelAwaitingWithHistory(this.outstanding_history, newHistory)
  }
}

let padChannelSynchronized = new PadChannelSynchronized()

export default padChannelSynchronized
```

以上，便实现了一个排队发送的场景。

除此之外，我们设计了一个 PadChannel 用来专门管理与服务器通信的事件，维护历史的状态，处理断线重传，操作转换与校验。

定义自己的历史（history) 协议
-------------------

解决了编辑器协同的问题，才是真正的问题的开始。每次的 ” 代码运行”，“编辑”，“清空终端”，“首次同步” 都是需要记录的历史操作。于是，ShowMeBug 定义了以下协议：

```
# 包含以下: edit( 更新编辑器内容 ), run( 执行命令 ), clear( 清空终端 ), sync( 同步数据 )
  # select( 光标 ), locate( 定位 )
  # history 格式如下:
  #
  # {
  # op: 'run' | 'edit' | 'select' | 'locate' | 'clear'
  # id: id // 全局唯一操作自增id, 首次前端传入时为 null, 服务端进行填充, 如果返回时为空, 则说明此 history 被拒绝写入
  # version: 'v1' // 数据格式版本
  # prev_id: prev_id // JS端生成 history 时上一次收到服务端的 id, 用于识别操作序列
  # client_id: client_id // 客户端生成的 history 的唯一标识
  # creator_id: creator_id // 操作人的用户id, 为了安全首次前端传入时为 null，由中台填充
  # event: { // op 为 edit 时, 记录编辑器 OT 转化后的数据, see here: https://github.com/Aaaaash/blog/issues/10
  #   [length, "string", length]
  #          // op 为 select 时, 记录编辑器选择区域(包括光标)
  #   }
  # snapshot: {
  #   editor_text: '' // 记录当前编辑器内容快照, 此快照由服务端填充
  #   language_type: '' // 记录当前编辑器的语言种类
  #   terminal_text: '' // 记录当前终端快照
  #   }
  # }
  # created_at: created_at // 生成时间
```

值得说明的是，`client_id` 是客户端生成的一个 8 位随机码，用于去重和与客户端进行 ACK 确认。

`id` 是由服务端 Redis 生成的自增 id，客户端会根据这个判断历史是否是新的。`prev_id` 用来操作转换时记录所需要进行转换操作的历史队列。

`event` 是最重要的操作记录，我们用 OT 的转换数据进行存储，如: `[length, "string", length]`

通过上述的设计，我们将面试板的所有操作细节涵盖了，从而实现多人面试实时同步，面试题和面试语言自动同步，操作回放等核心功能。

总结
--

篇幅限制，这里只讲到 ShowMeBug 的核心技术，更多的细节我们以后继续分享。

ShowMeBug 目前承载了 3000 场面试记录，成功支撑大量的实际面试官的面试，可靠性已得到进一步保障。这里面有两种重要编程范式值得考虑：

1.  如何在不可信链路上设计一种有序可靠的交付协议，定义清晰的协议数据和处理好异步事件是关键。
2.  如何平衡研发效率与稳定性之间的关系，比如实现的忙等锁，允许一定原因的失败，但处理好用户的提示与重试。既高效完成了功能，又不影响到用户体验。

ShowMeBug( [https://www.showmebug.com](https://www.showmebug.com/) ) 让你的技术面试更高效，助你找到你想要的候选人。

发表于 2019.10.26

© 自由转载 - 非商用 - 非衍生 - 保持署名

* * *