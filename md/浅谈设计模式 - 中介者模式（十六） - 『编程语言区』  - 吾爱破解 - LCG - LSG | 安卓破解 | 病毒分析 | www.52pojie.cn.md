> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1534780&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://avatar.52pojie.cn/data/avatar/001/49/77/18_avatar_middle.jpg)zxdsb666. 

浅谈设计模式 - 中介者模式（十六）
==================

前言
==

​ **中介者模式**是一种行为设计模式， 他的目的和门面模式类似，他负责的是将所有的底层交互细节隐藏，提供统一的对外接口供外部调用。 该模式会限制对象之间的直接交互， 迫使它们通过一个中介者对象进行合作。

优缺点：
====

**优点：** 1、将对象的交互模式由多对一变为一对一。 2、可以实现多个类之间的强耦合。 3、此设计模式十分符合迪米特原则。

> 迪米特法则（Law of Demeter）又叫作最少知识原则（The Least Knowledge Principle），一个类对于其他类知道的越少越好，就是说一个对象应当对其他对象有尽可能少的了解, 只和朋友通信，不和陌生人说话

**缺点：**中介者对象会逐渐庞大，如果组件过多会变得复杂难以维护。

使用场景
----

​ 1、在系统内部的对象之间通信紊乱的时候，按照设计模式**单一职责**的原则，通常会使用三方对象负责 “请求” 的转发，而中介的作用就是再次之上一个扩展，可以实现多个类之间的互相通信变为一对一的通信模式。

​ 2、想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类。

​ 3、如果多个用户之间存在复杂的通信和耦合的情况时候，对外使用外观模式，对内则可以使用中介者模式。

> **注意事项：**不应当在职责混乱的时候使用。

结构图
===

​ 下面是中介者模式的结构图，这里构建了多个具体的实现对象，他们需要实现统一的通信就需要

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20211027230458.png)

和门面模式的对比
--------

​ 下面是门面模式的结构图，门面模式也叫做外观模式，可以看到和中介者的对象还有一定的相似性，外观提供，这两个模式的区别是**外观主要提供对象的一致对外的接口，而中介模式实现的是让对象之间的通信由多对一变为一对一**，我们简单的认为，中介者主要针对的对内多个复杂对象的统一，外观针对对外多个对象的方法统一。

![](https://gitee.com/lazyTimes/imageReposity/raw/master/img/20210303223258.png)

实际案例
====

​ 下面我们用一段简短的代码快速介绍一下这个设计模式的代码， 其实从上面结构图基本可以快速的写出模板代码，下面按照一个租房的案例进行快速的讲解：

​ 首先是中介者的接口，中介者的接口需要由需要进行通信的子类实现，这里定义了租房的联络接口

```
public abstract class Mediator {
    //申明一个联络方法
    public abstract void constact(String message,Person person);
}
```

```
public abstract class Person {
    protected String name;
    protected Mediator mediator;

    Person(String name,Mediator mediator){
        this.name = name;
        this.mediator = mediator;
    }

}
```

房东类，继承 Person 类的同时，实现自己的通信方法

```
public class HouseOwner extends Person{

    HouseOwner(String name, Mediator mediator) {
        super(name, mediator);
    }

    /**
     * home.php?mod=space&uid=1781681 与中介者联系
     * home.php?mod=space&uid=952169 message
     * home.php?mod=space&uid=155549 void
     */
    public void constact(String message){
        mediator.constact(message, this);
    }

    /**
     * @desc 获取信息
     * @param message
     * @return void
     */
    public void getMessage(String message){
        System.out.println("房主:" + name +",获得信息：" + message);
    }
}
```

下面是另一个 Person 类，也就是中介者的对象，负责和房东进行通信

```
public class Tenant extends Person{

    Tenant(String name, Mediator mediator) {
        super(name, mediator);
    }

    /**
     * @desc 与中介者联系
     * @param message
     * @return void
     */
    public void constact(String message){
        mediator.constact(message, this);
    }

    /**
     * @desc 获取信息
     * @param message
     * @return void
     */
    public void getMessage(String message){
        System.out.println("租房者:" + name +",获得信息：" + message);
    }
}
```

下面是具体的中介对象，可以看到下面定义了房东的的信息以及租房者的对象，通过中介对象，我们实现了房东和房客之间的通信，让他们统一通过中介机构进行通信。

```
public class MediatorStructure extends Mediator{
    //首先中介结构必须知道所有房主和租房者的信息
    private HouseOwner houseOwner;
    private Tenant tenant;

    public HouseOwner getHouseOwner() {
        return houseOwner;
    }

    public void setHouseOwner(HouseOwner houseOwner) {
        this.houseOwner = houseOwner;
    }

    public Tenant getTenant() {
        return tenant;
    }

    public void setTenant(Tenant tenant) {
        this.tenant = tenant;
    }

    public void constact(String message, Person person) {
        if(person == houseOwner){          //如果是房主，则租房者获得信息
            tenant.getMessage(message);
        }
        else{       //反正则是房主获得信息
            houseOwner.getMessage(message);
        }
    }
}
```

最后是客户端的代码，可以看到通过中介的形式，可以实现中介和房东一对一，以及租房者和中介一对一的解耦合的形式，

```
public class Client {
    public static void main(String[] args) {
        //一个房主、一个租房者、一个中介机构
        MediatorStructure mediator = new MediatorStructure();

        //房主和租房者只需要知道中介机构即可
        HouseOwner houseOwner = new HouseOwner("张三", mediator);
        Tenant tenant = new Tenant("李四", mediator);

        //中介结构要知道房主和租房者
        mediator.setHouseOwner(houseOwner);
        mediator.setTenant(tenant);

        tenant.constact("听说你那里有三室的房主出租.....");
        houseOwner.constact("是的!请问你需要租吗?");
    }
}
```

总结
==

1.  由于中介者需要将多个对象的行为进行连接通信，虽然各个对象的改动会改变通信的细节，但是中介者的改动却会影响整体通信行为，这是一把双刃剑，既让对象的通信简化同时，也让复杂耦合到了一处。这时候可以结合策略或者工厂将中介的通信行为实现动态的扩展
2.  **在中介者模式中通过引用中介者对象，将系统中有关的对象所引用的其他对象数目减少到最少。**
3.  中介的行为类似一种对象的闭包和统一通信，中介者模式按照房东，中介，房客的理解可以快速的了解到这个结构
4.  其实我们可以把 MVC 看作一种中介的变种，Service 层起到了承接的作用。

写在最后
====

​ 中介的行为经常在使用，但是使用中介者模式需要小心的考虑类的耦合和中介者类是否会变得复杂或者难以维护。