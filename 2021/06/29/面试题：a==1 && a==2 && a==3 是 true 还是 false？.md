> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666555445&idx=2&sn=b96ecd049c476762e589a078f65ed742&chksm=80dcac9eb7ab2588998971e0cd0975aa732ed678ce636f69ff1c78b7903d202463576babbf20&scene=21#wechat_redirect)

```
var a = ???;
if(a == 1 && a == 12){  
  console.log(a);
}

```

> 如果你也觉得不可能的话，一起随着文章看下去。你也会觉得有点意思~

下文不仅仅涉及的是 JS，还有 Java 等等其他语言~

### **正文**

#### **一、JS 版本**

> 当然可能有 JS 的小伙伴想要求解答，这里碰巧写过解析，这里就贴出他的文章地址

看了 JS 的答案，我一直顺着这个解题思路再想：Java 中有没有可能完成这个等式？只能说自己 “功力太浅” 始终没有找到合适的解决方式... 因此自己就 Google 了一下，发现果然有 “闲的蛋疼” 的人提供了答案，甚至还提供了多种版本：

### **二、Java 版本**

这里就直接贴答案了，虽然不能说非常的贴近于题目，但也着实展示了其中的巧妙：

```
Class cache = Integer.class.getDeclaredClasses()[0];
Field c = cache.getDeclaredField("cache");
c.setAccessible(true);
Integer[] array = (Integer[]) c.get(cache);
// array[129] is 1
array[130] = array[129]; 
// Set 2 to be 1
array[131] = array[129]; 
// Set 3 to be 1
Integer a = 1;
if(a == (Integer)1 && a == (Integer)2 && a == (Integer)3){ 
   System.out.println("Success");
}

```

另一个答案，说实话比较 “牛逼” 了：

> 这里用到了 PowerMockRunner，也算是咱们解题思路的上最直接的帮手...

![](https://mmbiz.qpic.cn/mmbiz_jpg/eZzl4LXykQyhFnNjpLJ2jSf78SLTzI5oOvpgibzVh6cbZbuoicVgVsrd9xFwcusGcOACKRk2ZyZURbtFk7ughL2Q/640?wx_fmt=jpeg)

### **尾声**

写这篇文章其实并不是为了去深挖这些语言特性，只是单纯的觉得很有意思。

[如果从这个题目本身出发，它既可以考察解题者的语言特性掌握程度；也可以考察出解题者对待问题，尝试解决问题的方法论。学习这条路上，任重而道远。](http://mp.weixin.qq.com/s?__biz=MzI2NTAzNzgxNw==&mid=2247510871&idx=1&sn=e4682d125f207a3cc34633925cf04ba7&chksm=eaa199c1ddd610d73b1a5b61551273c71f562afe192d1f1279810546b269308702a67b7fc08c&scene=21#wechat_redirect)

> 链接：https://www.toutiao.com/i6805578326279717390

- EOF -