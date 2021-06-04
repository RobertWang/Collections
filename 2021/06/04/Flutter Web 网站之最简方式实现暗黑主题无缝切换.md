> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247506555&idx=4&sn=fb8053fcdfcffcfc9d3c393914f4cf91&chksm=e8797f01df0ef6179bfd2c480f7d2fd21f7b1b495e269ec3f9bfaf87841e7ab5a08f325cc0d1&mpshare=1&scene=1&srcid=0603UfTjb5TxNCBBR3mb8oc8&sharer_sharetime=1622729819562&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

往期
--

*   Flutter Web 网站搭建教程 
    

```
https://www.jianshu.com/p/cc1dcf3f5063

```

*   Flutter Web 网站之主页框架搭建
    

```
https://www.jianshu.com/p/fcd1bcd50fb2

```

*   Flutter Web 网站之 Jetpack 成型
    

```
https://www.jianshu.com/p/fd7ef411a642

```

*   Flutter Web 网站之 ScrollView+GridView 优化
    

```
https://www.jianshu.com/p/4759f7a05d1b

```

上期回顾
----

上期我们做了优化，主要针对 ScrollView+GridView 的使用场景，用了更加合适的组件，这期想做一个主题变更，为什么呢？

*   第一 暗黑主题越来越刚需化，现在哪个主流 App 没有暗黑都不好意思上架，而 ios 阵营更加强硬的要求平台实现，否则下架，库克牛逼，我们惹不起。
    
*   第二 项目还处于初期，这个时候重构改动成本较低
    
*   第三 主题的变更网上有很多框架可以快速实现，但我想寻求的是最简单的实现，不想引入别人的框架，一来自己了如指掌，二来不用依赖别人的升级来满足未来奇葩的需求。这期实现其实很简单，来往下看。
    

实现效果
----

大屏浅色![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQJHLMQlU42xnS63Iusva328BAOnUib7h04ZqacAoTBrQ4bTj6TqMXr4J5CV1tcp1LSYrkQ1LKedKw/640?wx_fmt=png)大屏暗黑![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQJHLMQlU42xnS63Iusva320EgDp2nghxdKPmgbSflVQxk7EFuDSYTJmEOp5T5VyXcDu5MI2hgQ9w/640?wx_fmt=png)小屏浅色![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQJHLMQlU42xnS63Iusva32PDsiatAGFkd03mAicUOKrXwK2Wjptz58okfTaLIJbvzINna61RaZK1Mg/640?wx_fmt=png)小屏暗黑![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQJHLMQlU42xnS63Iusva32ayBNoQ8tYACBicpuJUpvHnO9XnEBnAazcOgjzSHBic4icBbrhalTwe9nQ/640?wx_fmt=png)

代码实现
----

定义主题类 AppTheme，用来配置不同的 ThemeData

```
class AppTheme {

  ThemeData themeData;

  AppTheme(this.themeData);

  // ignore: non_constant_identifier_names
  static final AppTheme DARK_THEME = AppTheme(ThemeData.dark());

  // ignore: non_constant_identifier_names
  static final AppTheme LIGHT_THEME = AppTheme(
      ThemeData(brightness: Brightness.light, primaryColor: Colors.grey[50]));
}


```

DARK_THEME 暗黑主题 LIGHT_THEME 浅色主题、浅色标题栏默认蓝色，这里我改成浅灰色

定义 StreamController，用来动态变更数据

```
class ThemeBloc {
  // ignore: close_sinks
  final _themeStreamController = StreamController<AppTheme>();
  /// 变更主题调用方法
  get changeTheTheme => _themeStreamController.sink.add;
  /// 主题数据
  get themeData => _themeStreamController.stream;
}

final bloc = ThemeBloc();


```

给原来的 MaterialApp 包一层 StreamBuilder

```
 StreamBuilder<AppTheme>(
        initialData: AppTheme.LIGHT_THEME,
        stream: bloc.themeData,
        builder: (context, AsyncSnapshot<AppTheme> snapshot) {
          return MaterialApp(
            title: 'Jetpack',
            theme: snapshot.data.themeData,
            home: PageHome(),
            routes: <String, WidgetBuilder>{
              "/qq": (context) => PageQQLink(),
              "/pageChatGroup": (context) => PageChatGroup(),
            },
          );
        })


```

initialData 默认数据 stream 将 bloc.themeData 赋值给它，用来监听数据变化 snapshot 数据变化的快照，最终交给 MaterialApp 的 theme，从而实现动态变更。如何触发变更数据呢？![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQJHLMQlU42xnS63Iusva32DSgeLiaIUbomHxN9R3HibCJlB3w09pL0epL2O1ncWIkD8W7FJXt89vYQ/640?wx_fmt=png)如图设置页面添加了开关，代码如下

```
SwitchListTile(
          secondary: Icon(_isEnabled ? Icons.brightness_4 : Icons.brightness_5),
          title: Text("暗黑主题"),
          subtitle: Text("将主题调成暗黑色"),
          value: _isEnabled,
          onChanged: (value) {
            setState(() {
              _isEnabled = value;
            });
            if (value) {
              bloc.changeTheTheme(AppTheme.DARK_THEME);
            } else {
              bloc.changeTheTheme(AppTheme.LIGHT_THEME);
            }
          },
        )


```

这里调用 bloc.changeTheTheme 来变更主题。对变更主题就是这么简单，你是不是有疑问，我如何修改其他主题呢（如：文本，按钮，对话框等）

ThemeData
---------

再看下我得实现

```
  static final AppTheme LIGHT_THEME = AppTheme(
      ThemeData(brightness: Brightness.light, primaryColor: Colors.grey[50]));


```

我这里修改了 brightness、primaryColor，其实它还有很多，请看

```
 ThemeData({
    Brightness brightness,
    VisualDensity visualDensity,
    MaterialColor primarySwatch,
    Color primaryColor,
    Brightness primaryColorBrightness,
    Color primaryColorLight,
    Color primaryColorDark,
    Color accentColor,
    Brightness accentColorBrightness,
    Color canvasColor,
    Color scaffoldBackgroundColor,
    Color bottomAppBarColor,
    Color cardColor,
    Color dividerColor,
    Color focusColor,
    Color hoverColor,
    Color highlightColor,
    Color splashColor,
    InteractiveInkFeatureFactory splashFactory,
    Color selectedRowColor,
    Color unselectedWidgetColor,
    Color disabledColor,
    Color buttonColor,
    ButtonThemeData buttonTheme,
    ToggleButtonsThemeData toggleButtonsTheme,
    Color secondaryHeaderColor,
    Color textSelectionColor,
    Color cursorColor,
    Color textSelectionHandleColor,
    Color backgroundColor,
    Color dialogBackgroundColor,
    Color indicatorColor,
    Color hintColor,
    Color errorColor,
    Color toggleableActiveColor,
    String fontFamily,
    TextTheme textTheme,
    TextTheme primaryTextTheme,
    TextTheme accentTextTheme,
    InputDecorationTheme inputDecorationTheme,
    IconThemeData iconTheme,
    IconThemeData primaryIconTheme,
    IconThemeData accentIconTheme,
    SliderThemeData sliderTheme,
    TabBarTheme tabBarTheme,
    TooltipThemeData tooltipTheme,
    CardTheme cardTheme,
    ChipThemeData chipTheme,
    TargetPlatform platform,
    MaterialTapTargetSize materialTapTargetSize,
    bool applyElevationOverlayColor,
    PageTransitionsTheme pageTransitionsTheme,
    AppBarTheme appBarTheme,
    BottomAppBarTheme bottomAppBarTheme,
    ColorScheme colorScheme,
    DialogTheme dialogTheme,
    FloatingActionButtonThemeData floatingActionButtonTheme,
    NavigationRailThemeData navigationRailTheme,
    Typography typography,
    CupertinoThemeData cupertinoOverrideTheme,
    SnackBarThemeData snackBarTheme,
    BottomSheetThemeData bottomSheetTheme,
    PopupMenuThemeData popupMenuTheme,
    MaterialBannerThemeData bannerTheme,
    DividerThemeData dividerTheme,
    ButtonBarThemeData buttonBarTheme,
  }


```

这里面得主题你都可以修改，你是不是看到了 AppBarTheme、DialogTheme、TextTheme、BottomSheetThemeData 等等，不用我解释了吧，你应该知道是谁得样式了吧。

总结
--

20 几行新增得代码就搞定了，为什么还要用别人得框架呢？是吧。

啰嗦一句
----

计划真的是赶不上变化，不急，慢慢完善哈。

源码
--

请转 github 代码查看完整实现

```
https://github.com/ibaozi-cn/flutter-jetpack

```

ToDo  

-------

该部分内容后期慢慢给大家更新，请客观不要着急，当然你可以参与进来，私聊我就行哦。

*   示例部准备下期实现，跳转至详情页面，并展示用例。源码已经完成点击即可跳转至 github。
    
*   Tags 部分下期实现，这部分也需要新的 UI 展现，标签的功能类似与搜索，提供更快捷的方式查找想要的功能。
    
*   留言功能设计，在你们使用过程中肯定会有不同的建议，有了这个功能就能知道你们的心声，所以这也是我们需要的实现的一个功能。
    
*   优秀的项目推荐，有很多优秀的项目等待着我们去发现，我一个人的能力有限，如果有更多的人来推荐，会不断丰富我们的 Jetpack 内容。
    
*   博客，这里考虑到有很多优秀的大佬，写过相关技术博客，帮你寻找最优秀的资源。功能设计如下图新增按钮。![](https://mmbiz.qpic.cn/mmbiz_png/GIQWuu6gWEQJHLMQlU42xnS63Iusva32FkUIWJhc9SYlWibkSBnVhg205IrcUiayX6wT13TmCZjh0MicEH3a9qcWQ/640?wx_fmt=png)
    

结束
--

网站 jetpack.net.cn，欢迎常来，也希望能在你学习 Flutter 的道路上提供一丢丢的帮助。