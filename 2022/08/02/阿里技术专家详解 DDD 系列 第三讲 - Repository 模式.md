> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDEwNjk5OQ==&mid=2650406692&idx=1&sn=4a4ac4168299d8ca1905a4f457ae4c59&chksm=8395373cb4e2be2a2d066a5ea4e631fd6270e969ce61883b488f61c1ce33fbc0b362ec9cbf7b&scene=21#wechat_redirect)

作者 | 殷浩

出品 | 阿里巴巴新零售淘系技术部

写在前面：

这篇文章和[《阿里技术专家详解 DDD 系列 第二弹 - 应用架构》](http://mp.weixin.qq.com/s?__biz=MzAxNDEwNjk5OQ==&mid=2650404060&idx=1&sn=cacf40d19528f6c2d9fd165151d6e8b4&chksm=83953cc4b4e2b5d2bd4426e0d2103f2e95715b682f3b7ff333dbb123eaa79d3e5ad24f64beac&scene=21#wechat_redirect)隔了比较久，一方面是工作比较忙，另一方面是在讲 Repository 之前其实应该先讲 Entity（实体）、Aggregate Root（聚合根）、Bounded Context（限界上下文）等概念。但在实际写的过程中，发现单纯讲 Entity 相关的东西会比较抽象，很难落地。所以本文被推倒重来，从 Repository 开始入手，先把可以落地的、能形成规范的东西先确定下来，最后再尝试落地 Entity。这个当然也是我们可以在日常按照 DDD 重构时尝试的路径。

提前预告，接下来的一篇文章将覆盖 Anti-Corruption Layer（防腐层）的逻辑，但是你会发现跟 Repository 模式的理念非常接近。等所有周边的东西都覆盖之后，再详细讲 Entity 也许会变得不那么抽象。

DDD 的宏观理念其实并不难懂，但是如同 REST 一样，DDD 也只是一个设计思想，缺少一套完整的规范，导致 DDD 新手落地困难。  

我之前的架构篇主要从顶层设计往下看，从这一篇开始我希望能填补上一些 DDD 的代码落地规范，帮助同学在日常工作中落地 DDD 思想，并且希望能通过一整套规范，让不同的业务之间的同学能够更快的看懂、掌握对方的代码。但是规则是死的、人是活的，各位同学需要根据自己业务的实际情况去有选择的去落地规范，DDD 的规范不可能覆盖所有场景，但我希望能通过解释，让同学们了解 DDD 背后的一些思考和取舍。

****为什么要用 Repository****  

### ▐  实体模型 vs. 贫血模型

Entity（实体）这个词在计算机领域的最初应用可能是来自于 Peter Chen 在 1976 年的 “The Entity-Relationship Model - Toward a Unified View of Data"（ER 模型），用来描述实体之间的关系，而 ER 模型后来逐渐的演变成为一个数据模型，在关系型数据库中代表了数据的储存方式。

而 2006 年的 JPA 标准，通过 @Entity 等注解，以及 Hibernate 等 ORM 框架的实现，让很多 Java 开发对 Entity 的理解停留在了数据映射层面，忽略了 Entity 实体的本身行为，造成今天很多的模型仅包含了实体的数据和属性，而所有的业务逻辑都被分散在多个服务、Controller、Utils 工具类中，这个就是 Martin Fowler 所说的的 Anemic Domain Model（贫血领域模型）。

如何知道你的模型是贫血的呢？可以看一下你代码中是否有以下的几个特征：

1.  **有大量的 XxxDO 对象：**这里 DO 虽然有时候代表了 Domain Object，但实际上仅仅是数据库表结构的映射，里面没有包含（或包含了很少的）业务逻辑；
    
2.  **服务和 Controller 里有大量的业务逻辑：**比如校验逻辑、计算逻辑、格式转化逻辑、对象关系逻辑、数据存储逻辑等；
    
3.  大量的 Utils 工具类等。
    

而贫血模型的缺陷是非常明显的：

1.  **无法保护模型对象的完整性和一致性：**因为对象的所有属性都是公开的，只能由调用方来维护模型的一致性，而这个是没有保障的；之前曾经出现的案例就是调用方没有能维护模型数据的一致性，导致脏数据使用时出现 bug，这一类的 bug 还特别隐蔽，很难排查到。
    
2.  **对象操作的可发现性极差：**单纯从对象的属性上很难看出来都有哪些业务逻辑，什么时候可以被调用，以及可以赋值的边界是什么；比如说，Long 类型的值是否可以是 0 或者负数？
    
3.  **代码逻辑重复：**比如校验逻辑、计算逻辑，都很容易出现在多个服务、多个代码块里，提升维护成本和 bug 出现的概率；一类常见的 bug 就是当贫血模型变更后，校验逻辑由于出现在多个地方，没有能跟着变，导致校验失败或失效。
    
4.  **代码的健壮性差：**比如一个数据模型的变化可能导致从上到下的所有代码的变更。
    
5.  **强依赖底层实现：**业务代码里强依赖了底层数据库、网络 / 中间件协议、第三方服务等，造成核心逻辑代码的僵化且维护成本高。
    

虽然贫血模型有很大的缺陷，但是在我们日常的代码中，我见过的 99% 的代码都是基于贫血模型，为什么呢？我总结了以下几点：

1.  **数据库思维：**从有了数据库的那一天起，开发人员的思考方式就逐渐从 “写业务逻辑“转变为了” 写数据库逻辑”，也就是我们经常说的在写 CRUD 代码。
    
2.  **贫血模型 “简单”：**贫血模型的优势在于 “简单”，仅仅是对数据库表的字段映射，所以可以从前到后用统一格式串通。这里简单打了引号，是因为它只是表面上的简单，实际上当未来有模型变更时，你会发现其实并不简单，每次变更都是非常复杂的事情
    
3.  **脚本思维：**很多常见的代码都属于 “脚本” 或“胶水代码”，也就是流程式代码。脚本代码的好处就是比较容易理解，但长久来看缺乏健壮性，维护成本会越来越高。
    

但是可能最核心的原因在于，实际上我们在日常开发中，混淆了两个概念：

*   **数据模型（Data Model）：**指业务数据该如何持久化，以及数据之间的关系，也就是传统的 ER 模型；
    
*   **业务模型 / 领域模型（Domain Model）：**指业务逻辑中，相关联的数据该如何联动。
    

所以，解决这个问题的根本方案，就是要在代码里严格区分 Data Model 和 Domain Model，具体的规范会在后文详细描述。在真实代码结构中，Data Model 和 Domain Model 实际上会分别在不同的层里，Data Model 只存在于数据层，而 Domain Model 在领域层，而链接了这两层的关键对象，就是 Repository。

### ▐  Repository 的价值

在传统的数据库驱动开发中，我们会对数据库操作做一个封装，一般叫做 Data Access Object（DAO）。DAO 的核心价值是封装了拼接 SQL、维护数据库连接、事务等琐碎的底层逻辑，让业务开发可以专注于写代码。但是在本质上，DAO 的操作还是数据库操作，DAO 的某个方法还是在直接操作数据库和数据模型，只是少写了部分代码。在 Uncle Bob 的《代码整洁之道》一书里，作者用了一个非常形象的描述：

*   **硬件（Hardware）：**指创造了之后不可（或者很难）变更的东西。数据库对于开发来说，就属于” 硬件 “，数据库选型后基本上后面不会再变，比如：用了 MySQL 就很难再改为 MongoDB，改造成本过高。
    
*   **软件（Software）：**指创造了之后可以随时修改的东西。对于开发来说，业务代码应该追求做” 软件 “，因为业务流程、规则在不停的变化，我们的代码也应该能随时变化。
    
*   **固件（Firmware）：**即那些强烈依赖了硬件的软件。我们常见的是路由器里的固件或安卓的固件等等。固件的特点是对硬件做了抽象，但仅能适配某款硬件，不能通用。所以今天不存在所谓的通用安卓固件，而是每个手机都需要有自己的固件。
    

从上面的描述我们能看出来，数据库在本质上属于”硬件 “，DAO 在本质上属于” 固件 “，而我们自己的代码希望是属于” 软件“。但是，固件有个非常不好的特性，那就是会传播，也就是说当一个软件强依赖了固件时，由于固件的限制，会导致软件也变得难以变更，最终让软件变得跟固件一样难以变更。

举个软件很容易被 “固化” 的例子：

在上面的这段简单代码里，该对象依赖了 DAO，也就是依赖了 DB。虽然乍一看感觉并没什么毛病，但是假设未来要加一个缓存逻辑，代码则需要改为如下：

所以，我们需要一个模式，能够隔离我们的软件（业务逻辑）和固件 / 硬件（DAO、DB），让我们的软件变得更加健壮，而这个就是 Repository 的核心价值。

****模型对象代码规范****
----------------

### ▐  对象类型

在讲 Repository 规范之前，我们需要先讲清楚 3 种模型的区别，Entity、Data Object (DO) 和 Data Transfer Object (DTO)：

*   **Data Object （DO、数据对象）：**实际上是我们在日常工作中最常见的数据模型。但是在 DDD 的规范里，DO 应该仅仅作为数据库物理表格的映射，不能参与到业务逻辑中。为了简单明了，DO 的字段类型和名称应该和数据库物理表格的字段类型和名称一一对应，这样我们不需要去跑到数据库上去查一个字段的类型和名称。（当然，实际上也没必要一摸一样，只要你在 Mapper 那一层做到字段映射）
    
*   **Entity（实体对象）：**实体对象是我们正常业务应该用的业务模型，它的字段和方法应该和业务语言保持一致，和持久化方式无关。也就是说，Entity 和 DO 很可能有着完全不一样的字段命名和字段类型，甚至嵌套关系。Entity 的生命周期应该仅存在于内存中，不需要可序列化和可持久化。
    
*   **DTO（传输对象）：**主要作为 Application 层的入参和出参，比如 CQRS 里的 Command、Query、Event，以及 Request、Response 等都属于 DTO 的范畴。DTO 的价值在于适配不同的业务场景的入参和出参，避免让业务对象变成一个万能大对象。
    

### ▐  模型对象之间的关系

在实际开发中 DO、Entity 和 DTO 不一定是 1:1:1 的关系。一些常见的非 1:1 关系如下：

复杂的 Entity 拆分多张数据库表：常见的原因在于字段过多，导致查询性能降低，需要将非检索、大字段等单独存为一张表，提升基础信息表的检索效率。常见的案例如商品模型，将商品详细描述等大字段单独保存，提升查询性能：

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju9GpOruQRYGeb3wfBv2NUctCpN79icicVHVo302p7ico79K2mj0Z7TUZbibXic5JdNAfh7tXpCzlVLZOIA/640?wx_fmt=png)

多个关联的 Entity 合并一张数据库表：这种情况通常出现在拥有复杂的 Aggregate Root - Entity 关系的情况下，且需要分库分表，为了避免多次查询和分库分表带来的不一致性，牺牲了单表的简洁性，提升查询和插入性能。常见的案例如主子订单模型：

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju9GpOruQRYGeb3wfBv2NUctWkDMJZibEmbFUFsDL06TnSqfgKOQ2MBtElqKHghuMpUYZA6zicXuwxUg/640?wx_fmt=png)

从复杂 Entity 里抽取部分信息形成多个 DTO：这种情况通常在 Entity 复杂，但是调用方只需要部分核心信息的情况下，通过一个小的 DTO 降低信息传输成本。同样拿商品模型举例，基础 DTO 可能出现在商品列表里，这个时候不需要复杂详情：

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju9GpOruQRYGeb3wfBv2NUctFLqvSIkOBeclECWIfhLN66NibUIicwEBTyAibTiaO8HsFF0yTksMmYfKIA/640?wx_fmt=png)

合并多个 Entity 为一个 DTO：这种情况通常为了降低网络传输成本，降低服务端请求次数，将多个 Entity、DP 等对象合并序列化，并且让 DTO 可以嵌套其他 DTO。同样常见的案例是在订单详情里需要展示商品信息：

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju9GpOruQRYGeb3wfBv2NUctljEcIdFoAkwaCUICqZ0gs8cQYSlhxWriaA5KVTqCth0w9iaSniao3Hsnw/640?wx_fmt=png)

### ▐  模型所在模块和转化器

由于现在从一个对象变为 3 + 个对象，对象间需要通过转化器（Converter/Mapper）来互相转化。而这三种对象在代码中所在的位置也不一样，简单总结如下：

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju9GpOruQRYGeb3wfBv2NUctsbKDEt4fphI2YZ1b7kFrF2FOeX8S44sY9ib0DLjZyn2KXArjb0G2p9Q/640?wx_fmt=png)

DTO Assembler：在 Application 层，Entity 到 DTO 的转化器有一个标准的名称叫 DTO Assembler。Martin Fowler 在 P of EAA 一书里对于 DTO 和 Assembler 的描述：Data Transfer Object。DTO Assembler 的核心作用就是将 1 个或多个相关联的 Entity 转化为 1 个或多个 DTO。

Data Converter：在 Infrastructure 层，Entity 到 DO 的转化器没有一个标准名称，但是为了区分 Data Mapper，我们叫这种转化器 Data Converter。这里要注意 Data Mapper 通常情况下指的是 DAO，比如 Mybatis 的 Mapper。Data Mapper 的出处也在 P of EAA 一书里：Data Mapper

如果是手写一个 Assembler，通常我们会去实现 2 种类型的方法，如下；Data Converter 的逻辑和此类似，略过。

在调用方使用时是非常方便的（请忽略各种异常逻辑）：

虽然 Assembler/Converter 是非常好用的对象，但是当业务复杂时，手写 Assembler/Converter 是一件耗时且容易出 bug 的事情，所以业界会有多种 Bean Mapping 的解决方案，从本质上分为动态和静态映射。

动态映射方案包括比较原始的 BeanUtils.copyProperties、能通过 xml 配置的 Dozer 等，其核心是在运行时根据反射动态赋值。动态方案的缺陷在于大量的反射调用，性能比较差，内存占用多，不适合特别高并发的应用场景。

所以在这里我给用 Java 的同学推荐一个库叫 MapStruct（MapStruct 官网）。MapStruct 通过注解，在编译时静态生成映射代码，其最终编译出来的代码和手写的代码在性能上完全一致，且有强大的注解等能力。如果你的 IDE 支持，甚至可以在编译后看到编译出来的映射代码，用来做 check。在这里我就不细讲 MapStruct 的用法了，具体细节请见官网。

用了 MapStruct 之后，会节省大量的成本，让代码变得简洁如下：

### ▐  模型规范总结

<table><tbody><tr><td width="123" valign="top"><br></td><td width="123" valign="top"><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">DO</span></strong></td><td width="123" valign="top"><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">Entity</span></strong></td><td width="123" valign="top" class=""><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">DTO</span></strong></td></tr><tr><td width="123" valign="top"><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">目的</span></strong></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">数据库表映射</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">业务逻辑</span></td><td width="123" valign="top" class=""><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);" class="">适配业务场景</span></td></tr><tr><td width="123" valign="top"><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">代码层级</span></strong></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">Infrastructure</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">Domain</span></td><td width="123" valign="top" class=""><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">Application</span></td></tr><tr><td width="123" valign="top"><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">命名规范</span></strong></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">XxxDO</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">Xxx</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">XxxDTO<br data-darkmode-color-16594275252309="rgb(157, 164, 171)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)|rgb(36, 41, 46)" data-darkmode-bgcolor-16594275252309="rgba(25, 25, 25, 0.8)" data-darkmode-original-bgcolor-16594275252309="#fff|rgba(255, 255, 255, 0.8)" data-style="box-sizing: border-box; color: rgb(36, 41, 46); font-family: -apple-system, system-ui, &quot;Segoe UI&quot;, Helvetica, Arial, sans-serif, &quot;Apple Color Emoji&quot;, &quot;Segoe UI Emoji&quot;, &quot;Segoe UI Symbol&quot;; font-size: 16px; text-align: start; background-color: rgba(255, 255, 255, 0.8);">XxxCommand<br data-darkmode-color-16594275252309="rgb(157, 164, 171)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)|rgb(36, 41, 46)" data-darkmode-bgcolor-16594275252309="rgba(25, 25, 25, 0.8)" data-darkmode-original-bgcolor-16594275252309="#fff|rgba(255, 255, 255, 0.8)" data-style="box-sizing: border-box; color: rgb(36, 41, 46); font-family: -apple-system, system-ui, &quot;Segoe UI&quot;, Helvetica, Arial, sans-serif, &quot;Apple Color Emoji&quot;, &quot;Segoe UI Emoji&quot;, &quot;Segoe UI Symbol&quot;; font-size: 16px; text-align: start; background-color: rgba(255, 255, 255, 0.8);">XxxRequest 等</span></td></tr><tr><td width="123" valign="top"><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">字段名称标准</span></strong></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">数据库表字段名</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">业务语言</span></td><td width="123" valign="top" class=""><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);" class="">和调用方商定</span></td></tr><tr><td width="123" valign="top"><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">字段数据类型</span></strong></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">数据库字段类型</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">尽量是有业务含义的类型，比如 DP</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">和调用方商定</span></td></tr><tr><td width="123" valign="top"><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">是否需要序列化</span></strong></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">不需要</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">不需要</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">需要</span></td></tr><tr><td width="123" valign="top"><strong><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">转化器</span></strong></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">Data Converter</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">Data Converter<br data-darkmode-color-16594275252309="rgb(157, 164, 171)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)|rgb(36, 41, 46)" data-darkmode-bgcolor-16594275252309="rgba(25, 25, 25, 0.8)" data-darkmode-original-bgcolor-16594275252309="#fff|rgba(255, 255, 255, 0.8)" data-style="box-sizing: border-box; color: rgb(36, 41, 46); font-family: -apple-system, system-ui, &quot;Segoe UI&quot;, Helvetica, Arial, sans-serif, &quot;Apple Color Emoji&quot;, &quot;Segoe UI Emoji&quot;, &quot;Segoe UI Symbol&quot;; font-size: 16px; text-align: start; background-color: rgba(255, 255, 255, 0.8);">DTO Assembler</span></td><td width="123" valign="top"><span data-darkmode-color-16594275252309="rgb(163, 163, 163)" data-darkmode-original-color-16594275252309="#fff|rgb(0, 0, 0)" data-style="font-size: 16px; color: rgb(0, 0, 0); font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, &quot;PingFang SC&quot;, Cambria, Cochin, Georgia, Times, &quot;Times New Roman&quot;, serif; letter-spacing: 0.5px; white-space: pre-line; caret-color: rgb(255, 0, 0);">DTO Assembler</span></td></tr></tbody></table>

从使用复杂度角度来看，区分了 DO、Entity、DTO 带来了代码量的膨胀（从 1 个变成了 3+2+N 个）。但是在实际复杂业务场景下，通过功能来区分模型带来的价值是功能性的单一和可测试、可预期，最终反而是逻辑复杂性的降低。  

****Repository 代码规范****
-----------------------

### ▐  接口规范  

上文曾经讲过，传统 Data Mapper（DAO）属于 “固件”，和底层实现（DB、Cache、文件系统等）强绑定，如果直接使用会导致代码“固化”。所以为了在 Repository 的设计上体现出“软件” 的特性，主要需要注意以下三点：

1.  **接口名称不应该使用底层实现的语法：**我们常见的 insert、select、update、delete 都属于 SQL 语法，使用这几个词相当于和 DB 底层实现做了绑定。相反，我们应该把 Repository 当成一个中性的类 似 Collection 的接口，使用语法如 find、save、remove。在这里特别需要指出的是区分 insert/add 和 update 本身也是一种和底层强绑定的逻辑，一些储存如缓存实际上不存在 insert 和 update 的差异，在这个 case 里，使用中性的 save 接口，然后在具体实现上根据情况调用 DAO 的 insert 或 update 接口。
    
2.  **出参入参不应该使用底层数据格式：**需要记得的是 Repository 操作的是  Entity 对象（实际上应该是 Aggregate Root），而不应该直接操作底层的 DO 。更近一步，Repository 接口实际上应该存在于 Domain 层，根本看不到 DO 的实现。这个也是为了避免底层实现逻辑渗透到业务代码中的强保障。
    
3.  **应该避免所谓的 “通用”Repository 模式：**很多 ORM 框架都提供一个 “通用” 的 Repository 接口，然后框架通过注解自动实现接口，比较典型的例子是 Spring Data、Entity Framework 等，这种框架的好处是在简单场景下很容易通过配置实现，但是坏处是基本上无扩展的可能性（比如加定制缓存逻辑），在未来有可能还是会被推翻重做。当然，这里避免通用不代表不能有基础接口和通用的帮助类，具体如下。
    

我们先定义一个基础的 Repository 基础接口类，以及一些 Marker 接口类：

业务自己的接口只需要在基础接口上进行扩展，举个订单的例子：

每个业务需要根据自己的业务场景来定义各种查询逻辑。

这里需要再次强调的是 Repository 的接口是在 Domain 层，但是实现类是在 Infrastructure 层。

### ▐  Repository 基础实现

先举个 Repository 的最简单实现的例子。注意 OrderRepositoryImpl 在 Infrastructure 层：

从上面的实现能看出来一些套路：所有的 Entity/Aggregate 会被转化为 DO，然后根据业务场景，调用相应的 DAO 方法进行操作，事后如果需要则把 DO 转换回 Entity。代码基本很简单，唯一需要注意的是 save 方法，需要根据 Aggregate 的 ID 是否存在且大于 0 来判断一个 Aggregate 是否需要更新还是插入。

### ▐  Repository 复杂实现

针对单一 Entity 的 Repository 实现一般比较简单，但是当涉及到多 Entity 的 Aggregate Root 时，就会比较麻烦，最主要的原因是在一次操作中，并不是所有 Aggregate 里的 Entity 都需要变更，但是如果用简单的写法，会导致大量的无用 DB 操作。

举一个常见的例子，在主子订单的场景下，一个主订单 Order 会包含多个子订单 LineItem，假设有个改某个子订单价格的操作，会同时改变主订单价格，但是对其他子订单无影响：

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju9GpOruQRYGeb3wfBv2NUctDfgp2LOHyiaqqprZqQukkU9cj0U6JrTAbpYWcloQKPLqNhTm8gSFRIg/640?wx_fmt=png)

如果用一个非常 naive 的实现来完成，会导致多出来两个无用的更新操作，如下：

在这个情况下，会导致 4 个 UPDATE 操作，但实际上只需要 2 个。在绝大部分情况下，这个成本不高，可以接受，但是在极端情况下（当非 Aggregate Root 的 Entity 非常多时），会导致大量的无用写操作。

### ▐  Change-Tracking 变更追踪

在上面那个案例里，核心的问题是由于 Repository 接口规范的限制，让调用方仅能操作 Aggregate Root，而无法单独针对某个非 Aggregate Root 的 Entity 直接操作。这个和直接调用 DAO 的方式很不一样。

这个的解决方案是需要能识别到底哪些 Entity 有变更，并且只针对那些变更过的 Entity 做操作，就需要加上变更追踪的能力。换一句话说就是原来很多人为判断的代码逻辑，现在可以通过变更追踪来自动实现，让使用方真正只关心 Aggregate 的操作。在上一个案例里，通过变更追踪，系统可以判断出来只有 LineItem2 和 Order 有变更，所以只需要生成两个 UPDATE 即可。

业界有两个主流的变更追踪方案：

1.  **基于 Snapshot 的方案：**当数据从 DB 里取出来后，在内存中保存一份 snapshot，然后在数据写入时和 snapshot 比较。常见的实现如 Hibernate
    
2.  **基于 Proxy 的方案：**当数据从 DB 里取出来后，通过 weaving 的方式将所有 setter 都增加一个切面来判断 setter 是否被调用以及值是否变更，如果变更则标记为 Dirty。在保存时根据 Dirty 判断是否需要更新。常见的实现如 Entity Framework。
    

Snapshot 方案的好处是比较简单，成本在于每次保存时全量 Diff 的操作（一般用 Reflection），以及保存 Snapshot 的内存消耗。

Proxy 方案的好处是性能很高，几乎没有增加的成本，但是坏处是实现起来比较困难，且当有嵌套关系存在时不容易发现嵌套对象的变化（比如子 List 的增加和删除等），有可能导致 bug。

由于 Proxy 方案的复杂度，业界主流（包括 EF Core）都在使用 Snapshot 方案。这里面还有另一个好处就是通过 Diff 可以发现哪些字段有变更，然后只更新变更过的字段，再一次降低 UPDATE 的成本。

在这里我简单贴一下我们自己 Snapshot 的实现，代码并不复杂，每个团队自己实现起来也很简单，部分代码仅供参考：

**DbRepositorySupport**

使用方只需要继承 DbRepositorySupport：

AggregateManager 实现，主要是通过 ThreadLocal 避免多线程公用同一个 Entity 的情况

```
class ThreadLocalAggregateManager<T extends Aggregate<ID>, ID extends Identifier> implements AggregateManager<T, ID> {
    private ThreadLocal<DbContext<T, ID>> context;
    private Class<? extends T> targetClass;
    public ThreadLocalAggregateManager(Class<? extends T> targetClass) {
        this.targetClass = targetClass;
        this.context = ThreadLocal.withInitial(() -> new DbContext<>(targetClass));
    }
    public void attach(T aggregate) {
        context.get().attach(aggregate);
    }
    @Override
    public void attach(T aggregate, ID id) {
        context.get().setId(aggregate, id);
        context.get().attach(aggregate);
    }
    @Override
    public void detach(T aggregate) {
        context.get().detach(aggregate);
    }
    @Override
    public T find(ID id) {
        return context.get().find(id);
    }
    @Override
    public EntityDiff detectChanges(T aggregate) {
        return context.get().detectChanges(aggregate);
    }
    public void merge(T aggregate) {
        context.get().merge(aggregate);
    }
}

```

跑个单测（注意在这个 case 里我把 Order 和 LineItem 合并单表了）：

单测结果：

**并发乐观锁**

在高并发情况下，如果使用上面的 Change-Tracking 方法，由于 Snapshot 在本地内存的数据 * 有可能 * 和 DB 数据不一致，会导致并发冲突的问题，这个时候需要在更新时加入乐观锁。当然，正常数据库操作的 Best Practice 应该也要有乐观锁，只不过在这个 case 里，需要在乐观锁冲突后，记得更新本地 Snapshot 里的值。

**一个可能的 BUG**

这个其实算不上 bug，但是单独指出来希望大家能注意一下，使用 Snapshot 的一个副作用就是如果没更新 Entity 然后调用了 save 方法，这时候实际上是不会去更新 DB 的。这个逻辑跟 Hibernate 的逻辑一致，是 Snapshot 方法的天生特性。如果要强制更新到 DB，建议手动更改一个字段如 gmtModified，然后再调用 save。

****Repository 迁移路径****
-----------------------

在我们日常的代码中，使用 Repository 模式是一个很简单，但是又能得到很多收益的事情。最大的收益就是可以彻底和底层实现解耦，让上层业务可以快速自发展。  

我们假设现有的传统代码包含了以下几个类（还是用订单举例）：

*   **OrderDO**
    
*   **OrderDAO**
    

可以通过以下几个步骤逐渐的实现 Repository 模式：

1.  生成 Order 实体类，初期字段可以和 OrderDO 保持一致
    
2.  生成 OrderDataConverter，通过 MapStruct 基本上 2 行代码就能完成
    
3.  写单元测试，确保 Order 和 OrderDO 之间的转化 100% 正确
    
4.  生成 OrderRepository 接口和实现，通过单测确保 OrderRepository 的正确性
    
5.  将原有代码里使用了 OrderDO 的地方改为 Order
    
6.  将原有代码里使用了 OrderDAO 的地方都改为用 OrderRepository
    
7.  通过单测确保业务逻辑的一致性。
    

恭喜你！从现在开始 Order 实体类和其业务逻辑可以随意更改，每次修改你唯一需要做的就是变更一下 Converter，已经和底层实现完全解藕了。

****写在后面****
------------

感谢你，能有耐心看到这里的都是 DDD 真爱。一个问题，你是否在日常工作中能大量的利用 DDD 的架构来推进你的业务？你是否有一个环境能把你的所学用到真正实战中去？  

链接：

1、https://martinfowler.com/eaaCatalog/dataTransferObject.html?spm=ata.13261165.0.0.590a62fcaM6bCk

2、https://martinfowler.com/eaaCatalog/dataMapper.html?spm=ata.13261165.0.0.590a62fcaM6bCk

3、https://mapstruct.org/?spm=ata.13261165.0.0.590a62fcaM6bCk

END