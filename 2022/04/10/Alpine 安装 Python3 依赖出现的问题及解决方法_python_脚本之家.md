> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jb51.net](https://www.jb51.net/article/202876.htm)

> 这篇文章主要介绍了 Alpine 安装 Python3 依赖出现的问题及解决方法, 本文给大家介绍的非常详细，对大家的学习或工作具有一定的参考借鉴价值，需要的朋友可以参考下

这篇文章主要介绍了 Alpine 安装 Python3 依赖出现的问题及解决方法, 本文给大家介绍的非常详细，对大家的学习或工作具有一定的参考借鉴价值，需要的朋友可以参考下

apk 换源

```
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
```

安装 Python 的构建环境

```
apk add --no-cache --virtual build-dependencies \
python3-dev \
libffi-dev \
openssl-dev \
gcc \
libc-dev \
make
```

安装 Python 依赖包 `ImportError: cannot import name 'Feature' from 'setuptools'`

```
pip install --upgrade pip setuptools==45.2.0 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

`ModuleNotFoundError: No module named 'Cython'`

```
pip install cython -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**pymssql 安装不上**

`command 'gcc' failed with exit status 1`

后面发现是漏装了一个环境`freetds-dev`  
重新安装之后，就能成功安装依赖了

`apk add freetds-dev`

注意的是，依赖成功安装之后，如果为了 docker 镜像大小，卸载了`freetds-dev`这个环境包，会导致访问数据库的时候报错`libsybdb.so.5: cannot open shared object file: No such file or directory`

**grpcio 安装不上**

和上面一样，漏了环境`build-base linux-headers`

执行`apk add build-base linux-headers`之后，就能成功安装

**Pillow 安装不上**

和上面一样，漏了环境`jpeg-dev zlib-dev`

执行`apk add jpeg-dev zlib-dev`之后，就能成功安装

最后卸载依赖

```
apk del build-dependencies
```

到此这篇关于 Alpine 安装 Python3 依赖出现的问题及解决方法的文章就介绍到这了, 更多相关 Alpine 安装 Python3 依赖内容请搜索脚本之家以前的文章或继续浏览下面的相关文章希望大家以后多多支持脚本之家！