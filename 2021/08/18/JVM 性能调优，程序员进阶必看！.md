> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI5OTI4OTY5NQ==&mid=2247496034&idx=3&sn=c74e1f938f00db50a3965c74ef6709e4&chksm=ec9a6c52dbede544f74c00beecd1c98b2c6f9ae437fc8371d4b83ffe98cfae50acce85d0ee72&mpshare=1&scene=1&srcid=0818sdSHGJ6otE0K8mbDTion&sharer_sharetime=1629291544195&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

*   JVisualVM 简介
    
*   案例分析
    

*   准备模拟内存泄漏样例
    
*   使用 JVisualVM 分析内存泄漏
    
*   JVisualVM 远程监控 Tomcat
    

**JVisualVM 简介**
================

VisualVM 是 Netbeans 的 profile 子项目，已在 JDK6.0 update 7 中自带，能够监控线程，内存情况，查看方法的 CPU 时间和内存中的对 象，已被 GC 的对象，反向查看分配的堆栈 (如 100 个 String 对象分别由哪几个对象分配出来的)。在 JDK_HOME/bin(默认是 C:\Program Files\Java\jdk1.6.0_13\bin) 目录下面，有一个 jvisualvm.exe 文件，双击打开，从 UI 上来看，这个软件是基于 NetBeans 开发的了。

VisualVM 提供了一个可视界面，用于查看 Java 虚拟机 (Java Virtual Machine, JVM) 上运行的基于 Java 技术的应用程序的详细信息。VisualVM 对 Java Development Kit (JDK) 工具所检索的 JVM 软件相关数据进行组织，并通过一种使您可以快速查看有关多个 Java 应用程序的数据的方式提供该信息。您可以查看本地应用程序或远程主机上运行的应用程序的相关数据。此外，还可以捕获有关 JVM 软件实例的数据，并将该数据保存到本地系统，以供后期查看或与其他用户共享。

双击启动 jvisualvm.exe，启动起来后和 jconsole 一样同样可以选择本地和远程，如果需要监控远程同样需要配置相关参数。

主界面如下；

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92p2BbU6crIqV4NJQ3WJxqbA2vgpz7XKEexKibaSdZ4xOq2RLXyC6pAhBg/640?wx_fmt=jpeg)

VisualVM 可以根据需要安装不同的插件，每个插件的关注点都不同，有的主要监控 GC，有的主要监控内存，有的监控线程等。

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pQicdF3sHpGKKdKOaAg5aJduZ8CFBKhhLhZUXJ1SlY81NqicFFCEQtXuw/640?wx_fmt=jpeg)

如何安装：

> 1、从主菜单中选择 “工具”>“插件”。2、在“可用插件” 标签中，选中该插件的 “安装” 复选框。单击“安装”。3、逐步完成插件安装程序。

我这里以 Eclipse(pid 22296) 为例，双击后直接展开，主界面展示了系统和 jvm 两大块内容，点击右下方 jvm 参数和系统属性可以参考详细的参数信息.

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92picaJPaIGJdMgwPCZ3vYdUD8mc54qCv1ibgQkEUAj7u4ShI6WyuNI1MWw/640?wx_fmt=jpeg)

因为 VisualVM 的插件太多，我这里主要介绍三个我主要使用几个：监控、线程、Visual GC

监控的主页其实也就是，cpu、内存、类、线程的图表

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92p0x1MJeFia6iaPEZTeGnhO2Da2PaGEbBSzqdIkMgcqziaQfpchiaEZmxM6g/640?wx_fmt=jpeg)

线程和 jconsole 功能没有太大的区别

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pFicAPRMpGKC0y5EYy3iaMc5Z2JDstDNkib6o2gKKGJ6PZBBY8BUHZmO1w/640?wx_fmt=jpeg)

Visual GC 是常常使用的一个功能，可以明显的看到年轻代、老年代的内存变化，以及 gc 频率、gc 的时间等。

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92picaJPaIGJdMgwPCZ3vYdUD8mc54qCv1ibgQkEUAj7u4ShI6WyuNI1MWw/640?wx_fmt=jpeg)

以上的功能其实 jconsole 几乎也有，VisualVM 更全面更直观一些，另外 VisualVM 非常多的其它功能，可以分析 dump 的内存快照，

dump 出来的线程快照并且进行分析等，还有其它很多的插件大家可以去探索

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pwGuicdAD2uZDoZ3rGAaX2fic6NIo365NzRmIxY1jmNLpQKePCEebsfmA/640?wx_fmt=jpeg)

**案例分析**
========

**准备模拟内存泄漏样例**
--------------

1、定义静态变量 HashMap

2、分段循环创建对象，并加入 HashMap

代码如下：

```
import java.util.HashMap;
import java.util.Map;
public class CyclicDependencies {
    //声明缓存对象
    private static final Map map = new HashMap();
    public static void main(String args[]){
        try {
            Thread.sleep(10000);//给打开visualvm时间
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //循环添加对象到缓存
        for(int i=0; i<1000000;i++){
            TestMemory t = new TestMemory();
            map.put("key"+i,t);
        }
        System.out.println("first");
        //为dump出堆提供时间
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        for(int i=0; i<1000000;i++){
            TestMemory t = new TestMemory();
            map.put("key"+i,t);
        }
        System.out.println("second");
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        for(int i=0; i<3000000;i++){
            TestMemory t = new TestMemory();
            map.put("key"+i,t);
        }
        System.out.println("third");
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        for(int i=0; i<4000000;i++){
            TestMemory t = new TestMemory();
            map.put("key"+i,t);
        }
        System.out.println("forth");
        try {
            Thread.sleep(Integer.MAX_VALUE);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("qqqq");
    }
}
```

3、配置 jvm 参数如下：

```
         -Xms512m
         -Xmx512m
         -XX:-UseGCOverheadLimit
         -XX:MaxPermSize=50m
```

4、运行程序并打卡 visualvm 监控

**使用 JVisualVM 分析内存泄漏**
-----------------------

1、查看 Visual GC 标签，内容如下，这是输出 first 的截图

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92prPLn4gWa5us4rIJhNvrnmVFZHG7drRj0VlIy9BssoOXUhecHxjZ21w/640?wx_fmt=png)

这是输出 forth 的截图：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pe4rMqIF7l5Rvg9bLQ7OtWHmia7rVPiarI4fPn8xdhdyhVlicSgayGmQMQ/640?wx_fmt=png)

通过 2 张图对比发现：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pHBK2T3mAsOsaxwkxblZmDZqKOSbGBCppVoZqOiblvfVMNZoExLGYNibg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92ppia0FMEzjvuTADEzjicXwFH3Z78fBojomZyZNNEicuvOcJETzR83B4cTQ/640?wx_fmt=png)

老生代一直在 gc，当程序继续运行可以发现老生代 gc 还在继续：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pwXcqWHCYYkegKibbQJnLf1dFPicra1BTs2ezxvyib6rB9v52DdXheYQ9g/640?wx_fmt=png)

增加到了 7 次，但是老生代的内存并没有减少。说明存在无法被回收的对象，可能是内存泄漏了。

如何分析是那个对象泄漏了呢？打开抽样器标签：点击后如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92p8Fq6CvLic2s66mrGGXqCMISpncRq3VGjqehqz6FYacOSLNgbsUXicCBQ/640?wx_fmt=png)

按照程序输出进行堆 dump，当输出 second 时，dump 一次，当输出 forth 时 dump 一次。

进入最后 dump 出来的堆标签，点击类：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pecr9lWMvibbn0zLoS7LkI3WvCWwL9w2H0EwVlOia1CKby9teXniaH9mtA/640?wx_fmt=png)

点击右上角：“与另一个堆存储对比”。如图选择第一次导出的 dump 内容比较：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pwcgPcUPocRaWPN0cw1cribYzm6dLJALwIiaUQwTaCN6DOr12D537hicLg/640?wx_fmt=png)

比较结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pyRp6NoWlpcUgukmWXztStxHym8nhjy6smJdj4fy7gPuiavX5qdwIUzQ/640?wx_fmt=png)

可以看出在两次间隔时间内 TestMemory 对象实例一直在增加并且多了，说明该对象引用的方法可能存在内存泄漏。

如何查看对象引用关系呢？

右键选择类 TestMemory，选择 “在实例视图中显示”，如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92plreX3osohFDdkpbibdzcovFmv9Pylc6xJ7SdCI7ibnqCibeU370A3oYQA/640?wx_fmt=png)

左侧是创建的实例总数，右侧上部为该实例的结构，下面为引用说明，从图中可以看出在类 CyclicDependencies 里面被引用了，并且被 HashMap 引用。

如此可以确定泄漏的位置，进而根据实际情况进行分析解决。

**JVisualVM 远程监控 Tomcat**
-------------------------

1、修改远程 tomcat 的 catalina.sh 配置文件，在其中增加：

1.  JAVA_OPTS="$JAVA_OPTS
    
2.  -Djava.rmi.server.hostname=192.168.122.128
    
3.  -Dcom.sun.management.jmxremote.port=18999
    
4.  -Dcom.sun.management.jmxremote.ssl=false
    
5.  -Dcom.sun.management.jmxremote.authenticate=false"
    
    这次配置先不走权限校验。只是打开 jmx 端口。
    

2、打开 jvisualvm，右键远程，选择添加远程主机：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92pZd0nS0TYHibW4zJUQWn6Pic3aVxLaN5xPOHPLRQ0mgJYmhexGNiaPiaV0Q/640?wx_fmt=png)

3、输入主机的名称，直接写 ip，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffBdQ2u7mibDAK9aMrPQX92ptANgnaduTUHg2UIJibEtoh89es3ANuOeQ1KhA7cKibChKFFBMNgoFe9Q/640?wx_fmt=png)

右键新建的主机，选择添加 JMX 连接，输入在 tomcat 中配置的端口即可。

4、双击打开。完毕！