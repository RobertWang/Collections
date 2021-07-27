> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zixizixi.cn](https://zixizixi.cn/vscode-extension-vsix-install)

> 最新迷上了使用 VSCode 敲代码，在网上搜集了大量实用扩展组件，VSCode 支持离线安装扩展，但一次只能安装一个，很麻烦，故在此记录及分享一下 VSCode 扩展组件的离线安装方法及在 Windows 系统下批量安装扩展的 bat 脚本。

目录

离线安装 VSCode 扩展组件方法及批量安装脚本分享

离线安装 VSCode 扩展组件方法及批量安装脚本分享
---------------------------

> 本文最后更新于 `782` 天前，内容可能已经不够准确，请酌情参考！

最新迷上了使用 VSCode 敲代码，在网上搜集了大量实用扩展组件，VSCode 支持离线安装扩展，但一次只能安装一个，很麻烦，故在此记录及分享一下 VSCode 扩展组件的离线安装方法及在 Windows 系统下批量安装扩展的 bat 脚本。

一、下载离线安装扩展包
-----------

*   VSCode 扩展安装 / 下载地址：[https://marketplace.visualstudio.com/](https://marketplace.visualstudio.com/)
*   下载方法：
    1.  搜索要下载的扩展名称：  
        ![](https://img.hacpai.com/file/2019/06/image-ad72a1f3.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)  
        ![](https://img.hacpai.com/file/2019/06/image-a955c87b.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)
        
    2.  进入扩展详情页下载离线安装包：  
        ![](https://img.hacpai.com/file/2019/06/image-c64cd582.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)  
        点击右侧的 “**Download Extension**” 开始下载。
        

二、安装离线扩展包
---------

### 方法一：在 VSCode 界面安装

1.  打开 VSCode，点击扩展（EXTENSION）右侧的更多选项符号 `···`，选择 `从 VSIX 安装...`（`install from VXIS...`）  
    ![](https://img.hacpai.com/file/2019/06/image-88a14184.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)
    
2.  选择对应的扩展包，点击 “**安装**” 按钮，然后重启 VSCode 即可。  
    ![](https://img.hacpai.com/file/2019/06/image-ae0e4698.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)
    

### 方法二：通过命令行安装

1.  将扩展包 `.vsix` 文件放入 `%VSCode_HOME%\bin` 目录中：  
    ![](https://img.hacpai.com/file/2019/06/image-02a31638.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)
    
2.  在当前目录打开命令行，执行以下命令：
    
    ```
    	code --install-extension 扩展包名称.vsix
    
    
    ```
    
    ![](https://img.hacpai.com/file/2019/06/image-728b83c6.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)  
    当输出 `Extension '扩展包名称.vsix' was successfully installed.` 则说明安装成功。
    

三、Windows 批量安装扩展脚本
------------------

1.  脚本内容：
    
    ```
    	@echo off
    	title 批量安装 VSCode 扩展
    	echo 安装当前目录下所有的 VSCode 扩展组件...
    	echo.
    	D:
    	cd D:\Development\Tools\VSCode\bin
    	for %%1 in (*.vsix) do code --install-extension %%1
    
    
    ```
    
2.  运行效果：  
    ![](https://img.hacpai.com/file/2019/06/image-61a3a749.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)