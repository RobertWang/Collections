> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.awaimai.com](https://www.awaimai.com/2596.html)

> Spring 有跟多概念，其中最基本的一个就是 bean，那到底 spring bean 是什么?

Spring 有跟多概念，其中最基本的一个就是 bean，那到底 **spring bean 是什么**?

Bean 是 Spring 框架中**最核心**的两个概念之一（另一个是**面向切面编程 AOP**）。 是否正确理解 Bean 对于掌握和高效使用 Spring 框架至关重要。 遗憾的是，网上不计其数的文章，却没有简单而清晰的解释。 那么，Spring bean 到底是什么？ 接下来我们用图文方式来解析这一个概念。

1 定义
----

Spring 官方文档对 bean 的解释是：

> In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container.

翻译过来就是：

> 在 Spring 中，构成应用程序**主干**并由 **Spring IoC 容器**管理的**对象**称为 **bean**。bean 是一个由 Spring IoC 容器实例化、组装和管理的对象。

概念简单明了，我们提取处关键的信息：

1.  bean 是对象，一个或者多个不限定
2.  bean 由 Spring 中一个叫 IoC 的东西管理
3.  我们的应用程序由一个个 bean 构成

第 1 和 3 好理解，那么 IoC 又是什么东西？

2 控制反转（IoC）
-----------

**控制反转**英文全称：**Inversion of Control**，简称就是`IoC`。 控制反转通过依赖注入（DI）方式**实现对象之间的松耦合关系**。程序运行时，依赖对象由【辅助程序】动态生成并注入到被依赖对象中，动态绑定两者的使用关系。Spring IoC 容器就是这样的辅助程序，它负责对象的生成和依赖的注入，让后在交由我们使用。 简而言之，就是：IoC 就是**一个对象定义其依赖关系而不创建它们的过程**。 这里我们可以细分为两个点。

### 2.1 私有属性保存依赖

**第 1 点：使用私有属性保存依赖对象，并且只能通过构造函数参数传入，**构造函数的参数可以是**工厂方法**、**保存类对象的属性**、或者是**工厂方法返回值**。 假设我们有一个`Computer`类：

```
public class Computer {
private String cpu;     // CPU型号
private int ram;        // RAM大小，单位GB

    public Computer(String cpu, int ram) {
        this.cpu = cpu;
        this.ram = ram;
    }
}
```

我们有另一个`Person`类依赖于`Computer`类，符合 IoC 的做法是这样：

```
public class Person {
    private Computer computer;

    public Person(Computer computer) {
        this.computer = computer;
    }
}
```

不符合 IoC 的做法如下：

```
// 直接在Person里实例化Computer类
public class Person {
    private Computer computer = new Computer(AMD, 3);
}

// 通过【非构造函数】传入依赖
public class Person {
    private Computer computer;

    public void init(Computer computer) {
        this.computer = computer;
    }
}
```

### 2.2 让 Spring 控制类构建过程

**第 2 点：不用**`new`**，让 Spring 控制**`new`**过程。**在 Spring 中，我们基本不需要 `new` 一个类，这些都是让 Spring 去做的。 Spring 启动时会把所需的类实例化成对象，如果需要依赖，则先实例化依赖，然后实例化当前类。 因为依赖必须通过构建函数传入，所以实例化时，当前类就会接收并保存所有依赖的对象。 这一步也就是所谓的**依赖注入**。

### 2.3 这就是 IoC

在 Spring 中，**类的实例化、依赖的实例化、依赖的传入**都交由 Spring Bean 容器控制， 而不是用`new`方式实例化对象、通过非构造函数方法传入依赖等常规方式。 实质的控制权已经交由程序管理，而不是程序员管理，所以叫做控制反转。

3 Bean？
-------

至于 bean，则是几个概念。

*   概念 1：**Bean 容器**，或称 spring ioc 容器，主要用来管理对象和依赖，以及依赖的注入。
*   概念 2：bean 是一个 **Java 对象**，根据 bean 规范编写出来的类，并由 bean 容器生成的对象就是一个 bean。
*   概念 3：bean 规范。

![](https://www.awaimai.com/wp-content/uploads/2018/11/ioc-bean.png)

bean 规范如下：

1.  所有属性为`private`
2.  提供默认构造方法
3.  提供`getter`和`setter`
4.  实现`serializable`接口

**参考文章：**

1.  [Spring Dependency Injection](https://www.journaldev.com/2410/spring-dependency-injection)
2.  [Spring IoC, Spring Bean Example Tutorial](https://www.journaldev.com/2461/spring-ioc-bean-example-tutorial)
3.  [The IoC Container – Spring docs](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans)
4.  [What is a Spring Bean?](https://www.baeldung.com/spring-bean)
5.  [What in the world are Spring beans?](https://stackoverflow.com/questions/17193365/what-in-the-world-are-spring-beans)
6.  [How to understand Bean in Spring?](https://stackoverflow.com/questions/44475523/how-to-understand-bean-in-spring)
7.  [Java bean 是个什么概念？](https://www.zhihu.com/question/19773379)
8.  [Spring 中 Bean 及 @Bean 的理解](https://www.cnblogs.com/bossen/p/5824067.html)