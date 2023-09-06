> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/meJKzrSf10sjX5vUDLW-yg)

> 你一定遇到过 Go 语言传参不生效的问题，这里提供传参的正确打开方式，解决你的传参困扰！

**/** Go 语言函数参数传递效果测试详解 **/**

在 Go 语言中, 理解函数参数的传递机制很重要。参数传递对程序的行为有直接的影响。

本文将详细探讨 Go 语言中参数传递的各种效果, 主要内容包括:

> 1.  传值效果
>     
> 2.  指针传递
>     
> 3.  结构体传递
>     
> 4.  map 传递
>     
> 5.  channel 传递
>     
> 6.  切片传递
>     
> 7.  错误传递
>     
> 8.  传递效果示例
>     
> 9.  传递方式选择
>     

通过学习本文, 你将全面理解 Go 语言中参数的传递规则, 包括值类型、引用类型、指针等情况下的传递效果。

**一、传值效果**

Go 语言中向函数传递参数时采用的值传递 (传值调用), 实际上传递的是副本。

```
func foo(i int) {
  i++
}
func main() {
  i := 1
  foo(i)
  fmt.Println(i) // 1
}

```

在 foo() 中修改参数 i 的值, 不会影响到主调函数 main() 中 i 的值。

这是因为 foo() 中接收到的是 i 的一份副本, 函数内部对副本的修改不会反映到原始变量上。

****1. 值类型传值****

基本数据类型 (int、float、string 等) 都是值类型, 按值传递。

```
func foo(i int) {
  i = 20 // 改变副本值
}
func main() {
  i := 10
  foo(i)
  fmt.Println(i) // 10
}

```

foo() 内部重新赋值, 不影响外部 i 的值。

****2. 引用类型传值****

数组、slice、map、channel 等都是引用类型, 但传递给函数时同样是按值传递, 传递的是类型的一个副本。

```
func foo(s [2]int) {
  s[0] = 20 // 修改副本数据
}
func main() {
  s := [2]int{10}
  foo(s)
  fmt.Println(s[0]) // 10
}

```

foo() 内修改副本, 不影响外部原 slice。但如果直接重新赋值, 外部则会受影响:

```
func foo(s []int) {
  s = []int{20, 30} // 重新赋值
} 
func main() {
  s := []int{10}
  foo(s)
  fmt.Println(s[0]) // 10 
}

```

**二、指针传递**

要实现修改外部变量的值, 可以通过指针传递。

```
func foo(i *int) {
  *i = 20
}
func main() {
  i := 10
  foo(&i) 
  fmt.Println(i) // 20
}

```

指针 i 存储了 i 的内存地址, 在 foo() 内部通过 * i 可以修改主调函数中 i 的原始值。

****1. 值类型指针****

对基本数据类型, 可以传指针修改值:

```
func foo(i *int) {
  *i = 20
}
func main() {
  i := 10
  foo(&i)
  fmt.Println(i) // 20  
}

```

****2. 引用类型指针****

对于 slice、map 等引用类型, 直接传递指针, 就可以在函数内修改外部变量:

```
func foo(s *[]int) {
  (*s)[0] = 20 
}
func main() {
  s := []int{10}
  foo(&s)
  fmt.Println(s[0]) // 20
} 

```

**三、结构体传递**

结构体传参的情况类似, 传递副本, 可以通过指针修改外部变量。

****1. 结构体传值****

```
type user struct {
  name string
  age  int
}
func foo(u user) {
  u.age = 20
}
func main() {
  u := user{"John", 30}
  foo(u)
  fmt.Println(u.age) // 30
}

```

foo() 内修改 AGE, 不影响外部 u 变量。

****2. 结构体指针****

```
func foo(u *user) {
  u.age = 20 
}
func main() {
  u := user{"John", 30}  
  foo(&u)
  fmt.Println(u.age) // 20
}

```

传指针可以修改外部结构体成员。

**四、map 传递**

map 类型的传递也类似。

****1. map 传值****

传递的是 map 的副本, 无法修改外部变量:

```
func foo(m map[string]int) {
  m["age"] = 20
}
func main() {
  m := map[string]int{"age": 30}
  foo(m)
  fmt.Println(m["age"]) // 30
}

```

****2. map 指针****

传 map 指针可以修改外部变量:

```
func foo(m *map[string]int) {
  (*m)["age"] = 20
}
func main() {
  m := map[string]int{"age": 30}
  foo(&m)
  fmt.Println(m["age"]) // 20  
}

```

**五、channel 传递**

channel 的处理也类似, 传递副本。

****1. channel 传值****

```
func foo(c chan int) {
  c <- 20 
}
func main() {
  c := make(chan int)
  foo(c)
  <-c // 死锁
} 

```

传值无法在两端通信。

****2. channel 指针****

```
func foo(c *chan int) {
  *c <- 20
} 
func main() {
  c := make(chan int)
  foo(&c)
  fmt.Println(<-c) // 20
}

```

通过指针可以修改外部 channel。

**六、切片传递**

切片传参需要特殊考虑, 既可以传值也可以传指针。

**_6.1_**

****1. 切片传值****

可以直接传递切片:

```
func foo(s []int) {
  s[0] = 20
}
func main() {
  s := make([]int,10)
  foo(s)
  fmt.Println(s[0]) // 20
}

```

这与数组不同, 是因为切片包含指向底层数组的指针, 函数内部可以跟踪修改数组。

**_6.2_**

****2. 切片指针****

传递切片指针也可以:

```
func foo(s *[]int) {
  (*s)[0] = 20
}
func main() {
  s := []int{10}
  foo(&s)
  fmt.Println(s[0]) // 20
} 

```

但通常情况下传切片更常用。

**七、错误传递**

Go 语言中错误处理通过错误接口实现, 传递错误时要注意值或指针。

****1. 错误值传递****

错误作为值传递, 函数间无法共享同一个错误变量:

```
func readFile(path string) (err error) {
  f, err := os.Open(path)
  // ...
  return
}
func main() {
  err := readFile(path) 
  if err != nil {
    // 无法修改函数内部的错误变量
  } 
}

```

这样导致无法修改内部错误。

****2. 指针传递****

应使用指针传递错误:

```
func readFile(path string) (err *error) {
  // 通过*err修改
} 
func main() {
  var err error
  readFile(path, &err)
  if err != nil {
    // 可以检测到错误
  }
} 

```

修改内部 * err 可以将错误状态传递出来。

**八、传递效果示例**

综合以上几点, 下面我们看一些参数传递的效果示例:

```
type user struct {
  name string
  age int
}
func foo(u user, s []int, m map[string]int, p *int) {
  u.name = "Jack" // 无效
  s[0] = 20       // 有效
  m["count"] = 10 // 无效
  *p = 20         // 有效
}
func main() {
  u := user{"John", 30}
  s := []int{10} 
  m := map[string]int{"count": 1}
  p := 10
  foo(u, s, m, &p)
  fmt.Println(u.name, s[0], m["count"], p)
  // John 20 1 20
}

```

用户结构体、map 传值无效, slice 和指针传递有效。

**九、传递方式选择**

选择传值和传指针需要根据具体场景。主要考虑因素:

> *   函数需不需要修改外部变量
>     
> *   数据大小, 传指针节省内存
>     
> *   传递对象是否为 nil 指针
>     

简单规则:

> *   基础类型选择传值
>     
> *   大型结构体或接口传指针
>     
> *   需要修改外部变量时使用指针
>     

合理使用传值和传指针可以提升程序效率。

**十、总结**

Go 语言中参数传递遵循传值的方式, 除非使用指针, 否则无法修改外部变量。

主要注意值类型和引用类型的具体传递效果。错误处理需要传指针。

根据情况选择传值或传指针可以获得最佳性能。

掌握 Go 的参数传递机制, 可以编写出清晰、高效的函数代码。