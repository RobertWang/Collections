> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/m2N6YSEqseRfrqLjDWZIZQ)

> 在日常开发中，枚举类型是很常用的，虽然 Go 语言中没有内置枚举类型，但也不妨碍我们自己实现一个类似的 “枚举类型”。

前言
==

枚举类型是一种常用的数据类型，用于表示一组有限的、预定义的、具名的常量值。在枚举类型中，每个常量都是一个枚举值，它们之间的值相等且唯一。

枚举类型通常用于表示一组相关的常量，比如星期、月份、性别等等。在其他语言里（比如 `Java` 和 `C`），都内置了枚举类型，而在 `Go` 语言里是没有内置枚举类型的，因此我们需要采用其他方式实现类似的枚举类型功能，本文将介绍如何实现 “**枚举类型**”。

Go 语言中的 “枚举类型”
==============

枚举类型的值本质上是常量，因此我们可以使用 `Go` 语言中的常量来实现类似枚举类型的功能，例如：

```
const (
   Sunday    = 1
   Tuesday   = 2
   Wednesday = 3
   Thursday  = 4
   Friday    = 5
   Saturday  = 6
   Monday    = 7
)


```

在这个例子中，我们使用了 `const` 关键字定义了一组常量，其中每个常量的名称代表着一个枚举，其值为对应的整数。

虽然这个例子能实现类似的枚举类型，但它不具备枚举类型的所有特征，例如缺少**安全性**和**约束性**，为了解决这两个问题，我们可以使用自定义类型进行改进：

```
type WeekDay int

const (
   Sunday    WeekDay = 1
   Tuesday   WeekDay = 2
   Wednesday WeekDay = 3
   Thursday  WeekDay = 4
   Friday    WeekDay = 5
   Saturday  WeekDay = 6
   Monday    WeekDay = 7
)


```

在改进后的例子中，我们定义了一个自定义类型 `Weekday`，用于表示星期几。使用 `const` 关键字定义一个常量组，其中每个常量都被赋予了一个具体的值，同时使用 `Weekday` 类型进行类型约束和类型检查。这样，我们就可以通过枚举值的名称来表示某个特定的星期几，并且由于使用了自定义类型，编译器可以进行类型检查，从而提高了类型安全性。

使用 iota 优雅实现枚举
==============

通过前面的例子不难发现，当我们需要定义多个枚举值时，手动指定每个枚举常量的值会变得十分麻烦。为了解决这个问题，我们可以使用 `iota` 常量生成器，它可以帮助我们生成连续的整数值。

例如，使用 `iota` 定义一个星期几的枚举类型：

```
type WeekDay int

const (
   Sunday WeekDay = iota
   Tuesday
   Wednesday
   Thursday
   Friday
   Saturday
   Monday
)


```

在这个例子中，我们使用 `iota` 自增常量生成器来定义了一个星期几的枚举类型，每个枚举值都是一个 `Weekday` 类型的常量。由于 `iota` 的自增规则，每个枚举值的值将自动递增，从而生成一系列连续的整数值。

为自定义的枚举添加方法
===========

```
type WeekDay int

const (
   Sunday WeekDay = iota
   Tuesday
   Wednesday
   Thursday
   Friday
   Saturday
   Monday
)


```

为了能让我们实现的 “枚举类型” 更加具备枚举类型的特征，我们可以为其添加类似 `Java` 等其他语言中的枚举方法。

Name()
------

返回枚举值的名称。

```
// Name 返回枚举值的名称
func (w WeekDay) Name() string {
   if w < Sunday || w > Monday {
      return"Unknown"
   }
   return [...]string{"Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"}[w]
}


```

Original
--------

返回枚举值在枚举类型中的位置。

```
// Original 由于这个枚举类型的枚举值本质上是整数常量，因此我们可以直接使用枚举值作为它的序号
func (w WeekDay) Original() int {
   return int(w)
}


```

String()
--------

实现 `String` 方法，用于打印输出。

```
// 将枚举值转成字符串，便于输出
func (w WeekDay) String() string {
   return w.Name()
}


```

Values()
--------

返回一个包含所有枚举值的切片。

```
// Values 返回一个包含所有枚举值的切片
func Values() []WeekDay {
   return []WeekDay{Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday}
}


```

ValueOf()
---------

根据名称返回对应的枚举值。

```
// ValueOf 使用 switch 语句来根据名称返回对应的枚举值
func ValueOf(name string) (WeekDay, error) {
   switch name {
   case"Sunday":
      return Sunday, nil
   case"Monday":
      return Monday, nil
   case"Tuesday":
      return Tuesday, nil
   case"Wednesday":
      return Wednesday, nil
   case"Thursday":
      return Thursday, nil
   case"Friday":
      return Friday, nil
   case"Saturday":
      return Saturday, nil
   default:
      return 0, fmt.Errorf("invalid WeekDay name: %s", name)
   }
}


```

小结
==

在日常开发中，枚举类型是很常用的，虽然 Go 语言中没有内置枚举类型，但也不妨碍我们自己实现一个类似的 “**枚举类型**”。在实现的时候，需要考虑类型约束和安全性的问题。

实现类似枚举类型功能的方式有很多种，本文只是介绍了使用自定义类型的方式，如果理解了核心思想，我们还可以使用结构体等方式来实现类似枚举类型的功能。