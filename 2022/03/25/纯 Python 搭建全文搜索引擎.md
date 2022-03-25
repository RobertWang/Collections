> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247513212&idx=2&sn=95066c4f8e00eabcff2e79181f9f4959&chksm=e969ea71de1e636751fc6fb71373ac6a1f5efafce0d86a6d82523d256e384b866e99ce21992b&scene=21#wechat_redirect)

有一个群友在群里问个如何快速搭建一个搜索引擎，在搜索之后我看到了这个

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/iaahNUMjJc7u1U6QBrenrODusiahZJYsKqdHxnkibXs2dMJgrwoFVr4yuK2rRvibcxSiatRZYBSN2nsGWghAzpafIqg/640?wx_fmt=png)

# 代码所在
------

*   Git:https://github.com/asciimoo/searx
    

官方很贴心，很方便的是已经提供了 docker 镜像，基本 pull 下来就可以很方便的使用了，执行命令

```
cid=$(sudo docker ps -a | grep searx | awk '{print $1}')
echo searx  cid is $cid
if [ "$cid" != "" ];then
    sudo docker stop $cid
    sudo docker rm $cid
fi
sudo docker run -d --name searx -e IMAGE_PROXY=True -e BASE_URL=http://yourdomain.com  -p 7777:8888 wonderfall/searx


```

然后就可以使用了, 正常查看 docker 的状态，就可以正常的使用了

# 思考
----

怎么样，是不是很方便，我们先看看源码是怎么样实现的

![图片](https://mmbiz.qpic.cn/mmbiz_png/iaahNUMjJc7u1U6QBrenrODusiahZJYsKq8R4icKXrGfe4sE55ianBhu78x2PWbibIcQOfncpaBTFS4E6SUrZwfQW9Q/640?wx_fmt=png)

我们打开里面的代码，其实本质就是将 request 之后的结果做一个大的聚合，至于数据来源，我们可以是来于 DB, 或者文件，我们可以看一下他的核心代码。另外，搜索公众号 Linux 就该这样学后台回复 “git 书籍”，获取一份惊喜礼包。

```
from urllib import urlencode
from json import loads
from collections import Iterable

search_url = None
url_query = None
content_query = None
title_query = None
suggestion_query = ''
results_query = ''

# parameters for engines with paging support
#
# number of results on each page
# (only needed if the site requires not a page number, but an offset)
page_size = 1
# number of the first page (usually 0 or 1)
first_page_num = 1

def iterate(iterable):
    if type(iterable) == dict:
        it = iterable.iteritems()

    else:
        it = enumerate(iterable)
    for index, value in it:
        yield str(index), value

def is_iterable(obj):
    if type(obj) == str:
        return False
    if type(obj) == unicode:
        return False
    return isinstance(obj, Iterable)

def parse(query):
    q = []
    for part in query.split('/'):
        if part == '':
            continue
        else:
            q.append(part)
    return q

def do_query(data, q):
    ret = []
    if not q:
        return ret

    qkey = q[0]

    for key, value in iterate(data):

        if len(q) == 1:
            if key == qkey:
                ret.append(value)
            elif is_iterable(value):
                ret.extend(do_query(value, q))
        else:
            if not is_iterable(value):
                continue
            if key == qkey:
                ret.extend(do_query(value, q[1:]))
            else:
                ret.extend(do_query(value, q))
    return ret

def query(data, query_string):
    q = parse(query_string)

    return do_query(data, q)

def request(query, params):
    query = urlencode({'q': query})[2:]

    fp = {'query': query}
    if paging and search_url.find('{pageno}') >= 0:
        fp['pageno'] = (params['pageno'] - 1) * page_size + first_page_num

    params['url'] = search_url.format(**fp)
    params['query'] = query

    return params

def response(resp):
    results = []
    json = loads(resp.text)
    if results_query:
        for result in query(json, results_query)[0]:
            url = query(result, url_query)[0]
            title = query(result, title_query)[0]
            content = query(result, content_query)[0]
            results.append({'url': url, 'title': title, 'content': content})
    else:
        for url, title, content in zip(
            query(json, url_query),
            query(json, title_query),
            query(json, content_query)
        ):
            results.append({'url': url, 'title': title, 'content': content})

    if not suggestion_query:
        return results
    for suggestion in query(json, suggestion_query):
        results.append({'suggestion': suggestion})
    return results


```

# 结果
----

每个 response 的时候我们都要以轻松的定制返回的数据（可以是网络，可以是数据库，可以是文件），那我们进一步想一下，如果我们可以 hack response 结果，那我们完全可以将自己爬来的数据做为返回结果。如果是 1024 之类的，完全可以打造自己的 “爱好” 小引擎，代码我就不贴了，大家可以自己动手自己玩玩。结合 jieba 分词，可以更好玩一点。

****你还有什****么想要补充的吗？****  

> 免责声明：本文内容来源于网络，文章版权归原作者所有，意在传播相关技术知识 & 行业趋势，供大家学习交流，若涉及作品版权问题，请联系删除或授权事宜。