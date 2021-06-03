> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1452053&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline)  ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) 蜀黍说不能太长 自己东凑西凑来的，而且是用 selenium 的... 效率很低... 因为不会 requests... 大佬勿喷...  
作用：联通话费购每天抽奖 3 次。每天签到。将签到情况 + 近 3 次中奖记录推送。  
PS：一般中不到好东西，最多就 2 块话费卷，累积着可能有用吧.... 办公电脑空闲... 可以挂着定时跑... 分享下..  
[Python] _纯文本查看_ _复制代码_

```
from selenium import webdriver
import datetime
from time import sleep
from selenium.webdriver.common.touch_actions import TouchActions
import json
import requests
 
# 网址
url = "https://atp.bol.wo.cn/atplottery/ACT202009101956022770009xRb2UQ?product=hfgo&str="
url2 = "https://account.bol.wo.cn/accountlogin?clientId=activetemplate&redirectUrl=https%3A%2F%2Fatp.bol.wo.cn%2Fatplottery%2FACT202009101956022770009xRb2UQ%3Fproduct%3Dhfgo%26str%3D"
url3 = "https://atp.bol.wo.cn/atpsign/ACT202012221038331042965g65tNa?product=hfgo%3FactId%3D517&product=hfgo&actSetId=2426&imgUrl=https%3A%2F%2Fatp.bol.wo.cn%2Fatp_resource%2Fupload%2F%2Fatp_resource%2F2020%2F09%2F27%2F7c07b0f7dc84ef08f6f939f06e309690.png&targetActId=1516"
url4 = "https://atp.bol.wo.cn/atplottery/ACT202009101956022770009xRb2UQ/winRecord?id=517&product=hfgo"
haoma = "XXXXX"# XXXX改为自己号码
mima = "XXXXX"#XXXX改为自己的密码
browser = webdriver.Chrome()# 浏览器驱动
browser.get(url2)
sleep(3)
browser.find_element_by_xpath('//*[@id="cuauth"]/div/div[3]/div[1]/input').send_keys(haoma)
browser.find_element_by_xpath('//*[@id="cuauth"]/div/div[3]/div[2]/input').send_keys(mima)
browser.find_element_by_xpath('//*[@id="cuauth"]/div/div[3]/div[4]/span').click()
browser.find_element_by_xpath('//*[@id="cuauth"]/div/div[3]/div[3]/button').click()
sleep(8)
browser.find_element_by_xpath('//*[@id="turntable"]/div[2]/div[3]/div[1]/div[1]').click()
sleep(3)
browser.find_element_by_xpath('//*[@id="turntable"]/div[2]/div[8]/div[2]/button[2]').click()
sleep(10)
browser.get(url)
sleep(5)
browser.find_element_by_xpath('//*[@id="turntable"]/div[2]/div[3]/div[1]/div[1]').click()
sleep(10)
browser.get(url)
sleep(5)
browser.find_element_by_xpath('//*[@id="turntable"]/div[2]/div[3]/div[1]/div[1]').click()
sleep(10)
browser.get(url3)
sleep(5)
browser.find_element_by_xpath('//*[@id="dailysign"]/div[2]/div[3]/div[3]').click()
sleep(3)
browser.get(url3)
sleep(5)
yue1 = browser.find_element_by_xpath('//*[@id="dailysign"]/div[2]/div[3]/div[1]').text
sleep(3)
browser.get(url4)
sleep(3)
jl1 = browser.find_element_by_xpath('//*[@id="app"]/div/div[2]/div/div/div[1]/div[2]').text
jl2 = browser.find_element_by_xpath('//*[@id="app"]/div/div[2]/div/div/div[2]/div[2]').text
jl3 = browser.find_element_by_xpath('//*[@id="app"]/div/div[2]/div/div/div[3]/div[2]').text
sleep(3)
url='http://pushplus.hxtrip.com/send'
#'token':'XXXXX'改为自己的
data={'token':'XXXXX','title':'联通话费购','content':yue1+'||'+jl1+jl2+jl3,'template':'html'}
headers={
    "User-Agent":"Mozilla/5.0 (Linux; Android 9; SM-A102U) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.93 Mobile Safari/537.36",
    'Content-Type': 'application/json'}
res=requests.post(headers=headers,url=url,data=json.dumps(data),timeout=10)
 
browser.quit()
```

![](https://attach.52pojie.cn/forum/202106/02/143509ma0ctc880ztcx8tf.png)

**image.png** _(141.01 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjI5ODgxN3wxMTkxMTk4MXwxNjIyNjQyNDA3fDEwNzQxNTd8MTQ1MjA1Mw%3D%3D&nothumb=yes)

2021-6-2 14:35 上传

![](https://attach.52pojie.cn/forum/202106/02/144102mw9m2ll56kbwz16l.png)

**image.png** _(19.8 KB, 下载次数: 1)_

[下载附件](forum.php?mod=attachment&aid=MjI5ODgxOHwxYjhmN2U5MnwxNjIyNjQyNDA3fDEwNzQxNTd8MTQ1MjA1Mw%3D%3D&nothumb=yes)

2021-6-2 14:41 上传

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)agent011  没有大佬给打包一下吗 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) BIGOcean 楼主如果有时间的话，可以试着出个腾讯云函数版本的签到，这样每天都能触发抽奖，自己也不用管了，有 bug 也可以通过 serveky 报错。![](https://static.52pojie.cn/static/image/smiley/default/42.gif)![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)penz  试试看，谢谢 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) liujkk 有成品的吗 ![](https://avatar.52pojie.cn/data/avatar/000/45/60/24_avatar_middle.jpg) ghoob321 一般中不到好东西，最多就 2 块话费卷 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) savie 知识改变命运啊。。。![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)RuMeng  这不是 api，是通过浏览器操作的 ![](https://avatar.52pojie.cn/data/avatar/000/91/70/09_avatar_middle.jpg) jamessteed 感谢分享！试试看看