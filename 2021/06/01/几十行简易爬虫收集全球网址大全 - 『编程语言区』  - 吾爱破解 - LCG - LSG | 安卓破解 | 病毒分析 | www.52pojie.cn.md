> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1450132&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)allenmichael89  比较简单的爬取一些世界网址, 源站点是一个收录站点, 有很多不错的网站, 以下代码会把全站收录的网站保存到 db 数据文件内  
可以下载安装 SQLite Expert Personal 打开 比较方便  
新手学习, 求大佬指点  
有免费评分的, 可以的话, 动动小手给个评分, 谢谢  
[Python] _纯文本查看_ _复制代码_

```
import requestsimport os
from lxml import etree
from fake_useragent import UserAgent
import sqlite3
import time
 
# 设置 请求头
def RquestTools(url):
    headers = {
        'User-Agents': str(UserAgent().random)
    }
    response = requests.get(url, headers=headers).content.decode('gbk')
    html = etree.HTML(response)
    return html
#sqlite3字符处理，因为有些符号影响sqlite命令行，所以需要处理
def str_finishing(seif):
    str_temp = seif
    str = str_temp.replace("/","//")
    str = str.replace("\'","''")
    str = str.replace("[","/[")
    str = str.replace("]","/]")
    str = str.replace("%","/%")
    str = str.replace("_","/_")
    str = str.replace("(","/(")
    str = str.replace(")","/)")
    return str
# 设置请求网址
url = 'http://www.world68.com/country.asp'
# 获取url资源
html = RquestTools(url)
# 定位国家名称
country = html.xpath('//div[@class="content_all r"]/dl/dd/a[1]/text()')
# 定位国家地址
country_url = html.xpath('//div[@class="content_all r"]/dl/dd/a[1]/@href')
 
conn = sqlite3.connect("E:\\worldurl.db")  # 获取或创建数据库链接
litec = conn.cursor()  # 获取游标
 
for i, c in zip(country, country_url):
    sqltable = 'create table ' + i + '(urltype,urlname,urladdress,urlintroduce)'
    #print(str(sqltable))
    litec.execute(str(sqltable))#创建国家表\类型标签
    corntry_html = RquestTools(c)
    cont_r_sort_c = corntry_html.xpath('//div[@class="content_r_sort_c"]/ul/li/a/text()')
    cont_r_sort_c_url = corntry_html.xpath('//div[@class="content_r_sort_c"]/ul/li/a/@href')
    for v, m in zip(cont_r_sort_c, cont_r_sort_c_url):
        tryhtml = RquestTools(m)
        urls = tryhtml.xpath('//dl[@class="top_page"]/dt/a/@href')
        for ii in urls:
            tryhtm2 = RquestTools(ii)
            #网站名称
            country_name = "".join(tryhtm2.xpath('//div[@class="name_r r"]/a/text()'))
            country_name = str_finishing(country_name)
            #网站地址
            country_url = "".join(tryhtm2.xpath('//div[@class="name_r r"]/a/@href'))
            #网站简介
            country_lits = "".join(tryhtm2.xpath('//div[@class="jianjie_r r"]/p/text()'))
            country_lits = str_finishing(country_lits)
            sqladd = 'insert into ' + i + ' values (\'' + v + '\',\'' + country_name + '\',\'' + country_url + '\',\'' + country_lits + '\')'
            print(str(sqladd))
            litec.execute(str(sqladd))#新增 表单数据(网站类型\网站名称\网站Url\网站简介)
        conn.commit()# 提交修改事务
#关闭资料
litec.close()
conn.close()
```

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)allenmichael89 

> [17773441534 发表于 2021-5-30 20:53](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38743842&ptid=1450132)  
> 谢谢分享，对于刚学的人还是有点帮助的

客气了  不嫌气就好 我也是刚学 Python![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)allenmichael89 

> [mandarin 发表于 2021-5-30 21:16](https://www.52pojie.cn/forum.php?mod=redirect&goto=findpost&pid=38744036&ptid=1450132)  
> Java 没学好又想学 Python haha..

别慌 开启多线程学习功能 全都一起学![](https://static.52pojie.cn/static/image/smiley/default/4.gif) ![](https://avatar.52pojie.cn/data/avatar/000/77/33/27_avatar_middle.jpg) wujl82 不错，谢谢分享![](https://static.52pojie.cn/static/image/smiley/default/48.gif) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) 17773441534 谢谢分享，对于刚学的人还是有点帮助的 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) wencun 多谢楼主分享![](https://avatar.52pojie.cn/data/avatar/000/44/89/60_avatar_middle.jpg)无意之过  我还没开始学。。。。小白一个，可以把收集的网站分享上来看看吗？![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)有点小凡  厉害了，刚学就这么厉害，我刚学只会 HELLO World！。。哈哈哈 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) mandarin Java 没学好又想学 Python haha..![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)longling  谢谢楼主的分享