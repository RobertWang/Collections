> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247513030&idx=2&sn=1a8ba198560a212fb8721aac477d8b9a&chksm=e969edcbde1e64ddf8e2d17ddc3acd0f163cf6f2d592fb463effbcf50753bfe7e4dab062ad16&scene=21#wechat_redirect)

> 编辑：乐乐 | 来自：半壶砂
> 
> 链接：cnblogs.com/halfsand/p/7976636.html

[Pythn 人工智能技术 (ID:coder_experience) 第 768 期推文](http://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247489525&idx=1&sn=26e9ce9d935a845d03ce4a3d30ebc975&chksm=e96a49f8de1dc0eeda78a0f1fa1ab74bfa13222fecbb924f52e59b59cd8b431d6e0fd00cc1cb&scene=21#wechat_redirect)

**正文**

 **大家好，我是 Python 人工智能技术**

自动追踪算法，在我们制作射击类游戏时经常会用到。这个听起来很高大上的东西，其实并不是军事学的专利，从数学上来说就是解微分方程。

这个没有点数学基础是很难算出来的。但是我们有了计算机就不一样了，依靠计算机极快速的运算速度，我们利用微分的思想，加上一点简单的三角学知识，就可以实现它。

好，话不多说，我们来看看它的算法原理，看图：  
![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIrbLuffKE9vluxJFu49Ie7fJCVhe3icIkibjMVhJhK1JhglI0fmUiaLtgXkLbDPic0jxOc953DfI4aeBA/640?wx_fmt=png)

由于待会要用`pygame`演示，它的坐标系是 y 轴向下，所以这里我们也用 y 向下的坐标系。

算法总的思想就是根据上图，把时间 t 分割成足够小的片段（比如 1/1000，这个时间片越小越精确），每一个片段分别构造如上三角形，计算出导弹下一个时间片走的方向（即∠a）和走的路程（即`vt=|AC|`），这时候目标再在第二个时间片移动了位置，这时刚才计算的 C 点又变成了第二个时间片的初始点，这时再在第二个时间片上在 C 点和新的目标点构造三角形计算新的 vt，然后进入第三个时间片，如此反复即可。

假定导弹和目标的初始状态下坐标分别是`(x1,y1),(x,y)`，构造出直角三角形 ABE，这个三角形用来求∠a 的正弦和余弦值，因为 vt 是自己设置的，我们需要计算 A 到 C 点 x 和 y 坐标分别移动了多少，移动的值就是`AD`和`CD`的长度，这两个分别用`vt`乘`cos(a)`和`sin(a)`即可。

计算 sin(a) 和 cos(a)，正弦对比斜，余弦邻比斜，斜边可以利用两点距离公式计算出，即：

![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIrbLuffKE9vluxJFu49Ie7f2ayeI03U4j2B2Ou0MtK70ShC22PaxGdmrl5Ufw88JRboeCQFJleZ1A/640?wx_fmt=png)

于是  
![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIrbLuffKE9vluxJFu49Ie7f6JJNbpwdoblTaS4Nl5TOiaNOfIbmk1eCRHL8PFqS7icy3akxhXEsLtfg/640?wx_fmt=png)

AC 的长度就是导弹的速度乘以时间即 |AC|=vt，然后即可计算出 AD 和 CD 的长度，于是这一个时间片过去后，导弹应该出现在新的位置 C 点，他的坐标就是老的点 A 的 x 增加 AD 和 y 减去 CD。

于是，新的 C 点坐标就是：

![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIrbLuffKE9vluxJFu49Ie7fEqxeQp7K6p3cub3yictsBqkX0Y4sxzbYqWzdS56dgUeRqlnldsCzKLA/640?wx_fmt=png)

只要一直反复循环执行这个操作即可，好吧，为了更形象，把第一个时间片和第二个时间片放在一起看看：  
![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIrbLuffKE9vluxJFu49Ie7fIou283dXVbK3sDiaNO57gR127H6EZiaYsfJewLNG0ia1eco02jTXYf5ibA/640?wx_fmt=png)

第一个是时间片构造出的三角形是 ABE，经过一个时间片后，目标从 B 点走到了 D 点，导弹此时在 C 点，于是构造新的三角形 CDF，重复刚才的计算过程即可，图中的角∠b 就是导弹需要旋转的角度, 现实中只需要每个时间片修正导弹的方向就可以了，具体怎么让导弹改变方向，这就不是我们需要研究的问题了

好，由于最近在用`Python`的`pygame`库制作小游戏玩，接下来我们就用 pygame 来演示一下这个效果，效果如下图：  
![图片](https://mmbiz.qpic.cn/mmbiz_gif/PvP6qjUpvIrbLuffKE9vluxJFu49Ie7fHh5ic5kPOF662kTUKx9mvQuF1e0vXu2YnswTpziczrSPgIlDW0iaN8pDQ/640?wx_fmt=gif)

很简单的代码如下：

```
import pygame,sys
from math import *
pygame.init()
screen=pygame.display.set_mode((800,700),0,32)
missile=pygame.image.load('element/red_pointer.png').convert_alpha()
x1,y1=100,600           #导弹的初始发射位置
velocity=800            #导弹速度
time=1/1000             #每个时间片的长度
clock=pygame.time.Clock()
old_angle=0
while True:
    for event in pygame.event.get():
        if event.type==pygame.QUIT:
            sys.exit()
    clock.tick(300)
    x,y=pygame.mouse.get_pos()          #获取鼠标位置，鼠标就是需要打击的目标
    distance=sqrt(pow(x1-x,2)+pow(y1-y,2))      #两点距离公式
    section=velocity*time               #每个时间片需要移动的距离
    sina=(y1-y)/distance
    cosa=(x-x1)/distance
    angle=atan2(y-y1,x-x1)              #两点线段的弧度值
    x1,y1=(x1+section*cosa,y1-section*sina)
    d_angle = degrees(angle)        #弧度转角度
    screen.blit(missile, (x1-missile.get_width(), y1-missile.get_height()/2))
    dis_angle=d_angle-old_angle          #dis_angle就是到下一个位置需要改变的角度
    old_angle=d_angle                    #更新初始角度
    pygame.display.update()


```

如果仅把导弹考虑为一个质点的话，那么以上算法就已经足矣，我没有做导弹的旋转，因为一个质点也不分头尾不需要旋转，当然这前提得是你加载的导弹图片很小的时候不旋转看起来也没什么问题。但是在 pygame 里面做旋转并不是一件容易的事情，我们先把图片替换成一张矩形的，再加入旋转函数看看效果如何

![图片](https://mmbiz.qpic.cn/mmbiz_png/HZW0wwFxbQArrfsuP2naEB7kUYCpibicM04naicTTGXPLlPow3NNDTr5sJOZXK6fIh2523JdsofubMXqqSpcXelzQ/640?wx_fmt=png)

```
missiled = pygame.transform.rotate(missile, -(d_angle))
screen.blit(missiled, (x1-missile.get_width(), y1-
missile.get_height()/2))


```

因为图片的坐标点是它的左上角的点，所以如果我们想让图片的坐标固定在箭头尖点，那么把图片实际打印位置 x 减少图片长度，y 减少一半宽度就行。但是实际运行效果并不好：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/PvP6qjUpvIrbLuffKE9vluxJFu49Ie7fZibn5SuD5ibMKwqiasJUlSRs6MVZialP5Qu2OHlPl0DtdEUpiaXvVfNnxPg/640?wx_fmt=gif)

大致方向相同，但是图片箭头的尖点并没有一直跟随鼠标，这是为什么呢。经过一番研究，我发现原来是这个图旋转的机制问题，我们看看旋转后的图片变成什么样了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HZW0wwFxbQArrfsuP2naEB7kUYCpibicM0pJ3Jsp4Dw3rKodicUlWAszdanHGZkmmIymIjicicSdN2pcZsg96Ewkn2Q/640?wx_fmt=png)

旋转后的图片变成了蓝色的那个范围，根据旋转角度的不同，所变成的图片大小也不一样，我们看旋转 90 的情况

![图片](https://mmbiz.qpic.cn/mmbiz_png/HZW0wwFxbQArrfsuP2naEB7kUYCpibicM0sSXw9zIZ11vyyicwicONbwoXaXptwicfykc2icU7SVpLY0IXx3shjgMHCQ/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/HZW0wwFxbQArrfsuP2naEB7kUYCpibicM0cOsfFb47nhjh6qPoia1V9SP4qYmnmbpDIb3oA96AruOfqANrlyJAtag/640?wx_fmt=png)

我们发现，旋转后的图片不仅面积变大了，导弹头的位置也变了。那应该怎么解决这个问题呢？思路是，每一次旋转图片以后，求出旋转图的头位置（图中的绿色箭头点），然后把绿图的打印位置移动一下，下，x，y 分别移动两个头的距离，就可以让旋转后的导弹头对准实际我们参与运算的那个导弹头的位置，移动后应该是这样的：

![图片](https://mmbiz.qpic.cn/mmbiz_png/HZW0wwFxbQArrfsuP2naEB7kUYCpibicM0aBLnFbNMlia2BVibXunkxAbYbTMR0dtMoa0jNfIbmLUW1yFHBE0MoRaQ/640?wx_fmt=png)

这样，两个导弹头的点就一致了。接下来我们分析求旋转后的导弹头的算法。根据旋转角度的不同，旋转角在不同象限参数不一样，所以我们分为这四种情况。另外，搜索公众号 Linux 就该这样学后台回复 “git 书籍”，获取一份惊喜礼包。

1，2 象限  
![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIrbLuffKE9vluxJFu49Ie7fINuW9WASb8taRKkLCaaR1ibWgcXLgdU7fffLSUNVM089qbAHl1FJicdw/640?wx_fmt=png)

3，4 象限，它的旋转只有正负 0—180，所以 3，4 象限就是负角  
![图片](https://mmbiz.qpic.cn/mmbiz_png/PvP6qjUpvIrbLuffKE9vluxJFu49Ie7fNbOTrE7vFQpVVB5xDkIOr495UsNMf4cdvb0OKicHTDAOPpvNrias0uvg/640?wx_fmt=png)

显示图片的时候我们将他移动

```
screen.blit(missiled, (x1-width+(x1-C[0]),y1-height/2+(y1-C[1])))


```

这里的 (`x1-width, y1-height/2`) 其实才是上图中的 (x1, y1)

所以最后我们加入相关算法代码，效果就比较完美了

![图片](https://mmbiz.qpic.cn/mmbiz_gif/HZW0wwFxbQArrfsuP2naEB7kUYCpibicM0G0j1RsTV3UeB5YtvZ7apvAsVohBG1GxB08RMQdjawYoIF5qj62zTVA/640?wx_fmt=gif)

大功告成，最后附上全部的算法代码

```
import pygame,sys
from math import *
pygame.init()
font1=pygame.font.SysFont('microsoftyaheimicrosoftyaheiui',23)
textc=font1.render('*',True,(250,0,0))
screen=pygame.display.set_mode((800,700),0,32)
missile=pygame.image.load('element/rect1.png').convert_alpha()
height=missile.get_height()
width=missile.get_width()
pygame.mouse.set_visible(0)
x1,y1=100,600           #导弹的初始发射位置
velocity=800            #导弹速度
time=1/1000             #每个时间片的长度
clock=pygame.time.Clock()
A=()
B=()
C=()
while True:
    for event in pygame.event.get():
        if event.type==pygame.QUIT:
            sys.exit()
    clock.tick(300)
    x,y=pygame.mouse.get_pos()          #获取鼠标位置，鼠标就是需要打击的目标
    distance=sqrt(pow(x1-x,2)+pow(y1-y,2))      #两点距离公式
    section=velocity*time               #每个时间片需要移动的距离
    sina=(y1-y)/distance
    cosa=(x-x1)/distance
    angle=atan2(y-y1,x-x1)              #两点间线段的弧度值
    fangle=degrees(angle)               #弧度转角度
    x1,y1=(x1+section*cosa,y1-section*sina)
    missiled=pygame.transform.rotate(missile,-(fangle))
    if 0<=-fangle<=90:
        A=(width*cosa+x1-width,y1-height/2)
        B=(A[0]+height*sina,A[1]+height*cosa)

    if 90<-fangle<=180:
        A = (x1 - width, y1 - height/2+height*(-cosa))
        B = (x1 - width+height*sina, y1 - height/2)

    if -90<=-fangle<0:
        A = (x1 - width+missiled.get_width(), y1 - height/2+missiled.get_height()-height*cosa)
        B = (A[0]+height*sina, y1 - height/2+missiled.get_height())

    if -180<-fangle<-90:
        A = (x1-width-height*sina, y1 - height/2+missiled.get_height())
        B = (x1 - width,A[1]+height*cosa )

    C = ((A[0] + B[0]) / 2, (A[1] + B[1]) / 2)

    screen.fill((0,0,0))
    screen.blit(missiled, (x1-width+(x1-C[0]),y1-height/2+(y1-C[1])))
    screen.blit(textc, (x,y)) #鼠标用一个红色*代替
    pygame.display.update()


```

以上便是用 Python 模拟导弹自动追踪的代码实例。

****你还有什****么想要补充的吗？****  

> 免责声明：本文内容来源于网络，文章版权归原作者所有，意在传播相关技术知识 & 行业趋势，供大家学习交流，若涉及作品版权问题，请联系删除或授权事宜。