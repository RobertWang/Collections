> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/niwBEfq3A87PNxph1WJmmA)

> wire 是 Google 开源的一个依赖注入工具，它使用代码生成的方式实现。它可以自动帮助我们创建指定类型的对象，并组装它的依赖。

**什么是依赖注入**？有时候一个结构体非常复杂，包含了非常多各种类型的属性，这些属性又包含了更多的属性，当我们创建这样一个结构体时需要编写大量的代码。面向接口编程可以让我们的代码避免耦合更具扩展性，但统一更换接口实现时需要大范围的修改代码。

依赖注入帮助我们解决类似的问题，依赖注入框架能够自动解析依赖关系，帮助我们自动构建结构体实例。依赖注入可以对接口注入实例，让整个代码系统不用关注具体的接口实现。

由于 Go 语言静态的特性，依赖注入在 Go 中应用并不广泛，主要有两种实现方式：代码生成和反射。

`wire`[1] 是 Google 开源的一个依赖注入工具，它使用代码生成的方式实现。我们只需要在一个特殊的`go`文件中告诉`wire`类型之间的依赖关系，它会自动帮我们生成代码，帮助我们创建指定类型的对象，并组装它的依赖。

安装
--

如上所述，`wire`利用代码生成来实现依赖注入，所以我们需要安装`wire`命令到`PATH`中

```
$ go install github.com/google/wire/cmd/wire@latest


```

执行`wire -h` 有如下显示说明安装成功

```
$ wire -h
Usage: wire <flags> <subcommand> <subcommand args>

Subcommands:
 check            print any Wire errors found
 commands         list all command names
 diff             output a diff between existing wire_gen.go files and what gen would generate
 flags            describe all known top-level flags
 gen              generate the wire_gen.go file for each package
 help             describe subcommands and their syntax
 show             describe all top-level provider sets


Use "wire flags" for a list of top-level flags


```

快速入门
----

设计一个程序，其中 `Event`依赖`Greeter`，`Greeter`依赖`Message`

```
type Message string

func NewMessage() Message {
 return Message("Hi there!")
}

type Greeter struct {
 Message Message
}

func NewGreeter(m Message) Greeter {
 return Greeter{Message: m}
}

func (g Greeter) Greet() Message {
 return g.Message
}

type Event struct {
 Greeter Greeter // <- adding a Greeter field
}

func NewEvent(g Greeter) Event {
 return Event{Greeter: g}
}

func (e Event) Start() {
 msg := e.Greeter.Greet()
 fmt.Println(msg)
}


```

![](https://mmbiz.qpic.cn/mmbiz_png/kWF4acROZUDnLq2geic6waiaSibBlUmkdtTASkvlJK3ADH18sp5naEUzB8H3bp4IBqV5cApkTKXoXaafoQBYCND0A/640?wx_fmt=png)

如果运行`Event`需要逐个构建依赖，代码如下

```
func main() {
    message := NewMessage()
    greeter := NewGreeter(message)
    event := NewEvent(greeter)

    event.Start()
}


```

### 两个概念

正式开始前需要先了解一下 `wire` 当中的两个概念：`provider` 和 `injector`

#### Provider

`Provider` 你可以把它理解成工厂函数，这个函数的入参是依赖的属性，返回值为新一个新的类型实例

如下所示都是 `provider` 函数，在实际使用的时候，往往是一些简单的工厂函数，这个函数不会太复杂。

```
func NewMessage() Message {
 return Message("Hi there!")
}

func NewGreeter(m Message) Greeter {
 return Greeter{Message: m}
}


```

不过需要注意的是**在 wire 中不能存在两个 provider 返回相同的组件类型**。即如下两个函数不能同时存在

```
func NewMessage1() Message {
 return Message("Hi there!")
}

func NewMessage2() Message {
 return Message("Hi there!")
}


```

#### Injector

我们常常在 `wire.go` 文件中定义 `injector` ，`injector`也是一个普通函数，它用来声明组件之间的依赖关系

如下代码，我们把`Event`、`Greeter`、`Message` 的工厂函数 (`provider`) 一股脑塞入`wire.Build()`中，代表着构建 `Event`依赖`Greeter`、`Message`。我们不必关心`Greeter`、`Message`之间的依赖关系，`wire`会帮我们处理

```
// wire.go
func InitializeEvent() Event {
    wire.Build(NewEvent, NewGreeter, NewMessage)
    return Event{}
}


```

### 使用 wire 生成代码

当准备好`provider` 和 `injector`之后，通过 `wire` 命令可以自动生成一个完整的函数。如果 `wire.go` 不再当前路径下，也可以指定包名

```
# 等价 wire ./internal 
$ wire gen ./internal
wire: github.com/liangwt/note/golang/demo/wire/internal: wrote /golang/demo/wire/internal/wire_gen.go


```

在执行完`wire`命令之后就会生成`wire_gen.go`，它的内容就是构建 `Event`。我们需要连同`wire_gen.go`一起提交到版本控制系统里

```
// Code generated by Wire. DO NOT EDIT.

//go:generate go run github.com/google/wire/cmd/wire
//go:build !wireinject
// +build !wireinject

package internal

// Injectors from wire.go:

func InitializeEvent() Event {
 message := NewMessage()
 greeter := NewGreeter(message)
 event := NewEvent(greeter)
 return event
}


```

此时我们就可以直接在 main 里这样用，省去逐个构建依赖的麻烦

```
func main() {
 event := internal.InitializeEvent()
 event.Start()
}


```

**小技巧**

可以在`wire.go`第一行加入 `//+build wireinject` （与`//go:build wireinject`等效）注释，确保了这个文件在我们正常编译的时候不会被引用

而 `wire .` 生成的文件 `wire_gen.go` 会包含 `//+build !wireinject` 注释，正常编译的时候，不指定 tag 的情况下会引用这个文件

```
//go:build wireinject
// +build wireinject

// wire.go
package internal

import "github.com/google/wire"

func InitializeEvent() Event {
 wire.Build(NewEvent, NewGreeter, NewMessage)
 return Event{}
}


```

进阶用法
----

### 返回错误

在 Go 中如果遇到错误，我们会在最后一个返回值返回`error`，`wire`同样也支持返回错误的情况，只需要在 `injector`的函数签名中加上`error`返回值即可

**调整`provider`的签名**

```
type Event struct {
 Greeter Greeter
}

//func NewEvent(g Greeter) Event {
// return Event{Greeter: g}
//}
func NewEvent(g Greeter) (Event, error) {
 if time.Now().Unix()%2 == 0 {
  return Event{}, errors.New("could not create event: event greeter is grumpy")
 }

 return Event{Greeter: g}, nil
}

func (e Event) Start() {
 msg := e.Greeter.Greet()
 fmt.Println(msg)
}


```

**调整`injector`的签名**

```
// wire.go
func InitializeEvent() (Event, error) {
 panic(wire.Build(NewEvent, NewGreeter, NewMessage))
}


```

生成的代码如下所示，可以发现会像我们自己写代码一样判断一下 `if err` 然后返回

```
// Code generated by Wire. DO NOT EDIT.

//go:generate go run github.com/google/wire/cmd/wire
//go:build !wireinject
// +build !wireinject

package main

// Injectors from wire.go:

func InitializeEvent() (Event, error) {
 message := NewMessage()
 greeter := NewGreeter(message)
 event, err := NewEvent(greeter)
 if err != nil {
  return Event{}, err
 }
 return event, nil
}


```

main 中的调用

```
func main() {
    e, err := InitializeEvent()
    if err != nil {
        fmt.Printf("failed to create event: %s\n", err)
        os.Exit(2)
    }
    e.Start()
}


```

### 传入参数

如果要构建的目标组件需要外部的输入时，可以在定义`provider` 和 `injector`时同步加上输入

**调整`provider`的签名**

```
// provider.go
type Message string

func NewMessage(phrase string) Message {
 return Message(phrase)
}


```

**调整`injector`的签名**

```
// wire.go
func InitializeEvent(phrase string) (Event, error) {
 wire.Build(NewEvent, NewGreeter, NewMessage)
}


```

不展示生成的代码了，main 中的调用

```
func main() {
 event, err := InitializeEvent("Hi there!")
 if err != nil {
  fmt.Printf("failed to create event: %s\n", err)
  os.Exit(2)
 }

 event.Start()
}


```

### ProviderSet

有时候可能多个类型有相同的依赖，我们每次都将相同的构造器传给`wire.Build()`不仅繁琐，而且不易维护，一个依赖修改了，所有传入`wire.Build()`的地方都要修改。为此，`wire`提供了一个`ProviderSet`（构造器集合），可以将多个构造器打包成一个集合，后续只需要使用这个集合即可。

如下，后续再调整`NewGreeter`和`NewMessage`，就可以统一改了

```
// wire.go

var wireSet = wire.NewSet(NewGreeter, NewMessage)

func InitializeEvent(phrase string) (Event, error) {
 wire.Build(wireSet, NewEvent)
}


```

### 结构构造器

对于`Greeter`，前文使用`NewGreeter`作为`provider`

```
type Greeter struct {
 Message Message
}

func NewGreeter(m Message) Greeter {
 return Greeter{Message: m}
}


```

如果不显式实现`NewGreeter`，可以直接使用`wire`提供的**结构构造器**（Struct Provider）。结构构造器创建某个类型的结构，然后用参数或调用其它构造器填充它的字段

结构构造器使用`wire.Struct`函数，第一个参数固定为`new(结构名)`，后面可接任意多个参数，表示需要为该结构的哪些字段注入值

我们也可以使用通配符`*`表示注入所有字段

如下的例子代表`Greeter`需要注入`Message`字段，不用再单独实现`NewGreeter`了

```
Greeter
var wireSet = wire.NewSet(NewMessage, wire.Struct(new(Greeter), "Message"))

func InitializeEvent(phrase string) (Event, error) {
 wire.Build(wireSet, NewEvent)
}


```

### 结构字段作为构造器

有时候我们编写一个构造器，只是简单的返回某个结构的一个字段，这时可以使用`wire.FieldsOf`简化操作。

```
type Foo struct {
    S string
    N int
    F float64
}

func getS(foo Foo) string {
    // Bad! Use wire.FieldsOf instead.
    return foo.S
}

func provideFoo() Foo {
    return Foo{ S: "Hello, World!", N: 1, F: 3.14 }
}

func injectedMessage() string {
    wire.Build(
        provideFoo,
        getS)
    return ""
}


```

```
func injectedMessage() string {
    wire.Build(
        provideFoo,
        wire.FieldsOf(new(Foo), "S"))
    return ""
}


```

同样的，第一个参数为`new(结构名)`，后面跟多个参数表示将哪些字段作为构造器，`*`表示全部。

### 绑定值

有时候，我们需要为某个类型绑定一个值，而不想依赖构造器每次都创建一个新的值。有些类型天生就是单例，例如配置，数据库对象（`sql.DB`）。这时我们可以使用`wire.Value`绑定值，使用`wire.InterfaceValue`绑定接口

修改一下 `Greeter` 使他依赖一个`int`和`io.Reader`然后为它直接绑定 `a=10` 、`io.Reader = os.Stdin`

```
// main.go
var singletonMessage = NewMessage("Hello, world!")

type Message string

func NewMessage(phrase string) Message {
 return Message(phrase)
}

type Greeter struct {
 a int
 r io.Reader

 Message Message
}


```

```
var wireSet = wire.NewSet(
 wire.Struct(new(Greeter), "*"),
 wire.Value(10),
 wire.InterfaceValue(new(io.Reader), os.Stdin),
 wire.Value(singletonMessage),
)

func InitializeEvent(phrase string) (Event, error) {
 panic(wire.Build(wireSet, NewEvent))
}


```

### 绑定接口

使用 `wire.Bind` 将 `Struct` 和接口进行绑定，表示这个结构体实现了这个接口，`wire.Bind` 的使用方法就是 `wire.Bind(new(接口), new(实现))`

```
func NewGreeter(m Message, phrase string) Greeter {
 return Greeter{Message: m}
}

func (g Greeter) Greet() Message {
 return g.Message
}

type IGreeter interface{
 Greet() Message
}

type Event struct {
 Greeter IGreeter
}


```

```
var wireSet = wire.NewSet(
 wire.Struct(new(Greeter), "*"),
 wire.Value(10),
 wire.InterfaceValue(new(io.Reader), os.Stdin),
 wire.Value(singletonMessage),
)

func InitializeEvent(phrase string) (Event, error) {
 panic(wire.Build(wireSet, NewEvent, wire.Bind(new(IGreeter), new(*Greeter))))
}


```

总结
--

本文介绍依赖注入框架`wire`的基础用法：

*   实现各`struct`的工厂函数，`wire`称之为`provider`
    
*   在`wire.go`中利用函数签名和函数体中调用`wire.Build`描述一个`struct`的所有依赖，此函数被称为 `injector`
    
*   执行`wire`命令，会生成`wire_gen.go` 其中包含和`injector`签名相同的函数，函数的内容为构建的相关依赖并组合
    
*   使用`wire_gen.go`中的函数创建我们的实例
    

除了基础用法之外，还有很多高级用法，例如绑定接口、绑定值，来实现面向接口编程和单例模式，此部分可以参考官方文档

* * *

关注我，下篇为大家介绍 Go 中基于反射实现的依赖注入代表作`dig`

### 参考资料

[1]

`wire`: https://github.com/google/wire

[2]

Go 工程化 (三) 依赖注入框架 wire: https://lailin.xyz/post/go-training-week4-wire.html

[3]

Wire User Guide: https://github.com/google/wire/blob/main/docs/guide.md

[4]

Go 每日一库之 wire: https://darjun.github.io/2020/03/02/godailylib/wire/