---
title: linux命令tcpdump
date: 2020-09-22 16:55:12
categories: [linux,]  
tags: [linux,net,linuxcommand]
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux命令tcpdump](#linux命令tcpdump)
  - [什么是tcpdump](#什么是tcpdump)
  - [使用详情](#使用详情)
  - [监听网卡数据包](#监听网卡数据包)
  - [指定网卡监听](#指定网卡监听)
  - [监听特定主机](#监听特定主机)
  - [监听特定来源](#监听特定来源)
  - [监听特定目标地址](#监听特定目标地址)
  - [监听特定端口](#监听特定端口)
  - [监听特定主机之间的通信](#监听特定主机之间的通信)
  - [指定协议](#指定协议)
  - [禁用时间戳](#禁用时间戳)
  - [指定数据报数量](#指定数据报数量)
  - [禁止目标端口](#禁止目标端口)
  - [根据掩码规范网络地址范围](#根据掩码规范网络地址范围)
  - [保存文件](#保存文件)
  - [详细内容](#详细内容)

<!-- /code_chunk_output -->
<!-- more -->


# linux命令tcpdump
初始化的系统(centos7)中没有tcpdump这个包，需要自己安装
```bash
yum install tcpdump -y
```

## 什么是tcpdump
网络数据包截获分析工具。支持针对网络层、协议、主机、网络或端口的过滤。并提供and、or、not等逻辑语句帮助去除无用的信息。
dump traffic on a network

## 使用详情
默认情况下 服务器只有一个网卡 eth0,如下
![2020-09-22-17-00-00](http://img.noback.top/2020-09-22-17-00-00.png)
一个回环地址，一个网卡


## 监听网卡数据包
默认情况下会去监听第一张网卡上的数据包
```bash
tcpdump
```

## 指定网卡监听
```bash
tcpdump -i eth0
```

## 监听特定主机
监听本机和特定主机之间的通信包
出入包都会被监听
```bash
tcpmap host 10.0.6.39
```
如上监听了来自10.0.6.39的通信包，然后我们在10.0.6.39主机上面去`ping -c 1 10.0.6.35`看看变化

![2020-09-22-18-06-29](http://img.noback.top/2020-09-22-18-06-29.png)

## 监听特定来源
```bash
tcpdump src host 10.0.6.39
```

## 监听特定目标地址
```bash
tcpdump dst host 10.0.5.39
```


## 监听特定端口
```bash
tcpdump port 300
```

## 监听特定主机之间的通信
设置来源为10.0.6.35  地址为10.0.6.39
```bash
tcpdump ip host 10.0.6.35 and 10.0.6.39
```
设置来源为10.0.6.35 地址为除了10.0.6.39 其他
```bash
tcpdump ip host 10.0.6.35 and ! 10.0.6.39
```


## 指定协议
```bash
tcpdump icmp host 10.0.6.35
```
这里的协议包括  `ip  tcp icmp arp rarp tcp udp` 要放到第一个参数的位置

## 禁用时间戳
```bash
tcpdump -t host 10.0.6.35
```

## 指定数据报数量
```bash
tcpdump -c 100 host 10.0.6.35
```

## 禁止目标端口
```bash
tcpdump -c 100 -t and host 10.0.6.39 and port !22
```

## 根据掩码规范网络地址范围
```bash
tcpdump -c 100 -t  and src net 10.0.6.35/24
```

## 保存文件
备注：tcpdump默认会将输出写到缓冲区，只有缓冲区内容达到一定的大小，或者tcpdump退出时，才会将输出写到本地磁盘
```bash
tcpdump -c 100 -t  and host 10.0.6.30 -w ./target.cap
```



## 详细内容
https://www.cnblogs.com/chyingp/p/linux-command-tcpdump.html
tcpdump 很详细的
http://blog.chinaunix.net/uid-11242066-id-4084382.html

http://www.cnblogs.com/ggjucheng/archive/2012/01/14/2322659.html
Linux tcpdump命令详解

Tcpdump usage examples（推荐）
http://www.rationallyparanoid.com/articles/tcpdump.html

使用TCPDUMP抓取HTTP状态头信息
http://blog.sina.com.cn/s/blog_7475811f0101f6j5.html