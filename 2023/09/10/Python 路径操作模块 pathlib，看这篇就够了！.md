> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/475661402)

1 pathlib 简介
------------

pathlib 是跨平台的、面向对象的路径操作模块，可适用于不同的操作系统，其操作对象是各种操作系统中使用的路径（包括绝对路径和相对路径），pathlib 有两个主要的类，分别为 PurePath 和 Path。

### 1）PurePath

PurePath 访问实际文件系统的 “纯路径”，只负责对路径字符串执行操作。PurePath 有两个子类，即 PurePosixPath 和 PathWindowsPath，前者用于操作 UNIX（包括 Mac OS X）风格的路径，后者用于操作 Windows 风格的路径。

### 2）Path

Path 访问实际文件系统的 “真正路径”，Path 对象可用于判断对应的文件是否存在、是否为文件、是否为目录等。有两个子类，即 PosixPath 和 WindowsPath，前者用于操作 UNIX（包括 Mac OS X）风格的路径，后者用于操作 Windows 风格的路径。

### 3）PurePath 和 Path 的区别

Path 是 PurePath 的子类，除了支持 PurePath 的各种操作、属性和方法之外，还会真正访问底层的文件系统，包括判断 Path 对应的路径是否存在，获取 Path 对应路径的各种属性（如是否只读、是文件还是文件夹等），甚至可以对文件进行读写。

PurePath 和 Path 最根本的区别在于，PurePath 处理的仅是字符串，而 Path 则会真正访问底层的文件路径，因此它提供了属性和方法来访问底层的文件系统。

### 4）UNIX 和 Windows 风格路径区别

UNIX 风格的路径和 Windows 风格路径的主要区别在于根路径和路径分隔符，UNIX 风格路径的根路径是斜杠（/），而 Windows 风格路径的根路径是盘符（c:）；UNIX 风格的路径的分隔符是斜杠（/），而 Windows 风格路径的分隔符是反斜杠（\）。

**注意：**

考虑到操作系统的不同，在使用 PurePath 类时，如果在 UNIX 或 Mac OS X 系统上使用 PurePath 创建对象，该类的构造方法实际返回的是 PurePosixPath 对象；反之，如果在 Windows 系统上使用 PurePath 创建对象，该类的构造方法返回的是 PureWindowsPath 对象。

**考虑到操作系统的不同，Path 类的使用同 PurePath 类。**

2 pathlib 与 os 的区别
------------------

在 Python 3.4 之前，涉及路径相关操作，都用 os 模块解决，尤其是 os.path 这个子模块非常有用。在 Python 3.4 之后，pathlib 成为标准库模块，其使用面向对象的编程方式来表示文件系统路径，丰富了路径处理的方法。

### 1）pathlib 优势

**相对于传统的 os 及 os.path，pathlib 具体如下优势：**

*   pathlib 实现统一管理，解决了传统操作导入模块不统一问题；
*   pathlib 使得在不同操作系统之间切换非常简单；
*   pathlib 是面向对象的，路径处理更灵活方便，解决了传统路径和字符串并不等价的问题；
*   pathlib 简化了很多操作，简单易用。

### 2）pathlib 和 os 常用操作对比

通过常用路径操作的对比，可以更深刻理解 pathlib 和 os 的区别，便于在实际操作中做对照，也便于进行使用替代，详细对比如下：

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><th>pathlib 操作</th><th>os 及 os.path 操作</th><th>功能描述</th></tr><tr><td>Path.resolve()</td><td>os.path.abspath()</td><td>获得绝对路径</td></tr><tr><td>Path.chmod()</td><td>os.chmod()</td><td>修改文件权限和时间戳</td></tr><tr><td>Path.mkdir()</td><td>os.mkdir()</td><td>创建目录</td></tr><tr><td>Path.rename()</td><td>os.rename()</td><td>文件或文件夹重命名，如果路径不同，会移动并重新命名</td></tr><tr><td>Path.replace()</td><td>os.replace()</td><td>文件或文件夹重命名，如果路径不同，会移动并重新命名，如果存在，则破坏现有目标。</td></tr><tr><td>Path.rmdir()</td><td>os.rmdir()</td><td>删除目录</td></tr><tr><td>Path.unlink()</td><td>os.remove()</td><td>删除一个文件</td></tr><tr><td>Path.unlink()</td><td>os.unlink()</td><td>删除一个文件</td></tr><tr><td>Path.cwd()</td><td>os.getcwd()</td><td>获得当前工作目录</td></tr><tr><td>Path.exists()</td><td>os.path.exists()</td><td>判断是否存在文件或目录 name</td></tr><tr><td>Path.home()</td><td>os.path.expanduser()</td><td>返回电脑的用户目录</td></tr><tr><td>Path.is_dir()</td><td>os.path.isdir()</td><td>检验给出的路径是一个文件</td></tr><tr><td>Path.is_file()</td><td>os.path.isfile()</td><td>检验给出的路径是一个目录</td></tr><tr><td>Path.is_symlink()</td><td>os.path.islink()</td><td>检验给出的路径是一个符号链接</td></tr><tr><td>Path.stat()</td><td>os.stat()</td><td>获得文件属性</td></tr><tr><td>PurePath.is_absolute()</td><td>os.path.isabs()</td><td>判断是否为绝对路径</td></tr><tr><td>PurePath.joinpath()</td><td>os.path.join()</td><td>连接目录与文件名或目录</td></tr><tr><td>PurePath.name</td><td>os.path.basename()</td><td>返回文件名</td></tr><tr><td>PurePath.parent</td><td>os.path.dirname()</td><td>返回文件路径</td></tr><tr><td>Path.samefile()</td><td>os.path.samefile()</td><td>判断两个路径是否相同</td></tr><tr><td>PurePath.suffix</td><td>os.path.splitext()</td><td>分离文件名和扩展名</td></tr></tbody></table>

3 具体使用方法
--------

_**以下操作是在 Windows 系统内完成的。**_

### 1）路径获取

**传入字符串**

在创建 PurePath 和 Path 时，既可以传入单个字符串，也可传入多个路径字符串，PurePath 会将它们拼接成一个字符串。

```
from pathlib import *

PurePath('M工具箱','MTool工具/示例','info')
# 结果： PureWindowsPath('M工具箱/MTool工具/示例/info')

Path('M工具箱')
# 结果： WindowsPath('M工具箱')

input_path = r"C:\Users\okmfj\Desktop\MTool工具"
Path(input_path)
# 结果： WindowsPath('C:/Users/okmfj/Desktop/MTool工具')

```

**获取目录**

*   Path.cwd()，返回文件当前所在目录；
*   Path.home()，返回电脑用户的目录。

```
from pathlib import *

Path.cwd()
# 结果： WindowsPath('D:/M工具箱')

Path.home()
# 结果： WindowsPath('C:/Users/okmfj')

```

**目录拼接**

拼接出 Windows 桌面路径，当前路径下的子目录或文件路径。

```
from pathlib import *

# 拼接出Windows桌面路径
Path(Path.home(), "Desktop")
# 结果： WindowsPath('C:/Users/okmfj/Desktop')

# 拼接出Windows桌面路径
Path.joinpath(Path.home(), "Desktop")
# 结果： WindowsPath('C:/Users/okmfj/Desktop')

# 拼接出当前路径下的“MTool工具”子文件夹路径
Path.cwd() / 'MTool工具'
# 结果： WindowsPath('D:/M工具箱/MTool工具')

```

### **2）路径处理**

获取路径的不同部分、或不同字段等内容，用于后续的路径处理，如拼接路径、修改文件名、更改文件后缀等。

```
from pathlib import *

input_path = r"C:\Users\okmfj\Desktop\MTool工具"

Path(input_path).name  # 返回文件名+文件后缀


Path(input_path).stem  # 返回文件名

Path(input_path).suffix  # 返回文件后缀

Path(input_path).suffixes  # 返回文件后缀列表


Path(input_path).root  # 返回根目录


Path(input_path).parts  # 返回文件

Path(input_path).anchor  # 返回根目录

Path(input_path).parent  # 返回父级目录

Path(input_path).parents  # 返回所有上级目录的列表


```

### 3）路径判断

*   Path.exists()，判断 Path 路径是否是一个已存在的文件或文件夹；
*   Path.is_dir()，判断 Path 是否是一个文件夹；
*   Path.is_file()，判断 Path 是否是一个文件。

```
from pathlib import *

input_path = r"C:\Users\okmfj\Desktop\MTool工具"

if Path(input_path ).exists():
	if Path(input_path ).is_file():
		print("是文件！")
	elif Path(input_path ).is_dar():
		print("是文件夹！")
else:
	print("路径有误，请检查!")

```

### 4）路径建立、删除

*   Path.mkdir()，创建文件夹；
*   Path.rmdir()，删除文件夹，文件夹必须为空；
*   Path.unlink()，删除文件。

```
from pathlib import *

input_path = r"C:\Users\okmfj\Desktop\MTool工具"

# 创建文件夹函数
def creat_dir(dir_path):
	if Path(dir_path).exists():  # 如果已经存在，则跳过并提示
		print("文件夹已经存在！")
	else:
		Path.mkdir(Path(dir_path))  # 创建文件夹

creat_dir(input_path )  # 调用函数

# 删除文件夹函数
def del_dir(dir_path):
	if Path(dir_path).exists():  # 如果存在，则删除文件夹
		Path.rmdir(Path(dir_path))
	else:
		print("文件夹不存在！")  # 文件夹不存在

del_dir(input_path )  # 调用函数

# 删除文件函数
def del_file(dir_path):
	if Path(dir_path).exists():  # 如果存在，则删除文件
		Path.unlink()(Path(dir_path))
	else:
		print("文件不存在！")  # 文件不存在

del_file(input_path )  # 调用函数

```

### 5）路径匹配查找（迭代）

*   Path.iterdir()，查找文件夹下的所有文件，返回的是一个生成器类型；
*   Path.glob(pattern)，查找文件夹下所有与 pattern 匹配的文件，返回的是一个生成器类型；
*   Path.rglob(pattern)，查找文件夹下所有子文件夹中与 pattern 匹配的文件，返回的是一个生成器类型。

**iterdir 方法**

```
from pathlib import *

input_path = r"C:\Users\okmfj\Desktop\MTool工具"

# 获取文件夹下所有文件，返回文件路径（字符）列表，采用iterdir方法
[str(f) for f in Path(x).iterdir() if Path(f).is_file()]

# 获取文件夹下所有文件类型，返回文件后缀的列表，采用iterdir方法
list(set([Path(f).suffix for f in Path(x).iterdir() if Path(f).is_file()]))
# 结果： ['.pdf', '.txt', '.docx']

```

**glob 方法**

```
from pathlib import *

input_path = r"C:\Users\okmfj\Desktop\MTool工具"

# 获取文件夹下所有文件，返回文件路径（字符）列表，采用glog方法
[str(f) for f in Path(input_path).glob(f"*.*") if Path(f).is_file()]

# 获取文件夹下所有文件，返回文件路径（字符）列表，采用glog方法
[str(f) for f in Path(input_path).glob(f"**\*.*") if Path(f).is_file()]

# 获取文件夹下所有文件类型，返回文件后缀的列表，采用glog方法
list(set([Path(f).suffix for f in Path(input_path).glob(f"*.*") if Path(f).is_file()]))
# 结果： ['.pdf', '.txt', '.docx']

# 获取文件夹下（含子文件）所有文件类型，返回文件后缀的列表，采用glog方法
list(set([Path(f).suffix for f in Path(x).glob(f"**\*.*") if Path(f).is_file()]))
# 结果： ['.pdf', '.txt', '.docx']

```

**rglob 方法**

```
from pathlib import *

input_path = r"C:\Users\okmfj\Desktop\MTool工具"

# 获取文件夹下（含子文件）所有文件，返回文件路径(字符)列表，采用rglog方法
[str(f) for f in Path(x).rglob(f"*.*") if Path(f).is_file()]

# 获取文件夹下（含子文件）所有文件类型，返回文件后缀的列表，采用rglog方法
list(set([Path(f).suffix for f in Path(x).rglob(f"*.*") if Path(f).is_file()]))
# 结果： ['.pdf', '.txt', '.docx']

```

### 6）读、写文件

*   Path.open(mode='r')，以 "r" 格式打开 Path 路径下的文件，若文件不存在即创建后打开。
*   Path.read_bytes()，打开 Path 路径下的文件，以字节流格式读取文件内容，等同 open 操作文件的 "rb" 格式。
*   Path.read_text()，打开 Path 路径下的文件，以 str 格式读取文件内容，等同 open 操作文件的 "r" 格式。
*   Path.write_bytes()，对 Path 路径下的文件进行写操作，等同 open 操作文件的 "wb" 格式。
*   Path.write_text()，对 Path 路径下的文件进行写操作，等同 open 操作文件的 "w" 格式。

![](https://pic4.zhimg.com/v2-6575c60a57fbf05cf64144b74d8f03ab_r.jpg)

```
from pathlib import *

input_path = r"C:\Users\okmfj\Desktop\MTool工具\1.txt"

with Path(input_path).open('w') as f:  # 创建并打开文件
	f.write('M工具箱')  # 写入内容
f = open(input_path, 'r')  # 读取内容
print(f"读取的文件内容为：{f.read()}")
f.close()

```

### 7）属性和方法汇总

**PurePath 类属性和方法汇总**

![](https://pic3.zhimg.com/v2-dfcb4f5442973aa7f8058075f949660e_r.jpg)

**Path 类属性和方法汇总**

![](https://pic2.zhimg.com/v2-2983703aa2b4632016e25c4fe5a8b0d9_r.jpg)

4 应用实例
------

### 1）实例 1：获取文件列表（函数）

通过函数，返回文件列表，为后续迭代处理做好准备。

**参数：**

*   path：输入路径，支持文件路径和文件夹路径。
*   sub_dir：当为 True 时含子目录，为 False 时不含子目录
*   suffix_list：文件类型列表，按要求的列出全部符合条件的文件，为空时列出全部文件，如：[".xlsx",".xls"]

```
from pathlib import *

def file_list_handle(path, sub_dir=False, suffix_list=[]):
	"""
	path：输入路径，支持文件路径和文件夹路径	
        sub_dir：当为True时含子目录，为False时不含子目录 	
        suffix_list：文件类型列表，按要求的列出全部符合条件的文件，为空时列出全部文件，如：[".xlsx",".xls"]
	"""
	file_list = []
	if not suffix_list:
		if sub_dir:
			[suffix_list.append(Path(f).suffix) for f in Path(path).glob(f"**\*.*") if Path(f).is_file()]
		else:
			[suffix_list.append(Path(f).suffix) for f in Path(path).glob(f"*.*") if Path(f).is_file()]
		suffix_list = list(set(suffix_list))
	# [print(i) for i in suffix_list]
	if Path(path).exists():
		# 目标为文件夹
		if Path(path).is_dir():
			if sub_dir:
				for i in suffix_list:
					[file_list.append(str(f)) for f in Path(path).glob(f"**\*{i}")]
			else:
				[file_list.append(str(f)) for f in Path(path).iterdir() if Path(f).is_file and f.suffix in suffix_list]
		elif Path(path).is_file():
			file_list = [path]

		# 去除临时文件
		file_list_temp = []
		for y in file_list:
			if "~$" in Path(y).stem:
				continue
			file_list_temp.append(y)
		file_list = file_list_temp
		return file_list
	else:
		print("输入有误！")
		return []

```

### 2）实例 2：定义文件路径

在输入文件路径的基础上，通过函数，构造输出文件路径，为后续处理、存储做好准备。

**参数：**

*   in_path：输入文件路径
*   path_str：文件路径构造字符
*   file_suffix：输出文件后缀

```
from pathlib import *
from datetime import datetime

input_path = r"C:\Users\hp\Desktop\MTool工具\1.txt"

def out_path_handle(in_path, path_str="", file_suffix=""):
	"""
	in_path：输入文件路径
        path_str：文件路径构造字符
        file_suffix：输出文件后缀
	"""
	if path_str == "":
		path_str = str(datetime.now()).replace(":", "").replace(" ", "_").replace("-", "").replace(".", "_")[
		           :22]
	elif path_str[-1] == "_":
		path_str = path_str + str(datetime.now()).replace(":", "").replace(" ", "_").replace("-", "").replace(
			".", "_")[:22]
	if file_suffix == "":
		file_suffix = Path(in_path).suffix

	if Path(in_path).exists():
		if Path(in_path).is_dir():
			path = Path(in_path).joinpath(path_str + file_suffix)
		elif Path(in_path).is_file():
			path = Path(in_path).parent.joinpath(Path(in_path).stem + "_" + path_str + file_suffix)
		return path

	else:
		print("输入有误！")
		return []

# 调用函数
out_path_handle(input_path,"已处理",".xlsx")
# 结果： WindowsPath('C:/Users/hp/Desktop/MTool工具/1_已处理.xlsx')

# 调用函数
out_path_handle(input_path,"_",".xlsx")
# 结果：WindowsPath('C:/Users/hp/Desktop/MTool工具/1_20220304_135344_003210.xlsx')


```

5 总结
----

以上详细介绍了 Python 中路径操作的标准模块：pathlib，鉴于 pathlib 的优势，建议在 python 中优先使用，可以使路径处理更简洁、路径处理错误更少、编程效率更高、代码更易读。

其他未尽情况，可参见官方文档：[pathlib - pathlib 1.0.1 documentation](https://link.zhihu.com/?target=https%3A//pathlib.readthedocs.io/en/pep428/)