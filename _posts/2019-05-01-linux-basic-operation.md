---
layout: post
title: Linux基本操作
categories: Linux
description: Linux基本操作
keywords: Linux
---


linux系统的一些基本操作及解释


##### 切换命令行界面
	虚拟控制台切换（Ctrl+Alt+Fn组合键）
		- tty1:图形桌面
		- tty2~tty6:字符控制台

##### 命令提示标识的含义
    [当前用户@主机名 ~]#
    #表示管理员用户
    $表示普通用户

##### 查看及切换目录
    pwd - Print Working Directory---查看当前工作目录
    cd - Change Directory---切换工作目录
    ls - List---格式：ls[选项]...[目录或文件名]...

##### 查看系统版本
    检查红帽发行信息
    cat /etc/redhat-release

##### 列出内核版本
    uname -r

##### 列出CPU处理器信息
    lscpu

##### 检查内存大小、空闲情况
    cat /proc/meminfo

##### 列出当前主机名
    hostname

##### 修改主机名
    hostname name

##### 列出已激活的网卡连接信息
    ifconfig
    特殊IP地址:127.0.0.1（永远代表本机）
    ifconfig eht0 192.168.1.1配置临时IP

##### 创建文档
    创建目录mkdir---Make Directory [/路径/]目录名
    创建文件touch [/路径/]文件名

##### 分屏阅读工具
    less [/路径/]文件
    查找内容---/a 退出 q

##### 文本内容操作
    head -n数字 文件名--列出文件前n行
    tail -n数字 文件名---列出文件后n行
    grep：输出包含指定字符串的行---grep [选项]... ‘查找条件’ 目标文件