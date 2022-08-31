> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7072618382201192461)*   浮点算法：Gray = R0.3 + G0.59 + B0.11
*   整数方法：Gray = (R30+G59+B11)/100
*   移位方法：Gray =（R28+G151+B*77）>> 8
*   平均值法：Gray = (R+G+B)/3
*   仅取绿色：Gray = G

### 离散余弦变换 DCT(discrete cosine transform)

是一种将图片的特征值 “汇聚” 在“左上角”的算法

### 余弦相似度

用向量空间中的两个向量夹角的余弦值作为衡量两个个体间差异大小的度量，值越接近 1，就说明夹角角度越接近 0°。也就是两个向量越相似，就叫做余弦相似

### Javascript ImageData 对象

ImageData 里面有个 data 属性，它是一个一维数组。其中每四个元素代表一个像素点，分别为 Red、Green、Blue、Alpha。

譬如：[R,G,B,A,R,G,B,A,R,G,B,A,R,G,B,A] 代表一个尺寸为 2*2 的图片的颜色信息，即长度为 2*2*4

处理流程
----

1.  **伸缩尺寸**，将图片伸缩到 32*32 的尺寸（这样做的目的是简化了 DCT 的计算，而不是减小频率）
2.  **简化色彩**，将图片转化成灰度图（进一步简化计算量）
3.  **离散余弦变换**，经过离散余弦变换，得到新的 32*32 的矩阵（实际数据是一个长度为 1024 的一维数组）
4.  **截取左上角**，经过离散余弦变换后，矩阵的左上角将会呈现图片中的最低频率，所以我们只要保留左上角的 8*8 的矩阵，就足以代表整张图片
5.  **获取图片指纹**，根据 8*8 的矩阵，先计算总体平均值，每个点与平均值做比较，大于则为 1，小于则为 0，从而组成一个由 0 和 1 组成长度为 64 的一维数组作为 hash 值，这个值就是图片的指纹
6.  **计算差异值**，计算两张图片的指纹的余弦相似度（可以计算汉明距离）

### 常量及类型定义

```
const ExpectImgSize = 32; // 期望图片尺寸
const SamplingRadio = 4; // 取样比例，截取左上角四分之一（若原尺寸为 32*32，则截取 8*8 的区域）
const DCTScale = 2; // 变换参数
type ImageFile = File;
type ImageUrl = string;
type PixelColors = Array<number>; // 像素颜色列表
type Fingerprint = Array<0 | 1>; // 指纹，是由 0 和 1 组成数组
复制代码

```

### 伸缩尺寸（主要是缩小）

将目标图像进行伸缩成 32*32 的尺寸：

```
const getImageData = async function (src: ImageUrl) {
  return new Promise<ImageData>((resolve) => {
    const canvas = document.createElement("canvas");
    const context = canvas.getContext("2d")!;
    canvas.width = ExpectImgSize;
    canvas.height = ExpectImgSize;

    const img = new Image();
    img.onload = function () {
      const _this = this as HTMLImageElement;
      /**
       * 将 _this.width * _this.height 尺寸的图片以 ExpectImgSize * ExpectImgSize 比例绘制
       * 也就意味着会发生伸缩，最终结果是不再保持宽高比的
       * 我们是为了输出用于做对比的图片，所以不包吃宽高比的伸缩也无所谓
       *
       * 因为我们 ExpectImgSize 数值很小，所以一般图片是被缩小的，这也有利于减少计算量
       */
      context.drawImage(
        _this,
        0,
        0,
        _this.width,
        _this.height,
        0,
        0,
        ExpectImgSize,
        ExpectImgSize
      );
      resolve(context.getImageData(0, 0, ExpectImgSize, ExpectImgSize));
    };
    img.src = src;
  });
};
复制代码

```

### 简化色彩（灰度图）

使用**平均值法**将有色图转化为灰度图

顺便简化数据，因为原来 `ImageData` 对象中是以 4 个元素来代表一个像素点的颜色，这里我会合并

```
function getGrayscale(origin: ImageData): PixelColors {
  const result: Array<number> = [];
  for (let n = 0; n < origin.data.length; n++) {
    if ((n + 1) % 4 === 0) {
      const R = origin.data[n - 3];
      const G = origin.data[n - 2];
      const B = origin.data[n - 1];
      // const A = origin.data[n]

      /**
       * 譬如：[R,G,B,A,R,G,B,A,R,G,B,A,R,G,B,A]
       * 转成：[Gray,Gray,Gray,Gray]
       **/
      result.push(Math.floor((R + G + B) / 3)); // 平均后再取整
    }
  }
  return result;
}
复制代码

```

### 离散余弦变换（DCT）

```
function getDCT(colors: ReturnType<typeof getGrayscale>): PixelColors {
  const PI_N = Math.PI / colors.length;
  return colors.map((_, n) => {
    const num =
      DCTScale *
      colors.reduce(
        (total, current, m) => total + current * Math.cos(PI_N * (m + 0.5) * n),
        0
      );

    // 取值范围为 [0,255]
    return Math.min(255, Math.max(0, num));
  });
}
复制代码

```

### 截取左上角

因为数据的存储结构为一维数组，但是实际是表示一个二维的图片像素，所以理解下面的逻辑会有点绕

这里举个 4*4 转化为 2*2 的例子（截取左上角）：

```
/**
 * 原数据：
 *  存储结构：
 *    [A，B，C，D，E，F，G，H，I，J，K，L，M，N，O，P]
 *  实际结构：
 *    [
 *      A，B，C，D，
 *      E，F，G，H，
 *      I，J，K，L，
 *      M，N，O，P，
 *    ]
 */

/**
 * 转化成：
 *  存储结构：
 *    [A，B，E，F]
 *  实际结构：
 *    [
 *      A，B，
 *      E，F，
 *    ]
 **/
复制代码

```

所以逻辑为：**每 M 个截取 N 个，执行 N 次**（上面的例子中，M 为 4，N 为 2）

```
function getLTCornerColors(colors: ReturnType<typeof getDCT>) {
  const MatrixSize = Math.sqrt(colors.length);
  const SamplingSize = MatrixSize / SamplingRadio;

  const result: Array<any> = [];

  // 每 MatrixSize 个截取 SamplingSize 个，执行 SamplingSize 次
  for (let i = 0; i < SamplingSize; i++) {
    result.push(...colors.slice(i * MatrixSize, i * MatrixSize + SamplingSize));
  }
  return result;
}
复制代码

```

### 获取图片指纹

得到一个 64(8*8) 位由 0 或 1 组成的一维数组，这个就是图片的指纹

```
function getFingerprint(imgData: ImageData): Fingerprint {
  const colors = getLTCornerColors(getDCT(getGrayscale(imgData) as any)); // 颜色集合
  const average = colors.reduce((pre, cur) => pre + cur) / colors.length; // 颜色平均值

  // 当颜色比平均值高时取值为1，反之为0
  return colors.map((color) => (color >= average ? 1 : 0));
}
复制代码

```

### 计算差异值（余弦相似度）

计算差异值有很多种方式，譬如：

*   余弦相似度
*   汉明距离

> 有兴趣的同学可以自行去了解更多计算差异的算法，也是很有意思的课题

这里是使用**余弦相似度**：

```
const getCosineSimilarity = function (
  origin: ReturnType<typeof getFingerprint>,
  target: ReturnType<typeof getFingerprint>
) {
  let product = 0, // 乘积
    vecA = 0, // 向量 a
    vecB = 0; // 向量 b
  for (let i = 0; i < origin.length; i++) {
    vecA += Math.pow(origin[i], 2);
    vecB += Math.pow(target[i], 2);
    product += origin[i] * target[i];
  }

  /**
   * 数据可能会出现溢出的可能，导致最后得到的数不完全准确
   * 这里简单判断下两个数是否相同，如果相同就没必要开方后又平方
   *
   * p.s. 最可靠的方式是用大数相乘，但是这样会使处理时间变长，但是其实我们图片对比没必要这么精确
   */
  if (vecA === vecB) {
    return product / vecA;
  }

  return product / (Math.sqrt(vecA) * Math.sqrt(vecB));
};
复制代码

```

完整代码
----

下面是完整的代码，最终以 `compareImageSimilarity` 方法作为导出：

```
const ExpectImgSize = 32; // 期望图片尺寸
const SamplingRadio = 4; // 取样比例，截取左上角四分之一（若原尺寸为 32*32，则截取 8*8 的区域）
const DCTScale = 2; // 变换参数
type ImageFile = File;
type ImageUrl = string;
type PixelColors = Array<number>; // 像素颜色列表
type Fingerprint = Array<0 | 1>; // 指纹，是由 0 和 1 组成数组

/**
 * 通过 file 获取 url
 * @param file 图片文件对象
 * @returns 图片地址
 */
const getImageUrl = async function (file: ImageFile) {
  return new Promise<ImageUrl>((resolve) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = async function (e) {
      resolve(e.target!.result as ImageUrl);
    };
  });
};

/**
 * 通过 url 获取 ImageData 对象
 * @param src 图片地址
 * @returns 图片数据对象
 */
const getImageData = async function (src: ImageUrl) {
  return new Promise<ImageData>((resolve) => {
    const canvas = document.createElement("canvas");
    const context = canvas.getContext("2d")!;
    canvas.width = ExpectImgSize;
    canvas.height = ExpectImgSize;

    const img = new Image();
    img.onload = function () {
      const _this = this as HTMLImageElement;
      /**
       * 将 _this.width * _this.height 尺寸的图片以 ExpectImgSize * ExpectImgSize 比例绘制
       * 也就意味着会发生伸缩，最终结果是不再保持宽高比的
       * 我们是为了输出用于做对比的图片，所以不包吃宽高比的伸缩也无所谓
       *
       * 因为我们 ExpectImgSize 数值很小，所以一般图片是被缩小的，这也有利于减少计算量
       */
      context.drawImage(
        _this,
        0,
        0,
        _this.width,
        _this.height,
        0,
        0,
        ExpectImgSize,
        ExpectImgSize
      );
      resolve(context.getImageData(0, 0, ExpectImgSize, ExpectImgSize));
    };
    img.src = src;
  });
};

/**
 * 转化为灰度图（顺便把数据类型也简化了）
 * @param origin 图片数据对象
 * @returns 像素颜色列表
 */
function getGrayscale(origin: ImageData): PixelColors {
  const result: Array<number> = [];
  for (let n = 0; n < origin.data.length; n++) {
    if ((n + 1) % 4 === 0) {
      const R = origin.data[n - 3];
      const G = origin.data[n - 2];
      const B = origin.data[n - 1];
      // const A = origin.data[n]
      result.push(Math.floor((R + G + B) / 3)); // 平均后再取整
    }
  }
  return result;
}

/**
 * 对原数据进行离散余弦变换
 * @param signal
 * @returns
 */
function getDCT(colors: ReturnType<typeof getGrayscale>): PixelColors {
  const PI_N = Math.PI / colors.length;
  return colors.map((_, n) => {
    const num =
      DCTScale *
      colors.reduce(
        (total, current, m) => total + current * Math.cos(PI_N * (m + 0.5) * n),
        0
      );

    // 取值范围为 [0,255]
    return Math.min(255, Math.max(0, num));
  });
}

/**
 * 截取“左上角”数据
 * @param colors 完整的颜色数组
 * @returns 由左上角区域颜色组成新的一维数组
 */
function getLTCornerColors(colors: ReturnType<typeof getDCT>) {
  /**
   * 因为数据的存储结构为一维数组，但是实际是表示一个二维的图片像素，所以理解下面的逻辑会有点绕
   * 这里举个 4*4 转化为 2*2 的例子（截取左上角）
   *
   * 原数据：
   *  存储结构：
   *    [A，B，C，D，E，F，G，H，I，J，K，L，M，N，O，P]
   *  实际结构：
   *    [
   *      A，B，C，D，
   *      E，F，G，H，
   *      I，J，K，L，
   *      M，N，O，P，
   *    ]
   *
   * 转化成：
   *  存储结构：
   *    [A，B，E，F]
   *  实际结构：
   *    [
   *      A，B，
   *      E，F，
   *    ]
   *
   * 所以逻辑为：每M个截取N个，执行N次（M为4，N为2）
   */

  const MatrixSize = Math.sqrt(colors.length);
  const SamplingSize = MatrixSize / SamplingRadio;
  const result: Array<any> = [];

  // 每MatrixSize个截取SamplingSize个，执行SamplingSize次
  for (let i = 0; i < SamplingSize; i++) {
    result.push(...colors.slice(i * MatrixSize, i * MatrixSize + SamplingSize));
  }
  return result;
}

/**
 * 获取图片的指纹
 * @param imgData 图片数据对象
 * @returns 指纹
 */
function getFingerprint(imgData: ImageData): Fingerprint {
  const colors = getLTCornerColors(getDCT(getGrayscale(imgData) as any)); // 颜色集合
  const average = colors.reduce((pre, cur) => pre + cur) / colors.length; // 颜色平均值

  // 当颜色比平均值高时取值为1，反之为0
  return colors.map((color) => (color >= average ? 1 : 0));
}

/**
 * 计算出两张图片指纹的余弦相似度
 * @param origin 图片A
 * @param target 图片B
 * @returns 余弦相似度
 */
const getCosineSimilarity = function (
  origin: ReturnType<typeof getFingerprint>,
  target: ReturnType<typeof getFingerprint>
) {
  let product = 0, // 乘积
    vecA = 0, // 向量 a
    vecB = 0; // 向量 b
  for (let i = 0; i < origin.length; i++) {
    vecA += Math.pow(origin[i], 2);
    vecB += Math.pow(target[i], 2);
    product += origin[i] * target[i];
  }

  /**
   * 数据可能会出现溢出的可能，导致最后得到的数不完全准确
   * 这里简单判断下两个数是否相同，如果相同就没必要开方后又平方
   *
   * p.s. 最可靠的方式是用大数相乘，但是这样会使处理时间变长，但是其实我们图片对比没必要这么精确
   */
  if (vecA === vecB) {
    return product / vecA;
  }

  return product / (Math.sqrt(vecA) * Math.sqrt(vecB));
};

/**
 * 比对两张图片的差异
 * @param imageA
 * @param imageB
 * @returns 返回差异值，范围[0,1]，0为完全不一样，1为完全一样
 */
export const compareImageSimilarity = async function (
  imageA: ImageFile | ImageUrl,
  imageB: ImageFile | ImageUrl
) {
  const [fingerprintA, fingerprintB] = await Promise.all(
    [imageA, imageB].map(async (image) =>
      getFingerprint(
        await getImageData(
          Object.prototype.toString.call(image) === "[object String]"
            ? (image as ImageUrl)
            : await getImageUrl(image as ImageFile)
        )
      )
    )
  );
  return getCosineSimilarity(fingerprintA, fingerprintB);
};
复制代码

```