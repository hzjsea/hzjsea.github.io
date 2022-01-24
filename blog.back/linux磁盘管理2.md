---
title: linux磁盘管理2
categories:
  - linux
tags:
  - linux
abbrlink: cbc1fea8
date: 2020-02-16 23:41:43
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux磁盘和文件系统](#linux磁盘和文件系统)
  - [内存交换空间(swap)](#内存交换空间swap)
  - [创建内存交换空间(swap)](#创建内存交换空间swap)
    - [使用实体分区创建swap](#使用实体分区创建swap)

<!-- /code_chunk_output -->
<!-- more -->

# linux磁盘和文件系统
swap 有什么用

## 内存交换空间(swap)
为了防止某个程序过度或者说在某一个非常短的时间内用掉了额你大部分的内存，那你的系统恐怕会有损坏的情况。于是linux在安装的时候就被规定需要划分两个扇区
1.根目录
2.swap 内存交换空间

<font color='blue'>swap是如何工作的</font>
首先在内存足够的情况下,系统不会去调用swap中的内存，但是一旦内存不足时，就会把内存中暂时不实用的程序与数据放到swap中，此时主机的磁盘灯就会开始闪


## 创建内存交换空间(swap)
1. 使用实体分区创建swap
2. 使用虚拟内存创建swap

### 使用实体分区创建swap
首先我们需要在磁盘中划分出一块分区作为swap
```bash
fdisk /dev/sde

设备 Boot      Start         End      Blocks   Id  System
/dev/sde1            2048     1026047      512000   83  Linux 
```
其次我们需要对这个分区进行格式化 生成我们需要的文件系统mkswap
```bash
[root@test-ceph ~]# mkswap /dev/sde1
正在设置交换空间版本 1，大小 = 16380 KiB
无标签，UUID=e9b11822-d90f-4fb4-9884-e2c3ae3c9dc6
[root@test-ceph ~]# blkid /dev/sde1
/dev/sde1: UUID="e9b11822-d90f-4fb4-9884-e2c3ae3c9dc6" TYPE="swap"
```
检测,首先我们可以用free来查看下当前swap的使用情况
```bash
[root@test-ceph ~]# free
              total        used        free      shared  buff/cache   available
Mem:       16202644      317340    15316776       17188      568528    15557328
Swap:       8191996           0     8191996 
```
used 是0 ,他的总量是8191996,接下来我们挂载这个新建的swap,这里不是使用mount 而是用swapon与swapoff
```bash
swap /dev/sde1
[root@test-ceph ~]# free
              total        used        free      shared  buff/cache   available
Mem:       16202644      316220    15317572       17188      568852    15558420
Swap:       8208376           0     8208376
```
free的量已经上升了
查看交换分区由哪些构成
```bash
[root@test-ceph ~]# swapon -s
文件名                          类型            大小    已用    权限
/dev/dm-1                               partition       8191996 0       -2
/dev/sde1                               partition       16380   0       -3 
```
设置开机自动挂载
```bash
UUID="e9b11822-d90f-4fb4-9884-e2c3ae3c9dc6" swap swap defaults 0 0 
```


