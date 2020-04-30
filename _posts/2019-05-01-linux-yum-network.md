---
layout: post
title: Linux配置yum源（网络）
categories: Linux
description: Linux配置yum源（网络）
keywords: yum
---


RedHat Linux自带的yum网络资源不如CentOS,所以需要先卸载自带yum，并下载安装centOS的yum.


### 检查yum源：
	rpm -qa | grep yum

### 删除原有的yum源：
	rpm -qa | grep yum | xargs rpm -e --nodeps

### 下载安装CentOS的yum源

- 阿里云网络源地址：https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/

- 网易163网络源地址：http://mirrors.163.com/

- CentOS网络源地址：http://centos.ustc.edu.cn/centos/
	- ①点击centos进入
	- ②找到7.6版本的
	- ③依此进入到os/x86_64/Packages中
	- ④Ctrl+F搜索yum-->找到下面几个-->点击鼠标右键-->复制链接地址
		- yum-3.4.3-158.el7.centos.noarch.rpm
		- yum-metadata-parser-1.1.4-10.el7.x86_64.rp
		- yum-plugin-fastestmirror-1.1.31-45.el7.noarch.rpm
	- ⑤在终端中通过wget指令下载（确保联网，可通过ping一下百度服务器看看联网没，Crtl+c终止命令）
		- wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-3.4.3-163.el7.centos.noarch.rpm
		- wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
		- wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-52.el7.noarch.rpm

	- ⑥为了防止几个包安装时有互相依赖，使用 rpm -ivh yum-* 命令一次性安装三个包

	- ⑦使用第一条命令检查yum是否安装成功：rpm -qa |grep yum

### 配置repo文件（关键！前面的能不能起作用就看这一步了）

- ①在/etc目录下重命名备份原来的repo：mv yum.repos.d yum.repos.d.backup

- ②建一个新的yum.repos.d目录（确保在/etc目录下）mkdir yum.repos.d

- ③下载一个CentOS的repo（我们可以在网易镜像站的centos使用帮助中下载学习）
	- wget http://mirrors.163.com/.help/CentOS7-Base-163.repo 下载

- ④通过vim打开并编辑repo
	- 将所有的$releasever全部替换成版本号-->7.5.1804：
	- shift+: 编辑 输入下面的指令
	- %s/$releasever/7/g
	- :wq保存

- ⑤yum clean all  ##清理缓存

- ⑥使用yum repolist all查看是否成功

### Redhat7使用CentOS7的yum源

#### 操作步骤如下：

1. 查看操作系统版本
```shell
[root@sky ~]# cat /etc/redhat-release
Red Hat Enterprise Linux Server release 7.0 (Maipo)
```

2. 查看系统已经安装的yum源
```shell
[root@sky ~]# rpm -qa|grep yum
yum-utils-1.1.31-24.el7.noarch
yum-langpacks-0.4.2-3.el7.noarch
yum-metadata-parser-1.1.4-10.el7.x86_64
yum-rhn-plugin-2.0.1-4.el7.noarch
PackageKit-yum-0.8.9-11.el7.x86_64
yum-3.4.3-118.el7.noarch
```

3. 卸载原有系统软件包
```shell
[root@sky ~]# rpm -e  yum-utils-1.1.31-24.el7.noarch  --nodeps
[root@sky ~]# rpm -e  yum-langpacks-0.4.2-3.el7.noarch  --nodeps
[root@sky ~]# rpm -e  yum-metadata-parser-1.1.4-10.el7.x86_64 --nodeps
[root@sky ~]# rpm  -e  yum-rhn-plugin-2.0.1-4.el7.noarch  PackageKit-yum-0.8.9-11.el7.x86_64   yum-3.4.3-118.el7.noarch  --nodeps
```

4. 保证本系统能上网
```shell
[root@sky ~]# ping www.baidu.com
PING www.a.shifen.com (61.135.169.121) 56(84) bytes of data.
64 bytes from 61.135.169.121: icmp_seq=1 ttl=128 time=40.7 ms
64 bytes from 61.135.169.121: icmp_seq=2 ttl=128 time=41.5 ms
64 bytes from 61.135.169.121: icmp_seq=3 ttl=128 time=40.3 ms
64 bytes from 61.135.169.121: icmp_seq=4 ttl=128 time=40.4 ms
```

5. 下载CentOS7版本的软件包
```shell
[root@sky ~]# cd /usr/local/src
[root@sky src]# wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-kitchen-1.1.1-5.el7.noarch.rpm
[root@sky src]# wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-chardet-2.2.1-1.el7_1.noarch.rpm
[root@sky src]# wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-3.4.3-158.el7.centos.noarch.rpm
[root@sky src]# wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
[root@sky src]# wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-utils-1.1.31-45.el7.noarch.rpm
[root@sky src]# wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-updateonboot-1.1.31-45.el7.noarch.rpm
[root@sky src]# wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-45.el7.noarch.rpm
[root@sky src]# ls
grafana-5.3.2-1.x86_64.rpm
httpd-2.2.15-69.el6.centos.x86_64.rpm
httpd-2.4.6-80.el7.centos.x86_64.rpm
python-chardet-2.2.1-1.el7_1.noarch.rpm
python-kitchen-1.1.1-5.el7.noarch.rpm
yum-3.4.3-158.el7.centos.noarch.rpm
yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
yum-plugin-fastestmirror-1.1.31-45.el7.noarch.rpm
yum-updateonboot-1.1.31-45.el7.noarch.rpm
yum-utils-1.1.31-45.el7.noarch.rpm
```

6. 安装软件包
```shell
[root@sky src]# rpm  -ivh python-*
[root@sky src]# rpm  -ivh yum-*  --nodeps
```

7. 配置repo文件
```shell
root@sky src]# cd  /etc/yum.repos.d/
[root@sky yum.repos.d]# ll
总用量 0
[root@sky yum.repos.d]# wget  http://mirrors.163.com/.help/CentOS7-Base-163.repo
[root@sky yum.repos.d]# cat  CentOS7-Base-163.repo
# CentOS-Base.repo
.....
[base]
name=CentOS-7 - Base - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#released updates
[updates]
name=CentOS-$releasever - Updates - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
baseurl=http://mirrors.163.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#additional packages that may be useful
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - 163.com
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
baseurl=http://mirrors.163.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
#将上述$releasever改成数字7。
```

8. 已安装完成CentOS7 yum 源
```shell
[root@sky yum.repos.d]# yum clean all
```

9. 测试是否成功配置yum源
```shell
[root@sky yum.repos.d]# yum
[root@sky /]# yum repolist all
已加载插件：fastestmirror, product-id, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Determining fastest mirrors
base                                                          | 3.6 kB  00:00:00
extras                                                        | 3.4 kB  00:00:00
updates                                                       | 3.4 kB  00:00:00
(1/4): base/x86_64/group_gz                                   | 166 kB  00:00:00
(2/4): extras/x86_64/primary_db                               | 204 kB  00:00:00
(3/4): updates/x86_64/primary_db                              | 6.0 MB  00:00:07
(4/4): base/x86_64/primary_db                                 | 5.9 MB  00:00:08
源标识                    源名称                                          状态
base/x86_64               CentOS-7 - Base - 163.com                       启用: 9,911
centosplus/x86_64         CentOS-$releasever - Plus - 163.com             禁用    #该项失败
extras/x86_64             CentOS-$releasever - Extras - 163.com           启用:   432
updates/x86_64            CentOS-$releasever - Updates - 163.com          启用: 1,614
repolist: 11,957
.........
root@sky yum.repos.d]# vi CentOS7-Base-163.repo
[centosplus]
name=CentOS-$releasever - Plus - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/  #releasever改成数字"7"
gpgcheck=1
enabled=0     #数字“0”改成"1"
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
```

到此步骤，yum源才配置成功。
实战技巧：不要做#yum update，还是保持原有redhat7版本，如果一升级，则版本会更换到CentOS7。
建议升级软件包#yum upgrade。