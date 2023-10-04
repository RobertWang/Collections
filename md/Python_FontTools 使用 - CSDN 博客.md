> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43411585/article/details/103484643)

#### 目录

*   *   *   *   [Font_Tools 的使用](#Font_Tools_1)
            *   *   [1、fontTools 使用总结](#1fontTools_5)
                *   [2、加载字体文件](#2_38)
                *   [3、保存为 xml 文件：](#3xml_42)
                *   [4、获取各节点名称，返回为列表：](#4_46)
                *   [5、获取 getGlyphOrder 节点的 name 值，返回为列表：](#5getGlyphOrdername_51)
                *   [6、获取 cmap 节点 code 与 name 值映射, 返回为字典：](#6cmapcodename__58)
                *   [7、获取 glyf 节点 TTGlyph 字体 xy 坐标信息：](#7glyfTTGlyphxy_63)
                *   [8、获取 glyf 节点 TTGlyph 字体 xMin,yMin,xMax,yMax 坐标信息：](#8glyfTTGlyphxMinyMinxMaxyMax_68)
                *   [9、获取 glyf 节点 TTGlyph 字体 on 信息（0 表示弧形 / 1 表示矩形）：](#9glyfTTGlyphon01_76)
                *   [10、 获取 GlyphOrder 节点 GlyphID 的 id 信息, 返回 int 型：](#10_GlyphOrderGlyphIDid_int_81)
                *   [11、 将 woff2 文件转成 ttf 文件并解析：](#11_woff2ttf_86)
                *   [12、 识别 woff.2/ttf/woff 的字体结果](#12_woff2ttfwoff_99)
                *   [13、 乱码字母数字转成图片](#13__143)
                *   [14、网页上的乱码字和 ttf 的 camp_code 的转换](#14ttfcamp_code_175)

##### Font_Tools 的使用

*   pip install fontTools
*   [FontCreator 工具下载](https://www.high-logic.com/font-editor/fontcreator)
*   [在线 FontEditor 工具](https://font.qqe2.com/)

###### 1、fontTools 使用总结

*   [woff 字体文件点击下载](https://vfile.meituan.net/colorstone/20e8627b1aa9e979c6a8e975a83575bf2272.woff) ，直接这个脚本就是完整的，具体看注释

```
from fontTools.ttLib import TTFont
# 加载字体文件：
font = TTFont('local_fonts.woff')

# 保存为xml文件：
font.saveXML('local_fonts.xml')

# 获取各节点名称，返回为列表
print(font.keys())  # ['GlyphOrder', 'head', 'hhea', 'maxp', 'OS/2', 'hmtx', 'cmap', 'loca', 'glyf', 'name', 'post', 'GSUB']

# 获取getGlyphOrder节点的name值，返回为列表
print(font.getGlyphOrder())  # ['glyph00000', 'x', 'uniF013', 'uniF4D4', 'uniEE40', 'uniF7E1', 'uniF34B', 'uniE1A0', 'uniF1BE', 'uniE91E', 'uniF16F', 'uniF724']
print(font.getGlyphNames())  # ['glyph00000', 'uniE1A0', 'uniE91E', 'uniEE40', 'uniF013', 'uniF16F', 'uniF1BE', 'uniF34B', 'uniF4D4', 'uniF724', 'uniF7E1', 'x']

# 获取cmap节点code与name值映射, 返回为字典
print(font.getBestCmap())  # {120: 'x', 57760: 'uniE1A0', 59678: 'uniE91E', 60992: 'uniEE40', 61459: 'uniF013', 61807: 'uniF16F', 61886: 'uniF1BE', 62283: 'uniF34B', 62676: 'uniF4D4', 63268: 'uniF724', 63457: 'uniF7E1'}

# 获取glyf节点TTGlyph字体xy坐标信息
print(font['glyf']['uniE1A0'].coordinates)  # GlyphCoordinates([(50, 335),(50, 468),(76, 544),(95, 638),(148, 676),(202, 710),(282, 710),(402, 710),(459, 617),(487, 574),(504, 501),(520, 437),(519, 335),(520, 271),(508, 166),(494, 126),(466, 46),(362, -39),(282, -49),(176, -35),(115, 37),(43, 121),(43, 335),(135, 335),(135, 154),(177, 95),(229, 35),(282, 35),(343, 35),(385, 107),(428, 155),(428, 339),(428, 515),(385, 576),(344, 635),(286, 635),(218, 635),(179, 583),(135, 506)])

# 获取glyf节点TTGlyph字体xMin,yMin,xMax,yMax坐标信息
print(font['glyf']['uniE1A0'].xMin, font['glyf']['uniE1A0'].yMin,
      font['glyf']['uniE1A0'].xMax, font['glyf']['uniE1A0'].yMax)  # 0 -49 521 711
      
# 获取glyf节点TTGlyph字体on信息
print(font['glyf']['uniE1A0'].flags)  # array('B', [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 0, 1, 0, 0, 1, 0, 1, 0, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0])

# 获取GlyphOrder节点GlyphID的id信息, 返回int型
print(font.getGlyphID('uniE1A0'))  # 7

```

###### 2、加载字体文件

```
 font = TTFont('local_fonts.woff')

```

###### 3、保存为 xml 文件：

```
font.saveXML('local_fonts.xml')

```

###### 4、获取各节点名称，返回为列表：

```
font.keys()
# ['GlyphOrder', 'head', 'hhea', 'maxp', 'OS/2', 'hmtx', 'cmap', 'loca', 'glyf', 'name', 'post', 'GSUB']

```

###### 5、获取 getGlyphOrder 节点的 name 值，返回为列表：

```
font.getGlyphOrder()
# ['glyph00000', 'x', 'uniF013', 'uniF4D4', 'uniEE40', 'uniF7E1', 'uniF34B', 'uniE1A0', 'uniF1BE', 'uniE91E', 'uniF16F', 'uniF724']
font.getGlyphNames()
# ['glyph00000', 'uniE1A0', 'uniE91E', 'uniEE40', 'uniF013', 'uniF16F', 'uniF1BE', 'uniF34B', 'uniF4D4', 'uniF724', 'uniF7E1', 'x']

```

###### 6、获取 cmap 节点 code 与 name 值映射, 返回为字典：

```
font.getBestCmap() 
# {120: 'x', 57760: 'uniE1A0', 59678: 'uniE91E', 60992: 'uniEE40', 61459: 'uniF013', 61807: 'uniF16F', 61886: 'uniF1BE', 62283: 'uniF34B', 62676: 'uniF4D4', 63268: 'uniF724', 63457: 'uniF7E1'}

```

###### 7、获取 glyf 节点 TTGlyph 字体 xy 坐标信息：

```
font['glyf']['uniE1A0'].coordinates
# GlyphCoordinates([(50, 335),(50, 468),(76, 544),(95, 638),(148, 676),(202, 710),(282, 710),(402, 710),(459, 617),(487, 574),(504, 501),(520, 437),(519, 335),(520, 271),(508, 166),(494, 126),(466, 46),(362, -39),(282, -49),(176, -35),(115, 37),(43, 121),(43, 335),(135, 335),(135, 154),(177, 95),(229, 35),(282, 35),(343, 35),(385, 107),(428, 155),(428, 339),(428, 515),(385, 576),(344, 635),(286, 635),(218, 635),(179, 583),(135, 506)])

```

###### 8、获取 glyf 节点 TTGlyph 字体 xMin,yMin,xMax,yMax 坐标信息：

```
font['glyf']['uniE1A0'].xMin
font['glyf']['uniE1A0'].yMin
font['glyf']['uniE1A0'].xMax
font['glyf']['uniE1A0'].yMax
# 0 -49 521 711

```

###### 9、获取 glyf 节点 TTGlyph 字体 on 信息（0 表示弧形 / 1 表示矩形）：

```
font['glyf']['uniE1A0'].flags
# array('B', [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 0, 1, 0, 0, 1, 0, 1, 0, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0])

```

###### 10、 获取 GlyphOrder 节点 GlyphID 的 id 信息, 返回 int 型：

```
font.getGlyphID('uniE1A0')
# 7

```

###### 11、 将 woff2 文件转成 ttf 文件并解析：

```
from fontTools.ttLib import TTFont
from fontTools.ttLib.woff2 import decompress


woff2_path = "./woff/704224.woff2"
ttf_path = './woff/704224.ttf'
xml_path = './woff/704224.xml'
decompress(woff2_path, ttf_path)  # 将woff2文件转成ttf文件
font = TTFont(ttf_path)
font.saveXML(xml_path)

```

###### 12、 识别 woff.2/ttf/woff 的字体结果

```
from PIL import ImageFont, Image, ImageDraw
from io import BytesIO
import ddddocr
from fontTools.ttLib import TTFont
from fontTools.ttLib.woff2 import decompress


# woff2_path = "./woff/704224.woff2"
# ttf_path = './woff/704224.ttf'
# xml_path = './woff/704224.xml'
# decompress(woff2_path, ttf_path)  # 将woff2文件转成ttf文件
# _font = TTFont(ttf_path)
# _font.saveXML(xml_path)


def font_to_img(_code, filename):
    """将字体画成图片"""
    img_size = 1024
    img = Image.new('1', (img_size, img_size), 255)
    draw = ImageDraw.Draw(img)
    font = ImageFont.truetype(filename, int(img_size * 0.7))
    txt = chr(_code)
    x, y = draw.textsize(txt, font=font)
    draw.text(((img_size - x) // 2, (img_size - y) // 2), txt, font=font, fill=0)
    return img


def identify_word(_ttf_path):
    """识别ttf字体结果"""
    font = TTFont(_ttf_path)
    ocr = ddddocr.DdddOcr()
    for cmap_code, glyph_name in font.getBestCmap().items():
        bytes_io = BytesIO()
        pil = font_to_img(cmap_code, _ttf_path)
        pil.save(bytes_io, format="PNG")
        word = ocr.classification(bytes_io.getvalue())  # 识别字体
        print(cmap_code, glyph_name, word)
        # with open(f"./img/{cmap_code}_{glyph_name}.png", "wb") as f:
        #     f.write(bytes_io.getvalue())
        


```

###### 13、 乱码字母数字转成图片

```
from PIL import ImageFont, Image, ImageDraw
from io import BytesIO
import ddddocr
from fontTools.ttLib import TTFont


def luan_word_to_img(text, _ttf_path, img_path='./word.png'):
    """将字体画成图片"""
    img_size = 1024
    img = Image.new('1', (img_size, img_size), 255)
    draw = ImageDraw.Draw(img)
    word_size = int(img_size * (1 / len(text)))
    font = ImageFont.truetype(_ttf_path, word_size)
    x, y = draw.textsize(text, font=font)
    draw.text(((img_size - x) // 2, (img_size - y) // 2), text, font=font, fill=0)
    # img.show()
    # img.save(img_path)
    bytes_io = BytesIO()
    img.save(bytes_io, format="PNG")
    word = ocr.classification(bytes_io.getvalue())  # 识别字体
    # print(text, word)
    return [text, word]


ocr = ddddocr.DdddOcr()
luan_word_to_img('┋', './woff/599427.ttf')
luan_word_to_img('㑁', './woff/7SourceHanSansCN-Normal-2500.ttf')
luan_word_to_img('㖉㒍', './woff/7SourceHanSansCN-Normal-2500.ttf')

```

###### 14、网页上的乱码字和 ttf 的 camp_code 的转换

*   `乱码字= chr(cmap_code) ; ord(乱码字）=cmap_code`
    
    ```
    from fontTools.ttLib import TTFont
    ttf_path = './woff/704224.ttf'
    _font = TTFont(ttf_path)
    for cmap_code, glyph_name in _font.getBestCmap().items():
        # print(glyph_name.replace("uni", r"\u").encode('utf-8').decode('unicode_escape'))
        print(f"cmap_code:{cmap_code}, chr(cmap_code):{chr(cmap_code)}, glyph_name: {glyph_name}")
    
    ```