> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/307acf37adc5)

在 Go 的函数中，如果要捕获内部的 panic 函数，并将该 panic 作为 error 返回一般写法是:

```
func test() error {
    var err error
    defer func() {
        if r := recover(); r != nil {
            err = errors.New(fmt.Sprintf("%s", r))
        }
    }()
    raisePanic()
    return err
}


```

但这样的写法无法将 raisePanic 函数 panic 出来的信息返回出来，**即便 panic 最后函数的返回值也是 nil**  
**_正确的写法应该将函数签名的返回值改为命名返回值_**，如下：

```
func test() (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = errors.New(fmt.Sprintf("%s", r))
        }
    }()
    raisePanic()
    return err
}


```