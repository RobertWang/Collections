> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/lWe3dP4lP25btK26qWy8ww)

> 比较有名的方案有使用 viper 管理配置 [1] 支持多种配置文件格式，包括 JSON,TOML,YAML,HEC

比较有名的方案有

使用 viper 管理配置 [1]
-----------------

*   支持多种配置文件格式，包括 JSON,TOML,YAML,HECL,envfile，甚至还包括 Java properties
    
*   支持为配置项设置默认值
    
*   可以通过命令行参数覆盖指定的配置项
    
*   支持参数别名
    

viper[2] 按照这个优先级（从高到低）获取配置项的取值：

*   explicit call to Set: 在代码逻辑中通过 viper.Set() 直接设置配置项的值
    
*   flag：命令行参数
    
*   env：环境变量
    
*   config：配置文件
    
*   key/value store：etcd 或者 consul
    
*   default：默认值
    

按照这个优先级（从高到低）获取配置项的取值：

*   explicit call to Set: 在代码逻辑中通过 viper.Set() 直接设置配置项的值
    
*   flag：命令行参数
    
*   env：环境变量
    
*   config：配置文件
    
*   key/value store：etcd 或者 consul
    
*   default：默认值
    

### 优先级

#### 验证一下 viper.Set() 的优先级高于 配置文件

```
package main

import (
 "fmt"

 "github.com/spf13/viper"
)

func main() {

 loadConfig()
}

func loadConfig() {

 configVar := "shuang-config.yaml"
 configVar = "" // 这行如果注释掉，则从指定的configVar读取配置文件；否则就各种条件去找了

 viper.Set("Global.Source", "优先级最高")

 if configVar != "" {
  // SetConfigFile 显式定义配置文件的路径、名称和扩展名。
  // Viper 将使用它而不检查任何配置路径。
  viper.SetConfigFile(configVar)
 } else {

  // 如果没有显式指定配置文件，则

  // 会去下面的路径里找文件名`cui-config`的文件  name of config file (without extension)
  // 按照 []string{"json", "toml", "yaml", "yml", "properties", "props", "prop", "hcl", "tfvars", "dotenv", "env", "ini"}的顺序(居然还支持Java用的properties)
  viper.SetConfigName("cui-config")
  viper.AddConfigPath("/etc/myapp") // 找寻的路径
  viper.AddConfigPath("$HOME/.myapp/")
  viper.AddConfigPath(".")
 }

 err := viper.ReadInConfig()
 if err != nil {
  panic(fmt.Errorf("error reading config: %s", err))
 }

 fmt.Printf("到底用的是哪个配置文件: '%s'\n", viper.ConfigFileUsed())

 fmt.Printf("Global.Source这个字段的值为: '%s'\n", viper.GetString("global.source"))
}


```

输出:

```
到底用的是哪个配置文件: '/Users/fliter/config-demo/cui-config.yaml'
Global.Source这个字段的值为: '优先级最高'


```

#### 验证一下 环境变量 的优先级高于 配置文件

```
package main

import (
 "fmt"

 "github.com/spf13/viper"
)

func main() {

 loadConfig()
}

func loadConfig() {

 configVar := "shuang-config.yaml"
 configVar = "" // 这行如果注释掉，则从指定的configVar读取配置文件；否则就各种条件去找了

 viper.Set("Global.Source", "优先级最高")
 viper.AutomaticEnv()

 if configVar != "" {
  // SetConfigFile 显式定义配置文件的路径、名称和扩展名。
  // Viper 将使用它而不检查任何配置路径。
  viper.SetConfigFile(configVar)
 } else {

  // 如果没有显式指定配置文件，则

  // 会去下面的路径里找文件名`cui-config`的文件  name of config file (without extension)
  // 按照 []string{"json", "toml", "yaml", "yml", "properties", "props", "prop", "hcl", "tfvars", "dotenv", "env", "ini"}的顺序(居然还支持Java用的properties)
  viper.SetConfigName("cui-config")
  viper.AddConfigPath("/etc/myapp") // 找寻的路径
  viper.AddConfigPath("$HOME/.myapp/")
  viper.AddConfigPath(".")
 }

 err := viper.ReadInConfig()
 if err != nil {
  panic(fmt.Errorf("error reading config: %s", err))
 }

 fmt.Printf("到底用的是哪个配置文件: '%s'\n", viper.ConfigFileUsed())

 fmt.Printf("LANG这个字段的值为: '%s'\n", viper.GetString("LANG"))
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCJ4zXVicurLlvtd8LEicyHOWiak6F6CnFJhumXnrFjYXIGLfX7zQlObic4x5LzftiaAvNEb4AAkeZSXvQ/640?wx_fmt=png)

viper.AutomaticEnv() 会绑定所有环境变量，

如果只希望绑定特定的，可以使用 SetEnvPrefix("global.source", "MYAPP_GLOAL_SOURCE")，注意这个函数不会自动加上 MYAPP 的前缀.

#### 验证一下 命令行参数的优先级高于 配置文件

viper 可以配合 pflag 来使用，pflag 可以理解为标准库 flag 的一个增强版，viper 可以绑定到 pflag 上

和 cobra，viper 一样，pflag 也是同一作者的作品

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCJ4zXVicurLlvtd8LEicyHOWeibnhrMv9RFF3u7Ld9aPf9u0K6Ukbgmuz9aQvsJP3BPcpuJdRyG9R7Q/640?wx_fmt=png)

#### 验证一下 默认值的优先级低于 配置文件

```
package main

import (
 "fmt"

 "github.com/spf13/viper"
)

func main() {

 loadConfig()
}

func loadConfig() {

 configVar := "shuang-config.yaml"
 configVar = "" // 这行如果注释掉，则从指定的configVar读取配置文件；否则就各种条件去找了

 //viper.Set("Global.Source", "优先级最高")
 viper.AutomaticEnv()

 viper.SetDefault("Global.Source", "优先级最低")

 if configVar != "" {
  // SetConfigFile 显式定义配置文件的路径、名称和扩展名。
  // Viper 将使用它而不检查任何配置路径。
  viper.SetConfigFile(configVar)
 } else {

  // 如果没有显式指定配置文件，则

  // 会去下面的路径里找文件名`cui-config`的文件  name of config file (without extension)
  // 按照 []string{"json", "toml", "yaml", "yml", "properties", "props", "prop", "hcl", "tfvars", "dotenv", "env", "ini"}的顺序(居然还支持Java用的properties)
  viper.SetConfigName("cui-config")
  viper.AddConfigPath("/etc/myapp") // 找寻的路径
  viper.AddConfigPath("$HOME/.myapp/")
  viper.AddConfigPath(".")
 }

 err := viper.ReadInConfig()
 if err != nil {
  panic(fmt.Errorf("error reading config: %s", err))
 }

 fmt.Printf("到底用的是哪个配置文件: '%s'\n", viper.ConfigFileUsed())

 fmt.Printf("Global.Source这个字段的值为: '%s'\n", viper.GetString("Global.Source"))
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCJ4zXVicurLlvtd8LEicyHOWEGcdUbjQvlycBnfs4ZlfMc6U5Uvlu6nZdnv4UEUhWlmPyZ8vUqF9mQ/640?wx_fmt=png)

### Watch 机制 (配置更新后 热加载)

该机制可以监听配置文件的修改, 这样就实现了热加载，修改配置后，无需重启服务

*   对于本地文件，是通过`fsnotify`实现的，然后通过一个回调函数去通知应用来 reload;
    
*   对于 Remote KV Store，目前只支持 etcd，做法比较 ugly，(5 秒钟) 轮询一次 而不是 watch api
    

```
package main

import (
 "fmt"
 "time"

 "github.com/fsnotify/fsnotify"
 "github.com/spf13/viper"
)

func main() {

 loadConfig()
}

func loadConfig() {

 configVar := "shuang-config.yaml"
 configVar = "" // 这行如果注释掉，则从指定的configVar读取配置文件；否则就各种条件去找了

 if configVar != "" {
  // SetConfigFile 显式定义配置文件的路径、名称和扩展名。
  // Viper 将使用它而不检查任何配置路径。
  viper.SetConfigFile(configVar)
 } else {

  // 如果没有显式指定配置文件，则

  // 会去下面的路径里找文件名`cui-config`的文件  name of config file (without extension)
  // 按照 []string{"json", "toml", "yaml", "yml", "properties", "props", "prop", "hcl", "tfvars", "dotenv", "env", "ini"}的顺序(居然还支持Java用的properties)
  viper.SetConfigName("cui-config")
  viper.AddConfigPath("/etc/myapp") // 找寻的路径
  viper.AddConfigPath("$HOME/.myapp/")
  viper.AddConfigPath(".")
 }

 viper.WatchConfig()
 viper.OnConfigChange(func(e fsnotify.Event) {
  fmt.Printf("配置文件 %s 发生了更改!!! 最新的Global.Source这个字段的值为 %s:", e.Name, viper.GetString("Global.Source"))
 })

 err := viper.ReadInConfig()
 if err != nil {
  panic(fmt.Errorf("error reading config: %s", err))
 }

 fmt.Printf("到底用的是哪个配置文件: '%s'\n", viper.ConfigFileUsed())

 fmt.Printf("Global.Source这个字段的值为: '%s'\n", viper.GetString("Global.Source"))

 time.Sleep(10000e9)
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCJ4zXVicurLlvtd8LEicyHOW0hJphNLWOwCtIOfkmGe3GJoBQVwhuHrQPXO8OfqtytCLa3joXXb0Gg/640?wx_fmt=png)

Go viper 配置文件读取工具 [3]

动态获取配置文件 (viper）[4]

configor[5]
-----------

Configor: 一个 Golang 配置工具，支持 YAML，JSON，TOML，Shell 环境，支持热加载

出自 jinzhu 大佬 [6]

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCJ4zXVicurLlvtd8LEicyHOWw5KsGqT5miaiaibkzBOhTTO2IFWfBflurqWia8oPfppMbdicyyL2KeMJnBw/640?wx_fmt=png)

```
package main

import (
 "fmt"

 "github.com/jinzhu/configor"
)

type Config struct {
 APPName string `default:"app name"`
 DB      struct {
  Name     string
  User     string `default:"root"`
  Password string `required:"true" env:"DBPassword"`
  Port     uint   `default:"3306"`
 }
 Contacts []struct {
  Name  string
  Email string `required:"true"`
 }
}

func main() {
 var conf = Config{}
 err := configor.Load(&conf, "config.yml")

 // err := configor.New(&configor.Config{Debug: true}).Load(&conf, "config.yml")  // 测试模式，也可以通过环境变量开启测试模式(CONFIGOR_DEBUG_MODE=true go run main.go )，这样就无需修改代码

 //err := configor.New(&configor.Config{Verbose: true}).Load(&conf, "config.yml") // 模式，也可以通过环境变量开启详细模式(CONFIGOR_VERBOSE_MODE=true go run main.go )，这样就无需修改代码
 if err != nil {
  panic(err)
 }
 fmt.Printf("%v \n", conf)
}


```

### 开启 测试模式 or 详细模式

既可以在代码中显式开启，如 `err := configor.New(&configor.Config{Debug: true}).Load(&conf, "config.yml")`

也可以通过环境变量开启，如 `CONFIGOR_DEBUG_MODE=true go run main.go`

### 加载多个配置文件

```
// application.yml 的优先级 大于 database.json, 排在前面的配置文件优先级大于排在后的的配置
configor.Load(&Config, "application.yml", "database.json")


```

### 根据环境变量加载配置文件  or 从 shell 加载配置项

详细可参考 Golang Configor 配置文件工具 [7]

### 热更新

```
package main

import (
 "fmt"
 "time"

 "github.com/jinzhu/configor"
)

type Config struct {
 APPName string `default:"app name"`
 DB      struct {
  Name     string
  User     string `default:"root"`
  Password string `required:"true" env:"DBPassword"`
  Port     uint   `default:"3306"`
 }
 Contacts []struct {
  Name  string
  Email string `required:"true"`
 }
}

func main() {
 var conf = Config{}

 // reload模式,可实现热加载

 err := configor.New(&configor.Config{
  AutoReload:         true,
  AutoReloadInterval: time.Second,
  AutoReloadCallback: func(config interface{}) {
   // config发生变化后出发什么操作
   fmt.Printf("配置文件发生了变更%#v\n", config)
  },
 }).Load(&conf, "config.yml")

 // 无reload模式
 //err := configor.Load(&conf, "config.yml")

 // err := configor.New(&configor.Config{Debug: true}).Load(&conf, "config.yml")  // 测试模式，也可以通过环境变量开启测试模式(CONFIGOR_DEBUG_MODE=true go run main.go )，这样就无需修改代码

 //err := configor.New(&configor.Config{Verbose: true}).Load(&conf, "config.yml") // 模式，也可以通过环境变量开启详细模式(CONFIGOR_VERBOSE_MODE=true go run main.go )，这样就无需修改代码
 if err != nil {
  panic(err)
 }
 fmt.Printf("%v \n", conf)

 time.Sleep(100000e9)
}


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Q1ZRKiavkSiaCJ4zXVicurLlvtd8LEicyHOWe6ouG3PaU9tQsQRBnzq8H1AEumyOVfZTVU5TC8ApKzEITIAznPGFyQ/640?wx_fmt=png)

完整代码 [8]

### 参考资料

[1]

使用 viper 管理配置: _https://cloud.tencent.com/developer/article/1540672_

[2]

viper: _https://github.com/spf13/viper_

[3]

Go viper 配置文件读取工具: _https://cloud.tencent.com/developer/article/1677426_

[4]

动态获取配置文件 (viper）: _https://segmentfault.com/a/1190000022828484_

[5]

configor: _https://github.com/jinzhu/configor_

[6]

jinzhu 大佬: _https://github.com/jinzhu_

[7]

Golang Configor 配置文件工具: _https://www.jianshu.com/p/f826d2cc361b_

[8]

完整代码: _https://github.com/cuishuang/config-demo_