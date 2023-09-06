> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Fp0KxoXc4y3z8cgc7IzD-Q)

> 本期通过 Go 语言和大家分享了设计模式中的工厂模式：工厂模式的优势是，通过在组件类和使用方之间添加一个工厂类中间层，实现了代码的防腐和解耦，同时为一部分组件类的构造流程提供出公共切面.

0 前言
====

本期会基于 Go 语言，和大家一起聊聊设计模式中的工厂模式. 本期分享内容包含如下几个部分：

*   • 工厂模式的使用背景
    
*   • 简单工厂模式
    
*   • 工厂方法模式
    
*   • 抽象工厂模式
    
*   • 容器工厂模式
    

1 使用背景
======

Go 语言中没有针对类的构造器方法定义统一的规范，倘若每次需要创建类的实例时，都需要在业务方法中事无俱细地执行实例初始化的细节，那么会存在缺陷的包括：

*   • 业务方法和组件类之间产生过高的耦合度，需要了解到组件类的过多细节
    
*   • 倘若组件类的定义发生变更，那么散落在各处业务方法中对类的构造流程都需要配合改动
    

那么如何解决上述问题呢？在编程世界中，相当的一部分问题都可以通过增加一个中间层加以解决. 我们在此处遵循工厂模式的设计思路，在业务方法和类之间添加一个防腐中间层——工厂类，这样做能够带来的好处是：

*   • 实现类和业务方法之间的解耦，如果类的构造过程发生变更，可以统一收口在工厂类中进行处理，从而对业务方法屏蔽相关细节
    
*   • 倘若有多个类都聚拢在工厂类中进行构造，这样各个类的构造流程中就天然形成了一个公共的切面，可以进行一些公共逻辑的执行
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ic3aBqT2ibZvqxoRqdFZ7Ecg3BpSibXcSum5EJkAUMtU5pLsnh8HJasCJ9OxaMGWa5PJET0eUya41IWaFVtouM7Q/640?wx_fmt=png)

工厂模式属于设计模式中的一种，在实现上，又可以进一步细分为：简单工厂模式、工厂方法模式和抽象工厂模式三种类型，这三种类型我们会在本文的 2-4 章中进行逐一介绍. 除开上面提到的三种经典类型外，本文还会在第 5 章中额外和大家介绍一种比较另类的容器工厂模式.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ic3aBqT2ibZvqxoRqdFZ7Ecg3BpSibXcSuwSSpoJfchLHMvQAcNm95fbMYuzMYZsJFCGLRUUXqtvwzp9S39vjjbA/640?wx_fmt=png)

2 简单工厂模式
========

首先和大家聊聊简单工厂模式. 设计模式中的内容都相对抽象，我这里先不作具体的概念定义，而是通过具体的实例和大家一起进行设计过程的推演，最后再对简单工厂模式的内容进行小结.

假设现在有这样一个编程场景：

*   • 水果 Fruit 是一个抽象的 interface，水果的共性是都可以食用水果，这里我们不纠结主被动的语义，赋以 Fruit 一个 Eat 方法
    
*   • 有三个具体的水果实现类，橘子 Orange、草莓 Strawberry、樱桃 cherry，分别实现了 Fruit 对应的 Eat 方法
    
*   • 有一个具体的水果工厂类 FruitFactory，专门用于水果的生产工作，对应的生产方法为 CreateFruit 方法，可以按照用户指定的水果类型，生产出对应的水果
    

上面聊到的几个类之间形成的 UML 类图如下所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ic3aBqT2ibZvqxoRqdFZ7Ecg3BpSibXcSuuKhEGabP5MEVG0z2R0jt6rISocc4CUBBUyb0XHK7icYc1RgIgcYImFw/640?wx_fmt=png)

下面进行代码展示. 水果 Fruit interface 实现如下：

```
type Fruit interface {
    Eat()
}

```

橘子 Orange、草莓 Strawberry、樱桃 Cherry 三种水果的具体实现类如下：

```
type Orange struct {
    name string
}


func NewOrange(name string) Fruit {
    return &Orange{
        name: name,
    }
}


func (o *Orange) Eat() {
    fmt.Printf("i am orange: %s, i am about to be eaten...", o.name)
}


type Strawberry struct {
    name string
}


func NewStrawberry(name string) Fruit {
    return &Strawberry{
        name: name,
    }
}


func (s *Strawberry) Eat() {
    fmt.Printf("i am strawberry: %s, i am about to be eaten...", s.name)
}


type Cherry struct {
    name string
}


func NewCherry(name string) Fruit {
    return &Cherry{
        name: name,
    }
}


func (c *Cherry) Eat() {
    fmt.Printf("i am cherry: %s, i am about to be eaten...", c.name)
}

```

下面是关于生产水果的工厂类 FruitFactory 的定义，其中 CreateFruit 方法是用于生产水果的核心方法：

*   • 利用工厂生产三类时存在的公共切面，进行随机数的取值，用来给生产出来的水果命名
    
*   • 根据使用方传入的水果类型 typ，调用对应水果类型的构造器方法，并将生产出来的水果进行返回
    
*   • 如果使用方法传入的水果类型 typ 非法，则对外抛出错误
    

```
type FruitFactory struct {
}


func NewFruitFactory() *FruitFactory {
    return &FruitFactory{}
}


func (f *FruitFactory) CreateFruit(typ string) (Fruit, error) {
    src := rand.NewSource(time.Now().UnixNano())
    rander := rand.New(src)
    name := strconv.Itoa(rander.Int())


    switch typ {
    case "orange":
        return NewOrange(name), nil
    case "strawberry":
        return NewStrawberry(name), nil
    case "cherry":
        return NewCherry(name), nil
    default:
        return nil, fmt.Errorf("fruit typ: %s is not supported yet", typ)
    }
}

```

上述实现代码实现起来的直观明了，这正是简单工厂模式的优势所在，然而我们同样需要注意到其中存在的局限性——不利于实现类的扩展：

*   • 每当有新的水果实现类需要支持时，需要在 FruitFactory 生产水果的 CreateFruit 方法中进行修改，在 switch case 中增加新的分支，这样做是不符合代码设计规范中的开闭原则的（开闭原则：面向扩展开放，面向修改关闭）
    
*   • 此外，当需要支持的水果类型 typ 数量提升时，这个 CreateFruit 方法会存在方法圈复杂度过高的问题
    

针对于第一个问题，是简单工厂模式的固有硬伤，需要切换到本文第 3 章中介绍到的工厂方法模式才能予以解决.

针对于存在的第二个问题，可以采用表驱动替代 switch case 分支映射的方式进行优化：

*   • 将水果构造器函数定义为一个类型 fruitCreator
    
*   • 在水果构造工厂 FruitFactory 中，内置一个 map creators，根据水果类型映射到具体的构造器方法 fruitCreator
    
*   • 在水果构造工厂的构造器方法中，完成 creators map 的初始化
    
*   • 在 FruitFactory.CreateFruit 方法中，根据水果类型映射到对应的构造器方法 fruitCreator，然后进行水果的构造
    

```
type fruitCreator func(name string) Fruit


type FruitFactory struct {
    creators map[string]fruitCreator
}


func NewFruitFactory() *FruitFactory {
    return &FruitFactory{
        creators: map[string]fruitCreator{
            "orange":     NewOrange,
            "strawberry": NewStrawberry,
            "cherry":     NewCherry,
        },
    }
}


func (f *FruitFactory) CreateFruit(typ string) (Fruit, error) {
    fruitCreator, ok := f.creators[typ]
    if !ok {
        return nil, fmt.Errorf("fruit typ: %s is not supported yet", typ)
    }


    src := rand.NewSource(time.Now().UnixNano())
    rander := rand.New(src)
    name := strconv.Itoa(rander.Int())
    return fruitCreator(name), nil
}

```

下面通过一段单测代码，给出简单工厂模式的使用示例：

```
func Test_factory(t *testing.T) {
    // 构造工厂
    fruitFactory := NewFruitFactory()


    // 尝个橘子
    orange, _ := fruitFactory.CreateFruit("orange")
    orange.Eat()


    // 来颗樱桃
    cherry, _ := fruitFactory.CreateFruit("cherry")
    cherry.Eat()


    // 来个西瓜，因为未实现会报错
    watermelon, err := fruitFactory.CreateFruit("watermelon")
    if err != nil {
        t.Error(err)
        return
    }
    watermelon.Eat()
}

```

下面对简单工厂模式做个小结：

*   • 对于拟构造的组件，需要依据其共性，抽离出一个公共 interface
    
*   • 每个具体的组件类型对 interface 加以实现
    
*   • 定义一个具体的工厂类，在构造器方法接受具体的组件类型，完成对应类型组件的构造
    

简单工厂模式的优势包括：

*   • 属于工厂模式最为简单直观的一种类型
    
*   • 构造各类组件时的聚拢收口效果最好，提供的公共切面最全面到位
    

存在的劣势为：

*   • 组件类扩展时，需要直接修改工厂的组件构造方法，不符合开闭原则
    

3 工厂方法模式
========

为了解决简单工厂模式中存在的问题，我们对设计流程进行修改：

*   • 关于组件的定义模式不变. 一个抽象的 Fruit interface，多个具体的水果实现 Orange、Strawberry、Cherry
    
*   • 将工厂类 FruitFactory 由具体的实现类改为抽象的 interface
    
*   • 针对每类水果，提供出一个具体的工厂实现类，如 OrangeFactory、StrawberryFactory、CherryFactory
    

对应的 UML 类图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ic3aBqT2ibZvqxoRqdFZ7Ecg3BpSibXcSuZHib4KI5kGMyydkf7xEIeJniaIpNxmFqcUwyRc9KrQIR6ia8QmystP5fQ/640?wx_fmt=png)

在工厂方法模式中，对抽象组件 Fruit interface 以及几个具体实现类 Orange、Strawberry 和 Cherry 的定义和简单工厂模式如出一辙，这里不再重复展示代码.

与简单工厂模式有所区别的是， 水果工厂类 FruitFactory 在此处变成一个抽象的 interface，且针对每种具体的水果实现类需要对应地声明一种工厂实现类，包括 OrangeFactory、StrawberryFactory、CherryFactory，具体的实现代码如下：

```
type FruitFactory interface {
    CreateFruit() Fruit
}


type OrangeFactory struct {
}


func NewOrangeFactory() FruitFactory {
    return &OrangeFactory{}
}


func (o *OrangeFactory) CreateFruit() Fruit {
    return NewOrange("")
}


type StrawberryFactory struct {
}


func NewStrawberryFactory() FruitFactory {
    return &StrawberryFactory{}
}


func (s *StrawberryFactory) CreateFruit() Fruit {
    return NewStrawberry("")
}


type CherryFactory struct {
}


func NewCherryFactory() FruitFactory {
    return &CherryFactory{}
}


func (c *CherryFactory) CreateFruit() Fruit {
    return NewCherry("")
}

```

这样的设计模式下，即便后续有频繁扩展水果实现类的需求，也无须对老模块的代码进行修改，而是需要扩展实现一个水果 Fruit 的实现类以及对应的水果工厂实现类即可，比如，倘若此处我们需要在水果列表中扩展一个 “西瓜” 的话，那么需要新增的代码如下：

```
type Watermelon struct {
    name string
}


func NewWatermelon(name string) Fruit {
    return &Watermelon{
        name: name,
    }
}


func (w *Watermelon) Eat() {
    fmt.Printf("i am watermelon: %s, i am about to be eaten...", w.na


type WatermelonFactory struct {
}


func NewWatermelon() FruitFactory {
    return &WatermelonFactory{}
}


func (w *WatermelonFactory) CreateFruit() Fruit {
    return NewWatermelon("")
}

```

下面是使用到工厂方法模式的单测代码示例：

```
func Test_factory(t *testing.T) {
    // 尝个橘子
    orangeFactory := NewOrangeFactory()
    orange := orangeFactory.CreateFruit()
    orange.Eat()


    // 来颗樱桃
    cherryFactory := NewCherryFactory()
    cherry := cherryFactory.CreateFruit()
    cherry.Eat()
}

```

工厂方法模式相较于简单工厂模式而言，解决了扩展水果类不满足开闭原则的问题，然而工厂方法模式也有其固有的缺陷：

*   • 需要为每个水果单独实现一个工厂类, 代码冗余度较高
    
*   • 原本构造多个水果类时存在的公共切面不复存在，一些通用的逻辑需要在每个水果工厂实现类中重复声明一遍
    

4 抽象工厂模式
========

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ic3aBqT2ibZvqxoRqdFZ7Ecg3BpSibXcSurbsuL0NAwWJh7helBY61tibcz8251JmWncKibVialJVIaZ7Oa8w5YEqxA/640?wx_fmt=png)

接下来，我们进行抽象工厂模式的介绍. 这里，针对工厂需要构造的组件，我们通过两个维度进行拆解：

*   • 我们假设水果 Fruit 中仅包含两种具体的水果：草莓 strawberry 和柠檬 lemon
    
*   • 我们把每种具体的水果实现类称为一个产品等级，strawberry 是一个产品等级，lemon 也是一个产品等级
    
*   • 在同一个水果实现类中，我们额外新增一个品牌的维度，成为产品族. 例如 strawberry 和 lemon 可以由不同品牌的厂商进行生产，比如水果品牌佳农 GoodFarmer 生产的草莓为 GoodFarmerStrawberry，生产的柠檬为 GoodFarmerLemon；水果品牌 Dole 都乐生产的草莓为 DoleStrawberry，生产的柠檬为 DoleLemon
    

基于上述的两个维度，我们尝试对简单工厂模式和厂方法模式中优势进行聚拢：

*   • 首先，我们把种类相对稳定，不需要频繁扩展变更的维度定义为产品等级. 比如上述例子中的 Fruit，我们需要固定明确后续 Fruit 只包含草莓 strawberry 和柠檬 lemon 两类，没有频繁扩展的诉求
    
*   • 针对于种类需要频繁变更的维度，我们将其定义为产品族. 比如上述例子中的品牌，我们目前支持了佳农 GoodFarmer 和都乐 Dole，后续还可以继续扩展支持更丰富的水果品牌，如 佳沛 Zespri、佳沃 JOYVIO 等
    
*   • 每次需要扩展产品族时，都需要实现对应产品族的工厂 factory 实现类，而无需对老的实现方法直接进行修改，符合开闭原则
    
*   • 针对于不频繁变动的产品等级，如草莓 strawberry 与柠檬 lemon，每个品牌都会有一个具体的 factory 工厂实现类. 其中会统一声明对应于每种水果的构造方法，此时具备实现公共切面的能力
    

按照上述思路，抽象工厂模式我们定义的 UML 类图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ic3aBqT2ibZvqxoRqdFZ7Ecg3BpSibXcSup8wCFarSqqrK4rHicmTqCaueLxhzbkGwRmYdF4ctvXLv5bHDwBzOfRw/640?wx_fmt=png)

在抽象工厂模式下，我们需要将每种水果类型定义为一个抽象的 interface，包括草莓 strawberry 和柠檬 lemon. 其中 strawberry 包含一个方法 SweetAttack：当草莓被食用时，它会发起一轮甜蜜攻势；柠檬 Lemon 包含的方法为 AcidAttack，食用它时需要承受一轮酸劲攻势.

```
type Strawberry interface {
    SweetAttack()
}


type Lemon interface {
    AcidAttack()
}

```

下面定义一个抽象的水果工厂 FruitFactory，其中分别声明了用于生产草莓的 CreateStrawberry 方法以及创建柠檬的 CreateLemon 方法：

```
type FruitFactory interface {
    CreateStrawberry() Strawberry
    CreateLemon() Lemon
}

```

下面，针对每种水果类型，进行不同品牌下的具体实现. 比如草莓 strawberry 可以实现为 佳农生产的 GoodFarmerStrawberry 和都乐生产的 DoleStrawberry：

```
type GoodfarmerStrawberry struct {
    brand string
    Strawberry
}




func (g *GoodfarmerStrawberry) SweetAttack() {
    fmt.Printf("sweet attack from %s, ", g.brand)
}




type GoodfarmerLemon struct {
    brand string
    Lemon
}




func (g *GoodfarmerLemon) AcidAttack() {
    fmt.Printf("acid attack from %s, ", g.brand)
}




type DoleStrawberry struct {
    brand string
    Strawberry
}




func (d *DoleStrawberry) SweetAttack() {
    fmt.Printf("sweet attack from %s, ", d.brand)
}




type DoleLemon struct {
    brand string
    Lemon
}




func (d *DoleLemon) AcidAttack() {
    fmt.Printf("acid attack from %s,", d.brand)
}

```

针对每个品牌，声明一个水果工厂实现类： GoodFarmerFactory 和 DoleFactory，展示如下：

```
type GoodfarmerFactory struct{}


func (g *GoodfarmerFactory)myAspect(){
   fmt.Println("good farmer aspect...")
}


func (g *GoodfarmerFactory) CreateStrawberry() Strawberry {
    // 同一个产品族可以插入一个切面
    g.myAspect()
    defer g.myAspect()
    return &GoodfarmerStrawberry{
        brand: "goodfarmer",
    }
}
func (g *GoodfarmerFactory) CreateLemon() Lemon {
    // 同一个产品族可以插入一个切面
    g.myAspect()
    defer g.myAspect()
    return &GoodfarmerLemon{
        brand: "goodfarmer",
    }
}


type DoleFactory struct{}


func (d *DoleFactory)myAspect(){
   fmt.Println("dole aspect...")
}


func (d *DoleFactory) CreateStrawberry() Strawberry {
    // 同一个产品族可以插入一个切面
    d.myAspect()
    defer d.Myspect()
    return &DoleStrawberry{
        brand: "dole",
    }
}
func (d *DoleFactory) CreateLemon() Lemon {
    // 同一个产品族可以插入一个切面
    d.myAspect()
    defer d.Myspect()
    return &DoleLemon{
        brand: "dole",
    }
}

```

此时，倘若我们需要额外扩展一个新的水果品牌，比如佳沛 Zespri，此时需要额外新增如下代码：

```
type ZespriStrawberry struct {
    brand string
    Strawberry
}


func (z *ZespriStrawberry) SweetAttack() {
    fmt.Printf("sweet attack from %s, ", z.brand)
}


type ZespriLemon struct {
    brand string
    Lemon
}


func (z *ZespriLemon) AcidAttack() {
    fmt.Printf("acid attack from %s, ", z.brand)
}


type ZespriFactory struct{}




func (z *ZespriFactory)myAspect(){
   fmt.Println("dole aspect...")
}




func (z *ZespriFactory) CreateStrawberry() Strawberry {
    // 同一个产品族可以插入一个切面
    z.myAspect()
    defer z.Myspect()
    return &ZespriStrawberry{
        brand: "zespri",
    }
}
func (z *ZespriFactory) CreateLemon() Lemon {
    // 同一个产品族可以插入一个切面
    z.myAspect()
    defer z.Myspect()
    return &ZespriLemon{
        brand: "zespri",
    }
}

```

需要注意，抽象工厂模式，倘若需要扩展产品等级，对应的代价是很高昂的. 大家在此处不妨可以尝试一下扩展一类水果，看看涉及到的代码改动包括哪些内容.

下面给出一个针对于抽象工厂模式的测试代码示例：

```
func Test_factory(t *testing.T) {
    // 尝尝佳农品牌的水果
    goodfarmerFactory := GoodfarmerFactory{}
    goodfarmerStrawberry := goodfarmerFactory.CreateStrawberry()
    goodfarmerStrawberry.SweetAttack()


    goodfarmerLemon := goodfarmerFactory.CreateLemon()
    goodfarmerLemon.AcidAttack()


    // 尝尝都乐品牌的水果
    doleFactory := DoleFactory{}
    doleStrawberry := doleFactory.CreateStrawberry()
    doleStrawberry.SweetAttack()


    doleLemon := doleFactory.CreateLemon()
    doleLemon.AcidAttack()
}

```

抽象工厂模式通过将组件拆分为产品族和产品等级的维度，将需要频繁扩展的维度和相对稳定的维度进行拆分，尝试兼具简单工厂模式和工厂方法模式的优势，在使用过程中我们需要注意，在模块设计之初，就需要明确产品族和产品等级的维度定义，倘若这部分定义出现偏差，这种设计模式就会产生事与愿违的负面效果.

5 容器工厂模式
========

最后再给大家介绍另类的工厂模式——容器工厂. 这种模式的思路是，将工厂的改造为一个组件交易市场，每个组件的构造工作不再统一由工厂完成，取而代之的是，工厂会对外暴露一个统一的入口，所有组件的提供方通过这个入口完成组件的注入. 后续组件的使用方通过工厂这个中介提供的统一出口, 进行对应组件的获取.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/3ic3aBqT2ibZvqxoRqdFZ7Ecg3BpSibXcSua9vyw6dOXc5FAzgWlx7qqw1J8gFGefKGcWAGdZ8WUc5Q8lWEXw8Q1w/640?wx_fmt=png)

实现这种容器工厂模式，需要依赖到第三方依赖注入框架的能力. 这边笔者使用到的是 golang 开源 ioc 框架 dig：https://github.com/uber-go/dig.

在实现时，我们声明一个全局工厂类 factory ，同时 factory 中内嵌一个 dig container 的容器实例：

```
type Factory struct {
    container *dig.Container
}

```

factory 需要对外暴露两个方法：Inject 和 Invoke 方法，分别作为注入组件的入口方法和获取组件的出口方法：

```
func (f *Factory) Inject(constructor interface{}) error {
    return f.container.Provide(constructor)
}


func (f *Factory) Invoke(invoker interface{}) error {
    return f.container.Invoke(invoker)
}

```

接下来我们实现好一个工厂类的单例对象，方便让各处的组件提供方和组件使用方能够快速地获取到相同容器工厂实例. （此处涉及到单例模式的设计思路并用到了 golang 标准库提供的单例工具 sync.Once，大家对更细节的内容感兴趣的话，可以阅读我之前发表的文章——Golang 设计模式之单例模式）

```
package factory


var (
    once    sync.Once
    factory *Factory
)


func GetFactory() *Factory {
    once.Do(func() {
        factory = newFactory(dig.New())
    })
    return factory
}

```

各处的组件提供方，可以通过 GetFactory 方法快速获取到工厂单例 factory，并调用 Factory.Inject 方法，完成将组件注入到容器工厂的操作. （dig 采用组件懒加载的方式，此处注入组件实际上注入的是组件的构造器方法，组件真正的构造时机处于其第一次被真正使用到时）

```
func init() {
    f := factory.GetFactory()
    f.Inject(NewComponentX)
}


type ComponentX struct{}


func NewComponentX() *ComponentX{
    return &ComponentX{}
}

```

当需要通过工厂获取组件时，用户可以在任意位置调用 GetFactory 方法获取到工厂单例 factory，然后通过 Invoke 方法闭包注入组件的提取函数，容器工厂会对闭包函数的入参进行反射，映射到对应的组件实例，然后将其闭包传值返回

```
func GetComponentX()(*ComponentX,error){
    f := factory.GetFactory()  
    var componentX *ComponentX
    return componentX, f.Invoke(func(_x *ComponentX){
        componentX = _x
    })
}

```

倘若大家对于 golang 依赖注入框架 dig 的更多细节内容感兴趣的话，可以阅读我之前发表的文章——低配 Spring—Golang IOC 框架 dig 原理解析.

6 总结
====

本期通过 Go 语言和大家分享了设计模式中的工厂模式：

工厂模式的优势是，通过在组件类和使用方之间添加一个工厂类中间层，实现了代码的防腐和解耦，同时为一部分组件类的构造流程提供出公共切面.

工厂模式可以进一步细分为:

*   • 简单工厂模式: 工厂模式中最简单直观的实现方式, 有很好的切面效果, 但是在组件类扩展时无法满足开闭原则
    
*   • 工厂方法模式: 一个组件类对应一个工厂类, 存在一定的代码冗余以及对公共切面的削弱，但是能够在组件类扩展时满足开闭原则
    
*   • 抽象工厂模式: 通过两个维度对组件类进行拆解. 需要保证易于扩展、灵活可变的维度需要定义为产品族；相对稳定、不易于扩展维度需要定义为产品等级. 这样能同时保证产品族维度的扩展灵活性以及产品等级维度的切面能力.
    

此外，本文还额外介绍了一种另类的容器工厂模式，底层需要基于依赖注入框架实现，让组件提供能够在各处方便地完成组件类的注入操作，而组件的使用方，则通过容器工厂的统一出口进行组件的获取.