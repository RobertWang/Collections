> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/EU2DHSxur59ydcjIz8-JIA)

**技术干货哪里找？**
============

**👆 点击上方****蓝字****关注我们！**
==========================

背景  

=====

线下场景中，我们经常需要在 APK 中插入一些检测代码，来实现一些记录方法调用耗时，或者增加一些打印日志的功能。目前的常规做法都是在编译期修改 class 字节码达到，例如 byteX 提供了方便的修改 class 框架。

**工具目前可以实现：**

*   方法前插桩
    
*   方法后插桩
    
*   初始化插桩
    

技术方案调研
======

调研了一下市面上现有的字节码修改方案。

### smali

可以通过 smali 和 baksmali 工具将 dex 文件转换成可方便阅读的 smali 语法文件，但是 smali 的工具对 smali 字节码的解析是通过语法解析，如果要插入一个新的代码进去对寄存器等操作没有办法实现结构化操作。

### redex

redex 支持通过配置在方法前进行插桩，可以通过实现 pass 来完成自己的插桩功能。但是功能实现有限，使用起来比较复杂，而且在执行之后插入了一些 fb 自定义的代码，但 Redex 提供了一套强大的字节码修改能力，后续的版本会基于 redex 的字节码修改能力进行完善。

### dexter

dexter 工具是 google 开发的一个类似 dexdump 的工具，但其内部实现了对 dex 文件结构和字节码指令的一套完整的操作 api，轻量简洁，对字节码的操作可以达到 ASM 的体验。

**综合，选用 dexter 对 dex 进行操作。**

方案设计
====

需求
--

根据性能防劣化和流量统计的需求，都是在一个方法的方法体内部前后插入对其他方法的调用。以网络流量统计为例，需要在 `okhttp3.RealCall.getResponseWithInterceptorChain` 的方法内部开头插入一个方法来获取 request 请求的详细数据。

```
Response getResponseWithInterceptorChain() throws IOException {

    com.netflow.inject.hookRealCall(this);//插入的方法

    List<Interceptor> interceptors = new ArrayList<>();

    interceptors.addAll(client.interceptors());

          //.....省略部分代码

    return chain.proceed(originalRequest);

  }
```

Dex 插桩  

基本流程
----

### 1. Dex 文件分析

先要分析 Dex 文件格式，将其序列化成各种数据结构，Dex 文件的结构可以参照官方文档（见参考文献）

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTEXbOgBlU1KgC2ZicVtFHUp05HwWP2HrTZokI9CP0Q4RHrecHOfvlUJicA/640?wx_fmt=png)

2. 字节码解析  

在 code 段将二进制的字节码解析成可处理的数据结构

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTEJIMCjuRiaxAApoHMxBLXn4eySNMvu8dvcY5nrRVF0PicbnP5hR9QuBdg/640?wx_fmt=png)

3. 字节码构造  

按照字节码规范构造字节码指令，并插入到现有字节码的序列中即可完成字节码的插入。

### 4. 字节码序列化

将修改后的 Dex 结构重新计算 Index，然后将各个数据 Section 序列化为 Dex 的文件格式。

功能需求
----

插桩支持两种能力，在一个方法的方法体前面和后面插入一个静态方法调用。

### 1. 方法体前面插桩

如果被插入的方法为实例方法，则方法的第一参数为 `this`，随后的参数和被插入的方法一致 ，如果方法是静态方法则插入的方法定义需要和被插入的方法参数类型和个数一致，举例：

```
public class Tracer{

        //被插入的方法，为实例方法

        private void MethodA(int a,int b){

        }


        //被插入的方法，为静态方法

        private static void MethodB(int a,int b){

        }

}


public class Hooker{

        //插入的方法

        private static void TestHookA(Tracer this_,int a,int b){

        }


        private static void TestHookB(int a,int b){

        }

}


////////插入后/////////

public class Tracer{

        private void MethodA(int a,int b){

                Hooker.TestHookA(this,a,b);

                //......

        }

        private static void MethodB(int a,int b){

                Hooker.TestHookB(a,b);

                //.......
        }

}
```

### 2. 方法体后面插桩

需要注意的是返回值的处理，插入的方法的返回值需要和被插入方法的返回值类型一致。参数的处理需要注意，插入的方法需要符合以下规则：

`方法名(this,被插入的方法参数,返回值类型)`

举例：

```
public class Tracer{

        //被插入的方法

        private void MethodA(int a, int b){

            //......

        }

        private String MethodB(int a, int b){

            //......

            return str;

        }

}

public class Hooker{

        private static void TestHookA(Tracer this_,int a,int b){}
   
        private static String TestHookB(Tracer this_,int a,int b,String return_val){

            //return_val 参数的值为原方法的真是返回值

            return return_val;

        }

}


////////插入后/////////


public class Tracer{

        private void MethodA(int a, int b){

            //......

            Hooker.TestHookA(this, a, b);

        }

        private String MethodB(int a, int b){

            //......

            return Hooker.TestHookB(this, a, b, str);

        }

}
```

3. 初始化插桩

一般用来插入一些需要提前初始化的代码，该功能会解析`AndroidManifest.xml` 里`application` 节点里定义的 Application 类。

根据配置在 `OnCreate` 或者 `attachBaseContext` 方法里插入代码。如果没有定义 OnCreate 和 attachBaseContext 方法，插桩工具会生成这两个方法。

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTENvSRjibwTQwZpHRxJPeuoBvbdqdPSRusQRkkPLglxXuvpSw7KHrVrAg/640?wx_fmt=png)

常见问题处理  

由于 Dex 在格式和指令上的一些限制，在修改和插入字节码的过程中需要符合 Dex 和 dalvik 指令了一些规则，下面描述了直接操作 Dex 遇到的一些问题和解决方法。

方法数处理
-----

当代码量增大后，由于 Google 早年设计缺陷，一个 DEX 文件只能容纳 65535 个方法、方法引用等，插桩本身不可避免会引入新的方法以及方法引用。在某些时候会有如下情况，APP 的某个 dex 文件非常迫近 65k，导致无法再插入新的方法调用，这种情况在大多数 app 中常见。

一种方案是将 Dex 整体合并在一起，然后进行拆分，此种方法会破坏原有 Dex 的一些优化，并且需要实现类之间的应用关系计算，计算量比较大，这里采用一种轻量的解决方法。

### Dex 拆包

#### 解决方案 1：

通过编译时增加 `--set-max-idx-number`迫使编译器尽量不要塞满 dex，但是这种方案可能不会生效，如果这个 apk 被类似 redex 的工具处理后，dex 也有概率会被填满。

#### 解决方案 2：Dex 分拆逻辑

如果当前 dex 的方法数剩余量不满足插入新的方法则将现有 dex 拆出一部分类出来到一个额外的 dex 中。

以第一个 dex 的编译逻辑为例，在将 maindex list 和其引用的类都塞到 dex 后，一般方法数不会刚好到 65535，如果超过了在编译的过程中就会出现`Too many classes in --``main-dex``-list` 的错误。然后编译器会将一些引用关系比较小的类填入第一个 dex 中。这些类就是我们要拆分的目标。

主要找到这个 dex 里没有调用到的类就满足目标了，通过遍历所有方法调用、属性引用、类引用的位置将所有类的引用过滤出来，可以将没有调用到的类过滤出来，拆分到其他的 dex 中。

主要逻辑：

1. 判断该 dex 的方法数是否可以继续插桩，如果无法进行插桩则需要进行 dex 分割逻辑

2. 遍历每个类的每个方法的参数，记录类型

3. 遍历每个类的属性，记录类型

4. 遍历每个方法的字节码指令，通过方法调用，属性引用，类型强转的指令将引用的类型记录下来

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTE6GgfDnk1sOT5xH8HjZcGduwnJ2FuWnNgMP8bB5LtVEFnyueudZYSGg/640?wx_fmt=png)

字节码格式 ID 为 22c 21c 31c 35c 3rc 的指令在最后的操作数都是一个类的方法或者属性的引用，就可以将这个方法使用的类获取到。

5. 将所有 interface annotation 保留在原 dex 中

6. 剩下的 class 就是这个 dex 中没有使用到的 class，可以将其拆分出去而对这个 dex 的执行不产生影响。

7. 将没有用到的 class 单独打包到一个额外的 dex 中，比如 现有 dex 有四个，则创建一个新的 dex 来保存。

这样被插入的 dex 就会省出一部分方法空间可以继续插桩。

### Dex 合并

#### 1. 分割 dex 合并

在 Dex 分割完成后，dex 分为两部分，我们需要将分割出来的 dex 合并成一个 dex 附加到最后一个 dex 上面。

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTEPPdyJgLa6oMqlw3Xjlas0Xf2dkGHiapNaxn5J67C2iaWv7uic6sBOD6tg/640?wx_fmt=png)

如上图，classes.split.dex、classes3.split.dex、...... classes9.split.dex 会合并成同一个 dex 为`classes11.dex`

#### 2. 插桩 dex 合并

插桩方法调用的代码一般不会打包到 apk 中，需要将代码 merge 到 apk 中。这里直接将需要插入的 dex 合并到最后一个 dex 上，如果最后一个 dex 无法合并则创建一个新的 dex 合并进去。

String Jumbo 处理
---------------

在 Dalvik 字节码中从常量池中读取字符串到寄存器里有两个指令，`const-string vAA, string@BBBB` 和 `const-string/jumbo vAA, string@BBBBBBBB` ，第一条指令只支持访问 0-0xFFFF 范围的字符串，由于我们插入了新的方法调用，会新增字符串 (类名、方法名) 进去，在很多情况下会导致字符串总量超过 65535，由于 Dex 格式要求必须使用 UTF-16 代码点值按字符串内容进行排序，所以在插入新的字符串之后要进行重排序，重新排序之后会导致原先的字符串索引发生变化，引起原本使用 const-string 的指令访问到高于 0xFFFF 索引的字符串，引起虚拟机执行异常。

插桩工具对此做了处理，在插桩完成后会扫描所有 `const-string vAA, string@BBBB` 指令，如果访问的 string index 超过 65535 则强制将 `const-string` 修改为 `const-string/jumbo` 指令。

混淆处理
----

### 目标方法被混淆

大部分情况下，目标方法都有比较大的概率会被混淆，所以我们在插桩的时候要基于 mapping 文件找到混淆后的目标函数然后进行插桩。

### 插入的 dex 使用了原 APK 中的类

很多情况下插桩方法调用到我们插入的 dex 都有可能使用到原来 apk 里提供的类，由于原 apk 里的类经历过混淆所以直接通过混淆前的名称调用会出现类、方法、属性无法找到的异常。

通过 mapping 文件将插入的 dex 里类名、方法名、属性名进行一次混淆，将调用方强行修改成混淆后的名称。

### 类被删除，方法内联 / 被删除

1. 优先考虑在原 apk 编译的过程中增加混淆配置去解决。

2. 如果调用的类和原 apk 逻辑关联不是很大，则建议将使用到的类包名重命名，然后一起打入到 dex 中，这样会表现为 apk 中存在相同的类，但是包名不一致，插入的 dex 只调用自己集成的类，这样就不用关心这个类的混淆问题。

3. 很多情况下是需要使用到原 apk 的类，无法通过重命名包名来解决，比如通过参数传入的类，在调用这些类的方法的时候可能会出现这个方法被混淆器删除掉的情况，有可能是被内联或者没有其他位置使用到从而被删除，那么在调用过程中尽量避开调用方法。

4. 有部分情况一些属性的 get set 方法会被内联成直接访问属性的情况

混淆前：

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTE8rlzQYhqDg4fd6d4WkH7icEOx2MsrwJsGQ84RdUSd3pB0sNyC4W4X4w/640?wx_fmt=png)

混淆后：

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTE5byk7ZrHT5ooyduedJybn6Hy7MxyXhQLeLKTJRCicF5Qzo5llRo7Xaw/640?wx_fmt=png)

为了避免这种情况尽量在调用 get set 方法的时候直接使用属性访问。  

比如：

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTEeQWI3MjQhSiaCxv4yfqoLcsUXianzbSniamDVxAkicDWJhZVIEFEME3W6Q/640?wx_fmt=png)

如果这个 get set 方法没有被内联掉，那么会出现调用的属性是是 private 和 protected 则导致 fileld 验证不通过，出现`java.lang.IllegalAccessError: Field 'xxxx' is inaccessible to class` 错误，解决方法是强行把被调用的属性权限改成 public。需要提前指定要修改了哪些属性的访问权限。这些配置在一个配置文件里进行设置，后面会说明如何设置。

类重复问题处理
-------

大部分情况下我们会遇到插入的 Dex 与被插入的 APK 存在相同类名的类的问题，比如调用了共同的第三方库，这里最常见遇到的是使用 Kotlin 编写的插入的 dex，里面会存在 kotlin 库。

这里有两种解决方法：

### **1. 剔除插入的 Dex 里的重复类**

在制作插入 Dex 的时候使用 Dex 插桩工具的按包名提取类的功能，将需要的类裁剪出来做成 dex，这个时候就可以将一些与 APK 重复的类剔除出去，插入进去的 Dex 使用 APK 自身的库，这个时候需要将插入的 Dex 按照 mapping 进行混淆才能够正常进行调用。

### **2. 重命名冲突的第三方库**

将自身调用的重复类按照包名整体重名名调用。比如 kotlin 包重命名成 kotlin_copy ，这样自己的 dex 调用的是 kotlin_copy.xxxx 就与原 apk 不冲突了。

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTEID8fpibdibQXzhp85ic9rYVmNsoXAM4W0t5hWuWiaWfjziaaXyPKfCbibAtA/640?wx_fmt=png)

字节码插桩
-----

### 方法前插桩

在方法前面增加一条 `invoke-static/range {}` 的指令，将原方法的参数透传到 hook 方法中

```
.method public static monitorEvent(Ljava/lang/String;Lorg/json/JSONObject;Lorg/json/JSONObject;Lorg/json/JSONObject;)V

    .registers 9

    //插桩代码

    invoke-static/range {p0 .. p3}, Lcom/bytedance/apm_bypass_tool/monitor/BypassMonitor;->monitorEvent(Ljava/lang/String;Lorg/json/JSONObject;Lorg/json/JSONObject;Lorg/json/JSONObject;)V



    const/4 v0, 0x4

    ....
```

方法后插桩

1. 查找所有 return 指令，在执行前面进行插桩

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTErzAicujOgGVum4yAM07apIaZ3eibyTzJylF7rknlcfCiapuGzImQv1Dgg/640?wx_fmt=png)

2. 返回值处理

由于要将返回值通过参数传递给 hook 方法使用，所以需要申请一个寄存器保留返回值的结果然后传递过去。

除 `return-void` 指令以外，其他 return 指令都附带一个返回值，如下：

```
invoke-direct {p2, p0, p1}, Lcom/ss/android/lark/ico$1;-><init>(Lcom/ss/android/lark/ico;Ljava/lang/reflect/Type;)V

return-object v4
```

将 p2 寄存器里的值保存到一个额外的寄存器里，然后获取 hook 方法的返回值，再返回回去

```
invoke-direct {p2, p0, p1}, Lcom/ss/android/lark/ico$1;-><init>(Lcom/ss/android/lark/ico;Ljava/lang/reflect/Type;)V

move-result-object v4

invoke-static {p0, p1, p2, v4}, Lcom/netflow/inject/NetFlowHookReceiver;->hookCallServerInterceptor_executeCall_end(Lcom/ss/android/lark/ici;Lcom/ss/android/lark/idj;Lcom/ss/android/lark/icy;Lcom/ss/android/lark/idi;)Lcom/ss/android/lark/idi;

move-result-object v5 //如果不对返回值做修改的话这里可以直接使用v4

return-object v5
```

#### 参数寄存器复用问题

在某些情况下，编译器在返回一个值的时候为了复用寄存器，会复用参数寄存器来作为通用寄存器，这就导致我们在方法后面获取参数的时候，发现这个参数寄存器被复用了，就无法正确获取到参数的值。

函数中引入的参数命名从 p0 开始，依次递增。举例一个方法会用到 v0，v1，p0，p1，p2 这五个寄存器，v0 和 v1 表示局部变量寄存器，如果是实例方法的话，p0 表示的是被传入的 this 对象的引用，p1 和 p2 分别表示两个传入的参数。

比如下面，就复用了 p1 寄存器来保存返回值，导致我们插桩方法无法获取到正确的`p1`参数

```
    invoke-interface {p1, p2}, Lcom/ss/android/lark/idf;->a(Lcom/ss/android/lark/idh;)Lcom/ss/android/lark/idj

    move-result-object p1

    return-object p1
```

在原有寄存器数量上面扩展对应参数数量的寄存器即可解决这个问题，比如

```
一个方法寄存器布局如下

v0 v1 v2 p0 p1 p2

在当前字节码中复用了p1寄存器。


扩展当前同参数数量的寄存器之后，寄存器布局如下：

v0 v1 v2 v3 v4 v5 p0 p1 p2

原字节码引用p1的位置变成了v4,以上面的例子来说就是字节码变成了如下形态：


invoke-interface {p1, p2}, Lcom/ss/android/lark/idf;->a(Lcom/ss/android/lark/idh;)Lcom/ss/android/lark/idj

move-result-object v4

return-object v4


这样就防止参数寄存器被复用
```

寄存器扩展问题  

在扩展寄存器的时候会遇到指令异常的问题，主要原因是寄存器数量扩展过多超过 16 个导致的，原字节码的寄存器使用可以保证寄存器的正确使用，在插入的时候也要保证寄存器的正确。

> 在实践中，一个方法需要 16 个以上的寄存器不太常见，而需要 8 个以上的寄存器却相当普遍，因此很多指令仅限于寻址前 16 个寄存器。在合理的可能情况下，指令允许引用最多前 256 个寄存器。此外，某些指令还具有允许更多寄存器的变体，包括可寻址 `v0` - `v65535` 范围内的寄存器的一对 catch-all `move` 指令。如果指令变体不能用于寻址所需的寄存器，寄存器内容会（在运算前）从原始寄存器移动到低位寄存器和 / 或（在运算后）从低位结果寄存器移动到高位寄存器。

> 例如，在指令 “`move-wide/from16 vAA, vBBBB`” 中：
> 
> “`move`” 为基础运算码，表示基础运算（移动寄存器的值）。
> 
> “`wide`” 为名称后缀，表示指令对宽（64 位）数据进行运算。
> 
> “`from16`” 为运算码后缀，表示具有 16 位寄存器引用源的变体。
> 
> “`vAA`” 为目标寄存器（隐含在运算中；并且，规定目标参数始终在前），取值范围为 `v0` - `v255`。
> 
> “`vBBBB`” 是源寄存器，取值范围为 `v0` - `v65535`。

比如在使用超过 v16 的寄存器的时候，要将`move-object vA, vB` 指令转换为`move-object/from16 vAA, vBBBB` 或者 `move-object/16 vAAAA, vBBBB`

插桩 Dex 制作
---------

插桩 Dex 是我们要额外插入到 APK 里的 dex，也就是插桩代码调用到的代码。

### 生成 Dex

举个例子，将需要插入的代码单独放到一个 gradle module 中

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTEiblPTBRLXfhVicZIQQEYtZNZicxianUwAicgCicrSf3KoWD001VDzIymkAicA/640?wx_fmt=png)

编译完成后解压 aar，取出 jar 包，通过 d8 命令将 java 字节码转成 dex

```
mkdir resources

./gradlew inject-dex:clean

./gradlew inject-dex:assembleRelease

d8 inject-dex/build/intermediates/aar_main_jar/release/classes.jar --output resources/

mv resources/classes.dex resources/netflow_caller.dex

mv resources/netflow_caller.dex netflow_caller.dex
```

方案 1：抽取插桩类  

由于编译完成后一般会将一些系统库和与原 APK 重复的第三方库打包进去，所以需要将这些系统库或者第三方库过滤掉。

工具提供了一个根据包名抽取类的功能，可以将指定包名的类单独拆成一个 dex。

抽取前：

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTEQZjxMUVsseYsJm9qxNLENnyURD1Wfyu5EESWP3mfiaUaot46sh1KOFg/640?wx_fmt=png)

抽取后：

![](https://mmbiz.qpic.cn/mmbiz_png/cZycwcLIiajrWX8h5icnUQxpkuRsbAryTEPHrVDbGZwsQIgav1ThM2bIUDFx929oIrVRt7EPzjBKWiaib2w2vhxAxw/640?wx_fmt=png)

### 方案 2：将重复的第三方库重命名

可以将使用的第三方库使用重命名功能进行重命名，这样做比使用 APK 里类的好处就是可以解决第三方库的版本问题和混淆问题。

### **参考文献**

1.redex:

https://github.com/facebook/redex/blob/master/opt/instrument/Instrument.cpp

2.dexter:

 https://android.googlesource.com/platform/tools/dexter/+/refs/heads/master

3. Dalvik 可执行文件格式: 

https://source.android.google.cn/devices/tech/dalvik/dex-format?hl=zh-cn

4. --set-max-idx-number：

https://stackoverflow.com/questions/27631500/is-there-a-way-to-limit-method-amount-in-main-dex-file-while-using-multidex-feat/27766126

5. Dalvik 可执行指令格式：

https://source.android.com/devices/tech/dalvik/instruction-formats