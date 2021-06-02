> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1452056&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)mdl2999_52pj 

```
# include <iostream>
# include <iomanip>
# include <array>
//c++17
using namespace std;

int main()
{
    // 习题1
    auto values{ array<int,50>{} };
    size_t count = size(values);
    for(auto p = &values[0]; p!= &values[0] + count; ++p)
    {
        *p = 2*(p-&values[0]) +1;
    }

    for(size_t i{}; i!= size(values);++i)
    {
        cout << setw(4) << values[i] ;
        if ((i+1)% 10 ==0) cout << endl;
    }
    cout << endl;
    size_t i{};
    auto p = &values[0]+count;
    do{

        cout << setw(4) << *--p;
        if ((i+++1)%10==0) cout << endl;
    }
    while(p != &values[0]);

}
```

  
![](https://attach.52pojie.cn/forum/202106/02/144219ag6vd3nkoo6eon66.png)

**001.png** _(20.32 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjI5ODgxOXwxOWFmODE1MHwxNjIyNjQyNTA4fDEwNzQxNTd8MTQ1MjA1Ng%3D%3D&nothumb=yes)

2021-6-2 14:42 上传

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)kftjsdy  楼主坚持![](https://static.52pojie.cn/static/image/smiley/default/42.gif)![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)皮蛋瘦肉  楼主加油 ![](https://avatar.52pojie.cn/data/avatar/000/80/22/79_avatar_middle.jpg) tlf 支持楼主！![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)漩涡鸣人 12138  楼主加油 ![](https://avatar.52pojie.cn/data/avatar/000/44/62/88_avatar_middle.jpg) wax126 第 13 天了，不错哦。加油