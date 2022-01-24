---
title: linux命令ifconfig
date: 2020-09-23 10:01:29
categories: [linux,]  
tags: [linux,linuxcommand,net]
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux命令ifconfig](#linux命令ifconfig)
  - [什么是ifconfig](#什么是ifconfig)
  - [命令参数](#命令参数)
  - [网卡信息](#网卡信息)
  - [使用详情](#使用详情)
  - [启动关闭网卡](#启动关闭网卡)
  - [配置和删除IPV6地址](#配置和删除ipv6地址)
  - [修改mac地址](#修改mac地址)
  - [配置IP地址](#配置ip地址)
  - [启用和关闭ARP协议](#启用和关闭arp协议)
  - [设置最大传输单元](#设置最大传输单元)
    - [什么是MTU](#什么是mtu)
    - [数据包何时会分片](#数据包何时会分片)

<!-- /code_chunk_output -->
<!-- more -->   


# linux命令ifconfig
linux上一个获取网络接口配置信息的命令ifconfig 也就是interface config
命令格式
ifconfig [网路设备][参数]


## 什么是ifconfig
ifconfig 命令用来查看和配置网络设备。当网络环境发生改变时可通过此命令对网络进行相应的配置。
```bash
ifconfig
```
![2020-09-23-10-08-09](http://img.noback.top/2020-09-23-10-08-09.png)

## 命令参数

- up 启动指定网络/网卡
- down 关闭指定网络设备/网卡。该参数可以有效地阻止通过指定接口的IP信息流，如果想永久地关闭一个接口，我们还需要从核心路由表中将该接口的路由信息全部删除
- arp 设置指定网卡是否支持Arp协议
- a 显示全部接口信息
- s 显示摘要信息
- add 给指定网卡配置IPV6地址
- del 删除指定网卡的IPV6地址


## 网卡信息

![2020-09-23-10-17-13](http://img.noback.top/2020-09-23-10-17-13.png)
`<硬件地址>` 设置网卡的最大传输单元
`mtu<字节数>` 设置网卡的最大传输单元(bytes)
`netmask<子网掩码>` 设置网卡的子网掩码。掩码可以是有前缀0x的32位十六进制数，也可以是用点分开的4个十进制数。如果不打算将网络分成子网，可以不管这一选项；如果要使用子网，那么请记住，网络中每一个系统必须有相同子网掩码
`tunel` 建立隧道
`dstaddr` 设定一个远端地址，建立点对点通信
`broadcast` 广播地址
`address` 为网卡设置IPv4地址
`txqueuelen<长度>` 为网卡设置传输列队的长度


## 使用详情
查看网络设备信息(激活状态)
```bash
ifconfig
```
![2020-09-23-10-24-46](http://img.noback.top/2020-09-23-10-24-46.png)

- eth0 表示第一块网卡， 其中 HWaddr 表示网卡的物理地址，在Centos7中 网卡的物理地址为 ether
- inet 是IP地址
- netmask 是子网掩码
- boradcast 是广播地址
- lo 表示主机的回环地址，这个一般是用来测试一个网络程序，但又不想让局域网或外网的用户能够查看，只能在此台主机上运行和查看所用的网络接口。比如把 HTTPD服务器的指定到回坏地址，在浏览器输入 127.0.0.1 就能看到你所架WEB网站了。但只是您能看得到，局域网的其它主机或用户无从知道。


## 启动关闭网卡
```bash
ifconfig eth0 up
ifconfig eth0 down
```

## 配置和删除IPV6地址
```bash
ifconfig eth0 add 33ffe:3240:800:1005::2/64
ifconfig eth0 del 33ffe:3240:800:1005::2/64
```

## 修改mac地址
```bash
# 先关闭网卡
ifconfig eth0 down
ifconfig eth0 hw ether 00:AA:BB:CC:DD:EE
# 在开启网卡
ifconfig eth0 up
```

## 配置IP地址
```bash
ifconfig eth0 192.168.20.1
ifconfig eth0 192.168.20.1 netmask 255.255.255.0
ifconfig eth0 192.168.120.1 netmask 255.255.255.0 boradcast 192.168.120.255
```


## 启用和关闭ARP协议
```bash
ifconfig eth0 arp
ifocnfig eth0 -arp
```

## 设置最大传输单元
```bash
ifconfig eth0 mtu 1500
```

### 什么是MTU
在网络中，`最大传输单元(MTU)`是代表连接互联网的设备可以接受的最大数据包的度量单位。它可以比作高速公路地下通道或隧道的高度限制：超过高度限制的汽车和卡车无法通过，就像超过网络 MTU 的数据包无法通过该网络一样。
不过，与汽车和卡车不同的是，超过 MTU 的数据包可被分解成较小的碎片，从而能通过网络。这个过程称为分片。分片的数据包在到达目的地后便会重新组装。MTU 以字节数为单位,一个“字节”等于8位信息，即8个一和零。1500字节是最大 MTU 大小

### 数据包何时会分片
当两个计算设备打开连接并开始交换数据包时，这些数据包会在多个网络中路由。需要考虑的不仅是每一通信两端的两个设备的 MTU，还有中间的所有路由器、交换机和服务器。超过网络路径中任一点上 MTU 的数据包都会被分片。

假设服务器 A 和计算机 A 彼此连接，但是它们相互发送的数据包必须沿途经过路由器 B 和路由器 C。服务器 A、计算机 A 和路由器 B 的 MTU 均为 1,500 字节。不过，路由器 C 的 MTU 为 1,400 字节。如果服务器 A 和计算机 A 不知道路由器 C 的 MTU 并且发送了 1500 字节的数据包，则所有数据包会在传输过程中被路由器 B 分片

分片会给网络通信增加少许延迟和低效率，因此应当要尽可能避免。（过时的网络设备可能容易遭受利用分片的拒绝服务攻击，例如死亡之 Ping 攻击
不断的让网络设备进行数据分包，直到内存占用满
https://www.cloudflare.com/zh-cn/learning/network-layer/what-is-mtu/