> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODI5Njc2MA==&mid=2655882667&idx=2&sn=de587d1377d9f94577f7be29c4b931fa&chksm=bd75c07c8a02496a881b6667043cb96c86327434c435e6356e9f7497d16d4d9d46be0451332c&mpshare=1&scene=1&srcid=1002oj0fNlWzfYv7hImlyxuo&sharer_sharetime=1664717639824&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

> DDD 分为战略部分跟战术部分，相信大家都认同 DDD 的核心在战略而非战术。

在我看来并不是 MVC 的基础上增加领域层，使用充血模型，解耦基础服务，我的代码就符合 DDD 了。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6SlG4gLUichgR8iaYjPK3ZttOJch09cCxSSgq7E1eLAlSa4af2VQFice2g/640?wx_fmt=png)

**为什么要使用 DDD？**
===============

DDD 分为战略部分跟战术部分，相信大家都认同 DDD 的核心在战略而非战术。而战略方面的核心我认为在业务建模，领域划分、统一语言等都在为业务建模服务。

**为什么业务建模重要?**

### （1）以前的开发流程有什么问题？

先说结论，开发人员交付的程序对业务方，产品人员，测试人员来说就是一个黑盒子。除了开发人员自己，没人知道盒子里有什么。当新的需求加入来，需求方，产品人员，甚至测试人员都认为可行，开发人员却给出相反结论。

回顾一下以前的开发流程 大致可以归结为以下步骤：(开发跟测试人员最好能参与需求分析)

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naJOIaguiaiagnLiaLw0WWSdPj3lGrbpVdBa1xnA1RteCu3hHUgbHiaR6zMGrjSxwNdc9pnQhSoeKibMGHg/640?wx_fmt=png)

*   业务方描述抽象需求
    
*   产品将需求转化为可落地的产品 (需求具像化, PRD)
    
*   开发人员根据产品的 PRD 开发
    
*   测试人员根据产品的 PRD 测试
    
*   产品人员验收
    
*   业务方验收
    

依照经验来说, 在整个流程中开发人员是耗时最长的。与此同时测试人员可能在编写测试用例，而业务方跟产品人员在这段时间内是阻塞的。最终的程序质量靠测试人员来保障。

开发人员完成开发后:

*   测试人员关注测试用例是否通过。
    
*   产品人员确认展现出来的功能是否符合当初的 PRD。
    
*   业务方确认程序是否符合预期。
    

#### 举一个我开发项目的例子：一个审批系统

产品的 PRD 描述了一个三层模型：

流程实例, 流程节点, 审批任务。流程实例启动创建审批节点，审批节点触发审批任务，任务完成创建下一个节点...。

我是这样做的：

流程实例, 审批任务。流程实例启动创建 (一批) 审批任务, 任务被完成后创建后续任务或者流程结束。至于流程节点不存在，不是问题，从任务中提取信息虚拟一个出来。

第一版交付完成。

产品在第一版后追加需求，流程节点可以被非审批人评论。

我....

当时业务方，产品，测试都认为这是一个合理的需求。只有我一脸懵，因为我的程序中没有流程节点这个东西，需求又不能拒绝，无奈给出一个远超他们预期的开发计划。

gap 就出在 他们认为流程节点是一个确实存在的东西，而只有我知道这节点是虚拟的，没有标识，不能跟其他信息做关联。

### （2）业务建模怎么解决这个黑盒子问题？

DDD 引入了业务专家这个角色 (在我看来就是业务方，产品)。

*   假设业务专家听不懂 什么叫类，什么是方法，设计模式，他只知道他的业务，两方人马完全不在同一频道，这个时候就需要 “明确上下文”，“统一语言” 了。(不仅仅开发人员与业务专家达成了共识，也包含整个开发团队达成共识)
    
*   业务建模，用例分析法、事件风暴、四色建模等看看开始整上。最终达到划分领域，识别聚合的目的。
    
*   业务建模落地。开发人员开发过程中，应遵守已经建立的业务模型来编写代码。
    

至此终于实现了，业务专家可通过业务模型窥探到开发人员的代码实现。统一语言、业务模型在业务专家跟开发人员中间充当了沟通的桥梁。(有点像适配器)

当追加新的需求时，业务专家能合理评估需求的可行性。

### （3）让非开发人员参与到开发中

统一语言，业务建模，模型充血（OOP）。这一系列手段都是为了实现让非开发人员参与到开发中这一最终目的。与其说 DDD 是一种架构，不如认为他是指导开发的方法论。

好比盖房子，以前只要把房子交付了能住人就行。现在业务专家是设计师，业务模型是设计图，代码则是建材，程序员就是工地搬砖的，盖起来的房子得跟设计师给出的设计图一样。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6ojD8pHUQOsKpMfyhXp67maIRcFLyAdv0lgYYV3OI2g2zwpRHL6ljYg/640?wx_fmt=png)

**业务模型落地的问题？**
==============

这是一个让我纠结的问题。我感觉还没有找到符合我期望的答案。

业务模型具现到代码中，就是一个个聚合。这些聚合符合 OOP 的思想，通过聚合根，实体，值对象的组合来表达业务模型。

以网络上常见的 demo 为例：

```
//订单明细实体
public class OrderItemEntity {
  //id
  private Long id;
  //商品(值对象)
  private Product product;
  //数量(值对象)
  private Count count;
  // item总价(值对象)
  private ItemAmount itemAmt;
  
  public void modifyCount(Count count) {
    this.count = count;
    this.itemAmt = new ItemAmt(this.product.getPrice()*count.get());
  }
  
}

//订单聚合根
public class OrderAggregate {
  //聚合唯一标识
  private Long id;
  //订单号(值对象)
  private OrderNo orderNo;
  //总价 (值对象)
  private Amount amt;
  //订单明细
  private List<OrderItemEntity> items;
  
  public void modifyItemCount(Product product,Count count) {
    //找到商品
    this.items.stream().filter(product::equals).findAny().get();
    //修改数量 返回Item总价
    item.modifyCount(count);
    ItemAmount itemAmt = item.getItemAmt();
    //修改订单总价
    this.amt = new Amount(this.amt.get()+itemAmt.get());
  }
  
}

//要订单明细中 名称叫电脑的商品数量修改100

Product product = new Product("电脑");
Count count = new Count(100);
Amount amt = orderAggregate.modifyItemCount(product, count);


```

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6JkYWGCk2JF9nQdIF7cT1aXTCP3vYb9we91tAhMU02XkqjsicVwKuEWg/640?wx_fmt=png)

**理论与现实的矛盾**

从代码上可以看出这一段 1:n 关系的代码完全基于内存，非常的 OOP，也就是说，我们在获得 orderAggregate 时，已经加载整个聚合 (包括 List<OrderItemEntity>)，这里就隐含了一个条件内存无限大。

假设 OrderItemEntity 的量级是十万级，百万级，明显这段代码是不能上线的，理论与现实出现了矛盾。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6cugSvic8czqkVRmoRXljmS1qPjicIMu1iaHkaNzsBpxUTq9HsXhsfXZibg/640?wx_fmt=png)

**咨询了很多大佬 + 个人理解（以下方法为我自己命名）**

#### （1）模型提升法

#### _（无限套娃）_

大佬建议:“如果真有这种场景，就需要调整聚合，比如: 将 OrderItem 提升为 Order, Order 提升为 BatchOrder”

思路：

*   创建 BatchOrderAggregate,BatchOrderAggregate 持有 OrderEntity。
    
*   创建 OrderAggregate 持有部分 OrderItemEntity，通过分治的方式化整为零。
    

思考：

整体上符合业务模型，而且没有上限，即如果 BatchOrderAggregate 不能解决问题，那就祭出 BatchBatchOrderAggregate。

BatchBatchOrderAggregate 持有 BatchOrderEntity。

BatchOrderAggregate 持有 OrderEntity。

OrderAggregate 持有部分 OrderItemEntity。

#### （2）持有仓储法

#### _（隐藏了数据库查询，但是直觉上有点反模式）_

大佬建议：“聚合中构建索引，需要时再加载置换”

思路：

*   聚合根持有存储引用，需要时加载到内存中。
    
*   可以加入一层接口隔离。
    

```
public class OrderAggregate {
  //聚合唯一标识
  private Long id;
  //订单号(值对象)
  private OrderNo orderNo;
  //总价 (值对象)
  private Amount amt;
  //关联对象接口（接口实现在基础服务层，在实现中操作数据库）
  private OrderItemRel orderItemRel;

  
 public void modifyItemCount(Product product,Count count) {
    List<OrderItemEntity> items = orderItemRel.find(product);
    //找到商品
    OrderItemEntity item = items.stream().filter(product::equals).findAny().get();
    //修改数量 返回Item总价 这里有分支,item修改是否应该在modifyCount中持久化
    //1.modifyCount中持久化item那么数据库事务将被加载AppcationService层，容易产生大事务问题。
    //2.modifyCount中不持久化item 
    //2.1写入消息总线，当OrderAggregate通过Reponsitory持久化时刷出消息持久化
    //2.2 OrderAggregate中增加List<OrderItemEntity> items,modifyCount将item加入items。
    item.modifyCount(count);
    ItemAmount itemAmt = item.getItemAmt();
    //修改订单总价
    this.amt = new Amount(this.amt.get()+itemAmt.get());
    return this.amt;
  }
}


```

左右滑动查看完整代码