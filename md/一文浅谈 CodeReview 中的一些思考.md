> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dOtyMjJbE46qZv-DA0fbJw)

> CodeReview 在日常的开发过程中越来越被重视，它在提高代码质量同时促进团队成员之间的知识共享和技能提升方面发挥了诸多作用，本文将主要围绕 CodeReview 展开，简单聊聊在 CodeReview 过程中的心得和思考。

阿里妹导读

CodeReview 在日常的开发过程中越来越被重视，它在提高代码质量同时促进团队成员之间的知识共享和技能提升方面发挥了诸多作用，本文将主要围绕 CodeReview 展开，简单聊聊在 CodeReview 过程中的心得和思考。

引言

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3oUbGSJEKqq8agFDqqk6QHXJtibNlvKVZRXibxuNX71KKBftqG8lZxynw/640?wx_fmt=png&from=appmsg)

正如图中调侃的衡量代码质量的唯一有效标准就是 CodeReview 过程中 WTF/min，从中可以看出 CodeReview 对于保障代码质量的重要性。

CodeReview 在日常的开发过程中也越来越被重视，它在提高代码质量同时促进团队成员之间的知识共享和技能提升方面发挥了诸多作用，本文将主要围绕 CodeReview 展开，简单聊聊在 CodeReview 过程中的心得和思考。

CodeReview 的重要性

CodeReview 是开发过程不可或缺的重要一环，如果将代码发布比作一个工厂的流水线，那么 CodeReview 就是流水线接近于重点的质检员，他要担负着对产品质量的保障工作，将 “缺陷” 从众多的 “产品” 中挑出，反向推动 “生产方” 改进生产质量。

CodeReview 的作用：

1. **改善代码质量：**通过 CodeReview 机制，可以让团队其他同学帮忙协助把关代码质量，发现代码中潜在的质量问题，并给出改进建议，从而推动团队整体代码代码质量的提升。

2. **能够实现知识共享****，统一认知****：**CodeReview 过程是知识碰撞的过程，是学习别人的知识体系促进自我成长的过程，通过 CR 这样的过程能够将不同成长阶段的成员之间知识水位尽量拉齐，能够有效的提升团队编程的整体水平。

3. **能够及时潜在安全和性能问题等：**通过 CodeReview 能够及时发现代码中潜在的安全问题和性能问题，例如：SQL 注入问题、方案安全漏洞问题、部分业务场景查询实现性能较差等问题。

总之，通过严格的 CodeReview 能够帮助团队成员养成良好的编程习惯和规范，从提高整个团队的代码质量和团队认知拉齐。

CodeReview 实践经验

在 CodeReview 实验经验章节中，第一部分，将主要介绍 CodeReview 关注的哪些方面，通过表格的方式进行梳理和归纳；后续部分将主要围绕各个分类下的问题汇总以及从个人的视角出发提出建议。接下来，将主要围绕 2 个问题展开讨论：

1.CodeReview 应该关注那些问题；

2.CodeReview 问题点解读与建议。

**CodeReview 关注哪些**

CodeReview 关注的点比较多，这里简单做了一些归类，梳理了一部分核心关注点和场景问题。

<table width="790"><colgroup><col width="92"><col width="122"><col width="277"><col width="299"></colgroup><tbody><tr data-cangjie-key="74" data-sticky="false"><td data-cangjie-key="76" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>关注点分类</section></td><td data-cangjie-key="81" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>关注点细分</section></td><td data-cangjie-key="86" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>核心关注点</section></td><td data-cangjie-key="91" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>常见问题</section></td></tr><tr data-cangjie-key="96" data-sticky="false"><td data-cangjie-key="98" data-type="table-cell" rowspan="7" colspan="1" data-container-block="true"><section>代码规范与质量</section></td><td data-cangjie-key="103" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>命名</section></td><td data-cangjie-key="108" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" class=""><section>变量命名、方法命名、类命名、包命名</section></td><td data-cangjie-key="113" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>命名拼写错误、命名与实际含义不符（超出或小于）、用词不准</section></td></tr><tr data-cangjie-key="118" data-sticky="false"><td data-cangjie-key="125" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>注释</section></td><td data-cangjie-key="130" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" class=""><section class="">是否有注释、注释是否合理</section></td><td data-cangjie-key="135" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>通篇无注释、注释不准确</section></td></tr><tr data-cangjie-key="140" data-sticky="false"><td data-cangjie-key="147" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>日志打印</section></td><td data-cangjie-key="152" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>日志打印级别、日志打印参数、日志格式、异常打印、是否应该打印日志</section></td><td data-cangjie-key="157" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>日志打印级别误用、日志参数未打印、日志格式不规范、异常信息未打印、打印日志过多</section></td></tr><tr data-cangjie-key="162" data-sticky="false"><td data-cangjie-key="169" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>异常处理</section></td><td data-cangjie-key="174" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>异常是否抛出、是否规范的异常编码等</section></td><td data-cangjie-key="179" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>异常该抛出未抛、肆意的使用 RuntimeException 等</section></td></tr><tr data-cangjie-key="184" data-sticky="false"><td data-cangjie-key="191" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>逻辑正确</section></td><td data-cangjie-key="196" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>业务逻辑是否正常</section></td><td data-cangjie-key="201" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>空指针问题、逻辑不正确等</section></td></tr><tr data-cangjie-key="206" data-sticky="false"><td data-cangjie-key="213" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>代码风格一致</section></td><td data-cangjie-key="218" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>代码风格与应用整体风格一致</section></td><td data-cangjie-key="223" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>代码风格不一致（如）</section></td></tr><tr data-cangjie-key="228" data-sticky="false"><td data-cangjie-key="235" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>代码复杂度</section></td><td data-cangjie-key="240" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>圈复杂度</section></td><td data-cangjie-key="245" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>大量嵌套 if 导致非常复杂等</section></td></tr><tr data-cangjie-key="250" data-sticky="false"><td data-cangjie-key="252" data-type="table-cell" rowspan="3" colspan="1" data-container-block="true"><section>架构设计</section></td><td data-cangjie-key="257" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>关注分层</section></td><td data-cangjie-key="262" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>分层是否合理、是否存在跨层调用</section></td><td data-cangjie-key="267" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>分层混乱、跨层调用</section></td></tr><tr data-cangjie-key="272" data-sticky="false"><td data-cangjie-key="279" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>关注扩展性</section></td><td data-cangjie-key="284" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>扩展适配</section></td><td data-cangjie-key="289" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>硬编码扩展性低</section></td></tr><tr data-cangjie-key="294" data-sticky="false"><td data-cangjie-key="301" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>业务域划分</section></td><td data-cangjie-key="306" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>业务域划分清晰</section></td><td data-cangjie-key="311" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>业务域划分混乱</section></td></tr><tr data-cangjie-key="316" data-sticky="false"><td data-cangjie-key="318" data-type="table-cell" rowspan="2" colspan="1" data-container-block="true"><section>性能问题</section></td><td data-cangjie-key="323" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>慢 SQL</section></td><td data-cangjie-key="328" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>索引设计、是否存在慢 SQL 语法</section></td><td data-cangjie-key="333" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>索引未设计、慢 SQL 用法如 like %xxx 语句等</section></td></tr><tr data-cangjie-key="338" data-sticky="false"><td data-cangjie-key="345" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>缓存设计</section></td><td data-cangjie-key="350" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>添加缓存、是否存在缓存击穿问题</section></td><td data-cangjie-key="355" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>该加而未加缓存、缓存击穿问题等</section></td></tr><tr data-cangjie-key="360" data-sticky="false"><td data-cangjie-key="362" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>安全性问题</section></td><td data-cangjie-key="367" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>是否存在安全风险</section></td><td data-cangjie-key="372" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>文件上传验权、越权访问问题</section></td><td data-cangjie-key="377" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><section>文件上传未验权、越权访问数据等</section></td></tr></tbody></table>

**代码规范与质量**

CodeReview 过程中首先要关注代码的规范和质量问题，而且出现问题也是最多的，包括命名、注释是否合理、日志打印规范、异常处理是否合理等等，接下来主要阐述一下主要遇到的一些问题点以及哪些是合理的方案。

### 命名问题

命名是 CodeReview 阶段中常见的问题之一，开发中也经常为了一个优雅的命名而改来改去，无论是对变量还是方法、类都尽可能的给予一个恰当表示的命名。

1. 变量命名：推荐使用简洁明了带有业务语义的命名方式。不建议使用 tmp、list、a、b 等无法表述清楚当前变量是什么含义的命名方式。不建议使用下划线命名的变量名称如：_aop_signature。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3QpRnTLS1j6mapIcaoY1uW0q6WibXGIwkdpB1rcBALWaQCibrrVYGA25Q/640?wx_fmt=png&from=appmsg)

2. 常量命名：常量命名方式推荐大写字母，多个词之间通过下划线连接，如图中的常量值我们建议提取成常量定义 IS_RESHOOT。﻿﻿

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3BqPrUiaS9aGCHkrS9kicPf5qD55yCUVRcYibhuAPuAmXKjRKcmh2UwKLQ/640?wx_fmt=png&from=appmsg)﻿

3. 方法命名和类命名：采用合适的词描述清楚当前方法 / 类的功能即可，此外注意采用驼峰式命名，代码可读性更高。

比如图中的案例，原本的意图是获取语种类型，但是命名为 fillXXX 则与实际的功能不是很匹配，所以建议修改为 getXXX 命名。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3JaaQaVZOjm0rHV8cXdJU8LicG0ReKhMPAsrGRNpnGUF6lL9mQQVQ8Jw/640?wx_fmt=png&from=appmsg)﻿﻿

### 注释

CodeReview 中需要关注注释是否恰当，是否能够表达清楚当前的行 / 方法 / 类的功能，同时也要关注是否存在适当的注释，下面是我对注释的几点建议。

1. 注释恰当：能够清楚的表达当前行 / 方法 / 类的功能，不建议出现注释与代码不匹配情况。﻿

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3QedH0NbDxUIQiamoxLB5T5d5Zfqc9hGRKCvNLlSOVMGR7KiaaFQVTHvw/640?wx_fmt=png&from=appmsg)

2. 适当的注释：建议对于较为复杂的代码要提供适当的注释，都说好的代码是自注释的，但是无法保证的是有较复杂的业务背景情况下还有人能够看明白，此时需要添加部分注释。

比如，有大量复杂条件判断的 if 条件时，如果不了解业务需求的话很难从中了解这段方法是在做什么，所以建议添加适当的注释。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3DWqPdmwlfNn7KK9Cictic8gRESJiblQM9R6BBtNlNcIibUwXOlld1B77Zg/640?wx_fmt=png&from=appmsg)

### 日志打印

CodeReview 中需要关注日志打印级别是否合理，以及日志打印的格式是否符合规范，同时要注意是否有打印关键的参数，接下来是我对日志打印的几点建议。

1. 日志打印级别是否合理：建议采用适当的日志级别进行打印，比如系统异常情况采用 error 级别打印，如果只是打印流程中某一个结果可以选用 info 级别，当部分参数不符合预期的情况下可以采用 warn。

> info 级别：可以用来记录服务调用过程中的关键信息，不要盲目的打印日志。

> warn 级别：用户输入的信息不符合规范等情况。

> error 级别：系统异常, 可能需要人为干预处理等情况下。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz36P7xV2VFia3ic0BUTbmItV8w5DRbrrgXVlRqia7odWtfSLCv609UHORyQ/640?wx_fmt=png&from=appmsg)

2. 日志打印关键参数，建议在日志打印中打印出关键的参数信息，否则很难定位异常原因。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz37DxcauEVnGtjqNJchk7Z0RmEQkucwcNqjxUCQkQgGxIUf1TTW0DfoA/640?wx_fmt=png&from=appmsg)

3. 异常信息是否打印，异常处理日志打印时建议打印异常信息（e）到日志中，便于排查问题。﻿﻿

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3kjlQv7bic9Fu6ByyOqTrNogh3xzicat4QBZzD1Hu2yFBh15EoFicYvzbA/640?wx_fmt=png&from=appmsg)﻿

4. 日志是否打印过多，尤其是针对服务调用频繁的服务，需要重点关注是否有必要打印日志，高 QPS 服务容易导致日志打印过多，引发磁盘容量不足等风险。﻿﻿

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz34dlhYaZicvS5yiawYIV7ib0ATzAlEsSfibnnNiac4ial4EuqxWyI5UHyorhw/640?wx_fmt=png&from=appmsg)

### 异常处理

异常处理也是常见的问题之一，经常会遇到应该抛出异常的地方未抛出导致业务流程流转，进而导致部分流程数据异常。

比如下一个例子，在初始化二方包时会通过静态代码块的方式加载 Spring 配置文件并初始化 Spring 容器，此时如果初始化失败时 catch 住了异常并未抛出的话会导致应用启动成功，但是 Spring 容器并未初始化完成，容易导致线上问题。﻿﻿

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3mu7OibWKuJgkuj9kG4l4vvYN5lpDuOXhtU9YHPAuAFUM72h5AicbCPYA/640?wx_fmt=png&from=appmsg)

### 逻辑是否正确

要适当检查代码中是否存在逻辑性错误，比如常见的空指针问题、业务逻辑处理不正确等等。

比如这里的空指针问题，其实完全可以通过 Constants.Y.equals(object) 来完成，避免空指针引发的业务逻辑异常。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz31spnicUR0LghE6Ck9FpCVgg1QkkFYR1PA8ALKfc2KTGa5t48W74UxbA/640?wx_fmt=png&from=appmsg)﻿﻿

### 代码风格是否一致

代码风格包含很多，包括类的目录归属、参数检查的方式等等，不同的应用可能不尽相同，尽量与之前的代码结构和方案保持一致，不要我写我的你写你的，最后发现同一个应用中的 “规范” 五花八门，此时新人进来不知所措。

比如在处理异常信息返回的时候，将大量的中文编码写入业务代码中，其实是不太优雅的，推荐定义标准的异常 Code 及描述。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz39KgwSq0GcnCaBsKdiatk3H2E9icWM9ok8F1bdicRnibbVMYjG8icAcibyNUA/640?wx_fmt=png&from=appmsg)﻿﻿

### 代码复杂度

有一些代码没有逻辑问题也没有风格问题，但是看起来特别复杂，我将其归为复杂度问题，比如经常会看到有 if 嵌套过深的问题。比如，有的场景希望在集合数据非空的情况设置一些参数，所以在判断时通过 isNotEmpty 判断非空，然后在 if 里面嵌套 for 循环进行设置，此时我的建议是提前返回，如果集合为空就直接 return 返回，减少代码的复杂度。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz30UsLicBffvNDVyLLS8B3iaseHubNZYWVREOd1C2BGiaRYKSnLOgoPWStA/640?wx_fmt=png&from=appmsg)﻿﻿

**架构设计**

之前我觉得 CodeReview 中不需要关注架构设计，理论上应该在评审阶段提前确定好的，但是在实际过程中还是会有一些不合理的地方：比如分层是否合理，是否具备扩展性、业务域划分是否清晰等等。

### 关注分层

大部分应用都有自己的分层方案，增量的需求需要尽量保持一致的分层方案以使得分层架构统一，便于后期维护。比如，不同分层对应的类的命名方式有所区分，所以综合分层来看需要放在正确的分层中或者统一正确的命名方式。﻿﻿

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3gz2icEjTjibuzMBqaFhib2ZSqZiaBE1A0os3dY1sCQVBrgfMlluhBtFYUw/640?wx_fmt=png&from=appmsg)

### 关注扩展性

部分业务场景在开发的过程中由于限定于当前的业务诉求，未对后续的扩展能力做提前的规划，这种情况可以在 CR 阶段指出，引导代码编写者思考如何进行扩展性设计。

比如下面的例子，从扩展性角度来看，由于代码中设计了 OcrHandler 和 OcrAdaptor 两套扩展，OcrHandler 设计的比较薄（实际上核心是转调用 OcrAdaptor），大量的业务逻辑依然是在 OcrAdaptor，显然该设计中 OcrHandler 的实际意义并不大，所以我的建议是合并这两个扩展性设计为一套。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3D9yUtgTtmp3Kb7a0ibX4GGTEsvzQkgBTb15EV8Hqs6F7icwte15RXmpA/640?wx_fmt=png&from=appmsg)﻿﻿

### 关注业务域划分

业务域划分不清晰的系统往往更容易腐化，因为一旦业务域划分比较乱往往分不清应该放在哪个域的模块中，再加上人员迭代，会使得系统愈发的混乱。

这里简单举例，比如我们应用中有一部分接口定义放在了通用的域的服务中，其实从业务域划分上其并不够通用，显然放在这里是不合适的，我的建议是单独定义专门的业务域的包来管理该业务专门的服务接口等。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3QmWDUxFzDaLMxknm35N6FX1m20Qr2dZUWv5nvCt06TcYUfianx0kAxQ/640?wx_fmt=png&from=appmsg)﻿﻿

**性能问题**

性能问题容易被忽视，由于大量的业务代码的编写性能问题往往潜藏在大量的业务代码之后，在 CodeReview 阶段并不一定能够及时的发现问题。

### 慢 SQL 问题

慢 SQL 其实偶尔会遇到，在 CR 的过程需要关注是否有可能发生慢 SQL，此过程需要结合数据库索引设计和代码中的使用场景来共同完成定位。

举个例子，前段时间上线了一个日志服务，该服务在数据库降配后流量较大时段服务耗时飙升，经过排查是慢 SQL 导致的，核心问题在于设计的索引未能有效命中导致查询过慢，在数据库配置较高的情况下，慢 SQL 导致的时间损耗并未凸显，而当数据库降配后瞬间 CPU 被打满导致查询缓存，继而引发服务耗时的飙升。

### 关注缓存设计

部分接口在开发中并未添加缓存，如果流量较大的情况下容易导致雪崩，所以针对流量较大的场景要关注缓存设计是否合理，甚至是否有必要添加二级缓存等。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3gF2MRIQM077QBYXHhyn9iaha18JA0HUzHyOqxqqLZ17qibS0zENRI8QQ/640?wx_fmt=png&from=appmsg)﻿﻿

**安全性**

安全性是很容易忽视的问题，比如典型的无登录校验的上传接口、越权访问问题、SQL 注入风险等等。

SQL 注入：例如当前代码中使用的是字符串拼接的方案构造成完整的 SQL，然后直接调用数据库连接执行 SQL，该方案比较典型的问题就是 SQL 注入的问题，攻击者可以通过注入条件 OR 1=1 等条件即可实现对表拖库。

> SELECT * FROM users WHERE username = ''OR'1'='1'

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naIp2WODJXR2VPhuUibYSDLz3Oaxgn5ULgw03QibWicBMmtbzm8Tt4uoB4JQuLZRq6xG592olDNq3JlHQ/640?wx_fmt=png&from=appmsg)

﻿无登录权限等校验的上传接口：由于历史原因前辈们遗留了一个无登录上传的 http 接口，后被灰产利用疯狂上传文件，导致较为严重的问题，为了下线该服务耗费了大量的人力成本，如果在 CodeReview 阶段能够得到阻止，后续的问题也都可以有效避免。

越权访问问题：部分场景下如果充分信任前端传递的用户 ID，并且未充分对其进行校验的情况下直接查询对应客户的相关信息，很容易导致越权访问的问题，此类接口容易被灰产利用盗刷网站的用户信息，此类问题也是要在 CodeReview 阶段注意的点，不过此类场景可能比较少，但是风险极高。

CodeReviewer 的自我修养

我们谈 CodeReview 主要会关注代码层面的机制或者规范，而我觉得人的层面也需要关注。CodeReview 是相关尊重相互学习非常好的场景，甚至于可以认为是被推动成长的一个很好的地方，良好的自我修养，能够推进 CodeReview 文化的建设和落地，进而能够有效提升 CodeReviewer 各方的成长和代码质量提升。

**站在 CodeReviewer 角度**

作为 CodeReviewer，一方面作为代码的 "质检员"，借助于团队形成的代码规约来严格把控代码质量；另一方面，作为学习者，通过团队同学的 CodeReview 代码从中学习代码中的优秀设计。

1. 学习心态：我始终觉得代码 CR 的过程并不是一味的从代码中找问题的过程，也是相互学习的过程。从代码 CR 中汲取优秀的设计思想，学习被 CR 的代码中设计优秀的地方，每一份代码中都有一些不错的地方，内化为自己的设计理念。比如一个简单的场景：一处复杂设计的代码在当前的代码中被重构设计的更加优秀，可读性更好。

2. 专业的知识储备：在 CodeReview 之前我们需要储备对应的知识，知道为什么要这样做，为什么不能那样做，以及那样做会有什么样的坏处（如性能损耗等），了解底层的实现细节锤炼自身扎实的知识储备，此外不断加强学习新技术打开视野，拓展知识储备的边界，快速的成长。

3. 关注细节耐心 CR：作为 CodeReviewer 我们要充分的关注代码细节，包括命名、注释、异常处理、日志等等，甚至于代码中的一个空格。同时要有耐心，CodeReview 是很耗费时间和精力的，要保持耐心和长期坚持。

4. 有深度的 CR 意见：很多情况下，我们喜欢提 CR 的时候一步到位这一句要改成什么，却很少会关注为什么要这样写，对于复杂一些 / 少见一些的场景，我的建议是应该在 CR 意见中进行阐述，讲清楚前因后果，此时就成功的将知识储备输出给了被 CR 的同学，在 CR 的过程中获得了成长。

5. 尽可能了解业务背景：没有业务背景往往只能看看代码的规范性问题，然而对于业务逻辑中的问题如果没有深入的了解清楚业务诉求是比较难做出高质量的 CodeReview 意见的。

**站在被 CodeReviewer 角度**

作为被 CodeReviewer，一方面作为学习者，通过团队同学的 CodeReview 意见从中学习优秀设计思想和代码规范；另一方面，也可以作为规范的输出者，通过良好的代码设计输出自己的代码设计理念，通过 CodeReview 等场所对外输出；

1. 学习心态：面对 CodeReviewer 提出的意见，我们要保持良好的学习心态，思考下他意见背后的原因，汲取其中的设计和规范理念从中获得成长，敢于拉齐自己认知与团队内不一致的地方，快速的融入并遵从团队统一的代码规范体系。

2. 虚心接受建议：被 CodeReviewer 要学会接受建议，应该尊重别人的思考和想法，对于好的意见我们要虚心的接受，进而不断的提升自己的编程能力。

3. 有自己的主见：接受建议并不是说所有的都要接受，很多时候多种方案是都可以的，也有时候意见也有考虑不足之处，此时，被 CR 的同学需要保持主见，避免因为一味的接受别人的建议而引入了新的问题。

4. 开放包容的心态：有时候 CR 者并不一定提出的全部是合理的意见，此时，还是要保持开放包容的心态，保持友好的沟通与对方将问题讨论清楚即可。

5. 对自己的代码负责：虽然 CodeReview 能够起到保证代码质量的目的，但是作为代码的开发者尽量不要将还未测试过的代码丢给 CodeReviewer，CodeReview 的过程不能保证所有的问题都能够被发现，一定要为自己的代码负责，当确定自己的代码没有问题的时候再提交 CodeReview，这也是对 CodeReviewer 的一种尊重。

CodeReview 文化建设

团队依靠部分人参与的 CodeReview 力量是有限的，所以我们期望能够建设 CodeReview 文化。通过 CodeReview 的文化建设来充分的调动每个成员参与 CodeReview 的积极性，众人拾柴火焰高，形成文化建设应该是长期目标。﻿﻿

我们团队通过打破各自业务域范围，打造跨业务域 CR 的机制和文化，每次 CR 的时候需要当前业务域同学进行 CodeReview 的同时也要找跨领业务领域的一位同学协助 CodeReview，2 位以上的 CodeReview 都一致通过才能真正通过本次的 CodeReview，通过这样的文化和机制建设，团队也形成了 CodeReview 的风气，大家一起协助其他同学 CodeReview，有助于多个业务域的规范统一，也有助于团队层面形成良好的代码规范共识，优秀的设计经验可以跨越领域被分享和学习，促进共同成长。

关于培养团队中 CodeReview 的文化思考：

1. 建设代码规约：团队范围内的代码规约需要团队层面能够达成共识，可以是团队 leader 牵头制定，也可以由大家一起通过 CR 的过程碰撞产生，总之需要能够沉淀代码规约并在团队范围内达成共识，这是形成文化的基础。

2.CR 形式多样化：可以针对某一份代码在固定的时间（周会等）中进行分享并带领大家一起进行 Review，增强大家的参与感和趣味性；也可以跨越领域去交换 CR 别的团队的代码，学习一下别的团队的代码规范。

3. 调动积极性：鼓励团队同学积极参与，结合奖励机制不断的引导。

4. 坚守文化：文化的形成，关键还在于坚持，只有坚持长期主义才能形成良好的 CR 文化，很多团队可能会存在坚持了一阵，时间久了慢慢又回归了旧的状态，这是我们不愿看到的。

5. 适度的奖励机制：ICBU 部门层面通过设立适当的奖励机制来鼓励大家参与卓越工程，参与 CodeReview，有效的引导大家参与团队内的 CodeReview，也有效的助推文化的形成和建设。

总结

CodeReview 是代码质量保障的关键一环，作为 CodeReviewer 我们要坚守团队的统一规范，严格把控每一份代码中的质量和规范等问题，牢牢的把控好代码质量关口；同时作为被 CodeReviewer 我们也要尊重别人的时间和意见，共同维护团队的代码规范，从 CodeReview 中学习别人的意见和设计思想，促进自身的快速成长。

本文主要阐述了 CodeReview 过程中的心得和思考，良好的 CodeReview 文化能够推动团队共同成长，希望我们能够在前进的路上坚守住 CodeReview 的文化内涵，力争将工程实践做到卓越，打造面向未来有竞争力的卓越技术团队。