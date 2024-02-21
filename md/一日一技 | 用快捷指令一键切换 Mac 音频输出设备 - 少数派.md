> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/86520)

> 大家好，我是 NowScott。

原因
--

之前在使用 Windows 电脑时，由于经常需要把声音输出在音箱和耳机之间频繁切换，就找到了一个软件来实现一键切换（鼠标侧键映射到快捷键上）音频输出设备，但是换到 Mac 上暂时没发现一个好用的实现这个功能的软件，应该有吧，实际上是我没太仔细找，想着用快捷指令也能很好地实现，就自己写了一个快捷指令来实现这个功能。

原理部分
----

SwitchAudioSource 是一个在 macOS 系统上运行的命令行工具，用于管理系统的音频输入和输出设备。通过这个工具，用户可以在不打开系统偏好设置的情况下，快速而方便地切换音频设备，这也是本文主要使用的工具。

Mac 上的快捷指令可以用来调用 shell 脚本，同时支持快捷键对于指令的调用，因此我只需要把音频切换的代码放在快捷指令的 shell 脚本中，就可以实现使用快捷键来切换音频。

代码部分
----

代码部分如下，相信学过一些编程的人就可以轻松看懂，如果有什么问题可以评论区指出，大家共同进步！

```
#!/bin/bash

# 设置环境变量
export PATH=/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/opt/homebrew/Cellar/switchaudio-osx/1.2.2:$PATH

# 获取全部音频设备的列表
available_devices=$(SwitchAudioSource -a -t output)

# 定义音频设备列表
devices=("AirPods Max" "Mac mini扬声器")

# 获取当前音频设备
current_device=$(SwitchAudioSource -c)

# 查找当前设备在列表中的索引
current_index=-1
for i in "${!devices[@]}"; do
    if [ "${devices[i]}" == "$current_device" ]; then
        current_index=$i
        break
    fi
done

# 切换到下一个设备
next_index=$(( (current_index + 1) % ${#devices[@]} ))
next_device="${devices[next_index]}"

# 循环直到找到一个可用的设备
while true; do
    # 检查下一个设备是否在可用设备列表中
    if echo "$available_devices" | grep -q "$next_device"; then
        # 检查是否已经循环到了当前设备
        if [ "$next_device" != "$current_device" ]; then
            # 切换设备
            SwitchAudioSource -s "$next_device" >/dev/null
            echo "已切换设备到 $next_device"
            break
        else
            echo "切换失败，暂无其他可用设备"
            break
        fi
    fi
    
    # 尝试下一个设备
    next_index=$(( (next_index + 1) % ${#devices[@]} ))
    next_device="${devices[next_index]}"
done

```

其中 `devices=("AirPods Max" "Mac mini扬声器")` 要将括号中的内容换成你的音频设备，可以在终端中输入如下代码来获取

```
SwitchAudioSource -a -t output

```

当然在此之前要确保你已经安装了 SwitchAudioSource。

要安装 SwitchAudioSource，您可以使用 Homebrew 工具来进行安装。

首先，确保您已经安装了 Homebrew。如果您还没有安装 Homebrew，可以在终端中输入以下命令进行安装：

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" 

```

安装完成后，可以使用以下命令来安装 SwitchAudioSource：

```
 brew install switchaudio-osx

```

使用
--

下面是我分享的快捷指令 iCloud 链接，你可以点击链接获取捷径或者跟着我用以下几个步骤来创建一个自己的切换音频的快捷指令

🔗 链接：[https://www.icloud.com/shortcuts/2d1a8f9c4b054775906e6374436156cd](https://sspai.com/link?target=https%3A%2F%2Fwww.icloud.com%2Fshortcuts%2F2d1a8f9c4b054775906e6374436156cd)

⚠️ 注意：使用链接添加是不会自动添加快捷键的，同时还需要修改自己的音频设备名称到代码中。

创建步骤
----

1. 点击 + 创建新的快捷指令：

![](https://cdn.sspai.com/2024/02/19/13217c8a06a0d0ee0a4b3b6eb31d9ff6.png)

2. 搜索 shell，将运行 shell 脚本拖进来，接着修改一下快捷指令的名称：

![](https://cdn.sspai.com/2024/02/19/fdaee40f3fd648b98e081b39474713e2.png)

3. 复制前文代码到其中，并将下面的 shell 改为 bash：

![](https://cdn.sspai.com/2024/02/19/756a3d657a05ad37ff19ce79608a199d.png)

4. 修改设备列表为你的音频输出设备：

![](https://cdn.sspai.com/2024/02/19/12fd309ac90edb59744ea46a58b9e321.png)

5. 接下来添加键盘快捷键，我这里设置的是 Fn + control + F10：

![](https://cdn.sspai.com/2024/02/19/2961ca61dfcfeb8e0868c522d128fc50.png)

6. 如果你需要切换后通知一下，那么可以在后面添加一个通知或者提醒（管理员身份运行并不重要，我只是顺手点上了）：

![](https://cdn.sspai.com/2024/02/19/b125f6c0cb1e631db3d1fd7ce3fc102f.png)

这样就实现了这个快捷指令的制作，如果有需要可以把鼠标侧键绑定为刚才设置的快捷键，就能实现一键切换音频输出了。

结尾
--

如果你通过我的文章有多学到一点知识，那我的目的就达成了。

如果发现本文有什么错误可以在评论区发出来，我会虚心接受并及时改正。