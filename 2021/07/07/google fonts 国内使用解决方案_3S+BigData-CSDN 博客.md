> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/yangzhai/article/details/68062891?locationNum=12&fps=1)

由于众所周知的原因，国内使用 google font 库有很大的问题。

解决方案 1：使用国内镜像如 [360 网站卫士常用前端公共库 CDN 服务](http://libs.useso.com)

*   优点：使用方便
    
*   缺点：目标用户包含国外的开发者，不清楚国外用户的加载速度
    

解决方案 2：提供另外一种解决方案，可以自主决定资源下载源，自主配置 cdn 等服务。

1.  在 [google fonts 官网](https://www.google.com/fonts/)上选择字体并获取 css 链接，如下
    

```
<link href='https://fonts.googleapis.com/css?family=Oswald' rel='stylesheet' type='text/css'>

```

1.  将链接内容下载到本地保存，打开，内容如下：
    

```
/* latin */
@font-face {
  font-family: 'Oswald';
  font-style: normal;
  font-weight: 400;
  src: local('Oswald Regular'), 
       local('Oswald-Regular'), 
       url(https://fonts.gstatic.com/s/oswald/v10/pEobIV_lL25TKBpqVI_a2w.woff2) format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02C6, U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2212, U+2215, U+E0FF, U+EFFD, U+F000;
}
/* 如下可能还有更多代码，但结构是和上面的一样的 */
```

1.  将 @font-face 下 src 属性下 url 处的文件下载到本地并保存，并将 url 地址修改成本地地址
    
2.  引用修改后的本地 google fonts css 文件, 就可以使用了。
    

参考资料：

*   [https://www.zhihu.com/question/19578734](https://www.zhihu.com/question/19578734)
    
*   [https://www.google.com/fonts/](https://www.google.com/fonts/)