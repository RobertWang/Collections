> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2OTQxMTM4OQ==&mid=2247515370&idx=4&sn=db7b4235f91e1e7d33aafe343a10e678&chksm=eae249b8dd95c0ae87c6a6ed2ba0e43f0637c01cfb3d0a73671c2a24af236c92aac5e9c97703&mpshare=1&scene=1&srcid=08177ltV6ybizs8d85Fzp4jK&sharer_sharetime=1629189110886&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来源：https://my.oschina.net/xiaolei123/blog/3085607  

介绍
--

ProtoBuf 是 google 团队开发的用于高效存储和读取结构化数据的工具。什么是结构化数据呢，正如字面上表达的，就是带有一定结构的数据。比如电话簿上有很多记录数据，每条记录包含姓名、ID、邮件、电话等，这种结构重复出现。

同类
--

XML、JSON 也可以用来存储此类结构化数据，但是使用 ProtoBuf 表示的数据能更加高效，并且将数据压缩得更小。

原理
--

ProtoBuf 是通过 ProtoBuf 编译器将与编程语言无关的特有的 .proto 后缀的数据结构文件编译成各个编程语言 (Java,C/C++,Python) 专用的类文件, 然后通过 Google 提供的各个编程语言的支持库 lib 即可调用 API。(关于 proto 结构体怎么编写，可自行查阅文档)

ProtoBuf 编译器安装
--------------

Mac : brew install protobuf

举个例子
----

### 1. 先创建一个 proto 文件

message.proto

```
syntax = "proto3";
 
message Person {
    int32 id = 1;
    string name = 2;
    
    repeated Phone phone = 4;
    
    enum PhoneType {
        MOBILE = 0;
        HOME = 1;
        WORK = 2;
    }
 
    message Phone {
        string number = 1;
        PhoneType type = 2;
    }
}
```

### 2. 创建一个 Java 项目

并且将 proto 文件放置 src/main/proto 文件夹下

### 3. 编译 proto 文件至 Java 版本

用命令行 cd 到 `src/main` 目录下

终端执行命令 : `protoc --java_out=./java ./proto/*.proto`

会发现，在你的`src/main/java` 里已经生成里对应的 Java 类

### 4. 依赖 Java 版本的 ProtoBuf 支持库

这里只举一个用 Gradle 使用依赖的栗子

```
implementation 'com.google.protobuf:protobuf-java:3.9.1'
```

### 5. 将 Java 对象转为 ProtoBuf 数据

```
Message.Person.Phone.Builder phoneBuilder = Message.Person.Phone.newBuilder();
Message.Person.Phone phone1 = phoneBuilder
        .setNumber("100860")
        .setType(Message.Person.PhoneType.HOME)
        .build();
Message.Person.Phone phone2 = phoneBuilder
        .setNumber("100100")
        .setType(Message.Person.PhoneType.MOBILE)
        .build();
Message.Person.Builder personBuilder = Message.Person.newBuilder();
personBuilder.setId(1994);
personBuilder.setName("XIAOLEI");
personBuilder.addPhone(phone1);
personBuilder.addPhone(phone2);

Message.Person person = personBuilder.build();
long old = System.currentTimeMillis();
byte[] buff = person.toByteArray();
System.out.println("ProtoBuf 编码耗时：" + (System.currentTimeMillis() - old));
System.out.println(Arrays.toString(buff));
System.out.println("ProtoBuf 数据长度:" + buff.length);
```

### 6. 将 ProtoBuf 数据，转换回 Java 对象

```
System.out.println("-开始解码-");
old = System.currentTimeMillis();
Message.Person personOut = Message.Person.parseFrom(buff);
System.out.println("ProtoBuf 解码耗时：" + (System.currentTimeMillis() - old));
System.out.printf("Id:%d, Name:%s\n", personOut.getId(), personOut.getName());
List<Message.Person.Phone> phoneList = personOut.getPhoneList();
for (Message.Person.Phone phone : phoneList)
{
    System.out.printf("手机号:%s (%s)\n", phone.getNumber(), phone.getType());
}
```

比较
--

为了能体现 ProtoBuf 的优势，我写了同样结构体的 Java 类，并且将 Java 对象转换成 JSON 数据，来与 ProtoBuf 进行比较。JSON 编译库使用 Google 提供的 GSON 库，JSON 的部分代码就不贴出来了，直接展示结果

比较结果结果
------

运行 1 次

```
【 JSON 开始编码 】
JSON 编码1次，耗时：22ms
JSON 数据长度:106
-开始解码-
JSON 解码1次，耗时：1ms

【 ProtoBuf 开始编码 】
ProtoBuf 编码1次,耗时：32ms
ProtoBuf 数据长度:34
-开始解码-
ProtoBuf 解码1次,耗时：3ms
```

运行 10 次

```
【 JSON 开始编码 】
JSON 编码10次，耗时：22ms
JSON 数据长度:106
-开始解码-
JSON 解码10次，耗时：4ms

【 ProtoBuf 开始编码 】
ProtoBuf 编码10次,耗时：29ms
ProtoBuf 数据长度:34
-开始解码-
ProtoBuf 解码10次,耗时：3ms
```

运行 100 次

```
【 JSON 开始编码 】
JSON 编码100次，耗时：32ms
JSON 数据长度:106
-开始解码-
JSON 解码100次，耗时：8ms

【 ProtoBuf 开始编码 】
ProtoBuf 编码100次,耗时：31ms
ProtoBuf 数据长度:34
-开始解码-
ProtoBuf 解码100次,耗时：4ms
```

运行 1000 次

```
【 JSON 开始编码 】
JSON 编码1000次，耗时：39ms
JSON 数据长度:106
-开始解码-
JSON 解码1000次，耗时：21ms

【 ProtoBuf 开始编码 】
ProtoBuf 编码1000次,耗时：37ms
ProtoBuf 数据长度:34
-开始解码-
ProtoBuf 解码1000次,耗时：8ms
```

运行 1 万 次

```
【 JSON 开始编码 】
JSON 编码10000次，耗时：126ms
JSON 数据长度:106
-开始解码-
JSON 解码10000次，耗时：93ms

【 ProtoBuf 开始编码 】
ProtoBuf 编码10000次,耗时：49ms
ProtoBuf 数据长度:34
-开始解码-
ProtoBuf 解码10000次,耗时：23ms
```

运行 10 万 次

```
【 JSON 开始编码 】
JSON 编码100000次，耗时：248ms
JSON 数据长度:106
-开始解码-
JSON 解码100000次，耗时：180ms

【 ProtoBuf 开始编码 】
ProtoBuf 编码100000次,耗时：51ms
ProtoBuf 数据长度:34
-开始解码-
ProtoBuf 解码100000次,耗时：58ms
```

总结
--

### 编解码性能

上述栗子只是简单的采样，实际上据我的实验发现

*   次数在 1 千以下，ProtoBuf 的编码与解码性能，都与 JSON 不相上下，甚至还有比 JSON 差的趋势。
    
*   次数在 2 千以上，ProtoBuf 的编码解码性能，都比 JSON 高出很多。
    
*   次数在 10 万以上，ProtoBuf 的编解码性能就很明显了，远远高出 JSON 的性能。
    

### 内存占用

ProtoBuf 的内存 34，而 JSON 到达 106 ，ProtoBuf 的内存占用只有 JSON 的 1/3.

结尾
--

其实这次实验有很多可待优化的地方，就算是这种粗略的测试，也能看出来 ProtoBuf 的优势。

兼容
--

### 新增字段

*   在 proto 文件中新增 nickname 字段
    
*   生成 Java 文件
    
*   用老 proto 字节数组数据，转换成对象
    

```
Id:1994, Name:XIAOLEI
手机号:100860 (HOME)
手机号:100100 (MOBILE)
getNickname=
```

结果，是可以转换成功。

### 删除字段

*   在 proto 文件中删除 name 字段
    
*   生成 Java 文件
    
*   用老 proto 字节数组数据，转换成对象
    

```
Id:1994, Name:null
手机号:100860 (HOME)
手机号:100100 (MOBILE)
```

结果，是可以转换成功。