> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HLzsaveYJQMYouC6iIvZ9w)

> 背景 在 Go 中，对于变量声明，即使没有显示的赋初始值，Go 也默认给变量赋值，具体的值取决于不同类型对

背景
--

在 Go 中，对于变量声明，即使没有显示的赋初始值，Go 也默认给变量赋值，具体的值取决于不同类型对应的零值。本质是 Go 中的变量在创建时即分配内存，并且在内存中分配了对应类型的零值。

哪些类型有零值？
--------

### 基本类型

```
var a int
var b bool
var c string

func main() {
    fmt.Printf("%+v\n", a) // 0
    fmt.Printf("%+v\n", b) // false
    fmt.Printf("%+v\n", c) // ""
}


```

### 复合类型

```
var a [4]int
var b []int
var m map[string]int
var p *int
var f func()
var person Person
var c chan int

type Person struct {
 Name string
 age  int
}

func main() {
    fmt.Printf("%v\n", a) // [0 0 0 0]
    fmt.Printf("%v\n", b) // [] 
    fmt.Printf("%v\n", m) // map[]
    fmt.Printf("%v\n", p) // <nil>
    fmt.Printf("%v\n", f) // <nil>
    fmt.Printf("%v\n", person) <""0>
    fmt.Printf("%v\n", c) // <nil>
}


```

#### 结论

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">类型</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">零值</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;" class="">Integer</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">0</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;" class="">Floating point</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">0.0</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;" class="">Boolean</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">false</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;" class="">String</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">“”</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Array: [4]int</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">[0 0 0 0]</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">struct</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">{field:}</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;" class="">Pointer</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">nil</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Interface</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">nil</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Slice</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">nil</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Map</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">nil</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;" class="">Channel</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">nil</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Function</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">nil</td></tr></tbody></table>

如何对空值判断？
--------

### 基本类型

```
var a *int
var b *bool
var c *string

func main() {
    if a == nil {
        fmt.Println("空值判断", "a == 0")
    }
    if b == nil {
        fmt.Println("空值判断", "b == nil")
    }
    if c == nil {
        fmt.Println("空值判断", "c == nil")
    }
}


```

### 复合类型

```
var a *[4]int
var b []int
var m map[string]int
var p *int
var f func()
var person *Person
var c chan int

type Person struct {
 Name string
 age  int
}

func main() {
    if a == nil {
        fmt.Println("空值判断", "b == nil")
    }
    if b == nil {
        fmt.Println("空值判断", "b == nil")
    }
    if m == nil {
        fmt.Println("空值判断", "m == nil")
    }
    if p == nil {
        fmt.Println("空值判断", "p == nil")
    }
    if f == nil {
        fmt.Println("空值判断", "f == nil")
    }
    if person == nil {
        fmt.Println("空值判断", "person == nil")
    }
    if c == nil {
        fmt.Println("空值判断", "c == nil")
    }
}


```

结论
--

### 对于值类型

> 判断空值只能使用值类型的指针变量。因为你无法根据零值判空，因为零值既有可能是程序员赋值，也有可能是初始化赋值。

*   对于**数组**：`var a [4]int`。默认创建并初始化，可以直接赋值 `a[0] = 1`。
    
*   对于**结构体**：它是值类型。默认创建并初始化，内部 field 初始化成对应类型的空值。
    

### 对于引用类型

> 判断空值利用 **类型变量 == nil**

*   对于 slice 和 map，创建时没有初始化，赋值之前必须初始化。
    

```
var s []int
var m map[string]int

s[0] = 1   // panic
m["0"] = 1 // panic

s = make([]int, 4)
m = make(map[string]int)

s[0] = 1   
m["0"] = 1


```

如何区分零值和空值？
----------

在变量初始化时，Go 自动赋予变量零值`(0, false, “”)`。

我们来看一个例子体会零值和空值的区别。

按需更新，这里指的是接口设计时，规定传来的字段一定是要更新的字段，不需要更新的字段就不要出现。举个例子：

```
// PUT /score
{
 "id": 1,
 "name": "张三",
 "score": 100,
 "create_time": "2021-12-12"
}


```

通过以上 json，我们创建一条数据，这时该如何设计更新接口？

```
// POST /score
{ 
 "id": 1,
 "name": "不对，我叫李四",
 "score": 0,
 "create_time": ""
}


```

如上，假如规定零值字段不更新，只有非零值的字段参与更新。那么，如果用户真的想把 score 字段更新为 0 怎么办，实际上，零值和空值就产生了歧义。

```
// POST /score
{
 "id": 1,
 "name": "我叫李四"
}


```

所以为了避免歧义，我们规定不更新的字段禁止出现在 json 中。同样的，在 json 转为 Go 中的结构体时，我们也只能通过指针来接收：

```
type UpdateScore struct{
 Id int 
 Name *string
 Score *int
 CreateTime *string
}


```

总结
--

这么一梳理，发现确实和我们想象中的有一定的差距。因为我们在实际的零值和空值的判断中，更需要的是对内部字段的数值判断例如：结构体里的某些字段，比较少是只对结构体本身做空值判断。

- END -