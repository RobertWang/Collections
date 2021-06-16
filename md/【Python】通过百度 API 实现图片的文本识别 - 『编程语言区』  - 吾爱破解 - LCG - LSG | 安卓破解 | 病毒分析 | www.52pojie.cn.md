> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1459827&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)kuruite  _ 本帖最后由 kuruite 于 2021-6-16 13:24 编辑_  
[Python] _纯文本查看_ _复制代码_

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
    <百度云>:<文本识别>
    <通过百度云api识别图片中的文字>
    :copyright: (c) 2021 by debris.
    :license: GPLv3, see LICENSE File for more details.
"""
import base64
import requests
 
'''
access_token 获取
'''
# ak、sk参数配置,百度api上获取API_KEY、SECRET_KEY
API_KEY = 'XXXXXX'
SECRET_KEY = 'XXXXXX'
 
# client_id 为官网获取的AK， client_secret 为官网获取的SK
host = 'https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id=%s&client_secret=%s' % (
    API_KEY, SECRET_KEY)
response = requests.get(host)
access_token = response.json()['access_token']
 
'''
通用文字识别（高精度版）
'''
#
request_url = "https://aip.baidubce.com/rest/2.0/ocr/v1/accurate_basic"
# 二进制方式打开图片文件，填自己目录以及图片名称
f = open('./img/WechatIMG15289.jpeg', 'rb')
 
 
img = base64.b64encode(f.read())
params = {"image": img}
# access_token = '[调用鉴权接口获取的token]'
request_url = request_url + "?access_token=" + access_token
headers = {'content-type': 'application/x-www-form-urlencoded'}
response = requests.post(request_url, data=params, headers=headers).json()
outinfo = response['words_result']
 
#将识别到的文本写入到文件中
fo = open("./info.txt", "w")
for x in outinfo:
    # fo.write(x['words'])
    print(x['words'],file=fo)
fo.close()
```

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)husky9527  感谢分享 可以学习一下 ![](https://avatar.52pojie.cn/data/avatar/001/44/93/85_avatar_middle.jpg) QingYi. 建议放一下成果图 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) santus36 python 有个 ocr 库叫 cnocr，识别效果还是挺好的  
https://github.com/breezedeus/cnocr/![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)xixiaomeng  有开发者文档，直接看文档就行了。