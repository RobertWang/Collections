> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0D7LNHM12APNNgPv_qhacg)

![](https://mmbiz.qpic.cn/mmbiz_png/RxKDrJsia0MHvPUA46nqGwUialKibTqBuPLNCia6qFH6lfmIscwpv7sUWBlvGhYMrO8FEBF2qcdAcaDx7iaia68elfjg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RxKDrJsia0MHvPUA46nqGwUialKibTqBuPLswdibALjicaamPKMIQWCfSJgvmYUjvylKiad861KFoibAIC4jvJksmvaAw/640?wx_fmt=png)

你都看到了，二者的区别是**图标不同**。

这个有点冷啊。。。

切入正题。

OpenMV 和 OpenCV 都是用来进行**图像处理**的，但它们有一些区别和不同的应用场景。

**OpenMV** 是一个开源，低成本，功能强大的   机器视觉模块。OpenMV 上的机器视觉算法包括  寻找色块、人脸检测、眼球跟踪、边缘检测、标志跟踪等。以 STM32F427CPU 为核心，集成了 OV7725 摄像头芯片，在小巧的硬件模块上，用 C 语言高效地实现了核心机器视觉算法，提供 Python 编程接口。

**OpenCV**（Open Source Computer Vision Library）是一个广泛使用的计算机视觉库，支持多种编程语言，如 C++、Python 和 Java 等。OpenCV 提供了丰富的图像处理和计算机视觉算法，包括特征检测、图像分割、物体识别和跟踪等。OpenCV 可以在不同的平台上运行，包括嵌入式系统和桌面计算机。它是一个功能强大的工具库，适用于各种图像处理和计算机视觉任务，从简单的图像滤波到复杂的目标检测和人脸识别。

**区别：**

**开发重点**：OpenMV 专注于嵌入式系统和物联网应用，提供了一个硬件平台和相应的软件工具，旨在简化嵌入式图像处理和机器视觉的开发过程。而 OpenCV 是一个通用的计算机视觉库，支持多种编程语言，并且适用于各种平台和应用领域。

**编程语言**：OpenMV 主要使用 Python 编程语言，通过简单的脚本来编写图像处理代码。对于嵌入式系统而言，Python 的易用性和可移植性使得开发过程更加便捷。而 OpenCV 支持多种编程语言，包括 C++、Python 和 Java 等。

**功能和算法**：OpenMV 针对嵌入式应用提供了一系列简化的图像处理函数和 API，如颜色识别、目标跟踪和形状分析等。这些函数和 API 经过优化，能够在资源有限的嵌入式环境下高效运行。OpenCV 则提供了广泛且强大的图像处理和计算机视觉算法，如特征检测、图像分割、物体识别和跟踪等。它的功能更全面、更适用于通用计算机视觉任务。

**联系：**

**图像处理与计算机视觉**：OpenMV 和 OpenCV 都是用于图像处理和计算机视觉的工具库，可以应用于颜色识别、目标检测、形状分析、图像滤波等各种任务。

**嵌入式系统应用**：OpenMV 在嵌入式系统和物联网应用中有优势，它提供了硬件平台和相应的软件工具，便于开发者在资源有限的环境下进行图像处理和机器视觉开发。OpenCV 也可以在嵌入式系统上使用，但由于其更多功能和更大的资源需求，通常更适用于较强资源的平台。

**互补应用**：在某些场景下，OpenMV 和 OpenCV 可以结合使用。例如，可以使用 OpenMV 进行实时图像采集和初步处理，然后将处理后的图像传递给 OpenCV 进行更复杂的图像处理和计算机视觉任务。

总的来说，OpenMV 适用于嵌入式系统环境下对实时图像进行处理和分析的应用，例如机器人视觉、自动化和嵌入式视觉项目。而 OpenCV 则更适用于通用计算机视觉任务，可以在各种平台上进行开发和部署，包括桌面应用、移动设备和服务器等。根据具体的应用场景和需求，选择适合的工具库可以提高开发效率并实现所需的功能。