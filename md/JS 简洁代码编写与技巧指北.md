> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651578080&idx=2&sn=52d5bcf0dda0d7e83966fbacc03ec5ad&chksm=80250921b7528037caa44fc0fc95d86f9550235e2c358792863a22b38666771bd6637685a768&mpshare=1&scene=1&srcid=0623rY1houssQEEZms9ZNcw2&sharer_sharetime=1624420585711&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

if 层级嵌套优化  

============

如下面的三层 if 条件嵌套

```
function froutCheck(fruit, quantity) {
  const redFruits = ['apple', 'pear', 'cherry', 'banana'];
  // 条件 1: 水果存在
  if(fruit) {
    // 条件 2: 属于红色水果
    if(redFruits.includes(fruit)) {
      console.log('红色水果');
      // 条件 3: 水果数量大于 10 个
      if (quantity > 10) {
        console.log('数量大于 10 个');
      }
    }
  } else {
    throw new Error('没有水果啦!');
  }
}


```

“早处理早返回” 的原则来写的话，逻辑会更加清晰和容易维护

```
function supply(fruit, quantity) {
  const redFruits = ['apple', 'pear', 'cherry', 'banana'];
  if(!fruit) throw new Error('没有水果啦');     // 条件 1: 当 fruit 无效时，提前处理错误
  if(!redFruits.includes(fruit)) return; // 条件 2: 当不是红色水果时，提前 return
    
  console.log('红色水果');
    
  // 条件 3: 水果数量大于 10 个
  if (quantity > 10) {
    console.log('数量大于 10 个');
  }
}


```

Array 的妙用
---------

### Array.includes 判断多个 if 条件

可以在数组中存储多个值，使用数组 `include` 方法。

```
//Longhand
if (x === 'abc' || x === 'def' || x === 'ghi' || x ==='jkl') {
  //logic
}

//Shorthand
if (['abc', 'def', 'ghi', 'jkl'].includes(x)) {
  //logic
}


```

### Array.find 查找符合条件的数组元素

当我们确实有一个对象数组并且我们想要根据对象属性查找特定对象时，find 方法确实很有用。

```
const data = [
  {
    type: 'test1',
    name: 'abc'
  },
  {
    type: 'test2',
    name: 'cde'
  },
  {
    type: 'test1',
    name: 'fgh'
  },
]
function findtest1(name) {
  for (let i = 0; i < data.length; ++i) {
    if (data[i].type === 'test1' && data[i].name === name) {
      return data[i];
    }
  }
}

//Shorthand
filteredData = data.find(data => data.type === 'test1' && data.name === 'fgh');
console.log(filteredData); // { type: 'test1', name: 'fgh' }


```

### 数组中所有项都满足某条件：**Array.every**

### 数组中是否有某一项满足条件：**Array.some**

**还有 Array.find、Array.slice、Array.findIndex、Array.reduce、Array.splice 等，在实际场景中可以根据需要使用。**

真值判断简写
------

```
// Longhand
if (tmp === true) or if (tmp !== "") or if (tmp !== null)

// Shorthand 
//it will check empty string,null and undefined too
if (test1)


```

**注意**：该方式主要用于 `null` 或 `undefined` 的检查，** 特别需要注意 tmp=0 或者 tmp=‘0’** 都是 false。

多条件的 && 运算符
-----------

如果仅在变量为 `true` 的情况下才调用函数，则可以使用 `&&` 运算符。

```
//Longhand
if (test1) {
 callMethod();
}

//Shorthand
test1 && callMethod();


```

三元运算符实现短函数调用
------------

我们可以使用三元运算符来实现函数的直接执行。

```
// Longhand
function test1() {
  console.log('test1');
};
function test2() {
  console.log('test2');
};
var test3 = 1;
if (test3 == 1) {
  test1();
} else {
  test2();
}

// Shorthand
(test3 === 1? test1:test2)();


```

对象属性解构简写
--------

```
let test1 = 'a';
let test2 = 'b';

//Longhand
let obj = {test1: test1, test2: test2};

//Shorthand
let obj = {test1, test2};


```

**比较大小使用 a - b > 0 的方式更好**，用 a > b 有时候会出现错误
-------------------------------------------

```
错误用法
'20' > '100'  // true

预期结果
'20' - '100' > 0   // false

//数组排序算法
arr.sort((a, b ) =>{
  return a-b 
})


```

If,for…in,for…of 和的使用
---------------------

1. 能用三元运算符就用, 减少 if 的嵌套

2. 减少多余条件判断，查询结束立即返回 **（早返回，多返回）**, 如果是函数返回 if 里面和外面返回相同的数据类型

3.If…else if…else 多个条件时以 else 结尾, 因为符合防御性编程规则

4.NaN 不应该用于比较, 应该是判断是否是数字

5.Switch…case 使用在至少有三个判断值, case 不可省, 每次 case 必须用 break 跳出

6.for…of 遍历数组和字符串

7.for…in 遍历对象

For…in 遍历对象包括所有继承的属性, 所以如果只是想使用对象本身的属性需要做一个判断

8. 在循环内部声明函数慎用, 因为是循环执行完成函数调用才会执行

9.Return 后面不要写代码

if 多条件判断
--------

“早点返回，经常返回 “（return early，return often），If 语句用来处理主路径上的异常

比如下面的 find 函数。

```
function find(predicate, arr) {
    for (let item of arr) {
        if (predicate(item)) {
            return item;
        }
    }
}


```

在这个 find 函数内，一旦我们找到想查找的对象就立刻返回该对象并且退出循环。这使得我们的代码更加高效。

### switch 语句优化

低级版：太容易忘记写 break

```
let notificationPtrn;
switch (notification.type) {
    case 'citation':
        notificationPtrn = 'You received a citation from {{actingUser}}.';
        break;
    case 'follow':
        notificationPtrn = '{{actingUser}} started following your work';
        break;
    case 'mention':
        notificationPtrn = '{{actingUser}} mentioned you in a post.';
        break;
    default:
        // Well, this should never happen
}


```

### 优化一：用 return

```
function getnotificationPtrn(n) {
        switch (n.type) {
            case 'citation':
                return 'You received a citation from {{actingUser}}.';
            case 'follow':
                return '{{actingUser}} started following your work';
            case 'mention':
                return '{{actingUser}} mentioned you in a post.';
            default:
                throw new Error('You’ve received some sort of notification we don’t know about.';
        }
    }
    let notificationPtrn = getNotificationPtrn(notification);


```

### 优化二：采用对象字典

```
function getNotificationPtrn(n) {
    const textOptions = {
        citation: 'You received a citation from {{actingUser}}.',
        follow:   '{{actingUser}} started following your work',
        mention:  '{{actingUser}} mentioned you in a post.',
    }
    return textOptions[n.type];
}


```

### 优化三（推荐） 类似模式匹配方法

```
const textOptions = {
    citation: 'You received a citation from {{actingUser}}.',
    follow:   '{{actingUser}} started following your work',
    mention:  '{{actingUser}} mentioned you in a post.',
}
function getNotificationPtrn(textOptions, n) {
    return textOptions[n.type];
}
const notificationPtrn = getNotificationPtrn(textOptions, notification);


```

若包含未知处理，我们也可以把默认的信息作为参数

```
constdefaultTxt= 'You’ve received some sort of notification we don’t know about.';   // 默认值
function getNotificationPtrn(defaultTxt, textOptions, n) {
    return textOptions[n.type] || defaultTxt;
}
const notificationPtrn = getNotificationPtrn(defaultTxt, textOptions, notification.type);


```

现在 textOptions 是一个变量。并且该变量不用再被硬编码了。我们可以把它移动到一个 JSON 配置文件中，或者从服务器获取该变量。现在我们想怎么改 textOptions 就怎么改。我们可以增加新的选项，或者移除选项。我们也可以把多个地方不同的选项合并到一起。

链式取值 a[0].b 不存在的 undefined 问题
-----------------------------

开发中，链式取值是非常正常的操作，如：

```
res.data.goods.list[0].price


```

但是对于这种操作报出类似于 Uncaught TypeError: Cannot read property 'goods' of undefined 这种错误也是再正常不过了，如果说是 res 数据是自己定义，那么可控性会大一些，但是如果这些数据来自于不同端（如前后端），那么这种数据对于我们来说我们都是不可控的，因此为了保证程序能够正常运行下去，我们需要对此校验：

```
if (res.data.goods.list[0] && res.data.goods.list[0].price) {
// your code
}
如果再精细一点，对于所有都进行校验的话，就会像这样:
if (res && res.data && res.data.goods && res.data.goods.list && res.data.goods.list[0] && res.data.goods.list[0].price){
// your code
}


```

不敢想象，如果数据的层级再深一点会怎样，这种实现实在是非常不优雅，那么如果优雅地来实现链式取值呢？

### 方法一. 通过函数解析字符串（lodash 的 _.get 方法）

我们可以通过函数解析字符串来解决这个问题，这种实现就是 lodash 的 _.get 方法 (www.lodashjs.com/docs/4.17.5…)

```
// _.get(object, path, [defaultValue])  随后一个参数是取值为undefined时候的默认值
var object = { a: [{ b: { c: 3 } }] };
var result = _.get(object, 'a[0].b.c', 1);
console.log(result);
// output: 3
该方法源码实现起来也非常简单，只是简单的字符串解析而已：
function get (obj, props, def) {
    if((obj == null) || obj == null || typeof props !== 'string') return def;
    const temp = props.split('.');
    const fieldArr = [].concat(temp);
    temp.forEach((e, i) => {
        if(/^(\w+)\[(\w+)\]$/.test(e)) {
            const matchs = e.match(/^(\w+)\[(\w+)\]$/);
            const field1 = matchs[1];
            const field2 = matchs[2];
            const index = fieldArr.indexOf(e);
            fieldArr.splice(index, 1, field1, field2);
        }
    })
    return fieldArr.reduce((pre, cur) => {
        const target = pre[cur] || def;
        if(target instanceof Array) {
            return [].concat(target);
        }
        if(target instanceof Object) {
            return Object.assign({}, target)
        }
        return target;
    }, obj)
}
// 使用
var c = {
     a: {
          b : [1,2,3] 
     }
}
_get(c ,'a.b')     // [1,2,3]
_get(c, 'a.b[1]')  // 2
_get(c, 'a.d', 12)  // 12


```

其实类似的问题如果用 typescript 编写会在静态检查的时候就处理掉了，可以避免此类低级错误

### 方法二，使用解构赋值

这个思路是来自 github 上 You-Dont-Need-Lodash-Underscore 这个仓库，看到这个的时候真的佩服

缺点是：可读性不太好

```
const obj = {
    a:{
        b: [1,2,3,4]
    },
    a1: 121,
    a2: 'name'
}
let {a: result} = obj     // result : {b: [1,2,3,4]}
let {a1: result} = obj   // result: 121
let {b: result} = obj     // result: undefined
当然，这个时候为了保证不报undefined，我们仍然需要定义默认值， 
let {b: result = 'default'} = obj    // result: 'default'


```

### 方法三，使用 Proxy

```
function pointer(obj, path = []) {
    return new Proxy(() => {}, {
        get (target, property) {
            return pointer(obj, path.concat(property))
        },
        apply (target, self, args) {
            let val = obj;
            let parent;
            for(let i = 0; i < path.length; i++) {
                if(val === null || val === undefined) break;
                parent = val;
                val = val[path[i]]    
            }
            if(val === null || val === undefined) {
                val = args[0]
            }
            return val;
        }
    })
}
使用方法：
var c = {
    a: {
        b : [1,2,3] 
    }
}
pointer(c).a();   // {b: [1,2,3]}
pointer(c).a.b(); // [1,2,3]
pointer(d).a.b.d('default value');  // default value


```

为啥 new Proxy() 第一个参数要是一个箭头函数，我看大部分都是一个对象

*   彦勋
    
*   你没理解 Proxy 工作的原理，如果是对象的话，下面的 apply 函数内的内容是不会执行到的 因为 proxy 要代理函数执行的行为才会去执行 apply 里面的函数，所以必须要传一个箭头函数
    

## 类型强制转换

###string 强制转换为数字

可以用 * 1（乘以 1）来转化为数字 (实际上是调用. valueOf 方法) 然后使用 Number.isNaN 来判断是否为 NaN，或者使用 a !== a 来判断是否为 NaN，因为 NaN !== NaN

```
'32' * 1            // 32
'ds' * 1            // NaN
null * 1            // 0
undefined * 1    // NaN
1  * { valueOf: ()=>'3' }        // 3


```

常用：也可以使用 + 来转化字符串为数字

```
+ '123'            // 123
+ 'ds'               // NaN
+ ''                    // 0
+ null              // 0
+ undefined    // NaN
+ { valueOf: ()=>'3' }    // 3


```

### “+ -” 符号的特技

用 + 可以**将数值型的字符串转为数字**，但是必须结合 isNaN(), isNaN() 函数用于检查其参数是否是非数字，不是数字则为 true

```
基础知识

let a = "2" ,b= '20',c=34 ,d='2'

a-b = -18

c-d = 32

a+b = "220"

a+c = "234"

isNaN('ad')   // true

isNaN('22')   // false

骚操作

+('ab')  // NaN

+('22')  // 22

+(22)    // 22

let a = 1 , b ='2'

let c = a + +(b)    //3

示例

// 选择最小值的函数如下（源于《vuejs权威指南》里看到表单验证validator.js源码）

export default  min (val, areg){

  return !isNaN(val) && !isNaN(areg) && +(val)>= +(areg)

}


```

使用 filter 过滤数组中的所有假值
--------------------

我们知道 JS 中有一些假值：false，null，0，""，undefined，NaN，怎样把数组中的假值快速过滤呢，可以使用 Boolean 构造函数来进行一次转换

```
const compact = arr => arr.filter(Boolean)
compact([0, 1, false, 2, '', 3, 'a', 'e' * 23, NaN, 's', 34])             // [ 1, 2, 3, 'a', 's', 34 ]
```


```

小符号特技
-----

### 双位运算符 ~~ 实现向下取整

可以使用双位操作符来替代 Math.floor( )。双否定位操作符的优势在于它执行相同的操作运行速度更快。

_math.floor_(x) 返回小于参数 x 的最大整数, 即对浮点数向下取整

```
Math.floor(4.9) === 4      //true
// 简写为：
~~4.9 === 4      //true
```


```

不过要注意，对负数来说 ~~ 运算结果与 Math.floor( ) 运算结果不相同：

```
```
~~4.5            // 4
Math.floor(4.5)        // 4
~~-4.5        // -4
Math.floor(-4.5)        // -5
```


```

### 短路运算符 &&、||

**|| 分配默认值**

```
let test1 = null,
    test2 = test1 || '6';

console.log(test2); // 输出为 "6"


```

我们知道逻辑与 && 与逻辑或 || 是短路运算符，短路运算符就是从左到右的运算中前者满足要求，就不再执行后者了；可以理解为：

*   && 为取假运算，从左到右依次判断，如果遇到一个假值，就返回假值，以后不再执行，否则返回最后一个真值
    
*   || 为取真运算，从左到右依次判断，如果遇到一个真值，就返回真值，以后不再执行，否则返回最后一个假值
    

```
let param1 = expr1 && expr2
let param2 = expr1 || expr2
```


```

<table data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)"><thead data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)"><tr data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: center;">运 算 符</th><th data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">示例</th><th data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">说明</th></tr></thead><tbody data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)"><tr data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">&amp;&amp;</td><td data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">expr1&amp;&amp;expr2</td><td data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">如果 expr1 能转换成 false 则返回 expr1, 否则返回 expr2. 因此, 在 Boolean 环境中使用时, 两个操作结果都为 true 时返回 true, 否则返回 false.</td></tr><tr data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;"><br data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(248, 248, 248)"></td><td data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">|</td><td data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">expr1</td></tr><tr data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px; text-align: center;">!</td><td data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">!expr</td><td data-darkmode-color-16245327025581="rgb(163, 163, 163)" data-darkmode-original-color-16245327025581="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16245327025581="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16245327025581="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">如果单个表达式能转换为 true 的话返回 false, 否则返回 true.</td></tr></tbody></table>

因此可以用来做很多有意思的事，比如给变量赋初值：

```
let variable1
let variable2 = variable1  || 'foo'


```

如果 variable1 是真值就直接返回了，后面短路就不会被返回了，如果为假值，则会返回后面的 foo。

也可以用来进行简单的判断，取代冗长的 if 语句：

```
let variable = param && param.prop


```

如果 param 如果为真值则返回 param.prop 属性，否则返回 param 这个假值，这样在某些地方防止 param 为 undefined 的时候还取其属性造成报错。

**但是要注意以下反例如果在数据为 0（或者空字符串的时候也要返回数据），此时用 || 运算符就会搞错了，返回的是默认值了：**

```
// 返回值为
let res = {
     data: 0,     // 后台返回状态码（0表示不存在，1表示存在）
     code: 200
}
let val = res.data || '无数据'
console.log(val)       // '无数据'，其实我们要的是data的值


```

### 取整 | 0

对一个数字 | 0 可以取整，负数也同样适用，num | 0

```
1.3 | 0         // 1
-1.9 | 0        // -1
```


```

### 判断奇偶数 & 1

对一个数字 & 1 可以判断奇偶数，负数也同样适用，num & 1

```
const num=3;
!!(num & 1)                    // true
!!(num % 2)                    // true
```


```

函数
--

### 函数默认值

```
func = (x, m = 3, n = 4 ) => (x * m * n);
func(2)             //output: 24
func(null)             //output: 12   因为1*null = 0


```

注意，传入参数为 undefined 或者不传入的时候会使用默认参数，即使**传入 null 还是会覆盖默认参数**。

这一点和 x = params || '2'是有区别的，|| 的取值时间是 params == false 时

### 强制参数, 缺失报错

默认情况下，如果不向函数参数传值，那么 JS 会将函数参数设置为 undefined。其它一些语言则会发出警告或错误。要执行参数分配，可以使用 if 语句抛出未定义的错误，或者可以利用强制参数。

```
logError = ( ) => {
  throw new Error('Missing parameter!');
}
foo = (bar = logError( ) ) => {     // 这里如果不传入参数，就会执行logError 函数报出错误(此处也可以添加日志打印等操作)
  return bar;
}


```

### 箭头函数隐式返回值

返回值是我们通常用来返回函数最终结果的关键字。只有一个语句的箭头函数，可以隐式返回结果（函数必须省略大括号 { }，以便省略返回关键字）。

要返回多行语句（例如对象文本），需要使用 () 而不是 {} 来包裹函数体。这样可以确保代码以单个语句的形式进行求值。

```
function calcCircumference(diameter) {
  return Math.PI * diameter
}
// 简写为：
calcCircumference = diameter => (
  Math.PI * diameter;
)


```

### 惰性载入函数

在某个场景下我们的函数中有判断语句，这个判断依据在整个项目运行期间一般不会变化，所以判断分支在整个项目运行期间只会运行某个特定分支，那么就可以考虑惰性载入函数

// 低级版

```
function foo(){
    if(a !== b){
        console.log('aaa')
    }else{
        console.log('bbb')
    }
}
 
// 优化后
function foo(){
    if(a != b){
        foo = function(){
            console.log('aaa')
        }
    }else{
        foo = function(){
            console.log('bbb')
        }
    }
    return foo();
}


```

那么第一次运行之后就会**覆写**这个方法，下一次再运行的时候就不会执行判断了。当然现在只有一个判断，如果判断很多，分支比较复杂，那么节约的资源还是可观的。

### 一次性函数

跟上面的惰性载入函数同理，可以在函数体里覆写当前函数，那么可以创建一个一次性的函数，重新赋值之前的代码相当于只运行了一次，适用于运行一些只需要执行一次的初始化代码

```
var sca = function() {
    console.log('msg')
    sca = function() {
        console.log('foo')
    }
}
sca()        // msg
sca()        // foo
sca()        // foo


```

字符串
---

### 字符串拼接用 join(), 避免使用 + 或 += 的方式拼接较长的字符串

应使用数组保存字符串片段，使用时调用 join 方法。避免使用 + 或 += 的方式拼接较长的字符串，每个字符串都会使用一个小的内存片段，过多的内存片段会影响性能

### 时间字符串比较 (时间形式注意补 0)

比较时间先后顺序可以使用字符串：

```
var a = "2014-08-08";
var b = "2014-09-09";
 
console.log(a>b, a<b); // false true
console.log("21:00"<"09:10");  // false
console.log("21:00"<"9:10");   // true   时间形式注意补0


```

因为字符串比较大小是按照字符串从左到右每个字符的 charCode 来的，但所以特别要注意时间形式注意补 0

数字
--

### 不同进制表示法

ES6 中新增了不同进制的书写格式，在后台传参的时候要注意这一点。

```
29            // 10进制
035            // 8进制29      原来的方式
0o35            // 8进制29      ES6的方式
0x1d            // 16进制29
0b11101            // 2进制29


```

### 精确到指定位数的小数

将数字四舍五入到指定的小数位数。使用 Math.round() 和模板字面量将数字四舍五入为指定的小数位数。省略第二个参数 decimals ，数字将被四舍五入到一个整数。

```
const round = (n, decimals = 0) => Number(`${Math.round(`${n}e${decimals}`)}e-${decimals}`)
round(1.345, 2)                 // 1.35   Number(1.345e2e-2)
round(1.345, 1)                 // 1.3

// 此处e2表示乘以10的2次方 
1.23e1   //12.3
1.23e2   // 123
123.45e-2  // 1.2345
```


```

### 数字补 0 操作

比如显示时间的时候有时候会需要把一位数字显示成两位，这时候就需要补 0 操作，可以使用 slice 和 string 的 padStart 方法

```
const addZero1 = (num, len = 2) => (`0${num}`).slice(-len)    // 从字符串倒数第len处开始截取到最后
const addZero2 = (num, len = 2) => (`${num}`).padStart( len   , '0')   // padStart为ES6的字符串拼接方法
addZero1(3) // 03
 
addZero2(32,4)  // 0032

此处可以采用另外一个思路；可以将得到的字符串先拼接len个0，然后再截取len长的字符串


```

## 数组

### 统计数组中相同项的个数

很多时候，你希望统计数组中重复出现项的个数然后用一个对象表示。那么你可以使用 reduce 方法处理这个数组。

下面的代码将统计每一种车的数目然后把总数用一个对象表示。

```
var cars = ['BMW','Benz', 'Benz', 'Tesla', 'BMW', 'Toyota'];
var carsObj = cars.reduce(function (obj, name) {
  obj[name] = obj[name] ? ++obj[name] : 1;    // obj[name]存在就加一，不存在就为1
  return obj;
}, {});
carsObj; // => { BMW: 2, Benz: 2, Tesla: 1, Toyota: 1 }


```

### 交换参数数值

有时候你会将函数返回的多个值放在一个数组里。我们可以使用数组解构来获取其中每一个值。

```
let param1 = 1;
let param2 = 2;
[param1, param2] = [param2, param1];
console.log(param1) // 2
console.log(param2) // 1


```

当然我们关于交换数值有不少其他办法：

```
var temp = a; a = b; b = temp            
b = [a, a = b][0]                     
a = a + b; b = a - b; a = a - b        
```


```

### 批量接收多个请求返回结果

在下面的代码中，我们从 / post 中获取一个帖子，然后在 / comments 中获取相关评论。由于我们使用的是 async/await，函数把返回值放在一个数组中。而我们使用数组解构后就可以把返回值直接赋给相应的变量。

```
async function getFullPost(){
  return await Promise.all([
     fetch('/post'),
     fetch('/comments')
  ]);
}
const [post, comments] = getFullPost();


```

### 将数组平铺到指定深度

*   基础的可以使用 **[].flat()** 方法来拉平单层数组, 代码如下
    

```
let a1 = [{a:1},{a:2},[{a:3},{a:4},[{a:5}]]]
a1.flat()  // [{a:1},{a:2},{a:3},{a:4},[{a:5}]]


```

*   使用递归，为每个深度级别 depth 递减 1 。使用 Array.reduce() 和 Array.concat() 来合并元素或数组。基本情况下，depth 等于 1 停止递归。省略第二个参数，depth 只能平铺到 1 (单层平铺) 的深度。
    

```
const flatten = (arr, depth = 1) =>
  depth != 1
    ? arr.reduce((a, v) => a.concat(Array.isArray(v) ? flatten(v, depth - 1) : v), [])
    : arr.reduce((a, v) => a.concat(v), []);
flatten([1, [2], 3, 4]);                             // [1, 2, 3, 4]
flatten([1, [2, [3, [4, 5], 6], 7], 8], 2);           // [1, 2, 3, [4, 5], 6, 7, 8]
```


```

### 数组的对象解构

数组也可以对象解构，可以方便的获取数组的第 n 个值

```
const csvFileLine = '1997,John Doe,US,john@doe.com,New York';
const { 2: country, 4: state } = csvFileLine.split(',');
 
country            // US
state            // New Yourk


```

### 对象数组按照某个属性查询最大值

```
var array=[
        {
            "index_id": 119,
            "area_id": "18335623",
            "name": "满意度",
            "value": "100"
        },
        {
            "index_id": 119,
            "area_id": "18335624",
            "name": "满意度",
            "value": "20"
        },
        {
            "index_id": 119,
            "area_id": "18335625",
            "name": "满意度",
            "value": "80"
        }];
        
 // 一行代码搞定
Math.max.apply(Math, array.map(function(o) {return o.value}))

执行以上一行代码可返还所要查询的array数组中对象value属性的最大值100。


```

同理，要查找最小值如下即可：Math.min.apply(Math, array.map(function(o) {return o.value})) 是不是比 for 循环方便了很多。

对象
--

### 使用解构删除对象某个属性

有时候你不希望保留某些对象属性，也许是因为它们包含敏感信息或仅仅是太大了。你可能会枚举整个对象然后删除它们，但实际上只需要简单的将这些无用属性赋值给变量，然后把想要保留的有用部分作为剩余参数就可以了。

下面的代码里，我们希望删除_internal 和 tooBig 参数。我们可以把它们赋值给 internal 和 tooBig 变量，然后在 cleanObject 中存储剩下的属性以备后用。

```
let {_internal, tooBig, ...cleanObject} = {el1: '1', _internal:"secret", tooBig:{}, el2: '2', el3: '3'};
 
console.log(cleanObject);    // {el1: '1', el2: '2', el3: '3'}



```

### 解构嵌套对象属性

在下面的代码中，engine 是对象 car 中嵌套的一个对象。如果我们对 engine 的 vin 属性感兴趣，使用解构赋值可以很轻松地得到它。

```
var car = {
  model: 'bmw 2018',
  engine: {
    v6: true,
    turbo: true,
    vin: 12345
  }
}
const modelAndVIN = ({model, engine: {vin}}) => {
  console.log(`model: ${model} vin: ${vin}`);
}
modelAndVIN(car); // => model: bmw 2018  vin: 12345


```

代码复用
----

### Object [key]

虽然将 foo.bar 写成 foo ['bar'] 是一种常见的做法，但是这种做法构成了编写可重用代码的基础。许多框架使用了这种方法，比如 element 的表单验证。

请考虑下面这个验证函数的简化示例：

```
function validate(values) {
  if(!values.first)
    return false;
  if(!values.last)
    return false;
  return true;
}
console.log(validate({first:'Bruce',last:'Wayne'})); // true


```

上面的函数完美的完成验证工作。但是当有很多表单，则需要应用验证，此时会有不同的字段和规则。如果可以构建一个在运行时配置的通用验证函数，会是一个好选择。

###Object [key] 实现表单验证

```
// object validation rules
const schema = {
  first: {
    required:true
  },
  last: {
    required:true
  }
}
 
// universal validation function
const validate = (schema, values) => {
  for(field in schema) {
    if(schema[field].required) {
      if(!values[field]) {
        return false;
      }
    }
  }
  return true;
}
console.log(validate(schema, {first:'Bruce'})); // false
console.log(validate(schema, {first:'Bruce',last:'Wayne'})); // true



```

### 保持函数单一职责，灵活组合

保持函数的单一职责，保证一个函数只执行一个动作，每个动作互不影响，可以自由组合，就可以提高代码的复用性。

比如下面的代码，从服务端请求回来的订单数据如下，需要进行以下三个处理：1. 根据 status 进行对应值得显示（0 - 进行中，1 - 已完成，2 - 订单异常） 2. 把 startTime 由时间戳显示成 yyyy-mm-dd 3. 如果字段值为空字符串 ，设置字段值为 ‘--’

**方案一：最基本的直接循环一次，同时操作三步**

```
let orderList=[
    {
        id:1,
        status:0,
        startTime:1538323200000,
    },
    {
        id:2,
        status:2,
        startTime:1538523200000,
    },
    {
        id:3,
        status:1,
        startTime:1538723200000,
    },
    {
        id:4,
        status:'',
        startTime:'',
    },
];
需求似乎很简单，代码也少
let _status={
    0:'进行中',
    1:'已完成',
    2:'订单异常'
}
orderList.forEach(item=>{
    //设置状态
    item.status=item.status.toString()?_status[item.status]:'';
    //设置时间
    item.startTime=item.startTime.toString()?new Date(item.startTime).toLocaleDateString().replace(/\//g,'-'):'';
    //设置--
    for(let key in item){
        if(item[key]===''){
            item[key]='--';
        }
    }
})


```

运行结果也正常，但是这样写代码重复性会很多，

比如下面，另一组服务端请求回来的用户数据，用户数据没有 status，startTime，两个字段，而需要根据 type 对应显示用户的身份（0 - 普通用户，1-vip，2 - 超级 vip）。

```
let userList=[
    {
        id:1,
        name:'守候',
        type:0
    },
    {
        id:2,
        name:'浪迹天涯',
        type:1
    },
    {
        id:3,
        name:'曾经',
        type:2
    }
]
出现这样的需求，之前写的代码无法重用，只能复制过来，再修改下。
let _type={
    0:'普通用户',
    1:'vip',
    2:'超级vip'
}
userList.forEach(item=>{
    //设置type
    item.type=item.type.toString()?_type[item.type]:'';
    //设置--
    for(let key in item){
        if(item[key]===''){
            item[key]='--';
        }
    }
})


```

结果正常，想必大家已经发现问题了，代码有点多余。下面就使用单一职责的原则改造下操作函数，设置 status，startTime，type，-- 。这里拆分成四个函数。

**方案二：单一职责，拆分为多个函数，导致循环遍历次数增加**

```
let handleFn={
    setStatus(list){
        let _status={
            0:'进行中',
            1:'已完成',
            2:'订单异常'
        }
        list.forEach(item=>{
            item.status=item.status.toString()?_status[item.status]:'';
        })
        return list
    },
    setStartTime(list){
        list.forEach(item=>{
            item.startTime=item.startTime.toString()?new Date(item.startTime).toLocaleDateString().replace(/\//g,'-'):'';
        })
        return list;
    },
    setInfo(list){
        list.forEach(item=>{
            for(let key in item){
                if(item[key]===''){
                    item[key]='--';
                }
            }
        })
        return list;
    },
    setType(list){
        let _type={
            0:'普通用户',
            1:'vip',
            2:'超级vip'
        }
        list.forEach(item=>{
            item.type=item.type.toString()?_type[item.type]:'';
        })
        return list;
    }
}
下面直接调用函数就好
//处理订单数据
orderList=handleFn.setStatus(orderList);
orderList=handleFn.setStartTime(orderList);
orderList=handleFn.setInfo(orderList);
console.log(orderList);
//处理用户数据
userList=handleFn.setType(userList);
userList=handleFn.setInfo(userList);
console.log(userList);


```

运行结果也正常, 但是性能上由于循环次数变多会牺牲一点性能，但是如果数据不多的话是可以接受的

如果嫌弃连续赋值麻烦，可以借用 jQuery 的那个思想，进行链式调用。**方案三：多次调用函数比较烦，此处采用链式调用**

```
let ec=(function () {
    let handle=function (obj) {
        //深拷贝对象
        this.obj=JSON.parse(JSON.stringify(obj));
    };
    handle.prototype={
        /**
         * @description 设置保密信息
         */
        setInfo(){
            this.obj.map(item=>{
                for(let key in item){
                    if(item[key]===''){
                        item[key]='--';
                    }
                }
            });
            return this;
        },
        /**
         * @description 设置状态
         */
        setStatus(){
            let _status={
                0:'进行中',
                1:'已完成',
                2:'订单异常'
            }
            this.obj.forEach(item=>{
                item.status=item.status.toString()?_status[item.status]:''
            });
            return this;
        },
        /**
         * @description 设置时间
         */
        setStartTime(){
               this.obj.forEach(item=>{
                item.startTime=item.startTime.toString()?new Date(item.startTime).toLocaleDateString().replace(/\//g,'-'):'';
            });
            return this;
        },
        /**
         * @description 设置type
         */
        setType(){
            let _type={
                0:'普通用户',
                1:'vip',
                2:'超级vip'
            }
            this.obj.forEach(item=>{
                item.type=item.type.toString()?_type[item.type]:'';
            })
            return this;
        },
        /**
         * @description 返回处理结果
         * @return {Array|*}
         */
        end(){
            return this.obj;
        }
    }
    //暴露构造函数接口
    return function (obj) {
        return new handle(obj);
    }
})();
这样就可以链式调用了
//处理订单数据
orderList=ec(orderList).setStatus().setStartTime().setInfo().end();
console.log(orderList);
//处理用户数据
userList=ec(userList).setType().end();
console.log(userList);



```

事情到这里了，相信大家发现一个很严重的问题就是循环的次数增加了。没优化之前，只需要循环一次，就可以把设置状态，设置时间，设置 -- 这些步骤都完成， 但是现在 setStatus().setStartTime().setInfo() 这里的代码，每执行一个函数，都遍历了一次数组。

**方案四：优化方案，在每一个函数里面，只记录要处理什么，但是不进行处理，等到执行到 end 的时候再统一处理，以及返回**

```
let orderList=[
  {
   id:1,
   status:0,
   startTime:1538323200000,
  },
  {
   id:2,
   status:2,
   startTime:1538523200000,
  },
  {
   id:3,
   status:1,
   startTime:1538723200000,
  },
  {
   id:4,
   status:'',
   startTime:'',
  },
 ];
let userList=[
  {
   id:1,
   name:'守候',
   type:0
  },
  {
   id:2,
   name:'浪迹天涯',
   type:1
  },
  {
   id:3,
   name:'曾经',
   type:2
  }
 ]

let ec=(function () {
  let handle=function (obj) {
   //深拷贝对象
   this.obj=JSON.parse(JSON.stringify(obj));
   //记录要处理的步骤
   this.handleFnList=[];
  };
  handle.prototype={
   /**
    * @description 设置保密信息
    */
   handleSetInfo(item){
    for(let key in item){
     if(item[key]===''){
      item[key]='--';
     }
    }
    return this;
   },
   setInfo(){
    this.handleFnList.push('handleSetInfo');
    return this;
   },
   /**
    * @description 设置状态
    */
   handleSetStatus(item){
    let _status={
     0:'进行中',
     1:'已完成',
     2:'订单异常'
    }
    item.status=item.status.toString()?_status[item.status]:''
    return item;
   },
   setStatus(){
    this.handleFnList.push('handleSetStatus');
    return this;
   },
   /**
    * @description 设置时间
    */
   handleSetStartTime(item){
    item.startTime=item.startTime.toString()?new Date(item.startTime).toLocaleDateString().replace(/\//g,'-'):'';
    return item;
   },
   setStartTime(){
    this.handleFnList.push('handleSetStartTime');
    return this;
   },
   /**
    * @description 设置type
    */
   handleSetType(item){
    let _type={
     0:'普通用户',
     1:'vip',
     2:'超级vip'
    }
    item.type=item.type.toString()?_type[item.type]:'';
    return item;
   },
   setType(){
    this.handleFnList.push('handleSetType');
    return this;
   },
   /**
    * @description 返回处理结果
    * @return {Array|*}
    */
   end(){
    //统一处理操作
    this.obj.forEach(item=>{
     this.handleFnList.forEach(fn=>{
      item=this[fn](item);    // 依次执行handleFnList队列中的函数，并传入item作为参数
     })
    })
    return this.obj;
   }
  }
  //暴露构造函数接口
  return function (obj) {
   return new handle(obj);
  }
 })();
    // 调用
 //处理订单数据
 orderList=ec(orderList).setStatus().setStartTime().setInfo().end();
 console.log(orderList);
 //处理用户数据
 userList=ec(userList).setType().end();
 console.log(userList);


```

推荐阅读  点击标题可跳转

1、[JavaScript 代码简洁之道](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651555684&idx=1&sn=537a63f2d119f460307cf89dc028e098&chksm=802550a5b752d9b3fc0e42d5b1230869b1f390a0bf8e3616ab4411b108393143eaa916ac21a1&scene=21#wechat_redirect)

2、[async/await 是如何让代码更加简洁的？](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651554816&idx=3&sn=4134fe58341f7db9cb7bc86c5056e6a4&chksm=802553c1b752dad713d5c95968bb6318ff8ff1d08fd63b9347a5144a9d8b613067c23be027a0&scene=21#wechat_redirect)

3、[代码简洁之道：编写干净的 React Components & JSX](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651575732&idx=2&sn=6a79d46b5db5973dfd383c33fae67898&chksm=80250275b7528b635c3c9f2c7942c2bfdbe7ea46824d36c65b51c00f8c84dc235a3ce12f69a2&scene=21#wechat_redirect)

觉得本文对你有帮助？请分享给更多人

公众号