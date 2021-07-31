> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI4NTM1NDgwNw==&mid=2247494578&idx=2&sn=f94dc0acd3c6dece44bdc3dbdb351a9f&chksm=ebefd5eadc985cfcc794752f5cd92cb8ab1d3974a75da5b602076534d98fcc3aea51d484e318&scene=21#wechat_redirect)

**大家好，我是磊哥。**

在真实业务场景中，数据库中经常需要存储某些客户的关键性敏感信息如：身份证号、银行卡号、姓名、手机号码等，此类信息按照合规要求，通常需要实现加密存储以满足合规要求。

痛点一
---

通常的解决方案是我们书写 SQL 的时候，把对应的加密字段手动进行加密再进行插入，在查询的时候使用之前再手动进行解密。此方法固然可行，但是使用起来非常不便捷且繁琐，使得日常的业务开发与存储合规的细节紧耦合

痛点二
---

对于一些为了快速上线而一开始没有实现合规脱敏的系统，如何比较快速的使得已有业务满足合规要求的同时，尽量减少对原系统的改造。（通常的这个过程至少包括：

**1、** 新增脱敏列的存储

**2、** 同时做数据迁移

**3、** 业务的代码做兼容逻辑等

Apache ShardingSphere 下面存在一个数据脱敏模块，此模块集成的常用的数据脱敏的功能。其基本原理是对用户输入的 SQL 进行解析拦截，并依靠用户的脱敏配置进行 SQL 的改写，从而实现对字段的加密及加密字段的解密。最终实现对用户无感的加解密存储、查询。

脱敏配置 Quick Start——Spring 显示配置：
------------------------------

以下介绍基于 Spring 如何快速让系统支持脱敏配置。

### 1、引入依赖

```
<!-- for spring namespace -->
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-namespace</artifactId>
    <version>${sharding-sphere.version}</version>
</dependency>
```

### 2、创建脱敏配置规则对象

在创建数据源之前，需要准备一个 EncryptRuleConfiguration 进行脱敏的配置，以下是一个例子，对于同一个数据源里两张表 card_info，pay_order 的不同字段进行 AES 的加密

```
private EncryptRuleConfiguration getEncryptRuleConfiguration() {
Properties props = new Properties();

//自带aes算法需要
props.setProperty("aes.key.value", aeskey);
EncryptorRuleConfiguration encryptorConfig = new EncryptorRuleConfiguration("AES", props);

//自定义算法
//props.setProperty("qb.finance.aes.key.value", aeskey);
//EncryptorRuleConfiguration encryptorConfig = new EncryptorRuleConfiguration("QB-FINANCE-AES", props);

EncryptRuleConfiguration encryptRuleConfig = new EncryptRuleConfiguration();
encryptRuleConfig.getEncryptors().put("aes", encryptorConfig);

//START: card_info 表的脱敏配置
{
    EncryptColumnRuleConfiguration columnConfig1 = new EncryptColumnRuleConfiguration("", "name", "", "aes");
    EncryptColumnRuleConfiguration columnConfig2 = new EncryptColumnRuleConfiguration("", "id_no", "", "aes");
    EncryptColumnRuleConfiguration columnConfig3 = new EncryptColumnRuleConfiguration("", "finshell_card_no", "", "aes");
    Map<String, EncryptColumnRuleConfiguration> columnConfigMaps = new HashMap<>();
    columnConfigMaps.put("name", columnConfig1);
    columnConfigMaps.put("id_no", columnConfig2);
    columnConfigMaps.put("finshell_card_no", columnConfig3);
    EncryptTableRuleConfiguration tableConfig = new EncryptTableRuleConfiguration(columnConfigMaps);
    encryptRuleConfig.getTables().put("card_info", tableConfig);
}

//END: card_info 表的脱敏配置

//START: pay_order 表的脱敏配置
{
    EncryptColumnRuleConfiguration columnConfig1 = new EncryptColumnRuleConfiguration("", "card_no", "", "aes");
    Map<String, EncryptColumnRuleConfiguration> columnConfigMaps = new HashMap<>();
    columnConfigMaps.put("card_no", columnConfig1);
    EncryptTableRuleConfiguration tableConfig = new EncryptTableRuleConfiguration(columnConfigMaps);
    encryptRuleConfig.getTables().put("pay_order", tableConfig);
}

log.info("脱敏配置构建完成:{} ", encryptRuleConfig);
return encryptRuleConfig;
}
```

说明：

**1、** 创建 EncryptColumnRuleConfiguration 的时候有四个参数，前两个参数分表叫 plainColumn、cipherColumn，其意思是数据库存储里面真实的两个列（名文列、脱敏列），对于新的系统，只需要设置脱敏列即可，所以以上示例为 plainColumn 为”“。

**2、** 创建 EncryptTableRuleConfiguration 的时候需要传入一个 map，这个 map 存的 value 即 #1 中说明的 EncryptColumnRuleConfiguration ，而其 key 则是一个逻辑列，对于新系统，此逻辑列即为真实的脱敏列。Sharding Shpere 在拦截到 SQL 改写的时候，会按照用户的配置，把逻辑列映射为名文列或者脱敏列（默认）如下的示例

![](https://mmbiz.qpic.cn/mmbiz_png/GpcH5Yqqj0nff7h7ZMUnazUFy3T6ULjMT0kgPlpQKmGpFSZJGU7ibdTyrqDqmgRVtwVLQ0agexBOM0EY2icpL06w/640?wx_fmt=png)

  

### 3、使用 Sharding Sphere 的数据源进行管理

把原始的数据源包装一层

```
@Bean("tradePlatformDataSource")
public DataSource dataSource(@Qualifier("druidDataSource") DataSource ds) throws SQLException {
    return EncryptDataSourceFactory.createDataSource(ds, getEncryptRuleConfiguration(), new Properties());

}
```

脱敏配置 Quick Start——Spring Boot 版：
--------------------------------

以下步骤使用 Spring Boot 管理，可以仅用配置文件解决：

**1、** 引入依赖

```
<!-- for spring boot -->

<dependency>
<groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>${sharding-sphere.version}</version>
</dependency>

<!-- for spring namespace -->
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-namespace</artifactId>
    <version>${sharding-sphere.version}</version>
</dependency>
```

**2、** Spring 配置文件

```
spring.shardingsphere.datasource.name=ds
spring.shardingsphere.datasource.ds.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ds.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds.url=xxxxxxxxxxxxx
spring.shardingsphere.datasource.ds.username=xxxxxxx
spring.shardingsphere.datasource.ds.password=xxxxxxxxxxxx

# 默认的AES加密器
spring.shardingsphere.encrypt.encryptors.encryptor_aes.type=aes
spring.shardingsphere.encrypt.encryptors.encryptor_aes.props.aes.key.value=hkiqAXU6Ur5fixGHaO4Lb2V2ggausYwW

# card_info 姓名 AES加密
spring.shardingsphere.encrypt.tables.card_info.columns.name.cipherColumn=name
spring.shardingsphere.encrypt.tables.card_info.columns.name.encryptor=encryptor_aes

# card_info 身份证 AES加密
spring.shardingsphere.encrypt.tables.card_info.columns.id_no.cipherColumn=id_no
spring.shardingsphere.encrypt.tables.card_info.columns.id_no.encryptor=encryptor_aes

# card_info 银行卡号 AES加密
spring.shardingsphere.encrypt.tables.card_info.columns.finshell_card_no.cipherColumn=finshell_card_no
spring.shardingsphere.encrypt.tables.card_info.columns.finshell_card_no.encryptor=encryptor_aes

# pay_order 银行卡号 AES加密
spring.shardingsphere.encrypt.tables.pay_order.columns.card_no.cipherColumn=card_no
spring.shardingsphere.encrypt.tables.pay_order.columns.card_no.encryptor=encryptor_aes
```

来源 | https://sourl.cn/uLCCPw