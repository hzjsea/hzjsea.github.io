---
title: linux命令lspci
date: 2020-09-23 10:49:04
categories: [linux,]  
tags: [linuxcommand,linux,net]
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->   

# linux命令lspci

安装
```bash
yum install pciutils -y
```

## 什么是PCI
PCI是一种外设总线规范。我们先来看一下什么是总线：总线是一种传输信号的路径或信道。典型情况是，总线是连接于一个或多个导体的电气连线，总线上连接的所有设备可在同一时间收到所有的传输内容。总线由电气接口和编程接口组成。本文讨论Linux 下的设备驱动，所以，重点关注编程接口。
PCI是Peripheral Component Interconnect（外围设备互联）的简称，是普遍使用在桌面及更大型的计算机上的外 设总线。PCI架构被设计为ISA标准的替代品，它有三个主要目标：
- 获得在计算机和外设之间传输数据时更好的性能；
- 尽可能的平台无关；
- 简化往系统中添加和 删除外设的工作

https://www.cnblogs.com/machangwei-8/p/10403495.html