> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7003642574405648420)

> 1. 使用数字分隔符 这是处理大数字时最常用的运算符之一。当在数字中使用分隔符（即_）时，看上去比未分隔的数字更美观。 使用前： 使用后： 并且这技巧也适用于其他进制的数值： 注意事项： 不能用在首位

1. 使用数字分隔符
----------

这是处理大数字时最常用的运算符之一。当在数字中使用分隔符（即`_`）时，看上去比未分隔的数字更美观。

使用前：

```
let number = 98234567;
复制代码
```

使用后：

```
let number = 98_234_567;
复制代码
```

并且这技巧也适用于其他进制的数值：

```
const binary = 0b1000_0101;
const hex = 0x12_34_56_78;
复制代码
```

注意事项：

不能用在首位 0 后面。

```
let num= 0_12; //错误
复制代码
```

不允许出现在数字的末尾。

```
let num= 500_;
复制代码
```

2. 总是使用分号
---------

使用分号作为代码行终止是一种很好的做法。当然如果你忘记写了，也不会被警告，因为在大多数情况下 JavaScript 解析器会插入分号，但不鼓励依赖 Automatic Semicolon Insertion(自动分号插入) 。

Google、Airbnb 和 jQuery 的 JavaScript 样式指南中也包含了这一点。

3. 不要忘记 var
-----------

当你第一次给一个变量赋值时，一定要确保你不是在给一个未声明的变量赋值。

给未声明的变量赋值会导致自动创建全局变量。注意不要轻易使用全局变量！！

全局变量很容易被其他脚本覆盖。例如，如果应用程序的两个独立部分定义了名称相同但用途不同的全局变量，则可能会导致不可预测的错误，你肯定不会想经历调试此类问题的痛苦。

所以，你最好限定代码的范围，以便尽可能少地用到全局作用域。你在脚本中使用的全局变量越多，那么与另一个脚本交互的难度就越大。

一般而言，函数中的变量应该是局部的，这样当你退出函数时它们就可以功成身退。

4. delete vs splice
-------------------

从数组中删除项，应该通过使用`splice`而不是`delete`。使用`delete`将删除对象属性，但不会重新索引数组或更新数组长度。这使得看起来就像未定义一样。

**Delete**

```
> myArray = ['a', 'b', 'c', 'd']
  ["a", "b", "c", "d"]
> delete myArray[0]
  true
> myArray[0]
  undefined
复制代码
```

> 请注意，它实际上并未设置为`undefined`值，而是从数组中删除了属性，从而使其看起来像`undefined`。Chrome 开发工具通过在记录数组时打印空来明确区分。

**Splice**

`Splice()`实际上会删除元素，重新索引数组，并更改数组长度。

```
> myArray = ['a', 'b', 'c', 'd']
  ["a", "b", "c", "d"]
> myArray.splice(0, 2)
  ["a", "b"]
> myArray
  ["c", "d"]
复制代码
```

`delete`方法应该用于删除对象属性。

5. map vs for 循环
----------------

使用`map()`函数方法循环遍历数组的项。

```
var squares = [1,2,3,4].map(function (val) {  
    return val * val;  
}); 

// squares will be equal to [1, 4, 9, 16]
复制代码
```

**不变性**——原始数组不受影响。以防万一其他地方仍然需要原始数组。也可以编写`for`循环以便不更新原始数组，但这需要更多代码并将更新新数组作为循环操作的一部分。另一方面，`map()`更简洁，因为你只需要工作于一个作用域内。

**更简洁的代码**——当做同样的事情时，用`map`写的代码几乎总是可以比`for`更少：有时甚至一行代码就可以，而`for`需要至少两行或者通常是三行代码，并且要用到花括号。此外，作用域隔离和所需变量数量的减少都使代码在客观上更简洁。

6. 四舍五入
-------

`toFixed()`方法将四舍五入的数字转换为指定的小数位数。

```
var pi =3.1415;
pi = pi.toFixed(2);  // pi will be equal to 3.14
复制代码
```

> 注意：`toFixed()`返回的是字符串而不是数字。

7. 使用 console.table
-------------------

你可以使用`console.table`以表格格式显示对象：

```
table=[{state: "Texas"},{state: "New York"},{state: "Chicago"}]
console.table(table)
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac0a0258d5c545568d5d0660e3d283ce~tplv-k3u1fbpfcp-watermark.awebp)

8. 避免在循环中使用 try-catch
---------------------

每次执行`catch`子句时，`try-catch`结构都会在运行时中给当前作用域创建一个新变量，其中捕获的异常对象将会被分配给这个变量。

在循环中使用`try-catch`：

```
var object = ['foo', 'bar'], i;  
for (i = 0, len = object.length; i <len; i++) {  
    try {  
        // do something that throws an exception 
    }  
    catch (e) {   
        // handle exception  
    } 
}
复制代码
```

改进后：

```
var object = ['foo', 'bar'], i;  
try { 
    for (i = 0, len = object.length; i <len; i++) {  
        // do something that throws an exception 
    } 
} 
catch (e) {   
    // handle exception  
}
复制代码
```

发生错误时，第一段代码继续循环，而第二段则退出循环。如果代码抛出的异常并非严重到要停止整个程序，则第一段代码更适合。

9. 多条件检查
--------

对于多值匹配，我们可以将所有值放在一个数组中，并使用`indexOf()`或`includes()`方法。

下面这段代码

```
if (value === 1 || value === 'one' || value === 2 || value === 'two') { 

} 
复制代码
```

可以改成

`indexOf()`：

```
if ([1, 'one', 2, 'two'].indexOf(value) >= 0) { 

}
复制代码
```

或`includes()`:

```
if ([1, 'one', 2, 'two'].includes(value)) { 

}
复制代码
```

10. 按位双非运算符 (~~)
----------------

按位双非运算符 (`~~`) 是`Math.floor()`方法的替代品。

```
const floor = Math.floor(6.8); // 6 
复制代码
```

等价于

```
const floor = ~~6.8; // 6
复制代码
```

按位双非运算符方法仅适用于 32 位整数。所以对于任何大于这个数值的数字，建议使用`Math.floor()`。

结论
--

不要使用随心所欲的编码风格。否则你永远不知道会发生什么。相同的编码风格应该始终贯穿整个团队和应用程序代码库，从而提高代码的可读性。