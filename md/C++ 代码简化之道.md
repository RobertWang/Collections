> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651163756&idx=1&sn=001c0829f32dbe5108e196a22892c953&chksm=80645d33b713d425db465faaa88d90bdb39cba9ce6ef475ad52f622c4c35214a8ec56cf33e0a&scene=21#wechat_redirect)

本文会介绍 10 个条款。

1. 善用 emplace
-------------

```
class Point {
public:
    Point(int x, int y):_x(x),_y(y){}
private:
    int _x;
    int _y;
};
```

C++11 之前。大概的写法

```
std::vector<Point> vp;
std::map<std::string, Point> mp;

Point p(1, 2);
vp.push_back(p);
vp.push_back(Pointer(3, 4));

Point p1(10, 20);
mp.insert(std::pair<std::string, Point>("key1", p1));
Point p2(100, 200);
mp.insert(std::make_pair("key2", p2));
```

C++11 之后：

```
std::vector<Point> vp;
std::map<std::string, Point> mp;

vp.emplace_back(1, 2);
vp.emplace_back(3, 4);

Point p1(10, 20);
Point p2(100, 200);
mp.emplace("key1", p1);
mp.emplace("key2", p2);
```

注意，其实也不需要无脑使用 emplace_back。比如，当你的使用场景中，已经确切存在了一个 Point 的对象，你需要把它放进 vector：

```
// 彼时，你已经有了一个Point的对象p。不需要自己凭空构造。
vp.push_back(p);
vp.emplace_back(p);
```

这种情况下，两种写法的表现几乎无差别（push_back 反而短…… 当然可能也没必要追求这个）。见过一些老项目升级 C++11 之后，无脑给 push_back 全替换成 emplace_back 的。虽然也没啥问题，但其实有时候没必要。

当然，当需要从参数来构造出对象的时候。那么 emplace_back 明显会简洁许多。但此时 push_back 其实除了代码冗长外，其性能开销也没有比 emplace_back 高太多，因为

```
vp.push_back(Pointer(3, 4));
```

调用的是：

```
void push_back (value_type&& val);
```

有较真的网友提到 emplace 的置入功能，还是要比这种 push_back (value_type&& val) 稍胜一筹，anyway。两个函数实现逻辑不同，肯定无法做到性能完全一致，但是也没到足以影响自己编码习惯的地步。总而言之当要放入 vector 的对象不存在的时候，直接用 emplace_back 来构造，在已存在的时候用 emplace_back 或 push_back 都可以。

2. 在不影响可读性的情况下使用 auto，区分 auto& 、auto&&
--------------------------------------

auto 不多解释了。

很多 C++ 程序员被问『熟悉 C++11 吗？说一说』

答一个『auto』

没啦

auto 就是用来简化长类型的（比如命名空间嵌套曾经很深）。另外 auto & 和 auto&&(万能引用）也不多解释了。

当然滥用 auto 也会造成代码可读性变差。在我等不用 IDE，用 vim 开发 C++ 的程序员面前，auto 滥用犹如噩梦。没有类型提示啊。

3. lambda 表达式替换手写函数和函数对象
------------------------

lambda 表达式（或者说 lamba 对象）可能是 C++ 程序员在回答『熟悉 C++11 吗？』这个问题，答完 auto 之后，说出的第二个新语法。

有了 lambda，STL 的 algorithm 里的函数，用起来更简洁了。

另外 lambda 除了替代了定义普通函数、函数对象（重载 operator()）之外，还有其他便利。那就是闭包的特性。说闭包可能一时难以理解。你就可以理解成是 lambda 的引用捕获功能。

在 lambda 的参数之外，获取到了其他的参数。并且是可跨越 lambda 生命周期的。

唯一需要注意的是：引用捕获可能在后续 lambda 对象被实际调用的时候，出现引用悬空（类似空指针），从而出现 core dump。

4. 给冗长的类型建立别名，尤其是 std::function 类型
----------------------------------

看一段冗长的代码。

```
class FuncFactory {
public:
    void put_func(std::string, std::function<std::vector<std::string>(std::string)>);
    std::function<std::vector<std::string>(std::string)> get_func(std::string);
private:
    std::unordered_map<std::string, std::function<std::vector<std::string>(std::string)>> _func_map;
};
```

用 using 简化掉：

```
using func_t = std::function<std::vector<std::string>(std::string)>;

class FuncFactory {
public:
    void put_func(std::string, func_t);
    func_t get_func(std::string);
private:
    std::unordered_map<std::string, func_t> _func_map;
};
```

5. 头文件中使用 #pragma once 替换老破旧 #ifndef #define #endif
---------------------------------------------------

从上个世纪 70 年代 C 语言诞生之始，头文件都在使用 #ifndef #define #endif 来避免重复包含。

```
#ifndef HEADER_FILE
#define HEADER_FILE

...
#endif
```

C++ 也继承了这种写法。然而时至今日还可以这样写：

```
#pragma once
...
```

这个语法很久之前就有，但并非是 C++ 标准的一部分。但在很多编译器厂商的实现中，早早地支持了这种语法。C++11 中这个语法依旧没有转正，但是由于被编译器广泛支持，几乎可以放心使用了。在 Google 和 Facebook 的 C++ 开源项目中都有大量使用。#ifndef #define #endif 终于寿终正寝。

当然在个别情况下，这个语法也存在坑：

> 不同于头文件防护，这条语用使得错误地在多个文件中使用相同的宏名变得不可能。另一方面，因为带 #pragma once 的文件是基于其文件系统层次的身份所排除的，所以若头文件在项目中有多个位置，则这不能防止包含它两次。

可以参考：https://zh.cppreference.com/w/cpp/preprocessor/impl

简而言之，pragma 是基于头文件的文件路径来保持唯一的。而宏可以做到跨多个文件来保持 include 的唯一性。比如当你一个代码库中存在一个头文件的多个版本……

一般情况下，我们可能很少在一个项目中需要用到一个头文件的多个版本，反正我是没这种需求。

6. 善用 for range 遍历容器，也可以针对 PB 的 repeated 字段（甚至 mutable）
-------------------------------------------------------

还在用下标遍历容器吗？

```
for (int i = 0; i < v.size(); ++i) {
    cout<<v[i]<<endl;
    v[i] *= 10;
}
```

java 和其他语言早有不借助下标的 for - range 循环，C++11 也有了：

```
for (auto& e: v) {
    cout<<e<<endl;
    e *= 10;
}
```

最好用引用 & 来遍历，否则如果容器中存储的是对象，会出现拷贝。当然如果你不想修改容器内元素的话，也可以用 const auto& 遍历。

C++ 工程项目中，protobuf 肯定是会大量使用的。for range 也可以遍历 pb 的 repeated 字段

```
syntax = "proto3";
message Student {
    string name = 1;
    int32 score = 2;
}
message Report {
    repeated Student student = 1；
}
```

代码中：

```
// report 是一个Report类型的对象
for (auto& student: report.student()) {
    cout<< student.name << "'s score:" << student.score << endl;
}
```

工作中看多很多遍历 pb repeated 字段代码大多可以做到上面那样。但是当遍历 pb repeated 字段并修改其中变量的时候（mutable 返回的是指针，不能直接 for range），很多人还是选择了用传统的 for + 下标的形式来遍历。其实不用，依旧可以 for range

```
for (int i = 0; i < report.student_size(); ++i) {
    report.mutable_student(i)->set_score(60); // 60分万岁！
}
```

啰嗦！！！！！！！！！！！可以这样写：

```
for (auot& student: *report.mutable_student()) {
    student.set_score(60);  // 60分万岁！
}
```

7. 用 do while 或 IIFE 跳过部分连续逻辑，但不结束函数
------------------------------------

你有没有这种体验：在函数中一段平铺的逻辑中，依次经历 1，2，3 三个步骤，然后是其他逻辑（比如 4，5）。其中 1，如果失败就不执行 2，2 如果失败不执行 3。就是逻辑中断之后直接跳到 4 和 5。容易想到的实现思路有三：

其一：把步骤 1，2，3 抽象成函数。每次判断函数的返回值，成功才调用下一个函数。OK。这样没问题。但是如果顺序逻辑太多。那么要抽成很多个函数，而且每个函数内只有寥寥几行代码。反而啰嗦。

其二：使用异常。如果是 Java 语言应该很习惯用异常来实现这个逻辑，把顺序逻辑封在 try catch 块里。每个步骤失败直接 throw 异常。OK，C++ 也可以写类似的代码。然而 C++ 用异常隐患很多，不如 Java 安全，很多工程规范都竭力避免抛异常。另外就是抛异常也不是无开销的，而且这里只是逻辑中断，逻辑上也不算『异常』，通过 throw 异常和 catch 异常的方式未免更加影响表现力……

其三：`goto`。看过一些代码确实在这种场合使用过 goto。当然我们要严厉禁止 goto。这个方案直接略过。

其实还有第 4 种方案：do while(0)

```
do {
    // 步骤1
    ...
    if (步骤1失败) {
        break;
    }
    // 步骤2
    ...
    if (步骤2失败) {
        break;
    }
    // 步骤3
    ...
    if (步骤3失败) {
        break;
    }
} while(0);

// 步骤4
...
// 步骤5
...
```

这个其实也适用于其他有 do while 的语言，不止 C++。另外由于 C++11 中 lambda 函数的出现，你还可以这样写：

```
[]() {
    // 步骤1
    ...
    if (步骤1失败) {
        return;
    }
    // 步骤2
    ...
    if (步骤2失败) {
        return;
    }
    // 步骤3
    ...
    if (步骤3失败) {
        return;
    }
}();

// 步骤4
...
// 步骤5
...
```

这个是在普通 lambda 表达式的末尾加上了一个括号，也就是让定义的 lambda 可以立即执行。

这一特性也被人称为`IIFE`（Immediately Invoked Function Expression），即立即调用函数表达式。这是一个出自 Javascript 的术语，可能不是 C++ 中的正统称呼……

8. 某些情况下用 struct 替代 class，避免把 C++ 类写成 JavaBean
----------------------------------------------

因为种种原因，从 Java 转 C++ 的程序员，喜欢把 C++ 的类写成 JavaBean。动不动就 set()、get()

9. 函数直接返回 STL 容器或对象。不要返回指针，也不需要给函数加出参
-------------------------------------

第一种：

```
void split(std::string str, std:string del, std::vector<std::string>& str_list) {
   // 解析字符串str，按del分隔符分割，拆成小字符串存入str_list中
   ...
}

// 调用方：
std::vector<std::string> str_list;
split("a:b:c:d", ":", str_list);
```

这种用的时候不太方便。如果不是 split，而且其他例子。我可能想一行连续点点点调用返回值的成员变量（foo().bar().xxx()）。无疑，上面这种会中断我的一行语句写法。

第二种：

```
std::shard_ptr<std::vector<string>> split(std::string str, std:string del) {
    std::shard_ptr<std::vector<string>> p_str_list = std::make_shared<std::vector<std::string>>();
    // 解析字符串str，按del分隔符分割，拆成小字符串存入p_str_list中
    ...
    return p_str_list;
}
```

或者最原始版本:

```
std::vector<std::string>* split(std::string str, std:string del) {
    std::vector<std::string>* p_str_list = new std::vector<std::string>;
    // 解析字符串str，按del分隔符分割，拆成小字符串存入p_str_list中
    ...
    return p_str_list;
}
需要小心的处理返回值，自己控制delete掉指针，避免内存泄露。
```

都太啰嗦。但无一例外。熟悉 C++98 的老前辈们都不会建议你用函数直接返回 STL 容器。然而事情从 C++11 开始起了变化。那些不熟悉 C++98 的新手程序员们反而写出来最优解：

```
std::vector<std::string> split(std::string str, std:string del) {
    std::vector<std::string> str_list;
    // ...
    return str_list;
}
```

相信我，没问题。

这个变化，其实也在工作中造成一些尴尬。有时候我写这种代码，在给老同事过 core review 的时候，生怕被批一顿代码写的烂。如果被批一顿，我自然尴尬，然后我解释一番这种写法在 C++11 里面没问题，那么老同事就尴尬了。

为避免这种尴尬我总会在代码附近加个注释:

```
// it's ok in C++11
std::vector<std::string> split(std::string str, std:string del);
```

其实 C++11 之前也有这么用的。因为编译器自己做的 RVO，NRVO 优化，这当然是非标的。改一下编译选项可能就没啦。虽然 gcc 不显式关闭 RVO 的话，默认就开始的。但曾经我在 C++98 的环境下工作时，还是很少见到这种直接返回对象的写法。其实不是所有返回对象函数定义都能触发 RVO，如果不清楚，C++98 的程序员还是谨慎使用。

但是 C++11 开始，你不用担心了。

10. 利用 unordered_map/map 的 [] 运算符的默认行为
--------------------------------------

比如我们程序中有一个计数逻辑，使用了一个 unordered_map<string, int>（或 map<string, int>) 来对某个 string 类型的 tag 进行计数。之前看到有同事这样写：

```
// freq_map 是一个 unordered_map<string, int> 类型。
// 通过某个计算获取到了一个string类型的变量tag，下面进行计数
if (freq_map.find(tag) == freq_map.end()) {
    frea_map.emplace(tag, 1);
} else {
    freq_map[tag] += 1;
}

// 或者这种
if (freq_map.find(tag) == freq_map.end()) {
    frea_map.emplace(tag, 0);
}
freq_map[tag] += 1;
```

其实通通不用，上述两种大概是 python 中用 dict 来计数的写法（当年我写 MapReduce 任务的时候也有类似的写法）但是 C++ 不用，因为。C++ 的 map 在使用 [] 运算符的时候会在 key 不存在的时候默认创建出一个值！如果 value 是基本数据类型，那么就是 0。

所以可以直接写：

```
frep_map[tag]++;
// 或
freq_map[tag] += 1;
```

当然也正因为 [] 运算符的这个默认性质所以 Effective C++ 里面才有一条说要用 m.insert() 来插入 key，value(C++11 之后用 emplace) 而不要用 m[key] = value 的写法，因为后者会先构造一个空对象，再覆盖掉它。当然具体到我这里提到这个计数场景，不需要考虑这个。因为本来就需要在 key 不存在的时候初始化一个，而且 value 是基本数据类型，初始化成 0，然后覆盖成 1，开销不大。

- EOF -

推荐阅读  点击标题可跳转

1、[C++ 常见的三种内存破坏场景和分析](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651163670&idx=2&sn=554ef639de23afdb02fd04fe71dffd88&chksm=80645d49b713d45f843f455ba16d4efb8c20302a0cc2868238d98bb83942081dc44fcc32b4c7&scene=21#wechat_redirect)

2、[从四个问题透析 Linux 下 C++ 编译 & 链接](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651163344&idx=1&sn=a36c01c3d005f3fc6dd0fac00521caa7&chksm=80645b8fb713d299b7fb1e1875f61bda1193ec182e994961155a9d8a4401775ee274e16a9fbd&scene=21#wechat_redirect)

3、[如果你看完这篇文章还不懂计算机时间，那就掐死我吧](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651163408&idx=1&sn=27211d6cf7e84b0a504c241fdfb1e4f0&chksm=80645a4fb713d359f6bc3f4f4016d7cb8ca2d5cb6708078f864d35d4fafdab652989f6ed0386&scene=21#wechat_redirect)