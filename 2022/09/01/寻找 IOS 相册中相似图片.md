> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6969457808378953735)

[NSNotification 与类对象, 实例对象](https://juejin.cn/post/6966865296632053790 "https://juejin.cn/post/6966865296632053790")

原理分析
====

刚开始接到这个需求的时候还是挺懵圈的，由此产生了以下几个问题

*   一张图片是否需要和整个相册除它之外的所有照片进行相似度判断 ？
*   一组图片进行一次相似度筛选还是多次筛选
*   怎么样去处理大量图片进行对比时，产生的临时变量

市面上产品分析
-------

通过对市面上类似的相册清理产品进行分析，发现如下规律

*   同一张图片在同一时间段 (3 小时以内) 内导入相册，进行相册相似度筛选，可以被发现是相似图片
    
*   同一张图片在不同时间段 (相差 3 小时) 导入相册，进行相册相似度筛选，不能被发现为相似图片
    

在一个集合内可以判断相似度。在不同的集合，即使是相似的两张图片也不能找出来。

作者推测这样做的目的是，因为产生相似图片的大多数情况都是，在同一时间段内生成的图片。比如在同一个时间段内对某个物体或者人物的拍照，很可能因为取景的角度，或拍摄的方向产生相似的图片。这些相似的图片大多数都是在相差不大的时间里产生的。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a14373f4bfb149fd83405fc82d1beaae~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

流程设计
----

根据以上分析，我们可以分析出程序大致流程。

+ (NSArray*) fectchSimilarArray { NSMutableArray* outputArray = [NSMutableArray array]; //设置筛选条件 PHFetchOptions *momentOptions = [self options:@"startDate" ascending:NO]; PHFetchOptions *asstsOptions = [self options:@"creationDate" ascending:NO]; PHFetchResult* collectionList = [PHCollectionList fetchCollectionListsWithType:PHCollectionListTypeMomentList subtype:PHCollectionListSubtypeMomentListCluster options:momentOptions]; [collectionList enumerateObjectsUsingBlock:^(id _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) { //创建一个时刻存放数组 PHCollectionList* moment = (PHCollectionList*) obj; //获取时刻里面的Asset集合 PHFetchResult<PHAssetCollection*>* result = [PHAssetCollection fetchMomentsInMomentList:moment options:momentOptions]; [result enumerateObjectsUsingBlock:^(id _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) { //获取的时刻时刻 PHAssetCollection* collection = (PHAssetCollection*) obj; //获取以天为单位的资源集合 PHFetchResult<PHAsset*>* daysResults = [PHAsset fetchAssetsInAssetCollection:collection options:asstsOptions]; //天组中张数大于1才可以进入相似度判断 if (daysResults.count > 1) { //转换数组对象 NSMutableArray<PHAsset*>* assts = [NSMutableArray array]; //筛选媒体类型，只留照片类型 [daysResults enumerateObjectsUsingBlock:^(PHAsset * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) { if (obj.mediaType == PHAssetMediaTypeImage) { [assts addObject:obj]; } }]; //以小时为单位分离数组 检测相似度 NSArray* disArray = [AMSimilarityManager similarityGroup:assts]; if (disArray.count > 0) { [outputArray addObjectsFromArray:disArray]; } } }]; }]; return outputArray; } 复制代码

相似度判断
=====

前面我们已经使用系统提供的方法，完成资源集合的获取，并且按照天为单位分组了图片资源。下面我们需要对每个组里面的图片资源进行相识度判断。

并查集生成相似数组
---------

判断逻辑

1.  首先从输入数组中取出第一个元素作为代表元素，依次和数组里其他元素进行判断，如果符合条件则放入以第一个元素为代表的相似数组
    
2.  将以第一个元素为代表的相似数组元素从输入数组中移除，并且再次从输入数组中取一个元素作为相似度数组的代表元素对输入数组中的剩下的元素进行判断。
    
3.  按照以上逻辑，最后把一天当中的一组图片，分为多组相似图片数组，完成相似图片的查找。
    

```
+ (NSArray*) similarityGroup:(NSArray<PHAsset*>*) array{
    NSMutableArray* inputArray = [NSMutableArray arrayWithArray:array];
    NSMutableArray* outputArray = [NSMutableArray array];
    int i = 0;
    int j = 0;
    
    //理论上会根据输入数组的个数，创建相应的分类数组
    while(i < inputArray.count) {
        //取出代表元素
        //添加自动释放池，处理临时变量
        @autoreleasepool {

            PHAsset* representObjct = inputArray[i];
            //初始化一个离群数组，并且放入代表元素
            NSMutableArray* disGroupArray = [NSMutableArray arrayWithObject:representObjct];
            j = i + 1;

            while (j < inputArray.count) {
                //添加自动释放池，处理临时变量
                @autoreleasepool {
                    //获取对比对象
                    PHAsset* challengeObject = inputArray[j];
                    //判断时间是否间隔小于一定的时间间隔
                    if ([self judgeTime:representObjct and:challengeObject]) {
                        //判断相似度
                        if ([self judge:representObjct and:challengeObject]) {
                            [disGroupArray addObject:challengeObject];
                        }
                        //继续，指向下一位
                        j++;
                    }
                    else {
                        //因为是升序的如果当前不等，则以后的都不等
                        break;
                    }
                }
            }

            if (disGroupArray.count > 1) {

                for (PHAsset* asset in disGroupArray) {
                    //从输入数组中，移除已经放入相似集合的数组
                    [inputArray removeObject:asset];
                }

                [outputArray addObject:disGroupArray];
            }
            else {
                i++;
            }
        }
    }
    return outputArray;
}
复制代码

```

时间
--

在这里我们设置时间的判断是为了更好的区分一天中的某一些时刻，尽可能的减小需要进行下面图像相似度判断的数组大小。当然如果你是以一天为一个时刻的话就可以不需要进行时间的判断。

OpenCV
------

由于基于图片的相似度判断，需要用到大量的矩阵计算，所以这里选择使用 OpenCV 来完成以下图片相似度算法的实现。这里使用的是 OpenCV 4.3.0

```
pod 'OpenCV2', '~> 4.3.0'
复制代码

```

PHASH 算法
--------

PHASH 算法又称感知哈希算法，可以将一个图片矩阵转化为一个长度相等的字符串指纹，通过比较两张图片的指纹每个位置的值是否相等，如果不相等则距离 + 1, 距离越大图片越不相似。通常用来从轮廓上判断图片的相似度的。

### 实现步骤

1.  首先使用 OpenCV 提供的方法，将 UIImage 转为 Mat 图片矩阵。
2.  将 RGB 的图片矩阵转化为灰度图片矩阵，因为感知哈希是从轮廓上判断图片的，所以图片的颜色也就显得不那么重要
3.  使用 DCT 离散余弦变换，将图片的特征转换到左上角，然后在取左上角 8X8 的区域作为图片的特征区域
4.  图片矩阵是二维的数组，我们将这个二维数组转为一维数组，并求和，求平均数。
5.  使用转化出来的一维数组的每一位和上一步算出来的平均数进行比较，大于设置 1，小于等于设置为 0
6.  将得到的 0 和 1 数组和另一张图片的 0 和 1 数组，逐位进行比较，不相等则 distance 距离 + 1。
7.  最后判断两张图片的 distance 距离是否小于 10，如果小于 10 则表示两张图片相似

### 代码实现

```
- (vector<uchar>) phash:(Mat) originMat
{
    //RGB转灰度矩阵
    Mat src = [self rgbToGray:originMat];
    //矩阵离散余弦变化
    src = [self dctMat:src];
    
    //取左上角8x8区域
    cv::Rect rect(0, 0,8,8);
    Mat mat = src(rect);
//    Scalar ss = cv::sum(mat);
    //将二维数组转化为一维数组并求和，求平均数
    double mean = sumMat(mat);
//    NSLog(@"average %lf",mean);
    //判断数组里面每个元素和平均数的大小，大于平均数设置1，小于平均数设置0
    averageArray(mat,mean);
    
    vector<uchar> array;
    for (int i = 0; i < mat.rows; ++i) {
        array.insert(array.end(), mat.ptr<uchar>(i), mat.ptr<uchar>(i)+mat.cols);
      }
    return array;
}
复制代码

```

HSV 直方图
-------

经过上面的 phash 感知算法，我们已经能够识别一部分在轮廓上相似的图片，但是有一些轮廓相似但颜色区别很大的图片就无法识别。所以我们需要 HSV 颜色直方图来统计图片的颜色分布情况，进而比较图片的颜色接近程度。

### HSV 通道

如下图所以示

**H: 色调和色相 (不同的角度代表不同的颜色)**

**S: 代表饱和度（从左到右颜色越来越明显）**

**V: 代表明度 (从下到上越来越明亮)**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59e41709bc984411ad5d6902ec6aaaf5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 直方图

**横坐标** bin 代表一个区间 例如 (0-255) 可以分为 8 个区间 一个 bin 为 32，一共 8 个 bin，下图是 RGB 的直方图表示法，因为 RGB 每个通道的取值范围都是 (0-255)，所以一共有 8X8X8 个 bin, 即是 512 个 bin。

**纵坐标** 代表在这个 bin(区间) 中分布的像素数量。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b23512506e346cab1411753fe268af6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 实现步骤

1.  首先跟 phash 算法处理一样，我们需要把 UIImage 转化为 OpenCV 能处理的 Mat
2.  将两张图片都是转化为 Mat 之后，我们分别将两张图片从 RGB 通道转化为 HSV 通道
3.  设置直方图的 bin，这里我们只使用 HSV 通道中的 H 的值和 S 的值，先不考虑亮度，只考虑色相和饱和度。
4.  使用 OpenCV 来生成直方图。
5.  对两张图片的直方图进行归一化处理，归一化就是要把需要处理的数据经过处理后（通过某种算法）限制在你需要的一定范围内，这样我们就可以直观的比较两张图片的颜色分布。
6.  将两张图片归一化的数据通过 OpenCV 提供的 compareHist，进行相关性比较。如果大于 0.6 则表示两张图片在颜色分布上相似。

### 代码实现

```
- (BOOL) histogramCompare
{
//    :(Mat) base andMat:(Mat) compare
    Mat base = originMat;
    Mat compare = compareMat;
    
    Mat hsvbase,hsvcompare;
    cvtColor(base, hsvbase, COLOR_BGR2HSV);
    cvtColor(compare, hsvcompare, COLOR_BGR2HSV);
    
    //h通道的bin个数为50个，那么h通道每一个bin的宽度也就是 180 / 50 因为h取值范围是0到180
    //s通道的bin个数为60个，那么s通道每一个bin的宽度也就是 256 / 60 因为s取值范围是0到256
    int h_bins = 50; int s_bins = 60;
    int histSize[] = { h_bins, s_bins };
    // hue varies from 0 to 179, saturation from 0 to 255
    //h通道维度的横轴取值范围是0到180
    //s通道维度的横轴取值范围是0到256
    float h_ranges[] = { 0, 180 };
    float s_ranges[] = { 0, 256 };
    const float* ranges[] = { h_ranges, s_ranges };
    // Use the o-th and 1-st channels
    int channels[] = { 0, 1 };
    
    MatND hist_base;
    MatND hist_compare;
    
    calcHist(&hsvbase, 1, channels, Mat(), hist_base, 2, histSize, ranges,true, false);
    calcHist(&hsvcompare, 1, channels, Mat(), hist_compare, 2, histSize, ranges, true, false);
    
    normalize(hist_base, hist_base,0, 1, NORM_MINMAX, -1, Mat());
    normalize(hist_compare, hist_compare, 0, 1, NORM_MINMAX, -1, Mat());
    
    return  compareHist(hist_base, hist_compare, HISTCMP_CORREL) >= MaxHis ? YES : NO;
    
}
复制代码

```

ORB 特征提取
--------

在经过 PHASH 和 HSV 直方图两种算法的过滤之后，我们已经能够获取两张在轮廓上，颜色分布上相似的图片。但是在图片的局部细节上我们依旧无法判断是否相似。

ORB 特征是将 FAST 特征点的检测方法与 BRIEF 特征描述子结合起来，并在它们原来的基础上做了改进与优化。 ORB 采用 FAST（features from accelerated segment test）算法来检测特征点。FAST 核心思想就是找出那些卓尔不群的点，即拿一个点跟它周围的点比较，如果它和其中大部分的点都不一样就可以认为它是一个特征点。

### 实现步骤

1.  同样使用的是 OpenCV 实现 ORB 特征提取，所以我们需要将 UIImage 转化为 Mat
2.  初始化一个 ORB 提取器，传入两张图片的 Mat 矩阵。得到特征点集合
3.  初始化一个 BFMatcher, 将两个容器里面的特征点匹配关系
4.  如果第一个点的距离小于第二个点的距离的 75%，则表示第一个点是两张图都相似的特征点。
5.  如果相似的特征点占总数的 15% 则表示两张图特征相似

### 实现代码

```
- (BOOL) orbCompare {
    
    Mat img_1 = originMat;
    Mat img_2 = compareMat;
    
    cv::Ptr<ORB> orb = ORB::create();
    std::vector<KeyPoint> keypoints_1,keypoints_2;
    Mat descriptors_1, descriptors_2;


    orb->detectAndCompute(img_1, Mat(), keypoints_1, descriptors_1);
    orb->detectAndCompute(img_2, Mat(), keypoints_2, descriptors_2);

    if (keypoints_1.size() <= 0 || keypoints_2.size() <= 0) {
        //无法判断相似 特征提取失败
        return 0;
    }
    
    BFMatcher matcher(NORM_HAMMING);
    std::vector<vector<DMatch> > matches;
    matcher.knnMatch(descriptors_1, descriptors_2, matches,2);
    

    vector<double> good;
    for(int i = 0; i < matches.size();i++) {
        
        vector<DMatch> matchesValue = matches[i];
        double m = matchesValue[0].distance;
        double n = matchesValue[1].distance * 0.75;
        if (m < n) {
            good.insert(good.end(), m);
        }
    }
 
    return matches.size() == 0 ? NO :good.size()*1.0/(matches.size()*1.0) >= MaxOrb ? YES : NO ;
}
复制代码

```

样例测试
====

经过测试，我手机里面有 1653 张图片，大概 30 秒左右就已经计算完毕。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f52eafb53fd94d98b4c53932cf5fca09~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

总结
==

这个功能在实现过程中主要的难点集中在图像的处理上面，但因为有了 OpenCV 这个强大的库，我们站在巨人的肩膀上很轻易的就完成了图片的矩阵操作等等。

项目地址
----

### 觉得 Demo 有用的朋友，可以给我点个小星星哦

[IOS 相册相似图片寻找](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FcoderFrankenstain%2FWJSimilarPhotos "https://github.com/coderFrankenstain/WJSimilarPhotos")