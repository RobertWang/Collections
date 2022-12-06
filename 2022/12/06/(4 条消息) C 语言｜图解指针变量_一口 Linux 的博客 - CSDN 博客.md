> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/daocaokafei/article/details/127399046)

1 指针变量的基本操作基本操作
---------------

```
int a,*iptr,*jptr,*kptr;
iptr = &a;
jptr = iptr;
*jptr = 100;
kptr = NULL;

```

### 1.1 己址和己空间

指针变量也是一个变量，对应一块[内存](https://so.csdn.net/so/search?q=%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)空间，对应一个内存地址，指针名就是己址。这空内存空间多大？一个机器字长（machine word），32 位的 CPU 和操作系统就是 32 个位，4 个字节，其值域为：0x-0xFFFFFFFF。64 位的 CPU 和操作系统就是 64 个位，8 个字节，其值域为：0x-0xFFFFFFFFFFFFFFFF。

### 1.2 己值、他址、他空间

指针变量的值就是其指向的空间的地址，指向的地址的空间大小就是指针变量指向类型的大小。

### 1.3 声明与初始化

当声明一个指针变量，没有初始化时，指针变量只获得了其自身的内存空间，而其指向还没有确定，此时指针变量解引用做左值是非法操作。如果要使用指针变量解引用做左值，有三条途径：

```
int *ptr;
int *ptr_2;
int a = 1;
ptr_2 = &a;
// *ptr = 0;    // 非法操作，其指向其指向的内存空间还未确定
ptr = &a;                       // ① 右值是一个变量地址
ptr = ptr_2;                    // ② 右值是一个同类型指针，且已初始化
ptr = (int*)malloc(sizeof(int));// ③ 右值是一个内存分配函数返回一个void指针
*ptr = 0;       // 合法操作，ptr有了确定的指向及指向的内存空间；

```

### 1.4 函数之间指针值的传递

函数（如下例的 funcForSpace()）内定义局部变量（如下例的 a）保存在一个函数的[栈帧](https://so.csdn.net/so/search?q=%E6%A0%88%E5%B8%A7&spm=1001.2101.3001.7020)上，当一个函数执行完毕后，另一个函数（如下例的 stackFrame_reuse()）执行时，该空间会被 stackFrame_reuse() 重复使用，a 所使用的空间将不复存在，所以当一个指针变量指向局部变量的内存空间时，其地址值传递给主调函数时，并不是一个有效值。

```
#include <stdio.h>

void funcForSpace(int **iptr) {
    int a = 10;
    *iptr = &a;
}
void stackFrame_reuse()
{
    int a[1024] = {0};
}
int main()
{
    int *pNew;
    funcForSpace(&pNew);
    printf("%d\n",*pNew); // 10，此时栈帧还未被重复使用
    stackFrame_reuse();
    printf("%d\n",*pNew); // -858993460，垃圾值
    while(1);
    return 0;
}

```

可以在 funcForSpace() 内分配一块堆内存，传递给主调函数。

```
#include <stdio.h>
#include <malloc.h>
int g(int **iptr) { // 当试图修改主调函数的一级指针变量时，被调函数的参数是一个二级指针
    if ((*iptr = (int *)malloc(sizeof(int))) == NULL)
        return -1;
}
int main()
{
    int *jptr;
    g(&jptr);
    *jptr = 10;
    printf("%d\n",*jptr); // 10
    free(jptr);
    while(1);
    return 0;
}

```

可以图示一下以上代码指针的传递过程：  
![](https://img-blog.csdnimg.cn/c7df87000c604bed99ced17152ee6341.png)

以下图示 a 表示计算机内存，b 表示一个函数调用时在栈（stack）上开辟的栈帧空间：  
![](https://img-blog.csdnimg.cn/c20aea21dfc74d70aaf8a43e951aaf4c.png)

2 指针变量与数组名
----------

数组名在一定的上下文中会转换为指向数组首元素的地址，以方便指针的算术运算，如

```
#include <stdio.h>

int main()
{
    int a[5] = {0}; 
    char b[20] = {0};
    *(a+3) = 10;    // a+3是指相对于地址a，偏移sizeof(int)个字节
    *(b+3) = 'x';   // b+3是指相对于地址b，偏移sizeof(char)个字节

    printf("%d, %c\n",a[3],b[3]); // 10, x
    while(1);
    return 0;
}

```

可以图示一下以上代码指针的偏移细节：  
![](https://img-blog.csdnimg.cn/c6d635e9d38245e48e564eec1dadfde4.png)

3 主调函数与被调函数之间的指针传递
------------------

看以下代码：

```
#include <stdio.h>
void swap1(int x, int y) {
    int tmp;
    tmp = x; x = y; y = tmp;
}
void swap2(int *x, int *y) {
    int tmp;
    tmp = *x; *x = *y; *y = tmp;
}
void caller()
{
    int a = 10;
    int b = 20;
    swap1(a,b);
    printf("%d %d\n",a,b);
    swap2(&a,&b);
    printf("%d %d\n",a,b);
}
int main()
{
    caller();
    return 0;
}

```

以上代码可用以下图示理解：

swap1 传值：  
![](https://img-blog.csdnimg.cn/8e0f5914cee64bd994779bb0ebfd2dc2.png)

swap2 传址（指针传递）：  
![](https://img-blog.csdnimg.cn/55b79c53e2f64843a9ceeeaede770f56.png)

4 数组做函数参数
---------

[二维数组](https://so.csdn.net/so/search?q=%E4%BA%8C%E7%BB%B4%E6%95%B0%E7%BB%84&spm=1001.2101.3001.7020)是数组的数组，n 维数组是 n-1 维数组的数组。内存是一维的字节序列，所谓的 n 维数组其实只是一个逻辑意义的表示，其物理结构还是一维线性的。

n 维数组的元素是一个 n-1 维数组。如果用指针指向一个 n 维数组，其指针类型必须有 n-1 维的长度信息，当其用作函数参数时也是如此。

```
void g(int a[][2]) { // void g(int(*a)[2]){是相同写法
    a[2][0] = 5;
}
void caller()
{
    int a[3][2];
    int (*p)[2] = a;
    *(*(p+2)+0) = 7; // p=2表示相对于地址p偏移sizeof(*p)
    printf("%d\n",a[2][0]);  // 7
    g(a);
    printf("%d\n",a[2][0]); //  5
}

```

以下代码可以用以下图示辅助理解：

![](https://img-blog.csdnimg.cn/a3c7d087804045f484d7c6f536e878f9.png)

ref:

Kyle Loudon《 Mastering Algorithms with C》