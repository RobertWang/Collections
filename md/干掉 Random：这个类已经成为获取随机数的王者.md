> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455552258&idx=1&sn=f3861a06fdfb83b35520014eec8d7595&chksm=fb9cdd62cceb5474772494ff0ccb89824991db509da8a0de29b7bd6dd5df98346b0c0170189a&mpshare=1&scene=1&srcid=0220ulSmCDJ6JMtA2g95Wk8y&sharer_sharetime=1645328648841&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

*   背景
    
*   ThreadLocalRandom
    
*   Unsafe
    
*   疑问
    
*   小结
    

背景
--

#### 前言

#### Random 的性能问题

ThreadLocalRandom
-----------------

jdk 的开发者自然考虑到了这个问题，在 concurrent 包内添加了 `ThreadLocalRandom` 类，第一次看到这个类名，我以为它是通过 ThreadLocal 实现的，进而想到恐怖的内存泄漏问题，但点进源码却没有 ThreadLocal 的影子，而是存在着大量 Unsafe 相关的代码。

我们来看一下它的核心代码：

> ```
> UNSAFE.putLong(t = Thread.currentThread(), SEED, r = UNSAFE.getLong(t, SEED) + GAMMA);
> ```

翻译成更直观的 Java 代码就像：

```
Thread t = Thread.currentThread();
long r = UNSAFE.getLong(t, SEED) + GAMMA;
UNSAFE.putLong(t, SEED, r);
```

看上去非常眼熟，像我们平常往 Map 里 get/set 一样，以 `Thread.currentThread()` 获取到的当前对象里 key，以 SEED 随机种子作为 value。

Unsafe
------

#### 功能

```
public native long getLong(Object var1, long var2);
public native void putLong(Object var1, long var2, long var4);
```

*   putLong(object, offset, value) 可以将 object 对象内存地址偏移 offset 后的位置后四个字节设置为 value。
    
*   getLong(object, offset) 会从 object 对象内存地址偏移 offset 后的位置读取四个字节作为 long 型返回。
    

#### 不安全性

那么这两个方法” 不安全” 在哪呢？

它们的不安全并不是在这两个方法执行期间报错，而是未经保护地改变内存，会引起别的方法在使用这一段内存时报错。

```
public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        // Unsafe 设置了构造方法私有，getUnsafe 获取实例方法包私有，在包外只能通过反射获取
        Field field = Unsafe.class.getDeclaredField("theUnsafe");
        field.setAccessible(true);
        Unsafe unsafe = (Unsafe) field.get(null);
        // Test 类是一个随手写的测试类，只有一个 String 类型的测试类
        Test test = new Test();
        test.ttt = "12345";
        unsafe.putLong(test, 12L, 2333L);
        System.out.println(test.value);
    }
```

运行上面的代码会得到一个 fatal error，报错信息为 “A fatal error has been detected by the Java Runtime Environment: … Process finished with exit code 134 (interrupted by signal 6: SIGABRT)”。

可以从报错信息中看到虚拟机因为这个 fatal error abort 退出了，原因也很简单，我使用 unsafe 将 Test 类 value 属性的位置设置成了 long 型值 2333，而当我使用 value 属性时，虚拟机会将这一块内存解析为 String 对象，原 String 对象对象头的结构被打乱了，解析对象失败抛出了错误，更严重的问题是报错信息中没有类名行号等信息，在复杂项目中排查这种问题真如同大海捞针。

不过 Unsafe 的其他方法可不一定像这一对方法一样，使用他们时可能需要注意另外的安全问题，之后有遇到再说。

#### ThreadLocalRandom 的实现

那么 ThreadLocalRandom 是不是安全的呢，再回过头来看一下它的实现。

ThreadLocalRandom 的实现需要 Thread 对象的配合，在 Thread 对象内存在着一个属性 `threadLocalRandomSeed`，它保存着这个线程专属的随机种子，而这个属性在 Thread 对象的 offset，是在 ThreadLocalRandom 类加载时就确定了的，具体方法是 `SEED = UNSAFE.objectFieldOffset(Thread.class.getDeclaredField("threadLocalRandomSeed"));`

我们知道一个对象所占用的内存大小在类被加载后就确定了的，所以使用 `Unsafe.objectFieldOffset(class, fieldName)` 可以获取到某个属性在类中偏移量，而在找对了偏移量，又能确定数据类型时，使用 ThreadLocalRandom 就是很安全的。

疑问
--

在查找这些问题的过程中，我也产生了两个疑问点。

#### 使用场景

首先就是 ThreadLocalRandom 为什么非要使用 Unsafe 来修改 Thread 对象内的随机种子呢，在 Thread 对象内添加 get/set 方法不是更方便吗？

stackOverFlow 上有人跟我同样的疑问，why is threadlocalrandom implemented so bizarrely，被采纳的答案里解释说，对 jdk 开发者来说 Unsafe 和 get/set 方法都像普通的工具，具体使用哪一个并没有一个准则。这个答案并没有说服我，于是我另开了一个问题，里面的一个评论我比较认同，大意是 ThreadLocalRandom 和 Thread 不在同一个包下，如果添加 get/set 方法的话，get/set 方法必须设置为 public，这就有违了类的封闭性原则。

#### 内存布局

另一个疑问是我看到 Unsafe.objectFieldOffset 可以获取到属性在对象内存的偏移量后，自己在 IDEA 里使用 main 方法试了上文中提到的 Test 类，发现 Test 类的唯一一个属性 value 相对对象内存的偏移量是 12，于是比较疑惑这 12 个字节的组成。

我们知道，Java 对象的对象头是放在 Java 对象的内存起始处的，而一个对象的 MarkWord 在对象头的起始处，在 32 位系统中，它占用 4 个字节，而在 64 位系统中它占用 8 个字节，我使用的是 64 位系统，这毫无疑问会占用 8 个字节的偏移量。

紧跟 MarkWord 的应该是 Test 类的类指针和数组对象的长度，数组长度是 4 字节，但 Test 类并非数组，也没有其他属性，数据长度可以排除，但在 64 位系统下指针也应该是 8 字节的啊，为什么只占用了 4 个字节呢？

唯一的可能性是虚拟机启用了指针压缩，指针压缩只能在 64 位系统内启用，启用后指针类型只需要占用 4 个字节，但我并没有显示指定过使用指针压缩。查了一下，原来在 1.8 以后指针压缩是默认开启的，在启用时使用 `-XX:-UseCompressedOops` 参数后，value 的偏移量变成了 16。

小结
--

在写代码时还是要多注意查看依赖库的具体实现，不然可能踩到意想不到的坑，而且多看看并没有坏处，仔细研究一下还能学到更多。