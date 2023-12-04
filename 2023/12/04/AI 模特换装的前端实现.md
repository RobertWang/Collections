> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/EChbTBvDXADBL1sVcMnX2g)

> 本文作者为 360 奇舞团前端开发工程师

随着 AI 的火热发展，涌现了一些 AI 模特换装的前端工具（比如 weshop 网站），他们是怎么实现的呢？使用了什么技术呢？下文我们就来探索一下其实现原理。

总体的实现流程如下：我们将下图中的这个模特的图片，使用 Segment Anything Model 在后端分割图层，然后将分割后的图层 mask 信息返回给前端处理。在前端中选择需要保留的图层信息（如下图中的模特的衣服图层），然后将选中的图层信息交给后端中的 Stable Diffusion 处理。后端使用原始图片结合选中的图层蒙版图片结合图生图的功能，可以实现 weshop 等网站的模特换衣等功能。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/cAd6ObKOzEBUn4Csvnyia3RCaabLCXdySib5ia6icIH2z5v0hBACPmflOq5goBQrMymB2ITwbWIv8guQg7qz7G483g/640?wx_fmt=jpeg&from=appmsg)

本文先简单介绍一下使用 SAM 智能图层分割，然后主要介绍一下在前端中怎么对分割后的图层进行选择的处理流程。

### 使用 SAM 识别图层

首先我们需要对图层进行分割，在 SAM 出来之前，我们需要使用 PS 将模特的衣服选取出来，然后倒出衣服的模板，然后再使用其他工具进行替换。但是现在有了 SAM 后，我们可以对图片中的事物进去只能区分，获取各种物品的图层。

Segment Anything Model（SAM）是一种尖端的图像分割模型，可以进行快速分割，为图像分析任务提供无与伦比的多功能性。SAM 的先进设计使其能够在无需先验知识的情况下适应新的图像分布和任务，这一功能称为零样本传输。SAM 使任何人都可以在不依赖标记数据的情况下为其数据创建分段掩码。

要深入了解 Segment Anything 模型和 SA-1B 数据集，请访问 Segment Anything 网站（https://segment-anything.com/）并查看研究论文 Segment Anything（https://arxiv.org/abs/2304.02643）。

我们使用 SAM 进行图像分割，将一个图片中的物体分割成不同的部分。

```
def mask2rle(img):
    '''
    img: numpy array, 1 - mask, 0 - background
    Returns run length as string formated
    '''
    pixels = img.T.flatten()
    pixels = np.concatenate([[0], pixels, [0]])
    runs = np.where(pixels[1:] != pixels[:-1])[0] + 1
    runs[1::2] -= runs[::2]
    return ' '.join(str(x) for x in runs)


def trans_anns(anns):
    if len(anns) == 0:
        return
    sorted_anns = sorted(anns, key=(lambda x: x['area']), reverse=False)
    list = []
    index = 0
    # 对每个注释进行处理
    for ann in sorted_anns:
        bool_array = ann['segmentation']
        # 将boolean类型的数组转换为int类型
        int_array = bool_array.astype(int)
        # 转化为RLE格式
        rle = mask2rle(int_array)
        list.append({"index": index, "mask": rle})
        index += 1
    return list

image = cv2.imread('<your image path>')

import sys
sys.path.append('<your segment-anything link path>')
from segment_anything import sam_model_registry, SamAutomaticMaskGenerator, SamPredictor


# sam 模型路径
sam_checkpoint = '<your sam model path>'
# 根据下载的模型，设置对应的类型
model_type = "vit_h"

# device = "cuda"
sam = sam_model_registry[model_type](checkpoint=sam_checkpoint)
# sam.to(device=device)
mask_generator = SamAutomaticMaskGenerator(sam)
masks = mask_generator.generate(image)
# 处理sam返回的图层信息
mask_list = trans_anns(masks)

mask_obj = {
    "height": image.shape[0],
    "width": image.shape[1],
    "mask_list": mask_list
}

import json
print(json.dumps(mask_obj))


```

运行以上 python 代码之前，需要配置 sam 的 python 环境，具体的配置描述请查看 sam 的官方描述。

我们通过以上代码，将我们提供的图片，通过 SAM 处理后，返回图层分割数据。在 trans_anns 方法中，将图层按照 area 从小到大的顺序排序。遍历各个图层，将 boolean 类型的数组转换为 0 1 int 类型，然后对二维 numpy array 类型的 0 1 二进制 mask 图像转换为 RLE 格式。

RLE 是一种简单的无损数据压缩算法，通常用于表示连续的相同值的序列。RLE 编码的字符串通常用于在图像分割等任务中存储和传输二进制掩码信息，以便更有效地表示图像中的目标区域。并且方便数据压缩和传输。我们参照的这种编解码方式。也可以使用 coco RLE 的编解码方式。

将编码后的各图层信息存储到 list 中，就可以通过接口传输给前端处理了。

### 前端选择图层

下面这些是本文的重点，在前端将刚才解析后的 mask_list 信息展示，并可以通过交互选取需要保留的模版，并生成最终合并选取的 mask 生成一个需要保留的服装模版。

body 中的基本组件为

```
   <div id="layer-box" style=" width: 500px; height: 500px;position: relative">
        <img style="width: 100%; height: 100%; position: absolute" src="https://p0.ssl.qhimg.com/t01989f0d446bed3e58.jpg" />
    </div>
    <div id="save" @click="save" style="margin-top: 20px;margin-right: 20px; margin-left: 20px;">保存</div>
    <canvas id="mergedCanvas" style="border:1px solid #000;"></canvas>


```

id 为 layer-box 的 div 组件作为各个 mask 的父组件，用于查找和管理各个 mask 的隐藏和展示。其子组件中的第一个标签是展示原始的模特图片的。

id 为 save 的组件在点击时可以处理保存选中的各个 mask 为一个新的 mask 图片，用于处理图片合成。

id 为 mergedCanvas 的 canvas 是进行图片合成和展示合成后的图片的。

#### 解析 SAM 处理后的 mask_list 信息

```
        /**
         * rle格式图片信息转换为mask信息
         */
        function rle2mask(mask_rle, shape = [500, 500]) {
            /*
            mask_rle: run-length as string formatted (start length)
            shape: [width, height] of array to return
            Returns an array, 1 - mask, 0 - background
            */

            const s = mask_rle.split(" ");
            let starts = s.filter((_, index) => index % 2 === 0).map(Number);
            const lengths = s.filter((_, index) => index % 2 !== 0).map(Number);
            starts = starts.map(start => start - 1);
            const ends = starts.map((start, index) => start + lengths[index]);
            const img = new Array(shape[0] * shape[1]).fill(0);

            for (let i = 0; i < starts.length; i++) {
                for (let j = starts[i]; j < ends[i]; j++) {
                    img[j] = 1;
                }
            }

            // return transposeArray(img, shape);
            const transposed = new Array(shape[1]).fill(0).map(() => new Array(shape[0]).fill(0));
            for (let i = 0; i < shape[0]; i++) {
                for (let j = 0; j < shape[1]; j++) {
                    transposed[j][i] = img[i * shape[1] + j];
                }
            }
            return transposed;
        }


        /**
         * 转换mask图片信息，并设置mask的填充颜色
         */
        function transformMaskImage(item, _width, _height) {
            let canvas = document.createElement("canvas");
            let canvasContext = canvas.getContext("2d");
            canvas.width = _width;
            canvas.height = _height;
            let rgbaData = rle2mask(item.mask || '', [_width, _height])
            for (let y = 0; y < rgbaData.length; y++) {
                let row = rgbaData[y];
                for (let x = 0; x < row.length; x++) {
                    let dot = rgbaData[y][x];
                    if (1 === dot && canvasContext) {
                        // 值为1的点填充颜色
                        (canvasContext.fillStyle = "#4169eb"), canvasContext.fillRect(x, y, 1, 1);
                    }
                }
            }
            // canvas当前层的图片（base64格式）
            // matrix：上边生成的二维数组
            return { imageData: canvas.toDataURL("image/png"), matrix: rgbaData };
        }

        // 使用sam处理后的图层信息（rle编码后的，由于篇幅限制，已省略）
        const res = { "height": 500, "width": 500, "mask_list": [{ "index": 0, "mask": "109864 3 110361 7 110860 9 111359 10 111859 10 112359 10 112860 9 113360 10 113860 10 114360 10 114860 10 115360 10 115861 8" }, { "index": 1, "mask": "121910 2 122409 4 122908 6 123408 7 123907 8 124407 9 124907 9 125406 11 125905 12 126404 13 126905 12 127405 12 127906 12 128406 12 128907 11 129407 10 129908 8 130408 4" },......] }

        layers = res.mask_list.map((item) =>
            transformMaskImage(item, res.width, res.height)
        );


```

res 是 sam 处理后返回的图层信息（由于篇幅限制，已省略，详情请看 demo（https://github.com/yuhao1128/AI-model-mask-select-demo/blob/main/index.html）中的数据）。遍历 mask_list，使用 canvas 保存各个 mask 的信息。由于前面 sam 处理后的 mask_list 是经过压缩编码的，所以在 rle2mask 方法中对 rle 编码后的数据解码为 0/1 二维数组的格式。rle2mask 中的解码方式请参考这种解码（https://www.kaggle.com/code/pestipeti/decoding-rle-masks）方式。

然后遍历二维数组，将值为 1 的点填充颜色，此处是填充的 rgba 为 "#4169eb" 的颜色，可以根据需要自己修改为其他的颜色。此处填充的颜色会在下文中鼠标移动到 mask 上面时，在 mask 展示的时候呈现此颜色。

最后在 layers 中存储各个 mask 的 base64 格式的图片信息和二维数组信息。

#### 将各个 mask 添加到图层

```
        const box = document.querySelector("#layer-box");
        const baseStyle = "width:100%;height:100%;position: absolute;";
        //将各个mask添加为layer-box的子组件，并隐藏mask的展示
        layers.forEach((ele) => {
            const image = document.createElement("img");
            image.src = ele.imageData;
            image.style = `${baseStyle}opacity:0`;
            image.className = "layer";
            box.append(image);
        });


```

将各个 mask 添加的图片添加为 layer-box 组件的子组件，并且设置 opacity 为 0，先隐藏这些 mask 的展示，在下文会监听鼠标的位置，通过设置 mask 的 opacity 属性来展示 mask。

#### 监听鼠标的位置和点击

```
        // 鼠标移入mask组件的区域时，展示mask
        box.addEventListener("mousemove", (e) => {
            const { clientX, clientY } = e;
            const X = box.getBoundingClientRect().left + document.body.scrollLeft;
            const Y = box.getBoundingClientRect().top + document.body.scrollTop;
            const x = parseInt(res.width * (clientX - X) / box.getBoundingClientRect().width)
            const y = parseInt(res.height * (clientY - Y) / box.getBoundingClientRect().height)
            const allLayers = box.querySelectorAll(".layer");
            const index = layers.findIndex((item) => item.matrix?.[y]?.[x]);
            allLayers.forEach((ele, i) => {
                if (i === index) {
                    ele.style = `${baseStyle}opacity:0.7`;
                } else {
                    // 已经选中的不需要隐藏
                    if (selectedIndexList.indexOf(i) === -1) {
                        ele.style = `${baseStyle}opacity:0`;
                    }
                }
            });
        });

        // 鼠标移出mask组件的区域时，隐藏mask
        box.addEventListener("mouseout", (e) => {
            console.log('mouseout selectedIndexList', selectedIndexList);
            const allLayers = box.querySelectorAll(".layer");
            allLayers.forEach((ele, i) => {
                // 只有选中的才会展示
                if (selectedIndexList.indexOf(i) > -1) {
                    ele.style = `${baseStyle}opacity:0.7`;
                } else {
                    ele.style = `${baseStyle}opacity:0`;
                }
            });
        });

        // 用户点击时，保存用户选中的mask的index
        box.addEventListener("mousedown", (e) => {
            const { clientX, clientY } = e;
            const X = box.getBoundingClientRect().left + document.body.scrollLeft;
            const Y = box.getBoundingClientRect().top + document.body.scrollTop;
            const x = parseInt(res.width * (clientX - X) / box.getBoundingClientRect().width)
            const y = parseInt(res.height * (clientY - Y) / box.getBoundingClientRect().height)
            const index = layers.findIndex((item) => item.matrix?.[y]?.[x]);
            if (selectedIndexList.indexOf(index) === -1) {
                //保存点击选中的元素index
                selectedIndexList.push(index)
            }
        });


```

box 就是上文的 layer-box，是各个 mask 的父组件。layer-box 监听鼠标的 move 事件和 click 事件，当 move 到对应的 mask 上时，将 mask 展示，移除 mask 时，隐藏 mask。mask 在 list 中是从小到大的顺序，所以遍历匹配 mask 时，会优先匹配面积小的组件，方便灵活选择。当点击 mask 的位置时，保存 mask 在 list 中的 index 到 selectedIndexList 中，方便后续导出保存选择，并高亮展示选中的 mask。

#### 选中的 mask 合成图片

```
        // 存储各个图层图片信息
        let layers = []
        // 选择layer的index
        const selectedIndexList = []


        // 点击保存
        document.getElementById('save').onclick = function () {
            const images = [];
            selectedIndexList.forEach(index => {
                images.push(layers[index].imageData)
            })
            drawing(images)
        }

        /**
           * 图片合成
           */
        function drawing(images) {
            const canvas = document.getElementById("mergedCanvas");
            canvas.width = 500;  // 设置canvas宽
            canvas.height = 500; // 设置canvas高
            const ctx = canvas.getContext("2d");
            let loadedImages = 0;
            images.forEach(function (src) {
                const img = new Image();
                img.src = src;
                img.onload = function () {
                    loadedImages++;
                    // 绘制每张图片到 canvas 上
                    ctx.drawImage(img, 0, 0);
                    // 如果所有图片都加载完成，保存合并后的图片
                    if (loadedImages === images.length) {
                         // 获取图片的像素数据
                        const imageData = ctx.getImageData(0, 0, img.width, img.height);
                        const data = imageData.data;
                        // 转换为黑白效果
                        for (let i = 0; i < data.length; i += 4) {
                          // 将 R、G、B 设置为0
                          data[i] = 0;
                          data[i + 1] = 0;
                          data[i + 2] = 0;
                        }
                        // 将修改后的数据放回 canvas
                        ctx.putImageData(imageData, 0, 0);
                        // 导出为 base64 图片
                        const mergedImageBase64 = canvas.toDataURL("image/png");
                        // 如果需要，你可以将mergedImageBase64图片用于其他操作，比如发送到服务器
                    }
                };
            });
        }


```

当选择完成后，可以点击 “保存” 按钮，将选择的 mask 使用 canvas 生成一个合并后的图片。此处已将合成后的图片转换为黑白蒙版照片，之后可以使用这个合并后的图片进行后续的处理。

根据选中的图层，点击保存后，生成的模板如下图所示。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/cAd6ObKOzEBUn4Csvnyia3RCaabLCXdySnVfnKFu2v62icFuC4Ryql6hGdRAdkJQAYiaibic95jRUaLaibmXfQESrveA/640?wx_fmt=png&from=appmsg)

预览效果（https://yuhao1128.github.io/AI-model-mask-select-demo/）、代码详情（https://github.com/yuhao1128/AI-model-mask-select-demo/blob/main/index.html）

### 使用 Stable Diffusion 进行后续的处理

由于篇幅的限制，并且这部分网络上以及有很多的介绍资料，就不再本文中进行介绍了，可以参考这篇文章（https://www.uisdc.com/stable-diffusion-24）的介绍尝试体验一下在本地中使用 Stable Diffusion 的图生图的「重绘蒙版」来进行模特的重新绘制。

也可以在后端部署 Stable Diffusion 服务中处理模特换装。将前面的模特原图以及生成的蒙版图片，以及其他的 SD 的图生图功能的参数传给后端的 SD 服务处理。

除了模特换装的功能，上面的流程还可以应用到物品换背景的功能中。其他的一些智能抠图，智能替换的功能都可以扩展上面的处理流程来实现。

参考链接：

https://github.com/facebookresearch/segment-anything

https://juejin.cn/post/7248903246970503223#heading-2

https://www.uisdc.com/stable-diffusion-24

- END -