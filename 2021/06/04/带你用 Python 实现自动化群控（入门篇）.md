> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMzMzOTI3Nw==&mid=2247500957&idx=1&sn=759cef095d931400b66da7a8e8d32fa8&chksm=e885a47fdff22d69e36058fe28d4ab862d03c72452647bb431f4f98b11e257bdcb7f57b2570d&mpshare=1&scene=1&srcid=0604xS6o2W5f50WXg7J6L69C&sharer_sharetime=1622773396454&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

点击上方 “Python 编程时光”，选择 “加为星标”  

第一时间关注 Python 技术干货！

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/QB6G4ZoE184aEjscBD5N7h0W7rw7amwfKOdEmYOPWoXiaBegNialS627LHfgic47nFEBIHbo6IhEprJCQqZ2u1ZiaQ/640?wx_fmt=jpeg)

1. 前言
-----

群控，相信大部分人都不会陌生！印象里是一台电脑控制多台设备完成一系列的操作，更多的人喜欢把它和 Hui 产绑定在一起！

事实上，群控在自动化测试中也被广泛使用！接下来的几篇文章，我将带大家聊聊企业级自动化中，群控正确的使用姿势！

本篇先从基础篇开始，聊聊使用「 Python + adb 」命令如何编写一套群控脚本

2. 准备
-----

在本机安装 Android 开发环境，保证 adb 被添加到环境变量

将准备好的多台设备，使用数据线（ 或者通过 Hub ）连接到电脑上

通过 adb devices 命令查看已经连接的所有设备

```
# 下面显示连接了3台设备
xag:Test xingag$ adb devices
List of devices attached
822QEDTL225T7    device
ca2b3455        device
DE45d9323SE96   device


```

3. 实战
-----

自动化群控以闲鱼 App 的一次关键字搜索为例，步骤包含：打开应用、点击到搜索界面、输入内容、点击搜索按钮

下面通过 7 步来完成这一操作

1、获取目标应用的包名及初始化 Activity

获取方式有很多种，主流方式包含：adb 命令、解析 APK、第三方 APK、无障碍服务

这里推荐使用 adb 命令这种方式

```
# 获取当前运行应用的包名及初始Activity
adb shell dumpsys activity | grep -i run

```

打开闲鱼 App，在命令终端输入上面的命令，终端会将包名及 Activity 名称显示出来  

![](https://mmbiz.qpic.cn/mmbiz_png/atOH362BoytbLjMFyrOye0dC5209Oia3ZkjQzzGUicQibWbqy47r0O5rm13P89ZdibOthCRgicCa8vbKS7cVCrVLAiaA/640?wx_fmt=png)

2、获取所有在线的设备

通过 adb devices 命令，通过输出内容，进行一次过滤，得到所有连接到 PC 端的设备

```
# 所有设备ID
devices = []

def get_online_devices(self):
    """
    获取所有在线的设备
    :return:
    """
    global devices
    try:
        for device_serias_name in exec_cmd("adb devices"):
           # 过滤掉第一条数据及不在线的设备
           if "device" in device_serias_name:
              devices.append(device_serias_name.split("\t")[0])
           devices = devices[1:]
    except Exception as e:
            print(e)

    # 连上的所有设备及数量
    return devices


```

3、群控打开目标应用  

遍历设备列表，使用 adb -s 设备 ID shell am start -W 命令分别打开目标应用  

```
def start_app(self):
    """
    打开App
    :return: 
    """
    for device in devices:
        os.popen("adb -s " + device + " shell am start -W {}/{}".format(self.packageName, self.home_activity))
    print('等待加载完成...')
    sleep(10)


```

4、封装执行步骤

为了方便管理设备，将每一步的操作写入到 YAML 文件中，可以通过 ID 查找元素并执行点击操作、在输入框中输入内容、调用本地方法及输入参数

这里分别对应：保存 UI 树控件、查找输入框元素并执行点击操作、保存 UI 树控件（界面变化了）、输入文本内容、查看搜索按钮元素并执行点击操作

```
# steps_adb.yaml

# 包名和Activity
package_name:  com.taobao.idlefish
home_activity:  com.taobao.fleamarket.home.activity.InitActivity

# 执行步骤
steps:
  - save_ui_tree_to_local:
      method:  save_ui_tree_to_local
      args:
  - find_element_and_click:
      id:  com.taobao.idlefish:id/tx_id
  - save_ui_tree_to_local:
      method:  save_ui_tree_to_local
  - input_content:
      content:  Python
  - find_element_and_click:
      id:  com.taobao.idlefish:id/search_button


```

需要指出的是，为了提高群控的适配性，控件的实际坐标需要通过下面的步骤去获取：

*   导出界面的控件树
    
*   解析控件树 XML 文件，利用正则表达式得到目标控件的坐标值
    
*   计算出控件的中心点坐标
    

![](https://mmbiz.qpic.cn/mmbiz_png/atOH362BoysLWiaB47fg9Qq4ftV9ZbGNwVgdhXicGLyL9EroyV96Y9UibFvUqibfA2ia8emAuh9RJiaQG3XaGiczyiaB2w/640?wx_fmt=png)

利用控件 ID 获取元素中心点坐标的实现代码如下：

```
def get_element_position(element_id, uidump_name):
    """
    通过元素的id，使用ElementTree，解析元素控件树，查找元素的坐标中心点
    :param element_id: 元素id，比如：
    :return: 元素坐标
    """

    # 解析XML
    tree = ET.parse('./../%s.xml' % uidump_name)
    root = tree.getroot()

    # 待查找的元素
    result_element = None

    # print('查找数目', len(root.findall('.//node')))

    # 遍历查找node元素
    # 通过元素id
    for node_element in root.findall('.//node'):
        if node_element.attrib['resource-id'] == element_id:
            result_element = node_element
            break

    # 如果找不到元素，直接返回空
    if result_element is None:
        print('抱歉！找不到元素！')
        return None

    # 解析数据
    coord = re.compile(r"\d+").findall(result_element.attrib['bounds'])

    # 中心点坐标
    position_center = int((int(coord[0]) + int(coord[2])) / 2), int((int(coord[1]) + int(coord[3])) / 2)

    return position_center


```

5、区分设备

为了保证群控脚本执行不会产生干扰，在每个步骤执行之前，都应该将设备 ID 作为参数进行区分

比如：将控件的界面控件树按照设备保存为不同的名称、点击界面和输入的命令传相应设备 ID 作为入参

```
def save_ui_tree_to_local(dName):
    """
    获取当前Activity控件树，保存到本地
    文件名固定为：uidump.xml
    :param dName: 设备id
    :return:
    """

    exec_cmd("adb  -s %s shell uiautomator dump /data/local/tmp/%s.xml" % (dName, dName))

    sleep(2)

    exec_cmd("adb -s %s pull /data/local/tmp/%s.xml ./../" % (dName, dName))


```

6、执行步骤  

从 YAML 文件中读取执行步骤，遍历步骤集合，内部遍历设备列表，以保证每一个步骤，分别执行到每台设备上

```
# 执行步骤
for step in self.steps:
    # 设备
    for device in devices: 
        pass


```

接着，通过步骤名称匹配不同的操作，即可操作设备了  

```
# 操作名称
step_name = list(step)[0]

if step_name == 'save_ui_tree_to_local':
    # 保存UI数到本地
    method = step.get(step_name).get('method')
    save_ui_tree_to_local(device)
elif step_name == 'find_element_and_click':
    element_id = step.get(step_name).get('id')
    # 获取元素的坐标
    bound_search_input = get_element_position(element_id, device)
    # 点击元素
    exec_cmd('adb -s %s shell input tap %s %s' % (device, bound_search_input[0], bound_search_input[1]))
elif step_name == 'input_content':
    input_content = step.get(step_name).get('content')
    # 模拟输入
    exec_cmd('adb -s %s shell input text %s' % (device, input_content))
else:
    print('其他操作步骤')

```

7、关闭应用

当所有的操作完成之后，同样是遍历设备，利用 adb 命令去关闭 App 即可

```
def stop_all(self):
   """
   关闭应用
   :return:
   """
   for device in devices:
       os.popen("adb -s " + device + " shell am force-stop  %s" % self.packageName)

```

4. 最后
-----

本篇仅仅是 Python 自动化群控最简单的实现方式，后面将和大家讨论更加复杂的实现方式。

如果你觉得文章还不错，请大家 **点赞、分享、留言**下，因为这将是我持续输出更多优质文章的最强动力！

```
推荐阅读  点击标题可跳转

介绍一个 Python 发包收包利器 - scapy


终结 Python 原生字典？这个库要逆天改命了


Birdseye：极其强大的Python调试工具！


Git 各指令的本质，真是通俗易懂啊！


瞧瞧，这样的代码才叫 Pythonic （二）

如果对你有帮助。

请不吝点赞，点在看，谢谢

```