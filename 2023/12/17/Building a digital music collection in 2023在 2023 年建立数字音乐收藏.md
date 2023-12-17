> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.worthe-it.co.za](https://www.worthe-it.co.za/blog/2023-08-20-building-a-digital-music-collection-in-2023.html)

> Opus is a great free audio codecOpus 是一款出色的免费音频编解码器

Abstract 抽象
-----------

I've stopped paying for music streaming services in favour of going back to buying albums and building my own digital music collection. This article is about some of the modern tools I'm applying to this endeavour, including updating my choice in audio codecs for storing the collection.  
我已经停止为音乐流媒体服务付费，转而重新购买专辑并建立自己的数字音乐收藏。本文是关于我应用于这项工作的一些现代工具，包括更新我在音频编解码器中选择的用于存储收藏的音频编解码器。

I like to listen to music.  
我喜欢听音乐。

When I was small, this meant cassette tapes, or later CDs. I even had some adventurous dives into my parents' vinyl collection. As someone who grew up interested in computers, digital music was of course my medium of choice, and I started up a collection of MP3 files.  
当我小的时候，这意味着盒式磁带，或者后来的CD。我甚至对我父母的黑胶唱片收藏进行了一些冒险的潜水。作为一个从小就对计算机感兴趣的人，数字音乐当然是我选择的媒介，我开始收集 MP3 文件。

At some point in the last decade, internet access became omnipresent. I stopped collecting new music, because it was convenient enough to open up a music streaming site and search for the music I wanted to listen to.  
在过去十年的某个时候，互联网接入变得无处不在。我停止收集新音乐，因为它足够方便，可以打开一个音乐流媒体网站并搜索我想听的音乐。

But something was lost with music streaming. My music library didn't grow to suit my tastes anymore. I'd sometimes find that particular songs or artists were delisted, or just not available in my country anymore. Streaming sites seemed to become increasingly interested in trying to get me to listen the songs they were promoting, rather than the songs I was interested in. The "recommended for you" sections filled up with popular artists who I didn't want anything to do with. The amount of adverts went up, and with them the value of music streaming sites dropped.  
但是音乐流媒体丢失了一些东西。我的音乐库不再适合我的口味。我有时会发现某些歌曲或艺术家被除名，或者在我的国家不再可用。流媒体网站似乎越来越有兴趣让我听他们正在推广的歌曲，而不是我感兴趣的歌曲。“为您推荐”部分充满了我不想与之有任何关系的流行艺术家。广告数量增加，音乐流媒体网站的价值也随之下降。

I've decided to go back to curating a big directory of music files. Not to brag, but I know a lot more about managing directories of files now than I did when I was in school. This article is going to look at two aspects of this music management problem:  
我决定回去策划一个大的音乐文件目录。不是吹嘘，但我现在对管理文件目录的了解比我在学校时要多得多。本文将探讨这个音乐管理问题的两个方面：

1.  How I sync my music between devices, and keep backups so I don't lose it.  
    我如何在设备之间同步我的音乐，并保留备份，这样我就不会丢失它。
    
2.  How to keep the file sizes down to make the syncing problem easier.  
    如何减小文件大小以使同步问题更容易。
    

Push it to a Git repo  
将其推送到 Git 存储库
-------------------------------------

The underlying technical challenge to with keeping your own music collection is managing a big directory of files. I want it to be easy to sync that directory between various computers and players. I care about it having a backup so I don't lose it when things inevitably go wrong. Long time readers of this blog will not be surprised at all to discover that my solution for this is to put it all in a Git repo. Then I can easily commit, push, pull, and all the other normal Git actions to keep my work computer and personal computer in sync. Also, my Git server runs a daily backup, so anything I push to it is backed up automatically!  
保留自己的音乐收藏的潜在技术挑战是管理一个大的文件目录。我希望在各种计算机和播放器之间轻松同步该目录。我关心它有一个备份，这样当事情不可避免地出错时，我就不会丢失它。本博客的长期读者不会惊讶地发现，我的解决方案是将其全部放在 Git 存储库中。然后，我可以轻松地提交、推送、拉取和所有其他正常的 Git 操作，以保持我的工作计算机和个人计算机同步。此外，我的 Git 服务器每天都会运行备份，因此我推送到它的任何内容都会自动备份！

There are a few pitfalls here, and they all come down to media archives generally having larger file sizes than source code.  
这里有一些陷阱，它们都归结为媒体档案通常具有比源代码更大的文件大小。

1.  Your music repo might be too big for your Git hosting provider. I didn't run into this because I [run my own Git server](https://www.worthe-it.co.za/blog/2022-06-10-how-to-train-your-git-server.html) (although I did need to ask my cloud provider for a bigger hard drive). You can probably also get around this by using an extension to Git called Git LFS (Large File Storage), which many Git servers support. If you're not as invested in Git as I am, you can also use other file syncing products (I don't know which are the"good" ones these days).  
    对于您的 Git 托管服务提供商来说，您的音乐存储库可能太大了。我没有遇到这种情况，因为我运行自己的 Git 服务器（尽管我确实需要向我的云提供商询问更大的硬盘）。您可能还可以通过使用名为 Git LFS（大文件存储）的 Git 扩展来解决这个问题，许多 Git 服务器都支持该扩展。如果你不像我一样对 Git 投入，你也可以使用其他文件同步产品（我不知道现在哪些是“好”的）。
    
2.  Over time, Git doesn't automatically store the changes to your music efficiently. I found that regularly running `git gc` on the server helped to keep the repo's overall size closer to just the size of the music in it. I also found that [Pijul](https://pijul.org/) (a different version control system) keeps the file sizes under control better than Git. Unfortunately I also found that making new commits in Pijul got slow as the repo got big. I see a lot of promise in Pijul and I hope that this improves in future releases.  
    随着时间的流逝，Git 不会自动有效地存储对音乐的更改。我发现定期在服务器上运行 `git gc` 有助于使 repo 的整体大小更接近其中音乐的大小。我还发现 Pijul（一种不同的版本控制系统）比 Git 更好地控制文件大小。不幸的是，我还发现，随着 repo 变大，在 Pijul 中进行新提交变得很慢。我在 Pijul 中看到了很多希望，我希望在未来的版本中有所改善。
    

Let's talk about getting file sizes down  
让我们谈谈减小文件大小
------------------------------------------------------

Smaller music files are faster to sync and cheaper to store in the cloud. Big files make everything harder.  
较小的音乐文件同步速度更快，存储在云中的成本也更低。大文件使一切变得更加困难。

Now you can get some improvements by taking your music file and putting it into a `.zip` file, but to really get that size down you need to use a codec.  
现在，您可以通过将音乐文件放入 `.zip` 文件中来获得一些改进，但要真正减小该大小，您需要使用编解码器。

### What are audio codecs? 什么是音频编解码器？

The word "codec" comes from shoving the words "coder" and "decoder" together. Basically a codec is made up of two programs:  
“编解码器”一词来自将“编码器”和“解码器”这两个词推到一起。基本上，编解码器由两个程序组成：

*   The coder program takes some digital signal, like the [the air pressure measurements of a recorded sound](https://www.worthe-it.co.za/blog/2017-01-29-what-note-is-this-1.html), and converts it into some encoded form, usually compressed in some way.  
    编码程序获取一些数字信号，例如录制声音的气压测量值，并将其转换为某种编码形式，通常以某种方式压缩。
    
*   The decoder program takes the encoded data and transforms it back into the original signal, for example getting back those air pressure measurements to send to a speaker.  
    解码器程序获取编码数据并将其转换回原始信号，例如，获取气压测量值以发送到扬声器。
    

There are lots of different codecs that are good for different things. Some are good for video. Some are good for audio. For the purposes of this blog post, I'm going to focus specifically on audio codecs.  
有许多不同的编解码器适用于不同的事情。有些适合视频。有些对音频有好处。出于这篇博文的目的，我将特别关注音频编解码器。

When you're looking at the air pressure measurements that make up an audio signal, they aren't just random numbers. Audio follows a whole bunch of patterns just because it was created by and moved through the real corporeal world. Audio also has some common use cases like talking or music which have their own patterns. Audio codecs can take advantage of these patterns when compressing audio.  
当您查看构成音频信号的气压测量值时，它们不仅仅是随机数。音频遵循一大堆模式，只是因为它是由真实的物质世界创造并在现实世界中移动的。音频也有一些常见的用例，如说话或音乐，它们有自己的模式。音频编解码器在压缩音频时可以利用这些模式。

Broadly speaking, there are two types of codecs. "Lossless" codecs will give back exactly the same signal after decoding. "Lossy" codecs decode to a different signal that has the same effect. A lossless audio codec would give back exactly the same air pressure measurements. A lossy audio codec would give different air pressure measurements that sound the same to a human listener. A lossy codec might cut out sound that's outside of the range the human ear can hear, or sounds that you can't hear because they're covered by other sounds. Lossy codecs are great for our purposes because they can make the file much smaller than lossless codecs, and are just as good for listening to.  
从广义上讲，有两种类型的编解码器。“无损”编解码器在解码后会返回完全相同的信号。“有损”编解码器解码为具有相同效果的不同信号。无损音频编解码器将返回完全相同的气压测量值。有损音频编解码器会给出不同的气压测量值，这些测量值对人类听众来说听起来是一样的。有损编解码器可能会切断人耳可以听到的范围之外的声音，或者由于被其他声音覆盖而听不到的声音。有损编解码器非常适合我们的目的，因为它们可以使文件比无损编解码器小得多，并且同样适合收听。

Many codecs have different settings, where you can decrease the audio quality to get smaller files. The term "bitrate" refers to how many kilobits of file are needed for each second of audio. Ideally, we'd like to have a really low bitrate while the music still sounds the same.  
许多编解码器具有不同的设置，您可以在其中降低音频质量以获得较小的文件。术语“比特率”是指每秒音频需要多少千比特的文件。理想情况下，我们希望在音乐听起来仍然相同的情况下拥有非常低的比特率。

### How do you choose a bitrate?  
你如何选择比特率？

It's really difficult to say what a good quality level to choose is, since the"quality" of sound it fairly subjective. Researchers have asked people to listen to music encoded using a variety of codecs and a variety of bitrates and rate them.  
真的很难说选择什么是好的质量水平，因为声音的“质量”是相当主观的。研究人员要求人们听使用各种编解码器和各种比特率编码的音乐，并对其进行评分。

I skipped reading that research and went straight to [FFMPEG's high quality audio encoding guide](https://trac.ffmpeg.org/wiki/Encode/HighQualityAudio). They have minimum and recommended bitrates for a variety of codecs, and I just pick the "recommended" bitrate.  
我跳过了阅读该研究，直接阅读了 FFMPEG 的高质量音频编码指南。它们为各种编解码器提供了最小和推荐的比特率，我只选择“推荐”比特率。

### The audio codec everyone has heard of: MP3  
大家都听说过的音频编解码器：MP3

MP3 files were so ubiquitous when I was growing up that I hardly feel like I need to explain what they are. But with the rise of music streaming services, and people not really managing their own music files anymore, maybe it isn't as well known anymore.  
在我成长的过程中，MP3 文件无处不在，以至于我几乎不需要解释它们是什么。但随着音乐流媒体服务的兴起，人们不再真正管理自己的音乐文件，也许它不再那么广为人知了。

MP3 is a lossy codec which was developed in the 90s. When music started being downloaded on the internet (both legally and questionably legally), MP3 was the codec people generally used. It was also the codec generally supported by early portable players.  
MP3 是一种有损编解码器，于 90 年代开发。当音乐开始在互联网上下载时（无论是合法的还是有问题的），MP3 是人们普遍使用的编解码器。它也是早期便携式播放器普遍支持的编解码器。

This made MP3 the gold standard when I started building my music collection.  
这使得 MP3 成为我开始建立音乐收藏的黄金标准。

FFMPEG's high quality audio encoding guide recommends a bitrate of 192Kbps or higher for high quality audio in MP3 format.  
FFMPEG 的高质量音频编码指南建议 MP3 格式的高质量音频的比特率为 192Kbps 或更高。

### The new awesome audio codec: Opus  
全新精彩音频编解码器：Opus

In 2012, the Xiph.org Foundation released the first version of their Opus codec.  
2012 年，Xiph.org 基金会发布了其 Opus 编解码器的第一个版本。

Opus is also a lossy codec. For my use case, there's one important distinction:  
Opus 也是一个有损编解码器。对于我的用例，有一个重要的区别：

In listening tests, Opus files are significantly smaller than MP3 files of the same perceived quality. FFMPEG's high quality audio encoding guide recommends a bitrate of 64Kbps or higher for high quality audio in Opus format.  
在听力测试中，Opus 文件明显小于相同感知质量的 MP3 文件。FFMPEG 的高质量音频编码指南建议 Opus 格式的高质量音频的比特率为 64Kbps 或更高。

Opus is also a free and open codec. The source code for the Xiph.org implementation is released under an open source license. Anyone is free to write and distribute their own Opus implementation. This makes it a good choice in terms of sustainability. If someone wants to add Opus support to their project, they don't need consider the costs of patents and licensing, they can just use it.  
Opus 也是一个免费和开放的编解码器。Xiph.org 实现的源代码在开源许可下发布。任何人都可以自由地编写和分发自己的 Opus 实现。这使其成为可持续性方面的不错选择。如果有人想在他们的项目中添加 Opus 支持，他们不需要考虑专利和许可的成本，他们可以使用它。

### How do I convert my library to use Opus?  
如何将我的库转换为使用 Opus？

Opus format is great, so I wanted a nice way to bulk convert my existing collection. I also wanted the script to be easy to convert new albums I download if they weren't already using Opus.  
Opus 格式很棒，所以我想要一种很好的方法来批量转换我现有的收藏。我还希望脚本能够轻松转换我下载的新专辑，如果他们还没有使用 Opus。

This script is called `to-opus.sh`, and I use `find` to bulk run it on all the files that match a pattern in a directory. Be careful when trying this out. It deletes the input files after doing the conversion, so maybe try this on a backup of your collection at first!  
这个脚本叫做 `to-opus.sh` ，我用它来 `find` 批量运行它与目录中的模式匹配的所有文件。尝试此操作时要小心。它会在进行转换后删除输入文件，所以也许先在您的收藏备份中尝试一下！

```
#!/bin/sh

# Use together with find, for example:
# find ./ -name "*.mp3" -exec ./to-opus.sh "{}" \;

IN="$1"
OUT=${1%.*}.ogg
BITRATE=64k

echo "$IN->$OUT"

ffmpeg -y -loglevel "error" -i "$IN" -vn -acodec libopus -b:a $BITRATE "$OUT"
rm "$IN"


```

What do I look for when getting more music?  
获得更多音乐时，我应该寻找什么？
--------------------------------------------------------------

Unfortunately, most online music stores and other people putting their music online I've encountered don't offer Opus yet. Here are some guidelines that work for me:  
不幸的是，我遇到的大多数在线音乐商店和其他人将他们的音乐放到网上的人还没有提供 Opus。以下是一些对我有用的指南：

*   If possible, download in a lossless codec, then re-encode to Opus. That way you don't take a quality hit from encoding to two different lossy codecs.  
    如果可能，请以无损编解码器下载，然后重新编码到 Opus。这样一来，您就不会因编码到两种不同的有损编解码器而对质量造成影响。
    
*   FLAC is a popular lossless codec, also from the Xiph.org Foundation. So if possible, download in FLAC format.  
    FLAC是一种流行的无损编解码器，也来自 Xiph.org 基金会。因此，如果可能，请以FLAC格式下载。
    
*   Failing that, download in the highest audio quality you can, and re-encode to Opus to get the filesize down.  
    如果做不到这一点，请以尽可能高的音频质量下载，然后重新编码到 Opus 以减小文件大小。
    
*   I've found that [Bandcamp](https://bandcamp.com/) is a great platform for buying and downloading music to expand my collection. I've generally been able to download the music I bought in FLAC format from them.  
    我发现 Bandcamp 是购买和下载音乐以扩大我的收藏的绝佳平台。我通常能够从他们那里下载我以FLAC格式购买的音乐。
    

Actively curate the stuff you have  
积极策划你拥有的东西
-----------------------------------------------

It's easier than ever to download a lot of music and end up with a massive collection. Some of it bought, some of it is just freely put on the internet for download by its artists. It's very easy for a collection to get out of hand over time.  
下载大量音乐并最终获得大量收藏比以往任何时候都更容易。其中一些是购买的，一些只是免费放在互联网上供其艺术家下载。随着时间的流逝，收藏品很容易失控。

Here are some things I do to curate my collection:  
以下是我为策划我的收藏所做的一些事情：

*   I have a regular routine on my calendar to do a little bit of curation of my collection. Every few days, I look at an album or two in my collection that I haven't listened to in a while. It's also really nice to have some dedicated time just to listen to music.  
    我的日历上有一个固定的例行公事，对我的收藏进行一些策展。每隔几天，我就会看一两张我收藏的专辑，这些专辑我已经有一段时间没有听过了。有一些专门的时间只是听音乐也很好。
    
*   I use [Picard](https://picard.musicbrainz.org/) to make sure that my file metadata and folder structure are as I like it. I don't necessarily sort things the same way now as I did ten years ago, so it's useful to have a nice tool for bulk updating things. I do this as part of the regular routine, so I don't try to look through the metadata of my whole collection at once.  
    我使用 Picard 来确保我的文件元数据和文件夹结构符合我的喜好。我现在不一定像十年前那样对事物进行排序，因此拥有一个很好的工具来批量更新事物很有用。我这样做是常规工作的一部分，所以我不会尝试一次查看整个集合的元数据。
    
*   I delete things that no longer spark joy. Deleting things is an important part of curation, even if my general inclination is to keep things forever.  
    我删除了不再激发快乐的东西。删除内容是策展的重要组成部分，即使我的总体倾向是永远保留内容。
    

I've been enjoying how once again the music that I play when I open up my computer is evolving to match my particular taste. I like that if I don't like a song or genre I can just not have it in my collection and it will never be auto-played next.  
我一直很享受当我打开电脑时播放的音乐如何再次演变以符合我的特定品味。我喜欢这样，如果我不喜欢一首歌或一种流派，我就不能把它放在我的收藏中，而且它永远不会被自动播放。

I like that this is a collection that I control. That I can keep backups and sync it between my computers as I see fit, even if that way is unusual like using a Git repo. I really appreciate having found Opus as an alternative to MP3, which can basically get my file sizes down with no noticeable loss of quality!  
我喜欢这是我控制的集合。我可以保留备份并在我认为合适的计算机之间同步它，即使这种方式不寻常，例如使用 Git 存储库。我真的很感激找到 Opus 作为 MP3 的替代品，它基本上可以减小我的文件大小而不会明显降低质量！

This is working well for me. I can recommend you try it too. Keep the bits that work for you, and leave the rest.  
这对我来说效果很好。我也可以推荐你尝试一下。保留适合您的部分，其余的则保留。

If you get value from these blog articles, consider supporting me on [Patreon](https://www.patreon.com/worthe_it). Support via Patreon helps to cover hosting, buying computer stuff, and will allow me to spend more time writing articles and open source software.  
如果您从这些博客文章中获得价值，请考虑在 Patreon 上支持我。通过 Patreon 提供的支持有助于涵盖托管、购买计算机内容，并允许我花更多时间撰写文章和开源软件。

Have you considered hosting your own Git server? It's easier than you might think. In this article, I go step by step through setting up a simple self-hosted Git server which only supports private repositories for a single person.  
您是否考虑过托管自己的 Git 服务器？这比你想象的要容易。在本文中，我将逐步设置一个简单的自托管 Git 服务器，该服务器仅支持单个人的私有存储库。

I wanted to manage the process of syncing audiobooks from my computer to my phone better. The solution that worked well for me is to use a podcasting app and an RSS feed. This article explains why this works well for me, and how you can try it out.  
我想更好地管理将有声读物从计算机同步到手机的过程。对我来说效果很好的解决方案是使用播客应用程序和 RSS 提要。这篇文章解释了为什么这对我有用，以及如何尝试它。