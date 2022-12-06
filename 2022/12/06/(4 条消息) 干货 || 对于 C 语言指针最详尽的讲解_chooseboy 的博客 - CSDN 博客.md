> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/lyh290188/article/details/104319721)

指针对于 C 来说太重要。然而，想要全面理解指针，除了要对 C 语言有熟练的掌握外，还要有计算机硬件以及操作系统等方方面面的基本知识。所以本文尽可能的通过一篇文章完全讲解指针。

指针解决了一些编程中基本的问题。

第一，指针的使用使得不同区域的代码可以轻易的共享数据。当然小伙伴们也可以通过数据的复制达到相同的效果，但是这样往往效率不太好。  
因为诸如[结构体](https://so.csdn.net/so/search?q=%E7%BB%93%E6%9E%84%E4%BD%93&spm=1001.2101.3001.7020)等大型数据，占用的字节数多，复制很消耗性能。  
但使用指针就可以很好的避免这个问题，因为任何类型的指针占用的字节数都是一样的（根据平台不同，有 4 字节或者 8 字节或者其他可能）。

第二，指针使得一些复杂的链接性的数据结构的构建成为可能，比如链表，链式二叉树等等。

第三，有些操作必须使用指针。如操作申请的堆内存。  
还有：C 语言中的一切函数调用中，值传递都是 “按值传递” 的。  
如果我们要在函数中修改被传递过来的对象，就必须通过这个对象的指针来完成。

指针是什么？

我们知道：C 语言中的数组是指一类类型，数组具体区分为  int 类型数组，double 类型数组, char 数组 等等。  
同样指针这个概念也泛指一类数据类型，int 指针类型，double 指针类型，char 指针类型等等。

通常，我们用 int 类型保存一些整型的数据，如 int num = 97 ， 我们也会用 char 来存储字符：char ch = 'a'。

我们也必须知道：任何程序数据载入内存后，在内存都有他们的地址，这就是指针。  
而为了保存一个数据在内存中的地址，我们就需要指针变量。

因此：指针是程序数据在内存中的地址，而指针变量是用来保存这些地址的变量。

 ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejVkNHc5RngzQUNFZzhGU3JqdzkyMHFlNDZzcjBOV0NuQ29GRzFtZURpY0VTdFBkTkhJQ2M1V2F3LzY0MA?x-oss-process=image/format,png)

为什么程序中的数据会有自己的地址？

弄清这个问题我们需要从操作系统的角度去认知内存。

电脑维修师傅眼中的内存是这样的：内存在物理上是由一组 DRAM 芯片组成的。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejV4cmViVHFESW9hVlNpYlNGUTJkalU0TG0zeHJDb1h1MVVJRXhIbEs1d1dxekFiR2tSTEwxeVNRLzY0MA?x-oss-process=image/format,png)

而作为一个程序员，我们不需要了解内存的物理结构，操作系统将 RAM 等硬件和软件结合起来，给程序员提供的一种对内存使用的抽象。  
这种抽象机制使得程序使用的是虚拟存储器, 而不是直接操作和使用真实存在的物理存储器。  
所有的虚拟地址形成的集合就是虚拟地址空间。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejU3R2xUZU1jYVdhb0NHTm9pY3FSTHhpYmpmQ2d6YTlYYkpmM2NpYUxFNnhUVE5vaHo2UmxCOU9jb2cvNjQw?x-oss-process=image/format,png)

在程序员眼中的内存应该是下面这样的。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejVWbmx2TjRHMUNUUkdBSXZpYUI1NlQ3cHRFN290aWNEMEdqSUNXczh6d3lMR0d2aWN5OXM5UDB3ekEvNjQw?x-oss-process=image/format,png)

也就是说，内存是一个很大的，线性的字节数组（平坦寻址）。每一个字节都是固定的大小，由 8 个二进制位组成。  
最关键的是，每一个字节都有一个唯一的编号, 编号从 0 开始，一直到最后一个字节。  
如上图中，这是一个 256M 的内存，他一共有 256x1024x1024  = 268435456 个字节，那么它的地址范围就是 0 ~268435455  。

由于内存中的每一个字节都有一个唯一的编号。  
因此，在程序中使用的变量，常量，甚至数函数等数据，当他们被载入到内存中后，都有自己唯一的一个编号，这个编号就是这个数据的地址。  
指针就是这样形成的。

下面用代码说明

#include <stdio.h>

```
int main(void)
{
    char ch = 'a';
    int  num = 97;
    printf("ch 的地址:%p
",&ch);   //ch 的地址:0028FF47
    printf("num的地址:%p
",&num);  //num的地址:0028FF40
    return 0;
}

```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejVjUzN3UXZEOXZvd0ZWV0o2V3d4RTlTaEluMkw0Zk5RRHVEbG9GS09kanJQSFdRMTdTMWw4aWFBLzY0MA?x-oss-process=image/format,png)

指针的值实质是内存单元（即字节）的编号，所以指针单独从数值上看，也是整数，他们一般用 16 进制表示。  
指针的值（虚拟地址值）使用一个机器字的大小来存储。  
也就是说, 对于一个机器字为 w 位的电脑而言, 它的虚拟地址空间是 0~2w － 1 , 程序最多能访问 2w 个字节。  
这就是为什么 xp 这种 32 位系统最大支持 4GB 内存的原因了。

我们可以大致画出变量 ch 和 num 在内存模型中的存储。（假设 char 占 1 个字节，int 占 4 字节）

 ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejVicGJDbzB1NzZnWTNZUVlsdjIxTTNZTGljRWJIQ0Y3REZLazBOSWs5ckJzdDRabk5HSjNRWUlRLzY0MA?x-oss-process=image/format,png)

**变量和内存**

为了简单起见，这里就用上面例子中的  int num = 97 这个局部变量来分析变量在内存中的存储模型。

 ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejU1MGN1SnNSM0VFeEJsV3N3VnZIbFhiRUdIRnNoalFJZURndHppYUhpYVlVdXRINEhOd2FpY1Y4cGcvNjQw?x-oss-process=image/format,png)

已知：num 的类型是 int，占用了 4 个字节的内存空间，其值是 97，地址是 0028FF40。我们从以下几个方面去分析。

**1、内存的数据**

内存的数据就是变量的值对应的二进制，一切都是二进制。  
97 的二进制是 : 00000000 00000000 00000000 0110000 , 但使用的小端模式存储时，低位数据存放在低地址，所以图中画的时候是倒过来的。

**2、内存数据的类型**

内存的数据类型决定了这个数据占用的字节数，以及计算机将如何解释这些字节。  
num 的类型是 int，因此将被解释为 一个整数。

**3、内存数据的名称**

内存的名称就是变量名。实质上，内存数据都是以地址来标识的，根本没有内存的名称这个说法，这只是高级语言提供的抽象机制 ，方便我们操作内存数据。  
而且在 C 语言中，并不是所有的内存数据都有名称，例如使用 malloc 申请的堆内存就没有。

**4、内存数据的地址**

如果一个类型占用的字节数大于 1，则其变量的地址就是地址值最小的那个字节的地址。  
因此 num 的地址是 0028FF40。内存的地址用于标识这个内存块。

**5、内存数据的生命周期**

num 是 main 函数中的局部变量，因此当 main 函数被启动时，它被分配于栈内存上，当 main 执行结束时，消亡。

如果一个数据一直占用着他的内存，那么我们就说他是 “活着的”，如果他占用的内存被回收了，则这个数据就 “消亡了”。  
C 语言中的程序数据会按照他们定义的位置，数据的种类，修饰的关键字等因素，决定他们的生命周期特性。  
实质上我们程序使用的内存会被逻辑上划分为：栈区，堆区，静态数据区，方法区。  
不同的区域的数据有不同的生命周期。

无论以后计算机硬件如何发展，内存容量都是有限的，因此清楚理解程序中每一个程序数据的生命周期是非常重要的。

**指针变量和指向关系**

用来保存指针的变量，就是指针变量。  
如果指针变量 p1 保存了变量 num 的地址，则就说：p1 指向了变量 num，也可以说 p1 指向了 num 所在的内存块 ，这种指向关系，在图中一般用 箭头表示。

 ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejVpY1Yxc3pkUkcxekZJaWNMN3kyNExTTnhoWEZpYVRzbkpuTHBDdWY3WmUzRU5GWEZvWkJ0Tjl3eEEvNjQw?x-oss-process=image/format,png)

上图中，指针变量 p1 指向了 num 所在的内存块 ，即从地址 0028FF40 开始的 4 个 byte 的内存块。

定义指针变量

C 语言中，定义变量时，在变量名前写一个 * 星号，这个变量就变成了对应变量类型的指针变量。必要时要加 ( ) 来避免优先级的问题。

引申：C 语言中，定义变量时，在定义的最前面写上 typedef ，那么这个变量名就成了一种类型，即这个类型的同义词。

```
int a ; //int类型变量 a
int *a ; //int* 变量a
int arr[3]; //arr是包含3个int元素的数组
int (* arr )[3]; //arr是一个指向包含3个int元素的数组的指针变量
//-----------------各种类型的指针------------------------------
int* p_int; //指向int类型变量的指针 
double* p_double; //指向idouble类型变量的指针 
struct Student *p_struct; //结构体类型的指针
int(*p_func)(int,int); //指向返回类型为int，有2个int形参的函数的指针 
int(*p_arr)[3]; //指向含有3个int元素的数组的指针 
int** p_pointer; //指向 一个整形变量指针的指针

```

**取地址**

既然有了指针变量，那就得让他保存其它变量的地址，使用 & 运算符取得一个变量的地址。

```
int add(int a , int b)
{
    return a + b;
}
int main(void)
{
    int num = 97;
    float score = 10.00F;
    int arr[3] = {1,2,3};
    //-----------------------
    int* p_num = #
    float* p_score = &score;
    int (*p_arr)[3] = &arr;           
    int (*fp_add)(int ,int )  = add;  //p_add是指向函数add的函数指针
    return 0;
}

```

特殊的情况，他们并不一定需要使用 & 取地址：

*   数组名的值就是这个数组的第一个元素的地址。
    
*   函数名的值就是这个函数的地址。
    
*   字符串字面值常量作为右值时，就是这个字符串对应的字符数组的名称, 也就是这个字符串在内存中的地址。
    

```
int add(int a , int b){
    return a + b;
}
int main(void)
{
    int arr[3] = {1,2,3};
    //-----------------------
    int* p_first = arr;
    int (*fp_add)(int ,int )  =  add;
    const char* msg = "Hello world";
    return 0;
}

```

**解地址**

我们需要一个数据的指针变量干什么？  
当然使用通过它来操作（读 / 写）它指向的数据啦。  
对一个指针解地址，就可以取到这个内存数据，解地址的写法，就是在指针的前面加一个 * 号。

```
int main(void)
{
    int age = 19;
    int*p_age = &age;
    *p_age  = 20;  //通过指针修改指向的内存数据
    printf("age = %d
",*p_age);   //通过指针读取指向的内存数据
    printf("age = %d
",age);
    return 0;
}

```

**指针之间的赋值**

指针赋值和 int 变量赋值一样，就是将地址的值拷贝给另外一个。  
指针之间的赋值是一种浅拷贝，是在多个编程单元之间共享内存数据的高效的方法。

```
int* p1  = & num;
int* p3 = p1;
//通过指针 p1 、 p3 都可以对内存数据 num 进行读写，如果2个函数分别使用了p1 和p3，那么这2个函数就共享了数据num。

```

![](https://img-blog.csdnimg.cn/20200214211618220.gif)

**空指针**

指向空，或者说不指向任何东西。  
在 C 语言中，我们让指针变量赋值为 NULL 表示一个空指针，而 C 语言中，NULL 实质是 ((void*)0) ，  在 C++ 中，NULL 实质是 0。

换种说法：任何程序数据都不会存储在地址为 0 的内存块中，它是被操作系统预留的内存块。

下面代码摘自 stdlib.h

```
#ifdef __cplusplus
     #define NULL    0
#else    
     #define NULL    ((void *)0)
#endif

```

坏指针

指针变量的值是 NULL，或者未知的地址值，或者是当前应用程序不可访问的地址值，这样的指针就是坏指针。  
不能对他们做解指针操作，否则程序会出现运行时错误，导致程序意外终止。

任何一个指针变量在做解地址操作前，都必须保证它指向的是有效的，可用的内存块，否则就会出错。  
坏指针是造成 C 语言 Bug 的最频繁的原因之一。

```
下面的代码就是错误的示例。
void opp()
{
     int*p = NULL;
     *p = 10;      //Oops! 不能对NULL解地址
}
void foo()
{
     int*p;
     *p = 10;      //Oops! 不能对一个未知的地址解地址
}
void bar()
{
     int*p = (int*)1000; 
     *p =10;      //Oops!   不能对一个可能不属于本程序的内存的地址的指针解地址
}

```

**指针的 2 个重要属性**

指针也是一种数据，指针变量也是一种变量，因此指针 这种数据也符合前面变量和内存主题中的特性。  
这里要强调 2 个属性：指针的类型，指针的值。

```
int main(void)
{
    int num = 97;
    int *p1  = #
    char* p2 = (char*)(&num);
    printf("%d
",*p1);    //输出  97
    putchar(*p2);          //输出  a
    return 0;
}

```

指针的值：很好理解，如上面的 num 变量 ，其地址的值就是 0028FF40 ，因此 p1 的值就是 0028FF40。  
数据的地址用于在内存中定位和标识这个数据，因为任何 2 个内存不重叠的不同数据的地址都是不同的。

指针的类型：指针的类型决定了这个指针指向的内存的字节数并如何解释这些字节信息。  
一般指针变量的类型要和它指向的数据的类型匹配。

由于 num 的地址是 0028FF40，因此 p1 和 p2 的值都是 0028FF40

*p1  :  将从地址 0028FF40 开始解析，因为 p1 是 int 类型指针，int 占 4 字节，因此向后连续取 4 个字节，并将这 4 个字节的二进制数据解析为一个整数 97。

*p2  :  将从地址 0028FF40 开始解析，因为 p2 是 char 类型指针，char 占 1 字节，因此向后连续取 1 个字节，并将这 1 个字节的二进制数据解析为一个字符，即'a'。

同样的地址，因为指针的类型不同，对它指向的内存的解释就不同，得到的就是不同的数据。

void * 类型指针 

由于 void 是空类型，因此 void * 类型的指针只保存了指针的值，而丢失了类型信息，我们不知道他指向的数据是什么类型的，只指定这个数据在内存中的起始地址。  
如果想要完整的提取指向的数据，程序员就必须对这个指针做出正确的类型转换，然后再解指针。  
因为，编译器不允许直接对 void * 类型的指针做解指针操作。

**结构体和指针**

结构体指针有特殊的语法：-> 符号

如果 p 是一个结构体指针，则可以使用 p ->【成员】 的方法访问结构体的成员

```
typedef struct
{
    char name[31];
    int age;
    float score;
}Student;
int main(void)
{
    Student stu = {"Bob" , 19, 98.0};
    Student*ps = &stu;
    ps->age = 20;
    ps->score = 99.0;
    printf("name:%s age:%d
",ps->name,ps->age);
    return 0;
}

```

**数组和指针**

1、数组名作为右值的时候，就是第一个元素的地址。

```
int main(void)
{
    int arr[3] = {1,2,3};
    int*p_first = arr;
    printf("%d
",*p_first);  //1
    return 0;
}

```

2、指向数组元素的指针 支持 递增 递减 运算。  
（实质上所有指针都支持递增递减 运算 ，但只有在数组中使用才是有意义的）

```
int main(void)
{
    int arr[3] = {1,2,3};
    int*p = arr;
    for(;p!=arr+3;p++){
        printf("%d
",*p); 
    }
    return 0;
}

```

3、p= p+1 意思是，让 p 指向原来指向的内存块的下一个相邻的相同类型的内存块。

同一个数组中，元素的指针之间可以做减法运算，此时，指针之差等于下标之差。

4、p[n]    == *(p+n)

     p[n][m]  == *(  *(p+n)+ m )

5、当对数组名使用 sizeof 时，返回的是整个数组占用的内存字节数。当把数组名赋值给一个指针后，再对指针使用 sizeof 运算符，返回的是指针的大小。

这就是为什么将一个数组传递给一个函数时，需要另外用一个参数传递数组元素个数的原因了。

```
int main(void)
{
    int arr[3] = {1,2,3};
    int*p = arr;
    printf("sizeof(arr)=%d
",sizeof(arr));  //sizeof(arr)=12
    printf("sizeof(p)=%d
",sizeof(p));   //sizeof(p)=4
    return 0;
}

```

**函数和指针**

函数的参数和指针

C 语言中，实参传递给形参，是按值传递的，也就是说，函数中的形参是实参的拷贝份，形参和实参只是在值上面一样，而不是同一个内存数据对象。  
这就意味着：这种数据传递是单向的，即从调用者传递给被调函数，而被调函数无法修改传递的参数达到回传的效果。

```
void change(int a)
{
    a++;      //在函数中改变的只是这个函数的局部变量a，而随着函数执行结束，a被销毁。age还是原来的age，纹丝不动。
}
int main(void)
{
    int age = 19;
    change(age);
    printf("age = %d
",age);   // age = 19
    return 0;
}

```

有时候我们可以使用函数的返回值来回传数据，在简单的情况下是可以的。  
但是如果返回值有其它用途（例如返回函数的执行状态量），或者要回传的数据不止一个，返回值就解决不了了。

传递变量的指针可以轻松解决上述问题。

```
void change(int* pa)
{
    (*pa)++;   //因为传递的是age的地址，因此pa指向内存数据age。当在函数中对指针pa解地址时，
               //会直接去内存中找到age这个数据，然后把它增1。
}
int main(void)
{
    int age = 19;
    change(&age);
    printf("age = %d
",age);   // age = 20
    return 0;
}

```

再来一个老生常谈的，用函数交换 2 个变量的值的例子：

```
#include<stdio.h>
void swap_bad(int a,int b);
void swap_ok(int*pa,int*pb);
int main()
{
    int a = 5;
    int b = 3;
    swap_bad(a,b);       //Can`t swap;
    swap_ok(&a,&b);      //OK
    return 0;
}
//错误的写法
void swap_bad(int a,int b)
{
    int t;
    t=a;
    a=b;
    b=t;
}
//正确的写法：通过指针
void swap_ok(int*pa,int*pb)
{
    int t;
    t=*pa;
    *pa=*pb;
    *pb=t;
}

```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejVxWWlhU1REN2cxMXNEUUNscUI1N1RGUDdXTzZpYlczTVhucmdpYUpIdUtKWUJwTlBZcUZUZzRTQ1EvNjQw?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejVrOVU5UDVHTmliZnhVR25WRlh6MWpldHZJU1c5TmhlUkhqaElFZlR2SmhpYnVJc0hic0UzaEo3Zy82NDA?x-oss-process=image/format,png)

有的时候，我们通过指针传递数据给函数不是为了在函数中改变他指向的对象。  
相反，我们防止这个目标数据被改变。传递指针只是为了避免拷贝大型数据。

考虑一个结构体类型 Student。我们通过 show 函数输出 Student 变量的数据。

```
typedef struct
{
    char name[31];
    int age;
    float score;
}Student;
//打印Student变量信息
void show(const Student * ps)
{
    printf("name:%s , age:%d , score:%.2f
",ps->name,ps->age,ps->score);   
}

```

我们只是在 show 函数中取读 Student 变量的信息，而不会去修改它，为了防止意外修改，我们使用了常量指针去约束。  
另外我们为什么要使用指针而不是直接传递 Student 变量呢？

从定义的结构看出，Student 变量的大小至少是 39 个字节，那么通过函数直接传递变量，实参赋值数据给形参需要拷贝至少 39 个字节的数据，极不高效。  
而传递变量的指针却快很多，因为在同一个平台下，无论什么类型的指针大小都是固定的：X86 指针 4 字节，X64 指针 8 字节，远远比一个 Student 结构体变量小。

**函数的指针**

每一个函数本身也是一种程序数据，一个函数包含了多条执行语句，它被编译后，实质上是多条机器指令的合集。  
在程序载入到内存后，函数的机器指令存放在一个特定的逻辑区域：代码区。  
既然是存放在内存中，那么函数也是有自己的指针的。

C 语言中，函数名作为右值时，就是这个函数的指针。

```
void echo(const char *msg)
{
    printf("%s",msg);
}
int main(void)
{
    void(*p)(const char*) = echo;   //函数指针变量指向echo这个函数
    p("Hello ");      //通过函数的指针p调用函数，等价于echo("Hello ")
    echo("World
");
    return 0;
}

```

**const 和指针**

const 到底修饰谁？谁才是不变的？

如果 const 后面是一个类型，则跳过最近的原子类型，修饰后面的数据。  
（原子类型是不可再分割的类型，如 int, short , char，以及 typedef 包装后的类型）

如果 const 后面就是一个数据，则直接修饰这个数据。

```
int main()
{
    int a = 1;
    int const *p1 = &a;        //const后面是*p1，实质是数据a，则修饰*p1，通过p1不能修改a的值
    const int*p2 =  &a;        //const后面是int类型，则跳过int ，修饰*p2， 效果同上
    int* const p3 = NULL;      //const后面是数据p3。也就是指针p3本身是const .
    const int* const p4 = &a;  // 通过p4不能改变a 的值，同时p4本身也是 const
    int const* const p5 = &a;  //效果同上
    return 0;
}
typedef int* pint_t;  //将 int* 类型 包装为 pint_t,则pint_t 现在是一个完整的原子类型
int main()
{
    int a  = 1;
    const pint_t p1 = &a;  //同样，const跳过类型pint_t，修饰p1，指针p1本身是const
    pint_t const p2 = &a;  //const 直接修饰p，同上
    return 0;
}

```

**深拷贝和浅拷贝**

如果 2 个程序单元（例如 2 个函数）是通过拷贝他们所共享的数据的指针来工作的，这就是浅拷贝，因为真正要访问的数据并没有被拷贝。  
如果被访问的数据被拷贝了，在每个单元中都有自己的一份，对目标数据的操作相互不受影响，则叫做深拷贝。

 ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejVHazdlOHhOTkgyNHFiM0JpYXdrM29wRHhjQlA2bkpUUjRVaWE3azJ1bVdVV1JMYlZRSE5UWHlFdy82NDA?x-oss-process=image/format,png)

**附加知识**

指针和引用这个 2 个名词的区别。他们本质上来说是同样的东西。  
指针常用在 C 语言中，而引用，则用于诸如 Java，C# 等 在语言层面封装了对指针的直接操作的编程语言中。

大端模式和小端模式

1) Little-Endian 就是低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。个人 PC 常用，Intel X86 处理器是小端模式。

2) B i g-Endian 就是高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。

采用大端方式进行数据存放符合人类的正常思维，而采用小端方式进行数据存放利于计算机处理。  
有些机器同时支持大端和小端模式, 通过配置来设定实际的端模式。

假如 short 类型占用 2 个字节，且存储的地址为 0x30。

short a = 1;

如下图：

 ![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9ocDZXQTg4SlE0U2RKdXNiZU55QkYwZ2xpYk1OV21XejVpY0U5OEpFSHhld0JCSGFzdXc1dm5nclVBVWxOdnp3cHhGU1VoemUzMzFNcVNlamR2cDM2MGNRLzY0MA?x-oss-process=image/format,png)

// 测试机器使用的是否为小端模式。是，则返回 true，否则返回 false  
// 这个方法判别的依据就是：C 语言中一个对象的地址就是这个对象占用的字节中，地址值最小的那个字节的地址。  
 

```
bool isSmallIndain()
{
      unsigned int val = 'A';
      unsigned char* p = (unsigned char*)&val;  //C/C++：对于多字节数据，取地址是取的数据对象的第一个字节的地址，也就是数据的低地址
      return *p == 'A';
}

```