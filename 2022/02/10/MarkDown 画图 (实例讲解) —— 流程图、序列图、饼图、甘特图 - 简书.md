> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/77cc07f47cdc)

> 只有用过，才能掌握

写这篇文章的目的主要是**学习 markdown 画图**。目前支持画图的 MarkDown 工具只有 **Typora 和有道云笔记**

**ProcessOn VS MarkDown。**虽然 ProcessOn 也能做这件事情，但是每次使用都要以图片的形式进行编辑，不便于微调。在不同应用之间前切换也比较麻烦

**我的学习方法**。我会想出一个综合性的场景，然后把所有的知识点融入到一张图里去。这样做的好处是，回忆的时候只用回忆一张图就够了，提高回忆效率。

[语法参见官网](https://links.jianshu.com/go?to=https%3A%2F%2Fmermaid-js.github.io%2Fmermaid%2F%23%2F)

**文章目录如下：**

*   流程图代码如下：
*   代码说明：
*   typora 中效果图如下：
*   序列图代码如下：
*   代码说明:
*   typora 中效果图如下：
*   饼图代码如下：
*   typora 中效果图如下：
*   甘特图代码说明:
*   代码如下：
*   typora 中展示效果如下：

流程图
---

#### 代码如下：

```
graph TB;
subgraph 分情况
A(开始)-->B{判断}
end
B--第一种情况-->C[第一种方案]
B--第二种情况-->D[第二种方案]
B--第三种情况-->F{第三种方案}
subgraph 分种类
F-.第1个.->J((测试圆形))
F-.第2个.->H>右向旗帜形]
end
H---I(测试完毕)
C--票数100---I(测试完毕)
D---I(测试完毕)
J---I(测试完毕)


```

#### 代码说明：

<table><thead><tr><th>字母表示</th><th>含义</th></tr></thead><tbody><tr><td>TB</td><td>从上到下</td></tr><tr><td>BT</td><td>从下到上</td></tr><tr><td>LR</td><td>从左到右</td></tr><tr><td>RL</td><td>从右到左</td></tr></tbody></table>

<table><thead><tr><th>表述</th><th>说明</th><th>含义</th></tr></thead><tbody><tr><td>id[文字]</td><td>矩形节点</td><td>表示过程</td></tr><tr><td>id(文字)</td><td>圆角矩形节点</td><td>表示开始与结束</td></tr><tr><td>id((文字))</td><td>圆形节点</td><td>表示连接。为避免流程过长或有交叉，可将流程切开成对</td></tr><tr><td>id{文字}</td><td>菱形节点</td><td>表示判断、决策</td></tr><tr><td>id &gt; 文字 ]</td><td>右向旗帜节点</td><td></td></tr></tbody></table>

> 1.  T、B、L、T 分别是 Top/Bottom/Left/Right 的缩写
> 
> 2.  id 为节点的唯一标识，类似于变量。括号内为节点要显示的文字
> 
> 3.  单向箭头为流程进行方向。支持虚线与实线，有箭头与无箭头、有文字与无文字。分别是 ---、-.-、 -->、-.->、-- 文字 -->、-. 文字.->、-- 文字 ---、-. 文字.-
> 
> 4.  支持子图。如代码、效果图所示。

#### typora 中效果图如下：

![](http://upload-images.jianshu.io/upload_images/24068455-2f881f60be5c1c30.png) 流程图. png

序列图
---

序列图共有 5 个部分，分别是：**参与者、消息线、循环、选择、可选、并行、注解**。我们来模拟一个综合场景，如下图所示:

![](http://upload-images.jianshu.io/upload_images/24068455-1abd79caf00b05b9.jpg) 购买书籍序列图. jpg

#### 代码如下：

```
sequenceDiagram
Title: 小明买书
​
participant consumer as 小明
participant store as 书店
participant publisher as 出版社
​
consumer ->> store: 想买一本限量版书籍
store -->> consumer: 缺货
consumer ->> store: 隔一个月再次询问
store -->> consumer: 抢完了
loop 一个星期一次
consumer -x +store: 有货了吗
store --x -consumer: 正在订,有货马上通知你
end
​
store ->> publisher: 我要订购一批货
publisher --x store: 返回所有书籍的类别信息
​
alt 书籍类别符合要求
store ->> publisher: 请求书单信息
publisher --x store: 返回该类别书单信息
else 书单里的书有市场需求
store ->> publisher: 购买指定数据
publisher --x store: 确认订单
else 书籍不符合要求
store -->> publisher: 暂时不购买
end
​
par 并行执行
publisher ->> publisher : 生产
publisher ->> publisher : 销售
end
​
opt 书籍购买量>=500 && 库存>=50
publisher ->> store : 出货
store --x publisher : 确认收货
end
​
Note left of consumer : 图书收藏家
Note over consumer,store : 去书店购买书籍
Note left of store : 全国知名书店
Note over store,publisher : 去出版社进货
Note left of publisher : 持有版权的出版社


```

#### 代码说明:

1.  消息线

<table><thead><tr><th>类型</th><th>描述</th></tr></thead><tbody><tr><td>-&gt;</td><td>无箭头的实线</td></tr><tr><td>--&gt;</td><td>无箭头的虚线</td></tr><tr><td>-&gt;&gt;</td><td>有箭头的实线 (主动发出消息)</td></tr><tr><td>--&gt;&gt;</td><td>有箭头的虚线 (响应)</td></tr><tr><td>-x</td><td>末端为 X 的实线 (主动发出异步消息)</td></tr><tr><td>--x</td><td>有箭头的实线 (以异步形式响应消息)</td></tr></tbody></table>

> alt 可以理解为可替代的方案，可能的情况
> 
> opt 可以理解为一个 if 语句，满足条件下执行的操作

#### typora 中效果图如下：

![](http://upload-images.jianshu.io/upload_images/24068455-3a7f4fe83fdef536.png) 序列图. png

饼图
--

#### 代码如下：

```
pie
 title Pie Chart
 "Dogs" : 386
 "cats" : 567
 "rabbit" : 700
 "pig":365
 "tiger" : 15


```

#### typora 中效果图如下：

![](http://upload-images.jianshu.io/upload_images/24068455-99e43b5361e6805a.png) 饼图. png

甘特图
---

这是我用 Process 做的，我们就以张图为例，用 MarkDown 画一张甘特图吧。

顺便对比一下这两种方式，看看哪个展示效果好一点

![](http://upload-images.jianshu.io/upload_images/24068455-254b9f963e7832a5.jpg) 甘特图. jpg

#### 代码说明：

从官网上看一下**语法**，如下所示:

```
gantt
 dateFormat  YYYY-MM-DD    //底部的时间格式
 title     Adding GANTT diagram functionality to mermaid   //甘特图名称
 excludes   weekends               //周末有休息
​
 section A         //可以理解为一个功能模块
 task1        :done,des1, 2014-01-06,2014-01-08   //可以理解为这个功能模块的各项进度安排
 task2        :active,des2, 2014-01-09, 3d      //以下参数都是合法的
 task3        :des3, after des2, 5d          //我们等一下每个都试一下
 task4        :crit, done, 2014-01-06,24h
 task5        :crit, done, after des1, 2d
 task6        :crit, active, 3d
 task7        :crit, 5d
 task8        :2d
 task9        :1d
 ...         :[参数1:crit,可不填],[参数2:active,done,可不填表示待完成],[参数3:小名],
 [参数4:开始时间],[参数5:结束时间]
 section B
 section C 
 ...


```

这里要注意的是，时间的表示有多种，如上，

可以是

*   2014-01-06,2014-01-08
    
*   2014-01-09, 3d
    
*   after des2, 5d
    
*   5d
    
*   状态默认是待完成，建议给每个任务 desc
    

#### 代码如下：

```
gantt
 dateFormat  YYYY-MM-DD
 title     软件开发任务进度安排 
 excludes   weekends
​
 section 软硬件选型 
 硬件选择      :done,desc1, 2020-01-01,6w 
 软件设计      :active,desc2, after desc1,3w

 section 编码准备
 软件选择       :crit,done,desc3,2020-01-01,2020-01-29
 编码和测试软件   :1w
 安装测试系统    :2020-02-12,1w

 section 完成论文
 编写手册      :desc5,2020-01-01,10w
 论文修改      :crit,after desc3,3w
 论文定稿      :after desc5,3w


```

#### typora 中展示效果如下：

![](http://upload-images.jianshu.io/upload_images/24068455-ce4d6932263e281e.png) 甘特图. png