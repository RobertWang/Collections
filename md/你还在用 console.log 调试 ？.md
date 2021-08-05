> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488334&idx=2&sn=0763b56e8ab87663746f876c26f3de13&chksm=ce0da4cdf97a2ddbd47648f5104b7ac340572ad280c4a0707a15025809f3bbc97c5b459cfeec&scene=21#wechat_redirect)

英文 | Giancarlo Buomprisco 

译文 | 梁天培

链接 | juejin.im/post/5d18d6eb6fb9a07edc0b6cc4

**前言：Chrome 开发工具**  

当您的代码没有按照预期执行的时候，您是否还在用 console.log 来进行调试？如果是，那这篇文章就是为您准备的。

我写这篇文章的目的是让您了解 Chrome 开发工具提供的高效工具，让您可以更好、更快地调试 Javascript 代码。

本文主要讲述以下几点内容：

*   设置断点以调试特定行的代码
    
*   查看调用堆栈
    
*   暂停 / 恢复脚本执行
    
*   设置表达式
    
*   开发工具的生产力提示和技巧
    

**调试运行时代码**

当代码出现 bug 或没有按照预期执行时，我们通常会查看开发者工具中的 Sources 选项卡，接下来我们将通过不同场景来深入了解这个功能模块。

![](https://mmbiz.qpic.cn/mmbiz_jpg/meG6Vo0MeviabskVPvktXjP8XpibITl94aibHpaCJBm0Z2CLJwCDVxmzFAYzicLkrbUo03FMc1nPicbSicC8l1kqCI1w/640?wx_fmt=jpeg)

Sources 选项卡

#### 断点

在阅读本文之前，您可能习惯于使用控制台打印某个值来调试代码。但我希望向您介绍一种更高效的方法，一种能更深入代码中的方法：断点。

设置断点通常是调试过程的第一步。虽然目前大多数浏览器中的内置开发工具，都允许您调试正在浏览的页面，停止在特定代码行上或者在特定语句上执行代码，但在本文中，我们将主要讲解 Chrome 开发者工具。

##### 什么是断点？

通常，您可能希望停止执行代码，以便您可以逐行地查看特定的上下文。

一旦代码在断点处停止，我们就可以通过访问作用域，查看调用堆栈，甚至在运行时更改代码来进行调试。

##### 如何设置断点？

由于使用哪种前端技术对调试来说并不重要，为了更方便地向您解释断点，我将调试用于培训的一个 Angular 项目。

*   首先，打开开发工具并转到 Sources 选项卡
    
*   然后，打开我们要调试的文件
    
*   打开文件后，我们可以通过单击需要停止的那行代码来设置断点
    

小提示：在 Mac 上，使用快捷键 ⌘ + O 可以打开文件选择器，您可以在其中找到您需要调试的文件。在 Windows 上，可以使用 CTRL + O

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94aSuyoMoXHkTdhHWSya9fq9ObZu8wEsvarqXENQgAk5uWYW9WM1oCs2A/640?wx_fmt=gif)

设置断点

如上图所示，我们可以在一行代码上更深入地设置断点，例如在一行代码里的不同语句。

我们设置了 3 个断点：

*   第一个断点在代码定义时停止执行
    
*   第二个断点将在 priceReceived 函数执行之前停止
    
*   第三个断点将在 priceReceived 被调用后立即停止，因此我们也可以检查箭头函数的返回值
    

当调用箭头函数时，执行停止，右侧面板 Scope 将显示当前的上下文，并允许我们访问所有我们想查看的值。

如下图所示，我们可以看到变量 price 的值 。

![](https://mmbiz.qpic.cn/mmbiz_jpg/meG6Vo0MeviabskVPvktXjP8XpibITl94ac3f0yd4s4mVvZtGUvJhWIkVehB89bVzQh5J23Dmh7XDWjJ99uRNMEQ/640?wx_fmt=jpeg)

查看当前作用域

在下图中，一旦 priceReceived 执行，第三个断点就会被使用。

在右侧面板中您可以使用 Return value 查看匿名函数的返回值。

![](https://mmbiz.qpic.cn/mmbiz_jpg/meG6Vo0MeviabskVPvktXjP8XpibITl94asaOmyRAgiaMETJnW6BWaAfuR41UMy1PBekfRwbvWBKxHfQGfcK8j1ZQ/640?wx_fmt=jpeg)

查看匿名函数返回值

##### 临时取消断点

场景：您在代码中设置了一堆断点。

在调试时，多次刷新页面是很常见的操作。

您当前正在调试的代码可能有各种断点，有时候甚至会达到几百个。这几百个断点可能会浪费您大量的时间。  
在这种情况下，可以暂时暂停所有断点的执行，您可以通过切换下图中的图标来操作：

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94aF7Rgfby5OrjibahUaKInOJdE5BX9NDOxPZGzp0GW6gjnq6DHodSaObQ/640?wx_fmt=gif)

取消断点

##### 执行错误时停止

场景：您的代码执行产生了错误，但您不想设置断点，因为您不知道何时会抛出错误。

在您的代码中抛出错误，这样就可以查看代码出现了什么问题。

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94aWLCINbOwiaRiblywkq6a0KVJ0Jic3iaLhJJsaZaPBHNReG2QEVEXIvy07g/640?wx_fmt=gif)

报错时暂停

##### 条件断点

顾名思义，条件断点就是仅在条件为真时触发的断点。

例如，在上面的示例中，用户可以在文本区域中输入非数值。由于 JS 的兼容性只会显示 NaN 而不是抛出错误。

场景：您的代码比上面的代码更复杂，并且无法确定何时出现 NaN 。

当然，您可以设置一个断点，但复现错误并不容易，可能最终花费半小时来执行代码。在这种情况下，您可以使用条件断点，并仅在出现 NaN 时停止执行代码。

如下图：

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94aodIzpY0rEe4QUMK4HqhPocqPEBZWBGPvvuSiadHZEoyOVqeksdQObRw/640?wx_fmt=gif)

条件断点  

*   右键单击要添加断点的代码行
    
*   单击 “Add conditional breakpoint…”
    
*   添加有效的 JS 表达式。当然，在调用表达式时，您可以引用参数 x 和 y
    
*   当表达式为真时，断点将被触发
    

##### 单步执行代码

为了充分利用 Dev Tools，值得花一点时间学习开发工具如何帮助我们快速单步执行代码，而无需在每一行设置断点。

使用 Dev Tools 中的 navigator 可以顺序逐行执行代码。

我将在下面介绍 Step over next function call 与 Step 的不同。在调试异步代码时，点击 Step 按钮将按时间顺序移动到下一行。

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94aroNf2vgZs2C4xAmbC4mKtepuuKy1WkViawo5kfIvaPVtibM3FQdUxfOw/640?wx_fmt=gif)

Step  

##### 跳过下一个函数调用

Step over next function call 按钮也会顺序执行代码，但不会进入函数调用。也就是说，函数调用将被跳过，除非您在函数中设置了断点，否则调试器将不会在该函数中停止。

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94a4icarvbLBNYtgvadnAkQv3kY2D1fcibNfcL25zps3RM0GjibrrqsN6sYA/640?wx_fmt=gif)

Step over next function call

如果您仔细观察上图，会发现 multiplyBy 和 renderToDOM 这两个方法执行时没有像 Step 那样进入函数内部。

##### 进入下一个函数调用

自 Chrome 68 以来，Step Into Next function call 按钮的作用发生了改变。它类似于上面提到的 Step 。不同之处在于，当进入异步代码时，它将停止在异步代码中，而不是按时间顺序运行的代码

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94aiaqWmT1wg0mCRhx3PPPfCnN3UauswibJzgx9dynD4QgpPzqrD4iaFNqgA/640?wx_fmt=gif)

Step Into Next function call  

如上图所示：如果按时间顺序，第 32 行应该已经运行，但事实并非如此。调试器在等待 2 秒后才移动到第 29 行

###### 退出函数调用

假设调试代码时，您不想进入某个函数的内部，Step Out of function call 允许您退出函数并在函数调用后的下一行停止。

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94aAJqT4azBYqYoYCZt0t0n34iaaupZVibsvLpSpGicYe500MkIPH5FYJtGg/640?wx_fmt=gif)

Step Out of function call

上图中发生了什么？

*   代码在第 36 行的断点停了下来
    
*   然后跳出了函数 renderToDOM
    
*   调试器直接移到第 29 行并跳过 renderToDOM 函数的剩余部分
    

##### 全局变量和即时输出

有时，在全局范围内存储某些值（例如组件类，大型数组或复杂对象）会非常有用。

例如，当您想要传入不同的参数调到某个组件的方法时，在调试过程中将这些参数添加到全局范围可以节省大量时间。

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94auEdlUnP6fm73zbYRcwH1ul1fJib5ILG3RAaS5o9icVt22xwVyJzWo58g/640?wx_fmt=gif)

添加一个全局变量到当前作用域  

在上图中，我将数组 [previous, current] 存为全局变量。开发者工具会自动分配一个名为 temp{n} 的变量，n 基于先前保存的变量的数目。

如上图所示，变量被命名 temp2，您可以在控制台中使用它，因为它现在已是一个全局变量了！

即时输出是 Chrome 68 中发布的一项功能，开发工具允许您在输入代码时在控制台中显示执行的结果。

如果您仔细观察上图会发现，当我将保存的变量映射到字符串数组时，我没有按下 Enter 键，但结果立即显示在下一行。

#### 查看调用堆栈

查看调用堆栈是开发者工具提供的最有用的工具之一：您不仅可以在调用它们的函数中来回跳转，还可以在每个步骤检查它们的作用域。

假设我们有一个简单页面和一个输入数字的脚本，并在页面上呈现数字乘以 10. 我们将调用两个函数：一个用来做乘法，一个用来将结果渲染到页面中。

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MeviabskVPvktXjP8XpibITl94aTDerJzrNBJlaHibmP1ElscWzic8E8dCheMIUHL2icNrcOdIiaATzhdjaPg/640?wx_fmt=gif)

查看调用堆栈  

如上图所示，只需单击 “Call Stack” 窗格中的函数名称，我们就可以浏览它们的作用域。

如果您仔细观察会发现，每次我们从一个函数调用跳到另一个函数调用时，作用域都会保留，我们可以在这里对每一步进行分析！

##### Blackbox 脚本用于展平堆栈

Blackboxing 脚本将通过从堆栈中排除特定的脚本或某些匹配模式的脚本来过滤调用堆栈。

例如，如果我有 99% 的时间只调试 userland 中的代码感兴趣，我可以在 Blackbox 中添加一个模式，将 node_modules 文件夹下的所有脚本过滤掉。

要通过 Blackbox 过滤一个脚本，有两种方法：

*   右键单击 Sources 选项卡中的 JS 脚本，然后单击 “Blackbox Script”
    
*   转到 Chrome 设置页面，然后转到 Blackboxing 并单击 Add Pattern… 并输入您想要加入 Blackbox 的正则，在您想要过滤大量脚本时很有用。
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/meG6Vo0MeviabskVPvktXjP8XpibITl94a9jtjGia5TTHANom44fw0IgBs962qibFJ2NKU8rmQYAfr3GGEgUUCvNIQ/640?wx_fmt=jpeg)

过滤 node_modules 文件夹

##### 监视表达式

通过监视表达式，您可以定义一些 Javascript 语句，在开发者工具运行显示这些语句的结果。这是一个特别有趣的工具，因为您可以写任何您想要的虚拟情况，只要它是一个有效的 Javascript 表达式。

例如，您可以编写一个结果始终为 true 的表达式，当表达式结果为 false 时 ，您就可以发现当前的运行状态存在问题。

有一个需要注意问题：

*   当我们使用断点进行调试时，监视表达式将被立刻执行，不需要刷新页面
    
*   如果代码在正常运行时，则需要手动单击刷新按钮
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/meG6Vo0MeviabskVPvktXjP8XpibITl94ac0Kpo761YyGOrrmZvksIbbCDN3WaCicUHlib7yOep64ftjC8nq93xNgA/640?wx_fmt=jpeg)

监视表达式

#### 结语

浏览器开发者工具是调试复杂代码的利器。有时您可能需要比 console.log 更进一步的操作，上面提到的功能将提供深入代码底层的调试体验。这些工具需要一些练习才能完全掌握，所以如果您对部分功能还不熟悉，请不要放弃，继续坚持使用它们。