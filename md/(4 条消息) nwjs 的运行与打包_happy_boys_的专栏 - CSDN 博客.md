> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/happy_boys_/article/details/80868785?locationNum=13&fps=1)

nwjs 的运行与打包
===========

创建 nwjs 项目
----------

### 完整目录结构如下

```
nwjsDemo
  -index.html
  -package.json
```

index.html

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>我的nwjs示例</title>
</head>
<body >
<h1>你好，中国</h1>
</body>
</html>
```

package.json

```
{
    "main": "index.html",
    "name": "我的nwjs示例",
    "nodejs": true,
    "single-instance": true,
    "window": {
        "title": "我的nwjs示例",
        "width": 1000,
        "height": 500,
        "position": "center",
        "min_width": 0,
        "min_height": 0,
        "max_width": 0,
        "max_height": 0,
        "resizable": true,
        "always_on_top": false,
        "fullscreen": false,
        "show_in_taskbar": true,
        "frame": true,
        "show": true,
        "transparent": false,
        "kiosk": false
    },
    "webkit": {
        "plugin": false,
        "page-cache": false
    }
}
```

### 执行与打包

[由于 nwjs 运行与打包都挺麻烦的这里提供运行和打包的 shell 脚本 run.sh 和 package.sh](http://xn--nwjsshellrun-qy4s19cp5gszrba151euq7eda269bkzdoz1ae78czowae2hfa9996ezu1bia8470dja80am86aelhrz4k.xn--shpackage-rw9o.sh)。把这两个脚本文件放入工程目录下（注意修改 nw 的路径）。

```
开发过程中执行工程运行run.sh
打包运行package.sh
```

```
nwjsDemo
  -index.html
  -package.json
  -run.sh
  -package.sh
```

[run.sh](http://run.sh)

```
# nwjs可执行文件
nw=~/apps/nwjs-sdk-v0.31.4-linux-x64/nw

# 进入工程目录
cd `dirname $0`
# 工程目录
rootPath=`pwd`
$nw $rootPath
```

run.bat

```
echo off
rem nwjs可执行文件
set nw="D:\Program Files\nwjs-sdk-v0.31.4-win-x64\nw"

rem 进入工程目录
cd /d %~dp0
rem 工程目录
set rootPath=%cd%
%nw% %rootPath%
```

[package.sh](http://package.sh)

```
# nwjs可执行文件
nw=~/apps/nwjs-sdk-v0.31.4-linux-x64/nw

# 进入工程目录
cd `dirname $0`
# 工程目录
rootPath=`pwd`
# 工程名称
projectName=${rootPath##*/}

# 打包当前目录文件为zip
zip -r ./target.zip ./*

# 删除target目录
rm -rf target
# 创建target目录
mkdir target
cp -r ${nw%/*}/* target
mv target.zip target/target.nw
cd target
cat $nw target.nw > target && chmod + x app
rm target.nw
mv target $projectName
$projectName
```

package.bat

```
rem copy /b nw.exe+app.nw app.exe
rem nwjs可执行文件
set nw=D:\Program Files\nwjs-sdk-v0.31.4-win-x64
set zip=C:\Program Files (x86)\360\360zip\360zip.exe

rem 进入工程目录
cd /d %~dp0
rem 工程目录
set rootPath=%cd%
cd ..
set rootPathParent=%cd%
rem 工程名称
setlocal enabledelayedexpansion
set projectName=!rootPath:%rootPathParent%\=!
echo %cd%

cd %projectName%

echo %cd%

rem 打包当前目录文件为zip
rem zip -r ./target.zip ./*
jar cvf target.jar *

rem 静默删除target目录
rmdir /s /q target

rem 创建target目录
mkdir target

rem 拷贝nw文件到target
xcopy "%nw%" target /e/h

rem 移动工程文件到target
move /Y target.jar target/target.jar
cd target
rem 重命名
ren target.jar target.nw

copy /b "%nw%\nw.exe"+target.nw target.exe

del /q /f /s target.nw
del /q /f /s nw.exe
```

```
注意：nwjs有两个版本NORMAL和SDK。
开发请使用SDK，因为可以debug。
打包使用NORMAL，不支持debug
```