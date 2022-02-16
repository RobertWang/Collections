> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0NjgzMDIxMQ==&mid=2247585108&idx=2&sn=552a2d60cb6c69675ed6a37302a65bd7&chksm=fb546fb8cc23e6ae3bc6993dcea7fbbaceee36704e0f4ea28c96b0a11be284c8634e4ca86ac5&mpshare=1&scene=1&srcid=0206G8sqTeQY2q6LaSb7ulSg&sharer_sharetime=1644116099125&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

秘密数据可以是任何格式的数据，如文本甚至文件。简而言之，隐写术的主要目的是隐藏任何文件（通常是图像、音频或视频）中的预期信息，而不实际改变文件的外观，即文件外观看起来和以前一样。

在这篇文章中，我们将重点学习基于图像的隐写术，即在图像中隐藏秘密数据。

但在深入研究之前，让我们先看看图像由什么组成：

1.  像素是图像的组成部分。
    
2.  每个像素包含三个值：（红色、绿色、蓝色）也称为 RGB 值。
    
3.  每个 RGB 值的范围从 0 到 255。
    

现在，让我们看看如何将数据编码和解码到我们的图像中。

有很多算法可以用来将数据编码到图像中，实际上我们也可以自己制作一个。在这篇文章中使用的一个很容易理解和实现的算法。

算法如下：

1.  对于数据中的每个字符，将其 ASCII 值转换为 8 位二进制 [1]。
    
2.  一次读取三个像素，其总 RGB 值为 3*3=9 个。前八个 RGB 值用于存储一个转换为 8 位二进制的字符。
    
3.  比较相应的 RGB 值和二进制数据。如果二进制数字为 1，则 RGB 值将转换为奇数，否则为偶数。
    
4.  第 9 个值确定是否应该读取更多像素。如果有更多数据要读取，即编码或解码，则第 9 个像素变为偶数；否则，如果我们想停止进一步读取像素，那就让它变得奇数。
    

重复这个过程，直到所有数据都被编码到图像中。

假设要隐藏的消息是‘Hii’。

消息是三个字节，因此，对数据进行编码所需的像素为 3 x 3 = 9。考虑一个 4 x 3 的图像，总共有 12 个像素，这足以对给定的数据进行编码。

H 的 ASCII 值为 72 ，其二进制等效值为 01001000 。

**第 2 步**

读取前三个像素。

现在，将像素值更改为奇数为 1，偶数为 0，就像在二进制等效数据中一样。

例如，第一个二进制数字是 0，第一个 RGB 值是 27 ，它需要转换为偶数，这意味着 26 。类似地，64 被转换为 63 因为下一个二进制数字是 1 所以 RGB 值应该是奇数。

因此，修改后的像素为：

```
(26, 63, 164), (248, 243, 194), (174, 246, 250)
```

由于我们必须对更多数据进行编码，因此最后一个值应该是偶数。同样，i 可以在这个图像中进行编码。

通过执行 +1 或 -1 使像素值成为奇数 / 偶数时，我们应该注意二进制条件。即像素值应大于或等于 0 且小于或等于 255 。

新图像将如下所示：

对于解码，我们将尝试找到如何逆转之前我们用于数据编码的算法。

1.  同样，一次读取三个像素。前 8 个 RGB 值为我们提供了有关机密数据的信息，第 9 个值告诉我们是否继续前进。
    
2.  对于前八个值，如果值为奇数，则二进制位为 1 ，否则为 0 。
    
3.  这些位连接成一个字符串，每三个像素，我们得到一个字节的秘密数据，这意味着一个字符。
    
4.  现在，如果第 9 个值是偶数，那么我们继续一次读取三个像素，否则，我们停止。
    

让我们开始一次读取三个像素。

**第 1 步**

```
[(26, 63, 164), (248, 243, 194), (174, 246, 250)
```

**第 2 步**

**第 3 步**

**第 4 步**

上述算法的 Python 程序如下：

```
# Python program implementing Image Steganography
# PIL module is used to extract
# pixels of image and modify it
from PIL import Image
# Convert encoding data into 8-bit binary
# form using ASCII value of characters
def genData(data):
        # list of binary codes
        # of given data
        newd = []
        for i in data:
            newd.append(format(ord(i), '08b'))
        return newd
# Pixels are modified according to the
# 8-bit binary data and finally returned
def modPix(pix, data):
    datalist = genData(data)
    lendata = len(datalist)
    imdata = iter(pix)
    for i in range(lendata):
        # Extracting 3 pixels at a time
        pix = [value for value in imdata.__next__()[:3] +
                                imdata.__next__()[:3] +
                                imdata.__next__()[:3]]
        # Pixel value should be made
        # odd for 1 and even for 0
        for j in range(0, 8):
            if (datalist[i][j] == '0' and pix[j]% 2 != 0):
                pix[j] -= 1
            elif (datalist[i][j] == '1' and pix[j] % 2 == 0):
                if(pix[j] != 0):
                    pix[j] -= 1
                else:
                    pix[j] += 1
                # pix[j] -= 1
        # Eighth pixel of every set tells
        # whether to stop ot read further.
        # 0 means keep reading; 1 means thec
        # message is over.
        if (i == lendata - 1):
            if (pix[-1] % 2 == 0):
                if(pix[-1] != 0):
                    pix[-1] -= 1
                else:
                    pix[-1] += 1
        else:
            if (pix[-1] % 2 != 0):
                pix[-1] -= 1
        pix = tuple(pix)
        yield pix[0:3]
        yield pix[3:6]
        yield pix[6:9]
def encode_enc(newimg, data):
    w = newimg.size[0]
    (x, y) = (0, 0)
    for pixel in modPix(newimg.getdata(), data):
        # Putting modified pixels in the new image
        newimg.putpixel((x, y), pixel)
        if (x == w - 1):
            x = 0
            y += 1
        else:
            x += 1
# Encode data into image
def encode():
    img = input("Enter image name(with extension) : ")
    image = Image.open(img, 'r')
    data = input("Enter data to be encoded : ")
    if (len(data) == 0):
        raise ValueError('Data is empty')
    newimg = image.copy()
    encode_enc(newimg, data)
    new_img_name = input("Enter the name of new image(with extension) : ")
    newimg.save(new_img_name, str(new_img_name.split(".")[1].upper()))
# Decode the data in the image
def decode():
    img = input("Enter image name(with extension) : ")
    image = Image.open(img, 'r')
    data = ''
    imgdata = iter(image.getdata())
    while (True):
        pixels = [value for value in imgdata.__next__()[:3] +
                                imgdata.__next__()[:3] +
                                imgdata.__next__()[:3]]
        # string of binary data
        binstr = ''
        for i in pixels[:8]:
            if (i % 2 == 0):
                binstr += '0'
            else:
                binstr += '1'
        data += chr(int(binstr, 2))
        if (pixels[-1] % 2 != 0):
            return data
# Main Function
def main():
    a = int(input(":: Welcome to Steganography ::\n"
                        "1. Encode\n2. Decode\n"))
    if (a == 1):
        encode()
    elif (a == 2):
        print("Decoded Word :  " + decode())
    else:
        raise Exception("Enter correct input")
# Driver Code
if __name__ == '__main__' :
    # Calling main function
    main()
```

程序中使用的模块是 PIL ，它代表 Python 图像库，它使我们能够在 Python 中对图像执行操作。

**程序执行**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4AqSEnNUer9gB5iauIpDquqZW17ZvZ3VJiaEvdGtJcYskysygUwzGKDotVw3sbFzhcia8rnu2TT3lmS78SOUYLekA/640?wx_fmt=png)

数据编码

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4AqSEnNUer9gB5iauIpDquqZW17ZvZ3VJIkLrl68soon84W0UG7Ph1iaYKDn7zSLGk031g2Pr5nH4OXe4h2cGI3w/640?wx_fmt=png)

数据解码

**输入图像**
--------

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4AqSEnNUer9gB5iauIpDquqZW17ZvZ3VJuyNVh14X3w98YQuicialAlxVsaIV97dDXYv6gCUrjfK6x4rG82cQFaDQ/640?wx_fmt=png)

**输出图像**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4AqSEnNUer9gB5iauIpDquqZW17ZvZ3VJuyNVh14X3w98YQuicialAlxVsaIV97dDXYv6gCUrjfK6x4rG82cQFaDQ/640?wx_fmt=png)

**局限性**
=======

该程序可能无法对 JPEG 图像按预期处理，因为 JPEG 使用有损压缩，这意味着修改像素以压缩图像并降低质量，因此会发生数据丢失。

**参考**

1.  https://www.geeksforgeeks.org/program-decimal-binary-conversion/
    
2.  https://www.geeksforgeeks.org/working-images-python/
    
3.  https://dev.to/erikwhiting88/let-s-hide-a-secret-message-in-an-image-with-python-and-opencv-1jf5
    
4.  A code along with the dependencies can be found here: https://github.com/goelashwin36/image-steganography