> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/wanxiaoderen/article/details/82388091?utm_medium=distribute.pc_feed_404.none-task-blog-2~default~BlogCommendFromBaidu~Rate-5.control404&depth_1-utm_source=distribute.pc_feed_404.none-task-blog-2~default~BlogCommendFromBaidu~Rate-5.control40)

原始代码中有 awk 和 sed 理解起来比较费劲：

【sed 讲解】[http://man.linuxde.net/sed](http://man.linuxde.net/sed)

【awk 讲解】[https://blog.csdn.net/wanxiaoderen/article/details/82253714](https://blog.csdn.net/wanxiaoderen/article/details/82253714)

【网上博客都有读取，没有写入】

而且，读取功能为

```
`awk -F '=' '/‘$Section’/{a=1}a==1&&$1~/'$Key'/{print $2;exit}' $Configfile 

```

有 BUG 当 读取【Item0】中的 【newf】  正确应该返回【空】 但是 按照这个写法会跳到后面去找，会返回【Item1】的【newf】 

![](https://img-blog.csdn.net/20180905102049637?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbnhpYW9kZXJlbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

修正后

```
awk -F '=' "/\[${section}\]/{a=1}a==1" ${iniFile}|sed -e '1d' -e '/^$/d' -e '/^\[.*\]/,$d' -e "/^${option}=.*/!d" -e "s/^${option}=//"
# awk 找出出 section 之后的内容
# sed 条件1：去除第一行 条件2：去除空行 条件3：去除其他section的内容 条件4：去除不匹配${key}=的行 条件5：将${key}=字符剔除
```

正文：~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

保存下来直接可以使用：

### **读取：**

```
source dealIni.sh iniFile section option 

```

参数  iniFile ：读取全 sections    保存在环境变量中为 shell 数组 iniSections

```
source dealIni.sh iniFile 

```

 ![](https://img-blog.csdn.net/20180904182427931?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbnhpYW9kZXJlbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

参数  iniFile section：读取全 section 下全 options (option=value)    保存在环境变量中为 shell 数组 iniOptions

```
source dealIni.sh iniFile section

```

![](https://img-blog.csdn.net/20180904182850734?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbnhpYW9kZXJlbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 参数  iniFile section option：读取全 section 中 option 的 value   保存在环境变量中为 shell 数组 iniValue

```
source dealIni.sh iniFile section option

```

![](https://img-blog.csdn.net/20180905100917950?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbnhpYW9kZXJlbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**写入：**
=======

参数  -w(标记为写入) iniFile section option value

```
source dealIni.sh -w iniFile section option value

```

![](https://img-blog.csdn.net/20180905101238628?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbnhpYW9kZXJlbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```
#该脚本必须用 source 命令 而且结果获取为${var}获取，不是return 如：source readIni.sh 否则变量无法外传
# dealIni.sh iniFile section option
# read
# param : iniFile section option    return value --- a str    use: ${iniValue}
# param : iniFile section           return options (element: option = value) --- a str[]   use: arr_length=${#iniOptions[*]}}  element=${iniOptions[0]}
# param : iniFile                   returm sections (element: section ) --- a str[]   use: arr_length=${#iniSections[*]}}  element=${iniSections[0]}
# write
#param : -w iniFile section option value  add new element：section option    result:if not --->creat ,have--->update,exist--->do nothing
#option ,value can not be null
 
#params
iniFile=$1
section=$2
option=$3
#sun
mode="iniR"
echo $@ | grep "\-w" >/dev/null&&mode="iniW"
if [ "$#" = "5" ]&&[ "$mode" = "iniW" ];then
   iniFile=$2
   section=$3
   option=$4
   value=$5
   #echo $iniFile $section $option $value
fi
#resullt
iniValue='default'
iniOptions=()
iniSections=()
 
function checkFile()
{
    if [ "${iniFile}" = ""  ] || [ ! -f ${iniFile} ];then
        echo "[error]:file --${iniFile}-- not exist!"
    fi
}
 
function readInIfile()
{
    if [ "${section}" = "" ];then
        #通过如下两条命令可以解析成一个数组
        allSections=$(awk -F '[][]' '/\[.*]/{print $2}' ${iniFile})
        iniSections=(${allSections// /})
        echo "[info]:iniSections size:-${#iniSections[@]}- eles:-${iniSections[@]}- "
    elif [ "${section}" != "" ] && [ "${option}" = "" ];then
        #判断section是否存在
        allSections=$(awk -F '[][]' '/\[.*]/{print $2}' ${iniFile})
        echo $allSections|grep ${section}
        if [ "$?" = "1" ];then
            echo "[error]:section --${section}-- not exist!"
            return 0
        fi
        #正式获取options
        #a=(获取匹配到的section之后部分|去除第一行|去除空行|去除每一行行首行尾空格|将行内空格变为@G@(后面分割时为数组时，空格会导致误拆))
        a=$(awk "/\[${section}\]/{a=1}a==1"  ${iniFile}|sed -e'1d' -e '/^$/d'  -e 's/[ \t]*$//g' -e 's/^[ \t]*//g' -e 's/[ ]/@G@/g' -e '/\[/,$d' )
        b=(${a})
        for i in ${b[@]};do
          #剔除非法字符，转换@G@为空格并添加到数组尾
          if [ -n "${i}" ]||[ "${i}" i!= "@G@" ];then
              iniOptions[${#iniOptions[@]}]=${i//@G@/ }
          fi
        done
        echo "[info]:iniOptions size:-${#iniOptions[@]}- eles:-${iniOptions[@]}-"
    elif [ "${section}" != "" ] && [ "${option}" != "" ];then
 
       # iniValue=`awk -F '=' '/\['${section}'\]/{a=1}a==1&&$1~/'${option}'/{print $2;exit}' $iniFile|sed -e 's/^[ \t]*//g' -e 's/[ \t]*$//g'`
        iniValue=`awk -F '=' "/\[${section}\]/{a=1}a==1" ${iniFile}|sed -e '1d' -e '/^$/d' -e '/^\[.*\]/,$d' -e "/^${option}.*=.*/!d" -e "s/^${option}.*= *//"`
        echo "[info]:iniValue value:-${iniValue}-"
        fi
}
 
function writeInifile()
{
    #检查文件
    checkFile
    allSections=$(awk -F '[][]' '/\[.*]/{print $2}' ${iniFile})
    iniSections=(${allSections// /})
    #判断是否要新建section
    sectionFlag="0"
    for temp in ${iniSections[@]};do
        if [ "${temp}" = "${section}" ];then
            sectionFlag="1"
            break
        fi
    done
 
    if [ "$sectionFlag" = "0" ];then
        echo "[${section}]" >>${iniFile}
    fi
    #加入或更新value
    awk "/\[${section}\]/{a=1}a==1" ${iniFile}|sed -e '1d' -e '/^$/d'  -e 's/[ \t]*$//g' -e 's/^[ \t]*//g' -e '/\[/,$d'|grep "${option}.\?=">/dev/null
    if [ "$?" = "0" ];then
        #更新
        #找到制定section行号码
        sectionNum=$(sed -n -e "/\[${section}\]/=" ${iniFile})
        sed -i "${sectionNum},/^\[.*\]/s/\(${option}.\?=\).*/\1 ${value}/g" ${iniFile}
        echo "[success] update [$iniFile][$section][$option][$value]"
    else
        #新增
        #echo sed -i "/^\[${section}\]/a\\${option}=${value}" ${iniFile}
        sed -i "/^\[${section}\]/a\\${option} = ${value}" ${iniFile}
        echo "[success] add [$iniFile][$section][$option][$value]"
    fi
}
 
#main
if [ "${mode}" = "iniR" ];then
    checkFile
    readInIfile
elif [ "${mode}" = "iniW" ];then
    writeInifile
fi
```