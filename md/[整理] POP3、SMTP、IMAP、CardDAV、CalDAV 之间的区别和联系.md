> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/feily/p/15210847.html) 目录

*   [SMTP 管 “发”，POP3/IMAP 管 “收”。](#smtp管发pop3imap管收)
*   [概念](#概念)
    *   [POP3(Post Office Protocol)](#pop3post-office-protocol)
    *   [SMTP (Simple Mail Transfer Protocol)](#smtp-simple-mail-transfer-protocol)
    *   [IMAP(Internet Mail Access Protocol)](#imapinternet-mail-access-protocol)
    *   [SSL(Secure Sockets Layer)](#sslsecure-sockets-layer)
    *   [CardDAV、CalDAV](#carddavcaldav)

看了一下 QQ 邮箱的设置，发现邮箱服务这块的协议之前理解的不是很清楚，特此整理一下：

SMTP 管 “发”，POP3/IMAP 管 “收”。
===========================

你坐在电脑边用 mail client 写完邮件，点击 “发送”。这时你的 mail client 会发消息给邮件服务器上的 SMTP service。这时有两种情况：

1.  如果邮件的收信人也是处于同一个 domain，比如从 http://163.com 发送给 163 的邮箱，SMTP service 只需要转给 local 的 POP3 Service 即可
2.  如果邮件收信人是另外的 domain，比如 http://163.com 发送给 http://sina.com， SMTP service 需要通过询问 DNS, 找到属于 sina 的 SMTP service 的 host

概念
==

POP3(Post Office Protocol)
--------------------------

`邮局通讯协定`POP 是互联网上的一种通讯协定（比较老），主要功能是用在传送电子邮件，当我们寄信给另外一个人时，对方当时多半不会在线上，所以邮件服务器必须为收信者保存这封信，直到收信者来检查这封信件。当收信人收信的时候，必须通过 POP 通讯协定，才能取得邮件。

SMTP (Simple Mail Transfer Protocol)
------------------------------------

与 POP 同时出现的还有 SMTP(简易邮件传输通讯协议)，它也是用来传送网络上的电子邮件，不同的是 POP 是负责邮件程序和邮件服务器收信的通讯协定，SMTP 则是负责邮件服务器与邮件服务器之间的寄信的通讯协定。

IMAP(Internet Mail Access Protocol)
-----------------------------------

`交互式邮件存取协议`（比较新），是一个应用层协议（端口是 143）。IMAP 整体上为用户带来更为便捷和可靠的体验。POP3 更易丢失邮件或多次下载相同的邮件，但 IMAP 通过邮件客户端与 webmail 之间的双向同步功能很好地避免了这些问题。

SSL(Secure Sockets Layer)
-------------------------

`安全套接层`，及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议。TLS 与 SSL 在传输层对网络连接进行加密，客户与服务器应用之间的通信不被攻击者窃听。

CardDAV、CalDAV
--------------

一种通讯录同步的开放协议。使用 CardDAV 同步的通讯录可以编辑、修改或者删除，并且你在手机上的这些操作也同样会和服务器同步，并同时同步到你的其他设备上。  
CalDAV 是一种用于存取网络行事历及行程或会议排程的 client/server 协议。