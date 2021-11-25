> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5ODYwMjI2MA==&mid=2649765517&idx=1&sn=b5036484e2782e9302e3e98ff0d6d1ea&chksm=becca5f689bb2ce0a20578ec53c3f8e3ab700f48e31a21bc6d27dd2fb0dfd28803117b96e3b1&scene=21#wechat_redirect)

> Golang 的一些编程思维和思想，以及总结一些常见的优雅编程实战技巧。

### 一 Golang 编程思维

*   项目基本架构的组织
    
*   代码基本的编码封装
    
*   代码的基本原则规范
    
*   并发的设计思想
    
*   面向对象编程的设计思想
    
*   可扩展性的设计思想
    

然后就是实践，实实在在的跑一些代码示例，可以自己建立一个 base-code 的项目，里面就是你的各种示例，然后进行一些修改、执行。具体的代码示例可以从官方文档上来，推荐 [Go by Example](https://gobyexample.com/)，里面有大量非常好的例子。也可以自己网上随便搜下，重要的自己要修改并执行，查看和分析结果：[Go 101](https://go101.org/article/101.html)

其次，要理解 Golang 编程思维，首先要理解 Golang 这门语言的创始初衷，初衷就是为了解决好 Google 内部大规模高并发服务的问题，主要核心就是围绕高并发来开展；并且同时又不想引入面向对象那种很复杂的继承关系。首先，就是可以方便的解决好并发问题（包括高并发），那么就需要有并发思维，能够并发处理就通过并发来进行任务分配

*   这个就是涉及到了 context、 goroutine、channel（select）；
    
*   创建大量 goroutine， 但是需要能通过 context、 channel 建立 "父子" 关系，保证子任务可以能够被回收、被主动控制（如 杀死）。
    

再者，面向对象编程思想，利用好 interface、 struct 来实现继承、多态的用法：

*   struct 匿名组合来实现继承；
    
*   terface 和 struct 来实现多态；
    
*   interface 定义接口，尽可能的保持里面的方法定义简单，然后多个 interface 进行组合。
    

然后，理解 Golang 语言本身的一些特性: - 强类型，语法上要注意处理；- GC，实际中要观察 GC 日志并分析；- 注意语法语义尽可能的简单、保持各种类型定义尽可能精简。

最后，从 Golang 社区的一些最佳实践来看，Golang 的各种组件需要尽可能的精简。

*   Golang 中用好的一些开源组件库，都是比较轻量级的，然后可以各自随意组合来达到最佳实践。
    
*   我们自己进行组件封装、模块封装的时候，也是保持这个原则，尽可能的精简，然后使用方进行组合。
    

### 二、Golang 高级编码技巧

#### 1 优雅的实现构造函数编程思想

```
/*
一个更为优雅的构造函数的实现方式
参考：
https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
通过这个方式可以方便构造不同对象，同时避免了大量重复代码
*/

package main
import (
 "fmt"
 "time"

 "golang.org/x/net/context"
)

type Cluster struct {
 opts options
}

type options struct {
 connectionTimeout time.Duration
 readTimeout       time.Duration
 writeTimeout      time.Duration
 logError          func(ctx context.Context, err error)
}

// 通过一个选项实现为一个函数指针来达到一个目的：设置选项中的数据的状态
// Golang函数指针的用法
type Option func(c *options)

// 设置某个参数的一个具体实现，用到了闭包的用法。
// 不仅仅只是设置而采用闭包的目的是为了更为优化，更好用，对用户更友好
func LogError(f func(ctx context.Context, err error)) Option {
 return func(opts *options) {
  opts.logError = f
 }
}

// 对关键数据变量的赋值采用一个方法来实现而不是直接设置
func ConnectionTimeout(d time.Duration) Option {
 return func(opts *options) {
  opts.connectionTimeout = d
 }
}

func WriteTimeout(d time.Duration) Option {
 return func(opts *options) {
  opts.writeTimeout = d
 }
}

func ReadTimeout(d time.Duration) Option {
 return func(opts *options) {
  opts.readTimeout = d
 }
}

// 构造函数具体实现，传入相关Option，new一个对象并赋值
// 如果参数很多，也不需要传入很多参数，只需要传入opts ...Option即可
func NewCluster(opts ...Option) *Cluster {
 clusterOpts := options{}
 for _, opt := range opts {
  // 函数指针的赋值调用
  opt(&clusterOpts)
 }

 cluster := new(Cluster)
 cluster.opts = clusterOpts

 return cluster
}

func main() {

 // 前期储备，设定相关参数
 commonsOpts := []Option{
  ConnectionTimeout(1 * time.Second),
  ReadTimeout(2 * time.Second),
  WriteTimeout(3 * time.Second),
  LogError(func(ctx context.Context, err error) {
  }),
 }

 // 终极操作，构造函数
 cluster := NewCluster(commonsOpts...)

 // 测试验证
 fmt.Println(cluster.opts.connectionTimeout)
 fmt.Println(cluster.opts.writeTimeout)

}
```

除了构造函数这个思想之外，还有一个思想，就是我们要善于利用 struct 封装对象方法，然后再 new 一个对象出来，如下：

```
type Cluster struct {
 opts options
}

func NewCluster(opts ...Option) *Cluster {
   ....

 cluster := new(Cluster)
 cluster.opts = clusterOpts

 return cluster
}
```

#### 2 优雅的实现继承编程思想

Golang 里面没有 C++ 、Java 那种继承的实现方式，但是，我们可以通过 Golang 的匿名组合来实现继承，这里要注意，这个是实际编程中经常用到的一种姿势。具体实现就是一个 struct 里面包含一个匿名的 struct，也就是通过匿名组合，这最基础的基类就是一个 struct 结构，然后定义相关成员变量，然后再定义一个子类，也是一个 struct，里面包含前面的 struct，即可实现继承。

示例代码如下，代码里面有详细的解释：

```
package main
import (
 "fmt"
)

// 【基类】
//定义一个最基础的struct类MsgModel，里面包含一个成员变量msgId
type MsgModel struct {
 msgId   int
 msgType int
}

// MsgModel的一个成员方法，用来设置msgId
func (msg *MsgModel) SetId(msgId int) {
 msg.msgId = msgId
}

func (msg *MsgModel) SetType(msgType int) {
 msg.msgType = msgType
}

//【子类】
// 再定义一个struct为GroupMsgModel，包含了MsgModel，即组合，但是并没有给定MsgModel任何名字，因此是匿名组合
type GroupMsgModel struct {
 MsgModel

 // 如果子类也包含一个基类的一样的成员变量，那么通过子类设置和获取得到的变量都是基类的
 msgId int
}

func (group *GroupMsgModel) GetId() int {
 return group.msgId
}

/*
func (group *GroupMsgModel) SetId(msgId int) {
 group.msgId = msgId
}
*/

func main() {
 group := &GroupMsgModel{}

 group.SetId(123)
 group.SetType(1)

 fmt.Println("group.msgId =", group.msgId, "\tgroup.MsgModel.msgId =", group.MsgModel.msgId)
 fmt.Println("group.msgType =", group.msgType, "\tgroup.MsgModel.msgType =", group.MsgModel.msgType)
}
```

#### 3 优雅的实现虚多态编程思想

面向对象编程中，我们很多情况下，都会定义一个虚基类，然后利用多态去实现各种相似的场景或者说任务。

Golang 里面可以通过 interface + struct 来实现虚基类的用法。interface 用来定义一个 "虚基类"，然后一个 struct 结构定义，用来实现这个 interface 中定义的方法，并且可以有多个类似的 struct 来实现这个 interface，只要实现了这个 interface 中定义的方法即可。这也是典型的多态的一种编程思想，也就是说 Golang 通过接口去实现了多态。

具体流程如下，这个是我实际项目（大型 IM 架构）中的实现方式：

1.  定义一个 interface 接口 MsgModel，包含了一些方法，这个就相当于 "虚基类"
    
    ```
    type MsgModel interface {
        Persist(context context.Context, msg interface{}) bool
        PersistOnSensitive(context context.Context, session_type, level, SensitiveStatus int32, msg interface{}) bool
    }
    ```
    
2.  定义一个类型 msgModelImpl struct{}，用来实现上面的 interface 接口
    
    ```
    定义一个struct用来实现接口类型
    type msgModelImpl struct{}
    
    
    定义一个变量MsgModelImpl等于msgModelImpl，相当于可以通过MsgModelImpl来调用msgModelImpl的成员
    var MsgModelImpl = msgModelImpl{}
    
    实现接口的两个方法
    func (m msgModelImpl) Persist(context context.Context, msgIface interface{}) bool {
    // 具体实现省略
    }
    
    func (m msgModelImpl) UpdateDbContent(context context.Context, msgIface interface{}) bool {
    // 具体实现省略
    }
    ```
    

3.  再定义一个 struct 类型的 msgService，包含上述接口类型 MsgModel，相当于组合了。这样的话，这个类型就需要要实现接口方法。
    
    ```
    type msgService struct {
       msgModel MsgModel
    }
    ```
    

4.  再定义一个变量 MsgService，首字母大写，并且赋值为 msgService 对象，同时给成员 msgModel 赋值为上述已经实现了接口的 struct 对象 MsgModelImpl。
    
    ```
    将上述已经实现接口类型的类型(MsgModelImpl) 赋值给此变量(此变量并且要是包含了接口类型的类型), 然后这个变量就可以供外部调用
    var  MsgService = msgService{
            msgModel:  MsgModelImpl,
    }
    ```
    
5.  这样就全部实现了，后面只要通过 MsgService 中的接口方法就可以调用 interface 中定义的方法
    

*   注意，定义个 MsgService，里面的成员变量 msgModel 赋值为 MsgModelImpl 的目的是为了做封装，对外暴露接口的都是 MsgService，隐藏了内部具体的 MsgModelImpl 实现。
    

6.  小结：
    

*   MsgModel 是一个 interface
    

*   interface 是一组抽象方法的集合，interface 未具体实现的方法，仅包含方法名参数返回值的方法
    

*   msgModelImpl 是一个 struct，它实现了 MsgModel 这个 interface 的所有方法
    

*   如果实现了 interface 中的所有方法，即该类 / 对象就实现了该接口
    

*   MsgModelImpl 是 msgModelImpl 这个 struct 的对象
    
*   msgService 是一个 struct，它包含了 MsgModel，相当于组合
    
*   MsgService 是 msgService 这个 struct 的对象，并对成员变量赋值
    

*   后面就通过 MsgService 对外提供服务，隐藏内部具体的 MsgModelImpl 实现。
    

#### 4 Golang 的 model service 模型【类 MVC 模型】

在一个项目工程中，为了使得代码更优雅，需要抽象出一些模型出来，同时基于 C++ 面向对象编程的思想，需要考虑到一些类、继承相关。在 Golang 中，没有类、继承的概念，但是我们完全可以通过 struct 和 interface 来建立我们想要的任何模型。在我们的工程中，抽象出一种我自认为是类似 MVC 的模型，但是不完全一样，个人觉得这个模型抽象的比较好，容易扩展，模块清晰。对于使用 java 和 PHP 编程的同学对这个模型应该是再熟悉不过了，我这边通过代码来说明下这个模型

1.  首先一个 model 包，通过 interface 来实现，包含一些基础方法，需要被外部引用者来具体实现
    
    ```
    package model
    
    // 定义一个基础model
    type MsgModel interface {
     Persist(context context.Context, msg interface{}) bool
     UpdateDbContent(context context.Context, msgIface interface{}) bool
     GetList(context context.Context, uid, peerId, sinceMsgId, maxMsgId int64, count int) (interface{}, bool)]
    ```
    
2.  再定义一个 msg 包，用来具体实现 model 包中 MsgModel 模型的所有方法
    
    ```
    package msg
    type msgModelImpl struct{}
    var MsgModelImpl = msgModelImpl{}
    func (m msgModelImpl) Persist(context context.Context, msgIface interface{}) bool {
      // 具体实现
    }
    
    func (m msgModelImpl) UpdateDbContent(context context.Context, msgIface interface{}) bool {
        // 具体实现
    }
    
    func GetList(context context.Context, uid, peerId, sinceMsgId, maxMsgId int64, count int) (interface{}, bool)]{
        // 具体实现
    }
    ```
    

3.  model 和 具体实现方定义并实现 ok 后，那么就还需要一个 service 来统筹管理
    
    ```
    package service
    
    // 定义一个msgService struct包含了model里面的UserModel和MsgModel两个model
    type msgService struct {
     userModel  model.UserModel
     msgModel   model.MsgModel
    }
    
    // 定义一个MsgService的变量，并初始化，这样通过MsgService，就能引用并访问model的所有方法
    var (
        MsgService = msgService{
      userModel:      user.UserModelImpl,
      msgModel:       msg.MsgModelImpl,
     }
    )
    ```
    
4.  调用访问
    
    ```
    import service
    service.MsgService.Persist(ctx, xxx)
    ```
    

总结一下，model 对应 MVC 的 M，service 对应 MVC 的 C， 调用访问的地方对应 MVC 的 V

#### 5 Golang 单例模式

单例模式是一种常用的软件设计模式，在它的核心结构中只包含一个被称为单例的特殊类，通过单例模式可以保证系统中一个类有且仅有一个实例且该实例可以被外界访问。

在 Golang 中有一种非常优雅的姿势可以实现，就是通过 sync.Once 来实现，这个也是我在实际项目中所应用的，示例如下：

```
import  "github.com/dropbox/godropbox/singleton"
var SingleService = singleton.NewSingleton(func() (interface{}, error) {
 return &singleMsgProxy{
  MsgModel: msg.MsgModelImpl,
 }, nil
})
```

singleton.NewSingleton 就是具体单例模式的实现，然后赋值给 SingleService，这样，在程序中任何需要获取这个对象的时候，就直接通过 SingleService 来调用，这个调用，系统会保证，里面的 singleMsgProxy 只会被初始化对象一次，这个 singleMsgProxy 就是 new 了一个对象，并且这个对象是只需要被初始化一次的。

#### 6 Golang layout

Golang 工程 Layout 规范，网上有较多探讨，每个人的理解也会不一致，但是有些基础的理解是可以保持统一的：

*   cmd
    

*   main 函数文件目录，这个目录下面，每个文件在编译之后都会生成一个可执行的文件。如果只有一个 app 文件，那就是 main.go。这里面的代码尽可能简单。
    

*   conf
    

*   配置文件，如 toml、yaml 等文件
    

*   config
    

*   配置文件的解析
    

*   docs
    

*   文档
    

*   pkg
    

*   底层各种实现，每一种实现封装一个文件夹
    
*   业界知名开源项目如 Kubernetes、Istio 都是这样的姿势
    

*   build
    

*   编译脚本
    
*   CI 脚本
    
*   上下线脚本
    

*   vendor
    

*   依赖库
    

一个简单示例如下：

```
$ tree  -d  -L 2
├── build
├── cmd
│   ├── apply
│   └── check
├── conf
├── config
├── docs
├── pkg
│   ├── apply
│   ├── check
│   ├── files
│   ├── k8s
│   └── options
└── vendor
```

#### 7 cmd & command & flag

大家看 Kubernetes 的源码就可以发现，会有这么一个现象，Kubernetes 中会有很多二进制程序，然后每个程序，可能会有不同的指令，然后每个指令都会有很多命令行参数。如果大家对 Kubernetes 有一定了解，那么就知道 kubectl 会有如下命令：

```
kubectl apply  -f  进行部署
kubectl delete -f  删除部署
kubectl get pod    获取 Pod
```

那么 kubectl 这个二进制程序，如何能够优雅的支持不同的参数呢？

下面，还是以我实际项目工程中的应用为例，来进行演示。效果如下，程序 example 包含两个命令 apply 和 check，还有一个 help 命令：

```
$ ./example
Usage:
  example[command]

Available Commands:
  apply       apply request by json file
  check       check request validity by json file
  help        Help about any command

Flags:
      --config string   config file[/.xx.yaml] (default "none")
  -h, --help            help for example
      --mode string     mode[cpu  or all] (default "cpu")

Use "example[command] --help" for more information about a command.
```

代码示例如下：

```
package main

import (
 log "github.com/sirupsen/logrus"
 "github.com/spf13/cobra"
 "github.com/spf13/pflag"
 "os"

 "example/cmd/apply"
 "example/cmd/check"
 "example/config"
)

func main() {
 var cmdCheck = check.NewVPARequestCheck()
 var cmdApply = apply.NewVPARequestApply()

 var rootCmd = &cobra.Command{Use: "example"}

 flags := rootCmd.PersistentFlags()
 addFlags(flags)

 rootCmd.AddCommand(cmdApply, cmdCheck)
 if err := rootCmd.Execute(); err != nil {
  panic(err)
 }
}

func addFlags(flags *pflag.FlagSet) {
 flags.StringVar(&config.Cfg.KubeConfig, "config", "none", "config file[/.xx.yaml]")
 flags.StringVar(&config.Cfg.Mode, "mode", "cpu", "mode[cpu  or all]")
}
```

check 命令实现如下，具体 check 相关的 Run 方法忽略：

```
package check

import (
 "fmt"
 log "github.com/sirupsen/logrus"
 "github.com/spf13/cobra"
 "example/config"
 "example/pkg/check"
 "example/pkg/files"
)

type RequestCheckOptions struct {
 configPath string
}

func NewRequestCheckOptions() *RequestCheckOptions {
 o := &RequestCheckOptions{}

 return o
}

func NewVPARequestCheck() *cobra.Command {
 o := NewRequestCheckOptions()
 cmd := &cobra.Command{
  Use:   "check [json file]",
  Short: "check request validity by json file",
  Long:  "check request by new request json file",
  Args:  cobra.MinimumNArgs(1),
  RunE: func(c *cobra.Command, args []string) error {
   if err := o.Run(args); err != nil {
    return err
   }
   return nil
  },
 }

 return cmd
}
```

apply 命令如下，具体 apply 相关的 Run 方法忽略：

```
package apply

import (
 "fmt"
 "github.com/spf13/cobra"
 "example/pkg/apply"
 "example/pkg/files"
)

type RequestApplyOptions struct {
 configPath string
}

func NewRequestApplyOptions() *RequestApplyOptions {
 o := &RequestApplyOptions{}

 return o
}

func NewVPARequestApply() *cobra.Command {
 o := NewRequestApplyOptions()

 cmd := &cobra.Command{
  Use:   "apply [json file]",
  Short: "apply request by json file",
  Long:  "apply request by new request json file",
  Args:  cobra.MinimumNArgs(1),
  RunE: func(c *cobra.Command, args []string) error {
   if err := o.Run(args); err != nil {
    return err
   }
   return nil
  },
 }

 return cmd
}
```

然后只需要在各自的 Run 方法中实现对应的逻辑即可。

**腾讯程序员视频号最新视频**