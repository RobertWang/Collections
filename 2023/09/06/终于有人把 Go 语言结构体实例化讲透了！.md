> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MVha70OjKEPVL0RFipG0-A)

> *   结构体实例的内存布局
>     
> *   使用 new 关键字分配内存
>     
> *   各种初始化语法
>     
> *   通过工厂方法创建实例
>     
> *   自定义构造函数初始化
>     
> *   常见初始化错误及解决方法
>     

> 1.  结构体各个字段的内存区域
>     
> 2.  内存对齐填充
>     
> 3.  可选的附加信息, 如方法集等
>     

**三、使用 new 关键字分配内存**

Go 语言中可以使用 new 关键字为结构体分配内存:

```
type Person struct {
  Name string
  Age  int
}
p := new(Person)

```

这将分配 Person 类型所需的内存, 并返回结构体地址, 类型为 * Person 指针。

我们需要使用指针才能访问结构体字段:

```
p.Name = "John" 
p.Age = 18

```

new 仅分配内存, 不进行初始化。初始化工作需要我们自己完成。

**四、常见的初始化语法**

Go 语言中常见的结构体初始化语法有:

1、值列表初始化

```
p := Person{Name:"John", Age:18}

```

通过 Field:Value 形式初始化各个字段。

2、值列表初始化

```
p := Person{"John", 18}

```

直接通过值列表初始化, 值的顺序必须与声明时一致。

3、部分初始化

```
p := Person{Name:"John"}

```

只初始化部分字段, 其余为空值。

4、使用变量或表达式初始化

```
name := "John"
age := 18
p := Person{name, age}

```

字段值可以通过变量或表达式计算产生。

**五、通过工厂方法创建实例**

我们可以定义一个工厂方法负责实例化结构体:

```
func NewPerson(name string, age int) *Person {
  return &Person{
    Name: name,
    Age: age, 
  }
}
p := NewPerson("John", 18)

```

这种方式将实例化的逻辑抽象到工厂方法中, 使用时更简洁。

还可以在工厂方法中自定义必要的初始化逻辑。

**六、自定义构造函数初始化**

Go 语言中常见的模式是在结构体上定义一个构造函数方法:

```
type Person struct {
  Name string
  Age  int
}
func (p *Person) Constructor(name string, age int) {
  p.Name = name
  p.Age = age
}
p := new(Person)
p.Constructor("John", 18)

```

这种方式允许我们自定义初始化逻辑, 非常灵活。

构造函数也可以返回这个实例本身, 实现链式调用。

**七、初始化错误及解决方法**

结构体初始化中常见的错误包括:

> 1.  未初始化或部分初始化导致空指针错误
>     
> 2.  值顺序不匹配导致结果错误
>     
> 3.  循环依赖无法初始化
>     

主要的解决方法是:

> 1.  检查每个字段是否已初始化
>     
> 2.  按定义顺序传入值
>     
> 3.  合理设计结构, 避免循环依赖
>     

另外设置初始默认值也可以避免部分问题。

**八、完整示例**

我们通过一个完整的示例, 来看看结构体初始化和自定义构造函数的实际使用:

```
// 用户结构体
type User struct {
  Id     int
  Name   string
  Age    int
  Addr   string
}
// 构造函数
func (user *User) Constructor(id int, name string, age int) *User {
  // 初始化必须字段
  user.Id = id
  user.Name = name
  user.Age = age
  // 返回实例本身,链式调用
  return user 
}
// 设置地址
func (user *User) SetAddr(addr string) *User {
  user.Addr = addr
  return user
}
func main() {
  // 链式调用构造对象
  u := new(User).Constructor(1, "John", 18).SetAddr("Beijing")
  fmt.Println(u) // 输出用户信息
}

```

这实现了一个灵活可扩展的结构体初始化方式。

**九、总结**

Go 语言中实例化结构体需要为其分配内存并正确初始化。我们可以采用 new 关键字、各种初始化语法、工厂方法等方式来实现这一过程。

合理的实例化结构体可以使代码更简洁可维护。同时避免许多空指针和数据错误问题。这是使用结构体的基础, 希望本文可以帮助大家更好地理解和运用这些知识。