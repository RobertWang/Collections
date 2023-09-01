> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7273025938940624948) 点击查看代码:

```
package main

import (
	"bytes"
	"encoding/binary"
	"encoding/json"
	"fmt"
	"log"
	"math/rand"
	"net/http"
	"time"
)

func main() {

	http.HandleFunc("/register", deal) //设置访问的路由

	fmt.Println("1111:", 2222)

	err := http.ListenAndServe(":8088", nil) //设置监听的端口
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}

}

func deal(w http.ResponseWriter, r *http.Request) {

	// Golang: 接收GET和POST参数 https://www.cnblogs.com/liuhe688/p/11063945.html
	// 根据请求body创建一个json解析器实例
	decoder := json.NewDecoder(r.Body)
	// 用于存放参数key=value数据
	var params map[string]string

	// 解析参数 存入map
	decoder.Decode(¶ms)

	name := params["name"]

	rand.Seed(time.Now().UnixNano())

	key := crack(name)

	fmt.Println("name:", name, "    key:", crack(name))

	fmt.Fprintf(w, key) //这个写入到w的是输出到客户端的

}

const (
	rounds    = 12
	roundKeys = 2 * (rounds + 1)
)

func crack(text string) string {

	name := []byte(text)
	length := len(name) + 4
	padded := ((-length) & (8 - 1)) + length
	bs := make([]byte, 4)
	binary.BigEndian.PutUint32(bs, uint32(len(name)))
	buff := bytes.Buffer{}
	buff.Write(bs)
	buff.Write(name)

	var ckName int64 = 0x7a21c951691cd470
	var ckKey int64 = -5408575981733630035
	ck := newCkCipher(ckName)
	outBuff := bytes.Buffer{}

	for i := 0; i < padded; i += 8 {
		bf := buff.Bytes()[i : i+8]
		buf := bytes.NewBuffer(bf)
		var nowVar int64
		if err := binary.Read(buf, binary.BigEndian, &nowVar); err != nil {
			panic(err)
		}

		dd := ck.encrypt(nowVar)

		outBuff.WriteByte(byte(dd >> 56))
		outBuff.WriteByte(byte(dd >> 48))
		outBuff.WriteByte(byte(dd >> 40))
		outBuff.WriteByte(byte(dd >> 32))
		outBuff.WriteByte(byte(dd >> 24))
		outBuff.WriteByte(byte(dd >> 16))
		outBuff.WriteByte(byte(dd >> 8))
		outBuff.WriteByte(byte(dd))

	}
	var n int32
	for _, b := range outBuff.Bytes() {
		n = rotateLeft(n^int32(int8(b)), 0x3)
	}
	prefix := n ^ 0x54882f8a
	suffix := rand.Int31()
	in := int64(prefix) << 32
	s := int64(suffix)
	switch suffix >> 16 {
	case 0x0401:
	case 0x0402:
	case 0x0403:
		in |= s
		break
	default:
		in |= 0x01000000 | (s & 0xffffff)
		break
	}

	out := newCkCipher(ckKey).decrypt(in)

	var n2 int64
	for i := 56; i >= 0; i -= 8 {
		n2 ^= int64((uint64(in) >> i) & 0xff)
	}

	vv := int32(n2 & 0xff)
	if vv < 0 {
		vv = -vv
	}
	return fmt.Sprintf("%02x%016x", vv, uint64(out))
}

type ckCipher struct {
	rk [roundKeys]int32
}

func newCkCipher(ckKey int64) ckCipher {
	ck := ckCipher{}

	var ld [2]int32
	ld[0] = int32(ckKey)
	ld[1] = int32(uint64(ckKey) >> 32)

	ck.rk[0] = -1209970333
	for i := 1; i < roundKeys; i++ {
		ck.rk[i] = ck.rk[i-1] + -1640531527
	}
	var a, b int32
	var i, j int

	for k := 0; k < 3*roundKeys; k++ {
		ck.rk[i] = rotateLeft(ck.rk[i]+(a+b), 3)
		a = ck.rk[i]
		ld[j] = rotateLeft(ld[j]+(a+b), a+b)
		b = ld[j]
		i = (i + 1) % roundKeys
		j = (j + 1) % 2
	}
	return ck
}

func (ck ckCipher) encrypt(in int64) int64 {
	a := int32(in) + ck.rk[0]
	b := int32(uint64(in)>>32) + ck.rk[1]
	for r := 1; r <= rounds; r++ {
		a = rotateLeft(a^b, b) + ck.rk[2*r]
		b = rotateLeft(b^a, a) + ck.rk[2*r+1]
	}
	return pkLong(a, b)
}

func (ck ckCipher) decrypt(in int64) int64 {
	a := int32(in)
	b := int32(uint64(in) >> 32)
	for i := rounds; i > 0; i-- {
		b = rotateRight(b-ck.rk[2*i+1], a) ^ a
		a = rotateRight(a-ck.rk[2*i], b) ^ b
	}
	b -= ck.rk[1]
	a -= ck.rk[0]
	return pkLong(a, b)
}

func rotateLeft(x int32, y int32) int32 {
	return int32(x<<(y&(32-1))) | int32(uint32(x)>>(32-(y&(32-1))))
}

func rotateRight(x int32, y int32) int32 {
	return int32(uint32(x)>>(y&(32-1))) | int32(x<<(32-(y&(32-1))))
}

func pkLong(a int32, b int32) int64 {
	return (int64(a) & 0xffffffff) | (int64(b) << 32)
}


```

但到了 Linux 上, 就会报错:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/403a470bbdf447ce88b76510a1d42a9b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1716&h=630&s=492637&e=png&b=fefefe)

```
# command-line-arguments
./http_register.go:113:15: invalid operation: uint64(in) >> i (shift count type int, must be unsigned integer)
./http_register.go:175:16: invalid operation: x << (y & (32 - 1)) (shift count type int32, must be unsigned integer)
./http_register.go:175:47: invalid operation: uint32(x) >> (32 - y & (32 - 1)) (shift count type int32, must be unsigned integer)
./http_register.go:179:24: invalid operation: uint32(x) >> (y & (32 - 1)) (shift count type int32, must be unsigned integer)
./http_register.go:179:47: invalid operation: x << (32 - y & (32 - 1)) (shift count type int32, must be unsigned integer)


```

而在 Mac 上进行[交叉编译](https://link.juejin.cn?target=https%3A%2F%2Fstudygolang.com%2Farticles%2F14376 "https://studygolang.com/articles/14376"):

```
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build 文件名.go


```

(如果出现 GOROOT blabla 之类的, 执行 _go env -w GO111MODULE=off_ )

也没有什么问题

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/534b6326cbbe48c7ab2626b0a3ce73f5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1208&h=322&s=162941&e=png&b=fefefe)

导致这种情况的原因, 可能因 Go 版本不同而导致

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a37a32da2674a50ad79902aba1c945b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=554&h=104&s=30186&e=png&b=ffffff)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b71b76bd5fae4606a48263461d833470~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=858&h=98&s=58218&e=png&b=fefdfd)

Mac 上的 Go 版本为 1.16, 而 Linux 上 Go 版本为 1.11

### 解决:

最初想看一下有没有在线的不同 Go 版本执行工具, 无果而终.

想到之前用 php 时, 用过`brew switch`来切换不同的 php 版本. 但搜索之后发现, 这个命令被 brew 弃用了.

之前用过 node 版本工具`nvm`, 于是试图找寻 Go 有没有类似工具, 发现了 gvm.

> 安装 gvm

```
$ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)


```

N.B. : 不同操作系统还需要安装所需依赖，如 Mac 需要执行`brew install mercurial`安装 mercurial。具体见[原始项目](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fmoovweb%2Fgvm%23mac-os-x-requirements "https://github.com/moovweb/gvm#mac-os-x-requirements")

另外如果出现 “ERROR: Unrecognized command line argument: 'use'” 报错，可参考[此文](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fqq_45534061%2Farticle%2Fdetails%2F112256901 "https://blog.csdn.net/qq_45534061/article/details/112256901"), 执行：

```
cp -rp .gvm/scripts/env/use .gvm/scripts/use
chmod 775 .gvm/scripts/use


```

> 安装 go 1.11

```
gvm install go1.11


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26dbdd54bc8d43a38687a1f591a0e473~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1652&h=658&s=263044&e=png&b=ffffff)

> 选择版本

```
gvm use go1.11


```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89e4294eff204637862962c90a936573~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=632&h=212&s=58779&e=png&b=ffffff)

果然已经变为 Go 1.11

在 Go 1.11 环境下执行, 果然出现了和在 Linux 上 Go 1.11 下出现的同样错误

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c9b17c1a3dd485095bbe9b5c71342be~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1848&h=684&s=408561&e=png&b=ffffff)

### gvm 更多命令

> 查看版本

```
➜ gvm list

gvm gos (installed)

=> go1.11
   system


```

> 查看 Go 的所有版本 (版本来源于源码中的 tag 标签)

点击查看 Go 所有版本:

```
 gvm listall

gvm gos (available)

   go1
   go1.0.1
   go1.0.2
   go1.0.3
   go1.1
   go1.1rc2
   go1.1rc3
   go1.1.1
   go1.1.2
   go1.2
   go1.2rc2
   go1.2rc3
   go1.2rc4
   go1.2rc5
   go1.2.1
   go1.2.2
   go1.3
   go1.3beta1
   go1.3beta2
   go1.3rc1
   go1.3rc2
   go1.3.1
   go1.3.2
   go1.3.3
   go1.4
   go1.4beta1
   go1.4rc1
   go1.4rc2
   go1.4.1
   go1.4.2
   go1.4.3
   go1.5
   go1.5beta1
   go1.5beta2
   go1.5beta3
   go1.5rc1
   go1.5.1
   go1.5.2
   go1.5.3
   go1.5.4
   go1.6
   go1.6beta1
   go1.6beta2
   go1.6rc1
   go1.6rc2
   go1.6.1
   go1.6.2
   go1.6.3
   go1.6.4
   go1.7
   go1.7beta1
   go1.7beta2
   go1.7rc1
   go1.7rc2
   go1.7rc3
   go1.7rc4
   go1.7rc5
   go1.7rc6
   go1.7.1
   go1.7.2
   go1.7.3
   go1.7.4
   go1.7.5
   go1.7.6
   go1.8
   go1.8beta1
   go1.8beta2
   go1.8rc1
   go1.8rc2
   go1.8rc3
   go1.8.1
   go1.8.2
   go1.8.3
   go1.8.4
   go1.8.5
   go1.8.5rc4
   go1.8.5rc5
   go1.8.6
   go1.8.7
   go1.9
   go1.9beta1
   go1.9beta2
   go1.9rc1
   go1.9rc2
   go1.9.1
   go1.9.2
   go1.9.3
   go1.9.4
   go1.9.5
   go1.9.6
   go1.9.7
   go1.10
   go1.10beta1
   go1.10beta2
   go1.10rc1
   go1.10rc2
   go1.10.1
   go1.10.2
   go1.10.3
   go1.10.4
   go1.10.5
   go1.10.6
   go1.10.7
   go1.10.8
   go1.11
   go1.11beta1
   go1.11beta2
   go1.11beta3
   go1.11rc1
   go1.11rc2
   go1.11.1
   go1.11.2
   go1.11.3
   go1.11.4
   go1.11.5
   go1.11.6
   go1.11.7
   go1.11.8
   go1.11.9
   go1.11.10
   go1.11.11
   go1.11.12
   go1.11.13
   go1.12
   go1.12beta1
   go1.12beta2
   go1.12rc1
   go1.12.1
   go1.12.2
   go1.12.3
   go1.12.4
   go1.12.5
   go1.12.6
   go1.12.7
   go1.12.8
   go1.12.9
   go1.12.10
   go1.12.11
   go1.12.12
   go1.12.13
   go1.12.14
   go1.12.15
   go1.12.16
   go1.12.17
   go1.13
   go1.13beta1
   go1.13rc1
   go1.13rc2
   go1.13.1
   go1.13.2
   go1.13.3
   go1.13.4
   go1.13.5
   go1.13.6
   go1.13.7
   go1.13.8
   go1.13.9
   go1.13.10
   go1.13.11
   go1.13.12
   go1.13.13
   go1.13.14
   go1.13.15
   go1.14
   go1.14beta1
   go1.14rc1
   go1.14.1
   go1.14.2
   go1.14.3
   go1.14.4
   go1.14.5
   go1.14.6
   go1.14.7
   go1.14.8
   go1.14.9
   go1.14.10
   go1.14.11
   go1.14.12
   go1.14.13
   go1.14.14
   go1.14.15
   go1.15
   go1.15beta1
   go1.15rc1
   go1.15rc2
   go1.15.1
   go1.15.2
   go1.15.3
   go1.15.4
   go1.15.5
   go1.15.6
   go1.15.7
   go1.15.8
   go1.15.9
   go1.15.10
   go1.15.11
   go1.15.12
   go1.16
   go1.16beta1
   go1.16rc1
   go1.16.1
   go1.16.2
   go1.16.3
   go1.16.4
   release.r56
   release.r57
   release.r58
   release.r59
   release.r60
   release.r57.1
   release.r57.2
   release.r58.1
   release.r58.2
   release.r60.1
   release.r60.2
   release.r60.3


``` 

### 其他方案 --IDE

不管对于 Python, 还是 Golang, 面对版本问题时, 使用 PyCharm 或 GoLand, 始终是极好的选择

但现在只能下载最近几个版本，更久远的无法安装

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32d18424d5134315be3e3d62f27e24b5~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=934&h=490&s=83775&e=png&b=3a3d3f)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9214845c576e408599fc8ddc31f521dc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1728&h=452&s=148485&e=png&b=3b3e40)

### 其他方案 --Docker

`docker pull golang:1.xx`

`docker run -it --name golang-1.xx自定义名称 golang:1.xx`

进入到容器中 (默认是基于 ubuntu 的镜像)

(如果没有 vi/vim) 切换源，安装 vim

```
mv /etc/apt/sources.list /etc/apt/sources.list.bak

echo  "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free" >/etc/apt/sources.list

echo  "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free" >>/etc/apt/sources.list

echo  "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free" >>/etc/apt/sources.list

echo  "deb http://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free" >>/etc/apt/sources.list

apt-get update

apt-get install -y vim



```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc21d3278b3740dea06fec9756e46958~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1430&h=1078&s=233436&e=png&b=fefefe)

这种方式是万能的，任何版本都可以得到 (但是都是 Linux 版本的~)

### 其他方案 --brew

但如果想用 Mac 的以往版本的 Go，用 Docker 方式做不到~

还可以用 brew

可以在[这里](https://link.juejin.cn?target=https%3A%2F%2Fformulae.brew.sh%2Fformula%2Fgo%23default "https://formulae.brew.sh/formula/go#default")找到所有可以安装的 Go 版本

(从 1.16 开始支持 m1)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5e726f093694314886367ae9bb223c2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=652&h=953&s=119893&e=png&b=2b2722)

### 问题解决

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f096f14b0774507bd9bbc48d6703f62~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1972&h=1470&s=1324398&e=png&b=2f3133)

切换为 Go 1.11.5 版本后, IDE 会自动报错

y 为`int32`, 将其强制转为`uint32`即可,

i 为`int`, 将其强制转为`uint`即可.

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a59061703e7747b29a4999146125dd28~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1946&h=1222&s=1095876&e=png&b=2f3133)

参考:

[如何灵活地进行 Go 版本管理](https://juejin.cn/post/6844903949137346573 "https://juejin.cn/post/6844903949137346573")

类似的工具还有 [g](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvoidint%2Fg "https://github.com/voidint/g")

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c1ef381d3f54bd3bd8213ab5e6cf60d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp#?w=1034&h=350&s=48351&e=png&b=ffffff)

某位大佬推荐 [gvc](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fmoqsien%2Fgvc "https://github.com/moqsien/gvc"), 能管理很多种语言的版本