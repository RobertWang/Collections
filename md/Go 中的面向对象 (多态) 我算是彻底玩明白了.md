> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/n8Hzf9fs2wiNGH1vNwIsrg)

大家伙，我是 Mandy。

上一篇，我们分享了[在 Go 中是如何实现面向对象](https://mp.weixin.qq.com/s?__biz=MzI4NzE2MDI5NA==&mid=2650286542&idx=1&sn=d4964d28a4b40ed65813cd092d45717f&scene=21#wechat_redirect)，文章中对面向对象的三大特性中的继承、封装，做了一个理论和实践的总结，这一篇继续分享关于另外一个特性，那就是多态。

认识多态
----

老规矩，在代码实践之前，先对基础知识做一个普及。

1、面向对象中的多态（Polymorphism）是指一个对象可以具有多种不同的形态或表现方式。简单来说，就是同一个类型的对象，在不同的上下文中表现出不同的行为。多态性是面向对象的三大特性之一（封装、继承、多态）。

2、在多态中，父类的引用可以指向子类的对象，通过父类的引用调用子类的方法。这样可以实现代码的灵活性和扩展性，可以根据具体的对象类型调用相应的方法，无需关心对象的具体类型。

3、通过多态性，可以通过统一的接口来处理不同的对象，实现代码的简洁性和可维护性。多态性提供了一种抽象的方式来处理对象的不同行为，使得代码更具灵活性和可扩展性。

总结一句，就是同一个方法在不同的对象实例中，可以有不同的行为。这里简单用 PHP 举一个案例。

在 PHP 中实现多态需要遵循以下几个步骤：

1、创建父类（基类）：定义一个包含通用方法和属性的父类。

```
class Animal {
  public function makeSound() {
    echo "Animal makes sound.";
  }
}


```

2、创建子类（派生类）：继承父类，并添加自己特定的方法和属性。

```
class Dog extends Animal {
  public function makeSound() {
    echo "Dog barks.";
  }
}

class Cat extends Animal {
  public function makeSound() {
    echo "Cat meows.";
  }
}


```

3、创建对象并调用方法：通过父类的引用来实例化子类对象，可以根据具体的对象类型调用相应的方法。

```
$animal1 = new Dog();
$animal1->makeSound(); // Output: Dog barks.

$animal2 = new Cat();
$animal2->makeSound(); // Output: Cat meows.


```

通过以上步骤，我们可以实现多态性，即相同父类类型的对象（例如 Animal），在不同的对象实例下调用相应的子类方法（例如 Dog 或 Cat）。这样就实现了多态的效果。

Go 语言实现
-------

因为 Go 中没有面向对象的概念，但根据多态的定义和特点，我们可以使用 Go 中的 interface 来实现多态。

```
package main

import "fmt"

type Animal interface {
 run()
}

type Turtle struct {
}

func (t Turtle) run() {
 fmt.Println("乌龟爬行很慢")
}

type Rabbit struct {
}

func (r Rabbit) run() {
 fmt.Println("兔子跑步很快")
}

func main() {
 var animal Animal
 // 实例化一个Turtle对象
 animal = &Turtle{}
 animal.run()

 // 实例化一个Rabbit对象
 animal = &Rabbit{}
 animal.run()
}


```

1、首先我们定义了一个 Animal 的接口，并在接口中定义了一个约束，就是 run() 方法。

2、接着我们定义了 Turtle 和 Rabbit 两个结构体，并分别定义了一个 run() 方法。

3、此时两个结构体隐式的实现了 Animal 接口中的方法。

4、根据多态的特性，两个结构体中的方法，都可以具备自己的行为。我们在两个方法中分别打印了内容，此时能够打印出不同的内容。不同的内容就可以理解为不同的行为，当然你也可以在这个方法中做其他的操作。

5、在 main() 方法中，创建一个 Animal 的变量，并通过不同的结构体实例，调用相同的方法名，最终输出不同的内容。

实战案例
----

上面对多态有了一定的了解，接着列举一个实战的案例。

在系统支付，一般我们会对接微信和支付宝这样的平台，大致的流程就是，创建订单记录 -> 组装支付参数 -> 发起支付请求 -> 支付平台进行回调通知 -> 修改订单支付状态。

基于这样的逻辑，我们使用一个 Pay 接口，定义三个方法。

```
type Pay interface {
 createOrder() // 创建订单
 createPay()   // 常见支付参数
 notifyPay()   // 回调通知
}


```

接着定义具体的实现类，就是微信支付和支付宝支付。

1、定义一个微信支付，用来实现接口中的三个方法。

```
type WeChat struct {
 // 微信支付方式
}

func (a WeChat) createOrder() {
 fmt.Println("我是微信支付，现在我正在创建订单数据，用于记录到数据库中。")
}

func (a WeChat) createPay() {
 fmt.Println("我是微信支付，现在我正在创建支付数据，用于向微信发起支付请求使用。")
}

func (a WeChat) notifyPay() {
 fmt.Println("我是微信支付，现在我正在接受微信通知的参数，用于修改用户订单支付状态。")
}


```

2、定义一个支付宝支付，用来实现接口中的三个方法。

```
type Ali struct {
 // 支付宝支付方式
}

func (a Ali) createOrder() {
 fmt.Println("我是支付宝支付，现在我正在创建订单数据，用于记录到数据库中。")
}

func (a Ali) createPay() {
 fmt.Println("我是支付宝支付，现在我正在创建支付数据，用于向支付宝发起支付请求使用。")
}

func (a Ali) notifyPay() {
 fmt.Println("我是支付宝支付，现在我正在接受支付宝通知的参数，用于修改用户订单支付状态。")
}


```

接下来，就来进行实际的订单操作。

1、假设当前的支付渠道使用的是微信支付。

```
func main() {
 var pay Pay
 pay = &WeChat{}
 pay.createOrder()
 pay.createPay()
 pay.notifyPay()
}


```

最终输出的结果为：

```
我是微信支付，现在我正在创建订单数据，用于记录到数据库中。
我是微信支付，现在我正在创建支付数据，用于向微信发起支付请求使用。
我是微信支付，现在我正在接受微信通知的参数，用于修改用户订单支付状态。


```

2、假设当前的支付渠道使用的是支付宝支付。

```
func main() {
 var pay Pay
 pay = &Ali{}
 pay.createOrder()
 pay.createPay()
 pay.notifyPay()
}


```

最终输出结果为：

```
我是支付宝支付，现在我正在创建订单数据，用于记录到数据库中。
我是支付宝支付，现在我正在创建支付数据，用于向支付宝发起支付请求使用。
我是支付宝支付，现在我正在接受支付宝通知的参数，用于修改用户订单支付状态。


```

到此，在 Go 中实现面向对象以及三大特性 (封装、继承和多态) 就给大家分享完毕。