---
layout: post
title: linux目录结构
categories: linux
description: linux目录结构
keywords: linux
---

linux目录结构&磁盘与分区表示

#### / 根目录 ：所有的数据都放在此目录下（Linux系统的起点）
  - /dev：存放与设备相关的数据
  - Yvan
  - abc
  - haha

#### 如何使用硬盘--一块硬盘的“艺术”之旅
  - 物理硬盘----毛坯楼层
    - 分区规划----打隔断
    - 格式化----装修
    - 入驻----入驻
  - 格式化：指定空间存储数据规则（文件系统）
  - windows：NTFS FAT16 FAT32
  - Linux:
    - FAT32 ext4(RHEL6默认的文件系统) xfs(RHEL7默认的文件系统)
    - SWAP：交换空间（虚拟内存，主要作用：缓解真是物理内存不足）

#### 磁盘与分区表示
  - hd----表示IDE设备
  - sd----表示SCSI设备
  - vd----表示虚拟化设备
  - 磁盘----/根目录
    - /dev/sda----sd(SCSI)----a(第一块)
      - /dev/sda1---表示sd(SCSI)----a(第一块)第1个分区
      - /dev/sda7---表示sd(SCSI)----a(第一块)第7个分区
    - /dev/sdb----sd(SCSI)----b(第二块)
    - /dev/sdc----sd(SCSI)----c(第三块)
    - /dev/sdd----sd(SCSI)----d(第四块)