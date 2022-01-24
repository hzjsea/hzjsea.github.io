---
title: linux命令ping
date: 2020-09-22 17:07:52
categories: [linux,]  
tags: [linux,net,linuxcommand]
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- more -->

# linux命令ping
linux中的ping通过发送ICMP ECHO_REQUEST数据包到网络主机（send ICMP ECHO_REQUEST to network hosts），并显示响应情况，这样我们就可以根据它输出的信息来确定目标主机是否可访问（但这不是绝对的）。有些服务器为了防止通过ping探测到，通过防火墙设置了禁止ping或者在内核参数中禁止ping，这样就不能通过ping确定该主机是否还处于开启状态。

## 什么是ping
ping命令用于
- 确定网络和各外部主机的状态；
- 跟踪和隔离硬件和软件问题；
- 测试、评估和管理网络。
如果主机正在运行并连在网上，它就对回送信号进行响应。每个回送信号请求包含一个网际协议（IP）和 ICMP 头，后面紧跟一个 tim 结构，以及来填写这个信息包的足够的字节


## 使用详情
直接ping对端主机
```bash
ping 10.0.6.55
```
![2020-09-22-17-10-55](http://img.noback.top/2020-09-22-17-10-55.png)



## 指定次数ping 
```bash
ping -c 10 10.0.6.55
```
![2020-09-22-17-12-26](http://img.noback.top/2020-09-22-17-12-26.png)


## 指定时间间隔
每0.5s ping对端主机
```bash
ping -c 10 -i 0.5 10.0.6.55
```

## ping公网上的域名站点
```bash
ping -c 5 www.baidu.com
```

## 设置发送包大小
默认发送64bytes包的大小
```bash
ping -c 5 -i 1024 www.baidu.com
```
![2020-09-22-17-15-21](http://img.noback.top/2020-09-22-17-15-21.png)
1024+8 =1032

## 设置ping的时间
设置时间为100s
```bash
ping -W 100 10.0.6.35
```

## 什么是TTL值
当我们在使用ping命令时，返回结果里会带一个TTL值。这个东西的含义其实就是Time To Live，指的是报文在网络中能够‘存活’的限制。以前这个限制方式是设定一个时间（Time To Live中的Time就是这样来的），
- 当报文在网络中转发时，时间超过这个限制，最后一个收到报文的‘路由点’就会把它扔掉，而不继续转发。
- 后来把时间限制改为了跳数限制，就是当报文在网络中转发时，每经过一个‘路由点‘，就把预先设定的这个TTL数值减1，直到最后TTL=1时报文就被扔掉，不向下转发。
现在的TTL值都是跳数限制，这样不会因为网络限制导致时间上收到影响

ping里面的TTL值值得是他预设的TTL值到我本机还剩下多少TTL值，  比如他预设了TTL=64 那么他能到的主机只能在64跳里面完成，如果是65 则表示丢包
现在假设到我这里只需要4跳  那么到我这里就死64-4 = 60 跳 ping里面就会显示TTL=60


实验 
我们在测试机A上做实验
现在我们用traceroute来做测试
```bash
yum install traceroute -y
```
现在我们测下测试机A到对端机器B需要多少条
```bash
traceroute 10.0.6.35
```
![2020-09-22-17-43-38](http://img.noback.top/2020-09-22-17-43-38.png)
测试机A到对端机器B 需要消耗1跳 ，上图中第一个为当前IP
这样的话从对端机器Bping 测试机A的 消耗就是63
![2020-09-22-17-45-17](http://img.noback.top/2020-09-22-17-45-17.png)

相反的TTL值并不一定相同
当我们在B机器上看到A机器需要多少跳时 显示的是3-1= 2跳
![2020-09-22-17-46-05](http://img.noback.top/2020-09-22-17-46-05.png)
于是当A去pingB的时候 显示的则是64-2 = 62
![2020-09-22-17-46-45](http://img.noback.top/2020-09-22-17-46-45.png)


### 查看TTL值
在网络中，黑客如果用ping命令去探测  一个主机，根据TTL基数可以推测操作系统的类型。对于一个没有经过任何网关和路由的网络
因为在出厂的系统当中，TTL值是固定的，通常情况下  `Windows的TTL`的基数是128，而`Centos 6.3（linux系统）和 macos`的TTL基数是64

查看TTL值
```bash
cat /proc/sys/net/ipv4/ip_default_ttl
```
修改TTL值
```bash
echo 128 > /proc/sys/net/ipv4/ip_default_ttl   
```
永久修改
```bash
cat /etc/sysctl.conf < EOF
net.ipv4.ip_default_ttl = 128
EOF

sysctl -p # 重载配置文件
```

## ping的时候设置ttl值
默认ttl值为62
```bash
ping -c 5 -t 255 www.baidu.com
```

## 快速大包高ping
```bash
ping -f -s 65507 10.0.6.55
```
此用法非常危险，`65535（包头+内容）*100个包每秒=6.25MB`，每秒发送6.25MB的数据，相当于50Mbps的带宽，完全可能导致目标主机拒绝服务。请勿用于非法用途，造成不良后果自负

