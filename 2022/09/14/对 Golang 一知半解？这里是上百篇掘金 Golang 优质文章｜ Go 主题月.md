> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6953073994383753223)*   1.4.1 Go 入门指南 比较适合新手，内容相对基础一些
*   1.4.2 Go 语言圣经 书如其名
*   1.4.3 Go 语言中文网 找对圈子，学的更快
*   1.4.4 菜鸟教程 这个网站非常适合快速上手某门语言
*   1.4.5 Go 语言高级编程 内容适合进阶
*   1.4.6 go 语言原本 欧神出品，虽然号称进度只有 9.9%/100%，但不妨碍它的优秀，值得一看
*   1.4.7 golang 设计模式 设计模式 Golang 实现，《研磨设计模式》的 golang 实现
*   1.4.8 Go 实战开发 作者是著名的 Go 开源项目 beego 的作者，他的最佳实践非常值得阅读
*   1.4.9 Go palyground 不用搭建本地 Go 环境，在线就编写 Go 的代码

### 1.5 Go 语言开源项目

> 1.  xgen - 编写 XSD 工具基础库，可将 XML 模式定义为多语言类型或声明的代码
> 2.  GQLEngine - 高性能 Go 语言的 GraphQL 服务端落地框架
> 3.  Orange 一款基于 Golang 语言的 Web 开发框架
> 4.  Go-admin - 基于 Golang 快速搭建可视化数据管理后台的框架
> 5.  Go-snowflake Go 语言实现的 snowflake 算法，为分布式系统实现唯一 ID，单机测试 1s 可生成 20id
> 6.  KubeVela 一个简单易用且高度可扩展的应用管理平台与核心引擎
> 7.  TiDB 见识过 mysql 性能瓶颈之后你会想要选择的一款数据库
> 8.  EasyMIDI EasyMidi 是一个简单可靠的库，用于处理标准 Midi 文件（SMF）。

### 1.6 Go 语言环境安装

下载地址： [www.golangtc.com/download](https://link.juejin.cn?target=https%3A%2F%2Fwww.golangtc.com%2Fdownload "https://www.golangtc.com/download")

GOPATH
------

二、Go 工作环境设置
===========

1. 编辑器
------

```
- Jetbrains GoLand  强烈推荐
- Jetbrains IDEA + go插件
- VS Code
- Atom
- liteide
- Sublime Text
复制代码

```

2. 依赖管理
-------

```
- glide
	- 安装：<https://glide.sh/>
	- 初始化
	- 依赖下载
- mod
复制代码

```

3. Go 源码发布
----------

三、包、函数、变量、常量、数据类型
=================

1. 包相关
------

[Go 语言 json 包的使用技巧](https://juejin.cn/post/6945023713930641445 "https://juejin.cn/post/6945023713930641445")

### 1.1 包

[拜拜了，GOPATH 君！新版本 Golang 的包管理入门教程](https://juejin.cn/post/6844903808942735368 "https://juejin.cn/post/6844903808942735368")

[Go 的包管理工具（一）](https://juejin.cn/post/6844903778945237006 "https://juejin.cn/post/6844903778945237006")

[Go 语言标准库 text/template 包深入浅出](https://juejin.cn/post/6844903762901860360 "https://juejin.cn/post/6844903762901860360")

[Go 语言闭包详解](https://juejin.cn/post/6844903793771937805 "https://juejin.cn/post/6844903793771937805")

[Go 包管理工具 govendor 使用指南](https://juejin.cn/post/6844903759886155790 "https://juejin.cn/post/6844903759886155790")

### 1.2 第三方包

### 1.3 导入语句

#### 1.3.1 分组导入语句

#### 1.3.2 多个导入语句

### 1.4 导出名

[Go 语法之包、导入包、导出名](https://juejin.cn/post/6844903695893676045 "https://juejin.cn/post/6844903695893676045")

2. 函数相关
-------

[Go 语言从入门到精通：函数](https://juejin.cn/post/6955398927067643917 "https://juejin.cn/post/6955398927067643917")

### 2.1 函数操作

[[译] 解析 Go 中的函数调用](https://juejin.cn/post/6844903474568642567 "https://juejin.cn/post/6844903474568642567")

[[译] Go 函数调用 Redux](https://juejin.cn/post/6844903474572820487 "https://juejin.cn/post/6844903474572820487")

[Go 中的 init 函数](https://juejin.cn/post/6844903864555012104 "https://juejin.cn/post/6844903864555012104")

[Golang 中函数传参存在引用传递吗？](https://juejin.cn/post/6844903618890432520 "https://juejin.cn/post/6844903618890432520")

[Golang 函数式编程简述](https://juejin.cn/post/6877505132620333064 "https://juejin.cn/post/6877505132620333064")

[从内存分配策略 (堆、栈) 的角度分析, 函数传递指针真的比传值效率高吗？](https://juejin.cn/post/6844903800042422280 "https://juejin.cn/post/6844903800042422280")

[一篇文章 说清楚 Go 语言里的函数](https://juejin.cn/post/6844904175478767624 "https://juejin.cn/post/6844904175478767624")

### 2.2 函数多返回值

[通过汇编看 golang 函数的多返回值 | 🏆 技术专题第二期征文](https://juejin.cn/post/6863781364333248525 "https://juejin.cn/post/6863781364333248525")

### 2.3 函数值

#### 2.3.1 函数的闭包

[Go 语言闭包详解](https://juejin.cn/post/6844903793771937805 "https://juejin.cn/post/6844903793771937805")

[go 学习笔记之仅仅需要一个示例就能讲清楚什么闭包](https://juejin.cn/post/6844903950504689677 "https://juejin.cn/post/6844903950504689677")

[GO - 三个方面理解闭包](https://juejin.cn/post/6844904047175008263 "https://juejin.cn/post/6844904047175008263")

[Go 语言中的闭包实现](https://juejin.cn/post/6844903465274048519 "https://juejin.cn/post/6844903465274048519")

3. 变量
-----

### 3.1 变量简介

[Golang 环境变量设置详解](https://juejin.cn/post/6844903817071296525 "https://juejin.cn/post/6844903817071296525")

[Go 初始化变量的招式](https://juejin.cn/post/6844903613005824008 "https://juejin.cn/post/6844903613005824008")

[Golang 并发之共享内存变量](https://juejin.cn/post/6844903809139900423 "https://juejin.cn/post/6844903809139900423")

[Golang 从零开始：命名规范、变量和常量](https://juejin.cn/post/6844903789988675597 "https://juejin.cn/post/6844903789988675597")

[【Go 学习之路】Go 变量](https://juejin.cn/post/6856783152921346061 "https://juejin.cn/post/6856783152921346061")

[CGO_ENABLED 环境变量对 Go 静态编译机制的影响](https://juejin.cn/post/6844903619527983111 "https://juejin.cn/post/6844903619527983111")

[golang 面试题：reflect（反射包）如何获取字段 tag？为什么 json 包不能导出私有变量的 tag？](https://juejin.cn/post/6844904191358402574 "https://juejin.cn/post/6844904191358402574")

### 3.2 变量的初始化

### 3.3 短变量声明

[GO 的短变量声明](https://juejin.cn/post/6878161651724943368 "https://juejin.cn/post/6878161651724943368")

3.4 零值
------

> 没有明确初始值的变量声明会被赋予他们的零值

[编程书说的 “Go 程序员应该让聚合类型的零值也具有意义” 是在讲什么](https://juejin.cn/post/6844904003482943501 "https://juejin.cn/post/6844904003482943501")

[Golang 零值、空值与空结构](https://juejin.cn/post/6895231755091968013 "https://juejin.cn/post/6895231755091968013")

*   零值是:
    *   数值类型为 0
    *   布尔类型为 false
    *   字符串为 ""（空字符串）
*   零值和空值的关系
*   零值的空值的区别

4. 常量
-----

### 4.1 常量

[Golang 从零开始（二）：命名规范、变量和常量](https://juejin.cn/post/6844903789988675597 "https://juejin.cn/post/6844903789988675597")

[Golang 学习——常量 const 和 iota](https://juejin.cn/post/6844904145162338317 "https://juejin.cn/post/6844904145162338317")

[golang 进阶一：类型比较，常量，nil](https://juejin.cn/post/6844904167299874830 "https://juejin.cn/post/6844904167299874830")

### 4.2 数值常量

5. 基本类型
-------

### 5.1 bool

### 5.2 string

[Go 之如何截取 string 字符串？截取英文与中文字符串](https://juejin.cn/post/6844903796930265096 "https://juejin.cn/post/6844903796930265096")

[Go 系列 string、bytes、rune 的区别](https://juejin.cn/post/6844903743524175879 "https://juejin.cn/post/6844903743524175879")

[详解 Go regexp 包中 ReplaceAllString 的用法](https://juejin.cn/post/6844903773551230990 "https://juejin.cn/post/6844903773551230990")

[Go 之 int 整数与 string 字符串相互转换](https://juejin.cn/post/6844903796913471495 "https://juejin.cn/post/6844903796913471495")

[golang 中你不知道的 string](https://juejin.cn/post/6844904016720199693 "https://juejin.cn/post/6844904016720199693")

[Go 标准库介绍一: strings](https://juejin.cn/post/6844903465928359944 "https://juejin.cn/post/6844903465928359944")

[golang 的 fmt 包 String(),Error(),Format(),GoString() 的接口实现](https://juejin.cn/post/6844903760041345032 "https://juejin.cn/post/6844903760041345032")

### 5.3 int int8 int16 int32 int64uint uint8 uint16 uint32 uint64 uintptr

[Golang 中 int，int64 和字符串互转（译文）](https://juejin.cn/post/6844904120998952967 "https://juejin.cn/post/6844904120998952967")

[原来这才是 Go Interface](https://juejin.cn/post/6844903950911553550 "https://juejin.cn/post/6844903950911553550")

[Golang interface 接口深入理解](https://juejin.cn/post/6844903555141222407 "https://juejin.cn/post/6844903555141222407")

[从 goim 定制, 浅谈 go interface 解耦合与 gRPC](https://juejin.cn/post/6844903828177829896 "https://juejin.cn/post/6844903828177829896")

[golang 面试题：能说说 uintptr 和 unsafe.Pointer 的区别吗？](https://juejin.cn/post/6844904178280562701 "https://juejin.cn/post/6844904178280562701")

[Golang 中 MulUintptr 实现原理](https://juejin.cn/post/6886045069149732877 "https://juejin.cn/post/6886045069149732877")

### 5.4 byte // uint8 的别名

[Go 系列 string、bytes、rune 的区别](https://juejin.cn/post/6844903743524175879 "https://juejin.cn/post/6844903743524175879")

[Go 之 []byte 字节数组与 string 字符串相互转换](https://juejin.cn/post/6844903796917682189 "https://juejin.cn/post/6844903796917682189")

[Strings、bytes and runes -- 就要学习 Go 语言](https://juejin.cn/post/6844903745747156999 "https://juejin.cn/post/6844903745747156999")

[go 中的 strings, bytes, runes 和 characters](https://juejin.cn/post/6844904114745245709 "https://juejin.cn/post/6844904114745245709")

### 5.5 rune unicode 码点

[Go 系列 string、bytes、rune 的区别](https://juejin.cn/post/6844903743524175879 "https://juejin.cn/post/6844903743524175879")

[Golang 中 runes 和 字符串互转（译文）](https://juejin.cn/post/6844904121003147272 "https://juejin.cn/post/6844904121003147272")

[Strings、bytes and runes -- 就要学习 Go 语言](https://juejin.cn/post/6844903745747156999 "https://juejin.cn/post/6844903745747156999")

[Golang 中 []byte, string 和 []rune 的相互转化的底层原理和剖析](https://juejin.cn/post/6931523990280208392 "https://juejin.cn/post/6931523990280208392")

### 5.6 float32 float64

### 5.7complex64 complex128

### 5.8 类型转换

[Go 基础类型转换](https://juejin.cn/post/6844903973208457224 "https://juejin.cn/post/6844903973208457224")

[golang 中的四种类型转换总结](https://juejin.cn/post/6844904113914789902 "https://juejin.cn/post/6844904113914789902")

[Golang 中一个 time.Duration 相关类型转换问题](https://juejin.cn/post/6862976040042987527 "https://juejin.cn/post/6862976040042987527")

### 5.9 类型推倒

四、流程控制语句
========

> 流程控制语句：for、if、else、switch、defer

1. 循环语句
-------

### 1.1 for

#### 1.1.1 for 循环

[昨天那个在 for 循环里 append 元素的同事，今天还在么？](https://juejin.cn/post/6875102599595425800 "https://juejin.cn/post/6875102599595425800")

[Golang 高并发编程 For 循环中使用 Goroutine 最容易犯的错误](https://juejin.cn/post/6844904133208604679 "https://juejin.cn/post/6844904133208604679")

[Go 语言性能优化 - For Range 性能研究](https://juejin.cn/post/6844903696430546952 "https://juejin.cn/post/6844903696430546952")

[[Golang] 这几个 for-range 的坑，你必须要会呀，铁汁](https://juejin.cn/post/6868963573482127374 "https://juejin.cn/post/6868963573482127374")

#### 1.1.2 初始化语句

#### 1.1.3 条件表达式

[关于变量在 if-else 条件表达式里的作用域范围](https://juejin.cn/post/6844903605367996424 "https://juejin.cn/post/6844903605367996424")

#### 1.1.4 循环条件

#### 1.1.5 后置语句

2. 判断语句
-------

### 2.1 IF

#### 2.1.1 if 的简短语句

#### 2.1.2 if 和 else

### 2.2 switch

[Go 语言流程控制：switch-case](https://juejin.cn/post/6844904145913118728 "https://juejin.cn/post/6844904145913118728")

[[译] part 10: golang switch 语句](https://juejin.cn/post/6844903827359924237 "https://juejin.cn/post/6844903827359924237")

switch 的求值顺序

3. 后置调用 - defer
---------------

### 3.1 defer

>    包含该 defer 语句的函数执行完毕时，defer 后的函数才会被执行 -`推迟调用`  
>    在一个函数中执行多条 defer 语句，它们的执行顺序与声明顺序相反。

#### 3.1.1 原理

>   推迟的函数调用会被压入一个`栈`中。当外层函数返回时，被推迟的函数会按照后进先出的顺序调用。

[Go 延迟函数 defer 详解](https://juejin.cn/post/6844903507011567623 "https://juejin.cn/post/6844903507011567623")

[go 学习笔记之咬文嚼字带你弄清楚 defer 延迟函数](https://juejin.cn/post/6844904000567902216 "https://juejin.cn/post/6844904000567902216")

五、底层数据结构
========

1. 指针
-----

>    Go 拥有指针。指针保存了值的内存地址。

[Golang 研学：在用好 Golang 指针类型](https://juejin.cn/post/6844903831365648391 "https://juejin.cn/post/6844903831365648391")

[彻底学会 Go 指针 -- 就要学习 Go 语言](https://juejin.cn/post/6844903734745513991 "https://juejin.cn/post/6844903734745513991")

[Golang 中 range 指针数据的坑](https://juejin.cn/post/6844903773559619591 "https://juejin.cn/post/6844903773559619591")

[Golang 指针：使用方法、特点 和 运算](https://juejin.cn/post/6844903725773897742 "https://juejin.cn/post/6844903725773897742")

[Go 之反射实现类型与指针拷贝](https://juejin.cn/post/6844903922205720590 "https://juejin.cn/post/6844903922205720590")

2. 结构体
------

>    一个结构体（struct）就是一组字段（field）。

[golang | Go 语言入门教程——结构体初始化与继承](https://juejin.cn/post/6850418111531712520 "https://juejin.cn/post/6850418111531712520")

[15. 理解 Go 语言面向对象编程：结构体与继承](https://juejin.cn/post/6844904161616592903 "https://juejin.cn/post/6844904161616592903")

[包罗万象的结构体 -- 就要学习 Go 语言](https://juejin.cn/post/6844903765103886343 "https://juejin.cn/post/6844903765103886343")

[Golang 自定义结构体转 map](https://juejin.cn/post/6855129007193915400 "https://juejin.cn/post/6855129007193915400")

### 2.1 结构体字段

>   结构体字段使用点号来引用

[Golang 自定义结构体转 map](https://juejin.cn/post/6855129007193915400 "https://juejin.cn/post/6855129007193915400")

### 2.2 结构体指针

>   结构体字段可以通过结构体指针来访问

### 2.3 结构体声明

>   结构体声明可以通过直接列出字段的值来新分配一个结构体。

3. 数组
-----

>   类型 [n]T 表示拥有 n 个 T 类型的值的数组。

[Go 如何对数组切片进行去重](https://juejin.cn/post/6844903967114297352 "https://juejin.cn/post/6844903967114297352")

[《快学 Go 语言》第 4 课 —— 低调的数组](https://juejin.cn/post/6844903710791843854 "https://juejin.cn/post/6844903710791843854")

[Go 切片与 C 数组转换](https://juejin.cn/post/6844903921274585095 "https://juejin.cn/post/6844903921274585095")

Go 之 []byte 字节数组与 string 字符串相互转换 [juejin.cn/post/684490…](https://juejin.cn/post/6844903796917682189 "https://juejin.cn/post/6844903796917682189")

4. 切片
-----

>   每个数组的大小都是固定的。而切片则为数组元素提供动态大小的、灵活的视角。在实践中，切片比数组更常用。

[深度解析 Go 语言中「切片」的三种特殊状态](https://juejin.cn/post/6844903712654098446 "https://juejin.cn/post/6844903712654098446")

[（正经版）面试官：切片作为函数参数是传值还是传引用？](https://juejin.cn/post/6888117219213967368 "https://juejin.cn/post/6888117219213967368")

[Go 切片使用注意事项](https://juejin.cn/post/6844903556290445325 "https://juejin.cn/post/6844903556290445325")

[如何在 Go 中使用切片容量和长度](https://juejin.cn/post/6844903998734991368 "https://juejin.cn/post/6844903998734991368")

### 4.1 切片定义

>   切片就像引用的数组，切片并不直接存储数据，它只是描述了底层数组中的一段。

### 4.2 切片文法

>    切片文法类似于没有长度的数组文法。

### 4.3 切片的默认行为

### 4.4 切片的长度与容量

### 4.5 nil 切片

[连 nil 切片和空切片一不一样都不清楚？那 BAT 面试官只好让你回去等通知了。](https://juejin.cn/post/6882534005410005005 "https://juejin.cn/post/6882534005410005005")

### 4.6 用 make 创建切片

### 4.7 切片的切片

切片可包含任何类型，甚至包括其它的切片。

### 4.8 向切片追加元素

### 4.9 range

for 循环的 range 形式可遍历切片或映射。

5. 映射（map）
----------

[gin 自动映射参数及自动校验](https://juejin.cn/post/6925701771511726093 "https://juejin.cn/post/6925701771511726093")

[PHP 转 Go 系列：map 映射](https://juejin.cn/post/6844903865431638023 "https://juejin.cn/post/6844903865431638023")

### 5.1 映射的文法

### 5.2 修改映射

五、方法和接口
=======

1. 方法
-----

*   指针接收者
*   方法与指针重定向
*   选择值或指针作为接收者

2. 接口
-----

### 2.1 接口理解

[Golang interface 接口深入理解](https://juejin.cn/post/6844903555141222407 "https://juejin.cn/post/6844903555141222407")

[Go 语言接口详解（一）](https://juejin.cn/post/6844903809211170824 "https://juejin.cn/post/6844903809211170824")

[Go 语言接口详解（二）](https://juejin.cn/post/6844903821416595464 "https://juejin.cn/post/6844903821416595464")

### 2.2 接口与隐式形式

### 2.3 接口值

#### 2.3.1 nil 的接口值

[Go “一个包含 nil 指针的接口不是 nil 接口” 踩坑](https://juejin.cn/post/6844903905797603335 "https://juejin.cn/post/6844903905797603335")

#### 2.3.2 空接口

[31. 说说 Go 语言中的空接口](https://juejin.cn/post/6844904183770906638 "https://juejin.cn/post/6844904183770906638")

### 2.4 接口类型断言

[14. Go 语言中的类型断言是什么？](https://juejin.cn/post/6844904153056034823 "https://juejin.cn/post/6844904153056034823")

[聊聊 golang 的类型断言](https://juejin.cn/post/6900192355324952589 "https://juejin.cn/post/6900192355324952589")

### 2.5 接口类型选择

### 2.6 Stringer

六、错误异常
======

1. 错误
-----

[Go 语言 (golang) 的错误 (error) 处理的推荐方案](https://juejin.cn/post/6844903757101137934 "https://juejin.cn/post/6844903757101137934")

[Golang error 的突围](https://juejin.cn/post/6844903944490057736 "https://juejin.cn/post/6844903944490057736")

[[译] Part 31: golang 中的自定义 error](https://juejin.cn/post/6844903810943418375 "https://juejin.cn/post/6844903810943418375")

[[译] Go 1.13 errors 包错误处理](https://juejin.cn/post/6844903944741896206 "https://juejin.cn/post/6844903944741896206")

七、IO 读取
=======

1. Reader
---------

[Golang 最细节篇 —— Reader，ReaderAt 的区别，你如果是做存储的，可千万别搞错了；](https://juejin.cn/post/6900534635181113352 "https://juejin.cn/post/6900534635181113352")

2. 图像
-----

[golang 图像验证码](https://juejin.cn/post/6844903541568454670 "https://juejin.cn/post/6844903541568454670") 转载

3. 阻塞 & 非阻塞
-----------

[在 Golang 中各种永远阻塞的姿势](https://juejin.cn/post/6844903598216871943 "https://juejin.cn/post/6844903598216871943")

[Golang 实现轻量、快速的基于 Reactor 模式的非阻塞 TCP 网络库](https://juejin.cn/post/6844903945907748872 "https://juejin.cn/post/6844903945907748872")

4. 同步 vs 异步
-----------

[面试官让我用 channel 实现 sync 包里的同步锁，是不是故意为难我？](https://juejin.cn/post/6844904164749918216 "https://juejin.cn/post/6844904164749918216")

[Visual Studio Live Share - 和你的队友同步共享代码，即时编辑](https://juejin.cn/post/6844903511629496328 "https://juejin.cn/post/6844903511629496328")

[kingtask：一个由 Go 开发的轻量级的异步定时任务系统](https://juejin.cn/post/6844903423624609800 "https://juejin.cn/post/6844903423624609800")

[用一个简易的 web chat 说说 Python、Golang、Nodejs 的异步](https://juejin.cn/post/6844903485494788103 "https://juejin.cn/post/6844903485494788103")

5. Select
---------

[深入理解 go-channel 和 select 的原理](https://juejin.cn/post/6844903940190912519 "https://juejin.cn/post/6844903940190912519")

八、并发
====

1. 协程相关
-------

[Golang 的 协程调度机制 与 GOMAXPROCS 性能调优](https://juejin.cn/post/6844903662553137165 "https://juejin.cn/post/6844903662553137165")

[Go 并发调度器解析之实现一个协程池](https://juejin.cn/post/6844903619163062280 "https://juejin.cn/post/6844903619163062280") 转载

2. 原子性、可见性、有序性
--------------

[Golang 并发编程核心—内存可见性](https://juejin.cn/post/6911126210340716558 "https://juejin.cn/post/6911126210340716558")

3. 并发控制
-------

[深入 golang 之 ---goroutine 并发控制与通信](https://juejin.cn/post/6844903625773285384 "https://juejin.cn/post/6844903625773285384")

[Go 译文之通过 context 实现并发控制](https://juejin.cn/post/6844903886243758093 "https://juejin.cn/post/6844903886243758093")

[go 并发之 goroutine 和 channel，并发控制入门篇](https://juejin.cn/post/6906408822701719560 "https://juejin.cn/post/6906408822701719560")

九、语言特性
======

1. Goroutine
------------

2. Channel
----------

[go 语言之行 --golang 核武器 goroutine 调度原理、channel 详解](https://juejin.cn/post/6844903634497437709 "https://juejin.cn/post/6844903634497437709")

[Golang —— goroutine（协程）和 channel（管道）](https://juejin.cn/post/6844903778617917453 "https://juejin.cn/post/6844903778617917453")

[深入理解 Golang 之 channel](https://juejin.cn/post/6844904016254599176 "https://juejin.cn/post/6844904016254599176")

3. GMP 模型
---------

[深入理解 Golang 之 channel](https://juejin.cn/post/6844904016254599176 "https://juejin.cn/post/6844904016254599176")

[Golang 调度器的 GMP 模型](https://juejin.cn/post/6879589946982662158 "https://juejin.cn/post/6879589946982662158")

[go 并发奥秘：GMP 模型](https://juejin.cn/post/6944925506340913188 "https://juejin.cn/post/6944925506340913188")

[动图图解！GMP 模型里为什么要有 P？](https://juejin.cn/post/6956008643456139301 "https://juejin.cn/post/6956008643456139301")

十、框架
====

1. Web 框架
---------

[慢聊 Go 之 Go 常见的 Web 开发框架｜Go 主题月](https://juejin.cn/post/6943568184016535566juejin.cn "https://juejin.cn/post/6943568184016535566juejin.cn")

### 1.1 Revel

### 1.2 Beego

### 1.3 Martini

### 1.4 Gin Gonic

### 1.5 Buffalo

### 1.6 Goji

### 1.7 Tiger Tonic

### 1.8 Gocraft

### 1.9 Mango

### 1.10 GORM

2. 微服务框架
--------

### 2.1 go-kit

### 2.2 Micro

### 2.3 go-zero

### 2.4 gRPC

[Go RPC 入门指南 1：RPC 的使用边界在哪里？如何实现跨语言调用](https://juejin.cn/post/6946452659159171102 "https://juejin.cn/post/6946452659159171102")

十一、性能剖析
=======

1. Go 语言性能分析
------------

[go pprof 性能分析](https://juejin.cn/post/6844903588565630983 "https://juejin.cn/post/6844903588565630983")

[Go 程序性能分析 101](https://juejin.cn/post/6844903495703740424 "https://juejin.cn/post/6844903495703740424")

[golang 使用 pprof 和 go-torch 做性能分析](https://juejin.cn/post/6844903798373089287 "https://juejin.cn/post/6844903798373089287")

[多维度思考：如何提高项目的开发时间、提高安全性、提高运行速度，从多个维度带来的一些思考。](https://juejin.cn/post/6950959850994008094 "https://juejin.cn/post/6950959850994008094")

十二、问题排查
=======

十三、Golang 面试
============

[Go 语言控制 CPU 占用率呈正弦曲线 | Go 主题月](https://juejin.cn/post/6953034008351473678 "https://juejin.cn/post/6953034008351473678")