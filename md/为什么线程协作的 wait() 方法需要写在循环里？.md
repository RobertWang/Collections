> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzUzODU1MzEwNg==&mid=2247497979&idx=3&sn=9cf65b33a49a2e0ed4c5c6a5ea1fb11c&chksm=fad74686cda0cf90da7d1e1ecde40b5ea612fcbfc506b4f7f60a95fef85b451318b631231c1f&mpshare=1&scene=1&srcid=0604gyYM01XdsFp8HDBFgO1C&sharer_sharetime=1622742151011&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**问：为什么是 while 而不是 if ？**
-------------------------

大多数人都知道常见的使用 synchronized 代码:

```
synchronized (obj) {
     while (check pass) {
        wait();
    }
    // do your business
}


```

那么问题是为啥这里是 while 而不是 if 呢？这个问题我最开始也想了很久，按理来说已经在 synchronized 块里面了嘛，就不需要了。这个也是我前面一直是这么认为的，直到最近看了一个 Stackoverflow 上的问题才对这个问题有了比较深入的理解。

试想我们要试想一个有界的队列。那么常见的代码可以是这样：

```
static class Buf {
    private final int MAX = 5;
    private final ArrayList<Integer> list = new ArrayList<>();
    synchronized void put(int v) throws InterruptedException {
        if (list.size() == MAX) {
            wait();
        }
        list.add(v);
        notifyAll();
    }

    synchronized int get() throws InterruptedException {
        // line 0 
        if (list.size() == 0) {  // line 1
            wait();  // line2
            // line 3
        }
        int v = list.remove(0);  // line 4
        notifyAll(); // line 5
        return v;
    }

    synchronized int size() {
        return list.size();
    }
}


```

注意到这里用的 if，那么我们来看看它会报什么错呢？  
下面的代码用了 1 个线程来 put，10 个线程来 get：

```
final Buf buf = new Buf();
ExecutorService es = Executors.newFixedThreadPool(11);
for (int i = 0; i < 1; i++)
es.execute(new Runnable() {

    @Override
    public void run() {
        while (true ) {
            try {
                buf.put(1);
                Thread.sleep(20);
            }
            catch (InterruptedException e) {
                e.printStackTrace();
                break;
            }
        }
    }
});
for (int i = 0; i < 10; i++) {
    es.execute(new Runnable() {

        @Override
        public void run() {
            while (true ) {
                try {
                    buf.get();
                    Thread.sleep(10);
                }
                catch (InterruptedException e) {
                    e.printStackTrace();
                    break;
                }
            }
        }
    });
}

es.shutdown();
es.awaitTermination(1, TimeUnit.DAYS);


```

这段代码很快或者说一开始就会报错：  

```
java.lang.IndexOutOfBoundsException: Index: 0, Size: 0
at java.util.ArrayList.rangeCheck(ArrayList.java:653) 
at java.util.ArrayList.remove(ArrayList.java:492) 
at TestWhileWaitBuf.get(TestWhileWait.java:80)atTestWhileWait2.run(TestWhileWait.java:47) 
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) 
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) 
at java.lang.Thread.run(Thread.java:745)


```

很明显，在 remove 的时候报错了。那么我们来分析下：

假设现在有 A，B 两个线程来执行 get 操作，我们假设如下的步骤发生了：

1. A 拿到了锁 line 0。

2. A 发现 size==0, (line 1)，然后进入等待，并释放锁 (line 2)。

3. 此时 B 拿到了锁，line0，发现 size==0，(line 1)，然后进入等待，并释放锁 (line 2)。

4. 这个时候有个线程 C 往里面加了个数据 1，那么 notifyAll 所有的等待的线程都被唤醒了。

5. AB 重新获取锁，假设又是 A 拿到了。然后他就走到 line 3，移除了一个数据，(line4) 没有问题。

6. A 移除数据后想通知别人，此时 list 的大小有了变化，于是调用了 notifyAll (line5)，这个时候就把 B 给唤醒了，那么 B 接着往下走。

7. 这时候 B 就出问题了，因为其实此时的竞态条件已经不满足了 (size==0)。B 以为还可以删除就尝试去删除，结果就跑了异常了。

那么 fix 很简单，在 get 的时候加上 while 就好了：

```
synchronized int get() throws InterruptedException {
      while (list.size() == 0) {
          wait();
      }
      int v = list.remove(0);
      notifyAll();
      return v;
  }


```

同样的，我们可以尝试修改 put 的线程数和 get 的线程数来发现如果 put 里面不是 while 的话也是不行的。

我们可以用一个外部周期性任务来打印当前 list 的大小，你会发现大小并不是固定的最大 5：

```
final Buf buf = new Buf();
ExecutorService es = Executors.newFixedThreadPool(11);
ScheduledExecutorService printer = Executors.newScheduledThreadPool(1);
printer.scheduleAtFixedRate(new Runnable() {
    @Override
    public void run() {
        System.out.println(buf.size());
    }
}, 0, 1, TimeUnit.SECONDS);
for (int i = 0; i < 10; i++)
es.execute(new Runnable() {

    @Override
    public void run() {
        while (true ) {
            try {
                buf.put(1);
                Thread.sleep(200);
            }
            catch (InterruptedException e) {
                 e.printStackTrace();
                break;
            }
        }
    }
});
for (int i = 0; i < 1; i++) {
    es.execute(new Runnable() {

        @Override
        public void run() {
            while (true ) {
                try {
                    buf.get();
                    Thread.sleep(100);
                }
                catch (InterruptedException e) {
                    e.printStackTrace();
                    break;
                }
            }
        }
    });
}

es.shutdown();
es.awaitTermination(1, TimeUnit.DAYS);


```

![](https://mmbiz.qpic.cn/mmbiz_png/iarMRfJHmKibXGlmG3IT58zibaGw7Cg3OLvqOy7Dhf8VuxLqNrQdsK9V5Rq68skkyMQ2lyfJ25M2XpJg5w1cGMEqA/640?wx_fmt=jpeg)

这里我想应该说清楚了为啥必须是 while 还是 if 了。

**问：什么时候用 notifyAll 或者 notify？**
--------------------------------

大多数人都会这么告诉你，当你想要通知所有人的时候就用 notifyAll，当你只想通知一个人的时候就用 notify。但是我们都知道 notify 实际上我们是没法决定到底通知谁的 (都是从等待集合里面选一个)。那这个还有什么存在的意义呢？

在上面的例子中，我们用到了 notifyAll，那么下面我们来看下用 notify 是否可以工作呢？  

```
synchronized void put(int v) throws InterruptedException {
       if (list.size() == MAX) {
           wait();
       }
       list.add(v);
       notify();
   }

   synchronized int get() throws InterruptedException {
       while (list.size() == 0) {
           wait();
       }
       int v = list.remove(0);
       notify();
       return v;
   }


```

下面的几点是 jvm 告诉我们的：

1.  任何时候，被唤醒的来执行的线程是不可预知。比如有 5 个线程都在一个对象上，实际上我不知道 下一个哪个线程会被执行。
    
2.  synchronized 语义实现了有且只有一个线程可以执行同步块里面的代码。
    

那么我们假设下面的场景就会导致死锁：

P – 生产者 调用 put。  
C – 消费者 调用 get。

1. P1 放了一个数字 1。

2. P2 想来放，发现满了，在 wait 里面等了。

3. P3 想来放，发现满了，在 wait 里面等了。

4. C1 想来拿，C2，C3 就在 get 里面等着。

5. C1 开始执行，获取 1，然后调用 notify 然后退出。

*   如果 C1 把 C2 唤醒了，所以 P2 (其他的都得等) 只能在 put 方法上等着。(等待获取 synchoronized (this) 这个 monitor)。
    
*   C2 检查 while 循环发现此时队列是空的，所以就在 wait 里面等着。
    
*   C3 也比 P2 先执行，那么发现也是空的，只能等着了。
    

6. 这时候我们发现 P2、C2、C3 都在等着锁，最终 P2 拿到了锁，放一个 1，notify，然后退出。

7. P2 这个时候唤醒了 P3，P3 发现队列是满的，没办法，只能等它变为空。  
8. 这时候没有别的调用了，那么现在这三个线程 (P3, C2,C3) 就全部变成 suspend 了，也就是死锁了。

翻译：scugxl

译文链接：http://www.importnew.com/26584.html