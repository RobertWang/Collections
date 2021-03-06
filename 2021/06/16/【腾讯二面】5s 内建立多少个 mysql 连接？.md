> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247513446&idx=3&sn=0ee14b404e040ece30961694bbb44d8d&chksm=fbb13c98ccc6b58e5880ad635139ee98110832124e282b332afd5aee0356add52aaa49ff982b&mpshare=1&scene=1&srcid=0616HqqvSgCFAze5H7LxOWXc&sharer_sharetime=1623814402274&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

  

牛牛在 2020 年面试腾讯时面试官问过这样一个场景：  

以 100 每秒的速度向 mysql 写数据，持续 5s，此时我们的程序和 mysql 建立了多少个 tcp 连接？

  

*   如果负载正常的情况，mysql 1s 内一定能处理 100 个请求。
    
*   如果负载比较高，那 1s 内就处理不完，为了方便讨论，这里假设 1s 能处理 50 个请求。
    

PS: 正常实体机的 mysql，即使配置差到 1 核 1G，也完全能胜任 100/s 的单纯插入请求。只有在 mysql 本身异常，或有其他进程占用系统资源时，才会出现 1s 处理不过来 100 个请求的情况。这里的两个分支只是逻辑上的讨论。

连接池是实现连接复用的手段，和 mysql 交互时，每次需要建立一个连接，用完就会关掉，这就是短连接。如果在高并发场景，反复建立连接的成本是很高的，所以我们可以使用长连接，即连接用完后先不关闭，放到一个池子里等待复用，这个池子就叫连接池。

*   **最大连接数：**支持的最大连接数，即能打开连接的最大上限，如果超过这个阈值，就会出现 too many connections 的提示，这个数字通常可以设置得较大，以应对突如其来的峰值。
    

*   **最大空闲连接数：** 这个参数在上图有标注，表示连接池中最多有多少个空闲连接 ，某个连接做完事务之后暂时空闲，如果连接池中空闲连接数没有达到上限，即可放入连接池。该参数其实可以理解为一共可维护多少个长连接来节约连接建立的成本。
    
*   **最长空闲时间：** 连接池中连接使用完毕后会等到新的请求到来，表明了连接池中的连接在空闲时能在池子里摸鱼多久，如果长时间没有请求到来，说明请求量非常小，此时就需要释放掉连接来节省资源，等待多久，就是由该参数决定，本牛的建议是通常情况下 10-20s 就足够了。
    
*   **最小连接数：** 连接池中最小空闲连接数，当连接数少于此值时，连接池会创建新的连接来进行补充，作用主要是保持连接池始终处于就绪状态，如果需要的长连接特别多，且请求是周期性的，比如晚上才有的情况下，可以考虑使用该参数。
    
*   **初始化连接数**：初始化连接数目，实际意义不大，由最小连接数补齐即可。
    

可以看到，**最大连接数****、****最大空闲连接数****以及****最大空闲时间，这三个参数是起决定作用的**。最大连接数保证了长连接➕短连接的上限，避免了单一程序耗尽 mysql 的连接资源。最大空闲连接数决定了长连接的个数，最大空闲时间则决定了长连接的持续性。

回到该题目上看，我们利用控制变量来分析，最大连接数和最大空闲时间我们假设足够大，以保证 mysql 的正常响应和长连接的可持续性。剩下的就是 mysql 本身消费能力，和最大空闲连接数即长连接数两个维度的正交了，我们分如下情况：

1.  处理能力足够，且连接能完全复用：请求速度为 100 每秒，如果我们的最大空闲连接参数设置为 100，而 mysql 处于正常状态，每秒能完成 100 个请求 ·，则一共建立了 100 个连接。
    
2.  处理能力不足，最大空闲连接数足够大：请求以 100 每秒，如果我们的最大空闲连接数设置为 100， 而 mysql 有负载压力，每秒完成 50 个请求，这里我们假设 mysql 处理都是按先入先出，即同一秒产生的请求，因为会先复用连接池，所以连接池那部分会先处理完毕，处理流程如图所示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/HQKXnkPzzdvWA9gM71BJfLFnsNYnB5hdUpUkv5WuT9MOVzRTZyFEQIic9eomtDiap63AaREtcNiaZpX6TGFHN6ewg/640?wx_fmt=png)

圆圈中 **50:x** 表示这 50 个连接是在第 x 秒产生；长连接中 **doing** 表示准备处理中，**don****e** 表示做完；短连接默认都是 doing；红色字体表示下一秒就会处理，连线表明某个连接的前世今生。  

3.  处理能力足够，但连接池最大空闲连接数较小：请求速度为 100 每秒，如果我们的最大空闲连接数设置为 50，而 mysql 处于正常状态，每秒完成 100 个请求，则 100➕4✖50
    
4.  处理能力不足，且连接池最大空闲连接数较小：请求速度为 100 每秒，如果我们的最大空闲连接数设置为 50， 而 mysql 有负载压力，每秒只能完成 50 个请求。这种情况可按照场景 2 进行分析，欢迎有兴趣的读者在留言区交流。
    

  

  

  

  

**小结**

  

简简单单一个问题，牵引出的其实是 mysql 连接池的本质，这也是面试官面试时想考察的能力。

这个问题至少能考察出面试者两方面的能力，一是看面试者的思路是否清晰，在实际业务场景中提出来的需求，大多需要仔细思考来判断存在哪些影响因素，而在面试中留给面试者思考的时间并不多，大家的能力高下立判；第二点则是了解面试者对这些基本知识点的掌握情况，这也是对面试者的知识储备进行考察。

牛牛比较认可这种结合实际场景的面试题目，希望阅读完本篇文章之后大家能有所收获。

公众号