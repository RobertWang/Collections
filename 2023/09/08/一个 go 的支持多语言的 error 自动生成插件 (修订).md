> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ghQ9jgxAcAL3KN1aJtISSg)

> 大家好，我是 peachesTao，今天给大家推荐一个 go 的支持多语言的 error 自动生成的插件，插件主页可以

大家好，我是 peachesTao，今天给大家推荐一个 go 的支持多语言的 error 自动生成的插件，插件主页可以访问下方链接。

在一个多语言国际化的项目中，后端接口返回给前端的错误描述也需要国际化，我们来看一下后端给前端返回多语言错误描述的实现方式有哪些。

常规实现
----

服务端将错误码和不同语言的错误描述硬编码在代码中，通过前端从 http head 中传过来的 language 来决定是返回中文还是英文。

### 1、定义 Error 结构体

该结构体实现标准库的 error 接口，实现自定义 error

```
type Error struct {
 Code int
 Msg  string
}

func (e *Error) Error() string {
 return fmt.Sprintf("%d,%s", e.Code, e.Msg)
}


```

### 2、定义错误码和错误描述 map

```
const (
 Err_Code_Success       = 0
 Err_Code_UnKnown       = -1
 Err_Code_InValid_Phone = 10001
)

const (
 Language_Chinese = 0 //中文
 Language_Enligh  = 1 //英文
)

//不同语言对应的错误描述
var errMap = map[int]map[int]string{
 Language_Chinese: {
  Err_Code_Success:       "成功",
  Err_Code_InValid_Phone: "手机号格式不正确",
  Err_Code_UnKnown:       "未知错误",
 },
 Language_Enligh: {
  Err_Code_Success:       "success",
  Err_Code_InValid_Phone: "invalid phone no",
  Err_Code_UnKnown:       "unknown err",
 },
}


```

### 3、申明一个用户注册的 api

根据客户端传过来的 http header 中的 language 的值决定返回中文还是英文的错误描述

```
func main() {
 http.HandleFunc("/user/register", func(w http.ResponseWriter, r *http.Request) {
  languageStr := r.Header.Get("language")
  language, _ := strconv.Atoi(languageStr)
  values, _ := url.ParseQuery(r.URL.RawQuery)
  phone := values["phone"][0]
  err := checkPhone(phone)
  response(w, language, err)
 })
 http.ListenAndServe(":8080", nil)
}

func response(w http.ResponseWriter, language int, err error) {
 e := &Error{Code: Err_Code_Success}
 if err != nil {
  var ok bool
  if e, ok = err.(*Error); !ok {
   e = &Error{Code: Err_Code_UnKnown}
  }
 }
 msg := errMap[language][e.Code]
 res := make(map[string]interface{})
 res["code"] = e.Code
 res["msg"] = msg
 json, _ := json.Marshal(res)
 w.WriteHeader(200)
 w.Write(json)
}

func checkPhone(phoneNo string) error {
 if len(phoneNo) != 11 {
  return &Error{Code: Err_Code_InValid_Phone}
 }
 return nil
}


```

我们通过 curl 命令来看看效果  

语言设置为中文时：

```
curl -H "language:0" "http://127.0.0.1:8080/user/register?phone=187111111112"
{"code":10001,"msg":"手机号格式不正确"}


```

语言设置为英文时：

```
curl -H "language:1" "http://127.0.0.1:8080/user/register?phone=187111111112"
{"code":10001,"msg":"invalid phone no"}


```

这种实现方式确实能满足业务需求，但是有下面几个缺点：

*   当要将手机号格式不正确的描述改时需要修改代码
    
*   当添加新的错误时需要改动多个地方代码：添加新的错误码和在 errMap 中添加对应语言的错误描述，容易遗漏
    
*   当添加新的语言时要向 errMap 添加所有错误码的新语言错误描述，容易遗漏
    

一旦涉及到修改代码就存在出现 bug 的风险。

有人会想到将错误描述放在 json 文件中维护，这种方案只是在修改错误描述时比较便利，不需要改动业务代码，但在新增错误和新语言时存在同样的问题。

还有一种方案是通过翻译包将文本翻译为目标语言，但这种方法翻译的结果不可控，且性能比较低。

有没有一种更优雅的方案，尽量减少修改代码，且有足够的灵活性？

下面我们来看看通过 go-error-generator 插件的方法来实现

更优雅的实现
------

go-error-generator 是一个通过 protobuf 文件的 Enum 对象自动生成 Error 的插件，通过在扩展的 EnumValueOptions 中定义多个 option 轻松实现 error 的多语言。

它包含如下功能：

*   根据 Enum 定义的 errCode 和 msg 自动生成 error;
    
*   支持定义多个 EnumValueOption，实现多语言;
    
*   支持 error 合并功能;
    
*   支持自定义 Error 结构体、error Code 和 Msg 的名称;
    

关于插件的原理和其他细节可以访问 github 主页了解。

我们回到刚才那个需求，用插件的方式怎么实现错误多语言

### 1、定义 error 模板

删除代码中的的 Error 结构体，取代的是在 protobuf 中定义，新建一个 protobuf 文件，取名为 error.proto，在这里自定义 error 结构体和语言标识。

其中：

*   msg: 默认的语言标识，在错误码定义文件中没有定义其他语言的错误描述时就用它的错误描述
    
*   msg_english: 英文标识，当然你也可以取别的名字
    

```
syntax = "proto3";
package errors;
option go_package = "github.com/classtorch/go-error-generator-examples/internal/errors";
import "google/protobuf/descriptor.proto";

message Error {
  int32 code = 1;
  string msg = 2;
};
extend google.protobuf.EnumValueOptions {
  string msg = 1108;
  string msg_english = 1109;
}


```

### 2、定义错误码和错误描述

新建一个 protobuf 文件，取名为 account.proto 导入上面定义好的 error.proto, 自定义 msg 和 msg_english 对应的错误描述

```
syntax = "proto3";
package uclass.service.account;
option go_package = "/golang/account";
import "errors/errors.proto";

enum ErrorCode {
  SUCCESS = 0 [(errors.msg) = "成功", (errors.msg_english) = "success"];  // 成功
  UnKnown = -1 [(errors.msg) = "未知错误", (errors.msg_english) = "unknown err"]; // 账号不存在
  InValid_Phone = 10001 [(errors.msg) = "手机号格式不正确", (errors.msg_english) = "invalid phone no"];  // 登录失效，请重新登录
}


```

### 3、通过插件生成代码

该插件需要安装 go 和 protobuf 运行环境

*   go
    
*   protoc
    
*   protoc-gen-go
    

安装好运行环境后再安装 go-error-generator 插件

```
go install github.com/classtorch/go-error-generator/protoc-gen-go-error-generator


```

安装好后执行下面脚本生成代码

```
protoc --go-error-generator_out=:. \
 --go-error-generator_opt descriptor_file=errors/errors.proto \
 --go-error-generator_opt merge_error=false \
 --go-error-generator_opt merge_error_path=golang/errors \
 --go_out=. -I . account.proto


```

插件自动生成的代码如下，包含 error 对象和 error map

```
var (
 SUCCESS       = &errors.Error{Code: 0, Msg: "成功"}           //成功
 UnKnown       = &errors.Error{Code: -1, Msg: "未知错误"}        //未知错误
 InValid_Phone = &errors.Error{Code: 10001, Msg: "手机号格式不正确"} //手机号格式不正确
)

var (
 Msg = map[int32]*errors.Error{
  0:     &errors.Error{Code: 0, Msg: "成功"},
  -1:    &errors.Error{Code: -1, Msg: "未知错误"},
  10001: &errors.Error{Code: 10001, Msg: "手机号格式不正确"},
 }
  
 Msg_English = map[int32]*errors.Error{
  0:     &errors.Error{Code: 0, Msg: "success"},
  -1:    &errors.Error{Code: -1, Msg: "unknown err"},
  10001: &errors.Error{Code: 10001, Msg: "invalid phone no"},
 }
)


```

### 4、使用生成的 error 对象

使用生成的 error 对象和 error map 改写 response 和 checkPhone 方法

```
func response(w http.ResponseWriter, language int, err error) {
 e := account.SUCCESS
 var ok bool
 if err != nil {
  if e, ok = err.(*errors.Error); !ok {
   e = account.UnKnown
  }
 }
 if language == Language_Chinese {
  if e, ok = account.Msg[e.Code]; !ok {
   e = account.UnKnown
  }
 } else if language == Language_Enligh {
  if e, ok = account.Msg_English[e.Code]; !ok {
   e = account.UnKnown
  }
 }
 res := make(map[string]interface{})
 res["code"] = e.Code
 res["msg"] = e.Msg
 json, _ := json.Marshal(res)
 w.WriteHeader(200)
 w.Write(json)
}

func checkPhone(phoneNo string) error {
 if len(phoneNo) != 11 {
  return account.InValid_Phone
 }
 return nil
}


```

完整的代码可以访问 go-error-generator-examples 项目进行了解

我们来看下这是实现方式的优点

*   当我们需要修改某个错误描述时直接在 account.proto 文件中修改，无须修改代码
    
*   当需要增加新的错误时直接在 account.proto 文件中定义，生成代码后直接在业务代码中引用即可
    
*   当添加新的语言时只需要在 error.proto 中增加新的语言标识即，然后在 account.proto 中引入即可
    

可以看出对于第一个和和第三个需求来说只需要修改 protobuf 文件，重新生成代码就可以，无须修改业务代码。第二个需求也只是简单的引入新的错误对象。

由于该插件是基于 protobuf 实现的，如果项目中没有使用 prorobuf 技术栈的话会带来一些引入成本。不过这点成本相对于频繁修改业务代码还是值得的。

相关链接
----

go-error-generator:  
https://github.com/classtorch/go-error-generator

go-error-generator-examples:  
https://github.com/classtorch/go-error-generator-examples