> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU5NTA0OTA4MA==&mid=2247488593&idx=1&sn=014d9f27ccee0ebf36cfee1da4cd6715&chksm=fe76bb46c90132502a3f174d0d5f5bb58ccdd70f31f78298e2a24d6f53f31dfce389144451e6&scene=132#wechat_redirect)

  

### 观点一（灵剑）：

![](https://mmbiz.qpic.cn/mmbiz_png/OKUeiaP72uRyWIkUGPzibCuTdM5s2G2eaP66Ah5sKb0lJNRET4nibQJMQhDH7Ihy5ERYFCpLyCXAH74UdJjrToGdQ/640?wx_fmt=png)

#### 1. 提前 return

```
if (condition) {
 // do something
} else {
  return xxx;
}
```

其实，每次看到上面这种代码，我都心里抓痒，完全可以先判断`!condition`，干掉 else。

```
if (!condition) {
  return xxx;

} 
// do something
```

#### 2. 策略模式

```
if (strategy.equals("fast")) {
  // 快速执行
} else if (strategy.equals("normal")) {
  // 正常执行
} else if (strategy.equals("smooth")) {
  // 平滑执行
} else if (strategy.equals("slow")) {
  // 慢慢执行
}
```

看上面代码，有 4 种策略，有两种优化方案。

#### 2.1 多态

```
interface Strategy {
  void run() throws Exception;
}

class FastStrategy implements Strategy {
    @Override
    void run() throws Exception {
        // 快速执行逻辑
    }
}

class NormalStrategy implements Strategy {
    @Override
    void run() throws Exception {
        // 正常执行逻辑
    }
}

class SmoothStrategy implements Strategy {
    @Override
    void run() throws Exception {
        // 平滑执行逻辑
    }
}

class SlowStrategy implements Strategy {
    @Override
    void run() throws Exception {
        // 慢速执行逻辑
    }
}
```

```
Strategy strategy = map.get(param);
strategy.run();
```

#### 2.2 枚举

发现很多同学不知道在枚举中可以定义方法，这里定义一个表示状态的枚举，另外可以实现一个 run 方法。

```
public enum Status {
    NEW(0) {
      @Override
      void run() {
        //do something  
      }
    },
    RUNNABLE(1) {
      @Override
       void run() {
         //do something  
      }
    };

    public int statusCode;

    abstract void run();

    Status(int statusCode){
        this.statusCode = statusCode;
    }
}
```

重新定义策略枚举

```
public enum Strategy {
    FAST {
      @Override
      void run() {
        //do something  
      }
    },
    NORMAL {
      @Override
       void run() {
         //do something  
      }
    },

    SMOOTH {
      @Override
       void run() {
         //do something  
      }
    },

    SLOW {
      @Override
       void run() {
         //do something  
      }
    };
    abstract void run();
}
```

```
Strategy strategy = Strategy.valueOf(param);
strategy.run();
```

#### 3. 学会使用 Optional

Optional 主要用于非空判断，由于是 jdk8 新特性，所以使用的不是特别多，但是用起来真的爽。

使用之前：

```
if (user == null) {
    //do action 1
} else {
    //do action2
}
```

如果登录用户为空，执行 action1，否则执行 action 2，使用 Optional 优化之后，让非空校验更加优雅，间接的减少 if 操作

```
Optional<User> userOptional = Optional.ofNullable(user);
userOptional.map(action1).orElse(action2);
```

#### 4. 数组小技巧

来自 google 解释，这是一种编程模式，叫做表驱动法，本质是从表里查询信息来代替逻辑语句，比如有这么一个场景，通过月份来获取当月的天数，仅作为案例演示，数据并不严谨。

```
int getDays(int month){
    if (month == 1)  return 31;
    if (month == 2)  return 29;
    if (month == 3)  return 31;
    if (month == 4)  return 30;
    if (month == 5)  return 31;
    if (month == 6)  return 30;
    if (month == 7)  return 31;
    if (month == 8)  return 31;
    if (month == 9)  return 30;
    if (month == 10)  return 31;
    if (month == 11)  return 30;
    if (month == 12)  return 31;
}
```

优化后的代码

```
int monthDays[12] = {31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
int getDays(int month){
    return monthDays[--month];
}
```

#### 结束

if else 作为每种编程语言都不可或缺的条件语句，在编程时会大量的用到。一般建议嵌套不要超过三层，如果一段代码存在过多的 if else 嵌套，代码的可读性就会急速下降，后期维护难度也大大提高。

### 观点二（IT 技术控）：

不要去过度关注 if/else 的层数，而要关注接口语义是否足够清晰；单纯减少 if/else 的层数，然后拆出一堆 do_logic1, do_logic2… 这样的接口是毫无帮助的。

任何一个接口的执行过程都可以表示为：输入 + 内部状态 -> 输出这样的形式，我们分以下几种情况来讨论：

输入、内部状态、输出都很简单，但中间逻辑复杂。比如说一个精心优化过的数值计算程序，可能需要根据输入在不同的取值范围采取不同的策略，还有很多逻辑用来处理会引发问题（比如除 0）的边界值，这种情况下 if/else 数量多是难以避免的，根据步骤拆分出一些内部方法有一定帮助，但也不能完全解决问题。这种情况下最好的做法是写一篇详细的文档，从最原始的数学模型开始，然后表明什么情况下采取什么样的计算策略，策略如何推导，知道得到代码中使用的具体形式，然后给整个方法加上注释附上文档地址，并且在每个分支的地方加上注释指明对应到文档中哪个公式。这种情况下虽然方法很复杂，但是语义是清晰的，如果不修改实现的话理解语义就行了，如果要修改实现那么需要参考对照文档中的公式。

输入过于复杂，比如输入带有一堆不同的参数，或者有各种奇怪的 flag，每个 flag 有不同作用。这种情况下首先需要提高接口的抽象层次：如果接口有多个不同作用，需要拆分成不同接口；如果接口内部根据不同参数进不同分支，需要将这些参数和对应分支包在 Adapter 里，使用参数的地方改写成 Adapter 的接口，根据传入的 Adapter 类型不同进入不同的实现；如果接口内部有复杂的参数转换关系，需要改写成查找表。这种情况下的主要问题是接口本身抽象的有问题，有更清晰的抽象之后，实现也自然没有那么多 if/else 了。

输出过于复杂，为了省事一个过程计算出了太多东西，又为了性能加了一堆 flag 控制是否计算之类。这种情况下需要果断将方法拆分成多个不同方法，每个方法只返回自己需要的内容。如果不同计算之间有共用的内部结果呢？如果这个内部结果计算并不形成瓶颈，只要提取出内部方法然后在不同过程中分别调用即可；如果希望避免重复计算，可以增加一个额外的 cache 对象作为参数，cache 内容对用户不透明，用户只保证相同输入使用同一个 cache 对象即可，在计算中将中间结果保存到 cache 中，下次计算前先检查有没有已经得到的结果，就可以避免重复计算了。

内部状态过于复杂。首先检查状态设置的是否合理，是不是有一些本来应该作为输入参数的东西被放到了内部状态中（比如用来隐式地在两个不同方法调用之间传递参数）？其次，这些状态分别控制哪些方面，是否可以分组然后实现到不同的 StateManager 里面？第三，画出状态转移图，尝试将内部状态分成单层分支，然后分别实现到 on_xxx_state 这样的方法里面，然后通过单层的 switch 或者查找表来调用。

其实通常需要优化的都是整体接口抽象，而不是单个接口的实现，单个接口实现不清晰通常是因为接口实现和需求不同构造成的。