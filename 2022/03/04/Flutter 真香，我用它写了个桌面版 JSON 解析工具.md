> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7069689952459554830) void fromJson(Map<String, dynamic>? json) { if (json == null) return; age = json['age']; name = json['name'] ?? ''; } 复制代码

数据量少还能接受，一旦量大，那么光手写这个解析方法都能让你怀疑人生。更何况手写还有出错的可能。好在官方有个工具 **[json_serializable](https://link.juejin.cn?target=https%3A%2F%2Fpub.flutter-io.cn%2Fpackages%2Fjson_serializable "https://pub.flutter-io.cn/packages/json_serializable")** 可以自动生成这块转换代码，也解决了 flutter 界 json 转模型的空缺。当然，业界也有专门解析 json 的网站，可以自动生成 dart 代码，使用者在生成后复制进项目中即可，也是非常方便的。

本项目以 json 解析为切入点，和大家一起来看下 flutter 是如何开发桌面应用的。

1、创建项目
======

要让我们的 flutter 项目支持桌面设备。我们首先需要修改下 flutter 的设置。如下，让我们的项目支持`windows`和`macos`系统。

```
flutter config --enable-windows-desktop
flutter config --enable-macos-desktop
复制代码
```

接下来使用`flutter create`命令创建我们的模版工程。

```
flutter create -t app --platforms macos,windows  hello_desktop
复制代码
```

创建完项目后，我们就可以`run`起来了。

2、功能介绍
======

先来看下整体界面，界面四块，分别为功能模块、文件选择模块、输入模块、输出模块。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61ed792e805b4044843558374ff79e99~tplv-k3u1fbpfcp-watermark.awebp?)

> 这里自动修正的功能能帮助我们将异常的格式不正确的 json 转为正确的格式，不过处于开发阶段，可以不必理会。

3、关键技术点 & 难点记录：
===============

1、控制窗口 Window
-------------

我们在新建一个桌面应用时，默认的模版又一个 Appbar，此时应用可以用鼠标拖拽移动，放大缩小，还可以缩到很小。但是，我们一旦去掉这个导航栏，那么窗口就不能用鼠标拖动了，并且我们往往不希望用户将我们的窗口缩放的很小，这会导致页面异常，一些重要信息都展示不全。因此这里需要借助第三方组件`bitsdojo_window`。通过`bitsdojo_window`，我们可以实现窗口的定制化，拖动，最小尺寸，最大尺寸，窗口边框，窗口顶部放大、缩小、关闭的按钮等。

2、鼠标移动捕捉
--------

通过`InkWell`组件，可以捕捉到手势、鼠标、触控笔的移动和停留位置

```
tip = InkWell(
    child: tip,
    hoverColor: Colors.white,
    highlightColor: Colors.white,
    splashColor: Colors.white,
    onHover: (value) {
      bool needChangeState = widget.showTip != value;
      if (needChangeState) {
        if (value) {
          // 鼠标在tip上，显示提示
          showTip(context, PointerHoverEvent());
        } else {
          overlay?.remove();
        }
      } 
      widget.showTip = value;
    },
    onTap: () {},
  );
复制代码
```

3、鼠标停在指定文字上时显示提示框，移开鼠标时隐藏提示框
----------------------------

这个功能是鼠标移动后的 UI 交互界面。要在窗口上显示一个提示框，可以使用`Overlay`。需要注意的是，由于在`Overlay`上的`text`的根结点不是`Material`风格的组件，因此会出现黄色的下划线。因此一定要用`Material`包一下`text`。并且你必须给创建的`OverlayEntry`一个位置，否则它将全屏显示。

```
Widget entry = const Text(
      '自动修复指输入的JSON格式不正确时，工具将根据正确的JSON格式自动为其补其确实内容。如“”、{}、:等',
      style: TextStyle(
        fontSize: 14,
        color: Colors.black
      ),
      );

entry = Material(
      child: entry,
    );

// ... 其他代码
OverlayEntry overlay = OverlayEntry(
          builder: (_) {
            return entry;
          },
        );
       Overlay.of(context)?.insert(overlay);

       this.overlay = overlay;
  }
复制代码
```

4、读取鼠标拖拽的文件
-----------

读取说表拖拽的文件一开始想尝试使用`InkWell`组件，但是这个组件无法识别拖拽中的鼠标，并且也无法从中拿到文件信息。因此放弃。后来从文章《[Flutter-2 天写个桌面端 APP](https://juejin.cn/post/7038410166361915423 "https://juejin.cn/post/7038410166361915423")》中发现一个可读取拖拽文件的组件`desktop_drop` ，能满足要求。

5、本地文件选取
--------

使用开源组件`file_picker` ，选完图片后的操作和拖拽选择图片后的操作一致。

6、TextField 显示富文本
-----------------

`Textfield`如果要显示富文本，那么需要自定义`TextEditingController`。并重写`buildTextSpan`方法。

```
class RichTextEditingController extends TextEditingController {

// ...

@override
  TextSpan buildTextSpan(
      {required BuildContext context,
      TextStyle? style,
      required bool withComposing}) {
    if (highlight) {
      TextSpan text;
      String? input = OutputManager().inputJSON;
      text = _serializer.formatRich(input) ?? const TextSpan();
      return text;
    }
    String json = value.text;
    return TextSpan(text: json, style: style);
  }
}
复制代码
```

7、导出文件报错
--------

在做导出功能时遇到下列报错，保存提示为没有权限访问对应目录下的文件。

```
flutter: path= /Users/zl/Library/Containers/com.example.jsonFormat/Data/Downloads
[ERROR:flutter/lib/ui/ui_dart_state.cc(209)] Unhandled Exception: FileSystemException: Cannot open file, path = '/Users/zl/Library/Containers/com.example.jsonFormat/Data/Downloads/my_format_json.json' (OS Error: Operation not permitted, errno = 1)
复制代码
```

通过 Apple 的开发文档找到有关权限问题的[说明](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Farchive%2Fdocumentation%2FMiscellaneous%2FReference%2FEntitlementKeyReference%2FChapters%2FEnablingAppSandbox.html "https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/EntitlementKeyReference/Chapters/EnablingAppSandbox.html")。其中有个授权私钥的 key 为`com.apple.security.files.downloads.read-write` ，表示**对用户的下载文件夹的读 / 写访问权限**。那么，使用 Xcode 打开 Flutter 项目中的 mac 应用，修改工程目录下的`DebugProfile.entitlements`文件，向`entitlements`文件中添加`com.apple.security.files.downloads.read-write`，并将值设置为 YES，保存后重启 Flutter 项目。发现已经可以向下载目录中读写文件了。

当然，这是正常操作。还有个骚操作就是关闭系统的沙盒机制。将`entitlements`文件的`App Sandbox`设置为 NO。这样我们就可以访问任意路径了。当然关闭应用的沙盒也就相当于关闭了应用的防护机制，因此这个选项慎用。

**TODO List：**

*   [ ]  json 自动修正
*   [ ]  模型代码高亮
*   [ ]  自定义导出路径

**参考文档：**

[Flutter 桌面支持](https://link.juejin.cn?target=https%3A%2F%2Fflutter.cn%2Fdesktop "https://flutter.cn/desktop")

[Flutter desktop support](https://link.juejin.cn?target=https%3A%2F%2Fflutter.dev%2Fmulti-platform%2Fdesktop "https://flutter.dev/multi-platform/desktop")

[Flutter-2 天写个桌面端 APP](https://juejin.cn/post/7038410166361915423 "https://juejin.cn/post/7038410166361915423")

[pub.dev-window](https://link.juejin.cn?target=https%3A%2F%2Fpub.flutter-io.cn%2Fpackages%3Fq%3Dwindow "https://pub.flutter-io.cn/packages?q=window")

[Flutter Desktop - bitsdojo-window - bilibili](https://link.juejin.cn?target=https%3A%2F%2Fwww.notion.so%2FFlutter-JSON-353936d2c7ff45449878e2103e92ca13 "https://www.notion.so/Flutter-JSON-353936d2c7ff45449878e2103e92ca13")

[Apple 开发权限文档](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Farchive%2Fdocumentation%2FMiscellaneous%2FReference%2FEntitlementKeyReference%2FChapters%2FEnablingAppSandbox.html "https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/EntitlementKeyReference/Chapters/EnablingAppSandbox.html")