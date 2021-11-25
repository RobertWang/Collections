> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7005562586843906085)*   从 Flutter v1.1.7 版本开始, Flutter module 仅支持 AndroidX 应用
*   在 release 模式下 Flutter 仅支持以下架构: x86_64,armeabi-v7a,arm64-v8a, 不支持 mips 和 x86, 所以引入 Flutter 前需要选取 Flutter 支持的架构

```
android{
  //...
  defaultConfig {
        //配置支持的动态库类型
        ndk {
            abiFilters 'x86_64','armeabi-v7a', 'arm64-v8a'
        }
    }
}

复制代码
```

混合开发的一些适用场景
-----------

*   在原有项目中加入 Flutter 页面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80aafb2aa7d24cce87cfbd2cca349ebb~tplv-k3u1fbpfcp-watermark.awebp)

*   原生页面中嵌入 Flutter 模块

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60486ecb5c4a415bbf02e131ba9afc1f~tplv-k3u1fbpfcp-watermark.awebp)

*   在 Flutter 项目中嵌入原生模块

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20fc379ec4fb4d59a2fbc6ac16e29b40~tplv-k3u1fbpfcp-watermark.awebp)

主要步骤
----

*   创建 Flutter module
*   为已存在的 Android 项目添加 Flutter module 依赖
*   早 Kotlin/Java 中调用 Flutter module
*   编写 Dart 代码
*   运行项目
*   热重启 / 重新加载
*   调试 Dart 代码
*   发布应用

请把所有的项目都放在同一个文件夹内

```
- WorkProject
    -  AndroidProject
    -  iOSProject
    -  flutrter_module

复制代码
```

WorkProject 下面分别是原生 Android 模块, 原生 iOS 模块, flutter 模块, 并且这三个模块是并列结构

创建 Flutter module
-----------------

在做混合开发之前我们需要创建一个 Flutter module 这个时候需要

```
cd xxx/WorkProject /

复制代码
```

创建 flutter_module

```
flutter create -t module flutter_module

复制代码
```

如果要指定包名

```
flutter create -t module --org com.example flutter_module
复制代码
```

然后就会创建成功

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae3ef3fde97e4588ac5d05792cc8c93a~tplv-k3u1fbpfcp-watermark.awebp)

*   .android - flutter_module 的 Android 宿主工程
*   .ios - flutter_module 的 iOS 宿主工程
*   lib - flutter_module 的 Dart 部分代码
*   pubspec.yaml - flutter_module 的项目依赖配置文件 因为宿主工程的存在, 我们这个 flutter_module 在布甲额外的配置的情况下是可以独立运行的, 通过安装了 Flutter 和 Dart 插件的 AndroidStudio 打开这个 flutter_module 项目, 通过运行按钮可以直接运行

构建 flutter aar(非必须)
-------------------

可以通过如下命令构建 aar

```
cd .android/
./gradlew flutter:assembleRelease
复制代码
```

这会在. android/Flutter/build/outputs/aar / 中生成一个 flutter-release.aar 归档文件

为已存在的 Android 用意添加 Flutter module 依赖
------------------------------------

打开我们的 Android 项目的 settings.gradle 添加如下代码

```
setBinding(new Binding([gradle: this]))                              
evaluate(new File(                                                 
        settingsDir.parentFile,                                      
        'flutter_module/.android/include_flutter.groovy'                     
))

//可选，主要作用是可以在当前AS的Project下显示flutter_module以方便查看和编写Dart代码
include ':flutter_module'
project(':flutter_module').projectDir = new File('../flutter_module')

复制代码
```

setBinding 与 evaluate 允许 Flutter 模块包括它自己在内的任何 Flutter 插件, 在 setting.gradle 中以类似: flutter package_info :video_player 的方式存在

添加: flutter 依赖
--------------

```
dependencies {
  implementation project(':flutter')
}

复制代码
```

添加 Java8 编译选项
-------------

因为 Flutter 的 Android engine 使用了 Java8 的特性, 所有在引入 Flutter 时需要配置你的项目的 Java8 编译选项

```
//在app的build.gradle文件的android{}节点下添加
android {
    compileOptions {
        sourceCompatibility = 1.8
        targetCompatibility = 1.8
    }
}

复制代码
```

在 Kotlin 中调用 Flutter module
---------------------------

支持, 我们已经为我们的 Android 项目添加了 Flutter 所必须的依赖, 接下来我们来看如何在项目中以 Kotlin 的方式在 Fragment 中调用 Flutter 模块, 在这里我们能做到让 Flutter 优化提升加载速度，实现秒开 Flutter 模块

原生 Kotlin 端代码
-------------

```
/**
 * flutter抽象的基类fragment,具体的业务类fragment可以继承
 **/
abstract class FlutterFragment(moduleName: String) : IBaseFragment() {

    private val flutterEngine: FlutterEngine?
    private lateinit var flutterView: FlutterView

    init {
        flutterEngine =FlutterCacheManager.instance!!.getCachedFlutterEngine(AppGlobals.get(), moduleName)
    }

    override fun getLayoutId(): Int {
        return R.layout.fragment_flutter
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        (mLayoutView as ViewGroup).addView(createFlutterView(activity!!))
    }

    private fun createFlutterView(context: Context): FlutterView {
        val flutterTextureView = FlutterTextureView(activity!!)
        flutterView = FlutterView(context, flutterTextureView)
        return flutterView
    }

    /**
     * 设置标题
     */
    fun setTitle(titleStr: String) {
        rl_title.visibility = View.VISIBLE
        title_line.visibility = View.VISIBLE
        title.text = titleStr
        title.setOnClickListener {

        }
    }

    /**
     * 生命周期告知flutter
     */
    override fun onStart() {
        flutterView.attachToFlutterEngine(flutterEngine!!)
        super.onStart()
    }

    override fun onResume() {
        super.onResume()
        //for flutter >= v1.17
        flutterEngine!!.lifecycleChannel.appIsResumed()
    }

    override fun onPause() {
        super.onPause()
        flutterEngine!!.lifecycleChannel.appIsInactive()
    }

    override fun onStop() {
        super.onStop()
        flutterEngine!!.lifecycleChannel.appIsPaused()
    }

    override fun onDetach() {
        super.onDetach()
        flutterEngine!!.lifecycleChannel.appIsDetached()
    }

    override fun onDestroy() {
        super.onDestroy()
        flutterView.detachFromFlutterEngine()
    }
}

复制代码
```

**R.layout.fragment_flutter 的布局**

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <RelativeLayout
        android:id="@+id/rl_title"
        android:visibility="gone"
        android:layout_width="match_parent"
        android:layout_height="@dimen/dp_45"
        android:background="@color/color_white"
        android:gravity="center_vertical"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/title"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_centerInParent="true"
            android:layout_gravity="center"
            android:gravity="center"
            android:textColor="@color/color_000"
            android:textSize="16sp" />
    </RelativeLayout>

    <View
        android:id="@+id/title_line"
        android:visibility="gone"
        android:layout_width="match_parent"
        android:layout_height="2px"
        android:background="@color/color_eee" />
</LinearLayout>

复制代码
```

```
/**
 * flutter缓存管理,主要是管理多个flutter引擎
 **/
class FlutterCacheManager private constructor() {

    /**
     * 伴生对象,保持单例
     */
    companion object {

        //喜欢页面,默认是flutter启动的主入口
        const val MODULE_NAME_FAVORITE = "main"
        //推荐页面
        const val MODULE_NAME_RECOMMEND = "recommend"

        @JvmStatic
        @get:Synchronized
        var instance: FlutterCacheManager? = null
            get() {
                if (field == null) {
                    field = FlutterCacheManager()
                }
                return field
            }
            private set
    }

    /**
     * 空闲时候预加载Flutter
     */
    fun preLoad(context: Context){
        //在线程空闲时执行预加载任务
        Looper.myQueue().addIdleHandler {
            initFlutterEngine(context, MODULE_NAME_FAVORITE)
            initFlutterEngine(context, MODULE_NAME_RECOMMEND)
            false
        }
    }

    /**
     * 初始化Flutter
     */
    private fun initFlutterEngine(context: Context, moduleName: String): FlutterEngine {
        //flutter 引擎
        val flutterLoader: FlutterLoader = FlutterInjector.instance().flutterLoader()
        val flutterEngine = FlutterEngine(context,flutterLoader, FlutterJNI())
        flutterEngine.dartExecutor.executeDartEntrypoint(
            DartExecutor.DartEntrypoint(
                flutterLoader.findAppBundlePath(),
                moduleName
            )
        )
        //存到引擎缓存中
        FlutterEngineCache.getInstance().put(moduleName,flutterEngine)
        return flutterEngine
    }

    /**
     * 获取缓存的flutterEngine
     */
    fun getCachedFlutterEngine(context: Context?, moduleName: String):FlutterEngine{
        var flutterEngine = FlutterEngineCache.getInstance()[moduleName]
        if(flutterEngine==null && context!=null){
            flutterEngine=initFlutterEngine(context,moduleName)
        }
        return flutterEngine!!
    }

}

复制代码
```

**具体业务类使用**

```
//在app初始化中初始一下
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        FlutterCacheManager.getInstance().preLoad(this);
    }
}

复制代码
```

**收藏页面**

```
class FavoriteFragment : FlutterFragment(FlutterCacheManager.MODULE_NAME_FAVORITE) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        setTitle(getString(R.string.title_favorite))
    }
}

复制代码
```

**推荐页面**

```
class RecommendFragment : FlutterFragment(FlutterCacheManager.MODULE_NAME_RECOMMEND) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        setTitle(getString(R.string.title_recommend))
    }
}

复制代码
```

Dart 端代码
--------

```
import 'package:flutter/material.dart';
import 'package:flutter_module/favorite_page.dart';
import 'package:flutter_module/recommend_page.dart';

//至少要有一个入口,而且这下面的man() 和 recommend()函数名字 要和FlutterCacheManager中定义的对应上
void main() => runApp(MyApp(FavoritePage()));

//必须加注解
@pragma('vm:entry-point')
void recommend() => runApp(MyApp(RecommendPage()));

class MyApp extends StatelessWidget {
  final Widget page;
  const MyApp(this.page);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        body: page,
      ),
    );
  }
}

复制代码
```

Dart 侧收藏页面

```
import 'package:flutter/material.dart';

class FavoritePage extends StatefulWidget {
  @override
  _FavoritePageState createState() => _FavoritePageState();
}

class _FavoritePageState extends State<FavoritePage> {
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Text("收藏"),
    );
  }
}

复制代码
```

Dart 侧推荐页面

```
import 'package:flutter/material.dart';

class RecommendPage extends StatefulWidget {
  @override
  _RecommendPageState createState() => _RecommendPageState();
}

class _RecommendPageState extends State<RecommendPage> {
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Text("推荐"),
    );
  }
}

复制代码
```

最终效果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9226abab0b294d3392b9bc3732c14070~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f20c112dbbd4910923acb937cdfd288~tplv-k3u1fbpfcp-watermark.awebp)

结尾
--

> 有一起学习的小伙伴可以关注下我的公众号——❤️**程序猿养成中心**❤️ 每周会定期做技术分享。快加入和我一起学习吧！