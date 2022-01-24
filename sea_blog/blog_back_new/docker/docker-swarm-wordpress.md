---
title: docker-swarm-wordpress
categories:
  - docker
tags:
  - docker
abbrlink: 1976fb59
date: 2020-02-09 17:35:49
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker-swarm-wordpress](#docker-swarm-wordpress)
  - [实战](#实战)
  - [集群服务间通信](#集群服务间通信)

<!-- /code_chunk_output -->
<!-- more -->


# docker-swarm-wordpress
通过docker + swarm + wordpress 建立集群


## 实战
在不指定同网络组的情况下，docker默认将所有的ip都归类到名字叫做docker0的bridge组里面，但是这样在跨机器的容器中，构建会十分麻烦，无法通过容器名来进行ip通信
因此，在创建的过程中需要指定网络组

1. 创建一个overlay的网络
```bash
docker network create -d overlay demo
# 查看是否创建成功 
[root@manager55 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
ee5b54750620        bridge              bridge              local
mxmjjml84ldx        demo                overlay             swarm
6ad358de8e6b        docker_gwbridge     bridge              local
f127d4764edd        host                host                local
ihvz4z5lfwis        ingress             overlay             swarm   <----这里
83efd9726723        none                null                local
```
一个manager 两台work都有了这个网络
2. 创建服务
```bash
# mysql
docker service create --name mysql --env MYSQL_ROOT_PASSWORD=root --env MYSQL_DATABASE=wordpress --network demo --mount type=volume,source=mysql-data,destination=/var/lib/mysql mysql:5.7 

# wordpress
docker service create --name wordpress -p 80:80 --env WORDPRESS_DB_PASSWORD=root --env WORDPRESS_DB_HOST=mysql --network demo wordpress
```
## 集群服务间通信
在一个service ,加入到overlay网络的机器都会添加一条DNS,docker基于DNS服务发现记录主机名与ip的对应关系
但由于docker swarm的横向扩展性，机器的ip随时都会改变，因此记录的并不是容器所在机器的ip，而是一个vip  virtual ip 


1. 创建一个overlay的网络
```bash 
docker network create -d overylay demo 
```
2. 创建一个web服务,并加入demo这个网络
```bash
docker service create --name whoami -p 8000:8000 --network demo -d jwilder/whoami 
```
3. 查看所建立的service
```bash
[root@manager55 ~]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                   PORTS
nnw1gekvbv0t        client              replicated          1/1                 busybox:latest
3s9kfmq0eo8c        whoami              replicated          1/1                 jwilder/whoami:latest   *:8000->8000/tcp
[root@manager55 ~]# docker service ps client
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
t7xe5omklz28        client.1            busybox:latest      work196             Running             Running 15 minutes ago 
```
可以发现busybox的容器是运行在work196上面的，这个时候我们去busybox ping whoami这个容器名，
```bash
[root@work196 ~]# docker exec bd2b98fdc7f9 ping whoami
PING whoami (10.0.2.2): 56 data bytes
64 bytes from 10.0.2.2: seq=0 ttl=64 time=0.077 ms
64 bytes from 10.0.2.2: seq=1 ttl=64 time=0.194 ms
64 bytes from 10.0.2.2: seq=2 ttl=64 time=0.193 ms 
# 查看whoami容器的ip
[root@manager55 ~]# docker ecec dfcd2f4c0396 ip a
docker: 'ecec' is not a docker command.
See 'docker --help'
[root@manager55 ~]# docker exec  dfcd2f4c0396 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
31: eth1@if32: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue state UP
    link/ether 02:42:0a:00:00:08 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.8/24 brd 10.0.0.255 scope global eth1
       valid_lft forever preferred_lft forever
33: eth2@if34: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:12:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.3/16 brd 172.18.255.255 scope global eth2
       valid_lft forever preferred_lft forever
35: eth0@if36: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue state UP
    link/ether 02:42:0a:00:02:03 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.3/24 brd 10.0.2.255 scope global eth0
       valid_lft forever preferred_lft forever
```
在ping whoami容器时候，并没有其ping他的任何一个ip说明在这之上又做了一层vip，用来统一管理所有容器名为whoami的机器

4. 扩展whoamin容器
```bash
docker service scale whami=4 
```
再次使用ping的时候会发现，同样ping的是这个vip

5. 查看虚拟ip下面的ip
```bash
[root@work196 ~]# docker exec bd2b98fdc7f9 nslookup tasks.whoami
Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:
Name:   tasks.whoami
Address: 10.0.2.10
Name:   tasks.whoami
Address: 10.0.2.3
Name:   tasks.whoami
Address: 10.0.2.9
Name:   tasks.whoami
Address: 10.0.2.8

*** Can't find tasks.whoami: No answer 
```
其实这些内容就是vip下面的真正的IP
因此service 与service之间是通过vip进行通信的