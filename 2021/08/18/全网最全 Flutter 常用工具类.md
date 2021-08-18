> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903873262387207)

> 1、SpUtil : 单例 "同步"SharedPreferences 工具类。

Flutter 工具类库 [flustars](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FSky24n%2Fflustars "https://github.com/Sky24n/flustars")
-----------------------------------------------------------------------------------------------------------------------------------------

1、SpUtil : 单例 "同步"SharedPreferences 工具类。  
2、ScreenUtil : 屏幕工具类.  
3、WidgetUtil : Widget 具类.  
4、DirectoryUtil : 文件目录工具类。  
5、DioUtil : 单例 Dio 网络工具类。

Dart 常用工具类库 [common_utils](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FSky24n%2Fcommon_utils "https://github.com/Sky24n/common_utils")

1、TimelineUtil : 时间轴.  
2、TimerUtil : 倒计时，定时任务.  
3、MoneyUtil : 精确转换，元转分，分转元，支持格式输出.  
4、LogUtil : 简单封装打印日志.  
5、DateUtil : 日期转换格式化输出.  
6、RegexUtil : 正则验证手机号，身份证，邮箱等等.  
7、NumUtil : 保留 x 位小数, 精确加、减、乘、除, 防止精度丢失.  
8、ObjectUtil : 判断对象是否为空 (String List Map), 判断两个 List 是否相等.  
9、EnDecodeUtil : md5 加密，Base64 加 / 解密.  
10、TextUtil : 银行卡号每隔 4 位加空格，每隔 3 三位加逗号，隐藏手机号等等.

### SpUtil

单例 "同步"SharedPreferences 工具类。支持 get 传入默认值，支持存储对象，支持存储对象数组。由于 SharedPreferences 需要异步生成才能使用。  
方式一：简单粗暴，等待 Sp 生成后再运行 app。[示例项目一](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FSky24n%2FGreenTravel "https://github.com/Sky24n/GreenTravel")  
方式二：增加 splash 页面，在 splash 页面完初始化，进入主页就可以同步使用。[示例项目二](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FSky24n%2Fflutter_wanandroid "https://github.com/Sky24n/flutter_wanandroid")

```
/// 方式一
/// 等待sp初始化完成后再运行app。
/// sp初始化时间 release模式下30ms左右，debug模式下100多ms。
void main() async {
  await SpUtil.getInstance();
  runApp(MyApp());
}

class MyAppState extends State<MyApp> {
  @override
  void initState() {
    super.initState();
    /// 同步使用Sp。
    SpUtil.remove("username");
    String defName = SpUtil.getString("username", defValue: "sky");
    SpUtil.putString("username", "sky24");
    String name = SpUtil.getString("username");
    print("MyApp defName: $defName, name: $name");
  }
  @override
  Widget build(BuildContext context) {
    return MaterialApp();
  }
} 

// App启动时读取Sp数据，需要异步等待Sp初始化完成。
await SpUtil.getInstance();

// 常规存取示例。
//支持默认值。String，bool，int，double，StringList。
SpUtil.putString("username", "sky24");
String defName = SpUtil.getString("username");
String defName = SpUtil.getString("username", defValue: "sky");

// 存取实体对象示例。
City city = new City();
city.name = "成都市";
// 存储实体对象。
SpUtil.putObject("loc_city", city);
// 读取实体对象。
City hisCity = SpUtil.getObj("loc_city", (v) => City.fromJson(v)); 

// 存取实体对象list示例。
List<City> list = new List();
list.add(new City(name: "成都市"));
list.add(new City(name: "北京市"));
// 存储实体对象list。
SpUtil.putObjectList("loc_city_list", list);
// 读取实体对象list。
List<City> dataList = SpUtil.getObjList("loc_city_list", (v) => City.fromJson(v)); 
复制代码
```

### ScreenUtil

屏幕适配，兼容横 / 纵屏切换适配，获取屏幕宽、高、密度，AppBar 高，状态栏高度，屏幕方向.

```
一、不依赖context
// 屏幕宽
double screenWidth = ScreenUtil.getInstance().screenWidth;
// 屏幕高
double screenHeight = ScreenUtil.getInstance().screenHeight;
// 屏幕像素密度
double screenDensity = ScreenUtil.getInstance().screenDensity;
// 系统状态栏高度
double statusBarHeight = ScreenUtil.getInstance().statusBarHeight;
// BottomBar高度 
double bottomBarHeight = ScreenUtil.getInstance().bottomBarHeight;
// 系统AppBar高度
double appBarHeight = ScreenUtil.getInstance().appBarHeight;
// 获取适配后的尺寸
double adapterSize = ScreenUtil.getInstance().getAdapterSize(100);

二、依赖context
// 屏幕宽
double screenWidth = ScreenUtil.getScreenW(context);
// 屏幕高
double screenHeight = ScreenUtil.getScreenH(context);
// 屏幕像素密度
double screenDensity = ScreenUtil.getScreenDensity(context);
// 系统状态栏高度
double statusBarHeight = ScreenUtil.getStatusBarH(context);
// BottomBar高度
double bottomBarHeight = ScreenUtil.getBottomBarH(context);
// 屏幕方向
Orientation orientation = ScreenUtil.getOrientation(context);
// 获取适配后的尺寸
double adapterSize = ScreenUtil.getAdapterSizeCtx(context, 100);
复制代码
```

### WidgetUtil

监听 Widget 渲染状态，获取 Widget 宽高，在屏幕上的坐标，获取网络 / 本地图片尺寸.

```
asyncPrepare              : Widget渲染监听，监听widget宽高变化,callback返回宽高等参数.
getWidgetBounds           : 获取widget 宽高.
getWidgetLocalToGlobal    : 获取widget在屏幕上的坐标.
getImageWH                : 获取图片宽高，加载错误情况返回 Rect.zero.（单位 px）. 
getImageWHE               : 获取图片宽高，加载错误会抛出异常.（单位 px）. 

/// widget渲染监听。
WidgetUtil widgetUtil = new WidgetUtil();
widgetUtil.asyncPrepare(context, true, (Rect rect) {
  // widget渲染完成。
});

/// widget宽高。
Rect rect = WidgetUtil.getWidgetBounds(context);

/// widget在屏幕上的坐标。
Offset offset = WidgetUtil.getWidgetLocalToGlobal(context);
  
/// 获取CachedNetworkImage下的图片尺寸
Image image = new Image(image: new CachedNetworkImageProvider("Url"));
Rect rect1 = await WidgetUtil.getImageWH(image: image);  

/// 其他image
Image imageAsset = new Image.asset("");
Image imageFile = new Image.file(File("path"));
Image imageNetwork = new Image.network("url");
Image imageMemory = new Image.memory(null);

/// 获取网络图片尺寸
Rect rect2 = await WidgetUtil.getImageWH(url: "Url");

/// 获取本地图片尺寸 localUrl 需要全路径
Rect rect3 = await WidgetUtil.getImageWH(localUrl: "assets/images/3.0x/ali_connors.png");

/// 其他方式
WidgetUtil.getImageWH(url: "Url").then((Rect rect) {
  print("rect: " + rect.toString();
});

WidgetUtil.getImageWHE(url: "Url").then((Rect rect) {
  print("rect: " + rect.toString();
}).catchError((error) {
  print("rect: " + error.toString();
});
复制代码
```

### DirectoryUtil

文件目录工具类.

```
await DirectoryUtil.getInstance();
String path = DirectoryUtil.getTempPath(fileName: 'demo.png', category: 'image');
String path = DirectoryUtil.getAppDocPath(fileName: 'demo.mp4', category: 'video');
String path = DirectoryUtil.getStoragePath(fileName: 'flutterwanandroid.apk', package: 'com.thl.flutterwanandroid');

Directory dir = DirectoryUtil.createTempDirSync(package: 'doc', category: 'image');
复制代码
```

### DioUtil

单例 DioUtil，已迁移至此处 [DioUtil](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FSky24n%2FFlutterRepos%2Fblob%2Fmaster%2Fbase_library%2Flib%2Fsrc%2Fdata%2Fnet%2Fdio_util.dart "https://github.com/Sky24n/FlutterRepos/blob/master/base_library/lib/src/data/net/dio_util.dart")。(基于 v1.0.13，仅供参考～)

```
// 打开debug模式.
DioUtil.openDebug();   
  
// 配置网络参数.
Options options = DioUtil.getDefOptions();
options.baseUrl = "http://www.wanandroid.com/";
HttpConfig config = new HttpConfig(options: options);
DioUtil().setConfig(config);
  
// 两种单例请求方式.
DioUtil().request<List>(Method.get, "banner/json");
DioUtil.getInstance().request(Method.get, "banner/json");
  
//示例
LoginReq req = new LoginReq('username', 'password');
DioUtil().request(Method.post, "user/login",data: req.toJson());
  
//示例
FormData formData = new FormData.from({
      "username": "username",
      "password": "password",
    });
DioUtil().requestR(Method.post, "user/login",data: rformData);
复制代码
```

### TextUtil

```
isEmpty                     : isEmpty.(新)
formatSpace4                : 每隔4位加空格，格式化银行卡.(新)
formatComma3                : 每隔3三位加逗号.(新)
hidePhone                   : 隐藏手机号.(新)
replace                     : replace.(新)
split                       : split.(新)
  
/// example
String phoneNo = TextUtil.formatSpace4("15845678910"); // 1584 5678 910
String num     = TextUtil.formatComma3("1234"); // 123,4
String phoneNo = TextUtil.hidePhone("15845678910")// 158****8910
复制代码
```

### EnDecodeUtil

```
encodeMd5                   : md5 加密.(新)
encodeBase64                : Base64加密.(新)
decodeBase64()              : Base64解密.(新)
复制代码
```

### TimelineUtil

时间轴。例如：微信朋友圈，微博时间轴，支付宝时间轴。

```
///(xx)为可配置输出
enum DayFormat {
  ///(小于10s->刚刚)、x分钟、x小时、(昨天)、x天.
  Simple,
  ///(小于10s->刚刚)、x分钟、x小时、[今年: (昨天/1天前)、(2天前)、MM-dd],[往年: yyyy-MM-dd].
  Common,
  ///小于10s->刚刚)、x分钟、x小时、[今年: (昨天 HH:mm/1天前)、(2天前)、MM-dd HH:mm],[往年: yyyy-MM-dd HH:mm].
  Full,
}
///Timeline信息配置.
abstract class TimelineInfo {
  String suffixAgo(); //suffix ago(后缀 后).
  String suffixAfter(); //suffix after(后缀 前).
  String lessThanTenSecond() => ''; //just now(刚刚).
  String customYesterday() => ''; //Yesterday(昨天).优先级高于keepOneDay
  bool keepOneDay(); //保持1天,example: true -> 1天前, false -> MM-dd.
  bool keepTwoDays(); //保持2天,example: true -> 2天前, false -> MM-dd.
  String oneMinute(int minutes); //a minute(1分钟).
  String minutes(int minutes); //x minutes(x分钟).
  String anHour(int hours); //an hour(1小时).
  String hours(int hours); //x hours(x小时).
  String oneDay(int days); //a day(1天).
  String days(int days); //x days(x天).
  DayFormat dayFormat(); //format.
}
setLocaleInfo               : 自定义设置配置信息.
formatByDateTime            : 格式输出时间轴信息 by DateTime .
format                      : 格式输出时间轴信息.
复制代码
```

### TimerUtil

倒计时，定时任务。

```
setInterval                 : 设置Timer间隔.
setTotalTime                : 设置倒计时总时间.
startTimer()                : 启动定时Timer.
startCountDown              : 启动倒计时Timer.
updateTotalTime             : 重设倒计时总时间.
cancel                      : 取消计时器.
setOnTimerTickCallback      : 计时器回调.
isActive                    : Timer是否启动.

  TimerUtil timerUtil;
  //定时任务test
  timerUtil = new TimerUtil(mInterval: 1000);
  //timerUtil.setInterval(1000);
  timerUtil.setOnTimerTickCallback((int value) {
    LogUtil.e("TimerTick: " + value.toString());
  });
  timerUtil.startTimer();
  if (timerUtil != null) timerUtil.cancel(); //dispose()

  TimerUtil timerCountDown;
  //倒计时test
  timerCountDown = new TimerUtil(mInterval: 1000, mTotalTime: 3 * 1000);
//    timerCountDown.setInterval(1000);
//    timerCountDown.setTotalTime(3 * 1000);
  timerCountDown.setOnTimerTickCallback((int value) {
    double tick = (value / 1000);
    LogUtil.e("CountDown: " + tick.toInt().toString());
  });
  timerCountDown.startCountDown();
  if (timerCountDown != null) timerCountDown.cancel(); //dispose()

复制代码
```

### MoneyUtil

金额工具类，分 / 元互转，精确转换, 防止精度丢失。

```
changeF2Y                   : 分 转 元, format格式输出.
changeFStr2YWithUnit        : 分字符串 转 元, format 与 unit 格式 输出.
changeF2YWithUnit           : 分 转 元, format 与 unit 格式 输出.
changeYWithUnit             : 元, format 与 unit 格式 输出.
changeY2F                   : 元 转 分.
复制代码
```

### LogUtil

简单封装打印日志，可完整输出超长 log。

```
init(isDebug, tag)          : isDebug: 模式, tag 标签.
e(object, tag)              : 日志e
v(object, tag)              : 日志v，只在debug模式输出.
复制代码
```

### DateUtil

日期工具类，支付自定义格式输出。

```
enum DateFormat {
  DEFAULT, //yyyy-MM-dd HH:mm:ss.SSS
  NORMAL, //yyyy-MM-dd HH:mm:ss
  YEAR_MONTH_DAY_HOUR_MINUTE, //yyyy-MM-dd HH:mm
  YEAR_MONTH_DAY, //yyyy-MM-dd
  YEAR_MONTH, //yyyy-MM
  MONTH_DAY, //MM-dd
  MONTH_DAY_HOUR_MINUTE, //MM-dd HH:mm
  HOUR_MINUTE_SECOND, //HH:mm:ss
  HOUR_MINUTE, //HH:mm

  ZH_DEFAULT, //yyyy年MM月dd日 HH时mm分ss秒SSS毫秒
  ZH_NORMAL, //yyyy年MM月dd日 HH时mm分ss秒  /  timeSeparate: ":" --> yyyy年MM月dd日 HH:mm:ss
  ZH_YEAR_MONTH_DAY_HOUR_MINUTE, //yyyy年MM月dd日 HH时mm分  /  timeSeparate: ":" --> yyyy年MM月dd日 HH:mm
  ZH_YEAR_MONTH_DAY, //yyyy年MM月dd日
  ZH_YEAR_MONTH, //yyyy年MM月
  ZH_MONTH_DAY, //MM月dd日
  ZH_MONTH_DAY_HOUR_MINUTE, //MM月
  dd日 HH时mm分  /  timeSeparate: ":" --> MM月dd日 HH:mm
  ZH_HOUR_MINUTE_SECOND, //HH时mm分ss秒
  ZH_HOUR_MINUTE, //HH时mm分
}
formatDate                      : 格式化日期 DateTime.(新)
formatDateStr                   : 格式化日期 字符串.(新)
formatDateMs                    : 格式化日期 毫秒.(新)
getNowDateMs                    : 获取现在 毫秒.
getNowDateStr                   : 获取现在 日期字符串.(yyyy-MM-dd HH:mm:ss)
getDateMsByTimeStr              : 获取毫秒 By 日期字符串(Format格式输出).
getDateStrByTimeStr             : 获取日期字符串 By 日期字符串(Format格式输出).
getDateStrByMs                  : 获取日期字符串 By 毫秒(Format格式输出).
getDateStrByDateTime            : 获取日期字符串 By DateTime(Format格式输出).
getWeekDay                      : 获取WeekDay By DateTime.
getZHWeekDay                    : 获取星期 By DateTime.
getWeekDayByMs                  : 获取WeekDay By 毫秒.
getZHWeekDayByMs                : 获取星期 By 毫秒.
isLeapYearByYear                : 是否是闰年.
yearIsEqual                     : 是否同年.
getDayOfYear                    : 在今年的第几天.
isYesterday                     : 是否是昨天.
isToday                         : 是否是今天.
isWeek                          : 是否是本周.(新)

/// DateUtil
DateUtil.formatDateMs(DateTime.now().millisecondsSinceEpoch, format: DataFormats.full); // 2019-07-09 16:51:14
DateUtil.formatDateStr("2019-07-09 16:51:14", format: "yyyy/M/d HH:mm:ss"); // 2019/7/9 16:51:14
DateUtil.formatDate(DateTime.now(), format: "yyyy/MM/dd HH:mm:ss");  // 2019/07/09 16:51:14
复制代码
```

### RegexUtil

正则工具类。

```
isMobileSimple            : 简单验证手机号
isMobileExact             : 精确验证手机号
isTel                     : 验证电话号码
isIDCard                  : 验证身份证号码
isIDCard15                : 验证身份证号码 15 位
isIDCard18                : 简单验证身份证号码 18 位
isIDCard18Exact           : 精确验证身份证号码 18 位
isEmail                   : 验证邮箱
isURL                     : 验证 URL
isZh                      : 验证汉字
isDate                    : 验证 yyyy-MM-dd 格式的日期校验，已考虑平闰年
isIP                      : 验证 IP 地址
复制代码
```

### NumUtil

数值运算工具类，保留 x 位小数, 精确加、减、乘、除, 防止精度丢失.

```
getIntByValueStr            : 数字字符串转int.
getDoubleByValueStr         : 数字字符串转double.
getNumByValueStr            : 保留x位小数 by 数字字符串.
getNumByValueDouble         : 保留x位小数 by double.
add                         : 加(精确相加,防止精度丢失).
subtract                    : 减(精确相减,防止精度丢失).
multiply                    : 乘(精确相乘,防止精度丢失).
divide                      : 除(精确相除,防止精度丢失).
复制代码
```

### ObjectUtil

判断对象是否为空 (String List Map), 判断两个 List 是否相等.

```
isEmptyString             : 判断String是否为空.
isEmptyList               : 判断List是否为空.
isEmptyMap                : 判断Map是否为空.
isEmpty                   : 判断对象是否为空.(String List Map).
isNotEmpty                : 判断对象是否非空.(String List Map).
twoListIsEqual            : 判断两个List是否相等.
复制代码
```

关于作者
----

GitHub : [Sky24n](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FSky24n "https://github.com/Sky24n")  
简书     : [Sky24n](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fu%2Fcbf2ad25d33a "https://www.jianshu.com/u/cbf2ad25d33a")  
掘金     : [Sky24n](https://juejin.cn/user/3139860939675453 "https://juejin.cn/user/3139860939675453")  
Pub      : [Sky24n](https://link.juejin.cn/?target=https%3A%2F%2Fpub.flutter-io.cn%2Fpackages%3Fq%3Demail%253A863764940%2540qq.com "https://pub.flutter-io.cn/packages?q=email%3A863764940%40qq.com")

项目示例
----

GitHub : [flutter_demos](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FSky24n%2Fflutter_wanandroid%2Ftree%2Fmaster%2Flib%2Fdemos "https://github.com/Sky24n/flutter_wanandroid/tree/master/lib/demos")  
APK      : [点击下载 v0.2.1](https://link.juejin.cn/?target=https%3A%2F%2Fraw.githubusercontent.com%2FSky24n%2FLDocuments%2Fmaster%2FAppStore%2Fflutter_wanandroid_juejin.apk "https://raw.githubusercontent.com/Sky24n/LDocuments/master/AppStore/flutter_wanandroid_juejin.apk")  
Android 扫码下载 APK:

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/10/8/16da76070eb424cc~tplv-t2oaga2asx-watermark.awebp)

Screenshot
----------

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/23/16b841641f346bfc~tplv-t2oaga2asx-watermark.awebp) ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/6/23/16b8416288663e01~tplv-t2oaga2asx-watermark.awebp)