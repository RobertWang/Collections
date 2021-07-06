> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/TenosDoIt/p/3203137.html)

**【目录】**

[**lib dll 介绍**](#a)

[**生成动态库**](#b)

[**调用动态库**](#c)

[**生成静态库**](#d)

[**调用静态库**](#e)

首先介绍一下静态库（静态链接库）、动态库（动态链接库）的概念，首先两者都是代码共享的方式。

**静态库**：在链接步骤中，连接器将从库文件取得所需的代码，复制到生成的可执行文件中，这种库称为静态库，其特点是可执行文件中包含了库代码的一份完整拷贝；缺点就是被多次使用就会有多份冗余拷贝。即静态库中的指令都全部被直接包含在最终生成的 EXE 文件中了。在 vs 中新建生成静态库的工程，编译生成成功后，只产生一个. lib 文件

**动态库**：动态链接库是一个包含可由多个程序同时使用的代码和数据的库，DLL 不是可执行文件。动态链接提供了一种方法，使进程可以调用不属于其可执行代码的函数。函数的可执行代码位于一个 DLL 中，该 DLL 包含一个或多个已被编译、链接并与使用它们的进程分开存储的函数。在 vs 中新建生成动态库的工程，编译成功后，产生一个. lib 文件和一个. dll 文件

那么上述静态库和动态库中的 lib 有什么区别呢？

**静态库中的 lib**：该 LIB 包含函数代码本身（即包括函数的索引，也包括实现），在编译时直接将代码加入程序当中

**动态库中的 lib**：该 LIB 包含了函数所在的 DLL 文件和文件中函数位置的信息（索引），函数实现代码由运行时加载在进程空间中的 DLL 提供

**总之，lib 是编译时用到的，dll 是运行时用到的。如果要完成源代码的编译，只需要 lib；如果要使动态链接的程序运行起来，只需要 dll**。

以下例子均在 vs2010 上测试

**生成和使用动态库**

**生成动态库**

 新建项目 --win32 项目 -- 填写项目名 -- 确定 -- 下一步 -- 应用程序类型：选择 dll-- 附加选项：选择导出符号 -- 完成

可以看到生成了一个 dllmain.cpp 文件，这是 dll 应用程序的入口，注意它和普通工程的入口 main 函数不同，这个文件我们不需要修改。

 在这个动态库中我们举例导出一个变量，一个类，一个函数，头文件 dll.h 如下：

```
 1 //新建生成dll的工程时，vs默认定义了宏DLL_EXPORT,因此，DLL_API 是 __declspec(dllexport)，用来导出
 2 //当我们在静态调用dll时，我们包含该头文件，由于没有定义DLL_EXPORT，所以DLL_API是
 3 //__declspec(dllimport),用来导入
 4 #ifdef DLL_EXPORTS
 5 #define DLL_API __declspec(dllexport)
 6 #else
 7 #define DLL_API __declspec(dllimport)
 8 #endif
 9 
10 // 导出类
11 class DLL_API Cdll {
12 public:
13     Cdll(void);
14     // TODO: 在此添加您的方法。
15 };
16 
17 //导出变量，变量在.cpp文件中定义
18 extern DLL_API int ndll;
19 
20 //导出函数，加extern "C",是为了保证编译时生成的函数名不变，这样动态调用dll时才能
21 //正确获取函数的地址
22 extern "C" DLL_API int fndll(void);

```

dll.cpp 文件如下：

```
 1 #include "dll.h"
 2 
 3 
 4 // 这是导出变量的一个示例
 5 DLL_API int ndll=6;
 6 
 7 // 这是导出函数的一个示例。
 8 DLL_API int fndll(void)
 9 {
10     return 42;
11 }
12 
13 // 这是已导出类的构造函数。
14 // 有关类定义的信息，请参阅 dll.h
15 Cdll::Cdll()
16 {
17     return;
18 }

```

**调用动态库**

有两种方法调用动态库，一种隐式链接，一种显示链接。

**调用动态库：隐式链接**

隐式链接 需要. h 文件，dll 文件，lib 文件

（1）将 dll 放到工程的工作目录

（2）设置项目属性 --vc++ 目录 -- 库目录为 lib 所在的路径

（3）将 lib 添加到项目属性 -- 链接器 -- 输入 -- 附加依赖项（或者直接在源代码中加入#pragma comment(lib, “**.lib”)）

（4）在源文件中添加. h 头文件

然后就像平常一样调用普通函数、类、变量

**调用动态库：显示链接**

显示链接 只需要. dll 文件，但是这种调用方式不能调用 dll 中的变量或者类（其实可以调用类，但是相当麻烦，有兴趣者可参考 [http://blog.csdn.net/jdcb2001/article/details/1394883](http://blog.csdn.net/jdcb2001/article/details/1394883)）

显示调用主要使用 WIN32 API 函数 LoadLibrary、GetProcAddress，举例如下：

```
 1 typedef int (*dllfun)(void);//定义指向dll中函数的函数指针
 2     HINSTANCE hlib = LoadLibrary(".\\dll.dll");
 3     if(!hlib)
 4     {
 5         std::cout<<"load dll error\n";
 6         return -1;
 7     }
 8     dllfun fun = (dllfun)GetProcAddress(hlib,"fndll");
 9     if(!fun)
10     {
11         std::cout<<"load fun error\n";
12         return -1;
13     }
14     fun();

```

**生成和使用静态库**

**生成静态库**

新建项目 --win32 项目 -- 填写项目名 -- 确定 -- 下一步 -- 应用程序类型：选择静态库

静态库项目没有 main 函数，也没有像 dll 项目中的 dllmain。

创建项目后添加. h 文件，添加相应的导出函数、变量或类，如下所示

```
 1 #ifndef _MYLIB_H_
 2 #define _MYLIB_H_
 3 
 4 void fun(int a);
 5 
 6 extern int k;
 7 
 8 class testclass
 9 {
10 public:
11     testclass();
12     void print();
13 };
14 
15 #endif

```

.cpp 文件如下

```
 1 #include "stdafx.h"
 2 #include "lib.h"
 3 #include <iostream>
 4 
 5 void fun(int a)
 6 {
 7     std::cout<<a<<"lib gen\n";
 8 }
 9 
10 int k = 222;
11 
12 testclass::testclass()
13 {
14     std::cout<<"123\n";
15 }
16 
17 void testclass::print()
18 {
19     std::cout<<"this is testcalss\n";
20 }

```

编译工程后就会生成一个. lib 文件

**使用静态库**

需要. h 文件，lib 文件

（1）设置项目属性 --vc++ 目录 -- 库目录为 lib 所在的路径

（2）将 lib 添加到项目属性 -- 链接器 -- 输入 -- 附加依赖项（或者直接在源代码中加入 #pragma comment(lib, “**.lib”)）

（3）在源文件中添加. h 头文件

然后就像平常一样调用普通函数、类、变量，举例如下：

```
 1 #include <iostream>
 2 #include "lib.h"
 3 
 4 #pragma comment(lib, "lib.lib")
 5 
 6 int main()
 7 {
 8     fun(4);
 9     std::cout<<k<<std::endl;
10     testclass tc; 
11     tc.print();
12     return 0;
13 }

```

 **【版权声明】转载请注明出处 [http://www.cnblogs.com/TenosDoIt/p/3203137.html](http://www.cnblogs.com/TenosDoIt/p/3203137.html)**