> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.51hei.com](http://www.51hei.com/BBS/dpj-78772-1.html)

> 下图为点亮一个 LED 的两幅图

本文是关于声音传感器模块 MK152 的一些资料及一个简单实例。

其实这个实例很简单，就是一个开关功能。

下图为点亮一个 LED 的两幅图

![](http://c.51hei.com/d/forum/201703/11/163957ekyl5ujmayk3a3m2.png)

（点亮前）

![](http://c.51hei.com/d/forum/201703/11/164043fk669ffjy3mlbkpb.png)

（点亮后）

这个的实例所需元器件为：单片机最小系统（也可以不要，只要能提供 5v 电压即可）、声音传感器模块一块、LED 一个、杜邦线四根。

接线为：用杜邦线将传感器模块的 VCC 和 GND 分别与单片机最小系统的 VCC 和 GND 连接，提供电源；用杜邦线将传感器模块的 OUT 与 LED 的正极连接，用杜邦线将单片机最小系统的 GND 与 LED 的负极连接。至此实例就做完了。

接下来看看效果：打开单片机最小系统的电源开关，拍手观看 LED 的亮灭。若灵敏度不理想可调节蓝色数字电位器。

![](http://c.51hei.com/d/forum/201703/11/172556f9c9161zez1gc9kt.png)

原理图：

![](http://c.51hei.com/d/forum/201703/11/172314lrj5ledvfj3vqv5n.png)

单片机源程序：

1.  /******************************************  
    
2.  传感器触发测试  
    
3.  单片机：STC89C52  
    
4.  波特率：9600  
    
5.  *****************************************/  
    
6.  #include <reg52.h>  
    
7.  unsigned char date;  
    
8.  #define uchar unsigned char  
    
9.  #define uint unsigned int  
    
10.  sbit key1=P0^1;  
    
11.    
    
12.    
    
13.  /* 函数申明 -----------------------------------------------*/  
    
14.  void delay(uint z);  
    
15.  void Initial_com(void);  
    
16.    
    
17.  //***********************************************************  
    
18.    
    
19.  /*  
    
20.  ********************************************************************************  
    
21.  ** 函数名称 ： delay(uint z)  
    
22.  ** 函数功能 ： 延时函数  
    
23.  ********************************************************************************  
    
24.  */  
    
25.  void delay(uint z)  
    
26.  {                                                                                                                                                                                            
    
27.      uint i,j;  
    
28.      for(i=z;i>0;i--)  
    
29.          for(j=110;j>0;j--);  
    
30.  }  
    
31.    
    
32.    
    
33.  //******************************  
    
34.    
    
35.  //***** 串口初始化函数 ***********  
    
36.    
    
37.  //******************************  
    
38.  void Initial_com(void)  
    
39.  {  
    
40.  EA=1;        // 开总中断  
    
41.  ES=1;        // 允许串口中断  
    
42.  ET1=1;        // 允许定时器 T1 的中断  
    
43.  TMOD=0x20;   // 定时器 T1，在方式 2 中断产生波特率  
    
44.  PCON=0x00;   //SMOD=0  
    
45.  SCON=0x50;   // 方式 1 由定时器控制  
    
46.  TH1=0xfd;    // 波特率设置为 9600  
    
47.  TL1=0xfd;  
    
48.  TR1=1;       // 开定时器 T1 运行控制位  
    
49.    
    
50.  }  
    
51.    
    
52.    
    
53.    
    
54.    
    
55.  //*************************  
    
56.  //********** 主函数 *********  
    
57.  //*************************  
    
58.  main()  
    
59.  {  
    
60.           Initial_com();  
    
61.           while(1)  
    
62.           {  
    
63.            
    
64.                    if(key1==0)  
    
65.                  {  
    
66.                          delay();          // 消抖动  
    
67.                          if(key1==0)          // 确认触发  
    
68.                          {  
    
69.                                   SBUF=0X01;  
    
70.                                   delay(200);  
    
71.                                    
    
72.                            
    
73.                          }  
    
74.               
    
75.                  }  
    
76.                     
    
77.                    if(RI)  
    
78.                    {  
    
79.                          date=SBUF;    // 单片机接受  
    
80.                            
    
81.    
    
82.  ………… 余下代码请下载附件…………  
    

_复制代码_

![](http://c.51hei.com/d/forum/201703/11/172638ehhujbynjhf2fhyz.png)

下面是一些文字资料，感兴趣的可下载：

 ![](http://www.51hei.com/BBS/static/image/filetype/rar.gif) [声音传感器资料. rar](http://www.51hei.com/bbs/forum.php?mod=attachment&aid=NTE1MjR8ZDhmNWRiNWN8MTYyMjQzNTcxOHwwfDc4Nzcy) _(608.57 KB, 下载次数: 84)_