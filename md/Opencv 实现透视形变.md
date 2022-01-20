> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Ks-hI6nw0rArzVIthREHqQ)

计算机视觉现在很流行，世界各地的人们都在从事某种形式的基于深度学习的计算机视觉项目。但在深度学习出现之前，图像处理技术已被用来处理和转换图像，以获得有助于我们完成任务的见解。今天，让我们看看如何实现一种简单而有用的技术，即透视投影来扭曲图像。

那么扭曲图像是什么意思？我可以用很多花哨的词和技术术语来解释它。但是，展示最终结果很容易，这样我们就可以通过观察来学习。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/4AqSEnNUeribDx7y3CjcGqXWJqknf8cpFXz2YibsTWyKZSGLcticxx6aKd0sqIWMy2OkvAp5hcpT0pRsBsUJOarLA/640?wx_fmt=jpeg)  ![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/4AqSEnNUeribDx7y3CjcGqXWJqknf8cpFHxZeCW9b2C12ibb2OL4tjEeNW2uYc4XKyQPvthdpLKIra1gTpphAmrg/640?wx_fmt=jpeg)  ![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/4AqSEnNUeribDx7y3CjcGqXWJqknf8cpF1b6TNSU4KzQtYTyWGgvic2k5k383SictVUibB4W0KgKqUNRXOmDrzW2ww/640?wx_fmt=jpeg)

基础图像——主题图像——扭曲的输出

```
base_image = cv2.imread('base_img.jpg')
base_image_copy = base_image.copy()
subject_image = cv2.imread('subject.jpg')

```

                           基本图像（左）——主体图像（右）

```
def click_event(event, x, y, flags, params):
    if event == cv2.EVENT_LBUTTONDOWN:
        cv2.circle(base_image_copy, (x, y), 4, (0, 0, 255), -1)
        points.append([x, y])
        if len(points) <= 4:
            cv2.imshow('image', base_image_copy)
points = []
base_image = cv2.imread('base_img.jpg')
base_image_copy = base_image.copy()
subject_image = cv2.imread('subject.jpg')
cv2.imshow('image', base_image_copy)
cv2.setMouseCallback('image', click_event)
cv2.waitKey(0)
cv2.destroyAllWindows()

```

```
def sort_pts(points):
    sorted_pts = np.zeros((4, 2), dtype="float32")
    s = np.sum(points, axis=1)
    sorted_pts[0] = points[np.argmin(s)]
    sorted_pts[2] = points[np.argmax(s)]
    diff = np.diff(points, axis=1)
    sorted_pts[1] = points[np.argmin(diff)]
    sorted_pts[3] = points[np.argmax(diff)]
    return sorted_pts
sorted_pts = sort_pts(points)

```

```
h_base, w_base, c_base = base_image.shape
h_subject, w_subject = subject_image.shape[:2]
pts1 = np.float32([[0, 0], [w_subject, 0], [w_subject, h_subject],                     [0, h_subject]])
pts2 = np.float32(sorted_pts)

```

```
transformation_matrix = cv2.getPerspectiveTransform(pts1, pts2)
warped_img = cv2.warpPerspective(subject_image, transformation_matrix, (w_base, h_base))
cv2.imshow('Warped Image', warped_img)

```

变形的图像看起来像这样：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4AqSEnNUeribDx7y3CjcGqXWJqknf8cpFk3Ls3hFXxr8EPWichDR4dGukmJYph7crftKbevvWN1Sew7qq5d4zPiaA/640?wx_fmt=other)变形的图像

下一步是创建一个蒙版，我们为其创建一个具有基本图像形状的空白图像。

```
mask = np.zeros(base_image.shape, dtype=np.uint8)

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4AqSEnNUeribDx7y3CjcGqXWJqknf8cpFp7ib9Mv0YvwUcWoic8kOG7b0m8R0MSmicsdSHZib3UH9CT8cSXzFYL5b2g/640?wx_fmt=png)初始蒙版

在这个空白蒙版上，我们绘制一个具有由 “sorted_pts” 指定的角的多边形，并使用 cv2.fillConvexPoly() 方法将其填充为白色，生成的蒙版将如下所示。

```
roi_corners = np.int32(sorted_pts)
cv2.fillConvexPoly(mask, roi_corners, (255, 255, 255))

```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4AqSEnNUeribDx7y3CjcGqXWJqknf8cpF0oI0ediaf4pYbR3PcAk8ICPaoToHxj8W9F22MiaqNbbOIibia60l5apy1g/640?wx_fmt=png)填充蒙版

现在我们使用 cv2.bitwise_not() 方法反转蒙版颜色。

```
mask = cv2.bitwise_not(mask)

```

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/4AqSEnNUeribsINpD9lMeOdcb75poULycPyMct84bkg9mUTkxjIwUzviaEHezGK5kUX4Ny5yYAAm7JHcasV9LhRQ/640?wx_fmt=jpeg)倒置蒙版

现在我们使用 cv2.bitwise_and() 方法获取蒙版和基础图像并执行按位与运算。

```
masked_image = cv2.bitwise_and(base_image, mask)

```

这将为我们提供如下所示的图像。我们可以看到单独放置对象图像的区域是黑色的。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/4AqSEnNUeribDx7y3CjcGqXWJqknf8cpF89DIHU7GrYP97ggSAmNmb1O3f8p3iaWdQ4M3MRicHNoAo6ZEgSkgf39w/640?wx_fmt=png)蒙面基础图像

最后一步是使用 cv2.bitwise_or() 方法获取变形图像和蒙版图像并执行按位或运算，这将生成我们想要完成的融合图像。

```
output = cv2.bitwise_or(warped_img, masked_image)
cv2.imshow('Fused Image', output)
cv2.imwrite('Final_Output.png', output)
cv2.waitKey(0)
cv2.destroyAllWindows()

```

我们做到了！我们已经成功地将一张图片叠加到另一张图片上。

融合图像

这是透视变换的一个非常简单的用例。当我们跟踪框架中物体 / 人物的运动时，可以使用它来生成区域的鸟瞰图。

Github 代码连接：  

https://github.com/GSNCodes/Image_Overlaying_Using_Perspective_Transform