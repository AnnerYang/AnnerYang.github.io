---
layout: post
title: Linux软件包管理
categories: Linux
description: Linux软件包管理
keywords: software,package
---


在Linux下安装软件, 一个通常的办法是下载到程序的源代码, 并进行编译, 得到可执行程序.
但是这样太麻烦了, 于是有些人把一些常用的软件提前编译好, 做成软件包(可以理解成windows上的安 装程序)放在一个服务器上, 通过包管理器可以很方便的获取到这个编译好的软件包, 直接进行安装.
软件包和软件包管理器, 就好比 "App" 和 "应用商店" 这样的关系.
yum(Yellow dog Updater, Modiﬁed)是Linux下非常常用的一种包管理器. 主要应用在Fedora, RedHat


#### 远程管理

- 远程管理命令：ssh 登录用户名@主机IP地址

- 打开新终端：Ctrl + Shift +t

- ssh -x：在远程管理时，本地运行对方图形程序

#### 设置永久别名，修改配置文件

- 设置永久别名配置文件：/root/.bashrc

- alias 命令设置别名 别名=' ssh -x root@10.212.254.233'

- 设置完成后需重新打开终端生效

#### 软件包管理

- 具备软件包

- 下载软件包
	- wget http://......

- 安装软件包的命令
	
	- 使用rpm命令管理软件，默认不允许用户做任何选择

	- RPM Package Manager,RPM包管理器
		- rpm -q 软件名... 	#查询当前系统软件包是否安装
		- rpm -ivh 软件名-版本信息.rmp... 		#安装软件包
		- rpm -e 软件名... 	#卸载软件包
		- rpm -ql 软件名... 	#查看安装清单

- yum软件包仓库，自动解决依赖关系

	- 服务：为客户端自动解决依赖关系，安装软件包
		- 服务端
			- 众多的软件包
			- 仓库清单文件--repodata
			- 构建web服务传递数据

	- 客户端

		- 客户端配置文件--/etc/yum.repos.d/*.repo
			- 错误的配置文件会影响正确的配置文件

		- vim /etc/yum.repos.d/yum.repo
			- [rhel7] 			#仓库标识
			- name=rhel7.0	      #仓库的描述信息
			- baseurl=http://classrom.example.com/content/rhel7.0/x86_64/dvd/ 	#指定服务端位置
			- enabled=1 	      #是否启用该文件
			- gpgcheck=0 	      #是否检测红帽签名

	- yum使用
		- yum repolist			#列出仓库信息
		- yum -y install httpd	  	#安装软件包
		- yum remove 软件包	 #卸载软件包不要使用-y选项
		- yum clean all			 #清缓存
		- yum list [软件名]...		 #列软件

	- 快速建立repo配置文件
		- yum-config-manager --add-repo 软件仓库地址

	- 升级内核
		- 下载软件包到/opt文件夹
		- 内核软件：kernel-3.10.0-957.el7.x86_64
		- rpm -ivh kernel-3.10.0-957.el7.x86_64安装
		- 安装完成后重启


