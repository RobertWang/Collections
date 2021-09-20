> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7001529656188862494)

创作不易，求一个免费的赞，谢谢啦 ！！！

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8c5064b20204a1a99d30ac93730ac0c~tplv-k3u1fbpfcp-watermark.awebp)

本文已参与掘金创作者训练营第三期，详情查看：[掘力计划｜创作者训练营第三期正在进行，「写」出个人影响力](https://juejin.cn/post/6994417198164869133 "https://juejin.cn/post/6994417198164869133")。

前言
--

为啥会突然想起写一篇关于 console 的文章？笔者接触 JS 也不少时间了，除了用 vscode 的 debuger，其实大部分时间都在使用 console.log() 方法来输出一些或者调试程序，我相信很多刚开始接触 JS 的同志，应该也都习惯使用 console.log()。但是 log 的能力是有限的，并不能满足所有的场景。比如我们相用表格数据对象。

下面这张图，纯粹是看了扫黑风暴想到的！！！

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07e58e60cba941cabd57700e18c17cfe~tplv-k3u1fbpfcp-watermark.awebp)

console.log()
-------------

大家最常用的应该就是这个属性了，不过你有没有使用这个方法输出 console 对象。

```
console.log(console)
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59ef00e79ed542bcb5619bcdc00cf2d0~tplv-k3u1fbpfcp-watermark.awebp)

语法
--

### console.log(常见 JS 数据)

```
console.log(123)
>> 123
复制代码
```

### console.log(%s %d %f %o 等占位符写法（类似 C 的 print）)

```
console.log('我是 %s','前端picker')
>> 我是 前端picker
复制代码
```

### console.log(ES6 模板字符串)

```
const nickName = "前端picker"
console.log(`我是 ${nickName}``)
>> 我是 前端picker
复制代码
```

### 数组 / 对象会显示在一行

```
console.log({object: 'object'}, {object: 'object'});
console.log(['array', 'array'], ['array', 'array']);
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01d4ee1d37364feb90425b33d85ee2c6~tplv-k3u1fbpfcp-watermark.awebp)

### CSS 样式美化输出

在上面我们介绍了占位符输出，其实还有占位符 %c, 可以用来接收 css 样式。

```
console.log('我是红色 %c 文字', 'color: red;');
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1ab015fdfe64874acb125e3e4701ec1~tplv-k3u1fbpfcp-watermark.awebp)

每个 %c 只负责它之后的文字，知道遇到下一个 %c;

```
console.log('This is %cred text %c 我没颜色 %c 我是绿色.', 'color: red;', '', 'color: green;');
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03553f49d41e460b85d49364d13715cf~tplv-k3u1fbpfcp-watermark.awebp)

当然你也可以选择把样式定义成变量。

```
let style="background: white;border: 3px solid red;color: red;font-size: 50px;margin: 40px;padding: 20px;"
undefined
console.log('%c我是log!', style);
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39214dbb686a457fbe4bddab15dc9e9f~tplv-k3u1fbpfcp-watermark.awebp)

console.clear()
---------------

大部分的浏览器在开发者工具都内置了清除控制条的方法。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ec14ee42b674695b351f2bbb1eba673~tplv-k3u1fbpfcp-watermark.awebp)

其实 console 对象也提供了 clear 方法来清空控制台。

当你执行 console.clear() 之后

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf0c3ad41c354cb19df9c238372477c7~tplv-k3u1fbpfcp-watermark.awebp)

cosole.debug()
--------------

输出 “调试” 级别的消息, 并且在浏览器中只有你配置了 debug 才可以显示。 例如在火狐浏览器中：

当你没有勾选调试的时候：是无法显示 debug 的信息的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3add7562733a4081b2fae333776ffe78~tplv-k3u1fbpfcp-watermark.awebp) 只有你勾选调试的时候，才会显示。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b866515630c0479b8bf232febe210981~tplv-k3u1fbpfcp-watermark.awebp)

error()
-------

向 Web 控制台输出一条错误消息，并且只有才浏览器配置了 errors 才可以显示。

下图是在火狐浏览器的效果 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f728ce286cf54ed19fc3b1a6f47338d0~tplv-k3u1fbpfcp-watermark.awebp) 下图是在 chrome 的效果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/68007d61188d4099a38457993d8d9f87~tplv-k3u1fbpfcp-watermark.awebp)

两个浏览器都会在 error 的前面加上一个小图标。

info()
------

向 web 控制台输出一个通知信息，并且只有在浏览器配置 info 可见的时可以显示。 下图是在火狐浏览器的效果 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3004b579582b43c6805085f116a35575~tplv-k3u1fbpfcp-watermark.awebp) 下图是在 chrome 的效果 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a65485a59974a96940ac7eb2926cbfe~tplv-k3u1fbpfcp-watermark.awebp) 在火狐浏览器中会在前面加上小图标，而 chrome 没有

warn()
------

向 Web 控制台输出一条警告信息，并且只有在浏览器配置 warning 可见的时可以显示。

下图是在火狐浏览器的效果 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dadf966be6f45788ec1ba87fd6dcd03~tplv-k3u1fbpfcp-watermark.awebp) 下图是在 chrome 的效果 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbd965dfa7ae42fcb2538d445c22b152~tplv-k3u1fbpfcp-watermark.awebp) 两个浏览器都会在 warn 的前面加上一个小图标。

console.count()
---------------

输出 count() 被调用的次数。此函数接受一个可选参数 label，如果你不设置参数的话，这个参数默认名叫 **“default”**。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/daec57ce1cbf4359a4e2c916e3beeeb6~tplv-k3u1fbpfcp-watermark.awebp)

### 用来统计

```
console.count('我是');
console.count('前端picker');
console.count('我是');
console.count('前端picker');
console.count('我是');
console.count('前端picker');
复制代码
```

通过下图可以看到，针对不同的参数，count() 是分别累计的。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2dd72ae9ec14538a75b2a480a1cd3e6~tplv-k3u1fbpfcp-watermark.awebp)

console.countReset()
--------------------

用来重启计数器的. 同样也接收一个 label 参数，

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/252f4fc0bbcb4c9993b850fd1f8f35a4~tplv-k3u1fbpfcp-watermark.awebp) 如果提供了参数 label，此函数会重置与 label 关联的计数。

如果省略了参数 label，此函数会重置默认的 default 计数器。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24f4ca12c88c4a1093bdee623e30cc42~tplv-k3u1fbpfcp-watermark.awebp)

console.dir()
-------------

在控制台中显示指定 JavaScript 对象的属性，并通过类似文件树样式的交互列表显示。 也就是语法是:

```
console.dir(object)
复制代码
```

```
const auther = {
  name: '前端picker',
  age: '18',

};

console.log(auther);
console.dir(auther);
复制代码
```

在 chrome 浏览器中，是支持这个属性的，下图可以看出与 log 的不同

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0e5de72d6864dfa8004ffe681a503db~tplv-k3u1fbpfcp-watermark.awebp)

但是在火狐浏览器中，log 和 dir 的输出一致，不同的是火狐会默认展开 dir 的结果。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44d77a9aa03147f3837e61beb9e79b73~tplv-k3u1fbpfcp-watermark.awebp)

当然 log 和 dir 在输出 dom 结构的时候是完全不同的。不过这个我们放在 dirxml 方法中学习。

console.dirxml()
----------------

显示一个明确的 XML/HTML 元素的包括所有后代元素的交互树。 如果无法作为一个 element 被显示，那么会以 JavaScript 对象的形式作为替代。 它的输出是一个继承的扩展的节点列表，可以让你看到子节点的内容。

同时也支持 object。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8654929248aa46b382810cee886ceebc~tplv-k3u1fbpfcp-watermark.awebp)

在 dir 中我们把 DOM 留到了这里。 创建一个 DOM 对象

```
var childNode = document.createElement('p');
childNode.innerHTML = '<span>这里是提示信息〜〜</span>';
复制代码
```

使用 log/dir/dirxml 输出

log

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc774b1722e947c384a21214c464a111~tplv-k3u1fbpfcp-watermark.awebp) dir ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae5296bdee8a437e941a9ec4c2811db2~tplv-k3u1fbpfcp-watermark.awebp) dirxml ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3792cb570ed4b74ab1ea78bc176ff40~tplv-k3u1fbpfcp-watermark.awebp)

可以看出 log 和 dirxml 的输出效果是一致的，是以 DOM 的形式输出的，dir 则输出的是属性的值。

group()、groupCollapsed() 和 groupEnd()
-------------------------------------

将不同的控制台输出组合在一起以显示它们之间的一种关系形式。通常是组合在一起使用的。

group 接收一个参数，表示组名，如果不设置的话，在不同的浏览器表现不一致，浏览器默认展开组。

groupCollapsed() 与 group 的不同在于 groupCollapsed 会自动折叠组。

groupEnd() 表示组的结束。

### group+groupEnd

```
console.group();
console.log('name:');
console.log('前端picker');
console.groupEnd();

console.group('参数');
console.log('age');
console.log('18');
console.groupEnd();
复制代码
```

下面是 chrome 的效果。不设置组名的话，默认是 console.group，同时一组的输出会缩进。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66a61df8e01242f183ee80b0c7e87269~tplv-k3u1fbpfcp-watermark.awebp) 下面是火狐的效果不设置组名的话，默认是 <无组标签> ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83459e933fb64fe38f68e6465b39699e~tplv-k3u1fbpfcp-watermark.awebp)

### groupCollapsed+grounEnd

```
console.groupCollapsed();
console.log('name:');
console.log('前端picker');
console.groupEnd();

console.group('参数');
console.log('age');
console.log('18');
console.groupEnd('参数');
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da9b204280114e6fa2f4ce0d44af5364~tplv-k3u1fbpfcp-watermark.awebp)

### 嵌套

```
console.group('下面是作者信息');
console.log('第1项');
console.group('name');
console.log('前端picker');
console.groupEnd();
console.log('第2项');
console.group('age');
console.log('18');
console.groupEnd();
console.groupEnd();
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aaa132c9b3094b7b96d271e4fe7107d4~tplv-k3u1fbpfcp-watermark.awebp)

### CSS 样式美化 -%c

```
console.group('%c下面是作者信息','color:red;');
console.log('第1项');
console.group('name');
console.log('前端picker');
console.groupEnd();
console.log('第2项');
console.group('age');
console.log('18');
console.groupEnd();
console.groupEnd();
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e1d32f6954c4f8e925bb34abe29b990~tplv-k3u1fbpfcp-watermark.awebp)

console.table()
---------------

这个方法需要一个必须参数，参数 必须是一个数组或者是一个对象；还可以使用一个可选参数 columns。

它会参数以表格的形式打印出来。数组中的每一个元素（或对象中可枚举的属性）将会以行的形式显示在表格中。

表格的第一列是 index。如果数据 data 是一个数组，那么这一列的单元格的值就是数组的索引。 如果数据是一个对象。

### 数组

```
let ary = [
  '1',
  '2',
  '3'
];
console.table(ary);
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a7267f271e2428c924c4ae45ef25faa~tplv-k3u1fbpfcp-watermark.awebp)

### object 对象

```
let obj = {
  name: '前端picker',
  age: '18',
};
console.table(obj);
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a446c5c3f8b8484eb187d36b0f69453e~tplv-k3u1fbpfcp-watermark.awebp)

### 对象数组

```
let aryOfObjects = [
  {
    name: '张三',
    age: '12',

  },
  {
    name: '李四',
    age: '18',

  },
  {
    name: '王五',
    age: '19',
  }
];
console.table(aryOfObjects);
复制代码
```

如图所示，table() 为我们提供了一个很好的对象布局，其中重复键作为列标签。，每个对象中的所有键都将表示为一列，无论其他对象中是否有对应的键与数据。如果对象没有键列的数据，则它显示为空。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0c75d2a34984a359d8c4e42f375bd23~tplv-k3u1fbpfcp-watermark.awebp)

### 数组数组

```
let aryOfArray = [
[1,2,3],[3,4,5],[6,7,8]
];
console.table(aryOfArray);
复制代码
```

数组数组类似于对象数组。它使用内部数组的索引作为列标签，而不是作为列标签的键。因此，如果一个数组的项目数比其他数组多，那么这些列的表中将有空白项目。就像对象数组一样。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a59a876bccb6415b9cf888447026dbed~tplv-k3u1fbpfcp-watermark.awebp)

### 对象数组数组

```
let aryOfArraysWithObject = [
  ['1', '2',  {
    name: '张三',
    age: '12',

  },],
  ['3', '4',  {
    name: '李四',
    age: '18',

  },}],
  ['5', '6',  {
    name: '王五',
    age: '19',
  }]
];

console.table(aryOfArraysWithObject);
复制代码
```

在 Chrome 中，要查看第三列中这些对象中包含的内容，是无法展开的，也就是无法查看。。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46aef873916a48b789d112b29a43b3f9~tplv-k3u1fbpfcp-watermark.awebp) 不过在火狐浏览器中，会自动展开，可以清除的看到结果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72620ca0083843219ef993ae49150000~tplv-k3u1fbpfcp-watermark.awebp)

time()、timeLog() 和 timeEnd()
----------------------------

**time()**

你可以启动一个计时器来跟踪某一个操作的占用时长。每一个计时器必须拥有唯一的名字，页面中最多能同时运行 10,000 个计时器。当以此计时器名字为参数调用 console.timeEnd() 时，浏览器将以毫秒为单位，输出对应计时器所经过的时间。

```
console.time(计时器名称)
复制代码
```

**timeEnd()**

停止一个通过 console.time() 启动的计时器, 并并输出结束的时间

```
console.timeEnd(计时器名称)
复制代码
```

**timeLog()**

在控制台输出计时器的值，该计时器必须已经通过 console.time() 启动。

```
console.timeLog()(计时器名称)
复制代码
```

### 使用

```
console.time('this is a timer');
console.timeLog('this is a timer');
console.timeEnd('this is a timer');
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0437f8baa254ec1af20fdb6b795879a~tplv-k3u1fbpfcp-watermark.awebp)

### 计算 for 的时间

```
console.time('this is a timer');
for(let  i=0; i<10000000;i++){
    
}
console.timeLog('this is a timer');
console.timeEnd('this is a timer');
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88deff0a186a4d499d5f0acb384e9c9c~tplv-k3u1fbpfcp-watermark.awebp)

### console.assert()

console.assert() 命令类似于前面提到的错误命令。不同之处在于断言允许使用布尔条件来确定是否应该将文本输出到控制台。

例如：你想测试一个变量的值，并且这个值不等于'前端 picker'， 如果变量低于该数字并且条件解析为 true，则 assert 命令不执行任何操作。如果条件解析为 false，则显示输出文本。通过这种方式，你就不需要通过 if 判断是不是需要输出之后，再使用 console.error() 输出。

```
let name = '张三';
console.assert(name === '前端picker', '不是是前端picker，无法输出');
let name1 = '前端picker';
console.assert(name1 === '前端picker', '不是前端picker------1，可以输出');
复制代码
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d54066668d6d40e4aa12995d739e5f15~tplv-k3u1fbpfcp-watermark.awebp)

在 Chrome 中， 还可以输出显示断言来自何处。

```
let value = 2;
function chackValue() {
    chackValue3();
}
function chackValue2() {
    chackValue3();
}
function chackValue3() {
  console.assert(value < 1, 'This was false.');
}
chackValue();
复制代码
```

为了方便看，我们使用 vscode 标注了行数。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5aacd590af174c2982d09785b1b01b30~tplv-k3u1fbpfcp-watermark.awebp)

通过下图可以看出，Chrome 告诉我们断言在 9 行。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1c4355706ba46378a53910ca5a82d58~tplv-k3u1fbpfcp-watermark.awebp)

console.trace
-------------

### 调用堆栈

在学习 trace 之前我们先来学习什么是调用堆栈！！！ 有这样四个函数 function1 调用 function2 ，function2 调用 function3，function3 调用 function4 看这张图，这几个函数呈现出堆栈的特征。最后被调用的函数出现在最上方。因此称呼这种关系为调用堆栈 (call stack)。 ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/714d5740b60c448ba0b98742185a5a74~tplv-k3u1fbpfcp-watermark.awebp)

### trace 可以干啥

用来记录 JavaScript 堆栈跟踪, 同时我们还还可以添加参数，用来表示当前跟踪的名册灰姑娘。

```
function f1(){
     console.log('f1')
     f2()
}
function f2(){
     console.log('f2')
     f3()
}
function f3(){
     console.log('f3')
     f4()
}
function f4(){
     console.log('f4')
     console.trace("f4的追踪记录");
}
复制代码
```

从图中我们可以看到：触发 console.trace 之前调用的最后一个函数是 f4。所以这将是调用堆栈的顶部。然后一次是 f3,f2 ,f1 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/489b69fb132e4e33a8ad0c5dc6d21180~tplv-k3u1fbpfcp-watermark.awebp)

后记
--

**下面这句话是在掘金网站的某个隐蔽的地方看到的，大家可以评论区猜猜！！！！**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a971aa5c3b34db4996c3446329a416c~tplv-k3u1fbpfcp-watermark.awebp)