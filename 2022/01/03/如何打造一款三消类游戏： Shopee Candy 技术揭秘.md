> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/o6soLy9J07F4Vc2__ZkSCw)

> 样式繁多的 “消消乐” 游戏想必大家都不陌生，通关秘籍就是将三个或更多相同的元素配对消除，通常我们称这类游戏为 “三消” 游戏。Shopee 购物平台内嵌的三消游戏 Shopee Candy 也受到了不少用户的喜爱，这篇文章将带你从项目起源、游戏架构和项目工具集等方面了解如何打造一款这样的三消小游戏。

1. 项目起源
=======

1.1 游戏简介
--------

Shopee Candy 是一款面向多地区市场的三消类休闲 H5 游戏，用户可以在游戏中获得 Shopee Coins、商家购物券等优惠奖励，既可以增强用户粘性，激励用户消费，也可以为商家引流。同时，分享领奖、好友排行榜等机制增强了游戏的社交性，起到了为平台拉新的作用。此外，H5 游戏简单、轻量、高适配的特性与 Shopee 用户的使用习惯非常契合，可以即点即玩，参与门槛低的同时兼顾了趣味性。

2020 年 6 月，Shopee Candy 在 Shopee 全市场上线，推出 iOS 和安卓正式版本。自上线以来，Shopee Candy 在日常和大促活动中表现耀眼，用户的活跃度和在线时长屡创新高。

![](https://mmbiz.qpic.cn/mmbiz_gif/euN1ic17Jn04wiarJblSEQmZMKT43BX6zC1bceP2L25icAYwcIm2BrOvicm6KG5KFCnA2zZhA5D8w3DnweIuKXiclZA/640?wx_fmt=gif)

Shopee Candy 在风格上使用了缤纷可爱的糖果元素主题；玩法上主要是在限制操作步数的同时，设定收集元素数量作为通关条件。关卡内，用户通过交换相邻糖果，使三个或以上相同颜色的糖果连接起来，便可触发消除；根据消除糖果的数量和形状还可以生成有特殊消除能力的道具元素；模式上有关卡模式及无尽模式等。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8HHYhrchojpkXJg4pEiattI8jZ7RXrfoD6HqydzTZnamtto5tenANcE3A/640?wx_fmt=png)

1.2 项目简介
========

随着项目不断发展，Shopee Candy 的功能迭代可以被清晰地划分为四类。首先是各类业务功能模块（道具商城、任务系统和兑换商店等）；其次是负责消除逻辑算法，计算分数、关卡进度的算法库（Algorithm SDK）和负责消除效果的动画系统；最后是服务游戏的各种工具，包括解放设计关卡生产力的 Map Editor 关卡编辑器，可以量化关卡难度的 Score Runner 跑分器以及能进行操作复盘的 Replayer 回放工具等。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8HVOcNrgYSTCw7Pllehvp3oEKZvB5YgVKwLns7nbdhCc4Y6UicG8KxuicA/640?wx_fmt=png)

2. 游戏架构
=======

2.1 算法库（Algorithm SDK）
----------------------

作为一款元素消除种类丰富的三消游戏，Shopee Candy 的算法部分非常重要和复杂。早在项目之初，算法和动画是耦合在一起的。随着产品的上线和消除种类的增加，我们慢慢发现项目维护成本越来越高；同时算法本身是独立的，跟动画和业务没有依赖，所以将算法部分抽离了出来。

### 2.1.1 Algorithm SDK 实现

Algorithm SDK 的实现主要有三大部分：Map、Operator 以及 Logic Processing。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8H2WjEOUvABv9TicSiaY2icbBrIialKzGKumAHHpFXhcBwFOianLlHiaNw7slg/640?wx_fmt=png)

**地图（Map）**

Map 管理了游戏关卡中的地图对象和元素对象。我们根据每个元素的特性对元素进行了上、中、下三层的管理，这种元素分层的架构模式能够满足不同特效新元素的加入。每一层元素相互制约、相互影响，在消除流程中相辅相成，完成游戏中华丽的消除效果。

```
export default class Grid {
  public middle: CellModel;
  public upper: Upper;
  public bottom: Bottom;

  constructor(info: ICellInfo) {
    const { type } = info;

    this.upper = new Upper(/* ... */);
    this.middle = new CellModel(/* ... */);
    this.bottom = new Bottom(/* ... */);
  }
}
```

**操作（Operator）**

Operator 对算法的所有可行操作进行管理，是整个 Algorithm SDK 与外部通信的桥梁，负责接收外部的交换、双击等操作信号，进行相应算法的调用。在 Shopee Candy 主流程的调用中，Operator 会收集动画数据，并以此与动画系统进行通信。

```
// 元素交换
export function exchange(startPos, endPos): IAnimationData {
  // ... logic process
  // returns animation data
}

// 元素双击
export function doubleClick(pos): IAnimationData {
  // ... logic process
  // returns animation data
}
```

**逻辑运算（Logic Processing）**

Algorithm 对 Algorithm SDK 的所有逻辑运算进行管理，包括：开局有解保证、有解检测、消除、掉落等，是整个 Algorithm SDK 的核心。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06HU8ge0icdE4Gp2ANCGc2XEabkBoAVoEIpxyVvKDmZT3gXWfDGqP7ddCKtuticZaSHWRVLJNtbzibhA/640?wx_fmt=png)

流程中元素消除多次循环时，可能会出现逻辑执行用时过长的问题，导致用户操作时丢帧。为了避免这种情况，我们将逻辑运算分成了多段，让计算异步化，提前将数据发送到动画执行，后续操作分别在不同帧中完成。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8H0oJEicIRDlAEp12ldYuU9EzOaEw0rWP630IlSbEVFSmB2QGrjmjStOQ/640?wx_fmt=png)

### 2.1.2 Algorithm SDK 单元测试

在实现上我们已经做到了尽可能的分离和解耦，但是对于庞大的算法库来说，光是常规的 Code Review 远远不够，前端测试就显得非常重要了。

**单元测试介绍**

很多人说前端测试浪费时间且收效甚微，常规的前端业务确实会经常变动，包括 Shopee Candy 的业务视图也是经常变化。但是得益于算法库的分离和独立，我们可以对它进行无 UI、纯逻辑的单元测试。

**单元测试应用**

人工测试只能保证操作后代码不报错以及布局不错乱，但无法发现消除元素数量或分数不正确等多种情况。项目中用户的一次移动或者双击的操作，控制同样的布局下，最后得出相同的结果。这个结果包括了最终地图的数据、用户获得的分数、收集或者销毁元素的数量等，保证在多次迭代中的稳定性。

```
describe('BOMB', () => {
    it('Exchange each other should be the same crush', () => {
      const source = { row: 0, col: 3 };
      const target = { row: 1, col: 3 };
      const wrapper = mapDecorator(operator);
      const data1 = wrapper({ key: CRUSH_TYPE.BOMB }, source, target);
      const data2 = wrapper({ key: CRUSH_TYPE.BOMB }, target, source);
      // 地图比较
      expect(JSON.stringify(data1.map)).equal(JSON.stringify(data2.map)).equal('xxx');
      // 分数比较
      expect(data1.score).equal(data2.score).equal(150);
      // 步数比较
      expect(data1.passStep).equal(data2.passStep).equal(14);
    });
});
```

2.2 动画系统（Animation）
-------------------

动画与算法分离后，动画单独作为一个系统，这样有以下优点：

### 2.2.1 方案设计

分离动画系统后，需要与算法建立通信机制，来保证算法执行的消除结果有对应的动画播放。通信的实现方式如下：

*   建立事件机制，算法与动画通过事件进行相互通信；
    
*   定义动画数据结构，通过定义不同的动画类型来区分动画，例如消除和下落动画，同时定义完整的动画信息，动画系统解析后播放对应动画。
    

针对动画的播放，我们引入了一套**「**动画队列**」**的流程。将算法解析后的动画数据添加到队列中，递归播放队列，直至队列为空，结束动画播放。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8HThnKLeGDeibWoqD1FV0DVhAaZPZyvOpTCd6oyWwO9jPqbgy2nrxKhGw/640?wx_fmt=png)

从动画队列中播放单个动画时，为了确保各个元素动画的播放彼此之间不相互影响，动画系统采用**「**策略模式**」**进行设计，根据动画类型执行不同的消除策略，将元素的动画**「**内聚**」**到各自的策略方法中。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8HMiaJOzibro9icZUZ1mWP252k455IpQcHTUUvMGfkibgNLSZicWo6tntFUSg/640?wx_fmt=png)

```
//策略配置
const animStrategyCfg: Map<TElementType, IElementStrategy> = new Map([
    [AElement, AStrategy],
    [BElement, BStrategy],
    [CElement, CStrategy],
]);

//获取策略
function getStrategy(elementType):IElementStrategy{
    return animStrategyCfg.get(elementType);
}

//执行策略
function executeStrategy(elementType){
    const strategy = getStrategy(elementType);
    return strategy.execute();
}
```

算法负责计算消除逻辑，动画系统负责播放对应动画，除了播放龙骨等特效，还会操作棋盘元素的大小、位置和显示。正常情况下动画播放结束后棋盘的状态和算法状态应该是一致的，但极少数情况下可能会由于设备性能等原因，造成时序、定时器异常等问题，进而导致两者状态不一致，比如元素不显示或位置错位等。

所以，动画结束后，需要**「**兜底逻辑**」**实时获取算法状态，校验修正棋盘状态，使之与其匹配，避免展示上的错误。同时，为了避免性能问题，这里并非全量校验修正，而是只针对容易出错的中层元素。

### 2.2.2 解决回调地狱

游戏引擎自带的动画库采用回调方式执行动画完成后的逻辑，在动画较为复杂的情况下，常常会出现回调嵌套的写法，使得逻辑难以理解和维护。为了解决回调地狱问题，我们在动画库的原型链上封装了 promise 方法，这样就可以使用 async/await 同步的写法。

```
function animation(callback: Function) {
    tweenA.call(() => {
        tweenB.call(() => {
            tweenC.call(() => {
                sleep.call(callback);
            });
        });
    });
}
```

```
async function animation() {
    await tweenA.promise();
    await tweenB.promise();
    await tweenC.promise();
    await sleep.promise();
}
```

上面展示了从回调写法改成同步写法的效果，可以看到同步写法更加直观，解决了回调地狱的问题，更易于代码的维护。

3. 项目工具集
========

前面介绍了 Shopee Candy 的算法库和动画系统，实际上我们团队还做了很多工具。

3.1 Map Editor
--------------

在 Shopee Candy 游戏建立之初，我们需要制作一个既能灵活配置关卡也能测试关卡可玩性与通关率的工具。

![](https://mmbiz.qpic.cn/mmbiz_gif/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8Hw7ibdEocIg91zIjsZkla7PibY0GltHddPPsksjTjBcjmPJuArJYtHlCg/640?wx_fmt=gif)

在日常的工作中，通过拖拽、键盘快捷键的方式可以快速配置关卡棋盘元素。

目前棋盘元素已发展到 30 多个，各个元素之间存在复杂的互斥共存规则：

*   共存：A 元素和 B 元素可以同时出现在一个格子上；
    
*   互斥：A 元素和 B 元素不能同时出现在一个格子上；
    
*   大互斥：A 元素和 B 元素不能同时出现在一个棋盘上。
    

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8HAOub3NOp7V0Gzk2PQt1RpMoAdHRZZtb65tma1M0b1AYn2qDwKrUxOg/640?wx_fmt=png)

面对这么多元素之间的关系，单纯依靠策划在配置关卡的时候人工处理互斥关系是不合适的，所以需要一个关系互斥表去维护元素之间的关系。我们通过 Map Editor 服务端拉取这个表格的数据，将它返回给 Web 端，在做好关系限制的同时给出一定的错误反馈。

3.2 Score Runner
----------------

在上线前期，遇到的问题之一就是关卡策划速度追不上用户通关的速度，经常出现用户催更关卡的情况。一款三消游戏动辄几千关的设计量，对于任何团队来说都是研发工作中的一个巨大难点。其中，策划关卡最头疼和耗时是关卡的难度测试和调整，每一次调整都要人工重复试玩多次，然后统计通关率。Score Runner 正是一个可以解决图中红色枯燥费时流程的工具。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8HQ90K0WZIfdpFRhPjiblQfCOicdGJ5oNvX0Z6dXlBD8nueUZGEBwWlJ8Q/640?wx_fmt=png)

**Score Runner 实现**

先来看其中一关的布局，场上可以消除的操作有多种可能性，如果把这些可能性看成一张网状结构的数据，模拟用户的操作无外乎就只有这些可能性。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8HtCL9mUEDevaRecZhPd1sX9VYVMwqkcSlAThYcIs8eTZDjwmZVT8icaw/640?wx_fmt=png)

那么，下一步操作呢？在这些可能性操作后面有不同的布局，就会有更多的不同的可能性。流程大致如下：

*   遍历地图获取每一种操作的可能性；
    
*   根据每一种操作，继续获取下一步的可能性直到最后一步（通过还是失败）；
    
*   得到所有的操作步骤，就能得出最多和最少通关分数。
    

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8HvCP8A6RRxc0jJUopn3kbLQU693faEeh1U2DPcpVlZ3stnhRL8Hn59Q/640?wx_fmt=png)

从图中可以看到，每一种可能性都能走到最后结束，但是实际上很多可能性用户是不会去操作的，比如得分少的，没有消除意义的操作等，对于通关率来说不科学，而且对于计算效率来说非常低下。

于是我们加入了**聪明度策略**，对可能性的消除的数据集做计算得出操作权重顺序。以最高权重的操作为最佳可能性，对当前庞大的可能性网状结构进行剪枝，得出更符合的通关率。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06HU8ge0icdE4Gp2ANCGc2XEABIyv4k0hwiaIRwkPpSalvmxh3gaQDyl2sB4RIHJ4BKAIw4iaZsxm52g/640?wx_fmt=png)

例：

<table data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)"><thead data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)"><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(29, 29, 29)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)|rgb(255, 250, 248)" data-style="font-size: 14px; border-right: 0.2px; border-left: 0.2px; color: rgb(77, 77, 77); min-width: 90px; background-color: rgb(255, 250, 248); border-bottom-color: rgb(255, 255, 255); border-top-width: 1px; border-top-color: rgb(255, 255, 255); border-top-left-radius: 8px; border-bottom-left-radius: 8px; text-align: center;">关卡</th><th data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(29, 29, 29)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)|rgb(255, 250, 248)" data-style="font-size: 14px; border-right: 0.2px; border-left: 0.2px; color: rgb(77, 77, 77); min-width: 90px; background-color: rgb(255, 250, 248); border-bottom-color: rgb(255, 255, 255); border-top-width: 1px; border-top-color: rgb(255, 255, 255); text-align: center;">平均通关率<br data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(29, 29, 29)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)|rgb(255, 250, 248)">（100 次）</th><th data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(29, 29, 29)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)|rgb(255, 250, 248)" data-style="font-size: 14px; border-right: 0.2px; border-left: 0.2px; color: rgb(77, 77, 77); min-width: 110px; background-color: rgb(255, 250, 248); border-bottom-color: rgb(255, 255, 255); border-top-width: 1px; border-top-color: rgb(255, 255, 255); border-top-right-radius: 8px; border-bottom-right-radius: 8px; text-align: center;">通关记录<br data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(29, 29, 29)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)|rgb(255, 250, 248)">（%）</th></tr></thead><tbody data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)"><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">341</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">65%</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">63/62/71</td></tr><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">342</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">68%</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;" class="">65/63/76</td></tr><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">343</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">60%</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">56/57/67</td></tr><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">344</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">60%</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">63/64/56</td></tr><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">345</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">47%</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">46/47/47</td></tr><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">346</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">51%</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">51/52/49</td></tr><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">347</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">50%</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">50/52/42</td></tr><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">348</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">47%</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(248, 248, 248)|rgb(255, 255, 255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; background-color: rgb(255, 255, 255); text-align: center;">51/38/51</td></tr><tr data-darkmode-color-16412222920595="rgb(163, 163, 163)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">349</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">63%</td><td data-darkmode-color-16412222920595="rgb(153, 153, 153)" data-darkmode-original-color-16412222920595="#fff|rgb(0,0,0)|rgb(77, 77, 77)" data-darkmode-bgcolor-16412222920595="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16412222920595="#fff|rgb(255,255,255)" data-style="font-size: 14px; border-width: 0.2px; border-style: initial; border-color: initial; color: rgb(77, 77, 77); min-width: 85px; text-align: center;">65/61/62</td></tr></tbody></table>

通过分析平均通关成功率，可以减少策划对于关卡难度的验证时间。

3.3 Replayer
------------

在 Score Runner 之后，我们使用 Replayer 对跑分过程进行回放，以验证跑分的正确性。

回放时，面临的首要问题就是保证每次随机的结果与跑分时是一致的。如何保证？

**随机种子**是这个问题的答案。随机种子是一种以随机数作为对象的，以真随机数（种子）为初始条件的随机数。简单来说就是设置固定的种子，输出的结果、顺序完全相同的伪随机方法。

![](https://mmbiz.qpic.cn/mmbiz_png/euN1ic17Jn06VEJRnUVPGj4c5JS8gzc8HDqbiaJOCtN9AvsqIkFGCxlicAA79u441VicLEa2UqHHfQDdG2pjsKpoCQ/640?wx_fmt=png)

为了接入随机种子，我们采用了新的随机数策略，该策略可以对随机种子进行设置，且我们每一次随机数都是基于上一次随机数结果作为种子计算得出的结果。

这样的策略保证了整一局游戏中的每一次随机数都被记录，每一次随机的结果可以随时拿到。

同时这一策略也有以下收获：

*   逻辑算法执行的可回溯性；
    
*   断线重连后随机数的可追溯性；
    
*   复盘线上用户整局游戏步骤的可行性；
    
*   极少数据存储量。
    

4. 未来规划
=======

本文从项目起源、游戏架构和项目工具集等方面介绍了 Shopee Candy 这款游戏。

“Rome wasn't built in a day”，经过 “需求——重构——上线” 的循环才能造就更加完善的项目，后续我们将从以下几方面对 Shopee Candy 进行完善和优化：

1）**针对新元素实现配置化开发，减少开发成本**

2）**针对低端设备提供更流畅的游戏体验**

Shopee Candy 是一款强游戏性项目，特别关注游戏性能和操作手感。由于市面上移动设备性能参差不齐，则更加需要关注低端设备的用户体验。后续会针对低端设备制定逻辑和视图性能的优化策略，为用户提供更加流畅的游戏体验。

3）**操作行为验证，避免作弊现象**

前端依靠混淆或者加密等手段增加了破解成本，但无法完全防范作弊行为。目前，项目正在开发操作行为验证服务，结合现有的 Replyer 功能对可疑结算行为进行二次操作校验，从而保障游戏的公平性。

4）**利用机器学习进行关卡设计**

目前，Shopee Candy 已经开发了上千关卡。为提高关卡配置效率，后续计划结合机器学习和 Score Runner 统计的通关率进行模型训练，仅需提供少量的人工配置即可自动生成关卡。

> **本文作者**
> 
> Shopee Games - Candy 前端团队。

> **加入我们**
> 
> 我们不仅会做一些游戏化的营销工具，而且还做真正的游戏！经营、养成、关卡、AR、PVP 等，多达几十种并不断增加。“好玩” 是我们做游戏的初衷和目标，因为我们相信，游戏能建立情感、拉近距离，给用户带来快乐，为平台创造价值。
> 
> 目前大量 iOS、前端、后端、测试、大数据开发岗位空缺中，感兴趣的同学可将简历发送至：vicky.zeng@shopee.com（邮件主题请注明：Shopee Games - 来自技术博客）。