> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666556780&idx=3&sn=bd32e00daa832e93afeaff6fddbf1c85&chksm=80dca9c7b7ab20d110bebf3fe0689d56ed7b56fb6c9796d27591ee3c91451ccb768475904989&mpshare=1&scene=1&srcid=0818cfeiOTxzRPGCTwXm6jBd&sharer_sharetime=1629291152180&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

【导读】什么是对称加密？Go 语言做对称加密怎么做？本文作者从加密原理到代码实现带你上车。  

对称加密中，加密和解密使用相同的密钥，因此必须向解密者配送密钥，即密钥配送问题。而非对称加密中，由于加密和解密分别使用公钥和私钥，而公钥是公开的，因此可以规避密钥配送问题。非对称加密算法，也称公钥加密算法。

1977 年，Ron Rivest、Adi Shamir、Leonard Adleman 三人在美国公布了一种公钥加密算法，即 RSA 公钥加密算法。RSA 是目前最有影响力和最常用的公钥加密算法，可以说是公钥加密算法的事实标准。

一、RSA 加密原理
----------

使用 M 和 C 分别表示明文和密文，则 RSA 加密、解密过程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/IgylNib7ZE2JcXib5267hOJz3oyIUoMibpcgKvkcicM1Budv0AbSDRe7hWXBs6sj0aN1picHkYK5yWDurEhxCofy09w/640?wx_fmt=png)

img

其中 e、n 的组合 (e, n) 即为公钥，d、n 的组合 (d, n) 即为私钥。当然 e、d、n 并非任意取值，需要符合一定条件，如下即为 e、d、n 的求解过程。

### 生成密钥对

e、d、n 的求解过程，也即生成密钥对的过程。涉及如下步骤：　　1、取两个大质数（也称素数）p、q，n = pq。　　2、取正整数 e、d，使得 ed mod (p-1)(q-1) = 1，也即：ed ≡ 1 mod (p-1)(q-1)。　　e 和 d 是模 (p-1)(q-1) 的乘法逆元，仅当 e 与 (p-1)(q-1) 互质时，存在 d。　　举例验证：　　1、取 p、q 分别为 13、17，n = pq = 221。　　2、而 (p-1)(q-1) = 12x16 = 192，取 e、d 分别为 13、133，有 13x133 mod 192 = 1　　取明文 M = 60，公钥加密、私钥解密，加密和解密过程分别如下：

![](https://mmbiz.qpic.cn/mmbiz_png/IgylNib7ZE2JcXib5267hOJz3oyIUoMibpcypZ8g8Vt4n27QdgT44TV380W8PliaH40EPuQIricHu1HIY8dzFktVLyA/640?wx_fmt=png)

img

### RSA 加密原理证明过程

![](https://mmbiz.qpic.cn/mmbiz_png/IgylNib7ZE2JcXib5267hOJz3oyIUoMibpcaOsUSmLdva5xkibMR4vKJ6OzkPysATsXXeAvsGABP7JAh65ibl2icOasw/640?wx_fmt=png)

img

### 手动求解密钥对中的 d

ed mod (p-1)(q-1) = 1，已知 e 和 (p-1)(q-1) 求 d，即求 e 对模 (p-1)(q-1) 的乘法逆元。　　如上面例子中，p、q 为 13、17，(p-1)(q-1)=192，取 e=13，求 13d mod 192 = 1 中的 d。　　13d ≡ 1 (mod 192)，在右侧添加 192 的倍数，使计算结果可以被 13 整除。　　13d ≡ 1 + 192x9 ≡ 13x133 (mod 192)，因此 d = 133　　其他计算方法有：费马小定律、扩展欧几里得算法、欧拉定理。

### RSA 安全性

由于公钥公开，即 e、n 公开。　　因此破解 RSA 私钥，即为已知 e、n 情况下求 d。　　因 ed mod (p-1)(q-1) = 1，且 n=pq，因此该问题演变为：对 n 质因数分解求 p、q。　　目前已被证明，已知 e、n 求 d 和对 n 质因数分解求 p、q 两者是等价的。实际中 n 长度为 2048 位以上，而当 n>200 位时分解 n 是非常困难的，因此 RSA 算法目前仍被认为是安全实用的。

### RSA 计时 *** 和防范

RSA 解密的本质是模幂运算，即：

![](https://mmbiz.qpic.cn/mmbiz_png/IgylNib7ZE2JcXib5267hOJz3oyIUoMibpcbsVBYoicTrnOwSFNL3s7Xib2e2ec6z1rSYmEeyFfrtoxnCsicoPZzVNBg/640?wx_fmt=png)

img

其中 C 为密文，(d,n) 为私钥，均为超过 1024 位的大数运算，直接计算并不可行，因此最经典的算法为蒙哥马利算法。而这种计算是比较是耗时的，因此者可以观察不同的输入对应的解密时间，通过分析推断私钥，称为计时。而防范 RSA 计时的办法，即在解密时加入随机因素，使得 *** 者无法准确获取解密时间。　　具体实现步骤如下：

![](https://mmbiz.qpic.cn/mmbiz_png/IgylNib7ZE2JcXib5267hOJz3oyIUoMibpcic0AwaXiaY4ZQepyDOR7ttZb4wG0KJaR6cmGKic0MibfTOiaacD0iaWUIPiaQ/640?wx_fmt=png)

img

二、Go RSA 加密解密
-------------

### 1、rsa 加解密，必然会去查 crypto/ras 这个包

> Package rsa implements RSA encryption as specified in PKCS#1.

这是该包的说明：实现 RSA 加密技术，基于 PKCS#1 规范。

对于什么是 PKCS#1，可以查阅相关资料。PKCS（公钥密码标准），而 #1 就是 RSA 的标准。可以查看：PKCS 系列简介

从该包中函数的名称，可以看到有两对加解密的函数。

> EncryptOAEP 和 DecryptOAEPEncryptPKCS1v15 和 DecryptPKCS1v15

这称作加密方案，详细可以查看，PKCS #1 v2.1 RSA 算法标准

可见，当与其他语言交互时，需要确定好使用哪种方案。

PublicKey 和 PrivateKey 两个类型分别代表公钥和私钥，关于这两个类型中成员该怎么设置，这涉及到 RSA 加密算法，本文中，这两个类型的实例通过解析文章开头生成的密钥得到。

### 2、解析密钥得到 PublicKey 和 PrivateKey 的实例

这个过程，我也是花了好些时间（主要对各种加密的各种东东不熟）：怎么将 openssl 生成的密钥文件解析到公钥和私钥实例呢？

在 encoding/pem 包中，看到了—–BEGIN Type—–这样的字样，这正好和 openssl 生成的密钥形式差不多，那就试试。

在该包中，一个 block 代表的是 PEM 编码的结构，关于 PEM，请查阅相关资料。我们要解析密钥，当然用 Decode 方法：

> func Decode(data []byte) (p *Block, rest []byte)

这样便得到了一个 Block 的实例（指针）。

解析来看 crypto/x509。为什么是 x509 呢？这又涉及到一堆概念。先不管这些，我也是看 encoding 和 crypto 这两个包的子包摸索出来的。在 x509 包中，有一个函数：

> func ParsePKIXPublicKey(derBytes []byte) (pub interface{}, err error)

从该函数的说明：_ParsePKIXPublicKey parses a DER encoded public key. These values are typically found in PEM blocks with “BEGIN PUBLIC KEY”_。可见这就是解析 PublicKey 的。另外，这里说到了 PEM，可以上面的 encoding/pem 对了。（PKIX 是啥东东，查看这里 ）

而解析私钥的，有好几个方法，从上面的介绍，我们知道，RSA 是 PKCS#1，刚好有一个方法：

> func ParsePKCS1PrivateKey(der []byte) (key *rsa.PrivateKey, err error)

返回的就是 rsa.PrivateKey。

代码实现：
-----

```
package main
 
import (
 "crypto/rsa"
 "crypto/rand"
 "crypto/x509"
 "encoding/pem"
 "os"
 "fmt"
)
 
func RSAGenKey(bits int) error {
 /*
  生成私钥
 */
 //1、使用 RSA 中的 GenerateKey 方法生成私钥
 privateKey, err := rsa.GenerateKey(rand.Reader, bits)
 if err != nil {
  return err
 }
 //2、通过 X509 标准将得到的 RAS 私钥序列化为：ASN.1 的 DER 编码字符串
 privateStream := x509.MarshalPKCS1PrivateKey(privateKey)
 //3、将私钥字符串设置到 pem 格式块中
 block1 := pem.Block{
  Type:  "private key",
  Bytes: privateStream,
 }
 //4、通过 pem 将设置的数据进行编码，并写入磁盘文件
 fPrivate, err := os.Create("privateKey.pem")
 if err != nil {
  return err
 }
 defer fPrivate.Close()
 err = pem.Encode(fPrivate, &block1)
 if err != nil {
  return err
 }
 
 /*
 生成公钥
 */
 publicKey:=privateKey.PublicKey
 publicStream,err:=x509.MarshalPKIXPublicKey(&publicKey)
 //publicStream:=x509.MarshalPKCS1PublicKey(&publicKey)
 block2:=pem.Block{
  Type:"public key",
  Bytes:publicStream,
 }
 fPublic,err:=os.Create("publicKey.pem")
 if err!=nil {
  return  err
 }
 defer fPublic.Close()
 pem.Encode(fPublic,&block2)
 return nil
}
//对数据进行加密操作
func  EncyptogRSA(src []byte,path string) (res []byte,err error) {
 //1. 获取秘钥（从本地磁盘读取）
 f,err:=os.Open(path)
 if err!=nil {
  return
 }
 defer f.Close()
 fileInfo,_:=f.Stat()
 b:=make([]byte,fileInfo.Size())
 f.Read(b)
 // 2、将得到的字符串解码
 block,_:=pem.Decode(b)
 
 // 使用 X509 将解码之后的数据 解析出来
 //x509.MarshalPKCS1PublicKey(block): 解析之后无法用，所以采用以下方法：ParsePKIXPublicKey
 keyInit,err:=x509.ParsePKIXPublicKey(block.Bytes)  //对应于生成秘钥的 x509.MarshalPKIXPublicKey(&publicKey)
 //keyInit1,err:=x509.ParsePKCS1PublicKey(block.Bytes)
 if err!=nil {
  return
 }
 //4. 使用公钥加密数据
 pubKey:=keyInit.(*rsa.PublicKey)
 res,err=rsa.EncryptPKCS1v15(rand.Reader,pubKey,src)
 return
}
//对数据进行解密操作
func DecrptogRSA(src []byte,path string)(res []byte,err error)  {
 //1. 获取秘钥（从本地磁盘读取）
 f,err:=os.Open(path)
 if err!=nil {
  return
 }
 defer f.Close()
 fileInfo,_:=f.Stat()
 b:=make([]byte,fileInfo.Size())
 f.Read(b)
 block,_:=pem.Decode(b)//解码
 privateKey,err:=x509.ParsePKCS1PrivateKey(block.Bytes)//还原数据
 res,err=rsa.DecryptPKCS1v15(rand.Reader,privateKey,src)
 return
}
func main() {
 //rsa.GenerateKey()
 err:=RSAGenKey(4096)
 if err!=nil {
  fmt.Println(err)
  return
 }
 fmt.Println("秘钥生成成功！")
 str:="山重水复疑无路，柳暗花明又一村！"
 fmt.Println("加密之前的数据为：",string(str))
 data,err:=EncyptogRSA([]byte(str),"publicKey.pem")
 data,err=DecrptogRSA(data,"privateKey.pem")
 fmt.Println("加密之后的数据为：",string(data))
}
```

> 转自：Wuman
> 
> wumansgy.github.io/2018/10/18/GO%E8%AF%AD%E8%A8%80RSA%E5%8A%A0%E5%AF%86%E8%A7%A3%E5%AF%86

- EOF -