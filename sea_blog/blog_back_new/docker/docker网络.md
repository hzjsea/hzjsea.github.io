---
title: docker网络
categories:
  - docker
tags:
  - docker
abbrlink: '20783356'
date: 2020-02-06 22:11:37
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker网络](#docker网络)
  - [linux的网络命名空间namesapce](#linux的网络命名空间namesapce)
    - [network namespace管理工具](#network-namespace管理工具)
  - [docker网络管理工具](#docker网络管理工具)
  - [容器与容器之间如何互联](#容器与容器之间如何互联)
  - [容器与外界互联](#容器与外界互联)
  - [docker容器之间的link](#docker容器之间的link)
  - [自建docker网络桥接](#自建docker网络桥接)
    - [修改旧容器到指定bridge](#修改旧容器到指定bridge)
  - [none与host](#none与host)

<!-- /code_chunk_output -->
<!-- more -->

# docker网络
docker网络的几种模式
![2020-02-06-22-12-35](http://noback.upyun.com/2020-02-06-22-12-35.png)


<font color='red'>网路地址转换NAT</font>
内网转外网
![2020-02-06-22-21-07](http://noback.upyun.com/2020-02-06-22-21-07.png)

## linux的网络命名空间namesapce
```bash
root:~# sudo ip a | tail -6
8: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:4f:aa:73:10 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0                  <-------------看这里
       valid_lft forever preferred_lft forever
    inet6 fe80::42:4fff:feaa:7310/64 scope link    
       valid_lft forever preferred_lft forever 
root:~# docker run -it -d  --name b1 busybox  /bin/sh -c "while true; do sleep 3600;done"
890e3e47feea65be2d11122e01739247d41cf45e9ff8a2b88157482ee02a3b6f
root:~# docker run -it -d  --name b2 busybox  /bin/sh -c "while true; do sleep 3600;done"
db45d33d512104ec2d799bd45ba47095ffb5c1c8253ede8becb808c47b902f7b
root:~# dokcer exec -it b1 /bin/sh
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
167: eth0@if168: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0         <-------------看这里
       valid_lft forever preferred_lft forever
root:~# docker exec -it b2 /bin/sh
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
169: eth0@if170: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0           <-------------看这里
       valid_lft forever preferred_lft forever
```
<font color='red'>可以发现实体机专门给docker机器独立了一个内网ip给docker，并且docker容器之间是可以相互ping同的</font>

### network namespace管理工具
https://www.cnblogs.com/bakari/p/10443484.html
```bash
root:~# sudo ip netns help
Usage: ip netns list
       ip netns add NAME
       ip netns set NAME NETNSID
       ip [-all] netns delete [NAME]
       ip netns identify [PID]
       ip netns pids NAME
       ip [-all] netns exec [NAME] cmd ...
       ip netns monitor
       ip netns list-id 
```
在docker中，我们安装了两个容器 b1 与 b2 同样的我们也可以用这个管理工具创建两个空间b1 b2 然后进入查看ip
```bash
root:~# sudo ip netns add b1
root:~# sudo ip netns add b2
root:~# sudo ip netns exec b1 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
root:~# sudo ip netns exec b2 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```
可以看到在b1 b2两个空间中都有一张网卡，使用link将它开启
```bash
root:~# sudo ip netns exec b1 ip link set dev lo up
root:~# sudo ip netns exec b2 ip link set dev lo up
root:~# sudo ip netns exec b1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
root:~# sudo ip netns exec b2 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever 
```
现在只有本地的回环口,添加一队veth
```bash
root:~# sudo ip link add veth-test1 type veth peer name veth-test2
root:~# sudo ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
7: enp1s0f1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:1b:21:bf:0d:9e brd ff:ff:ff:ff:ff:ff
8: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:4f:aa:73:10 brd ff:ff:ff:ff:ff:ff
168: veth5b2efd9@if167: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether c6:64:94:8a:43:0a brd ff:ff:ff:ff:ff:ff link-netnsid 0
170: veth6d83aa9@if169: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 36:a5:3e:2a:69:f7 brd ff:ff:ff:ff:ff:ff link-netnsid 1
171: veth-test2@veth-test1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 56:0d:bc:10:fa:c0 brd ff:ff:ff:ff:ff:ff
172: veth-test1@veth-test2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether be:37:b1:1c:09:55 brd ff:ff:ff:ff:ff:ff 
```
添加到建立的namespace中去
```bash
root:~# sudo ip link set veth-test1 netns b1
root:~# sudo ip link set veth-test2 netns b2
root:~# sudo ip netns exec b1 ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
172: veth-test1@if171: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether be:37:b1:1c:09:55 brd ff:ff:ff:ff:ff:ff link-netnsid 1
root:~# sudo ip netns exec b2 ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
171: veth-test2@if172: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 56:0d:bc:10:fa:c0 brd ff:ff:ff:ff:ff:ff link-netnsid 0 
```
分配ip
```bash
root:~# sudo ip netns exec b1 ip addr add 192.168.20.1/24 dev veth-test1
root:~# sudo ip netns exec b2 ip addr add 192.168.20.2/24 dev veth-test2
root:~# sudo ip netns exec b1 ip link set dev veth-test1 up
root:~# sudo ip netns exec b2 ip link set dev veth-test2 up
root:~# sudo ip netns exec b1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
172: veth-test1@if171: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether be:37:b1:1c:09:55 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 192.168.20.1/24 scope global veth-test1
       valid_lft forever preferred_lft forever
    inet6 fe80::bc37:b1ff:fe1c:955/64 scope link
       valid_lft forever preferred_lft forever
root:~# sudo ip netns exec b2 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
171: veth-test2@if172: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 56:0d:bc:10:fa:c0 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.20.2/24 scope global veth-test2
       valid_lft forever preferred_lft forever
    inet6 fe80::540d:bcff:fe10:fac0/64 scope link
       valid_lft forever preferred_lft forever
```
测试是否连通
```bash
root:~# sudo ip netns exec b2 ping 192.168.20.1
PING 192.168.20.1 (192.168.20.1) 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_seq=1 ttl=64 time=0.045 ms
64 bytes from 192.168.20.1: icmp_seq=2 ttl=64 time=0.035 ms
64 bytes from 192.168.20.1: icmp_seq=3 ttl=64 time=0.045 ms 
```
![2020-02-06-23-14-30](http://noback.upyun.com/2020-02-06-23-14-30.png)


## docker网络管理工具
```bash
# 查看dokcer状态
root:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
dcb97917d7b9        bridge              bridge              local  # 网桥连接
fd363f5c42dc        host                host                local
2e0f440a1b7b        none                null                local
```

安装bridge-utils，这个是一种网桥状态查看工具
```bash
yum install bridge-utils -y

root:~# sudo brctl
Usage: brctl [commands]
commands:
        addbr           <bridge>                add bridge
        delbr           <bridge>                delete bridge
        addif           <bridge> <device>       add interface to bridge
        delif           <bridge> <device>       delete interface from bridge
        hairpin         <bridge> <port> {on|off}        turn hairpin on/off
        setageing       <bridge> <time>         set ageing time
        setbridgeprio   <bridge> <prio>         set bridge priority
        setfd           <bridge> <time>         set bridge forward delay
        sethello        <bridge> <time>         set hello time
        setmaxage       <bridge> <time>         set max message age
        setpathcost     <bridge> <port> <cost>  set path cost
        setportprio     <bridge> <port> <prio>  set port priority
        show            [ <bridge> ]            show a list of bridges
        showmacs        <bridge>                show a list of mac addrs
        showstp         <bridge>                show bridge stp info
        stp             <bridge> {on|off}       turn stp on/off
```

## 容器与容器之间如何互联
之前做的network namespace是通过veth来形成对，达到两个namespace的空间进行ip通信，在docker中则是使用docker0(bridge)进行ip通信的 

开启两个docker容器
```bash
root:~# docker run -it -d  --name b1 busybox  /bin/sh -c "while true; do sleep 3600;done"
890e3e47feea65be2d11122e01739247d41cf45e9ff8a2b88157482ee02a3b6f
root:~# docker run -it -d  --name b2 busybox  /bin/sh -c "while true; do sleep 3600;done"
db45d33d512104ec2d799bd45ba47095ffb5c1c8253ede8becb808c47b902f7b
root:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
db45d33d5121        busybox             "/bin/sh -c 'while t…"   About an hour ago   Up About an hour                        b2
890e3e47feea        busybox             "/bin/sh -c 'while t…"   About an hour ago   Up About an hour                        b1
```
查看本机网卡状态
```bash
root:~# sudo ip link | tail -6
8: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:4f:aa:73:10 brd ff:ff:ff:ff:ff:ff
168: veth5b2efd9@if167: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether c6:64:94:8a:43:0a brd ff:ff:ff:ff:ff:ff link-netnsid 0
170: veth6d83aa9@if169: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 36:a5:3e:2a:69:f7 brd ff:ff:ff:ff:ff:ff link-netnsid 1 
```
查看网桥状态
```bash
root:~# sudo brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.02424faa7310       no              veth5b2efd9
                                                        veth6d83aa9
```
我们会法相两个interface正好对应网卡中的dev,其实这是容器连接外部host主机的两个接口
查看两个容器内部的接口
```bash
root:~# docker exec -it b1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
167: eth0@if168: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
root:~# docker exec -it b2 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
169: eth0@if170: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever 
```
内部通过eth0连接docker0 
```bash
docker network inspect bridge  
```
![2020-02-07-00-01-42](http://noback.upyun.com/2020-02-07-00-01-42.png)

## 容器与外界互联
主要是做nat内外网地址转发
![2020-02-07-00-03-58](http://noback.upyun.com/2020-02-07-00-03-58.png)



## docker容器之间的link
可以直接通过ping name的形式ping ip
创建一个容器
```bash
root:~# docker run -it -d  --name b1 busybox  /bin/sh -c "while true; do sleep 3600;done"
890e3e47feea65be2d11122e01739247d41cf45e9ff8a2b88157482ee02a3b6f
root:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
db45d33d5121        busybox             "/bin/sh -c 'while t…"   About an hour ago   Up About an hour                        b2
```
创建第二个容器，在创建的过程中，通过--link连接到b1
```bash
root:~# docker run -it --link b1 busybox
/ # ping b1
PING b1 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.248 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.102 ms
```
<font color='red'>link只是单向的连接，并不是双向连接</font>

<font color='blue'>有一个例外，由于docker0的bridge是系统默认已经创建好的，上面的容器是没有link的，但是我们创建了自己的bridge  new-bridge，上面的容器默认是已经互相link的</font>


## 自建docker网络桥接
docker默认有三种形式的连接
```bash
root:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
dcb97917d7b9        bridge              bridge              local   <----叫做docker0
fd363f5c42dc        host                host                local
2e0f440a1b7b        none                null                local 
root:~# sudo brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.02424faa7310       no              veth5b2efd9
                                                        veth74e040d
```
其中容器之间使用bridge进行连接的，但我们也可以自己创建
```bash
root:~# docker network create -d bridge new-bridge
a52b83252b814bbe2299142fb25551e6734b311fce3d749d12b7d1b2defd0d41
root:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
dcb97917d7b9        bridge              bridge              local   <----叫做docker0
fd363f5c42dc        host                host                local
a52b83252b81        new-bridge          bridge              local   <----叫做br-a52b83252b81
2e0f440a1b7b        none                null                local 
root:~# sudo brctl show
bridge name     bridge id               STP enabled     interfaces
br-a52b83252b81         8000.02429efea748       no
docker0         8000.02424faa7310       no              veth5b2efd9
                                                        veth74e040d
```
创建新容器的时候指定bridge
```bash
root:~# docker run -it -d  --name b4 --network new-bridge  busybox  /bin/sh -c "while true; do sleep 3600;done"
a60bdef872f7c16305b32920d60e4d973bcd5329a912eaff32b476debf3e6aaa
root:~# sudo brctl show
bridge name     bridge id               STP enabled     interfaces
br-a52b83252b81         8000.02429efea748       no              veth8b0f5f0
docker0         8000.02424faa7310       no              veth5b2efd9
                                                        veth74e040d 
```

### 修改旧容器到指定bridge
```bash
docker network connect new-bridge b1
docker network connect new-bridge b2 
```
这时候其实是我们又给容器新添了一张网卡
```bash
root:~# docker exec b2 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
179: eth0@if180: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
190: eth1@if191: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:12:00:04 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.4/16 brd 172.18.255.255 scope global eth1
       valid_lft forever preferred_lft forever 
```
查看bridge
```bash
root:~# sudo brctl show
bridge name     bridge id               STP enabled     interfaces
br-a52b83252b81         8000.02429efea748       no              veth1ce8b2f
                                                        veth7001337
                                                        veth8b0f5f0
docker0         8000.02424faa7310       no              veth5b2efd9
                                                        veth74e040d 
```
查看bridge详情
```bash
# docker0的网卡
docker network inspect bridge
"Containers": {
            "890e3e47feea65be2d11122e01739247d41cf45e9ff8a2b88157482ee02a3b6f": {
                "Name": "b1",
                "EndpointID": "13c791d9040e3cb9521e53ef0e56b6f87b9564d9ad3e6af4ee38bd2d8fdc53b5",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "db45d33d512104ec2d799bd45ba47095ffb5c1c8253ede8becb808c47b902f7b": {
                "Name": "b2",
                "EndpointID": "d155a2a32b4db93753084739ee81710c83c397c8fdf7aa2d799ccad123fa71c9",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
# new-bridge的网卡
docker network inspect new-bridge
 "Containers": {
            "890e3e47feea65be2d11122e01739247d41cf45e9ff8a2b88157482ee02a3b6f": {
                "Name": "b1",
                "EndpointID": "8cd4ff8ff0b032fb72cdfaa3b02b16b9e564c75f2d8844e5cad680b02e76387a",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "a60bdef872f7c16305b32920d60e4d973bcd5329a912eaff32b476debf3e6aaa": {
                "Name": "b4",
                "EndpointID": "e41fb0b3e878af1596e80f2752fcc1dff2ac68a00d53163aded9aea31ead893d",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "db45d33d512104ec2d799bd45ba47095ffb5c1c8253ede8becb808c47b902f7b": {
                "Name": "b2",
                "EndpointID": "e77bf7b9ae54e3f652fa4e053c683f85ec3c4308c50ca8225d8722ddbae2e474",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            }
        },
```

## none与host
创建一个只有本地回环的容器
```bash
docker run -it -d --name b5 --network none busybox /bin/sh 
root:~# docker exec b5 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

<font color='red'>创建一台网卡配置与主机相同容器</font>

```bash
docker run -it -d --name b6 --network host busybox /bin/sh
root:~# docker exec b6 ip a
root:~# docker exec b6 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp3s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq qlen 1000
    link/ether 00:8c:fa:59:e9:ac brd ff:ff:ff:ff:ff:ff
3: enp4s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq qlen 1000
    link/ether 00:8c:fa:59:e9:ad brd ff:ff:ff:ff:ff:ff
4: enp9s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq qlen 1000
    link/ether 00:8c:fa:59:e9:ae brd ff:ff:ff:ff:ff:ff
    inet 10.0.5.233/20 brd 10.0.15.255 scope global enp9s0
       valid_lft forever preferred_lft forever
    inet6 fe80::748:8e4c:c635:1963/64 scope link
       valid_lft forever preferred_lft forever
5: enp10s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq qlen 1000
    link/ether 00:8c:fa:59:e9:af brd ff:ff:ff:ff:ff:ff
...

```


