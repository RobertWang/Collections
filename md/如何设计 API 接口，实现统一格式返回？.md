> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4NzQ0Njc4Ng==&mid=2247503800&idx=2&sn=85b0d0db4f666c381a3d452b247e6cf7&chksm=903bcbd5a74c42c342376d43b95ad2df3f498039dd6afe60455d581a3566ebcb11f9efa901c7&mpshare=1&scene=1&srcid=0316L0rrvL4bNTREVngrjKNF&sharer_sharetime=1647401163270&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来源：老顾聊技术

*   前言
    
*   接口相互作用
    
*   返回格式
    
*   控制层控制器
    
*   美观美化
    
*   优雅优化
    
*   实现方案
    

* * *

![](https://mmbiz.qpic.cn/mmbiz_png/obDoO79MTFHgSHNqfGANv7LtWdWz46NSyrInic06NqYGQXiaBXvkk1ZiaVXVllemxQlicBDl8WZ1ic0iab1LyeicQ9JLA/640?wx_fmt=png)

前言  

=====

在移动互联网，分布式，微服务盛行的今天，现在项目绝大部分都采用的微服务框架，前分离分离方式，（题外话：前重新的工作分配越来越明确，现在的前端都称为大前端，技术栈以及生态圈都已经非常成熟；以前官员人员瞧不起前端人员，那现在高层人员要重新认识一下前端，前端已经很成体系了）。

一般系统的大致整体架构图如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu83jEMqIPQ46ocribBdcHLAHHCAhkwEUfoz77r9ibRsfQLjs7vXYEibCNKA/640?wx_fmt=jpeg)

> 需要说明的是，有些小伙伴会回复说，这个架构太简单了吧，太 low 了，什么网关啊，缓存啊，消息中间件啊，都没有。因为老顾介绍主要介绍的是 API 接口，所以我们聚焦点，其他的模块小伙伴们自行去补充。

接口相互作用
======

前端和前端进行交互，前端按约定的请求 URL 路径，并合并相关参数，进入服务器接收请求，进行业务处理，返回数据给前端。

> 针对 URL 路径的 restful 风格，以及引用参数的公共请求头的要求（如：app_version，api_version，device 等），老顾这里就不介绍了，小伙伴们可以自行去了解，也比较简单。

前端服务器如何实现把数据返回给前端？

返回格式
====

最初返回给前端我们一般用 JSON 体方式，定义如下：

```
{
  #返回状态码
  code:integer,
  #返回信息描述
  message:string,
  #返回值
  data:object
}
```

**CODE 状态码**

code 返回状态码，一般小伙伴们是在开发的时候需要什么，就添加什么。

如接口要返回用户权限异常，我们加一个状态码为 101 吧，下一次又要加一个数据参数异常，就加一个 102 的状态码。这样虽然能够照常满足业务，但状态码太凌乱了

我们应该可以参考 HTTP 请求返回的状态码

```
：下面是常见的HTTP状态码：
200 - 请求成功
301 - 资源（网页等）被永久转移到其它URL
404 - 请求的资源（网页等）不存在
500 - 内部服务器错误
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu8HSswndhb7ibVXgmia5djlLlf9fBMP3ReC5ZF3Qz9vWsIQUZZKMBVzS5g/640?wx_fmt=jpeg)

我们可以参考这样的设计，这样的好处就把错误类型归类到某个区间内，如果区间不够，可以设计成 4 个数字。

```
#1000～1999 区间表示参数错误
#2000～2999 区间表示用户错误
#3000～3999 区间表示接口异常
```

这样前端开发人员在得到返回值后，根据状态码就可以知道，大概什么错误，再根据消息相关的信息描述，可以快速定位。

**资讯**

这个相对相对理解比较简单，就是发生错误时，如何友好的进行提示。一般的设计是和 code 状态码一起设计，如

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu85g2wa6uks3icsvCcqSicR2UicGyYz9HicoUXweQxL17QCaMUJVHIkjm30A/640?wx_fmt=jpeg)

再在枚举中定义，状态码

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu8owjhicx33TJBvTicpyMlQVoxYj6qBFXESIYWvtGMxC3Hll78ibxGmoTTg/640?wx_fmt=jpeg)

状态码和信息就会一一对应，比较好维护。

**数据**

返回数据体，JSON 格式，根据不同的业务又不同的 JSON 体。

我们要设计一个返回体类结果

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu8CVnECib1TQDCu8YO3wQP6ogP03IficaLMr6cu21bpeS5vF805ibfvymNA/640?wx_fmt=jpeg)

控制层控制器
======

我们会在控制器层处理业务请求，并返回给前端，以 orderorder 为例

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu89rkozDdMh5f6qIyAe37ntddiadXqLvqmE1eTF1l30MS3Jr4dibQpjoKQ/640?wx_fmt=jpeg)

我们看到在获得订单对象之后，我们是用的结果构造方法进行包装赋值，然后进行返回。小伙伴们有没有发现，构造方法这样的包装是不是很麻烦，我们可以优化一下。

美观美化
====

我们可以在结果类中，加入静态方法，一看就懂

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu8W1nQqoMibqR963WMFZEml8q3PcicSWSSiazyKcYcPdhlyr1DK66EhK12g/640?wx_fmt=jpeg)

那我们来改造一下 Controller

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu8zwfQf7z4Ns9u2bIQ6OXSrC4bB0pL7W1nC8bMJufaUHeD34Gf5cbjow/640?wx_fmt=jpeg)

代码是不是比较简洁了，也美观了。

优雅优化
====

上面我们看到在结果类中增加了静态方法，正确的业务处理代码简洁了。但小伙伴们有没有发现这样有几个问题：

> 1，每个方法的返回都是 Result 封装对象，没有业务含义
> 
> 2，在业务代码中，成功的时候我们调用 Result.success，异常错误调用 Result.failure。是不是很多余
> 
> 3，上面的代码，判断 id 是否为 null，实际上我们可以使用 hibernate validate 做校验，没有必要在方法体中做判断。

我们最好的方式直接返回真实业务对象，最好不要改变之前的业务方式，如下图

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu85QgXOePxSWP3wSvk4tRYmia1F6mFxmHOPlqlS1j2q9YCLO9BSCtu17g/640?wx_fmt=jpeg)

这个和我们平时的代码是一样的，非常直观，直接返回 order 对象，这样是不是很完美。那实现方案是什么呢？

实现方案
====

小伙伴们怎么去实现是不是有点思路，在这个过程中，我们需要做几个事情

> 1，定义一个注解 @ResponseResult，表示此接口返回的值需要包装一下
> 
> 2，拦截请求，判断此请求是否需要被 @ResponseResult 注解
> 
> 3，核心步骤就是实现接口 ResponseBodyAdvice 和 @ControllerAdvice，判断是否需要包装返回值，如果需要，就把控制器接口的返回值进行转换。

**注解类**

采用标记方法的返回值，是否需要包装

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu8UrcIEnMN4PLdtkibqfkaCzLz4ELZnRR7ZGVGbd1VPGe9A0HIdbH4VrA/640?wx_fmt=jpeg)

**拦截器**

拦截请求，是否此请求返回的值需要包装，实际上就是运行的时候，解析 @ResponseResult 注解

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu8DHzjwbTT9TEzRibcQQs70hrsyyoRkht2Rd1W808ueurQ6exEsQlsjKA/640?wx_fmt=jpeg)

此代码核心思想，就是获取此请求，是否需要返回值包装，设置一个属性标记。

**重建返回体**

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu8CrLWFQX82sAVynYicnx7zEKUAQibjibX7rcv1oCMZVicnS7E95Yic3Rpj0A/640?wx_fmt=jpeg)

上面的代码就是肯定肯定判断是否需要返回值包装，如果需要就直接包装。这里我们只处理了正常成功的包装，如果方法体报异常怎么办？处理异常也比较简单，只要判断 body 是否为异常类。

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu89cDuOAev7jlEY44qsxnTCICNibX8QoWgAichH9FWb8T8zdCmicicrNvI8w/640?wx_fmt=jpeg)

怎么做的异常处理，篇幅原因，老顾这里就不做介绍了，只要思路理清楚了，自行改造就行。

**改造控制器**

![](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffUstrsicqnPMIoP91TTibMu8Qfy1HpibS4kyKoneDb9qIib51EnTuHhmCaV8FHj5RPtJWLvhLMd5ia3DQ/640?wx_fmt=jpeg)

在控制器类上或者方法体上加上 @ResponseResult 注解，这样就 ok 了，简单吧。到此返回的设计思路完成，是不是又简洁，又优雅。

这个方案还有没有别的优化空间，当然是有的。如：每次请求都要反射一下，获取请求的方法是否需要包装，其实可以做个缓存，不需要每次都需要解析。当然整体思路了解，小伙伴们就可以在此基础之上自我扩展，