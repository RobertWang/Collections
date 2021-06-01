> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1451607&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)729 

### 这是写着玩的，能有多好

##### 灯带 30 个而已，然后一米长，这是弄着玩的 会的人不一定要用 RGB 灯，这是之前弄毕业设计一直找不到的资料，这两天闲着无聊不小心看到的 sounddevice 库，可以获取实时的音频数据，才再次弄的，不过没兴趣了，就是单纯的弄着玩滴。

> ### 所需要的库
> 
> #### librosa(必须)
> 
> #### pygame(必须)
> 
> #### sounddevice(必须必须必须)
> 
> #### numpy(看需求)
> 
> #### queue(看需求)
> 
> #### serial(看需求)
> 
> #### serial.tools.list_ports(看需求)
> 
> * * *

电脑滴
---

```
import sounddevice as sd
import numpy as np
import librosa
import pygame
import queue
sceen = pygame.display.set_mode([10,100])#设置界面宽高
sceen.fill([0,0,0])##填充背景颜色
pygame.draw.rect(sceen,[255,0,0],[30, 10, 10, 0],0)##画出矩形

def callback(indata,frames, time, status):##回调函数(参数：1.输入信号 2.采样频率 3.应该是时间啦 4.状态)
    global Xdb
    if status:##输出状态 如果回调函数处理太久 会出现输出输入溢出的警告信息
        print(status)
    aaa=np.array(indata[::50],dtype='float64').flatten()##进行转换64位的数据，原本是32位的数据
    Xdb = np.abs(librosa.amplitude_to_db(abs(aaa)))##转换分贝值，并取绝对值
    for i in Xdb:
        q.put(i)##加入队列

stream=sd.InputStream(channels=1,samplerate=44100, callback=callback)#参数：1.声道数 2.采样频率 3.回调函数体 声道数不同，indata的数组维度不同，这采用一维
running = True##运行界面
q=queue.Queue()##创建队列
with stream:##
    while running:##死循环运行
        i=q.get()
        pygame.draw.rect(sceen,[255,0,0],[30, 10, 10, 80],0)##音量柱
        pygame.draw.rect(sceen,[0,0,0],[30, 10, 10, i],0)##黑色蒙版
        pygame.display.update()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False##退出死循环
pygame.quit()
```

ESP8266 单片机和电脑滴
---------------

```
import sounddevice as sd
import numpy as np
import librosa
import pygame
import queue
import serial
import serial.tools.list_ports

def callback(indata,frames, time, status):##回调函数(参数：1.输入信号 2.采样频率 3.应该是时间啦 4.状态)
    global Xdb##这是本人需要的而已
    if status:##输出状态 如果回调函数处理太久 会出现输出输入溢出的警告信息
        print(status)
    aaa=np.array(indata[::100],dtype='float64').flatten()##进行转换，这里取少一点，嘿嘿，看个人
    Xdb = np.abs(librosa.amplitude_to_db(abs(aaa)))##转换分贝值，并取绝对值
    for i in Xdb:
        q.put(i)##加入队列这样子

sceen = pygame.display.set_mode([10,100])#设置界面宽高
sceen.fill([0,0,0])##填充背景颜色
pygame.draw.rect(sceen,[255,0,0],[30, 10, 10, 0],0)##画出矩形
running = True##运行界面
q=queue.Queue()##创建队列
stream=sd.InputStream(channels=1,samplerate=44100, callback=callback)#参数：1.声道数 2.采样频率 3.回调函数体
with serial.Serial('COM7',9600) as ser:##打开串口以及配置对应的波特率，是真的粗糙版
    with stream:##
        while running:##死循环运行
            i=q.get()##获取队列的值
            data=[int(-i+90)]##这是配合单片机的数据而已，看个人处理咯
            pygame.draw.rect(sceen,[255,0,0],[30, 10, 10, 80],0)##整体音量
            pygame.draw.rect(sceen,[0,0,0],[30, 10, 10, i],0)##黑色蒙版
            pygame.display.update()##更新图像而已啦
            if data[0]<0:
                ser.write(bytes([0]))##同下
            else:
                ser.write(bytes(data))##串口发送数据
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False##退出死循环
pygame.quit()
```

ESP8266（Arduino)
----------------

```
#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
#include <avr/power.h> // 16Mhz Adafruit Trket所需
#endif

// Arduino上的哪个引脚与NeoPixels相连
#define PIN        4 // 定义数据输出引脚(,)

// 有多少个像素连接到Arduino上
#define NUMPIXELS 30 // WS2181灯珠个数
Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);//第一个参数灯珠个数,第二个参数输出口位置,第三个参数未知
unsigned char incomedate = 0;
unsigned char dat=0;

void RGB_R_b(unsigned char l){
          for(int i=0; i<l; i++) { 
          pixels.setPixelColor(i, pixels.Color(255, 0, 0));
          }
          for(int i=l; i<30; i++) { 
          pixels.setPixelColor(i, pixels.Color(0, 0, 0));
          }
          pixels.show();
  }

void RGB_G_r(unsigned char l){
          for(int i=0; i<l; i++) { 
          pixels.setPixelColor(i, pixels.Color(0, 255, 0));
          }
          for(int i=l; i<30; i++) { 
          pixels.setPixelColor(i, pixels.Color(255, 0, 0));
          }
          pixels.show();
  }

void RGB_B_g(unsigned char l){
          for(int i=0; i<l; i++) { 
          pixels.setPixelColor(i, pixels.Color(0, 0, 255));
          }
          for(int i=l; i<30; i++) { 
          pixels.setPixelColor(i, pixels.Color(0, 255, 0));
          }
          pixels.show();
  }

void go_to(){
while(dat!=incomedate){
if (dat<incomedate)dat++;
if (dat>incomedate)dat--;
    if (dat<=30){
      RGB_R_b(dat);
      goto bar;
      }
    if (dat>60 && dat<=90){
      RGB_B_g(dat-60);
      goto bar;
      }
    if (30<dat<=60){
      RGB_G_r(dat-30);
      goto bar;
      }

bar:delay(10);
  }
}

void setup() {

  Serial.begin(9600); //设置串口波特率9600
      #if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
    clock_prescale_set(clock_div_1);
    #endif

        pixels.begin(); // INITIALIZE NeoPixel strip object (REQUIRED)
        pixels.setBrightness(2);//调节亮度

}

void loop() {
  if (Serial.available() > 0)//串口接收到数据
  {
    incomedate = Serial.read();//获取串口接收到的数据
    go_to();

  }

}
```

### 后面就是图片了别看了

* * *

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)729  librosa 这个库是可以使用傅里叶变换的 所以弄频段也是可以的 ，但是毕竟是粗糙版本嘛，所以提醒就得了，不弄了。。。加上这个 什么乱七八糟的音乐喷泉啊也行的，我的毕设就是音乐喷泉了。。。。。。。