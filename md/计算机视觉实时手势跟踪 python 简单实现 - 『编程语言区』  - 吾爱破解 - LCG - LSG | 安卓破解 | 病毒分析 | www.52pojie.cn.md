> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1451179&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://avatar.52pojie.cn/data/avatar/000/03/50/75_avatar_middle.jpg)kk159  _ 本帖最后由 kk159 于 2021-6-1 11:00 编辑_  
**这是一个简单的实验，**  
**利用谷歌开发的框架可以更快速帮助我们处理非常基本的 ai 问题,  
需要的类库 opencv-python,mediapipe, 实现实时的手势跟踪**  
效果图:  
![](https://attach.52pojie.cn/forum/202106/01/105109k8ztnjzqoxjfsopn.png)

**image.png** _(99.55 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjI5ODU0M3w2MzEzMWQ1NnwxNjIyNTU1MjE4fDEwNzQxNTd8MTQ1MTE3OQ%3D%3D&nothumb=yes)

2021-6-1 10:51 上传

  
**1. 创建摄像头对象**  
[Python] _纯文本查看_ _复制代码_

```
import cv2
 
cap=cv2.VideoCapture(0)
 
while 1:
    success,img=cap.read()
    cv2.imshow('xxxx',img)
    cv2.waitKey(1)
```

**  
2. 捕获坐标**  
[Python] _纯文本查看_ _复制代码_

```
import mediapipe
 
mpHands=mediapipe.solutions.hands
hands=mpHands.Hands()
 
##cv2的颜色是BGR的，摄像头的是RGB，所有要转换一下
imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    results = hands.process(imgRGB)
    print(results.multi_hand_landmarks)
```

![](https://attach.52pojie.cn/forum/202106/01/105640dzmdja2z3acs29g6.png)

**image.png** _(26.01 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjI5ODU0NHw3ZDI0YzBlNnwxNjIyNTU1MjE4fDEwNzQxNTd8MTQ1MTE3OQ%3D%3D&nothumb=yes)

2021-6-1 10:56 上传

  
**3. 描点和链接骨干显示**  
[Python] _纯文本查看_ _复制代码_

```
mpDraw=mediapipe.solutions.drawing_utils
 
##如果检测到 
if results.multi_hand_landmarks:
##每只手
    for hand in results.multi_hand_landmarks:
        mpDraw.draw_landmarks(img,hand,mpHands.HAND_CONNECTIONS)
```

**完整代码：**  
[Python] _纯文本查看_ _复制代码_

```
# *_* coding : UTF-8 *_*
# author  ：  Leemamas
# 开发时间  ：  2021/6/1  10:24
import cv2
import mediapipe
 
cap=cv2.VideoCapture(0)
mpHands=mediapipe.solutions.hands
hands=mpHands.Hands()
mpDraw=mediapipe.solutions.drawing_utils
 
while 1:
    success,img=cap.read()
 
    imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    results = hands.process(imgRGB)
    ##如果检测到
    if results.multi_hand_landmarks:
        ##每只手
        for hand in results.multi_hand_landmarks:
            mpDraw.draw_landmarks(img,hand,mpHands.HAND_CONNECTIONS)
 
    cv2.imshow('xxxx',img)
    cv2.waitKey(1)
```

扩展：  
![](https://attach.52pojie.cn/forum/202106/01/105924wubenpfl70perbmq.png)

**Snipaste_2021-06-01_10-21-39.png** _(256.17 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjI5ODU0NXxjZTM3ZDg2N3wxNjIyNTU1MjE4fDEwNzQxNTd8MTQ1MTE3OQ%3D%3D&nothumb=yes)

2021-6-1 10:59 上传

  
[Python] _纯文本查看_ _复制代码_

```
class HandLandmark(enum.IntEnum):
  """The 21 hand landmarks."""
  WRIST = 0
  THUMB_CMC = 1
  THUMB_MCP = 2
  THUMB_IP = 3
  THUMB_TIP = 4
  INDEX_FINGER_MCP = 5
  INDEX_FINGER_PIP = 6
  INDEX_FINGER_DIP = 7
  INDEX_FINGER_TIP = 8
  MIDDLE_FINGER_MCP = 9
  MIDDLE_FINGER_PIP = 10
  MIDDLE_FINGER_DIP = 11
  MIDDLE_FINGER_TIP = 12
  RING_FINGER_MCP = 13
  RING_FINGER_PIP = 14
  RING_FINGER_DIP = 15
  RING_FINGER_TIP = 16
  PINKY_MCP = 17
  PINKY_PIP = 18
  PINKY_DIP = 19
  PINKY_TIP = 20
```

[Python] _纯文本查看_ _复制代码_

```
HAND_CONNECTIONS = frozenset([
    (HandLandmark.WRIST, HandLandmark.THUMB_CMC),
    (HandLandmark.THUMB_CMC, HandLandmark.THUMB_MCP),
    (HandLandmark.THUMB_MCP, HandLandmark.THUMB_IP),
    (HandLandmark.THUMB_IP, HandLandmark.THUMB_TIP),
    (HandLandmark.WRIST, HandLandmark.INDEX_FINGER_MCP),
    (HandLandmark.INDEX_FINGER_MCP, HandLandmark.INDEX_FINGER_PIP),
    (HandLandmark.INDEX_FINGER_PIP, HandLandmark.INDEX_FINGER_DIP),
    (HandLandmark.INDEX_FINGER_DIP, HandLandmark.INDEX_FINGER_TIP),
    (HandLandmark.INDEX_FINGER_MCP, HandLandmark.MIDDLE_FINGER_MCP),
    (HandLandmark.MIDDLE_FINGER_MCP, HandLandmark.MIDDLE_FINGER_PIP),
    (HandLandmark.MIDDLE_FINGER_PIP, HandLandmark.MIDDLE_FINGER_DIP),
    (HandLandmark.MIDDLE_FINGER_DIP, HandLandmark.MIDDLE_FINGER_TIP),
    (HandLandmark.MIDDLE_FINGER_MCP, HandLandmark.RING_FINGER_MCP),
    (HandLandmark.RING_FINGER_MCP, HandLandmark.RING_FINGER_PIP),
    (HandLandmark.RING_FINGER_PIP, HandLandmark.RING_FINGER_DIP),
    (HandLandmark.RING_FINGER_DIP, HandLandmark.RING_FINGER_TIP),
    (HandLandmark.RING_FINGER_MCP, HandLandmark.PINKY_MCP),
    (HandLandmark.WRIST, HandLandmark.PINKY_MCP),
    (HandLandmark.PINKY_MCP, HandLandmark.PINKY_PIP),
    (HandLandmark.PINKY_PIP, HandLandmark.PINKY_DIP),
    (HandLandmark.PINKY_DIP, HandLandmark.PINKY_TIP)
])
```

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)GiaoMan-wei 

> [yiwozhutou 发表于 2021-6-1 13:30](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38766407&ptid=1451179)  
> 强啊 现在还不敢输入密码了 咋办  楼上说的延迟 嗯 不知道咋办 呢  大佬强啊  祝你早点去 少林 欧里给

你这。。水的过 fin 了~~~~~![](https://static.52pojie.cn/static/image/smiley/laohu/laohu4.gif)![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)yiwozhutou  强啊 现在还不敢输入密码了 咋办  楼上说的延迟 嗯 不知道咋办 呢  大佬强啊  祝你早点去 少林 欧里给 ![](https://avatar.52pojie.cn/data/avatar/001/56/80/66_avatar_middle.jpg) qianshang666 学到了，谢谢分享![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)龍謹  这个干活含金量高啊，谢谢大佬无私分享！![](https://avatar.52pojie.cn/data/avatar/001/46/54/05_avatar_middle.jpg)pj 明  不错，没事可以下来学习一下![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)上单小鳄鱼  学习学习 去 ![](https://avatar.52pojie.cn/data/avatar/001/27/89/26_avatar_middle.jpg) sam 喵喵 这玩意最大问题就是延迟 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) GiaoMan-wei 强啊，，，大佬，，![](https://static.52pojie.cn/static/image/smiley/laohu/laohu39.gif)![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)cs7392000  感谢楼主分享