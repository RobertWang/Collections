> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/czusM05qvmEHeAMSSUtu3w)

> 本篇博文使用 ESP32-S3 搭建网络摄像头，相比较局域网摄像头，本篇博文将分享如何搭建外网可以访问的网络摄像头。

本篇博文使用 ESP32-S3 搭建网络摄像头，相比较局域网摄像头，本篇博文将分享如何搭建外网可以访问的网络摄像头。

这主要是使用内网穿透技术，内网穿透是为了使具有某一个特定源 IP 地址和源端口号的数据包（这里指局域网摄像头）不被网络地址转换设备屏蔽而正确路由到内网主机。

主要流程分为两步：

1、先实现局域网访问网络摄像头；

2、在此基础，使用内网穿透的方式，搭建外网可访问的网络摄像头。

**局域网摄像头**

ESP32 实现局域网摄像头的方式比较简单，驱动代码如下：

```
#include "esp_camera.h"
#include
//
// WARNING!!! PSRAM IC required for UXGA resolution and high JPEG quality
//            Ensure ESP32 Wrover Module or other board with PSRAM is selected
//            Partial images will be transmitted if image exceeds buffer size
//
//            You must select partition scheme from the board menu that has at least 3MB APP space.
//            Face Recognition is DISABLED for ESP32 and ESP32-S2, because it takes up from 15 
//            seconds to process single frame. Face Detection is ENABLED if PSRAM is enabled as well
// ===================
// Select camera model
// ===================
//#define CAMERA_MODEL_WROVER_KIT // Has PSRAM
// #define CAMERA_MODEL_ESP_EYE // Has PSRAM
//#define CAMERA_MODEL_ESP32S3_EYE // Has PSRAM
//#define CAMERA_MODEL_M5STACK_PSRAM // Has PSRAM
//#define CAMERA_MODEL_M5STACK_V2_PSRAM // M5Camera version B Has PSRAM
//#define CAMERA_MODEL_M5STACK_WIDE // Has PSRAM
//#define CAMERA_MODEL_M5STACK_ESP32CAM // No PSRAM
//#define CAMERA_MODEL_M5STACK_UNITCAM // No PSRAM
//#define CAMERA_MODEL_AI_THINKER // Has PSRAM
//#define CAMERA_MODEL_TTGO_T_JOURNAL // No PSRAM
//#define CAMERA_MODEL_XIAO_ESP32S3 // Has PSRAM
// ** Espressif Internal Boards **
//#define CAMERA_MODEL_ESP32_CAM_BOARD
//#define CAMERA_MODEL_ESP32S2_CAM_BOARD
//#define CAMERA_MODEL_ESP32S3_CAM_LCD
#define CAMERA_MODEL_DFRobot_FireBeetle2_ESP32S3 // Has PSRAM
//#define CAMERA_MODEL_DFRobot_Romeo_ESP32S3 // Has PSRAM
#include "camera_pins.h"
#include "DFRobot_AXP313A.h"
DFRobot_AXP313A axp;
// ===========================
// Enter your WiFi credentials
// ===========================
const char* ssid = "";
const char* password = "";
void startCameraServer();
void setupLedFlash(int pin);
void setup() {
  Serial.begin(115200);
  Serial.setDebugOutput(true);
  Serial.println();
  while(axp.begin() != 0){
    Serial.println("init error");
    delay(1000);
  }
  axp.enableCameraPower(axp.eOV2640);  // 给摄像头供电
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sccb_sda = SIOD_GPIO_NUM;
  config.pin_sccb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.frame_size = FRAMESIZE_UXGA;
  config.pixel_format = PIXFORMAT_JPEG; // for streaming
  //config.pixel_format = PIXFORMAT_RGB565; // for face detection/recognition
  config.grab_mode = CAMERA_GRAB_WHEN_EMPTY;
  config.fb_location = CAMERA_FB_IN_PSRAM;
  config.jpeg_quality = 12;
  config.fb_count = 1;
  // if PSRAM IC present, init with UXGA resolution and higher JPEG quality
  //                      for larger pre-allocated frame buffer.
  if(config.pixel_format == PIXFORMAT_JPEG){
    if(psramFound()){
      config.jpeg_quality = 10;
      config.fb_count = 2;
      config.grab_mode = CAMERA_GRAB_LATEST;
    } else {
      // Limit the frame size when PSRAM is not available
      config.frame_size = FRAMESIZE_SVGA;
      config.fb_location = CAMERA_FB_IN_DRAM;
    }
  } else {
    // Best option for face detection/recognition
    config.frame_size = FRAMESIZE_240X240;
#if CONFIG_IDF_TARGET_ESP32S3
    config.fb_count = 2;
#endif
  }
#if defined(CAMERA_MODEL_ESP_EYE)
  pinMode(13, INPUT_PULLUP);
  pinMode(14, INPUT_PULLUP);
#endif
  // camera init
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }
  sensor_t * s = esp_camera_sensor_get();
  // initial sensors are flipped vertically and colors are a bit saturated
  if (s->id.PID == OV3660_PID) {
    s->set_vflip(s, 1); // flip it back
    s->set_brightness(s, 1); // up the brightness just a bit
    s->set_saturation(s, -2); // lower the saturation
  }
  // drop down frame size for higher initial frame rate
  if(config.pixel_format == PIXFORMAT_JPEG){
    s->set_framesize(s, FRAMESIZE_QVGA);
  }
#if defined(CAMERA_MODEL_M5STACK_WIDE) || defined(CAMERA_MODEL_M5STACK_ESP32CAM)
  s->set_vflip(s, 1);
  s->set_hmirror(s, 1);
#endif
#if defined(CAMERA_MODEL_ESP32S3_EYE)
  s->set_vflip(s, 1);
#endif
// Setup LED FLash if LED pin is defined in camera_pins.h
#if defined(LED_GPIO_NUM)
  setupLedFlash(LED_GPIO_NUM);
#endif
  WiFi.begin(ssid, password);
  WiFi.setSleep(false);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  startCameraServer();
  Serial.print("Camera Ready! Use 'http://");
  Serial.print(WiFi.localIP());
  Serial.println("' to connect");
}
void loop() {
  // Do nothing. Everything is done in another task by the web server
  delay(10000);
}

```

代码中有几点需要注意：

1、宏定义选择适配的摄像头模式。

```
// ===================
// Select camera model
// ===================
//#define CAMERA_MODEL_WROVER_KIT // Has PSRAM
// #define CAMERA_MODEL_ESP_EYE // Has PSRAM
//#define CAMERA_MODEL_ESP32S3_EYE // Has PSRAM
//#define CAMERA_MODEL_M5STACK_PSRAM // Has PSRAM
//#define CAMERA_MODEL_M5STACK_V2_PSRAM // M5Camera version B Has PSRAM
//#define CAMERA_MODEL_M5STACK_WIDE // Has PSRAM
//#define CAMERA_MODEL_M5STACK_ESP32CAM // No PSRAM
//#define CAMERA_MODEL_M5STACK_UNITCAM // No PSRAM
//#define CAMERA_MODEL_AI_THINKER // Has PSRAM
//#define CAMERA_MODEL_TTGO_T_JOURNAL // No PSRAM
//#define CAMERA_MODEL_XIAO_ESP32S3 // Has PSRAM
// ** Espressif Internal Boards **
//#define CAMERA_MODEL_ESP32_CAM_BOARD
//#define CAMERA_MODEL_ESP32S2_CAM_BOARD
//#define CAMERA_MODEL_ESP32S3_CAM_LCD
#define CAMERA_MODEL_DFRobot_FireBeetle2_ESP32S3 // Has PSRAM
//#define CAMERA_MODEL_DFRobot_Romeo_ESP32S3 // Has PSRAM

```

2、无线路由器 SSID 和密码要填写正确。

```
// ===========================
// Enter your WiFi credentials
// ===========================
const char* ssid = "";
const char* password = "";

```

3、给摄像头供电

```
axp.enableCameraPower(axp.eOV2640);  // 给摄像头供电

```

4、板卡需要外接天线，否则可能无法连接路由器。

编译下载程序到板卡中，确保局域网访问网络摄像头可正常使用。

  

**内网穿透网络摄像头**

内网穿透我们使用花生壳这款软件提供的内网穿透服务。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwH2icGJHjeP3o8CvyP3ibWnuek7H7cuQ4e0jfmFenQ1MOdMlHhEprGScNetYm7SiaX1RnwLYV7JwE4pQ/640?wx_fmt=png)

在官网下载 APP：https://hsk.oray.com/

下载安装完成后，在内网穿透服务点击新建映射，如下图所示： 

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwH2icGJHjeP3o8CvyP3ibWnueW6ic8fqG8Qz6u1ygJ0gtubKfB20ibWib9uJ5VMEUILLiaSl7SqF3IEcMicw/640?wx_fmt=png)

填写新建映射的基本信息，请注意内网主机和内网端口是局域网摄像头的主机和端口（端口默认为 80） ，如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwH2icGJHjeP3o8CvyP3ibWnueFTeHHpBvwpmnkAM0C1agJFJfb9bibY1AmJyCdUX2BCaFmS04BeKjQjw/640?wx_fmt=png)

新建映射完成后，可以在 APP 看到新增的设备列表，如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwH2icGJHjeP3o8CvyP3ibWnueSkvWrPVWxp2Rf4n1hADywLSAtYUO1IPuNicZFrLywrTQw3OeuY6qoTw/640?wx_fmt=png)

复制访问网址，在浏览器中打开：http://2j90962r69.goho.co:47918/

即使不在同一个局域网内也可以正常访问摄像头啦。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwH2icGJHjeP3o8CvyP3ibWnueibrrIv3mKMAGaW2oKJaNX1EKt6XVMAnVCiaT1dm6o7yZp3DvvKfZpAWg/640?wx_fmt=png)