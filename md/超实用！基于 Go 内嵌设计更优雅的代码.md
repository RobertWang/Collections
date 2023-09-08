> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IW20Nh0O8DfdtRFa6uQ89w)

> Go 语言通过组合关系、内嵌和接口实现代码复用，详细对比三种方式的区别与应用场景。

**/ Go 语言类型内嵌和结构体内嵌详解 /**

**一、概述**

Go 语言支持类型内嵌和结构体内嵌, 可以在类型系统中建立类型的继承关系。本文将详细介绍 Go 语言内嵌的语法、实现原理和实际应用, 以及类型内嵌和结构体内嵌的区别。

主要内容包括:

> *   类型内嵌语法
>     
> *   结构体内嵌语法
>     
> *   内嵌类型与组合类型
>     
> *   接口内嵌与实现
>     
> *   内嵌类型的方法集
>     
> *   内嵌共享字段名
>     
> *   内嵌构成类型系统
>     
> *   内嵌与继承的差异
>     
> *   实际应用案例
>     

希望通过本文可以全面理解 Go 语言内嵌机制的设计思想, 并能运用自如。

**二、类型内嵌语法**

类型内嵌可以将一个命名类型嵌入到另一个类型中, 形式为:

```
type A struct {
  // ... 
}
type B A

```

这将类型 A 内嵌到类型 B 中。这样 B 就包含了 A 的所有方法和字段。

类型内嵌类似其他语言中的继承, 但又有些不同。

**三、结构体内嵌语法**

与类型内嵌类似, 结构体内嵌也可以嵌入一个已命名的类型:

```
type Person struct {
  Id int
  Name string
}
type Employee struct {
  Person // 内嵌类型Person
  Job string
}

```

这使得 Employee 包含了 Person 的字段 Id 和 Name。可以通过:

```
e := Employee{Person: Person{...}, Job: "dev"}

```

来初始化内嵌的 Person 类型。

**四、内嵌类型与组合类型**

结构体内嵌和结构体组合非常相似, 但也存在一些差异:

> *   内嵌可以直接访问内嵌类型的字段和方法, 组合需要通过实例访问
>     
> *   内嵌共享类型名称, 组合不共享名称
>     
> *   组合 Relationship 更强, 内嵌为 is-a 关系
>     

实际上内嵌是组合关系的语法糖, 编译器会自动转换为组合关系。

**五、接口内嵌与实现**

接口内嵌也常被用来扩展接口:

```
type Reader interface {
  Read(p []byte) (n int, err error)
}
type Writer interface {
  Writer(p []byte) (n int, err error)
}
// 通过内嵌扩展接口
type ReadWriter interface {
  Reader
  Writer
}

```

这使得实现 ReadWriter 接口的类型必须同时实现 Reader 和 Writer 接口。

**六、内嵌类型的方法集**

当一个类型内嵌另一个类型时, 会获得内嵌类型的方法集:

```
type A struct {} 
func (A) method1() {}
type B struct {
  A
}
var b B
b.method1() // B获得A的方法

```

组合关系需要通过实例调用内嵌类型的方法, 内嵌关系可以直接调用。

**七、内嵌共享字段名**

当内嵌的类型包含与外层类型字段名重复的字段时, 默认采取就近访问原则:

```
type A struct {
  field int
}
type B struct {
  A
  field int
}
var b B 
b.field // 就近访问B的field

```

如果需要访问内嵌字段, 需要带上类型名:

```
b.A.field // 访问内嵌的A.field

```

**八、内嵌构成类型系统**

通过内嵌构成合理的类型层级, 不同的类型可以复用公共字段和方法, 并扩展自定义的功能, 这在建模领域对象非常有用。

同时接口内嵌提供了扩展接口的能力。这无需复杂的继承即可构建类型系统。

```
package main
import "fmt"
// 基类Shape
type Shape struct {
  x float64
  y float64
}
func (s *Shape) draw() {
  fmt.Printf("Drawing shape at (%f, %f)\n", s.x, s.y)
}
// Rectangle 内嵌Shape
type Rectangle struct {
  Shape
  height float64
  width  float64
}
// Circle 内嵌Shape
type Circle struct {
  Shape 
  radius float64
}
func main() {
  r := &Rectangle{Shape{0, 0}, 2, 3}
  c := &Circle{Shape{0, 0}, 5}
  r.draw() // 调用内嵌的Shape.draw()
  c.draw()
}

```

**九、内嵌与继承的差异**

尽管内嵌类似继承, 但实际上它们存在一些关键差异:

> *   Go 没有类型继承, 只有结构体内嵌
>     
> *   内嵌并不继承私有字段和方法
>     
> *   优先就近原则, 而非线性查找规则
>     

内嵌应被视为语法糖, 更重要的是组合关系而非继承关系。

```
package main
import "fmt"
type Base struct {
  field1 int
}
func (b *Base) show() {
  fmt.Println(b.field1)
}
type Child struct {
  Base
  field2 int
  field1 int // 与Base字段冲突
}
func main() {
  c := &Child{
    Base: Base{1},
    field1: 2, // 就近覆盖
  }
  c.show() // 内嵌字段隐藏冲突
  // 显示访问内嵌字段
  fmt.Println(c.Base.field1)
}

```

**十、实际应用案例**

我们可以借助内嵌实现一些设计模式:

> *   多重继承: 通过内嵌同时获取多个类型的方法集
>     
> *   装饰器: 通过内嵌扩展功能
>     
> *   适配器: 内嵌类型实现接口, 起到适配作用
>     
> *   桥接: 抽象与实现分离, 通过内嵌搭建桥梁
>     
> *   策略: 定义策略接口, 内嵌实现策略
>     

```
package main
import "fmt"
//    装饰器模式
//    绘制接口
type Drawer interface {
  Draw()
}
// Circle 类型
type Circle struct {
  radius float64  
}
func (c *Circle) Draw() {
  fmt.Printf("Draw circle, radius: %f\n", c.radius)
}
// 颜色装饰器
type ColoredCircle struct {
  Circle
  color string
}
func (c *ColoredCircle) Draw() {
  fmt.Printf("Draw circle with color: %s\n", c.color)
  c.Circle.Draw() // 委托给内嵌类型
}
func main() {
  circle := &Circle{10}
  ccircle := &ColoredCircle{Circle{5}, "Red"}
  circle.Draw()
  ccircle.Draw()
}

```

**十一、总结**

Go 语言通过类型内嵌和结构体内嵌提供了类似继承复用的能力, 可以建立类型间的继承关系。但是与传统面向对象继承有些不同。

正确理解和应用内嵌, 可以编写出更清晰的代码, 复用公共逻辑。同时利用接口也可以实现灵活的扩展。这些是 Go 语言中实现代码复用的主要方式。