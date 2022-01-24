---
title: linux命令iprouter
date: 2020-11-30 10:21:45
categories: [linux]  
tags: [linux,linuxcc]
---


<!-- more -->

# linux命令iprouter
之前使用的网络命令可能是ifconfig,在后面随着linux版本的迭代,iproute是一个新的网络工具包,简称`ip命令`


## 简介
```bash
# 查看iproute包是否已经安装,正常情况下在centos7下面会自带安装的
rpm -qa|grep iproute
iproute-3.10.0-54.el7_2.1.x86_64 

# 查看iproute生成的文件
rpm -ql iproute

# 查看iproute配置文件
rpm -qc iproute

# 查看iproute包生成的二进制文件
rpm -ql iproute | grep "bin"
```

## 功能模块

```bash
ip link
        网络设备配置命令，如可以启用/禁用某个网络设备，改变mtu及mac地址等
[root@ops-2 ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
    link/ether fa:16:3e:8d:cc:28 brd ff:ff:ff:ff:ff:ff
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT
    link/ether 02:42:7a:21:f8:fa brd ff:ff:ff:ff:ff:ff


ip addr
        用于管理某个网络设备与协议(ip或ipv6)有关的地址。
        与ip link类似，不过增加了协议有关的管理(ip地址管理)
[root@ops-2 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc pfifo_fast state UP qlen 1000
    link/ether fa:16:3e:8d:cc:28 brd ff:ff:ff:ff:ff:ff
    inet 10.0.6.39/20 brd 10.0.6.127 scope global dynamic eth0
       valid_lft 114sec preferred_lft 114sec
    inet6 fe80::f816:3eff:fe8d:cc28/64 scope link tentative dadfailed
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN
    link/ether 02:42:7a:21:f8:fa brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:7aff:fe21:f8fa/64 scope link
       valid_lft forever preferred_lft forever

ip addrlabel 
        ipv6的地址标签，主要用于RFC3484中描述的ipv6地址的选择。
        RFC3484主要介绍了2个算法，用于ipv6地址(源地址和目标地址)的选择策略
[root@ops-2 ~]# ip addrlabel
prefix ::1/128 label 0
prefix ::/96 label 3
prefix ::ffff:0.0.0.0/96 label 4
prefix 2001::/32 label 6
prefix 2001:10::/28 label 7
prefix 3ffe::/16 label 12
prefix 2002::/16 label 2
prefix fec0::/10 label 11
prefix fc00::/7 label 5
prefix ::/0 label 1

ip route    
        管理路由，如添加，删除
[root@ops-2 ~]# ip route
default via 10.0.0.130 dev eth0
10.0.0.0/20 dev eth0  proto kernel  scope link  src 10.0.6.39
172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.0.1

ip rule    
        管理路由策略数据库。这里边有一个算法，用来控制路由的选择策略
[root@ops-2 ~]# ip rule
0:      from all lookup local
32766:  from all lookup main
32767:  from all lookup default

ip neigh    
        用于neighbor/ARP表的管理，如显示，插入，删除等
[root@ops-2 ~]# ip neigh
172.17.0.4 dev docker0 lladdr 02:42:ac:11:00:04 STALE
10.0.1.142 dev eth0 lladdr 94:8b:03:01:01:1d STALE
10.0.6.12 dev eth0 lladdr 00:1b:21:8d:4d:81 STALE
10.0.6.55 dev eth0 lladdr fa:16:3e:15:1c:43 STALE
10.0.6.3 dev eth0 lladdr 00:1b:21:8d:4c:a5 STALE
10.0.6.250 dev eth0 lladdr 00:1b:21:8d:4c:a5 STALE
10.0.2.17 dev eth0 lladdr 00:0e:c6:d3:91:d4 STALE
10.0.2.128 dev eth0 lladdr d0:17:c2:9b:71:45 STALE
172.17.0.3 dev docker0 lladdr 02:42:ac:11:00:03 STALE
10.0.5.201 dev eth0 lladdr e4:3a:6e:2d:5f:12 STALE
172.17.0.2 dev docker0 lladdr 02:42:ac:11:00:02 STALE
10.0.0.130 dev eth0 lladdr 84:d9:31:7f:57:c4 DELAY

ip tunel
        隧道配置
        隧道的作用是将数据(可以是不同协议)封装成ip包然后再互联网传输

ip monitor
        状态监控。如可以持续监控ip地址和路由的状态
```
