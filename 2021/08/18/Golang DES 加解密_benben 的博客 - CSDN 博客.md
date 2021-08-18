> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/benben_2015/article/details/81254023)

### 概念理解

DES 是以 64 比特的明文为一个单位来进行加密，并生成 64 比特的密文。由于它每次只能处理特定长度的一块数据，所以 DES 属于分组密码算法。`cypto/des`包提供了有关 des 加密的功能。

#### 模式

由于分组密码算法只能加密固定长度的分组，所以当加密的明文超过分组密码的长度时，就需要对分组密码算法进行迭代，而迭代的方法就称为分组密码的模式。模式主要有 ECB(电子密码本)、CBC(密码分组链接模式)、CTR(计数器模式)、OFB(输出反馈模式)、CFB(密码反馈模式) 五种。下面简单介绍下前两种：

1.  ECB(electronic code book) 是最简单的方式，它将明文分组加密后的结果直接成为密文分组。  
    优缺点：模式操作简单；明文中的重复内容将在密文中表现出来，特别对于图像数据和明文变化较少的数据；适于短报文的加密传递。  
    ![](https://img-blog.csdn.net/20180727152908938?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbmJlbl8yMDE1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
2.  CBC(cipher block chaining) 的原理是加密算法的输入是当前的明文分组和前一密文分组的异或，第一个明文分组和一个初始向量进行异或，这样同一个明文分组重复出现时会产生不同的密文分组。  
    特点：同一个明文分组重复出现时产生不同的密文分组；加密函数的输入是当前的明文分组和前一个密文分组的异或；每个明文分组的加密函数的输入与明文分组之间不再有固定的关系；适合加密长消息。  
    ![](https://img-blog.csdn.net/20180727152929756?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JlbmJlbl8yMDE1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 填充方式

在按 8 个字节对 DES 进行加密或解密时，如果最后一段字节不足 8 位，就需要对数据进行补位。即使加密或解密的数据刚好是 8 的倍数时，也会再补 8 位。举个栗子，如果末尾刚好出现 1，这时你就无法判断这个 1 是原来数据，还是经过补位得到的 1。因此，可以再补 8 位进行标识。填充方式主要有以下几种：pkcs7padding、pkcs5padding、zeropadding、iso10126、ansix923。

1.  pkcs7padding 和 pkcs5padding 的填充方式相同，填充字节的值都等于填充字节的个数。例如需要填充 4 个字节，则填充的值为 "4 4 4 4"。
2.  zeropadding 填充字节的值都为 0。

#### 密码

DES 的密钥长度是 64 比特，但由于每隔 7 个比特会设置一个用于错误检测的比特，因此其实质密钥长度为 56 比特。

#### 偏移量

上面模式中，例如 CBC，再加密第一个明文分组时，由于不存在 “前一个密文分组”，因此需要事先准备一个长度为一个分组的比特序列来代替 “前一个密文分组”，这个比特序列成为初始化向量，也称偏移量，通常缩写为 IV。一般来说，每次加密时都会随机产生一个不同的比特序列来作为初始化向量。偏移量的长度必须和块的大小相同。

#### 输出

加密后的字节在显示时可以进行 hex 和 base64 编码，hex 是十六进制编码，base64 是一种基于 64 个可打印字符来标识二进制数据的方法。

下面以上面提到的几种模式和填充方式为例，进行演示如何在代码中使用。

> 加密模式采用 ECB、填充方式采用 pkcs5padding、密码使用 "12345678", 输出时经 hex 编码。自己可以通过一些[在线测试工具](http://tool.chacuo.net/cryptdes)进行测试，看结果是否一致。

```
package main

import (
	"crypto/des"
	"qiniupkg.com/x/errors.v7"
	"bytes"
	"fmt"
	"encoding/hex"
)

func main() {
	data:=[]byte("hello world")
	key:=[]byte("12345678")
	result,err:=DesECBEncrypt(data,key)
	if err != nil {
		fmt.Println(err)
	}
	a:=hex.EncodeToString(result)
	fmt.Println(a)
}

func DesECBEncrypt(data, key []byte) ([]byte, error) {
    //NewCipher创建一个新的加密块
	block, err := des.NewCipher(key)
	if err != nil {
		return nil, err
	}

	bs := block.BlockSize()
	data = Pkcs5Padding(data, bs)
	if len(data)%bs != 0 {
		return nil, errors.New("need a multiple of the blocksize")
	}

	out := make([]byte, len(data))
	dst := out
	for len(data) > 0 {
        //Encrypt加密第一个块，将其结果保存到dst
		block.Encrypt(dst, data[:bs])
		data = data[bs:]
		dst = dst[bs:]
	}
	return out, nil
}

func Pkcs5Padding(ciphertext []byte, blockSize int) []byte {
	padding := blockSize - len(ciphertext)%blockSize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padtext...)
}
```

> 下面加密模式采用 CBC、填充方式采用 pkcs5padding、密码使用 "12345678"、偏移量 "43218765"，输出时以 hex 方式输出。自己可以通过一些[在线测试工具](http://tool.chacuo.net/cryptdes)进行测试，看结果是否一致。

```
package main

import (
	"crypto/des"
	"bytes"
	"fmt"
	"encoding/hex"
	"crypto/cipher"
)

func main() {
	data := []byte("hello world")
	key := []byte("12345678")
	iv := []byte("43218765")

	result, err := DesCBCEncrypt(data, key, iv)
	if err != nil {
		fmt.Println(err)
	}
	b := hex.EncodeToString(result)
	fmt.Println(b)
}

func DesCBCEncrypt(data, key, iv []byte) ([]byte, error) {
	block, err := des.NewCipher(key)
	if err != nil {
		return nil, err
	}

	data = pkcs5Padding(data, block.BlockSize())
	cryptText := make([]byte, len(data))

	blockMode := cipher.NewCBCEncrypter(block, iv)
	blockMode.CryptBlocks(cryptText, data)
	return cryptText, nil
}

func pkcs5Padding(cipherText []byte, blockSize int) []byte {
	padding := blockSize - len(cipherText)%blockSize
	padText := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(cipherText, padText...)
}
```

第三方包
----

`github.com/marspere/goencrypt`包实现了多种加密算法，包括对称加密和非对称加密等。

```
package main

import (
	"fmt"
	"github.com/marspere/goencrypt"
)

func main() {
	// key为12345678
	// iv为空
	// 采用ECB分组模式
	// 采用pkcs5padding填充模式
	// 输出结果使用base64进行加密
	cipher := goencrypt.NewDESCipher([]byte("12345678"), []byte(""), goencrypt.ECBMode, goencrypt.Pkcs5, goencrypt.PrintBase64)
	cipherText, err := cipher.DESEncrypt([]byte("hello world"))
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(cipherText)
}
```