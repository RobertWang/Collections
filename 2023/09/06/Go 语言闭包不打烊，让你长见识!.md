> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LB0wvz2unTJrInKexs47wg)

> 本文深入探索闭包在 Go 语言中的强大用途, 让你看到闭包更多的可能性, 长见识！

**/** Go 语言闭包详解 **/**

闭包是一种特殊的函数, 它可以记住并访问函数外部的变量。本文主要内容

> 1.  闭包函数
>     
> 2.  调用闭包
>     
> 3.  闭包修改外部变量
>     
> 4.  闭包实际应用
>     
> 5.  闭包注意事项
>     

**一、闭包函数**

闭包通常是一个匿名函数, 在函数体内引用了外部的变量:

```
func outer() func() int {
  x := 0
  return func() int { 
    x++
    return x 
  }
}

```

outer() 函数返回一个闭包, 闭包 Capture 了外部的 x 变量。

**二、调用闭包**

调用闭包时, 它可以访问并修改外部变量:

```
f := outer() 
fmt.Println(f()) 
fmt.Println(f()) 
f2 := outer() 
fmt.Println(f2()) 

```

每个闭包都 Capture 了不同的 x 变量。

**三、闭包修改外部变量**

闭包内部可以修改外部变量:

```
func outer() func() {
  x := 0
  return func() {
    x++ 
    fmt.Println(x)
  }
}

```

这里闭包修改了外部的 x 变量。

**四、闭包实际应用**

闭包常用于:

> 1.  封装函数计算状态  
>     
> 2.  回调函数保存状态
>     

例如实现计算移动平均值:

```
func makeAvg() func(int) float64 {
  nums := []int{}
  return func(n int) float64 {
    nums = append(nums, n)
    total := 0
    for _, num := range nums {
      total += num
    }
    return float64(total / len(nums))
  }
}
avg := makeAvg()
fmt.Println(avg(10)) 
fmt.Println(avg(20)) 
fmt.Println(avg(30)) 

```

**五、闭包注意事项**

> 1、避免在循环中创建闭包, 可能引发错误
> 
> 2、及时释放不再需要的闭包变量, 避免内存泄露

**六、总结**

闭包是一种特殊的函数, 它可以 capture 外部变量, 使得函数可以保持状态。闭包常用于封装计算状态、保存回调函数的执行上下文等场景。

正确使用闭包可以编写出非常灵活优雅的代码, 但也需要注意一些错误用法。总体来说, 闭包是 Go 语言一个很有特色的功能。