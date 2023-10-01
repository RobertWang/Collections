> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/392b80e8ea87)

_最近~闲来无事~搞了个 TGAM 蓝牙脑电波模块_  
_事实证明有钱还是去买游戏吧买什么传感器_  
_谁能帮我打塞尔达剑之试炼免费送脑波代码_

故事还要从多年前 NeuroSky 公司搞了个 Mindwave 脑电波蓝牙耳机说起。笔者曾有幸借到过这个蓝牙耳机感觉很高大上，长这样：

![](http://upload-images.jianshu.io/upload_images/3820578-98dbe33c3e0e4019.jpg) Mindwave.jpg

有了脑电波数据后，就可以…… 比如把自己的注意力集中度投影到 boss 的屏幕上，让 boss 知道你在认真工作，听起来还有些小激动呢~（谁会做这种事啊 (╯‵□′)╯︵┻━┻！）~。

Anyway，最近 ST 也想重新尝试一下脑电波，但是耳机套要一千多呢，作为一个穷人，ST 只淘宝了神念科技用的脑电波采集模块——ThinkgearAM，嗯…… 长这样：

![](http://upload-images.jianshu.io/upload_images/3820578-c1cd2d8f9ba716ac.jpg) TGAM.jpg

虽然看着很简陋但是功能是一样的！（并且省了一个任天堂健身环的钱呢。）

言归正传，开始开发。

### 一. 开发思路

TGAM 套件自带一个蓝牙从机，一般来讲，如果直接用 Thinkgear 官方的 api 开发，用电脑配上蓝牙，运行官方的 C 或者 Java 就行。  
但是如果你想用脑电波点亮个灯或驱动个车，还是直接连 Arduino 单片机比较方便。具体思路是在 Arduino 板上接一个蓝牙主机，接收传感器发送的数据，并在 Arduino 程序中使用它们。  
我们需要：  
Arduino Uno 开发板一块；  
HC-05 蓝牙模块一个（用作主蓝牙）；  
TGAM 套件一组，套件自带从蓝牙；  
导线若干；

### 二. AT 命令与主从蓝牙配对

###### 1. HC-05 与 Arduino 连接

HC-05 是主从一体的蓝牙模块，HC-05 与 Arduino 连接可以参照 Arduino 实验室里面的

> [Arduino 使用 HC05 蓝牙模块与手机连接](https://links.jianshu.com/go?to=https%3A%2F%2Farduino.nxez.com%2F2018%2F04%2F19%2Farduino-uses-hc05-bluetooth-module-to-connect-the-phone.html)

在上面教程上稍微做些更改，Arduino 板与 HC-05 芯片连接如下：

![](http://upload-images.jianshu.io/upload_images/3820578-9603525e8b9ebb97.jpg) 连线方式. jpg

```
Arduino 5V – VCC
Arduino GND – GND
Arduino Pin10 – TXD
Arduino Pin11 – RXD
Arduino 3.3V – KEY


```

这里 Pin10 和 Pin11 的引脚可换，但要和程序里`SoftwareSerial(rxPin, txPin)`对应。rxPin 是软串口接收引脚，txPin 是软串口发送引脚，保证单片机发送对芯片接收，单片机接收对芯片发送。  
EN/KEY 接 3.3V 是为了用 AT+INIT 初始化命令，这个命令只有 EN/KEY 引脚置高电平时才能使用。只有初始化了之后，才能用 AT+INQ 搜索其他蓝牙设备。

```
#include <SoftwareSerial.h> 
 
// Pin10为RX，接HC05的TXD
// Pin11为TX，接HC05的RXD
SoftwareSerial BT(10, 11); 
char val;
 
void setup() {
  Serial.begin(38400); 
  Serial.println("BT is ready!");
  // HC-05默认，38400
  BT.begin(38400);
}
 
void loop() {
  if (Serial.available()) {
    val = Serial.read();
    BT.print(val);
  }
 
  if (BT.available()) {
    val = BT.read();
    Serial.print(val);
  }
}


```

程序中的`Serial.begin`波特率我们不用动，**它跟 HC-05 与其他蓝牙模块配对传输时的波特率是两个概念**，此处的波特率只是 Arduino 给 HC-05 写指令用的波特率，不影响配对。

HC-05 有两种工作模式，普通模式和 AT 模式。准备完程序并且插好线后，按住 HC-05 模块角落的黑色小按钮，同时给 Arduino 开发板通电，这时会看到蓝牙模块的 LED 灯大概 2 秒闪烁一下，证明蓝牙模块已经进入 AT 模式，可以编写指令了 ♪（＾∀＾●）。

###### 2. 写入 AT 指令

在写 AT 指令前，我们要知道，两个蓝牙模块成功配对有几个条件：  
**配对密码要一样；  
两个蓝牙模块的串口波特率要一样；  
主模块找到从模块地址；**

根据 Thinkgear 给的官方 api，TGAM 的默认密码是 0000，波特率是 57600；我们需要将 HC-05 的密码和波特率也设置成这个才能配对。

![](http://upload-images.jianshu.io/upload_images/3820578-c4a299e9bac35d46.png) Thinkgear 参数. png

打开 Arduino 的串口监视器，选 NL 和 CR，调到 38400 的波特率，会看到显示了`BT is ready!`  
此时发送 AT，会回复 OK，可以配置 HC-05 蓝牙芯片了。_这里我第一次发送 AT 的时候程序总会返回 ERROR，再发一遍才显示 OK_

![](http://upload-images.jianshu.io/upload_images/3820578-de69aaf4ec4e65a4.png) 串口监视器. png

AT 指令百度一下有很多也很全，这里只写重要的几个。  
**AT+INIT**：初始化。初始化了之后才能 **AT+INQ** 查询其他蓝牙设备，当然如果你已经知道了从机蓝牙的 MAC 地址，可以不查询。  
**AT+ROLE**：选择 HC05 蓝牙模块的角色，AT+ROLE=0 为从机，AT+ROLE=1 为主机，默认是从机，这里我们需要输入`AT+ROLE=1`改成主机。用`AT+ROLE?`可以查看当前角色。  
**AT+PSWD**：设置蓝牙模块的配对密码，HC-05 的默认配对密码是 1234，TGAM 从机密码是 0000，我们要输入`AT+PSWD=0000`改成一样的。  
**AT_UART**：设置蓝牙与其他蓝牙通讯的波特率，指令格式是`AT+UART=<Param>,<Param1>,<Param2>`，第一个参数是波特率，后面两个参数是停止位和校验位（一般置 0），HC-05 默认波特率是 9600，TGAM 从机默认波特率是 57600，我们输入`AT+UART=57600,0,0`  
**AT+INQ**：查询蓝牙设备，返回设备的 mac 地址。  
**AT+BIND**：绑定其他蓝牙 MAC 地址，`AT+BIND=<address>`，这里，如果你的从蓝牙 MAC 地址是`A44A:0E:08008D`，输入时需要把冒号改为逗号，输入`AT+BIND=A44A,0E,08008D`  
**AT+CMODE**：是否允许蓝牙连接任何设备，0 表示只能连接绑定地址的蓝牙，1 表示可以连接任何蓝牙。  
**AT+LINK**：这是最后一步，连接蓝牙设备，输入`AT+LINK=A44A,0E,08008D`，返回 OK 为成功，返回 FAIL 为失败。失败的话，请确认两个蓝牙的配对密码和波特率是否一致。

###### 3. 主从蓝牙配对

如果两个蓝牙成功连接上，HC-05 的 LED 灯会每 2 秒很快的闪 2 下，而 TGAM 蓝牙的指示灯会一直亮着，此时就说明连接成功了。  
接下来，断开 Arduino 板，去掉 HC-05 KEY 引脚的导线，再次插上时会回到 HC-05 的工作模式，同时两个蓝牙模块还是会自动匹配。

_友情提示，当两个蓝牙成功连接上时，串口监视器可能会突然蹦出好多乱码，影响判断连接状态，可以直接通过观察两个蓝牙芯片指示灯状态判断是否连接成功。_

![](http://upload-images.jianshu.io/upload_images/3820578-1c39cf77a693bfb9.jpg) 连接成功. jpg

### 三. Arduino 读取数据

连接上了之后，我们就可以在 Arduino 中编程读取脑电波传感器的数据了。  
此时需要改变一下连线：  
**HC-05 的 TXD 接 Uno 板的 0->RX 引脚，RXD 接 TX->1 引脚**

```
Arduino 5V – VCC
Arduino GND – GND
Arduino Pin0 – TXD
Arduino Pin1 – RXD


```

接完后重新插上 Arduino Uno 板，并烧录 Arduino 读取脑电波数值的程序。

_这里遇到个问题，TXD 和 RXD 接了 Pin0 和 Pin1 时，程序一直上传不上去（⊙.⊙），只好先在接 10 和 11 引脚的时候把程序上传上去，然后改导线到 Pin0 和 Pin1，这样串口监视器才会显示数据，不知为何，还请大神赐教。_

总之，磕磕绊绊的总算是调通了 (ง •̀灬 •́)ง。这样就可以拿数据为所欲为了。

![](http://upload-images.jianshu.io/upload_images/3820578-1c5c2467fdd2077a.jpg) 成功. jpg

示例代码如下：

```
 /*
 通过UART串口显示信号值、注意力及放松度的值
 */
#define BAUDRATE 57600
#define DEBUGOUTPUT 0

//校验相关变量
int   generatedChecksum = 0;
byte  checksum = 0; 

//接收数据长度和数据数组
byte  payloadLength = 0;
byte  payloadData[32] = {0};//总共接收32个自己的数据

//需要读取的信息变量
byte signalquality = 0;//信号质量
byte attention = 0;    //注意力值
byte meditation = 0;   //放松度值

//初始化
void setup() 
{
  Serial.begin(BAUDRATE); 
}

//从串口读取一个字节数据
byte ReadOneByte() 
{
  int ByteRead;
  while(!Serial.available());
  ByteRead = Serial.read();
  return ByteRead;//返回读到的字节
}

//读取串口数据
void read_serial_data()
{
    //寻找数据包起始同步字节，2个
    if(ReadOneByte() == 0xAA)//先读一个
    {
      if(ReadOneByte() == 0xAA)//读第二个
      {
        payloadLength = ReadOneByte();//读取第三个，数据包字节的长度
        if(payloadLength == 0x20)//如果接收到的是大包数据才继续读取，小包数据则舍弃不读取
        {
          generatedChecksum = 0; //校验变量清0       
          for(int i = 0; i < payloadLength; i++)//连续读取32个字节
          {  
            payloadData[i] = ReadOneByte();//读取指定长度数据包中的数据
            generatedChecksum += payloadData[i];//计算数据累加和
          }         
          checksum = ReadOneByte();//读取校验字节  
          //校验
          generatedChecksum = (~generatedChecksum)&0xff;       
          //比较校验字节
          if(checksum == generatedChecksum)//数据接收正确，继续处理 
          {    
            signalquality = 0;//信号质量变量
            attention = 0;    //注意力值变量
            //赋值数据
            signalquality = payloadData[1];//信号值      
            attention = payloadData[29];//注意力值
            meditation = payloadData[31];//放松度值
            #if !DEBUGOUTPUT         
            //打印信号质量
            Serial.print("SignalQuality: ");
            Serial.print(signalquality, DEC);
            //打印注意力值
            Serial.print("Attation: ");
            Serial.print(attention, DEC);
            //打印放松度值
            Serial.print("Meditation: ");
            Serial.print(meditation, DEC);
            //换行
            Serial.print("\n");       
            #endif              
          } 
        } 
      }
    }
}

//主循环
void loop() 
{
  read_serial_data();//读取串口数据 
}


```

折腾了三天，总算是配对了蓝牙也接收了数据，心满意足！然后……  
Arduino 关闭！  
Idea 启动！  
直接用 Java 读数不香吗？什么单片机导线太费脑细胞了吧乛 з乛

感谢 YCK 大神教我 Thinkgear 的 Java api 怎么使用。  
感谢 Doggy 看串口看了个两秒左右。  
感谢面包匠同志在我身边看沙雕视频保证了我愉悦的好心情。  
感谢 Berber 啃了两口传感器芯片，让它突然就配对成功了。

![](http://upload-images.jianshu.io/upload_images/3820578-09c4393e7fc274cd.jpg) 最大功臣. jpg