> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.ibm.com](https://www.ibm.com/docs/zh/rtw/9.0.1?topic=transports-mq-transport-read-write-browse-queue-settings)

> IBM Documentation.

使用选项选项卡可以指定必须用于读取、写入和浏览消息的 MQ 传输队列标识。

![](https://www.ibm.com/docs/zh/SSBLQQ_9.0.1/com.ibm.rational.rit.integration.doc/images/MQ_option_tab.jpg)

要指定用于读取、编写或浏览消息的队列选项，请执行以下操作：

1.  单击 ![](https://www.ibm.com/docs/zh/SSBLQQ_9.0.1/com.ibm.rational.rit.integration.doc/images/more_button.jpg) 以选择操作支持的选项。要读取，选择以 “MQOO_INPUT_” 开头的某个“读取队列打开选项”。缺省值为 MQOO_INPUT_SHARED。要写入，选择 MQOO_OUTPUT。要浏览，选择 MQOO_BROWSE。
2.  单击确定。所选选项会转换为字段中显示的小数值。

设置消息读取格式
--------

选择选项将字符串数据视为字节后，将使 Rational® Integration Tester 能够读取 MQSTR 格式的 MQ 消息的主体作为原始字节而不是作为字符串。

![](https://www.ibm.com/docs/zh/SSBLQQ_9.0.1/com.ibm.rational.rit.integration.doc/images/MQ_option_tab_msg_format.jpg)

通过使用该选项，可处理字节而不是字符串数据。当接收到的消息的字符集信息不正确时，这非常有用。启用该选项后，执行以下步骤：

*   进入测试或存根中，并手动向消息主体的**数据字段**应用字节模式。在 Rational Integration Tester 中，缺省情况下，MQSTR 格式的 MQ 消息在主体中包含**文本字段**。要更新消息，以便它可处理字节数据，首先删除现有文本字段，添加**数据字段**，然后应用字节模式。
*   选择可用于将消息视为字符串的编码方法。这样做会消除自动转换的需求。自动转换在某些时候会因为字符集信息不正确而生成意外结果。

从 Rational Integration Tester 发送回消息时，例如在存根中使用传递操作，那么将自动取消从 MQSTR 到字节的转换，而且消息将以其原始格式（MQSTR 格式）到达队列。