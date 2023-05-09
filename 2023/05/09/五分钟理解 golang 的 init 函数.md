> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/34211611) ![](https://pic4.zhimg.com/v2-2f1d9a4f23f63da8c0fd6c3fcb17d8fb_b.jpg)

大家都知道 golang 里的 main 函数是程序的入口函数，main 函数返回后，程序也就结束了。golang 还有另外一个特殊的函数 init 函数，先于 main 函数执行，实现包级别的一些初始化操作，今天我们就深入介绍下 init 的特性。

**init 函数的主要作用：**

*   初始化不能采用初始化表达式初始化的变量。
*   程序运行前的注册。
*   实现 sync.Once 功能。
*   其他

**init 函数的主要特点：**

*   init 函数先于 main 函数自动执行，不能被其他函数调用；
*   init 函数没有输入参数、返回值；
*   每个包可以有多个 init 函数；
*   **包的每个源文件也可以有多个 init 函数**，这点比较特殊；
*   同一个包的 init 执行顺序，golang 没有明确定义，编程时要注意程序不要依赖这个执行顺序。
*   不同包的 init 函数按照包导入的依赖关系决定执行顺序。

**golang 程序初始化**

golang 程序初始化先于 main 函数执行，由 runtime 进行初始化，初始化顺序如下：

1.  初始化导入的包（包的初始化顺序并不是按导入顺序（“从上到下”）执行的，runtime 需要解析包依赖关系，没有依赖的包最先初始化，与变量初始化依赖关系类似，参见 [golang 变量的初始化](https://mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
2.  初始化包作用域的变量（该作用域的变量的初始化也并非按照 “从上到下、从左到右” 的顺序，runtime 解析变量依赖关系，没有依赖的变量最先初始化，参见 [golang 变量的初始化](https://mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
3.  执行包的 init 函数；

示例 1：

**main.go**

```
package main                                                                                                                     

import (
   "fmt"              
)

var T int64 = a()

func init() {
   fmt.Println("init in main.go ")
}

func a() int64 {
   fmt.Println("calling a()")
   return 2
}
func main() {                  
   fmt.Println("calling main")     
}

```

输出：

```
calling a()
init in main.go
calling main

```

初始化顺序：**变量初始化 ->init()->main()**

示例 2：

**pack.go**

```
package pack                                                                                                                     

import (
   "fmt"
   "test_util"
)

var Pack int = 6               

func init() {
   a := test_util.Util        
   fmt.Println("init pack ", a)    
} 


```

**test_util.go**

```
package test_util                                                                                                                

import "fmt"

var Util int = 5

func init() {
   fmt.Println("init test_util")
}  

```

**main.go**

```
package main                                                                                                                     

import (
   "fmt"
   "pack"
   "test_util"                
)

func main() {                  
   fmt.Println(pack.Pack)     
   fmt.Println(test_util.Util)
}


```

输出：

```
init test_util
init pack  5
6
5

```

**由于 pack 包的初始化依赖 test_util，因此运行时先初始化 test_util 再初始化 pack 包；**

示例 3：

**sandbox.go**

```
package main
import "fmt"
var _ int64 = s()
func init() {
   fmt.Println("init in sandbox.go")
}
func s() int64 {
   fmt.Println("calling s() in sandbox.go")
   return 1
}
func main() {
   fmt.Println("main")
}

```

**a.go**

```
package main
import "fmt"
var _ int64 = a()
func init() {
   fmt.Println("init in a.go")
}
func a() int64 {
   fmt.Println("calling a() in a.go")
   return 2
}

```

**z.go**

```
package main
import "fmt"
var _ int64 = z()
func init() {
   fmt.Println("init in z.go")
}
func z() int64 {
   fmt.Println("calling z() in z.go")
   return 3
}

```

输出：

```
calling a() in a.go
calling s() in sandbox.go
calling z() in z.go
init in a.go
init in sandbox.go
init in z.go
main

```

**同一个包不同源文件的 init 函数执行顺序，golang spec 没做说明，以上述程序输出来看，执行顺序是源文件名称的字典序。**

示例 4：

```
package main
import "fmt"
func init() {
   fmt.Println("init")
}
func main() {
   init()
}

```

**init 函数不可以被调用，上面代码会提示：undefined: init**

示例 5：

```
package main
import "fmt"
func init() {
   fmt.Println("init 1")
}
func init() {
   fmt.Println("init 2")
}
func main() {
   fmt.Println("main")
}

```

输出：

```
init 1
init 2
main

```

**init 函数比较特殊，可以在包里被多次定义。**

示例 6：

```
var initArg [20]int
func init() {
   initArg[0] = 10
   for i := 1; i < len(initArg); i++ {
       initArg[i] = initArg[i-1] * 2
   }
}

```

**init 函数的主要用途：初始化不能使用初始化表达式初始化的变量**

示例 7：

import _ "net/http/pprof"

**golang 对没有使用的导入包会编译报错，但是有时我们只想调用该包的 init 函数，不使用包导出的变量或者方法，这时就采用上面的导入方案。**

执行上述导入后，init 函数会启动一个异步协程采集该进程实例的资源占用情况，并以 http 服务接口方式提供给用户查询。

[http://weixin.qq.com/r/mi7v9x7EDUNyrVqU93uI](http://weixin.qq.com/r/mi7v9x7EDUNyrVqU93uI) (二维码自动识别)

想了解更多 golang 编程相关知识，**请扫描上面二维码关注微信公众号 bloomingTony**