> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651600406&idx=2&sn=cc5ea842b03bc02c01fc8b96d2060101&chksm=8022e1d7b75568c1011ea45615500ecfc8341d494f5915d86bdaf980db4f7942ca7807587ee4&mpshare=1&scene=1&srcid=0315EIBxXlwKI3LQlYLGTCG1&sharer_sharetime=1647323423884&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

Typescript 支持泛型，也叫类型参数，可以对类型参数做一系列运算之后返回新的类型，这就是`类型编程`。

因为类型编程实现一些逻辑还是有难度的，所以被戏称为`类型体操`。

社区有用 Typescript 类型实现 Lisp 解释器、实现象棋等案例的（知乎可以搜到），这足够说明了 Typescript 类型可以实现各种复杂逻辑。

那 Typescript 类型体操这么难，有没有什么快速掌握的方式呢？

确实有，我总结了一些套路，可以快速提升 ts 类型体操水平。比如今天要讲的套路 -- **模式匹配**。

Typescript 类型的模式匹配
------------------

我们知道，字符串可以和正则做模式匹配，找到匹配的部分，提取子组，之后可以用 $1,$2 等引用匹配的子组。

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pdsfbIcjGrqPgD6hNh9iaqT7icicXrovS4XbiaaFagKnpenQdq7Z70TaNvNWw/640?wx_fmt=png)

Typescript 的类型也同样可以做模式匹配。

比如，提取 Promise<T> 的值的类型：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pds3BhTp2KcsZ0E1JjKwdCeof9HicfHnIT15fwz8O2UO7dnk51LOuo4uBQ/640?wx_fmt=png)

我们通过 extends 对传入的类型参数 T 做模式匹配，其中 value 部分是需要提取的，通过 infer 类声明一个局部变量 R 来保存，如果匹配，就返回匹配到的 R，否则就返回 never 代表没匹配到。

这就是 Typescript 类型的模式匹配。

小结一下： **Typescript 类型的模式匹配是通过 extends 对类型参数做匹配，结果保存到通过 infer 声明的局部类型变量里，如果匹配就能从该局部变量里拿到提取出的类型。**

这个模式匹配的套路有多有用呢？我们来看下在数组、字符串、函数等类型里的应用。

### 数组类型的模式匹配

#### pop

pop 是去掉最后一个元素，可以通过模式匹配来实现：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pdssG8V0sY2934VJYAslOhRxMRJ18eBV3rgPibLKNQmePVrRUTdBuWfrpw/640?wx_fmt=png)

我们通过模式匹配取出最后一个元素的类型和前面的元素的类型，分别用 infer 放入不同的变量里，然后构造一个新的数组类型返回。

#### shift

同样，shift 是去掉最开始的元素，也是类似的匹配方式来实现：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pds4MYGmSM5dzPF9R4CcMkHSicTfCeSV9qX2nj7qE6QkzHsWqB6pgC1QVQ/640?wx_fmt=png)

### 字符串类型的模式匹配

#### trim

trim 是去掉前后的空格、制表符、换行符，那么就通过模式匹配取出后面的字符，通过 infer 放入新的变量返回就行。

先实现 TrimLeft：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pdsatGNt0wgZBK1dPnTpjbSo2JT9SyS6R9JSjXXekoG3UmickiamFRr3ibTw/640?wx_fmt=png)

如果匹配就继续递归 TrimLeft，直到前面没有空白字符。

再实现 TrimRight：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pdsLHTSnHJldb6PbnkT2eh8eqRO1PiaSZoPe4ibBoQuJ0aiaONpacgcZ6kmg/640?wx_fmt=png)

然后两者结合，就是 Trim：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pdsYx375es29y2ibBcjB7eGKRF7rgJdvJ4rnsxibWVKjYicVxgNaKI58FI8g/640?wx_fmt=png)

#### replace

replace 是替换字符串中的一部分，可以通过模式匹配取出这段字符串前后的子串，通过 infer 放入不同的变量，然后和替换后的部分组成新字符串。

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pds3xPb0ub6swMRJ7fDex9XW4OZUibREwic3d082EUuF9bsOC2orn82U4vg/640?wx_fmt=png)

### 函数类型的模式匹配

#### 参数类型

取出参数的类型是通过模式匹配拿到参数部分，放入 infer 声明的变量里返回。

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pdscIxbzVuib44he52ddF7hOyWqH0VYXfib68CX4ZCd5EGEbtYTLDWvkEPA/640?wx_fmt=png)

#### 返回值类型

取出返回值类型也是通过模式匹配拿到返回值部分，放入 infer 声明的类型变量里返回。

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGgxL9jXHfS11Y5lVFkk2pdshV2DyD3EIRfL9DWJDXL0oOBeB9Tls4wfVUHiaRSiaveSXZpsguFXPbYg/640?wx_fmt=png)

总结
--

类型编程是对类型参数（泛型）做一系列运算之后返回新的类型，也叫类型体操。

类型体操可以实现很多复杂的逻辑，学习起来也有一定的难度，但是掌握一些套路之后也能快速掌握。

这些套路里面最常用的就是模式匹配了，类似字符串匹配和提取子串，类型也可以通过 extends 对类型参数做匹配，把需要提取的部分保存到通过 infer 声明的局部类型变量里。

类型参数的模式匹配的套路在字符串类型、数组类型、函数类型等都有大量的应用，掌握这一个套路可以提升一大截类型体操的水平。

- EOF -

推荐阅读  点击标题可跳转

1、[用 TS 类型系统实现大数加法](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651598708&idx=3&sn=7b70fa94943307c34467a5fecd3af1d3&chksm=8022f8b5b75571a3a56bb0d21ae9d1e240f60f98f458507b3e3cb1d8af5194a93cfb952f7bfc&scene=21#wechat_redirect)

2、[React 中的 TS 类型过滤原来是这么做的！](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651594965&idx=3&sn=3717171fd60d99ad890f3304edc76de5&chksm=8022f714b7557e02f50235fb2f03ba7de97a0d19d5b155c7e893dd679c8c1d8454e4ed463950&scene=21#wechat_redirect)

3、[vite+vue3+ts 搭建通用后台管理系统](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651588138&idx=3&sn=aede6605b77b5100fee432d9e075a458&chksm=8022d1ebb75558fdaf22346282754ce8996cefecb3f39a116bba7944ed2022f2a03dbdc3e39d&scene=21#wechat_redirect)