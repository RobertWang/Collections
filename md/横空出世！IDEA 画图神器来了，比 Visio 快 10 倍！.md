> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4NzQ0Njc4Ng==&mid=2247507993&idx=1&sn=dea9ff659ccb8914e125f1e7f24d2c82&chksm=903ba474a74c2d62faac9936781eb9e00629cb7137e91dfeac1df2bec3a535b9c413a15f9ff4&mpshare=1&scene=1&srcid=0201tdF5iICbcughabkeaYsG&sharer_shareinfo=4b9507a64f577eef159b609058208dbd&sharer_shareinfo_first=4b9507a64f577eef159b609058208dbd#rd)

> 推荐一款画图神器，可以配合 IDEA 使用，画图更高效！

> 程序员在工作中，经常会有绘制时序图、流程图的需求，尤其是在写文档的时候。平时我们会选择 ProcessOn 这类工具来绘制，但有时候用代码来画图可能会更高效一点，毕竟没有比程序员更熟悉代码的了。今天给大家推荐一款画图工具 PlantUML，可以配合 IDEA 使用，画图更高效！

PlantUML 简介
-----------

PlantUML 是一款开源的 UML 图绘制工具，支持通过文本来生成图形，使用起来非常高效。可以支持时序图、类图、对象图、活动图、思维导图等图形的绘制。

下面使用 PlantUML 来绘制一张流程图，可以实时预览，速度也很快！

![](https://mmbiz.qpic.cn/mmbiz_gif/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSoYooibmwmQOWEvTrG1O59aLmRCet4ttGaVQQaQyiavLCXIMMmzicT6JTQ/640?wx_fmt=gif&from=appmsg)

安装
--

> 通过在 IDEA 中安装插件来使用 PlantUML 无疑是最方便的，接下来我们来安装下 IDEA 的 PlantUML 插件。

*   首先在 IDEA 的插件市场中搜索`PlantUML`，安装这个排名第一的插件；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSMYnr6UIv88myCsBEkmvcSwkb7mJZv8JcxHDDjxSEiaqgHnuVpicD0fcA/640?wx_fmt=png&from=appmsg)

*   有时候网络不好的话可能下载不下来，可以点击`Plguin homepage`按钮访问插件主页，然后选择合适的版本下载压缩包；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSrxftJusaENgj48tY9UOE0XTWia7HOPGcicA1qEYpINYuBdzEXCgy7Vlg/640?wx_fmt=png&from=appmsg)

*   下载成功后，选择从本地安装即可。
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSopCibibhVpC7UTyhHVCSQNSYdpiafbNyc7fWibkwW53grendFb63VehT1w/640?wx_fmt=png&from=appmsg)

使用
--

> 接下来我们使用 PlantUML 插件分别绘制时序图、用例图、类图、活动图、思维导图，来体验下 PlantUML 是不是真的好用！

### 时序图

> 时序图（Sequence Diagram），是一种 UML 交互图。它通过描述对象之间发送消息的时间顺序显示多个对象之间的动态协作。我们在学习 Oauth2 的时候，第一步就是要搞懂 Oauth2 的流程，这时候有个时序图帮助可就大了。下面我们使用 PlantUML 来绘制 Oauth2 中使用授权码模式颁发令牌的时序图。

*   首先我们需要新建一个 PlantUML 文件，选择时序图；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSb0uGibwDdXd4UR1niaebKRCkoT6YiaDJ14OEPfU7Z80SzE90iaCl73layQ/640?wx_fmt=png&from=appmsg)

*   我们可以通过 PlantUML 提供的语法来生成 Oauth2 的时序图，语法还是非常简单的，具体内容如下；
    

```
@startuml
title Oauth2令牌颁发之授权码模式

actor User as user
participant "User Agent" as userAgent
participant "Client" as client
participant "Auth Login" as login
participant "Auth Server" as server

autonumber
user->userAgent:访问客户端
activate userAgent
userAgent->login:重定向到授权页面+clientId+redirectUrl
activate login
login->server:用户名+密码+clientId+redirectUrl
activate server
server-->login:返回授权码
login-->userAgent:重定向到redirectUrl+授权码code
deactivate login
userAgent->client:使用授权码code换取令牌
activate client
client->server:授权码code+clientId+clientSecret
server-->client:颁发访问令牌accessToken+refreshToken
deactivate server
client-->userAgent:返回访问和刷新令牌
deactivate client
userAgent--> user:令牌颁发完成
deactivate userAgent
@enduml

```

*   该代码将生成如下时序图，用写代码的方式来画时序图，是不是够炫酷；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDS2AkSMOW8Bu0ic2icePiaMMicYQ1q0dqDh8Px1ke3pKIyXia4k3263CVx1uQ/640?wx_fmt=png&from=appmsg)

*   本时序图关键说明如下：
    

*   `title`可以用于指定 UML 图的标题；
    
*   通过`actor`可以声明人形的参与者；
    
*   通过`participant`可以声明普通类型的参与者；
    
*   通过`as`可以给参与者取别名；
    
*   通过`->`可以绘制参与者之间的关系，虚线箭头可以使用`-->`；
    
*   在每个参与者关系后面，可以使用`:`给关系添加说明；
    
*   通过`autonumber`我们可以给参与者关系自动添加序号；
    
*   通过`activate`和`deactivate`可以指定参与者的生命线。
    

*   这里还有个比较神奇的功能，当我们右键时序图时，可以生成一个在线访问的链接；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSk96CIqTbOjLEjNHjpUqVzkSZYmhSmovuhBnLDxGgk5EB3kDEyj7cHQ/640?wx_fmt=png&from=appmsg)

*   直接访问这个链接，可以在线访问 UML 时序图，并进行编辑，是不是很酷！
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSeb6qUMH5SRG6fDGTp7R6dFgsFjicdaSjTIzRvKYribgicjyribVd89I4sw/640?wx_fmt=png&from=appmsg)

### 用例图

> 用例图（Usecase Diagram）是用户与系统交互的最简表示形式，展现了用户和与他相关的用例之间的关系。通过用例图，我们可以很方便地表示出系统中各个角色与用例之间的关系，下面我们用 PlantUML 来画个用例图。

*   首先我们需要新建一个 PlantUML 文件，选择用例图，该用例图用于表示顾客、主厨、美食家与餐馆中各个用例之间的关系，具体内容如下；
    

```
@startuml
left to right direction
actor Guest as g
package Professional {
    actor Chief as c
    actor "Food Critic" as fc
}
package Restaurant {
    usecase "Eat Food" as uc1
    usecase "Pay For Food" as uc2
    usecase "Drink" as uc3
    usecase "Review" as uc4
}
g--> uc1
g--> uc2
g--> uc3
fc--> uc4
@enduml


```

*   该代码将生成如下用例图；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSGTZMbwQycsiczxsEQ0vx05d7ia2ycfeasia4dJuTVSTibWQRpmgmkibK1nw/640?wx_fmt=png&from=appmsg)

*   本用例图关键说明如下：
    

*   `left to right direction`表示按从左到右的顺序绘制用例图，默认是从上到下；
    
*   通过`package`可以对角色和用例进行分组；
    
*   通过`actor`可以定义用户；
    
*   通过`usecase`可以定义用例；
    
*   角色和用例之间的关系可以使用`-->`来表示。
    

### 类图

> 类图 (Class Diagram) 可以表示类的静态结构，比如类中包含的属性和方法，还有类的继承结构。下面我们用 PlantUML 来画个类图。

*   首先我们需要新建一个 PlantUML 文件，选择类图，该图用于表示 Person、Student、Teacher 类的结构，具体内容如下；
    

```
@startuml
class Person {
    # String name
    # Integer age
    + void move()
    + void say()
}
class Student {
    - String studentNo
    + void study()
}
class Teacher {
    - String teacherNo
    + void teach()
}
Person <|-- Student
Person <|-- Teacher
@enduml


```

*   该代码将生成如下类图，看下代码和类图，是不是发现和我们用代码定义类还挺像的；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSNOKZ3mSqOhibhZEzJibVmbNkXu86ao9cgWA5gCuLRgmzorUZSdAMzuPw/640?wx_fmt=png&from=appmsg)

*   本类图关键说明如下：
    

*   通过`class`可以定义类；
    
*   通过在属性和方法左边加符号可以定义可见性，`-`表示`private`，`#`表示`protected`，`+`表示`public`；
    
*   通过`<|--`表示类之间的继承关系。
    

### 活动图

> 活动图（Activity Diagram）是我们用的比较多的 UML 图，经常用于表示业务流程，比如电商中的下单流程就可以用它来表示。下面我们用 PlantUML 来画个活动图。

*   首先我们需要新建一个 PlantUML 文件，选择活动图，这里使用了 mall 项目中购物车中生成确认单的流程，具体内容如下；
    

```
@startuml
title 生成确认单流程
start
:获取购物车信息并计算好优惠;
:从ums_member_receive_address表中\n获取会员收货地址列表;
:获取该会员所有优惠券信息;
switch(根据use_type判断每个优惠券是否可用)
case(0)
    :全场通用;
    if (判断所有商品总金额是否\n满足使用起点金额) then (否)
        :得到用户不可用优惠券列表;
        stop
    endif
case(-1)
    :指定分类;
    if (判断指定分类商品总金额\n是否满足使用起点金额) then (否)
        :得到用户不可用优惠券列表;
        stop
    endif
case(-2)
    :判断指定商品总金额是否满足使用起点金额;
    if (判断指定分类商品总金额\n是否满足使用起点金额) then (否)
        :得到用户不可用优惠券列表;
        stop
    endif
endswitch
:得到用户可用优惠券列表;
:获取用户积分;
:获取积分使用规则;
:计算总金额，活动优惠，应付金额;
stop
@enduml


```

*   该代码将生成如下活动图，在活动图中我们既可以用`if else`，又可以使用`switch`，甚至还可以使用`while循环`，功能还是挺强大的；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDS7KlfuvadZfibnmu4ktyRy0dfiaSFckKVKnNLeu7SUSqK7NcyPrgU0KLQ/640?wx_fmt=png&from=appmsg)

*   本活动图关键说明如下：
    

*   通过`start`和`stop`可以表示流程的开始和结束；
    
*   通过`:`和`;`中间添加文字来定义活动流程节点；
    
*   通过`if`+`then`+`endif`定义条件判断；
    
*   通过`switch`+`case`+`endswitch`定义 switch 判断。
    

### 思维导图

> 思维导图（Mind Map），是表达发散性思维的有效图形工具，它简单却又很有效，是一种实用性的思维工具。之前在我的 mall 学习教程中就有很多地方用到了，下面我们用 PlantUML 来画个思维导图。

*   首先我们需要新建一个 PlantUML 文件，选择思维导图，这里使用了 mall 学习路线中的大纲视图，具体内容如下；
    

```
@startmindmap
+[#17ADF1] mall学习路线
++[#lightgreen] 推荐资料
++[#lightblue] 后端技术栈
+++_ 项目框架
+++_ 数据存储
+++_ 运维部署
+++_ 其他
++[#orange] 搭建项目骨架
++[#1DBAAF] 项目部署
+++_ Windows下的部署
+++_ Linux下使用Docker部署
+++_ Linux下使用Docker Compose部署
+++_ Linux下使用Jenkins自动化部署
--[#1DBAAF] 电商业务
---_ 权限管理模块
---_ 商品模块
---_ 订单模块
---_ 营销模块
--[#orange] 技术要点
--[#lightblue] 前端技术栈
--[#lightgreen] 进阶微服务
---_ Spring Cloud技术栈
---_ 项目部署
---_ 技术要点
--[#yellow] 开发工具
--[#lightgrey] 扩展学习
@endmindmap


```

*   该代码将生成如下思维导图，其实使用 PlantUML 我们可以自己定义图形的样式，这里我自定义了下颜色；
    

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwmw8Dy1IDrS1dXAEaLO6aDSqGBQjTPA1TNOyLTvibyx9j7r7KA6G3ZHWzFdBJDL2h84X6tvniceg80A/640?wx_fmt=png&from=appmsg)

*   本思维导图关键说明如下：
    

*   通过`+`和`-`可以表示思维导图中的节点，具有方向性；
    
*   通过`[#颜色]`可以定义节点的边框颜色；
    
*   通过`_`可以去除节点的边框；
    

总结
--

虽然目前可以绘制 UML 图的图形化工具很多，但是对于程序员来说，使用代码来绘图可能更直接，效率更高，尤其是配合 IDEA 使用。如果你想使用代码来绘图，不妨尝试下 PlantUML 吧。

参考资料
----

官方文档：https://plantuml.com/zh/