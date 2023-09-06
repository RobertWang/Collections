> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/BxKoRUTMzowo6bJ3LeRSNA)

**01** 

介绍

Golang 配置信息管理库 Viper[1]，它提供一套完整的管理配置信息的解决方案。

Go 语言中很多知名开源项目也都选择使用 Viper，它功能非常强大，本文介绍 Viper 读取结构体嵌套配置信息的使用方式。

**02** 

读取结构体嵌套配置信息

在实际项目开发中，我们经常会遇到一些比较复杂的配置信息，比如多层嵌套的配置信息，在结构体中嵌套结构体和切片。

```
user_data:
  uid: 10000
  uname: "frank"
  other_info:
    email: "gopher@email.cn"
    address: "Beijing China"
  language:
    - name: "go"
      score: 90
    - name: "php"
      score: 95
    - name: "JavaScript"
      score: 80


```

阅读上面 `yaml` 文件，`user_data` 是一个多层嵌套的配置信息。

读取该多层嵌套配置信息，如果我们使用 `GetXXX` 函数获取值，代码会非常繁琐。

Viper 提供了 2 个解析函数，`Unmarshal` 和 `UnmarshalKey`，我们可以使用它们非常方便地读取多层嵌套配置信息，可以将所有或指定配置信息解析到 `struct`、`map` 等数据结构中。

我们通过示例代码，介绍它们的使用方式。

目录：

```
├── configs
│   ├── default.yaml
│   └── test.yaml
├── go.mod
├── go.sum
└── main.go


```

示例代码：

```
package main

import (
 "fmt"
 "github.com/spf13/viper"
)

func main() {
 v := viper.New()
 //v.SetConfigFile("./configs/test.yaml")
 v.SetConfigFile("./configs/default.yaml")
 err := v.ReadInConfig()
 if err != nil {
  fmt.Println(err)
  return
 }
 //err = v.Unmarshal(&userData)
 err = v.UnmarshalKey("user_data", &userData)
 if err != nil {
  fmt.Println(err)
  return
 }
 fmt.Printf("userData=%+v\n", userData)
}

type UserData struct {
 Uid       int        `json:"uid"`
 Uname     string     `json:"uname"`
 OtherInfo OtherInfo  `json:"other_info" mapstructure:"other_info"`
 Language  []Language `json:"language" mapstructure:"language"`
}

type OtherInfo struct {
 Email   string
 Address string
}

type Language struct {
 Name  string
 Score int
}

var userData UserData


```

输出结果：

```
userData={Uid:10000 Uname:frank OtherInfo:{Email:gopher@email.cn Address:Beijing China} Language:[{Name:go Score:90} {Name:php Score:95} {Name:JavaScript Score:80}]}


```

阅读上面这段代码，结构体 `UserData` 中嵌套结构体 `OtherInfo` 和切片 `Language`，我们使用 Viper 提供的 tag 标签 `mapstructure`，将读取到的配置信息解析到 `struct` 中。

需要注意的是，解析指定配置信息使用 `UnmarshalKey` 函数，解析全部配置信息使用 `Unmarshal`，二者的 `yaml` **文件格式也不一样**，读者朋友们小心踩 “坑”。

**03** 

总结

本文我们通过示例代码，介绍怎么使用 Viper 读取嵌套配置信息，它提供两个函数 `Unmarshal` 和 `UnmarshalKey`，分别用于解析全部配置信息，和解析指定配置信息。

需要注意的是，针对结构体中的嵌套结构体类型或切片类型的字段，我们需要使用 Viper 提供的 tag 标签 `mapstructure`，否则将无法读取到配置信息的内容。

此外，yaml 格式也需要熟练掌握，尽量不要因为 yaml 格式不对，导致解析不出配置信息中的内容。  

### 参考资料

[1]

Viper: _https://github.com/spf13/viper_