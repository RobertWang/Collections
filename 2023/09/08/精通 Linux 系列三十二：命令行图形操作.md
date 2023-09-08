> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IPnzgYK9Jcnz7yF9Ik3tSA)

> 精通 Linux 系列，只讲最实用的 linux 知识

图形
==

<table><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;" class="">命令</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;" class="">含义</td></tr></thead><tbody><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">display</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);" class="">显示图形文件。</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">convert</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">转换一个图形格式到另一个格式。</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">mogrify</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">修改图形文件。</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);" class=""><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">montage</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">合并图形文件。</td></tr></tbody></table>

对于查看或编辑图形，Linux 拥有许多有用的工具并且拥有众多选项。我们主要关注来自名为 ImageMagick(http://imagemagick.org) 的软件包中的命令行工具。其命令使用方法相似，完整解释在 http://imagemagick.org/script/command-line-processing.php。

display
-------

stdin  stdout  - 文件  -- 选项  -- 帮助  -- 版本

```
display [选项] 文件

```

`display`命令允许您查看多种格式的图像：JPEG、PNG、GIF、BMP 等。如果您点击显示的图像，它还包括一小套图像编辑工具。输入`q`退出程序。

```
→ display photo.jpg

```

该命令非常强大，其 manpage 列出了 100 多个选项。

#### 有用的选项

<table><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">-resize *</code>大小<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">*</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;" class="">调整图像大小。*<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">大小</code>* 的值非常灵活，包括设置宽度 (<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">800</code>)、高度 (<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">x600</code>)、两者都有 (<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">800x600</code>)、增长或缩小的百分比 (<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">50%</code>)、以像素为单位的面积 (<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">480000@</code>) 等。</td></tr></thead><tbody><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">-flip</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);" class="">垂直翻转图像。</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">-flop</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);" class="">水平翻转图像。</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">-rotate *</code>N<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">*</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">旋转图像&nbsp;<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">N </code>&nbsp;度。</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">-backdrop</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">在覆盖屏幕其余部分的纯色背景上显示图像。</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">-fill</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);" class="">设置由&nbsp;<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">-backdrop</code>&nbsp;选项使用的纯色。</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">-delay *</code>N<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">*</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">显示图像&nbsp;<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">N </code>&nbsp;秒然后退出。如果您列出多个图像，您将得到一个幻灯片，每个图像之间有&nbsp;<code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">N</code>&nbsp;秒的延迟。</td></tr><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);"><code data-style="white-space: pre-wrap; line-height: 1.75; font-size: 12.6px; color: rgb(221, 17, 68); background: rgba(27, 31, 35, 0.05); padding: 3px 5px; border-radius: 4px;">-identify</code></td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">打印关于图像的格式、大小和其他统计信息到标准输出。</td></tr></tbody></table>

convert
-------

stdin  stdout  - 文件  -- 选项  -- 帮助  -- 版本

```
convert [输入选项] 输入文件 [输出选项] 输出文件

```

`convert`命令复制一个图像但转换为不同的图形格式。例如，如果您有一个 JPEG 文件，您可以生成同一图像的 PNG 文件：

```
→ convert photo.jpg newphoto.png

```

同时，您可以在复制中执行修改，如调整大小或翻转：

```
→ convert photo.jpg -resize 50% -flip newphoto.png

```

`convert`接受的选项与`display`大致相同。

mogrify
-------

stdin  stdout  - 文件  -- 选项  -- 帮助  -- 版本

```
mogrify [选项] 文件

```

`mogrify`命令就像`convert`那样转换图像，但是更改直接应用于您提供的图像文件，而不是在副本中。（因此，当在喜欢的照片上实验时，`convert`是一个更安全的命令。）它接受的选项与`convert`大致相同：

```
→ mogrify -resize 25% photo.jpg

```

montage
-------

stdin  stdout  - 文件  -- 选项  -- 帮助  -- 版本

```
montage 输入文件 [选项] 输出文件

```

`montage`从一系列输入文件产生一个单一的图像文件。例如，您可以在单一图像中创建一系列缩略图，每个缩略图都带有其原始文件名：

```
→ montage photo.jpg photo2.png photo3.gif \
  -geometry 120x176+10+10 -label '%f' outfile.jpg

```

`montage`提供了对图像出现方式的高度控制。例如，前述命令生成大小为 120x176 像素的缩略图，水平和垂直偏移 10 像素（在缩略图之间创建空间），并用其输入文件名标记。

#### 有用的选项

<table class=""><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;" class="">-geometry&nbsp;宽度 x 高度 [+-]x[+-]y</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em;">设置图像的高度、宽度和 *(x,y)* 偏移。</td></tr></thead><tbody><tr><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);" class="">-frame&nbsp;N</td><td data-style="line-height: 1.75; border-color: rgb(223, 223, 223); padding: 0.25em 0.5em; color: rgb(63, 63, 63);">在每张图片周围绘制一个宽度为 * N 像素的边框。</td></tr></tbody></table>