> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666563907&idx=2&sn=afcdc888d29911f2334e272711c3bf08&chksm=80dc4de8b7abc4fe83511f98803cf71907462eff03bd4debab38076ee55e94529dd331115e33&mpshare=1&scene=1&srcid=0702iBVI9ZMT94Y7hWA1OUi3&sharer_sharetime=1656703657514&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

 【导读】本文介绍了事件总线实现。

最近在学习开源项目`Grafana`的代码，发现作者实现了一个事件总线的机制，在项目里面大量应用，效果也非常好，代码也比较简单，介绍给大家看看。

> 源码文件地址：grafana/bus.go at main · grafana/grafana · GitHub

1. 注册和调用
--------

在这个项目里面随处可见这种写法：

关键是`bus.Dispatch(&query)`这段代码，它的参数是一个结构体`GetAlertByIdQuery`，内容如下：

```
type GetAlertByIdQuery struct {
    Id int64
    Result *Alert
}


```

根据名字可以看出这个方法就是通过 Id 去查询 Alert，其中`Alert`结构体就是结果对象，这里就不贴出来了。

通过查看源码可以得知，Dispatch 背后是调用了`GetAlertById`这个方法，然后把结果赋值到 query 参数的 Result 中返回。

```
func GetAlertById(query *models.GetAlertByIdQuery) error {
    alert := models.Alert{}
    has, err := x.ID(query.Id).Get(&alert)
    if !has {
        return fmt.Errorf("could not find alert")
    }
    if err != nil {
        return err
    }
    query.Result = &alert
    return nil
}


```

问题来了，这是怎么实现的呢？Dispatch 到底做了哪些操作？这样做有什么好处？

下面我来一一解答：

首先，在 Dispatch 之前，你需要先注册这个方法，也就是调用`AddHandler`，在这个项目里面可以看到 init 函数里面有大量这样的代码：

```
func init() {
    bus.AddHandler("sql", SaveAlerts)
    bus.AddHandler("sql", HandleAlertsQuery)
    bus.AddHandler("sql", GetAlertById)
    ...
}


```

其实这个方法的逻辑也很简单，所谓注册也就是把通过一个 map 把函数名和对应的函数做一个映射关系保存起来，当我们 Dispatch 的时候其实就是通过参数名查找之前注册过的函数，然后通过反射调用该函数。

Bus 结构体里面有几个 map 成员，在这个项目里面作者定义了 3 种不同类型的 handler，一种是普通的 handler，也就是刚才展示的那种，第二种是带上下文的 handler，还有一种则是事件订阅用到的 handler，我们给一个事件注册多个监听者，当事件触发的时候会依次调用多个监听函数，其实就是一个观察者模式。

```
// InProcBus defines the bus structure
type InProcBus struct {
    handlers        map[string]HandlerFunc
    handlersWithCtx map[string]HandlerFunc
    listeners       map[string][]HandlerFunc
    txMng           TransactionManager
}


```

下面就看看具体的源码，`AddHandler`方法内容如下：

```
func (b *InProcBus) AddHandler(handler HandlerFunc) {
    handlerType := reflect.TypeOf(handler)
    queryTypeName := handlerType.In(0).Elem().Name() // 获取函数第一个参数的名称，在上面例子里面就是GetAlertByIdQuery
    b.handlers[queryTypeName] = handler
}


```

Dispatch 方法的源码如下：

```
func (b *InProcBus) Dispatch(msg Msg) error {
    var msgName = reflect.TypeOf(msg).Elem().Name()

    withCtx := true
    handler := b.handlersWithCtx[msgName] // 根据参数名查找注册过的函数，先查找带Ctx的handler
    if handler == nil {
        withCtx = false
        handler = b.handlers[msgName]
        if handler == nil {
            return ErrHandlerNotFound
        }
    }
    var params = []reflect.Value{}
    if withCtx {
     // 如果查找到的handler是带Ctx的就给个默认的Background的Ctx
        params = append(params, reflect.ValueOf(context.Background()))
    }
    params = append(params, reflect.ValueOf(msg))

    ret := reflect.ValueOf(handler).Call(params) // 通过反射机制调用函数
    err := ret[0].Interface()
    if err == nil {
        return nil
    }
    return err.(error)
}


```

对于`AddHandlerCtx`和`DispatchCtx`这个 2 个方法基本上是一样的，只不过多了一个上下文参数，可以拿来做超时控制或者其它用途。

2. 订阅和发布
--------

除此之外，还有 2 个方法`AddEventListener`和`Publish`，即事件的订阅和发布。

```
func (b *InProcBus) AddEventListener(handler HandlerFunc) {
    handlerType := reflect.TypeOf(handler)
    eventName := handlerType.In(0).Elem().Name()
    _, exists := b.listeners[eventName]
    if !exists {
        b.listeners[eventName] = make([]HandlerFunc, 0)
    }
    b.listeners[eventName] = append(b.listeners[eventName], handler)
}


```

查看源码可以得知，可以给一个事件注册多个 handler 函数，而 Publish 的时候则是依次调用注册的函数，逻辑也不复杂。

```
func (b *InProcBus) Publish(msg Msg) error {
    var msgName = reflect.TypeOf(msg).Elem().Name()
    var listeners = b.listeners[msgName]

    var params = make([]reflect.Value, 1)
    params[0] = reflect.ValueOf(msg)

    for _, listenerHandler := range listeners {
        ret := reflect.ValueOf(listenerHandler).Call(params)
        e := ret[0].Interface()
        if e != nil {
            err, ok := e.(error)
            if ok {
                return err
            }
            return fmt.Errorf("expected listener to return an error, got '%T'", e)
        }
    }
    return nil
}


```

这里面有一点不好，所有订阅函数的调用是顺序的，并没有使用协程，所以如果注册了很多个函数，这样效率也不高啊。

3. 好处
-----

可能有人会好奇，为什么明明可以直接调用函数就行，为啥非得绕个弯子，整这么复杂？

况且，每次调用都得使用反射机制，性能也不行。

我觉得主要有以下几点：

1. 这种写法逻辑清晰，解耦

2. 方便单元测试

3. 性能不是最大考量，虽然说反射会降低性能

> 转自：
> 
> wangbjun.site/2021/coding/golang/event-bus.html

- EOF -