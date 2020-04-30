---
layout: post
title: rhel使用iso配置本地源
categories: Linux
description: rhel使用iso配置本地源
keywords: 
---


Redhat 系统想要直接在线通过yum的条件时需要注册，一般用户都是非注册的，这个时候如果要想通过yum安装新软件，我们可以通过将安装盘镜像ISO文件设置为yum源的方式来进行。



### rhel使用iso配置本地源

#### 创建文件夹

- mkdir /opt/iso #创建iso存放目录

- mkdir /mnt/cdrom #创建iso挂在目录

#### 上传iso并挂载

- 使用的是MobaXterm

- cd /etc/yum.repos.d/                               #进入/etc/yum.repos.d/目录

- touch yum.repo                                     #创建yum.repo文件

#### 配置本地yum源-----注意配置文件中不能有注释

- [rhel7.6-yum]

- name=redhat7.6 #自定义名称

- baseurl=file:///mnt/cdrom #本地iso文件的挂载路径

- enabled=1 #启用yum源，0为不启用，1为启用

- gpgcheck=1 #检查GPG-KEY，0为不检查，1为检查

- gpgkey=file:///mnt/cdrom/RPM-GPG-KEY-redhat-release #GPG-KEY路径，就在ISO文件的根目录，红帽公司的软件签名。
#### 测试

- yum clean all #清除yum缓存

- yum repolist all #显示可用的yum源

- yum install mesa-* #安装软件包
#### 开机自动挂载

- 修改开机引导配置文件

- 编辑/etc/fstab最后一行添加