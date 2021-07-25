> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2MDY2NzQwMQ==&mid=2247492400&idx=1&sn=f54fb9f7000aecaf200fadfa1ab61f26&chksm=ea648256dd130b4090d002209792d81ae768221d3b4f50a389a63e4bb564bd6a64f4251a6701&mpshare=1&scene=1&srcid=0724VtXnYebVdIt624D0UBJ5&sharer_sharetime=1627115581509&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

作者：小小明，博客地址：_https://blog.csdn.net/as604049322_

最近发现有个玩数独游戏的网站：_https://www.sudoku.name/index-cn.php_

游戏界面如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AlvZF8lGnXkgFSFkevnjdWCVMDuFLbeyDvzjncdlCAtXAwWfpia3SgsOptFx8JicHuSun9q776BSKhQ/640?wx_fmt=png)

当然这类玩数独游戏的网站很多，现在我们先以该网站为例进行演示。希望能用 **Python 实现自动计算并填好数独游戏**！

大概效果能像下面这样就好啦👇

![](https://mmbiz.qpic.cn/mmbiz_gif/P33BNFicia1AlvZF8lGnXkgFSFkevnjdWCzzCLqh8lbB96ErzjeOI3p4p7LrSPibnroj02CtiadNLx2VAO8KnnL9dQ/640?wx_fmt=gif)

玩过的都非常清楚数独的基本规则：

1.  数字 1-9 在每一行只能出现一次。
    
2.  数字 1-9 在每一列只能出现一次。
    
3.  数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。
    

如何让程序辅助我们玩这个数独游戏呢？

思路：

*   我们可以通过 web 自动化测试工具（例如 selenium）打开该网页
    
*   解析网页获取表格数据
    
*   传入处理程序中自动解析表格
    
*   使用程序自动写入计算好的数独结果
    

下面我们尝试一步步解决这个问题：

通过 Selenium 访问目标网址
------------------

关于 selenium 的安装请参考：

_https://blog.csdn.net/as604049322/article/details/114157526_

首先通过 selenium 打开游览器：

如果你的 selenium 已经正确安装，运行上述代码会打开谷歌游览器：

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AlvZF8lGnXkgFSFkevnjdWCBicAeecY8uJ0UsicSgfIAZ3h0JhHcMCDX93akXVGCPsySdwibLUEibSPvw/640?wx_fmt=png)

此时我们可以通过直接在受控制的游览器输入 url 访问，也可以用代码控制游览器访问数独游戏的网址：

```
url = "https://www.sudoku.name/index.php?ln=cn&puzzle_num=&play=1&difficult=4&timer=&time_limit=0"
browser.get(url) 
```

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AlvZF8lGnXkgFSFkevnjdWCYYqXiad62V9EwtoW1t0zD6705W9zEfBYUUvfkQJicBoiaEH5kK3ibg7jow/640?wx_fmt=png)

> 内心 PS：以后还是得给谷歌游览器装个去广告的插件

数独数据提取
------

**节点分析**

table 节点的 id 为：

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AlvZF8lGnXkgFSFkevnjdWCqmiawkaK5vOWzAllfwqA2rl6sH6ibs5WLuNLmgbCEcxNgwgS8lAZUrQA/640?wx_fmt=png)

节点值存在于 value 属性中：

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AlvZF8lGnXkgFSFkevnjdWCUicr1zlIfAnGlttWEHYeUf1jPiaTWt5tcG31ia0Yicv3RuJ6QsDvEW9mlA/640?wx_fmt=png)

使用 Selenium 控制游览器就是这个好处，可以随时让程序提取我们需要的数据。

首先获取目标 table 标签：

```
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(browser, 10)

table = wait.until(EC.element_to_be_clickable(
    (By.CSS_SELECTOR, 'table#sudoku_main_board'))) 
```

下面我们根据对节点的分析提取出我们需要的数独数据：

```
board = []
for tr in table.find_elements_by_xpath(".//tr"):
    row = []
    for input_e in tr.find_elements_by_xpath(".//input[@class='i3']"):
        cell = input_e.get_attribute("value")
        row.append(cell if cell else ".")
    board.append(row)
board 
```

```
[['7', '.', '.', '.', '.', '4', '.', '.', '.'],
 ['.', '4', '.', '.', '.', '5', '9', '.', '.'],
 ['8', '.', '.', '.', '.', '.', '.', '2', '.'],
 ['.', '.', '6', '.', '9', '.', '.', '.', '4'],
 ['.', '1', '.', '.', '.', '.', '.', '3', '.'],
 ['2', '.', '.', '.', '8', '.', '5', '.', '.'],
 ['.', '5', '.', '.', '.', '.', '.', '.', '1'],
 ['.', '.', '3', '7', '.', '.', '.', '8', '.'],
 ['.', '.', '.', '2', '.', '.', '.', '.', '6']] 
```

> 将凡是需要填写的位置都用. 表示。

数独计算程序
------

如何对上述数独让程序来计算结果呢？这就需要逻辑算法的思维了。

这类问题最基本的解题思维就是通过递归 + 回溯算法遍历所有可能的填法挨个验证有效性，直到找到没有冲突的情况。在递归的过程中，如果当前的空白格不能填下任何一个数字，那么就进行回溯。

在此基础上，我们可以使用位运算进行优化。常规方法我们需要使用长度为 99 的数组表示每个数字是否出现过，借助位运算，仅使用一个整数就可以表示每个数字是否出现过。例如二进制表 (011000100) 表示数字 3，7，8 已经出现过。

具体而言最终的程序还算比较复杂的，无法理解代码逻辑的可以直接复制粘贴：

```
def solveSudoku(board):
    def flip(i: int, j: int, digit: int):
        line[i] ^= (1 << digit)
        column[j] ^= (1 << digit)
        block[i // 3][j // 3] ^= (1 << digit)

    def dfs(pos: int):
        nonlocal valid
        if pos == len(spaces):
            valid = True
            return

        i, j = spaces[pos]
        mask = ~(line[i] | column[j] | block[i // 3][j // 3]) & 0x1ff
        while mask:
            digitMask = mask & (-mask)
            digit = bin(digitMask).count("0") - 1
            flip(i, j, digit)
            board[i][j] = str(digit + 1)
            dfs(pos + 1)
            flip(i, j, digit)
            mask &= (mask - 1)
            if valid:
                return

    line = [0] * 9
    column = [0] * 9
    block = [[0] * 3 for _ in range(3)]
    valid = False
    spaces = list()

    for i in range(9):
        for j in range(9):
            if board[i][j] == ".":
                spaces.append((i, j))
            else:
                digit = int(board[i][j]) - 1
                flip(i, j, digit)

    dfs(0) 
```

然后我们运行一下：

```
solveSudoku(board)
board 
```

```
[['7', '2', '9', '3', '6', '4', '1', '5', '8'],
 ['3', '4', '1', '8', '2', '5', '9', '6', '7'],
 ['8', '6', '5', '9', '7', '1', '4', '2', '3'],
 ['5', '3', '6', '1', '9', '2', '8', '7', '4'],
 ['9', '1', '8', '5', '4', '7', '6', '3', '2'],
 ['2', '7', '4', '6', '8', '3', '5', '1', '9'],
 ['6', '5', '2', '4', '3', '8', '7', '9', '1'],
 ['4', '9', '3', '7', '1', '6', '2', '8', '5'],
 ['1', '8', '7', '2', '5', '9', '3', '4', '6']] 
```

可以看到，程序已经计算出了数独的结果。

不过对于数据：

```
[['.', '.', '.', '6', '.', '.', '.', '3', '.'],
 ['5', '.', '.', '.', '.', '.', '6', '.', '.'],
 ['.', '9', '.', '.', '.', '5', '.', '.', '.'],
 ['.', '.', '4', '.', '1', '.', '.', '.', '6'],
 ['.', '.', '.', '4', '.', '3', '.', '.', '.'],
 ['8', '.', '.', '.', '9', '.', '5', '.', '.'],
 ['.', '.', '.', '7', '.', '.', '.', '4', '.'],
 ['.', '.', '5', '.', '.', '.', '.', '.', '8'],
 ['.', '3', '.', '.', '.', '8', '.', '.', '.']] 
```

上述算法耗时居然达到 17 秒，还需继续优化算法：

```
def solveSudoku(board: list) -> None:
    def flip(i: int, j: int, digit: int):
        line[i] ^= (1 << digit)
        column[j] ^= (1 << digit)
        block[i // 3][j // 3] ^= (1 << digit)

    def dfs(pos: int):
        nonlocal valid
        if pos == len(spaces):
            valid = True
            return

        i, j = spaces[pos]
        mask = ~(line[i] | column[j] | block[i // 3][j // 3]) & 0x1ff
        while mask:
            digitMask = mask & (-mask)
            digit = bin(digitMask).count("0") - 1
            flip(i, j, digit)
            board[i][j] = str(digit + 1)
            dfs(pos + 1)
            flip(i, j, digit)
            mask &= (mask - 1)
            if valid:
                return

    line = [0] * 9
    column = [0] * 9
    block = [[0] * 3 for _ in range(3)]
    valid = False
    spaces = list()

    for i in range(9):
        for j in range(9):
            if board[i][j] != ".":
                digit = int(board[i][j]) - 1
                flip(i, j, digit)

    while True:
        modified = False
        for i in range(9):
            for j in range(9):
                if board[i][j] == ".":
                    mask = ~(line[i] | column[j] |
                             block[i // 3][j // 3]) & 0x1ff
                    if not (mask & (mask - 1)):
                        digit = bin(mask).count("0") - 1
                        flip(i, j, digit)
                        board[i][j] = str(digit + 1)
                        modified = True
        if not modified:
            break

    for i in range(9):
        for j in range(9):
            if board[i][j] == ".":
                spaces.append((i, j))

    dfs(0) 
```

再次运行：

```
solveSudoku(board)
board 
```

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AlvZF8lGnXkgFSFkevnjdWCWiarHDicJw789ByrRrX0BDZib8DUpicfOMc9k0yq561p16icKQicODkAm2gg/640?wx_fmt=png)

耗时仅 3.2 秒，性能提升不少。

> 优化思路：如果一个空白格只有唯一的数可以填入，也就是其对应的 b 值和 b-1 进行按位与运算后得到 0（即 b 中只有一个二进制位为 1）。此时，我们就可以确定这个空白格填入的数，而不用等到递归时再去处理它。

下面我们需要做的就是将结果填入到相应的位置中，毕竟自己手敲也挺费劲的。

写结果回写到网页
--------

对于 Selenium，我们可以模拟人工点击按钮并发送键盘操作。

下面我们重新遍历 table 标签，并使用 click 和 send_keys 方法：

```
for i, tr in enumerate(table.find_elements_by_xpath(".//tr")):
    for j, input_e in enumerate(tr.find_elements_by_xpath(".//input[@class='i3']")):
        if input_e.get_attribute("readonly") == "true":
            continue
        input_e.click()
        input_e.clear()
        input_e.send_keys(board[i][j]) 
```

运行过程中的效果：

![](https://mmbiz.qpic.cn/mmbiz_gif/P33BNFicia1AlvZF8lGnXkgFSFkevnjdWCzzCLqh8lbB96ErzjeOI3p4p7LrSPibnroj02CtiadNLx2VAO8KnnL9dQ/640?wx_fmt=gif)

△程序自动填写

**骨灰级数独玩家证明：**

![](https://mmbiz.qpic.cn/mmbiz_png/P33BNFicia1AlvZF8lGnXkgFSFkevnjdWCmMnJKMLVkEdHx4Rpomttzgb7vyfEJSD7hXbTfhuuW2Fiaf1jANFX3PQ/640?wx_fmt=png)

别人 14 分钟，你用程序 10 秒填完。

用 Python 后终于也体验了一次 “最强大脑” 的感觉了，先容我装个 B 去🚀

大家如果喜欢这类文章，欢迎关注小明哥的博客👉_https://blog.csdn.net/as604049322_

大家如果本文涉及的代码感兴趣，可以扫描下面二维码👇在【快学 Python】后台回复 “**数独**” 即可获取对应代码文件。