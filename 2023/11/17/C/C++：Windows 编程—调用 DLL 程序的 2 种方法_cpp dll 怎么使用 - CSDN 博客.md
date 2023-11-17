> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_29542611/article/details/86618902)

前言
--

先简单介绍下 DLL。DLL：**Dynamic Link Library 动态链接库** 是一个被其他应用程序调用的程序模块，其中封装了可以**被调用的资源或函数**。**DLL 文件属于可执行文件**，它符合 Windows 系统的 PE 文件格式，**不过它是依附于 EXE 文件创建的的进程来执行的，不能单独运行**。为了演示调用 DLL 程序的 2 种方法，我们先建一个简单的 DLL 程序。

建一个简单的 DLL 程序
-------------

IDE 使用 vs2015，新建工程 DLLTest1，选择空项目，创建完毕 右击项目 -> 属性 -> 常规 -> 配置类型 选择 动态库. dll。还是上一张图吧。  
![](https://img-blog.csdnimg.cn/20190123225422622.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NTQyNjEx,size_16,color_FFFFFF,t_70)

添加头文件 Calc.h 在头文件中添加导出函数 add 函数

```
#pragma once

extern "C" __declspec(dllexport) int add(int a, int b);

```

cpp 文件中进行实现

```
#include "Calc.h"

int add(int a, int b)
{
	return a + b;
}

```

生成解决方案，在 Debug 下生成 DLLTest1.dll 和 DLLTest1.lib  
![](https://img-blog.csdnimg.cn/20190123225808983.)

对 DLL 程序调用方式一
-------------

同样是新建空项目，添加 main.cpp 文件，将 DLLTest1.dll 和 DLLTest1.lib 拷贝到工程代码目录，然后项目添加添加现有项。项目目录如下  
在这里插入图片描述  
![](https://img-blog.csdnimg.cn/20190123230333583.)

使用代码如下：

```
#include <stdio.h>
#include <stdlib.h>
#include <Windows.h>

#pragma comment(lib,"DLLTest1.lib")

extern "C" int add(int a, int b);

// 静态调用DLL库
void StaticUse()
{
	int sum = add(10, 20);
	printf("静态调用，sum = %d\n", sum);
}

```

方式一 是静态调用，在连接阶段 将 DLL 库信息编写到 EXE 文件中，当调用 DLL 库中的函数是会加重 DLL 库。#pragma comment(lib,“DLLTest1”) 告诉连机器需要在 FirstDll.lib 文件中找到 DLL 中导出函数的信息。

对 DLL 程序调用方式二
-------------

**方法一属于静态调用**，其方式是通过**链接器将 DLL 函数的导出函数写进可执行文件**。现在使用第二种方式，**相对前一种 是动态调用**。动态调用不是链接时完成的，**而是在运行时完成的。动态调用不会在可执行文件中写入 DLL 相关的信息**。代码如下：

```
// 动态调用DLL库
void DynamicUse()
{
    // 运行时加载DLL库
	HMODULE module = LoadLibrary("DLLTest1.dll");
	if (module == NULL)
	{
		printf("加载DLLTest1.dll动态库失败\n");
		return;
	}
	typedef int(*AddFunc)(int, int); // 定义函数指针类型
	AddFunc add; 
    // 导出函数地址
	add = (AddFunc)GetProcAddress(module, "add");

	int sum  = add(100, 200);
	printf("动态调用，sum = %d\n",sum);
}

```

用到了以下 2 个函数：

```
// 根据DLL文件名 加载DLL
// suc，返回一个模块句柄
HMODULE WINAPI LoadLibrary(
  _In_ LPCTSTR lpFileName
);
// suc,返回lpProcName指向的函数名的函数地址。
FARPROC GetProcAddress(
  HMODULE hModule,
  LPCSTR  lpProcName
);

```

测试
--

测试代码如下：

```
#include <stdio.h>
#include <stdlib.h>
#include <Windows.h>

#pragma comment(lib,"DLLTest1.lib")

extern "C" int add(int a, int b);

// 静态调用DLL库
void StaticUse()
{
	int sum = add(10, 20);
	printf("静态调用，sum = %d\n", sum);
}

// 动态调用DLL库
void DynamicUse()
{
	HMODULE module = LoadLibrary("DLLTest1.dll");
	if (module == NULL)
	{
		printf("加载DLLTest1.dll动态库失败\n");
		return;
	}
	typedef int(*AddFunc)(int, int); // 定义函数指针类型
	AddFunc add;
	add = (AddFunc)GetProcAddress(module, "add");

	int sum = add(100, 200);
	printf("动态调用，sum = %d\n", sum);
}

int main(char argc, char* argv[])
{
	StaticUse();
	DynamicUse();
	system("pause");
	return 0;
}

```

验证结果，和我们想象的一样。  
![](https://img-blog.csdnimg.cn/20190123230857714.)

完整项目
----

如果 有需要，这个 2 个工程[这里下载](https://download.csdn.net/download/qq_29542611/10935130)。