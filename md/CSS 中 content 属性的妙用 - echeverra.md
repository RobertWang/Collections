> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [echeverra.cn](https://echeverra.cn/2021/08/06/css-content/)

> 前言 本文讲解 CSS 中使用频率并不高的 content 属性，通过多个实用的案例，带你由浅入深的掌握 content […]

本文讲解 CSS 中使用频率并不高的 content 属性，通过多个实用的案例，带你由浅入深的掌握 content 的用法，让代码变得更加简洁、高效。

W3school 中这样定义：

> content 属性与 :before 及 :after 伪元素配合使用，来插入生成内容。
> 
> 该属性用于定义元素之前或之后放置的生成内容。默认地，这往往是行内内容，不过该内容创建的框类型可以用属性 display 控制。

在前端日常开发中，content 属性使用频率并不高，所以开发者一般对它的理解并不深入，通常会在清除浮动、字体图标时偶尔使用。下面通过各种案例，来一起看看 content 的妙用。

```
<!--css-->
	.left {float: left}
	.right {float: right}
	.clear:after {
	    content: '';
	    clear: both;
	    display: block;
	}
	 
	<!--html-->
	<div class="container clear">
	    <div class="left">左</div>
	    <div class="right">右</div>
	</div>
```

父元素`.container`中两个子元素`.left`和`.right`浮动后会脱离文档流，无法撑起容器，造成`.container`高度为`0`。使用伪元素`:after`清除浮动，三个属性缺一不可：

*   `content: '';` 通过`:after`添加一个内容为空的伪元素。
*   `clear: both;` 伪元素`:after`两侧浮动清除。
*   `dispaly: block;` 设置块元素，因为`clear`只对块元素有效。

说到`clear`属性，使用最多的就是`clear: both`，`left/right`用的却很少，因为`both`已经包含`left/right`特性，简单直接还有效。想更加深入了解`clear`属性，可以看看张鑫旭大神的[准确理解 CSS clear:left/right 的含义及实际用途](http://www.zhangxinxu.com/wordpress/?p=4179)。

```
<!--css-->
	.box {
	    width: 200px;
	    height: 100px;
	    border-radius: 5px;
	    background: #fff;
	    position: relative;
	}
	.box:after {
	    content: '';
	    position: absolute;
	    bottom: -20px;
	    right: 20px;
	    width: 0;
	    height: 0;
	    border-top: 10px solid #fff;
	    border-bottom: 10px solid transparent;
	    border-left: 10px solid transparent;
	    border-right: 10px solid transparent;
	}
	 
	<!--html-->
	<div class="box"></div>
```

效果：

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAASsAAAC2CAIAAAAOSTzKAAAGbUlEQVR4Ae3ZzW4TSRiF4fEIySwiR6Pc/6XlEiwUyxuvTE+QIpD58YETVVfzZMWYqq/aj+sFz8zu+fn5Hz8ECAwS+HfQuY4lQOB/AQW6BwRGCihwpL6zCSjQHSAwUkCBI/WdTUCB7gCBkQIKHKnvbAIKdAcIjBRQ4Eh9ZxNQoDtAYKSAAkfqO5uAAt0BAiMFFDhS39kEFOgOEBgpoMCR+s4moEB3gMBIAQWO1Hc2AQW6AwRGCihwpL6zCSjQHSAwUkCBI/WdTUCB7gCBkQIKHKnvbAIKdAcIjBRQ4Eh9ZxNQoDtAYKSAAkfqO5uAAt0BAiMFFDhS39kEFOgOEBgpoMCR+s4moEB3gMBIgQ8jD//27IeHh8PhsN/vd7vdt7/jnwjcJXC9Xi+Xy+l0Op/Pd21YwaK1FPj09PT4+LgCEI8wscDyZ/fH15/lz/Hj8TjFO1nFt9Dlbz/5TXFdZnnI5Totl2qKp11FgcuXzymwPOREArNcqlUUuHxnmOij9ahTCMxyqVZRoP/0MsWdnushZ7lUqyhwro/W0xIoCiiwiGkUgVhAgTGZDQSKAgosYhpFIBZQYExmA4GigAKLmEYRiAUUGJPZQKAooMAiplEEYgEFxmQ2ECgKKLCIaRSBWECBMZkNBIoCCixiGkUgFlBgTGYDgaKAAouYRhGIBRQYk9lAoCigwCKmUQRiAQXGZDYQKAoosIhpFIFYQIExmQ0EigIKLGIaRSAWUGBMZgOBooACi5hGEYgFFBiT2UCgKKDAIqZRBGIBBcZkNhAoCiiwiGkUgVhAgTGZDQSKAgosYhpFIBZQYExmA4GigAKLmEYRiAUUGJPZQKAooMAiplEEYgEFxmQ2ECgKKLCIaRSBWECBMZkNBIoCCixiGkUgFlBgTGYDgaKAAouYRhGIBRQYk9lAoCigwCKmUQRiAQXGZDYQKAoosIhpFIFYQIExmQ0EigIKLGIaRSAWUGBMZgOBooACi5hGEYgFFBiT2UCgKKDAIqZRBGIBBcZkNhAoCiiwiGkUgVhAgTGZDQSKAgosYhpFIBZQYExmA4GigAKLmEYRiAUUGJPZQKAooMAiplEEYgEFxmQ2ECgKKLCIaRSBWECBMZkNBIoCCixiGkUgFlBgTGYDgaKAAouYRhGIBRQYk9lAoCigwCKmUQRiAQXGZDYQKAoosIhpFIFYQIExmQ0EigIKLGIaRSAWUGBMZgOBooACi5hGEYgFFBiT2UCgKKDAIqZRBGIBBcZkNhAoCiiwiGkUgVhAgTGZDQSKAgosYhpFIBZQYExmA4GigAKLmEYRiAUUGJPZQKAooMAiplEEYgEFxmQ2ECgKKLCIaRSBWECBMZkNBIoCCixiGkUgFlBgTGYDgaKAAouYRhGIBRQYk9lAoCigwCKmUQRiAQXGZDYQKAqsosDr9Vp8S0YRWARmuVSrKPByubg0BLoCs1yqVRR4Op26+qYRmOVSraLA8/n88vLi0hBoCSzXablUrWnvOufDu06/f/jxeFy+NhwOh/1+v9vt7t9oJYE3geXf/ZZbtPztN0t+y5OvpcDlURa1ieDePnW/+KXAf68/v1x2u+DT68/t61t6ZRXfQrcE6r3cCvxeSL+36/b0lb+iwJV/QBt5vDSndP28TAqc97Ob7Mnvj+r+lZMRfO9xFfg9Fa+9j8A9ad2z5n2ebsxUBY5x/2tP/XlgP//dTaIpcJMf66rf1I8y+9Hrq34zf/xwCvxjQgNygdvYbl/Jp065Q4FTfmwbeOivk/v61xt4a9FbWNH/kY+e2+INCCzhfXkXb7/YwJtK34ICUzHrmwJ/c3tfHH0Lbd4nswikAgpMxawn0BRQYFPTLAKpgAJTMesJNAUU2NQ0i0AqoMBUzHoCTQEFNjXNIpAKKDAVs55AU0CBTU2zCKQCCkzFrCfQFFBgU9MsAqmAAlMx6wk0BRTY1DSLQCqgwFTMegJNAQU2Nc0ikAooMBWznkBTQIFNTbMIpAIKTMWsJ9AUUGBT0ywCqYACUzHrCTQFFNjUNItAKqDAVMx6Ak0BBTY1zSKQCigwFbOeQFNAgU1NswikAgpMxawn0BRQYFPTLAKpgAJTMesJNAUU2NQ0i0AqoMBUzHoCTQEFNjXNIpAKKDAVs55AU0CBTU2zCKQCCkzFrCfQFFBgU9MsAqmAAlMx6wk0BRTY1DSLQCqgwFTMegJNgc+o17DRqoyuZwAAAABJRU5ErkJggg==)

配合伪元素`:after`，实现了一个右下角带倒三角指向性的气泡窗口。通过调整`border`各边的颜色和绝对定位位置，可以绘制出指向不同的气泡窗口，只用一个`div`标签实现，是不是既简洁又美观。可能你会想到这是伪元素`:after`的效果，和`content`属性无关，实际上去掉`content`后`:after`是不生效的。

![](data:image/png;base64,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)

第一种是浏览器自带的特殊字符：

```
<!--css-->
	.music:before {
	    content: '\266C';
	    color: red;
	}
	 
	<!--html-->
	<span class="music">晴天-周杰伦</span>
```

效果：

![](data:image/png;base64,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)

浏览器支持很多字体图标，如：天气![](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAzNiAzNiI+PHBhdGggZmlsbD0iI0ZGQUMzMyIgZD0iTTE2IDJzMC0yIDItMiAyIDIgMiAydjJzMCAyLTIgMi0yLTItMi0yVjJ6bTE4IDE0czIgMCAyIDItMiAyLTIgMmgtMnMtMiAwLTItMiAyLTIgMi0yaDJ6TTQgMTZzMiAwIDIgMi0yIDItMiAySDJzLTIgMC0yLTIgMi0yIDItMmgyem01LjEyMS04LjcwN3MxLjQxNCAxLjQxNCAwIDIuODI4LTIuODI4IDAtMi44MjggMEw0Ljg3OCA4LjcwOHMtMS40MTQtMS40MTQgMC0yLjgyOWMxLjQxNS0xLjQxNCAyLjgyOSAwIDIuODI5IDBsMS40MTQgMS40MTR6bTIxIDIxczEuNDE0IDEuNDE0IDAgMi44MjgtMi44MjggMC0yLjgyOCAwbC0xLjQxNC0xLjQxNHMtMS40MTQtMS40MTQgMC0yLjgyOCAyLjgyOCAwIDIuODI4IDBsMS40MTQgMS40MTR6bS0uNDEzLTE4LjE3MnMtMS40MTQgMS40MTQtMi44MjggMCAwLTIuODI4IDAtMi44MjhsMS40MTQtMS40MTRzMS40MTQtMS40MTQgMi44MjggMCAwIDIuODI4IDAgMi44MjhsLTEuNDE0IDEuNDE0em0tMjEgMjFzLTEuNDE0IDEuNDE0LTIuODI4IDAgMC0yLjgyOCAwLTIuODI4bDEuNDE0LTEuNDE0czEuNDE0LTEuNDE0IDIuODI4IDAgMCAyLjgyOCAwIDIuODI4bC0xLjQxNCAxLjQxNHpNMTYgMzJzMC0yIDItMiAyIDIgMiAydjJzMCAyLTIgMi0yLTItMi0ydi0yeiIvPjxjaXJjbGUgZmlsbD0iI0ZGQUMzMyIgY3g9IjE4IiBjeT0iMTgiIHI9IjEwIi8+PC9zdmc+)、雪花![](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAzNiAzNiI+PHBhdGggZmlsbD0iIzg4QzlGOSIgZD0iTTE5IDI3LjU4NlY4LjQxNWw0LjgyOC00LjgyOXMuNzA3LS43MDcgMC0xLjQxNWMtLjcwNy0uNzA3LTEuNDE0IDAtMS40MTQgMEwxOSA1LjU4NlYxczAtMS0xLTEtMSAxLTEgMXY0LjU4NmwtMy40MTQtMy40MTVzLS43MDctLjcwNy0xLjQxNCAwYy0uNzA3LjcwOCAwIDEuNDE1IDAgMS40MTVMMTcgOC40MTV2MTkuMTcxbC00LjgyOCA0LjgyOHMtLjcwNy43MDcgMCAxLjQxNCAxLjQxNCAwIDEuNDE0IDBMMTcgMzAuNDE0VjM1czAgMSAxIDEgMS0xIDEtMXYtNC41ODZsMy40MTQgMy40MTRzLjcwNy43MDcgMS40MTQgMCAwLTEuNDE0IDAtMS40MTRMMTkgMjcuNTg2eiIvPjxwYXRoIGZpbGw9IiM4OEM5RjkiIGQ9Ik0zNC42MjIgMjAuODY2Yy0uMjU5LS45NjYtMS4yMjUtLjcwNy0xLjIyNS0uNzA3bC02LjU5NSAxLjc2Ny0xNi42MDMtOS41ODYtMS43NjctNi41OTVzLS4yNTktLjk2Ni0xLjIyNS0uNzA3QzYuMjQgNS4yOTcgNi41IDYuMjYzIDYuNSA2LjI2M2wxLjI1IDQuNjY0LTMuOTcyLTIuMjk0cy0uODY2LS41LTEuMzY2LjM2NmMtLjUuODY2LjM2NiAxLjM2Ni4zNjYgMS4zNjZsMy45NzEgMi4yOTMtNC42NjQgMS4yNDlzLS45NjcuMjU5LS43MDcgMS4yMjVjLjI1OS45NjcgMS4yMjUuNzA4IDEuMjI1LjcwOGw2LjU5Ni0xLjc2NyAxNi42MDMgOS41ODYgMS43NjcgNi41OTZzLjI1OS45NjYgMS4yMjUuNzA3Yy45NjYtLjI2LjcwNy0xLjIyNS43MDctMS4yMjVsLTEuMjUtNC42NjQgMy45NzIgMi4yOTNzLjg2Ny41IDEuMzY3LS4zNjVjLjUtLjg2Ny0uMzY3LTEuMzY3LS4zNjctMS4zNjdsLTMuOTcxLTIuMjkzIDQuNjYzLTEuMjQ5YzAtLjAwMS45NjYtLjI2LjcwNy0xLjIyNnoiLz48cGF0aCBmaWxsPSIjODhDOUY5IiBkPSJNMzMuOTE1IDEzLjkwN2wtNC42NjQtMS4yNSAzLjk3Mi0yLjI5M3MuODY3LS41MDEuMzY3LTEuMzY3Yy0uNTAxLS44NjctMS4zNjctLjM2Ni0xLjM2Ny0uMzY2bC0zLjk3MSAyLjI5MiAxLjI0OS00LjY2M3MuMjU5LS45NjYtLjcwNy0xLjIyNWMtLjk2Ni0uMjU5LTEuMjI1LjcwNy0xLjIyNS43MDdsLTEuNzY3IDYuNTk1LTE2LjYwNCA5LjU4OS02LjU5NC0xLjc2OHMtLjk2Ni0uMjU5LTEuMjI1LjcwN2MtLjI2Ljk2Ny43MDcgMS4yMjUuNzA3IDEuMjI1bDQuNjYzIDEuMjQ5LTMuOTcxIDIuMjkzcy0uODY1LjUwMS0uMzY1IDEuMzY3Yy41Ljg2NSAxLjM2NS4zNjUgMS4zNjUuMzY1bDMuOTcyLTIuMjkzLTEuMjUgNC42NjNzLS4yNTkuOTY3LjcwNyAxLjIyNWMuOTY3LjI2IDEuMjI2LS43MDYgMS4yMjYtLjcwNmwxLjc2OC02LjU5NyAxNi42MDQtOS41ODUgNi41OTUgMS43NjhzLjk2Ni4yNTkgMS4yMjUtLjcwN2MuMjU1LS45NjctLjcxLTEuMjI1LS43MS0xLjIyNXoiLz48L3N2Zz4=)、三叶草![](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAzNiAzNiI+PHBhdGggZmlsbD0iIzc3QjI1NSIgZD0iTTMyLjM0IDE5LjIwOGMuMTk4LTEuNDc1IDIuNjY5LTIuMjA4IDIuNjY5LTQuNTQyQzM1LjAwOSAxMi4zMzMgMjkuMjUgOC4yNSAyMSAxNmMzLjgxMi01LjMxMiA2LjU0Mi0xNC41MDkgMi4yMDMtMTQuNTA5LTEuNDcgMC0yLjU3OSAxLjM0Mi00LjMyOCAxLjM0MkMxNi41IDIuODMzIDE2LjIwOCAwIDE0IDBjLTIuNTgzIDAtNSA2LjI1IDEgMTUtMS41LTEuNjI1LTExLjA4My02LjYyNS0xMy4wMDItMS4yNjktLjYyOCAxLjc1MyAxLjczMSAyLjc4MSAxLjU0NCA0LjUxOS0uMTc4IDEuNjU0LTIuNTk3IDIuMDM3LTIuNTk3IDQuMzcyIDAgMy4yOTMgOC4wNDggNC44MzQgMTYuMDQ0LTEuNjEzLS4yNDUgNC41MjQuNTU3IDEwLjUxNSAzLjg1NyAxNC4xNTggMS4zNDQgMS40ODMgMi40MDcuNTUxIDIuODIyLjE4Ny40MTYtLjM2NSAxLjYwNC0xLjQxNC4xODUtMi44MjItMy44MzQtMy44MDctNS4yMjUtOC4xNzQtNS42MTktMTEuMzgxIDIuMDQzIDIuMjE1IDEyLjM1NyA4Ljc0MiAxNS41MTcgMi43NjcuODgxLTEuNjY4LTEuNjQzLTIuOTc3LTEuNDExLTQuNzF6Ii8+PC9zdmc+)、太极![](https://s.w.org/images/core/emoji/13.1.0/svg/262f.svg)等等有趣的字符。参考[网页 HTML 特殊字符编码对照表](https://www.22vd.com/33993.html)。

第二种是外部引入字体图标，如 Bootstrap 中的 Glyphicon 字体图标：

```
<link rel="stylesheet" href="../static/css/bootstrap.min.css">
	 
	<!--html-->
	<span class="glyphicon glyphicon-heart"></span>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210731101622.png)

这里为什么没有写 CSS 样式呢，因为 bootstrap.min.css 中已经定义好了 glyphicon-heart 的样式，直接在[官网](https://v3.bootcss.com/components/#glyphicons)上查看：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210731102030.png)

需要说明的是，本地引入 bootstrap.min.css 后，还需要引入字体图标库 glyphicons-halflings-regular.woff2，字体图标才能生效。

```
<!--bootstrap.min.css-->
	 
	<!--format 属性是帮助浏览器识别字体的-->
	@font-face {
	    font-family: 'Glyphicons Halflings';
	    src: url(../fonts/glyphicons-halflings-regular.eot);
	    src: url(../fonts/glyphicons-halflings-regular.eot?#iefix) format('embedded-opentype'),
	    url(../fonts/glyphicons-halflings-regular.woff2) format('woff2'),
	    url(../fonts/glyphicons-halflings-regular.woff) format('woff'),
	    url(../fonts/glyphicons-halflings-regular.ttf) format('truetype'),
	    url(../fonts/glyphicons-halflings-regular.svg#glyphicons_halflingsregular) format('svg')
	}
```

按照上述 url 路径，将 glyphicons-halflings-regular.woff2 放到如下目录结构中即可。

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210731102356.png)

```
<!--css-->
	<!--:empty为空时匹配-->
	div:empty:after {
	    content: '暂无数据';
	    color: red;
	}
	 
	<!--html-->
	<div>有内容数据</div>
	<div></div>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210731111914.png)

当元素内容为空时，通过`content`内容 “暂无数据” 进行展示。可使用场景：后台接口返回数据后，插入到页面 DOM 中，当后台返回数据为空时，通过 CSS 直接提示暂无数据，不需要使用 JavaScript 处理。

一个有趣的现象，`content`内容 “暂无数据” 无法被选中，这是因为伪元素用于创建一些不在文档树中的元素，虽然用户可以看到这些文本，但是这些文本实际上不在文档树中。

```
<!--css-->
	ul li {
	    display: inline-block;
	    font-weight: bold;
	}
	 
	ul li:not(:last-child):after {
	    content: '\276D';
	    margin: 5px;
	}
	 
	<!--html-->
	<ul>
	    <li>首页</li>
	    <li>商品</li>
	    <li>详情</li>
	</ul>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210731152716.png)

又是一个`content`值为特殊字符的例子，配合伪类和伪元素完成面包屑菜单。

```
<!--css-->
	.loading:after {
	    content: ".";
	    animation: loading 2s ease infinite;
	}
	 
	@keyframes loading {
	    33% {
	        content: "..";
	    }
	    66% {
	        content: "...";
	    }
	}
	 
	<!--html-->
	<p class="loading">加载中 </p>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210731172044.gif)

`animation`动画值含义：

*   loading：animation-name，规定需要绑定到选择器的 keyframe 名称为 loading。
*   2s：animation-duration，规定完成动画所花费的时间 2 秒。
*   ease：animation-timing-function，规定动画的速度曲线，先慢中快后慢。
*   infinite：animation-iteration，规定动画应该播放的次数为无限次。

通过`animation`动画完成 33%、66% 时与`content`内容配合，实现动态 “加载中...” 的效果。

```
<!--css-->
	.loading:before {
	    content: url("../static/img/loading.gif");
	    vertical-align: middle;
	}
	 
	<!--html-->
	<div class="loading"> 加载中... </div>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210731164425.png)

除了插入字体图标，`content`还可以使用`url()`方法插入图片，和`background`属性类似可以使用`url`指定一个图片路径，不同的是`content`属性无法控制图片大小，使用条件有限。

```
<!--css-->
	.web:after {
	    content: "（"attr(href)"）"
	}
	 
	<!--html-->
	<a class="web" href="https://echeverra.cn">echevera</a>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210806120233.png)

`content`值也可以是`attr()`方法，用来获取指定属性的值，可插入到指定的位置。

```
<!--css-->
	span{
	    font-family: sans-serif;
	    font-size: 30px;
	    font-weight: bold;
	    position: relative;
	    color: green;
	}
	span:before{
	    content: attr(text);
	    color: orange;
	    position: absolute;
	    left: 0;
	    top: 0;
	    width: 50%;
	    overflow: hidden;
	}
	 
	<!--html-->
	<span text="echeverra">echeverra</span>
	<span text="博">博</span>
	<span text="客">客</span>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210803114204.png)

半边特效是通过`attr()`方法获取`text`属性值，属性值与其内容相同，巧妙的设置绝对定位，只显示一半并覆盖了原文本内容，实现文字半边特效，是不是还挺炫酷的。需要注意的是有些`font-family`字体会有文字无法重合的问题。

```
<!--css-->
	span {
	    quotes: '“' '”';
	}
	span:before {
	    content: open-quote;
	}
	span:after {
	    content: close-quote;
	}
	 
	<!--html-->
	<p>鲁迅说过：<span>真正的勇士,敢于直面惨淡的人生,敢于正视淋漓的鲜血。</span></p>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210804200817.png)

利用元素的`quotes`属性指定双引号，使用`content`的`open-quote`属性值设置开口引号，`close-quote`属性值设置闭合引号，当然`quotes`也可以指定其他符号。

```
<!--css-->
	ul{
	    counter-reset: section;
	}
	li{
	    list-style-type: none;
	    counter-increment: section;
	}
	li:before{
	    content: counters(section, '-') '.';
	}
	 
	<!--html-->
	<ul>
	    <li>章节一</li>
	    <li>章节二
	        <ul>
	            <li>章节二一</li>
	            <li>章节二二</li>
	            <li>章节二三</li>
	        </ul>
	    </li>
	    <li>章节三</li>
	    <li>章节四</li>
	    <li>章节五
	        <ul>
	            <li>章节五一</li>
	            <li>章节五二</li>
	        </ul>
	    </li>
	    <li>章节六</li>
	</ul>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210806094636.png)

这里用到了`counter`计数器方法，涉及到`counter-reset`、`counter-increment`、`counter()`和`counters()`几个属性。

*   `counter-reset`用来指定计数器名称，例子中命名为`section`，同时可以指定计数器开始计数的数值，如指定开始计数数值为 1：`counter-reset: section 1;`，不指定默认为`0`。
    
*   `counter-increment`用来指定计数器每次递增的值，如指定计数器递增值为 2：`counter-increment: section 2;`，默认值为 1，`counter-increment`只要指定了`counter-reset`，对应的计数器的值就会变化。
    
*   `counter(name, style)`用来输出计数器的值。其中`name`为计数器名称，`style`可选参数，默认为阿拉伯数字，也可指定`list-style-type`支持的值，如罗马数字`lower-roman`。
    
*   `counters(name, strings, style)`用来处理嵌套计数器，同样是输出计数器的值，和`counter()`不同的是多了一个`strings`参数，表示子序号的连接字符串。例如`1.1`的`string`就是`'.'`，`1-1`就是`'-'`。
    

关于计数器的详细的教程，同样可以参考 CSS 大神张鑫旭的 [CSS counter 计数器 (content 目录序号自动递增) 详解](http://www.zhangxinxu.com/wordpress/?p=4303)。

```
<!--css-->
	form {
	    counter-reset: count 0;
	}
	 
	input[type="checkbox"]:checked {
	    counter-increment: count 1;
	}
	 
	.result:after {
	    content: counter(count);
	}
	 
	<!--html-->
	<form>
	    <input type="checkbox" id="javaScript">
	    <label for="javaScript">javaScript</label>
	    <input type="checkbox" id="PHP">
	    <label for="PHP">PHP</label>
	    <input type="checkbox" id="Python">
	    <label for="Python">Python</label>
	 
	    <div class="result">已选：</div>
	</form>
```

效果：

![](https://cdn.jsdelivr.net/gh/echeverra/img@main/resource/20210806105757.gif)

巧妙运用计数器配合`:checked`伪类，选中计数器增加 1，取消选中减 1，用 CSS 实现动态计数功能。

总结 content 属性值可以为以下几种：

*   string 字符串，最常见的，对应案例：清除浮动、小三角的气泡窗口、字体图标、无内容提示、面包屑菜单、加载中... 动画。
*   url() 方法，对应案例：插入图片。
*   attr() 方法，对应案例：attr 属性内容生成、半边特效。
*   quotes 引号，对应案例：文字引号。
*   counter() 方法，计数器功能，对应案例：添加章节数，计算 checkbox 选中数。

尽管使用 javaScript 同样可以实现上述的大部分功能，灵活性也更高，但使用 CSS 的好处就是可以极大地简化开发，不占用 JS 主线程，提升 web 的性能。

其实 content 的案例远不止于此，在查阅相关资料的同时，我还见识到了另外一些神奇的用法，当然原理大致相同，本文的案例只是尽可能的带你了解 content 不为人知的一面，打开一个全新的世界，让你举一反三。如果能在实际开发中运用上，那就更 Nice 了，希望能对大家有所帮助。

你学 “废” 了么？

参考文章：

*   [CSS 进阶之熟悉又陌生的 content](https://juejin.cn/post/6989017411261300750)
*   [你所知道或不知道的 CSS content 属性](https://juejin.cn/post/6844903705712525319)
*   [如何把 css'content 的操作跟价值发挥到最大](https://juejin.cn/post/6844903917348732936)
*   [CSS counter 计数器 (content 目录序号自动递增) 详解](https://www.zhangxinxu.com/wordpress/2014/08/css-counters-automatic-number-content/)

(完)