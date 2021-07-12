> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/rick168/p/4652180.html?ivk_sa=1024320u)

RC4 相对是速度快、安全性高的加密算法。在实际应用中，我们可以对安全系数要求高的文本进行多重加密，这样破解就有一定困难了。如下测试给出了先用 RC4 加密，然后再次用 BASE64 编码，这样双重锁定，保证数据信息安全 (个人见解，不周之处请谅解！)。

package com.bao.tools.encryption;  
import java.io.IOException;  
import java.util.Scanner;  
import org.junit.Test;  
/**  
 * @title RC4 加密解密算法  
 * @author Administrator  
 * @date 2015-7-16  
 */  
public class CRC4 {  
    /*  
     * 加密解密算法  
     */  
    public static String setEncrypted(String value, String key) {  
        // 声明 256 位数组  
        int[] iS = new int[256];  
        byte[] iK = new byte[256];  
        // 赋值  
        for (int i = 0; i < 256; i++)  
            iS[i] = i;  
        int j = 1;  
        // 检索位置  
        for (short i = 0; i < 256; i++) {  
            iK[i] = (byte) key.charAt((i % key.length()));  
        }  
        j = 0;  
          
        // 替换位置  
        for (int i = 0; i < 255; i++) {  
            j = (j + iS[i] + iK[i]) % 256;  
            int temp = iS[i];  
            iS[i] = iS[j];  
            iS[j] = temp;  
        }  
        int i = 0;  
        j = 0;  
        char[] iInputChar = value.toCharArray();// 字符转换  
        char[] iOutputChar = new char[iInputChar.length];// 获得字符长度  
        for (short x = 0; x < iInputChar.length; x++) {  
            i = (i + 1) % 256;  
            j = (j + iS[i]) % 256;  
            int temp = iS[i];  
            iS[i] = iS[j];  
            iS[j] = temp;  
            int t = (iS[i] + (iS[j] % 256)) % 256;  
            int iY = iS[t];  
            char iCY = (char) iY;  
            iOutputChar[x] = (char) (iInputChar[x] ^ iCY);// 做字符异或运算  
        }  
        return new String(iOutputChar);  
    }  
    /*  
     * 测试  
     */  
    @Test  
    public void test() throws IOException {  
        Scanner scanner = new Scanner(System.in);  
          
        System.out.println("------------ 使用 RC4 和 BASE64 多重加密算法 ---------");  
        System.out.println("请输入 RC4 使用的加密值 (value)：");  
        String inputStr = scanner.nextLine();  
        System.out.println("请输入 RC4 加密键 (key)：");  
        String key = scanner.nextLine();  
        String str = setEncrypted(inputStr, key);// 使用 RC4 加密  
        str = CBase64.setEncrypted(str);// 使用 base64 编码 (CBase64 加密类请看 <[Java 中使用 BASE64 加密 & 解密](http://www.cnblogs.com/Xiawu168/p/4650674.html) >)  
        // 打印加密后的字符串  
        System.out.println("\n---------------------\n 加密后字符："+ str);  
        str = CBase64.getEncrypted(str);// 首先使用 CBase64 解码  
        // 打印解密后的字符串  
        System.out.println("解密后字符 ："+ setEncrypted(str, key));

　　　scanner.close();  
    }  
}  
运行结果如下

![](https://images0.cnblogs.com/blog2015/523241/201507/161846426264004.png)