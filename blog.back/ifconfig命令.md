---
title: ifconfig命令
categories:
  - linux
tags:
  - linux
abbrlink: 9db2af15
date: 2020-06-05 14:03:02
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [ifconfig命令](#ifconfig命令)
  - [使用](#使用)

<!-- /code_chunk_output -->
<!-- more -->

# ifconfig命令
`up` - 启动指定网络设备/网卡。
`down` - 关闭指定网络设备/网卡。该参数可以有效地阻止通过指定接口的IP信息流，如果想永久地关闭一个接口，我们还需要从核心路由表中将该接口的路由信息全部删除。
`arp` - 设置指定网卡是否支持ARP协议。
`- a` - 显示全部接口信息
`- s` - 显示摘要信息（类似于 netstat -i）
`add` - 给指定网卡配置IPv6地址
`del` - 删除指定网卡的IPv6地址
`address` - 为网卡设置IPv4地址




## 使用
查看网卡信息
```bash
[root@localhost ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:50:56:BF:26:20  
          inet addr:192.168.120.204  Bcast:192.168.120.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8700857 errors:0 dropped:0 overruns:0 frame:0
          TX packets:31533 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:596390239 (568.7 MiB)  TX bytes:2886956 (2.7 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:68 errors:0 dropped:0 overruns:0 frame:0
          TX packets:68 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:2856 (2.7 KiB)  TX bytes:2856 (2.7 KiB)
```
eth0 表示第一块网卡， 其中 HWaddr 表示网卡的物理地址，可以看到目前这个网卡的物理地址(MAC地址）是 00:50:56:BF:26:20
inet addr 用来表示网卡的IP地址，此网卡的 IP地址是 192.168.120.204，广播地址， Bcast:192.168.120.255，掩码地址Mask:255.255.255.0 
lo 是表示主机的回坏地址，这个一般是用来测试一个网络程序，但又不想让局域网或外网的用户能够查看，只能在此台主机上运行和查看所用的网络接口。比如把 HTTPD服务器的指定到回坏地址，在浏览器输入 127.0.0.1 就能看到你所架WEB网站了。但只是您能看得到，局域网的其它主机或用户无从知道。
第一行：连接类型：Ethernet（以太网）HWaddr（硬件mac地址）
第二行：网卡的IP地址、子网、掩码
第三行：UP（代表网卡开启状态）RUNNING（代表网卡的网线被接上）MULTICAST（支持组播）MTU:1500（最大传输单元）：1500字节
第四、五行：接收、发送数据包情况统计
第七行：接收、发送数据字节数统计信息。

<font color='red'>上面记的好像有点问题，看的时候再试一下</font>

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 70:8b:cd:7d:be:ed brd ff:ff:ff:ff:ff:ff
    inet 10.0.43.102/30 brd 10.0.43.103 scope global dynamic noprefixroute enp3s0
       valid_lft 71134sec preferred_lft 71134sec
    inet6 fdb9:d50d:a7cf:0:b489:176e:4bc1:8305/64 scope global temporary dynamic
       valid_lft 580094sec preferred_lft 61264sec
    inet6 fdb9:d50d:a7cf:0:55e:1889:4190:8a50/64 scope global temporary deprecated dynamic
       valid_lft 494125sec preferred_lft 0sec
    inet6 fdb9:d50d:a7cf:0:2dc5:f2ac:cba6:df42/64 scope global temporary deprecated dynamic
       valid_lft 408156sec preferred_lft 0sec
    inet6 fdb9:d50d:a7cf:0:247d:c394:1f04:d684/64 scope global temporary deprecated dynamic
       valid_lft 322187sec preferred_lft 0sec
    inet6 fdb9:d50d:a7cf:0:2d6f:e64b:25a6:6f48/64 scope global temporary deprecated dynamic
       valid_lft 236218sec preferred_lft 0sec
    inet6 fdb9:d50d:a7cf:0:d8ba:55aa:6c3f:d35d/64 scope global temporary deprecated dynamic
       valid_lft 150249sec preferred_lft 0sec
    inet6 fdb9:d50d:a7cf:0:c0f6:ab2c:2bf7:9957/64 scope global temporary deprecated dynamic
       valid_lft 64280sec preferred_lft 0sec
    inet6 fdb9:d50d:a7cf:0:84bd:fb4c:2eba:eafb/64 scope global mngtmpaddr noprefixroute
       valid_lft forever preferred_lft forever
    inet6 fe80::796a:41df:a3c7:daff/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

```bash
#启动关闭指定网卡
ifconfig eth0 up
ifconfig eth0 down
#为网卡配置和删除IPv6地址
ifconfig eth0 add 33ffe:3240:800:1005::2/64
ifconfig eth0 del 33ffe:3240:800:1005::2/64
#修改mac地址
ifconfig eth0 hw ether 00:AA:BB:CC:DD:EE
#配置ip地址
ifconfig eth0 192.168.120.56 netmask 255.255.255.0 broadcast 192.168.120.255
#启用和关闭arp
ifconfig eth0 arp
ifconfig eth0 -arp
#设置最大传输单元
ifconfig eth0 mtu 1500
```