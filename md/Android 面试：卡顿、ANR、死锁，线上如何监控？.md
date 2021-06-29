> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555676&idx=2&sn=3dc8123a357b61a0aee9a64a615becf5&chksm=80dcadb7b7ab24a152ccaac0945c9facd37bb2eb8dcb043a149a6d124ed90a6ac70dfce3107e&mpshare=1&scene=1&srcid=0629UviXITlSJwjODcqwhiRt&sharer_sharetime=1624940096375&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

↓推荐关注↓

一、前言
----

最近参加了几轮面试，发现很多 5-7 年工作经验的候选人在性能优化这一块，基本上只能说出传统的分析方式，**例如 ANR 分析，是通过查看 / data/anr/ 下的 log，分析主线程堆栈、cpu、锁信息等**，

然而，这种方法有一定的局限性，并不是每次都奏效，很多时候是没有堆栈信息给你分析的，例如有些高版本设备需要 root 权限才能访问 / data/anr/ 目录，或者是线上用户的反馈，只有一张 ANR 的截图加上一句话描述。

假如你的 App 没有实现 ANR 监控上报，那么你大概率会把这个问题当成 “未复现” 处理掉，而没有真正解决问题。

于是我整理了这一篇文章，主要关于卡顿、ANR、死锁监控方案。

二、卡顿原理和监控
---------

### 2.1 卡顿原理

一般来说，主线程有耗时操作会导致卡顿，卡顿超过阈值，触发 ANR。

从源码层面一步步分析卡顿原理：

首先应用进程启动的时候，`Zygote`会反射调用 `ActivityThread` 的 main 方法，启动 loop 循环

```
->ActivityThread

public static void main(String[] args) {
      ...
 Looper.prepareMainLooper();
 Looper.loop();
 ...
}


```

看下 Looper 的 loop 方法

```
->Looper

public static void loop() {
      for (;;) {
            //1、取消息
            Message msg = queue.next(); // might block
            ...
            //2、消息处理前回调
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            ...
            
            //3、消息开始处理
            msg.target.dispatchMessage(msg);// 分发处理消息
            ...
            
            //4、消息处理完回调
            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }
       }
       ...
}



```

由于 loop 循环存在，所以主线程可以长时间运行。如果想要在主线程执行某个任务，唯一的办法就是通过主线程 Handler post 一个任务到消息队列里去，然后 loop 循环中拿到这个 msg，交给这个 msg 的 target 处理，这个 target 是 Handler。

从上面的代码块可以看出，导致卡顿的原因可能有两个地方

*   注释 1 的`queue.next()`阻塞，
    
*   注释 3 的`dispatchMessage`耗时太久。
    

#### 2.1.1 MessageQueue#next 耗时

看下源码

MessageQueue#next

```
    Message next() {
        for (;;) {
            //1、nextPollTimeoutMillis 不为0则阻塞
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                // 2、先判断当前第一条消息是不是同步屏障消息，
                if (msg != null && msg.target == null) {
                    //3、遇到同步屏障消息，就跳过去取后面的异步消息来处理，同步消息相当于被设立了屏障
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                
                //4、正常的消息处理，判断是否有延时
                if (msg != null) {
                    if (now < msg.when) {
                        //3.1 
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    //5、如果没有取到异步消息，那么下次循环就走到1那里去了，nativePollOnce为-1，会一直阻塞
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }
    }


```

`next`方法的大致流程是这样的：

1.  MessageQueue 是一个链表数据结构，判断 MessageQueue 的头部（第一个消息）是不是一个同步屏障消息，所谓同步屏障消息，就是给同步消息加一层屏障，让同步消息不被处理，只会处理异步消息；
    
2.  如果遇到同步屏障消息，就会跳过 MessageQueue 中的同步消息，只获取里面的异步消息来处理。如果里面没有异步消息，那就会走到注释 5，nextPollTimeoutMillis 设置为 - 1，下次循环调用注释 1 的 nativePollOnce 就会阻塞；
    
3.  如果 looper 能正常获取到消息，不管是异步消息或者同步消息，处理流程都是一样的，在注释 4，先判断是否带延时，如果是，nextPollTimeoutMillis 就会被赋值，然后下次循环调用注释 1 的 nativePollOnce 就会阻塞一段时间。如果不是 delay 消息，就直接返回这个 msg，给 handler 处理；
    

从上面分析可以看出，`next`方法是不断从 MessageQueue 里取出消息，有消息就处理，没有消息就调用`nativePollOnce`阻塞，`nativePollOnce` 底层是 Linux 的 epoll 机制，这里涉及到一个 Linux IO 多路复用的知识点

##### Linux IO 多路复用，select、poll、epoll

Linux 上 IO 多路复用方案有 **select、poll、epol**l。它们三个中 epoll 的性能表现是最优秀的，能支持的并发量也最大。

1.  **select** 是操作系统提供的系统调用函数，通过它，我们可以把一个文件描述符的数组发给操作系统， 让操作系统去遍历，确定哪个文件描述符可以读写， 然后告诉我们去处理。
    
2.  **poll**：它和 select 的主要区别就是，去掉了 select 只能监听 1024 个文件描述符的限制
    
3.  **epoll**：epoll 主要就是针对 select 的这三个可优化点进行了改进
    

> 1、内核中保存一份文件描述符集合，无需用户每次都重新传入，只需告诉内核修改的部分即可。2、内核不再通过轮询的方式找到就绪的文件描述符，而是通过异步 IO 事件唤醒。3、内核仅会将有 IO 事件的文件描述符返回给用户，用户也无需遍历整个文件描述符集合。

回到 `MessageQueue`的`next` 方法，看看哪里可能阻塞

##### 同步屏障消息没移除导致 next 一直阻塞

有一种情况，在存在同步屏障消息的情况下，当异步消息被处理完之后，如果没有及时把同步屏障消息移除，会导致同步消息一直没有机会处理，一直阻塞在 **nativePollOnce**。

##### 同步屏障消息

Android 是禁止 App 往 MessageQueue 插入同步屏障消息的，代码会报错

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIibJU185m7qUZlKFich2psibJIRHic3SED0lCfYt3cDR5aeMdRc4cp3Gm43g/640?wx_fmt=png)

系统一些高优先级的操作会使用到同步屏障消息，例如 View 在绘制的时候，最终都要调用`ViewRootImpl`的`scheduleTraversals`方法，会往`MessageQueue`插入同步屏障消息，绘制完成后会移除同步屏障消息。

```
->ViewRootImpl

    @UnsupportedAppUsage
    void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            //插入同步屏障消息
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            if (!mUnbufferedInputDispatch) {
                scheduleConsumeBatchedInput();
            }
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }

    void unscheduleTraversals() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            //移除同步屏障消息
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
            mChoreographer.removeCallbacks(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        }
    }



```

为了保证 View 的绘制过程不被主线程其它任务影响，View 在绘制之前会先往 MessageQueue 插入同步屏障消息，然后再注册 Vsync 信号监听，`Choreographer$FrameDisplayEventReceiver`就是用来接收 vsync 信号回调的

**Choreographer$FrameDisplayEventReceiver**

```
    private final class FrameDisplayEventReceiver extends DisplayEventReceiver
            implements Runnable {
        ...
        @Override
        public void onVsync(long timestampNanos, long physicalDisplayId, int frame) {
           ...
            //
            mTimestampNanos = timestampNanos;
            mFrame = frame;
            Message msg = Message.obtain(mHandler, this);
            //1、发送异步消息
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtils.NANOS_PER_MS);
        }

        @Override
        public void run() {
            // 2、doFrame优先执行
            doFrame(mTimestampNanos, mFrame);
        }
    }


```

收到 Vsync 信号回调，注释 1 会往主线程`MessageQueue` post 一个异步消息，保证注释 2 的`doFrame`优先执行。

`doFrame`才是 View 真正开始绘制的地方，会调用`ViewRootImpl`的`doTraversal`、`performTraversals`，

而`performTraversals`里面会调用我们熟悉的 View 的`onMeasure`、`onLayout`、`onDraw`。

_这里还可以延伸到 vsync 信号原理，以及为什么要等 vsync 信号回调才开始 View 的绘制流程、掉帧的原理、屏幕的双缓冲、三缓冲，由于文章篇幅关系，不是本文的重点，就不一一分析了~_

虽然 app 无法发送同步屏障消息，但是使用异步消息是允许的

##### 异步消息

首先，SDK 中限制了 App 不能 post 异步消息到 MessageQueue 里去的，相关字段被加了`UnsupportedAppUsage`注解

```
-> Message

    @UnsupportedAppUsage
    /*package*/ int flags;
    
    /**
     * Returns true if the message is asynchronous, meaning that it is not
     * subject to {@link Looper} synchronization barriers.
     *
     * @return True if the message is asynchronous.
     *
     * @see #setAsynchronous(boolean)
     */
    public boolean isAsynchronous() {
        return (flags & FLAG_ASYNCHRONOUS) != 0;
    }


```

不过呢，高版本的 Handler 的构造方法可以通过传 async=true，来使用异步消息

```
public Handler(@Nullable Callback callback, boolean async) {}


```

然后在`Handler`发送消息的时候，都会走到 `enqueueMessage` 方法，如下代码块所示，每个消息都带了异步属性，有优先处理权

```
    private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,long uptimeMillis) {
        ...
        //如果mAsynchronous为true，就都设置为异步消息
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }


```

对于低版本 SDK，想要使用异步消息，可以通过反射调用`Handler(@Nullable Callback callback, boolean async)`，参考 androidx 内部的一段代码如下

```
->androidx.arch.core.executor.DefaultTaskExecutor

    private static Handler createAsync(@NonNull Looper looper) {
        if (Build.VERSION.SDK_INT >= 28) {
            return Handler.createAsync(looper);
        }
        if (Build.VERSION.SDK_INT >= 16) {
            try {
                return Handler.class.getDeclaredConstructor(Looper.class, Handler.Callback.class,
                        boolean.class)
                        .newInstance(looper, null, true);
            } catch (IllegalAccessException ignored) {
            } catch (InstantiationException ignored) {
            } catch (NoSuchMethodException ignored) {
            } catch (InvocationTargetException e) {
                return new Handler(looper);
            }
        }
        return new Handler(looper);
    }


```

需要注意的是，**App 要谨慎使用异步消息，使用不当的情况下可能会出现主线程假死的问题，排查也比较困难**

分析完`MessageQueue#next`再回头来看看 `Handler`的`dispatchMessage`方法

#### 2.1.2 dispatchMessage

上面说到 next 方法轮循取消息一般情况下是没有问题的，那么只剩下处理消息的逻辑

**Handler#dispatchMessage**

```
    /**
     * Handle system messages here.
     */
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }


```

dispatchMessage 有三个逻辑，分别对应 Handler 使用的三种方式

1.  Handler#post(Runnable r)
    
2.  构造方法传 CallBack，`public Handler(@Nullable Callback callback, boolean async) {}`
    
3.  Handler 重写 handleMessage 方法
    

**所以，应用卡顿，原因一般都可以认为是 Handler 处理消息太耗时导致的**，细分的原因可能是方法本身太耗时、算法效率低、cpu 被抢占、内存不足、IPC 超时等等。

### 2.2 卡顿监控

面试中，被问到如何监控 App 卡顿，统计方法耗时，我们可以从源码开始切入，讲讲如何通过`Looper`提供的`Printer`接口，计算`Handler`处理一个消息的耗时，判断是否出现卡顿。

#### 2.2.1 卡顿监控方案一

看下 Looper 循环的注释 2 和注释 4，可以找到一种卡顿监控的方法

##### Looper#loop

```
public static void loop() {
        for (;;) {
            //1、取消息
            Message msg = queue.next(); // might block
            ...
            //2、消息处理前回调
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            ...
            
            //3、消息开始处理
            msg.target.dispatchMessage(msg);// 分发处理消息
            ...
            
            //4、消息处理完回调
            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }
       }
       ...
}


```

注释 2 和注释 4 的`logging.println`是谷歌提供给我们的一个接口，可以监听 Handler 处理消息耗时，我们只需要调用`Looper.getMainLooper().setMessageLogging(printer)`，即可从回调中拿到 Handler 处理一个消息的前后时间。

需要注意的是，监听到发生卡顿之后，`dispatchMessage` 早已调用结束，已经出栈，此时再去获取主线程堆栈，堆栈中是不包含卡顿的代码的。

所以需要在后台开一个线程，定时获取主线程堆栈，**将时间点作为 key，堆栈信息作为 value，保存到 Map 中，在发生卡顿的时候，取出卡顿时间段内的堆栈信息即可。**

不过这种方案只适合线下使用，原因如下：

1.  `logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);`存在字符串拼接，频繁调用，会创建大量对象，造成内存抖动。
    
2.  后台线程频繁获取主线程堆栈，对性能有一定影响，**获取主线程堆栈，会暂停主线程的运行**。
    

#### 2.2.2 卡顿监控方案二

对于线上卡顿监控，需要了解**字节码插桩**技术。

通过 Gradle Plugin+ASM，编译期在每个方法开始和结束位置分别插入一行代码，统计方法耗时，

伪代码如下

```
插桩前
fun method(){
   run()
}

插桩后
fun method(){
   input(1)
   run()
   output(1)
}



```

目前微信的 Matrix 使用的卡顿监控方案就是字节码插桩，如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIibUvCtDWiafmnpCVM8uyhpYtRQzR6wOxcWLooicFAFPadNtCb7HATF8zlg/640?wx_fmt=png)

插桩需要注意的问题：

1.  **避免方法数暴增**：在方法的入口和出口应该插入相同的函数，在编译时提前给代码中每个方法分配一个独立的 ID 作为参数。
    
2.  **过滤简单的函数**：过滤一些类似直接 return、i++ 这样的简单函数，并且支持黑名单配置。对一些调用非常频繁的函数，需要添加到黑名单中来降低整个方案对性能的损耗。
    

微信 Matrix 做了大量优化，整体包体积增加 1%-2%，帧率下降 2 帧以内，对性能影响整体可以接受，不过依然只会在灰度包使用。

再来说说 ANR~

三、ANR 原理
--------

ANR 的类型和触发 ANR 的流程

### 3.1 哪些场景会造成 ANR 呢

*   Service Timeout: 比如前台服务在 20s 内未执行完成，后台服务是 10s；
    
*   BroadcastQueue Timeout：比如前台广播在 10s 内未执行完成，后台 60s
    
*   ContentProvider Timeout：内容提供者, 在 publish 过超时 10s;
    
*   InputDispatching Timeout: 输入事件分发超时 5s，包括按键和触摸事件。
    

相关超时定义可以参考`ActivityManagerService`

```
// How long we allow a receiver to run before giving up on it.
static final int BROADCAST_FG_TIMEOUT = 10*1000;
static final int BROADCAST_BG_TIMEOUT = 60*1000;

// How long we wait until we timeout on key dispatching.
static final int KEY_DISPATCHING_TIMEOUT = 5*1000;


```

### 3.2 ANR 触发流程

来简单分析下源码，ANR 触发流程其实可以比喻成**埋炸弹**和**拆炸弹**的过程，

**以后台 Service 为例**

#### 3.2.1 埋炸弹

`Context.startService`  
调用链如下：  
`AMS.startService`  
`ActiveServices.startService`  
`ActiveServices.realStartServiceLocked`

##### ActiveServices.realStartServiceLocked

```
private final void realStartServiceLocked(ServiceRecord r, ProcessRecord app, boolean execInFg) throws RemoteException {
    ...
    //1、这里会发送delay消息(SERVICE_TIMEOUT_MSG)
    bumpServiceExecutingLocked(r, execInFg, "create");
    try {
        ...
        //2、通知AMS创建服务
        app.thread.scheduleCreateService(r, r.serviceInfo,
                mAm.compatibilityInfoForPackageLocked(r.serviceInfo.applicationInfo),
                app.repProcState);
    } 
    ...
}


```

注释 1 的 bumpServiceExecutingLocked 内部调用 scheduleServiceTimeoutLocked

```
    void scheduleServiceTimeoutLocked(ProcessRecord proc) {
        ...
        Message msg = mAm.mHandler.obtainMessage(
                ActivityManagerService.SERVICE_TIMEOUT_MSG);
        msg.obj = proc;
        // 发送deley消息，前台服务是20s，后台服务是10s
        mAm.mHandler.sendMessageDelayed(msg,
                proc.execServicesFg ? SERVICE_TIMEOUT : SERVICE_BACKGROUND_TIMEOUT);
    }


```

注释 2 通知 AMS 启动服务之前，注释 1 处发送 Handler 延时消息，埋下炸弹，如果 10s 内（前台服务是 20s）没人来拆炸弹，炸弹就会爆炸，即`ActiveServices#serviceTimeout`方法会被调用

#### 3.2.2 拆炸弹

启动一个 Service，先要经过 AMS 管理，然后 AMS 会通知应用进程执行 Service 的生命周期， `ActivityThread`的`handleCreateService`方法会被调用

```
-> ActivityThread#handleCreateService

    private void handleCreateService(CreateServiceData data) {
        try {
           ...
            Application app = packageInfo.makeApplication(false, mInstrumentation);
            service.attach(context, this, data.info.name, data.token, app,
                    ActivityManager.getService());
             //1、service onCreate调用
            service.onCreate();
            mServices.put(data.token, service);
            try {
             //2、拆炸弹在这里
                ActivityManager.getService().serviceDoneExecuting(
                        data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
            } catch (RemoteException e) {
                throw e.rethrowFromSystemServer();
            }
        }
    
    }


```

注释 1，`Service`的`onCreate`方法被调用，  
注释 2，调用 AMS 的`serviceDoneExecuting`方法，最终会调用到`ActiveServices. serviceDoneExecutingLocked`

```
private void serviceDoneExecutingLocked(ServiceRecord r, boolean inDestroying,
              boolean finishing) {

...
 //移除delay消息
 mAm.mHandler.removeMessages(ActivityManagerService.SERVICE_TIMEOUT_MSG, r.app);
...
            
 }


```

可以看到，`onCreate`方法调用完之后，就会移除 delay 消息，炸弹被拆除。

#### 3.2.3 引爆炸弹

假设 Service 的 onCreate 执行超过 10s，那么炸弹就会引爆，也就是

`ActiveServices#serviceTimeout`方法会被调用

```
    void serviceTimeout(ProcessRecord proc) {
   
    ...
    if (anrMessage != null) {
            mAm.mAppErrors.appNotResponding(proc, null, null, false, anrMessage);
        }
    ...
    }


```

所有 ANR，最终都会调用`AppErrors`的`appNotResponding`方法

##### AppErrors #appNotResponding

```
    final void appNotResponding(ProcessRecord app, ActivityRecord activity,
            ActivityRecord parent, boolean aboveSystem, final String annotation) {
          ...
          
          //1、写入event log
          // Log the ANR to the event log.
          EventLog.writeEvent(EventLogTags.AM_ANR, app.userId, app.pid,
                    app.processName, app.info.flags, annotation);
           ...
          //2、收集需要的log，anr、cpu等，StringBuilder凭借
         // Log the ANR to the main log.
         StringBuilder info = new StringBuilder();
         info.setLength(0);
         info.append("ANR in ").append(app.processName);
         if (activity != null && activity.shortComponentName != null) {
             info.append(" (").append(activity.shortComponentName).append(")");
         }
         info.append("\n");
         info.append("PID: ").append(app.pid).append("\n");
         if (annotation != null) {
             info.append("Reason: ").append(annotation).append("\n");
         }
         if (parent != null && parent != activity) {
             info.append("Parent: ").append(parent.shortComponentName).append("\n");
         }
 
         ProcessCpuTracker processCpuTracker = new ProcessCpuTracker(true);

        ...
        // 3、dump堆栈信息，包括java堆栈和native堆栈，保存到文件中
        // For background ANRs, don't pass the ProcessCpuTracker to
        // avoid spending 1/2 second collecting stats to rank lastPids.
        File tracesFile = ActivityManagerService.dumpStackTraces(
                true, firstPids,
                (isSilentANR) ? null : processCpuTracker,
                (isSilentANR) ? null : lastPids,
                nativePids);

        String cpuInfo = null;
        ...

      //4、输出ANR 日志
        Slog.e(TAG, info.toString());
        if (tracesFile == null) {
             // 5、没有抓到tracesFile，发一个SIGNAL_QUIT信号
            // There is no trace file, so dump (only) the alleged culprit's threads to the log
            Process.sendSignal(app.pid, Process.SIGNAL_QUIT);
        }

        StatsLog.write(StatsLog.ANR_OCCURRED, ...)
        // 6、输出到drapbox
        mService.addErrorToDropBox("anr", app, app.processName, activity, parent, annotation, cpuInfo, tracesFile, null);

        ...

        synchronized (mService) {
            mService.mBatteryStatsService.noteProcessAnr(app.processName, app.uid);
           //7、后台ANR，直接杀进程
            if (isSilentANR) {
                app.kill("bg anr", true);
                return;
            }

           //8、错误报告
            // Set the app's notResponding state, and look up the errorReportReceiver
            makeAppNotRespondingLocked(app,
                    activity != null ? activity.shortComponentName : null,
                    annotation != null ? "ANR " + annotation : "ANR",
                    info.toString());

            //9、弹出ANR dialog，会调用handleShowAnrUi方法
            // Bring up the infamous App Not Responding dialog
            Message msg = Message.obtain();
            msg.what = ActivityManagerService.SHOW_NOT_RESPONDING_UI_MSG;
            msg.obj = new AppNotRespondingDialog.Data(app, activity, aboveSystem);

            mService.mUiHandler.sendMessage(msg);
        }
    }



```

主要流程如下：  
1、写入 event log  
2、写入 main log  
3、生成 tracesFile  
4、输出 ANR logcat（控制台可以看到）  
5、如果没有获取到 tracesFile，会发一个`SIGNAL_QUIT`信号，这里看注释是会触发收集线程堆栈信息流程，写入 traceFile  
6、输出到 drapbox  
7、后台 ANR，直接杀进程  
8、错误报告  
9、弹出 ANR dialog，会调用 `AppErrors#handleShowAnrUi`方法。

#### ANR 触发流程小结

> ANR 触发流程，可以比喻为埋炸弹和拆炸弹的过程，  
> 以启动 Service 为例，Service 的 onCreate 方法调用之前会使用 Handler 发送延时 10s 的消息，Service 的 onCreate 方法执行完，会把这个延时消息移除掉。  
> 假如 Service 的 onCreate 方法耗时超过 10s，延时消息就会被正常处理，也就是触发 ANR，会收集 cpu、堆栈等信息，弹 ANR Dialog。

service、broadcast、provider 的 ANR 原理都是**埋定时炸弹和拆炸弹**原理，

但是 input 的超时检测机制稍微有点不同，需要等收到下一次 input 事件，才会去检测上一次 input 事件是否超时，input 事件里埋的炸弹是普通炸弹，需要通过**扫雷**来排查。

四、ANR 分析方法
----------

上面已经分析了 ANR 触发流程，最终会把发生 ANR 时的线程堆栈、cpu 等信息保存起来，我们一般都是分析 **/data/anr/traces.txt** 文件

### 4.1 模拟死锁导致 ANR

```
    private fun testAnr(){

        val lock1 = Object()
        val lock2 = Object()
        
        //子线程持有锁1，想要竞争锁2
        thread {
            synchronized(lock1){
                Thread.sleep(100)

                synchronized(lock2){
                    Log.d(TAG, "testAnr: getLock2")
                }
            }
        }

        //主线程持有锁2，想要竞争锁1
        synchronized(lock2){
            Thread.sleep(100)

            synchronized(lock1){
                Log.d(TAG, "testAnr: getLock1")
            }
        }
    }


```

触发 ANR 之后，一般我们会拉取 anr 日志：**adb pull /data/traces.txt**(文件名可能是 anr_xxx.txt)

### 4.2 分析 ANR 文件

首先看主线程，搜索 main

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIibgoib7rB8iazqMHiaZhPxmOOFsamRxoe5SnLYsPozytnsXtme2aYuXeNMA/640?wx_fmt=png)

ANR 日志中有很多信息，可以看到，主线程 id 是 1（tid=1），在等待一个锁，这个锁一直被 id 为 22 的程持有，那么看下 22 号线程的堆栈

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIibrxwcRPoP7mMMh6HMBHofLoNXPOr9u3AQGpaU9pEtr6AMpNhotVqpOw/640?wx_fmt=png)

id 为 22 的线程是 Blocked 状态，正在等待一个锁，这个锁被 id 为 1 的线程持有，同时这个 22 号线程还持有一个锁，这个锁是主线程想要的。

通过 ANR 日志，可以很清楚分析出这个 ANR 是死锁导致的，并且有具体堆栈信息。

上面只是举例一种**死锁导致 ANR** 的情况，实际项目中，可能有很多情况会导致 ANR，例如**内存不足、CPU 被抢占、系统服务没有及时响应**等等。

_如果是线上问题，怎么样才能拿到 ANR 日志呢？_

五、ANR 监控
--------

前面已经分析了 ANR 触发流程，以及常规的线下分析方法，看起来还是有点繁琐的，需要 pull 出 anr 日志，然后分析线程堆栈等信息。对于线上 ANR，如何搭建一个完善的 ANR 监控系统呢？

下面将介绍 ANR 监控的方式

### 5.1 抓取系统 traces.txt 上传

1、当监控线程发现主线程卡死时，主动向系统发送 SIGNAL_QUIT 信号。  
2、等待 / data/anr/traces.txt 文件生成。  
3、文件生成以后进行上报。

看起来好像可行，不过有以下两个问题：  
1、traces.txt 里面包含所有线程的信息，上传之后需要人工过滤分析  
2、很多高版本系统需要 root 权限才能读取 /data/anr 这个目录

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIibSqqAQ4UaQeN2jSG1j3hTwHECUCmMEFn5q5TiaIaCUoz5w2EXQgsQZ4Q/640?wx_fmt=png)

既然这个方案存在问题，那么可还有其它办法？

### 5.2 ANRWatchDog

ANRWatchDog 是一个自动检测 ANR 的开源库

#### 5.2.1 ANRWatchDog 原理

其源码只有两个类，核心是`ANRWatchDog`这个类，继承自 Thread，它的 run 方法如下，看注释处

```
    public void run() {
        setName("|ANR-WatchDog|");

        long interval = _timeoutInterval;
       // 1、开启循环
        while (!isInterrupted()) {
            boolean needPost = _tick == 0;
            _tick += interval;
            if (needPost) {
               // 2、往UI线程post 一个Runnable，将_tick 赋值为0，将 _reported 赋值为false                      
              _uiHandler.post(_ticker);
            }

            try {
                // 3、线程睡眠5s
                Thread.sleep(interval);
            } catch (InterruptedException e) {
                _interruptionListener.onInterrupted(e);
                return ;
            }

            // If the main thread has not handled _ticker, it is blocked. ANR.
            // 4、线程睡眠5s之后，检查 _tick 和 _reported 标志，正常情况下_tick 已经被主线程改为0，_reported改为false，如果不是，说明 2 的主线程Runnable一直没有被执行，主线程卡住了
            if (_tick != 0 && !_reported) {
                ...
                if (_namePrefix != null) {
                    // 5、判断发生ANR了，那就获取堆栈信息，回调onAppNotResponding方法
                    error = ANRError.New(_tick, _namePrefix, _logThreadsWithoutStackTrace);
                } else {
                    error = ANRError.NewMainOnly(_tick);
                }
                _anrListener.onAppNotResponding(error);
                interval = _timeoutInterval;
                _reported = true;
            }

        }

    }


```

`ANRWatchDog` 的原理是比较简单的，概括为以下几个步骤

1.  开启一个线程，死循环，循环中睡眠 5s
    
2.  往 UI 线程 post 一个 Runnable，将_tick 赋值为 0，将 _reported 赋值为 false
    
3.  线程睡眠 5s 之后检查_tick 和_reported 字段是否被修改
    
4.  如果_tick 和_reported 没有被修改，说明给主线程 post 的 Runnable 一直没有被执行，也就说明主线程卡顿至少 5s**（只能说至少，这里存在 5s 内的误差）**。
    
5.  将线程堆栈信息输出
    

其中涉及到并发的一个知识点，关于 `volatile` 关键字的使用，面试中的常客， **`volatile`的特点是：保证可见性，禁止指令重排，适合在一个线程写，其它线程读的情况。**

面试中一般会展开问 JMM，工作内存，主内存等，以及为什么要有工作内存，能不能所有字段都用 `volatile` 关键字修饰等问题。

回到 ANRWatchDog 本身，细心的同学可能会发现一个问题，使用 ANRWatchDog 有时候会捕获不到 ANR，是什么原因呢？

#### 5.2.2 ANRWatchDog 缺点

ANRWatchDog 会出现漏检测的情况，看图

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIibxcnR1Z9lo4eEGBhlOI4KL2cqEVUHSibbictCiaHxuBK67QcACfJh54PVA/640?wx_fmt=png)

如上图这种情况，红色表示卡顿，

1.  假设主线程卡顿了 2s 之后，ANRWatchDog 这时候刚开始一轮循环，将_tick 赋值为 5，并往主线程 post 一个任务，把_tick 修改为 0
    
2.  主线程过了 3s 之后不卡顿了，将_tick 赋值为 0
    
3.  等到 ANRWatchDog 睡眠 5s 之后，发现_tick 的值是 0，判断为没有发生 ANR。而实际上，主线程中间是卡顿了 5s，ANRWatchDog 误差是在 5s 之内的（5s 是默认的，线程的睡眠时长）
    

针对这个问题，可以做一下优化。

### 5.3 ANRMonitor

ANRWatchDog 漏检测的问题，根本原因是因为线程睡眠 5s，不知道前一秒主线程是否已经出现卡顿了，**如果改成每间隔 1 秒检测一次，就可以把误差降低到 1s 内**。

接下来通过改造 ANRWatchDog ，来做一下优化，命名为 **ANRMonitor**。

我们想让子线程间隔 1s 执行一次任务，可以通过 `HandlerThread`来实现

流程如下：

核心的 Runnable 代码

```
    @Volatile
    var mainHandlerRunEnd = true
    
    //子线程会间隔1s调用一次这个Runnable
    private val mThreadRunnable = Runnable {
        
        blockTime++
        //1、标志位 mainHandlerRunEnd 没有被主线程修改，说明有卡顿
        if (!mainHandlerRunEnd && !isDebugger()) {
            logw(TAG, "mThreadRunnable: main thread may be block at least $blockTime s")
        }

        //2、卡顿超过5s，触发ANR流程，打印堆栈
        if (blockTime >= 5) {
            if (!mainHandlerRunEnd && !isDebugger() && !mHadReport) {
                mHadReport = true
                //5s了，主线程还没更新这个标志，ANR
                loge(TAG, "ANR->main thread may be block at least $blockTime s ")
                loge(TAG, getMainThreadStack())
                //todo 回调出去，这里可以按需把其它线程的堆栈也输出
                //todo debug环境可以开一个新进程，弹出堆栈信息
            }
        }

        //3、如果上一秒没有卡顿，那么重置标志位，然后让主线程去修改这个标志位
        if (mainHandlerRunEnd) {
            mainHandlerRunEnd = false
            mMainHandler.post {
             mainHandlerRunEnd = true
            }
            
        }

     //子线程间隔1s调用一次mThreadRunnable
        sendDelayThreadMessage()

    }



```

1.  子线程每隔 1s 会执行一次 mThreadRunnable，检测标志位 mainHandlerRunEnd 是否被修改
    
2.  假如 mainHandlerRunEnd 如期被主线程修改为 true，那么重置 mainHandlerRunEnd 标志位为 false，然后继续执行步骤 1
    
3.  假如 mainHandlerRunEnd 没有被修改 true，说明有卡顿，累计卡顿 5s 就触发 ANR 流程
    

在监控到 ANR 的时候，除了获取主线程堆栈，还有 cpu、内存占用等信息也是比较重要的，demo 中省略了这部分内容。

#### 5.3.1 测试 ANR

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIib9tEwnyw6iaF32rFucHhSOCicSsO13jXicQWbEOv9z2ZW1NRTWaY8ZrRyw/640?wx_fmt=png)

#### 5.3.2 ANR 检测结果

logcat 打印所示

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIib4hmkJbGVoDeEq68gI5dCyD4LoBtXStS5ias2BHGM8m0MDa7iaibIV4EqA/640?wx_fmt=png)

主线程卡顿超过 5s，会打堆栈信息，如果是卡顿 1-5s 内，会有 warning 的 log 提示，线下可以做成弹窗或者 toast 提示，

**看到这里，大家应该能想到，线下也可以用这种方法检测卡顿，定位到耗时的代码。**

_此方案可以结合`ProcessLifecycleOwner`，应用在前台才开启检测，进入后台则停止检测。_

六、死锁监控
------

在发生 ANR 的时候，有时候只有主线程堆栈信息可能还不够，例如发生死锁的情况，**需要知道当前线程在等待哪个锁，以及这个锁被哪个线程持有**，然后把发生死锁的线程堆栈信息都收集到。

流程如下：

1.  获取当前 blocked 状态的线程
    
2.  获取该线程想要竞争的锁
    
3.  获取该锁被哪个线程持有
    
4.  通过关系链，判断死锁的线程，输出堆栈信息
    

在 Java 层并没有相关 API 可以实现死锁监控，可以从 Native 层入手。

### 6.1 获取当前 blocked 状态的线程

这个比较简单，一个 for 循环就搞定，不过我们要的线程 id 是 native 层的线程 id，Thread 内部有一个 native 线程地址的字段叫 `nativePeer`，通过反射可以获取到。

```
        Thread[] threads = getAllThreads();
        for (Thread thread : threads) {
            if (thread.getState() == Thread.State.BLOCKED) {
                long threadAddress = (long) ReflectUtil.getField(thread, "nativePeer");
                // 找不到地址，或者线程已经挂了，此时获取到的可能是0和-1
                if (threadAddress <= 0) {
                    continue;
           }
          ...后续
            }
        }


```

有了 native 层线程地址，还需要找到 native 层相关函数

### 6.2 获取当前线程想要竞争的锁

从 ART 源码可以找到这个函数 androidxref.com/8.0.0_r4/xref/art/runtime/monitor.cc

函数：**Monitor::GetContendedMonitor**

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIibPczdud5UbpbOBKib0ANXCMn6MKXGErE1L1UZCl3YAocEUCmicWSia0kvg/640?wx_fmt=png)

从源码和源码的解释可以看出，这个函数是用来获取当前线程等待的 Monitor。

顺便说说 Monitor 以及 Java 对象结构

##### Monitor

**Monitor 是一种并发控制机制**，提供多线程环境下的互斥和同步，以支持安全的并发访问。

Monitor 由以下 3 个元素组成：

1.  临界区：例如 synchronize 修饰的代码块
    
2.  条件变量：用来维护因不满足条件而阻塞的线程队列
    
3.  Monitor 对象，维护 Monitor 的入口、临界区互斥量（即锁）、临界区和条件变量，以及条件变量上的阻塞和唤醒
    

##### Java 的 Class 对象

Java 的 Class 对象包括三部分组成：

1.  对象头：MarkWord 和对象指针
    
    > **MarkWord（标记字段）**：保存哈希码、分代年龄、**锁标志位**、偏向线程 ID、偏向时间戳等信息 **对象指针**：即指向当前对象的类的元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。
    
2.  实例数据：对象实际的数据
    
3.  对齐填充：按 8 字节对齐（JVM 自动内存管理系统要求对象起始地址必须是 8 字节的整数倍）。例如 Integer 对象，对象头 MarkWord 和对象指针分别占用 4 字节，实例数据 4 字节，那么对齐填充就是 4 字节，Integer 占用内存是 int 的 4 倍。
    

回到 `GetContendedMonitor` 函数，我们可以通过打开动态库`libart.so`，然后使用`dlsym`获取函数的符号地址，然后就可以进行调用了。

由于 Android 7.0 开始，系统限制 App 中调用`dlopen`，`dlsym`等函数打开系统动态库，我们可以使用 ndk_dlopen 这个库来绕过这个限制

```
    //1、初始化
    ndk_init(env);

    //2、打开动态库libart.so
    void *so_addr = ndk_dlopen("libart.so", RTLD_NOLOAD);
    if (so_addr == NULL) {
        return 1;
    }
    


```

打开动态库之后，会返回动态库的内存地址，接下来就可以通过`dlsym`获取`GetContendedMonitor`这个函数的符号地址，只不过要注意，c++ 可以重载，所以它的函数符号比较特殊，需要从`libart.so`中搜索匹配找到

```
    //c++跟c不一样，c++可以重载，描述符会变，需要打开libart.so，在里面搜索查找GetContendedMonitor的函数符号
    //http://androidxref.com/8.0.0_r4/xref/system/core/libbacktrace/testdata/arm/libart.so

    //获取Monitor::GetContendedMonitor函数符号地址
    get_contended_monitor = ndk_dlsym(so_addr, "_ZN3art7Monitor19GetContendedMonitorEPNS_6ThreadE");
    if (get_contended_monitor == NULL) {
        return 2;
    }


```

到此，第一个函数的符号地址找到了，接下来要找另外一个函数

### 6.3 获取目标锁被哪个线程持有

**函数：Monitor::GetLockOwnerThreadId**

![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIibDIicNYYrNNXakHMLbN5Yf567RaRyv1sBXAcibIBOTYmbZ9yM1xxMW64Q/640?wx_fmt=png)

用同样的方式来获取这个函数符号地址

```
    
    // Monitor::GetLockOwnerThreadId
    //这个函数是用来获取 Monitor的持有者,会返回线程id
    get_lock_owner_thread = ndk_dlsym(so_addr, get_lock_owner_symbol_name(api_level));
    if (get_lock_owner_thread == NULL) {
        return 3;
    }
    


```

由于从 android 10 开始，这个`GetLockOwnerThreadId`函数符号有变化，所以需要通过 api 版本来判断使用哪一个

```
const char *get_lock_owner_symbol_name(jint level) {
    if (level <= 29) {
        //android 9.0 之前
        //http://androidxref.com/9.0.0_r3/xref/system/core/libbacktrace/testdata/arm/libart.so 搜索 GetLockOwnerThreadId
        return "_ZN3art7Monitor20GetLockOwnerThreadIdEPNS_6mirror6ObjectE";
    } else {
        //android 10.0
        // todo 10.0 源码中这个函数符号变了，需要自行查阅
        return "_ZN3art7Monitor20GetLockOwnerThreadIdEPNS_6mirror6ObjectE";
    }
}


```

到此，就得到了两个函数符号地址，接下来就把 blocked 状态的 native 线程 id 传过去，调用就行了

### 6.4 找到一直不释放锁的线程

```
Java_com_lanshifu_demo_anrmonitor_DeadLockMonitor_getContentThreadIdArt(JNIEnv *env,jobject thiz,jlong native_thread) {

    LOGI("getContentThreadIdArt");
    int monitor_thread_id = 0;
    if (get_contended_monitor != NULL && get_lock_owner_thread != NULL) {
        LOGI("get_contended_monitor != NULL");
        //1、调用一下获取monitor的函数，返回当前线程想要竞争的monitor
        int monitorObj = ((int (*)(long)) get_contended_monitor)(native_thread);
        if (monitorObj != 0) {
            LOGI("monitorObj != 0");
            // 2、获取这个monitor被哪个线程持有，返回该线程id
            monitor_thread_id = ((int (*)(int)) get_lock_owner_thread)(monitorObj);
        } else {
            LOGE("GetContendedMonitor return null");
            monitor_thread_id = 0;
        }
    } else {
        LOGE("get_contended_monitor == NULL || get_lock_owner_thread == NULL");

    }
    return monitor_thread_id;
}



```

两个步骤：

1.  获取当前线程要竞争的锁
    
2.  获取这个锁被哪个线程持有
    

通过两个步骤，得到的是那个一直不释放锁的线程 id。

### 6.5 通过算法，找到死锁

前面已经知道当前 blocked 状态的线程 id（还需要转换成 native 线程 id），以及这个 blocked 线程在等待哪个线程释放锁，也就是得到关系链：

1.  A 等待 B B 等待 A
    
2.  A 等待 B B 等待 C C 等待 A ...
    
3.  其它...
    

如何判断有死锁？我们可以用 Map 来保存对应关系

map[A]=B

map[B]=A

最后通过互斥条件判断出死锁线程，把造成死锁的线程堆栈信息输出，如下

### ![](https://mmbiz.qpic.cn/mmbiz_png/FoiciaVBBCfia5KthCgOabku2ZiaPg1GTjIibotgFcHMFyUY2oEiajPdPsO5icSXvnauEJVFpEXRtWcnFCrgIveOqPianQ/640?wx_fmt=png)

检查出死锁，线下可以弹窗或者 toast，线上则可以采集数据上报。

### 6.6 死锁监控小结

死锁监控原理还是比较清晰的：

1.  获取 blocked 状态的线程
    
2.  获取该线程想要竞争的锁（native 层函数）
    
3.  获取这个锁被哪个线程持有（native 层函数）
    
4.  有了关系链，就可以找出造成死锁的线程
    

由于死锁监控涉及到 native 层代码，对于很多应用层开发的同学来说可能有点难度，

**但是正因为有难度，我们去了解，去学习，并且掌握了，才能在众多竞争者中脱颖而出。**

七、形成闭环
------

前面分开讲了卡顿监控、ANR 监控和死锁监控，我们可以把它连接起来，在发生 ANR 的时候，将整个监控流程形成一个闭环

1.  发生 ANR
    
2.  获取主线程堆栈信息
    
3.  检测死锁
    
4.  获取死锁对应线程堆栈信息
    
5.  上报到服务器
    
6.  结合 git，定位到最后修改代码的同学，给他提一个线上问题单
    

八、总结
----

这篇文章从源码层面分析了卡顿、ANR，以及死锁监控，平时开发中，大部分同学可能都是做业务需求为主，对于 ANR 问题，可能不太注重，或者直接依赖第三方，例如 Bugly，但是呢，在面试中，面试官基本不太会问你这些工具的使用，要问也是从原理层面问。

本文以卡顿作为切入点

1.  讲解卡顿原理以及卡顿监控的方式；
    
2.  引申了 Handler 机制、Linux 的 epoll 机制
    
3.  分析 ANR 触发流程，可以比喻为埋炸弹和拆炸弹过程
    
4.  ANR 常规分析方案，/data/anr/traces.txt，
    
5.  ANRWatchDog 方案
    
6.  ANRWatchDog 存在问题，进行优化
    
7.  死锁导致的 ANR，死锁监控
    
8.  形成闭环
    

有问题可以在评论区留言探讨，期待与大家共同进步。

本文源码:

ANRMonitorDemo  
https://github.com/lanshifu/ANRMonitorDemo

参考文章：

线程监控 - 死锁、存活周期与 CPU 占用率

《手 Q Android 线程死锁监控与自动化分析实践》

说一说管程（Monitor）及其在 Java synchronized 机制中的体现

图解 | 深入揭秘 epoll 是如何实现 IO 多路复用的

理解 Android ANR 的触发原理

ANRWatchDog

> 转自：掘金 蓝师傅
> 
> https://juejin.cn/post/6973564044351373326

推荐阅读  点击标题可跳转

1、[Google 和 Facebook 为什么不用 Docker？](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555545&idx=3&sn=9c9c7bcaf19b60cf382e8618a5b17edd&chksm=80dcac32b7ab2524acebee9c868681b7c4a8395362e14350a6991732af88f845a370230b0389&scene=21#wechat_redirect)

2、[k8s 和 Docker 关系简单说明](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555539&idx=3&sn=973b8014cb03a91d2d6cd471b8f478ed&chksm=80dcac38b7ab252e53c4229fcf8cf82338fa1f92bb362bfa20777e699e40cb5df116c950d652&scene=21#wechat_redirect)

3、[面试题：a==1 && a==2 && a==3 是 true 还是 false？](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555445&idx=2&sn=b96ecd049c476762e589a078f65ed742&chksm=80dcac9eb7ab2588998971e0cd0975aa732ed678ce636f69ff1c78b7903d202463576babbf20&scene=21#wechat_redirect)

公众号