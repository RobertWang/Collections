> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1628770329&ver=3248&signature=1ScXHP5Ck7wW6PfnOqFAaERvBeiCSlU9*ZnjGepreG3Q5NtOPYIQ0*-WXY02TfzWpzGzDsyX8kkgqOyggxPzATaHHPFPSaEwXwhunS8A02u*XZl24pFBB3MGw8JU-0SA&new=1)

前言

每家公司的 JD，有心的童鞋可以多关注了解。今日早读文章由腾讯 @朱林翻译授权分享。

正文从这开始～

> GraphQL 日渐成为数据查询的主流标准之一，整个生态圈也蓬勃发展。本文则由浅入深地详细介绍基础的 GraphQL 格式与关键字，有助于初学者对于 GraphQL 的使用形成体系认知。

GraphQL 日渐成为数据查询的主流标准之一。每天都会产生许多围绕这项技术发展的精彩讨论和新工具。 GraphQL 最棒的特性就是提供了一个丰富语言集来描述获取数据的 API 。但是用户该如何描述这种查询语言，以及 GraphQL 这项核心技术本身呢？ let’s talk !

GraphQL specification 解释了几乎所有出现在 GraphQL 查询语言中的概念，但是这篇文档实在是太长了，所以我准备在这篇博客中，借助一些具体的栗子来阐述其中一些最重要的概念，来帮助你成为 GraphQL 专家！至少在纸上谈兵方面 : )

> 注！ 这篇文章可不是 GraphQL 的入门读物。首先，你应该通读 concepts on the graphql.org docs ，然后通过 Learn Apollo tutorial 来学习使用 GraphQL ，最后当你想继续深入了解这项技术时，再回到这里来吧！

#### 最基本的 GraphQL 查询

大家通常会使用 “查询” 来称呼 GraphQL API 服务的一切。但是这样称呼会有太多东西混杂在一起了。我们可能会把我们跪求服务端的一系列行为称为一次查询、一次修改或者一次订阅，但我想 “请求（ request ）” 这个词可能更加符合 HTTP 通信的概念，下面我们先来定义一些最基础的概念：

*   GraphQL 请求体： 使用 GraphQL 语言定义的一个或多个操作或者数据片段，类型是字符串。
    
*   操作： 可以被 GraphQL 执行引擎理解的一次查询、修改或订阅。
    

为了搞清楚 GraphQL 各种基本操作之间的区别，让我们先来看一个简单的 GraphQL 请求体：

![](https://mmbiz.qpic.cn/mmbiz_png/meG6Vo0MevjEnTMC6JUQibd88WVbQuTsQkicNQTIh56zYRBdGax6HeHyLtUYO5zxT3uoc1wPkjF8AfP8sAaiaiajAw/640?wx_fmt=png)

这个请求体显示了 GraphQL 的主要构建块，它指定了你尝试获取的数据。  

*   字段 (Fields)：客户端请求的数据单元，最后作为 JSON 响应数据中的一个字段。请注意，它们始终称为 “字段”，无论它们所在的层次有多深。在你的查询中，对根节点字段的处理和最底层字段应该是一样的。
    
*   参数 (Arguments)：一组与特定字段关联的键值对。这些参数会跟它们相关的字段一起被传递到服务器端执行，并影响服务器对字段的处理方式。如上面的示例，参数可以是字面量，接下来还有参数作为变量形式的栗子。请注意，参数可以显示在任何字段中，即使是嵌套层次很深的字段。
    

为了让你以非常简洁的形式定义一个 GraphQL 查询，上面的栗子是 GraphQL 的一种非常简单的形式。但是在 GraphQL 操作中三种可选的部分都没有在上述栗子中使用。如果你不仅仅是用 GraphQL 执行查询操作，或是希望传递动态变量到 GraphQL 查询中，你就需要利用到这些新的 GraphQL 特性。

这里恰好有一个包含了所有可选部分的栗子：

![](https://mmbiz.qpic.cn/mmbiz_png/meG6Vo0MevjEnTMC6JUQibd88WVbQuTsQzKicV5dmGOPicvy3BAkC6nnibiaFLmiaO0dWchwCUwKNJxyvRYHDO2GhPqw/640?wx_fmt=png)

*   操作类型 (Operation type)：共三种类型：查询 ( query )、更新 ( mutation )、订阅 ( subscription )。它描述了你试图进行何种操作。然而这些看起来意思很接近的操作，GraphQL 服务器处理它们时还是会有一些不同。
    
*   操作名称 (Operation name)：为了方便调试和服务端打日志，最好给你的查询赋予语义化的命名。这样，无论你是在网络日志中或者 GraphQL 服务器上发现错误，你都可以通过名字很轻松的在代码库中定位问题，而不是靠猜测（类似的工具有 Apollo Optics ）。可以把操作名称想象成你最喜欢的编程语言中，一个语义化的函数名。
    
*   变量定义 (Variable definitions)：当客户端向 GraphQL 服务器发送查询时，会存在查询文档不变，当某些字段会动态变化的情况。这些就是查询中的变量。因为 GraphQL 是静态类型的，它可以实时验证你是否传递了正确的变量。这正是你声明变量类型时所计划提供的能力。
    
  

变量使用特定的序列化协议 (在目前的 GraphQL 服务实现中，通常是使用 JSON) 通过查询文档独立传输。下面是一个变量对象在查询文档中的示例：

![](https://mmbiz.qpic.cn/mmbiz_png/meG6Vo0MevjEnTMC6JUQibd88WVbQuTsQWGicMowCq8FRRMoGbHPCjtVtIB2o6AYobO5Tfu0Hue3hicXJoAJylMibQ/640?wx_fmt=png)

可以看到，这里的关键是变量名称需要与变量定义所匹配，其名称是 Episode 枚举中的一个成员。  

*   变量 (Variables): 它是传递给 GraphQL operation 的值的字典，提供了 operation 的动态入参。
    

这里有一个在谈及 GraphQL 的技术意义时很重要，却不常被提及的核心概念——花括号之间的所有东西叫什么？

![](https://mmbiz.qpic.cn/mmbiz_png/meG6Vo0MevjEnTMC6JUQibd88WVbQuTsQZrMic9HNLHYJFliazAbAzRYaIMVMFdJRwzYKmOxmwAWuxjbNPyaoAGjA/640?wx_fmt=png)

选择集 (selection set) 是一个会在 GraphQL 文档中经常出现的概念，它赋予了 GraphQL 递归的特性，允许你获取嵌套形式的数据。  

*   选择集 (selection set)：它是一次 operation 中需要的一组字段，或者被嵌套在其他的字段中。 GraphQL 查询必须包含一个标识选择集的字段，且该字段返回的是对象类型，选择集不能设置在返回值是标量类型（ Scalar Types ）的字段上，例如 Int 或者 String。
    

#### 片段 (Fragments)

当开始介绍片段 (fragments) 之后，GraphQL 将变得更加强大。它带来了一系列新的概念。

片段定义 (Fragment definition)：定义一个片段是 GraphQL 文档的一部分。为了区别于我们下面会介绍的内联片段，它有时候也被称为片段命名。

![](https://mmbiz.qpic.cn/mmbiz_png/meG6Vo0MevjEnTMC6JUQibd88WVbQuTsQCzY8x6oDeHpXYUicme6FAMKung1epp3cJjpDA1jdricQMbOzQZydy8LA/640?wx_fmt=png)

*   片段名称 (Fragment name): 片段( fragments ) 名在 GraphQL 文档中必须是唯一的。这个名称用于在操作 ( operations ) 或者其他片段 ( fragments ) 引用片段 ( fragments )。就像操作( operations ) 名称一样，片段名也能用于服务端日志调试，所以我们推荐使用明确且有意义的片段 ( fragments ) 名。如果你使用了正确的片段 ( fragments ) 名，在优化数据获取时，你能够很好的追踪你的代码。
    
*   类型条件 (Type condition)： GraphQL 操作总是开始于查询、修改或者订阅 schema 中的类型，但是片段( fragments ) 能够用于任一选择，所以为了将校验片段 ( fragments ) 与校验 schema 独立开，你需要指定片段 ( fragments ) 能够使用的类型，而这就是类型条件 ( Type condition ) 的作用。
    

就像操作 (operations) 一样，片段也选择集，使用起来也跟在操作 ( operations ) 中使用选择集一样。

#### 在你的操作 (operations) 中使用片段( fragments )

片段 (fragments) 只有在操作 ( operations ) 中使用才能发挥出作用。片段是 GraphQL 的主要组合数据结构，通过片段可以重用重复的字段选择，减少 query 中的重复内容。接下来我们将介绍使用片段 ( fragments ) 的两种方式：

![](https://mmbiz.qpic.cn/mmbiz_png/meG6Vo0MevjEnTMC6JUQibd88WVbQuTsQuhsia3ckfVe18BKOichQ9vsd48q07MLPDK7PbOErgFjwbx0kHDBnfiaTQ/640?wx_fmt=png)

片段扩展运算符（ Fragment spread ）: 当你在操作或者其他片段中使用片段时，你可以将片段名置于… 之后来表示片段。例如没有片段时需要这样编写 query：  

```
query noFragments {
  user(id: 4) {
    friends(first: 10) {
      id      
      name      
      profilePic(size: 50)
    }
    mutualFriends(first: 10) {
      id      
      name      
      profilePic(size: 50)
    }
  }
}

```

query 中存在下列重复的选择集合：

```
{
  id
  name
  profilePic(size: 50)
}

```

可以用片段简化为：

```
query withFragments {
  user(id: 4) {
    friends(first: 10) {
      ...friendFields
    }
    mutualFriends(first: 10) {
      ...friendFields
    }
  }
}
fragment friendFields on User {
    id  
    name  
    profilePic(size: 50)
}

```

使用片段时需要加上 … 操作符表示展开片段内容，这称为片段扩展运算符 (fragment spread)，它可以用在任何选择集( selection set ) 中，用以匹配片段的类型条件。

内联片段（ Inline fragment ）: 如果你仅仅是想执行一些依赖结果类型的字段，却不想把它们抽离成独立的定义，你可以使用内联片段（ inline fragment ）。它使用起来跟独立的命名片段一样，但是写在查询的内部。有一点不同的是，对于内联片段来说类型条件 (type condition) 不是必须的，可以像使用指令一样来使用它，接下来我们会演示指令 ( directive ) 的栗子。

#### 指令 (Directives)

指令是独立于 GraphQL server 之外的一个附加功能。指令不会对结果的值产生影响，但是会影响哪些结果会被返回，也许还会影响这些结果是如何被执行的。指令可以出现在查询的任何地方，但在这篇文章中我们只关注当前 GrahpQL 文档所描述的 skip(忽略) 和 include(包括) 两个指令。

![](https://mmbiz.qpic.cn/mmbiz_png/meG6Vo0MevjEnTMC6JUQibd88WVbQuTsQ5tacskbVia3iabbtGmbsDcV3Oo5GkLgicoyCUjahOCsI4IokRgSt7Fsibw/640?wx_fmt=png)

你能在上文厨房水槽的栗子中使用指令 skip 和 include。include 指令表示只有在 if 参数为 true 时才引入片段表示的字段。skip 指令表示在 if 参数为 true 时忽略片段中的字段。由于指令的语法相当灵活，我们可以利用它来给 GraphQL 添加更多的特性，而不是使用语法解析或者引入更复杂的工具的方式。  

指令 (Directive): 在字段、片段或者查询中的一个注释，include 指令表示只有在 if 参数为 true 时才引入片段表示的字段。skip 指令表示在 if 参数为 true 时忽略片段中的字段。

指令参数 (Directive arguments)： 与字段参数类似，只不过它们是被执行引擎处理，而不是传递给字段解析器 ( field resolver )。

#### 总结

GraphQL 是在应用层对业务数据模型的抽象，是对数据请求定制的 DSQL，它解除了接口和数据之间的绑定，对业务数据结构做了抽象和整理，业务逻辑中的数据依赖于底层数据库结构，并且可以由具体业务场景来定制，不同的业务场景只要基于同样一套基础业务数据模型就可以得到复用，在我看来，这才是 GraphQL 带来的最大改变和收益。 目前 GitHub 整站 API 已迁移 GraphQL，淘宝也在生产环境有所实践。

关于本文  
译者：@朱林  
译文：  
https://zhuanlan.zhihu.com/p/40418866  
作者：@Sashko Stubailo  
原文：  
https://blog.apollographql.com/the-anatomy-of-a-graphql-query-6dffa9e9e747