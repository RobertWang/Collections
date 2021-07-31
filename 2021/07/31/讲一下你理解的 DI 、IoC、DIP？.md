> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488486&idx=1&sn=fc40caa326bfb826ede30691205f19f6&chksm=ce0da465f97a2d736f1f428a120232374835e1da3c57fdcbb1e14c604ece2a9b7768e6687922&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/oTKHc6F8tsia8HxiazcbMria4jN1ibu6XYVmRPkCicE0sfHQNfaOyyskic02Wr57Tub14huFE3WfGMYu30goyLtEu9xw/640?wx_fmt=jpeg)

作者 | 木小楠
--------

链接 |cnblogs.com/liuhaorain/p/3747470.html

摘要  

-----

面向对象设计（OOD）有助于我们开发出高性能、易扩展以及易复用的程序。其中，OOD 有一个重要的思想那就是依赖倒置原则（DIP），并由此引申出 IoC、DI 以及 Ioc 容器等概念。本文我们将一起学习这些概念，并理清他们之间微妙的关系。在学习之前，大家可以把自己的理解发表在留言区，共同探讨。

目录
--

*   前言
    
*   依赖倒置原则（DIP)
    
*   控制反转（IoC）
    
*   依赖注入（DI）
    
*   IoC 容器
    
*   总结
    

前言
--

对于大部分小菜来说，当听到大牛们高谈 DIP、IoC、DI 以及 IoC 容器等名词时，有没有瞬间石化的感觉？其实，这些 “高大上” 的名词，理解起来也并不是那么的难，关键在于入门。只要我们入门了，然后循序渐进，假以时日，自然水到渠成。

好吧，我们先初略了解一下这些概念。

**依赖倒置原则（DIP）：**一种软件架构设计的原则（抽象概念）。

**控制反转（IoC）：**一种反转流、依赖和接口的方式（DIP 的具体实现方式）。

**依赖注入（DI）：**IoC 的一种实现方式，用来反转依赖（IoC 的具体实现方式）。

**IoC 容器：**依赖注入的**框架**，用来映射依赖，管理对象创建和生存周期（DI 框架）。

哦！也许你正为这些陌生的概念而伤透脑筋。不过没关系，接下来我将为你一一道破这其中的玄机。

依赖倒置原则（DIP）
-----------

在讲概念之前，我们先看生活中的一个例子。

![](https://mmbiz.qpic.cn/mmbiz_jpg/oTKHc6F8tsia8HxiazcbMria4jN1ibu6XYVmM1g1I4AhcfDTZVibq80JCgxAp1jHibEZg611ngYdVfDNbAAQHjstbGxw/640)

相信大部分取过钱的朋友都深有感触，只要有一张卡，随便到哪一家银行的 ATM 都能取钱。在这个场景中，ATM 相当于高层模块，而银行卡相当于低层模块。ATM 定义了一个插口（接口），供所有的银行卡插入使用。也就是说，ATM 不依赖于具体的哪种银行卡。

它只需定义好银行卡的规格参数（接口），所有实现了这种规格参数的银行卡都能在 ATM 上使用。现实生活如此，软件开发更是如此。**依赖倒置原则，它转换了依赖，高层模块不依赖于低层模块的实现，而低层模块依赖于高层模块定义的接口**。通俗的讲，就是高层模块定义接口，低层模块负责实现。

> Bob Martins 对 DIP 的定义： 
> 
> 高层模块不应依赖于低层模块，两者应该依赖于抽象。 
> 
> 抽象不不应该依赖于实现，实现应该依赖于抽象。

如果生活中的实例不足以说明依赖倒置原则的重要性，那下面我们将通过软件开发的场景来理解为什么要使用依赖倒置原则。

**场景一  依赖无倒置（低层模块定义接口，高层模块负责实现）**

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsia8HxiazcbMria4jN1ibu6XYVmABZRQGD7amKIXmJla6hIhBHa3AGwZLNqG6z3prBPdSH3s9QibIqVUzw/640)

从上图中，我们发现高层模块的类依赖于低层模块的接口。因此，低层模块需要考虑到所有的接口。如果有新的低层模块类出现时，高层模块需要修改代码，来实现新的低层模块的接口。这样，就破坏了开放封闭原则。

**场景二 依赖倒置（高层模块定义接口，低层模块负责实现）**

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsia8HxiazcbMria4jN1ibu6XYVmzWB3D05olJzgZv9Hxliaib07RvfJKOWoibQKlAiaDh1ickDylbT0CNBkK6A/640)

在这个图中，我们发现高层模块定义了接口，将不再直接依赖于低层模块，低层模块负责实现高层模块定义的接口。这样，当有新的低层模块实现时，不需要修改高层模块的代码。

由此，我们可以总结出使用 DIP 的优点：

**系统更柔韧：**可以修改一部分代码而不影响其他模块。

**系统更健壮：**可以修改一部分代码而不会让系统崩溃。

**系统更高效：**组件松耦合，且可复用，提高开发效率。

控制反转（IoC）
---------

DIP 是一种**软件设计原则**，它仅仅告诉你两个模块之间应该如何依赖，但是它并没有告诉如何做。IoC 则是一种**软件设计模式**，它告诉你应该如何做，来解除相互依赖模块的耦合。**控制反转（IoC），它为相互依赖的组件提供抽象，将依赖（低层模块）对象的获得交给第三方（系统）来控制**，**即依赖对象不在被依赖模块的类中直接通过 new 来获取**。

在图 1 的例子我们可以看到，ATM 它自身并没有插入具体的银行卡（工行卡、农行卡等等），而是将插卡工作交给人来控制，即我们来决定将插入什么样的银行卡来取钱。同样我们也通过软件开发过程中场景来加深理解。

> **软件设计原则：**原则为我们提供指南，它告诉我们什么是对的，什么是错的。它不会告诉我们如何解决问题。它仅仅给出一些准则，以便我们可以设计好的软件，避免不良的设计。一些常见的原则，比如 DRY、OCP、DIP 等。
> 
> **软件设计模式：**模式是在软件开发过程中总结得出的一些可重用的解决方案，它能解决一些实际的问题。一些常见的模式，比如工厂模式、单例模式等等。

做过电商网站的朋友都会面临这样一个问题：**订单入库**。假设系统设计初期，用的是 SQL Server 数据库。通常我们会定义一个 SqlServerDal 类，用于数据库的读写。

```
public class SqlServerDal
{
     public void Add()
    {
        Console.WriteLine("在数据库中添加一条订单!");
    }
}
```

然后我们定义一个 Order 类，负责订单的逻辑处理。由于订单要入库，需要依赖于数据库的操作。因此在 Order 类中，我们需要定义 SqlServerDal 类的变量并初始化。

```
public class Order
{
        private readonly SqlServerDal dal = new SqlServerDal();//添加一个私有变量保存数据库操作的对象
         public void Add()
       {
           dal.Add();
       }
}
```

最后，我们写一个控制台程序来检验成果。

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
namespace DIPTest
{
    class Program
    {
        static void Main(string[] args)
        {
            Order order = new Order();
            order.Add();
            Console.Read();
        }
    }
}
```

 输出结果：

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsia8HxiazcbMria4jN1ibu6XYVmSHRknOU5DQibYiapdBwiceNRsAktY22gmATnTDq8X5DDEyZBCqibRmBx0Q/640)

OK，结果看起来挺不错的！正当你沾沾自喜的时候，这时 BOSS 过来了。“小刘啊，刚客户那边打电话过来说数据库要改成 Access”，“对你来说，应当小 CASE 啦！”BOSS 又补充道。带着自豪而又纠结的情绪，我们思考着修改代码的思路。

 由于换成了 Access 数据库，SqlServerDal 类肯定用不了了。因此，我们需要新定义一个 AccessDal 类，负责 Access 数据库的操作。

```
public class AccessDal
{
    public void Add()
   {
       Console.WriteLine("在ACCESS数据库中添加一条记录！");
   }
}
```

 然后，再看 Order 类中的代码。由于，Order 类中直接引用了 SqlServerDal 类的对象。所以还需要修改引用，换成 AccessDal 对象。

```
public class Order
{
        private readonly AccessDal dal = new AccessDal();//添加一个私有变量保存数据库操作的对象
         public void Add()
       {
           dal.Add();
       }
}
```

输出结果：

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsia8HxiazcbMria4jN1ibu6XYVm09GX3q1POudAgdpUz6Riafw9nw9WDdDnwaibd9VOdkVEGqXialqtoDRnQ/640)

费了九牛二虎之力，程序终于跑起来了！试想一下，如果下次客户要换成 MySql 数据库，那我们是不是还得重新修改代码？

显然，这不是一个良好的设计，组件之间高度耦合，可扩展性较差，它违背了 DIP 原则。高层模块 Order 类不应该依赖于低层模块 SqlServerDal，AccessDal，两者应该依赖于抽象。那么我们是否可以通过 IoC 来优化代码呢？答案是肯定的。IoC 有 2 种常见的实现方式：依赖注入和服务定位。其中，依赖注入使用最为广泛。下面我们将深入理解依赖注入（DI），并学会使用。

依赖注入（DI）
--------

控制反转（IoC）一种重要的方式，**就是将依赖对象的创建和绑定转移到被依赖对象类的外部来实现**。在上述的实例中，Order 类所依赖的对象 SqlServerDal 的创建和绑定是在 Order 类内部进行的。事实证明，这种方法并不可取。既然，不能在 Order 类内部直接绑定依赖关系，那么如何将 SqlServerDal 对象的引用传递给 Order 类使用呢？

依赖注入（DI），**它提供一种机制，将需要依赖（低层模块）对象的引用传递给被依赖（高层模块）对象。**通过 DI，我们可以在 Order 类的外部将 SqlServerDal 对象的引用传递给 Order 类对象。那么具体是如何实现呢？

### **方法一 构造函数注入**

构造函数函数注入，毫无疑问通过构造函数传递依赖。因此，构造函数的参数必然用来接收一个依赖对象。那么参数的类型是什么呢？具体依赖对象的类型？还是一个抽象类型？根据 DIP 原则，我们知道高层模块不应该依赖于低层模块，两者应该依赖于抽象。那么构造函数的参数应该是一个抽象类型。我们再回到上面那个问题，**如何将 SqlServerDal 对象的引用传递给 Order 类使用呢**？

首选，我们需要定义 SqlServerDal 的抽象类型 IDataAccess，并在 IDataAccess 接口中声明一个 Add 方法。

```
public interface IDataAccess
{
        void Add();
}
```

 然后在 SqlServerDal 类中，实现 IDataAccess 接口。

```
public class SqlServerDal:IDataAccess
{
       public void Add()
       {
           Console.WriteLine("在数据库中添加一条订单！");
       }
}
```

接下来，我们还需要修改 Order 类。

```
public class Order
  {
        private IDataAccess _ida;//定义一个私有变量保存抽象
        //构造函数注入
        public Order(IDataAccess ida)
        {
            _ida = ida;//传递依赖
      }
        public void Add()
        {
            _ida.Add();
        }
}
```

 OK，我们再来编写一个控制台程序。

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
namespace DIPTest
{
    class Program
    {
        static void Main(string[] args)
        {
            SqlServerDal dal = new SqlServerDal();//在外部创建依赖对象
            Order order = new Order(dal);//通过构造函数注入依赖
            order.Add();
            Console.Read();
        }
    }
}
```

 输出结果：

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsia8HxiazcbMria4jN1ibu6XYVmSHRknOU5DQibYiapdBwiceNRsAktY22gmATnTDq8X5DDEyZBCqibRmBx0Q/640)

从上面我们可以看出，我们将依赖对象 SqlServerDal 对象的创建和绑定转移到 Order 类外部来实现，这样就解除了 SqlServerDal 和 Order 类的耦合关系。当我们数据库换成 Access 数据库时，只需定义一个 AccessDal 类，然后外部重新绑定依赖，不需要修改 Order 类内部代码，则可实现 Access 数据库的操作。

定义 AccessDal 类：

```
public class AccessDal:IDataAccess
{
        public void Add()
        {
            Console.WriteLine("在ACCESS数据库中添加一条记录！");
        }
}
```

然后在控制台程序中重新绑定依赖关系：

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
namespace DIPTest
{
    class Program
    {
        static void Main(string[] args)
        {
             AccessDal dal = new AccessDal();//在外部创建依赖对象
               Order order = new Order(dal);//通过构造函数注入依赖
               order.Add();
            Console.Read();
        }
    }
}
```

输出结果：

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsia8HxiazcbMria4jN1ibu6XYVm09GX3q1POudAgdpUz6Riafw9nw9WDdDnwaibd9VOdkVEGqXialqtoDRnQ/640)

显然，我们不需要修改 Order 类的代码，就完成了 Access 数据库的移植，这无疑体现了 IoC 的精妙。

###  **方法二 属性注入**

顾名思义，属性注入是通过属性来传递依赖。因此，我们首先需要在依赖类 Order 中定义一个属性：

```
public class Order
{
      private IDataAccess _ida;//定义一个私有变量保存抽象
        //属性，接受依赖
        public IDataAccess Ida
       {
           set { _ida = value; }
           get { return _ida; }
       }
       public void Add()
       {
           _ida.Add();
       }
}
```

 然后在控制台程序中，给属性赋值，从而传递依赖：

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
namespace DIPTest
{
    class Program
    {
        static void Main(string[] args)
        {
            AccessDal dal = new AccessDal();//在外部创建依赖对象
            Order order = new Order();
            order.Ida = dal;//给属性赋值
            order.Add();
            Console.Read();
        }
    }
}
```

我们可以得到上述同样的结果。

**方法三 接口注入**

相比构造函数注入和属性注入，接口注入显得有些复杂，使用也不常见。具体思路是先定义一个接口，包含一个设置依赖的方法。然后依赖类，继承并实现这个接口。

首先定义一个接口： 

```
public interface IDependent
{
           void SetDependence(IDataAccess ida);//设置依赖项
}
```

依赖类实现这个接口：

```
public class Order : IDependent
 {
     private IDataAccess _ida;//定义一个私有变量保存抽象
     //实现接口
     public void SetDependence(IDataAccess ida)
     {
         _ida = ida;
     }
     public void Add()
     {
         _ida.Add();
     }
 }
```

控制台程序通过 SetDependence 方法传递依赖：

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
namespace DIPTest
{
    class Program
    {
        static void Main(string[] args)
        {
            AccessDal dal = new AccessDal();//在外部创建依赖对象
          Order order = new Order();
            order.SetDependence(dal);//传递依赖
            order.Add();
            Console.Read();
        }
    }
}
```

我们同样能得到上述的输出结果。

IoC 容器
------

前面所有的例子中，我们都是通过**手动**的方式来创建依赖对象，并将引用传递给被依赖模块。比如：

```
SqlServerDal dal = new SqlServerDal();
Order order = new Order(dal);
```

 对于大型项目来说，相互依赖的组件比较多。如果还用手动的方式，自己来创建和注入依赖的话，显然效率很低，而且往往还会出现不可控的场面。正因如此，IoC 容器诞生了。IoC 容器实际上是一个 DI 框架，它能简化我们的工作量。它包含以下几个功能：

*   动态创建、注入依赖对象。
    
*   管理对象生命周期。
    
*   映射依赖关系。
    

目前，比较流行的 Ioc 容器有以下几种：

1.**Ninject**:  http://www.ninject.org/

2.**Castle Windsor**:  http://www.castleproject.org/container/index.html

3.**Autofac**:  http://code.google.com/p/autofac/

4.**StructureMap**：http://docs.structuremap.net/

5.**Unity**：  http://unity.codeplex.com/

6.**MEF**:  http://msdn.microsoft.com/zh-cn/library/dd460648.aspx 

7.**Spring.NET**：http://www.springframework.net/

8.**LightInject**:  http://www.lightinject.net/ （推荐使用 Chrome 浏览器访问）

以 Ninject 为例，我们同样来实现 **[方法一 构造函数注入]** 的功能

首先在项目添加 Ninject 程序集，同时使用 using 指令引入。 

```
using Ninject;
```

然后，Ioc 容器注册绑定依赖：

```
StandardKernel kernel = new StandardKernel();
kernel.Bind<IDataAccess>().To<SqlServerDal>();//注册依赖
```

接下来，我们获取需要的 Order 对象（注入了依赖对象）：

```
Order order = kernel.Get<Order>();
```

 下面，我们写一个完整的控制台程序

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Ninject;
namespace DIPTest
{
    class Program
    {
        static void Main(string[] args)
        {
           StandardKernel kernel = new StandardKernel();//创建Ioc容器
           kernel.Bind<IDataAccess>().To<SqlServerDal>();//注册依赖
             Order order = kernel.Get<Order>();//获取目标对象
             order.Add();
           Console.Read();
        }
    }
}
```

 输出结果：

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsia8HxiazcbMria4jN1ibu6XYVmSHRknOU5DQibYiapdBwiceNRsAktY22gmATnTDq8X5DDEyZBCqibRmBx0Q/640)

使用 IoC 容器，我们同样实现了该功能。

总结
--

在本文中，我试图以最通俗的方式讲解，希望能帮助大家理解这些概念。下面我们一起来总结一下：DIP 是软件设计的一种思想，IoC 则是基于 DIP 衍生出的一种软件设计模式。DI 是 IoC 的具体实现方式之一，使用最为广泛。IoC 容器是 DI 构造函注入的框架，它管理着依赖项的生命周期以及映射关系。

如果喜欢本篇文章，欢迎转发、点赞。关注订阅号「Web 项目聚集地」，回复「技术博文」即可获取更多图文教程、技术博文。

推荐阅读

_**1.**_ [因 转账失败 引发的思考...](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488455&idx=1&sn=7dcd97f1824cf791c7e0f2c4232e6bbf&chksm=ce0da444f97a2d5285e9244fdd38b5ee404c1e71fc19fba486835a57ce08f0d0794e31f87546&scene=21#wechat_redirect)

_**2.**_ [如何优雅地编码？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488455&idx=2&sn=3e7956ce8909a4cc5526867af0ff8ec4&chksm=ce0da444f97a2d524dbe79eb845c94cbe8a1f45829f73e1923d2b0f2f81cfb510fcec6fd9c7f&scene=21#wechat_redirect)

_**3.**_ [Java 程序员 “吃完饭直接走”](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=1&sn=4cef717f2e51f8c8e969d549e014cbed&chksm=ce0da43bf97a2d2d8a572536f7c2e92525e5f374f0298b2d85925c2243f84129c7a2d47ce21e&scene=21#wechat_redirect)

_**4. [](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=2&sn=818107afc6c7e62ffec4bbecc0f11359&chksm=ce0da43bf97a2d2d1810a6c565e6bc9f38bd757b758a9637a20ff246bba0f9a54a2c3589198e&scene=21#wechat_redirect)**_ [一键登陆了解一下？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=2&sn=818107afc6c7e62ffec4bbecc0f11359&chksm=ce0da43bf97a2d2d1810a6c565e6bc9f38bd757b758a9637a20ff246bba0f9a54a2c3589198e&scene=21#wechat_redirect)

_**5. [](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=3&sn=6d2c3de5fd95236e471c1ed0361843df&chksm=ce0da43bf97a2d2d25473580e1571b0b182cd62c4f9a0eba92c4c0a8b43ea6c1049a06abc776&scene=21#wechat_redirect)**_ [9 张 Java 技术流程图](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=3&sn=6d2c3de5fd95236e471c1ed0361843df&chksm=ce0da43bf97a2d2d25473580e1571b0b182cd62c4f9a0eba92c4c0a8b43ea6c1049a06abc776&scene=21#wechat_redirect)

_**6. [](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=4&sn=e2970c3db59752acf282741c4aed0364&chksm=ce0da43bf97a2d2df92bcf57aaf8e444e6631035e60c828bfb7416465fc5a14f7978c2fba60e&scene=21#wechat_redirect)**_ [MySQL 请不要使用 “utf-8”](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488440&idx=4&sn=e2970c3db59752acf282741c4aed0364&chksm=ce0da43bf97a2d2df92bcf57aaf8e444e6631035e60c828bfb7416465fc5a14f7978c2fba60e&scene=21#wechat_redirect)

_**7.**_ [还不懂 Java 中的多线程 ？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488085&idx=1&sn=66d41311ea6c4eeb4418a007e5d67b27&chksm=ce0da5d6f97a2cc0e4ea693f403adbd11614eedc9fbb2e3e7e6c51c3b9d03c60e8844ec6eb9d&scene=21#wechat_redirect)

_**8.**_ [如何编写轻量级 CSS 框架](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488286&idx=1&sn=08d7d117bac2c520e9847c03c70f576e&chksm=ce0da49df97a2d8b7c0fc13620eef9542e5bbec1de7d85454aedc9eb64483985bca7ff1ab0bc&scene=21#wechat_redirect)
