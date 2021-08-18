> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247518280&idx=4&sn=39413521707e5c336557b742a69cb59b&chksm=fbb101b6ccc688a005b9da2b040f353c571c7861c2d279a5f6981432d0c1aade91cde0f82b76&mpshare=1&scene=1&srcid=0816N2t2sn462Jx4BrjAkBPy&sharer_sharetime=1629092580515&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

 我将常用的软件设计模式，做了汇总，目录如下：

![](https://mmbiz.qpic.cn/mmbiz_png/2KTof9YshwdUFgUFjwYicGyCx7mA6UjRdWvpsQgEBJoZoanIqLRK9SSMnibRPeibJ6vyich6ebJqGRlDsQ0nsOfyBw/640?wx_fmt=png)

(考虑到内容篇幅较大，为了便于大家阅读，将软件设计模式系列（共 23 个）拆分成四篇文章，每篇文章讲解六个设计模式，采用不同的颜色区分，便于快速消化记忆）

本文，主要讲解`模板模式`、`策略模式`、`状态模式`、`观察者模式`、`访问者模式`、`备忘录模式`

1、模板模式
------

**定义：**

> 定义一个操作中的算法的骨架，将一些步骤延迟到子类中。模板模式使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

优点：1、封装不变部分，扩展可变部分。2、提取公共代码，便于维护。3、行为由父类控制，子类实现。缺点：每一个不同的实现都需要一个子类来实现，导致类的个数增加，维护成本高。

**核心思路：**

*   AbstractTemplate：定义一个完整的框架，方法的调用顺序已经确定，但会定义一些抽象的方法留给子类去实现
    
*   AHandler：具体的业务子类，实现`AbstractTemplate`中定义的抽象方法，从而形成一个完整的流程逻辑
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshwdUFgUFjwYicGyCx7mA6UjRdJeLh0f4ic4khAcNxFO0gWnd3CyDAUoKMJ17jsee9me7RtCvpNSZImHw/640?wx_fmt=jpeg)

**代码示例：**  

```
public TradeFlowActionResult execute(TradeFlowActionParam param, Map context) throws ServiceException {
    try {    // 业务逻辑校验
        this.validateBusinessLogic(param, context);
    } catch (ServiceException ex) {
        sendGoodsLog.info("SendGoodsAction->validateBusinessLogic got exception. param is " + param, ex);
        throw ex;
    } catch (RuntimeException ex) {
        sendGoodsLog.info("SendGoodsAction->validateBusinessLogic got runtime exception. param is " + param, ex);
        throw ex;
    }
    try {
        // 卖家发货业务逻辑
        this.sendGoods(param, context);
    } catch (ServiceException ex) {
        sendGoodsLog.info("SendGoodsAction->sendGoods got exception. param is " + param, ex);
        throw ex;
    } catch (RuntimeException ex) {
        sendGoodsLog.info("SendGoodsAction->sendGoods got runtime exception. param is " + param, ex);
        throw ex;
    }
    try {
        // 补充业务（结果不影响核心业务）
        this.addition(param, context);
    } catch (ServiceException ex) {
        sendGoodsLog.info("SendGoodsAction->addition got exception. param is " + param, ex);
        throw ex;
    } catch (RuntimeException ex) {
        sendGoodsLog.info("SendGoodsAction->addition got runtime exception. param is " + param, ex);
        throw ex;
    }
    // 处理结果
    return null;
}
```

上面提到的三个抽象方法（业务逻辑校验、卖家发货业务逻辑、补充业务）都是在子类中实现的。即控制了主流程结构，又不失灵活性，可以让使用者在其基础上定制开发。如果有新的业务玩法进来，原来的流程满足不了需求，我们可以基于模板类编写新的子类。

**适用场景：**

*   希望控制算法的主流程，不能随意变更框架，但又想保留子类业务的个性扩展。
    
*   去除重复代码。保留父类通用的代码逻辑，让子类不再需要重复处理公用逻辑，只关注特定逻辑，起到去除子类中重复代码的目的。
    
*   案例很多，比如 Jenkins 的拉取代码、编译、打包、发布、部署的流程作为一个通用的流程，不同系统（java、python、nodejs 等）可以根据自身的需求开发，定制自己的持续发布流程。
    

2、策略模式
------

**定义：**

> 定义一系列算法，并将每种算法分别放入独立的类中，以使算法的对象能够相互替换。

由客户端自己决定在什么样的情况下使用哪些具体的策略。

**核心思路：**

*   上下文信息类（Context）：使用不同的策略环境，根据自身的条件选择不同的策略实现类来完成所需要的操作。他持有一个策略实例的引用。
    
*   抽象策略类（Strategy）：抽象策略，定义每个策略都要实现的方法
    
*   具体策略类（A Realize）：负责实现抽象策略中定义的策略方法。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshwdUFgUFjwYicGyCx7mA6UjRdAYacfPxibYfSwQj0EZ3R7YWDbibIrRVdnQsCKtMmIuricZkhTftZCM0Cw/640?wx_fmt=jpeg)

**代码示例：**

```
/**
 * @author 微信公众号：微观技术
 */
public interface PromotionStrategy {
    // 活动类型
    String promotionType();
    // 活动优惠
    int recommand(String productId);
}

public class FullSendPromotion implements PromotionStrategy {
    @Override
    public String promotionType() {
        return "FullSend";
    }

    @Override
    public int recommand(String productId) {
        System.out.println("参加满送活动");
        return 0;
    }
}

public class FullReducePromotion implements PromotionStrategy {
    @Override
    public String promotionType() {
        return "FullReduce";
    }

    @Override
    public int recommand(String productId) {
        System.out.println("参加满减活动");
        return 0;
    }
}

@author 微信公众号：微观技术
public class Context {

    private static List<PromotionStrategy> promotionStrategyList = new ArrayList<>();
    static {
        promotionStrategyList.add(new FullReducePromotion());
        promotionStrategyList.add(new FullSendPromotion());
    }
    public void recommand(String promotionType, String productId) {
        PromotionStrategy promotionStrategy = null;
        // 找到对应的策略类
        for (PromotionStrategy temp : promotionStrategyList) {
            if (temp.promotionType().equals(promotionType)) {
                promotionStrategy = temp;
            }
        }
        // 策略子类调用
        promotionStrategy.recommand(productId);
    }
```

首先，定义活动策略的接口类`PromotionStrategy`，定义接口方法，每一种具体的策略算法都要实现该接口，`FullSendPromotion`和`FullReducePromotion`。

`Context`负责存储和使用策略，汇集了所有的策略子类。并根据传入的参数，匹配到具体的策略子类，然后调用`recommand`方法，处理具体的业务。

**使用策略的好处：**

*   提升代码的可维护性。不同策略类隔离，互不影响。每一次新增策略时都通过新增类来进行隔离，避免了 if-else 超大的复杂类
    
*   动态快速地替换更多的算法。由于调度策略与算法实现分离，且接口规范固定，我们可以灵活的调整选择不同的策略子类。
    

**适用场景：**

*   压缩文件，提供了 gzip、zip 、rar 等格式，由客户端自己选择哪一种压缩策略。不同策略可以相互替换。
    
*   营销活动，根据策略路由选择不同的活动玩法，不同的营销活动隔离，满足开闭原则。
    
*   选择权交给了客户端，适合那些经常调整策略的 to C 业务，灵活性高。
    

3、状态模式
------

**定义：**

> 一种行为设计模式，让你能在一个对象的内部状态变化时改变其行为，使其看上去就像改变了自身所属的类一样。

通过定义一系列状态的变化来控制行为的变化。以电商为例，用户的订单会经历以下这些状态：已下单、已付款、已发货、派送中、待取件、已签收、交易成功、交易关闭等状态。

**核心思路：**

*   上下文信息类 (OrderContext)：存储`当前状态`的类，对外提供更新状态的方法。
    
*   抽象状态类（OrderState）：可以是一个接口或抽象类，用于声明状态更新时执行哪些操作
    
*   具体状态类（MakeOrderState、PayOrderState、ReceiveGoodOrderState）：实现抽象状态类定义的方法，根据具体的场景来指定对应状态改变后的代码实现逻辑。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshwdUFgUFjwYicGyCx7mA6UjRdfGwicCrI6vqjaS6dMQDnibw5eh48zq9s2KZib5LXTJXC2ricvg1kzby31g/640?wx_fmt=jpeg)

**代码示例：**

```
/**
 * @author 微信公众号：微观技术
 * 订单状态，接口定义（扩展实现若干不同状态的子类）
 */
public interface OrderState {
    void handle(OrderContext context);
}

// 下单
public class MakeOrderState implements OrderState {
    public static MakeOrderState instance = new MakeOrderState();
    @Override
    public void handle(OrderContext context) {

        System.out.println("1、创建订单");
        context.setCurrentOrderState(PayOrderState.instance);
    }
}

// 付款、确认收货，实现类相似，这里省略。。。。

// 订单上下文
public class OrderContext {
    private OrderState currentOrderState;

    public OrderContext(OrderState currentOrderState) {
        if (currentOrderState == null) {
            this.currentOrderState = new MakeOrderState();
        } else {
            this.currentOrderState = currentOrderState;
        }
    }

    public void setCurrentOrderState(OrderState currentOrderState) {
        this.currentOrderState = currentOrderState;
    }

    public void execute() {
        currentOrderState.handle(this);
    }
}
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshwdUFgUFjwYicGyCx7mA6UjRd4A7SvIW3Sh8JdjJ9YEGqDmFia8TX6jx4H1YHUkgpS8rwpXyo9rHnia9Q/640?wx_fmt=jpeg)

运行结果：

```
1、创建订单
2、支付宝付款
3、确认收到货物
```

状态模式设计的核心点在于找到合适的抽象状态以及状态之间的转移关系，通过改变状态来达到改变行为的目的。

**适用场景：**

*   业务需要根据状态的变化，进行不同的操作。比如：电商下单的全流程
    
*   不希望有大量的`if-else`代码堆在一起，希望不同的状态处理逻辑隔离，遵守开闭原则
    

4、观察者模式
-------

**定义：**

> 也称 发布 - 订阅模式，是一种通知机制，当一个对象改变状态时，它的所有依赖项都会自动得到通知和更新。让发送通知的一方（被观察者）和接收通知的一方（观察者，支持多个）能彼此分离，互不影响，该模式在软件开发中非常流行。

像我们常见的`Kafka`、`RocketMQ`等消息中间件都是采用这种架构模式，还有`Spring`的 `ApplicationEvent` 异步事件驱动，有很好的低耦合特性。

**类似其他叫法：**

*   发布者 --- 订阅者
    
*   生产者 --- 消费者
    
*   事件发布 --- 事件监听
    

**核心思路：**

*   发布者（Publisher）：定义一系列接口，用来管理和触发订阅者
    
*   具体发布者（PublisherImpl）：具体实现类，实现`Publisher`接口定义的方法
    
*   订阅者（Observer）：观察者接口，当发布者发布消息或事件时，会通知到订阅者进行处理。
    
*   具体订阅者（WeixinObserver、EmailObserver）：`Observer`的子类，用来处理具体的业务逻辑
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshwdUFgUFjwYicGyCx7mA6UjRdpfH2YrHS54nYqDJ9no1gOQakcs19cTl5qtEKichUmKZmmvURh0hycDg/640?wx_fmt=jpeg)

**代码示例：**

```
/**
 * @author 微信公众号：微观技术
 * <p>
 * 被观察者
 */
public interface Publisher {
    void add(Observer observer);
    void remove(Observer observer);
    void notify(Object object);
}

public class PublisherImpl implements Publisher {
    private List<Observer> observerList = new ArrayList<>();
    @Override
    public void add(Observer observer) {
        observerList.add(observer);
    }
    @Override
    public void remove(Observer observer) {
        observerList.remove(observer);
    }
    @Override
    public void notify(Object object) {
        System.out.println("创建订单，并发送通知事件");
        observerList.stream().forEach(t -> t.handle());
    }
}

// 观察者接口
public interface Observer {
    public void handle();
}

// 观察者子类
public class EmailObserver implements Observer {
    @Override
    public void handle() {
        System.out.println("发送邮件通知！");
    }
}
```

观察者模式的核心精髓：被观察者定义了一个通知列表，收集了所有的观察者对象，当`被观察者`需要发出通知时，就会通知这个列表的所有`观察者`。

**适用场景：**

*   当一个对象状态的改变需要改变其他对象时。比如：订单支付成功后，需要通知扣减账户余额
    
*   一个对象发生改变时只想要发送通知，而不需要知道接收者是谁。比如：微博 feed 流，粉丝能拉到最新微博
    
*   代码的扩展性强，如果需要新增一个`观察者`业务处理，只需新增一个`子类观察者`，并注入到`被观察者`的通知列表即可，代码的耦合性非常低。
    

5、访问者模式
-------

**定义：**

> 访问者模式是一种将操作与对象结构分离的软件设计模式，允许在运行时将一个或多个操作应用于一组对象。

**核心思路：**

*   行为接口（RouterVisitor）：定义对每个 Element 访问的行为，方法参数就是被访问的元素，它的方法个数理论上与元素的个数是一样的。
    
*   行为接口实现类（LinuxRouterVisitor、WindowRouterVisitor）：它需要给出对每一个元素类访问时所产生的具体行为。
    
*   元素接口（RouterElement）：定义一个可以获取访问操作的接口，使客户端对象能够 “访问” 的入口点。
    
*   元素接口实现类（JhjRouterElement、LyqRouterElement）：将访问者`RouterVisitor`传递给此对象作为参数。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshwdUFgUFjwYicGyCx7mA6UjRdTv3OXDibyTcFoiciaVLolQtCj7LqPibjLZ91Odmic9PzydmURQRVegzgeXw/640?wx_fmt=jpeg)

**代码示例：**

```
/**
 * @author 微信公众号：微观技术
 * 定义行为动作
 */
public interface RouterVisitor {
    // 交换机，发送数据
    void sendData(JhjRouterElement jhjRouterElement);
    // 路由器，发送数据
    void sendData(LyqRouterElement lyqRouterElement);
}

public class LinuxRouterVisitor implements RouterVisitor {
    @Override
    public void sendData(JhjRouterElement jhjRouterElement) {
        System.out.println("Linux 环境下，交换机发送数据");
    }

    @Override
    public void sendData(LyqRouterElement lyqRouterElement) {
        System.out.println("Linux 环境下，路由器发送数据");
    }
}

public class WindowRouterVisitor implements RouterVisitor {
   // 省略。。。
}


/**
 * @author 微信公众号：微观技术
 * <p>
 * 路由元素
 */
public interface RouterElement {
    // 发送数据
    void sendData(RouterVisitor routerVisitor);
}
public class JhjRouterElement implements RouterElement {

    @Override
    public void sendData(RouterVisitor routerVisitor) {
        System.out.println("交换机开始工作。。。");
        routerVisitor.sendData(this);
    }
}
public class LyqRouterElement implements RouterElement {
 // 省略。。。
}
```

**适用场景：**

*   动态绑定不同的对象和对象操作
    
*   通过行为与对象结构的分离，实现对象的职责分离，提高代码复用性
    

6、备忘录模式
-------

**定义：**

> 也叫快照模式，用来存储另外一个对象内部状态的快照，便于以后可以恢复。

**核心思路：**

*   原始对象（Originator）：除了创建自身所需要的属性和业务逻辑外，还通过提供方法 `bak()` 和 `restore(memento)` 来保存和恢复对象副本。
    
*   备忘录（Memento）：用于保存原始对象的所有属性状态，以便在未来进行恢复操作。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/2KTof9YshwdUFgUFjwYicGyCx7mA6UjRd0zKWQNKNYDTSrMlSmyLibA1qzZJGhFd8icR9Keib3CssYLCVHwvdBWjMA/640?wx_fmt=jpeg)

**代码示例：**

```
/**
 * @author 微信公众号：微观技术
 * 原始对象
 */
@Data
public class Originator {
    private Long id;
    private String productName;
    private String picture;
    // 创建快照
    public Memento bak() {
        return new Memento(id, productName, picture);
    }
    //恢复
    public void restore(Memento m) {
        this.id = m.getId();
        this.productName = m.getProductName();
        this.picture = m.getPicture();
    }
}


// 快照
@Data
@AllArgsConstructor
public class Memento {
    private Long id;
    private String productName;
    private String picture;
}
```

**适用场景：**

*   在线编辑器，不小心关闭浏览器，重新打开页面，可以从草稿箱恢复之前内容
    
*   不希望外界直接访问对象的内部状态，比如：物流包裹
    
*   另外像操作系统自动备份，数据库的`SAVEPOINT`
    

写在最后
----

设计模式很多人都学习过，但项目实战时总是晕晕乎乎，原因在于没有了解其核心是什么，底层逻辑是什么，《设计模式：可复用面向对象的基础》有讲过，

> 在设计中思考什么应该变化，并封装会发生变化的概念。

**软件架构的精髓：找到变化，封装变化。**

业务千变万化，没有固定的编码答案，千万不要硬套设计模式。无论选择哪一种设计模式，尽量要能满足`SOLID`原则，自我 review 是否满足业务的持续扩展性。有句话说的好，“不论白猫黑猫，能抓老鼠就是好猫。”