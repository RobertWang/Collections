> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [evanhahn.com](https://evanhahn.com/worlds-smallest-png/)

> It takes 67 bytes to make one black pixel. How does it work?制作一个黑色像素需要 67 个字节。它是如何工作的？

by [Evan Hahn](https://evanhahn.com/) 埃文·哈恩（Evan Hahn）

, updated Jan 21, 2024 (originally posted Jan 4, 2024)  
， 更新 Jan 21， 2024 （最初发布于 Jan 4， 2024）

The smallest PNG file is 67 bytes. It’s a single black pixel. Here’s what it looks like, zoomed in 200×:  
最小的 PNG 文件是 67 字节。这是一个黑色像素。这是放大 200 ×的样子：

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABAQAAAAA3bvkkAAAACklEQVR4AWNgAAAAAgABc3UBGAAAAABJRU5ErkJggg==)

Wow, what a beauty. 哇，真是太美了。

This file has four sections:  
此文件包含四个部分：

1.  The PNG signature, the same for every PNG: 8 bytes  
    PNG 签名，每个 PNG 相同：8 个字节
2.  The image’s metadata, which includes its dimensions: 25 bytes  
    图像的元数据，包括其尺寸：25 字节
3.  The image’s pixel data: 22 bytes  
    图像的像素数据：22 字节
4.  An “end of image” marker: 12 bytes  
    “图像结束”标记：12 字节

The rest of this post describes this file in more detail and tries to explain how PNGs work along the way.  
本文的其余部分将更详细地描述此文件，并尝试解释 PNG 在此过程中的工作方式。

There’s a big twist at the end, if that excites you. But I hope you’re just excited to learn about PNGs.  
最后有一个很大的转折，如果这让你兴奋的话。但我希望你对了解 PNG 感到兴奋。

Part 1: the PNG signature  
第 1 部分：PNG 签名
-----------------------------------------

Every single PNG, including this one, starts with the same 8 bytes. Encoded in hex, those bytes are:  
每个 PNG，包括这个，都以相同的 8 个字节开头。以十六进制编码，这些字节是：

```
89 50 4E 47 0D 0A 1A 0A


```

This is called the [**PNG signature**](https://www.w3.org/TR/2022/WD-png-3-20221025/#5PNG-file-signature). Try doing a hex dump on any PNG and you’ll see that it starts with these bytes.  
这称为 PNG 签名。尝试对任何 PNG 进行十六进制转储，您会看到它以这些字节开头。

PNG decoders use the signature to ensure that they’re reading a PNG image. Typically, they reject the file if it doesn’t start with the signature. Data can get corrupted in various ways (ever had a file with the wrong extension?) and this helps address that.[1](#fn:1)  
PNG 解码器使用签名来确保他们正在读取 PNG 图像。通常，如果文件不以签名开头，他们会拒绝该文件。数据可能会以各种方式损坏（曾经有过扩展名错误的文件吗？），这有助于解决这个问题。 [1](#fn:1)

Fun fact: if you decode these bytes as ASCII, you’ll see the letters “PNG” in there:  
有趣的事实：如果你把这些字节解码为ASCII，你会看到字母“PNG”在里面：

```
.PNG....


```

So that’s the first 8 bytes. One part done! Here’s our “checklist”:  
这是前 8 个字节。完成一部分！以下是我们的“清单”：

1.  ~PNG signature PNG 签名~
2.  Image metadata chunk 图像元数据块
3.  Pixel data chunk 像素数据块
4.  “End of image” chunk “图像结束”块

What about the rest? 其余的呢？

The next part of the PNG is the image metadata, which is one of several **chunks**. What’s a chunk?  
PNG 的下一部分是图像元数据，它是几个块之一。什么是块？

### Quick intro to chunks 区块快速介绍

Other than the PNG signature at the start, PNGs are made up of chunks.  
除了开头的 PNG 签名外，PNG 由块组成。

Chunks have two logical pieces: a **type** and some **data bytes**. Types are things like “image header” or “text metadata”. The data depends on the type—the text metadata chunk is encoded differently from the image header chunk.  
块有两个逻辑部分：一个类型和一些数据字节。类型是“图像标题”或“文本元数据”之类的东西。数据取决于类型 - 文本元数据块的编码方式与图像标头块的编码方式不同。

These logical pieces are encoded with [four fields](https://www.w3.org/TR/2022/WD-png-3-20221025/#5Chunk-layout). These fields are always in the same order for every chunk. They are:  
这些逻辑片段使用四个字段进行编码。对于每个块，这些字段始终以相同的顺序排列。他们是：

1.  **Length**: the number of bytes in the chunk’s data field (field #3 below). Encoded as a 4-byte integer.[2](#fn:2)  
    长度：块数据字段中的字节数（下面的字段 #3）。编码为 4 字节整数。 [2](#fn:2)
2.  **Chunk type**: the type of chunk this is. There are lots of different chunk types. Encoded as a 4-byte ASCII string, such as “IHDR” for “image header” or “tEXt” for “text metadata”.  
    Chunk type：这是块的类型。有许多不同的块类型。编码为 4 字节 ASCII 字符串，例如“图像标头”的“IHDR”或“文本元数据”的“tEXt”。
3.  **Data**: the data for the chunk. See the “length” field for how many bytes there will be. Varies based on the chunk type. For example, the IHDR chunk encodes the image’s dimensions. May be empty, but usually isn’t.  
    数据：块的数据。请参阅“长度”字段，了解将有多少字节。因区块类型而异。例如，IHDR 区块对图像的尺寸进行编码。可能是空的，但通常不是。
4.  **Checksum**: a checksum for the rest of the chunk, to make sure no data was corrupted. 4 bytes.  
    校验和：块其余部分的校验和，以确保没有数据被损坏。4 字节。

As you can see, each chunk is a minimum of 12 bytes long (4 for the length, 4 for the type, and 4 for the checksum).  
如您所见，每个块的长度至少为 12 个字节（长度为 4 个字节，类型为 4 个字节，校验和为 4 个字节）。

Note that the “length” field is the size of the “data” field, _not_ the entire chunk. If you want to know the whole size of the chunk, just add 12—4 bytes for the length, 4 bytes for the type, and 4 bytes for the checksum.  
请注意，“length”字段是“data”字段的大小，而不是整个块的大小。如果您想知道块的整个大小，只需添加 12 个字节 - 长度为 4 个字节，类型为 4 个字节，校验和为 4 个字节。

You have some wiggle room but chunks have a specific order. For example, the image metadata chunk has to appear before the pixel data chunk. Once you reach the “image is done” chunk, the PNG is done.  
你有一些回旋余地，但块有特定的顺序。例如，图像元数据块必须出现在像素数据块之前。到达“图像完成”块后，PNG 就完成了。

Our tiny PNG will have just three of these chunks.  
我们小小的 PNG 将只有其中的三个块。

The first chunk of every PNG, including ours, is of type **IHDR**, short for [“image header”](https://www.w3.org/TR/2022/WD-png-3-20221025/#11IHDR).  
每个 PNG（包括我们的 PNG）的第一个块都是 IHDR 类型，即“图像标题”的缩写。

Each chunk starts with the **length** of the data in that chunk.  
每个块都以该块中数据的长度开头。

The IHDR chunk always has 13 bytes of associated data, as we’ll see in a moment. 13 is `0D` in hex, which gets encoded like this:  
IHDR 块始终具有 13 个字节的关联数据，我们稍后将看到。13 是 `0D` 十六进制的，编码如下：

```
00 00 00 0D


```

The **chunk type** is next. This is another four bytes. “IHDR” is encoded as:  
接下来是块类型。这是另外四个字节。“IHDR”编码为：

```
49 48 44 52


```

This is just ASCII encoding. Chunk types are made up of ASCII letters. [The capitalization of each letter is significant.](https://www.w3.org/TR/2022/WD-png-3-20221025/#5Chunk-naming-conventions) For example, the first letter is capitalized which means it’s a required chunk.  
这只是 ASCII 编码。块类型由 ASCII 字母组成。每个字母的大写都很重要。例如，第一个字母是大写的，这意味着它是必需的块。

Next, the chunk’s **data**. IHDR’s data happens to be 13 total bytes, arranged as follows:  
接下来是块的数据。IHDR 的数据恰好总共有 13 个字节，排列如下：

*   The first eight bytes encode the image’s width and height. Because this is a 1×1 image, that’s encoded as `00 00 00 01 00 00 00 01`.  
    前 8 个字节对图像的宽度和高度进行编码。因为这是一个 1×1 图像，所以它被编码为 `00 00 00 01 00 00 00 01` .
    
*   The next two bytes are the [bit depth](https://www.w3.org/TR/2022/WD-png-3-20221025/#3bitDepth) and [color type](https://www.w3.org/TR/2022/WD-png-3-20221025/#6Colour-values).  
    接下来的两个字节是位深度和颜色类型。
    
    These values are probably the most confusing part of this PNG.  
    这些值可能是此 PNG 中最令人困惑的部分。
    
    There are five possible color types. Our image is black-and-white so we use the “greyscale” color type (encoded as `00`). If our image had color, we might use the “truecolor” type (encoded with `02`). There are three other color types which we don’t need today, but you can [read more about them in the PNG specification](https://www.w3.org/TR/2022/WD-png-3-20221025/#4Concepts.PNGImage).  
    有五种可能的颜色类型。我们的图像是黑白的，因此我们使用“灰度”颜色类型（编码为 `00` ）。如果我们的图像有颜色，我们可能会使用“truecolor”类型（用 `02` 编码）。我们今天不需要其他三种颜色类型，但您可以在 PNG 规范中阅读有关它们的更多信息。
    
    Once you’ve picked a color type, you need to pick a bit depth. The bit depth depends on the color type, but usually means the number of bits per color channel in an image. For example, hex colors like `#FE9802` have a bit depth of eight—eight bits for red, eight bits for green, and eight bits for blue. Our all-black image doesn’t need all that…we only need _one_ bit! The pixel is either completely black (`0`) or completely white (`1`)—in our case, it’s completely black.  
    一旦你选择了颜色类型，你需要选择一点深度。位深度取决于颜色类型，但通常是指图像中每个颜色通道的位数。例如，十六进制颜色 `#FE9802` 的位深度为 8 位，红色为 8 位，绿色为 8 位，蓝色为 8 位。我们的全黑图像不需要所有这些......我们只需要一点！像素要么是全黑 （ `0` ） 要么是全白 （ `1` ），在我们的例子中是完全黑的。
    
    If we picked a more “expressive” color type and bit depth, we could make the same 1×1 image visually, but the file could be bigger because there could be more bits per pixel that we don’t actually need. For example, if we used the “truecolor” type and 16 bits per channel, each pixel would require 48 bits instead of just one—not necessary to encode “completely black”.  
    如果我们选择一种更具“表现力”的颜色类型和位深度，我们可以在视觉上制作相同的 1×1 图像，但文件可能会更大，因为每个像素可能有更多的位，而我们实际上并不需要。例如，如果我们使用“真彩色”类型和每通道 16 位，则每个像素将需要 48 位，而不仅仅是一个——编码“全黑”不是必需的。
    
    With bit depth of 1 and a color type of 0, we encode these two values with `00 01`.  
    当位深度为 1 且颜色类型为 0 时，我们用 `00 01` 编码这两个值。
    
*   The next byte is the [compression method](https://www.w3.org/TR/2022/WD-png-3-20221025/#10CompressionCM0). All PNGs set this to `00` for now. This is here just in case they want to add another compression method later. As far as I know, nobody has.  
    下一个字节是压缩方法。所有 PNG 都暂时将其设置为 `00` 。这是为了以防万一他们以后想添加另一种压缩方法。据我所知，没有人。
    
*   Same story for the [filter method](https://www.w3.org/TR/2022/WD-png-3-20221025/#3filter). It’s always `00`.  
    过滤方法也是如此。它总是 `00` .
    
*   The last part of the chunk’s data is the [interlace method](https://www.w3.org/TR/2022/WD-png-3-20221025/#8InterlaceMethods). PNGs support progressive decoding which allows images to be partly rendered as they download. We aren’t going to use that feature so we set this to `00`.  
    块数据的最后一部分是隔行扫描方法。PNG 支持渐进式解码，允许在下载时部分渲染图像。我们不打算使用该功能，因此我们将其设置为 `00` 。
    

Finally, every chunk ends with a four-byte **checksum**. It uses [a common checksum function](https://www.w3.org/TR/2022/WD-png-3-20221025/#5CRC-algorithm) called CRC32, and uses the rest of the chunk as an input. Computing that checksum gives us the following bytes:  
最后，每个块都以一个四字节的校验和结尾。它使用称为 CRC32 的通用校验和函数，并使用块的其余部分作为输入。计算该校验和可得到以下字节：

```
37 6E F9 24


```

All together, here’s the whole IHDR chunk:  
总而言之，这是整个 IHDR 块：

<table data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><thead data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">Bytes&nbsp;字节</th><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">What?&nbsp;什么？</th></tr></thead><tbody data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00 00 00 0D</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">data length of 13 bytes<br>数据长度为 13 字节</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">49 48 44 52</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">“IHDR” as ASCII&nbsp;“IHDR”作为 ASCII</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00 00 00 01</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">width&nbsp;宽度</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00 00 00 01</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">height&nbsp;高度</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">01</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">bit depth&nbsp;位深度</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">color type&nbsp;颜色类型</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">compression method&nbsp;压缩方式</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">filter method&nbsp;过滤方式</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">interlace method&nbsp;隔行法</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">37 6E F9 24</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">checksum&nbsp;校验和</td></tr></tbody></table>

So that’s our first chunk! Let’s take another look at our checklist:  
这就是我们的第一块！让我们再看一下我们的清单：

1.  ~PNG signature PNG 签名~
2.  ~Image metadata chunk 图像元数据块~
3.  Pixel data chunk 像素数据块
4.  “End of image” chunk “图像结束”块

Two more chunks to go—pixel data is next.  
还有两个块要处理 - 接下来是像素数据。

Part 3: pixel data chunk  
第 3 部分：像素数据块
---------------------------------------

Our next chunk is **IDAT**, short for [“image data”](https://www.w3.org/TR/2022/WD-png-3-20221025/#11IDAT). This is where the actual pixels are encoded…or just one pixel, in our case.  
我们的下一个块是 IDAT，即“图像数据”的缩写。这是实际像素编码的地方......或者只有一个像素，在我们的例子中。

Remember that each chunk has four parts: the data’s _length_, the _chunk type_, the _data_, and a _checksum_.  
请记住，每个块都有四个部分：数据的长度、块类型、数据和校验和。

This chunk will have 10 bytes of data. We’ll talk about _what_ that data is shortly, but I promise it’s 10 bytes. Let’s encode that length:  
此块将包含 10 个字节的数据。我们稍后会讨论这些数据是什么，但我保证它是 10 个字节。让我们对这个长度进行编码：

```
00 00 00 0A


```

Next, let’s encode “IDAT” for the chunk type:  
接下来，让我们对块类型进行编码“IDAT”：

```
49 44 41 54


```

Again, this is just ASCII, and I’m showing the hex-encoded values.  
同样，这只是 ASCII，我显示的是十六进制编码值。

Now for the interesting part: the image data.  
现在是有趣的部分：图像数据。

### First step: uncompressed pixels  
第一步：未压缩的像素

Image data is encoded in a series of [“scanlines”](https://www.w3.org/TR/2022/WD-png-3-20221025/#3scanline), and then compressed.  
图像数据被编码为一系列“扫描线”，然后进行压缩。

A scanline represents a horizontal line of pixels. For example, a 123×456 image has 456 scanlines. In our case, we have just one scanline, because our image is just one pixel tall.  
扫描线表示像素的水平线。例如，一张 123×456 的图像有 456 条扫描线。在我们的例子中，我们只有一条扫描线，因为我们的图像只有 1 个像素高。

Scanlines start with something called a [filter type](https://www.w3.org/TR/2022/WD-png-3-20221025/#9Filter-types) which can improve compression, depending on your image. Our image is so small that this is irrelevant, so we use filter type 0, or “None”.[3](#fn:3)  
扫描线从一种称为滤镜类型的东西开始，它可以提高压缩率，具体取决于您的图像。我们的图像太小了，这无关紧要，所以我们使用过滤器类型 0，或“无”。 [3](#fn:3)

After the filter type, each pixel is encoded with one or more bits, depending on the bit depth. In our case, we just need one bit per pixel—recall that we have a bit depth of 1; all black or all white.  
在筛选器类型之后，每个像素都使用一个或多个位进行编码，具体取决于位深度。在我们的例子中，我们只需要每个像素一个位——回想一下，我们的位深度为 1;全黑或全白。

If your pixel data doesn’t line up with a byte boundary—in other words, if it’s not a multiple of 8 bits—you pad the end of your scanline with zeroes. That’s true in our case, so we add seven padding bits to fill out a byte.  
如果像素数据与字节边界不一致（换句话说，如果它不是 8 位的倍数），则在扫描线的末端填充零。在我们的例子中是正确的，所以我们添加七个填充位来填充一个字节。

Putting that together (a zero byte to start the scanline, the single zero bit, and seven zero padding bits), our single scanline is:  
将它们放在一起（一个用于启动扫描线的零字节、单个零位和七个零填充位），我们的单个扫描线是：

```
00 00


```

Now it’s time to “compress” the data.  
现在是时候“压缩”数据了。

### Second step: “compression”  
第二步：“压缩”

Next, we compress the scanline data…well, not quite.  
接下来，我们压缩扫描线数据...嗯，不完全是。

More accurately, we run it through a compression algorithm. Most of the time, compression algorithms produce smaller outputs—that’s the whole point! But sometimes, “compressing” tiny inputs actually produces _bigger_ outputs because of some small overhead. Unfortunately for us, that’s what happens here. But the PNG file format makes us do it.  
更准确地说，我们通过压缩算法运行它。大多数时候，压缩算法产生的输出更小——这就是重点！但有时，“压缩”微小的输入实际上会产生更大的输出，因为开销很小。对我们来说不幸的是，这就是这里发生的事情。但是PNG文件格式使我们这样做。

PNG image data is encoded in [the zlib format](https://www.rfc-editor.org/rfc/rfc1950) using the [DEFLATE compression algorithm](https://zlib.net/feldspar.html). DEFLATE is also used with [gzip](https://en.wikipedia.org/wiki/Gzip) and [ZIP](https://en.wikipedia.org/wiki/ZIP_(file_format)), two very popular compression formats.  
PNG 图像数据使用 DEFLATE 压缩算法以 zlib 格式编码。DEFLATE 还与 gzip 和 ZIP 一起使用，这是两种非常流行的压缩格式。

I won’t go in depth on DEFLATE here (in part because I am not an expert[4](#fn:4)), but here’s what our chunk’s data contains:  
我不会在这里深入讨论 DEFLATE（部分原因是我不是专家 [4](#fn:4) ），但以下是我们块的数据包含的内容：

1.  The zlib header: 2 bytes  
    zlib 标头：2 个字节
2.  One compressed DEFLATE block that encodes two literal zeroes[5](#fn:5): 4 bytes  
    一个压缩的 DEFLATE 块，编码两个文字零 [5](#fn:5) ：4 字节
3.  The zlib checksum (this is separate from the PNG chunk checksum!): 4 bytes  
    zlib 校验和（这与 PNG 块校验和是分开的！）：4 个字节

For more on how DEFLATE works, check out [“An Explanation of the DEFLATE Algorithm”](https://zlib.net/feldspar.html). I also recommend [infgen](https://github.com/madler/infgen/), a useful tool for inspecting DEFLATE streams.  
有关 DEFLATE 工作原理的更多信息，请查看“DEFLATE 算法的解释”。我还推荐 infgen，这是一个用于检查 DEFLATE 流的有用工具。

All together, here are the ten data bytes:  
总而言之，这里是十个数据字节：

```
78 01 63 60 00 00 00 02 00 01


```

Again, unfortunate that we had to run our two-byte scanline through an algorithm that made it _five times bigger_, but PNG makes us do it!  
同样，不幸的是，我们不得不通过一种算法来运行我们的双字节扫描线，使其大五倍，但 PNG 让我们做到了！

With that, we can compute the PNG’s checksum field and finish off the chunk.  
有了它，我们可以计算 PNG 的校验和字段并完成块。

<table data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><thead data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">Bytes&nbsp;字节</th><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">What?&nbsp;什么？</th></tr></thead><tbody data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00 00 00 0A</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">data length of 10 bytes<br>数据长度为 10 字节</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">49 44 41 54</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">“IDAT” as ASCII&nbsp;“IDAT”作为 ASCII</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">78 01</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">zlib header&nbsp;zlib 标头</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">63 60 00 00</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">“compressed” DEFLATE block<br>“压缩”DEFLATE 块</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00 02 00 01</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">zlib checksum&nbsp;zlib 校验和</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">73 75 01 18</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">chunk checksum&nbsp;块校验和</td></tr></tbody></table>

Just one more chunk to go! Taking a final look at our checklist before everything is crossed off:  
再多一大块！在划掉所有内容之前，最后看一下我们的清单：

1.  ~PNG signature PNG 签名~
2.  ~Image metadata chunk 图像元数据块~
3.  ~Pixel data chunk 像素数据块~
4.  “End of image” chunk “图像结束”块

Let’s finish this up. 让我们完成这个。

Part 4: the end 第 4 部分：结束
-------------------------

Poetically, PNGs end like they begin: with a small number of constant bytes.  
从诗意上讲，PNG 的结尾就像它们开始的那样：使用少量常量字节。

**IEND** is the final chunk, short for [“image trailer”](https://www.w3.org/TR/2022/WD-png-3-20221025/#11IEND).  
IEND 是最后一个块，是“图像预告片”的缩写。

The zero length is encoded with 4 zeroes:  
零长度用 4 个零编码：

```
00 00 00 00


```

“IEND” is then encoded:  
然后对“IEND”进行编码：

```
49 45 4E 44


```

There’s no data in IEND chunks, so we just move onto the checksum. Because everything else in the chunk is constant, this checksum is always the same:  
IEND 块中没有数据，因此我们只需转到校验和即可。由于块中的所有其他内容都是常量，因此此校验和始终相同：

```
AE 42 60 82


```

Here’s the whole trailer chunk:  
这是整个预告片部分：

<table data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><thead data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">Bytes&nbsp;字节</th><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">What?&nbsp;什么？</th></tr></thead><tbody data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00 00 00 00</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">data length of 0&nbsp;数据长度为 0</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">49 45 4E 44</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">“IEND” as ASCII&nbsp;“IEND”作为 ASCII</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">AE 42 60 82</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">checksum&nbsp;校验和</td></tr></tbody></table>

And our PNG is done!  
我们的PNG完成了！

Admiring our work 欣赏我们的工作
-------------------------

Here it is one more time, scaled up 200×:  
这是又一次，放大了 200×：

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABAQAAAAA3bvkkAAAACklEQVR4AWNgAAAAAgABc3UBGAAAAABJRU5ErkJggg==)

Beautiful. It starts with the classic PNG signature, follows up with a bit of metadata, “compresses” the pixel data, and signs off with an empty chunk.  
美丽。它从经典的 PNG 签名开始，然后是一些元数据，“压缩”像素数据，然后用一个空块签字。

And that’s the world’s smallest PNG!  
这是世界上最小的巴布亚新几内亚！

…or is it? ...或者是吗？

The twist: there are lots of champions  
转折点：有很多冠军
--------------------------------------------------

Throughout this post, I’ve said that this is the world’s smallest PNG. But that’s not quite true: it’s _tied_ for first. There are several “world’s smallest PNGs”!  
在整篇文章中，我都说过这是世界上最小的 PNG。但这并不完全正确：它并列第一。有几个“世界上最小的PNG”！

_As long as we encode all pixel data in a single byte, we can tie for the world’s smallest PNG.  
只要我们将所有像素数据编码在一个字节中，我们就可以并列成为世界上最小的 PNG。_

For example, you could encode this 8×1 black image, which is also 67 bytes:  
例如，您可以对这个 8×1 黑色图像进行编码，它也是 67 个字节：

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAgAAAABAQAAAADLe9LuAAAACklEQVR4AWNgAAAAAgABc3UBGAAAAABJRU5ErkJggg==)

This works because we use all eight bits are used to encode pixel data.  
这是有效的，因为我们使用所有八个位来编码像素数据。

With our 1×1 image, recall that seven bits were effectively “wasted” on padding. Here’s basically what happened:  
在我们的 1×1 图像中，回想一下，在填充上有效地“浪费”了 7 位。基本上发生的事情是这样的：

<table data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><thead data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">Bits&nbsp;位</th><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">What?&nbsp;什么？</th></tr></thead><tbody data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">0</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">a black pixel&nbsp;黑色像素</td></tr><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">0000000</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">padding&nbsp;填充</td></tr></tbody></table>

An 8×1 image can encode eight black pixels like so:  
一个 8×1 图像可以编码 8 个黑色像素，如下所示：

<table data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><thead data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">Bits&nbsp;位</th><th data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">What?&nbsp;什么？</th></tr></thead><tbody data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><tr data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a"><code data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a">00000000</code></td><td data-immersive-translate-walked="ced091c1-7b45-4a2b-bec8-b4cc7506e26a" data-immersive-translate-paragraph="1">eight black pixels&nbsp;8 个黑色像素</td></tr></tbody></table>

Instead of adding more pixels, you could also add more color resolution. Many grey colors can be encoded in a single byte, letting us tie for first. For example, this 1×1 grey pixel is also 67 bytes:  
除了添加更多像素外，您还可以添加更多颜色分辨率。许多灰色可以在一个字节中编码，让我们并列第一。例如，这个 1×1 灰色像素也是 67 字节：

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAAAAAA6fptVAAAACklEQVR4AWOwBQAAPwA+Eq7IEAAAAABJRU5ErkJggg==)

Again, this “uses up” the whole byte we have available, unlike our 1×1 image.  
同样，这“用完”了我们可用的整个字节，这与我们的 1×1 图像不同。

For more on this, my former coworker Jordan Rose published [“The Biggest Smallest PNG”](https://belkadan.com/blog/2024/01/The-Biggest-Smallest-PNG/) in response to this post. It shows the biggest 67-byte PNG: a 1×2064 black line.  
为了了解更多信息，我的前同事乔丹·罗斯（Jordan Rose）发表了“最大最小的巴布亚新几内亚”（The Biggest Minimal PNG）来回应这篇文章。它显示了最大的 67 字节 PNG：一条 1×2064 的黑线。

Summary 总结
----------

PNGs start with a “signature”. The rest of the file is made up of chunks. Each chunk has a length, type, data, and checksum. Some chunks are always required, like the image header (IHDR) chunk.  
PNG 以“签名”开头。文件的其余部分由块组成。每个块都有长度、类型、数据和校验和。某些块始终是必需的，例如映像标头 （IHDR） 块。

The smallest PNGs use the minimum number of chunks and the smallest possible data.  
最小的 PNG 使用最少的块数和尽可能小的数据。

Our PNGs are made up of four parts:  
我们的PNG由四个部分组成：

1.  The constant PNG signature (8 bytes)  
    常量 PNG 签名（8 字节）
2.  The IHDR chunk, containing metadata (25 bytes)  
    包含元数据的 IHDR 区块（25 字节）
3.  The IDAT chunk, image pixel data (22 bytes)  
    IDAT 块，图像像素数据（22 字节）
4.  The IEND chunk, an image trailer (12 bytes)  
    IEND 块，图像尾部（12 字节）

If you’re interested in learning more about PNGs interactively, I built [PNG Chunk Explorer](https://evanhahn.gitlab.io/png-explorer/), which lets you analyze PNGs. Try uploading your own images to see what they’re made of! (It doesn’t work well on mobile, apologies.)  
如果您有兴趣以交互方式了解有关 PNG 的更多信息，我构建了 PNG Chunk Explorer，可让您分析 PNG。 尝试上传您自己的图像，看看它们是由什么组成的！（很抱歉，它在移动设备上效果不佳。

I also built [Single Color Image](https://singlecolorimage.com/), which generates monochromatic PNGs of arbitrary sizes. For example, you could generate a 12×34 purple rectangle. The images should be small but I haven’t yet implemented the most sophisticated compression, so you might need to run its results through a PNG compressor to achieve the smallest sizes.  
我还构建了单色图像，它可以生成任意大小的单色 PNG。例如，您可以生成一个 12×34 的紫色矩形。图像应该很小，但我还没有实现最复杂的压缩，所以你可能需要通过 PNG 压缩器运行它的结果来实现最小的尺寸。

Finally, I also wrote about the [_largest_ possible PNG](https://evanhahn.com/largest-possible-png/). There’s no theoretical file size limit, but there is a maximum number of pixels, and many decoders impose various limits.  
最后，我还写了关于最大的 PNG。理论上没有文件大小限制，但有最大像素数，许多解码器施加了各种限制。

I hope this long post has given you a good understanding of the PNG file format. If you read this far and have anything to say, [let me know](https://evanhahn.com/contact/)!  
我希望这篇长文能让您对 PNG 文件格式有很好的了解。如果您读到这里并有什么要说的，请告诉我！