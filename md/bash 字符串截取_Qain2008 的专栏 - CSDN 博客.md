> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Qain2008/article/details/84471002)

命令的 2 种替换形式 $() 和 ``  
示例：截断字符串      
a):  
    # 截取文件名称  
    var1=$(basename /home/aimybbe/bash/test.sh)  
    echo $var1  
      
    # 截取目录  
    var2=$(dirname /home/aimybbe/bash/test.sh)  
    echo $var2  
b):  
    var1=`basename /home/aimybbe/bash/test.sh`  
    echo $var1  
      
    var2=$(dirname /home/aimybbe/bash/test.sh)  
    echo $var2  
      
更专业的[字符串截取](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%88%AA%E5%8F%96&spm=1001.2101.3001.7020)方法：  
示例：testfile.tar.gz  
a) 获取后缀名  
你想截取 tar.gz  
    filename=testfile.tar.gz  
    file=${filename#*.}  
    echo $file  
你想截取 gz  
    filename=testfile.tar.gz  
    file=${filename##*.}  
    echo $file  
说明：  
这里的 ${filename##*.} 什么意思呢？在 ${ } 中输入环境变量名称，两个 ##(或一个 #)，然后是通配符 ("*.")。  
然后，bash 取得 filename，找到从字符串 "testfile.tar.gz" 开始处开始、且匹配通配符 "*." 的最长子字符串 (或最短)，然后将其从字符串的开始处截去。  
注意：  
1.# 意思是从字符串的开始处开始截取。  
2. 两个 ## 代表匹配的最大长度，一个 #代表匹配的最小长度 (也就是说这里不是一个 #匹配一个‘.’)  
b) 获取文件名称 (也就是去除后缀名)  
你想截取 testfile.tar  
    filename=testfile.tar.gz  
    file=${filename%.*}  
    echo $file  
你想截取 testfile  
    filename=testfile.tar.gz  
    file=${filename%%.*}  
    echo $file      
注意：  
1. 这个方法和上面原理相同 % 就是从末尾字符串开始截取，%% 就是最大长度,% 就是最小长度  
c) 截取任意的字符  
你想截取 file  
    filename=testfile.tar.gz  
    file=${filename:4:4}  
    echo $file  
      
你想截取 test  
    filename=testfile.tar.gz  
    file=${filename:0:4}  
    echo $file  
说明：  
格式为 ${filename::} 第一个':'后面的数字是字符串的索引从左边开始，索引从 0 开始，第二个':'后面的数字是长度，两处的数字都是十进制数值。