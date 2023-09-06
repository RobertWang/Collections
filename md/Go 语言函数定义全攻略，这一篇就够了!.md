> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/8OTdCx5r-FGqCsXAkdor3w)

> 函数定义是 Go 语言的重要组成部分, 本文全面介绍函数定义的语法、多返回值、匿名函数等, 一篇文章让你彻底掌握函数定义。

**/** Go 语言函数定义与声明详解 **/**

函数是组织代码的基本单元, Go 语言中函数的声明和定义有一些特殊的设计。本文将详细介绍 Go 语言函数定义和声明的相关用法。

**一、函数定义**

定义一个函数的基本语法:

```
func functionName(input1 type1, input2 type2) (output1 type1, output2 type2) {
  // 函数体实现
  // 返回多个值
  return value1, value2 
}

```

函数定义包含函数名、参数列表、返回值列表和函数体。

**二、参数定义**

函数可以定义多种类型的形参:

```
func sum(numbers []int) int {
  // 求和算法
  return sum
}

```

参数可以为数组、切片、map、结构体等任意类型。

**三、多返回值**

Go 语言函数可以返回多个值:

```
func sumAndProduct(numbers []int) (int, int) {
  sum := 0 
  product := 1
  // 计算过程
  return sum, product
}

```

返回值可以命名, 并在函数体直接返回。

**四、命名返回值**

返回值列表可以命名, 然后直接使用这些变量:

```
func sum(nums []int) (s int) {
  // 使用返回值变量s
  for _, n := range nums {
    s += n
  }
  return
}

```

return 时可以不必写返回值变量名。

**五、匿名函数**

可以使用函数字面量定义匿名函数:

```
func(input1 type1, input2 type2) (output1 type1, output2 type2) {
  // 匿名函数体
}

```

一般会将匿名函数保存到变量中使用。

**六、方法**

Go 语言中的方法实际上也是函数:

```
type User struct {
  Id int
  Name string
}
func (user *User) Greet() {
  fmt.Printf("Hello %s", user.Name)
}

```

方法需要定义接收者, 指明方法属于哪个类型。

**七、函数指针**

函数也是一种类型, 可以取地址得到函数指针:

```
func sum(numbers []int) int {
  // 求和 
}
var sumFunc = sum // 函数值
var sumPtr = &sum // 函数指针

```

函数指针可以支持回调模式。

**八、类型定义**

可以使用 type 定义函数类型别名:

```
type SumFunc func([]int) int
func main() {
  var f SumFunc 
  f = sum
  fmt.Println(f([]int{1, 2})) // 类似函数指针
}

```

函数类型与函数指针类似, 可以作为回调使用。

**九、闭包函数**

Go 支持闭包的函数定义:

```
func adder() func(int) int {
  sum := 0
  return func(x int) int {
    sum += x
    return sum
  }
}

```

闭包函数可以构成状态机、累加器等模型。

**十、 defer**

defer 可以推迟函数执行到外层返回之前:

```
func readFile() error {
  f, err := os.Open() 
  if err != nil {
    return err
  }
  defer f.Close() // 函数结束前关闭
  // 处理文件
} 

```

defer 经常用于资源清理、错误处理等场景。

以上内容详细介绍了 Go 语言中函数的声明、定义、语法特性等用法, 可以帮助大家全面理解 Go 语言函数的设计。