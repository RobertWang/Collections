> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/D0i5uUVXo7qPkKaFoP7cog)

> 查看 Hacker News[1] 上的讨论

TypeScript 的 `never` 类型被讨论得非常少，因为它不像其他类型那样常用，或者不可替代。对于 TypeScript 的初学者来说，`never` 类型很容易被忽略，因为它只会出现在处理高级类型（比如条件类型）时，或者阅读那些神秘兮兮的类型错误信息时。

实际上 `never` 类型在 TypeScript 中的优秀用例比想象中要多。当然，它也有一些特有的你需要小心的陷阱。

本文的主要内容包含以下几个部分：

*   `never` 类型的意义和我们需要它的原因。
    
*   `never` 的应用场景以及需要小心避开的坑。
    

`never` 类型的定义
-------------

在充分理解 `never` 类型和它的设计目之前，我们需要先理解什么是**类型**，以及 `never` 在类型系统中扮演的角色。

一个**类型**就是一种值的集合。例如：`string` 类型表达的是任意字符串的无限集。因此，当我们将一个变量注释为 `string` 类型时，那么它的取值只能是这个集合中的值，也就是任意字符串：

```
let foo: string = 'foo'
foo = 3 // ❌ 数字不在字符串集合内


```

在 TypeScript 中，`never` 是值集为空的集合。事实上，在另一种流行的 JavaScript 类型系统 Flow[2] 中，相同的类型被叫做 empty[3]。

因为集合里面没有值，所以 `never` 类型就不能被赋值，包括 `any` 类型的值（这听起来很绕）。也就是说 `never` 类型代表永远不会发生的类型 [4]，或者换句话说是一个底层类型 [5] 的概念。

```
decalre const any: any
const never: never = any // ❌ 'any' 类型的值不能赋值给 'never' 类型的变量


```

“底层类型” 是 TypeScript 手册中 [6] 对其的定义方式。我发现当我们把它放在类型层次树 [7] 中时，它更有意义，这是我用来理解子类型 [8] 的思想模型。

下一个逻辑问题是，为什么我们需要 `never` 类型呢？

我们为什么需要 `never` 类型
------------------

正如我们在数字系统中需要 **0** 来表述什么都没有一样，我们的系统中也需要一个类型用来表述**不可能**。

"不可能" 这个词本身是一种模糊的表述。在 TypeScript 中，"不可能" 表现出多种含义，即：

*   一个不能有任何值的空类型，它可以用来表示：
    

*   泛型和函数中不允许的参数
    
*   互斥的交叉类型
    
*   一个空的联合类型（“什么都没有” 的联合类型）
    

*   一个函数的返回值——当该函数执行完毕后，不会返回调用进程（例如：node 中的 `process.exit`）
    

*   不要将其和 `void` 搞混，`void` 的意思是函数返回调用进程时值为空。
    

*   一个在条件类型中永远不会进入的 else 分支
    
*   一个在 `promise` 中 reject 分支中返回值的类型:
    

```
const p = Promise.reject('foo') // const p: Promise<never>


```

### `never` 在联合类型和交叉类型中的作用

类似于数字 0 在加法和乘法中的作用，`never` 类型在联合类型和交叉类型中使用时具有特殊的意义：

*   `never` 在联合类型中不起作用，类似于 0 在加法运算中没有意义一样：
    

*   `type Res = never | string // string`
    

*   `never` 在交叉类型中会覆盖其他类型，类似于 0 在乘法中会使结果为 0 一样：
    

*   `type Res = never & string // never`
    

`never` 类型的这两个行为 / 特征为它的一些最重要的用例奠定了基础，我们将在后面看到。

如何使用 `never` 类型
---------------

由于我们永远不能给 `never` 类型赋值，所以我们可以用它来对各种函数用例施加限制。

### 确保对 `switch` 和 `if-else` 语句中的所有条件都做处理

如果一个函数只能接受一个 `never` 的参数，那么这个函数就永远不能用任何非 `never` 的值来调用（不用 TypeScript 编译器对我们发出警报）。

```
function fn(input: never) {}

// 只允许 `never` 类型参数 
declare let myNever: never
fn(myNever) // ✅

// 传其他类型的参数（或者不传）都会引起类型错误：
fn() // ❌  An argument for 'input' was not provided.
fn(1) // ❌ Argument of type 'number' is not assignable to parameter of type 'never'.
fn('foo') // ❌ Argument of type 'string' is not assignable to parameter of type 'never'.

// 哪怕参数是 `any` 类型也不可以
declare let myAny: any
fn(myAny) // ❌ Argument of type 'any' is not assignable to parameter of type 'never'.


```

我们可以用这类函数来确保 `switch` 和 `if-else` 语句中，每个条件都覆盖了处理方法：将其放在 `default` 条件中，我们可以确保每个条件都被处理，否则取值必须是 `never` 类型。如果我们不小心漏掉了一个可能的条件，我们会得到一个类型错误。如下：

```
function unknownColor(x: never): never {
    throw new Error("unknown color");
}


type Color = 'red' | 'green' | 'blue'

function getColorName(c: Color): string {
    switch(c) {
        case 'red':
            return 'is red';
        case 'green':
            return 'is green';
        default:
            return unknownColor(c); // Argument of type 'string' is not assignable to parameter of type 'never'
    }
}


```

### 禁用结构化类型中的一部分

假设我们有一个函数，它接受一个 `VariantA` 类型或 `VariantB` 类型的参数。但是，不能接受一个同时包含两种类型所有属性的类型，即两种类型的一个子类型 [9]。

我们可以利用一个联合类型 `VariantA | VariantB` 来作为参数。然而，由于 TypeScript 中的类型兼容性是基于结构子类型 [10] 的，所以允许向函数传递一个属性多于参数类型的对象类型（除非你传递对象字面量）。

```
type VariantA = {
    a: string,
}

type VariantB = {
    b: number,
}

declare function fn(arg: VariantA | VariantB): void


const input = {a: 'foo', b: 123 }
fn(input) // 这违背了我们的设计，但是 TypeScript 不会报警


```

以上的代码片段中，TypeScript 不会给出类型错误。

但使用 `never` 后，我们就可以将类型结构中的部分给禁用掉，从而阻止用户向其传递包含两种类型属性的对象：

```
type VariantA = {
    a: string
    b?: never
}

type VariantB = {
    b: number
    a?: never
}

declare function fn(arg: VariantA | VariantB): void


const input = {a: 'foo', b: 123 }
fn(input) // ❌ Types of property 'a' are incompatible


```

### 防止意外的 API 使用

让我们假设我们需要编写一个缓存实例，用于存储和读取数据：

```
type Read = {}
type Write = {}
declare const toWrite: Write

declare class MyCache<T, R> {
  put(val: T): boolean;
  get(): R;
}

const cache = new MyCache<Write, Read>()
cache.put(toWrite) // ✅ 允许


```

现在，由于一些原因我们呢需要将其改为只读，也就是只允许 `get` 方法从中读取数据。此时我们可以将 `put` 方法的参数设置为 `never` 类型，这样它就不允许任何类型的值传入：

```
declare class ReadOnlyCache<R> extends MyCache<never, R> {} // 此时 'MyCache' 的参数 'T' 类型变为 'never'

const readonlyCache = new ReadOnlyCache<Read>()
readonlyCache.put(data) // ❌ 参数是 'never' 类型，不能传入 'Data' 类型的值


```

> 需要提醒一下，这可能不是派生类的很好用例，与'never' 类型本身无关。我不是面向对象编程的专家，所以仅供参考。

### 用于表示理论上无法到达的条件分支

当我们在条件类型中使用 `infer` 创建一个类型变量时，我们必须为每个 `infer` 关键字创建 else 分支：

```
type A = 'foo';
type B = A extends infer C ? (  
    C extends 'foo'? true : false // 在此表达式中，C 等同于 A
) : never // 这个分支永远不会执行，但是我们也不能不写它


```

为什么 `extends infer` 非常有用？

在我之前的文章中，我提到了如何将声明 "local (type) variable" 与 `extends infer` 联系在一起。你可以参考这篇 [11]。

### 在联合类型中做过滤

除了用于表示不可能的分支，`never` 也可以用于在条件类型中做过滤。

正如我们之前讨论过的那样，当用于联合类型时，`never` 类型会自动删除。换句话说，在联合类型中，`never` 类型没有用处。

当我们编写工具类用于根据某些标准选择来自联合类型的某些成员时，`never` 类型的 “无用” 性恰恰成为最适合放在 else 分支的类型。

假设我们有一个工具类 `ExtractTypeByName`，用于在联合类型中找出'name' 属性为'foo' 的类型成员，并将其他的成员过滤掉：

```
type Foo = {  
    name: 'foo'  
    id: number
}

type Bar = {   
    name: 'bar'  
    id: number
}

type All = Foo | Bar

type ExtractTypeByName<T, G> = T extends {name: G} ? T : never

type ExtractedType = ExtractTypeByName<All, 'foo'>


```

让我们看看它具体是如何工作的：

以下是 Typescript 如何一步一步得到类型结果的:

*   条件类型首先分发成联合类型：
    

```
type ExtractedType = ExtractTypeByName<All, Name
⬇️                    
type ExtractedType = ExtractTypeByName<Foo | Bar, 'foo'>
⬇️    
type ExtractedType = ExtractTypeByName<Foo, 'foo'   | ExtractTypeByName<Bar, 'foo'>


```

*   将类型实现和赋值拆分：
    

```
type ExtractedType = Foo extends {name: 'foo'} ? Foo : never 
                    | Bar extends {name: 'foo'} ? Bar : never
⬇️
type ExtractedType = Foo | never


```

*   将'never' 从联合类型中移除
    

```
type ExtractedType = Foo | never
⬇️
type ExtractedType = Foo


```

从映射类型中过滤属性
----------

在 TypeScript 中，类型是不可变的。如果想要从一个对象类型中删除一个属性，我们只能新建一个类型，通过转换和过滤达到这个目的。而我们只要在映射类型中用条件做重映射 [12] 就可以达到相同的效果。

以下 `Filter` 类型，是基于对象类型的值对对象类型进行筛选的例子。

```
type Filter<Obj extends Object, ValueType> = {  
    [Key in keyof Obj     
        as ValueType extends Obj[Key] ? Key : never]    
        : Obj[Key]
}


interface Foo { 
    name: string; 
    id: number;
}

type Filtered = Filter<Foo, string>; // {name: string;}


```

在控制流分析中收窄类型范围
-------------

当我们把一个函数的返回值类型设为 `never` 时，就意味着该函数永远不会将控制权返回给调用者。我们可以利用它来帮助控制流分析来收窄类型范围。

> 函数调用可能有以下几个原因导致无法返回: 在所有的代码路径上抛出异常，进入死循环，或者退出程序，例如 Node 中的 `process.exit`

下面的代码片段中，我们令一个函数返回 `never` 类型，用于从一个联合类型 `foo` 中剔除 `undefined` :

```
function throwError(): never {  
    throw new Error();
}

let foo: string | undefined;

if (!foo) { 
    throwError();
}

foo; // string


```

也可以在 `||` 或 `??` 操作符后调用 `throwError` :

```
let foo: string | undefined;
const guaranteedFoo = foo ?? throwError(); // string


```

### 表示不兼容类型的交叉类型

这一点感觉上更像是 TypeScript 语言的行为特征，而不是一个 `never` 类型的用例。然而，这对于理解一些神秘的错误消息是至关重要的。

任何不兼容的交叉类型都是 `never` 类型

```
type Res = number & string // never


```

同时，任何类型与 `never` 类型的交叉类型也是 `never` 类型

```
type Res = number & never // never


```

对于对象类型，情况会有些复杂...

在交叉对象类型时，根据属性的类型是否为可辨别属性（字面量类型或字面量类型的联合类型），可能会也可能不会将整个类型简化为 `never` 类型

此例中，只有 `name` 属性会推导为 `never` 类型，因为 `string` 和 `number` 不是可辨别属性

```
type Foo = {
    name: string,
    age: number
    }
    type Bar = {   
        name: number,   
        age: number 
    } 
      
    type Baz = Foo & Bar // {name: never, age: number}  


```

而在下面这个例子中，整个 `Baz` 类型会推导为 `never` 类型，因为 `boolean` 类型是可辨别属性（类型 `boolean` 就是 `true | false` 的联合类型）

```
type Foo = {
    name: boolean,
    age: number
    }

    type Bar = {   
        name: number,    
        age: number 
    }
    
    type Baz = Foo & Bar // never


```

通过这个 PR[13] 来了解更多。

如何读懂 never 类型（的错误信息）
--------------------

您可能在没有显式声明 `never` 类型的代码中意外的获得 `never` 类型的错误消息。这通常是因为 TypeScript 编译器交叉了这些类型。之所以隐式地这样做，是为了保证类型安全以及代码稳健。

接下来的例子（在 TypeScript playground[14]）我在之前的博文 [15] 中曾提到过的多态函数的类型：

```
type ReturnTypeByInputType = { 
  int: number 
  char: string 
  bool: boolean
}

function getRandom<T extends 'char' | 'int' | 'bool'>( 
  str: T
): ReturnTypeByInputType[T] { 
  if (str === 'int') {  
    // 生成一个随机数 
    return Math.floor(Math.random() * 10) // ❌ Type 'number' is not assignable to type 'never'. 
  } else if (str === 'char') { 
    // 生成一个随机字符 
    return String.fromCharCode(   
      97 + Math.floor(Math.random() * 26) // ❌ Type 'string' is not assignable to type 'never'.  
    ) 
  } else {  
    // 生成一个随机布尔值  
    return Boolean(Math.round(Math.random())) // ❌ Type 'boolean' is not assignable to type 'never'.
  }
}


```

该函数设计目的是通过参数类型的不同返回数字、字符串或布尔值。我们使用泛型索引访问 `ReturnTypeByInputType[T]` 来推导相应的返回类型。

但是，每个返回分支我们都会得到一个类型错误：

```
Type X is not assignable to type 'never' // 'X' 是 number, string 或 boolean


```

这是 TypeScript 尝试帮助我们缩小程序中可能出问题的范围：每一个返回值应该分配到类型 `ReturnTypeByInputType[T]` (例子中注释说明的) 在运行时推导出的结果—— `number`, `string` 或者 `boolean` 类型。

只有在返回值的类型满足 `ReturnTypeByInputType[T]` 推导类型的所有可能性，该类型才被认为是安全的。包括 `number`, `string` 和 `boolean` 的**交叉类型**。那么，这三种类型的交叉类型是什么呢？当然是 `never` ——因为他们互不兼容。这就是为什么我们得到了 `never` 的错误信息。

要解决这个问题，你必须使用类型断言（或函数重载）：

*   `return Math.floor(Math.random() * 10) as ReturnTypeByInputType[T]`
    
*   `return Math.floor(Math.random() * 10) as never`
    

另一个显而易见的例子：

```
function f1(obj: { a: number, b: string }, key: 'a' | 'b') {   
    obj[key] = 1;    // Type 'number' is not assignable to type 'never'. 
    obj[key] = 'x';  // Type 'string' is not assignable to type 'never'.
}


```

`obj[key]` 的推导结果是 `string` 还是 `number` 取决于运行时的 `key`。因此，TypeScript 加上了这个限制——我们写入 `obj[key]` 的任何值必须兼容 `string` 和 `number` 才是安全的。于是，这两个类型的交叉，我们就得到了 `never`。

如何检查类型推导是否为 never
-----------------

检查一个类型是否会推导为 `never` 比想象中要难得多。

思考以下代码：

```
type IsNever<T> = T extends never ? true : false
type Res = IsNever<never> // never 🧐


```

结果是 `true` 还是 `false` ? 结果可能会让你感到困惑——二者都不是，而是 `never`。

事实上，当我第一次看到这个的时候，我也糊涂了。根据 Ryan Cavanaugh[16] 在这个 issue[17] 中的解释，原因可以总结为：

*   TypeScript 会自动将联合类型分发为条件类型
    
*   `never` 是一个空联合类型
    
*   因此，当分发发生时，缺没有内容可分发，所以条件类型再次将其推导为 `never`
    

唯一的解决方法是不使用隐式分发，而是将类型参数封装在一个元组中:

```
type IsNever<T> = [T] extends [never] ? true : false;
type Res1 = IsNever<never> // 'true' ✅
type Res2 = IsNever<number> // 'false' ✅


```

这实际上是从 TypeScript 源代码 [18] 中直接得到的，如果 TypeScript 能够将其暴露出来就更好了。

总结
--

本文中我们聊了很多：

*   首先，我们讨论了 `never` 类型的定义和设计目的。
    
*   然后，我们讨论了它的各种用例：
    

*   利用 `never` 类型为空类型的特性，对函数施加限制
    
*   从联合类型中过滤掉不需要的成员或从对象类型中过滤不需要的属性
    
*   辅助控制流程分析
    
*   表示无效或者不可达的条件分支
    

*   我们之后又讨论了为什么会得到意外的 `never` 类型推导——由于隐式的类型交叉
    
*   最后，我们还讨论了如何去检查一个类型是否为 `never`
    

> 特别感谢我的好友 Josh[19] 审阅了这篇文章并给予了宝贵的意见!

### 参考资料

[1]

Hacker News: _https://news.ycombinator.com/item?id=30616912_

[2]

Flow: _https://flow.org/_

[3]

empty: _https://github.com/facebook/flow/commit/c603505583993aa953904005f91c350f4b65d6bd_

[4]

永远不会发生的类型: _https://cs.stackexchange.com/questions/134215/what-is-an-uninhabited-type_

[5]

底层类型: _https://en.wikipedia.org/wiki/Bottom_type_

[6]

TypeScript 手册中: _https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html#other-important-typescript-types_

[7]

类型层次树: _https://www.zhenghao.io/posts/type-hierarchy-tree#the-bottom-of-the-tree_

[8]

子类型: _https://en.wikipedia.org/wiki/Subtyping_

[9]

子类型: _https://en.wikipedia.org/wiki/Subtyping_

[10]

结构子类型: _https://en.wikipedia.org/wiki/Subtyping_

[11]

这篇: _https://www.zhenghao.io/posts/type-programming#local-variable-declaration_

[12]

重映射: _https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#key-remapping-via-as_

[13]

PR: _https://github.com/microsoft/TypeScript/pull/36696_

[14]

TypeScript playground: _https://www.typescriptlang.org/play/?#code/C4TwDgpgBAShwFcBOA7AKuCAhEBJFYCwGkUAvFAN4BQUUAlisAFxQoIC2ARhErVAGMAFgEMkrAM7AkjAOb8uAe0UAbVktUQRKagF9q1AGYIUA4PUUoos+DG0ATRRwA8aKBAAewCCnsSoAOTCYgFQAD6BjMChEQEaKgEAfAAU-FLiUGjUAJSscIioJNh4BERFANpoALpU-PSGUMnp5GQUAVEB2bV0dAD0vdY+vCLeUCJQSA5ObJw8fD0T8MhWALIjQgB0hirKSMlrwJuTvk7JXQBUUACMAAxd-LruKhLQ9Y3NrW3BSJ3dPf2DFDDUbjY6ODiCUTzHpIJaoKAAZWkci2SCcAGEoejFPYIKkFnQAJwAdigAGooAdNttdvt1hswacLlAAEwANmy-DonLojwgz2gNAWAJsQMmIImUwh8S0OgWsIKViwyhUsrphwZihM9nVRylZ2yPKg+l0QA_

[15]

博文: _https://www.zhenghao.io/posts/type-functions_

[16]

Ryan Cavanaugh: _https://twitter.com/searyanc_

[17]

issue: _https://github.com/microsoft/TypeScript/issues/23182#issuecomment-379094672_

[18]

TypeScript 源代码: _https://github.com/microsoft/TypeScript/blob/main/tests/cases/conformance/types/conditional/conditionalTypes1.ts#L212_

[19]

Josh: _https://twitter.com/JoshuaKGoldberg_

[20]

参考原文: _https://www.zhenghao.io/posts/ts-never_

- EOF -

推荐阅读  点击标题可跳转

1、[真实案例说明 TypeScript 类型体操的意义](http://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247490859&idx=1&sn=f07393bec2f1ad2eb9d4158cc5c07373&chksm=90afa9c5a7d820d353155daa9b684f56c1817f27b0dd79a488ac3b4bfea8a4d3faf9c8a0b13e&scene=21#wechat_redirect)

2、[Vue3 拥抱 TypeScript 的完整项目结构搭建](http://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247490638&idx=1&sn=2759fc935bbc52228e8b4fcd210fc305&chksm=90afa8a0a7d821b60ae4cc36b00bdca17e6114f985c88f2847ff6882e9855bd765c8eae63fd1&scene=21#wechat_redirect)

3、[Typescript 史上最强学习入门文章 (2w 字)](http://mp.weixin.qq.com/s?__biz=MzA5NjUxMTM2MQ==&mid=2247490275&idx=1&sn=fc0d656e0ac0ab08ea8542bd927059eb&chksm=90afae0da7d8271bdbca55f65c8e21232d7e2dcc437192498edf430143e77a7af893036be6eb&scene=21#wechat_redirect)