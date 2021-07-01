> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247504001&idx=1&sn=f6ed553b6b18a169482363177a3e1dee&chksm=e92aed8ede5d6498a1e2d5887d1e684f473e70496c5f5569e504cf91028da83f926f6439739a&mpshare=1&scene=1&srcid=0701GdugmyAMZyBBRFCdjp3W&sharer_sharetime=1625128188524&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**一  为什么讲这个？**

总结 AQS 之后，对这方面顺带的复习一下。本文从以下几个高频问题出发：

*   对象在内存中的内存布局是什么样的？
    
*   描述 synchronized 和 ReentrantLock 的底层实现和重入的底层原理。
    
*   谈谈 AQS，为什么 AQS 底层是 CAS+volatile？
    
*   描述下锁的四种状态和锁升级过程？
    
*   Object  o = new Object() 在内存中占用多少字节？
    
*   自旋锁是不是一定比重量级锁效率高？
    
*   打开偏向锁是否效率一定会提升？
    
*   重量级锁到底重在哪里？
    
*   重量级锁什么时候比轻量级锁效率高，同样反之呢？
    

**二  加锁发生了什么？**

无意识中用到锁的情况：

```
//System.out.println都加了锁
public void println(String x) {
  synchronized (this) {
    print(x);
    newLine();
  }
}

```

简单加锁发生了什么？  

要弄清楚加锁之后到底发生了什么需要看一下对象创建之后再内存中的布局是个什么样的？

一个对象在 new 出来之后在内存中主要分为 4 个部分：

*   markword 这部分其实就是加锁的核心，同时还包含的对象的一些生命信息，例如是否 GC、经过了几次 Young GC 还存活。
    

*   klass pointer 记录了指向对象的 class 文件指针。
    

*   instance data 记录了对象里面的变量数据。
    

*   padding 作为对齐使用，对象在 64 位服务器版本中，规定对象内存必须要能被 8 字节整除，如果不能整除，那么就靠对齐来补。举个例子：new 出了一个对象，内存只占用 18 字节，但是规定要能被 8 整除，所以 padding=6。
    

知道了这 4 个部分之后，我们来验证一下底层。借助于第三方包 JOL  = Java Object Layout java 内存布局去看看。很简单的几行代码就可以看到内存布局的样式：

```
public class JOLDemo {
    private static Object  o;
    public static void main(String[] args) {
        o = new Object();
        synchronized (o){
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }
    }
}

```

将结果打印出来：  

从输出结果看：

1）对象头包含了 12 个字节分为 3 行，其中前 2 行其实就是 markword，第三行就是 klass 指针。值得注意的是在加锁前后输出从 001 变成了 000。Markword 用处：8 字节 (64bit) 的头记录一些信息，锁就是修改了 markword 的内容 8 字节 (64bit) 的头记录一些信息，锁就是修改了 markword 的内容字节 (64bit) 的头记录一些信息。从 001 无锁状态，变成了 00 轻量级锁状态。

2）New 出一个 object 对象，占用 16 个字节。对象头占用 12 字节，由于 Object 中没有额外的变量，所以 instance = 0，考虑要对象内存大小要被 8 字节整除，那么 padding=4，最后 new Object() 内存大小为 16 字节。

拓展：什么样的对象会进入老年代？很多场景例如对象太大了可以直接进入，但是这里想探讨的是为什么从 Young GC 的对象最多经历 15 次 Young GC 还存活就会进入 Old 区（年龄是可以调的，默认是 15）。上图中 hotspots 的 markword 的图中，用了 4 个 bit 去表示分代年龄，那么能表示的最大范围就是 0-15。所以这也就是为什么设置新生代的年龄不能超过 15，工作中可以通过 - XX:MaxTenuringThreshold 去调整，但是一般我们不会动。

**三  锁的升级过程**

1  锁的升级验证

探讨锁的升级之前，先做个实验。两份代码，不同之处在于一个中途让它睡了 5 秒，一个没睡。看看是否有区别。

```
public class JOLDemo {
    private static Object  o;
    public static void main(String[] args) {
        o = new Object();
        synchronized (o){
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }
    }
}
----------------------------------------------------------------------------------------------
public class JOLDemo {
    private static Object  o;
    public static void main(String[] args) {
      try { Thread.sleep(5000); } catch (InterruptedException e) { e.printStackTrace(); }
        o = new Object();
        synchronized (o){
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }
    }
}

```

这两份代码会不会有什么区别？运行之后看看结果：  

有点意思的是，让主线程睡了 5s 之后输出的内存布局跟没睡的输出结果居然不一样。

Syn 锁升级之后，jdk1.8 版本的一个底层默认设置 4s 之后偏向锁开启。也就是说在 4s 内是没有开启偏向锁的，加了锁就直接升级为轻量级锁了。

那么这里就有几个问题了？

*   为什么要进行锁升级，以前不是默认 syn 就是重量级锁么？要么不用要么就用别的不行么？
    

*   既然 4s 内如果加了锁就直接到轻量级，那么能不能不要偏向锁，为什么要有偏向锁？
    

*   为什么要设置 4s 之后开始偏向锁？
    

**问题 1：为什么要进行锁升级？锁了就锁了，不就要加锁么？**

首先明确早起 jdk1.2 效率非常低。那时候 syn 就是重量级锁，申请锁必须要经过操作系统老大 kernel 进行系统调用，入队进行排序操作，操作完之后再返回给用户态。

内核态：用户态如果要做一些比较危险的操作直接访问硬件，很容易把硬件搞死（格式化，访问网卡，访问内存干掉、）操作系统为了系统安全分成两层，用户态和内核态 。申请锁资源的时候用户态要向操作系统老大内核态申请。Jdk1.2 的时候用户需要跟内核态申请锁，然后内核态还会给用户态。这个过程是非常消耗时间的，导致早期效率特别低。有些 jvm 就可以处理的为什么还交给操作系统做去呢？能不能把 jvm 就可以完成的锁操作拉取出来提升效率，所以也就有了锁优化。

**问题 2：为什么要有偏向锁？**

其实这本质上归根于一个概率问题，统计表示，在我们日常用的 syn 锁过程中 70%-80% 的情况下，一般都只有一个线程去拿锁，例如我们常使用的 System.out.println、StringBuffer，虽然底层加了 syn 锁，但是基本没有多线程竞争的情况。那么这种情况下，没有必要升级到轻量级锁级别了。偏向的意义在于：第一个线程拿到锁，将自己的线程信息标记在锁上，下次进来就不需要在拿去拿锁验证了。如果超过 1 个线程去抢锁，那么偏向锁就会撤销，升级为轻量级锁，其实我认为严格意义上来讲偏向锁并不算一把真正的锁，因为只有一个线程去访问共享资源的时候才会有偏向锁这个情况。

无意使用到锁的场景：

```
/***StringBuffer内部同步***/
public synchronized int length() {
  return count;
} 
//System.out.println 无意识的使用锁
public void println(String x) {
   synchronized (this) {
     print(x);
     newLine();
   }
 }

```

**问题 3：为什么 jdk8 要在 4s 后开启偏向锁？**  

其实这是一个妥协，明确知道在刚开始执行代码时，一定有好多线程来抢锁，如果开了偏向锁效率反而降低，所以上面程序在睡了 5s 之后偏向锁才开放。为什么加偏向锁效率会降低，因为中途多了几个额外的过程，上了偏向锁之后多个线程争抢共享资源的时候要进行锁升级到轻量级锁，这个过程还的把偏向锁进行撤销在进行升级，所以导致效率会降低。为什么是 4s？这是一个统计的时间值。

当然我们是可以禁止偏向锁的，通过配置参数 - XX:-UseBiasedLocking = false 来禁用偏向锁。jdk15 之后默认已经禁用了偏向锁。本文是在 jdk8 的环境下做的锁升级验证。

2  锁的升级流程

上面已经验证了对象从创建出来之后进内存从无锁状态 -> 偏向锁（如果开启了）-> 轻量级锁的过程。对于锁升级的流程继续往下，轻量级锁之后就会变成重量级锁。首先我们先理解什么叫做轻量级锁，从一个线程抢占资源（偏向锁）到多线程抢占资源升级为轻量级锁，线程如果没那么多的话，其实这里就可以理解为 CAS，也就是我们说的 Compare and Swap，比较并交换值。在并发编程中最简单的一个例子就是并发包下面的原子操作类 AtomicInteger。在进行类似 ++ 操作的时候，底层其实就是 CAS 锁。

```
public final int getAndIncrement() {
  return unsafe.getAndAddInt(this, valueOffset, 1);
}
public final int getAndAddInt(Object var1, long var2, int var4) {
   int var5;
   do {
       var5 = this.getIntVolatile(var1, var2);
   } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
   return var5;
}

```

**问题 4：什么情况下轻量级锁要升级为重量级锁呢？**

首先我们可以思考的是多个线程的时候先开启轻量级锁，如果它 carry 不了的情况下才会升级为重量级。那么什么情况下轻量级锁会 carry 不住。1、如果线程数太多，比如上来就是 10000 个，那么这里 CAS 要转多久才可能交换值，同时 CPU 光在这 10000 个活着的线程中来回切换中就耗费了巨大的资源，这种情况下自然就升级为重量级锁，直接叫给操作系统入队管理，那么就算 10000 个线程那也是处理休眠的情况等待排队唤醒。2、CAS 如果自旋 10 次依然没有获取到锁，那么也会升级为重量级。

总的来说 2 种情况会从轻量级升级为重量级，10 次自旋或等待 cpu 调度的线程数超过 cpu 核数的一半，自动升级为重量级锁。看服务器 CPU 的核数怎么看，输入 top 指令，然后按 1 就可以看到。

**问题 5：都说 syn 为重量级锁，那么到底重在哪里？**

JVM 偷懒把任何跟线程有关的操作全部交给操作系统去做，例如调度锁的同步直接交给操作系统去执行，而在操作系统中要执行先要入队，另外操作系统启动一个线程时需要消耗很多资源，消耗资源比较重，重就重在这里。

整个锁升级过程如图所示：

**四  synchronized 的底层实现**

上面我们对对象的内存布局有了一些了解之后，知道锁的状态主要存放在 markword 里面。这里我们看看底层实现。

```
public class RnEnterLockDemo {
     public void method() {
         synchronized (this) {
             System.out.println("start");
         }
     }
}

```

对这段简单代码进行反解析看看什么情况。javap -c RnEnterLockDemo.class  

首先我们能确定的是 syn 肯定是还有加锁的操作，看到的信息中出现了 monitorenter 和 monitorexit，主观上就可以猜到这是跟加锁和解锁相关的指令。有意思的是 1 个 monitorenter 和 2 个 monitorexit。为什么呢？正常来说应该就是一个加锁和一个释放锁啊。其实这里也体现了 syn 和 lock 的区别。syn 是 JVM 层面的锁，如果异常了不用自己释放，jvm 会自动帮助释放，这一步就取决于多出来的那个 monitorexit。而 lock 异常需要我们手动补获并释放的。

关于这两条指令的作用，我们直接参考 JVM 规范中描述：

> monitorenter ：
> 
> Each object is associated with a monitor. A monitor is locked if and only if it has an owner. The thread that executes monitorenter attempts to gain ownership of the monitor associated with objectref, as follows: • If the entry count of the monitor associated with objectref is zero, the thread enters the monitor and sets its entry count to one. The thread is then the owner of the monitor. • If the thread already owns the monitor associated with objectref, it reenters the monitor, incrementing its entry count. • If another thread already owns the monitor associated with objectref, the thread blocks until the monitor's entry count is zero, then tries again to gain ownership

翻译一下：

每个对象有一个监视器锁（monitor）。当 monitor 被占用时就会处于锁定状态，线程执行 monitorenter 指令时尝试获取 monitor 的所有权，过程如下：

*   如果 monitor 的进入数为 0，则该线程进入 monitor，然后将进入数设置为 1，该线程即为 monitor 的所有者。
    

*   如果线程已经占有该 monitor，只是重新进入，则进入 monitor 的进入数加 1。
    

*   如果其他线程已经占用了 monitor，则该线程进入阻塞状态，直到 monitor 的进入数为 0，再重新尝试获取 monitor 的所有权。
    

> monitorexit： 
> 
> The thread that executes monitorexit must be the owner of the monitor associated with the instance referenced by objectref. The thread decrements the entry count of the monitor associated with objectref. If as a result the value of the entry count is zero, the thread exits the monitor and is no longer its owner. Other threads that are blocking to enter the monitor are allowed to attempt to do so.

翻译一下：

执行 monitorexit 的线程必须是 objectref 所对应的 monitor 的所有者。指令执行时，monitor 的进入数减 1，如果减 1 后进入数为 0，那线程退出 monitor，不再是这个 monitor 的所有者。其他被这个 monitor 阻塞的线程可以尝试去获取这个 monitor 的所有权。 

通过这段话的描述，很清楚的看出 Synchronized 的实现原理，Synchronized 底层通过一个 monitor 的对象来完成，wait/notify 等方法其实也依赖于 monitor 对象，这就是为什么只有在同步的块或者方法中才能调用 wait/notify 等方法，否则会抛出 java.lang.IllegalMonitorStateException 的异常。

每个锁对象拥有一个锁计数器和一个指向持有该锁的线程的指针。

当执行 monitorenter 时，如果目标对象的计数器为零，那么说明它没有被其他线程所持有，Java 虚拟机会将该锁对象的持有线程设置为当前线程，并且将其计数器加 i。在目标锁对象的计数器不为零的情况下，如果锁对象的持有线程是当前线程，那么 Java 虚拟机可以将其计数器加 1，否则需要等待，直至持有线程释放该锁。当执行 monitorexit 时，Java 虚拟机则需将锁对象的计数器减 1。计数器为零代表锁已被释放。

**总结**

以往的经验中，只要用到 synchronized 就以为它已经成为了重量级锁。在 jdk1.2 之前确实如此，后来发现太重了，消耗了太多操作系统资源，所以对 synchronized 进行了优化。以后可以直接用，至于锁的力度如何，JVM 底层已经做好了我们直接用就行。

最后再看看开头的几个问题，是不是都理解了呢。带着问题去研究，往往会更加清晰。希望对大家有所帮助。

《Spring Boot 2.5 开发实战》  

本书包含了 Spring Boot 2.5 新特性、自动化配置原理、REST API 开发、MySQL、Redis 高并发缓存、MongoDB、MQ 消息队列、安全机制、 性能监控等核心知识点，带你上手实战！

点击 “阅读原文”，立即下载电子书~