> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wnKMO-03GSvZxv1FMxsysw)

导读：在平时工作中，我们会把通用的代码，合并到一个通用的 SDK 中，增加大家工作效率，本文主要分享我们在编写 SDK 时候的准入标准以及相关编码思想。

1.  避免重复造轮子
    
2.  减少线上 bug 概率
    

避免重复造轮子：

好的 sdk 可以帮助团队省时省力，将相同的功能抽象到一个通用 sdk 中，前人栽树后人乘凉。

### 

**减少线上 bug 概率：**  

1.  经过大家共同的优化出 bug 的可能性较低，即使出 bug，也只需要修改 sdk 即可；
    
2.  若每个代码库都实现一遍，那每个代码库就都需要修复这个 bug；
    
3.  每个同学能力不同，写出代码的质量参差不齐；
    

**一、遵循的设计理念 SOLID**

在程序设计的领域内，SOLID 是由 Robert C.Martin 提出的面向对象编程以及面向对象设计的五个基本原则的缩写，这五个原则分别是：

*   单一职责原则（Single Responsibility Principle）
    
*   开闭原则 (Open Close Principle)
    
*   里氏替换原则 (Liskov Substitution Principle)
    
*   接口隔离原则 (InterfaceSegregation Principle)
    
*   依赖反转原则 (Dependence InversionPrinciple)
    

在编写 SDK 中，我们坚信好的代码是设计出来的，而不是编写出来的这一理念，所以在设计之初我们就会按照这五大原则，逐一考量我们是否设计得足够优秀，我们是否违背了某项原则。

接下来我们来逐一介绍我们对于这五大原则的理解：

**1.1、单一职责原则**

#### 定义

类的设计，尽量做到只有一个可以引起它变化的原因

#### 理解

单一职责原则的存在是为了保证对象的高内聚、保证同一对象的细粒度。在程序的设计上，如果我们采用了单一职责原则，那么就要注意，开发代码过程中不要设计那些功能五花八门、包含很多不相关功能的类，这样的类我们可以认为是违反单一职责原则的。按照单一职责原则设计的类中，应该有且只有一个改变因素；

#### 举例子

```
package phone

// 只负责调整电话音量
type MobileSound interface {
    // 音量+
    AddSound() error
    // 音量-
    ReduceSound() error
}
// 只负责调整照片格式
type MobilePhoto interface{
    // 亮度
    Brightness() error
    // 纵横比
    AspectRatio() error
}
```

这段代码是对于手机的两个类的构造，在设计的时候，我们将手机的音量控制和手机图片的调整类进行了区分，在同一个类中不同功能的共同载体分别是音量控制和图片调整，这两个类中的功能是不产生依赖的，相互平行的关系，当手机的配置发生改变，那么这两个类中的功能都会产生一定的变化；

如果我们要对其中一个类进行调整，那么只需要在相应的 interface 中进行功能的优化即可，不会影响到其他功能的结构。

**1.2、开闭原则**

定义：一个软件实体，应该对扩展开放，对修改关闭；

#### 理解

针对于这个问题的产生是来自于对于代码进行维护的时候，如果对旧代码进行修改，那么可能会引入错误，这可能会导致我们对整个功能进行重新架构，并且重新测试；

为了防止这种情况的发生，我们可以采用开闭原则，我们只对代码进行添加功能，扩展类中的功能，而不去修改原先的功能，这样可以避开因为修改老代码而产生的坑（深有体会）；

#### 举例子

```
// 只负责调整电话音量,同时具备一键最大和一键最小的功能
type MobileSound interface {
    // 音量+
    AddSound() error
    // 音量-
    ReduceSound() error
    // 静音
    Mute() error
    // 一键最大
    MaxSound() error
}
```

还是这个手机的类，如果后期业务需求增加，要求我们加入静音和一键音量最大的功能，那么我们可以直接在这个音量控制接口中进行功能扩展，而不是去在音量加减的功能里进行修改，这样可以避免因为对内部逻辑的调整而产生的功能架构问题。

#### 注意点

开闭原则是一个非常虚的原则，我们需要在提前预期好变化并做出规划，然后需求的变化总是远远超过我们的预期，遵循这个原则我们应该把频繁变化的部分做出抽象，但却不能将每个部分都刻意的进行抽象；

如果我们无法预期变化，就不要去做刻意的抽象，我们应该拒绝并不成熟的抽象

**1.3、里氏替换原则**

#### 定义

所有引用基类的地方必须能透明的使用其子类的对象；

#### 理解

子类对象替换父类对象的时候，程序本身的逻辑不能发生变化，同时不能对程序的功能造成影响；

#### 举例

仍然是用手机来举例子，我们定义的手机类中，加入了一个充电功能接口；

*   其中包含了无线充电和有线充电两个功能；
    

```
type Charge interface {
    //有线充电
    ChargeWithLine() error
    //无线充电
    ChargeWithoutLine() error
}
```

*   但是我们并没有想到 8848 钛金手机这么一款高端商务上流手机并不具备无线充电的功能；
    

因为我们定义的父类并不能由子类完全替换，8848 也是手机，但是因为它的功能并不完全具备手机类的功能，所以导致这个问题的产生，那么如何来解决这个问题呢？

我们可以将手机类的充电功能进行拆分，在父类中，我们只定义充电功能，那么我们就可以在 8848 子类里面设计具体的充电方式，来完善这个功能的接口。我们在手机类中仅定义了最基础的方法集，通过子接口 SpecialPhone 添加 ChargeWithLine 方法；

```
type MobilPhone interface {
    Name() string
    Size() int
}

type NormalCharge interface {
    MobilPhone
    ChargeWithLine() error
    ChargeWithoutLine() error
}

type SpecialCharge interface {
    MobilPhone
    ChargeWithLine() error
}
```

*   我们再通过 SpecialPhone 来提供对 MobilPhone 的基本实现：
    

```
type SpecialPhone struct {
    name string
    size int
}

func NewSpecialPhone(name string, size int) *SpecialPhone {
    return &SpecialPhone{
        name,
        size,
    }
}

func (mobile *SpecialPhone) Name() string {
    return mobile.name
}

func (mobile *SpecialPhone) Size() int {
    return mobile.size
}
```

*   最后，Mobil8848 通过聚合 SpecialPhone 实现 MobilPhone 接口, 通过提供 ChargeWithLine 方法实现 Mobil8848 子接口：
    

```
type Mobil8848 struct {
    SpecialPhone
}

func NewMobil8848(name string, size int) MobilPhone {
    return &Mobil8848{
        *NewSpecialPhone(name, size),
    }
}

func (mobile *Mobil8848) ChargeWithLine() error {
    return nil
}
```

#### 注意点

在项目中，采用里氏替换原则时，尽量避免子类的特殊性，采用该原则的目的就是增加健壮性，在版本升级时也能有很好的兼容性，而一旦有了特殊性，那么代码间的耦合关系就会变得极其复杂；

**1.4、接口隔离原则**

#### 定义

*   客户端不应该依赖它不需要的接口；
    
*   类间的依赖关系应该建立在最小的接口上；
    

#### 理解

*   客户端不能依赖于它所不需要使用的接口，这里的客户端我们可以理解为接口的调用方或者使用者；
    
*   尽量保证一个接口对应一个功能模块；
    

接口隔离原则和单一职责原则的区别是什么？

单一职责原则主要是在业务逻辑层面上，保证一个类对应一个职责，注重的是对于模块的设计；而接口隔离原则则是主要针对接口的设计，对不同的模块提供不同的接口。

#### 举例

比如下面这个例子，这段代码就是清晰的将接口按照不同的返回值进行了区分，这样在调用的时候就可以避免因为不清楚返回的认证信息状态而造成的逻辑上的错误。

```
// CarOwnerCertificationSuccessScheme 返回认证成功的scheme
func CarOwnerCertificationSuccessScheme() string {
    return CertificationSuccessScheme
}

// CarOwnerCertificationFailedScheme 返回认证失败的scheme
func CarOwnerCertificationFailedScheme() string {
    return CertificationFailedScheme
}

// CarOwnerMessageRecallScheme 返回车主认证邀请的scheme
func CarOwnerMessageRecallScheme() string {
    return MessageRecallScheme
}
```

#### 注意点

同单一职责原则类似：

*   在接口设计时，如果粒度太小会导致接口数据太多，开发人员被众多的接口淹没；
    
*   如果粒度太大，又可能导致灵活性降低。无法支持业务变化；
    
*   我们需要深入了解业务逻辑，不可盲目照抄大师的接口；
    

**1.5、依赖倒置原则**

#### 定义

高层模块不能依赖底层模块，两者都应该依赖于其抽象；抽象不应该依赖细节；细节应该依赖抽象；

#### 理解

1.  好的抽象更稳定，可以覆盖到更多的细节；
    
2.  具体的业务则要面临着各种方法和细节，非常多样，是不确定不稳定的；
    

#### 举例

比如下面这个例子，如果我们想要充实手机的功能，那么我们在定义的时候，对手机这个类只定义抽象类，类里面并不涉及到具体的功能设计，具体的功能设计我们要放在功能里面，这样就可以保证类结构的稳定，不会因为功能的调整而导致整个手机类的调整。

```
package phone

type Phone interface{
    // 音量调整
    MobileSound

    // 照片调整
    MobilePhoto
}

// 只负责调整电话音量
type MobileSound interface {
    // 音量+
    AddSound() error
    // 音量-
    ReduceSound() error
}
// 只负责调整照片格式
type MobilePhoto interface{
    // 亮度
    Brightness() error
    // 纵横比
    AspectRatio() error
}
```

#### 注意点

依赖倒置最难支持就是找到抽象，好的抽象可以减少大量的代码，糟糕的抽象，就突增工作量。

关于设计理念，我们坚信好的系统是设计出来的，而不是开发出来的。所以在任何项目的启动开发之前，我们都会进行极其严格的设计评审。

**二、遵循的编码原则**

**2.1、稳定、高效**

公共库是提供给众多业务方使用的第三方组件，如果公共库运行时程序崩溃，会危及业务方的项目，可能会造成线上事故，所以稳定是一个公共库的基本保证。

**暴露异常** 

异常可以通过日志和接口返回暴露给用户。对于异常情况一定要打日志，方便用户排查具体问题，并且约定错误码，通过错误码和 error message 将错误信息暴露给用户。在公共库内部，所有可能返回错误的地方都不能忽略。

参数校验 

用户很有可能无意中给你封装的方法传入了不合法的参数，你的方法要能处理任何不合法的参数，让你的公共库不至于因为传入不合法的参数造成崩溃。

**Panic 和 Recover** 

在 Go 语言中，一些非法的操作会导致程序 panic，例如访问超出 slice 边界的元素时，通过值为 nil 的指针访问结构体的字段，关闭已经关闭的 channel 等。当一个 panic 在调用栈中的没有被 recover 时，程序终将因堆栈溢出而终止。在编写公共库时，我们不希望某个地方出现异常就导致整个程序崩溃。正确的做法是 recover 这个 panic，转换成 error 返回给调用方，同时输出到日志，使得公共库及整个工程项目还能保持正常运行。如示例代码：

```
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

> 在每个对包外公开的函数和方法都应该 recover 到内部的 panic 并且将这些 panic 转换为错误信息，防止内部 panic 造成程序终止.

### **2.2、测试**

封装好一个函数或者功能模块后，我们需要测试其逻辑的正确性和函数的性能，这就需要单元测试和性能测试。单元测试的重点在于验证程序设计或实现的逻辑是否正确，而压力测试的重点在于测试程序性能，确认程序在高并发的情况下还能保持稳定运行。

单元测试

单元测试就是针对某一个函数或者进行测试，通过各种可能的样例（即函数的输入和期望的输出）验证函数各分支是否都得到了预期的结果，或者可能的问题都能被预知并提前处理。通过单测能解决以下问题：

*   提早发现发现程序设计或实现上的逻辑错误，便于及时定位解决，确保每个函数是可运行的，运行结果是正确的。
    
*   一次编写，多次运行。编写好一个函数的单元测试后，后续对函数的修改或重构，都可以使用此前编写的单测来确保修改后的函数依然保持正确的逻辑，防止手动测试遗漏一些边界情况，给代码留下隐患。
    

压力测试

压力测试的重点是测试程序在高并发的情况是否还能保持稳定运行，以及测试程序能够承受的并发量。公共库除了保证逻辑正确外，还应该保证其在高并发的情况下还能正常运行。

**2.3、保持向下兼容**

公共库应该版本向下兼容，即使更新了公共库版本的用户，其此前使用旧版本公共库的项目代码还能正常运行。

*   公共库对外暴露的接口应该避免改动，其函数签名不应该再改动，对函数逻辑的修改应该让用户无感知。
    
*   如果此前的接口确实无法满足需要，那么可以升级一个库版本，在新版本中开发一个新的接口，而不是在原版本中直接对接口进行修改。
    

**2.4、减少外部依赖**

公共库应该尽量减少外部依赖，应该避免使用不稳定的外部依赖。在封装公共库时，也应该精简代码与功能，以最精简的形态提供最核心的功能，避免公共库的体积过大。

**2.5、易用性**

#### 统一调用

SDK 应主动封装复杂的调用方式，屏蔽底层复杂逻辑，尽量提供给使用方简单的调用形式。让使用者能在几行代码中实现功能，减少使用者在调⽤的流程和对参数的理解成本。同时提供友好的提示，便于使用者调用调试。

这些需要在接口设计阶段做到极致，不仅要调用简单，又在提供基础功能的基础上，支持扩展定制化。例如，我们在工程中，通常会将仅需一次加载，整个工程生命周期生效的参数配置化，以一个配置文件作统一管理。但实际上配置文件的格式分多种，例如 yaml、json、xml 等，可扩展性设计在对配置内容的含义、默认的配置项详细说明的基础上，支持多类配置文件的加载...... 等。

在接口设计时统一风格，可以给使用者留下专业的印象，同时它也可以传递出开发者 SDK 的设计理念。当出现跨平台同功能的使用时，可沿用平台风格，通过延续风格，让使用者更直观的调用 SDK 功能。

**2.6、可理解性**

#### 目录结构

代码的目录结构可以定义整个 SDK 的层次架构，我们可以通过目录大致知晓其内部实现围绕的主题。在目录结构定义上，应命名上直接突出主题，拆分明确，尽量不存在交集，不应让人去猜或等读完源码才知道在做什么。

好的拆分能避免重复代码并方便使用者查找，例如 log、doc、util、config、test、build 等能让人一眼明确该目录下所做的工作。同时目录层次也尽量不要太深，太深会一定程度上加重使用者负担。

```
.
├── config   //配置文件
│   ├── agent.json
│   ├── alarm.json
│   ......
├── docs  //说明文档
│   ├── introduce
│   │   └── index.html
│   ├── api
│   │   └── index.html
│   ├── index.html
│   ├── LICENSE.md
│   ......
── test  //单元测试
│   ├── util
│   │   ├── common
│   │   │   ├── date_test.go
│   │   │   └── md5_test.go
│   ......
├── util  //通用工具方法
│   ├── common
│   │   ├── date.go
│   │   ├── md5.go
│   ├── errno
│   │   └── errno.go
│   ......
├── modules  //核心模块
│   ├── agent //监控agent
│   ├── alarm //告警
│   ......
├── README.md  //代码库说明
├── script  //脚本文件
│   └── mysql
│   │   └── init-db.sql
│   ├── build.sh
│   ......
├── cmd  //工程启停操作等
│   ├── start.go
│   ├── stop.go
......
```

#### **2.6、统一风格**

SDK 支持的功能通常是由多人合作完成，相信每位同学都有自己独有的代码风范。SDK 是对外提供的工具包，是一个整体，若在内容风格上跳跃太大，容易造成不严谨等...... 的负面体验。好的体验是让使用者在使用或学习过程中无感内部差异。在这里，我们主要从代码、注释、文档三方面来介绍：

**代码风格**

代码的书写风格，常见的如：代码缩进、换行（if…else…、for、函数等的 "{" 是否换行）、空行 (函数与函数、变量、注释等之间空几行)、命名方式 (驼峰还是下划线，转换函数的命名用 AtoB 还是 A2B...)。

```
// 文件名(驼峰或下划线)：diskstats.go / disk_stats.go

// 结构体SafeAgents 与 变量Agents 应间隔几行
// 函数 Put与NewSafeAgents 间隔1行 or 更多

type SafeAgents struct {
	sync.RWMutex
	M map[string]*model.AgentUpdateInfo
}

var Agents = NewSafeAgents()

func NewSafeAgents() *SafeAgents {
	return &SafeAgents{M: make(map[string]*model.AgentUpdateInfo)}
}

func (this *SafeAgents) Put(req *model.AgentReportRequest) {
	val := &model.AgentUpdateInfo{
		LastUpdate:    time.Now().Unix(),
		ReportRequest: req,
	}
	......
}
```

代码风格方面还有其他许多注意事项，此处就不一一介绍了，总体说来，统一代码风格，能让整个 SDK 代码库看起来更整洁。

**注释风格**

注释是对当前代码的解释，好的注释应让使用者在不阅读源码的情况下，知晓当前代码的意图。注释的内容应简洁明了，突出重点。SDK 中包含的注释类型分多种:

1.  出现在文件头部的注释，申明当前文件内容的描述、修改信息等；
    
2.  函数的注释，说明当前函数的功能、参数、返回值、抛出的异常等；
    
3.  重要的变量名常量、枚举值的含义注释等；
    
4.  代码内部重要的逻辑注释，例如算法的实现、重要的条件分支等。
    

在每一类注释下，会出现多种注释风格，以函数 ToSlice 与 HistoryData 为例，ToSlice 注释重点说明函数的功能，HistoryData 用 @param、@return、@desc 特殊符对函数的功能，参数，返回值进行说明。

```
C语言风格的/* */块注释,也支持C++风格的//行注释

// ToSlice 转换
func (this *SafeLinkedList) ToSlice() []*model.JudgeItem {
	this.RLock()
	defer this.RUnlock()
	sz := this.L.Len()
	if sz == 0 {
		return []*model.JudgeItem{}
	}

	ret := make([]*model.JudgeItem, 0, sz)
	for e := this.L.Front(); e != nil; e = e.Next() {
		ret = append(ret, e.Value.(*model.JudgeItem))
	}
	return ret
}
// @desc 获取历史数据
// @param limit 本次请求至多返回的历史数据量，如果未达到limit限制，返回已有的所有数据量
// @return bool isEnough 数据未达到limit量，false；
func (this *SafeLinkedList) HistoryData(limit int) ([]*model.HistoryData, bool) {
	if limit < 1 {
		// 其实limit不合法，此处也返回false吧，上层代码要注意
		// 因为false通常使上层代码进入异常分支，这样就统一了
		return []*model.HistoryData{}, false
	}
	......
}
```

我们不对以上列举的注释风格进行评价，仅举例说明注释风格的差异性。实际上我们可以在 SDK 开发启动前，借助第三方工具配置每一类注释模版及快捷键，这样可在一定程度上约束开发者，统一注释风格。

**文档风格**  

文档风格对于项目作者而言，编写项目文档的过程中，可以为自己的项目设计进行逻辑梳理，为自己的编码逻辑打好草稿。因为修改文档的成本，要比修改代码的成本低的多。

拥有好的文档也可以降低项目的维护成本，文档可以作为项目的维护指南，也便于项目的迭代和交接。

对于读者而言，一个好的项目文档，可以清晰地获取项目作者的思考过程，在涉及到项目合作沟通沟通的场景下，能够很好地提高交流的效率。

通过项目的文档，可以让读者了解：

1.  项目的背景
    
2.  项目可以提供什么能力 (使用场景)
    
3.  项目能够解决什么问题
    
4.  ......
    

文档又分为设计文档、用户文档等：

1.  设计文档
    

```
1) 需求分析文档 - 研发对需求的理解
2) 系统设计文档 - 研发人员设计系统的思路
3) 接口设计文档 - 研发人员设计接口的思路
4) ......
```

2.  用户文档
    

```
1) 产品使用文档
2) 操作指南
3) ......
```

3.  ......
    

SDK 文档的维护不仅关系到使用人员的接入体验，同时也能减少开发人员对业务咨询问题的处理负担。制定标准写作规范和格式规范能增加文档的可理解性。例如：

1.  格式规范（字体大小颜色、标题、特殊表示...)
    
2.  图表规范（图表统一用某一软件作图，并在某一目录保存原文件，方便以后维护同学更新...）
    
3.  文案规范（禁含糊用词、断句、中英标点混用...）
    
4.  ......
    

规范的文档更具专业性与权威性。

其实现已有许多成熟的工具包可以帮助我们检查 go 代码中的不规范，例如常用的 Golint，它定义的只能采用驼峰命名方法，一些特殊的变量必须大写 (如 ID、IP...)，全局变量 & 函数 & 结构体必须有注释... 等等，一定程度上帮助我们约束了编码习惯。在许多工具的选择和使用中，找到适合自己的才是最有效的，同时我们也可以根据团队自有特色制定自己的约束标准。

**三、实践经验**

**3.1、代码准入原则**

#### 所有的 commit, 必须关联内部需求卡片

在内部需求管理中，我们一般会通过内部工具来管理需求、bug 等信息，每次开发人员提交的 commit 信息大多比较简洁，无法从展示出详细的信息，所以我们要求研发同学在提交代码时，必须关联卡片，方便后续追溯代码、定位问题

#### 引入 golint 代码风格检查, 强制要求开发人员通过 golint 代码风格校验

为保证代码风格统一，内部 sdk 库统一使用 golint 工具进行初步验证，无法通过 golint 检查的代码无法合入，从机制上解决代码风格不一致的问题

#### git 分支版本管理

由于内部 sdk 代码库，需求相对独立，代码间冲突较少，所以我们并未强制大家必须拉分支后才能合入，为简化提交规则，小型的代码提交，例如改动某个函数等，可以直接在 master 上进行改动。但大型项目，则必须拉分支

#### CR 机制，成立内部 sdk 委员会

内部评审较多，且大家都是在工作之余进行代码 cr，若由 1-2 名同学负责维护，则难以保证 cr 速度以及质量，所以我们成立了 CR 委员会，每个委员会的同学都有权限对代码进行 + 2（准入）评审权限，但定期 SDK 委员会将会组织 code reivew，以保证代码库的可维护性

**四、总结**

一个好的 SDK 可以减轻工作量、降低平台使用门槛、增加系统健壮性等多个好处，在本文中我们分别从设计模式、代码原则、文档、注释等多个方面描述了我们是如何在百度内部维护好一个 SDK 库，想要写出一个好的代码本身就不是一件容易的事情，需要我们不断的更新自己的知识体系，持续优化、重构之前的代码。

我们始终崇尚：每一个项目、每一行代码都应该是按照最高的标准去设计以及开发！

我们也一直努力能写出大师级的代码，当我们回首自己代码的时候，不因文档注释混乱而懊恼，也不因糟糕的实践而悔恨。

**四、参考文献**
----------

*   https://www.jianshu.com/p/33056c9018ef
    
*   https://studygolang.com/articles/32928?fr=sidebar
    
*   https://liwenzhou.com/
    
*   https://draveness.me/golang-101/
    
*   https://eli.thegreenplace.net/2018/on-the-uses-and-misuses-of-panics-in-go/
    
*   http://yunxin.163.com/blog/fenxiang/
    
*    http://yunxin.163.com/blog/fenxiang/
    
*   https://www.jianshu.com/p/b9d57514daa0
    
*   https://www.jianshu.com/p/953a2b37e587
    
*   https://github.com/open-falcon/falcon-plus/tree/master/docs
    

感谢刘东阳、陈洋、董瑶、王海汀、孙琳同学对本文做出的贡献~