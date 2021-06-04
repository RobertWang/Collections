> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Marksinoberg/article/details/52116088?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_v2~rank_aggregation-18-52116088.pc_agg_rank_aggregation&utm_term=linux%E7%BB%88%E7%AB%AF%E6%98%BE%E7%A4%BA%E5%9B%BE%E7%89%87&spm=1000.2123.3001.4430)

Linux 终端里面可谓是奇妙无限，很多优秀的软件都诞生在终端里面。相较之下，Windows 本身的理念和 Linux 就不一致，所以，你懂得。  
下面，我们不妨先思考一下，如何在终端里面显示一张图片？

在终端里面显示，肯定就不像在看图软件里那样的细腻了，我们只是以字符代替某一点的像素，把大致的轮廓显示出来罢了。

编码
--

既然思路很清晰了，下面就来编码了。

```
# coding:utf-8
import sys

reload(sys)
sys.setdefaultencoding('utf8')
#    __author__ = '郭 璞'
#    __date__ = '2016/8/4'
#    __Desc__ = 一个可以将图片转换成终端字符形式的小工具

from time import *
import Image
class ImageTool():

    def __init__(self):
        print 'Initialization Completed! @',ctime()

    def getChars(self,image_pixels,image_width,image_height):
        replace_chars = 'ABCDEFGHIJKLMNO '
        terminal_chars = ''
        for h in xrange(image_height):
            for w in xrange(image_width):
                point_pixel = image_pixels[w,h]
                terminal_chars +=replace_chars[int(sum(point_pixel)/3.0/256.0*16)]
            terminal_chars+='\n'
        return terminal_chars

    def formatImage(self,imagename,image_width,image_height):
        img = Image.open(imagename,'rb')
        if img.mode != 'RGB':
            img = img.convert('RGB')
        w,h = img.size
        rw = image_width*1.0/w
        rh = image_height*1.0/h
        r = rw if rw<rh else rh
        rw = int(r*w)
        rh = int(r*h)
        img = img.resize((rw,rh),Image.ANTIALIAS)
        return img

    def entrance(self,image_path,out_width,out_height):
        image = self.formatImage(imagename=image_path,image_width=out_width,image_height=out_height)
        image_pixels = image.load()
        out_width ,out_height = image.size
        terminal_chars = self.getChars(image_pixels=image_pixels,image_width=out_width,image_height=out_height)

if __name__ == "__main__":
    tool = ImageTool()
    imagename = sys.argv[1]
    w = int(sys.argv[2])
    h = int(sys.argv[3])
    tool.entrance(imagename,w,h)
```

运行
--

运行程序很简单，我们按照命令行参数来执行即可。

```
python Image2Chars.py target_image_name output_width output_height
```

> 注意，图片名称是包含完整的路径的图片名称

结果
--

*   素材图片  
    ![](https://img-blog.csdn.net/20160804092359509)
    
*   终端显示效果  
    ![](https://img-blog.csdn.net/20160804092436369)
    

文字类型的看起来还凑活，细腻类型的图片就不太好了。这就是因为我们转换像素的时候的粒度有点大了的缘故。