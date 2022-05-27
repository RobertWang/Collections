> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU4Mzc4NzI2NA==&mid=2247484735&idx=1&sn=596a2ea4f1ae3a8aeff602019c4bf6d8&chksm=fda2f350cad57a466b603b14f96f859260112efaeeb8e56f0a7bbeb591e6b47c6aa3e01adf8b&cur_album_id=1443939833554272257&scene=189#wechat_redirect)

  

  

点击上方 “蓝字” 关注本公众号

> 原文地址：https://felixgerschau.com/javascript-memory-management/#memory-life-cycle
> 
> 原文标题：JavaScript's Memory Management Explained

大多数时候，作为一个 JavaScript 开发人员，您可能不知道任何关于内存管理的知识。毕竟，JavaScript 引擎会为您处理这些问题。

不过，在某个时刻，您会遇到一些问题，比如内存泄漏，只有知道内存分配是如何工作的，才能解决这些问题。

在本文中，我将向你介绍内存分配和垃圾回收的工作原理，以及如何避免一些常见的内存泄漏。

#### 内存生命周期

在 JavaScript 中，当我们创建变量、函数或任何你能想到的东西时，JS 引擎会为此分配内存，并在不再需要时释放它。

分配内存是在内存中占用空间的过程，而释放内存是释放空间，准备用于其他用途。

每当我们分配一个变量或创建一个函数时，它的内存总是经过以下相同的阶段：

![](https://mmbiz.qpic.cn/mmbiz_png/R7oib8BJQz2jn4EJbwDOkLX7QzSj3V4dFeAxfwDWzZatxrBdric7cyYgWctKSicpTb8LrScVElXcwADMZ7lYOq5KQ/640?wx_fmt=png)

*   分配内存
    

JavaScript 为我们处理这个问题：它为我们创建的对象分配我们需要的内存。

*   使用内存
    

使用内存是我们在代码中显式地做的事情：读写内存只不过是读写变量。

*   释放内存
    

这个步骤也由 JavaScript 引擎处理。一旦分配的内存被释放，它就可以用于新的用途。

> 内存管理上下文中的 “对象” 不仅包括 JS 对象，还包括函数和函数作用域。

#### 内存堆和堆栈

我们现在知道，对于我们在 JavaScript 中定义的所有内容，引擎会分配内存并在不再需要时释放内存。

我想到的下一个问题是：这个数据要放在哪里？

##### 堆栈：静态内存分配

由于引擎知道大小不会改变，它将为每个值分配固定数量的内存。

在执行之前分配内存的过程称为**静态内存分配**。

因为引擎为这些值分配了固定数量的内存，**所以原始值的大小是有限制的**。

这些值和整个堆栈的限制因浏览器而异。

##### 堆：动态内存分配

堆是存储数据的不同空间，JavaScript 在其中存储对象和函数。

与堆栈不同，引擎不会为这些对象分配固定数量的内存。相反，将根据需要分配更多空间。

以这种方式分配内存也称为**动态内存分配**。

总结一下，

堆栈：原始值和引用；大小在编译时已知；分配固定数量的内存

堆：对象和函数；大小在运行时已知；每个对象没有限制

#### 例子

我们来看几个例子：

```
const person = {
  name: 'John',
  age: 24,
};
```

JS 在堆中为这个对象分配内存。实际值仍然是原始值，这就是为什么它们存储在堆栈中。

```
const hobbies = ['hiking', 'reading'];
```

数组也是对象，这就是它们存储在堆中的原因。

#### JS 中的引用

所有变量首先指向堆栈。如果它是非基元值，则堆栈包含对堆中对象的引用。

堆的内存没有以任何特定的方式排序，这就是为什么我们需要在堆栈中保留对它的引用。您可以将引用视为地址，而堆中的对象则视为这些地址所属的房屋。

> 记住 JavaScript 在堆中存储对象和函数。原始值和引用存储在堆栈中。

![](https://mmbiz.qpic.cn/mmbiz_png/R7oib8BJQz2jn4EJbwDOkLX7QzSj3V4dFNp0ARuno1TBBbLsIAOJeM5m9FxD7f7dY4a9QT02BZ8XMgSukMIwlNA/640?wx_fmt=png)

在这幅图中，我们可以观察到不同的值是如何存储的。注意 person 和 newPerson 如何指向同一个对象。

举例说明：

这将在堆中创建一个新对象，并在堆栈中创建对它的引用。

#### 垃圾回收

我们现在知道 JavaScript 是如何为所有类型的对象分配内存的，但是如果我们还记得内存的生命周期，就缺少最后一步：释放内存。

就像内存分配一样，JavaScript 引擎也为我们处理这一步。更具体地说，垃圾收集器负责处理这个问题。

一旦 JavaScript 引擎识别出某个给定的变量或函数不再需要，它就会释放它所占用的内存。

主要的问题是，是否仍然需要一些内存是一个**不可解问题**，这意味着不可能有一个算法能够在内存过时时收集所有不再需要的内存。

有些算法对这个问题提供了很好的近似值。在本节中，我将讨论最常用的方法：引用计数垃圾回收和标记和清除算法。

##### 引用计数垃圾回收

这是最简单的近似值。它收集没有被引用的对象。

让我们看看下面的例子。

![](https://mmbiz.qpic.cn/mmbiz_gif/R7oib8BJQz2jn4EJbwDOkLX7QzSj3V4dF6mX3E0TmZ033ibg7mQSqFx30rX90Drgm48bJ4SJkLiaxTcw3u26wrarg/640?wx_fmt=gif)

请注意，在最后一帧中，只有 hobbies 才保留在堆中，因为它是最终有引用的对象。

##### 环

这个算法的问题是它没有考虑循环引用。当一个或多个对象相互引用，但不能再通过代码访问它们时，就会发生这种情况。

```
let name='John'；//为字符串分配内存

const age=24；//为数字分配内存

name='John Doe'；//为新字符串分配内存

const firstName = name.slice(0,4); //为新字符串分配内存
```

![](https://mmbiz.qpic.cn/mmbiz_png/R7oib8BJQz2jn4EJbwDOkLX7QzSj3V4dFzk3EibpopSEgkmUp8ppHhV9ib4X0ExTmvUIiabCFEv5ick0iaibE9JajSlLw/640?wx_fmt=png)

因为子对象和父对象相互引用，所以算法不会释放分配的内存。我们再也无法接近这两个对象了。

将它们设置为 null 不会使引用计数算法认识到它们不能再使用，因为它们都有传入的引用。

##### 清除标记算法

标记和扫描算法有一个循环依赖的解决方案。它不是简单地计算对给定对象的引用，而是检测它们是否可以从根对象访问。

浏览器中的根是 window 对象，而在 NodeJS 中是 global 对象。

![](https://mmbiz.qpic.cn/mmbiz_png/R7oib8BJQz2jn4EJbwDOkLX7QzSj3V4dFIleicHU1Q6VIcNsnbY4ZXVpzcRMEnRvYVtf3aeJuEmOPjZn8u4RXRfg/640?wx_fmt=png)

该算法将不可访问的对象标记为垃圾，然后对其进行清理（收集）。不会收集根对象。

这样，循环依赖就不再是问题了。在前面的示例中，不能从根访问 dad 或 son 对象。因此，它们都将被标记为垃圾并被收集。

自 2012 年以来，该算法在所有现代浏览器中都得到了实现。改进的只是性能和实现，而不是算法的核心思想本身。

##### 权衡

自动垃圾回收使我们能够专注于构建应用程序，而不是浪费时间进行内存管理。然而，我们需要注意一些权衡。

###### **内存使用**

由于算法无法知道何时不再需要内存，JavaScript 应用程序可能会使用比实际需要更多的内存。

即使对象被标记为垃圾，也要由垃圾收集器决定何时以及是否收集分配的内存。

如果你需要你的应用程序尽可能的节省内存，你最好使用较低级别的语言。但请记住，这是有其自身的一套权衡。

###### **性能**

为我们收集垃圾的算法通常定期运行以清理未使用的对象。

问题是，我们，开发者，不知道什么时候会发生。收集大量垃圾或频繁收集垃圾可能会影响性能，因为这样做需要一定的计算能力。

然而，对于用户或开发人员来说，影响通常是不明显的。

#### 内存泄露

有了关于内存管理的所有知识，让我们来看看最常见的内存泄漏。

你会发现，如果你了解幕后的情况，这些都是很容易避免的。

##### 全局变量

在全局变量中存储数据可能是最常见的内存泄漏类型。

在浏览器的 JavaScript 中，如果省略 var、const 或 let，变量将附加到 window 对象。

```
const person = {
  name: 'John',
  age: 24,
};
```

通过在严格模式下运行代码来避免这种情况。

除了意外地将变量添加到根目录之外，在许多情况下，您可能会故意这样做。

您当然可以使用全局变量，但请确保在不再需要数据时释放空间。

要释放内存，请将全局变量指定为 null。

```
let son = {
  name: 'John',
};

let dad = {
  name: 'Johnson',
}

son.dad = dad;
dad.son = son;

son = null;
dad = null;
```

##### 被遗忘的定时器和回调函数

忘记计时器和回调会增加应用程序的内存使用率。特别是在单页应用程序（spa）中，动态添加事件侦听器和回调时必须小心。

###### **被遗忘的定时器**

```
users = getUsers();
```

上面的代码每 2 秒运行一次函数。如果您的项目中有这样的代码，您可能不需要一直运行此代码。

只要 interval 没有取消，interval 中引用的对象就不会被垃圾回收。

一旦不再需要，一定要清除 interval。

```
window.users = null;
```

这在 Spa 中尤其重要。即使离开需要此 interval 的页面，它仍将在后台运行。

**被遗忘的回调函数**

假设您将 onclick 侦听器添加到按钮中，稍后将删除该侦听器。

旧的浏览器无法回收侦听器，但现在，这不再是一个问题。

不过，当您不再需要事件侦听器时，最好删除它们：

```
const object = {};
const intervalId = setInterval(function() {
  // everything used in here can't be collected
  // until the interval is cleared
  doSomething(object);
}, 2000);
```

**DOM 外引用**  

这种内存泄漏与前面的类似：在 JavaScript 中存储 DOM 元素时会发生这种情况。

```
clearInterval(intervalId);
```

当您删除这些元素中的任何一个时，您可能需要确保也从数组中删除这个元素。

否则，无法收集这些 DOM 元素。

```
const element = document.getElementById('button');
const onClick = () => alert('hi');

element.addEventListener('click', onClick);

element.removeEventListener('click', onClick);
element.parentNode.removeChild(element);
```

由于每个 DOM 元素也保留对其父节点的引用，因此将阻止垃圾回收器收集元素的父节点和子节点。

#### 结论

在本文中，我总结了 JavaScript 内存管理的核心概念。

写这篇文章帮助我理清了一些我不完全理解的概念，我希望这篇文章能很好地概述 JavaScript 中内存管理的工作原理。

符合预期的 CoyPan 发起了一个读者讨论 欢迎交流~

关于内存管理，推荐阅读本公众号之前的一些文章：

[从 JS 中的内存管理说起 —— JS 中的弱引用](http://mp.weixin.qq.com/s?__biz=MzU4Mzc4NzI2NA==&mid=2247484610&idx=1&sn=bb4c61a68ff3fc05803da278e2fd8dce&chksm=fda2f2adcad57bbb720899275d413a164c62d40baf835f4e1886338326ad49cb556a748d65a1&scene=21#wechat_redirect)  

[V8 引擎的内存管理](http://mp.weixin.qq.com/s?__biz=MzU4Mzc4NzI2NA==&mid=2247484379&idx=1&sn=afa1c92f995f3228ed597d7c45ed618b&chksm=fda2f5b4cad57ca249a7258d2b967d323effe941a01d01f761b1e6af907d1ab1c1d6bc851204&scene=21#wechat_redirect)