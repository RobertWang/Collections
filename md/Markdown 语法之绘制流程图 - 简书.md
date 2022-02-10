> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/7864c1cf5660)

流程图语法
-----

### 1、创建流程图模块

语法如下：

```
  ```mermaid


```

### 2、流程图方向

<table><thead><tr><th>标志</th><th>方向</th></tr></thead><tbody><tr><td>TB</td><td>top bottom - 从上到下</td></tr><tr><td>BT</td><td>bottom top - 从下到上</td></tr><tr><td>RL</td><td>right left - 从右到左</td></tr><tr><td>LR</td><td>left right - 从左到右</td></tr><tr><td>TD</td><td>等同于 TB</td></tr></tbody></table>

创建一个从上到下流程图语法如下：

```
```mermaid
graph TB
    1[开始] --> 2[结束]


```

![](http://upload-images.jianshu.io/upload_images/3096223-12b0026a953e59f2.png)

### 3、流程块形状样式

```
```mermaid
graph LR
    1[方形] --> 2(圆角) --> 3((圆形)) --> 4>非对称] --> 5{菱形} --> 6{{六角形}}


```

![](http://upload-images.jianshu.io/upload_images/3096223-eb7c672614f71467.png)

```
```mermaid
graph LR
    1[\平行四边形\] --> 2[/平行四边形/] --> 3[/梯形\] --> 4[\梯形/]


```

![](http://upload-images.jianshu.io/upload_images/3096223-9d6287fea2300a9a.png)

### 4、连接线样式

> 样式的含义：
> 
> *   有箭头：一般指数据流方向
> *   无箭头：仅表示相关性
> *   实线：强关联
> *   虚线：弱关联

#### 4.1、箭头样式

```
```mermaid
graph LR
    1[开始] --> 2[结束]


```

![](http://upload-images.jianshu.io/upload_images/3096223-75fd57d4a02d42e9.png)

#### 4.2、无向线段连接线

```
```mermaid
graph LR
    1[begin] --- 2[end] -- 带文字的无向连接线 --- 3[ooooooo] 


```

![](http://upload-images.jianshu.io/upload_images/3096223-255ae47a4b0ed280.png)

#### 4.3、点状链接线（虚线）

```
```mermaid
graph LR
    1[one] -.- 2[two] -.带文字的虚线.- 3[three] -.带文字和箭头的虚线.-> 4[four]


```

![](http://upload-images.jianshu.io/upload_images/3096223-90ae8b141df4c30d.png)

#### 4.4、加粗线条

```
```mermaid
graph LR
    1[one] === 2[two] == 带文字粗箭头 ==> 3[three]


```

![](http://upload-images.jianshu.io/upload_images/3096223-261daddc5e3e1dd8.png)

### 5、分组

```
```mermaid
graph TB
    c1-->a2
    subgraph 第一组
    a1-->a2
    end
    subgraph 第二组
    b1-->b2
    end
    subgraph 第三组
    c1-->c2
    end


```

![](http://upload-images.jianshu.io/upload_images/3096223-98563d5243369960.png)

6、实例
----

```
```mermaid
graph LR
    执行1[i = 1]
  执行2[j = 0]
  执行3[i ++]
  执行4["a = arr[j], b = arr[j + 1]"]
  执行5[交换 a, b]
  执行6[j ++]
    判断1["i < n"]
    判断2["j < n - i"]
  判断3["a > b"]
  开始 --> 执行1
  执行1 --> 判断1
  判断1 --Y--> 执行2
  执行2 --> 判断2
  判断2 --Y--> 执行4
  判断2 --N--> 执行3
  执行3 --> 判断1
  执行4 --> 判断3
  判断3 --N--> 判断2
  判断3 --Y--> 执行5
  执行5 --> 执行6
  执行6 --> 判断2
  判断1 --N--> 结束


```

![](http://upload-images.jianshu.io/upload_images/3096223-7cc85a72c352ff2a.png)