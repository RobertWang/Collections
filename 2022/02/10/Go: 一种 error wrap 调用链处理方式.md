> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CVP_7LjKqcwn6ivFXXisLg)

【导读】go API 返回错误有什么好实践？本文做了详细介绍。

背景
--

在做一套内部公共服务, 提供后台 API 调用的时候, 某些情况下, 会返回 500 错误, 此时有两种做法

1.  在后台记录详细的 500 日志, 返回给调用方一个`request_id`, 调用方拿 request_id 查询日志定位问题
    
2.  直接在 response body 中返回错误信息
    

选择了问题排查路径短一些的 `2`, 错误信息中, 需要包含调用链路以及根因错误的详细信息, 提升问题排查的效率

这样, 开发者在看到报错 response body 的同时, 可以直接知道调用链, 不需要使用请求重新复现问题, 节约了大量的复现 / 沟通的时间

最终效果
----

```
system error[request_id=0838c4090cfa4f4f9dceea5fd8c7b029]: [Handler:validateSystemSuperUser] impls.ListSubjectRoleSystemID subjectType=`user`, subjectID=`test1` fail%!(EXTRA string=test1) => [Cache:GetSubjectRole] SubjectRoleCache.Get subjectType=`user`, subjectID=`test1` fail => [Cache:GetSubjectPK] SubjectPKCache.Get _type=`user`, id=`test1` fail => [SubjectSVC:GetPK] GetPK _type=`user`, id=`test1` fail => [Raw:Error] sql: no rows in result set


```

处理机制
----

使用 error wrap, 一层层抛出

### 1. 定义 pkg/errorx

注意, package name 使用`errorx`, 可以规避同标准库命名的冲突; 同时 import 的时候不需要 alias

```
type Error struct {
    message string
    err     error
}

func (e Error) Error() string {
    return e.message
}

func (e Error) Is(target error) bool {
    if target == nil || e.err == nil {
        return e.err == target
    }

    return errors.Is(e.err, target)
}

func (e *Error) Unwrap() error {
    u, ok := e.err.(interface {
        Unwrap() error
    })
    if !ok {
        return e.err
    }

    return u.Unwrap()
}


```

### 2. 封装 wrapfunc helper

注意,

*   这里如果是最原始的 err(即 wrap 前的 err), 会使用`[Raw:Error]`标注
    
*   如果是中间层的, 会附加`layer`(哪一层) 和`function`(哪个函数) 这两个信息
    
*   这里定义了一个 helper 方法, 用于在
    

```
func makeMessage(err error, layer, function, msg string) string {
    var message string
    var e Error
    if errors.As(err, &e) {
        message = fmt.Sprintf("[%s:%s] %s => %s", layer, function, msg, err.Error())
    } else {
        message = fmt.Sprintf("[%s:%s] %s => [Raw:Error] %v", layer, function, msg, err.Error())
    }

    return message
}

func Wrapf(err error, layer string, function string, format string, args ...interface{}) error {
    if err == nil {
        return nil
    }

    msg := fmt.Sprintf(format, args...)

    return Error{
        message: makeMessage(err, layer, function, msg),
        err:     err,
    }
}

type WrapfFuncWithLayerFunction func(err error, format string, args ...interface{}) error

func NewLayerFunctionErrorWrapf(layer string, function string) WrapfFuncWithLayerFunction {
    return func(err error, format string, args ...interface{}) error {
        return Wrapf(err, layer, function, format, args...)
    }
}


```

### 3. 使用

在某个函数体内, 涉及到需要 return err 或中间层

```
errorWrapf := errorx.NewLayerFunctionErrorWrapf("Handler", "ListSubject")

_, err := svc.ListPaging(body.Type, body.Limit, body.Offset)
if err != nil {
    err = errorWrapf(err, "svc.ListPaging type=`%s` limit=`%d` offset=`%d` fail!",
            body.Type, body.Limit, body.Offset)
        return err
}


```

注意, **wrap 时包含关键的调用参数 / 返回值等等, 但是不能包含敏感数据**

> 转自：
> 
> wklken.me/posts/2021/01/26/golang-error-wrap.html

 - EOF -