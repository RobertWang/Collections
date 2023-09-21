> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/_Ws7bzhJtX67HIuk_S-big)

本文介绍下当前常见的场景之一：Mysql 数据同步 Elasticsearch 的实现方案，这里以电商为例，其实所有相关搜索内容都可以使用此方案。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtCfd92SpRoIGkrC8oOVou9aia93OpdFiccmyzWjq4icKoHsiasb3kHw5qpg/640?wx_fmt=png)

对于搜索，应该是所有 APP 必备的基础功能，不同时期有不同的解决方案，本次重点讲解 Elasticsearch。

那么，对于运营系统将商品上架后，数据肯定是要写入 DB 的，这个 DB 我们直接假设为 Mysql，那么，mysql 中的数据是如何同步到 Elasticsearch 呢？这也是本文我们重点讨论的问题。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtPxgUCXIuhfia2y2LvEupAKEQNfFEfQJ79nMkcLA6KZnfPOxqSickERxQ/640?wx_fmt=png)

这是能想到的最直接的方式，在写入 MySQL，直接也同步往 ES 里写一份数据。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtdqtpWjJOEkgF5dSmy8ibSNIZ4yBLCNrGYhPQbqm0RtRqAAibp0VERibhw/640?wx_fmt=png)

总结：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRt6pKvV2ry3vPucUJMria95nMOdiciaWAQqf50AtSN2hYicsrszJUdOP1WiaQ/640?wx_fmt=png)

*   2. 异步双写
    

我们也很容易想到异步双写的办法，上架商品的时候，先把商品数据丢进 MQ，为了解耦合，我们一般会拆分一个搜索服务，由搜索服务去订阅商品变动的消息，来完成同步。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtBaAq7gyFMxfczoPp8JBdxic5OBKsibibIlh9ia6kgXjIekOD2OZgkibWfYQ/640?wx_fmt=png)

前面说的，一些数据需要聚合处理成类似宽表的结构怎么办呢？例如商品库的商品品类、spu、sku 表是分开的，但是查询是跨维度的，在 ES 里再聚合一次效率就低一些，最好就是把商品的数据给聚合起来，在 ES 里以类似大宽表的形式存储，这样一来查询效率就高一些。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtr1icGPF6tfKzRGImM86SgmicjDial30CsKzrxUH7lict4PX3TMy3FroqQg/640?wx_fmt=png)

对于多维度查询条件这种其实没什么好办法，基本上还是得搜索服务直接查库，或者远程调用，再查询一遍商品的数据库，就是所谓的回查。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtP5Tr1Mlp2eKLiacvUibOZnYKj7TmN94Fib5nYrJEeFZoKkjpcHicOuE8jw/640?wx_fmt=png)

总结:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtolVnMeGQib27HibGbuY3ERxQja7W5IevPWPg4aeQKEqcgjJN44uUCfVg/640?wx_fmt=png)

3. 定时任务

假如我们要快速搞搞，数据量有没那么大，怎么办呢？定时任务也可以。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtOtXs3eiciafNhJiaG0IZzxibypef13KfDLcB7LicbI603Klq1PKjL7zgxQQ/640?wx_fmt=png)

*   定时任务这种情况最麻烦的一点是频率不好选，频率高的话，会非自然地形成业务的波峰，导致存储的 CPU、内存占用波峰式上升，频率低的话实时性比较差，而且也有波峰的情况。  
    
    总结:
    
    ![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRteRdZlRicy3S4icby13DFvwBQPHupZiarelxk6VxHna1iacibggnywHYbbtA/640?wx_fmt=png)
    

4. 数据订阅

这是目前最流行的就是数据订阅，做到了用户完全无感知，系统完全自动处理。

MySQL 通过 binlog 订阅实现主从同步，各路数据订阅框架比如 canal 就依据这个原理，将 client 组件伪装成从库，来实现数据订阅。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRt1icnRhKgXrlC4PmrJAwYPpMm4WFAKXAxSblWABNUbvMq3gJvPnvCsWg/640?wx_fmt=png)

我们以应用最广泛的 canal 为例，canal 通过`canal-adapter`，支持多种适配器，其中就有 ES 适配器，通过一些配置，启动之后，就可以直接把 MySQL 数据同步到 ES，这个过程是零代码的。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtIqG6WibSCticAnicDVfcA1PPUoWKwssbMXQodRW4IjlPice6KR4Qp3Om6g/640?wx_fmt=png)

需要特别注意，使用 canal 看起来很美好，帮我们把同步的事情都干了，但其实，还是要写代码。为什么呢？

前面提到的多张表数据聚合，canal 的支持没那么好，所以还是得回查。这时候用 canal-adapter 就不合适了，需要自己实现 canal-client，监听和聚合数据，将数据写入 ES：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRt5Dicmicl2Z3ue9vxCnsn0uN0YsUKHQJcS6ICBl1eZt2oXwqIx1mT8tQw/640?wx_fmt=png)

这种看起来和异步双写比较像，但是第一降低了商品服务的耦合，第二数据的实时性更好。

总结:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/FAicwPFdhxnof9NmreGL9zHTJfTNelrRtVn85JJdm8Ia97UFeymcdeOYITic6WIoW7hg2USn6SsS9mXRnpclTVIg/640?wx_fmt=png)

至于数据订阅框架的选型，目前的组件较多，这里不再做更多介绍，如有兴趣，大家可以搜索 CDC(Change Data Capture) 进行更多的扩展和了解。

以上为全部内容。