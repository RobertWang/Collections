> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/asafield/article/details/126610886)

### 文章目录

*   *   [情况一：导入的包为标准库中的包](#_10)
    *   [情况二：导入的包为网络上的第三方包](#_17)
    *   [情况三：导入的包为本地自定义的包](#_26)

`import "./mypackage"`

这种格式往往就会出现以下报错：

```
local import "./XXX" in non-local package

```

在查询资料发现有人说以前的版本这样是可以的，不过反正现在（我的 go 版本为`1.18.3`）是不行的。解决的方法也很简单，只要按照一定的格式导入包就可以了，尽管是比较基础的内容，但对于像我一样的初学者，难免有疑惑，下面具体分析。

情况一：导入的包为标准库中的包
---------------

当导入的包为`fmt`或者`os`这类包含在标准库中的包时，可以直接通过以下方式导入：

```
//import "包名" ，例如：
import "fmt"

```

情况二：导入的包为网络上的第三方包
-----------------

当导入的包既不是标准包，也不是在本地自定义的包时，可以通过与情况一相似的格式直接调用。如这个教程中所示：  
[go 语言之旅](https://tour.go-zh.org/moretypes/18)  
上面的例子需要设置好环境变量`GOPROXY`的值为`https://goproxy.cn`，不然在国内可能由于网络问题而出现错误。  
因此，这种情况下，导入包的格式与情况一相同：

```
//import "网址/包名”，例如：
import "golang.org/x/tour/pic"

```

情况三：导入的包为本地自定义的包
----------------

当导入的包属于本地自定义的包时，也分为两种情况进行说明：

*   导入的包位于同一`module`下时：  
    当需要导入包的源文件与该包属于同一个`module`（module 的概念可以看我的前一篇[博客](https://blog.csdn.net/asafield/article/details/126587525?spm=1001.2014.3001.5501)，需要理解`go mod`的用法）时，可以通过`import "模块名/包名"`的方式导入：
    
    ```
    //import "模块名/包名"，例如：
    import "moduleName/packageName"
    
    ```
    
    下面的图片举例更加直观：  
    ![](https://img-blog.csdnimg.cn/37df246563504652aa0407fcdde06a99.jpeg#pic_center)
*   导入的包位于不同的`module`下时：  
    当需要导入包的源文件与该包属于不同的`module`时，按照上面的方法就无法定位到正确的位置了，究其原因，是因为要导入一个包，就需要知道它所在的位置，对于本地自定义的包来说，也就是需要知道存放包的路径。现在的`go`版本不再支持在源文件中直接通过`import`相对路径的方法查找包了，因此就需要通过其它方式确定包所在的位置，而这种方法就是通过配置`go.mod`实现的。  
    `go.mod`文件正是对某个`module`需要的依赖包进行管理的文件，里面配置的详细方法可以参照我的上一篇[博客](https://blog.csdn.net/asafield/article/details/126587525?spm=1001.2014.3001.5501)。这里给出具体的解决方式：
    
    1.  修改`go.mod`文件，添加以下内容：
        
        ```
        require packagename v0.0.0 //packagename为包的名称
        replace packagename => dir //dir修改为包所在的相对路径。如：“../dir”
        
        ```
        
    2.  在源文件中导入包：
        
        ```
        import "packagename"
        
        ```
        
    
    下面的图片中的例子更加直观：  
    ![](https://img-blog.csdnimg.cn/875104f9319047a486856155b8ff061d.jpeg#pic_center)  
    这样，就完成了对自定义的本地包的导入。