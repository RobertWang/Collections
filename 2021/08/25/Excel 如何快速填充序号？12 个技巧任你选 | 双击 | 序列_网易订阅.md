> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.163.com](https://www.163.com/dy/article/FJCGUR5R0536A55Q.html)

> Excel 如何快速填充序号？12 个技巧任你选, 序号, 双击, 序列

用 Excel 制作表格时，经常要给表格内容添加序号，你是一个个手工输入吗？今天小编就以 Excel 2019 版本为例，教大家一些序号的技巧，别以为太简单，看完再说，以下技巧知道 5 个的举手，连 5 个都不知道的那就点赞分享吧！

**一、拖动生成连续序号**

较早的 Excel 版本中，输入一个 1，然后选中再下拉会自动填充一个连续的序号；

在 Excel2019 版本中，需要先按住 Ctrl 键不放，再下拉填充生成连续的序号，动图演示如下：

![](http://dingyue.ws.126.net/2020/0806/27a10dc1g00qen70m003uc0008000h8m.gif)  

**二、拖动选择生成连续序号**

在单元格输入 1 后，再选中向下拖动，停止拖动后在右下角出现【自动填充选项】，选择【填充序列】，一个从 1 开始的序号就生成了，动图演示如下：

![](http://dingyue.ws.126.net/2020/0806/c0570076g00qen70m003yc0008000h8m.gif)  

**三、双击生成序号**

双击生成序号适用于需要填充序号相邻列有数据的情况，可以直接双击单元格右下角，完成填充序号

分别在 A1、A2 单元格输入 1、2 ，然后选中这两个单元格，双击右下角，快速完成序号填充，动图演示如下：

![](http://dingyue.ws.126.net/2020/0806/f4f33d36g00qen70m003oc0008000h8m.gif)  

**四、利用名称框输入序号**

此方法适用于需要填充的序号较多时，比如我们需要在 A 列中生成 10000 个序号

在名称框中输入：A1：A10000 按回车键，这时 A1：A10000 区域被选中了，不要点击鼠标键，直接键盘输入：=ROW（），按 Ctrl + 回车键，10000 个序号就生成了，动图演示如下：

![](http://dingyue.ws.126.net/2020/0806/14322dc4g00qen70m005yc0008000h8m.gif)  

**五、填充 10000 个序号**

上面是利用名称框中公式生成的 10000 个序号，再教你一个方法

在 A1 单元输入 1，然后选中 A1 单元格，点击【开始】选项卡中的【填充】按钮，选择【序列】

在弹出的序列窗口中，序列产生在选择【列】，步长值输入 1 ，终止值输入 10000，最后点击【确定】10000 个序号就生成了，动图演示如下：

![](http://dingyue.ws.126.net/2020/0806/7a09f020g00qen70m00boc000cg00k8m.gif)  

**六、按部门填充序号**

表格中按部门进行了分类，如何按不同部门填充序号都是从 1 开始，并且同一部门要按顺序递增？

在 A2 单元格输入公式：=COUNTIF(B$2:B2,B2) 然后再选中 A2 单元格，鼠标双击右下角完成序号填充。

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0806%2F222a330ej00qen70m0027c000f200mim.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  

**七、隔行填充序号**

表格中按小计进行了求和，那么如何隔行填充序号，并且每组序号从 1 顺序递增？

1、选择序号数据列，按 Ctrl+G 键，弹出的窗口中点击【定位条件】，选择【空值】，点击【确定】；

2、在输入栏中输入公式：=IF(ISTEXT(B2),1,B2+1)，输入完成后按 Ctrl + 回车键，瞬间完成自动填充序号。演示如下图：

![](http://dingyue.ws.126.net/2020/0806/7408d7f6g00qen70m00krc000f000hwm.gif)  

**八、筛选后保持连续的序号**

当表格按常规方法填充完成序号后，使用筛选功能时，筛选出一部分数据，序号可能会发生错乱，如何让序号筛选后也是从 1 开始递增显示？

在 A2 单元格输入公式：=SUBTOTAL(3,B$1:B2)-1 再双击填充公式即可

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0806%2F6a7ea212j00qen70m0028c000ew00m3m.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  

再筛选时序号还是从 1 开始递增，效果如下：

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0806%2Fdf64205dj00qen70m000wc0009f00d6m.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  

**九、合并单元格填充序号**

表格中按部门合并了单元格，如何填充序号呢？

首先选中需要填充序号的合并单元格区域 A2：A21，然后输入公式：=MAX($A$1:A1)+1

输入完成后按 Ctrl + 回车键确认，合并单元格序号填充完成了。

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0806%2Fd777649dj00qen70m001zc000ds00m6m.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  

**十、生成按规律递增序号**

表格中需要生成 1、1、2、2、3、3…… 或 1、1、1、2、2、2、3、3、3……，即重复 N 个相同数值的递增序列

通用公式，直接套用：**＝INT（ROE（n:n）/n** 其中 n 代表重复的数值个数。

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0806%2Ff4f7515cj00qen70m0015c000db00h9m.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  

**十一、生成循环序号**

生成 1、2、1、2…… 或 1、2、3、1、2、3…… 序列，即 1 至 N 的循环序列

通用公式：**=MOD(ROW(n:n),n)+1**，n 代表循环不到几

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0806%2F95ad27d2j00qen70m0015c000db00hbm.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  

**十二、生成从 N 开始的 N 次循环的递增序号**

生成 2、2、4、4…… 或 3、3、3、6、6、6…… ，即从 N 开始 N 次循环的序号

通用公式：**=CEILING(ROW(1:1),n)**n 代表从几开始

![](https://nimg.ws.126.net/?url=http%3A%2F%2Fdingyue.ws.126.net%2F2020%2F0806%2Fb84d4618j00qen70m0017c000dk00h3m.jpg&thumbnail=650x2147483647&quality=80&type=jpg)  

小伙伴们，在使用 Excel 中还碰到过哪些问题，评论区留言一起讨论学习，坚持原创不易，您的点赞转发就是对小编最大的支持

特别声明：以上内容 (如有图片或视频亦包括在内) 为自媒体平台 “网易号” 用户上传并发布，本平台仅提供信息存储服务。

Notice: The content above (including the pictures and videos if any) is uploaded and posted by a user of NetEase Hao, which is a social media platform and only provides information storage services.