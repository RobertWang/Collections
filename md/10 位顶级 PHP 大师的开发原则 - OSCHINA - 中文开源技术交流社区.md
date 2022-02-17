> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.oschina.net](https://www.oschina.net/question/253614_103258)

> 在 WEB 开发世界里，PHP 是最流行的语言之一，从 PHP 里，你能够很容易的找到你所需的脚本，遗憾的是，很少人会去用 “最佳做法” 去写一个 PHP 程序。

在 WEB 开发世界里，PHP 是最流行的语言之一，从 PHP 里，你能够很容易的找到你所需的脚本，遗憾的是，很少人会去用 “最佳做法” 去写一个 PHP 程序。这里，我们向大家介绍 PHP 的 10 种最佳实践，当然，每一种都是经过大师们证明而得出的。 

**1. 在合适的时候使用 PHP – Rasmus Lerdorf**

没有谁比 PHP 的创建者 Rasmus Lerdorf 明白 PHP 用在什么地方是更合理的，他于 1995 年发布了 PHP 这门语言，从那时起，PHP 就像燎原之火，烧遍了整个开发阵营，改变了互联网的世界。可是，Rasmus 并不是因此而创建 PHP 的（ [http://www.oracle.com/technetwork/articles/index.html](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fwww.oracle.com%2Ftechnetwork%2Farticles%2Findex.html)）。PHP 是为了解决 WEB 开发者的实际问题而诞生的。 

  

> 和许多开源项目一样，PHP 变得流行，流行的动机并不能用正常的哲学来进行解释，甚至流行得有些孤芳自赏。它完全可以作为一个案例，一个解决各种 Web 问题的工具需求所引起的案例，因此当 PHP 刚出现的时候，这种工具需求全部聚焦到 PHP 的身上。 

但是，你不能奢望 PHP 可以解决所有问题。Lerdorf 是第一个承认 PHP 只是一种工具的人，并且 PHP 也有很多力所不能及的情况。 

> 根据工作的不同来选择合适的工具。我跑了很多家公司，为了说服他们部署和使用 PHP，但是这并不意味着 PHP 对所有问题都适用。它只是可以一个解决大部分问题的 front－end 脚步语言。 

作为一个 web 开发者，尝试用 PHP 解决所有问题是不科学的，同时也会浪费你的时间。当 PHP 玩不转的时候，不要犹豫，试用一下其他的语言吧。 

**2. 使用多表存储提高规模伸缩性 – Matt Mullenweg**

没有人愿意质疑 Matt Mullenweg 在 PHP 方面的权威性，他开发了这个星球上最流行的 blog 系统,(依靠一个强大的社区力量支持)： WordPress（ [http://wordpress.org/](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fwordpress.org%2F)）. 创建 Wordpress 以后，Matt 和他的团队启动了 WordPress.com 平台，一个基于 WordPress MU 的免费 blog 站点。现在，Wordpress.com 已经拥有大约 400 万用户， 这些用户每天提供超过 140,000 篇的日志。 

如果有人知道如何让网站的规模伸缩自如，这个人一定是 Matt Mullenweg。2006 年的时候 Matt 对 Wordpress 的数据结构进行了前瞻性的改进，并且解释了为什么 Wordpress MU 对每个 blog 使用独立的 MYSQL 表格， 而不是把所有的 blog 数据都塞进一个巨大的表格。 

> 我们测试过这个方法，但是发现如果要扩展它的伸缩性，代价太高。如果用一个整体的数据结构，在大流量面前，你将会面临服务器硬件的问题。在 MU 里面。用户们都被分布到独立的表格当中，并且可以轻易地组织起来。举个例子，WordPress.com 把用户的数据分散存储到 4096 个数据库中，这些数据库可以分散大规模的数据访问，实现流量和压力分流。 

数据表的可迁移性让代码（blog）可以运行得更快，并且让系统具备更强的伸缩性。依靠强大的缓存策略和灵活的数据库运用策略， Matt 向人们展示了时下最流行的 Facebook 和 Wordpress.com 都可以在 PHP 下稳定运行，并且处理惊人的访问量。 

**3. 千万不要相信用户 – Dave Child**

Dave Child 是 Added Bytes (previously ilovejackdaniels.com)  网站的核心人物，这个网站以他出色的《cheat sheets for many programming languages》（ [http://www.addedbytes.com/cheat-sheets/](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fwww.addedbytes.com%2Fcheat-sheets%2F)）而闻名。 Dave 为很多英国的公司服务，并且已经在编程世界里树立起相当的权威。 

Dave 为 PHP 开发者提供了很多深谋远虑的建议，并总结成了《writing secure code in PHP》：千万不要相信你的用户，他们甚至可能会伤害你。

  

> 有一条 web 开发的基本原则，我重复多少遍都觉得不够，那就是：千万不要相信你的用户，同时要假设你网站中的每个数据单元都是从用户那里收集来的恶意代码。很多时候，你必须用 JAVAscript 在客户端检验表单提交过来的内容， 如果你习惯了如此，那么，这是一个好习惯。如果安全性对你来说很重要，这就是最重要最需要学习的原则。 

Dave 目前正致力于为它的《Writing Secure PHP》系列书籍整理实例，书的最后他说: 

> 最后，变得偏执一点吧。除非你认为你的站点永远不会受到攻击，否则就正视所有的问题，当问题真正发生的时候，你的情况会变得很糟。你需要把每个用户都看成会带来一场攻防站的黑客，想尽一切办法来保护站点的安全，同时想好相应问题的解决方案。 

**4. 多使用 PHP 缓存 – Ben Balbo**

Ben Balbo 开发了 Site Point，一个为 developers 和 designers 提供指导的网站。他是墨尔本 PHP 开发和开源俱乐部的成员， 因此他对 PHP 有一定的了解，同时对 PHP caching （ [http://www.sitepoint.com/article/caching-php-performance/](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fwww.sitepoint.com%2Farticle%2Fcaching-php-performance%2F)）有一定的想法和经验。 

> 如果你拥有一个访问量很大，但更新并不频繁的站点（比如 blog，基于某种 CMS），或许它需要进行一些改造，这些改造不会花费太多的时间，但是对性能有突出的贡献。 如果要为一个复杂／更新频率很快的站点建立缓存机制，过程可能会很曲折，但是好处也是显而易见的。 

PHP 缓存技术有很多种，Ben 为我们推荐了如下一些: 

*   缓存函数的运行结果
*   设置过期时间
*   缓存 IE 下载的文件
*   模板缓存技术
*   Cache_Lite

由于 PHP 作为动态语言的特性，缓存机制对于更新频率并不快的站点来说非常重要。 

**5. 使用 IDE, Templates 和 Snippets 加速 PHP 开发 – Chad Kieffer**

当 Chad Kieffer 从 UI 设计和数据库优化的工作中抽身出来的时候，他会在他的博客 2 tablespoons 上分享很多技术经验。由于 Chad 多方面的全面发展，他经常可以发现其他程序员不能发现的问题，并形成相关经验，尤其是他开发网站的方法。他参与了网站开发的各个环节，因此他的建议对于提高网站开发的大局观非常有用。 

Chad 认为使用 Eclipse PDT(Eclipse’s PHP development package)  这样的 IDE，同时使用一些模板技术和开源项目可以有效地提高 PHP 的开发速度。 

> 紧凑的计划，长长的 to do lists 以及 deadlines 让开发人员非常苦闷。不过有些功能，比如 Eclipse Templates，可以有效减少编码的时间和出错的几率。 

通常来说，任何项目都可以自动化，自动化程度越高， 你完成项目的时间就越短。花时间来开发使用频率很高的框架和模板，将会节省你以后更多时间。同时，使用像 Eclipse and the PDT package 这样的 IDE，你会发现效率得到明显提高，IDE 可以自动闭合，补全分号并且可以在本地 debug。 

**6. 利用好 PHP 的过滤函数 – Joey Sochacki**

或许 Joey Sochacki 并不像 Matt Mullenweg 那样有名 ，但他也是一个经验丰富的开发者，并且通过他的博客 Devolio 分享了很多技术经验 。

Joey 发现在编写 php 代码的过程中有很多地方需要进行过滤，但却并没有太多的 coder 关注 php 的内置过滤函数。 

> 过滤数据是我们经常需要做的事情，但是很多功能丰富的 PHP 内置过滤函数却不为人知。使用类似 filter_*  的 PHP 内置函数，我们几乎可以处理所有的过滤任务，包括数据类型验证／URL／email 和 IP 地址验证／特殊字符处理等等。 

过滤是一件复杂的事情，但是我相信 joey 的发现会给你很多启发，让你认识到 PHP 强大的过滤功能。 

**7. 使用 PHP 框架 – Josh Sharp**

对于是否应该使用 Zend, CakePHP, CodeIgniter, 或者 其他 PHP 框架，一直存在着很多争议，但是在 web 开发者的心中，他们有自己衡量的标准。 

Josh Sharp 自己创建了一家提供面包和黄油服务的网站，因此他对于使用 PHP 框架来开发网站有一定的经验。他认为使用一个 PHP 框架来进行项目开发（use a PHP framework ），可以有效地节省时间，并且减少出错的几率。为什么？因为他觉得 PHP 实在是太好上手了。 

> PHP 的易于使用有时候也有缺陷，因为并不严格的语法，经常会导致很多错误代码的诞生。但如果使用一个 PHP 框架，出错的几率就会大大减少。 

PHP 框架可以让你的代码结构更加规范，并且节省大量时间。 

**8. 不要使用 PHP 框架 – Rasmus Lerdorf**

与 Josh 的观点恰恰相反，PHP 的鼻祖 Rasmus Lerdorf 却认为最好不要使用 PHP 框架，为什么？因为不基于框架的 PHP 性能更好。Rasmus 在 Drupalcon 2008 的演讲上，用 “Hello World” 的例子来对比了一些框架 PHP 和简单 PHP 之间的性能，结果显示框架 PHP 的性能要远远落后。 

**9. 使用批处理 – Jack D. Herrington**

Jack Herrington 对 PHP 世界并不陌生， 并且为大名鼎鼎的 IBM developerWorks 贡献过超过 30 篇的专搞， 同时出版过《PHP Hacks》的书，因此他是一个真正的专家。 

Herrington 推荐使用批处理和 Cron 来代替那些可以运行在后台的程序脚步，Web 用户并不愿意在线 等待你的处理过程，所以有些事情更适合放到后台来处理。 

> 诚然，在某些情况下，这有点大材小用了，但是你可以清楚地看到，使用 Cron, MySQL, PHP 面向对象的方法以及 Pear::DB 这些便捷的工具来创建一个批处理工具并不是一件复杂的事情。 

Jack 认为使用 cron, PHP 和 MySQL 在后台处理一些任务，比起多进程的业务逻辑要划算得多。 

> 两种方法我都尝试过，我认为 Cron 非常符合 ”Keep It Simple, Stupid” (KISS)  的原则，它让后台处理变得简单。与多进程的业务逻辑相比，它没有内存溢出的风险。你可以创建一个简单的批处理脚本，并且在 cron 中运行，这个脚本会定时检查是否有任务需要处理，处理完之后就会自动退出，因此你不用担心是否有进程卡壳，或者陷入死循环。 

**10. 及时启用错误报告 – David Cummings**

David Cummings 有一个专门提供 CMS 软件服务的公司 ，并且获得过几次奖 ，他有非常丰富的 PHP 开发经验。David 曾经写过《two PHP tips he wished he’d learned in the beginning》，其中一点就是：及时启用错误报告，这会节省大量的时间。 

> 我告诉人们，最重要的事情就是最大程度地开启 PHP 的错误报告，为什么？因为 PHP 可能会隐藏很多小问题： 

  

*   变量没有预定义
*   在代码片段中引用了不可用的变量
*   使用了未定义的常量这些因素看起来并不是什么大事，除非你在使用面向对象的方法编写一些类库。通常，关闭错误报告将可能使你付出更大的成本来维护你的代码。

错误报告可以帮你轻易地找到代码的问题所在，如果错误报告的等级够高，细微的错误都能被立即发现，帮助你节省整体 debug 的时间。 

英文原文： [10 Principles of the PHP Masters](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fnet.tutsplus.com%2Ftutorials%2Fphp%2F10-principles-of-the-php-masters%2F)