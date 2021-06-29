> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMjk5Mjk3Mw==&mid=2247497416&idx=1&sn=7e1ff71fe5d61434762b0bbf3bd0a471&chksm=9ac348eaadb4c1fcd07b903a151f466689ca8b92e305fda97c0c3b30a82caf329cae6a307c13&mpshare=1&scene=1&srcid=0629e0Tdl2yaZb5qaHkxv1Wx&sharer_sharetime=1624940202866&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

看到一款基于多端的 UI 调试工具，一套代码适应多端，真的是太棒了，下面分享给大家。  

1

前言

```
fun greet() = listOf("Hello", "Hallo", "Hola", "Servus").random()

renderComposable("greetingContainer") {
    var greeting by remember { mutableStateOf(greet()) }
    Button(attrs = { onClick { greeting = greet() } }) {
        Text(greeting)
    }
}
Result: Servus

```

2

使用 Compose for Web 构建用户界面

*   通过 DOM 元素和 HTML 标签表达设计和布局
    
*   使用类型安全的 HTML DSL 构建 UI 表示形式
    
*   通过使用类型安全的 CSS DSL 创建样式表来完全控制应用程序的外观
    
*   通过 DOM 子树与其他 JavaScript 库集成
    

```
fun main() {
    renderComposable("root") {
        var platform by remember { mutableStateOf("a platform") }
        P {
            Text("Welcome to Compose for $platform! ")
            Button(attrs = { onClick { platform = "Web" } }) {
                Text("...for what?")
            }
        }
        A("https://www.jetbrains.com/lp/compose-web") {
            Text("Learn more!")
        }
    }
}

```

![](https://mmbiz.qpic.cn/mmbiz_gif/YscRq4LT5WuQFjZJeF6PW9F4vBKgv1icPIBkSsDxM06thnA3IeltD81zoVl1kHG19bP9pwJeWkEIicCiceSy5G6kw/640?wx_fmt=gif)

**具有 Web 支持的多平台小部件**

*   通过利用 Kotlin 的 Expect-actual 机制来提供特定于平台的实现，从而使用和构建可在 Android、桌面和 Web 上运行的 Compose 小部件
    
*   处于实验性阶段的一组布局原语 (layout primitives) 和 API，这些原语和 API 与 Compose for Desktop/Android 的功能相似
    

3

示例代码

**使用 Composable DOM 编写简单的计数器**

```
fun main() {
    val count = mutableStateOf(0)
    renderComposable(rootElementId = "root") {
        Button(attrs = {
            onClick { count.value = count.value - 1 }
        }) {
            Text("-")
        }
        Span(style = { padding(15.px) }) { /* we use inline style here */
            Text("${count.value}")
        }
        Button(attrs = {
            onClick { count.value = count.value + 1 }
        }) {
            Text("+")
        }
    }
}

```

**声明和使用样式表**

```
object MyStyleSheet : StyleSheet() {
    val container by style { /* define a class `container` */
        border(1.px, LineStyle.Solid, Color.RGB(255, 0, 0))
    }
}

@Composable
fun MyComponent() {
    Div(attrs = {
        classes(MyStyleSheet.container) /* use `container` class */
    }) {
        Text("Hello world!")
    }
}

fun main() {
    renderComposable(rootElementId = "root") {
        Style(MyStyleSheet) /* mount the stylesheet */
        MyComponent()
    }
}

```

**声明和使用 CSS 变量**

```
object MyVariables : CSSVariables {
    val contentBackgroundColor by variable<Color>() /* declare a variable */
}

object MyStyleSheet: StyleSheet() {
    val container by style {
        MyVariables.contentBackgroundColor(Color("blue")) /* set its value */
    }

    val content by style {
        backgroundColor(MyVariables.contentBackgroundColor.value()) /* use it */
    }
}

@Composable
fun MyComponent() {
    Div(attrs = {
        classes(MyStyleSheet.container)
    }) {
        Span(attrs = {
            classes(MyStyleSheet.content)
        }) {
            Text("Hello world!")
        }
    }
}

```

来源：

https://mp.weixin.qq.com/s/8iabDTcw99kEa4i52w-XCQ