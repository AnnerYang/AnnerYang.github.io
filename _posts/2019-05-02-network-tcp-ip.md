---
layout: post
title: TCP/IP协议及配置
categories: NetWork
description: TCP/IP协议及配置
keywords: TCP/IP
---



TCP/IP（Transmission Control Protocol/Internet Protocol，传输控制协议/网际协议）是指能够在多个不同网络间实现信息传输的协议簇。TCP/IP协议不仅仅指的是[TCP](https://baike.baidu.com/item/TCP/33012) 和[IP](https://baike.baidu.com/item/IP/224599)两个协议，而是指一个由[FTP](https://baike.baidu.com/item/FTP/13839)、[SMTP](https://baike.baidu.com/item/SMTP/175887)、TCP、[UDP](https://baike.baidu.com/item/UDP/571511)、IP等协议构成的协议簇， 只是因为在TCP/IP协议中TCP协议和IP协议最具代表性，所以被称为TCP/IP协议。



#### TCP/IP协议的组成

TCP/IP协议在一定程度上参考了OSI的体系结构。OSI模型共有七层，从下到上分别是物理层、数据链路层、网络层、运输层、会话层、表示层和应用层。但是这显然是有些复杂的，所以在TCP/IP协议中，它们被简化为了四个层次。

1. 应用层、表示层、会话层三个层次提供的服务相差不是很大，所以在TCP/IP协议中，它们被合并为应用层一个层次。
2. 由于运输层和网络层在网络协议中的地位十分重要，所以在TCP/IP协议中它们被作为独立的两个层次。
3. 因为数据链路层和物理层的内容相差不多，所以在TCP/IP协议中它们被归并在网络接口层一个层次里。只有四层体系结构的TCP/IP协议，与有七层体系结构的OSI相比要简单了不少，也正是这样，TCP/IP协议在实际的应用中效率更高，成本更低。


- 应用层：应用层是TCP/IP协议的第一层，是直接为应用进程提供服务的。
1. 对不同种类的应用程序它们会根据自己的需要来使用应用层的不同协议，邮件传输应用使用了SMTP协议、万维网应用使用了HTTP协议、远程登录服务应用使用了有TELNET协议。
2. 应用层还能加密、解密、格式化数据。
3. 应用层可以建立或解除与其他节点的联系，这样可以充分节省网络资源。
- 运输层：作为TCP/IP协议的第二层，运输层在整个TCP/IP协议中起到了中流砥柱的作用。且在运输层中，TCP和UDP也同样起到了中流砥柱的作用。
- 网络层：网络层在TCP/IP协议中的位于第三层。在TCP/IP协议中网络层可以进行网络连接的建立和终止以及IP地址的寻找等功能。
- 网络接口层：在TCP/IP协议中，网络接口层位于第四层。由于网络接口层兼并了物理层和数据链路层所以，网络接口层既是传输数据的物理媒介，也可以为网络层提供一条准确无误的线路。



#### IP地址分类（网：网络位；主：主机位）
  - 用于一般计算机网络
    - A类：1~127 网+主+主+主
    - B类：128~191 网+网+主+主
    - C类：192~233 网+网+网+主
  - 组播及科研专用
    - D类：224~239 组播
    - E类：240~254 科研

#### IP地址
  - 用于标志计算机的地址，具有唯一性

#### 子网掩码
  - 区分IP地址的网络位与主机位
  - 用二进制的1表示网络位，用二进制的0表示主机位
  - IP地址由网络位与主机位组成
    - 座机号：区号+区号
      - 010-12345678
      - 0375-5685905
      - 区号：标识一个区域
      - 号码：在区域中座机的标号
    - 192.168.1.1
      - 来自192.168.1区域（网络）
      - 编号为1的主机
    - 192.168.2.1
      - 来自192.168.2区域（网络）
      - 编号为1的主机
    - 192.168.10.1
      - 第一个数字位192，C类地址，网+网+网+主
      - 来自192.168.10区域（网络），编号为1的主机
    - IP地址与子网掩码对应信息
      - 1100 0000‬（192）.10101000（168）.00001010（10）.00000001（1）
      - 11111111（255）.11111111（255）.11111111（255）.00000000（0）
      - 也可表示为192.168.10.1/24（24：表示24个网络位）

#### 网关（跨网络通信）
  - 什么是网关
    - 从一个网络连接到另一个网络的“关口”
    - 通常是一台里尤其，或者防火墙/接入服务器

#### DNS服务器的作用
  - 将域名解析为IP地址
    - www.qq.com-------->IP地址