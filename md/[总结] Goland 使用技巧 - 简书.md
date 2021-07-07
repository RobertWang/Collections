> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/17b65fd38ece)

Goland 是 JetBrains 公司出品的一款开发 Golang 的工具。由于本人是 JetBrains 公司产品的重度使用者，好多都作为主力开发工具（IntellijI IDEA、PyCharm、WebStorm 等），所以 Goland 一经推出，就成为我开发 Golang 的首选 IDE。

如果你之前使用过 Intellij IDEA 开发 Java 程序，那么使用 Goland 几乎可以做到无缝衔接、直接上手。两者的项目组织结构、快捷键等好多特性都可以通用。  
笔者之前总结过一些[使用 Intellij IDEA 的技巧](https://www.jianshu.com/p/e226c085ce69)。有部分在 Goland 下同样适用，在此就不再赘述，下面仅总结一些 Goland 下特有的部分。

注释（"//"）不在行首显示，而是自动跟着代码缩进。
--------------------------

在 Goland 下，注释默认是在行首的，并且其并没有提供修改注释 style 的选项。

![](http://upload-images.jianshu.io/upload_images/54256-861c00f4ed7d937b.png) image.png

可以通过修改配置文件的方式解决此问题：  
第一步：在 “Code Style -> Go” 下导出配置文件。

![](http://upload-images.jianshu.io/upload_images/54256-af4737b6b9e2fe99.png) image.png

打开配置文件添加如下语句：

```
<code_scheme >
  <codeStyleSettings language="go">
    <indentOptions>
      <option  />
    </indentOptions>
    <!-- 添加下面4行 -->
    <option  />
    <option  />
    <option  />
    <option  />
  </codeStyleSettings>
</code_scheme>
```

保存后，从新导入到 Goland 中：

![](http://upload-images.jianshu.io/upload_images/54256-89310b38d2a779e1.png) image.png

这样注释就不会每次都在行首了：

![](http://upload-images.jianshu.io/upload_images/54256-208518e370d728eb.png) image.png

##### Update 2019.01.24:

上面是 Goland 2017.3 版本的配置方法，而最新版本 (2018.3) 的注释默认就是不在行数显示了，上述配置方法已不需要。另外，新版本也可以自由配置注释后面是否跟一个空格了，看着舒爽了很多。

![](http://upload-images.jianshu.io/upload_images/54256-805290a90a765d64.png) 注释后跟一个空格. png

Goland 快捷键
----------

大部分快捷键和 intellij idea 通用（[使用 Intellij IDEA 的技巧](https://www.jianshu.com/p/e226c085ce69)）。下面仅总结 Goland 特有的快捷键：  
`CMD+N`：新建，如果鼠标焦点在 struct 上，会弹出提示是否实现某个接口。  

![](http://upload-images.jianshu.io/upload_images/54256-f430c54d82d7053e.png) image.png

设置变量函数等的搜索范围
------------

由于 golang 语言特有的包管理方式，几乎所有 go 代码文件都在 GOPATH 路径下面。如果在关键字（变量函数等）上使用`cmd+b`快捷键搜索其引用时，默认是搜索整个 GOPATH 路径下的所有文件，这样会造成搜索出一些与本工程不相关的内容。此时可以自定义配置，仅搜索本工程内的文件：按`cmd+alt+shift+f7`弹出配置对话框，添加一个自定义 scope：  

![](http://upload-images.jianshu.io/upload_images/54256-0cbc3d6642a27b9b.png) image.png

![](http://upload-images.jianshu.io/upload_images/54256-426e2cacf9d53fce.png) image.png

![](http://upload-images.jianshu.io/upload_images/54256-d729df18aa108d24.png) image.png

如果想要在搜索范围中排除一些文件，比如

`_test.go`

类型的文件，可如下设置：

```
!file[quorum]:*//*_test.go
```

![](http://upload-images.jianshu.io/upload_images/54256-279168a7ebee153d.png) image.png

搜索时排除指定文件，比如_test.go 文件
-----------------------

排除的文件在 “File mask” 选项中使用! 开头，如图：

![](http://upload-images.jianshu.io/upload_images/54256-a6b5cf68f8250ad3.png) image.png