> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651601526&idx=1&sn=f5f429fd71f95817f69a100cbffc61cc&chksm=8022edb7b75564a18aa0698dcb564e92030cd01ea7e00b6cafdb72238648ac37fcbfccedf5e0&mpshare=1&scene=1&srcid=0405Eh6quM29nCDiVp5tQLyN&sharer_sharetime=1649144783417&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

TypeScript 类型系统支持类型编程，也就是对类型参数做一系列运算产生新的类型。比如这样：

```
type isTwo<T> = T extends 2 ? true: false;
```

这种类型编程逻辑可以写的很复杂，所以被戏称为 “类型体操”。

它是 TS 中最强大也是最复杂的部分了，属于深水区的内容。

很多同学不知道类型编程学了有什么用，好像做业务也用不到这个。那今天我们就来看一个具体的例子，来感受下类型体操的意义。

我们想实现这样一个 JS 方法：

```
function parseQueryString(queryStr) {
    if (!queryStr || !queryStr.length) {
        return {};
    }
    const queryObj = {};
    const items = queryStr.split('&');
    items.forEach(item => {
        const [key, value] = item.split('=');
        if (queryObj[key]) {
            if(Array.isArray(queryObj[key])) {
                queryObj[key].push(value);
            } else {
                queryObj[key] = [queryObj[key], value]
            }
        } else {
            queryObj[key] = value;
        }
    });
    return queryObj;
}
```

这段代码很容易看出来就是做 query string 的 parse 的，会把'a=1&b=2&c=3' 的字符串 parse 成 {a: 1, b: 2, c: 3} 返回。如果有同名的 key 的话，就合并到一个数组里。

JS 的逻辑大家写的比较多，这部分很容易理解：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjh0K0sf8xR3eIaSOEGf6MjRia9tbvIlia0pPqMn9TrWaH0pCO0Cp304hg/640?wx_fmt=png)

那如果要给这个函数加上类型，大家会怎么加呢？

我猜，大部分人会这么加：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjCUdKgBujAkykLaSArhPjd35tuC2LYRV4I2QjxbpeExvJsbeTFOCfPA/640?wx_fmt=png)

参数是 string 类型，返回值是 parse 之后的对象类型 object。

这样是可以的，而且 object 还可以写成 Record<string, any>，因为对象是索引类型（索引类型就是聚合多个元素的类型，比如对象、class、数组都是）。

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjTiaicrETUO16wIX8X1S1Jz92FRS9eM0qB4XMarIevic5nvor450fiaLgcw/640?wx_fmt=png)Record 是 TS 内置的一个高级类型，是通过映射类型的语法来生成索引类型的：

```
type Record<K extends string | number | symbol, T> = { 
    [P in K]: T;
}
```

比如传入'a' | 'b' 作为 key，1 作为 value，就可以生成这样索引类型：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjVWXRPs6vloeaI6txrfUQWY8ha30c2O3uTvYu5DDx06tCVx8eQEH3Jw/640?wx_fmt=png)所以这里的 Record<string, any> 也就是 key 为 string 类型，value 为任意类型的索引类型，可以代替 object 来用，更加语义化一点：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjBnLDwcicF0eFAYTibwjYD5XFQfGdFouhWiceCD0r1JxAfaicZibzn25oibEg/640?wx_fmt=png)

但是不管是返回值类型为 object 还是 Record<string, any> 都存在一个问题：返回的对象不能提示出有哪些属性：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjpAzMsX3WtAQtsicke6dhtfb44z6kSmRgEsFp1VQopSDXbTjPic7SUVHA/640?wx_fmt=png)

对于习惯了 ts 的提示的同学来说，没有提示太不爽了。怎么能让这个函数的返回的类型有提示呢？

这就要用到类型编程了。

我们把函数的类型定义改成这样：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjjT88GiasYNicKcGqSqD9vVKP6jZg6kDMSqFh1y20MUh1zIlFHQOwm44w/640?wx_fmt=png)声明一个类型参数 Str，约束为 string 类型，函数参数的类型指定是这个 Str，返回值的类型通过对 Str 做类型运算得到，也就是 ParseQueryString<Str>。

这个 ParseQueryString 的类型做的事情就是把传入的 Str 通过各种类型运算产生对应的索引类型。

这样返回的类型就有提示了：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjXhfXUSicia0T2y9HYmicXJu6icI8j3owuFHcIWlpSRHhOiamhjFxcP7Iz8g/640?wx_fmt=png)

是不是很神奇！这就是类型体操的魅力！能够实现更精准的类型提示。

那这个 ParseQueryString 的高级类型是怎么实现的呢？

首先我们要把'a=1&b=2&c=3' 的字符串按照 & 分割开，使用模式匹配的方式。

把提取出来的每一个 a=1、b=2、c=3 这种字符串再做一次处理，把结果合并起来返回。

也就是：

```
type ParseQueryString<Str extends string>
    = Str extends `${infer Param}&${infer Rest}`
        ? MergeParams<ParseParam<Param>, ParseQueryString<Rest>>
        : ParseParam<Str>;
```

类型参数 Str 为待处理的字符串类型，通过模式匹配的方式提取 & 分割的字符串到 infer 声明的局部变量 Param 中，剩下的放到 infer 声明的局部变量 Rest 中。

对提取出来的 Param 再做处理，也就是 ParseParam<Param>，剩下的递归处理，也就是 ParseQueryString<Rest>，然后把结果合并。

如果模式匹配不满足，就说明没有 & 了，那就把剩下的也做一次处理返回。

这里的 ParseParam 就是处理 a=1、b=2、c=3 这种字符串的，也是通过模式匹配来提取：

```
type ParseParam<Param extends string> = 
    Param extends `${infer Key}=${infer Value}`
        ? {
            [K in Key]: Value 
        } : Record<string, any>;
```

类型参数 Param 是待处理的字符串，通过模式匹配提取 = 分隔的字符串到局部变量 Key 和 Value 中，构造成索引类型返回，

如果模式匹配不满足，说明不是 = 分隔的字符串字面量类型，就返回 Record<string, any> 代表任意索引类型。

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjllFTmSn8w88ZAiajbfDiaowKK7MJANmnbg2pica5CjGUSGwAuLZiclsWww/640?wx_fmt=png)

测试下：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjyYLSjMQWQUibAdCLY99b51yTdlbOan4icq58IKsQMHgRSfrEg899EvLg/640?wx_fmt=png)

然后对多个索引类型的合并，就是通过映射类型的语法构造一个新的索引类型：

```
type MergeParams<
    OneParam extends Record<string, any>,
    OtherParam extends Record<string, any>
> = {
  readonly [Key in keyof OneParam | keyof OtherParam]: 
    Key extends keyof OneParam
        ? Key extends keyof OtherParam
            ? MergeValues<OneParam[Key], OtherParam[Key]>
            : OneParam[Key]
        : Key extends keyof OtherParam 
            ? OtherParam[Key] 
            : never
}
```

类型参数 OneParam 和 OtherParam 是两个索引类型，通过 Record<string,any> 来约束。

通过映射类型的语法构造一个新的索引类型返回，Key 来自两者的合并，也就是 Key in keyof OneParam | keyof OtherParam。并且加上一个 readonly 的修饰，这样就是让返回的索引类型不能被修改。

值要判断下如果是两者都有的值，那就做合并，否则分别取对应的值。

合并的逻辑是这样的：

```
type MergeValues<One, Other> = 
    One extends Other 
        ? One
        : Other extends unknown[]
            ? [One, ...Other]
            : [One, Other];
```

类型参数 One、Other 为待合并的两个值的类型，如果两个一样就返回其中一个，否则如果是数组就合并数组，也就是 [One, ...Other]，否则把两个值合并成数组 [One, Other]。

这样就完成了两个索引类型的合并，测试下：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOj83hx99FQep4Z8f8XO0FcC0TyVEfTnf1xXStcYWLrBnUUs7PyC0bLZA/640?wx_fmt=png)

整体测试下：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjat9fbU7kOE4r9N8x18TY5JadC3YZquf03V0SKHWumE2J466DClP8Lg/640?wx_fmt=png)

成功了！我们实现了 ParseQueryString 的高级类型！

当然，这只是纯粹的类型体操，把它用到 JS 里才是最终目的，所以我们把 parseQueryString 的类型定义改成了这样：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjdjgtE93lvjHaxt8wqyJialiaVqehSHKWn2bjO9ibcgmK3jKuqab4WclHg/640?wx_fmt=png)

把函数参数的类型传入 ParseQueryString 的高级类型做类型运算，返回的结果作为函数返回值的类型。（这里要用 as any 把返回值断言为 any，因为默认推导的类型不精确，我们用根据 Str 动态算出来的类型）

这样也就能实现精准的类型提示：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjib6KqltSWrGNSj40PAzblLibokATEZuRSZkuXZ95p6NdJibgXb3SUkNEA/640?wx_fmt=png)

并且因为我们 readonly 的限制，不能修改属性的值：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOjOMDPqygQt5aVASqnDuZ2KGZDIGmaTc1Mc4nMC02sGyicGuZlmHLEG5w/640?wx_fmt=png)

对比下没用类型体操的时候：

![](https://mmbiz.qpic.cn/mmbiz_png/YprkEU0TtGhbgOzJOtZhcVDh2McCzYOj90rGklz9ylIqQhjttzwCejd77oSLkG5xBrB8fzCkdWCTTRoPXPJRPQ/640?wx_fmt=png)

就可以得出结论：

**类型编程可以通过类型运算产生更准确的类型，配合编辑器可以做更精准的类型提示和检查，这就是类型体操的意义。**

总结
--

类型编程是 TypeScript 的深水区内容，它是对类型做一系列类型运算后产生新的类型，它可以实现更精准的类型提示和检查。

我们通过 parseQueryString 这个函数的类型定义来直观感受了下用类型体操和不用类型体操的区别，在类型提示这方面，体验是相差很多的。

实现更精准的类型提示和检查，这就是类型体操的意义！

- EOF -

推荐阅读  点击标题可跳转

1、[TypeScript 中的泛型你真搞懂了吗？](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651595811&idx=2&sn=92b55478e9646d68dc4cbf988fd2b8cd&chksm=8022f3e2b7557af434f362b9991ba8a4d9e5fc712bb0f33c116aebce7d0937a25e50072047eb&scene=21#wechat_redirect)

2、[TypeScript 高级类型及用法](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651585160&idx=3&sn=f8ad3187e14ef0b0cbc4defe80680bff&chksm=80252d49b752a45fe916966995169de203b29c6b0a0c115659f32c7216111e8a8b58c0b347cf&scene=21#wechat_redirect)

3、[2021 typescript 史上最强学习入门文章 (2w 字)](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651590390&idx=2&sn=3c71df7f89e9e0d00e7a96c7e40210c1&chksm=8022d937b755502156d41952ff60f280185dd862f84604edebfa13ce0dcbcc3a6023fb1ee344&scene=21#wechat_redirect)