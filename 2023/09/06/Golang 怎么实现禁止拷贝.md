> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VRSgFRdEjrz1KMz5JMwGvA)

> 一、前言禁止拷贝即是用户无法对对象进行拷贝，比如源码包中的 sync.waitgroup，就是不可以拷贝的。二

**一、前言**

禁止拷贝即是用户无法对对象进行拷贝，比如源码包中的 sync.waitgroup，就是不可以拷贝的。

**二、如何实现禁止复制**

查看 sync 包下的源码得知看一下 go 的源码如何实现禁止拷贝的，如下：

```
type noCopy struct{}
// Lock is a no-op used by -copylocks checker from `go vet`.
func (*noCopy) Lock()   {}
func (*noCopy) Unlock() {}

```

noCopy 是一个 golang 基础包内部的结构体（小写开头，说明不对外暴露，不想让用户调用）。

当我们声明的结构体里包含 noCopy 时：

```
package main
import (
  "fmt"
)
type noCopy struct{}
// Lock is a no-op used by -copylocks checker from `go vet`.
func (*noCopy) Lock()   {}
func (*noCopy) Unlock() {}
type DoNotCopyMe struct {
  noCopy
}
func main() {
  var d DoNotCopyMe
  d2 := d // 这里发生了拷贝
  fmt.Println(d2)
}

```

直接运行的话，可以正常输出：

```
> go run .\main.go
{{}}

```

 只有执行 go vet 检查才可以发现问题，如下：

```
> go vet .\main.go
# command-line-arguments
.\main.go:20:8: assignment copies lock value to d2: command-line-arguments.DoNotCopyMe    
.\main.go:21:14: call of fmt.Println copies lock value: command-line-arguments.DoNotCopyMe

```

如果错误的使用不能被 copy 的代码审核，运行命令时不会看到输出。

从上面的实验我们可以得出两点结论：

1.  如果一个结构体实现了 Lock 和 Unlock 函数，原则上就不能被拷贝。
    
2.  但是编译器没有做限制，如果想知道代码里是否有错误的用法，需要用 go vet 命令检查。
    

**三、实现案例**  

```
package main
import (
  "fmt"
)
type noText struct{}
func (*noText) Lock()   {}
func (*noText) Unlock() {}
type Demo struct {
  noCopy noText
}
func Copy(d Demo) {
  CopyTwice(d)
}
func CopyTwice(d Demo) {}
func main() {
  d := Demo{}
  fmt.Printf("%+v", d)
  Copy(d)
  fmt.Printf("%+v", d)
}

```