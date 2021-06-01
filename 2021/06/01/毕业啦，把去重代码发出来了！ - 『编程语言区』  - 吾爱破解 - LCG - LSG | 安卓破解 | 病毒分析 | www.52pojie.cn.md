> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1451483&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline)  ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) 凹凸曼大人 这个去重，是[破解](https://www.52pojie.cn)别人 APP，并获取接口，来获得去重。亲测，很流畅，对维普查重无效（因为人家是查关键词），这个去重改顺序，改几个词挺流畅的。  
![](https://attach.52pojie.cn/forum/202106/01/170139pl8df6bfqbqzd6db.png)

**image.png** _(89.77 KB, 下载次数:)_

[下载附件](forum.php?mod=attachment&aid=MjI5ODYyMnxhZGMzMWE2MXwxNjIyNTU1MjA5fDEwNzQxNTd8MTQ1MTQ4Mw%3D%3D&nothumb=yes)

2021-6-1 17:01 上传

  
我用的是 REMI 库，我用网页来显示，比较流畅，也可以客户端显示。  
主代码：  
[Python] _纯文本查看_ _复制代码_

```
# -*- coding: utf-8 -*-
# 论文去重
 
 
#@athor：凹凸曼大人
#@time：2021/5/8
 
import requests
import jiami
import qingqiu
import jpype
from remi.gui import Container,Button,TextInput,Label,InputDialog
import remi.gui as gui
import time
from remi import start, App
class untitled(App):
    def __init__(self, *args):
        super(untitled, self).__init__(*args)
 
    def idle(self):
        # idle function called every update cycle
        pass
 
    def main(self):
        return untitled.construct_ui(self)
 
 
    def construct_ui(self):
        # DON'T MAKE CHANGES HERE, THIS METHOD GETS OVERWRITTEN WHEN SAVING IN THE EDITOR
        container0 = Container()
        container0.attr_editor_newclass = False
        container0.css_height = "100%"
        container0.css_left = "0px"
        container0.css_margin = "px"
        container0.css_position = "absolute"
        container0.css_top = "0px"
        container0.css_width = "100%"
        container0.variable_name = "container0"
        textinput0 = TextInput()
        textinput0.attr_editor_newclass = False
        textinput0.css_height = "315.0px"
        textinput0.css_left = "30.0px"
        textinput0.css_margin = "0px"
        textinput0.css_position = "absolute"
        textinput0.css_top = "45.0px"
        textinput0.css_width = "210.0px"
        textinput0.text = ""
        textinput0.variable_name = "textinput0"
        self.textinput0=textinput0
        container0.append(textinput0, 'textinput0')
        label0 = Label()
        label0.attr_editor_newclass = False
        label0.css_background_color = "rgb(255,255,255)"
        label0.css_color = "rgb(150,0,0)"
        label0.css_font_size = "20px"
        label0.css_height = "30px"
        label0.css_left = "105.0px"
        label0.css_margin = "0px"
        label0.css_position = "absolute"
        label0.css_top = "15.0px"
        label0.css_width = "100px"
        label0.text = "原文"
        label0.variable_name = "label0"
        container0.append(label0, 'label0')
        textinput1 = TextInput()
        textinput1.attr_editor_newclass = False
        textinput1.css_height = "135.0px"
        textinput1.css_left = "330.0px"
        textinput1.css_margin = "0px"
        textinput1.css_position = "absolute"
        textinput1.css_top = "30.0px"
        textinput1.css_width = "450.0px"
        textinput1.text = ""
        textinput1.variable_name = "textinput1"
        self.textinput1 = textinput1
        container0.append(textinput1, 'textinput1')
        textinput2 = TextInput()
        textinput2.attr_editor_newclass = False
        textinput2.css_height = "150.0px"
        textinput2.css_left = "330.0px"
        textinput2.css_margin = "0px"
        textinput2.css_position = "absolute"
        textinput2.css_top = "210.0px"
        textinput2.css_width = "450.0px"
        textinput2.text = ""
        textinput2.variable_name = "textinput2"
        self.textinput2 = textinput2
        container0.append(textinput2, 'textinput2')
        label1 = Label()
        label1.attr_editor_newclass = False
        label1.css_border_color = "rgb(0,114,0)"
        label1.css_color = "rgb(0,134,0)"
        label1.css_height = "30.0px"
        label1.css_left = "255.0px"
        label1.css_margin = "0px"
        label1.css_position = "absolute"
        label1.css_top = "75.0px"
        label1.css_width = "60.0px"
        label1.text = "简单去重"
        label1.variable_name = "label1"
        container0.append(label1, 'label1')
        label2 = Label()
        label2.attr_editor_newclass = False
        label2.css_border_color = "rgb(0,123,0)"
        label2.css_color = "rgb(0,125,0)"
        label2.css_height = "45.0px"
        label2.css_left = "255.0px"
        label2.css_margin = "0px"
        label2.css_position = "absolute"
        label2.css_top = "285.0px"
        label2.css_width = "60.0px"
        label2.text = "深度去重"
        label2.variable_name = "label2"
        container0.append(label2, 'label2')
        button0 = Button()
        button0.attr_editor_newclass = False
        button0.css_background_color = "rgb(132,175,210)"
        button0.css_height = "30.0px"
        button0.css_left = "255.0px"
        button0.css_margin = "0px"
        button0.css_position = "absolute"
        button0.css_top = "180.0px"
        button0.css_width = "75.0px"
        button0.text = "去重"
        button0.variable_name = "button0"
        container0.append(button0, 'button0')
        container0.children['button0'].onclick.do(self.onclick_button0)
 
        self.container0 = container0
        return self.container0
 
    def onclick_button0(self, emitter):
        jiandan_jieguo,nan_jieguo=jiami.suanfa(self.textinput0.text)
        jiandan=qingqiu.huoqu(jiandan_jieguo)
        nan=qingqiu.huoqu(nan_jieguo)
        self.textinput1.text=jiandan
        self.textinput2.text=nan
 
 
 
 
 
 
# processing_code
 
 
 
 
if __name__ == "__main__":
    # starts the webserver
    # optional parameters
    start(untitled,address='127.5.5.1', port=8581, multiple_instance=False,enable_file_cache=True, update_interval=0.1, start_browser=True)
    #start(untitled)
    #start(untitled,standalone=True)
```

加密代码，因为人家是 APP 加密的，加密部分我用 JAVA 实现，然后转成包，用 python 调用，java 代码  
[Java] _纯文本查看_ _复制代码_

```
package 注册码;
 
 
import java.io.UnsupportedEncodingException;
import java.nio.charset.StandardCharsets;
  
public class RC4Util {
  
    /**
     * RC4加密，将加密后的数据进行哈希
     * [url=home.php?mod=space&uid=952169]@Param[/url] data 需要加密的数据
     * @param key 加密密钥
     * @param chartSet 编码方式
     * [url=home.php?mod=space&uid=155549]@Return[/url] 返回加密后的数据
     * @throws UnsupportedEncodingException
     */
    public static String encryRC4String(String data, String key, String chartSet) throws UnsupportedEncodingException {
        if (data == null || key == null) {
            return null;
        }
        return bytesToHex(encryRC4Byte(data, key, chartSet));
    }
  
    /**
     * RC4加密，将加密后的字节数据
     * @param data 需要加密的数据
     * @param key 加密密钥
     * @param chartSet 编码方式
     * @return 返回加密后的数据
     * @throws UnsupportedEncodingException
     */
    public static byte[] encryRC4Byte(String data, String key, String chartSet) throws UnsupportedEncodingException {
        if (data == null || key == null) {
            return null;
        }
        if (chartSet == null || chartSet.isEmpty()) {
            byte bData[] = data.getBytes();
            return RC4Base(bData, key);
        } else {
            byte bData[] = data.getBytes(chartSet);
            return RC4Base(bData, key);
        }
    }
  
    /**
     * RC4解密
     * @param data 需要解密的数据
     * @param key 加密密钥
     * @param chartSet 编码方式
     * @return 返回解密后的数据
     * @throws UnsupportedEncodingException
     */
    public static String decryRC4(String data, String key,String chartSet) throws UnsupportedEncodingException {
        if (data == null || key == null) {
            return null;
        }
        return new String(RC4Base(hexToByte(data), key),chartSet);
    }
  
    /**
     * RC4加密初始化密钥
     * @param aKey
     * @return
     */
    private static byte[] initKey(String aKey) {
        byte[] bkey = aKey.getBytes();
        byte state[] = new byte[256];
  
        for (int i = 0; i < 256; i++) {
            state[i] = (byte) i;
        }
        int index1 = 0;
        int index2 = 0;
        if (bkey.length == 0) {
            return null;
        }
        for (int i = 0; i < 256; i++) {
            index2 = ((bkey[index1] & 0xff) + (state[i] & 0xff) + index2) & 0xff;
            byte tmp = state[i];
            state[i] = state[index2];
            state[index2] = tmp;
            index1 = (index1 + 1) % bkey.length;
        }
        return state;
    }
  
  
    /**
     * 字节数组转十六进制
     * @param bytes
     * @return
     */
    public static String bytesToHex(byte[] bytes) {
        StringBuffer sb = new StringBuffer();
        for(int i = 0; i < bytes.length; i++) {
            String hex = Integer.toHexString(bytes[i] & 0xFF);
            if(hex.length() < 2){
                sb.append(0);
            }
            sb.append(hex);
        }
        return sb.toString();
    }
  
    /**
     * 十六进制转字节数组
     * @param src
     * @return
     */
    public static byte[] hexToByte(String inHex){
        int hexlen = inHex.length();
        byte[] result;
        if (hexlen % 2 == 1){
            hexlen++;
            result = new byte[(hexlen/2)];
            inHex="0"+inHex;
        }else {
            result = new byte[(hexlen/2)];
        }
        int j=0;
        for (int i = 0; i < hexlen; i+=2){
            result[j]=(byte)Integer.parseInt(inHex.substring(i,i+2),16);
            j++;
        }
        return result;
    }
  
    /**
     * RC4解密
     * @param input
     * @param mKkey
     * @return
     */
    private static byte[] RC4Base(byte[] input, String mKkey) {
        int x = 0;
        int y = 0;
        byte key[] = initKey(mKkey);
        int xorIndex;
        byte[] result = new byte[input.length];
        for (int i = 0; i < input.length; i++) {
            x = (x + 1) & 0xff;
            y = ((key[x] & 0xff) + y) & 0xff;
            byte tmp = key[x];
            key[x] = key[y];
            key[y] = tmp;
            xorIndex = ((key[x] & 0xff) + (key[y] & 0xff)) & 0xff;
            result[i] = (byte) (input[i] ^ key[xorIndex]);
        }
        return result;
         
    }
    public static String sayHello(String user) throws UnsupportedEncodingException{ //注意！作为被 python调用的接口函数，需要是静态的，否则 python 汇报错
        String encryStr = encryRC4String("{\"type\":\"simple\",\"query\":\""+user+"\"}", "zxcoiua839","GBK");
        return encryStr;
          }
    public static String sayno(String user) throws UnsupportedEncodingException{ //注意！作为被 python调用的接口函数，需要是静态的，否则 python 汇报错
        String encryStr = encryRC4String("{\"type\":\"mid\",\"query\":\""+user+"\"}", "zxcoiua839","GBK");
        return encryStr;
          }
    public static void main(String[] args) throws UnsupportedEncodingException {  
        //String encryStr = encryRC4String("{\"type\":\"simple\",\"query\":\"1\"}", "zxcoiua839","GBK");
        //System.out.println("加密后得到得字符串："+encryStr);
        //String decryStr = decryRC4(encryStr, "zxcoiua839","GBK");
        //System.out.println("解密后得到得字符串："+decryStr);  
         
    }
 
}
```

调用 java 包加密代码  
[Python] _纯文本查看_ _复制代码_

```
#!/usr/bin/python
# -*- coding: utf-8 -*-
 
# 引入jpype模块
import jpype
import os
 
"""
基本的开发流程如下：
①、使用jpype开启jvm
②、加载java类
③、调用java方法
④、关闭jvm（不是真正意义上的关闭，卸载之前加载的类）
"""
# ①、使用jpype开启虚拟机（在开启jvm之前要加载类路径）
 
# 加载刚才打包的jar文件
def suanfa(zhi):
    if not jpype.isJVMStarted():
        jarpath = os.path.join(os.path.abspath("."), "jiami.jar")
 
        # 获取jvm.dll 的文件路径
        jvmPath = jpype.getDefaultJVMPath()
 
        # 开启jvm
        jpype.startJVM(jvmPath, "-ea", "-Djava.class.path=%s" % (jarpath))
        # ②、加载java类（参数是java的长类名）
        javaClass = jpype.JClass("注册码.RC4Util")
        # 实例化java对象
        # javaInstance = javaClass()
 
        # ③、调用java方法，由于我写的是静态方法，直接使用类名就可以调用方法
        jieguo = javaClass.sayHello(zhi)
        jieguo1 = javaClass.sayno(zhi)
        # print(javaClass.sayHello(zhi))
 
        return jieguo, jieguo1
        # ④、关闭jvm
        jpype.shutdownJVM()
    else:
        # ②、加载java类（参数是java的长类名）
        javaClass = jpype.JClass("注册码.RC4Util")
        # 实例化java对象
        # javaInstance = javaClass()
 
        # ③、调用java方法，由于我写的是静态方法，直接使用类名就可以调用方法
        jieguo = javaClass.sayHello(zhi)
        jieguo1 = javaClass.sayno(zhi)
        # print(javaClass.sayHello(zhi))
 
        return jieguo, jieguo1
        # ④、关闭jvm
        jpype.shutdownJVM()
 
 
if __name__=='__main__':
 
    print(suanfa_jiandan("1"))
```

最后的代码，是获取去重结果的代码  
[Python] _纯文本查看_ _复制代码_

```
import requests
def huoqu(zhi):
 
    headers = {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Host': '180.76.119.117:13888',
        'Connection': 'Keep-Alive',
        'User-Agent': 'Apache-HttpClient/UNAVAILABLE (java 1.4)',
    }
 
    data = str(zhi)
 
    response = requests.post('http://180.76.119.117:13888/quchong', headers=headers, data=data, verify=False)
    return response.text
```

到此就结束了。![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)tony666  不用这么麻烦，百度翻译 yyds![](https://static.52pojie.cn/static/image/smiley/laohu/laohu23.gif)  
先中文翻译英文，再翻译回来就行了![](https://avatar.52pojie.cn/data/avatar/000/96/24/79_avatar_middle.jpg)王星星  谢谢分享![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)虫子虫子丶  不会用代码![](https://static.52pojie.cn/static/image/smiley/default/cry.gif) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) First 丶云心 谢谢分享，学习学习!! ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) 风 / 生 / 水 / 起 已学习，谢谢分享 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) xiao_1819 谢谢分享 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) Roc. 代码小白，特想知道如何用 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) Kay0905 谢谢分享![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)我心她有丶  说实话，可真吊啊！![](https://static.52pojie.cn/static/image/smiley/laohu/laohu25.gif)![](https://static.52pojie.cn/static/image/smiley/laohu/laohu25.gif)![](https://static.52pojie.cn/static/image/smiley/laohu/laohu25.gif)![](https://static.52pojie.cn/static/image/smiley/laohu/laohu25.gif)![](https://static.52pojie.cn/static/image/smiley/laohu/laohu25.gif)![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)jlkk8  代码咋个用的  老大们![](https://static.52pojie.cn/static/image/smiley/default/9.gif)![](https://static.52pojie.cn/static/image/smiley/default/9.gif)