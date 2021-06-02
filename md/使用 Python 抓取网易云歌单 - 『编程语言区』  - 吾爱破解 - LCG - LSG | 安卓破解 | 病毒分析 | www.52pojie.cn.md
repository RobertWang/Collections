> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1451920&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://avatar.52pojie.cn/data/avatar/001/44/93/85_avatar_middle.jpg)QingYi.  其实这个东西昨天就在写了，今天早上才竣工。  
纯练手了  
[Python] _纯文本查看_ _复制代码_

```
import requests
from bs4 import BeautifulSoup
 
for i in range(0, 500, 35):
    # 注意，这里有个陷阱，URL要审查元素去找
    # url = "https://music.163.com/#/discover/playlist/?order=hot&cat=欧美&limit=35&offset={}".format(str(i))
    url = "https://music.163.com/discover/playlist/?cat=欧美&order=hot&limit=35&offset={}".format(str(i))
    # print(url)
 
    head = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:83.0) Gecko/20100101 Firefox/83.0"
    }
    resp = requests.get(url, headers=head)
    # 拿到响应的网页
    html = resp.text
    bs = BeautifulSoup(html, "html.parser")
    # 选择class = dec 下面的a标签
    res = bs.select('.dec a')
    # print(li)
    print(res)
    for j in range(35):
        url = "https://music.163.com/#/" + res[j]['href']
        title = res[j]['title']
        # print(url, title)
        with open("lt.csv", "a+", encoding="utf-8") as f:
            f.write(url + "," + title + "\n")
```

在新标签打开所有链接复制所有链接 URL 复制所有链接 URL（反向）复制所有链接标题 + URL 复制所有链接标题 + URL (MD) 复制所有链接标题 + URL (BBS) 复制所有链接标题 + URL (筛选) 复制所有链接标题 + URL (设置复制格式) 在新标签页打开所有图片链接在一个标签页显示所有图片链接  
复选框 - 选中  
复选框 - 取消  
复选框 - 反选  
单选框 - 选中  
单选框 - 取消  
特殊单选框 - 选中 ![](https://avatar.52pojie.cn/data/avatar/000/23/89/70_avatar_middle.jpg) 806785900 

> [QingYi. 发表于 2021-6-2 11:49](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38779824&ptid=1451920)  
> 学生罢了... 毕业证还没发

会者为师！你做得很棒！![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)fangqiezi  chrome_options = webdriver.ChromeOptions()  
chrome_options.add_argument('headless')  
browser = webdriver.Chrome(chrome_options=chrome_options)  
abc={"http://www."}  
browser.get(abc)   # 要测试的页面  
urls = browser.find_elements_by_xpath("a")   # 匹配出所有 a 元素里的链接  
# 计数器  
mycount=0  
urlss=[]  
for link in browser.find_elements_by_tag_name('a'):  
    u = link.get_attribute('href')  
    if u == 'None':  # 很多的 a 元素没有链接，所有是 None  
        continue  
    else:  
      urlss.append(u)  
      mycount = mycount + 1  
print("打印超链接个数:",mycount)  
print("打印超链接列表",urlss)  
#  保存链接  
f = open('url.txt','w')  
for i in urlss:  
    print(str(i), file=f)  
f.close()  
browser.close()  
请问大哥，我怎么才能一次爬好几个网站的链接呢（abc 是链接）![](https://avatar.52pojie.cn/data/avatar/000/23/89/70_avatar_middle.jpg)806785900  老师真厉害！作品不断！![](https://avatar.52pojie.cn/data/avatar/001/44/93/85_avatar_middle.jpg)QingYi. 

> [806785900 发表于 2021-6-2 11:46](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38779778&ptid=1451920)  
> 老师真厉害！作品不断！

学生罢了... 毕业证还没发 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) C 哥 888 代码好像和抓取视频的有些相似 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) spyer 厉害啊，楼主 ![](https://avatar.52pojie.cn/data/avatar/000/88/74/69_avatar_middle.jpg) blindcat 学习下，感谢分享![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)康铎严  学习了，感谢分享~![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)liuzhen86472796  膜拜大神 1231131![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)KevINBy  受教了，感谢