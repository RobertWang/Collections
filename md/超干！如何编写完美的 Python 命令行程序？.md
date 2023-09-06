> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI0MzU2NzQ1OA==&mid=2247525240&idx=1&sn=8141217fb6813d67e5a96abf703837dd&chksm=e9693d75de1eb463b7c418e3832c0844b27797ea65239f59ff044879a5ea413511f38a45c087&scene=21#wechat_redirect)

> 编辑：乐乐 | 来自：网络

这篇文章将教你如何编写完美的 Python 命令行程序，提高团队的生产力，让大家的工作更舒适。

作为 Python 开发者，我们经常要编写命令行程序。比如在我的数据科学项目中，我要从命令行运行脚本来训练模型，以及计算算法的准确率等。

因此，更方便更易用的脚本能够很好地提高生产力，特别是在有多个开发者从事同一个项目的场合下。

因此，我建议你遵循以下四条规则：

尽可能提供默认参数值

所有错误情况必须处理（例如，参数缺失，类型错误，找不到文件）

所有参数和选项必须有文档

不是立即完成的任务应当显示进度条

**举个简单的例子**
-----------

我们把这些规则应用到一个具体的例子上。这个脚本可以使用凯撒加密法加密和解密消息。  

假设已经有个写好的 encrypt 函数（实现如下），我们需要创建一个简单的脚本，用来加密和解密消息。我们希望让用户通过命令行参数选择加密模式（默认）和解密模式，并选择一个秘钥（默认为 1）。

```
def encrypt(plaintext, key):
    cyphertext = ''
    for character in plaintext:
        if character.isalpha():
            number = ord(character)
            number += key
            if character.isupper():
                if number > ord('Z'):
                    number -= 26
                elif number < ord('A'):
                    number += 26
            elif character.islower():
                if number > ord('z'):
                    number -= 26
                elif number < ord('a'):
                    number += 26
            character = chr(number)
        cyphertext += character

    return cyphertext

```

我们的脚本需要做的第一件事就是获取命令行参数的值。当我搜索 “python command line arguments” 时，出现的第一个结果是关于 sys.argv 的，所以我们来试试这个方法……

**“初学者” 的方法**
-------------

sys.argv 是个列表，包含用户在运行脚本时输入的所有参数（包括脚本名自身）。

例如，如果我输入：

```
> python caesar_script.py --key 23 --decrypt my secret message
pb vhfuhw phvvdjh


```

该列表将包含：

```
['caesar_script.py', '--key', '23', '--decrypt', 'my', 'secret', 'message'] 


```

因此只需遍历该参数列表，找到'--key'（或'-k'）以得到秘钥值，找到'--decrypt'以设置解密模式（实际上只需要使用秘钥的反转作为秘钥即可）。另外，搜索公众号 Linux 就该这样学后台回复 “猴子”，获取一份惊喜礼包。

最后我们的脚本大致如下：

```
import sys

from caesar_encryption import encrypt

def caesar():
    key = 1
    is_error = False

    for index, arg in enumerate(sys.argv):
        if arg in ['--key', '-k'] and len(sys.argv) > index + 1:
            key = int(sys.argv[index + 1])
            del sys.argv[index]
            del sys.argv[index]
            break

    for index, arg in enumerate(sys.argv):
        if arg in ['--encrypt', '-e']:
            del sys.argv[index]
            break
        if arg in ['--decrypt', '-d']:
            key = -key
            del sys.argv[index]
            break

    if len(sys.argv) == 1:
        is_error = True
    else:
        for arg in sys.argv:
            if arg.startswith('-'):
                is_error = True

    if is_error:
        print(f'Usage: python {sys.argv[0]} [ --key <key> ] [ --encrypt|decrypt ] <text>')
    else:
        print(encrypt(' '.join(sys.argv[1:]), key))

if __name__ == '__main__':
    caesar()

```

这个脚本遵循了一些我们前面推荐的规则：

*   支持默认秘钥和默认模式
    
*   基本的错误处理（没有提供输入文本的情况，以及提供了无法识别的参数的情况）
    
*   出错时或者不带任何参数调用脚本时会显示文档：
    

```
> python caesar_script_using_sys_argv.py
Usage: python caesar.py [ --key <key> ] [ --encrypt|decrypt ] <text>


```

但是，这个凯撒加密法脚本太长了（39 行，其中甚至还没包括加密代码本身），而且很难读懂。

解析命令行参数应该还有更好的办法……

**试试 argparse？**
----------------

argparse 是 Python 用来解析命令行参数的标准库。  

我们来看看用 argparse 怎样编写凯撒加密的脚本：

```
import argparse

from caesar_encryption import encrypt

def caesar():
    parser = argparse.ArgumentParser()
    group = parser.add_mutually_exclusive_group()
    group.add_argument('-e', '--encrypt', action='store_true')
    group.add_argument('-d', '--decrypt', action='store_true')
    parser.add_argument('text', nargs='*')
    parser.add_argument('-k', '--key', type=int, default=1)
    args = parser.parse_args()

    text_string = ' '.join(args.text)
    key = args.key
    if args.decrypt:
        key = -key
    cyphertext = encrypt(text_string, key)
    print(cyphertext)

if __name__ == '__main__':
    caesar()

```

这段代码也遵循了上述规则，而且与前面的手工编写的脚本相比，可以提供更准确的文档，以及更具有交互性的错误处理：

```
> python caesar_script_using_argparse.py --encode My message

usage: caesar_script_using_argparse.py [-h] [-e | -d] [-k KEY] [text [text ...]]
caesar_script_using_argparse.py: error: unrecognized arguments: --encode
> python caesar_script_using_argparse.py --help

usage: caesar_script_using_argparse.py [-h] [-e | -d] [-k KEY] [text [text ...]]

```

```
positional arguments:
  text
optional arguments:
  -h, --help         show this help message and exit
  -e, --encrypt
  -d, --decrypt
  -k KEY, --key KEY


```

但是，仔细看了这段代码后，我发现（虽然有点主观）函数开头的几行（从 7 行到 13 行）定义了参数，但定义方式并不太优雅：它太臃肿了，而且完全是程式化的。应该有更描述性、更简洁的方法。

**click 能做得更好！**
----------------

幸运的是，有个 Python 库能提供与 argparse 同样的功能（甚至还能提供更多），它的代码风格更优雅。这个库的名字叫 click。

这里是凯撒加密脚本的第三版，使用了 click：

```
import click

from caesar_encryption import encrypt

@click.command()
@click.argument('text', nargs=-1)
@click.option('--decrypt/--encrypt', '-d/-e')
@click.option('--key', '-k', default=1)
def caesar(text, decrypt, key):
    text_string = ' '.join(text)
    if decrypt:
        key = -key
    cyphertext = encrypt(text_string, key)
    click.echo(cyphertext)

if __name__ == '__main__':
    caesar()

```

注意现在参数和选项都在修饰器里定义，定义好的参数直接作为函数参数提供。

我来解释一下上面代码中的一些地方：

*   脚本参数定义中的 nargs 参数指定了该参数期待的单词的数目（一个用引号括起来的字符串算一个单词）。默认值是 1。这里 nargs=-1 允许接收任意数目的单词。
    
*   --encrypt/--decrypt 这种写法可以定义完全互斥的选项（类似于 argparse 中的 add_mutually_exclusive_group 函数），它将产生一个布尔型参数。
    
*   click.echo 是该库提供的一个工具函数，它的功能与 print 相同，但兼容 Python 2 和 Python 3，还有一些其他功能（如处理颜色等）。
    

**添加一些隐秘性**
-----------

这个脚本的参数（被加密的消息）应当是最高机密。而我们却要求用户直接在终端里输入文本，使得这些文本被记录在命令历史中，这不是很讽刺吗？

解决方法之一就是使用隐藏的提示。或者可以从输入文件中读取文本，对于较长的文本来说更实际一些。或者可以干脆让用户选择。

输出也一样：用户可以保存到文件中，也可以输出到终端。这样就得到了凯撒脚本的最后一个版本：

```
import click

from caesar_encryption import encrypt

@click.command()
@click.option(
    '--input_file',
    type=click.File('r'),
    help='File in which there is the text you want to encrypt/decrypt.'
         'If not provided, a prompt will allow you to type the input text.',
)
@click.option(
    '--output_file',
    type=click.File('w'),
    help='File in which the encrypted / decrypted text will be written.'
         'If not provided, the output text will just be printed.',
)
@click.option(
    '--decrypt/--encrypt',
    '-d/-e',
    help='Whether you want to encrypt the input text or decrypt it.'
)
@click.option(
    '--key',
    '-k',
    default=1,
    help='The numeric key to use for the caesar encryption / decryption.'
)
def caesar(input_file, output_file, decrypt, key):
    if input_file:
        text = input_file.read()
    else:
        text = click.prompt('Enter a text', hide_input=not decrypt)
    if decrypt:
        key = -key
    cyphertext = encrypt(text, key)
    if output_file:
        output_file.write(cyphertext)
    else:
        click.echo(cyphertext)

if __name__ == '__main__':
    caesar()

```

这个版本有什么新东西吗？

*   首先，注意到我给每个参数选项都加了个 help 参数。由于脚本变得复杂了，help 参数可以给脚本的行为添加一些文档。运行结果如下：
    

```
> python caesar_script_v2.py --help
Usage: caesar_script_v2.py [OPTIONS]
Options:
  --input_file FILENAME          File in which there is the text you want to encrypt/decrypt. If not provided, a prompt will allow you to type the input text.
  --output_file FILENAME         File in which the encrypted/decrypted text will be written. If not provided, the output text will just be printed.
  -d, --decrypt / -e, --encrypt  Whether you want to encrypt the input text or decrypt it.
  -k, --key INTEGER              The numeric key to use for the caesar encryption / decryption.
  --help                         Show this message and exit.


```

*   两个新的参数：input_file 和 output_file，类型均为 click.File。该库能够用正确的模式打开文件，处理可能的错误，再执行函数。例如：
    

```
> python caesar_script_v2.py --decrypt --input_file wrong_file.txt
Usage: caesar_script_v2.py [OPTIONS]
Error: Invalid value for "--input_file": Could not open file: wrong_file.txt: No such file or directory


```

*   正像 help 文本中解释的那样，如果没有提供 input_file，就使用 click.promp 让用户直接在提示符下输入文本，在加密模式下这些文本是隐藏的。如下所示：
    

```
> python caesar_script_v2.py --encrypt --key 2
Enter a text: **************
yyy.ukectc.eqo

```