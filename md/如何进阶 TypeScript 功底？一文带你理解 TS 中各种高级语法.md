> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rx-FYjUog12v9CqcOsWGEg)

> 作者：19 组清风
> 
> https://juejin.cn/post/7089809919251054628

引言
--

TypeScript 的重要性我不在强调了，我相信仍然会有大多数前端开发者碰到复杂类型一概使用 any 处理。

我写这篇文章的目的就是为了让你告别 AnyScript ，文章告别晦涩的概念结合实例来为你讲述一系列 TS 高级用法：分发、循环、协变、逆变、unknown ... 等等之类。让我们告别枯燥的概念，结合真实用例来掌握 TypeScript 从此彻底告别 AnyScript ！

> 文章并不会从基础的 TS 语法开始讲解，如果你还不了解什么是 TypeScript 强烈建议阅读 TS 官方文档 [1]。

泛型
--

### 泛型基础

熟悉 Java 或者 C# 的朋友对于 泛型的概念也许非常了解，关于泛型的概念这里我并不打算在文章中进行赘述了。

关于如何解释泛型，我看到的最好的一句话概括**把明确类型的工作推迟到创建对象或调用方法的时候才去明确的特殊的类型，简单点来讲我们可以将泛型理解成为把类型当作参数一样去传递。**

比如这样一个简单的例子：

```
function identity<T>(arg: T): T { 
  return arg;
}

// 调用identity时传入name，函数会自动推导出泛型T为string，自然arg类型为T，返回值类型也为T
const userName = identity('name');
// 同理，当然你也可以显示声明泛型
const id = identity<number>(1); 


```

它在 TS 中的确非常重要，同时也有许多非常优秀的文章来讲述它的基础用法。它既重要又基础，是掌握 TS 高级用法的重中之重。

如果你目前还不是非常了解泛型，那么强烈建议你去阅读 Generics Type[2]。

接口泛型位置
------

之所以将接口中的泛型单独拉出来和大家讲述，是因为在日常工作中经常会碰到一些同事对于**泛型接口位置的不理解。**

空口无凭，我们来看看这样一个简单的例子：

上边的 Demo 是一个非常再不同不过的例子了，我们定义接口 IPerson 时，这个接口定义了一个泛型参数 T 表示返回的实例类型。

当使用时，我们需要在使用接口时声明该 T 类型，比如`IPerson<T>`。

接下来我们在看对比另外一个例子：

```
// 声明一个接口IPerson代表函数
interface IPerson { 
  // 此时注意泛型是在函数中参数 而非在IPerson接口中 
  <T>(a: T): T;
}

// 函数接受泛型
const getPersonValue: IPerson = <T>(a: T): T => {  
  return a;
};

// 相当于getPersonValue<number>(2)
getPersonValue(2)


```

这里上下两个例子特别像强调的是关于泛型接口中泛型的位置是代表完全不同的含义：

*   **当泛型出现在接口中时，比如`interface IPerson<T>` 代表的是使用接口时需要传入泛型的类型，比如`IPerson<T>`。**
    
*   **当泛型出现在接口内部时，比如第二个例子中的 `IPerson`接口代表一个函数，接口本身并不具备任何泛型定义。而接口代表的函数则会接受一个泛型定义。换句话说接口本身不需要泛型，而在实现使用接口代表的函数类型时需要声明该函数接受一个泛型参数。**
    

趁热打铁，我们来看这样一个例子：当我们希望实现一个数组的 forEach 方法时，尝试使用泛型来实现：

```
// 定义callback遍历方法 两种方式 应该采用哪一种？
type Callback = <T>(item: T) => void
// 第二种声明方式
type Callback<T> = (item: T) => void;

const forEach = <T>(arr: T[], callback: Callback) => { 
  for (let i = 0; i < arr.length - 1; i++) {  
    callback(arr[i]) 
  }
};

forEach(['1', 2, 3, '4'], (item) => {});


```

关于 forEach 方法我相信大伙儿都已经非常了解了，这里我们尝试使用 TS 来实现这个方法。此时我们将 callback 类型单独抽离出来。

上边的写法有两种声明方式，小伙伴们觉得应该关于 forEach 中的 callback 类型定义应该采用第几种呢？第一种 or 第二种？

> 大家可以结合上边的两个例子自己先稍微思考下。

答案是第二种方式`type Callback<T> = (item: T) => void;`。

这里有一个非常关键的点需要注意，**所谓 TS 是一种静态类型检测，并不会执行你的代码。**

我们先来分析第二种方式的类型定义，我稍微将调用时的代码补充完整（这样方便大伙儿理解）：

```
// item的类型取决于调用函数时传入的类型参数
type Callback = <T>(item: T) => void;

const forEach = <T>(arr: T[], callback: Callback) => { 
  for (let i = 0; i < arr.length - 1; i++) {   
    // 这里调用callback时，ts并不会执行我们的代码。   
    // 换句话说：它并不清楚arr[i]是什么类型   
    callback(arr[i]); 
  }
};

// 所以这里我们并不清楚 callback 定义中的T是什么类型，自然它的类型还是T
forEach(['1', 2, 3, '4'], <T>(item: T) => {});


```

如果采用第二种声明方式，在 forEach 内部的 callback 函数调用时，才会清楚函数传入的参数类型。显然 forEach 调用时无法正确推断出 item 的类型定义。

接下来，我们来看看第二种方式：

```
// item 的类型取决于使用类型时传入的泛型参数
type Callback<T> = (item: T) => void;

// 在声明阶段就已经确定了 callback 接口中的泛型参数为外部传入的
const forEach = <T>(arr: T[], callback: Callback<T>) => { 
  for (let i = 0; i < arr.length - 1; i++) {  
    callback(arr[i]);
  }
};

// 自然，我们在调用forEach时显式声明泛型参数为 string | number 类型
// 所以根据forEach的函数类型定义时，
// 自然 callback 的 item 也会在定义时被推导为 T 也就是所谓的 string | number 类型
forEach<string | number>(['1', 2, 3, '4'], (item) => {});


```

所以，这一点在日常开发中希望小伙伴们一定要特别留意：在泛型接口中泛型的声明位置不同所产生的效果是完全不同的。

### 泛型约束

所谓泛型约束，通俗点来讲就是**约束泛型需要满足的格式**。提到它，有一个非常经典的案例：

```
// 定义方法获取传入参数的length属性
function getLength<T>(arg: T) {  
  // throw error: arr上不存在length属性 
  return arg.length;
}


```

这里，我们定义了一个 getLength 方法，希望函数获取传入参数的 length 属性。

因为传入的参数是不固定的，有可能是 string 、 array 、 arguments 对象甚至一些我们自己定义的 `{ name:"19Qingfeng", length: 100 }`，所以我们为函数增加泛型来为函数增加更加灵活的类型定义。

可是随之而来的问题来了，那么此时我们在函数内部访问了 arg.length 属性。**但是此时，arg 所代表的泛型可以是任意类型。**

比如我们可以传入一个 boolean ，那么此时函数中的泛型 T 代表 boolean 类型，访问 boolean.length ? 这显然是一个 bug 。

那么如果解决这个问题呢，**当然就提到了所谓的泛型约束 extends 关键字。**

我们先来看看如何使用它：

```
interface IHasLength { 
  length: number;
}

// 利用 extends 关键字在声明泛型时约束泛型需要满足的条件
function getLength<T extends IHasLength>(arg: T) { 
  // throw error: arr上不存在length属性 
  return arg.length;
}

getLength([1, 2, 3]); // correct
getLength('123'); // correct
getLength({ name: '19Qingfeng', length: 100 }); // correct
// error 当传入true时，TS会进行自动类型推导 相当于 getLength<boolean>(true)
// 显然 boolean 类型上并不存在拥有 length 属性的约束，所以TS会提示语法错误getLength(true); 


```

类型关键字
-----

> 其实原本一些简单的类型关键字我并不打算在文章中去阐述的，但是后续许多高级类型以及高级概念正是基于这些来实现的，所以文章为了照顾一些不是特别熟悉 TS 的小伙伴还是对于这部分特意进行了讲述。

### keyof 关键字

**所谓 keyof 关键字代表它接受一个对象类型作为参数，并返回该对象所有 key 值组成的联合类型。**

比如：

```
interface IProps { 
  name: string;
  age: number; 
  sex: string;
}

// Keys 类型为 'name' | 'age' | 'sex' 组成的联合类型
type Keys = keyof IProps


```

看上去非常简单对吧，需要额外注意的一点是**当 `keyof any` 时候我们会得到什么类型呢？**

小伙伴们可以稍微思考下 `keyof any` 会得到什么样的类型。

```
// Keys 类型为 string | number | symbol 组成的联合类型
type Keys = keyof any


```

其实这是非常容易理解，any 可以代表任何类型。那么任何类型的 key 都可能为 string 、 number 或者 symbol 。所以自然 keyof any 为 string | number | symbol 的联合类型。

在之后的高级类型中会利用到`keyof any`的特性，所以这里提前拿出来让大家预热下。

在了解了 keyof 关键字之后，让我们结合泛型来实现一个简单的例子来练练手。

比如此时，我们希望实现一个函数。该函数希望接受两个参数，第一个参数为一个对象 object，第二个参数为该对象的 key 。函数内部通过传入的 object 以及对应的 key 返回 `object[key]` 。

```
function getValueFromKey(obj: object, key: string) { 
  // throw error 
  // key的值为string代表它仅仅只被规定为字符串  
  // TS无法确定obj中是否存在对应的key
  return obj[key];
}


```

显然，我们直接为参数声明类型这是会报错的。同学们可以结合刚刚学过的 keyof 关键字配合泛型来思考一下如何消除 TS 的错误提示。

```
// 函数接受两个泛型参数
// T 代表object的类型，同时T需要满足约束是一个对象
// K 代表第二个参数K的类型，同时K需要满足约束keyof T （keyof T 代表object中所有key组成的联合类型）
// 自然，我们在函数内部访问obj[key]就不会提示错误了
function getValueFromKey<T extends object, K extends keyof T>(obj: T, key: K) { 
  return obj[key];
}


```

它的实现非常简单，这里没有写出来的同学可以好好看看文章中上述的内容。

### is 关键字

原本是不打算讲述这个基础概念的，奈何之前在一次面试中因为 is 关键字翻了车哈哈。

面试官问我熟悉 Ts 吗，答案一定是肯定的。结果问了我一个 is 关键字代表的含义，当时的我简直是百思不得其解.. “难道你问的不是 as 吗”，is 究竟是个什么东西好像从来没有听说过。

于是面试结束后赶快搜了搜，结果竟然就是业务中经常用到的**类型谓词**。。。

所谓 is 关键字其实更多用在函数的返回值上，用来表示对于函数返回值的类型保护。

它的用法非常简单：

```
// 函数的返回值类型中 通过类型谓词 is 来保护返回值的类型
function isNumber(arg: any): arg is number { 
  return typeof arg === 'number'
}

function getTypeByVal(val:any) { 
  if (isNumber(val)) {  
    // 此时由于isNumber函数返回值根据类型谓词的保护  
    // 所以可以断定如果 isNumber 返回true 那么传入的参数 val 一定是 number 类型   
    val.toFixed() 
  }
}


```

所以，通常我们使用 is 关键字（类型谓词）在函数的返回值中，从而对于函数传入的参数进行类型保护。

### infer 关键字

> infer 关键字我不打算放在这里和大家描述了，我会在下面的内容和大家逐步切入对应的关键字。

TS 高级概念
-------

### 分发

在讲述分发的概念，我会先和你聊聊 TS 中的 Conditional Types （条件类型）。

因为大多数高级类型都是基于条件类型，同时分发的概念也和 Conditional Types 息息相关，所以我们先来看看所谓的 Conditional Types 究竟是什么。

```
type isString<T> = T extends string ? true : false;

// a 的类型为 true
let a: isString<'a'>

// b 的类型为 false
let b: isString<1>;


```

上边我们通过 type 关键字定义了一个所谓的 isString 类型，它接受一个泛型参数 T 。

isString 类型内部通过 extends 关键字结合 ? 和 : 实现了所谓的 Conditional Types （条件类型）判断。

`type isString<T> = T extends string ? true : false;`

稍微翻译翻译上边这段代码，当泛型 T 满足 string 类型的约束时，它会返回 true ，否则则会返回 false 类型。

**其实所谓的条件类型就是这么简单，看起来和三元表达式非常相似，甚至你完全可以将它理解成为三元表达式。只不过它接受的是类型以及判断的是类型而已。**

需要额外注意的是：

*   **这里的 `T extends string` 更像是一种判断泛型 T 是否满足 string 的判断，和之前所讲的泛型约束完全不是同一个意思。**
    

上述我们讲的泛型约束是在定义泛型时进行对于传入泛型的约束，而这里的 `T extends string ? true : false` 并不是在传入泛型时进行的约束。

在使用 isString 时，你可以为它传入任意类型作为泛型参数的实现。但是 isString 类型内部会对于传入的泛型类型进行判断，如果 T 满足 string 的约束条件，那么返回类型 true，反过来则是 false 。

*   **其次，需要注意的是条件类型 `a extends b ? c : d` 仅仅支持在 type 关键字中使用。**
    

在了解了泛型约束之后，我们在回到所谓**分发**的概念上来。

一起来看看这样一个例子：

```
type GetSomeType<T extends string | number> = T extends string ? 'a' : 'b';

let someTypeOne: GetSomeType<string> // someTypeone 类型为 'a'

let someTypeTwo: GetSomeType<number> // someTypeone 类型为 'b'

let someTypeThree: GetSomeType<string | number>; // what ? 


```

这里我们定义了一个 GetSomeType 的类型，它接受一个泛型参数 T 。这个泛型参数 T 在传入时需要满足为 string 和 number 的联合类型的约束。（换句话说，要么为 string 要么为 number 要么为 string ｜ number）。

*   首先，someTypeOne 变量的类型为 `GetSomeType<string>`。
    

因为我们为 someTypeOne 定义时传入了 string 的类型参数，所以按照条件类型来判断，`string extends string` 明显是满足的，所以返回类型'a'。

*   同理，someTypeTwo 变量的类型也会被推断为'b'，这个过程我就不在累赘了。
    

**那么重点来了，someTypeThree 定义时的类型 GetSomeType<'string' | 1> 我们传入的泛型参数为联合类型时'string' | 1 时，它会的到什么类型呢？**

首先不难想象，我们按照正常逻辑来思考。'string' | 1 一定是不满足 `T extends string`，因为一个 `'string' | 1` 的联合类型一定是无法和 string 类型做兼容的。

那么按照我们之前的逻辑来梳理，理所应当 someTypeThree 的类型应该是'b' 对吧。

可是结果真如那么简单的话，那么我还举出来这个例子做什么呢？

![](https://mmbiz.qpic.cn/sz_mmbiz/H8M5QJDxMHrAd0uNJn1aKnryja5Ilq40ficNsECjdGamJdrLWnamwdkSdSOReR753LibicKvh40lx3NpQYtWq0WJA/640?wx_fmt=jpeg)

很惊讶吧，someTypeThree 的类型竟然被推导成为了'a' | 'b' 组成的联合类型，那么为什么会这样呢。

**其实这就是所谓分发在捣鬼。**

**我们抛开晦涩的概念来解读分发，结合上边的 Demo 来说所谓的分发简单来说就是分别使用 string 和 number 这两个类型进入 GetSomeType 中进行判断，最终返回两次类型结果组成的联合类型。**

> 当然，你可以在使用 GetSomeType 你可以传入 n 个类型组成的联合类型作为泛型参数，同理它会进行进入 GetSomeType 类型中进行 n 次分发判断。

```
// 你可以这样理解分发
// 伪代码：GetSomeType<string | number> = GetSomeType<string> | GetSomeType<number>
let someTypeThree: GetSomeType<string | number>


```

*   **首先，毫无疑问分发一定是需要产生在 extends 产生的类型条件判断中，并且是前置类型。**
    

比如`T extends string | number ? 'a' : 'b';` 那么此时，产生分发效果的也只有 extends 关键字前的 T 类型，string | number 仅仅代表一种条件判断。

*   **其次，分发一定是要满足联合类型，只有联合类型才会产生分发**（其他类型无法产生分发的效果，比如 & 交集中等等）。
    
*   **最后，分发一定要满足所谓的裸类型中才会产生效果。**
    

这里的裸类型稍微和大家解释下，比如这样:

```
// 此时的T并不是一个单独的”裸类型“T 而是 [T]
type GetSomeType<T extends string | number | [string]> = [T] extends string[]  
  ? 'a' 
  : 'b';
  
// 即使我们修改了对应的类型判断，仍然不会产生所谓的分发效果。因为[T]并不是一个裸类型
// 只会产生一次判断  [string] | number extends string[]  ? 'a' : 'b'
// someTypeThree 仍然只有 'b' 类型 ，如果进行了分发的话那么应该是 'a' | 'b'
let someTypeThree: GetSomeType<[string] | number>;


```

同样，在了解了分发的概念和清楚了如何会产生分发的效果后。趁热打铁我们来看看利用分发我们可以实现什么样的效果：

在 TypeScript 内部拥有一个高级内置类型 Exclude 意为排除，它的用法如下：

```
type TypeA = string | number | boolean | symbol;

// ExcludeSymbolType 类型为 string | number | boolean，排除了symbol类型
type ExcludeSymbolType = Exclude<TypeA, symbol>;


```

用法非常简单，Exclude 内置类型会接受两个类型泛型参数。它会构造一个新的类型，这个类型会排除所有 TypeA 类型中满足 symbol 的类型。

那么，如果让你来实现一个 Exclude 内置类型，你会如何实现呢？同学们可以结合分发自行思考下。

如果没有想出来的小伙伴，强烈建议在重新好好温习一下分发这个章节。

```
type TypeA = string | number | boolean | symbol;

type MyExclude<T, K> = T extends K ? never : T;

// ExcludeSymbolType 类型为 string | number | boolean，排除了symbol类型

type ExcludeSymbolType = MyExclude<TypeA, symbol | boolean>;


```

其实它的实现非常简单，上述我们通过分发来实现了对应的 MyExclude 类型。

MyExclude 类型接受两个泛型参数，因为 `T extends K ? never : T` 中 T 满足裸类型并且在 extends 关键字前。

同时，我们传入的 TypeA 为联合类型，那么满足分发的所有条件。则会产生分发效果，也就是说会将联合类型 TypeA 中所有的单个类型依次进入 `T extends K ? never : T;` 去判断。

当满足条件时，也就是 `T extends symbol | boolean` 时，此时会得到 never 。（这里的 never 代表的也就是一个无法达到的类型，不会产生任何效果），自然就会被忽略。

而如果不满足 `T extends symbol | boolean` 则会被记录，最终返回不满足 `T extends symbol | boolean` 的所有类型组成的联合类型，也就是所谓的 `string | number` 。

> 当然和 Exclude 相反效果的内置类型 Extract[3]、NonNullable[3] 也是基于分发实现的，有兴趣的小伙伴可以自行查阅实现。

### 循环

TypeScript 中同样存在对于类型的循环语法 (Mapping Type)，通过我们可以通过 in 关键字配合联合类型来对于类型进行迭代。比如这样：

```
interface IProps {
  name: string;
  age: number; 
  highSchool: string;  
  university: string;
}

// IPropsKey类型为
// type IPropsKey = {
//  name: boolean;
//  age: boolean;
//  highSchool: boolean;
//  university: boolean;
//  }
type IPropsKey = { [K in keyof IProps]: boolean };


```

其实相对来说循环关键字 in 比较简单，上述代码我们声明了一个所谓的 IPropsKey 的类型：

首先可以看到这个类型是一个对象，对象中的 key 为 `[]` 包裹的可计算值，value 为 boolean。

`keyof IProps` 我们在之前提到过它会返回 IProps 所有 key 组成的联合类型，也就是 `'name' | 'age' | 'highSchool' | 'university'` 。

而 `[K in keyof IProps]` 正是我们在类型内部声明了一个变量 K 。

你可以理解为 in 关键字的作用类似于 for 循环，它会循环 keyof IProps 这个联合类型中的每一项类型，同时在每一次循环中将对应的类型赋值给 K 。

最终，通过一次一次循环我们到了最终的新类型 `interface IProps { name: string; age: number; highSchool: string; university: string; }` 。

那么在 TS 中我们可以利用循环的特性来做什么呢？不知道大家有没有用到 Partial 之类的内置类型。

所谓的 Partial 功能非常简单：它会构造一个新的类型，这个类型会将之前类型中的所有属性都变为可选。

比如这样：

![](https://mmbiz.qpic.cn/sz_mmbiz/H8M5QJDxMHrAd0uNJn1aKnryja5Ilq40lNyNkmWHEKjXXruYbV9uLNLkmIs4OkSU5tQh5eMuITpNHpF7MDve4A/640?wx_fmt=jpeg)

可以看到我们通过 Partial 传入 IInfo 类型，它返回一个新类型 OptinalInfo，OptinalInfo 会将 IInfo 中所有的属性都变为可选类型。

同样，大家可以自己动手来实现一下它。其实结合我们刚刚说到的循环来实现会非常简单。

```
interface IInfo { 
  name: string;
  age: number;
}

type MyPartial<T> = { [K in keyof T]?: T[K] };

type OptionalInfo = MyPartial<IInfo>;


```

看起来非常简单对吧，我们通过 in 循环传入的 IInfo，同时构造了一个新的类型它的 key 和 IInfo 是一模一样的。

只不过仅仅是所有属性 key 是可选的，而非必填。

> 当然需要注意的是我们刚才提到的所有关键字，比如 extends 进行条件判断或者 in 进行类型循环时，仅仅支持在 type 类型声明中使用，并不可以在 interface 中使用，这也是 type 和 interface 声明的一个不同。

当然，还有许多内置类型同样利用了循环，比如 Required、Readonly 等等。

同学们可以思考下如果让你实现一个 Required 应该如何实现，它会利用到一些 Mapping Modifiers[4]（映射修饰符），有兴趣的朋友可以实现一下练练手。

当然在循环的最后，我们来思考另一个问题。其实你会发现无论是 TS 内置的 Partial 还是我们刚刚自己实现的 Partial ，它仅仅对象中一层的转化并不能递归处理。

比如说：

```
interface IInfo { 
  name: string; 
  age: number;  
  school: {  
    middleSchool: string;   
    highSchool: string;  
    university: string; 
  }
}

type OptionalInfo = Partial<IInfo>;


```

![](https://mmbiz.qpic.cn/sz_mmbiz/H8M5QJDxMHrAd0uNJn1aKnryja5Ilq40dDFpNiaiah7h6UtlicAeFhnlhIWJASbyBs7ba4TQAZp5WicnsueuL4kIGg/640?wx_fmt=jpeg)

可以看到利用 Partial 关键字仅仅对于对象类型中的最外层进行了可选标记。

但是对于内层嵌套类型比如 school 仍是一个对象类型，那么此时是无法深度进入 school 类型中进行标记的。

那么假如此时我有需求希望实现深度可选，应该如何做呢？大家可以往上边提到过的条件判断和循环结合来考虑下。

```
interface IInfo { 
  name: string;  
  age: number; 
  school: {   
    middleSchool: string;   
    highSchool: string;  
    university: string; 
  };
}

// 其实实现很简单，首先我们在构造新的类型value时
// 利用 extends 条件判断新的类型value是否为 object 
// 如果是 -> 那么我仍然利用 deepPartial<T[K]> 进行包裹递归可选处理
// 如果不是 -> 普通类型直接返回即可
type deepPartial<T> = { 
  [K in keyof T]?: T[K] extends object ? deepPartial<T[K]> : T[K];
};

type OptionalInfo = deepPartial<IInfo>;

let value: OptionalInfo = { 
  name: '1', 
  school: {  
    middleSchool:'xian' 
  },
};


```

不卖关子了，如果对于文章之前的知识点你都掌握了，那么我相信实现这个功能对你来说简直是小菜一碟。

> 其实看到这里，TS 内置的一些类型比如 Pick 、 Omit 大家都可以尝试自己去实现下了。我们之前说到了知识点已经可以完全涵盖这些内置类型的实现。

### 逆变

> 许多不是很熟悉 TS 的朋友对于逆变和协变的概念会感到莫名的恐惧，没关系。它们仅仅代表阐述表现的概念而已，放心我们并不会从概念入手而是通过实例来逐步为你揭开它的面纱。

首先，我们先来思考这样一个场景：

```
let a!: { a: string; b: number };
let b!: { a: string };

b = a


```

我们都清楚 TS 属于静态类型检测，所谓类型的赋值是要保证安全性的。

**通俗来说也就是多的可以赋值给少的**，上述代码因为 a 的类型定义中完全包括 b 的类型定义，所以 a 类型完全是可以赋值给 b 类型，这被称为类型兼容性。

之后，我们再来思考这样一段代码：

```
let fn1!: (a: string, b: number) => void;
let fn2!: (a: string, b: number, c: boolean) => void;

fn1 = fn2; // TS Error: 不能将fn2的类型赋值给fn1


```

我们将 fn2 赋值给 fn1 ，刚刚才提到类型兼容性的原因 TS 允许不同类型进行互相赋值（只需要父 / 子集关系），那么明明 fn2 的参数包括了所有的 fn1 为什么会报错？

上述的问题，其实和刚刚没有什么本质区别。我们来换一个角度来理解这个问题：

针对于 fn1 声明时，函数类型需要接受两个参数，换句话说调用 fn1 时我需要支持两个参数的传入分别是 `a:string`和`b:number`。

同理 fn2 函数定义时，定义了三个参数那么调用 fn2 时自然也需要传入三个参数。

那么此时，我们将 fn2 赋值给 fn1 ，我们可以思考下。如果赋值成功了，当我调用 fn1 时，其实相当于调用 fn2 没错吧。

**但是，由于 fn1 的函数类型定义仅仅支持两个参数 `a:string`和`b:number` 即可。但是由于我们执行了 `fn1 = fn2`。**

**调用 fn1 时，实际相当于调用了 fn2 函数。但是类型定义上来说 fn1 满足两个参数传入即可，而 fn2 是实打实的需要传入 3 个参数。**

那么此时，如果执行了 fn1 = fn2 当调用 fn1 时明显参数个数会不匹配（由于类型定义不一致）会缺少一个第三个参数，显然这是不安全的，自然也不是被 TS 允许的。

那么反过来呢？

```
let fn1!: (a: string, b: number) => void;
let fn2!: (a: string, b: number, c: boolean) => void;

fn2 = fn1; // 正确，被允许


```

按照刚才的思路来分析，我们将 fn1 赋值给 fn2 。fn2 的类型定义需要支持三个参数的传入，但实际 fn2 内部指针已经被修改称为 fn1 的指针。

fn1 在执行时仅仅需要两个参数 `a: string, b: number`，显然 fn2 的类型定义中是满足这个条件的（当然它还多传递了第三个参数 `c:boolean`，在 JS 中对于函数而言调用时的参数个数大于定义时的参数个数是被允许的）。

自然，这是安全的也是被 TS 允许赋值。

**就比如上述函数的参数类型赋值就被称为逆变，参数少（父）的可以赋给参数多（子）的那一个。看起来和类型兼容性（多的可以赋给少的）相反，但是通过调用的角度来考虑的话恰恰满足多的可以赋给少的兼容性原则。**

> 上述这种函数之间互相赋值，他们的参数类型兼容性是典型的逆变 [5]。

我们再来看一个稍微复杂点的例子来加深所谓逆变的理解：

```
class Parent {}

// Son继承了Parent 并且比parent多了一个实例属性 name
class Son extends Parent { 
  public name: string = '19Qingfeng';
}

// GrandSon继承了Son 在Son的基础上额外多了一个age属性
class Grandson extends Son { 
  public age: number = 3;
}

// 分别创建父子实例
const son = new Son();

function someThing(cb: (param: Son) => any) { 
  // do some someThing 
  // 注意：这里调用函数的时候传入的实参是Son 
  cb(Son);
}

someThing((param: Grandson) => param); // error
someThing((param: Parent) => param); // correct


```

这里我们定义了三个类，他们之间的关系分别是 Parent 是基类，Son 继承 Parent ，Grandson 继承 Son 。

同时我们定义了一个函数，它接受一个 cb 回调参数作为参数，我们定义了这个回调函数的类型为接受一个 param 为 Son 实例类型的参数，此时我们不关心它的返回值给一个 any 即可。

注意这里，我们先用刚才的结论来推导。刚才我们提到过**函数的参数的方式被称为逆变**，所以当我们调用 someThing 时传递的 callback 需要赋给定义 something 函数中的 cb 。

换句话说类型 `(param: Grandson) => param` 需要赋给 `cb: (param: Son) => any`，这显然是不被允许的。

因为逆变的效果函数的参数只允许 “从少的赋值给多的”，显然 Grandson 相较于 Son 来说多了一个 name 属性少，所以这是不被允许的。

相反，第二个`someThing((param: Parent) => param);`相当于函数参数重将 Parent 赋给 Son 将少的赋给多的满足逆变，所以是正确的。

之后我们在尝试分析为什么第二个`someThing((param: Parent) => param);`是正确的。

首先我们需要注意到我们在定义 someThing 函数时，声明了这个函数接受一个 cb 的函数。这个函数接受一个类型为 Son 的参数。

someThing 内部 **cb 函数声明时需要满足 Son 的参数，它会在 cb 函数调用时传入一个 Son 参数的实参**。

所以当我们传入 `someThing((param: Parent) => param)` 时，相当于在 something 函数内部调用 `(param: Parent) => param` 时会根据 someThing 中 callback 的定义传入一个 Son 。

那么此时，我们函数真实调用时期望得到是 Parent，但是实际得到了 Son 。Son 是 Parent 的子类涵盖所有 Parent 的公共属性方法，自然也是满足条件的。

反而言之，当我们使用`someThing((param: Grandson) => param);` ，由于 something 定义 cb 的类型传入 Son，但是真实调用 someThing 时，我们确需要一个 Grandson 类型参数的函数，这显然是不符合的。

> 关于逆变我用了比较多的篇幅去描述它，我希望通过文章大家都可以对于逆变结合实例来理解并应用它。因为它的确稍微有些绕。

### 协变

解决了逆变之后，其实协变对于大伙儿来说都是小意思。我们先来看看这个 Demo:

```
let fn1!: (a: string, b: number) => string;
let fn2!: (a: string, b: number) => string | number | boolean;

fn2 = fn1; // correct 
fn1 = fn2 // error: 不可以将 string|number|boolean 赋给 string 类型


```

这里，**函数类型赋值兼容时函数的返回值就是典型的协变场景**，我们可以看到 fn1 函数返回值类型规定为 string，fn2 返回值类型规定为 string | number | boolean 。

显然 string | number | boolean 是无法分配给 string 类型的，但是 string 类型是满足 string | number | boolean 其中之一，所以自然可以赋值给 string | number | boolean 组成的联合类型。

其实这就是协变.... 当然你也可以尝试从函数运行角度来解读协变的概念，比如当 fn1 运行结束要求返回 string ， fn2 运行结束后要求返回 string | number | boolean 。

将 fn1 赋给 fn2 ，fn1 要求返回值是 string ，而真实调用的 fn1=fn2 相当于调用了 fn2 自然 string | number | boolean 无法满足 string 类型的要求，所以 TS 会认为这是错误的。

### 待推断类型

**infer 代表待推断类型，它的必须和 extends 条件约束类型一起使用。**

之前，我们在 类型关键字中遗留了 infer 关键字并没有展开讲述，这里我们了解了所谓的 extends 代表的类型约束之后我们来一起看看所谓 infer 带来的待推断类型效果。

在条件类型约束中为我们提供了 infer 关键字来提供实现更多的类型可能，它表示我们可以在条件类型中推断一些暂时无法确定的类型，比如这样：

```
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;


```

上述我们定义了一个 Flatten 类型，它接受一个传入的泛型 Type ，我们在类型定义内部对于传入的泛型 Type 进行了条件约束：

*   如果 Type 满足 `Array<infer Item>`，那么此时返回 Item 类型。
    
*   如果 Type 不满足 `Array<infer Item>`类型，那么此时返回 Type 类型。
    

关于如何理解 `Array<infer Item>`，**一句话描述就是我们利用 infer 声明了一个数组类型，数组中值的类型我们并不清楚所以使用 infer 来进行推断数组中的值。**

比如：

![](https://mmbiz.qpic.cn/sz_mmbiz/H8M5QJDxMHrAd0uNJn1aKnryja5Ilq40uicvAibLDJLJQaAlM2TWs5gfv3licKtsQAaIUoP5kYEP04IWPdtVMKXKA/640?wx_fmt=jpeg)

我们为类型 Flatten 传入一个 string 类型，显然传入的 string 并不满足数组的约束。自然直接返回传入的 string 类型。

此时我们试试传入一个数组类型呢：

![](https://mmbiz.qpic.cn/sz_mmbiz/H8M5QJDxMHrAd0uNJn1aKnryja5Ilq40qLLzqrv8nj0IIKW02nicqibedPhWJLibPjUiaYAug5nYxPKOIuwAXdMWHA/640?wx_fmt=jpeg)

可以看到返回的 subType 类型为 string ｜ number 。我们来稍微分析这一过程：

声明`Flatten<[string, number]>`时，Flatten 接受到一个 `[string,number]` 的泛型参数。

显然 `[string,number]`是满足数组的条件的，`Type extends Array<infer Item>`。

所谓的 `Array<infer Item>`代表的进行条件判断时要求前者（Type）必须是一个数组，但是数组中的类型我并不清楚（或者说可以是任意）。

自然我们使用 infer 关键字表示待推断的类型， infer 后紧跟着类型变量 Item 表示的就是待推断的数组元素类型。

**我们类型定义时并不能立即确定某些类型，而是在使用类型时来根据条件来推断对应的类型**。之后，因为数组中的元素可能为 string 也可能为 number，自然在使用类型时 infer Item 会将待推断的 Item 推断为 string | number 联合类型。

> 需要注意的是 infer 关键字类型，必须结合 Conditional Types 条件判断来使用。

那么，在条件类型中结合 infer 会帮助我们带来什么样的作用呢？我们一起来看看 infer 的实际用法。

在 TS 中存在一个内置类型 Parameters ，它接受传入一个函数类型作为泛型参数并且会返回这个函数所有的参数类型组成的元祖。

```
// 定义函数类型
interface IFn { 
  (age: number, name: string): void;
}

// type FnParameters = [age: number, name: string]
type FnParameters = Parameters<IFn>;

let a: FnParameters = [25, '19Qingfeng'];


```

它的内部实现恰恰是利用 infer 来实现的，同学们可以自己尝试来实现这个内置类型。

```
type MyParameters<T extends (...args: any) => any> = T extends (  
  ...args: infer R
) => any 
  ? R 
  : never;


```

其实它的实现非常简单，定义的 MyParameters 类型中接受一个泛型 T 当传入 T 时需要满足它为函数类型的约束。

其次我们在 MyParameters 内部对于 传入的泛型参数进行了条件判断，如果满足条件也就是 `T extends ( ...args: infer R ) => any`，需要注意的是条件判断中函数的参数并不是在类型定义时就确认的，函数的参数需要根据传入的泛型来确认后赋给变量 R 所以使用了 infer R 来表示待推断的函数参数类型。

那么此时我会返回满足条件的函数推断参数组成的数组也就是 ...args 的类型 R ，否则则返回 never 。

> 当然 TS 内部还存在比如 ReturnType 、ThisParameterType 等类型都是基于条件判断中的 infer 来推断出结果的，有兴趣的朋友可以自行查阅。

日常工作中，我们经常会碰到将元祖转化成为联合类型的需求，比如 `['a',1,true]` 我们希望快速得到元组中元素的类型应该如何实现呢？

![](https://mmbiz.qpic.cn/sz_mmbiz/H8M5QJDxMHrAd0uNJn1aKnryja5Ilq40qPRnBtoLWaPDyibFgaqH6qy9bFiaDtMMzEAPzEpJb4WY2Nw4mN6b9kgQ/640?wx_fmt=jpeg)

### unknown & any

在 TypeScript 中同样存在一个高级类型 unknown ，**它可以代表任意类型的值**，这一点和 any 是非常类型的。

但是我们清楚将类型声明为 any 之后会跳过任何类型检查，比如这样：

```
let myName: any;

myName = 1

// 这明显是一个bug
myName()


```

而 unknown 和 any 代表的含义完全是不一样的，虽然 unknown 可以和 any 一样代表任意类型的值，但是这并不代表它可以绕过 TS 的类型检查。

```
let myName: unknown;

myName = 1

// ts error: unknown 无法被调用，这被认为是不安全的
myName()

// 使用typeof保护myName类型为function
if (typeof myName === 'function') { 
  // 此时myName的类型从unknown变为function  
  // 可以正常调用 
  myName()
}


```

通俗来说 unknown 就代表一些并不会绕过类型检查但又暂时无法确定值的类型，我们在一些无法确定函数参数（返回值）类型中 unknown 使用的场景非常多。比如：

```
// 在不确定函数参数的类型时
// 将函数的参数声明为unknown类型而非any
// TS同样会对于unknown进行类型检测，而any就不会
function resultValueBySome(val:unknown) { 
  if (typeof val === 'string') {  
    // 此时 val 是string类型   
    // do someThing 
  } else if (typeof val === 'number') { 
    // 此时 val 是number类型   
    // do someThing  
  } 
  // ...
}


```

当然，在描述了 unknown 类型的含义之后，关于 unknown 类型有一个特别重要的点我想和大家强调：

![](https://mmbiz.qpic.cn/sz_mmbiz/H8M5QJDxMHrAd0uNJn1aKnryja5Ilq401WFiartJMhKM7Et6aMDTerrZZQHwpC9fz0yAXIm7uQNQwkUG5ECPDyA/640?wx_fmt=jpeg)

**unknown 类型可以接收任意类型的值，但并不支持将 unknown 赋值给其他类型。**

**any 类型同样支持接收任意类型的值，同时赋值给其他任意类型（除了 never）。**

any 和 unknown 都代表任意类型，但是 unknown 只能接收任意类型的值，而 any 除了可以接收任意类型的值，也可以赋值给任意类型（除了 never）。

比如下面这样：

```
let a!: any;
let b!: unknown;

// 任何类型值都可以赋给any、unknown
a = 1;
b = 1;

// callback函数接受一个类型为number的参数
function callback(val: number): void {}

// 调用callback传入aaa（any）类型 correct
callback(a);

// 调用callback传入b（unknown）类型给 val（number）类型 error
// ts Error: 类型“unknown”的参数不能赋给类型“number”的参数callback(b);


```

当然，对于以后并不确定类型的变量希望大家尽量使用更多的 unknown 来代替 any 让你的代码更加强壮。

写在结尾
----

至此，文章对于 TypeScript 的内容就在这里和大家告一段落了。

感谢每一位看到这里的小伙伴，其实关于如何精进 TypeScript 功底在我个人看来可以总结为以下两点：

第一，碰到问题一定是要结合文档多查阅文档（当然 TypeScript 一定是要去尝试阅读英文文档，及时你的英语不是那么好），它的中文文档实在是过于简陋了。

第二，关于 TS 中的确存在对于普通开发者太多的陌生概念。掌握它最直接的办法就是去用 TS 在任何你能用到的地方，哪怕只是一个特别小的项目，正所谓所谓熟能生巧嘛。

### 参考资料

[1]

TS 官方文档: https://www.typescriptlang.org/docs/handbook/2/basic-types.html

[2]

Generics Type: https://www.typescriptlang.org/docs/handbook/2/generics.html

[3]

Extract:  https://www.typescriptlang.org/docs/handbook/utility-types.html#extracttype-union

[4]

Mapping Modifiers: https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers

[5]

逆变: https://zh.wikipedia.org/wiki / 协变与逆变